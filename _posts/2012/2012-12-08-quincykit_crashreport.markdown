---
author: demon
comments: true
date: 2012-12-08 12:46:19
layout: post
slug: quincykit_crashreport
title: QuincyKit的crashReport框架
wordpress_id: 116
categories:
- IOS
- skills
- 开发
- 开发工具
tags:
- CocoaLumberjack
- CrashReport
- HockyKit
- QuincyKit
- xcodeColors
---

在进行开发或者产品发布后，测试人员或者用户在进行操作的过程中难免会碰到这样或者那样的异常，有时候甚至程序直接crash，如果产品还没有发布出去，我们可以直接拿到程序的错误日志，之后可以对这些日志进行分析，找到问题所在然后改善产品质量，但如果产品已经发布出去了，可能就比较麻烦了。

之前在用HockyKit的时候发现作者还有另外一个开源的项目叫[QuincyKit](http://quincykit.net/),它就是专门针对产品发布后收集错误日志的一个开源框架，在[github](https://github.com/therealkerni/QuincyKit)上可以找到他基本的客户端demo和服务端源码，搭建起来也非常简单，一个支持php5.2以上还有Mysql的服务器就可以了。客户端部分使用了CrashReporter.framework的第三方framework,github的wiki上有详细的说明。

自己搭好之后使用了一下，确实不错。但是目前为止有一点还没搞定，就是[Remote symbolication](https://github.com/TheRealKerni/QuincyKit/wiki/Remote-symbolication)，现在拿到crashReport之后都是直接在xcode中的Organizer中解决符号化问题的，也就是[Symbolication via Xcode Organizer](https://github.com/TheRealKerni/QuincyKit/wiki/Symbolication-via-Xcode-Organizer)。

于是我又去issues里面看了看，虽然没解决我的问题，但也有其他的收获，找到了一个叫objc 的logging framework,叫[CocoaLumberjack](https://github.com/robbiehanson/CocoaLumberjack)，支持[xcodeColors](https://github.com/robbiehanson/CocoaLumberjack/wiki/XcodeColors)，用起来很简单，也很灵活。可以根据Loglevel来进行过滤，这样开发的时候控制台中我们需要的信息就清晰多了。

-------分割线-----

[解决](http://www.taofengping.com/2013/03/28/crash_again/)remote symbolicatecrash的问题
