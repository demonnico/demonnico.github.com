---
author: demon
comments: true
date: 2012-12-11 18:04:49
layout: post
slug: uitextfield_uitextview_sucks
title: UITextField与UITextView一起使用的奇怪问题
wordpress_id: 120
categories:
- IOS
- 开发
tags:
- textFieldShouldReturn
- UITextField
- UITextView
---

今天在修改一个界面的时候碰到了一个很奇怪的问题。

当我在视图中同时使用UITextField和UITextView的时候，想通过点击keyboard的return键来实现textField到textView的焦点切换，于是在委托中重写了

{% highlight objc %}    
-(BOOL)textFieldShouldReturn:(UITextField *)textField
{% endhighlight  %}

在这里[textView becomeFirstResponder]，然后我再手动切换焦点到textField上之后，再点击return键，在textView获得焦点的同时，我发现textView中的光标也会随着操作的次数下移一行。好像在textView中自动换行了····

于是我又将当前试图同时设为textView的delegate，重写了textView所有的委托方法后，debug后发现在TextView获得焦点后执行了以下三个委托方法：

{% highlight objc %}    
- (BOOL)textViewShouldBeginEditing:(UITextView *)textView;
- (void)textViewDidBeginEditing:(UITextView *)textView;
- (void)textViewDidChange:(UITextView *)textView;
{% endhighlight %}


前两个方法很容易理解，但是纳闷为什么会调第三个呢···因为只是获得焦点而已，没理由textView中的文字也会发生改变额··

求助google大神后在[StackOverFlow](http://stackoverflow.com/questions/6825558/problem-with-textfieldshouldreturn-method-when-using-uitextfield-along-with-uite)上找到了解决方案，在委托中重写textFieldShouldReturn方法时，当前的textField return之后刚好要转移焦点到textView，那这时候方法应该返回NO而不是YES。

这时候在运行demo，在textField的键盘上点击return，textView也不会执行委托中的textViewDidChange方法了。
