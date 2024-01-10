---
title: "Minecraft Fabric模组开发教程#7 添加盔甲"
date: 2024-01-10T14:53:56+08:00
---

## 盔甲材料

我们创建一个`EndArmorMaterial`枚举类，之后实现`ArmorMaterial`接口，这样我们就可以创建盔甲了。

```java
package com.example.armor;

import net.minecraft.item.ArmorItem;
import net.minecraft.item.ArmorMaterial;
import net.minecraft.recipe.Ingredient;
import net.minecraft.sound.SoundEvent;

public enum EndArmorMaterial implements ArmorMaterial {
    ;

    @Override
    public int getDurability(ArmorItem.Type type) {
        return 0;
    }

    @Override
    public int getProtection(ArmorItem.Type type) {
        return 0;
    }

    @Override
    public int getEnchantability() {
        return 0;
    }

    @Override
    public SoundEvent getEquipSound() {
        return null;
    }

    @Override
    public Ingredient getRepairIngredient() {
        return null;
    }

    @Override
    public String getName() {
        return null;
    }

    @Override
    public float getToughness() {
        return 0;
    }

    @Override
    public float getKnockbackResistance() {
        return 0;
    }
}
```

之后仿照`net.minecraft.item.ArmorMaterials`枚举类完成这些方法。

```diff
 import net.minecraft.item.ArmorItem;
 import net.minecraft.item.ArmorMaterial;
+import net.minecraft.item.Items;
 import net.minecraft.recipe.Ingredient;
 import net.minecraft.sound.SoundEvent;
+import net.minecraft.sound.SoundEvents;
+import net.minecraft.util.Util;
+
+import java.util.EnumMap;
+import java.util.function.Supplier;

 public enum EndArmorMaterial implements ArmorMaterial {
-    ;
+    END("end_heart", 5, Util.make(new EnumMap<>(ArmorItem.Type.class), (map) -> {
+        map.put(ArmorItem.Type.BOOTS, 1);
+        map.put(ArmorItem.Type.LEGGINGS, 2);
+        map.put(ArmorItem.Type.CHESTPLATE, 3);
+        map.put(ArmorItem.Type.HELMET, 1);
+    }), 15, SoundEvents.ITEM_ARMOR_EQUIP_LEATHER, 0.0F, 0.0F, () -> Ingredient.ofItems(Items.LEATHER));
+
+    private static final EnumMap<ArmorItem.Type, Integer> BASE_DURABILITY = Util.make(new EnumMap<>(ArmorItem.Type.class), (map) -> {
+        map.put(ArmorItem.Type.BOOTS, 13);
+        map.put(ArmorItem.Type.LEGGINGS, 15);
+        map.put(ArmorItem.Type.CHESTPLATE, 16);
+        map.put(ArmorItem.Type.HELMET, 11);
+    });
+
+    private final String name;
+    private final int durabilityMultiplier;
+    private final EnumMap<ArmorItem.Type, Integer> protectionAmounts;
+    private final int enchantability;
+    private final SoundEvent equipSound;
+    private final float toughness;
+    private final float knockbackResistance;
+    private final Supplier<Ingredient> repairIngredientSupplier;
+
+    EndArmorMaterial(String name, int durabilityMultiplier, EnumMap<ArmorItem.Type, Integer> protectionAmounts, int enchantability, SoundEvent equipSound, float toughness, float knockbackResistance, Supplier<Ingredient> repairIngredientSupplier) {
+        this.name = name;
+        this.durabilityMultiplier = durabilityMultiplier;
+        this.protectionAmounts = protectionAmounts;
+        this.enchantability = enchantability;
+        this.equipSound = equipSound;
+        this.toughness = toughness;
+        this.knockbackResistance = knockbackResistance;
+        this.repairIngredientSupplier = repairIngredientSupplier;
+    }

     @Override
     public int getDurability(ArmorItem.Type type) {
-        return 0;
+        return BASE_DURABILITY.get(type) * this.durabilityMultiplier;
     }

     @Override
     public int getProtection(ArmorItem.Type type) {
-        return 0;
+        return this.protectionAmounts.get(type);
     }

     @Override
     public int getEnchantability() {
-        return 0;
+        return this.enchantability;
     }

     @Override
     public SoundEvent getEquipSound() {
-        return null;
+        return this.equipSound;
     }

     @Override
     public Ingredient getRepairIngredient() {
-        return null;
+        return this.repairIngredientSupplier.get();
     }

     @Override
     public String getName() {
-        return null;
+        return this.name;
     }

     @Override
     public float getToughness() {
-        return 0;
+        return this.toughness;
     }

     @Override
     public float getKnockbackResistance() {
-        return 0;
+        return this.knockbackResistance;
     }
 }
```

## 注册盔甲物品

```java
public static final Item END_HELMET = Registry.register(Registries.ITEM, newIdentifier("awesome", "end_helmet"), new ArmorItem(EndArmorMaterial.END,ArmorItem.Type.HELMET, new FabricItemSettings()));
public static final Item END_CHESTPLATE = Registry.register(Registries.ITEM,new Identifier("awesome", "end_chestplate"), new ArmorItem(EndArmorMaterialEND, ArmorItem.Type.CHESTPLATE, new FabricItemSettings()));
public static final Item END_LEGGINGS = Registry.register(Registries.ITEM,new Identifier("awesome", "end_leggings"), new ArmorItem(EndArmorMaterialEND, ArmorItem.Type.LEGGINGS, new FabricItemSettings()));
public static final Item END_BOOTS = Registry.register(Registries.ITEM, new Identifier("awesome", "end_boots"), new ArmorItem(EndArmorMaterial.END, ArmorItem.Type.BOOTS, new FabricItemSettings()));
```

## 添加盔甲纹理

### 盔甲物品

![helmet](/assets/fabric/end_helmet.png)
![chestplate](/assets/fabric/end_chestplate.png)
![leggings](/assets/fabric/end_leggings.png)
![boots](/assets/fabric/end_boots.png)

```json
{
  "parent": "item/generated",
  "textures": {
    "layer0": "awesome:item/end_helmet"
  }
}
```

```json
{
  "parent": "item/generated",
  "textures": {
    "layer0": "awesome:item/end_chestplate"
  }
}
```

```json
{
  "parent": "item/generated",
  "textures": {
    "layer0": "awesome:item/end_leggings"
  }
}
```

```json
{
  "parent": "item/generated",
  "textures": {
    "layer0": "awesome:item/end_boots"
  }
}
```


### 盔甲模型

位置`assets\minecraft\textures\models\armor`

![layer 1](/assets/fabric/end_heart_layer_1.png)
![layer 2](/assets/fabric/end_heart_layer_2.png)

### 多语言

```json
{
  "item.awesome.end_helmet": "End Helmet",
  "item.awesome.end_chestplate": "End Chestplate",
  "item.awesome.end_leggings": "End Leggings",
  "item.awesome.end_boots": "End Boots"
}
```

### 合成表

```json
{
  "type": "minecraft:crafting_shaped",
  "pattern": [
    "EEE",
    "E E"
  ],
  "key": {
    "E": {
      "item": "awesome:end_heart"
    }
  },
  "result": {
    "item": "awesome:end_helmet",
    "count": 1
  }
}
```

```json
{
  "type": "minecraft:crafting_shaped",
  "pattern": [
    "E E",
    "EEE",
    "EEE"
  ],
  "key": {
    "E": {
      "item": "awesome:end_heart"
    }
  },
  "result": {
    "item": "awesome:end_chestplate",
    "count": 1
  }
}
```

```json
{
  "type": "minecraft:crafting_shaped",
  "pattern": [
    "EEE",
    "E E",
    "E E"
  ],
  "key": {
    "E": {
      "item": "awesome:end_heart",
    }
  },
  "result": {
    "item": "awesome:end_leggings",
    "count": 1
  }
}
```

```json
{
  "type": "minecraft:crafting_shaped",
  "pattern": [
    "E E",
    "E E"
  ],
  "key": {
    "E": {
      "item": "awesome:end_heart"
    }
  },
  "result": {
    "item": "awesome:end_boots",
    "count": 1
  }
}
```

---

最后将盔甲物品添加到物品组里面。

```diff
@@ -38,6 +38,10 @@
             .entries((context, entries) -> {
                 entries.add(END_HEART);
                 entries.add(END_HEART_BLOCK);
+                entries.add(END_HELMET);
+                entries.add(END_CHESTPLATE);
+                entries.add(END_LEGGINGS);
+                entries.add(END_BOOTS);
             })
             .build());
```

![7-1](/assets/fabric2024/7-1.png)