---
layout: post
title: "Minecraft Fabric Modding Toturial #3 To Write Some Simple Code"
date: 2024-05-03T15:32:28+08:00
---

## Preface

In the previous article, we have known the mod and project structure, in this article, we will write some simple code.

## Modify the Mod Information

First, we need delete some useless file.

### Delete the `client` folder

We have to modify other files after delete the `client` folder.

- Delete the `client` in the `gradle.build` file, because we don't need to split the client and server, so we can delete the `loom` block in the `gradle.build` file.

```patch 
-loom {
-    splitEnvironmentSourceSets()
-
-       mods {
-               "modid" {
-                       sourceSet sourceSets.main
-                       sourceSet sourceSets.client
-               }
-       }
-
-}
-
 dependencies {
        // To change the versions see the gradle.properties file
        minecraft "com.mojang:minecraft:${project.minecraft_version}"
```

- Delete the `client` in the `src/main/resources/fabric.mod.json` file.

```patch
        "entrypoints": {
                "main": [
                        "com.example.ExampleMod"
-               ],
-               "client": [
-                       "com.example.ExampleModClient"
                ]
        },
        "mixins": [
-               "modid.mixins.json",
-               {
-                       "config": "modid.client.mixins.json",
-                       "environment": "client"
-               }
+               "modid.mixins.json"
        ],
        "depends": {
                "fabricloader": ">=0.15.10",
```

### Modify the `fabric.mod.json` file

- Change `id`, `name`, `description`, `authors`, `icon` in the `fabric.mod.json` file.

```patch
        "schemaVersion": 1,
-       "id": "modid",
+       "id": "awesome",
        "version": "${version}",
-       "name": "Example mod",
-       "description": "This is an example description! Tell everyone what your mod is about!",
+       "name": "Awesome",
+       "description": "This is an awesome mod!",
        "authors": [
-               "Me!"
+               "Enaium"
        ],
        "contact": {
                "homepage": "https://fabricmc.net/",
                "sources": "https://github.com/FabricMC/fabric-example-mod"
        },
        "license": "CC0-1.0",
-       "icon": "assets/modid/icon.png",
+       "icon": "assets/awesome/icon.png",
        "environment": "*",
        "entrypoints": {
                "main": [
@@ -20,7 +20,7 @@
                ]
        },
        "mixins": [
-               "modid.mixins.json"
+               "awesome.mixins.json"
        ],
        "depends": {
                "fabricloader": ">=0.15.10",
```

- Rename the `modid.mixins.json` file to `awesome.mixins.json`.

- Rename the `assets/modid` folder to `assets/awesome`.

- Change the log name in the `ExampleMod.java` file.

```patch
public class ExampleMod implements ModInitializer {
        // This logger is used to write text to the console and the log file.
        // It is considered best practice to use your mod id as the logger's name.
        // That way, it's clear which mod wrote info, warnings, and errors.
-    public static final Logger LOGGER = LoggerFactory.getLogger("modid");
+    public static final Logger LOGGER = LoggerFactory.getLogger("awesome");
 
        @Override
        public void onInitialize() {
```

## To Write Code

Add a log in the `ExampleMixin` class.

```patch
 package com.example.mixin;
 
+import com.example.ExampleMod;
 import net.minecraft.server.MinecraftServer;
 import org.spongepowered.asm.mixin.Mixin;
 import org.spongepowered.asm.mixin.injection.At;
@@ -11,5 +12,6 @@ public class ExampleMixin {
        @Inject(at = @At("HEAD"), method = "loadWorld")
        private void init(CallbackInfo info) {
                // This code is injected into the start of MinecraftServer.loadWorld()V
+               ExampleMod.LOGGER.info("Hello Mixin World!");
        }
 }
```

## Test

Run the project, and load a world, you will see the log in the console.

```
[16:43:56] [Server thread/INFO] (awesome) Hello Mixin World!
```