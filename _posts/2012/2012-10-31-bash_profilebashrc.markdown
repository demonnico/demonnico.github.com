---
author: demon
comments: true
date: 2012-10-31 13:05:32
layout: post
slug: bash_profilebashrc
title: .bash_profile和bashrc的区别
wordpress_id: 84
categories:
- skills
- 开发
tags:
- .bashrc
- .bash_profile
- terminal
- 自动化
---

今天写了一个xcode打包后自动上传ipa到svn服务器的脚本，一下子感觉自动化+命令行才是王道。于是想平时的一些操作用图形化的界面虽然看上去人性化了，是不是有点过于繁琐，那就试试简洁的。

首先我想把本地的一个字典程序换成command line的，google了一下有个朋友已经写好了一个[基于php的程序](http://code.google.com/p/chinese-dictionary/wiki/install)，这里我按照他的步骤安装了一下，发现在命令行里面输入dict的时候提示command not found.我猜测应该是.bashrc没有加载的缘故，之前只用过.bash_profile来设置一些环境变量，而.bashrc是我刚刚vi新建的。

于是继续google,查询了一下[bash_profile和bashrc的一些区别](http://linux.chinaunix.net/doc/system/2005-02-03/1084.shtml)
	
	* <~/.bash_profile>每个用户都可使用该文件输入专用于自己使用的shell信息,当用户登录时,该文件**仅仅执行一次**!默认情况下,他设置一些环境变量,执行用户的.bashrc文件.
	* <~/.bashrc>该文件包含专用于你的bash shell的bash信息,当登录时以及每次打开新的shell时,该该文件被读取
	* <~/.bash_logout>当每次退出系统(退出bash shell)时,执行该文件.


于是我尝试command+n一个新的Terminal，还是没有加载dict命令,然后我又找到了[这篇文章](http://coder.aqualuna.me/2012/03/bashrc-in-mac-terminal-os-x-lion.html)，其中引用到一段话:

> 
> ### [Mac OS X — an exception](http://www.joshstaiger.org/archives/2005/07/bash_profile_vs.html)
> 
> 
[An exception to the terminal window guidelines is Mac OS X’s Terminal.app, which runs a login shell by default for each new terminal window, calling `.bash_profile` instead of `.bashrc`. Other GUI terminal emulators may do the same, but most tend not to.](http://www.joshstaiger.org/archives/2005/07/bash_profile_vs.html)


也就是说在mac是一朵奇葩，他每次都会重新加载.bash_profile中的设置，默认的时候并不会加载.bashrc，按照文章中说的，我在.bash_profile中加了一个判断:

>     
>     if [ -f ~/.bashrc ]; then
>        source ~/.bashrc
>     fi
>  

最后重新command+n一个新的Terminal,搞定～
