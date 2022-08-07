###### 十二、结构化命令

1. 使用 if-then 语句
- 最基本的结构化命令是 if-then 语句。if-then 语句的格式如下：        
    ```
    if command
    then
        commands
    fi
    ```
- bash shell 的 if 语句会运行 if 之后的命令。如果该命令的退出状态码为 0（命令成功运行）， 那么位于 then 部分的命令就会被执行。如果该命令的退出状态码是其他值，则 then 部分的命令不会被执行，bash shell 会接着处理脚本中的下一条命令。
- fi 语句用来表示 if-then 语句到此结束。
- 如果在 if 语句中故意使用了一个不存在的命令 IamNotaCommand，if 语句中的那个错误命令所产生的错误消息依然会显示在脚本的输出中。
- 通过把分号（ ; ）放在待求值的命令尾部，可以将 then 语句写在同一行，这样看起来更像其他编程语言中的 if-then 语句。
    ```
    if command; then
        commands
    fi
    ```
- 能出现在 then 部分的命令可不止一条。可以像脚本中其他地方一样在这里列出多条命令。 bash shell 会将这些命令视为一个代码块，如果 if 语句行命令的退出状态值为 0，那么代码块中 所有的命令都会被执行；否则，会跳过整个代码块。

2. if-then-else 语句
- if-then-else 语句在语句中提供了另外一组命令：
    ```
    if command
    then
        commands
    else
        commands
    fi
    ```
- 当 if 语句中的命令返回退出状态码 0 时， then 部分中的命令会被执行；当 if 语句中的命令返回非 0 退出状态码时， bash shell 会执行 else 部分中的命令。
- 跟 then 部分一样， else 部分可以包含多条命令。fi 语句说明 else 部分结束。

3. 嵌套 if 语句
- ls 命令的另一些使用选项（以及选项组合）：
    - -d：只显示目录，但不显示目录内容。
    - -sh：以人类易读的格式显示文件大小。
    - -g：在文件长列表中不显示文件属主。
    - -o：在文件长列表中不显示文件属组。
- 嵌套部分位于主 if-then-else 语句的 else 代码块中时，代码不易阅读。
    ```
    if command1
    then
        commands
    else
        if command2
        then 
            more commands
        fi
    fi
    ```
- 可以使用 else 部分的另一种名为 elif 的形式， 这样就不用再写多个 if-then 语句了。 elif 使用另一个 if-then 语句延续 else 部分：
    ```
    if command1
    then
        commands
    elif command2
    then
        more commands
    fi
    ```
    - 如果 elif 之后的命令的退出状态码是 0，则 bash 会执行第二个 then 语句部分的命令。这种嵌套形式使得代码更清晰，逻辑更易懂。
- 可以通过在嵌套 elif 中加入一个 else 语句：    
    ```
    if command1
    then
        commands
    elif command2
        then
            more commands1
        else
            more commands2  
    fi
    ```
    - 在 elif 语句中， 紧跟其后的 else 语句语句的代码块属于 elif 代码块，不属于之前的 if-then 语句的代码块。
- 可以继续将多个 elif 语句串起来， 形成一个更大的 if-then-elif 嵌套组合：
    ```
    if command1
    then
        command set 1
    elif command2
    then
        command set 2
    elif command3
    then
        command set 3
    elif command4
    then
        command set 4
    fi
    ```
    - 每个代码块会根据命令是否会返回退出状态码 0 来执行。
    - bash shell 会依次执行 if 语句，只有第一个返回退出状态码 0 的语句中的 then 部分会被执行。

4. test 命令
- test 命令可以在 if-then 语句中测试不同的条件。如果 test 命令中列出的条件成立， 那么 test 命令就会退出并返回退出状态码 0。这样 if-then 语句的工作方式就和其他编程语言中的 if-then 语句差不多了。如果条件不成立，那么 test 命令就会退出并返回非 0 的退出状态码， 这使得 if-then 语句不会再被执行。
- test 命令的格式非常简单：
    ```
    test condition
    
    if test condition
    then
        commands1
    else
        commands2
    fi
    ```
    - condition 是 test 命令要测试的一系列参数和值。
    - 如果不写 test 命令的 condition 部分，则它会以非 0 的退出状态码退出并执行 else 代码块语句。
    - 如果加入了条件，则 test 命令会测试该条件。
- 可以使用 test 命令确定变量中是否为空。这只需要一个简单的条件表达式：
    ```
    $ cat test6.sh
    #!/bin/bash
    # testing if a variable has content
    #
    my_variable="Full"      // 情况二：my_variable="" 
    #
    if test $my_variable
    then
        echo "The my_variable variable has content and returns a True."
        echo "The my_variable variable content is: $my_variable"
    else
        echo "The my_variable variable doesn't have content,"
        echo "and returns a False."
    fi
    $
    $ ./test6.sh
    The my_variable variable has content and returns a True.
    The my_variable variable content is: Full
    $
    ```
    - 由于变量 my_variable 中包含内容（ Full），因此当 test 命令测试条件时，返回的退出状态码为 0。这使得 then 语句块中的语句得以执行。
- bash shell 提供了另一种条件测试方式， 无须在 if-then 语句中写明 test 命令：
    ```
    if [ condition ]
    then
        commands
    fi
    ```
    - 方括号定义了测试条件。**注意，第一个方括号之后和第二个方括号之前必须留有空格， 否则 就会报错。**
- test 命令和测试条件可以判断 3 类条件：
    - 数值比较
    - 字符串比较
    - 文件比较

5. 数值比较
- test 命令的数值比较功能：
    比较 | 描述
    ---|---
    n1 -eq n2 | 检查 n1 是否等于 n2
    n1 -ge n2 | 检查 n1 是否大于或等于 n2
    n1 -gt n2 | 检查 n1 是否大于 n2
    n1 -le n2 | 检查 n1 是否小于或等于 n2
    n1 -lt n2 | 检查 n1 是否小于 n2
    n1 -ne n2 | 检查 n1 是否不等于 n2

- 数值条件测试可用于数字和变量。
    ```
    if [ $value1 -gt 5 ]
    if [ $value1 -eq $value2 ]
    ```
- 对于条件测试， bash shell 只能处理整数。尽管可以将浮点值用于某些命令（比如 echo），但它们在条件测试下无法正常工作。

6. 字符串比较
- 条件测试还允许比较字符串值。
- test 命令的字符串比较功能：
    比较 | 描述
    ---|---
    str1 = str2     | 检查 str1 是否和 str2 相同
    str1 != str2    | 检查 str1 是否和 str2 不同
    str1 < str2     | 检查 str1 是否小于 str2
    str1 > str2     | 检查 str1 是否大于 str2
    -n str1         | 检查 str1 的长度是否不为 0
    -z str1         | 检查 str1 的长度是否为 0

- 字符串相等性
    ```
    if [ $testuser = christine ]
    if [ $testuser != christine ]
    ```
    - 在比较字符串的相等性时，比较测试会将所有的标点和大小写情况都考虑在内。
- 字符串顺序
    - 使用测试条件的大于或小于功能时，会出现两个经常困扰 shell 程序员的问题：
        - 大于号和小于号必须转义， 否则 shell 会将其视为重定向符，将字符串值当作文件名。
        - 大于和小于顺序与 sort 命令所采用的不同。
        ```
        if [ $string1 > $string2 ]          // 没有报错，但结果与预期结果不符。需要使用反斜线（\）正确地转义大于号。
        if [ $string1 \> $string2 ]
        ```
    - 字符串 soccer 小于 zorbfootball，因为在比较的时候使用的是每个字符的 Unicode 编码值。
- sort 命令处理大写字母的方法刚好与 test 命令相反：
    - 在比较测试中，大写字母被认为是小于小写字母的。但 sort 命令正好相反。当你将同样的 字符串放进文件中并用 sort 命令排序时，小写字母会先出现。
    - 比较测试中使用的是标准的 Unicode 顺序， 根据每个字符的 Unicode 编码值来决定排序结果。 sort 命令使用的是系统的语言环境设置中定义的排序顺序。对于英语，语言环境设置指定了在排序顺序中小写字母出现在大写字母之前。
- test 命令和测试表达式使用标准的数学比较符号来表示字符串比较，而用文本代码来表示数值比较。
- 字符串大小
    - -n 和-z 可以很方便地用于检查一个变量是否为空：
        ```
        if [ -n $string1 ]
        if [ -z $string2 ]
        ```
        - shell 脚本中并未定义该变量时，长度也视为 0，尽管它未被定义过。
- 空变量和未初始化的变量会对 shell 脚本测试造成灾难性的影响。如果不确定变量的内容，那么最好在将其用于数值或字符串比较之前先通过-n 或-z 来测试一下变量是否为空。

7. 文件比较
- 比较测试允许测试 Linux 文件系统中文件和目录的状态。
- test 命令的文件比较功能
    比较 | 描述
    ---|---
    -d file         | 检查 file 是否存在且为目录
    -e file         | 检查 file 是否存在
    -f file         | 检查 file 是否存在且为文件
    -r file         | 检查 file 是否存在且可读
    -s file         | 检查 file 是否存在且非空
    -w file         | 检查 file 是否存在且可写
    -x file         | 检查 file 是否存在且可执行
    -O file         | 检查 file 是否存在且属当前用户所有
    -G file         | 检查 file 是否存在且默认组与当前用户相同 
    file1 -nt file2 | 检查 file1 是否比 file2 新
    file1 -ot file2 | 检查 file1 是否比 file2 旧

- 检查目录：-d 测试会检查指定的目录是否存在于系统中。如果打算将文件写入目录或是准备切换到某个目录， 那么先测试一下总是件好事：
    ```
    #!/bin/bash
    jump_directory=/home/Torfa
    if [ -d $jump_directory ]
    then
        cd $jump_directory
        ls
    else
        echo "The $jump_directory directory does NOT exist."
    fi
    ```
- 检查对象是否存在：-e 测试允许在使用文件或目录前先检查其是否存在：
    ```
    location=$HOME
    file_name="sentinel"
    if [ -d $location ]
    if [ -e $location/$file_name ]
    ```
- 检查文件：-e 测试可用于文件和目录。如果要确定指定对象为文件，那就必须使用-f 测试：
    ```
    object_name=$HOME           // object_name=$HOME/sentinel
    if [ -e $object_name ]
    if [ -f $object_name ]
    ```
- 检查是否可读：在尝试从文件中读取数据之前，最好先使用-r 测试检查一下文件是否可读：
    ```
    pwfile=/etc/shadow
    if [ -f $pwfile ]
    if [ -r $pwfile ]
    ```
- 检查空文件：应该用-s 测试检查文件是否为空， 尤其是当你不想删除非空文件时。要当心，如果-s 测试 成功，则说明文件中有数据：
    ```
    file_name=$HOME/sentinel
    if [ -f $file_name ]
    if [ -s $file_name ]
    ```
- 检查是否可写：-w 测试可以检查是否对文件拥有可写权限：
    ```
    item_name=$HOME/sentinel
    if [ -f $item_name ]
    if [ -w $item_name ]
    ```
- 检查文件是否可以执行：-x 测试可以方便地判断文件是否有执行权限。虽然可能大多数命令用不到它，但如果想在 shell 脚本中运行大量程序， 那就得靠它了：
    ```
    item_name=$HOME/scripts/can-I-write-to-it.sh
    if [ -x $item_name ]
    ```
- 检查所有权：-O 测试可以轻松地检查你是否是文件的属主：
    ```
    if [ -O /etc/passwd ]
    ```
    - 使用-O 测试来检查运行脚本的用户是否是/etc/passwd 文件的属主。
- 检查默认属组关系：-G 测试可以检查文件的属组， 如果与用户的默认组匹配，则测试成功。 -G 只会检查默认组而非用户所属的所有组：
    ```
    if [ -G $HOME/TestGroupFile ]
    ```
- 检查文件日期：
    - 比较两个文件的创建日期。这在编写软件安装脚本时非常有用。有时， 你 可不想安装一个比系统中已有文件还要旧的文件。
    - -nt 测试会判定一个文件是否比另一个文件更新。如果文件较新， 那意味着其文件创建日期 更晚。 -ot 测试会判定一个文件是否比另一个文件更旧。如果文件较旧， 则意味着其文件创建日期更早。
    ```
    if [ $HOME/Downloads/games.rpm -nt $HOME/software/games.rpm ]
    ```
    - 如果有其中一个文件不存在， 那么-nt测试返回的信息就不正确。在-nt 或-ot 测试之前，务必确保文件存在。
    
8. 复合条件测试
- if-then 语句允许使用布尔逻辑将测试条件组合起来。可以使用以下两种布尔运算符：
    - [ condition1 ] && [ condition2 ]
    - [ condition1 ] || [ condition2 ]
- 布尔逻辑是一种将可能的返回值简化（reduce）为真（TRUE）或假（FALSE）的方法。

9. if-then 的高级特性
- bash shell 还提供了 3 个可在 if-then 语句中使用的高级特性:
    - 在子 shell 中执行命令的单括号。
    - 用于数学表达式的双括号。
    - 用于高级字符串处理功能的双方括号。

10. 使用单括号
- 单括号允许在 if 语句中使用子 shell。单括号形式的 test 命令格式如下：
    ```
    (command)
    ```
- 在 bash shell 执行 command 之前，会先创建一个子 shell，然后在其中执行命令。如果命令成功结束， 则退出状态码会被设为 0，then 部分的命令就会被执行。如果命令的退出状态码不为 0，则不执行 then 部分的命令。
    ```
    echo $BASH_SUBSHELL
    if (echo $BASH_SUBSHELL)
    then
        xxx
    else
        xxx
    fi
    ```
    - 当脚本第一次（在 if 语句之前）执行 echo $BASH_SUBSHELL 命令时，是在当前 shell 中完成的。该命令会输出 0，表明没有使用子 shell。在 if 语句内，脚本在子 shell 中执行 echo $BASH_SUBSHELL 命令，该命令会输出 1，表明使用了子 shell 。
- 当你在 if test 语句中使用进程列表时，可能会出现意料之外的结果。哪怕进程列表中除最后一个命令之外的其他命令全都失败，子 shell 仍会将退出状态码设为 0，then 部分的命令将得以执行。

11. 使用双括号
- 双括号命令允许在比较过程中使用高级数学表达式。 test 命令在进行比较的时候只能使用 简单的算术操作。双括号命令提供了更多的数学符号。双括号命令的格式如下：
    ```
    (( expression ))
    ```
    - expression 可以是任意的数学赋值或比较表达式。
- 除了test 命令使用的标准数学运算符，下表还列出了双括号中可用的其他运算符。双括号命令符号：
    符号 | 描述
    ---|---
    val++   | 后增
    val--   | 后减
    ++val   | 先增
    --val   | 先减
    !       | 逻辑求反
    ~       | 位求反
    **      | 幂运算
    <<      | 左位移
    >>      | 右位移
    &       | 位布尔 AND
    \|      | 位布尔 OR
    &&      | 逻辑 AND
    \|\|    | 逻辑 OR

- 双括号命令既可以在 if 语句中使用，也可以在脚本中的普通命令里用来赋值。
    ```
    val1=10
    if (( $val1 ** 2 > 90 ))
    then
        (( val2 = $val1 ** 2 ))
    fi
    ```
    - 注意，双括号中表达式的大于号不用转义。

12. 使用双方括号
- 双方括号命令提供了针对字符串比较的高级特性。双方括号的格式如下：
    ```
    [[ expression ]]
    ```
    - expression 可以使用 test 命令中的标准字符串比较。除此之外， 它还提供了 test 命令 所不具备的另一个特性——模式匹配。
- 双方括号在 bash shell 中运行良好。不过要小心， 不是所有的 shell 都支持双方括号。
- 在进行模式匹配时，可以定义通配符或正则表达式来匹配字符串：
    ```
    if [[ $BASH_VERSION == 5.* ]]
    ```
    - 双等号会将右侧的字符串（ 5.* ）视为一个模式并应用模式匹配规则。双方括号命令会对$BASH_VERSION 环境变量进行匹配，看是否以字符串 5.起始。 如果是，则测试通过，shell 会执行 then 部分的命令。
- 当在双中括号内使用==运算符或!=运算符时，运算符的右侧被视为通配符。如果使用的是=~运算符，则运算符的右侧被视为 POSIX 扩展正则表达式。

13. case 命令
- 有了 case 命令，就无须再写大量的 elif 语句来检查同一个变量的值了。 case 命令会采用列表格式来检查变量的多个值：
    ```
    case variable in
    pattern1 | pattern2) commands1;;
    pattern3) commands2;;
    *) default commands;;
    esac
    ```
    - case 命令会将指定变量与不同模式进行比较。如果变量与模式匹配，那么shell 就会执行为 该模式指定的命令。你可以通过竖线运算符在一行中分隔出多个模式。星号会捕获所有与已知模 式不匹配的值。
- 下面是一个将 if-then-else 程序转换成使用 case 命令的例子：
    ```
    $ cat ShortCase.sh
    #!/bin/bash
    # Using a short case statement
    #
    case $USER in
    rich | christine)
        echo "Welcome $USER"
        echo "Please enjoy your visit.";;
    barbara | tim)
        echo "Hi there, $USER"
        echo "We're glad you could join us.";;
    testing)
        echo "Please log out when done with test.";;
    *)
        echo "Sorry, you are not allowed here."
    esac
    $
    $ ./ShortCase.sh
    Welcome christine
    Please enjoy your visit.
    $
    ```

14. 输出重定向出现在单括号内的 which 命令之后。which 命令的常规（标准） 输出和标准错误信息都通过&>符号被重定向至/dev/null，这个地方被幽默地称为 黑洞，因为被送往这里的东西从来都是有去无回。
    ```
    if (which yum &> /dev/null)     // 检查是否有 yum 工具
    ```
