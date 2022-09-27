---
title: "Minecraft Fabric 教程 #5 添加语言文件"
date: 2019-12-15 20:26
categories: fabric
---

## 创建语言文件

lang也就是你模组的翻译比如 中文简体 zh_cn 中文正體 zh_tw 英文 en_us

resources/assets/endarmor/lang/zh_cn.json

```
{
  "item.endarmor.end_heart": "End心",
  "block.endarmor.end_heart_block": "End心块",
    [...]
}
```

## 格式

`<object-type>.<modid>.<registry-id>`

```
block.<modid>.<registry-id>
item.<modid>.<registry-id>
itemGroup.<modid>.<registry-id>
fluid.<modid>.<registry-id>
sound_event.<modid>.<registry-id>
mob_effect.<modid>.<registry-id>
enchantment.<modid>.<registry-id>
entity_type.<modid>.<registry-id>
potion.<modid>.<registry-id>
biome.<modid>.<registry-id>
```

获取翻译文件的翻译

```java
new TranslatableText("item.tutorial.fabric_item.tooltip_1")
```