---
title: "使用 Rust 控制树莓派点亮 oled"
date: 2019-11-16T15:12:17+08:00
showDate: true
draft: true
tags: ["blog","story"]
---

上周给 x200 刷了 libreboot，用到了吃灰已久的树莓派。在准备把它放回角落继续吃灰时，我看到了一块 oled，突发奇想，想用 rust 控制树莓派，点亮它。

搜索一圈以后，发现 rust 其实已经有一些相关的库做了驱动，于是直接用就好了。这里主要记录一下整个流程，提出一些问题供后续继续深入。

## ssd1306 是什么？

### i2c 和 spi 又是什么？

## 我们需要做哪些准备让树莓派可以控制它？

### 对应到 rust 里面是什么样
