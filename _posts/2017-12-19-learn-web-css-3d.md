---

layout: post

title: ""

date: 2017-12-19

categories: [DOM]

---

* content
{:toc}

## 新建项目
1. 安装Express4并新建项目
```bash
 npm i -g express-generator
```
然后安装依赖，会自动创建一个工程。
2. 使用supervisor 监听文件变化
```bash
  npm i -g supervisor
```
 然后把``package.json``中的脚本命令做如下更改：
 ```json
   "scripts": {
    "start": "supervisor ./bin/www"
  },
 ```
