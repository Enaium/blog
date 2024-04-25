---
layout: post
title: "Minecraft Fabric 教程 #9 添加盔甲"
date: 2019-12-16T17:36:00+08:00
categroy: fabric
---

创建一个盔甲类

```java
public class EndArmorMaterials implements ArmorMaterial {
    [...]
}
```

复制一下内容 

```java
    private static final int[] BASE_DURABILITY = {13, 15, 16, 11};
    private final String name;
    private final int durabilityMultiplier;
    private final int[] protectionAmounts;
    private final int enchantability;
    private final SoundEvent equipSound;
    private final float toughness;
    private final Lazy<Ingredient> repairIngredientSupplier;

    public EndArmorMaterials(String name, int durabilityMultiplier, int[] armorValueArr, int enchantability, SoundEvent soundEvent, float toughness, Supplier<Ingredient> repairIngredient) {
        this.name = name;
        this.durabilityMultiplier = durabilityMultiplier;
        this.protectionAmounts = armorValueArr;
        this.enchantability = enchantability;
        this.equipSound = soundEvent;
        this.toughness = toughness;
        this.repairIngredientSupplier = new Lazy(repairIngredient);
    }

    public int getDurability(EquipmentSlot equipmentSlot_1) {
        return BASE_DURABILITY[equipmentSlot_1.getEntitySlotId()] * this.durabilityMultiplier;
    }

    public int getProtectionAmount(EquipmentSlot equipmentSlot_1) {
        return this.protectionAmounts[equipmentSlot_1.getEntitySlotId()];
    }

    public int getEnchantability() {
        return this.enchantability;
    }

    public SoundEvent getEquipSound() {
        return this.equipSound;
    }

    public Ingredient getRepairIngredient() {
        return this.repairIngredientSupplier.get();
    }

    @Environment(EnvType.CLIENT)
    public String getName() {
        return this.name;
    }

    public float getToughness() {
        return this.toughness;
    }
```

然后把class 改成 enum 

制作盔甲材料
```java
public enum EndArmorMaterials implements ArmorMaterial {



    END("end_heart" , 15 , new int[]{1,3,2,1}, 15, SoundEvents.BLOCK_WOOL_PLACE,0.0F, () -> {
        return Ingredient.ofItems(Items.WHITE_WOOL);
    });

    [...]
}
```
参数一 材料名字 参数二 耐久倍数 参数三 盔甲数也就是穿上盔甲加的盔甲值 参数四 使用的时候发出的声音 参数五 耐性

## 创建盔甲物品
```java
	public static final Item END_HELMET = new ArmorItem(EndArmorMaterials.END, EquipmentSlot.HEAD, (new Item.Settings().group(ItemGroup.COMBAT)));
	public static final Item END_CHESTPLATE = new ArmorItem(EndArmorMaterials.END, EquipmentSlot.CHEST, (new Item.Settings().group(ItemGroup.COMBAT)));
	public static final Item END_LEGGINGS = new ArmorItem(EndArmorMaterials.END, EquipmentSlot.LEGS, (new Item.Settings().group(ItemGroup.COMBAT)));
	public static final Item END_BOOTS = new ArmorItem(EndArmorMaterials.END, EquipmentSlot.FEET, (new Item.Settings().group(ItemGroup.COMBAT)));
```
## 注册盔甲物品
```java
	Registry.register(Registry.ITEM,new Identifier("endarmor","end_helmet"), END_HELMET);
	Registry.register(Registry.ITEM,new Identifier("endarmor","end_chestplate"), END_CHESTPLATE);
	Registry.register(Registry.ITEM,new Identifier("endarmor","end_leggings"), END_LEGGINGS);
	Registry.register(Registry.ITEM,new Identifier("endarmor","end_boots"), END_BOOTS);
```
## 添加纹理

先添加物品纹理

![helmet](/assets/fabric/end_helmet.png)
![chestplate](/assets/fabric/end_chestplate.png)
![leggings](/assets/fabric/end_leggings.png)
![boots](/assets/fabric/end_boots.png)
    
发现只有物品纹理穿上后没有模型纹理然后添加模型

位置 src\main\resources\assets\minecraft\textures\models\armor


一共有两层end_heart_layer_1.png 和 end_heart_layer_2.png


![layer 1](/assets/fabric/end_heart_layer_1.png)
![layer 2](/assets/fabric/end_heart_layer_2.png)

最终效果

![9 1](/assets/fabric/9-1.jpg)




