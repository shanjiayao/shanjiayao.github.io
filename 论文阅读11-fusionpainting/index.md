# 论文阅读11 FusionPainting



本文介绍了一种*lidar-camera*融合感知的方案，类似*pointpainting*，本工作叫做*fusionpainting*，通过*Adaptive Attention*融合两部分传感器特征。

<!--more-->

## 简介

-   论文：《FusionPainting: Multimodal Fusion with Adaptive Attention for 3D Object Detection》
-   作者：_Shaoqing Xu, Dingfu Zhou, Jin Fang, Junbo Yin, Zhou Bin and Liangjun Zhang_
-   机构：_Beihang University, Baidu Research_
-   论文水平：_ITSC2021_
-   关键词：**Sensor fusion && 3D Detection**
-   论文链接：[paper](https://arxiv.org/pdf/2106.12449.pdf)  [code](https://github.com/Shaoqing26/FusionPainting)

## 摘要

准确检测 3D 障碍物是自动驾驶和智能交通的一项重要任务。在这项工作中，我们提出了一个通用的*多模态融合框架 FusionPainting*，在语义级别融合 2D RGB 图像和 3D 点云，以提升 3D 对象检测任务。特别是，FusionPainting框架由三个主要模块组成：多模态语义分割模块、基于自适应注意力的语义融合模块和3D对象检测器。首先，基于 2D 和 3D 分割方法获得 2D 图像和 3D 激光雷达点云的语义信息。然后基于所提出的基于注意力的语义融合模块自适应融合来自不同传感器的分割结果。最后，将涂有融合语义标签的点云发送到 3D 检测器以获取 3D 对象结果。通过与三个不同的Baseline进行比较，已在大规模 nuScenes 检测基准上验证了所提出框架的有效性。实验结果表明，与仅使用点云的方法和使用仅绘制二维分割信息的点云的方法相比，融合策略可以显着提高检测性能。此外，所提出的方法在 nuScenes 测试基准上优于其他最先进的方法。代码将在 https://github.com/Shaoqing26/FusionPainting/ 上提供。

## Motivation

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220328155443.png)

相比于之前的PointPainting，本工作指出其具有一些潜在的问题，以上图为例，PointPainting的框架是将2D语义分割的结果映射到三维点云，进而输入检测器进行检测，但是这样存在的问题就是：**boundary-blurring effect**， 是指语义分割任务很难分割出准确的边界，因为一般语义分割都是Encoder-Decoder的过程，特征图从大到小再反采样到与输入一致，但是特征却只能通过线性插值来进行补全，所以就会出现边界模糊的问题，而在投影到三维点云之后，问题更为严重，如上图a，卡车的边界错误地将地面以及后面地背景都包含进来，导致c的检测结果。

所以针对上述问题，本文选择对点云也进行一次语义分割，这样至少可以通过点云分割结果抑制图像分割带来的边界模糊问题。而关于如何融合，作者选择使用Attention，以上就是本文的初衷所在。

## 方法框架

### Overview

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220328160338.png)

1. 点云语义分割，得到逐点的语义标签，使用Cylinder3D
2. 图像语义分割，然后通过传感器外参投影到激光点云，得到逐点的语义标签
	1. 图像语义分割一般只关注前景，那么地面、背景等如何处理？投影到三维点云之后，地面点和背景点的语义label是什么？
		1. 仔细想想应该是合理的，背景信息统一label为0，点云的分割也是如此，所以二者并不冲突。
3. 使用Adaptive Attention Module对两分支Painting的结果进行融合
4. 将融合后的点云输入Detector进行检测


### Adaptive Attention

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220328161351.png)

输入有三部分信息：a. 原始点云；b.点云语义分割结果；c. 图像语义分割投射到点云的结果；

1. Voxelization 将量级从N降到HxW，避免大量计算
2. 学习Attention Mask
	1. local feature learning
		1. 对体素化之后（其实就是pillar）的每个pixel过一遍pointnet，学习局部特征
	2. global feature learning
		1. 对全局HxW的特征图过一遍pointnet，学习全局特征，维度降到1xD
	3. local 和 global 逐像素cat到一起，global需要在HxW维度进行广播
	4. sigmoid学习每个位置的权重，作为最后的Attention Mask，感觉这里有点类似通道注意力，总之就是用特征算一个逐pixel的得分。
3. 使用Attention Mask对三部分输入做加权，等同于 Output = α * A + (1 - α) * B，其中A和B是两部分分割的特征，α是Attention Mask，最终再将Output与原始点云cat，输出的维度与输入一致。

## 实验

### 定量实验

在nuScenes上的结果

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220328162800.png)

值得一提的是，这片工作的结果与之前的[MVP](https://tianweiy.github.io/mvp/) 不相上下。

### 定性实验

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220328162934.png)


## 总结

本文旨在解决2D分割边界模糊带来的分类不准的问题，通过3D分割以及自适应注意力加权，达到了较为不错的精度。

从PointPainting开始，传感器融合感知现在已经有了很多的相关工作，本文只是其中之一，抛开论文提出方法的baseline不谈，貌似尚未出现标志性的工作来一统天下，下面放了此时(2022-03-26)的benchmark，以供参考。

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220328163244.png)

