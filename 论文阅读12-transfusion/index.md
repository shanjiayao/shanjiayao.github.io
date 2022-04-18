# 论文阅读12 Transfusion



继FusionPainting之后，本工作进一步探究Sensor Fusion做检测的方案，提出了一种soft-association，来映射点云和图像的特征信息，并且利用Transformer两阶段的decoder逐步检测目标。

<!--more-->

## 简介

-   论文：《TransFusion: Robust LiDAR-Camera Fusion for 3D Object Detection with Transformers》
-   作者：_[Xuyang Bai](https://arxiv.org/search/cs?searchtype=author&query=Bai%2C+X), [Zeyu Hu](https://arxiv.org/search/cs?searchtype=author&query=Hu%2C+Z), [Xinge Zhu](https://arxiv.org/search/cs?searchtype=author&query=Zhu%2C+X), [Qingqiu Huang](https://arxiv.org/search/cs?searchtype=author&query=Huang%2C+Q), [Yilun Chen](https://arxiv.org/search/cs?searchtype=author&query=Chen%2C+Y), [Hongbo Fu](https://arxiv.org/search/cs?searchtype=author&query=Fu%2C+H), [Chiew-Lan Tai](https://arxiv.org/search/cs?searchtype=author&query=Tai%2C+C)_
-   机构：_HKUST, Huawei, CUHK_
-   论文水平：_CVPR2022_
-   关键词：**Sensor fusion && 3D Detection**
-   论文链接：[paper](https://arxiv.org/abs/2203.11496)  [code](https://github.com/XuyangBai/TransFusion/)


## 摘要

Lidar和Camera自动驾驶中用于 3D 物体检测的两个重要传感器。尽管传感器融合在该领域越来越受欢迎，但对劣质图像条件（例如，不良照明和传感器未对准）的鲁棒性仍未得到充分探索。现有的融合方法很容易受到这些条件的影响，主要是由于**外参矩阵建立的 LiDAR 点和图像像素的硬关联**。通常，车辆传感器标定的外参信息很容易受到外界环境影响，而此前的传感器融合方案一旦无法提供精准的外参，就会造成语义信息的误匹配。

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220328215351.png)

所以本文，我们提出了 TransFusion，这是一种强大的 LiDAR 相机融合解决方案，具有**软关联机制**来处理劣质图像条件。

具体来说，我们的 TransFusion 由卷积骨干网和基于Transformer Decoder的检测头组成。解码器的第一层使用一组稀疏的对象查询从 LiDAR 点云预测初始边界框，其第二个解码器层自适应地将对象查询与有用的图像特征融合在一起，同时利用空间和上下文关系。 Transformer 的注意力机制使我们的模型能够自适应地确定应该从图像中获取的位置和信息，从而形成稳健有效的融合策略。

我们还设计了一种图像引导查询初始化策略来处理难以在点云中检测到的对象。 

TransFusion 在大规模数据集上实现了最先进的性能。我们提供了广泛的实验来证明它对退化的图像质量和校准误差的鲁棒性。我们还将所提出的方法扩展到 3D 跟踪任务，并在 nuScenes 跟踪排行榜中获得第一名，显示了其有效性和泛化能力。

## 主要贡献

1. 摆脱了以往算法对传感器之间外参精度的依赖，提出了一种软关联框架，其更加鲁棒
2. 将Transformer引入以提高模型性能
3. 我们为对象查询引入了几个简单而有效的调整，以提高图像融合初始边界框预测的质量。


## 方法框架

### overview

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220328215634.png)

1. lidar分支
	1. Backbone
		1. 经过主干网络，提取BEV视角下的特征
	2. Query 初始化
		1. 提出了Input-dependent初始化方案，可以通过浅层的网络学习到较好的特征，具体而言就是先用一个Centerpoint-like的结构预测每个前景在BEV视角下中心点的heatmap
	3. Transformer Decoder and FFN
		1. decoder的结构与DETR类似，以初始化好的Q和原始Lidar的BEV特征作为K和V，计算attention并自我加权，再使用FFN输出box的信息
2. camera分支
	1. Backbone
		1. 经过主干网络，学习图像的perspective视角的特征
	2. Image guidance
		1. 图像辅助寻找BEV下物体的中心
	3. fusion lidar tokens
		1. 将点云的输出作为Q，图像信息作为K和V，做第二阶段的回归，这里的decoder才真正用到了多传感器的信息，也算是cross domain的融合了。

### Input-dependent Query Initialization

展开来说，这里的input dependent，其实就是相对之前的工作来说的，作者指出，如DETR，Sparse RCNN，Deformable DETR，都是随机生成或者作为网络参数学习，进而产生Query的，这种叫做input independent，这需要额外的decoder stage才能拟合出较为准确的结果，而近期，Efficient DETR的工作提出了一种和输入相关的初始化策略，通过预测中心点的heatmap来实现浅层的decoder也可以达到多层decoder级联的结果。

具体而言，给定lidar的BEV视角的特征图，首先预测K个类别相关的heatmap，这里是有label约束的，K是数据集的类别数量，然后每个类别取TopN个结果，最后将KN个再取TopN，就得到了初始化的Query坐标，通过这个坐标，在Backbone出来的BEV特征索引对应位置的feature，输出是NxD的Query

此外，考虑到lidar世界中，物体的尺度都是绝对的，所以其尺度和类别是存在一定的关系的，为了将这种关系编码到网络中，作者还对Query加上了category embedding。通过MLP将onehot向量映射到D维。

### SMCA for Image Feature Fusion.

我们利用交叉注意机制在 LiDAR 和图像之间建立软关联，使网络能够自适应地确定应该从图像中获取的位置和信息。

具体来说，我们首先使用先前的第一阶段预测作为Query以及外参标定矩阵来识别图像中，Query所在位置，然后计算cross attention。

这里存在一个问题就是，BEV视角下的Query和Perspective视角下的K和V，存在一定的domain gap，所以作者提出了**spatially modulated cross attention (SMCA) module,** 大概意思应该是利用外参，将BEV视角下box的中心投影到图像，然后对cross attention的结果进行加权，这样可以抑制由于domain gap引起的错误注意力得分。

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220328230652.png)


### Image-Guided Query Initialization Since

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220328230711.png)

作者还提到，可以使用image的信息辅助，先对BEV特征做一下加权，然后再初始化Query。

## 实验

### 定量实验

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220328230818.png)

总体而言本工作的目的就是通过transformer暴力融合两模态数据，这种思路虽然可以带来精度的提升，但是最终实时性应该也会存在一定的负担。
