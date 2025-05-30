# 第4章　常用命令

除了注释和空白符， CMake程序中就只剩下命令调用了。本章会为大家介绍CMake提供的常用命令，它们能够帮助我们在CMake程序中实现各种功能，包括对数值、字符串、列表、文件、路径的操作，生成文件、输出日志、执行命令行程序、引用模块、实现元编程、辅助调试等。

本章篇幅较长，建议用作参考，随时查阅，不必仔细通读。不过目录还是值得浏览一遍的，因为这样就可以知道CMake到底提供了哪些命令、这些命令能够完成怎样的任务。这样日后如有相关需求，便能及时想起查阅本章了。

本章内容相比CMake官方文档，提供了更加合理的结构设计和详实的例程，理解起来更轻松。不过，书中的内容终究是静态的，而官方文档是动态更新的，因此建议时常去官方文档逛一逛，也许能收获更多！

## 4.1　数值操作命令：math

由于CMake脚本语言几乎完全由命令构成，不存在数学表达式这种编程语言中常见的语法结构。那么如何在CMake中完成数值运算等操作呢？当然还是依靠命令。

CMake提供了math命令用于计算数学表达式。尽管这不如表达式语法那样简单直接，但鉴于构建过程中涉及数值计算的需求少之又少，使用math命令也算是在保持CMake语法单一性的前提下较为简单实用的一种方式了。math命令的参数构成如下：

```
math(EXPR <结果变量> "<表达式字符串>" [OUTPUT_FORMAT <格式选项>])
```

该命令会计算<表达式字符串>参数中的数学表达式的结果，并将计算结果按照一定格式存放到<结果变量>中，其结果格式可以通过可选参数<格式选项> 指定。它有以下两种取值：

- HEXADECIMAL，即采用类似C语言中的十六进制表示形式，以0x开头；

- DECIMAL，即十进制表示形式，是默认选项。

表达式支持的数值字面量包括十进制数值和十六进制数值，其中十六进制数值必须以0x开头。表达式支持的运算符如下所示，它们的计算方式与C语言中对应的运算符相同：

```
加：+
减：-
乘：*
除：/
求余：%
位或：|
位与：&
位异或：^
位取反：~
左移：<<
右移：>>
括号：(...)    
```

math命令要求表达式中的数值和计算结果都必须是一个64位有符号整数能够表示的。对于表达式中超出表示范围的数值，CMake会报错；对于计算结果超出表示范围的，CMake会在结果变量中存放溢出的结果。

如下所示例程中演示了有关输出格式的参数设置，以及刚刚提到的整型大小的问题。

```
math(EXPR a 10*10 OUTPUT_FORMAT DECIMAL) # a = 100
math(EXPR b "0x7FFFFFFFFFFFFFFF + 0x7FFFFFFFFFFFFFFF") # b = -2
math(EXPR c "16" OUTPUT_FORMAT HEXADECIMAL) # c = 0x10
math(EXPR d "~16" OUTPUT_FORMAT HEXADECIMAL) # d = 0xffffffffffffffef
# math(EXPR err "0xFFFFFFFFFFFFFFFF")
```

若将程序中第5行的注释取消，再次运行，CMake就会报告如下错误，提示数值超出了允许的范围：

```
CMake Error at 11.math.cmake:5 (math):
math cannot evaluate the expression: "0xFFFFFFFFFFFFFFFF": a numeric value
is out of range.
```

## 4.2　字符串操作命令：string

字符串操作命令，以及本章后面还会介绍的列表、文件操作命令等，都采用了类似一般编程语言中函数调用的形式：提供一个操作名称，再提供操作的参数，就可以获得一个结果。

本书在特指命令的某个具体操作时，会将同一个命令的不同操作名称对应的命令形式称为子命令，例如，string(FIND)子命令即string命令首个参数为FIND时对应的操作。

### 4.2.1　搜索和替换

#### 搜索字符串

```
string(FIND <字符串> <子字符串> <结果变量> [REVERSE])
```

该命令会在<字符串>中搜索<子字符串>，并将其第一次出现的位置存放到 <结果变量>中。当指定了REVERSE时，搜索从后向前进行，即搜索<子字符串>最后一次出现的位置。如果<子字符串>不存在，结果变量的值将为−1。如下所示中是一些实例。

```
string(FIND aba a res)
message("${res}") # 输出：0

string(FIND aba a res REVERSE)
message("${res}") # 输出：2

string(FIND aba c res)
message("${res}") # 输出：-1
```

该命令将字符串视为ASCII编码，搜索过程实际上是逐字节进行的，结果变量中的位置索引就是字节索引。因此，对多字节编码的字符串进行搜索可能无法得到正确的结果。

由于CMake程序文件需要采用兼容ASCII的编码保存，一般是UTF-8编码，而UTF-8编码是自同步（self-synchronizing）的，因此字符串搜索可正常实现。如果CMake程序文件采用GBK编码保存，字符串搜索就有可能出现问题。例如，string(FIND 狜@ res)中，由于“@”的GBK编码为“40”，“狜”的GBK编码为“AA40”，CMake在进行逐字节搜索时，就会将“@”当作“狜”的子字符串。因此结果变量res的值为1。总而言之，为了避免这类问题，请务必使用UTF-8编码保存CMake程序文件。

#### 替换字符串

```
string(REPLACE <匹配字符串> <替换字符串> 
       <结果变量> 
       <输入字符串> [<输入字符串>...])
```

该命令会将若干<输入字符串>连接在一起，将其中出现的所有<匹配字符串> 替换为<替换字符串>，并将最终结果存入<结果变量>。如下所示中是一些实例。

```
string(REPLACE a b res aba)
message("${res}") # 输出：bbb

string(REPLACE a b res abc cab)
message("${res}") # 输出：bbccbb
```

### 4.2.2　正则匹配和替换

#### 正则表达式语法

CMake所支持的正则表达式（regular expression）没有采用标准的语法，而是使用CMake自定义的一套简单语法，支持的特性并不丰富，但也足够日常使用。CMake正则表达式支持的语法结构如下表所示。

| 语法结构 | 描述 |
| --- | --- |
| `^` | 匹配输入的起始点 |
| `$` | 匹配输入的终止点 |
| `.` | 匹配任意单个字符 |
| `\<字符>` | 匹配指定的<字符>，一般用于转义正则中的特殊字符。 例如，`\.`可用于匹配`.`，`\\`可用于匹配`\`。其他非特殊字符也可以使用这种语法，但并无必要。例如，`\a`可用于匹配a，它等价于直接写a |
| `[ ]` | 匹配方括号中的任意某个字符 |
| `[^ ]` | 匹配任意某个不在方括号中的字符 |
| `-` | 在方括号中表示字符区间。 例如，`[a-c]`等价于`[abc]`。如果想在方括号中包含`“-”`字符本身用于匹配或不匹配它，应当将`“-”`放在方括号的开头或结尾，使其不可能表示一个区间 |
| `*` | 匹配其前面的模式零次或多次 |
| `+` | 匹配其前面的模式一次或多次 |
| `?` | 匹配其前面的模式零次或一次 |
| `|` | 匹配其两侧的模式之一 |
| `()` | 用于声明一个子表达式，可以在字符串的正则替换命令中引用。正则匹配后，可以通过变量`CMAKE_MATCH_<n>`获取各个子表达式的匹配结果，即捕获组 |

相比每一个模式之间的串联而言，`*`、`+`和`?`这些紧跟某个模式之后的符号具有更高的优先级，而`|`这个用于分隔模式的符号具有更低的优先级。如`^ab+c$`这个正则表达式中重复的模式是b而非ab，因此它可以匹配abbc，但不能匹配ababc；`^(ab|cd)$`这个正则表达式中二选一的两个模式分别是ab和cd，而非b和c，因此它可以匹配ab或cd，但不能匹配abd或acd。

`\<字符>`不同于CMake的转义字符。`\<字符>`是正则的一种模式，而转义字符是传递 CMake参数值时可能用到的语法。例如，通过下列程序定义一个匹配空白符的正则表达式：

```
set(regex "[ \t\n\r]")
```

其中的`\t`、`\n`和`\r`都是转义字符，实际的正则表达式中并没有`“\”`这个符号。正则表达式支持匹配这些转义后的字符，因此上述正则表达式可以匹配字符串中的各类空白符。

如果想匹配字符串中的反斜杠，有以下两种定义正则表达式的方式：

```
set(regex "[\\]")
set(regex "\\\\")
```

第一种方式实际的正则表达式为`[\]`，该方式通过匹配方括号中的字符来实现对反斜杠的匹配；第二种方式的正则表达式则是`\\`，则利用`\<字符>`模式来匹配反斜杠。

为了避免转义使用的反斜杠和正则模式使用的反斜杠互相混淆，可以使用括号参数传递正则表达式，这样就无须转义了。例如，定义一个匹配字符串`“(a+b)”`的正则表达式，有以下两种等价的写法：

```
set(regex [[\(\a\+\b\)]])
set(regex "\\(\\a\\+b\\)")
```

显而易见，使用括号参数可以更清晰地书写正则表达式。

#### 单次正则匹配

```
string(REGEX MATCH <正则表达式> 
       <结果变量> 
       <输入字符串> [<输入字符串>...])
```

该命令会将若干<输入字符串>连接在一起，在其中匹配指定的<正则表达式> 一次，并将最终匹配结果存入<结果变量>中。如下所示中是一些实例。

```
set(regex "[abc]+")

string(REGEX MATCH ${regex} res aaa)
message("${res}") # 输出：aaa

string(REGEX MATCH ${regex} res aaa bbb ccc abc)
message("${res}") # 输出：aaabbbcccabc

string(REGEX MATCH ${regex} res aaad)
message("${res}") # 输出：aaa

string(REGEX MATCH ${regex} res bcd aaa)
message("${res}") # 输出：bc

set(regex ^[abc]+$)

string(REGEX MATCH ${regex} res aaa)
message("${res}") # 输出：aaa

string(REGEX MATCH ${regex} res aaa bbb ccc abc)
message("${res}") # 输出：aaabbbcccabc

string(REGEX MATCH ${regex} res aaad)
message("${res}") # 输出空

string(REGEX MATCH ${regex} res bcd aaa)
message("${res}") # 输出空
```

#### 全部正则匹配

```
string(REGEX MATCHALL <正则表达式> 
       <结果变量> 
       <输入字符串> [<输入字符串>...])
```

该命令会将若干<输入字符串>连接在一起，在其中匹配指定的<正则表达式> 尽可能多的次数（即全部匹配），并将最终匹配结果以列表的形式存入<结果变量> 中。如下所示中是一个实例。

```
set(regex "([abc]+)")

string(REGEX MATCHALL ${regex} res adb dcd abcdcba)
message("${res}") # 输出：a;b;c;abc;cba
```

#### 正则替换

```
string(REGEX REPLACE <正则表达式>
       <替换表达式> <结果变量> 
       <输入字符串> [<输入字符串>...])
```

该命令会将若干<输入字符串>连接在一起，在其中匹配指定的<正则表达式> 尽可能多的次数，并将匹配的部分根据<替换表达式>进行替换，保存结果至<结果变量> 中。

<替换表达式>之所以被称为“表达式”，是因为可以引用正则匹配的子表达式。在正则表达式的语法中介绍过：括号用于声明一个子表达式。在<替换表达式>中，可以通过`\1`～`\9`分别引用第1个～第9个子表达式的匹配结果。另外，`\0`表示匹配到的完整字符串。如下所示中是一些实例。

```
# 将a、b、c组成的部分替换为-
string(REGEX REPLACE "[abc]+" "-" res adabc dccc)
message("${res}") # 输出：-d-d-

# 将由a、b、c组成的部分用括号括起
string(REGEX REPLACE "[abc]+" "(\\0)" res adabc dccc)
message("${res}") # 输出：(a)d(abc)d(ccc)

# 将长度至少为2的由a、b、c组成的部分的第1个字母用括号括起
string(REGEX REPLACE "([abc])([abc]+)" [[(\1)\2]] res adabcdccc)
message("${res}") # 输出：ad(a)bcd(c)cc
```

#### 捕获组变量

捕获组（capture group）即正则表达式中子表达式匹配的结果。捕获组可以在正则替换的替换表达式中通过反斜杠语法来访问。除此之外，捕获组还可以在正则匹配后通过CMake的预定义变量来访问。相关的预定义变量如下：

- `CMAKE_MATCH_COUNT`，其值为捕获组的数量；

- `CMAKE_MATCH_<n>`，其值为第n个捕获组的匹配内容。当n为0时，其值为正则匹配的完整结果而非子表达式匹配的结果。

这些变量会在正则匹配后被CMake赋值，这包括：

- 单次正则匹配后；

- 全部正则匹配后（对应最后一次正则匹配的捕获组）；

- 正则替换后（对应最后一次正则匹配的捕获组）；

- 字符串匹配条件判断（即if(<字符串变量> MATCHES <正则表达式>)）后。

如下所示中展示了一些实例。

```
function(print_matches)
    message("CMAKE_MATCH_COUNT: ${CMAKE_MATCH_COUNT}")
    message("CMAKE_MATCH_0: ${CMAKE_MATCH_0}")
    message("CMAKE_MATCH_1: ${CMAKE_MATCH_1}")
    message("CMAKE_MATCH_2: ${CMAKE_MATCH_2}")
    message("---")
endfunction()

set(regex "([abc])([abc]+)")

# 单次正则匹配
string(REGEX MATCH ${regex} res aaa d abc)
print_matches()
# 输出：
# CMAKE_MATCH_COUNT: 2
# CMAKE_MATCH_0: aaa
# CMAKE_MATCH_1: a
# CMAKE_MATCH_2: aa
# ---

# 全部正则匹配
string(REGEX MATCHALL ${regex} res aaa d abc)
print_matches()
# 输出：
# CMAKE_MATCH_COUNT: 2
# CMAKE_MATCH_0: abc
# CMAKE_MATCH_1: a
# CMAKE_MATCH_2: bc
# ---

# 正则替换
string(REGEX REPLACE ${regex} [[(\1)\2]] res aaa d abc)
print_matches()
# 输出：
# CMAKE_MATCH_COUNT: 2
# CMAKE_MATCH_0: abc
# CMAKE_MATCH_1: a
# CMAKE_MATCH_2: bc
# ---

# 字符串匹配条件判断
if("aaadabc" MATCHES ${regex})
    print_matches()
    # 输出：
    # CMAKE_MATCH_COUNT: 2
    # CMAKE_MATCH_0: aaa
    # CMAKE_MATCH_1: a
    # CMAKE_MATCH_2: aa
    # ---
endif()
```

### 4.2.3　取字符串长度

```
string(LENGTH <输入字符串> <结果变量>)
```

该命令将<输入字符串>的长度存入<结果变量>中。

### 4.2.4　字符串变换

#### 追加字符串

```
string(APPEND <字符串变量> [<输入字符串>...])
```

该命令将全部<输入字符串>依次追加到<字符串变量>末尾。

#### 前插字符串

```
string(PREPEND <字符串变量> [<输入字符串>...])
```

该命令会将<输入字符串>连接为一个整体，直接插入到<字符串变量>开头。

#### 连接字符串

```
string(CONCAT <结果变量> [<输入字符串>...])
```

该命令将全部<输入字符串>连接在一起，并存入<结果变量>中。如下所示中是一些实例。

```
set(res "原值")
string(CONCAT res a b c)
message("${res}") # 输出：abc
```

该命令与字符串追加命令不同——结果变量的原值不会保留。这也是为什么称这里的变量为“结果变量”，而非字符串追加命令中的“字符串变量”。本书中，凡是“结果变量”都会在命令调用后被重新赋值（覆盖原值）。

#### 分隔符连接

```
string(JOIN <分隔符> <结果变量> [<输入字符串>...])
```

该命令将全部<输入字符串>以指定的<分隔符>分隔并连接在一起，并存入<结果变量>。如下所示中是一些分隔符连接实例。

```
string(JOIN "-" res 2021 02 21)
message("${res}") # 输出：2021-02-21

string(JOIN "->" res A B C)
message("${res}") # 输出：A->B->C
```

#### 大小写转换

```
string(TOLOWER <输入字符串> <结果变量>) # 转小写
string(TOUPPER <输入字符串> <结果变量>) # 转大写
```

这两个命令分别将<输入字符串>中的字符转换为小写和大写形式。

#### 重复字符串

```
string(REPEAT <输入字符串> <重复次数> <结果变量>)
```

该命令将<输入字符串>重复指定的<重复次数>，并将结果存入<结果变量>。

#### 取子字符串

```
string(SUBSTRING <输入字符串> <起始位置> <截取长度> <结果变量>)
```

该命令将<输入字符串>从<起始位置>开始截取指定的<截取长度>，并将其存入<结果变量>。

当指定的<起始位置>与<截取长度>之和超过了<输入字符串>的长度，或<截取长度>为−1时，该命令会一直截取到<输入字符串>的结尾。另外，该命令对字符串进行字节粒度的截取，因此处理多字节编码字符时需特别注意。如下所示中是一些取子字符串实例。

```
string(SUBSTRING "abc" 1 1 res)
message(${res}) # 输出：b

string(SUBSTRING "abc" 1 -1 res)
message(${res}) # 输出：bc

string(SUBSTRING "abc" 1 10 res)
message(${res}) # 输出：bc

string(SUBSTRING "你好" 3 3 res)
message(${res}) # 输出：好
```

#### 删除首尾空白符

```
string(STRIP <输入字符串> <结果变量>)
```

该命令将<输入字符串>首尾的空白符删除，并将结果存入<结果变量>。

#### 删除生成器表达式

```
string(GENEX_STRIP <输入字符串> <结果变量>)
```

该命令会删除<输入字符串>中的生成器表达式，并将结果存入<结果变量>。生成器表达式是在CMake构建过程中才会涉及的一个概念，将在第8章中详细介绍。如下所示展示了一个例程。

```
string(GENEX_STRIP "a$<1:b;c>d" res)
message("${res}") # 输出：ad

string(GENEX_STRIP "a;$<1:b;c>;d;$<TARGET_OBJECTS:some_target>" res)
message("${res}") # 输出：a;d
```

该命令会将<输入字符串>当作一个列表字符串。如果删除字符串中的生成器表达式之后产生了空元素，那么它会将该空元素一并删除。也就是说，结果变量中不会出现相邻或在首尾的分号。

### 4.2.5　比较字符串

```
string(COMPARE LESS <字符串1> <字符串2> <结果变量>) # 小于
string(COMPARE GREATER <字符串1> <字符串2> <结果变量>) # 大于
string(COMPARE EQUAL <字符串1> <字符串2> <结果变量>) # 等于
string(COMPARE NOTEQUAL <字符串1> <字符串2> <结果变量>) # 不等于
string(COMPARE LESS_EQUAL <字符串1> <字符串2> <结果变量>) # 小于或等于
string(COMPARE GREATER_EQUAL <字符串1> <字符串2> <结果变量>) # 大于或等于
```

这些命令会将<字符串1>和<字符串2>按照字典序进行比较，如果满足命令指定的比较关系，则将<结果变量>赋值为1，否则赋值为0。如下所示中是一些实例。

```
string(COMPARE LESS a abc res)
message("${res}") # 输出：1

string(COMPARE GREATER a abc res)
message("${res}") # 输出：0
```

### 4.2.6　取哈希值

```
string(<哈希算法> <结果变量> <输入字符串>)
```

该命令将使用指定的<哈希算法>计算<输入字符串>的加密哈希（cryptographic hash，又称加密散列、密码散列等）值，并将结果存入<结果变量>中。该命令支持以下哈希算法：

- MD5，一种信息摘要（Message-Digest，MD）算法；

- SHA1，第一代安全散列算法（Secure Hash Algorithm，SHA）；

- SHA224，第二代安全散列算法，摘要长度为224位；

- SHA256，第二代安全散列算法，摘要长度为256位；

- SHA384，第二代安全散列算法，摘要长度为384位；

- SHA512，第二代安全散列算法，摘要长度为512位；

- SHA3_224，第三代安全散列算法，又称Keccak算法，摘要长度为224位；

- SHA3_256，第三代安全散列算法，又称Keccak算法，摘要长度为256位；

- SHA3_384，第三代安全散列算法，又称Keccak算法，摘要长度为384位；

- SHA3_512，第三代安全散列算法，又称Keccak算法，摘要长度为512位。

### 4.2.7　字符串生成

#### ASCII转字符串

```
string(ASCII <ASCII值> [<ASCII值>...] <结果变量>)
```

该命令将输入的多个`<ASCII值>`逐一转换为对应的字符并连接在一起，然后存入 <结果变量>中。事实上，CMake会将数值直接转换为对应的字节，因此转换并不局限于ASCII，UTF-8编码的中文字符也可以转换，如下所示。

```
string(ASCII 65 66 67 res)
message("${res}") # 输出：ABC

string(ASCII 228 189 160 res)
message("${res}") # 输出：你
```

#### 字符串转十六进制表示

```
string(HEX <输入字符串> <结果变量>)
```

该命令将<输入字符串>的每一个字符转换为编码字节的十六进制数值表示，并以字符串形式连接在一起存入<结果变量>。这种生成字符串的方法可以看作ASCII转字符串的逆操作，只不过数值表示采用的是十六进制。

另外，该命令生成的十六进制数值的文本形式均由数字和小写字母构成。如下所示中是一些实例。

```
string(HEX "ABC" res)
message("${res}") # 输出：414243

string(HEX "JKL" res)
message("${res}") # 输出：4a4b4c

string(HEX "你好" res)
message("${res}") # 输出：e4bda0e5a5bd
```

#### 生成C标识符

```
string(MAKE_C_IDENTIFIER <输入字符串> <结果变量>)
```

该命令将<输入字符串>中的非字母或数字的字符转换为下画线，并将结果存入 <结果变量>中。如果<输入字符串>的第一个字符为数字，则在结果前面插入一个下画线。

该命令可用于将一些名称转换成合法的C和`C++`标识符，对于代码生成等任务会有所帮助。如下所示中是一些实例。

```
string(MAKE_C_IDENTIFIER a123! res)
message("${res}") # 输出：a123_

string(MAKE_C_IDENTIFIER 123a! res)
message("${res}") # 输出：_123a_

string(MAKE_C_IDENTIFIER a哈b哈c res)
message("${res}") # 输出：a___b___c
```

实例中，标点符号“!”和中文字符“哈”都不是字母或数字，因此被转换为下画线。另外，因为中文字符在UTF-8编码中由三个字节构成，所以在这里被相应地转换为三个下画线。

#### 生成随机文本

```
string(RANDOM [LENGTH <长度>] [ALPHABET <字符集合>] 
       [RANDOM_SEED <随机种子>] <结果变量>)
```

该命令将生成指定<长度>的随机文本，且该文本由指定的<字符集合>中的字符构成。<长度>如果被省略，默认为5；<字符集合>如果被省略，默认为所有数字及大小写字母构成的集合。

如果<随机种子>被指定，随机数生成器将使用它作为种子。当随机种子相同时，随机数生成器不论被调用几次都会生成相同的结果。如下所示中是一些实例。

```
string(RANDOM res)
message("${res}")

string(RANDOM res)
message("${res}")

string(RANDOM LENGTH 10 ALPHABET 123abc RANDOM_SEED 1 res)
message("${res}")

string(RANDOM LENGTH 10 ALPHABET 123abc RANDOM_SEED 1 res)
message("${res}")
```

为了检验随机种子的效果，连续执行该CMake脚本程序两次，执行结果如下：

```
> cd CMake-Book/src/ch004/string
> cmake -P 生成随机文本.cmake
P5qpQ
sTvi4
1a2ba33cbb
1a2ba33cbb
> cmake -P 生成随机文本.cmake
jzMIh
yZLhe
1a2ba33cbb
1a2ba33cbb
```

#### 生成时间戳

```
string(TIMESTAMP <结果变量> [<时间戳格式>] [UTC])
```

该命令将把当前日期时间的组合以指定的<时间戳格式>或默认格式存入<结果变量>。该命令默认获取本地的当前日期时间；如果UTC可选参数被指定，则会获取当前的协调世界时（Coordinated Universal Time，UTC）。如果CMake无法获取时间戳，结果变量将被赋空值。

<时间戳格式>参数可以由描述符和其他字符组成，其中描述符如下表所示，它们会根据当前日期时间被替换为相应格式的值，而其他字符则会保持原样存入<结果变量>。

| 描述符 | 含义 |
| --- | --- |
| `%%` | 转义为百分号（%） |
| `%d` | 日（当月的日期，01～31） |
| `%H` | 24进制小时（00～23） |
| `%I` | 12进制小时（01～12） |
| `%j` | 当年的日数（001～366） |
| `%m` | 月份（当年的月数，01～12） |
| `%b` | 月份的英文缩写（例如，十月表示为Oct） |
| `%B` | 月份的英文（例如，十月表示为October） |
| `%M` | 分钟（00～59） |
| `%s` | UNIX时间（UTC 1970年1月1日0时0分0秒至今的总秒数） |
| `%S` | 秒数（60表示闰秒，00～60） |
| `%U` | 当年的周次（00～53） |
| `%w` | 星期（0为星期日，0～6） |
| `%a` | 星期的英文缩写（例如，周五表示为Fri） |
| `%A` | 星期的英文（例如，周五表示为Friday） |
| `%y` | 两位数年份（00～99） |
| `%Y` | 四位数年份 |

本地时间默认的时间戳格式为%Y-%m-%dT%H:%M:%S，例如：

```
2021-02-24T23:07:37    
```

UTC时间默认的时间戳格式为%Y-%m-%dT%H:%M:%SZ，例如：

```
2021-02-24T15:07:37Z
```

生成时间戳实例如下所示。

```
string(TIMESTAMP res "%Y/%m/%d %H时%M分%S秒")
message("${res}") # 输出：2021/02/24 23时15分26秒

string(TIMESTAMP res "%Y年%m月%d日 %H:%M:%S UTC" UTC)
message("${res}") # 输出：2021年02月24日 15:15:26 UTC
```

有时候，为了可重复的结果，需要让程序所获取的“当前日期时间”是一个固定值，而非总是在变化的真实值。此时，可以定义环境变量`SOURCE_DATE_EPOCH`的值为希望获取到的“当前日期时间”。该环境变量采用UNIX时间格式，代表UTC时间1970年1月1日0时0分0秒至今的总秒数。下面的执行过程演示了该环境变量的作用：

```
> cd CMake-Book/src/ch004/string
> $env:SOURCE_DATE_EPOCH=0
> cmake -P 生成时间戳.cmake
1970/01/01 08时00分00秒
1970年01月01日 00:00:00 UTC
```

该执行过程在PowerShell终端中完成，PowerShell定义环境变量的语法为`$env:<环境变量>=<值>`。Linux Shell定义环境变量的语法有所不同，使用export命令：`export SOURCE_DATE_EPOCH=0`。

#### 生成UUID

```
string(UUID <结果变量> NAMESPACE <命名空间UUID> NAME <名称>
       TYPE <MD5|SHA1> [UPPER])
```

该命令会根据RFC4122（UUID统一资源名称命名空间提案）来创建一个全局唯一标识符（Globally Unique IDentifier，GUID）。全局唯一标识符也被称为统一唯一标识符（Universally Unique IDentifier，UUID）。

该命令首先会将<命名空间UUID>与<名称>合并，用TYPE参数指定的哈希算法计算其哈希值，然后将哈希值转为UUID格式存入<结果变量>。UUID的格式如下：

```
xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

其中每一个“x”都代表一个十六进制字符。如果指定了UPPER，则结果中的字母会被转换为大写形式，否则均为小写形式，如下所示。

```
set(namespace d5c54b4a-2500-47f2-87a3-c35a5ca5abb6)

string(UUID res NAMESPACE ${namespace} NAME a TYPE MD5)
message("${res}") # 输出：97767c4e-ad2d-3ec2-a886-dff6f9fbeb86

string(UUID res NAMESPACE ${namespace} NAME b TYPE SHA1 UPPER)
message("${res}") # 输出：0460F7B4-BE40-5B00-B09A-C990B316B752
```

### 4.2.8　字符串模板

```
string(CONFIGURE <模板字符串> <结果变量> [@ONLY] [ESCAPE_QUOTES])
```

该命令将依照当前执行上下文中的变量对<模板字符串>进行配置，并将结果存入<结果变量>。

如下所示为一个模板定义例程，其中包含变量模板和宏定义模板。

```
set(template [=[
替换变量a: ${a}
替换变量b: @b@

定义宏C
#cmakedefine C

定义0/1宏D
#cmakedefine01 D

定义值为e的宏E
#cmakedefine E e

定义值为F变量的值的宏F
#cmakedefine F @F@
]=])
```

创建一个CMake脚本程序，定义模板中需要的变量，然后配置该模板字符串，并输出配置好的最终值，如下所示。

```
include(模板定义.cmake)

set(a "a的值")
set(b "b的值")
set(C "C的值")
set(D "D的值")
set(E "E的值")
set(F "F的值")

string(CONFIGURE ${template} res)
message("${res}")
```

其执行结果如下：

```
> cd CMake-Book/src/ch004/string/字符串模板
> cmake -P 配置.cmake
替换变量a: a的值
替换变量b: b的值
　
定义宏C
#define C
　
定义0/1宏D
#define D 1
　
定义值为e的宏E
#define E e
　
定义值为F变量的值的宏F
#define F F的值
```

下面分别讲解变量模板和宏定义模板的配置规则。

#### 变量模板

变量模板支持两种变量替换语法，一种与CMake变量引用语法一致，如`${<变量>}`；另一种则用一对`@`符号包裹变量名，如`@<变量>@`。CMake之所以引入第二种语法，是因为很多脚本程序（如Shell脚本）引用变量的语法也是`${<变量>}`，容易产生歧义。在配置模板的string命令中指定`@ONLY`参数，即可规定变量模板的语法只能使用第二种形式。如下所示例程在调用命令时就指定了`@ONLY`参数。

```
include(模板定义.cmake)

set(a "a的值")
set(b "b的值")
set(C "C的值")
set(D "D的值")
set(E "E的值")
set(F "F的值")

string(CONFIGURE ${template} res @ONLY)
message("${res}")
```

其执行结果如下：

```
> cd CMake-Book/src/ch004/string/字符串模板
> cmake -P @ONLY参数.cmake
替换变量a: ${a}
替换变量b: b的值
...
```

`“${a}”`在结果中仍然保持原样，因为指定`@ONLY`参数后，`${a}`将不会被替换。

#### 宏定义模板

CMake主要为构建服务，因此提供了专门生成宏定义的模板，可以非常方便地用于代码生成。例如，可以将定义在CMake程序中的版本号等信息以宏定义的形式生成到头文件中，并最终构建到可执行文件中。

宏定义模板主要分为两种形式：一种是值为0或1的宏定义，另一种是自定义值的宏定义。后一种形式的自定义值可为空值，用于表示存在性。这两种形式的语法分别如下：

```
#cmakedefine01 <变量>
#cmakedefine <变量> [<值>]
```

当<变量>的值为假值常量时，这两种模板形式会分别被替换为：

```
#define <变量> 0
/* #undef <变量> */
```

如下所示中是一个实例。

```
include(模板定义.cmake)

set(a "a的值")
set(b "b的值")

set(C "OFF")
set(D "FALSE")
set(E "0")
set(F "IGNORE")

string(CONFIGURE ${template} res)
message("${res}")
```

其执行结果如下：

```
> cd CMake-Book/src/ch004/string/字符串模板
> cmake -P 假值常量.cmake
...
　
定义宏C
/* #undef C */
　
定义0/1宏D
#define D 0
　
定义值为e的宏E
/* #undef E */
　
定义值为F变量的值的宏F
/* #undef F */
```

当变量值不是假值常量（真值常量或其他值）时，它们则会被替换为

```
#define <变量> 1
#define <变量> [<值>]
```

如下所示中是一个实例。

```
include(模板定义.cmake)

set(a "a的值")
set(b "b的值")

set(C "ON")
set(D "TRUE")
set(E "1")
set(F "YES")

string(CONFIGURE ${template} res)
message("${res}")
```

其执行结果如下：

```
> cd CMake-Book/src/ch004/string/字符串模板
> cmake -P 真值常量.cmake
...
　
定义宏C
#define C
　
定义0/1宏D
#define D 1
　
定义值为e的宏E
#define E e
　
定义值为F变量的值的宏F
#define F YES
```

#### 转义引号

在该string命令中，还有一个可选参数`ESCAPE_QUOTES`。它用于指定是否需要转义用于模板替换的变量值中的引号。若指定了该参数，CMake将会使用C语言中的转义形式来转义这些引号，即在引号前面加一个反斜杠。

下面的例程会演示`ESCAPE_QUOTES`参数对模板配置结果的影响。如下所示，变量a的值包含一对引号。

```
cmake_minimum_required(VERSION 3.20)

set(a "a的值，有\"引号\"")

string(CONFIGURE "std::string str = \"@a@\";" res)
message("${res}")

string(CONFIGURE "std::string str = \"@a@\";" res ESCAPE_QUOTES)
message("${res}")
```

其执行结果如下：

```
> cd CMake-Book/src/ch004/string/字符串模板
> cmake -P 转义引号.cmake
std::string str = "a的值，有"引号"";
std::string str = "a的值，有\"引号\"";
```

### 4.2.9　JSON操作

现在JSON文件格式越来越多地被应用于配置文件等场景。CMake在3.19版本中，也为string 命令增加了一些用于JSON操作的形式，为读写JSON提供了巨大的便利。

#### 错误处理

JSON操作中的每个命令都会涉及错误的处理，首先来了解其处理错误的方式。

这些用于JSON操作的命令都会涉及两个参数：<结果变量>和<错误变量>。

在命令正常执行的情况下，<结果变量>用于保存结果；而<错误变量>会被赋值为NOTFOUND，表示没有错误。

当命令执行出错时，<错误变量>用于存放错误信息，而<结果变量>则用于存放出错的 JSON键的路径，格式如下：

```
<JSON键>[-<JSON键>...]-NOTFOUND
```

当然，如果某些错误与某个路径的JSON键无关，则结果变量的值为NOTFOUND。另外，<错误变量>是可选参数。如果命令出错且<错误变量>未指定，则会触发CMake的致命错误，直接终止程序运行。

#### 获取JSON元素值

```
string(JSON <结果变量> [ERROR_VARIABLE <错误变量>]
       GET <JSON字符串> [<JSON键>...])
```

该命令将解析`<JSON字符串>`，并根据指定的若干`<JSON键>`定位到对应 JSON元素，获取其值，然后存入<结果变量>中。其中，`<JSON键>`可以是JSON对象的成员名称或JSON数组的索引数值。

如果定位到的JSON元素是一个JSON对象或数组，则获取的结果为对应元素的JSON字符串表示；如果其为布尔值元素，则获取的结果为ON或OFF；如果其为空元素“null”，则获取的结果为空字符串；其他情况下，获取的结果均为对应元素的字符串表示。如下所示中展示了一些实例。

```
set(json [=[{
    "a": {
        "b": ["x", "y", "z"],
        "c": true,
        "d": null
    }
}]=])

string(JSON res ERROR_VARIABLE err GET ${json} a b)
message("a.b=${res}") # 输出：a.b=[ "x", "y", "z" ]

string(JSON res ERROR_VARIABLE err GET ${json} a b 1)
message("a.b[1]=${res}") # 输出：a.b[1]=y

string(JSON res ERROR_VARIABLE err GET ${json} a c)
message("a.c=${res}") # 输出：a.c=ON

string(JSON res ERROR_VARIABLE err GET ${json} a d)
message("a.d=${res}") # 输出：a.d=

string(JSON res ERROR_VARIABLE err GET ${json} a e)
message("a.e=${res}, ${err}") 
# 输出：a.e=a-e-NOTFOUND, member 'a e' not found

string(JSON res GET ${json} a e) 
# 致命错误：string sub-command JSON member 'a e' not found.
```

#### 获取JSON元素类型

```
string(JSON <结果变量> [ERROR_VARIABLE <错误变量>]
       TYPE <JSON字符串> [<JSON键>...])
```

该命令将解析`<JSON字符串>`，并根据指定的若干`<JSON键>`定位到对应 JSON元素，将其类型存入<结果变量>中。结果类型有如下取值：

- NULL，空类型；

- NUMBER，数值类型；

- STRING，字符串类型；

- BOOLEAN，布尔类型；

- ARRAY，数组类型；

- OBJECT，对象类型。

如下所示例程展示了不同类型的输出结果。

```
set(json [=[{
    "a": {},
    "b": [],
    "c": 123,
    "d": "123",
    "e": false,
    "f": null
}]=])

string(JSON res TYPE ${json})
message("${res}") # 输出：OBJECT

foreach(key a b c d e f)
    string(JSON res TYPE ${json} ${key})
    message("${key}: ${res}")
endforeach()
# 输出：
# a: OBJECT
# b: ARRAY
# c: NUMBER
# d: STRING
# e: BOOLEAN
# f: NULL
```

#### 索引获取JSON键名

```
string(JSON <结果变量> [ERROR_VARIABLE <错误变量>]
       MEMBER <JSON字符串> [<JSON键>...] <N>)
```

该命令将解析`<JSON字符串>`，并根据指定的若干`<JSON键>`定位到对应 JSON元素，将其第`<N>`个键的名称存入<结果变量>中。该命令要求定位到的JSON元素必须是一个JSON对象类型的元素。如下所示中是一些实例。

```
set(json [=[{
    "a": {
        "b": 0,
        "c": {}
    },
    "d": {}
}]=])

string(JSON res MEMBER ${json} 0)
message("${res}") # 输出：a

string(JSON res MEMBER ${json} 1)
message("${res}") # 输出：d

string(JSON res MEMBER ${json} a 0)
message("${res}") # 输出：b

string(JSON res MEMBER ${json} a 1)
message("${res}") # 输出：c

string(JSON res MEMBER ${json} a b 0) 
# 致命错误：string sub-command JSON MEMBER needs to be called with an element of type OBJECT, got NUMBER.
```

#### 获取JSON元素大小

```
string(JSON <结果变量> [ERROR_VARIABLE <错误变量>]
       LENGTH <JSON字符串> [<JSON键>...])
```

该命令将解析`<JSON字符串>`，并根据指定的若干`<JSON键>`定位到对应 JSON元素，将其大小存入<结果变量>。该命令要求定位到的JSON元素必须是一个JSON对象类型或JSON数组类型的元素。如下所示中是一些实例。

```
set(json [=[{
    "a": {
        "x": 0,
        "y": {}
    },
    "b": [0, 1, 2, 3, 4],
    "c": "cmake"
}]=])

string(JSON res LENGTH ${json})
message("${res}") # 输出：3

string(JSON res LENGTH ${json} a)
message("${res}") # 输出：2

string(JSON res LENGTH ${json} b)
message("${res}") # 输出：5

string(JSON res LENGTH ${json} c) 
# 致命错误：string sub-command JSON LENGTH needs to be called with an element of type ARRAY or OBJECT, got STRING.
```

#### 移除JSON元素

```
string(JSON <结果变量> [ERROR_VARIABLE <错误变量>]
       REMOVE <JSON字符串> <JSON键> [<JSON键>...])
```

该命令将解析`<JSON字符串>`，并根据指定的若干`<JSON键>`定位到对应 JSON元素，然后将其移除，同时将移除元素后的JSON字符串存入<结果变量> 中。

#### 修改JSON元素值

```
string(JSON <结果变量> [ERROR_VARIABLE <错误变量>]
       SET <JSON字符串> <JSON键> [<JSON键>...] <新值>)
```

该命令将解析`<JSON字符串>`，并根据指定的若干`<JSON键>`定位到对应 JSON元素，若该元素不存在，则在定位位置创建该元素，然后将其值修改为指定的<新值>，并将修改后的JSON字符串存入<结果变量>中。其中，<新值>必须是一个合法的JSON字符串。也就是说，如果想设置一个元素的值为一个字符串，参数值需要加双引号。

#### 比较JSON对象

```
string(JSON <结果变量> [ERROR_VARIABLE <错误变量>]
       EQUAL <JSON字符串1> <JSON字符串2>)
```

该命令将比较指定的`<JSON字符串1>`和`<JSON字符串2>`对应的JSON对象，若二者等价，则将真值常量存入<结果变量>中；否则存入假值常量。如下所示中展示了两个实例。

```
string(JSON res EQUAL [[{"b": 0, "a": 1}]] [[{"a": 1, "b": 0}]])
message("${res}") # 输出：ON

string(JSON res EQUAL [[{"b": null, "a": 1}]] [[{"a": 1}]])
message("${res}") # 输出：OFF
```

## 4.3　列表操作命令：list

讲解列表之前再次强调：CMake中的一切变量值本质上都是字符串。

为了服务于各种目的，区分不同的类型以进行不同的操作是不可避免的。因此，CMake提供了很多将字符串视为某种类型并进行相关操作的命令，列表操作命令就是其中的典型。

### 4.3.1　回顾列表

先回顾一下列表的表示方式：由分号分隔的一系列字符串构成一个列表，分号隔开的每一个字符串都是列表的元素。列表可以直接通过set命令来创建，例如：

- set(list_1 a b c)创建了一个列表a;b;c，其中包含三个元素——a、b和c；

- set(list_2 "a;b;c")创建了与上例相同的列表；

- set(list_3 "a b c")创建了一个字符串"a b c"，或者说一个具有单个元素a b c的列表。

#### 列表操作的作用域

list命令对列表变量的创建或修改是在当前作用域完成的，这一点与set命令的默认行为是一致的。因此，如果要将结果修改到上层作用域的变量中，仍然需要调用set命令并指定PARENT_SCOPE参数，如下所示。

```
function(append list_var)
    list(APPEND ${list_var} d) # 向列表变量中追加元素d
    message("${${list_var}}") # 输出：a;b;c;d
endfunction()

function(append2 list_var)
    list(APPEND ${list_var} d) # 向列表变量中追加元素d
    set(${list_var} ${${list_var}} PARENT_SCOPE)
    message("${${list_var}}") # 输出：a;b;c;d
endfunction()

set(x "a;b;c")
append(x)
message("${x}") # 输出：a;b;c

append2(x)
message("${x}") # 输出：a;b;c;d
```

#### 列表索引的表示

很多列表操作免不了用到列表元素的索引，如获取、插入、删除元素等。CMake 列表操作命令要求索引从0开始计数，即0表示第一个元素的索引；同时还支持负数索引，可用于从后向前索引元素，如−1表示倒数第一个元素的索引，如下所示。

```
set(x "a;b;c")

foreach(i 0 1 2 -1 -2 -3)
    list(GET x ${i} res) # 获取列表变量x中的第i个元素到res
    message("x[${i}] = ${res}")
endforeach()
# 输出：
# x[0] = a
# x[1] = b
# x[2] = c
# x[-1] = c
# x[-2] = b
# x[-3] = a
```

### 4.3.2　访问列表元素

#### 索引获取元素

```
list(GET <列表变量> <元素索引> [<元素索引>...] <结果变量>)
```

该命令会根据若干<元素索引>参数获取列表中对应元素的值，并存入<结果变量>中。若指定多个<元素索引>，对应的元素值将以列表形式存入结果变量。如下所示中是一些实例。

```
set(x "a;b;c")

list(GET x 0 res)
message("${res}") # 输出：a

list(GET x 0 2 res)
message("${res}") # 输出：a;c
```

不同于string命令直接对字符串字面量进行操作，list命令接受的参数是列表变量的名称。

#### 搜索列表元素

```
list(FIND <列表变量> <搜索值> <结果变量>)
```

该命令会在列表中搜索指定的<搜索值>，并将搜索到的元素索引存入<结果变量>中。若该元素不存在，则结果为−1。

### 4.3.3　获取列表长度

```
list(LENGTH <列表变量> <结果变量>)
```

该命令将获取列表中元素的数量，即列表长度，并将其存入<结果变量>中。

### 4.3.4　列表元素增删

#### 追加元素

```
list(APPEND <列表变量> [<元素值>...])
```

该命令将指定的<元素值>追加到列表末尾。

#### 前插元素

```
list(PREPEND <列表变量> [<元素值>...])
```

该命令将指定的<元素值>插入到列表开头。

#### 插入元素

```
list(INSERT <列表变量> <插入索引> [<元素值>...])
```

该命令将指定的<元素值>插入到列表中<插入索引>所对应的位置，使得插入的第一个<元素值>对应的索引成为<插入索引>，如下所示。

```
set(x a;d;e)
list(INSERT x 1 b c)
message("${x}") # 输出：a;b;c;d;e
```

#### 弹出末项元素

```
list(POP_BACK <列表变量> [<结果变量>...])
```

若未指定<结果变量>，该命令将移除指定列表中的最后一个元素；若指定了若干<结果变量>，该命令将会移除指定列表中位于末尾的同样数量的元素，并依次赋值给对应的结果变量。如下所示中是一些实例。

```
set(x a b c d e)

list(POP_BACK x)
message("${x}") # 输出：a;b;c;d

list(POP_BACK x x0 x1)
message("${x}") # 输出：a;b
message("${x0}, ${x1}") # 输出：d, c
```

#### 弹出首项元素

```
list(POP_FRONT <列表变量> [<结果变量>...])
```

若未指定<结果变量>，该命令将移除指定列表中的第一个元素；若指定了若干<结果变量>，该命令将会移除指定列表中位于首部的同样数量的元素，并依次赋值给对应的结果变量。

#### 索引移除元素

```
list(REMOVE_AT <列表变量> <索引> [<索引>...])
```

该命令将移除列表中指定的若干<索引>所对应的元素。

#### 移除元素值

```
list(REMOVE_ITEM <列表变量> <值> [<值>...])
```

该命令将移除列表中所有值为指定<值>的元素。

#### 移除重复元素

```
list(REMOVE_DUPLICATES <列表变量>)
```

该命令将移除列表中所有重复的值。列表中重复的值将仅保留首次出现的一个，其他值则仍然存在于列表中，且保持原有顺序。

### 4.3.5　列表变换

#### 连接列表

```
list(JOIN <列表变量> <分隔字符串> <结果变量>)
```

该命令将以<分隔字符串>作为分隔符来连接列表中的每一个元素，并将结果存入<结果变量> 中。该命令与string(JOIN)命令的功能相同，但参数形式不同，读者可以根据实际情况选择更便利的形式。如下所示中是一些实例。

```
set(x a;b;c)
list(JOIN x "-" res)
message("${res}") # 输出：a-b-c

string(JOIN "-" res ${x})
message("${res}") # 输出：a-b-c
```

#### 取子列表

```
list(SUBLIST <列表变量> <起始索引> <截取长度> <结果变量>)
```

该命令将从<起始索引>开始截取列表指定<截取长度>数量的元素，作为结果的子列表，并将其存入<结果变量>中。

与string(SUBSTRING)取子字符串的命令类似，这里的<截取长度>也可以取值为−1，即一直截取到最后一个元素。同样，当<起始索引>与<截取长度>之和超过列表长度时，该命令也会一直截取到最后一个元素。

#### 列表筛选

```
list(FILTER <列表变量> <INCLUDE|EXCLUDE> REGEX <正则表达式>)
```

该命令可以在列表中筛选出匹配指定<正则表达式>的元素。当指定INCLUDE 时，该命令将仅保留匹配到的元素；当指定EXCLUDE时，则将匹配的元素移除。其中，正则表达式的语法与前面字符串正则匹配中的正则表达式一致，参见4.2.2小节。

在如下所示的例程中，第一个list(FILTER)命令将单字母的元素筛选出来，保存到了x中；第二个命令则将单字母或单字母跟着一个数字的形式筛选掉，将剩下的元素保存到了y中。

```
set(x a1 b2 c0 d e f 1 2 3)
set(y ${x})

list(FILTER x INCLUDE REGEX "^[a-z]$")
message("${x}") # 输出：d;e;f

list(FILTER y EXCLUDE REGEX "^[a-z][0-9]?$")
message("${y}") # 输出：1;2;3
```

### 4.3.6　列表重排

#### 翻转列表

```
list(REVERSE <列表变量>)
```

该命令可以将列表中的元素翻转顺序，即首尾对调。

#### 排序列表

```
list(SORT <列表变量> [COMPARE <排序依据>] [CASE <大小写敏感>] [ORDER <排序方向>])
```

该命令可以将列表中的元素根据指定的<排序依据>、<大小写敏感>、<排序方向>等选项排序。

其中，<排序依据>可以是下列选项之一。

- STRING，即按照字典序对列表元素进行排序，这是省略COMPARE参数时的默认值。

- FILE_BASENAME，即假定列表元素都是文件路径，并按照路径中的文件名称部分对列表元素进行排序。

- NATURAL，即按照自然序对列表元素进行排序。如果比较元素中含有非数字字符且它们不是这些元素的共同前缀，那么比较将退化为字典序比较。例如，a10比a2大，因为a是它们的共同前缀，于是按照自然序比较10和2；但a10比aa2小，因为aa并非它们的共同前缀，那么只能按照字典序对两个字符串进行比较。

<大小写敏感>可以是下列选项之一。

- SENSITIVE，即启用大小写敏感的比较，这是省略CASE参数时的默认值。

- INSENSITIVE，即禁用大小写敏感的比较。

<排序方向>可以是下列选项之一。

- ASCENDING，即升序排序，这是省略ORDER参数时的默认值。

- DESCENDING，即降序排序。

如下所示中是一些实例。

```
set(x a;b;c;D;e)

# 字典序升序，大小写敏感
list(SORT x)
message("${x}") # 输出：D;a;b;c;e

# 字典序升序，大小写不敏感
list(SORT x CASE INSENSITIVE) 
message("${x}") # 输出：a;b;c;D;e

set(x 1 10 2 3 100 4 5)

# 字典序降序，大小写敏感
list(SORT x ORDER DESCENDING)
message("${x}") # 输出：5;4;3;2;100;10;1

# 自然序降序，大小写敏感
list(SORT x COMPARE NATURAL ORDER DESCENDING)
message("${x}") # 输出：100;10;5;4;3;2;1

# 自然序升序，大小写敏感
set(x a2 a10)
list(SORT x COMPARE NATURAL)
message("${x}") # 输出：a2;a10

# 自然序升序，大小写敏感
set(x aa2 a10)
list(SORT x COMPARE NATURAL)
message("${x}") # 输出：a10;aa2

# 文件名升序，大小写敏感
set(x "a/a/c.txt" "a/b.txt" "a/b.pdf")
list(SORT x COMPARE FILE_BASENAME)
message("${x}") # 输出：a/b.pdf;a/b.txt;a/a/c.txt
```

### 4.3.7　列表元素变换

```
list(TRANSFORM <列表变量> <变换操作> 
     [<元素选择器>] [OUTPUT_VARIABLE <结果变量>])
```

该命令会对<元素选择器>选中的元素进行指定的<变换操作>，并将变换后的列表存入<结果变量>中；若未指定<结果变量>，则会直接修改<列表变量>。

该命令不会改变列表元素的个数。当未提供<元素选择器>参数时，它会对所有列表元素进行变换操作。否则，它仅会对选择器选中的元素进行变换操作，并保持其他元素不变。

#### 变换操作

列表中的每一个元素实质上都是一个字符串，因此对元素的变换操作自然就是一系列对字符串的操作。list(TRANSFORM)命令支持的变换操作正是如此，它们大都与string命令的子命令对应，参见4.2.4小节。下面列举了列表元素变换支持的变换操作。

```
list(TRANSFORM <列表变量> <APPEND|PREPEND> <值> ...)
```

上述变换会对列表的元素值追加或前插字符串。

```
list(TRANSFORM <列表变量> <TOUPPER|TOLOWER> ...)
```

上述变换会将列表元素值转换为大小或小写字符。

```
list(TRANSFORM <列表变量> STRIP ...)
```

上述变换会将列表元素值中的首尾空白符删除。

```
list(TRANSFORM <列表变量> GENEX_STRIP ...)
```

上述变换会将列表元素值中的生成器表达式删除。有关生成器表达式的介绍，参见第8章。

```
list(TRANSFORM <列表变量> REPLACE <正则表达式> <替换表达式> ...)
```

上述变换会对列表元素值反复进行正则匹配，并将匹配到的部分进行正则替换。正则表达式的语法及替换的规则参见4.2.2小节。

#### 元素选择器

元素选择器用于在列表中选择需要被变换的元素，可以是下列三种元素选择器之一。

```
list(TRANSFORM ... AT <索引> [<索引>...]  ...)
```

上述选择器按索引选择元素。

```
list(TRANSFORM ... FOR <起始索引> <终止索引> [<索引步进>] ...)
```

上述选择器按区间选择元素。

```
list(TRANSFORM ... REGEX <正则表达式> ...)
```

上述选择器按正则表达式选择元素，只有匹配正则表达式的元素才会被选择。

#### 实例

列表元素变换实例如下所示。

```
set(x a b c d e)

# 前插和追加字符串
list(TRANSFORM x PREPEND " (")
list(TRANSFORM x APPEND ") ")
message("${x}") # 输出：(a) ; (b) ; (c) ; (d) ; (e) 

# 转换为大写字母，存入变量y
list(TRANSFORM x TOUPPER OUTPUT_VARIABLE y)
message("${y}") # 输出：(A) ; (B) ; (C) ; (D) ; (E)

# 对x[0], x[2]删除首尾空白符
list(TRANSFORM x STRIP AT 0 2)
message("${x}") # 输出：(a); (b) ;(c); (d) ; (e)

# 从x[1]到x[4]（间隔为2），替换单个字母为两个字母
list(TRANSFORM x REPLACE [a-z] "\\0\\0" FOR 1 4 2)
message("${x}") # 输出：(a); (bb) ;(c); (dd) ; (e)

# 对值存在连续两个字母的元素，追加!
list(TRANSFORM x APPEND "!" REGEX [a-z][a-z])
message("${x}") # 输出：(a); (bb) !;(c); (dd) !; (e)
```

## 4.4　文件操作命令：file

file命令用于需要访问文件系统的文件和路径操作。对于无须访问文件系统的路径字符串语法层面的一些操作，通常使用cmake_path命令来完成。

### 4.4.1　读取文件

#### 读取文件内容

```
file(READ <文件名> <结果变量>
     [OFFSET <偏移量>] [LIMIT <最大长度>] [HEX])
```

该命令会将<文件名>指定的文件内容读取到<结果变量>中。

它还支持指定<偏移量>，即读取文件内容的起始字节位置；以及<最大长度>，即读取到<结果变量>中的最大字节数。HEX参数一般用于读取二进制文件内容，指定 HEX后，数据将会被转换为十六进制（小写字母）表示。如下所示中是一些实例。

```
file(READ example.txt res)
message("${res}") # 输出：你好，CMake！

file(READ example.txt res LIMIT 6)
message("${res}") # 输出：你好

file(READ example.txt res OFFSET 9)
message("${res}") # 输出：CMake！

file(READ example.txt res OFFSET 9 HEX)
message("${res}") # 输出：434d616b65efbc81
```

实例中涉及的example.txt文本文件如下所示。

```
你好，CMake！
```

<偏移量>和<最大长度>均按照字节数计算，因此UTF-8编码的中文等特殊字符的长度并不能视为1。

#### 读取字符串列表

```
file(STRINGS <文件名> <结果变量>
     [LENGTH_MAXIMUM <最大长度>]
     [LENGTH_MINIMUM <最小长度>]
     [LIMIT_COUNT <最大字符串数>]
     [LIMIT_INPUT <最大输入字节数>]
     [LIMIT_OUTPUT <最大输出字节数>]
     [REGEX <正则表达式>]
     [NEWLINE_CONSUME]
     [NO_HEX_CONVERSION]
     [ENCODING <UTF-8|UTF-16LE|UTF-16BE|UTF-32LE|UTF-32BE>]
     )
```

该命令会将<文件名>指定的文件内容中的每一行字符串，以列表的形式读取到 <结果变量>中。下列参数用于设置提取过程的限制条件：

- LENGTH_MAXIMUM <最大长度>，用于限制提取的字符串的最大长度；

- LENGTH_MINIMUM <最小长度>，用于限制提取的字符串的最小长度；

- LIMIT_COUNT <最大字符串数>，用于限制提取的字符串元素的数量；

- LIMIT_INPUT <最大输入字节数>，用于限制读取文件的最大字节数；

- LIMIT_OUTPUT <最大输出字节数>，用于限制存入<结果变量>中的值的最大字节数；

- REGEX <正则表达式>，用于限制提取的字符串必须匹配该正则表达式。

`NEWLINE_CONSUME`参数用于忽略换行符。它会将换行符视为字符串内容而非多个字符串的分隔符。指定该参数后，该命令的功能与file(READ)基本一致。

`NO_HEX_CONVERSION`参数用于禁止在读取时自动将Intel Hex文件和Motorola SREC文件转换为二进制。

ENCODING <编码>用于指定读取文件的编码方式。目前仅支持UTF-8、 UTF-16LE、UTF-16BE、UTF-32LE和UTF-32BE五种编码方式。省略该参数时，该命令会参考文件的字节顺序标记（Byte Order Mark，BOM）判断文件编码。

如下所示中是一些实例。

```
file(STRINGS list.txt res)
message("${res}") # 输出：abc;;CMake;abc;123

file(STRINGS list.txt res ENCODING UTF-8)
message("${res}") # 输出：abc;你好;CMake;abc;123

file(STRINGS list.txt res ENCODING UTF-8 REGEX [a-z]+)
message("${res}") # 输出：abc;CMake;abc

file(STRINGS list.txt res NEWLINE_CONSUME ENCODING UTF-8)
message("${res}")
# 输出：
# abc
# 你好
# CMake
# abc
# 123
```

实例中涉及的list.txt如下所示。

```
abc
你好
CMake
abc
123
```

对比前两行的输出结果可知：当未指定编码方式时，CMake无法正确提取中文字符串；指定文本文件list.txt编码方式UTF-8后，CMake正确提取出了中文字符串。

#### 计算文件哈希

```
file(<哈希算法> <文件名> <结果变量>)
```

该命令会对<文件名>指定的文件内容按照指定的<哈希算法>计算哈希值，并将该值存入<结果变量>。

4.2.6小节中介绍过字符串取哈希值的子命令，它支持多种哈希算法。这里的计算文件哈希命令同样支持这些哈希算法。

#### 获取修改时间

```
file(TIMESTAMP <文件名> <结果变量> [<时间戳格式>] [UTC])
```

该命令将获取<文件名>所指定文件的修改时间戳，并将其存入<结果变量>。文件修改时间戳的格式可以通过<时间戳格式>参数指定，UTC参数用于指定是否采用协调时间时。有关两个参数的说明，参见4.2.7小节中对字符串生成时间戳子命令的相关介绍。如下所示中是一组实例。

```
file(TIMESTAMP example.txt res)
message("${res}") # 输出：2021-05-10T23:22:13

file(TIMESTAMP example.txt res "%Y年%m月%d日 %H:%M:%S UTC" UTC)
message("${res}") # 输出：2021年05月10日 15:22:13 UTC
```

### 4.4.2　获取运行时依赖

```
file(GET_RUNTIME_DEPENDENCIES
     [RESOLVED_DEPENDENCIES_VAR <已解析依赖的结果变量>]
     [UNRESOLVED_DEPENDENCIES_VAR <未解析依赖的结果变量>]
     [CONFLICTING_DEPENDENCIES_PREFIX <冲突依赖的变量前缀名>]
     [EXECUTABLES [<可执行文件>...]]
     [LIBRARIES [<动态库文件>...]]
     [MODULES [<模块文件>...]]
     [DIRECTORIES [<搜索目录>...]]
     [BUNDLE_EXECUTABLE <Bundle可执行文件>]
     [PRE_INCLUDE_REGEXES [<正则表达式>...]]
     [PRE_EXCLUDE_REGEXES [<正则表达式>...]]
     [POST_INCLUDE_REGEXES [<正则表达式>...]]
     [POST_EXCLUDE_REGEXES [<正则表达式>...]]
)
```

该命令用于递归获取指定文件的运行时依赖库。下面将依次介绍它的参数。

```
RESOLVED_DEPENDENCIES_VAR <已解析依赖的结果变量>
UNRESOLVED_DEPENDENCIES_VAR <未解析依赖的结果变量>
```

命令获取依赖后，会尝试解析这些依赖库文件的存放位置。成功解析的依赖库文件路径会被存入<已解析依赖的结果变量>中；未能解析到文件路径的依赖库文件名则被存入<未解析依赖的结果变量>。

```
CONFLICTING_DEPENDENCIES_PREFIX <冲突依赖的变量前缀名>
```

上述参数用于指定一个变量前缀，一些依赖冲突信息会被分别存入变量名以该前缀开头的变量中。

当两个不同的搜索目录中存在文件名相同的依赖库时，这个依赖会被认为是冲突依赖。所有冲突依赖的文件名将以列表形式存入`<冲突依赖的变量前缀名>_FILENAMES`变量中。对于每一个冲突的依赖库文件，还会有一个变量`<冲突依赖的变量前缀名>_<冲突依赖文件名>`用以保存所有存在该文件名的目录路径。

例如，如果希望同时获取a.exe和b.exe的依赖，而它们分别依赖位于目录`dir_a`和目录`dir_b`中的同名库文件liba.dll。假设设置`CONFLICTING_DEPENDENCIES_PREFIX`为`X`，则变量`X_FILENAMES`的值为liba.dll，变量`X_liba.dll`的值为`dir_a;dir_b`。

若省略这个参数，那么冲突依赖会导致致命报错。

```
EXECUTABLES [<可执行文件>...]
LIBRARIES [<库文件>...]
MODULES [<模块文件>...]
```

上述参数分别用于指定需要获取其依赖的若干可执行文件、动态库文件和模块文件。

在macOS中，<可执行文件>的路径决定了递归解析依赖库时需要参照的`@executable_path`值。

```
DIRECTORIES [<搜索目录>...]
```

上述参数用于指定搜索依赖的若干目录。

在Linux中，该命令首先根据给定的要获取依赖的可执行文件、库文件等二进制文件中的编码信息，在默认搜索目录中解析依赖库文件。只有解析失败时，该命令才会考虑在指定的 <搜索目录>中搜索。如果依赖库文件确实在指定的<搜索目录>中解析到了，该命令会输出一条警告信息。因为通常来说，可执行文件、库文件等都会将依赖路径编码在自身二进制文件中（如RPATH），若无法根据默认搜索目录找到依赖，说明二进制中记录的信息很可能不完整。

在Windows中，该命令也会先在默认搜索目录解析依赖文件，再从指定的 <搜索目录>中搜索。但不同的是，它不会产生任何警告信息，这是因为在Windows中，从任意路径解析依赖库是非常常见的行为。

在macOS中，这个参数没有作用。

```
BUNDLE_EXECUTABLE <Bundle可执行文件>
```

上述参数仅在macOS中生效。它决定了递归解析<库文件>和<模块文件>的依赖库时所需参照的`@executable_path`值。该参数不会影响对<可执行文件>依赖库的解析。

```
PRE_INCLUDE_REGEXES [<正则表达式>...]
PRE_EXCLUDE_REGEXES [<正则表达式>...]
POST_INCLUDE_REGEXES [<正则表达式>...]
POST_EXCLUDE_REGEXES [<正则表达式>...]
```

这四个参数用于通过正则表达式来过滤需要解析的依赖库。

以PRE开头的参数用于过滤尚未解析的依赖库，以POST开头的参数用于过滤成功解析的依赖库。INCLUDE用于筛选需要包含的依赖库，EXCLUDE则用于筛选需要排除的依赖库。

在递归解析的过程中，每一次迭代都会过滤一次。具体的过滤流程如下。

1．若尚未解析的依赖匹配`PRE_INCLUDE_REGEXES`中的任意一个正则表达式，则继续解析该依赖，直接跳到步骤4。

2．若尚未解析的依赖匹配`PRE_EXCLUDE_REGEXES`中的任意一个正则表达式，则停止解析该依赖。

3．其他情况下，依赖的解析过程正常进行。该命令根据不同平台的链接规则搜索该依赖以解析其存放路径。

4．若搜索到该依赖，且其绝对路径能够匹配`POST_INCLUDE_REGEXES`中的任意一个正则表达式，或无法匹配`POST_EXCLUDE_REGEXES`开头的任意正则表达式，则将该绝对路径存入<已解析依赖的结果变量>对应的列表，并递归解析其依赖。否则，进行下一步判断。

5．若搜索到该依赖，且其绝对路径能够匹配`POST_EXCLUDE_REGEXES`中的任意一个正则表达式，则该绝对路径不会被存入结果变量中，并停止对该依赖的递归解析。

合理设置这些参数可以根据需要中断递归解析的过程，避免最终结果中产生大量不需要的依赖，如系统依赖等。

#### Windows中的依赖解析过程

由于Windows对文件名是大小写不敏感的，而链接器写入依赖库文件名时有可能混用大小写字母。这就为用于过滤库文件名的正则表达式参数带来了麻烦，例如，为了匹配各种可能的Kernel32.DLL，我们要将正则表达式写作`^[kK][eE][rR][nN][eE][lL]32\\.[dD][lL][lL]$`。

为了避免这个麻烦，该命令在Windows中解析依赖时，会先将依赖库的文件名都转换为小写形式。这样，使用正则表达式参数过滤依赖时，只考虑小写即可，如`^kernel32\\.dll$`。

需要注意：只有依赖库文件名会被转换为小写，其绝对路径中的其他目录部分不会被转换。具体结果过程如下。

1．将依赖库文件名转换为小写形式。

2．在目标文件的同一目录中解析依赖库文件。若该目录中不存在该库，继续下一步。

3．在操作系统的system32目录和Windows目录中依次解析该依赖库文件。若该目录中不存在该库，继续下一步。

4．在指定的<搜索目录>参数中，按照参数指定的目录顺序依次解析该依赖库文件。若仍不能解析到该库，将该库视为未能解析的依赖。

#### Linux中的依赖解析过程

1．若目标文件包含RUNPATH项，且依赖库存在于RUNPATH项指定的某一目录中，则成功解析该依赖库。

2．若依赖库存在于目标文件的RPATH项或其上级依赖者的RPATH项指定的某一目录中，则成功解析该依赖库。

3．若依赖库存在于ldconfig指定的目录中，则成功解析该依赖库。

4．若依赖库存在于<搜索目录>参数指定的某一目录中，则成功解析该依赖库，但同时该命令会产生警告信息，因为这通常意味着目标文件未完整列举依赖路径，很可能存在异常。

5．其他情况下，将该库视为未能解析的依赖。

#### macOS中的依赖解析过程

1．若依赖库路径以`@executable_path/`开头，且<可执行文件>参数中存在一个文件正被解析，且将`@executable_path/`替换为该可执行文件的路径后可以找到对应的依赖库文件，则成功解析该依赖库。

2．若依赖库路径以`@executable_path/`开头，且指定了`<Bundle可执行文件>`参数，且将`@executable_path/`替换为该参数的路径后可以找到对应的依赖库文件，则成功解析该依赖库。

3．若依赖库路径以`@loader_path/`开头，且将其替换为依赖该库的目标文件的所在目录路径后，可以找到对应的依赖库文件，则成功解析该依赖库。

4．若依赖库路径以`@rpath/`开头，且将其替换为依赖该库的目标文件的某一RPATH项后，可以找到对应的依赖库文件，则成功解析该依赖库。另外，RPATH项中若存在`@executable_path/`或`@loader_path/`，它们也会被替换为相应的值。

5．若依赖库路径是一个存在的绝对路径，则成功解析该依赖库。

6．其他情况下，将该库视为未能解析的依赖。

#### 实例

下面的实例将获取在第1章构建动态库时生成的主程序main.exe的运行时依赖，如下所示。

```
# 请先使用NMake生成 ch001/动态库 项目

set(dll_dir "${CMAKE_CURRENT_LIST_DIR}/../../ch001/动态库/")

file(GET_RUNTIME_DEPENDENCIES
    RESOLVED_DEPENDENCIES_VAR resolved
    UNRESOLVED_DEPENDENCIES_VAR unresolved
    EXECUTABLES ${dll_dir}/main.exe
    PRE_EXCLUDE_REGEXES kernel32.dll
)

message("${resolved}")
# 输出：C:/CMake-Book/src/ch004/file/../../ch001/动态库/liba.dll
message("${unresolved}")
# 输出空
```

该例程中过滤掉了kernel32.dll，避免输出大量系统库依赖。可以看到，输出结果中有一个已解析的依赖：liba.dll。因为它与main.exe 位于同一目录中，所以该命令可以成功解析到它所在的位置。

### 4.4.3　写入文件

#### 写文件内容

```
file([WRITE|APPEND] <文件名> <内容字符串>...)
```

该命令会将指定的<内容字符串>写入<文件名>指定的文件。若指定了多个 <内容字符串>参数，那么它们会被连接在一起作为最终的内容写入<文件名>指定的文件。

当<文件名>指定的文件已经存在时，WRITE写入模式会覆盖该文件内容，而APPEND写入模式则会在文件末尾追加内容。如果指定文件不存在，则两种模式都会先创建该文件。

#### Touch文件

```
file([TOUCH|TOUCH_NOCREATE] <文件名>...)
```

当<文件名>参数对应的文件存在时，该命令用于改变它们的访问时间或修改时间，而不改变文件内容。

当<文件名>参数对应的文件不存在时，使用TOUCH参数，该命令会创建指定<文件名>的空文件；而使用 TOUCH_NOCREATE参数，该命令则会被忽略。

### 4.4.4　模板文件

```
file(CONFIGURE OUTPUT <生成文件名>
     CONTENT <模板字符串>
     [ESCAPE_QUOTES] [@ONLY]
     [NEWLINE_STYLE UNIX|DOS|WIN32|LF|CRLF])
```

与4.2.8小节中介绍的字符串模板功能类似，该命令同样依照当前执行上下文中的变量对<模板字符串>进行配置，不同的是结果会存入<生成文件名>指定的文件中。模板规则及`ESCAPE_QUOTES`和`<@ONLY>`参数均可参考4.2.8小节。

`NEWLINE_STYLE`可选参数用于指定生成文件中换行符的格式：UNIX和LF指使用`\n`作为换行符；DOS、WIN32、CRLF指使用`\r\n`作为换行符。

如下所示中是一个实例。

```
cmake_minimum_required(VERSION 3.20)

file(CONFIGURE OUTPUT out3.txt
    CONTENT "CMAKE_VERSION: @CMAKE_VERSION@
#cmakedefine A"
    @ONLY
)
# out3.txt文件的最终内容为：
# CMAKE_VERSION: 3.20.2
# #define A
```

### 4.4.5　遍历路径

#### 遍历路径

```
file(GLOB <结果变量>
     [LIST_DIRECTORIES true|false]
     [RELATIVE <父路径>]
     [CONFIGURE_DEPENDS]
     [<路径遍历表达式>...]
)
```

该命令会根据指定的<路径遍历表达式>匹配相应的文件或目录，并将它们的路径按照字典序以列表形式存入<结果变量>中。在Windows和macOS中，它会将路径和该表达式都转换为小写形式再进行匹配，即不区分大小写，在其他平台中则会区分大小写。

<路径遍历表达式>是一种支持通配符的表达式，或者说是一种简化过的正则表达式：

- `*`直接用于匹配任意字符（除目录分隔符外）；

- `?`直接用于匹配单个字符（除目录分隔符外）；

- `[ ]`用于匹配方括号内的任意某个字符；

- `-`在方括号中表示字符区间，与正则表达式的语法一致。

指定<父路径>参数可以将结果变量中的路径转换为相对<父路径>的相对路径。

`LIST_DIRECTORIES`参数用于指定是否将匹配到的目录存入结果变量，默认为真值，即结果变量中既包含文件路径也包含目录路径。

`CONFIGURE_DEPENDS`参数在CMake脚本程序中没有作用，主要用于构建项目的CMake程序。指定该参数后，CMake会在每次构建时重新执行该file(GLOB)命令，检查输出结果是否发生改变。如果结果有所改变，则重新生成构建系统。

如下所示中是一个实例。

```
# 目录结构如下：
# 遍历路径
# |-- 1.csv
# |-- 1.jpg
# |-- 1.txt
# |-- a
#     |-- 2.txt
#     |-- b
#         |-- 3.txt

file(GLOB res 
    RELATIVE "${CMAKE_CURRENT_LIST_DIR}/遍历路径" 
    遍历路径/*)
message("${res}") # 输出：1.csv;1.jpg;1.txt;a

file(GLOB res 
    RELATIVE "${CMAKE_CURRENT_LIST_DIR}/遍历路径" 
    遍历路径/*.txt 遍历路径/*.jpg)
message("${res}") # 输出：1.jpg;1.txt

file(GLOB res 
    LIST_DIRECTORIES false
    RELATIVE "${CMAKE_CURRENT_LIST_DIR}/遍历路径" 
    遍历路径/*)
message("${res}") # 输出：1.csv;1.jpg;1.txt

file(GLOB res 
    LIST_DIRECTORIES false
    RELATIVE "${CMAKE_CURRENT_LIST_DIR}/遍历路径" 
    遍历路径/*/*)
message("${res}") # 输出：a/2.txt
```

#### 递归遍历路径

```
file(GLOB_RECURSE <结果变量>
     [FOLLOW_SYMLINKS]
     [LIST_DIRECTORIES true|false]
     [RELATIVE <父路径>]
     [CONFIGURE_DEPENDS]
     [<路径遍历表达式>...]
)
```

该命令与遍历路径的功能大体一致，只不过多了对递归遍历的支持，以及一个`FOLLOW_ SYMLINKS`参数。该命令会遍历所有匹配到的目录及其子目录中的路径。若某子目录是符号链接（symbolic link），则仅当指定了`FOLLOW_SYMLINKS`参数时，才会对其进行遍历。

另外，与递归遍历命令不同，该命令的`LIST_DIRECTORIES`参数的默认取值为false。

如下所示中是一个实例。

```
# 目录结构如下：
# 遍历路径
# |-- 1.csv
# |-- 1.jpg
# |-- 1.txt
# |-- a
#     |-- 2.txt
#     |-- b
#         |-- 3.txt

file(GLOB_RECURSE res
    RELATIVE "${CMAKE_CURRENT_LIST_DIR}/遍历路径" 
    遍历路径/*)
message("${res}") # 输出：1.csv;1.jpg;1.txt;a/2.txt;a/b/3.txt

file(GLOB_RECURSE res
    RELATIVE "${CMAKE_CURRENT_LIST_DIR}/遍历路径" 
    遍历路径/*.txt)
message("${res}") # 输出：1.txt;a/2.txt;a/b/3.txt

file(GLOB_RECURSE res
    RELATIVE "${CMAKE_CURRENT_LIST_DIR}/遍历路径" 
    遍历路径/*/*.txt)
message("${res}") # 输出：a/2.txt;a/b/3.txt
```

观察上例中第三个file命令的结果，`“遍历路径/*/*.txt”`中间的`*`必须匹配到某个目录路径，然后才会对这些目录及其子目录进行遍历。换句话说，这相当于把表达式拆成两部分：先用其目录部分`遍历路径/*`匹配目录，然后在这些目录及其子目录中，遍历文件名部分`*.txt`匹配到的文件。

递归遍历看上去有些类似Python glob库中支持的`**`模式，但实际上并不一样。Python glob库中的`**`可用于匹配目录中的任何文件，以及零个或多个层级的目录；CMake递归遍历中的`*`仍然只是匹配到当前层级的文件或目录路径，只不过会递归进入其中的子目录，然后按照<路径遍历表达式>的文件名部分再去匹配文件。

### 4.4.6　移动文件或目录

```
file(RENAME <原始路径> <目标路径>)
```

该命令可以将位于<原始路径>的一个文件或目录移动到<目标路径>，该命令要求<目标路径>的父目录必须存在。

### 4.4.7　删除文件或目录

#### 删除文件

```
file(REMOVE [<文件路径>...])
```

该命令用于删除若干<文件路径>对应的文件。<文件路径>可以是相对当前源文件目录的相对路径，当前源文件目录的定义参见6.3.1小节。另外，该命令会忽略不存在的文件路径，并不会提示警告或错误。

如果<文件路径>参数为空字符串，则该路径会被忽略，同时CMake产生警告信息。

#### 递归删除

```
file(REMOVE_RECURSE [<路径>...])
```

该命令用于递归删除若干<路径>对应的文件或目录，包括非空目录。

### 4.4.8　创建目录

```
file(MAKE_DIRECTORY [<目录路径>...])
```

该命令用于创建<目录路径>对应的目录及其父目录（若父目录不存在）。

### 4.4.9　复制文件或目录

```
file(<COPY|INSTALL> <路径>... DESTINATION <目标目录>
     [NO_SOURCE_PERMISSIONS | USE_SOURCE_PERMISSIONS]
     [FILE_PERMISSIONS <文件权限>...]
     [DIRECTORY_PERMISSION <目录权限>...]
     [FOLLOW_SYMLINK_CHAIN]
     [FILES_MATCHING]
     [
         [PATTERN <路径遍历表达式> | REGEX <正则表达式>]
         [EXCLUDE]
         [PERMISSIONS <权限>...]
     ]...
)
```

该命令会将<路径>对应的文件、目录和符号链接复制到<目标目录>中。文件被复制时，其时间戳会被保留。这样，如果复制时发现<目标目录>中存在同名文件且其时间戳也相同时，该命令就会跳过对该文件的复制。

COPY和INSTALL参数在功能上没有区别，只不过后者会额外输出一些状态信息，且默认隐式地指定了`NO_SOURCE_PERMISSIONS`参数，常用于CMake生成的安装脚本程序。

被复制的文件或目录的<路径>可为相对当前源文件目录的相对路径或绝对路径，<目标目录>可为相对当前构建目录的相对路径或绝对路径。对于CMake脚本程序而言，当前源文件目录和当前构建目录均为当前工作目录。这两个目录在构建过程中的具体定义参见6.3.1小节。

若干<路径遍历表达式>或<正则表达式>可用于匹配指定<路径>中的文件或目录，并对这些文件或目录进行细粒度的复制配置：

1．若指定`FILES_MATCHING`参数，则仅复制匹配到的文件或目录；

2．EXCLUDE参数的模式所匹配到的文件或目录将不被复制；

3．<权限>参数可用于对匹配到的文件或目录专门设定权限。

匹配<路径遍历表达式>时，仅匹配路径中的完整文件名；匹配<正则表达式>时，则会直接匹配绝对路径。后面的实例中可以观察到二者的区别。

<文件权限>、<目录权限>、<权限>都是用于设定权限的参数，其取值可为下表中的权限参数值的任意组合。

| 权限参数值 | 含义 |
| --- | --- |
| OWNER_READ | 所有者的读权限 |
| OWNER_WRITE | 所有者的写权限 |
| OWNER_EXECUTE | 所有者的执行权限 |
| GROUP_READ | 所在组的读权限 |
| GROUP_WRITE | 所在组的写权限 |
| GROUP_EXECUTE | 所在组的执行权限 |
| WORLD_READ | 任何人的读权限 |
| WORLD_WRITE | 任何人的写权限 |
| WORLD_EXECUTE | 任何人的执行权限 |
| SETUID | 切换身份为所有者执行程序的权限 |
| SETGID | 切换身份为所在组执行程序的权限 |

其中，部分权限参数值仅在受支持的平台中有效。在Windows中，除了设定所有者的权限外，其他参数值都会被忽略。

该命令对遍历到的每一个文件或目录进行权限设置的流程如下。

1．若当前正在处理的路径被某个模式匹配到，且该模式设定了<权限>参数，则使用该参数设定该文件或目录的权限，否则继续。

2．若当前正在处理某文件路径，且命令中指定了<文件权限>参数，则使用该参数设定其权限，否则继续。

3．若当前正在处理某目录路径，且命令中指定了<目录权限>参数，则使用该参数设定其权限，否则继续。

4．若指定了`NO_SOURCE_PERMISSIONS`，则使用默认权限设定当前处理的文件或目录，否则继续。

5．若指定了`USE_SOURCE_PERMISSIONS`，则保留被复制的文件或目录的原始权限，否则继续。

6．使用COPY参数时默认采用`USE_SOURCE_PERMISSIONS`参数行为，使用INSTALL参数时默认采用`NO_SOURCE_PERMISSIONS`参数行为。

其中，文件的默认权限为所有者的读、写权限，以及所在组和任何人的读权限，即：

```
OWNER_READ OWNER_WRITE GROUP_READ WORLD_READ
```

目录的默认权限在文件的默认权限的基础上增加了所有者、所在组和任何人的执行权限，即：

```
OWNER_READ OWNER_WRITE OWNER_EXECUTE 
GROUP_READ GROUP_EXECUTE 
WORLD_READ WORLD_EXECUTE
```

`FOLLOW_SYMLINK_CHAIN`参数用于递归解析<路径>中遇到的符号链接，直到找到其指向的真实文件，并将它们都复制到<目标目录>中。举个例子，假设有如下文件及其对应的符号链接：

```
/usr/local/lib/libA.so -> libA.so.1
/usr/local/lib/libA.so.1 -> libA.so.1.2 
/usr/local/lib/libA.so.1.2 -> libA.so.1.2.3
/usr/local/lib/libA.so.1.2.3
```

那么执行如下CMake命令时，该命令会将libA.so.1.2.3文件及其三个符号链接都复制到lib目录中，且复制后的符号链接都会指向lib目录中的libA.so.1.2.3文件。

```
file(COPY /usr/local/lib/libA.so DESTINATION lib 
     FOLLOW_SYMLINK_CHAIN)
```

最后再来看一个该命令的综合实例吧，如下所示。

```
file(COPY 复制文件 DESTINATION 复制目标/1
    FILES_MATCHING
    PATTERN *.txt
)

file(COPY 复制文件 DESTINATION 复制目标/2
    REGEX /[0-9]+.txt$ EXCLUDE
)

file(COPY 复制文件 DESTINATION 复制目标/3
    FILES_MATCHING
    PATTERN *.jpg
    REGEX /[0-9].txt$
)
```

其执行结果如下：

```
> cd CMake-Book/src/ch004/file
> cmake -P 复制文件或目录.cmake
> tree 复制目标
复制目标
|-- 1
|   |-- 复制文件
|       |-- 10.txt
|       |-- 1.txt
|-- 2
|   |-- 复制文件
|       |-- 2.csv
|       |-- 3.jpg
|-- 3
    |-- 复制文件
        |-- 1.txt
        |-- 3.jpg
```

其中，第一个命令是将复制文件目录复制到复制目标/1目录中，且要求只复制指定表达式匹配到的文件。由于参数中的表达式是一个路径遍历表达式`*.txt`，最终复制到目标目录中的文件就只有10.txt和1.txt了。

第二个命令是将目录复制到复制目标/2目录中，并仅指定了一个表达式用于排除复制的文件。该表达式参数是一个正则表达式`/[0-9]+.txt$`，因此最终复制的文件不包含 10.txt和1.txt。

正则表达式会匹配文件的绝对路径，因此为了匹配文件名部分，需要在最前面加一个路径分隔符`/`，在最后加一个`$`匹配终止点。

第三个命令主要是为了展示多个模式匹配参数的情形。该命令仅复制能够匹配到两个模式匹配表达式参数的文件，即1.txt和3.jpg。

#### 取文件大小

```
file(SIZE <文件名> <结果变量>)
```

该命令用于获取<文件名>指定的文件的大小（字节数），并将其值存入<结果变量>中。

#### 解析符号链接

```
file(READ_SYMLINK <符号链接路径> <结果变量>)
```

该命令用于获取<符号链接路径>指定的符号链接指向的路径，并将其存入<结果变量>中。若 <符号链接路径>对应的文件不存在，或不是一个符号链接，CMake会报告致命错误。

该命令会将获取的指向路径原样存入结果变量中，也就是说，如果某符号链接指向一个相对路径，该命令不会将其转换为绝对路径。

#### 创建链接

```
file(CREATE_LINK <源路径> <链接路径>
     [RESULT <结果变量>] [COPY_ON_ERROR] [SYMBOLIC])
```

该命令用于创建一个位于<链接路径>的指向<源路径>的硬链接或符号链接。默认情况下，它会创建一个硬链接；若指定了SYMBOLIC参数，它则会创建符号链接。硬链接要求<源路径>存在且指向一个文件。当<链接路径>已存在，该命令会覆盖该路径对应的文件。

当该命令执行成功时，会将<结果变量>的值设为0。否则，错误信息将存入<结果变量>。若<结果变量>可选参数并未指定，则该命令会在执行失败时报告致命错误。

指定`COPY_ON_ERROR`参数时，该命令会在创建链接失败时，复制<源路径>的文件到<链接文件名>对应的路径。该参数相当于提供了一个创建链接的备选方案。

#### 设置权限

```
file(CHMOD <文件或目录路径>...
     [PERMISSIONS <权限>...]
     [FILE_PERMISSIONS <文件权限>...]
     [DIRECTORY_PERMISSIONS <目录权限>...]
)
```

该命令用于修改<文件或目录路径>指定的若干文件或目录的权限。权限参数的取值参见上面`### 4.4.9　复制文件或目录`表。

<权限>参数用于指定文件和目录的权限，<文件权限>参数用于指定文件的权限， <目录权限>参数用于指定目录的权限，后两个参数相比<权限>参数具有更高的优先级。换句话说，如果同时指定了<权限>和<文件权限>参数，则所有文件的权限依照 <文件权限>参数来设定。<目录权限>参数同理。

另外，如果仅指定了<文件权限>，而未指定<权限>和<目录权限>，则该命令只会修改文件的权限，即使<文件或目录路径>参数中包含目录路径。仅指定<目录权限>参数同理。

同时指定全部三个权限参数是没有意义的，因为此时<权限>参数起不到任何效果。

#### 递归设置权限

```
file(CHMOD_RECURSIVE <文件或目录路径>...
     [PERMISSIONS <权限>...]
     [FILE_PERMISSIONS <文件权限>...]
     [DIRECTORY_PERMISSIONS <目录权限>...]
)
```

该命令用于递归修改<文件或目录路径>指定的若干文件或目录的权限。该命令与设置权限命令的不同之处在于递归，即对于参数中指定的目录路径，它会递归地将其全部子目录和文件的权限一并修改。

### 4.4.10　文件传输

```
file(DOWNLOAD <URL> [<文件路径>] [<下载选项>...])
file(UPLOAD <文件路径> <URL> [<上传选项>...])
```

这两个命令分别用于从指定的`<URL>`下载文件和将`<文件路径>`指定的文件上传到 `<URL>`。其中，下载文件的子命令可以省略`<文件路径>`参数，此时不会真正地下载文件，可用于检查文件是否能够正常下载。

#### 通用选项

下面首先介绍一下<下载选项>和<上传选项>通用的参数形式。

```
SHOW_PROGRESS
```

上述参数用于输出文件传输的进度信息。

```
STATUS <状态变量>
```

上述参数用于将文件传输操作的结果状态存入<状态变量>中。

状态是一个长度为2的列表，即由“;”分隔的两个元素：第一个元素是一个代表结果状态的数值，第二个元素则是结果状态的错误信息。结果状态数值为0时表示操作成功，没有错误。

```
LOG <日志变量>
```

上述参数用于将命令产生的文件传输日志存入<日志变量>中。

```
TIMEOUT <超时秒数>
```

指定上述参数后，当文件传输操作执行超过<超时秒数>时，该命令会终止文件传输。

```
INACTIVITY_TIMEOUT <无活动超时秒数>
```

指定上述参数后，当文件传输操作处于无活动状态超过<无活动超时秒数>时，该命令会终止文件传输。

```
HTTPHEADER <HTTP头信息>
...
```

上述参数用于指定文件传输时的HTTP头信息。该选项参数可以多次出现，用于设置多个HTTP头，例如：

```
file(DOWNLOAD "http://www.example.com/1.json" "res.json"
     HTTPHEADER "Host: www.example.com"
     HTTPHEADER "Content-Type: application/json"
)
USERPWD <用户名>:<密码>
```

上述参数用于指定文件传输时，HTTP Basic Auth所需的<用户名>和<密码>。

```
NETRC <IGNORED|OPTIONAL|REQUIRED>
```

上述参数用于指定文件传输时是否需要用于身份认证的`.netrc文件`。若省略该选项参数，默认值为 `CMAKE_NETRC`变量中的值。该参数的三个取值的含义如下：

- IGNORED代表忽略`.netrc文件`，这是默认行为；

- OPTIONAL代表`.netrc文件`非必需，文件传输时会优先考虑URL中的身份认证信息，利用`.netrc文件`补充URL中缺少的身份认证信息；

- REQUIRED代表`.netrc`是必需的，URL中的身份认证信息将被忽略。

```
NETRC_FILE <.netrc文件>
```

上述参数用于指定文件传输时额外使用的`<.netrc文件>`（作为Home目录中的`.netrc文件`的补充）。若省略该选项参数，其默认值为`CMAKE_NETRC_FILE`变量中的值。

```
TLS_VERIFY <ON|OFF>
```

上述参数用于指定是否对`https://`的URL验证服务器证书，其默认值为OFF。

```
TLS_CAINFO <CA文件>
```

上述参数用于指定用于验证服务器证书的自定义证书颁发机构（Certificate Authority，CA）文件。

若同时省略`TLS_VERIFY`和`TLS_CAINFO`，CMake会使用`CMAKE_TLS_VERIFY `变量和`CMAKE_TLS_CAINFO`变量的值作为这两个参数的默认值。

#### 额外的下载选项

这里有几个额外的<下载选项>参数，仅可用于文件下载的子命令。

```
EXPECTED_HASH <哈希算法>=<预期哈希值>
```

指定上述参数后，该命令会使用<哈希算法>计算下载的文件内容的哈希值，并将其与<预期哈希值>作比较，以验证下载文件的完整性和正确性。若哈希值不匹配，则验证失败，该命令会报告错误。该参数要求下载子命令中的<文件名>参数必须存在，否则该命令也会报告错误。

<哈希算法>的取值参见4.2.6小节中有关字符串取哈希值的介绍。

```
EXPECTED_MD5 <预期MD5值>
```

上述参数等价于`EXPECTED_HASH MD5=<预期MD5值>`。

#### 实例

如下所示中是一个关于文件下载的实例。

```
file(DOWNLOAD www.example.com download.txt STATUS status)
file(READ download.txt res LIMIT 64)
message("${status}") # 输出：0;"No error"
message("${res}")
# 输出：
# <!doctype html>
# <html>
# <head>
#     <title>Example Domain</title>

file(DOWNLOAD www.example.com/不存在.txt STATUS status)
message("${status}")
# 输出：22;"HTTP response code said error"
```

### 4.4.11　锁定文件

```
file(LOCK <文件或目录路径> [DIRECTORY] [RELEASE]
     [GUARD <FUNCTION|FILE|PROCESS>]
     [RESULT_VARIABLE <结果变量>]
     [TIMEOUT <超时秒数>]
)
```

该命令可用于为指定的<文件或目录路径>加锁。当为目录加锁时，必须指定 DIRECTORY参数，这实际上等同于为该目录下的cmake.lock文件加锁（若该文件不存在，先创建它）。

CMake为文件加的锁只是针对CMake程序而言的，并不能保证其他程序也会遵守，甚至CMake为目录加锁也不能阻止其他的锁再加到该目录的文件或子目录上。读者在使用时还需多加注意。

RELEASE参数用于显式释放锁。指定该参数后，GUARD和TIMEOUT参数没有任何意义，会被忽略。

GUARD参数用于指定锁的生效作用域。一旦程序在该作用域中执行结束，锁就会被释放。可以选择的作用域如下。

- PROCESS，在当前CMake进程中保持锁定。也就是说，其他CMake进程实例无法在该锁释放前，同时锁定同一路径。这是默认的作用域。

- FILE，在当前CMake程序文件中保持锁定。

- FUNCTION，在当前CMake函数中保持锁定。

<结果变量>用于存放结果状态，若成功则其值为0，否则其值为错误信息。如果省略该参数，则执行该命令遇到的任何错误，都会被视作致命错误，导致程序终止运行。

<超时秒数>用于指定该命令等待加锁操作完成的最长秒数。若设置该参数为0，则该命令仅尝试加锁一次，一旦失败就直接产生错误信息；若设置该参数为非零值，该命令会在超时之前一直尝试加锁，直到成功加锁或遇到其他致命错误。

### 4.4.12　归档压缩

#### 创建归档文件

```
file(ARCHIVE_CREATE OUTPUT <归档文件名>
     PATHS <被归档路径>...
     [FORMAT <归档文件格式>]
     [COMPRESSION <压缩算法>]
     [COMPRESSION_LEVEL <压缩级别>]
     [MTIME <文件修改时间>]
     [VERBOSE]
)
```

该命令将创建一个位于<归档文件名>的归档文件，其中包含全部<被归档路径>参数指定的文件或目录。

<归档文件格式>参数目前支持的取值包括：7zip、gnutar、pax、 paxr、raw和zip。指定其中的7zip和zip两种格式，相当于同时隐式地指定了压缩算法，因此无须再指定<压缩算法>参数。

对于其他的格式，可以指定<压缩算法>参数，其支持的取值包括：None、BZip2 、GZip、XZ和Zstd。其中，None表示不进行压缩，也是该参数的默认值。

<压缩级别>参数的取值为0～9，默认为0。该参数要求<压缩算法>参数必须被指定，且不为None。

<文件修改时间>参数用于指定归档文件内，被归档文件对应的修改时间。

指定VERBOSE参数，该命令会输出创建归档文件过程的详细信息。

#### 提取归档文件

```
file(ARCHIVE_EXTRACT INPUT <归档文件名>
     [DESTINATION <目标提取目录>]
     [LIST_ONLY]
     [PATTERNS <路径遍历表达式>...]
     [VERBOSE]
)
```

该命令将把<归档文件名>指定的归档文件中的全部内容，提取到指定的 <目标提取目录>中。若<目标提取目录>不存在，该命令会先创建该目录。若省略<目标提取目录>参数，当前构建目录会作为其默认值，在CMake脚本程序中，即当前工作目录。当前构建目录在构建过程中的定义参见6.3.1小节。

指定LIST_ONLY参数后，该命令不会真正提取归档中的文件，而仅会列举其中的文件。

<路径遍历表达式>用于筛选归档文件中需要被提取或列举的文件或目录路径。

这里的表达式仅支持使用通配符进行模式匹配，不支持方括号的模式。

指定VERBOSE参数，该命令会输出提取归档文件过程的详细信息。

### 4.4.13　生成文件

```
file(GENERATE OUTPUT <输出文件路径>
     INPUT <输入文件路径>|CONTENT <文件内容>
     [CONDITION <条件表达式>]
     [TARGET <构建目标>]
     [NO_SOURCE_PERMISSIONS | USE_SOURCE_PERMISSIONS | 
      FILE_PERMISSIONS <文件权限>...]
     [NEWLINE_STYLE UNIX|DOS|WIN32|LF|CRLF]
)
```

该命令在CMake脚本程序中没有效果，仅用于在CMake构建生成阶段中，对每一个构建模式生成位于<输出文件路径>的文件。

CMake在构建的生成阶段中，已经知道具体的生成器是什么，以及有哪些构建模式，因此它就能够借助这些信息生成对构建有帮助的文件内容。另外，该命令会解析输入文件内容中的生成器表达式。

读者第一次读到这里时可以先跳过下面的内容，后续遇到相关内容时再对照阅读。有关CMake构建的介绍，参见第6章；有关生成器表达式的介绍，参见第8章。

#### 参数

下面依次介绍该命令中的参数。

```
OUTPUT <输出文件路径>
```

上述参数用于指定<输出文件路径>，即生成文件的保存路径，路径可为相对于当前构建目录的相对路径。

如果对于不同的构建模式，生成的文件内容也有不同，则<输出文件路径>也必须不同。这可以通过`$<CONFIG>`等生成器表达式来实现。该参数支持使用生成器表达式来针对不同的构建模式指定文件名。

```
INPUT <输入文件路径>|CONTENT <文件内容>
```

上述参数的两种取值是互斥的，即应仅指定<输入文件路径>和<文件内容>参数之一。

若指定<输入文件路径>参数，则其对应文件的内容会作为生成文件的输入；若指定 <文件内容>参数，则该参数的字符串值直接作为生成文件的输入，内容可以包含生成器表达式。

<输入文件路径>可为相对于当前源文件目录的相对路径。

```
CONDITION <条件表达式>
```

上述参数指定的<条件表达式>用于确定是否要生成文件。<条件表达式>中可以包含生成器表达式，其被解析后的值应为0或1；当且仅当其解析后的值为1时，指定文件才会生成。

```
TARGET <构建目标>
```

上述参数用于指定解析生成器表达式时参考的<构建目标>。这是因为部分生成器表达式依赖某个构建目标的相关属性。

```
NO_SOURCE_PERMISSIONS | USE_SOURCE_PERMISSIONS
| FILE_PERMISSOINS <文件权限>...
```

上述参数的含义在介绍复制文件或目录的命令时已经提到过了，参见4.4.8小节。

<文件权限>参数用于自定义生成文件的权限，其取值参见`### 4.4.9　复制文件或目录`表。

```
NEWLINE_STYLE UNIX|DOS|WIN32|LF|CRLF]
```

上述参数在介绍模板文件时也已提过，参见4.4.4小节。

#### 实例

下面的实例会在CMake生成阶段中，使用该命令将库目标构建后的二进制文件名输出到指定文件内容中，如下所示。

```
cmake_minimum_required(VERSION 3.20)

project(file_generate)

add_library(lib lib.cpp)

file(GENERATE OUTPUT out-$<CONFIG>.txt
    CONTENT $<TARGET_FILE:lib>
    TARGET lib
)
```

为了演示该命令会针对不同的构建模式生成不同的文件，我们使用Visual Studio生成器来生成项目：

```
> cd CMake-Book/src/ch004/file/生成文件
> mkdir build
> cd build
> cmake -G "Visual Studio 16" ..
-- ...
-- Configuring done
-- Generating done
-- Build files have been written to: .../20.生成文件/build
```

项目生成完毕后，查看构建目录，其中确实生成了指定的文件：

```
> ls out-*.txt
out-Debug.txt
out-MinSizeRel.txt
out-Release.txt
out-RelWithDebInfo.txt
　
> cat out-Debug.txt
.../生成文件/build/Debug/lib.lib
　
> cat out-Release.txt
.../生成文件/build/Release/lib.lib
```

### 4.4.14　路径转换

file命令中用于路径转换的四个形式中，有三个已经不再推荐使用了，因此这里放到最后来介绍。 CMake 3.20版本加入了一个新的命令cmake_path，可以进行更多路径字符串相关的操作，也包含了这里介绍的三种操作。不过，为了便于读者理解历史遗留代码，本节还是会介绍这些子命令。

#### 计算绝对路径

```
file(REAL_PATH <路径> <结果变量> [BASE_DIRECTORY <父目录>])
```

该命令用于计算指定<路径>的绝对路径，若其对应文件为符号链接，则解析到链接目标的绝对路径。该命令也会将路径开头的`“~”`符号展开为用户目录。

<路径>可以是相对于特定<父目录>的相对路径。若省略<父目录>参数，则其默认为当前源文件目录。当前源文件目录的具体定义参见6.3.1小节，对于CMake脚本程序而言，即当前工作目录。

该命令未被废弃，因为该命令仍需访问文件系统以解析符号链接，所以它并不能被专用于路径字符串操作的 cmake_path命令替代。

#### 计算相对路径

```
file(RELATIVE_PATH <结果变量> <父目录> <文件绝对路径>)
```

该命令在CMake 3.20版本中不再推荐使用，被`cmake_path(RELATIVE_PATH)`子命令替代。

该命令用于计算<文件绝对路径>相对于<父目录>的相对路径，并将其存入<结果变量>中。

#### 转为CMake路径

```
file(TO_CMAKE_PATH <路径> <结果变量>)
```

该命令在CMake 3.20版本中不再推荐使用，被`cmake_path`命令的`CONVERT ... TO_CMAKE_PATH_LIST`子命令替代。

该命令可将指定<路径>转换为CMake路径格式，并存入<结果变量>中。

CMake路径格式统一采用斜杠作为目录分隔符，并使用CMake列表表示多个路径，即分号分隔的多个路径字符串。

在类UNIX操作系统中，多个路径通常使用冒号“:”来分隔，因此该命令也会将它们转换为分号，也就是使用CMake列表来表示多个路径。

#### 转为原生路径

```
file(TO_NATIVE_PATH <路径> <结果变量>)
```

该命令在CMake 3.20版本中不再推荐使用，被`cmake_path`命令的`CONVERT ... TO_NATIVE_PATH_LIST`子命令替代。

该命令可将指定<路径>转换为原生路径格式，并存入<结果变量>中。

对于Windows操作系统而言，原生路径格式采用反斜杠作为目录分隔符；对于类UNIX操作系统来说，原生路径格式则采用斜杠作为目录分隔符。

该命令转换后的原生路径格式统一使用CMake列表来表示多个路径，并不会针对类UNIX操作系统把CMake列表转换为冒号分隔的多个路径字符串。

## 4.5　路径操作命令：cmake_path

`cmake_path`命令用于对路径字符串进行操作。与file命令不同，`cmake_path`命令并不访问文件系统，仅在路径的字符串层面进行操作。

### 4.5.1　路径结构

路径字符串的结构如下：

```
根名 目录分隔符 (目录或文件名 目录分隔符)* 目录或文件名 目录分隔符?
```

- “根名”用于在多根文件系统中表示不同的根，如Windows操作系统中的磁盘分区`C:`，网络路径中的主机名`//server`等。该组成部分是可选的。

- “目录分隔符”即斜杠“/”。连续重复的目录分隔符会被当作一个目录分隔符，即`/a///b`等价于`/a/b`。

- “目录或文件名”由除目录分隔符以外的字符组成，可用于表示文件、符号链接文件、硬链接和目录。另外还有两种表示特殊含义的目录：“.”表示当前目录，“..”表示上级目录。

#### 绝对路径与相对路径

若某个路径中，除了根名以外，第一个组成部分是目录或文件名而非目录分隔符，则该路径为相对路径。否则，该路径为绝对路径。

一个路径存在根名，并不意味着它是一个绝对路径。例如，`C:a`的含义是C磁盘分区中的一个名为a的目录路径，而a目录未必位于磁盘分区的根目录中；`C:\a`才表示C磁盘分区根目录中的目录a的绝对路径。

#### 文件名结构

文件名由基本名称（stem）和扩展名（extension）构成。默认情况下，扩展名指从文件名中第一个圆点一直到最后的部分，包括圆点本身。例如，文件`a.b.c`的扩展名为`.b.c`。文件名的扩展名部分是可选的，扩展名之前的部分则是基本名称。

有一些`cmake_path`的子命令接受`LAST_ONLY`参数。指定该参数后将改变文件扩展名的提取范围：从文件名中最后一个小数点一直到最后的部分。例如，指定 `LAST_ONLY`参数后，该命令会认为文件`a.b.c`的扩展名为`“.c”`。

另外，若文件名以圆点开头，则该圆点会被忽略。例如，文件`“.abc”``“.”``“..”`均被认为是没有扩展名的。

### 4.5.2　创建路径变量

路径也是字符串，因此可以使用set命令创建一个值为路径的变量。不过，使用 `cmake_path`命令也可以创建路径变量，而且是更推荐的做法，因为它能够帮助将设置的路径转换为CMake路径格式。`cmake_path`的路径赋值和追加路径这两类子命令可用于创建路径变量。

#### 路径赋值

```
cmake_path(SET <结果变量> [NORMALIZE] <路径>)
```

该命令将把<路径>转换为CMake路径格式，并存入<结果变量>中。

NORMALIZE参数用于将路径正规化。其正规化流程如下。

1．若路径为空，直接返回空路径。

2．将连续重复的目录分隔符替换为单个目录分隔符。例如，`/a///b`正规化后为`/a/b`。

3．将冗余的当前目录符号及其后面的目录分隔符（即`./`）删除。例如，`./a/./b`正规化后为`a/b`。

4．将冗余的上级目录符号`“..”`及其前面的目录名、后面的目录分隔符`/`删除。例如，`a/../b`正规化后为`b`。

5．删除紧跟根名（若存在）后的上级目录符号`..`及其后面可能存在的路径分隔符`/`。例如，`/../a`正规化后为`/a`。

6．若路径最后以上级目录符号和目录分隔符（即`../`）结尾，删除最后的路径分隔符`/`。例如，`../`正规化后为`..`。

7．若经过上述步骤处理后路径成为了一个空路径，则为其添加一个圆点，即返回`“.”`作为最终结果。例如，`./`经过第3步处理后，就会成为空路径，因此它最终正规化后的路径为`“.”`。

如下所示中是一个实例。

```
cmake_path(SET x NORMALIZE "a/../b")
message("${x}") # 输出：b
```

#### 追加路径

```
cmake_path(APPEND <路径变量> <路径>...  [OUTPUT_VARIABLE <结果变量>])
cmake_path(APPEND_STRING <路径变量> <路径>...  [OUTPUT_VARIABLE <结果变量>])
```

这两个命令会将所有<路径>连接在一起，然后追加到<路径变量>末尾。不同的是，APPEND形式会将它们用目录分隔符连接在一起，而APPEND_STRING形式则会将它们直接拼接在一起，不添加目录分隔符。

若指定了<结果变量>，最终的追加结果会存入<结果变量>，而<路径变量>不会被修改。

如下所示中是一些实例。

```
cmake_path(APPEND res a b c)
message("${res}") # 输出：a/b/c

cmake_path(APPEND_STRING res a b c)
message("${res}") # 输出：a/b/cabc

cmake_path(APPEND res d e)
message("${res}") # 输出：a/b/cabc/d/e

if(WIN32)
    cmake_path(APPEND res a c:b c:c OUTPUT_VARIABLE res2)
else()
    cmake_path(APPEND res a /b c OUTPUT_VARIABLE res2)
endif()

message("${res2}")
# 在Windows操作系统中输出：c:b/c
# 在类UNIX操作系统中输出：/b/c
```

在追加过程中，若追加部分的路径是一个绝对路径，或追加部分的路径存在根名且根名与当前被追加的路径根名不同，则该追加部分的路径会直接替换掉当前被追加的路径。

### 4.5.3　分解路径结构

```
cmake_path(GET <路径变量> ROOT_NAME <结果变量>) 
cmake_path(GET <路径变量> ROOT_DIRECTORY <结果变量>) 
cmake_path(GET <路径变量> ROOT_PATH <结果变量>) 
cmake_path(GET <路径变量> FILENAME <结果变量>)
cmake_path(GET <路径变量> EXTENSION [LAST_ONLY] <结果变量>)
cmake_path(GET <路径变量> STEM [LAST_ONLY] <结果变量>)
cmake_path(GET <路径变量> RELATIVE_PART <结果变量>)
cmake_path(GET <路径变量> PARENT_PATH <结果变量>)
```

以上这些命令分别用于提取路径变量值中的不同组成部分，并将对应组成部分的值存入<结果变量>。关于路径组成部分的参数说明参见下表。

| 参数名 | 说明 | 示例 |
| --- | --- | --- |
| ROOT_NAME | 根名 | `C:` |
| ROOT_DIRECTORY | 根目录分隔符 | `/` |
| ROOT_PATH | 根路径（根名和根目录分隔符） | `C:/` |
| FILENAME | 文件名 | `b.txt` |
| EXTENSION | 扩展名 | `.txt` |
| STEM | 文件基本名称 | `b` |
| RELATIVE_PART | 相对路径部分 | `a/b.txt `|
| PARENT_PATH | 父目录 | `C:/a` |

若获取的组成部分在路径中不存在，则结果为空字符串。另外，根目录的父目录是根目录自身，如`C:/`的父目录还是`C:/`。

#### 实例

如下所示的实例中演示了一些比较特殊的情形，供读者仔细分析。

```
set(a "/")
cmake_path(GET a RELATIVE_PART res)
message("${res}") # 输出空
cmake_path(GET a PARENT_PATH res)
message("${res}") # 输出：/

set(b "1.2.3.txt")
cmake_path(GET b EXTENSION res)
message("${res}") # 输出：.2.3.txt
cmake_path(GET b EXTENSION LAST_ONLY res)
message("${res}") # 输出：.txt

set(c "a/b")
cmake_path(GET c FILENAME res)
message("${res}") # 输出：b

set(d "a/b/")
cmake_path(GET d FILENAME res)
message("${res}") # 输出空

set(e "/a/.x.txt")
cmake_path(GET e FILENAME res)
message("${res}") # 输出：.x.txt
cmake_path(GET e STEM res)
message("${res}") # 输出：.x
cmake_path(GET e EXTENSION res)
message("${res}") # 输出：.txt
```

### 4.5.4　路径判别

#### 判别结构存在性

```
cmake_path(HAS_ROOT_NAME <路径变量> <结果变量>) 
cmake_path(HAS_ROOT_DIRECTORY <路径变量> <结果变量>) 
cmake_path(HAS_ROOT_PATH <路径变量> <结果变量>) 
cmake_path(HAS_FILENAME <路径变量> <结果变量>) 
cmake_path(HAS_EXTENSION <路径变量> <结果变量>) 
cmake_path(HAS_STEM <路径变量> <结果变量>) 
cmake_path(HAS_RELATIVE_PART <路径变量> <结果变量>) 
cmake_path(HAS_PARENT_PATH <路径变量> <结果变量>) 
```

以上这些命令分别用于判别路径变量值中的不同组成部分是否存在。若对应结构存在，则该命令将结果变量赋值为真值常量；否则，赋值为假值常量。

其中，`HAS_ROOT_PATH`的判别结果仅在路径的根名或根目录分隔符不为空时为真。另外，对根目录的`HAS_PARENT_PATH`的判别结果也为真，因为根目录的父目录为其自身；当且仅当路径只由文件名构成时，`HAS_PARENT_PATH`的判别结果为假。如下所示的例程中展示了这些特殊情况。

```
set(a "/")
cmake_path(HAS_ROOT_PATH a res)
message("${res}") # 输出：ON
cmake_path(HAS_PARENT_PATH a res)
message("${res}") # 输出：ON

set(b "1.txt")
cmake_path(HAS_ROOT_PATH b res)
message("${res}") # 输出：OFF
cmake_path(HAS_PARENT_PATH b res)
message("${res}") # 输出：OFF
```

#### 判别路径类型

```
cmake_path(IS_ABSOLUTE <路径变量> <结果变量>) 
cmake_path(IS_RELATIVE <路径变量> <结果变量>) 
```

这两个命令用于判别路径类型，其中`IS_ABSOLUTE`用于判别路径是否为绝对路径， `IS_RELATIVE`用于判别路径是否为相对路径，二者的值正好相反。如下所示中是一些实例。

```
set(a "c:a")
cmake_path(IS_ABSOLUTE a res)
message("${res}") # 输出：OFF

set(b "c:/a")
cmake_path(IS_ABSOLUTE b res)
message("${res}") # 仅在Windows操作系统中输出：ON

set(c "/a")
cmake_path(IS_ABSOLUTE c res)
message("${res}") # 仅在类UNIX操作系统中输出：ON
```

在Windows操作系统中，当且仅当路径中同时存在根名和根目录分隔符时，它才被认为是一个绝对路径。因此，在Windows操作系统中，有些相对路径的`HAS_ROOT_DIRECTORY`也可能为真。在其他平台中，路径只要带有根目录分隔符即被视为绝对路径。

#### 判别路径前缀

```
cmake_path(IS_PREFIX <路径变量> <输入路径> [NORMALIZE] <结果变量>) 
```

该命令用于判别<路径变量>的路径值是否为<输入路径>的路径前缀。通常来说，路径任意层级的上级目录（父目录）均为路径的前缀，同时每个路径也是其自身的前缀。

指定NORMALIZE参数，该命令将首先正规化路径变量和输入路径，再进行前缀的判别。 如下所示中是一些实例。

```
set(path "/a/b/c")
cmake_path(IS_PREFIX path "/a/b/c/d" res0) # ON
cmake_path(IS_PREFIX path "/a/b/c" res1) # ON
cmake_path(IS_PREFIX path "/a/b" res2) # OFF
cmake_path(IS_PREFIX path "/a/b/cd" res3) # OFF
message("${res0} ${res1} ${res2} ${res3}") # 输出：ON ON OFF OFF

set(path "/a/b/..")
cmake_path(IS_PREFIX path "/a/b" res0) # OFF
cmake_path(IS_PREFIX path "/a/b" NORMALIZE res1) # ON
message("${res0} ${res1}") # 输出：OFF ON

set(path "/a/b/c")
cmake_path(IS_PREFIX path "/a/c/../b/c/d" res0) # OFF
cmake_path(IS_PREFIX path "/a/c/../b/c/d" NORMALIZE res1) # ON
message("${res0} ${res1}") # 输出：OFF ON
```

### 4.5.5　比较路径

```
cmake_path(COMPARE <输入路径1> <EQUAL|NOT_EQUAL> <输入路径2> <结果变量>) 
```

该命令用于比较两个路径字面量<输入路径1>和<输入路径2>，并将比较结果的真值或假值常量存入<结果变量>中。EQUAL表示判断两个路径是否相等，NOT_EQUAL表示判断两个路径是否不相等。

该命令在比较路径时，不会对路径进行正规化。它会按照下面描述的步骤进行相等判断：

1．若二者根名不同，则认为二者不相等；

2．若二者并非同时具备或同时不具备根目录分隔符，则认为二者不相等；

3．对其余路径结构部分一一比较，若有任何不相等的部分，则认为二者不相等，否则认为二者相等。

如下所示中是一些实例。

```
cmake_path(COMPARE "a\\b" EQUAL "a/b" res)
message("${res}") # 输出：ON

cmake_path(COMPARE "a/b" EQUAL "a/b/" res)
message("${res}") # 输出：OFF

cmake_path(COMPARE "a/b" NOT_EQUAL "a/b/c/.." res)
message("${res}") # 输出：ON
```

### 4.5.6　路径修改

#### 移除文件名

```
cmake_path(REMOVE_FILENAME <路径变量> [OUTPUT_VARIABLE <结果变量>])
```

该命令可将<路径变量>值中的文件名移除，若移除后路径末尾有一个目录分隔符，则仍然保留它，意味着路径中最后一部分的名称代表一个目录。

如果指定了<结果变量>参数，那么路径变量值不会发生改变，最终结果将被存入<结果变量>。

#### 替换文件名

```
cmake_path(REPLACE_FILENAME <路径变量> <新文件名> [OUTPUT_VARIABLE <结果变量>])
```

该命令可将<路径变量>值中的文件名替换为指定的<新文件名>。若路径变量值中不存在文件名，则保持其值不变。

如果指定了<结果变量>参数，那么路径变量值不会发生改变，最终结果将被存入<结果变量>。

#### 移除扩展名

```
cmake_path(REMOVE_EXTENSION <路径变量> [LAST_ONLY] 
           [OUTPUT_VARIABLE <结果变量>])
```

该命令可将<路径变量>值中的扩展名移除。

如果指定了<结果变量>参数，那么路径变量值不会发生改变，最终结果将被存入 <结果变量>。

其中LAST_ONLY参数的含义参见4.5.1小节中介绍文件名结构的部分。

#### 替换扩展名

```
cmake_path(REPLACE_EXTENSION <路径变量> [LAST_ONLY] <新扩展名> 
           [OUTPUT_VARIABLE <结果变量>])
```

该命令可将<路径变量>值中的扩展名替换为指定的<新扩展名>。若路径变量值中不存在扩展名，则为路径追加<新扩展名>。

如果指定了<结果变量>参数，那么路径变量值不会发生改变，最终结果将被存入<结果变量>。

### 4.5.7　路径转换

#### 正规化路径

```
cmake_path(NORMAL_PATH <路径变量> [OUTPUT_VARIABLE <结果变量>])
```

该命令将正规化<路径变量>的路径值。若指定<结果变量>参数，则不改变路径变量的值，而是将正规化后的结果存入<结果变量>中。正规化的步骤参见4.5.2小节。如下所示中是一个实例。

```
set(path "/a/../b/./c")
cmake_path(NORMAL_PATH path)
message("${path}") # 输出：/b/c
```

#### 计算绝对路径

```
cmake_path(ABSOLUTE_PATH <路径变量> [BASE_DIRECTORY <父路径>] [NORMALIZE]
           [OUTPUT_VARIABLE <结果变量>])
```

该命令将<路径变量>的路径值根据指定的<父路径>转换为绝对路径。若省略<父目录>参数，则其默认为当前源文件目录，对于CMake脚本程序而言，即当前工作目录。当前源文件目录的定义参见6.3.1小节。

若指定了NORMALIZE参数，则转换后的绝对路径会被正规化。

由于`cmake_path`命令不会访问文件系统，该命令不会解析符号链接，也不会展开路径开头的`“~”`符号。若需要对这两项功能的支持，请使用`file(REAL_PATH)`子命令。

#### 计算相对路径

```
cmake_path(RELATIVE_PATH <路径变量> [BASE_DIRECTORY <父路径>] [OUTPUT_VARIABLE <结果变量>])
```

该命令用于计算<路径变量>的路径值，并将其转换为相对于<父目录>的相对路径。若省略<父目录>参数，则其默认为当前源文件目录，对于CMake脚本程序而言，即当前工作目录。当前源文件目录的定义参见6.3.1小节。

#### 转为CMake路径

```
cmake_path(CONVERT <路径> TO_CMAKE_PATH_LIST <结果变量> [NORMALIZE])
```

该命令可将指定的<路径>转换为CMake路径格式，并存入<结果变量>中。若指定了NORMALIZE参数，则结果中的路径会被正规化。注意，<路径>参数是路径字面量而非变量。

CMake路径格式统一采用斜杠作为目录分隔符，并使用CMake列表表示多个路径，即分号分隔的多个路径字符串。在类UNIX操作系统中，多个路径通常使用冒号来分隔，因此该命令也会将它们转换为分号，也就是使用CMake列表来表示多个路径。

如下所示中是一个实例。

```
if(WIN32)
    cmake_path(CONVERT "a\\.\\1.txt;b/2.txt" TO_CMAKE_PATH_LIST res NORMALIZE)
else()
    cmake_path(CONVERT "a\\.\\1.txt:b/2.txt" TO_CMAKE_PATH_LIST res NORMALIZE)
endif()

message("${res}")
# 在Windows操作系统中输出：a/1.txt;b/2.txt
# 在类Unix操作系统中输出：a\.\1.txt;b/2.txt
```

在非Windows操作系统中，`cmake_path`不会把反斜杠转换为斜杠，这一点不同于`file(TO_CMAKE_PATH)`子命令。事实上在类UNIX操作系统中，文件名中是允许出现反斜杠的，因此`cmake_path`的行为更为合理。

#### 转为原生路径

```
cmake_path(NATIVE_PATH <路径变量> [NORMALIZE] <结果变量>)
cmake_path(CONVERT <路径> TO_NATIVE_PATH_LIST <结果变量> [NORMALIZE])
```

这两个命令可将<路径变量>的路径值或指定的<路径>转换为原生路径格式，并存入<结果变量>。若指定了NORMALIZE参数，则结果中的路径会被正规化。

对于Windows操作系统而言，原生路径格式采用反斜杠作为目录分隔符，分号作为多个路径的分隔符；对于类UNIX操作系统来说，原生路径格式则采用斜杠作为目录分隔符，冒号作为多个路径的分隔符。

如下所示中是一些实例。

```
set(path "a/1.txt;b/2.txt")
cmake_path(NATIVE_PATH path res)
message("${res}")
# 在Windows操作系统中输出：a\1.txt;b\2.txt
# 在类UNIX操作系统中输出：a/1.txt;b/2.txt

cmake_path(CONVERT "a/1.txt;b/2.txt" TO_NATIVE_PATH_LIST res)
message("${res}")
# 在Windows操作系统中输出：a\1.txt;b\2.txt
# 在类UNIX操作系统中输出：a/1.txt:b/2.txt
```

`cmake_path(NATIVE_PATH)`子命令类似于`file(TO_NATIVE_PATH)`，转换后的原生路径统一使用CMake列表来表示多个路径，并不会针对类UNIX操作系统把CMake列表转换为冒号分隔的多个路径字符串。而`cmake_path(CONVERT ... TO_NATIVE_PATH_LIST)`子命令则会做这个转换。

#### 计算路径哈希

```
cmake_path(HASH <路径变量> <结果变量>)
```

该命令将计算<路径变量>的路径值（注意是路径值，而非路径对应文件的内容）的哈希值，并存入<结果变量>。在计算哈希值前，路径值总是先被正规化，如下所示中的实例展示了这一点。

```
set(path ./a/)
cmake_path(HASH path res)
message("${res}") # 输出：c5a3bfd9b0e89d1d

set(path ./a/b/..)
cmake_path(HASH path res)
message("${res}") # 输出：c5a3bfd9b0e89d1d
```

## 4.6　路径操作命令：get_filename_component

`get_filename_component`命令也用于路径相关的操作，但其绝大多数功能已被 `cmake_path`命令取代，因此不再建议使用该命令。

不过，鉴于`cmake_path`命令是CMake 3.20版本才引入的，很多遗留下来的CMake 程序仍在使用`get_filename_component`命令。因此本节仍会对它进行介绍。同样，读者可以先跳过下面的内容。

### 4.6.1　分解路径结构

```
get_filename_component(<结果变量> <路径> <组成部分> 
                       [BASE_DIR <父目录>] [CACHE])
```

该命令在CMake 3.20版本中不再推荐使用，其大部分功能可使用`cmake_path`命令替代。 REALPATH组成部分的获取则可以使用`file(REAL_PATH)`命令替代。

该命令用于获取<路径>的特定<组成部分>（主要是与文件名相关的部分），并将其存入<结果变量>。

`get_filename_component`命令中<组成部分>参数的取值参见下表。

| 参数取值 | 说明 | 示例 |
| --- | --- | --- |
| DIRECTORY | 所在目录名 | `a` |
| NAME | 文件名 | `b.c.txt` |
| EXT | （最长的）扩展名 | `.c.txt` |
| NAME_WE | 文件基本名称（取最长的扩展名） | `b` |
| LAST_EXT | （最后的）扩展名 | `.txt` |
| NAME_WLE | 文件基本名称（取最后的扩展名） | `b.c` |
| ABSOLUTE | 绝对路径 | `C:/a/b.c.txt` |
| REALPATH | 绝对路径（解析符号链接） | `C:/a/b.c.txt` |

<父目录>参数仅用于ABSOLUTE和REALPATH这两种<组成部分>的参数取值，作为将路径转换为绝对路径的参考父目录。若省略<父目录>参数，则默认将当前源文件目录作为父目录。源文件目录的定义参见6.3.1小节，对于CMake脚本程序而言，它就是指当前工作目录。

CACHE可选参数用于决定是否将结果变量创建为缓存变量。

如下所示中是一些实例。

```
function(f mode)
    get_filename_component(res "a/b.c.txt" ${mode})
    message("${res}")
endfunction()

f(DIRECTORY) # 输出：a
f(NAME) # 输出：b.c.txt
f(EXT) # 输出：.c.txt
f(NAME_WE) # 输出：b
f(LAST_EXT) # 输出：.txt
f(NAME_WLE) # 输出：b.c

get_filename_component(res "a/b.c.txt" ABSOLUTE BASE_DIR C:/)
message("${res}") # 输出：C:/a/b.c.txt
```

### 4.6.2　解析命令行

```
get_filename_component(<结果变量> <命令行> PROGRAM 
                       [PROGRAM_ARGS <参数变量>] [CACHE])
```

该命令在CMake 3.20版本中不再推荐使用，已被`separate_arguments(PROGRAM)`命令替代。

该命令将在系统搜索目录中寻找<命令行>中指定的程序，并将其绝对路径存入<结果变量>。若指定了<参数变量>，则该命令会将 <命令行>中的参数部分存入<参数变量>。

CACHE可选参数用于决定是否将结果变量创建为缓存变量。

如下所示中是一个实例。

```
get_filename_component(cmd "notepad 1.txt 2.txt" 
                       PROGRAM PROGRAM_ARGS args)
message("${cmd}") # 输出：C:/Windows/System32/notepad.exe
message("${args}") # 输出： 1.txt 2.txt
```

## 4.7　配置模板文件：configure_file

`configure_file`命令用于配置模板文件。该命令会根据模板文件中定义的模板修改文件中的内容，并将结果输出到指定路径中。

该命令与4.2.8小节中介绍的string(CONFIGURE)字符串模板子命令，以及4.4.4小节中介绍的file (CONFIGURE)模板文件子命令功能相似。它们的主要不同在于模板的输入方式和结果的输出方式。

#### 基本用法

```
configure_file(<输入模板文件路径> <输出文件路径>
               [NO_SOURCE_PERMISSIONS | USE_SOURCE_PERMISSIONS |
               FILE_PERMISSIONS <权限>...]
               [COPYONLY] [ESCAPE_QUOTES] [@ONLY]
               [NEWLINE_STYLE [UNIX|DOS|WIN32|LF|CRLF] ])
```

<输入模板文件路径>可以是相对于当前源文件目录的相对路径或绝对路径， <输出文件路径>可以是相对于当前构建目录的相对路径或绝对路径。对于CMake脚本程序而言，当前源文件目录和当前构建目录均为当前工作目录。这两个目录在构建过程中的具体定义参见6.3.1小节。

不同于string(CONFIGURE)和file(CONFIGURE)子命令，`configure_file`命令所用到的模板是以文件形式输入的。因此该命令中增加了一些与文件相关的参数，如与权限相关的参数，以及COPYONLY参数。

有关权限设置的参数在4.4.8节中介绍file(COPY)子命令时已经介绍过，这里仅做简单回顾：

- `NO_SOURCE_PERMISSIONS`，指定输出文件不复制输入文件的权限，而是采用默认的文件权限，即所有者的读、写权限，以及所在组和任何人的读权限；

- `USE_SOURCE_PERMISSIONS`，指定输出文件使用输入文件的权限，这也是默认的行为，因此可以说指定该参数只是为了更清晰地表明行为语义；

- `FILE_PERMISSIONS`，自定义文件的权限，其取值参见`### 4.4.9　复制文件或目录`表。

`COPY_ONLY`参数表示仅复制，指定该参数后，`configure_file`命令将仅复制输入的模板文件到输出文件路径，不会对内容进行任何修改。由于文件内容不会改变，该参数不能与`NEWLINE_STYLE`同时指定。

其余几个参数均在file(CONFIGURE)子命令中出现过，此处仅做简单概括：

- `ESCAPE_QUOTES`，指定是否转义模板变量中的引号；

- `@ONLY`，指定是否仅替换用一对`“@”`符号包裹的模板变量名（形如`@<变量>@`）；

- `NEWLINE_STYLE`，指定输出文件的换行符格式。

#### 构建模式下的自动重配置

在构建模式下的CMake目录程序中使用`configure_file`命令时，若输入的模板文件发生改变，执行构建系统会自动触发该模板文件的重新配置和输出。

这一特性使得该命令非常适用于代码生成。例如，可以利用CMake检测系统环境配置，然后结合用户定义的变量等信息，通过包含宏定义等模板的输入文件生成头文件代码，实现针对不同环境配置、用户需求的条件编译。

#### 实例

该实例中包含`configure_file`命令的几种不同参数组合，可以观察其生成结果的差异。该实例的主程序如下所示。

```
set(a "a的值")
set(b "b的值")
set(C "C的值")
set(D "D的值")
set(E "E的值")
set(F "F的值")

configure_file(模板.h.in res1.h)
configure_file(模板.h.in res2.h @ONLY)
configure_file(模板.h.in res3.h COPYONLY)
```

其中用于输入的模板文件如下所示。

```
// 替换变量a: ${a}
// 替换变量b: @b@

// 定义宏C
#cmakedefine C

// 定义0 / 1宏D
#cmakedefine01 D

// 定义值为e的宏E
#cmakedefine E e

// 定义值为F变量的值的宏F
#cmakedefine F @F @
```

在终端中执行下列命令以运行该例程：

```
> cd CMake-Book/src/ch004/configure_file
> cmake -P configure_file.cmake
```

读者可以对比三个结果文件。

## 4.8　日志输出命令：message

在本节，我们会重新认识一下老朋友——message命令！它不仅可以将一句话简单地输出到屏幕上，还可以对输出的信息设置模式、格式等。另外，结合一些命令行参数，它还能实现对输出信息的筛选。

### 4.8.1　输出日志

```
message([<模式>] <日志消息字符串>...)
```

该命令用于在日志中输出指定的若干<日志消息字符串>，其中多个字符串会被连接在一起输出。<模式>参数用于指定日志的模式，不同的模式可能具有不同的日志级别、消息格式，也可能会对CMake程序执行构成不同的影响。

这里需要注意区分日志模式（log mode）和日志级别（log level）：日志模式是该子命令中<模式>参数的取值，而日志级别则是输出日志的重要程度，可用于筛选日志。日志级别按照从重要到不重要的顺序排列依次为错误（error）、警告（warning）、提示（notice）、状态（status）、详细信息（verbose）、调试信息（debug）、追踪信息（trace）。多个日志模式参数可能对应同一个日志级别。

接下来会按照上面列举的日志级别的顺序，对message命令的各个日志模式参数进行介绍。

#### 输出错误日志

```
message(FATAL_ERROR <日志消息字符串>...)
message(SEND_ERROR <日志消息字符串>...)
```

这两个具有不同<模式>参数的命令均用于输出错误级别的日志。这也印证了多个日志模式参数可能对应同一个日志级别这一点。

错误日志会被输出到标准错误输出（stderr）中。对于用于构建项目的CMake目录程序来说，错误会导致CMake跳过项目的生成阶段。`FATAL_ERROR`和`SEND_ERROR`二者的不同在于前者会导致CMake程序立即停止执行而后者不会。如下所示例程对比了二者的不同。

```
message(SEND_ERROR "错误1")
# CMake Error at 01.Error.cmake:1 (message):
#   错误1
message(FATAL_ERROR "错误2")
# CMake Error at 01.Error.cmake:2 (message):
#   错误2
message("这句消息不会被输出")
```

#### 输出警告日志

```
message(WARNING <日志消息字符串>...)
message(AUTHOR_WARNING <日志消息字符串>...)
```

这两个命令用于输出警告日志，警告日志会被输出到标准错误输出（stderr）中。

`AUTHOR_WARNING`模式一般用于面向该项目开发者的警告信息。那些只构建和引用当前项目的开发者无须关心通过`AUTHOR_WARNING`参数输出的警告信息。CMake命令行提供了一些与之相关的参数用于控制其行为：

- `-Wno-dev`，用于禁用面向开发者的警告信息的输出；

- `-Wdev`，用于启用面向开发者的警告信息的输出；

- `-Werror=dev`，用于将面向开发者的警告信息（包括弃用警告信息，见下文）视为错误；

- `-Wno-error=dev`，用于取消将面向开发者的警告信息（包括弃用警告信息）视为错误。

这些命令行参数仅用于CMake的构建模式，对于CMake执行脚本程序的`-P`命令行形式无效。

从未指定过上述参数时，CMake默认会输出面向开发者的警告信息，且不会将其视为错误，因此不会导致CMake跳过生成阶段。然而，上述这些参数一旦指定，便会以缓存变量的形式记录下来，后续再次调用CMake命令行时，即使不再指定上述参数，也仍然会依照缓存的参数设置决定输出的信息。这也是为什么每一个设置都会有启用和禁用两种形式。

为了展示不同命令行参数的效果，下面的实例会在CMake目录程序中输出警告日志，如下所示。

```
cmake_minimum_required(VERSION 3.20)

project(warning)

message(WARNING "一般警告")
message(AUTHOR_WARNING "开发警告")
message("程序运行到这里了")
```

如果读者还不熟悉CMake目录程序，可以先大概浏览，后续再回顾。

在默认情况下，CMake运行时只会报告警告，但不会报告错误，最终也会生成构建系统的配置文件。其执行结果如下：

```
> cd CMake-Book/src/ch004/message/Warning
> mkdir build
> cd build
> cmake ..
-- ...
　
CMake Warning at CMakeLists.txt:5 (message):
  一般警告
  
CMake Warning (dev) at CMakeLists.txt:6 (message):
开发警告
This warning is for project developers.  Use -Wno-dev to suppress it.
　
程序运行到这里了   
-- Configuring done
-- Generating done
-- ...
```

指定`-Wno-dev`命令行参数后，用于开发者的警告信息将被忽略，不再输出。其执行结果如下：

```
> cmake .. -Wno-dev
-- ...
　
CMake Warning at CMakeLists.txt:5 (message):
  一般警告
  
程序运行到这里了   
-- Configuring done
-- Generating done
```

指定`-Werror=dev`命令行参数后，面向开发者的警告将被视为错误。其执行结果如下：

```
> cmake .. -Werror=dev 
-- ...
　
CMake Warning at CMakeLists.txt:5 (message):
  一般警告
　
CMake Error (dev) at CMakeLists.txt:6 (message):
开发警告
This error is for project developers. Use -Wno-error=dev to suppress it.
　
-- Configuring incomplete, errors occurred!
```

#### 输出弃用日志

```
message(DEPRECATION <日志消息字符串>...)
```

该命令可用于提示用户某些功能或组件等已被弃用。弃用信息的输出受到以下两个CMake变量的控制。

- `CMAKE_WARN_DEPRECATED`变量决定是否将弃用信息以警告日志级别输出。若其值不为假值，则以警告日志级别输出。也就是说，若该变量未被设置，行为等同于将其设置为真值。另外，在CMake的构建模式中，`-Wdeprecated`命令行参数也可用于启用该选项，`-Wno-deprecated`命令行参数用于禁用该选项。

- `CMAKE_ERROR_DEPRECATED`变量决定是否将弃用信息以错误日志级别输出。若其值为真值，则以错误日志级别输出。也就是说，若该变量未被设置，行为等同于将其设置为假值。

如下所示中是一个实例。

```
message(DEPRECATION "A已弃用")
# CMake Deprecation Warning at 03.Deprecation.cmake:1 (message):
#   A已弃用
message(DEPRECATION "B已弃用")
# CMake Deprecation Warning at 03.Deprecation.cmake:2 (message):  
#   B已弃用
```

启用`CMAKE_ERROR_DEPRECATED`选项后，CMake会在遇到第一个弃用信息时终止　执行：

```
> cmake -DCMAKE_ERROR_DEPRECATED=TRUE -P Deprecation.cmake
CMake Deprecation Error at Deprecation.cmake:1 (message):
  A已弃用
```

#### 输出提示日志

```
message(<日志消息字符串>...)
message(NOTICE <日志消息字符串>...)
```

平时使用的不含<模式>参数的message命令输出的是提示（notice）级别的日志。提示日志会被输出到标准错误输出（stderr）中，一般用于需要吸引用户注意的重要提示信息。

#### 输出状态日志

```
message(STATUS <日志消息字符串>...)
```

状态日志应当是用户很可能感兴趣并想要查看的简短信息，常用于输出对环境配置等检查的状态。

状态日志会被输出到标准输出（stdout）中，输出时会带有由两个横线和空格组成的前缀。

#### 输出详细信息

```
message(VERBOSE <日志消息字符串>...)
```

详细信息用于提供用户在大多数情况下并不关心的更多信息。用户想要深入了解项目的构建过程时，才可能会查看详细信息。

详细信息在默认情况下不会输出，输出时会被输出到标准输出（stdout）中，且带有由两个横线和空格组成的前缀。

#### 输出调试信息

```
message(DEBUG <日志消息字符串>...)
```

调试信息一般用于项目开发者调试项目的构建过程。对于仅仅希望构建项目的用户而言，调试信息通常没有什么用处。

调试信息在默认情况下不会输出，输出时会被输出到标准输出（stdout）中，且带有由两个横线和空格组成的前缀。

#### 输出追踪信息

```
message(TRACE <日志消息字符串>...)
```

追踪信息是最细粒度的日志信息，涉及最底层的实现细节。一般来说，在开发过程中为了调试等目的也许会临时输出一些追踪信息。这些输出最好在项目发布前移除。

追踪信息在默认情况下不会输出，输出时会被输出到标准输出（stdout）中，且带有由两个横线和空格组成的前缀。

### 4.8.2　筛选日志级别

刚刚介绍了各种日志级别，其中后三种默认都不会输出。那么，如果要查看后三种日志级别的日志该怎么做呢？只关注错误和警告，甚至不想看到提示和状态信息时，又该怎么做呢？此时可以使用命令行参数`--log-level`对输出日志的级别进行筛选。筛选日志级别的例程如下所示。

```
message(TRACE "trace")
message(DEBUG "debug")
message(VERBOSE "verbose")
message(STATUS "status")
message(NOTICE "notice")
message(WARNING "warning")
message(SEND_ERROR "error")
```

在不带`--log-level`命令行参数时，其执行结果如下：

```
> cd CMake-Book/src/ch004/message
> cmake -P 筛选日志级别.cmake
-- status
notice
CMake Warning at 筛选日志级别.cmake:6 (message):
  warning
　
CMake Error at 筛选日志级别.cmake:7 (message):  
  error
```

下面，输出所有的日志级别：

```
> cmake --log-level=trace -P 筛选日志级别.cmake
-- trace
-- debug
-- verbose
-- status
notice
CMake Warning at 筛选日志级别.cmake:6 (message):
  warning
　
CMake Error at 筛选日志级别.cmake:7 (message):
  error
```

同样地，也可以只关注错误和警告：

```
> cmake --log-level=warning -P 筛选日志级别.cmake
CMake Warning at 筛选日志级别.cmake:6 (message):
  warning
　
CMake Error at 筛选日志级别.cmake:7 (message):
  error
```

除了使用`--log-level`命令行参数外，也可使用`CMAKE_MESSAGE_LOG_LEVEL` CMake变量筛选日志级别。将该变量设置为缓存变量可以使得后续执行CMake命令行时都采用设置的日志级别，而不必重复指定命令行参数。

### 4.8.3　输出检查状态

在CMake中，经常需要检查硬件配置、系统环境或依赖文件等，以确定设定的编译选项是否适配当前环境。通常检查过程和结果会被输出到日志中，用于帮助开发者确认环境配置，调试问题。 message命令提供了对输出这类日志的支持。

```
message([<检查状态>] <状态描述字符串>...)
```

该命令输出的日志的级别始终为状态（status），其中<检查状态>参数有以下取值：

- `CHECK_START`，用于输出开始检查的日志；

- `CHECK_PASS`，用于输出检查成功的日志；

- `CHECK_FAIL`，用于输出检查失败的日志。

准备开始某项检查时，应当使用`CHECK_START`参数输出检查的内容；检查结束后，根据检查是否成功，分别使用`CHECK_PASS`或`CHECK_FAIL`参数输出检查结果。

`CHECK_PASS`或`CHECK_FAIL`应当与`CHECK_START`成对使用，它输出的检查结果日志由`CHECK_START`的日志内容与检查结果共同组成。如下所示中是一个实例。

```
message(CHECK_START "寻找依赖库")

    message(CHECK_START "寻找头文件")

    set(HEADER_FOUND True)
    if(HEADER_FOUND)
        message(CHECK_PASS "已找到")
    else()
        message(CHECK_FAIL "未找到")
    endif()

    message(CHECK_START "寻找源文件")

    set(SOURCE_FOUND False)
    if(SOURCE_FOUND)
        message(CHECK_PASS "已找到")
    else()
        message(CHECK_FAIL "未找到")
    endif()

if(HEADER_FOUND AND SOURCE_FOUND)
    message(CHECK_PASS "已找到")
else()
    message(CHECK_FAIL "未找到")
endif()
```

其执行结果如下：

```
> cd CMake-Book/src/ch004/message
> cmake -P 输出检查状态.cmake
-- 寻找依赖库
-- 寻找头文件
-- 寻找头文件 - 已找到
-- 寻找源文件
-- 寻找源文件 - 未找到
-- 寻找依赖库 - 未找到
```

### 4.8.4　设置输出格式

#### 设置缩进格式

`CMAKE_MESSAGE_INDENT`变量可用于设置输出提示（notice）及以下级别日志的缩进格式。该变量值是一个列表，缩进内容即连接列表元素得到的字符串。在输出日志时，缩进内容将被插入到日志消息字符串的前面，前缀（若存在）的后面。如下所示中是一个实例。

```
function(f0)
    list(APPEND CMAKE_MESSAGE_INDENT "  ")
    message(STATUS "f0被调用")
endfunction()

function(f1_1)
    list(APPEND CMAKE_MESSAGE_INDENT "  ")
    message(STATUS "f1_1被调用")
endfunction()

function(f1)
    list(APPEND CMAKE_MESSAGE_INDENT "  ")
    message(STATUS "f1被调用")
    f1_1()
endfunction()

list(APPEND CMAKE_MESSAGE_INDENT "++")
message("开始")
f0()
f1()
f0()
list(POP_BACK CMAKE_MESSAGE_INDENT)
message("结束")
```

其执行结果如下：

```
> cd CMake-Book/src/ch004/message
> cmake -P 设置缩进格式.cmake
-- ++开始
-- ++  f0被调用
-- ++  f1被调用
-- ++    f1_1被调用
-- ++  f0被调用
结束
```

使用`list(APPEND)`和`list(POP_BACK)`命令可以方便地逐级调整缩进格式，这也是缩进变量值采用列表格式的原因。另外，函数具有独立的变量作用域，在函数内部对CMake变量的修改都不会影响外部同名变量的值，因此函数结束后无须调用`list(POP_BACK)`。这对于其他具有独立作用域的结构同样适用。

#### 设置输出上下文

`CMAKE_MESSAGE_CONTEXT`变量可用于设置输出提示（notice）及以下级别日志的上下文信息。该变量值也是一个列表，上下文信息的内容即连接列表元素得到的字符串。在输出日志时，它将被插入到日志消息字符串或缩进（若存在）的前面、前缀（若存在）的后面。如下所示中是一组实例。

```
function(f0)
    list(APPEND CMAKE_MESSAGE_CONTEXT "f0")
    list(APPEND CMAKE_MESSAGE_INDENT "  ")
    message(STATUS "f0被调用")
endfunction()

function(f1)
    list(APPEND CMAKE_MESSAGE_CONTEXT "f1")
    list(APPEND CMAKE_MESSAGE_INDENT "  ")
    message(STATUS "f1被调用")
    f0()
endfunction()

list(APPEND CMAKE_MESSAGE_CONTEXT "主程序")
f0()
f1()
```

其执行结果如下：

```
> cd CMake-Book/src/ch004/message
> cmake -P 设置输出上下文.cmake
-- [主程序.f0]   f0被调用
-- [主程序.f1]   f1被调用
-- [主程序.f1.f0]     f0被调用
```

与设置缩进格式类似，使用`list(APPEND)`和`list(POP_BACK)`命令可以方便地按照层级设置上下文名称。

不要默认`CMAKE_MESSAGE_CONTEXT`变量在程序最外层是空值。对于具有多层级结构的项目而言，CMake可能会预设该变量的值。另外，合法的上下文命名规则与CMake变量的命名规则一致，且以下画线或`“cmake_”`开头的名称均为CMake的保留命名，不应当用作项目自定义的上下文名称。

## 4.9　执行程序：execute_process

同其他脚本程序一样，CMake脚本程序中也常常需要执行外部程序，以进一步扩展CMake功能。CMake提供了用于执行程序（进程）的命令`execute_process`，使用起来非常简单，功能也很完备。其命令形式如下：

```
execute_process(COMMAND <命令1> [<命令行参数>...]
    [COMMAND <命令2> [<命令行参数>...]]...
    [WORKING_DIRECTORY <工作目录>]
    [TIMEOUT <超时秒数>]
    [RESULT_VARIABLE <返回值变量>]
    [RESULTS_VARIABLE <返回值列表变量>]
    [OUTPUT_VARIABLE <标准输出变量>]
    [ERROR_VARIABLE <标准错误输出变量>]
    [INPUT_FILE <标准输入文件路径>]
    [OUTPUT_FILE <标准输出文件路径>]
    [ERROR_FILE <标准错误输出文件路径>]
    [OUTPUT_QUIET]
    [ERROR_QUIET]
    [COMMAND_ECHO <STDERR|STDOUT|NONE>]
    [OUTPUT_STRIP_TRAILING_WHITESPACE]
    [ERROR_STRIP_TRAILING_WHITESPACE]
    [ENCODING <NONE|AUTO|ANSI|OEM|UTF8|UTF-8>]
    [ECHO_OUTPUT_VARIABLE]
    [ECHO_ERROR_VARIABLE]
    [COMMAND_ERROR_IS_FATAL <ANY|LAST>])
```

该命令的参数很多，但读者大可不必惊慌，后面会分类介绍。现在重点关注 COMMAND参数，它后面跟着想要执行的命令（程序路径）及其参数。COMMAND参数可以多次指定以执行多个命令。

`execute_process`命令功能的准确定义：管道并行执行若干子进程。这个定义虽然简短，但体现了该命令的重要特性：“管道”“并行”和“子进程”。

下面将对该命令的特性及参数进行介绍。

### 4.9.1　管道输出

当指定了多个COMMAND参数时，前一个命令对应子进程执行过程中产生的标准输出会被管道输出至后一个命令对应子进程的标准输入中。因此，在默认情况下，只有最后一个命令对应子进程的标准输出会被输出到终端中。

对于标准错误输出，所有子进程共享同一个标准错误输出管道，且在默认情况下都会被输出到终端中。

### 4.9.2　并行执行

当指定了多个COMMAND参数时，尽管它们对应子进程的标准输入输出会被管道串联起来，但它们并非按照指定的顺序依次执行。输入输出管道并不阻碍程序的并行执行，如果后一个命令的进程完全不需要接受标准输入，那么它完全可以与前一个命令的进程同时开始执行，而不必等待任何标准输入。另外，管道输出也是异步的，后一个命令的进程不必等待前一个命令的进程运行结束才接收完整的标准输出，而是在并行的同时，伴随着前一个命令的持续输出而不断输入。下面这个实例可以证明这一点。

首先，创建一个CMake脚本程序，它会输出两次变量text的值，但第二次输出前会延时停顿1秒，如下所示。

CMake -E命令行的用法将在6.3.5小节中讲解。该命令行形式主要用于提供一些跨平台的IO相关功能，类似于Shell命令。

```
message("${text}")
execute_process(COMMAND ${CMAKE_COMMAND} -E sleep 1)
message("${text}")
```

然后，创建另一个CMake脚本程序，这是将要执行的主程序，其中调用`execute_process`命令并行执行了三个命令。这三个命令都是执行刚才创建的延时输出日志的CMake脚本程序，只不过传递不同的-D参数，定义了不同的变量值，用于区分它们的输出，如下所示。

```
execute_process(
    COMMAND ${CMAKE_COMMAND} -Dtext=1 -P 延时输出日志.cmake
    COMMAND ${CMAKE_COMMAND} -Dtext=2 -P 延时输出日志.cmake
    COMMAND ${CMAKE_COMMAND} -Dtext=3 -P 延时输出日志.cmake
)
```

其执行结果如下：

```
> cd CMake-Book/src/ch004/execute_process
> cmake -P 并行执行程序.cmake
2
3
1
2
1
3
```

可见，输出的数值是乱序的，也就是说这三个命令的执行顺序是不确定的，是并行执行的。

如果确实需要顺序执行多个命令，可多次调用execute_process命令，且每一个命令中仅使用一个COMMAND参数指定一个命令。

### 4.9.3　子进程继承环境变量

CMake在执行该命令时，会调用系统的API启动子进程。这些子进程会继承父进程（也就是CMake进程）的环境变量。下面来看一个实例。

首先，创建一个CMake脚本程序用于输出环境变量text，如下所示。

```
message("$ENV{text}")
```

然后，在主程序中调用两次`execute_process`命令，并分别提前设置好不同的text值，如下所示。

```
set(ENV{text} hello)
execute_process(COMMAND ${CMAKE_COMMAND} -P 输出环境变量.cmake)
# 输出：hello

set(ENV{text} world)
execute_process(COMMAND ${CMAKE_COMMAND} -P 输出环境变量.cmake COMMAND_ECHO STDERR)
# 输出：world
```

根据程序输出可以看出主程序的环境变量确实被继承至执行的子进程中。

### 4.9.4　设置工作目录

```
WORKING_DIRECTORY <工作目录>
```

<工作目录>参数用于指定执行子进程时的工作目录。

### 4.9.5　获取进程返回值

```
RESULT_VARIABLE <返回值变量>
```

`RESULT_VARIABLE`参数用于指定<返回值变量>的名称。该变量用于存放最后一个子进程执行结束后返回的数值，即返回码（return code），也称退出码（exit code）。当执行程序超时时，返回值变量的值则为描述错误信息的字符串。

```
RESULTS_VARIABLE <返回值列表变量>
```

`RESULTS_VARIABLE`参数用于指定<返回值列表变量>的名称。返回值列表变量的元素按照 COMMAND参数指定的命令顺序，分别对应其每个子进程的返回码。当某个子进程执行超时时，其对应元素值则为描述错误信息的字符串。下面的实例中分别调用了两个命令，其中第二个命令包含不合法的参数，如如下所示。

```
execute_process(
    COMMAND ${CMAKE_COMMAND} -E echo hello
    COMMAND ${CMAKE_COMMAND} -E xxx # 不合法的参数
    RESULTS_VARIABLE res
) # 输出：
# CMake Error: cmake version 2.20
# Usage: C:/Program Files/CMake/bin/cmake.exe -E <command> [arguments...]
# ...

message("${res}") # 输出：0;1
```

终端中先输出了一些错误信息，然后输出了返回值列表变量的值。其中，第一个命令的返回值为0，说明执行成功；第二个命令的返回值为1，表示执行失败。

### 4.9.6　设置超时时长

```
TIMEOUT <超时秒数>
```

若子进程运行时间超过了<超时秒数>参数指定的值（可为小数），这些子进程会被强制终止，且返回值变量的值会被设置成一个描述错误的字符串（该字符串一定包含timeout字符串）。 如下所示中是一个实例，其中执行进程时设置的<超时秒数>参数值为0.1秒，而命令行中执行的“延时输出日志.cmake”程序中存在1秒的延时，因此该命令必然会执行超时。

```
execute_process(
    COMMAND ${CMAKE_COMMAND} -Dtext=1 -P 延时输出日志.cmake
    TIMEOUT 0.1
    RESULT_VARIABLE res
) # 输出：1

message("${res}") # 输出：Process terminated due to timeout
```

### 4.9.7　设置输出变量

```
OUTPUT_VARIABLE <标准输出变量>
ERROR_VARIABLE <标准错误输出变量>
```

这两个参数分别用于将最后一个子进程的标准输出和全部子进程的标准错误输出存入<标准输出变量> 和<标准错误输出变量>中。二者可以指定为同一个变量，这样标准输出和标准错误输出的结果会按照输出的时间顺序一同存入这个变量。

当设置该参数后，标准输出和标准错误输出就不会再被输出到终端中了。如果想要让它们既能输出到终端中，又可以存入变量中，那么就需要设置下面这两个参数：

```
ECHO_OUTPUT_VARIABLE
ECHO_ERROR_VARIABLE
```

这两个参数分别用于将标准输出和标准错误输出复制一份，同时重定向到设置的变量中和终端输出中。

下面的实例分别演示了标准错误输出和标准输出的重定向。首先，CMake脚本程序“输出变量.cmake”会使用message命令将变量text的值输出到标准错误输出中，如下所示。

```
message("${text}")
```

主程序中，首先调用两次`cmake -E echo命令行`，输出不同字符串到标准输出中，并将其重定向到变量中，然后调用两次“输出变量.cmake”脚本程序，使其输出不同的字符串到标准错误输出中，也同样重定向到变量中。主程序如下所示。

```
execute_process(
    COMMAND ${CMAKE_COMMAND} -E echo hello
    COMMAND ${CMAKE_COMMAND} -E echo world
    OUTPUT_VARIABLE out
    ECHO_OUTPUT_VARIABLE
) # 输出：world

message("${out}") # 输出：world

execute_process(
    COMMAND ${CMAKE_COMMAND} -Dtext=hello -P 输出变量.cmake
    COMMAND ${CMAKE_COMMAND} -Dtext=world -P 输出变量.cmake
    ERROR_VARIABLE out
    ECHO_ERROR_VARIABLE
) # 输出：
# world
# hello

message("${out}") 
# 输出：
# world
# hello
```

该命令只会重定向最后一个子进程的标准输出，而对于标准错误输出，则会重定向全部子进程的。另外，两个命令是并行执行的，因此它们输出到标准错误输出的顺序是不稳定的。

`execute_process`命令中与标准输出重定向相关的行为，都仅涉及最后一个子进程的标准输出。这是由`execute_process`的管道输出特性所决定的，即前面命令对应子进程的标准输出都被重定向至其相邻的下一个命令对应子进程的标准输入中了。

### 4.9.8　设置输入输出文件

```
INPUT_FILE <标准输入文件路径>
```

<标准输入文件路径>参数用于将其对应文件的内容作为第一个子进程的标准输入。

```
OUTPUT_FILE <标准输出文件路径>
ERROR_FILE <标准错误输出文件路径>
```

<标准输出文件路径>参数用于将最后一个子进程的标准输出重定向到其指定路径的文件中。<标准错误输出文件路径>参数用于将全部子进程的标准错误输出重定向到其指定路径的文件中。

如下所示例程演示了`ERROR_FILE`的使用。

```
execute_process(
    COMMAND ${CMAKE_COMMAND} -Dtext=hello -P 输出变量.cmake
    COMMAND ${CMAKE_COMMAND} -Dtext=world -P 输出变量.cmake
    ERROR_FILE out.txt
)
# out.txt文件的内容如下：
# world
# hello
```

### 4.9.9　屏蔽输出

```
OUTPUT_QUIET
ERROR_QUIET
```

这两个参数分别用于屏蔽子进程的标准输出和标准错误输出。屏蔽后，子进程的标准输出或标准错误输出将不会被输出到终端，且标准输出变量或标准错误输出变量的值也不会被设置。

### 4.9.10　删除输出尾部空白

```
OUTPUT_STRIP_TRAILING_WHITESPACE
ERROR_STRIP_TRAILING_WHITESPACE
```

这两个参数分别用于删除重定向到变量中的标准输出和标准错误输出中的尾部空白符。

### 4.9.11　输出命令行调用

```
COMMAND_ECHO <STDERR|STDOUT|NONE>
```

该参数用于设置是否将`execute_process`正在调用的命令行输出到终端中。设置其参数值为NONE时，命令行调用不会被输出；设置其值为STDERR时，命令行调用会被输出到标准错误输出中；设置其值为STDOUT时，命令行调用会被输出到标准输出中。

当省略该参数时，CMake变量`CMAKE_EXECUTE_PROCESS_COMMAND_ECHO`的值会作为该参数的默认值。

如下所示中的实例将命令行调用输出到了标准输出中。

```
execute_process(
    COMMAND ${CMAKE_COMMAND} -Dtext=hello -P 输出变量.cmake
    COMMAND ${CMAKE_COMMAND} -Dtext=world -P 输出变量.cmake
    COMMAND ${CMAKE_COMMAND} -E echo hello
    COMMAND ${CMAKE_COMMAND} -E echo world
    COMMAND_ECHO STDOUT
) # 输出：
# '.../cmake' '-Dtext=hello' '-P' '输出变量.cmake'
# '.../cmake' '-Dtext=world' '-P' '输出变量.cmake'
# '.../cmake' '-E' 'echo' 'hello'
# '.../cmake' '-E' 'echo' 'world'
# world
# world
# hello
```

由于只有最后一个命令对应子进程的标准输出才会被输出到终端，最终的输出结果中仅存在一个属于标准错误输出的“hello”。

### 4.9.12　设置输出编码

```
ENCODING <NONE|UTF8|UTF-8|AUTO|ANSI|OEM>
```

该参数仅对Windows操作系统有效，可用于指定对进程输出进行解码时所采用的编码方式。其取值可为下列参数值之一。

- NONE，即无须解码。当进程输出的编码使用UTF-8编码方式时，可选择该值。这也是默认取值。

- UTF8或UTF-8，即使用UTF-8编码方式进行解码。

- AUTO，即使用终端的当前代码页（active code page）。若当前代码页不可用，则使用ANSI代码页进行解码。

- ANSI，即使用ANSI代码页。

- OEM，即使用OEM代码页。

### 4.9.13　设置失败条件

```
COMMAND_ERROR_IS_FATAL <ANY|LAST>
```

该参数用于设置`execute_process`在遇到命令执行出错时的行为。其取值可为下列参数值之一。

- ANY，当任意命令出现错误时，终止程序运行并报告致命错误。

- LAST，当且仅当最后一个命令出现错误时，终止程序运行并报告致命错误；前面的命令执行出错不会导致致命错误。

### 4.9.14　解析命令行参数：separate_arguments

当需要将一个完整的字符串形式的命令行解析成一个个命令行参数时，可以使用`separate_arguments`命令实现。鉴于它与执行命令行程序息息相关，本小节同时也会介绍`separate_arguments`命令的功能。

```
separate_arguments(<结果变量> <解析模式> [PROGRAM [SEPARATE_ARGS]] <命令行>)
```

该命令会将<命令行>字符串参数解析成一个列表变量，并存入<结果变量>中，其中的每一个元素都对应命令行中的一个参数。

解析命令行的规则与操作系统有关，因此该命令提供了<解析模式>参数，可用于选择解析命令行的规则。其取值可为下列参数值之一。

- `UNIX_COMMAND`，表示UNIX模式。在该模式下，命令行参数是被不在引号内的空格隔开的。单引号和双引号都可用于表示作为整体的字符串。反斜杠可用于转义字符，其转义后的字符即反斜杠后面紧跟着的字符（如`\"`转义为`"`，`\n`转义为`n`）。

- `WINDOWS_COMMAND`，表示Windows模式，在该模式下，命令行参数是被不在双引号内的空格隔开的。反斜杠仅当其位于双引号前时，用于转义双引号。其他细节可以参阅微软官方文档网站中与解析C命令行参数相关的文档。

- `NATIVE_COMMAND`，表示本机模式。在该模式下，CMake会根据当前操作系统选择对应的`UNIX_COMMAND`或`WINDOWS_COMMAND`模式。

如下所示例程展示了`UNIX_COMMAND`和`WINDOWS_COMMAND`模式的区别。

```
set(cmd [[cmd a 'b' c]]) # 注意这里用了括号参数

separate_arguments(out WINDOWS_COMMAND "${cmd}")
message("{out}") # 输出：cmd;a;'b';c

separate_arguments(out UNIX_COMMAND "${cmd}")
message("${out}") # 输出：cmd;a;b;c
```

该命令还有两个参数：`PROGRAM`和`SEPARATE_ARGS`。其中后者仅在前者被指定的情况下才有效。

指定`PROGRAM`参数后，该命令将以命令行中的第一个参数为命令的可执行文件，并在系统搜索路径中解析其所在路径。若未能找到该可执行文件，结果变量会被设置为空值；若成功找到该可执行文件，结果变量将被设置为包含两个元素的列表，其中第一个元素为可执行文件的绝对路径，第二个元素为剩余的命令行参数。也就是说，指定`PROGRAM`参数后，该命令将不再默认把参数切分成一个个独立的元素。不过，如果同时指定`SEPARATE_ARGS`参数，该命令将继续切分参数字符串。

如下所示中是一个实例。

```
# 因为CMake在不同平台的安装路径不同，此处在输出中略去了其绝对路径
separate_arguments(out WINDOWS_COMMAND PROGRAM "cmake -P a.cmake")
message("${out}") # 输出：.../cmake.exe; -P a.cmake

separate_arguments(out WINDOWS_COMMAND PROGRAM 
    SEPARATE_ARGS "cmake -P a.cmake")
message("${out}") # 输出：.../cmake.exe;-P;a.cmake
```

## 4.10　引用CMake程序：include

几乎所有编程语言都会提供代码复用机制，即引用外部程序的方法。第3章提到过 CMake的模块程序可以算作CMake中主要的代码复用单元，能够被CMake脚本程序及目录程序通过include命令引用。

### 4.10.1　引用CMake程序

```
include(<CMake程序文件|CMake模块> [OPTIONAL] 
        [RESULT_VARIABLE <结果变量>] [NO_POLICY_SCOPE])
```

该命令会加载并执行`<CMake程序文件>`或`<CMake模块>`参数指定的外部CMake程序。带扩展名（一般是.cmake）的路径将被视为`<CMake程序文件>`，可以是绝对路径，也可以是相对于当前目录的相对路径；不带扩展名的路径会被视为`<CMake模块>`，只能是相对路径，而且并非相对于当前目录。该命令会依次将`CMAKE_MODULE_PATH`列表变量中的路径作为父目录来搜索`<CMake模块>`，若仍未能搜索到，则从CMake预置模块目录中搜索指定的`<CMake模块>`。

引用的外部程序与该命令所在的调用上下文共享相同的作用域，也就是说，引用的程序内创建的变量，在include命令之后仍然存在。

默认情况下，若外部程序不存在，该命令会报错。指定OPTIONAL参数后，该命令则会忽略该错误。

<结果变量>用于存放外部程序的绝对路径，若外部程序不存在，该变量的值会被设置为NOTFOUND。

如下所示中是一个实例。

```
include(外部程序.cmake)
message("a: ${a}")

include(外部程序 OPTIONAL RESULT_VARIABLE out) 
message("include(外部程序): ${out}")

set(CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR})
include(外部程序 RESULT_VARIABLE out)
message("${out}")
```

其中引用的“外部程序.cmake”如下所示。

```
message("模块被执行")
set(a "变量a")
```

其执行结果如下：

```
> cd CMake-Book/src/ch004/include
> cmake -P include.cmake
模块被执行
a: 变量a
include(外部程序): NOTFOUND
模块被执行
.../CMake-Book/src/ch004/10.include/外部程序.cmake
```

参数`NO_POLICY_SCOPE`会在10.3.4小节中介绍。

### 4.10.2　引用卫哨：include_guard

有时候，一些外部CMake程序可能同时又引用了一些其他外部程序，这时就有可能出现重复引用的情况。重复引用会导致重复加载和执行，可能会造成执行效率下降，甚至逻辑错误等。

如同C和`C++`通过宏定义或`“#pragma once”`预处理指令来实现头文件卫哨一样，CMake也提供了`include_guard`命令来实现引用卫哨：

```
include_guard([DIRECTORY|GLOBAL])
```

该命令可以为当前CMake文件（即`CMAKE_CURRENT_LIST_FILE`变量的值）设置指定作用域中的引用卫哨。当CMake执行某个程序文件时，若它在当前作用域中已被加载执行过，那么该命令就会阻止该程序继续执行，如同执行return命令一样。

引用卫哨的作用域默认与set命令定义的变量的作用域相同，即当前函数或当前目录的作用域。此时，该命令的行为可以用下面的伪代码来描述：

```
if(当前文件是否已执行)
    return()
endif()
set(当前文件是否已执行 TRUE)
```

另外，通过指定下列参数之一，可以更改引用卫哨的作用域。

- DIRECTORY，即当前目录及其子目录的作用域。指定该参数后，在这些目录的作用域中，程序文件保证仅被引用一次。

- GLOBAL，即全局作用域。指定该参数后，在CMake的执行过程中，程序文件保证仅被引用一次。

在宏或函数中，`CMAKE_CURRENT_LIST_FILE`变量的值是调用上下文所在的 CMake文件名，而非宏或函数定义所在的CMake文件名。因此，应避免在宏或函数中调用 `include_guard`命令，以免为错误的程序文件设定了引用卫哨。

下面的实例中创建了两个带有引用卫哨的程序，分别采用默认作用域和全局作用域，如下所示。

```
include_guard()
message("带卫哨的程序 被执行")
```

```
include_guard(GLOBAL)
message("带全局卫哨的程序 被执行")
```

再创建两个CMake程序，分别引用上面两个CMake程序，如下所示。

```
message("引用带卫哨的程序 被执行")
include(带卫哨的程序.cmake)
```

```
message("引用带全局卫哨的程序 被执行")
include(带全局卫哨的程序.cmake)
```

最后，在主程序中实现两个函数并分别调用两次。其中，函数A和函数B会分别引用不同的带卫哨的程序，如下所示。

```
function(A)
    include(带卫哨的程序.cmake)
    include(引用带卫哨的程序.cmake)
endfunction()

function(B)
    include(带全局卫哨的程序.cmake)
    include(引用带全局卫哨的程序.cmake)
endfunction()

A()
A()

message("---")

B()
B()
```

其执行结果如下：

```
> cd CMake-Book/src/ch004/include
> cmake -P include_guard.cmake
带卫哨的程序 被执行
引用带卫哨的程序 被执行    
带卫哨的程序 被执行        
引用带卫哨的程序 被执行    
---
带全局卫哨的程序 被执行    
引用带全局卫哨的程序 被执行
引用带全局卫哨的程序 被执行
```

在函数A的一次调用中，“带卫哨的程序”仅被执行了一次。当引用“引用带卫哨的程序”时，“带卫哨的程序”不会再次被执行。不过，第二次调用函数A，“带卫哨的程序”仍然要被执行一次。这是因为该卫哨采用默认作用域，这里对应函数A的作用域。

“带全局卫哨的程序”则只被执行了一次。对于全局卫哨而言，不管调用多少次函数B，它都一定能够保证程序只被引用一次。

## 4.11　执行代码片段：cmake_language

脚本程序往往都具备动态执行一段程序代码的功能，如Python的eval、exec函数， JavaScript的eval函数和Function对象等。CMake在其3.18版本以后，提供了一个新的命令`cmake_language`，用以实现这种动态执行代码片段的功能。

### 4.11.1　调用命令

```
cmake_language(CALL <CMake命令> [<命令参数>...])
```

该命令用于在当前调用上下文中调用指定的`<CMake命令>`，并通过<命令参数>向被调用的命令传递参数。由于`cmake_language`命令在当前调用上下文中执行命令，不会引入新的作用域。

`<CMake命令>`参数可以是内置命令或者通过macro或function命令定义的宏或函数的名称。不过，由于该命令仅用于执行单个CMake命令，为了保证程序的可读性，不支持成对或配套使用的命令，包括：

- if、elseif、else和endif；

- while和endwhile；

- foreach和endforeach；

- function和endfunction；

- macro和endmacro。

如下所示中是一个调用命令实例。

```
set(cmd "message")
cmake_language(CALL "${cmd}" "您好") # 输出：您好
```

该例程的代码与`message("您好")`是等价的。

### 4.11.2　执行代码

```
cmake_language(EVAL CODE <CMake代码>...)
```

该命令用于执行若干CMake代码片段。如下所示中是一个实例。

```
set(a TRUE)

cmake_language(EVAL CODE "
    if(a)
        message(TRUE)
    else()
        message(FALSE)
    endif()
" "
    if(b)
        message(TRUE)
    else()
        message(FALSE)
    endif()
") # 输出：
# TRUE
# FALSE
```

事实上，`cmake_language(EVAL)`这个子命令相当于将代码片段写入临时文件，然后通过 include命令引用这个临时程序文件。因此，上述所示的例程实际上等价于如下所示的例程。

```
set(a TRUE)

file(WRITE temp.cmake "
    if(a)
        message(TRUE)
    else()
        message(FALSE)
    endif()
" "
    if(b)
        message(TRUE)
    else()
        message(FALSE)
    endif()
")

include(temp.cmake)
```

### 4.11.3　延迟调用命令

该系列与延迟调用命令相关的子命令仅用于CMake目录程序（即CMakeLists.txt），如果读者还不熟悉目录程序，可以先跳过本小节。

#### 延迟调用命令

```
cmake_language(DEFER 
    [DIRECTORY <目录>]
    [ID <ID>]
    [ID_VAR <ID变量>]
    CALL <CMake命令> [<命令参数>...]
)
```

该命令用于在CMake目录程序中延迟调用指定的`<CMake命令>`。默认情况下，该命令会在当前目录的目录程序执行结束后被调用。<命令参数>如果包含变量引用，那么这些变量引用会在延迟调用的命令真正被执行时解析成对应的变量值。

DIRECTORY参数用于设置当前延迟调用命令的调用时机为指定<目录>对应的目录程序执行结束后。<目录>可以被设置为某个CMake目录程序的源文件目录或对应的构建目录。若设置为源文件目录，则可以使用相对于当前目录程序源文件目录的相对路径。省略该参数时，其值默认为当前目录。

给定的<目录>必须已经被CMake感知到。也就是说，该目录要么已经被`add_subdirectory`命令添加为子目录，要么是顶级目录程序的源文件目录，要么是上述两种目录对应的构建目录。

`<ID>`参数用于指定延迟调用的自定义ID。该ID可以结合其他几个子命令，用来获取对应延迟调用的命令及其参数，或用来取消对应的延迟调用。自定义ID不可以由大写字母或下画线开头。若省略该参数，CMake会为延迟调用自动生成一个以下画线开头的ID。

`<ID变量>`参数用于指定一个变量来获取延迟调用的ID。

#### 获取延迟调用命令

```
cmake_language(DEFER [DIRECTORY <目录>] GET_CALL <ID> <结果变量>)
```

该命令用于获取<目录>（若省略，则默认为当前目录）中指定`<ID>`的延迟调用命令及其参数。命令名称和参数会作为列表元素存入<结果变量>。

若存在多个延迟调用的ID相同，该命令仅可获取其中第一个延迟调用；若不存在指定ID的延迟调用，则结果变量会被赋为空值。

#### 获取全部延迟调用命令ID

```
cmake_language(DEFER [DIRECTORY <目录>] GET_CALL_IDS <结果变量>)
```

该命令用于获取<目录>（若省略，则默认为当前目录）中全部延迟调用的ID，并将其以列表形式存入<结果变量>。

#### 取消延迟调用

```
cmake_language(DEFER [DIRECTORY <目录>] CANCEL_CALL <ID>)
```

该命令用于取消<目录>（若省略，则默认为当前目录）中指定`<ID>`对应的延迟调用。若不存在对应ID的延迟调用，该命令会被忽略，并不产生错误。

#### 实例

下面的实例中综合应用了上述子命令。

为了演示延迟调用可以作用于指定的目录，首先创建一个名为“子目录”的子目录，并在其中创建一个CMake目录程序。该程序中定义了2个延迟调用，二者作用于子目录的上级目录（通过..指定<目录>参数），如下所示。

```
message("----子目录程序开始----")

# 注意这里的延迟调用都是定义在上级目录中的
cmake_language(DEFER
    DIRECTORY ..
    ID_VAR id1
    CALL message "结束2: ${var}"
)
cmake_language(DEFER 
    DIRECTORY ..
    ID 自定义ID
    ID_VAR id2
    CALL message 结束3
)

message("id1: ${id1}")
message("id2: ${id2}")

message("----子目录程序结束----")
```

回到项目顶层目录，创建CMake目录程序，在其中调用多个与延迟调用命令相关的命令，如下所示。

```
cmake_minimum_required(VERSION 3.20)

project(延迟调用)

cmake_language(DEFER CALL message 结束)

message("----程序开始----")
add_subdirectory(子目录)

cmake_language(DEFER GET_CALL_IDS ids)
message("当前目录的延迟调用ID: ${ids}")

cmake_language(DEFER GET_CALL 自定义ID cmd)
message("取消调用 自定义ID\n其命令及参数为: ${cmd}")

cmake_language(DEFER CANCEL_CALL 自定义ID)

set(var "再见")
message("----程序结束----")
```

其执行结果如下：

```
> cd CMake-Book/src/ch004/cmake_language/延迟调用
> mkdir build
> cd build
> cmake ..
-- ...
　
----程序开始----
----子目录程序开始----
id1: __1
id2: 自定义ID
----子目录程序结束----
当前目录的延迟调用ID: __0;__1;自定义ID
取消调用 自定义ID
其命令及参数为: message;结束3
----程序结束----
结束
结束2: 再见
　
-- Configuring done
-- Generating done
-- ...
```

最终只有两个延迟调用成功执行，而本应输出“结束3”的延迟调用则被成功取消。

## 4.12　监控变量：variable_watch

虽然CMake在功能上完全不逊于很多常见的脚本语言，但它在调试方面确实还有欠缺。至少在本书写作之时，笔者仍未见到有任何一个开发工具可以支持CMake的断点调试。不过好在CMake提供了很多其他有助于调试的手段。本节将介绍用于监控变量访问的`variable_watch`命令。

```
variable_watch(<变量> [<回调命令名>])
```

当<变量>被其他命令访问时，CMake会调用<回调命令名>指定的命令。若 <回调命令名>参数被省略，CMake则会输出与访问变量相关的信息。

回调命令的参数形式如下：

```
回调命令名(<变量> <访问形式> <新值> <CMake程序文件路径> <调用栈>)
```

<变量>即监控的变量名称，与传给`variable_watch`命令的参数一致。

<访问形式>取值如下：

- `READ_ACCESS`，表示读取变量值；

- `UNKNOWN_READ_ACCESS`，表示读取未定义变量的值（定义后被unset的变量不属于未定义变量）；

- `MODIFIED_ACCESS`，表示修改变量值；

- `UNKNOWN_MODIFIED_ACCESS`，表示修改未定义变量的值；

- `REMOVED_ACCESS`，表示移除变量的访问（如unset命令就会做这种访问）。

<新值>即变量的值。若变量被修改，则为其修改后的值；若变量被移除，则为空值。

`<CMake程序文件路径>`即访问变量的CMake程序文件的绝对路径。

<调用栈>是一个列表，其中包含从CMake当前执行的程序，到访问变量的CMake程序的完整调用过程中涉及的全部程序文件的绝对路径。

有些命令，如list(APPEND)，会访问变量两次：第一次读取变量值，第二次修改变量值。因此，使用`variable_watch`监控它访问的变量时会触发两次回调。而对于像`if(DEFINED)`这样的条件判断命令，因为无须访问变量值，所以不会触发`variable_watch`命令的回调。

`variable_watch`命令仅可用于监控非缓存变量。

### 实例

下面的实例创建了两个CMake脚本程序。首先，创建主程序main.cmake，用于监控变量A和变量B，其中对变量A的监控使用了自定义的回调命令，如下所示。

```
function(callback var access value filename stack)
    message("${access}访问变量${var}，其值为${value}")
    message("文件路径：${filename}")
    foreach(item ${stack})
        message("    ${item}")
    endforeach()
    message("---")
endfunction()

variable_watch(A callback)
variable_watch(B)

# 访问B变量，触发默认回调，输出变量信息
set(B "${B}")

# 访问A变量，触发自定义回调命令callback
set(A "1")
list(APPEND A "2")
unset(A)

# 在引用的程序中访问变量A，注意观察输出的调用栈
include(b.cmake)
```

然后，创建第二个脚本程序b.cmake，其中仅对变量A进行赋值操作，如下所示。

```
set(A "A 位于 b.cmake")
```

其执行结果如下：

```
> cd CMake-Book/src/ch004/variable_watch
> cmake -P main.cmake
CMake Debug Log at main.cmake:14 (set):
  Variable "B" was accessed using UNKNOWN_READ_ACCESS with value "".
　
CMake Debug Log at main.cmake:14 (set):
  Variable "B" was accessed using MODIFIED_ACCESS with value "".
　
MODIFIED_ACCESS访问变量A，其值为1
文件路径：.../CMake-Book/src/ch004/variable_watch/main.cmake
    .../CMake-Book/src/ch004/variable_watch/main.cmake
---
READ_ACCESS访问变量A，其值为1
文件路径：.../CMake-Book/src/ch004/variable_watch/main.cmake
    .../CMake-Book/src/ch004/variable_watch/main.cmake
---
MODIFIED_ACCESS访问变量A，其值为1;2
文件路径：.../CMake-Book/src/ch004/variable_watch/main.cmake
    .../CMake-Book/src/ch004/variable_watch/main.cmake
---
REMOVED_ACCESS访问变量A，其值为
文件路径：...CMake-Book/src/ch004/variable_watch/main.cmake
    .../CMake-Book/src/ch004/variable_watch/main.cmake
---
MODIFIED_ACCESS访问变量A，其值为A 位于 b.cmake
文件路径：.../CMake-Book/src/ch004/variable_watch/b.cmake
    .../CMake-Book/src/ch004/variable_watch/main.cmake
    .../CMake-Book/src/ch004/variable_watch/b.cmake
---
```

开头的两次日志是默认的日志输出，它们表明`set(B "${B}")`命令会对未定义变量B进行读取，而后修改其值。

另外，最后一段日志输出的调用栈清晰地展现了CMake程序的包含关系，这在调试复杂的CMake程序时非常有用。
