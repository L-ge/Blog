###### 五、理解 shell

1. shell 的类型
- 当你登录系统时，系统启动什么样的 shell 程序取决于你的个人用户配置。在/etc/passwd 文件中，用户记录的第 7 个字段中列出了该用户的默认 shell 程序。只要用户登录某个虚拟控制台终端或是在 GUI 中启动终端仿真器，默认的 shell 程序就会启动。
- 在现代 Linux 系统中， bash shell 程序（bash）通常位于/usr/bin 目录（也有可能位于/bin 目录）。which bash 命令可以帮助我们找出bash shell 的位置。
- 长列表中文件名尾部的星号（ * ）表明 bash 文件（bash shell）是一个可执行程序。
- 在大多数 Linux 系统中， /etc/shells 文件中列出了各种已安装的 shell ，这些 shell 可以作为用 户的默认 shell。
- 在很多 Linux 发行版中，你会发现 shell 文件似乎存在于两个位置： /bin 和/usr/bin。这是因为在现代 Linux 系统中， /bin 是指向/usr/bin 的符号链接。
- 默认的交互式 shell（default interactive shell） 也称登录shell（login shell）， 只要用户登录某个虚拟控制台终端或是在 GUI 中启动终端仿真器，该 shell 就会启动。作为默认的系统 shell（default system shell）， sh（/bin/sh）用于那些需要在启动时使用的系统 shell 脚本。
    - CentOS 发行版使用软链接将默认的系统 shell指向 bash shell。
    - Ubuntu 发行版默认的系统 shell（/usr/bin/sh ）指向的是 dash shell。
    - 对 bash shell 脚本来说，这两种 shell （默认的交互 shell 和默认的系统 shell）可能会导致问题。
- 命令 echo $0 会显示当前 shell 的名称。
    - 使用 echo $0 显示当前所用 shell 的做法仅限在 shell 命令行中使用。如果在 shell 脚本中 使用，则显示的是该脚本的名称。
        ```
        $ echo $0 
        -bash
        $ dash
        $ echo $0
        dash
        $ exit
        $ echo $0
        -bash
        ```
        - echo $0 命令的输出： bash 之前有一个连字符（-）。 表明该 shell 是用户的登录 shell。
        - 输入命令 exit 就可以退出 dash shell 程序。

2. shell 的父子关系
- 用户登录某个虚拟控制台终端或在 GUI 中运行终端仿真器时所启动的默认的交互式 shell （登录 shell ）是一个父 shell。
- 当你在 CLI 提示符处输入 bash 命令（或是其他 shell 程序名）时， 会创建新的 shell 程序。 这是一个子 shell。
- 子 shell 的父进程 ID（PPID）就是父 shell 进程 ID（可以通过在生成子 shell 的前后使用 ps -f 命令查看）。
- 在生成子 shell 进程时， 只有部分父进程的环境被复制到了子 shell环境中。这会对包括变量 在内的一些东西造成影响。
- 子 shell 既可以从父 shell 中创建， 也可以从另一个子 shell 中创建。
    - 可以通过在连续多次生成子 shell 的后使用 ps --forest 命令查看。
    - ps -f 命令也能够表现子 shell 间的嵌套关系， 因为它会通过 PPID 列显示出谁是谁的父进程。
- bash shell 程序可以使用命令行选项来修改 shell 的启动方式。
- bash 的命令行选项：
    选项 | 描述
    ---         |---
    -c string   | 从 string 中读取命令并进行处理
    -i          | 启动一个能够接收用户输入的交互式 shell
    -l          | 作为登录 shell 启动
    -r          | 启动一个受限 shell，将用户限制在默认目录中
    -s          | 从标准输入中读取命令
- 可以输入 man bash 获得关于 bash 命令的更多帮助信息，了解更多的命令行选项。 bash --help 命令也会提供一些额外的协助。
- 如果想查看 bash shell 的版本号，在命令行中输出 bash --version 即可。该命令不会创建子 shell，只会显示系统中 GNU bash shell 程序的当前版本。
- 可以使用 exit 命令有条不紊地退出子 shell。exit 命令不仅能够退出子 shell，还可以注销（log out）当前的虚拟控制台终端或终端仿真器软件。只需在父 shell 中输入 exit，就能从容退出 CLI 了。
- 有时运行 shell 脚本也会创建子 shell。

3. 查看进程列表
- 可以在单行中指定要依次运行的一系列命令。这可以通过命令列表来实现，只需将命令之间以分号（;）分隔即可。
    ```
    $ pwd ; ls test* ; cd /etc ; pwd ; cd ; pwd ; ls my*
    ```
    - 在上面的例子中，所有命令依次执行。不过这并不是进程列表。
- 要想成为进程列表，命令列表必须将命令放入圆括号内：
    ```
    $ (pwd ; ls test* ; cd /etc ; pwd ; cd ; pwd ; ls my*)
    ```
    - 圆括号的加入使命令列表摇身变成了进程列表，生成了一个子 shell 来执行这些命令。
- 进程列表是命令分组（command grouping）的一种。另一种命令分组是将命令放入花括号内，并在命令列表尾部以分号（ ; ）作结。语法为： { command; }。使用花括号进行命令分组并不会像进程列表那样创建子 shell。
- 要想知道是否生成了子 shell，需要使用命令输出一个环境变量（参见第 6 章）的值。这个命令就是 echo $BASH_SUBSHELL。如果该命令返回 0，那么表明没有子 shell。如果该命令返回 1 或者其他更大的数字， 则表明存在子 shell。
- 进程列表就是使用圆括号包围起来的一组命令，它能够创建子 shell 来执行这些命令。甚至可以在进程列表中嵌套圆括号来创建子 shell 的子 shell。
- 子 shell 在 shell 脚本中经常用于多进程处理。但是，创建子 shell 的成本不菲（意思是要消耗更多的资源，比如内存和处理能力），会明显拖慢任务进度。在交互式 CLI shell 会话中， 子shell 同样存在问题，它并非真正的多进程处理，原因在于终端与子 shell 的 I/O 绑定在了一起。

4. 在交互式 shell 中， 一种高效的子 shell 用法是后台模式。

5. sleep 命令会接受一个参数作为希望进程等待（睡眠）的秒数。该命令在 shell 脚本中常用于引入一段暂停时间。命令 sleep 10 会将会话暂停 10 秒，然后返回 shell CLI 提示符。
    ```
    $ sleep 3000&
    [1] 2542
    $ ps -f
    UID        PID  PPID  C STIME TTY   TIME CMD
    christi+  2356  2352  0 13:27 pts/0 00:00:00 -bash
    christi+  2542  2356  0 13:44 pts/0 00:00:00 sleep 3000
    christi+  2543  2356  0 13:44 pts/0 00:00:00 ps -f
    ```
    - 要想将命令置入后台模式， 可以在命令末尾加上字符&。
    - sleep 命令会在后台（&）睡眠 3000 秒。当其被置入后台时，在 shellCLI 提示符返回之前，屏幕上会出现两条信息。第一条信息是方括号中的后台作业号（ 1）。第二条信息是后台作业的进程 ID（ 2542）。
    - 除了 ps 命令，也可以使用 jobs 命令来显示后台作业信息。 jobs 命令能够显示当前运行在 后台模式中属于你的所有进程（作业）。
        ```
        $ jobs
        [1]+  Running                   sleep 3000 &
        
        $ jobs -l
        [1]+  2542 Running              sleep 3000 &
        
        $
        [1]+  Done                      sleep 3000
        ```
        - jobs 命令会在方括号中显示作业号（1）。除此之外，还有作业的当前状态（Running）以 及对应的命令（sleep 3000 &）。
        - 利用 jobs 命令的-l （小写字母 l）选项，还可以看到更多的相关信息。除了默认信息， -l 选项还会显示命令的 PID。    
        - 如果运行多个后台进程， 则还有一些额外信息可以显示哪个后台作业是最近启动的。在 jobs 命令的显示中， 最近启动的作业在其作业号之后会有一个加号（+），在它之前启 动的进程（the second newest process）则以减号（-）表示。
        - 一旦后台作业完成，就会显示出结束状态。

6. 通过将进程列表置入后台，可以在子 shell 中进行大量的多进程处理。由此带来的一个好处是终端不再和子 shell 的 I/O 绑定在一起。
- 将进程列表置入后台会产生一个作业号和进程 ID，然后会返回提示符。
- 在后台使用进程列表可谓是在 CLI 中运用子 shell 的一种创造性方法。可以用更少的键盘输入换来更高的效率。
- 使用tar（参见第 4 章） 创建备份文件是有效利用后台进程列表的一个更实用的例子：
    ```
    $ (tar -cf Doc.tar Documents ; tar -cf Music.tar Music)&
    [1] 2567
    $
    $ ls *.tar
    Doc.tar Music.tar
    [1]+ Done       ( tar -cf Doc.tar Documents; tar -cf Music.tar Music )
    $
    ```

7. 将进程列表置入后台并不是子 shell在 CLI 中仅有的创造性用法，还有一种方法是协程。
- 协程同时做两件事：一是在后台生成一个子 shell，二是在该子 shell 中执行命令。
- 要进行协程处理，可以结合使用 coproc 命令以及要在子 shell 中执行的命令：
    ```
    $ coproc sleep 10
    [1] 2689
    $ jobs
    [1]+  Running                   coproc COPROC sleep 10 &
    
    $ coproc My_Job { sleep 10; }
    $ jobs
    [1]+ Running                    coproc My_Job { sleep 10; } &
    ```
    - 除了会创建子 shell，协程基本上就是将命令置入后台。当输入 coproc 命令及其参数之后， 你会发现后台启用了一个作业。屏幕上会显示该后台作业号（1）以及进程 ID（2689）。
    - jobs 命令可以显示协程的状态。
    - 可以看到，在子 shell 中执行的后台命令是 coproc COPROC sleep 10。
    - COPROC 是 coproc 命令给进程起的名字。可以使用命令的扩展语法来自己设置这个名字。使用扩展语法，协程名被设置成了 My_Job。
    - 注意，扩展语法写起来有点儿小麻烦。 你必须确保在左花括号（ {）和内部命令名之间有一个空格。还必须保证内部命令以分号（;）结 尾。另外，分号和右花括号（ } ）之间也得有一个空格。
- 只有在拥有多个协程时才需要对协程进行命名， 因为你要和它们进行通信。否则的话， 让 coproc 命令 将其设置成默认名称 COPROC 即可。
- 可以将协程与进程列表结合起来创建嵌套子 shell。只需将命令 coproc 放在进程列表之前即可。
- 记住，生成子 shell 的成本可不低，而且速度很慢。创建嵌套子 shell更是火上浇油。

8. 外部命令
- 外部命令（有时也称为文件系统命令）是存在于 bash shell 之外的程序。也就是说，它并不 属于 shell 程序的一部分。外部命令程序通常位于/bin 、/usr/bin 、/sbin 或/usr/sbin 目录中。
- ps 命令就是一个外部命令。可以使用 which 命令和 type 命令找到其对应的文件名。
- 每当执行外部命令时，就会创建一个子进程。这种操作称为衍生（forking）。外部命令 ps 会显示其父进程以及自己所对应的衍生子进程。
- 只要涉及进程衍生， 就需要耗费时间和资源来设置新子进程的环境。因此， 外部命令系统开销较高。
- 无论是衍生出子进程还是创建了子 shell，都仍然可以通过信号与其互通，这一点无论是在使用命令行还是在编写脚本时都极其有用。进程间以发送信号的方式彼此通信。
- 在使用内建命令时，不需要衍生子进程。因此，内建命令的系统开销较低。

9. 内建命令
- 与外部命令不同， 内建命令无须使用子进程来执行。内建命令已经和 shell 编译成一体， 作 为 shell 的组成部分存在， 无须借助外部程序文件来执行。
- cd 命令和 exit 命令都内建于 bash shell。可以使用 type 命令来判断某个命令是否为内建。
- 因为内建命令既不需要通过衍生出子进程来执行，也不用打开程序文件，所以执行速度更快， 效率也更高。
- 注意， 有些命令有多种实现。例如， echo 和 pwd 既有内建命令也有外部命令。两种实现略 有差异。要查看命令的不同实现，可以使用 type 命令的-a 选项。
- 注意， which 命令只显示外部 命令文件。
- 对于有多种实现的命令，如果想使用其外部命令实现，直接指明对应的文件即可。例如，要使用外部命令 pwd，可以输入/usr/bin/pwd。
- 使用 history 命令
    - bash shell 会跟踪你最近使用过的命令。history是一个实用的内建命令，能帮助你管理先前执行过的命令。
    - 要查看最近用过的命令列表，可以使用不带任何选项的 history 命令。
    - 可以设置保存在 bash 历史记录中的命令数量。为此，需要修改名为 HISTSIZE 的环境变量。
    - 输入 !!， 然后按 Enter 键，唤回并重用最近那条命令。当输入!!时， bash 会先显示从 shell 的历史记录中唤回的命令， 然后再执行该命令。
    - 命令历史记录被保存在位于用户主目录的隐藏文件.bash_history之中。
    - 在 CLI 会话期间， bash 命令的历史记录被保存在内存中。当shell 退出时才被写入历史文件。
    - 可以在不退出 shell  的情况下强制将命令历史记录写入.bash_history 文件。为此，需要使用 history 命令的-a 选项。
    - 如果打开了多个终端会话，则仍然可以使用 history -a 命令在每个打开的会话中向 .bash_history 文件添加记录。但是历史记录并不会在其他打开的终端会话中自动更新。 这是因为.bash_history 文件只在首次启动终端会话的时候才会被读取。要想强制重新读取.bash_history 文件，更新内存中的终端会话历史记录，可以使用history -n 命令。
    - 可以唤回历史记录中的任意命令。只需输入惊叹号和命令在历史记录中的编号即可。和执行最近的命令一样， bash shell 会先显示从历史记录中唤回的命令， 然后再执行该命令。
    - 如果需要清除命令历史，很简单，输入 history -c 即可。接下来再输入 history-a，清除 .bash_history 文件。
    - 可以通过 man history 来查看 history 命令的 bash手册页。
- 使用命令别名
    - alias 命令是另一个实用的 shell 内建命令。命令别名允许为常用命令及其参数创建另一个名称，从而将输入量减少到最低。
    - 使用 alias 命令以及选项-p 可以查看当前可用的别名。
    - alias ls='ls --color=auto'
        - 该别名中加入了 --color=auto 选项，以便在终端支持彩色显示的情况下， ls 命令可以使用色彩编码（比如， 使用蓝色表示目录）。LS_COLORS 环境变量（环境变量的相关内容参见第 6 章） 控制着所用的色彩编码。
        - 如果经常跳转于不同的发行版，那么在使用色彩编码来分辨某个名称究竟是目录还是文件时，一定要小心。因为色彩编码并未实现标准化， 所以最好使用 ls -F 来判断文件类型。
    - 可以使用 alias 命令创建自己的别名:
        ```
        $ alias li='ls -i'
        ```
        - 要注意，因为命令别名属于内建命令，所以别名仅在其被定义的 shell 进程中才有效。
        - 如果需要，可以在命令行中输入 unalias alias-name 删除指定的别名。记住， 如果被删除的别名不是你设置的， 那么等下次重新登录系统的时候， 该别名就会再次出现。可 以通过修改环境文件永久地删除某个别名。
