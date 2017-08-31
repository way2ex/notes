---

layout: post

title: "尾递归优化"

date: 2017-08-31

categories: [JavaScript]


---

* content
{:toc}

详情参考[这里](http://es6.ruanyifeng.com/#docs/function#尾调用优化)

## 尾递归优化
由于v8引擎和浏览器都没有实现尾递归优化，或者实现了，但是不能使用，所以需要手动去部署优化。
最合理的方法是使用下面这种：
```js
function tco(f) {
  var value;
  var active = false;
  var accumulated = [];

  return function accumulator() {
    accumulated.push(arguments);
    if (!active) {
      active = true;
      while (accumulated.length) {
        value = f.apply(this, accumulated.shift());
      }
      active = false;
      return value;
    }
  };
}
// 使用
var sum = tco(function(x, y) {
  if (y > 0) {
    return sum(x + 1, y - 1)
  }
  else {
    return x
  }
});

sum(1, 100000)
// 100001
```

## 其他方法
### 方法一
还可以这样循环执行：
```js
function sum(x, y){
  while(y>0) {
  	[x, y] = [x+1, y-1]
  }
  return x
}
```

### 方法二
使用蹦床函数(Trampolining)：
```js
function trampoline(f) {
  while( f && f instanceof Function ) {
    f = f()
  }
  return f
}

function sum(x, y) {
  if (y > 0) {
    return f.bind(null, n - 1, x + 1, y - 1)
  } else {
  	return x
  }
}
```


