---
layout: post
title: "lsof-进程和被访问文件的查看工具"
description: ""
category: ""
tags: lsof,linux
---
{% include JB/setup %}

netstat大家都知道，可以查看当前系统开启的端口和所属进程，但以下要求netstat就力所不能及了：

* 查看某个文件或文件夹当前被哪个进程访问
* 某个进程打开了哪些文件

而这对`lsof`就小菜一碟,`lsof`在系统manual里介绍的很简洁：  
　`列出系统内打开的文件（list open files）．`  
因为在Unix/Linux下，系统设计的理念是:一切都是文件，所以`lsof`的功能不止能列出普通文件，  
还有网络链接,共享库text,内存映射文件等等.  

不过由于lsof的功能过于强大，在此不一一列举，只介绍下自己用过的几个功能:

* 根据进程的命令名称列出进程打开的所有文件:`lsof -c lsof`
* 有哪些tcp端口被哪些进程打开：`lsof -itcp`
* 查看某个文件被哪个进程打开：`lsof /tmp/werid-suffix`
* 某个文件夹下递归的所有文件都被哪些进程打开:`lsof +D /tmp`
