# 07 Registration Benchmarks



本文是对Simon Lucey组一篇IROS2021工作的解读，其目标是评价lidar2map的配准方法在压缩地图上的性能。

<!--more-->


## 简介

-   论文：《Map Compressibility Assessment for LiDAR Registration》
-   作者：_Ming-Fang Chang, Wei Dong, Joshua Mangelson, Michael Kaess, and Simon Lucey_
-   机构：_CMU_
-   论文水平：_IROS2021_
-   关键词：**lidar2map && Registration**
-   论文链接：[paper](https://www.cs.cmu.edu/~kaess/pub/Chang21iros.pdf) 

## 摘要

本工作的目标是评估lidar-to-map的配准在压缩地图上的性能。现代自动驾驶汽车利用预先构建的 HD（高精度）地图来执行传感器到地图的配准，从而在位姿估计失败的场景进行重定位并减少大规模环境中的漂移。 然而，传感器到地图的配准通常是通过将传感器配准到一个密集的3D模型来实现的，对于高清地图而言,这会占用大量的存储空间，需要大量的数据处理开销。因此使用压缩地图进行配准是一个可行的方案，但具体使用哪种地图压缩方式才能保证点云拥有最佳配准性能，目前仍未有人探索这一问题。

综上所述，本工作提出了一种新颖且具有挑战性的基准，从三个角度评估现有的 LiDAR 到地图配准方法：地图可压缩性、鲁棒性和精度。我们比较了各种地图格式，包括原始点云、分层 GMM 和特征点，并在真实场景下的 LiDAR 数据集：KITTI odometry数据集和 Argoverse 跟踪数据集上展示了它们在可压缩性和鲁棒性之间的性能权衡。基准测试表明，当允许的地图尺寸上限很高时，最先进的基于深度特征点的方法明显优于传统方法。然而，当地图大小预算较低时，由于空间覆盖率差，在Argoverse 跟踪数据集中, 深度方法的性能低于使用更简单模型的传统方法。

## Motivation

地图对于现代自动驾驶系统至关重要。 具有丰富先验知识的地图提供了在线传感器无法观察到的有价值的离线精炼信息，从而提高了系统性能。 现代地图，例如自动驾驶汽车使用的高精地图，大多包含高质量的密集 3D 模型和语义标签。然而，这些密集的 3D 模型需要巨大的存储空间并导致额外的在线数据处理开销。

**密集的 3D 模型主要用于实现传感器到地图的精确配准**，这是自动驾驶车辆在姿态估计失败时重新定位地图的关键任务，同时也减少了大规模场景下的姿态漂移误差。**在实践中，除了重新定位之外，对于自动驾驶中的基本任务来说，密集的 3D 模型是不必要的**。例如，运动规划、运动预测、目标跟踪和避障只需要传感器输入和带有粗略 3D 信息的语义地图标签，例如车道方向和交通灯的边界框。由于高清地图中的其他信息尺寸要小得多，因此在点云配准的过程中如果可以消除对稠密的3D模型的依赖会显著减少高精地图的尺寸大小。

虽然消除对密集 3D 模型的需求是可取的，但大多数现有的点云配准研究仅关注配准具有相似数据分布的两次扫描的准确性和速度，而传感器扫描和地图的数据分布则大不相同。相关基准通过重建精度而不是传感器到地图的配准精度来评估点云压缩性能 [1]。事实上，如果可以使用较轻量化的地图实现准确的传感器到地图配准，则无需加载完美重建的稠密3D模型。尽管一些工作已经针对所提出的特定数据格式针对地图压缩率评估了传感器到地图的配准，但没有通用标准可用于在不同压缩地图格式之间进行公平的定量比较[2], [3].

在本文中，我们关注一个通用的设置——将 3D LiDAR 点云帧注册到 3D 地图，这是现代自动驾驶汽车执行传感器到地图注册的最常见配置。这种情况下的原始地图是离线构建的高质量、密集、大规模的点云。我们建议传感器到地图配准算法应该直接在某种压缩地图格式上运行，而不是原始点云，以消除存储和处理原始大比例点云的需要。我们在下面将此过程称为**压缩注册**。如图 1所示，所提出的压缩配准管道与使用原始点云图的方法相比有几个优点：

1）地图特征可以离线预先计算，因为它不需要任何在线输入。

2)  如果需要，在线地图数据解压缩花费的时间更少，因为它不需要恢复密集的 3D 地图。

3)  占用更少的存储空间和数据传输时间。

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/1.png)

## 主要贡献

1.  提出了第一个压缩式的 LiDAR-to-map的点云注册基准. 从三个角度评估现有的 LiDAR 到地图配准方法：地图可压缩性、鲁棒性和精度
    
2.  评估了最近的基于深度学习的和经典的点云配准方法，包括基于原始点、基于 GMM 和基于特征点的方法。 定量结果揭示了不同方法的优劣，并为未来的研究提供了有价值的参考。
    
3.  将向社区发布基准，以便将来方便地评估更多方法。

## 相关工作

参与对比的方法如下

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/2.png)

其中，Map type表示了注册算法处理的数据格式。Data dimension表示使用的数据所拥有的特征维度。Deep表示是否使用深度学习。Global表示此方法是否需要一个较好的初始位姿。Scalable表示注册算法能否适应大场景的map。

## 方法流程

### Overview

我们为各种压缩地图格式提出了一个用于压缩传感器到地图注册的通用基准。 提出的压缩配准管道如图 1 所示。在管道中，我们首先执行离线地图特征计算和压缩，使用嘈杂的初始姿势裁剪局部地图，然后将输入 LiDAR 扫描注册到裁剪后的压缩地图。 LiDAR 扫描被转换为评估配准方法中使用的相应格式，例如特征点或 GMM。 设 P 为输入 LiDAR 扫描，Q 为局部地图，T ∈ SE(3) 为包含旋转矩阵和平移的变换矩阵。 点云配准问题可以定义为：

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/7.png)

其中f表示点云变换函数, L表示损失函数. 损失函数 L 因不同方法而异。 例如，点对点 ICP 使用选定点对之间的欧几里德距离，而点对平面 ICP 使用从点到成对局部平面块的平方距离。 对于在其他格式而不是原始点云上操作的方法，将 φ(.) 表示为一般特征提取函数，等式(1) 变成

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/8.png)

其中φp(.) 和 φq(.) 不一定相同。例如，可以将原始点云注册到 GMM 模型。

### Benchmark构建

本工作提出的benchmark与已有工作的不同之处在于，**要匹配的两部分数据是不对称的。**lidar scan更加稀疏, 携带噪声以及包含了运动目标。然而map更加稠密，并且离线构建好的,经历了去噪了移除运动物体等操作。

为了模拟实际场景中的初始位姿误差，本工作应用了随机均匀噪声。在xyz三个方向上的误差区间为[-10, 10]m，在roll pitch yaw三个角度上的误差区间为[-10, 10]度。这样的随机误差可以再现现实场景中的绝大多数可能的初始误差。

为了评估图一中的pipeline，本工作使用了KITTI Odometry以及Argoverse Tracking数据集。大概的流程是先将数据集预处理成若干数据对，每一对数据中包含局部地图以及lidar的点云帧。其中局部地图被裁剪为初始位姿周围40m的范围大小，接着被压缩成对应的形式， 点云压缩的实例如下图所示:

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/6.png)

对于数据的预处理，我们对多帧lidar的点云进行聚合，以形成稠密的地图。由于KITTI中的真值噪声较多，我们使用SLAM替代了真值。而Argoverse中则是使用真值，对于移动物体的剔除，两个数据集同样存在差异。KITTI使用PVRCNN，Argoverse则是直接裁剪到可通行区域来移除车辆点。

### Evaluation Metrics

#### 1. Precision

该项主要评估平移和旋转的误差，令 R ∈ SO(3) 是一个旋转矩阵，t ∈ R3 是一个来自 T 的平移向量。我们通过以下方式测量精度：

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/10.png)
![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/11.png)

#### 2. Success Rate (SR)

平移和旋转误差都低于指定的阈值的数据对所占总数据对的比例。 我们选择平移误差阈值为 2m, 旋转误差阈值为 5°

## 结果讨论

在我们的实验中，我们使用裁剪的地图作为目标，使用 LiDAR 扫描作为源。 对于非对称方法，切换源和目标可能会影响结果。 在我们评估的非对称方法中，我们使用从 GICP 和点到平面 ICP 的地图计算出的法线，因为地图更密集且噪声更少。 对于 HGMR，我们在地图上构建 GMM 树，因为 GMM 树的压缩比比对原始点进行下采样更重要。 我们还为 FilterReg 交换了源和目标，发现它的性能比当前配置差

对于基于点的方法，如果配准方法未提供分数，我们会随机对点进行下采样。 还可以将其他下采样技术与评估方法一起应用以提高性能。

我们基于特征点的方法的结果表明，特征提取和对应过滤方法对于最终性能都至关重要。 TEASER++ 通常优于经典的 RANSAC，但只有在我们的地图大小预算下与深度描述符一起使用时才能发挥最佳效果。

我们还注意到，当地图尺寸较小时，基于深度学习的方法失败主要是因为目标地图是基于特征点的，并且在密集下采样后过于稀疏。 基于深度形状模型的方法可能会增加下采样地图的空间覆盖范围并提高性能。 在不牺牲全局特征匹配精度的情况下进一步降低特征维度也值得更多研究。

### 定量实验

下面两个表格是分别在KITTI以及Argoverse数据集上的定量结果。SR表示成功率，TE表示平移误差，RE表示旋转误差。SR10 是指在给地图大小预算 10 字节/平方米的情况下测得的成功率。其他同理。

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/4.png)
![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/5.png)

下表是成功率曲线。 我们观察到，在 KITTI 和 Argoverse 中，在地图大小 = 1 字节/平方米的情况下，FCGF (TEASER++) 和 HGMR (L2) 的表现优于所有其他方法。

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/3.png)

## Reference

[1] Cao, C., Preda, M., & Zaharia, T. (2019). 3D point cloud compression: A survey. Proceedings - Web3D 2019: 24th International ACM Conference on 3D Web Technology. https://doi.org/10.1145/3329714.3338130

[2] Bai, X., Luo, Z., Zhou, L., Fu, H., Quan, L., & Tai, C. L. (2020). D3Feat: Joint learning of dense detection and description of 3D local features. Proceedings of the IEEE Computer Society Conference on Computer Vision and Pattern Recognition. https://doi.org/10.1109/CVPR42600.2020.00639

[3] Yin, H., Wang, Y., Tang, L., Ding, X., Huang, S., & Xiong, R. (2021). 3D LiDAR Map Compression for Efficient Localization on Resource Constrained Vehicles. IEEE Transactions on Intelligent Transportation Systems. https://doi.org/10.1109/TITS.2019.2961120

