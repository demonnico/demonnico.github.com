---
author: demon
comments: true
date: 2012-11-01 22:53:28
layout: post
slug: auto_create_objc-c_model
title: 自动导出objc-c的Model
wordpress_id: 86
categories:
- IOS
- 开发
- 开发工具
tags:
- objc
- python
- 自动脚本
---

接上[前几天提到的](http://demon1105.gotoip4.com/2012/10/29/cocoapods/)，在开发的时候经常碰到一些重复性的劳动，比如说服务器返回的都是统一的json的格式，客户端根据key进行解析，然后创建Model,设置各个字段，写parse方法...(此处省略几百字)。

一两个Model的话应该还好，数量一多的话就很麻烦了，尤其是当Model的字段很多的时候。这里我写了一个[python的脚本](https://github.com/demon1105/AutoCreateModelScript)，用法很简单，只要按照demo里面在plist里设置相对的一些参数，就能根据预置的参数生成Model.h和Model.m文件，文件的名字还有类名都是用户自己定义的。当然还有一些不足的地方，github上的wiki也写得不是很清楚，准备等版本再完善以后逐渐补充起来。目前为止基本的一些需求应该完全可以满足了。python写得很烂，因为第一次写，就当练习基本语法了···
