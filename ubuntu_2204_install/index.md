# ubuntu22_04升级记录


<!--more-->

![tp.web.random_picture](https://images.unsplash.com/photo-1497436072909-60f360e1d4b1?crop=entropy&cs=tinysrgb&fit=crop&fm=jpg&h=1080&ixid=MnwxfDB8MXxyYW5kb218MHx8bGFuZHNjYXBlLHdhdGVyfHx8fHx8MTY4MTAyMDIxNQ&ixlib=rb-4.0.3&q=80&utm_campaign=api-credit&utm_medium=referral&utm_source=unsplash_source&w=1920)

### 安装deep wine系列

参考 https://github.com/zq1997/deepin-wine

2204直接安装即可

```bash
wget -O- https://deepin-wine.i-m.dev/setup.sh | sh
sudo apt-get install com.qq.weixin.deepin
sudo apt-get install com.qq.office.deepin
```

```bash
sudo apt install snapd
sudo snap install slack
```



### system load monitor

```
sudo apt install python3-psutil gir1.2-appindicator3-0.1

git clone https://github.com/fossfreedom/indicator-sysmonitor.git
cd indicator-sysmonitor
sudo make install
cd ..
rm -rf indicator-sysmonitor
nohup indicator-sysmonitor &
```


### neovim && config

```
sudo apt install neovim




```


### ffmpeg && youtube-dl && lux
```bash
sudo apt install ffmpeg
```

### docker

```bash
  231  sudo apt install apt-transport-https curl gnupg-agent ca-certificates software-properties-common -y
  232  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
  233  sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"
  234  sudo apt install docker-ce docker-ce-cli containerd.io -y
  236  sudo usermod -aG docker jiayao
  237  newgrp docker
  238  docker version
  239  sudo systemctl status docker
  240  docker run hello-world
  241  docker login
  242  docker images

```


### 自动挂载sftp的目录

来自chatgpt～,使用时需要注意，目录名字不是18742038032，而是187XXXX8032
如果您想在Ubuntu启动时自动挂载sftp目录，可以通过使用SSHFS工具实现。 SSHFS是一个基于FUSE的工具，它可以通过SSH协议连接到远程服务器，并将远程文件系统挂载到本地文件系统上。

以下是实现自动挂载sftp目录的步骤：

1. 安装SSHFS

首先，您需要安装SSHFS。在终端中输入以下命令进行安装：

```
sudo apt-get update
sudo apt-get install sshfs
```

2. 创建本地目录

在本地文件系统上创建一个目录，用于挂载远程目录。您可以选择任何名称和位置。例如，我们将创建一个名为“remote_dir”的目录：

```
mkdir ~/remote_dir
```

3. 编辑/etc/fstab文件

打开/etc/fstab文件并在文件末尾添加以下行：

```
user@host:/remote/directory /path/to/local/mountpoint fuse.sshfs defaults,_netdev,user,idmap=user,IdentityFile=/path/to/private/key 0 0
```

将“user”和“host”替换为您的远程服务器用户名和主机名。将“/remote/directory”替换为您要挂载的远程目录的路径。将“/path/to/local/mountpoint”替换为本地目录的路径，也就是在第2步中创建的目录。将“/path/to/private/key”替换为您的私钥文件的路径。

4. 重新挂载文件系统

现在，您可以重新挂载文件系统以使更改生效。输入以下命令：

```
sudo mount -a
```

如果一切顺利，您应该可以在本地文件系统中看到挂载的远程目录了。

### 通过samba挂载极空间

首先安装 cmbclient

```bash
sudo apt-get install smbclient
```

然后可以通过如下命令查看

```bash
mbclient -L 192.168.1.9 -U your_username
```

通过如下命令挂载

```
sudo mount.cifs -o username=username,password="passwd" //192.168.1.9/18742038032 /efs
```

写入fstab

```bash
sudo vim /etc/fstab
//192.168.1.9/187XXXX8032 /efs cifs username=username,password=passwd. 0 0
```

### 安装cuda

To install CUDA on Ubuntu 22.04, follow these steps:

1.  Download the CUDA toolkit installer from the NVIDIA website. You can find the latest version at [https://developer.nvidia.com/cuda-downloads](https://developer.nvidia.com/cuda-downloads).
    
2.  Open a terminal window and navigate to the directory where the CUDA installer was downloaded.
    
3.  Make the installer executable by running the following command:
 
    `chmod +x cuda_<version>_linux.run`
    
    Replace `<version>` with the version number of the CUDA toolkit that you downloaded.
    
4.  Disable the Nouveau driver by creating a file in the `/etc/modprobe.d/` directory with the following content:

    `blacklist nouveau options nouveau modeset=0`
    
5.  Update the initramfs by running the following command:
    
    `sudo update-initramfs -u`
    
6.  Reboot your computer.
    
7.  Switch to a console terminal by pressing `Ctrl+Alt+F3`.
    
8.  Stop the X server by running the following command:
    
    `sudo service lightdm stop`
    
9.  Run the CUDA installer by running the following command:
    
    `sudo ./cuda_<version>_linux.run`
    
    Replace `<version>` with the version number of the CUDA toolkit that you downloaded.
    
10.  Follow the prompts in the installer to complete the installation.
    
11.  Reboot your computer.
    
12.  Verify that CUDA is installed correctly by running the following command:
    
`nvcc --version`

This should display the version number of the CUDA toolkit that you installed.
