# 论文阅读14 BEV感知系列-VPN



本文是了解BEV感知系列的第二篇论文阅读，BEV感知是指将其他传感器或视角下的感知结果融合到鸟瞰视图，本文提出的网络结构通过MLP学习视角变换，进而融合多视角特征，以构建BEV的语义Map。

<!--more-->


## 简介

-   论文：《Cross-view Semantic Segmentation for Sensing Surroundings》
-   作者：Bowen Pan, Jiankai Sun, Ho Yin Tiga Leung, Alex Andonian, and Bolei Zhou
-   机构：_CUHK，MIT_
-   论文水平：_RAL2020_
-   关键词：**Perception && BEV segmentation**
-   论文链接：[paper](https://arxiv.org/abs/1906.03560)  [project](https://view-parsing-network.github.io/)


![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220414185948.png)

## TL;DR

[总结与思考](#总结与思考)

## 摘要

为了强化机器人的环境感知能力，我们引入了一种跨视图语义分割的任务以及一种名为VPN的视图解析网络。

在跨视图语义分割任务中，将输入的多相机的前向透视视角的图片投影到BEV视图，并输出像素级的位置。然而这项任务的严重问题在于，很难获取BEV视角下的训练监督信息。

所以本文提出的VPN在三维仿真环境CARLA中训练，并通过域适应技术转移到真实场景，最终在合成数据以及真实数据上进行联合评估VPN。

实验结果表明，我们的模型可以有效地利用不同视角和多模态的数据来生成BEV视图，进而在LoCoBot机器人上实验，进行感知和探索任务。

## 主要贡献

1. 提出了一种跨视角语义分割的benchmark，以灵活地融合多元数据感知周边环境。
2. 提出了VPN网络，有效地对输入前向视图做变换以及多视角的融合。
3. 通过域适应技术，将虚拟场景训练的模型应用到现实数据中。

## 方法框架

### 问题定义

对于跨视角语义分割任务，给定一个或多个前向视角的图像观测，算法需要输出BEV视角下的语义地图。其中，输入的观测可以是环视的多个相机以及不同的模态，比如RGB与深度图。BEV语义地图是通过相机在固定的高度从上而下俯视获取到的图像经过语义标注生成的。

### overview

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220418095102.png)

因本文任务的主要问题是缺乏真实场景下的BEV真值，所以本工作在虚拟环境中训练，并用domain adaptive技术前移到真实场景。

整体框架分为两阶段：

- 第一阶段是View Parsing Network (VPN)，在虚拟场景中完成，对输入的多视角图像进行编码解码的过程，其中编码器包括特征提取、多视角投影，以及BEV视图融合三部分。
- 第二阶段是domain adaptation，在实际场景中，做迁移学习。

### pipeline

- 输入：NxM个图像，N表示不同视角下的相机数量，M表示每个视角给出的模态，因为是在虚拟场景下，可以拿到深度信息，因此本工作的输入N为6，每个相机的水平FOV大于等于60度，M为2，包括RGB与深度。
- **VPN**
	- **Backbone**：多视角下不同的模态经过CNN提取特征，注意这里对**不同模态使用了不同的编码器**。
	- **Encoder**：提取到的NxM个特征图，经过VTM模块，做前向透视视角到BEV视角的domain 映射与合成
		- VTM包括两部分，**View Relation Module (VRM)** and **View Fusion Module (VFM)**
			- VRM先将输入的每张图像特征HxWxC做flatten操作，展开成HWxC，这样将二维图像展开成一维序列，然后使用relation module R学习透视视图以及BEV视图两两像素之间的映射关系。本文的R使用两层MLP作为简易的实验。得到的BEV视图的分辨率与透视视图保持一致，因此得到了HWxC的特征，再reshape成HxWxC，就得到了BEV视角下的坐标，以及对应的特征。***注意，这里的模块R（VRM），是对每个视角单独学习特征的。***
			- VFM对上述VRM输出的BEV视角下的不同相机的特征图进行合成，本文中的操作是将N个视角的特征图直接在C的维度上cat，以保证尺寸的一致性。
	- **Decoder**：对得到的BEV特征图进行解码，但是本文并没有给出详细的decoder结构。
- **Sim2Real Adaptation**
	- 上述VPN网络需要在虚拟环境中预先训练好参数，那么为了可以在实际场景中应用，本工作还训练了一个网络，用作知识前移。
	- **Pixel-level adaptation**
		- 为了让实际场景与虚拟场景的图像差异尽可能小，先对实际场景的图像作语义分割，得到语义图像，然后再做语义类别的映射，使实际的图像转换成虚拟训练的图像风格。
	- **Output space adaptation**
		- 使用了一种对抗性训练的方案，以对输出空间做结构化。这里的具体方案感觉作者没有写的特别清楚，根据框图猜测，应该是先将实际场景的图片做上一步的风格转换，论文中称之为source domain的语义图像，然后对实际场景直接做语义分割，得到target domain的语义图像，将二者输入Semantic VPN，输出两个映射好的BEV特征图，然后用二者计算对抗性损失。


## 实验

### 数据集

使用了两个虚拟合成数据集：House3D cross-view dataset 和 Carla cross-view dataset，以及一个实际场景数据集，NuScenes。

### 实验分析

本文没有与其他方法对比，而是自己构建了不同设置下的结构，看起来更像是消融。

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220418112900.png)

此外，还做了一些可视化，如下

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220418113119.png)

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220418113145.png)

最后还在机器人上做了部署，并且后续接了探索的算法进行了测试，在此不做赘述。


## 总结与思考

本文通过MLP做视角映射，通过domain adaptation做风格前移，以解决真实场景真值缺失的问题。

这里有一些思考和疑惑：

1. 关于视角映射，其映射后的BEV视角尺寸与前向透视的尺寸保持一致，不确定这样是否有必要。
2. 关于映射后多视角融合，论文里提到是直接cat特征，感觉应该有更好的方法。
3. 个人感觉，本文的写作不是很清楚，也可能是我理解不到位T.T

