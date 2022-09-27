---
title: "Minecraft启动器开发#3下载游戏"
data: 2021-5-15 16:31
categories: java
---

`gameVersion`改为`titleVersion `

`var gameVersion = "1.8.9";`声明变量，指定游戏版本

引入`commons-io`和`gson`

```xml
<dependency>
    <groupId>commons-io</groupId>
    <artifactId>commons-io</artifactId>
    <version>2.8.0</version>
</dependency>
<dependency>
    <groupId>com.google.code.gson</groupId>
    <artifactId>gson</artifactId>
    <version>2.8.6</version>
</dependency>
```

读取游戏列表访问`https://launchermeta.mojang.com/mc/game/version_manifest_v2.json`

```java
var gson = new Gson();
        var gameListJson = IOUtils.toString(new URL("https://launchermeta.mojang.com/mc/game/version_manifest_v2.json"), StandardCharsets.UTF_8);
        var gameList = gson.fromJson(gameListJson, JsonObject.class).get("versions").getAsJsonArray();
        var gameJsonURL = "";
        for (JsonElement jsonElement : gameList) {
            if (jsonElement.getAsJsonObject().get("id").getAsString().equals(gameVersion)) {
                gameJsonURL = jsonElement.getAsJsonObject().get("url").getAsString();
            }
        }

        if (gameJsonURL.equals("")) {
            throw new RuntimeException(gameVersion + " Not Found!");
        }
```

获取`versions`遍历出所以版本，之后获取指定版本的`json`

如果没找到版本就抛出异常

```java
var gameJson = gson.fromJson(IOUtils.toString(new URL(gameJsonURL), StandardCharsets.UTF_8), JsonObject.class);
var gameJarUrl = gameJson.get("downloads").getAsJsonObject().get("client").getAsJsonObject().get("url").getAsString();
```

获取jar文件的url

```java
var versionsDir = new File(gameDir, "versions");

if (!versionsDir.exists()) {
    versionsDir.mkdir();
}

var versionDir = new File(versionsDir, gameVersion);

if (!versionDir.exists()) {
    versionDir.mkdir();
}

var gameJarFile = new File(versionDir,gameVersion + ".jar");
```

获取本地的Jar路径

```java
if (!gameJarFile.exists()) {
    FileUtils.writeByteArrayToFile(gameJarFile, IOUtils.toByteArray(new URL(gameJarUrl)));
}
```

如果没有这个文件那么就下载

```java
var gameDir = new File(".",".minecraft");
```

修改游戏目录为当前目录下的`.minecraft`