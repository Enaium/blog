---
title: "Minecraft Fabric Client 教程 #4 添加Modules"
date: 2020-01-09T10:58:00+0800
categories: fabricclient
---

在`cn.enaium.excel`下新建一个包`module`

在`module`包里新建`Module`、`ModuleManager`这2个类 然后再新建一个`Category`枚举


`Module`内容
```java
package cn.enaium.excel.module;

/**
 * @Author Enaium
 * @Date 2020/1/9 11:03
 */
public class Module {

    private String name;
    private Category category;

    private boolean toggled;


    public Module(String name,Category category) {
        this.name = name;
        this.category = category;
        this.toggled = false;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Category getCategory() {
        return category;
    }

    public void setCategory(Category category) {
        this.category = category;
    }

    public boolean isToggled() {
        return toggled;
    }

    public void setToggled(boolean toggled) {
        this.toggled = toggled;
    }
}

```

`ModuleManager` 内容


```java
package cn.enaium.excel.module;

import java.util.ArrayList;

/**
 * @Author Enaium
 * @Date 2020/1/9 11:03
 */
public class ModuleManager {
    ArrayList<Module> modules;

    public ModuleManager() {
        modules = new ArrayList();
    }

    private void addModule(Module e) {
        this.modules.add(e);
    }

    public void loadModules() {
        
    }
}

```



`Category`内容

```java
package cn.enaium.excel.module;

/**
 * @Author Enaium
 * @Date 2020/1/9 11:03
 */
public enum Category {

    COMBAT,MOVEMENT,RENDER,OTHER;

}

```

然后再`Excel`里面添加Module

```java
public enum Excel {
    [...]
    public ModuleManager moduleManager;

    public void onEnable() {
        [...]
        moduleManager = new ModuleManager();

        moduleManager.loadModules();
    }
    [...]
}
```
