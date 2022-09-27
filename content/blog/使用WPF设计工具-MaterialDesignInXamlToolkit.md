---
title: "使用WPF设计工具-MaterialDesignInXamlToolkit"
date: 2020-2-3 10:18
categories: csharp
---

[MaterialDesignInXamlToolkit](https://github.com/MaterialDesignInXAML/MaterialDesignInXamlToolkit)



## 安装Nuget包`MaterialDesignThemes`

![Nuget](/assets/csharp/2020-2-3-1.png)

## 添加到`App.xaml`

```html
<ResourceDictionary>
    <ResourceDictionary.MergedDictionaries>
        <ResourceDictionary Source="pack://application:,,,/MaterialDesignThemes.Wpf;component/Themes/MaterialDesignTheme.Light.xaml" />
        <ResourceDictionary Source="pack://application:,,,/MaterialDesignThemes.Wpf;component/Themes/MaterialDesignTheme.Defaults.xaml" />
        <ResourceDictionary Source="pack://application:,,,/MaterialDesignColors;component/Themes/Recommended/Primary/MaterialDesignColor.DeepPurple.xaml" />
        <ResourceDictionary Source="pack://application:,,,/MaterialDesignColors;component/Themes/Recommended/Accent/MaterialDesignColor.Lime.xaml" />
    </ResourceDictionary.MergedDictionaries>
</ResourceDictionary>
```


![img 2](/assets/csharp/2020-2-3-2.png)

## 加入`Window`中

`xmlns:materialDesign="http://materialdesigninxaml.net/winfx/xaml/themes"`

![img 3](/assets/csharp/2020-2-3-3.png)



完成