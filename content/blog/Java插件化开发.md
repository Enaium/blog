---
title: "Java插件化开发"
date: 2020-05-04T09:17:00+0800
categories: java
---

在java程序开发过程中. 可能需要加载插件的功能. 所以要动态加载Jar文件来实现插件的加载.

我这边用了Kotlin

## 一. 创建接口

```kotlin
interface PluginInitializer {
    fun onInitialize()//插件初始化
}
```

## 二. 加载插件

加载的插件可能会抛出什么异常所以要用try

```kotlin
private val file = File("plugins")
private val plugins: ArrayList<PluginInitializer> = ArrayList()

init {
    try {
            if (file.listFiles().isNotEmpty()) {
                for (f in file.listFiles()) {
                    if (f.name.endsWith(".jar") {//判断文件后缀是否为.jar
                        val u = URLClassLoader(arrayOf<URL>(f.toURL()), Thread.currentThread().contextClassLoader)//加载Jar
                        plugins.add(u.loadClass("cn.enaium.plugin.Test").newInstance() as PluginInitializer)//加载主类
                    }
                }
            }

            if (plugins.isNotEmpty()) {
                for (p in plugins) {
                    p.onInitialize()//初始化插件
                }
            }
        } catch (e: Exception) {

    }
}
```

## 三. 写插件

```java
public class Test implements PluginInitializer {

    @Override
    public void onInitialize() {
        System.out.println("HELLO WORLD!");
    }

}
```

导入Jar然后放入插件目录就可以加载了.


这个主类是固定的 如何把他改为随意的呢？
1. 我们可以在在每一个插件的文件里面都指定一个配置的json里面 然后加载Jar的时候可以用`getResourceAsStream`来获取中的配置

比如 获取配置中的`mainClass`就可以了

```json
{
    "mainClass": "cn.enaium.plugin.Test"
}
```

2. 还可以通过注解来查找主类 只要自定义一个注解比如`@Plugin` 然后遍历Jar文件中全部类中也没用这个 注解就可以找出这个类

## 四. 使用[BullPlugin](https://enaium.github.io/BullPlugin/)框架

这么麻烦有没有更简单的方法呢？

我写了一个加载插件的框架 就是通过寻找指定注解来 查找主类的

