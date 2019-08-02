---
title: nand2tetris chapter 1 (逻辑门电路)
date: 2018-05-02 09:07:33
img: http://nand2tetris.org/banner.png
categories:
- 学习
- coursera
tags:
- 笔记
- coursera
---

笔记为 `Build a Modern Computer from First Principles: From Nand to Tetris` 第一周记录，逻辑门电路的一些实现。

## 知识点

- AND 和 NOT 电路可以创造一切
- NAND 电路可以创造一切

## 门电路实现

已有门电路NAND，需要实现其他基础门电路。

### NOT

推导

> NOT(x) = (x NAND x)

```hdl
CHIP Not {
    IN in;
    OUT out;

    PARTS:
    
    Nand(a=in, b=in, out=out);
}
```

### AND

推导

> x AND y = NOT ( x NAND y )


```hdl
CHIP And {
    IN a, b;
    OUT out;

    PARTS:
    Nand(a=a, b=b, out=nandab);
    Not(in=nandab, out=out);
}
```

### OR

推导

> a OR b = NOT ((NOT a) AND (NOT b))

```hdl
CHIP Or {
    IN a, b;
    OUT out;

    PARTS:
    Not(in=a, out=nota);
    Not(in=b, out=notb);
    And(a=nota, b=notb, out=notab);
    Not(in=notab, out=out);
}
```

### XOR

推导

> a XOR b = (a AND (NOT b)) OR ((NOT a) AND b)

```hdl
CHIP Xor {
    IN a, b;
    OUT out;

    PARTS:
    Not(in=a, out=nota);
    Not(in=b, out=notb);
    And(a=a, b=notb, out=w1);
    And(a=nota, b=b, out=w2);
    Or(a=w1, b=w2, out=out);
}
```

### MUX(Multiplexor)

推导

> MUX(a, b, sel) = ((NOT sel) AND a) OR (sel AND b)

```hdl
CHIP Mux {
    IN a, b, sel;
    OUT out;

    PARTS:
    Not(in=sel, out=notsel);
    And(a=a, b=notsel, out=w1);
    And(a=sel, b=b, out=w2);
    Or(a=w1, b=w2, out=out);
}
```

### DMUX(Demultiplexor)

推导

> a = (NOT sel) AND in, b = sel AND in

```hdl
CHIP DMux {
    IN in, sel;
    OUT a, b;

    PARTS:
    Not(in=sel, out=notsel);
    And(a=in, b=notsel, out=a);
    And(a=sel, b=in, out=b);
}
```

### 16bit AND/OR/NOT/MUX

### 16bit 4way MUX

推导

```text
先实现16bit的MUX，使用三个16bit的MUX，并联两个，再级联输出
```

```hdl
CHIP Mux4Way16 {
    IN a[16], b[16], c[16], d[16], sel[2];
    OUT out[16];

    PARTS:

    Mux16(a=a, b=b, sel=sel[0], out=w1);
    Mux16(a=c, b=d, sel=sel[0], out=w2);
    Mux16(a=w1, b=w2, sel=sel[1], out=out);
}
```

### 16bit 8way MUX

思路同4路
真值表如下

| sel[2] | sel[1] | sel[0] | out  |
| :----: | :----: | :----: | :--: |
|    0   |    0   |    0   |   a  |
|    0   |    0   |    1   |   b  |
|    0   |    1   |    0   |   c  |
|    0   |    1   |    1   |   d  |
|    1   |    0   |    0   |   e  |
|    1   |    0   |    1   |   f  |
|    1   |    1   |    0   |   g  |
|    1   |    1   |    1   |   h  |

```hdl
CHIP Mux8Way16 {
    IN a[16], b[16], c[16], d[16],
       e[16], f[16], g[16], h[16],
       sel[3];
    OUT out[16];

    PARTS:
    Mux16(a=a, b=b, sel=sel[0], out=w1);
    Mux16(a=c, b=d, sel=sel[0], out=w2);
    Mux16(a=e, b=f, sel=sel[0], out=w3);
    Mux16(a=g, b=h, sel=sel[0], out=w4);
    Mux16(a=w1, b=w2, sel=sel[1], out=w12);
    Mux16(a=w3, b=w4, sel=sel[1], out=w34);
    Mux16(a=w12, b=w34, sel=sel[2], out=out);
}
```

### 4way DMUX

思路类似MUX，不过sel的顺序颠倒

```hdl
CHIP DMux4Way {
    IN in, sel[2];
    OUT a, b, c, d;

    PARTS:
    DMux(in=in, sel=sel[1], a=w12, b=w34);
    DMux(in=w12, sel=sel[0], a=a, b=b);
    DMux(in=w34, sel=sel[0], a=c, b=d);
}
```

### 8way DMUX

```hdl
CHIP DMux8Way {
    IN in, sel[3];
    OUT a, b, c, d, e, f, g, h;

    PARTS:
   	DMux(in=in, sel=sel[2], a=wleft, b=wright);
   	DMux(in=wleft, sel=sel[1], a=w12, b=w34);
   	DMux(in=wright, sel=sel[1], a=w56, b=w78);
   	DMux(in=w12, sel=sel[0], a=a, b=b);
   	DMux(in=w34, sel=sel[0], a=c, b=d);
   	DMux(in=w56, sel=sel[0], a=e, b=f);
   	DMux(in=w78, sel=sel[0], a=g, b=h);
}
```
