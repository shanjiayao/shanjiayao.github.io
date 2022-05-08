# 论文阅读20 BEV感知系列-M^2 BEV



本文是了解BEV感知系列的第七篇论文阅读，来自港大、Nvidia等多方联合的工作，其提出了一种BEV视角下的检测与分割的统一框架，使用共享的主干网络，利用Lift的思想将多视角图像lift到3D空间，并对速度做了优化，最终实现高实时性以及高精度的性能。

<!--more-->


## 简介

-   论文：《M2BEV: Multi-Camera Joint 3D Detection and Segmentation with Unified Bird’s-Eye View Representation》
-   作者：[Enze Xie](https://xieenze.github.io/), [Zhiding Yu](https://chrisding.github.io/), [Daquan Zhou](https://scholar.google.com/citations?user=DdCAbWwAAAAJ&hl=en), [Jonah Philion](https://www.cs.toronto.edu/~jphilion/), [Anima Anandkumar](http://tensorlab.cms.caltech.edu/users/anima/), [Sanja Fidler](https://www.cs.utoronto.ca/~fidler/), [Ping Luo](http://luoping.me/), [Jose M. Alvarez](https://alvarezlopezjosem.github.io/)
-   机构：_The University of Hong Kong, NVIDIA, National University of Singapore, University of Toronto, Vector Institute, Caltech_ 
-   论文水平：_Arxiv 2022_
-   关键词：**Perception && Multi Task**
-   论文链接：[paper](https://arxiv.org/abs/2204.05088) |  [project](https://xieenze.github.io/projects/m2bev/)

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220508233448.png)


## 摘要

本文提出了 M2BEV，这是一个统一的框架，可以在鸟瞰 (BEV) 空间中与多摄像头图像输入联合执行 3D 检测和地图分割。与之前大多数单独处理检测和分割的工作不同，M2BEV 使用统一的主干网络，以提升运行速度。 M2BEV 有效地将多视图 2D 图像特征转换为 ego-car 坐标中的 3D BEV 特征。这种 BEV 表示很重要，因为它使不同的任务能够共享单个编码器。本文框架进一步包含了四种重要的设计，它们既有益于准确性又有益于效率：

1. 一种有效的 BEV 编码器设计，可减少体素特征图的空间维度
2. 一种动态框分配策略，使用匹配学习来分配带有锚点的真实 3D 框 
3. BEV 中心重加权，通过更大的权重来加强更远的预测
4. 大规模 2D 检测预训练和辅助监督。

结果表明，这些设计显著有利于缺少深度信息的基于任意相机的 3D 感知任务。 M2BEV 具有更好的内存效率，允许以更高的分辨率图像作为输入，并具有更快的推理速度。 nuScenes 上的实验表明，M2BEV 在 3D 检测和 BEV 分割方面均取得了最先进的结果，最佳单一模型在这两个任务中分别达到了 42.5 mAP 和 57.0 mIoU。

## 讨论

前面介绍很多BEV做segmentation的paper，也包含一篇BEV做detection的DETR3D，其实归根结底这两种任务都可以在BEV视角下做有机的融合，以进行多任务联合训练，而通用的多任务框架通常使用独立的主干网络，本文则是统一到了BEV视角下，如下图所示。

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220508233506.png)


## 主要贡献

- 一个联合统一的框架，可以利用环视多相机输入，将2D图像信息转换到3DBEV平面，同时进行3D检测以及分割任务，学习车道线、可通行区域以及前景运动目标等信息
- 高效的BEV encoder，dunamic box assignment以及BEV中心度加权机制，这些设计均针对GPU友好，且有效提升模型性能。
- 实验证明大尺度的与训练模型以及2D辅助监督训练有效地提升了模型精度，并且不影响测试推理实践。

## 方法框架

### pipeline

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220508233643.png)

1. 2D 主干网络
	1. 多相机的前景透视图像，通过ResNet和FPN网络提取每个相机的图像特征，这里通过segformer中用到的多尺度融合的方法，将多个尺度分别上采样然后cat到一起
	2. 此外，为了提升模型精度，作者还利用了training阶段添加Detection head进一步约束网络参数优化，这里仅在2D Encoder网络的训练阶段使用。
	3. 此外x2，作者还尝试了在nuImage大型数据集上预训练了Cascade Mask RCNN，其对应的主干网络包括ResNet-50以及ResNeXt-101，而后直接在M2BEV的网络中应用对应主干网络的参数即可，这可以显著提升模型性能。
2. 2D-3D模块
	1. 简单来说，就是Lift中对每个像素预测D维特征的阉割版，不单独预测D维深度分布，而是将同一像素的D维深度默认均匀分布，设定成一样的值
	2. 论文中给了一个图，对应的是图像一个像素对应3D空间Voxel中的一条射线，
		![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220509000007.png)

	3. 至于多个像素对应一个voxel的情况，论文里貌似没说如何处理，不过估计大概率MaxPooling或者SumPooling
1. 3D BEV Encoder
	1. 计算得到的3D Voxel下的坐标以及图像特征按位scatter之后，得到的shape应该是XxYxZxC的4D Tensor，这里作者提到，为了提高效率，将4D转换为3D，对应为XxYxZC，作者称为“Spatial to Channel” (S2C) operator，（个人理解这里应该会一定程度上吃到2D Conv的红利？以及的确是通过2D Conv降低了计算量）
2. Head
	1. 这里就直接利用BEV特征接对应任务的Head即可，值得注意的是，作者对不同任务提出了不同的优化操作
		1. 检测任务而言，对应dynamic box assignment
			1. 参考了“**FreeAnchor: Learning to match** **anchors for visual object detection“**. _NeurIPS_, 2019，用动态框分配策略
		2. 分割任务而言，对应BEV centerness
			1. 作者提出了一种观点，既2D-3D得到的BEV，其特征稀疏程度与距离ego car的远近正相关，因为对于相机而言，距离越远像素越少，那么投影到BEV后特征越少，因此为了平衡这种特征层面的不均衡，作者提出了中心度的概念，通过对边缘特征赋予更高的权重，让网络进一步关注远距离物体。中心度的示意图如下，范围从1到2
				![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220508235920.png)


			   

## 实验

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220509000225.png)

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220509000251.png)

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220509000259.png)

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220509000307.png)

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220509000326.png)


## 总结

本文提出了一种联合多任务框架，个人任务这会是BEV感知任务实际应用的潮流趋势所在，考虑到实时性以及内存等因素，检测分割等各自分别提取特征不太现实，所以联合、共享会是最优的解决方案，而本文通过各种方法优化，很好地将检测任务以及分割任务结合到一起，并提出了很多工程上较为使用的技术，值得参考。

有关缺点，文中提出了在一些大目标检测以及车道线分割场景中存在一定缺陷，感觉从Tesla AI Day的经验来看，貌似融合时序信息会是一种解决方案。
