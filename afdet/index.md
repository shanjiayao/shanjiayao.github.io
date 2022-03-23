# 论文阅读03 AFDet



本文是CVPR2020Waymo挑战赛的冠军方案的baseline，由地平线提出，整体结构与CenterPoint类似，是一种轻量化的baseline，其针对嵌入式平台做了优化，抛弃了传统的基于anchor的方法，将二维`centernet`以及`cornernet`的思想应用到三维，第一个提出三维`anchor free`以及`nms free`的方法。 后续地平线基于这个网络改进,提出了`HorizonLiDAR3D` 拿到了`2020 3D Detection and Domain Adaptation`的第一名

<!--more-->

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220323143601.png)

## 简介

-   论文：《AFDet: Anchor Free One Stage 3D Object Detection》
-   作者：_Runzhou Ge∗ Zhuangzhuang Ding∗ Yihan Hu∗ Yu Wang Sijia Chen Li Huang Yuan Li_
-   机构：_Horizon Robotics_
-   论文水平：_Arxiv_
-   关键词：**Anchor Free && One Stage**
-   论文链接：[paper](https://arxiv.org/pdf/2006.12671.pdf)


## 摘要

之前的三维点云检测工作,大多基于anchor的方案,anchor的缺陷主要体现在两方面:

-   为了提高recall.需要产生大量的anchor,对应地就需要使用NMS等后处理操作,大大拖延了算法运行速度
-   anchor的尺寸属于hyper parameter,需要手动调参

因此,本工作是第一个提出`anchor free`以及`NMS free`的一阶段检测器,称为`AFDet`.其对嵌入式系统较为友好.

## 方法

总体框架分为三部分,pillar编码/backbone/head

### pillar编码

使用了pointpillar的编码方式,对于一帧点云,提前定义好`P`个pillar,每个pillar中允许`M`个点,多则采样,少则补0.每个点编码9个维度的特征,具体参考[pointpillar](https://www.notion.so/PointPillars-e8d38cc1ef484587aa20ecdcee864e8c)的paper.

第二步,对于每个pillar中的点进行MLP学习,也可以理解为使用pointnet学习其特征,然后将点的特征传递给对应的pillar.

最后就是形成BEV视图.对应`WxHxF`,F就是特征的维度

### BACKBONE

为了保证轻量化,本工作对backbone进行了裁剪,只进行一次下采样和上采样,最终输出的特征图保证了与原图同样的尺度.但是特征层数变多了.

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220323143803.png)


### Heads

head是本工作较为重要的部分,但是并不是多么新颖的工作,思想基本来自centernet.

1.  heatmap cls head
    
    这部分是在BEV图上对bbox的中心点进行分类.输入是`WxHxF` 的特征.输出是`WxHxK` 的heatmap,K表示类别的数量.意味着每一类都有一个heatmap,其中每个像素都预测K个类别的概率.值越大,代表是对应类别的中心点的可能性越高.
    
    由于anchor free的方法,不能像anchor based的方法那样通过大量的覆盖来提高召回率,所以要求对中心点的预测要尽可能准,因此本工作对centernet进行了改进:
    
    ![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220323143852.png)

    
    原有的centernet给定中心位置的label就只有一个中心点是1,其他位置都是0,,这样会对分类任务及其不友好,因为正样本的比例太低了.所以后续工作的改进分为两种:
    
    1.  **高斯核** 如上图(b)以bbox的中心为中心,向四周扩散一个方形的区域,对应的label值递减
        
    2.  本工作的方案, 如上图(a) 对box中所有的像素都进行label编码, 主要分为三种情况
        
        ![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220323143929.png)

        
        其中d是当前像素相对bbox中心点的l2 norm,这种方式保证了box中所有像素都能有正样本的label覆盖.本文的消融实验也证明了这种方式会更好一些.
        
2.  heatmap offset reg head
    
    本工作第二个重点就是这个offset的回归,其作用是抵消转换BEV时的量化误差以及抑制上面heatmap分类网络对中心预测不准的情况.
    
    输入是`WxHxF` 的特征.输出是`WxHx2` 其中2代表两个方向x和y的offset.注意这个offset是在3D空间上的,并不是在二维BEV像素坐标上的.
    
    offset实际上就是弥补将bev上的真实bbox中心,进行坐标转换后,存在的从float形式变为int形式的精度损失,下图公式也表明了,预测的大O表示实际二维平面xy上的offset
    
    ![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220323144112.png)

    
    这里需要注意的是,作者为了抑制误差较大的offset,还对真实的bbox中心周围进行了编码,将其位置加入到loss计算中.上述公式的目的就是,如果预测的offset恰巧等于实际的offset,那么radius在两个方向都应该是接近于0,否则,越远离真实的bbox中心,radius带来的loss 越大,起到了正则化的作用.
    
3.  z reg head
    
    直接进行l1 loss的预测,找到heatmap预测的最大值的坐标对应的z预测值,作为当前box在三维空间中的中心店的高度z
    
4.  size reg head
    
    直接回归长宽高,,使用l1 loss
    
5.  Orientation predict head
    
    角度采用分bin的方式,将一周360度分为两部分,一部分是-7pi/6 - pi/6,另外一部分是-pi/6 - 7pi/6.两部分存在一定的重叠.对于yaw角的回归,需要8个标量,每个bin4个,其中两个预测softmax类别,另外两个预测相对于bin中心的角度,真实角度与bin中心角度的sin和cos的值.
    

### decode

上面5个head的信息可以帮助我们解码出三维bbox.对应有7个维度的信息,(x,y,z,w,h,l,yaw)

其中,x与y由heatmap cls head输出的预测与heatmap offset reg head输出的offset组成,具体计算方式为

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220323144200.png)


此外,其他信息均直接使用head预测的结果即可

### data aug

经典三板斧

1.  database 把所有真值框收集起来,然后选场景,再把gtbox以及对应的点云放进场景中,并且保持一定的数量比例
2.  随机rotate以及translate
3.  随机flip along z

## 实验结果

### kitti

pillar side length 0.16 m

max number of points per pillar 100

max number of pillars P = 12000.

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220323144254.png)


### waymo
![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220323144305.png)


