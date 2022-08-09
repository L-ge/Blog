###### 十八、图形化桌面环境中的脚本编程

1. 创建交互式 shell 脚本最常用的方法是使用菜单。各种菜单项有助于用户了解脚本能做什么以及不能做什么。
- 菜单式脚本通常会清空显示区域，然后显示可用的菜单项列表。用户可以按下与每个菜单项关联的字母或数字来选择相应的选项。
- shell 脚本菜单的核心是 case 命令。case 命令会根据用户在菜单上的选择来执行相应的命令。

2. 创建菜单布局
- 创建菜单的第一步显然是确定在菜单上显示的元素以及想要显示的布局方式。
- 在创建菜单前， 最好先清除屏幕上已有的内容。
- clear 命令使用终端会话的终端设置信息来清除屏幕上的文本。运行 clear 命令之后，可以用 echo 命令来显示菜单。clear 命令会查看由环境变量 TERM 给出的终端类型，然后在 terminfo 数据库中确定如何清除屏幕上的内容，参见 man clear。
- 在默认情况下， echo 命令只显示可打印文本字符。在创建菜单项时，非可打印字符（比如 制表符和换行符）往往也能派上用场。要在 echo 命令中包含这些字符，必须加入-e 选项。因此，下列命令：
    ```
    echo -e "1.\tDisplay disk space"
    ```
    - 会生成如下输出行：
        ```
        1.        Display disk space
        ```
- -en 选项会去掉结尾的换行符。
    ```
    echo -en "\t\tEnter option: "
    ```
- 创建菜单的最后一步是获取用户输入。这要用到 read 命令。因为我们期望的是单字符输入，所以会在 read 命令中使用-n 选项来限制只读取一个字符。这样用户只需要输入一个数字，而不用按 Enter 键：
    ```
    read -n 1 option
    ```

3. 创建菜单函数
- 创建 shell 菜单脚本的第一步是确定脚本要实现的功能，然后将这些功能以函数的形式放在代码中。
- 通常我们会为还没有实现的函数创建一个桩函数（stub function）。桩函数既可以是一个空函 数，也可以只包含一个 echo 语句，用于说明最终这里需要什么内容：
    ```
    function diskspace {
        clear
        echo "This is where the diskspace commands will go"
    }
    ```
    - 这个桩函数允许在实现某个函数的同时，菜单仍能正常操作。无须等到写出所有函数之后才让菜单投入使用。
    - 函数从 clear 命令开始。这是为了能在一个干净的屏幕上执行函数，不让它受到原先菜单的干扰。
- 将菜单布局本身作为一个函数来创建，有助于制作 shell 脚本菜单：
    ```
    function menu {
        clear
        echo
        echo -e "\t\t\tSys Admin Menu\n"
        echo -e "\t1. Display disk space"
        echo -e "\t2. Display logged on users"
        echo -e "\t3. Display memory usage"
        echo -e "\t0. Exit program\n\n"
        echo -en "\t\tEnter option: "
        read -n 1 option
    }
    ```
    - 任何时候你都能调用 menu 函数来重现菜单。

4. 添加菜单逻辑
- case 命令应该根据用户输入的字符来调用相应的函数。使用默认的case 命令字符（星号）来处理所有不正确的菜单项是一种不错的做法。
- 下面的代码展示了菜单中 case 命令的典型用法：
    ```
    menu
    case $option in
    0)
        break ;;
    1)
        diskspace ;;
    2)
        whoseon ;;
    3)
        memusage ;;
    *)
        clear
        echo "Sorry, wrong selection";;
    esac
    ```
    - 这段代码首先用 menu 函数清除屏幕并显示菜单。 menu 函数中的 read 命令会一直等待，直到用户在键盘上输入了字符。然后， case 命令就会接管余下的处理过程。 case 命令会基于用户输入的字符调用相应的函数。函数运行结束后， case 命令退出。

5. 整合 shell 脚本菜单
- 一个完整的菜单脚本示例：
    ```
    $ cat menu1
    #!/bin/bash
    # simple script menu
    
    function diskspace {
        clear
        df -k
    }
    
    function whoseon {
        clear
        who
    }
    
    function memusage {
        clear
        cat /proc/meminfo
    }
    
    function menu {
        clear
        echo
        echo -e "\t\t\tSys Admin Menu\n"
        echo -e "\t1. Display disk space"
        echo -e "\t2. Display logged on users"
        echo -e "\t3. Display memory usage"
        echo -e "\t0. Exit program\n\n"
        echo -en "\t\tEnter option: "
        read -n 1 option
    }
    
    while [ 1 ]
    do
        menu
        case $option in
        0)
            break ;;
        1)
            diskspace ;;
        2)
            whoseon ;;
        3)
            memusage ;;
        *)
            clear
            echo "Sorry, wrong selection";;
        esac
        echo -en "\n\n\t\t\tHit any key to continue"
        read -n 1 line
    done
    clear
    $
    ```
    - 这个菜单创建了 3 个函数，以使用常见命令提取 Linux 系统的管理信息。 while 循环用于持续处理菜单，除非用户选择了选项 0，即通过 break 命令跳出while 循环。
    - 可以用这个模板创建任何 shell 脚本的菜单界面。它提供了一种与用户交互的简单途径。

6. 使用 select 命令
- select 命令只需要一个命令就可以创建出菜单， 然后获取输入并自动处理。 select 命令的格式如下：
    ```
    select variable in list
    do
        commands
    done
    ```
    - list 参数是由空格分隔的菜单项列表， 该列表构成了整个菜单。 select 命令会将每个列 表项显示成一个带编号的菜单项，然后显示一个由 PS3 环境变量定义的特殊提示符，指示用户做出选择。
- 一个 select 命令的简单示例：
    ```
    PS3="Enter option: "
    select option in "Display disk space" "Display logged on users" "Display memory usage" "Exit program"
    do
    case $option in
        "Exit program")
            break ;;
        "Display disk space")
            diskspace ;;
        "Display logged on users")
            whoseon ;;
        "Display memory usage")
            memusage ;;
        *)
            clear
            echo "Sorry, wrong selection";;
    esac
    done
    clear
    ```
    - select 语句中的所有内容必须作为一行出现。
    - 运行该脚本会自动生成如下菜单：
        ```
        $ ./smenu1
        1) Display disk space       3) Display memory usage
        2) Display logged on users  4) Exit program
        Enter option:
        ```
- 在使用 select 命令时， 存储在指定变量中的值是整个字符串，而不是跟菜单选项相 关联的数字。字符串才是要在 case 语句中进行比较的内容。

7. 创建文本窗口部件
- dialog 软件包最早是 Savio Lam编写的一款小巧的工具，现在由 Thomas E. Dickey 负责维护。 dialog 能够用 ANSI 转义控制字符，在文本环境中创建标准的窗口对话框。你可以轻而易举地将这些对话框融入自己的 shell 脚本中，以实现与用户的交互。
- 并不是所有的Linux 发行版中都会默认安装dialog 软件包。
    - 在 Ubuntu Linux 发行版中，使用下列命令安装该软件包：
        ```
        sudo apt-get install dialog
        ```
    - 基于 Red Hat 的发行版（比如 CentOS ）系统，使用 dnf 命令安装该软件包：
        ```
        sudo dnf install dialog
        ```
    - 软件包管理器会为你安装 dialog 软件包以及所需的库。

8. dialog 软件包

略。

9. dialog 选项

略。

10. 在脚本中使用 dialog 命令

略。

11. 图形化窗口部件

略。

12. KDE 环境

略。

13. GNOME 环境

略。

14. 实战演练

略。
