---
title: "Raspi3 Os 01"
date: 2019-11-16T16:24:14+08:00
showDate: true
draft: true
tags: ["blog","story"]
---

基于树莓派3的操作系统学习，第一部分。

## 工具链

### qemu

这是个模拟器，我们开发时，可以先用 qemu 模拟树莓派，不用每次真的用真机测试。
这里进行手动编译。

```sh
./configure --target-list=aarch64-softmmu --enable-modules \
  --enable-tcg-interpreter --enable-debug-tcg \
  --python=/usr/bin/python2.7; \
```

### clang & llvm

clang 被设计时，定位就是 cross-compiler

安装 llvm 主要是要使用到 llvm-objcopy

另外还需要 lld

编译：

1. 编译 asm

> clang --target=aarch64-elf -Wall -O2 -ffreestanding -nostdinc -nostdlib -mcpu=cortex-a53+nosimd -c start.S -o start.o

参数说明： TODO

2. 链接到 elf 格式

> ld.lld -m aarch64elf -nostdlib start.o -T link.ld -o kernel8.elf

3. 变成二进制镜像

> llvm-objcopy --input-target=aarch64-elf -O binary kernel8.elf kernel8.img

4. 使用 qemu 运行查看效果

>  qemu-system-aarch64 -M raspi3 -kernel kernel8.img -d in_asm
