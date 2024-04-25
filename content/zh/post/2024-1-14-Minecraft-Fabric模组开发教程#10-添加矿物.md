---
layout: post
title: "Minecraft Fabric模组开发教程#10 添加矿物"
date: 2024-01-14T14:44:09+08:00
---

## 注册矿物

我们都知道，在矿物被挖掘之后，会掉落经验，所以我们需要使用`ExperienceDroppingBlock`来创建方块对象，之后需要创建两种矿物，一种是普通的矿物，一种是深层矿物。

`ExperienceDroppingBlock`的构造函数需要两个参数，第一个是掉落的经验数量，这里也就是创建了一个随机为 10-17 的经验掉落，第二个参数是方块的设置，这里设置了方块的颜色，敲击音效，硬度抗性，必须使用工具，以及挖掘音效。

```java
public static final Block END_HEART_ORE = new ExperienceDroppingBlock(UniformIntProvider.create(10, 17), AbstractBlock.Settings.create().mapColor(MapColor.STONE_GRAY).instrument(Instrument.BASEDRUM).requiresTool().strength(3.0f, 3.0f).sounds(BlockSoundGroup.NETHER_ORE));
```

深层矿物的设置和普通矿物差不多，只不过需要调用`copy`方法。

```java
public static final Block DEEPSLATE_END_HEART_ORE = new ExperienceDroppingBlock(UniformIntProvider.create(10, 17), AbstractBlock.Settings.copy(END_HEART_ORE).mapColor(MapColor.DEEPSLATE_GRAY).instrument(Instrument.BASEDRUM).requiresTool().strength(4.5f, 3.0f).sounds(BlockSoundGroup.DEEPSLATE));
```

最后，我们需要注册这两个矿物。

```java
Registry.register(Registries.BLOCK, new Identifier("awesome", "end_heart_ore"), END_HEART_ORE);
Registry.register(Registries.BLOCK, new Identifier("awesome", "deepslate_end_heart_ore"), DEEPSLATE_END_HEART_ORE);
```

## 注册矿物物品

使用之前学习的方法，我们可以很容易的注册矿物物品。

## 矿物纹理

![end_heart_ore](/assets/fabric2024/end_heart_ore.png)
![deepslate_end_heart_ore](/assets/fabric2024/deepslate_end_heart_ore.png)

使用之前学习的方法，我们可以很容易的注册矿物纹理。

## 多语言文件

```json
{
    "item.awesome.end_heart_ore": "End Heart Ore",
    "item.awesome.deepslate_end_heart_ore": "Deepslate End Heart Ore",
    "block.awesome.end_heart_ore": "End Heart Ore",
    "block.awesome.deepslate_end_heart_ore": "Deepslate End Heart Ore"
}
```

## 掉落物

这里会比较复杂，需要考虑`精准采集`和`时运`的影响，但我们可以使用原版中已经存在的配置。

我们进入到原版游戏中的`data/minecraft/loot_tables/blocks/diamond_ore.json`和`data/minecraft/loot_tables/blocks/deepslate_diamond_ore.json`将钻石矿的掉落配置加以修改。

```diff
@@ -24,7 +24,7 @@
                   }
                 }
               ],
-              "name": "minecraft:diamond_ore"
+              "name": "awesome:end_heart_ore"
             },
             {
               "type": "minecraft:item",
@@ -38,7 +38,7 @@
                   "function": "minecraft:explosion_decay"
                 }
               ],
-              "name": "minecraft:diamond"
+              "name": "awesome:end_heart"
             }
           ]
         }
@@ -46,5 +46,5 @@
       "rolls": 1.0
     }
   ],
-  "random_sequence": "minecraft:blocks/diamond_ore"
+  "random_sequence": "awesome:blocks/end_heart_ore"
 }
```

```diff
@@ -24,7 +24,7 @@
                   }
                 }
               ],
-              "name": "minecraft:deepslate_diamond_ore"
+              "name": "awesome:deepslate_end_heart_ore"
             },
             {
               "type": "minecraft:item",
@@ -38,7 +38,7 @@
                   "function": "minecraft:explosion_decay"
                 }
               ],
-              "name": "minecraft:diamond"
+              "name": "awesome:end_heart"
             }
           ]
         }
@@ -46,5 +46,5 @@
       "rolls": 1.0
     }
   ],
-  "random_sequence": "minecraft:blocks/deepslate_diamond_ore"
+  "random_sequence": "awesome:blocks/deepslate_end_heart_ore"
 }
```

## 可挖掘方块

我们需要在`data/minecraft/tags/blocks/mineable/pickaxe.json`中添加我们的矿物。

```json
{
  "replace": false,
  "values": [
    "awesome:end_heart_ore",
    "awesome:deepslate_end_heart_ore"
  ]
}
```

![10-1](/assets/fabric2024/10-1.png)
![10-2](/assets/fabric2024/10-2.png)

## 矿物配方

首先添加熔炉的配方，还是在`recipes`下创建，格式就是`物品ID_from_smelting_矿物ID.json`，高炉的配方就是`物品ID_from_blasting_矿物ID.json`。

### 熔炉

```json
{
  "type": "minecraft:smelting",
  "category": "misc",
  "cookingtime": 200,
  "experience": 1.0,
  "group": "end_heart",
  "ingredient": {
    "item": "awesome:end_heart_ore"
  },
  "result": "awesome:end_heart"
}
```


```json
{
  "type": "minecraft:smelting",
  "category": "misc",
  "cookingtime": 200,
  "experience": 1.0,
  "group": "end_heart",
  "ingredient": {
    "item": "awesome:deepslate_end_heart_ore"
  },
  "result": "awesome:end_heart"
}
```

### 高炉

```json
{
  "type": "minecraft:blasting",
  "category": "misc",
  "cookingtime": 100,
  "experience": 1.0,
  "group": "end_heart",
  "ingredient": {
    "item": "awesome:end_heart_ore"
  },
  "result": "awesome:end_heart"
}
```

```json
{
  "type": "minecraft:blasting",
  "category": "misc",
  "cookingtime": 100,
  "experience": 1.0,
  "group": "end_heart",
  "ingredient": {
    "item": "awesome:deepslate_end_heart_ore"
  },
  "result": "awesome:end_heart"
}
```

![10-3](/assets/fabric2024/10-3.png)
![10-4](/assets/fabric2024/10-4.png)

## 矿物生成

首先创建矿物的`configured_feature`，在`data\awesome\worldgen\configured_feature`中，创建配置`end_heart_ore.json`。

其中`discard_chance_on_air_exposure`就是矿物在空气中曝露的概率，`size`就是矿物的大小，`targets`就是矿物生成的目标方块，`predicate_type`的意思就是判断`tag`中的方块是否符合要求，不过要注意的是`state`中的`Name`必须首字母大写。

```json
{
  "type": "minecraft:ore",
  "config": {
    "discard_chance_on_air_exposure": 0.0,
    "size": 12,
    "targets": [
      {
        "state": {
          "Name": "awesome:end_heart_ore"
        },
        "target": {
          "predicate_type": "minecraft:tag_match",
          "tag": "minecraft:stone_ore_replaceables"
        }
      },
      {
        "state": {
          "Name": "awesome:deepslate_end_heart_ore"
        },
        "target": {
          "predicate_type": "minecraft:tag_match",
          "tag": "minecraft:deepslate_ore_replaceables"
        }
      }
    ]
  }
}
```

然后创建`placed_feature`，在`data\awesome\worldgen\placed_feature`中，创建配置`end_heart_ore.json`。

在`placement`中有四个参数，第一个是`minecraft:count`，就是矿物的数量，第二个是`minecraft:in_square`，就是矿物的生成范围，第三个是`minecraft:height_range`，就是矿物的生成高度，第四个是`minecraft:biome`，表示会在群系中生成。

```json
{
  "feature": "awesome:end_heart_ore",
  "placement": [
    {
      "type": "minecraft:count",
      "count": 20
    },
    {
      "type": "minecraft:in_square"
    },
    {
      "type": "minecraft:height_range",
      "height": {
        "type": "minecraft:trapezoid",
        "max_inclusive": {
          "absolute": 70
        },
        "min_inclusive": {
          "absolute": -24
        }
      }
    },
    {
      "type": "minecraft:biome"
    }
  ]
}
```

最后注册`PlacedFeature`。

```java
public static final RegistryKey<PlacedFeature> END_HEART_ORE_PLACED_FEATURE = RegistryKey.of(RegistryKeys.PLACED_FEATURE, new Identifier("awesome","end_heart_ore"));
```

并且在`BiomeModifications`中注册`END_HEART_ORE_PLACED_FEATURE`。

```java
BiomeModifications.addFeature(BiomeSelectors.foundInOverworld(), GenerationStep.Feature.UNDERGROUND_ORES, END_HEART_ORE_PLACED_FEATURE);
```

![10-5](/assets/fabric2024/10-5.png)
![10-6](/assets/fabric2024/10-6.png)
