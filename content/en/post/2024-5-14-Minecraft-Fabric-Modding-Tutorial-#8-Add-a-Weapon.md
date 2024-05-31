---
title: "Minecraft Fabric Modding Tutorial #8 Add a Weapon"
date: 2024-05-15T17:50:03+08:00
---

## Tool Material

First, we need to create a enum class, and implement the `ToolMaterial` interface, so we can use it to create a new weapon.

```java
package com.example.tool;

import net.minecraft.block.Block;
import net.minecraft.item.ToolMaterial;
import net.minecraft.recipe.Ingredient;
import net.minecraft.registry.tag.TagKey;

public enum EndToolMaterial implements ToolMaterial {
    ;

    @Override
    public int getDurability() {
        return 0;
    }

    @Override
    public float getMiningSpeedMultiplier() {
        return 0;
    }

    @Override
    public float getAttackDamage() {
        return 0;
    }

    @Override
    public TagKey<Block> getInverseTag() {
        return null;
    }

    @Override
    public int getEnchantability() {
        return 0;
    }

    @Override
    public Ingredient getRepairIngredient() {
        return null;
    }
}
```

Next, we must complete the enum class `EndToolMaterial` by imitating the enum class`net.minecraft.item.ToolMaterials`.

```patch
package com.example.tool;

+import com.example.ExampleMod;
+import com.google.common.base.Suppliers;
 import net.minecraft.block.Block;
 import net.minecraft.item.ToolMaterial;
 import net.minecraft.recipe.Ingredient;
+import net.minecraft.registry.tag.BlockTags;
 import net.minecraft.registry.tag.TagKey;

+import java.util.function.Supplier;
+
 public enum EndToolMaterial implements ToolMaterial {
-    ;
+    END_HEART(BlockTags.INCORRECT_FOR_NETHERITE_TOOL, 2592, 17.0f, 4.0f, 15, () -> Ingredient.ofItems(ExampleMod.END_HEART));
+
+    private final TagKey<Block> inverseTag;
+    private final int itemDurability;
+    private final float miningSpeed;
+    private final float attackDamage;
+    private final int enchantability;
+    private final Supplier<Ingredient> repairIngredient;
+
+    private EndToolMaterial(TagKey<Block> inverseTag, int itemDurability, float miningSpeed, float attackDamage, int enchantability, Supplier<Ingredient> repairIngredient) {
+        this.inverseTag = inverseTag;
+        this.itemDurability = itemDurability;
+        this.miningSpeed = miningSpeed;
+        this.attackDamage = attackDamage;
+        this.enchantability = enchantability;
+        this.repairIngredient = Suppliers.memoize(repairIngredient::get);
+    }

     @Override
     public int getDurability() {
-        return 0;
+        return itemDurability;
     }

     @Override
     public float getMiningSpeedMultiplier() {
-        return 0;
+        return miningSpeed;
     }

     @Override
     public float getAttackDamage() {
-        return 0;
+        return attackDamage;
     }

     @Override
     public TagKey<Block> getInverseTag() {
-        return null;
+        return inverseTag;
     }

     @Override
     public int getEnchantability() {
-        return 0;
+        return enchantability;
     }

     @Override
     public Ingredient getRepairIngredient() {
-        return null;
+        return repairIngredient.get();
     }
 }
```

## Register the Weapone

It's different from the registration of the armor, we need to pass the damage value and the attack speed value to the `SwordItem`.

```java
public static final Item END_SWORD = Registry.register(Registries.ITEM, new Identifier("awesome:end_sword"), new SwordItem(EndToolMaterial.END_HEART, new Item.Settings().attributeModifiers(SwordItem.createAttributeModifiers(EndToolMaterial.END_HEART, 4, 0))));
```

## Texture

![end_sword](https://s2.loli.net/2024/05/15/cbCt6N29sfZniIX.png)

```json
{
  "parent": "item/handheld",
  "textures": {
    "layer0": "awesome:item/end_sword"
  }
}
```

## Recipe

```json
{
  "type": "minecraft:crafting_shaped",
  "pattern": [
    "E",
    "E",
    "S"
  ],
  "key": {
    "E": {
      "item": "awesome:end_heart"
    },
    "S": {
      "item": "minecraft:stick"
    }
  },
  "result": {
    "item": "awesome:end_sword",
    "count": 1
  }
}
```

![20240515185100](https://s2.loli.net/2024/05/15/RzL4jvui5xSDB2H.png)