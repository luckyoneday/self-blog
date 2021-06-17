---
title: "得物"
date: 2021-06-16T11:45:05+08:00
lastmod: 2021-06-16T11:45:05+08:00
draft: false
toc: true
keywords: ["interview"]
description: "得物"
author: "xuyou"
---

## 一面

### 1. 原型链

### 2. 怎么比较两个对象是否相等

函数怎么比较是否相等

### 3. ES6 的语法

### 4. Set WeakSet

`WeakSet` 何时会被清除

### 5. CSS 吸顶

### 6. CSS 复合层

### 7. React 常用的方法

### 8. useMemo 和 useCallback 的区别

### 8. 会不会重新渲染

```js
function B() {
  console.log("render B");
  return (
    <div>
      <h1>Hello B</h1>
    </div>
  );
}

function A() {
  const [num, setNum] = useState(0);
  const [count, setCount] = useState(0);

  console.log("render  A");

  const handleClick = () => {
    // 1.
    // setNum(num + 1);
    // setCount(count + 1);
    // ------------------------
    // 2.
    // setNum(num);
    // setCount(count);
    // ------------------------
    // 3.
    // setNum(num + 1);
    // setCount(count);
    // ------------------------
    // 4.
    // setNum(num);
    // setCount(count + 1);
  };

  return (
    <div>
      <h1 style={{ backgroundColor: "#eee" }} onClick={handleClick}>
        Hello A
      </h1>
      <B count={count} />
    </div>
  );
}

export default function App() {
  return (
    <div className="App">
      <A />
    </div>
  );
}
```

## 二面

需要现场，不想去了
