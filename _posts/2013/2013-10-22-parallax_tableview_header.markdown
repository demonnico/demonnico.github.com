---

author: demon
comments: true
date: 2013-10-22 22:55:00
layout: post
slug: parallax
title: 类Path中视差效果tableHeaderView的一种实现
keywords: Parallax 知乎日报 视差 Path
description: Path中视差效果的tableHeaderView的一种实现
categories:
- iOS XCode 
tags:
- XCode
- 技巧
- iOS

---
接上篇文章中提到的类似知乎日报（类似Path2.0中的个人页面）中带前后视差效果的tableView如何实现，前段时间换了种实现方式，用起来也简单多了。具体实现的话时涉及到的技术点也无外乎Category、KVO这些，几句话就写完了。废话不多说，直接上[代码](https://github.com/demon1105/NTParallaxView)吧。:D
![](https://raw.github.com/demon1105/ImagesLib/master/demo_parallaxview.gif)
