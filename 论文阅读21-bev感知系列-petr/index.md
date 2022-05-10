# 论文阅读21 BEV感知系列-PETR



本文是了解BEV感知系列的第八篇论文阅读，来自旷视，其主要针对DETR3D进行改进，优化其object query部分，通过Lift的方式编码3D的position encoding，进而联合object query共同学习目标信息。

<!--more-->


## 简介

-   论文：《PETR: Position Embedding Transformation for Multi-View 3D Object Detection》
-   作者：[Yingfei Liu](https://arxiv.org/search/cs?searchtype=author&query=Liu%2C+Y), [Tiancai Wang](https://arxiv.org/search/cs?searchtype=author&query=Wang%2C+T), [Xiangyu Zhang](https://arxiv.org/search/cs?searchtype=author&query=Zhang%2C+X), [Jian Sun](https://arxiv.org/search/cs?searchtype=author&query=Sun%2C+J)
-   机构：_MEGVII Technology_ 
-   论文水平：_CVPR 2022_
-   关键词：**Perception && Position Embedding**
-   论文链接：[paper](https://arxiv.org/pdf/2203.05625.pdf) 


## 摘要

本文提出了PETR，其在DETR3D基础上深入设计了位置编码模块，在DETR3D中，其通过一组object queries隐式从2D图像中学习3D框的特征，在decoder中使用的key和value均来自于图像特征，而在这个过程中是没有位置编码信息的，所以本文的目的就是给图像的特征加上三维的位置编码，以更好地实现与queries的交互。

## 讨论
 
![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220509145211.png)

对于DETR系列任务中的位置编码，原始的DETR中直接使用二维图像的位置编码，而DETR3D则是没有具体设计位置编码，是通过投影的方式将3D转到2D进而实现了3D query与2D来自图像的key和value的特征交互。而本文则是选择通过位置编码，将2D的特征转移到3D空间，进而让特征交互、query索引的操作均发生在3D空间中，这样特征的尺度是对齐的。

## 主要贡献

- 提出了一种简单优雅的框架，将2D图像特征编码成3D的位置信息，然后通过3D的object queries与3D位置编码做交互，直接学习3D的预测框
- 将liftsplat中的2D-3D的方法应用在transformer框架中，形成了新颖的3D位置编码方案

## 方法框架

### pipeline

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220509153319.png)


1. 2D 主干网络
	1. 多相机的前景透视图像，通过ResNet网络提取每个相机的图像特征
2. 3D Coordinates Generator
	1. 简单来说，就是LiftSplat中的2D-3D操作，先对每个像素预测D维的深度分布，然后构建ego周围的Voxel空间网格，将相机视角的像素投影到3D空间中，如下所示
	    ![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220509153659.png)
		![](https://pic4.zhimg.com/v2-05652e9fb3b91619c962c76d403b4dab_r.jpg)
		
		个人理解，这里只是类似编码了ego周围的特征，通过划分voxel，以及内外参，将每个视角下的图像像素与voxel对应起来，但应该没有涉及到图像的特征信息。	
		此外，得到的3D坐标在各自xyz维度上均进行了归一化处理。
3. Position Encoder
	![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220509154122.png)

	1. 图像特征用1x1卷积对齐维度
	2. 3D的voxel坐标通过全连接学习特征
	3. 将二者相加，然后做flatten操作，得到每个点的位置编码特征。
	4. 在实际使用中，应该是直接将3D的voxel坐标特征做key_pos，然后将img的特征经过1x1 conv做key和value，object query则是以0初始化，一起送到后续decoder中的。并没有提前使用这个add操作
1. Decoder
	1. 标准的DETR的decoder，包括了L层decoder layers
	2. 通过输入的3D PE以及2D img 特征，做cross atte，然后输出目标类别以及回归量。
	3. 每一层的decoder都会输出，包含独立的分类和回归，但是没有迭代地对xyz进行更新。
	4. query的初始化参考了Anchor DETR的操作，通过预先设置一组可学习的anchor points，其服从0-1均匀分布
		1. 这个anchor detr也是旷视大佬们的团队，其本质是通过预初始化一组queries来解决随机初始化queries导致的模型收敛慢的问题。随机初始化的值导致模型无法迅速定位到queries表达的显式的意义，比较难优化。所以anchor points的提出就是为了赋予queries显式意义，加快模型收敛。详情可以参考[论文](https://arxiv.org/pdf/2109.07107.pdf)。
					   

## 实验

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220509162149.png)

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220509162200.png)

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220509162206.png)

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220509162219.png)

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220509162228.png)

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220509162243.png)


