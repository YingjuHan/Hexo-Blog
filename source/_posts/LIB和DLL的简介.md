---
title: LIB和DLL的简介 (1. Windows MSVC)
date: 2025-02-09 18:25:01
tags:
  - C/C++基础
  - 动态链接库
  - 静态链接库
categories:
  - C/C++基础
---

# LIB和DLL的简介

## 1. 动态链接库与静态链接库：
1. 动态链接库dynamic link library: LIB包含了函数所在的DLL文件和文件中函数位置的信息（入口），代码由运行时加载在进程空间中的DLL提供
2. 静态链接库static link library: LIB包含函数代码本身，在编译时直接将代码加入程序当中

## 2. 动态链接方式与静态链接方式：
1. 动态链接使用动态链接库，允许可执行模块（.dll文件或.exe文件）仅包含在运行时定位DLL函数的可执行代码所需的信息
2. 静态链接使用静态链接库，链接器从静态链接库LIB获取所有被引用函数，并将库同代码一起放到可执行文件中

## 3. lib和dll的区别如下：
1. lib是编译时用到的，dll是运行时用到的。如果要完成源代码的编译，只需要lib；如果要使动态链接的程序运行起来，只需要dll;
2. 如果有dll文件，那么lib一般是一些索引信息，记录了dll中函数的入口和位置，dll中是函数的具体内容；如果只有lib文件，那么这个lib文件是静态编译出来的，索引和实现都在其中。使用静态编译的lib文件，在运行程序时不需要再挂动态库，缺点是导致应用程序比较大，而且失去了动态库的灵活性，发布新版本时要发布新的应用程序才行;
3. 动态链接的情况下，有两个文件：一个是LIB文件，一个是DLL文件。LIB包含被DLL导出的函数名称和位置，DLL包含实际的函数和数据，应用程序使用LIB文件链接到DLL文件。在应用程序的可执行文件中，存放的不是被调用的函数代码，而是DLL中相应函数代码的地址，从而节省了内存资源。DLL和LIB文件必须随应用程序一起发行，否则应用程序会产生错误。如果不想用lib文件或者没有lib文件，可以用WIN32 API函数LoadLibrary、GetProcAddress装载;
4. 静态链接的情况下，只有一个LIB文件。在编译链接时，WIN32 Link 将会从LIB文件中寻找相应的函数代码，将它们直接链接到生成的可执行文件或生成的库文件中。

## 4. 使用lib需要注意的两个文件
1. H文件：头文件，包含了函数的声明，包含LIB中导出类、符号原型或数据结构,应用程序调用LIB中的函数或访问数据结构时，需要将对应的头文件包含在应用程序中;
2. LIB文件：库文件，包含函数的实现，应用程序链接LIB文件时，需要将LIB文件包含在应用程序的链接选项中;

## 5. 使用dll需要注意的三个文件
1. h头文件，包含dll中说明输出的类或符号原型或数据结构的.h文件。应用程序调用dll时，需要将该文件包含入应用程序的源文件中。
2. LIB文件，是dll在编译、链接成功之后生成的文件，作用是当其他应用程序调用dll时，需要将该文件引入应用程序，否则产生错误。如果不想用lib文件或者没有lib文件，可以用WIN32 API函数LoadLibrary、GetProcAddress装载。
3. DLL文件，真正的可执行文件，开发成功后的应用程序在发布时，只需要有可执行文件和DLL文件，并不需要LIB文件和H头文件。

## 6. 使用LIB的方法
静态lib中，一个lib文件实际上是任意个obj文件的集合，obj文件是cpp文件编译生成的。在编译这种静态库工程时，根本不会遇到链接错误；即使有错，也只会在使用这个LIB的可执行文件或者库工程里暴露出来。

### 6.1 创建静态库(MyLib.lib)
```cmake
# 设置最低CMake版本
cmake_minimum_required(VERSION 3.10)

# 定义工程名称
project(MyLib)

# 添加源文件
add_library(MyLib STATIC test.cpp)

# 指定库的输出路径
set(LIBRARY_OUTPUT_PATH ${CMAKE_BINARY_DIR}/lib)
```
该文件将生成一个静态库 MyLib.lib，并将其放置在 CMake 构建目录下的 lib 文件夹中。

### 6.2 创建目标工程, 并链接静态库
- 法1：将 MyLib.lib 作为外部库添加到目标工程
文件夹结构
```
/projectA
  CMakeLists.txt
  main.cpp
  /lib
    MyLib.lib
```
projectA 是使用静态库 MyLib.lib 的目标工程，需要将 MyLib.lib 作为外部库链接到projectA中。
```cmake
# 设置最低CMake版本
cmake_minimum_required(VERSION 3.10)

# 定义工程名称
project(projectA)

# 添加源文件
add_executable(projectA main.cpp)

# 添加静态库的路径
link_directories(${CMAKE_SOURCE_DIR}/lib)

# 将静态库链接到工程
target_link_libraries(projectA PRIVATE MyLib)

# 添加头文件路径
include_directories(${CMAKE_SOURCE_DIR}/path/to/lib/include)
```
这样，CMake 会找到 MyLib.lib 并将其链接到 projectA，其中：
link_directories() 用于告诉 CMake 该去哪里查找库文件（例如 MyLib.lib）。
target_link_libraries() 用于将库 MyLib.lib 链接到目标 projectA。

- 法2: 直接指定静态库的路径
```cmake
# 设置最低CMake版本
cmake_minimum_required(VERSION 3.10)

# 定义工程名称
project(projectA)

# 添加源文件
add_executable(projectA main.cpp)

# 链接静态库
target_link_libraries(projectA PRIVATE ${CMAKE_SOURCE_DIR}/lib/Lib.lib)

# 添加头文件路径
include_directories(${CMAKE_SOURCE_DIR}/path/to/lib/include)
```
这种方式省略了 link_directories，直接通过库的绝对路径进行链接。

## 7. 使用DLL的方法
使用动态链接中的LIB，不是obj文件的集合，即里面不会有实际的实现，它只是提供动态链接到DLL所需要的信息，这种LIB可以在编译一个DLL工程时由编译器生成。

### 7.1 创建动态库(MyDLL.dll)
```cmake
# 设置最低CMake版本
cmake_minimum_required(VERSION 3.10)
# 定义工程名称
project(MyDLL)
# 添加源文件
add_library(MyDLL SHARED test.cpp)
# 指定库的输出路径
set(LIBRARY_OUTPUT_PATH ${CMAKE_BINARY_DIR}/lib)
```
该文件将生成一个动态库 MyDLL.dll，并将其放置在 CMake 构建目录下的 lib 文件夹中。
### 7.2 创建目标工程, 并链接动态库
#### 7.2.1 隐式链接
第一种方法是：通过project->link->Object/Library Module中加入.lib文件（或者在源代码中加入指令#pragma comment(lib, “Lib.lib”)），并将.dll文件置入工程所在目录，然后添加对应的.h头文件。
```C++
#include "stdafx.h"
#include "myLib.h"

#pragma comment(lib, "MyLib.lib")    //你也可以在项目属性中设置库的链接

int main()
{
        TestDLL(123);   //dll中的函数，在MyLib.h中声明
        return(1);
}
```

#### 7.2.2 显式链接
需要函数指针和WIN32 API函数LoadLibrary、GetProcAddress装载，使用这种载入方法，不需要.lib文件和.h头文件，只需要.dll文件即可（将.dll文件置入工程目录中）。
```C++
#include <iostream>
#include <windows.h>         //使用函数和某些特殊变量
typedef void (*DLLFunc)(int);
int main()
{
        DLLFunc dllFunc;
        HINSTANCE hInstLibrary = LoadLibrary("MyLib.dll");

        if (hInstLibrary == NULL)
        {
          FreeLibrary(hInstLibrary);
        }
        dllFunc = (DLLFunc)GetProcAddress(hInstLibrary, "TestDLL");
        if (dllFunc == NULL)
        {
          FreeLibrary(hInstLibrary);
        }
        dllFunc(123);
        std::cin.get();
        FreeLibrary(hInstLibrary);
        return(1);
}
```
LoadLibrary函数利用一个名称作为参数，获得DLL的实例（HINSTANCE类型是实例的句柄），通常调用该函数后需要查看一下函数返回是否成功，如果不成功则返回NULL（句柄无效），此时调用函数FreeLibrary释放DLL获得的内存。
GetProcAddress函数利用DLL的句柄和函数的名称作为参数，返回相应的函数指针，同时必须使用强转；判断函数指针是否为NULL，如果是则调用函数FreeLibrary释放DLL获得的内存。此后，可以使用函数指针来调用实际的函数。
最后要记得使用FreeLibrary函数释放内存。

**应用程序如何找到DLL文件？**
使用LoadLibrary显式链接，那么在函数的参数中可以指定DLL文件的完整路径；如果不指定路径，或者进行隐式链接，Windows将遵循下面的搜索顺序来定位DLL：
（1）包含EXE文件的目录
（2）工程目录
（3）Windows系统目录
（4）Windows目录
（5）列在Path环境变量中的一系列目录