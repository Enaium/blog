---
layout: post
title: "[BlocklyNukkit入门]#5自定义物品"
date: 2020-10-19T19:51:00+08:00
categroy: blocklynukkit
---

## 自定义物品

### 创建一个木棍

```javascript
item = blockitem.buildItem(280, 0, 1);
```

设置名字

```javascript
item.setCustomName("棍");
```

设置信息,用分号隔开换行

```javascript
blockitem.setItemLore(item, "第一行;第二行;第三行;第四行");
```

### 添加有序合成

添加有序合成,设置G为橡木原木的键,G就代表原木.
参数1用字符串数组类型,3个字符串代表合成台的3行,每一行有3个物品,用键来代表,空格代表没物品.
参数2是合成后的物品.
参数3是追加结果物品，物品数组类型，比如合成蛋糕会返给3个桶.

```javascript
manager.putEasy("G", blockitem.buildItemFromBlock(blockitem.buildBlock(17, 0)));
blockitem.addShapedCraft(Java.to(["G  ", "G  ", "G  "], "java.lang.String[]"), item, Java.to([], "cn.nukkit.item.Item[]"));
```

### 创建第二个木棍

```javascript
superItem = blockitem.buildItem(369, 0, 1);
superItem.setCustomName("Super棍");
blockitem.setItemLore(superItem, "第五行;第六行;第七行;第八行;第九行");
```

### 添加无序合成

参数1就是需要合成的配方，参数2就是合成后的物品

```javascript
blockitem.addShapelessCraft(Java.to([item, blockitem.buildItemFromBlock(blockitem.buildBlock(41, 0))], "cn.nukkit.item.Item[]"), superItem);
```
