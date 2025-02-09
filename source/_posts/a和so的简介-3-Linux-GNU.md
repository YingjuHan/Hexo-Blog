---
title: .a和so的简介 (3. Linux GNU)
date: 2025-02-09 20:11:41
tags:
  - C/C++基础
  - 动态链接库
  - 静态链接库
categories:
  - C/C++基础
---

以下是如何使用 CMake 来生成静态库（`.a` 文件）和动态库（`.so` 文件）的详细教程，结合如何在其他项目中使用这些库。

### 1. 创建静态库和动态库

首先，创建一个包含库代码的项目。假设库代码位于 `MyLibrary` 目录下，源代码文件为 `mylib.cpp`，头文件为 `mylib.h`。

#### 1.1. 目录结构

```plaintext
/MyLibrary
  /src
    mylib.cpp
    mylib.h
  CMakeLists.txt
  /build
```

#### 1.2. 库的源代码 `mylib.cpp`

```cpp
// mylib.cpp
#include "mylib.h"
#include <iostream>

void hello() {
    std::cout << "Hello from MyLibrary!" << std::endl;
}
```

#### 1.3. 库的头文件 `mylib.h`

```cpp
// mylib.h
#ifndef MYLIB_H
#define MYLIB_H

void hello();

#endif // MYLIB_H
```

#### 1.4. `CMakeLists.txt` 文件

在 `MyLibrary` 根目录下创建 `CMakeLists.txt` 文件：

```cmake
# 设置 CMake 最低版本和项目名称
cmake_minimum_required(VERSION 3.10)
project(MyLibrary)

# 选项：用户可以选择构建静态库或动态库
option(BUILD_STATIC_LIBS "Build static libraries" OFF)

# 设置输出目录
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)

# 根据选项决定是生成静态库还是动态库
if(BUILD_STATIC_LIBS)
    add_library(mylib STATIC src/mylib.cpp)
else()
    add_library(mylib SHARED src/mylib.cpp)
endif()

# 包含头文件目录
target_include_directories(mylib PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/src)
```

#### 1.5. 生成库

在命令行中运行以下命令来生成静态库或动态库：

1. **生成动态库**（默认）：

```bash
cd build
cmake ..
cmake --build .
```

CMake 会在 `build/lib/` 目录下生成 `libmylib.so` 文件。

2. **生成静态库**：

```bash
cd build
cmake -DBUILD_STATIC_LIBS=ON ..
cmake --build .
```

CMake 会在 `build/lib/` 目录下生成 `libmylib.a` 文件。

### 2. 在另一个项目中使用库

假设已经生成了 `libmylib.a`（静态库）或 `libmylib.so`（动态库），现在创建一个新的项目来使用这个库。

#### 2.1. 目录结构

```plaintext
/ClientProject
  /src
    main.cpp
  CMakeLists.txt
  /build
```

#### 2.2. `main.cpp` 文件

```cpp
// main.cpp
#include "mylib.h"

int main() {
    hello();  // 调用 MyLibrary 中的函数
    return 0;
}
```

#### 2.3. `CMakeLists.txt` 文件

```cmake
# 设置 CMake 最低版本和项目名称
cmake_minimum_required(VERSION 3.10)
project(ClientProject)

# 创建可执行文件
add_executable(clientApp src/main.cpp)

# 链接 MyLibrary 静态库或动态库
# 假设 MyLibrary 的生成库位于 ../MyLibrary/build/lib
target_link_libraries(clientApp PRIVATE ${CMAKE_SOURCE_DIR}/../MyLibrary/build/lib/libmylib.a) # 静态库链接
# target_link_libraries(clientApp PRIVATE ${CMAKE_SOURCE_DIR}/../MyLibrary/build/lib/libmylib.so) # 动态库链接

# 包含 MyLibrary 的头文件
target_include_directories(clientApp PRIVATE ${CMAKE_SOURCE_DIR}/../MyLibrary/src)
```

#### 2.4. 生成并运行

进入 `ClientProject/build` 目录，运行以下命令生成并运行可执行文件：

```bash
cd build
cmake ..
cmake --build .
```

如果使用静态库，生成的可执行文件可以直接运行：

```bash
./clientApp
```

如果使用动态库，需要确保 `.so` 文件在运行时能被找到。可以将动态库拷贝到可执行文件目录，或者设置 `LD_LIBRARY_PATH` 环境变量：

```bash
export LD_LIBRARY_PATH=/path/to/your/lib:$LD_LIBRARY_PATH
./clientApp
```

### 3. CMake 管理 `.a` 和 `.so` 的自动选择

还可以使用 CMake 的 `find_library()` 函数，自动选择静态库或动态库：

#### 修改 `ClientProject` 的 `CMakeLists.txt`

```cmake
# 设置 CMake 最低版本和项目名称
cmake_minimum_required(VERSION 3.10)
project(ClientProject)

# 查找 MyLibrary 库，自动选择静态库或动态库
find_library(MYLIBRARY_LIB NAMES mylib PATHS ${CMAKE_SOURCE_DIR}/../MyLibrary/build/lib)

# 创建可执行文件
add_executable(clientApp src/main.cpp)

# 链接找到的库
target_link_libraries(clientApp PRIVATE ${MYLIBRARY_LIB})

# 包含 MyLibrary 的头文件
target_include_directories(clientApp PRIVATE ${CMAKE_SOURCE_DIR}/../MyLibrary/src)
```

### 4. 总结

1. **静态库 `.a`** 在编译时链接到可执行文件，生成的可执行文件无需在运行时依赖库文件。
2. **动态库 `.so`** 在运行时动态加载，减小了可执行文件的体积，但需要确保库文件在运行时可用。
3. 使用 CMake 生成静态库或动态库时，可以通过选项动态选择，方便管理。
4. 在其他项目中链接库时，可以使用 `target_link_libraries()` 来链接静态或动态库，并通过 `find_library()` 来自动查找库文件。

通过以上配置，可以在 CMake 项目中灵活使用静态库和动态库。