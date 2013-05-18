---
author: demon
comments: true
date: 2012-11-08 21:01:01
layout: post
slug: texturepacker_for_uiimage
title: TexturePacker纹理图对UIImage的支持
wordpress_id: 94
categories:
- Cocos2d
- IOS
- 开发
- 开发工具
tags:
- TextureManager
- texturepacker
- UIImage
- UIKit
---

之前在做HEROX项目的时候，想到过一个问题。项目如果是cocos2d+UIKit混合的模式，碰到一些图片如果要共用的话就会有点麻烦。因为Cocos2d一般会用textruePacker或者其他一些软件将一些资源图进行打包，合并到一张图上，而且在这个过程中有可能会对图片进行旋转和切边(trim)，如果单纯试用UIImage的切图方法是不行的。这些天晚上回去抽空写了一个方法可以先对TexturePacker中发布出的材质图进行重新解析，然后还原成我们需要的原图，统一放到内存中。要用的时候根据原图的名字进行读取就行了。

项目里面还有点小问题，在TextureManager中有比较详细的注释，因为对CoreGraphics框架不是非常熟悉，一时也没找到原因。项目先传到[github](https://github.com/demon1105/TextureCocoa)上。

但是目前有个内存方面的小问题（没有内存泄露，但运行的时候控制台会输出对象释放提前什么的警告：incorrect checksum for freed object - object was probably modified after being freed），我这里只是为了实现功能去做这个模块，效率方面暂时也没有考虑，之后有时间再去修改完善。
