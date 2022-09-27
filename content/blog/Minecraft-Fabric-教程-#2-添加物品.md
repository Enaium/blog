---
title: "Minecraft Fabric 教程 #2 添加物品"
date: 2019-12-15T19:06:00+0800
categories: fabric
---

## 创建物品


```java
public class ExampleMod implements ModInitializer
{
    private static final Item END_HEART = new Item(new Item.Settings().group(ItemGroup.COMBAT).maxCount(32));
    [...]
}
```

ItemGroup.COMBAT //分类为COMBAT

maxCount(32) //一组最大堆叠数 一组最大只能叠32个物品

## 注册物品
```java
    public class ExampleMod implements ModInitializer
    {
        private static final Item END_HEART = new Item(new Item.Settings().group(ItemGroup.COMBAT).maxCount(32));
     
        @Override
        public void onInitialize()
        {
            Registry.register(Registry.ITEM, new Identifier("endarmor", "end_heart"), END_HEART);
        } 
    }
```


Registry.ITEM //类别是物品

new Identifier("endarmor", "end_heart") //第一个参数是MOD ID 第二个参数是 物品的名字

END_HEART //要注册的物品的变量名


运行看看

发现是一个紫色方块 而且 名字是 item.endarmor.end_heart 紫色方块是没用纹理(材质)

接下来要添加纹理
需要的文件

```java
  Item model: .../resources/assets/endarmor/models/item/end_heart.json
  Item texture: .../resources/assets/endarmor/textures/item/end_heart.png
```

end_heart.json 内容

```java
{
  "parent": "item/generated",
  "textures": {
    "layer0": "endarmor:item/end_heart"
  }
}
```

end_heart.png 就是纹理
![my alternate text](/assets/fabric/end_heart.png)

![2 1](/assets/fabric/2-1.jpg)
## 创建物品类

```java
    public EndHeart(Settings settings) {
        super(settings);
    }
```

这是一个使用物品然后发出声音的例子

```java
public class FabricItem extends Item
{
    public FabricItem(Settings settings)
    {
        super(settings);
    }
 
    @Override
    public TypedActionResult<ItemStack> use(World world, PlayerEntity playerEntity, Hand hand)
    {
        playerEntity.playSound(SoundEvents.BLOCK_WOOL_BREAK, 1.0F, 1.0F);
        return new TypedActionResult<>(ActionResult.SUCCESS, playerEntity.getStackInHand(hand));
    }
}
```

替换

```java
private static final EndHeart END_HEART = new EndHeart(new Item.Settings().group(ItemGroup.COMBAT).maxCount(32));
```
