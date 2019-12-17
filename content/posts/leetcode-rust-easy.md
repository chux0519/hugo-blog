---
title: "Leetcode Rust Easy"
date: 2019-12-17T10:37:14+08:00
showDate: true
draft: true
tags: ["blog", "story"]
---

## 算法系列

简单题

这里记录刷简单题过程中感到有意思的题目

## Maximum subarray problem

这类型的题目描述在：[wiki](https://en.wikipedia.org/wiki/Maximum_subarray_problem#Kadane's_algorithm)

算法还有一个名称，叫做 Kadane's algorithm。在刷到 leetcode 的 121 和 122 题目时，卡住了，不像其他类型的题目，看完题目后就会有比较清晰的想法，直接实现即可。这里学习的思想主要是累加，即给出一个数组，要算法求出获利最大，即 A[n] - A[m] 最大(n > m)

核心是等式

> A[n] - A[m] = A[n] - A[n-1] + A[n-1] - A[n-2] + .. + A[m+1] - A[m]

