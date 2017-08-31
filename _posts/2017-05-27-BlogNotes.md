---
layout: post
title: "jekyll搭建博客"
date: 2017-05-27
categories: MyNotes

---

#### jekyll官方形成摘要的方法
[这里](http://www.cnblogs.com/coderzh/p/jekyll-readmore.html)
#### 日期格式
Liquid过滤：
	input  ``{{ article.published_at | date: "%a, %b %d, %y" }}``
    output ``Tue, Apr 22, 14``

    input ``{{ article.published_at | time_tag: '%b %d, %Y' }}``
    output ``<time datetime="2016-02-24T14:47:51Z">Feb 24, 2016</time>``

#### 移动端overflow:hidden 失效解决
给html和body同时设置height:100% ; overflow-x:hidden;

#### 移动端滑动不流畅
给body加上``overflow: auto;-webkit-overflow-scrolling:touch;``

#### html不能设置高度100%
100%是相对于屏幕高度，这样的话，整个页面就只有屏幕高度，body也不会无限伸展，而是会在body或者其中的容器中出现滚动条
