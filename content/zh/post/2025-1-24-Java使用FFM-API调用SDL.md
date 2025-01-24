---
title: "Java使用FFM API调用SDL"
date: 2025-01-24T20:41:52+08:00
---

首先我们需要创建一个`Gradle`项目，之后设置项目的`JDK`版本，设置为`22`及以上版本。

```kotlin
plugins {
    kotlin("jvm") version "2.1.0"
}

group = "cn.enaium"
version = "1.0-SNAPSHOT"

repositories {
    mavenCentral()
}

dependencies {
    testImplementation(kotlin("test"))
}

tasks.test {
    useJUnitPlatform()
}
kotlin {
    jvmToolchain(23)
}
```

接着我们在当前目录初始化`git`仓库，之后需要添加一个子模块。、

```
git init
git submodule add git@github.com:libsdl-org/SDL.git SDL
```

之后编写生成接口的脚本，在这之前你必须安装[CMake](https://cmake.org/)和[`jextract`](https://jdk.java.net/jextract/)到环境变量中。

```powershell
$sdl_path = "SDL"
mkdir "$sdl_path/build";cmake -DCMAKE_BUILD_TYPE=Release .. -B "$sdl_path/build";cmake --build . --config Release --parallel -B "$sdl_path/build"
jextract --include-dir "$sdl_path/include" --dump-includes "$sdl_path/build/includes.txt" "$sdl_path/include/SDL3/SDL.h"
jextract --include-dir "$sdl_path/include" --output src/main/java --target-package org.libsdl --library SDL3 --use-system-load-library "@$sdl_path/build/includes.txt" "$sdl_path/include/SDL3/SDL.h"
```

首先是使用`CMake`编译`SDL`，之后使用`jextract`生成`Java`接口，之后运行脚本，这样就会在`src/main/java`生成`SDL3`的接口。

接着我们回到`build.gradle.kts`，添加`application`插件，之后将编译好的路径添加到启动参数中。

```kotlin
plugins {
    application
}

application {
    mainClass = "MainKt"
    applicationDefaultJvmArgs = listOf("-Djava.library.path=SDL/build/Release", "--enable-native-access=ALL-UNNAMED")
}
```

之后就可以调用`SDL`的接口了。

```kotlin
import org.libsdl.*
import org.libsdl.SDL_h_1.*
import java.lang.foreign.Arena

/**
 * @author Enaium
 */
fun main() {
    Arena.ofConfined().use {
        val init = SDL_h_2.SDL_Init(SDL_INIT_VIDEO() and SDL_INIT_EVENTS())

        if (!init) {
            println("SDL_Init Error: ${SDL_h_3.SDL_GetError()}")
            return
        }

        val windowPtr = it.allocate(C_POINTER)
        val rendererPtr = it.allocate(C_POINTER)

        SDL_CreateWindowAndRenderer(it.allocateFrom("Hello World"), 640, 480, 0, windowPtr, rendererPtr)

        val window = windowPtr.get(C_POINTER, 0)
        val renderer = rendererPtr.get(C_POINTER, 0)

        val rect = SDL_FRect.allocate(it)
        SDL_FRect.x(rect, 100f)
        SDL_FRect.y(rect, 100f)
        SDL_FRect.w(rect, 440f)
        SDL_FRect.h(rect, 280f)


        val event = SDL_Event.allocate(it)
        var quit = false
        while (!quit) {
            while (SDL_PollEvent(event)) {
                when (SDL_Event.type(event)) {
                    SDL_EVENT_QUIT() -> {
                        quit = true
                    }

                    SDL_EVENT_KEY_DOWN() -> {
                        when (SDL_KeyboardEvent.key(SDL_Event.key(event))) {
                            SDLK_UP() -> {
                                SDL_FRect.y(rect, SDL_FRect.y(rect) - 5)
                            }

                            SDLK_DOWN() -> {
                                SDL_FRect.y(rect, SDL_FRect.y(rect) + 5)
                            }

                            SDLK_LEFT() -> {
                                SDL_FRect.x(rect, SDL_FRect.x(rect) - 5)
                            }

                            SDLK_RIGHT() -> {
                                SDL_FRect.x(rect, SDL_FRect.x(rect) + 5)
                            }
                        }
                    }
                }
            }

            SDL_h_2.SDL_SetRenderDrawColor(renderer, 33.toByte(), 33.toByte(), 33.toByte(), 255.toByte())
            SDL_h_2.SDL_RenderClear(renderer)

            SDL_h_2.SDL_SetRenderDrawColor(renderer, 0.toByte(), 0.toByte(), 255.toByte(), 255.toByte())

            SDL_h_2.SDL_RenderFillRect(renderer, rect)

            SDL_h_2.SDL_RenderPresent(renderer)
        }

        SDL_h_2.SDL_DestroyRenderer(renderer)
        SDL_h_3.SDL_DestroyWindow(window)
    }
}
```

首先这里创建了一个窗口和渲染器，还渲染了一个矩形。之后做了事件处理，关闭的时候跳出循环，之后销毁窗口和渲染器。按下键盘上下左右键可以移动矩形。

之后调用`./gradlew run`就可以运行程序了。

![](https://s2.loli.net/2025/01/24/5J7FOMwGSIeQVz4.gif)