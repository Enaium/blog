---
layout: post
title: "Vue项目整合Cordova一键打包apk"
date: 2024-07-13T23:40:06+08:00
---

`Cordova`是一个依赖于`Node.js`的工具，它可以将`HTML`页面打包为`apk`等移动端应用，本篇文章将介绍如何使用`Cordova`将`Vue`项目打包为`apk`。

## 创建`Corodva`项目

首先，我们需要安装`Cordova`，可以使用`npm`进行安装：

```shell
npm install -g cordova
```

安装完成后，我们可以使用`cordova create`命令创建一个`Cordova`项目：

```shell
cordova create test
```

之后进入到`test`目录，添加到`Android`平台：

```shell
cd test
cordova platform add android
```

这样就可以测试`Cordova`项目了，可以使用`cordova build`进行打包。

在打包之前需要注意以下几点：

- 如果之前没有使用过`Java`编程语言，则需要安装`JDK`，并且版本需要大于等于`11`。

- 如果之前没有使用过`Gradle`，则需要安装`Gradle`，并配置它的`bin`目录到`PATH`环境变量。

- 如果之前没有开发过`Android`项目，则需要安装`Android Studio`，并下载`Android SDK`，然后配置`ANDROID_HOME`环境变量。

如果过程顺利，那么会在`app/build/outputs`目录下生成`apk`文件。

## 整合`Vue`项目

首先我们可以将`vue`和`cordova`当成一个子项目，之后统一打包。

这里我们可以将`cordova`项目中的`config.xml`文件复制到一个空白的`node`项目中，之后创建一个空白的`www`目录，之后添加`cordova`依赖，这样就可以直接使用`npx cordova add platform android`添加`android`平台，通过`npx cordova build`进行打包。

之后通过修改`vite.config.ts`配置文件，将`outputDir`修改为`www`目录，这样就可以将`vue`项目打包到`cordova`项目中。

```typescript
export default defineConfig({
  build: {
    outDir: "../cordova/www"
  }
})
```

这样`vue`和`cordova`就整合到一起了。


## 一键打包

我们可以通过配置根项目的`package.json`文件，添加`scripts`命令，这样就可以一键打包`apk`了。

```json
{
  "scripts": {
    "vue:dev": "cd packages/vue && yarn dev",
    "vue:build": "cd packages/vue && yarn build",
    "cordova:build": "cd packages/cordova && yarn build",
    "cordova:run": "cd packages/cordova && yarn run",
    "build": "yarn vue:build && yarn cordova:build"
  }
}
```
