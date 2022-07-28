---
title: "在使用 `memo()` 之前"
date: 2022-05-06T09:08:30+08:00
lastmod: 2022-05-06T09:08:30+08:00
draft: false
toc: false
keywords: ["daily"]
description: "是一篇翻译"
tags: ["React", "Translation"]
author: "xuyou"
---

> 标题：Before You memo()
>
> 原文链接：https://overreacted.io/before-you-memo/

关于 React 性能优化的文章很多啦。通常情况下，当某些状态更新较慢时，我们需要：

- 检查当前代码是否为生产环境版本（开发版本一般会比较慢，极端情况甚至会差一个数量级）。
- 检查是否把状态提的“太高了”，比如一个 `input` 的状态提到了全局 store 里。
- 使用 React DevTools Profiler 来获取触发了重复渲染的组件，将渲染昂贵的组件包裹在 `memo()` 中。

在这篇文章中，我们会学习到两种技巧，可以通过一些基础操作来提升性能。他们并不是用来替换 `memo` 和 `useMemo` 的，但是是很好用的方式。

### 一个人工减慢的例子

下面是一个拥有严重渲染性能问题的组件：

```jsx
import { useState } from "react";

export default function App() {
  let [color, setColor] = useState("red");
  return (
    <div>
      <input value={color} onChange={(e) => setColor(e.target.value)} />
      <p style={{ color }}>Hello, world!</p>
      <ExpensiveTree />
    </div>
  );
}

function ExpensiveTree() {
  let now = performance.now();
  while (performance.now() - now < 100) {
    // Artificial delay -- do nothing for 100ms
  }
  return <p>I am a very slow component tree.</p>;
}
```

（[点击这里试一下](https://codesandbox.io/s/frosty-glade-m33km?file=/src/App.js:23-513)）

这里的问题在于 `<App />` 组件中的 `color` 状态变化时，都会重新渲染 `<ExpensiveTree />` 组件。当然可以在外面[裹一个`memo`](https://codesandbox.io/s/amazing-shtern-61tu4?file=/src/App.js)了事，但是我们还有别的解决方案。

### 第一种方式：将状态下放

上面的例子其实只有一部分代码会依赖 `color` 状态变量：

```jsx {2, 5-6}
export default function App() {
  let [color, setColor] = useState("red");
  return (
    <div>
      <input value={color} onChange={(e) => setColor(e.target.value)} />
      <p style={{ color }}>Hello, world!</p>
      <ExpensiveTree />
    </div>
  );
}
```

可以把这部分逻辑抽出一个 `<Form>` 组件，并将状态变化写在这个组件中：

```jsx
export default function App() {
  return (
    <>
      <Form />
      <ExpensiveTree />
    </>
  );
}

function Form() {
  let [color, setColor] = useState("red");
  return (
    <>
      <input value={color} onChange={(e) => setColor(e.target.value)} />
      <p style={{ color }}>Hello, world!</p>
    </>
  );
}
```

（点击这里[尝试一下](https://codesandbox.io/s/billowing-wood-1tq2u?file=/src/App.js:64-380)）

这个时候如果 `color` 状态变化了，只会有 `Form` 组件重新渲染，问题就解决啦。

### 第二种方式：组件内容提升

但是当这个状态在父组件中，就不能避免组件渲染问题了。例如我们把 `color` 变量提升到父 `<div>` 组件中：

```jsx
export default function App() {
  let [color, setColor] = useState("red");
  return (
    <div style={{ color }}>
      <input value={color} onChange={(e) => setColor(e.target.value)} />
      <p>Hello, world!</p>
      <ExpensiveTree />
    </div>
  );
}
```

（[点击这里试一下](https://codesandbox.io/s/bold-dust-0jbg7?file=/src/App.js:58-313)）

这种情况我们不能将有 `color` 状态变量的部分单独抽一个组件出来，因为状态变量在父 `<div>` 中， 这次不能避免用 `memo` 了。

但是我们也是有法子的~

```jsx
export default function App() {
  return (
    <ColorPicker>
      <p>Hello, world!</p>
      <ExpensiveTree />
    </ColorPicker>
  );
}

function ColorPicker({ children }) {
  let [color, setColor] = useState("red");
  return (
    <div style={{ color }}>
      <input value={color} onChange={(e) => setColor(e.target.value)} />
      {children}
    </div>
  );
}
```

（[点击这里试一下](https://codesandbox.io/s/wonderful-banach-tyfr1?file=/src/App.js:58-423)）

我们把 `App` 组件分成了两个。依赖 `color` 状态变量的部分，移到了叫 `ColorPicker` 的组件。不依赖 `color` 的部分依然在`App` 组件中，并将 `ColorPicker` 组件作为 `children` 属性传入。当 `color` 变化时，`ColorPicker` 组件会重新渲染，但是它拿到的 `children` 属性还是相同的，所以 React 并不会重新渲染 `ExpensiveTree`。

## 更多的是

在我们使用 `memo` 或者 `useMemo` 做优化时，把不变的部分和变化的部分分开会很有用。

使用 `children` 属性来传递组件通常会使应用程序的数据流更容易追踪，并且可以减少树中传递的 `props` 数量。所以这个特性并不是为了性能优化设计的，所以提升性能只是锦上添花，并不是终极目标！

奇怪的是，这种模式在将来还会带来更多的性能好处。

举个例子，当 [服务器组件](https://reactjs.org/blog/2020/12/21/data-fetching-with-react-server-components.html) 稳定且可被采用时，我们的 `ColorPicker` 组件就可以从 [服务端](https://youtu.be/TQQPAU21ZUw?t=1314) 获取到它的 `children`。整个`<ExpensiveTree />` 组件或其部分都可以在服务器上运行，即使是顶层的 React 状态更新也会在客户端“跳过”这些部分。

这些都是 `memo` 做不到的事情。但是，这些方法是互补的。不要忽视 state 下移和内容提升！

如果这两种方式不够，那就使用 Profiler 然后用 `memo` 来写吧。

也可以看[这篇文章](https://kentcdodds.com/blog/optimize-react-re-renders)来了解相关内容。
