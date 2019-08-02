---
title: nand2tetris chapter 3 (sequential)
date: 2018-05-14 23:17:44
img: http://nand2tetris.org/banner.png
categories:
- 学习
- coursera
tags:
- 笔记
- coursera
---

## sequential logic

首先引入了时钟的概念，然后解释组合逻辑(combinatorial)和顺序逻辑(sequential).

- 时钟将时间离散化，从而使计算能够有顺序、状态可言
- 电子元器件充放电有延时，因此时钟间隔必须大一些，足够忽略延时
- 组合逻辑立即计算当前状态，顺序逻辑和上一个状态有关

> combinatorial: out[t] = function(in[n])
>
> sequential: out[t] = function(in[n-1])

## flip flop

触发器

DFF (clocked data flip flop)

> out[t] = in[t-1]

在本课程中，DFF将作为一种原始的触发器，不必了解其实现细节，虽然实现细节非常优雅，但是可能会造成误解。

## 1bit register

画图有助于分析。使用MUX，in作为a，上次结果out作为b，load作为select。

![1bit register](1bitregister.png)

## ram unit

- ram抽象为n个可以被寻址的寄存器序列
- 同一时间内，只有一个寄存器可以被选中
- 地址线的宽度应该是log2n
- 需要时钟驱动

![ramunit](ramunit.png)

### 读逻辑

- 设置 address
- emit out

### 写逻辑

- 设置 address
- 设置 in
- 设置 load为1

## counter(pc)

可能存在的操作有三种

- reset: 取第一条指令 `pc = 0`
- next: 取下一条指令 `pc++`
- goto: 取第n条指令 `pc = n`

## 实现

### bit

需注意，直接使用out作为MUX的输入是不可行的，需要同时借助第三个变量，之后使用Or门电路进行输出。

```hdl
/**
 * 1-bit register:
 * If load[t] == 1 then out[t+1] = in[t]
 *                 else out does not change (out[t+1] = out[t])
 */

CHIP Bit {
    IN in, load;
    OUT out;

    PARTS:
    Mux(a=dffout, b=in, sel=load, out=dffin);
    DFF(in=dffin, out=dffout);
    Or(a=dffout, b=dffout, out=out);
}
```

### register

直接使用16个1bit的寄存器实现即可

```hdl
/**
 * 16-bit register:
 * If load[t] == 1 then out[t+1] = in[t]
 * else out does not change
 */

CHIP Register {
    IN in[16], load;
    OUT out[16];

    PARTS:
    Bit(in=in[0], load=load, out=out[0]);
    Bit(in=in[1], load=load, out=out[1]);
    Bit(in=in[2], load=load, out=out[2]);
    Bit(in=in[3], load=load, out=out[3]);
    Bit(in=in[4], load=load, out=out[4]);
    Bit(in=in[5], load=load, out=out[5]);
    Bit(in=in[6], load=load, out=out[6]);
    Bit(in=in[7], load=load, out=out[7]);
    Bit(in=in[8], load=load, out=out[8]);
    Bit(in=in[9], load=load, out=out[9]);
    Bit(in=in[10], load=load, out=out[10]);
    Bit(in=in[11], load=load, out=out[11]);
    Bit(in=in[12], load=load, out=out[12]);
    Bit(in=in[13], load=load, out=out[13]);
    Bit(in=in[14], load=load, out=out[14]);
    Bit(in=in[15], load=load, out=out[15]);
}
```

### ram 8

八块16bit寄存器的芯片，注意使用8way的dmux进行load的选择，使用8way，16bit的mux进行结果的选择。

```hdl
/**
 * Memory of 8 registers, each 16 bit-wide. Out holds the value
 * stored at the memory location specified by address. If load==1, then
 * the in value is loaded into the memory location specified by address
 * (the loaded value will be emitted to out from the next time step onward).
 */

CHIP RAM8 {
    IN in[16], load, address[3];
    OUT out[16];

    PARTS:
    DMux8Way(
      in=load, sel=address,
      a=load0, b=load1, c=load2, d=load3,
      e=load4, f=load5, g=load6, h=load7
    );
    Register(in=in, load=load0, out=out0);
    Register(in=in, load=load1, out=out1);
    Register(in=in, load=load2, out=out2);
    Register(in=in, load=load3, out=out3);
    Register(in=in, load=load4, out=out4);
    Register(in=in, load=load5, out=out5);
    Register(in=in, load=load6, out=out6);
    Register(in=in, load=load7, out=out7);
    Mux8Way16(
      a=out0, b=out1, c=out2, d=out3,
      e=out4, f=out5, g=out6, h=out7,
      sel=address, out=out
    );
}
```

### ram 64

使用八块ram8组成，思路同ram8。（使用高三位选ram8，使用低三位传入ram8）。

```hdl
/**
 * Memory of 64 registers, each 16 bit-wide. Out holds the value
 * stored at the memory location specified by address. If load==1, then
 * the in value is loaded into the memory location specified by address
 * (the loaded value will be emitted to out from the next time step onward).
 */

CHIP RAM64 {
    IN in[16], load, address[6];
    OUT out[16];

    PARTS:
    DMux8Way(
      in=load, sel=address[3..5],
      a=ram00, b=ram01, c=ram02, d=ram03,
      e=ram04, f=ram05, g=ram06, h=ram07
    );
    RAM8(in=in, load=ram00, address=address[0..2], out=ramout00);
    RAM8(in=in, load=ram01, address=address[0..2], out=ramout01);
    RAM8(in=in, load=ram02, address=address[0..2], out=ramout02);
    RAM8(in=in, load=ram03, address=address[0..2], out=ramout03);
    RAM8(in=in, load=ram04, address=address[0..2], out=ramout04);
    RAM8(in=in, load=ram05, address=address[0..2], out=ramout05);
    RAM8(in=in, load=ram06, address=address[0..2], out=ramout06);
    RAM8(in=in, load=ram07, address=address[0..2], out=ramout07);
    Mux8Way16(
      a=ramout00, b=ramout01, c=ramout02, d=ramout03,
      e=ramout04, f=ramout05, g=ramout06, h=ramout07,
      sel=address[3..5], out=out
    );
}
```

### pc

使用register和一些逻辑电路进行组合。完成 load, inc, reset操作。

- load: 直接输出in
- inc: inc16
- reset: 直接输出false
- default: 使用register，把 三个sel位全部或运算，然后作为load传入register

最后使用Mux8Way16

| reset | load | inc| out |
| :--: | :--: | :--:  | :--: |
|  1  |  * |  * |   0  |
|  0  |  1 |  * |  in  |
|  0  |  0 |  1 |  out[t] + 1  |
|  0  |  0 |  0 |  out[t] |

```hdl
/**
 * A 16-bit counter with load and reset control bits.
 * if      (reset[t] == 1) out[t+1] = 0
 * else if (load[t] == 1)  out[t+1] = in[t]
 * else if (inc[t] == 1)   out[t+1] = out[t] + 1  (integer addition)
 * else                    out[t+1] = out[t]
 */

CHIP PC {
    IN in[16],load,inc,reset;
    OUT out[16];

    PARTS:

    // inc
    Inc16(in=defaultout, out=incout);

    // default
    Or(a=reset, b=load, out=load0);
    Or(a=load0, b=inc, out=load1);
    Register(in=result, load=load1, out=defaultout);

    Mux8Way16(
      a=defaultout, b=incout, c=in, d=in,
      e=false, f=false, g=false, h=false,
      sel[2]=reset, sel[1]=load, sel[0]=inc,
      out=result
    );
    Register(in=result, load=load1, out=out);
}
```