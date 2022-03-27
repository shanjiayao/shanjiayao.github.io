# 论文阅读11 FusionPainting



本文介绍了一种*lidar-camera*融合感知的方案，类似*pointpainting*，本工作叫做*fusionpainting*，通过*Adaptive Attention*融合两部分传感器特征。

<!--more-->

## 简介

-   论文：《FusionPainting: Multimodal Fusion with Adaptive Attention for 3D Object Detection》
-   作者：_Shaoqing Xu, Dingfu Zhou, Jin Fang, Junbo Yin, Zhou Bin and Liangjun Zhang_
-   机构：_Beihang University, Baidu Research_
-   论文水平：_ITSC2021_
-   关键词：**Sensor fusion && 3D Detection**
-   论文链接：[paper](https://arxiv.org/pdf/2106.12449.pdf) 

## 摘要

准确检测 3D 障碍物是自动驾驶和智能交通的一项重要任务。在这项工作中，我们提出了一个通用的多模态融合框架 FusionPainting，在语义级别融合 2D RGB 图像和 3D 点云，以提升 3D 对象检测任务。特别是，FusionPainting框架由三个主要模块组成：多模态语义分割模块、基于自适应注意力的语义融合模块和3D对象检测器。首先，基于 2D 和 3D 分割方法获得 2D 图像和 3D 激光雷达点云的语义信息。然后基于所提出的基于注意力的语义融合模块自适应融合来自不同传感器的分割结果。最后，将涂有融合语义标签的点云发送到 3D 检测器以获取 3D 对象结果。通过与三个不同的Baseline进行比较，已在大规模 nuScenes 检测基准上验证了所提出框架的有效性。实验结果表明，与仅使用点云的方法和使用仅绘制二维分割信息的点云的方法相比，融合策略可以显着提高检测性能。此外，所提出的方法在 nuScenes 测试基准上优于其他最先进的方法。代码将在 https://github.com/Shaoqing26/FusionPainting/ 上提供。




