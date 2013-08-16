---
layout: post
title: "emacs下查询单词"
description: ""
category: emacs
tags: [emacs,ubuntu,dictd,dict,dictem]
---

# 原因

emacs下浏览代码库时，方法的名称、注释中有些单词比较生癖，去查吧，得打开浏览器查询半天，最主要是程序间切换打断了思路

# 查找工具

在cygwin下，折腾过dictd和dict，当时由于能下载的词典少且配置起来繁琐，很少用到。现在ubuntu下，包管理很完善了，于是
`aptitude search dict` ，果然发现宝贝了dictd、dict外加各种词典dict-\*，更有知名的dict-jargon黑客词典。
嗯爽啊，感谢[ubuntu](http://ubuntu.com/)及[网易镜像](http://mirrors.163.com/)。哦，还有让词典可以在emacs里飞起来的
dict调用工具dictem

# 软件简单介绍

-   dictd 词典的服务器端，需要词典文件作为数据来源

-   dict 查询单词的客户端

-   dictem emacs下调用dict的工具

# 安装使用

## 安装

-   工具 `sudo aptitude install dictd dict dictem`

-   词典 `sudo aptitude install dict-jargon dict-wn`

-   词典有好多 `aptitude search dict-` 再按下TAB键，可用的词典都列出来了

## 查个试试

![dict]({{ site.url }}/images/dict.png)

## 上面是在终端下查的，emacs下配置也很简单，把如下代码加到.emacs下就ok了

    (require 'dictem)
    (setq dictem-server "localhost")
    
    ; 绑定了两个快捷键 C-c,d 单词定义、C-c,m 单词匹配
    (global-set-key "\C-cm" 'dictem-run-match)
    (global-set-key "\C-cd" 'dictem-run-define)
    ; 自定义了三个用户查新词典数据库
    (setq dictem-user-databases-alist
       '(("en-en"  . ("gcide" "wn"))
         ("en-zh" . ("xdict" "stardic"))
         ("hk". ("jargon"))))
    (setq dictem-default-database "en-en")
    (setq dictem-use-user-databases-only t)
    (define-key dictem-mode-map [tab] 'dictem-next-link)
    (define-key dictem-mode-map [(backtab)] 'dictem-previous-link)
    
    ;; 以下是查询结果显示样式的处理
    (add-hook 'dictem-postprocess-definition-hook
              'dictem-postprocess-definition-remove-header)
    
    ; For creating hyperlinks on database names
    ; and found matches.
    (add-hook 'dictem-postprocess-match-hook
              'dictem-postprocess-match)
    
    ; For highlighting the separator between the definitions found.
    ; This also creates hyperlink on database names.
    (add-hook 'dictem-postprocess-definition-hook 
              'dictem-postprocess-definition-separator)
    
    ; For creating hyperlinks in dictem buffer
    ; that contains definitions.
    (add-hook 'dictem-postprocess-definition-hook 
              'dictem-postprocess-definition-hyperlinks)
    
    ; For creating hyperlinks in dictem buffer
    ; that contains information about a database.
    (add-hook 'dictem-postprocess-show-info-hook
              'dictem-postprocess-definition-hyperlinks)

## 在某个单词(then)上按下C-c,d看看结果

![结果]({{ site.url }}/images/emacs-dictem-lookup.png)
