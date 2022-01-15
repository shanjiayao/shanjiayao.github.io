# NuScenes数据集笔记



![https://github.com/nutonomy/nuscenes-devkit](https://camo.githubusercontent.com/25210c94eb51537bb7ad7f6bd87da0751416b0640c33a9942bb6e16054688792/68747470733a2f2f7777772e6e757363656e65732e6f72672f7075626c69632f696d616765732f726f61642e6a7067)




<!-- more -->


## 数据集简介

nuScenes数据集是由Motional（以前为nuTonomy）的团队开发的用于自动驾驶的公共大型数据集，其中，Motional是由现代汽车集团(Hyundai Motor Group)和Aptiv PLC的合资企业，在2020年成立后，将原有的nuTonomy团队进行了扩展。哦对，这个团队还是`PointPainting`以及`PointPillars`的作者。

数据集部分，目前全部的nuScenes数据集包含四部分，分别是nuPlan、nuScenes、nuImages以及nuReality，其中nuReality是用作VR(Virtual Reality)的数据集。

对于nuScenes这部分三维场景的数据集来说，

- 2019 年 3 月，发布了包含全部 1000 个场景的完整 nuScenes 数据集，其数据量大概是KITTI数据集的7倍左右。
- 2020 年 7 月，发布了 nuScenes-lidarseg。在 nuScenes-lidarseg 中，共有 32 种语义标签（即激光雷达语义分割）对 nuScenes 中关键帧中的每个激光雷达点进行注释。因此，nuScenes-lidarseg 在 40,000 个点云和 1000 个场景（850 个用于训练和验证的场景，以及 150 个用于测试的场景）中包含 14 亿个注释点。

### Features

这里只关注数据集中一些三维数据的feature，更多信息可以自行查阅[官网](https://www.nuscenes.org/).

![](https://www.nuscenes.org/static/media/data.9ef46c59.png)

- 传感器部署
	- `lidar x 1`
		-   20Hz  32 channels
		-   水平FOV360° ,  竖直FOV +10° 到 -30° 
		-   点云范围在 80m-100m左右, 最远70m左右仍可以保证点云具有 ± 2 cm 的精度
		-   每秒点云最多 ~139万个点
	- `radar x 5`
		-   77GHz的波频  13Hz 采样频率
		-   最远可测250m
		-   测速精度大概 ±0.1 km/h
	- `camera x 6`
		-    12Hz 采样频率
		-   1/1.8'' CMOS，1600x1200 分辨率
		-   像素编码格式为 Bayer8 
	- `imu x 1`
	- `gps x 1`
- 数据量信息
	- 1000个场景，每个场景20s，地点在波士顿和新加坡这两个城市；20s时长以显示各种驾驶操作，交通状况和意外行为
	- 140万帧相机的图片
	- 39万帧lidar点云
	- 覆盖23种类别的目标以及1.4M的手动标注的3D bounding boxes
	- 覆盖了32种类别以及1.1B个手动标注的lidar points
- 各传感器时间同步
	- 为了在 LIDAR 和摄像头之间实现良好的跨模态数据对齐，当顶部 LIDAR 扫过摄像头 FOV 的中心时，会触发摄像头的曝光。图像的时间戳为曝光触发时间；而激光雷达扫描的时间戳是当前激光雷达帧实现全旋转的时间。鉴于相机的曝光时间几乎是瞬时的，这种方法通常会产生良好的数据对齐。请注意，相机以 12Hz 运行，而 LIDAR 以 20Hz 运行。12 次相机曝光尽可能均匀地分布在 20 次激光雷达扫描中，因此并非所有激光雷达扫描都有相应的相机帧。

### Benchmarks

由于NuScenes包含了时序信息，所以除了常见的检测跟踪分割任务，还有一些轨迹预测以及规划的任务benchmarks，具体包括：`detection`、`tracking`、`prediction`、`lidar segmentation`、`panoptic`、`planning`


## 数据集下载

需要注册账号并登陆后才可下载，界面如下：

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220115105120.png)


## 数据集使用以及devkit说明

官方给了非常详细的使用教程，具体可以参考github上的[devkit](https://github.com/nutonomy/nuscenes-devkit)

### 数据集文件结构

我这里只下载了nuScenes部分的数据集，下载好的目录结构如下：

```bash
/data/sets/nuscenes
    samples	-	关键帧的传感器数据.
    sweeps	-	所有帧的传感器数据.
    maps	-	地图相关文件.
    v1.0-*	-	trainval对应的元数据以及标注文件，json格式
```

值得注意的是，nuScenes采用了token的方式，将传感器、时间戳、帧id等都抽象成token，对应地，上述目录结构中，sweeps以及samples中也是根据不同传感器的名称来存放的，如下所示。

```bash
.
├── CAM_BACK
├── CAM_BACK_LEFT
├── CAM_BACK_RIGHT
├── CAM_FRONT
├── CAM_FRONT_LEFT
├── CAM_FRONT_RIGHT
├── LIDAR_TOP
├── RADAR_BACK_LEFT
├── RADAR_BACK_RIGHT
├── RADAR_FRONT
├── RADAR_FRONT_LEFT
└── RADAR_FRONT_RIGHT
```

### 架构中的基础概念

此外，nuScenes中的一些基础概念如下：

1.  `log` - Log information from which the data was extracted.
2.  `scene` - 20 second snippet of a car's journey.
3.  `sample` - An annotated snapshot of a scene at a particular timestamp.
4.  `sample_data` - Data collected from a particular sensor.
5.  `ego_pose` - Ego vehicle poses at a particular timestamp.
6.  `sensor` - A specific sensor type.
7.  `calibrated sensor` - Definition of a particular sensor as calibrated on a particular vehicle.
8.  `instance` - Enumeration of all object instance we observed.
9.  `category` - Taxonomy of object categories (e.g. vehicle, human).
10.  `attribute` - Property of an instance that can change while the category remains the same.
11.  `visibility` - Fraction of pixels visible in all the images collected from 6 different cameras.
12.  `sample_annotation` - An annotated instance of an object within our interest.
13.  `map` - Map data that is stored as binary semantic masks from a top-down view.

### 架构示意图解析

nuScenes数据集中的架构是基于token的机制，将上述13个概念连接起来，如下图所示：

![](https://www.nuscenes.org/public/images/nuscenes-schema.svg)

**首先**，我们可以从左到右看，采集数据的车在运行时会直接记录一些实时的信息，比如车辆编号，地点，日期，地图文件名，传感器之间的内外参等。

**接着**，我们可以对采集到的数据做解析，将所有数据分成一个又一个的scene，每个场景20s。场景之下是sample，表示的是某个特定时间戳在场景中的`关键帧`，它是有标注的（注意，在nuScenes中对每个scene，只以`2hz`的频率标注，也就是说，20Hz的lidar，10Hz只有一帧是有label的）。与sample对应的是sample data，其关联了底层的传感器数据源文件以及一些标定数据和ego的位姿信息。

**最后**，就是对数据的整理和标注，对于一个scene中不同帧之间相同的object，其表示为一个instance。此外还会有一些属性、类别和可见性的表达。注意，可见程度是根据目标在6个相机视野中的状态来衡量的。

### 数据集中类别定义

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220115155626.png)

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220115155643.png)

### Tutorial示例（超详细）

https://www.nuscenes.org/nuscenes?tutorial=nuscenes
