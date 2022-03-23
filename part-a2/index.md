# Part-A2


本文是shaoshuai的3D点云检测工作，发表在TPAMI，融合了点在box中的位置分布规律, 通过这个位置约束确定更精准的box。

<!--more-->

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220323145250.png)


## 简介

-   论文：《From Points to Parts: 3D Object Detection from Point Cloud with Part-aware and Part-aggregation Network》
-   作者：_Shaoshuai Shi, Zhe Wang, Jianping Shi, Xiaogang Wang, Hongsheng Li_
-   机构：_The Chinese University of Hong Kong, Hong Kong, China._
-   论文水平：_TPAMI 2021_
-   关键词：**intra-object part location**
-   论文链接：[paper](https://arxiv.org/pdf/1907.03670.pdf) | [code](https://github.com/open-mmlab/OpenPCDet)
-   视频链接：[presentation](https://www.bilibili.com/video/BV1E741177wr) 



## 摘要

LiDAR 点云的 3D 对象检测是 3D 场景理解中的一个具有挑战性的问题，具有许多实际应用。在本文中，我们将我们的初步工作 PointRCNN 扩展到一个新颖且强大的基于点云的 3D 对象检测框架，即部分感知和聚合神经网络（Part-A2 网络）。整个框架由部分感知阶段和部分聚合阶段组成。首先，部分感知阶段首次充分利用从 3D 真实框派生的免费部分监督，同时预测高质量的 3D 提议和准确的对象内部位置得分。同一提案中预测的对象内部分位置由我们新设计的 RoI 感知点云池模块分组，从而有效地表示对每个 3D 提案的几何特定特征进行编码。然后，部分聚合阶段通过探索池化的对象内部分位置的空间关系来学习对框进行重新评分并细化框位置。进行了广泛的实验以证明我们提出的框架的每个组件的性能改进。我们的 Part-A2 网络优于所有现有的 3D 检测方法，并通过仅利用 LiDAR 点云数据在 KITTI 3D 对象检测数据集上实现了新的最新技术。

## 什么是intra-object part location

通过判断每个 object 内各个点的坐标与 box的位置关系, 得到每个点在物体中的相对位置关系, 提供了一个位置上的约束

作者说,前景点也就是在框中的点,分布都是有规律的, 而背景点一般都是杂乱的, 所以可以通过这种点和box之间的关系来增强判别能力

## 如何表述 intra-object part location?

将当前的物体中的点云进行坐标变换, 如图, 一个顶点是 (0, 0, 0), 对角顶点是 (1, 1, 1)

三个轴的坐标都映射到 [0, 1] 之间, 这样物体中每个点的分布一定都是 在[0, 1] 之间的, 就可以用来计算每个点分布的 label

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220323145801.png)


## 如何用 intra-object part location?

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220323145825.png)


### 1. Part-aware stage

第一部分主要是将前景点分割出来, 并且预测前景点的 **intra-object part location** 还会初步预测 proposal

-   Sparse Convolution
    
    针对输入点云, 先体素化, 进行稀疏卷积
    
    ![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220323145919.png)

    
    可以看到, 体素化之后的网格与原始点云, 包含的信息几乎一致
    
    稀疏卷积相比于PointNet++, 会更快, 优化的更好
    
-   Semantic segmentation + Intra-object part location
    
    ![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220323145942.png)

    
    分割得到前景点
    
    预测前景点的 intra-object part location 信息
    
-   Genarate 3D proposals to **group** the intra-object part locations for each 3D ROI
    
    两种生成proposal的策略:
    
    1.  Anchor-free
        
        利用分割分数作为 proposal 的大概分数
        
        较为简单, 对于小物体较为友好, 不需要下采样
        
        ![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220323150008.png)

        
    2.  Anchor-Based
        
        将稀疏卷积编码之后的特征转换为 BEV 视角, 进而在俯视图上生成Anchor
        
        更高的 recall 更高的 performance
        
        ![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220323150208.png)



        

### 2. Part-aggregation stage

第二部分就是使用 intra-object part location 以及分割信息等, 对生成的 proposal 进行精调

第一阶段输出的信息有:

point-wise features

point-wise intra-object part locations

3D proposals

-   ROI-aware point cloud region pooling
    
    进行框的预测, 打分以及 refine
    
    PointRCNN的方法可能存在歧义性, 相同的点可能产生不同的框
    
    ![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220323150407.png)
    
    这里会保留空的体素, 这样就相当与间接保留了物体的形状信息
    

## 结果

三个方向预测的 part location 平均误差为 6.28cm

高度( Z )方向的误差一般较小, 容易预测一些

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220323150328.png)


**Failure Case**

背景点与前景点的区分, 背景点与前景点相似情况下, 容易误检测

3D IOU的评价对于定位结果较为敏感

检测准确率还是较高的, 只有4.5% 的概率误检测

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220323150432.png)


可视化结果

前景物体点的颜色越靠近 corner, 颜色越接近corner的颜色

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220323150603.png)

