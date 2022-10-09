# Cmake cookbook读书笔记

## 第0章：环境配置

1.CMake的两个应用方向：

- 源代码构建开发库
- 源代码构建应用程序

2.应用程序基于框架，框架基于开发库，开发库基于更小的库和可执行文件。

3.CMake、CTest、CPack和CDash为主流的源代码构建工具

- CMake：描述如何在所有主要硬件和操作系统上配置、构建和安装项目；
- CTest：定义测试、测试套件，并设置如何执行；
- CPack：为打包需求提供DSL；
- CDash：将项目的测试结果在面板上展示；

4.CMake是什么？CMake是一个构建生成器，提供强大的领域特定语言（DSL）描述构建系统应该实现的功能。

5.几个关键时间点：

- CMake time: 在这个阶段，CMake将处理并配置项目中CMakeLists.txt文件；
- Generation time: 在这个阶段，CMake将生成本地构建工具所需的脚本，以执行项目中的后续步骤；
- Build time: 在这个阶段，CMake在平台和工具原生构建脚本上调用原生构建工具，这些脚本是由CMake生成的。此时，将调用编译器，并在特定的构建目录中构建目标（如build文件夹）。

![](./%5CFig%5C1.png)

6.本书的代码网址：https://github.com/dev-cafe/cmake-cookbook

**7.CMake一般配合自动化构建工具GNU Make一起使用，也可以使用Ninja**

8.OpenMPI是一个消息传递接口

## 第一章：从可执行文件到库

### 1.1将单个源文件编译为可执行文件

```C++
#include<cstdlib>
#include<iostream>
#include<string>

std::string say_hello(){
    returnstd::string("Hello, CMake world!");
}

int main(){
std::cout << say_hello() << std::endl;
return EXIT_SUCCESS;
}
```

1.CMakeList.txt: 是向CMake提供配置描述的文件

**NOTE**:*文件的名称区分大小写，必须命名为`CMakeLists.txt`，CMake才能够解析。*

2.具体步骤如下：

1. 用编辑器打开一个文本文件，将这个文件命名为`CMakeLists.txt`。

2. 第一行，设置CMake所需的最低版本。如果使用的CMake版本低于该版本，则会发出致命错误：

   ```cmake
   cmake_minimum_required(VERSION 3.5 FATAL_ERROR)
   ```

3. 第二行，声明了项目的名称(`recipe-01`)和支持的编程语言(CXX代表C++)：

   ```cmake
   project(recipe-01 LANGUAGES CXX)
   ```

4. 指示CMake创建一个新目标：可执行文件`hello-world`。这个可执行文件是通过编译和链接源文件`hello-world.cpp`生成的。**CMake将为编译器使用默认设置，并自动选择生成工具**：

   ```cmake
   add_executable(hello-world hello-world.cpp)
   ```

5. 现在，可以通过创建`build`目录，在`build`目录下来配置项目：

   ```shell
   $ mkdir -p build
   $ cd build
   $ cmake ..
   
   -- The CXX compiler identification is GNU 8.1.0
   -- Check for working CXX compiler: /usr/bin/c++
   -- Check for working CXX compiler: /usr/bin/c++ -- works
   -- Detecting CXX compiler ABI info
   -- Detecting CXX compiler ABI info - done
   -- Detecting CXX compile features
   -- Detecting CXX compile features - done
   -- Configuring done
   -- Generating done
   -- Build files have been written to: /home/user/cmake-cookbook/chapter-01/recipe-01/cxx-example/build
   ```

6. 如果一切顺利，项目的配置已经在`build`目录中生成。我们现在可以编译可执行文件：

   ```shell
   $ cmake --build .
   这一步如果使用的是GNU Make，则可以：
   $ make -j x
   
   Scanning dependencies of target hello-world
   [ 50%] Building CXX object CMakeFiles/hello-world.dir/hello-world.cpp.o
   [100%] Linking CXX executable hello-world
   [100%] Built target hello-world
   ```

**NOTE**:*CMake语言不区分大小写，但是参数区分大小写。*

**TIPS**:*CMake中，C++是默认的编程语言。不过，我们还是建议使用`LANGUAGES`选项在`project`命令中显式地声明项目的语言。*

3.要配置项目并生成构建器，我们必须通过命令行界面(CLI)运行CMake，`cmake -help`将展示所有的命令。

```shell
$ mkdir -p build
$ cd build
$ cmake ..
```

可以使用以下命令行来实现相同的效果：

```shell
$ cmake -H. -Bbuild
```

该命令是跨平台的，使用了`-H`和`-B`为CLI选项。`-H`表示当前目录中搜索根`CMakeLists.txt`文件。`-Bbuild`告诉CMake在一个名为`build`的目录中生成所有的文件。

**NOTE**:*`cmake -H. -Bbuild`也属于CMake标准使用方式: https://cmake.org/pipermail/cmake-developers/2018-January/030520.html 。不过，我们将在本书中使用传统方法(创建一个构建目录，进入其中，并通过将CMake指向`CMakeLists.txt`的位置来配置项目)。*

**NOTE**:*在与`CMakeLists.txt`相同的目录中执行`cmake .`，原则上足以配置一个项目。

GNU/Linux上，CMake默认生成Unix Makefile来构建项目：

* `Makefile`: `make`将运行指令来构建项目。
* `CMakefile`：包含临时文件的目录，CMake用于检测操作系统、编译器等。此外，根据所选的生成器，它还包含特定的文件。
* `cmake_install.cmake`：处理安装规则的CMake脚本，在项目安装时使用。
* `CMakeCache.txt`：如文件名所示，CMake缓存。CMake在重新运行配置时使用这个文件。

要构建示例项目，我们运行以下命令：

```shell
$ cmake --build .
```

最后，CMake不强制指定构建目录执行名称或位置，我们完全可以把它放在项目路径之外。这样做同样有效：

```shell
$ mkdir -p /tmp/someplace
$ cd /tmp/someplace
$ cmake /path/to/source
$ cmake --build .
```

## 更多信息

官方文档 https://cmake.org/runningcmake/ 给出了运行CMake的简要概述。由CMake生成的构建系统，即上面给出的示例中的Makefile，将包含为给定项目构建目标文件、可执行文件和库的目标及规则。`hello-world`可执行文件是在当前示例中的唯一目标，运行以下命令：

```shell
$ cmake --build . --target help

The following are some of the valid targets for this Makefile:
... all (the default if no target is provided)
... clean
... depend
... rebuild_cache
... hello-world
... edit_cache
... hello-world.o
... hello-world.i
... hello-world.s
```

CMake生成的目标比构建可执行文件的目标要多。可以使用`cmake --build . --target <target-name>`语法，实现如下功能：

* **all**(或Visual Studio generator中的ALL_BUILD)是默认目标，将在项目中构建所有目标。
* **clean**，删除所有生成的文件。
* **rebuild_cache**，将调用CMake为源文件生成依赖(如果有的话)。
* **edit_cache**，这个目标允许直接编辑缓存。

对于更复杂的项目，通过测试阶段和安装规则，CMake将生成额外的目标：

* **test**(或Visual Studio generator中的**RUN_TESTS**)将在CTest的帮助下运行测试套件。我们将在第4章中详细讨论测试和CTest。
* **install**，将执行项目安装规则。我们将在第10章中讨论安装规则。
* **package**，此目标将调用CPack为项目生成可分发的包。打包和CPack将在第11章中讨论。



## 第二章：检测环境

## 第三章：检测外部库和程序

## 第四章：创建和运行测试

## 第五章：配置时和构建时的操作

## 第六章：生成源码

## 第七章：构建项目

## 第八章：超级构建模式

## 第九章：语言混合项目

## 第十章：编写安装程序

## 第十一章：打包项目

## 第十二章：构建文档

## 第十三章：选择生成器和交叉编译

## 第十四章：测试面板

## 第十五章：使用CMake构建已有项目