---
layout: post
title: "Minecraft Fabric模组开发教程#1 配置开发环境"
date: 2024-01-09T18:17:05+08:00
---

## 前言

在几年前我已经写过很多关于Mincraft开发相关的文章，今年已经是2024年了所以我决定重新写一遍，这次我会更加详细的介绍一些内容。

## 安装JDK

由于最新的Minecraft最低只支持JDK17，所以我们需要安装JDK17，这里我选择下载[Libreica OpenJDK 17](https://bell-sw.com/pages/downloads/#/java-17-lts)。

![1-1](/assets/fabric2024/1-1.png)

这里选择`Full JDK`之后点击后缀为`msi`的文件下载，下载完成后双击安装即可。

## 安装IDEA

这里我选择下载[IDEA](https://www.jetbrains.com/idea/download/#section=windows)，选择`Community`版本即可。

![1-2](/assets/fabric2024/1-2.png)

下载完成后双击安装即可。

## 下载项目模板

这里需要到Fabric的GitHub仓库下载模板，[点击这里](https://github.com/FabricMC/fabric-example-mod)进入仓库，点击`Code`按钮，选择`Download ZIP`下载。

![1-3](/assets/fabric2024/1-3.png)

## 运行项目

下载完成后解压，打开IDEA，选择`Open`，选择解压后的文件夹，点击`OK`，最后点击`Trust Project`。

这样就算是将项目导入IDEA了，接下来只需等待IDEA加载完成即可，如果是第一次开发Fabric模组，可能会需要下载一些依赖，这个过程可能会比较慢，需要耐心等待。

点击左下角有一个`Build`图标的按钮，之后再右侧出现`BUILD SUCCESSFUL`时，说明项目已经完成构建，可以运行了。

之后点击右上角的`Gradle`图标的按钮，展开之后找到`Tasks`下的`fabric`，点击`runClient`，等待运行完成即可。

![1-4](/assets/fabric2024/1-4.png)