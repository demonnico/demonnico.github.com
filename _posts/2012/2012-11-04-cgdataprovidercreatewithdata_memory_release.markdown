---
author: demon
comments: true
date: 2012-11-04 14:42:04
layout: post
slug: cgdataprovidercreatewithdata_memory_release
title: CGDataProviderCreateWithData对内存数据的释放
wordpress_id: 91
categories:
- IOS
- skills
- 开发
tags:
- CGDataProviderCreateWithData
- texturepacker
- 内存释放
---

这几天在搞个东西，想对TexturePacker生成的图片和plist进行还原，网上找了个一些朋友的方法，大致的思路基本上就是使用UIImage，然后通过读取.plist文件中的数据，解析之后对原图进行clip操作，这样做的话如果之前Publish出来的texture图是经过了优化处理的(trim，如果在texturePacker中勾选了这个选项，对Publish出来之后图进行仔细比较后应该不难发现，如果原图的有效像素点周围存在大片的R=0,G=0,B=0A=0的区域，texturePacker导出时会对这些区域进行优化，认为这些区域是没有意义的并切除，具体可以对看plist文件中source字段的信息进行比较)，那我们得到的UIImage将不是和原图完全一样的。

我要做的工作就是先在内存中先绘制一片和原图一样大小的区域(sourceSize)，然后得到纹理(texture)图中一小块切图区域的内存信息(frame)，最后通过sourceColorRect和offset对数据进行还原，将frame绘制到sourceSize中正确的位置，使它和原图的信息保持一致。

在最后绘制CGImageRef的时候我用到了

    
    /* Create an image. */
    CG_EXTERN CGImageRef CGImageCreate(size_t width, size_t height,
        size_t bitsPerComponent, size_t bitsPerPixel, size_t bytesPerRow,
        CGColorSpaceRef space, CGBitmapInfo bitmapInfo, CGDataProviderRef provider,
        const CGFloat decode[], bool shouldInterpolate,
        CGColorRenderingIntent intent)


这个方法，在这里传递图像的信息是通过provider这个参数传入的，要得到这个对象我们则需要通过如下的方法创建：

    
    /* Create a direct-access data provider using `data', an array of `size'
       bytes. `releaseData' is called when the data provider is freed, and is
       passed `info' as its first argument. */
    
    CG_EXTERN CGDataProviderRef CGDataProviderCreateWithData(void *info,
      const void *data, size_t size, CGDataProviderReleaseDataCallback releaseData)
      CG_AVAILABLE_STARTING(__MAC_10_0, __IPHONE_2_0);


在这个构建方法中我们可以将数据(data)作为第二个参数传入，具体各个参数的含义可以看注释

创建CGDataProviderRef对象后有个问题我们需要注意的是，当我们使用完该对象需要对这个对象进行释放，当然我们malloc出来的数据区域也要进行free，释放CGDataProviderRef对象我们需要用到CGDataProviderRelease，而我们的data却不能在此时立即free掉，否则就会看到创建出的图片在绘制的屏幕上之后数据不对，出现花屏的情况。在这里我推断有可能IOS在绘制UIImage的时候使用的内存信息只是作了一个简单的引用指向，所以我们立即释放data的话就会造成数据错误。

如果仔细看CGDataProviderCreateWithData方法的注释，应该就能明白，正确的做法应该是实现自己的释放方法，然后将该方法作为CGDataProviderCreateWithData的最后一个参数进行传入，那么CGDataProviderRef释放的时候就会对该CGDataProviderReleaseDataCallback进行回调，在里面我们可以安全释放我们的图像数据。

CGDataProviderReleaseDataCallback的写法可以参考下面的格式，具体的实现应该根据需求的不同而不同:

    
    static void providerReleaseData(void *info, const void *data, size_t size)
    {
        free((void *)data);
    }



