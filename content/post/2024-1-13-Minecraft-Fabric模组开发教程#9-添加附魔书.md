---
layout: post
title: "Minecraft Fabric模组开发教程#9 添加附魔书"
date: 2024-01-13T15:17:47+08:00
---

## 创建附魔类

```java
public class FireBoomEnchantment extends Enchantment {
    public FireBoomEnchantment(Rarity rarity, EnchantmentTarget target, EquipmentSlot[] slotTypes) {
        super(rarity, target, slotTypes);
    }
}
```

如果目标被攻击，目标就会爆炸，这里直接使用`FireballEntity`类中`onCollision`方法的部分代码。

```java
@Override
public void onTargetDamaged(LivingEntity user, Entity target, int level) {
    if (target instanceof LivingEntity) {
        boolean bl = user.getWorld().getGameRules().getBoolean(GameRules.DO_MOB_GRIEFING);
        user.getWorld().createExplosion(target, target.getX(), target.getY(), target.getZ(), 1, bl, World.ExplosionSourceType.MOB);
    }
}
```

## 注册附魔

创建`FireBoomEnchantment`对象的时候，需要传入`Enchantment.Rarity`、`EnchantmentTarget`和`EquipmentSlot[]`，其中`VERY_RARE`表示非常稀有，`EnchantmentTarget.WEAPON`表示只能附魔武器，`EquipmentSlot[]`表示只能附魔主手。

```java
public static final FireBoomEnchantment FIRE_BOOM = Registry.register(Registries.ENCHANTMENT, new Identifier("awesome", "fire_boom"), new FireBoomEnchantment(Enchantment.Rarity.VERY_RARE, EnchantmentTarget.WEAPON, new EquipmentSlot[]{
        EquipmentSlot.MAINHAND
}));
```

## 多语言

```json
{
  "enchantment.awesome.fire_boom": "Fire Boom"
}
```

---

附魔书在在原材料的最后

![9-1](/assets/fabric2024/9-1.png)

![9-2](/assets/fabric2024/9-2.png)