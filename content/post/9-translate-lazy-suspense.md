---
title: "React 18 中的 Suspense SSR 架构"
date: 2022-03-16T20:54:44+08:00
lastmod: 2022-03-16T20:54:44+08:00
draft: true
keywords: ["SSR"]
description: "是一篇翻译 -- React 18 中的 Suspense SSR 架构"
tags: ["SSR", "React"]
author: "xuyou"
---

> 原文标题: New Suspense SSR Architecture in React 18
>
> 原文链接: https://github.com/reactwg/react-18/discussions/37

React 18 将包括对服务端渲染（SSR）性能的架构改进。最重要的新的 API 是 `renderToPipeableStream`。可以在[这里](https://github.com/reactwg/react-18/discussions/22)看到相关的 API 介绍。主要涉及到的之前的 API 是 `<Suspense>`。本篇是对新架构设计以及它解决的问题的介绍。

<!--more-->
