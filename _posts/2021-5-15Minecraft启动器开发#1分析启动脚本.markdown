---
layout: post
title: "Minecraft启动器开发#1分析启动脚本"
data: 2021-5-15 11:11
categories: java
---

用HMCL生成启动脚本

```java
var java = System.getProperty("java.home");
```

获取运行的java路径

`-Dminecraft.client.jar=xxx`没用可以删掉

`-XX:xx`JVM参数 可能需要

`-Dfml.ignoreInvalidMinecraftCertificates=true -Dfml.ignorePatchDiscrepancies=true -XX:HeapDumpPath=MojangTricksIntelDriversForPerformance_javaw.exe_minecraft.exe.heapdump`

这段可以直接删掉

`-Djava.library.path` navite 启动需要的native

`-Dminecraft.launcher.brand`启动器

`-Dminecraft.launcher.version`启动器版本

`-cp` classpath 启动需要的库

`net.minecraft.client.main.Main` 启动的主类

`--username`用户名(玩家名)

`--version`游戏左下角显示的版本

`--gameDir`游戏运行的目录save、log等文件会生成在这里

`--assetsDir`游戏资源路径

`--assetIndex`游戏资源的Index文件名

`--uuid`玩家的UUID

`--accessToken` Token会过期 每次登录都会重新生成

`--userProperties {}`玩家的资产 不用管

`--userType`玩家类型 mojang表示是mojang账户

`--width 854 --height 480`窗口的大小