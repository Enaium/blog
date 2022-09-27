---
title: "Minecraft启动器开发#5下载资源"
date: 2021-05-16T11:16:00+0800
categories: java
---

```java
var assetsDir = new File(gameDir, "assets");
```

将资源目录改为`.minecraft`的assets目录

```java
var indexDir = new File(assetsDir, "indexes");
if (!indexDir.exists()) {
    indexDir.mkdir();
}

var objectDir = new File(assetsDir, "objects");
if (!objectDir.exists()) {
    objectDir.mkdir();
}
```

创建`indexes`和`objects`目录

```java
public class AssetObject {
    private final String hash;
    private final long size;

    public AssetObject(String hash, long size) {
        this.hash = hash;
        this.size = size;
    }

    public String getHash() {
        return hash;
    }

    public long getSize() {
        return size;
    }
}
```

创建`AssetObject`类

```java
var indexContent = IOUtils.toString(new URL(gameJson.get("assetIndex").getAsJsonObject().get("url").getAsString()), StandardCharsets.UTF_8);
var indexObject = gson.fromJson(indexContent, JsonObject.class);
for (Map.Entry<String, JsonElement> objects : indexObject.get("objects").getAsJsonObject().entrySet()) {
    var assetObject = gson.fromJson(objects.getValue(), AssetObject.class);
    var objectIndexDir = new File(objectDir, assetObject.getHash().substring(0, 2));
    if (!objectIndexDir.exists()) {
        objectIndexDir.mkdir();
    }
    var objectIndexFile = new File(objectIndexDir, assetObject.getHash());

    if (!objectIndexFile.exists()) {
        FileUtils.writeByteArrayToFile(objectIndexFile, IOUtils.toByteArray(new URL("https://resources.download.minecraft.net/" + assetObject.getHash().substring(0, 2) + "/" + assetObject.getHash())));
    }
}
```

遍历出所有资源，下载，路径hash前两位/hash全部，连接为`https://resources.download.minecraft.net/`

之后把`assetIndex`改为`gameVersion`

```java
FileUtils.writeStringToFile(new File(indexDir, gameVersion + ".json"), indexContent, StandardCharsets.UTF_8);
```

将`assetsIndex`的内容保存到`indexes`
