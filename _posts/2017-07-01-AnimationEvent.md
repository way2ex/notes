---
layout: post
title: AnimationEvent与TransitionEvent
date: 2017-07-01
categories: [MyNotes, Web API]
tags: JavaScript
---

**Animation相关的事件，目前只有chrome和FireFox支持**

## AnimationEvent中的属性
有下列属性是有使用价值的的:

- animationName: 
 表示当前触发事件的动画名字

- elapsedTime:
 表示当前动画持续的时间，以秒为单位。不包括暂停的时间和延迟的时间。在``animationstart``事件中，若指定了``animation-delay``为负数，则该属性返回``-1``*``animation-delay``的值。

- pseudoElement: 
 表示以``::``开头的伪元素的名字，如果当前动画运行在一个元素上，则返回空字符串，目前没有浏览器实现这个。

## animation有下面几个事件
- animationstart
- animationcancel
- animationiteration
- animationend

### animationstart
当[CSS Animation](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Animations)开始时触发该事件。

### animationcancel
当动画在没有发出``animationend``事件就被终止时触发该事件，比如``animation-name``被中途改变，或者元素被隐藏了。

### animationiteration
顾名思义，该事件在动画完成一次循环时触发

### animationend
在动画结束时触发

## TranstionEvent中的属性
- propertyName: 
 表示当前transition所完成的属性

- elapsedTime:
 表示当前transition完成使用的时间
 
- pseudoElement:
  表示以``::``开头的伪元素的名字，如果当前transition运行在一个元素上，则返回空字符串，目前没有浏览器实现这个。