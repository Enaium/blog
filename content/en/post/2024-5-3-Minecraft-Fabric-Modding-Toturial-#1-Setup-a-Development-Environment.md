---
layout: post
title: "Minecraft Fabric Modding Toturial #1 Setup a Development Environment"
date: 2024-05-03T02:21:18+08:00
---

## Install JDK

Since the latest(1.20.6) Minecraft only supports JDK21, so we need to install JDK21, here I choose to download [Libreica OpenJDK 21](https://bell-sw.com/pages/downloads/#/java-21-lts).

![20240503030829](https://s2.loli.net/2024/05/03/cuFKqWhePMUIE5V.png)

Choose `Full JDK` and click the file with the suffix `msi` to download if you are using Windows, double-click to install after downloading.

## Install Intellij IDEA

Here I choose to download [Intellij IDEA Community](https://www.jetbrains.com/idea/download), choose the `Community` version.

![20240503030858](https://s2.loli.net/2024/05/03/YBP3HGiyrsNOmXd.png)


Double-click to install after downloading if you are using Windows.

## Download Project Template

Here you need to download the template from the Fabric GitHub repository, [click here](https://github.com/FabricMC/fabric-example-mod) to enter the repository, click the `Code` button, and choose `Download ZIP` to download.

![20240503030917](https://s2.loli.net/2024/05/03/ZrxAbVwvR38F172.png)

## Run the Project

After downloading, unzip it, open `Intellij IDEA`, click `Open`, select the unzipped folder, click `OK`, and finally click `Trust Project`.

This way, the project has been imported into `Intellij IDEA`, just wait for `Intellij IDEA` to load, if it is the first time to develop a Fabric mod, it may need to download some dependencies, this process may be relatively slow, please be patient.

Click the button with a `Build` icon in the lower-left corner of the screen, and the when `BUILD SUCCESSFUL` printed on the right side, it means the project has been built successfully and can be run.

Finally, click the button with a `Gradle` icon in the upper right corner, expand it and find `fabric` under `Tasks`, click `runClient`, wait for the run to complete.

![20240503032959](https://s2.loli.net/2024/05/03/GC82aWVHoY1iqyK.png)