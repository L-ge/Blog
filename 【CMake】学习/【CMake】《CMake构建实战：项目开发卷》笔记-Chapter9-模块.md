# 第9章　模块

第3章在讲解基础语法的时候，就提到过CMake程序的三种类型之一：模块程序。CMake模块程序与脚本程序具有相同的扩展名，都是.cmake，但不同的是，脚本程序可以看作一个入口程序，或者说主程序，能够独立执行，而模块程序则是代码复用单元，通常用于提供一些辅助功能等，如同CMake语言的类库，通常会被CMake目录程序或脚本程序引用。

本章将带领大家认识一些CMake预置的模块，这些模块有的能够用于更好地调试CMake程序，有的能够用于检查系统、编译环境配置等，有的能够用于便捷地生成一些构建所需的文件……此外，还有一种更特殊的模块：查找模块。它能极大地简化引用第三方依赖库这种繁琐的操作。另外，本章末尾还会为大家介绍如何编写和使用自定义模块。

由于CMake预置的模块相当多，本章将仅介绍一些较为常用的部分模块。对更多CMake预置模块感兴趣的读者，可以在日后的开发中自行查阅CMake官方文档。另外，CMake预置的模块程序均位于CMake安装目录的`share/…/Modules`子目录中，读者也可以直接查阅其源码学习。

## 9.1　引用功能模块

第4章中介绍过include命令可以在CMake程序中引用一个功能模块，include命令的参数即模块的名称（不含扩展名.cmake），如下所示：

```
include(CMakePrintSystemInformation)
```

另外，还可以使用include命令引用自己编写的功能模块。有关include命令的详细介绍参见4.10.1小节。

## 9.2　常用的预置功能模块

本节将介绍一些较为常用的CMake预置的功能模块。

### 9.2.1　用于调试的模块

本小节中介绍的功能模块可以通过输出一些信息或检查一些配置来帮助调试CMake程序。

#### 输出系统信息：CMakePrintSystemInformation

引用这个模块后，CMake程序会直接输出一系列内部变量的值以供调试参考，如下所示。

```
cmake_minimum_required(VERSION 3.20)
project(print-system-info)
include(CMakePrintSystemInformation)
# 输出：
# CMAKE_SYSTEM is Windows-...
# CMAKE_SYSTEM file is Platform/Windows
# CMAKE_C_COMPILER is ...
# CMAKE_CXX_COMPILER is ...
# CMAKE_SHARED_LIBRARY_CREATE_C_FLAGS is -shared
# ...
```

#### 输出辅助函数：CMakePrintHelpers

该模块中提供了如下两个命令，分别用于辅助输出属性和变量的值。

```
cmake_print_properties([TARGETS <构建目标>...]
                       [SOURCES <源文件>...]
                       [DIRECTORIES <目录>...]
                       [TESTS <测试目标>...]
                       [CACHE_ENTRIES <缓存变量>...]
                       PROPERTIES <属性>...)
```

该命令可用于输出作用于若干<构建目标>、<源文件>、<目录>、<测试目标>或<缓存变量> 的指定<属性>的值，但每次调用该函数时仅可指定同一类型的作用域。例如，不能同时指定TARGETS <构建目标>和SOURCES <源文件>。

```
cmake_print_variables(<变量>...)
```

该命令可以输出指定<变量>的名称和值，相比使用message命令将多个变量的名称和值同时输出要简便很多。

下面的实例中演示了上述两个命令的用法，CMake目录程序如下所示。

```
cmake_minimum_required(VERSION 3.20)
project(print-helpers VERSION 1.0)
include(CMakePrintHelpers)

cmake_print_properties(DIRECTORIES . 
    PROPERTIES 
        BINARY_DIR 
        SOURCE_DIR
)
# 输出：
# --
#  Properties for DIRECTORY .:
#    ..BINARY_DIR = "C:/CMake-Book/src/ch009/CMakePrintHelpers/build"
#    ..SOURCE_DIR = "C:/CMake-Book/src/ch009/CMakePrintHelpers"      
#

cmake_print_variables(PROJECT_NAME PROJECT_VERSION)
# 输出：
# -- PROJECT_NAME="print-helpers" ; PROJECT_VERSION="1.0"
```

### 9.2.2　用于检查环境的模块

#### 检查是否可编译：CheckSourceCompiles

该模块中提供了如下命令：

```
check_source_compiles(<编程语言> <源程序> <结果缓存变量>
                      [FAIL_REGEX <正则表达式>...]
                      [SRC_EXT <扩展名>])
```

该命令用于检查<源程序>是否可以被指定<编程语言>的编译器编译成可执行文件，并将检查结果在配置阶段输出，同时向<结果缓存变量>中存入一个真值常量或假值常量。

<源程序>参数的值应为源程序代码，而非文件路径。若想检查某个源文件能否编译，可以先使用`file(READ)`命令将源文件的内容读取到变量中，再将变量值传入该命令。另外，<源程序>会被尝试编译为可执行文件，因此必须包含主函数入口。

该命令会将<源程序>写入临时文件，作为源文件输入到编译器。SRC_EXT参数用于指定这个临时源文件的<扩展名>。省略该参数时，该命令会采用<编程语言>对应的默认扩展名。

<结果缓存变量>是一个INTERNAL类型的缓存变量，因此检查结果会被持久化缓存。也就是说，该检查不会在CMake配置阶段重复执行。这就要求每一个检查命令对应的结果变量名称应当唯一。

`FAIL_REGEX`参数用于自定义检查失败的标准。当编译器输出的日志能够匹配到任一<正则表达式>时，认为检查失败。

另外，有一些CMake变量可以影响编译检查过程中编译器所采用的参数，如下表所示。

| 变量名 | 说明 |
| --- | --- |
| `CMAKE_<编程语言>_FLAGS` | 用于指定向对应<编程语言>的编译器传递的额外编译选项参数列表 |
| `CMAKE_REQUIRED_FLAGS` | 用于指定向所有编译器传递的额外编译选项参数列表。它传递的参数在`CMAKE_<编程语言>_FLAGS`之后 |
| `CMAKE_REQUIRED_DEFINITIONS` | 用于指定宏定义列表，如`-DA;-DB=1` |
| `CMAKE_REQUIRED_INCLUDES` | 用于指定头文件搜索目录列表。该命令不会考虑 `INCLUDE_DIRECTORIES`目录属性中的头文件搜索目录。也就是说，`include_directories`命令设置的目录对该命令无效 |
| `CMAKE_REQUIRED_LINK_OPTIONS` | 用于指定链接选项列表 |
| `CMAKE_REQUIRED_LIBRARIES` | 用于指定链接库列表。其元素可以是系统库的名称或导入库目标的名称 |
| `CMAKE_REQUIRED_QUIET` | 用于设置是否关闭检查结果的输出。若将其设置为真值常量，则该命令将不会在配置阶段的日志中输出检查结果 |

该命令是通过`try_compile`命令实现的，由于`try_compile`命令过于复杂，CMake通过该模块提供了这个简化的命令。

下面的实例演示了该命令的使用。CMake目录程序如下所示。

```
cmake_minimum_required(VERSION 3.20)
project(check-source-compiles)
include(CheckSourceCompiles)

check_source_compiles(C "int main() { return 0; }" res) # 输出：
# -- Performing Test res
# -- Performing Test res - Success
message("${res}") # 输出：1

set(CMAKE_REQUIRED_QUIET True)
check_source_compiles(C "invalid code" res2) # 输出：
# -- Performing Test res2
# -- Performing Test res2 - Failed
message("${res2}") # 输出空值
```

#### 检查是否可运行：CheckSourceRuns

该模块中提供了如下命令：

```
check_source_runs(<编程语言> <源程序> <结果缓存变量>
                 [SRC_EXT <扩展名>])
```

该命令用于检查<源程序>是否可以被指定<编程语言>的编译器编译成可执行文件并成功运行。检查结果会在配置阶段输出，同时也会存入<结果缓存变量>。

该命令的参数和用法均与`check_source_compiles`相同，此处不再赘述。影响编译选项的变量也同样适用，如上面`#### 检查是否可编译：CheckSourceCompiles`表。

#### 检查编译选项：CheckCompilerFlag

该模块中提供了如下命令：

```
check_compiler_flag(<编程语言> <选项> <结果缓存变量>)
```

该命令用于检查当前<编程语言>的编译器是否支持<选项>指定的命令行参数，并将检查结果在配置阶段的日志中输出，同时向<结果缓存变量>中存入一个真值常量或假值常量。

该命令实际上就是通过`check_source_compiles`命令实现的，因此也可以通过如上面`#### 检查是否可编译：CheckSourceCompiles`表所示的变量来改变编译选项。需要注意的是，如果这些变量值中存在非法的选项参数，将导致检查失败。

#### 检查C语言符号存在性：CheckSymbolExists

该模块中提供了如下命令：

```
check_symbol_exists(<符号> <头文件列表> <结果缓存变量>)
```

该命令用于检查程序在引用了<头文件列表>中任意的头文件后是否会存在指定的<符号>。检查结果会在配置阶段的日志中输出，并存入<结果缓存变量>。

该命令可以检查的符号类型包括宏、可以链接的变量和函数等，前面介绍的影响编译选项的变量同样适用于该命令。类型、枚举值、内建函数（intrinsic）等无法被作为符号检查，建议使用CheckSourceCompiles命令检查它们的存在性。

下面的实例用于检查`stdio.h`头文件中是否存在printf符号。CMake目录程序如下所示。

```
cmake_minimum_required(VERSION 3.20)
project(check-symbol-exists)
include(CheckSymbolExists)

check_symbol_exists(printf "stdio.h" res) # 输出：
# -- Looking for printf
# -- Looking for printf - found
message("${res}") # 输出：1
```

该命令仅适用于C语言，若想检查`C++`语言的符号存在性，如带命名空间的符号`“std::cout”`等，可以使用`CheckCXXSymbolExists`模块提供的`check_cxx_symbol_exists`命令。两个模块的命令参数和用法是一致的。

#### 检查结构体成员：CheckStructHasMember

该模块中提供了如下命令：

```
check_struct_has_member(<结构体> <成员变量> <头文件列表> <结果缓存变量>
                       [LANGUAGE <编程语言>])
```

该命令用于检查<结构体>（可以是struct或class）是否存在指定的<成员变量>。另外，为了使用这些<结构体>，程序需要引用<头文件列表>中的头文件。检查结果会在配置阶段的日志中输出，并存入<结果缓存变量>中。影响编译选项的变量也同样适用，如上面`#### 检查是否可编译：CheckSourceCompiles`表所示。

<编程语言>参数仅支持C和CXX（即`C++`）两个取值。默认情况下，其值为C。下面的实例检查`std::pair<int, int>`结构体是否存在first成员。CMake目录程序如下所示。

```
cmake_minimum_required(VERSION 3.20)
project(check-struct-member)
include(CheckStructHasMember)

check_struct_has_member("std::pair<int,int>" "first" "utility" res 
    LANGUAGE CXX) # 输出：
# -- Performing Test res
# -- Performing Test res - Success
```

#### 检查函数原型：CheckPrototypeDefinition

该模块中提供了如下命令：

```
check_prototype_definition(<函数名> <函数原型> <返回值> 
                           <头文件列表> <结果缓存变量>
                           [LANGUAGE <编程语言>])
```

该命令用于检查程序在引用<头文件列表>中的头文件后，是否存在一个名为<函数名>的函数，其原型与<函数原型> 相匹配。函数原型即用于声明函数的函数签名。检查结果会在配置阶段的日志中输出，并存入<结果缓存变量> 中。影响编译选项的变量也同样适用，如上面`#### 检查是否可编译：CheckSourceCompiles`表所示。

<返回值>可以是任意一个满足<函数原型>中返回值类型的值，该值用于生成检查所用的代码。

<编程语言>参数仅支持C和CXX（即`C++`）两个取值。默认情况下，其值为C。下面的实例中将检查`math.h`头文件中定义的sinf函数的原型。CMake目录程序如下所示。

```
cmake_minimum_required(VERSION 3.20)
project(check-prototype-def)
include(CheckPrototypeDefinition)

check_prototype_definition(sinf 
    "float sinf(float a)" "0"
    "math.h" res) # 输出：
# -- Checking prototype sinf for res
# -- Checking prototype sinf for res - True

check_prototype_definition(sinf 
    "double sinf(float a)" "0"
    "math.h" res2) # 输出：
# -- Checking prototype sinf for res2
# -- Checking prototype sinf for res2 - False
```

#### 检查选项状态栈：CMakePushCheckState

本小节介绍的与检查环境相关的模块中的命令，大都会受到如上面`#### 检查是否可编译：CheckSourceCompiles`表所示的几个变量的影响。这里将再次列举除与特定编程语言相关的`CMAKE_<编程语言>_FLAGS`以外的变量：

- `CMAKE_REQUIRED_FLAGS`；

- `CMAKE_REQUIRED_DEFINITIONS`；

- `CMAKE_REQUIRED_INCLUDES`；

- `CMAKE_REQUIRED_LINK_OPTIONS`；

- `CMAKE_REQUIRED_LIBRARIES`；

- `CMAKE_REQUIRED_QUIET`。

如果想通过上述变量为不同的检查命令设置不同的编译选项，那么就需要频繁清空、设置这些变量的值。`CMakePushCheckState`模块可以对上述变量维护一个状态栈，便于频繁更改其设置。该模块提供了如下三个命令：

```
cmake_push_check_state([RESET]) # 压栈（保存）状态
cmake_pop_check_state() # 出栈（还原）状态
cmake_reset_check_state() # 清空栈
```

这三个命令分别用于保存、还原和清空上述几个变量的当前值。

`cmake_push_check_state`命令用于接收一个可选参数RESET，指定它时，该命令会在保存状态之后，清空变量的值（相当于调用了`cmake_reset_check_state`命令）。

#### 检查是否支持链接时优化：CheckIPOSupported

该模块中提供了如下命令：

```
check_ipo_supported([RESULT <结果变量>] [OUTPUT <输出变量>]
                    [LANGUAGES <编程语言>...])
```

该命令用于检查指定的若干<编程语言>对应的编译器是否支持过程间优化（InterProcedural Optimization，IPO），也称链接时优化（Link-Time Optimization，LTO）。

检查结果会以真值或假值常量的形式存入<结果变量>中。若省略该参数，则检查失败会直接造成致命错误，终止CMake程序的运行。

OUTPUT参数指定的<输出变量>用于存放检查过程中的详细错误。

检查通过后，将构建目标的`INTERPROCEDURAL_OPTIMIZATION`属性设置为真值，即可启用链接时优化。如下所示的实例中演示了启用链接时优化的两种方式。

```
cmake_minimum_required(VERSION 3.20)
project(check-ipo-supported)
include(CheckIPOSupported)

add_executable(main main.c)

# 强制IPO：若检查失败，则报告致命错误并终止执行
check_ipo_supported() 
set_property(TARGET main PROPERTY INTERPROCEDURAL_OPTIMIZATION TRUE)

# 可选IPO：若检查失败，则输出警告且不启用IPO
check_ipo_supported(RESULT result OUTPUT output)
if(result)
    set_property(TARGET main PROPERTY INTERPROCEDURAL_OPTIMIZATION TRUE)
else()
    message(WARNING "不支持链接时优化：${output}")
endif()
```

### 9.2.3　用于生成导出头文件的模块：GenerateExportHeader

在开发供外部使用的静态库或动态库时，需要编写一个接口头文件，在其中声明库中所有的接口函数、类型等。尤其是在开发动态库时，往往还需要设置接口函数的导出属性等，同时通过设置编译选项来默认隐藏其他未导出的符号，从而隐藏实现细节。

然而，在实现过程中会遇到一些很现实的问题：各个编译器对导出接口所需的属性不同，隐藏符号的编译选项也不同；另外，有时需要更灵活地配置接口，使其能够同时用于静态库和动态库；又或者需要通过一些宏来标记某些过时的接口为弃用状态……

通常我们都会编写一个头文件，用宏来判断不同的编译器，再定义一些宏来对应不同的属性设置等，以此将不同编译器的差异隐藏起来。每次开发一个库时，恐怕都会这样“重复造轮子”。

CMake提供了GenerateExportHeader模块，它可以将这些宏按要求定义好。该模块提供了如下命令：

```
generate_export_header(<库目标>
    [EXPORT_FILE_NAME <导出头文件名称>]
    [BASE_NAME <基本名称>]
    [EXPORT_MACRO_NAME <导出接口宏名称>]
    [NO_EXPORT_MACRO_NAME <非导出接口宏名称>]
    [DEPRECATED_MACRO_NAME <已弃用接口宏名称>]
    [NO_DEPRECATED_MACRO_NAME <非弃用接口宏名称>]
    [DEFINE_NO_DEPRECATED]
    [INCLUDE_GUARD_NAME <头文件卫哨宏名称>]
    [STATIC_DEFINE <静态库宏名称>]
    [PREFIX_NAME <宏前缀名称>]
    [CUSTOM_CONTENT_FROM_VARIABLE <追加内容变量>]
)
```

该命令将在当前二进制目录中创建一个头文件，该头文件中会定义用于导出接口的常用宏。其中各个参数的含义参见下表。

| 参数 | 描述 | 默认值 |
| --- | --- | --- |
| `<导出头文件名称>` | 用于设置生成的头文件的名称 | `<库目标>_export.h` （<库目标>会转为小写） |
| `<基本名称>` | 默认命名的重要组成部分（会被转为大写的合法C标识符） | `<库目标>` |
| `<导出接口宏名称>` | 用于标记导出接口的宏的名称 | `<基本名称>_EXPORT` |
| `<非导出接口宏名称>` | 用于标记非导出接口的宏的名称 | `<基本名称>_NO_EXPORT` |
| `<已弃用接口宏名称>` | 用于标记已弃用接口的宏的名称 | `<基本名称>_DEPRECATED` |
| `<忽略弃用接口宏名称>` | 用于忽略弃用接口的宏的名称（该宏是否定义取决于下面的参数） | `<基本名称>_NO_DEPRECATED` |
| `DEFINE_NO_DEPRECATED` | 用于定义忽略弃用接口宏 | — |
| `<头文件卫哨宏名称>` | 用于指定头文件的卫哨宏的名称 | `<导出接口宏名称>_H` |
| `<静态库宏名称>` | 用于判断是否为静态库的宏的名称（仅用于判断，并无定义） | `<基本名称>_STATIC_DEFINE` |
| `<宏前缀名称>` | 用于为上面涉及的宏名称增加前缀 | 空 |
| `<追加内容变量>` | 将<追加内容变量>的内容追加到头文件最后 | 空 |


除了上表中涉及的宏定义，该命令还会在头文件中使用或定义如下三个宏。

- `<库目标>_EXPORTS`（<库目标>会转为合法C标识符），用于判断当前是在构建该库还是使用该库的接口，以定义不同的导出接口宏的值。该宏仅用于判断，因此同静态库宏一样，仅在生成的头文件中作判断使用，并无定义。用户应当在构建<库目标>时使用PRIVATE参数为目标定义该宏，使其仅作用于该库。

- `<基本名称>_DEPRECATED_EXPORT`，该宏等价于同时使用已弃用接口宏和导出接口宏。

- `<基本名称>_DEPRECATED_NO_EXPORT`，该宏等价于同时使用已弃用接口宏和非导出接口宏。

至此，该命令涉及的全部宏定义都已经介绍完毕，但也许读者仍然不知道如何使用它们。接下来，我们会通过不同应用场景的代码实例来展示这些宏的具体用法。

#### 实例：导出接口

本例将创建一个动态库，构建目标名称为print，它有一个导出接口函数`void print()`。接口头文件的定义如下所示。

```
#ifndef PRINT_H
#define PRINT_H

#include "print_export.h"

PRINT_EXPORT void print();
PRINT_NO_EXPORT void _internal();

#ifndef PRINT_NO_DEPRECATED
PRINT_DEPRECATED_EXPORT void old_print();
#endif // PRINT_NO_DEPRECATED

#endif // PRINT_H
```

其中，`print_export.h`就是接下来要通过GenerateExportHeader模块生成的头文件。为了进行演示，接口头文件中还增加了一个不会导出的内部函数`void _internal()`，以及已弃用的导出接口函数`void old_print()`。

如下所示的部分CMake目录程序展示了如何配置该动态库目标。

```
cmake_minimum_required(VERSION 3.20)
project(export-api)
include(GenerateExportHeader)

add_library(print SHARED print.c)
generate_export_header(print
    # DEFINE_NO_DEPRECATED # 取消注释以定义忽略弃用接口宏
) # 导出头文件会被生成到二进制目录中

# 将当前二进制目录追加到头文件搜索目录中
target_include_directories(print PUBLIC ${CMAKE_CURRENT_BINARY_DIR})

# 定义print_EXPORTS宏，表明正在构建该库（若不定义，则表示使用该库）
target_compile_definitions(print PRIVATE print_EXPORTS)

# 设置目标属性，默认隐藏符号和内联函数
set_target_properties(print PROPERTIES
    CXX_VISIBILITY_PRESET hidden
    VISIBILITY_INLINES_HIDDEN 1
)
# 也可以通过下面两个变量，设置上述两个目标属性的默认值
# set(CMAKE_CXX_VISIBILITY_PRESET hidden)
# set(CMAKE_VISIBILITY_INLINES_HIDDEN 1)
```

其中，在调用`generate_export_header`命令时没有提供任何可选参数，因此所有宏名称都采用默认值，如接口头文件`print.h`中用到的宏`PRINT_EXPORT`和`PRINT_NO_EXPORT`都是默认的宏名称。

最后，print.c源文件中简单定义了上面几个接口函数的具体实现，如下所示。

```
#include "print.h"
#include <stdio.h>

void print() { printf("Hello\n"); }
void _internal() {}

#ifndef PRINT_NO_DEPRECATED
void old_print() { printf("Hi\n"); }
#endif // PRINT_NO_DEPRECATED
```

#### 实例：使用接口

现在已经有了print动态库，提供了print和old_print函数接口。下面看看如何调用它。

首先，创建一个主程序main.c，其中引用了print.h头文件，并调用了它提供的两个函数，如下所示。

```
#include "print.h"

int main() {
    print();
    old_print();
    return 0;
}
```

然后，在目录程序中追加主程序对应的可执行文件目标，如下所示。

```
# 主程序
add_executable(main main.c)
target_link_libraries(main PRIVATE print)
if(MSVC)
    # 为MSVC编译器启用级别3的警告
    target_compile_options(main PRIVATE /W3)
endif()
```

就是这么简单！读者可以尝试在各个平台中使用不同的编译器构建该项目。在构建main目标时，编译器应当会报告`old_print`函数已被废弃的警告。

#### 实例：忽略弃用接口

现在解释接口头文件print.h及其对应源文件中是如何声明和定义已弃用的接口函数`old_print`的，对应代码分别如下所示。

```
#ifndef PRINT_NO_DEPRECATED
PRINT_DEPRECATED_EXPORT void old_print();
#endif // PRINT_NO_DEPRECATED
```

```
#ifndef PRINT_NO_DEPRECATED
void old_print() { printf("Hi\n"); }
#endif // PRINT_NO_DEPRECATED
```

代码中使用了预处理器`#ifndef`来判断是否定义了忽略弃用接口宏`PRINT_NO_DEPRECATED`，并仅在其未定义时包含已弃用的接口函数的声明和定义。这样一来，如果在编译时定义了忽略弃用接口宏，这些已弃用的接口就不再包含在最终的接口头文件及其源文件中。

忽略弃用接口宏无须使用`target_compile_definitions`来定义，在使用`generate_export_header`命令时指定`DEFINE_NO_DEPRECATED`参数即可定义它。

#### 实例：同时构建静态库

如果想在复用动态库代码的同时构建一个静态库`print_static`，应该怎么做呢？其实也非常简单。`generate_export_header`命令生成的导出头文件已经考虑到了这一点。只需定义静态库宏就可以了。对应于print这个目标的静态库宏名称是`PRINT_STATIC_DEFINE`，因此在目录程序中追加如下所示的内容即可。

```
# 静态库
add_library(print_static STATIC print.c)
target_include_directories(print_static PUBLIC ${CMAKE_CURRENT_BINARY_DIR})
target_compile_definitions(print_static PUBLIC PRINT_STATIC_DEFINE)
```

事实上，定义了静态库宏之后，并没有什么神奇的事情发生，它仅仅是定义导出接口宏 `PRINT_EXPORT`和非导出接口宏`PRINT_NO_EXPORT`为空值而已，毕竟静态库接口中的符号都是外部可见的。

## 9.3　查找模块

查找模块（find module）是一系列用于搜索第三方依赖软件包（包括库或可执行文件）的模块。对查找模块的引用一般不使用include命令，而是使用`find_package`命令。

### 9.3.1　查找软件包命令：find_package（模块模式）

CMake中的`find_package`命令可以用于搜索第三方依赖软件包，获得其路径、版本等信息，以便在程序中链接或使用它。

`find_package`命令有两种模式：模块模式和配置模式。其中，配置模式涉及导入依赖与分发软件包等相关内容。本书仅介绍第一种模式——模块模式。在模块模式下，该命令的形式如下。

```
find_package(<软件包名> [<版本号>] [EXACT] [QUIET] [MODULE]
             [REQUIRED] [[COMPONENTS] [<子组件>...]]
             [OPTIONAL_COMPONENTS <可选子组件>...]
             [NO_POLICY_SCOPE])
```

该命令会调用名为`“Find<软件包名>.cmake”`的查找模块来完成对软件包的搜索。它首先在`CMAKE_MODULE_PATH`变量定义的路径列表中搜索查找模块，若找不到，则从CMake安装目录中搜索符合该名称的CMake预置的查找模块。如果仍未找到对应的查找模块，该命令会切换到配置模式再进行处理。

<版本号>参数用于指定查找的软件包的版本应当满足的条件，它支持如下两种限制条件。

- 直接指定版本号，其格式为`“主版本号[.次版本号[.补丁版本号[.修订版本号]]]”`，如1.2或1.2.3.4。该条件要求查找的软件包的版本号应当兼容指定的版本号，如比指定的版本号大（软件包一般都会向后兼容，即新版本会兼容旧版本）。

- 指定版本号区间，其格式为`“最小版本号...[<]最大版本号”`，如1.2...1.4或 1.2.1...<2.0.0。该条件要求查找的软件包的版本号落在指定区间内。区间默认为闭区间，指定“<”后则是左闭右开区间。

EXACT参数表示版本号必须与指定的<版本号>一致。指定该参数后，要求<版本号>参数采用第一种直接指定版本号的条件格式。

QUIET参数用于启用静默模式，即关闭部分查找信息的输出。

MODULE参数用于指定该命令仅采用模块模式。指定该参数后，即使不存在对应软件包的查找模块，该命令也不会切换到配置模式再进行处理。

REQUIRED参数表示该软件包是构建过程所必需的，查找失败将导致致命错误，并使程序终止。

部分软件包可能由多个子组件组成，如Boost库可以作为一个软件包，其中包含了container、date_time、filesystem等组件。但我们在使用Boost库时往往只会使用其中提供的部分组件，此时可以使用COMPONENTS参数指定<子组件>。指定的<子组件>中只要有一个没有被查找到，该命令就会认为该软件包没有被成功查找。

当指定了REQUIRED参数时，可以省略COMPONENTS参数，直接在REQUIRED参数后列举所需的<子组件>。另外，REQUIRED参数也会使得该命令在任一子组件未被成功查找时报告致命错误，终止程序执行。

`OPTIONAL_COMPONENTS`参数用于指定<可选子组件>。可选子组件是否能够被成功查找不影响软件包是否被成功查找的判断。即使指定了REQUIRED参数，可选子组件未被成功查找也不会导致致命错误。

该命令执行后会定义一个变量`<软件包名>_FOUND`，当软件包被成功查找时，它会被设置为真值，否则会被设置为假值。查找模块可能还会定义其他与软件包信息相关的变量，查阅相关查找模块的文档可以了解这些变量的定义。

最后一个参数`NO_POLICY_SCOPE`的用法可参见10.3.4小节。

### 9.3.2　实例：使用FindThreads引用线程库

在Linux操作系统中开发多线程应用时，应该都遇到过下面这个报错：

```
undefined reference to `pthread_create'
```

该报错表示未定义对`pthread_create`的引用。通常的解决办法就是为编译器添加一个编译选项`-lpthread`，也就是链接pthread这个线程库。

那么，该如何在CMake目录程序中设置链接线程库呢？最直接的方法就是判断当前的操作系统，如果是Linux操作系统，就为编译器设置上面提到的编译选项。除此之外，还可使用CMake专门提供的查找模块FindThreads。FindThreads查找模块可以针对不同的平台找到对应的线程库，并为它创建好方便易用的导入目标，无须关心平台之间的差异。如下所示的CMake目录程序展示了该查找模块的用法。

```
cmake_minimum_required(VERSION 3.20)
project(find-threads)

# 调用FindThreads模块，它会创建一个导入目标Threads::Threads
find_package(Threads) 

add_executable(main main.cpp)
# 如果不链接Threads::Threads，在Linux环境中构建会出错：
# undefined reference to `pthread_create'
target_link_libraries(main PRIVATE Threads::Threads)
```

其中主程序main.cpp的代码如下所示。

```
#include <cstdio>
#include <thread>

void worker(int i) { printf("worker%d\n", i); }

int main() {
    std::thread th(worker, 0);
    th.join();
    return 0;
}
```

### 9.3.3　实例：使用FindBoost引用Boost库

除了导入线程库，CMake还提供了大量的查找模块用于导入其他第三方依赖。本小节将以Boost库为例，展示如何使用预置的FindBoost查找模块导入Boost库。

还记得在介绍`find_package`命令时提到的<子组件>参数吗？对于Boost而言，子组件可以是Boost库集合中提供的每一个库。这些库不带前后缀的名称可以作为<子组件>参数的取值，如filesystem、date_time等。

#### 查找条件变量

通过设置一些查找条件变量的值，可以指定Boost库的位置、版本等条件，以提示或要求FindBoost模块。下表列举了部分常用的查找条件变量。

| 变量名 | 描述 |
| --- | --- |
| `BOOST_ROOT`或`BOOSTROOT` | Boost库的安装根目录 |
| `BOOST_INCLUDEDIR` | Boost库的头文件目录 |
| `BOOST_LIBRARYDIR` | Boost库的库文件目录 |
| `Boost_NO_SYSTEM_PATHS` | 只在查找条件变量设置的路径中搜索Boost库。若为ON则表示仅搜索变量设置的路径。默认为OFF，即同时搜索系统默认路径 |
| `Boost_ADDITIONAL_VERSIONS` | Boost库的版本号。由于Boost的安装目录路径中可能出现版本号，FindBoost查找模块只有能够识别版本号才能正确解析Boost库的路径。一般情况下，Boost版本号能够被自动识别，若部分版本号无法被识别，可用该变量进行设置 |
| `Boost_USE_<DEBUG\|RELEASE>_LIBS` | Boost库的构建模式，用于指定查找Debug还是Release模式构建的Boost库 |
| `Boost_USE_STATIC_LIBS` | 查找Boost静态库 |
| `Boost_USE_MULTITHREAD` | 使用Boost多线程库（库文件名带mt） |
| `Boost_USE_STATIC_RUNTIME` | 使用链接到静态`C++`运行时的Boost库（库文件名带s） |
| `Boost_USE_DEBUG_RUNTIME` | 使用链接到Debug模式`C++`运行时的Boost库（库文件名带g） |
| `Boost_DEBUG` | 启用FindBoost查找模块的调试模式。启用后会输出一些调试信息，便于定位查找失败的问题 |

#### 查找结果变量

FindBoost查找模块执行后会将查找结果存入查找结果变量中。下表列举了一些常用的查找结果变量。

| 变量名 | 描述 |
| --- | --- |
| `Boost_FOUND` | 是否成功查找到指定的库（需同时查找到全部必要的子组件） |
| `Boost_INCLUDE_DIRS` | 查找到的Boost的头文件目录列表 |
| `Boost_LIBRARY_DIRS` | 查找到的Boost库文件目录列表 |
| `Boost_LIBRARIES` | 查找到的Boost各个组件的库文件的列表 |
| `Boost_<子组件>_FOUND` | 是否成功查找到指定的<子组件> |
| `Boost_<子组件>_LIBRARY` | 指定<子组件>的库文件 |
| `Boost_VERSION` | 查找到的Boost库的版本号 |


借助这些查找结果变量，可以将Boost库链接到程序中，如下所示。

```
find_package(Boost REQUIRED COMPONENTS filesystem)
include_directories(${Boost_INCLUDE_DIRS})
add_executable(main main.cpp)
target_link_libraries(main ${Boost_LIBRARIES})
```

#### 导入目标

除了设置结果变量，FindBoost查找模块还会创建一些导入目标，以方便链接Boost库，如下表所示。

| 导入目标名称 | 描述 |
| --- | --- |
| `Boost::boost`或`Boost::headers` | 代表Boost全部头文件库的导入库目标。Boost的头文件库一般均位于同一个头文件目录，具有相同的使用要求，因此被统一到一个导入目标 |
| `Boost::<子组件>` | 代表指定Boost<子组件>的导入库目标 |
| `Boost::diagnostic_definitions` | 该目标设置了`-DBOOST_LIB_DIAGNOSTIC`宏定义使用要求，用于启用Boost自动链接相关的调试信息 |
| `Boost::disable_autolinking` | 该目标设置了`-DBOOST_ALL_NO_LIB`宏定义使用要求，用于禁用Boost库针对MSVC编译器的自动链接特性 |
| `Boost::dynamic_linking` | 该目标设置了`-DBOOST_ALL_DYN_LINK`宏定义使用要求，用于启用Boost库针对MSVC编译器的动态链接特性 |

FindBoost查找模块会自动处理好子组件间的依赖关系，因此在使用导入目标链接Boost指定的组件库时，只需在`target_link_libraries`中指定想要链接到的组件库对应的导入目标，而不必关心它依赖哪些库。例如，当链接`Boost::algorithm`目标时，`Boost::range`、`Boost::assert`等目标也会作为其依赖被链接。

如下所示为一个借助导入目标链接Boost库的实例。

```
find_package(Boost REQUIRED COMPONENTS date_time filesystem)
add_executable(main main.cpp)
target_link_libraries(foo Boost::date_time Boost::filesystem)
```

可见，借助导入目标链接Boost库更加简洁清晰。那么，还有必要使用结果变量吗？

面向目标的CMake毕竟是现代CMake。在CMake 3.5及后续版本中，FindBoost查找模块才开始创建这些导入目标。因此在过去，使用结果变量是唯一的选择。现在，使用导入目标当然是更推荐的做法。然而，相比使用结果变量，导入目标仍然存在限制：如果多次使用不同的参数选项和条件变量调用`find_package(Boost)`命令，FindBoost模块可能会查找到不同位置的Boost库，并将它们的信息设置到结果变量中，但不会覆盖首次调用时创建的导入目标。因此导入目标仅代表了首次查找Boost库的结果，若想多次获取不同Boost库的路径等信息，就只能使用结果变量。

#### 实例：使用Boost Regex库

第1章曾使用Makefile构建了一个使用Boost Regex静态库的程序，其主程序参见`### 1.5.4　链接Boost C++库 #### 使用Boost Regex库提取URL`代码。本例将复用该主程序，并借助FindBoost查找模块，改用CMake来完成其构建。CMake目录程序如下所示。

```
cmake_minimum_required(VERSION 3.20)

project(find-boost)

if(WIN32)
    set(BOOST_ROOT "C:/boost_prebuilt") # 使用预编译的Boost库
    # set(BOOST_ROOT "C:/boost/stage") # 使用自己构建的Boost库
else()
    # 默认在系统目录查找Boost库，无须设置条件变量
    # set(BOOST_ROOT "$ENV{HOME}/boost") # 取消注释以使用自己构建的Boost库
endif()

find_package(Boost REQUIRED COMPONENTS regex)

add_executable(main "../../ch001/链接Boost/main.cpp")
target_link_libraries(main Boost::regex)
```

## 9.4　编写自定义查找模块

尽管CMake内置了相当多软件包的查找模块，但难免还会有特殊的需求。本节将介绍如何自己编写一个查找模块。

在查找模块的实现中，往往需要查找软件包所需的可执行文件、库文件、头文件目录等路径。CMake提供了一系列命令用以辅助完成这些工作。下面先介绍这些命令。

### 9.4.1　查找文件：find_file

该命令有两种形式，一种是简单形式，另一种是完整形式：

```
find_file(<结果缓存变量> <文件名> [<候选路径...>]) # 简单形式
　
find_file(<结果缓存变量> # 完整形式
    <文件名> | NAMES <候选文件名>...
    [HINTS|PATHS [<候选路径>|ENV <候选路径环境变量>]...]
    [PATH_SUFFIXES <子目录>...]
    [DOC <结果缓存变量的描述>]
    [REQUIRED]
    [<搜索路径选项>...]
    [CMAKE_FIND_ROOT_PATH_BOTH |
    ONLY_CMAKE_FIND_ROOT_PATH |
    NO_CMAKE_FIND_ROOT_PATH]
)
```

该命令用于查找<文件名>对应文件的绝对路径，并将其存入<结果缓存变量>。若该命令执行前<结果缓存变量>不为空值，则该命令不会执行，以免重复查找。若想强制重新查找文件，可以编辑CMakeCache.txt，删除对应缓存变量的值。当文件查找失败时，<结果缓存变量>会被赋值为<结果缓存变量>-NOTFOUND。

NAMES参数可以指定若干<候选文件名>，该命令会依次查找，并将第一个查找到的文件名的绝对路径作为查找结果。

`find_file`命令在查找指定文件时，会从一系列默认搜索路径中查找，HINTS或PATHS参数可用于补充搜索的<候选路径>。该参数同时支持通过ENV指定<候选路径环境变量>，候选路径会从指定的环境变量中读取。

`PATH_SUFFIXES`参数用于指定若干用于追加到<候选路径>的<子目录>。`find_file`命令在候选路径中查找文件时，会同时在这些候选路径对应目录的子目录中查找文件（不追加子目录的候选路径本身也会被搜索）。

DOC参数用于设置<结果缓存变量的描述>。在介绍使用set命令定义缓存变量时，就提到过用于设置<变量描述>的参数，其值可以在CMake GUI中看到。DOC参数就是用于设置<结果缓存变量>描述文本的。

REQUIRED参数用于指定该文件是必需的。当文件未能成功查找时，CMake会报告错误并终止执行。

#### 搜索顺序与搜索路径选项

<搜索路径选项>有很多选项参数可以指定，分别用于禁用一些默认的搜索路径。下表将按照`find_file`搜索各类路径的顺序，依次介绍各类路径及禁用对应搜索路径类别所需的选项参数。

| 路径类型 | 路径样例 | 禁用选项参数 |
| --- | --- | --- |
| 软件包根目录 | `<软件包根目录>/include[/<架构>]` | `NO_PACKAGE_ROOT_PATH` |
| CMake变量路径 | `${CMAKE_PREFIX_PATH}/include[/<架构>]` `${CMAKE_INCLUDE_PATH}` `${CMAKE_FRAMEWORK_PATH}` | `NO_CMAKE_PATH` |
| CMake环境变量路径 | `$ENV{CMAKE_PREFIX_PATH}/include[/<架构>]` `$ENV{CMAKE_INCLUDE_PATH}` `$ENV{CMAKE_FRAMEWORK_PATH}` | `NO_CMAKE_ENVIRONMENT_PATH` |
| HINTS候选路径 | <候选路径> | — |
| 系统环境变量路径 | `$ENV{INCLUDE} $ENV{PATH}` | `NO_SYSTEM_ENVIRONMENT_PATH` |
| 平台相关路径 | `${CMAKE_SYSTEM_PREFIX_PATH}` `${CMAKE_SYSTEM_INCLUDE_PATH}` `${CMAKE_SYSTEM_FRAMEWORK_PATH}` | `NO_CMAKE_SYSTEM_PATH` |
| PATHS候选路径 | <候选路径> | — |


下面对上表进行一些补充说明。

- 软件包根目录（package root path）即查找模块查找到的软件包的根目录，也就是 `<软件包>_ROOT`变量或环境变量的值。因此，该路径仅对通过`find_package` 命令调用的查找模块中的`find_file`命令有效。如果查找模块被嵌套调用，则每一层的软件包根目录都会被维护于栈结构中。这些目录对应的`include[/架构]`子目录都会被作为搜索路径，搜索顺序则为当前软件包根目录、父级软件包根目录，以此类推。

- 平台相关路径涉及的CMake变量值由CMake根据平台预定义，尽管也可以被修改，但并不建议。通过修改CMake变量路径或CMake环境变量路径涉及的变量值来自定义搜索路径是推荐的做法。其中平台相关路径`${CMAKE_SYSTEM_PREFIX_PATH}`的值，在Windows操作系统中包括`C:\Program Files`、`C:\Program Files (x86)`等目录；在类UNIX操作系统中，包括`/usr/local`等目录。

- 在Windows操作系统中，系统环境变量路径`$ENV{PATH}`中的值一般会被追加include子目录后作为搜索路径。若环境变量存在以bin或sbin结尾的值，则取其上层目录并追加`include[/<架构>]`作为搜索路径。

- PATHS参数指定的<候选路径>优先级最低，往往用于一些写死的目录。`find_file`简单形式中的<候选路径>参数等价于这里的路径。

- 若`CMAKE_LIBRARY_ARCHITECTURE`变量被定义，那么路径中涉及`[/<架构>]`的可选部分将会存在，且`<架构>`的值即该变量的值。

- 除了上表中的禁用选项参数，还有一个`NO_DEFAULT_PATH`。指定它将禁用全部默认搜索路径，仅根据参数提供的<候选路径>和<子目录>来搜索。

- `find_file`一般用于在头文件目录中搜索头文件，这也是为什么大多数搜索路径都会包含include子目录。

#### 重定向根目录

CMake变量`CMAKE_FIND_ROOT_PATH`可以用于重定向前面提到的所有搜索路径所对应的根目录，且可以设置为若干目录。

例如，假设`CMAKE_PREFIX_PATH`变量的值为`/a`，那么默认情况下`find_file`命令会搜索`/a/include`；但如果设置`CMAKE_FIND_ROOT_PATH`变量的值为`/b;/c`，那么它就会在`/b/a/include` 和`/c/a/include`这两个目录中搜索了。如果搜索不到，它才会再次尝试搜索未被重定向的路径。

`find_file`命令的最后一个参数为三选一的选项，可以用于改变这一搜索逻辑，如下所示：

- `CMAKE_FIND_ROOT_PATH_BOTH`，即按照默认重定向根目录的搜索顺序搜索；

- `NO_CMAKE_FIND_ROOT_PATH`，即禁用重定向根目录，忽略`CMAKE_FIND_ROOT_PATH`变量；

- `ONLY_CMAKE_FIND_ROOT_PATH`，即仅搜索重定向根目录后的路径。当搜索不到时，不再考虑重定向前的路径。

重定向根目录这个特性往往用于交叉编译，因为交叉编译工具链中库的目录结构与系统库的目录结构一般是相同的，将根目录重定向到编译工具链的安装根目录，即可查找位于编译工具链子目录中的文件。

#### 实例

由于`find_file`在CMake脚本程序中也可以使用，为方便起见，这里直接使用脚本程序编写实例。CMake脚本程序如下所示。

```
if(WIN32)
    find_file(notepad_path notepad.exe)
    message("${notepad_path}") # 输出：C:/Windows/System32/notepad.exe
endif()

# 使用tree .查看当前目录的树形结构
# ├── a
# │   └── b
# │       └── 1.txt     
# ├── b
# │   └── a
# │       └── 1.txt     
# └── find_file.cmake   

find_file(res1 1.txt HINTS a b PATH_SUFFIXES b)
message("${res1}") # 输出：.../CMake-Book/src/ch009/find_file/a/b/1.txt

find_file(res2 1.txt HINTS a b PATH_SUFFIXES a)
message("${res2}") # 输出：.../CMake-Book/src/ch009/find_file/b/a/1.txt
```

### 9.4.2　查找库文件：find_library

与`find_file`类似，`find_library`命令也有两种形式，参数几乎完全一样，下面将仅择其特殊之处介绍：

```
find_library(<结果缓存变量> <库名称> [<候选路径...>]) # 简单形式
　
find_library(<结果缓存变量> # 完整形式
    <库名称> | NAMES <候选库名称>... [NAMES_PER_DIR]
    [HINTS|PATHS [<候选路径>|ENV <候选路径环境变量>]...]
    [PATH_SUFFIXES <子目录>...]
    [DOC <结果缓存变量的描述>]
    [REQUIRED]
    [<搜索路径选项>...]
    [CMAKE_FIND_ROOT_PATH_BOTH |
    ONLY_CMAKE_FIND_ROOT_PATH |
    NO_CMAKE_FIND_ROOT_PATH]
)
```

该命令用于查找<库名称>对应库文件的绝对路径，并将其存入<结果缓存变量>中。

库名称参数可以仅指定库的原始名称，无须包含前缀后缀，如a。该命令在查找库文件时会根据当前平台的惯例补齐文件名，例如，在Linux中可能会搜索liba.a，而在Windows中可能会搜索a.lib。

`find_library`命令一般用于查找库文件，因此默认搜索的路径与`find_file`会有所不同，但这些路径的类别和顺序是相似的，如下表所示。

| 路径类型 | 路径样例 | 禁用选项参数 |
| --- | --- | --- |
| 软件包根目录 | `<软件包根目录>/lib[/<架构>]` | `NO_PACKAGE_ROOT_PATH` |
| CMake变量路径 | `${CMAKE_PREFIX_PATH}/lib[/<架构>]` `${CMAKE_LIBRARY_PATH}` `${CMAKE_FRAMEWORK_PATH}` | `NO_CMAKE_PATH` |
| CMake环境变量路径 | `$ENV{CMAKE_PREFIX_PATH}/lib[/<架构>]` `$ENV{CMAKE_LIBRARY_PATH}` `$ENV{CMAKE_FRAMEWORK_PATH}` | `NO_CMAKE_ENVIRONMENT_PATH` |
| HINTS候选路径 | <候选路径> | — |
| 系统环境变量路径 | `$ENV{LIB}` `$ENV{PATH}` | `NO_SYSTEM_ENVIRONMENT_PATH` |
| 平台相关路径 | `${CMAKE_SYSTEM_PREFIX_PATH}` `${CMAKE_SYSTEM_LIBRARY_PATH}` `${CMAKE_SYSTEM_FRAMEWORK_PATH}` | `NO_CMAKE_SYSTEM_PATH` |
| PATHS候选路径 | <候选路径> | — |

在Windows操作系统中，系统环境变量路径`$ENV{PATH}`中的值一般会被追加lib子目录后作为搜索路径。若环境变量存在以bin或sbin结尾的值，则取其上层目录并追加`lib[/<架构>]`作为搜索路径。

另外，该命令还有一个`NAMES_PER_DIR`参数，用于改变查找库文件的策略。默认情况下，如果通过NAMES参数指定了多个候选的<库名称>，该命令会依次搜索候选的<库名称>，并且对每一个<库名称>都会遍历全部搜索路径。但如果指定了`NAMES_PER_DIR`参数，那么就会反过来依次在每一个搜索路径中尝试查找全部的候选<库名称>。换句话说，默认是库优先搜索，指定该参数后则是目录优先搜索。如下所示的CMake目录程序展示了该参数的作用。

```
cmake_minimum_required(VERSION 3.20)
project(find-library)

# 使用tree .查看当前目录的树形结构
# ├── CMakeLists.txt
# ├── dir1
# │   ├── b.lib
# │   └── libb.a
# └── dir2
#     ├── a.lib
#     └── liba.a
# 注意，在不同平台中的输出结果不同
find_library(res1 NAMES a b HINTS dir1 dir2)
message("${res1}") # 输出.../find_library/dir2/ liba.a 或 a.lib

find_library(res2 NAMES a b NAMES_PER_DIR HINTS dir1 dir2)
message("${res2}") # 输出.../find_library/dir1/ libb.a 或 b.lib
```

### 9.4.3　查找目录：find_path

`find_path`命令同样有两种形式：

```
find_path(<结果缓存变量> <库名称> [<候选路径...>]) # 简单形式
　
find_path(<结果缓存变量> # 完整形式
    <文件名> | NAMES <文件名>...
    [HINTS|PATHS [<候选路径>|ENV <候选路径环境变量>]...]
    [PATH_SUFFIXES <子目录>...]
    [DOC <结果缓存变量的描述>]
    [REQUIRED]
    [<搜索路径选项>...]
    [CMAKE_FIND_ROOT_PATH_BOTH |
    ONLY_CMAKE_FIND_ROOT_PATH |
    NO_CMAKE_FIND_ROOT_PATH]
)
```

该命令用于查找指定<文件名>所在目录的绝对路径，并将其存入<结果缓存变量>。尽管该命令的结果是一个目录路径，但也需要先搜索到指定的文件，因此该命令的参数及搜索路径的顺序等均与`find_file`命令对应一致，这里不再赘述。

### 9.4.4　查找可执行文件：find_program

find_program命令同样有两种形式：

```
find_program(<结果缓存变量> <库名称> [<候选路径...>]) # 简单形式
　
find_program(<结果缓存变量> # 完整形式
    <文件名> | NAMES <文件名>... [NAMES_PER_DIR]
    [HINTS|PATHS [<候选路径>|ENV <候选路径环境变量>]...]
    [PATH_SUFFIXES <子目录>...]
    [DOC <结果缓存变量的描述>]
    [REQUIRED]
    [<搜索路径选项>...]
    [CMAKE_FIND_ROOT_PATH_BOTH |
    ONLY_CMAKE_FIND_ROOT_PATH |
    NO_CMAKE_FIND_ROOT_PATH]
)
```

该命令用于查找<文件名>对应的可执行文件的绝对路径，并将其存入<结果缓存变量>中。该命令的参数与`find_library`一致，这里不再赘述。下表中展示了该命令的搜索路径类型。

| 路径类型 | 路径样例 | 禁用选项参数 |
| --- | --- | --- |
| 软件包根目录 | `<软件包根目录>/lib[/<架构>]` | `NO_PACKAGE_ROOT_PATH` |
| CMake变量路径 | `${CMAKE_PREFIX_PATH}/lib[/<架构>]` `${CMAKE_PROGRAM_PATH}` `${CMAKE_APPBUNDLE_PATH}` | `NO_CMAKE_PATH` |
| CMake环境变量路径 | `$ENV{CMAKE_PREFIX_PATH}/lib[/<架构>]` `$ENV{CMAKE_PROGRAM_PATH}` `$ENV{CMAKE_APPBUNDLE_PATH}` | `NO_CMAKE_ENVIRONMENT_PATH` |
| HINTS候选路径 | <候选路径> | — |
| 系统环境变量路径 | `$ENV{PATH}` | `NO_SYSTEM_ENVIRONMENT_PATH` |
| 平台相关路径 | `${CMAKE_SYSTEM_PREFIX_PATH}` `${CMAKE_SYSTEM_PROGRAM_PATH}` `${CMAKE_SYSTEM_APPBUNDLE_PATH}` | `NO_CMAKE_SYSTEM_PATH` |
| PATHS候选路径 | <候选路径> | — |

下面的实例展示了该命令的用法。CMake目录程序如下所示。

```
find_program(res NAMES cmake)
message("${res}") # 输出的值应与${CMAKE_COMMAND}一致
```

### 9.4.5　与查找参数相关的变量

`find_package`命令会调用名为`Find<软件包名>.cmake`的查找模块来完成对软件包的搜索，那么我们自己编写的`Find<软件包名>.cmake`查找模块如何获取调用`find_package`命令时提供的参数呢？

事实上，`find_package`命令被调用时，会根据调用者提供的参数定义一系列变量来描述查找的要求，如下表所示。它们仅作用于查找模块的作用域。

| 变量名称 | 描述 |
| --- | --- |
| `CMAKE_FIND_PACKAGE_NAME` | 当前搜索中的<软件包名> |
| `<软件包名>_FIND_REQUIRED` | 是否指定了REQUIRED参数 |
| `<软件包名>_FIND_QUIETLY` | 是否指定了QUIET参数 |
| `<软件包名>_FIND_VERSION_EXACT` | 是否指定了EXACT参数 |
| `<软件包名>_FIND_COMPONENTS` | 要搜索的软件包子组件的列表，包括必要组件和可选组件 |
| `<软件包名>_FIND_REQUIRED_<子组件>` | <子组件>是否是必要组件 |
| `<软件包名>_FIND_VERSION_COMPLETE` | 指定的<版本号>参数原始字符串 |
| `<软件包名>_FIND_VERSION` | 要搜索的软件包版本号的完整字符串 |
| `<软件包名>_FIND_VERSION_MAJOR` `<软件包名>_FIND_VERSION_MINOR` `<软件包名>_FIND_VERSION_PATCH` `<软件包名>_FIND_VERSION_TWEAK` | 要搜索的软件包版本号的指定部分。若对应部分未明确指定，则变量值为0 |
| `<软件包名>_FIND_VERSION_COUNT` | 用于描述软件包版本号指定了几部分 |

若要搜索的版本号是以版本号区间形式指定的，则上表中后三个与版本号相关的变量将根据指定区间的最低版本号来确定值。这样，即便调用的查找模块仅支持按固定版本号搜索软件包的逻辑，也可以兼容区间形式的版本号搜索，即搜索区间下限，也就是最低版本。与此同时，使用区间形式版本号时，下表中列举的变量也会被额外定义。

| 变量名称 | 描述 |
| --- | --- |
| `<软件包名>_FIND_VERSION_RANGE` | 版本号区间的完整字符串 |
| `<软件包名>_FIND_VERSION_RANGE_MIN` | 版本号区间的下限（最低版本）是否被包含在区间内。其值必然为INCLUDE，即包含 |
| `<软件包名>_FIND_VERSION_RANGE_MAX` | 版本号区间的上限（最高版本）是否被包含在区间内。其值可为INCLUDE或EXCLUDE，分别表示包含和不包含 |
| `<软件包名>_FIND_VERSION_MIN` | 版本号区间的最低版本的完整字符串 |
| `<软件包名>_FIND_VERSION_MAX` | 版本号区间的最低版本的完整字符串 |
| `<软件包名>_FIND_VERSION_MIN_MAJOR；<软件包名>_FIND_ VERSION_MIN_MINOR；<软件包名>_FIND_VERSION_MIN_ PATCH ；<软件包名>_FIND_VERSION_MIN_TWEAK` | 版本号区间的最低版本的指定部分。若对应部分未明确指定，则变量值为0 |
| `<软件包名>_FIND_VERSION_MIN_COUNT` | 描述版本号区间的最低版本指定了几个部分 |
| `<软件包名>_FIND_VERSION_MAX_MAJOR` `<软件包名>_FIND_VERSION_MAX_MINOR` `<软件包名>_FIND_VERSION_MAX_PATCH` `<软件包名>_FIND_VERSION_MAX_TWEAK` | 版本号区间的最高版本的指定部分。若对应部分未明确指定，则变量值为0 |
| `<软件包名>_FIND_VERSION_MAX_COUNT` | 描述版本号区间的最高版本指定了几个部分 |

下面这个实例展示了在查找模块中部分上述变量的取值。该实例不涉及构建过程，采用CMake脚本程序编写，主脚本程序和自定义查找模块的程序分别如下所示。

```
set(CMAKE_MODULE_PATH "${CMAKE_CURRENT_LIST_DIR};${CMAKE_MODULE_PATH}")
find_package(Custom 2.1...<2.3 REQUIRED COMPONENTS a)
```

```
message("${CMAKE_FIND_PACKAGE_NAME}") # 输出：Custom
message("${Custom_FIND_REQUIRED}") # 输出：1
message("${Custom_FIND_VERSION_COMPLETE}") # 输出：2.1...2.3
message("${Custom_FIND_VERSION}") # 输出：2.1
message("${Custom_FIND_VERSION_COUNT}") # 输出：2
message("${Custom_FIND_VERSION_RANGE}") # 输出：2.1...2.3
message("${Custom_FIND_VERSION_RANGE_MAX}") # 输出：EXCLUDE
message("${Custom_FIND_VERSION_MIN}") # 输出：2.1
message("${Custom_FIND_VERSION_MAX}") # 输出：2.3
message("${Custom_FIND_COMPONENTS}") # 输出：a
```

### 9.4.6　查找条件变量

前面介绍FindBoost查找模块时提到过查找条件变量。它们可以用来提示查找模块在哪里查找、查找满足何种条件的库。之所以说是“提示”，是因为即使不设置这些变量，查找模块往往也能够自动搜索软件包的相关路径信息，并设置好这些变量的值。

查找条件变量一般是可以修改编辑的缓存变量，这样，当用户未定义它们时，查找模块就可以借助前面介绍的`find_file`、`find_library`等命令查找相关文件或目录的路径，并设置这些缓存变量；当用户定义了这些缓存变量时，它们对应的查找命令也可以自动跳过。查找模块支持的查找条件变量应当由查找模块的制作者在程序注释中详细描述，一般来说，查找模块都会提供如下表所示的查找条件变量。

| 变量名称 | 描述 |
| --- | --- |
| `<软件包名>_LIBRARY` | 用于设置库文件的路径。该变量通常作为`find_library`的结果变量（该变量仅适用于只包含一个库文件的软件包） |
| `<软件包名>_<库名>_LIBRARY` | 用于设置软件包中<库名>对应的库文件的路径。该变量也通常作为`find_library`的结果变量 |
| `<软件包名>_INCLUDE_DIR` | 用于设置头文件搜索目录。该变量通常作为`find_path`的结果变量 |
| `<软件包名>_<库名>_INCLUDE_DIR` | 用于设置软件包中<库名>对应的库的头文件搜索目录。该变量也通常作为`find_path`的结果变量 |

### 9.4.7　查找结果变量

查找模块最终会将查找到的第三方软件包的相关信息存入一些变量中，这些变量的命名遵循一定惯例：通常都将<软件包名>_作为前缀（大小写也应一致）。这样可以避免不同查找模块的结果变量的名称出现冲突。

下表中是一些常见的查找结果变量。

| 变量名称 | 描述 |
| --- | --- |
| `<软件包名>_INCLUDE_DIRS` | 头文件搜索目录列表 |
| `<软件包名>_LIBRARIES` | 库列表。其中的元素可以是库目标的名称、库文件的绝对路径，或位于库文件搜索目录的库名称 |
| `<软件包名>_DEFINITIONS` | 作为使用要求的编译器宏定义 |
| `<可执行文件名>_EXECUTABLE` | 可执行文件的绝对路径。该变量名称的前缀未必是 <软件包名>，也可以是搜索到的可执行文件的名称（一般全大写），通常用于存放该软件包提供的工具的路径。该变量通常是缓存变量，作为`find_program`的结果变量 |
| `<软件包名>_<可执行文件名>_EXECUTABLE` | 可执行文件的绝对路径。该变量通常是缓存变量，作为`find_program`的结果变量。该变量可以用于避免不同查找模块所查找的可执行文件重名 |
| `<软件包名>_LIBRARY_DIRS` | 库文件搜索目录 |
| `<软件包名>_RUNTIME_LIBRARY_DIRS` | 运行时库文件搜索目录。在Windows操作系统中，该变量的值通常用于设置PATH环境变量；在类UNIX操作系统中，该变量的值通常用于设置`LD_LIBRARY_PATH`环境变量 |
| `<软件包名>_ROOT_DIR` | 软件包的根目录路径 |
| `<软件包名>_FOUND` | 软件包是否成功查找到 |
| `<软件包名_<子组件>_FOUND` | 软件包对应<子组件>是否成功查找到。这些变量通常被调用者用于判断可选子组件是否成功查找到，以确定最终链接哪些子组件 |
| `<软件包名>_VERSION` `<软件包名>_VERSION_STRING` | 查找到的软件包的版本号 |
| `<软件包名>_VERSION_MAJOR` `<软件包名>_VERSION_MINOR` `<软件包名>_VERSION_PATCH` | 查找到的软件包的版本号的指定部分 |

`<软件包名>_INCLUDE_DIRS`和`<软件包名>_LIBRARIES`这两个结果变量很容易和查找条件变量`<软件包名>_INCLUDE_DIR`和`<软件包名>_LIBRARY`混淆。查找条件变量是缓存变量，经常直接用作`find_file`等命令的结果缓存变量，可以支持自定义路径，避免重复查找。而这里的查找结果变量是最终的结果变量，可能包含多个路径。一般情况下，直接将全部查找条件变量的值赋给查找结果变量即可。

### 9.4.8　FindPackageHandleStandardArgs模块

FindPackageHandleStandardArgs模块是一个CMake预置的功能模块，可不要因为它的名称以Find开头就认为它是查找模块。

该模块提供了两个命令，可以在自定义查找模块中判断结果变量是否都已正确赋值，同时还能检查查找到的软件包版本号等信息是否满足查找的要求。

#### 检查条件变量：find_package_handle_standard_args

该命令的名称与模块名称一致，也是该模块提供的最重要的命令。它具有如下简单形式和完整形式：

```
find_package_handle_standard_args(<软件包名>   #简化形式
    DEFAULT_MSG|<自定义错误>
    <待检查的变量>...
)
　
find_package_handle_standard_args(<软件包名>   #完整形式
    [FOUND_VAR <查找结果状态变量>] # 已弃用
    [REQUIRED_VARS <待检查的变量>...]
    [VERSION_VAR <版本号变量>]
    [HANDLE_VERSION_RANGE]
    [HANDLE_COMPONENTS]
    [REASON_FAILURE_MESSAGE <错误原因>]
    [FAIL_MESSAGE <自定义错误>]
    [NAME_MISMATCHED]
    [CONFIG_MODE]
)
```

其中，简单形式的`DEFAULT_MSG`参数表示在查找失败时输出默认错误信息，其他参数与完整形式中同名参数的含义相同，因此接下来仅会介绍完整形式中的参数。

该命令会检查<待检查的变量>中的值是否都是有效值，检查<版本号变量>的值是否满足`find_package`的参数设置、子组件是否都已成功查找到等，并根据这些检查结果设置<查找结果状态变量>的值。当所有检查均通过时，其值为真值，否则为假值。

`FOUND_VAR`参数用于设置<查找结果状态变量>的名称，名称仅可取值为`<软件包名>_FOUND`或全大写的`<软件包名>_FOUND`之一。该参数在CMake 3.3版本之后已被弃用，因为这两种写法的变量都会同时被隐式地作为<查找结果状态变量>，该参数不再起任何作用。

`REQUIRED_VARS`参数用于设置<待检查的变量>。当这些变量未被正确定义时，该命令会报告错误，要求用户设置缺失的变量值。这些变量应当是可以设置的缓存变量，如查找条件变量，因此该参数通常用于检查`find_file`等查找命令的结果路径是否被正确设置。

`VERSION_VAR`参数用于设置<版本号变量>。该命令会检查<版本号变量>的值是否与调用`find_package`命令时通过参数设置的版本号相匹配。若想让该命令检查其值是否满足区间形式的版本号要求，需要额外指定`HANDLE_VERSION_RANGE`参数。

指定`HANDLE_COMPONENTS`参数后，该命令会检查所有的必要子组件是否成功查找到（可选子组件不会被检查），也就是检查这些子组件对应的`<软件包名>_<子组件>_FOUND`变量是否均为真值。

`REASON_FAILURE_MESSAGE`参数可以用于补充<错误原因>，并被追加到输出的软件包查找失败提示信息之后。而`FAIL_MESSAGE`参数用于指定<自定义错误>，替代默认的软件包查找失败提示信息，因此一般不推荐设置该参数。

该命令还会在<软件包名>参数与`CMAKE_FIND_PACKAGE_NAME`变量的值不一致时产生警告信息，这通常意味着查找模块的代码有错误。当然，如果这个不一致是预期内的，那么可以指定`NAME_MISMATCHED`参数来避免误报。

另外，有时编写的查找模块可能会嵌套调用`find_package`命令的配置模式来查找软件包。指定`CONFIG_MODE`参数后，`find_package_handle_standard_args`命令可以用来检查配置模式是否成功执行，以及配置模式查找到的软件包版本号是否满足要求。这种嵌套配置模式的查找模块相当于配置模式的再封装，例如，可以在查找模块首先为配置模式指定路径提示，提供一些额外的功能命令等。

#### 检查版本号：find_package_check_version

这是FindPackageHandleStandardArgs模块提供的另一个命令，功能较为纯粹，仅用于检查版本号：

```
find_package_check_version(<版本号> <结果变量>
    [HANDLE_VERSION_RANGE]
    [RESULT_MESSAGE_VARIABLE <检查信息变量>]
)
```

该命令用于检查<版本号>是否满足`find_package`命令被调用时所设置的版本号参数的要求，并将检查结果以布尔值存入<结果变量>。

`HANDLE_VERSION_RANGE`参数同样用于设置是否支持对版本号区间形式的检查。

检查结果的详细信息会存入<检查信息变量>。一般会将其值在检查后输出，如下所示。

```
find_package_check_version(1.2.0 res
    HANDLE_VERSION_RANGE
    RESULT_MESSAGE_VARIABLE msg)
if(res)
    message(STATUS "${msg}")
else()
    message(FATAL_ERROR "${msg}")
endif()
```

### 9.4.9　实例：onnxruntime的查找模块

onnxruntime是微软开发的一个跨平台的高性能机器学习推理和训练加速库，被广泛应用于产业界。如果我们想将一些机器学习、深度学习模型应用于低延迟服务中，可以考虑使用onnxruntime库。

onnxruntime库没有官方提供的便于导入依赖的CMake配置文件。因此，为了能够借助`find_package`命令简化导入过程，我们需要自行实现一个针对onnxruntime库的查找模块`Findonnxruntime.cmake`。

现在下载onnxruntime库的预编译包并解压到安装目录，查看其目录结构，以便接下来编写查找模块时确定查找头文件、库文件等的候选路径参数等。onnxruntime库的预编译包可以在其官方GitHub代码仓库的Release页面中下载，其中包含不同平台的预编译包，读者可根据自己使用的平台进行选择。笔者使用的是64位Windows操作系统，且未配置CUDA运行环境，不打算利用GPU加速功能，因此选择的是`“onnxruntime-win-x64-<版本号>.zip”`这一预编译包。

为了方便演示，将该压缩包解压至`CMake-Book/src/ch009/Findonnxruntime/<压缩包名>`目录，其中包含两个子文件夹：include和lib。很明显，前者包含onnxruntime库的头文件，后者则包含其库文件，结果简单清晰。那么，下面就开始编写查找模块吧！

首先，利用`find_path`命令查找`“onnxruntime_c_api.h”`头文件的所在目录，如下所示。

```
find_path(onnxruntime_INCLUDE_DIR onnxruntime_c_api.h
  HINTS ENV onnxruntime_ROOT
  PATH_SUFFIXES include)
```

这里将环境变量`onnxruntime_ROOT`的值作为候选路径，其值应被设置为onnxruntime库的安装根目录。另外，这里还将include设置为查找的子目录。`find_path`的查找结果将被存入`onnxruntime_INCLUDE_DIR`缓存变量中，也就是onnxruntime库的头文件目录。

同理，再利用`find_library`命令查找onnxruntime库的路径，如下所示。

```
find_library(onnxruntime_LIBRARY
  NAMES onnxruntime
  HINTS ENV onnxruntime_ROOT
  PATH_SUFFIXES lib)
```

这里也将环境变量`onnxruntime_ROOT`的值作为候选路径参数。现在，`onnxruntime_LIBRARY`结果缓存变量的值就是onnxruntime库的库文件路径了。

接下来还需要获取查找到的onnxruntime库的版本号，以便后面判断它是否符合`find_package`的参数要求。在onnxruntime的安装根目录中，有一个名为`VERSION_NUMBER`的文件，其所含内容正是版本号。我们可以直接使用`find_file`命令查找它，并读取其中的内容，如下所示。

```
find_file(onnxruntime_VERSION_FILE VERSION_NUMBER
  HINTS ENV onnxruntime_ROOT)

if(onnxruntime_VERSION_FILE)
  file(STRINGS ${onnxruntime_VERSION_FILE} onnxruntime_VERSION LIMIT_COUNT 1)
endif()
```

此时，`onnxruntime_VERSION_FILE`缓存变量的值是版本号文件的路径，`onnxruntime_VERSION`变量的值是版本号文件的内容，也就是版本号本身。

现在，所有必要的信息都已经获取完成，是时候使用FindPackageHandleStandardArgs模块来检查这些信息了！如下所示的查找模块片段中，我们调用了`find_package_handle_standard_args`命令以检查路径变量是否被正确设置，以及版本号是否满足要求。

```
include(FindPackageHandleStandardArgs)

find_package_handle_standard_args(onnxruntime 
  REQUIRED_VARS onnxruntime_LIBRARY onnxruntime_INCLUDE_DIR 
  VERSION_VAR onnxruntime_VERSION
  HANDLE_VERSION_RANGE)
```

此时，`onnxruntime_FOUND`变量就会根据查找成功与否，被赋值为真值或假值。

最后，创建一个导入库目标onnxruntime::onnxruntime，以方便用户链接到它，如下所示。

```
if(onnxruntime_FOUND)
  set(onnxruntime_INCLUDE_DIRS ${onnxruntime_INCLUDE_DIR})
  set(onnxruntime_LIBRARIES ${onnxruntime_LIBRARY})

  add_library(onnxruntime::onnxruntime SHARED IMPORTED)
  target_include_directories(onnxruntime::onnxruntime INTERFACE ${onnxruntime_INCLUDE_DIRS})
  if(WIN32)
    set_target_properties(onnxruntime::onnxruntime PROPERTIES 
      IMPORTED_IMPLIB "${onnxruntime_LIBRARY}")
  else()
    set_target_properties(onnxruntime::onnxruntime PROPERTIES 
      IMPORTED_LOCATION "${onnxruntime_LIBRARY}")
  endif()
endif()
```

在Windows操作系统中，导入动态库的`IMPORTED_IMPLIB`属性应设置为动态库的导入库（`.lib`而不是`.dll`）的路径；在其他平台中，导入动态库的`IMPORTED_LOCATION`属性应设置为导入的动态库文件自身的路径。这一点需要注意。

至此，查找模块的全部代码就完成了。但查找模块真的完成了吗？还没有。按照惯例，应提供一个友好全面的注释文档，放在查找模块的最前面。不然，用户怎么知道这个查找模块会定义哪些结果变量，又会受到哪些条件变量控制呢？CMake的注释文档一般采用reStructuredText语法来书写，如下所示。

```
#[=======================================================================[.rst:
Findonnxruntime
-------

Finds the onnxruntime library.（查找onnxruntime库）

Imported Targets（导入目标）
^^^^^^^^^^^^^^^^^^^^^^^^^^

This module provides the following imported targets, if found（若查找成功，该模块会创建如下导入目标）:

``onnxruntime::onnxruntime``
  The onnxruntime library（onnxruntime库）

Result Variables（结果变量）
^^^^^^^^^^^^^^^^^^^^^^^^^^

This will define the following variables（该模块会定义如下变量）:

``onnxruntime_FOUND``
  True if the system has the onnxruntime library.（若成功查找onnxruntime库，则为真值）
``onnxruntime_VERSION``
  The version of the onnxruntime library which was found.（查找到的onnxruntime库的版本号）
``onnxruntime_INCLUDE_DIRS``
  Include directories needed to use onnxruntime.（作为使用要求的onnxruntime的头文件目录）
``onnxruntime_LIBRARIES``
  Libraries needed to link to onnxruntime.（作为使用要求的onnxruntime的链接库文件路径）

Cache Variables（缓存变量）
^^^^^^^^^^^^^^^^^^^^^^^^^

The following cache variables may also be set（该模块会定义如下缓存变量）:

``onnxruntime_INCLUDE_DIR``
  The directory containing ``onnxruntime_c_api.h``.（``onnxruntime_c_api.h``所在目录）
``onnxruntime_LIBRARY``
  The path to the onnxruntime library.（onnxruntime库文件的路径）

Hints（作为提示的查找条件变量）
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``onnxruntime_ROOT`` 
  The environment variable that points to the root directory of onnxruntime.（指向onnxruntime安装根目录的环境变量）
#]=======================================================================]
```

现在，我们终于彻底完成了查找模块的全部内容，赶紧在项目中试用一下吧！首先，创建一个CMake目录程序，如下所示。

```
cmake_minimum_required(VERSION 3.20)

project(find-onnxruntime)

set(CMAKE_MODULE_PATH "${CMAKE_CURRENT_LIST_DIR};${CMAKE_MODULE_PATH}")

set(CMAKE_CXX_STANDARD 11)# 设置C++标准为11

set(onnx_version 1.10.0)# 根据下载的版本进行设置，本例使用1.10.0版本

# 请下载onnxruntime库的压缩包，并解压至该目录中

# 下面的环境变量设置仅做演示目的，实际开发中不应将依赖的安装目录硬编码在代码中

if("$ENV{onnxruntime_ROOT}" STREQUAL "")
  if(WIN32)
      set(ENV{onnxruntime_ROOT} "${CMAKE_CURRENT_LIST_DIR}/onnxruntime-win-x64-${onnx_version}")
  elseif(APPLE)
      set(ENV{onnxruntime_ROOT} "${CMAKE_CURRENT_LIST_DIR}/onnxruntime-osx-universal2-${onnx_version}")
  else()
      set(ENV{onnxruntime_ROOT} "${CMAKE_CURRENT_LIST_DIR}/onnxruntime-linux-x64-${onnx_version}")
  endif()
endif()

find_package(onnxruntime 1.10) #指定依赖的最小版本

add_executable(main main.cpp)

target_link_libraries(main onnxruntime::onnxruntime)

target_compile_definitions(main PRIVATE ORT_NO_EXCEPTIONS)
```

该目录程序首先向`CMAKE_MODULE_PATH`变量中插入了当前目录，以便调用刚刚编写的查找模块，接着设置`onnxruntime_ROOT`环境变量为onnxruntime库的安装根目录，用来提示查找模块，然后调用`find_package`命令，真正开始查找onnxruntime库，最后创建一个可执行文件目标，并链接了`onnxruntime::onnxruntime`这个导入库目标。

如上所示中还为可执行文件目标设置了`ORT_NO_EXCEPTIONS`宏定义，主要是为了便于演示，让onnxruntime在遇到错误时输出错误信息而非抛出异常。

主程序如下所示，其中几乎没有什么内容，仅仅试图从一个非法的空路径中加载模型文件。

```
#include <onnxruntime_cxx_api.h>

int main() {
    Ort::Env env;
    Ort::Session session(env, ORT_TSTR(""), Ort::SessionOptions(nullptr));
    return 0;
}
```

下面就可以配置生成该项目了。

```
> cd CMakeBook/src/ch009/Findonnxruntime
> mkdir build
> cd build
> cmake ..
...
-- Found onnxruntime: .../Findonnxruntime/onnxruntime-win-x64-1.10.0/lib/onnxruntime.lib (found suitable version "1.10.0", minimum required is "1.10")
...
> cmake --build .
...
```

读者可以尝试运行构建好的可执行文件main，它没有什么实际用处，只会输出一条错误信息，即加载的模型文件不存在。不过这证明了onnxruntime已经被成功链接到主程序中了。

若要在Windows操作系统中运行main.exe，记得将动态库onnxruntime.dll复制到main.exe的同一目录中。

## 9.5　小结

本章的前半部分介绍了CMake提供的一些常用的预置模块，善用它们可以极大地简化CMake程序的编写过程，方便调试程序、获取系统环境信息等。另外，本章结合丰富的实例介绍了用于生成导出头文件的GenerateExportHeader模块。这个模块可以便捷地导出接口函数，对于库的开发者来说非常实用。

本章的后半部分主要介绍查找模块，并以CMake预置的Boost查找模块为例展示了其具体应用。除此之外，本章还介绍了如何从头编写一个自定义查找模块，包括编写过程中常用的一些find命令、功能模块等，最后通过一个onnxruntime的自定义查找模块实例详细解释了编写自定义查找模块的完整过程。

CMake模块程序是CMake程序复用的基本单元，社区中很多热心的开发者开源了他们自己编写的CMake模块。读者如果有一些个性化的需求，不妨先在互联网上搜索，看看是否能够复用已有的模块。当然，热心的读者也可以将自己编写的模块开放给大家！

第10章将一起认识CMake的策略特性，正是这一特性让CMake可以一直保持良好的向后兼容性。这也是本书介绍的最后一个CMake特性了，一起继续吧！
