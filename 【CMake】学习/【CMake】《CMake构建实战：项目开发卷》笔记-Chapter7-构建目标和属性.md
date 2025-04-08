# 第7章　构建目标和属性

本章重点关注CMake的构建目标和属性，它们是用来组织项目构建流程的核心概念。毫不夸张地说，如果学习CMake的目标就是组织简单的C和`C++`小项目的构建流程，那么阅读掌握本章内容就足够了。

本章与第1章的“旅行笔记”遥相呼应，将会逐一介绍如何在CMake 的目录程序（CMakeLists.txt）中定义各种类型的构建目标，包括可执行文件、静态库、动态库、接口库及伪目标等。可以在第1章中找到这些基本概念的对应内容。

## 7.1　二进制构建目标

还记得二进制构建目标吗？在构建过程中被构建的可执行文件、库文件或目标文件都可以作为二进制构建目标。

二进制构建目标是全局可见的。也就是说，无论在哪个子目录的CMake目录程序中创建了一个二进制构建目标，都可以被当前项目中的其他所有CMake目录程序访问到。

下面将依次介绍各种二进制构建目标的定义方法。

### 7.1.1　可执行文件目标

第6章的实例中曾经创建过可执行文件类型的构建目标，当时也正是使用的`add_executable`命令。该命令的参数形式如下：

```
add_executable(<目标名称> 
    [WIN32] [MACOSX_BUNDLE] 
    [EXCLUDE_FROM_ALL]
    [<源文件>...]
)
```

该命令会创建一个可执行文件类型的构建目标，其中第一个参数<目标名称>是必选参数，且应当在项目中唯一。

目标名称并不一定是最终可执行文件的名称。最终生成的文件名可以通过`OUTPUT_NAME`目标属性来修改，有关属性的内容将在7.5节中介绍。若未定义上述属性，在默认情况下，其文件名会根据目标名称，结合当前平台的习惯确定。例如，可执行文件目标myProgram在Windows中构建后的可执行文件的名称默认为myProgram.exe。

<源文件>参数用于指定该可执行文件目标的源文件路径。该参数可以被暂时省略，CMake 允许先创建构建目标，再通过`target_sources`命令设置所需的源程序，参见7.6.15小节。

WIN32参数用于将该目标的`WIN32_EXECUTABLE`属性设置为真值。该属性仅作用于Windows中，表示使用WinMain而不是main作为入口函数，常用于图形界面程序。若省略该参数，则该属性值取决于CMake变量`CMAKE_WIN32_EXECUTABLE`的值，默认为假值。

`MACOSX_BUNDLE`参数用于将该目标的`MACOSX_BUNDLE`属性设置为真值。该属性仅作用于Apple平台中，表示将可执行文件构建为应用程序Bundle，常用于图形界面程序。若省略该参数，则该属性值取决于CMake变量`CMAKE_MACOSX_BUNDLE`的值。当目标构建系统（即`CMAKE_SYSTEM_NAME`变量的值）为iOS、tvOS或watchOS时，`CMAKE_MACOSX_BUNDLE`变量的值默认为真值，否则为假值。

`EXCLUDE_FROM_ALL`参数用于将该目标的`EXCLUDE_FROM_ALL`属性设置为真值。该属性表示是否将当前目标排除在表示构建全部的目标（`all`或`ALL_BUILD`）之外。当目标的`EXCLUDE_FROM_ALL`属性为真值时，它在项目构建时默认不会被构建，除非通过命令行参数显式地指定该构建目标，如使用`cmake --build`命令行的`--target`参数指定该构建目标。

这里涉及一些与定义构建目标的命令参数紧密相关的目标属性，因此放在这里介绍。本章会在7.5节中详细讲解CMake属性相关的内容。

#### 实例：创建可执行文件构建目标

该实例项目的CMake目录程序如下所示，其中定义了两个可执行文件构建目标。二者的主程序相同，如下所示。这两个目标的唯一不同是myProgramExcludedFromAll目标在被定义时指定了`EXCLUDE_FROM_ALL`参数。

```
cmake_minimum_required(VERSION 3.20)

project(myProgram)

add_executable(myProgram main.c)

add_executable(myProgramExcludedFromAll 
    EXCLUDE_FROM_ALL
    main.c
)
```

```
#include <stdio.h>

int main() { printf("Hello\n"); }
```

下面用Visual Studio生成器按照标准流程构建它：

```
> cd CMakeBook/src/ch007/可执行文件
> mkdir build
> cd build
> cmake ..
...
> cmake --build .
...
myProgram.vcxproj -> ...\Debug\myProgram.exe
> ls Debug
myProgram.exe  myProgram.pdb
```

构建结果中只有可执行文件myProgram.exe及其pdb符号文件，也就是说，只有 myProgram目标被构建。如果想要构建myProgramExcludedFromAll目标，需要手动指定它：

```
> cmake --build . --target myProgramExcludedFromAll
...
myProgramExcludedFromAll.vcxproj -> ...\Debug\myProgramExcludedFromAll.exe
> ls Debug
myProgram.exe  myProgram.pdb
myProgramExcludedFromAll.exe  myProgramExcludedFromAll.pdb
```

### 7.1.2　一般库目标

一般库目标包括静态库目标、动态库目标和模块库目标，其定义形式如下。

```
add_library(<目标名称> <库类型>
    [EXCLUDE_FROM_ALL]
    [<源文件>...]
)
```

该命令会创建一个一般库类型的构建目标，其中<目标名称>参数是必选参数，且应当在项目中唯一。与可执行文件目标类似，一般库目标最终构建的库文件也是结合当前平台的习惯来命名的，如目标名称为mylib的静态库，在Windows中构建的库文件名称为mylib.lib，在Linux中构建的库文件名称则为libmylib.a。

<库类型>参数有以下三个取值。

- STATIC，代表该构建目标为静态库构建目标。

- SHARED，代表该构建目标为动态库构建目标。动态库构建目标的`POSITION_ INDEPENDENT_CODE`属性会被设置为真值，以支持地址无关代码，相关原理参见第1章。

- MODULE，代表该构建目标为模块库构建目标。模块库是一种插件形式的动态链接库，不会在构建时被链接到任何一个程序中，仅用于运行时动态链接（通过LoadLibrary或dlopen等API）。模块库构建目标的`POSITION_INDEPENDENT_CODE`属性也会被设置为真值。

若省略<库类型>参数，则目标库类型取决于CMake变量`BUILD_SHARED_LIBS`的值：若其值为真，则该构建目标为动态库构建目标，否则为静态库构建目标。

如果一个库不导出任何符号名称，则它不能被声明为动态库（SHARED）。例如，Windows中存在一些资源动态链接库（resource DLL）。对于这类库目标，应当使用模块库类型（MODULE）。因为CMake在Windows中使用动态库时，要求该动态库拥有对应的导入库lib文件，而资源动态链接库这类库文件是不具有导入库文件的。

`EXCLUDE_FROM_ALL`和<源文件>参数的含义与定义可执行文件目标时一致，这里不再赘述。

#### 实例：创建一般库构建目标

下面这个实例中没有显式地指定一般库目标的<库类型>参数，而是使用CMake变量`BUILD_SHARED_LIBS`来决定该目标的类型。库源文件和CMake目录程序分别如下所示。

```
int add(int a, int b) { return a + b; }
```

```
cmake_minimum_required(VERSION 3.20)

project(myLib)

add_library(myLib lib.c)
```

首先，在不定义`BUILD_SHARED_LIBS`变量的情况下构建它：

```
> cd CMakeBook/src/ch007/一般库
> mkdir build
> cd build
> cmake ..
...
> cmake --build .
...
myLib.vcxproj -> ...\Debug\myLib.lib
　
> ls Debug
myLib.lib  myLib.pdb
```

可以看到，不定义`BUILD_SHARED_LIBS`变量时，默认构建静态库，构建目录中只有静态库文件myLib.lib及其符号文件。

下面定义`BUILD_SHARED_LIBS`变量为真值，然后配置生成并构建CMake项目：

```
> rm Debug/myLib.* # 删除刚刚构建的静态库及其符号文件
> cmake -DBUILD_SHARED_LIBS=ON ..
...
> cmake --build .
...
myLib.vcxproj -> ...\Debug\myLib.dll
　
> ls Debug
myLib.dll  myLib.pdb
```

果然，这次构建出的就是动态库及其符号文件了。

### 7.1.3　目标文件库目标

目标文件库类型的构建目标仅编译其包含的源文件，生成一系列目标文件，并不会将这些目标文件打包或链接到某个库文件中。因此目标文件库是一个逻辑上的概念，实际是很多目标文件的集合。创建一个目标文件库目标同样使用`add_library`命令，参数形式如下：

```
add_library(<目标名称> OBJECT
    [<源文件>...]
)
```

该命令会创建一个目标文件库的构建目标，其参数与创建一般库目标时基本一样。

如果想在构建其他可执行文件或库时链接目标文件库对应的目标文件，只需在`add_executable`和`add_library`命令的<源文件>参数中指定下面这个表达式：

```
$<TARGET_OBJECTS:<目标文件库的目标名称>>
```

如下所示例程中，可执行文件main目标将链接 myObjLib目标文件库构建的目标文件（即a.c和b.c构建的目标文件）。

```
cmake_minimum_required(VERSION 3.20)

project(myObjLib)

add_library(myObjLib OBJECT a.c b.c)
add_executable(main main.c $<TARGET_OBJECTS:myObjLib>)
```

### 7.1.4　指定源文件的方式

本节介绍的二进制构建目标均涉及源文件的构建，因此需要指定<源文件>参数。有时候，某个构建目标可能对应非常多的源文件，难道都要手动一一罗列吗？

答案是最好如此。如果不想深究其原理，那就一一罗列吧！

当然，很多开源项目或网络文章中常有使用`file(GLOB)`命令或` aux_source_directory`命令等获取目录中的全部源文件并将它们传入<源文件>参数的案例。一般来说，如果项目是只读的，不会为构建目标添加新的源文件，那么这样做是没有问题的。但一旦要为构建目标添加新的源文件，问题就来了。CMake怎么知道构建目标添加了新的源文件呢？毕竟，CMakeLists.txt没有改变，CMake就不会重新配置生成项目，那么新建源文件之后尝试构建项目时，它总是会提示项目已经是最新的，无须构建。这时就只能手动重新配置生成CMake项目，然后构建项目了。

当然，`file(GLOB)`提供了`CONFIGURE_DEPENDS`参数，用以每次构建时重新执行遍历操作，并在遍历结果发生变化时重新生成项目。没错，但这只是理想情况。现实是，`CONFIGURE_ DEPENDS`参数并不支持全部构建系统生成器。另外，每次构建都去遍历文件是一个非常耗时的操作。

因此，最佳实践是永远一一罗列全部源文件。这样，添加新的源文件必然导致CMakeLists.txt修改。构建项目时，也就会重新配置生成该项目，源文件的变动自然也会被检测到，从而导致相应部分的重新构建。

#### 遍历目录中的源文件：aux_source_directory

尽管遍历源文件作为参数不是最佳实践，但这里还是有必要介绍一下这个命令。毕竟有时候还是会使用它，而且社区中也有很多人已经在使用它。

```
aux_source_directory(<目录> <结果变量>)
```

该命令用于遍历指定<目录>中的源文件，并将它们的路径存入<结果变量>。

## 7.2　伪构建目标

伪构建目标，指那些并不会被构建二进制文件的目标，通常用于表明仅具有使用要求的可链接的对象。伪构建目标分为接口库目标、导入目标和别名目标三种类型。除导入目标类型的伪构建目标外，其他类型的伪构建目标均为全局可见。

### 7.2.1　接口库目标

接口库目标通常用于对头文件库的抽象，同样使用`add_library`命令创建。

```
add_library(<目标名称> INTERFACE)
```

该命令会创建一个目标文件库的构建目标，只有<目标名称>和INTERFACE两个参数。毕竟，接口库自身并不需要被构建，也就无须指定源文件。读者可以回顾第1章中的内容。

接口库虽然不需要被构建，却有责任声明使用要求。7.5节中会详细介绍有关目标属性、构建要求和使用要求的内容。这里仅演示如何为头文件库创建构建目标。

首先，新建一个目录include，在该目录中创建一个头文件a.h，如下所示。

```
static int add(int a, int b) { return a + b; }
```

然后，回到项目顶层目录中，创建主程序main.c，如下所示。

```
#include <a.h>
#include <stdio.h>

int main() {
    printf("%d\n", add(1, 2));
    return 0;
}
```

本实例的CMake目录程序如下所示。

```
cmake_minimum_required(VERSION 3.20)

project(myInterfaceLib)

add_library(myInterfaceLib INTERFACE)

# 声明myInterfaceLib的使用要求：头文件搜索目录应为include
target_include_directories(myInterfaceLib INTERFACE include)

add_executable(main main.c)

# 声明main的构建要求：链接myInterfaceLib库，也就是说
# 将myInterfaceLib接口库的使用要求作为main的构建要求
target_link_libraries(main myInterfaceLib)
```

其中，`target_include_directories`命令用于声明有关头文件搜索目录的要求，`target_link_libraries`命令用于声明有关链接库的要求。7.6节中会详细介绍这些命令。

### 7.2.2　导入目标

导入目标，顾名思义，指导入项目之外的可执行文件或库的目标，但不会构建它。可以想象，导入目标必然有一个与路径相关的属性，指向其导入的那个可执行文件或库。

#### 可执行文件导入目标

使用`add_executable`命令可以创建一个可执行文件导入目标，对应参数形式如下。

```
add_executable(<目标名称> IMPORTED [GLOBAL])
```

默认情况下，导入目标的作用域为当前目录及其子目录的目录程序。也就是说，在当前目录的CMake目录程序中创建的导入目标，默认不能被其上级目录的目录程序访问。GLOBAL参数则用于打破这个限制，使该目标全局可见。

不过，可执行文件导入进来有什么用？我们又不会链接一个可执行文件！

既然是可执行文件，那当然是用来执行的了！7.8节会介绍如何创建自定义命令形式的构建目标，用于在构建阶段中调用可执行文件。如果在自定义构建规则的命令行参数中提供的不是命令路径，而是一个可执行文件目标的名称，那么它会自动找到目标对应的路径来完成调用。如此一来，不论是调用自己构建的可执行文件，还是外部的可执行文件，都能以统一的目标形式实现。

可执行文件导入目标对应的实际路径通过`IMPORTED_LOCATION`目标属性来设置。设置属性的方法将在7.5节详细讲解，这里先看一个简单的例程，目录程序如下所示。

```
cmake_minimum_required(VERSION 3.20)

project(import-notepad)

add_executable(notepad_exe IMPORTED)
set_target_properties(notepad_exe PROPERTIES 
    IMPORTED_LOCATION "C:/Windows/System32/notepad.exe")

add_custom_target(run-notepad ALL notepad_exe ${CMAKE_CURRENT_LIST_FILE})
```

该例程中包含一个可执行文件导入目标`notepad_exe`，其 `IMPORTED_LOCATION`属性设置为记事本应用程序所在的路径。最后通过 `add_custom_target`命令创建了一个自定义构建规则的构建目标`run-notepad`，该自定义构建目标会调用`notepad_exe`可执行文件目标，调用参数为当前CMake目录程序的路径。7.8节会详细介绍这个命令。

在Windows中尝试配置生成并构建该项目：

```
> cd CMakeBook/src/ch007/可执行文件导入目标
> mkdir build
> cd build
> cmake ..
...
> cmake --build .
...
```

当执行`cmake --build .`命令后，记事本应用程序将打开，显示本例程的CMake目录程序。

#### 库导入目标

使用`add_library`命令可以创建一个库导入目标，对应参数形式如下。

```
add_library(<目标名称> 
    <STATIC|SHARED|MODULE|UNKNOWN|OBJECT|INTERFACE> 
    IMPORTED [GLOBAL])
```

默认情况下，库导入目标的作用域也是当前目录及其子目录的目录程序。命令中的 GLOBAL参数同样用于使该目标全局可见。

STATIC、SHARED、MODULE参数分别用于导入外部的静态库、动态库和模块库。对于这三种类型的库导入目标，目标属性`IMPORTED_LOCATION`用于指定导入库所在路径。

不过在Windows中，动态库导入目标的`IMPORTED_IMPLIB`属性应指定为动态库对应导入库的路径，而`IMPORTED_LOCATION`属性（动态库自身的路径）则是可选的。在支持为动态库设置SONAME的平台，如Linux操作系统中，动态库导入目标的 `IMPORTED_SONAME`属性也应当正确设置。如果动态库未设置SONAME，则应当设置其导入目标的 `IMPORTED_NO_SONAME`属性为真值。

UNKNOWN参数表示导入目标的库类型不确定，一般仅用于自动查找依赖库的功能模块中。

OBJECT参数用于导入外部的目标文件库，其`IMPORTED_OBJECTS`属性应指定为目标文件的路径。

INTERFACE参数用于导入外部的接口库。接口库本身没有对应的二进制文件，因此无须为它指定任何以`IMPORTED_`开头的属性，仅设置其使用要求即可。

下面的实例将复用1.5.4小节中引用Boost Regex静态库的程序，将其Makefile构建脚本改写为CMake目录程序，如下所示。

```
cmake_minimum_required(VERSION 3.20)

project(import-boost)

add_library(boost-regex STATIC IMPORTED)

if(WIN32)
    set(BOOST_REGEX_LIB_PATH 
        "C:/boost/stage/lib/libboost_regex-vc142-mt-gd-x64-1_74.lib")
    set(BOOST_INCLUDE_DIR "C:/boost")
else()
    set(BOOST_REGEX_LIB_PATH 
        "$ENV{HOME}/boost/stage/lib/libboost_regex.a")
    set(BOOST_INCLUDE_DIR "$ENV{HOME}/boost")
endif()

# 设置boost-regex目标的头文件目录的使用要求
target_include_directories(boost-regex INTERFACE ${BOOST_INCLUDE_DIR})

# 设置boost-regex目标的导入路径
set_target_properties(boost-regex PROPERTIES 
    IMPORTED_LOCATION "${BOOST_REGEX_LIB_PATH}")

add_executable(main "../../ch001/09.链接Boost/main.cpp")

# 声明main的构建要求：链接boost-regex库，也就是说
# 将boost-regex导入库的使用要求作为main的构建要求
target_link_libraries(main boost-regex)
```

目录程序中首先利用`add_library(... IMPORTED)`命令创建了库导入目标，然后通过`target_include_ directories`命令对头文件目录的使用要求进行设置，通过`set_target_properties`命令设置了`IMPORTED_LOCATION`属性（即导入库的实际路径）。关于目标属性和使用要求的设置方法参见7.5节和7.6节。

若想成功构建该实例，请先根据第1章中讲解的相关步骤下载并构建安装Boost库。

#### 按构建模式导入

很多第三方软件包都会同时提供调试模式和发布模式构建的两份二进制文件。当发生错误时，调试模式的二进制文件更易于调试；当发布程序时，发布模式的二进制文件能够提供最佳的性能。然而，前面介绍的`IMPORTED_LOCATION`、 `IMPORTED_IMPLIB`、`IMPORTED_OBJECTS`等属性均不区分当前构建模式。那么，该如何为不同构建模式导入对应模式下构建的第三方库或可执行文件呢？

首先，将导入目标支持的全部构建模式设置到导入目标的`IMPORTED_CONFIGURATIONS`属性值：

```
add_library(test SHARED IMPORTED)
set_property(TARGET test PROPERTY
    IMPORTED_CONFIGURATIONS DEBUG;RELEASE
)
```

然后，即可分别为指定的构建模式设置`IMPORTED_LOCATION`等属性了。换句话说，只需在原属性名称后追加构建模式的后缀，如`IMPORTED_LOCATION_DEBUG`和`IMPORTED_LOCATION_ RELEASE`，然后分别为它们设置不同的文件路径：

```
set_target_properties(test PROPERTIES
    IMPORTED_LOCATION_DEBUG "testd.so"
    IMPORTED_LOCATION_RELEASE "test.so"
)
```

### 7.2.3　别名目标

别名目标即另一个构建目标的别名，是只读的构建目标。

有时候，我们希望在构建程序时可以切换依赖的可执行文件或库文件的版本。例如，某个依赖既可以是由项目自带的代码构建的版本，又可以是从外部导入的版本，那么使用别名目标就能方便地满足这一需求：

```
add_executable(<目标名称> ALIAS <指向的实际目标名称>)
add_library(<目标名称> ALIAS <指向的实际目标名称>)
```

上面这两个命令都可以用于创建指向<指向的实际目标名称>的别名目标。当别名目标指向非全局的导入目标时，其作用域同样会被限制在当前目录及其子目录中。

下面将使用别名目标，解决刚才提到的可切换内部或外部构建的依赖库的问题。CMake目录程序如下所示。

```
cmake_minimum_required(VERSION 3.20)

project(alias-target)

add_library(my_liba STATIC "a.cpp")

if(USE_EXTERNAL_LIBA)
    add_library(liba STATIC IMPORTED)
    set_target_properties(liba PROPERTIES IMPORTED_LOCATION "liba.lib")
else()
    add_library(liba ALIAS my_liba)
endif()

add_executable(main "main.cpp")
target_link_libraries(main liba)
```

目录程序中首先定义了一个静态库目标`my_liba`，然后通过变量`USE_EXTERNAL_LIBA`指定是否使用外部的liba库：若其值为真，则导入外部的liba.lib库，并将库导入目标命名为liba；若其值为假，则不导入任何库，仅为my_libs库目标创建别名目标liba。最后，不论是使用内部还是外部构建的liba库，在构建主程序main时让主程序直接链接liba即可。

## 7.3　子目录

组织源程序文件时，往往都需要创建一些子目录使其结构变得清晰。例如， include目录常用于存放公开的头文件，src目录常用于存放源程序文件，thirdparty目录常用于存放第三方依赖库等。

不同的源程序被安排到了不同的目录，正是因为它们彼此用途不同。而被安排在同一目录的源程序，则自然拥有很多共性。换句话说，同一个目录中的源程序很可能会有相同的构建要求和使用要求。CMake中的每一个目录都可以具有独立的目录属性，这样可以方便地针对不同子目录分别进行构建配置。

### 加入子目录：add_subdirectory

要将子目录加入到要构建的项目中，可以使用`add_subdirectory`命令：

```
add_subdirectory(<源文件目录> [<二进制目录>] [EXCLUDE_FROM_ALL])
```

该命令用于将<源文件目录>这个子目录加入项目。该子目录中必须含有一个CMake目录程序，即CMakeLists.txt。当CMake执行该命令时，会立即进入子目录中执行这个目录程序，而当前目录程序（即调用该命令的目录程序）的执行会被暂停，直到子目录的目录程序执行结束。

<源文件目录>可以为绝对路径或相对于当前源文件目录的相对路径。<二进制目录>也可以为绝对路径或相对于当前二进制目录的相对路径。<二进制目录>即存放输出的二进制的目录，省略该参数时，其值根据<源文件目录>的相对位置决定。有关源文件目录和二进制目录的详细介绍参见6.3.1小节。

`EXCLUDE_FROM_ALL`参数的作用与创建构建目标时的同名参数一致：指定该参数意味着子目录中定义的所有构建目标不会被包括在父目录的“构建全部”目标中。如果想构建这个子目录中的构建目标，就必须显式地指定它们。不过有一个例外：如果父目录中的其他构建目标依赖了该子目录中的某个构建目标，那么被依赖的构建目标仍然会被默认构建。

## 7.4　项目：project

项目（project）也是CMake中组织项目的一个逻辑概念。在CMake目录程序中，可以有若干项目。定义在CMake顶层目录程序中的第一个项目称为顶层项目。定义项目的命令有如下两种形式，当无须声明项目的各种属性时，第一种形式最简便。

```
project(<项目名称> [<编程语言>...])
project(<项目名称>
        [VERSION <主版本号>[.<次版本号>[.<补丁版本号>[.<修订版本号>]]]]
        [DESCRIPTION <项目描述>]
        [HOMEPAGE_URL <项目主页URL>]
        [LANGUAGES <编程语言>...])
```

其中，<编程语言>参数支持C、CXX（即`C++`）、CUDA、OBJC（即Objective-C）、 OBJCXX（即`Objective-C++`）、Fortran、HIP、ISPC和ASM。CMake会在配置阶段检查相应的编译环境等。

通过该命令定义的项目属性，均可以通过CMake变量获取，参见下表。下表中变量名称的<项目名称>部分除了填写具体的项目名称外，还可以填写 `PROJECT`和`CMAKE_PROJECT`，它们分别表示获取当前最近项目和顶层项目的相关属性。

| 变量 | 描述 |
| --- | --- |
| `<项目名称>_SOURCE_DIR` | 指定项目源文件目录的绝对路径 |
| `<项目名称>_BINARY_DIR` | 指定项目二进制文件目录的绝对路径 |
| `<项目名称>_VERSION` | 指定项目的版本号 |
| `<项目名称>_VERSION_MAJOR` | 指定项目的主版本号 |
| `<项目名称>_VERSION_MINOR` | 指定项目的次版本号 |
| `<项目名称>_VERSION_PATCH` | 指定项目的补丁版本号 |
| `<项目名称>_VERSION_TWEAK` | 指定项目的修订版本号 |
| `<项目名称>_DESCRIPTION` | 指定项目的描述文本 |
| `<项目名称>_HOMEPAGE_URL` | 指定项目的主页URL |

另外，还有两个用于获取项目名称的变量：

- `PROJECT_NAME`，用于获取当前最近项目的名称；

- `CMAKE_PROJECT_NAME`，用于获取顶层项目的名称。

### 代码注入

CMake项目支持一项“黑科技”：代码注入。通过设置几个CMake变量的值为CMake脚本程序或模块程序的路径，就可以实现向CMake项目定义的前后注入代码，参见下表。

| 变量 | 描述 |
| --- | --- |
| `CMAKE_PROJECT_INCLUDE_BEFORE` | 将CMake程序注入所有项目定义前 |
| `CMAKE_PROJECT_INCLUDE` | 将CMake程序注入所有项目定义后 |
| `CMAKE_PROJECT_<项目名称>_INCLUDE_BEFORE` | 将CMake程序注入指定项目定义前 |
| `CMAKE_PROJECT_<项目名称>_INCLUDE` | 将CMake程序注入指定项目定义后 |

若同时指定了以`CMAKE_PROJECT_`和`CMAKE_PROJECT_<项目名称>_`开头的变量，则以 `CMAKE_PROJECT_`开头的变量所对应的代码会先被注入。

## 7.5　属性：get_property、set_property

我们已经见过了一些属性，现在，一起正式地了解一下属性吧！CMake中的属性根据作用域分为以下7种类型：

- 全局属性；

- 目录属性；

- 目标属性；

- 源文件属性；

- 缓存变量属性；

- 测试属性；

- 安装文件属性。

本书不涉及测试、安装等内容，因此本节将仅介绍前5种作用域的属性。另外，CMake属性其实非常多，本节仅讲解部分常用属性，以及设置属性的方法等。若想了解CMake提供的全部属性，读者可以自行查阅CMake官方文档。

### 7.5.1　全局属性

全局属性（global property）即CMake进程所具有的属性，通常用于获取一些全局状态。

#### 获取全局属性

```
get_cmake_property(<结果变量> <全局属性>)
get_property(<结果变量> GLOBAL PROPERTY <全局属性> 
    [SET|DEFINED|BRIEF_DOCS|FULL_DOCS])
```

这两个命令均可用于获取<全局属性>，并将值结果存入<结果变量>。当指定属性不存在时，第一个命令中的结果变量会被赋值为NOTFOUND，而第二个命令中的结果变量会被赋空值。

事实上，`get_property`命令支持对各种属性的获取，后面在介绍其他类型的属性时也会用到该命令。这里的GLOBAL就是用于获取全局属性时特有的参数。最后的可选参数是通用参数，可用于获取各种类型的属性。其取值含义如下。

- 指定SET参数时，结果变量的值为一个布尔值，表示指定的属性是否被设置。

- 指定DEFINED参数时，结果变量的值为一个布尔值，表示指定的属性是否为自定义属性。自定义属性的相关内容参见7.5.7小节。

- 指定`BRIEF_DOCS`或`FULL_DOCS`参数时，结果变量将被赋值为指定属性的简介文档或完整文档。若该属性不是自定义属性，则不存在文档，结果变量将被赋值为NOTFOUND。

#### 设置全局属性

```
set_property(GLOBAL [APPEND] [APPEND_STRING] 
    PROPERTY <全局属性> [<属性值>...])
```

该命令用于将<全局属性>的值设为<属性值>，多个<属性值>会被视为一个列表。

指定APPEND参数后，<属性值>列表中的每个非空元素将依次被追加到指定属性原值列表的末尾，即属性原值不会清空。指定APPEND_STRING参数时，指定属性的原值与将要追加的<属性值>之间将不会插入分号，也就是直接进行字符串追加。

例如，假设全局属性A的原值为a，指定不同参数设置属性值的结果分别如下。

- `set_property(GLOBAL PROPERTY A x y)`将覆盖其值为`x;y`。

- `set_property(GLOBAL APPEND PROPERTY A x y)`将追加元素，即设置其值为`a;x;y`。

- `set_property(GLOBAL APPEND_STRING PROPERTY A x y)`则会进行字符串追加，将其值设置为`ax;y`。

另外，就像`get_property`命令一样，`set_property`命令同样是多种属性通用的，后面还会多次介绍它。

#### 实例：全局属性

事实上，CMake的全局属性很少会被用到。下面这个实例将使用两个不同的命令来获取 `GENERATOR_IS_MULTI_CONFIG`这个只读的全局属性。这是一个比较常用的全局属性，可用于判断当前构建系统生成器是否为多构建模式的生成器（如Visual Studio和Xcode生成器）。CMake目录程序如下所示。

```
cmake_minimum_required(VERSION 3.20)

project(global_property)

get_cmake_property(res GENERATOR_IS_MULTI_CONFIG)
message("GENERATOR_IS_MULTI_CONFIG: ${res}")

get_property(res GLOBAL PROPERTY GENERATOR_IS_MULTI_CONFIG SET)
message("GENERATOR_IS_MULTI_CONFIG is SET: ${res}")
```

可以尝试使用不同的构建系统生成器来配置生成该CMake目录程序，然后观察配置过程中输出的值。例如，使用Visual Studio生成器时，其输出内容如下：

```
> cd CMakeBook/src/ch007/全局属性
> mkdir build
> cd build
> cmake ..
-- ...
GENERATOR_IS_MULTI_CONFIG: 1
GENERATOR_IS_MULTI_CONFIG is SET: 1
-- ...
```

获取和设置全局属性的命令也可用于CMake脚本程序。

### 7.5.2　目录属性

目录属性（directory property）即作用于目录（包括子目录）的属性，通常用于为目录中的构建目标统一设置编译选项等构建要求，或者获取目录信息。

#### 获取目录属性

```
get_directory_property(<结果变量> [DIRECTORY <目录>] <目录属性>)
get_property(<结果变量> DIRECTORY [<目录>] PROPERTY <目录属性> 
    [SET|DEFINED|BRIEF_DOCS|FULL_DOCS])
```

这两个命令均用于获取指定<目录>的<目录属性>，并将其值存入<结果变量>。

其中<目录>参数为可选参数，省略它则默认获取当前目录的目录属性。该参数可以为某个子目录，或子目录对应的二进制目录，并且支持绝对路径或相对于当前源文件目录的相对路径。需要注意的是，使用该命令获取某一子目录的<目录属性>时，该子目录必须已经被`add_subdirectory`命令加入当前项目。

当指定的目录属性未被定义时，<结果变量>会被赋空值，不过有一种例外：若<目录属性>具有INHERITED特性，即继承特性，则CMake会继续向上层作用域查找同名属性的定义。上层作用域即父目录作用域。属性值的继承过程会递归进行，直至项目顶层目录；若项目顶层目录作用域中仍未定义该属性，它会继承全局作用域中同名全局属性的值。继承特性的相关内容参见7.5.7小节。

#### 设置目录属性

```
set_directory_properties(PROPERTIES <目录属性> <属性值> 
    [<目录属性> <属性值>]...)
set_property(DIRECTORY [<目录>] [APPEND] [APPEND_STRING] 
    PROPERTY <目录属性> [<属性值>...])
```

这两个命令均用于将<目录属性>的值设为<属性值>。第一个命令仅用于设置当前目录的若干目录属性；第二个命令用于设置指定<目录>的一个目录属性，且其中的多个<属性值>会被视为一个列表。

#### 实例：目录属性

下面这个实例中，分别访问了`COMPILE_DEFINITIONS`和`SUBDIRECTORIES`属性，前者用于定义编译时使用的宏，后者用于获取当前目录程序中加入了哪些子目录。顶层目录的目录程序如下所示，主程序源文件如下所示。

```
cmake_minimum_required(VERSION 3.20)

project(directory_property)

add_subdirectory(a)
add_subdirectory(b)

# 获取目录属性SUBDIRECTORIES的值到res
get_directory_property(res SUBDIRECTORIES)
message("SUBDIRECTORIES: ${res}")
```

```
#include <stdio.h>

int main() {
    printf("DIR: %s\n", DIR);
    return 0;
}
```

子目录a中的目录程序如下所示。

```
# 设置目录属性COMPILE_DEFINITIONS的值为DIR="a"
set_directory_properties(PROPERTIES COMPILE_DEFINITIONS DIR="a")
add_executable(a ../main.c)
```

子目录b中的目录程序如下所示。

```
# 设置目录属性COMPILE_DEFINITIONS的值为DIR="b"
set_directory_properties(PROPERTIES COMPILE_DEFINITIONS DIR="b")
add_executable(b ../main.c)
```

构建该项目，并运行生成的两个可执行文件a和b：

```
> cd CMakeBook/src/ch007/目录属性
> mkdir build
> cd build
> cmake ..
-- ...
SUBDIRECTORIES: /.../CMake-Book/src/ch007/目录属性/a;/.../CMake-Book/src/ch007/目录属性/b
-- ...
> ./a/a
DIR: a
> ./b/b
DIR: b
```

若使用Visual Studio生成器，执行过程中调用可执行文件的路径应为`./a/Debug/a.exe`和`./b/Debug/b.exe`。

可见，对于不同的子目录，编译时的宏定义也是不同的。这就说明设置的目录属性是仅作用于子目录（及其子目录）中的。另外，输出结果中包含顶层目录的CMake目录程序中加入的子目录的绝对路径。

获取和设置目录属性的命令也可用于CMake脚本程序。

### 7.5.3　目标属性

目标属性（target property）即作用于构建目标的属性，通常用于为构建目标设置构建要求或使用要求，也可以用于获取构建目标信息等。

一般来说，用于定义构建目标使用要求的属性会以INTERFACE开头。

#### 获取目标属性

```
get_target_property(<结果变量> <构建目标> <目标属性>)
get_property(<结果变量> TARGET [<构建目标>] PROPERTY <目标属性> 
    [SET|DEFINED|BRIEF_DOCS|FULL_DOCS])
```

这两个命令均用于获取关联到<构建目标>的<目标属性>，并将其值存入<结果变量>。

当指定的目标属性未被定义，且该目标属性不具有INHERITED特性（继承特性）时，第一个命令的结果变量会被赋值为“<结果变量>-NOTFOUND”，而第二个命令的结果变量会被赋空值。若其具有继承特性，CMake会继续向上层作用域查找同名属性的定义；若仍未找到该属性的定义，则将结果变量赋为空值。目标属性的上层作用域即构建目标所在目录的作用域，再向上则与目录属性的上层作用域相同，一直递归查找到项目顶层目录的作用域及全局作用域。

#### 设置目标属性

```
set_target_properties(<构建目标>...
    PROPERTIES <目标属性> <属性值> [<目标属性> <属性值>]...)
set_property(TARGET [<构建目标>...] [APPEND] [APPEND_STRING] 
    PROPERTY <目标属性> [<属性值>...])
```

这两个命令均用于将关联到若干<构建目标>的<目标属性>的值设置为<属性值>，其中第二个命令中多个<属性值>会被视为一个列表。这些构建目标可以是在不同目录程序中定义的，但不可以是别名目标，因为别名目标是只读的构建目标。

#### 实例：目标属性

下面这个实例展示了目标属性在构建过程中发挥的重要作用——既可以定义目标的构建要求，又能定义目标的使用要求。

首先，创建一个liba子目录，用于存放liba库的相关程序文件。在liba子目录中再创建两个子目录include和src，分别用于存放如下所示的静态库的头文件和源文件。

```
const char *fa();
```

```
#include <liba.h>

const char *fa() { return "liba"; }
```

然后，在liba子目录中创建如下所示的目录程序，定义静态库构建目标及其属性。

```
add_library(a STATIC src/liba.c)

# 设置a的目标属性
set_property(TARGET a PROPERTY 
    # 构建要求：将include加入头文件搜索目录
    INCLUDE_DIRECTORIES ${CMAKE_CURRENT_SOURCE_DIR}/include)

# 设置a的目标属性
set_property(TARGET a PROPERTY 
    # 使用要求：将include加入（引用者的）头文件搜索目录
    INTERFACE_INCLUDE_DIRECTORIES ${CMAKE_CURRENT_SOURCE_DIR}/include)
```

同样，在项目顶层目录中创建一个子目录include，用于存放主程序所使用的头文件。如下所示，该头文件中定义了一个函数f。

```
const char *f() { return "main"; }
```

最后，创建主程序文件，分别调用liba库提供的fa函数和f函数，如下所示。

```
#include <f.h>
#include <liba.h>
#include <stdio.h>

int main() {
    printf("fa: %s\n", fa());
    printf("f: %s\n", f());
    return 0;
}
```

项目顶层目录的CMake目录程序如下所示。

```
cmake_minimum_required(VERSION 3.20)

project(target_property)

add_executable(main main.c)

# 设置main的目标属性
set_target_properties(main PROPERTIES 
    # 构建要求：将include加入头文件搜索目录
    INCLUDE_DIRECTORIES ${CMAKE_CURRENT_SOURCE_DIR}/include
    # 构建要求：链接a库（即传递a库的使用要求到main中作为main的构建要求）
    LINK_LIBRARIES a
)

add_subdirectory(liba)
```

在该实例中，liba.c源文件直接引用了liba.h头文件，main.c源文件直接引用了f.h头文件，尽管它们并不在同一目录中。这说明头文件搜索目录这一目标属性的配置生效了。另外，main.c也能直接引用liba.h这个库提供的头文件，说明通过`LINK_LIBRARIES`目标属性定义的目标依赖关系也发挥了作用，成功地将静态库的使用要求传递到了主程序可执行文件目标的构建要求中。本实例涉及的属性将在7.5.6小节中详细介绍。

最后，构建该实例。

```
> cd CMakeBook/src/ch007/目标属性
> mkdir build
> cd build
> cmake ..
...
> cmake --build .
...
> ./main
fa: liba
f: main
```

若使用Visual Studio生成器构建，执行过程中调用可执行文件的路径应为./Debug/main.exe。

尽管通过本例中的目标属性来控制构建目标的构建要求和使用要求，以及建立不同构建目标之间的依赖关系是可行的，但这不是一种推荐的做法。本实例是为了方便读者理解目标属性才这样做的，而7.5.6小节中介绍的一些常用属性专门的设置命令才是推荐的方式。事实上，本小节展示的正是那些命令背后的原理。

### 7.5.4　源文件属性

源文件属性（source file property）即作用于源文件的属性，通常用于对源文件构建进行配置，也可以用于获取源文件信息等。

#### 获取源文件属性

```
get_source_file_property(<结果变量> <文件路径> 
    [DIRECTORY <目录>|TARGET_DIRECTORY <构建目标>]
    <源文件属性>)
get_property(<结果变量> SOURCE <文件路径> 
    [DIRECTORY <目录>|TARGET_DIRECTORY <构建目标>] 
    PROPERTY <源文件属性> 
    [SET|DEFINED|BRIEF_DOCS|FULL_DOCS])
```

这两个命令均用于获取位于<文件路径>的源文件的<源文件属性>，并将其值存入<结果变量>。

同一个源文件可能被用于不同目录中的不同构建目标，而它们又可能具有不同的构建要求，因此源文件属性的作用域是由源文件路径和某个特定目录共同决定的。在不指定DIRECTORY或TARGET_DIRECTORY可选参数的情况下，该命令默认在当前目录中获取源文件属性。

若通过DIRECTORY参数指定<目录>，则该命令会在指定目录中获取源文件属性。<目录>必须是已经加入项目的子目录或项目的根目录，它可以是绝对路径或相对于当前源文件目录的相对路径。若通过TARGET_DIRECTORY参数指定<构建目标>，则该命令会在<构建目标>被创建的源文件目录中获取源文件的属性。<构建目标>必须已经被创建。

当指定的源文件属性未被定义，且该源文件属性不具有INHERITED特性（继承特性）时，第一个命令的结果变量会被赋值为NOTFOUND，而第二个命令的结果变量会被赋空值。若其具有继承特性，CMake会继续向上层作用域查找同名属性的定义；若仍未找到该源文件属性的定义，则将结果变量赋为空值。上层作用域即其所在目录的作用域，再向上则与目录属性的上层作用域相同，一直递归查找到项目顶层目录的作用域及全局作用域。

#### 设置源文件属性

```
set_source_files_properties(<文件路径>...
    [DIRECTORY <目录>...]
    [TARGET_DIRECTORY <构建目标>...]
    PROPERTIES <源文件属性> <属性值> [<源文件属性> <属性值>]...)
set_property(SOURCE [<文件路径>...]
    [DIRECTORY <目录>...]
    [TARGET_DIRECTORY <构建目标>...]
    [APPEND] [APPEND_STRING] 
    PROPERTY <源文件属性> [<属性值>...])
```

这两个命令均用于将位于若干<文件路径>的源文件的<源文件属性>的值设置为<属性值>，其中第二个命令中的多个<属性值>会被视为一个列表。

DIRECTORY或TARGET_DIRECTORY参数同样用于指定源文件属性作用于哪些目录，若未指定该参数，定义的源文件属性默认作用于当前源文件目录。

#### 实例：源文件属性

下面的实例中将展示如何在不同目录中为同一个源文件定义不同的源文件属性。

首先，在主程序main.c中输出宏VERSION的值，如下所示。

```
#include <stdio.h>

int main() {
    printf("VERSION: %s\n", VERSION);
    return 0;
}
```

然后，在a和b这两个子目录中，利用同一份main.c分别创建可执行文件构建目标a和b。这两个子目录的目录程序分别如下所示。

```
add_executable(a ../main.c)
```

```
add_executable(b ../main.c)
```

最后，在项目顶层目录的目录程序中加入这两个子目录，并为main.c在不同目录中设置不同的`COMPILE_DEFINITIONS`（宏定义）源文件属性。宏VERSION的值在a目录中被定义为0.1，在b目录中则被定义为0.2。顶层目录的CMake目录程序如下所示。

```
cmake_minimum_required(VERSION 3.20)

project(source_file_property)

add_subdirectory(a)
add_subdirectory(b)

set_source_files_properties(main.c DIRECTORY a
    PROPERTIES COMPILE_DEFINITIONS VERSION="0.1")

set_source_files_properties(main.c DIRECTORY b
    PROPERTIES COMPILE_DEFINITIONS VERSION="0.2")
```

构建该实例并执行a和b这2个可执行文件：

```
> cd CMakeBook/src/ch007/源文件属性
> mkdir build
> cd build
> cmake ..
...
> cmake --build .
...
> ./a/a
VERSION: 0.1
> ./b/b
VERSION: 0.2
```

若使用Visual Studio生成器，执行过程中调用可执行文件的路径应为`./a/Debug/a.exe`和`./b/Debug/b.exe`。

### 7.5.5　缓存变量属性

缓存变量属性（cache entry property）即作用于缓存变量的属性。这类属性很少用到，一般用于获取缓存变量的描述文本、类型、枚举值、是否为高级配置等信息。

#### 获取缓存变量属性

```
get_property(<结果变量> CACHE <缓存变量>
    PROPERTY <缓存变量属性> 
    [SET|DEFINED|BRIEF_DOCS|FULL_DOCS])
```

该命令用于获取与<缓存变量>关联的名为<缓存变量属性>的属性，并将其值存入<结果变量>。若指定属性不存在，则结果变量为空值。

#### 设置缓存变量属性

```
set_property(CACHE [<缓存变量>...]
    [APPEND] [APPEND_STRING] 
    PROPERTY <缓存变量属性> [<属性值>...])
```

该命令用于设置若干<缓存变量>的<缓存变量属性>为<属性值>，其中多个<属性值>会被视为一个列表。

#### 实例：缓存变量属性

缓存变量属性共有6个，分别如下。

- ADVANCED，其值为布尔值，表示缓存变量是否为高级配置。

- HELPSTRING，其值为缓存变量的描述文本（帮助信息）。

- MODIFIED，其值为缓存变量的修改状态，仅供内部使用。请不要获取或设置该属性。

- STRINGS，其值为缓存变量的枚举值。STRING类型的缓存变量可以拥有一系列枚举值，用以作为CMake GUI下拉框中的选项。

- TYPE，其值为缓存变量类型。该属性可能的取值参见`### 3.5.2　定义变量`表。另外还有两个取值：STATIC表示CMake内部管理的缓存变量，UNINITIALIZED表示尚未指定类型的缓存变量。

- VALUE，其值为缓存变量的值。通过该属性修改缓存变量的值将不做任何检查，如类型检查，因此应当避免通过该属性设置缓存变量值。

下面的实例将演示如何使用STRINGS缓存变量属性实现CMake GUI中的下拉框选择，目录程序如下所示。

CMake GUI对中文路径支持不佳，因此该实例的文件夹名称均使用英文。

```
cmake_minimum_required(VERSION 3.20)

project(cache_entry_property)

set(VERSION "" CACHE STRING "")
set_property(CACHE VERSION PROPERTY STRINGS 1.0 2.0)
```

打开CMake GUI配置该项目，在设置VERSION缓存变量的值时可以通过下拉框选择。

### 7.5.6　构建中常用的属性

本小节介绍一些在构建过程中用于定义构建要求或使用要求的常用属性，其中有很多已经在前面实例中直接或间接地使用过了。之所以说“间接”使用，是因为有些属性并非直接通过 set_property命令进行设置，而是由专门的配套命令设置。一般来说，频繁用到的属性都会有专门的命令来简化操作，相关命令参见7.6节。

用于定义构建要求的属性，若无特别说明，往往可以同时作为目录属性、目标属性或源文件属性，分别定义某个目录中构建目标的构建要求、某个构建目标的构建要求及某个源文件的构建要求。而用于定义构建目标使用要求的属性仅可作为目标属性使用，毕竟只有构建目标才会被使用。用于定义使用要求的属性名通常以INTERFACE_开头。

#### 宏定义构建要求

`COMPILE_DEFINITIONS`属性用于定义编译（预处理）C和`C++`程序时所用到的宏。该属性值为列表字符串，每一个元素都代表一个宏定义。其元素格式为`<宏名称>或<宏名称>=<值>`。

#### 宏定义使用要求

`INTERFACE_COMPILE_DEFINITIONS`属性用于定义库目标对使用者的宏定义要求。任何构建目标，若链接到具有该属性的库目标，都会定义该属性所要求的宏。换句话说，该属性作为使用要求，会被自动传递给依赖者的构建要求。

该传递过程由CMake在配置生成阶段隐式地完成，并不会改变依赖者的`COMPILE_DEFINITIONS`属性。`COMPILE_DEFINITIONS`属性仅包含为目标自身定义的构建要求，不包含传递过来的构建要求。本小节中介绍的作为使用要求的属性都是如此隐式传递的，下文不再赘述。

#### 编译参数构建要求

`COMPILE_OPTIONS`属性用于定义作为构建要求的编译参数。该属性值是一个列表字符串。当然，构建目标最终的编译参数不仅仅由构建目标自身的`COMPILE_OPTIONS`属性决定，而是由 `CMAKE_<编程语言>_FLAGS`变量值、`COMPILE_OPTIONS`属性值及被依赖库的`INTERFACE_COMPILE_OPTIONS`属性传递过来的值共同组成。

#### 编译参数使用要求

`INTERFACE_COMPILE_OPTIONS`属性用于定义库目标对使用者编译参数的要求。任何构建目标，若链接到具有该属性的库目标，都会在构建时指定相应的编译参数。该属性作为使用要求，会被自动传递给依赖者的构建要求。

#### 编译特性构建要求

`COMPILE_FEATURES`属性用于定义作为构建要求的编译特性，常用于指定C和`C++`语言标准的版本，也可以用于细粒度控制要求的编译特性，如`C++`的constexpr特性等。CMake会对编译器支持的编译特性进行检查。该属性仅作为目标属性使用。

`CMAKE_<编程语言>_KNOWN_FEATURES`全局属性中列举了CMake中能够检测的<编程语言>的全部特性。例如， `CMAKE_CXX_KNOWN_FEATURES`中就列举了`C++`的标准版本，以及细粒度的编译特性。下面是其中一些常用的值：

- `cxx_std_98`，即`C++98`标准；

- `cxx_std_11`，即`C++11`标准；

- `cxx_std_14`，即`C++14`标准；

- `cxx_std_17`，即`C++17`标准；

- `cxx_std_20`，即`C++20`标准；

- `cxx_auto_type`，即auto自动类型推断特性；

- `cxx_constexpr`，即constexpr常量表达式特性；

- `cxx_decltype`，即decltype表达式类型推断特性。

编译特性的取值还有很多，这里就不一一列举了，有兴趣的读者可以自行查阅CMake官方文档。通常来说，能够根据语言的标准版本要求编译特性就足够了。

CMake还有一个变量`CMAKE_<编程语言>_COMPILE_FEATURES`，其值为上面这个全局属性取值的子集，表示当前编译器支持的特性列表。

#### 编译特性使用要求

`INTERFACE_COMPILE_FEATURES`属性用于定义库目标对使用者编译特性的要求。任何构建目标，若链接到具有该属性的库目标，都会要求编译器支持该属性中列举的编译特性，CMake会对编译器支持的编译特性进行检查。

#### 头文件目录构建要求

`INCLUDE_DIRECTORIES`属性用于定义头文件的搜索目录，这些目录必须是绝对路径。

#### 头文件目录使用要求

`INTERFACE_INCLUDE_DIRECTORIES`属性用于定义库目标对使用者头文件搜索目录的要求。任何构建目标，若链接到具有该属性的库目标，都会将该属性中列举的目录设置为头文件搜索目录。

#### 链接库构建要求

`LINK_LIBRARIES`属性用于定义作为构建要求的将要链接的库文件或库目标。如果设置当前构建目标的`LINK_LIBRARIES`属性值为某个库目标，CMake会将被链接的库目标及其依赖的使用要求递归传递到当前构建目标的构建要求中。同时，若链接的是一般库目标，还会使用链接器链接它构建后生成的库文件到当前构建目标。该属性仅作为目标属性使用。

#### 链接库使用要求

`INTERFACE_LINK_LIBRARIES`属性用于定义库目标对链接库的使用要求。任何构建目标，若链接到具有该属性的库目标，都会同时链接该属性指定的库文件或库目标，同时传递其中库目标的使用要求到当前构建目标的构建要求中。该属性决定了构建目标的依赖关系，同时赋予这个关系以传递性。该属性仅作为目标属性使用。

举个例子，假设库目标B的`INTERFACE_LINK_LIBRARIES`属性中包含库目标C，库目标 C的`INTERFACE_LINK_LIBRARIES`属性中包含库目标D。那么，当目标A链接到库目标B（可以通过设置目标A的LINK_LIBRARIES属性为B来实现）时，它就会同时链接库目标B、C和D。这就是在介绍链接库构建要求时介绍过的递归传递。

#### 链接目录构建要求

`LINK_DIRECTORIES`属性用于定义构建时链接器的搜索目录，通常用于设置部分第三方库二进制文件的所在目录。

#### 链接目录使用要求

`INTERFACE_LINK_DIRECTORIES`属性用于定义库目标对使用者链接器搜索目录的要求。任何构建目标，若链接到具有该属性的库目标，都会将该属性指定的目录加入链接器搜索目录。

#### 链接参数构建要求

`LINK_OPTIONS`属性用于定义链接器参数，作为动态库目标、模块库目标及可执行文件目标的构建要求。

静态库目标与上述构建目标不同，应使用`STATIC_LIBRARY_OPTIONS`属性定义作为构建要求的链接器参数。

#### 链接参数使用要求

`INTERFACE_LINK_OPTIONS`属性用于定义库目标对使用者链接器参数的要求。任何构建目标，若链接到具有该属性的库目标，都会在构建过程中被链接时指定属性要求的链接参数。

#### 源文件构建要求

SOURCES属性用于定义构建目标需要编译的源文件列表。该属性仅作为目标属性使用。

#### 源文件使用要求

`INTERFACE_SOURCES`属性用于定义当前构建目标使用者需要编译的源文件。任何构建目标，若链接到具有该属性的库目标，都会构建该属性指定的源文件。该属性仅作为目标属性使用。

### 7.5.7　自定义属性：define_property

CMake允许通过`define_property`命令自定义应用于不同作用域的属性，并为其添加文档说明。

```
define_property(<GLOBAL|DIRECTORY|TARGET|SOURCE|
    CACHED_VARIABLES|TEST|VARIABLE>
    PROPERTY <属性名称> [INHERITED]
    BRIEF_DOCS <简介文档>
    FULL_DOCS <完整文档>)
```

该命令用于在指定作用域中定义属性并为其添加文档说明。其中，第一个参数用于指定属性的作用域，它的取值如下：

- GLOBAL，即全局作用域；

- DIRECTORY，即目录作用域；

- TARGET，即构建目标作用域；

- SOURCE，即源文件作用域；

- CACHED_VARIABLES，即缓存变量作用域；

- TEST，即测试作用域；

- VARIABLE，该参数仅用于为变量添加文档说明。

INHERITED可选参数用于指定自定义的属性是否会在未被显式设置时，继承上层作用域同一属性的值。继承规则如下。

- 对于DIRECTORY目录作用域而言，上层作用域即父目录作用域。属性值的继承过程会递归进行，直至项目顶层目录。若项目顶层目录作用域中仍未定义该属性，则会继承GLOBAL全局作用域中同一属性的值。

- 对于TARGET构建目标作用域、SOURCE源文件作用域和TEST测试作用域中的属性而言，上层作用域即其所在目录的DIRECTORY作用域。若上层目录作用域仍未定义该属性，则按照目录作用域的继承规则继续从上层作用域中继承属性。

属性继承仅在获取属性值时生效。set_property命令仅会设置直接绑定到指定作用域的属性值。

## 7.6　属性相关命令

7.5.6小节中介绍了很多在构建时常用的属性。这些属性可以针对宏定义、编译选项等各方各面进行配置，而且对于每一方面，都分别会有针对构建要求和使用要求的两种属性。为了简化操作，CMake为大多数常用的属性提供了专门的配置命令，下面一起来了解一下这些命令。

### 7.6.1　设置目标链接库：target_link_libraries

7.6.2小节将会介绍CMake如何实现使用要求的传递性，这就要求了解如何将库目标链接到当前目标中来。那么，先来看看如何设置目标的链接库属性。

```
target_link_libraries(<构建目标> <库文件|库目标>...)
target_link_libraries(<构建目标> 
    <PRIVATE|INTERFACE|PUBLIC> <库文件|库目标>...
    [<PRIVATE|INTERFACE|PUBLIC> <库文件|库目标>...]...
)
```

该命令会将对<库文件|库目标>的链接过程作为<构建目标>的构建要求或使用要求，而具体是作为构建要求还是使用要求，取决于PRIVATE、INTERFACE和PUBLIC参数。

- PRIVATE参数表示将紧跟其后的<库文件|库目标>仅设置为<构建目标>的构建要求。

- INTERFACE参数表示将紧跟其后的<库文件|库目标>仅设置为<构建目标>的使用要求。

- PUBLIC参数表示将紧跟其后的<库文件|库目标>同时设置为<构建目标>的构建要求和使用要求。

- 忽略该参数（即使用第一种命令形式）时，默认采用PUBLIC参数的方式，即将指定的所有<库文件|库目标>同时设置为<构建目标>的构建要求和使用要求。

当参数<库文件|库目标>被指定为一个<库目标>时，CMake会自动建立当前目标与指定<库目标>的依赖关系。建立依赖后，当被依赖的库目标发生改变时，构建系统能够重新链接它到当前构建目标中。

CMake有一个约定：当指定的<库文件|库目标>参数中包含`“::”`（两个冒号）时，CMake认为该参数值一定是一个导入库目标或别名库目标。

当参数<库文件|库目标>被指定为一个<库文件>时，它可以是<库文件>的文件名或绝对路径。若仅指定库文件名，链接器会自行在系统库目录等默认搜索目录中查找其所在位置。

<构建目标>参数不可以是别名目标，因为别名目标是只读的构建目标。所有用于设置目标属性的命令均不能用于别名目标，后面将不再赘述。

#### 实例：链接库的递归依赖

下面这个实例中会分别创建两个动态库目标a和b，以及一个可执行文件目标c。其中，动态库b会链接动态库a，可执行文件c会链接动态库b。CMake目录程序如下所示。

```
cmake_minimum_required(VERSION 3.20)

project(a)

function(print_link_libraries tgt)
    get_target_property(res ${tgt} LINK_LIBRARIES)
    message("${tgt} LINK_LIBRARIES: ${res}")
    get_target_property(res ${tgt} INTERFACE_LINK_LIBRARIES)
    message("${tgt} INTERFACE_LINK_LIBRARIES: ${res}")
endfunction()

add_library(a SHARED a.c)

add_library(b SHARED b.c)
target_link_libraries(b ${MY_LINK_OPTION} a)
print_link_libraries(b)

add_executable(c c.c)
target_link_libraries(c PRIVATE b)
print_link_libraries(c)
```

在设置目标b如何链接目标a时，这里使用了`MY_LINK_OPTION`变量，以便展示不同参数的作用，后面配置生成CMake项目时将使用命令行对该变量赋值。另外，目录程序开头的自定义命令`print_link_libraries`可用于输出指定目标的`LINK_LIBRARIES`和`INTERFACE_LINK_LIBRARIES`目标属性。

首先看一下设置变量`MY_LINK_OPTION`值为PUBLIC时配置生成项目的情况。鉴于使用Makefile构建项目时可以输出清晰简单的日志，这里使用Linux操作系统及GNU Makefile构建系统生成器进行演示：

```
$ cd CMakeBook/src/ch007/target_link_libraries
$ mkdir build
$ cd build
$ cmake -DMY_LINK_OPTION=PUBLIC ..
...
b LINK_LIBRARIES: a
b INTERFACE_LINK_LIBRARIES: a
c LINK_LIBRARIES: b
c INTERFACE_LINK_LIBRARIES: res-NOTFOUND
...
```

通过输出的属性值可以看出，目标b的链接库构建要求和使用要求都包含目标a。也就是说，构建动态库b时，链接器需要链接动态库a；构建可执行文件c时，由于它链接了动态库b，所以也应同时链接动态库a。

下面通过make工具的VERBOSE=1参数启用详细日志输出，然后构建该项目进行验证：

```
$ cmake --build . -- VERBOSE=1 # 或 make VERBOSE=1，打开详细日志输出
...
[ 33%] Linking C shared library liba.so # 链接动态库 liba.so
...
/usr/bin/cc -fPIC -shared -Wl,-soname,liba.so -o liba.so CMakeFiles/a.dir/a.c.o 
...
[ 66%] Linking C shared library libb.so # 链接动态库 libb.so
...
/usr/bin/cc -fPIC -shared -Wl,-soname,libb.so -o libb.so CMakeFiles/b.dir/b.c.o  -Wl,-rpath,/CMake-Book/src/ch007/target_link_libraries/build liba.so
...
[100%] Linking C executable c # 链接可执行文件 c
...
/usr/bin/cc CMakeFiles/c.dir/c.c.o -o c  -Wl,-rpath,/CMake-Book/src/ch007/target_link_libraries/build libb.so liba.so 
```

在阅读上面这些命令前，先回顾一下其中出现的一些命令行参数：`-fPIC`用于启用地址无关代码；`-shared`用于指定构建动态库；`-o`用于指定输出文件名；`-Wl,-soname`用于指定动态库的SONAME属性（一个用于标识动态库ABI版本的属性）；`-Wl,-rpath`用于指定动态库的RPATH属性（参见1.4.2小节）。

现在分析一下上面的构建日志。动态库a不依赖任何其他库，因此它只需链接其目标文件a.c.o。动态库b链接到了动态库a，因此其构建命令参数中除了包含其自身目标文件b.c.o外，还包含动态库liba.so。

重点关注可执行文件c，它直接链接的库只有动态库b。不过，动态库b具有使用要求，即链接动态库a，因此可执行文件c也应当链接动态库a。链接可执行文件c的命令行参数最后同时包含了libb.so和liba.so，与预期一致。

至于变量`MY_LINK_OPTION`值分别设置为INTERFACE或PRIVATE的情况，就留给读者思考了，当然也可以亲自尝试构建一下。

本例使用动态库而非静态库来演示，是因为静态库的链接比较特殊，其构建日志并不完全符合本例中的描述。静态库相当于目标文件的集合，因此构建静态库不需要真正链接其依赖的其他库。静态库及其依赖的库都会在最终被链接到动态库或可执行文件时一起被链接。

### 7.6.2　PUBLIC、INTERFACE、PRIVATE与传递性

使用要求会被传递到使用者的构建要求中，那么它会一直被递归传递到最顶层的使用者吗？例如，A使用B，B使用C，那么C的使用要求会作为B的构建要求，但是否也会传递到A作为A的构建要求呢？换一种理解，这是不是意味着C的使用要求会被传递到B的使用要求中呢？

其实在1.6.4小节中就讨论过这个问题，使用要求不一定总被一直传递下去，读者可以返回重温一下相关内容。

那么在CMake中，应该如何区分这几种情况呢？这与7.6.1小节中介绍的PRIVATE、INTERFACE和PUBLIC三个参数有关。

- PRIVATE参数表示<构建目标>使用但不传递其后<库目标>的使用要求，即<库目标>的使用要求不作为<构建目标>的使用要求，而仅作为其构建要求。

- INTERFACE参数表示<构建目标>仅传递其后<库目标>的使用要求，即<库目标>的使用要求会作为<构建目标>的使用要求，但不会作为其构建要求。

- PUBLIC参数表示<构建目标>使用并传递其后<库目标>的使用要求，即<库目标>的使用要求会同时作为<构建目标>的使用要求和构建要求。

`target_link_libraries`命令在<库文件|库目标>参数是一个<库目标>时有两层含义：一方面定义了<构建目标>与<库目标>的依赖关系，另一方面定义了<库目标>使用要求的传递方式。

而这里介绍的三个参数同样具有两重含义：一方面决定了对指定<库文件|库目标>的链接过程是作为<构建目标>的构建要求还是使用要求，这是7.6.1小节介绍过的；另一方面决定了<库目标>的使用要求是作为<构建目标>的构建要求还是使用要求，这是本小节刚刚介绍的。

为什么两重含义可以用一个参数来确定呢？为什么不用两个参数，一个设置链接库文件或库目标是构建要求还是使用要求，另一个设置依赖库的使用要求传递与否？

这是因为这两重含义本就是相互关联的。例如，指定PRIVATE参数时，<构建目标>链接<库目标>，但不要求依赖者继续链接它，那自然就没必要传递<库目标>的使用要求了。其他参数的情况也同理。简言之，这两重含义可以同时被蕴含在这三个参数中，不必分别设置。

使用要求的传递是CMake在配置生成阶段中隐式完成，并直接体现在最终构建系统的构建规则中的。因此，构建目标的各个属性仅包含直接绑定到当前构建目标的属性值，不包含传递过来的属性值。例如，在7.6.1小节的实例中，我们看到输出的目标“c”的`LINK_LIBRARIES`属性值仅包含b，而不包含应当传递过来的a。

这三个参数还会在本节中介绍的其他命令中出现，因为这些命令都是用于设置有关构建要求和（或）使用要求的属性的。不过在其他命令中，由于不涉及链接库这种建立依赖关系和传递关系的操作，这三个参数仅具有第一重含义，即仅用于决定定义的属性是作为构建要求还是使用要求。简单总结如下：

- PRIVATE用于定义构建要求，仅影响当前构建目标的构建过程；

- INTERFACE用于定义使用要求，仅影响依赖当前目标的其他构建目标的构建过程；

- PUBLIC用于同时定义构建要求和使用要求。

下面的公式抽象地概括了这三个参数的关系。

```
PUBLIC = PRIVATE (构建要求) + INTERFACE (使用要求)
```

下面总结了一些这三个参数分别对应的具体使用场景，读者可以根据项目的实际需求选择使用的参数：

- 接口库（如头文件库）、导入库等伪构建目标不存在构建过程，所有构建相关的属性都应使用INTERFACE参数定义为使用要求；

- 当构建目标仅在内部代码实现中使用某库，而不暴露依赖库的部分在构建目标的接口（头文件）中时，应使用PRIVATE参数链接依赖的库；

- 当构建目标仅在接口（头文件）中暴露了依赖库的部分，而没有在内部代码实现中使用该依赖库时，应使用INTERFACE参数链接依赖的库；

- 当构建目标在接口（头文件）中暴露了依赖库的部分，同时也在内部代码实现中使用了该依赖库时，应使用PUBLIC参数链接依赖的库。

### 7.6.3　设置宏定义：add_compile_definitions

```
add_compile_definitions(<宏定义>...)
```

该命令仅对当前目录及其子目录中的构建目标生效，用于设置编译源文件时的<宏定义>。

<宏定义>会被加入到当前目录程序的`COMPILE_DEFINITIONS`目录属性，以及当前目录程序中定义的构建目标的`COMPILE_DEFINITIONS`目标属性。

<宏定义>不支持带参数的宏，其格式可为<宏名称>或<宏名称>=<值>。CMake在生成构建系统的构建规则时会自动转义特殊字符。

这里的自动转义只针对构建系统，使用CMake程序语言本身进行传参时仍需自行转义特殊字符。例如，`add_compile_definitions("A=\"a\"")`这段代码的引号参数中仍需转义宏定义内的引号，此时<宏定义>的参数值实际上是`A="a"`。所谓自动转义发生在CMake生成构建系统的构建规则时，例如，生成Makefile时，Makefile中变量`C_DEFINES`的值将被设置为`-DA=\"a\"`。

### 7.6.4　设置目标宏定义：target_compile_definitions

```
target_compile_definitions(<构建目标> 
    <PRIVATE|INTERFACE|PUBLIC> <宏定义>...
    [<PRIVATE|INTERFACE|PUBLIC> <宏定义>...]...
)
```

该命令用于设置编译指定<构建目标>的源文件时的<宏定义>。

调用该命令后，<构建目标>的`COMPILE_DEFINITIONS`属性中会加入PRIVATE和PUBLIC参数后的<宏定义>； <构建目标>的`INTERFACE_COMPILE_DEFINITIONS`属性中会加入PUBLIC和 INTERFACE参数后的<宏定义>。

### 7.6.5　设置编译参数：add_compile_options

```
add_compile_options(<编译参数>...)
```

该命令仅对当前目录及其子目录中的构建目标生效，用于设置编译源文件时的<编译参数>。

调用该命令后，<编译参数>会被加入到当前目录程序的`COMPILE_OPTIONS`目录属性，以及当前目录程序中定义的构建目标的`COMPILE_OPTIONS`目标属性中。

### 7.6.6　设置目标编译参数：target_compile_options

```
target_compile_options(<构建目标> 
    <PRIVATE|INTERFACE|PUBLIC> <编译参数>...
    [<PRIVATE|INTERFACE|PUBLIC> <编译参数>...]...
)
```

该命令用于设置编译指定<构建目标>的源文件时的<编译参数>。

调用该命令后，<构建目标>的`COMPILE_OPTIONS`属性中会加入PRIVATE和PUBLIC参数后的<编译参数>； <构建目标>的`INTERFACE_COMPILE_OPTIONS`属性中会加入PUBLIC和INTERFACE参数后的<编译参数>。

### 7.6.7　设置目标编译特性：target_compile_features

```
target_compile_features(<构建目标> 
    <PRIVATE|INTERFACE|PUBLIC> <编译特性>...
    [<PRIVATE|INTERFACE|PUBLIC> <编译特性>...]...
)
```

该命令用于设置编译指定<构建目标>的源文件时所需的<编译特性>，CMake会自动设置相应的编译参数。另外，<编译特性>必须是在CMake变量`CMAKE_<编程语言>_COMPILE_FEATURES`中存在的值，否则CMake会报错。关于编译特性属性值，参见7.5.6小节中对编译特性构建要求的介绍。

调用该命令后，<构建目标>的`COMPILE_FEATURES`属性中会加入PRIVATE和PUBLIC参数后的<编译特性>； <构建目标>的`INTERFACE_COMPILE_FEATURES`属性中会加入PUBLIC和INTERFACE参数后的<编译特性>。

编译特性属性只能作为目标属性，因此仅能针对构建目标设置。如果想为某个目录下的构建目标统一设置C和`C++`标准版本，可以通过设置CMake变量`CMAKE_<编程语言>_STANDARD`来实现。例如，设置`CMAKE_CXX_STANDARD`为17，可以使得当前目录及其子目录中的构建目标默认使用`C++17`标准。

### 7.6.8　设置头文件目录：include_directories

```
include_directories([AFTER|BEFORE] [SYSTEM] <目录>...)
```

该命令仅对当前目录及其子目录中的构建目标生效，用于将<目录>设置为构建目标的头文件搜索目录。<目录>可以是绝对路径或相对于当前源文件目录的相对路径。

调用该命令后，<目录>会被同时加入到当前目录程序的`INLCUDE_DIRECTORIES`目录属性，以及当前目录程序中定义的构建目标的`INLCUDE_DIRECTORIES`目标属性中。

AFTER和BEFORE参数用于控制<目录>是追加至头文件搜索目录列表的末尾还是插入到其开头，省略该参数时则默认采用AFTER行为，即追加至末尾。该默认行为可以通过设置CMake变量`CMAKE_INCLUDE_DIRECTORIES_BEFORE`的值来修改。

SYSTEM参数用于在构建时告知编译器这些目录中的头文件是系统头文件。有些编译器会针对系统头文件做一些特殊处理，如禁用编译警告等。

### 7.6.9　设置目标头文件目录：target_include_directories

```
target_include_directories(<构建目标> 
    [SYSTEM] [AFTER|BEFORE]
    <PRIVATE|INTERFACE|PUBLIC> <目录>...
    [<PRIVATE|INTERFACE|PUBLIC> <目录>...]...
)
```

该命令用于将<目录>加入到<构建目标>的头文件搜索目录列表中。<目录>可以是绝对路径或相对于当前源文件目录的相对路径。其他参数与include_directories命令中的对应参数功能一致。

调用该命令后，<构建目标>的`INCLUDE_DIRECTORIES`属性中会加入PRIVATE和PUBLIC参数后的<目录>； <构建目标>的`INTERFACE_INCLUDE_DIRECTORIES`属性中会加入PUBLIC和INTERFACE参数后的<目录>。

若指定了SYSTEM参数，那么PUBLIC或INTERFACE参数后指定的<目录>会同时被加入`INTERFACE_SYSTEM_INCLUDE_DIRECTORIES`目标属性。

### 7.6.10　设置链接库：link_libraries

```
link_libraries([库文件或库目标]...)
```

该命令仅对当前目录及其子目录中的构建目标生效，用于将<库文件或库目标>链接到构建目标中。只有在该命令调用之后创建的构建目标会被设置链接库属性。

不推荐使用该命令。建议先考虑使用能够更加清淅地管理依赖关系的`target_link_libraries`命令，参见7.6.1小节。

### 7.6.11　设置链接目录：link_directories

```
link_directories([AFTER|BEFORE] <目录>...)
```

该命令仅对当前目录及其子目录中的构建目标生效，用于将<目录>设置为构建目标的链接库搜索目录。<目录>可以是绝对路径或相对于当前源文件目录的相对路径。

调用该命令后，<目录>会被加入当前目录程序的`LINK_DIRECTORIES`目录属性，以及当前目录程序中定义的构建目标的`LINK_DIRECTORIES`目标属性。

AFTER和BEFORE参数用于控制<目录>是追加至头文件搜索目录列表的末尾还是插入到其开头，省略该参数时默认采用AFTER行为，即追加至末尾。该默认行为可以通过设置CMake变量`CMAKE_LINK_DIRECTORIES_BEFORE`的值来修改。

### 7.6.12　设置目标链接目录：target_link_directories

```
target_link_directories(<构建目标> 
    [BEFORE]
    <PRIVATE|INTERFACE|PUBLIC> <目录>...
    [<PRIVATE|INTERFACE|PUBLIC> <目录>...]...
)
```

该命令用于将<目录>设置为<构建目标>的链接库搜索目录。<目录>可以是绝对路径或相对于当前源文件目录的相对路径。

指定BEFORE参数后，<目录>会被插入到<构建目标>的链接库搜索目录列表的开头。默认情况下，它们会被追加到列表末尾。

调用该命令后，<构建目标>的`LINK_DIRECTORIES`属性中会加入PRIVATE和PUBLIC参数后的<目录>；<构建目标>的`INTERFACE_LINK_DIRECTORIES`属性中会加入PUBLIC和INTERFACE参数后的<目录>。

借助CMake提供的依赖库的查找和导入机制，能够实现比手动指定链接目录提供更好的可移植性。因此通常不需要手动指定链接目录。9.3节会介绍CMake中的查找模块。

### 7.6.13　设置链接参数：add_link_options

```
add_link_options(<链接参数>...)
```

该命令仅对当前目录及其子目录中的构建目标生效，用于设置链接构建目标时所需的<链接参数>。

调用该命令后，<链接参数>会被加入当前目录程序的`LINK_OPTIONS`目录属性及当前目录程序中定义的构建目标的`LINK_OPTIONS`目标属性。

### 7.6.14　设置目标链接参数：target_link_options

```
target_link_options(<构建目标> [BEFORE]
    <PRIVATE|INTERFACE|PUBLIC> <链接参数>...
    [<PRIVATE|INTERFACE|PUBLIC> <链接参数>...]...
)
```

该命令用于设置链接指定的<构建目标>时所需的<链接参数>。

指定BEFORE参数后，<链接参数>会被插入到<构建目标>的链接参数列表的开头。默认情况下，它们会被追加到列表末尾。

调用该命令后，<构建目标>的`LINK_OPTIONS`属性中会加入PRIVATE和PUBLIC参数后的<链接参数>， <构建目标>的`INTERFACE_LINK_OPTIONS`属性中会加入PUBLIC和INTERFACE参数后的<链接参数>。

### 7.6.15　设置目标源文件：target_sources

```
target_sources(<构建目标>
    <PRIVATE|INTERFACE|PUBLIC> <源文件>...
    [<PRIVATE|INTERFACE|PUBLIC> <源文件>...]...
)
```

该命令用于设置构建指定的<构建目标>时所需的<源文件>，<源文件>可以是绝对路径或相对于当前源文件目录的相对路径。

调用该命令后，<构建目标>的`SOURCES`属性中会加入PRIVATE和PUBLIC参数后的 <链接参数>； <构建目标>的`INTERFACE_SOURCES`属性中会加入PUBLIC和INTERFACE参数后的<链接参数>。

有了这个命令，就不必在创建目标的时候把全部源文件设置好了，甚至可以在创建目标时不提供任何源文件，而后再通过该命令为构建目标设置源文件。例如：

```
add_executable(main)
target_sources(main PRIVATE main.c)
```

### 7.6.16　无须递归传递的例程

这里将用CMake重新组织1.6.4小节中涉及的两个重要例程之一——无须递归传递的例程。

这个例程中创建了两个静态库：A和B。其中，库B在内部实现中依赖了库A，但并未暴露库A的任何接口函数或类型等。另外，该例程中还包含一个可执行文件main，它会调用库B的函数。

为了更好地展示CMake程序本身，这里直接复用1.6.4小节`#### 无须递归传递的例程`中的源文件。

下面为该例程创建CMake目录程序，如下所示。

```
cmake_minimum_required(VERSION 3.20)

project(no-transition)

add_library(A STATIC "../../ch002/无须传递/liba/a.cpp")
target_include_directories(A PUBLIC "../../ch002/无须传递/liba")

add_library(B STATIC "../../ch002/无须传递/libb/b.cpp")
target_include_directories(B PUBLIC "../../ch002/无须传递/libb")
target_link_libraries(B PRIVATE A)

add_executable(main "../../ch002/无须传递/main.cpp")
target_link_libraries(main PRIVATE B)
```

因为库B仅在内部实现中依赖库A且并未暴露其接口，因此在设置库B链接库A时应当使用PRIVATE参数。这样，库A的使用要求就仅会作为库B的构建要求，而不会进一步传递到可执行文件main中。

### 7.6.17　存在间接引用的例程

本例将用CMake重新组织1.6.4小节中涉及的另一个重要例程。该例程与前者的主要区别在于：库B不仅仅在内部实现中使用了库A，还将库A中定义的类型暴露在了库B自身的接口中，因此可执行文件main可以访问库A中定义的类型。这意味着该例程中存在间接引用，因此库B需要递归传递库A的使用要求。

这里仍然直接复用源文件，修改后的库B的头文件和源文件参见1.6.4小节`#### 存在间接引用的例程`。

针对本例创建的CMake目录程序如下所示。

```
cmake_minimum_required(VERSION 3.20)

project(transitive-usage-for-indirect-reference)

add_library(A STATIC "../../ch002/间接引用/liba/a.cpp")
target_include_directories(A PUBLIC "../../ch002/间接引用/liba")

add_library(B STATIC "../../ch002/间接引用/libb/b.cpp")
target_include_directories(B PUBLIC "../../ch002/间接引用/libb")
target_link_libraries(B PUBLIC A)

add_executable(main "../../ch002/间接引用/main.cpp")
target_link_libraries(main PRIVATE B)
```

本例与前一例程的CMake目录程序的唯一区别就是在设置库B链接库A时，使用了PUBLIC参数而非PRIVATE参数。如此一来，库A的使用要求不仅会作为库B的构建要求，还会进一步递归传递下去，同时作为可执行文件main的构建要求。

## 7.7　自定义构建规则：add_custom_command

至此，我们已经了解了CMake中定义的各类构建目标，它们的生成过程和属性的传递过程都由CMake掌控。不过有时候，开发者也需要掌控一些构建过程。例如，通过一系列命令行调用来生成构建时所需的数据文件，或者在构建某个构建目标之前或之后执行一些外部的命令行来完成环境配置或清理操作等。因此，自定义构建规则是构建过程中的常见需求。

在Makefile中，自定义构建规则非常简单，只需像下面这样定义一个新的构建目标：

```
custom_rule: <依赖目标>
    <调用命令行工具>...
```

在CMake中，可以通过调用`add_custom_command`命令实现类似的功能，本节将对它进行详细的介绍。

### 7.7.1　生成文件

`add_custom_command`命令有两种形式，先来看第一种用于生成文件的形式。

```
add_custom_command(OUTPUT <生成文件1> [<生成文件2>...]
    COMMAND <命令1> [ARGS] [<命令行参数>...]
    [COMMAND <命令2> [ARGS] [<命令行参数>...] ...]
    [BYPRODUCTS [<副产品文件>...]]
    [MAIN_DEPENDENCY <主依赖文件>]
    [DEPENDS [<依赖文件>...]]
    [IMPLICIT_DEPENDS <编程语言1> <隐式依赖文件>
        [<编程语言2> <隐式依赖文件>] ...]
    [DEPFILE <依赖清单文件>]
    [WORKING_DIRECTORY <工作目录>]
    [COMMENT <注释>]
    [VERBATIM] [APPEND] 
    [COMMAND_EXPAND_LISTS]
    [JOB_POOL <Ninja构建工具的Job Pool>]
    [USES_TERMINAL]
)
```

该命令用于将一系列<命令>定义为生成若干<生成文件>的生成规则。当<生成文件>已存在，且<依赖文件>未发生改变时，该生成规则不会重复执行。该命令相当于在Makefile中定义如下构建规则：

```
生成文件1: 依赖文件...
    命令1 命令行参数
    命令2 命令行参数
    ...
　
生成文件2: 生成文件1
...
```

需要注意的是，该命令仅用于定义如何生成文件，但生成规则对应的命令并不会立即执行，因此文件也不会在命令调用后立即生成。只有当该命令指定的<生成文件>在构建过程中被引用时，这些生成规则对应的命令才会执行，文件才会被生成。例如，可以在`add_library`或`target_sources`等命令的源文件参数中指定某个自定义构建规则中的<生成文件>，当然也可以将当前自定义构建规则中的<生成文件>设置为另一个自定义构建规则的<依赖文件>。

#### 生成文件与副产品文件

<生成文件>参数应为相对于当前二进制目录（注意，不是源文件目录）的相对路径，该参数部分支持生成器表达式（不支持目标相关表达式）。有关生成器表达式的介绍参见第8章。

BYPRODUCTS参数应为相对于当前二进制目录的相对路径，用于指定生成文件时可能产生的<副产品文件>。该参数同样部分支持生成器表达式（不支持目标相关表达式）。

<副产品文件>可以被其他自定义构建规则作为<依赖文件>，它主要服务于Ninja构建工具。Ninja要求每一个<依赖文件>都有明确对应的构建规则生成它，即必须是<生成文件>或<副产品文件>。

<副产品文件>与<生成文件>的区别在于，构建系统在决定是否需要重新生成文件时，只会判断<生成文件>的时间戳是否早于<依赖文件>的时间戳，而不会考虑<副产品文件>。在生成文件的过程中，可能存在一些未必总是需要重新生成的文件。这些文件不适合用于对比时间戳，也就应当作为<副产品文件>。

#### 命令相关设置

<命令>参数用于指定构建规则中需要执行的命令，多个命令会被顺序执行。<命令>若为可执行文件目标的名称，CMake会自动将其替换为该目标对应可执行文件的路径。

多个命令在执行时并不保证共享执行环境，例如，前一个命令设置的环境变量，后一个命令未必能获取到。

<命令>中可以包含生成器表达式。若其中包含`TARGET_FILE`或`TARGET_<文件类型>_FILE`这些与构建目标文件相关的表达式，CMake会自动建立该自定义构建规则对相关构建目标的依赖关系。

ARGS这个可选参数是历史遗留下来的，仅为了兼容性而存在，因此一般省略它即可。

<工作目录>参数用于指定执行<命令>的工作目录，支持绝对路径或相对于当前二进制目录的相对路径。其默认值为当前二进制目录。该参数中也可以包含生成器表达式。

<注释>参数的值会在自定义构建规则将被执行时输出到日志中。

#### 命令行参数自动转义

指定VERBATIM参数后，CMake会针对不同构建系统将<命令行参数>自动转义，以确保最终调用的命令行参数与该命令参数一致。推荐始终指定VERBATIM参数，以避免跨平台时出现不一致的行为。例如，下面的自定构建规则在使用Makefile生成器时，就不能正常执行：

```
add_custom_command(OUTPUT output
    COMMAND ${CMAKE_COMMAND} -E echo "1;2"
    COMMAND ...
    DEPENDS input)
```

这是因为它生成的Makefile对应构建规则为：

```
output: input
    /usr/local/bin/cmake -E echo 1;2
    ...
```

而分号在Shell脚本语言中是有特殊含义的，可作为语句分隔符。因此执行该规则时，2会被当作独立的一个命令，导致错误“2: not found”。

若指定了VERBATIM参数，则其生成的Makefile对应构建规则变更为：

```
output: input
    /usr/local/bin/cmake -E echo "1;2"
    ...
```

该参数保证了引号被自动转义到Makefile构建系统的规则中，这样就不会再出现上面的错误了。

#### 命令行参数列表展开

指定`COMMAND_EXPAND_LISTS`参数后，CMake会将<命令行参数>中的列表字符串展开为多个参数。仍然复用上面的例子，加上该参数来做个对比：

```
add_custom_command(OUTPUT output
    COMMAND ${CMAKE_COMMAND} -E echo "1;2")
    COMMAND ...
    DEPENDS input
    VERBATIM COMMAND_EXPAND_LISTS
```

该CMake程序生成的Makefile对应构建规则为：

```
output: input
    /usr/local/bin/cmake -E echo 1 2
    ...
```

其中，传入的列表字符串参数`1;2`被转换成了1和2两个独立的参数。

#### 依赖相关设置

DEPENDS用于指定生成文件时的<依赖文件>。<依赖文件>可以是文件的绝对路径或相对路径，也可以是构建目标的名称。其路径解析规则如下。

1．若其值为构建目标名称，则建立对<依赖文件>指定的构建目标的依赖关系。另外，若指定的构建目标是一个库目标或可执行文件目标，则当它被重新构建时，当前的自定义构建规则也会重新执行。

2．若其值为绝对路径，则建立对该路径对应文件的依赖关系。

3．若其值为一个已知的源文件的名称，则建立对该源文件的依赖关系。

4．若其值为相对路径，则首先认为该路径是相对于当前源文件目录的相对路径，检查文件是否存在：若存在，则建立对该文件的依赖关系；否则，认为它是相对当于前二进制目录的相对路径，并建立对该路径对应文件的依赖关系。

<依赖文件>参数中支持生成器表达式。

`IMPLICIT_DEPENDS`参数用于指定一些源文件（或头文件）依赖，并让CMake在构建时自动解析这些源文件的`#include`语句等，找到它们的间接依赖，同时作为当前构建规则的依赖文件。这样，当任何一个间接依赖的头文件发生改变时，该构建规则都会重新执行。这里的<隐式依赖文件>应使用绝对路径，<编程语言>仅支持C和CXX。另外，该参数仅支持Makefile构建系统生成器，因此不建议用于跨平台构建。

`MAIN_DEPENDENCY`用于指定生成文件时的<主依赖文件>。任何一个文件都只能被作为主依赖文件一次，因此如果设置某个源文件为主依赖文件，那么CMake自动为该源文件生成的构建规则会被覆盖。例如，构建如下所示的CMake程序时，构建工具会报找不到main.c对应目标文件的错误，这正是编译main.c到对应目标文件的规则被覆盖所导致的。主依赖文件参数主要是为了与Visual Studio构建系统的CustomBuild任务相对应，应当谨慎使用。

```
add_executable(main main.c)
add_custom_command(OUTPUT main2.c 
    COMMAND cp main.c main2.c 
    MAIN_DEPENDENCY main.c)
```

#### 自定义依赖清单

DEPFILE用于指定自定义的<依赖清单文件>，应指定相对于当前二进制目录的相对路径。自定义<依赖清单文件>通常由该构建规则的<命令>生成，这样能够细粒度控制每个<生成文件>的<依赖文件>。下面将介绍一个实例。

在本例的CMake目录程序中，依赖清单模板将被首先填充并生成到二进制目录中。然后，创建自定义构建规则以生成main2.c文件，同时创建一个可执行文件目标main2，它使用生成的main2.c作为源文件。CMake目录程序如下所示。

```
cmake_minimum_required(VERSION 3.20)

project(custom-depfile)

add_executable(main2 main2.c)

configure_file(mydep.d ${CMAKE_BINARY_DIR}/mydep.d)

add_custom_command(OUTPUT main2.c
    COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_SOURCE_DIR}/main.c main2.c
    DEPENDS main.c
    DEPFILE mydep.d
)
```

在当前项目目录中创建1.txt和2.txt这两个文件，内容任意。依赖清单文件模板如下所示，其中声明了main2.c依赖1.txt和2.txt。

```
main2.c: \
    @CMAKE_SOURCE_DIR@/1.txt \
    @CMAKE_SOURCE_DIR@/2.txt \
```

接下来使用Makefile生成器构建一次该实例。

```
$ cd CMakeBook/src/ch007/自定义构建规则/自定义依赖清单
$ mkdir build
$ cd build
$ cmake ..
...
$ make
[ 33%] Generating main2.c
[ 66%] Building C object CMakeFiles/main2.dir/main2.c.o
[100%] Linking C executable main2
[100%] Built target main2
```

可以再重复构建几次该实例，会发现Generating main2.c这个日志不会再出现，因为该自定义构建规则的依赖文件均未发生改变，它便不再执行了。此时若修改main.c、1.txt或2.txt任一文件，再次构建该实例时，该自定义构建规则会重新执行。

#### 追加规则

指定APPEND参数后，当前自定义构建规则的<命令>和<依赖文件>将被追加到<生成文件>的前一条自定义构建规则中。它要求程序中确实已经声明过<生成文件>的自定义构建规则。若指定了多个<生成文件>，则该自定义构建规将仅追加到第一个<生成文件>的对应规则中。

#### 其他构建系统相关的设置

指定`USES_TERMINAL`参数后，<命令>将尽可能在终端命令行中执行以访问标准输入输出。该参数主要用于Ninja构建系统，可以使其将该自定义构建规则的执行放置于console的Job Pool中。

`JOB_POOL`参数用于指定该自定义构建规则在执行<命令>时所在的Ninja构建工具的Job Pool。相关概念请参阅Ninja构建工具文档。

### 7.7.2　响应构建事件

`add_custom_command`命令的第二种形式用于响应构建事件。

```
add_custom_command(TARGET <构建目标>
    PRE_BUILD | PRE_LINK | POST_BUILD
    COMMAND <命令1> [ARGS] [<命令行参数>...]
    [COMMAND <命令2> [ARGS] [<命令行参数>...] ...]
    [BYPRODUCTS [<副产品文件>...]]
    [WORKING_DIRECTORY <工作目录>]
    [COMMENT <注释>]
    [VERBATIM] [COMMAND_EXPAND_LISTS]
    [USES_TERMINAL]    
)
```

该命令用于将一系列<命令>声明为当<构建目标>的指定构建事件触发时需要执行的自定义构建规则。构建事件有如下三种。

- `PRE_BUILD`事件，即构建前事件。对于Visual Studio构建系统生成器而言，它会在任何<构建目标>的构建规则执行前触发；对于其他生成器而言，它会在源文件编译之后、`PRE_LINK`事件之前触发。

- `PRE_LINK`事件，即链接前事件。它会在源文件编译之后，目标文件被链接之前（对于静态库而言则是归档打包工具被调用之前）触发。该事件要求<构建目标>不能是自定义构建目标。

- `POST_BUILD`事件，即构建后事件。它会在<构建目标>的所有构建规则执行后触发。

该子命令的其他参数与生成文件的子命令中的参数基本一致，这里不再赘述。下面通过一个简单实例展示该命令的使用。CMake目录程序如下所示。

```
cmake_minimum_required(VERSION 3.20)

project(PRE_BUILD_vs_PRE_LINK)

add_executable(main main.c)

add_custom_command(TARGET main PRE_BUILD
    COMMAND ${CMAKE_COMMAND} -E echo "run PRE_BUILD"
    VERBATIM)

add_custom_command(TARGET main PRE_LINK
    COMMAND ${CMAKE_COMMAND} -E echo "run PRE_LINK"
    VERBATIM)

add_custom_command(TARGET main POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E echo "run POST_BUILD"
    VERBATIM)
```

使用Makefile构建系统构建该实例。这里调用make命令时指定了VERBOSE=1参数，可以启用详细日志输出，以方便观察构建过程中调用的命令。其具体执行过程如下。

```
$ cd CMakeBook/src/ch007/自定义构建规则/响应构建事件
$ mkdir build
$ cd build
$ cmake ..
...
$ make VERBOSE=1
...
[50%] Building C object CMakeFiles/main.dir/main.c.o
...
[100%] Linking C executable main
/usr/local/bin/cmake -E echo "run PRE_BUILD"
run PRE_BUILD
/usr/local/bin/cmake -E echo "run PRE_LINK"
run PRE_LINK
/usr/local/bin/cmake -E cmake_link_script CMakeFiles/main.dir/link.txt --verbose=1
/usr/bin/cc CMakeFiles/main.dir/main.c.o -o main 
/usr/local/bin/cmake -E echo "run POST_BUILD"
run POST_BUILD
...
```

由执行过程可知，`PRE_BUILD`事件最先触发，然后是`PRE_LINK`事件，而且这两个事件都在链接之前触发。另外，对于除Visual Studio之外的构建系统而言，它们也都在源文件编译之后触发，该实例的构建过程便是如此。`POST_BUILD`事件则最后触发。

`PRE_BUILD`事件本意是构建前事件，但事实上只有Visual Studio构建系统能够正确支持该事件，并在源文件编译之前触发它。其他构建系统从原理上无法实现这一事件，因此对于它们而言，`PRE_BUILD`事件与`PRE_LINK`事件没有本质区别，仅仅是二者之间有个先后顺序。

## 7.8　自定义构建目标：add_custom_target

7.7节介绍的自定义构建规则可以用于生成其他构建目标依赖的文件或响应构建事件，但并不作为一个独立的构建目标。换句话说，我们并不能通过`cmake --build`命令行的`--target`参数执行指定的自定义构建规则。

本节将要介绍的自定义构建目标命令`add_custom_target`则会真正创建一个构建目标，其命令形式如下。

```
add_custom_target(<构建目标名称> [ALL]
    [<命令1> [<命令行参数>...]]
    [COMMAND <命令2> [<命令行参数>...] ...]
    [DEPENDS [<依赖文件>...]]
    [BYPRODUCTS [<副产品文件>...]]
    [WORKING_DIRECTORY <工作目录>]
    [COMMENT <注释>]
    [JOB_POOL <Ninja构建工具的Job Pool>]
    [VERBATIM] [COMMAND_EXPAND_LISTS]
    [USES_TERMINAL]    
    [SOURCES <源文件>...]
)
```

该命令用于创建一个名为<构建目标名称>的构建目标，其对应构建规则为指定的一系列<命令>。不像7.7节中的自定义构建规则仅在依赖文件改变或构建事件触发时执行，自定义构建目标总被认为是“过期”的，因此每次构建都会重新执行其对应的构建规则。

不过，自定义构建规则在默认情况下不会在构建全部时被构建。`ALL`参数用于指定是否将当前创建的目标包含在all目标（即表示“构建全部”的目标）之内。也就是说，在不指定`<ALL>`参数的默认情况下，自定义构建目标的`EXCLUDE_FROM_ALL`属性为真值。当目标的`EXCLUDE_FROM_ALL`属性为真值时，不指定构建目标参数，直接调用CMake或构建系统的构建命令是不会构建该目标的。

创建自定义构建目标命令与创建自定义构建规则的命令的参数几乎一致，仅多了<源文件>参数。该参数主要用于在IDE中显示与自定义构建目标相关联的源文件。其他参数在这里不再赘述。

下面来看一个实例，CMake目录程序如下所示。

```
cmake_minimum_required(VERSION 3.20)

project(custom-target)

add_custom_target(a
    ${CMAKE_COMMAND} -E echo "custom target: a"
    SOURCES a.c
    VERBATIM)

add_custom_target(b ALL
    ${CMAKE_COMMAND} -E echo "custom target: b"
    SOURCES b.c
    VERBATIM)
```

构建该实例的过程如下：

```
> cd CMakeBook/src/ch007/自定义构建目标
> mkdir build
> cd build
> cmake ..
...
> cmake --build .
...
custom target: b
...
```

可以看到，默认情况下，只有指定ALL参数创建的自定义构建目标b可以被默认构建。如果希望同时构建自定义构建目标a和b，必须在命令行参数中显式指定它们：

```
> cmake --build . --target a b
...
custom target: a
custom target: b
...
```

另外，该实例还通过SOURCES参数指定了与自定义构建目标相关联的源文件，这主要是为了便于在IDE中编辑使用。

使用Visual Studio生成器生成该项目，并使用Visual Studio打开生成的解决方案文件。在“解决方案资源管理器”中可以看到该例定义的两个自定义构建目标，以及与它们相关联的源文件。

## 7.9　设置依赖关系：add_dependencies

有时候我们希望构建目标A在构建目标B构建之后再构建，也就是说，构建目标A依赖构建目标B。通常来说，如果通过`target_link_libraries`为构建目标A链接了构建目标B，那么这个依赖关系会自动被建立。不过有时，这两个构建目标并没有这种明显的链接关系，如某个可执行文件可能在运行时依赖模块库（参见7.1.2小节），但模块库并不能被链接。此时就需要借助`add_dependencies`命令来显式地指定二者的依赖关系：

```
add_dependencies(<构建目标> [<依赖的构建目标>]...)
```

该命令可以使<构建目标>依赖若干指定的<依赖的构建目标>。换句话说，<依赖的构建目标>一定会在构建<构建目标>之前被构建。

注意将该命令与自定义构建规则及自定义构建目标中的DEPENDS参数区分。该命令用于设置构建目标间的依赖关系，而DEPENDS参数则主要用于设置对文件的依赖关系。

## 7.10　小结

本章先介绍了在CMake中创建一般构建目标的方法，以及添加子目录、声明项目的方法等，这些都是通过CMake组织一个完整项目结构所需的核心知识。

紧接着，本章介绍了构建过程中会接触到的一些常用属性，以及读写属性的相关命令。为了便于使用，很多属性都有专门的设置命令，而且常常是一对命令：一个用于设置目录属性及目录中构建目标的属性，如`include_directories`；一个用于设置指定构建目标的属性，如`target_include_directories`。

如果读者见过多年前的CMake 2.0时代的程序，应该会看到大量直接对目录属性进行设置的命令，毕竟那时还没有对目标属性进行设置的命令。CMake 3.0以后的版本之所以被称为现代CMake，正是因为它开创了面向目标的构建配置时代。总而言之，面向目标的属性设置能够将构建目标的配置相互解耦，是现代CMake推崇的做法。

本章最后主要介绍了自定义构建规则和自定义构建目标。借助它们，可以将外部命令行作为构建规则的一部分，甚至直接作为一个可以通过命令行独立执行的构建目标。

本章基本上全面涵盖了组织项目结构所需的各类功能特性，而且大部分概念都是本书第1章中引入的概念，读者不妨对比CMake和Makefile的不同，体会CMake对项目结构的抽象带来的便利性。至此，CMake的核心功能基本介绍完毕，我们将在后续章节中开始逐步探索CMake中的一些常用特性，第8章将介绍本章经常提到的生成器表达式。
