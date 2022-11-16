---
layout: post
title: "Linux下制作Windows启动U盘"
date: 2019-12-28T21:24:00+08:00
categroy: linux
---

## 安装WoeUsb

`sudo apt-get install woeusb`

或者

[WosUsb](https://github.com/slacka/WoeUSB)


## 找一个U盘

格式化为NTFS格式


## 制作

`sudo woeusb -d /media/e/Enaium/镜像/windows64/win101909.iso /dev/sdb --target-filesystem NTFS`

注意路径！


接下来等就好了

![img](/assets/linux/2019-12-28-1.png)
