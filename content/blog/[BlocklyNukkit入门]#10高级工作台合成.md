---
title: "[BlocklyNukkit入门]#10高级工作台合成"
data: 2021-1-19 18:34
categories: blocklynukkit
---

创建一个物品
```javascript
item = blockitem.buildItem(280, 0, 1);
item.setCustomName("棍");
blockitem.setItemLore(item, "第一行;第二行;第三行;第四行");
```

添加高级工作台合成，背包里有材料才会显示

参数1类别，参数2描述，参数3材料，参数4合成的东西，参数5用时，参数6概率最大为1
```javascript
blockitem.addBNCraft("锻造", "合成棍", Java.to([item = blockitem.buildItem(264, 0, 1), item = blockitem.buildItem(41, 0, 2)], "cn.nukkit.item.Item[]"), Java.to([item], "cn.nukkit.item.Item[]"), 160, 1.0)

```

打开高级工作台合成

```javascript
manager.newCommand("dz", "锻造", function (sender, args) {
    blockitem.openBNCraftForPlayer("锻造", sender.getPlayer());
})
```
