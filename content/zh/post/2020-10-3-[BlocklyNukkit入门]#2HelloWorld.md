---
layout: post
title: "[BlocklyNukkit入门]#2HelloWorld"
date: 2020-10-03T20:32:00+08:00
categroy: blocklynukkit
---

## 编写插件

`BlocklyNukkit`以下简称`bn`

bn支持`JavaScript`、`Python`和`Lua`等脚本语言 也有[图形编辑器](https://tools.blocklynukkit.com/) 

### JavaScript

```javascript
logger.info("Hello JavaScript!");
```

### Python

注意！ 如果有汉字等特殊字符要加上`# -*- encoding: utf-8 -*-`注释

```python
print u"Hello Python!"
```

### Lua

```lua
logger:info("Hello Lua!")
```

## 使用插件

放入文件夹`./plugin/BlocklyNukkit`里面
