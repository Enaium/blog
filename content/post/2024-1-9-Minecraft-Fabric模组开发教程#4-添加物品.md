---
title: "Minecraft Fabric模组开发教程#4 添加物品"
date: 2024-01-09T22:38:21+08:00
---

## 注册物品

首先在`ExampleMod`类中声明`Item`对象。

接着在`onInitialize`方法中注册物品。


```diff
 import net.fabricmc.api.ModInitializer;

+import net.fabricmc.fabric.api.item.v1.FabricItemSettings;
+import net.minecraft.item.Item;
+import net.minecraft.registry.Registries;
+import net.minecraft.registry.Registry;
+import net.minecraft.util.Identifier;
 import org.slf4j.Logger;
 import org.slf4j.LoggerFactory;

 public class ExampleMod implements ModInitializer {
     public static final Logger LOGGER = LoggerFactory.getLogger("awesome");

+       public static final Item END_HEART = new Item(new FabricItemSettings());
+
        @Override
        public void onInitialize() {
-               LOGGER.info("Hello Fabric world!");
+               Registry.register(Registries.ITEM, new Identifier("awesome", "end_heart"), END_HEART);
        }
 }
```

其中`Registry.register`方法的第一个参数是注册的类型，第二个参数是注册的ID(格式为`modid:itemid`)，第三个参数是注册的对象。

进入游戏后使用命令将这个物品添加到背包中

```mcfunction
give Player590 awesome:end_heart
```

![4-1](/assets/fabric2024/4-1.png)

我们可以直接调用`Registry.register`方法来注册物品。

```diff
 public class ExampleMod implements ModInitializer {
     public static final Logger LOGGER = LoggerFactory.getLogger("awesome");

-       public static final Item END_HEART = new Item(new FabricItemSettings());
+       public static final Item END_HEART =
+                       Registry.register(Registries.ITEM, new Identifier("awesome", "end_heart"),
+                                       new Item(new FabricItemSettings()));

        @Override
        public void onInitialize() {
-               Registry.register(Registries.ITEM, new Identifier("awesome", "end_heart"), END_HEART);
        }
 }
```

## 添加纹理

注册物品的时候，必须有一个模型配置文件和一个纹理图片。

### 模型配置文件

位置:`assets/awesome/models/item/end_heart.json`

```json
{
  "parent": "item/generated",
  "textures": {
    "layer0": "awesome:item/end_heart"
  }
}
```

### 纹理图片

位置:`assets/awesome/textures/item/end_heart.png`

![end_heart](/assets/fabric/end_heart.png)

![4-1](/assets/fabric2024/4-2.png)

## 使用物品事件

创建一个`com.example.items`包，然后在包中创建一个`EndHeartItem`类。

```java
package com.example.items;

import net.minecraft.item.Item;

public class EndHeartItem extends Item {
    public EndHeartItem(Settings settings) {
        super(settings);
    }
}
```

重写`use`方法，当玩家使用物品时，会触发这个方法，我们可以在这里添加一些逻辑，比如播放一段音效。

```diff
package com.example.items;

+import net.minecraft.entity.player.PlayerEntity;
 import net.minecraft.item.Item;
+import net.minecraft.item.ItemStack;
+import net.minecraft.sound.SoundEvent;
+import net.minecraft.sound.SoundEvents;
+import net.minecraft.util.Hand;
+import net.minecraft.util.TypedActionResult;
+import net.minecraft.world.World;

 public class EndHeartItem extends Item {
     public EndHeartItem(Settings settings) {
         super(settings);
     }

-
+    @Override
+    public TypedActionResult<ItemStack> use(World world, PlayerEntity user, Hand hand) {
+        user.playSound(SoundEvents.BLOCK_WOOL_BREAK, 1.0F, 1.0F);
+        return TypedActionResult.success(user.getStackInHand(hand));
+    }
 }
```

PlayerEntity对象的`playSound`方法可以播放音效，第一个参数是音效对象，第二个参数是音量，第三个参数是音调。

最后返回`TypedActionResult.success`表示使用成功，这里需要传入一个`ItemStack`对象，我们可以通过`PlayerEntity`对象的`getStackInHand`方法来获取玩家手中的物品。

之后我们将`new Item`替换为`new EndHeartItem`

```diff
 public class ExampleMod implements ModInitializer {
     public static final Logger LOGGER = LoggerFactory.getLogger("awesome");

-       public static final Item END_HEART =
-                       Registry.register(Registries.ITEM, new Identifier("awesome", "end_heart"),
-                                       new Item(new FabricItemSettings()));
+    public static final Item END_HEART =
+            Registry.register(Registries.ITEM, new Identifier("awesome", "end_heart"),
+                    new EndHeartItem(new FabricItemSettings()));
```