---
title: Docker与虚拟机
date: 2020-08-07 15:54:51
tags: docker, 虚拟机
categories: 云计算
---

## 虚拟化技术

`在计算机技术中，虚拟化（技术）或虚拟技术（英语：Virtualization）是一种资源管理技术，是将计算机的各种实体资源（CPU、内存、磁盘空间、网络适配器等），予以抽象、转换后呈现出来并可供分割、组合为一个或多个电脑配置环境。`

<!--more-->


## 虚拟机:hypervisor虚拟化

OS作为硬件上第一层软件，认为自己拥有全部的硬件的访问和控制权，且自己是唯一的控制者。在这种情况下，如果两个OS共存，必然产生问题。OS主要负责管理的是CPU和内存，以及众多的IO设备。于是我们可以分别讨论。hypervisor是实现虚拟化的关键，它会以一个内核态的驱动存在。

### CPU虚拟化

一个CPU的全部状态其实就是所有寄存器的值，只要保证任何操作之后寄存寄的值在OS看来是正确的，guest OS就可以正常执行。hypervisor会为每个虚拟的CPU创建一个数据结构，模拟CPU的全部寄存器的值，在适当的时候跟踪并修改这些值。

### 内存虚拟化

hypervisor虚拟化内存的方法是创建一个shadow page table。正常的情况下，一个page table可以用来实现从虚拟内存到物理内存的翻译。在虚拟化的情况下，由于所谓的物理内存仍然是虚拟的，因此shadow page table就要做到：虚拟内存->虚拟的物理内存->真正的物理内存。

### I/O虚拟化

I/O虚拟化：背景知识：memory mapped I/O device。大多数的PCI设备都是直接将自己的某些控制寄存器映射到物理内存空间上，CPU访问这些控制寄存器的方法和访问内存相同。CPU通过修改和读取这些寄存器来操作I/O设备。虚拟化的方法很简单，没当hypervisor接到page fault，并发现实际上虚拟的物理内存地址对应的是一个I/O设备，hypervisor就用软件模拟这个设备的工作情况，并返回。比如当CPU想要写磁盘时，hypervisor就把相应的东西写到一个host OS的文件上，这个文件实际上就模拟了虚拟的磁盘。

## Docker:容器级虚拟化

容器技术是一种全新意义上的虚拟化技术，按分类或者实现方式分类，属于操作系统虚拟化的范畴，也就是在由操作系统提供虚拟化的支持，操作系统提供接口，能够让应用程序间可以互不干扰的独立运行，并且能够对其在运行中所使用的资源进行管理。

与虚拟机相比少了的hypervisor(虚拟机监控器)+GuestOs。大幅减少了应用程序运行带来的额外消耗。更准确的来说，所有在容器中的应用程序其实完全运行在了宿主操作系统中，与其他真实运行在其中的应用程序在指令运行层面是完全没有任何区别的。因为无需指令转换，所以运行效率高。


![](https://img2018.cnblogs.com/blog/546912/201903/546912-20190306180649459-1948109870.png)

容器在 linux 中使用 namesapce 实现资源隔离，使用 cgroup 实现资源限制。

