---
layout: post
title: 06-05笔记
date: 2017-06-05
categories: [MyNotes]
tags: [DOM]

---

###mouseenter和mouseover的区别
mouseenter不会冒泡，发生在某个元素后，不会传递给父元素
而mouseover会，如果发生在子元素上，也会冒泡到父元素。
**注意**:mouseenter的性能是有问题的，因为会不停的在不同的层级的元素上触发

### nrm
nrm 是一个管理 npm 源的工具，，用来切换官方 npm 源和国内的 npm 源（如: cnpm），当然也可以用来切换官方 npm 源和公司私有 npm 源。
- 查看当前 nrm 内置的几个 npm 源的地址：
```
nrm ls
```
- 切换到 cnpm：
```
nrm use cnpm
```