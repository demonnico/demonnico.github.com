---

author: demon
comments: true
date: 2013-06-05 22:22:00
layout: post
slug: cocoalumberjack_xcodecolors
title: Colorful XCode Console
wordpress_id: 208
keywords: XCode CocoaLumberjack XcodeColors 彩色输出
description: 让你的XCode输出Colorful
categories:
- iOS 
tags:
- XCode
- 技巧
- iOS

---

前几天在[唐巧](http://blog.devtang.com/)的微信公共账号（iosDevTips）里，收到一条关于如何在开发的时候，在控制台中更友好地输出Log的消息。里面大致提到了三个方案

 * 用Emoji放到每行行首，作为区分。（From [汤圣罡](http://lextang.com/) tang sheng gang，第三个字我去查了:D）
 * 基于[CocoaLumberjack](https://github.com/robbiehanson/CocoaLumberjack)的开源方案
 * AppCode，有插件可以设置Log着色(From 黎伟)
 
我只用过第二种，而且用起来也挺方便。关于什么是CocoaLumberjack，有什么优点，我这里就不再累述了，反正谁用谁知道，想了解的直接去Github上看吧。因为看到许多朋友说尝试后都失败了，这里说说我的设置步骤。
 
首先我的项目(ProjectA)是用Cocoapods管理的（当然你可以直接把CocoaLumberjack所有的文件拖到自己的项目里，如果你不觉得这么做很麻烦的话），所以在podfile里面添加`pod 'CocoaLumberjack'`，然后pod update就可以了。
 
因为CocoaLumberjack的Color Log是依赖[XcodeColor](https://github.com/robbiehanson/XcodeColors)插件的，因此我们也必需安装这个插件。装完后重新启动一下Xcode，然后先试试XcodeColors的Demo Project,确定插件已经成功安装。
 
接下来在ProjectA里（可以是applicationDidFinishLaunchingWithOptions)设置一下日志的输出级别（可以按照级别输出，只输出警告，错误，或者全部），对lumberjack进行初始化，并且打开Color Log的开关
 
{% highlight objc %}
static const int ddLogLevel = LOG_LEVEL_VERBOSE;

// Standard lumberjack initialization
[DDLog addLogger:[DDTTYLogger sharedInstance]];

// And we also enable colors
[[DDTTYLogger sharedInstance] setColorsEnabled:YES];

{% endhighlight %}

这样的话你就可以用DDLogInfo,DDLogError等取代之前的NSLog了。默认的话DDLogError输出是红色的，DDLogWarn是黄色的（我喜欢这个颜色:D）。

到目前为止，对于我来说至少是在模拟器上调试的时候输出的Log都带颜色了，但后来我发现在真机上调试的时候输出的还是黑色的Log。后来在Wiki的[XcodeColors in iOS](https://github.com/robbiehanson/CocoaLumberjack/wiki/XcodeColors)上发现了这么一条

```
You may occasionally notice that colors don't work when you're debugging your app in the simulator. And you may also notice that you colors never work when debugging on the actual device. How do I fix it so it works everywhere, all the time?
```

我运气可能算比较好，每次在模拟器上调试的时候都正常有颜色输出。按照提示我对Project的Scheme作了如下调整：

* In Xcode bring up the Scheme Editor (Product -> Edit Scheme...)
* Select "Run" (on the left), and then the "Arguments" tab
* Add a new Environment Variable named "XcodeColors", with a value of "YES"

最后重新Cmd+R一次，搞定，我在Xcode4.4和Xcode4.6上都做过测试，都是可以的。所以在我这里目前不存在Xcode版本不一致而造成彩色日志输出失败的问题。如果有朋友按照如上的步骤设置还是不成功，欢迎在评论里留言提出疑问，一起讨论。

最后附上真相

![](https://raw.github.com/demon1105/ImagesLib/master/color_log.png)

------------------
今天微博上有位叫[开星儿](http://weibo.com/kaixinger)的朋友提到了[这篇文章](http://bluedogtech.blogspot.com/2010/12/global-log-level-control-with.html)，感兴趣的朋友可以翻过去看一下，你应该不会后悔。


