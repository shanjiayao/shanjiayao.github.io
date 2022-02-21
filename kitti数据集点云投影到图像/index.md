# KITTI数据集点云图像的投影


本文要完成的工作是将KITTI数据集中的点云信息投影到图像中，以达到信息融合的目的.　使用KITTI中的经过时钟同步和校准后的数据文件 raw_data 来进行投影变换。其主要操作就是坐标系之间的变换。

<!--more-->

### 1. KITTI数据集介绍

kitti的数据采集平台，配置有四个摄像机和一个激光雷达，四个摄像机中有两个灰度摄像机，两个彩色摄像机

从图中可看出，关于相机坐标系(camera)的方向与雷达坐标系(velodyne)的方向规定

![kitti_coordinate](https://gitee.com/yanglysysu/pic_assets/raw/master/kitti_coord.png)

camera: x = right, y = down, z = forward

velodyne: x = forward, y = left, z = up

那么velodyne所采集到的点云数据中，各点的x轴坐标，即为所需的深度信息。

更多详细的简介网络上都能搜索到，这里只列举了与当前目的相关的必要信息。

### 2. KITTI数据集中的raw_data

raw_data对于每个序列都提供了同步且校准后的数据、标定数据。

同步且校准后的数据：

- ./imageXX 包含有各个摄像机采集到的图像序列
  
  - 'image_00': left rectified grayscale image sequence
    - 'image_01': right rectified grayscale image sequence
    - 'image_02': left rectified color image sequence
    - 'image_03': right rectified color image sequence

- ./velodyne_points 包含有雷达扫描到的数据，点云形式，每个点以 (x,y,z,i) 格式存储，i为反射值
  
  - 雷达采集数据时，是绕着竖直轴旋转扫描，只有当雷达旋转到与相机的朝向一致时会触发相机采集图像。不过在这里无需关注这一点，直接使用给出的同步且校准后的数据即可，它已将雷达数据与相机数据对齐，也就是可以认为同一文件名对应的图像数据与雷达点云数据属于同一个场景。

标定数据：

- ./cam_to_cam 包含有各个摄像机的标定参数

- ./velo_to_cam 包含有雷达到摄像机的变换参数

对于raw_data，kitti还提供了样例工具，方便读取各种数据文件并输出，参见官网raw_data下载页的development kit

### 3. 利用kitti提供的devkit以及相应数据集的calib文件

##### 解读calib文件夹

- cam_to_cam，包含各相机的标定参数
  - S_xx: 1x2 矫正前xx号相机的图片尺寸
  - K_xx: 3x3 矫正前xx号相机的标定参数
  - D_xx: 1x5 矫正前xx号相机的畸变系数
  - R_xx: 3x3 外参，xx号相机的旋转矩阵
  - T_xx: 3x1 外参，xx号相机的平移矩阵
  - S_rect_xx: 1x2 矫正后XX号相机的图片尺寸
  - R_rect_xx: 3x3 旋转矩阵，用于矫正xx号相机，使得图像平面共面(原话是make image planes co-planar)。
  - P_rect_0x: 3x4 投影矩阵，用于从矫正后的0号相机坐标系 投影到 X号相机的图像平面。

这里只用到最后两个矩阵R_rect和P_rect

- velo_to_cam，从雷达坐标系到0号相机坐标系的转换
  - R: 3x3 旋转矩阵
  - T:  3x1 平移矩阵
  - delta_f 和delta_c 已被弃用

由此可以得出从雷达坐标系变换到xx号相机的图像坐标系的公式：

设X为雷达坐标系中的齐次坐标 X = [x y z 1]'，对应于xx号相机的图像坐标系的齐次坐标Y = [u v 1]'，则：

**Y = P_rect_xx * R_rect_00 * (R|T)_velo_to_cam * X**

- (R|T) ： 雷达坐标系 -> 0号相机坐标系
- R_rect_00： 0号相机坐标系 -> 矫正后的0号相机坐标系
- P_rect_0x： 矫正后的0号相机坐标系 -> 0号相机的图像平面

##### 解读devkit

官网提供的样例代码中 run_demoVelodyne.m 实现了将雷达点云投影到相机图像

大致说一下步骤：

从所给路径中读取标定文件，获取具体矩阵数值

- 根据上述公式，计算投影矩阵 P_velo_to_img，即 Y = P_velo_to_img * X
- 从所给路径中读取相机图片，并加载雷达的点云数据。由于只做展示用，为了加快运行速度，对于雷达点云，每隔5个点只保留1个点
- 移除那些距离雷达5米之内(雷达的x方向)的点 (猜测这些点落在相机和雷达之间，故不会出现在图像平面上)
- 作投影计算，得到投影到二维图像上的点
- 在图像上画出投影后的点，按照深度(雷达点的x方向值)确定颜色，彩色则是暖色越近，冷色越远；灰度则是深色越近，浅色越远。

