---
title: "Raspi3 Os 03"
date: 2019-11-27T13:41:09+08:00
showDate: true
draft: true
tags: ["blog","story"]
---

这是 raspi-os 的第三篇，主要想要设置好 uart，方便开发和调试。

## UART

可以理解为是一种用于信息传输用的硬件，见[维基百科](https://zh.wikipedia.org/wiki/UART)

在树莓派 3b 里面，有两个相关硬件，一个是 mini uart，另一个是 PL011 UART。

这一节，我们使用 qemu 模拟这个过程，qemu 已经实现 UART，因此我们直接使用 PL011 的 UART 地址，往对应的地址写数据即可模拟数据发送了。

## MMIO

- 这些名词是什么 MMU / MMIO
- 在哪里查找资料

## 技术手册

网上公开的资料比较难找到 bcm2837(raspi3b) 的资料，大多都是 bcm2835 的资料。然而一番搜索后，发现 stanford 大学的 cs140e 课程，对资料做了更新，提供了 (bcm2837)[https://web.archive.org/web/20190407101616/https://cs140e.sergio.bz/docs/BCM2837-ARM-Peripherals.pdf] 的资料。

两个芯片的主要区别在于 GPIO 的物理基址，bcm2835 是 0x20000000，而 bcm2837 是 0x3F000000

### 几个地址

技术手册中提到了几个概念，这里是我对这些概念的概括（可能存在偏差，欢迎指正）

- 物理地址：即芯片真实的地址。
- 虚拟地址：即 MMU 映射的 4GB 虚拟地址空间，通常是给各个程序使用(CPU访问)
- 总线地址(bus address)：即 video core 映射的芯片总线的地址，主要映射 IO 口，SD 卡等等的地址（DMA操作 RAM 不需要通过 CPU，因此要通过总线）

即，软件要用到的是物理地址或是虚拟地址，DMA 机制使用到的地址必须是总线地址。

这里对技术手册中的地址做一些总结 ：

1. 物理地址： 0x00000000 - 0x04000000 (64K)
2. 虚拟地址，在 kernel 模式下，用到了 0xC0000000 - 0xEFFFFFFF;在用户模式下，一般用 0x00000000 - 0xBFFFFFFF。外设所映射到的虚拟地址基址是 0xF2000000 。即物理地址 0x3F000000 对应的虚拟地址基址是 0xF20000000 。
3. 总线地址，在总线中，GPIO 的基址是 0x7E000000

因此可以看出：对于 GPIO 来讲，总线 0x7Exxxxxx 对应虚拟地址 0xF2xxxxxx，对应物理地址 0x3Fxxxxxx
