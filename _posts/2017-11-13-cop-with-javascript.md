---

layout: post

title: "使用javascript复制元素的文本"

date: 2017-11-13

categories: [Javascript]

---

* content
{:toc}

## javascript复制
代码如下：
```js
function CopyToClipboard(containerid) {
if (document.selection) { 
    var range = document.body.createTextRange();
    range.moveToElementText(document.getElementById(containerid));
    range.select().createTextRange();
    document.execCommand("copy"); 

} else if (window.getSelection) {
    var range = document.createRange();
     range.selectNode(document.getElementById(containerid));
     window.getSelection().addRange(range);
     document.execCommand("copy");
}}

```

