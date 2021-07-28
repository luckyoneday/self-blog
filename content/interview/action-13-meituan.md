---
title: "thirteenth"
date: 2021-06-25T14:04:00+08:00
lastmod: 2021-06-25T14:04:00+08:00
draft: false
toc: true
keywords: ["interview"]
description: "美团"
author: "xuyou"
---

## 一面

### 1. 问项目

- sketch 插件如何开发
- 测试项目中是什么角色

### 2. 讲一下 js 中的 this

看输出

```js
var test = {
  a: 40,
  init: () => {
    console.log(this.a);
    function go() {
      console.log(this.a);
    }
    go.prototype.a = 50;
    return go;
  },
};
var init = function () {};
var p = test.init();
p();
```

- 箭头函数改成 function 会怎么样
- 如何输出 50

### 3. 讲一下事件循环

看输出

```js
const promise = new Promise((resolve, reject) => {
  console.log(1);

  resolve(5);

  console.log(2);
}).then((val) => {
  console.log(val);
});

promise.then(() => {
  console.log(3);
});

console.log(4);

setTimeout(function () {
  console.log(6);
});
```

- 事件循环是什么来控制的
- 浏览器什么时候渲染呢

### 4. 手写 Promise.all

### 5. 同源策略

- 怎么带 cookie
- 两个域名怎么共享 cookie
- `sameSite` 作用

### 6. 打包工具

- 如何优化打包速度
- commonJS 和 ESM 的区别
- 还知道别的打包工具吗，rollup 了解吗，vite 呢

## 二面

全程问项目。业务也问得很细，包括业务背景，有没有做负责人，怎么排期怎么把握业务进度，怎么事后反思总结等。

## 三面

全程问项目。问项目背景、技术选型、中间遇到的问题、印象深刻的技术点、达到的效果、如何衡量效果，如果继续做下一步会朝哪些方面改进等。

---

offer ✅  
L7
