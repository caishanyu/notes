# CMake

> CMake 是个一个开源的跨平台自动化建构系统，用来管理软件建置的程序，并不依赖于某特定编译器，并可支持多层目录、多个应用程序与多个函数库。
>
> CMake 通过使用简单的配置文件 CMakeLists.txt，自动生成不同平台的构建文件（如 Makefile、Ninja 构建文件、Visual Studio 工程文件等），简化了项目的编译和构建过程。
>
> CMake 本身不是构建工具，而是生成构建系统的工具，它生成的构建系统可以使用不同的编译器和工具链。
>

## 目录

- [CMake](#cmake)
  - [目录](#目录)
  - [CMake安装和配置](#cmake安装和配置)
  - [CMake基础](#cmake基础)
    - [文件结构和基本语法](#文件结构和基本语法)
  - [CMake构建流程](#cmake构建流程)
    - [创建构建目录](#创建构建目录)
    - [使用CMake生成构建文件](#使用cmake生成构建文件)
    - [编译与构建](#编译与构建)
    - [清理构建文件](#清理构建文件)
    - [重新配置](#重新配置)

## CMake安装和配置

CMake可以在不同操作系统上进行安装，这里只记录Linux系统（笔者用的树莓派，属于Debian系统）下的安装和配置

CMake可以通过包管理器或者源码进行安装，这里使用包管理器

```bash
sudo apt-get install cmake  # 安装CMake
cmake --version     # 输出版本号说明安装成功
```

## CMake基础

CMakeLists.txt 是 CMake 的配置文件，用于定义项目的构建规则、依赖关系、编译选项等。

每个 CMake 项目通常都有一个或多个 CMakeLists.txt 文件。

### 文件结构和基本语法

CMakeLists.txt 文件使用一系列的 CMake 指令来描述构建过程。常见的指令包括：

|作用|语句|举例|
|--|--|--|
|指定 CMake 的最低版本要求|`cmake_minimum_required(VERSION <version>)`|`cmake_minimum_required(VERSION 3.10)`|
|定义项目的名称和使用的编程语言|`project(<project_name> [<language>...])`|`project(MyProject CXX)`|
|指定要生成的可执行文件和其源文件|`add_executable(<target> <source_files>...)`|`add_executable(MyExecutable main.cpp other_file.cpp)`|
|创建一个库（静态库或动态库）及其源文件|`add_library(<target> <source_files>...)`|`add_library(MyLibrary STATIC library.cpp)`|
|链接目标文件与其他库|`target_link_libraries(<target> <libraries>...)`|`target_link_libraries(MyExecutable MyLibrary)`|
|添加头文件搜索路径|`target_include_directories(<dirs>...)`|`target_include_directories(${PROJECT_SOURCE_DIR}/include)`|
|设置变量的值|`set(<variable> <value>...)`|`set(CMAKE_CXX_STANDARD 11)`|
|设置目标属性|`target_include_directories(...)`|`target_include_directories(MyExecutable PRIVATE ${PROJECT_SOURCE_DIR}/include)`|
|安装规则|`install(...)`|`install(TARGETS MyExecutable RUNTIME DESTINATION bin)`|
|条件语句|`if(expression) elseif(expression) else() endif()`||
|自定义命令|`add_custom_command(...)`||

## CMake构建流程

### 创建构建目录

CMake推荐使用"Out-of-source"构建方式，即将构建文件放在源代码目录之外的独立目录中。这样可以保持源代码目录的整洁，并方便管理不同的构建配置。

在项目的根目录下，创建一个新的构建目录。例如，可以创建一个名为 build 的目录，后续命令在这个目录下执行

```bash
mkdir build
cd build
```

### 使用CMake生成构建文件

在构建目录中运行 CMake 命令，指定源代码目录。源代码目录是包含 CMakeLists.txt 文件的目录

```bash
cmake ..
```

### 编译与构建

使用生成的构建文件进行编译和构建，对于 Makefile

```bash
make
```

### 清理构建文件

构建过程中生成的中间文件和目标文件可以通过清理操作删除（如果定义了清理规则）

```
make clean
```

### 重新配置

如果修改了 CMakeLists.txt 文件或项目设置，可能需要重新配置和构建项目

在构建目录中重新运行 CMake 配置命令，并使用构建命令重新编译项目

```bash
cmake ..
make
```