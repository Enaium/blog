---
title: "Minecraft Fabric模组开发教程#8 添加武器"
date: 2024-01-10T16:00:24+08:00
---

## 工具材料

和添加盔甲差不多，需要创建一个`EndToolMaterial`类，之后实现`ToolMaterial`接口，这样我们就可以创建武器了。

```java
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
    public int getMiningLevel() {
        return 0;
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

之后仿照`net.minecraft.item.ToolMaterials`枚举类完成这些方法。

```diff
 package com.example.tool;

+import com.example.ExampleMod;
 import net.minecraft.item.ToolMaterial;
 import net.minecraft.recipe.Ingredient;

+import java.util.function.Supplier;
+
 public enum EndToolMaterial implements ToolMaterial {
-    ;
+    END(5, 2592, 17.0f, 4.0f, 15, () -> Ingredient.ofItems(ExampleMod.END_HEART));
+
+    private final int miningLevel;
+    private final int itemDurability;
+    private final float miningSpeed;
+    private final float attackDamage;
+    private final int enchantability;
+    private final Supplier<Ingredient> repairIngredient;
+
+    private EndToolMaterial(int miningLevel, int itemDurability, float miningSpeed, float attackDamage, int enchantability, Supplier<Ingredient> repairIngredient) {
+        this.miningLevel = miningLevel;
+        this.itemDurability = itemDurability;
+        this.miningSpeed = miningSpeed;
+        this.attackDamage = attackDamage;
+        this.enchantability = enchantability;
+        this.repairIngredient = repairIngredient;
+    }
+
     @Override
     public int getDurability() {
-        return 0;
+        return this.itemDurability;
     }

     @Override
     public float getMiningSpeedMultiplier() {
-        return 0;
+        return this.miningSpeed;
     }

     @Override
     public float getAttackDamage() {
-        return 0;
+        return this.attackDamage;
     }

     @Override
     public int getMiningLevel() {
-        return 0;
+        return this.miningLevel;
     }

     @Override
     public int getEnchantability() {
-        return 0;
+        return this.enchantability;
     }

     @Override
     public Ingredient getRepairIngredient() {
-        return null;
+        return this.repairIngredient.get();
     }
 }
```

## 注册武器

和盔甲不同的是，在创建`SwordItem`时，额外需要传入武器的伤害和攻击速度，需要注意的是攻击速度始终会多出4.0f。

```java
    public static final Item END_SWORD = Registry.register(Registries.ITEM, new Identifier("awesome", "end_sword"), new SwordItem(EndToolMaterial.END, 4, 0f, new FabricItemSettings()));
```

## 添加武器纹理

### 武器物品

![end_sword](/assets/fabric2024/end)

```json
{
  "parent": "item/handheld",
  "textures": {
    "layer0": "awesome:item/end_sword"
  }
}
```

### 合成表

```json
{
  "type": "minecraft:crafting_shaped",
  "pattern": [
    " E ",
    " E ",
    " S "
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

### 多语言

```json
{
  "item.awesome.end_sword": "End Sword"
}
```

---

最后将武器添加到物品组中。

```diff
@@ -44,6 +44,7 @@
                 entries.add(END_CHESTPLATE);
                 entries.add(END_LEGGINGS);
                 entries.add(END_BOOTS);
+                entries.add(END_SWORD);
             })
             .build());
```

![8-1](/assets/fabric2024/8-1.png)

![8-2](/assets/fabric2024/8-2.png)