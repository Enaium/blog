---
layout: post
title: "Minecraft Client 教程 #3 添加OptiFine"
date: 2020-2-25 12:16
categories: minecraftclient
---

一. 下载[Optifine](https://optifinesource.co.uk/)本站[下载](/assets/minecraftclient/OptiFine.zip)

二. 删除src\minecraft所有内容 将Optifine解压进去(保留你写的包)

![3-1](/assets/minecraftclient/2-1.png)

三. 根据#2的方法修改Minecraft类调用Start 和 Stop

四. 导入[vecmath](/assets/minecraftclient/vecmath-1.5.2.jar)包

![3-2](/assets/minecraftclient/3-2.png)

五. 修改类名`net.minecraft.client.renderer`包下的`EntityRenderer$1` `EntityRenderer$2` 改为`EntityRenderer1` `EntityRenderer2`

六. 启动


![3-3](/assets/minecraftclient/3-3.png)




