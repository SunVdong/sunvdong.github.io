---
title: 'Process Thread Coroutine and Goroutine'
date: '2025-04-22T16:47:07+08:00'
draft: true
author: vdong
description: ''
categories:
  - 技术
tags:
  - golang
  - goroutine
typora-root-url: ..\..\..\static
---

## process

进程可以理解为运行中的程序。

操作系统可以同时运行多个程序，即多个进程同时运行。其手段是 **时分共享** CPU技术。让一个进程只运行一个时间片，然后切换到其他进程。如下：

| 时间 | Process0 | Process1 |               注               |
| :--: | :------: | :------: | :----------------------------: |
|  1   |   运行   |   就绪   |                                |
|  2   |   运行   |   就绪   |                                |
|  3   |   运行   |   就绪   |       Process0 发起 I/O        |
|  4   |   阻塞   |   运行   | Process0 被阻塞，Process1 运行 |
|  5   |   阻塞   |   运行   |                                |
|  6   |   就绪   |   运行   |           I/O 完成了           |
|  7   |   就绪   |   运行   |       Process1 运行完成        |
|  8   |   运行   |    -     |                                |
|  9   |   运行   |    -     |       Process0 运行完成        |

> ps1：进程有三个状态，运行 running、就绪 ready、阻塞 blocked。
>
> ps2:  Process0 发起 I/O 被阻塞，OS 发现 Process0 不使用 CPU 切换到 Process1 运行。I/O 完成，Process0 切换到就绪状态。最后，Process1 运行完成，Process0 运行，直到结束。

其中 Process0 和 Process1 的调度是操作系统 内核 完成的。其目的是保持 CPU 繁忙提高资源利用率。

操作系统也可以简单抽象进程，那它是如何获得 CPU 的控制权，来做进程切换呢？答案是，可以显式的 yield 一个系统调用，更多的是通过 **时钟中断**（通过硬件设备和预先配置的预处理程序实现） 。

前提条件：操作系统在管理资源时，会限制进程的做一些操作（比如获取更多的系统内存或CPU等），即用户模式和内核模式，用户模式切换的内核模式时，需要执行特殊的陷阱（trap）指令。



