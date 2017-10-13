---

layout: post

title: "一点小区别"

date: 2017-10-10

categories: [JavaScript]

---

* content
{:toc}

```js
var num = 10000000
var a = ''
var b
for(var i = 0; i < num; i++) {
  a += '1'
}
console.time('str.charCodeAt(i)')
for(var i = 0; i < num; i++) {
  b = a.charCodeAt(i)
}
console.timeEnd('str.charCodeAt(i)')

console.time('str[i].charCodeAt(0)')
for(var i = 0; i < num; i++) {
  b = a[i].charCodeAt(0)
}
console.timeEnd('str[i].charCodeAt(0)')
```
结果是
```js
// str.charCodeAt(i): 139.428ms
// str[i].charCodeAt(0): 13.365ms
```


