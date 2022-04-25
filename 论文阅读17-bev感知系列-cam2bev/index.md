# 论文阅读17 BEV感知系列-Cam2BEV



本文是了解BEV感知系列的第五篇论文阅读，来自亚琛工大，其针对IPM投影算法中的前景前景遮挡问题进行探究，标注遮挡类别，让网络预测遮挡类别，代码已开源。

<!--more-->


## 简介

-   论文：《A Sim2Real Deep Learning Approach for the Transformation of Images from Multiple Vehicle-Mounted Cameras to a Semantically Segmented Image in Bird’s Eye View》
-   作者：Lennart Reiher and Bastian Lampe, Lutz Eckstein
-   机构：_亚琛工大_
-   论文水平：_ITSC 2020_
-   关键词：**Perception && BEV segmentation**
-   论文链接：[paper](https://github.com/ika-rwth-aachen/Cam2BEV)  [code](https://github.com/ika-rwth-aachen/Cam2BEV)


![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/teaser.gif)

## TL;DR

[总结与思考](#总结与思考)

## 摘要

准确的环境感知对于自动驾驶至关重要。当使用单目相机时，环境中元素的距离估计是一个重大挑战。将相机视角转换为鸟瞰图 (BEV) 时，可以更轻松地估算距离。对于平面，逆透视映射 (IPM) 可以准确地将图像转换为 BEV。车辆和易受伤害的道路使用者等三维物体会因这种变换而变形，从而难以估计它们相对于传感器的位置。

本文描述了一种从**多个车载摄像头**获取给定图像的校正 360° BEV 图像的方法。**校正后的 BEV 图像被分割成语义类别，并包括对遮挡区域的预测**。神经网络方法**不依赖于手动标记**的数据，而是在**合成**数据集上进行训练，使其能够很好地推广到真实世界的数据。通过使用语义分割的图像作为输入，我们减少了模拟和真实世界数据之间的现实差距，并且能够证明我们的方法可以成功地应用于现实世界。对合成数据进行的大量实验证明了我们的方法与 IPM 相比的优越性。

## 讨论

与前面说过的VPN类似，都将语义分割结果作为输入，以减少sim2real的gap，而本工作针对IPM的改进主要体现在动态物体方面，与LookAroundObjects不同，本文是预测动态物体，而LookAroundObjects则是通过mask，去除动态物体，进而让网络去自主学习遮挡下的背景深度。

不过比较好奇的是，本文没有引用VPN以及LookAroundObject，不知道是否是因为投稿会议的领域不同。

## 主要贡献

- 一种将多相机BEV语义分割的方法，通过输入校正的透视语义分割图像，以及针对前景预测的改进的IPM算法，将透视转换为BEV视角。
- 设计了两个变种网络
- 不需要BEV视角的手工标注即可训练网络

## 方法框架

### 问题定义

本文一开始讨论了IPM作为常见的pv2bev的方法，其虽然受到前景动态目标的影响，但其转换投影的过程是合理的，因此我们应该倾向于关注，如何处理运动目标以减少IPM的投影误差。

### 如何解决前景目标的遮挡问题？

本文通过额外引入一种语义类别来引导网络对遮挡区域进行预测，额外的类别被定义遮挡那个，对应为透视视角下的被遮挡部分。具体的操作是对BEV下的真值做预处理实现的，对四个相机各自视角中前景物体遮挡的区域边界形成射线，然后在BEV中将对应的区域定义为遮挡类别。

定义遮挡区域的原则如下：
1. 如building、truck，认为他们永远会产生视线遮挡
2. 比如road，永远不会产生视线遮挡
3. cars类则是会遮挡一些比他低的类别
4. 被部分遮挡的目标拥有完整的可见性
5. 只有在所有相机的透视视角中都被遮挡了，目标才会被定义为*occluded*类别

### pipeline

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220424203326.png)

1. 输入
	1. 4个相机的前景透视图像，先做校正以及语义分割，其中语义分割是为了保持仿真和现实场景的一致性。
2. 坐标系变换
	1. 图中的v就是具体操作，使用了之前的工作，Spatital Transformer，其中投影变换就是IPM
3. BEV语义输出，对应预处理过的BEV真值计算loss，通过对遮挡区域的预测来实现前景遮挡的处理。

## 实验

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220424203714.png)


![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220424203745.png)


## 总结与思考

本文与LookAroundObject类似，只不过定义了遮挡label，然后让网络预测，而投影变换的方式则是去除了动态遮挡的IPM。作为早期开源的代码工作，其对IPM的实现值得学习。

