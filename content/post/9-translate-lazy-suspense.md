---
title: "React 18 中的 Suspense SSR 架构"
date: 2022-03-16T20:54:44+08:00
lastmod: 2022-03-16T20:54:44+08:00
draft: false
keywords: ["SSR"]
description: "是一篇翻译 -- React 18 中的 Suspense SSR 架构"
tags: ["SSR", "React"]
author: "xuyou"
---

> 原文标题: New Suspense SSR Architecture in React 18
>
> 原文链接: https://github.com/reactwg/react-18/discussions/37

<!--more-->

React 18 将包括对服务端渲染（SSR）性能的架构改进。最重要的新的 API 是 `renderToPipeableStream`。可以在[这里](https://github.com/reactwg/react-18/discussions/22)看到相关的 API 介绍。主要涉及到的之前的 API 是 `<Suspense>`。本篇是对新架构设计以及它解决的问题的介绍。

## 太长不看版

服务端渲染（SSR）即在服务端用 React 组件生成 HTML，并且将 HTML 直接返回给用户。SSR 允许用户在 JavaScript 资源加载和执行之前看到页面内容。

SSR 通常有以下几步：

1. 在服务端，获取 app 所需要的数据；
2. 在服务端，生成 HTML 并返回；
3. 在客户端，加载应用所需 JavaScript 代码；
4. 在客户端，把服务端生成的 HTML 与 JavaScript 逻辑结合（即 “hydration”）。

重点在于 **每一步需要在上一步完成之后才可以开始**。所以当某些部分比剩余部分逻辑慢的时候，就会影响整个应用加载的效率。

React 18 允许我们使用 `<Suspense>` **将应用分成独立的单元** ，这些单元可以单独完成这些步骤而且不会影响到应用的其他部分。因此用户可以快速看到应用的内容并且与之交互。即逻辑最慢的部分不会“拖慢”速度快的部分。这里有个[demo](https://codesandbox.io/s/kind-sammet-j56ro?file=/src/App.js)。

## 什么是 SSR

当用户访问我们的应用时，作为开发者当然希望尽快呈现出可交互的页面：

![full page](../../static-img/9-translate-suspense/1.png)

使用绿色代表可交互的页面，即 JavaScript 事件处理程序也都添加结束。

这个页面在 JavaScript 代码加载结束之前，页面是不能响应用户交互的。这些代码包括 React 和业务代码。所以大多数时间都是在等待代码下载。

如果不使用 SSR，在 JavaScript 代码加载的时候用户只能看到一个空白页面：

![blank page](../../static-img/9-translate-suspense/2.png)

因为空白页面交互不友好所以推荐使用 SSR。但是 SSR 也存在一个问题：无法马上可交互，除非是内置链接或者表单处理项。然而，SSR 可以让用户在 JavaScript 加载的时候看到页面：

![non-interactive page](../../static-img/9-translate-suspense/3.png)

这里使用灰色代替无法交互的部分。这段时间 JavaScript 代码还没有加载完，所以点击按钮并不会有任何响应。但对于内容繁重的网站 SSR 非常有用，因为它可以让网络连接较差的用户在加载 JavaScript 的同时开始阅读或查看内容。

当 React 和应用代码加载结束后，React 会将事件处理程序添加到 HTML 上，React 会在内存中渲染组件树，但不会为它生成 DOM 节点，而是将所有逻辑附加到现有的 HTML 中。

**渲染组件并且添加事件的过程被称为“hydration”**。因为这个过程像是在用交互性和事件处理程序的“水”注入到“干燥”的 HTML 中。在注水之后，我们的页面就会响应事件和状态更新，就像下图：

![full page](../../static-img/9-translate-suspense/1.png)

SSR 不会使应用程序更快地可交互，它只是尽快展示出应用无法交互的版本，所以用户可以在 JS 代码加载的时候看到静态页面。这个技巧对于网络连接不佳的用户来说体验有很大的不同，并提高了整体的感知性能。由于更容易索引和更快的速度，它还可以帮助我们进行搜索引擎排名。

> 提示：不要混淆 SSR 和 Server Components。Server Components 是一个实验中的特性，大概率不会成为初始化 React 18 中的一部分，可以在[这里](https://reactjs.org/blog/2020/12/21/data-fetching-with-react-server-components.html)看到更多关于 Server Components 的内容。 Server Components 是 SSR 的补充，将成为推荐的数据获取方法的一部分，但这篇文章不涉及到相关知识。

## 当前的 SSR 存在哪些问题

### 需要在展示页面之前获取所有数据

目前我们了解到 SSR 的一个问题是不允许组件“等待数据”。在渲染页面之前，需要先获取到所有的数据。例如我们想渲染一篇有评论的博文，评论数据自然是很重要的，所以我们想要在服务端的 HTML 返回中就包括评论组件。但是如果数据库或 API 层较慢，我们就需要从中做选择。可以选择服务端生成的 HTML 中不包括评论组件，那用户就只能在 js 资源加载后看到评论信息。或者就选择服务端生成的 HTML 中包括评论组件，页面的导航栏、侧边栏就都需要等到评论数据返回才能渲染。

### 注水需要等待资源加载完成

JavaScript 代码加载结束后，React 会开始给 HTML 绑定事件来使页面可交互。
