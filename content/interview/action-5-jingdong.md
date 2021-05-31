---
title: "fifth"
date: 2021-05-25T11:47:45+08:00
lastmod: 2021-05-25T11:47:45+08:00
draft: true
toc: true
keywords: ["interview"]
description: "京东"
author: "xuyou"
---

## 一面

### 1. css 多行两栏布局

### 2. flex 还有哪些属性

### 3. 跨域

### 4. 关于 this 的指向问题

箭头函数不会改变指向啊，天。

### 5. 遍历的方式

### 6. `for...in`、`for...of` 遍历的区别

### 7. 一道变量提升的题，i 闭包

### 8. 说说 promise

### 9. 手写 ajax

### 10. 代码题

写一个 f，满足：

```js
console.log(f(12345678)); // '12,345,678'
```

`toLocaleString` 是数字的方法。

### 11. 手写

设计一个函数 `clickOnce`，对指定的 dom 对象绑定 `click` 事件，但要求无论点击多少次，只有第一次的点击才触发事件回调 `callback`

```js
clickOnce(document.getElementById("app"), () => console.log("click"));
```

### 12. 看输出

```js
let arr = [];
for (let i = 0; i < 10; i++) {
  arr.push((finish) => {
    console.log(i);

    return function () {
      // do something
      finish();
    };
  });
}
var func = arr.reduce((pre, cur) => cur(pre));

func();
```
