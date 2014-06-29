---
author: demon
comments: true
date: 2014-06-27 22:55:00
layout: post
slug: camera AVCaptureVideoPreviewLayer
title: AVCaptureVideoPreviewLayer自定义相机
categories:
- iOS
- backup
tags:
- AVCaptureVideoPreviewLayer
- Camera
- 
---

之前在做自定义相机的时候，首先会想到用`UIImagePickerViewController`构建一个对象，直接设置`sourceType`为`UIImagePickerControllerSourceTypeCamera`就可以了。这样的做法简单暴力，而且界面和系统自带的Camera风格一致，比较省功夫。写了个简单的[Demo](https://github.com/demon1105/UIImagePickerDemo)，但这样做会有一个问题，在保存照片的时候会出现一个内存使用的小高峰。

![](https://raw.githubusercontent.com/demon1105/ImagesLib/master/uiimagepicker_memory.png)

于是自己花了点时间用[AVCaptureVideoPreviewLayer](https://github.com/demon1105/AVCapturePreviewLayerCamera)重新实现了相机的功能，如果对定制要求比较高的话这样做更为适合，而且在保存的时候也不会出现像使用`UIImagePickerViewController`时出现的内存问题。但有一些需要注意的是，存储时需要自己根据当前屏幕的朝向自行设置metadata的orientation信息，否则会出现打开后照片朝向不对的问题。但如果用户在Control Center把屏幕旋转给关了，我们无法在UIViewController的回调中得到当前屏幕的旋转信息怎么办？这里我们可以用CMMotionManager的加速度器传感器来做判断。

Over.



