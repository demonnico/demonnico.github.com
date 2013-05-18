---
author: demon
comments: true
date: 2013-05-03 22:58:38
layout: post
slug: something_nsrunloop
title: NSRunloop的简单认识
wordpress_id: 193
categories:
- IOS
- 开发
tags:
- RunLoop
- UIScrollView
---

最早接触runloop的概念，是第一次用NSTimer的时候。一个最简单的例子:

    
    - (void)viewDidLoad
    {
        [super viewDidLoad];
    	// Do any additional setup after loading the view, typically from a nib.
        NSTimer * timer = [NSTimer scheduledTimerWithTimeInterval:1
                                                  target:self
                                                selector:@selector(printMessage:)
                                                userInfo:nil
                                                 repeats:YES];
    //    [[NSRunLoop currentRunLoop] addTimer:timer
    //                                 forMode:NSRunLoopCommonModes];
    }


如果我们同时在界面上滚动一个scrollview,那么我们会发现在滚动停止之前，控制台是不会有输出的，就好像scrollView在滚动的时候将timer暂停了一样。通过了解后发现，其实是Cocoa的RunLoop Mode在作怪。

我把runloop理解为一种cocoa下的一种消息循环的机制，用来处理各种消息事件。我们在开发的时候一般并不需要手动去创建一个runloop，因为在程序进入mainThread之后其实就为我们创建了默认的的mainRunLoop，通过[NSRunloop currentRunLoop]我们就可以得到当前线程对应的RunLoop对象，而我们需要留意的是在多个runloop之间消息的通知方式。

接上面说到的，开启一个NSTimer实质上是开启了一个新的线程(Runloop)，也就是说除了MainRunloop之外还有一个Runloop存在。而当scrollView在滚动的时候，当前MainRunLoop是处于UITrackingRunLoopMode，在该模式下，不会处理 NSDefaultRunLoopMode的消息，而NSTimer在创建后的RunLoop(B)默认会以NSDefaultRunLoopMode与当前context的runloop(A)发送消息。要想在scrollView滚动的同时也接受其他runloop的消息，则需要改变两者之间的RunLoopMode

    
     [[NSRunLoop currentRunLoop] addTimer:timer
                                  forMode:NSRunLoopCommonModes];


类似的问题在前几天修改一个http异步通信模块的时候也碰到了，简单地说是向服务器异步获取图片数据后通知主线程刷新tableView中的图片，但在tableView滚动还没有停止或用户手指还停留在屏幕上的时候，图片一直不会出来。后来发现请求数据的时候用到了NSURLConnection的`
`

    
    - (id)initWithRequest:(NSURLRequest *)request 
                 delegate:(id)delegate;


了解后发现该方法创建的异步请求线程和NSTimer一样，也是NSDefaultRunLoopMode的，与

    
    - (id)initWithRequest:(NSURLRequest *)request 
                 delegate:(id)delegate 
         startImmediately:(BOOL)startImmediately


不同的是，上面的方法默认创建后默认直接发起请求，并以NSDefaultRunLoopMode与runloop进行消息传递，因此我们需要和NSTimer一样更改他的RunLoopMode。

    
    NSURLConnection *connection = [[NSURLConnection alloc]
                                   initWithRequest:request
                                   delegate:self
                                   startImmediately:NO];
    [connection scheduleInRunLoop:[NSRunLoop currentRunLoop]
                forMode:NSRunLoopCommonModes];
    [connection start];


[StackOverFlow](http://stackoverflow.com/questions/1826913/delayed-uiimageview-rendering-in-uitableview)

[Github Demo](https://github.com/demon1105/RunloopDemo)
