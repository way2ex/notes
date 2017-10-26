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

### defer
  这个属性表明脚本会在文档解析完,并在触发[DOMContentLoaded](https://developer.mozilla.org/en-US/docs/Web/Events/DOMContentLoaded)之前被执行。
  >defer 属性表示**延迟执行**引入的 JavaScript，即这段 JavaScript 加载时 HTML 并未停止解析，这两个过程是并行的。整个 document 解析完毕且 defer-script 也加载完成之后（这两件事情的顺序无关），会执行所有由 defer-script 加载的 JavaScript 代码，然后触发 DOMContentLoaded 事件。

### crossorigin
  普通的``script``如果没有通过浏览器对``CORS``的检查，会传给``window.onerror``必要的信息。对于那些使用单独域来存放静态资源的站点，可以使用这个属性来使其记录错误日志。
 ``<img>, <video> or <script>``这些都可以跨域请求资源，设置``crossorigin``属性来对这些请求进行配置。

  取值如下：
  - ``anonymous``: 使用这个属性，请求资源时，不会发送自己的`` credentials flag set``。无效值或者空字符串都会当做这个值来处理。
  - ``use-credentials``: [CORS](https://developer.mozilla.org/en-US/docs/HTTP/Access_control_CORS) requests for this element will have the credentials flag set; this means the request will provide credentials.

  不指定该属性时，CORS 不会被使用。``anonymous``表明不会通过 cookies , client-side  SSL certificates or HTTP authentication 来像[Terminology section of the CORS specification](http://www.w3.org/TR/cors/#user-credentials)中描述的那样交换证书。

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
- **Any other value or MIME type**:这些值指示脚本中的内容当做数据块来处理。l浏览器不会去执行这些内容。

## defer 与 async的区别与共性
### 共性
1. 都会对脚本进行**异步加载**，也就是加载的时候**不会阻塞渲染**
2. 运行时都**不会阻塞渲染**,因为``async``是异步执行，``defer``是HTML解析后执行。
### 区别
1. 代码的执行时间
 - ``async``指示代码应该在加载完成后立刻运行，这一过程是在window的load事件之前的。
 - ``defer``属性的脚本会在整个HTML结构解析结束后，并在document的DOMContentLoaded事件之前运行。
2. 脚本执行的顺序
 - ``async``修饰的脚本之间不确定按照顺序执行。
 - ``defer``则会按照出现的顺序在HTML解析结束后执行。


## 相关概念
### MIME type
 [MIME type](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types)是用来告诉浏览器传输的文档的类型的。在web中，文件的扩展名没有意义，所以浏览器需要根据这个类型值来决定对文件做如何的处理。

 MIME type不是传达文件类型信息的唯一方法:
 - 文件名后缀。
 - Magic numbers.不同类型的文件有不同的语法，所以可以根据文件的结构来判断文件类型。例如： each GIF files starts with the 47 49 46 38 hexadecimal value [GIF89] or PNG files with 89 50 4E 47 [.PNG]. 

 #### 语法
 ```js 
 type/subtype
 ```
#### Discrete types
 ```html
text/plain
text/html
image/jpeg
image/png
audio/mpeg
audio/ogg
audio/*
video/mp4
application/octet-stream
 ```
 **注意点**：
 - ``application``用于指示任何类型的二进制数据。
 - 对于没有指定z类型的text文档，应使用``text/plain``。对于没有指定具体类型的二进制文档，应使用``application/octet-stream``.

 #### Multipart types
 ```js
multipart/form-data
multipart/byteranges
 ```
 Multipart types指示被分成多个独立部分的文档的某个范围。HTTP只处理两种Multipart类型：``multipart/form-data``和``multipart/byteranges``。
 前者和HTML 表单以及POST方法有关，后者用来和[206](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/206)``Partial Content``状态信息有关，这是用来发送整个文档的部分子集的。
 对于其他的multipart类型，HTTP只是简单的传给浏览器。大部分情况下，浏览器不知道如何显示时，就会出现Save AS窗口。

 #### Important MIME types for Web developers
 - ``multipart/form-data``
 这种类型用来发送完整的HTML 表单数据，它由多个部分组成，以``--开头的字符串``作为分割， 每个部分都是独立的整体，会有自己的http 头部信息。
 ```
Content-Type: multipart/form-data; boundary=aBoundaryString
(other headers associated with the multipart document as a whole)

--aBoundaryString
Content-Disposition: form-data; name="myFile"; filename="img.jpg"
Content-Type: image/jpeg

(data)
--aBoundaryString
Content-Disposition: form-data; name="myField"

(data)
--aBoundaryString
(more subparts)
--aBoundaryString--
```
- multipart/byteranges
这种MIME类型用在向浏览器发送partial responses的上下文中。当后台发送了``206 Partial Content``状态码时，这个类型用来指示文档被分成了几个部分，每个部分对应  each of the requested range。在``Content-Type``中，通过``boundary``来确定各个部分的分隔字符串。
每个部分都有自己的``Content-Type``和``Content-Range``，来指示实际的文档类型和范围。

```
HTTP/1.1 206 Partial Content
Accept-Ranges: bytes
Content-Type: multipart/byteranges; boundary=3d6b6a416f9b5
Content-Length: 385

--3d6b6a416f9b5
Content-Type: text/html
Content-Range: bytes 100-200/1270

eta http-equiv="Content-type" content="text/html; charset=utf-8" />
    <meta name="vieport" content
--3d6b6a416f9b5
Content-Type: text/html
Content-Range: bytes 300-400/1270

-color: #f0f0f2;
        margin: 0;
        padding: 0;
        font-family: "Open Sans", "Helvetica
--3d6b6a416f9b5--

```
### 206 Partial Content
``206 Partial Content``状态码指示request has succeeded and has the body contains the requested ranges of data, as described in the Range header of the request.
如果只有一个range，那么响应的[Content-Type](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Type)设置为文档的类型，并且会提供一个[Content-Range](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Type).

如果有多个ranges，the Content-Type is set to multipart/byteranges and each fragment cover one range, with Content-Range and Content-Type describing it.

**总结**： 就是后台把这次返回的数据分成了几个部分，而每个部分的文档类型不同，有的是text/html，有的是image/gif。每个部分都有自己的``Content-Type``和``Content-Range``，来指示具体的文档类型以及字节所处的范围。
#### 只有一个Range的响应
```
HTTP/1.1 206 Partial Content
Date: Wed, 15 Nov 2015 06:25:24 GMT
Last-Modified: Wed, 15 Nov 2015 04:58:08 GMT
Content-Range: bytes 21010-47021/47022
Content-Length: 26012
Content-Type: image/gif

... 26012 bytes of partial image data ...
```

```
HTTP/1.1 206 Partial Content
Date: Wed, 15 Nov 2015 06:25:24 GMT
Last-Modified: Wed, 15 Nov 2015 04:58:08 GMT
Content-Length: 1741
Content-Type: multipart/byteranges; boundary=String_separator

--String_separator
Content-Type: application/pdf
Content-Range: bytes 234-639/8000

...the first range...
--String_separator
Content-Type: application/pdf
Content-Range: bytes 4590-7999/8000

...the second range
--String_separator--
```