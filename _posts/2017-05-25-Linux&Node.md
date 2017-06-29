---

layout: post

title: "Linux和Node知识"

date: 2017-05-25

categories: [MyNotes]


---

[TOC]



#### linux命令

- ``cd ~``: 进入当前用户的目录，比如当前用户是vac,则进入/home/vac

- ``cd /``: 进入根目录  ，例如linux系统进入到 /

- ``pwd``: 输出当前目录的全路径


##### linux下载程序

下载的程序统一保存在``/usr/local/src``中



#### vim操作

- ``i`` : 进入编辑模式

- ``esc`` : 退出编辑模式

- ``:wq`` : 保存并退出

#### vim安装插件管理器Vundle不成功
在``~``运行命令
```
mv .vimrc .vimrc.backup
or
rm .vimrc
rm -rf .vim
git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim
```
然后在``.vimrc``文件输入
```
set nocompatible
filetype off
set rtp+=~/.vim/bundle/Vundle.vim
call vundle#begin()
Plugin 'VundleVim/Vundle.vim'
call vundle#end()
filetype plugin indent on
```


#### 阿里云操作

- ``yum install vim`` : 安装vim

- `` vim example.js`` : 用vim打开文件



#### 查看公网IP

在linux终端提示符下，输入以下命令：

curl members.3322.org/dyndns/getip



#### 阿里云配置外网访问

https://help.aliyun.com/document_detail/25475.html?spm=5176.2020520101.121.2.eKLyed#allowHttp

#### 安装nvm
[here](https://github.com/creationix/nvm/blob/master/README.md#install-script)
先输入
```
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.2/install.sh | bash
```
然后
```
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh" # This loads nvm
```

#### nrm
nrm 是一个管理 npm 源的工具，，用来切换官方 npm 源和国内的 npm 源（如: cnpm），当然也可以用来切换官方 npm 源和公司私有 npm 源。
- 查看当前 nrm 内置的几个 npm 源的地址：
```
nrm ls
```
- 切换到 cnpm：
```
nrm use cnpm
```

### Node命令行程序
新建文件``hello.js``，在首行写入
```
#!/usr/bin/env node
console.log('Hello World!')
```
然后修改权限
```
chmod 755 hello.js
```
执行
```
./hello.js
```
新建``package.json``
```
{
  "name": "hello-world",
  "version": "1.0.0",
  "bin": {
      "runhello": "hello.js"
  }
}
```
将当前目录模块安装到全局，一定要有”version”才可以
```
sudo npm install . -g
```
也可以执行``npm link``命令添加全局的``symbolic link``
```
sudo npm link
```
这时，在任何目录下运行``runhello``都可以。
删除
```
sudo npm uninstall hello-world -g
```



