# 论文阅读06 PVT


Transformer在CV任务中的应用已经得到了很多工作的验证，但是对于点云处理任务，Transformer的尝试还不是特别多，这篇文章就是以PVCNN为基础，将Transformer融入进来。

<!--more-->

## 简介

-   论文：《PVT: Point-Voxel Transformer for Point Cloud Learning》
-   作者：_Zhang Cheng1*, Haocheng Wan1∗, Xinyi Shen2, Zizhao Wu1†_
-   机构：_Hangzhou Dianzi University, University College London_
-   论文水平：_Arxiv_
-   关键词：**point-voxel && transformer**
-   论文链接：[paper](https://arxiv.org/abs/2108.06076) 

## 摘要

与卷积神经网络相比，近期一些基于纯Transformer架构的工作在点云领域的评价基准上取得了令人惊叹的准确性。然而，现有的点云Transformer的计算成本很高，因为点云数据的不规则性浪费了大量时间。为了解决这个问题，我们提出了稀疏窗口注意（Sparse Window Attention）模块来从非空体素中收集粗糙的局部特征，这不仅避免了计算昂贵的不规则数据结构和无效的空体素，而且维持了线性的计算复杂度。同时，为了收集有关全局形状的细粒度特征，我们引入了相对注意力（Relative Attention）模块，其针对物体刚性变换设计了相对位置编码。配备 SWA 和 RA，我们构建了称为 PVT 的神经架构，将两个模块集成到点云学习的联合框架中。与以前的基于 Transformer 和基于注意力的模型相比，我们的方法在分类基准上达到了 94.0% 的最高准确率，平均推理速度提高了 10 倍。大量实验也验证了 PVT 在Part Segmentation和Semantic Segmentation任务上的有效性（分别为 86.6% 和 69.2% mIoU）。

## 主要贡献

1.  提出了PVT的框架，同时使用Transformer增强基于点的特征提取以及基于体素的特征体素方案。
    
2.  提出了一种高效的局部注意力模块，名为SWA，结合了稀疏卷积以及Swin Transformer[1]中的shift window设计，使计算复杂度降低到线性。
    
3.  针对点云的多视角变换问题，使用相对位置编码来计算相对注意力（RA），从而对不同视角下的点云输入鲁棒。
    
4.  在点云分类、局部分割以及语义分割任务上均做了实验，证明方法的有效性。

## 方法流程

### 网络总览


![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220323195226.png)

如上图所示，网络输入一帧点云数据，经过若干组PVT模块提取特征之后，后面接多尺度的特征融合以及对应任务的输出头。这里每个PVT模块都包含了两个分支，分别从原始点云以及体素化点云两种形式上学习点云特征。其中作者指出，由于体素结构是规则且连续的，所以基于体素化的分支可以学习到局部上下文信息。而基于点的分支则更侧重于提取全局的细粒度特征，最终两分支的信息融合，以实现一个PVT模块的特征学习。

### 基于体素的分支

体素化分支主要解决的问题有两个，其一是降低Transformer中自注意力机制的计算复杂度，其二是解决点云稀疏性带来的大量空体素的问题。  

为了降低自注意力的计算复杂度，作者借鉴了Swin Transformer中划分window的思想，选择将三维体素网格划分为若干局部框，这样在每个局部框中计算自注意力，可以将计算复杂度降低到与体素数量呈线性关系的级别。复杂度的对比如下

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220323195416.png)

式1表示全局的自注意力的计算复杂度，式2表示划分局部框之后的计算复杂度，总体来说，划分局部框之后，计算复杂度与体素数量R^3成线性关系。

为了解决点云稀疏性的问题，作者借鉴了Sparse Conv的思想，通过维护一个哈希表，来保留非空体素的索引，加快了计算效率。如下图所示：

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220323195431.png)

此外，为了使不同的局部框之间进行信息交流，作者同样使用了Swin Transformer中的shift方案

### 基于点的分支

这里使用了一个通用的transformer的自注意力框架，以获得全局的点云特征，自注意力计算公式如下

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220323195537.png)

为了使网络与输入点云的角度以及位置信息解耦，作者使用了相对位置编码，对每个点云中的点，都计算其余所有点相对于当前点的曼哈顿距离，公式如下

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220323195600.png)

## 实验

### 定量实验

作者在点云形状分类，局部分割以及语义分割任务上都做了实验，效果均达到了SOTA。

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220323195647.png)

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220323195737.png)

### 定性实验

下面是S3DIS数据集上做的可视化定性分析

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220323195809.png)

### 消融实验

作者评估了基于点的分支(PB)、基于体素的分支(VB)、局部框平移操作(shifting)以及每个模块的重复次数(NPB)对模型精度的关系。

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220323195852.png)

## Reference

[1] Liu, Z., Lin, Y., Cao, Y., Hu, H., Wei, Y., Zhang, Z., Lin, S., & Guo, B. (n.d.). Swin Transformer: Hierarchical Vision Transformer using Shifted Windows. ICCV 2021.
