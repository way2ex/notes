---
layout: post
date: 2017-06-20
title: Node学习02
categories: [Node]
tags: [Node]

---

## 文件操作
### 文件拷贝
NodeJS没有提供文件拷贝的功能，下面用代码来实现文件拷贝。

#### 小文件拷贝
使用NodeJS内置的``fs``模块实现如下:
```
var fs = require('fs');

function copy(src, dst) {
    fs.writeFileSync(dst, fs.readFileSync(src));
}

function main(argv) {
    copy(argv[0], argv[1]);
}

main(process.argv.slice(2));
```
**小知识**：
 process是一个全局变量，可通过``process.argv``获得命令行参数。由于``argv[0]``固定等于NodeJS执行程序的绝对路径，``argv[1]``固定等于主模块的绝对路径，因此第一个命令行参数从argv[2]这个位置开始。

#### 大文件拷贝
上边的程序拷贝一些小文件没啥问题，但这种一次性把所有文件内容都读取到内存中后再一次性写入磁盘的方式不适合拷贝大文件，内存会爆仓。对于大文件，我们只能读一点写一点，直到完成拷贝。
```
var fs = require('fs');

function copy(src, dst) {
    fs.createReadStream(src).pipe(fs.createWriteStream(dst));
}

function main(argv) {
    copy(argv[0], argv[1]);
}

main(process.argv.slice(2));
```

### [常用API]( http://nodejs.org/api/buffer.html)

#### Buffer
JS语言中只有字符串类型的数据，没有二进制类型的数据，NodeJS提供了一个与``String``对等的的全局构造函数``Buffer``来提供对二进制数据的操作。除了可以读取文件得到``Buffer``的实例外，还可以直接构造。
```
var bin = new Buffer([ 0x68, 0x65, 0x6c, 0x6c, 0x6f ]);
```
Buffer与字符串类似，除了可以用``.length``属性得到字节长度外，还可以用``[index]``方式读取指定位置的字节，例如：
```
bin[0]; // => 0x68;
```
``Buffer``与字符串能够互相转化，例如可以使用指定编码将二进制数据转化为字符串：
```
var str = bin.toString('utf-8'); // => "hello"
```
或者反过来，将字符串转换为指定编码下的二进制数据：

```
var bin = new Buffer('hello', 'utf-8'); // => <Buffer 68 65 6c 6c 6f>
```

Buffer与字符串有一个重要区别。字符串是只读的，并且对字符串的任何修改得到的都是一个新字符串，原字符串保持不变。至于``Buffer``，更像是可以做指针操作的C语言数组。例如，可以用``[index]``方式直接修改某个位置的字节。
```
bin[0] = 0x48;
```
而``.slice``方法也不是返回一个新的``Buffer``，而更像是返回了指向原Buffer中间的某个位置的指针，如下所示。
```
[ 0x68, 0x65, 0x6c, 0x6c, 0x6f ]
    ^           ^
    |           |
   bin     bin.slice(2)
```
因此对``.slice``方法返回的``Buffer``的修改会作用于原``Buffer``，例如：
```
var bin = new Buffer([ 0x68, 0x65, 0x6c, 0x6c, 0x6f ]);
var sub = bin.slice(2);

sub[0] = 0x65;
console.log(bin); // => <Buffer 68 65 65 6c 6f>
```
也因此，如果想要拷贝一份``Buffer``，得首先创建一个新的``Buffer``，并通过``.copy``方法把原``Buffer``中的数据复制过去。这个类似于申请一块新的内存，并把已有内存中的数据复制过去。以下是一个例子。
```
var bin = new Buffer([ 0x68, 0x65, 0x6c, 0x6c, 0x6f ]);
var dup = new Buffer(bin.length);

bin.copy(dup);
dup[0] = 0x48;
console.log(bin); // => <Buffer 68 65 6c 6c 6f>
console.log(dup); // => <Buffer 48 65 65 6c 6f>
```

#### [Stream](http://nodejs.org/api/stream.html)
当内存中无法一次装下需要处理的数据时，或者一边读取一边处理更加高效时，就需要用到数据流。NodeJS通过各种``Stream``来处理数据流的操作。
上面大文件的拷贝就使用了``fs.createReadStream(pathname)``和``fs.createWriteStream(pathname)``.

**Knowledge**: ``Stream`` works based on EVENT SYSTEM, all the ``Stream`` instance is inherited from [EventEmitter](http://nodejs.org/api/events.html)

```
var rs = fs.createReadStream(pathname);

rs.on('data', function (chunk) {
    doSomething(chunk);
});

rs.on('end', function () {
    cleanUp();
});
```
上边的代码中``data``事件会源源不断地被触发，不管``doSomething``函数是否处理得过来。代码可以继续做如下改造，以解决这个问题。
```
var rs = fs.createReadStream(src);

rs.on('data', function(chunk){
	rs.pause();
    dosomething(chunk, function(){
    	rs.resume();
    });
});

rs.on('end', function(){
	cleanUp();
});
```
以上代码给doSomething函数加上了回调，因此我们可以在处理数据前暂停数据读取，并在处理数据后继续读取数据。

还可以创建一个只写数据流，
```
var rs = fs.createReadStream(src);
var ws = fs.createWriteStream(dst);

rs.on('data', (chunk)=>{
	ws.write(chunk);
});

rs.on('end', () => {
	ws.end();
});
```
我们把``doSomething``换成了往只写数据流里写入数据后，以上代码看起来就像是一个文件拷贝程序了。但是以上代码存在上边提到的问题，如果写入速度跟不上读取速度的话，只写数据流内部的缓存会爆仓。我们可以根据``.write``方法的返回值来判断传入的数据是写入目标了，还是临时放在了缓存了，并根据``drain``事件来判断什么时候只写数据流已经将缓存中的数据写入目标，可以传入下一个待写数据了。因此代码可以改造如下：
```
var rs = fs.createReadStream(src);
var ws = fs.createWriteStream(dst);

rs.on('data', (chunk) => {
	if(ws.write(chunk) === false){
    	rs.pause();
    }
});

rs.on('end', () => {
	ws.end();
});

rs.on('drain', () => {
	rs.resume();
});
```

#### [File System](http://nodejs.org/api/fs.html)
NodeJS通过``fs``内置模块提供对文件的操作。``fs``的API有三类：
1. 文件属性读写
 常用的有``fs.stat``,``fs.chmod``,``fs.chown``.

2. 文件内容读写
 常用的有``fs.readFile``,``fs.readdir``, ``fs.writeFile``, ``fs.mkdir``.

3. 底层文件操作
 其中常用的有``fs.open``、``fs.read``、``fs.write``、``fs.close``等等。
 
NodeJS最精华的异步IO模型在fs模块里有着充分的体现，例如上边提到的这些API都通过回调函数传递结果。以fs.readFile为例：
```
fs.readFile(pathname, function (err, data) {
    if (err) {
        // Deal with error.
    } else {
        // Deal with data.
    }
});
```
如上边代码所示，基本上所有fs模块API的回调参数都有两个。第一个参数在有错误发生时等于异常对象，第二个参数始终用于返回API方法执行结果。

此外，fs模块的所有异步API都有对应的同步版本，用于无法使用异步操作时，或者同步操作更方便时的情况。同步API除了方法名的末尾多了一个Sync之外，异常对象与执行结果的传递方式也有相应变化。同样以fs.readFileSync为例:
```
try {
    var data = fs.readFileSync(pathname);
    // Deal with data.
} catch (err) {
    // Deal with error.
}
```

#### [Path](http://nodejs.org/api/path.html)
NodeJS内置了``path``来简化对路径的操作。
- path.normalize
 将传入的路径转换为标准路径，具体讲的话，除了解析路径中的.与..外，还能去掉多余的斜杠。如果有程序需要使用路径作为某些数据的索引，但又允许用户随意输入路径时，就需要使用该方法保证路径的唯一性。
 ```
   var cache = {};

  function store(key, value) {
      cache[path.normalize(key)] = value;
  }

  store('foo/bar', 1);
  store('foo//baz//../bar', 2);
  console.log(cache);  // => { "foo/bar": 2 }
 ```
**坑出没注意**： 标准化之后的路径里的斜杠在Windows系统下是``\``，而在Linux系统下是``/``。如果想保证任何系统下都使用``/``作为路径分隔符的话，需要用``.replace(/\\/g, '/')``再替换一下标准路径。

- path.join
将传入的多个路径拼接为标准路径。该方法可避免手工拼接路径字符串的繁琐，并且能在不同系统下正确使用相应的路径分隔符。以下是一个例子：
```
 path.join('foo/', 'baz/', '../bar'); // => "foo/bar"
```

- path.extname
当我们需要根据不同文件扩展名做不同操作时，该方法就显得很好用。
```
path.extname('foo/bar.js'); // => ".js"
```

### 遍历目录
#### 递归算法

#### 遍历算法
```
          A
         / \
        B   C
       / \   \
      D   E   F
```

#### 同步遍历
```
function travel(dir, callback){
	fs.readdirSync(dir).forEach(function(file){
    	var pathname = path.join(dir, file);

        if(ts.statSync(pathname).isDirectory()){
        	travel(pathname, callback);
        } else {
        	callback(pathname)
        }
    });
}
```

#### 异步遍历
如果读取目录或读取文件状态时使用的是异步API，目录遍历函数实现起来会有些复杂，但原理完全相同。travel函数的异步版本如下。

```

function travel(dir, callback, finish){
  fs.readdir(dir, function(err, files){
    (function next(i){
      if(i < files.length){
        var pathname = path.join(dir, files[i]);

        fs.stat(pathname, function(err, stats){
          if (stats.isDirectory()){
            travel(pathname, callback, function(){
              next(i+1);
            });
          }else{
            callback(pathname, function(){
              next( i+1 );
            });
          }
        });
      }else {
        finish && finish();
      }
    })(0);
  });
}

```

### 文本编码
使用NodeJS编写前端工具时，操作得最多的是文本文件，因此也就涉及到了文件编码的处理问题。我们常用的文本编码有UTF8和GBK两种，并且UTF8文件还可能带有BOM。在读取不同编码的文本文件时，需要将文件内容转换为JS使用的UTF8编码字符串后才能正常处理。

#### BOM的移除
BOM用于标记一个文本文件使用Unicode编码，其本身是一个Unicode字符（"\uFEFF"），位于文本文件头部。在不同的Unicode编码下，BOM字符对应的二进制字节如下：
```
    Bytes      Encoding
----------------------------
    FE FF       UTF16BE
    FF FE       UTF16LE
    EF BB BF    UTF8
```

因此，我们可以根据文本文件头几个字节等于啥来判断文件是否包含BOM，以及使用哪种Unicode编码。但是，BOM字符虽然起到了标记文件编码的作用，其本身却不属于文件内容的一部分，如果读取文本文件时不去掉BOM，在某些使用场景下就会有问题。例如我们把几个JS文件合并成一个文件后，如果文件中间含有BOM字符，就会导致浏览器JS语法错误。因此，使用NodeJS读取文本文件时，一般需要去掉BOM。例如，以下代码实现了识别和去除UTF8 BOM的功能。

```
function readText(pathname){
	var bin = fs.readFileAsync(pathname);

    if(bin[0] === 0xEF && bin[1]=== 0xBB && bin[2]===0xBF){
    	bin = bin.slice[3];
    }
    return bin.toString('utf-8');
}
```

#### GBK转UTF8
NodeJS支持在读取文本文件时，或者在``Buffer``转换为字符串时指定文本编码，但遗憾的是，GBK编码不在NodeJS自身支持范围内。因此，一般我们借助``iconv-lite``这个三方包来转换编码。使用NPM下载该包后，我们可以按下边方式编写一个读取GBK文本文件的函数。
```
var iconv = require('iconv-lite');

function readGBKText(pathname) {
    var bin = fs.readFileSync(pathname);

    return iconv.decode(bin, 'gbk');
}
```

#### 单字节编码
有时候，我们无法预知需要读取的文件采用哪种编码，因此也就无法指定正确的编码。比如我们要处理的某些CSS文件中，有的用GBK编码，有的用UTF8编码。虽然可以一定程度可以根据文件的字节内容猜测出文本编码，但这里要介绍的是有些局限，但是要简单得多的一种技术。

首先我们知道，如果一个文本文件只包含英文字符，比如Hello World，那无论用GBK编码或是UTF8编码读取这个文件都是没问题的。这是因为在这些编码下，ASCII0~128范围内字符都使用相同的单字节编码。

反过来讲，即使一个文本文件中有中文等字符，如果我们需要处理的字符仅在ASCII0~128范围内，比如除了注释和字符串以外的JS代码，我们就可以统一使用单字节编码来读取文件，不用关心文件的实际编码是GBK还是UTF8。以下示例说明了这种方法。
```
1. GBK编码源文件内容：
    var foo = '中文';
2. 对应字节：
    76 61 72 20 66 6F 6F 20 3D 20 27 D6 D0 CE C4 27 3B
3. 使用单字节编码读取后得到的内容：
    var foo = '{乱码}{乱码}{乱码}{乱码}';
4. 替换内容：
    var bar = '{乱码}{乱码}{乱码}{乱码}';
5. 使用单字节编码保存后对应字节：
    76 61 72 20 62 61 72 20 3D 20 27 D6 D0 CE C4 27 3B
6. 使用GBK编码读取后得到内容：
    var bar = '中文';
```
这里的诀窍在于，不管大于0xEF的单个字节在单字节编码下被解析成什么乱码字符，使用同样的单字节编码保存这些乱码字符时，背后对应的字节保持不变。

NodeJS中自带了一种``binary``编码可以用来实现这个方法，因此在下例中，我们使用这种编码来演示上例对应的代码该怎么写。
```
function replace(pathname){
	var str = fs.readFileSync(pathname, 'binary');
    str = str.replace('foo', 'bar');
    fs.writeFileSync(pathname, str, 'binary');
}
```