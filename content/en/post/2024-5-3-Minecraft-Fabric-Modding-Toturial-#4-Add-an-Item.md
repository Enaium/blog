---
layout: post
title: "Minecraft Fabric Modding Toturial #4 Add an Item"
date: 2024-05-03T16:53:43+08:00
---

## Register an Item

First, we need to declare an item in the `ExampleMod` class.

```patch
+       public static final Item END_HEART = new Item(new Item.Settings());
+
        @Override
        public void onInitialize() {
                // This code runs as soon as Minecraft is in a mod-load-ready state.
```

Next, register the item in the `onInitialize` method.

```patch
package com.example;
 import net.fabricmc.api.ModInitializer;
 
 import net.minecraft.item.Item;
+import net.minecraft.registry.Registries;
+import net.minecraft.registry.Registry;
+import net.minecraft.util.Identifier;
 import org.slf4j.Logger;
 import org.slf4j.LoggerFactory;
 
@@ -16,10 +19,6 @@ public class ExampleMod implements ModInitializer {
 
        @Override
        public void onInitialize() {
-               // This code runs as soon as Minecraft is in a mod-load-ready state.
-               // However, some things (like resources) may still be uninitialized.
-               // Proceed with mild caution.
-
-               LOGGER.info("Hello Fabric world!");
+               Registry.register(Registries.ITEM, new Identifier("awesome:end_heart"), END_HEART);
        }
 }
```

In this code, first parameter of the `Registry.register` method is the registry type, the second parameter is the item id, and the third parameter is the item instance.

We can use the command `/give @p awesome:end_heart` to get the item in the game.

![20240503171137](https://s2.loli.net/2024/05/03/4XKRwmV5qhBDc6C.png)

We may also direct invoke the `Registry.register` method.

```patch
     public static final Logger LOGGER = LoggerFactory.getLogger("awesome");
 
-       public static final Item END_HEART = new Item(new Item.Settings());
+       public static final Item END_HEART = Registry.register(Registries.ITEM, new Identifier("awesome:end_heart"), new Item(new Item.Settings()));
 
        @Override
        public void onInitialize() {
-               Registry.register(Registries.ITEM, new Identifier("awesome:end_heart"), END_HEART);
        }
 }
```

## Add a Texture

An item have a texture, we need to add a texture for the item.

### Model Configuration File

Location: `assets/awesome/models/item/end_heart.json`

```json
{
  "parent": "item/generated",
  "textures": {
    "layer0": "awesome:item/end_heart"
  }
}
```

### Texture File

Location: `assets/awesome/textures/item/end_heart.png`

![end_heart](https://s2.loli.net/2024/05/03/jpu1mlzKQxbZiCD.png)

![20240503172936](https://s2.loli.net/2024/05/03/9b73MPLnFJZhAQV.png)

## Use Event of the Item

We can create a new class for the item.

```java
package com.example.items;

import net.minecraft.item.Item;

public class EndHeartItem extends Item {
    public EndHeartItem(Settings settings) {
        super(settings);
    }
}
```

Override the `use` method, when the item is used by player, it will be triggered, so we can write some code, such as play a sound.

```patch
 package com.example.items;

+import net.minecraft.entity.player.PlayerEntity;
 import net.minecraft.item.Item;
+import net.minecraft.item.ItemStack;
+import net.minecraft.sound.SoundEvents;
+import net.minecraft.util.Hand;
+import net.minecraft.util.TypedActionResult;
+import net.minecraft.world.World;

 /**
  * @author Enaium
@@ -9,4 +15,10 @@ public class EndHeartItem extends Item {
     public EndHeartItem(Settings settings) {
         super(settings);
     }
+
+    @Override
+    public TypedActionResult<ItemStack> use(World world, PlayerEntity user, Hand hand) {
+        user.playSound(SoundEvents.BLOCK_END_PORTAL_FRAME_FILL, 1.0F, 1.0F);
+        return super.use(world, user, hand);
+    }
 }
```

In this code, we override the `use` method, and play a sound when the item is used,the first parameter is the sound type, the second parameter is the volume, and the third parameter is the pitch.

Finally, the method returns a `TypedActionResult` object, which is the result of the item being used.

Next, we need to replace the `new Item` with `new EndHeartItem` in the `ExampleMod` class.

```patch
-       public static final Item END_HEART = Registry.register(Registries.ITEM, new Identifier("awesome:end_heart"), new Item(new Item.Settings()));
+       public static final Item END_HEART = Registry.register(Registries.ITEM, new Identifier("awesome:end_heart"), new EndHeartItem(new Item.Settings()));

        @Override
        public void onInitialize() {
```

## Add an Item Group

### Direct Add

Actually we can add an item to a existing group, such as `ItemGroup.BUILDING_BLOCKS`, the item will be added to the group in this way.

```patch
+import net.fabricmc.fabric.api.itemgroup.v1.ItemGroupEvents;
 import net.minecraft.item.Item;
+import net.minecraft.item.ItemGroups;
 import net.minecraft.registry.Registries;
 import net.minecraft.registry.Registry;
 import net.minecraft.util.Identifier;
@@ -20,5 +22,8 @@ public class ExampleMod implements ModInitializer {

        @Override
        public void onInitialize() {
+               ItemGroupEvents.modifyEntriesEvent(ItemGroups.BUILDING_BLOCKS).register(content -> {
+                       content.add(END_HEART);
+               });
        }
 }
```

![20240503175311](https://s2.loli.net/2024/05/03/sz4NRxuOtimp139.png)

### Create an Item Group

Like register an item, we need to declare an item group in the `ExampleMod` class.

First, use the `FabricItemGroup.builder()` method to create a builder, and then use the `icon` method to set the icon of the item group, and then use the `display` method to set the display name of the item group, and then use the `entries` method to set the items in the item group.

```patch
 import com.example.items.EndHeartItem;
 import net.fabricmc.api.ModInitializer;
 
+import net.fabricmc.fabric.api.itemgroup.v1.FabricItemGroup;
+import net.fabricmc.fabric.api.itemgroup.v1.ItemGroupEvents;
 import net.minecraft.item.Item;
+import net.minecraft.item.ItemGroup;
+import net.minecraft.item.ItemGroups;
+import net.minecraft.item.ItemStack;
 import net.minecraft.registry.Registries;
 import net.minecraft.registry.Registry;
+import net.minecraft.text.Text;
 import net.minecraft.util.Identifier;
 import org.slf4j.Logger;
 import org.slf4j.LoggerFactory;
@@ -17,6 +23,11 @@ public class ExampleMod implements ModInitializer {
     public static final Logger LOGGER = LoggerFactory.getLogger("awesome");

        public static final Item END_HEART = Registry.register(Registries.ITEM, new Identifier("awesome:end_heart"), new EndHeartItem(new Item.Settings()));
+       public static final ItemGroup ITEM_GROUP = Registry.register(Registries.ITEM_GROUP, new Identifier("awesome:item_group"), FabricItemGroup.builder()
+                       .icon(() -> new ItemStack(END_HEART))
+                       .displayName(Text.literal("Awesome"))
+                       .entries(((displayContext, entries) -> entries.add(END_HEART)))
+                       .build());

        @Override
        public void onInitialize() {
```

![20240503180317](https://s2.loli.net/2024/05/03/Y9RDEfBAajMHqC3.png)