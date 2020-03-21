---
layout: post
title: "Minecraft Client 教程 #12 绘制ClickGUI"
date: 2020-3-14 17:52
categories: minecraftclient
---

一. 先复制进去`FontUtils`

## FontUtils

```java
package cn.enaium.coreium.utils;

import net.minecraft.client.Minecraft;
import net.minecraft.client.gui.FontRenderer;
import net.minecraft.util.StringUtils;

public class FontUtils {
    private static FontRenderer fontRenderer;

    public static void setupFontUtils() {
        fontRenderer = Minecraft.getMinecraft().fontRendererObj;
    }

    public static int getStringWidth(String text) {
        return fontRenderer.getStringWidth(StringUtils.stripControlCodes(text));
    }

    public static int getFontHeight() {
        return fontRenderer.FONT_HEIGHT;
    }

    public static void drawString(String text, int x, int y, int color) {
        fontRenderer.drawString(text, x, y, color);
    }

    public static void drawStringWithShadow(String text, double x, double y, int color) {
        fontRenderer.drawStringWithShadow(text, (float)x, (float)y, color);
    }

    public static void drawCenteredString(String text, int x, int y, int color) {
        FontUtils.drawString(text, x - fontRenderer.getStringWidth(text) / 2, y, color);
    }

    public static void drawCenteredStringWithShadow(String text, double x, double y, int color) {
        FontUtils.drawStringWithShadow(text, x - (double)(fontRenderer.getStringWidth(text) / 2), y, color);
    }

    public static void drawTotalCenteredString(String text, int x, int y, int color) {
        FontUtils.drawString(text, x - fontRenderer.getStringWidth(text) / 2, y - fontRenderer.FONT_HEIGHT / 2, color);
    }

    public static void drawTotalCenteredStringWithShadow(String text, double x, double y, int color) {
        FontUtils.drawStringWithShadow(text, x - (double)(fontRenderer.getStringWidth(text) / 2), y - (double)((float)fontRenderer.FONT_HEIGHT / 2.0f), color);
    }
}
```

二. 开始绘制

## ClickGUI

```java
package cn.enaium.coreium.gui.clickgui;

import cn.enaium.coreium.module.Category;
import net.minecraft.client.gui.Gui;
import net.minecraft.client.gui.GuiScreen;

import java.io.IOException;
import java.util.ArrayList;

public class ClickGUI extends GuiScreen {

    public ArrayList<CategoryPanel> categoryPanels;

    public ClickGUI() {
        //添加Category
        categoryPanels = new ArrayList<>();
        int categoryPanelsY = 5;
        for (Category c : Category.values()) {
            categoryPanels.add(new CategoryPanel(c, 5, categoryPanelsY, 100, 20));
            categoryPanelsY += 30;
        }
    }

    @Override
    public void drawScreen(int mouseX, int mouseY, float partialTicks) {
        for (CategoryPanel c : categoryPanels) {
            c.drawScreen(mouseX, mouseY);//绘制所有Category
        }
        super.drawScreen(mouseX, mouseY, partialTicks);
    }

    @Override
    public void mouseClicked(int mouseX, int mouseY, int mouseButton) throws IOException {
        for (CategoryPanel c : categoryPanels) {
            c.mouseClicked(mouseX, mouseY,mouseButton);//调用所有CategoryPanel的mouseClicked方法
        }
        super.mouseClicked(mouseX, mouseY, mouseButton);
    }

    @Override
    public void mouseReleased(int mouseX, int mouseY, int state) {
        for (CategoryPanel c : categoryPanels) {
            c.mouseReleased(mouseX, mouseY,state);//调用所有CategoryPanel的mouseReleased方法
        }
        super.mouseReleased(mouseX, mouseY, state);
    }

    public static boolean isHovered(int mouseX, int mouseY, int x, int y, int width, int height) {
        return mouseX >= x && mouseX - width <= x && mouseY >= y && mouseY - height <= y;//获取鼠标位置是否在指定位置
    }


    public static void drawRect(int x, int y, int width, int height, int color) {
        Gui.drawRect(x, y, x + width, y + height, color);//绘制Rect
    }
}
```

## CategoryPanel

```java
package cn.enaium.coreium.gui.clickgui;

import cn.enaium.coreium.Coreium;
import cn.enaium.coreium.module.Category;
import cn.enaium.coreium.module.Module;
import cn.enaium.coreium.utils.FontUtils;
import net.minecraft.client.Minecraft;
import net.minecraft.client.gui.FontRenderer;
import net.minecraft.client.gui.Gui;

import java.awt.*;
import java.util.ArrayList;

public class CategoryPanel {


    private Category category;
    private boolean hovered;
    //单独Category的位置
    private int x;
    private int y;
    //单独Category的长高
    private int width;
    private int height;

    public boolean dragging;//是否为移动状态
    //临时单独Category的位置（上一个位置）
    private int tempX;
    private int tempY;
    //是否显示ModulePanel
    private boolean displayModulePanel;
    //ModulePanel列表
    private ArrayList<ModulePanel> modulePanels;

    public CategoryPanel(Category category, int x, int y, int width, int height) {
        this.category = category;
        this.x = x;
        this.y = y;
        this.width = width;
        this.height = height;
        FontUtils.setupFontUtils();//设置Font
        modulePanels = new ArrayList<>();

        ArrayList<Module> modules = new ArrayList<>();
        modules.addAll(Coreium.instance.moduleManager.getModulesForCategory(this.category));//获取该分类所以Module
        for (Module m : modules) {
            modulePanels.add(new ModulePanel(m));//添加ModulePanel
        }
    }


    public void drawScreen(int mouseX, int mouseY) {
        this.hovered = ClickGUI.isHovered(mouseX, mouseY, this.x, this.y, this.width, this.height);//获取鼠标是否在指定位置
        if (this.dragging) {
            //移动CategoryPanel
            this.x = this.tempX + mouseX;
            this.y = this.tempY + mouseY;
        }
        //改变Category颜色
        int color = new Color(0, 190, 255).getRGB();
        if (this.hovered) color = new Color(0, 88, 120).getRGB();
        ClickGUI.drawRect(x, y, this.width, this.height, color);//绘制Category背景
        FontUtils.drawCenteredString(this.category.name(), x + this.width / 2, y + this.height / 2, Color.WHITE.getRGB());//绘制Category的标题
        int modulePanelsY = this.y + this.height;
        //绘制该Category下的所有Module
        if(this.displayModulePanel) {
            for (ModulePanel module : modulePanels) {
                module.drawScreen(mouseX,mouseY,this.x + 10, modulePanelsY, 80, 20);
                modulePanelsY += 20;
            }
        }
    }

    public void mouseClicked(int mouseX, int mouseY, int mouseButton) {
        //如果鼠标在指定位置
        //如果鼠标左键按下
        if (this.hovered && mouseButton == 0) {
            //移动状态为true
            dragging = true;
            //给临时坐标赋值
            this.tempX = this.x - mouseX;
            this.tempY = this.y - mouseY;
        } else if (this.hovered && mouseButton == 1) {//如果鼠标右键被按下
            this.displayModulePanel = !this.displayModulePanel;//是否显示Module
        }
        for (ModulePanel modulePanel : modulePanels) {//调用所有ModulePanel的mouseClicked方法
            modulePanel.mouseClicked(mouseX,mouseY,mouseButton);
        }
    }

    public void mouseReleased(int mouseX, int mouseY, int state) {
        //如果鼠标左键被释放退出移动Category模式
        if (state == 0) {
            this.dragging = false;
        }
    }


}
```

## CategoryPanel

```java
package cn.enaium.coreium.gui.clickgui;

import cn.enaium.coreium.module.Module;
import cn.enaium.coreium.utils.FontUtils;

import java.awt.*;

public class ModulePanel {

    private Module module;
    private boolean hovered;

    public ModulePanel(Module module) {
        this.module = module;
        FontUtils.setupFontUtils();//设置Font
    }

    public void drawScreen(int mouseX, int mouseY, int x, int y, int width, int height) {
        this.hovered = ClickGUI.isHovered(mouseX, mouseY, x, y, width, height);//鼠标是否在指定位置
        int color = new Color(200, 190, 255).getRGB();//颜色
        if (this.module.isToggle()) color = new Color(200, 0, 120).getRGB();//Module打开的颜色
        if (this.hovered) color = new Color(200, 88, 120).getRGB();//鼠标在指定位置的颜色
        ClickGUI.drawRect(x, y, width, height, color);//绘制Module的背景
        FontUtils.drawCenteredString(this.module.getName(), x + width / 2, y + height / 2, Color.WHITE.getRGB());//绘制Module的名字
    }


    public void mouseClicked(int mouseX, int mouseY, int mouseButton) {
        if(this.hovered && mouseButton == 0) {
            this.module.toggle();//当鼠标在指定位置并且鼠标被按下设置Module为关闭或打开
        }
    }
}
```

三. 打开`ClickGUI`

## Click

```java
package cn.enaium.coreium.module.modules.render;

import cn.enaium.coreium.gui.clickgui.ClickGUI;
import cn.enaium.coreium.module.Category;
import cn.enaium.coreium.module.Module;
import org.lwjgl.input.Keyboard;

public class Click extends Module {
    public Click() {
        super("ClickGUI", Keyboard.KEY_RSHIFT, Category.RENDER);
    }

    @Override
    public void onEnable() {
        super.onEnable();
        mc.displayGuiScreen(new ClickGUI());
        toggle();
    }
}
```


## getModulesForCategory

```java
    public ArrayList<Module> getModulesForCategory(Category c) {
        ArrayList<Module> modules = new ArrayList<>();

        for (Module m : this.modules ) {
            if(m.getCategory().equals(c))
                modules.add(m);
        }

        return modules;
    }
```