---
title: "TCP"
date: 2021-05-31T12:38:36+08:00
lastmod: 2021-05-31T12:38:36+08:00
draft: false
toc: true
keywords: ["daily"]
description: "TCP"
tags: ["HTTP"]
author: "youting"
---

TCP 协议是传输层协议，实现的是 **端口到端口** 的通信。

## TCP 如何保证传输可靠

TCP 头部会存一个带有该发送片段的序号，标识该片段在整个文本流的位置。在接收方接收到正确的、符合次序的片段之后，会向发送方发送 ACK 回复，表示接下来期待接收的序号。这样的过程一旦一端发送异常，就会导致资源不完整，所以需要重传机制。

{{% admonition info "超时重传" %}}

如果发送方在等待一定时间之后，还是没有收到 ACK 回复，发送方会 **重复发送** 那个出现异常的片段，直到收到该片段对应的 ACK 回复(回复号为 L+1 的 ACK)。

- RTT (Round-Trip Time 往返时延，数据从发送方到接收方的往返时间)
- RTO (Retransmission Timeout 超时重传时间)

超时重传可能存在的问题是超时周期可能较长。

{{% /admonition %}}

{{% admonition info "快速重传" %}}

当收到三个相同的 ACK 报文时，触发重传。

可能存在的问题是重传一个还是重传之后的所有。

{{% /admonition %}}

{{% admonition info "SACK" %}}

TCP 头部加一个叫 SACK 的字段，将缓存的地图发送给发送方，这样发送方就可以知道哪些数据收到了，哪些数据没收到，知道了这些信息，就可以只重传丢失的数据。

{{% /admonition %}}

{{% admonition info "D-SACK" %}}

使用 SACK 来告诉发送方有哪些数据被重复接收了。

- 可以让发送方知道，是发出去的包丢了，还是接收方回应的 ACK 包丢了;
- 可以知道是不是发送方的数据包被网络延迟了;

{{% /admonition %}}

## TCP 如何使用滑窗

发送方同时发送多个片段，对于提前到达的乱序片段在缓存中先存放着。之前的片段接收到后，滑窗向后移动。  
接收窗口的大小 **约等于** 发送窗口的大小。

## TCP 流量控制

如果一直发送数据，接收方处理不过来，就会触发重传机制，随后造成流量浪费，所以 TCP 会根据流量情况自动改变发送的数据量，即 TCP 流量控制。

当接收方接收窗口大小降为 0，发送方会隔一段时间进行 **窗口探测**。(发送方向接收方发送包含 1 byte 文本流的 TCP 片段，并等待 ACK 回复，该 ACK 回复包含有 window size).如果探测结果显示窗口依然为 0，发送方会等待更长的时间，然后再次进行窗口探测，直到 TCP 传输恢复。

## TCP 拥塞机制

如果网络出现拥塞，大量数据包丢失就会触发重传，可能会导致更大的延迟和丢包，所以 TCP 提供了拥塞机制。

> TCP 如何检测到拥塞？
>
> 只要发送方认为 TCP 丢失，则认为网络出现了拥塞。

{{% admonition info "慢启动" %}}

- 初始化拥塞窗口 (cwnd) 大小是 1。
- 发送方每收到一个 ACK，拥塞窗口 (cwnd) 大小就增加 1。
- 慢启动窗口呈指数性增长。

当 cwnd < ssthresh (慢启动门限) 时，使用慢启动算法。  
当 cwnd >= ssthresh 时，就会使用拥塞避免算法。

{{% /admonition %}}

{{% admonition info "拥塞避免" %}}

- 每收到一个 ACK 回复，cwnd 大小增加 1/cwnd，呈线性增长。
- 当触发了重传，就进入了拥塞发生阶段。

{{% /admonition %}}

{{% admonition info "拥塞发生" %}}

- ssthresh 设为 cwnd/2 。
- cwnd 重置为 1 。
- 重新进入了慢启动阶段。

{{% /admonition %}}

{{% admonition info "快速恢复" %}}

若重传时还可以收到三个 ACK 回复，TCP 会认为拥塞情况并不严重，会将

- cwnd 设为 cwnd/2
- ssthresh 设为 cwnd

进入快速恢复阶段。然后将

- cwnd 设为 ssthresh+3
- 重传丢失的数据包
- 收到新的 ACK 回复之后，可以重新进入之前的拥塞避免阶段

{{% /admonition %}}

![img](https://cdn.jsdelivr.net/gh/xiaolincoder/ImageHost2/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/TCP-%E5%8F%AF%E9%9D%A0%E7%89%B9%E6%80%A7/31.png)
（图片来源于参考文章）

## 参考资料

- [协议森林](https://www.cnblogs.com/vamei/archive/2012/12/05/2802811.html)
- [30 张图解： TCP 重传、滑动窗口、流量控制、拥塞控制](https://www.cnblogs.com/xiaolincoding/p/12732052.html)
