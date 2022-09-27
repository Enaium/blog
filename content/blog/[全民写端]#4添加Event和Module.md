---
title: "[全民写端]#4添加Event和Module"
date: 2020-2-25 13:03
categories: minecraftclient
---


一. 添加Event
> 1. 下载[Event](/assets/minecraftclient/event.zip)解压到你想要的目录
> 2. 修改Event
>> ![4-1](/assets/minecraftclient/4-1.png)

二. 钩子

> 1. 在`EntityPlayerSP`类里面找到`onUpdate`方法 在第一行写 `new EventUpdate().call();`
> 2. 在`Minecraft`类里找到`runTick`方法 找到`Keyboard.next()`循环 在`if (k == 62 && this.entityRenderer != null)`下面写 `new EventKeyboard(k).call();`

```java
    public void onUpdate()
    {

        new EventUpdate().call();
```

```java
if (k == 62 && this.entityRenderer != null)
{
    this.entityRenderer.switchUseShader();
}

    new EventKeyboard(k).call();

    if (this.currentScreen != null)
```

三. 写Category枚举
```java
package cn.enaium.coreium.module;

public enum Category {
    COMBAT,
    RENDER,
    MOVEMENT,
    PLAYER,
    OTHER
}
```

四. 写Module类
```java
package cn.enaium.coreium.module;

import cn.enaium.coreium.Coreium;

public class Module {
    private boolean toggle;
    private String name;
    private int keyCode;
    private Category category;

    public Module(String name, int keyCode, Category category) {
        this.toggle = false;
        this.name = name;
        this.keyCode = keyCode;
        this.category = category;
    }

    public boolean isToggle() {
        return toggle;
    }

    public void setToggle(boolean toggle) {
        this.toggle = toggle;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getKeyCode() {
        return keyCode;
    }

    public void setKeyCode(int keyCode) {
        this.keyCode = keyCode;
    }

    public Category getCategory() {
        return category;
    }

    public void setCategory(Category category) {
        this.category = category;
    }

    public void toggle()
    {
        this.toggle = !this.toggle;
        if(this.toggle) onEnable(); else onDisable();
    }

    public void onEnable() {
        Coreium.INSTANCE.eventManager.register(this);
    }

    public void onDisable() {
        Coreium.INSTANCE.eventManager.unregister(this);
    }
}
```

五. 写ModuleManager类
```java
package cn.enaium.coreium.module;

import cn.enaium.coreium.Coreium;
import cn.enaium.coreium.event.EventTarget;
import cn.enaium.coreium.event.events.EventKeyboard;

import java.util.ArrayList;

public class ModuleManager {
    private ArrayList<Module> modules;

    public ModuleManager() {
        this.modules = new ArrayList();
        Coreium.INSTANCE.eventManager.register(this);
    }

    public void loadMods() {

    }


    private void addModule(Module m) {
        modules.add(m);
    }

    @EventTarget
    public void onKey(EventKeyboard eventKeyBoard) {
        for (Module mod : modules) {
            if (mod.getKeyCode() == eventKeyBoard.getKey())
                mod.toggle();
        }
    }

    public ArrayList<Module> getModules() {
        return modules;
    }
}
```

六. 在Start添加
```java
    public void start() {
        eventManager = new EventManager();
        moduleManager = new ModuleManager();
        Display.setTitle("Coreium");
        moduleManager.loadMods();
    }
```

