---
title: "Minecraft Fabric Client 教程 #2 修改文件和目录名添加run和stop"
date: 2020-01-04T23:08:00+0800
categories: fabricclient
---

## 修改 包名、json文件、添加run stop


![a](/assets/fabricclient/2020-1-4-1.png)

首先先修改包名

![a](/assets/fabricclient/2020-1-4-2.png)

将`ExampleMod.java`改为`ExcelInitializer.java`

修改json文件`modid.mixins.json`和`fabric.mod.json`


将`modid.mixins.json`改为`excel.mixins.json`

`excel.mixins.json`内容：
```java

{
  "required": true,
  "package": "cn.enaium.excel.mixin",
  "compatibilityLevel": "JAVA_8",
  "mixins": [
  ],
  "client": [
    "ExampleMixin",
    "MinecraftClientMixin"
  ],
  "injectors": {
    "defaultRequire": 1
  }
}

```

`package`mixin的包名

`client` mixin的类名

`compatibilityLevel` java版本


`fabric.mod.json`内容：
```java
{
  "schemaVersion": 1,
  "id": "excel",
  "version": "1.0",

  "name": "Excel",
  "description": "!",
  "authors": [
    "Enaium!"
  ],
  "contact": {
    "homepage": "https://fabricmc.net/",
    "sources": "https://github.com/FabricMC/fabric-example-mod"
  },

  "license": "CC0-1.0",
  "icon": "assets/excel/icon.png",

  "environment": "*",
  "entrypoints": {
    "main": [
      "cn.enaium.excel.ExcelInitializer"
    ]
  },
  "mixins": [
    "excel.mixins.json"
  ],

  "depends": {
    "fabricloader": ">=0.7.2",
    "fabric": "*",
    "minecraft": "1.15.x"
  },
  "suggests": {
    "flamingo": "*"
  }
}

```

`id`就是modid
`name`mod名字
`description`说明
`authors`作者
`mixins`mixinjson的文件名
`license`如果有开源开源协议
`main`ModInitializer的文件名



然后在`cn.enaium.excel`新建一个枚举`Excel.java`

内容

```java
package cn.enaium.excel;

/**
 * @Author Enaium
 * @Date 2020/1/4 20:23
 */
public enum Excel {

    INSTANCE;

    public final String NMAE = "Excel";

    public final String VERSION = "1";

    public final String MINECRAFT_VERSION = "1.15.1";

    public void onEnable() {

    }

    public void onDisable() {

    }

}

```

## 注入run和stop

再mixin这个包里面新建一个`MinecraftClientMixin`类

内容

```java
package cn.enaium.excel.mixin;

import cn.enaium.excel.Excel;
import net.minecraft.client.MinecraftClient;
import org.spongepowered.asm.mixin.Mixin;
import org.spongepowered.asm.mixin.injection.At;
import org.spongepowered.asm.mixin.injection.Inject;
import org.spongepowered.asm.mixin.injection.callback.CallbackInfo;

/**
 * @Author Enaium
 * @Date 2020/1/4 20:25
 */
@Mixin(MinecraftClient.class)
public class MinecraftClientMixin {
    @Inject(at = @At("HEAD"), method = "run()V")
    private void onEnable(CallbackInfo info) {
        Excel.INSTANCE.onEnable();
    }

    @Inject(at = @At("HEAD"), method = "stop()V")
    private void onDisable(CallbackInfo info) {
        Excel.INSTANCE.onDisable();
    }
}
```

然后再mixin json 里面添加这个mixin

```java
{
  "required": true,
  "package": "cn.enaium.excel.mixin",
  "compatibilityLevel": "JAVA_8",
  "mixins": [
  ],
  "client": [
    "ExampleMixin",
    "MinecraftClientMixin"
  ],
  "injectors": {
    "defaultRequire": 1
  }
}
```
完成
