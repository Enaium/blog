---
title: "Minecraft Fabric Client 教程 #3 添加自定义Command"
date: 2020-1-5 12:52
categories: fabricclient
---

## 添加ChatUtils

先在`cn.enaium.excel`里新建一个包`utils`

然后创建一个`ChatUtils.java`类

内容


```java
package cn.enaium.excel.utils;

import net.minecraft.client.MinecraftClient;
import net.minecraft.client.gui.hud.ChatHud;
import net.minecraft.text.LiteralText;
import net.minecraft.text.Text;

/**
 * @Author Enaium
 * @Date 2020/1/5 12:54
 */
public class ChatUtils {

    public static void component(Text component)
    {
        ChatHud chatHud = MinecraftClient.getInstance().inGameHud.getChatHud();
        LiteralText prefix = new LiteralText("\u00a7c[\u00a76Excel\u00a7c]\u00a7r ");
        chatHud.addMessage(prefix.append(component));
    }

    public static void message(String message)
    {
        component(new LiteralText(message));
    }
}

```


## 添加自定义command


先在`cn.enaium.excel`里新建一个包`command`

下载[![command.zip](/assets/icon/zip.png)](/assets/fabricclient/command.zip)

将压缩包里面的内容全部放进去



![a](/assets/fabricclient/2020-1-5-1.png)

然后再`Excel.java`里添加`command`

```java
    public CommandManager commandManager;

    public void onEnable() {
        commandManager = new CommandManager();
        commandManager.loadCommands();
    }
```

## 注入Mixin
在mixin包里面新建一个`ClientPlayerEntityMixin.java`类

内容

```java
package cn.enaium.excel.mixin;

import cn.enaium.excel.Excel;
import net.minecraft.client.network.ClientPlayerEntity;
import org.spongepowered.asm.mixin.Mixin;
import org.spongepowered.asm.mixin.injection.At;
import org.spongepowered.asm.mixin.injection.Inject;
import org.spongepowered.asm.mixin.injection.callback.CallbackInfo;

/**
 * @Author Enaium
 * @Date 2020/1/5 13:27
 */
@Mixin(ClientPlayerEntity.class)
public class ClientPlayerEntityMixin {

    @Inject(at = @At("HEAD"),
            method = "sendChatMessage(Ljava/lang/String;)V",
            cancellable = true)
    private void onSendChatMessage(String message, CallbackInfo info)
    {
        if (Excel.INSTANCE.commandManager.processCommand(message))
            info.cancel();
    }

}
```

添加到`mixin.json`里面



```java
  "client": [
    "ExampleMixin",
    "MinecraftClientMixin",
    "ClientPlayerEntityMixin"
  ],
```


启动


![a](/assets/fabricclient/2020-1-5-2.png)

输入`-`或者`-help`  输入返回的信息不会在控制台出现

完成