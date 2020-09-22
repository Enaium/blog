---
layout: post
title: "Minecraft Fabric Client 教程 #5 添加Event、Sprint和ToggleCommand"
date: 2020-1-20 13:27
categories: fabricclient
---

## 添加Event

下载[![event.zip](/assets/icon/zip.png)](/assets/fabricclient/event.zip)

放在`cn.enaium.excel`里

然后在`Excel.java`里面添加`EventManager`

```java
public enum Excel {
    
    [...]
    public EventManager eventManager;

    public void onEnable() {
        eventManager = new EventManager();
        commandManager = new CommandManager();
        moduleManager = new ModuleManager();

        moduleManager.loadModules();
    }
    [...]

}


```

在`Module.java`里面添加`Event`、`onEnable()`、`onDisable()`、`Toggle()`


```java

public class Module {
    
    [...]

    public void Toggle() {
        this.toggled = !this.toggled;
        if (this.toggled) {
            onEnable();
        } else {
            onDisable();
        }
    }

    public void onEnable() {
        Excel.INSTANCE.eventManager.register(this);
    }

    public void onDisable() {
        Excel.INSTANCE.eventManager.unregister(this);
    }

}


```



### 注入Mixin

在 `ClientPlayerEntityMixin.java` 里面添加以下内容

```java
@Mixin(ClientPlayerEntity.class)
public class ClientPlayerEntityMixin {

    [...]

    @Inject(method = "tick", at = @At("HEAD"))
    private void preTick(CallbackInfo callbackInfo) {
        new EventUpdate().call();
    }
}

```


## 添加Sprint

在`cn.enaium.excel.module`里面新建一个`modules.movement`包

在`movement`包里面新建`Sprint`类

内容

```java
package cn.enaium.excel.module.modules.movement;

import cn.enaium.excel.event.EventTarget;
import cn.enaium.excel.module.Category;
import cn.enaium.excel.module.Module;
import net.minecraft.client.MinecraftClient;

/**
 * @Author Enaium
 * @Date 2020/1/20 13:46
 */
public class Sprint extends Module {
    public Sprint() {
        super("Sprint", Category.MOVEMENT);
    }

    @EventTarget
    public void onUpdate(EventUpdate e) {
        MinecraftClient.getInstance().player.setSprinting(true);
    }
}

```


然后添加到`ModuleManager.java`里面

```java
public class ModuleManager {
    [...]

    public void loadModules() {
        addModule(new Sprint());
    }
}

```


## 添加ToggleCommand

在`ModuleManager.java`里面添加`getModules()`

```java
public class ModuleManager {
    ArrayList<Module> modules;

    [...]

    public Module getModule(String name) {
        for (Module m : modules) {
            if (m.getName().equalsIgnoreCase(name))
                return m;
        }
        return null;
    }
}


```


在`cn.enaium.excel.command.commands`包里面新建`ToggleCommand`类

内容


```java

package cn.enaium.excel.command.commands;

import cn.enaium.excel.Excel;
import cn.enaium.excel.command.Command;
import cn.enaium.excel.module.Module;
import cn.enaium.excel.utils.ChatUtils;

public class ToggleCommand implements Command {

    @Override
    public boolean run(String[] args) {

        if (args.length == 2) {

            Module module = Excel.INSTANCE.moduleManager.getModule(args[1]);

            if (module == null) {
                ChatUtils.message("The module with the name " + args[1] + " does not exist.");
                return true;
            }

            module.Toggle();

            return true;
        }


        return false;
    }

    @Override
    public String usage() {
        return "USAGE: -toggle [module]";
    }
}


```

然后添加到`CommandManager.java`里面

```java

public class CommandManager {
    [...]
        public void loadCommands() {
        [...]
        commands.put(new String[]{"toggle","t"},new ToggleCommand());
    }
    [...]
}

```

完成!
