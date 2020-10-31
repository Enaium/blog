---
layout: post
title: "[BlocklyNukkit入门]#6简单窗口"
data: 2020-10-31 23:05
categories: blocklynukkit
---

```javascript
manager.newCommand("pa", "pa", function (sender, args) {
    var pa = window.getSimpleWindowBuilder("Pa", "选择一个玩家");//创建一个简单的窗口
    //遍历全服玩家
    onlines = Java.type("cn.nukkit.Server").getInstance().getOnlinePlayers().values().toArray();
    for (let index = 0; index < onlines.length; index++) {
        const element = onlines[index];
        pa.buildButton(element.getName(), "");//添加按钮 参数1标题 参数2图片 没有留空
    }
    pa.showToPlayer(sender, "Call");//参数1显示的玩家 参数2函数名
});//创建命令


function Call(e)//显示给玩家的函数
{
    server.getPlayer(window.getEventResponseText(e)/* 获取按下的按钮名 */)/* 获取玩家 */.sendMessage("Pa!");//给玩家发送消息
}
```