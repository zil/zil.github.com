---
layout: post
title: "activerecord中的model和migration"
description: ""
category: "activerecord"
tags: [activerecord,sinatra,migration]
---
{% include JB/setup %}

  家里开着宾馆，赶上过年在家，发现用的旅馆管理系统不大完善，折腾了一番
cygwin + ruby + sinatra + activerecord
  sinatra简单易用自不必说，倒是activerecord的migration让自己吃了些苦头，
虽然主要归因于单独用activerecord，没有了rails完善的工具做支持，但也让自己
对activerecord有了进一步的认识。

## 模型的关系与外键所在位置
  自己就因为没搞太清，在调用模型的查询方法时，报`*_id`未找到的错误，总以为
是migration的事，搞到很晚，第二天参看rails guide才恍然大悟

## 关联的列无需显示指明，有add_reference
  用的add_column指定关联列`*_id`,虽然不影响model的查询关联，但add_reference
可以指定其他选项比如索引，另外看起来更直观，和model的has_对应

## 通过版本控制工具，记录每一次的migration和schema.rb
  大部分的问题，通过比较就发现了，害自己折腾半天的第一点可能就是因为没及时加入
版本控制，变化不可跟踪的原因

## 多读支持文档
  以上的问题，rails guide里都有提及。有问题时，及时参看下文档很可能就及时把
问题解决了。

## 题外话
  之前博客push到github page，一些内容更新不了，前段时间看到github page功能的
更新，顺便借本文测试下。
