---
layout: post
title: "模拟窗口操作"
description: ""
category:
tags: [X,ubuntu,eOS]
---
{% include JB/setup %}

## 系统自启动程序
   每次开机后，都要照例打开emacs，firefox，termial，另外自己习惯把它们放到不同的desktop，虽然绑定了快捷键很方便，但作为linux用户怎么不折腾下？！
## 解决的办法
   首先想到找一个模拟按键输入的工具，google出`xdotool`，模拟X窗口操作的工具
   然后apt-get install,接着man，把窗口移动部分了解了，写了个脚本。

*初始版本*:
   系统启动后执行脚本，在desktop1中启动emacs，然后切到desktop2在启动firefox，最后在desktop3中启动terminal。
   但因为程序启动到窗口显示需要1、2秒的时间，所以需要在切换窗口的间隙sleep几秒，ugly..


*升级版*：
   细致阅读了xdotool的manual的其它部分后萌生的第二个方法，
   在窗口1里启动所有程序，然后分别选择窗口并将它们移到各自的desktop中，
   另外xdotool有--sync参数指定等待窗口出现和命令链的特性，最终效果很好。

## 最终的shell脚本
	#!/usr/bin/env bash

	#don‘t kill child process when this script exit
	trap "" 1

	#search window by class($1) and move them to desktop($2)
	move2Desktop() {
	xdotool search --sync --onlyvisible --class $1 \
	set_desktop_for_window $2
	}

	#`optirun` come with bumblebeed which is driver for nvidia video card.
	# normally just emacs24 is enough,below is the same
	optirun emacs24 --maximized &

	optirun firefox &
	move2Desktop firefox 1
	pantheon-terminal &
	move2Desktop pantheon-terminal 2
