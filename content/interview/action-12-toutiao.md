---
title: "twelfth"
date: 2021-06-22T23:22:17+08:00
lastmod: 2021-06-22T23:22:17+08:00
draft: false
toc: true
keywords: ["interview"]
description: "字节面试"
author: "xuyou"
---

## 一面

### 1. CSS: rem、em、px、vh、vw 区别

### 2. CSS: 盒模型

### 3. CSS: 怎么居中一个元素

### 4. CSS: 常用的函数有哪些

### 5. 事件机制

- react 事件机制

### 6. 浏览器缓存

### 7. 跨域

- 简单请求和非简单请求

### 8. ES6 新特性

### 9. Promise 有哪些静态方法

### 10. HTTP2 的新特性

### 11. 手写 `debounce` `throttle`

### 12. 手写 `Promise.all` `Promise.race`

### 13. 算法题

- [第 k 大的元素](https://leetcode-cn.com/problems/kth-largest-element-in-an-array/)

## 二面

### 1. `React.Suspense`

- 做了什么、如何实现

### 2. 前端异常分类

- 异步错误怎么捕获

### 3. 输入 URL 到页面展示发生了什么

- 四次挥手必须是客户端发起吗

### 4. 跨域

- 同源策略是什么
- JSONP 怎么判断是哪个请求返回的数据
- 跨域可以发送 cookie 吗
- 前端可以获取跨域发送过来的 cookie 吗

### 5. Hooks 里面 setState 和 useState 的区别

### 6. 看输出

```js
var a = 0;
(function () {
  console.log(a);
  var a = 1;
})();
```

```js
var a = 0;
(function () {
  console.log(a);
  let a = 1;
})();
```

```js
var a = 0;
function Parent() {
  this.a = 1;
  return this;
}
Parent.a = 2;

Parent.prototype.setA = function (value) {
  this.a = value;
  return this;
};

console.log(new Parent().a);
console.log(Parent().a);
console.log(new Parent().setA(4).a);
console.log(a);
```

### 7. 编程题

```
已知数组 a = [1, [2, [3, [4, null]]]]，实现数组 b = [4, [3, [2, [1, null]]]]，考虑 n 级嵌套的情况。
```
