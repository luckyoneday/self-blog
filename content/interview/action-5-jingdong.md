---
title: "fifth"
date: 2021-05-31T15:47:45+08:00
lastmod: 2021-05-31T15:47:45+08:00
draft: false
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

## 二面

### 1. 实现一个旋转有序数组搜索

先从有序数组搜索开始写，然后写旋转有序数组搜索。

### 2. 数组 flat

### 3. js 对象扁平

### 4. 介绍项目、有什么难点

- sketch 通信
- canvas 怎么适配屏幕：可能是想让说使用 `window.devicePixelRatio` 放大再缩小一下。。。
- websocket 服务端怎么向客户端发消息

## 三面

### 自动化测试流程、项目收益

### 介绍近一年自己有成长的地方或者攻克的难点

- electron 通信、sketch 通信
- 服务端渲染

---

offer ✅
