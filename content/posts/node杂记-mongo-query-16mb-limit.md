---
title: node杂记 mongo query 16mb limit
date: 2018-07-12 13:21:10
img: https://cdn-images-1.medium.com/proxy/1*H-25KB7EbSHjv70HXrdl6w.png
categories:
- node杂记
tags:
- 工作
- 笔记
---

在使用 mongodb 查询时，有时需要传入很长的 array 进行 $in 操作符的过滤。因此 query 的对象存在大小限制。查询文档得知，mongo 的 bson object 大小限制是 16mb，意味着，query 对象也是遵守这个限制。因此，根据自身需要，计算大概的 array 长度限制，分批进行查询。

![error](16mb-limit.jpeg)

- [参考文档](https://docs.mongodb.com/manual/reference/limits/)
- [stackoverflow](https://stackoverflow.com/questions/11688658/what-is-the-maximum-length-of-a-mongodb-query)
- [验证代码](https://bitbucket.org/chux0519/mongo-query-limit-proof)
