---
title: "星露谷MOD开发教程 #1 配置开发环境"
date: 2020-2-9 19:00
categories: stardewvalley
---

## 要求

* 安装了[SMAPI](smapi.io/)
* IDE (推荐使用Visual Studio或者Rider)

## 准备

1. 创建一个类库项目

![](/assets/stardewvalley/2020-2-9-1.png)

2. 引用[Pathoschild.Stardew.ModBuildConfig](https://www.nuget.org/packages/Pathoschild.Stardew.ModBuildConfig)Nuget包

![](/assets/stardewvalley/2020-2-9-2.png)

3. 创建一个类

![](/assets/stardewvalley/2020-2-9-3.png)

4. 新建`manifest.json`文件

![](/assets/stardewvalley/2020-2-9-4.png)

格式

```json
{
  "Name": "<your project name>",
  "Author": "<your name>",
  "Version": "1.0.0",
  "Description": "<One or two sentences about the mod>",
  "UniqueID": "<your name>.<your project name>",
  "EntryDll": "<your project name>.dll",
  "MinimumApiVersion": "2.10.0",
  "UpdateKeys": []
}
```

## 开始

继承父类`Mod`

添加一个事件

```c#
using StardewModdingAPI;
using StardewModdingAPI.Events;

namespace NewMod
{
     public class NewMod : Mod
     {
         public override void Entry(IModHelper helper)
         {
             helper.Events.Input.ButtonPressed += onButtonPressed;
         }

         private void onButtonPressed(object sender, ButtonPressedEventArgs e)
         {
             //如果世界没有完成返回
             if(!Context.IsWorldReady)
                return;
            
             //输出玩家按下的某个键
             Monitor.Log(e.Button.ToString(),LogLevel.Debug);
         }
     }
 }
```

这样一个mod就完成了