---
title: "useEffect 使用指南"
date: 2021-07-29T21:03:31+08:00
lastmod: 2021-07-29T21:03:31+08:00
draft: true
keywords: ["React"]
description: "A Complete Guide to useEffect 翻译"
tags: ["React", "翻译"]
author: "youting"
---

> 原文标题: A Complete Guide to useEffect
>
> 原文链接: https://overreacted.io/a-complete-guide-to-useeffect/

<!--more-->

只有踩过一些坑之后才会发现，[Dan Abramov](https://mobile.twitter.com/dan_abramov) 写的文章是真的很优秀。之前理解的不深刻，所以还是需要重新阅读一下经典文章~

## 从几个问题开始：

{{% admonition "tip" "🤔 怎么使用 `useEffect` 代替 `componentDidMount` ？"  %}}

虽然可以使用 `useEffect(fn, [])` 作为替代，但是其实二者并不一样，`useEffect` 会「捕获」 `props` 和 `state`，所以在回调函数 `fn` 中，我们拿到的始终是初始的 `props` 和 `state`。如果想获取到最新的值，需要使用 ref 值来记录，当然也有比记录 ref 简单的方法。需要铭记 `useEffect` 和 `componentDidMount` 和其他声明周期函数的心智模型不一样，思考他们在什么地方相同可能会更困惑。"thinking in effects" 比生命周期函数更接近状态同步的概念。

{{% /admonition %}}

{{% admonition "tip" "🤔 怎么正确在 `useEffect` 中请求数据？什么是 `[]`？"  %}}

[这篇文章](https://www.robinwieruch.de/react-hooks-fetch-data)描述了怎么在 `useEffect` 中获取数据。推荐阅读！`[]` 表示在当前 effect 中没有使用任何 React 数据流中的值，所以只会调用一次。

{{% /admonition %}}

{{% admonition "tip" "🤔 需要将函数作为 `useEffect` 依赖项吗？具体如何操作呢？"  %}}

{{% /admonition %}}

{{% admonition "tip" "🤔 为什么有的时候会触发死循环"  %}}

{{% /admonition %}}

{{% admonition "tip" "🤔 为什么有时会在 `useEffect` 内部拿到旧的 `state` 或者 `props`"  %}}

{{% /admonition %}}
