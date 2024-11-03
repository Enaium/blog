---
title: "DLL劫持并使用MinHook"
date: 2024-11-03T01:28:23+08:00
---

## 测试用例

首先我使用`CLion`写了一个简单的程序，这个程序会加载一个`dinput8.dll`，然后调用一个函数显示一段文字，然后等待用户按下任意键。这个程序的代码如下：

```cpp
#include<windows.h>
#include<iostream>


int display(const char *text) {
    std::cout << text << std::endl;
    std::cout << "Press any key to continue..." << std::endl;
    std::cin.ignore();
    return 0;
}

int main() {
    LoadLibrary("dinput8.dll");
    printf("Address: %p\n", display);
    display("Hello World!");
}
```

我们所做的就是 Hook 这个`display`函数，然后在这个函数被调用时，将这个文字改为另一个文字。之后将项目进行编译，别忘了在`CMakeLists.txt`中添加`set(CMAKE_EXE_LINKER_FLAGS "-static")`，这样我们就可以得到一个静态链接的可执行文件。之后我们就可以开始 Hook 这个函数了。

## DLL 劫持

首先我们需要了解一下 DLL 的加载机制，Windows 系统在加载 DLL 时，会按照一定的顺序搜索 DLL 文件，如果找到了就加载，如果没有找到就会报错。这个顺序是怎么样的呢？我们可以通过查看[官方文档](https://docs.microsoft.com/zh-cn/windows/win32/dlls/dynamic-link-library-search-order)来了解。简单来说就是首先搜索应用程序目录，然后搜索系统目录，最后搜索环境变量中指定的路径。所以我们可以使用这个机制来劫持 DLL。

## 包装 DLL

上面我们知道了 DLL 的加载机制，那么我们就可以知道如何让程序加载我们自己的 DLL，加载我们的 DLL 后，我们需要让这个 DLL 也拥有原 DLL 的功能，这里我们需要对原 DLL 进行包装。我们可以使用[wrap_dll](https://github.com/mavenlin/wrap_dll)这个工具来包装 DLL。这是个`Python`脚本，所以你必须安装了`Python`，之后这个脚本还需要`dumpbin.exe`和`undname.exe`这两个工具，这两个工具是 Visual Studio 自带的，所以你需要安装了`Visual Studio`，并且需要将`C:\Program Files\Microsoft Visual Studio\2022\Community\VC\Tools\MSVC\14.40.33807\bin\Hostx64\x64`设置到环境变量中。之后你就可以使用这个脚本了。

```shell
python .\wrap_dll.py "C:\Windows\System32\dinput8.dll"
```

运行完成这个条命令后，你会得到一个`dinput8`项目，这个就是我们包装的 DLL。我们可以先把项目中的`empty.h`和`hook_macro.h`给删掉，这两个文件对我们没有用。

## MinHook

MinHook 是一个轻量级的钩子库，可以用来 Hook 函数。我们可以在[minhook](https://github.com/TsudaKageyu/minhook)下载到这个库。下载完成后我们在 dinput8 这个项目中新建一个文件夹`MinHook`，然后将 MinHook 的`include`文件夹和`src`文件夹复制到这个文件夹中。之后我们在`CMakeLists.txt`中添加这个库。

```cmake
ADD_LIBRARY(
    dinput8
    SHARED
    dinput8_asm.asm
    dinput8.cpp
    dinput8.def
    MinHook/include/MinHook.h
    MinHook/src/hde/hde32.c
    MinHook/src/hde/hde32.h
    MinHook/src/hde/hde64.c
    MinHook/src/hde/hde64.h
    MinHook/src/hde/pstdint.h
    MinHook/src/hde/table32.h
    MinHook/src/hde/table64.h
    MinHook/src/buffer.c
    MinHook/src/buffer.h
    MinHook/src/hook.c
    MinHook/src/trampoline.c
    MinHook/src/trampoline.h
)
```

之后我们就可以开始 Hook 这个函数了。

## Hook 函数

首先我们需要再项目中执行`cmake .`，之后会生成一个`Visual Studio`的项目，我们使用`Visual Studio`打开这个项目，这里需要注意一下，需要通过打开解决方案的方式打开这个项目，不要直接打开这个项目文件夹。之后我们打开`dinput8.cpp`，删掉`_hook_setup`这个函数，接着我们需要将加载库的地址改为系统的`LoadLibrary`函数，这样我们就可以加载原 DLL 了。之后我们就可以开始 Hook 这个函数了。

```cpp
#include <windows.h>
#include <stdio.h>

HINSTANCE mHinst = 0, mHinstDLL = 0;

extern "C" UINT_PTR mProcs[6] = {0};

LPCSTR mImportNames[] = {
  "DirectInput8Create",
  "DllCanUnloadNow",
  "DllGetClassObject",
  "DllRegisterServer",
  "DllUnregisterServer",
  "GetdfDIJoystick",
};

#ifndef _DEBUG
inline void log_info(const char* info) {
}
#else
FILE* debug;
inline void log_info(const char* info) {
  fprintf(debug, "%s\n", info);
  fflush(debug);
}
#endif

BOOL WINAPI DllMain(HINSTANCE hinstDLL, DWORD fdwReason, LPVOID lpvReserved) {
  mHinst = hinstDLL;
  if (fdwReason == DLL_PROCESS_ATTACH) {
    mHinstDLL = LoadLibrary("C:/Windows/System32/dinput8.dll");
    if (!mHinstDLL) {
      return FALSE;
    }
    for (int i = 0; i < 6; ++i) {
      mProcs[i] = (UINT_PTR)GetProcAddress(mHinstDLL, mImportNames[i]);
    }

#ifdef _DEBUG
    debug = fopen("./debug.log", "a");
#endif
  } else if (fdwReason == DLL_PROCESS_DETACH) {
#ifdef _DEBUG
    fclose(debug);
#endif
    FreeLibrary(mHinstDLL);
  }
  return TRUE;
}

extern "C" void DirectInput8Create_wrapper();
extern "C" void DllCanUnloadNow_wrapper();
extern "C" void DllGetClassObject_wrapper();
extern "C" void DllRegisterServer_wrapper();
extern "C" void DllUnregisterServer_wrapper();
extern "C" void GetdfDIJoystick_wrapper();
```

接着我们可以在当`fdwReason`等于`DLL_PROCESS_ATTACH`时，也就是当 DLL 被加载时，我们进行 Hook。

首先我们需要编写一个原函数的函数指针，这个函数指针的类型需要和原函数的类型一样，这里我们使用`typedef`来定义这个函数指针。

```cpp
typedef int (*display_t)(const char*);
```

之后我们需要定义一个函数指针，这个函数指针用来指向我们 Hook 后的函数。

```cpp
display_t display = nullptr;
```

之后我们创建一个Hook函数，这个函数的参数和返回值都需要和原函数一样。

```cpp
int display_hook(const char *text) {
    return display("Hello MinHook!");
}
```

最后我们使用`MinHook`来`Hook`这个函数。

首先我们需要初始化`MinHook`，之后使用`MH_CreateHook`来创建一个 Hook，传入原函数的地址，Hook 函数的地址，和一个函数指针的指针，之后我们就可以使用`MH_EnableHook`来启用这个 Hook 了。

```cpp
MH_Initialize();
MH_CreateHook((LPVOID)0x00007ff62fb51634, &display_hook, reinterpret_cast<LPVOID*>(&display));
MH_EnableHook(nullptr);
```

这里的`0x00007ff62fb51634`是我们运行这个测试程序后得到的`display`函数的地址，你可以通过打印这个函数的地址来得到这个地址。除了这个方法我们还可以使用`x64dbg`这个工具来得到这个地址。

我们打开`x64dbg`通过符号切换到这个程序，之后使用字符串搜索`Hello World!`，之后我们找到`call`这个指令，然后我们就可以得到这个函数的地址了。

![20241103121643](https://s2.loli.net/2024/11/03/caPVHrGoNl4qOTF.png)

![20241103121851](https://s2.loli.net/2024/11/03/nsCbup7EdI95JAr.png)

![20241103122021](https://s2.loli.net/2024/11/03/aNA6lk9h7beLTfd.png)

之后编译我们的项目，然后将生成的 DLL 放到我们的测试程序的目录下，之后我们就可以运行这个程序了。

![20241103122428](https://s2.loli.net/2024/11/03/oSGywfmebITKju7.png)

项目地址: [dll-hijack-minhook](https://github.com/Enaium/dll-hijack-minhook)