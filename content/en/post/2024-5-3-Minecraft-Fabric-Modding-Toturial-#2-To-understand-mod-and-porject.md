---
title: "Minecraft Fabric Modding Toturial #2 To Understand Mod and Porject"
date: 2024-05-03T03:33:38+08:00
---

## Preface

In the previous article we have completed the development environment for fabric modding, in this article we will introduce some basic information about mod and project.

## Directory Structure

Click the `Project` button in the upper left corner, you can see the directory, then we expand all the folders in `src`.

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

We can see that there are two folders under `src`, `client` and `main`, these two folders correspond to the client and server, they have their own `java` and `resources` folders, the `java` folder contains Java code, and the `resources` folder contains resource files.

## Mod Information

In the `resources` folder under the `main` folder, there is a `fabric.mod.json` file, which is the mod information file, we can modify the mod information here.

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
		"fabricloader": ">=0.15.10",
		"minecraft": "~1.20.6",
		"java": ">=21",
		"fabric-api": "*"
	},
	"suggests": {
		"another-mod": "*"
	}
}
```

### Basic Information

Mod information is stored in a JSON file, the basic information includes: `id`, `version`, `name`, `description`, `authors`, `contact`, `license`, `icon`, etc.

- `id`: The id of the mod, it is unique, and it is only lowercase letters, no spaces, and no special characters.

- `version`: The version of the mod.

- `name`: The name of the mod.

- `description`: The description of the mod.

- `authors`: The authors of the mod.

- `contact`: The contact information of the mod, including `homepage` and `sources`.

- `license`: The license of the mod.

- `icon`: The icon of the mod, the path is `assets/modid/icon.png`, `modid` is the id of the mod.

### Other Information

- `environment`: The environment of the mod, `*` means it can be used in any environment, `client` means it can only be used in the client environment, `server` means it can only be used in the server environment.

- `entrypoints`: The entrypoints of the mod, `main` means it can be used in any environment, `client` means it can only be used in the client environment, the values are the class names of the entrypoints.

- `mixins`: The mixin configuration of the mod, we will introduce it later.

- `depends`: The dependencies of the mod, `fabricloader` means the version of Fabric, `minecraft` means the version of Minecraft, `java` means the version of Java, `fabric-api` means the version of Fabric API.

- `suggests`: The suggested dependencies of the mod, it is the same as `depends`, but the dependencies here are not required.

## Project Information

We have introduced the mod information above, here we will introduce the project information, it is too complex, so we only introduce the key parts.

### `build.gradle`

```groovy
// The version of the mod
version = project.mod_version
// The package name of the mod
group = project.maven_group
```

```groovy
loom {
    // Split the source code of different environments
    splitEnvironmentSourceSets()

	mods {
        // The id of the mod
		"modid" {
            // Any environment
			sourceSet sourceSets.main
            // The client environment
			sourceSet sourceSets.client
		}
	}

}
```

```groovy
dependencies {
    // Game
	minecraft "com.mojang:minecraft:${project.minecraft_version}"
    // Yarn mappings
	mappings "net.fabricmc:yarn:${project.yarn_mappings}:v2"
    // Fabric loader
	modImplementation "net.fabricmc:fabric-loader:${project.loader_version}"
    // Fabric API
	modImplementation "net.fabricmc.fabric-api:fabric-api:${project.fabric_version}"
	
}
```

```groovy
java {
	// Package the source code
	withSourcesJar()

    // The compatibility of the source code
	sourceCompatibility = JavaVersion.VERSION_21
    // The compatibility of the target code
	targetCompatibility = JavaVersion.VERSION_21
}
```

### `gradle.properties`

This is configuration file of the build script, it includes some variables, `build.gradle` will use these variables.

```properties
# Done to increase the memory available to gradle.
org.gradle.jvmargs=-Xmx1G
org.gradle.parallel=true

minecraft_version=1.20.6
yarn_mappings=1.20.6+build.1
loader_version=0.15.10

mod_version=1.0.0
maven_group=com.example
archives_base_name=modid

fabric_version=0.97.8+1.20.6
```