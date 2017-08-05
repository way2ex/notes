---

layout: post

title: "encodeURI和encodeURIComponent的区别。"

date: 2017-08-03

categories: [Web]


---

* content
{:toc}

``encodeURI``和``encodeURIComponent``两个函数的区别。
## 结论
都是对URI字符进行编码，``encodeURI``用于对完整的URI进行编码，而``encodeURIComponent``则用于对URI中的一部分进行编码。
具体的区别就是:
``encodeURI``**不会**对以下字符编码:
``;``,``/``, ``?``, ``:``, ``@``, ``&``, ``=``, ``+``, ``$``
而``encodeURIComponent``**会**对这些字符编码。

而对于以下符号，两个函数均**不会**进行编码
``-``, ``_``, ``.``, ``!``, ``~``, ``*``, ``'``, ``(``, ``)``
对于英文字母和阿拉伯数字，也都不会进行编码。

出现在URI中的其他所有特殊符号，都将进行编码。

## URI字符
URI字符分为
- uri保留字符
	``;``,``/``, ``?``, ``:``, ``@``, ``&``, ``=``, ``+``, ``$``
    ``encodeURI``不编码，``encodeURIComponent``编码

- uri非转义字符
	包括：
	- 英文字母
	- 阿拉伯数字
	- uri标记符号
		``-``, ``_``, ``.``, ``!``, ``~``, ``*``, ``'``, ``(``, ``)``

   	``encodeURI``和``encodeURIComponent``均不编码

- uri转义字符
	uri转义字符指 ``%xx``这种形式的字符，其中``xx``代表十六进制的数字，即0-9和A-F
    ``encodeURI``和``encodeURIComponent``均会进行编码
    

<div>markdown可以放进可运行的代码吗？</div>
<button>点击我</button>
<script>
document.querySelector("button").onclick = function(){
	alert('haha')
}
<script>
