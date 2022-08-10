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
