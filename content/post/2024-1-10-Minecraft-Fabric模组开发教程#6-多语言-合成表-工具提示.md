---
title: "Minecraft Fabric模组开发教程#6 多语言 合成表 提示"
date: 2024-01-10T02:49:55+08:00
---

## 多语言

在之前我们已经添加了物品、物品组和方快，但是它们还没有名称，这里可以使用多语言来为它们添加名称。

我们在`assets/awesome/lang`目录下创建一个`en_us.json`文件，这个文件就是英文语言文件，我们可以在这个文件中添加多语言。

```json
{
  "item.awesome.end_heart": "End Heart",
  "item.awesome.end_heart_block": "End Heart Block",
  "block.awesome.end_heart_block": "End Heart Block",
  "itemGroup.awesome.item_group": "Awesome"
}
```

![6-1](/assets/fabric2024/6-1.png)

## 合成表

合成表是用来合成物品的，我们可以在这里设置合成物品的配方。

在`data\awesome\recipes`创建一个`end_heart_block.json`文件，也就是如何在工作台中合成`end_heart_block.json`。

其中`type`是合成表的类型，`pattern`是合成表的合成模式，也就是在工作台中的物品摆放位置，`key`是合成表的材料，`result`是合成表的结果。

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
    "item": "awesome:end_heart_block",
    "count": 1
  }
}
```

接着我们需要再添加一个分解的合成表，也就是如何将`end_heart_block`分解成`end_heart`。

在`data\awesome\recipes`创建一个`end_heart.json`文件。

和之前的合成表不同的是，这个合成表的`type`是`minecraft:crafting_shapeless`，也就是无序合成表，`ingredients`是合成表的材料，`result`是合成表的结果。

```json
{
  "type": "minecraft:crafting_shapeless",
  "ingredients": [
    {
      "item": "awesome:end_heart_block"
    }
  ],
  "result": {
    "item": "awesome:end_heart",
    "count": 9
  }
}
```

![6-2](/assets/fabric2024/6-2.png)

![6-3](/assets/fabric2024/6-3.png)

## 提示

我们可以在方块上添加提示，当玩家将鼠标悬停在方块上时，就会显示提示。

在`EndHeartBlock`中添加`appendTooltip`方法，之后调用`tooltip.add`添加提示，使用`Text.translatable`方法可以将多语言的key转换为对应的多语言。

```java
public class EndHeartBlock extends Block {
    @Override
    public void appendTooltip(ItemStack stack, @Nullable BlockView world, List<Text> tooltip, TooltipContext options) {
        tooltip.add(Text.translatable("tooltip.awesome.end_heart_block"));
    }
}
```

```java
public class EndHeartItem extends Item {
    @Override
    public void appendTooltip(ItemStack stack, @Nullable World world, List<Text> tooltip, TooltipContext context) {
        tooltip.add(Text.translatable("tooltip.awesome.end_heart"));
    }
}
```

```json
{
  "tooltip.awesome.end_heart": "End Heart",
  "tooltip.awesome.end_heart_block": "End Heart Block"
}
```

![6-4](/assets/fabric2024/6-4.png)

![6-5](/assets/fabric2024/6-5.png)