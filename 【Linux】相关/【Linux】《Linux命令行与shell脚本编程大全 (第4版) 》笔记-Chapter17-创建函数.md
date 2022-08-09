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
