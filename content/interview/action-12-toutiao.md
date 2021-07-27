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

## 三面

三面面试官会要求把所有的事情都说的很清楚。并且会追问，`还有呢，还有呢`。问的技术点列一下：

### 1. 以下 Promise 输出什么

```js
new Promise((resolve, reject) => {
  reject(1);
})
  .catch(() => {
    console.log(2);
  })
  .then(
    () => console.log(3),
    () => console.log(4)
  );
```

我说是 `2 3` 之后他问了很多次你确认吗，又问为什么输出 `2 3`，我说 `catch` 完 `catch` 的这个 `promise` 就变为 `resolve` 了，他说你确定是你说的这个原因吗...那不然呢还能是我蒙的吗。

### 2. webpack loader 有哪些用法

- 重点竟然在用法，也是没想到

### 3. react fiber 何时中断更新

- 一开始我说有高优先级任务插入的时候会中断，他说这些都是文档上能看到的东西，具体呢
- 怎么做到中断和插入高优任务

### 4. react 新特性

- 以及相应的原理

### 5. 滚动加载实现方式

- 我说了计算当前列表滚动距离到底部的位置差，他问还有呢，我说针对可以定高的列表 item 可以通过 index 来算，他说这个不行，不能定高
- 然后我就不知道了

### 6. React 有哪几种路由

- 原理分别是什么

### 7. 一个比较简单的编程题

一个字符串和数字的序列怎么倒序提取所有的数字。

一开始说用正则，但是确实不会用正则提取，就写了个循环。

面试官就说，你不是说用正则吗，我就实话说了不会提取。

> 这次面试把不善于表达发挥到了极致，在连环追问下面会很紧张，很多东西都想不起来，=。=。

---

三面挂掉了
