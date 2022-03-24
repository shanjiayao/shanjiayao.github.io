# 论文阅读05 AFDetV2



AFDetV2是地平线在2021waymo挑战赛上夺冠方案。本文主要创新点有三：`self-calibrated block`, `keypoint auxiliary loss`,  `the IoU prediction branch`

<!--more-->

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220323142728.png)


## 简介

-   论文：《Rethinking the Necessity of the Second Stage for Object Detection from Point Clouds》
-   作者：_Yihan Hu*, Zhuangzhuang Ding*, Runzhou Ge* Wenxin Shao, Li Huang† , Kun Li, Qiang Liu†_
-   机构：_Horizon Robotics_
-   论文水平：_CVPR Waymo Challenge 2021_
-   关键词：**self-calibrated block && keypoint && IoU prediction**
-   论文链接：[paper](https://arxiv.org/pdf/2112.09205v1.pdf) 

## 摘要

本工作概括地对现有的点云目标检测方法做了总结.首先讨论单阶段和两阶段这两种主流方向.进而提出通过对比实验发现,单阶段也能达到和两阶段类似的性能,并且两阶段的第二个阶段的主要作用是对分数的再优化.根本作用是解决了分类和回归的misalignment.

综上,本工作基于之前的AFDet,提出了AFDetv2版本,主要引入了 IoU prediction branch来实现二阶段的作用,并且使用self-calibrated block以达到通道注意力以及扩大感受野的目的.此外,还在训练阶段使用了关键点的loss辅助约束模型.**总体实现了通过一阶段达到了两阶段的精度,并且保持了一阶段的实时性,被称为“Most Efficient Model”(55.86 ms)**

## 引言讨论

为什么需要两阶段来完成检测的任务?

主流观点有两个

1.  基于点的检测器均表示使用原始点云的信息可以弥补体素化等操作带来的精度损失或者感受野缺失.所以两阶段的检测器依靠两阶段实现了较高的精度
2.  另外一点从分类和回归的misalignment出发.由于分类和回归一般都是两个单独的分支,所以分类最高的预测并不一定与回归最准确的预测相匹配.

针对观点1,目前很多方法,都是基于体素化的方式,转换成BEV来做的,他们都实现了与原始点云方案的同等精度.比如centerpoint

针对观点2,本工作做了对比试验,在一阶段检测器后面单独加上分类或者单独加上回归,以探究第二阶段究竟是哪里在起作用.结果表明分类的部分对一阶段的分类结果做了二次优化,并引起了显著的精度提升.而回归网络则是无显著作用.

## 网络结构

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220323142812.png)


网络结构主要分为三部分,分别是voxel以及点云特征提取,BEV backbone以及heads

### Voxelization

同HorizonLidar3D.先进行体素化,取每个体素所有点的均值作为体素信息.然后使用稀疏卷积对体素化的点云提取特征.

### BEV Backbone

对于上述转换好的bev feature map,使用二维的backbone进一步学习特征.这里作者提出了self cali backbone.主要工作就是引入了**[SCNet](https://github.com/MCG-NKU/SCNet)**以增强一阶段检测器对特征的学习能力.网络结构如下.SCNet可以一定程度上增加网络的感受野以及实现通道级别的注意力机制.

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220323142905.png)


### Heads

相比于AFDet中的五个head,本工作为了实现一阶段检测器对类别的优化,额外引入了一个head进行IOU的预测,实现IoU-aware confidence score prediction.然后将IOU的score与heatmap的score进行相乘,以抑制那些与回归不匹配的分数.


![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220323143015.png)


此外,还在训练阶段引入了一个辅助的head,用来计算keypoint,包括中心点以及周围四个点.这部分在测试阶段会被禁止掉.只是起到训练时辅助约束模型的作用.

综上,共七个head,最终对所有head的输出做解码.得到三维的box

## 实验结果

### 单帧,不加ensemble waymo val set

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220323143102.png)

### waymo test set

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220323143047.png)


经过ensemble以及多帧之后,可以达到84,,差不多是3DAL的精度.tql

### 三个模块的消融实验

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220323143137.png)


其他实验略,详见paper
