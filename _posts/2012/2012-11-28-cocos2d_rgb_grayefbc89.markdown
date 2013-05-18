---
author: demon
comments: true
date: 2012-11-28 23:24:50
layout: post
slug: cocos2d_rgb_gray%ef%bc%89
title: cocos2d中实现黑白动画（RGB-> Grayscale）
wordpress_id: 107
categories:
- Cocos2d
- IOS
- 开发
tags:
- Cocos2d
- grayscale
- opengl
- 动态纹理
---

在做Hero项目战斗模块的时候碰到一个需求，说要实现一个Sprite动画，效果是从全彩慢慢变到黑白（灰度图）。刚开始以为用TintTo就能解决了，用了一下之后发现其实Tint其实是一个着色的过程，并不能达到自己要的效果。

回头想了想，感觉好像这个效果做起来还有点麻烦，因为首先我们Sprite的纹理位图，有RGBA四个通道。但是灰度图是单通道的，蛋疼····没办法，还是得求助google大神，然后看到在[wiki上关于灰度图的说明](http://en.wikipedia.org/wiki/Grayscale)，里面有一章提到了converting RGB to grayscale，文章大致浏览了一下，关键用到了这个公式

![Y' =  0.299 R + 0.587 G + 0.114 B](http://upload.wikimedia.org/math/0/d/9/0d94a32c1f2f53a4ce8cb2cd564b6627.png)

自己在UIKit里随便找了一张图试了一下，果然，把每个像素的RBG值进行换算后得到的Y后进行赋值，得到了一张看似灰度的rgb图。

接下来的事情相对就好办了，要得到动画的话我们只要在RGB->finalRGB之间做差值运算就可以了。只要得到CCSprite在openGL里的纹理信息，然后动态改变它就能打到我要的效果了。

废话说完。写了一个[demo](https://github.com/demon1105/TextureMotify)，已上传·
