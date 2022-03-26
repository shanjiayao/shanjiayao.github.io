# 论文阅读08 Rigid3DSceneFlow



现有的3D sceneflow方法大多是自监督，因为flow的gt太难获得，而本文是一篇关于sceneflow弱监督的工作，发表在CVPR2021。

<!--more-->

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220326160916.png)


## 简介

-   论文：《Weakly Supervised Learning of Rigid 3D Scene Flow》
-   作者：_Zan Gojcic,Litany Andreas,Wieser Leonidas,J. Guibas, Tolga Birdal_
-   机构：_ETH Zurich, Stanford University, NVIDIA_
-   论文水平：_CVPR2021_
-   关键词：**sceneflow && weakly supervised**
-   论文链接：[paper](https://openaccess.thecvf.com/content/CVPR2021/papers/Gojcic_Weakly_Supervised_Learning_of_Rigid_3D_Scene_Flow_CVPR_2021_paper.pdf) 


## 摘要

本文中，利用很多3D场景都能被表示成若干运动刚体运动这一观测，我们提出了一种数据驱动的scene flow估计算法。我们方法的核心在于能够通过考虑 3D 场景流与其他 3D 任务的结合在对象级别进行推理的深层架构。这种对象级抽象，使我们能够通过更简单的前背景分割和车辆自身运动来放宽对密集场景流监督信息的要求。使我们的方法非常适合最近发布的不包含密集的场景流注释的自动驾驶数据集。 作为输出，我们的模型提供了低级线索（如逐点的sceneflow）和高级线索（如刚性对象级别的整体场景理解）。 我们进一步提出了一个测试阶段的优化来改进预测的刚性场景流。 我们展示了我们的方法在四个不同的自动驾驶数据集上的有效性和泛化能力。代码开源，链接为：github.com/zgojcic/Rigid3DSceneFlow.

## 主要贡献

1.  本工作提出一种弱监督方案，利用刚体假设，将场景流物体转换为刚体运动的变换矩阵求解问题。这带来的优点就是，我们只需要前景和背景的分割信息以及传感器自身的运动变化信息，而不用稠密的逐点级别的对场景流做真值标注。
    
2.  基于数据驱动的方案，本工作取代了以往方法在点级别(point-wise)的场景流预测，而是先进行前景-背景分割，进而在实例级别对运动物体的场景流进行估计。此外，还在测试阶段提出了一种优化方案。
    
3.  我们的方法是一种新颖且灵活的主干网络，可以基于此网络来拓展各种下游任务。

## 相关工作

本文总结了现有的一些场景流预测的方法，根据监督信号的类型，可以分为两大类，自监督和无监督。

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220326160930.png)


此外，本文还总结了现有方法的两大缺点：

1.  无约束的点级别的场景流预测容易产生较大的误差，比如同一辆车上面的点，可能会有不同的场景流预测大小。
    
2.  有监督的方法需要大量的、稠密的标注数据，需要对点云中每个点都标注场景流真值才能训练出较好的结果。

## 方法流程

### 问题定义

给定来自同一观测传感器的连续的两帧点云数据，对应的时刻分别是T0和T1，根据刚体运动的假设，本文将室外场景中的运动目标都认定为刚体，那么场景流的估计任务是估计逐点的运动速度。对应地，这个问题就可以转化为估计每个刚体的刚体变换矩阵，矩阵由平移和旋转两部分得到，如下式：

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220326161024.png)

同理，由于传感器观测自身导致的静止/背景点的运动，同样可以通过对自身的刚体变换矩阵求解得到，对应地就是计算Tego。

### 方法概览

本文通过先分割，再分别求解前景和背景的刚体变换，来实现场景流预测的目的。其中前景需要先进行聚类，以得到单独运动的实例的点云，再估计变换矩阵。背景则是降采样后对两帧点云直接估计变换矩阵。网络结构如下：

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220326161118.png)


算法的整体流程为：

1.  输入两帧连续的点云，然后均经过前景背景分割网络，分别得到前景点云和背景点云。
    
2.  Instance cluster head，对得到的前景点云进行聚类，得到若干个运动的实例。
    
3.  Ego motion head，对得到的两帧背景点云进行自身刚体变换矩阵的估计。
    
4.  Scene flow head，根据两帧点云经过Encoder之后的相似性特征直接预测场景流信息。
    
5.  最后，根据场景流估计的值以及对聚类得到的每个实例进行刚体变换的矩阵估计。
    
6.  Loss部分，通过传感器自身的位姿变换以及前景背景分割的真值作为监督信息，计算对应的损失，迭代优化。

### 损失函数

整体的损失函数分为三部分，分别是前背景分割损失、自身运动估计损失以及前景实例运动估计损失，分述如下：

**1. 前背景分类损失**

由于使用了逐点的分割监督信息，所以直接使用分类较为常见的二元交叉熵损失

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220326161219.png)

**2. 自身运动估计损失**

由于背景点数量较大，所以为了减少计算量，需要先对两帧点云的背景点做下采样处理，采样点数为N=1024

运动估计损失的衡量是针对于刚体变换的矩阵，其中包括旋转**R**ego和平移矩阵**t**ego，优化的目的如下式：

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220326161242.png)

其中x表示第一帧点云中的点，Y表示第二帧点云，函数Φ表示在点集Y中找到和x匹配的点。对于变换矩阵**T**，可以使用Kabsch算法得出，对应的损失为

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220326161302.png)

**3. 前景实例运动估计损失**

对于分割得到的前景点，先使用DBSCAN聚类算法，得到若干实例级别的点云簇C，然后根据scene flow head输出的场景流预测V，来优化Kabsch算法得到的变换矩阵T，公式如下：

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220326161323.png)

## 实验结果

### 定量实验

作者为了验证backbone主干网络以及scene flow head部分的网络结构的有效性，先仅使用这部分网络做监督学习，对应的性能对比如下

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220326161501.png)


第二部分，使用了弱监督方法的性能对比如下

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220326161512.png)

### 消融实验

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220326161551.png)

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220326161604.png)

分别探究了网络的不同loss的作用，以及前背景分割和背景运动估计任务的作用。

### 定性实验

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220326161638.png)

