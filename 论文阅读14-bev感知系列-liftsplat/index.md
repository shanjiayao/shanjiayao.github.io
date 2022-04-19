# 论文阅读13 BEV感知系列-LiftSplat



本文是了解BEV感知系列的第二篇论文阅读，来自Nvidia，Lift是指将相机表达提升到三维视锥，Splat指将各个相机的三维视锥通过外参以及PointPillar，拍平到BEV视角。通过独特的lift过程实现二维到三维的映射。

<!--more-->

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220419203757.png)


## 简介

-   论文：《Lift, Splat, Shoot: Encoding Images from Arbitrary Camera Rigs by Implicitly Unprojecting to 3D》
-   作者：[Jonah Philion](https://scholar.google.com/citations?user=VVIAoY0AAAAJ&hl=en), [Sanja Fidler](http://www.cs.toronto.edu/~fidler/)
-   机构：_NVIDIA, Vector Institute, University of Toronto_
-   论文水平：_ECCV 2020_
-   关键词：**Perception && BEV segmentation**
-   论文链接：[paper](https://arxiv.org/abs/2008.05711)  [project](https://nv-tlabs.github.io/lift-splat-shoot/)


## TL;DR

[总结与思考](#总结与思考)

## 摘要

自动驾驶汽车感知的目标是从多个传感器中提取语义表示，并将这些表示融合到一个单一的“鸟瞰”坐标系中，以供运动规划使用。

我们提出了一种新的端到端架构，它直接从任意数量的摄像机中提取给定图像数据的场景的鸟瞰图表示。我们方法背后的核心思想是将每个图像单独“提升”为每个相机的特征截锥体，然后将所有截锥体“分解”到栅格化的鸟瞰网格中。通过在整个摄像机装置上进行训练，我们提供了证据，证明我们的模型不仅能够学习如何表示图像，而且能够学习如何将来自所有摄像机的预测融合到一个单一的场景表示中，同时对校准误差具有鲁棒性。在诸如对象分割和地图分割等标准鸟瞰任务中，我们的模型优于所有基线和先前的工作。

为了追求学习运动规划的密集表示的目标，我们表明我们的模型推断的表示通过将模板轨迹“拍摄”到我们的网络输出的鸟瞰成本图来实现可解释的端到端运动规划.我们将我们的方法与使用来自激光雷达的预言深度的模型进行基准测试。

## 讨论

对于如何将单个相机的范式扩展到多相机，最粗暴的方法就是每个相机做一次感知，然后根据内外参变换到三维坐标系进行感知融合。

本文探讨了关于多视角融合任务的一些必要特性：

- 平移等变性：输出目标的位置需要随着像素位置的改变而改变。
- 排列不变性：输出的感知结果不与多个相机的图像顺序有关。
- 关于ego的等距等变性：无论多个相机相对于ego的位置如何，ego的平移和旋转应该可以引起感知输出的平移和旋转。

## 主要贡献

- 提出了一种lift-splat的框架，满足上述提出的三种特性，通过预测图像中每个像素的深度分布，形成对一些深度歧义性位置的自适应能力。
- 提出了一种将候选轨迹shooting到参考平面的方法，用于可解释的端到端运动规划。

## 方法框架

本文侧重于lift和splat的部分，对于shoot而言，不做过多涉猎。

### 问题定义

给定多个相机的视角图像，以及相机到ego的外参以相机自身的内参。需要将感知结果映射到BEV平面，并且不需要提前预知深度信息。

### overview

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220419194639.png)

模型将 n 个图像（左）及其相应的内外参作为输入。 在“lift”步骤中，为每个单独的图像（左中）生成一个视锥状点云。 然后使用外参和内参将每个截锥体“splat”到鸟瞰平面上（中右）。 最后，BEV视角下的CNN 用于 BEV 语义分割或规划（右）。

以单个相机的一张图像而言，lift的做法就是对每个像素预测C维语义特征以及D维深度特征，然后将D与C做外积，得到HxWxCxD的特征图，其编码了自适应的深度下的逐像素的语义信息。

而splat则是需要使用内外参，将ego自身周围的空间划分成若干个只有yaw角不同的视锥，其数量等同于相机的数量，然后将上述lift得到的每个相机的逐像素特征映射到ego周围的视锥空间中的每一个体素，最后用sumpooling对高度进行处理，得到BEV下的特征图。

### pipeline

因论文描述不够清晰，方法细节以代码为准，参考了代码中的[model.py](https://github.com/nv-tlabs/lift-splat-shoot/blob/master/src/models.py)，可以初步分为以下几部分:

- *CreateFrustum*：根据配置参数构建单个相机的视锥
	- 给出长宽以及深度，最终的输出是DxHxWx3，D表示深度，这里只是初步的对空间进行划分，而且是一个相机的视角，后续会对N个相机进行变换，以及范围筛选，得到视锥的形状。
- *GetGeometry*：根据外参内参等计算坐标变换，构建ego周围的几何空间。
	- 对上面的frustum进行坐标变换，简单来说就是内参外参以及六个视角的变换，输出的结果就是BxNxDxHxWx3，其中3表示坐标，B是batch_id，N表示相机个数，D则是不同深度。这样就等同于对ego周围的空间进行分块，水平来看，每一块都是一个视锥，每一个视锥都包含了D层深度以及HW的特征图大小。
- *CamEncode*：对原始图像进行逐像素地深度和语义的预测
	- 首先对原始图像过EfficientNet学习特征
	- 然后一层卷积直接出D+C的维度，D属于深度分布，C是语义特征
	- 对D维深度做softmax归一化
	- 将D与C做外积
	- 最终输出的是HxWxDxC
- *VoxelPooling*：将上述三者的输出作为输入，输出B,C,Z,X,Y，再对Z做切片，并cat到C的维度上。
	- 首先将特征reshape成MxC，其中M=BxNxDxHxW
	- 然后将*GetGeometry*输出的空间点云转换到体素坐标下，得到对应的体素坐标。并通过范围参数过滤掉无用的点。
	- 将体素坐标展平，reshape成一维的向量，然后argsort，在三个维度上都排排站好，拿到对应的索引。
	- 然后是一个神奇的操作，对每个voxel中的点进行sumpooling，代码中使用了*cumsum_trick*，巧妙地运用前缀和以及上述argsort的索引。输出是去重之后的Voxel特征，BxCxZxXxY。
	- 最后使用unbind将Z维度切片，然后cat到C的维度上。
- *BEVEncode*：用类似UNet的结构对上述的输出进行特征提取，输出应该是BxCxXxY

### Loss

loss的计算较为简单，真值来自于对框的坐标变换，将其变换到二维BEV视角，然后取底层的四个角点，连成四边形即可，真值实际上是一个二值图，长宽是BEV设定的参数，而每个格子的像素就是0或者1，1表示属于框的边。

而网络的输出则是，BEV视角下的，特征图，因此可以直接按照每个像素计算二元交叉熵损失。

## 实验

### 数据集

使用了NuScenes数据集，做了segmentation、robustness等试验项目，其中实验设计充分考虑了前面提到的三点特性，因此对平移、输入图像顺序以及旋转等各种情况做了实验测试。

1. 外参平移

![](https://nv-tlabs.github.io/lift-splat-shoot/imgs/sym.gif)

2. 外参旋转

![](https://nv-tlabs.github.io/lift-splat-shoot/imgs/rot.gif)

3. 图像顺序变化

![](https://nv-tlabs.github.io/lift-splat-shoot/imgs/perm.gif)

4. 图像平移

![](https://nv-tlabs.github.io/lift-splat-shoot/imgs/im.gif)

5. Bird's-Eye-View Segmentation IOU (nuScenes and Lyft)

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220419203051.png)

在Nuscenes数据集上进行在线建图的过程如下

![](https://nv-tlabs.github.io/lift-splat-shoot/imgs/nusc.gif)

此外，为了测试鲁棒性和泛化性，还做了camera dropout以及cross dataset的实验。具体见[视频](https://youtu.be/oL5ISk6BnDE)


## 总结与思考

对于二维向三维的转换，本文并没有直接通过IPM，而是通过对每个像素预测一组深度分布来实现，深度分布相对于直接预测深度值的好处在于，分布可以更鲁棒，在一些深度值具有歧义性的像素点上表现更好。

此外，代码中cumsum的部分很惊艳，苦苦思考之后发现原来sumpooling还可以这样，值得学习。

而对于BEV视角的元素学习，只需要给定BEV视角下的语义label，就可以将任务变成分类任务，学习BEV像素下的逐像素的语义，但是代码里貌似只给了box的label计算，而可行驶区域与车道线的并没有看到。
