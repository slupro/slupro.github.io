---
layout: post
title:  "Linux网络-接收数据包"
subtitle:   "Linux network - receive packets"
date:   2021-10-15 12:00:00 +10:00
author: "Steven Lu @slupro"
categories: [学习笔记]  # up to 2.
tags: [Linux, network]  # TAG names should always be lowercase, 0 to infinity.
---

# Linux内核收包总览

```flow
  start=>start: 数据帧从外部网络到达网卡
  inf1=>operation: 网卡把帧 DMA 到内存
  inf2=>operation: 网卡硬中断通知 CPU
  inf3=>operation: CPU 响应硬中断，简单处理后发出软中断
  inf4=>operation: ksoftirqd线程处理软中断，调用网卡驱动注册的poll函数开始收包
  inf5=>operation: (kernal)帧被从RingBuffer上取下来保存为一个skb
  inf6=>operation: (user)协议层开始处理帧，处理完后被放到socket的接收队列中
  end=>end: 处理结束
  start->inf1->inf2->inf3->inf4->inf5->inf6->end
```

