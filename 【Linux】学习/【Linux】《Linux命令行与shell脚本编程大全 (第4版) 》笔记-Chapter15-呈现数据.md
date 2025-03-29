###### 十五、呈现数据

1. 标准文件描述符
- Linux 用文件描述符来标识每个文件对象。文件描述符是一个非负整数，唯一会标识的是会话中打开的文件。每个进程一次最多可以打开 9 个文件描述符（这个数量并不是固定的）。
- 出于特殊目的， bash shell 保留了前 3 个文件描述符（0、1 和 2），Linux 的标准文件描述符：
    文件描述符 | 缩写 | 描述
    ---|---|---
    0 | STDIN   | 标准输入
    1 | STDOUT  | 标准输出
    2 | STDERR  | 标准错误
    
    - 这 3个特殊的文件描述符会处理脚本的输入和输出。shell 会用它们将其默认的输入和输出送往适合的位置。
- STDIN
    - STDIN 文件描述符代表 shell 的标准输入。对终端界面来说，标准输入就是键盘。shell 会从 STDIN
    文件描述符对应的键盘获得输入并进行处理。
    - 在使用输入重定向符（<）时， Linux 会用重定向指定的文件替换标准输入文件描述符。 其实就是将标准输入文件描述符的指向由原先的键盘更改为指定的文件。 
- STDOUT
    - STDOUT 文件描述符代表 shell 的标准输出。在终端界面上，标准输出就是终端显示器。shell 的所有输出（包括 shell 中运行的程序和脚本）会被送往标准输出， 也就是显示器。
    - 在默认情况下，大多数 bash 命令会将输出送往 STDOUT 文件描述符。
    - 通过输出重定向符（>），原本应该出现在屏幕上的所有输出会被 shell 重定向到指定的文件。
    - 也可以使用>>将数据追加到某个文件。
    - 当命令产生错误消息时， shell 并未将错误消息重定向到指定文件。shell 创建了输出重定向文件，但错误消息依然显示在屏幕上。
    - shell 对于错误消息的处理是跟普通输出分开的。如果创建了一个在后台运行的 shell 脚本，则通常必须依赖发送到日志文件的输出消息。用这种方法的话，如果出现错误消息，这些消息也不会出现在日志文件中，因此需要换一种方法来处理。
- STDERR
    - shell 通过特殊的 STDERR 文件描述符处理错误消息。 STDERR 文件描述符代表 shell 的标准错误输出。 shell 或运行在 shell 中的程序和脚本报错时，生成的错误消息都会被送往这个位置。
    - 在默认情况下， STDERR 和 STDOUT 指向同一个地方（尽管二者的文件描述符索引值不同）。也就是说，所有的错误消息也都默认会被送往显示器。
    - STDERR 并不会随着 STDOUT 的重定向发生改变。

2. 重定向错误
- 只要在使用重定向符时指定 STDERR 文件描述符就可以重定向 STDERR 数据了。
- 只重定向错误
    - STDERR 的文件描述符为 2。可以将该文件描述符索引值放在重定向符号之前，只重定向错误消息。注意， 两者必须紧挨着，否则无法正常工作：
        ```
        $ ls -al badfile 2> test4
        $ cat test4
        ls: cannot access badfile: No such file or directory
        ```
        - 命令生成的任何错误消息都会保存在指 定文件中。用这种方法， shell 只重定向错误消息，而非普通数据。
    - 一个混合使用 STDOUT 和 STDERR 错误消息的例子：
        ```
        $ ls -al test badtest test2 2> test5
        -rw-rw-r-- 1 rich rich 158 2020-06-20 11:32 test2
        $ cat test5
        ls: cannot access test: No such file or directory
        ls: cannot access badtest: No such file or directory
        $
        ```
        - ls 命令尝试列出了 3 个文件（test、badtest 和 test2）的信息。正常输出被送往默认的 STDOUT 文件描述符， 也就是显示器。由于该命令将文件描述符 2（ STDERR）重定向到了一个输出文件， 因此 shell 会将产生的所有错误消息直接送往指定文件。
- 重定向错误消息和正常输出
    - 如果想重定向错误消息和正常输出， 则必须使用两个重定向符号。需要在重定向符号之前 放上需要重定向的文件描述符，然后让它们指向用于保存数据的输出文件：
        ```
        $ ls -al test test2 test3 badtest 2> test6 1> test7
        $ cat test6
        ls: cannot access test: No such file or directory
        ls: cannot access badtest: No such file or directory
        $ cat test7
        -rw-rw-r-- 1 rich rich 158 2020-06-20 11:32 test2
        -rw-rw-r-- 1 rich rich   0 2020-06-20 11:33 test3
        $
        ```
        - ls 命令的正常输出本该送往 STDOUT，shell 使用 1>将其重定向到了文件 test7，而本该送往 STDERR 的错误消息则通过 2>被重定向到了文件 test6。
        - 可以用这种方法区分脚本的正常输出和错误消息。
    - 也可以将 STDERR 和 STDOUT 的输出重定向到同一个文件。为此， bash shell 提供了特殊的重定向符&>：
        ```
        $ ls -al test test2 test3 badtest &> test7
        $ cat test7
        ls: cannot access test: No such file or directory
        ls: cannot access badtest: No such file or directory
        -rw-rw-r-- 1 rich rich 158 2020-06-20 11:32 test2
        -rw-rw-r-- 1 rich rich   0 2020-06-20 11:33 test3
        $
        ```
        - 当使用&>时，命令生成的所有输出（正常输出和错误消息）会被送往同一位置。
        - 注意，其 中一条错误消息出现的顺序和预想不同。 badtest 文件（列出的最后一个文件）的这条错误消息出 现在了输出文件的第二行。为了避免错误消息散落在输出文件中，相较于标准输出，bash shell 自动赋予了错误消息更高的优先级。

3. 在脚本中重定向输出的方法有两种：
    - 临时重定向每一行。
    - 永久重定向脚本中的所有命令。

4. 临时重定向
- 如果需要在脚本中生成错误消息，可以将单独的一行输出重定向到 STDERR。这只需要使用输出重定向符号将输出重定向到 STDERR 文件描述符。在重定向到文件描述符时， 必须在文件 描述符索引值之前加一个&：
    ```
    echo "This is an error message" >&2
    ```
    - 这行会在脚本的 STDERR 文件描述符所指向的位置显示文本。
- 例子：
    ```
    $ cat test8
    #!/bin/bash
    # testing STDERR messages
    echo "This is an error" >&2
    echo "This is normal output"
    $
    $ ./test8
    This is an error
    This is normal output
    $ 
    $ ./test8 2> test9
    This is normal output
    $ cat test9
    This is an error
    ```
    - 在默认情况下， STDERR 和 STDOUT 指向的位置是一样的。但是，如果在运行脚本时重定向了 STDERR，那么脚本中所有送往 STDERR 的文本都会被重定向：
        ```
        $ ./test8 2> test9
        This is normal output
        $ cat test9
        This is an error
        $
        ```
        - 这种方法非常适合在脚本中生成错误消息，直接通过 STDERR 文件描述符重定向错误消息。

5. 永久重定向
- 如果脚本中有大量数据需要重定向，那么逐条重定向所有的 echo 语句会很烦琐。这时可以用 exec 命令，它会告诉 shell 在脚本执行期间重定向某个特定文件描述符：
    ```
    $ cat test10
    #!/bin/bash
    # redirecting all output to a file
    exec 1>testout
    
    echo "This is a test of redirecting all output"
    echo "from a script to another file."
    echo "without having to redirect every individual line"
    $ ./test10
    $ cat testout
    This is a test of redirecting all output
    from a script to another file.
    without having to redirect every individual line
    $
    ```
    - exec 命令会启动一个新 shell 并将 STDOUT 文件描述符重定向到指定文件。脚本中送往 STDOUT 的所有输出都会被重定向。
- 也可以在脚本执行过程中重定向 STDOUT：
    ```
    $ cat test11
    #!/bin/bash
    # redirecting output to different locations
    exec 2>testerror
    echo "This is the start of the script"
    echo "now redirecting all output to another location"
    exec 1>testout
    echo "This output should go to the testout file"
    echo "but this should go to the testerror file" >&2
    $
    $ ./test11
    This is the start of the script
    now redirecting all output to another location
    $ cat testout
    This output should go to the testout file
    $ cat testerror
    but this should go to the testerror file
    $
    ```
    - 该脚本使用 exec 命令将送往 STDERR 的输出重定向到了文件 testerror。接下来，脚本用 echo 语句向 STDOUT 显示了几行文本。随后再次使用 exec 命令将 STDOUT 重定向到 testout 文件。注意，尽管 STDOUT 被重定向了，仍然可以将 echo 语句的输出发送给 STDERR，在本例中仍是重定向到 testerror 文件。
    - 当只想将脚本的部分输出重定向到其他位置（比如错误日志）时，这个特性用起来非常方便。不过这样做的话，会遇到一个问题。一旦重定向了 STDOUT 或 STDERR，就不太容易将其恢复到原先的位置。

6. 在脚本中重定向输入
- 可以使用与重定向 STDOUT 和 STDERR 相同的方法， 将 STDIN 从键盘重定向到其他位置。在 Linux 系统中， exec 命令允许将 STDIN 重定向为文件：   
    ```
    exec 0< testfile
    ```
    - 该命令会告诉 shell，它应该从文件 testfile 中而不是键盘上获取输入。只要脚本需要输入，这个重定向就会起作用。
- 例子：
    ```
    $ cat test12
    #!/bin/bash
    # redirecting file input
    exec 0< testfile
    count=1
    while read line
    do
        echo "Line #$count: $line"
        count=$[ $count + 1 ]
    done
    $ ./test12
    Line #1: This is the first line.
    Line #2: This is the second line.
    Line #3: This is the third line.
    $
    ```
    - 将 STDIN 重定向为文件后，当 read 命令试图从 STDIN 读入数据时，就会到文件中而不是键盘上检索数据。
    - 这是一种在脚本中从待处理的文件读取数据的绝妙技术。 Linux 系统管理员的日常任务之一就是从日志文件中读取并处理数据。这是完成该任务最简单的办法。

7. 创建自己的重定向
- 在脚本中重定向输入和输出时，并不局限于这 3 个默认的文件描述符。在 shell 中最多可以打开 9个文件描述符。替代性文件描述符从 3 到 8 共 6个，均可用作输入或输出重定向。这些文件描述符中的任意一个都可以分配给文件并用在脚本中。
- 在重定向时， 如果使用大于 9 的文件描述符， 那么一定要小心，因为有可能会与 shell 内部使用的文件描述符发生冲突。

8. 创建输出文件描述符
- 可以用 exec 命令分配用于输出的文件描述符。和标准的文件描述符一样， 一旦将替代性文件描述符指向文件， 此重定向就会一直有效， 直至重新分配。
- 在脚本中使用替代性文件描述符的简单例子：
    ```
    $ cat test13
    #!/bin/bash
    # using an alternative file descriptor
    exec 3>test13out
    echo "This should display on the monitor"
    echo "and this should be stored in the file" >&3
    echo "Then this should be back on the monitor"
    $ ./test13
    This should display on the monitor
    Then this should be back on the monitor
    $ cat test13out
    and this should be stored in the file
    $
    ```
    - 这个脚本使用 exec 命令将文件描述符 3 重定向到了另一个文件。
- 也可以不创建新文件，而是使用 exec 命令将数据追加到现有文件：
    ```
    exec 3>>test13out
    ```
    - 现在，输出会被追加到 test13out 文件，而不是创建一个新文件。

9. 重定向文件描述符
- 可以将另一个文件描述符分配给标准 文件描述符，反之亦可。这意味着可以将 STDOUT 的原先位置重定向到另一个文件描述符， 然后再利用该文件描述符恢复 STDOUT。
- 例子：
    ```
    $ cat test14
    #!/bin/bash
    # storing STDOUT, then coming back to it
    exec 3>&1
    exec 1>test14out
    echo "This should store in the output file"
    echo "along with this line."
    exec 1>&3
    echo "Now things should be back to normal"
    $
    $ ./test14
    Now things should be back to normal
    $ cat test14out
    This should store in the output file
    along with this line.
    $
    ```
    - 第一个 exec 命令将文件描述符 3 重定向到了文件描述符 1（STDOUT）的当前位置，也就是显示器。这意味着任何送往文件描述符 3 的输出都会出现在屏幕上。
    - 第二个 exec 命令将 STDOUT 重定向到了文件， shell 现在会将发送给 STDOUT 的输出直接送往该文件。但是，文件描述符 3 仍然指向 STDOUT 原先的位置（显示器）。如果此时将输出数据 发送给文件描述符 3，则它仍然会出现在显示器上，即使 STDOUT 已经被重定向了。
    - 第三个 exec 命令将 STDOUT 重定向到了文件描述符 3 的当前位置（现在仍然是显示器）。这意味着现在 STDOUT 又恢复如初了，即指向其原先的位置——显示器。

10. 创建输入文件描述符
- 可以采用和重定向输出文件描述符同样的办法来重定向输入文件描述符。在重定向到文件之前， 先将 STDIN 指向的位置保存到另一个文件描述符， 然后在读取完文件之后将 STDIN 恢复到原先的位置：
    ```
    $ cat test15
    #!/bin/bash
    # redirecting input file descriptors
    exec 6<&0
    exec 0< testfile
    count=1
    while read line
    do
        echo "Line #$count: $line"
        count=$[ $count + 1 ]
    done
    exec 0<&6
    read -p "Are you done now? " answer
    case $answer in
    Y|y) echo "Goodbye";;
    N|n) echo "Sorry, this is the end.";;
    esac
    $ ./test15
    Line #1: This is the first line.
    Line #2: This is the second line.
    Line #3: This is the third line.
    Are you done now? y
    Goodbye
    $
    ```
    - 在这个例子中，文件描述符 6 用于保存 STDIN 指向的位置。然后脚本将 STDIN 重定向到一个文件。 read 命令的所有输入都来自重定向后的 STDIN （也就是输入文件）。
    - 在读完所有行之后，脚本会将 STDIN 重定向到文件描述符 6，恢复 STDIN 原先的位置。该脚本使用另一个 read 命令来测试 STDIN 是否恢复原位，这次 read 会等待键盘的输入。

11. 创建读/写文件描述符
- 也可以打开单个文件描述符兼做输入和输出，这样就能用同一个文件描述符对文件进行读和写两种操作了。
- 但使用这种方法时要特别小心。由于这是对一个文件进行读和写两种操作，因此 shell 会维 护一个内部指针，指明该文件的当前位置。任何读或写都会从文件指针上次的位置开始。
- 例子：
    ```
    $ cat test16
    #!/bin/bash
    # testing input/output file descriptor
    exec 3<> testfile
    read line <&3
    echo "Read: $line"
    echo "This is a test line" >&3
    $ cat testfile
    This is the first line.
    This is the second line.
    This is the third line.
    $ ./test16
    Read: This is the first line.
    $ cat testfile
    This is the first line.
    This is a test line
    ine.
    This is the third line.
    $
    ```
    - 在这个例子中，exec 命令将文件描述符 3 用于文件 testfile 的读和写。接下来，使用分配好的文件描述符，通过 read 命令读取文件中的第一行，然后将其显示在 STDOUT 中。最后，使用 echo 语句将一行数据写入由同一个文件描述符打开的文件中。
    - 在运行脚本时，一开始还算正常。输出内容表明脚本读取了 testfile 文件的第一行。但如果在脚本运行完毕后查看 testfile 文件内容，则会发现写入文件中的数据覆盖了已有数据。
    - 当脚本向文件中写入数据时， 会从文件指针指向的位置开始。 read 命令读取了第一行数据，这使得文件指针指向了第二行数据的第一个字符。当 echo 语句将数据输出到文件时，会将数据写入文件指针的当前位置，覆盖该位置上的已有数据。

12. 关闭文件描述符
- 如果创建了新的输入文件描述符或输出文件描述符，那么 shell 会在脚本退出时自动将其关闭。然而在一些情况下，需要在脚本结束前手动关闭文件描述符。
- 要关闭文件描述符，只需将其重定向到特殊符号&-即可。在脚本中如下所示：
    ```
    exec 3>&-
    ```
    - 该语句会关闭文件描述符 3，不再在脚本中使用。
- 一旦关闭了文件描述符，就不能在脚本中向其写入任何数据，否则 shell 会发出错误消息。
- 在关闭文件描述符时还要注意另一件事。如果随后你在脚本中打开了同一个输出文件， 那么 shell 就会用一个新文件来替换已有文件。这意味着如果你输出数据，它就会覆盖已有文件。例子

13. 列出打开的文件描述符
- lsof 命令会列出整个 Linux 系统打开的所有文件描述符，这包括所有后台进程以及登录用户打开的文件。
- 有大量的命令行选项和参数可用于过滤 lsof 的输出。最常用的选项包括-p 和-d，前者允许指定进程 ID（PID ），后者允许指定要显示的文件描述符编号（多个编号之间以逗号分隔）。
- 要想知道进程的当前 PID，可以使用特殊环境变量$$（shell 会将其设为当前 PID ）。-a 选项 可用于对另外两个选项的结果执行 AND 运算，命令输出如下：
    ```
    $ /usr/sbin/lsof -a -p $$ -d 0,1,2
    COMMAND  PID USER   FD   TYPE DEVICE SIZE NODE NAME
    bash    3344 rich    0u   CHR  136,0         2 /dev/pts/0
    bash    3344 rich    1u   CHR  136,0         2 /dev/pts/0
    bash    3344 rich    2u   CHR  136,0         2 /dev/pts/0
    $
    ```
    - 显示了当前进程（bash shell）的默认文件描述符（0、1 和 2）。
- lsof 的默认输出
    列 | 描述
    ---|---    
    COMMAND | 进程对应的命令名的前 9 个字符
    PID     | 进程的 PID
    USER    | 进程属主的登录名
    FD      | 文件描述符编号以及访问类型（r 代表读， w 代表写， u 代表读/写）
    TYPE    | 文件的类型（ CHR 代表字符型， BLK 代表块型， DIR 代表目录， REG 代表常规文件）
    DEVICE  | 设备号（主设备号和从设备号）
    SIZE    | 如果有的话，表示文件的大小
    NODE    | 本地文件的节点号
    NAME    | 文件名

- 与 STDIN、STDOUT 和 STDERR 关联的文件类型是字符型，因为文件描述符 STDIN、STDOUT 和 STDERR 都指向终端，所以输出文件名就是终端的设备名。这 3 个标准文件都支持读和写。
- 例子：
    ```
    $ cat test18
    #!/bin/bash
    # testing lsof with file descriptors
    exec 3> test18file1
    exec 6> test18file2
    exec 7< testfile
    /usr/sbin/lsof -a -p $$ -d0,1,2,3,6,7
    $ ./test18
    COMMAND  PID USER   FD   TYPE DEVICE SIZE   NODE NAME
    test18  3594 rich    0u   CHR  136,0           2 /dev/pts/0
    test18  3594 rich    1u   CHR  136,0           2 /dev/pts/0
    test18  3594 rich    2u   CHR  136,0           2 /dev/pts/0
    18  3594 rich    3w   REG  253,0    0 360712 /home/rich/test18file1
    18  3594 rich    6w   REG  253,0    0 360715 /home/rich/test18file2
    18  3594 rich    7r   REG  253,0   73 360717 /home/rich/testfile
    $
    ```

14. 抑制命令输出
- 如果不想显示脚本输出，可以将 STDERR 重定向到一个名为 null 文件的特殊文件。shell 输出到 null 文件的任何数据都不会被保存，全部会被丢弃。
- 在 Linux 系统中，null 文件的标准位置是/dev/null。重定向到该位置的任何数据都会被丢弃，不再显示。这是抑制错误消息出现且无须保存它们的一种常用方法。
- 也可以在输入重定向中将/dev/null 作为输入文件。由于/dev/null 文件不含任何内容，因此程 序员通常用它来快速清除现有文件中的数据，这样就不用先删除文件再重新创建了：
    ```
    $ cat /dev/null > testfile
    ```
    - 文件 testfile 仍然还在，但现在是一个空文件。这是清除日志文件的常用方法，因为日志文件必须时刻等待应用程序操作。

15. 使用临时文件
- Linux 系统有一个专供临时文件使用的特殊目录/tmp，其中存放那些不需要永久保留的文件。大多数 Linux 发行版配置系统在启动时会自动删除/tmp 目录的所有文件。
- 系统中的任何用户都有权限读写/tmp 目录中的文件。这个特性提供了一种创建临时文件的简单方法，而且还无须担心清理工作。
- 还有一个专门用于创建临时文件的命令 mktemp，该命令可以直接在/tmp 目录中创建唯一的临时文件。所创建的临时文件不使用默认的 umask 值。作为临时文件属主，你拥有该文件的读写权限，但其他用户无法访问（当然，root 用户除外）。

16. 创建本地临时文件
- 在默认情况下， mktemp 会在本地目录中创建一个文件。在使用 mktemp 命令时， 只需指定 一个文件名模板即可。模板可以包含任意文本字符，同时在文件名末尾要加上 6 个 X：
    ```
    $ mktemp testing.XXXXXX
    $ ls -al testing*
    -rw-------   1 rich      rich      0 Jun 20 21:30 testing.UfIi13
    ```
    - mktemp 命令会任意地将 6 个 X 替换为同等数量的字符，以保证文件名在目录中是唯一的。可以创建多个临时文件，mktemp 命令会确保每个文件名都不重复。
- 在脚本中使用 mktemp 命令时，可以将文件名保存到变量中，这样就能在随后的脚本中引用了：
    ```
    tempfile=$(mktemp test19.XXXXXX)
    ```

17. 在/tmp 目录中创建临时文件
- -t 选项会强制 mktemp 命令在系统的临时目录中创建文件。在使用这个特性时， mktemp 命令会返回所创建的临时文件的完整路径名，而不只是文件名：
    ```
    $ mktemp -t test.XXXXXX
    /tmp/test.xG3374
    ```
- 由于 mktemp 命令会返回临时文件的完整路径名，因此可以在文件系统的任何位置引用该临时文件：
    ```
    tempfile=$(mktemp -t tmp.XXXXXX)
    echo "The temp file is located at: $tempfile"
    ```
    - 在创建临时文件时， mktemp 会将全路径名返回给环境变量。这样就能在任何命令中使用该值来引用临时文件了。

18. 创建临时目录
- -d 选项会告诉 mktemp 命令创建一个临时目录。可以根据需要使用该目录，比如在其中创建其他的临时文件：
    ```
    tempdir=$(mktemp -d dir.XXXXXX)
    cd $tempdir
    tempfile1=$(mktemp temp.XXXXXX)
    tempfile2=$(mktemp temp.XXXXXX)
    ```

19. 记录消息
- 如果需要将输出同时送往显示器和文件，与其对输出进行两次重定向，不如改用 tee 命令。
- tee 命令就像是连接管道的 T 型接头，它能将来自 STDIN 的数据同时送往两处。一处是 STDOUT，另一处是 tee 命令行所指定的文件名：
    ```
    tee filename
    ```
- 由于 tee 会重定向来自 STDIN 的数据，因此可以用它配合管道命令来重定向命令输出：
    ```
    $ date | tee testfile
    Sun Jun 21 18:56:21 EDT 2020
    $ cat testfile
    Sun Jun 21 18:56:21 EDT 2020
    ```
    - 输出出现在了 STDOUT 中， 同时写入了指定文件。
- 注意，在默认情况下，tee 命令会在每次使用时覆盖指定文件的原先内容。如果想将数据追加到指定文件中，就必须使用-a 选项。

20. 实战演练-读取 CSV 格式的数据文件，输出 SQL INSERT 语句，并将数据插入数据库
    ```
    outfile='members.sql'
    IFS=','
    while read lname fname address city state zip
    do
    cat >> $outfile << EOF
    INSERT INTO members (lname,fname,address,city,state,zip) VALUES
    ('$lname', '$fname', '$address', '$city', '$state', '$zip');
    EOF
    done < ${1}
    ```
    - $1 代表第一个命令行参数， 指明了待读取数据的文件。
    - read 语句使 用 IFS 字符解析读入的文本，这里将 IFS 指定为逗号。
    - cat 那行语句包含一个输出追加重定向（双大于号）和一个输入追加重定向（双小于号）。输出重定向将 cat 命令的输出追加到由 $outfile 变量指定的文件中。 cat 命令的输入不再取自标准 输入，而是被重定向到脚本内部的数据。EOF 符号标记了文件中的数据起止。
    - 上述文本生成了一个标准的 SQL INSERT 语句。注意，其中的数据由变量来替换，变量中的内容则由 read 语句存入。
