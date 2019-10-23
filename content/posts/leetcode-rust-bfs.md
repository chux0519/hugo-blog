---
title: "Leetcode Rust Bfs"
date: 2019-10-22T22:40:20+08:00
showDate: true
tags: ["rust","leetcode"]
---

## 算法系列 - BFS

BFS 认为是必须要掌握的几个算法之一，这里做一些题目进行练习。

## 简单题

### 题目 [101](https://leetcode.com/problems/symmetric-tree/)

要求实现函数，检查二叉树是否对称，两种做法，比较容易想到的是直接递归，左右子树进行递归解。

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

### 题目 [107](https://leetcode.com/problems/binary-tree-level-order-traversal-ii/)

和题目 102 没有本质区别，这里跳过，参考 102 的解法即可。

### rust 解法链接

- [101](https://github.com/chux0519/leetcode-rust/blob/823b0afc42d3a86f42aa43b98596f9c9b7b2c894/src/q101_symmetric_tree.rs#L26)
- [107](https://github.com/chux0519/leetcode-rust/blob/44034794de7a3fe4817ddd15ce8a15ca5a10ee38/src/q107_binary_tree_level_order_traversal_ii.rs#L26)

## 中等题

### 题目 [102](https://leetcode.com/problems/binary-tree-level-order-traversal/)

要求依照层数遍历二叉树，思想依然是利用一个队列，记录每层的待遍历节点。

伪代码如下：

```python
def level_order(root):
  ret = []
  q = []
  if q is null:
    return ret
  q.push_back(root)
  while len(q) > 0:
    level = []
    for i in 0..len(q):
      front = q.pop_front()
      level.push(front.val)
      if front.left is not null:
        q.push_back(front.left)
      if front.right is not null:
        q.push_back(front.right)
    ret.push(level)
  return ret
```

上面的部分其实可以作为很多 BFS 的模板，思想都是利用一个 queue 存储下一个待处理元素，然后做逻辑处理

### 题目 [103](https://leetcode.com/problems/binary-tree-zigzag-level-order-traversal/submissions/)

原理同 102 题，只是利用一个标志位在对应 level 进行翻转就行了。

### rust 解法链接

- [102](https://github.com/chux0519/leetcode-rust/blob/44034794de7a3fe4817ddd15ce8a15ca5a10ee38/src/q102_binary_tree_level_order_traversal.rs#L26)
- [103](https://github.com/chux0519/leetcode-rust/blob/44034794de7a3fe4817ddd15ce8a15ca5a10ee38/src/q103_binary_tree_zigzag_level_order_traversal.rs#L26)
