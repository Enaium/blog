---
layout: post
title: "Minecraft Fabric Modding Tutorial #6 Language Recipes Tooltip"
date: 2024-05-11T21:05:53+08:00
---

## Language

We had added an item, an item group, and a block, but they do not have a name yet. Here, We can use a language file to provide a name for them.

We can create a language file `en_us.json` under the path `assets/awesome/lang`, it is an English language file.

```json
{
  "item.awesome.end_heart": "End Heart",
  "item.awesome.end_heart_block": "End Heart Block",
  "block.awesome.end_heart_block": "End Heart Block",
  "itemGroup.awesome.item_group": "Awesome"
}
```

![20240511212016](https://s2.loli.net/2024/05/11/Pj8OohkJpaLfS7b.png)

## Recipes

The recipe is used to craft an item, Here, We can configure the recipe of the item.

Create a recipe file `end_heart_block.json` under the path `data/awesome/recipes`, which is used to craft the item `end_heart_block`.

```json
{
  "type": "minecraft:crafting_shaped",
  "pattern": [
    "EEE",
    "EEE",
    "EEE"
  ],
  "key": {
    "E": {
      "item": "awesome:end_heart"
    }
  },
  "result": {
    "id": "awesome:end_heart_block",
    "count": 1
  }
}
```

In this file, the type is the crafting type, the pattern is the put place of the item, the key is an alias of the items, and the result is the result of crafting the item.

Next, we need to create an item recipe file `end_heart.json` to decompose the item.

```json
{
  "type": "minecraft:crafting_shapeless",
  "ingredients": [
    {
      "item": "awesome:end_heart_block"
    }
  ],
  "result": {
    "id": "awesome:end_heart",
    "count": 9
  }
}
```

In this file, the type is `crafting_shapeless`, it does not follow the put place. The `ingredients` are the required items for crafting, and the `result` is the result of crafting the item, the count is the number of the result.

![20240511231146](https://s2.loli.net/2024/05/11/nzp3FYksuotEHvr.png)
![20240511231213](https://s2.loli.net/2024/05/11/JMBrdhS5n8xTORa.png)

## Tooltip

We can add a tooltip to the item, it shows when the mouse is over the item

```patch
 package com.example.items;

+import net.minecraft.client.item.TooltipType;
 import net.minecraft.entity.player.PlayerEntity;
 import net.minecraft.item.Item;
 import net.minecraft.item.ItemStack;
 import net.minecraft.sound.SoundEvents;
+import net.minecraft.text.Text;
 import net.minecraft.util.Hand;
 import net.minecraft.util.TypedActionResult;
 import net.minecraft.world.World;

+import java.util.List;

 /**
  * @author Enaium
  */
@@ -25,9 +21,4 @@ public class EndHeartItem extends Item {
         user.playSound(SoundEvents.BLOCK_END_PORTAL_FRAME_FILL, 1.0F, 1.0F);
         return super.use(world, user, hand);
     }
+
+    @Override
+    public void appendTooltip(ItemStack stack, TooltipContext context, List<Text> tooltip, TooltipType type) {
+        tooltip.add(Text.translatable("tooltip.awesome.end_heart"));
+    }
 }
```

We can override the `appendTooltip` method to add a tooltip to the item.

```json
{
  "tooltip.awesome.end_heart": "End Heart"
}
```

Use the `Text.translatable` method to get the text from the language file.

![20240511232139](https://s2.loli.net/2024/05/11/lgAhNxOTZzkGos6.png)