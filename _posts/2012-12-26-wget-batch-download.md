---
layout: post
title: "wget批量下载资源"
description: ""
category: 
tags: [wget,download,ubuntu,SICP]
---

## 背景
  之前看过几集[Structure and Interpretation of Computer Programs][1]，感觉太难懂了，就放下了。后来开始使用Emacs,接触Elisp，慢慢开始了解函数式编程，感觉有看完的必要了。于是重新找到下载链接，准备download。可是用的Ubuntu,没有现成的下载工具，总不能一个个另存吧~_~,想到wget。

## wget是什么
  _web get_ **or** _get web_ 从网络获取内容的工具，通过指定url和一些选项控制，一些网络资源就手到擒来了

## 我下载视频用到的选项  
     wget -e robots=off -A mp4 -r -nd -nH -P Documents/SICP/Video \
     http://ia700401.us.archive.org/8/items/MIT_Structure_of_Computer_Programs_1986/

     -e  robots=off 忽略服务器对robots的设置(IMHO,网站对网络爬虫的控制）  
     -A  mp4 只下载mp4文件   
     -r  递归下载该url对应的路径  
     -nd 本地不创建目录(host之后的路径8/items/MIT_Structure_of_Computer_Programs_1986/)  
     -nH 不创建url层级目录(/ia700401.us.archive.org/..1986/),太长了   
     -P  Documents/SICP/Video 把资源下载到该目录  
     最后就是视频所在的url了


[1]: http://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-001-structure-and-interpretation-of-computer-programs-spring-2005/index.htm
