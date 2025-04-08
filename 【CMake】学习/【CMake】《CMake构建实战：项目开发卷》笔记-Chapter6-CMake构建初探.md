# 第6章　CMake构建初探

从本章开始，我们不再仅仅停留在CMake脚本程序中，而是开始将CMake看作一个用于构建项目的利器了。在了解CMake的具体用法之前，先要清楚地掌握CMake项目的构建流程。换句话说，就是要清楚地掌握CMake项目的生命周期——从源程序和CMake目录程序，到构建好的二进制文件，再到这些二进制文件的安装和打包分发，最终到其他项目的源程序借助CMake使用这些二进制文件提供的功能。

本章会说明CMake是怎样实现这样一个生命周期的，重点介绍其构建项目过程中的6个阶段，尤其是与构建紧密相关的阶段。另外，本章还会介绍CMake缓存变量配置相关内容、CMake命令行的使用及与Visual Studio的搭配使用方法等，了解这些内容将有助于更好地使用CMake完成项目的构建。

另外，本章在介绍构建项目的后续阶段及CMake命令行的部分形式时，会涉及安装与打包的内容，若想了解这些主题的详细内容，可参阅CMake官方文档。如果在阅读过程中还遇到了其他生疏的概念，不必担心，它们还会在这里等待读者的第二次见面。

## 6.1　CMake项目的生命周期

### 6.1.1　配置阶段和生成阶段

CMake本身并不实际调用编译器和链接器等，而是根据整个构建流程，生成Makefile或者其他构建工具的配置文件，通过它们来实际调用各种命令完成构建。正因如此，CMake常被称为构建系统生成器。

CMake构建系统生成器在构建项目的过程中涉及两个重要阶段：一是执行CMake目录程序（CMakeLists.txt）的阶段，二是根据程序执行结果生成构建系统配置文件的阶段。前者往往是对项目的构建环境、构建目标等进行配置，因此称为配置阶段（configure stage）；而后者用于生成构建系统的配置文件，因此称为生成阶段（generation stage）。这两个阶段均由CMake独立完成，其关系如下图所示。

```mermaid
graph TB
    CMakeLists.txt -- 配置阶段 --> CMake持久化缓存和CMakeFiles
    
    CMake持久化缓存和CMakeFiles -- 生成阶段 --> IDE项目文件或Makefiles
    
    CMakeLists.txt -- 生成阶段 --> IDE项目文件或Makefiles
    
    CMakeLists.txt -- 配置阶段 --> 生成的程序或资源文件
    
    IDE项目文件或Makefiles -. 构建阶段 .-> 生成的程序或资源文件
    
    生成的程序或资源文件 -. 构建阶段 .-> 目标文件和库文件和可执行文件等二进制文件
    
    源程序文件 -. 构建阶段 .-> 目标文件和库文件和可执行文件等二进制文件
```

之所以要区分这两个阶段，是因为在配置阶段，CMake仅需确定项目构建目标的依赖关系、构建需求等，与选用的具体构建系统的特殊化配置无关。在生成阶段，CMake必须根据目标构建系统（如Makefile等）的要求，生成出符合具体构建系统要求格式的具体配置。

上图中的“生成的程序或资源文件”既可能是配置阶段由CMake脚本直接生成的，又可能是构建阶段生成的。这是因为有些信息可能与构建系统的配置有关，因此CMake不能直接生成它们，而是将它们的生成过程在生成阶段定义到构建系统的配置中，再由构建系统最终在构建阶段生成。

例如，项目通常有Debug（调试）模式和Release（发布）模式两种构建模式，以对应不同的编译优化级别。对于Makefile构建系统而言，CMake需要对这两种构建模式分别生成不同的Makefile项目配置；而对于Visual Studio构建系统而言，由于其本身支持多种构建模式的切换， CMake只需生成一个Visual Studio的项目配置。试想，如果需要根据当前的构建模式生成不同的头文件内容，该怎么做呢？是否可以直接在CMake目录程序中获取当前构建模式并使用if命令来判断呢？

这对于Makefile这种单构建模式的构建系统是可行的，但对于Visual Studio这种支持多构建模式的构建系统则是不可行的。因为在配置阶段执行CMake目录程序时，CMake并不能够确定多构建模式构建系统当前选择的是哪种构建模式，这个信息只有在构建时才会确定。为了解决这类问题，CMake提供了生成器表达式。顾名思义，生成器表达式就是构建系统生成器的表达式，也就是生成阶段才会被解析的表达式。总而言之，这类问题只有在生成阶段结合了构建系统的具体特性后才可解决。其相关内容将在第8章中讲解。

#### 常用构建系统生成器

执行`cmake --help`命令可以查看CMake支持的全部构建系统生成器。较为常用的构建系统生成器有以下几种。

- `Visual Studio <主版本号>[ <版本年>]`，如`Visual Studio 16 2019`或`Visual Studio 16`均表示可以生成`Visual Studio 2019`解决方案的构建系统生成器；

- Unix Makefiles，即生成标准的UNIX平台的Makefile（包括GNU Makefile）的构建系统生成器；

- NMake Makefiles，即生成NMake的Makefile的构建系统生成器；

- Ninja，生成ninja-build（一个性能极高的构建工具）项目配置的构建系统生成器。

有一些构建系统生成器的名称形如`<附加生成器> - <主生成器>`。短横线前的附加生成器（extra generator）一般是IDE的名称，短横线后的主生成器（main generator）则是常见构建系统的名称。例如，`CodeBlocks - Unix Makefiles`、`CodeBlocks - NMake Makefiles`、`CodeBlocks - Ninja`等，这些生成器分别用于生成基于不同构建系统CodeBlocks集成开发环境的项目文件。CMake还支持其他一些附加生成器，如`CodeLite`、`Eclipse CDT4`、`Kate`和`Sublime Text 2`等。

#### 实例：CMake的配置和生成阶段

现在通过一个静态库实例来演示CMake的配置和生成阶段。首先，在lib.c 源程序中实现一个用于整数加法的函数add，如下所示。

```
int add(int a, int b) { return a + b; }
```

然后，定义CMake目录程序CMakeLists.txt。其中，部分命令与项目构建、安装和打包相关，这里可以参考代码注释来大致理解，如下所示。

```
cmake_minimum_required(VERSION 3.20)

# 定义项目名称和版本
project(mylib VERSION 1.0.0)

# 添加静态库目标mylib，其源代码包含lib.c
add_library(mylib STATIC lib.c)

# 安装mylib构建目标
install(TARGETS mylib)

# 打包
set(CPACK_PACKAGE_NAME "mylib")
include(CPack)
```

最后，配置和生成项目。需要注意的是，CMake命令行仅支持同时执行CMake的配置和生成阶段，因此很难观察出CMake的持久化缓存文件CMakeCache.txt 和临时文件夹CMakeFiles是先生成的，而构建系统配置文件是后生成的。不过，CMake提供了一个可视化工具CMake GUI，用于分别执行CMake的配置和生成阶段。读者如果感兴趣，可以自行尝试。

下面以在Windows中使用Visual Studio 2019为例，展示调用cmake命令来进行配置和生成阶段的操作：

```
> cd CMakeBook/src/ch006/mylib
> mkdir build
> cd build
> cmake ..
-- Building for: Visual Studio 16 2019
-- Selecting Windows SDK version 10.... to target Windows 10....
-- The C compiler identification is MSVC 19....
-- The CXX compiler identification is MSVC 19....
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Check for working C compiler: C:/.../cl.exe - skipped
-- Detecting C compile features
-- Detecting C compile features - done
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Check for working CXX compiler: C:/.../cl.exe - skipped
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Configuring done
-- Generating done
-- Build files have been written to: C:/CMake-Book/src/ch006/mylib/build
```

cmake命令接受源文件目录作为参数，此处即上级目录`“..”`。在CMake输出的日志中，Configuring done表示配置完成，Generating done表示生成完成。

由于Visual Studio 2019是支持多构建模式的构建系统，在配置生成阶段无须指定构建模式。如果使用的是单构建模式的构建系统，如Makefile，则必须在配置生成阶段使用`CMAKE_BUILD_TYPE`变量指定构建模式。下面是在Linux操作系统中使用CMake配置和生成Makefile构建系统项目的执行过程：

```
$ cd CMakeBook/src/ch006/mylib
$ mkdir build_debug
$ mkdir build_release
$ cd build_debug
$ cmake -DCMAKE_BUILD_TYPE=Debug ..
...
$ cd ../build_release
$ cmake -DCMAKE_BUILD_TYPE=Release ..
...
```

### 6.1.2　构建阶段

在`### 6.1.1　配置阶段和生成阶段`图中，已经了解了构建阶段（build stage）的作用，即构建源程序为构建目标的二进制文件等。这个阶段没有特别之处，几乎完全依靠构建系统本身而非CMake来完成。也许构建阶段中会有部分小任务是靠调用CMake来完成的，但那也是由构建系统负责完成这些任务的组织调度。简言之，构建阶段发生的事情和本书第1章所介绍的内容没有本质区别。

#### 构建模式

前面已经提到过构建模式（build configuration）了，这里有必要再详细介绍一下。不同的构建模式分别对应一系列不同的预置编译链接选项，我们可以根据需求方便地切换。 CMake默认提供如下四种构建模式。

- Debug调试模式，禁用代码优化，便于调试。

- Release发布模式，启用代码优化并针对速度优化，启用内联并丢失调试符号，几乎无法调试。

- RelWithDebInfo发布调试模式，启用代码优化，但保留符号且不会内联函数，仍可调试。

- MinSizeRel最小体积发布模式，启用代码优化，但针对二进制体积进行优化，使其尽可能小。

对于单构建模式的构建系统而言，应在CMake配置生成阶段通过`CMAKE_BUILD_TYPE`变量选择所需的构建模式。若想同时构建Debug和Release模式的程序，必须分别在两个目录中以不同的`CMAKE_BUILD_TYPE`变量值配置生成项目，再分别进行构建。6.1.1小节最后的实例中正是这样做的。

对于多构建模式的构建系统而言，无须在配置生成阶段指定构建模式，且仅需在一个目录中配置生成一次，就可以在构建阶段通过具体的构建系统的命令行工具（或`cmake --build` 命令行的`--config`参数）指定所需的构建模式。下面的实例展示了这一过程。

#### 实例：使用CMake命令行构建项目

为了构建项目，可以打开Visual Studio的命令行工具，然后调用MSBuild命令行工具构建CMake生成的Visual Studio构建系统配置，即Visual Studio解决方案。也可以直接使用Visual Studio集成开发环境打开解决方案，然后使用可视化界面中提供的构建菜单或工具栏按钮进行构建。但这样未免有些烦琐，而且每一个构建系统都有不同的用户界面或交互方式，学习成本颇高。

CMake命令行提供的`--build`参数可以帮助调用构建系统（因此实际的构建过程仍由构建系统完成）。这样，无论使用何种构建系统，都可以使用统一的CMake命令接口来调用，从而避免记忆各式构建系统的命令接口，也便于编写自动化构建脚本。构建过程如下：

```
> cd CMakeBook/src/ch006/mylib
> cd build
> cmake --build . --config Release
用于 .NET Framework 的 Microsoft (R) 生成引擎版本 16....
版权所有(C) Microsoft Corporation。保留所有权利。
　
Checking Build System
Building Custom Rule C:/CMake-Book/src/ch006/mylib/CMakeLists.txt
lib.c
mylib.vcxproj -> C:\CMake-Book\src\ch006\mylib\build\Release\mylib.lib
Building Custom Rule C:/CMake-Book/src/ch006/mylib/CMakeLists.txt
```

在调用`cmake --build`命令行时，首先指定CMake项目的构建目录，即当前目录`“.”`；然后，通过`--config`参数指定构建模式为Release模式，即发布模式。命令执行完成后，可以在build目录中的Release子目录中找到构建好的静态库mylib.lib。

对于单构建模式的构建系统而言，`--config`参数是没有意义的，因为构建模式在配置生成阶段就已经通过`CMAKE_BUILD_TYPE`变量指定了。

### 6.1.3　安装阶段和打包阶段

如果只是希望把源程序文件构建成可执行文件，然后双击运行，那么上述三个阶段就已经足够完成这个目标了。但事实上，我们常常还需要分发程序包，以便用户安装使用。此时就需要用到CMake的install命令配置程序的安装规则了。我们还会用到CMake集成的CPack提供的打包功能，将需要安装的二进制程序文件打包，以便分发。这两个阶段分别对应安装阶段（install stage）和打包阶段（CPack stage）。

下图展示了CMake项目的完整生命周期。这张图是`### 6.1.1　配置阶段和生成阶段`图的超集，并且图中的虚线代表不属于本项目的部分，具体内容将在6.1.4小节介绍。

```mermaid
graph TB
    CMakeLists.txt -- 配置阶段 --> CMake持久化缓存和CMakeFiles
    
    CMake持久化缓存和CMakeFiles -- 生成阶段 --> IDE项目文件或Makefiles
    
    CMakeLists.txt -- 生成阶段 --> IDE项目文件或Makefiles
    
    CMakeLists.txt -- 配置阶段 --> 生成的程序或资源文件
    
    IDE项目文件或Makefiles -- 构建阶段 --> 生成的程序或资源文件
    
    生成的程序或资源文件 -- 构建阶段 --> 目标文件和库文件和可执行文件等二进制文件
    
    源程序文件 -- 构建阶段 --> 目标文件和库文件和可执行文件等二进制文件
    
    生成的程序或资源文件 -- 安装阶段 --> 安装后的文件
    
    目标文件和库文件和可执行文件等二进制文件 -- 安装阶段 --> 安装后的文件
    
    源程序文件 -- 安装阶段 --> 安装后的文件
    
    安装后的文件 -- 打包CPack阶段 --> 打包后的程序包
    
    打包后的程序包 -. 程序包安装阶段 .-> 安装的程序包
    
    安装的程序包 -. 引用程序包 .-> CMakeLists.txt
```


#### 实例：安装和打包CMake项目

下面演示如何使用CMake来安装和打包mylib项目。安装和打包这两个阶段并没有先后依赖关系，可以根据需求执行任一或全部阶段。

首先，使用`cmake --install`命令安装CMake项目：

```
> cd CMakeBook/src/ch006/mylib
> cd build
> cmake --install . --prefix ../install
-- Install configuration: "Release"
-- Installing: C:/CMake-Book/src/ch006/mylib/install/lib/mylib.lib
```

CMake项目的构建目录，即当前目录`“.”`，应作为`cmake --install`命令行的第一个参数。然后是`--prefix`参数，用于指定安装目录前缀，即CMake项目安装位置的根目录。在该实例中，安装目录前缀为与构建目录同级的install目录。最后一行安装日志说明mylib.lib被安装到了安装目录前缀的lib子目录中。

下面使用cpack命令打包CMake项目：

```
> cd CMakeBook/src/ch006/mylib
> cd build
> cpack -G NSIS
CPack: Create package using NSIS
CPack: Install projects
CPack: - Install project: mylib []
CPack: Create package
CPack: - package: C:/.../build/mylib-1.0.0-win64.exe generated.
```

cpack命令会打包当前构建目录中的CMake项目，`-G`参数用于指定程序包生成器，这里指定了`NSIS`程序包生成器。NSIS是Nullsoft Scriptable Install System的简称，它是Nullsoft公司开发的一个脚本驱动的安装程序生成器。

命令执行后，当前构建目录中将会生成一个基于NSIS的安装程序mylib-1.0.0-win64.exe，双击执行它即可安装构建好的程序包。

### 6.1.4　程序包安装阶段

这个阶段不属于当前CMake项目，而是属于使用当前项目的其他项目。另外，`### 6.1.3　安装阶段和打包阶段`图中还有一个“引用程序包”的过程，这并非构建过程的一个阶段，因为它是通过另一个项目的CMake脚本来实现的。尽管如此，这二者仍然是构成CMake项目完整生命周期的重要组成部分。如果一个程序包能方便地被他人使用，那么编写这个程序包就能创造更大的价值。

用户安装打包后的程序包的阶段称为程序包安装阶段（package install stage）。需要注意区分它和安装阶段：程序包安装阶段是将打包好的程序文件解包并安装到当前环境中，而安装阶段则是将构建好的二进制程序文件安装到环境中。二者最终效果是一样的，但来源不同。如果用户直接获取了开发者提供的源程序并构建安装，那么实际上我们只需进行到安装阶段，但这样对用户来说非常耗时，不方便，且要求项目开源。如果用户下载并安装开发者提供的预编译安装包，那么实际上就是在进行程序包安装阶段，这样不需要用户自行构建二进制程序文件。当然，开发者需要预先为不同的构建环境编译好二进制程序并制作相应的安装包。

#### 实例：安装和使用预编译的程序包

本例将新建一个CMake项目，演示如何在该项目中引用前面实例中打包好的静态库。

首先，运行上一实例中打包的安装程序mylib-1.0.0-win64.exe以安装静态库，安装路径可以使用默认值C:\Program Files\mylib 1.0.0。

然后，创建一个C程序main.c，在其中调用mylib静态库提供的加法函数add，并输出1加2的结果，如下所示。

```
#include <stdio.h>

extern int add(int, int);

int main() {
    printf("%d\n", add(1, 2));
    return 0;
}
```

该项目的CMake目录程序如下所示。

```
cmake_minimum_required(VERSION 3.20)

# 定义项目名称
project(use_mylib VERSION 1.0.0)

# 在默认搜索目录的mylib 1.0.0/lib子目录中查找mylib库
# 将目录路径存入mylib_LIBRARY变量中
find_library(mylib_LIBRARY 
    mylib 
    PATH_SUFFIXES "mylib 1.0.0/lib"
)

# 输出 "C:/Program Files/mylib 1.0.0/lib/mylib.lib"
message("${mylib_LIBRARY}")

# 添加可执行文件目标main，其源代码包含main.c
add_executable(main main.c)

# 链接mylib库到main程序
target_link_libraries(main ${mylib_LIBRARY})
```

其中，`find_library`就是用于查找库的CMake命令。它会在系统默认目录中进行查找，包括`C:\Program Files`目录。同时，命令参数中还指定了查找库的路径后缀`mylib 1.0.0/lib`，这样CMake就能找到静态库的具体位置。静态库的绝对路径会被存到`find_library`命令的第一个参数所指定的变量中。

`target_link_libraries`命令用于将一系列库文件或库目标链接到指定的构建目标中。其第一个参数为构建目标，其后的参数为一系列库的构建目标名称或路径，这里填写查找到的mylib库的路径。

下面配置生成、构建该项目，并运行主程序进行测试：

```
> cd CMakeBook/src/ch006/使用mylib
> mkdir build
> cd build
> cmake ..
...
C:/Program Files/mylib 1.0.0/lib/mylib.lib
-- Configuring done
-- Generating done
-- Build files have been written to: C:/CMake-Book/src/ch006/使用mylib/build
　
> cmake --build .
...
> ./Debug/main.exe
3
```

## 6.2　项目配置与缓存变量

在CMake项目的完整生命周期中，缓存变量始终起着至关重要的作用。在`### 6.1.3　安装阶段和打包阶段`图中可以看到CMake在配置阶段会产生持久化缓存文件。这些持久化缓存文件会在后续执行CMake时直接被加载，因此通常用于存储一些花费较大代价获取的信息。例如，CMake在第一次配置项目时，会检测当前编译环境，搜索确定编译器、链接器等工具的路径，并将这些信息保存到缓存中。总而言之，缓存变量能够被持久化，通常被用于实现对项目的配置。

本节会使用CMake GUI工具，直观地展示CMake在配置阶段做了哪些主要的配置。

### 6.2.1　使用CMake GUI配置缓存变量

在CMake配置阶段产生的持久化缓存文件CMakeCache.txt中，定义了CMake在该阶段通过对环境（如编译器、构建工具等）的检测所作出的默认配置。下面来了解mylib实例项目中到底有哪些配置吧！

首先，清理之前的build目录，以便重新配置。打开CMake GUI可视化工具，设置好源程序路径及用于构建二进制的路径，点击“Configure”按钮以配置mylib项目。

第一次配置时，CMake GUI会弹出一个对话框，询问采用何种构建系统。这里选择默认选项即可，直接点击“Finish”按钮完成选择。接着，CMake GUI开始配置阶段的执行，日志会输出到界面下方的文本框中。

界面中，以红色高亮显示的缓存变量是当前配置阶段中首次定义的缓存变量。如果再一次点击“Configure”按钮进行配置，大部分红色高亮就会消失。当然，也存在部分项目需要迭代地进行配置，因此第二次配置仍然有部分新的缓存变量被定义，依然以红色高亮显示。对于这类项目，我们应当反复点击“Configure”按钮进行配置，直到再没有红色高亮显示的缓存变量出现。

另外，当前显示的缓存变量只有两个，这并非配置阶段定义的全部缓存变量。一般来说，CMake默认只显示用户更关心的缓存变量，而把其他缓存变量隐藏起来。勾选缓存变量列表上方的Advanced（高级配置）复选框可以显示全部缓存变量。将鼠标指针悬停在列表项上方，可以查看对应缓存变量的描述文本。

### 6.2.2　常用缓存变量

简单浏览一下部分缓存变量配置。

首先，查看非高级配置，毕竟它们是更需要关注的缓存变量。

- `CMAKE_CONFIGURATION_TYPES`，即支持的构建模式列表。其中元素取值可为Debug、Release、 MinSizeRel、RelWithDebInfo，分别代表调试模式、发布模式、最小大小发布模式、带调试信息的发布模式。

- `CMAKE_INSTALL_PREFIX`，即安装目录前缀。在介绍安装阶段时介绍过，`cmake --install`命令可以用`--prefix`指定安装目录前缀。若省略该参数，则该命令会使用`CMAKE_INSTALL_PREFIX`缓存变量的值。

勾选Advanced复选框，即可使用高级配置查看所有的缓存变量。首先是文件路径类型的缓存变量：

- `CMAKE_AR`，即用于打包静态库的归档工具的路径；

- `CMAKE_LINKER`，即链接器的路径。

其次是一些与编译参数相关的缓存变量配置。

- `CMAKE_<编程语言>_FLAGS`，即<编程语言>编译器的参数选项列表。该配置对所有构建模式生效。勾选缓存变量列表上方的Advanced（高级配置）复选框时有对C和CXX（即`C++`）编程语言的相关配置。

- `CMAKE_<编程语言>_FLAGS_<构建模式>`，即配置特定<构建模式>下对应<编程语言>编译器的参数选项列表。该配置仅对指定构建模式生效。

接下来是一些与链接器参数相关的缓存变量配置。

- `CMAKE_<目标类型>_LINKER_FLAGS`，即对应<目标类型>的链接参数列表。该配置对所有构建模式生效。 “目标类型”可为EXE、MODULE、SHARED、STATIC，即可执行程序、模块、动态库和静态库。

- `CMAKE_<目标类型>_LINKER_FLAGS_<构建模式>`，即配置特定<构建模式>下的对应<目标类型>的链接参数列表。该配置仅对指定构建模式生效。

另外还有很多其他缓存变量配置，在此就不一一介绍了。其中有一些可能在后面的章节中会涉及，建议感兴趣的读者自行查阅相关CMake官方文档。

### 6.2.3　标记缓存变量为高级配置：mark_as_advanced

第3章介绍过如何使用set或option命令自定义缓存变量。这些自定义缓存变量默认情况下显示在CMake GUI中。如果想将某些缓存变量隐藏到高级配置中，可以调用`mark_as_advanced`命令：

```
mark_as_advanced([CLEAR|FORCE] <缓存变量>...)
```

该命令可以将指定的若干<缓存变量>设置为高级配置，即使其仅在CMake GUI中勾选Advanced复选框后才显示在缓存变量列表中。

若指定CLEAR参数，指定的缓存变量会被取消标记为高级配置；若指定FORCE参数，则指定的缓存变量不论是否已经被标记，都会被强制标记为高级配置；在不指定这两个参数的默认情况下，该命令只会标记尚未标记的缓存变量为高级配置。

如下所示中是一个实例。

```
cmake_minimum_required(VERSION 3.20)

project(cache-var)

set(MY_CACHE "MY_CACHE" CACHE STRING "")
set(MY_ADVANCED_CACHE "MY_ADVANCED_CACHE" CACHE STRING "")
mark_as_advanced(MY_ADVANCED_CACHE)
```

使用CMake GUI配置该项目后，`MY_CAHCE`默认显示在缓存变量列表中；`MY_ADVANCED_ CACHE`则需要勾选Advanced复选框后才能显示出来。

## 6.3　CMake命令行的使用

至此，我们已经大致了解了CMake命令行在不同阶段中的用法。在本节中，我们将对本章前面介绍的内容进行总结，并介绍更多的可选参数。

### 6.3.1　配置和生成

前面提到过CMake是构建系统生成器。因此，为了配置一个CMake项目并为其生成构建系统的配置文件，需要有如下设置。

- 源文件目录（source directory）。该目录中应当包含该项目全部的源程序文件，包括使用CMake脚本语言编写的CMake目录程序。在源文件目录中（非子目录中）应当有一个CMake目录程序作为顶层CMake目录程序，也就是CMake配置项目的入口。

- 构建目录（build directory），又称二进制目录（binary directory）。该目录会被作为生成构建系统配置文件及构建的二进制文件的目标目录。CMake持久化缓存文件也会输出到构建目录中。

- 生成器（generator），即构建系统生成器。CMake支持生成多种构建系统，因此需要选择一个生成器。 `cmake --help`命令可以列举所有可供选择的生成器及默认的生成器，这通常包括Makefile、Visual Studio等生成器。

构建目录一般会在源文件目录之外，以免污染源文件目录。这种在源文件目录之外进行构建的方式，通常称为源外部编译（out-of-source build）。对应地，在源文件目录中直接生成构建的二进制文件的构建方式，称为源内部编译（in-source build）。

下面看看用于配置和生成项目的CMake命令行该如何书写。其一共有三种调用形式：

```
cmake [<其他选项>...] <源文件目录>
cmake [<其他选项>...] <已存在的构建目录>
cmake [<其他选项>...] -S <源文件目录> -B <构建目录>
```

其中，第一种形式只需指定<源文件目录>，使用当前工作目录作为<构建目录>。

当重新配置和生成一个已经生成过的CMake项目时，还可以使用第二种形式，仅指定 <已存在的构建目录>。

第三种形式是设置最全的。由于它不与当前工作目录相关，常常用于自动化构建脚本。

#### 其他常用选项

<其他选项>有很多可选参数可供设置，在4.8节中就介绍过一些控制日志输出的参数，这里仅再列举两个较为常用的参数。完整的参数选项请参考CMake官方文档。

```
-G <生成器名称>
```

上述参数用于指定构建系统生成器的名称。若该参数被省略，CMake会先尝试读取`CMAKE_GENERATOR`环境变量中指定的值。若其值仍不存在，则选用默认的生成器。

执行`cmake --help`命令可以查看CMake支持的全部构建系统生成器。

```
-D <变量>:<类型>=<值>
-D <变量>=<值>
```

上述参数可以指定多次，用于创建或更新若干CMake缓存。该参数定义的CMake缓存值具有最高优先级，也会修改持久化缓存文件CMakeCache.txt，覆盖其中对应缓存变量的值。

若在指定该参数时省略<类型>，则对应缓存变量的类型由持久化缓存文件中定义的类型为准；若持久化缓存文件尚未生成，则根据CMake程序中set命令定义的同名缓存变量的类型来确定；若同名缓存变量从未被set定义过，则认为其没有类型（即文本类型）。若该参数定义的缓存变量被确定为PATH或FILEPATH类型，其值会被转换为绝对路径。

另外，该参数`-D`与后面变量定义之间的空格可以省略，直接作为一个独立的参数整体，例如，`-D<变量>=<值>`。

### 6.3.2　构建

介绍构建阶段时，已经调用过`cmake --build`命令行形式了，其完整形式如下：

```
cmake --build <构建目录> [<选项>...] [-- <传递给构建系统的选项>...]
```

由于CMake构建命令行实际上会调用其生成的构建系统来完成构建，因此可以看到，这里有两组参数选项：用于CMake构建命令行本身的<选项>参数和<传递给构建系统的选项>参数。其中，<传递给构建系统的选项>参数的取值取决于在CMake配置和生成阶段选用的哪种构建系统，与CMake无关。这里仅介绍部分比较常用的<选项>参数取值。

#### 常用选项

下面这些常用选项均用于CMake构建命令的<选项>参数。

```
--parallel [<并行数量>]
-j [<并行数量>]
```

上述参数用于指定构建项目的最大并行进程数量。<并行数量>参数部分可以省略，省略后，其取值会先尝试从`CMAKE_BUILD_PARALLEL_LEVEL`环境变量中读取。若该环境变量未定义，则默认为底层构建系统采用的默认并行数量。

```
--target <目标名称>...
-t <目标名称>...
```

上述参数用于构建<目标名称>参数指定的若干目标，多个<目标名称>参数应当使用空格隔开。省略该参数则构建默认构建目标，即构建全部。

```
--config <构建模式>
```

对于支持多构建模式的底层构建系统来说，构建时需通过<构建模式>参数指定采用何种构建模式进行构建，其取值通常为Debug、Release、RelWithDebInfo、MinSizeRel之一。

```
--clean-first
```

上述参数要求构建系统在构建目标之前先清理项目生成的文件，即执行clean指令构建目标。

```
--verbose
-v
```

上述参数用于开启详细的构建日志输出，一般会包含构建的每一步执行的命令行。

若定义了VERBOSE环境变量或`CMAKE_VERBOSE_MAKEFILE`缓存变量被设置为真值，CMake也会启用详细的构建日志输出。

### 6.3.3　打开生成的项目

如果采用的构建系统生成器是对应某个集成开发环境的（如Visual Studio、Xcode等生成器），那么生成项目后，可以通过CMake命令行直接用对应的集成开发环境打开项目：

```
cmake --open <构建目录>
```

### 6.3.4　安装

在前面介绍安装阶段时已经调用过`cmake --install`命令行了。其完整形式如下：

```
cmake --install <二进制目录> [<选项>...]
```

<二进制目录>参数实际上就是构建目录。通常，在配置生成和构建时会将该目录称为 “构建目录”，而在构建出二进制文件后称为“二进制目录”。

#### 常用选项

下面是用于`cmake --install`命令行的常用选项。

```
--config <构建模式>
```

对于支持多构建模式的构建系统来说，上述参数用于安装以指定<构建模式>构建的二进制文件。

```
--component <组件名称>
```

指定安装的<组件名称>后，CMake安装命令行仅会安装对应组件的二进制文件。组件名称可以通过CMake的install命令的COMPONENT参数设置，具体用法参见CMake官方文档。

```
--prefix <安装目录前缀>
```

上述参数用于指定<安装目录前缀>。CMake在进行安装时，所有二进制文件都会安装到相对于指定<安装目录前缀>的子目录中。

```
--strip
```

指定上述参数后，CMake在安装前会先清除二进制文件中所包含的调试信息。

```
--default-directory-permissions <默认目录权限>
```

上述参数用于指定安装目录的默认权限。<默认目录权限>的格式为`u=rwx`,`g=rx`,`o=rw`，其中等号前的u、g、o分别代表所有者、所在组和任何人，等号后的r、w、x 则分别代表读权限、写权限和执行权限。

```
--verbose
-v
```

上述参数用于启用详细日志输出。若VERBOSE环境变量被设置为真值，CMake 也会启用详细日志输出。

### 6.3.5　内置命令行工具

CMake还提供了一些内置命令行工具，用于替代常用的Shell命令，以文件系统操作为主。 CMake提供这些命令主要是出于跨平台的考虑，避免执行一些系统功能时，还需要对不同平台的Shell进行适配。前面实例中就曾经使用过`“${CMAKE_COMMAND} -E echo”`这个形式，用于输出指定字符串到标准输出。这些CMake命令行工具通常会配合CMake的`execute_command`等命令使用。CMake命令行工具的形式主体如下：

```
cmake -E <命令> [<命令参数>...]
```

下面会把<命令>和<命令参数>这两个参数放在一起介绍，毕竟二者是不可分割的。例如，在`cmake -E cat a.txt`这个命令行调用中，cat是<命令>参数， a.txt是<命令参数>参数。

#### 输出文件内容

```
cmake -E cat <文件名>
```

该命令用于将指定的多个文件的内容同时输出到标准输出中。

#### 切换当前目录

```
cmake -E chdir <目录> <命令行> [<命令行参数>...]
```

该命令用于切换到指定的<目录>，然后执行指定的<命令行>。

#### 比较文件

```
cmake -E compare_files [--ignore-eol] <文件路径1> <文件路径2>
```

该命令用于比较指定的两个文件是否内容相同。若相同，则该命令的退出码为0；若不同，则为1；若参数不正确，则为2。`--ignore-eol`参数可用于忽略换行符的不同。

#### 复制文件

```
cmake -E copy <文件路径>... <目标路径>
```

该命令用于复制若干<文件路径>指定的文件到 <目标路径>。其中，<目标路径>可以是一个文件路径或目录路径，但若指定了多个要复制的文件，则<目标路径>必须是一个已存在的目录路径。

该命令在复制时会解析符号链接，即复制符号链接指向的文件或目录。

#### 复制目录

```
cmake -E copy_directory <目录路径>... <目标路径>
```

该命令用于复制若干<目录路径>指定的目录到 <目标路径>。其中，<目标路径>必须是一个目录路径（若不存在，该命令会创建该目标目录）。

该命令在复制时会解析符号链接，即复制符号链接指向的目录。

#### 复制修改的文件

```
cmake -E copy_if_different <文件路径>... <目标路径>
```

该命令仅在文件发生改变时复制文件，其复制行为与copy命令一致。

#### 创建符号链接

```
cmake -E create_symlink <指向路径> <符号链接路径>
```

该命令用于创建一个指向<指向路径>的符号链接。<符号链接路径>的所在目录必须存在。

#### 创建硬链接

```
cmake -E create_hardlink <指向路径> <硬链接路径>
```

该命令用于创建一个指向<指向路径>的硬链接。<硬链接路径> 的所在目录和<指向路径>都必须存在。

#### 输出字符串

```
cmake -E echo [<字符串>...]
```

该命令用于将指定的字符串和一个换行符输出到标准输出中。

#### 输出字符串（不换行）

```
cmake -E echo_append [<字符串>...]
```

该命令用于将指定的字符串输出到标准输出中，最后不换行。

#### 设置环境变量并执行命令

```
cmake -E env [--unset=<要删除的环境变量>]... 
             [<环境变量>=<环境变量值>]... 
             <命令行> [<命令行参数>...]
```

该命令用于删除或设置环境变量，然后在新的环境变量设置中执行指定的<命令行>程序。

#### 列举环境变量

```
cmake -E environment
```

该命令用于列举所有环境变量。

#### 创建目录

```
cmake -E make_directory <目录>...
```

该命令用于创建若干<目录>。当指定<目录>的父级目录不存在时，该命令也会递归地创建它们；当指定的<目录> 已存在时，该命令会忽略它。

#### 重命名或移动

```
cmake -E rename <原始路径> <目标路径>
```

该命令用于重命名<原始路径>所对应的文件或目录，或将其移动到<目标路径>。若<目标路径>已存在，该命令会将已存在的文件或目录覆盖。

#### 删除文件或目录

```
cmake -E rm [-rRf] <文件或目录路径>...
```

该命令用于删除若干<文件或目录路径>。

-r或-R参数用于递归删除目录。当指定的<文件或目录路径>不存在时，该命令默认返回非0退出码表示发生错误；但指定-f参数后，则会忽略不存在的路径，退出码为0。

#### 延时

```
cmake -E sleep <秒数>
```

该命令用于延时指定的<秒数>。

#### 打包或提取归档文件

```
cmake -E tar [c|x|t][v][z|j|J] <归档文件路径> 
             [--zstd] 
             [--files-from=<清单文件路径>] 
             [--format=<格式>] 
             [--mtime=<修改时间>] 
             [--] [<文件路径>...]
```

该命令用于打包或提取归档文件，其参数含义分别如下。

- 指定c参数表示打包归档文件。此时<文件路径>参数用于指定需要被打包的文件，是必须提供的参数。

- 指定x参数表示提取归档文件。此时<文件路径>参数可用于筛选需要被提取的文件（用于筛选的文件路径可通过-t参数查看）。

- 指定t参数表示列举归档文件中的文件路径。此时<文件路径>参数可用于筛选需要被列举的文件。

- 指定v参数表示输出详细日志。

- 指定z参数表示使用gzip压缩归档。

- 指定j参数表示使用bzip2压缩归档。

- 指定J参数表示使用XZ压缩归档。

- `--zstd`参数表示使用Zstandard压缩归档。

- <清单文件路径>参数用于指定一个清单文件，该文件内容的每一行是一个文件路径。这些文件路径会作为<文件路径>参数。清单文件中的空行会被忽略，另外文件路径不能以`“-”`开头。如果一定要包含以`“-”`开头的路径，可以通过为该文件路径添加前缀`“--add-file=”`的方式将其加入清单文件中。

- <格式>参数用于指定归档格式，其值可为7zip、gnutar、pax、paxr及zip。

    - pax代表可移植归档交换格式（portable archive exchange）。

    - paxe代表受限可移植归档交换格式（restricted pax），也是默认格式。

- <修改时间>参数用于指定归档中文件的修改时间。

- `--`参数用于分隔<文件路径>参数和其他参数，即在`--`之后的参数都会作为<文件路径>，可用于引用横杠开头的文件路径。

#### 计时执行命令

```
cmake -E time <命令行> [<命令行参数>...]
```

该命令用于执行指定的<命令行>程序，并输出执行时间。

#### Touch文件

```
cmake -E touch <文件>...
```

该命令用于创建指定的<文件>，若其已存在，则将其修改和访问时间更新为当前时间。

#### Touch文件（不创建）

```
cmake -E touch_nocreate <文件>...
```

该命令用于将指定<文件>的修改和访问时间更新为当前时间，若<文件>不存在，则忽略它。

#### 返回成功

```
cmake -E true
```

该命令仅用于返回成功，即其退出码为0。它不做任何其他操作。

#### 返回错误

```
cmake -E false
```

该命令仅用于返回错误，即其退出码为1。它不做任何其他操作。

#### 生成文件校验和

```
cmake -E md5sum <文件>...
cmake -E sha1sum <文件>...
cmake -E sha224sum <文件>...
cmake -E sha256sum <文件>...
cmake -E sha384sum <文件>...
cmake -E sha512sum <文件>...
```

这些命令用于创建指定的若干<文件>的校验和文件。该命令创建的校验和文件格式可以兼容对应算法的校验工具。例如，使用md5sum工具可以校验`cmake -E md5sum`命令生成的校验和。

#### 获取CMake能力

```
capabilities
```

该命令用于获取当前CMake版本所具有的能力，并通过一个JSON对象输出。该JSON对象具有如下键和子键。

- version，这是一个表示CMake版本信息的JSON对象，其子键如下。

    – string，即CMake版本号的字符串表示，如3.20.2。
    
    – major，即CMake主版本号，如3。
        
    – minor，即CMake的次版本号，如20。
        
    – patch，即CMake的修订版本号，如2。
        
    – suffix，即CMake的版本后缀字符串，如rc1。
        
    – isDirty，表示CMake是否从一个修改过的Git目录树中构建。

- generators，这是一个表示受支持的构建系统生成器的JSON对象数组。其中每一个对象的键如下。

    – name，即生成器的名称，如Visual Studio 16 2019。
    
    – toolsetSupport，即该生成器是否支持工具链设置。
    
    – platformSupport，即该生成器是否支持平台设置。
    
    – supportedPlatforms，即该生成器支持的平台名称（当且仅当该生成器支持设置平台时存在该键），例如，Visual Studio 16 2019生成器的supportedPlatforms值包含x64、Win32、ARM和ARM64等。
    
    – extraGenerators，即与该生成器关联使用的附加生成器的名称数组，关于附加生成器的介绍参见6.1.1节。例如，Unix Makefiles生成器的extraGenerators值包含Kate、Sublime Text 2和CodeBlocks等。

- fileApi，这是一个表示受支持的CMake File API的JSON对象。它具有一个子键requests。

    – requests，即表示受支持的File API请求对象的数组，其中每一个对象的键如下。

        ■ kind，即File API请求对象的类型。
        
        ■ version，即File API请求对象的版本号。这是一个JSON数组，每一个元素代表版本号的一部分。

- serverMode，这是一个表示是否支持CMake服务端模式的布尔值。由于CMake服务端已被弃用，在CMake 3.20版本以后，该值总是false。

## 6.4　使用Visual Studio打开CMake项目

### 6.4.1　生成Visual Studio的原生解决方案

使用Visual Studio这个强大的集成开发环境来编辑、调试程序能够极大地提高开发效率。很多构建工具虽然功能强大，但缺少集成开发环境的支持，使得开发效率大大降低，因此很难被广泛的人群采用。CMake作为构建工具的生成器，自然能够更好地解决这一问题——只需使用Visual Studio生成器，将CMake项目配置生成出Visual Studio 支持的解决方案文件。

前面已经介绍过如何使用cmake命令行配置生成项目：

```
> cd CMakeBook/src/ch006/mylib
> mkdir build
> cd build
> cmake -G "Visual Studio 16" ..
...
```

配置生成结束后，build构建目录中会生成出一个名为mylib.sln的解决方案文件。双击打开它，即可启动Visual Studio！在Visual Studio的解决方案资源管理器中，可以看到多个 Visual C++ 项目，它们分别对应该CMake项目的各个构建目标。

其中，`ALL_BUILD`目标表示构建全部，`INSTALL`目标用于安装项目，`mylib`目标即创建的静态库目标，`PACKAGE`目标用于打包项目，`ZERO_CHECK`目标用于在CMake目录程序发生改变时重新配置生成该项目。

不允许通过Visual Studio的用户界面来为项目添加新的源文件（例如，通过项目的右键菜单来为某个构建目标新建源文件），因为这样创建的文件并不会同步到CMakeLists.txt目录程序中，是没有意义的。应当通过手动修改CMakeLists.txt的方式来添加源文件。

### 6.4.2　使用Visual Studio直接打开CMake项目

Visual Studio 2015及以前的版本不支持直接打开CMake项目。由于 CMake逐渐成为业界C和`C++`程序的构建标准，Visual Studio 2017版本终于提供了对CMake的原生支持。

启动Visual Studio，依次选择“文件”→“打开”→“CMake”（或“文件夹”）。在弹出的打开文件对话框中选择CMake目录程序文件（若刚刚执行的是“文件夹”命令，应选择CMake目录程序所在目录）即可打开该项目。

有关Visual Studio直接打开CMake项目后提供的各种功能的具体使用方法，请参考Visual Studio的官方文档。

Visual Studio相当于对CMake做了一定的封装，通过用户界面来提供对CMake的各种操作，对初学者比较友好。因此建议先使用第二种方式打开CMake项目。

不过，由于CMake确实非常复杂，Visual Studio提供的封装可能不够全面，有时也不够稳定。如果使用Visual Studio直接打开CMake项目时遇到了问题，那么不妨先生成项目的解决方案后再用Visual Studio打开CMake项目。

## 6.5　小结

本章带领读者体验了CMake项目从构建到使用的完整生命周期，尽管并未涉及很多细节，但也希望本章能够帮助读者对CMake在构建过程中发挥的作用建立宏观的认识。

另外，本章也详细地介绍了CMake命令行工具和GUI工具的使用方法，以及如何在Visual Studio中打开CMake项目。读者阅读完本章后，应当已经可以通过CMake完成简单项目的配置生成和构建了。不妨从互联网上下载一个使用CMake作为构建工具的开源项目，尝试在自己的计算机中构建一下。

既然已经能够使用CMake构建项目，那么，如何使用CMake组织一个自己的项目呢？这正是第7章将带领大家深入了解的内容。请继续阅读吧！
