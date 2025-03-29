###### 十六、脚本控制

1. Linux 利用信号与系统中的进程进行通信。

2. 重温 Linux 信号
- Linux 系统和应用程序可以产生超过 30 个信号。在 shell 脚本编程时会遇到的最常见的 Linux 系统信号如下：
    信号 | 值 | 描述
    ---|---|---
    1 | SIGHUP | 挂起（hang up ）进程
    2 | SIGINT | 中断（interrupt）进程
    3 | SIGQUIT | 停止（stop）进程
    9 | SIGKILL | 无条件终止（terminate）进程
    15 | SIGTERM | 尽可能终止进程
    18 | SIGCONT | 继续运行停止的进程
    19 | SIGSTOP | 无条件停止，但不终止进程
    20 | SIGTSTP | 停止或暂停（ pause ），但不终止进程

- 在默认情况下， bash shell 会忽略收到的任何 SIGQUIT(3)信号和 SIGTERM(15)信号（因此交互式 shell 才不会被意外终止）。但是，bash shell 会处理收到的所有 SIGHUP(1)信号和 SIGINT(2)信号。
- 如果收到了 SIGHUP 信号（比如在离开交互式 shell 时）， bash shell 就会退出。但在退出之前，它会将 SIGHUP 信号传给所有由该 shell 启动的进程， 包括正在运行的shell 脚本。
- 随着收到 SIGINT 信号， shell 会被中断。 Linux 内核将不再为 shell 分配 CPU 处理时间。当出现这种情况时， shell 会将 SIGINT 信号传给由其启动的所有进程，以此告知出现的状况。
- shell 会将这些信号传给 shell 脚本来处理。而 shell 脚本的默认行为是忽略这些信号，因为可能不利于脚本运行。

3. bash shell 允许使用键盘上的组合键来生成两种基本的 Linux 信号。这个特性在需要停止或暂停失控脚本时非常方便。
- 中断进程
    - Ctrl+C 组合键会生成 SIGINT 信号，并将其发送给当前在 shell 中运行的所有进程。
    - sleep 命令会按照指定的秒数暂停 shell 操作一段时间，然后返回 shell 提示符。 Ctrl+C 组合键会发送 SIGINT 信号，停止 shell 中当前运行的进程。在超时前（60 秒）按下 Ctrl+C 组合键，就可以提前终止 sleep 命令。
- 暂停进程
    - 也可以暂停进程，而不是将其终止。尽管有时这可能比较危险（比如，脚本打开了一个关键的系统文件的文件锁），但它往往可以在不终止进程的情况下，使你能够深入脚本内部一窥究竟。
    - Ctrl+Z 组合键会生成 SIGTSTP 信号，停止 shell 中运行的任何进程。停止（stopping）进程跟终止（terminating）进程不同，前者让程序继续驻留在内存中，还能从上次停止的位置继续运行。 
    - 当使用 Ctrl+Z 组合键时， shell 会通知你进程已经被停止了：
        ```
        $ sleep 60
        ^Z
        [1]+  Stopped                  sleep 60
        $
        ```
        - 方括号中的数字是 shell 分配的作业号。shell 将运行的各个进程称为作业，并为作业在当前 shell 内分配了唯一的作业号。作业号从 1 开始，然后是 2，依次递增。
    - 如果 shell 会话中有一个已停止的作业， 那么在退出shell 时， bash 会发出提醒：
        ```
        $ sleep 70
        ^Z
        [2]+  Stopped                  sleep 70
        $
        $ exit
        logout
        There are stopped jobs.
        $
        ```
    - 可以用 ps 命令查看已停止的作业：
        ```
        $ ps -l
        F   S   UID PID PPID    [...]   TTY TIME    CMD
        0   S   1001 1509 1508  [...] pts/0 00:00:00 bash
        0   T   1001 1532 1509  [...] pts/0 00:00:00 sleep
        0   T   1001 1533 1509  [...] pts/0 00:00:00 sleep
        0   R   1001 1534 1509  [...] pts/0 00:00:00 ps
        $
        ```
        - 在 S 列（进程状态）中， ps 命令将已停止作业的状态显示为 T。这说明命令要么被跟踪，要么被停止。
    - 如果在有已停止作业的情况下仍旧想退出 shell，则只需再输入一遍 exit 命令即可。 shell 会退出，终止已停止作业。
    - 或者，如果知道已停止作业的 PID，那就可以用 kill 命令发送 SIGKILL(9)信号将其终止：
    - 每当 shell 生成命令行提示符时，也会显示 shell 中状态发生改变的作业。“杀死”作业后， shell 会显示一条消息，表示运行中的作业已被“杀死”，然后生成提示符。
    - 在某些 Linux 系统中， “杀死”作业时不会得到任何回应。但当下次执行能让 shell 生成命令行提示符的操作时（比如， 按下 Enter 键），会看到一条消息，表示作业已被“杀死”。

4. 捕获信号
- 也可以用其他命令在信号出现时将其捕获，而不是忽略信号。 trap 命令可以指定 shell 脚本需要侦测并拦截的 Linux 信号。如果脚本收到了 trap 命令中列出的信号，则该信号不再由 shell 处理，而是由本地处理。
- trap 命令的格式如下：
    ```
    trap commands signals
    ```
    - 在 trap 命令中， 需要在 commands 部分列出想要 shell 执行的命令， 在 signals 部分列出 想要捕获的信号（多个信号之间以空格分隔）。指定信号的时候，可以使用信号的值或信号名。
- 如何使用 trap 命令捕获 SIGINT 信号并控制脚本的行为：
    ```
    $ cat trapsignal.sh
    #!/bin/bash
    #Testing signal trapping
    #
    trap "echo ' Sorry! I have trapped Ctrl-C'" SIGINT
    #
    echo This is a test script.
    #
    count=1
    while [ $count -le 5 ]
    do
        echo "Loop #$count"
        sleep 1
        count=$[ $count + 1 ]
    done
    #
    echo "This is the end of test script."
    exit
    $
    ```
    - 每次侦测到 SIGINT 信号时，本例中的 trap 命令都会显示一行简单的文本消息。捕获这些信号可以阻止用户通过组合键 Ctrl+C 停止脚本。
    - 每次使用 Ctrl+C 组合键，脚本都会执行 trap 命令中指定的echo 语句，而不是忽略信号并让 shell 停止该脚本。
- 如果脚本中的命令被信号中断，使用带有指定命令的 trap 未必能让被中断的命令继续执行。为了保证脚本中的关键操作不被打断，请使用带有空操作命令的 trap 以及要捕获的信号列表，例如：
    ```
    trap "" SIGINT
    ```
    - 这种形式的 trap 命令允许脚本完全忽略 SIGINT 信号，继续执行重要的工作。

5. 捕获脚本退出
- 除了在 shell 脚本中捕获信号， 也可以在 shell 脚本退出时捕获信号。这是在 shell 完成任务时执行命令的一种简便方法。
- 要捕获 shell 脚本的退出， 只需在 trap 命令后加上 EXIT 信号即可：
    ```
    $ cat trapexit.sh
    #!/bin/bash
    #Testing exit trapping
    #
    trap "echo Goodbye..." EXIT
    #
    count=1
    while [ $count -le 5 ]
    do
        echo "Loop #$count"
        sleep 1
        count=$[ $count + 1 ]
    done
    #
    exit
    $
    ```
    - 当脚本运行到正常的退出位置时，触发了 EXIT，shell 执行了在 trap 中指定的命令。如果提前退出脚本，则依然能捕获到 EXIT。
    - 该例子中，因为 SIGINT 信号并未在 trap 命令的信号列表中，所以当按下Ctrl+C 组合键发送 SIGINT 信号时，脚本就退出了。但在退出之前已经触发了 EXIT，于是 shell 会执行 trap 命令。

6. 修改或移除信号捕获
- 要想在脚本中的不同位置进行不同的信号捕获处理， 只需重新使用带有新选项的 trap 命令 即可：
    ```
    $ cat trapmod.sh
    #!/bin/bash
    #Modifying a set trap
    #
    trap "echo ' Sorry...Ctrl-C is trapped.'" SIGINT
    #
    count=1
    while [ $count -le 3 ]
    do
        echo "Loop #$count"
        sleep 1
        count=$[ $count + 1 ]
    done
    #
    trap "echo ' I have modified the trap!'" SIGINT
    #
    count=1
    while [ $count -le 3 ]
    do
        echo "Second Loop #$count"
        sleep 1
        count=$[ $count + 1 ]
    done
    #
    exit
    $
    ```
    - 修改了信号捕获之后， 脚本处理信号的方式就会发生变化。但如果信号是在捕获被修改前接收到的，则脚本仍然会根据原先的 trap 命令处理该信号。
- 如果在交互式 shell 会话中使用trap 命令，可以使用 trap -p 查看被捕获的信号。如果什么都没有显示，则说明 shell 会话按照默认方式处理信号。
- 也可以移除已设置好的信号捕获。在 trap 命令与希望恢复默认行为的信号列表之间加上两个连字符即可：
    ```
    trap "echo ' Sorry...Ctrl-C is trapped.'" SIGINT
    trap -- SIGINT
    ```
    - 也可以在 trap 命令后使用单连字符来恢复信号的默认行为。单连字符和双连字符的效果一样。
    - 移除信号捕获后， 脚本会按照默认行为处理 SIGINT 信号， 也就是终止脚本运行。但如果信号是在捕获被移除前接收到的，那么脚本就会按照原先 trap 命令中的设置进行处理。

7. 以后台模式运行脚本
- 使用 ps -e 命令，可以看到 Linux 系统中运行的多个进程：
    ```
    $ ps -e
    PID TTY          TIME CMD
    1 ?        00:00:02 systemd
    2 ?        00:00:00 kthreadd
    3 ?        00:00:00 rcu_gp
    4 ?        00:00:00 rcu_par_gp
    [...]
    2585 pts/0    00:00:00 ps
    $
    ```
    - 其实有很多进程没有运行在任何终端——它们是在后台运行的。在后台模式中，进程运行时不和终端会话的 STDIN、STDOUT 以及 STDERR 关联。

8. 后台运行脚本
- 以后台模式运行 shell 脚本只需在脚本名后面加上&即可：
    ```
    $ ./backgroundscript.sh &
    [1] 2595
    $
    [1]+  Done                      ./backgroundscript.sh
    ```
    - 方括号中的数字（1）是 shell 分配给后台进程的作业号，之后的数字（2595）是 Linux 系统为进程分配的进程 ID（PID）。
    - 当后台进程结束时，终端上会显示一条消息。其中指明了作业号、作业状态（Done），以及用于启动该作业的命令（删除了&）。
- 在脚本名之后加上&会将脚本与当前 shell 分离开来，并将脚本作为一个独立的后台进程运行。
- 注意，当后台进程运行时， 它仍然会使用终端显示器来显示 STDOUT 和 STDERR 消息。最好是将后台脚本的 STDOUT 和 STDERR 进行重定向，避免这种杂乱的输出。

9. 运行多个后台作业
- 在使用命令行提示符的情况下，可以同时启动多个后台作业。
- 每次启动新作业时， Linux 系统都会为其分配新的作业号和PID。通过 ps 命令可以看到后台脚本处于运行状态。
- 在终端会话中使用后台进程一定要小心。注意，在 ps 命令的输出中，每一个后台进程都和终端会话（pts/0）终端关联在一起。如果终端会话退出，那么后台进程也会随之退出。
- 当要退出终端会话时，如果还有被停止的进程，就会出现警告信息。但如果是后台进程，则只有部分终端仿真器会在退出终端会话前提醒你尚有后台作业在运行。

10. 在非控制台下运行脚本
- nohup 命令能阻断发给特定进程的 SIGHUP 信号。当退出终端会话时，这可以避免进程退出。
- nohup 命令的格式如下：
    ```
    nohup command
    ```
- 使用一个后台脚本作为 command：
    ```
    $ nohup ./testAscript.sh &
    [1] 1828
    $ nohup: ignoring input and appending output to 'nohup.out'
    ```
    - 和普通后台进程一样，shell 会给 command 分配一个作业号，Linux 系统会为其分配一个 PID 号。区别在于， 当使用 nohup 命令时，如果关闭终端会话，则脚本会忽略其发送的 SIGHUP 信号。
    - 由于 nohup 命令会解除终端与进程之间的关联， 因此进程不再同 STDOUT 和 STDERR 绑定在一起。为了保存该命令产生的输出，nohup 命令会自动将 STDOUT 和 STDERR 产生的消息重定向到一个名为 nohup.out 的文件中。
    - nohup.out 文件一般在当前工作目录中创建，否则会在$HOME 目录中创建。
    - nohup.out 文件包含了原本要发送到终端显示器上的所有输出。进程结束之后，可以查看 nohup.out 文件中的输出结果。nohup.out 文件中的输出结果和脚本在命令行中运行时产生的一样。
- 如果使用nohop运行了另一个命令，那么该命令的输出会被追加到已有的nohup.out文件中。所以当运行位于同一目录中的多个命令时，所有的命令输出都会发送到同一个 nohup.out 文件中。
- 有了 nohup，就可以在后台运行脚本。在无须停止脚本进程的情况下，登出终端会话去完成其他任务，随后再检查结果。

11. 作业控制
- 在作业停止后，Linux 系统会让你选择是“杀死”该作业还是重启该作业。用 kill 命令可以“杀死”该作业。要重启停止的进程，则需要向其发送 SIGCONT 信号。
- 作业控制包括启动、停止、“杀死”以及恢复作业。通过作业控制，能完全控制 shell 环境中所有进程的运行方式。

12. 查看作业
- jobs 是作业控制中的关键命令，该命令允许用户查看 shell 当前正在处理的作业。
- 通过 jobs 命令可以查看分配给 shell 的作业，例如：
    ```
    $ jobs
    [1]+  Stopped   ./jobcontrol.sh
    [2]-  Running   ./jobcontrol.sh > jobcontrol.out &
    ```
    - jobs 命令显示了一个已停止的作业和一个运行中的作业，以及两者的作业号和作业使用的命令。
    - 带有加号的作业为默认作业。如果作业控制命令没有指定作业号，则引用的就是该作业。带有减号的作业会在默认作业结束之后成为下一个默认作业。
    - 任何时候，不管 shell 中运行着多少作业，带加号的作业只能有一个，带减号的作业也只能有一个。
- 可以使用 jobs 命令的-l 选项（小写字母 l）查看作业的 PID。
    ```
    $ jobs -l
    [1]+  1580  Stopped   ./jobcontrol.sh
    [2]-  1603  Running   ./jobcontrol.sh > jobcontrol.out &
    ```
- jobs 命令提供了一些命令行选项：
    选项 | 描述
    ---|---
    -l  | 列出进程的 PID 以及作业号
    -n  | 只列出上次 shell 发出通知后状态发生改变的作业
    -p  | 只列出作业的 PID
    -r  | 只列出运行中的作业
    -s  | 只列出已停止的作业

- 如果需要删除已停止的作业， 那么使用 kill 命令向其 PID 发送 SIGKILL(9)信号即可。
    ```
    $ jobs -l
    [1]+  1580 Stopped  ./jobcontrol.sh
    $
    $ kill -9 1580
    [1]+  Killed    ./jobcontrol.sh
    $
    ```

13. 重启已停止的作业
- 在 bash 作业控制中，可以将已停止的作业作为后台进程或前台进程重启。前台进程会接管当前使用的终端，因此在使用该特性时要小心。
- 要以后台模式重启作业，可以使用 bg 命令：
    ```
    $ ./restartjob.sh
    ^Z
    [1]+  Stopped                  ./restartjob.sh
    $
    $ bg
    [1]+ ./restartjob.sh &
    $
    $ jobs
    [1]+  Running                  ./restartjob.sh &
    $
    ```
    - 因为该作业是默认作业（从加号可以看出），所以仅使用 bg 命令就可以将其以后台模式重启。注意，当作业被转入后台模式时，并不会显示其 PID。
- 如果存在多个作业，则需要在 bg 命令后加上作业号，以便于控制：
    ```
    $ jobs
    $
    $ ./restartjob.sh
    ^Z
    [1]+  Stopped                  ./restartjob.sh
    $
    $ ./newrestartjob.sh
    ^Z
    [2]+  Stopped                  ./newrestartjob.sh
    $
    $ bg 2
    [2]+ ./newrestartjob.sh &
    $
    $ jobs
     
    [1]+  Stopped                   ./restartjob.sh
    [2]-  Running                   ./newrestartjob.sh & 
    $
    ```
    - bg 2 命令用于将第二个作业置于后台模式。
    - 注意，当使用 jobs 命令时，它列出了作业及其状态，即便默认作业当前并未处于后台模式。
- 要以前台模式重启作业，可以使用带有作业号的 fg 命令：
    ```
    $ jobs
    [1]+  Stopped   ./restartjob.sh
    [2]-  Running   ./newrestartjob.sh &
    $
    $ fg 2
    ./newrestartjob.sh
    This is the script's end.
    $
    ```
    - 由于作业是在前台运行的，因此直到该作业完成后，命令行界面的提示符才会出现。

14. 调整谦让度
- 在多任务操作系统（比如 Linux）中，内核负责为每个运行的进程分配 CPU 时间。调度优先级 ［也称为谦让度（nice value）］是指内核为进程分配的 CPU时间（相对于其他进程）。在 Linux 系统中，由 shell 启动的所有进程的调度优先级默认都是相同的。
- 调度优先级是一个整数值， 取值范围从-20（最高优先级）到+19（最低优先级）。在默认情况下，bash shell 以优先级 0 来启动所有进程。**-20（最低值）代表最高优先级，+19（最高值）代表最低优先级。**

15. nice 命令
- nice 命令允许在启动命令时设置其调度优先级。要想让命令以更低的优先级运行，只需用 nice 命令的-n 选项指定新的优先级即可：
    ```
    $ nice -n 10 ./jobcontrol.sh > jobcontrol.out &
    [2] 16462
    $
    $ ps -p 16462 -o pid,ppid,ni,cmd
    PID    PPID  NI CMD
    16462    1630  10 /bin/bash ./jobcontrol.sh
    $
    ```
    - nice 命令和要启动的命令必须出现在同一行中。
    - ps 命令的输出证实，谦让度（ NI 列）已经调整到了 10。
    - nice 命令使得脚本以更低的优先级运行。
- nice 命令会阻止普通用户提高命令的优先级。注意，即便提高其优先级的操作没有成功，指定的命令依然可以运行。只有 root 用户或者特权用户才能提高作业的优先级。
- nice 命令的-n 选项并不是必需的，直接在连字符后面跟上优先级也可以。当要设置的优先级是负数时，这种写法则很容易造成混淆，因为出现了双连字符。在这种情况下，最好还是使用-n 选项。

16. renice 命令
- 如果想要修改系统中已运行命令的优先级，可以用 renice 命令通过指定运行进程的 PID 来改变其优先级：
    ```
    $ ./jobcontrol.sh > jobcontrol.out &
    [2] 16642
    $
    $ ps -p 16642 -o pid,ppid,ni,cmd
    PID    PPID  NI CMD
    16642  1630  0 /bin/bash ./jobcontrol.sh
    $
    $ renice -n 10 -p 16642
    16642 (process ID) old priority 0, new priority 10
    $
    $ ps -p 16642 -o pid,ppid,ni,cmd
    PID    PPID  NI CMD
    16642  1630  10 /bin/bash ./jobcontrol.sh
    $
    ```
    - renice 命令会自动更新运行进程的调度优先级。
    - renice 命令对于非特权用户也有一些限制：只能对属主为自己的进程使用 renice 且只能降低调度优先级。但是，root 用户和特权用户可以使用renice 命令对任意进程的优先级做任意调整。

17. 定时运行作业
- Linux 系统提供了多个在预选时间运行脚本的方法：at 命令、 cron 表以及 anacron。每种方法都使用不同的技术 来安排脚本的运行时间和频率。

18. 使用 at 命令调度作业
- at 命令允许指定 Linux 系统何时运行脚本。该命令会将作业提交到队列中，指定 shell 何时运行该作业。
- at 的守护进程 atd 在后台运行，在作业队列中检查待运行的作业。很多 Linux 发行版会在启动时运行此守护进程，但有些发行版甚至都没安装这个软件包。如果你的 Linux 属于后一种情况，则可以自行安装软件包 at。
- atd 守护进程会检查系统的一个特殊目录（通常位于/var/spool/at 或/var/spool/cron/atjobs），从中获取 at 命令提交的作业。在默认情况下，atd 守护进程每隔 60 秒检查一次这个目录。如果其中有作业， 那么 atd 守护进程就会查看此作业的运行时间。如果时间跟当前时间一致， 就运行此作业。
- at 命令的格式：
    ```
    at [-f filename] time
    ```
    - 在默认情况下，at 命令会将 STDIN 的输入放入队列。你可以用-f 选项指定用于从中读取命令（脚本文件）的文件名。
    - time 选项指定了你希望何时运行该作业。如果指定的时间已经过去，那么 at 命令会在第二天的同一时刻运行指定的作业。
    - at 命令能识别多种时间格式：
        - 标准的小时和分钟，比如 10:15。
        - AM/PM 指示符，比如 10:15 PM。
        - 特定的时间名称，比如 now、noon、midnight 或者 teatime（4:00 p.m.）
    - 也可以通过不同的日期格式指定特定的日期：
        - 标准日期，比如 MMDDYY 、MM/DD/YY 或 DD.MM.YY。
        - 文本日期，比如 Jul 4或 Dec 25 ，加不加年份均可。
        - 时间增量。
            - Now + 25 minutes
            - 10:15 PM tomorrow
            - 10:15 + 7 days
- at 命令可用的日期和时间格式有很多种，具体参见/usr/share/doc/at/timespec 文件。
- 在使用 at 命令时，该作业会被提交至作业队列。作业队列保存着通过 at 命令提交的待处理作业。针对不同优先级，有 52 种作业队列。作业队列通常用小写字母 a~z 和大写字母 A~Z 来指代，A 队列和 a 队列是两个不同的队列。
- batch 命令可以安排脚本在系统处于低负载时运行。batch 命令是一个脚本（/usr/bin/batch），它会调用 at 命令将作业提交到 b 队列中。
- 作业队列的字母排序越高，此队列中的作业运行优先级就越低（谦让度更大）。在默认情况下， at 命令提交的作业会被放入 a 队列。如果想以较低的优先级运行作业， 可以用-q 选项指定其他的队列。如果相较于其他进程你希望你的作业尽可能少地占用 CPU，可以将其放入 z 队列。
- 当在 Linux 系统中运行 at 命令时，显示器并不会关联到该作业。 Linux 系统反而会将提交该作业的用户 email 地址作为 STDOUT 和 STDERR。任何送往 STDOUT 或 STDERR 的输出都会通过邮件系统传给该用户。
- 在 CentOS发行版中使用 at 命令调度作业的例子：
    ```
    $ cat tryat.sh
    #!/bin/bash
    # Trying out the at command
    #
    echo "This script ran at $(date +%B%d,%T)"
    echo
    echo "This script is using the $SHELL shell."
    echo
    sleep 5
    echo "This is the script's end."
    #
    exit
    $
    $ at -f tryat.sh now
    warning: commands will be executed using /bin/sh
    job 3 at Thu Jun 18 16:23:00 2020
    $
    ```
    - at 命令会显示分配给作业的作业号以及为作业安排的运行时间。
    - -f 选项指明使用哪个脚本文件。now 指示 at 命令立刻执行该脚本。
    - 无须在意 at 命令输出的警告消息，因为脚本的第一行是#!/bin/bash，该命令会由 bash shell 执行。
- at 命令通过 sendmail 应用程序发送 email。如果系统中没有安装 sendmail，那就无法获得任何输出。因此在使用 at 命令时，最好在脚本中对 STDOUT 和 STDERR 进行重定向，如下例所示：
    ```
    $ cat tryatout.sh
    #!/bin/bash
    # Trying out the at command redirecting output
    #
    outfile=$HOME/scripts/tryat.out
    #
    echo "This script ran at $(date +%B%d,%T)" > $outfile
    echo >> $outfile
    echo "This script is using the $SHELL shell." >> $outfile
    echo >> $outfile
    sleep 5
    echo "This is the script's end." >> $outfile
    #
    exit
    $
    $ at -M -f tryatout.sh now
    warning: commands will be executed using /bin/sh
    job 4 at Thu Jun 18 16:48:00 2020
    $
    $ cat $HOME/scripts/tryat.out
    This script ran at June18,16:48:21
    
    This script is using the /bin/bash shell.
    
    This is the script's end.
    $
    ```
    - 如果不想在 at 命令中使用 email 或者重定向，则最好加上-M 选项，以禁止作业产生的输出信息。
- atq 命令可以查看系统中有哪些作业在等待：
    ```
    $ at -M -f tryatout.sh teatime
    warning: commands will be executed using /bin/sh
    job 5 at Fri Jun 19 16:00:00 2020
    $
    $ at -M -f tryatout.sh tomorrow
    warning: commands will be executed using /bin/sh
    job 6 at Fri Jun 19 16:53:00 2020
    $ atq
    1   Thu Jun 18 16:11:00 2020 a christine
    5   Fri Jun 19 16:00:00 2020 a christine
    6   Fri Jun 19 16:53:00 2020 a christine
    ```
    - 作业列表中显示了作业号、系统运行该作业的日期和时间，以及该作业所在的作业队列。
- 一旦知道了哪些作业正在作业队列中等待，就可以用 atrm 命令删除等待中的作业。指定要删除的作业号即可：
    ```
    $ atq
    1   Thu Jun 18 16:11:00 2020 a christine
    5   Fri Jun 19 16:00:00 2020 a christine
    6   Fri Jun 19 16:53:00 2020 a christine
    $ atrm 5
    $ atq
    1   Thu Jun 18 16:11:00 2020 a christine
    6   Fri Jun 19 16:53:00 2020 a christine
    ```
    - 只能删除自己提交的作业，不能删除其他人的。

19. 调度需要定期运行的脚本
- Linux 系统使用 cron 程序调度需要定期执行的作业。cron 在后台运行，并会检查一个特殊的表（cron 时间表），从中获知已安排执行的作业。
- cron 时间表通过一种特别的格式指定作业何时运行，其格式如下：
    ```
    minutepasthour hourofday dayofmonth month dayofweek command
    ```
    - cron 时间表允许使用特定值、取值范围（比如 1~5）或者通配符（星号）来指定各个字段。
    - dayofmonth 字段指定的是月份中的日期值（1~31）。
- 如果想在每天的 10:15 运行一个命令，可以使用如下 cron 时间表字段：
    ```
    15 10 * * * command
    ```
    - dayofmonth、month 以及 dayofweek 字段中的通配符表明，cron 会在每天 10:15 执行该命令。
- 要指定一条在每周一的下午 4:15（4:15 p.m. ）执行的命令， 可以使用军事时间（1:00 p.m. 是 13:00，2:00 p.m.是 14:00 ，3:00 p.m.是 15:00，以此类推），如下所示：
    ```
    15 16 * * 1 command
    ```
    - 24 小时制在美国和加拿大被称为军事时间（military time），在英国则被称为大陆时间（continental time）。
- 可以使用三字符的文本值（mon、tue、wed、thu、fri、sat、sun）或数值（0 或 7 代表 周日，6 代表周六）来指定 dayofweek 字段。
- 要想在每月第一天的中午 12 点执行命令，可以使用下列字段： 
    ```
    00 12 1 * * command
    ```
- 如何设置才能让命令在每月的最后一天执行，因为无法设置一个 dayofmonth 值，涵盖所有月份的最后一天。常用的解决方法是加一个 if-then 语句，在其中使用 date 命令检查明天的日期是不是某个月份的第一天（01）：
    ```
    00 12 28-31 * * if [ "$(date +%d -d tomorrow)" = 01 ] ; then command ; fi
    ```
这行脚本会在每天中午 12 点检查当天是不是当月的最后一天（28~31），如果是，就由 cron 执行 command。另一种方法是将 command 替换成一个控制脚本（controlling script），在可能是每月最后一 天的时候运行。控制脚本包含 if-then 语句，用于检查第二天是否为某个月的第一天。如果是，则由控制脚本发出命令，执行必须在当月最后一天执行的内容。
- 命令列表必须指定要运行的命令或脚本的完整路径。可以像在命令行中那样，添加所需的任何选项和重定向符：
    ```
    15 10 * * * /home/christine/backup.sh > backup.out
    ```
    - cron 程序会以提交作业的用户身份运行该脚本，因此你必须有访问该脚本（或命令）以及输出文件的合理权限。
- 每个用户（包括 root 用户） 都可以使用自己的 cron 时间表运行已安排好的任务。 Linux 提供了 crontab 命令来处理 cron 时间表。要列出已有的 cron 时间表，可以用-l 选项：
    ```
    $ crontab -l
    no crontab for christine
    $
    ```
    - 在默认情况下，用户的 cron 时间表文件并不存在。
    - 可以使用-e 选项向 cron 时间表添加字段。 在添加字段时， crontab 命令会启动一个文本编辑器，使用已有的 cron 时间表作为文件内容（如果时间表不存在，就是一个空文件）。
- 如果创建的脚本对于执行时间的精确性要求不高，则用预配置的 cron 脚本目录会更方便。预配置的基础目录共有 4 个： hourly 、daily 、monthly 和 weekly。
    ```
    $ ls /etc/cron.*ly
    /etc/cron.daily:
    0anacron  apt-compat    cracklib-runtime  logrotate  [...]
    apport    bsdmainutils  dpkg              man-db     [...]
    
    /etc/cron.hourly:
    
    /etc/cron.monthly:
    0anacron
    
    /etc/cron.weekly:
    0anacron  man-db  update-notifier-common
    $
    ```
    - 如果你的脚本需要每天运行一次，那么将脚本复制到 daily 目录， cron 就会每天运行它。
- 如果某个作业在 cron 时间表中设置的运行时间已到，但这时候 Linux 系统处于关闭状态，那么该作业就不会运行。当再次启动系统时， cron 程序不会再去运行那些错过的作业。为了解决这个问题，许多 Linux 发行版提供了 anacron 程序。
- 如果 anacron 判断出某个作业错过了设置的运行时间，它会尽快运行该作业。这意味着如果 Linux 系统关闭了几天，等到再次启动时，原计划在关机期间运行的作业会自动运行。有了 anacron，就能确保作业一定能运行，这正是通常使用 anacron 代替 cron 调度作业的原因。
- anacron 程序只处理位于 cron 目录的程序，比如/etc/cron.monthly。它通过时间戳来判断作业是否在正确的计划间隔内运行了。每个 cron 目录都有一个时间戳文件，该文件位于/var/spool/anacron：
    ```
    $ ls /var/spool/anacron
    cron.daily  cron.monthly  cron.weekly
    $
    $ sudo cat /var/spool/anacron/cron.daily
    [sudo] password for christine:
    20200619
    $
    ```
- anacron 程序使用自己的时间表（通常位于/etc/anacrontab）来检查作业目录。
- anacron 时间表的基本格式和 cron 时间表略有不同：
    ```
    period delay identifier command
    ```
    - period 字段定义了作业的运行频率（以天为单位）。anacron 程序用该字段检查作业的时间戳文件。delay 字段指定了在系统启动后，anacron 程序需要等待多少分钟再开始运行错过的脚本。
    - identifier 字段是一个独特的非空字符串，比如 cron.weekly。它唯一的作用是标识出现在日志消息和错误 email 中的作业。 command 字段包含了 run-parts 程序和一个 cron 脚本目录名。 run-parts 程序负责运行指定目录中的所有脚本。
- **anacron 不会运行位于/etc/cron.hourly 目录的脚本。 这是因为 anacron 并不处理执行时间需求少于一天的脚本。**

20. 使用新 shell 启动脚本
- Linux 系统提供了一些脚本文件，可以让脚本在启动新的 bash shell 时运行。与此类似，位于用户主目录中的启动文件（比如.bashrc）提供了一个位置，以存放新 shell 启动时需要运行 的脚本和命令。
- 基本上，以下所列文件中的第一个文件会被运行， 其余的则会被忽略。（这个列表中并没有$HOME/.bashrc 文件。这是因为该文件通常通过其他文件运行）
    - $HOME/.bash_profile
    - $HOME/.bash_login
    - $HOME/.profile
    - 因此，应该将需要在登录时运行的脚本放在上述第一个文件中。
- 每次启动新 shell，bash shell 都会运行.bashrc 文件。
- 一般而言，用户登录时会运行从$ HOME/.bash_profile 、$ HOME/.bash_login 或$ HOME/.profile 中找到的第一个文件，而$HOME/.bashrc 则是由非登录 shell（nonlogin shell）运行的文件。
- .bashrc 文件通常也借由某个 bash 启动文件来运行，因为.bashrc 文件会运行两次：一次是当用户登录 bash shell 时，另一次是当用户启动 bash shell 时。如果需要某个脚本在两个时刻都运行，可以将其放入该文件中。

21. 实战演练
- 例子：
    ```
    source $scriptToRun > $scriptOutput &   #Run script in background
    ```
    - 注意，以上脚本的运行方式并没有使用 bash 或./运行文件， 而是改用了 source 工具。这是另一种运行 bash 脚本的方法，称为源引（sourcing）。这种操作与使用 bash 运行脚本差不多，只是不会创建子 shell。
    - 当使用 source 命令运行脚本时，就像bash 一样，无须在文件中设置执行权限。

