---
title: "Minecraft启动器开发#4下载库"
date: 2021-05-15T19:49:00+0800
categories: java
---

```java
var libraryDir = new File(gameDir,"libraries");
```

设置库的路径

```java
for (JsonElement jsonElement : gameJson.get("libraries").getAsJsonArray()) {
    var downloads = jsonElement.getAsJsonObject().get("downloads").getAsJsonObject();
    if (downloads.has("artifact")) {
        var artifact = downloads.get("artifact").getAsJsonObject();
        var path = new File(libraryDir, artifact.get("path").getAsString());
        if (!path.exists()) {
            FileUtils.writeByteArrayToFile(path,IOUtils.toByteArray(new URL(artifact.get("url").getAsString())));
        }
    }
}
```

遍历出所有库 获取url下载到指定路径

```java
var libraryDir = new File(gameDir,"libraries");

for (JsonElement jsonElement : gameJson.get("libraries").getAsJsonArray()) {
    var downloads = jsonElement.getAsJsonObject().get("downloads").getAsJsonObject();
    if (downloads.has("artifact")) {
        var artifact = downloads.get("artifact").getAsJsonObject();
        var path = new File(libraryDir, artifact.get("path").getAsString());
        libraries.append(path).append(";");
        if (!path.exists()) {
            FileUtils.writeByteArrayToFile(path,IOUtils.toByteArray(new URL(artifact.get("url").getAsString())));
        }
    }
}

libraries.append(gameJarFile);
```

添加到classpath中用`;`分号隔开，最后再把游戏的Jar文件添加进去
