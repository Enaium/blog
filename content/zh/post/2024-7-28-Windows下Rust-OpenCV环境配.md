---
layout: post
title: "Windows下Rust OpenCV环境配"
date: 2024-07-28T21:14:31+08:00
---

## 安装Chocolatey

首先我们需要[安装`Chocolatey`](https://chocolatey.org/)，`Chocolatey`是一个Windows的包管理器。

我们点击右上角的`Install`进入到`Installing Chocolatey`，选择`Individual`

复制命令
```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
```

使用管理员模式打开`PowerShell`，粘贴命令，等待安装完成。

## 安装llvm和opencv

安装过程中可能会由于网络问题导致安装失败，可以多次尝试。

```shell
choco install llvm opencv
```

## 配置环境变量

首先设置`OPENCV_INCLUDE_PATHS`环境变量，值为`C:\tools\opencv\build\include`，然后设置`OPENCV_LINK_PATHS`环境变量，值为`C:\tools\opencv\build\x64\vc16\lib`，接着设置`OPENCV_LINK_LIBS`环境变量，值为`opencv_worldxxxx`，`xxxx`是你的`OpenCV`版本号，例如`opencv_world4100`。

最后我们需要将`C:\tools\opencv\build\x64\vc16\bin`添加到`Path`环境变量中。

## 配置Cargo.toml

在`Cargo.toml`中添加如下内容：

```toml
[dependencies]
opencv = "0.92.1"
```

## 测试

```rs
use opencv::{Error, highgui, imgcodecs};

fn main() -> Result<(), Error> {
    let image_path = "images/mugshot.png";
    let image = imgcodecs::imread(&image_path, imgcodecs::IMREAD_COLOR)?;

    highgui::imshow("trump.png", &image)?;
    highgui::wait_key(0)?;
    return Ok(());
}
```

运行`cargo run`，如果一切正常，你将看到一张图片弹出。

![trump.png](https://s21.ax1x.com/2024/07/28/pkqHbVg.png)