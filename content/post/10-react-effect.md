---
title: "React Effect"
date: 2022-07-13T11:40:17+08:00
lastmod: 2022-07-13T11:40:17+08:00
draft: false
toc: true
keywords: ["daily"]
description: "新文档里的 React Effects"
tags: ["JavaScript", "React"]
author: "xuyou"
---

在 React 组件中有两种逻辑：

<!--more-->

- 渲染代码（Rendering code）：使用 `props` 和 `state` 计算出一段 JSX。渲染代码需要是纯函数，不能有额外的操作。
- 事件处理（Event handlers）：组件内部包含的函数，用于响应用户操作，可以包含副作用。

但是并不是所有逻辑都可以在 Event handlers 中解决，比如一个聊天室，接受消息的逻辑并没有用户交互触发，这个时候就需要 effect。所以 **Effect 用于处理由渲染本身而不是由特定事件引起的副作用。**

## 可能不需要用 Effect 

以下是一些 Effect 的替代场景。

### 1、通过 props 和 state 更新视图

可以在渲染代码中计算并使用，例如：

```jsx
// ...
const [firstName, setFirstName] = useState('Taylor');
const [lastName, setLastName] = useState('Swift');

// 🔴 避免额外的state 参数
const [fullName, setFullName] = useState('');
useEffect(() => {
  setFullName(firstName + ' ' + lastName);
}, [firstName, lastName]);

// ✅ 直接使用计算的变量
const fullName = firstName + ' ' + lastName;
```

如果是比较复杂的计算获得的变量，可以使用 memo 来进行优化：

```jsx
// ✅ 如果 getFilteredTodos() 计算量不大
const visibleTodos = getFilteredTodos(todos, filter);

const visibleTodos = useMemo(() => {
// ✅ 如果计算量较大，仅 todos 和 filter 变化了才会重新计算
  return getFilteredTodos(todos, filter);
}, [todos, filter]);
```

### 2、props 变化时重置 state

例如 `userId` 变化时清空 `comment`：

```jsx
export default function Profile({ userId }) {
  const [comment, setComment] = useState('');

  // 🔴 避免在 effect 里面重置监听 props 变化重置 state
  useEffect(() => {
    setComment('');
  }, [userId]);
  // ...
}
```

可以通过父组件传递不同的 key prop 来实现 state 重置的效果：

```jsx
export default function ProfilePage({ userId }) {
  return (
    <Profile
      userId={userId} 
      key={userId}
    />
  );
}

export default function Profile({ userId }) {
  // ✅ 父组件会对子组件进行重新挂载
  const [comment, setComment] = useState('');
}
```

### 3、props 变化时修改部分 state

这种场景也很常见，例如：

```jsx
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selection, setSelection] = useState(null);

  // 🔴 items 变化时不需要使用 effect 来取消选中，会触发重新渲染
  useEffect(() => {
    setSelection(null);
  }, [items]);
  // ...
}
```

但是我们完全可以在渲染代码中实现这部分逻辑：

```jsx
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selection, setSelection] = useState(null);

  // Better: 通过判断上一次的 props 和这一次不一样
  const [prevItems, setPrevItems] = useState(items);
  if (items !== prevItems) {
    setPrevItems(items);
    setSelection(null);
  }
  // ...
}
```

当在渲染期间更新组件时，React 会丢弃之前的 JSX 并立即重新渲染。为了避免非常缓慢的级联重试（cascading retries），React 只允许在渲染期间更新**相同**组件的状态。如果想在渲染期间更新另一个组件的状态，则会抛出错误。像 `items !== prevItems` 这样的条件判断可以有效地避免循环。这样设置 state 是 OK 的，但其他副作用（如更改 DOM 或设置定时器）应保留在事件处理程序或 Effect 中，以保证我们的组件的可预测性。

自然还是有更好的方法来解决这个场景，例如：

```jsx
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selectedId, setSelectedId] = useState(null);
  // ✅ 在渲染期间计算选中态
  const selection = items.find(item => item.id === selectedId) ?? null;
  // ...
}
```

### 4、初始化 APP

一些逻辑只需要运行一次，例如初始化一个程序，我们可能会这样写：

```jsx
function App() {
  // 🔴 这里的逻辑只应该运行一次
  useEffect(() => {
    loadDataFromLocalStorage();
    checkAuthToken();
  }, []);
  // ...
}
```

但是在开发模式下这段逻辑会跑两次，对于某些程序来说可能会有问题，所以推荐下面的解决方式：

```jsx
let didInit = false;

function App() {
  useEffect(() => {
    if (!didInit) {
      didInit = true;
      // ✅ 保证只会运行一次
      loadDataFromLocalStorage();
      checkAuthToken();
    }
  }, []);
  // ...
}
```

或者像下面这样：

```jsx
if (typeof window !== 'undefined') { 
   // ✅ 仅会运行一次
  checkAuthToken();
  loadDataFromLocalStorage();
}

function App() {
  // ...
}
```

### 5、解决竞态问题

```jsx
function SearchResults({ query }) {
  const [results, setResults] = useState([]);
  const [page, setPage] = useState(1);

  useEffect(() => {
    // 🔴 下面的逻辑没有 cleanup function
    fetchResults(query, page).then(json => {
      setResults(json);
    });
  }, [query, page]);

  function handleNextPageClick() {
    setPage(page + 1);
  }
  // ...
}
```

这段代码为什么没有放到事件处理函数中呢，因为依赖的字段并不一定是来自用户交互（输入、点击），比如 `query` 参数来自于页面 url，就不能在事件处理函数中进行响应了。

想象一个场景，`query` 参数迅速地从 `1` 变成了 `11`，然而响应却不一定会按照顺序，如果后面的请求先返回了结果，页面显示就会有问题，即竞态问题（[race condition](https://maxrozen.com/race-conditions-fetching-data-react-with-useeffect)）。

怎么解决这个问题呢，可以看下面的代码：

```jsx
function SearchResults({ query }) {
  const [results, setResults] = useState([]);
  const [page, setPage] = useState(1); 
  useEffect(() => {
    let ignore = false;
    fetchResults(query, page).then(json => {
      if (!ignore) {
        // ✅ 保证返回的数据是当前页面所需的数据
        setResults(json);
      }
    });
    return () => {
      ignore = true;
    };
  }, [query, page]);

  function handleNextPageClick() {
    setPage(page + 1);
  }
  // ...
}
```

竞态问题并不是数据请求现存的唯一问题，还有比如怎么缓存返回数据（以便用户可以单击返回并立即查看上一页内容而不是白屏）、如何在服务器上获取数据（以便初始服务器呈现的 HTML 包含内容而不是白屏）、如何避免瀑布问题（需要获取数据的子组件就不需要等待每个父组件完成数据获取）。这些问题并不是 React 特有的，而是所有 UI 库的通病，所以可以使用一些推荐的数据请求方案来规避这些问题。

## 参考资料

- [you-might-not-need-an-effect](https://beta.reactjs.org/learn/you-might-not-need-an-effect)
- [官方答：在React18中请求数据的正确姿势（其他框架也适用）](https://segmentfault.com/a/1190000042059978?utm_source=sf-backlinks)
