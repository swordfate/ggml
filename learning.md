[TOC]


# 编译安装debug
cmake .. 是根据 CMakeLists.txt 生成 Makefile 的过程
cmake --build . 是根据 Makefile 进行编译和连接的过程

执行的时候报错：
/usr/bin/ld: CMakeFiles/test-backend-ops.dir/test-backend-ops.cpp.o: undefined reference to symbol 'pthread_create@@GLIBC_2.2.5'
/usr/bin/ld: /lib/x86_64-linux-gnu/libpthread.so.0: error adding symbols: DSO missing from command line
collect2: error: ld returned 1 exit status

使用 roo 的 deepseek-v3 后提示，在 CMakeLists.txt 中的 find_package 命令下面添加了一个语句解决了这个 bug
```shell
find_package(Threads REQUIRED)  # 查找系统线程库
link_libraries(Threads::Threads)  # 确保所有目标链接pthread库
```


# CMake 学习笔记

## 基础语法

### 1. 项目设置命令
```cmake
# 设置CMake最低版本
cmake_minimum_required(VERSION 版本号)

# 定义项目
project(项目名 [语言...])

# 设置变量
set(变量名 值)

# 包含其他CMake模块
include(模块名)
```

### 2. 条件语句
```cmake
# if-else基本语法
if (条件)
    # 命令
elseif (条件)
    # 命令
else()
    # 命令
endif()

# 常见条件判断
if (APPLE)                         # 判断是否是MacOS
if (WIN32)                         # 判断是否是Windows
if (CMAKE_SYSTEM_NAME STREQUAL "Linux")  # 判断是否是Linux
if (DEFINED 变量)                  # 判断变量是否已定义
```

### 3. 选项设置
```cmake
# 定义开关选项
option(选项名 "说明文字" ON/OFF)

# 设置缓存变量
set(变量名 默认值 CACHE STRING "说明文字")
```

### 4. 依赖处理
```cmake
# 查找依赖包
find_package(包名 [REQUIRED])

# 链接库
link_libraries(库名)

# 添加子目录
add_subdirectory(目录名)
```

### 5. 目标管理
```cmake
# 设置目标属性
set_target_properties(目标名 PROPERTIES
    属性名 属性值)

# 安装规则
install(TARGETS 目标名 
    DESTINATION 目标路径
    [其他选项...])
```

## 重要概念

### 1. 变量作用域
- CMAKE_SOURCE_DIR: 顶级源代码目录
- CMAKE_BINARY_DIR: 顶级构建目录
- CMAKE_CURRENT_SOURCE_DIR: 当前CMakeLists.txt所在目录
- PROJECT_NAME: 当前项目名称

### 2. 构建类型
- Debug: 调试模式
- Release: 发布模式
- MinSizeRel: 最小体积发布
- RelWithDebInfo: 带调试信息的发布

### 3. 常用选项类型
- 编译器选项 (例如: -Wall, -Werror)
- 优化选项 (例如: -O2, -march=native)
- 链接选项 (例如: -static, -shared)
- 特性开关 (例如: BUILD_TESTING, BUILD_SHARED_LIBS)

### 4. 最佳实践
1. 保持CMakeLists.txt结构清晰:
   - 基本设置
   - 选项定义
   - 依赖处理
   - 目标定义
   - 安装规则

2. 命名约定:
   - 选项名使用全大写
   - 变量名使用大写
   - 函数名使用小写

3. 注释规范:
   - 为重要配置添加说明性注释
   - 为复杂逻辑添加解释性注释

4. 版本控制:
   - 明确指定所需的CMake最低版本
   - 说明依赖的版本要求

## 高级特性
- 自定义命令和目标
- 生成器表达式
- 跨平台配置
- 工具链文件
- ExternalProject管理

_注: 这个文档会随着学习的深入不断更新和补充。_



# 运行 demo