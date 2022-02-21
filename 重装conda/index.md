# 重装Conda


由于之前的conda版本比较低，且环境较为混乱，可能是误删了什么东西，丢失了base环境，综合考虑以上种种因素，决定重装conda~

<!--more-->

## 1. 卸载

找到conda的文件夹

`sudo rm -rf your_dir`

注释掉conda的环境配置

`sudo gedit ~/.bashrc`

## 2. 重新安装

下载conda[官网](https://www.anaconda.com/products/individual#macos)

点击下载, 选 `64-Bit (x86) Installer (550 MB)`

下载好之后, 找到目录

给权限, 运行

```bash
sudo chmod 777 Anaconda3-2020.07-Linux-x86_64.sh  
bash Anaconda3-2020.07-Linux-x86_64.sh
```

安装过程中, 在安装目录处修改目录, 第二个问题选yes

## 3. 环境配置

配置conda环境有几种方式

### 私人使用
首先第一种,刚刚安装时第二个问题选yes的话, 默认应该已经配置好了, 在~/.bashrc下.应该已经有conda的配置了, 如下:

```bash
__conda_setup="$('/usr/local/conda/anaconda3/bin/conda' 'shell.bash' 'hook' 2> /dev/null)"  
if [ $? -eq 0 ]; then  
 eval "$__conda_setup"  
else  
 if [ -f "/usr/local/conda/anaconda3/etc/profile.d/conda.sh" ]; then  
 . "/usr/local/conda/anaconda3/etc/profile.d/conda.sh"  
 else  
 export PATH="/usr/local/conda/anaconda3/bin:$PATH"  
 fi  
fi  
unset __conda_setup
```

这时conda就已经可以使用了

### 公共使用

那因为我刚刚安装到根目录下了,这种安装方式适用于服务器等多用户, 安装到根目录下之后就可以多用户一起使用conda, 这时可以直接配置profile文件, 这样在其他用户下就可以共享conda的环境了,如下:

```bash
sudo gedit /etc/profile  
export PATH=/usr/local/conda/anaconda3/bin:$PATH # 注意路径  
source /etc/profile
```


不过我自己的电脑只需要自己用,而且平时我需要使用ros,所以需要系统的python, 如果不是需要, 我不会启动conda的环境, 那么这时我们可以取消第一种和第二种的配置, 直接更改自己的bashrc文件

如下:

```bash
alias de='conda deactivate'  
alias base='. /usr/local/conda/anaconda3/etc/profile.d/conda.sh && conda activate base'
```

更改之后重启或者重新打开终端即可
