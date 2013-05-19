---
author: demon
comments: true
date: 2012-10-22 22:36:08
layout: post
slug: ios_6_0_screen_undefine
title: IOS 6.0之后的屏幕问题
wordpress_id: 33
categories:
- Cocos2d
- IOS
- 开发
tags:
- Cocos2d
- IOS
- 屏幕旋转
---

之前IOS6.0发布后天真地认为siri支持所有设备，于是手贱了把自己ipad升了个级。之后却发现没有siri，差点把home键都给按爆了。悲剧的是因为IDE没有升级所以也没法调试了，只能用模拟器。用过appstore在线下载xcode的朋友应该体会过那种速度，在这里我只能为具有中国特色的网络再次感到惊叹。

装了xcode4.5.1以后运行之前的程序发现控制台会输出一句缺少xxx图片，应该是为了支持iphone5而设置的默认背景图，还有一句:
	
	Application windows are expected to have a root view controller at the end of application launch.
大致是说程序的窗口对象应该在程序加载结束前设置一个根控制器对象。进入程序后发现原来在IOS6.0里苹果把屏幕的旋转事件传递机制修改过了。以前可以直接在window上面添加一个view,现在的如果要想得到旋转事件的话就必须通过rootController,在window中设置跟控制器为rootController之后事件才会传递到rootController里面。

说到这里，以为问题都解决了。于是运行了一个cocos2d的程序，结果发现还是有类似的旋转问题。网上一搜发现已经有很多人抛出了问题的解决方案：
	
首先还是和之前说的一样，需要在
{% highlight objc %}
-(BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{% endhighlight %}
内设置根控制器对象，在这里我们可以根据自己的需要添加一下判断之后再进行处理，然后我们需要在RootViewController方法中根据需要实现另外两个委托方法
{% highlight objc %}    
- (NSUInteger) supportedInterfaceOrientations
{ 
	return UIInterfaceOrientationMaskLandscape;
}
- (BOOL) shouldAutorotate 
{
	return YES;
}
{% endhighlight %}
这里我看了一下，貌似之前的几个委托方法在6.0里面已经被放入到弃用的分类里了
> 
网上大部分的朋友解决方案到此就结束了，但是我的问题还是依旧。后来看了下发现原来程序target的summry一直都没有设置support interface orientations,设置了支持两种landscape模式之后，cmd+R.搞定·
