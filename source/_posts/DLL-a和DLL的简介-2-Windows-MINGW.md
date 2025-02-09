---
title: DLL.a和DLL的简介 (2. Windows MINGW)
date: 2025-02-09 19:27:05
tags:
  - C/C++基础
  - 动态链接库
  - 静态链接库
categories:
  - C/C++基础
---
# DLL.a和DLL的简介 (2. Windows MINGW)
使用 CMake 来配置和生成 DLL 和 `.dll.a`（导入库）文件非常方便。以下是如何使用 CMake 创建一个动态链接库（DLL）和导入库（`.dll.a`），并在其他项目中链接它们的详细步骤。

## 1. 创建 DLL 工程

假设有一个动态链接库需要导出函数，并且源代码为 `mydll.cpp`。通过 CMake 来生成 DLL 和导入库。

### 1.1 目录结构

首先，设置工程目录结构：

```plaintext
/Project
  /src
    mydll.cpp
    mydll.h
  CMakeLists.txt
  /build
```

### 1.2 源代码 `mydll.cpp`

```cpp
#include <iostream>

// 使用 __declspec(dllexport) 导出函数
extern "C" __declspec(dllexport) void hello() {
    std::cout << "Hello from the DLL!" << std::endl;
}
```

### 1.3 `CMakeLists.txt` 文件

在 `Project` 根目录下创建 `CMakeLists.txt` 文件：

```cmake
# 设置 CMake 最低版本和工程名称
cmake_minimum_required(VERSION 3.10)
project(MyDLLProject)

# 设置输出路径（可选）
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)

# 指定动态库的源文件
add_library(mydll SHARED src/mydll.cpp)

# 设置导出符号的宏定义，确保 Windows 上导出正确
if (WIN32)
    target_compile_definitions(mydll PRIVATE MYDLL_EXPORTS)
endif()

# 指定库文件的输出名称（dll 文件和导入库）
set_target_properties(mydll PROPERTIES OUTPUT_NAME "mydll")

# 导出头文件供其他项目使用
target_include_directories(mydll PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/src)
```

在 `CMakeLists.txt` 中：
- `add_library(mydll SHARED src/mydll.cpp)`：告诉 CMake 创建一个共享库（动态库，DLL）。
- `set_target_properties(mydll PROPERTIES OUTPUT_NAME "mydll")`：设置输出的 DLL 文件名为 `mydll.dll`，导入库为 `libmydll.dll.a`。
- `target_include_directories(mydll PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/src)`：将头文件目录暴露给其他使用此库的项目。

### 1.4 生成 DLL 和 `.dll.a` 文件

在命令行中执行以下命令生成动态库：

```bash
cd build
cmake ..
cmake --build .
```

CMake 会在 `build/bin/` 目录下生成 `mydll.dll`，并在 `build/lib/` 目录下生成 `libmydll.dll.a`。

## 2. 在另一个项目中使用 DLL 和 `.dll.a`

现在，创建一个新项目来使用刚才生成的动态链接库 `mydll.dll` 和导入库 `libmydll.dll.a`。

### 2.1 目录结构

```plaintext
/ClientProject
  /src
    main.cpp
  CMakeLists.txt
  /build
```

#### 2.2. `main.cpp`

```cpp
#include "mydll.h"

extern "C" void hello();  // 导入 DLL 中的函数声明

int main() {
    hello();  // 调用 DLL 函数
    return 0;
}
```

#### 2.3. `CMakeLists.txt`

在 `ClientProject` 目录下创建 `CMakeLists.txt`：

```cmake
# 设置 CMake 最低版本和工程名称
cmake_minimum_required(VERSION 3.10)
project(ClientProject)

# 设置可执行文件
add_executable(clientApp src/main.cpp)

# 链接到动态库
# 假设之前生成的 mydll.dll.a 和头文件位于 ../Project/build
target_link_libraries(clientApp PRIVATE ${CMAKE_SOURCE_DIR}/../Project/build/lib/libmydll.dll.a)

# 包含头文件路径
target_include_directories(clientApp PRIVATE ${CMAKE_SOURCE_DIR}/../Project/src)
```

在这里：
- `target_link_libraries(clientApp PRIVATE ../Project/build/lib/libmydll.dll.a)`：指定要链接的导入库。
- `target_include_directories(clientApp PRIVATE ../Project/src)`：指定头文件的路径，以便编译器能找到 `mydll.h`。

#### 2.4. 生成并运行

```bash
cd build
cmake ..
cmake --build .
```

运行生成的可执行文件时，确保 `mydll.dll` 在可执行文件的同一目录下，或将其放在系统路径中。可以将 DLL 复制到 `build/bin` 目录中。

```bash
cp ../Project/build/bin/mydll.dll ./bin/
./bin/clientApp
```

运行后，程序应输出：

```plaintext
Hello from the DLL!
```

通过以上步骤：
1. 使用 CMake 创建了一个包含导入库（`.dll.a`）和 DLL 文件的动态库项目。
2. 使用了这个 DLL 库，并通过 `.dll.a` 在另一个项目中链接并调用了 DLL 中的函数。


# 文件路径
在使用动态链接库（DLL）时，`.a` 文件通常是 GCC 或 MinGW 编译器生成的**导入库**（import library），它的作用类似于 MSVC 生成的 `.lib` 文件。`.a` 文件用于在**编译时**告诉链接器如何解析 DLL 文件中的符号，而 DLL 文件是在**运行时**加载的。

因此，通常 `.a` 文件（导入库）应该放在 `lib` 目录，而 `.dll` 文件应该放在 `bin` 目录或可执行文件所在的目录。

### 具体说明：

1. **`.a` 文件（导入库）**：
   - 这是编译时用的文件，通常放在项目的 `lib` 目录中，因为编译器或链接器会从该目录中查找导入库。
   - `lib` 目录一般是专门存放静态库和导入库的地方。

2. **`.dll` 文件（动态链接库）**：
   - 这是运行时用的文件，通常放在 `bin` 目录中，或者可以将它与可执行文件一起放在同一个目录下。
   - `bin` 目录通常用来存放可执行文件和动态链接库，程序在运行时会从该目录中加载 DLL 文件。

### 示例目录结构：
```plaintext
/project
  /bin            # 运行时所需文件
    myExecutable.exe
    DLLSample.dll  # DLL 文件放在 bin 目录中
  /lib            # 编译时所需文件
    DLLSample.a    # 导入库放在 lib 目录中
  /include        # 头文件目录
    DLLSample.h    # 头文件放在 include 目录中
  /src            # 源代码
    main.cpp
  CMakeLists.txt
```

### CMake 配置示例：
如果使用 CMake 构建项目，可以这样配置：

```cmake
# 添加可执行文件
add_executable(myExecutable main.cpp)

# 链接导入库 (.a)
target_link_libraries(myExecutable PRIVATE ${CMAKE_SOURCE_DIR}/lib/DLLSample.a)

# 设置 DLL 文件的路径
set_target_properties(myExecutable PROPERTIES
    RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin
)

# 如果需要头文件路径
target_include_directories(myExecutable PRIVATE ${CMAKE_SOURCE_DIR}/include)
```

通过这种方式，CMake 会在编译时链接 `.a` 文件（导入库），并且生成的可执行文件在运行时从 `bin` 目录加载对应的 `.dll` 文件。

### 总结：
- **`.a` 文件** 放在 `lib` 目录，因为它是编译时需要的导入库。
- **`.dll` 文件** 放在 `bin` 目录或可执行文件的目录，因为它是运行时需要加载的动态库。