---
layout: post
title: "[BlocklyNukkit入门]#4喊话"
data: 2020-10-18 22:26
categories: blocklynukkit
---

## 喊话

创建一个命令

```js
manager.newCommand("shout", "喊话", function (sender, args) { });
```

Python可以吧function改为方法名,然后写一个方法来实现

遍历全服玩家

```js
alllist = Java.type("cn.nukkit.Server").getInstance().getOnlinePlayers().values().toArray();
```

向全服玩家发送消息

```js
    for (var i = 0; i < alllist.length; i++) {
        alllist[i].sendMessage(message);
    }
```