# 论文阅读10 IA-SSD





<!--more-->

## 简介

-   论文：《Not All Points Are Equal: Learning Highly Efficient Point-based Detectors for 3D LiDAR Point Clouds》
-   作者：_Yifan Zhang, Qingyong Hu, Guoquan Xu, Yanxin Ma, Jianwei Wan, Yulan Guo_
-   机构：_National University of Defense Technology, § University of Oxford_
-   论文水平：_CVPR2022_
-   关键词：**lidar segmentation**
-   论文链接：[paper](https://arxiv.org/pdf/2203.11139v1.pdf) 

## 摘要

本工作中，我们研究了 3D LiDAR 点云的有效目标检测问题。现有的基于原始点的算法框架，为了降低内存和计算成本，通常采用与任务无关的随机采样或最远点采样来对输入点云进行逐步的下采样，但并非所有点对目标检测任务都同等重要。特别地，对于目标检测器，前景点本质上比背景点更重要。受此启发，我们在本文中提出了一种高效的单阶段地基于点的 3D 检测器，称为 IA-SSD。我们方法的关键是利用两种可学习的、面向任务的、实例感知的下采样策略来分层选择属于感兴趣对象的前景点。此外，我们还引入了上下文质心感知模块来进一步估计精确的实例中心。最后，我们按照仅包含编码器的框架来构建我们的 IA-SSD 以提高效率。在几个大规模检测数据集上进行的实验证明了我们的 IA-SSD 的性能具有竞争力。由于低内存占用和高度并行性，它使用单个 RTX2080Ti GPU 在 KITTI 数据集上实现了每秒 80+ 帧的卓越速度。该代码可在 https://github.com/yifanzhang713/IA-SSD 获得。


## 主要贡献

1.  确定了现有基于点的检测器中的采样问题，并通过引入两种基于学习的实例感知下采样策略，提出了一种有效的基于点的 3D 检测器。
    
2.  所提出的 IA-SSD 是高效的，并且能够使用一个模型检测 LiDAR 点云上的多类对象。 我们还提供了详细的内存占用与推理速度分析，以进一步验证所提出方法的优越性。
    
3.  在多个大规模数据集上进行的大量实验证明了所提出方法的卓越效率和准确检测性能。

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/1.jpg)

## 方法流程

### 网络总览

我们的 IA-SSD 遵循 [1] 中使用的轻量级的，仅包含编码器的架构以提高效率。 输入的 LiDAR 点云首先被输入网络以提取逐点特征，然后是提出的实例感知下采样以逐步降低计算成本，同时保留信息前景点。 学习到的潜在特征进一步输入到上下文质心感知模块，以生成实例建议并回归最终的边界框。

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/Snipaste_2022-03-26_14-31-59.jpg)

### 可学习的下采样策略设计

对于高效的 3D 对象检测，必须通过渐进式下采样来降低内存和计算成本，尤其是对于大规模 3D 点云。 然而，大尺度下采样可能会丢失大部分前景目标的信息。 总体而言，目前尚不清楚如何在计算效率和前景点保留之间实现理想的权衡。 为此，我们首先进行实证研究，定量评估不同的抽样方法。

特别是，我们遵循常用的编码架构（即具有 4 个编码层的 PointNet++ [2]），并在下表中报告了不同采样方法的每层的 Instance Recall（即采样后保留的实例的比率）。方法包括随机点采样[3]、基于欧几里得距离（D-FPS）[2]和特征距离（Feat-FPS）[1]的FPS。

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/Snipaste_2022-03-26_14-33-02.jpg)

可以看出基于实例的采样方法可以很好地保留前景信息。下面介绍本文所设计的可学习的基于实例的下采样方案。

#### Class-aware Sampling

这种采样策略旨在学习每个点的语义，从而实现选择性下采样。 为了实现这一点，我们引入了额外的分支来利用潜在特征中的丰富语义。 具体做法就是，使用两个 MLP 层预测逐点的类别信息，Loss计算如下：

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/Snipaste_2022-03-26_14-32-14.jpg)

在推理阶段，取Top K个分数较高的点作为下采样的结果。

#### Centroid-aware Sampling

考虑到实例中心估计是最终目标检测的关键，我们进一步提出了一种质心感知下采样策略，为更接近实例质心的点赋予更高的权重。对中心度编码的公式如下

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/Snipaste_2022-03-26_14-32-44.jpg)

这种方式平滑了训练阶段的ground truth，通过计算中心度进行了加权。

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/Snipaste_2022-03-26_14-32-26.jpg)

在推理阶段，同样取Top K个分数较高的点作为下采样的结果。

### 利用上下文信息预测实例中心点

主要包含中心点预测、中心点聚合、候选框生成三部分。

#### Contextual Centroid Prediction

我们尝试利用边界框周围的上下文线索来进行质心预测。通过每个实例表面以及周围的点云信息，对当前实例的中心进行预测，loss如下：

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/Snipaste_2022-03-26_15-04-14.jpg)

#### Centroid-based Instance Aggregation

对于投票得到的的代表（质心）点，我们进一步利用 PointNet++ 模块来学习每个实例的潜在表示。 具体来说，我们将相邻点转换为局部规范坐标系，然后通过共享 MLP 和对称函数聚合点特征。

#### Proposal Generation Head

对上述的聚合特征进行处理，我们将候选框编码为中心点位置、尺寸和方向的多维表示。 最后，所有候选框都通过具有特定 IoU 阈值的 3D-NMS 后处理进行过滤。

## 实验结果

### 定量实验

作者在Kitti数据集上与现有方法进行了对比。

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/Snipaste_2022-03-26_14-33-33.jpg)

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/Snipaste_2022-03-26_14-34-02.jpg)

此外，还在Waymo和ONCE数据集上进行了实验。

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/Snipaste_2022-03-26_14-34-13.jpg)

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/Snipaste_2022-03-26_14-34-13.jpg)

### 定性实验

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/Snipaste_2022-03-26_14-34-32.jpg)

### 消融实验

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/Snipaste_2022-03-26_14-34-46.jpg)

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/Snipaste_2022-03-26_14-34-46.jpg)

作者评估了不同采样方式以及不同上下文中心点感知策略对模型精度的关系。

## Reference

[1] Zetong Yang, Yanan Sun, Shu Liu, and Jiaya Jia. 3dssd: Point-based 3D single stage object detector. In CVPR, pages 11040–11048, 2020.

[2] Charles Ruizhongtai Qi, Li Yi, Hao Su, and Leonidas J Guibas. Pointnet++: Deep hierarchical feature learning on point sets in a metric space. In NeurIPS, pages 5099–5108, 2017.

[3] Qingyong Hu, Bo Yang, Linhai Xie, Stefano Rosa, Yulan Guo, Zhihua Wang, Niki Trigoni, and Andrew Markham. Randla-net: Efficient semantic segmentation of large-scale point clouds. In CVPR, pages 11108–11117, 2020.
