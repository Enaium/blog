---
title: "Minecraft Fabric 教程 #4 添加分组"
date: 2019-12-15T19:56:00+0800
categories: fabric
---

在 ItemGroup 显示 使用 FabricItemGroupBuilder

```java
	public static final ItemGroup END_ITEM_GROUP = FabricItemGroupBuilder.create(
			new Identifier("endarmor", "endarmor"))
			.icon(() -> new ItemStack(END_HEART))
			.appendItems(stacks ->
			{
				stacks.add(new ItemStack(END_HEART));
				stacks.add(new ItemStack(END_HEART_BLOCK));
			})
			.build();
```

直接创建即可

new Identifier("endarmor", "endarmor") //第一个参数modid 第二个参数名字 只能用的 [a-z0-9_.-] 不要使用其他符号
