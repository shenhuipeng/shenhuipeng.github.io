---
layout:     post
title:      "(CVPR2020)TubeTK: Adopting Tubes to Track Multi-Object in a One-Step Training Model"
subtitle:   CVPR2020, MOTS
date:       2020-06-18
author:     pshow
header-img: img/post-bg-debug.png
typora-root-url: /img
catalog: true
categories: codes
tags:
    - paper
    - codes
---

##  (CVPR2020)TubeTK: Adopting Tubes to Track Multi-Object in a One-Step Training Model

### 主要贡献：

- 提出端到端的TubeTK结构，只需要利用“bounding-tube”结构在视频切片上训练

  类似其他的CV任务仅需一步即可完成追踪任务。

  a concise end-to-end model TubeTK which only needs one step training by introducing the “bounding-tube” to indicate temporal-spatial locations of objects in a short video clip.

- TubeTK提供了一种全新的目标追踪思路

  TubeTK provides a novel direction of multi-object tracking, and we demonstrate its potential to solve the above challenges without bells and whistles.

- 利用时-空特征，可以不依靠ReID实现遮挡目标的追踪恢复

  TubeTK has the ability to overcome occlusions to some extent without any ancillary technologies like ReID.

### 简介

Video multi-object tracking (MOT)中往往采用tracking-by-detection (TBD) 框架。As a two-step method,把目标追踪拆分为两步，逐帧检测目标的空间位置，然后在时域上将检测结果串联为一个个轨迹。

目前TDB面临以下三个问题

- 采用TBD框架的检测模型的性能会因检测模型的不同而发生显著变化。过于依赖目标检测限制了MOT的发挥

- 仅仅依靠空间信息不能很好地处理严重遮挡的问题

  ![image-20200618143809346](/img/(CVPR2020)TubeTK%20Adopting%20Tubes%20to%20Track%20Multi-Object%20in%20a%20One-Step%20Training%20Model.assets/image-20200618143809346.png)

- 追踪策略需要视频的spatial-temporal information (STI)但是TBD破坏了语义的连贯性。

Can we solve the multi-object tracking in a neat one-step framework?

we demonstrate that the much simpler one-step tracker even achieves better performance than the TBD-based counterparts.

As shown
in Fig. 1, a Btube is defined by 15 points in space-time compared to the traditional 2D box of 4 points. 3个bbox 加 3个时间戳。

使用了3D CNN，将视频作为3D数据输入。

To predict the Btube that captures spatial-temporal information, we employ a 3D CNN framework. By treating a video as 3D data instead of a group of 2D image frames, it can extract spatial-temporal features simultaneously.

简单的IOU后处理

simple IoU-based post-processing is applied to link Btubes and form final tracks.

### 相关工作

Tracking-by-detection-based model，两步，Re-ID techniques

Bridging the gap between detection and tracking，利于运动或者时域信息来辅助检测

Tracking framework based on trajectories or tubes

### 提出的追踪模型

#### 将bbox构建为btube

**Btube definition：** Adopting a 3D Bbox to point out an object across frames is the simplest extension method, but obviously, this 3D Bbox is too sparse to precisely represent the target’s moving trajectory.

一个Btube可以由15个时空坐标点构成

A Btube can be uniquely identified in space and time by 15 coordinate values and it is generated by a method similar to the linear spline interpolation which splits a whole track into several overlapping Btubes.

![image-20200618151927852](/img/(CVPR2020)TubeTK%20Adopting%20Tubes%20to%20Track%20Multi-Object%20in%20a%20One-Step%20Training%20Model.assets/image-20200618151927852.png)

As shown in Fig. 2 a, a Btube T is a decahedron composed of 3 Bboxes in different video frames, namely $B_s$, $B_m$, and $B_e$, which need 12 coordinate values to define. And 3 other values are used to point out their temporal positions.

允许目标在短时间内改变一次运动方向。

This setting allows the target to change its moving direction once in a short time.

长宽比可以改变，也可以进行差值从而在时间维度上得到一些列bbox，$\{B_s,B_{s+1},...,B_m,...,B_{e-1},B_e\}$. $B_m$没必要居中。

**Generating Btubes from tracks：**Btubes can only capture simple linear trajectories, thus we need to disassemble complex target’s tracks into short clips in which motions can approximately be seen as linear and captured by our Btubes.

Btubes只能捕获线性的运动轨迹，所以需要将实际目标的运动轨迹切片。The disassembly process is shown in Fig. 2 b. We split a whole track into multiple overlapping Btubes by extending EVERY Bbox in it to a Btube. 每3个bbox为一个切片，切片之间有重叠。

虽然可以将切片加长，但是线性差值不能很好地拟合复杂的轨迹。

We can extend Bboxes to longer Btubes for capturing more temporal information, but long Btubes generated by linear interpolation cannot well represent the complex moving trail (see Fig. 2).

![image-20200618155713467](/img/(CVPR2020)TubeTK%20Adopting%20Tubes%20to%20Track%20Multi-Object%20in%20a%20One-Step%20Training%20Model.assets/image-20200618155713467.png)

**Overcoming the occlusion：**Btubes可以预测目标的运动趋势，从而可以短时预测被遮挡的物体，当两个轨迹交错时也能减少ID Switch。

#### 模型结构

we adopt the 3D convolutional structure [28] to capture spatial-temporal features, which is widely used in the video action recognition task [6, 24, 19]. The whole pipeline is shown in Fig. 3.

![image-20200618160047169](/img/(CVPR2020)TubeTK%20Adopting%20Tubes%20to%20Track%20Multi-Object%20in%20a%20One-Step%20Training%20Model.assets/image-20200618160047169.png)**Network structure**

![image-20200618160730047](/img/(CVPR2020)TubeTK%20Adopting%20Tubes%20to%20Track%20Multi-Object%20in%20a%20One-Step%20Training%20Model.assets/image-20200618160730047.png)

![image-20200618161016410](/img/(CVPR2020)TubeTK%20Adopting%20Tubes%20to%20Track%20Multi-Object%20in%20a%20One-Step%20Training%20Model.assets/image-20200618161016410.png)

**输出结构** 为了捕捉运动信息$B_m$是绝对坐标，而$B_s$和$B_e$都是基于$B_m$的相对坐标

![image-20200618161702728](/img/(CVPR2020)TubeTK%20Adopting%20Tubes%20to%20Track%20Multi-Object%20in%20a%20One-Step%20Training%20Model.assets/image-20200618161702728.png)

![image-20200618162123485](/img/(CVPR2020)TubeTK%20Adopting%20Tubes%20to%20Track%20Multi-Object%20in%20a%20One-Step%20Training%20Model.assets/image-20200618162123485.png)

#### 训练方法

**Tube GIoU** 将GIoU推广到3D

![image-20200618162615646](/img/(CVPR2020)TubeTK%20Adopting%20Tubes%20to%20Track%20Multi-Object%20in%20a%20One-Step%20Training%20Model.assets/image-20200618162615646.png)

![image-20200618162632678](/img/(CVPR2020)TubeTK%20Adopting%20Tubes%20to%20Track%20Multi-Object%20in%20a%20One-Step%20Training%20Model.assets/image-20200618162632678.png)

**损失函数**

![image-20200618163943097](/img/(CVPR2020)TubeTK%20Adopting%20Tubes%20to%20Track%20Multi-Object%20in%20a%20One-Step%20Training%20Model.assets/image-20200618163943097.png)

#### 链接TBox

NMS时利用$B_s$和$B_e$来预测目标运动的轨迹

![image-20200618164452320](/img/(CVPR2020)TubeTK%20Adopting%20Tubes%20to%20Track%20Multi-Object%20in%20a%20One-Step%20Training%20Model.assets/image-20200618164452320.png)

链接时考虑了TBox 的朝向。

![image-20200618165027291](/img/(CVPR2020)TubeTK%20Adopting%20Tubes%20to%20Track%20Multi-Object%20in%20a%20One-Step%20Training%20Model.assets/image-20200618165027291.png)

![image-20200618165238096](/img/(CVPR2020)TubeTK%20Adopting%20Tubes%20to%20Track%20Multi-Object%20in%20a%20One-Step%20Training%20Model.assets/image-20200618165238096.png)

### 实验

使用了由GTA5构成的额外数据集

![image-20200618165800030](/img/(CVPR2020)TubeTK%20Adopting%20Tubes%20to%20Track%20Multi-Object%20in%20a%20One-Step%20Training%20Model.assets/image-20200618165800030.png)

![image-20200618170653755](/img/(CVPR2020)TubeTK%20Adopting%20Tubes%20to%20Track%20Multi-Object%20in%20a%20One-Step%20Training%20Model.assets/image-20200618170653755.png)

![image-20200618171429770](/img/(CVPR2020)TubeTK%20Adopting%20Tubes%20to%20Track%20Multi-Object%20in%20a%20One-Step%20Training%20Model.assets/image-20200618171429770.png)

![image-20200618171455734](/img/(CVPR2020)TubeTK%20Adopting%20Tubes%20to%20Track%20Multi-Object%20in%20a%20One-Step%20Training%20Model.assets/image-20200618171455734.png)

![image-20200618171531287](/img/(CVPR2020)TubeTK%20Adopting%20Tubes%20to%20Track%20Multi-Object%20in%20a%20One-Step%20Training%20Model.assets/image-20200618171531287.png)

![image-20200618171647080](/img/(CVPR2020)TubeTK%20Adopting%20Tubes%20to%20Track%20Multi-Object%20in%20a%20One-Step%20Training%20Model.assets/image-20200618171647080.png)

![image-20200618171704303](/img/(CVPR2020)TubeTK%20Adopting%20Tubes%20to%20Track%20Multi-Object%20in%20a%20One-Step%20Training%20Model.assets/image-20200618171704303.png)

![image-20200618171731449](/img/(CVPR2020)TubeTK%20Adopting%20Tubes%20to%20Track%20Multi-Object%20in%20a%20One-Step%20Training%20Model.assets/image-20200618171731449.png)

### 讨论

![image-20200618172300386](/img/(CVPR2020)TubeTK%20Adopting%20Tubes%20to%20Track%20Multi-Object%20in%20a%20One-Step%20Training%20Model.assets/image-20200618172300386.png)

![image-20200618172341160](/img/(CVPR2020)TubeTK%20Adopting%20Tubes%20to%20Track%20Multi-Object%20in%20a%20One-Step%20Training%20Model.assets/image-20200618172341160.png)

![image-20200618172403613](/img/(CVPR2020)TubeTK%20Adopting%20Tubes%20to%20Track%20Multi-Object%20in%20a%20One-Step%20Training%20Model.assets/image-20200618172403613.png)

![image-20200618172535256](/img/(CVPR2020)TubeTK%20Adopting%20Tubes%20to%20Track%20Multi-Object%20in%20a%20One-Step%20Training%20Model.assets/image-20200618172535256.png)

![image-20200618172549954](/img/(CVPR2020)TubeTK%20Adopting%20Tubes%20to%20Track%20Multi-Object%20in%20a%20One-Step%20Training%20Model.assets/image-20200618172549954.png)
