###### 六、Linux 环境变量

1. bash shell 使用环境变量来存储 shell 会话和工作环境的相关信息（这也是被称作环境变量的原因）。环境变量允许在内存中存储数据， 以便 shell 中运行的程序或脚本能够轻松访问到这些数据。这也是存储持久数据的一种简便方法。

2. bash shell 中有两种环境变量。
    - 全局变量
    - 局部变量
- 尽管 bash shell 使用的专有环境变量是一致的，但不同的 Linux 发行版经常会添加自己的环境变量。

3. 全局环境变量
- 全局环境变量对于 shell 会话和所有生成的子 shell 都是可见的。局部环境变量则只对创建它的 shell 可见。如果程序创建的子 shell需要获取父 shell 信息，那么全局环境变量就能派上用场了。
- Linux 系统在你启动 bash 会话时就设置好了一些全局环境变量。系统环境变量基本上会使用全大写字母， 以区别于用户自定义的环境变量。
- 可以使用 env 命令或 printenv 命令来查看全局变量。
- 要显示个别环境变量的值， 可以使用 printenv 命令， 但不要使用env 命令，例如 $ printenv HOME。
- 也可以使用 echo 命令显示变量的值。在引用某个环境变量时， 必须在该变量名前加上美元 符号（ $），例如 $ echo $HOME。
- 使用 echo 命令时，在变量名前加上$可不仅仅是能够显示变量当前的值，它还能让变量作 为其他命令的参数，例如 $ ls $HOME。

4. 局部环境变量
- 局部环境变量只能在定义它的进程中可见。事实上，Linux 系统默认也定义了标准的局部环境变量。不过， 也可以定义自己的局部变量，这些变量被称为用户自定义局部变量。
- set 命令可以显示特定进程的所有环境变量，既包括局部变量、全局变量，也包括用户自定义变量。
- 所有通过 env 命令或 printenv 命令能看到的全局环境变量都出现在了set 命令的输出中。除此之外， 还包括局部环境变量、用户自定义变量以及局部 shell 函数（比如_command 函数）。
- set 命令既会显示全局和局部环境变量、用户自定义变量以及局部 shell 函数，还会按照字母顺序对结果进行排序。 与 set 命令不同， env 命令和 printenv 命令既不会对变量进行排序，也不会输出局部环境变量、局部用户自定义变量以及局部 shell 函数。在这种情况下， env 命令和 printenv命令的输出是重复的。

5. 设置局部用户自定义变量
- 启动 bash shell （或者执行 shell 脚本） 之后， 就能创建仅对该 shell 进程可见的局部用户自定 义变量。可以使用等号为变量赋值， 值可以是数值或字符串：
    ```
    $ my_variable=Hello
    $ echo $my_variable
    Hello
    
    $ my_variable="Hello World"
    $ echo $my_variable
    Hello World
    ```
    - 如果要引用 my_variable 变量的值， 使用$my_variable 即可。
    - 如果用于赋值的字符串包含空格，则必须用单引号或双引号来界定该字符串的起止。
    - 如果没有引号，则 bash shell 会将下一个单词（World）视为另一个要执行的命令。
    - 注意，你定义的局部变量用的是小写字母，而系统环境变量用的都是大写字母。
    - **记住， 在变量名、等号和值之间没有空格， 这一点非常重要。如果在赋值表达式中加上了空 格，那么 bash shell 会将值视为单独的命令。**
-  bash shell 的惯例是所有的环境变量均使用大写字母命名。如果是你自己创建或在 shell 脚本中使用的局部变量，则使用小写字母命名。变量名区分大小写。坚持使用小写字母命 名用户自定义的局部变量， 能够让你避免不小心与系统环境变量同名可能带来的灾难。
- 设置好局部变量后，就能在 shell 进程中随意使用了。但如果又生成了另一个 shell，则该变量在子 shell 中不可用。
- 类似地，如果在子进程中设置了一个局部变量， 那么一旦退出子进程，该局部变量就不能用了。返回父 shell 后，子 shell 中设置的局部变量就不存在了。可以通过将局部用户自定义变量改为全局变量来解决这个问题。

6. 设置全局环境变量
- 全局环境变量在设置该变量的父进程所创建的子进程中都是可见的。
- 创建全局环境变量的方 法是先创建局部变量，然后再将其导出到全局环境中。这可以通过 export 命令以及要导出的变量名（不加$符号）来实现：
    ```
    $ my_variable="I am Global now"
    $ export my_variable
    
    或者：可以将设置变量和导出变量放在一个命令里完成。
    $ export my_variable="I am Global Now"
    ```
- 修改子 shell 中的全局环境变量并不会影响父 shell 中该变量的值。子 shell 甚至无法使用 export 命令改变父 shell 中全局环境变量的值。

7. 删除环境变量
- 可以用 unset 命令删除已有的环境变量。在 unset 命令中引用环境变量时，记住不要使用$。
- **如果要用到变量，就使用$；如果要操作变量，则不使用$。这条规则的一个例外是使用printenv显示某个变量的值。**
- 和修改变量一样，在子 shell 中删除全局变量后， 无法将效果反映到父 shell 中。

8. 默认的 shell 环境变量
- bash shell 与 Unix Bourne shell 兼容的环境变量。bash shell 支持的 Bourne 变量
    变量 | 描述
    ---|---
    CDPATH      | 以冒号分隔的目录列表， 作为 cd 命令的搜索路径
    HOME        | 当前用户的主目录
    IFS         | shell 用来将文本字符串分割成字段的若干字符
    MAIL        | 当前用户收件箱的文件名（bash shell 会检查这个文件来确认有没有新邮件）
    MAILPATH    | 以冒号分隔的当前用户收件箱的文件名列表（bash shell 会检查列表中的每个文件来确认有没有新邮件） 
    OPTARG      | 由 getopt 命令处理的最后一个选项参数
    OPTIND      | 由 getopt 命令处理的最后一个选项参数的索引
    PATH        | shell 查找命令时使用的目录列表， 以冒号分隔
    PS1         | shell 命令行的主提示符
    PS2         | shell 命令行的次提示符

- 除了默认的 Bourne 环境变量， bash shell 还提供一些自有的变量。bash shell 环境变量
    变量                    | 描述
    ---                     |---
    BASH                    | bash shell 当前实例的完整路径名
    BASH_ALIASES            | 关联数组，包含当前已设置的别名
    BASH_ARGC               | 数组变量，包含传入函数或 shell 脚本的参数个数
    BASH_ARCV               | 数组变量，包含传入函数或 shell 脚本的参数
    BASH_ARCV0              | 包含 shell 的名称或 shell 脚本的名称（如果在脚本中使用的话）
    BASH_CMDS               | 关联数组，包含 shell 已执行过的命令的位置
    BASH_COMMAND            | 正在执行或将要执行的 shell 命令
    BASH_COMPAT             | 指定 shell 兼容级别的值
    BASH_ENV                | 如果设置的话， bash 脚本会在运行前先尝试运行该变量定义的启动文件
    BASH_EXECUTION_STRING   | 使用 bash 命令的-c 选项传递过来的命令
    BASH_LINENO             | 数组变量，包含当前正在执行的 shell 函数在源文件中的行号
    BASH_LOADABLE_PATH      | 以冒号分隔的目录列表， shell 会在其中查找可动态装载的内建命令
    BASH_REMATCH            | 只读数组变量， 在使用正则表达式的比较运算符=~进行肯定匹配（positive match）时， 包含整个模式及子模式所匹配到的内容
    BASH_SOURCE             | 数组变量，包含当前正在执行的 shell 函数所在的源文件名
    BASH_SUBSHELL           | 当前子 shell 环境的嵌套级别（初始值是 0 ）
    BASH_VERSINFO           | 数组变量，包含 bash shell 当前实例的主版本号和次版本号
    BASH_VERSION            | bash shell 当前实例的版本号
    BASH_XTRACEFD           | 如果设置为有效的文件描述符（ 0、1、2），则 'set -x'调试选项生成的跟踪输出可被 重定向。通常用于将跟踪信息输出到文件中
    BASHOPTS                | 当前启用的 bash shell选项
    BASHPID                 | 当前 bash进程的 PID
    CHILD_MAX               | 设置 shell 能够记住的已退出子进程状态的数量
    COLUMNS                 | bash shell 当前实例所用的终端显示宽度
    COMP_CWORD              | 变量 COMP_WORDS 的索引，其中包含当前光标的位置
    COMP_LINE               | 当前命令行
    COMP_POINT              | 相对于当前命令起始处的光标位置索引
    COMP_KEY                | 用来调用 shell 函数补全功能的最后一个按键
    COMP_TYPE               | 一个整数值，指明了用以完成 shell 函数补全所尝试的补全类型
    COMP_WORDBREAKS         | Readline 库中用于单词补全的分隔符
    COMP_WORDS              | 数组变量，包含当前命令行所有单词
    COMPREPLY               | 数组变量，包含由 shell 函数生成的可能的补全代码
    COPROC                  | 数组变量，包含用于匿名协程 I/O 的文件描述符
    DIRSTACK                | 数组变量，包含目录栈的当前内容
    EMACS                   | 设置为't'时，表明 emacs shell 缓冲区正在工作，行编辑功能被禁止
    EPOCHREALTIME           | 包含自 Unix 纪元时（ 1970 年 1 月 1 日 00:00:00 UTC ）以来的秒数，包括微秒   
    EPOCHSECONDS            | 包含自 Unix 纪元时（ 1970 年 1 月 1 日 00:00:00 UTC ）以来的秒数，不包括微秒
    ENV                     | 如果设置，则会在 bash shell 脚本运行之前先执行已定义的启动文件（仅当 bash shell 以 POSIX 模式被调用时）
    EUID                    | 当前用户的有效用户 ID （数字形式）
    EXECIGNORE              | 以冒号分隔的过滤器列表， 在使用 PATH 搜索命令时， 用于决定要忽略的可执行文件 （比如共享库文件）
    FCEDIT                  | 供 fc 命令使用的默认编辑器
    FIGNORE                 | 在进行文件名补全时可以忽略后缀名列表，以冒号分隔
    FUNCNAME                | 当前正在执行的 shell 函数的名称
    FUNCNEST                | 当设置成非 0 值时， 表示所允许的函数最大嵌套级数（一旦超出， 当前命令即被终止） 
    GLOBIGNORE              | 以冒号分隔的模式列表，定义了在进行文件名扩展时可以忽略的一组文件名
    GROUPS                  | 数组变量，包含当前用户的属组
    histchars               | 控制历史记录扩展，最多可有 3 个字符
    HISTCMD                 | 当前命令在历史记录中的编号
    HISTCONTROL             | 控制哪些命令留在历史记录列表中
    HISTFILE                | 保存 shell 历史记录的文件名（默认是.bash_history）
    HISTFILESIZE            | 历史记录文件（history file）能保存的最大命令数量
    HISTIGNORE              | 以冒号分隔的模式列表， 用于决定忽略历史文件中的哪些命令
    HISTSIZE                | 能写入历史记录列表（ history list）的最大命令数量①
    HISTTIMEFORMAT          | 如果设置且不为空，则作为格式化字符串，用于打印 bash 历史记录中命令的时间戳
    HOSTFILE                | shell 在补全主机名时读取的文件名
    HOSTNAME                | 当前主机的名称
    HOSTTYPE                | 字符串， 用于描述当前运行 bash shell 的机器
    IGNOREEOF               | shell 在退出前必须连续接收到的 EOF字符数量（如果该值不存在， 则默认为 1 ）
    INPUTRC                 | Readline 的初始化文件名（默认为.inputrc）
    INSIDE_EMACS            | 仅当进程在 Emacs 编辑器的缓冲区中运行时才设置， 并且可以禁用行编辑（行编辑的 禁用也取决于 TERM 变量的值）
    LANG                    | shell 的语言环境种类（locale category）
    LC_ALL                  | 定义语言环境种类，能够覆盖 LANG 变量
    LC_COLLATE              | 设置字符串排序时采用排序规则
    LC_CTYPE                | 决定如何解释出现在文件名扩展和模式匹配中的字符
    LC_MESSAGES             | 决定在解释前面带有$的双引号字符串时采用的语言环境设置
    LC_NUMERIC              | 决定格式化数字时采用的语言环境设置
    LC_TIME                 | 决定格式化日期和时间时采用的语言环境设置
    LINENO                  | 当前正在执行的脚本语句的行号
    LINES                   | 定义了终端上可见的行数
    MACHTYPE                | 用“CPU–公司–系统”（CPU-company-system）格式定义的系统类型
    MAILCHECK               | shell 应该多久检查一次新邮件（以秒为单位，默认为 60 秒）
    MAPFILE                 | 数组变量，当未指定数组变量作为参数时，其中保存了 mapfile 所读入的文本
    OLDPWD                  | shell 先前使用的工作目录
    OPTERR                  | 如果设置为 1 ，则 bash shell会显示 getopts 命令产生的错误
    OSTYPE                  | 定义了 shell 所在的操作系统
    PIPESTATUS              | 数组变量，包含前台进程的退出状态
    POSIXLY_CORRECT         | 如果设置的话， bash 会以 POSIX 模式启动
    PPID                    | bash shell父进程的 PID
    PROMPT_COMMAND          | 如果设置的话，在显示命令行主提示符之前执行该命令
    PROMPT_DIRTRIM          | 用来定义使用提示符字符串\w 和\W转义时显示的拖尾（trailing）目录名的数量（使用 一组英文句点替换被删除的目录名）
    PS0                     | 如果设置的话，指定了在输入命令之后、执行命令之前由交互式 shell 显示的内容
    PS3                     | select 命令的提示符
    PS4                     | 在命令行之前显示的提示符（如果使用了 bash的-x 选项的话）
    PWD                     | 当前工作目录
    RANDOM                  | 返回一个 0 ～ 32 767 的随机数（对该变量的赋值可作为随机数生成器的种子） 
    READLINE_LINE           | 当使用 bind –x 命令时， 保存 Readline 缓冲区的内容
    READLINE_POINT          | 当使用 bind –x 命令时， 指明了Readline 缓冲区内容插入点的当前位置 
    REPLY                   | read 命令的默认变量
    SECONDS                 | 自 shell 启动到现在的秒数（对其赋值会重置计数器）
    SHELL                   | bash shell 的完整路径名
    SHELLOPTS               | 以冒号分隔的已启用的 bash shell选项
    SHLVL                   | shell 的层级，每启动一个新的 bash shell，该值增加 1
    TIMEFORMAT              | 指定了 shell 的时间显示格式
    TMOUT                   |select 命令和 read 命令在无输入的情况下等待多久（以秒为单位，默认值为 0 ，表 示一直等待）
    TMPDIR                  | 目录名， 保存 bash shell 创建的临时文件
    UID                     | 当前用户的真实用户 ID （数字形式）

- HISTFILESIZE 和 HISTSIZE 这两个环境变量的区别。先要区分“历史记录列表”和“历史记  录文件”。前者位于内存中，在 bash 会话进行期间更新。后者位于硬盘上， 在 bash shell 中通常是~/.bash_history。 会话结束后，历史记录列表中的内容会被写入历史记录文件。如果 HISTFILESIZE=200，表示历史记录文件中最多能保存 200 个命令；如果 HISTSIZE=20，表示不管输入多少命令，历史记录列表中只记录 20 个命令，最终也只有这 20个命令会在会话结束后被写入历史记录文件。
- 不是所有的默认环境变量都会在 set 命令的输出中列出。如果用不到，默认环境变量并不要求必须有值。
- 系统使用的默认环境变量有时取决于 bash shell 的版本。例如， EPOCHREALTIME 仅在
bash shell 版本 5 及更高版本中可用。可以在 CLI 中输入 bash --version 来查看 bash shell 的版本号。

9. 设置 PATH 环境变量
- 当你在 shell CLI 中输入一个外部命令时，shell 必须搜索系统，从中找到对应的程序。PATH 环境变量定义了用于查找命令和程序的目录。
- PATH 中的目录之间以冒号分隔。
- 如果命令或者程序所在的位置没有包括在 PATH 变量中，那么在不使用绝对路径的情况下， shell 是无法找到的。 
- 应用程序的可执行文件目录有时不在 PATH 环境变量所包含的目录中。解决方法是保证 PATH 环境变量包含所有存放应用程序的目录。
- 有些脚本编写人员使用 env 命令作为 bash shell 脚本的第一行，就像这样： #!/usr/bin/env bash。这种方法的优点在于 env 会在$PATH 中搜索 bash，使脚本具备更好的可移植性。
- 可以把新的搜索目录添加到现有的 PATH 环境变量中，无须从头定义。 PATH 中各个目录 之间以冒号分隔。只需引用原来的 PATH 值， 添加冒号（:），然后再使用绝对路径输入新目录即可。
    ```
    $ PATH=$PATH:/home/christine/Scripts
    ```
    - 如果希望程序位置也可用于子 shell，则务必确保将修改后的 PATH 环境变量导出。
    - 对于 PATH 变量的修改只能持续到退出或重启系统。这种效果并不能一直奏效。
- 可以创建自己的全局环境变量和局部环境变量。这些变量在整个 shell 会话期间都是可用的。

10. 定位系统环境变量
- 当你登录 Linux 系统启动 bash shell 时， 默认情况下 bash 会在几个文件中查找命令。这些文件 称作启动文件或环境文件。bash 进程的启动文件取决于你启动 bash shell 的方式。启动bash shell 有以下 3 种方式：
    - 登录时作为默认登录 shell；
    - 作为交互式 shell，通过生成子 shell 启动；
    - 作为运行脚本的非交互式 shell。

11. 登录 shell
- 当你登录 Linux 系统时， bash shell 会作为登录 shell启动。登录 shell 通常会从 5 个不同的启动文件中读取命令。
    - /etc/profile
    - $HOME/.bash_profile
    - $HOME/.bashrc
    - $HOME/.bash_login
    - $HOME/.profile
- /etc/profile 文件是系统中默认的 bash shell 的主启动文件。系统中的每个用户登录时都会执行这个启动文件。
- 另外 4 个启动文件是针对用户的，位于用户主目录中，可根据个人具体需求定制。
- /etc/profile 文件是bash shell 默认的主启动文件。只要登录Linux 系统，bash 就会执行/etc/profile 启动文件中的命令。不同的 Linux 发行版在这个文件中放置了不同的命令。
- /etc/profile 文件使用 for 语句来迭代/etc/profile.d 目录下的所有文件。这为 Linux 系统提供了一个放置特定应用程序启动文件和/或管理员自定义启动文件的地方，shell 会在用户登录时执行这些文件。
- /etc/profile.d 目录下，大部分应用程序会创建两个启动文件： 一个供 bash shell 使用（扩展名为.sh）， 另一个供 C shell 使用（扩展名为.csh）。

12. $HOME 目录下的启动文件
- 其余的启动文件都用于同一个目的：提供用户专属的启动文件来定义该用户所用到的环境变量。大多数 Linux 发行版只用这 4 个启动文件中的一两个。
    - $HOME/.bash_profile
    - $HOME/.bashrc
    - $HOME/.bash_login
    - $HOME/.profile
- 注意，这些文件都以点号开头，说明属于隐藏文件（不会出现在一般的 ls 命令输出中）。 因为它们位于用户的$HOME 目录下，所以每个用户可以对其编辑并添加自己的环境变量， 其中的环境变量会在每次启动 bash shell 会话时生效。
- Linux 发行版在环境文件方面存在的差异非常大。本节所列出的$HOME 文件下的那些 文件并非每个用户都有。例如，有些用户可能只有一个$HOME/.bash_profile 文件。
- shell 会按照下列顺序执行第一个被找到的文件，余下的则被忽略。（这个列表中并没有$HOME/.bashrc 文件。这是因为该文件通常通过其他文件运行）
    - $HOME/.bash_profile
    - $HOME/.bash_login
    - $HOME/.profile
- $HOME 代表某个用户的主目录， 和波浪号（ ~ ）的效果一样。

13. 交互式 shell 进程
- 如果不是在登录系统时启动的 bash shell（比如在命令行中输入 bash），那么这时的 shell 称作交互式 shell。与登录 shell 一样，交互式 shell 提供了命令行提示符供用户输入命令。
- 作为交互式 shell 启动的 bash并不处理/etc/profile 文件，只检查用户$HOME 目录中的.bashrc 文件。
- CentOS Linux 系统中 .bashrc 文件会做两件事：首先，检查/etc 目录下的通用 bashrc 文件；其次， 为用户提供一个定制自己的命令别名和脚本函数的地方。

14. 非交互式 shell
- 系统执行 shell 脚本时用的就是这种 shell。不同之处在于它没有命令行提示符。但是，当你在系统中运行脚本时， 也许希望能够运行一些特定的启动命令。
- 为了处理这种情况， bash shell 提供了 BASH_ENV 环境变量。当 shell 启动一个非交互式 shell 进程时， 会检查这个环境变量以查看要执行的启动文件名。如果有指定的文件，则 shell 会执行 该文件里的命令，这通常包括 shell脚本变量设置。
- 脚本能以不同的方式执行。只有部分执行方式会启动子shell。
- 那如果未设置 BASH_ENV 变量， shell 脚本到哪里去获取其环境变量呢？别忘了有些 shell 脚 本是通过启动一个子 shell 来执行的。子 shell 会继承父 shell 的导出变量。
- 如果父 shell 是登录 shell，在/etc/profile 文件、 /etc/profile.d/*.sh 文件和$HOME/.bashrc 文件 中设置并导出了变量， 那么用于执行脚本的子 shell就能继承这些变量。
- 任何由父 shell 设置但未导出的变量都是局部变量，不会被子shell 继承。
- 对于那些不启动子 shell 的脚本， 变量已经存在于当前 shell 中了。就算没有设置 BASH_ENV， 也可以使用当前 shell 的局部变量和全局变量。

15. 环境变量持久化
- 可以利用环境文件文件创建自己的永久性全局变量或局部变量。
- 对全局环境变量（Linux 系统的所有用户都要用到的变量）来说，可能更倾向于将新的或修 改过的变量设置放在/etc/profile 文件中， 但这可不是什么好主意。如果升级了所用的发行版，则 该文件也会随之更新， 这样一来，所有定制过的变量设置可就都没有了。
- 最好在/etc/profile.d 目录中创建一个以.sh 结尾的文件。把所有新的或修改过的全局环境变量设置都放在这个文件中。
- 在大多数发行版中，保存个人用户永久性 bash shell 变量的最佳地点是$HOME/.bashrc 文件。 这适用于所有类型的 shell 进程 。但如果设置了 BASH_ENV 变量，请记住：除非值为 $HOME/.bashrc，否则， 应该将非交互式 shell 的用户变量放在别的地方。
- 图形化界面组成部分（比如 GUI 客户端）的环境变量可能需要在另外一些配置文件中设置，这和设置 bash shell 环境变量的文件不一样。
- alias 命令设置无法持久生效。你可以把个人的 alias 设置放在$HOME/.bashrc 启动文件中， 使其效果永久化。

16. 数组变量
- 环境变量的一个很酷的特性是可以作为数组使用。数组是能够存储多个值的变量。这些值既 可以单独引用，也可以作为整体引用。
- 要为某个环境变量设置多个值，可以把值放在圆括号中，值与值之间以空格分隔：
    ```
    $ mytest=(zero one two three four)
    $ echo $mytest
    zero
    $ echo ${mytest[2]}
    two
    $ echo ${mytest[*]}
    zero one two three four
    $ mytest[2]=seven
    $ echo ${mytest[2]}
    seven
    
    $ unset mytest[2]
    $ echo ${mytest[*]}
    zero one three four
    $ echo ${mytest[2]}
    
    $ echo ${mytest[3]}
    three
    
    $ unset mytest
    $ echo ${mytest[*]}
    
    ``` 
    - 要引用单个数组元素， 必须使用表示其在数组中位置的索引。索引要写在方括号中， $符号之后的所有内容都要放入花括号中。
    - 要显示整个数组变量， 可以用通配符*作为索引。
    - 也可以改变某个索引位置上的值。
    - 甚至能用 unset 命令来删除数组中的某个值， 但是要小心。
    - 上面例子用 unset 命令来删除索引 2 位置上的值。显示整个数组时，看起来好像其他索引 已经填补了这个位置。但如果专门显示索引 2 位置上的值时，你会发现这个位置是空的。
    - 可以在 unset 命令后跟上数组名来删除整个数组。
- 环境变量数组的索引都是从 0 开始的。
- 有时候， 数组变量只会把事情搞得更复杂，所以在 shell 脚本编程时并不常用。数组并不太方便移植到其他 shell 环境。
