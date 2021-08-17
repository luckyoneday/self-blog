---
title: "React Hooks 持续踩坑"
date: 2021-08-06T20:37:40+08:00
lastmod: 2021-08-06T20:37:40+08:00
draft: true
keywords: ["React"]
description: "React Hooks 的一些注意事项"
tags: ["React"]
author: "youting"
---

汇总一些关于 hooks 使用时的注意事项~

<!--more-->

## 怎么理解 rules of Hooks

## React 函数中怎么使用防抖和节流

## useEffect 的返回值

如果在 useEffect 中写成如下形式，会发现控制台有警告信息：`useEffect function must return a cleanup function or nothing. Promises and useEffect(async () => ...) are not supported, but you can call an async function inside an effect.`

```js
useEffect(async () => {
  const result = await axios("xxx");
}, []);
```

useEffect 的返回值可以是清理函数或者不返回，不可以是 Promise，所以可以这样写：

```js
useEffect(() => {
  const fetchData = async () => {
    const result = await axios("xxx");
  };

  fetchData();
}, []);
```

## 参考链接

- [Rules of Hooks](https://reactjs.org/docs/hooks-rules.html)
- [React hooks: not magic, just arrays](https://medium.com/@ryardley/react-hooks-not-magic-just-arrays-cd4f1857236e)
- [Making setInterval Declarative with React Hooks](https://overreacted.io/making-setinterval-declarative-with-react-hooks/)
- [stackOverflow](https://stackoverflow.com/questions/54666401/how-to-use-throttle-or-debounce-with-react-hook)
