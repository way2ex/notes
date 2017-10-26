---

layout: post

title: "load，DOMContentLoaded和readystatechange事件"

date: 2017-10-20

categories: [DOM]

---

* content
{:toc}

## load事件
### XMLHttpRequestEventTarget.onload
该事件是在[XMLHttpRequest](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest)**成功完成**之后触发的。
#### 语法
```js
XMLHttpRequest.onload = callback;
```
``callback``回调函数会接收一个[ProgressEvent](https://developer.mozilla.org/en-US/docs/Web/API/ProgressEvent)作为第一个参数.

#### ProgressEvent
``ProgressEvent``接口代表一些用来测量过程的事件，例如ajax请求，或者``img``,``audio``等资源的加载。其父类是``Event``类。
有以下属性：
- [``ProgressEvent.lengthComputable``](https://developer.mozilla.org/en-US/docs/Web/API/ProgressEvent/lengthComputable)：
只读的boolean，在某些underlying process中，用来指示整个工作完成后旳数量以及已经完成的工作数量是否可以计算。在其他工作中，用来指示整个工作是否可测量。

- [``ProgressEvent.loaded``](https://developer.mozilla.org/en-US/docs/Web/API/ProgressEvent/loaded)：无符号长整型，用来指示已经完成的工作的数量。已完成的比例可以用这个属性和``ProgressEvent.total``相除.
当使用HTTP下载资源时，这个属性仅仅指示内容本身，而不包括头部信息以及其他开销。
- [``ProgressEvent.total``](https://developer.mozilla.org/en-US/docs/Web/API/ProgressEvent/total)

### window.onload
> The load event fires at the end of the document loading process. At this point, all of the objects in the document are in the DOM, and all the images, scripts, links and sub-frames have finished loading.

当文档中所有的对象都添加到DOM中，并且所有的图片，链接资源和sub-frames都加载完成后，才会触发 window 的 ``load`` 事件。

## [``DOMContentLoaded``](https://developer.mozilla.org/en-US/docs/Web/Events/DOMContentLoaded)事件:
该事件会在HTML文档被加载并解析完成后触发，但不会等待文档中的 stylesheet， image 和 subframe 加载完成。

也就是说，当文档结构形成的时候，这个事件就会触发。而``window.onload``事件是在所有的资源加载完成后才触发的，那个时候页面已经完全加载了。

由于同步的脚本代码执行会阻塞HTML渲染，如果希望DOM尽快加载并解析，可以使用[Javascript asynchronous](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/Synchronous_and_Asynchronous_Requests) 和 

#### 语法
```js
document.addEventListener('DOMContentLoaded', function(){})
```
#### 属性
- ``target``: 
- ``type``: 事件类型
- ``bubbles``： 该事件是否会冒泡
- ``cancelable``: 事件是否可以取消

## window.onload和 document 的 DOMContentLoaded 的区别
简单来说，二者的区别是``window.onload``会等待所有的资源加载完成，HTML解析完成才会触发，而``DOMContentLoaded``会在HTML加载并解析完成，形成了完成的DOM结构后触发，而不管image, link和subframe，audio, video是否都加载完成。
**注意**：这里不等待的资源中，不包括同步script脚本。

## readystatechange事件
```js
document.onreadystatechange = function () {
    if (document.readyState === "interactive") {
        initApplication();
    }
}
```

### readyState
- ``loading``:
The document is still loading.
- ``interactive``: The document has finished loading and the document has been parsed but sub-resources such as images, stylesheets and frames are still loading.
等于 ``DOMContentLoaded``时的状态。
- ``complete``: The document and all sub-resources have finished loading. ``window.onload``事件将要触发。
