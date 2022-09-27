---
title: "Minecraft Fabric 进阶教程 #2 绘制界面"
date: 2020-02-24T16:27:00+0800
categories: fabric
---

绘制界面不需用注入Mixin


## 新建一个类

`ExampleGui.java`

继承`Screen`构造器

```java
public ExampleGui() {
    super(new LiteralText(""));
}
```

## 绘制


这是绘制一个背景

绘制背景这种全部覆盖时要在super上面不然按钮或者其他东西会被背景盖住

```java
@Override
public void render(int mouseX, int mouseY, float delta) {
    renderBackground();
    super.render(mouseX, mouseY, delta);
}
```


我们也可以添加一个按钮
```java
@Override
public void init() {
    super.init();
    this.addButton(new ButtonWidget(20,20,100,20,"Done",(action)->{  
        
    }));
}
```
打开界面

将上集绘制的按钮的action改为打开这个界面
```java
@Inject(at = @At("HEAD"), method = "init()V")
private void init(CallbackInfo info) {
	this.addButton(new ButtonWidget(20,20,200,20,"233",(action)->{
		MinecraftClient.getInstance().openScreen(new ExampleGui());
	}));
}
```


打开界面后我们发现只能用ESC来关闭界面


接下来我们要写返回上一界面的功能


在构造器里传入上个界面Screen
```java
private Screen screen;
public ExampleGui(Screen screen) {
    super(new LiteralText(""));
    this.screen = screen;
}
```

打开界面 `MinecraftClient.getInstance().openScreen(new ExampleGui(this));`

将按钮的action写为`MinecraftClient.getInstance().openScreen(screen);`

运行


![2-1](/assets/fabric/s2-1.png)


完成
