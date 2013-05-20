---
author: demon
comments: true
date: 2013-01-07 20:52:28
layout: post
slug: uiscrollview-touchevent_something
title: UIScrollView touchEvent的一些认识
wordpress_id: 128
categories:
- IOS
- 开发
tags:
- touchEvent
- UIControl
- UIScrollView
---

之前在做东西的时候用到了UIScrollView，只是单纯地将它作为一个可滚动的容器来使用，并没有对他以及子视图的touch事件处理进行过了解。其实很好奇如果在UIScrollView的子子视图上要处理自身的touchEvent的话，它是怎么工作的呢？

一个简单的例子，假如我们在UIScrollView上添加的子视图需要相应自己的事件，假如是一个UIButton，在点击的时候的确也能够相应的动作，并且UIScrollView的滚动事件也能正常响应，但是如果细心观察的话我们可以发现，button的事件并不立即做出相应的，有一个很短的延迟。

而这时候如果我们的UIButton设置了高亮状态，点击后实现的界面跳转的动作，注册的事件是touchUpInside，那就可能没法看到高亮后的按钮就直接执行界面跳转了。

造成事件响应的延迟的原因就是因为UIScrollView再接收到事件触发的时候，需要判断这个touch到底是应该传递下去还是自己处理以便滑动。

假如我们将UIScrollView delaysContentTouches属性设置为NO,那么这个延迟就不存在了，但于此同时，恰好touch在子视图上被接收，那不管怎么滑动界面，界面都不再滚动。

    
    @property(nonatomic) BOOL delaysContentTouches;       
    // default is YES. if NO, we immediately call -touchesShouldBegin:withEvent:inContentView:


当然如果我们要在滑动的时候可以获得事件也是可以的，通过重写touchesShouldCancelInContentView方法可以实现。

    
    // called before scrolling begins if touches have already been delivered to
    // a subview of the scroll view. if it returns NO the touches will continue
    // to be delivered to the subview and scrolling will not occur
    // not called if canCancelContentTouches is NO. default returns YES 
    // if view isn't a UIControl
    - (BOOL)touchesShouldCancelInContentView:(UIView *)view;


简答翻译一下注释：如果touches已经被传递到当前ScrollView的某个子视图上，当滑动即将发生之前该方法会被调用。如果返回NO的话，touches将会继续重新移交至ScrollView上进行处理并且滑动事件不会被打断(前提是canCancelContentTouches[可取消内容视图的touches]为YES，如果是NO的话就不行了)，默认的情况下如果参数view不是UIControl的实例的话，总是返回YES
