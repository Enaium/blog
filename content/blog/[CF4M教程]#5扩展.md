---
title: "[CF4M教程]#5扩展"
data: 2021-2-22 23:30
categories: cf4m
---

现在CF4M已经更新到了1.4.7版本，现在不需用调用stop了。

```java
public enum Example {

    INSTANCE;

    public void run() {
        CF4M.INSTANCE.run(this, Minecraft.getMinecraft().mcDataDir.toString() + "/Example");
    }
}
```

只需用在Module中创建一个变量，加上Setting注解即可。

```java
@Module("Sprint")
public class Sprint {
    @Setting("test1")
    private EnableSetting test1 = new EnableSetting(false);
}
```

CF4M没有附带Setting，需要自定义一个Setting。


```java
public class EnableSetting {

    private boolean enable;

    public EnableSetting(boolean enable) {
        this.enable = enable;
    }

    public boolean getEnable() {
        return enable;
    }

    public void setEnable(boolean enable) {
        this.enable = enable;
    }
}
```

自定义Event，继承Listener，要注意是CF4M的Listener，At这个参数是枚举型，分别是HEAD、TAIL和NONE，表示监听的位置，如果没有确定位置就用NONE。

要调用Call才能使Event监听这个事件。

```java
public class Render2DEvent extends Listener {
    public Render2DEvent() {
        super(At.HEAD);
    }
}
```

CF4M还给Module提供了扩展变量，创建一个类加上Extend注解，变量加上Value注解，Value的值表示这个变量的名字。

```java
@Extend
public class Module {
    @Value("tag")
    String Haha;
}
```

可以使用SetValue来进行变量的复制，有三个参数，分别是Module、变量名字、和值，如果变量比较特殊可以加上泛型。

```java
@Event
private void onUpdate(UpdateEvent updateEvent) {
    CF4M.INSTANCE.module.setValue(this, "tag", "Auto");
}
```

```java
CF4M.INSTANCE.module.<String>setValue(this, "tag", "Auto");
```

如果要获取module的话，也可以使用getValue来获取变量，当然获取变量也可以使用泛型。

```java
private String getDisplayName(Object module) {
    String name = CF4M.INSTANCE.module.getName(module);
    String tag = CF4M.INSTANCE.module.getValue(module, "tag");
    String displayName;
    if (tag != null) {
        displayName = name + " " + tag;
    } else {
        displayName = name;
    }
    return displayName;
}
```

```java
CF4M.INSTANCE.module.<String>getValue(module, "tag");
```