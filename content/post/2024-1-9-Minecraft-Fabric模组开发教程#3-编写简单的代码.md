---
layout: post
title: "Minecraft Fabric模组开发教程#3 编写简单的代码"
date: 2024-01-09T21:50:27+08:00
---

## 前言

在上篇文章中我们已经了解了模组和项目的基本信息，这篇文章我们将会编写一些简单的代码。

## 调整模组信息

我们首先删除无用的文件。

### 删除`client`文件夹

删除之后我们还需要调整其他地方。

- 删除`build.gradle`中的`client`部分，由于之后不需要区分客户端和服务端，所以就把`loom`的一整块代码删除。
- 删除`src/main/resources`中的`fabric.mod.json`中的`client`部分。

```diff
        "entrypoints": {
                "main": [
                        "com.example.ExampleMod"
-               ],
-               "client": [
-                       "com.example.ExampleModClient"
                ]
        },
        "mixins": [
-               "modid.mixins.json",
-               {
-                       "config": "modid.client.mixins.json",
-                       "environment": "client"
-               }
+               "modid.mixins.json"
        ],
        "depends": {
                "fabricloader": ">=0.15.0",
```

### 修改模组信息

- 修改`id`、`name`、`description`、`authors`、`icon`。

```diff
 {
        "schemaVersion": 1,
-       "id": "modid",
+       "id": "awesome",
        "version": "${version}",
-       "name": "Example mod",
-       "description": "This is an example description! Tell everyone what your mod is about!",
+       "name": "Awesome",
+       "description": "This is an awesome mod",
        "authors": [
-               "Me!"
+               "Enaium"
        ],
        "contact": {
                "homepage": "https://fabricmc.net/",
                "sources": "https://github.com/FabricMC/fabric-example-mod"
        },
        "license": "CC0-1.0",
-       "icon": "assets/modid/icon.png",
+       "icon": "assets/awesome/icon.png",
        "environment": "*",
        "entrypoints": {
                "main": [
@@ -20,7 +20,7 @@
                ]
        },
        "mixins": [
-               "modid.mixins.json"
+               "awesome.mixins.json"
        ],
        "depends": {
                "fabricloader": ">=0.15.0",
```

- 修改`modid.mixins.json`为`awesome.mixins.json`
- 修改`assets/modid/icon.png`为`assets/awesome/icon.png`
- 修改`ExampleMod.java`中的日志名称

```diff
-    public static final Logger LOGGER = LoggerFactory.getLogger("modid");
+    public static final Logger LOGGER = LoggerFactory.getLogger("awesome");
```

## 编写代码

在`ExampleMixin`类中添加一行日志。

```diff
@@ -1,5 +1,6 @@
 package com.example.mixin;

+import com.example.ExampleMod;
 import net.minecraft.server.MinecraftServer;
 import org.spongepowered.asm.mixin.Mixin;
 import org.spongepowered.asm.mixin.injection.At;
@@ -11,5 +12,6 @@
        @Inject(at = @At("HEAD"), method = "loadWorld")
        private void init(CallbackInfo info) {
                // This code is injected into the start of MinecraftServer.loadWorld()V
+               ExampleMod.LOGGER.info("Hello Mixin World!");
        }
 }
```

## 测试

运行游戏，创建一个世界，查看日志。

```
[22:23:52] [Server thread/INFO] (awesome) Hello Mixin World!
```