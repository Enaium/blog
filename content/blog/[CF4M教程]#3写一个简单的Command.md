---
title: "[CF4M教程]#3写一个简单的Command"
date: 2021-02-18T14:41:00+0800
categories: cf4m
---

创建一个简单的Command，就叫“ToggleCommand”吧。

```java
public class ToggleCommand {
}
```

添加一个Command注解，只有一个参数，所以value不用写，String数组类型，就是使用这个Command的命令，可以使用“h”或者“help”来使用，当然，也可以添加更多的。

```java
@Command({"t", "toggle"})
public class ToggleCommand {
}
```

只需要一个方法，加上Exec注解，命令有几个参数，方法写几个参数，就可以。

```java
@Command({"t", "toggle"})
public class ToggleCommand {
    @Exec
    private void exec(@Param("module") String name) {
        Object module = CF4M.INSTANCE.module.getModule(name);

        if (module == null) {
            CF4M.INSTANCE.configuration.message("The module with the name " + name + " does not exist.");
            return;
        }

        CF4M.INSTANCE.module.enable(module);
    }
}
```

那么这个“configuration.message”是什么呢？

我们需要重写配置接口，来进行文本输出，默认是在控制台输出，注意要加上“Configuration” 

```java
@Configuration
public class ExampleConfig implements IConfiguration {
    @Override
    public void message(String message) {
        MinecraftClient.getInstance().inGameHud.getChatHud().addMessage(new LiteralText(message));
    }

    @Override
    public String prefix() {
        return "-";
    }

}
```

还可以重写prefix来进行更改，Command的前缀，默认是“`”，就是键盘左上角哪个斜点，
我们把它改成“-”。

支持这几种类型直接转换。

```java
@Exec
private void exec(Boolean a, Integer b, Float c, Double d, Long e, Short f, Byte g) {
}
```

也可以有无参数方法，相同数量的参数只方法只能有一个，如果参数没有Param注解那么这个参数就没有名字。

```java
@Exec
private void exec() {
}

@Exec
private void exec(Integer integer) {

}
```
