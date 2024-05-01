---
title: CMake教程
date: 2024-04-30 19:07:40
tags:
---

## 构建可执行程序

```cmake
add_executable(demo demo.cxx)
```

## 添加版本号

`cmakelists.txt`写入：

```cmake
cmake_minimum_required(VERSION 2.6)
project(Demo)
# 版本号 1.0
set(Demo_VERSION_MAJOR 1)
set(Demo_VERSION_MINOR 0)

# 用.in文件生成.h文件
configure_file(
        "${PROJECT_SOURCE_DIR}/DemoConfig.h.in"
        "${PROJECT_BINARY_DIR}/DemoConfig.h"
)

include_directories("${PROJECT_BINARY_DIR}")

add_executable(demo demo.cpp)
```

新建`DemoConfig.h.in`文件，写入：

```c
#define VERSION_MAJOR @Demo_VERSION_MAJOR@
#define VERSION_MINOR @Demo_VERSION_MINOR@
```

新建`demo.cpp`文件，写入：

```c
#include <stdio.h>
#include "DemoConfig.h"

int main (int argc, char *argv[])
{
    fprintf(stdout, "%s --- 版本号： %d.%d\n",
            argv[0],
            VERSION_MAJOR,
            VERSION_MINOR);
    return 0;
}
```

打印版本号。

为什么不直接把版本号写到源码里呢？当然可以，写在cmake配置文件里比较灵活。

## 添加一个库

新建一个目录，名为`MathFunctions`,里面新建两个文件，`MathFunctions.h`和`math.cpp`。
.h文件里写入：

```c
#ifndef DEMO_MATHFUNCTIONS_H
#define DEMO_MATHFUNCTIONS_H
int square(int x);
#endif //DEMO_MATHFUNCTIONS_H
```

.cpp文件里写入:

```c
int square(int x) {
    int res;
    res = x * x;
    return res;
}
```

然后在此目录中新建一个``CmakeLists.txt`，写入：
```cmake
add_library(MathFunctions math.cpp)
```

回到父目录，`DemoConfig.h.in`里写入:
```cmake
#cmakedefine USE_MYMATH
```

回到父目录的cmkaelist.txt文件中，修改为:
```cmake
cmake_minimum_required(VERSION 2.6)
project(Demo)
# 版本号 1.0
set(Demo_VERSION_MAJOR 1)
set(Demo_VERSION_MINOR 0)

# 用.in文件生成.h文件
configure_file(
        "${PROJECT_SOURCE_DIR}/DemoConfig.h.in"
        "${PROJECT_BINARY_DIR}/DemoConfig.h"
)

include_directories("${PROJECT_BINARY_DIR}")

# 是否使用我们自己的函数？
option(USE_MYMATH
        "Use demo provided math implementation" ON)

# add the MathFunctions library?
if (USE_MYMATH)
    include_directories("${PROJECT_SOURCE_DIR}/MathFunctions")
    add_subdirectory(MathFunctions)
    set(EXTRA_LIBS ${EXTRA_LIBS} MathFunctions)
endif (USE_MYMATH)

# add the executable
add_executable(demo demo.cpp)
target_link_libraries(demo ${EXTRA_LIBS})
```

在demo.cpp里测试一下:
```c
#include <stdio.h>
#include "DemoConfig.h"
#ifdef USE_MYMATH
#include "MathFunctions.h"
#endif

int main (int argc, char *argv[])
{
    fprintf(stdout, "%s --- 版本号： %d.%d\n",
            argv[0],
            VERSION_MAJOR,
            VERSION_MINOR);

    printf("%d\n",square(5));
    return 0;
}
```

## 安装与打包
先安装NSIS程序：https://nsis.sourceforge.io/Main_Page

顶层`cmakelists.txt`中写入：
```cmake
# add the install targets
install (TARGETS demo DESTINATION bin)
install (FILES "${PROJECT_BINARY_DIR}/DemoConfig.h"
        DESTINATION include)

# build a CPack driven installer package
include (InstallRequiredSystemLibraries)
set (CPACK_RESOURCE_FILE_LICENSE
        "${CMAKE_CURRENT_SOURCE_DIR}/License.txt")
set (CPACK_PACKAGE_VERSION_MAJOR "${demo_VERSION_MAJOR}")
set (CPACK_PACKAGE_VERSION_MINOR "${demo_VERSION_MINOR}")
include (CPack)
```
`mathfunctions`目录中的`cmakelists.txt`中写入：
```cmake
install (TARGETS MathFunctions DESTINATION bin)
install (FILES MathFunctions.h DESTINATION include)
```
命令行运行：
```bash
cpack --config CPackConfig.cmake
```

生成了Demo--win64.exe安装程序。
