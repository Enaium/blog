---
title: "Minecraft Fabric模组开发教程#5 添加方块"
date: 2024-01-10T01:53:12+08:00
---

## 注册方块

和注册物品相同，我们先声明一个方块对象，和物品不同的是，方块有一个`strength`属性，这个属性代表方块的硬度，也就是玩家破坏方块需要花费的时间，这个属性的值越大，方块的硬度越高，破坏方块需要花费的时间越长。

```java
public static final Block END_HEART_BLOCK  = new Block(FabricBlockSettings.create().strength(4.0f));
```

之后我们需要注册这个方块，和注册物品相同，我们需要在`onInitialize`方法中注册方块。

```diff
@@ -31,5 +31,6 @@

     @Override
     public void onInitialize() {
+        Registry.register(Registries.BLOCK, new Identifier("awesome", "end_heart_block"), END_HEART_BLOCK);
     }
 }
```

虽然说已经将方块注册到了游戏中，但是我们还是获取不到这个方块的物品，因为我们还没有注册这个方块的物品。

```diff
@@ -32,5 +32,6 @@
     @Override
     public void onInitialize() {
         Registry.register(Registries.BLOCK, new Identifier("awesome", "end_heart_block"), END_HEART_BLOCK);
+        Registry.register(Registries.ITEM, new Identifier("awesome", "end_heart_block"), new BlockItem(END_HEART_BLOCK, new FabricItemSettings()));
     }
 }
```

## 添加纹理

需要方快的状态配置、方快模型配置、方快物品模型配置和方快纹理。

### 方块状态配置

位置:`assets/awesome/blockstates/end_heart_block.json`

```json
{
  "variants": {
    "": { "model": "awesome:block/end_heart_block" }
  }
}
```

### 方块模型配置

位置:`assets/awesome/models/block/end_heart_block.json`

```json
{
  "parent": "block/cube_all",
  "textures": {
    "all": "awesome:block/end_heart_block"
  }
}
```

### 方块物品模型配置

位置:`assets/awesome/models/item/end_heart_block.json`

```json
{
  "parent": "awesome:block/end_heart_block"
}
```

### 方块纹理

位置:`assets/awesome/textures/block/end_heart_block.png`

![end_heart](/assets/fabric/end_heart_block.png)

## 方快掉落物

掉落物就是玩家破坏方块后掉落的物品，其中`minecraft:survives_explosion`是一个条件，代表方块在爆炸后是否会掉落物品，这个条件是必须的，否则方块在爆炸后不会掉落物品。

位置:`data\awesome\loot_tables\blocks\end_heart_block.json`

```json
{
  "type": "minecraft:block",
  "pools": [
    {
      "rolls": 1,
      "entries": [
        {
          "type": "minecraft:item",
          "name": "awesome:end_heart_block"
        }
      ],
      "conditions": [
        {
          "condition": "minecraft:survives_explosion"
        }
      ]
    }
  ]
}
```

## 方快类

和物品一样，我们也可以创建一个方快类。

```java
package com.example.blocks;

import net.minecraft.block.Block;

public class EndHeartBlock extends Block {
    public EndHeartBlock(Settings settings) {
        super(settings);
    }
}
```

```diff
@@ -22,7 +22,8 @@
     public static final Item END_HEART =
             Registry.register(Registries.ITEM, new Identifier("awesome", "end_heart"),
                     new EndHeartItem(new FabricItemSettings()));
-    public static final Block END_HEART_BLOCK = new EndHeartBlock(FabricBlockSettings.create().strength(4.0f));
+    public static final Block END_HEART_BLOCK = Registry.register(Registries.BLOCK, new Identifier("awesome", "end_heart_block"),
+            new EndHeartBlock(FabricBlockSettings.create().strength(4.0f)));
     private static final ItemGroup ITEM_GROUP = Registry.register(Registries.ITEM_GROUP, new Identifier("awesome", "item_group"), FabricItemGroup.builder()
             .icon(() -> new ItemStack(END_HEART))
             .displayName(Text.translatable("itemGroup.awesome.item_group"))
@@ -33,7 +34,6 @@

     @Override
     public void onInitialize() {
-        Registry.register(Registries.BLOCK, new Identifier("awesome", "end_heart_block"), END_HEART_BLOCK);
         Registry.register(Registries.ITEM, new Identifier("awesome", "end_heart_block"), new BlockItem(END_HEART_BLOCK, new FabricItemSettings()));
     }
 }
```

---

最后我们将方快添加到物品组中。

```diff
@@ -29,6 +29,7 @@
             .displayName(Text.translatable("itemGroup.awesome.item_group"))
             .entries((context, entries) -> {
                 entries.add(END_HEART);
+                entries.add(END_HEART_BLOCK);
             })
             .build());
```

![5-1](/assets/fabric2024/5-1.png)