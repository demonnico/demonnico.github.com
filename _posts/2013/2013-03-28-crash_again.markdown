---

author: demon
comments: true
date: 2013-03-28 22:51:54
layout: post
slug: crash_again
title: 炒冷饭，再提一次QuincyKit
wordpress_id: 169
categories:
- skills
- 开发
- 开发工具
tags:
- QuincyKit
- remote symbolicatecrash

---

之前在[这篇](http://www.taofengping.com/2012/12/08/quincykit_crashreport/)里面已经提到过一次QuincyKit。换了新公司，第一件事情就是让我整理一下这个框架，写写流程结构图什么的，因为之前已经搭过好几次，所以搞起来还算顺手。这里顺带做个总结。

### 1、什么是QuincyKit?

QuincyKit([http://quincykit.net](http://quincykit.net/)) 是一个开源的，针对iOS和Mac OS X系统的实时crashReport管理系统

###2、为什么用QuincyKit?

或许有人会问，why QuincyKit?why not CrashReport in itunesconnect? 请问：在你的itunesconnect中曾经看到过几次CrashReport?sometime?few?or never? 就我个人来说的话，从来没有在自己的itunesconnect账户中看到过crashReport。难道是自己的App本身就没有出现过crash吗？我想不是的。 找到一篇文章([http://www.techmemo.com.tw/wp/2011/02/23/553/iOS-how-to-obtain-the-users-crash-report/),](http://www.techmemo.com.tw/wp/2011/02/23/553/iOS-how-to-obtain-the-users-crash-report/%29,) 文章提到， 在itunesconnect中的日志，并不是无条件自动通过iOS设备把crashlog提交到Apple。当用户的iOS设备插上电脑，itunes会询问用户：是否要协助Apple改进它的产品质量？问题就出在这里，如果选择了NO，crashLog就不会提交到itunesconnect。并且这以后itunes也不会再询问你的意见， 所以itunesconnect就拿不到crashlog了。

###3、QuincyKit的特点

	A:自动发送crashlog到服务端数据库（可选后台自动发布或用户选择性提交）
	B:提供简单的管理界面访问数据库
	C:提供脚本解析批量解析crashLog（需要在包含对应.dSYM文件的Mac端运行）

###4、QuincyKit工作的大致的流程

客户端每次运行时检查当前App的crash路径
	
	[[NSString stringWithFormat:@"%@", [[paths objectAtIndex:0]                       stringByAppendingPathComponent:@"/crashes/"]])
下的日志文件，如果有新的未提交的crashlog,则post到开发者自己的服务器,写入数据库（未经过symbolicateCrash）。服务端可通过web管理界面查看对应程序(通过bundle区分，同一个App可根据version区分)的crash日志。如果要在web界面直接呈现symbolicateCrash之后的信息，则需要web管理界面开启相应App的symbolicate选项，然后在Mac端（有xcode clt，需atos命令）上运行symbolicate.php。他会将上文中提到数据库中的crash日志抓取到本地后尝试用atos命令对地址进行解释，完成后将结果再写入数据库，这样我们重新在web端再看到的crashlog将会是处理过之后的内容。

PS:

顺带解决了之前提到的remote symbolicatecrash的问题，原来是之前在写Host地址的时候带上了子路径，看了symbolicate.php中的源码后发现在config里只能是[纯粹的主机地址](https://github.com/TheRealKerni/QuincyKit/issues/127)（因为丫很单纯地在host后面加了个:端口）····

搞定。
