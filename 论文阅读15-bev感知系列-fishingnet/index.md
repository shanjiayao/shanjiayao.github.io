# 论文阅读15 BEV感知系列-FishingNet



本文是了解BEV感知系列的第三篇论文阅读，来自zoox，其相机到BEV的变换follow了VPN的做法，主要是引入了lidar与radar的数据做融合处理。

<!--more-->


## 简介

-   论文：《FISHING Net: Future Inference of Semantic Heatmaps In Grids》
-   作者：_Noureldin Hendy , Cooper Sloan, Feng Tian..._
-   机构：_Zoox_
-   论文水平：_CVPR 2020_
-   关键词：**Perception && BEV segmentation && Sensor Fusion**
-   论文链接：[paper](https://arxiv.org/pdf/2006.09917.pdf)   [talk](https://www.youtube.com/watch?t=1004&v=WRH7N_GxgjE&feature=youtu.be)


## TL;DR

[总结与思考](#总结与思考)

## 摘要

对于要在复杂环境中导航的自主机器人，从几何和语义上理解周围场景至关重要。现代自主机器人采用多组传感器，包括激光雷达、雷达和摄像头。管理传感器的不同参考框架和特性，并将它们的观察结果合并到一个单一的表示中会使感知复杂化。为所有传感器选择一个统一的表示可以简化感知和融合的任务。

在这项工作中，我们提出了一个端到端的pipeline，它使用自上而下的表示来执行语义分割和短期预测。我们的方法由一组神经网络组成，这些神经网络从不同的传感器模态中获取传感器数据，并将它们转换为一个通用的自上而下的语义网格表示。我们发现这种表示是有利的，因为它与特定于传感器的参考帧无关，并且可以捕获周围场景的语义和几何信息。因为多模态共享一个单一的BEV输出表示，它们可以很容易地聚合以产生一个融合的输出。并且可以扩展到其他任务。这种方法为多模态感知和预测提供了一种简单、可扩展、端到端的方法。

对于实验，在公开的NuScenes以及Lyft数据集上做了两种模态（相机与激光）的测试，在自己收集的数据集上做了三模态的融合。


## 主要贡献

- 提出了一个全新的多传感器融合感知的框架，最终的输出是在BEV视角
- 提出了一个使用Rader信息做短期预测的框架
- 提出的框架易于扩展到其他传感器

## 方法框架

主题框架可以分为两部分，分别是各传感器特征提取以及多传感器特征聚合。首先通过K个网络单独学习K个传感器模态的特征，其输入是一组过去的数据，输出是一组包含未来的BEV语义网格。然后再通过对多个模态的语义网格进行跨模态融合，输出最终的BEV语义map。

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220420100836.png)

### 问题定义

- 输入：K个传感器以及过去P帧的数据，当前时刻记为t0
- 输出：当前时刻t0以及未来f帧的BEV语义grid map
- map的语义类别：
	- vulnerable road users (VRU) including pedestrians, bicyclists and motorists
	- vehicles
	- background

输出语义地图的示例如下：

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220420111059.png)

### 多模态数据预处理


#### Lidar输入编码

直接对Lidar点云进行BEV投影，然后编码了8个维度的特征，分别是：

1. 占据信息，有点的网格为1，否则为0
2. Density，对每个网格中的点数密度做归一化
3. Z轴极大值，直接做类Pillar处理，仅保留z方向上的最大值
4. Z轴分层，将高度分为5层，0-2.5米的区间

将上述8个单独的特征图cat到一起，再经过ego的pose transformation，把过去p帧时间戳的信息变换到当前t0帧的坐标系下，然后使用了p+1个通道维护各自的时间戳信息。

所以最终的lidar分支输入是WxHx8x(p+1)，作者还十分贴心的做了可视化。abcd分别对应上述1234.

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220420111032.png)


#### Radar输入编码

类似lidar，同样编码成BEV视角下的特征，6个维度：

1. 网格占据状态，1表示有点，反之0
2. 经过ego 运动补偿之后的多普勒速度的XY值
3. Radar截面，Radar cross section (RCS)，属于传感器自身的特性，雷达截面积越小，雷达对目标的信号特征就越小，探测距离也越短
4. 信噪比，Signal to noise ratio (SNR).
5. 歧义的多普勒区间，Ambiguous Doppler interval.

对于时间戳的处理，同lidar，多帧pose向t0时刻对齐，并cat上时间的维度编码值。单帧的Radar点云如下图

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220420111013.png)


#### Camera输入编码

使用了多个相机均匀分布在ego周围，以覆盖360度的FOV，分辨率大小是192x320，同样cat并做时间编码。示例如下图：

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220420110946.png)


### 网络结构

整体采用编码解码的方式，不过相机的编码解码器与radar和lidar的不同，radar与lidar则共用一个编码解码网络。

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220420111342.png)

具体而言，radar与lidar的网络设计由于不需要考虑视角变换，直接应用类似U-Net的网络结构提取特征进而做编解码即可。而Vision部分则需要考虑如何将透视视角转换到BEV视角下面，所以需要独立出来考虑。具体的做法就是在encoder和decoder之间做了一层正交特征变换（orthographic feature transformation），细节部分参考了VPN的工作，正交层由一系列具有 ReLU 激活的无偏全连接层组成。 这提供了变换的非线性特性。 由于每个摄像机的参考帧不同，将每个摄像机视图传递给一个共享的编码器，将投影特征加在一起，然后将结果传递给一个解码器。 这种方法允许网络学习像素空间中的特征，然后可以将其解码为BEV视图输出。

关于多模态特征聚合部分，所有模态共享相同的融合框架，这里作者提出了两种融合方案，均应用在每个模态输出的softmax上进行计算，分别是平均池化以及优先级池化，平均池化类似于ensemble，这样可以减少输出的方差；而优先级池化则是对多模态的输出进行分级别处理，设定行人 > 车辆 > 背景。如果同一个像素对应的三模态传感器输出的不一致，那么优先选择最高优先级的类别作为当前像素的类别。

## 实验

由于使用了lidar，因此训练的label来自于lidar的标注数据后处理。

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220420112628.png)


![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220420112724.png)


## 总结与思考

本文是为数不多的带有Radar的多传感器融合工作，个人了解的还有一篇工作是赵行老师组的FUTR3D，个人理解本文的创新点是引入了lidar和rader的元素，并且构建了一个可扩展的网络框架，首先进行多模态各自的特征编码，然后对三者均使用encoder-decoder结构，得到BEV视角下的语义map，只不过相机分支follow了VPN的视角变换操作，使用MLP学习两视角之间的映射，这样需要保证输入输出分辨率一致。最后设计了两种多模态map的融合方式，分别是均值融合和优先级融合。

从结果来看，车辆的短期预测尚可，而且貌似不需要使用camera和radar，lidar自身便可以预测得到不错的结果。不过本文的设定是，只关注动态目标，而将所有静态目标作为背景处理，这一点对于构建语义map而言并不是特别友好。

