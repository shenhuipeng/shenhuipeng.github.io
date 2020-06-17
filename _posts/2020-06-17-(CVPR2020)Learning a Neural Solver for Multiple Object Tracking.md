---
layout:     post
title:      "(CVPR2020)Learning a Neural Solver for Multiple Object Tracking"
subtitle:   CVPR2020, MOTS
date:       2020-06-17
author:     pshow
header-img: img/post-bg-debug.png
typora-root-url: /img
catalog: true
categories: codes
tags:
    - paper
    - codes
---
## Learning a Neural Solver for Multiple Object Tracking

### 主要贡献

- 基于Message Passing Networks (MPNs)提出一种全可微的框架

- 直接在图域上操作，使得本方法可以在整个检测集上直接作出推论。

  By operating directly on the graph domain, our method can reason globally over an entire set of detections and predict final solutions.

- We propose a novel time-aware neural message passing update step inspired by classic graph formulations of MOT.

- 网络对MOT的学习不仅仅局限于特征提取，而是可以深入到数据关联过程中

  we show that learning in MOT does not need to be restricted to feature extraction, but it can also be applied to the data association step.

### 简介

tracking-by-detection 分为目标检测和数据关联两步。目标检测有很多基于深度学习的检测器。数据关联往往被视为图分割问题。从图的观点来看MOT，节点代表着检测到的目标，边则意味着俩个节点有所关联。一条激活的边两端的节点属于同一个追踪轨迹。解决图分割问题也可以分为两步，首先计算每条边的cost，cost反映了两端的节点属于同一个轨迹的可能性。随后cost被用来获得最终优化后的分割图。

两个方向：优化图的表达，优化cost的选择与计算。

(i) learn features for MOT, and (ii) learn to provide a solution by reasoning over the entire graph.

并不成对的计算cost然后计算匹配，本文通过整张图直接预测最终的轨迹。

message passing network (MPN) 直接在图域上学习。

因此，我们的方法能够解释探测之间的全局相互作用，尽管依赖于一个简单的图表公式。

Hence, our method is able to account for global interactions among detections despite relying on a simple graph formulation.

We show that our framework yields substantial improvements with respect to state of the art, without requiring heavily engineered features and being up to one order of magnitude faster than some traditional graph partitioning methods.

### 相关工作

分两步，利用目标检测算法逐帧检测行人的位置，利用关联算法将统一轨迹时间线上的目标串联起来。

![image-20200616142015619](/img/Learning%20a%20Neural%20Solver%20for%20Multiple%20Object%20Tracking.assets/image-20200616142015619.png)

we propose to directly learn a solver and treat data association as a classification task,

we propose to leverage the common graph formulation of MOT as a domain in which to perform learning.

![image-20200616142750310](/img/Learning%20a%20Neural%20Solver%20for%20Multiple%20Object%20Tracking.assets/image-20200616142750310.png)

### 将追踪视为图问题

#### 问题陈述

![image-20200617085802930](/img/Learning%20a%20Neural%20Solver%20for%20Multiple%20Object%20Tracking.assets/image-20200617085802930.png)

![image-20200617090137318](/img/Learning%20a%20Neural%20Solver%20for%20Multiple%20Object%20Tracking.assets/image-20200617090137318.png)

![image-20200617090400877](/img/Learning%20a%20Neural%20Solver%20for%20Multiple%20Object%20Tracking.assets/image-20200617090400877.png)

将追踪问题分解为目标检测和图分割问题

#### 网络流程

![image-20200617091347538](/img/Learning%20a%20Neural%20Solver%20for%20Multiple%20Object%20Tracking.assets/image-20200617091347538.png)

一个检测节点只能被分配给一个轨迹

![image-20200617091449160](/img/Learning%20a%20Neural%20Solver%20for%20Multiple%20Object%20Tracking.assets/image-20200617091449160.png)

流守恒约束

![image-20200617091746579](/img/Learning%20a%20Neural%20Solver%20for%20Multiple%20Object%20Tracking.assets/image-20200617091746579.png)

每个节点最多能够与之前的一个节点与之后的一个节点相连

#### 从可学习损失到预测结果

将边的激活分配问题转化为01二分类问题

![image-20200617092214105](/img/Learning%20a%20Neural%20Solver%20for%20Multiple%20Object%20Tracking.assets/image-20200617092214105.png)

### 利用Message Passing Networks 来追踪

外观与几何信息在整个检测集中传播，模型可以拥有整张图的视野。

![image-20200617092857176](/img/Learning%20a%20Neural%20Solver%20for%20Multiple%20Object%20Tracking.assets/image-20200617092857176.png)

![image-20200617093119596](/img/Learning%20a%20Neural%20Solver%20for%20Multiple%20Object%20Tracking.assets/image-20200617093119596.png)

![image-20200617093222156](/img/Learning%20a%20Neural%20Solver%20for%20Multiple%20Object%20Tracking.assets/image-20200617093222156.png)

#### Message Passing Networks

节点属性和边属性利用Message Passing Networks 在整张图里传播。

![image-20200617093958428](/img/Learning%20a%20Neural%20Solver%20for%20Multiple%20Object%20Tracking.assets/image-20200617093958428.png)

每次信息传递过程包含两步，节点到边，边到节点。

![image-20200617094553080](/img/Learning%20a%20Neural%20Solver%20for%20Multiple%20Object%20Tracking.assets/image-20200617094553080.png)

#### 时间感知的信息传递

MOT任务基于时间戳，有着十分独特的图构造，需要对MPN进行特化，特别是在节点更新步骤。

如果利用所有相邻节点与边的信息来更新节点，则无法判断是否遵守了流守恒限制。

![image-20200617095306160](/img/Learning%20a%20Neural%20Solver%20for%20Multiple%20Object%20Tracking.assets/image-20200617095306160.png)

将对节点的更新修改为针对时间敏感的两部分，将过去的节点聚合在一起，将将来的节点聚合在一起，从而凸显时域上下文特征

![image-20200617095657219](/img/Learning%20a%20Neural%20Solver%20for%20Multiple%20Object%20Tracking.assets/image-20200617095657219.png)

输入始终包含初始节点信息与初始边信息

![image-20200617100124345](/img/Learning%20a%20Neural%20Solver%20for%20Multiple%20Object%20Tracking.assets/image-20200617100124345.png)

#### 特征编码

由其他网络获得初始化信息

The initial embeddings that our MPN receives as input are produced by other backpropagatable networks.

 ##### 特征嵌入

![image-20200617100643047](/img/Learning%20a%20Neural%20Solver%20for%20Multiple%20Object%20Tracking.assets/image-20200617100643047.png)

##### 几何嵌入

![image-20200617100945494](/img/Learning%20a%20Neural%20Solver%20for%20Multiple%20Object%20Tracking.assets/image-20200617100945494.png)

#### 训练与推理

将最终迭代的边的信息作为二分类网络的输入，带权重的二分类交叉熵损失，由于是稀疏图，正例的权重要增加。

![image-20200617101519772](/img/Learning%20a%20Neural%20Solver%20for%20Multiple%20Object%20Tracking.assets/image-20200617101519772.png)

![image-20200617102211989](/img/Learning%20a%20Neural%20Solver%20for%20Multiple%20Object%20Tracking.assets/image-20200617102211989.png)

![image-20200617102225940](/img/Learning%20a%20Neural%20Solver%20for%20Multiple%20Object%20Tracking.assets/image-20200617102225940.png)

### 实验

#### 数据集与评价指标

![image-20200617103307127](/img/Learning%20a%20Neural%20Solver%20for%20Multiple%20Object%20Tracking.assets/image-20200617103307127.png)

#### 实施细节

![image-20200617104143977](/img/Learning%20a%20Neural%20Solver%20for%20Multiple%20Object%20Tracking.assets/image-20200617104143977.png)

![image-20200617105120275](/img/Learning%20a%20Neural%20Solver%20for%20Multiple%20Object%20Tracking.assets/image-20200617105120275.png)

#### 消融学习

![image-20200617105404887](/img/Learning%20a%20Neural%20Solver%20for%20Multiple%20Object%20Tracking.assets/image-20200617105404887.png)

![image-20200617105952933](/img/Learning%20a%20Neural%20Solver%20for%20Multiple%20Object%20Tracking.assets/image-20200617105952933.png)

![image-20200617110415026](/img/Learning%20a%20Neural%20Solver%20for%20Multiple%20Object%20Tracking.assets/image-20200617110415026.png)

#### 跑分

![image-20200617110610017](/img/Learning%20a%20Neural%20Solver%20for%20Multiple%20Object%20Tracking.assets/image-20200617110610017.png)

### 补充材料

#### 贪心舍入

![image-20200617114558864](/img/Learning%20a%20Neural%20Solver%20for%20Multiple%20Object%20Tracking.assets/image-20200617114558864.png)

对于具有多输入多输出的节点，选择一条最高概率的输入边和一条概率最高的输出边

![image-20200617114733798](/img/Learning%20a%20Neural%20Solver%20for%20Multiple%20Object%20Tracking.assets/image-20200617114733798.png)

#### 精准舍入

![image-20200617134927689](/img/Learning%20a%20Neural%20Solver%20for%20Multiple%20Object%20Tracking.assets/image-20200617134927689.png)