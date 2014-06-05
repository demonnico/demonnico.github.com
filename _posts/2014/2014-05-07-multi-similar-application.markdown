---
author: demon
comments: true
date: 2014-05-07 19:55:00
layout: post
slug: iOS7.1 架构 cocoapods multipleTarget 
title: 使用cocoapods创建多个类似工程的尝试
categories:
- iOS
- backup
tags:
- iOS7.1
- target
- cocoapods
---
因为业务的需要，最近做了一批类似[无聊图](https://itunes.apple.com/cn/app/wu-liao-tu-da-fa-wu-liao-shi/id687237182)的App，应用本身没有复杂的逻辑，所有类“无聊图”的应用（以下称为S应用（similar App））都针对特定的用户群，设计了不同界面风格的，比如适合宅男的女神应用，适合吃货的美食应用等等诸如此类，大概一共七八个。

首先我想到的办法是创建多个不同的target(`多target`)，分别在target里设置应用的特定资源（图片、类文件等等）。当然，首先我们要把公用的模块和资源抽取出来，以便在多个target之间共享。其余各target之间互斥的内容我们可以设置一个Config（配置中心）来做。假设我们有A、B、C三个应用，那么我们就要在Config中对应A、B、C填入不同的信息。这里可以是我们App的颜色主题，也可以是应用中所包含第三方平台（微博，微信等）分别分发给A、B、C的appKey信息。所以不拘一格，配置中心可以是一个简单的头文件，也可以是.plist文件（出于安全考虑，不要把一些关键的信息写到plist中。因为一般来说.plist文件会被Copy到`Bundle Resources`中，所以信息会直接暴露在X.app目录下）。

上面这个思路我在之前的公司做一款多语言包的游戏的时候用过，因为游戏中会涉及不同的资源包（主要是图片），创建多个target并且配合宏来做的话还算方便。这个思路具体的实现过程可以参照唐巧的[这篇文章](http://blog.devtang.com/blog/2013/10/17/the-tech-detail-of-ape-client-1/)，写得很详细。

但是当我准备创建第四个target的时候就觉得有些烦躁了。因为在创建的时候很多过程都需要用鼠标点来点去。比如在copy一个target的时候你需要反选不需要出现在这个target中的资源文件，你需要用黑色的小箭头（鼠标指针）拖动新的图片到工程里，然后在松开的时候选择正确的target。如果手抖选错了那就得重新一张张图片去和target建立reference。做完这些工作之后我们还需要去scheme里修改target名称，否则默认的名称会以`source Target->source Target copy`的样式来呈现，第n个就是`source Target->source Target copy copy`。这样的名字在项目里当然是不允许的。总得来说，还是觉得这个`多targets`的解决方案里重复的劳动太多，难道办法解决这个问题了吗？当然有。

[cocoapods](cocoapods.org)对大家来说应该不算陌生了，于是我想到了用pod来替换target，每次切换target的行为转换为替换不同的pod。这样我们创建新项目的时候就可以不用鼠标来回点，取而代之的将是建立新的pod，替换pod中不同的资源后放在各个独立的git repo下（如果你喜欢用鼠标操作，亦或者觉得pod每次都要去update很麻烦。那很遗憾，这个办法可能不适合你）。这个方案我暂且称为`多pod`。

接下来我将详细介绍这个方法。

首先我们来看一下这个私有的Pod里包含了哪些内容：

![](https://raw.githubusercontent.com/demon1105/ImagesLib/master/resources.png)

	·图片资源文件（Icon、启动图等等）
	·plist配置文件（可以是用来填写配置信息的文件，也可以是Info.plist等）
	·头文件和一些.m实现文件（按照业务来，如果没有必要也可以不包含）
	
这里我把整个项目工程成为X，因为我之前已经将X中公用的模块理好了，所以创建新的工程的话就剩下如下这些操作：
	
	1、添加A的启动图和Icon
	2、添加预编译宏，以便程序在运行时知道当前到底是哪个App，从而实现一些差异化的代码。
	3、一些程序中特有独立界面模块，逻辑的处理
	4、替换Info.plist（填写Bundle display name，版本信息，第三方平台分发的一些需要填入到URL Types中的URL Scheme信息）
	
这里先分享我的podspec信息，看完后大家可能就已经明白一大半了。

	spec.source_files = 'classes/*.{h,m}'
    spec.resource = 'resource/*'
  	spec.xcconfig = {"GCC_PREPROCESSOR_DEFINITIONS" => '$(inherited) APP_A'}

启动图的命名为[Default.png、Default@2x.png、Default-568h@2x.png](https://developer.apple.com/library/ios/documentation/iphone/conceptual/iphoneosprogrammingguide/App-RelatedResources/App-RelatedResources.html)，icon的命名规范[参照这里](https://developer.apple.com/LIBRARY/IOS/qa/qa1686/_index.html)和[这里](http://stackoverflow.com/questions/18663013/icon-file-names-ios-7)，这些文件全都可以放在resource的。这样的话我们就无须在project里设置项目对启动图和icon的关联了。只要你的Bundle下存在这些图片，App在安装后自动选择最适合的icon，启动时会自动选取适合的启动图。关于在cocoapods里添加预编译参数可以[参考这个issue](https://github.com/CocoaPods/Specs/issues/103)。 这样的话，上面提到的1、2疑问就解决了。如果你的程序里有特殊的逻辑需要处理，那么也可以创建需要的类文件。然后在项目X中根据sepc.xcconfig中的预编译参数APP_A来判断：
	
	#ifdef APP_A
	//这里是你的一些特定的逻辑，爱干嘛干嘛吧
	#endif
所以问题3也就迎刃而解了。

因为我们已经把Info.plist也放在了Pod中（如果你切换了私有Pod的git repo指向后进行update，就相当于我们换了一个target），所以只要在X的工程设置里指定一次`Info.plist`路径就可以了。

![](https://raw.githubusercontent.com/demon1105/ImagesLib/master/plist-setting.png)

咯里啰嗦说了一大堆，自己实现的过程中为了偷懒写了一个自动创建App中需要[各种尺寸icon的小脚本](https://gist.github.com/demon1105/8b3778a092150b30c3fe) ，还写了一个[模板CommonPod](https://github.com/demon1105/common-pod)
	
从现在开始，如果产品经理说需要新开发一个App，开发要做的就是clone CommonPod为一个新的Pod，替换资源，然后push到新的remote repo之后将更改项目的Podfile，pod update一次。是不是轻松些了呢？

![](https://raw.githubusercontent.com/demon1105/ImagesLib/master/pod-setting.png)

如果还不明白的的话就看这个[Demo](https://github.com/demon1105/demoProject)吧。

对比一下以上的两个方案

	·在项目配置的过程中，多target的方案需要做一些鼠标选择的操作。多pod方案只需要替换pod中的资源（命名都是一致的）。	
	·在配置结束后的调试阶段，多target方案选择不同的target之后run即可，多pod方案则需要更改podfile后update（每次pod update觉得慢的同学可以试试--no-repo-update参数）。
	
~~最后补充一个多pod方案不足的地方，因为切换pod之后重新run相当于只是修改了`Info-common.plist`的`Bundle identifier`，在模拟器上调试的时候会出现Xcode无法直接进入调试模式的问题。状况如下：~~

![](https://raw.githubusercontent.com/demon1105/ImagesLib/master/bug-xcode.png)

~~同样的情况会发生在新建Project运行之后，更改`Info.plist`的`Bundle identifier`，然后再run一次。解决这个bug的方法是在下一次run之前先退出模拟器就可以了。但是如果在真机上进行调试，everything gonna be ok。~~
####（当当当！该问题在Xcode6.0上面已经完美解决）

总结一下，多target方案麻烦在项目的设置上，~~多pod方案麻烦在模拟器的调试阶段上（上面提到的bug），其实这两个办法都有些美中不足。~~写这篇文章的目的也并不是比个孰优孰劣，只是提供了一个新的思路去完成开篇时提到的业务需求。

~~PS：如果在多pod方案中的bug有可以解决的办法，请告诉我。谢谢~~

Over



