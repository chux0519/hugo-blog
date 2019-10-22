---
title: "Leetcode Rust Bfs"
date: 2019-10-22T22:40:20+08:00
showDate: true
tags: ["rust","leetcode"]
---

## 算法系列 - BFS

BFS 认为是必须要掌握的几个算法之一，这里做一些题目进行练习。

## 简单题

题目 [101](https://leetcode.com/problems/symmetric-tree/), 两种做法，比较容易想到的是直接递归，左右子树进行递归解。

伪代码如下：

```python
def symmetric(n1, n2):
  if n1 is null or n2 is null:
    return true
  if n1.val == n2.val:
    return symmetric(n1.left, n2.right) && symmetric(n1.right, n2.left)
```

思路简单，但是需要利用程序调用栈。另一种方法更接近 BFS 的解法，使用队列，伪代码如下：

```python
def symmetric(root):
  q = []
  q.push_back(root)
  q.push_back(root)
  while true:
    if len(q) == 0:
      break
    l = q.pop_front()
    r = q.pop_front()
    if l is not null and r is not null and l.val == r.val:
      q.push_back(l.left)
      q.push_back(r.right)
      q.push_back(l.right)
      q.push_back(r.left)
    else:
      return false
  return true
```

利用 rust 实现写法类似, 只是要注意满足编译器需求即可，代码在 [这里](https://github.com/chux0519/leetcode-rust/blob/823b0afc42d3a86f42aa43b98596f9c9b7b2c894/src/q101_symmetric_tree.rs#L26)
