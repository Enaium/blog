---
layout: post
title: "Minecraft Fabric 进阶教程 #1 绘制按钮"
date: 2020-2-24 13:56
categories: fabric
---

## 在Mixin包里新建一个类

注入到`TitleScreen.class`里

注入到这个类的init的方法的头部也就是最上面所以at是`HEAD`

因为init方法没有参数所以方法就是`init()V`

因为注入都有一个回调信息所以要有一个回调参数`CallbackInfo`


```java
@Mixin(TitleScreen.class)
public class TitleMixin {
	@Inject(at = @At("HEAD"), method = "init()V")
	private void init(CallbackInfo info) {

	}
}
```


用IDEA反编译`TitleScreen.class`init方法可以看到`this.addButton`这个就是添加按钮


![1-1](/assets/fabric/s1-1.png)

所以我们要在Mixin里面写添加按钮

需要继承Screen类

按照提示生成构造器就行了
```java
protected TitleMixin(Text title) {
	super(title);
}
```

## 现在添加按钮

```java
@Inject(at = @At("HEAD"), method = "init()V")
private void init(CallbackInfo info) {
	this.addButton(new ButtonWidget(20,20,200,20,"233",(action)->{
		System.out.println("By Enaium");
	}));
}
```

在x20 y20处绘制一个长200高20(高必须为20)的按钮标题为“233”点击触发action输出`By Enaium`

添加到mixin.json

我们运行一下看看

![1-2](/assets/fabric/s1-2.png)

成功