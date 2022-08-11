###### 附录A、bash 命令快速指南

1. bash 内建命令
    命令 | 描述
    ---|---
    &           | 在后台启动作业
    ((x))       | 执行数学表达式 x
    .           | 在当前 shell 中读取并执行指定文件中的命令
    :           | 什么都不做，始终成功退出
    [ t ]       | 对条件表达式 t 进行求值
    [[ e ]]     | 对条件表达式 e 进行求值
    alias       | 为指定命令定义别名
    bg          | 将当前作业置于后台运行
    bind        | 将组合键绑定到 readline 函数或宏
    Break       | 退出 for、while、select 或 until 循环
    builtin     | 执行指定的 shell 内建命令
    caller      | 返回活动子函数调用的上下文
    case        | 根据模式有选择地执行命令
    cd          | 将当前目录切换为指定的目录
    command     | 执行指定的命令，不进行正常的 shell 查找（也就是说，绕过同名的别名或函数）
    compgen     | 为指定单词生成可能的补全匹配
    complete    | 显示指定的单词是如何补全的
    compopt     | 修改指定单词的补全选项
    continue    | 继续执行 for、while、select 或 until 循环的下一次迭代
    coproc      | 在后台生成子 shell 并执行指定的命令
    declare     | 声明变量或变量类型
    dirs        | 显示当前已保存的目录列表
    disown      | 从进程作业表中删除指定的作业
    echo        | 将指定字符串输出到 STDOUT
    enable      | 启用或禁用指定的内建 shell 命令
    eval        | 将指定的参数拼接成一个命令， 然后执行该命令
    exec        | 用指定命令替换 shell 进程
    exit        | 强制 shell 以指定的退出状态码退出
    export      | 设置可用于子 shell 进程的变量
    false       | 将结果设置为 false 状态
    fc          | 从历史记录列表（history list）中选择一组命令
    fg          | 将作业恢复至前台
    for         | 对列表中的每一项执行指定的命令
    function    | 定义一个 shell 脚本函数
    getopts     | 解析指定的位置参数
    hash        | 查找并记住指定命令的完整路径名
    help        | 显示 bash 内建命令的帮助页面
    history     | 显示命令历史记录
    if          | 根据条件表达式执行命令
    jobs        | 列出活动作业
    kill        | 向指定的进程 ID（PID ）发送系统信号
    let         | 计算数学表达式
    local       | 在函数中创建局部变量
    logout      | 退出已登录的 shell
    mapfile     | 从 STDIN 中读取输入并将其放入索引数组（每个数组元素包含一行） popd        | 从目录栈中删除记录
    printf      | 使用格式化字符串显示文本
    pushd       | 向目录栈压入一个目录
    pwd         | 显示当前工作目录的完整路径名
    read        | 从 STDIN 读取一行数据并将其中的每个单词赋给指定变量
    readarray   | 从 STDIN 读取数据行并将其放入索引数组（一个数组元素对应一行）
    readonly    | 从 STDIN 读取一行数据并将其赋给一个不可修改的变量
    return      | 使函数以某个值退出，该值可由调用脚本（calling script）提取
    select      | 显示带编号的单词列表， 允许用户进行选择
    set         | 设置并显示环境变量的值和 shell 属性
    shift       | 将位置参数依次向前移动一个位置
    shopt       | 打开/关闭 shell 选项
    source      | 在当前 shell 中读取并执行指定文件中的命令
    suspend     | 暂停 shell，直至收到 SIGCONT 信号
    test        | 根据指定条件返回退出状态码 0 或 1
    time        | 显示执行指定命令所累计的真实时间（real time）、用户时间和系统时间
    times       | 显示累计的用户时间和系统时间
    trap        | 如果接收到特定的系统信号，就执行指定命令
    true        | 返回为 0 的退出状态码
    type        | 显示指定的单词作为命令名时， 如何被 shell 解释（也就是显示指定名称是外部命令、内建命令、别名、 shell 关键字或shell 函数）
    typeset     | 声明变量或变量类型
    ulimit      | 为系统用户设置指定的资源上限
    umask       | 为新建的文件和目录设置默认权限
    unalias     | 删除指定的别名
    unset       | 删除指定的环境变量或 shell 属性
    until       | 执行指定的命令，直到条件语句返回 true
    wait        | 等待指定的进程结束，返回退出状态码
    while       | 当条件语句返回 true 时，执行指定的命令
    { c; }      | 在当前 shell 中指定一组命令

- 准确地说，表中的 coproc、funtion、while 和 until 属于 shell 关键字（keyword），并非内建命令，通过 type 命令即可得知。

2. bash shell 外部命令
    命令 | 描述
    ---|---
    at          | 在未来的特定时间执行指定的脚本或命令
    atq         | 显示 at 命令队列中的作业
    atrm        | 从 at 命令队列中删除指定的作业
    bash        | 执行来自标准输入或指定文件中的命令， 或是启动一个子 shell
    bc          | 使用 bc 的专用语言执行算术运算
    bzip2       | 采用 Burrows-Wheeler 块排序文本压缩算法和霍夫曼编码进行压缩 
    cat         | 列出指定文件的内容
    chage       | 修改指定系统用户账户的密码过期日期
    chfn        | 修改指定用户账户的备注信息
    chgrp       | 修改指定文件或目录的属组
    chmod       | 修改指定文件或目录的权限
    chown       | 修改指定文件或目录的属主
    chpasswd    | 读取包含用户名/密码的文件并更新相应用户的密码
    chsh        | 修改指定用户账户的默认 shell
    clear       | 清空终端仿真器或虚拟控制台终端中的文字
    compress    | 最初的 Unix 文件压缩工具
    cp          | 将指定文件复制到另一个位置
    crontab     | 启动用户的 cron 表文件对应的编辑器（如果允许的话）
    cut         | 打印文件中指定的部分
    date        | 以各种格式显示日期
    df          | 显示所有已挂载设备的当前磁盘使用情况
    dialog      | 在文本终端环境中创建窗口对话框
    du          | 显示指定目录的磁盘使用情况
    emacs       | 调用 Emacs 文本编辑器
    env         | 在修改过的环境中执行指定命令或显示所有的环境变量
    exit        | 终止当前进程
    expr        | 执行指定的算术表达式
    fdisk       | 维护或创建指定磁盘的分区表
    file        | 查看指定文件的文件类型
    find        | 查找文件
    free        | 查看系统可用的和已用的内存
    fsck        | 检查并根据需要修复指定的文件系统
    gawk        | 调用 gawk 编辑器
    Grep        | 在文件中查找指定模式的字符串
    gedit       | 调用 GNOME 桌面编辑器
    getopt      | 解析命令选项（包括长格式选项）
    gdialog     | 创建 GNOME Shell 窗口对话框
    groups      | 显示指定用户的组成员关系
    groupadd    | 创建新的用户组
    groupmod    | 修改已有的用户组
    gunzip      | 出自 GNU 项目的文件解压缩工具，采用 Lempel-Ziv 压缩算法
    gzcat       | 出自 GNU 项目的压缩文件内容显示工具， 采用 Lempel-Ziv 压缩算法
    gzip        | 出自 GNU 项目的文件压缩工具， 采用 Lempel-Ziv 压缩算法
    head        | 显示指定文件的开头部分
    kdialog     | 创建 KDE 窗口对话框
    killall     | 根据进程名向运行中的进程发送系统信号
    kwrite      | 调用 KWrite 文本编辑器
    less        | 查看文件内容的高级命令
    link        | 使用别名创建文件链接
    ln          | 创建指定文件的符号链接或硬链接
    ls          | 列出目录内容或文件信息
    lvcreate    | 创建 LVM 卷
    lvdisplay   | 显示 LVM 卷
    lvextend    | 增加 LVM 卷的大小
    lvreduce    | 减少 LVM 卷的大小
    mandb       | 创建能够使用手册页关键字进行搜索的数据库
    man         | 显示指定命令或话题的手册页
    mkdir       | 创建指定目录
    mkfs        | 使用指定文件系统格式化分区
    mktemp      | 创建临时文件或目录
    more        | 显示指定文件的内容，每显示一屏数据后就暂停
    mount       | 显示或挂载磁盘设备到虚拟文件系统中
    mv          | 重命名文件或目录
    nano        | 调用 nano 文本编辑器
    nice        | 在系统中用指定的优先级运行命令
    nohup       | 执行指定的命令，同时忽略 SIGHUP 信号
    passwd      | 修改用户的账户密码
    printenv    | 显示指定环境变量或所有的环境变量的值
    ps          | 显示系统中运行进程的信息
    pvcreate    | 创建物理 LVM 卷
    pvdisplay   | 显示物理 LVM 卷
    Pwd         | 显示当前工作目录
    renice      | 修改系统中运行进程的优先级
    rm          | 删除指定文件或目录
    rmdir       | 删除指定的空目录
    sed         | 调用流编辑器 sed
    setterm     | 修改终端设置
    sleep       | 在指定的一段时间内暂停 bash shell 操作
    sort        | 根据指定的顺序对文件内容进行排序
    stat        | 显示指定文件的相关信息
    sudo        | 以 root 用户账户身份运行应用程序
    tail        | 显示指定文件的末尾部分
    tar         | 将数据和目录归档到单个文件中
    tee         | 将信息发送到 STDOUT 和 STDIN
    top         | 显示活动进程以及重要的系统统计数据
    touch       | 新建一个空文件或更新已有文件的时间戳
    umount      | 从虚拟文件系统中卸载磁盘设备
    uptime      | 显示系统已经运行了多久
    useradd     | 新建用户账户
    userdel     | 删除用户账户
    usermod     | 修改用户账户
    vgchange    | 激活或停用 LVM 卷组
    vgcreate    | 创建 LVM 卷组
    vgdisplay   | 显示 LVM 卷组
    vgextend    | 增加 LVM 卷组大小
    vgreduce    | 减少 LVM 卷组大小
    vgremove    | 删除 LVM 卷组
    vi          | 调用 vi 文本编辑器
    vim         | 调用 vim 文本编辑器
    vmstat      | 生成一份详尽的系统内存和 CPU 使用情况的报告
    wc          | 显示文本文件统计情况
    whereis     | 显示指定命令的相关文件，包括二进制文件、源代码文件以及手册页
    which       | 查找可执行文件的位置
    who         | 显示当前系统中的登录用户
    whoami      | 显示当前用户的用户名
    xargs       | 从 STDIN 中获取数据项， 构建并执行命令
    xterm       | 调用 xterm 终端仿真器
    zenity      | 创建 GNOME Shell 窗口小部件
    zip         | Windows PKZIP 程序的 Unix 版本

3. bash shell 环境变量
    命令 | 描述
    ---|---
    | *                     | 包含所有命令行参数（以单个文本值的形式）
    | @                     | 包含所有命令行参数（以多个文本值的形式）
    | #                     | 命令行参数数目
    | ?                     | 最近使用的前台进程的退出状态码
    | -                     | 当前命令行选项标记
    $                       | 当前 shell 的进程 ID（PID ）
    !                       | 最近执行的后台进程的 PID
    0                       | 命令行中使用的命令名
    _                       | shell 的绝对路径名
    BASH                    | 用来调用 shell 的完整路径名
    BASHOPTS                | 已启用的 shell 选项（以冒号分隔形式显示）
    BASHPID                 | 当前 bash shell 的 PID
    BASH_ALIASES            | 数组变量，包含当前所用的别名
    BASH_ARGC               | 当前函数的参数数量
    BASH_ARGV               | 数组变量，包含所有的命令行参数
    BASH_CMDS               | 数组变量，包含命令的内部哈希表
    BASH_COMMAND            | 当前正在运行的命令名
    BASH_ENV                | 如果设置的话， 每个 bash 脚本都会尝试在运行前执行由该变量定义的启动文件
    BASH_EXECUTION_STRING   | 在-c 命令行选项中指定的命令
    BASH_LINENO             | 数组变量，包含脚本中每个命令的行号
    BASH_REMATCH            | 数组变量，包含正则表达式所匹配的文本（索引为 0 的元素是整个正则表达式所匹配的部分。索引为 n 的元素是第 n 个带有圆括号的子正则表达式所匹配的部分）
    BASH_SOURCE             | 数组变量，包含 shell 中已定义函数所在源文件名
    BASH_SUBSHELL           | 当前 shell 生成的子 shell 数目
    BASH_VERSINFO           | 数组变量，包含当前 bash shell 实例的主版本号和次版本号
    BASH_VERSION            | 当前 bash shell 实例的版本号
    BASH_XTRACEFD           | 如果设置为有效的文件描述符整数，则所产生跟踪信息会与诊断和错误消息分开。文件 描述符必须事先执行 set -x
    BROWSER                 | 首选 Web 浏览器的绝对路径名
    COLUMNS                 | 当前 bash shell 实例所用的终端宽度
    COMP_CWORD              | 变量 COMP_WORDS 的索引值， COMP_WORDS 包含当前光标所在的位置
    COMP_KEY                | 调用补全功能的按键
    COMP_LINE               | 当前命令行
    COMP_POINT              | 当前光标位置相对于当前命令起始处的索引
    COMP_TYPE               | 补全类型对应的整数值
    COMP_WORDBREAKS         | 在进行单词补全时作为单词分隔符的一组字符
    COMP_WORDS              | 数组变量，包含当前命令行上的所有单词
    COMPREPLY               | 数组变量，包含可能的补全结果
    COPROC                  | 数组变量，包含用于匿名协程 I/O 的文件描述符
    DBUS_SESSION_BUS_ADDRESS | 当前登录会话的 D-Bus 地址，用于提供连接映射
    DE                      | 当前登录 shell 的桌面环境
    DESKTOP_SESSION         | 在 LXDE 环境中，包含当前登录 shell 的桌面环境
    DIRSTACK                | 数组变量，包含目录栈当前内容
    DISPLAY                 | 图形应用程序映射，用于显示图形用户界面的位置
    EDITOR                  | 定义部分 shell 命令使用的默认编辑器
    EMACS                   | 如果设置的话， shell 会认为其使用的是 Emacs shell 缓冲区，同时禁止行编辑功能   
    ENV                     | 当 shell 以 POSIX模式调用时，每个 bash 脚本在运行之前都会执行由该环境变量所定义的启动文件
    EUID                    | 当前用户的有效用户 ID （数字形式）
    FCEDIT                  | fc 命令使用的默认编辑器
    FIGNORE                 | 以冒号分隔的后缀名列表，在文件名补全时会被忽略
    FUNCNAME                | 当前执行的 shell 函数的名称
    FUNCNEST                | 嵌套函数的最高级别
    GLOBIGNORE              | 以冒号分隔的模式列表， 文件名扩展时会将其忽略
    GROUPS                  | 数组变量，包含当前用户属组
    histchars               | 控制历史记录扩展，最多可有 3 个字符
    HISTCMD                 | 当前命令在历史记录中的编号
    HISTCONTROL             | 控制哪些命令会被保存在历史记录列表中
    HISTFILE                | 保存 shell 历史记录列表的文件名（默认是~/.bash_history）
    HISTFILESIZE            | 历史记录文件（history file）能保存的最大命令数量
    HISTIGNORE              | 以冒号分隔的模式列表， 用来决定哪些命令不会被保存在历史文件中 
    HISTSIZE                | 能写入历史记录列表（history list）的最大命令数量
    HISTTIMEFORMAT          | 如果设置的话， 该变量决定了历史文件条目时间戳所使用的格式字符串 
    HOME                    | 当前登录会话的主目录名
    HOSTALIASES             | 文件名， 某些 shell 命令要用到的各种主机名别名都保存在该文件中
    HOSTFILE                | shell 在补全主机名时要读取的文件名
    HOSTNAME                | 当前主机名
    HOSTTYPE                | 当前运行 bash shell 的机器
    IFS                     | 在分割单词时作为分隔符使用的一系列字符
    IGNOREEOF               | shell 在退出前必须收到的一系列 EOF 字符的数量。如果未设置，则默认是 1
    INFODIR                 | info 命令的搜索目录列表（以冒号分隔）
    INPUTRC                 | readline 初始化的文件名（默认是~/.inputrc）
    INVOCATION_ID           | systemd 用于标识登录 shell 和其他单元的 128 位（128-bit）随机标识符
    JOURNAL_STREAM          | 文件描述符的设备和 inode 编号（十进制格式） 列表（以冒号分隔）。仅当 STDOUT 或 STDERR 连接到日志系统时才设置
    LANG                    | shell 的语言环境种类（locale category）
    LC_ALL                  | 定义语言环境种类，能够覆盖 LANG 变量
    LC_ADDRESS              | 确定地址信息的显示方式
    LC_COLLATE              | 设置字符串排序时采用的排序规则
    LC_CTYPE                | 决定如何解释出现在文件名扩展和模式匹配中的字符
    LC_IDENTIFICATION       | 包含语言环境的元数据信息
    LC_MEASUREMENT          | 设置用于测量单位的语言环境
    LC_MESSAGES             | 决定在解释前面带有$的双引号字符串时采用的语言环境设置
    LC_MONETARY             | 定义货币数值的格式
    LC_NAME                 | 设置名称的格式
    LC_NUMERIC              | 决定格式化数字时采用的语言环境设置
    LC_PAPER                | 设置用于纸张标准和格式的语言环境
    LC_TELEPHONE            | 设置电话号码的结构
    LD_LIBRARY_PATH         | 以冒号分隔的目录列表， 其中的目录会先于标准库目录被搜索
    LC_TIME                 | 决定格式化日期和时间时采用的语言环境设置
    LINENO                  | 当前正在执行的脚本语句的行号
    LINES                   | 定义了终端上可见的行数
    LOGNAME                 | 当前登录会话的用户名
    LS_COLORS               | 定义用于显示文件名的颜色
    MACHTYPE                | 用“CPU–公司–系统”（CPU-company-system）格式定义的系统类型
    MAIL                    | 如果设置的话，定义当前登录会话的邮件文件会被一些邮件程序间歇地搜索，以查找新邮件
    MAILCHECK               | shell 应该多久检查一次新邮件（以秒为单位，默认为 60 秒）
    MAILPATH                | 以冒号分隔的邮件文件名列表，一些邮件程序会间歇性地在其中搜索新邮件 
    MANPATH                 | 以冒号分隔的手册页目录列表， 由 man 命令搜索
    MAPFILE                 | 数组变量，当未指定数组变量作为参数时，其中保存了 mapfile 所读入的文本 
    OLDPWD                  | shell 先前使用的工作目录
    OPTARG                  | 包含选项所需的参数值， 由 getopts 命令设置
    OPTERR                  | 如果设置为 1 ，则 bash shell 会显示 getopts 命令产生的错误
    OPTIND                  | getopts 命令要处理的下一个参数的索引
    OSTYPE                  | 定义了 shell 所在的操作系统
    PAGER                   | 设置某些 shell 命令在查看文件时使用的分页实用工具
    PATH                    | 以冒号分隔的目录列表， shell 会在其中搜索外部命令
    PIPESTATUS              | 数组变量，包含前台进程的退出状态
    POSIXLY_CORRECT         | 如果设置的话， bash 会以 POSIX 模式启动
    PPID                    | bash shell 父进程的 PID
    PROMPT_COMMAND          | 如果设置的话， 在显示命令行主提示符之前执行该命令
    PROMPT_DIRTRIM          | 用来定义使用提示符字符串\w 或\W 转义时显示的拖尾（trailing）目录名的数量（使用一组英文句点替换被删除的目录名）
    PS0                     | 如果设置的话， 该变量会指定在输入命令之后、执行命令之前，由交互式 shell 显示的内容
    PS1                     | 主命令行提示符
    PS2                     | 次命令行提示符
    PS3                     | select 命令的提示符
    PS4                     | 在命令行之前显示的提示符（如果使用了 bash 的-x 选项的话）
    PWD                     | 当前工作目录
    RANDOM                  | 返回一个介于 0 ～ 32 767 的随机数（对该变量的赋值可作为随机数生成器的种子） 
    READLINE_LINE           | 当使用 bind –x 命令时， 保存 Readline 缓冲区的内容
    READLINE_POINT          | 当使用 bind –x 命令时， 指明了 Readline 缓冲区内容插入点的当前位置 
    REPLY                   | read 命令的默认变量
    SECONDS                 | 自 shell 启动到现在的秒数（对其赋值会重置计数器）
    SHELL                   | bash shell 的完整路径
    SHELLOPTS               | 已启用的 bash shell 选项（以冒号分隔）
    SHLVL                   | shell 的层级；每启动一个新的 bash shell，该值增加 1
    TERM                    | 登录会话当前使用的终端类型， 相关信息由该变量所指向的文件提供 
    TERMCAP                 | 登录会话当前使用的终端类型， 相关信息由该变量提供
    TIMEFORMAT              | 指定了 shell 的时间显示格式
    TMOUT                   | select 命令和 read 命令在无输入的情况下等待多久（以秒为单位。默认值为 0，表示一直等待）
    TMPDIR                  | 目录名， 保存 bash shell 创建的临时文件
    TZ                      | 如果设置的话， 用于指定系统的时区
    TZDIR                   | 定义时区文件所在的目录
    UID                     | 当前用户的真实用户 ID （数字形式）
    USER                    | 当前登录会话的用户名
    VISUAL                  | 如果设置的话， 用于定义某些 shell 命令默认使用的全屏编辑器

- 可以使用 printenv 命令来显示当前定义的环境变量。在启动时建立的 shell 环境变量在不同的 Linux 发行版中（经常）会有所不同。
