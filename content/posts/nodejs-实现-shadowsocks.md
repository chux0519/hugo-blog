---
title: nodejs 实现 shadowsocks
date: 2018-08-29 09:30:44
img: http://thatinterpreter.net/wp-content/uploads/2017/08/Shadowsocks.jpg
categories:
- 学习
tags:
- 笔记
- shadowsocks
---

前段时间，使用 nodejs 实现了 shadowsocks 的 local server。相关项目链接：

- [ss-local](https://github.com/chux0519/ss-local)
- [socks-5](https://github.com/chux0519/socks-5)

这里记录一下摘要，后续再详细记录某些部分，作为一个系列来完成。

## 背景

### [shadowsocks](https://www.wikiwand.com/zh-hans/Shadowsocks) 是什么？

一种基于 [socks5](https://www.wikiwand.com/zh/SOCKS) 代理方式的加密传输协议。通常用来跨越长城，访问被限制、封锁的内容。

流程大概是：

> client <---> ss-local <--[encrypted]--> ss-remote <---> target

### 为什么要实现

为了了解科学上网原理，甚至掌握造梯子的能力。


## socks5 协议实现

由于 ss 基于 socks5 协议，并且社区没有满足需求的 socks5 包，手动实现 TCP only 无权限认证的 socks5 协议。过程主要涉及 buffer 的读写，参考 nodejs api [buffer](https://nodejs.org/dist/latest-v10.x/docs/api/buffer.html)。

ps: 关于[字节序](https://www.wikiwand.com/zh-hans/%E5%AD%97%E8%8A%82%E5%BA%8F)，[ITEF RFC](https://www.wikiwand.com/en/Request_for_Comments) 协议一般采用大端，network order 也叫做 big-endian，因此 buffer 时，请使用带 BE 结尾的 api。源码可以参考: [socks-5](https://github.com/chux0519/socks-5/blob/master/src/socks5.js)。


## 加密相关

ss 支持的[加密方式](https://shadowsocks.org/en/spec/Stream-Ciphers.html)比较多，大多数都是 openssl 中支持的加密函数，在 node 的 crypto 模块中都有包含。需要了解的点主要有两个：

1. 加解密函数需要一定的配置进行生成。
2. 加解密函数是有状态的，即加解密结果和加解密内容顺序是相关的。*(这是踩坑发现的，没有对所有 ss 的加解密函数进行验证)


ps: 原版 ss 中，使用了 openssl 的 `EVP_BytesToKey` 函数生成 key（同样的 password 一定会生成同样的 key）。在 ss-local 项目中，搬运了这个算法，使用 js 实现了。


## 流

在 ss-local 项目中，数据转发、加密部分使用了流。这里主要有以下几点注意：

1. 流速控制（backpressure），使用 pipe 的时候自带 backpressure 处理
2. pipe 层数太多时，需要在每一步都进行错误处理，回收资源，避免内存泄漏。对于 node <= 8 的版本，使用 [pump](https://github.com/mafintosh/pump) 包即可，对于最新版的 node，使用 [pipeline](https://nodejs.org/api/stream.html#stream_stream_pipeline_streams_callback) 即可。
3. 参考 [backpressuring-in-streams](https://nodejs.org/en/docs/guides/backpressuring-in-streams/)
