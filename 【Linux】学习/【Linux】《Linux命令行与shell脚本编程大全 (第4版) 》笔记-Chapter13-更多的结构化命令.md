###### 十三、更多的结构化命令

1. for 命令
- bash shell 提供了 for 命令，以允许创建遍历一系列值的循环。每次迭代都使用其中一个值来执行已定义好的一组命令。for 命令的基本格式如下：
    ```
    for var in list
    do
        commands
    done
    ```
    - 需要提供用于迭代的一系列值作为 list 参数。指定这些值的方法不止一种。
    - 在每次迭代中，变量 var 会包含列表中的当前值。第一次迭代使用列表中的第一个值，第二次迭代使用列表中的第二个值，以此类推，直到用完列表中的所有值。
    - do 语句和done 语句之间的 commands 可以是一个或多个标准的 bash shell 命令。在这些命令中， $var 变量包含着此次迭代对应的列表中的当前值。
    - 也可以将 do 语句和 for 语句放在同一行，但必须用分号将其同列表中的值分开： for var in list; do。

2. 读取列表中的值
- for 命令最基本的用法是遍历其自身所定义的一系列值：
    ```
    for test in Alabama Alaska Arizona Arkansas California Colorado
    do
        echo The next state is $test
    done
    echo "The last state we visited was $test"
    ```
    - 每次遍历值列表时，for 命令会将列表中的下一个值赋给$test 变量。
    - 在最后一次迭代结束后， $test 变量的值在 shell 脚本的剩余部分依然有效。它会一直保持最后一次迭代时的值（除非做了修改）。
    - $test 变量保持着它的值， 也允许我们对其做出修改， 在 for 循环之外跟其他变量一样使用。

3. 读取列表中的复杂值
- 有两种方法可以解决列表值中的单引号的解析问题：
    - 使用转义字符（反斜线）将单引号转义。
    - 使用双引号来定义含有单引号的值。
    ```
    for test in I don't know if this'll work
    for test in I don\'t know if "this'll" work
    ```
- for 循环假定各个值之间是以空格分隔的（准确地说，既可以是空格，也可以是制表符或换行符）。
- 如果某个值含有空格，则必须将其放入双引号内：
    ```
    for test in Nevada New Hampshire New Mexico New York North Carolina
    for test in Nevada "New Hampshire" "New Mexico" "New York"
    ```
    - 当使用双引号引用某个值时， shell 并不会将双引号当成值的一部分。

4. 从变量中读取值列表
- 将一系列值集中保存在了一个变量中，然后需要遍历该变量中的整个值列表。可以通过 for 命令完成这个任务：
    ```
    list="Alabama Alaska Arizona Arkansas Colorado"
    list=$list" Connecticut"
    for state in $list
    ```
    - $list 变量包含了用于迭代的值列表。
    - 注意，脚本中还使用了另一个赋值语句向 $list 变量包含的值列表中追加（或者说是拼接）了一项。这是向变量中已有的字符串尾部添加文本的一种常用方法。

5. 从命令中读取值列表
- 生成值列表的另一种途径是使用命令的输出。可以用命令替换来执行任何能产生输出的命令，然后在 for 命令中使用该命令的输出：
    ```
    file="states.txt"
    for state in $(cat $file)
    
    $ cat states.txt
    Alabama
    Alaska
    Arizona
    
    $ ./test5
    Visit beautiful Alabama
    Visit beautiful Alaska
    Visit beautiful Arizona
    ```
    - 这个例子在命令替换中使用 cat 命令来输出文件 states.txt 的内容。注意，states.txt 文件中每个值各占一行，而不是以空格分隔。for 命令仍然以每次一行的方式遍历 cat 命令的输出。但这并没有解决数据中含有空格的问题。如果你列出了一个名字中有空格的州，则 for 命令仍然会用空格来分隔值。
    - 这个例子将不包含路径的文件名赋给了变量。这要求文件和脚本位于同一个目录中。如果并非如此， 则需要使用完整路径名（不管是绝对路径还是相对路径）来引用文件位置。
    
6. 更改字段分隔符
- 造成这个问题的原因是特殊的环境变量 IFS（internal field separator，内部字段分隔符）。 IFS 环境变量定义了 bash shell 用作字段分隔符的一系列字符。在默认情况下，bash shell 会将下列字符视为字段分隔符。
    - 空格
    - 制表符
    - 换行符
- 如果 bash shell 在数据中看到了这些字符中的任意一个，那么它就会认为这是列表中的一个新字段的开始。在处理可能含有空格的数据（比如文件名）时，这就很烦人了。解决这个问题的办法是在 shell脚本中临时更改 IFS 环境变量的值来限制被bash shell视为字段分隔符的字符。如果想修改 IFS 的值，使其只能识别换行符，可以这么做：
    ```
    IFS=$'\n'
    
    file="states.txt"
    IFS=$'\n'
    for state in $(cat $file)
    ```
    - 将该语句加入脚本，告诉 bash shell 忽略数据中的空格和制表符。
- 在处理代码量较大的脚本时，可能在一个地方需要修改 IFS 的值，然后再将其恢复原状，而脚本的其他地方则继续沿用 IFS 的默认值。一种安全的做法是在修改 IFS 之前保存原来的 IFS 值，之后再恢复它。这种技术可以像下面这样来实现：
    ```
    IFS.OLD=$IFS
    IFS=$'\n'
    <在代码中使用新的 IFS 值>
    IFS=$IFS.OLD
    ```
- 如果要遍历文件中以冒号分隔的值（比如/etc/passwd 文件），则只需将 IFS 的值设为冒号即可：
    ```
    IFS=:
    ```
- 如果要指定多个 IFS 字符，则只需在赋值语句中将这些字符写在一起即可：
    ```
    IFS=$'\n:;"'
    ```
    - 该语句会将换行符、冒号、分号和双引号作为字段分隔符。如何使用 IFS 字符解析数据没有任何限制。
    
7. 使用通配符读取目录
- 可以用 for 命令来自动遍历目录中的文件。为此，必须在文件名或路径名中使用通配符，这会强制 shell 使用文件名通配符匹配（file globbing）。文件名通配符匹配是生成与指定通配符匹配的文件名或路径名的过程。
    ```
    for file in /home/rich/test/*
    do
        if [ -d "$file" ]
        then
            echo "$file is a directory"
        elif [ -f "$file" ]
        then
            echo "$file is a file"
        fi
    done
    ```
    - for 命令会遍历/home/rich/test/*匹配的结果。
    - 在 Linux 中，目录名和文件名中包含空格是完全合法的。要应对这种情况，应该将$file 变量放入双引号内。否则，遇到含有空格的目录名或文件名时会产生错误。在 test 命令中， bash shell 会将额外的单词视为参数，引发错误。
- 也可以在 for 命令中列出多个目录通配符：
    ```
    for file in /home/rich/.b* /home/rich/badtest
    ```
    - for 语句首先遍历了由文件名通配符匹配生成的文件列表，然后遍历了列表中的下一个文件。
    - 可以将任意多的通配符放进列表中。
- 注意，可以在值列表中放入任何东西。即使文件或目录不存在， for 语句也会尝试把列表处理完。如果是和文件或目录打交道，那就要出问题了。你无法知道正在遍历的目录是否存在：最好在处理之前先测试一下文件或目录。

8. C 语言中的 for 命令
- C 语言中的 for 命令包含循环变量初始化、循环条件以及每次迭代时修改变量的方法。当指 定的条件不成立时， for 循环就会停止。迭代条件使用标准的数学符号定义。
- bash shell 也支持 for 循环，bash 中仿 C 语言的 for 循环的基本格式如下：
    ```
    for (( variable assignment ; condition ; iteration process ))
    ```
- 注意，有些地方与 bash shell 标准的 for 命令并不一致。
    - 变量赋值可以有空格。
    - 迭代条件中的变量不以美元符号开头。
    - 迭代过程的算式不使用 expr 命令格式。
- 下面这个例子在 bash shell 程序中使用了仿 C 语言的 for 命令：
    ```
    for (( i=1; i <= 10; i++ ))
    do
        echo "The next number is $i"
    done
    ```
    - for 循环通过定义好的变量（本例中是变量 i）来迭代执行这些命令。在每次迭代中， $i 变量都包含 for 循环中赋予的值。在每次迭代后，循环的迭代过程会作用于变量，在本例中，是将变量值增 1。

9. 使用多个变量
- 仿 C 语言的 for 命令也允许为迭代使用多个变量。循环会单独处理每个变量，你可以为每个变量定义不同的迭代过程。尽管可以使用多个变量， 但只能在 for 循环中定义一种迭代条件：
    ```
    for (( a=1, b=10; a <= 10; a++, b-- ))
    ```

10. while 命令
- while 命令允许定义一个要 测试的命令， 只要该命令返回的退出状态码为 0，就循环执行一组命令。它会在每次迭代开始时 测试 test 命令，如果 test 命令返回非 0 退出状态码， while 命令就会停止执行循环。
- while 命令的格式如下：
    ```
    while test command
    do
        other commands
    done
    ```
    - while 命令中定义的 test command 与 if-then 语句中的格式一模一样。 可以使用任何 bash shell 命令，或者用 test command 进行条件测试，比如测试变量值。
    - while 命令的关键在于所指定的 test command 的退出状态码必须随着循环中执行的命令而改变。如果退出状态码不发生变化，那 while 循环就成了死循环。
    - test command 最常见的用法是使用方括号来检查循环命令中用到的 shell 变量值：
        ```
        var1=10
        while [ $var1 -gt 0 ]
        do
            echo $var1
            var1=$[ $var1 - 1 ]
        done
        ```

11. 使用多个测试命令
- while 命令允许在 while 语句行定义多个测试命令。只有最后一个测试命令的退出状态码会被用于决定是否结束循环。
    ```
    var1=10
    while echo $var1
        [ $var1 -ge 0 ]
    do
        echo "This is inside the loop"
        var1=$[ $var1 - 1 ]
    done
    ```
    - 上述例子在 while 语句中定义了两个测试命令：第一个测试简单地显示了 var1 变量的当前值；第二个测试用方括号来判断 var1 变量的值。
    - while 循环会在 var1 变量等于 0 时执行 echo 语句，然后将 var1 变量的值减 1 。接下来 再次执行测试命令，判断是否进行下一次迭代。首先执行 echo 测试命令，显示 var 变量的值（小于 0）。接着执行 test 命令，因为条件不成立，所以 while 循环停止。
- 在含有多个命令的 while 语句中，在每次迭代时所有的测试命令都会被执行，包括最后一个测试命令失败的末次迭代。
- **注意要把每个测试命令都单独放在一行中。**

12. until 命令
- 与 while 命令工作的方式完全相反， until 命令要求指定一个返回非 0 退出状态码的测试命令。只要测试命令的退出状态码不为 0，bash shell 就会执行循环中列出的命令。一旦测试命令 返回了退出状态码 0，循环就结束了。
- until 命令的格式如下：
    ```
    until test commands
    do
        other commands
    done
    ```
    - 与 while 命令类似，你可以在 until 命令语句中放入多个 test command。最后一个命令的退出状态码决定了 bash shell 是否执行已定义的 other commands。
- 一个 until 命令的例子：
    ```
    $ cat test12
    #!/bin/bash
    # using the until command
    var1=100
    until [ $var1 -eq 0 ]
    do
        echo $var1
        var1=$[ $var1 - 25 ]
    done
    ```
- 同 while 命令一样， 在 until 命令中使用多个测试命令时也要注意：
    ```
    var1=100
    until echo $var1
          [ $var1 -eq 0 ]
    do
        echo Inside the loop: $var1
        var1=$[ $var1 - 25 ]
    done
    ```
    - shell 会执行指定的多个测试命令，仅当最后一个命令成立时才停止。

13. 嵌套循环
- 循环语句可以在循环内使用任意类型的命令，包括其他循环命令，这称为嵌套循环。注意， 在使用嵌套循环时是在迭代中再进行迭代， 命令运行的次数是乘积关系。
- shell 能够区分开内层 for 循环和外层 while 循环各自的 do 命令和 done 命令。

14. 循环处理文件数据
- 通过修改 IFS 环境变量，能强制 for 命令将文件中的每一行都作为单独的条目来处理，即便数据中有空格也是如此。从文件中提取出单独的行后，可能还得使用循环来提取行中的数据。
- 典型的例子是处理/etc/passwd 文件。这要求你逐行遍历该文件， 将 IFS 变量的值改成冒号， 以便分隔开每行中的各个字段：
    ```
    #!/bin/bash
    # changing the IFS value
    
    IFS.OLD=$IFS
    IFS=$'\n'
    for entry in $(cat /etc/passwd)
    do
        echo "Values in $entry –"
        IFS=:
        for value in $entry
        do
            echo "   $value"
        done
    done
    $
    ```
    - 第一个 IFS 值解析出/etc/passwd 文件中的 各行。内层 for 循环接着将 IFS 的值修改为冒号，以便解析出/etc/passwd 文件各行中的字段。

15. 循环控制
- 有两个命令可以控制循环的结束时机：
    - break 命令
    - continue 命令

16. break 命令
- break 命令是退出循环的一种简单方法。你可以用 break 命令退出任意类型的循环，包括 while 循环和 until 循环。
- 跳出单个循环：shell 在执行 break 命令时会尝试跳出当前正在执行的循环：
    ```
    for var1 in 1 2 3 4 5 6 7 8 9 10
    do
        if [ $var1 -eq 5 ]
        then
            break
        fi
        echo "Iteration number: $var1"
    done
    ```
- 跳出内层循环：在处理多个循环时， break 命令会自动结束你所在的最内层循环：
    ```
    for (( a = 1; a < 4; a++ ))
    do
        echo "Outer loop: $a"
        for (( b = 1; b < 100; b++ ))
        do
            if [ $b -eq 5 ]
            then
                break
            fi
            echo "   Inner loop: $b"
        done
    done
    ```
    - 即使 break 命令结束了内层循环， 外层循环依然会继续执行。
- 跳出外层循环：有时你位于内层循环， 但需要结束外层循环。 break 命令接受单个命令行参数： 
    ```
    break n
    ```
    - 其中 n 指定了要跳出的循环层级。在默认情况下，n 为 1，表明跳出的是当前循环。如果将 n 设 为 2，那么 break 命令就会停止下一级的外层循环：
        ```
        for (( a = 1; a < 4; a++ ))
        do
            echo "Outer loop: $a"
            for (( b = 1; b < 100; b++ ))
            do
                if [ $b -gt 4 ]
                then
                    break 2
                fi
                echo "   Inner loop: $b"
            done
        done
        ```
        - 当 shell 执行了 break 命令后，外部循环就结束了。

17. continue 命令
- continue 命令可以提前中止某次循环， 但不会结束整个循环。你可以在循环内部设置shell 不执行命令的条件。
- 来看一个在 for 循环中使用continue 命令的简单例子：
    ```
    for (( var1 = 1; var1 < 15; var1++ ))
    do
        if [ $var1 -gt 5 ] && [ $var1 -lt 10 ]
        then
            continue
        fi
        echo "Iteration number: $var1"
    done
    ```
    - 当 if-then 语句的条件成立时（值大于 5 且小于 10），shell 会执行 continue 命令， 跳过 此次循环中剩余的命令， 但整个循环还会继续。当 if-then 的条件不成立时， 一切会恢复如常。
    - 也可以在 while 循环和until 循环中使用 continue 命令， 但要特别小心。记住， 当shell 执行 continue 命令时，它会跳过剩余的命令。如果将测试变量的增值操作放在了其中某个条件里， 那么问题就出现了。
- 和 break 命令一样， continue 命令也允许通过命令行参数指定要继续执行哪一级循环： 
    ```
    continue n
    ```
    - 其中 n 定义了要继续的循环层级。

18. 处理循环的输出
- 在 shell 脚本中，可以对循环的输出使用管道或进行重定向。这可以通过在 done 命令之后 添加一个处理命令来实现：
    ```
    for file in /home/rich/*
    do
        if [ -d "$file" ]
        then
            echo "$file is a directory"
        elif
            echo "$file is a file"
        fi
    done > output.txt
    ```
    - shell 会将 for 命令的结果重定向至文件output.txt ，而不再显示在屏幕上。
- 这种方法同样适用于将循环的结果传输到另一个命令：
    ```
    for state in "North Dakota" Connecticut Illinois Alabama Tennessee
    do
        echo "$state is the next place to go"
    done | sort
    ```
    - for 命令的输出通过管道传给了 sort 命令，由后者对输出结果进行排序。运行该脚本， 可以看出结果已经按 state 的值排好序了。

19. 实战演练-查找可执行文件
    ```
    $ cat test25
    #!/bin/bash
    # finding files in the PATH
    
    IFS=:
    for folder in $PATH
    do
        echo "$folder:"
        for file in $folder/*
        do
            if [ -x $file ]
            then
                echo "   $file"
            fi
        done
    done
    ```
    - 当运行该脚本时，会得到一个可以在命令行中使用的可执行文件列表（输出显示了环境变量 PATH 所包含的所有目录中的所有可执行文件）。

20. 实战演练-创建多个用户账户
- 要想把数据从文件中传入 while 命令， 只需在 while 命令尾部使用一个重定向符即可：
    ```
    $ cat test26
    #!/bin/bash
    # process new user accounts
    
    input="users.csv"
    while IFS=',' read -r loginname name
    do
        echo "adding $loginname"
        useradd -c "$name" -m $loginname
    done < "$input"
    ```
    - users.csv 文本文件的格式如下：loginname, name
    - 第一项是为新用户账户所选用的用户 id。第二项是用户的全名。两个值之间以逗号分隔，这样就形成了一种叫作 CSV（comma-separated value，逗号分隔值）的文件格式。
    - 将 IFS 分隔符设置成逗号， 并将其作为 while 语句的条件测试部分。然后使用 read 命令读取文件中的各行。-r 屏蔽\，如果没有该选项，则\作为一个转义字符，有的话 \就是个正常的字符了。
    - read 命令会自动移往 CSV 文本文件的下一行。
    - 当 read 命令返回假值的时候（也就是读取完整个文件），while 命令就会退出。
    - $input 变量中保存的是数据文件名，该数据文件被作为 while 命令的数据源。 
        ```
        $ cat users.csv
        rich,Richard Blum
        christine,Christine Bresnahan
        ```
    - 必须以 root 用户身份运行该脚本，因为 useradd 命令需要 root权限。
    - 最后可查看 /etc/passwd 文件验证执行结果。
