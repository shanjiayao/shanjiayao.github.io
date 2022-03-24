# BEV感知公开课笔记



本文是对2022年3月23日将门创投的公开课分享的笔记记录，主讲人微赵行老师。

<!--more-->

## 简介
BEV感知的定义，是将原本透视视角的感知信息，投影到俯视视角（鸟瞰图，BEV），这里投影并不是结果级投影，而是多个传感器的中间特征投影到BEV，并且在BEV视图进行后续的感知。

关于BEV感知，之前小鹏汽车的技术分享会上，刘兰个川老师就已经指出过这个概念，并且其medium博客上专门介绍了相关的工作，其认为BEV感知会是以后自动驾驶感知的大一统方向，此外，Tesla的AI Day，也是表明了他们使用transformer取代原本的IPM，将多个相机的特征投影到vector space，这些都是BEV感知的例子。

本文主要介绍的就是，以视觉为中心的自动驾驶感知，BEV感知算法。

1. 为什么以视觉为中心？
	1. Under-explored
		1. 过往有大量的工作被提出，并且携带丰富的语义信息。
	2. Scalable
		1. 视觉数据将会是自动驾驶积累最多的数据，并且其数据压缩、存储、传输等算法也较为完善。
2. 过去的研究如何？

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220323234033.png)
1. 为什么要用BEV感知?
	1. 自动驾驶本身就是一个2.5D的问题，其对高度方向上关注度不高，因为车辆默认被定义在地面上运动。
	2. 一些人为的通过参数标定的投影，会带有一些传感器级别的误差，导致感知结果不精准。

## 介绍的主要工作

### HDMapNet

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220323234803.png)

#### Motivation

根据环视相机的数据，进行车周围的交通信息检测，比如车道线、路沿等。

对于传统的高精地图的构建，对应的pipeline如下，其需要数据采集车，搭载高线数的lidar、高精度的imu、rtk等，通过在实际道路场景进行数据采集之后，后处理将采集到的信息进行标注、拼接，以形成高精度地图。后面实际使用时，就可以加载高精地图进行定位了。

这样有三个主要的缺点：
-  整体的pipeline较为复杂
- 人力财力耗费严重，数据标注尤其明显
- 需要不断地进行地图维护和更新，循环往复

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220323235126.png)

因此，以上就是HDMapNet地motivation，其希望将建图地过程转化为感知识别地过程，其将原本冗余地框架进行了简化，并且不需要人工标注，低成本且自动化，并且不需要进行地图更新和维护。

要注意，这里地高精地图并不是整个城市级别的，而是ego周边环境的高精地图的实时构建，是为了满足自动驾驶的需要。

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220323235336.png)

#### Input/Output

总体而言，输入包括图像信息以及Lidar信息。输出则是矢量化的信息，如下图

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220323235639.png)

#### Network

具体地HDMapNet的结构如下：
- 首先是encoder，使用一些轻量化的backbone对输入的图片做提取
- 然后是每个相机的视角转换，这里只是说用了一个神经网络，但是没有具体表明，Tesla使用的是Transformer，而毫末智行使用的是常用的卷积，所以这里应该不需要具体指定，实现功能即可。
- 接着是多相机投影，将多个相机的BEV图通过外参投影到一张BEV视图中。
- 最后接BEV的Decoder

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220323235742.png)

如果是Lidar点云，则使用Pillar拍平，然后再使用Encoder-View Transform-Decoder。

具体的Decoder后面接不同的head，比如车道线的分割，实例的分割，以及线段的方向，最后对上述前两种信息进行后处理，得到instance mask，再结合方向，就可以拼凑成矢量化的局部高精地图。

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220324000451.png)

#### 结果

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220324000556.png)


超过了IPM、Lift以及VPR，并且融合激光和相机的效果最优。


### DETR3D 

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220324000755.png)

#### Related Work

- 伪Lidar 先预测深度，然后再进行3D的目标检测，两阶段，容易产生累计误差，也就是深度估计不准导致检测框不准
![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220324000824.png)

- 将3D检测投影到2D来做，速度快，但是没有利用到3D的几何信息。

#### Proposed Method

提出的DETR3D有以下几种优点

1. 检测过程发生在3D，信息利用更充分
2. 没有进行3D世界的预测和重建
3. 避免了NMS等后处理的需求。

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220324001154.png)

#### 实验

相比于FCOS3D，有了很大提升，作者给出的原因是，在2D图像中，物体距离ego较近的话，物体大多会被框的边缘截断，所以在3D空间可以明显提升检测的性能。

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220324001816.png)

### FUTR3D

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220324001939.png)
针对多种传感器的融合，比如Radar、Lidar以及Camera，提出一种新的解决方案。

#### Related Work

- MV3D 
- F-Pointnet

这些方案都比较依赖于我们对传感器的先验，那么是否有一种方案简单地实现多个传感器地融合呢？

#### Proposed Method

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220324002508.png)

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220324002531.png)


#### 实验

- FUTR3D可以有效地融合多传感器地优势
- 可以使用4线Lidar+camera达到不错地效果

## 总结与思考
赵行老师从建图、感知静态目标、感知动态目标三个角度来介绍他们地工作，实际上，对于局部高精地图的构建，貌似Tesla、小鹏、毫末以及赵行老师讲到的，方案都大同小异。而对于静态和动态目标地感知，DETR3D的这种方案的确会有效克服近距离截断问题，尤其是在多挂卡车的感知任务上，而多传感器的感知结果融合统一也是之前的一个难点，现阶段貌似Tesla、包括学术界都给出了一些答案。总体来讲，以BEV为中心的感知，的确会相对lidar的感知更具影响力，不过最终到底是lidar量产速度快还是camera算法迭代速度快，就需要我们一起等待了。
