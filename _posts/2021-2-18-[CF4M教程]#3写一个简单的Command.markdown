---
layout: post
title: "[CF4M教程]#3写一个简单的Module"
data: 2021-2-18 14:41
categories: cf4m
---

创建一个简单的Command，就叫“HelpCommand”吧。

```java
public class HelpCommand {
}
```

添加一个Command注解，只有一个参数，所以value不用写，String数组类型，就是使用这个Command的命令，可以使用“h”或者“help”来使用，当然，也可以添加更多的。

```java
@Command({"h", "help"})
public class HelpCommand {
}
```

使用“ICommand”这个接口

```java
@Command({"h", "help"})
public class HelpCommand implements ICommand {
}
```

需要写这个两个方法，先看run方法，参数先不管，如果返回值为false就说明这个Command执行失败，还有usage方法，返回Command参数，比如：需要一个Module参数，就写“<Module>”,如果一个Command有多个参数，并且有多个使用方法，那么就用”\n”隔开，如果没有参数就返回一个空字符串。

```java
@Command({"h", "help"})
public class HelpCommand implements ICommand {
    @Override
    public boolean run(String[] args) {
        return false;
    }
        
    @Override
    public String usage() {
        return "";
    }
}
```

当我们输入这个指令的时候，遍历出所有Command的使用方法，

```java
@Command({"h", "help"})
public class HelpCommand implements ICommand {

    @Override
    public boolean run(String[] args) {
        CF4M.INSTANCE.configuration.message("Here are the list of commands:");
        for (Map.Entry<String[], ICommand> entry : CF4M.INSTANCE.command.getCommands().entrySet()) {
            CF4M.INSTANCE.configuration.message(Arrays.toString(entry.getKey()));
        }
        return true;
    }

    @Override
    public String usage() {
        return "help";
    }
}
```

那么这个“configuration.message”是什么呢？

```java
@Configuration
public class ExampleConfig implements IConfiguration {
    @Override
    public void message(String message) {
        MinecraftClient.getInstance().inGameHud.getChatHud().addMessage(new LiteralText(message));
    }
}
```

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

通过上面的学习，我们知道了如何创建一个Command，
接下来我们创建一个EnableCommand，用于打开或者关闭Module

```java
@Command({"e", "enable"})
public class EnableCommand implements ICommand {

    @Override
    public boolean run(String[] args) {
        if (args.length == 2) {//先判断参数是不是1个，因为输入e或者enable本身也占用一个位置，所以长度就是2，虽然数组从零开始但是长度不是。
            Object module = CF4M.INSTANCE.module.getModule(args[1]);//通过输入的字符串来获取Module，要注意的是CF4M和通常的Base不一样获取的是对象（Object），没有Module这父类。

            if (module == null) {//继续判断，如果没找到这个Module就返回null，也要判断是不是null。
                CF4M.INSTANCE.configuration.message("The module with the name " + args[1] + " does not exist.");//输出这个类么有找到。
                return true;
            }

            CF4M.INSTANCE.module.enable(module);//如果找到了，就执行这个方法，打开这个Module。
            return true;
        }
        return false;
    }

    @Override
    public String usage() {
        return "<module>";//有一个参数，返回这个参数。
    }
}
```