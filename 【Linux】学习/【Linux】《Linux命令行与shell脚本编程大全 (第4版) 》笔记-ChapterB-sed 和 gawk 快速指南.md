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
