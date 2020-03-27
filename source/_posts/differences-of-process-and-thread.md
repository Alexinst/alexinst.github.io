---
title: differences-of-process-and-thread
date: 2020-03-27 11:24:39
tags:
	- OS
	- process
	- thread
categories:
	- OS
---



**简述：**进程和线程是一对朦朦胧胧的概念，每次面试提问二者区别，都感觉自己没全说对。

<!-- more -->

<br />



# 本质描述

**进程（Process）和线程（Thread）都是一个时间段的描述，是 CPU 工作时间段的描述。**

- 进程是 程序执行时间总和 = CPU 加载执行环境 -> CPU 执行程序 -> CPU 保存执行环境。

- 线程是 程序模块执行时间总和 = CPU 加载执行环境（共享进程的）-> CPU 执行程序模块-> CPU 保存执行环境（共享进程的）

  

进程和线程都是描述 CPU 工作的时间段，线程是更细小的时间段。

<br />



# 具体区别

- **两者关系**：一个进程可以包含多个线程，线程在进程下运行。
- **同侪关系**：
  - 进程间不会相互影响，一个线程崩溃将导致整个进程挂掉。
  - 进程之间是一种树的层级关系，P0 分出 P1 等等。而线程是一种平级的关系。
- **数据共享**：
  - 进程有自己的内存，通过分页将虚拟地址空间映射到物理地址空间来存储数据。不同进程间**数据共享很复杂**，需要借助进程间通信机制（如管道，消息队列，`Socket`，共享内存和信号量等）
  - 同一进程下，不同线程间数据**共享地址空间**（如段、数据段、用户 ID 和组 ID、文件描述符表、当前工作目录等）。
- **资源消耗**：进程要比线程消耗更多的计算机资源。
- **上下文切换**：对于操作系统内核而言，进程的上下文切换（`context switch`）时间消耗比线程的上下文切换更长（heavier）。

<br />




# 参考

- [线程和进程的区别是什么？ - ZhiHu](https://www.zhihu.com/question/25532384/answer/81152571)
- [What is the difference between a process and a thread? - Quora](https://www.quora.com/What-is-the-difference-between-a-process-and-a-thread#)