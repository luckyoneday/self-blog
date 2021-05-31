---
title: "fourth"
date: 2021-05-30T22:32:27+08:00
lastmod: 2021-05-30T22:32:27+08:00
draft: false
toc: true
keywords: ["interview"]
description: ""
author: "xuyou"
---

### 1. 数据类型、怎么判断

### 2. 浏览器缓存

### 3. `Connection: Keep-Alive`

### 4. 多路复用

### 5. HTTPS 安全在哪里

### 6. git 工作流，怎么回滚

### 7. 介绍服务端渲染

### 8. ts 泛型约束

### 9. ts 怎么表示 Promise 的返回值

### 10. 看输出

```js
console.log(1);
const promise = new Promise((resolve) => {
  console.log(2);
  resolve();
  console.log(3);
});

async function func() {
  console.log(4);
  await promise;
  console.log(5);
}

func();

promise.then(() => {
  console.log(6);
});
setTimeout(() => {
  console.log(7);
}, 0);

console.log(8);
```

### 11. 编程题

[有效的括号](https://leetcode-cn.com/problems/valid-parentheses/)
