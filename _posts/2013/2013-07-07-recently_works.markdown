---

author: demon
comments: true
date: 2013-07-07 12:55:00
layout: post
slug: recently_works
title: 最近的一些整理
keywords: XCode5 自动注释 Parallax 知乎日报 视差
description: 最近的知识点总结
categories:
- iOS XCode 
tags:
- XCode
- 技巧
- iOS

---
这段时间公司的事情很多，所以没有及时更新。

周五快下班的时候有同事问起类似知乎日报（类似Path2.0中的个人页面）中带前后视差效果的tableView如何实现，昨天花了点时间写了个[Demo](https://github.com/demon1105/NTParallaxScrollView)，这里给大家奉上。细心的朋友应该可以看得出来，知乎日报的实现方式和我写的其实略有不同。顶部的导航部分他们应该是添加了手势去做，所以拖动的时候没有类似ScrollView的分页效果。这里我参照了国外一个朋友实现的思路简单封装了一下，所以提供的只是一种实现的思路，如果有更好的方法也希望大家能够告诉我。

还有一个想和大家分享的和XCode5有关，之前在@onevcat的blog上看到一篇介绍XCode5特点的文章（不过因为NDA的关系，目前这几篇文章都暂时看不到了，不过可以借助google大神），其中有一点我这里想提的是，类似如下的注释：

{% highlight objc %}
/**
 * Setup a recorder for a specified file path. After setting it, you can use the other control method to control the shared recorder.
 *
 * @param talkingPath An NSString indicates in which path the recording should be created
 * @returns YES if recorder setup correctly, NO if there is an error
 */
- (BOOL)recordWithFilePath:(NSString *)talkingPath;
{% endhighlight %}

在XCode5中将直接被检测到并集成进代码提示中，并且在Quick Help中也会有相关的提示。文章提到，可用的标识符除了上面的```@param``` 和 ```@return``` 外，还有例如``` @see``` ，``` @discussion ```等，关于Javadoc的更多格式规则，可以参考 [Wiki](http://en.wikipedia.org/wiki/Javadoc)。

但是真正的高潮还没到，类似如上的注释如果一个一个去敲，岂不是太反人类了？所以这里奉上一个自动在[XCode中添加注释的脚本还有设置方法](http://blog.chukong-inc.com/index.php/2012/05/16/xcode4_fast_doxygen/)，等XCode5发布会再结合这个脚本，写注释应该会变成一件很爽的事情。所以也希望大家以后养成勤写注释，写好注释的好习惯:D

------------------
还有如下的一些文章，感兴趣的朋友也可以学习一下

破船翻译的iOS汇编教程
[ARM--1](http://beyondvincent.com/2013/06/19/ios%E6%B1%87%E7%BC%96%E6%95%99%E7%A8%8B%EF%BC%9Aarm/)
[ARM--2](http://beyondvincent.com/2013/06/20/ios%E6%B1%87%E7%BC%96%E6%95%99%E7%A8%8B%EF%BC%9Aarm2/)

唐巧-[支付宝插件机制分析](http://blog.devtang.com/blog/2013/06/23/alipay-plugin-mechanism/)

Allen-[浅析支付宝钱包插件](http://imallen.com/blog/2013/06/26/inside-alipay-plugin.html)

