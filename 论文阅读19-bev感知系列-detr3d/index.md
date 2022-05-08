# 论文阅读19 BEV感知系列-Cam2BEV



本文是了解BEV感知系列的第六篇论文阅读，是MIT, CMU, THU多家单位联合的工作，其基于DETR的二维检测工作，在多视角3D检测任务上开创了一种全新的方案，利用object queries隐式地编码了2D-3D的投影信息，从而避免了深度预测以及IPM类方法的投影误差。

<!--more-->


## 简介

-   论文：《DETR3D: 3D Object Detection from Multi-view Images via 3D-to-2D Queries》
-   作者：[Yue Wang](https://arxiv.org/search/cs?searchtype=author&query=Wang%2C+Y), [Vitor Guizilini](https://arxiv.org/search/cs?searchtype=author&query=Guizilini%2C+V), [Tianyuan Zhang](https://arxiv.org/search/cs?searchtype=author&query=Zhang%2C+T), [Yilun Wang](https://arxiv.org/search/cs?searchtype=author&query=Wang%2C+Y), [Hang Zhao](https://arxiv.org/search/cs?searchtype=author&query=Zhao%2C+H), [Justin Solomon](https://arxiv.org/search/cs?searchtype=author&query=Solomon%2C+J)
-   机构：_MIT, CMU, THU, Li Auto, Toyota Research_ 
-   论文水平：_CORL 2021_
-   关键词：**Perception && Multi Camera Detection**
-   论文链接：[paper](https://arxiv.org/abs/2110.06922)  [code](https://github.com/WangYueFt/detr3d)


## 摘要

本文介绍了一种较为新颖的多相机3D检测框架，与现有的工作不同（MLP投影、IPM、深度预测），本文直接从二维的图像特征中回归三维框的信息，是一种Top-Down的思路，通过一组稀疏的3D object queries在多个视角图像特征中进行索引，进而实现queries的自我学习与更新。

使用object queries的好处在于，其直接针对检测任务的输出建模，并不需要从图像2D像素点或者实现2D-3D的操作来构建大量的预测框，并通过set-to-set二分图匹配损失约束构建的queries输出。此外，这种方案还避免了使用冗余的NMS，实现了NMS-free，这可以有效提升模型推理速度。

最终的结果，3D Object queries隐式地编码了从2D-3D的变换信息，并且避免了2D-3D的误差，使得DETR3D实现了不错的精度。

## 讨论

现有的2D多相机感知算法，前面已经介绍了一些，比如使用MLP投影的VPN，改进IPM的Cam2BEV，以及深度预测的LiftSplat等，这些方法均遵循显式地2D-3D的空间变换，但不可避免的会引入变换投影的误差。

另一方面，DETR的工作在图像检测领域其实引领了另外一种潮流，就是Top-Down的检测方式，其打破了现有工作bottom-up的范式，直接通过object queries学习图像场景中目标框的信息，queries通过与图像特征的交互不断更新和学习信息，再通过set-to-set的二分图匹配loss约束其参数更新的方向，而本文要做的就是，给object queries再加入一个新的任务，既隐式编码2D-3D的空间变换，使其直接从2D图像学习3D框，这是本文的出发点所在。

## 主要贡献

- 一种连接2D特征与3D框预测的方法，通过将预测得到的3D框反投影到每一个2D的相机图像中，这不需要受2D-3D投影误差的影响。
- 网络框架直接预测框，不需要NMS后处理

## 方法框架

### 问题定义

输入一系列图像，表示相同时刻不同相机的数据流，假定外参以及内参已知，目标3D真值框已知，需要根据图像信息预测每一个目标在3D空间中的位置以及类别，并且不需要使用点云数据。


### pipeline

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220507235101.png)


1. 输入
	1. 多相机的前景透视图像，通过ResNet和FPN网络提取每个相机的图像特征，
2. 迭代预测网络，包含L层
	1. 通过object queries预测目标的三维框中心点
	2. 通过内外参投影到每个相机图像坐标系
		1. 投影的点不一定会在所有相机上均可见，因此定义了一个bool值，对应的每个相机只考虑可见的点
	3. 通过三线性插值对特征进行采样
		1. 采样后将对应特征与object queries对应相加
		2. 这里的三线性插值，是对同一像素对应不对分辨率的FPN特征进行的，其实就是融合对应像素的多尺度特征。
	4. 使用Multi-head Attention对特征进行加权
3. 输出，计算loss
	1. set-to-set loss对预测的框与真值框进行二分图匹配
	2. focal loss对框的类别计算损失
	   

## 实验

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220508000220.png)
![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220508000225.png)
![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220508000235.png)

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220508000257.png)


## 总结

总体而言，个人认为本文最大的创新在于将DETR的object queries进一步扩展到了3D中，进而隐式编码位置变换的过程，DETR中的Top-Down的学习目标框的方式引领了新的潮流，而DETR3D则是进一步提出了一种避免预测深度的方法，直接从2D特征中预测3D框，简单粗暴。

而话说回来，直接通过object queries学习框的三维中心点，难免有些过于粗暴了，最近一篇PETR就针对这一点问题做了工作，其等同于LiftSplat以及DETR3D的联合版本，后续应该会出阅读笔记~
