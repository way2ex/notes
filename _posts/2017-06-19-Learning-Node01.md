---
layout: post
date: 2017-06-19
title: Node学习
categories: [Node]
tags: [Node]

---

## 基础
### 权限问题
在Linux系统下，想要监听80或443端口，需要有root权限，有两种方式：
1. 使用``sudo``命令运行NodeJS
2. 使用``chmod +s``命令让NodeJS总是以root权限运行，这种方式让任何JS脚本都有了root权限，不安全
 ```
 $ sudo chmod +s /usr/local/bin/node
 ```

### 模块
编写稍大一点的程序时，一般会将代码模块化。一般将代码合理拆分到不同JS文件中，每一个文件就是一个模块，而文件路径就是模块名。

#### require
``require``函数用于在当前模块中加载其他模块。可以省略``.js``扩展名。

#### exports
``exports``对象是当前模块的导出对象。与``require``函数导入的对象是对应的。
```
exports.hello = function() {
	console.log('Hello World!');
}
```

#### module
通过``module``对象可以访问到当前模块的一些相关信息，但最多的用途是替换当前模块的导出对象。例如模块导出对象默认是一个普通对象，如果想改成一个函数的话，可以使用以下方式。
```
module.exports = function(){
	console.log("Hello World!");
}
```

#### 模块初始化
一个模块的代码仅在第一次被使用时执行一次，并在执行过程中初始化模块的导出对象。之后，缓存起来的导出对象被重复利用。

## 代码的组织与部署
### 模块路径解析规则
``require``函数支持斜杠（``/``）或盘符（``C:``）开头的绝对路径，也支持``./``开头的相对路径,但这两种路径在模块之间建立了强耦合关系，一旦某个模块文件的存放位置需要变更，使用该模块的其它模块的代码也需要跟着调整，变得牵一发动全身。因此，``require``函数支持第三种形式的路径，写法类似于``foo/bar``，并依次按照以下规则解析路径，直到找到模块位置。

1. 内置模块
 如果传递给``require``函数的是NodeJS内置模块的名称，直接返回模块的导出对象。

2. node_modules目录
 NodeJS定义了一个特殊的``node_modules``目录用于存放模块。例如某个模块的绝对路径是``/home/user/hello.js``，在该模块中使用``require('foo/bar')``方式加载模块时，则NodeJS依次尝试使用以下路径。
 ```
  /home/user/node_modules/foo/bar
 /home/node_modules/foo/bar
 /node_modules/foo/bar
 ```

3. NODE_PATH环境变量
 与PATH环境变量雷系，NodeJS允许通过NODE_PATH环境变量来制定额外的模块搜索路径。NODE_PATH环境变量中包含一到多个目录路径，路径之间在Linux下使用``:``分割，在Windows下使用``;``分割。

### 包(package)
我们已经知道了JS模块的基本单位是单个JS文件，但复杂的模块往往由多个子模块组成。为了便于管理和使用，我们可以把由多个子模块组成的大模块叫做``包``，并把所有的子模块放在同一个目录里。

在组成一个包的所有子模块中，需要有一个入口模块，入口模块的导出对象被称为包的导出对象。
```
- /home/user/lib/
	- cat/
		head.js
        body.js
        main.js
```
其中，``cat``目录定义了一个包，其中包含了3个子模块。``main.js``作为入口模块，其内容如下：
```
var head = require('./head');
var bodu = require('./body');

exports.create = function(name){
	return {
    	name: name,
        head: head.create(),
        body: body.create()
    };
}
```
在其他模块中使用``require('/home/user/lib/cat/main')``就可以导入该包导出的对象。

#### index.js
如果模块的文件名是``index.js``，加载模块是可以使用模块的所在目录的路径代替模块文件路径。
```
var cat = require('/home/userlib/cat');
var cat = require('/home/userlib/cat/index');  // 等价
```

#### package.json
如果想自定义入口模块的文件名和存放位置，就需要在包目录下包含一个``package.json``，并指定入口模块的路径。
```
- /home/user/lib/
	- cat/
		+ doc/
		- lib/
			head.js
            body.js
            main.js
        + tests/
        package.json
```
其中``package.json``内容如下。
```
{
	"name": "cat",
    "main": "./lib/main.js"
}
```
如此一来，就同样可以使用``require('/home/user/lib/cat')``的方式加载模块。NodeJS会根据包目录下的``package.json``找到入口模块所在位置。

### 命令行程序
使用NodeJS编写的东西，要么是一个包，要么是一个命令行程序，而前者最终也会用于开发后者。因此我们在部署代码时需要一些技巧，让用户觉得自己是在使用一个命令行程序。

例如我们用NodeJS写了个程序，可以把命令行参数原样打印出来。该程序很简单，在主模块内实现了所有功能。并且写好后，我们把该程序部署在/home/user/bin/node-echo.js这个位置。为了在任何目录下都能运行该程序，我们需要使用以下终端命令。
```
$ node /home/user/bin/node-echo.js Hello World
Hello World
```

这种使用方式看起来不怎么像是一个命令行程序，下边的才是我们期望的方式。
```
$ node-echo Hello World
```
#### Linux
在Linux系统下，我们可以把JS文件当作shell脚本来运行，从而达到上述目的：
1. 在shell脚本中，剋通过``#!``注释来制定当前脚本使用的解析器。所以我们首先在``node-echo.js``文件顶部增加以下一行注释，表明当前脚本使用NodeJS解析。
 ```
 #! /usr/bin/env node
 ```

2. 然后我们使用以下命令赋予``node-echo.js``文件执行权限。
 ```
 $ chmod +x /home/user/bin/node-echo.js
 ```

3. 最后，我们在PATH环境变量中指定的某个目录下，例如在```/usr/local/bin```下边创建一个软链文件，文件名与我们希望使用的终端命令同名，命令如下：
 ```
  $ sudo ln -s /home/user/bin/node-echo.js /usr/local/bin/node-echo
 ```

 这样处理后，我们就可以在任何目录下使用node-echo命令了。

#### Windows
在Windows系统下的做法完全不同，得靠``.cmd``文件来解决问题。假设``node-echo.js``存放在``C:\Users\user\bin``目录，并且该目录已经添加到PATH环境变量里了。接下来需要在该目录下新建一个名为``node-echo.cmd``的文件，文件内容如下：
```
@node "C:\User\user\bin\node-echo.js" %*
```

这样处理后，我们就可以在任何目录下使用node-echo命令了。

#### 工程目录
了解了以上知识后，现在我们可以来完整地规划一个工程目录了。以编写一个命令行程序为例，一般我们会同时提供命令行模式和API模式两种使用方式，并且我们会借助三方包来编写代码。除了代码外，一个完整的程序也应该有自己的文档和测试用例。因此，一个标准的工程目录都看起来像下边这样。
```
- /home/user/workspace/node-echo/   # 工程目录
    - bin/                          # 存放命令行相关代码
        node-echo
    + doc/                          # 存放文档
    - lib/                          # 存放API相关代码
        echo.js
    - node_modules/                 # 存放三方包
        + argv/
    + tests/                        # 存放测试用例
    package.json                    # 元数据文件
    README.md                       # 说明文件
```
部分文件内内容如下：
```
/* bin/node-echo */
var argv = require('argv'),
    echo = require('../lib/echo');
console.log(echo(argv.join(' ')));

/* lib/echo.js */
module.exports = function (message) {
    return message;
};

/* package.json */
{
    "name": "node-echo",
    "main": "./lib/echo.js"
}
```