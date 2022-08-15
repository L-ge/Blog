###### 十七、创建函数

1. bash shell 提供了用户自定义函数功能，可以将 shell 脚本代码放入函数中封装起来。

2. 函数是一个脚本代码块，你可以为其命名并在脚本中的任何位置重用它。每当需要在脚本中使用该代码块时，直接写函数名即可（这叫作调用函数）。

3. 在 bash shell 脚本中创建函数的语法有两种：
- 第一种语法是使用关键字 function，随后跟上分配给该代码块的函数名：
    ```
    function name {
        commands
    }
    ```
    - name 定义了该函数的唯一名称。脚本中的函数名不能重复。
    - commands 是组成函数的一个或多个 bash shell 命令。调用该函数时， bash shell 会依次执行函数内的命令，就像在普通脚本中一样。
- 第二种在 bash shell 脚本中创建函数的语法更接近其他编程语言中定义函数的方式：
    ```
    name() {
        commands
    }
    ```
    - 函数名后的空括号表明正在定义的是一个函数。这种语法的命名规则和第一种语法一样。

4. 要在脚本中使用函数，只需像其他 shell 命令一样写出函数名即可：
    ```
    function func1 {
        echo "This is an example of a function"
    }
    
    count=1
    while [ $count -le 5 ]
    do
        func1
        count=$[ $count + 1 ]
    done
    echo "This is the end of the loop"
    func1
    echo "Now this is the end of the script"
    ```
- 每次引用函数名 func1 时， bash shell 会找到 func1 函数的定义并执行在其中定义的命令。
- 函数定义不一定非要放在 shell 脚本的最开始部分，但是要注意这种情况。如果试图在函数被定义之前调用它，则会收到一条错误消息。
- 函数名必须是唯一的，否则就会出问题。如果定义了同名函数，那么新定义就会覆盖函数原先的定义，而这一切不会有任何错误消息。

5. bash shell 把函数视为一个小型脚本，运行结束时会返回一个退出状态码。有 3 种方法能为函数生成退出状态码：
    - 默认的退出状态码
    - 使用 return 命令
    - 使用函数输出

6. 在默认情况下，函数的退出状态码是函数中最后一个命令返回的退出状态码。函数执行结束后，可以使用标准变量$?来确定函数的退出状态码。
- 这个方法无法知道该函数中的其他命令是否执行成功。

7. bash shell 会使用 return 命令以特定的退出状态码退出函数。 return 命令允许指定一个整数值作为函数的退出状态码，从而提供了一种简单的编程设定方式：
    ```
    function dbl {
        read -p "Enter a value: " value
        echo "doubling the value"
        return $[ $value * 2 ]
    }
    
    dbl
    echo "The new value is $?"
    ```
- 当用这种方法从函数中返回值时，一定要小心。为了避免出问题，牢记以下两个技巧。
    - 函数执行一结束就立刻读取返回值。
    - 退出状态码必须介于 0~255。
- 如果在用$?变量提取函数返回值之前执行了其他命令，那么函数的返回值会丢失。记住， $?变量保存的是最后执行的那个命令的退出状态码。
- 由于退出状态码必须小于 256，因此函数结果也必须 为一个小于 256 的整数值。大于 255 的任何数值都会产生错误的值。
- 如果需要返回较大的整数值或字符串，就不能使用 return 方法。

8. 正如可以将命令的输出保存到 shell 变量中一样，也可以将函数的输出保存到shell 变量中： 
    ```
    result=$(dbl)
    ```
    - 这个命令会将 dbl 函数的输出赋给$result 变量。
- 例子：
    ```
    function dbl {
        read -p "Enter a value: " value
        echo $[ $value * 2 ]
    }
    result=$(dbl)
    echo "The new value is $result"
    ```
    - 新函数会用 echo 语句来显示计算结果。该脚本会获取 dbl 函数的输出，而不是查看退出状态码。
    - 注意，dbl 函数实际上输出了两条消息。read 命令输出了一条简短的消息来向用户询问输入值。bash shell 脚本非常聪明，并不将其作为 STDOUT 输出的一部分，而是直接将其忽略。如果用 echo 语句生成这条消息来询问用户，那么它会与输出值一起被读入 shell变量。
- 这种方法还可以返回浮点值和字符串，这使其成为一种获取函数返回值的强大方法。
- 函数能用标准的 echo 语句返回值。跟其他 shell 命令一样，可以用反引号来获取输出的数据，这样就能从函数中返回任意类型的数据（包括字符串和浮点数）了。

9. 向函数传递参数
- 函数可以使用标准的位置变量来表示在命令行中传给函数的任何参数。例如， 函数名保存在 $0 变量中，函数参数依次保存在$1、$2 等变量中。也可以用特殊变量$#来确定传给函数的参数 数量。
- 在脚本中调用函数时，必须将参数和函数名放在同一行，就像下面这样： 
    ```
    func1 $value1 10
    ```
- 然后函数可以用位置变量来获取参数值。来看一个使用此方法向函数传递参数的例子：
    ```
    function addem {
        if [ $# -eq 0 ] || [ $# -gt 2 ]
        then
            echo -1
        elif [ $# -eq 1 ]
        then
            echo $[ $1 + $1 ]
        else
            echo $[ $1 + $2 ]
        fi
    }
    
    echo -n "Adding 10 and 15: "
    value=$(addem 10 15)
    echo $value
    echo -n "Let's try adding just one number: "
    value=$(addem 10)
    echo $value
    echo -n "Now try adding no numbers: "
    value=$(addem)
    echo $value
    echo -n "Finally, try adding three numbers: "
    value=$(addem 10 15 20)
    echo $value
    ```
- 由于函数使用位置变量访问函数参数，因此无法直接获取脚本的命令行参数。
    ```
    function badfunc1 {
        echo $[ $1 * $2 ]
    }
    if [ $# -eq 2 ]
    then
        value=$(badfunc1)
        echo "The result is $value"
    else
        echo "Usage: badtest1 a b"
    fi
    $
    $ ./badtest1
    Usage: badtest1 a b
    $ ./badtest1 10 15
    ./badtest1: *  : syntax error: operand expected (error token is "*
    ")
    The result is
    ```
    - 尽管函数使用了$1变量和$2 变量，但它们和脚本主体中的$1变量和$2 变量不是一回事。
- 要在函数中使用脚本的命令行参数，必须在调用函数时手动将其传入：
    ```
    function func7 {
        echo $[ $1 * $2 ]
    }
    if [ $# -eq 2 ]
    then
        value=$(func7 $1 $2)
        echo "The result is $value"
    else
        echo "Usage: badtest1 a b"
    fi
    ```
    - 在将$1变量和$2 变量传给函数后，它们就能跟其他变量一样，可供函数使用了。

10. 在函数中处理变量
- 函数有两种类型的变量：
    - 全局变量
    - 局部变量
- 全局变量是在 shell 脚本内任何地方都有效的变量。如果在脚本的主体部分定义了一个全局变量，那么就可以在函数内读取它的值。类似地，如果在函数内定义了一个全局变量，那么也可以在脚本的主体部分读取它的值。
- 在默认情况下，在脚本中定义的任何变量都是全局变量。在函数外定义的变量可在函数内正常访问。
- 无须在函数中使用全局变量，任何在函数内部使用的变量都可以被声明为局部变量。为此，只需在变量声明之前加上 local 关键字即可：  
    ```
    local temp
    ```
- 也可以在变量赋值语句中使用 local 关键字：
    ```
    local temp=$[ $value + 5 ]
    ```
- local 关键字保证了变量仅在该函数中有效。如果函数之外有同名变量，那么 shell 会保持这两个变量的值互不干扰。

11. 向函数传递数组
- 将数组变量当作单个参数传递的话，它不会起作用：
    ```
    function testit {
        echo "The parameters are: $@"
        thisarray=$1
        echo "The received array is ${thisarray[*]}"
    }
    myarray=(1 2 3 4 5)
    echo "The original array is: ${myarray[*]}"
    testit $myarray
    $
    $ ./badtest3
    The original array is: 1 2 3 4 5
    The parameters are: 1
    The received array is 1
    ```
    - 如果试图将数组变量作为函数参数进行传递，则函数只会提取数组变量的第一个元素。
- 要解决这个问题，必须先将数组变量拆解成多个数组元素，然后将这些数组元素作为函数参数传递。最后在函数内部，将所有的参数重新组合成一个新的数组变量。
    ```
    function testit {
        local newarray
        newarray=(`echo "$@"`)
        echo "The new array value is ${newarray[*]}"
        
        local sum=0
        for value in ${newarray[*]}
        do
            sum=$[ $sum + $value ]
        done
        echo $sum
    }
    myarray=(1 2 3 4 5)
    echo "The original array is: ${myarray[*]}"
    testit ${myarray[*]}
    ```
    - 该脚本用$myarray 变量保存所有的数组元素，然后将其作为参数传递给函数。该函数随后根据参数重建数组变量。

12. 从函数返回数组
- 函数向 shell 脚本返回数组变量也采用类似的方法。函数先用 echo 语句按正确顺序输出数组的各个元素，然后脚本再将数组元素重组成一个新的数组变量：
    ```
    function arraydblr {
        local origarray
        local newarray
        local elements
        local i
        origarray=($(echo "$@"))
        newarray=($(echo "$@"))
        elements=$[ $# - 1 ]
        for (( i = 0; i <= $elements; i++ ))
        {
            newarray[$i]=$[ ${origarray[$i]} * 2 ]
        }
        echo ${newarray[*]}
    }
    
    myarray=(1 2 3 4 5)
    echo "The original array is: ${myarray[*]}"
    arg1=$(echo ${myarray[*]})
    result=($(arraydblr $arg1))
    echo "The new array is: ${result[*]}"
    ```
    - 该脚本通过$arg1 变量将数组元素作为参数传给 arraydblr 函数。 arraydblr 函数将传入的参数重组成新的数组变量，生成该数组变量的副本。然后对数据元素进行遍历，将每个元素的值翻倍，并将结果存入函数中的数组变量副本。
    - arraydblr 函数使用 echo 语句输出每个数组元素的值。脚本用 arraydblr 函数的输出重组了一个新的数组变量。

13. 函数递归
- 递归函数通常有一个最终可以迭代到的基准值。
- 阶乘函数用其自身计算阶乘的值：
    ```
    function factorial {
        if [ $1 -eq 1 ]
        then
            echo 1
        else
            local temp=$[ $1 - 1 ]
            local result=$(factorial $temp)
            echo $[ $result * $1 ]
        fi
    }
    
    read -p "Enter value: " value
    result=$(factorial $value)
    echo "The factorial of $value is: $result"
    ```

14. 创建库
- bash shell 允许创建函数库文件，然后在多个脚本中引用此库文件。
- 这个过程的第一步是创建一个包含脚本中所需函数的公用库文件。来看一个库文件 myfuncs，其中定义了一个简单的函数：
    ```
    $ cat myfuncs
    # my script functions
    
    function addem {
    echo $[ $1 + $2 ]
    }
    $
    ```
- 第二步是在需要用到这些函数的脚本文件中包含 myfuncs 库文件。
- 和环境变量一样，shell 函数仅在定义它的shell 会话内有效。 如果在 shell 命令行界面运行 myfuncs 脚本， 那么 shell 会创建一个新的 shell 并在其中运行这个脚本。在这种情况下，以上函数会定义在新 shell 中，当你运行另一个要用到这函数的脚本时，它是无法使用的。
- 使用函数库的关键在于 source 命令。 source 命令会在当前 shell 的上下文中执行命令， 而不是创建新的 shell 并在其中执行命令。可以用 source 命令在脚本中运行库文件。这样脚本就可以使用库中的函数了。
- source 命令有个别名，称作点号操作符。要在 shell 脚本中运行 myfuncs 库文件， 只需添加下面这一行代码：
    ```
    . ./myfuncs
    ```
- 假定 myfuncs 库文件和 shell 脚本位于同一目录。如果不是，则需要使用正确路径 访问该文件。来看一个使用 myfuncs 库文件创建脚本的例子：
    ```
    $ cat test14
    #!/bin/bash
    # using functions defined in a library file
    . ./myfuncs
    
    value1=10
    value2=5
    result1=$(addem $value1 $value2)
    echo "The result of adding them is: $result1"
    $
    $ ./test14
    The result of adding them is: 15
    ```

15. 在命令行中使用函数
- 就像在 shell 脚本中将脚本函数当作命令使用一样，在命令行界面中也可以这样做。这个特性很不错，因为一旦在 shell 中定义了函数，就可以在整个系统的任意目录中使用它，而无须担心该函数是否位于 PATH 环境变量中。

16. 在命令行中创建函数
- 因为 shell 会解释用户输入的命令，所以可以在命令行中直接定义一个函数。有两种方法：
    - 一种方法是采用单行方式来定义函数：
        ```
        $ function divem { echo $[ $1 / $2 ];  }
        $ divem 100 5
        20
        ```
    - 当你在命令行中定义函数时，必须在每个命令后面加个分号，这样 shell 就能知道哪里是命令的起止了：
        ```
        $ function doubleit { read -p "Enter value: " value; echo $[ $value * 2 ]; }
        $
        $ doubleit
        Enter value: 20
        40
        ```
    - 另一种方法是采用多行方式来定义函数。在定义时，bash shell 会使用次提示符来提示输入更多命令。使用这种方法，无须在每条命令的末尾放置分号，只需按下回车键即可：
        ```
        $ function multem {
        > echo $[ $1 * $2 ]
        > }
        $ multem 2 5
        10
        ```
        - 输入函数尾部的花括号后， shell 就知道你已经完成函数的定义了。
- 在命令行创建函数时要特别小心。如果给函数起了一个跟内建命令或另一个命令相同的 名字，那么函数就会覆盖原来的命令。

17. 在.bashrc 文件中定义函数
- 在命令行中直接定义 shell 函数的一个明显缺点是，在退出shell 时，函数也会消失。
- 有一种非常简单的方法可以解决这个问题：将函数定义在每次新 shell 启动时都会重新读取该函数的地方。
- .bashrc 文件就是最佳位置。不管是交互式 shell 还是从现有 shell 启动的新 shell，bash shell 在每次启动时都会在用户主目录中查找这个文件。
- 方式一：直接定义函数。
    - 可以直接在用户主目录的.bashrc 文件中定义函数。 大多数 Linux 发行版已经在该文件中定义了部分内容，注意不要误删，只需将函数放在文件末尾即可。定义好函数后，该函数会在下次启动新的 bash shell 时生效。随后你就能在系统中的任意地方使用这个函数了。
- 方式二：源引函数文件
    - 只要是在 shell 脚本中，就可以用 source 命令（或者其别名，即点号操作符） 将库文件中的函数添加到.bashrc脚本中。
    - 要确保库文件的路径名正确，以便 bash shell 找到该文件。下次启动shell 时， 库中的所有函数都可以在命令行界面使用了。
- shell 还会将定义好的函数传给子 shell 进程，这样一来，这些函数就能够自动用 于该 shell 会话中的任何 shell 脚本了。

18. 实战演练
- 可以下载各种 shell 脚本函数并将其用于自己的应用程序中。
- shtool 库提供了一些简单的 shell 脚本函数，可用于实现日常的shell 功能，比如处理临时文件和目录、格式化输出显示等。
- 下载及安装
    ```
    shtool 软件包的下载地址如下：
    ftp://ftp.gnu.org/gnu/shtool/shtool-2.0.8.tar.gz
    tar -zxvf shtool-2.0.8.tar.gz
    cd shtool-2.0.8
    ```
- 接下来就可以构建 shell 脚本库文件了。shtool 文件必须针对特定的 Linux 环境进行配置。配置过程必须使用标准的 configure 命令和 make 命令，这两个命令常用于 C 编程环境。要构建库文件，只需输入如下内容即可：
    ```
    $ ./configure
    $ make
    ```
    - configure 命令会检查构建 shtool 库文件所必需的软件。一旦发现了所需要的工具，就会使用工具路径修改配置文件。
    - make 命令负责构建 shtool 库文件。最终的结果文件（shtool）是一个完整的库软件包文件。
- 也可以使用 make 命令测试这个库文件：
    ```
    $ make test
    Running test suite:
    echo...........ok
    mdate..........ok
    ...
    OK: passed: 19/19
    ```
    - 测试模式会测试 shtool 库中所有的函数。如果全部通过了测试，就可以将库安装到 Linux 系统中的公用位置，这样你的所有脚本就都能使用这个库了。
- 要完成安装，可以使用 make 命令的 install 选项。不过需要以 root 用户的身份运行该命令：
    ```
    # make install
    ```
    - 完成后，就可以在自己的 shell 脚本中使用这些函数了。
- shtool 库提供了大量方便的函数。
    函数 | 描述
    ---|---
    arx         | 创建归档文件（包含一些扩展功能）
    echo        | 显示字符串，并提供了一些扩展构件
    fixperm     | 改变目录树中的文件权限
    install     | 安装脚本或文件
    mdate       | 显示文件或目录的修改时间
    mkdir       | 创建一个或多个目录
    mkln        | 使用相对路径创建链接
    mkshadow    | 创建一棵阴影树（shadow tree）
    move        | 带有替换功能的文件移动
    path        | 处理程序路径
    platform    | 显示平台标识
    prop        | 显示一个带有动画效果的进度条
    rotate      | 轮替（rotate）日志文件
    scpp        | 共享的 C 预处理器
    slo         | 根据库的类别， 分离出链接器选项
    subst       | 使用 sed 的替换操作
    table       | 以表格的形式显示由字段分隔（field-separated）的数据
    tarball     | 从文件和目录中创建 tar 文件
    version     | 创建版本信息文件
    
    - 每个 shtool 函数都包含大量的选项和参数，可以用来调整函数的工作方式。 shtool 函数使用 格式如下。
        ```
        shtool [options] [function [options] [args]]
        ```
- 可以在命令行或 shell 脚本中直接使用 shtool 函数。下面是一个在 shell 脚本中使用platform 函数的例子：
    ```
    $ cat test16
    #!/bin/bash
    shtool platform
    $ ./test16
    Ubuntu 20.04 (AMD64)
    ```
    - platform 函数会返回 Linux 发行版以及系统所使用 CPU 硬件的相关信息。
- prop 函数使用\、 |、/和-字符创建了一个旋转的进度条，可以告诉 shell 脚本用户目前正在进行一些后台处理工作。要使用 prop 函数，只需将希望监看的输出管接到 shtool 脚本即可：
    ```
    $ ls –al /usr/bin | shtool prop –p "waiting..."
    waiting..
    ```
    - prop 函数会在处理过程中不停地变换进度条字符。
    - 在本例中，输出信息来自 ls 命令。能看到多少进度条取决于 CPU 能以多快的速度列出/usr/bin 目录中的文件。-p 选项可以设置出现在进度条之前的文本。

---

###### 十八、图形化桌面环境中的脚本编程

1. 创建交互式 shell 脚本最常用的方法是使用菜单。各种菜单项有助于用户了解脚本能做什么以及不能做什么。
- 菜单式脚本通常会清空显示区域，然后显示可用的菜单项列表。用户可以按下与每个菜单项关联的字母或数字来选择相应的选项。
- shell 脚本菜单的核心是 case 命令。case 命令会根据用户在菜单上的选择来执行相应的命令。

2. 创建菜单布局
- 创建菜单的第一步显然是确定在菜单上显示的元素以及想要显示的布局方式。
- 在创建菜单前， 最好先清除屏幕上已有的内容。
- clear 命令使用终端会话的终端设置信息来清除屏幕上的文本。运行 clear 命令之后，可以用 echo 命令来显示菜单。clear 命令会查看由环境变量 TERM 给出的终端类型，然后在 terminfo 数据库中确定如何清除屏幕上的内容，参见 man clear。
- 在默认情况下， echo 命令只显示可打印文本字符。在创建菜单项时，非可打印字符（比如 制表符和换行符）往往也能派上用场。要在 echo 命令中包含这些字符，必须加入-e 选项。因此，下列命令：
    ```
    echo -e "1.\tDisplay disk space"
    ```
    - 会生成如下输出行：
        ```
        1.        Display disk space
        ```
- -en 选项会去掉结尾的换行符。
    ```
    echo -en "\t\tEnter option: "
    ```
- 创建菜单的最后一步是获取用户输入。这要用到 read 命令。因为我们期望的是单字符输入，所以会在 read 命令中使用-n 选项来限制只读取一个字符。这样用户只需要输入一个数字，而不用按 Enter 键：
    ```
    read -n 1 option
    ```

3. 创建菜单函数
- 创建 shell 菜单脚本的第一步是确定脚本要实现的功能，然后将这些功能以函数的形式放在代码中。
- 通常我们会为还没有实现的函数创建一个桩函数（stub function）。桩函数既可以是一个空函 数，也可以只包含一个 echo 语句，用于说明最终这里需要什么内容：
    ```
    function diskspace {
        clear
        echo "This is where the diskspace commands will go"
    }
    ```
    - 这个桩函数允许在实现某个函数的同时，菜单仍能正常操作。无须等到写出所有函数之后才让菜单投入使用。
    - 函数从 clear 命令开始。这是为了能在一个干净的屏幕上执行函数，不让它受到原先菜单的干扰。
- 将菜单布局本身作为一个函数来创建，有助于制作 shell 脚本菜单：
    ```
    function menu {
        clear
        echo
        echo -e "\t\t\tSys Admin Menu\n"
        echo -e "\t1. Display disk space"
        echo -e "\t2. Display logged on users"
        echo -e "\t3. Display memory usage"
        echo -e "\t0. Exit program\n\n"
        echo -en "\t\tEnter option: "
        read -n 1 option
    }
    ```
    - 任何时候你都能调用 menu 函数来重现菜单。

4. 添加菜单逻辑
- case 命令应该根据用户输入的字符来调用相应的函数。使用默认的case 命令字符（星号）来处理所有不正确的菜单项是一种不错的做法。
- 下面的代码展示了菜单中 case 命令的典型用法：
    ```
    menu
    case $option in
    0)
        break ;;
    1)
        diskspace ;;
    2)
        whoseon ;;
    3)
        memusage ;;
    *)
        clear
        echo "Sorry, wrong selection";;
    esac
    ```
    - 这段代码首先用 menu 函数清除屏幕并显示菜单。 menu 函数中的 read 命令会一直等待，直到用户在键盘上输入了字符。然后， case 命令就会接管余下的处理过程。 case 命令会基于用户输入的字符调用相应的函数。函数运行结束后， case 命令退出。

5. 整合 shell 脚本菜单
- 一个完整的菜单脚本示例：
    ```
    $ cat menu1
    #!/bin/bash
    # simple script menu
    
    function diskspace {
        clear
        df -k
    }
    
    function whoseon {
        clear
        who
    }
    
    function memusage {
        clear
        cat /proc/meminfo
    }
    
    function menu {
        clear
        echo
        echo -e "\t\t\tSys Admin Menu\n"
        echo -e "\t1. Display disk space"
        echo -e "\t2. Display logged on users"
        echo -e "\t3. Display memory usage"
        echo -e "\t0. Exit program\n\n"
        echo -en "\t\tEnter option: "
        read -n 1 option
    }
    
    while [ 1 ]
    do
        menu
        case $option in
        0)
            break ;;
        1)
            diskspace ;;
        2)
            whoseon ;;
        3)
            memusage ;;
        *)
            clear
            echo "Sorry, wrong selection";;
        esac
        echo -en "\n\n\t\t\tHit any key to continue"
        read -n 1 line
    done
    clear
    $
    ```
    - 这个菜单创建了 3 个函数，以使用常见命令提取 Linux 系统的管理信息。 while 循环用于持续处理菜单，除非用户选择了选项 0，即通过 break 命令跳出while 循环。
    - 可以用这个模板创建任何 shell 脚本的菜单界面。它提供了一种与用户交互的简单途径。

6. 使用 select 命令
- select 命令只需要一个命令就可以创建出菜单， 然后获取输入并自动处理。 select 命令的格式如下：
    ```
    select variable in list
    do
        commands
    done
    ```
    - list 参数是由空格分隔的菜单项列表， 该列表构成了整个菜单。 select 命令会将每个列 表项显示成一个带编号的菜单项，然后显示一个由 PS3 环境变量定义的特殊提示符，指示用户做出选择。
- 一个 select 命令的简单示例：
    ```
    PS3="Enter option: "
    select option in "Display disk space" "Display logged on users" "Display memory usage" "Exit program"
    do
    case $option in
        "Exit program")
            break ;;
        "Display disk space")
            diskspace ;;
        "Display logged on users")
            whoseon ;;
        "Display memory usage")
            memusage ;;
        *)
            clear
            echo "Sorry, wrong selection";;
    esac
    done
    clear
    ```
    - select 语句中的所有内容必须作为一行出现。
    - 运行该脚本会自动生成如下菜单：
        ```
        $ ./smenu1
        1) Display disk space       3) Display memory usage
        2) Display logged on users  4) Exit program
        Enter option:
        ```
- 在使用 select 命令时， 存储在指定变量中的值是整个字符串，而不是跟菜单选项相 关联的数字。字符串才是要在 case 语句中进行比较的内容。

7. 创建文本窗口部件
- dialog 软件包最早是 Savio Lam编写的一款小巧的工具，现在由 Thomas E. Dickey 负责维护。 dialog 能够用 ANSI 转义控制字符，在文本环境中创建标准的窗口对话框。你可以轻而易举地将这些对话框融入自己的 shell 脚本中，以实现与用户的交互。
- 并不是所有的Linux 发行版中都会默认安装dialog 软件包。
    - 在 Ubuntu Linux 发行版中，使用下列命令安装该软件包：
        ```
        sudo apt-get install dialog
        ```
    - 基于 Red Hat 的发行版（比如 CentOS ）系统，使用 dnf 命令安装该软件包：
        ```
        sudo dnf install dialog
        ```
    - 软件包管理器会为你安装 dialog 软件包以及所需的库。

8. dialog 软件包

略。

9. dialog 选项

略。

10. 在脚本中使用 dialog 命令

略。

11. 图形化窗口部件

略。

12. KDE 环境

略。

13. GNOME 环境

略。

14. 实战演练

略。

---

###### 十九、初识 sed 和 gawk

1. sed 编辑器
- sed 编辑器被称作流编辑器（stream editor），与普通的交互式文本编辑器截然不同。在交互式文本编辑器（比如 Vim）中， 可以用键盘命令交互式地插入、删除或替换文本数据。流编辑器则是根据事先设计好的一组规则编辑数据流。
- sed 编辑器根据命令来处理数据流中的数据，这些命令要么从命令行中输入，要么保存在命令文本文件中。 sed 编辑器可以执行下列操作：
    - (1) 从输入中读取一行数据。
    - (2) 根据所提供的编辑器命令匹配数据。
    - (3) 按照命令修改数据流中的数据。
    - (4) 将新的数据输出到 STDOUT。
- 在流编辑器匹配并针对一行数据执行所有命令之后，会读取下一行数据并重复这个过程。在流编辑器处理完数据流中的所有行后，就结束运行。由于命令是按顺序逐行执行的，因此 sed 编辑器只需对数据流处理一遍（one pass through）即可完成编辑操作。
- sed 命令的格式如下：
    ```
    sed options script file
    ```
    - options 参数允许修改 sed 命令的行为。
    - script 参数指定了应用于流数据中的单个命令。如果需要多个命令，则要么使用-e 选项在命令行中指定，要么使用-f 选项在单独的文件中指定。
- sed 命令选项：
    选项 | 描述
    ---|---
    -e commands | 在处理输入时，加入额外的 sed 命令
    -f file     | 在处理输入时，将 file 中指定的命令添加到已有的命令中 
    -n          | 不产生命令输出，使用 p（print）命令完成输出

- 在命令行中定义编辑器命令
    - 在默认情况下， sed 编辑器会将指定的命令应用于 STDIN 输入流中。因此， 可以直接将数据通过管道传入 sed 编辑器进行处理。
        ```
        $ echo "This is a test" | sed 's/test/big test/'
        This is a big test
        
        $ sed 's/dog/cat/' data1.txt
        ```
        - 这个例子在 sed 编辑器中使用了替换（s）命令。替换命令会用斜线间指定的第二个字符串 替换第一个字符串。
        - 这个简单的测试只修改了一行数据。不过就算编辑整个文件，速度也差不了多少。
        - 在处理每行数据的同时，结果也随之显现。
    - **sed 编辑器并不会修改文本文件的数据。它只是将修改后的数据发送到 STDOUT。**

- 在命令行中使用多个编辑器命令
    - 如果要在 sed 命令行中执行多个命令， 可以使用-e 选项：
        ```
        $ sed -e 's/brown/red/; s/dog/cat/' data1.txt
        ```
        - 两个命令都应用于文件的每一行数据。命令之间必须以分号（;）分隔，并且在命令末尾和分号之间不能出现空格。
    - 如果不想用分号，那么也可以用 bash shell 中的次提示符来分隔命令。只要输入第一个单引号标示出 sed 程序脚本（也称作sed 编辑器命令列表）的起始， bash 就会提示继续输入命令，直 到输入了标示结束的单引号： 
        ```
        $ sed -e '
        > s/brown/green/
        > s/fox/toad/
        > s/dog/cat/' data1.txt
        The quick green toad jumps over the lazy cat.
        The quick green toad jumps over the lazy cat.
        The quick green toad jumps over the lazy cat.
        The quick green toad jumps over the lazy cat.
        $
        ```
        - **必须记住，要在闭合单引号所在行结束命令。** bash shell 一旦发现了闭合单引号，就会执行命令。
        - sed 命令会将你指定的所有命令应用于文本文件中的每一行。

- 从文件中读取编辑器命令
    - 可以在 sed 命令中用-f 选项来指定文件：
        ```
        $ cat script1.sed
        s/brown/green/
        s/fox/toad/
        s/dog/cat/
        $ sed -f script1.sed data1.txt
        ```
    - 在这种情况下， 不用在每条命令后面加分号。 sed 编辑器知道每一行都是一条单独的命令。
    - 和在命令行输入命令一样， sed 编辑器会从指定文件中读取命令并应用于文件中的每一行。
    - sed 编辑器脚本文件容易与bash shell 脚本文件混淆。为了避免这种情况， 可以使用.sed 作为 sed 脚本文件的扩展名。

2. gawk 编辑器
- 并不是所有的发行版中都默认安装了gawk。
- gawk 是 Unix 中最初的 awk 的 GNU 版本。 gawk 比 sed 的流编辑提升了一个“段位”，它提供了一种编程语言，而不仅仅是编辑器命令。在 gawk 编程语言中，可以实现以下操作：
    - 定义变量来保存数据。
    - 使用算术和字符串运算符来处理数据。
    - 使用结构化编程概念（比如 if-then 语句和循环）为数据处理添加处理逻辑。
    - 提取文件中的数据将其重新排列组合，最后生成格式化报告。
- gawk 的报告生成能力多用于从大文本文件中提取数据并将其格式化成可读性报告。最完美的应用案例是格式化日志文件。在日志文件中找出错误行可不是一件容易事。 gawk 能够从日志文件中过滤出所需的数据，将其格式化，以便让重要的数据更易于阅读。
- gawk 的基本格式如下。
    ```
    gawk options program file
    ```
- gawk 选项
    选项 | 描述
    ---|---
    -F fs           | 指定行中划分数据字段的字段分隔符
    -f file         | 从指定文件中读取 gawk 脚本代码
    -v var=value    | 定义 gawk 脚本中的变量及其默认值
    -L [keyword]    | 指定 gawk 的兼容模式或警告级别

- gawk 的强大之处在于脚本。可以编写脚本来读取文本行中的数据，然后对其进行处理并显示，形成各种输出报告。

- 从命令行读取 gawk 脚本
    - gawk 脚本用一对花括号来定义。必须将脚本命令放到一对花括号（{}）之间。如果误把 gawk 脚本放在了圆括号内，就会得到错误消息。
    - 由于 gawk 命令行假定脚本是单个文本字符串，因此还必须将脚本放到单引号中。下面的例子在命令行中指定了一个简单的 gawk 程序脚本：
        ```
        $ gawk '{print "Hello World!"}'
        ```
        - 运行这个命令，什么都不会发生。由于没有在命令行中指定文件名， 因此 gawk 程序会从 STDIN 接收数据。在脚本运行时，它会一直等待来自 STDIN 的文本。
        - 如果你输入一行文本并按下 Enter 键， 则 gawk 会对这行文本执行一遍脚本。和 sed 编辑器一样， gawk 会对数据流中的每一行文本都执行脚本。由于脚本被设为显示一行固定的文本字符串，因此不管在数据流中输入什么文本，你都会得到同样的文本输出。
        - 要终止这个 gawk 程序， 必须表明数据流已经结束了。 bash shell 提供了Ctrl+D 组合键来生成 EOF（end-of-file）字符。使用该组合键可以终止 gawk 程序并返回到命令行界面。

- 使用数据字段变量
    - gawk 的主要特性之一是处理文本文件中的数据。它会自动为每一行的各个数据元素分配一个变量。在默认情况下，gawk 会将下列变量分配给文本行中的数据字段。
        - $0 代表整个文本行。
        - $1 代表文本行中的第一个数据字段。
        - $2 代表文本行中的第二个数据字段。
        - $n 代表文本行中的第 n 个数据字段。
    - 文本行中的数据字段是通过字段分隔符来划分的。在读取一行文本时， gawk 会用预先定义好的字段分隔符划分出各个数据字段。在默认情况下，字段分隔符是任意的空白字符（比如空格或制表符）。
    - 在下面的例子中， gawk 脚本会读取文本文件，只显示第一个数据字段的值：
        ```
        $ cat data2.txt
        One line of test text.
        Two lines of test text.
        Three lines of test text.
        $
        $ gawk '{print $1}' data2.txt
        One
        Two
        Three
        $
        ```
        - 该脚本使用$1 字段变量来显示每行文本的第一个数据字段。
    - 如果要读取的文件采用了其他的字段分隔符，可以通过-F 选项指定：
        ```
        $ gawk -F: '{print $1}' /etc/passwd
        ```
        - 由于/etc/passwd 文件使用冒号 （:）来分隔数据字段，因此要想划出数据字段，就必须在 gawk 选项中将冒号指定为字段分隔符 （-F:）。

- 在脚本中使用多条命令
    - gawk 编程语言允许将多条命令组合成一个常规的脚本。要在命令行指定的脚本中使用多条命令，只需在命令之间加入分号即可：
        ```
        $ echo "My name is Rich" | gawk '{$4="Christine"; print $0}'
        My name is Christine
        $
        ```
        - 第一条命令会为字段变量$4 赋值。第二条命令会打印整个文本行。注意， gawk 在输出中已经将原文本中的第四个数据字段替换成了新值。
    - 也可以用次提示符一次一行地输入脚本命令： 
        ```
        $ gawk '{
        > $4="Christine "
        > print $0 }'
        My name is Rich
        My name is Christine
        ```
        - 在使用了表示起始的前单引号后， bash shell 会使用次提示符来提示输入更多数据。你可以一次一行地添加命令， 直到输入结尾的后单引号。
        - **因为没有在命令行中指定文件名，所以 gawk 程序会从 STDIN 中获取数据。当运行这个脚本的时候，它会等着读取来自 STDIN 的文本。要退 出的话，只需按下 Ctrl+D 组合键表明数据结束即可。**

- 从文件中读取脚本
    - 跟 sed 编辑器一样， gawk 允许将脚本保存在文件中， 然后在命令行中引用脚本：
        ```
        $ cat script2.gawk
        {print $1 "'s home directory is " $6}
        $
        $ gawk -F: -f script2.gawk /etc/passwd
        root's home directory is /root
        ...
        ```
        - script2.gawk 会再次使用 print 命令打印/etc/passwd 文件的主目录数据字段（字段变量$6），以及用户名数据字段（字段变量$1）。
    - 可以在脚本文件中指定多条命令。为此，只需一行写一条命令即可，且无须加分号：
        ```
        $ cat script3.gawk
        {
        text = "'s home directory is "
        print $1 text $6
        }
        $
        ```
        - script3.gawk 脚本定义了变量 text 来保存 print 命令中用到的文本字符串。注意，在 gawk 脚本中，引用变量值时无须像 shell 脚本那样使用美元符号。

- 在处理数据前运行脚本
    - gawk 还允许指定脚本何时运行。在默认情况下， gawk 会从输入中读取一行文本，然后对这一行的数据执行脚本。但有时候，可能需要在处理数据前先运行脚本，比如要为报告创建一个标题。BEGIN 关键字就是用来做这个的。它会强制 gawk 在读取数据前执行 BEGIN 关键字之后指定的脚本：
        ```
        $ gawk 'BEGIN {print "Hello World!"}'
        Hello World!
        $
        ```
        - 这次 print 命令会在读取数据前显示文本。但在显示过文本后，脚本就直接结束了，不等待任何数据。
    - 原因在于 BEGIN 关键字在处理任何数据之前仅应用指定的脚本。如果想使用正常的脚本来 处理数据，则必须用另一个区域来定义脚本：
        ```
        $ cat data3.txt
        Line 1
        Line 2
        Line 3
        $
        $ gawk 'BEGIN {print "The data3 File Contents:"}
        > {print $0}' data3.txt
        The data3 File Contents:
        Line 1
        Line 2
        Line 3
        $
        ```
        - 在 gawk 执行了 BEGIN 脚本后，会用第二段脚本来处理文件数据。这么做时要小心，因为这两段脚本仍会被视为 gawk 命令行中的一个文本字符串，所以需要相应地加上单引号。

- 在处理数据后运行脚本
    - 和 BEGIN 关键字类似， END 关键字允许指定一段脚本， gawk 会在处理完数据后执行这段脚本：
        ```
        $ gawk 'BEGIN {print "The data3 File Contents:"}
        > {print $0}
        > END {print "End of File"}' data3.txt
        The data3 File Contents:
        Line 1
        Line 2
        Line 3
        End of File
        $
        ```
        - gawk 脚本在打印完文件内容后，会执行 END 脚本中的命令。这是在处理完所有正常数据后给报告添加页脚的最佳方法。

- 整合在一起，从一个简单的数据文件中创建一份完整的报告：
    ```
    $ cat script4.gawk
    BEGIN {
    print "The latest list of users and shells"
    print "UserID  \t Shell"
    print "------- \t -------"
    FS=":"
    }
    
    {
    print $1 "       \t "  $7
    }
    
    END {
    print "This concludes the listing"
    }
    $

    ```
    - 其中， BEGIN 脚本用于为报告创建标题。
    - 另外还定义了一个殊变量 FS。这是定义字段分隔符的另一种方法。这样就无须依靠脚本用户通过命令行选项定义字段分隔符了。
    - 下面是这个 gawk 脚本的输出（有部分删节）：
        ```
        $ gawk -f script4.gawk /etc/passwd
        The latest list of users and shells
        UserID      Shell
        --------    -------
        root        /bin/bash
        daemon      /usr/sbin/nologin
        [...]
        christine   /bin/bash
        sshd        /usr/sbin/nologin
        This concludes the listing
        $
        ```
        - 和预想的一样， BEGIN 脚本创建了标题，主体脚本处理了特定数据文件（/etc/passwd）中的信息， END 脚本生成了页脚。 print 命令中的\t 负责生成美观的选项卡式输出（tabbed output）。

3. sed 编辑器基础命令——更多的替换选项
- 替换命令在替换多行中的文本时也能正常工作，但在默认情况下它只替换每行中出现的第一处匹配文本。要想替换每行中所有的匹配文本，必须使用替换标志（substitution flag）。替换标志在替换命令字符串之后设置。
    ```
    s/pattern/replacement/flags
    ```
- 有 4 种可用的替换标志：
    - 数字，指明新文本将替换行中的第几处匹配。
    - g，指明新文本将替换行中所有的匹配。
    - p，指明打印出替换后的行。
    - w file，将替换的结果写入文件。
- 替换标志 p 会打印出包含替换命令中指定匹配模式的文本行。该标志通常和 sed 的-n 选项配合使用：
    ```
    $ cat data5.txt
    This is a test line.
    This is a different line.
    $
    $ sed -n 's/test/trial/p' data5.txt
    This is a trial line.
    $
    ```
    - -n 选项会抑制 sed 编辑器的输出，而替换标志 p 会输出替换后的行。将二者配合使用的结果就是只输出被替换命令修改过的行。
    - 替换标志 w 会产生同样的输出，不过会将输出保存到指定文件中。
- 如果想将/etc/passwd 文件中的bash shell 替换为 C shell，则必须这么做：
    ```
    $ sed 's/\/bin\/bash/\/bin\/csh/' /etc/passwd
    ```
    - 由于正斜线被用作替换命令的分隔符， 因此它在匹配模式和替换文本中出现时， 必须使用反斜线来转义。这很容易造成混乱和错误。
- 为了解决这个问题，sed 编辑器允许选择其他字符作为替换命令的替代分隔符：
    ```
    $ sed 's!/bin/bash!/bin/csh!' /etc/passwd
    ```
    - 在这个例子中，感叹号（!）被用作替换命令的分隔符，这样就更容易阅读和理解其中的路径了。

4. sed 编辑器基础命令——使用地址
- 在默认情况下， 在 sed 编辑器中使用的命令会应用于所有的文本行。如果只想将命令应用于特定的某一行或某些行，则必须使用行寻址。
- 在 sed 编辑器中有两种形式的行寻址：
    - 以数字形式表示的行区间。
    - 匹配行内文本的模式。
    - 以上两种形式使用相同的格式来指定地址：
        ```
        [address]command
        ```
    - 也可以将针对特定地址的多个命令分组：
        ```
        address {
            command1
            command2
            command3
        }
        ```
- sed 编辑器会将指定的各个命令应用于匹配指定地址的文本行。
- 在使用数字形式的行寻址时，可以用行号来引用文本流中的特定行。 sed 编辑器会将文本流中的第一行编号为 1，第二行编号为 2，以此类推。
- 在命令中指定的行地址既可以是单个行号， 也可以是用起始行号、逗号以及结尾行号指定的行区间。例如：
    ```
    $ sed '2s/dog/cat/' data1.txt       // 只修改了地址所指定的第二行的文本。
    $ sed '2,3s/dog/cat/' data1.txt     // 使用了行区间（第2行-第3行）
    $ sed '2,$s/dog/cat/' data1.txt     // 将命令应用于从某行开始到结尾的所有行，可以使用美元符号作为结尾行号
    ```
- sed 编辑器允许指定文本模式来过滤出命令所应用的行，其格式如下：
    ```
    /pattern/command
    ```
    - 必须将指定的模式（patten）放入正斜线内。sed 编辑器会将该命令应用于包含匹配模式的行。
- 例如，如果只想修改用户 rich 的默认 shell，可以使用 sed 命令：
    ```
    $ grep /bin/bash /etc/passwd
    root:x:0:0:root:/root:/bin/bash
    christine:x:1001:1001::/home/christine:/bin/bash
    rich:x:1002:1002::/home/rich:/bin/bash
    $
    $ sed '/rich/s/bash/csh/' /etc/passwd
    root:x:0:0:root:/root:/bin/bash
    daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
    [...]
    christine:x:1001:1001::/home/christine:/bin/bash
    sshd:x:126:65534::/run/sshd:/usr/sbin/nologin
    rich:x:1002:1002::/home/rich:/bin/csh
    $
    ```
    - 该命令只应用于包含匹配模式的行。sed 编辑器在文本模式中引入了正则表达式来创建匹配效果更好的模式。
- 如果需要在单行中执行多条命令，可以用花括号将其组合在一起， sed 编辑器会执行匹配地址中列出的所有命令：
    ```
    $ sed '2{
    > s/fox/toad/
    > s/dog/cat/
    > }' data1.txt
    The quick brown fox jumps over the lazy dog.
    The quick brown toad jumps over the lazy cat.
    The quick brown fox jumps over the lazy dog.
    The quick brown fox jumps over the lazy dog.
    $
    $ sed '3,${
    > s/brown/green/
    > s/fox/toad/
    > s/lazy/sleeping/
    > }' data1.txt
    ```
    - 这两条命令都会应用于该地址。当然，也可以在一组命令前指定行区间。sed 编辑器会将所有命令应用于该区间内的所有行。

5. sed 编辑器基础命令——删除行
- 如果需要删除文本流中的特定行，可以使用删除（d）命令。
- 删除命令很简单， 它会删除匹配指定模式的所有行。使用该命令时要特别小心， 如果忘记加入寻址模式，则流中的所有文本行都会被删除：
    ```
    $ sed 'd' data1.txt
    ```
- 当和指定地址一起使用时，删除命令显然能发挥出最大的功用。可以从数据流中删除特定的文本行，这些文本行要么通过行号指定，要么通过特定行区间指定，要么通过特殊的末行字符指定：
    ```
    $ cat data6.txt
    This is line number 1.
    This is line number 2.
    This is the 3rd line.
    This is the 4th line.
    $ sed '3d' data6.txt
    This is line number 1.
    This is line number 2.
    This is the 4th line.
    $ sed '2,3d' data6.txt
    This is line number 1.
    This is the 4th line.
    $ sed '3,$d' data6.txt
    This is line number 1
    This is line number 2.
    ```
- sed 编辑器的模式匹配特性也适用于删除命令：
    ```
    $ sed '/number 1/d' data6.txt
    This is line number 2.
    This is the 3rd line.
    This is the 4th line.
    $
    ```
    - sed 编辑器会删掉与指定模式相匹配的文本行。
- **sed 编辑器不会修改原始文件。你删除的行只是从 sed 编辑器的输出中消失了。原始文件中仍然包含那些“被删掉”的行。**
- 也可以使用两个文本模式来删除某个区间内的行。但这么做时要小心，你指定的第一个模式会“启用”行删除功能，第二个模式会“关闭”行删除功能，而 sed 编辑器会删除两个指定行之间的所有行（包括指定的行）：
    ```
    $ sed '/1/,/3/d' data6.txt
    This is the 4th line.
    $
    ```
- 只要 sed 编辑器在数据流中匹配到了开始模式，就会启用删除功能，这可能会导致意想不到的结果：
    ```
    $ cat data7.txt
    This is line number 1.
    This is line number 2.
    This is the 3rd line.
    This is the 4th line.
    This is line number 1 again; we want to keep it.
    This is more text we want to keep.
    Last line in the file; we want to keep it.
    $
    $ sed '/1/,/3/d' data7.txt
    This is the 4th line.
    $
    ```
    - 第二个包含数字“1”的行再次触发了删除命令，因为没有找到停止模式，所以数据流中的剩余文本行全部被删除了。

6. sed 编辑器基础命令——插入和附加文本
- sed 编辑器也可以向数据流中插入和附加文本行。这两种操作的区别可能比较费解：
    - 插入（insert）（i）命令会在指定行前增加一行。
    - 附加（append）（a）命令会在指定行后增加一行。
- 两者的费解之处在于格式。这两条命令不能在单个命令行中使用。必须指定是将行插入还是附加到另一行，其格式如下：
    ```
    sed '[address]command\
    new line '
    ```
    - new line 中的文本会出现在你所指定的 sed 编辑器的输出位置。记住， 当使用插入命令时，文本会出现在数据流文本之前；当使用附加命令时，文本会出现在数据流文本之后：
        ```
        $ echo "Test Line 2" | sed 'i\Test Line 1'
        Test Line 1
        Test Line 2
        $
        
        $ echo "Test Line 2" | sed 'a\Test Line 1'
        Test Line 2
        Test Line 1
        $
        ```
- 在命令行界面使用 sed 编辑器时，你会看到次提示符，它会提醒输入新一行的数据。必须在此行完成 sed 编辑器命令。一旦输入表示结尾的后单引号，bash shell 就会执行该命令：
    ```
    $ echo "Test Line 2" | sed 'i\
    > Test Line 1'
    Test Line 1
    Test Line 2
    $
    ```
- 要向数据流内部插入或附加数据，必须用地址告诉 sed 编辑器希望数据出现在什么位置。用这些命令时只能指定一个行地址。使用行号或文本模式都行，但不能用行区间。
- 下面的例子将一个新行插入数据流中第三行之前：
    ```
    $ sed '3i\
    > This is an inserted line.
    > ' data6.txt
    ```
- 如果你有一个多行数据流，想要将新行附加到数据流的末尾，那么只需用代表数据最后一行的美元符号即可：
    ```
    $ sed '$a\
    > This line was added to the end of the file.
    > ' data6.txt
    ```
    - 同样的方法也适用于在数据流的起始位置增加一个新行。这只要在第一行之前插入新行就可以了。
- 要插入或附加多行文本，必须在要插入或附加的每行新文本末尾使用反斜线（\）：
    ```
    $ sed '1i\
    > This is an inserted line.\
    > This is another inserted line.
    > ' data6.txt
    This is an inserted line.
    This is another inserted line.
    This is line number 1.
    This is line number 2.
    This is the 3rd line.
    This is the 4th line.
    $
    ```
    - 指定的两行都会被添加到数据流中。

7. sed 编辑器基础命令——修改行
- 修改（c）命令允许修改数据流中整行文本的内容。它跟插入和附加命令的工作机制一样，必须在 sed 命令中单独指定一行：
    ```
    $ sed '2c\
    > This is a changed line of text.
    > ' data6.txt
    This is line number 1.
    This is a changed line of text.
    This is the 3rd line.
    This is the 4th line.
    $
    ```
- 也可以用文本模式来寻址：
    ```
    $ sed '/3rd line/c\
    > This is a changed line of text.
    > ' data6.txt
    ```
    - 文本模式修改命令会修改所匹配到的任意文本行。
- 可以在修改命令中使用地址区间，但结果未必如愿：
    ```
    $ cat data6.txt
    This is line number 1.
    This is line number 2.
    This is the 3rd line.
    This is the 4th line.
    $
    $ sed '2,3c\
    > This is a changed line of text.
    > ' data6.txt
    This is line number 1.
    This is a changed line of text.
    This is the 4th line.
    $
    ```
    - sed 编辑器会用指定的一行文本替换数据流中的两行文本，而不是逐一修改。

8. sed 编辑器基础命令——转换命令
- 转换（y）命令是唯一可以处理单个字符的 sed 编辑器命令。该命令格式如下所示：
    ```
    [address]y/inchars/outchars/
    ```
    - 转换命令会对 inchars 和 outchars 进行一对一的映射。 inchars 中的第一个字符会被转换为 outchars 中的第一个字符， inchars 中的第二个字符会被转换成 outchars 中的第二个字符。这个映射过程会一直持续到处理完指定字符。如果 inchars 和 outchars 的长度不同，则 sed 编辑器会产生一条错误消息。
- 使用转换命令的简单例子：
    ```
    $ cat data9.txt
    This is line 1.
    This is line 2.
    This is line 3.
    This is line 4.
    This is line 5.
    This is line 1 again.
    This is line 3 again.
    This is the last file line.
    $
    $ sed 'y/123/789/' data9.txt
    This is line 7.
    This is line 8.
    This is line 9.
    This is line 4.
    This is line 5.
    This is line 7 again.
    This is line 9 again.
    This is the last file line.
    $
    ```
    - inchars 中的各个字符都会被替换成 outchars 中相同位置的字符。
- 转换命令是一个全局命令，也就是说，它会对文本行中匹配到的所有指定字符进行转换， 不考虑字符出现的位置：
    ```
    $ echo "Test #1 of try #1." | sed 'y/123/678/'
    Test #6 of try #6.
    ```
    - sed 编辑器转换了在文本行中匹配到的字符 1 的两个实例。你无法对特定位置字符的转换进行限制。

9. sed 编辑器基础命令——再探打印
- 还有 3 个命令也能打印数据流中的信息：
    - 打印（p）命令用于打印文本行。
    - 等号（=）命令用于打印行号。
    - 列出（l）命令用于列出行。
- 和替换命令中的 p 标志类似，打印命令用于打印sed 编辑器输出中的一行：
    ```
    $ echo "this is a test" | sed 'p'
    this is a test
    this is a test
    $
    ```
    - 如果只用这个命令，倒也没什么特别的。它所做的就是打印出已有的数据文本。
- 打印命令最常见的用法是打印包含匹配文本模式的行：    
    ```
    $ cat data6.txt
    This is line number 1.
    This is line number 2.
    This is the 3rd line.
    This is the 4th line.
    $
    $ sed -n '/3rd line/p' data6.txt
    This is the 3rd line.
    $
    ```
    - 在命令行中用-n 选项可以抑制其他行的输出，只打印包含匹配文本模式的行。
- 也可以用它来快速打印数据流中的部分行：
    ```
    $ sed -n '2,3p' data6.txt
    This is line number 2.
    This is the 3rd line.
    $
    ```
- 如果需要在使用替换或修改命令做出改动之前查看相应的行，可以使用打印命令：
    ```
    $ sed -n '/3/{
    > p
    > s/line/test/p
    > }' data6.txt
    This is the 3rd line.
    This is the 3rd test.
    $
    ```
    - sed 编辑器命令会查找包含数字 3 的行，然后执行两条命令。首先，脚本用打印命令打印出原始行；然后用替换命令替换文本并通过 p 标志打印出替换结果。输出同时显示了原始的文本行 和新的文本行。
- 等号命令会打印文本行在数据流中的行号。行号由数据流中的换行符决定。数据流中每出现一个换行符，sed 编辑器就会认为有一行文本结束了：
    ```
    $ sed '=' data1.txt
    1
    The quick brown fox jumps over the lazy dog.
    2
    The quick brown fox jumps over the lazy dog.
    3
    The quick brown fox jumps over the lazy dog.
    4
    The quick brown fox jumps over the lazy dog.
    $
    ```
    - sed 编辑器在实际文本行之前会先打印行号。
- 如果要在数据流中查找特定文本，那么等号命令用起来非常方便：
    ```
    $ cat data7.txt
    This is line number 1.
    This is line number 2.
    This is the 3rd line.
    This is the 4th line.
    This is line number 1 again; we want to keep it.
    This is more text we want to keep.
    Last line in the file; we want to keep it.
    $
    $ sed -n '/text/{
    > =
    > p
    > }' data7.txt
    6
    This is more text we want to keep.
    $
    ```
    - 利用-n 选项，就能让 sed 编辑器只显示包含匹配文本模式的文本行的行号和内容。
- 列出命令可以打印数据流中的文本和不可打印字符。在显示不可打印字符的时候，要么在其八进制值前加一个反斜线，要么使用标准的 C 语言命名规范（用于常见的不可打印字符），比如\t用于代表制表符：
    ```
    $ cat data10.txt
    This    line    contains        tabs.
    This line does contain tabs.
    This line contains an escape character.
    $
    $ sed -n 'l' data10.txt
    This\tline\tcontains\ttabs.$
    This line does contain tabs.$
    This line contains an escape character. \a$
    $
    ```
    - 制表符所在的位置显示为\t。行尾的美元符号表示换行符。
    - 如果数据流包含转义字符，则列出命令会在必要时用八进制值显示，例如，\a 是一个用于产生铃声的转义控制码。
    - 当用 cat 命令显示文本文件时，转义控制码不会显示出来，你只能听到声音（如果打开了音箱的话）。但利用列出命令，就能显示出所使用的转义控制码。

10. sed 编辑器基础命令——使用 sed 处理文件
- 替换命令包含一些文件处理标志。一些常规的 sed 编辑器命令也可以让你无须替换文本即可完成此操作。
- 写入（w）命令用来向文件写入行。该命令格式如下所示：
    ```
    [address]w filename
    ```
    - filename 可以使用相对路径或绝对路径，但不管使用哪种，运行 sed 编辑器的用户都必须有文件的写权限。
    - 地址可以是 sed 支持的任意类型的寻址方式，比如单个行号、文本模式、行区间或文本模式区间。
- 将数据流中的前两行写入文本文件：
    ```
    $ sed '1,2w test.txt' data6.txt
    This is line number 1.
    This is line number 2.
    This is the 3rd line.
    This is the 4th line.
    $
    $ cat test.txt
    This is line number 1.
    This is line number 2.
    $
    ```
    - 如果不想在 STDOUT 中显示文本行，可以使用 sed 命令的-n 选项。
- 如果要根据一些公用的文本值，从主文件（比如下面的邮件列表）中创建一份数据文件， 则使用写入命令会非常方便：
    ```
    $ cat data12.txt
    Blum, R       Browncoat
    McGuiness, A  Alliance
    Bresnahan, C  Browncoat
    Harken, C     Alliance
    $
    $ sed -n '/Browncoat/w Browncoats.txt' data12.txt
    $
    $ cat Browncoats.txt
    Blum, R       Browncoat
    Bresnahan, C  Browncoat
    $
    ```
    - sed 编辑器会将匹配文本模式的数据行写入目标文件。
- 读取（r）命令允许将一条独立文件中的数据插入数据流。
- 读取命令的格式如下所示：
    ```
    [address]r filename
    ```
    - filename 参数指定了数据文件的绝对路径或相对路径。读取命令中无法使用地址区间，只能指定单个行号或文本模式地址。 
- sed 编辑器会将文件内容插入指定地址之后。 
    ```
    $ cat data13.txt
    This is an added line.
    This is a second added line.
    $
    $ sed '3r data13.txt' data6.txt
    This is line number 1.
    This is line number 2.
    This is the 3rd line.
    This is an added line.
    This is a second added line.
    This is the 4th line.
    $
    ```
    - sed 编辑器会将数据文件中的所有文本行都插入数据流。
- 在使用文本模式地址时，同样的方法也适用：
    ```
    $ sed '/number 2/r data13.txt' data6.txt
    ```
- 要在数据流的末尾添加文本，只需使用美元符号地址即可：
    ```
    $ sed '$r data13.txt' data6.txt
    ```
- 读取命令还有一种很酷的用法是和删除命令配合使用，利用另一个文件中的数据来替换文件中的占位文本。假如你保存在文本文件中的套用信件如下所示：
    ```
    $ cat notice.std
    Would the following people:
    LIST
    please report to the ship's captain.
    $
    
    $ sed '/LIST/{
    > r data12.txt
    > d
    > }' notice.std
    Would the following people:
    Blum, R       Browncoat
    McGuiness, A  Alliance
    Bresnahan, C  Browncoat
    Harken, C     Alliance
    please report to the ship's captain.
    $
    ```
    - 套用信件将通用占位文本 LIST 放在了人物名单的位置。要在占位文本后插入名单，只需使用读取命令即可。但这样的话，占位文本仍然会留在输出中。为此，可以用删除命令删除占位文本，其结果见上面。
    - 现在占位文本已经被替换成了数据文件中的名单。

11. 实战演练
- -s 选项可以告知 sed 将目录内的各个文件作为单独的流：
    ```
    $ sed -sn '1s!/bin/sh!/bin/bash!' OldScripts/*.sh
    ```
- sed 命令 F 会告知 sed 打印出当前正在处理的文件名，且不受-n 选项的影响。
    ```
    $ sed -sn '1F;
    > 1s!/bin/sh!/bin/bash!' OldScripts/*.sh
    ```

---

###### 二十、正则表达式

1. 正则表达式是一种可供 Linux 工具过滤文本的自定义模板。 Linux 工具（比如 sed 或 gawk） 会在读取数据时使用正则表达式对数据进行模式匹配。如果数据匹配模式，它就会被接受并进行处理。如果数据不匹配模式，它就会被弃用。

2. 正则表达式模式使用元字符来描述数据流中的一个或多个字符。
- 正则表达式的工作方式与通配符类似。正则表达式包含文本和/或特殊字符(这些特殊字符在正则表达式中称作元字符)，定义了 sed 和 gawk 匹配数据时使用的模板。你可以在正则表达式中使用不同的特殊字符来定义特定的数据过滤模式。
- 准确地说，通配符和正则表达式并不是一回事，虽然正则表达式中也有*和?，但作用和通配符完全不一样。

3. 正则表达式是由正则表达式引擎实现的。这是一种底层软件，负责解释正则表达式并用这些模式进行文本匹配。
- 尽管在 Linux 世界中有很多不同的正则表达式引擎，但最流行的是以下两种。
    - POSIX 基础正则表达式（basic regular expression，BRE）引擎。
    - POSIX 扩展正则表达式（extended regular expression，ERE）引擎。

- 大多数 Linux 工具至少符合 POSIX BRE 引擎规范，能够识别该规范定义的所有模式符号。有些工具（比如 sed）仅符合 BRE 引擎规范的一个子集。这是出于速度方面的考虑导致的，因为 sed 希望尽可能快地处理数据流中的文本。
- POSIX ERE 引擎多见于依赖正则表达式过滤文本的编程语言中。它为常见模式（比如数字、单词以及字母数字字符）提供了高级模式符号和特殊符号。 gawk 使用 ERE 引擎来处理正则表达式。

4. 最基本的 BRE 模式是匹配数据流中的文本字符。

5. 正则表达式并不关心模式在数据流中出现的位置，也不在意模式出现了多少次。只要能匹配文本字符串中任意位置的模式，正则表达式就会将该字符串传回 Linux 工具。

6. 正则表达式区分大小写。

7. 在正则表达式中，无须写出整个单词。只要定义的文本出现在数据流中，正则表达式就能够匹配。

8. 也无须局限于在正则表达式中只使用单个文本单词，空格和数字也是可以的。
    ```
    $ echo "This is line number 1" | sed -n '/ber 1/p'
    This is line number 1
    $
    $ echo "This is line number1" | sed -n '/ber 1/p'
    $
    ```
    - 在正则表达式中，空格和其他的字符没有什么区别。如果在正则表达式中定义了空格， 那么它必须出现在数据流中。
- 甚至可以创建匹配多个连续空格的正则表达式：
    ```
    $ cat data1
    This is a normal line of text.
    This is  a line with too many spaces.
    $ sed -n '/  /p' data1
    This is  a line with too many spaces.
    $
    ```
    - 单词间有两个空格的行匹配了正则表达式模式。这是查找文本文件中空格的好办法。

9. 正则表达式能识别的特殊字符如下所示：
    ```
    .*[]^${}\+?|()
    ```
    - 不能在匹配普通文本的模式中单独使用这些字符即可。
- 如果要将某个特殊字符视为普通字符，则必须使用反斜线（\）将其转义。
- 尽管正斜线（/）不属于正则表达式的特殊字符，但如果它出现在 sed 或 gawk 的正则表达式中，就会出现错误。因此，使用正斜线也需要进行转义。

10. 在默认情况下，当指定一个正则表达式模式时，只要模式出现在数据流中的任何地方，它就能匹配。有两个特殊字符可以用来将模式锁定在数据流中的行首或行尾。

- 锚定行首
    - 脱字符（^）可以指定位于数据流中文本行行首的模式。如果模式出现在行首之外的位置，则正则表达式无法匹配。
    - 要使用脱字符，就必须将其置于正则表达式之前：
        ```
        $ echo "The book store" | sed -n '/^book/p'
        $
        $ echo "Books are great" | sed -n '/^Book/p'
        Books are great
        $
        ```
        - 脱字符使得正则表达式引擎在每行（由换行符界定）的行首检查模式。
        - 只要模式出现在行首，脱字符就能将其锚定。
    - 如果将脱字符放到正则表达式开头之外的位置， 那么它就跟普通字符一样， 没有什么特殊含义了，sed 会将其视为普通字符来匹配。
    - 如果正则表达式模式中只有脱字符，就不必用反斜线来转义。但如果在正则表达式中先指定脱字符，随后还有其他文本，那就必须在脱字符前用转义字符。即，如果只是匹配脱字符，可以不用转义，比如 echo "This ^ is a test" | sed -n '/^/p'。 但如果要匹配脱字符以及其他文本，则需要转义， 比如 echo "I love ^regex" | sed -n '/\^regex/p'。

- 锚定行尾
    - 特殊字符美元符号（$）定义了行尾锚点。将这个特殊字符放在正则表达式之后则表示数据行必须以该模式结尾：
        ```
        $ echo "This is a good book" | sed -n '/book$/p'
        This is a good book
        $ echo "This book is good" | sed -n '/book$/p'
        $
        $ echo "There are a lot of good books" | sed -n '/book$/p'
        $
        ```
        - 要想匹配，文本模式必须是行的最后一部分。

- 组合锚点
    - 可以在同一行中组合使用行首锚点和行尾锚点。
    - 第一种情况是，假定要查找只含有特定文本模式的数据行：
        ```
        $ cat data4
        this is a test of using both anchors
        I said this is a test
        this is a test
        I'm sure this is a test.
        $ sed -n '/^this is a test$/p' data4
        this is a test
        $
        ```
        - sed 忽略了那些不单单包含指定模式的行。
    - 第二种情况，将这两个锚点直接组合在一起，之间不加任何文本，可以过滤出数据流中的空行：
        ```
        $ cat data5
        This is one test line.
    
        This is another test line.
        $ sed '/^$/d' data5
        This is one test line.
        This is another test line.
        $
        ```
        - 指定的正则表达式会查找行首和行尾之间什么都没有的那些行。由于空行在两个换行符之间没有文本，因此刚好匹配正则表达式。
        - sed 用删除命令来删除与该正则表达式匹配的行，因此也就删除了文本中的所有空行。这是从文档中删除空行的一种行之有效的方法。

11. 点号字符
- 点号字符可以匹配除换行符之外的任意单个字符。点号字符必须匹配一个字符，如果在点号字符的位置没有可匹配的字符，那么模式就不成立。

12. 字符组
- 可以在正则表达式中定义用来匹配某个位置的一组字符。如果字符组中的某个字符出现在了数据流中，那就能匹配该模式。
- 方括号用于定义字符组。在方括号中加入你希望出现在该字符组中的所有字符，就可以在正则表达式中像其他特殊字符一样使用字符组了。
- 一个创建字符组的例子：
    ```
    $ sed -n '/[ch]at/p' data6
    The cat is sleeping.
    That is a very nice hat.
    $
    ```
    - 字符组中必须有个字符来匹配相应的位置。
- 在不太确定某个字符的大小写时非常适合使用字符组：
    ```
    $ echo "Yes" | sed -n '/[Yy]es/p'
    ```
- 在单个正则表达式中可以使用多个字符组：
    ```
    $ echo "Yes" | sed -n '/[Yy][Ee][Ss]/p'
    ```
- 字符组并非只能含有字母，也可以在其中使用数字：
    ```
    $ sed -n '/[0123]/p' data7    
    ```
    - 这个正则表达式模式匹配任意含有数字 0 、1、2 或 3 的行。含有其他数字以及不含有数字的行都会被忽略。
- 匹配邮政编码出错的例子：
    ```
    $ cat data8
    60633
    46201
    223001
    4353
    22203
    $ sed -n '
    >/[0123456789][0123456789][0123456789][0123456789][0123456789]/p
    >' data8
    60633
    46201
    223001
    22203
    $
    ```
    - 它成功过滤掉了不可能是邮政编码的那些过短的数字，因为最后一个字符组没有字符可匹配。但其中有一个 6 位数也被正则表达式保留了下来，尽管我们只定义了 5 个字符组。
    - 正则表达式可以匹配数据流中任何位置的文本。
- 匹配模式之外经常会有其他字符。如果要确保只匹配 5 位数，就必须将其与其他字符分开，要么用空格，要么像下面例子中那样，指明要匹配数字的起止位置：
    ```
    $ sed -n '
    > /^[0123456789][0123456789][0123456789][0123456789][0123456789]$/p
    > ' data8
    60633
    46201
    22203
    $
    ```

13. 排除型字符组
- 在正则表达式中，你也可以反转字符组的作用：匹配字符组中没有的字符。为此，只需在字符组的开头添加脱字符即可：
    ```
    $ sed -n '/[^ch]at/p' data6
    This test is at line four.
    $
    ```
    - 通过排除型字符组， 正则表达式会匹配除 c 或 h 之外的任何字符以及文本模式。由于空格字符属于这个范围，因此通过了模式匹配。但即使是排除型，字符组仍必须匹配一个字符，以 at 为起始的行还是不能匹配模式。

14. 区间
- 可以用单连字符在字符组中表示字符区间。只需指定区间的第一个字符、连字符以及区间的最后一个字符即可。根据 Linux 系统使用的字符集（参见第 2 章），字符组会包括在此区间内的任意字符。
- 可以通过指定数字区间来简化邮政编码的例子：
    ```
    $ sed -n '/^[0-9][0-9][0-9][0-9][0-9]$/p' data8
    60633
    46201
    45902
    $
    ```
    - 每个字符组都会匹配 0~9 的任意数字。如果字母出现在数据中的任何位置，则这个模式都不成立。
    - 同样的方法也适用于字母。
- 还可以在单个字符组内指定多个不连续的区间：
    ```
    $ sed -n '/[a-ch-m]at/p' data6
    The cat is sleeping.
    That is a very nice hat.
    $
    $ echo "I'm getting too fat." | sed -n '/[a-ch-m]at/p'
    $
    ```
    - 该字符组允许区间 a~c 和 h~m 中的字母出现在 at 文本前，但不允许出现区间 d~g 中的字母。

15. 特殊的字符组
- 除了定义自己的字符组， BRE 还提供了一些特殊的字符组，以用来匹配特定类型的字符。 
    字符组 | 描述
    ---|---
    [[:alpha:]] | 匹配任意字母字符，无论是大写还是小写
    [[:alnum:]] | 匹配任意字母数字字符， 0~9 、A~Z 或 a~z
    [[:blank:]] | 匹配空格或制表符
    [[:digit:]] | 匹配 0~9 中的数字
    [[:lower:]] | 匹配小写字母字符 a~z
    [[:print:]] | 匹配任意可打印字符
    [[:punct:]] | 匹配标点符号
    [[:space:]] | 匹配任意空白字符：空格、制表符、换行符、分页符（formfeed）、垂直制表符和回车符 
    [[:upper:]] | 匹配任意大写字母字符 A~Z

- 特殊字符组在正则表达式中的用法和普通字符组一样：
    ```
    $ echo "abc" | sed -n '/[[:digit:]]/p'
    $
    $ echo "abc" | sed -n '/[[:alpha:]]/p'
    abc
    $ echo "abc123" | sed -n '/[[:digit:]]/p'
    abc123
    $ echo "This is, a test" | sed -n '/[[:punct:]]/p'
    This is, a test
    $ echo "This is a test" | sed -n '/[[:punct:]]/p'
    $
    ```
    - 使用特殊字符组定义区间更方便，可以用[[:digit:]]来代替区间 [0-9]。

16. 星号
- 在字符后面放置星号表明该字符必须在匹配模式的文本中出现 0 次或多次。
- 如果需要写一个可在美式英语或英式英语中使用的脚本，可以这么做：
    ```
    $ echo "I'm getting a color TV" | sed -n '/colou*r/p'
    I'm getting a color TV
    $ echo "I'm getting a colour TV" | sed -n '/colou*r/p'
    I'm getting a colour TV
    $
    ```
    - 模式中的 u*表示字母 u 可以出现，也可以不出现。
- 另一个方便的特性是将点号字符和星号字符组合起来。这个组合能够匹配任意数量的任意字符，通常用在数据流中两个可能相邻或不相邻的字符串之间：
    ```
    $ echo "this is a regular pattern expression" | sed -n '
    > /regular.*expression/p'
    this is a regular pattern expression
    $
    ```
    - 通过这种模式可以轻松查找可能出现在文本行内任意位置的多个单词。
- 星号还能用于字符组，指定可能在文本中出现 0 次或多次的字符组或字符区间：
    ```
    $ echo "bt" | sed -n '/b[ae]*t/p'
    bt
    $ echo "bat" | sed -n '/b[ae]*t/p'
    bat
    $ echo "bet" | sed -n '/b[ae]*t/p'
    bet
    $ echo "btt" | sed -n '/b[ae]*t/p'
    btt
    $ echo "baat" | sed -n '/b[ae]*t/p'
    baat
    $ echo "baaeeet" | sed -n '/b[ae]*t/p'
    baaeeet
    $ echo "baeeaeeat" | sed -n '/b[ae]*t/p'
    baeeaeeat
    $ echo "baakeeet" | sed -n '/b[ae]*t/p'
    $
    ```
    - 只要 a 和 e 字符以任何组合形式出现在 b和 t 字符之间（完全不出现也行），模式就能够匹配。如果出现了字符组之外的其他字符，那么模式就不能匹配。

17. 扩展正则表达式
- POSIX ERE 模式提供了一些可供 Linux 应用程序和工具使用的额外符号。 gawk 支持 ERE 模式，但 sed 不支持。
- sed 和 gawk 的正则表达式引擎之间是有区别的。 gawk 可以使用大多数扩展的正则表达式符号， 并且能够提供了一些 sed 所不具备的额外过滤功能。但正因如此， gawk 在处理数据时往往比较慢。
- gawk 脚本中的常见 ERE 模式符号：
    - 问号
    - 加号
    - 花括号
    - 竖线符号
    - 表达式分组

18. 问号
- 问号表明前面的字符可以出现 0 次或 1 次，它不会匹配多次出现的该字符。
- 跟星号一样， 可以将问号和字符组一起使用：
    ```
    $ echo "bt" | gawk '/b[ae]?t/{print $0}'
    bt
    ```
    - 如果字符组中的字符出现了 0 次或 1 次，那么模式匹配就成立。但如果两个字符都出现了，或者其中一个字符出现了两次，那么模式匹配就不成立。

19. 加号
- 加号表明前面的字符可以出现 1 次或多次， 但必须至少出现 1 次。如果该字符没有出现，那么模式就不会匹配：
    ```
    $ echo "beeet" | gawk '/be+t/{print $0}'
    beeet
    ```
    - 如果字符 e 没有出现，那么模式匹配就不成立。
- 加号同样适用于字符组， 跟星号和问号的使用方式相同：
    ```
    $ echo "bt" | gawk '/b[ae]+t/{print $0}'
    $
    ```
    - 如果出现了字符组中定义的任一字符，那么文本就会匹配指定的模式。

20. 花括号
- ERE 中的花括号允许为正则表达式指定具体的可重复次数，这通常称为区间。可以用两种格式来指定区间：
    - m：正则表达式恰好出现 m 次。
    - m, n：正则表达式至少出现 m 次，至多出现 n 次。
    - 这个特性可以精确指定字符（或字符组）在模式中具体出现的次数。
- 在默认情况下，gawk 不识别正则表达式区间，必须指定 gawk 的命令行选项--re-interval才行。
- 例子：
    ```
    $ echo "bt" | gawk --re-interval '/be{1}t/{print $0}'
    $
    $ echo "beet" | gawk --re-interval '/be{1,2}t/{print $0}'
    beet
    ```
    - 通过指定区间为 1，限定了该字符应该出现的次数。如果该字符出现多次，那么模式匹配就不成立。
    - 字符 e 出现一次或两次，模式都能匹配；否则，模式无法匹配。 
- 区间也适用于字符组：
    ```
    $ echo "bat" | gawk --re-interval '/b[ae]{1,2}t/{print $0}'
    bat
    ```
    - 如果字母 a 或 e 在文本模式中只出现了 1~2 次，则正则表达式模式匹配；否则，模式匹配失败。

21. 竖线符号
- 竖线符号允许在检查数据流时， 以逻辑 OR 方式指定正则表达式引擎要使用的两个或多个模式。如果其中任何一个模式匹配了数据流文本，就视为匹配。如果没有模式匹配，则匹配失败。
- 竖线符号的使用格式如下：
    ```
    expr1 |expr2|...
    ```
- 例子：
    ```
    $ echo "The cat is asleep" | gawk '/cat|dog/{print $0}'
    The cat is asleep
    $ echo "The dog is asleep" | gawk '/cat|dog/{print $0}'
    The dog is asleep
    $ echo "The sheep is asleep" | gawk '/cat|dog/{print $0}'
    $
    ```
    - 这个例子会在数据流中查找正则表达式 cat 或 dog。
    - 正则表达式和竖线符号之间不能有空格，否则竖线符号会被认为是正则表达式模式的一部分。
- 竖线符号两侧的子表达式可以采用正则表达式可用的任何模式符号（包括字符组）：
    ```
    $ echo "He has a hat." | gawk '/[ch]at|dog/{print $0}'
    He has a hat.
    $
    ```
    - 这个例子会匹配数据流文本中的 cat、hat 或 dog。

22. 表达式分组
- 也可以用圆括号对正则表达式进行分组。分组之后，每一组会被视为一个整体，可以像对普通字符一样对该组应用特殊字符。例如：
    ```
    $ echo "Sat" | gawk '/Sat(urday)?/{print $0}'
    Sat
    $ echo "Saturday" | gawk '/Sat(urday)?/{print $0}'
    Saturday
    $
    ```
    - 结尾的 urday 分组和问号使得该模式能够匹配 Saturday 的全写或 Sat 缩写。
- 将分组和竖线符号结合起来创建可选的模式匹配组是很常见的做法：
    ```
    $ echo "cat" | gawk '/(c|b)a(b|t)/{print $0}'
    cat
    ```
    - 正则表达式(c|b)a(b|t)匹配的模式是第一组中任意字母、a 以及第二组中任意字母的各种组合。

23. 实战演练1——目录文件计数
- 对 PATH 环境变量中各个目录所包含的文件数量进行统计。
- PATH 中的各个路径由冒号分隔。要获取可在脚本中使用的目录列表，必须用空格替换冒号。
- 对于单个目录，可以用 ls 命令列出其中的文件，再用另一个 for 语句来遍历每个文件，对文件计数器增值。
- 这个脚本的最终版本如下：
    ```
    $ cat countfiles
    #!/bin/bash
    # count number of files in your PATH
    mypath=$(echo $PATH | sed 's/:/ /g')
    count=0
    for directory in $mypath
    do
        check=$(ls $directory)
        for item in $check
        do
            count=$[ $count + 1 ]
        done
        echo "$directory - $count"
        count=0
    done
    $ ./countfiles /usr/local/sbin - 0
    /usr/local/bin - 2
    /usr/sbin - 213
    /usr/bin - 1427
    /sbin - 186
    /bin - 152
    /usr/games - 5
    /usr/local/games – 0
    $
    ```

24. 实战演练2——验证电话号码
- 在美国， 电话号码的几种常见形式如下所示：
    ```    
    (123)456-7890
    (123) 456-7890
    123-456-7890
    123.456.7890
    ```
- 完整的正则表达式如下：
    ```
    ^\(?[2-9][0-9]{2}\)?(| |-|\.)[0-9]{3}( |-|\.)[0-9]{4}$
    
    拆解看：^\(?   [2-9]   [0-9]{2}   \)?     (| |-|\.)     [0-9]{3}    ( |-|\.)    [0-9]{4}$
    ```
- 可以在 gawk 中用这个正则表达式过滤掉格式不符的电话号码。现在只需创建一个使用该正则表达式的 gawk 脚本，然后用这个脚本来过滤你的电话簿。记住，在 gawk 中使用正则表达式区间时，必须加入--re-interval命令行选项，否则无法得到正确的结果。脚本如下：
    ```
    $ cat isphone
    #!/bin/bash
    # script to filter out bad phone numbers
    gawk --re-interval '/^\(?[2-9][0-9]{2}\)?(| |-|\.)
    [0-9]{3}( |-|\.)[0-9]{4}/{print $0}'
    $
    $ echo "317-555-1234" | ./isphone
    317-555-1234
    $ cat phonelist
    000-000-0000
    123-456-7890
    ...
    $ cat phonelist | ./isphone
    ```
    - 也可以将含有电话号码的整个文件通过管道传给脚本，过滤掉无效的号码。只有匹配该正则表达式模式的有效电话号码才会出现。

25. 实战演练3——解析 email 地址
- email 地址的基本格式。
    ```
    username@hostname
    ```
    - username 可以包含字母数字字符以及下列特殊字符：
        - 点号
        - 连字符
        - 加号
        - 下划线
    - email 地址的hostname 部分由一个或多个域名和一个服务器名组成。服务器名和域名也必须遵照严格的命名规则，允许包含字母数字字符以及下列特殊字符：
        - 点号
        - 下划线
- 将各部分组合在一起，得到下列正则表达式：
    ```
    ^([a-zA-Z0-9_\-\.\+]+)@([a-zA-Z0-9_\-\.]+)\.([a-zA-Z]{2,5})$
    
    拆解看：^([a-zA-Z0-9_\-\.\+]+)  @   ([a-zA-Z0-9_\-\.]+)     \. ([a-zA-Z]{2,5})$ 
    ```

---

###### 二十一、sed 进阶

1. sed 编辑器提供了 3 个可用于处理多行文本的特殊命令：
    - N：加入数据流中的下一行，创建一个多行组进行处理。
    - D：删除多行组中的一行。
    - P：打印多行组中的一行。

2. next 命令
- 单行 next 命令
    - 单行 next（n）命令会告诉 sed 编辑器移动到数据流中的下一行，不用再返回到命令列表的最开始位置。
    - 记住，通常 sed 编辑器在移动到数据流中的下一行之前，会在当前行中执行完所有定义好的命令，而单行 next 命令改变了这个流程。
    - 例子，删除首行之后的空行， 留下末行之前的空行：
        ```
        $ cat data1.txt
        Header Line
        Data Line #1
        End of Data Lines
        $
        $ sed '/Header/{n ; d}' data1.txt
        Header Line
        Data Line #1
        End of Data Lines
        $
        ```
        - 先用脚本查找含有单词 Header 的那一行，找到之后，单行 next 命令会让 sed 编辑器移动到文本的下一行，也就是我们想删除的空行。
        - 这时，sed 编辑器会继续执行命令列表，即使用删除命令删除空行。sed 编辑器在执行完命令脚本后会读取数据流中下一行文本，并从头开始执行脚本。因为 sed 编辑器再也找不到包含单词 Header 的行了，所以也不会再有其他行被删除。

- 合并文本行
    - 单行 next 命令会将数据流中的下一行移入 sed 编辑器的工作空间（称为模式空间）。多行版本的 next（N）命令则是将下一行添加到模式空间中已有文本之后。
    - 这样的结果是将数据流中的两行文本合并到同一个模式空间中。文本行之间仍然用换行符分隔，但 sed 编辑器现在会将两行文本当成一行来处理。
    - 例子，N 命令的工作方式：
        ```
        $ cat data2.txt
        Header Line
        First Data Line
        Second Data Line
        End of Data Lines
        $
        $ sed '/First/{ N ; s/\n/ / }' data2.txt
        Header Line
        First Data Line Second Data Line
        End of Data Lines
        $
        ```
        - sed 编辑器脚本先查找含有单词 First 的那行文本，找到该行后，使用 N 命令将下一行与该行合并，然后用替换命令将换行符（\n）替换成空格。这样一来，两行文本在 sed 编辑器的输出中就成了一行。
    - 如果要在数据文件中查找一个可能会分散在两行中的文本短语：
        ```
        $ cat data3.txt
        On Tuesday, the Linux System
        Admin group meeting will be held.
        All System Admins should attend.
        Thank you for your cooperation.
        $
        $ sed 's/System Admin/DevOps Engineer/' data3.txt
        On Tuesday, the Linux System
        Admin group meeting will be held.
        All DevOps Engineers should attend.
        Thank you for your cooperation.
        $ 
        $ sed 'N ; s/System.Admin/DevOps Engineer/' data3.txt
        On Tuesday, the Linux DevOps Engineer group meeting will be held.
        All DevOps Engineers should attend.
        Thank you for your cooperation.
        $
        $ sed 'N
        > s/System\nAdmin/DevOps\nEngineer/
        > s/System Admin/DevOps Engineer/
        > ' data3.txt
        On Tuesday, the Linux DevOps
        Engineer group meeting will be held.
        All DevOps Engineers should attend.
        Thank you for your cooperation.
        $
        $ sed '
        > s/System Admin/DevOps Engineer/
        > N
        > s/System\nAdmin/DevOps\nEngineer/
        > ' data4.txt
        ```
        - 替换命令会在文本文件中查找特定的双词短语 System Admin。如果短语是在一行中，那么事情就很好办，替换命令直接就能搞定。但如果短语分散在两行中，那么替换命令就没辙了。
        - 用 N 命令将第一个单词所在行与下一行合并，即使短语内出现了换行，仍然可以查找到该短语。
        - 注意，替换命令在 System 和 Admin 之间用了点号模式（.）来匹配空格和换行符这两种情况。但如果点号匹配的是换行符，则删掉换行符会导致两行被合并成一行。这可能不是你想要的结果。
        - 要解决这个问题， 可以在 sed 编辑器脚本中用两个替换命令，一个用来处理短语出现在多行中的情况，另一个用来处理短语出现在单行中的情况。
        - 第一个替换命令专门查找两个单词间的换行符，并将其放在了替换字符串中。这样就能在新文本的相同位置添加换行符了。但还有个不易察觉的问题。该脚本总是在执行 sed 编辑器命令前将下一行文本读入模式空间，当抵达最后一行文本时，就没有下一行可读了，这时 N 命令会叫停 sed 编辑器。如果要匹配的文本正好在最后一行，那么命令就无法找到要匹配的数据。要解决这个问题，将单行编辑命令放到 N 命令前面，将多行编辑命令放到 N 命令后面。

3. 多行删除命令
- 单行删除命令会在不同的行中查找单词 System 和 Admin，然后在模式空间中将两行都删掉：
    ```
    $ sed 'N ; /System\nAdmin/d' data4.txt
    All System Admins should attend.
    $
    ```
- sed 编辑器提供了多行删除（D）命令，该命令只会删除模式空间中的第一行，即删除该行中的换行符及其之前的所有字符：
    ```
    $ sed 'N ; /System\nAdmin/D' data4.txt
    Admin group meeting will be held.
    All System Admins should attend.
    $
    ```
    - 文本的第二行虽然被 N 命令加入了模式空间，但仍然完好。
    - 如果需要删除目标数据字符串所在行的前一行，那么 D 命令就能派上用场了。
- 例子，删除数据流中出现在第一行之前的空行：
    ```
    $ cat data5.txt
    
    Header Line
    First Data Line
    
    End of Data Lines
    $
    $ sed '/^$/{N ; /Header/D}' data5.txt
    Header Line
    First Data Line
    
    End of Data Lines
    $
    ```
    - sed 编辑器脚本会查找空行，然后用 N 命令将下一行加入模式空间。如果模式空间中含有单词 Header，则 D 命令会删除模式空间中的第一行。如果不结合使用 N 命令和 D 命令，则无法做到在不删除其他空行的情况下只删除第一个空行。

4. 多行打印命令
- 多行打印命令（P）只打印模式空间中的第一行，即打印模式空间中换行符及其之前的所有字符。当用-n 选项来抑制脚本输出时，它和显示文本的单行 p 命令的用法大同小异：
    ```
    $ sed -n 'N ; /System\nAdmin/P' data3.txt
    On Tuesday, the Linux System
    $
    ```
- 当出现多行匹配时，P 命令只打印模式空间中的第一行。该命令的强大之处体现在其和 N 命令及 D 命令配合使用的时候。
- D 命令的独特之处在于其删除模式空间中的第一行之后，会强制 sed 编辑器返回到脚本的起始处，对当前模式空间中的内容重新执行此命令（D 命令不会从数据流中读取新行）。在脚本中加入 N 命令，就能单步扫过（single-step through）整个模式空间，对多行进行匹配。（还是看下面例子理解吧）
- 接下来，先使用 P 命令打印出第一行，然后用 D 命令删除第一行并绕回到脚本的起始处，接着 N 命令会读取下一行文本并重新开始此过程。这个循环会一直持续到数据流结束。（还是看下面例子理解吧）
- 数据文件被破坏了，在一些行的末尾有#，接着在下一行有@。
    ```
    $ cat corruptData.txt
    Header Line#
    @
    Data Line #1
    Data Line #2#
    @
    End of Data Lines#
    @
    $
    $ sed -n '
    > N
    > s/#\n@//
    > P
    > D
    > ' corruptData.txt
    Header Line
    Data Line #1
    Data Line #2
    End of Data Lines
    $
    ```
    - 用 sed 将 Header Line#行载入模式空间，然后用 N 命令载入第二行（@），将其附加到模式空间内的第一行之后。替换命令用空值替换来删除违规数据（#\n@），然后 P 命令只打印模式空间中已经清理过的第一行。 D 命令将第一行从模式空间中删除， 并返回到脚本的开头，下一个 N 命令将第三行（Data Line #1）文本读入模式空间， 继续进行编辑循环。

5. 保留空间
- 模式空间（pattern space ）是一块活跃的缓冲区，在 sed 编辑器执行命令时保存着待检查的文本，但它并不是 sed 编辑器保存文本的唯一空间。
- sed 编辑器还有另一块称作保留空间（hold space）的缓冲区。当你在处理模式空间中的某些行时，可以用保留空间临时保存部分行。
- 与保留空间相关的命令有 5 个，sed 编辑器的保留空间命令如下表：
    命令 | 描述
    ---|---
    h   | 将模式空间复制到保留空间
    H   | 将模式空间附加到保留空间
    g   | 将保留空间复制到模式空间
    G   | 将保留空间附加到模式空间
    x   | 交换模式空间和保留空间的内容

- 这些命令可以将文本从模式空间复制到保留空间，以便清空模式空间，载入其他要处理的字符串。
- 通常， 在使用 h 命令或 H 命令将字符串移入保留空间后，最终还是要用 g 命令、 G 命令或 x 命令将保存的字符串移回模式空间（否则，一开始就不用考虑保存的问题）。
- 例子，如何用 h 命令和 g 命令在缓冲空间之间移动数据：
    ```
    $ cat data2.txt
    Header Line
    First Data Line
    Second Data Line
    End of Data Lines
    $
    $ sed -n '/First/ {
    > h ; p ;
    > n ; p ;
    > g ; p }
    > ' data2.txt
    First Data Line
    Second Data Line
    First Data Line
    $
    ```
    - (1) sed 脚本使用正则表达式作为地址，过滤出含有单词 First 的行。
    - (2) 当出现含有单词 First 的行时， {}中的第一个命令 h 会将该行复制到保留空间。这时，模式空间和保留空间中的内容是一样的。
    - (3) p 命令会打印出模式空间的内容（First Data Line），也就是被复制进保留空间中的那一行。
    - (4) n 命令会提取数据流中的下一行（Second Data Line），将其放入模式空间。现在，模式空间和保留空间的内容就不一样了。
    - (5) p 命令会打印出模式空间的内容（Second Data Line）。
    - (6) g 命令会将保留空间的内容（First Data Line）放回模式空间，替换模式空间中的当前文本。模式空间和保留空间的内容现在又相同了。
    - (7) p 命令会打印出模式空间的当前内容（First Data Line）。
- 通过保留空间来回移动文本行，可以强制 First Data Line 输出在 Second Data Line
之后。如果去掉第一个 p 命令，则可以将这两行以相反的顺序输出：
    ```
    $ sed -n '/First/ {
    > h ;
    > n ; p
    > g ; p }
    > ' data2.txt
    Second Data Line
    First Data Line
    $
    ```
    - 可以用这种方法来创建一个 sed 脚本，反转整个文件的各行文本。

6. 排除命令
- 也可以指示命令不应用于数据流中的特定地址或地址区间。感叹号（!）命令用于排除（negate）命令，也就是让原本会起作用的命令失效。例如，
    ```
    $ sed -n '/Header/!p' data2.txt
    First Data Line
    Second Data Line
    End of Data Lines
    $
    ```
    - 正常的 p 命令只打印data2 文件中包含单词 Header 的那一行。加了感叹号之后，情况反过来了：除了包含单词 Header 的那一行，文件中的其他行都被打印出来了。
- 这种方法可以反转数据流中文本行的先后顺序。要实现这种效果（先显示最后一行， 最后显示第一行）， 需要利用保留空间做一些特别的铺垫工作。为此，可以使用 sed 做以下工作：
    - (1) 在模式空间中放置一行文本。
    - (2) 将模式空间中的文本行复制到保留空间。
    - (3) 在模式空间中放置下一行文本。
    - (4) 将保留空间的内容附加到模式空间。
    - (5) 将模式空间中的所有内容复制到保留空间。
    - (6) 重复执行第(3)~(5)步，直到将所有文本行以反序放入保留空间。
    - (7) 提取并打印文本行。
- 在使用这种方法时，你不想在处理行的时候打印。这意味着要使用 sed 的-n 选项。然后要决定如何将保留空间的文本附加到模式空间的文本之后。这可以用 G 命令完成。不想将保留空间的文本附加到要处理的第一行文本之后。这可以用感叹号命令轻松搞定：1!G。
    ```
    $ cat data2.txt
    Header Line
    First Data Line
    Second Data Line
    End of Data Lines
    $
    $ sed -n '{1!G ; h ; $p }' data2.txt
    End of Data Lines
    Second Data Line
    First Data Line
    Header Line
    $
    ```
    - sed 编辑器脚本和预期一样，输出了反转后的文本文件。这体现了保留空间的强大之处。它提供了一种在脚本输出中控制行顺序的简单方法。
- 有一个现成的 bash shell 命令可以实现同样的效果： tac 命令会以倒序显示文本文件。因为它的功能正好和 cat 命令相反，所以也采用了相反的命令。    

7. 改变执行流程
- 通常， sed 编辑器会从脚本的顶部开始，一直执行到脚本的结尾（D 命令是个例外，它会强制 sed 编辑器在不读取新行的情况下返回到脚本的顶部）。
- sed 编辑器提供了一种方法，可以改变脚本的执行流程，其效果与结构化编程类似。

8. 分支
- sed 编辑器还提供了一种方法，这种方法可以基于地址、地址模式或地址区间排除一整段命令。这允许你只对数据流中的特定行执行部分命令。
- 分支（b）命令的格式如下：
    ```
    [address]b [label]
    ```
    - address 参数决定了哪些行会触发分支命令。
    - label 参数定义了要跳转到的位置。如果没有 label 参数，则跳过触发分支命令的行，继续处理余下的文本行。
- 下面这个例子使用了分支命令的 address 参数，但未指定 label：
    ```
    $ cat data2.txt
    Header Line
    First Data Line
    Second Data Line
    End of Data Lines
    $
    $ sed '{2,3b ;
    > s/Line/Replacement/}
    > ' data2.txt
    Header Replacement
    First Data Line
    Second Data Line
    End of Data Replacements
    $
    ```
    - 分支命令在数据流中的第二行和第三行处跳过了两次替换命令。
- 如果不想跳到脚本末尾，可以定义 label 参数，指定分支命令要跳转到的位置。标签以冒号开始，最多可以有 7 个字符：
    ```
    :label2
    ```
    - 要指定 label，把它放在分支命令之后即可。有了标签，就可以使用其他命令处理匹配分支 address 的那些行。对于其他行，仍然沿用脚本中原先的命令处理：
        ```
        $ sed '{/First/b jump1 ;
        > s/Line/Replacement/
        > :jump1
        > s/Line/Jump Replacement/}
        > ' data2.txt
        Header Replacement
        First Data Jump Replacement
        Second Data Replacement
        End of Data Replacements
        $
        ```
        - 分支命令指定，如果文本行中出现了 First，则程序应该跳到标签为 jump1 的脚本行。如果文本行不匹配分支 address，则 sed 编辑器会继续执行脚本中的命令，包括分支标签 jump1 之后的命令。（因此， 两个替换命令都会被应用于不匹配分支 address 的行。）
        - 如果某行匹配分支 address，那么 sed 编辑器就会跳转到带有分支标签 jump1 的那一行， 因此只有最后一个替换命令会被执行。

- 也可以跳转到靠前的标签，达到循环的效果：
    ```
    $ echo "This, is, a, test, to, remove, commas." |
    > sed -n {'
    > :start
    > s/,//1p
    > /,/b start
    > }'
    ```
    - 脚本的每次迭代都会删除文本中的第一个逗号并打印字符串。
    - 分支命令只会在行中有逗号的情况下跳转。在最后一个逗号被删除后， 分支命令不再执行，脚本正常结束。

9. 测试
- 与分支命令类似，测试（t）命令也可以改变 sed 编辑器脚本的执行流程。测试命令会根据先前替换命令的结果跳转到某个 label 处，而不是根据 address 进行跳转。
- 如果替换命令成功匹配并完成了替换，测试命令就会跳转到指定的标签。如果替换命令未能匹配指定的模式，测试命令就不会跳转。
- 测试命令的格式与分支命令相同：
    ```
    [address]t [label]
    ```
    - 跟分支命令一样，在没有指定 label 的情况下，如果测试成功，sed 会跳转到脚本结尾。
- 测试命令提供了一种低成本的方法来对数据流中的文本执行基本的 if-then 语句。如果需要做二选一的替换操作， 也就是执行这个替换就不执行另一个替换， 那么测试命令可以助你一臂之力（无须指定 label）：
    ```
    $ sed '{s/First/Matched/ ; t
    > s/Line/Replacement/}
    > ' data2.txt
    Header Replacement
    Matched Data Line
    Second Data Replacement
    End of Data Replacements
    $
    ```
    - 第一个替换命令会查找模式文本 First。如果匹配了行中的模式，就替换文本， 而且测试命令会跳过后面的替换命令。如果第一个替换未能匹配，则执行第二个替换命令。

10. . &符号
- &符号可以代表替换命令中的匹配模式。不管模式匹配到的是什么样的文本，都可以使用&符号代表这部分内容。这样就能处理匹配模式的任何单词了：
    ```
    $ echo "The cat sleeps in his hat." |
    > sed 's/.at/"&"/g'
    The "cat" sleeps in his "hat".
    $
    ```
    - 当模式匹配到单词 cat，"cat"就会成为替换后的单词。当模式匹配到单词hat，"hat"就会成为替换后的单词。

11. 替换单独的单词
- sed 编辑器使用圆括号来定义替换模式中的子模式。随后使用特殊的字符组合来引用每个子模式匹配到的文本（在正则表达式中，这称作“反向引用”（back reference））。反向引用由反斜线和数字组成。数字表明子模式的序号，第一个子模式为 \1，第二个子模式为\2，以此类推。
- 在替换命令中使用圆括号时，必须使用转义字符，以此表明这不是普通的圆括号，用于划分子模式。这跟转义其他特殊字符正好相反。
- 一个在 sed 编辑器脚本中使用反向引用的例子：
    ```
    $ echo "The Guide to Programming" |
    > sed '
    > s/\(Guide to\) Programming/\1 DevOps/'
    The Guide to DevOps
    $
    ```
    - 这个替换命令将 Guide To 放入圆括号，将其标示为一个子模式。然后使用\1 来提取该子模式匹配到的文本。
- 如果需要用一个单词来替换一个短语，而这个单词刚好又是该短语的子串，但在子串中用到了特殊的模式字符，那么这时使用子模式会方便很多：
    ```
    $ echo "That furry cat is pretty." |
    > sed 's/furry \(.at\)/\1/'
    That cat is pretty.
    $
    $ echo "That furry hat is pretty." |
    > sed 's/furry \(.at\)/\1/'
    That hat is pretty.
    $
    ```
    - 在这种情况下，不能用&符号，因为它代表的是整个模式所匹配到的文本。而反向引用则允许将某个子模式匹配到的文本作为替换内容。
- 当需要在两个或多个子模式间插入文本时，这个特性尤其有用。下面的脚本使用子模式在大数（long number）中插入逗号：
    ```
    $ echo "1234567" | sed '{
    > :start
    > s/\(.*[0-9]\)\([0-9]\{3\}\)/\1,\2/
    > t start}'
    1,234,567
    $
    ```
    - 这个脚本将匹配模式分成了两个子模式：
        ```
        .*[0-9]
        [0-9]{3}
        ```
    - sed 会在文本行中查找这两个子模式。第一个子模式是以数字结尾的任意长度的字符串。第二个子模式是 3位数字。如果匹配到了相应的模式，就在两者之间加一个逗号，每个子模式都通过其序号来标示。这个脚本使用测试命令来遍历这个大数，直到所有的逗号都插入完毕。

12. 使用包装器
- 可以将 sed 编辑器命令放入 shell脚本包装器，这样就不用每次使用时都重新键入整个脚本。包装器充当着 sed 编辑器脚本和命令行之间的中间人角色。
- 在 shell 脚本中，可以将普通的 shell 变量及命令行参数和 sed 编辑器脚本一起使用。这里有个将位置变量作为 sed 脚本输入的例子：
    ```
    $ cat reverse.sh
    #!/bin/bash
    # Shell wrapper for sed editor script
    # to reverse test file lines.
    #
    sed -n '{1!G; h; $p}' $1
    #
    exit
    $
    $ ./reverse.sh data2.txt
    ```
    - 名为 reverse.sh 的 shell 脚本用 sed 编辑器脚本来反转数据流中的文本行。脚本通过位置变量 $1 获取第一个命令行参数，而这正是要进行反转的文件名。

13. 重定向 sed 的输出
- 在默认情况下， sed 编辑器会将脚本的结果输出到 STDOUT。但你可以在 shell 脚本中通过各种标准方法重定向 sed 编辑器的输出。
- 在 shell 脚本中， 可以用$()将 sed 编辑器命令的输出重定向到一个变量中， 以备后用。下面的例子使用 sed 脚本为数值计算结果添加逗号：
    ```
    $ cat fact.sh
    #!/bin/bash
    # Shell wrapper for sed editor script
    # to calculate a factorial, and
    # format the result with commas.
    #
    factorial=1
    counter=1
    number=$1
    #
    while [ $counter -le $number ]
    do
        factorial=$[ $factorial * $counter ]
        counter=$[ $counter + 1 ]
    done
    #
    result=$(echo $factorial |
    sed '{
    :start
    s/\(.*[0-9]\)\([0-9]\{3\}\)/\1,\2/
    t start
    }')
    #
    echo "The result is $result"
    #
    exit
    $
    $ ./fact.sh 20
    The result is 2,432,902,008,176,640,000
    $
    ```
    - 将阶乘计算的结果作为 sed 编辑器脚本的输入，由后者为其添加逗号，然后使用 echo 输出最终结果。把冗长的 sed 脚本放在 bash shell 脚本中实在太棒了，以后使用的时候就无须一遍遍地重新输入 sed 命令了。

14. 加倍行间距
- 一个向文本文件的行间插入空行的简单 sed脚本：
    ```
    $ sed 'G' data2.txt
    Header Line
    
    First Data Line
    
    Second Data Line
    
    End of Data Lines
    
    $
    ```
    - G 命令只是将保留空间内容附加到模式空间内容之后。当启动 sed 编辑器时， 保留空间只有一个空行。将它附加到已有行之后，就创建出了空行。
- 可以用排除符号（!）和行尾符号（$）来确保脚本不会将空行附加到数据流的最后一行之后：
    ```
    $ sed '$!G' data2.txt
    Header Line
    
    First Data Line
    
    Second Data Line
    
    End of Data Lines
    $
    ```
    - 只要该行不是最后一行，G 命令就会附加保留空间的内容。当 sed 编辑器到最后一行时，它会跳过 G 命令。

15. 对可能含有空行的文件加倍行间距
- 首先删除数据流中的所有空行，然后用 G 命令在每行之后插入新的空行。要删除已有的空行，需要将 d 命令和一个匹配空行的模式一起使用：
    ```
    /^$/d
    ```
    - 这个模式使用了行首锚点（^）和行尾锚点（$）。将这个模式加入脚本就能生成想要的结果：
        ```
        $ sed '/^$/d ; $!G' data6.txt
        Line one.
        Line two.
        Line three.
        Line four.
        $
        ```

16. 给文件中的行编号
- 在获得了等号命令的输出之后，可以通过管道将输出传给另一个 sed 编辑器脚本，由后者使用 N 命令来合并这两行。还需使用替换命令将换行符更换成空格或制表符。最终的解决方法如下所示：
    ```
    $ sed '=' data2.txt | sed 'N; s/\n/ /'
    1 Header Line
    2 First Data Line
    3 Second Data Line
    4 End of Data Lines 
    $
    ```
    - 在查看错误消息的行号时，这是一个很好用的小工具。

17. 打印末尾行
- 美元符号代表数据流中最后一行，因此只显示最后一行很容易：
    ```
    $ sed -n '$p' data2.txt
    End of Data Lines
    $
    ```
- 通过创建滚动窗口（rolling window）来显示数据流末尾的若干行。
- 滚动窗口通过 N 命令将行合并， 是一种检查模式空间中文本行块的常用方法。 N 命令会将下一行文本附加到模式空间中已有文本行之后。一旦模式空间中有了一个包含 10 行的文本块，就可以使用美元符号来检查是否已经处于数据流的尾部。如果不是，就继续向模式空间增加行，同时删除已有的行（记住， D 命令会删除模式空间的第一行）。
- 通过循环 N 命令和 D 命令，你向模式空间的文本行块增加新行的同时也删除了旧行。分支命令非常适合这个循环。要结束循环，只需识别出最后一行并用退出（q）命令退出即可。
- 最终的 sed 编辑器脚本如下所示：
    ```
    $ cat data7.txt
    Line1
    Line2
    Line3
    Line4
    Line5
    Line6
    Line7
    Line1
    Line2
    Line3
    Line4
    Line5
    Line6
    Line7
    Line8
    Line9
    Line10
    Line11
    Line12
    Line13
    Line14
    Line15
    $
    $ sed '{
    > :start
    > $q ; N ; 11,$D
    > b start
    > }' data7.txt
    Line6
    Line7
    Line8
    Line9
    Line10
    Line11
    Line12
    Line13
    Line14
    Line15
    $
    ```
    - 该脚本首先检查当前行是否为数据流中的最后一行。如果是，则退出命令会停止循环， N 命令会将下一行附加到模式空间中的当前行之后。如果当前行在第 10 行之后，则 11,$D 命令会删除模式空间中的第 1 行。这就在模式空间中创造了滑动窗口的效果。因此，这个 sed 程序脚本只会显示 data7.txt 文件最后 10 行。

18. 删除行
- 删除连续的空行
    - 删除连续空行的最简单方法是用地址区间来检查数据流。
    - 删除连续空行的关键在于创建包含一个非空行和一个空行的地址区间。如果 sed 编辑器遇到了这个区间，它不会删除行。但对于不属于该区间的行（两个或更多的空行），则执行删除操作：
        ```
        $ sed '/./,/^$/!d' data8.txt
        ```
        - 指定的区间是/./到/^$/。区间的开始地址会匹配任何至少含有一个字符的行。区间的结束地址会匹配一个空行。在这个区间内的行不会被删除。
        - 不管文件的数据行之间出现了多少空行，在输出中只保留行间的一个空行。

- 删除开头的空行
    ```
    $ sed '/./,$!d' data9.txt  
    ```
    - 这个脚本用地址区间来决定要删除哪些行。这个区间从含有字符的行开始，一直到数据流结束。在这个区间内的任何行都不会从输出中删除。这意味着含有字符的第一行之前的任何行都会被删除。

- 删除结尾的空行
    - 利用循环来实现。
        ```
        $ sed '{
        > :start
        > /^\n*$/{$d; N; b start}
        > }' data10.txt
        ```
        - 花括号内的花括号可以在整个命令脚本中将部分命令分组。命令分组会被应用于指定的地址模式。该地址模式能够匹配只含一个换行符的行。如果找到了这样的行，而且还是最后一行，删除命令就会将它删除。如果不是最后一行，那么 N 命令会将下一行附加到它后面，然后分支命令会跳到循环起始位置重新开始。
    - 在多行模式空间（multiline pattern space）中，^匹配的是整个字符串（其中可能包含换行符）的起始位置，$匹配 的是整个字符串（其中可能包含换行符）的结束位置。

19. 删除 HTML 标签
- 标准的 HTML Web 页面包含各种 HTML 标签，用以标明正确显示页面信息所需要的格式化功能。
- HTML 标签由小于号和大于号来标识。大多数 HTML 标签是成对出现的：一个起始标签（比 如<b>用来加粗）和一个闭合标签（比如</b>用来结束加粗）。
- 让 sed 编辑器忽略任何嵌入原始标签中的大于号。可以使用字符组来排除大于号：
    ```
    $ sed 's/<[^>]*>//g' data11.txt
    $ sed 's/<[^>]*>//g ; /^$/d' data11.txt
    ```
    - 可以加一个 D 命令，删掉多余的空行。

---

###### 二十二、gawk 进阶

1. gawk 编程语言支持两类变量：
    - 内建变量
    - 自定义变量
- gawk 的内建变量包含用于处理数据文件中的数据字段和记录的信息。也可以在 gawk脚本 中创建自己的变量。

2. gawk 脚本使用内建变量来引用一些特殊的功能。
- 字段和记录分隔符变量
    - 数据字段变量允许使用美元符号（$）和字段在记录中的位置值来引用对应的字段。因此，要引用记录中的第一个数据字段，就用变量$1；要引用第二个数据字段，就用$2，以此类推。
    - 数据字段由字段分隔符划定。在默认情况下，字段分隔符是一个空白字符，也就是空格或者制表符。
    - 有一组内建变量可以控制 gawk 对输入数据和输出数据中字段和记录的处理方式。gawk 数据字段和记录变量如下表：
        变量 | 描述
        ---|---
        FIELDWIDTHS | 由空格分隔的一列数字， 定义了每个数据字段的确切宽度
        FS          | 输入字段分隔符
        RS          | 输入记录分隔符
        OFS         | 输出字段分隔符
        ORS         | 输出记录分隔符

        - 变量 FS 和 OFS 定义了 gawk 对数据流中数据字段的处理方式。
    - 变量 OFS 用于 print 命令的输出。在默认情况下， gawk 会将 OFS 变量的值设置为一个空格。
        ```
        命令：print $1,$2,$3
        输出：field1 field2 field3
        ```
    - print 命令会自动将 OFS 变量的值置于输出的每个字段之间。通过设置 OFS 变量，可以在输出中用任意字符串来分隔字段。
    - FIELDWIDTHS 变量可以不通过字段分隔符读取记录。有些应用程序并没有使用字段分隔符，而是将数据放置在记录中的特定列。在这种情况下，必须设定 FIELDWIDTHS 变量来匹配数据在记录中的位置。一旦设置了 FIELDWIDTHS 变量， gawk 就会忽略 FS 变量， 并根据提供的字段宽度来计算字段。
    - 一定要记住，一旦设定了 FIELDWIDTHS 变量的值，就不能再改动了。这种方法并不适用于变长的数据字段。
    - 变量 RS 和 ORS 定义了 gawk 对数据流中记录的处理方式。在默认情况下， gawk 会将 RS 和 ORS 设置为换行符。默认的 RS 值表明，输入数据流中的每行文本就是一条记录。

- 数据变量
    - 更多的 gawk 内建变量
        变量 | 描述
        ---|---
        ARGC        | 命令行参数的数量
        ARGIND      | 当前处理的文件在 ARGV 中的索引
        ARGV        | 包含命令行参数的数组
        CONVFMT     | 数字的转换格式（参见 printf 语句），默认值为%.6g
        ENVIRON     | 当前 shell 环境变量及其值组成的关联数组
        ERRNO       | 当读取或关闭输入文件发生错误时的系统错误号
        FILENAME    | 用作 gawk 输入的数据文件的名称
        FNR         | 当前数据文件中的记录数
        IGNORECASE  | 设成非 0 值时，忽略 gawk 命令中出现的字符串的大小写
        NF          | 数据文件中的字段总数
        NR          | 已处理的输入记录数
        OFMT        | 数字的输出显示格式。默认值为%.6g.，以浮点数或科学计数法显示，以较短者为准，最多使用 6 位小数
        RLENGTH     | 由 match 函数所匹配的子串的长度
        RSTART      | 由 match 函数所匹配的子串的起始位置

    -  gawk 并不会将程序脚本视为命令行参数的一部分：
        ```
        $ gawk 'BEGIN{print ARGC,ARGV[1]}' data1
        2 data1 
        $
        ```
        - ARGC 变量表明命令行上有两个参数。这包括 gawk 命令和 data1 参数（记住， 程序脚本并不算参数）。
        - ARGV 数组从索引 0 开始， 代表的是命令。第一个数组值是 gawk 命令后的第一个命令行参数。
    - 跟 shell 变量不同，在脚本中引用 gawk 变量时，变量名前不用加美元符号。
    - ENVIRON 使用关联数组来提取 shell 环境变量。关联数组用文本（而非数值）作为数组索引。
    - 数组索引中的文本是 shell 环境变量名， 对应的数组元素值是 shell 环境变量的值。
        ```
        $ gawk '
        > BEGIN{
        > print ENVIRON["HOME"]
        > print ENVIRON["PATH"]
        > }'
        /home/rich
        /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
        $
        ```
        - 可以用这种方法从 shell 中提取任何环境变量的值，以供 gawk 脚本使用。
    - NF 变量可以让你在不知道具体位置的情况下引用记录中的最后一个数据字段。NF 变量含有数据文件中最后一个字段的编号。可以在 NF 变量之前加上美元符号，将其用作字段变量。
    - FNR 变量包含当前数据文件中已处理过的记录数， NR 变量则包含已处理过的记录总数。因此，如果只使用一个数据文件作为输入， 那么 FNR 和 NR 的值是相同的； 如果使用多个数据文件作为输入，那么 FNR 的值会在处理每个数据文件时被重置， NR 的值则会继续计数直到处理完所有的数据文件。

3. 自定义变量
- gawk 自定义变量的名称由任意数量的字母、数字和下划线组成，但不能以数字开头。**gawk 变量名区分大小写。**
- 在脚本中给变量赋值
    - 在 gawk 脚本中给变量赋值与给 shell 脚本中的变量赋值一样，都用赋值语句：
        ```
        $ gawk '
        > BEGIN{
        > testing="This is a test"
        > print testing
        > testing=45
        > print testing
        > }'
        This is a test
        45
        $
        ```
        - 跟 shell 脚本变量一样， gawk 变量可以保存数值或文本值。
    - gawk 编程语言也包含了用来处理数值的标准算术运算符，其中包括求余运算符（%） 和幂运算符（^或**）。
    
- 在命令行中给变量赋值
    - 也可以通过 gawk 命令行来为脚本中的变量赋值。这允许你在正常的代码之外赋值，即时修改变量值。
    - 下面的例子使用命令行变量来显示文件中特定的数据字段：
        ```
        $ cat script1
        BEGIN{FS=","}
        {print $n}
        $ gawk -f script1 n=2 data1
        data12
        data22
        data32
        $ gawk -f script1 n=3 data1
        data13
        data23
        data33
        $
        ```
        - 这个特性可以让你在不修改脚本代码的情况下就改变脚本的行为。第一个例子显示了文件的第二个字段，而第二个例子显示了第三个字段，这只需在命令行中设置变量 n 的值即可。
    - 使用命令行参数来定义变量值会产生一个问题：在设置过变量之后，这个值在脚本的 BEGIN 部分不可用：
        ```
        $ cat script2
        BEGIN{print "The starting value is",n; FS=","}
        {print $n}
        $ gawk -f script2 n=3 data1
        The starting value is
        data13
        data23
        data33
        $
        ```
    - 可以用-v 选项来解决这个问题，它允许在 BEGIN 部分之前设定变量。在命令行中， -v 选项必须放在脚本代码之前：
        ```
        $ gawk -v n=3 -f script2 data1
        The starting value is 3
        data13
        data23
        data33
        $
        ```
        - 现在，BEGIN 部分中的变量 n 的值就已经是命令行中设定的那个值了。

4. gawk 编程语言使用关联数组来提供数组功能。
- 与数字型数组 numerical array）不同，关联数组的索引可以是任意文本字符串。你不需要用连续的数字来标识数组元素。相反，关联数组用各种字符串来引用数组元素。每个索引字符串都必须能够唯一地标识出分配给它的数组元素。如果你熟悉其他编程语言，就知道这跟哈希表和字典是同一个概念。

5. 定义数组变量
- 可以用标准赋值语句来定义数组变量。数组变量赋值的格式如下：
    ```
    var[index] = element
    
    例如，capital["Illinois"] = "Springfield"
    ```
    - 其中， var 是变量名， index 是关联数组的索引值， element 是数组元素值。
- 在引用数组变量时，必须包含索引， 以便提取相应的数组元素值：
    ```
    $ gawk 'BEGIN{
    > capital["Illinois"] = "Springfield"
    > print capital["Illinois"]
    > }'
    Springfield
    $
    ```

6. 遍历数组变量
- 如果要在 gawk 脚本中遍历关联数组，可以用 for 语句的一种特殊形式：
    ```
    for (var in array)
    {
        statements
    }
    ```
    - **var 变量中存储的是索引而不是数组元素值。可以将这个变量 用作数组索引，轻松地取出数组元素值。**
    - **注意， 索引值没有特定的返回顺序， 但它们都能够指向对应的数组元素值。不能指望返回值有固定的顺序，只能保证索引值和数据值是对应的。**

7. 删除数组变量
- 从关联数组中删除数组元素要使用一个特殊的命令：
    ```
    delete array[index]
    ```
- delete 命令会从关联数组中删除索引值及其相关的数组元素值：
    ```
    $ gawk 'BEGIN{
    > var["a"] = 1
    > var["g"] = 2
    > for (test in var)
    > {
    >    print "Index:",test," - Value:",var[test]
    > }
    > delete var["g"]
    > print "---"
    > for (test in var)
    >    print "Index:",test," - Value:",var[test]
    > }'
    Index: a  - Value: 1
    Index: g  - Value: 2
    ---
    Index: a  - Value: 1
    $
    ```

8. gawk 脚本支持几种类型的匹配模式来过滤数据记录。
- 关键字 BEGIN 和 END 可以在读取数据流之前或之后执行命令的特殊模式。同样，你可以创建其他模式，在数据流中出现匹配数据时执行命令。

9. 正则表达式
- 在使用正则表达式时， 它必须出现在与其对应脚本的左花括号前：
    ```
    $ gawk 'BEGIN{FS=","} /11/{print $1}' data1
    data11
    $
    ```
    - 正则表达式/11/匹配了数据字段中含有字符串 11 的记录。
- gawk 脚本会用正则表达式对记录中所有的数据字段进行匹配，包括字段分隔符：
    ```
    $ gawk 'BEGIN{FS=","} /,d/{print $1}' data1
    data11
    data21
    data31
    $
    ```
- 如果需要用正则表达式匹配某个特定的数据实例，则应该使用匹配操作符。

10. 匹配操作符
- 匹配操作符（~）能将正则表达式限制在记录的特定数据字段。可以指定匹配操作符、数据字段变量以及要匹配的正则表达式：
    ```
    $1 ~ /^data/
    ```
    - $1 变量代表记录中的第一个数据字段。该表达式会过滤出第一个数据字段以文本 data 开头的所有记录。
- 下面例子演示了在 gawk 脚本中使用匹配操作符的情况：
    ```
    $ gawk 'BEGIN{FS=","} $2 ~ /^data2/{print $0}' data1
    data21,data22,data23,data24,data25
    $
    ```
    - 匹配操作符使用正则表达式/^data2/来比较第二个数据字段，该正则表达式指明这个数据字段要以文本 data2 开头。
- 这可是个强大的工具，gawk 脚本中经常用它在文件中搜索特定的数据元素：
    ```
    $ gawk -F: '$1 ~ /rich/{print $1,$NF}' /etc/passwd
    rich /bin/bash
    $
    ```
    - 这个例子会在第一个数据字段中查找文本 rich。如果匹配该模式，则打印该记录的第一个数据字段和最后一个数据字段。
- 也可以用!符号来排除正则表达式的匹配：
    ```
    $1 !~ /expression/
    ```

11. 数学表达式
- 也可以在匹配模式中使用数学表达式。这个功能在匹配数据字段中的数值时非常方便。
- 如果想显示所有属于 root用户组（组 ID 为 0）的用户，可以使用下列脚本：
    ```
    $ gawk -F: '$4 == 0{print $1}' /etc/passwd
    root
    $
    ```
    - 这段脚本会检查记录中值为 0 的第四个字段。在该 Linux 系统中，只有一个用户账户属于root 用户组。
- 可以使用任何常见的数学比较表达式。
    - x == y：x 的值等于 y 的值。
    - x <= y：x 的值小于等于 y 的值。
    - x < y：x 的值小于 y 的值。
    - x >= y：x 的值大于等于 y 的值。
    - x > y：x 的值大于 y 的值。
- 也可以对文本数据使用表达式，但必须小心。跟正则表达式不同，表达式必须完全匹配。数据必须跟模式严格匹配：
    ```
    $ gawk -F, '$1 == "data"{print $1}' data1
    $ 
    $ gawk -F, '$1 == "data11"{print $1}' data1
    $ data11
    ```

12. gawk 编程语言支持常见的结构化编程命令。

13. if 语句
- gawk 编程语言支持标准格式的 if-then-else 语句。你必须为 if 语句定义一个求值的条件，并将其放入圆括号内。如果条件求值为 TRUE，就执行紧跟在 if 语句后的语句。如果条件求值为 FALSE，则跳过该语句。格式如下所示：
    ```
    if (condition)
        statement1
    
    也可以写在一行中：
    if (condition) statement1
    ```
- 如果需要在 if 语句中执行多条语句，则必须将其放入花括号内。
- gawk 的 if 语句也支持 else 子句，允许在 if 语句条件不成立的情况下执行一条或多条语句。
- 可以在单行中使用 else 子句，但必须在 if 语句部分之后使用分号：
    ```
    if (condition) statement1; else statement2
    ```

14. while 语句
- while 语句为 gawk 脚本提供了基本的循环功能。下面是 while 语句的格式：
    ```
    while (condition)
    {
        statements
    }
    ```
    - while 循环允许遍历一组数据，并检查迭代的结束条件。
- gawk 编程语言支持在 while 循环中使用 break 语句和 continue 语句，允许从循环中跳出。

15. do-while 语句
- do-while 语句与 while 语句类似，但会在检查条件语句之前先执行命令。下面是do-while 语句的格式：
    ```
    do
    {
        statements
    } while (condition)
    ```
    - 这种格式保证了 statements 会在条件被求值之前至少执行一次。

16. for 语句
- gawk 编程语言支持 C 风格的 for 循环：
    ```
    for( variable assignment; condition; iteration process)
    ```
- 将多个功能合并到一条语句有助于简化循环：
    ```
    $ gawk '{
    > total = 0
    > for (i = 1; i < 4; i++)
    > {
    >    total += $i
    > }
    > avg = total / 3
    > print "Average:",avg
    > }' data5
    Average: 128.333
    Average: 137.667
    Average: 176.667
    $
    ```
    - 

17. 格式化打印
- 使用格式化打印命令 printf。如果熟悉 C 语言编程， 那么 gawk 中 printf 命 令的用法也是一样，允许指定具体如何显示数据的指令。
- printf 命令的格式如下：
    ```
    printf "format string", var1, var2
    ```
    - format string 是格式化输出的关键。它会用文本元素和格式说明符（format specifier）来具体指定如何呈现格式化输出。格式说明符是一种特殊的代码，可以指明显示什么类型的变量以及如何显示。 gawk 脚本会将每个格式说明符作为占位符，供命令中的每个变量使用。第一个格式说明符对应列出的第一个变量，第二个对应第二个变量，以此类推。
- 格式说明符的格式如下：
    ```
    %[modifier]control-letter
    ```
    - 其中， control-letter 是一个单字符代码， 用于指明显示什么类型的数据， modifier 定义了可选的格式化特性。
- 格式说明符的控制字母
    控制字母 | 描述
	---|---
    c   | 将数字作为 ASCII 字符显示
    d   | 显示整数值
    i   | 显示整数值（和 d 一样）
    e   | 用科学计数法显示数字
    f   | 显示浮点值
    g   | 用科学计数法或浮点数显示（较短的格式优先）
    o   | 显示八进制值
    s   | 显示字符串
    x   | 显示十六进制值
    X   | 显示十六进制值，但用大写字母 A~F

- 如果要显示一个字符串变量，可以用格式说明符%s。如果要显示一个整数值，可以用%d 或%i（ %d 是十进制数的 C 语言风格显示方式）。如果要用科学计数法显示很大的值， 可以使用格式说明符%e。
- 除了控制字母，还有 3 种修饰符可以进一步控制输出。
    - width：指定输出字段的最小宽度。如果输出短于这个值，则 printf 会将文本右对齐，并用空格进行填充。如果输出比指定的宽度长，则按照实际长度输出。
    - prec：指定浮点数中小数点右侧的位数或者字符串中显示的最大字符数。
    - -（减号）：指明格式化空间（formatted space）中的数据采用左对齐而非右对齐。
- 注意，你需要在 printf 命令的末尾手动添加换行符，以便生成新行，否则，printf 命令会继续在同一行打印后续输出。
- 例子：
    ```
    $ gawk 'BEGIN{FS="\n"; RS=""} {printf "%16s %s\n", $1, $4}' data2
            Ima Test  (312)555-1234
        Frank Tester  (317)555-9876
       Haley Example  (313)555-4938
    $
    $ gawk 'BEGIN{FS="\n"; RS=""} {printf "%-16s %s\n", $1, $4}' data2
    Ima Test          (312)555-1234
    Frank Tester      (317)555-9876
    Haley Example     (313)555-4938
    $
    
    printf "Average: %5.1f\n",avg
    ```
    - 在默认情况下， printf 命令使用右对齐来将数据放入格式化空间中。要改成左对齐，只需给修饰符加上一个减号即可。
    - 使用格式说明符%5.1f 强制 printf 命令将浮点值近似到小数点后一位。
    
18. gawk 编程语言提供了不少内置函数，以用于执行一些常见的数学、字符串以及时间运算。

19. 数学函数
- gawk 数学函数
    函数 | 描述
    ---|---
    atan2(x, y) | x/y 的反正切， x 和 y 以弧度为单位
    cos(x)      | x 的余弦， x 以弧度为单位
    exp(x)      | x 的指数
    int(x)      | x 的整数部分， 取靠近 0 一侧的值
    log(x)      | x 的自然对数
    rand( )     | 比 0大且比 1 小的随机浮点值
    sin(x)      | x 的正弦， x 以弧度为单位
    sqrt(x)     | x 的平方根
    srand(x)    | 为计算随机数指定一个种子值

- int() 函数会生成一个值的整数部分，但并不会四舍五入取近似值。它的做法更像其他编程语言中的 floor 函数，会生成该值和 0 之间最接近该值的整数。
- rand()函数会返回一个随机数，但这个随机数只在 0 和 1 之间（不包括 0 或 1）。要得到更大的数，就需要放大返回值。产生较大随机整数的常见方法是综合运用函数 rand()和 int()创建一个算法：
    ```
    x = int(10 * rand())
    ```
    - 这会返回一个 0~9 （包括 0 和 9）的随机整数值。只要在程序中用上限值替换等式中的 10 就可以了。
- 在使用一些数学函数时要小心，因为gawk 编程语言对于其能够处理的数值有一个限定区间。如果超出了这个区间，就会得到一条错误消息：
    ```
    $ gawk 'BEGIN{x=exp(1000); print x}'
    gawk: warning: exp argument 1000 is out of range
    inf
    $
    ```
    - 计算 e 的 1000 次幂，这已经超出了系统的数值区间，因此产生了一条错误消息。
- gawk 还支持一些按位操作数据的函数。
    - and(v1, v2)：对 v1 和 v2 执行按位 AND 运算。
    - compl(val)：对 val 执行补运算。
    - lshift(val, count)：将 val 左移 count 位。
    - or(v1, v2)：对 v1 和 v2 执行按位 OR 运算。
    - rshift(val, count)：将 val 右移 count 位。
    - xor(v1, v2)：对 v1 和 v2 执行按位 XOR 运算。 

20. 字符串函数
- gawk 字符串函数
    函数 | 描述
    ---|---
    asort(s [,d])               | 将数组 s 按照数组元素值排序。索引会被替换成表示新顺序的连续数字。另外，如果指定了 d，则排序后的数组会被保存在数组 d 中
    asorti(s [,d])              | 将数组 s 按索引排序。生成的数组会将索引作为数组元素值，用连续数字索引表明排序顺序。另外，如果指定了 d，则排序后的数组会被保存在数组 d 中
    gensub(r, s, h [,t])        | 针对变量$0或目标字符串 t（如果提供了的话）来匹配正则表达式 r。如果 h 是一个以 g 或 G 开头的字符串，就用 s 替换匹配的文本。如果 h 是一个数字，则表示要替换 r 的第 h 处匹配
    gsub(r, s [,t])             | 针对变量$0或目标字符串 t（如果提供了的话） 来匹配正则表达式 r。如果找到了，就将所有的匹配之处全部替换成字符串 s
    index(s, t)                 | 返回字符串 t 在字符串 s 中的索引位置；如果没找到，则返回 0
    length([s])                 | 返回字符串 s 的长度；如果没有指定，则返回$0 的长度
    match(s, r [,a])            | 返回正则表达式 r 在字符串 s 中匹配位置的索引。如果指定了数组 a，则将 s 的 匹配部分保存在该数组中
    split(s, a [,r])            | 将 s 以 FS（字段分隔符）或正则表达式 r（如果指定了的话）分割并放入数组 a 中。返回分割后的字段总数
    sprintf(format, variables)  | 用提供的 format 和 variables 返回一个类似于 printf 输出的字符串
    sub(r, s [,t])              | 在变量$0 或目标字符串 t 中查找匹配正则表达式 r 的部分。如果找到了，就用字符串 s 替换第一处匹配
    substr(s, i [,n])           | 返回 s 中从索引 i 开始、长度为 n 的子串。如果未提供 n，则返回 s 中剩下的部分
    tolower(s)                  | 将 s 中的所有字符都转换成小写
    toupper(s)                  | 将 s 中的所有字符都转换成大写

- asort 和 asorti 是新加入的 gawk 函数，允许基于数据元素值（asort）或索引（asorti）对数组变量进行排序。这里有个使用 asort 的例子：
    ```
    $ gawk 'BEGIN{
    > var["a"] = 1
    > var["g"] = 2
    > var["m"] = 3
    > var["u"] = 4
    > asort(var, test)
    > for (i in test)
    >     print "Index:",i," - value:",test[i]
    > }'
    Index: 4  - value: 4
    Index: 1  - value: 1
    Index: 2  - value: 2
    Index: 3  - value: 3
    ```
    - 新数组 test 包含经过排序的原数组的数据元素，但数组索引变成了表明正确顺序的数字值。
- split 函数是将数据字段放入数组以供进一步处理的好办法：
    ```
    $ gawk 'BEGIN{ FS=","}{
    > split($0, var)
    > print var[1], var[5]
    > }' data1
    data11 data15
    data21 data25
    data31 data35
    $
    ```
    - 新数组使用连续数字作为数组索引，从含有第一个数据字段的索引值 1 开始。

21. 时间函数
-  gawk 的时间函数
    函数 | 描述
    ---|---
    mktime(datespec)                | 将一个按 YYYY MM DD HH MM SS [DST]格式指定的日期转换成时间戳
    strftime(format [, timestamp])  | 将当前时间的时间戳或 timestamp （如果提供了的话）转化为格式化日期 （采用 shell 命令 date 的格式）
    systime()                       | 返回当前时间的时间戳

- 时间戳（ timestamp ）是自 1970-01-01 00:00:00 UTC 到现在，以秒为单位的计数，通常称为纪元时（epoch time）。systime()函数的返回值也是这种形式。
- 时间函数多用于处理日志文件。日志文件中通常含有需要进行比较的日期。通过将日期的文本表示形式转换成纪元时（自 1970-01-01 00:00:00 UTC 到现在的秒数），可以轻松地比较日期。
- 在 gawk 脚本中使用时间函数的例子：
    ```
    $ gawk 'BEGIN{
    > date = systime()
    > day = strftime("%A, %B %d, %Y", date)
    > print day
    > }'
    Friday, December 26, 2014
    $
    ```
    - 这个例子用 systime 函数从系统获取当前的时间戳，然后用 strftime 函数将其转换成用户可读的格式，转换过程中用到了 shell 命令 date 的日期格式化字符。

22. 除了 gawk 中的内建函数， 还可以在 gawk 脚本中创建自定义函数。

23. 定义函数
- 要定义自己的函数，必须使用 function 关键字：
    ```
    function name([variables])
    {
        statements
    }
    ```
- 函数名必须能够唯一标识函数。你可以在调用该函数的 gawk 脚本中向其传入一个或多个变量，还可以使用 return 语句返回一个值：
    ```
    function myrand(limit)
    {
        return int(limit * rand())
    }
    
    x = myrand(100)
    ```
    - 可以将函数的返回值赋给 gawk脚本中的变量。

24. 使用自定义函数
- 在定义函数时，它必须出现在所有代码块之前（包括 BEGIN 代码块）。这有助于将函数代码与 gawk 脚本的其他部分分开：
    ```
    $ gawk '
    > function myprint()
    > {
    >     printf "%-16s - %s\n", $1, $4
    > }
    > BEGIN{FS="\n"; RS=""}
    > {
    >     myprint()
    > }' data2
    Ima Test        - (312)555-1234
    Frank Tester    - (317)555-9876
    Haley Example   - (313)555-4938
    $
    ```
    - 脚本中定义了 myprint()函数，该函数负责格式化记录中的第一个和第四个数据字段以供打印输出。然后， gawk 脚本会调用该函数以显示数据文件中的数据。
- 一旦定义了函数， 就可以在程序的代码中随意使用了。

25. 创建函数库
- 可以将多个函数放入单个库文件中，这样就可以在所有的 gawk 脚本中使用了。
- 首先，需要创建一个包含所有 gawk 函数的文件：
    ```
    $ cat funclib
    function myprint()
    {
        printf "%-16s - %s\n", $1, $4
    }
    function myrand(limit)
    {
        return int(limit * rand())
    }
    function printthird()
    {
        print $3
    }
    $
    ```
    - funclib 文件含有 3 个函数定义。加上-f 命令行选项就可以使用该文件了。很遗憾， -f 选项不能和内联 gawk 脚本（inline gawk script）一起使用，不过可以在同一命令行中使用多个-f 选项。
- 因此，要使用库，只要创建好 gawk 脚本文件，然后在命令行中同时指定库文件和脚本文件即可：
    ```
    $ cat script4
    BEGIN{ FS="\n"; RS=""}
    {
        myprint()
    }
    $ gawk -f funclib -f script4 data2
    Ima Test        - (312)555-1234
    Frank Tester    - (317)555-9876
    Haley Example   - (313)555-4938
    $	
    ```

---

###### 二十三、使用其他 shell

1. 可以看看自己的账户使用的默认交互式 shell。例如：
    ```
    $ cat /etc/passwd | grep rich
    rich:x:1000:1000:Rich,,,:/home/rich:/bin/bash
    $
    ```
- Ubuntu 系统使用 bash shell 作为默认的交互式 shell。
- 想知道默认的系统 shell 是什么，只需使用 ls 命令查看/bin 目录中的 sh 文件即可：
    ```
    $ ls -al /bin/sh
    lrwxrwxrwx 1 root root 4 Jul 21 08:10 /bin/sh -> dash
    $
    ```
    - Ubuntu 系统使用 dash shell 作为默认的系统 shell 。
- 在大多数 Linux 发行版中， /bin/sh 文件是指向/bin/bash 的一个符号链接。但 Ubuntu Linux 发行版将/bin/sh 文件链接到了 shell 程序/bin/dash。由于 dash shell 只包含原始 Bourne shell 的一部分命令， 因此这可能会（而且经常会） 让一些 shell 脚本无法正确工作。

2. dash shell 的特性

略。

3. dash 脚本编程

略。

4. zsh shell
- 下面是 zsh shell 的一些独有特性：
    - 改进的 shell 选项处理。
    - shell 兼容性模式。
    - 可加载模块。
- 每种 shell 都包含一组内建命令，这些命令无须借助外部程序即可使用。内建命令的好处在于执行速度快。 shell 不必在运行命令前先加载一个外部程序，因为内建命令已经在内存中了，随时可用。
- zsh shell 提供了一组核心内建命令，并具备增添附加命令模块的能力。每个命令模块都为特定场景提供了一组内建命令，比如网络支持和高级数学功能。可以只添加你认为有用的模块。
- 大多数 Linux 发行版没有默认安装 zsh shell。

5. zsh shell 的组成——shell 选项
- 可以在命令行或 shell 中用 set 命令设置 shell选项。
- zsh shell 命令行选项
    选项 | 描述
    ---|---
    -c  | 只执行指定的命令，然后退出
    -i  | 作为交互式 shell 启动，提供命令行提示符
    -s  | 强制 shell 从 STDIN 读取命令
    -o  | 指定命令行选项
- -o 选项允许设置 shell 选项来定义 shell 的各种特性。
- shell 选项可以分成以下几类：
    - 更改目录：该选项用于控制 cd 命令和 dirs 命令如何处理目录更改。
    - 补全：该选项用于控制命令补全功能。
    - 扩展和通配符匹配：该选项用于控制命令中文件扩展。
    - 历史：该选项用于控制命令历史记录回调。
    - 初始化：该选项用于控制 shell 在启动时如何处理变量和启动文件。
    - 输入输出：该选项用于控制命令处理。
    - 作业控制：该选项用于控制 shell 如何处理作业和启动作业。
    - 提示符：该选项用于控制 shell 如何处理命令行提示符。
    - 脚本和函数：该选项用于控制 shell 如何处理 shell 脚本和定义 shell 函数。
    - shell 仿真：该选项允许设置 zsh shell 来模拟其他类型 shell 的行为。
    - shell 状态：该选项用于定义启动哪种 shell。
    - zle ：该选项用于控制 zsh 行编辑器（zle）功能。
    - 选项别名：可以用作其他选项别名的特殊选项。

6. zsh shell 的组成——内建命令
- zsh shell 的独到之处在于能够扩展 shell 的内建命令。
- zsh 核心内建命令
    命令 | 描述
    ---|---
    alias       | 为命令及参数定义一个替代性名称
    autoload    | 将 shell 函数预加载到内存中以便快速访问
    bg          | 以后台模式执行作业
    bindkey     | 将组合键和命令绑定到一起
    builtin     | 执行指定的内建命令，而非同名的可执行文件
    bye         | 同 exit
    cd          | 切换当前工作目录
    chdir       | 切换当前工作目录
    command     | 执行以外部可执行文件为形式的命令，而非同名的函数或内建命令
    declare     | 设置变量的数据类型（同 typeset）
    dirs        | 显示目录栈的内容
    disable     | 临时禁用指定的哈希表元素
    disown      | 从作业表中移除指定的作业
    echo        | 显示变量和文本
    emulate     | 用 zsh 来仿真另一种 shell，比如 Bourne、Korn 或 C shell
    enable      | 启用指定的哈希表元素
    eval        | 在当前 shell 进程环境中执行指定的命令和参数
    exec        | 执行指定的命令和参数来替换当前 shell 进程
    exit        | 退出 shell 并返回指定的退出状态码。如果未指定，则使用最后一个命令的退出状态码 
    export      | 允许在子 shell 进程中使用指定的环境变量
    false       | 返回退出状态码 1
    fc          | 从历史记录中选择某个范围内的命令
    fg          | 以前台模式执行指定的作业
    float       | 将指定变量设为浮点类型
    functions   | 将指定名称设为函数
    getln       | 从缓冲栈中读取下一个值并将其放入指定变量
    getopts     | 提取命令行参数中的下一个有效选项并将其放入指定变量
    hash        | 直接修改命令哈希表的内容
    history     | 列出历史记录文件中的命令
    integer     | 将指定变量设为整数类型
    jobs        | 列出指定作业的信息，或是分配给 shell 进程的所有作业
    kill        | 向指定进程或作业发送信号（默认为 SIGTERM）
    let         | 执行数学运算并将结果赋给变量
    limit       | 设置或显示资源限制
    local       | 将指定变量设为局部变量
    log         | 显示受 watch 参数影响的所有当前登录用户
    logout      | 同 exit，但仅适用于当前 shell 为登录 shell
    popd        | 从目录栈中删除下一项
    print       | 显示变量和文本
    printf      | 用 C 语言风格的格式字符串来显示变量和文本
    pushd       | 改变当前工作目录，并将上一个目录放入目录栈
    pushln      | 将指定参数放入编辑缓冲栈
    pwd         | 显示当前工作目录的完整路径
    read        | 读取一行并用 IFS 变量将字段赋给指定变量
    readonly    | 将值赋给只读变量
    rehash      | 重建命令哈希表
    set         | 为 shell 设置选项或位置参数
    setopt      | 为 shell 设置选项
    shift       | 读取并删除第一个位置参数，然后将剩余的参数向前移动一个位置
    source      | 找到指定文件并将其内容复制到当前位置
    suspend     | 挂起 shell 的执行，直至收到 SIGCONT 信号
    test        | 如果指定条件为 TRUE，就返回退出状态码 0
    times       | 显示当前 shell 以及 shell 中所有运行进程的累计用户时间和系统时间
    trap        | 阻断 shell 处理指定信号， 如果收到信号则执行指定命令
    true        | 返回退出状态码 0
    ttyctl      | 锁定和解锁显示
    type        | 显示 shell 会如何解释指定的命令
    typeset     | 设置或显示变量的属性
    ulimit      | 设置或显示 shell 或 shell 中运行进程的资源限制
    umask       | 设置或显示创建文件和目录的默认权限
    unalias     | 删除指定的命令别名
    unfunction  | 删除指定的已定义函数
    unhash      | 删除哈希表中的指定命令
    unlimit     | 取消指定的资源限制
    unset       | 删除指定的变量属性
    unsetopt    | 禁用指定的 shell 选项
    wait        | 等待指定的作业或进程完成
    whence      | 显示 shell 如何解释指定命令
    where       | 显示指定命令的路径（如果 shell 能找到的话）
    which       | 用 csh shell 风格的输出显示指定命令的路径
    zcompile    | 编译指定的函数或脚本， 提高自动加载速度
    zmodload    | 对可加载 zsh 模块执行特定操作
    
- 有大量的模块可以为 zsh shell 提供额外的内建命令。一些流行的 zsh 模块如下表：
    模块 | 描述
    ---|---
    zsh/datetime    | 附加的日期和时间命令及变量
    zsh/files       | 基础的文件处理命令
    zsh/mapfile     | 通过关联数组来访问外部文件
    zsh/mathfunc    | 附加的科学函数
    zsh/pcre        | 扩展正则表达式库
    zsh/net/socket  | Unix 域套接字支持
    zsh/stat        | 访问 stat 系统调用来提供系统的统计状况
    zsh/system      | 各种底层系统功能的接口
    zsh/net/tcp     | 访问 TCP 套接字
    zsh/zftp        | FTP 客户端命令
    zsh/zselect     | 阻塞，直到文件描述符就绪才返回
    zsh/zutil       | 各种 shell 实用工具

- zmodload 命令是 zsh 模块的管理接口。你可以在zsh shell 会话中使用该命令查看、添加或删除模块。
- 不加任何参数的 zmodload 命令会显示 zsh shell 中当前已安装的模块：
    ```
    % zmodload
    zsh/complete
    zsh/files
    zsh/main
    zsh/parameter
    zsh/stat
    zsh/terminfo
    zsh/zle
    zsh/zutil
    %
    ```
- 不同的 zsh shell 实现在默认情况下包含了不同的模块。要添加新模块，只需在 zmodload 命令行中指定模块名称即可：
    ```
    % zmodload zsh/net/tcp
    %
    ```
    - 无显示信息则表明模块已经加载成功。再运行一次 zmodload 命令，新模块会出现在已安装模块的列表中。一旦加载了模块，该模块中的命令就成了可用的内建命令。
- 将 zmodload 命令放入$HOME/.zshrc 启动文件是一种常见的做法，这样在 zsh 启动时就会自动加载常用的模块。

7. zsh 脚本编程——数学运算
- zsh shell 在所有数学运算中都提供了对浮点数的全面支持。
- zsh shell 提供了执行数学运算的两种方法：
    - let 命令
    - 双圆括号
- 在使用 let 命令时，应该在算式前后加上双引号，这样才能使用空格：
    ```
    % let value1=" 4 * 5.1 / 3.2 "
    % echo $value1
    6.3749999999999991
    % printf "%6.3f\n" $value1
    6.375
    %
    ```
    - 注意，使用浮点数会带来精度问题。要解决这个问题，最好使用 printf 命令指定所需的小数点精度，以便正确显示结果。
- 第二种方法是使用双圆括号。这种方法结合了两种定义数学运算的方法：
    ```
    % value1=$(( 4 * 5.1 ))
    % (( value2 = 4 * 5.1 ))
    % printf "%6.3f\n" $value1 $value2
    20.400
    20.400
    %
    ```
    - 注意，可以将双圆括号放在算式两边（前面加上美元符号）或整个赋值表达式两边。两种方法能输出同样的结果。
- 如果一开始未用 typeset 命令声明变量的数据类型，那么 zsh shell 会尝试自动分配数据类型。这在处理整数和浮点数时很容易出问题：
    ```
    % value=10
    % value2=$(( $value1 / 3 ))
    % echo $value2
    3
    %
    
    % value=10.
    % value2=$(( $value1 / 3. ))
    % echo $value2
    3.3333333333333335
    %
    ```
    - 在指定数值时未指定小数部分的话， zsh shell 会将其 视为整数值并进行整数运算。如果想保证结果是浮点数，则必须指定小数部分。
- 默认的 zsh shell 不含任何特殊的数学函数，可安装 zsh/mathfunc 模块。
    ```
    % value1=$(( sqrt(9) ))
    zsh: unknown function: sqrt
    % zmodload zsh/mathfunc
    % value1=$(( sqrt(9) ))
    % echo $value1
    3.
    %
    ```
- zsh 支持大量数学函数。要查看 zsh/mathfunc 模块提供的所有数学函数的清单，可参见 zshmodules 的手册页。	

8. zsh 脚本编程——结构化命令
- zsh shell 为 shell 脚本提供了常用的结构化命令：
    - if-then-else 语句
    - for 循环（包括 C 语言风格的）
    - while 循环
    - until 循环
    - select 语句
    - case 语句
- zsh 中的结构化命令采用的语法和你熟悉的bash shell 一样。
- zsh shell 还提供了另一个结构化命令 repeat。该命令格式如下：
    ```
    repeat param
    do
        commands
    done
    ```
    - param 参数必须是一个数值， 或是能计算出一个值的数学运算。然后，repeat 命令就会执行那么多次指定的命令：
        ```
        value1=$(( 10 / 2 ))
        repeat $value1
        do
            echo "This is a test"
        done
        $ ./test4
        This is a test
        This is a test
        This is a test
        This is a test
        This is a test
        ```
        - 该命令允许基于计算结果执行指定的代码块若干次。

9. zsh 脚本编程——函数
- zsh shell 支持使用 function 命令或函数名加圆括号的形式来创建自定义函数：
    ```
    % function functest1 {
    > echo "This is the test1 function"
    }
    % functest2() {
    > echo "This is the test2 function"
    }
    % functest1
    This is the test1 function
    % functest2
    This is the test2 function
    %
    ```
- 跟 bash shell 函数一样，可以在 shell 脚本中定义函数，然后使用全局变量或向函数传递参数。

10. 实战演练
- zsh shell 的 tcp 模块尤为实用。该模块允许创建 TCP 套接字，侦听传入的连接， 然后与远程系统建立连接。这是在 shell 脚本之间传输数据的绝佳方式。
- 首先，打开 shell 窗口作为服务器。启动 zsh，加载 tcp 模块，然后定义 TCP套接字的侦听端口号。相关命令如下：
    ```
    server$ zsh
    server% zmodload zsh/net/tcp
    server% ztcp -l 8000
    server% listen=$REPLY
    server% ztcp -a $listen
    ```
    - ztcp 命令的-l 选项指定了侦听的 TCP 端口号（在本例中是 8000）。特殊的$REPLY 变量包含与网络套接字关联的文件句柄（file handle）。 ztcp 命令的-a 选项会一直等待传入连接建立完毕。
- 在系统中（或是在处于同一网络的另一个 Linux 系统中）打开另一个 shell 窗口作为 客户端，输入下列命令来连接服务器端 shell：
    ```
    client$ zsh
    client% zmodload zsh/net/tcp
    client% ztcp localhost 8000
    client% remote=$REPLY
    client%
    ```
- 当连接建立好之后，你会在服务器端的 shell 窗口中看到 zsh shell 命令行提示符。你可以在服务器端将新连接的句柄保存在变量中：
    ```
    server% remote=$REPLY
    ```
- 这样就可以收发数据了。要想发送消息，可以使用 print 语句将文本发送到$remote 连接句柄：
    ```
    client% print 'This is a test message' >&$remote
    client%
    ```
- 在另一个 shell 窗口中，可以使用 read 命令接收发送到$remote 连接句柄的数据，然后使用 print 命令将其显示出来：
    ```
    server% read -r data <&$remote; print -r $data
    This is a test message
    server%
    ```
- 最后，使用-c 选项关闭各个系统中对应的句柄。对服务器端来说，可以使用下 列命令：
    ```
    server% ztcp -c $listen
    server% ztcp -c $remote
    ```
    - 对客户端来说，可以使用下列命令：
        ```
        client% ztcp -c $remote
        ```

---

###### 二十四、编写简单的脚本实用工具

1. 日常备份
- 在 Linux 世界中，备份数据的工作是由 tar 命令完成的。tar 命令可以将整 个目录归档到单个文件中。
- 使用 tar 命令来创建工作目录归档文件的例子：
    ```
    $ tar -cf archive.tar /home/christine/Project/*.*
    tar: Removing leading `/' from member names
    tar: Removing leading `/' from hard link targets
    $
    $ ls -og archive.tar
    -rw-rw-r-- 1 112640 Aug 6 13:33 archive.tar
    
    $ tar -cf archive.tar Project/*.* 2>/dev/null
    $
    
    $ tar -zcf archive.tgz Project/*.* 2>/dev/null
    $
    $ ls -hog archive.tgz
    -rw-rw-r-- 1 11K Aug 6 13:40 archive.tgz
    ```
    - 注意， tar 命令会显示一条警告消息，指出它删除了路径开头的斜线。这意味着将路径从绝对路径改为了相对路径， 以便将 tar 归档文件提取到文件系统中的任何位置。如果不想在脚本中输出这条消息，则可以将 STDERR 重定向到/dev/null 文件。
    - 由于 tar 归档文件会占用大量的磁盘空间，因此最好压缩一下。这只需加一个-z 选项即可。该选项会使用 gzip压缩 tar 归档文件，由此生成的文件称作tarball。请务必使用恰当的文件扩展名来表示这种文件是一个 tarball，采用.tar.gz 或.tgz 都行。
    - 经过压缩， archive.tgz 比 archive.tar 小了约 99KB。
- 使用 exec 命令来重定向标准输入（STDIN）。实现方法如下：
    ```
    exec 0 < $config_file
    read file_name
    ```
    - 注意，我们对归档配置文件使用了一个变量 config_file。配置文件中每一条记录都会被读入。
    - 只要 read 命令在配置文件中发现还有记录可读，就会在?变量中返回一个表示成功的退出状态码 0。这可以作为 while 循环的测试条件来读取配置文件中的所有记录。
        ```
        while [ $? -eq 0 ]
        do
         [...]
        read file_name
        done
        ```
        - 一旦 read 命令读到配置文件的末尾，就会在?变量中返回一个非 0 状态码。这时，脚本会退出 while 循环。
- 记住， tar 只是使用 bash shell 命令在系统中执行备份的一种方法。有一些其他的实用程序（或命令组合）也许能更好地满足你的需求，比如 rsync。要查看可能有助于备份工作的各种实用工具名称，可以在命令行提示符下输入 man -k archive 和 man -k copy。

2. 创建按小时归档的脚本
- 如果你当天运行 Hourly_Archive.sh 脚本，那么当小时数是单个数字时， 归档文件名中只会出现 3 位数字。如果运行脚本的时间是 1:15 am ，那么归档文件名就是 archive115.tar.gz。如果希望文件名中总是保留 4 位数字， 则可以将脚本中的 TIME=$(date +%k%M)修改成 TIME=$(date +%k0%M)。在%k 后加入数字 0 后，所有的单位（single-digit）小时数都会加上一个前导数字 0，被填充成两位数字 。因此 ，archive115.tar.gz 就变成了 archive0115.tar.gz。

3. 删除账户
- cut 命令的-c1 选项可以删除 answer 中除第一个字符之外的所有内容。
- xargs 命令可以使用从标准输入 STDIN 获取的命令参数并执行指定的命令。它非常适合放在管道的末尾处。
- xargs 的现代版本不要求使用命令（比如 sudo 和 kill）的绝对路径。

4. 系统监控
- 获得默认的 shell 审计功能
    - 系统账户用于提供服务或执行特殊任务。一般来说，这类账户需要在/etc/passwd 文件中有对应的记录，但禁止登录系统（root 账户是一个典型的例外）。防止有人使用这些账户登录的方法是，将其默认 shell 设置为/bin/false 、/usr/sbin/nologin 或 /sbin/nologin。
    - 使用 cut 命令获取/etc/passwd 文件中所有账户的默认 shell。该命令可以指定文件的字段分隔符以及要提取的记录字段。对于/etc/passwd 文件，分隔符是冒号（:），账户的默认 shell 位于记录的第 7 个字段：
        ```
        $ cut -d: -f7 /etc/passwd
        /bin/bash
        /usr/sbin/nologin
        /usr/sbin/nologin
        /usr/sbin/nologin
         [...]
        /bin/false
        /bin/bash
        /usr/sbin/nologin
        /bin/bash
        /usr/sbin/nologin
        /bin/bash
        $
        ```
- find 命令的-perm（permissions） 选项可以使用八进制值指定要查找的具体权限。
- diff 命令只会逐行对文件进行比较。因此， diff 会比较两份报告的第一行，然后是第二行、第三行，以此类推。如果由于要安装软件，添加了一个或一批新文件，而这些文件需要 SUID 权限或 SGID 权限，那么在下一次审计时， diff 就会显示大量的差异。为了解决这个潜在的问题，可以在 diff 命令中使用-q 选项或--brief 选项，只显示消息，说明这两份报告存在不同。

---

###### 二十五、井井有条

1. 版本控制（也称为源代码控制或修订控制）是一种组织各种项目文件并跟踪其更新的方法（或系统）。
- 版本控制系统（ version control system，VCS）提供了一个公共中央位置来存储和合并 bash 脚本文件， 以便轻松访问脚本的最新版本。 VCS 能够保护文件，使其不会被另一个脚本编写者意外覆盖。同时还消除了谁当前正在修改什么内容这类额外通信。
- 分布式 VCS 使脚本项目开发变得更加容易。脚本编写者可以在自己的 Linux 系统中进行开发或修改工作。 一旦达到修改目标， 就将修改后的文件副本和 VCS 元数据发送到远程中央系统，其他团队成员可以下载这个最新的项目版本并进行测试，或是继续他们自己的修改任务。
- Git 是一种分布式VCS，多部署于敏捷和持续软件开发环境中。不过它也可用于管理bash shell 脚本。

2. 工作目录
- 工作目录是所有脚本的创建、修改和审查之地。它通常是脚本编写者的主目录中的某个子目录， 类似于/home/christine/scripts。最好为每个项目都创建一个新的子目录，因为 Git 会在其中放置文件，以便进行跟踪。

3. 暂存区
- 暂存区也称为索引。该区域和工作目录位于同一系统。 bash 脚本通过 Git 命令（git add） 在暂存区内注册。通过 git init 命令，暂存区在工作目录中设置了一个名为.git 的隐藏子目录。
- 当脚本被编入暂存区时， Git 会在索引文件.git/index 中创建或更新脚本信息。记录的数据包括校验和、时间戳和相关的脚本文件名。
- 除了更新索引文件， Git 还会压缩脚本文件并将这些压缩文件作为对象（也称为 blob）存储 在.git/objects/目录中。如果脚本已被修改，则将其作为一个新对象压缩并存储在.git/objects/目录中。 Git 不只存储脚本的改动，还会保留每个已修改的脚本的压缩副本。

4. 本地仓库
- 本地仓库包括每个脚本文件的历史记录。它也会用到工作目录的.git 子目录。脚本文件的各 个版本（称为项目树）和提交信息之间的关系通过 Git 命令（ git commit），以对象的方式存储在.git/objects/ 目录中。
- 项目树和提交数据合起来称为快照。每次提交数据都会创建一个新快照。不过，旧快照并不 会被删除，依然可以查看。如果需要，还可以返回到之前的快照，这是 Git 另一个不错的特性。

5. 远程仓库
- 在 Git 配置中，远程仓库通常位于云端，提供代码托管服务。然而，也可以在本地网络中的 另一台服务器上建立代码托管站点作为远程仓库。
- 著名的远程仓库有 GitHub 、GitLab 、BitBucket 和 Launchpad。

6. 分支
- Git 还提供了一个名为分支的特性， 该特性可以在各种脚本项目中发挥作用。
- 支是本地仓库中属于特定项目的一个区域。举例来说，你可以将脚本项目的主分支命名为 main，当你打算对 main 分支中的脚本进行改动时，最好的做法是创建一个新分支（比如命名为 modification），在该分支中修改脚本。一旦脚本的改动通过了测试， modification 分支中的脚本通常就会被合并回主分支。
- 使用这种方法的好处在于，存放在 main 分支中的脚本仍具有生产价值，因为正在被修改和 测试的 bash shell 脚本位于另一个分支。只有当修改过的脚本通过测试之后，才会被并入 main主分支。

7. 克隆
- Git 的另一个特性是复制项目。这个过程称为克隆。如果你的团队有新人加入，那么他可以 从远程仓库克隆脚本和跟踪文件，获得开展工作所需的一切资源。
- 在 Git 中，克隆（cloning）和分叉（forking）是两种紧密相关的操作。使用 git clone
命令将文件从远程仓库下载到本地系统，这一过程是克隆。将文件从一个远程仓库复制到另一个远程仓库，这一过程是分叉。

8. 使用 Git 作为 VCS
- 将 Git 作为 VCS 能带来如下好处：
    - 性能：Git 只操作本地文件，这提高了其部署速度。同远程仓库之间收发文件属于例外情况。
    - 历史文件：从文件被注册那一刻起， Git 就开始使用索引来记录文件的内容。当对本地存储库的提交完成时， Git 会及时创建并存储对该快照的引用。
    - 准确性：Git 使用校验和来保护文件完整性。
    - 去中心化：脚本编写者可以在同一个项目中工作，但不必位于同一网络或系统。

9. 设置 Git 环境
- Git 通常并非默认安装项，在设置 Git 环境之前，需要自行安装 git 软件包。
- 在 CentOS Linux 发行版中安装 Git 的过程如下：
    ```
    $ sudo dnf install git
    ...
    Complete!
    $ which git
    /usr/bin/git
    ```
- 在 Ubuntu Linux 发行版中安装 Git 的过程如下：
    ```
    $ sudo apt install git
    ...
    Processing triggers for man-db (2.9.1-1) ...
    $ which git
    /usr/bin/git
    ```
- 安装好 git 软件包之后，为新的脚本项目设置 Git 环境涉及以下 4 个基本步骤：
    - (1) 创建工作目录。
    - (2) 初始化.git/子目录。
    - (3) 设置本地仓库选项。
    - (4) 确定远程仓库位置
- 具体步骤如下：
    ```
    1、首先，创建工作目录。在本地主目录下创建一个子目录即可：
    $ mkdir MWGuard
    $
    $ cd MWGuard/
    $
    $ pwd
    /home/christine/MWGuard
    $

    2、然后，在工作目录中初始化.git/子目录。这要用到 git init 命令：
    $ git init
    Initialized empty Git repository in /home/christine/MWGuard/.git/
    $
    $ ls -ld .git
    drwxrwxr-x 7 christine christine 4096 Aug 24 14:49 .git
    $

    3、如果是首次使用，则设置name和email：
    $ git config --global user.name "Christine Bresnahan"
    $
    $ git config --global user.email "cbresn1723@gmail.com"
    $
    $ git config --get user.name
    Christine Bresnahan
    $
    $ git config --get user.email
    cbresn1723@gmail.com
    $
    
    4、配置好本地 Git 环境之后，就可以建立项目的远程仓库了。建立好项目的远程仓库之后，需要把仓库地址记下来。随后向远程仓库发送项目文件时， 要 用到这个地址。
    ```
    - 创建的子目录 MWGrard 用于脚本项目。然后， 使用 cd 命令进入工作目录。
    - git init 命令创建了.git/子目录。因为目录名之前有点号（.），所以普通的 ls 命令无法将其显示出来。使用 ls -la 命令或将该目录名作为 ls -ld 命令的参数就可以看到相关信息。
    - 可以同时拥有多个项目目录。为每个项目创建单独的工作目录即可。
    - 如果你是首次在系统中构建.git/子目录， 则需将姓名和 email 地址添加到 Git 的全局仓库配置文件中。这些标识信息有助于跟踪文件变更，尤其是多人参与项目的时候。
    - 在 git config 命令中加入--global 选项，就能把 user.name 和 user.email 保存在 Git 全局配置文件中。注意，要想查看此信息，可以使用--get 选项，并将数据名称作为参数。
    - Git 全局配置信息表示这些数据会应用于系统中的所有Git 项目。 Git 本地配置信息仅应用于工作目录中的特定 Git 项目。
    - Git 全局配置信息保存在主目录的.gitconfig 文件中，本地配置信息保存在 working-directory/ .git/config 文件中。注意，有些系统还有系统级的配置文件/etc/gitconfig。

- 要查看这些文件中的各种配置信息， 可以使用 git config --list 命令：
    ```
    $ git config --list
    user.name=Christine Bresnahan
    user.email=cbresn1723@gmail.com
    core.repositoryformatversion=0
    core.filemode=true
    core.bare=false
    core.logallrefupdates=true
    $ cat /home/christine/.gitconfig
    [user]
        name = Christine Bresnahan
        email = cbresn1723@gmail.com
    $
    $ cat /home/christine/MWGuard/.git/config
    [core]
        repositoryformatversion = 0
        filemode = true
        bare = false
        logallrefupdates = true
    $
    ```
    - 通过--list 选项显示的设置信息采用 file-section.name 格式。注意，当使用 cat 命令将两个 Git 配置文件（全局和项目的本地仓库）输出至 STDOUT 时， 会显示节名（section name）及其包含的数据。
    - 
- 尽管 Git 能够处理任意文件类型，但其相关工具主要针对的是纯文本文件，比如 bash shell 脚本。因此要注意，不是所有的 git 工具都能用于非文本文件。

10. 使用 Git 提交文件
- 建立好 Git 环境之后，就可以使用它的各种组织功能了。这也有 4 个基本步骤：
    - (1) 创建或修改脚本。
    - (2) 将脚本添加到暂存区（索引）。
    - (3) 将脚本提交至本地仓库。
    - (4) 将脚本推送至远程仓库。
- 例子：
    ```
    1、创建好脚本之后， 使用 git add 命令将其添加到暂存区（索引）。由于该脚本目前不在工作 目录/home/christine/MWGuard  中，因此需要先把它复制过来。然后切换到工作目录（通过 pwd 命令确认），执行 git add 命令：
    $ pwd
    /home/christine/scripts
    $
    $ cp MyGitExampleScript.sh /home/christine/MWGuard/
    $
    $ cd /home/christine/MWGuard/
    $
    $ pwd
    /home/christine/MWGuard
    $
    $ ls *.sh
    MyGitExampleScript.sh
    $
    $ git add MyGitExampleScript.sh
    $
    $ git status
    [&hellip;]
    No commits yet
    Changes to be committed:
    (use "git rm --cached <file>..." to unstage)
    new file:   MyGitExampleScript.sh
    $
    
    2、下一步是使用 git commit 命令将项目提交至本地仓库。可以使用-m 选项来添加注释，这有助于记录（ documenting ）提交。
    $ git commit -m "Initial Commit"
    [&hellip;] Initial Commit
        1 file changed, 5 insertions(+)
        create mode 100644 MyGitExampleScript.sh
    $
    $ cat .git/COMMIT_EDITMSG
    Initial Commit
    $
    $ git status
    [&hellip;]
    nothing to commit, working tree clean
    $
    
    3、创建 README.md 文件，将其加入暂存区并提交到本地仓库。
    $ pwd
    /home/christine/MWGuard
    $
    $ ls
    MyGitExampleScript.sh
    $
    $ echo "# Milky Way Guardian" > README.md
    $ echo "## Script Project" >> README.md
    $
    $ cat README.md
    # Milky Way Guardian
    ## Script Project
    $
    $ git add README.md
    $
    $ git status
    [...]
    Changes to be committed:
        (use "git restore --staged <file>..." to unstage)
            new file: README.md
    $
    $ git commit -m "README.md commit"
     [...] README.md commit
        1 file changed, 2 insertions(+)
        create mode 100644 README.md
    $
    $ git status
     [...]
    nothing to commit, working tree clean
    $
    ```
    - git add 命令不会产生任何输出。因此， 要想知道脚本是否成功， 需要使用git status 命令。 该命令显示，一个新文件 MyGitExampleScript.sh 已经被加入索引。
    - 注释被保存在 COMMIT_EDITMSG 文件中，它能够帮助我们记录为什么要修改脚本。
    - 执行过 git commit 之后， git status 命令会显示以下消息：nothing to commit, working directory clean。这说明 Git 现在认为，工作目录中的所有文件都已经提交至本地仓库了。
    - 如果使用 git commit 命令时没有加上-m 选项，那么你会被引至 vim 编辑器，要求手动编辑.git/COMMIT_EDITMSG 文件。
   
- 如果需要，可以将当前工作目录中的所有脚本同时添加到暂存区（索引）。只需使用 git
add .命令即可。注意，该命令结尾有个点号（.）！ 这个点号相当于一个通配符，告诉 Git 把工作目录中的所有文件都加入暂存区。但是，如果不想把某些文件添加到暂存区，则可以在工作目录中创建一个.gitignore  文件，将不希望加入暂存区的文件或目录名写入该文件。这样， git add .命令就会忽略这些文件或目录，只把其他的文件或目录加入暂存区。
- 暂存区的索引文件是.git/index。如果对该文件使用 file 命令，则其类型会显示为 Git index。Git 会使用此文件跟踪变更：
    ```
    $ file .git/index
    .git/index: Git index, version 2, 1 entries
    $
    ```
- 如果这是一个新的脚本项目，那么在注册过远程仓库账户后，需要创建一个称为 Markdown file 的特殊文件，其内容会显示在远程仓库的 Web 页面上，描述该仓库的相关信息。该文件使用 Markdown 语言编写。你需要将文件命名为 README.md。
- 可以随时查看 Git 日志，但最好在将脚本项目推送到远程存储库之前做这件事。每次提交都有一个对应的哈希值作为标识，这个值也会出现在日志中。此外，请注意各种注释以及日期和作者信息。
    ```
    $ git log
    commit 898330bd0b01e0b6eee507c5eeb3c72f9544f506[...]
    Author: Christine Bresnahan <cbresn1723@gmail.com>
    Date:   Mon Aug 24 15:58:52 2020 -0400
        
        README.md commit
    
    commit 3b484638bc6e391d0a1b816946cba8c5f4bbc8e6
    Author: Christine Bresnahan <cbresn1723@gmail.com>
    Date:   Mon Aug 24 15:46:56 2020 -0400
    
        Initial Commit
    $
    ```
- 在向远程仓库推送项目之前，需要先在系统中配置远程仓库地址。在使用 Git 服务提供商 （比如 GitHub）设置远程仓库时，它会向你提供此地址。
- 可以使用 git remote add origin URL 命令来添加地址， 其中 URL 就是远程仓库地址：
    ```
    $ git remote add origin https://github.com/C-Bresnahan/MWGuard.git
    $
    $ git remote -v
    origin https://github.com/C-Bresnahan/MWGuard.git (fetch)
    origin https://github.com/C-Bresnahan/MWGuard.git (push)
    $
    ```
    - 注意，我们通过 git remote -v 命令检查了远程仓库地址的状态。在推送项目之前，最好先检查一下地址。如果地址有误或有输入错误，那么推送操作就会失败，因此一定要仔细检查！
    - 如果地址不对（比如有输入错误）， 则可以通过 git remote rm origin 命令删除远程仓库地址， 然后使用正确的地址重新设置。
- 配置好远程仓库地址之后，就可以向其推送脚本项目了。但在此之前，为简单起见，我们打算使用 git branch 命令把主分支重命名为main：
    ```
    $ git branch -m main
    $
    $ git branch --show-current
    main
    $
    ```
    - 注意，可以使用 git branch --show-current 命令查看分支的当前名称。在推送之前， 最好先确认分支名无误，在 push 命令中也要用到此分支名。
- 将脚本复制到远程仓库。这需要在 push 命令中加入-u origin 选项来指定仓库位置和当前使用的分支名 main：
    ```
    $ git remote add origin https://github.com/C-Bresnahan/MWGuard.git
    $
    $ git push -u origin main
    Username for 'https://github.com ': C-Bresnahan
    Password for 'https://C-Bresnahan@github.com ':
    Enumerating objects: 6, done.
    Counting objects: 100% (6/6), done.
    Compressing objects: 100% (4/4), done.
    Writing objects: 100% (6/6), 604 bytes | 60.00 KiB/s, done.
    Total 6 (delta 0), reused 0 (delta 0)
    To https://github.com/C-Bresnahan/MWGuard.git
        * [new branch]      main -> main
    Branch 'main' set up to track remote branch 'main' from 'origin'.
    $
    ```
    - 远程仓库通常要求输入用户名和密码。项目被推送至远程仓库之后，你应该能通过 Web 浏览器看到。
- 远程仓库真正的美妙之处在于， Linux 管理团队中参与此项目的任何人都可以使用 git pull 命令从中拉取最新版本的脚本。
    ```
    $ git remote add origin https://github.com/C-Bresnahan/MWGuard.git
    $
    $ git pull origin main
    remote: Enumerating objects: 6, done.
    remote: Counting objects: 100% (6/6), done.
    remote: Compressing objects: 100% (4/4), done.
    remote: Total 6 (delta 0), reused 6 (delta 0), pack-reused 0
    Unpacking objects: 100% (6/6), 584 bytes | 58.00 KiB/s, done.
    From https://github.com/C-Bresnahan/MWGuard
        * branch
        * [new branch] 
    $
    ```
    - 如果拉取项目文件的个人在其本地存储库中有尚未上传到远程仓库的特定脚本的修改版 本， 则 git pull 命令会失败并会保护该脚本。
- 如果未参与该项目的人想得到脚本的最新版本， 那么当他们尝试使用 git remote add origin 命令时， 会收到类似于 fatal: not a git repository 的错误消息。对这些人而言， 最好先克隆该项目。
- 开发团队的新成员可以使用 git clone 命令将整个脚本项目从远程仓库复制到自己的本地 系统：
    ```
    $ git clone https://github.com/C-Bresnahan/MWGuard.git
    Cloning into 'MWGuard'...
    remote: Enumerating objects: 6, done.
    remote: Counting objects: 100% (6/6), done.
    remote: Compressing objects: 100% (4/4), done.
    remote: Total 6 (delta 0), reused 6 (delta 0), pack-reused 0
    Unpacking objects: 100% (6/6), 584 bytes | 58.00 KiB/s, done.
    $
    $ ls
    MWGuard
    $
    $ cd MWGuard/
    $
    $ ls -a
    .  ..  .git MyGitExampleScript.sh README.md
    $
    $ git log
    commit [...](HEAD -> main, origin/main, origin/HEAD)
    Author: Christine Bresnahan <cbresn1723@gmail.com>
    Date:   Mon Aug 24 15:58:52 2020 -0400
    
        README.md commit
    
    commit 3b484638bc6e391d0a1b816946cba8c5f4bbc8e6
    Author: Christine Bresnahan <cbresn1723@gmail.com>
    Date:   Mon Aug 24 15:46:56 2020 -0400
    
        Initial Commit
    $
    ```
    - 从远程仓库克隆项目时会自动创建工作目录以及.git/目录、 Git 暂存区（索引）和本地仓库。 git log 命令可以显示项目历史。

---

###### 附录A、bash 命令快速指南

1. bash 内建命令
    命令 | 描述
    ---|---
    &           | 在后台启动作业
    ((x))       | 执行数学表达式 x
    .           | 在当前 shell 中读取并执行指定文件中的命令
    :           | 什么都不做，始终成功退出
    [ t ]       | 对条件表达式 t 进行求值
    [[ e ]]     | 对条件表达式 e 进行求值
    alias       | 为指定命令定义别名
    bg          | 将当前作业置于后台运行
    bind        | 将组合键绑定到 readline 函数或宏
    Break       | 退出 for、while、select 或 until 循环
    builtin     | 执行指定的 shell 内建命令
    caller      | 返回活动子函数调用的上下文
    case        | 根据模式有选择地执行命令
    cd          | 将当前目录切换为指定的目录
    command     | 执行指定的命令，不进行正常的 shell 查找（也就是说，绕过同名的别名或函数）
    compgen     | 为指定单词生成可能的补全匹配
    complete    | 显示指定的单词是如何补全的
    compopt     | 修改指定单词的补全选项
    continue    | 继续执行 for、while、select 或 until 循环的下一次迭代
    coproc      | 在后台生成子 shell 并执行指定的命令
    declare     | 声明变量或变量类型
    dirs        | 显示当前已保存的目录列表
    disown      | 从进程作业表中删除指定的作业
    echo        | 将指定字符串输出到 STDOUT
    enable      | 启用或禁用指定的内建 shell 命令
    eval        | 将指定的参数拼接成一个命令， 然后执行该命令
    exec        | 用指定命令替换 shell 进程
    exit        | 强制 shell 以指定的退出状态码退出
    export      | 设置可用于子 shell 进程的变量
    false       | 将结果设置为 false 状态
    fc          | 从历史记录列表（history list）中选择一组命令
    fg          | 将作业恢复至前台
    for         | 对列表中的每一项执行指定的命令
    function    | 定义一个 shell 脚本函数
    getopts     | 解析指定的位置参数
    hash        | 查找并记住指定命令的完整路径名
    help        | 显示 bash 内建命令的帮助页面
    history     | 显示命令历史记录
    if          | 根据条件表达式执行命令
    jobs        | 列出活动作业
    kill        | 向指定的进程 ID（PID ）发送系统信号
    let         | 计算数学表达式
    local       | 在函数中创建局部变量
    logout      | 退出已登录的 shell
    mapfile     | 从 STDIN 中读取输入并将其放入索引数组（每个数组元素包含一行） popd        | 从目录栈中删除记录
    printf      | 使用格式化字符串显示文本
    pushd       | 向目录栈压入一个目录
    pwd         | 显示当前工作目录的完整路径名
    read        | 从 STDIN 读取一行数据并将其中的每个单词赋给指定变量
    readarray   | 从 STDIN 读取数据行并将其放入索引数组（一个数组元素对应一行）
    readonly    | 从 STDIN 读取一行数据并将其赋给一个不可修改的变量
    return      | 使函数以某个值退出，该值可由调用脚本（calling script）提取
    select      | 显示带编号的单词列表， 允许用户进行选择
    set         | 设置并显示环境变量的值和 shell 属性
    shift       | 将位置参数依次向前移动一个位置
    shopt       | 打开/关闭 shell 选项
    source      | 在当前 shell 中读取并执行指定文件中的命令
    suspend     | 暂停 shell，直至收到 SIGCONT 信号
    test        | 根据指定条件返回退出状态码 0 或 1
    time        | 显示执行指定命令所累计的真实时间（real time）、用户时间和系统时间
    times       | 显示累计的用户时间和系统时间
    trap        | 如果接收到特定的系统信号，就执行指定命令
    true        | 返回为 0 的退出状态码
    type        | 显示指定的单词作为命令名时， 如何被 shell 解释（也就是显示指定名称是外部命令、内建命令、别名、 shell 关键字或shell 函数）
    typeset     | 声明变量或变量类型
    ulimit      | 为系统用户设置指定的资源上限
    umask       | 为新建的文件和目录设置默认权限
    unalias     | 删除指定的别名
    unset       | 删除指定的环境变量或 shell 属性
    until       | 执行指定的命令，直到条件语句返回 true
    wait        | 等待指定的进程结束，返回退出状态码
    while       | 当条件语句返回 true 时，执行指定的命令
    { c; }      | 在当前 shell 中指定一组命令

- 准确地说，表中的 coproc、funtion、while 和 until 属于 shell 关键字（keyword），并非内建命令，通过 type 命令即可得知。

2. bash shell 外部命令
    命令 | 描述
    ---|---
    at          | 在未来的特定时间执行指定的脚本或命令
    atq         | 显示 at 命令队列中的作业
    atrm        | 从 at 命令队列中删除指定的作业
    bash        | 执行来自标准输入或指定文件中的命令， 或是启动一个子 shell
    bc          | 使用 bc 的专用语言执行算术运算
    bzip2       | 采用 Burrows-Wheeler 块排序文本压缩算法和霍夫曼编码进行压缩 
    cat         | 列出指定文件的内容
    chage       | 修改指定系统用户账户的密码过期日期
    chfn        | 修改指定用户账户的备注信息
    chgrp       | 修改指定文件或目录的属组
    chmod       | 修改指定文件或目录的权限
    chown       | 修改指定文件或目录的属主
    chpasswd    | 读取包含用户名/密码的文件并更新相应用户的密码
    chsh        | 修改指定用户账户的默认 shell
    clear       | 清空终端仿真器或虚拟控制台终端中的文字
    compress    | 最初的 Unix 文件压缩工具
    cp          | 将指定文件复制到另一个位置
    crontab     | 启动用户的 cron 表文件对应的编辑器（如果允许的话）
    cut         | 打印文件中指定的部分
    date        | 以各种格式显示日期
    df          | 显示所有已挂载设备的当前磁盘使用情况
    dialog      | 在文本终端环境中创建窗口对话框
    du          | 显示指定目录的磁盘使用情况
    emacs       | 调用 Emacs 文本编辑器
    env         | 在修改过的环境中执行指定命令或显示所有的环境变量
    exit        | 终止当前进程
    expr        | 执行指定的算术表达式
    fdisk       | 维护或创建指定磁盘的分区表
    file        | 查看指定文件的文件类型
    find        | 查找文件
    free        | 查看系统可用的和已用的内存
    fsck        | 检查并根据需要修复指定的文件系统
    gawk        | 调用 gawk 编辑器
    Grep        | 在文件中查找指定模式的字符串
    gedit       | 调用 GNOME 桌面编辑器
    getopt      | 解析命令选项（包括长格式选项）
    gdialog     | 创建 GNOME Shell 窗口对话框
    groups      | 显示指定用户的组成员关系
    groupadd    | 创建新的用户组
    groupmod    | 修改已有的用户组
    gunzip      | 出自 GNU 项目的文件解压缩工具，采用 Lempel-Ziv 压缩算法
    gzcat       | 出自 GNU 项目的压缩文件内容显示工具， 采用 Lempel-Ziv 压缩算法
    gzip        | 出自 GNU 项目的文件压缩工具， 采用 Lempel-Ziv 压缩算法
    head        | 显示指定文件的开头部分
    kdialog     | 创建 KDE 窗口对话框
    killall     | 根据进程名向运行中的进程发送系统信号
    kwrite      | 调用 KWrite 文本编辑器
    less        | 查看文件内容的高级命令
    link        | 使用别名创建文件链接
    ln          | 创建指定文件的符号链接或硬链接
    ls          | 列出目录内容或文件信息
    lvcreate    | 创建 LVM 卷
    lvdisplay   | 显示 LVM 卷
    lvextend    | 增加 LVM 卷的大小
    lvreduce    | 减少 LVM 卷的大小
    mandb       | 创建能够使用手册页关键字进行搜索的数据库
    man         | 显示指定命令或话题的手册页
    mkdir       | 创建指定目录
    mkfs        | 使用指定文件系统格式化分区
    mktemp      | 创建临时文件或目录
    more        | 显示指定文件的内容，每显示一屏数据后就暂停
    mount       | 显示或挂载磁盘设备到虚拟文件系统中
    mv          | 重命名文件或目录
    nano        | 调用 nano 文本编辑器
    nice        | 在系统中用指定的优先级运行命令
    nohup       | 执行指定的命令，同时忽略 SIGHUP 信号
    passwd      | 修改用户的账户密码
    printenv    | 显示指定环境变量或所有的环境变量的值
    ps          | 显示系统中运行进程的信息
    pvcreate    | 创建物理 LVM 卷
    pvdisplay   | 显示物理 LVM 卷
    Pwd         | 显示当前工作目录
    renice      | 修改系统中运行进程的优先级
    rm          | 删除指定文件或目录
    rmdir       | 删除指定的空目录
    sed         | 调用流编辑器 sed
    setterm     | 修改终端设置
    sleep       | 在指定的一段时间内暂停 bash shell 操作
    sort        | 根据指定的顺序对文件内容进行排序
    stat        | 显示指定文件的相关信息
    sudo        | 以 root 用户账户身份运行应用程序
    tail        | 显示指定文件的末尾部分
    tar         | 将数据和目录归档到单个文件中
    tee         | 将信息发送到 STDOUT 和 STDIN
    top         | 显示活动进程以及重要的系统统计数据
    touch       | 新建一个空文件或更新已有文件的时间戳
    umount      | 从虚拟文件系统中卸载磁盘设备
    uptime      | 显示系统已经运行了多久
    useradd     | 新建用户账户
    userdel     | 删除用户账户
    usermod     | 修改用户账户
    vgchange    | 激活或停用 LVM 卷组
    vgcreate    | 创建 LVM 卷组
    vgdisplay   | 显示 LVM 卷组
    vgextend    | 增加 LVM 卷组大小
    vgreduce    | 减少 LVM 卷组大小
    vgremove    | 删除 LVM 卷组
    vi          | 调用 vi 文本编辑器
    vim         | 调用 vim 文本编辑器
    vmstat      | 生成一份详尽的系统内存和 CPU 使用情况的报告
    wc          | 显示文本文件统计情况
    whereis     | 显示指定命令的相关文件，包括二进制文件、源代码文件以及手册页
    which       | 查找可执行文件的位置
    who         | 显示当前系统中的登录用户
    whoami      | 显示当前用户的用户名
    xargs       | 从 STDIN 中获取数据项， 构建并执行命令
    xterm       | 调用 xterm 终端仿真器
    zenity      | 创建 GNOME Shell 窗口小部件
    zip         | Windows PKZIP 程序的 Unix 版本

3. bash shell 环境变量
    命令 | 描述
    ---|---
    | *                     | 包含所有命令行参数（以单个文本值的形式）
    | @                     | 包含所有命令行参数（以多个文本值的形式）
    | #                     | 命令行参数数目
    | ?                     | 最近使用的前台进程的退出状态码
    | -                     | 当前命令行选项标记
    $                       | 当前 shell 的进程 ID（PID ）
    !                       | 最近执行的后台进程的 PID
    0                       | 命令行中使用的命令名
    _                       | shell 的绝对路径名
    BASH                    | 用来调用 shell 的完整路径名
    BASHOPTS                | 已启用的 shell 选项（以冒号分隔形式显示）
    BASHPID                 | 当前 bash shell 的 PID
    BASH_ALIASES            | 数组变量，包含当前所用的别名
    BASH_ARGC               | 当前函数的参数数量
    BASH_ARGV               | 数组变量，包含所有的命令行参数
    BASH_CMDS               | 数组变量，包含命令的内部哈希表
    BASH_COMMAND            | 当前正在运行的命令名
    BASH_ENV                | 如果设置的话， 每个 bash 脚本都会尝试在运行前执行由该变量定义的启动文件
    BASH_EXECUTION_STRING   | 在-c 命令行选项中指定的命令
    BASH_LINENO             | 数组变量，包含脚本中每个命令的行号
    BASH_REMATCH            | 数组变量，包含正则表达式所匹配的文本（索引为 0 的元素是整个正则表达式所匹配的部分。索引为 n 的元素是第 n 个带有圆括号的子正则表达式所匹配的部分）
    BASH_SOURCE             | 数组变量，包含 shell 中已定义函数所在源文件名
    BASH_SUBSHELL           | 当前 shell 生成的子 shell 数目
    BASH_VERSINFO           | 数组变量，包含当前 bash shell 实例的主版本号和次版本号
    BASH_VERSION            | 当前 bash shell 实例的版本号
    BASH_XTRACEFD           | 如果设置为有效的文件描述符整数，则所产生跟踪信息会与诊断和错误消息分开。文件 描述符必须事先执行 set -x
    BROWSER                 | 首选 Web 浏览器的绝对路径名
    COLUMNS                 | 当前 bash shell 实例所用的终端宽度
    COMP_CWORD              | 变量 COMP_WORDS 的索引值， COMP_WORDS 包含当前光标所在的位置
    COMP_KEY                | 调用补全功能的按键
    COMP_LINE               | 当前命令行
    COMP_POINT              | 当前光标位置相对于当前命令起始处的索引
    COMP_TYPE               | 补全类型对应的整数值
    COMP_WORDBREAKS         | 在进行单词补全时作为单词分隔符的一组字符
    COMP_WORDS              | 数组变量，包含当前命令行上的所有单词
    COMPREPLY               | 数组变量，包含可能的补全结果
    COPROC                  | 数组变量，包含用于匿名协程 I/O 的文件描述符
    DBUS_SESSION_BUS_ADDRESS | 当前登录会话的 D-Bus 地址，用于提供连接映射
    DE                      | 当前登录 shell 的桌面环境
    DESKTOP_SESSION         | 在 LXDE 环境中，包含当前登录 shell 的桌面环境
    DIRSTACK                | 数组变量，包含目录栈当前内容
    DISPLAY                 | 图形应用程序映射，用于显示图形用户界面的位置
    EDITOR                  | 定义部分 shell 命令使用的默认编辑器
    EMACS                   | 如果设置的话， shell 会认为其使用的是 Emacs shell 缓冲区，同时禁止行编辑功能   
    ENV                     | 当 shell 以 POSIX模式调用时，每个 bash 脚本在运行之前都会执行由该环境变量所定义的启动文件
    EUID                    | 当前用户的有效用户 ID （数字形式）
    FCEDIT                  | fc 命令使用的默认编辑器
    FIGNORE                 | 以冒号分隔的后缀名列表，在文件名补全时会被忽略
    FUNCNAME                | 当前执行的 shell 函数的名称
    FUNCNEST                | 嵌套函数的最高级别
    GLOBIGNORE              | 以冒号分隔的模式列表， 文件名扩展时会将其忽略
    GROUPS                  | 数组变量，包含当前用户属组
    histchars               | 控制历史记录扩展，最多可有 3 个字符
    HISTCMD                 | 当前命令在历史记录中的编号
    HISTCONTROL             | 控制哪些命令会被保存在历史记录列表中
    HISTFILE                | 保存 shell 历史记录列表的文件名（默认是~/.bash_history）
    HISTFILESIZE            | 历史记录文件（history file）能保存的最大命令数量
    HISTIGNORE              | 以冒号分隔的模式列表， 用来决定哪些命令不会被保存在历史文件中 
    HISTSIZE                | 能写入历史记录列表（history list）的最大命令数量
    HISTTIMEFORMAT          | 如果设置的话， 该变量决定了历史文件条目时间戳所使用的格式字符串 
    HOME                    | 当前登录会话的主目录名
    HOSTALIASES             | 文件名， 某些 shell 命令要用到的各种主机名别名都保存在该文件中
    HOSTFILE                | shell 在补全主机名时要读取的文件名
    HOSTNAME                | 当前主机名
    HOSTTYPE                | 当前运行 bash shell 的机器
    IFS                     | 在分割单词时作为分隔符使用的一系列字符
    IGNOREEOF               | shell 在退出前必须收到的一系列 EOF 字符的数量。如果未设置，则默认是 1
    INFODIR                 | info 命令的搜索目录列表（以冒号分隔）
    INPUTRC                 | readline 初始化的文件名（默认是~/.inputrc）
    INVOCATION_ID           | systemd 用于标识登录 shell 和其他单元的 128 位（128-bit）随机标识符
    JOURNAL_STREAM          | 文件描述符的设备和 inode 编号（十进制格式） 列表（以冒号分隔）。仅当 STDOUT 或 STDERR 连接到日志系统时才设置
    LANG                    | shell 的语言环境种类（locale category）
    LC_ALL                  | 定义语言环境种类，能够覆盖 LANG 变量
    LC_ADDRESS              | 确定地址信息的显示方式
    LC_COLLATE              | 设置字符串排序时采用的排序规则
    LC_CTYPE                | 决定如何解释出现在文件名扩展和模式匹配中的字符
    LC_IDENTIFICATION       | 包含语言环境的元数据信息
    LC_MEASUREMENT          | 设置用于测量单位的语言环境
    LC_MESSAGES             | 决定在解释前面带有$的双引号字符串时采用的语言环境设置
    LC_MONETARY             | 定义货币数值的格式
    LC_NAME                 | 设置名称的格式
    LC_NUMERIC              | 决定格式化数字时采用的语言环境设置
    LC_PAPER                | 设置用于纸张标准和格式的语言环境
    LC_TELEPHONE            | 设置电话号码的结构
    LD_LIBRARY_PATH         | 以冒号分隔的目录列表， 其中的目录会先于标准库目录被搜索
    LC_TIME                 | 决定格式化日期和时间时采用的语言环境设置
    LINENO                  | 当前正在执行的脚本语句的行号
    LINES                   | 定义了终端上可见的行数
    LOGNAME                 | 当前登录会话的用户名
    LS_COLORS               | 定义用于显示文件名的颜色
    MACHTYPE                | 用“CPU–公司–系统”（CPU-company-system）格式定义的系统类型
    MAIL                    | 如果设置的话，定义当前登录会话的邮件文件会被一些邮件程序间歇地搜索，以查找新邮件
    MAILCHECK               | shell 应该多久检查一次新邮件（以秒为单位，默认为 60 秒）
    MAILPATH                | 以冒号分隔的邮件文件名列表，一些邮件程序会间歇性地在其中搜索新邮件 
    MANPATH                 | 以冒号分隔的手册页目录列表， 由 man 命令搜索
    MAPFILE                 | 数组变量，当未指定数组变量作为参数时，其中保存了 mapfile 所读入的文本 
    OLDPWD                  | shell 先前使用的工作目录
    OPTARG                  | 包含选项所需的参数值， 由 getopts 命令设置
    OPTERR                  | 如果设置为 1 ，则 bash shell 会显示 getopts 命令产生的错误
    OPTIND                  | getopts 命令要处理的下一个参数的索引
    OSTYPE                  | 定义了 shell 所在的操作系统
    PAGER                   | 设置某些 shell 命令在查看文件时使用的分页实用工具
    PATH                    | 以冒号分隔的目录列表， shell 会在其中搜索外部命令
    PIPESTATUS              | 数组变量，包含前台进程的退出状态
    POSIXLY_CORRECT         | 如果设置的话， bash 会以 POSIX 模式启动
    PPID                    | bash shell 父进程的 PID
    PROMPT_COMMAND          | 如果设置的话， 在显示命令行主提示符之前执行该命令
    PROMPT_DIRTRIM          | 用来定义使用提示符字符串\w 或\W 转义时显示的拖尾（trailing）目录名的数量（使用一组英文句点替换被删除的目录名）
    PS0                     | 如果设置的话， 该变量会指定在输入命令之后、执行命令之前，由交互式 shell 显示的内容
    PS1                     | 主命令行提示符
    PS2                     | 次命令行提示符
    PS3                     | select 命令的提示符
    PS4                     | 在命令行之前显示的提示符（如果使用了 bash 的-x 选项的话）
    PWD                     | 当前工作目录
    RANDOM                  | 返回一个介于 0 ～ 32 767 的随机数（对该变量的赋值可作为随机数生成器的种子） 
    READLINE_LINE           | 当使用 bind –x 命令时， 保存 Readline 缓冲区的内容
    READLINE_POINT          | 当使用 bind –x 命令时， 指明了 Readline 缓冲区内容插入点的当前位置 
    REPLY                   | read 命令的默认变量
    SECONDS                 | 自 shell 启动到现在的秒数（对其赋值会重置计数器）
    SHELL                   | bash shell 的完整路径
    SHELLOPTS               | 已启用的 bash shell 选项（以冒号分隔）
    SHLVL                   | shell 的层级；每启动一个新的 bash shell，该值增加 1
    TERM                    | 登录会话当前使用的终端类型， 相关信息由该变量所指向的文件提供 
    TERMCAP                 | 登录会话当前使用的终端类型， 相关信息由该变量提供
    TIMEFORMAT              | 指定了 shell 的时间显示格式
    TMOUT                   | select 命令和 read 命令在无输入的情况下等待多久（以秒为单位。默认值为 0，表示一直等待）
    TMPDIR                  | 目录名， 保存 bash shell 创建的临时文件
    TZ                      | 如果设置的话， 用于指定系统的时区
    TZDIR                   | 定义时区文件所在的目录
    UID                     | 当前用户的真实用户 ID （数字形式）
    USER                    | 当前登录会话的用户名
    VISUAL                  | 如果设置的话， 用于定义某些 shell 命令默认使用的全屏编辑器

- 可以使用 printenv 命令来显示当前定义的环境变量。在启动时建立的 shell 环境变量在不同的 Linux 发行版中（经常）会有所不同。

---

###### 附录B、sed 和 gawk 快速指南

1. sed 编辑器
- sed 编辑器可以根据命令来操作数据流中的数据， 这些命令要么从命令行中输入，要么保存在包含命令的文本文件中。 sed 每次从输入中读取一行数据，然后按照指定的命令匹配数据、修改数据， 最后将结果输出到 STDOUT 中。

2. 启动 sed 编辑器
- sed 命令的格式如下：
    ```
    sed options script file
    ```
- options 允许定制 sed 命令的行为，sed 命令选项如下表：
    选项 | 描述
    ---|---
    -e script   | 在处理输入时， 加入 script 中指定的命令
    -f file     | 在处理输入时， 加入文件file 中包含的命令
    -n          | 不再为每条命令产生输出，而是等待打印（p）命令

- script 指定了应用于数据流的单条命令。如果用到的命令不止一条，则要么使用-e 选项在命令行上指定，要么使用-f 选项在一个单独的文件中指定。

3. sed 命令
- sed 编辑器脚本包含的命令是针对输入流中的每一行数据而执行的。
- 替换（s）命令会替换输入流中的文本。该命令的格式如下：
    ```
    s/pattern/replacement/flags
    ```
    - 其中， pattern 是要被替换的文本模式， replacement 是用于更换pattern 的新文本。
    - flags 控制如何进行替换。有 4 种类型的替换标志可用：
        - 数字：指明第几处出现的模式（pattern）应该被替换。
        - g：指明所有该模式出现的地方都应该被替换。 
     	- p：指明原始行的内容应该被打印出来。        
     	- w file：指明替换的结果应该写入文件 file 中。
    - 在第一种类型的替换中， 可以指定 sed 编辑器应该替换第几处匹配。举例来说， 可以用数字 2 来指明只替换该模式第二次出现的地方。
- 在默认情况下， sed 命令会应用于文本数据的每一行。如果想让命令只应用于指定行或某些行，则必须使用行寻址（line addressing）。
- 在 sed 编辑器中，有两种形式的行寻址。
    - 行区间（数字形式）。
    - 可以过滤出特定行的文本模式。
- 这两种形式使用相同的格式来指定地址：
    ```
    [address]command
    ```
- 当使用数字形式的行寻址时，我们通过行在文本流中的位置对其进行引用。 sed 编辑器会为数据流中的第一行分配行号 1，然后对接下来的每一行按序分配行号。要将文件的第二行或第三行中的单词“dog”替换成单词“cat”， 可以使用下列命令：
    ```
    $ sed '2,3s/dog/cat/' data1
    ```
- sed 编辑器允许指定文本模式来过滤所需要的行。格式如下：
    ```
    /pattern/command
    ```
    - 必须将 pattern 放入一对正斜线之间。 sed 编辑器会将 command 应用于匹配文本模式的那些行：
        ```
        $ sed '/rich/s/bash/csh/' /etc/passwd
        ```
        - 这个过滤器会找到含有文本 rich 的行，然后用文本 csh 替换bash。
- 也可以针对某个特定地址应用多条命令：
    ```
    address {
        command1
        command2
        command3 }
    ```
    - sed 编辑器会将你指定的所有命令都应用于匹配指定地址的行。它会处理地址行上列出的每条命令：
        ```
        $ sed '2{
        > s/fox/elephant/
        > s/dog/cat/
        > }' data1
        ```
        - sed 编辑器会将每一条替换命令都应用于数据文件的第二行。
- 删除（d）命令会删除匹配指定地址的所有行。使用删除命令时要小心，如果忘记加地址的话， 数据流中的所有行都会被删除。
- 可以通过以下方式从数据流中删除特定的文本行：
    ```
    要么通过行号指定地址：
    $ sed '3d' data1
    
    要么通过行区间指定地址：
    $ sed '2,3d' data1
    ```
- sed 编辑器的模式匹配特性也适用于删除命令：
    ```
    $ sed '/number 1/d' data1
    ```
    - 只有匹配指定模式的行才会被删除。
- sed 编辑器同样允许向数据流中插入文本行和附加文本行：
    - 插入（i）命令会在指定行前面添加一个新行。
    - 附加（a）命令会在指定行后面添加一个新行。
    - 要注意这两个命令的格式： 不能在单个命令行上使用它们。
- 要插入或附加的行必须作为单独 的一行出现。该命令格式如下：
    ```
    sed ' [address]command\
    new line '
    ```
    - new line 中的文本会按照指定的位置出现在 sed 编辑器的输出中。
- 修改（c）命令可以修改数据流中的整行文本，其格式跟插入命令和附加命令一样，必须将 新行与 sed 命令的其余部分分开：
    ```
    $ sed '3c\
    > This is a changed line of text.' data1
    ```
    - 反斜线字符用来表明脚本中的新数据行。
- 转换（y）命令是唯一应用于单个字符的 sed 命令。该命令格式如下：
    ```
    [address]y/inchars/outchars/
    ```
    - 转换命令对 inchars 和 outchars 执行一对一的映射。 inchars 中的第一个字符会转换为 outchars 中的第一个字符， inchars 中的第二个字符会转换为 outchars 中的第二个字符，以此类推，直到超过了指定字符的长度。如果 inchars 和 outchars 长度不同，则 sed 编辑器会报错。
- 打印（p）命令会打印 sed 编辑器输出中的一行。打印命令最常见的用法是打印匹配指定模式的文本行：
    ```
    $ sed -n '/number 3/p' data1
    This is line number 3.
    $
    ```
    - 打印命令可以只打印输入数据流中的特定行。
- 写入（w）命令可用于将文本行写入文件。该命令格式如下：
    ```
    [address]w filename
    ```
    - filename 可以用相对路径或绝对路径指定， 但不管怎样， 运行 sed 编辑器的用户必须拥有该文件的写权限。 address 可以是任意类型的寻址方法， 比如行号、文本模式、行区间或模式区间。
- 读取（r）命令可以插入单个文件中的数据。该命令格式如下：
    ```
    [address]r filename
    ```
    - 其中， filename 使用相对路径名或绝对路径名的形式来指定含有数据的文件。读取命令不能使用地址区间，只能使用单个行号或模式地址， 然后 sed 编辑器会将文件中的文本插入指定地址之后：
        ```
        $ sed '3r data' data1
        ```
        - sed 编辑器将 data 文件中的全部文本都插到了data1 文件的第三行之后。

4. gawk 命令格式
- gawk 程序的基本格式如下：
    ```
    gawk options program file
    ```
- 可用的 gawk 选项：
    选项 | 描述
    ---|---
    -F fs           | 指定用于划分行中各个数据字段的字段分隔符
    -f file         | 指定要从哪个文件中读取脚本
    -v var=value    | 定义 gawk 脚本中要使用的变量及其默认值
    -mf N           | 指定数据文件中的最大字段数
    -mr N           | 指定数据文件中的最大记录数
    -W keyword      | 指定 gawk 的兼容模式或警告等级。使用 help 选项列出所有可用的关键字

5. 使用 gawk
- 既可以直接通过命令行使用 gawk，也可以在 shell 脚本中使用 gawk。
- 从命令行读取脚本
    - gawk 脚本是由一对花括号定义的， 必须将脚本命令放在两个花括号之间。
    - 由于 gawk 命令行会假定脚本是单个字符串， 因此必须用单引号将脚本引用起来。
    - 一个在命令行上指定的简单 gawk脚本：
        ```
        $ gawk '{print $1}'
        ```
        - 这个脚本会显示输入流中每一行的第一个数据字段。
- 在脚本中使用多条命令
    - gawk 编程语言允许将多条命令组合成一个脚本。要在命令行上指定的脚本中使用多条命令，只需在每条命令之间加上一个分号即可。
        ```
        $ echo "My name is Rich" | gawk '{$4="Dave"; print $0}'
        My name is Dave
        $
        ```
        - 这个脚本执行了两条命令：先用另一个值替换第四个数据字段，然后显示流中的整行数据。
- 从文件中读取脚本
    - gawk 编辑器允许将脚本保存在文件中， 然后在命令行上引用脚本文件：
        ```
        $ cat script1
        { print $5 "'s userid is " $1 }
        $ gawk -F: -f script1 /etc/passwd
        ```
        - gawk 对输入数据流执行了指定文件中的所有命令。
- 在处理数据前运行脚本
    - gawk 还允许指定何时运行脚本。可以使用 BEGIN 关键字。它会强制 gawk 先执行 BEGIN 关键字后面指定的脚本，再读取数据：
        ```
        $ gawk 'BEGIN {print "This is a test report"}'
        This is a test report
        $
        ```
        - 可以在 BEGIN 部分放置任意的 gawk 命令，比如给变量设置默认值。
- 在处理数据后运行脚本
    - END 关键字允许指定在 gawk 读取数据后执行的脚本：
        ```
        $ gawk 'BEGIN {print "Hello World!"} {print $0} END {print
        "byebye"}' data1
        Hello World!
        This is a test
        This is a test
        This is another test.
        This is another test.
        byebye
        $
        ```
        - gawk 会先执行 BEGIN 部分的代码，然后处理输入流中的数据，最后执行 END 部分的代码。

6. gawk 变量
- gawk 脚本使用内建变量来引用特定的数据信息。
- gawk 脚本会将数据定义为记录和数据字段。记录是一行数据（默认以换行符分隔），而数据字段则是行中独立的数据元素（默认以空白字符分隔，比如空格或制表符）。
- gawk 脚本使用数据字段来引用每条记录中的数据元素。
- gawk 数据字段和记录变量：
    变量 | 描述
    ---|---
    $0          | 整条记录
    $1          | 记录中的第一个数据字段
    $2          | 记录中的第二个数据字段
    $n          | 记录中的第 n 个数据字段
    FIELDWIDTHS | 由空格分隔的数字列表，定义了每个数据字段的具体宽度
    FS          | 输入字段分隔符
    RS          | 输入记录分隔符
    OFS         | 输出字段分隔符
    ORS         | 输出记录分隔符
 
- 更多的 gawk 内建变量：
    变量 | 描述
    ---|---
    ARGC        | 当前命令行参数的个数
    ARGIND      | 当前文件在 ARGV 数组中的索引
    ARGV        | 包含命令行参数的数组
    CONVFMT     | 数字的转换格式（参见 printf 语句），默认值为%.6g
    ENVIRON     | 由当前 shell 环境变量及其值组成的关联数组
    ERRNO       | 当读取或关闭输入文件发生错误时的系统错误号
    FILENAME    | 作为 gawk 输入的数据文件的文件名
    FNR         | 当前数据文件中的记录数
    IGNORECASE  | 如果设置成非 0 值，则 gawk 会忽略所有字符串函数（包括正则表达式）中的字符大小写 
    NF          | 数据文件中的数据字段总数
    NR          | 已处理的输入记录数
    OFMT        | 数字的输出格式，默认值为%.6g
    RLENGTH     | 由 match 函数所匹配的子串的长度
    RSTART      | 由 match 函数所匹配的子串的起始位置
    
    - 可以在 gawk 脚本中的任何位置使用内建变量，包括 BEGIN 和 END 部分。
- 在 gawk 脚本中给变量赋值需要使用赋值语句：
    ```
    $ gawk '
    > BEGIN{
    > testing="This is a test"
    > print testing
    > }'
    This is a test
    $
    ```
    - 给变量赋值后，就可以在 gawk 脚本中的任何位置使用该变量了。
- 也可以在 gawk 命令行上给变量赋值。这允许在正常脚本之外即时设置或修改变量值。下面的例子使用命令行变量显示文件中特定数据字段：
    ```
    $ cat script1
    BEGIN{FS=","}
    {print $n}
    $ gawk -f script1 n=2 data1
    $ gawk -f script1 n=3 data1
    ```
    - 这个特性是在 gawk 脚本中处理 shell 脚本数据的一种好方法。

7. gawk 程序特性
- 可以使用基础正则表达式（BRE）或扩展正则表达式（ERE）从数据流中过滤出脚本要处理 的文本行。
- 正则表达式在使用时必须出现在与其对应的脚本命令的左花括号之前。
    ```
    $ gawk 'BEGIN{FS=","} /test/{print $1}' data1
    This is a test
    $
    ```
- 匹配运算符可以将正则表达式限制在数据行中的特定数据字段。匹配运算符是波浪号（~）。你可以指定匹配运算符、数据字段变量以及要匹配的正则表达式：
    ```
    $1 ~ /^data/
    ```
    - 这个表达式会过滤出第一个数据字段以文本 data 开头的记录。
- 还可以在匹配模式中使用数学表达式。这个功能在匹配数据字段中的数字值时非常有用。如果要显示所有属于 root用户组（组 ID 为 0）的用户，可以使用下列脚本：
    ```
    $ gawk -F: '$4 == 0{print $1}' /etc/passwd
    ```
    - 这个脚本会找出第四个数据字段值为 0 的所有行，显示出这些行的第一个数据字段。
- gawk 脚本支持多种结构化命令：
    ```
    if-then-else 语句：
    if (condition) statement1; else statement2
    
    while 语句：
    while (condition)
    {
        statements
    }
    
    do-while 语句：
    do {
        statements
    } while (condition)
    
    for 语句：
    for(variable assignment; condition; iteration process)
    ```
