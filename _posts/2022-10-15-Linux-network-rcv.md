---
layout: post
title:  "Linux网络-接收数据包"
subtitle:   "Linux network - receive packets"
date:   2021-10-15 12:00:00 +10:00
author: "Steven Lu @slupro"
categories: [学习笔记]  # up to 2.
tags: [Linux, network]  # TAG names should always be lowercase, 0 to infinity.
---

> 基于Linux kernel 6.0

# Linux内核收包总览

```mermaid
sequenceDiagram
  participant Network
  participant NIC
  participant CPU
  participant Kernel
  participant User
  Network->>NIC: 数据帧从外部网络到达网卡
  NIC->>Kernel: 网卡把帧 DMA 到内存RingBuffer
  NIC->>CPU: 网卡硬中断通知 CPU
  CPU->>Kernel: CPU 响应硬中断，简单处理后发出软中断
  Kernel-->>Kernel: ksoftirqd线程处理软中断，调用网卡驱动注册的poll函数开始收包
  Note right of Kernel: 帧被从RingBuffer上取下来保存为一个skb
  Kernel->>User: 协议层开始处理帧，处理完后被放到socket的接收队列中
```

# Linux启动

* kernel/softirq.c 创建 ksoftirqd
并不直接以函数调用的方式创建，通过 early_initcall(spawn_ksoftirqd)，将spawn_ksoftirqd加入vmlinux文件的.init段，在程序启动时，即被调用。

* include/linux/init.h 中的 early_initcall(fn)

init.h中也有其他的一些 *_initcall(fn)，early_initcall(fn)在初始化SMP前就被运行，只用于build-in的代码，不能用于modules。

spawn_ksoftirqd 函数被加入vmlinux的.init段后，最终会被do_initcalls()调用。

```
start_kernel # init/main.c
	-> rest_init() # init/main.c
		-> kernel_thread(kernel_init, NULL, CLONE_FS | CLONE_FILES); # init/main.c
			-> kernel_init()
				-> kernel_init_freeable();
					-> do_basic_setup();
						-> do_initcalls();
```

