---
layout:     post
title:      " Tracking without bells and whistles"
subtitle:  ICCV2019,  MOTS
date:       2020-07-01
author:     pshow
header-img: img/post-bg-debug.png
typora-root-url: /img
catalog: true
categories: codes
tags:
    - paper
    - codes
---
## Tracking without bells and whistles

### 主要贡献

使用目标检测作为前端，配合上reID和相机运动补偿。

- introduce the Tracktor which tackles multi-object tracking by exploiting the regression head of a detector to perform temporal realignment of object bounding boxes.
- present two simple extensions to Tracktor, a reidentification Siamese network and a motion model.
- conduct a detailed analysis on failure cases and challenging tracking scenarios, and show none of the dedicated tracking methods perform substantially better than our regression approach.
- propose our method as a new tracking paradigm which exploits the detector and allows researchers to focus on the remaining complex tracking challenges

### 简介

分两步 逐帧检测目标，将上下帧的目标关联起来。

由于缺检，误检，遮挡，目标在复杂场景下的交互，使得数据关联十分困难。

只训练深度学习检测器即可达到良好的追踪效果。

**If a detector can solve most of the tracking problems, what are the real situations where a dedicated tracking algorithm is necessary?**

如果仅仅使用检测器就可以解决绝大多数的追踪问题，边不在亟需专用的追踪算法。

### 相关工作

略

### A detector is all you need

积极需要一个检测器

- 不需要对追踪进行特化训练
- 在测试时不需要复杂的优化过程

#### Object detector

The detector yields the final set of object detections by applying non-maximum-suppression (NMS) to the refined bounding box proposals.

本文也是在这个阶段进行bbox的回归于预测从而实现追踪的目的

#### Tracktor

轨迹（trajectory）可以被定义为一系列bbox的集合：

$T_k=\{b^k_{t_1}, b^k_{t_2},...\}$

而bbox又可以被坐标和帧号所定义，t是帧号

$b^k_t=(x,y,w,h)$

所以帧t里所包含的目标bbox可以被描述为

$B_t=\{b^{k_1}_t, b^{k_2}_t,...\}$

在$t=0$时，追踪器基于初始的检测结果进行初始化

$D_0=\{d^1_0, d^2_0,...\}=B_0$

![image-20200701155017610](/img/Tracking%20without%20bells%20and%20whistles(ICCV2019).assets/image-20200701155017610.png)

**Bounding box regression.**

如蓝色箭头所示，我们想获得目标$k$在第$t$帧中的位置。我们便需要将$b^{k}_{t-1}$修正为$b^{k}_t$

我们假设两帧之间目标的位移很小，所以在$t$帧的ROI polling 过程中我们使用$b^{k}_{t-1}$来代替RPN的结果。通过现有的bbox回归即可得到当前帧的$b^{k}_t$

一下两种情况会杀死轨迹

- 当分类分数$s_t^k$低于阈值$\sigma_{active}$，我们认为目标离开视野，或者是目标被非目标物体遮挡。
- 目标之间的相互遮挡，在对$B_t$进行NMS时出现IoU大于阈值$\lambda_{active}$

**Bounding box initialization.**

红色箭头是初始化并添加新的轨迹。检测器会提供当前帧的检测结果$D_t$，当存在bbox与任何现有的轨迹在当前帧的bbox都小于阈值$\lambda_{new}$的时候便判定出现了新的目标并添加为新的轨迹。

#### Tracking extensions

提出一个运动模型和ReID算法来提升性能

**Motion model.**

当相机大幅移动，或者帧率很低时，之间假设的帧与帧之间目标位移很小的假设便不再成立。

极端情况系上一帧与当前帧同一目标的bbox会完全不重合。

使用两种运动模型

- 对于移动的摄像机we apply a straightforward camera motion compensation (CMC) by aligning frames via image registration using the Enhanced Correlation Coefficient (ECC) maximization as introduced in [16].
- 对于低帧率的视频，we apply a constant velocity assumption (CVA) for all objects as in [11, 2].

**Re-identification.**

为了保持在线追踪的能力，使用基于Siamese外观特征的短时间ReID（short-term re-identification）。我们会保存在当前帧被杀死的轨迹$b^{k}_{t-1}$，直到经过了制定的时间$F_{reID}$

在这个时间内，我们会将新检测到的轨迹与之前被杀死的轨迹进行比较，如果相似度满足要求则激活之前的轨迹，并将新检测的轨迹合并到之前的轨迹。显然，ReID网络需要在追踪数据集上进行训练。为了减少错误的ReID，只有两者的bbox的IoU很大时才合并（比较时，先前被杀死的bbox会进行运动预测）。

### 实验

private detections和public detections版本

在public detections版本中每一帧的目标检测结果由benchmark提供

bounding box regressor 和 classifier 用的自己的网络。

#### 消融学习

![image-20200701162912968](/img/Tracking%20without%20bells%20and%20whistles(ICCV2019).assets/image-20200701162912968.png)

reID和CMC作用很大。

#### 跑分测试

![image-20200701163253548](/img/Tracking%20without%20bells%20and%20whistles(ICCV2019).assets/image-20200701163253548.png)

We evaluate the performance of our Tracktor++ on the test set of the respective benchmark, without any training or optimization on the tracking train set.

并没有在benchmark提供的数据集上训练

Note, we use the same Tracktor++ tracker, trained on MOT17Det object detections, for all benchmarks

### 分析

本文的方法能够很好地解决简单场景下的追踪问题，而没有使用任何先验的追踪策略。，对复杂场景下的性能进行分析

#### 对追踪场景的分析

![image-20200701164139200](/img/Tracking%20without%20bells%20and%20whistles(ICCV2019).assets/image-20200701164139200.png)

**Object visibility.**

Intuitively, we expect diminished tracking performance for object-object or object-non-object occlusions, i.e., for targets with diminished visibility.

当目标被其他目标或非目标遮挡造成能见度下降的情况

![image-20200701164345255](/img/Tracking%20without%20bells%20and%20whistles(ICCV2019).assets/image-20200701164345255.png)

在能见度0.3-0.6之间表现优异，改进版本提升并不多

**Object size.**

目标大小对追踪性能的影响

![image-20200701165419656](/img/Tracking%20without%20bells%20and%20whistles(ICCV2019).assets/image-20200701165419656.png)

**Robustness to detections.**

如果目标检测的gap很大也会影响追踪性能

**Identity preservation.**

#### Oracle trackers

the impact of the object detector on the killing policy and bounding box regression,

identify performance upper bounds for potential extensions to our Tracktor.

![image-20200701170749688](/img/Tracking%20without%20bells%20and%20whistles(ICCV2019).assets/image-20200701170749688.png)

