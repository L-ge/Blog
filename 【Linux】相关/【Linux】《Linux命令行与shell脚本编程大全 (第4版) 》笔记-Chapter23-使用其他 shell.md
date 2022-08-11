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
