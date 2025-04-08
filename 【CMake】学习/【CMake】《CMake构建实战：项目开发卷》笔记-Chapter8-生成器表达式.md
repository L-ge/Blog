# 第8章　生成器表达式

生成器表达式（generator expression）是由CMake生成器进行解析的表达式，因此，这些表达式只有在CMake的生成阶段才被解析为具体的值。

CMake在生成阶段，能够根据具体选用的构建系统生成器生成特定的配置，因此生成器表达式常常用于获取与构建系统相关的信息，如当前构建模式（Debug或Release等）、构建目标对应的二进制文件名称等。一般来说，这些信息在构建系统生成器确定之前，也就是在配置阶段时，是无从知晓的。当然也有一些特殊情况，如对于单构建模式的构建系统来说，当前构建模式是在配置阶段由`CMAKE_BUILD_TYPE`变量确定的。不过，生成器表达式对它们同样有效，因此更为通用。

在构建跨平台程序的过程中，可能涉及不同的构建系统，而生成器表达式对它们之间的差异做了很好的封装隐藏，因此不可避免地会用到生成器表达式。CMake中很多与构建相关的命令都支持生成器表达式作为参数的一部分。本章将介绍一些常用的生成器表达式及其在一些常见命令中的用法。

生成器表达式大体分为两类：布尔型生成器表达式和字符串生成器表达式，后面将会依次介绍它们。

## 8.1　支持生成器表达式的命令

在介绍各种生成器表达式之前，首先来了解一下它们到底能用在哪些命令中。其实，有很多命令支持将生成器表达式作为参数的一部分。基本上只要在生成阶段工作的命令都会对生成器表达式有所支持。早在介绍string命令和file命令时，就已经提到过生成器表达式了，下面做个简单回顾。

- string(GENEX_STRIP)命令用于删除输入字符串中的生成器表达式。事实上，该命令与生成器表达式的解析无关，因为它仅仅是根据语法删除字符串中的生成器表达式。该命令在CMake配置阶段执行。

- file(GENERATE)命令用于在CMake生成阶段为每一个构建模式生成一个指定的文件。其中与输出文件名、文件内容、生成条件、构建目标等相关的参数中均可包含生成器表达式。该命令常用于调试生成器表达式的值。

message命令在CMake配置阶段执行，不能正确解析并输出生成器表达式的值。这也是为什么常使用file(GENERATE)命令调试生成器表达式的值，而不能用message命令。

### 8.1.1　创建构建目标的命令

还记得第7章讲过的下面几个命令吗？它们分别用于创建可执行文件目标和一般库目标：

```
add_executable(<目标名称> [<源文件>...])
add_library(<目标名称> <库类型> [<源文件>...])
```

这些命令的<源文件>参数中可以使用生成器表达式。这样，就可以根据不同的构建模式使用不同的源文件来编译构建目标。下面的实例展示了这一用途，其中涉及尚未讲解的生成器表达式`$<CONFIG>`及其语法，读者暂时不必深究。

#### 实例

首先，为Debug和Release两种构建模式分别创建不同的主程序源文件，用于输出不同的字符串，如下所示。

```
#include <stdio.h>

int main() {
    printf("Debug\n");
    return 0;
}
```

```
#include <stdio.h>

int main() {
    printf("Release\n");
    return 0;
}
```

然后，在CMake目录程序中创建一个源文件目标，使用`$<CONFIG>`生成器表达式分别为Debug和Release两种构建模式设置不同的源文件，如下所示。

```
cmake_minimum_required(VERSION 3.20)

project(build_config_and_sources)

add_executable(main 
    $<$<CONFIG:Debug>:debug.c>
    $<$<CONFIG:Release>:release.c>
)
```

项目创建完毕，接下来着手构建该项目。

#### 使用Visual Studio构建实例

Visual Studio支持多构建模式，使用该构建系统来构建该项目的过程如下：

```
> cd CMakeBook\src\ch008\构建模式与源文件
> mkdir build
> cd build
> cmake ..
...
> cmake --build . --config Debug
...
> cmake --build . --config Release
...
```

至此，我们成功构建了Debug和Release两种构建模式的main.exe可执行文件。下面立刻运行一下它们：

```
> .\Debug\main.exe
Debug
> .\Release\main.exe
Release
```

果然！两个可执行文件分别编译了不同的源文件，输出了不同的内容。

#### 使用Makefile构建实例

这里同样演示一下如何使用Makefile构建系统来构建该项目。Makefile构建系统仅支持单构建模式，因此一个构建目录中只能使用一种构建模式来构建项目，而且在CMake配置生成阶段还要通过`CMAKE_BUILD_TYPE`变量指定构建模式。该项目Debug模式的构建过程如下：

```
> cd CMakeBook/src/ch008/构建模式与源文件
> mkdir build_debug
> cd build_debug
> cmake -DCMAKE_BUILD_TYPE=Debug ..
...
> cmake --build .
...
> ./main
Debug
```

可以看到最终可执行文件输出的正是“Debug”字符串。Release模式的构建过程几乎一样，这里不再赘述，读者可以自己动手尝试一下。

### 8.1.2　属性相关命令

除了`link_libraries`命令外，第7章介绍的用于设置目录属性及目标属性的命令也都支持使用生成器表达式作为参数。因此，我们可以为目录或构建目标针对不同的构建模式设置不同的属性值。

### 8.1.3　自定义构建规则和目标

下面是第7章中介绍的两个命令，它们分别用于创建自定义构建规则和自定义构建目标：

```
add_custom_command
add_custom_target
```

关于这两个命令中的哪些参数支持生成器表达式，第7章中均已介绍，这里不再赘述。

#### 调试生成器表达式

前面提到file(GENERATE)命令可以用于调试生成器表达式。不过这仍稍显麻烦，需要通过输出文件来查看生成器表达式的值。其实，借助自定义构建目标，可以在构建过程中输出生成器表达式的值。如下所示的CMake目录程序展示了这一用法。

```
cmake_minimum_required(VERSION 3.20)

project(output_gen_exp)

# 构建debug-gen-exp自定义目标，以输出当前构建模式
add_custom_target(debug-gen-exp 
    COMMAND ${CMAKE_COMMAND} -E echo "CONFIG: $<CONFIG>"
)
```

使用Visual Studio生成器配置生成该项目，并构建“debug-gen-exp”自定义目标。其构建过程如下：

```
> cd CMakeBook\src\ch008\输出表达式
> mkdir build
> cd build
> cmake ..
...
> cmake --build . --config Debug --target debug-gen-exp
...
CONFIG: Debug
...
> cmake --build . --config Release --target debug-gen-exp
...
CONFIG: Release
...
```

如果使用仅支持单构建模式的生成器，如Makefile生成器，请务必在配置生成阶段指定CMake变量`CMAKE_BUILD_TYPE`的值为某个构建模式。否则，`$<CONFIG`>生成器表达式的值将为空值。

## 8.2　布尔型生成器表达式

在生成阶段，布尔型生成器表达式会被解析为0或1。布尔型生成器表达式通常作为分支条件的参数嵌套在字符串生成器表达式中。有关分支型字符串生成器表达式的内容参见8.3.2小节。

此前在介绍CMake命令参数时，表示生成器表达式的语法形式为`“$<生成器表达式>”`，其中尖括号是语法的组成部分。因此，本章介绍生成器表达式中的参数时，将不再使用尖括号表示参数占位符，而是使用花括号`{`和`}`。

### 8.2.1　转换字符串为布尔值：BOOL

```
$<BOOL:{字符串}>
```

该表达式会把`{字符串}`转换为0或1。仅当下列情况之一成立时，其值为0：

- {字符串}为空；

- {字符串}为假值常量，即NOTFOUND、IGNORE、OFF、NO、N、FALSE、0或以`-NOTFOUND`结尾（均不区分大小写）。

其他情况下，其值为1。

### 8.2.2　逻辑运算

本小节中的生成器表达式用于执行逻辑运算，如下表所示。其中`{条件}`参数可以是嵌套的布尔型生成器表达式。

| 表达式 | 描述 |
| --- | --- |
| `$<AND:{条件1}[,{条件2}]...>` | 逻辑与运算，当所有{条件}都为真时，其值为1，否则为0 |
| `$<OR:{条件1}[,{条件2}]...>` | 逻辑或运算，当任一{条件}为真时，其值为1，否则为0 |
| `$<NOT:{条件}>` | 逻辑非运算，当{条件}为假时，其值为1，否则为0 |

#### 实例

下面是展示上述用于逻辑运算的生成器表达式的实例，其中会使用自定义构建目标命令来调试生成器表达式的值。其CMake目录程序如下所示。

```
cmake_minimum_required(VERSION 3.20)

project(logical_calc)

add_custom_target(debug-gen-exp ALL
    COMMAND ${CMAKE_COMMAND} -E echo 
        "$<BOOL:ABC>" # 输出：1
    COMMAND ${CMAKE_COMMAND} -E echo 
        "$<BOOL:NO>" # 输出：0
    COMMAND ${CMAKE_COMMAND} -E echo 
        "$<NOT:$<BOOL:NO>>" # 输出：1
    COMMAND ${CMAKE_COMMAND} -E echo 
        "$<AND:$<BOOL:A>,$<BOOL:X-NOTFOUND>>" # 输出：0
    COMMAND ${CMAKE_COMMAND} -E echo 
        "$<OR:$<BOOL:A>,$<BOOL:X-NOTFOUND>>" # 输出：1
)
```

### 8.2.3　关系比较

本小节中的生成器表达式均用于进行关系的比较或判断。

#### 数值相等：EQUAL

```
$<EQUAL:{数值1},{数值2}>
```

当{数值1}与{数值2}相等时，该表达式的值为1，否则为0。该表达式仅支持比较整数。

#### 字符串相等：STREQUAL

```
$<STREQUAL:{字符串1},{字符串2}>
```

当{字符串1}与{字符串2}相等时，该表达式的值为1，否则为0。该比较是区分大小写的，如果希望比较时不区分大小写，可以使用`LOWER_CASE`或`UPPER_CASE`字符串变换表达式将字符串大小写统一（参见8.3.3小节）。

#### 列表元素判断：IN_LIST

```
$<IN_LIST:{字符串},{列表}>
```

当{字符串}是{列表}（分号分隔的列表字符串）中的一个元素时，该表达式的值为1，否则为0。字符串的比较是区分大小写的。

#### 版本号关系比较：VERSION_

用于版本号关系比较的生成器表达式如下表所示。

| 表达式 | 比较关系 |
| --- | --- |
| `$<VERSION_LESS:{版本号1},{版本号2}>` | 小于 |
| `$<VERSION_LESS_EQUAL:{版本号1},{版本号2}>` | 小于等于 |
| `$<VERSION_EQUAL:{版本号1},{版本号2}>` | 等于 |
| `$<VERSION_GREATER_EQUAL:{版本号1},{版本号2}>` | 大于等于 |
| `$<VERSION_GREATER:{版本号1},{版本号2}>` | 大于 |

当{版本号1}和{版本号2}满足对应关系时，该表达式的值为1，否则为0。版本号的具体比较规则参见3.8.4小节中对版本号比较的双参数条件的介绍。

#### 实例

下面是展示上述用于关系比较的生成器表达式的实例。其CMake目录程序如下所示。

```
cmake_minimum_required(VERSION 3.20)

project(comparison)

add_custom_target(debug-gen-exp ALL
    COMMAND ${CMAKE_COMMAND} -E echo 
        "$<EQUAL:0,-0>" # 输出：1
    COMMAND ${CMAKE_COMMAND} -E echo 
        "$<STREQUAL:0,-0>" # 输出：0
    COMMAND ${CMAKE_COMMAND} -E echo 
        "$<IN_LIST:2,1;2;3>" # 输出：1
    COMMAND ${CMAKE_COMMAND} -E echo
        "$<VERSION_LESS:1.1.2,1.2.0>" # 输出：1
)
```

### 8.2.4　谓词查询

本小节中的生成器表达式均用于查询某些谓词的结果是否满足一定的条件，通常用来判断当前构建系统的环境配置等信息是否满足要求。

#### 构建目标存在性：TARGET_EXISTS

```
$<TARGET_EXISTS:{构建目标}>
```

当{构建目标}存在时，该表达式的值为1，否则为0。

#### 当前构建模式：CONFIG

```
$<CONFIG:{构建模式1}[,{构建模式2}]...>
```

当当前构建模式是指定的{构建模式}之一时，该表达式的值为1，否则为0。该表达式在比较构建模式的字符串时不区分大小写。

#### 当前操作系统：PLATFORM_ID

```
$<PLATFORM_ID:{操作系统1}[,{操作系统2}]...>
```

当当前操作系统是指定的{操作系统}之一时，该表达式的值为1，否则为0。当前操作系统的值即变量`CMAKE_SYSTEM_NAME`的值，一般为Linux、Windows或Darwin（macOS）之一。

#### 当前编译器：{编程语言}_COMPILER_ID

```
$<{编程语言}_COMPILER_ID:{编译器1}[,{编译器2}]...>
```

当前对指定{编程语言}使用的编译器是指定的{编译器}之一时，该表达式的值为1，否则为0。其中{编程语言}仅支持如下取值：

- C；

- CXX，即`C++`；

- CUDA；

- OBJC，即`Objective-C`；

- OBJCXX，即`Objective-C++`；

- Fortran；

- HIP；

- ISPC。

当前编译器的值即变量`CMAKE_<编程语言>_COMPILER_ID`的值，最常见的取值如下：

- AppleClang，即Apple的Clang编译器；

- Clang，即LLVM的Clang编译器；

- GNU，即GNU的GCC编译器；

- Intel，即Intel的编译器；

- MSVC，即`Microsoft Visual C++`编译器；

- NVIDIA，即NVIDIA的CUDA编译器。

#### 当前编译器版本号：{编程语言}_COMPILER_VERSION

```
$<{编程语言}_COMPILER_VERSION:{版本号}...>
```

当前对指定{编程语言}使用的编译器版本号与指定的{版本号}匹配时，该表达式的值为1，否则为0。{编程语言}的取值与“当前编译器”表达式中对应参数的取值一致。

当前编译器版本号的值即变量`CMAKE_<编程语言>_COMPILER_VERSION`的值。

#### 当前支持的编译特性：COMPILE_FEATURES

```
$<COMPILE_FEATURES:{编译特性1}[,{编译特性2}]...>
```

当前编译器支持的编译特性包含全部指定的{编译特性}时，该表达式的值为1，否则为0。

#### 当前编程语言：COMPILE_LANGUAGE

```
$<COMPILE_LANGUAGE:{编程语言1}[,{编程语言2}]...>
```

当前编译单元对应的编程语言是指定的{编程语言}之一时，该表达式的值为1，否则为0。使用该表达式，可以在使用多种编程语言构建的目标中对不同编程语言的源文件进行不同的属性设置，包括编译选项、宏定义、头文件目录的设置。

对于Visual Studio和Xcode生成器而言，CMake无法分别对C和`C++`语言的源文件设置不同的宏定义或头文件目录；对于Visual Studio生成器而言，甚至不能为C和`C++`语言设置不同的编译选项。在使用这两个生成器时，当目标的源文件中包含`C++`源文件时，该表达式一律按编程语言为CXX来匹配参数，否则按编程语言为C进行匹配。

#### 当前编程语言和编译器：COMPILE_LANG_AND_ID

```
$<COMPILE_LANG_AND_ID:{编程语言},{编译器1}[,{编译器2}]...>
```

该表达式等价于下面的复合表达式，可以同时对当前编译单元对应的编程语言及其使用的编译器进行判断：

```
$<AND:$<COMPILE_LANGUAGE:{编程语言}>,$<{编程语言}_COMPILER_ID:{编译器1}[,{编译器2}]...>
```

#### 实例：“当前编程语言”表达式的应用

本小节涉及的表达式较多，这里仅对“当前编程语言”表达式进行实例展示。CMake目录程序如下所示。其中定义了一个可执行文件目标main，它包含三个源文件：a.c、b.cpp和main.c。然后，在该目录程序中分别为C语言和`C++`语言的编译单元设置了不同的宏定义：A=1和A=10，这里结合了“当前编程语言”表达式及尚未介绍的条件表达式来实现条件定义。有关条件表达式的内容参见8.3.2小节。

```
cmake_minimum_required(VERSION 3.20)

project(compile_lang)

add_executable(main a.c b.cpp main.c)
target_compile_definitions(main PRIVATE
    $<$<COMPILE_LANGUAGE:C>:A=1>
    $<$<COMPILE_LANGUAGE:CXX>:A=10>
)
```

可执行文件的三个源文件的代码分别如下所示。

```
#include <stdio.h>

void print_c() { printf("%d\n", A); }
```

```
#include <iostream>

extern "C" void print_cpp() { std::cout << A << std::endl; }
```

```
extern void print_c();
extern void print_cpp();

int main() {
    print_c();
    print_cpp();
    return 0;
}
```

主程序分别调用了定义在a.c和b.cpp中的函数`print_c`和`print_cpp`，用于输出各自编译单元（即源文件）中宏A的值。

由于Visual Studio生成器无法很好地区分C语言和`C++`语言的编译单元，下面使用Makefile生成器构建该项目，并执行构建生成的可执行文件，过程如下：

```
$ cd CMakeBook/src/ch008/当前编程语言
$ mkdir build
$ cd build
$ cmake ..
...
$ cmake --build . # 或make
...
$ ./main
1
10
```

可见，对于使用不同编程语言的编译单元，宏定义的设置不同，执行程序后输出的值也就不同。

## 8.3　字符串生成器表达式

顾名思义，字符串生成器表达式在生成阶段会被解析为一段字符串。与布尔型生成器表达式一样，字符串生成器表达式也可以嵌套使用在其他表达式的参数中。

### 8.3.1　字符转义

```
$<ANGLE-R> # 转义为">"
$<COMMA> # 转义为","
$<SEMICOLON> # 转义为";"
```

`“>”``“,”``“;”`这三个字符通常用于构成生成器表达式的语法结构，因此需要通过字符转义表达式来表示。

### 8.3.2　条件表达式：IF

```
$<IF:{条件},{字符串1},{字符串2}>
```

该表达式的值取决于{条件}的值：当其为1时，该表达式的值将被解析为{字符串1}，否则将被解析为{字符串2}。

如果给定的{字符串2}为空字符串，那么不妨使用下面这个简化的形式：

```
$<{条件}:{字符串}>
```

#### 实例

下面这个实例可以在构建过程中输出当前的构建模式是否是调试模式，CMake目录程序如下所示。

```
cmake_minimum_required(VERSION 3.20)

project(condition)

add_custom_target(debug-gen-exp ALL
    COMMAND ${CMAKE_COMMAND} -E echo 
        "Is Debug: <$<IF:$<CONFIG:Debug>,Yes,No>$<ANGLE-R>"
    VERBATIM
)
```

读者可以分别使用Debug和Release模式配置生成该项目，然后构建debug-gen-exp目标，看看是否会分别输出`“Is Debug: <Yes>”`和`“Is Debug: <No>”`。注意，目录程序中还使用了`$<ANGLE-R>`生成器表达式来转义右尖括号。

### 8.3.3　字符串变换

#### 分隔符连接：JOIN

```
$<JOIN:{列表字符串},{分隔符}>
```

该表达式会将{列表字符串}中的每一个元素以指定的{分隔符}分隔并连接在一起，作为最终的值。例如，`“$<JOIN:1;2;3,$<COMMA>>”`将被解析为`“1,2,3”`。

#### 大小写转换：LOWER_CASE和UPPER_CASE

```
$<LOWER_CASE:{字符串}> # 转小写
$<UPPER_CASE:{字符串}> # 转大写
```

这两个表达式的值分别为其{字符串}参数转换为小写和大写后的结果。

#### 移除重复元素：REMOVE_DUPLICATES

```
$<REMOVE_DUPLICATES:{列表字符串}>
```

该表达式会将{列表字符串}中重复的元素移除，并将最终的列表字符串作为表达式的值。

#### 列表筛选：FILTER

```
$<FILTER:{列表字符串},INCLUDE|EXCLUDE,{正则表达式}>
```

该表达式会将{列表字符串}中能够匹配到{正则表达式}的元素保留（若指定了INCLUDE参数）或移除（若指定了EXCLUDE参数），并将最终的列表字符串作为表达式的值。

#### 生成C标识符：MAKE_C_IDENTIFIER

```
$<MAKE_C_IDENTIFIER:{字符串}>
```

该表达式会将{字符串}转换为合法的C标识符，并作为最终的值。与`string(MAKE_C_IDENTIFIER)`命令一样，它也将非字母或数字的字符转换为下画线。

#### 转换为Shell路径：SHELL_PATH

```
$<SHELL_PATH:{绝对路径列表字符串}>
```

该表达式会将{绝对路径列表字符串}中的每一个绝对路径的格式转换为当前Shell所需的路径格式。

例如，在Windows操作系统中，如果使用命令提示符或者PowerShell生成该项目，这些路径会被转换为以反斜杠作为路径分隔符的格式；如果使用MSYS或者Cygwin等Shell生成该项目，这些路径就会被转换为以斜杠作为路径分隔符的POSIX路径格式。

若参数中有多个路径，那么用于分隔多个路径的分隔符也会被转换为平台中的相关字符：在Windows操作系统中为分号“;”，在类UNIX操作系统中为冒号“:”。

### 8.3.4　目标相关表达式

本小节中的表达式均与构建目标相关，用于查询目标相关的属性等。需要注意的是，由于这类表达式较为特殊，与特定构建目标相关，有一些支持包含生成器表达式的命令参数可能并不支持这一类表达式。这一点通常在对应命令的官方文档中都会有所强调。

#### 目标名称：TARGET_NAME_IF_EXISTS

```
$<TARGET_NAME_IF_EXISTS:{目标}>
```

当指定{目标}存在时，该表达式解析后的值即{目标}的名称，否则其值为空。

#### 目标的二进制文件：TARGET_FILE

```
$<TARGET_FILE:{目标}>
```

该表达式解析后的值即{目标}构建后生成的二进制文件的绝对路径。

如果希望获取路径的不同组成部分，还可以使用下表中的这些生成器表达式。

| 表达式 | 描述 | 示例 |
| --- | --- | --- |
| `$<TARGET_FILE:{目标}>` | 绝对路径 | `/a/libbase.so` |
| `$<TARGET_FILE_BASE_NAME:{目标}>` | 基本名称 | `base` |
| `$<TARGET_FILE_PREFIX:{目标}>` | 前缀 | `lib` |
| `$<TARGET_FILE_SUFFIX:{目标}>` | 后缀 | `.so` |
| `$<TARGET_FILE_NAME:{目标}>` | 文件名称 | `libbase.so` |
| `$<TARGET_FILE_DIR:{目标}>` | 目录名称 | `/a` |

#### 目标的链接文件：TARGET_LINKER_FILE

```
$<TARGET_LINKER_FILE:{目标}>
```

该表达式解析后的值即链接{目标}时所需文件的绝对路径。一般来说，它和`TARGET_FILE`是一致的；但在Windows操作系统中，在链接动态链接库时，链接器链接的文件是其导入库文件，也就是说，Windows动态链接库的`TARGET_LINKER_FILE`是其.lib导入库的路径。

该表达式同样拥有类似于上表`#### 目标的二进制文件：TARGET_FILE`中的变种，读者可以根据需要选用。

#### 目标的带SONAME的文件：TARGET_SONAME_FILE

```
$<TARGET_SONAME_FILE:{目标}>
```

该表达式解析后的值即{目标}对应的带SONAME的文件（如扩展名为`.so.1`的文件）的绝对路径。{目标}参数必须是一个动态库目标的名称。

该表达式仅拥有下面两个变种以获取路径的不同部分，具体描述请参考表`#### 目标的二进制文件：TARGET_FILE`：

```
$<TARGET_SONAME_FILE_NAME:{目标}> # 文件名称
$<TARGET_SONAME_FILE_DIR:{目标}> # 目录名称
```

#### 目标的PDB文件：TARGET_PDB_FILE

```
$<TARGET_PDB_FILE:{目标}>
```

该表达式解析后的值即{目标}对应的`.pdb`符号文件的绝对路径。该表达式仅适用于MSVC编译器，因此往往在`if(MSVC)`的条件分支中使用。

该表达式同样拥有类似于表`#### 目标的二进制文件：TARGET_FILE`中的变种，读者可以根据需要选用。

#### 目标的BUNDLE目录：TARGET_BUNDLE_DIR

```
$<TARGET_BUNDLE_DIR:{目标}>
```

该表达式解析后的值即{目标}对应的Bundle目录的绝对路径。{目标}参数必须是一个Bundle目标（具有`MACOSX_BUNDLE`属性的可执行文件目标或具有`FRAMEWORK`或`BUNDLE`属性的库目标）的名称。

如果想获取目标Bundle内容中的目录路径，可以用下面这个表达式：

```
$<TARGET_BUNDLE_CONTENT_DIR:{目标}
```

#### 目标属性：TARGET_PROPERTY

```
$<TARGET_PROPERTY:{目标},{目标属性}>
$<TARGET_PROPERTY:{目标属性}>
```

这两个表达式可用于获取构建目标的{目标属性}，属性值会作为表达式最终的值。其中，第一个表达式可以指定{目标}参数，而第二个表达式则会根据所在上下文来确定用于获取属性的目标。如下所示的部分CMake目录程序中展示了这一点。

```
add_executable(main main.c)
set_target_properties(main PROPERTIES A 1)
target_compile_definitions(main PRIVATE 
    A=$<TARGET_PROPERTY:A> # 等价于 A=$<TARGET_PROPERTY:main,A>
)
```

有一点需要格外注意：如果使用要求中存在该生成器表达式，其被解析的上下文并非定义该使用要求的构建目标，而是它被传递到的构建目标，如下所示。如下所示中，最终主程序main中宏B的值是1而非2。这是因为lib的使用要求定义宏B为表达式`$<TARGET_PROPERTY:A>`的值，将被传递到构建目标main中，并在main的上下文中被解析成main的目标属性A的值，也就是1。

```
add_library(lib dummy.c)
set_target_properties(lib PROPERTIES A 2)
target_compile_definitions(lib PUBLIC
    B=$<TARGET_PROPERTY:A>
)
target_link_libraries(main PRIVATE lib)
```

#### 目标的目标文件：TARGET_OBJECTS

```
$<TARGET_OBJECTS:{目标}>
```

该表达式解析后的值即{目标}对应的目标文件的列表。{目标}参数必须是一个目标文件库目标的名称，具体应用参见7.1.3小节。

### 8.3.5　解析生成器表达式

本小节介绍的生成器表达式更为特殊，其值是将其参数作为生成器表达式解析后的结果值，其作用类似“元生成器表达式”。

#### 解析一般表达式：GENEX_EVAL

```
$<GENEX_EVAL:{生成器表达式}>
```

该表达式会将指定的{生成器表达式}参数在当前上下文中解析为具体的值，并作为该表达式的值。一般来说，这种用例通常出现在构建目标的自定义目标属性中，如下所示。

```
cmake_minimum_required(VERSION 3.20)

project(genex_eval)

add_executable(main main.c)

# 设置main的自定义目标属性CUSTOM_EXP的值
set_property(TARGET main PROPERTY CUSTOM_EXP "$<CONFIG>")

add_custom_target(debug-gen-exp ALL
    COMMAND ${CMAKE_COMMAND} -E echo # 输出Debug或Release等
    "$<GENEX_EVAL:$<TARGET_PROPERTY:main,CUSTOM_EXP>>" 
)
```

#### 解析目标相关表达式：TARGET_GENEX_EVAL

有时，需要在构建目标的上下文中解析生成器表达式。例如，要解析的{生成器表达式}中包含`$<TARGET_PROPERTY:{属性}>`表达式时，可以使用下面这个`TARGET_GENEX_EVAL`表达式。

```
$<TARGET_GENEX_EVAL:{构建目标},{生成器表达式}>
```

该表达式会将指定的{生成器表达式}参数在{构建目标}的上下文中解析为具体的值，并作为该表达式的值。如下所示CMake目录程序展示了该表达式的用法。

```
cmake_minimum_required(VERSION 3.20)

project(genex_eval)

add_executable(main main.c)
add_executable(main2 MACOSX_BUNDLE main.c)

set_property(TARGET main PROPERTY A "$<TARGET_PROPERTY:A>")
set_property(TARGET main2 PROPERTY A "main2.A")

add_custom_target(debug-gen-exp ALL
    COMMAND ${CMAKE_COMMAND} -E echo # 输出main2.A
    "$<TARGET_GENEX_EVAL:main2,$<TARGET_PROPERTY:main,A>>" 
)
```

## 8.4　小结

本章首先介绍了CMake中可以使用生成器表达式的常用命令，然后将CMake中常用的生成器表达式进行了分类讲解。

生成器表达式是初学者学习CMake时的一个难点，因为它们的语法和用途与CMake中其他的参数形式有很大的不同，不容易理解。其实，只需时刻牢记生成器表达式是在CMake的生成阶段解析的，很多问题就容易理解了。

由于CMake是构建系统的生成器，它自身不会独立进行程序的构建，所以CMake在生成阶段就不得不与目标构建系统有所耦合，很多信息也不得不等到生成阶段才能获得。笔者认为，CMake作为构建系统的生成器，有利有弊。一方面，它总能生成用户习惯使用的构建系统的项目文件，以便发挥原生构建系统的最大优势；另一方面，它也带来了不够完美的封装性，增加了目录程序编写的复杂度。

生成器表达式其实就是为了尽可能减少这一复杂度而生的，相当于对不同构建系统的差异性做了抽象封装。例如，不同平台或不同构建系统针对同一构建目标生成的文件名称可能有所不同，而`TARGET_FILE`表达式能将这一不同隐藏起来，让CMake程序更容易跨平台。

总而言之，生成器表达式是CMake的一个重要特性，能够让CMake程序更加简洁易读、易维护，读者可以在实际项目中多多尝试，体会这一特性的好处。第9章，我们将一起了解CMake中另一重要的特性——模块。
