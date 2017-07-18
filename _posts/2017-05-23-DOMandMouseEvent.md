---
layout: post
title: "元素的位置和鼠标事件的属性"
date: 2017-05-23
categories: [Web API, MyNotes]

---

## 概念
- MouseEvent.clientX:
获取鼠标点击时相对于屏幕的水平坐标，页面发生滚动时不考虑滚动，各个浏览器都实现了基本功能

- event.stopPropagation() :
防止事件传播


- event.preventDefault() :
防止默认事件的执行，但是不阻止事件的传播


- event.stopImmediatePropagation():
添加事件监听器后，事件发生时会按照添加的顺序依次调用监听函数，调用此方法后，会阻止其他监听器函数的调用


- event.cancelBubble :
和``event.stoppropagation()``相同，set ``event.cancelBubble = true;`` to prevents propagation of the event

### 元素位置
- HTMLElement.offsetParent:
在DOM层级关系中，一个元素的所有父容器中，距离该元素最近的，并且position属性为``fixed | relative | absolute``的容器，即为该元素的offsetParent。如果没有符合条件的父容器，则offsetParent为最近的table cell或者根元素(根元素在标准兼容模式下为html,在混合渲染模式下为body)。
当元素的display属性设置为``display: none;``的时候，会返回null

- HTMLElement.offsetTop & HTMLElement.offsetLeft:
 分别是当前元素上边界和左边界距离offsetParent的padding相应边界的距离

- HTMLElement.clientTop & HTMLElement.clientLeft:
左边框和上边框的宽度，如果元素方向是从右到左并且左边出现了滚动条，则包括滚动条的宽度。

- HTMLElement.offsetWidth  & HTMLElement.offsetHeight :
元素的实际宽度和高度，包括边框、padding和内容区域。滚动条也被包含在内。

- HTMLElement.clientHeight & HTMLElement.clientWidth:
 内容区域加padding的宽度或高度，不包括滚动条。

- HTMLElement.scrollWidth & HTMLElement.scrollWidth:
 等于全部内容区域高度或宽度加padding的高度或宽度

### 鼠标点击位置
- MouseEvent.offsetX & MouseEvent.offsetY:
鼠标点击的位置相对于目标元素最左侧的padding边界或最上侧的padding边界的距离
**注意**,基本功能已经得到现代浏览器的实现

- MouseEvent.clientX & MouseEvent.clientY:
 相对于整个窗口左侧和上侧的距离
- MouseEvent.x & MouseEvent.y:
 相对于整个窗口左侧和上侧的距离,与clientX相同

- MouseEvent.screenX & MouseEvent.screenY:
鼠标事件发生的地方相对于整个屏幕的距离

- MouseEvent.pageX & MouseEvent.pageY:
鼠标点击发生的地方相对于页面的距离，包括滚动后的距离

### mouseenter和mouseover的区别
mouseenter不会冒泡，发生在某个元素后，不会传递给父元素
而mouseover会，如果发生在子元素上，也会冒泡到父元素。
**注意**:mouseenter的性能是有问题的，因为会不停的在不同的层级的元素上触发