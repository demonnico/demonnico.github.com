---
author: demon
comments: true
date: 2012-10-29 22:18:15
layout: post
slug: cocoapods
title: CocoaPods
wordpress_id: 82
categories:
- iOS
- 开发
- 开发工具
tags:
- cocoapod
- plistlib
- xcode
---

今天快下班的时候和同事闲聊，说写model是一件很痛苦的事情，服务器返回的都是JSON的格式，客户端在解析的时候无非是根据key来得到相应的value。于是捉摸着可以写一个程序，[简单配置一下之后，可以自动生成头文件和实现文件](http://demon1105.gotoip4.com/2012/11/01/auto_create_objc-c_model/)，讨论到后来觉得这个方案可行，考虑到跨平台的问题，准备着手用python来写，找了一下python刚好有一个Model叫plistlib可以直接读取plist中的数据，想想应该很快就能实现大致的功能。

python虽然接触过一段时间，但是从来没实际操作过，于是先在google上查点资料。查着查着，也不知道怎么查到了[cocoaPods](http://cocoapods.org/)的git，看了下介绍感觉也蛮有意思的，查了下，原来是一个比较著名的在cocoa下面的项目包管理器。晚上回来之后就在自己的mac上装了一下，感觉还不错。

有了这东西，我们在开发的时候如果碰到项目中添加了什么第三方的库（比如经常用到的JSONKit,AFNetworing等等），就不需要杂七杂八一堆往项目里面拖入了。只要按照documents里面说明的设置Podfile之后,pod install，整个世界清净了···

至于文章开头的自动model工具，早被我抛到九霄云外了···今天没时间了，以后写好再补上。
