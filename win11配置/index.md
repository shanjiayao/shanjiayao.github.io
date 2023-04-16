# win11配置





<!--more-->


## 修改资源管理器的右键菜单为不折叠

Win11的右键菜单默认折叠时改回Win10样式：

首先Win+X打开Windows powershell（管理员）（现在叫Windows 终端（管理员））

1）切换回经典右键菜单

reg add "HKCU\Software\Classes\CLSID\{86ca1aa0-34aa-4e8b-a509-50c905bae2a2}\InprocServer32" /f /ve

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20221113121618.png)

2）恢复到新版右键菜单

reg delete "HKCU\Software\Classes\CLSID\{86ca1aa0-34aa-4e8b-a509-50c905bae2a2}" /f

选择想要的类型，粘贴后回车。

Win+E打开Windows资源管理器，然后Ctrl+Shift+Esc任务管理器重启Windows资源管理器。

单击右键，大功告成。

## 系统监控

原有的win10系统使用的是Xmeters

但是win11发现这个软件不支持，所以更换了 https://github.com/zhongyang219/TrafficMonitor

最终效果如下：

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20221113131652.png)


