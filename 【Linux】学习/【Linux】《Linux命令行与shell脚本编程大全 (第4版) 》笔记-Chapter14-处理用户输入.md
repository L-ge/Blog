###### 十四、处理用户输入

1. bash shell 提供了一些不同的方法来从
用户处获取数据，包括命令行参数（添加在命令后的数据）、命令行选项（可改变命令行为的单个字母） 以及直接从键盘读取输入。

2. 命令行参数允许运行脚本时在命令行中添加数据：
    ```
    $ ./addem 10 30
    ```
    - 向脚本 addem 传递了两个命令行参数（10 和 30）。脚本会通过特殊的变量来处理命令行参数。

3. 读取参数
- bash shell 会将所有的命令行参数都指派给称作位置参数（positional parameter）的特殊变量。这也包括 shell 脚本名称。位置变量的名称都是标准数字：$0 对应脚本名，$1 对应第一个命令行参数，$2 对应第二个命令行参数，以此类推，直到$9。
- 命令行参数是在命令/脚本名之后出现的各个单词，位置参数是用于保存命令行参数（以及函数参数）的变量。
- 下面是在 shell 脚本中使用单个命令行参数的简单例子：
    ```
    $ cat positional1.sh
    #!/bin/bash
    # Using one command-line parameter
    #
    factorial=1
    for (( number = 1; number <= $1; number++ ))
    do
        factorial=$[ $factorial * $number ]
    done
    echo The factorial of $1 is $factorial
    exit
    $
    ```
    - 在 shell 脚本中， 可以像使用其他变量一样使用$1 变量。 shell 脚本会自动将命令行参数的值 分配给位置变量，无须做任何特殊处理。
- 如果需要输入更多的命令行参数， 则参数之间必须用空格分开。 shell 会将其分配给对应的位置变量。    
- 也可以在命令行中用文本字符串作为参数。
- 参数之间是以空格分隔的，所以 shell 会将字符串包含的空格视为两个参数的分隔符。要想在参数值中加入空格，必须使用引号（单引号或双引号均可）。
- 将文本字符串作为参数传递时，引号并不是数据的一部分，仅用于表明数据的起止位置。
- 如果脚本需要的命令行参数不止 9 个， 则仍可以继续加入更多的参数， 但是需要稍微修改一下位置变量名。在第 9 个位置变量之后，必须在变量名两侧加上花括号，比如${10}。
    ```
    $ cat positional10.sh
    #!/bin/bash
    # Handling lots of command-line parameters
    #
    product=$[ ${10} * ${11} ]
    echo The tenth parameter is ${10}.
    echo The eleventh parameter is ${11}.
    echo The product value is $product.
    exit
    $
    $ ./positional10.sh 1 2 3 4 5 6 7 8 9 10 11 12
    The tenth parameter is 10.
    The eleventh parameter is 11.
    The product value is 110.
    ```

4. 读取脚本名
- 可以使用位置变量$0 获取在命令行中运行的 shell 脚本名。这在编写包含多种功能或生成日志消息的工具时非常方便。
- 如果使用另一个命令来运行 shell 脚本，则命令名会和脚本名混在一起，出现在位置变量$0 中；如果运行脚本时使用的是绝对路径，那么位置变量$0 就会包含整个路径：
    ```
    $ ./positional0.sh
    This script name is ./positional0.sh .
    
    $ $HOME/scripts/positional0.sh
    This script name is /home/christine/scripts/positional0.sh .
    ```
- basename 命令可以返回不包含路径的脚本名：
    ```
    name=$(basename $0)
    echo This script name is $name.
    ```

5. 参数测试
- 在 shell 脚本中使用命令行参数时要当心。如果运行脚本时没有指定所需的参数，则可能会出问题。当脚本认为位置变量中应该有数据，而实际上根本没有的时候，脚本很可能会产生错误消息。这种编写脚本的方法并不可取。在使用位置变量之前一定要检查是否为空。
- 例如，可以使用 -n 测试来检查命令行参数$1 中是否为空。

6. 在 bash shell 中有一些跟踪命令行参数的特殊变量：
    - 参数统计
    - 获取所有的数据

7. 参数统计
- 特殊变量$#含有脚本运行时携带的命令行参数的个数。可以在脚本中的任何地方使用这个特殊变量， 就跟普通变量一样。
    ```
    if [ $# -eq 1 ]
    if [ $# -ne 2 ]
    ```
- 这个变量还提供了一种简便方法来获取命令行中最后一个参数：
    ```
    echo The last parameter is ${$#}        // 错误写法
    echo The last parameter is ${!#}        // 正确写法
    ```
    - 这说明不能在花括号内使用$，必须将$换成!。
- 当命令行中没有任何参数时， $#的值即为 0，但${!#}会返回命令行中的脚本名。

8. 获取所有的数据
- $*变量和$@变量可以轻松访问所有参数，它们各自包含了所有的命令行参数。
- $*变量会将所有的命令行参数视为一个单词。这个单词含有命令行中出现的每一个参数。 基本上， $*变量会将这些参数视为一个整体，而不是一系列个体。
- $@变量会将所有的命令行参数视为同一字符串中的多个独立的单词，以便能遍历并处理全部参数。这通常使用 for 命令完成。
- 相同之处，例子1：
    ```
    $ cat grabbingallparams.sh
    #!/bin/bash
    # Testing different methods for grabbing all the parameters
    #
    echo
    echo "Using the \$* method: $*"
    echo
    echo "Using the \$@ method: $@"
    echo
    exit
    $
    $ ./grabbingallparams.sh alpha beta charlie delta
    
    Using the $* method: alpha beta charlie delta
    
    Using the $@ method: alpha beta charlie delta
    
    $
    ```
    - 从表面上看， 两个变量产生的输出相同， 均显示了所有命令行参数。
- 不同之处，例子2：
    ```
    $ cat grabdisplayallparams.sh
    #!/bin/bash
    # Exploring different methods for grabbing all the parameters
    #
    echo
    echo "Using the \$* method: $*"
    count=1
    for param in "$*"
    do
        echo "\$* Parameter #$count = $param"
        count=$[ $count + 1 ]
    done
    #
    echo
    echo "Using the \$@ method: $@"
    count=1
    for param in "$@"
    do
        echo "\$@ Parameter #$count = $param"
        count=$[ $count + 1 ]
    done
    echo
    exit
    $ ./grabdisplayallparams.sh alpha beta charlie delta
    Using the $* method: alpha beta charlie delta
    $* Parameter #1 = alpha beta charlie delta
    Using the $@ method: alpha beta charlie delta
    $@ Parameter #1 = alpha
    $@ Parameter #2 = beta
    $@ Parameter #3 = charlie
    $@ Parameter #4 = delta
    $
    ```
    - 通过使用 for 命令遍历这两个特殊变量，可以看出二者如何以不同的方式处理命令行参数。$*变量会将所有参数视为单个参数， 而$@变量会单独处理每个参数。这是遍历命令行参数的一种绝妙方法。
- 当$*出现在双引号内时，会被扩展成由多个命令行参数组成的单个单词， 每个参数之间以 IFS 变量值的第一个字符分隔，也就是说，"$*"会被扩展为"$1c$2c..." （其中 c 是 IFS 变量值的第一个字符）。当$@出现在双引号内时，其所包含的各个命令行参数会被扩展成独立的单词，也就是说，"$@" 会被扩展为"$1""$2"...。

9. 移动参数
- shift 命令可用于操作命令行参数。shift 命令会根据命令行参数的相对位置进行移动。
- 在使用 shift 命令时，默认情况下会将每个位置的变量值都向左移动一个位置。因此，变量$3 的值会移入$2，变量$2 的值会移入$1，而变量$1 的值则会被删除（注意，变量$0 的值，也就是脚本名，不会改变）。
- 这是遍历命令行参数的另一种好方法，尤其是在不知道到底有多少参数的时候。可以只操作第一个位置变量，移动参数，然后继续处理该变量。
    ```
    $ cat shiftparams.sh
    #!/bin/bash
    # Shifting through the parameters
    #
    echo
    echo "Using the shift method:"
    count=1
    while [ -n "$1" ]
    do
        echo "Parameter #$count = $1"
        count=$[ $count + 1 ]
        shift
    done
    echo
    exit
    $
    $ ./shiftparams.sh alpha bravo charlie delta
    Using the shift method:
    Parameter #1 = alpha
    Parameter #2 = bravo
    Parameter #3 = charlie
    Parameter #4 = delta
    ```
    - 该脚本在 while 循环中测试第一个参数值的长度。当第一个参数值的长度为 0 时，循环结 束。测试完第一个参数后， shift 命令会将所有参数移动一个位置。
- 使用 shift命令时要小心。如果某个参数被移出， 那么它的值就被丢弃了，无法再恢复。
- 也可以一次性移动多个位置。只需给 shift 命令提供一个参数，指明要移动的位置数即可：
    ```
    $ cat bigshiftparams.sh
    #!/bin/bash
    # Shifting multiple positions through the parameters
    #
    echo
    echo "The original parameters: $*"
    echo "Now shifting 2..."
    shift 2
    echo "Here's the new first parameter: $1"
    echo
    exit
    $
    $ ./bigshiftparams.sh alpha bravo charlie delta
    
    The original parameters: alpha bravo charlie delta
    Now shifting 2...
    Here's the new first parameter: charlie
    
    $
    ```
    - 使用 shift 命令的值可以轻松地跳过不需要的参数。

10. 处理选项
- 选项是在连字符之后出现的单个字母，能够改变命令的行为。
- 还有另一种选项，即以双连字符（--）起始， 紧跟着一个字符串（比如--max-depth），这种形式的选项称作长 选项。
- 3 种在 shell 脚本中处理选项的方法：
    - 查找选项
    - 使用 getopt 命令
    - 使用 getopts 命令

11. 查找选项
- 在命令行中，选项紧跟在脚本名之后，就跟其他命令行参数一样。实际上，如果愿意，可以像处理命令行参数一样处理命令行选项。
- 命令行参数是在命令/脚本名之后出现的各个单词， 其中，以连字符（ - ）或双连字符（ --）起始的参数， 因其能 够改变命令的行为，称作命令行选项。所以，命令行选项是一种特殊形式的命令行参数。
- 处理简单选项
    - 可以用 shift 命令来处理命令行选项。
    - 在提取单个参数时，使用 case 语句来判断某个参数是否为选项：
        ```
        $ cat extractoptions.sh
        #!/bin/bash
        # Extract command-line options
        #
        echo
        while [ -n "$1" ]
        do
            case "$1" in
                -a) echo "Found the -a option" ;;
                -b) echo "Found the -b option" ;;
                -c) echo "Found the -c option" ;;
                *) echo "$1 is not an option" ;;
            esac
            shift
        done
        echo
        exit
        $
        $ ./extractoptions.sh -a -b -c -d
        Found the -a option
        Found the -b option
        Found the -c option
        -d is not an option
        ```
        - case 语句会检查每个参数，确认是否为有效的选项。找到一个，处理一个。
- 分离参数和选项
    - 在 Linux 中，处理这个问题的标准做法是使用特殊字符将两者分开，该字符会告诉脚本选项何时结束，普通参数何时开始。
    - 在 Linux 中，这个特殊字符是双连字符（--）。shell 会用双连字符表明选项部分结束。在双连字符之后， 脚本就可以放心地将剩下的部分作为参数处理了。
    - 要检查双连字符，只需在 case 语句中加一项即可：
        ```
        $ cat extractoptionsparams.sh
        #!/bin/bash
        # Extract command-line options and parameters
        #
        echo
        while [ -n "$1" ]
        do
            case "$1" in
                -a) echo "Found the -a option" ;;
                -b) echo "Found the -b option" ;;
                -c) echo "Found the -c option" ;;
                --) shift
                    break;;
                *) echo "$1 is not an option" ;;
            esac
            shift
        done
        #
        echo
        count=1
        for param in $@
        do
            echo "Parameter #$count: $param"
            count=$[ $count + 1 ]
        done
        echo
        exit
        $
        ```
        - 在遇到双连字符时，脚本会用 break 命令跳出while 循环。由于提前结束了循环，因此需要再加入另一个 shift 命令来将双连字符移出位置变量。
- 处理含值的选项
    - 当命令行选项要求额外的参数时，脚本必须能够检测到并正确地加以处理。来看下面的处理方法：
        ```
        $ cat extractoptionsvalues.sh
        #!/bin/bash
        # Extract command-line options and values
        #
        echo
        while [ -n "$1" ]
        do
            case "$1" in
                -a) echo "Found the -a option" ;;
                -b) param=$2
                    echo "Found the -b option with parameter value $param"
                    shift;;
                -c) echo "Found the -c option" ;;
                --) shift
                    break;;
                *) echo "$1 is not an option" ;;
            esac
            shift
        done
        #
        echo
        count=1
        for param in $@
        do
            echo "Parameter #$count: $param"
            count=$[ $count + 1 ]
        done
        exit
        $
        $ ./extractoptionsvalues.sh -a -b BValue -d
        Found the -a option
        Found the -b option with parameter value BValue
        -d is not an option
        $
        ```
        -  -b 选项还需要一个额外的参数值。由于要处理的选项位于$1，因此额外的参数值就应该位于$2（因为所有的参数在处理完之后都会被移出）。只要将参数值从 $2 变量中提取出来就可以了。
        - 因为这个选项占用了两个位置，所以还需要使用 shift 命令多移动一次。

12. 使用 getopt 命令
- 在 Linux 中，合并选项是一种很常见的用法。
- getopt 命令在处理命令行选项和参数时非常方便。它能够识别命令行参数， 简化解析过程。
- 命令格式
    - getopt 命令可以接受一系列任意形式的命令行选项和参数，并自动将其转换成适当的格式。
    - getopt 的命令格式如下：
        ```
        getopt optstring parameters
        ```
        - optstring 是这个过程的关键所在。它定义了有效的命令行选项字母，还定义了哪些选项字母需要参数值。
        - 首先，在 optstring 中列出要在脚本中用到的每个命令行选项字母。然后，在每个需要参数值的选项字母后面加一个冒号。 getopt 命令会基于你定义的 optstring 解析提供的参数。
    - getopt 命令有一个更高级的版本叫作 getopts（注意这是复数形式）。注意区分。
    - 例子：
        ```
        $ getopt ab:cd -a -b BValue -cd test1 test2
        -a -b BValue -c -d -- test1 test2
        ```
        - optstring 定义了 4 个有效选项字母： a、b、c 和 d。冒号（ :）被放在了字母 b 后面， 因为 b 选项需要一个参数值。当 getopt 命令运行时， 会检查参数列表（-a -b BValue -cd test1 test2），并基于提供的 optstring 进行解析。注意，它会自动将-cd 分成两个单独的选项，并插入双连字符来分隔命令行中额外的参数。
    - 如果 optstring 未包含你指定的选项，则在默认情况下，getopt 命令会产生一条错误消息；如果想忽略这条错误消息， 可以使用 getopt 的-q 选项：
        ```
        $ getopt ab:cd -a -b BValue -cde test1 test2
        getopt: invalid option -- 'e'
        -a -b BValue -c -d -- test1 test2
        
        $ getopt -q ab:cd -a -b BValue -cde test1 test2
        -a -b 'BValue' -c -d -- 'test1' 'test2'
        ```
        - getopt 命令选项必须出现在 optstring 之前。
- 在脚本中使用 getopt
    - 可以在脚本中使用 getopt 命令来格式化脚本所携带的任何命令行选项或参数。
    - set 命令能够处理 shell 中的各种变量。set 命令有一个选项是双连字符（--），可以将位置变量的值替换成 set 命令所指定的值。
    - 具体做法是将脚本的命令行参数传给 getopt 命令，然后再将 getopt 命令的输出传给 set 命令，用 getopt 格式化后的命令行参数来替换原始的命令行参数，如下所示：
        ```
        set -- $(getopt -q ab:cd "$@")
        ```
    - 处理命令行参数的脚本例子：
        ```
        set -- $(getopt -q ab:cd "$@")
        echo
        while [ -n "$1" ]
        do
            case "$1" in
                -a) echo "Found the -a option" ;;
                -b) param=$2
                    echo "Found the -b option with parameter value $param"
                    shift;;
                -c) echo "Found the -c option" ;;
                --) shift
                    break;;
                *) echo "$1 is not an option" ;;
            esac
            shift
        done
        #
        echo
        count=1
        for param in $@
        do
            echo "Parameter #$count: $param"
            count=$[ $count + 1 ]
        done
        exit
        ```
    - getopt 命令并不擅长处理带空格和引号的参数值。它会将空格当作参数分隔符， 而不是根 据双引号将二者当作一个参数。

13. 使用 getopts 命令
- getopts（注意是复数） 是 bash shell 的内建命令。
- getopt 与 getopts 的不同之处在于，前者在将命令行中选项和参数处理后只生成一个输出，而后者能够和已有的 shell 位置变量配合默契。
- getopts 每次只处理一个检测到的命令行参数。在处理完所有的参数后， getopts 会退出并返回一个大于 0 的退出状态码。这使其非常适合用在解析命令行参数的循环中。
- getopts 命令的格式如下：
    ```
    getopts optstring variable
    ```
    - optstring 值与 getopt 命令中使用的值类似。有效的选项字母会在 optstring 中列出， 如果选项字母要求有参数值，就在其后加一个冒号。不想显示错误消息的话， 可以在 optstring 之前加一个冒号。 getopts 命令会将当前参数保存在命令行中定义的 variable 中。
    - getopts 命令要用到两个环境变量。如果选项需要加带参数值，那么 OPTARG 环境变量保存的就是这个值。 OPTIND 环境变量保存着参数列表中 getopts 正在处理的参数位置。这样在处理完当前选项之后就能继续处理其他命令行参数了。
    - 例子：
        ```
        $ cat extractwithgetopts.sh
        #!/bin/bash
        # Extract command-line options and values with getopts
        #
        echo
        while getopts :ab:c opt
        do
            case "$opt" in
                a) echo "Found the -a option" ;;
                b) echo "Found the -b option with parameter value $OPTARG";;
                c) echo "Found the -c option" ;;
                *) echo "Unknown option: $opt" ;;
            esac
        done
        exit
        $
        $ ./extractwithgetopts.sh -ab BValue -c
        
        Found the -a option
        Found the -b option with parameter value BValue
        Found the -c option
        $
        ```
        - while 语句定义了getopts 命令， 指定要查找哪些命令行选项， 以及每次迭代时存储它们 的变量名（opt）。
        - 在解析命令行选项时， getopts 命令会移除起始的连字符，所以在 case 语句中不用连字符。
- getopts 命令有几个不错的特性：
    - 对新手来说， 可以在参数值中加入空格：
        ```
        $ ./extractwithgetopts.sh -b "BValue1 BValue2" -a
        ```
    - 另一个好用的特性是可以将选项字母和参数值写在一起，两者之间不加空格：
        ```
        $ ./extractwithgetopts.sh -abBValue
        ```
        - getopts 命令能够从-b 选项中正确解析出 BValue 值。
- 除此之外， getopts 命令还可以将在命令行中找到的所有未定义的选项统一输出成问号：  
    ```
    $ ./extractwithgetopts.sh -d
    
    Unknown option: ?
    ```
    - optstring 中未定义的选项字母会以问号形式传给脚本。
- getopts 命令知道何时停止处理选项，并将参数留给你处理。在处理每个选项时， getopts 会将 OPTIND 环境变量值增 1。处理完选项后，可以使用 shift 命令和 OPTIND 值来移动参数：
    ```
    while getopts :ab:cd opt
    do
        case "$opt" in
            a) echo "Found the -a option" ;;
            b) echo "Found the -b option with parameter value $OPTARG";;
            c) echo "Found the -c option" ;;
            d) echo "Found the -d option" ;;
            *) echo "Unknown option: $opt" ;;
        esac
    done
    #
    shift $[ $OPTIND - 1 ]
    #
    echo
    count=1
    for param in "$@"
    do
        echo "Parameter $count: $param"
        count=$[ $count + 1 ]
    done
    exit
    ```

14. 选项标准化
- 在 Linux 中，有些选项字母在某种程度上已经有了标准含义。如果能在 shell 脚本中支持 这些选项，则你的脚本会对用户更友好。
- 常用的 Linux 命令行选项
    选项 | 描述
    ---|---
    -a | 显示所有对象
    -c | 生成计数
    -d | 指定目录
    -e | 扩展对象
    -f | 指定读入数据的文件
    -h | 显示命令的帮助信息
    -i | 忽略文本大小写
    -l | 产生长格式输出
    -n | 使用非交互模式（批处理）
    -o | 将所有输出重定向至指定的文件
    -q | 以静默模式运行
    -r | 递归处理目录和文件
    -s | 以静默模式运行
    -v | 生成详细输出
    -x | 排除某个对象
    -y | 对所有问题回答 yes

15. 获取用户输入
- bash shell 提供了read 命令在脚本运行时询问用户并等待用户回答。

16. 基本的读取
- read 命令从标准输入（键盘）或另一个文件描述符中接受输入。获取输入后， read 命令会 将数据存入变量。下面是该命令最简单的用法：
    ```
    echo -n "Enter your name: "
    read name
    echo "Hello $name, welcome to my script."
    exit
    ```
    - 用于生成提示的 echo 命令使用了-n 选项。该选项不会在字符串末尾输出换行符，允许脚本用户紧跟其后输入数据。
- read 命令也提供了-p 选项，允许直接指定提示符：
    ```
    read -p "Please enter your age: " age
    days=$[ $age * 365 ]
    echo "That means you are over $days days old!"
    ```
- read 命令会将提示符后输入的所有数据分配给单个变量。如果指定多个变量，则输入的每个数据值都会分配给变量列表中的下一个变量。如果变量数量不够，那么剩下的数据就全部分配给最后一个变量：
    ```
    read -p "Enter your first and last name: " first last
    ```
- 也可以在 read 命令中不指定任何变量，这样 read 命令便会将接收到的所有数据都放进特殊环境变量 REPLY 中：
    ```
    read -p "Enter your name: "
    echo
    echo "Hello $REPLY, welcome to my script."
    ```
    - REPLY 环境变量包含输入的所有数据，其可以在 shell 脚本中像其他变量一样使用。
    
17. 超时
- 可以用-t 选项来指定一个计时器。 -t 选项会指定 read 命令等待输入的秒数。如果计时器超时，则 read 命令会返回非 0 退出状态码：
    ```
    if read -t 5 -p "Enter your name: " name
    then
        echo "Hello $name, welcome to my script."
    else
        echo
        echo "Sorry, no longer waiting for name."
    fi
    ```
- 也可以不对输入过程计时，而是让 read 命令统计输入的字符数。当字符数达到预设值时，就自动退出，将已输入的数据赋给变量：
    ```
    read -n 1 -p "Do you want to continue [Y/N]? " answer
    #
    case $answer in
    Y | y) echo
        echo "Okay. Continue on...";;
    N | n) echo
        echo "Okay. Goodbye"
        exit;;
    esac
    echo "This is the end of the script."
    exit
    ```
    - 本例中使用了-n 选项和数值 1，告诉 read 命令在接收到单个字符后退出。只要按下单个字符进行应答， read 命令就会接受输入并将其传给变量，无须按 Enter 键。

18. 无显示读取
- -s 选项可以避免在 read 命令中输入的数据出现在屏幕上（其实数据还是会被显示，只不 过 read 命令将文本颜色设成了跟背景色一样）。
- 在脚本中使用-s 选项的例子：
    ```
    read -s -p "Enter your password: " pass
    echo
    echo "Your password is $pass"
    exit
    ```
    - 屏幕上不会显示输入的数据，但这些数据会被赋给变量，以便在脚本中使用。

19. 从文件中读取
- 也可以使用 read 命令读取文件。每次调用read 命令都会从指定文件中读取一行文本。当文件中没有内容可读时， read 命令会退出并返回非 0 退出状态码。
- 将文件数据传给 read 命令最常见的方法是对文件使用cat 命令，将结果通过管道直接传给含有 read 命令的 while 命令：
    ```
    count=1
    cat $HOME/scripts/test.txt | while read line
    do
        echo "Line $count: $line"
        count=$[ $count + 1 ]
    done
    echo "Finished processing the file."
    exit
    ```
    - while 循环会持续通过 read 命令处理文件中的各行， 直到 read 命令以非0 退出状态码退出。
