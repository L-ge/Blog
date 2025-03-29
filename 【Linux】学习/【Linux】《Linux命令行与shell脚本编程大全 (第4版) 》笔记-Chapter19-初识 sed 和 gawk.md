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
