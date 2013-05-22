---
author: demon
comments: true
date: 2013-05-09 19:38:24
layout: post
slug: ios_uiimage_optimizer
title: iOS如何避免图像解压缩的时间开销（转）
wordpress_id: 198
categories:
- iOS
- 开发
tags:
- iOS
- UIImageView
- UITableView
- 优化
- 性能
---

这几天一直在折腾项目中UITableView的优化。

因为cell中涉及到很多（而且很大）的图片，所以在快速滚动的时候，ImageView重新绘制新的图片造成了比较大的性能消耗。查了挺多的资料，看到一篇分析jpg,png（优化前后）在各个设备上测试分析，原文在这里（[voiding Image Decompression Sickness](http://www.cocoanetics.com/2011/10/avoiding-image-decompression-sickness/)），本来想自己翻译的，后来在LongTimeNoC看到已经翻译好的，就转过来了。

---------BEGIN---------

当开始[iCatalog.framework](http://www.cocoanetics.com/2010/10/icatalog-framework-brings-digital-catalogs-to-life-on-ipad/)的工作时，我发现使用大尺寸图片会引起一些恼人的问题，“大”意味着这个图片有足够大的分辨率（1024×768）来覆盖iPad的整个屏幕，或者覆盖未来Retina Display iPad（如果有的话）的双倍分辨率（2048×1536）屏幕。

想像一个杂志类型的App，一个分页的UIScrollView，每页显示一个UIImageView，一旦某一页进入屏幕区域你就要为这个页创建或者重用一个UIImageView并把它放到scrollView的当前显示区域，即使这个页只有一个像素进入到屏幕区域，你还是要做这些工作。这在模拟器上运行得非常好，但在真机上进行测试，你会发现每次进入下一页时都会有一个明显的延迟。这个延迟来自于将图片从文件解压缩并渲染到屏幕上这一系列的工作。不幸的是UIImage仅在图片将要显示的时候做这个解压工作。

因为添加一个view到当前的view层次结构中必须在主线程上进行，所以图片的解压缩和之后渲染到屏幕上的工作也在主线程进行，这就是这个延迟产生的原因，这个问题也可以在store里的其他有类似这种效果的app中发现。

一般我们使用的图片有两种主要格式，jpeg和png。Apple通常推荐你使用png作为用户界面的图片格式，这些图片会被一个叫[pngcrush](http://pmt.sourceforge.net/pngcrush/)开源的工具优化(译者注:这个工具就在/Developer/Platforms/iPhoneOS.platform/Developer/usr/bin/pngcrush )，这样对于iOS设备就可以在显示时更快地进行解压和渲染。iPad平台上首批出现的杂志应用，比如Wired，就曾用过png作为杂志内容图片的格式，这导致了这个应用的某一版本大小超过了500MB(link)[http://www.cocoanetics.com/2010/05/saturday-morning-breakfast-wired-emag/]

虽然png格式的图片会被事先优化好，但是这并不意味着在所有情况下png都是最佳的图片格式，png对于那些app中自带的图片来说非常好，但是对于要从internet上down下来的图片来说又会怎样呢。png和jpeg这两种格式都有各自的优缺点：

png格式的图片有alpha通道，jpeg则没有。png无损压缩，jpeg允许你选择0-100%的压缩质量。如果需要alpha通道(透明)，就只能用png格式。但是如果你不需要一个完美的图片，就可以使用jpeg格式，jpeg格式会忽略那些你看不到的信息，对于大部分的图片可以使用60-70%的压缩质量而不对图片造成明显的影响，对于比如文字那样有"sharp pixels"的图片就可能需要较高的压缩质量，对于照片可以使用较低的压缩质量。

来看一下一个图片的空间消耗：

	  1. 磁盘空间或者通过internet传输所消耗的空间

	  2. 解压缩空间，通常是长X宽X高X4字节（RGBA）

	  3. 当显示在一个view中时，view本身也需要空间来存储layer


对于这里的第一个问题，有一个可能的优化方法：将压缩的文件拷贝到内存中不如映射到内存中，NSData有能力来假设一块磁盘空间是在内存中的，这样当访问这个图片时实际上就是从磁盘访问而不是从内存。据说CGImage知道哪种访问方式是最高效的，UIImage只是将CGImage封装了一下。

对于“将这些像素显示到屏幕上最快要多久？”这个问题，显示一个图片所消耗的时间由以下三个因素决定：

	
	  1. 从磁盘上alloc/init UIImage的时间

	  2. 解压缩的时间

	  3. 将解压缩后的比特转换成CGContext的时间，通常需要改变尺寸，混合，抗锯齿工作。


要逐一解答各个问题，我们需要一个benchmark来测量。


## 测试环境和测试内容


我做了一个在iOS设备上运行的[benchmark app](https://github.com/Cocoanetics/Cocoanetics-Benchmarks)。对优化和未优化的png图片以及不同尺寸的图片进行了测试，最小时间单位为1ms，不是特别精确，但已经有足够的参考价值了。

测试包括128×96, 256×192, 512×384, 1024×768 和 2048×1536这几种比较有代表性的分辨率，以及优化过的png，未优化的png，压缩质量从10-100%的jpeg这几种格式。benchmark运行在iPad 1+2，iPhone3G，iPhone3G，iPhone4上。

![Flower 2048x1536 90 300x225](http://longtimenoc.com/wordpress/wp-content/uploads/2012/01/iOSFlower_2048x1536.90-300x225.jpg)

以下为各个设备的硬件参数，来自Wikipedia：
	
	  * iPhone 3G:Samsung 32-bit RISC ARM11 620 MHz 处理器 (降频到 412 MHz), PowerVR MBX Lite 3D GPU, 128 MB eDRAM
	  * iPhone 3GS: Samsung APL0298C05 芯片，ARM Cortex-A8架构，降频到600MHz(从833MHz)，集成PowerVR SGX 535 GPU，256 MB eDRAM。
	  * iPhone 4:基于ARM Cortex-A8的Apple A4芯片，集成 PowerVR SGX 535 GPU，在iPad中频率为1GHz，iPhone 4中的频率没有披露(译者注：不都说是800MHz么，还有512MB RAM都忘了写了，不过这些不说大家也知道)
	  * iPad 1:1GHz Apple A4芯片 256MB DDR Ram (译者吐槽：原文写的是256GB，吾派的壮哉！)
	  * iPad 2:1GHz 双核Apple A5芯片，512MB DDR2 RAM

这其中iPad 1和iPhone 4有相同的处理器，在Apple使用自家的芯片后我们看到了性能的明显提升，锁频的A4比之前的Cortex-A8+PowerVR SGC 535 GPU要快一倍


## 测试数据


1024×768分辨率，90%压缩质量的jpeg图片从加载，解压缩到渲染的时间：

	* iPhone 3G: 527 ms
	* iPhone 3GS: 134 ms
	* iPad: 79 ms
	* iPhone 4: 70 ms
	* iPad 2: 51 ms


同样是1024×768分辨率，对比优化和未优化的PNG图片
	
	* iPhone 3G: 866 ms – 1032 ms = 快16%
	* iPhone 3GS: 249 ms – 458 ms = 快46%
	* iPad: 130 ms – 256 ms = 快49%
	* iPhone 4: 179 ms – 309 ms = 快42%
	* iPad 2: 105 ms – 208 ms = 快49%


3GS的数据充分体现了优化过的png好处：比未优化的快一倍。

以下的图表记录了所有测试数据。

我们首先可以发现同一图表中(相同分辨率)不同的压缩质量有着相似的时间开销，为了数据的完整我还是把所有数据都提供出来：


###### ![Chart 0256](http://longtimenoc.com/wordpress/wp-content/uploads/2012/01/iOSChart_0256.png)![Chart 0128](http://longtimenoc.com/wordpress/wp-content/uploads/2012/01/iOSChart_0128.png)


这两张图表代表了那些可能用在用户界面中的图片。忽略那个古老的设备，我们可一看到所有格式基本都在20ms左右完成显示，这个时间是图片足够小以便即时显示的下限。如果你有一个古老的设备而又不想让你的App很卡的话就要把解压缩工作放到主线程之外进行。

以下的三张图表测试的是那些拥有可以充满iPhone，iPad屏幕的分辨率的图片。


###### ![Chart 0512](http://longtimenoc.com/wordpress/wp-content/uploads/2012/01/iOSChart_0512.png)


到此之前如果我们忽略古老设备的话，大部分图片都能很快显示，我们可以看到这种条件下使用png图片会对性能造成冲击，我们会更倾向于使用jpeg。


###### ![Chart 1024](http://longtimenoc.com/wordpress/wp-content/uploads/2012/01/iOSChart_1024.png)


![Chart 2048](http://longtimenoc.com/wordpress/wp-content/uploads/2012/01/iOSChart_2048.png)

这些分辨率下即使最快的设备每秒也只能处理2(iPad Retina全屏)／10(iPad全屏)张大图片。

即使在非主线程上进行解压缩工作，绘制图片仍需要消耗可观的时间，你可能应该将图片分成小块并使用CATiledLayer来完成显示。

综上我们可一看到红色区域(解压缩)总是消耗最多时间的部分，渲染(绘制)的时间仅取决于分辨率而不是压缩质量，因为像素占组要因素。

通常100%质量的jpeg和优化过的png图片时间开销大致相同。我可以想到两个使用jpeg的理由：
	
	1）在设备上不能动态创建优化过的png 
	2）你只是需要一个完美的图片而不需要考虑磁盘空间开销

横向对比文件大小(文件大小的图表在下面)，可以看到从jpeg 10%到jpeg 90%的时间开销线性增长。

在最后这张图表中有一个有趣的问题，在未来可能Retina Display iPad设备上，我们所用的图片的解码时间应当比iPad 2快3－4倍(因为我们需要让图片即时显示在双倍分辨率屏目上，也就是比之前多3倍的像素)，而从iPad 1到iPad 2仅有一倍的性能提升，所以这就是iPad 2还不能支持Retina显示的原因。


## 文件大小


让我们看一下文件大小，优化过的png图片渲染起来更快，但是它减小了文件大小么？

![Screen Shot 2011 09 30 at 5 08 04 PM](http://longtimenoc.com/wordpress/wp-content/uploads/2012/01/iOSScreen-Shot-2011-09-30-at-5.08.04-PM.png)

优化过的png格式仅仅对大尺寸图片的文件大小有少量减小，100%压缩质量的jpeg图片文件大小比优化过的png要小，但是100%的质量似乎失去了压缩的目的。我们可一看到使用90%压缩质量的文件要比100%压缩质量的文件小一半还多，jpeg格式的文件大小从10%到90%线性增长，但从90%到100%却有个巨大的增加。


## 即时性？


对于全屏尺寸的图片我们可以发现一个自从Apple推出平板设备以来就有的一个问题：一个70%压缩质量的全屏尺寸图片在iPad 1上显示需要高达75ms，iPad 2上仍要49ms，完全不能满足即时显示的要求。这与我们60fps的目标相差甚远，13fps或20fps与让人感觉流畅的30祯相比也有很大差距。这会导致在我们拖动scrollview，新图片进入屏幕时，将会卡20分之一秒，之后scrollview必须跳动来追赶上你的手势。

如果排除解压缩的时间，结果就比之前强多了，17－18ms的时间将带来大约55fps的流畅度。有趣的是两代iPad将像素混合到layer的时间没有多少区别，区别仅在解压缩上。

对此我在iCatalog中绘制catalog页的一个简单解决方法就是使用CATiledLayer，并禁用fading。这样就可以在后台线程处理图片的显示而不影响scroll的性能。当然如果快速向右滑动（scroll），相应页的显示就会有一个明显的延迟。这种解决方法的缺点就是很难将横竖屏切换做得很好。

一个更先进的方法就是提前强制解压缩图片。


## 强制解压缩


当第一次使用图片时，iOS会解压它。通常这个解压缩后的版本将滞留一段时间（内存允许）。尽管这么做没什么意义，但你可以通过将图片渲染成一个新的图片来解压缩这个图片。这样你将在一小段时间内获得**两个**解压缩的版本。

`- (void)decompressImage:(UIImage *)image
{
UIGraphicsBeginImageContext(CGSizeMake(1, 1));
[image drawAtPoint:CGPointZero];
UIGraphicsEndImageContext();
}
`

这一段代码会解压缩这个image，即时它只有一个像素。

奇怪的是如果UIImage只是通过initWithContentsOfFile创建的，我不能始终保持这个解压缩的版本。所以我必须使用ImageIO framework(iOS 4之后可用) 中提供的一个选项来显式保持这个解压缩的版本:

	NSDictionary *dict = [NSDictionary dictionaryWithObject:[NSNumber 	numberWithBool:YES]
	forKey:(id)kCGImageSourceShouldCache];

CGImageSourceRef source = CGImageSourceCreateWithURL((CFURLRef)url, NULL);
CGImageRef cgImage = CGImageSourceCreateImageAtIndex(source, 0, (CFDictionaryRef)dict);

	UIImage *retImage = [UIImage imageWithCGImage:cgImage];
	CGImageRelease(cgImage);
	CFRelease(source);

这样初始化图片就可以让解压缩仅发生一次：第一次解压缩消耗很长一段时间，第二次完全不消耗。这其中的关键就是kCGImageSourceShouldCache，你可以为CGImageSource和CGImageSourceCreateImageAtIndex使用这个选项，头文件中是这样说明的：

	Specifies whether the image should be cached in a decoded form. The value of this key must be a CFBooleanRef; the default value is kCFBooleanFalse.

如果这个选项设置为NO，绘制图片的时间又会随着解压缩时间增长，如果设置为YES就仅仅解压缩一次。


## 结论


如果你需要alpha通道或者必须使用PNG格式，那么我推荐你在你的web服务器上安装pngcrush并处理好所有的png图片。其他情况下，高质量的jpeg能带来较小的文件大小以及更快的解压缩和渲染。

事实证明，png格式对于那些使用在UI元素中的小图片来说非常好，但是对于那些全屏显示图片的应用来说则完全不是。替代png的通常是60-80%压缩质量的jpeg，至于压缩质量取多少合适，这取决于你的图片内容。

你可能希望所有显示过的图片都能保持他们的解压缩版本，但是这也将带来大量的内存开销并导致你的App进程被杀掉。此时使用NSCache就是一个很好的解决方案。它可以自动在内存短缺的时候照看好这些图片。

虽然不幸的是我们不能知道一个图片是否需要解压缩，同样一个图片的解压缩版本消失时我们也不会获得任何通知（这或许非常适合提交到Apple的bug反馈网站上）。但幸运的是，通过上述方法访问的解压缩后的图片不会再在解压缩上消耗时间。所以你可以同时在恰当的时间和恰当的条件下使用这种方法而不造成额外的开销。

------OVER------

这里还要说的是，如果你的图片很大，可能还需要做其他的工作，比如可以牺牲一下及时性来换取流畅的滚动的话，延迟加载（取消实时的绘制，待tableView滚动停止后才对cell中的UIImageView进行绘制）也是一个可参考的方案。

感谢**ultragtx**的辛勤劳动，这里附上[原文](http://longtimenoc.com/archives/ios%E5%A6%82%E4%BD%95%E9%81%BF%E5%85%8D%E5%9B%BE%E5%83%8F%E8%A7%A3%E5%8E%8B%E7%BC%A9%E7%9A%84%E6%97%B6%E9%97%B4%E5%BC%80%E9%94%80)的连接。
