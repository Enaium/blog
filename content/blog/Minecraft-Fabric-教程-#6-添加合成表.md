---
title: "Minecraft Fabric 教程 #6 添加合成表"
date: 2019-12-15T20:48:00+0800
categories: fabric
---

位置 `src\main\resources\data\endarmor\recipes\end_heart_block.json`

```java
{
  "type": "minecraft:crafting_shaped",
  "pattern": [
    "WWW",
    "WWW",
    "WWW"
  ],
  "key": {
    "W": {
      "item": "endarmor:end_heart"
    }
  },
  "result": {
    "item": "endarmor:end_heart_block",
    "count": 4
  }
}
```

W就是key里面的物品如果是空的就打一个空格比如W W
