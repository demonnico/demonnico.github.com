---
author: demon
comments: true
date: 2012-11-19 15:46:37
layout: post
slug: uiscrollview_inifiti_animatio
title: UIScrollView做无尽循环滚动
wordpress_id: 104
categories:
- iOS
- 开发
tags:
- UIScrollView
- UITableView
- 无尽循环
---

今天接到一个需求，首先要做几个UITableView页，然后实现在tableView页上作水平滚动进行翻页，并且可以做无尽的循环滚动。（----3,1,2,3,1----，类似这样走马灯的效果）

首先分解了一下的：

1、要实现水平滚动可以简单通过scrollView来实现，简单地就是设置其contentSize为我们子页面的总宽度，然pagingEnabled为yes，当滚动到下一个子页面的时候就会进行判断是否翻页。

2、无尽循环的话借用[StackOverFlow](http://stackoverflow.com/questions/2735804/objective-c-endless-uiscrollview-without-pagingenabled)上的一个思路，在首页index=0前面再加上一页，内容刚好是我们尾页的内容。在尾页最后刚好加上一页，内容刚好是首页的内容。最后滑动到首页或者尾页的时候，做一个伪处理:

{% highlight objc %}
[_tv_scrollView scrollRectToVisible:rect animated:NO];
{% endhighlight %} 

关于第二点，找到了一篇[中文的](http://furnacedigital.blogspot.fr/2011/04/uiscrollview.html)，讲得还比较详细。

写了一个[demo](https://github.com/demon1105/scroll-List),已上传~
