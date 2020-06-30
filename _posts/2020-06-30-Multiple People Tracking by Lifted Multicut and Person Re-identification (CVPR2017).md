---
layout:     post
title:      "Multiple People Tracking by Lifted Multicut and Person Re-identification"
subtitle:  CVPR2017,  MOTS
date:       2020-06-30
author:     pshow
header-img: img/post-bg-debug.png
typora-root-url: /img
catalog: true
categories: codes
tags:
    - paper
    - codes
---
## Multiple People Tracking by Lifted Multicut and Person Re-identification

### 主要贡献

- we propose a novel graph based formulation that links and clusters person hypotheses over time by solving an instance of a minimum cost lifted multicut problem.
- ReID模型包含了外观与姿势特征。
- propose to cast multi-person tracking as the minimum cost lifted multicut problem. We introduce two types of edges (regular and lifted edges) for the tracking graph.

### 简介

使用CNN提取的特征有很强的鲁棒性，也有利于ReID

外观相似的目标并不一定属于同一个ID

similar looking people are considered as the same person only if they are connected by at least one feasible track (possibly skipping occlusion)

每一个检测框都被表述为图中的一个节点。带有损失权重的边将跨越时间，空间将节点连接起来。

优点：

- 追踪目标的个数没有理论定义上的限制或者偏重。
- 由于同一帧中同一个目标的重复检测会被自然的聚类在一起，所以可以取消掉启发式的NMS算法

In order to avoid that distinct but similar looking people are assigned to the same track, a distinction must be made between the edges that define possible connections (i.e., a feasible set) and the edges that define the costs or rewards for assigning the incident nodes to distinct tracks (i.e., an objective function).

### 相关工作

流问题，最小损失分割问题。

### 模型

边的权重被描述为两个节点属于同一ID的可能性

![image-20200630110728961](/img/Multiple%20People%20Tracking%20by%20Lifted%20Multicut%20and%20Person%20Re-identification%20(CVPR2017).assets/image-20200630110728961.png)

**Parameters.**

![image-20200630111637394](/img/Multiple%20People%20Tracking%20by%20Lifted%20Multicut%20and%20Person%20Re-identification%20(CVPR2017).assets/image-20200630111637394.png)

常规的边会连接同一帧的节点或者低于阈值的附近帧的节点。

lifted edges 提升边链接时间距离较远的相似目标。

**Feasible Set.**

图分割问题

![image-20200630115006307](/img/Multiple%20People%20Tracking%20by%20Lifted%20Multicut%20and%20Person%20Re-identification%20(CVPR2017).assets/image-20200630115006307.png)

![image-20200630115609785](/img/Multiple%20People%20Tracking%20by%20Lifted%20Multicut%20and%20Person%20Re-identification%20(CVPR2017).assets/image-20200630115609785.png)

![image-20200630135937798](/img/Multiple%20People%20Tracking%20by%20Lifted%20Multicut%20and%20Person%20Re-identification%20(CVPR2017).assets/image-20200630135937798.png)

**Objective function.**

![image-20200630140840693](/img/Multiple%20People%20Tracking%20by%20Lifted%20Multicut%20and%20Person%20Re-identification%20(CVPR2017).assets/image-20200630140840693.png)

边的可能性越小，边的cost越大。

**Optimization.**

![image-20200630141255871](/img/Multiple%20People%20Tracking%20by%20Lifted%20Multicut%20and%20Person%20Re-identification%20(CVPR2017).assets/image-20200630141255871.png)

### Person Re-identification for Tracking

#### Architectures

![image-20200630142957550](/img/Multiple%20People%20Tracking%20by%20Lifted%20Multicut%20and%20Person%20Re-identification%20(CVPR2017).assets/image-20200630142957550.png)

![image-20200630143940354](/img/Multiple%20People%20Tracking%20by%20Lifted%20Multicut%20and%20Person%20Re-identification%20(CVPR2017).assets/image-20200630143940354.png)

#### Fusing Body Part Information

![image-20200630144051124](/img/Multiple%20People%20Tracking%20by%20Lifted%20Multicut%20and%20Person%20Re-identification%20(CVPR2017).assets/image-20200630144051124.png)

![image-20200630144258118](/img/Multiple%20People%20Tracking%20by%20Lifted%20Multicut%20and%20Person%20Re-identification%20(CVPR2017).assets/image-20200630144258118.png)

![image-20200630144514264](/img/Multiple%20People%20Tracking%20by%20Lifted%20Multicut%20and%20Person%20Re-identification%20(CVPR2017).assets/image-20200630144514264.png)

#### ReID实验分析

略

### Pairwise Potentials

![image-20200630145620664](/img/Multiple%20People%20Tracking%20by%20Lifted%20Multicut%20and%20Person%20Re-identification%20(CVPR2017).assets/image-20200630145620664.png)

![image-20200630145959108](/img/Multiple%20People%20Tracking%20by%20Lifted%20Multicut%20and%20Person%20Re-identification%20(CVPR2017).assets/image-20200630145959108.png)

![image-20200630150401961](/img/Multiple%20People%20Tracking%20by%20Lifted%20Multicut%20and%20Person%20Re-identification%20(CVPR2017).assets/image-20200630150401961.png)

### Tracking Experiments and Results

![image-20200630153143885](/img/Multiple%20People%20Tracking%20by%20Lifted%20Multicut%20and%20Person%20Re-identification%20(CVPR2017).assets/image-20200630153143885.png)

![image-20200630153909722](/img/Multiple%20People%20Tracking%20by%20Lifted%20Multicut%20and%20Person%20Re-identification%20(CVPR2017).assets/image-20200630153909722.png)

主要探讨$\delta_t$ 和$\delta_{max}$对MP ， LMP 的影响。