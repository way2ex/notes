---

layout: post

title: "Ant Design文件上传遇到的问题"

date: 2017-10-10

categories: [JavaScript]

---

* content
{:toc}

## 显式的设置Upload 的 ``fileList``属性
如果显示的设置了``fileList``属性，那么在发生``onChange``事件时，必须显式地更新Upload的该属性，否则，上传的文件的状态永远是``uploading``。


