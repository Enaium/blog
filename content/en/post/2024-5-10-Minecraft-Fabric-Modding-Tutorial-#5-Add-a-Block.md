---
layout: post
title: "Minecraft Fabric Modding Tutorial #5 Add a Block"
date: 2024-05-10T23:11:31+08:00
---

## Register a Block

It is similar to registering an item, first, we need to declare a block, but a block has a `strength` property which is the time required for the player to mine a block.

```java
public static final Block END_HEART_BLOCK  = new Block(Block.Settings.create().strength(4.0f));
```

Next, we need to register the block, similar to registering an item, we need to register the block in the `onInitialize` method.

```java
public static final Block END_HEART_BLOCK = Registry.register(Registries.BLOCK, new Identifier("awesome:end_heart_block"), new Block(Block.Settings.create().strength(4.0f)));
```

Although we have registered the block, we can not get the item of this block, because we have not registered the item of this block.

```java
public static final Item END_HEART_BLOCK_ITEM = Registry.register(Registries.ITEM, new Identifier("awesome:end_heart_block"), new BlockItem(END_HEART_BLOCK, new Item.Settings()));
```

## Add Textures

We need the block's state, model, item, and texture.

### Block State

Location: `assets/awesome/blockstates/end_heart_block.json`

```json
{
  "variants": {
    "": {
      "model": "awesome:block/end_heart_block"
    }
  }
}
```

### Block Model

Location: `assets/awesome/models/block/end_heart_block.json`

```json
{
  "parent": "block/cube_all",
  "textures": {
    "all": "awesome:block/end_heart_block"
  }
}
```

### Block Item Model

Location: `assets/awesome/models/item/end_heart_block.json`

```json
{
  "parent": "awesome:block/end_heart_block"
}
```

### Block Texture

Location: `assets/awesome/textures/block/end_heart_block.png`

![end_heart_block](https://s2.loli.net/2024/05/11/Cow9HdE5FbVqMPT.png)

### Block Drop Items

We need to configure a loot table of the block, it drops items when the block is broken.

It must have a `survives_explosion` condition, otherwise, the block is unable to drop items if it is broken by an explosion.

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

### Block Class

It similar item and we may also create a block class.

```java
package com.example.blocks;

import net.minecraft.block.Block;

public class EndHeartBlock extends Block {
    public EndHeartBlock(Settings settings) {
        super(settings);
    }
}
```

Finally, we add the block to the item group.

```patch
+import com.example.blocks.EndHeartBlock;
 import com.example.items.EndHeartItem;
 import net.fabricmc.api.ModInitializer;
 
@@ -20,12 +21,15 @@ public class ExampleMod implements ModInitializer {
     public static final Logger LOGGER = LoggerFactory.getLogger("awesome");
 
        public static final Item END_HEART = Registry.register(Registries.ITEM, new Identifier("awesome:end_heart"), new EndHeartItem(new Item.Settings()));
-    public static final Block END_HEART_BLOCK = Registry.register(Registries.BLOCK, new Identifier("awesome:end_heard_block"), new Block(Block.Settings.create().strength(4.0f)));
+    public static final Block END_HEART_BLOCK = Registry.register(Registries.BLOCK, new Identifier("awesome:end_heard_block"), new EndHeartBlock(Block.Settings.create().strength(4.0f)));
     public static final Item END_HEART_BLOCK_ITEM = Registry.register(Registries.ITEM, new Identifier("awesome:end_heart_block"), new BlockItem(END_HEART_BLOCK, new Item.Settings()));
        public static final ItemGroup ITEM_GROUP = Registry.register(Registries.ITEM_GROUP, new Identifier("awesome:item_group"), FabricItemGroup.builder()
                        .icon(() -> new ItemStack(END_HEART))
                        .displayName(Text.literal("Awesome"))
-                       .entries(((displayContext, entries) -> entries.add(END_HEART)))
+                       .entries(((displayContext, entries) -> {
+                entries.add(END_HEART);
+                               entries.add(END_HEART_BLOCK);
+            }))
                        .build());
```

![20240511021524](https://s2.loli.net/2024/05/11/XjQPeUMWiuYv83r.png)