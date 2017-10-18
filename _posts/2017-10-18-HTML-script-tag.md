---

layout: post

title: "HTML中的script标签属性"

date: 2017-10-18

categories: [HTML]

---

* content
{:toc}

## 属性
### async
该属性指示浏览器是否对script脚本**异步执行**。仅仅对于指定了``src``的脚本有效。
动态插入的``script``脚本默认是**异步执行**的。
> 表示异步执行引入的 JavaScript，与 defer 的区别在于，如果已经加载好，就会开始执行——无论此刻是 HTML 解析阶段还是 DOMContentLoaded 触发之后

### crossorigin
  普通的``script``如果没有通过浏览器对``CORS``的检查，会传给``window.onerror``必要的信息。对于那些使用单独域来存放静态资源的站点，可以使用这个属性来使其记录错误日志。
 ``<img>, <video> or <script>``这些都可以跨域请求资源，设置``crossorigin``属性来对这些请求进行配置。

  取值如下：
  - ``anonymous``: 使用这个属性，请求资源时，不会发送自己的`` credentials flag set``。无效值或者空字符串都会当做这个值来处理。
  - ``use-credentials``: [CORS](https://developer.mozilla.org/en-US/docs/HTTP/Access_control_CORS) requests for this element will have the credentials flag set; this means the request will provide credentials.

  不指定该属性时，CORS 不会被使用。``anonymous``表明不会通过 cookies , client-side  SSL certificates or HTTP authentication 来像[Terminology section of the CORS specification](http://www.w3.org/TR/cors/#user-credentials)中描述的那样交换证书。
### defer
  这个属性表明脚本会在文档解析完,并在触发[DOMContentLoaded](https://developer.mozilla.org/en-US/docs/Web/Events/DOMContentLoaded)之前被执行。
  >defer 属性表示延迟执行引入的 JavaScript，即这段 JavaScript 加载时 HTML 并未停止解析，这两个过程是并行的。整个 document 解析完毕且 defer-script 也加载完成之后（这两件事情的顺序无关），会执行所有由 defer-script 加载的 JavaScript 代码，然后触发 DOMContentLoaded 事件。
### integrity
该属性包含一个metadata,user agent 可以使用它来确定资源在没有引发意外操作的情况下被获取到。看这里[Subresource Integrity](https://developer.mozilla.org/en-US/docs/Web/Security/Subresource_Integrity).

### nomodule 
表明脚本不能在支持ES6的module语法的浏览器中执行。

### src

### text
类似于``textContent``，用于设置元素的文本内容。和``textContent``不同的是，在节点被插入到DOM中后，这里的内容将被当做可执行代码.

### type
指示脚本的类型。属性的取值可以是以下范围中的某个：
- **Omitted or a JavaScript MIME type**：在支持HTML5的浏览器中，这个属性表明脚本是JavaScript。. JavaScript MIME types are [listed in the specification](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types).
- **module**: 指示脚本代码会被当做JavaScript Module 来对待。对代码的处理不会被``charset``和``defer``属性影响。对于模块的使用，参考[ES6 in Depth: Modules](https://hacks.mozilla.org/2015/08/es6-in-depth-modules/).
- **Any other value or MIME type**:这些值指示脚本中的内容当做数据块



