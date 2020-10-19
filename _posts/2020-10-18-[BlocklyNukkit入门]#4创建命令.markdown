---
layout: post
title: "[BlocklyNukkit入门]#4创建命令"
data: 2020-10-18 22:26
categories: blocklynukkit
---

## 喊话

创建一个命令

```javascript
manager.newCommand("shout", "喊话", function (sender, args) { });
```

Python可以吧function改为方法名 比如

```python
manager.createCommand("shout", u"喊话", "myCallBack")

def myCallBack(sender, args):
    pass
```

遍历全服玩家

```javascript
alllist = Java.type("cn.nukkit.Server").getInstance().getOnlinePlayers().values().toArray();
```

向全服玩家发送消息

```javascript
    for (var i = 0; i < alllist.length; i++) {
        alllist[i].sendMessage(message);
    }
```