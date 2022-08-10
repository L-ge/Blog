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
