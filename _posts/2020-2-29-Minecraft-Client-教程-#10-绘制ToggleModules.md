---
layout: post
title: "Minecraft Client 教程 #10 绘制ToggleModules"
date: 2020-2-29 18:25
categories: minecraftclient
---

一.添加`EventRender2D`

```java
public class EventRender2D extends Event {
    public EventRender2D()  {
        super(Type.PRE);
    }
}
```

二.钩子

> 1. 搜索`GuiIngame`这个类打开
>> ![10-1](/assets/minecraftclient/10-1.png)
> 2. 找到`renderGameOverlay`这个类
> 3. 在`if (this.mc.playerController.isSpectator()){[...]}else{[...]}`后面添加
```java
GlStateManager.pushMatrix();
new EventRender2D().call();
GlStateManager.popMatrix();
```

三. 绘制HUD
> 1. 新建HUD类
> 2. 在onRender方法里面写绘制
> 3. 添加到loadMods`this.addModule(new HUD());`

```java
package cn.enaium.coreium.module.render;

import cn.enaium.coreium.event.EventTarget;
import cn.enaium.coreium.event.events.EventRender2D;
import cn.enaium.coreium.module.Category;
import cn.enaium.coreium.module.Module;
import org.lwjgl.input.Keyboard;

public class HUD extends Module {
    public HUD() {
        super("HUD", Keyboard.KEY_P, Category.RENDER);
    }

    @EventTarget
    public void onRender(EventRender2D e) {
        
    }
}
```

```java
        //字体
        FontRenderer fr = mc.fontRendererObj;
        //获取屏幕长和高
        ScaledResolution sr = new ScaledResolution(mc);
        ArrayList<Module> modules = new ArrayList();
        for (Module m : Coreium.INSTANCE.moduleManager.getModules()) {
            //过滤没打开的Module
            if (m.isToggle())
                modules.add(m);
        }
        //根据Module名长度从长到短排序
        modules.sort((o1, o2) -> fr.getStringWidth(o2.getName()) - fr.getStringWidth(o1.getName()));
        //第一个显示的位置
        int index = 0;

        for (Module m : modules) {
            //获取当前要绘制的字体长度
            int mWidth = fr.getStringWidth(m.getName());
            //绘制字体
            fr.drawStringWithShadow(m.getName(), sr.getScaledWidth() - mWidth - 2, index, new Color(0, 16, 255).getRGB());
            //每绘制一个Module下一个Module的高度增加12
            index += 12;
        }
```

完成
![10-2](/assets/minecraftclient/10-2.png)
