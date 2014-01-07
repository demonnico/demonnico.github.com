---

author: demon
comments: true
date: 2013-12-26 22:55:00
layout: post
slug: iOS7,barButtonItem,
title: iOS7下自定义barButtonItem后的奇怪问题
keywords: iOS barButtonItem navigationController interactivePopGestureRecognizer
description: iOS7下自定义barButtonItem后的奇怪问题
categories:
- iOS7 barButtonItem 
tags:
- interactivePopGestureRecognizer
- barButtonItem
- iOS7

---

前几天在开发的时候碰到个奇怪的问题，将initWithCustomView出来的barButtonItem对象设置到navigationBar上之后，iOS7的设备下会出现interactivePopGestureRecognizer手势pop失效的问题。网上有解决方案是设置完navigationBar的buttonItem之后，重新将navigationController中的interactivePopGestureRecognizer设置为nil，这样虽然解决了手势的问题，但是在某些操作的情况下会出现新的问题。

假设navigationController在push一个viewController的时候，迅速地通过手势进行pop操作，有时会出现navigationBar的titleView和对应的内容视图不一致的情况，有时界面会出现假死的情况（这种情况下home退出，再返回app，有一定的概率能回到正常状态），就好像navigationController中viewControllers的栈关系和导航里的内容错乱了，控制台中也会输出如下错误```nested push animation can result in corrupted navigation bar```，我随机挑选了手机中为iOS7优化过的app，其中Pinterest 3.3.1和啪啪 3.1.2都有这个类似的问题。和群里的朋友们讨论了一下，找到一个解决方案：设置interactivePopGestureRecognizer的delegate后，实现如下方法

```
- (BOOL)gestureRecognizerShouldBegin:(UIGestureRecognizer *）
```

在 pushViewController 的时候适当设置标记禁用手势，0.5s 后重置，然后在 gestureRecognizerShouldBegin: 里修改标记。这样做的思路就是在push的时候禁用手势，等动画结束后再重新开启。

但是一开始自己在用这个方法的时候发现并不是所有情况都能解决，后来发现原来是因为当初图省事，在开发的时候自己写了个BaseViewController的基类，然后override了ViewWillAppear和ViewWillDisappear，因为产品那边经常要统计进入某个界面的次数。但是由于部分的viewController继承BaseViewController:UIViewController的时候，在BaseViewController的viewDidLoad中错误地设置了navigationController.interactivePopGestureRecognizer的delegate，所以造成了刚才提到的情况。这里推荐使用MethodSwizzle替代继承的方案，不然下次不知道什么时候’基类‘又变成‘鸡肋’了···

到此为止，这个奇怪的问题算是解决了。最后想说的是，如果可能的话，在iOS7下设置barButtonItem尽量不要用initWithCustomView方法，it makes me sick...

PS：附上一个UIView的[category库](https://github.com/demon1105/UIView-Utils)，如果你是在代码里布局而不是通过IB的的话，应该用得到:D

Over.

----
附上一篇[@雨lei不是人](http://blog.sina.com.cn/yzykhq) 的[推荐方案👍](http://keighl.com/post/ios7-interactive-pop-gesture-custom-back-button/)