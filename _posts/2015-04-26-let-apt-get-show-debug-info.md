---
layout: post
title: "让apt-get打印debug信息"
description: ""
category: ""
tags: [apt-get,debug]
---
{% include JB/setup %}

Linux Mint下升级docker到1.6时,`apt-get update`获取docker的packages时报错导致拿不到最新的包的元文件.

参考了docker官方安装文档重新安装仍有同样的错误,可以定位到是和apt-get相关的问题.

通过`man apt-get`找到`man apt.conf`手册搜索`debug`,找到https相关的debug配置项添加到`/etc/apt/apt.conf`

    Debug {
	 Acquire {
	   https "true";
	 }
	}

再运行`apt-get update`,找到原因:`/etc/ssl/certs/ca-certificates.crt`格式有误导致https的包服务器无法访问.

通过`update-ca-certificates`重新生成下上面的`crt`文件,这次`apt-get update`没有错误,`apt-get install lxc-docker`成功更新docker到1.6
