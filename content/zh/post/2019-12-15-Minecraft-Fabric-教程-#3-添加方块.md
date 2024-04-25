---
layout: post
title: "Minecraft Fabric 教程 #3 添加方块"
date: 2019-12-15T19:36:00+08:00
categroy: fabric
---

## 创建方块

```java
public class ExampleMod implements ModInitializer
{
    // an instance of our new block
    public static final Block END_HEART_BLOCK = new Block(FabricBlockSettings.of(Material.METAL).build());
    [...]
}
```

## 注册

```java
public class ExampleMod implements ModInitializer
{
    // block creation
    […]
 
    @Override
    public void onInitialize()
    {
        Registry.register(Registry.BLOCK, new Identifier("endarmor", "end_heart_block"), END_HEART_BLOCK);
    }
}
```

运行游戏发现无法找到方块是因为没有创建方块物品 但是可以使用命令在创建这个方块


## 创建方块物品

直接注册就行

```java
public class ExampleMod implements ModInitializer
{
    // block creation
    […]
 
    @Override
    public void onInitialize()
    {
        // block registration
        [...]
 
        Registry.register(Registry.ITEM, new Identifier("endarmor", "end_heart_block"), new BlockItem(END_HEART_BLOCK, new Item.Settings().itemGroup(ItemGroup.MISC)));
    }
}
```


## 添加纹理

```
Blockstate: src/main/resources/assets/endarmor/blockstates/end_heart_block.json
Block Model: src/main/resources/assets/endarmor/models/block/end_heart_block.json
Item Model: src/main/resources/assets/endarmor/models/item/end_heart_block.json
Block Texture: src/main/resources/assets/endarmor/textures/block/end_heart_block.png
```


blockstates/end_heart_block.json

```java
{
  "variants": {
    "": { "model": "endarmor:block/end_heart_block" }
  }
}
```

models/block/end_heart_block.json

```java
{
  "parent": "block/cube_all",
  "textures": {
    "all": "endarmor:block/end_heart_block"
  }
}
```

models/item/end_heart_block.json

```java
{
  "parent": "endarmor:block/end_heart_block"
}
```

textures/block/end_heart_block.png

![my alternate text](/assets/fabric/end_heart_block.png)

![3 1](/assets/fabric/3-1.jpg)
## 掉落物

位置 `\src\main\resources\data\endarmor\loot_tables\blocks\end_heart_block.json`

```java
{
  "type": "minecraft:block",
  "pools": [
    {
      "rolls": 1,
      "entries": [
        {
          "type": "minecraft:item",
          "name": "endarmor:end_heart_block"
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


## 创建一个Block类

```java
public class ExampleBlock extends Block
{
    public ExampleBlock(Settings settings)
    {
        super(settings);
    }
}
```

```java
	private static final EndHeartBlock END_HEART_BLOCK = new EndHeartBlock(FabricBlockSettings.of(Material.METAL).build());
```
