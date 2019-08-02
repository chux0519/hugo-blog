---
title: node杂记 从递归到内存泄漏
date: 2018-06-15 16:42:22
img: https://cdn-images-1.medium.com/proxy/1*H-25KB7EbSHjv70HXrdl6w.png
categories:
- node杂记
tags:
- 笔记
---

## 前言

最近在 [stackoverflow](https://stackoverflow.com/questions/42788139/es6-tail-recursion-optimisation-stack-overflow) 看到了 v8 支持 TCO 后又废弃的原因。大意是指 TCO 带来了一些调试问题和 bug，v8 团队支持改变提案规范，在时间提案时间节点之前 v8 团队没有完成，因此在代码中删除了这个特性，避免它成为隐患。了解到目前版本的 node 不支持尾递归优化，所以，即使是写出了尾递归版本的代码，v8还是不会避免压栈，因此仍然会有栈溢出的风险。

## CPS变换（continuation-passing style）

个人理解，CPS变换是函数式编程里的一个概念，大意指显式使用一个函数，表达函数返回后的执行流程。其中 `C` 代指 continuation，关于这个名词的解释，知乎上有一个很好的回答[怎样理解 Continuation-passing style? - 韦冠楠的回答 - 知乎](https://www.zhihu.com/question/20259086/answer/141162748)。个人的浅显理解为，将计算过程做一个抽象，能够表示从任意一点截取之后的操作流程。它本身是链式的。

这里举个例子，用求和的算法来说明（整个例子参考简书用户： [灼弦](https://www.jianshu.com/p/bf84d0b76111) 的文章中的线性递归的代码]）。

```javascript
// 简单递归求和写法
const sumRec = x => { if (x === 0) return 0; else return x + sumRec(x - 1) }

// CPS变换，转递归为尾递归
// nodejs 依赖于v8，当前版本的实现已经不支持尾递归优化，因此下面的写法虽然是尾递归，但是仍会爆栈
const sumCPS = (x, kont) => {
  if (x === 0) return kont(0)

  else return sumCPS(x - 1, res => kont(res + x))
}
```

这里的kont使用简单的 `x => x` 即可（从最后一步开始思考流程）。因此求10以下和可以写为 `sumCPS(10, x => x)`

## Trampoline

紧接上文，使用 CPS 变换后，尾调用函数层层嵌套，永不返回，在缺乏尾调用优化的 v8 中，并不知道函数不会返回，状态、参数压栈的行为还是会发生，因此需要手动强制弹出下一层调用的函数，禁止解释器的压栈行为，这种方法就叫做 Trampoline。

```javascript
// Trampoline方法避免爆栈： 手动强制弹出下一层调用的函数，禁止解释器的压栈行为
function trampoline (kont) {
  while (typeof kont === 'function') { kont = kont() }

  return kont.val
}
```

结合 CPS 变换，我们需要稍微改变一下 kont 函数的使用，使其可以缓存每一次调用的结果（val），达到最终返回的目的。

```javascript
// 利用bind 将val保存在闭包中
function sumBounce (x, kont) {
  if (x === 0) return kont.bind(null, {val: 0})

  else return sumBounce.bind(null, x - 1, res => kont.bind(null, {val: res.val + x}))
}

// 箭头函数也行，主要是利用闭包缓存变量
// function sumBounce (x, kont) {
//   if (x === 0) return () => kont({val: 0})

//   else return () => sumBounce(x - 1, res => () => kont({val: res.val + x}))
// }

const sum = x => trampoline(sumBounce(x, res => res))
```

最终我们得到了一个不会爆栈的 sum 函数。全部代码及测试均保存在 [gist](https://gist.github.com/chux0519/912ec5e118e191a04a0446f235bb1e0b) 上，可以直接在浏览器（最新版本chrome）或是 node 中进行粘贴运行。

## Promise 的内存泄漏

在 js 中，有许多著名的异步流程控制库，多年前 tj 大神写的 `co` 便是一种，最近在网络上看到，其思想便是 CPS，展开了一堆 promise，使其一个接一个的链起来。于是我又去翻看了 `co` 那精简的代码，这次发现了另一个关于 node/v8 的有趣事情。于是本文从 CPS -> 异步流程控制 -> Promise 内存泄漏，这条诡异的知识链条便出现了。

最初是在代码中看到了一条注释，引向了 [co/issues/180](https://github.com/tj/co/issues/180)。这个古老的 issue 下面还发现了很多国内的大佬参与讨论，例如 fengmk2、dead-horse 和 hax。大意是指，在 Promise 的链式引用写法下，会出现内存泄漏，然后导向了另一个 issue ([promises-aplus/promises-spec/issues/179](https://github.com/promises-aplus/promises-spec/issues/179))。

简单的举例说明，使用 fengmk2 的例子即可，实验前开启全局的手动 gc 来排查问题。

> node --harmony --expose-gc

以下代码会造成内存泄漏

```javascript
var i = 0

function next () {
  return new Promise(function (resolve) {
    i++
    if (i % 100000 === 0) {
      global.gc()
      console.log(process.memoryUsage())
    }
    setImmediate(resolve)
  }).then(next)
}

next()
```

但是删去一个 `return` 关键词后就不会

```javascript
var i = 0

function next () {
 new Promise(function (resolve) {
    i++
    if (i % 100000 === 0) {
      global.gc()
      console.log(process.memoryUsage())
    }
    setImmediate(resolve)
  }).then(next)
}

next()
```

这是什么原理呢？实际上可以类比为递归爆栈的现象，代码会在内存中不断维护新的 promise，最后内存错误。可以构造以下代码便于理解。

```javascript
// 栈溢出
function run(i) {
  return i < 99999999 ? run(i + 1) : i;
}
console.log(run(0));


// 内存泄漏
function run(i) {
    return  new Promise(function(resolve) {
        setImmediate(resolve);
    }).then(function () {
      return i < 99999999 ? run(i + 1) : i;
    });
}
run(0).then(function (result) {
  console.log(result);
});
```

因此本质上是因为代码产生了许多需要维护的 promise，最终造成内存泄漏。

### 解决方案

我们可以在 Promise 内部实现递归调用函数，来打破 Promise chain，从容避免内存泄漏，详见 [promises-aplus/promises-spec/issues/179](https://github.com/promises-aplus/promises-spec/issues/179)。

而 `co` 库采用，使用 Promise 包装整个过程，避免了 Promise 的 chaining。可以参考[相关提交记录](https://github.com/tj/co/pull/181/files)。