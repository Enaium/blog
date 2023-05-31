---
layout: post
title: "解决IDEA创建或打开项目时下载Gradle"
date: 2023-02-03T15:42:31+08:00
---

# 引言

我们在使用IDEA创建或打开一个Gradle项目时,IDEA会下载一个新或旧的Gradle版本,虽然下载很快,但到下次IDEA更新后又会是一个新版的Gradle版本,这样很影响我们的开发效率,所以我做了一个东西,可以让IDEA在创建或打开一个项目时使用自己指定的版本


## 使用

我做了一个Agent,可以让他来加载自己写的`Transformer`大大减少了开发时间,它叫做[Cafully](https://github.com/Cafully/cafully)

首先下载它到任意位置

接着在给IDEA添加下面JVM参数(idea64.exe.vmoptions),agent的位置就是刚才下载的位置

```
--add-opens=java.base/jdk.internal.org.objectweb.asm=ALL-UNNAMED
--add-opens=java.base/jdk.internal.org.objectweb.asm.tree=ALL-UNNAMED

-javaagent:/absolute/path/to/cafully-agent.jar
```

## 安装插件

下载好后放入Agent同级目录下的`plugin`文件夹里面

[cafully-plugin-asm](https://github.com/Cafully/cafully-plugin-asm) 这个是ASM插件,用来生成和修改字节码

接下来下载解决这个问题的插件

[KeepGradleVersion](https://github.com/Enaium/KeepGradleVersion)

## 配置

首先呢这个插件有一个配置,在Agent同级目录下的`config`文件夹里面放入`keep-gradle-version.properties`文件即可

```properties
#默认 7.6, Gradle的版本
version=7.6
#默认 false, 是否在当打开一个项目时使用指定的版本
open=true
```

这样IDEA无论在创建或打开一个项目时使用指定的版本了

[视频](https://www.bilibili.com/video/BV1Ye4y1N7Km)