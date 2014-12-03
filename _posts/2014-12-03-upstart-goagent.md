---
layout: post
title: "upstart goagent"
description: ""
category: ""
tags: [goagent upstart ubuntu]
---
{% include JB/setup %}

  双11前,入手了台戴尔笔记本,把[Linux Mint](http://linuxmint.com/) 折腾好后,就差翻山越岭,体验墙外的风光了
还好有之前申请的[goagent](https://github.com/goagent/goagent)的帐号,果断下载.

  之前通过cron开机启动,总觉得不够格,想到最近被批的[sysetmd](http://freedesktop.org/wiki/Software/systemd/)再联想到[upstart](http://upstart.ubuntu.com/),于是google,askubunu,配置了下upstart,最后的成果: **/etc/init/goagent.conf**

    author "goagent"
    description "goagent daemon"

	env PATH=/usr/bin:/bin

	start on filesystem and startup
	stop on runlevel [016]

	respawn
	respawn limit 5 1

	console none
	setuid lee
	exec python /home/lee/src/goagent/local/proxy.py >> /tmp/goagent.log 2>&1

   对就这么简单,只要把对应的文件放到**/etc/init/**下再重启即可

   想立即使服务生效的话可以

   `sudo initctl reload-configuration` 重新加载配置

   `sudo service goagent stop/start/restart` 停止/开始/重启 服务.
