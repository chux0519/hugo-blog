---
title: nand2tetris chapter 2 (Hack ALU 实现)
date: 2018-05-03 21:23:08
img: http://nand2tetris.org/banner.png
categories:
- 学习
- coursera
tags:
- 笔记
- coursera
---

## 二进制位运算原理

进制不再展开，主要讲负数相关计算

| 正数 | 二进制 | 二进制 | 负数 |
| :--: | :--: | :--:  | :--: |
|  0  |  0000 |       |      |
|  1  |  0001 |  1111 |  -1  |
|  2  |  0010 |  1110 |  -2  |
|  3  |  0011 |  1101 |  -3  |
|  4  |  0100 |  1100 |  -4  |
|  5  |  0101 |  1011 |  -5  |
|  6  |  0110 |  1010 |  -6  |
|  7  |  0111 |  1001 |  -7  |
|     |       |  1000 |  -8  |

上表为4bit二进制数表示范围，正数第一位全部为0，负数第一位全部为1。负数算法为 `2^n - m`,
例如`-1 = 2^4 - 1 = 1111`。二进制转换为负数的步骤为：

1. 去掉第一位
2. 剩下几位全部取非
3. 取非结果加1，为负数绝对值
4. 绝对值加上负号即负数

## ALU 

如其名，包含算数运算和逻辑运算两种运算，在课程中，实现18种函数

引脚及功能如下

![alu](chap2-alu-1.png)

真值表如下

|  zx  |  nx  |  zy  |  ny  |   f  |  no  |  out |
| :--: | :--: | :--: | :--: | :--: | :--: | :--: |
| if zx then x =0 | if nx then x = !x | if zy then y = 0 | if ny then y = !y | if f then out = x + y else out = x & y | if no then out = !out | out |
|   1  |   0  |   1  |   0  |   1  |   0  |   0  |
|   1  |   1  |   1  |   1  |   1  |   1  |   1  |
|   1  |   1  |   1  |   0  |   1  |   0  |  -1  |
|   0  |   0  |   1  |   1  |   0  |   0  |   x  |
|   1  |   1  |   0  |   0  |   0  |   0  |   y  |
|   0  |   0  |   1  |   1  |   0  |   1  |  !x  |
|   1  |   1  |   0  |   0  |   0  |   1  |  !y  |
|   0  |   0  |   1  |   1  |   1  |   1  |  -x  |
|   1  |   1  |   0  |   0  |   1  |   1  |  -y  |
|   0  |   1  |   1  |   1  |   1  |   1  |  x+1 |
|   1  |   1  |   0  |   1  |   1  |   1  |  y+1 |
|   0  |   0  |   1  |   1  |   1  |   0  |  x-1 |
|   1  |   1  |   0  |   0  |   1  |   0  |  y-1 |
|   0  |   0  |   0  |   0  |   1  |   0  |  x+y |
|   0  |   1  |   0  |   0  |   1  |   1  |  x-y |
|   0  |   0  |   0  |   1  |   1  |   1  |  y-x |
|   0  |   0  |   0  |   0  |   0  |   0  |  x&y |
|   0  |   1  |   0  |   1  |   0  |   1  |  x|y |

if out == 0 then zr = 1, else zr = 0
if out < 0 then ng = 1, else ng = 0



### Half Adder

> a+b

sum 使用 XOR实现，carry 使用 AND 实现

### Full Adder

> a+b+c

使用两个Half addeer，先计算 b+c, 然后用 sumbc 加上 a 得到sum，两次的carry取或运算，结果即为carry

```hdl
CHIP FullAdder {
    IN a, b, c;  // 1-bit inputs
    OUT sum,     // Right bit of a + b + c
        carry;   // Left bit of a + b + c

    PARTS:
    HalfAdder(a=b, b=c, sum=sumbc, carry=carry1);
    HalfAdder(a=a, b=sumbc, sum=sum, carry=carry2);
    Or(a=carry1, b=carry2, out=carry);
}
```

### 16bit Adder

 低两位使用 Half Adder, 之后使用 Full Adder

```hdl
CHIP Add16 {
    IN a[16], b[16];
    OUT out[16];

    PARTS:
	HalfAdder(a=a[0], b=b[0], sum=out[0], carry=carry1);
	FullAdder(a=a[1], b=b[1], c=carry1, sum=out[1], carry=carry2);
	FullAdder(a=a[2], b=b[2], c=carry2, sum=out[2], carry=carry3);
	FullAdder(a=a[3], b=b[3], c=carry3, sum=out[3], carry=carry4);
	FullAdder(a=a[4], b=b[4], c=carry4, sum=out[4], carry=carry5);
	FullAdder(a=a[5], b=b[5], c=carry5, sum=out[5], carry=carry6);
	FullAdder(a=a[6], b=b[6], c=carry6, sum=out[6], carry=carry7);
	FullAdder(a=a[7], b=b[7], c=carry7, sum=out[7], carry=carry8);
	FullAdder(a=a[8], b=b[8], c=carry8, sum=out[8], carry=carry9);
	FullAdder(a=a[9], b=b[9], c=carry9, sum=out[9], carry=carry10);
	FullAdder(a=a[10], b=b[10], c=carry10, sum=out[10], carry=carry11);
	FullAdder(a=a[11], b=b[11], c=carry11, sum=out[11], carry=carry12);
	FullAdder(a=a[12], b=b[12], c=carry12, sum=out[12], carry=carry13);
	FullAdder(a=a[13], b=b[13], c=carry13, sum=out[13], carry=carry14);
	FullAdder(a=a[14], b=b[14], c=carry14, sum=out[14], carry=carry15);
	FullAdder(a=a[15], b=b[15], c=carry15, sum=out[15], carry=ignore);
}
```

### Inc16

```hdl
CHIP Inc16 {
    IN in[16];
    OUT out[16];

    PARTS:
    Add16(a=in, b[0]=true, out=out);
}
```
 
### ALU 实现

条件判断使用Mux来模拟，即计算出结果，然后进行Mux

```hdl
CHIP ALU {
    IN  
        x[16], y[16],  // 16-bit inputs        
        zx, // zero the x input?
        nx, // negate the x input?
        zy, // zero the y input?
        ny, // negate the y input?
        f,  // compute out = x + y (if 1) or x & y (if 0)
        no; // negate the out output?

    OUT 
        out[16], // 16-bit output
        zr, // 1 if (out == 0), 0 otherwise
        ng; // 1 if (out < 0),  0 otherwise

    PARTS:
    Not16(in=x, out=notx);
    Not16(in=y, out=noty);
    Mux4Way16(sel[1]=zx, sel[0]=nx, a=x, b=notx, c=false, d=true, out=x1);
    Mux4Way16(sel[1]=zy, sel[0]=ny, a=y, b=noty, c=false, d=true, out=y1);

    Add16(a=x1, b=y1, out=xaddy);
    And16(a=x1, b=y1, out=xandy);

    Mux16(sel=f, a=xandy, b=xaddy, out=out1);
    
    Not16(in=out1, out=nout1);
    Mux16(sel=no, a=out1, b=nout1, out[0..7]=lo, out[8..15]=hi, out[15]=ng, out=out);

    Or8Way(in=lo, out=low);
    Or8Way(in=hi, out=high);
    Or(a=low, b=high, out=notzero);

    Not(in=notzero, out=zr);
}
```


## 参考

- [http://nand2tetris.org/02](http://nand2tetris.org/02.php)