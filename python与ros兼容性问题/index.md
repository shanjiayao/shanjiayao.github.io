# Python与ros兼容问题记录


----

<!-- more -->

## 1. python与ros兼容性问题

如果使用了conda环境或者是系统本身的python环境，但是在运行某一条命令时，显示在某一个路径下找不到固定的模块，类似

```javascript
  (python3) $ CUDA_VISIBLE_DEVICES=0 python -u train.py --work-path ./experiments/cifar10/lenet 
  Traceback (most recent call last):
    File "train.py", line 9, in <module>
      import yaml
    File "/opt/ros/kinetic/lib/python2.7/dist-packages/yaml/__init__.py", line 1, in <module>
      from error import *
  ModuleNotFoundError: No module named 'error'
```

使用pip list查找之后，发现对应的yaml包是有安装的

**这种问题的原因就是**：在sys的路径中，混入了ros的python路径，导致找不到对应的包，所以我们可以在对应的python文件最上面，加入以下代码：

```javascript
  import sys
  ros_path = '/opt/ros/kinetic/lib/python2.7/dist-packages'
  if ros_path in sys.path:
      sys.path.remove(ros_path)
```

## 2. python3、ros以及cv_bridge的兼容性问题

在使用python3以及ros中的cv包的时候，想要通过python3来创建一个节点接收话题形式的图像信息，然而在消息回调中如果想要通过cv_bridge来将消息格式进行转换，则会报错，形式如下：

```javascript
[ERROR] [1520780674.845066]: bad callback: <bound method ViewsBuffer.update of <__main__.ViewsBuffer object at 0x7f5f45a07f28>>
Traceback (most recent call last):
  File "/opt/ros/kinetic/lib/python2.7/dist-packages/rospy/topics.py", line 750, in _invoke_callback
    cb(msg)
  File "test.py", line 48, in update
    im = self.bridge.imgmsg_to_cv2(im, "bgr8")
  File "/opt/ros/kinetic/lib/python2.7/dist-packages/cv_bridge/core.py", line 163, in imgmsg_to_cv2
    dtype, n_channels = self.encoding_to_dtype_with_channels(img_msg.encoding)
  File "/opt/ros/kinetic/lib/python2.7/dist-packages/cv_bridge/core.py", line 99, in encoding_to_dtype_with_channels
    return self.cvtype2_to_dtype_with_channels(self.encoding_to_cvtype2(encoding))
  File "/opt/ros/kinetic/lib/python2.7/dist-packages/cv_bridge/core.py", line 91, in encoding_to_cvtype2
    from cv_bridge.boost.cv_bridge_boost import getCvType
ImportError: dynamic module does not define module export function (PyInit_cv_bridge_boost)
```

这里我尝试了stackflow中给出的解决方案，如下：

```javascript
# `python-catkin-tools` is needed for catkin tool
# `python3-dev` and `python3-catkin-pkg-modules` is needed to build cv_bridge
# `python3-numpy` and `python3-yaml` is cv_bridge dependencies
# `ros-kinetic-cv-bridge` is needed to install a lot of cv_bridge deps. Probaply you already have it installed.
sudo apt-get install python-catkin-tools python3-dev python3-catkin-pkg-modules python3-numpy python3-yaml ros-kinetic-cv-bridge
# Create catkin workspace
mkdir catkin_workspace
cd catkin_workspace
catkin init
# Instruct catkin to set cmake variables
catkin config -DPYTHON_EXECUTABLE=/usr/bin/python3 -DPYTHON_INCLUDE_DIR=/usr/include/python3.5m -DPYTHON_LIBRARY=/usr/lib/x86_64-linux-gnu/libpython3.5m.so

# Instruct catkin to install built packages into install place. It is $CATKIN_WORKSPACE/install folder
catkin config --install
catkin config --space-suffix _cb
# Clone cv_bridge src
git clone https://github.com/ros-perception/vision_opencv.git src/vision_opencv
# Find version of cv_bridge in your repository
apt-cache show ros-kinetic-cv-bridge | grep Version
    Version: 1.12.8-0xenial-20180416-143935-0800
# Checkout right version in git repo. In our case it is 1.12.8
cd src/vision_opencv/
git checkout 1.12.8
cd ../../
# Build
catkin build cv_bridge
# Extend environment with new package
source install/setup.bash --extend
```

还有一种方案,需要替换系统中的libboost_python到指定的环境下，因为我使用的是connda 环境，所以测试之后并不好用。

#### 最终方案

使用ubuntu16.04 以及系统环境中的python3

解决思路：下载cv_bridge的源码到对应的ros工作空间中,　然后使用catkin config进行单独编译，其余的package可以使用catkin_make进行编译，仅需在catkin config时配置新的编译输出文件夹即可。

解决详细步骤如下：

- 首先更改系统的~/.bashrc文件，取消conda的环境变量配置，并且设置python为python3
  
  ```javascript
  #__conda_setup="$('/home/echo/anaconda2/bin/conda' 'shell.bash' 'hook' 2> /dev/null)"
  #if [ $? -eq 0 ]; then
  #    eval "$__conda_setup"
  #else
  #    if [ -f "/home/echo/anaconda2/etc/profile.d/conda.sh" ]; then
  #        . "/home/echo/anaconda2/etc/profile.d/conda.sh"
  #    else
  #        export PATH="/home/echo/anaconda2/bin:$PATH"
  #    fi
  #fi
  #unset __conda_setup
  
  alias python=python3 #切换系统默认的python3.5　屏蔽以后默认为python2.7
  ```

- 安装对应的python3和ros功用的依赖，具体[参考](https://medium.com/@beta_b0t/how-to-setup-ros-with-python-3-44a69ca36674)

- 接下来catkin config 配置cv_bridge
  
  ```javascript
  # `python-catkin-tools` is needed for catkin tool
  # `python3-dev` and `python3-catkin-pkg-modules` is needed to build cv_bridge
  # `python3-numpy` and `python3-yaml` is cv_bridge dependencies
  # `ros-kinetic-cv-bridge` is needed to install a lot of cv_bridge deps. Probaply you already have it installed.
  sudo apt-get install python-catkin-tools python3-dev python3-catkin-pkg-modules python3-numpy python3-yaml ros-kinetic-cv-bridge
  # Create catkin workspace
  mkdir catkin_workspace
  cd catkin_workspace
  catkin init
  # Instruct catkin to set cmake variables
  catkin config -DPYTHON_EXECUTABLE=/usr/bin/python3 -DPYTHON_INCLUDE_DIR=/usr/include/python3.5m -DPYTHON_LIBRARY=/usr/lib/x86_64-linux-gnu/libpython3.5m.so
  
  # Instruct catkin to install built packages into install place. It is $CATKIN_WORKSPACE/install folder
  catkin config --install
  # 因为需要同时使用catkin_make 和 catkin build　所以需要指定catkin build 的路径
  catkin config -d devel_cb
  catkin config -b build_cb
  # Clone cv_bridge src
  git clone https://github.com/ros-perception/vision_opencv.git src/vision_opencv
  # Find version of cv_bridge in your repository
  apt-cache show ros-kinetic-cv-bridge | grep Version
      Version: 1.12.8-0xenial-20180416-143935-0800
  # Checkout right version in git repo. In our case it is 1.12.8
  cd src/vision_opencv/
  git checkout 1.12.8
  cd ../../
  # Build
  catkin build cv_bridge
  # Extend environment with new package
  source install/setup.bash --extend
  ```

- 安装结束之后，如果工作空间下还有其他的package需要编译，那么可以使用 catkin_make --only-pkg-with-deps 进行单包编译，完成之后正常执行 source devel/setup.bash 即可运行，这时的工作空间目录树结构如下（catkin config 输出的文件夹带有 _cb）
  
  ```
  .
  ├── build
  ├── build_cb
  ├── devel
  ├── devel_cb
  ├── install
  ├── logs
  └── src
  ```
