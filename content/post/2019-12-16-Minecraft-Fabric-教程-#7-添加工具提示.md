---
layout: post
title: "Minecraft Fabric 教程 #7 添加工具提示"
date: 2019-12-16T12:30:00+08:00
categroy: fabric
---


## 添加工具提示

在EndHeart类中添加

```java
    @Override
    public void appendTooltip(ItemStack stack, World world, List<Text> tooltip, TooltipContext context) {
        tooltip.add(new TranslatableText("tooltip.endarmor.end_heart"));
    }
```

![7 1](/assets/fabric/7-1.jpg)
