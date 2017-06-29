---
layout: post
title: "06-03笔记-Linux学习"
date: 2017-05-25
categories: [MyNotes]

---

## 在Linux中安装软件
- 安装git
 ```apt-get install git``
- 下载安装atom
 - 在Ubuntu系统下，下载官网的.deb文件
  ``wget  https://github.com/atom/atom/releases/download/v1.8.0/atom-amd64.deb``
  最好加上sudo，不然会有权限问题
 - 解压包并安装
  ``sudo dpkg --install atom-amd64.deb``
  出错了--
  因为有一些依赖包没有安装，使用以下命令
  ``sudo apt-get -f install``
- 另一种方法安装
 ```
 sudo apt-get install software-properties-common
sudo apt-add-repository ppa:webupd8team/atom
sudo apt-get update
sudo apt-get install atom
 ```
- 运行atom
安装后名字为atom-beta,所以运行命令``sudo atom-beta``
出错！
```
Vinux@DESKTOP-SGIPJ0K:/usr/lib$ /usr/share/atom-beta/atom: error while loading shared libraries: libXss.so.1: cannot open shared object file: No such file or directory
```
网上搜到解决办法
``sudo apt-get install libxscrnsaver``
出错！
```
E: Unable to locate package libxscrnsaver
```
百度到说是软件源未更新,解决办法:``sudo apt-get update``,还是不行
再百度：更换软件源
```
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak #备份
sudo vim /etc/apt/sources.list #修改
sudo apt-get update #更新列表
```
修改为阿里云源
```
deb http://mirrors.aliyun.com/ubuntu/ trusty main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ trusty-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ trusty-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ trusty-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ trusty-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ trusty main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ trusty-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ trusty-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ trusty-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ trusty-backports main restricted universe multiverse
```
最后安装了libxss1，终于换了另一个错误。。。
``sudo apt-get install libxss1``
错误:
```
Vinux@DESKTOP-SGIPJ0K:/etc/apt$ /usr/bin/atom-beta: line 130:  6656 Aborted                 (core dumped) nohup "$ATOM_PATH" --executed-from="$(pwd)" --pid=$$ "$@" > "$ATOM_HOME/nohup.out" 2>&1
[6656:0603/182115:FATAL:render_sandbox_host_linux.cc(40)] Check failed: 0 == shutdown(renderer_socket_, SHUT_RD). shutdown: Invalid argument
#0 0x000001e09b2e <unknown>
#1 0x000001e1f73b <unknown>
#2 0x000001e1fcfd <unknown>
#3 0x000002893352 <unknown>
#4 0x00000265e7f9 <unknown>
#5 0x000002664dbf <unknown>
#6 0x00000265de96 <unknown>
#7 0x000001204397 <unknown>
#8 0x000001202e70 <unknown>
#9 0x0000033a9803 main
#10 0x7f7ede701f45 __libc_start_main
#11 0x000000575279 <unknown>
```
放弃了！！

## 查看是否安装包
- rpm包安装
查看所有
```
 rpm -qa
```
查找特定包
```
 rpm -qa|grep packagename
```
- deb包安装
```
 dpkg -l|grep packagename
```

## vim中安装emmet插件
[here](https://askubuntu.com/questions/745318/how-to-install-emmet-or-any-web-development-plugins-for-vim-editor)