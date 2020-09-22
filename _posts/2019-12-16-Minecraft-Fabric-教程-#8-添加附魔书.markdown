---
layout: post
title: "Minecraft Fabric 教程 #8 添加附魔书"
date: 2019-12-16 13:45
categories: fabric
---

创建一个附魔书类

```java
public class FireBoomEnchantment extends Enchantment {
    [...]
}
```

在类中添一下


```java
    @Override
    public int getMinimumPower(int level) {
        return 15;
    }

    @Override
    public int getMaximumLevel() {
        return 1;
    }

    @Override
    public void onTargetDamaged(LivingEntity user, Entity target, int level) {
        if(target instanceof LivingEntity) {
            World world = user.world;
            boolean bl = world.getGameRules().getBoolean(GameRules.MOB_GRIEFING);
            world.createExplosion(target, target.prevX, target.prevY, target.prevZ, 1.0f, bl, bl ? Explosion.DestructionType.DESTROY : Explosion.DestructionType.NONE);
            world.spawnEntity(target);
        }
    }
```

这就创建了一个FireBoom附魔书

onTargetDamaged //当目标被攻击

在mc  FireballEntity类有一个 方法就是当火球碰撞了就创建一个火焰爆炸的效果

```java
   protected void onCollision(HitResult hitResult) {
      super.onCollision(hitResult);
      if (!this.world.isClient) {
         if (hitResult.getType() == HitResult.Type.ENTITY) {
            Entity entity = ((EntityHitResult)hitResult).getEntity();
            entity.damage(DamageSource.explosiveProjectile(this, this.owner), 6.0F);
            this.dealDamage(this.owner, entity);
         }

         boolean bl = this.world.getGameRules().getBoolean(GameRules.MOB_GRIEFING);
         this.world.createExplosion((Entity)null, this.getX(), this.getY(), this.getZ(), (float)this.explosionPower, bl, bl ? Explosion.DestructionType.DESTROY : Explosion.DestructionType.NONE);
         this.remove();
      }

   }
```

我们可以加以利用

```java
    boolean bl = world.getGameRules().getBoolean(GameRules.MOB_GRIEFING);
    world.createExplosion(target, target.prevX, target.prevY, target.prevZ, 1.0f, bl, bl ? Explosion.DestructionType.DESTROY : 
    Explosion.DestructionType.NONE);
```

 this.world.createExplosion()

 我们替换相对应的参数 参数一就是实体 target就是攻击目标 参数二、三、四 就是目标 X Y Z 由于 xyz是private 只能用 public 的 prevX prevY prevZ 参数五就是爆炸大小 参数六不用管
world.spawnEntity(target);//生成实体在target

## 创建附魔书

```java
	private static final FireBoomEnchantment END_FIRE_BOOM_ENCHANTMENT = new FireBoomEnchantment(
			Enchantment.Weight.VERY_RARE,
			EnchantmentTarget.WEAPON,
			new EquipmentSlot[] {
					EquipmentSlot.MAINHAND
			}
	);
```

## 注册

```java
		Registry.register(Registry.ENCHANTMENT,new Identifier("endarmor","end_fire_boom_enchantment"),END_FIRE_BOOM_ENCHANTMENT);
```


![8 1](/assets/fabric/8-1.jpg)
![8 2](/assets/fabric/8-2.jpg)
![8 3](/assets/fabric/8-3.jpg)





