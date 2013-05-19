---
author: demon
comments: true
date: 2013-05-18 00:15:43
layout: post
slug: designing-for-ios-graphics-performance
title: iOS图形处理和性能
wordpress_id: 203
categories:
tags:
- CALayer
- CPU
- GPU
- Graphics
- IOS
- perfermence
---

原文的题目是[Designing for iOS: Graphics & Performance](http://robots.thoughtbot.com/post/36591648724/designing-for-ios-graphics-performance)，晚上花了两个不到小时大致翻译了下。

---Begin---

在[之前的文章](http://robots.thoughtbot.com/post/33427366406/designing-for-ios-taming-uibutton)里，我们探讨了基于多种不同技术来实现自定义的UIButton，当然不同的技术所涉及到的代码复杂度和难度也不一样。但是我也有意提到了基于不同方法的实现所体现出的性能表现也不一一相同。

【在屏幕背后的东西】

为了了解性能是如何受到影响的，我们需要进一步地观察iOS里图形实现背后的一些内容。下面这张图呈现了不同的frameworks和libraries之间的一些联系：

![](http://pic.yupoo.com/demon42111915/CRYk1ku8/medish.jpg)

在最顶层的就是UIKit，一个在iOS中用来管理用户图形交互的Objc高级的框架，它由一系列的集合类构成，例如UIButton、UILabel，每一个都负责他们指定的UI Control角色。UIKit本身构建在一个叫Core Animation的框架之上，它因为被用于处理更为强大的平滑的转场效果而引入OS X Leopard和iOS而出名。

再往下深入一点就是OpenGL ES了，一个用在移动设备上绘制2D和3D计算机图形的标准开源库，它广泛地被用在游戏的图形绘制上，同样在Core Animation和UIKit中发挥着强大的作用。

最后一部分就是Core Graphics，曾经在Quartz（一个基于CPU的绘制引擎，在OS X系统上初次露脸）中被引入。这两个较为底层的框架都是用C语言编写的。

上图中的最底下一行是硬件层，由GPU和CPU组成。

我们经常说到的硬件加速其实是指OpenGL,Core Animation/UIKit基于GPU之上对计算机图形合成以及绘制的实现，直到目前为止，iOS上的硬件加速能力还是大大领先与android，后者由于依赖CPU的绘制，绝大多数的动画实现都会让人感觉明显的卡顿。

而离屏绘制(Offscreen drawing)的话就是指GPU一边在当前屏幕上进行绘制，而另一边在屏幕还没有处理图像信息之前通过CPU来生成图像信息的处理过程。在iOS当中，离屏绘制在以下的情况下会自动触发：

    * Core Graphics(任何以CG开头的类)
	* 在drawRect方法里，甚至是空方法实现
	* 所有shouldRasterize属性是YES的CALayers对象	
	* 所有用了masks(setMasksToBounds)和动态阴影的(setShadow*)的CALayers对象
	* 所有文字的绘制，包括CoreText
	* Group opacity(UIViewGroupOpacity)


总地来说，当有涉及到动画的时候离屏绘制就会影响到性能，你可以通过Instruments进行真机调试，从而检测到底是哪部分UI正在进行离屏绘制：

	
  1. 接入设备
  2. 在XCode的Developer Applications里打开Instruments（Command+Shift+i） ![](http://pic.yupoo.com/demon42111915/CRYj9u3F/medish.jpg)
  3. 选择iOS>Graphics>Core Animation template ![](http://pic.yupoo.com/demon42111915/CRYj8kna/medish.jpg)
  4. 打开详情面板，选择适当的窗口模式![](http://pic.yupoo.com/demon42111915/CRYj7Ik3/medish.jpg)[
]	
  5. 选择你的target设备	
  6. 检查Color Offscreen-Rendered Yellow的debug选项
  7. 在你设备上所有的离屏绘制都会呈现出黄色的色调![](http://pic.yupoo.com/demon42111915/CRYj7ix4/medish.jpg)


现在让我们逐一检查上一篇文章里涉及的一些技术点的性能表现。

【预资源加载绘制】

使用UIImage来加载原先就在磁盘中的图片作为自定义UIButton的背景图片的话，完全依赖与GPU的绘制。而且如果用可resizable的图片，也可适当避免当背景大小动态变换的时候需要不同大小的图片，这样可以让我们App的bundle做得更小，而且在绘制像素的图的时候也重复地利用了硬件加速。

【使用CALayer】

因为基于CALayer实现的方法里我们使用到了遮罩去绘制圆角的效果，所以这里会涉及到离屏绘制。当我们使用Core Animation框架的时候，如果上面提到的用遮罩绘制圆角的属性是开启的话，我们也必须明确地禁止使用动画效果。作为底线，除非你非要进行一个动画转场效果，否则这种方法并不适用。

【使用drawRect】

drawRect方法依赖Core Graphics框架来进行自定义的绘制，但这种方法主要的缺点就是它处理touch事件的方式：每次按钮被点击后，都会用setNeddsDisplay进行强制重绘；而且不止一次，每次单点事件触发两次执行。这样的话从性能的角度来说，对CPU和内存来说都是欠佳的。特别是如果在我们的界面上有多个这样的UIButton实例。

【技巧混合 hybrid approach】

这样来说的话，是不是就意味着使用预资源加载进行绘制就是唯一可行的方案了呢？答案是否定的。如果你坚持需要用代码来灵活地进行绘制的话，还是有一些优化代码还有减少性能消耗方面的技巧的。有一种可行的方案就是创建stretchable的位图并在多个实例对象之间进行重用。

我们按照在之前的教程的相同步骤创建一个新的UIButton的子类，然后如下定义一些静态变量

    
    // In CBHybrid.m
    #import "CBHybrid.h"
    
    @implementation CBHybrid
    
    // Resizable background image for normal state
    static UIImage *gBackgroundImage;
    
    // Resizable background image for highlighted state
    static UIImage *gBackgroundImageHighlighted;
    
    // Background image border radius and height
    static int borderRadius = 5;
    static int height = 37;


接下来我们把CBBezie内drawRect里的代码换个地方，通过一系列的改变之后：我们可以创建一个resizable的Image用来取代之前固定尺寸的Image，然后我们可以持有该静态对象并便于重用

    
    - (UIImage *)drawBackgroundImageHighlighted:(BOOL)highlighted {
        // Drawing code goes here
    }


首先，我们需要知道我们resizable Image的宽，为了优化性能，我们应该在图片的中间保留一个像素的宽

    
    float width = 1 + (borderRadius * 2);


高的话在这个case里就不是非常重要了，因为这个按钮的高度对渐变层来说已经足够可见，设置为37pt的话也是出于和其余几个按钮的高度相同的原因。

搞定之后，我们需要一个bitmap context来进行绘制，搞起~

    
    UIGraphicsBeginImageContextWithOptions(CGSizeMake(width, height), NO, 0.0);
    CGContextRef context = UIGraphicsGetCurrentContext();
    CGColorSpaceRef colorSpace = CGColorSpaceCreateDeviceRGB();


UIGraphicsBeginImageContextWithOptions第二个参数为NO的话确保我们创建的Image context是透明的（带Alpha），最后的参数是scale factor(屏幕密度)，如果是0的话就是当前设备的默认scale factor。

接下来的代码就和我们之前用Core Graphics来实现CBBezier的Demo里用到的非常相像了。我们用highlighted参数替换默认的self.highlighted属性，把用作更新界面的图像的信息值保存下来。

    
    // Gradient Declarations
    // NSArray *gradientColors = ...
    
    // Draw rounded rectangle bezier path
    UIBezierPath *roundedRectanglePath = [UIBezierPath bezierPathWithRoundedRect: CGRectMake(0, 0, width, height) cornerRadius: borderRadius];
    
    // Use the bezier as a clipping path
    [roundedRectanglePath addClip];
    
    // Use one of the two gradients depending on the state of the button
    CGGradientRef background = highlighted? highlightedGradient : gradient;
    
    // Draw gradient within the path
    CGContextDrawLinearGradient(context, background, CGPointMake(140, 0), CGPointMake(140, height-1), 0);
    
    // Draw border
    // [borderColor setStroke...
    
    // Draw Inner Glow
    // UIBezierPath *innerGlowRect...


我们唯一需要添加的步骤就是，用UIGraphicsEndImageContext来保存图像信息，并且放到UIImage对象中。

    
    UIImage* backgroundImage = UIGraphicsGetImageFromCurrentImageContext();
    
    // Cleanup
    UIGraphicsEndImageContext();




现在我们已经完成了创建背景图片的方法，接下来我们必须实现一个通用的初始化方法用来实例化Images，并且把他们设置为CBHybird实例真正的背景。

    
    - (void)setupBackgrounds {
    
        // Generate background images if necessary
        if (!gBackgroundImage &amp;&amp; !gBackgroundImageHighlighted) {
            gBackgroundImage = [[self drawBackgroundImageHighlighted:NO] resizableImageWithCapInsets:UIEdgeInsetsMake(borderRadius, borderRadius, borderRadius, borderRadius) resizingMode:UIImageResizingModeStretch];
            gBackgroundImageHighlighted = [[self drawBackgroundImageHighlighted:YES] resizableImageWithCapInsets:UIEdgeInsetsMake(borderRadius, borderRadius, borderRadius, borderRadius) resizingMode:UIImageResizingModeStretch];
        }
    
        // Set background for the button instance
        [self setBackgroundImage:gBackgroundImage forState:UIControlStateNormal];
        [self setBackgroundImage:gBackgroundImageHighlighted forState:UIControlStateHighlighted];
    }


我们可以用custom的类型来实例化我们的CBHybird，当然也可以用initWithCoder，如果想用在代码里实现的话我们还可以用initWithFrame(···其实这里我是不想翻译的，原作者写得真是太详细了，第一次翻译技术文章，还是彻底点吧。。。)

    
    + (CBHybrid *)buttonWithType:(UIButtonType)type
    {
        return [super buttonWithType:UIButtonTypeCustom];
    }
    
    - (id)initWithCoder:(NSCoder *)aDecoder {
        self = [super initWithCoder:aDecoder];
        if (self) {
            [self setupBackgrounds];
        }
    
        return self;
    }


为了确保我们新建的BHybird类能正常使用，在Interface Builder里我们赋值一个button，把实现类改成CBHybird后，把button的content内容改为_CGContext-generated image（_便于区分）。是驴是马，咱们cmd+R跑起来试试~

![](http://pic.yupoo.com/demon42111915/CRYj7cxT/medish.jpg)

完整的子类实现代码在[这里](https://github.com/demon1105/custom-UIButton/blob/master/Custom%20UIButtons/CBHybrid.m)~~~

【结束语】

所有都搞定之后，我们发现用预资源加载绘制仍然比任何一种基于代码实现的方案表现得更为出色。然而，Core Graphicsis在效率和灵活性上更有优势。因此，基于两种技巧的混合方案在现在看来包含了以上两个的长处，而且在性能上也并没有明显的影响。

---Over---

第一次翻译，有些地方可能不是很到位，不足的地方请见谅:D

原文的评论中有篇精彩的回复，作者也有提到，感兴趣的可以看看。
