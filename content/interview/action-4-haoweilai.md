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

## 一面

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

## 二面

### 1. 介绍项目

- 问大文件怎么传输、数据压缩（只说了分片上传）
- 怎么计算用户在页面的停留时长（说了 location 监听路有变化、visibilitychange ）
- 直播推流（不了解）

### 2. tcp 拥塞机制

### 3. http 四次挥手

### 4. tcp、udp 的区别

### 5. 数组乱序

### 6. 编写代码

```
请编写一个函数
function forbidWordReplace(text, keyWords, replace)
接收三个参数, 将 `text` 中存在的 `keyWords` 内容, 替换为 `replace` 内容, 如:

text: '小刘同学是今年北京高考文科状元, 高考英语成绩排名第一, 成功被北大录取'
keyWords: ['状元','排名','北大']
replace :'\*'

期望结果:
小刘同学是今年北京高考文科**, 高考英语成绩**第一, 成功被 \*\*录取
```

## 三面

### 1. 首页加载白屏时间过长，有什么解决方法

### 2. 写一个函数

```
实现一个add方法, 使计算结果能够满足如下预期:
add(1)(2)(3)() === 6;
add(1, 2, 3)(4)() === 10;
add(1)(2)(3)(4)(5)() === 15;
```

### 3. 内存泄露场景，怎么解决

---

offer ✅
