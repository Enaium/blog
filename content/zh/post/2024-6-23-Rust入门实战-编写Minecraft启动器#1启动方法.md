---
layout: post
title: "Rust入门实战 编写Minecraft启动器#1启动方法"
date: 2024-06-23T22:12:39+08:00
---

## 引言

想必大家都知道`Minecraft`这个游戏，它是一个非常有趣的游戏，有没有想过它是如何启动的呢？在本系列中，我们将使用`Rust`编写一个简单的`Minecraft`启动器。

本系列文章涉及的`Rust`知识并不多，了解`Rust`的基本语法即可，如果你对`Minecraft`或者`Java`有一定了解，那么会更容易理解本系列的内容。

## 观前须知

本系列文章只考虑如何完成一个简单的`Minecraft`启动器，不考虑支持启动老的游戏版本，也不考虑第三方的客户端，更不会去考虑正版验证等等，只是一个简单的启动器。

## 启动方法

首先`Minecarft`是使用`Java`编写的，所以我们需要通过执行`Java`命令来启动`Minecraft`，并且也需要添加一些参数，比如游戏的一些资源文件，`Java`的一些参数等等。

大概就像这样：

```shell
java -Xmx10745m -cp "minecraft.jar;lib/*" -Djava.library.path="natives" net.minecraft.client.main.Main --username yourname --version 1.21
```

### 下载资源

第一步我们需要下载`Minecraft`的资源文件，包括游戏本体、依赖库、资源文件。

`http://launchermeta.mojang.com/mc/game/version_manifest.json`这个地址可以读取到游戏所有版本。

`https://piston-meta.mojang.com/v1/packages/<sha1>/<version>.json`这个地址可以获取到游戏的配置信息。

`http://resources.download.minecraft.net/<sha1_first_two>/<sha1>`这个地址可以获取到游戏的资源文件。

`https://libraries.minecraft.net/<group>/<artifact>/<version>/<artifact>-<version>.jar`这个地址可以获取到游戏的依赖库。

### 游戏目录结构

首先游戏的所有资源都在一个目录中，这个目录包括`libraries`、`assets`、`versions`等文件夹，这个目录通常是`.minecraft`。

`libraries`文件夹存放游戏的依赖库，目录结构和`Maven`的目录结构类似。

`assets`文件夹存放游戏的资源文件，其中包括`objects`和`indexes`，`objects`存放资源文件，所有的资源文件都是以`sha1`命名的，所以`objects`文件夹下有一个以2位`sha1`命名的文件夹，所有前两位`sha1`相同的资源文件都在这个文件夹下，`indexes`存放资源文件的索引文件，也就是包含了资源文件的真实名称和对应的`sha1`值。

`versions`文件夹存放游戏的版本文件，每个版本都有一个以版本号命名的文件夹，这个文件夹包括`natives`，它存放游戏的本地库文件，我们需要在游戏启动的时候将所有`native`的`jar`文件都解压到这个文件夹下，这个文件夹还包括`<version>.json`，这个文件包括了游戏的配置信息，比如`mainClass`，`assets`等等，还有`<version>.jar`，这个文件是游戏的本体文件。

### 拼接命令

完成以上步骤后，我们就可以拼接命令了，首先是`java`的程序路径，然后是`JVM`参数，然后是`classpath`，接着是`main`方法的类路径，最后是`Minecraft`的参数。

拼接完成之后就可以执行这个命令启动游戏了。

## 项目结构

本项目分为 4 个模块：`download`, `launch`, `model`, `parse`。

- `download`模块用于下载游戏资源文件。

- `launch`模块用于启动游戏。

- `model`模块用建立游戏的数据模型。

- `parse`模块用于解析游戏的配置文件。

## 流程

```mermaid
graph LR
    建立资源模型 --> 解析资源配置 --> 下载游戏资源 --> 拼接启动命令 --> 启动游戏
```

## 结语

好了，本文就到这里了，之后的一些内容会在后续文章中介绍。