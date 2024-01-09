---
title: "Minecraft Fabric模组开发教程#2 了解模组和项目"
date: 2024-01-09T21:18:20+08:00
---

## 前言

在上篇文章中我们已经完成了开发环境的配置，这篇文章我们将会介绍一下模组和项目的基本信息。

## 目录结构

点击左上角的`Project`按钮，可以看到项目的目录结构，之后我们展开`src`中的所有文件夹。

```
├─client
│  ├─java
│  │  └─com
│  │      └─example
│  │          │  ExampleModClient.java
│  │          │
│  │          └─mixin
│  │              └─client
│  │                      ExampleClientMixin.java
│  │
│  └─resources
│          modid.client.mixins.json
│
└─main
    ├─java
    │  └─com
    │      └─example
    │          │  ExampleMod.java
    │          │
    │          └─mixin
    │                  ExampleMixin.java
    │
    └─resources
        │  fabric.mod.json
        │  modid.mixins.json
        │
        └─assets
            └─modid
                    icon.png
```

我们可以看到`src`下有两个文件夹，`client`和`main`，这两个文件夹分别对应客户端和任意端，他们分别有自己的`java`和`resources`文件夹，`java`文件夹中存放的是 Java 代码，`resources`文件夹中存放的是资源文件。

## 模组信息

在`main`文件夹下的`resources`文件夹中有一个`fabric.mod.json`文件，这个文件就是模组的信息文件，我们可以在这里修改模组的信息。

```
{
	"schemaVersion": 1,
	"id": "modid",
	"version": "${version}",
	"name": "Example mod",
	"description": "This is an example description! Tell everyone what your mod is about!",
	"authors": [
		"Me!"
	],
	"contact": {
		"homepage": "https://fabricmc.net/",
		"sources": "https://github.com/FabricMC/fabric-example-mod"
	},
	"license": "CC0-1.0",
	"icon": "assets/modid/icon.png",
	"environment": "*",
	"entrypoints": {
		"main": [
			"com.example.ExampleMod"
		],
		"client": [
			"com.example.ExampleModClient"
		]
	},
	"mixins": [
		"modid.mixins.json",
		{
			"config": "modid.client.mixins.json",
			"environment": "client"
		}
	],
	"depends": {
		"fabricloader": ">=0.15.0",
		"minecraft": "~1.20.4",
		"java": ">=17",
		"fabric-api": "*"
	},
	"suggests": {
		"another-mod": "*"
	}
}
```

### 基本信息

模组的基本信息包括`id`、`version`、`name`、`description`、`authors`、`contact`、`license`、`icon`。

- `id`：模组的 ID，必须是小写字母不能包含空格和特殊字符。
- `version`：模组的版本。
- `name`：模组的名称。
- `description`：模组的描述。
- `authors`：模组的作者。
- `contact`：模组的联系方式，包括`homepage`和`sources`。
- `license`：模组的开源协议。
- `icon`：模组的图标，路径为`assets/modid/icon.png`，`modid`为模组的 ID。

### 其他信息

- `environment`：模组的运行环境，`*`代表所有环境，`client`代表客户端，`server`代表任意端。
- `entrypoints`：模组的入口点，`main`代表任意端，`client`代表客户端，它们的值为入口点的类名。
- `mixins`：模组的`mixin`配置，这里先不介绍，后面会详细介绍。
- `depends`：模组的依赖，`fabricloader`代表 Fabric 的版本，`minecraft`代表 Minecraft 的版本，`java`代表 Java 的版本，`fabric-api`代表 Fabric API 的版本。
- `suggests`：模组的建议依赖，和`depends`一样，只是这里的依赖不是必须的。

## 项目信息

上面已经介绍了模组的信息，这里我们来介绍一下项目的信息，这里由于太过复杂，所以只介绍关键的部分。

### `build.gradle`

```groovy
//模组版本
version = project.mod_version
//模组的包名
group = project.maven_group
```

```groovy
loom {
	//分开不同环境的源码
    splitEnvironmentSourceSets()

	mods {
		//模组ID
		"modid" {
			//任意端
			sourceSet sourceSets.main
			//客户端
			sourceSet sourceSets.client
		}
	}

}
```

```groovy
dependencies {
	//游戏
	minecraft "com.mojang:minecraft:${project.minecraft_version}"
	//混淆映射
	mappings "net.fabricmc:yarn:${project.yarn_mappings}:v2"
	//模组加载器
	modImplementation "net.fabricmc:fabric-loader:${project.loader_version}"
	//模组API
	modImplementation "net.fabricmc.fabric-api:fabric-api:${project.fabric_version}"
}
```

```groovy
java {
	//打包源码
	withSourcesJar()

	//源文件的JDK版本
	sourceCompatibility = JavaVersion.VERSION_17

	//编译产物的JDK版本
	targetCompatibility = JavaVersion.VERSION_17
}
```

### `gradle.properties`

这个是构建脚本的配置文件，里面包含了很多变量，上面介绍的`build.gradle`中的变量都是从这里读取的。

```properties
minecraft_version=1.20.4
yarn_mappings=1.20.4+build.1
loader_version=0.15.0

mod_version=1.0.0
maven_group=com.example
archives_base_name=modid

fabric_version=0.91.1+1.20.4
```