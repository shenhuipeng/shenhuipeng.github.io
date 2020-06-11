---
layout:     post
title:      "MOTS: Multi-object tracking and segmentation"
subtitle:   CVPR2019, MOTS
date:       2020-06-08
author:     pshow
header-img: img/post-bg-debug.png
typora-root-url: /img
catalog: true
categories: codes
tags:
    - paper
    - codes
---
# (CVPR2019)[MOTS Multi-object tracking and segmentation](http://openaccess.thecvf.com/content_CVPR_2019/html/Voigtlaender_MOTS_Multi-Object_Tracking_and_Segmentation_CVPR_2019_paper.html)

### 主要贡献：

- 使用半自动的方式在两个追踪数据集上构建了像素级别的掩膜 KITTI [13] and MOTChallenge [38] datasets for training and evaluating methods that tackle the MOTS task.

  Our new annotations comprise 65,213 pixel masks for 977 distinct objects (cars and pedestrians) in 10,870 video frames.

- 提出新的多目标追踪与分割评估基准

  We propose the new **soft Multi-Object Tracking and Segmentation Accuracy (sMOTSA)** measure that can be used to simultaneously evaluate all aspects of the new task.

- 使用单一CNN网络实现了目标检测追踪和语义分割 TrackR-CNN as a baseline

### 简介

bounding box level tracking performance is saturating 基于bbox的追踪已经趋于饱和

将目标检测，语义分割，目标追踪相关联

常规的VOS数据集do not provide annotations on video data or even information on object identities across different images.

常规的VOS数据集provide only bounding box annotations of objects.

![image-20200611145912394](/img/MOTSMulti-objecttrackingandsegmentation/image-20200611145912394.png)

当目标被遮挡时使用bbox作为gt就会引入多余的信息，使用语义掩膜是恰当的。

propose **TrackR-CNN** as a **baseline** method which addresses all aspects of the MOTS task.

TrackR-CNN extends Mask R-CNN [15] with 3D convolutions to incorporate temporal information and by an associassociation head which is used to link object identities over time.

### 相关工作

#### 多目标追踪数据集

In particular, targets may enter and leave the scene at any time and must be recovered after long-time occlusion and under appearance changes. Many MOT datasets focus on street scenarios,

#### VOS数据集

Existing VOS datasets contain only few objects which are also present in most frames. In addition, the common evaluation metrics for this task (region Jaccard index and boundary F-measure) do not take error cases like id switches into account that can occur when tracking multiple objects.

MOTS focuses on a set of pre-defined classes and considers crowded scenes with many interacting objects. MOTS also adds the difficulty of discovering and tracking a varying number of new objects as they appear and disappear in a scene.

#### VIS数据集

Cityscapes [12],BDD [62], and ApolloScape [19]

#### 半自动标注

generating segmentation masks from scribbles [50], or clicks [59]. These methods require user input for every object to be segmented, while our annotation procedure can segment many objects fully-automatically, letting annotators focus on improving results for difficult cases.

### 数据集

there are some datasets with MOT annotations, i.e., tracks annotated at the bounding box level. For the MOTS task, these datasets lack segmentation masks. Our annotation procedure therefore adds segmentation masks for the bounding boxes in two MOT datasets.

#### 半自动标注过程

We use a convolutional network to automatically produce segmentation masks from bounding boxes, followed by a correction step using manual polygon annotations.