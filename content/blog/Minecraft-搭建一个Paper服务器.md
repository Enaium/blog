---
title: "Minecraft 搭建一个Paper服务器"
date: 2020-01-02T18:44:00+0800
categories: minecraft
---

## 下载核心

[Downloads – PaperMC](https://papermc.io/downloads)

我这里是1.15.1的版本

选最新版本就行了

![a](/assets/minecraft/2020-1-2-1.png)

## 运行

下载好后放到一个文件夹然后放到一个文件夹然后再新建一个bat文件`Launcher.bat`里面写

`java -Xms512M -Xmx1024M -jar paper.jar`

然后发现下载好后瞬间退出了是因为要同意一些eula

打开`eula.exe`将`eula=false`改成`就好了`

然后再次运行

![a](/assets/minecraft/2020-1-2-2.png)

这样一个服务器就开好了
