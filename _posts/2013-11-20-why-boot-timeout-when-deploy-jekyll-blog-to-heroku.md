---
layout: post
title: "为什么部署jekyll博客到heroku后启动超时？"
description: ""
category:
tags: [heroku,jekyll]
---
{% include JB/setup %}

   本地安装jekyll，配置博客环境及部署到heroku需要的步骤，不再详细解释，
总结下自己屡次push博客到heroku后，页面错误，查看日志(heroku logs -t)总有
类似`boot timeout..60 seconds`错误的两个原因：

### \_config.xml的exclude值没有包括 _vendor_
  heroku拉取gem时会把它们放到vendor文件夹下，不exclude的话，jekyll会build
它下面的文件，错误就难免了。
### jekyll命令参数的变化
  如果jekyll的版本是1.3.0的话，**Procfile**中启动jekyll命令指定端口的参数`-p`应为`-P`。  
  jekyll升级惹得祸，没啥多说的。
