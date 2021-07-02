---
title: "fourteenth"
date: 2021-06-25T16:07:00+08:00
lastmod: 2021-06-25T16:07:00+08:00
draft: false
toc: true
keywords: ["interview"]
description: "百度"
author: "xuyou"
---

## 一面

### 1. 上来就是一道算法题

- [最长不含重复字符的子字符串](https://leetcode-cn.com/problems/zui-chang-bu-han-zhong-fu-zi-fu-de-zi-zi-fu-chuan-lcof/)

这个刷过后来忘记了，写了半个小时。

### 2. 节流和防抖

- 区别是什么
- 写一个防抖

### 3. 问项目

- sketch 插件是怎么个机制
- e2e 测试用例怎么维护

## 二面

### 1. 项目相关

- sketch 怎么导出图片
- 文件上传怎么分片，怎么并发上传
- e2e 测试用例如何维护
- websocket 比起 http 的优缺点在哪里

### 2. 二叉树寻找 target 并且返回路径

### 3. 浮点数相加

### 4. 写一个 sum 函数

```js
function asyncAdd(a, b, callback) {
  setTimeout(function () {
    callback(a + b);
  }, 100);
}

/**
 * 请在此方法中调用asyncAdd方法, 完成数值计算
 * @param  {...any} rest 传入的参数
 */
async function sum(...rest) {}

sum(1, 2, 3, 4, 5, 6).then((res) => {
  // 请保证在调用sum方法之后, 返回结果21
  console.log(res);
});
```

## 三面

问项目。

一个思考题，问有向图一共多少种输出。
