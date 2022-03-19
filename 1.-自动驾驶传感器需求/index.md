# 多传感器融合感知之自动驾驶传感器需求




<!--more-->


## 需求概述
自动驾驶的本质是解决出行问题，那么出行涉及到的问题包括
- 我在哪？要去哪？  *建图定位问题 SLAM*
- 如何去？ *规划控制问题*
- 路况如何？ *环境感知问题*

## 传感器概述
首先，要实现上述功能需要各种传感器，结合不同传感器的特性才可以使算法更加鲁棒。那么自动驾驶涉及到的传感器如下：
![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220317111726.png)
上图的传感器包括：
- 激光雷达 （*Light Detection And Ranging*）
- 相机  （*Camera*）
- 毫米波雷达  （*Radio Detection And Ranging*）
- 超声波雷达  （*ultrasonic Radar*）
- 全球卫星定位系统  （*Global Navigation Satellite System*以及 *Real-Time Kinematic*）
- 惯性传感器  （*Inertial Measurement Unit*）
- 轮速记  （*Wheel Speedometer*）

上述传感器可以分为两类：
- 运动感知类-侧重于感知自身，解决建图定位
	- GNSS IMU 轮速计 Lidar Camera
- 环境感知类-侧重于感知环境，解决环境感知
	- Lidar Camera Radar 超声波

## 需求具体分析
从需求侧分析，传感器需要帮助自动驾驶车辆解决各种各样的corner case，覆盖的场景包括
![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220317193202.png)

从供给侧分析，各种传感器由于其物体特性不同，各有优劣
- camera 颜色细节丰富，但缺乏尺度，且无主动光源
- lidar 三维距离准确，但成本高，量产难度大，且对雨雾敏感
- radar 可以感知速度，量产成熟，但高度和角度精度低，静止感知能力弱

所以，综上分析，我们才需要多传感器融合在一起。
