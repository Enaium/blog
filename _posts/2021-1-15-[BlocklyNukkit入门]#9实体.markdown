---
layout: post
title: "[BlocklyNukkit入门]#9实体"
data: 2021-1-15 15:03
categories: blocklynukkit
---

当玩家使用铁锭时,把掉落物传送到玩家的位置.

事件`PlayerInteractEvent` 玩家交互,比如使用等等.

```javascript
function PlayerInteractEvent(e) {
    player = e.getPlayer()//获取事件的玩家
    item = blockitem.getItemInHand(player)//获取玩家手中的物品
    dropItems = blockitem.getDropItems(player.getPosition())//获取玩家所在世界的掉落物
    if (item.getId() == 265) {//获取物品ID是否为铁锭
        for (let index = 0; index < dropItems.length; index++) {
            const element = dropItems[index];
            element.setPosition(player.getPosition())//设置物品位置到玩家的位置
        }
    }
}
```