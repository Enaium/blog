---
title: "Minecraft启动器开发#6下载Native"
date: 2021-05-16T13:49:00+0800
categories: java
---

```java
if (downloads.has("classifiers")) {
                var classifiers = downloads.get("classifiers").getAsJsonObject();
                var nativeName = "natives-linux";
                var osName = System.getProperty("os.name").toLowerCase(Locale.ROOT);
                if (osName.contains("win")) {
                    nativeName = "natives-windows";
                    if (!classifiers.has(nativeName)) {
                        nativeName = "natives-windows-64";
                    }
                } else if (osName.contains("mac")) {
                    nativeName = "natives-osx";
                    if (!classifiers.has(nativeName)) {
                        nativeName = "natives-macos";
                    }
                }

                if (!classifiers.has("nativeName")) {
                    continue;
                }

                var path = new File(libraryDir, classifiers.get(nativeName).getAsJsonObject().get("path").getAsString());
                var url = classifiers.get(nativeName).getAsJsonObject().get("url").getAsString();

                if (!path.exists()) {
                    FileUtils.writeByteArrayToFile(path, IOUtils.toByteArray(new URL(url)));
                }
    }
```

下载库的时候下载Native，`osx`在以后的版本会变成`macos`需要再次判断，windows还需要判断系统架构

```java
if (!path.exists()) {
    FileUtils.writeByteArrayToFile(path, IOUtils.toByteArray(new URL(url)));
} else {
    var jarFile = new JarFile(path);
    var entries = jarFile.entries();
    while (entries.hasMoreElements()) {
        var jarEntry = entries.nextElement();
        if (jarEntry.isDirectory() || jarEntry.getName().contains("META-INF")) {
            continue;
        }

        var inputStream = jarFile.getInputStream(jarEntry);
        FileUtils.writeByteArrayToFile(new File(nativeDir, jarEntry.getName()), IOUtils.toByteArray(inputStream));
        inputStream.close();
    }
    jarFile.close();
}
```

如果文件存在就读取，然后解压
