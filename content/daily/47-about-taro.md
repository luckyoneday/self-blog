---
title: "Taro--新手上路"
date: 2021-08-04T20:57:35+08:00
lastmod: 2021-08-04T20:57:35+08:00
draft: false
toc: true
keywords: ["daily"]
description: "Taro--新手上路"
tags: ["JavaScript", "Taro"]
author: "xuyou"
---

最近的工作集中于 Taro 小程序的开发，可以记录一些踩坑的点。

## 如何自定义 TabBar

### 完全自定义

之前是想过要自己写自定义 tabbar 然后使用 `navigateTo` 进行页面跳转，但是会有从右向左划出的动画过渡效果，而且也会算在路由栈里面（小程序路由跳转最多能打开 10 个页面），所以考虑用官方提供的自定义方式。

### 配置 `app.config`

添加 `custom: true`，并且推荐填充 `list`，便于在不支持自定义 tabBar 的平台上（例如 H5）也会有占位。

```js
// app.config.js
// ...
tabBar: {
  custom: true, // 比较重要
  color: '#999',
  selectedColor: '#333',
  list: [
    // ...
  ]
}
```

### 创建自定义组件

在 `src` 下面创建 `custom-tab-bar` 文件夹，在里面写自己定义的 tabBar 组件。

不需要自己设置组件为 `position: fixed`。因为小程序会直接将组件挂在底部，且层级最高。这样就导致有 tabBar 的页面需要将底部的位置让出来，不然会发生覆盖。

Taro 限制不能使用 `navigateTo` 跳转到 tabBar 页面，因此只能使用 `switchTab` 方法跳转。但是 [ `switchTab` 不能在路由上带参数](http://taro-docs.jd.com/taro/docs/apis/route/switchTab/)，此时可以考虑派发 `event` 或者全局变量记录参数。

### 切换更新

当点击 tabBar 的某个 item 时，需要有相应的跳转操作，还需要通知 tabBar 更新选中态。tabBar 是每个页面都会挂载，所以需要在页面 `didShow` 的时候去更新当前选中态，小程序官方推荐使用 [ `getTabBar` 获取组件实例，并调用 `setData` 更新选中态](https://developers.weixin.qq.com/miniprogram/dev/framework/ability/custom-tabbar.html)，但这是原生小程序的写法。 Taro 有这样一个 [issue](https://github.com/NervJS/taro/issues/7302)，意思是可以在 `wx.onAppRoute` 的时候拿路由去更新选中态，但是我自己试有闪动，所以使用了 `eventCenter` 的方法来通知更新。

## 小程序登录

![登录流程](https://res.wx.qq.com/wxdoc/dist/assets/img/api-login.2fcc9f35.jpg)

## Taro 自定义导航栏

- 写自定义导航组件的时候，需要将组件结构一分为二：状态栏 + 标题栏
- 状态栏高度可通过 `Taro.getSystemInfoSync().statusBarHeight` 获取
- 标题栏高度：安卓：`48px`，iOS：`44px`
- 单位需要跟胶囊按钮一致，用 `px`
- 具体可见[如何完美适配刘海屏](https://juejin.cn/post/6844903810578513928)

## and last

- **都是引用本地的图片，为什么有的不展示？** `<Text>` 组件中不能放置组件，只能放置字符串。意味着如果 `<Image>` 组件放在 `<Text>` 组件内部，是显示不出来的。
- tabBar 底部让出安全距离： `padding-bottom: constant(safe-area-inset-bottom); padding-bottom: env(safe-area-inset-bottom)`
- 小程序使用 `background-image` 设置背景图需要使用网络地址，或者将图片转为 base64；想使用本地图片地址的话可以使用 `Image` 组件并将 `z-index` 置为 -1。
- 若是企业微信小程序模式下看不到自定义 tabBar，需要将调试基础库版本调到 2.8.3 或以上~

## 参考

- [官方文档--微信小程序登录](https://developers.weixin.qq.com/miniprogram/dev/framework/open-ability/login.html)
- [网页适配 iPhoneX，就是这么简单](https://jelly.jd.com/article/6006b1055b6c6a01506c87fd)
