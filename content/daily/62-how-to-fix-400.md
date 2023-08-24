---
title: "如何修复 http code 400"
date: 2023-08-12T10:27:19+08:00
lastmod: 2023-08-12T10:27:19+08:00
draft: false
toc: false
keywords: ["HTTP"]
description: "如何修复400报错"
tags: ["HTTP"]
author: "xuyou"
---

## 什么情况会导致 400 错误

当我们访问一个网站的时候，客户端会向服务器发起请求，服务端响应请求并将页面信息返回。`400 Bad Request` 响应通常发生在服务器不理解该请求的时候，因此大多数是客户端错误导致，例如：

- 错误的 URL -- 请求 url 包含了不规范的字符例如 `{`，`}`，`[]`，`]`。或是包含了错误使用的 ASCII 字符。
- 文件大小超出限制 -- 某些网站会设置[文件限制](https://www.hostinger.com/tutorials/how-to-increase-the-maximum-file-upload-size-in-wordpress)，当上传超出限制的文件时会被视为错误的请求从而返回 400 错误码。
- 损坏的 cookie 或者缓存 -- 过期的 cookie 或者损坏的浏览器缓存可能会干扰请求并触发错误。这个错误通常发生在登录页。
- 错误的 DNS 缓存 --  如果网站迁移到了新的[域名](https://www.hostinger.com/domain-name-search)或者虚拟主机，我们操作系统中的 DNS 缓存数据可能会无效。

## 怎么修复 400 报错

推荐的第一个解决方式自然是刷新页面，如果不好用则可以根据上述可能存在的错误，按照以下步骤来排查：

- 仔细检查访问地址 -- 检查请求路径中是否存在不规范字符或是错误的 ASCII 码，可以使用 [这个工具](https://meyerweb.com/eric/tools/dencoder/)来验证路径是否正确。
- 通过关键字来寻找地址 -- 如果访问网络地址遇到了 400 错误，则可以使用搜索引擎的关键字搜索来获取相应的地址。可以使用 `site: host + keyword` 的格式可以获取相应地址，例如：
  ![img](https://www.hostinger.com/tutorials/wp-content/uploads/sites/2/2021/09/google-search.webp)
- 清除 cookie 和浏览器缓存 -- 浏览器缓存数据过大或是 cookie 过期都可能会导致 400。
- 关闭浏览器插件 -- 很多情况下插件会影响传输给服务端或用户 cookie。
- 检查文件大小 -- 当上传文件过大时，压缩一下。
- 刷新 DNS 缓存
  - windows：`ipconfig /flushdns`
  - macos：`sudo killall -HUP mDNSResponder`
  - chrome：访问 `chrome://net-internals/#dns`，点击清除缓存
- 联系网站开发者，上报这个错误。
- 最后的最后可以试一下重启电脑，重启可以解决 98% 的问题 😄

## 参考链接 

- [how-to-fix-400-bad-request-error](https://www.hostinger.com/tutorials/how-to-fix-400-bad-request-error)