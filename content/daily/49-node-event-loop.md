---
title: "Node 事件循环"
date: 2022-02-12T13:07:37+08:00
lastmod: 2022-02-12T13:07:37+08:00
draft: true
toc: false
keywords: ["daily"]
description: "Node 事件循环"
tags: ["Node"]
author: "xuyou"
---

Node 的事件循环是处理高吞吐场景的核心，是即使单线程但是依然可以后台进行多数量处理的基础。

## 事件驱动编程

了解事件循环的基础是要先理解事件驱动编程。事件驱动编程是通过事件或状态变化来决定应用程序流控制。一般的实现是有一个中心机制监听事件并在检测到事件后调用回调函数。对熟悉客户端 JavaScript 的开发者来说，类似 `.on*` 回调，比如 `element.onclick()`，是与 DOM 元素结合使用来传达用户交互。这种模式在某个单个项目触发很多可能的事件时很有效。Node 中以 `EventEmitter` 来实现这种模式，并且位于 `Server`、`Socket` 和 `‘http’` 等。

一个常见的误解是 `EventEmitter` 本身在某种程度上是异步的，但这是不正确的。下面是一个简单的代码片段：

```js
function MyEmitter() {
  EventEmitter.call(this);
}
util.inherits(MyEmitter, EventEmitter);

MyEmitter.prototype.doStuff = function doStuff() {
  console.log('before')
  emitter.emit('fire')
  console.log('after')}
};

var me = new MyEmitter();
me.on('fire', function() {
  console.log('emit fired');
});

me.doStuff();
// 输出:
// before
// emit fired
// after
```

`EventEmitter` 看起来像是异步，因为它经常用于表示异步操作的完成，但 `EventEmitter` API 是同步的,可以异步调用 `emit` 函数，但所有侦听器函数都将按照添加的顺序同步执行，然后任何执行都可以在调用 `emit` 之后的语句中继续执行。

## 事件循环机制

Nodejs 依赖多个库，其中之一就是 libuv，负责处理异步事件的排队和触发。Node 会尽可能多地利用操作系统内核已有的功能，类似发出请求、保持连接等都被委托给系统并由系统处理。

## 参考链接

- [Understanding the Node.js Event Loop](https://nodesource.com/blog/understanding-the-nodejs-event-loop/)
- [Introduction to Event Loop Utilization in Node.js](https://nodesource.com/blog/event-loop-utilization-nodejs)
