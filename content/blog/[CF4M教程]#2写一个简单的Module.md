---
title: "[CF4M教程]#2写一个简单的Module"
date: 2021-02-16T13:02:00+0800
categories: cf4m
---

创建一个`Sprint`类.

```java
@Module("Sprint")
public class Sprint {
}
```

加上了一个`@Module`注解.

默认参数`value` 就是Module的名字.

## 打开或关闭

在加上2个方法 分别加上`@Enable`和`@Disbale`.

```java
@Module("Sprint")
public class Sprint {
    @Enable
    public void onEnable() {
        System.out.println("onEnable");
    }

    @Disable
    public void onDisable() {
        System.out.println("onDisable");
    }
}
```

在这个Module打开或者关闭的时候,分别调用`onEnable`和`onDisable`. 当然方法名可以随别写.

## 事件

```java
@Module("Sprint")
public class Sprint {
    @Event
    private void onUpdate(UpdateEvent updateEvent) {
        Minecraft.getMinecraft().thePlayer.setSprinting(true);//设置疾跑状态为true
    }

    @Enable
    public void onEnable() {
        System.out.println("onEnable");
    }

    @Disable
    public void onDisable() {
        System.out.println("onDisable");
    }
}
```

在已经打开这个Module的前提下,如果触发了这个事件,那么就会执行这个方法.

## 快捷键,分类,默认启用

`@Module`注解有4个值,其中3个值都有默认值
```java
String value();//Module名无默认值,必须填写

boolean enable() default false;//如果需要默认启用的话那么就设置为true

int key() default 0;//快捷键

Category category() default Category.NONE;//分类
```

设置快捷键,需要注意的是,如果如果只有一个参数可以省略掉`value`,因为某些参数有默认值,如果不指定哪一变量的值是什么的话,分不清哪个是哪个,就像在其他语言中有默认值的函数参数只能放在最后一样.

```java
@Module(value = "Sprint",key = Keyboard.KEY_V)
public class Sprint {
    @Event
    private void onUpdate(UpdateEvent updateEvent) {
        Minecraft.getMinecraft().thePlayer.setSprinting(true);//设置疾跑状态为true
    }

    @Enable
    public void onEnable() {
        System.out.println("onEnable");
    }

    @Disable
    public void onDisable() {
        System.out.println("onDisable");
    }
}
```

## 事件

需要在游戏按下快捷键的时候调用`CF4M.MODULE.onKey(key)`才可以使Module启用或者禁用

也可以写一个`Event` 然后使用`CF4M.EVENT.post(new Event())`
