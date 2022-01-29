# ubuntu18_04升级记录


------

<!-- more -->


## 1. 系统安装

1.  关闭secure boot
    
2.  插上系统盘，开机后狂按ESC，暂停启动，然后弹出BIOS设置，选择启动设备，使用U盘启动
    
3.  在选择‘Install UBUNTU’之前，按E进入设置，在quiet splash ---那处,把---替换为nomodeset,再按F10进入系统进行安装，否则会报错acpi bios error以及花屏！！！
    
4.  然后正常选择，我选的最小系统安装，免去安装多余的游戏软件等
    
5.  分区设置，先选择something else，自己设置分区，如下
    
    -   设置swap空间:主分区/空间起始/交换空间/大小一般是实际内存1-2倍
        
    -   设置引导EFI:逻辑分区/空间起始/**EFI分区**/大小一般1GB
        
    -   设置挂载的/:这个是ubuntu计算机的安装位置,逻辑分区/空间起始/EXT4/挂载点为/
        
    -   最后设置引导,最下面的那个选项,选择之前设置的EFI引导在的那个分区
        
    
    注意，这里建议不单独分出来/home的空间，直接全都放在/下面就好，免得空间不够难以重新分区
    
6.  进入安装,完成后会重启,重启后同样按e进入,在quiet splash后加nomodeset再按F10进入
    

## 2. 换源，更新，升级

进入系统后第一步操作应该就是将源改成国内的清华或者阿里，然后运行以下命令

sudo apt update  
sudo apt upgrade

## 3. Nvidia驱动安装

上述操作装完系统后，没有显卡驱动是无法进入的，所以必须加上nomodeset

除此之外，进系统后的分辨率应该也不对，会更大一些，这一切的原因都是显卡没有驱动程序，以及没有集显导致的

装一个独显驱动即可，方法：

-   进入`Software & Updates`
    
-   在第一个选项卡，`ubuntu software`处，勾选`source code`
    
-   在第五个选项卡，`additional drivers`处，选择nvidia驱动
    
    using nvidia driver metapackage from nvidia-driver-450（proprietary）
    
-   点击右下角的`apply changes`，重启即可
    

## 4. 安装常用软件

sudo apt install python python3 gconf-service-backend gconf-service gconf2

## 5. 安装小飞机

先确保安装python以及python3

建议使用appimage，方便开机自启动

#! /bin/bash  
cd ~/Softwares  
./electron-ssr-0.2.6.AppImage

-   自启动
    
    将以上写成脚本，在`start application`里面添加启动程序，直接查找脚本位置即可，不需要 . 或者 sh
    

注意权限

## 6. 挂载其它ntfs磁盘，实现开机自动挂载

# 查看磁盘  
sudo fdisk -l  
# 安装包  
sudo apt install ntfs-3g  
sudo apt install ntfs-config  
# 查看uuid  
sudo blkid  
​  
# 创建要挂载的文件夹  
mkdir /home/disks  
sudo mkdir /media/echo/Docs  
sudo mkdir /media/echo/Storage  
​  
# 修改fstab  
sudo gedit /etc/fstab  
 # 添加以下内容  
 #Entry for /dev/sda5 :  
 UUID=10AA0A8510AA0A85    /media/echo/Docs    ntfs    defaults,nodev,nosuid,uid=1000,gid=1000,uhelper=udisks2    0    0  
 #Entry for /dev/sda1 :  
 UUID=10A8073A10A8073A    /media/echo/Storage    ntfs    defaults,nodev,nosuid,uid=1000,gid=1000,uhelper=udisks2    0    0  
 #Entry for /dev/nvme0n1p4 :  
 UUID=0BE104A20BE104A2    /home/echo/disks    ntfs    defaults    0    0  
​  
​  
# 测试挂载，不报错即为没问题  
sudo umount -a  
sudo mount -a

## 7. 安装ipgw

找到对应ipgw安装包

sudo chmod +x ipgw  
sudo mv ipgw /usr/local/bin  
ipgw version  
ipgw login -u 1901938 -p xxx -s

## 8. 安装chrome

sudo dpkg -i google-chrome-stable_current_amd64.deb  
sudo apt upgrade

## 9. 安装搜狗输入法

先装fcitx

sudo apt install fcitx    # 如果报错，使用 sudo apt-get install -f 再执行  
sudo dpkg -i sogoupinyin_2.3.1.0112_amd64.deb  
reboot

打开设置，找到 `language`-> `Manage Installed Languages`, 將“Keyboard input method system”設定為“fcitx”

在系統中搜索fcitx configuration，點選左下角新增輸入法，在彈出的對話方塊中將Only Show Current Language取消，即可看到sogou Pinyin，選擇新增即可

## 10. 安装terminator

sudo apt install terminator
# 刷新配置
cp terminator_config ~/.config/terminator/config

## 11. 安装typora

# or run:
# sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys BA300B7755AFCFAE
wget -qO - https://typora.io/linux/public-key.asc | sudo apt-key add -
# add Typora's repository
sudo add-apt-repository 'deb https://typora.io/linux ./'
sudo apt-get update
# install typora
sudo apt-get install typora

## 12. 安装CUDA\CUDNN

### cuda10.2

1.  cuda下载列表
    
    [https://developer.nvidia.com/cuda-toolkit-archive](https://developer.nvidia.com/cuda-toolkit-archive)
    
2.  选择18.04 cuda10.2 使用deb安装
    
    wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/cuda-ubuntu1804.pin
    sudo mv cuda-ubuntu1804.pin /etc/apt/preferences.d/cuda-repository-pin-600
    wget https://developer.download.nvidia.com/compute/cuda/10.2/Prod/local_installers/cuda-repo-ubuntu1804-10-2-local-10.2.89-440.33.01_1.0-1_amd64.deb
    sudo dpkg -i cuda-repo-ubuntu1804-10-2-local-10.2.89-440.33.01_1.0-1_amd64.deb
    sudo apt-key add /var/cuda-repo-10-2-local-10.2.89-440.33.01/7fa2af80.pub
    sudo apt-get update
    sudo apt-get -y install cuda
    
3.  在bashrc或者zshrc中加入
    
    # cuda add
    export CPATH=/usr/local/cuda/targets/x86_64-linux/include:$CPATH
    export LD_LIBRARY_PATH=/usr/local/cuda/targets/x86_64-linux/lib:$LD_LIBRARY_PATH
    export PATH=/usr/local/cuda/bin:$PATH
    export CUDA_HOME=/usr/local/cuda
    
    使用`nvcc -V`测试
    
    nvcc: NVIDIA (R) Cuda compiler driver
    Copyright (c) 2005-2019 NVIDIA Corporation
    Built on Wed_Oct_23_19:24:38_PDT_2019
    Cuda compilation tools, release 10.2, V10.2.89
    
4.  安装patch
    
    [https://developer.nvidia.com/cuda-10.2-download-archive?target_os=Linux&target_arch=x86_64&target_distro=Ubuntu&target_version=1804&target_type=deblocal](https://developer.nvidia.com/cuda-10.2-download-archive?target_os=Linux&target_arch=x86_64&target_distro=Ubuntu&target_version=1804&target_type=deblocal)
    
    # patch1
    sudo dpkg -i cuda-repo-ubuntu1804-10-2-local_10.2.1-1_amd64.deb 
    sudo apt update
    sudo apt -y upgrade
    # patch2
    sudo dpkg -i cuda-repo-ubuntu1804-10-2-local_10.2.2-1_amd64.deb 
    sudo apt update
    sudo apt -y upgrade
    

### cudnn

需要登录，使用google账号登录， 下载 **cudnn-10.2-linux-x64-v8.1.1.33**

-   source安装，先解压，出现一个cuda的文件夹
    
    tar -zxf cudnn-10.2-linux-x64-v8.1.1.33.tgz
    sudo cp cuda/include/cudnn.h /usr/local/cuda/include
    sudo cp cuda/lib64/libcudnn* /usr/local/cuda/lib64
    sudo chmod a+r /usr/local/cuda/include/cudnn.h 
    sudo chmod a+r /usr/local/cuda/lib64/libcudnn*
    
-   deb安装-未尝试
    
    要下载的文件也是两个，分别是`cuDNN Runtime Library for Ubuntu18.04 (Deb`和 `[cuDNN Developer Library for Ubuntu18.04 (Deb)]`
    
    下载完成后一次安装这两个文件就可以了**（先安装runtime library，再安装developer library）**
    
    至此，CUDA10.2+cuDNN8就成功的在Ubuntu18.04上安装成功了。
    
-   更换安装方法！！！！
    
    由于源码安装的cudnn无法被找到，所以换deb安装
    
    sudo rm /usr/local/cuda/include/cudnn.h
    sudo rm /usr/local/cuda/lib64/libcudnn*
    sudo dpkg -i libcudnn8_8.1.0.77-1+cuda10.2_amd64.deb
    sudo dpkg -i libcudnn8-dev_8.1.0.77-1+cuda10.2_amd64.deb
    sudo dpkg -i libcudnn8-samples_8.1.0.77-1+cuda10.2_amd64.deb
    
-   由于8.1.0无法通过cudnn测试，所以换成7.6.5
    
-   又安装了源码tar格式的cudnn
    

关于查看GPU显卡的状态，除了使用显卡驱动中提供的命令 nvidia-smi 外，推荐使用

[https://github.com/Syllo/nvtopgithub.com](https://link.zhihu.com/?target=https%3A//github.com/Syllo/nvtop)

## 13. 安装pycharm

由于ubuntu自带了snap，使用snap安装

# sudo snap install [pycharm-professional|pycharm-community] --classic
sudo snap install pycharm-professional --classic

## 14. 安装openvpn

## 补充在ubuntu18.04安装的记录

sudo apt update
sudo apt install openvpn

接下来和ubuntu16.04一致，运行脚本以及ovpn文件即可

## 15. 安装anaconda

wget https://repo.anaconda.com/archive/Anaconda3-2020.11-Linux-x86_64.sh

给权限, 运行

sudo chmod 777 Anaconda3-2020.07-Linux-x86_64.sh
bash Anaconda3-2020.07-Linux-x86_64.sh

安装过程中, 在安装目录处修改目录, 第二个问题选yes

## 16. 安装git

sudo apt install -y git

自动保存账户密码

git config --global -e

-> 添加以下行
[credential]
    helper = store

## 17. 安装zsh

确保安装了git

[https://ohmyz.sh/#install](https://ohmyz.sh/#install)

[https://github.com/romkatv/powerlevel10k](https://github.com/romkatv/powerlevel10k)

sudo apt-get install zsh
sh -c "$(wget https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh -O -)"
git clone --depth=1 https://gitee.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions

## 18. 安装opencv

## 19. 安装ros

> sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'

> sudo apt-key adv --keyserver 'hkp://keyserver.ubuntu.com:80' --recv-key C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654

Executing: /tmp/apt-key-gpghome.GnF5MH9S4s/gpg.1.sh --keyserver hkp://keyserver.ubuntu.com:80 --recv-key C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654
gpg: key F42ED6FBAB17C654: public key "Open Robotics <info@osrfoundation.org>" imported
gpg: Total number processed: 1
gpg:               imported: 1

报错

/sbin/ldconfig.real: /usr/local/cuda-10.2/targets/x86_64-linux/lib/libcudnn_cnn_infer.so.8 is not a symbolic link

/sbin/ldconfig.real: /usr/local/cuda-10.2/targets/x86_64-linux/lib/libcudnn_adv_train.so.8 is not a symbolic link

/sbin/ldconfig.real: /usr/local/cuda-10.2/targets/x86_64-linux/lib/libcudnn_adv_infer.so.8 is not a symbolic link

/sbin/ldconfig.real: /usr/local/cuda-10.2/targets/x86_64-linux/lib/libcudnn_ops_train.so.8 is not a symbolic link

/sbin/ldconfig.real: /usr/local/cuda-10.2/targets/x86_64-linux/lib/libcudnn_cnn_train.so.8 is not a symbolic link

/sbin/ldconfig.real: /usr/local/cuda-10.2/targets/x86_64-linux/lib/libcudnn.so.8 is not a symbolic link

/sbin/ldconfig.real: /usr/local/cuda-10.2/targets/x86_64-linux/lib/libcudnn_ops_infer.so.8 is not a symbolic link

sudo ln -sf libcudnn_adv_infer.so.8.1.1 libcudnn_adv_infer.so.8
sudo ln -sf libcudnn_adv_infer.so.8 libcudnn_adv_infer.so
sudo ln -sf libcudnn_adv_train.so.8.1.1 libcudnn_adv_train.so.8
sudo ln -sf libcudnn_adv_train.so.8 libcudnn_adv_train.so
sudo ln -sf libcudnn_cnn_train.so.8.1.1 libcudnn_cnn_train.so.8
sudo ln -sf libcudnn_cnn_train.so.8 libcudnn_cnn_train.so
sudo ln -sf libcudnn_ops_infer.so.8.1.1 libcudnn_ops_infer.so.8
sudo ln -sf libcudnn_ops_train.so.8.1.1 libcudnn_ops_train.so.8
sudo ln -sf libcudnn_ops_train.so.8 libcudnn_ops_train.so
sudo ln -sf libcudnn.so.8.1.1 libcudnn.so.8
sudo ln -sf libcudnn.so.8 libcudnn.so

执行

sudo rosdep init

报错

sudo: rosdep: command not found

执行

sudo apt install rospack-tools

开代理

sudo rosdep init
rosdep update

写入bashrc以及zshrc

alias ros='source /opt/ros/melodic/setup.zsh'

## 20. 安装hexo&Gitbook&slidev

### 1. 安装nodejs v10.x

[https://github.com/nodesource/distributions](https://github.com/nodesource/distributions)

curl -fsSL https://deb.nodesource.com/setup_10.x | sudo -E bash -
sudo apt-get install -y nodejs

### 2. 安装Git[略]

### 3. 安装Hexo

npm install hexo
PATH="$PATH:/home/echo/disks/HexoBlogs/node_modules/.bin"

### 4. 安装Gitbook

sudo npm install -g gitbook-cli
gitbook init 

将node升级到了14，但是发现hexo不好用，无法hexo d，所以降级到12，但是gitbook还不好使，于是修改gitbook

查看和更改node版本

node -v
sudo n

查看hexo版本

hexo -v

---

### 6. 重装hexo以及部署新的博客主题，还有gitbook[2021-6-10]

1.  解决npm全局安装的问题
    
    在 /home 下
    
    mkdir ~/.npm-global
    npm config set prefix '~/.npm-global'   # 设置npm全局包的安装路径:
    export PATH=~/.npm-global/bin:$PATH   # 在用户的根目录下查看有没有.profile文件, 如果没有就创建
    source ~/.profile
    
2.  安装hexo
    
    npm install -g hexo-cli
    
    | 注：还有第二种安装方式
    
    在blog目录下安装
    
    npm install hexo
    
    然后在 `~/.profile`中添加路径，就可以在终端启动hexo命令了
    
    PATH="$PATH:/home/echo/disks/blog/node_modules/.bin"
    
3.  接下来就可以初始化并创建博客了，新建一个博客文件夹（原来的HexoBlogs被我弄的坏了，运行命令报错）
    
    hexo init blog #初始化博客站点
    cd blog
    npm install #安装依赖
    
4.  把原来的博客文件夹 HexoBlogs路径下的 source 拷贝到新建的 blog对应的source下面
    
5.  运行命令
    
    hexo g & hexo s
    
    这时没改config里面的主题，还应该是可以显示原来博客内容的，只是主题不对
    
6.  更改主题为butterfly
    
    在blog目录下，开终端，先下载，然后安装渲染器
    
    git clone -b master https://github.com/jerryc127/hexo-theme-butterfly.git themes/butterfly
    npm install hexo-renderer-pug hexo-renderer-stylus --save
    

npm install hexo-deployer-git --save 
npm install --save hexo-word-counter

## 21. 安装deepin相关

deepin wine基础框架，星外之神那个不好用，下载下面的安装

[https://github.com/chenyingzhou/deepin-wine-ubuntu](https://github.com/chenyingzhou/deepin-wine-ubuntu)

deepin wine相关软件

[http://packages.deepin.com/deepin/pool/non-free/d/](http://packages.deepin.com/deepin/pool/non-free/d/)

下载里面的deepin.com.wechat/ 以及 deepin.com.qq.office/ 这两个是好用的

dpkg安装

sudo apt install chrome-gnome-shell
sudo apt install gnome-shell-extensions

除此之外，还需要安装topicons plus插件，然后使用tweak打开插件，方便图标显示

### 微信

#### 解决中文乱码问题

装完后中文乱码，按照下列教程

sudo gedit /opt/deepinwine/tools/run.sh
sudo gedit /opt/deepinwine/tools/run_v2.sh

找到WINE_CMD 修改为：

WINE_CMD="LC_ALL=zh_CN.UTF-8 deepin-wine"

然后下载字体，更换

修改字体 下载字体msyh.ttc, 下载地址一：蓝奏云 （推荐）[https://www.lanzous.com/i5wivmd](https://www.lanzous.com/i5wivmd) 下载地址二：百度网盘 链接: [https://pan.baidu.com/s/1rkjkmGJlpdaijCEWi7TZIw](https://pan.baidu.com/s/1rkjkmGJlpdaijCEWi7TZIw) 提取码: btxw

将下载的字体解压，然后：

cp msyh.ttc ~/.deepinwine/Deepin-WeChat/drive_c/windows/Fonts

修改系统注册表

gedit ~/.deepinwine/Deepin-WeChat/system.reg

更改以下两行内容为：

"MS Shell Dlg"="msyh"
"MS Shell Dlg 2"="msyh"

gedit msyh_config.reg

在文件msyh_config.reg内添加如下内容：

REGEDIT4
[HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion\FontLink\SystemLink]
"Lucida Sans Unicode"="msyh.ttc"
"Microsoft Sans Serif"="msyh.ttc"
"MS Sans Serif"="msyh.ttc"
"Tahoma"="msyh.ttc"
"Tahoma Bold"="msyhbd.ttc"
"msyh"="msyh.ttc"
"Arial"="msyh.ttc"
"Arial Black"="msyh.ttc"

注册字体

deepin-wine regedit msyh_config.reg

#### 解决中心黑框问题

[https://blog.diqigan.cn/posts/wine-wechat-black-square-fix.html](https://blog.diqigan.cn/posts/wine-wechat-black-square-fix.html)

按照操作，做一遍，调试了一下sleep的值，改为0的话就成功了

## 22. 定制Mac主题

未尝试

1.  安装gnome-tweak-tool 和 chrome-gnome-shell 插件 (`sudo aptitude install [name]`)
    
2.  安装GTK3主题 => [X-Arc-Collection](https://www.gnome-look.org/p/1167049/)
    
3.  使用tweak载入应用程序主题 => tweak — 外观 — 应用程序 — 选择X-Arc-Collection
    
4.  安装gnome-shell 主题 => [macOS High Sierra](https://www.gnome-look.org/p/1167049/)
    
5.  安装gnome-shell 插件 => [User Themes](https://extensions.gnome.org/extension/19/user-themes/) ( 之后重启Gnome => [Alt + F2] & [输入 r] & [点击 Enter] )
    
6.  使用tweak载入shell主题 => tweak — 外观 — shell — 选择Sierra shell主题
    
7.  下载Mac图标主题 [la-capitaine-icon-theme](https://github.com/keeferrourke/la-capitaine-icon-theme/releases) 或 [McMojave-circle](https://www.pling.com/p/1305429/)
    
8.  图标文件夹移动到 ~/.icons目录下(没有则新建目录)
    
9.  使用tweak载入icon主题 => tweak — 外观 — 图标 — 选择对应的图标主题
    
10.  安装gnome-shell插件 => Dash to dock (将原生dock转变为可定制的浮动dock)
    
11.  定制firefox主题 => [Majave-gtk-theme](https://github.com/vinceliuice/Mojave-gtk-theme)
    

## 23. 安装handbrake

sudo add-apt-repository ppa:stebbins/handbrake-releases
sudo apt update
sudo apt-get install handbrake-gtk

## 24. 安装ffmpeg

[https://octopuslian.github.io/2019/11/13/ubuntu1804-ffmpeg-install/](https://octopuslian.github.io/2019/11/13/ubuntu1804-ffmpeg-install/)

安装在/home/echo/softwares下

可执行文件在ffmerg_install/bin下，将两个文件复制到/usr/local/bin下面

## 25.安装视频播放软件

vlc

## 26. 安装notion

[https://appmaker.xyz/web2desk/](https://appmaker.xyz/web2desk/)将notion桌面化

解压后，找到notion，./执行即可

-   因难于使用，放弃
    
    使用snap安装软件版notion
    
    sudo snap install notion-snap
    sudo apt install fonts-wqy-zenhei    # 解决部分字体变为楷体的问题
    

## 27. 其他appimage

将其放在/home/echo/softwares下，使用terminator右键执行

## 28. 安装mendeley

下载软件deb，安装

由于使用了sudo打开，导致正常权限无法打开软件

sudo chown -R $(whoami) ~/.local/share/data/Mendeley Ltd.

运行，注意空格

sudo chown -R echo ~/.local/share/data/Mendeley\ Ltd.

解决！

此外，由于1.19.8版本没有文献搜索，所以最好使用1.19.6

[https://www.mendeley.com/autoupdates/installers/1.19.6](https://www.mendeley.com/autoupdates/installers/1.19.6)

## 29. 安装flameshot

sudo apt install flameshot

更改默认截图键为flameshot

#### On Ubuntu (Tested on 18.04)

To use Flameshot instead of the default screenshot application in Ubuntu we need to remove the binding on Prt Sc key, and then create a new binding for `/usr/bin/flameshot gui` ([adaptated](https://askubuntu.com/posts/1039949/revisions) from [Pavel's answer on AskUbuntu](https://askubuntu.com/revisions/1036473/1)).

1.  Remove the binding on Prt Sc using the following command.
    

gsettings set org.gnome.settings-daemon.plugins.media-keys screenshot '[]'

1.  Go to Settings > Device > Keyboard and press the '+' button at the bottom.
    
2.  Name the command as you like it, e.g. `flameshot`. And in the command insert `/usr/bin/flameshot gui`.
    
3.  Then click "_Set Shortcut.._" and press Prt Sc. This will show as "_print_".
    

Now every time you press Prt Sc, it will start the Flameshot GUI instead of the default application.

## 30. 安装filezilla

sudo apt-get install filezilla

## 31. 安装youtube-dl && annie

### youtube-dl

sudo curl -L https://yt-dl.org/downloads/latest/youtube-dl -o /usr/local/bin/youtube-dl
sudo chmod a+rx /usr/local/bin/youtube-dl

### annie

wget https://github.com/iawia002/annie/releases/download/0.10.3/annie_0.10.3_Linux_64-bit.tar.gz
tar zxvf annie_*.tar.gz
mv annie /usr/local/bin/

## 32. 安装Tex相关

sudo apt update
sudo apt install texlive-full
sudo apt install texstudio

## 33.右键打开terminator

[https://blog.csdn.net/bestBT/article/details/81221378](https://blog.csdn.net/bestBT/article/details/81221378)

## 34.更新cmake

不知什么原因，cmake版本是3.10，无法安装pcdet里需要的spconv

故升级到3.19.6

./bootstrap --prefix=/usr
 make -j`proc`
 sudo make install

使用`cmake --version`查看版本

## 35. 安装tensorrt

在官网下载对应的版本，TensorRT-7.0.0.11.Ubuntu-18.04.x86_64-gnu.cuda-10.2.cudnn7.6

下载tar版本以及deb版本

### 安装tar到python

tar版本的解压到 /home/echo/misc/extra_libs/

同时在zshrc中加入

export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/home/echo/misc/extra_libs/TensorRT-7.0.0.11.Ubuntu-18.04.x86_64-gnu.cuda-10.2.cudnn7.6/TensorRT-7.0.0.11/lib

进入到python环境，然后到TensorRT-7.0.0.11目录下，打开终端执行：

cd python
pip install tensorrt-7.0.0.11-cp37-none-linux_x86_64.whl
cd ..
cd uff
pip install uff-0.6.5-py2.py3-none-any.whl
cd .. 
cd graphsurgeon
pip install graphsurgeon-0.4.1-py2.py3-none-any.whl

测试下

python
import tensorrt    # 无反应即为成功

### 安装deb版本到c++

$ sudo dpkg -i nv-tensorrt-repo-ubuntu1804-cuda10.2-trt7.0.0.11-ga-20191216_1-1_amd64.deb
$ sudo apt-key add /var/nv-tensorrt-repo-cuda10.2-trt7.0.0.11-ga-20191216/7fa2af80.pub
$ sudo apt-get update
$ sudo apt-get install tensorrt

$ dpkg -l | grep TensorRT
----------------------------------
ii  libnvinfer-bin                                               7.0.0-1+cuda10.2                                 amd64        TensorRT binaries
ii  libnvinfer-dev                                               7.0.0-1+cuda10.2                                 amd64        TensorRT development libraries and headers
ii  libnvinfer-doc                                               7.0.0-1+cuda10.2                                 all          TensorRT documentation
ii  libnvinfer-plugin-dev                                        7.0.0-1+cuda10.2                                 amd64        TensorRT plugin libraries
ii  libnvinfer-plugin7                                           7.0.0-1+cuda10.2                                 amd64        TensorRT plugin libraries
ii  libnvinfer-samples                                           7.0.0-1+cuda10.2                                 all          TensorRT samples
ii  libnvinfer7                                                  7.0.0-1+cuda10.2                                 amd64        TensorRT runtime libraries
ii  libnvonnxparsers-dev                                         7.0.0-1+cuda10.2                                 amd64        TensorRT ONNX libraries
ii  libnvonnxparsers7                                            7.0.0-1+cuda10.2                                 amd64        TensorRT ONNX libraries
ii  libnvparsers-dev                                             7.0.0-1+cuda10.2                                 amd64        TensorRT parsers libraries
ii  libnvparsers7                                                7.0.0-1+cuda10.2                                 amd64        TensorRT parsers libraries
ii  tensorrt                                                     7.0.0.11-1+cuda10.2                              amd64        Meta package of TensorRT

说明安装成功

## 36. 安装苹果主题

[https://zhuanlan.zhihu.com/p/71588449](https://zhuanlan.zhihu.com/p/71588449)

按照以上链接执行即可

sudo apt install gnome-tweak-tool gnome-shell-extensions chrome-gnome-shell

sudo apt-get install gtk2-engines-murrine gtk2-engines-pixbuf
sudo add-apt-repository ppa:dyatlov-igor/sierra-theme
sudo apt update
sudo apt install sierra-gtk-theme

git clone https://github.com/USBA/Cupertino-iCons
sudo unzip -o Cupertino-iCons.zip -d /usr/share/icons

安装gnome插件的方法如下

[https://zhuanlan.zhihu.com/p/36265103](https://zhuanlan.zhihu.com/p/36265103)

## 37. 安装惠普打印机驱动

sudo apt-get install hplip hplip-gui

然后找到hplip toolsbox，按照提示安装即可

## 其他小工具

# 小火车
sudo apt install sl # 使用sl调用
# 代码雨
sudo apt install cmatrix
# CPU内存等显示插件 https://www.sohu.com/a/301992941_495675
sudo apt-get install gir1.2-gtop-2.0 gir1.2-networkmanager-1.0 gir1.2-clutter-1.0
打开Ubuntu软件，然后搜索“system monitor extension” 选择system monitor 可以显示network的那一个

## 更改锁屏输入密码时的壁纸

sudo cp /usr/share/gnome-shell/theme/ubuntu.css /usr/share/gnome-shell/theme/ubuntu.css.bak
sudo gedit /usr/share/gnome-shell/theme/ubuntu.css

修改配置文件，记得修改路径

#lockDialogGroup {
  background: #2c001e url(resource:///org/gnome/shell/theme/noise-texture.png);
  background-repeat: repeat; }

---------------- 改为 -----------------------

#lockDialogGroup {
  background: #2c001e url(file:///home/echo/.lock.jpg);
  height: 100%;
  background-size: contain;
  background-attachment: fixed;
  background-position: 0px 0px;
  background-repeat: repeat; 
}
这个版本可以解决多显示器问题
注意壁纸的分辨率应该符合显示器分辨率

## 安装Ceres

[http://ceres-solver.org/installation.html](http://ceres-solver.org/installation.html) 官网安装教程

[http://ceres-solver.org/ceres-solver-2.0.0.tar.gz](http://ceres-solver.org/ceres-solver-2.0.0.tar.gz)

下载最新的 2.0.0 的压缩包

# CMake
sudo apt-get install cmake
# google-glog + gflags
sudo apt-get install libgoogle-glog-dev libgflags-dev
# BLAS & LAPACK
sudo apt-get install libatlas-base-dev
# Eigen3
sudo apt-get install libeigen3-dev
# SuiteSparse and CXSparse (optional)
sudo apt-get install libsuitesparse-dev

tar zxf ceres-solver-2.0.0.tar.gz
mkdir ceres-bin
cd ceres-bin
cmake ../ceres-solver-2.0.0
make -j3
make test
sudo make install    # 这个和官网安装教程不一致，需要sudo

2.0.0版本不满足，安装1.14

进入 ceres-bin 执行`sudo make uninstall`

然后解压

mkdir ceres-bin
cd ceres-bin
cmake ../ceres-solver-1.14.0
make -j3
make test
sudo make install    # 这个和官网安装教程不一致，需要sudo

## 安装网络可视化工具zetane

[https://github.com/shanjiayao/viewer](https://github.com/shanjiayao/viewer)
