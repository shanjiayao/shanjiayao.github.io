# 论文阅读之HorizonLiDAR3D




本文是CVPR2020Waymo挑战赛的冠军方案，相比于AFDet，在数据增广、网络深度以及模型Ensemble等方面都做了改进。

<!--more-->

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220323144555.png)


## 简介

-   论文：《1 st Place Solution for Waymo Open Dataset Challenge - 3D Detection and Domain Adaptation》
-   作者：_Zhuangzhuang Ding∗ Yihan Hu∗ Runzhou Ge∗ Li Huang Sijia Chen Yu Wang Jie Liao_
-   机构：_Horizon Robotics_
-   论文水平：_CVPR Waymo Challenge 2020_
-   关键词：**data aug && multi frame &&  ensemble**
-   论文链接：[paper](https://arxiv.org/pdf/2006.15505.pdf) 

## 摘要

本工作主要是基于AFDet,继续优化这种anchor free以及NMS free的网络,以挑战waymo数据集上的检测任务.相比于AFDet,本工作主要的修改方向包括:

### **data aug**

在AFDet的基础上进行了test time data aug,主要包括`point cloud rotation around pitch, roll and yaw axis`, `point cloud global scaling` and `point cloud translation along z-axis`

在每帧中加入了6个车辆、8个行人和10个自行车

### **network stronger**

网络结构如下

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220323144720.png)


在AFDet的基础上,使用了**稀疏卷积**,并且使用体素化替换了pillar,应该是为了保留z的特征细粒度.

grid size 0.04m, 0.04m, 0.1m along x, y, z axis respectively

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220323144738.png)


在上图的框架下,本工作在backbone后面加入了**RPN**网络,进而通过调整不同的backbone以及不同的rpn结构,形成了三种不同的网络版本,结构如下

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220323144843.png)


剩下的部分,head与loss应该与AFDet保持一致

### **first and second returns fusion in waymo**

waymo的数据集,激光雷达点云是双回波的,所以在这个版本中,为了加强点云密度,新增加了对第二次回波点云的使用.以及将5个lidar的点云都使用上了

### **multi frame accumulation [-4, 0]**

融合了多帧的信息,操作大概应该是位姿变换对齐,然后把每一帧的点云打上相对的时间戳.本工作是融合了前面四帧的点云.

### **utilizing image data: point painting**

这里借助了pointpainting的思想,将图像的信息融合进来.

对图像信息的使用包括两阶段,首先是使用box,第二阶段是使用语义分割的信息.

-   如果只有box,对点云特征增加一个维度,具体计算方式为: 将lidar的点投影到图像,如果点在box中,那么将box的预测得分赋给这个点,如果不在box中,得分为0
-   如果可以有seg的信息,那么将seg的语义特征加到点云后面

训练阶段,使用图像的真值,保存成点云.测试阶段,使用Cascade-RCNN对图像进行检测.

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220323144930.png)


### **model ensemble**

涨点利器

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220323145022.png)


## 总结

整体网络结构分为三部分,首先是体素化并且提取点云特征,转换到BEV视角,第二步是使用二维的backbone(也就是RPN)提取高维度特征,最终接五个head,作为三维box信息的预测以及编码.

本工作在AFDet的基础上,网络结构主要改进了backbone,替换掉原有的轻量化backbone,所谓的RPN,个人理解就是Encoder-Decoder的结构.并且在Decoder部分融合了多个分辨率的feature map.

此外的主要工作就是采用painting的方式融合了图像的信息,并且使用了ensemble涨点.这些操作都很值得借鉴






