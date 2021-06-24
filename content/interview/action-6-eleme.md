---
title: "sixth"
date: 2021-06-11T16:02:59+08:00
lastmod: 2021-06-11T16:02:59+08:00
draft: false
toc: true
keywords: ["interview"]
description: "饿了么"
author: "xuyou"
---

## 一面

### 1. 层次遍历

```js
const data = [
  {
    name: "a",
    children: [
      { name: "b", children: [{ name: "e" }] },
      { name: "c", children: [{ name: "f" }] },
      { name: "d", children: [{ name: "g" }] },
    ],
  },
  {
    name: "a2",
    children: [
      { name: "b2", children: [{ name: "e2" }] },
      { name: "c2", children: [{ name: "f2" }] },
      { name: "d2", children: [{ name: "g2" }] },
    ],
  },
];

console.log(getName2(data)); // a,a2,b,c,d,b2,c2,d2,e,f,g,e2,f2,g2
```

### 2. LRU

```js
// 题目：缓存机制
// 运用你所掌握的数据结构，设计和实现一个  LRU (最近最少使用) 缓存机制。它应该支持以下操作： 获取数据 get 和 写入数据 put 。

// 获取数据 get(key) - 如果密钥 (key) 存在于缓存中，则获取密钥的值（总是正数），否则返回 -1。
// 写入数据 put(key, value) - 如果密钥不存在，则写入其数据值。当缓存容量达到上限时，它应该在写入新数据之前删除最久未使用的数据值，从而为新的数据值留出空间。
// 示例:
LRUCache cache = new LRUCache( 2 /* 缓存容量 */ );

cache.put(1, 1);
cache.put(2, 2);
cache.get(1);       // 返回  1
cache.put(3, 3);    // 该操作会使得密钥 2 作废
cache.get(2);       // 返回 -1 (未找到)
cache.put(4, 4);    // 该操作会使得密钥 1 作废
cache.get(1);       // 返回 -1 (未找到)
cache.get(3);       // 返回  3
cache.get(4);       // 返回  4
```

### 3. 关于捕获错误

1. `try{} catch{}` 能否捕获到 promise 抛出的错误，为什么
2. 那要如何捕获错误： 说了`window.onerror`，React 的 `ErrorBoundary`
3. Hook 里面如何捕获错误

### 4. useCallback 和 useMemo 解决了什么问题

### 5. 问项目

- 服务端渲染原理、优缺点
- node 使用程度
- e2e 测试是什么流程，怎么维护

### 6. 开发过程中遇到技术问题且无法解决，怎么办

### 7. 遇到了估时不准确的情况，怎么解决

## 二面

### 1. node 怎么读取大文件

疯狂问 node 的问题

- node 内存管理
- node 上线出问题怎么排查

### 2. H5 和原生怎么通信

- H5 怎么访问还没有挂载好的方法

### 3. webpack loader 的作用

### 4. Hooks 的一些例子

- 怎么封装一个埋点的 hooks
- 怎么清除事件绑定

### 5. 前端白屏会因为什么，如何排查

### 6. 接口成功率突然降为 80%，会是因为什么

### 7. safe touch 怎么处理，有没有更高效的方式

- 面试官提示使用 ts 做默认值

### 8. HTTP 协商缓存

### 9. 跨域，为什么 JSONP 可以跨越

## 三面

### 1. 介绍项目

- sketch 插件提效体现在哪里
- 服务端渲染
- e2e 测试流程

### 2. React 技术体系

- 打包、 状态管理、 路由、 typescript、 eslint

### 3. 日常开发怎么提效

- 为什么建议直接复制粘贴

### 4. 怎么快速生成一个风格基本统一却又不太一样的站点

- 需要有自定义修改的时候怎么办

### 5. 三年工作中学到了什么
