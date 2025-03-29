###### 二十四、编写简单的脚本实用工具

1. 日常备份
- 在 Linux 世界中，备份数据的工作是由 tar 命令完成的。tar 命令可以将整 个目录归档到单个文件中。
- 使用 tar 命令来创建工作目录归档文件的例子：
    ```
    $ tar -cf archive.tar /home/christine/Project/*.*
    tar: Removing leading `/' from member names
    tar: Removing leading `/' from hard link targets
    $
    $ ls -og archive.tar
    -rw-rw-r-- 1 112640 Aug 6 13:33 archive.tar
    
    $ tar -cf archive.tar Project/*.* 2>/dev/null
    $
    
    $ tar -zcf archive.tgz Project/*.* 2>/dev/null
    $
    $ ls -hog archive.tgz
    -rw-rw-r-- 1 11K Aug 6 13:40 archive.tgz
    ```
    - 注意， tar 命令会显示一条警告消息，指出它删除了路径开头的斜线。这意味着将路径从绝对路径改为了相对路径， 以便将 tar 归档文件提取到文件系统中的任何位置。如果不想在脚本中输出这条消息，则可以将 STDERR 重定向到/dev/null 文件。
    - 由于 tar 归档文件会占用大量的磁盘空间，因此最好压缩一下。这只需加一个-z 选项即可。该选项会使用 gzip压缩 tar 归档文件，由此生成的文件称作tarball。请务必使用恰当的文件扩展名来表示这种文件是一个 tarball，采用.tar.gz 或.tgz 都行。
    - 经过压缩， archive.tgz 比 archive.tar 小了约 99KB。
- 使用 exec 命令来重定向标准输入（STDIN）。实现方法如下：
    ```
    exec 0 < $config_file
    read file_name
    ```
    - 注意，我们对归档配置文件使用了一个变量 config_file。配置文件中每一条记录都会被读入。
    - 只要 read 命令在配置文件中发现还有记录可读，就会在?变量中返回一个表示成功的退出状态码 0。这可以作为 while 循环的测试条件来读取配置文件中的所有记录。
        ```
        while [ $? -eq 0 ]
        do
         [...]
        read file_name
        done
        ```
        - 一旦 read 命令读到配置文件的末尾，就会在?变量中返回一个非 0 状态码。这时，脚本会退出 while 循环。
- 记住， tar 只是使用 bash shell 命令在系统中执行备份的一种方法。有一些其他的实用程序（或命令组合）也许能更好地满足你的需求，比如 rsync。要查看可能有助于备份工作的各种实用工具名称，可以在命令行提示符下输入 man -k archive 和 man -k copy。

2. 创建按小时归档的脚本
- 如果你当天运行 Hourly_Archive.sh 脚本，那么当小时数是单个数字时， 归档文件名中只会出现 3 位数字。如果运行脚本的时间是 1:15 am ，那么归档文件名就是 archive115.tar.gz。如果希望文件名中总是保留 4 位数字， 则可以将脚本中的 TIME=$(date +%k%M)修改成 TIME=$(date +%k0%M)。在%k 后加入数字 0 后，所有的单位（single-digit）小时数都会加上一个前导数字 0，被填充成两位数字 。因此 ，archive115.tar.gz 就变成了 archive0115.tar.gz。

3. 删除账户
- cut 命令的-c1 选项可以删除 answer 中除第一个字符之外的所有内容。
- xargs 命令可以使用从标准输入 STDIN 获取的命令参数并执行指定的命令。它非常适合放在管道的末尾处。
- xargs 的现代版本不要求使用命令（比如 sudo 和 kill）的绝对路径。

4. 系统监控
- 获得默认的 shell 审计功能
    - 系统账户用于提供服务或执行特殊任务。一般来说，这类账户需要在/etc/passwd 文件中有对应的记录，但禁止登录系统（root 账户是一个典型的例外）。防止有人使用这些账户登录的方法是，将其默认 shell 设置为/bin/false 、/usr/sbin/nologin 或 /sbin/nologin。
    - 使用 cut 命令获取/etc/passwd 文件中所有账户的默认 shell。该命令可以指定文件的字段分隔符以及要提取的记录字段。对于/etc/passwd 文件，分隔符是冒号（:），账户的默认 shell 位于记录的第 7 个字段：
        ```
        $ cut -d: -f7 /etc/passwd
        /bin/bash
        /usr/sbin/nologin
        /usr/sbin/nologin
        /usr/sbin/nologin
         [...]
        /bin/false
        /bin/bash
        /usr/sbin/nologin
        /bin/bash
        /usr/sbin/nologin
        /bin/bash
        $
        ```
- find 命令的-perm（permissions） 选项可以使用八进制值指定要查找的具体权限。
- diff 命令只会逐行对文件进行比较。因此， diff 会比较两份报告的第一行，然后是第二行、第三行，以此类推。如果由于要安装软件，添加了一个或一批新文件，而这些文件需要 SUID 权限或 SGID 权限，那么在下一次审计时， diff 就会显示大量的差异。为了解决这个潜在的问题，可以在 diff 命令中使用-q 选项或--brief 选项，只显示消息，说明这两份报告存在不同。
