---
title: "React Ref"
date: 2022-08-07T19:57:18+08:00
lastmod: 2022-08-07T19:57:18+08:00
draft: false
toc: false
keywords: ["daily"]
description: "新文档里的 React Refs"
tags: ["JavaScript", "React"]
author: "xuyou"
---


### 怎么用 ref callback 获取一个列表的 DOM 节点

当我们需要获取一个列表的 DOM 节点，但不知道列表长度时，像下面这样写 ref 显然是不行的：

```jsx
<ul>
  {items.map((item) => {
    // Doesn't work!
    const ref = useRef(null);
    return <li ref={ref} />;
  })}
</ul>
```

这样写违反了 **Hooks 必须在组件顶部被调用** 的规则，`useRef` 不能在条件语句和循环中被调用。`map()` 自然也不可以。

一种方法是获取到父节点，然后使用操作 DOM 的方法例如 `querySelectorAll` 来获取到列表节点，但这样很不稳定，因为会受到子节点个数或 DOM 结构变化的影响。 

另一种解决方案就是给 ref 传递一个函数，即 ref callback。

```jsx
function getMap() {
  if (!itemsRef.current) {
    // 初始化
    itemsRef.current = new Map();
  }
  return itemsRef.current;
}

return (
  <ul>
   {catList.map(cat => 
      (<li 
        key={cat.id}
        ref={node => {
          const map = getMap();
          if (node) {
            // 节点添加到 map 中
            map.set(cat.id, node);
          } else {
            // 不存在就移除
            map.delete(cat.id);
          }
        }}
      >{cat.name}</li>)
    )}
</ul>
)
```

这样 ref 中存储的是一个 `<Id, Node>` 的 `Map` 映射。这样就可以通过 id 的 `Map` 映射来获取到相应节点~

### 获取另一个组件的 DOM 节点

当我们为一个原生节点设置 ref 时，React 会将节点挂在 `current` 属性上。但是当我们想在封装好的组件中使用 `ref`，就会遇到这样的报错：

![error-1](../../static-img/53-react-ref/ref-error.png)

这是因为 React 不允许组件获取另一个组件的 DOM，因为这样的做法会让我们的代码更难以预测与维护。

但是 React 也提供了相应的 `forwardRef` 允许我们获取到 DOM：

```jsx
const MyInput = forwardRef((props, ref) => {
  return <input {...props} ref={ref} />;
});
```

上面的 ref 转发会暴露整个 DOM 节点出去，这样外层就可以修改 DOM，例如它的样式。为了防止非预期的 DOM 操作，React 还提供了一个 `useImperativeHandle` Hook，用于暴露特定的 ref 操作方法，如下：

```js
const MyInput = forwardRef((props, ref) => {
  const realInputRef = useRef(null);
  useImperativeHandle(ref, () => ({
    // 只暴露 focus 方法出去
    focus() {
      realInputRef.current.focus();
    },
  }));
  return <input {...props} ref={realInputRef} />;
});
```

这样尽管 `realInputRef` 挂载了内部 DOM 元素，但是暴露出去的 `ref` 通过 `useImperativeHandle` 限制了操作 DOM 的方法，有效的增加了外部代码的可预期性。

### 为什么要限制操作 DOM

如果只是页面滚动或 focus，其实并不会有任何问题，但是文档中提供了一个例子，当同时使用 React state 和移除 DOM 的原生 API 时，就会产生如下的报错：

![error-2](../../static-img/53-react-ref/ref-error-2.png)

将原生的 DOM 操作和 React 控制的部分分开，也可以避免这样的问题~~但是自然是推荐限制 ref 使用来规范我们的代码。

> 官网最后还有一个 [使用 flushSync 同步更新 state 的例子](https://beta.reactjs.org/learn/manipulating-the-dom-with-refs#when-react-attaches-the-refs)，也是在表达 React state 和 DOM 操作存在的不符合预期的地方，值得注意一下。
