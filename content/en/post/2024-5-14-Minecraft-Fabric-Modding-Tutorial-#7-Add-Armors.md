---
layout: post
title: "2024 5 14 Minecraft Fabric Modding Tutorial #7 Add Armors"
date: 2024-05-14T02:01:12+08:00
---

## Armor Material

First, we need to create a material class, and then we need to declare a new armor material.

```java
public static final RegistryEntry<ArmorMaterial> END_HEART = EndArmorMaterial.register("end_heart", Util.make(new EnumMap<>(ArmorItem.Type.class), map -> {
    map.put(ArmorItem.Type.BOOTS, 4);
    map.put(ArmorItem.Type.LEGGINGS, 6);
    map.put(ArmorItem.Type.CHESTPLATE, 8);
    map.put(ArmorItem.Type.HELMET, 4);
    map.put(ArmorItem.Type.BODY, 11);
}), 20, SoundEvents.ITEM_ARMOR_EQUIP_NETHERITE, 3.0f, 0.1f, () -> Ingredient.ofItems(Items.NETHERITE_INGOT));

private static RegistryEntry<ArmorMaterial> register(String id, EnumMap<ArmorItem.Type, Integer> defense, int enchantability, RegistryEntry<SoundEvent> equipSound, float toughness, float knockbackResistance, Supplier<Ingredient> repairIngredient) {
    List<ArmorMaterial.Layer> list = List.of(new ArmorMaterial.Layer(new Identifier(id)));
    return register(id, defense, enchantability, equipSound, toughness, knockbackResistance, repairIngredient, list);
}

private static RegistryEntry<ArmorMaterial> register(String id, EnumMap<ArmorItem.Type, Integer> defense, int enchantability, RegistryEntry<SoundEvent> equipSound, float toughness, float knockbackResistance, Supplier<Ingredient> repairIngredient, List<ArmorMaterial.Layer> layers) {
    EnumMap<ArmorItem.Type, Integer> enumMap = new EnumMap<>(ArmorItem.Type.class);
    for (ArmorItem.Type type : ArmorItem.Type.values()) {
        enumMap.put(type, defense.get(type));
    }
    return Registry.registerReference(Registries.ARMOR_MATERIAL, new Identifier(id), new ArmorMaterial(enumMap, enchantability, equipSound, repairIngredient, layers, toughness, knockbackResistance));
}
```

You can copy the code from the class `net.minecraft.item.ArmorMaterials` and modify it to your needs.

## Register Armor Items

```java
public static final Item END_HELMET = Registry.register(Registries.ITEM, new Identifier("awesome:end_helmet"), new ArmorItem(EndArmorMaterial.END_HEART, ArmorItem.Type.HELMET, new Item.Settings()));
public static final Item END_CHESTPLATE = Registry.register(Registries.ITEM, new Identifier("awesome:end_chestplate"), new ArmorItem(EndArmorMaterial.END_HEART, ArmorItem.Type.CHESTPLATE, new Item.Settings()));
public static final Item END_LEGGINGS = Registry.register(Registries.ITEM, new Identifier("awesome:end_leggings"), new ArmorItem(EndArmorMaterial.END_HEART, ArmorItem.Type.LEGGINGS, new Item.Settings()));
public static final Item END_BOOTS = Registry.register(Registries.ITEM, new Identifier("awesome:end_boots"), new ArmorItem(EndArmorMaterial.END_HEART, ArmorItem.Type.BOOTS, new Item.Settings()));
```

## Add Armor Texture

### Armor Items

![end_helmet](https://s2.loli.net/2024/05/14/7NMCDylpYAHumtI.png)
![end_chestplate](https://s2.loli.net/2024/05/14/wOh1EIzRrxFmiMU.png)
![end_leggings](https://s2.loli.net/2024/05/14/bJrXeNzZudSp8Tq.png)
![end_boots](https://s2.loli.net/2024/05/14/YC5ubKs9EagPxw1.png)

```java
{
  "parent": "item/generated",
  "textures": {
    "layer0": "awesome:item/end_helmet"
  }
}
```

```java
{
  "parent": "item/generated",
  "textures": {
    "layer0": "awesome:item/end_chestplate"
  }
}
```

```java
{
  "parent": "item/generated",
  "textures": {
    "layer0": "awesome:item/end_leggings"
  }
}
```

```java
{
  "parent": "item/generated",
  "textures": {
    "layer0": "awesome:item/end_boots"
  }
}
```

### Arrmor Models

Location: `assets\minecraft\textures\models\armor`

![end_heart_layer_1](https://s2.loli.net/2024/05/14/L9hBOUoc1iPI4n5.png)

![end_heart_layer_2](https://s2.loli.net/2024/05/14/7tmldJ4g5MwxCuy.png)

## Lanauges

```java
{
  "item.awesome.end_helmet": "End Helmet",
  "item.awesome.end_chestplate": "End Chestplate",
  "item.awesome.end_leggings": "End Leggings",
  "item.awesome.end_boots": "End Boots"
}
```

## Recipes

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
    "id": "awesome:end_helmet",
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
    "id": "awesome:end_chestplate",
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
      "item": "awesome:end_heart"
    }
  },
  "result": {
    "id": "awesome:end_leggings",
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
    "id": "awesome:end_boots",
    "count": 1
  }
}
```

Finally, add thses arrmor items to the item group.

```patch
             .entries(((displayContext, entries) -> {
                 entries.add(END_HEART);
                 entries.add(END_HEART_BLOCK);
+                entries.add(END_HELMET);
+                entries.add(END_CHESTPLATE);
+                entries.add(END_LEGGINGS);
+                entries.add(END_BOOTS);
             }))
             .build());

```

![20240514032826](https://s2.loli.net/2024/05/14/fyJ5zqN39HGEsak.png)

![20240514032846](https://s2.loli.net/2024/05/14/Im6RYCoiFSJql2k.png)

![20240514032853](https://s2.loli.net/2024/05/14/3Y1HsAW6rczmXvl.png)

![20240514032900](https://s2.loli.net/2024/05/14/vnmky1MxrofXPAJ.png)

![20240514032914](https://s2.loli.net/2024/05/14/fXBaJUQhFVLDOCp.png)