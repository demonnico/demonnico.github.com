---
author: demon
comments: true
date: 2014-09-09 11:17:00
layout: post
slug: GPUImage Leak Cocoapods
title: 你的Pod出现内存泄露了吗
categories:
- iOS
- backup
tags:
- Leak
- GPUImage
- Cocoapods
---

事情的起因是产品出了一个需求，要对Camera做实时模糊的效果。

首先我想到的是用GPUImage来做，但是在使用GPUImage滤镜的时候发现每次都会出现[内存泄露](https://github.com/demon1105/GPUImageLeakDemo)，而且几乎所有滤镜都会有[泄漏](https://github.com/BradLarson/GPUImage/issues/1681)。后来想到了CoreImage内置了模糊的滤镜，而且CoreImage也支持硬件加速，于是放弃了GPUImage的方案，用CoreImage取而代之（iOS8已经支持[自定义Kernel](https://developer.apple.com/library/prerelease/ios/releasenotes/General/WhatsNewIniOS/Articles/iOS8.html)了，感兴趣的可以移步[这里](http://holko.pl/2014/07/21/motion-blur/)）。

就在前几天在github上看到有朋友已经找到了上文中我提到的leak原因，并且提到了解决方案（不想看一大堆废话的可以直接看[这里](https://github.com/BradLarson/GPUImage/issues/1748)）。不敢私藏，写出来和大家一起分享。

我们知道GCD中的对象在6.0之前是[不参与ARC](http://stackoverflow.com/questions/12730202/do-you-need-to-release-gcd-queues-under-arc-in-ios-6-0)的，而GPUImage目前支持最低的iOS版本是4.3，并且用到了ARC，因此Brad Larson在GPUImage里，用了一些条件编译来实现在iOS6.0之前GCG对象的手动释放。

GPUImage中的条件语句如下：

{% highlight objc %}

- (void)dealloc
{
    [self destroyDataFBO];

#if ( (__IPHONE_OS_VERSION_MIN_REQUIRED < __IPHONE_6_0) || (!defined(__IPHONE_6_0)) )
    if (movieWritingQueue != NULL)
    {
        dispatch_release(movieWritingQueue);
    }
    if( audioQueue != NULL )
    {
        dispatch_release(audioQueue);
    }
    if( videoQueue != NULL )
    {
        dispatch_release(videoQueue);
    }
#endif
}
{% endhighlight %}

这里判断了当前系统支持的最小版本是否小于6.0，如果你在管理第三方库的时候没有用到Cocoapods，那应该说大多数情况下这么判断是没有问题的。但是如果用到了Cocoapods，细心的你会发现在GPUImage的pod里，compile sources的flag中多了一项`DOS_OBJECT_USE_OBJC=0`。

![](https://raw.githubusercontent.com/demon1105/ImagesLib/master/GPUImage_DOS.png)

查了一下关于`DOS_OBJECT_USE_OBJC`的解释：

![](https://raw.githubusercontent.com/demon1105/ImagesLib/master/GPUImage_OS_define.png)

我们发现恰恰是因为这一行定义，禁止了当前文件中的GCD对象参与ARC。至此，泄露的原因已经找到。事实上较好的做法是通过检查`OS_OBJECT_USE_OBJC`，因为在上述这种特定的条件下版本检测并不是完全通用。

但问题是我们还得用Cocoapods来管理所有第三方库啊！或许你可以这样：

1、设置pod时将将GPUImage的git地址改为我的[分支](https://github.com/demon1105/GPUImage)，这里我将用到的条件语句改为了推荐的`OS_OBJECT_USE_OBJC`，[检查特性而不是检查版本](https://github.com/CocoaPods/CocoaPods/issues/1001)。
	
2、偷懒的办法，fork GPUImage之后将.podspec文件中`deployment_target`设置为6.0或以上，更新一下pod。

因此，如果你的项目使用Cocoapods来管理第三方库，那就有必要稍微花点时间检查一下，确保没有出现因上述问题而造成的内存泄露。

Over.



