---
layout: post
title: "[BlocklyNukkit入门]#3玩家进服欢迎"
data: 2020-10-3 21:09
categories: blocklynukkit
---

## 进服欢迎

我们可以在bn的[文档](http://www.blocklynukkit.info/1735257)里查到`PlayerJoinEvent`玩家进入服务器的事件 这样我们就可以很“方便”的编写插件了

### JavaScript

```javascript
function PlayerJoinEvent(e) {
    e.getPlayer().sendMessage("欢迎" + e.getPlayer().getName() + "进入服务器！JavaScript");
}
```

### Python

```python
def PlayerJoinEvent(e):
    e.getPlayer().sendMessage(u"欢迎" + e.getPlayer().getName() + u"进入服务器！Python")
```

### Lua

```lua
function PlayerJoinEvent(e)
    e:getPlayer():sendMessage("欢迎" .. e:getPlayer():getName() .. "进入服务器！Lua")
end
```

