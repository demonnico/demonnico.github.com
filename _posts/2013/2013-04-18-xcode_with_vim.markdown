---
author: demon
comments: true
date: 2013-04-18 21:53:46
layout: post
slug: xcode_with_vim
title: Xcode集成Vim
wordpress_id: 183
categories:
- 开发
- 开发工具
tags:
- xcode
- XVim
- 插件
---

介绍一个XCode插件，叫[XVim](https://github.com/JugglerShu/XVim)。今天一同事（汗一下，人是做android的）推荐的，之前只是偶尔傻傻地用MacVim，代码的补全确实不好用，只有一些基本语法级别补全，相对iOS下面一大推的API方法名来说根本不够用。

装好插件原本的编辑窗口就变成了下面这个样子，在Editor和Console之间多了一条VI交互窗口·

[![XVim](http://www.taofengping.com/wp-content/uploads/2013/04/Screen-Shot-2013-04-18-at-9.41.22-PM-300x159.png)](http://www.taofengping.com/wp-content/uploads/2013/04/Screen-Shot-2013-04-18-at-9.41.22-PM.png)

如果你不想用的话可以在Edit下面去掉XVim的勾选

[![XVim_select](http://www.taofengping.com/wp-content/uploads/2013/04/Screen-Shot-2013-04-18-at-9.43.35-PM-258x300.png)](http://www.taofengping.com/wp-content/uploads/2013/04/Screen-Shot-2013-04-18-at-9.43.35-PM.png)

这里对我来说有个小冲突，Vim进入交互模式是Esc，xcode的代码补全快捷方式我也习惯使用Esc(其实command+.也可以，看自己洗好了)，用了XVim之后就Esc就没截获了，可以command+,进入偏好设置，设置另外的代码不全快捷键

[![Screen Shot 2013-04-18 at 9.49.40 PM](http://www.taofengping.com/wp-content/uploads/2013/04/Screen-Shot-2013-04-18-at-9.49.40-PM-300x101.png)](http://www.taofengping.com/wp-content/uploads/2013/04/Screen-Shot-2013-04-18-at-9.49.40-PM.png)

最后附上[Xcode的主题](https://github.com/demon1105/xcode-theme)(和我的iterm差不多一个风格)，Over。
