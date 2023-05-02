# docker安装记录


------

记录安装docker的过程。

<!-- more -->

nvidia-docker2已经弃用，现在都是装nvidia-container-toolkit

1. 清除旧版本
   
   ```bash
   sudo apt-get remove docker docker-engine docker-ce docker.io
   sudo rm -rf /var/lib/docker
   dpkg -l | grep docker
   sudo apt-get purge docker-ce
   ```

2. 更新apt-get
   
   ```bash
   sudo apt-get update
   ```

3. 安装添加使用 HTTPS 传输的软件包
   
   ```bash
   sudo apt-get -y install apt-transport-https ca-certificates curl software-properties-common
   ```

4. 添加软件源的GPG密钥---清华源
   
   ```bash
   curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
   sudo add-apt-repository "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
   ```

5. 更新apt-get，安装docker
   
   ```bash
   sudo apt-get update
   sudo apt-get install docker-ce
   ```

6. 查看docker版本
   
   ```bash
   docker version
   ```

7. 这里docker安装初步完成，可以使用`docker hello-world`测试，如果出现权限问题报错，可以尝试如下方法：
   
   ```bash
   # 添加一个docker属组（如果没有）
   sudo groupadd docker
   
   # 将用户加入该group中，退出并重新登陆
   sudo gpasswd -a ${USER} docker
   
   # 重启docker服务
   sudo service docker restart
   
   # 切换当前会话到新group或重启会话
   newgrp - docker
   ```

8. 接下来安装深度学习相关的环境，首先安装docker-nvidia
   
   参考[官网git](https://github.com/NVIDIA/nvidia-docker)
   
   ```bash
   # Add the package repositories
   distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
   curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
   curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list
   
   sudo apt-get update && sudo apt-get install -y nvidia-container-toolkit
   sudo systemctl restart docker
   ```

9. 安装nvidia-container-runtime
   
   参考[官网git](https://github.com/nvidia/nvidia-container-runtime#environment-variables-oci-spec)
   
   ```bash
   sudo apt-get install nvidia-container-runtime
   sudo mkdir -p /etc/systemd/system/docker.service.d
   sudo tee /etc/systemd/system/docker.service.d/override.conf <<EOF
   [Service]
   ExecStart=
   ExecStart=/usr/bin/dockerd --host=fd:// --add-runtime=nvidia=/usr/bin/nvidia-container-runtime
   EOF
   sudo systemctl daemon-reload
   sudo systemctl restart docker
   
   sudo tee /etc/docker/daemon.json <<EOF
   {
       "runtimes": {
           "nvidia": {
               "path": "/usr/bin/nvidia-container-runtime",
               "runtimeArgs": []
           }
       }
   }
   EOF
   
   sudo pkill -SIGHUP dockerd
   
   sudo dockerd --add-runtime=nvidia=/usr/bin/nvidia-container-runtime [...]
   ```


10. 修改镜像和容器的存放路径
    
    1. 指定镜像和容器存放路径的参数是–graph=/var/lib/docker，我们只需要修改配置文件指定启动参数即可。Docker 的配置文件可以设置大部分的后台进程参数，在各个操作系统中的存放位置不一致，在 Ubuntu 中的位置是：`/etc/default/docker`，在 CentOS 中的位置是：`/etc/sysconfig/docker`
    
    2. 打开`/etc/default/docker`
       
       ```bash
       sudo gedit /etc/default/docker
       
       # 如果是 CentOS 则添加下面这行：
       OPTIONS=--graph="/root/data/docker" --selinux-enabled -H fd://
       
       # 如果是 Ubuntu 则添加下面这行（因为 Ubuntu 默认没开启 selinux）：
       OPTIONS=--graph="/root/data/docker" -H fd://
       # 或者
       DOCKER_OPTS="-g /root/data/docker"
       ```
    
    3. 最后重新启动，`service docker restart`
       
       通过`docker info` 查看
       
       Docker 的路径是否改成 `/root/data/docker`
    
    4. 如果没有生效，按如下操作
       
       ```bash
       mkdir -p /etc/systemd/system/docker.service.d  
       
       cat /etc/systemd/system/docker.service.d/Using_Environment_File.conf  
       如果没有该文件则自行创建，添加以下内容
       [Service]  
       EnvironmentFile=-/etc/default/docker  
       ExecStart=  
       ExecStart=/usr/bin/docker daemon -H fd:// $DOCKER_OPTS
       
       载入配置重启服务
       systemctl daemon-reload  
       service docker restart  
       ```

11. 修改为阿里的镜像仓库
    
    ```bash
    # 这里我在注册阿里云之后，将镜像仓库更改为阿里的云仓库，使用如下命令
    
sudo tee /etc/docker/daemon.json <<-'EOF'

> {
> "registry-mirrors": ["https://b68wbkqs.mirror.aliyuncs.com"]
> }
> EOF                                                                  
```

