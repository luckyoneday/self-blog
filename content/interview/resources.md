---
title: "资源汇总"
date: 2021-05-07T17:58:27+08:00
lastmod: 2021-05-07T17:58:27+08:00
draft: false
toc: true
keywords: ["interview"]
description: "一些资源"
author: "xuyou"
---

## 项目相关

### codecept

- [codeceptjs basic](https://codecept.io/basics/)
- [puppeteer](https://pptr.dev/)
- puppeteer-core 不包括 `PUPPETEER_*` 的环境变量，不下载 Chromium。

### electron

[electron 通信](../../post/5-electron-communication/)

### sketch

[sketch 通信](../../post/4-sketch-plugin/)

### drag & drop

- [react-beautiful-dnd](https://github.com/atlassian/react-beautiful-dnd)

- drag and drop

设置 `draggable = true`，`dragStart` 时设置 `event.dataTransfer.setData()`，`drop` 事件时通过 `event.dataTransfer.getData()` 获取自定义数据。拖放文件时可以通过 `event.dataTransfer.files` 拿到文件。

| 针对对象     | 事件名称    | 说明                                             |
| ------------ | ----------- | ------------------------------------------------ |
| 被拖动的元素 | `dragstart` | 在元素开始被拖动时候触发                         |
|              | `drag`      | 在元素被拖动中反复触发                           |
|              | `dragend`   | 在拖动操作完成时触发                             |
| 目的地对象   | `dragenter` | 当被拖动元素进入目的地元素所占据的屏幕空间时触发 |
|              | `dragover`  | 当被拖动元素在目的地元素内时触发                 |
|              | `dragleave` | 当被拖动元素没有放下就离开目的地元素时触发       |

- 缩放拖拽时 wheel、mousedown、mousemove、mouseup 事件，touchstart、touchmove、touchend 事件。

### canvas

为什么 img 换成 canvas 会有优化效果，img 绘制会影响 js 的执行，会造成操作上的不流畅。使用 canvas 会启用硬件加速。

`canvas.getContext('2d).drawImage(img, posX, posY, imgWidth, imgHeight)` 是位图，在高 dpi 设备上会模糊。
canvas 的画布大小，使用的是 canvas.width 和 canvas.height；要设置画布的实际渲染大小，使用的 style 属性或 CSS 设置的 width 和 height。

```html
<canvas width="640" height="800" style="width:320px; height:400px"></canvas>
```

绘制的时候使用 `canvas.scale(radio, radio)`。

## 一些参考资料

### 算法

- [HTTP 和数据结构与算法](https://hit-alibaba.github.io/interview/)
- [数据结构和算法一系列](http://www.dennisgo.cn/Articles/DataStructureAndAlgorithm/DP.html)
- [算法](https://leetcode-solution-leetcode-pp.gitbook.io/leetcode-solution/)
- 算法小抄：labuladong

### react

- [React 的秘密](https://github.com/neroneroffy/react-source-code-debug)
- [一文吃透 react 事件系统原理](https://juejin.cn/post/6955636911214067720)

- [react 技术揭秘](https://react.iamkasong.com/)
- [react 源码 - 掘金](https://juejin.cn/post/6859528127010471949)
- [人人都能读懂的 react 源码解析](https://xiaochen1024.com/article_item/600ac4384bf83f002edaf54a)
- [react-router](https://github.com/youngwind/blog/issues/109)

### typescript

- [细数 TS 中那些奇怪的符号](https://cloud.tencent.com/developer/article/1695444)

### AND

- [promise](https://juejin.cn/post/6850037281206566919)
- [webpack](https://mp.weixin.qq.com/s/TTIRDG15T3l5VDm8SrUZWg)
- [tapable](https://www.ahonn.me/blog/about-tapable-you-need-to-know-these)

### 纯面试题

- [前端面试之道](https://doc.xuwenliang.com/docs/frontend/2731)
- [字节面试题](https://juejin.cn/post/6866082181455249422)
- [手写题](https://juejin.cn/post/6855129007852093453)

## 最后的最后

<p style="font-size: 72px">坚强 !!! 坚强他环绕着我</p>
