## 前言：
**《Linux高并发服务器开发》《Linux高性能服务器编程》**

## 一、Linux系统编程入门
1.安装命令 sudo apt install gcc g++

查看版本 gcc/g++ -v/--version

2.GCC常用参数选项

  gcc编译选项 | 说明  
  ----   | ----  
 -E  | 预处理指定的源文件，不进行编译
 -S  | 编译指定的源文件，但是不进行汇编
 -c | 编译、汇编指定的源文件，但是不进行链接
 -o [file1] [file2] / [file2] -o [file1]  | 将文件 file2 编译成可执行文件 file1 
-I directory | 指定 include 包含文件的搜索目录
-g | 在编译的时候，生成调试信息，该程序可以被调试器调试
-D | 在程序编译的时候，指定一个宏
-w | 不生成任何警告信息
-Wall | 生成所有警告信息
-On | n的取值范围：0~3。编译器的优化选项的4个级别，-O0表示没有优化，-O1为缺省值，-O3优化级别最高
-l | 在程序编译的时候，指定使用的库
-L | 指定编译的时候，搜索的库的路径
-fPIC/fpic | 生成与位置无关的代码
-shared | 生成共享目标文件，通常用在建立共享库时
-std  | 指定C方言，如:-std=c99，gcc默认的方言是GNU C

3.在编译阶段，g++ 会调用 gcc，对于 C++ 代码，g++ 和 gcc 是等价的，但是因为 gcc 命令不能自动和 C++ 程序使用的库链接，所以通常用 g++ 来完成链接。

编译可以用 gcc/g++，而链接可以用 g++ 或者 gcc -lstdc++。

编译可执行文件test01：g++ test01.cpp -o test01

4.静态库在程序的链接阶段被复制到程序中；动态库在链接阶段没有被复制到程序中，而是程序在运行时由系统动态库加载到内存中供程序调用。

5.静态库的制作：
* gcc 获得.o文件
* 将.o文件打包，使用ar工具(archive)  
  ar rcs libxxx.a file1.o file2.o file3.0   
  r - 将文件插入备存文件中  
  c - 建立备存文件  
  s - 索引  

gcc -c file1.c file2.c file3.c  
ar rcs libxxx.a file1.o file2.o file3.o

6.查看文件目录树的命令是tree

sudo apt install tree

7.静态库的使用：

要头文件(放在include目录下)和libcalc.a文件(放在lib目录下)。  
gcc main.c -o app -I ./include -l calc -L ./lib  
注：calc是库的名字，libcalc.a是库文件的名字。

8.动态库的制作：
* gcc 获得.o文件。用-fpic得到和位置无关的代码，在x86架构-fpic和-fPIC没有区别。   
  gcc -c -fpic/-fPIC file1.c file2.c file3.c   
* gcc 得到动态库。  
  gcc -shared file1.o file2.o file3.o -o libxxx.so
  
在Linux下，动态库是一个可执行文件。

9.动态库的使用：

gcc main.c -o app -I ./include -L ./lib -l calc

动态库可以实现进程间资源共享。两个进程加载同一个动态库，共享一份内存空间，但共享的是代码段，并不是数据段。

* 静态库：GCC 进行链接时，会把静态库中代码打包到可执行程序中。
* 动态库：GCC 进行链接时，动态库的代码不会被打包到可执行程序中。
* 程序启动之后，动态库会被动态加载到内存中，通过 ldd （list dynamic dependencies）命令检查动态库依赖关系。
  ldd main  
* 如何定位共享库文件呢？  
    当系统加载可执行代码时候，能够知道其所依赖的库的名字，但是还需要知道绝对路径。此时就需要系统的动态载入器来获取该绝对路径。对于elf格式的可执行程序，是由ld-linux.so来完成的，它先后搜索elf文件的 
    DT_RPATH段 ——> 环境变量LD_LIBRARY_PATH ——> /etc/ld.so.cache文件列表 ——> /lib/，/usr/lib
    目录找到库文件后将其载入内存。

方式一：配置环境变量(临时的)：  
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/home/bread/testcode/lib  
echo $LD_LIBRARY_PATH

方式二：配置环境变量(永久的，用户级别)：  
vim .bashrc  
最后一行加上export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/home/bread/testcode/lib  
保存退出后 source .bashrc  

方式三：配置环境变量(系统级别)：  
sudo vim /etc/profile  
最后一行加上export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/home/bread/testcode/lib  
保存退出后 source /etc/profile  

方式四：间接配置/etc/ld.so.cache文件列表的方式：  
sudo vim /etc/ld.so.conf  
最后一行加上/home/bread/testcode/lib  
保存退出后 sudo ldconfig  

10.Makefile文件命名和规则：  
* 文件命名：makefile 或者 Makefile
* Makefile规则：
  - 一个 Makefile 文件中可以有一个或者多个规则  
    目标 ... : 依赖 ...    
        命令（Shell 命令）  
        ...  
 其中，  
    目标：最终要生成的文件（伪目标除外）  
    依赖：生成目标所需要的文件或是目标  
    命令：通过执行命令对依赖操作生成目标（命令前必须 Tab 缩进）  
  - Makefile 中的其它规则一般都是为第一条规则服务的。

例子：  
vim Makefile  
```
app:file1.c file2.c file3.c  
    gcc file1.c file2.c file3.c -o app  
```
sudo apt install make  
make  

11.Makefile工作原理：
* 命令在执行之前，需要先检查规则中的依赖是否存在。如果存在，执行命令；如果不存在，向下检查其它的规则，检查有没有一个规则是用来生成这个依赖的，如果找到了，则执行该规则中的命令。
* 检测更新，在执行规则中的命令时，会比较目标和依赖文件的时间。如果依赖的时间比目标的时间晚，需要重新生成目标；如果依赖的时间比目标的时间早，目标不需要更新，对应规则中的命令不需要被执行。
```
app:file1.o file2.o file3.o  
    gcc file1.o file2.o file3.o -o app  
    
file1.o:file1.c
    gcc -c file1.c -o file1.o
    
file2.o:file2.c
    gcc -c file2.c -o file2.o
    
file3.o:file3.c
    gcc -c file3.c -o file3.o
```

12.Makefile中的变量： 
* 自定义变量  
    变量名=变量值 var=hello
* 预定义变量  
    AR : 归档维护程序的名称，默认值为 ar  
    CC : C 编译器的名称，默认值为 cc  
    CXX : C++ 编译器的名称，默认值为 g++  
    $@ : 目标的完整名称  
    $< : 第一个依赖文件的名称  
    $^ : 所有的依赖文件  
* 获取变量的值  
    $(变量名)

模式匹配：
* %.o:%.c
    - %：通配符，匹配一个字符串
    - 两个%匹配的是同一个字符串
* %.o:%.c  
    gcc -c $< -o $@

```
app:file1.c file2.c file3.c
    gcc -c file1.c file2.c file3.c

# 自动变量只能在规则的命令中使用
app:file1.c file2.c file3.c
    $(CC) -c $^ -o $@
```    
```
#定义变量
src=file1.o file2.o file3.o  
target=app
$(target):$(src)
    $(CC) $(src) -o $(target) 

%.o:%.c  
    $(CC) -c $< -o $@
```

13.Makefile中的函数：
* $(wildcard PATTERN...)
    - 功能：获取指定目录下指定类型的文件列表
    - 参数：PATTERN 指的是某个或多个目录下的对应的某种类型的文件，如果有多个目录，一般使用空格间隔
    - 返回：得到的若干个文件的文件列表，文件名之间使用空格间隔
    - 示例：  
    $(wildcard *.c ./sub/ *.c)  
    返回值格式: a.c b.c c.c d.c e.c f.c
* $(patsubst <pattern>,<replacement>,<text>)
    - 功能：查找<text>中的单词(单词以“空格”、“Tab”或“回车”“换行”分隔)是否符合模式<pattern>，如果匹配的话，则以<replacement>替换。
    - <pattern>可以包括通配符`%`，表示任意长度的字串。如果<replacement>中也包含`%`，那么，<replacement>中的这个`%`将是<pattern>中的那个%所代表的字串。(可以用`\`来转义，以`\%`来表示真实含义的`%`字符)
    - 返回：函数返回被替换过后的字符串
    - 示例：  
        $(patsubst %.c, %.o, x.c bar.c)  
        返回值格式: x.o bar.o
```
#定义变量
#获取file1.c file2.c file3.c
src=$(wildcard ./*.c)
objs=$(patsubst %.c, %.o, $(src))  
target=app
$(target):$(objs)
    $(CC) $(objs) -o $(target) 

%.o:%.c  
    $(CC) -c $< -o $@

.PHONY:clean    
clean:
    rm $(objs) -f
```
删除.o文件：make clean

14.GDB调试：
* 通常，在为调试而编译时，我们会关掉编译器的优化选项（`-O`）， 并打开调试选项（`-g`）。另外，`-Wall`在尽量不影响程序行为的情况下选项打开所有warning，也可以发现许多问题，避免一些不必要的BUG。
* gcc -g -Wall program.c -o program
* `-g` 选项的作用是在可执行文件中加入源代码的信息，比如可执行文件中第几条机器指令对应源代码的第几行，但并不是把整个源文件嵌入到可执行文件中，所以在调试时才能保证 gdb 能找到源文件。

15.GDB命令 – 启动、退出、查看代码：
* 启动和退出
    - gdb 可执行程序
    - quit
* 给程序设置参数/获取设置参数
    - set args 10 20
    - show args
* GDB 使用帮助
     - help
* 查看当前文件代码
    - list/l （从默认位置显示）
    - list/l 行号 （从指定的行显示）
    - list/l 函数名（从指定的函数显示）
* 查看非当前文件代码
    - list/l 文件名:行号
    - list/l 文件名:函数名
* 设置显示的行数
    - show list/listsize
    - set list/listsize 行数
```
gcc test.c -o test -g
ll -h test      // test加了编译选项-g后,编译的可执行文件变大
gdb test
(gdb) set args 10 20
(gdb) show args
(gdb) list      // 输入list(或者l)默认显示源文件test.c的前10行
(gdb) list      // 输入list(或者l或者直接回车)继续往下显示源文件test.c
(gdb) help
(gdb) help all
(gdb) help set
(gdb) quit
```

16.GDB 命令 – 断点操作
* 设置断点
    - b/break 行号
    - b/break 函数名
    - b/break 文件名:行号
    - b/break 文件名:函数
* 查看断点
    - i/info b/break
* 删除断点
    - d/del/delete 断点编号
* 设置断点无效
    - dis/disable 断点编号
* 设置断点生效
    - ena/enable 断点编号
* 设置条件断点（一般用在循环的位置）
    - b/break 10 if i==5
```
g++ main.cpp -o main -g
gdb main
(gdb) break 9           // 在第9行打断点,但是第9行还没有执行的
(gdb) info break        // 查看断点信息 
(gdb) break main        // 在main函数所在行打断点
(gdb) delete 2          // 删除编号为2的断点
(gdb) break 10 if i==5  // 在第10行当i==5的时候断点
```

17.GDB 命令 – 调试命令
* 运行GDB程序
    - start（程序停在第一行）
    - run（遇到断点才停）
* 继续运行，到下一个断点停
    - c/continue
* 向下执行一行代码（不会进入函数体）
    - n/next
* 变量操作
    - p/print 变量名（打印变量值）
    - ptype 变量名（打印变量类型）
* 向下单步调试（遇到函数进入函数体）
    - s/step
    - finish（跳出函数体）
* 自动变量操作
    - display 变量名（自动打印指定变量的值）
    - i/info display
    - undisplay 编号
* 其它操作
    - set var 变量名=变量值 （循环中用的较多）
    - until （跳出循环）
```
g++ main.cpp -o main -g
gdb main
(gdb) start             // 相当于默认在第一行打了断点,start之后直接停在了第一行
(gdb) c                 // 第一行后面都没有设置断点,因此c之后直接运行完了程序
(gdb) break 9           // 在第9行打断点
(gdb) run               // 停在第9行
(gdb) print i           // 打印第9行变量i的值
(gdb) ptype i           // 打印第9行变量i的类型
(gdb) set var i=1       // 设置变量i的值等于1
```

18.文件IO

Linux系统IO函数是没有缓存区的，不同于标准C库的IO函数。  
在终端输入 man 2 open 查看Linux库的帮助文档。    
在终端输入 man 3 fopen 查看标准C库的帮助文档。  

* Linux 系统 IO 函数
    - int open(const char *pathname, int flags);  
        打开一个已经存在的文件.   
        flags是打开文件后对文件的操作权限.  

    - int open(const char *pathname, int flags, mode_t mode);
        创建一个新的文件.  
        mode是文件的权限属性，但最终权限是 mode&~umask .  
        可以在终端直接输入umask看当前用户的umask的值。  
        umask的作用就是抹去某些权限。  
        int fd = open("a.txt", O_RDWR | O_CREAT, 0777);  
        普通用户的话，得到a.txt的权限一般是0775，因为umask一般是0002。  
        
    - int close(int fd);
        
    - ssize_t read(int fd, void *buf, size_t count);
    
    - ssize_t write(int fd, const void *buf, size_t count);
    
    - off_t lseek(int fd, off_t offset, int whence);
        对文件指针进行操作。  
        offset：偏移量  
        whence：SEEK_SET-设置文件指针的偏移量、SEEK_CUR-当前位置+offset的偏移量、SEEK_END-文件大小+offset的偏移量。  
        作用：  
            移动文件指针到文件头部 lseek(fd, 0, SEEK_SET);  
            获取当前文件指针的位置 off_t curIndex = lseek(fd, 0, SEEK_CUR);  
            获取文件长度 off_t len = lseek(fd, 0, SEEK_END);  
            拓展文件的长度 lseek(fd, nLen, SEEK_END); 最后还要写入数据，文件的长度才会变长。  
    
    - int stat(const char *pathname, struct stat *statbuf);  
        获取文件的一些属性信息。  
        也可以直接在终端输入 stat a.txt 查看文件的信息。  
        
    - int lstat(const char *pathname, struct stat *statbuf);  
        获取软链接文件的一些属性信息。  
        直接在终端输入创建软连接 ln -s a.txt b.txt  
        如果用stat b.txt获取到的是a.txt的文件信息。  
    
* 文件权限
    - 第15位-第12位：4位-表示文件类型
    - 第11位-第9位：3位-表示特殊权限位
    - 第8位-第6位：3位-分别表示rwx,当前用户.
    - 第5位-第3位：3位-分别表示rwx,当前用户所在的组.
    - 第2位-第0位：3位-分别表示rwx,其他组.
* 文件属性操作函数
    - int access(const char *pathname, int mode);  
        作用：判断某个文件是否有某个权限或者判断文件是否存在。  
        mode：R_OK判断是否有读权限、W_OK判断是否有写权限、X_OK判断是否有执行权限、F_OK判断文件是否存在。  
            
    - int chmod(const char *filename, int mode);  
        作用：修改文件的权限。  

    - int chown(const char *path, uid_t owner, gid_t group);  
        作用：修改文件的所在组或所有者。  
        查看用户id：vim /etc/passwd  
        查看组id：vim /etc/group  
        或者查看id用：id xiaoming  
        
    - int truncate(const char *path, off_t length);  
        作用：对文件尺寸缩减或者扩展至指定大小。  
        length：需要最终文件变成的大小。  
    
* 目录操作函数  
    - 一般来说，Web服务器的逻辑根目录并非文件系统的根目录“/”，而是站点的根目录（对于Linux的Web服务来说，该目录一般是/var/www/）.
    
    - int rename(const char *oldpath, const char *newpath);
        
    - int chdir(const char *path);  
        作用：修改进程的工作目录。
        
    - char *getcwd(char *buf, size_t size);  
        作用：获取进程当前的工作目录。  
        返回值：返回的其实就是第一个参数。  
        
    - int mkdir(const char *pathname, mode_t mode);  
        作用：创建一个目录。  

    - int rmdir(const char *pathname);  
        作用：删除一个空目录。  
    
    - int chroot(const char* path);  
        path参数指定要切换到的目标根目录。该函数并不改变进程的当前工作目录。此外，只有特权进程才能改变根目录。
        
* 目录遍历函数
    - DIR *opendir(const char *name);  
        作用：打开一个目录，返回目录流。

    - struct dirent *readdir(DIR *dirp);  
        作用：读取目录中的数据，返回读取到的文件的信息。如果读取到了末尾或者失败了，返回NULL。
        
    - int closedir(DIR *dirp);
        
* dup和dup2函数
    - int dup(int oldfd);   
        复制得到一个新的文件描述符(从空闲的文件描述符表中找一个最小的)，与oldfd指向同一个文件。
        
    - int dup2(int oldfd, int newfd);  
        重定向文件描述符。调用函数成功后，newfd close掉原来它所指向的文件，然后指向和oldfd所指向的文件。    
        返回值和newfd的值相同。

    - **注意：通过dup和dup2创建的文件描述符并不继承原文件描述符的属性，比如close-on-exec和non-blocking等。**
    
* readv函数和writev函数
    - readv函数将数据从文件描述符读到分散的内存块中，即分散读；writev函数则将多块分散的内存数据一并写入文件描述符中，即集中写。
    - #include <sys/uio.h>
    - ssize_t readv(int fd, const struct iovec* vector, int count);
    - ssize_t writev(int fd, const struct iovec* vector, int count);

* sendfile函数
    - sendfile函数在两个文件描述符之间直接传递数据(完全在内核中操作)从而避免了内核缓冲区和用户缓冲区之间数据拷贝，效率很高，这被称为零拷贝。
    - #include <sys/sendfile.h>
    - ssize_t sendfile(int out_fd, int in_fd, off_t* offset, size_t count);  
        out_fd是待写入内容的文件描述符；  
        in_fd是待读出内容的文件描述符；  
        offset指定从读入文件流的哪个位置开始读，如果为空，则使用读入文件流默认的起始位置。  
        count指定在文件描述符in_fd和out_fd之间传输的字节数。  
        **in_fd必须是一个支持类似mmap函数的文件描述符，即它必须指向真实的文件，不能是socket和管道；而out_fd则必须是一个socket。**  

* splice函数  
    - splice函数用于在两个文件描述符之间移动数据，也是零拷贝操作。  
    - #include <fcntl.h>
    - ssize_t splice(int fd_in, loff_t* off_in, int fd_out, loff_t* off_out, size_t len, unsigned int flags);  
        fd_in是待输入的文件描述符。如果fd_in是一个管道文件描述符，那么off_in参数必须被设置为NULL。
    - 使用splice函数时，fd_in和fd_out必须至少有一个是管道文件描述符。

* tee函数
    - tee函数在两个管道文件描述符之间复制数据，也是零拷贝操作。它不消耗数据(和splice函数不同，它是复制)，因此源文件描述符上的数据仍然可以用于后续的读操作。
    - #include <fcntl.h>
    - ssize_t tee(int fd_in, int fd_out, size_t len, unsigned int flags);
    - fd_in和fd_out必须都是管道文件描述符。

* fcntl 函数
    - int fcntl(int fd, int cmd, ... /* arg */ );  
        作用是：复制文件描述符；设置/获取文件的状态标志等等。  
        cmd表示对文件描述符进行如何操作。  
        F_DUPFD是复制文件描述符，新文件描述符通过返回值返回  
        F_GETFL是获取指定文件描述符的文件状态，也就是open函数设置的那个flag。  
        F_SETFL是设置指定文件描述符的文件状态，但不能设置文件权限属性。常用的状态标志如：O_APPEND表示追加数据。O_NONBLOCK表示设置为不阻塞。  
    - 在网络编程中，fcntl函数通常用来将一个文件描述符设置为非阻塞的。
    - SIGIO和SIGURG这两个信号与其他Linux信号不同，它们必须与某个文件描述符相关联方可使用：当被关联的文件描述符可读或可写时，系统将触发SIGIO信号；当被关联的文件描述符(而且必须是一个socket)上有带外数据可读时，系统将触发SIGURG信号。将信号和文件描述符关联的方法，就是使用fcntl函数为目标文件描述符指定宿主进程或进程组，那么被指定的宿主进程或进程组将捕获这两个信号。
        
* 其他
    - errno：属于Linux系统函数库，库里面的一个全局变量，记录的是最近的错误号。
    - perror：属于标准C库，打印errno对应的错误描述。
        ```
        int fd = open("a.txt", O_RDONLY);
        if(-1 == fd)
            perror("Error");   //如果没有这个文件,会输出Error:No such file or directory
        ```
    - 传递给main函数的第一个参数argv[0]是可执行程序的名称。
        例如 C:\Desktop\build-untitled-Desktop_Qt_5_7_0_MinGW_32bit-Debug\debug\untitled.exe
    - 阻塞和非阻塞：描述的是函数调用的行为。

19.模拟实现ls -l a.txt命令
```
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <unistd.h>
#include <pwd.h>
#include <grp.h>
#include <time.h>
#include <string.h>

int main(int argc, char * argv[]) 
{
    if(argc < 2)    // 判断输入的参数是否正确
    {
        printf("%s filename\n", argv[0]);
        return -1;
    }

    // 通过stat函数获取用户传入的文件的信息
    struct stat st;
    int ret = stat(argv[1], &st);
    if(ret == -1) 
    {
        perror("stat");
        return -1;
    }

    // 获取文件类型和文件权限
    char perms[11] = {0};   // 用于保存文件类型和文件权限的字符串

    switch(st.st_mode & S_IFMT) {
        case S_IFLNK:
            perms[0] = 'l';
            break;
        case S_IFDIR:
            perms[0] = 'd';
            break;
        case S_IFREG:
            perms[0] = '-';
            break; 
        case S_IFBLK:
            perms[0] = 'b';
            break; 
        case S_IFCHR:
            perms[0] = 'c';
            break; 
        case S_IFSOCK:
            perms[0] = 's';
            break;
        case S_IFIFO:
            perms[0] = 'p';
            break;
        default:
            perms[0] = '?';
            break;
    }

    // 判断文件的访问权限

    // 文件所有者
    perms[1] = (st.st_mode & S_IRUSR) ? 'r' : '-';
    perms[2] = (st.st_mode & S_IWUSR) ? 'w' : '-';
    perms[3] = (st.st_mode & S_IXUSR) ? 'x' : '-';

    // 文件所在组
    perms[4] = (st.st_mode & S_IRGRP) ? 'r' : '-';
    perms[5] = (st.st_mode & S_IWGRP) ? 'w' : '-';
    perms[6] = (st.st_mode & S_IXGRP) ? 'x' : '-';

    // 其他人
    perms[7] = (st.st_mode & S_IROTH) ? 'r' : '-';
    perms[8] = (st.st_mode & S_IWOTH) ? 'w' : '-';
    perms[9] = (st.st_mode & S_IXOTH) ? 'x' : '-';

    int linkNum = st.st_nlink;  // 硬连接数

    char * fileUser = getpwuid(st.st_uid)->pw_name; // 文件所有者

    char * fileGrp = getgrgid(st.st_gid)->gr_name;  // 文件所在组

    long int fileSize = st.st_size; // 文件大小

    char * time = ctime(&st.st_mtime);  // 获取修改的时间

    char mtime[512] = {0};
    strncpy(mtime, time, strlen(time) - 1);

    char buf[1024];
    sprintf(buf, "%s %d %s %s %ld %s %s", perms, linkNum, fileUser, fileGrp, fileSize, mtime, argv[1]);

    printf("%s\n", buf);
    return 0;
}
```


## 二、Linux多进程开发
1.为了管理进程，内核必须对每个进程所做的事情进行清楚的描述。内核为每个进程分配一个 PCB(Processing Control Block)进程控制块，维护进程相关的信息，Linux 内核的进程控制块是 task_struct 结构体。

2.在 /usr/src/linux-headers-xxx/include/linux/sched.h 文件中可以查看 struct task_struct 结构体定义。其内部成员有很多，我们只需要掌握以下部分即可：
* 进程id：系统中每个进程有唯一的 id，用 pid_t 类型表示，其实就是一个非负整数
* 进程的状态：有就绪、运行、挂起、停止等状态
* 进程切换时需要保存和恢复的一些CPU寄存器
* 述虚拟地址空间的信息
* 描述控制终端的信息
* 当前工作目录（Current Working Directory）
* umask 掩码
* 文件描述符表，包含很多指向 file 结构体的指针
* 和信号相关的信息
* 用户 id 和组 id
* 会话（Session）和进程组
* 进程可以使用的资源上限（Resource Limit）

当前系统的资源情况：ulimit -a 

3. 进程相关命令
* 查看当前终端的命令
    tty
* 查看进程  
    ps aux / ajx  
    a：显示终端上的所有进程，包括其他用户的进程  
    u：显示进程的详细信息  
    x：显示没有控制终端的进程   
    j：列出与作业控制相关的信息  
* 执行ps aux命令结果中STAT参数意义：  
    D 不可中断 Uninterruptible（usually IO）  
    R 正在运行，或在队列中的进程  
    S(大写) 处于休眠状态  
    T 停止或被追踪  
    Z 僵尸进程  
    W 进入内存交换（从内核2.6开始无效）  
    X 死掉的进程  
    < 高优先级  
    N 低优先级  
    s 包含子进程  
    + 位于前台的进程组  
* 实时显示进程动态   
    top  
    可以在使用 top 命令时加上 -d 来指定显示信息更新的时间间隔，在 top 命令执行后，可以按以下按键对显示的结果进行排序：  
    M 根据内存使用量排序  
    P 根据 CPU 占有率排序  
    T 根据进程运行时间长短排序  
    U 根据用户名来筛选进程  
    K 输入指定的 PID 杀死进程  
* 杀死进程  
    kill [-signal] pid  
    kill –l                 // 列出所有信号  
    kill –SIGKILL 进程ID    // SIGKILL就是9  
    kill -9 进程ID          // 9就是SIGKILL  
    killall name            // 根据进程名杀死进程  
* 让进程在后台运行的命令  
    ./a.out &
* 后台运行的进程切换至前台的命令  
    fg
* 进程号和相关函数  
    - 每个进程都由进程号来标识，其类型为pid_t（int类型），进程号的范围：0～32767。进程号总是唯一的，但可以重用。当一个进程终止后，其进程号就可以再次使用。
    - 任何进程（除init进程）都是由另一个进程创建，该进程称为被创建进程的父进程，对应的进程号称为父进程号（PPID）。
    - 进程组是一个或多个进程的集合。他们之间相互关联，进程组可以接收同一终端的各种信号，关联的进程有一个进程组号（PGID）。默认情况下，当前的进程号会当做当前的进程组号。
    - 进程号和进程组相关函数：  
        pid_t getpid(void);  
        pid_t getppid(void);  
        pid_t getpgid(pid_t pid);  


4.进程创建  
系统允许一个进程创建新进程，新进程即为子进程，子进程还可以创建新的子进程，形成进程树结构模型。
```
    #include <sys/types.h>
    #include <unistd.h>
    pid_t fork(void);
    返回值：
    fork的返回值会返回两次，一次是在父进程中，一次是在子进程中。所以，通过fork的返回值区分父进程和子进程。
    成功：子进程中返回0，父进程中返回子进程ID。
    失败：返回-1（在父进程中返回，表示创建子进程失败）
    失败的两个主要原因：
    1. 当前系统的进程数已经达到了系统规定的上限，这时 errno 的值被设置
    为 EAGAIN；
    2. 系统内存不足，这时 errno 的值被设置为 ENOMEM。
```
fork()以后，子进程的用户区数据和父进程一样，内核区也会拷贝过来，但是pid、ppid、信号集不一样。

5.读时共享、写时拷贝的原理

Linux 的 fork() 使用是通过写时拷贝实现。写时拷贝是一种可以推迟甚至避免拷贝数据的技术。内核并不复制整个进程的地址空间，而是让父子进程共享同一个地址空间。只有在写入时才会复制地址空间（重新开辟一块内存），从而使各个进程拥有自己的地址空间。即资源的复制只有在写入时才会进行，在此之前，只有以只读的方式进行。

fork() 之后的父子进程共享文件，此时的 fork() 产生的子进程与父进程相同的文件描述符指向相同的文件表，引用计数增加，共享文件偏移指针。 

6.GDB 多进程调试
* 使用 GDB 调试的时候，GDB 默认只能跟踪一个进程，可以在 fork 函数调用之前，通过指令设置GDB调试工具跟踪父进程或者是跟踪子进程，默认跟踪父进程。
* 设置调试父进程或者子进程：set follow-fork-mode [parent（默认）| child]
* 显示当前跟踪的模式：show follow-fork-mode
* 设置调试模式：set detach-on-fork [on | off]  
    默认为 on，表示调试当前进程的时候，其它的进程继续运行，如果为 off，调试当前进程的时候，其它进程被 GDB 挂起。
* 查看调试的进程：info inferiors
* 切换当前调试的进程：inferior 编号
* 使进程脱离 GDB 调试：detach inferiors 编号

7.exec 函数族介绍
* exec 函数族的作用是根据指定的文件名找到可执行文件，并用它来取代调用进程的内容，换句话说，就是在调用进程内部执行一个可执行文件。一般就是fork之后，在子进程里面调用exec函数。
* exec 函数族的函数执行成功后不会返回，因为调用进程的实体，包括代码段，数据段和堆栈等都已经被新的内容取代，只留下进程ID等一些表面上的信息仍保持原样。只有调用失败了，它们才会返回 -1，从原程序的调用点接着往下执行。

8.exec 函数族
* int execl(const char *path, const char *arg, .../* (char *) NULL */);
    - 常用。
    - 例子execl("mainApp", "mainApp", NULL);    // 执行当前目录下可执行程序mainApp
    - 例子execl("/bin/ps", "ps", "aux", NULL);  // (通过which ps可以知道ps的位置)执行shell命令 ps -aux
    - path是需要执行的文件路径或者名称
    - arg就是传给需要执行的文件的main函数的参数。而第一个参数和常用的main函数用法一样，传的是执行程序的名称；第二个参数开始才是程序执行所需要的参数列表；参数最后需要以NULL结束。
    - 该函数调用失败才会有返回值-1；调用成功的话，不会有返回值。

* int execlp(const char *file, const char *arg, ... /* (char *) NULL */);
    - 常用。
    - 例子execlp("mainApp", "mainApp", NULL);
    - 例子execlp("ps", "ps", "aux", NULL);
    - 该函数会到环境变量中查找指定的可执行文件，如果找到了就执行，否则执行失败。
    - file是需要执行的可执行文件的文件名

* int execle(const char *path, const char *arg, .../*, (char *) NULL, char *const envp[] */);

* int execv(const char *path, char *const argv[]);

* int execvp(const char *file, char *const argv[]);

* int execvpe(const char *file, char *const argv[], char *const envp[]);

* int execve(const char *filename, char *const argv[], char *const envp[]);
    - 前面6个函数都是标准C库的函数，而execve是Linux的函数。
    - filename即可执行程序的程序名，例如"mainApp"
    - argv中第一个参数一定要把路径写对。例如{"./mainApp", "mainApp",(char*)0};
    - 第三个参数envp数组参数不是用来查找可执行程序的，而是为可执行程序运行期间增加新的环境变量。例如{"PATH=/home/bread/Linux/code/lesson19",(char*)0}

* 函数名中的字母有如下含义：
    - l(list) 参数地址列表，以空指针结尾
    - v(vector) 存有各参数地址的指针数组的地址
    - p(path) 按 PATH 环境变量指定的目录搜索可执行文件
    - e(environment) 存有环境变量字符串地址的指针数组的地址

9.进程退出
```
// 标准C库函数
#include <stdlib.h>
void exit(int status);

// Linux系统函数
#include <unistd.h>
void _exit(int status);
```
* 调用标准C库函数的exit函数实际上是先调用退出处理函数，然后刷新I/O缓冲关闭文件描述符，最后调用_exit系统调用，然后进程终止运行。
    - printf("aaa\n");    // 因为有\n所以会刷新缓冲区
    - printf("aaa");    // 因为没有\n所以数据会暂存缓冲区，调用_exit函数将不会打印。
* status参数是进程退出时的一个状态信息，父进程回收子进程资源的时候可以获取到。

10.孤儿进程
* 父进程运行结束，但子进程还在运行（未运行结束），这样的子进程就称为孤儿进程
（Orphan Process）。
* 每当出现一个孤儿进程的时候，内核就把孤儿进程的父进程设置为init，而init进程会循环地wait()它的已经退出的子进程。这样，当一个孤儿进程凄凉地结束了其生命周期的时候，init进程就会代表党和政府出面处理它的一切善后工作。
* 因此孤儿进程并不会有什么危害。

11.僵尸进程
* 每个进程结束之后, 都会释放自己地址空间中的用户区数据，内核区的PCB没有办法自己释放掉，需要父进程去释放。
* 进程终止时，父进程尚未回收，子进程残留资源（PCB）存放于内核中，变成僵尸（Zombie）进程。
* 僵尸进程不能被 kill -9 杀死。这样就会导致一个问题，如果父进程不调用wait()或waitpid()的话，那么保留的那段信息就不会释放，其进程号就会一直被占用，但是系统所能使用的进程号是有限的，如果大量的产生僵尸进程，将因为没有可用的进程号而导致系统不能产生新的进程，此即为僵尸进程的危害，应当避免。

12.进程回收
* 在每个进程退出的时候，内核释放该进程所有的资源、包括打开的文件、占用的内存等。但是仍然为其保留一定的信息，这些信息主要主要指进程控制块PCB的信息（包括进程号、退出状态、运行时间等）。
* 父进程可以通过调用wait或waitpid得到它的退出状态同时彻底清除掉这个进程。
* wait() 和 waitpid() 函数的功能一样，区别在于，wait()函数会阻塞，waitpid()可以设置不阻塞，waitpid()还可以指定等待哪个子进程结束。
* 注意：一次wait或waitpid调用只能清理一个子进程，清理多个子进程应使用循环。
```
#include <sys/types.h>
#include <sys/wait.h>
pid_t wait(int *wstatus);
功能：等待任意一个子进程结束，如果任意一个子进程结束了，此函数会回收子进程的资源。
参数：int *wstatus 进程退出时的状态信息，传入的是一个int类型的地址，这个是传出参数。
返回值：
    - 成功：返回被回收的子进程的id
    - 失败：-1 (所有的子进程都结束，调用函数失败)

调用wait函数的进程会被挂起（阻塞），直到它的一个子进程退出或者收到一个不能被忽略的信号时才被唤醒（相当于继续往下执行）
如果没有子进程了，函数立刻返回，返回-1；如果子进程都已经结束了，也会立即返回，返回-1.
    
#include <sys/types.h>
#include <sys/wait.h>
pid_t waitpid(pid_t pid, int *wstatus, int options);
功能：回收指定进程号的子进程，可以设置是否阻塞。
参数：
    - pid:
        pid > 0 : 某个子进程的pid
        pid = 0 : 回收当前进程组的任意子进程    
        pid = -1 : 回收任意的子进程，waitpid(-1,&st,0)相当于wait(&st)  （最常用）
        pid < -1 : 某个进程组的组id的绝对值，回收指定进程组中的子进程
    - options：设置阻塞或者非阻塞
        0 : 阻塞（默认）
        WNOHANG : 非阻塞
    - 返回值：
        > 0 : 返回子进程的id
        = 0 : 在options=WNOHANG时才会返回0, 表示还有子进程活着
        = -1 ：错误，或者没有子进程了
```

13.退出信息相关宏函数
* WIFEXITED(status) 非0，进程正常退出
* WEXITSTATUS(status) 如果宏为真，获取进程退出的状态（exit的参数）
* WIFSIGNALED(status) 非0，进程异常终止
* WTERMSIG(status) 如果宏为真，获取使进程终止的信号编号
* WIFSTOPPED(status) 非0，进程处于暂停状态
* WSTOPSIG(status) 如果宏为真，获取使进程暂停的信号的编号
* WIFCONTINUED(status) 非0，进程暂停后已经继续运行
```
int st;
int ret = wait(&st);
if(-1 == ret)
    return;
if(WIFEXITED(st))
    printf("非正常退出状态码为:%d\n", WEXITSTATUS(st));
```

14.同一主机进程间通信
* Unix进程间通信方式
    - 匿名管道
    - 有名管道
    - 信号
* System V进程间通信方式、POSIX进程间通信方式
    - 消息队列
    - 共享内存
    - 信号量
还有一种方式是内存映射。

15.管道的特点
* 管道其实是一个在内核内存中维护的缓冲器，这个缓冲器的存储能力是有限的，不同的操作系统大小不一定相同。
* 管道拥有文件的特质：读操作、写操作，匿名管道没有文件实体，有名管道有文件实体，但不存储数据。可以按照操作文件的方式对管道进行操作。
* 一个管道是一个字节流，使用管道时不存在消息或者消息边界的概念，从管道读取数据的进程可以读取任意大小的数据块，而不管写入进程写入管道的数据块的大小是多少。
* 通过管道传递的数据是顺序的，从管道中读取出来的字节的顺序和它们被写入管道的顺序是完全一样的。管道其实是个环形队列。
* 在管道中的数据的传递方向是单向的，一端用于写入，一端用于读取，管道是半双工的。
* 从管道读数据是一次性操作，数据一旦被读走，它就从管道中被抛弃，释放空间以便写更多的数据，在管道中无法使用 lseek() 来随机的访问数据。
* 匿名管道只能在具有公共祖先的进程（父进程与子进程，或者两个兄弟进程，具有亲缘关系）之间使用。因为它们共享文件描述符(例如在fork之前创建管道)。
* 自Linux2.6.11内核起，管道容量的大小默认是65536字节。我们可以使用fcntl函数来修改管道容量。

16. 匿名管道的使用

管道也叫无名（匿名）管道，它是 UNIX 系统 IPC（进程间通信）的最古老形式，所有的 UNIX 系统都支持这种通信机制。  
例如：统计一个目录中文件的数目命令：ls | wc –l，为了执行该命令，shell 创建了两个进程来分别执行 ls 和 wc。(|是管道符，前面的进程通过管道发数据给后面的进程，后面的进程处理后把结果显示在界面上)

匿名管道是半双工的，不允许同时双向读写。因此，父进程只保留读端或写段，而将写段或者读端关闭；子进程也一样。

* 创建匿名管道  
    #include <unistd.h>  
    int pipe(int pipefd[2]);    // pipefd[0]是读，pipefd[1]是写.  
* 查看管道缓冲大小命令  
    ulimit –a  
* 查看管道缓冲大小函数  
    #include <unistd.h>  
    long fpathconf(int fd, int name);  
```
int pipefd[2];
pipe(pipefd);
long size = fpathconf(pipefd[0], _PC_PIPE_BUF);
// size和命令ulimit –a的结果是一样的。
```
注意：匿名管道只能用于具有关系的进程之间的通信(父子进程、兄弟进程)。

管道默认是阻塞的：如果管道中没有数据，read阻塞；如果管道满了，write阻塞。

如果要实现双向的数据传输，就应该使用两个管道。

把管道设置为非阻塞：  
例如把读端设置为非阻塞：  
int flags = fcntl(pipefd[0], F_GETFL);  
flags |= O_NONBLOCK;  
fcntl(pipefd[0], F_SETFL, flags);  

```
/*
    实现 ps aux | grep xxx 父子进程间通信
    
    子进程： ps aux, 子进程结束后，将数据发送给父进程
    父进程：获取到数据，过滤
    pipe()
    execlp()
    子进程将标准输出 stdout_fileno 重定向到管道的写端。  dup2
*/
#include <unistd.h>
#include <sys/types.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <wait.h>

int main() {

    // 创建一个管道
    int fd[2];
    int ret = pipe(fd);
    if(ret == -1) 
    {
        perror("pipe");
        exit(0);
    }

    pid_t pid = fork();    // 创建子进程

    if(pid > 0)     // 父进程
    {
        close(fd[1]);   // 关闭写端
        
        // 从管道中读取
        char buf[1024] = {0};
        int len = -1;
        while((len = read(fd[0], buf, sizeof(buf) - 1)) > 0) // 减1是因为字符串的结束符
        {
            // 过滤数据输出
            printf("%s", buf);
            memset(buf, 0, 1024);
        }
        wait(NULL);     // 回收子进程的资源
    } 
    else if(pid == 0)   // 子进程
    {
        close(fd[0]);   // 关闭读端

        // 文件描述符的重定向 stdout_fileno -> fd[1]
        dup2(fd[1], STDOUT_FILENO);
        
        // 执行 ps aux
        execlp("ps", "ps", "aux", NULL);
        perror("execlp");
        exit(0);
    } 
    else
    {
        perror("fork");
        exit(0);
    }
    return 0;
}
```

17.管道的读写特点

使用管道时，需要注意以下几种特殊的情况（假设都是阻塞I/O操作）
* 所有的指向管道写端的文件描述符都关闭了（管道写端引用计数为0），有进程从管道的读端读数据，那么管道中剩余的数据被读取以后，再次read会返回0，就像读到文件末尾一样。
* 如果有指向管道写端的文件描述符没有关闭（管道的写端引用计数大于0），而持有管道写端的进程也没有往管道中写数据，这个时候有进程从管道中读取数据，那么管道中剩余的数据被读取后，再次read会阻塞，直到管道中有数据可以读了才读取数据并返回。
* 如果所有指向管道读端的文件描述符都关闭了（管道的读端引用计数为0），这个时候有进程向管道中写数据，那么该进程会收到一个信号SIGPIPE,通常会导致进程异常终止。
* 如果有指向管道读端的文件描述符没有关闭（管道的读端引用计数大于0），而持有管道读端的进程也没有从管道中读数据，这时有进程向管道中写数据，那么在管道被写满的时候再次write会阻塞，直到管道中有空位置才能再次写入数据并返回。

总结：（下面的有名管道也是下面这些特点）
* 读管道：
    - 管道中有数据，read返回实际读到的字节数。
    - 管道中无数据：写端被全部关闭，read返回0（相当于读到文件的末尾）；写端没有完全关闭，read阻塞等待
* 写管道：
    - 管道读端全部被关闭，进程异常终止（进程收到SIGPIPE信号）
    - 管道读端没有全部关闭：管道已满，write阻塞；管道没有满，write将数据写入，并返回实际写入的字节数

18.有名管道
* 匿名管道，由于没有名字，只能用于亲缘关系的进程间通信。为了克服这个缺点，提出了有名管道（FIFO），也叫命名管道、FIFO文件。
* 有名管道（FIFO）不同于匿名管道之处在于它提供了一个路径名与之关联，以FIFO的文件形式存在于文件系统中，并且其打开方式与打开一个普通文件是一样的，这样即使与 FIFO 的创建进程不存在亲缘关系的进程，只要可以访问该路径，就能够彼此通过 FIFO 相互通信，因此，通过 FIFO，不相关的进程也能交换数据。
* 一旦打开了 FIFO，就能在它上面使用与操作匿名管道和其他文件的系统调用一样的I/O系统调用了（如read()、write()和close()）。与管道一样，FIFO 也有一个写入端和读取端，并且从管道中读取数据的顺序与写入的顺序是一样的。FIFO的名称也由此而来：先入先出。
* 有名管道（FIFO)和匿名管道（pipe）有一些特点是相同的，不一样的地方在于：
    - FIFO 在文件系统中作为一个特殊文件存在，但 FIFO 中的内容却存放在内存中。
    - 当使用 FIFO 的进程退出后，FIFO文件将继续保存在文件系统中以便以后使用。
    - FIFO 有名字，不相关的进程可以通过打开有名管道进行通信。

19.有名管道的使用
* 通过命令创建有名管道
    mkfifo 名字
* 通过函数创建有名管道  
    #include <sys/types.h>  
    #include <sys/stat.h>  
    int mkfifo(const char *pathname, mode_t mode);  
    - pathname是管道名称的途径
    - mode是文件的权限，和open文件的权限是一样的。可以是一个八进制的数。
* 一旦使用 mkfifo 创建了一个FIFO，就可以使用open打开它，常见的文件I/O函数都可用于fifo。如：close、read、write、unlink 等。
* FIFO 严格遵循先进先出（First in First out），对管道及FIFO的读总是从开始处返回数据，对它们的写则把数据添加到末尾。它们不支持诸如 lseek() 等文件定位操作。
* 当只有写断程序(权限是只写)或者读端程序(权限是只读)在运行时，写断程序或者读端程序会在open管道的函数处阻塞。

20.内存映射
* 概念：内存映射（Memory-mapped I/O）是将磁盘文件的数据映射到内存，用户通过修改内存就能修改磁盘文件。
* 内存映射相关系统调用
```
    #include <sys/mman.h>  
    void *mmap(void *addr, size_t length, int prot, int flags, int fd, off_t offset);  
        - mmap函数用于申请一段内存空间。我们可以将这段内存作为进程间通信的共享内存也可以将文件直接映射到其中。  
        - addr是映射的内存的首地址，一般传NULL，由内核指定，然后通过返回值返回。  
        - length必须大于0，映射数据的长度。建议使用文件的长度(stat或lseek函数可以获取文件的长度)。  
        - prot是对申请的内存映射区的操作权限。  
            -PROT_EXEC ：可执行的权限  
            -PROT_READ ：读权限  
            -PROT_WRITE ：写权限  
            -PROT_NONE ：没有访问权限  
            要操作映射内存，必须要有读的权限。  
        - flags :  
            -MAP_SHARED :   映射区的数据会自动和磁盘文件进行同步，进程间通信，必须要设置这个选项  
            -MAP_PRIVATE   ：不同步，内存映射区的数据改变了，对原来的文件不会修改，会重新创建一个新的文件。（copy on write）  
            -MAP_SHARED和MAP_PRIVATE是互斥的，不能同时指定。  
        -  fd: 需要映射的那个文件的文件描述符  
                -通过open得到，open的是一个磁盘文件  
                -注意：文件的大小不能为0，open指定的权限不能和prot参数有冲突。  
                    prot: PROT_READ                要求open:只读/读写   
                    prot: PROT_READ | PROT_WRITE   要求open:读写  
        - offset：偏移量，一般不用。必须指定的是4k的整数倍，0表示不偏移。  
        - 返回值：返回创建的内存的首地址。如果失败返回MAP_FAILED，其实就是(void *) -1。  
    int munmap(void *addr, size_t length);  
        - 功能：释放内存映射（释放由mmap创建的这段内存空间）  
        - 参数：  
            - addr : 要释放的内存的首地址  
            - length : 要释放的内存的大小，要和mmap函数中的length参数的值一样。  
```

21.使用内存映射实现进程间通信：
* 有关系的进程（父子进程）  
    还没有子进程的时候，通过唯一的父进程，先创建内存映射区。有了内存映射区以后，创建子进程。父子进程共享创建的内存映射区。
* 没有关系的进程间通信  
    先准备一个大小不是0的磁盘文件。进程1通过磁盘文件创建内存映射区得到一个操作这块内存的指针。进程2通过磁盘文件创建内存映射区得到一个操作这块内存的指针。它俩使用内存映射区通信。

注意：内存映射区通信，是非阻塞。
```
#include <stdio.h>
#include <sys/mman.h>
#include <fcntl.h>
#include <sys/types.h>
#include <unistd.h>
#include <string.h>
#include <stdlib.h>
#include <sys/wait.h>
int main() 
{
    int fd = open("test.txt", O_RDWR);  // 打开一个文件(假设文件已经存在)
    int size = lseek(fd, 0, SEEK_END);  // 获取文件的大小

    void *ptr = mmap(NULL, size, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);    // 创建内存映射区
    
    // 这里也可以使用匿名内存映射区(不需要文件实体进程一个内存映射)
    // 匿名内存映射只能用于父子进程之间的通信
    // int len = 4096;
    // void * ptr = mmap(NULL, len, PROT_READ | PROT_WRITE, MAP_SHARED | MAP_ANONYMOUS, -1, 0);
    
    if(ptr == MAP_FAILED)
    {
        perror("mmap");
        exit(0);
    }

    pid_t pid = fork();
    if(pid > 0)         // 父进程
    {
        wait(NULL);

        char buf[64];
        strcpy(buf, (char *)ptr);
        printf("read data : %s\n", buf);
    }
    else if(pid == 0)  // 子进程
    {
        strcpy((char *)ptr, "helloworld!");
    }

    munmap(ptr, size);  // 关闭内存映射区
    return 0;
}
```

22.内存映射的注意事项
* 如果对mmap的返回值(ptr)做++操作(ptr++), munmap是否能够成功?  
答：void * ptr = mmap(...);  
    ptr++;  可以对其进行++操作  
    munmap(ptr, len);   // 错误,要保存原地址

* 如果open时O_RDONLY, mmap时prot参数指定PROT_READ | PROT_WRITE会怎样?  
答：错误，返回MAP_FAILED。open()函数中的权限建议和prot参数的权限保持一致。

* 如果文件偏移量为1000会怎样?  
答：偏移量必须是4K的整数倍，返回MAP_FAILED

* mmap什么情况下会调用失败?  
答：第二个参数：length = 0  
    第三个参数：prot只指定了写权限。  
    第三个参数：prot指定了读写权限(prot PROT_READ | PROT_WRITE)，而第五个参数fd通过open函数打开时指定的是 O_RDONLY / O_WRONLY

* 可以open的时候O_CREAT一个新文件来创建映射区吗?  
答：可以的，但是创建的文件的大小如果为0的话，肯定不行。  
    可以对新的文件进行扩展用lseek()或truncate()  

* mmap后关闭文件描述符，对mmap映射有没有影响？  
答：int fd = open("XXX");  
    mmap(,,,,fd,0);  
    close(fd);   
    映射区还存在，创建映射区的fd被关闭，没有任何影响。  

* 对ptr越界操作会怎样？  
答：void * ptr = mmap(NULL, 100,,,,,);  
    4K  
    越界操作操作的是非法的内存 -> 段错误  

23.通过内存映射实现拷贝文件(只能拷贝小文件)
```
#include <stdio.h>
#include <sys/mman.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <string.h>
#include <stdlib.h>
int main() 
{
    int fd = open("text.txt", O_RDWR);   // 打开原始的文件
    if(fd == -1) 
    {
        perror("open");
        exit(0);
    }
   
    int fd1 = open("cpy.txt", O_RDWR | O_CREAT, 0664);  // 创建一个新文件（拓展该文件）
    if(fd1 == -1) 
    {
        perror("open");
        exit(0);
    }
    
    int len = lseek(fd, 0, SEEK_END);   // 获取原始文件的大小
    truncate("cpy.txt", len);   // 对新创建的文件进行拓展
    write(fd1, " ", 1);     // 如果不写的话它是拓展不了的。

    // 分别做内存映射
    void * ptr = mmap(NULL, len, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);
    void * ptr1 = mmap(NULL, len, PROT_READ | PROT_WRITE, MAP_SHARED, fd1, 0);

    if(ptr == MAP_FAILED) 
    {
        perror("mmap");
        exit(0);
    }
    if(ptr1 == MAP_FAILED) 
    {
        perror("mmap");
        exit(0);
    }
    
    memcpy(ptr1, ptr, len);     // 内存拷贝
    
    // 释放资源，后来创建的先释放的原则(就有点像继承时的析构)
    munmap(ptr1, len);
    munmap(ptr, len);
    close(fd1);
    close(fd);
    return 0;
}
```

24.信号的概念
* 信号是 Linux 进程间通信的最古老的方式之一，是事件发生时对进程的通知机制，有时也称之为软件中断，它是在软件层次上对中断机制的一种模拟，是一种异步通信的方式。信号可以导致一个正在运行的进程被另一个正在运行的异步进程中断，转而处理某一个突发事件。
* 发往进程的诸多信号，通常都是源于内核。引发内核为进程产生信号的各类事件如下：
    - 对于前台进程，用户可以通过输入特殊的终端字符来给它发送信号。比如输入Ctrl+C通常会给进程发送一个中断信号。
    - 硬件发生异常，即硬件检测到一个错误条件并通知内核，随即再由内核发送相应信号给相关进程。比如执行一条异常的机器语言指令，诸如被 0 除，或者引用了无法访问的内存区域。
    - 系统状态变化，比如 alarm 定时器到期将引起 SIGALRM 信号，进程执行的CPU时间超限，或者该进程的某个子进程退出。
    - 运行 kill 命令或调用 kill 函数。
* 使用信号的两个主要目的是：
    - 让进程知道已经发生了一个特定的事情。
    - 强迫进程执行它自己代码中的信号处理程序。
* 信号的特点：
    - 使用简单
    - 不能携带大量信息
    - 满足某个特定条件才发送
    - 优先级比较高
* 查看系统定义的信号列表：kill –l 

25.Linux 信号一览表（前 31 个信号为常规信号，其余为实时信号）

编号 | 信号名称 | 对应事件 | 默认动作
---|---|---|---
1 | SIGHUP | 用户退出shell时，由该shell启动的所有进程将收到这个信号 | 终止进程
2 | SIGINT | 当用户按下了<Ctrl+C>组合键时，用户终端向正在运行中的由该终端启动的程序发出此信号 | 终止进程
3 | SIGQUIT | 用户按下<Ctrl+\>组合键时产生该信号，用户终端向正在运行中的由该终端启动的程序发出些信号 | 终止进程
4 | SIGILL | CPU检测到某进程执行了非法指令 | 终止进程并产生core文件
5 | SIGTRAP | 该信号由断点指令或其他trap指令产生 | 终止进程并产生core文件
6 | SIGABRT | 调用abort函数时产生该信号 | 终止进程并产生core文件
7 | SIGBUS | 非法访问内存地址，包括内存对齐出错 终止进程并产生core文件
8 | SIGFPE | 在发生致命的运算错误时发出。不仅包括浮点运算错误，还包括溢出及除数为0等所有的算法错误 | 终止进程并产生core文件
9 | SIGKILL | 无条件终止进程。该信号不能被忽略，处理和阻塞 | 终止进程，可以杀死任何进程
10 | SIGUSE1 | 用户定义的信号。即程序员可以在程序中定义并使用该信号 | 终止进程
11 | SIGSEGV | 指示进程进行了无效内存访问(段错误) | 终止进程并产生core文件
12 | SIGUSR2 | 另外一个用户自定义信号，程序员可以在程序中定义并使用该信号 | 终止进程
13 | SIGPIPE | Broken pipe向一个没有读端的管道写数据 | 终止进程
14 | SIGALRM | 定时器超时，超时的时间由系统调用alarm设置 | 终止进程
15 | SIGTERM | 程序结束信号，与SIGKILL不同的是，该信号可以被阻塞和终止。通常用来要示程序正常退出。执行shell命令Kill时，缺省产生这个信号 | 终止进程
16 | SIGSTKFLT | Linux早期版本出现的信号，现仍保留向后兼容 | 终止进程
17 | SIGCHLD | 子进程结束时，父进程会收到这个信号 | 忽略这个信号
18 | SIGCONT | 如果进程已停止，则使其继续运行 | 继续/忽略
19 | SIGSTOP | 停止进程的执行。信号不能被忽略，处理和阻塞 | 为终止进程
20 | SIGTSTP | 停止终端交互进程的运行。按下<ctrl+z>组合键时发出这个信号 | 暂停进程
21 | SIGTTIN | 后台进程读终端控制台 | 暂停进程
22 | SIGTTOU | 该信号类似于SIGTTIN，在后台进程要向终端输出数据时发生 | 暂停进程
23 | SIGURG | 套接字上有紧急数据时，向当前正在运行的进程发出些信号，报告有紧急数据到达。如网络带外数据到达 | 忽略该信号
24 | SIGXCPU | 进程执行时间超过了分配给该进程的CPU时间，系统产生该信号并发送给该进程 | 终止进程
25 | SIGXFSZ | 超过文件的最大长度设置 | 终止进程
26 | SIGVTALRM | 虚拟时钟超时时产生该信号。类似于SIGALRM，但是该信号只计算该进程占用CPU的使用时间 | 终止进程
27 | SGIPROF | 类似于SIGVTALRM，它不公包括该进程占用CPU时间还包括执行系统调用时间 | 终止进程
28 | SIGWINCH | 窗口变化大小时发出 | 忽略该信号
29 | SIGIO | 此信号向进程指示发出了一个异步IO事件 | 忽略该信号
30 | SIGPWR | 关机 | 终止进程
31 | SIGSYS | 无效的系统调用 | 终止进程并产生core文件
34~64 | SIGRTMIN ～SIGRTMAX | LINUX的实时信号，它们没有固定的含义（可以由用户自定义）| 终止进程

26.信号的 5 种默认处理动作
* 查看信号的详细信息：man 7 signal
* 信号的 5 种默认处理动作
    - Term 终止进程
    - Ign 当前进程忽略掉这个信号
    - Core 终止进程，并生成一个Core文件
    - Stop 暂停当前进程
    - Cont 继续执行当前被暂停的进程
* 信号的几种状态：产生、未决(还没有处理的状态)、递达
* SIGKILL 和 SIGSTOP 信号不能被捕捉、阻塞或者忽略，只能执行默认动作。

27.Core文件
* 文件内容是程序运行过程中的错误信息
* 生成core文件
    - ulimit -a  看到core file size是0
    - ulimit -c 1024    把core file size设置为1024或者直接设置为unlimited
    - gcc test.c -g
    - ./a.out
    - 如果a.out异常退出例如报"段错误(核心已转储)"，然后就生成了一个core文件
    - gdb a.out
    - (gdb) core-file core 看错误信息

28.信号相关的函数
```
* int kill(pid_t pid, int sig);
    - 给任何的进程或进程组pid，发送任何的信号sig
    - pid ：
        > 0 : 将信号发送给指定的进程  
        = 0 : 将信号发送给当前的进程组所有的进程  
        = -1 : 将信号发送给每一个有权限接收这个信号的进程  
        < -1 : 这个pid=某个进程组的ID取反(取负数)
    - sig : 需要发送的信号的编号或者是宏值，0表示不发送任何信号  
    - 例如：  
        kill(getppid(), 9);  
        kill(getpid(), 9);  

* int raise(int sig);
    - 给当前进程发送信号
    
* void abort(void);
    - 发送SIGABRT信号给当前的进程(默认是杀死当前进程)。相当于kill(getpid(), SIGABRT);
    
* unsigned int alarm(unsigned int seconds);
    - 设置定时器。函数调用，开始倒计时，当倒计时为0的时候，函数会给当前的进程发送一个信号：SIGALARM
    - seconds: 倒计时的时长，单位：秒。如果参数为0，定时器无效（不进行倒计时，不发信号）。取消一个定时器，通过alarm(0)。
    - 返回值：之前没有定时器，返回0；之前有定时器，返回之前定时器的剩余时间。
    - SIGALARM ：默认终止当前的进程，每一个进程都有且只有唯一的一个定时器。  
        alarm(10);  -> 返回0  
        过了1秒  
        alarm(5);   -> 返回9  
    - alarm函数是非阻塞的。
    - 定时器，与进程的状态无关（自然定时法）。无论进程处于什么状态，alarm都会计时。

* int setitimer(int which, const struct itimerval *new_value, struct itimerval *old_value);
    - 设置定时器。可以替代alarm函数。精度微妙us，可以实现周期性定时
    - which : 定时器以什么时间计时  
            ITIMER_REAL: 真实时间，时间到达，发送 SIGALRM。(常用)  
            ITIMER_VIRTUAL: 用户时间，时间到达会发送 SIGVTALRM  
            ITIMER_PROF: 以该进程在用户态和内核态下所消耗的时间来计算，时间到达会发送 SIGPROF  
    - new_value: 设置定时器的属性  
        struct itimerval {      // 定时器的结构体
            struct timeval it_interval;  // 每个阶段的时间，间隔时间
            struct timeval it_value;     // 延迟多长时间执行定时器
        };
        struct timeval {        // 时间的结构体
            time_t      tv_sec;     //  秒数     
            suseconds_t tv_usec;    //  微秒    
        };
        例如，过10秒后，每隔2秒定时一次：则it_value是10，it_interval是2.
    - old_value ：记录上一次的定时的时间参数，一般不使用，指定NULL
    - 返回值：成功 0；失败 -1 并设置错误号
    - setitimer函数是非阻塞的。
```

29.信号捕捉函数
```
* sighandler_t signal(int signum, sighandler_t handler);
    - typedef void (*sighandler_t)(int);    // int参数表示捕获到的信号的值
    - 设置某个信号的捕捉行为
    - signum: 要捕捉的信号
    - handler: 捕捉到信号要如何处理
        SIG_IGN ： 忽略信号
        SIG_DFL ： 使用信号默认的行为
        回调函数 :  这个函数是内核调用，程序员只负责写，捕捉到信号后如何去处理信号。
    - 回调函数：需要程序员实现，提前准备好的，函数的类型根据实际需求，看函数指针的定义；不是程序员调用，而是当信号产生，由内核调用。函数指针是实现回调的手段，函数实现之后，将函数名放到函数指针的位置就可以了。
    - 返回值：成功，返回上一次注册的信号处理函数的地址。第一次调用返回NULL；失败，返回SIG_ERR，设置错误号
    - SIGKILL和SIGSTOP不能被捕捉，不能被忽略。

* int sigaction(int signum, const struct sigaction *act, struct sigaction *oldact);
    - 检查或者改变信号的处理。其实就是信号捕捉。
    - signum : 需要捕捉的信号的编号或者宏值（信号的名称）
    - act ：捕捉到信号之后的处理动作
    - oldact : 上一次对信号捕捉相关的设置，一般不使用，传递NULL
    - 返回值：成功 0；失败 -1
    struct sigaction {
        void     (*sa_handler)(int);    // 函数指针，指向的函数就是信号捕捉到之后的处理函数
        void     (*sa_sigaction)(int, siginfo_t *, void *); // 不常用
        sigset_t   sa_mask; // 临时阻塞信号集，在信号捕捉函数执行过程中，临时阻塞某些信号。
        int        sa_flags;    // 使用哪一个信号处理对捕捉到的信号进行处理。这个值可以是0表示使用sa_handler,也可以是SA_SIGINFO表示使用sa_sigaction
        void     (*sa_restorer)(void);// 被废弃掉了
    };
    - 系统处理完临时阻塞信号集之后，又会回到系统的阻塞信号集。
* 信号处理函数的原型如下：
    #include <signal.h>
    typedef void (*_sighandler_t) (int);
    - 信号处理函数应该是可重入的，否则很容易引发一些竞态条件。因此在信号处理函数中严禁调用一些不安全的函数。
```

30.信号集
* 许多信号相关的系统调用都需要能表示一组不同的信号，多个信号可使用一个称之为信号集的数据结构来表示，其系统数据类型为 sigset_t。
* 在 PCB 中有两个非常重要的信号集。一个称之为 “阻塞信号集” ，另一个称之为“未决信号集” 。这两个信号集都是内核使用位图机制来实现的。但操作系统不允许我们直接对这两个信号集进行位操作。而需自定义另外一个集合，借助信号集操作函数来对 PCB 中的这两个信号集进行修改。
* 信号的 “未决” 是一种状态，指的是从信号的产生到信号被处理前的这一段时间。
* 信号的 “阻塞” 是一个开关动作，指的是阻止信号被处理，但不是阻止信号产生。
* 信号的阻塞就是让系统暂时保留信号留待以后发送。由于另外有办法让系统忽略信号，所以一般情况下信号的阻塞只是暂时的，只是为了防止信号打断敏感的操作。

31.阻塞信号集和未决信号集
* 用户通过键盘  Ctrl + C, 产生2号信号SIGINT (信号被创建)
* 信号产生但是没有被处理 （未决）
    - 在内核中将所有的没有被处理的信号存储在一个集合中 （未决信号集）
    - SIGINT信号状态被存储在第二个标志位上。这个标志位的值为0， 说明信号不是未决状态；这个标志位的值为1， 说明信号处于未决状态。
* 这个未决状态的信号，需要被处理，处理之前需要和另一个信号集（阻塞信号集），进行比较
    - 阻塞信号集默认不阻塞任何的信号
    - 如果想要阻塞某些信号需要用户调用系统的API
* 在处理的时候和阻塞信号集中的标志位进行查询，看是不是对该信号设置阻塞了
    - 如果没有阻塞，这个信号就被处理
    - 如果阻塞了，这个信号就继续处于未决状态，直到阻塞解除，这个信号就被处理
* 常规信号的处理不是进队列，假设在A信号处理的过程中，又发生了一次A信号，此时改变未决信号集的A信号标记位为1，然后又发生了一次A信号，那么等A信号处理完之后，系统会再调用一次A信号的处理函数(虽然后面又发生了两次A信号，但只会执行一次)。但是实时信号的处理是进队列的(34~64号)。

32.信号集相关的函数
```
以下信号集相关的函数都是对自定义的信号集进行操作。
* int sigemptyset(sigset_t *set);
    - 功能：清空信号集中的数据,将信号集中的所有的标志位置为0
    - 参数：set,传出参数，需要操作的信号集
    - 返回值：成功返回0， 失败返回-1
* int sigfillset(sigset_t *set);
    - 功能：将信号集中的所有的标志位置为1
    - 参数：set,传出参数，需要操作的信号集
    - 返回值：成功返回0， 失败返回-1
* int sigaddset(sigset_t *set, int signum);
    - 功能：设置信号集中的某一个信号对应的标志位为1，表示阻塞这个信号
    - set：传出参数，需要操作的信号集
    - signum：需要设置阻塞的那个信号
    - 返回值：成功返回0， 失败返回-1
* int sigdelset(sigset_t *set, int signum);
    - 功能：设置信号集中的某一个信号对应的标志位为0，表示不阻塞这个信号
    - set：传出参数，需要操作的信号集
    - signum：需要设置不阻塞的那个信号
    - 返回值：成功返回0， 失败返回-1
* int sigismember(const sigset_t *set, int signum);
    - 功能：判断某个信号是否阻塞（即是否在这个信号集里面）
    - set：需要操作的信号集
    - signum：需要判断的那个信号
    - 返回值：1 ： signum被阻塞（在）；0 ： signum不阻塞（不在）；-1 ： 失败
    
以下信号集相关的函数可以对系统的信号集进行操作。但也不能修改未决信号集，只能修改阻塞信号集。阻塞信号集也称信号掩码。
* int sigprocmask(int how, const sigset_t *set, sigset_t *oldset);
    - 功能：将自定义信号集中的数据设置到内核中（设置阻塞，解除阻塞，替换）
    - how : 如何对内核阻塞信号集进行处理
        SIG_BLOCK: 将用户设置的阻塞信号集添加到内核中。假设内核中默认的阻塞信号集是mask， 则mask |= set
        SIG_UNBLOCK: 根据用户设置的数据，对内核中的数据进行解除阻塞，即 mask &= ~set
        SIG_SETMASK：覆盖内核中原来的值
    - set ：已经初始化好的用户自定义的信号集
    - oldset : 保存设置之前的内核中的阻塞信号集的状态，可以是 NULL
    - 返回值：成功：0；失败：-1，设置错误号：EFAULT、EINVAL
* int sigpending(sigset_t *set);
    - 功能：获取内核中的未决信号集
    - 参数：set,传出参数，保存的是内核中的未决信号集中的信息。
```

33.SIGCHLD信号
* SIGCHLD信号产生的条件
    - 子进程终止时
    - 子进程接收到 SIGSTOP 信号停止时
    - 子进程处在停止态，接受到SIGCONT后唤醒时
* 以上三种条件都会给父进程发送 SIGCHLD 信号，父进程默认会忽略该信号
* 使用SIGCHLD信号解决僵尸进程的问题。

34.网络编程相关的信号
* SIGHUP
    - 当挂起进程的控制终端时，SIGHUP信号将被触发。对于没有控制终端的网络后台程序而言，它们通常利用SIGHUP信号来强制服务器重读配置文件。
* SIGPIPE
    - 默认情况下，往一个读端关闭的管道或socket连接中写数据将引发SIGPIPE信号。
* SIGURG
    - 在Linux环境中，内核通知应用程序带外数据到达主要有两种方法：一种是I/O复用技术，select等系统调用在接收到带外数据时将返回，并向应用程序报告socket上的异常事件；另一种方法就是使用SIGURG信号。
    - 使用SIGURG信号之前，我们必须设置socket的宿主进程或进程组。

35.统一信号源

信号处理函数通常使用管道来将信号“传递”给主循环：信号处理函数往管道的写端写入信号值，主循环则从管道的读端读出该信号值。

我们需要使用I/O复用系统调用来监听管道的读端文件描述符上的可读时间。如此一来，信号事件就能和其他I/O事件一样被处理，即统一事件源。

36.共享内存
* 共享内存允许两个或者多个进程共享物理内存的同一块区域（通常被称为段）。由于一个共享内存段会成为一个进程用户空间的一部分，因此这种 IPC 机制无需内核介入(相对于其他IPC而言，进内核次数少，并不是完全没有)。所有需要做的就是让一个进程将数据复制进共享内存中，并且这部分数据会对其他所有共享同一个段的进程可用。
* 与管道等要求发送进程将数据从用户空间的缓冲区复制进内核内存和接收进程将数据从内核内存复制进用户空间的缓冲区的做法相比，这种 IPC 技术的速度更快。

37.共享内存使用步骤
* 调用 shmget() 创建一个新共享内存段或取得一个既有共享内存段的标识符（即由其他进程创建的共享内存段）。这个调用将返回后续调用中需要用到的共享内存标识符。
* 使用 shmat() 来附上共享内存段，使该段成为调用进程的虚拟内存的一部分。
* 此刻在程序中可以像对待其他可用内存那样对待这个共享内存段。为引用这块共享内存，程序需要使用由 shmat() 调用返回的 addr 值，它是一个指向进程的虚拟地址空间中该共享内存段的起点的指针。
* 调用 shmdt() 来分离共享内存段。在这个调用之后，进程就无法再引用这块共享内存了。这一步是可选的，并且在进程终止时会自动完成这一步。
* 调用 shmctl() 来删除共享内存段。只有当当前所有附加内存段的进程都与之分离之后内存段才会销毁。只有一个进程需要执行这一步。

38.共享内存操作函数
```
* int shmget(key_t key, size_t size, int shmflg);
    - 功能：创建一个新的共享内存段，或者获取一个既有的共享内存段的标识。新创建的内存段中的数据都会被初始化为0
    - key : key_t类型是一个整形，通过这个找到或者创建一个共享内存。一般使用16进制表示，非0值。
    - size: 共享内存的大小，单位是字节
    - shmflg: 属性
        -访问权限
        -附加属性：创建/判断共享内存是不是存在
                -创建：IPC_CREAT
                -判断共享内存是否存在： IPC_EXCL，需要和IPC_CREAT一起使用
                -例如 IPC_CREAT | IPC_EXCL | 0664
    - 返回值：
        -失败：-1 并设置错误号
        -成功：>0 返回共享内存的引用的ID，后面操作共享内存都是通过这个值。
* void *shmat(int shmid, const void *shmaddr, int shmflg);
    - 功能：和当前的进程进行关联
    - shmid : 共享内存的标识（ID）,由shmget返回值获取
    - shmaddr: 申请的共享内存的起始地址，一般为指定NULL，由内核指定
    - shmflg : 对共享内存的操作
        -读 ： SHM_RDONLY, 必须要有读权限
        -读写： 0
    - 返回值：成功：返回共享内存的首（起始）地址。  失败(void *) -1
* int shmdt(const void *shmaddr);
    - 功能：解除当前进程和共享内存的关联
    - shmaddr：共享内存的首地址
    - 返回值：成功 0， 失败 -1
* int shmctl(int shmid, int cmd, struct shmid_ds *buf);
    - 功能：对共享内存进行操作。主要是用来删除共享内存，共享内存要删除才会消失，创建共享内存的进程被销毁了对共享内存是没有任何影响。
    - shmid: 共享内存的ID
    - cmd : 要做的操作
        -IPC_STAT : 获取共享内存的当前的状态
        -IPC_SET : 设置共享内存的状态
        -IPC_RMID: 标记共享内存被销毁
    - buf：需要设置或者获取的共享内存的属性信息
        -cmd为IPC_STAT时: buf用来存储数据，传出参数
        -cmd为IPC_SET时: buf中需要初始化数据，设置到内核中
        -cmd为IPC_RMID时: buf没有用，传NULL
* key_t ftok(const char *pathname, int proj_id);
    - 功能：根据指定的路径名和int值，生成一个共享内存的key
    - pathname：指定一个必须存在的路径
    - proj_id: int类型的值，但是只会使用其中的1个字节。范围：0-255， 一般指定一个字符 'a'
```

39.共享内存常见问题
* 操作系统如何知道一块共享内存被多少个进程关联？
    - 共享内存维护了一个结构体struct shmid_ds 这个结构体中有一个成员 shm_nattch。shm_nattach 记录了关联的进程个数

* 可不可以对共享内存进行多次删除shmctl
    - 可以的
    - 因为shmctl 标记删除共享内存，不是直接删除
    - 什么时候真正删除呢?  
        当和共享内存关联的进程数为0的时候，就真正被删除
    - 当共享内存的key为0的时候，表示共享内存被标记删除了。  
        如果一个进程和共享内存取消关联，那么这个进程就不能继续操作这个共享内存，也不能再次进行关联。

40.共享内存和内存映射的区别
* 共享内存可以直接创建，内存映射需要磁盘文件（匿名映射除外）
* 共享内存效果更高
* 内存
    - 所有的进程操作的是同一块共享内存。
    - 内存映射，每个进程在自己的虚拟地址空间中有一个独立的内存。
* 数据安全
    - 进程突然退出  
        共享内存还存在。内存映射区消失
    - 运行进程的电脑死机，宕机了  
        数据存储在共享内存中，没有了。内存映射区的数据 ，由于磁盘文件中的数据还在，所以内存映射区的数据还存在。
* 生命周期
    - 内存映射区：进程退出，内存映射区销毁。
    - 共享内存：进程退出，共享内存还在，标记删除（所有的关联的进程数为0），或者关机。  
    如果一个进程退出，会自动和共享内存进行取消关联。

41.共享内存操作命令
* ipcs 用法
    - ipcs -a   // 打印当前系统中所有的进程间通信方式的信息
    - ipcs -m   // 打印出使用共享内存进行进程间通信的信息
    - ipcs -q   // 打印出使用消息队列进行进程间通信的信息
    - ipcs -s   // 打印出使用信号进行进程间通信的信息
* ipcrm 用法
    - ipcrm -M shmkey   // 移除用shmkey创建的共享内存段
    - ipcrm -m shmid    // 移除用shmid标识的共享内存段(连接数不变，键变全0)
    - ipcrm -Q msgkey   // 移除用msqkey创建的消息队列
    - ipcrm -q msqid    // 移除用msqid标识的消息队列
    - ipcrm -S semkey   // 移除用semkey创建的信号
    - ipcrm -s semid    // 移除用semid标识的信号

42.终端
* 在 UNIX 系统中，用户通过终端登录系统后得到一个 shell 进程，这个终端成为 shell 进程的控制终端（Controlling Terminal），在进程中，控制终端是保存在PCB中的信息，而fork()会复制PCB中的信息，因此由shell进程启动的其它进程的控制终端也是这个终端。
* 默认情况下（且没有重定向），每个进程的标准输入、标准输出和标准错误输出都指向控制终端，进程从标准输入读也就是读用户的键盘输入，进程往标准输出或标准错误输出写也就是输出到显示器上。
* 在控制终端输入一些特殊的控制键可以给前台进程发信号，例如 Ctrl + C 会产生 SIGINT 信号，Ctrl + \ 会产生 SIGQUIT 信号。
* 查看当前终端的进程号 echo $$ 
* 查看当前终端号 tty

43.进程组
* 进程组和会话在进程之间形成了一种两级层次关系：进程组是一组相关进程的集合，会话是一组相关进程组的集合。进程组和会话是为支持 shell 作业控制而定义的抽象概念，用户通过 shell 能够交互式地在前台或后台运行命令。
* 进行组由一个或多个共享同一进程组标识符（PGID）的进程组成。一个进程组拥有一个进程组首进程，该进程是创建该组的进程，其进程 ID 为该进程组的 ID，新进程会继承其父进程所属的进程组 ID。
* 进程组拥有一个生命周期，其开始时间为首进程创建组的时刻，结束时间为最后一个成员进程退出组的时刻。一个进程可能会因为终止而退出进程组，也可能会因为加入了另外一个进程组而退出进程组。进程组首进程无需是最后一个离开进程组的成员。
* 一个进程只能设置自己或者其子进程的PGID。并且，当子进程调用exec系列函数后，我们也不能再在父进程中对它设置PGID。

44.会话
* 会话是一组进程组的集合。会话首进程是创建该新会话的进程，其进程 ID 会成为会话 ID。新进程会继承其父进程的会话 ID。
* 一个会话中的所有进程共享单个控制终端。控制终端会在会话首进程首次打开一个终端设备时被建立。一个终端最多可能会成为一个会话的控制终端。
* 在任一时刻，会话中的其中一个进程组会成为终端的前台进程组，其他进程组会成为后台进程组。只有前台进程组中的进程才能从控制终端中读取输入。当用户在控制终端中输入终端字符生成信号后，该信号会被发送到前台进程组中的所有成员。
* 当控制终端的连接建立起来之后，会话首进程会成为该终端的控制进程。

45.进程组、会话操作函数
* pid_t getpgrp(void);
* pid_t getpgid(pid_t pid);
    - 获取指定进程的PGID。
* int setpgid(pid_t pid, pid_t pgid);
* pid_t getsid(pid_t pid);
* pid_t setsid(void);
    - 该函数不能由进程组的首领进程调用。

46.守护进程
* Linux服务器程序一般会以后台进程形式运。后台进程又称守护进程。它没有控制终端，因而也不会意外接收到用户输入。守护进程的父进程通常是init进程(PID为1的进程)。
* 守护进程（Daemon Process），也就是通常说的Daemon进程（精灵进程），是Linux中的后台服务进程。它是一个生存期较长的进程，通常独立于控制终端并且周期性地执行某种任务或等待处理某些发生的事件。一般采用以 d 结尾的名字。
* 守护进程具备下列特征：
    - 生命周期很长，守护进程会在系统启动的时候被创建并一直运行直至系统被关闭。
    - 它在后台运行并且不拥有控制终端。没有控制终端确保了内核永远不会为守护进程自动生成任何控制信号以及终端相关的信号（如 SIGINT、SIGQUIT）。
* Linux 的大多数服务器就是用守护进程实现的。比如，Internet 服务器 inetd，Web 服务器 httpd 等。

47.守护进程的创建步骤
* 执行一个 fork()，之后父进程退出，子进程继续执行。（为了进程组id不重复、会话id不冲突）
* 子进程调用 setsid() 开启一个新会话。（脱离控制终端）
* 清除进程的 umask 以确保当守护进程创建文件和目录时拥有所需的权限。
* 修改进程的当前工作目录，通常会改为根目录（/）。
* 关闭守护进程从其父进程继承而来的所有打开着的文件描述符。
* 在关闭了文件描述符0、1、2之后，守护进程通常会打开/dev/null 并使用dup2() 使所有这些描述符指向这个设备。
* 核心业务逻辑

```
// 将服务器程序以守护进程的方式运行
bool daemonize()
{
    // 创建子进程，关闭父进程，这样可以使程序在后台运行
    pid_t pid = fork();
    if(pid < 0)
        return false;
    else if(pid > 0)
        exit(0);
        
    // 设置文件权限掩码。
    // 当进程创建新文件（使用open(const char *pathname, int flags, mode_t mode)系统调用）时，文件的权限将是 mode & 0777
    umask(0);
    
    // 创建新的会话，设置本进程为进程组的首领
    pit_t sid = setsid();
    if(sid < 0)
        return false;
    
    // 切换工作目录
    if(chdir("/") < 0)
        return false;
    
    // 关闭标准输入设备、标准输出设备和标准错误输出设备    
    close(STDIN_FILENO);
    close(STDOUT_FILENO);
    close(STDERR_FILENO);
    
    // 关闭其他已经打开的文件描述符，代码省略
    // 将标准输入、标准输出和标准错误输出都定向到/dev/null文件
    open("/dev/null", O_RDONLY);
    open("/dev/null", O_RDWR);
    open("/dev/null", O_RDWR);
    
    return true;
}
```
实际上，Linux提供了完成同样功能的库函数：  
#include <unistd.h>  
int daemon(int nochdir, int noclose);  
其中，nochdir参数用于指定是否改变工作目录。如果给它传递0，则工作目录将被设置为“/”（根目录），否则继续使用当前工作目录。  
noclose参数为0时标准输入、标准输出和标准错误输出都被重定向到/dev/null文件，否则依然使用原来的设备。  

48.用守护进程获取系统时间，写入磁盘文件
```
#include <stdio.h>
#include <sys/stat.h>
#include <sys/types.h>
#include <unistd.h>
#include <fcntl.h>
#include <sys/time.h>
#include <signal.h>
#include <time.h>
#include <stdlib.h>
#include <string.h>

void work(int num)  // 捕捉到信号之后，获取系统时间，写入磁盘文件
{
    time_t tm = time(NULL);
    struct tm * loc = localtime(&tm);
    char * str = asctime(loc);
    int fd = open("time.txt", O_RDWR | O_CREAT | O_APPEND, 0664);
    write(fd ,str, strlen(str));
    close(fd);
}
int main() 
{
    pid_t pid = fork();         // 创建子进程，退出父进程
    if(pid > 0) 
        exit(0);

    setsid();                   // 将子进程重新创建一个会话
    umask(022);                 // 设置掩码
    chdir("/home/bread/");      // 更改工作目录

    int fd = open("/dev/null", O_RDWR); // 关闭、重定向文件描述符
    dup2(fd, STDIN_FILENO);
    dup2(fd, STDOUT_FILENO);
    dup2(fd, STDERR_FILENO);

    // 下面是业务逻辑

    // 捕捉定时信号
    struct sigaction act;
    act.sa_flags = 0;
    act.sa_handler = work;
    sigemptyset(&act.sa_mask);
    sigaction(SIGALRM, &act, NULL);

    struct itimerval val;
    val.it_value.tv_sec = 2;
    val.it_value.tv_usec = 0;
    val.it_interval.tv_sec = 2;
    val.it_interval.tv_usec = 0;
    setitimer(ITIMER_REAL, &val, NULL); // 创建定时器

    while(1)    // 不让进程结束
        sleep(10);
    return 0;
}
```


## 三、Linux多线程开发
1. 线程概述
* 与进程（process）类似，线程（thread）是允许应用程序并发执行多个任务的一种机制。一个进程可以包含多个线程。同一个程序中的所有线程均会独立执行相同程序，且共享同一份全局内存区域，其中包括初始化数据段、未初始化数据段，以及堆内存段。（传统意义上的 UNIX 进程只是多线程程序的一个特例，该进程只包含一个线程）
* 进程是 CPU 分配资源的最小单位，线程是操作系统调度执行的最小单位。
* 线程是轻量级的进程（LWP：Light Weight Process），在 Linux 环境下线程的本质仍是进程。
* 查看指定进程的 LWP 号：ps –Lf pid(进程号)

2.线程和进程区别
* 进程间的信息难以共享。由于除去只读代码段外，父子进程并未共享内存，因此必须采用一些进程间通信方式，在进程间进行信息交换。
* 调用 fork() 来创建进程的代价相对较高，即便利用写时复制技术，仍然需要复制诸如内存页表和文件描述符表之类的多种进程属性，这意味着 fork() 调用在时间上的开销依然不菲。
* 线程之间能够方便、快速地共享信息。只需将数据复制到共享（全局或堆）变量中即可。
* 创建线程比创建进程通常要快 10 倍甚至更多。线程间是共享虚拟地址空间的，无需采用写时复制来复制内存，也无需复制页表。

3.线程之间共享和非共享资源
* 共享资源
    - 进程 ID 和父进程 ID
    - 进程组 ID 和会话 ID
    - 用户 ID 和 用户组 ID
    - 文件描述符表
    - 信号处置
    - 文件系统的相关信息：文件权限掩码（umask）、当前工作目录
    - 虚拟地址空间（除栈、.text代码段）
* 非共享资源
    - 线程 ID
    - 信号掩码-阻塞信号集
    - 线程特有数据
    - error 变量
    - 实时调度策略和优先级
    - 栈，本地变量和函数的调用链接信息

4.查看当前 pthread 库版本：getconf GNU_LIBPTHREAD_VERSION

5.一般情况下,main函数所在的线程我们称之为主线程（main线程），其余创建的线程称之为子线程。   
程序中默认只有一个进程，fork()函数调用后，2个进程。  
程序中默认只有一个线程，pthread_create()函数调用后，2个线程。  

6.线程操作
* 查看帮助文档man ppthread_create
* 编译的时候要加上 -lpthread 链接线程库。
* int pthread_create(pthread_t *thread, const pthread_attr_t *attr, void *(*start_routine) (void *), void *arg);
    - 功能：创建一个子线程
    - thread：传出参数，线程创建成功后，子线程的线程ID被写到该变量中。
    - attr : 设置线程的属性，一般使用默认值，NULL
    - start_routine : 函数指针，这个函数是子线程需要处理的逻辑代码
    - arg : 给第三个参数使用，传参
    - 返回值：  
        成功：0；失败：返回错误号。这个错误号和之前errno不太一样。  
        获取错误号的信息：  char * strerror(int errnum);  
```
void* callback(void* arg)
{
    printf("arg value: %d\n", *(int *)arg);
    return NULL;
}
pthread_t tid;
int num = 10;
int ret = pthread_create(&tid, NULL, callback, (void *)&num);
```

* pthread_t pthread_self(void);
    - 功能：获取当前线程的线程ID

* int pthread_equal(pthread_t t1, pthread_t t2);
    - 功能：比较两个线程ID是否相等。不同的操作系统，pthread_t类型的实现不一样，有的是无符号的长整型，有的是使用结构体去实现的。

* void pthread_exit(void *retval);
    - 功能：终止一个线程，在哪个线程中调用，就表示终止哪个线程
    - retval:需要传递一个指针，作为一个返回值，可以在pthread_join()中获取到。
    - 当主线程退出时，不会影响其他正常运行的线程。比如在main函数return 0之前执行pthread_exit(NULL)退出主线程，则return 0不会被执行。而执行return 0的话，子线程也会退出。

* int pthread_join(pthread_t thread, void **retval);
    - 功能：和一个已经终止的线程进行连接。回收子线程的资源。这个函数是阻塞函数，调用一次只能回收一个子线程。一般在主线程中使用。
    - thread：需要回收的子线程的ID
    - retval: 接收子线程退出时的返回值(注意这个参数是二级指针)
    - 返回值：0 : 成功；非0 : 失败，返回的错误号

* int pthread_detach(pthread_t thread);
    - 功能：分离一个线程。被分离的线程在终止的时候，会自动释放资源返回给系统。
    - thread：需要分离的线程的ID
    - 返回值：成功：0；失败：返回错误号。
    - 不能多次分离，会产生不可预料的行为。
    - 不能去连接(pthread_join)一个已经分离的线程，会报错。

* int pthread_cancel(pthread_t thread);
    - 功能：取消线程（让线程终止）。
    - 取消某个线程，可以终止某个线程的运行，但是并不是立马终止，而是当子线程执行到一个取消点，线程才会终止。
    - 取消点：系统规定好的一些系统调用，我们可以粗略的理解为从用户区到内核区的切换，这个位置称之为取消点。

7.线程属性
* 线程属性类型 pthread_attr_t
* int pthread_attr_init(pthread_attr_t *attr);
    - 初始化线程属性变量
* int pthread_attr_destroy(pthread_attr_t *attr);
    - 释放线程属性的资源
* int pthread_attr_getdetachstate(const pthread_attr_t *attr, int *detachstate);
    - 获取线程分离的状态属性
* int pthread_attr_setdetachstate(pthread_attr_t *attr, int detachstate);
    - 设置线程分离的状态属性
* 可以通过命令查看线程属性的函数 man pthread_attr_两次tab键

8.线程同步
* 线程的主要优势在于，能够通过全局变量来共享信息。不过，这种便捷的共享是有代价的：必须确保多个线程不会同时修改同一变量，或者某一线程不会读取正在由其他线程修改的变量。
* 临界区是指访问某一共享资源的代码片段，并且这段代码的执行应为原子操作，也就是同时访问同一共享资源的其他线程不应终端该片段的执行。
* 线程同步：即当有一个线程在对内存进行操作时，其他线程都不可以对这个内存地址进行操作，直到该线程完成操作，其他线程才能对该内存地址进行操作，而其他线程则处于等待状态。

9.互斥量
* 为避免线程更新共享变量时出现问题，可以使用互斥量（mutex 是 mutual exclusion的缩写）来确保同时仅有一个线程可以访问某项共享资源。可以使用互斥量来保证对任意共享资源的原子访问。
* 互斥量有两种状态：已锁定（locked）和未锁定（unlocked）。任何时候，至多只有一个线程可以锁定该互斥量。试图对已经锁定的某一互斥量再次加锁，将可能阻塞线程或者报错失败，具体取决于加锁时使用的方法。
* 一旦线程锁定互斥量，随即成为该互斥量的所有者，只有所有者才能给互斥量解锁。一般情况下，对每一共享资源（可能由多个相关变量组成）会使用不同的互斥量，每一线程在访问同一资源时将采用如下协议：
    - 针对共享资源锁定互斥量
    - 访问共享资源
    - 对互斥量解锁
* 如果多个线程试图执行这一块代码（一个临界区），事实上只有一个线程能够持有该互斥量（其他线程将遭到阻塞），即同时只有一个线程能够进入这段代码区域，

10.互斥量相关操作函数
* 互斥量的类型 pthread_mutex_t
* int pthread_mutex_init(pthread_mutex_t *restrict mutex, const pthread_mutexattr_t *restrict attr);
    - 初始化互斥量
    - mutex ： 需要初始化的互斥量变量
    - attr ： 互斥量相关的属性，一般用NULL
    - restrict : C语言的修饰符，被修饰的指针，不能由另外的一个指针进行操作。例如：  
            pthread_mutex_t *restrict mutex = xxx;  
            pthread_mutex_t * mutex1 = mutex;   // 用mutex1操作xxx，会报错  
* int pthread_mutex_destroy(pthread_mutex_t *mutex);
    - 释放互斥量的资源
* int pthread_mutex_lock(pthread_mutex_t *mutex);
    - 加锁，阻塞的，如果有一个线程加锁了，那么其他的线程只能阻塞等待
* int pthread_mutex_trylock(pthread_mutex_t *mutex);
    - 尝试加锁，如果加锁失败，不会阻塞，会直接返回。
* int pthread_mutex_unlock(pthread_mutex_t *mutex);
    - 解锁

11.死锁
* 有时，一个线程需要同时访问两个或更多不同的共享资源，而每个资源又都由不同的互斥量管理。当超过一个线程加锁同一组互斥量时，就有可能发生死锁。
* 两个或两个以上的进程在执行过程中，因争夺共享资源而造成的一种互相等待的现象，若无外力作用，它们都将无法推进下去。此时称系统处于死锁状态或系统产生了死锁。
* 死锁的几种场景：
    - 忘记释放锁
    - 重复加锁
    - 多线程多锁，抢占锁资源

12.读写锁
* 当有一个线程已经持有互斥锁时，互斥锁将所有试图进入临界区的线程都阻塞住。但是考虑一种情形，当前持有互斥锁的线程只是要读访问共享资源，而同时有其它几个线程也想读取这个共享资源，但是由于互斥锁的排它性，所有其它线程都无法获取锁，也就无法读访问共享资源了，但是实际上多个线程同时读访问共享资源并不会导致问题。
* 在对数据的读写操作中，更多的是读操作，写操作较少，例如对数据库数据的读写应用。为了满足当前能够允许多个读出，但只允许一个写入的需求，线程提供了读写锁来实现。
* 读写锁的特点：
    - 如果有其它线程读数据，则允许其它线程执行读操作，但不允许写操作。
    - 如果有其它线程写数据，则其它线程都不允许读、写操作。
    - 写是独占的，写的优先级高。

13.读写锁相关操作函数
* 读写锁的类型 pthread_rwlock_t
* int pthread_rwlock_init(pthread_rwlock_t *restrict rwlock, const pthread_rwlockattr_t *restrict attr);
* int pthread_rwlock_destroy(pthread_rwlock_t *rwlock);
* int pthread_rwlock_rdlock(pthread_rwlock_t *rwlock);
* int pthread_rwlock_tryrdlock(pthread_rwlock_t *rwlock);
* int pthread_rwlock_wrlock(pthread_rwlock_t *rwlock);
* int pthread_rwlock_trywrlock(pthread_rwlock_t *rwlock);
* int pthread_rwlock_unlock(pthread_rwlock_t *rwlock);

14.条件变量
* 条件变量的类型 pthread_cond_t
* int pthread_cond_init(pthread_cond_t *restrict cond, const pthread_condattr_t *restrict attr);
* int pthread_cond_destroy(pthread_cond_t *cond);
* int pthread_cond_wait(pthread_cond_t *restrict cond, pthread_mutex_t *restrict mutex);
    - 等待，调用了该函数，线程会阻塞。
    - 当这个函数调用阻塞时，会对互斥锁进行解锁；当不阻塞时，继续向下执行，会重新加锁。
* int pthread_cond_timedwait(pthread_cond_t *restrict cond, pthread_mutex_t *restrict mutex, const struct timespec *restrict abstime);
    - 等待多长时间，调用了这个函数，线程会阻塞，直到指定的时间结束。
* int pthread_cond_signal(pthread_cond_t *cond);
    - 唤醒一个或者多个等待的线程
* int pthread_cond_broadcast(pthread_cond_t *cond);
    - 唤醒所有的等待的线程

15.信号量
* 信号量的类型 sem_t
* int sem_init(sem_t *sem, int pshared, unsigned int value);
    - 初始化信号量
    - sem : 信号量变量的地址
    - pshared : 0 用在线程间 ，非0 用在进程间
    - value : 信号量中的值
* int sem_destroy(sem_t *sem);
    - 释放资源
* int sem_wait(sem_t *sem);
    - 对信号量加锁，调用一次对信号量的值-1，如果值为0，就阻塞
* int sem_trywait(sem_t *sem);
* int sem_timedwait(sem_t *sem, const struct timespec *abs_timeout);
* int sem_post(sem_t *sem);
    - 对信号量解锁，调用一次对信号量的值+1
* int sem_getvalue(sem_t *sem, int *sval);

* 生产者消费者例子
```
sem_t psem;
sem_t csem;
init(psem, 0, 8);
init(csem, 0, 0);

producer() 
{
    sem_wait(&psem);
    sem_post(&csem)
}

customer() 
{
    sem_wait(&csem);
    sem_post(&psem)
}
```


## 四、Linux网络编程

1.以太网的MTU是1500字节，因此过长的IP数据报可能需要被分片传输。  
帧才是最终在物理网络上传送的字节序列。

2.所有知名应用层协议使用的端口号都可在/etc/services文件中找到。

3.Linux下可以使用arp命令来查看和修改ARP高速缓存。
```
sudo arp -a                                         # 查看当前时刻的ARP缓存内容
sudo arp -d 192.168.1.109                           # 删除192.168.1.109对应的ARP缓存项
sudo arp -s 192.168.1.109 08:00:27:53:10:67         # 添加192.168.1.109对应的ARP缓存项
```
ARP通信在TCP连接建立之前就已经完成。

4.Linux使用/etc/resolv.conf文件来存放DNS服务器的IP地址。    
host命令使用DNS协议和DNS服务器通信，其-t选项告诉DNS协议使用哪种查询类型。下面使用A类型，即通过机器的域名获得其IP地址(但实际上返回的资源记录中还包含机器的别名)。
```
host -t A www.baidu.com
```

5.IPv4协议的头部长度通常为20字节，最长为60字节。

6.路由器都能执行数据报的转发操作，而主机一般只发送和接收数据报，这是因为主机上/proc/sys/net/ipv4/ip_forward内核参数默认被设置为0。我们可以通过修改它来使能主机的数据报转发功能。

7.在TCP/IP四层模型中，只有应用层是在用户空间的，其他都是在内核空间的。

8.字节序分为大端字节序（Big-Endian） 和小端字节序（Little-Endian）。大端字节序是指一个整数的最高位字节（23 ~ 31 bit）存储在内存的低地址处，低位字节（0 ~ 7 bit）存储在内存的高地址处；小端字节序则是指整数的高位字节存储在内存的高地址处，而低位字节则存储在内存的低地址处。

9.判断当前主机的字节序
```
#include <stdio.h>
int main() 
{
    union 
    {
        short value;    // 2字节
        char bytes[sizeof(short)];  // char[2]
    } test;

    test.value = 0x0102;
    if((test.bytes[0] == 1) && (test.bytes[1] == 2)) 
        printf("大端字节序\n");
    else if((test.bytes[0] == 2) && (test.bytes[1] == 1)) 
        printf("小端字节序\n");
    else 
        printf("未知\n");
    return 0;
}
```

10.网络字节顺序是 TCP/IP 中规定好的一种数据表示格式，它与具体的 CPU 类型、操作系统等无关，从而可以保证数据在不同主机之间传输时能够被正确解释，网络字节顺序采用大端排序方式。  
BSD Socket提供了封装好的转换接口，方便程序员使用。包括从主机字节序到网络字节序的转换函数：htons、htonl；从网络字节序到主机字节序的转换函数：ntohs、ntohl。
```
// h:host-主机；n:network-网络；s:short-unsigned short；l:long-unsigned int
#include <arpa/inet.h>
// 转换端口
uint16_t htons(uint16_t hostshort); // 主机字节序 - 网络字节序
uint16_t ntohs(uint16_t netshort);  // 网络字节序 - 主机字节序
// 转IP
uint32_t htonl(uint32_t hostlong);  // 主机字节序 - 网络字节序
uint32_t ntohl(uint32_t netlong);   // 网络字节序 - 主机字节序
```

11.socket 网络编程接口中表示 socket 地址的是结构体 sockaddr，其定义如下：
```
// 通用的 socket 地址结构体
#include <bits/socket.h>
struct sockaddr 
{
    sa_family_t sa_family;  // sa_family 成员是地址族类型（sa_family_t）的变量。
    char sa_data[14];       // sa_data 成员用于存放 socket 地址值。
};
typedef unsigned short int sa_family_t;

// 下面是新的通用的 socket 地址结构体，这个结构体不仅提供了足够大的空间用于存放地址值，而且是内存对齐的(这是__ss_align成员的作用)。
#include <bits/socket.h>
struct sockaddr_storage
{
sa_family_t sa_family;
unsigned long int __ss_align;
char __ss_padding[ 128 - sizeof(__ss_align) ];
};
typedef unsigned short int sa_family_t;
```
地址族类型通常与协议族类型对应。常见的协议族（protocol family，也称 domain）和对应的地址族入下所示：

协议族 | 地址族 | 描述
---|---|---
PF_UNIX | AF_UNIX | UNIX本地域协议族
PF_INET | AF_INET | TCP/IPv4协议族
PF_INET6 | AF_INET6 | TCP/IPv6协议族

很多网络编程函数诞生早于 IPv4 协议，那时候都使用的是 struct sockaddr 结构体，为了向前兼容，现在sockaddr 退化成了（void *）的作用，传递一个地址给函数，至于这个函数是 sockaddr_in 还是 sockaddr_in6，由地址族确定，然后函数内部再强制类型转化为所需的地址类型。  
例如 TCP/IP 协议族有 sockaddr_in 和 sockaddr_in6 两个专用的 socket 地址结构体，它们分别用于 IPv4 和 IPv6。  
所有专用 socket 地址（以及 sockaddr_storage）类型的变量在实际使用时都需要转化为通用 socket 地址类型 sockaddr（强制转化即可），因为所有 socket 编程接口使用的地址参数类型都是 sockaddr。

**但是日常开发中，我们一般不会使用通用的 socket 地址结构体。**
```
// UNIX 本地域协议族使用如下专用的 socket 地址结构体：
#include <sys/un.h>
struct sockaddr_un
{
    sa_family_t sin_family;
    char sun_path[108];
};

// TCP/IP 协议族有 sockaddr_in 和 sockaddr_in6 两个专用的 socket 地址结构体，它们分别用于 IPv4 和 IPv6：
#include <netinet/in.h>
struct sockaddr_in
{
    sa_family_t sin_family; /* __SOCKADDR_COMMON(sin_) */
    in_port_t sin_port; /* Port number. */
    struct in_addr sin_addr; /* Internet address. */
    /* Pad to size of `struct sockaddr'. */
    unsigned char sin_zero[sizeof (struct sockaddr) - __SOCKADDR_COMMON_SIZE - sizeof (in_port_t) - sizeof (struct in_addr)];
};

struct in_addr
{
    in_addr_t s_addr;
};

struct sockaddr_in6
{
    sa_family_t sin6_family;
    in_port_t sin6_port; /* Transport layer port # */
    uint32_t sin6_flowinfo; /* IPv6 flow information */
    struct in6_addr sin6_addr; /* IPv6 address */
    uint32_t sin6_scope_id; /* IPv6 scope-id */
};

typedef unsigned short uint16_t;
typedef unsigned int uint32_t;
typedef uint16_t in_port_t;
typedef uint32_t in_addr_t;
#define __SOCKADDR_COMMON_SIZE (sizeof (unsigned short int))
```

12.下面 3 个函数可用于用点分十进制字符串表示的 IPv4 地址和用网络字节序整数表示的 IPv4 地址之间的转换：
```
// 旧的3个转换函数，仅适用于IPv4，不推荐使用.
#include <arpa/inet.h>
in_addr_t inet_addr(const char *cp);
int inet_aton(const char *cp, struct in_addr *inp);
char *inet_ntoa(struct in_addr in);     // 不可重入

// 下面这对更新的函数也能完成前面 3 个函数同样的功能，并且它们同时适用 IPv4 地址和 IPv6 地址：
#include <arpa/inet.h>

// p:点分十进制的IP字符串，n:表示network，网络字节序的整数
int inet_pton(int af, const char *src, void *dst);
    - af:地址族： AF_INET AF_INET6
    - src:需要转换的点分十进制的IP字符串
    - dst:转换后的结果保存在这个里面
    
// 将网络字节序的整数，转换成点分十进制的IP地址字符串
const char *inet_ntop(int af, const void *src, char *dst, socklen_t size);
    - af:地址族： AF_INET AF_INET6
    - src: 要转换的ip的整数的地址
    - dst: 转换成IP地址字符串保存的地方
    - size：第三个参数的大小（数组的大小）
        下面两个宏能帮助我们指定这个大小(分别用于IPv4和Ipv6)：
        #include <netinet/in.h>
        #define INET_ADDRSTRLEN 16
        #define INET6_ADDRSTRLEN 46
    - 返回值：返回转换后的数据的地址（字符串），和 dst 是一样的
```

13.套接字函数
```
#include <sys/types.h>
#include <sys/socket.h>
#include <arpa/inet.h> // 包含了这个头文件，上面两个就可以省略

int socket(int domain, int type, int protocol);
    - 功能：创建一个套接字
    - domain: 协议族
        AF_INET : ipv4
        AF_INET6 : ipv6
        AF_UNIX, AF_LOCAL : 本地套接字通信（进程间通信）
    - type: 通信过程中使用的协议类型
        SOCK_STREAM : 流式协议
        SOCK_DGRAM : 报式协议
    - protocol : 具体的一个协议。一般写0
    - SOCK_STREAM : 流式协议默认使用 TCP
    - SOCK_DGRAM : 报式协议默认使用 UDP
    - 返回值：
        - 成功：返回文件描述符，操作的就是内核缓冲区。
        - 失败：-1
    - 自Linux内核版本2.6.17起，type参数可以接受上述服务类型(SOCK_STREAM或SOCK_DGRAM)与下面两个重要的标记相与的值：SOCK_NONBLOCK和SOCK_CLOEXEC。它们分别表示将新创建的socket设置为非阻塞的，以及用fork调用创建子进程时在子进程中关闭该socket。

int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen); // socket命名
    - 功能：绑定，将 sockfd 和本地的IP + 端口进行绑定
    - sockfd : 通过socket函数得到的文件描述符
    - addr : 需要绑定的socket地址，这个地址封装了ip和端口号的信息
    - addrlen : 第二个参数结构体占的内存大小
    - 返回值：失败返回-1并设置errno。其中两种常见的errno是EACESS和EADDRINUSE。
        EACESS表示被绑定是地址是受保护的地址，仅超级用户能够访问。
        EADDRINUSE表示被绑定的地址正在使用中。

int listen(int sockfd, int backlog); // /proc/sys/net/core/somaxconn
    - 功能：监听这个socket上的连接
    - sockfd : 通过socket()函数得到的文件描述符
    - backlog : 半连接的和完全连接的socket的和的最大值， 一般指定5
        在内核版本2.2之后，它只表示处于完全连接状态的socket的上限，处于半连接状态的socket的上限则由/proc/sys/net/ipv4/tcp_max_syn_backlog内核参数定义。

int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
    - 功能：接收客户端连接，默认是阻塞的，阻塞等待客户端连接
    - sockfd : 用于监听的文件描述符
    - addr : 传出参数，记录了连接成功后客户端的地址信息（ip，port）
    - addrlen : 指定第二个参数的对应的内存大小
    - 返回值：
        - 成功 ：用于通信的文件描述符
        - 失败 ： -1
    - accpet只是在监听队列中取出连接，而不论连接处于何种状态，更不关心任何网络状况的变化。

int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
    - 功能： 客户端连接服务器
    - sockfd : 用于通信的文件描述符
    - addr : 客户端要连接的服务器的地址信息
    - addrlen : 第二个参数的内存大小
    - 返回值：成功 0， 失败 -1。其中两种常见的errno是ECONNREFUSED和ETIMEOUT。
        ECONNREFUSED表示目标端口不存在，连接被拒绝。
        ETIMEOUT表示连接超时。

ssize_t write(int fd, const void *buf, size_t count); // 写数据
ssize_t read(int fd, void *buf, size_t count); // 读数据
```

14.TCP协议
* 16 位端口号（port number）：告知主机报文段是来自哪里（源端口）以及传给哪个上层协议或应用程序（目的端口）的。进行 TCP 通信时，客户端通常使用系统自动选择的临时端口号。

* 32 位序号（sequence number）：一次 TCP 通信（从 TCP 连接建立到断开）过程中某一个传输方向上的字节流的每个字节的编号。假设主机 A 和主机 B 进行 TCP 通信，A 发送给 B 的第一个 TCP 报文段中，序号值被系统初始化为某个随机值 ISN（Initial Sequence Number，初始序号值）。那么在该传输方向上（从 A 到 B），后续的 TCP 报文段中序号值将被系统设置成 ISN 加上该报文段所携带数据的第一个字节在整个字节流中的偏移。例如，某个 TCP 报文段传送的数据是字节流中的第 1025 ~ 2048 字节，那么该报文段的序号值就是 ISN + 1025。另外一个传输方向（从 B 到 A）的 TCP 报文段的序号值也具有相同的含义。

* 32 位确认号（acknowledgement number）：用作对另一方发送来的 TCP 报文段的响应。其值是收到的 TCP 报文段的序号值 + 标志位长度（SYN，FIN） + 数据长度 。假设主机 A 和主机 B 进行 TCP 通信，那么 A 发送出的 TCP 报文段不仅携带自己的序号，而且包含对 B 发送来的 TCP 报文段的确认号。反之，B 发送出的 TCP 报文段也同样携带自己的序号和对 A 发送来的报文段的确认序号。

* 4 位头部长度（head length）：标识该 TCP 头部有多少个 32 bit(4 字节)。因为 4 位最大能表示15，所以 TCP 头部最长是60 字节。

* 6 位标志位包含如下几项：
    - URG 标志，表示紧急指针（urgent pointer）是否有效。
    - ACK 标志，表示确认号是否有效。我们称携带 ACK 标志的 TCP 报文段为确认报文段。
    - PSH 标志，提示接收端应用程序应该立即从 TCP 接收缓冲区中读走数据，为接收后续数据腾出空间（如果应用程序不将接收到的数据读走，它们就会一直停留在 TCP 接收缓冲区中）。
    - RST 标志，表示要求对方重新建立连接。我们称携带 RST 标志的 TCP 报文段为复位报文段。
    - SYN 标志，表示请求建立一个连接。我们称携带 SYN 标志的 TCP 报文段为同步报文段。
    - FIN 标志，表示通知对方本端要关闭连接了。我们称携带 FIN 标志的 TCP 报文段为结束报文段。

* 16 位窗口大小（window size）：是 TCP 流量控制的一个手段。这里说的窗口，指的是接收通告窗口（Receiver Window，RWND）。它告诉对方本端的 TCP 接收缓冲区还能容纳多少字节的数据，这样对方就可以控制发送数据的速度。

* 16 位校验和（TCP checksum）：由发送端填充，接收端对 TCP 报文段执行 CRC 算法以校验 TCP 报文段在传输过程中是否损坏。注意，这个校验不仅包括 TCP 头部，也包括数据部分。这也是 TCP 可靠传输的一个重要保障。

* 16 位紧急指针（urgent pointer）：是一个正的偏移量。它和序号字段的值相加表示最后一个紧急数据的下一个字节的序号。因此，确切地说，这个字段是紧急指针相对当前序号的偏移，不妨称之为紧急偏移。TCP 的紧急指针是发送端向接收端发送紧急数据的方法。


15.TCP三次握手

三次握手的目的是保证双方互相之间建立了连接。(保证自己和对方的收发都是没有问题的)  
三次握手发生在客户端连接的时候，当调用connect()，底层会通过TCP协议进行三次握手。  
SYN=1的报文是不能携带数据的，也就是前两次握手是带不了数据的。

* 第一次握手：
    - 客户端将SYN标志位置为1。
    - 生成一个随机的32位的序号seq=J。
* 第二次握手：
    - 服务器端接收客户端的连接： ACK=1
    - 服务器会回发一个确认序号： ack=客户端的序号J + 1
    - 服务器端会向客户端发起连接请求： SYN=1
    - 服务器会生成一个随机序号：seq = K
* 第三次握手：
    - 客户单应答服务器的连接请求：ACK=1
    - 客户端回复收到了服务器端的数据：ack=服务端的序号K + 1

16. TCP四次挥手

四次挥手发生在断开连接的时候，在程序中当调用了close()会使用TCP协议进行四次挥手。  
客户端和服务器端都可以主动发起断开连接，谁先调用close()谁就是发起。  
因为在TCP连接的时候，采用三次握手建立的的连接是双向的，在断开的时候需要双向断开。  

* 第一次挥手(客户端调用close())
    - 客户端将FIN标志位置为1
    - 发送seq=客户端的序号J+1，ack=服务端的序号K+1
* 第二次挥手
    - 服务端发送ack=客户端的序号J+2
* 第三次挥手(服务端调用close())
    - 服务端将FIN标志位置为1
    - 服务端发送seq=服务端的序号K+1
* 第四次挥手
    - 客户端发送ack=服务端的序号K+2
    - 客户端发送完就马上进入TIME_WAIT阶段，时间长度为2MSL。

* 2MSL（Maximum Segment Lifetime，最大报文生存时间）  
    - 主动断开连接的一方, 最后进入一个 TIME_WAIT状态, 这个状态会持续2msl  
    - msl: 官方建议: 2分钟, 实际是30s  
        当 TCP 连接主动关闭方接收到被动关闭方发送的 FIN 和最终的 ACK 后，连接的主动关闭方必须处于TIME_WAIT 状态并持续 2MSL 时间。  
        这样就能够让 TCP 连接的主动关闭方在它发送的 ACK 丢失的情况下重新发送最终的 ACK。主动关闭方重新发送的最终 ACK 并不是因为被动关闭方重传了 ACK（它们并不消耗序列号，被动关闭方也不会重传），而是因为被动关闭方重传了它的 FIN。事实上，被动关闭方总是重传 FIN 直到它收到一个最终的 ACK。
* 半关闭
    - 当 TCP 链接中 A 向 B 发送 FIN 请求关闭，另一端 B 回应 ACK 之后（A 端进入 FIN_WAIT_2 状态），并没有立即发送 FIN 给 A，A 方处于半连接状态（半开关），此时 A 可以接收 B 发送的数据，但是 A 已经不能再向 B 发送数据。

* 从程序的角度，可以使用 API 来控制实现半连接状态：
```
#include <sys/socket.h>
int shutdown(int sockfd, int how);
    - sockfd: 需要关闭的socket的描述符
    - how: 允许为shutdown操作选择以下几种方式:
        SHUT_RD(0)： 关闭sockfd上的读功能，此选项将不允许sockfd进行读操作。该套接字不再接收数据，任何当前在套接字接受缓冲区的数据将被无声的丢弃掉。
        SHUT_WR(1): 关闭sockfd的写功能，此选项将不允许sockfd进行写操作。进程不能在对此套接字发出写操作。sockfd的发送缓冲区中的数据会在真正关闭连接之前全部发送出去。这种情况下，连接处于半关闭状态。
        SHUT_RDWR(2):关闭sockfd的读写功能。相当于调用shutdown两次：首先是以SHUT_RD,然后以SHUT_WR。
```

* 使用 close 中止一个连接，但它只是减少描述符的引用计数，并不直接关闭连接，只有当描述符的引用计数为 0 时才关闭连接。多进程程序中，一次fork系统调用默认将使父进程中打开的socket的引用计数加1，因此我们必须在父进程和子进程中都对该socket执行close调用才能将连接关闭。
* shutdown 不考虑描述符的引用计数，直接关闭描述符。也可选择中止一个方向的连接，只中止读或只中止写。
* 注意:
    - 如果有多个进程共享一个套接字，close 每被调用一次，计数减 1 ，直到计数为 0 时，也就是所用进程都调用了 close，套接字将被释放。
    - 在多进程中如果一个进程调用了 shutdown(sfd, SHUT_RDWR) 后，其它的进程将无法进行通信。但如果一个进程 close(sfd) 将不会影响到其它进程。

17.端口复用

端口复用最常用的用途是:
    - 防止服务器重启时之前绑定的端口还未释放
    - 程序突然退出而系统没有释放端口
```
#include <sys/types.h>
#include <sys/socket.h>
// 设置套接字的属性（不仅仅能设置端口复用）
int setsockopt(int sockfd, int level, int optname, const void *optval, socklen_t optlen);
    参数：
    - sockfd : 要操作的文件描述符
    - level : 级别 - SOL_SOCKET (端口复用的级别)
    - optname : 选项的名称
        SO_REUSEADDR
        SO_REUSEPORT
    - optval : 端口复用的值（整形）
        1 : 可以复用
        0 : 不可以复用
    - optlen : optval参数的大小

// 端口复用，设置的时机是在服务器绑定端口之前。
int optval = 1;
setsockopt(sockfd, SOL_SOCKET, SO_REUSEPORT, &optval, sizeof(optval);
bind();
```
此外，我们也可以通过修改内核参数/proc/sys/net/ipv4/tcp_tw_recycle来快速回收被关闭的socket，从而使得TCP连接根本就不进入TIME_WAIT状态，进而允许应用程序立即重用本地的socket地址。

18.TCP提供了异常终止一个连接的方法，即给对方发送一个复位报文段。一旦发送了复位报文段，发送端所有排队等待发送的数据都将被丢弃。应用程序可以使用socket选项SO_LINGER来发送复位报文段，以异常终止一个连接。

如果客户端(或服务端)往处于半打开状态的连接写入数据，则对方将回应一个复位报文段。

19.Nagle算法要求一个TCP连接的通信双方在任意时刻都最多只能发生一个未被确认的TCP报文段，在该TCP报文段的确认到达之前不能发生其他TCP报文段。

20.iperf是一个测量网络状况的工具，-s选项表示将其作为服务器运行。iperf默认监听5001端口，并丢弃该端口上接收到的所有数据，相当于一个discard服务器。

21.service是一个脚本程序(/usr/sbin/service)，它为/etc/init.d目录下的众多服务器程序(比如，httpd、vsftpd、sshd、mysqld等)的启动(start)、停止(stop)和重启(restart)等动作提供了统一的管理。

22.UDP 通信
```
#include <sys/types.h>
#include <sys/socket.h>
ssize_t sendto(int sockfd, const void *buf, size_t len, int flags, const struct sockaddr *dest_addr, socklen_t addrlen);
    - sockfd : 通信的fd
    - buf : 要发送的数据
    - len : 发送数据的长度
    - flags : 0
    - dest_addr : 通信的另外一端的地址信息
    - addrlen : 地址的内存大小

ssize_t recvfrom(int sockfd, void *buf, size_t len, int flags, struct sockaddr *src_addr, socklen_t *addrlen);
    - sockfd : 通信的fd
    - buf : 接收数据的数组
    - len : 数组的大小
    - flags : 0
    - src_addr : 用来保存另外一端的地址信息，不需要可以指定为NULL
    - addrlen : 地址的内存大小
```
值得一提的是，recvfrom/sendto系统调用也可以用于面向连接(STREAM)的socket的数据读写，只需要把最后两个参数都设置为NULL以忽略发送端/接收端的socket地址(因为我们已经和对方建立了连接，所以已经知道其socket地址了)。

23.通用数据读写函数

socket编程接口还提供了一对通用的数据读写系统调用。它们不仅能用于TCP流数据，也能用于UDP数据报：
```
#include <sys/socket.h>
ssize_t recvmsg(int sockfd, struct msghdr* msg, int flags);
ssize_t sendmsg(int sockfd, struct msghdr* msg, int flags);

struct msggdr
{
    void *msg_name;             // socket地址
    socklen_t msg_namelen;      // socket地址的长度
    struct iovec* msg_iov;      // 分散的内存块
    int msg_iovlen;             // 分散内存块的数量
    void* msg_control;          // 指向辅助数据的起始位置
    socklen_t msg_controllen;   // 辅助数据的大小
    int msg_flags;              // 赋值函数中的flags参数，并在调用过程中更新
}

// msg_name成员指向一个socket地址结构变量。它指定通信对方的socket地址。对于面向连接的TCP协议，该成员没有意义，必须被设置为NULL。
// 对于recvmsg而言，数据将被读取并存放在msg_iovlen块分散的内存中，这些内存的位置和长度则由msg_iov指向的数组指定，这称为分散读；对于sendmsg而言，msg_iovlen块分散内存中的数据将被一并发送，这称为集中写。

struct iovec
{
    void *iov_base;     // 内存起始地址
    size_t iov_len;     // 这块内存的长度
}
```

24.带外标记

Linux内核检测到TCP紧急标志时，将通知应用程序有带外数据需要接受。内核通知应用程序带外数据到达的两种常见方式是：I/O复用产生的异常事件和SIGURG信号。但是，即使应用程序得到了有带外数据需要接收的通知，还需要知道带外数据在数据流中的具体位置，才能准确接收带外数据。这一点可通过如下系统调用实现:
```
#include <sys/socket.h>
int sockatmark(int sockfd)
    - sockatmark判断sockfd是否处于带外标记，即下一个被读取到的数据是否是带外数据。如果是，sockatmark返回1，此时我们就可以利用MSG_OOB标志的recv调用来接收带外数据。如果不是，则sockatmark返回0。
int ret = recv(connfd, buffer, BUF_SIZE-1， MSG_OOB);
```

25、地址信息函数
```
#include <sys/socket.h>
// 获取sockfd对应的本端socket地址
int getsockname(int sockfd, struct sockaddr* address, socklen_t* address_len);

// 获取sockfd对应的远端socket地址
int getpeername(int sockfd, struct sockaddr* address, socklen_t* address_len);
```

26.广播

向子网中多台计算机发送消息，并且子网中所有的计算机都可以接收到发送方发送的消息，每个广播消息都包含一个特殊的IP地址，这个IP中子网内主机标志部分的二进制全部为1。
    - 只能在局域网中使用。
    - 客户端需要绑定服务器广播使用的端口，才可以接收到广播消息。
```
// 下面两个系统调用是专门用来读取和设置socket文件描述符属性的方法。
#include <sys/socket.h>    
int setsockopt(int sockfd, int level, int optname, const void *optval, socklen_t optlen);
int getsockopt(int sockfd, int level, int optname, void *option_value, socklen_t* restrict option_len);

// 例如，设置广播属性的函数
int setsockopt(int sockfd, int level, int optname,const void *optval, socklen_t optlen);
    - sockfd : 文件描述符
    - level : SOL_SOCKET
    - optname : SO_BROADCAST
    - optval : int类型的值，为1表示允许广播
    - optlen : optval的大小
```
值得指出的是，对服务器而言，有部分socket选项只能在调用listen系统调用前针对监听socket设置才有效。这是因为连接socket只能有accept调用返回，而accept从listen监听队列中接收的连接至少已经完成了TCP三次握手的前两个步骤(因为listen监听队列中的连接至少已进入SYN_RCVD状态)，这说明服务器已经往被接受的连接上发送出了TCP同步报文段。

对于这种情况，Linux给开发人员提供的解决方案是：对监听socket这些socket选项，那么accpet返回的连接socket将自动继承这些选项。这些socket选项包括：SO_DEBUG、SO_DONTROUTE、SO_KEEPALIVE、SO_LINGER、SO_OOBINLINE、SO_RCVBUF、SO_RCVLOWAT、SO_SNDBUF、SO_SNDLOWAT、TCP_MAXSEG和TCP_NODELAY。

而对客户端而言，这些socket选项则应该在调用connect函数之前设置，因为connect调用成功返回之后，TCP三次握手已经完成。

SO_RCVBUF和SO_SNDBUF选项分别表示TCP接收缓存区和发送缓存区的大小。不过，当我们用setsockopt来设置TCP的接收缓存区和发送缓冲区的大小时，系统都会将其值加倍，并且不得小于某个最小值。

SO_RCVLOWAT和SO_SNDLOWAT选项分别表示TCP接收缓冲区和发送缓冲区的低水位标记。它们一般被I/O复用系统调用用来判断socket是否可读或可写。默认都是1.

SO_LINGER选项用来控制close系统调用在关闭TCP连接时的行为。默认情况下，当我们使用close系统调用来关闭一个socket时，close将立即返回，TCP模块负责把该socket对应的TCP发送缓冲区中残留的数据发送给对方。

27.组播(多播)

* 单播地址标识单个 IP 接口，广播地址标识某个子网的所有 IP 接口，多播地址标识一组 IP 接口。
单播和广播是寻址方案的两个极端（要么单个要么全部），多播则意在两者之间提供一种折中方案。多播数据报只应该由对它感兴趣的接口接收，也就是说由运行相应多播会话应用系统的主机上的接口接收。另外，广播一般局限于局域网内使用，而多播则既可以用于局域网，也可以跨广域网使用。
    - 组播既可以用于局域网，也可以用于广域网
    - 客户端需要加入多播组，才能接收到多播的数据

* 组播地址
    - IP 多播通信必须依赖于 IP 多播地址，在 IPv4 中它的范围从 224.0.0.0 到 239.255.255.255 ，并被划分为局部链接多播地址、预留多播地址和管理权限多播地址三类:

IP地址 | 说明
---|---
224.0.0.0~224.0.0.255 | 局部链接多播地址：是为路由协议和其它用途保留的地址，路由器并不转发属于此范围的IP包
224.0.1.0~224.0.1.255 | 预留多播地址：公用组播地址，可用于Internet；使用前需要申请
224.0.2.0~238.255.255.255 | 预留多播地址：用户可用组播地址(临时组地址)，全网范围内有效
239.0.0.0~239.255.255.255 | 本地管理组播地址，可供组织内部使用，类似于私有 IP 地址，不能用于 Internet，可限制多播范围

* 设置组播
```
int setsockopt(int sockfd, int level, int optname,const void *optval, socklen_t optlen);
    // 服务器设置多播的信息，外出接口
    - level : IPPROTO_IP
    - optname : IP_MULTICAST_IF
    - optval : struct in_addr
    
    // 客户端加入到多播组：
    - level : IPPROTO_IP
    - optname : IP_ADD_MEMBERSHIP
    - optval : struct ip_mreq

struct ip_mreq
{
    /* IP multicast address of group. */
    struct in_addr imr_multiaddr; // 组播的IP地址

    /* Local IP address of interface. */
    struct in_addr imr_interface; // 本地的IP地址
};

typedef uint32_t in_addr_t;
struct in_addr
{
    in_addr_t s_addr;
};
```

28.本地套接字
* 本地套接字的作用：本地的进程间通信
    - 有关系的进程间的通信
    - 没有关系的进程间的通信

本地套接字实现流程和网络套接字类似，一般采用TCP的通信流程。
```
// 本地套接字通信的流程 - tcp
// 服务器端
1. 创建监听的套接字
    int lfd = socket(AF_UNIX或AF_LOCAL, SOCK_STREAM, 0);
2. 监听套接字绑定本地的套接字文件 -> server端
    struct sockaddr_un addr;
    // 绑定成功之后，指定的sun_path中的套接字文件会自动生成。
    bind(lfd, addr, len);
3. 监听
    listen(lfd, 100);
4. 等待并接受连接请求
    struct sockaddr_un cliaddr;
    int cfd = accept(lfd, &cliaddr, len);
5. 通信
    接收数据：read/recv
    发送数据：write/send
6. 关闭连接
    close();

// 客户端的流程
1. 创建通信的套接字
    int fd = socket(AF_UNIX/AF_LOCAL, SOCK_STREAM, 0);
2. 监听的套接字绑定本地的IP 端口
    struct sockaddr_un addr;
    // 绑定成功之后，指定的sun_path中的套接字文件会自动生成。
    bind(lfd, addr, len);
3. 连接服务器
    struct sockaddr_un serveraddr;
    connect(fd, &serveraddr, sizeof(serveraddr));
4. 通信
    接收数据：read/recv
    发送数据：write/send
5. 关闭连接
    close();
```
```
// 头文件: sys/un.h
#define UNIX_PATH_MAX 108
struct sockaddr_un {
    sa_family_t sun_family;         // 地址族协议 af_local
    char sun_path[UNIX_PATH_MAX];   // 套接字文件的路径, 这是一个伪文件, 大小永远=0
}
```
本地套接字使用的伪文件相当于就是tcp里面的端口，因此也会出现程序重启后有一段时间不能复用端口的情况，我们可以使用在程序启动后删除该文件的做法，也可以使用unlink函数(从文件系统中删除一个名字)。例如，unlink("server.sock");

29.查看网络相关信息的命令
* netstat  
参数：   
        -a 所有的socket  
        -p 显示正在使用socket的程序的名称  
        -n 直接使用IP地址，而不通过域名服务器  
        -l 显示正在监听的socket  
        -t 显示TCP的socket  
        -u 显示UDP的socket  

30.网络信息API
```
// gethostbyname函数根据主机名称获取主机的完整信息，gethostbyaddr函数根据IP地址获取主机的完整信息。
// gethostbyname函数通常先在本地的/etc/hosts配置文件中查找主机，如果没有找到，再去访问DNS服务器。
#include <netdb.h>
struct hostent* gethostbyname(const char* name); // 不可重入、非线程安全的
    - name是目标主机的主机名
    
struct hostent* gethostbyaddr(const void* addr, size_t len, int type); // 不可重入、非线程安全的
    - addr是目标主机的IP地址
    - len是addr的长度
    - type是addr所指IP地址的类型，填AF_INET或AF_INET6.

struct hostent
{
    char* h_name;       // 主机名
    char** h_aliases;   // 主机别名列表，可能有多个
    int h_addrtype;     // 地址类型(地址族)
    int h_length;       // 地址长度
    char** h_addr_list; // 按网络字节序列出的主机IP地址列表
}
```

```
// getservbyname函数根据名称获取某个服务的完整信息，getservbyport函数根据端口号获取某个服务的完整信息。它们实际上都是通过读取/etc/services文件来获取服务的信息的。
#include <netdb.h>
struct servent* getservbyname(const char* name, const char* proto); // 不可重入、非线程安全的
    - name是指定服务的名字
    - proto参数指定服务类型，例如"tcp"、"udp"、"NULL"(所有类型)

struct servent* getservbyport(int port, const char* proto); // 不可重入、非线程安全的
    - port是指定服务对应的端口号
    
struct servent
{
    char* s_name;       // 服务名
    char** s_aliases;   // 服务的别名列表，可能有多个
    int s_port;         // 端口号
    char* s_proto;      // 服务类型，通常是tcp或者udp
}
```

**Linux下所有其他函数的可重入版本的命名规则，都是在原函数名尾部加上_r（re-entrant）。**

```
// getaddrinfo函数既能通过主机名获得ip地址(内部使用的是gethostbyname函数)，也能通过服务名获得端口号(内部使用的是getservbyname函数)。它是否可重入取决于其内部调用的gethostbyname和getservbyname函数是否是它的可重入版本。
#include <netdb.h>
int getaddrinfo(const char* hostname, const char* service, const struct addrinfo* hints, struct addrinfo** result);
    - hostname可以接收主机名，也可以接收字符串表示的IP地址(IPv4采用点分十进制字符串，IPv6则采用十六进制字符串)。
    - service可以接收服务名，也可以接收字符串表示的十进制端口号。
    - hints是应用程序给getaddrinfo的一个提示，以对getaddrinfo的输出进行更精确的控制。hints可以设置为NULL，表示允许getaddrinfo反馈任何可用的结果。
    - result参数指向一个链表，该链表用于存储getaddrinfo反馈的结果。
    - getaddrinfo反馈的每一条结果都是addrinfo结构体类型的对象。
    - getaddrinfo将对result隐式地分配堆内存，因此getaddrinfo调用结束后，必须使用如下配对函数来释放这块内存：void freeaddrinfo(struct addrinfo* res);

sturct addrinfo
{
    int ai_flags;
    int ai_family;                  // 地址族
    int ai_socktype;                // 服务类型，SOCK_STREAM或SOCK_DGRAM
    int ai_protocol;                // 具体的网络协议，其含义和socket系统调用的第三个参数相同，通常被设置为0
    socketlen_t ai_addrlen;         // socket地址ai_addr的长度
    char* ai_canonname;             // 主机的别名
    struct sockaddr* ai_addr;       // 指向socket地址
    struct addrinfo* ai_next;       // 指向下一个sockinfo结构的对象
}
```

```
// getnameinfo函数能通过socket地址同时获得以字符串表示的主机名(内部使用的是gethostbyaddr函数)和服务名(内部使用的是getservbyport函数)。它是否可重入取决于其内部调用的gethostbyaddr和getservbyport函数是否是它的可重入版本。
#include <netdb.h>
int getnameinfo(const struct sockaddr* sockaddr, socklen_t addrlen, char* host, socklen_t hostlen, char* serv, socklen_t servlen, int flags);
```

**Linux下strerror函数能将数值错误码errno转换成易读的字符串形式**

31.socket的基础API中有一个sockpair函数。它能够方便地创建双向管道。其定义如下：
```
#include <sys/types.h>
#include <sys/socket.h>
int socketpair(int domain, int type, int protocol, int fd[2]);
    - domain只能使用UNIX本地域协议族AF_UNIX，因为我们仅能在本地使用这个双向管道。
    - 创建出来的这对文件描述符都是即可读又可写的。
```


## 五、I/O 多路复用

1.I/O 多路复用使得程序能同时监听多个文件描述符，能够提高程序的性能，Linux 下实现 I/O 多路复用的系统调用主要有 select、poll 和 epoll。

2.select
* 主旨思想：
    - 首先要构造一个关于文件描述符的列表，将要监听的文件描述符添加到该列表中。
    - 调用一个系统函数，监听该列表中的文件描述符，直到这些描述符中的一个或者多个进行I/O操作时，该函数才返回。  
        -这个函数是阻塞  
        -函数对文件描述符的检测的操作是由内核完成的  
    - 在返回时，它会告诉进程有多少描述符要进行I/O操作。(不会告诉你是哪几个,需要你去遍历)
```
// sizeof(fd_set) = 128 ，也就是1024位
#include <sys/time.h>
#include <sys/types.h>
#include <unistd.h>
#include <sys/select.h>
int select(int nfds, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);
    - nfds : 委托内核检测的最大文件描述符的值 + 1
    - readfds : 要检测的文件描述符的读的集合，委托内核检测哪些文件描述符的读的属性
        -一般检测读操作
        -对应的是对方发送过来的数据，因为读是被动的接收数据，检测的就是读缓冲区
        -是一个传入传出参数
    - writefds : 要检测的文件描述符的写的集合，委托内核检测哪些文件描述符的写的属性
        -委托内核检测写缓冲区是不是还可以写数据（不满的就可以写）
    - exceptfds : 检测发生异常的文件描述符的集合
    - timeout : 设置的超时时间
        struct timeval {
            long tv_sec;    /* seconds */
            long tv_usec;   /* microseconds */
        };
        -NULL : 永久阻塞，直到检测到了文件描述符有变化
        -tv_sec = 0 && tv_usec = 0， 不阻塞
        -tv_sec > 0    tv_usec > 0， 阻塞对应的时间
    - 返回值 :
        -1 : 失败
        >0(n) : 检测的集合中有n个文件描述符发生了变化

// 将参数文件描述符fd对应的标志位设置为0
void FD_CLR(int fd, fd_set *set);

// 判断fd对应的标志位是0还是1， 返回值 ： fd对应的标志位的值，0，返回0； 1，返回1
int FD_ISSET(int fd, fd_set *set);

// 将参数文件描述符fd 对应的标志位，设置为1
void FD_SET(int fd, fd_set *set);

// fd_set一共有1024 bit, 全部初始化为0
void FD_ZERO(fd_set *set);
```

* select的缺点：
    - 每次调用select，都需要把fd集合从用户态拷贝到内核态，这个开销在fd很多时会很大。
    - 同时每次调用select都需要在内核遍历传递进来的所有fd，这个开销在fd很多时也很大。
    - select支持的文件描述符数量太小了，默认是1024。(poll改进这个缺点)
    - fds集合不能重用，每次都需要重置。(poll改进这个缺点)

3.poll
```
#include <poll.h>
struct pollfd {
    int fd;         /* 委托内核检测的文件描述符 */
    short events;   /* 委托内核检测文件描述符的什么事件 */
    short revents;  /* 文件描述符实际发生的事件 */
};

struct pollfd myfd;
myfd.fd = 5;
myfd.events = POLLIN | POLLOUT;     // 同时检测读写

int poll(struct pollfd *fds, nfds_t nfds, int timeout);
    - fds : 是一个struct pollfd 结构体数组，这是一个需要检测的文件描述符的集合
    - nfds : 这个是第一个参数数组中最后一个有效元素的下标 + 1
    - timeout : 阻塞时长
        0 : 不阻塞
        -1 : 阻塞，当检测到需要检测的文件描述符有变化，解除阻塞
        >0 : 阻塞的时长，单位ms
    - 返回值：
        -1 : 失败
        >0（n） : 成功，n表示检测到集合中有n个文件描述符发生变化
```

4.epoll
```
#include <sys/epoll.h>

// 创建一个新的epoll实例。在内核中创建了一个数据，这个数据中有两个比较重要的数据，一个是需要检测的文件描述符的信息（红黑树），还有一个是就绪列表，存放检测到数据发送改变的文件描述符信息（双向链表）。
int epoll_create(int size);
    - size : Linux 2.6.8之后，这个值没有意义了。随便写一个数即可，但必须大于0
    - 返回值：
        -1 : 失败
        >0 : 文件描述符，用来操作epoll实例的
    - 程序最后使用close(epfd)进行关闭

typedef union epoll_data {  // 是个union
    void *ptr;
    int fd;
    uint32_t u32;
    uint64_t u64;
} epoll_data_t;

struct epoll_event {
    uint32_t events;    /* Epoll events */
    epoll_data_t data;  /* User data variable */
};

常见的Epoll检测事件：
- EPOLLIN
- EPOLLOUT
- EPOLLERR

// 对epoll实例进行管理：添加文件描述符信息，删除信息，修改信息
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
    - epfd : epoll实例对应的文件描述符
    - op : 要进行什么操作
        EPOLL_CTL_ADD: 添加
        EPOLL_CTL_MOD: 修改
        EPOLL_CTL_DEL: 删除
    - fd : 要检测的文件描述符
    - event : 检测文件描述符什么事情

// 检测函数
int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout);
    - epfd : epoll实例对应的文件描述符
    - events : 传出参数，保存了发生了变化的文件描述符的信息
    - maxevents : 第二个参数结构体数组的大小
    - timeout : 阻塞时间
        0 : 不阻塞
        -1 : 阻塞，直到检测到fd数据发生变化，解除阻塞
        >0 : 阻塞的时长（毫秒）
    - 返回值：
        -成功，返回发送变化的文件描述符的个数，即 > 0
        -失败 -1
```

5.Epoll 的工作模式
* LT 模式 （水平触发、默认工作模式）  
    - 假设委托内核检测读事件 -> 检测fd的读缓冲区  
    读缓冲区有数据 - > epoll检测到了会给用户通知  
        - 用户不读数据，数据一直在缓冲区，epoll 会一直通知  
        - 用户只读了一部分数据，epoll会通知  
        - 缓冲区的数据读完了，不通知  
    - LT（level - triggered）是缺省的工作方式，并且同时支持 block 和 no-block socket。在这种做法中，内核告诉你一个文件描述符是否就绪了，然后你可以对这个就绪的 fd 进行 IO 操作。如果你不作任何操作，内核还是会继续通知你的。

* ET 模式（边沿触发）
    - 假设委托内核检测读事件 -> 检测fd的读缓冲区  
    读缓冲区有数据 - > epoll检测到了会给用户通知  
        - 用户不读数据，数据一致在缓冲区中，epoll下次检测的时候就不通知了
        - 用户只读了一部分数据，epoll不通知
        - 缓冲区的数据读完了，不通知
    - ET（edge - triggered）是高速工作方式，只支持 no-block socket。在这种模式下，当描述符从未就绪变为就绪时，内核通过epoll告诉你。然后它会假设你知道文件描述符已经就绪，并且不会再为那个文件描述符发送更多的就绪通知，直到你做了某些操作导致那个文件描述符不再为就绪状态了。但是请注意，如果一直不对这个 fd 作 IO 操作（从而导致它再次变成未就绪），内核不会发送更多的通知（only once）。
    - ET 模式在很大程度上减少了 epoll 事件被重复触发的次数，因此效率要比 LT 模式高。
    - epoll工作在 ET 模式的时候，必须使用非阻塞套接口，以避免由于一个文件句柄的阻塞读/阻塞写操作把处理多个文件描述符的任务饿死。

```
struct epoll_event {
    uint32_t events; /* Epoll events */
    epoll_data_t data; /* User data variable */
};

Epoll ET事件：
- EPOLLET
```

6.select、poll和epoll的区别

系统调用 | select | poll | epoll
---|---|---|---
事件集合 | 用户通过3个参数分别传入感兴趣的可读、可写及异常等事件，内核通过对这些参数的在线修改来反馈其中的就绪事件。这使得用户每次调用select都要重置这3个参数 | 统一处理所有事件类型，因此只需一个事件集参数。用户通过pollfd.events传入感兴趣的事件，内核通过修改pollfd.revents反馈其中就绪的事件 | 内核通过一个事件表直接管理用户感兴趣的所有事件。因此每次调用epoll_wait时，无须反复传入用户感兴趣的事件。epoll_wait系统调用的参数events仅用来反馈就绪的事件
应用程序索引就绪文件描述符的时间复杂度 | O(n) | O(n) | O(1)
最大支持文件描述符数 | 一般有最大值限制 | 65535 | 65535
工作模式 | LT | LT | 支持ET高效模式
内核时间和工作效率 | 采用轮询方式来检测就绪事件，算法时间复杂度为O(n) | 采用轮询方式来检测就绪事件，算法时间复杂度为O(n) | 采用回调方式来检测就绪事件，算法时间复杂度为O(1)

7.IO复用的应用之一：非阻塞connect

connect出错时的一种errno值：EINPROGRESS。这种错误发生在对非阻塞的socket调用connect，而连接又没有立即建立时。这种情况下，我们可以调用select、poll等函数来监听这个连接失败的socket上的可写事件。当select、poll等函数返回后，再利用getsockopt来读取错误码并清除该socket上的错误。如果错误码为0，表示连接成功建立，否则连接失败。


## 六、项目实战与总结  
1.阻塞/非阻塞、同步/异步
* 数据就绪：根据系统IO操作的就绪状态
    - 阻塞
    - 非阻塞
* 数据读写：根据应用程序和内核的交互方式
    - 同步
    - 异步

陈硕：在处理 IO 的时候，阻塞和非阻塞都是同步 IO，只有使用了特殊的 API 才是异步 IO。
(因此,select、poll、epoll都是同步IO)

一个典型的网络IO接口调用，分为两个阶段，分别是“数据就绪” 和“数据读写”，数据就绪阶段分为阻塞和非阻塞，表现得结果就是，阻塞当前线程或是直接返回。

同步表示A向B请求调用一个网络IO接口时（或者调用某个业务逻辑API接口时），数据的读写都是由请求方A自己来完成的（不管是阻塞还是非阻塞）；异步表示A向B请求调用一个网络IO接口时（或者调用某个业务逻辑API接口时），向B传入请求的事件以及事件发生时通知的方式，A就可以处理其它逻辑了，当B监听到事件处理完成后，会用事先约定好的通知方式，通知A处理结果。

2.Unix/Linux上的五种IO模型
* 阻塞 blocking
    - 调用者调用了某个函数，等待这个函数返回，期间什么也不做，不停的去检查这个函数有没有返回(系统内部去做的)，必须等这个函数返回才能进行下一步动作。

* 非阻塞 non-blocking（NIO）
    - 非阻塞等待，每隔一段时间就去检测IO事件是否就绪。没有就绪就可以做其他事。非阻塞I/O执行系统调用总是立即返回，不管事件是否已经发生，若事件没有发生，则返回-1，此时可以根据 errno 区分这两种情况，对于accept，recv 和 send，事件未发生时，errno 通常被设置成 EAGAIN。

* IO复用（IO multiplexing）
    - Linux 用 select/poll/epoll 函数实现 IO 复用模型，这些函数也会使进程阻塞(可以通过timeout参数变成非阻塞的)，但是和阻塞IO所不同的是这些函数可以同时阻塞多个IO操作。而且可以同时对多个读操作、写操作的IO函数进行检测。直到有数据可读或可写时，才真正调用IO操作函数。

* 信号驱动（signal-driven）
    - Linux 用套接口进行信号驱动 IO，安装一个信号处理函数，进程继续运行并不阻塞，当IO事件就绪，进程收到SIGIO 信号，然后处理 IO 事件。
    - 内核在第一个阶段(数据就绪)是异步，在第二个阶段(数据读写)是同步；与非阻塞IO的区别在于它提供了消息通知机制，不需要用户进程不断的轮询检查，减少了系统API的调用次数，提高了效率。 

* 异步（asynchronous）
    - Linux中，可以调用 aio_read 函数告诉内核描述字缓冲区指针和缓冲区的大小、文件偏移及通知的方式，然后立即返回，当内核将数据拷贝到缓冲区后，再通知应用程序。(第一个阶段(数据就绪)和第二个阶段(数据读写)都是异步)

3.超文本传输协议（Hypertext Transfer Protocol，HTTP，应用层协议）是一个简单的请求-响应协议，它通常运行在 TCP 之上。它指定了客户端可能发送给服务器什么样的消息以及得到什么样的响应。请求和响应消息的头以 ASCII 形式给出；而消息内容则具有一个类似 MIME 的格式。

尽管 TCP/IP 协议是互联网上最流行的应用，HTTP 协议中，并没有规定必须使用它或它支持的层。事实上，HTTP可以在任何互联网协议上，或其他网络上实现。HTTP 假定其下层协议提供可靠的传输。因此，任何能够提供这种保证的协议都可以被其使用。因此也就是其在 TCP/IP 协议族使用 TCP 作为其传输层。

http协议默认端口是80。
https协议默认端口是443。

4.Http的工作原理

HTTP 协议定义 Web 客户端如何从 Web 服务器请求 Web 页面，以及服务器如何把 Web 页面传送给客户端。HTTP 协议采用了请求/响应模型。客户端向服务器发送一个请求报文，请求报文包含请求的方法、URL、协议版本、请求头部和请求数据。服务器以一个状态行作为响应，响应的内容包括协议的版本、成功或者错误代码、服务器信息、响应头部和响应数据。

以下是 HTTP 请求/响应的步骤：
* 客户端连接到 Web 服务器  
    一个HTTP客户端，通常是浏览器，与 Web 服务器的 HTTP 端口（默认为 80 ）建立一个 TCP 套接字连接。例如，http://www.baidu.com。（URL）
* 发送 HTTP 请求  
    通过 TCP 套接字，客户端向 Web 服务器发送一个文本的请求报文，一个请求报文由请求行、请求头部、空行和请求数据 4 部分组成。
* 服务器接受请求并返回 HTTP 响应  
    Web 服务器解析请求，定位请求资源。服务器将资源复本写到 TCP 套接字，由客户端读取。一个响应由状态行、响应头部、空行和响应数据 4 部分组成。
* 释放连接 TCP 连接  
    若 connection 模式为 close，则服务器主动关闭 TCP连接，客户端被动关闭连接，释放 TCP 连接；若connection 模式为 keepalive，则该连接会保持一段时间，在该时间内可以继续接收请求;
* 客户端浏览器解析 HTML 内容  
    客户端浏览器首先解析状态行，查看表明请求是否成功的状态代码。然后解析每一个响应头，响应头告知以下为若干字节的 HTML 文档和文档的字符集。客户端浏览器读取响应数据 HTML，根据 HTML 的语法对其进行格式化，并在浏览器窗口中显示。

5.HTTP请求方法

HTTP/1.1 协议中共定义了八种方法（也叫“动作”）来以不同方式操作指定的资源：
* GET：向指定的资源发出“显示”请求。使用 GET 方法应该只用在读取数据，而不应当被用于产生“副作用”的操作中，例如在 Web Application 中。其中一个原因是 GET 可能会被网络蜘蛛等随意访问。
* HEAD：与 GET 方法一样，都是向服务器发出指定资源的请求。只不过服务器将不传回资源的本文部分。它的好处在于，使用这个方法可以在不必传输全部内容的情况下，就可以获取其中“关于该资源的信息”（元信息或称元数据）。
* POST：向指定资源提交数据，请求服务器进行处理（例如提交表单或者上传文件）。数据被包含在请求本文中。这个请求可能会创建新的资源或修改现有资源，或二者皆有。
* PUT：向指定资源位置上传其最新内容。
* DELETE：请求服务器删除 Request-URI 所标识的资源。
* TRACE：回显服务器收到的请求，主要用于测试或诊断。
* OPTIONS：这个方法可使服务器传回该资源所支持的所有 HTTP 请求方法。用'*'来代替资源名称，向 Web 服务器发送 OPTIONS 请求，可以测试服务器功能是否正常运作。
* CONNECT：HTTP/1.1 协议中预留给能够将连接改为管道方式的代理服务器。通常用于SSL加密服务器的链接（经由非加密的 HTTP 代理服务器）。
* PATCH：对某个资源做部分修改

HEAD、GET、OPTIONS、TRACE被视为安全的方法，因为它们只是从服务器获得资源或信息，而不对服务器进行任何修改。而POST、PUT、DELETE和PATCH则影响服务器上的资源。

"Content-Type:text/html;charset=gbk"表示目标文档的MIME类型。其中"text"是主文档类型，"html"是子文档类型。"text/html"表示目标文档index.html是text类型中的html文档。"charset"是text文档类型的一个参数，用于指定文档的字符编码。

在所有头部字段之后，HTTP应答必须包含一个空行，以标识头部字段的结束。状态行和每个头部字段都必须以<CR><LF>结束；而空行则必须只包含一个<CR><LF>，不能有其他字符，甚至是空白字符。


6.HTTP状态码

所有HTTP响应的第一行都是状态行，依次是当前HTTP版本号，3位数字组成的状态代码，以及描述状态的短语，彼此由空格分隔。

* 状态代码的第一个数字代表当前响应的类型：
    - 1xx消息——请求已被服务器接收，继续处理
    - 2xx成功——请求已成功被服务器接收、理解、并接受
    - 3xx重定向——需要后续操作才能完成这一请求
    - 4xx请求错误——请求含有词法错误或者无法被执行
    - 5xx服务器错误——服务器在处理某个正确请求时发生错误

虽然 RFC 2616 中已经推荐了描述状态的短语，例如"200 OK"，"404 Not Found"，但是WEB开发者仍然能够自行决定采用何种短语，用以显示本地化的状态描述或者自定义信息。

--- | 类别 | 原因短语
---|---|---
1XX | Informational(信息性状态码) | 接收的请求正在处理
2XX | Success(成功状态码) | 请求正常处理完毕
3XX | Redirection(重定向状态码) | 需要进行附加操作以完成请求
4XX | Client Error(客户端错误状态码) | 服务器无法处理请求
5XX | Server Error(服务端错误状态码) | 服务器处理请求出错

7.服务器编程基本框架  

虽然服务器程序种类繁多，但其基本框架都一样，不同之处在于逻辑处理。

模块 | 单个服务器程序 | 服务器机群 
---|---|---
I/O 处理单元 | 处理客户连接，读写网络数据 | 作为接入服务器，实现负载均衡(从所有逻辑服务器中选取负荷最小的一台来为新客户服务)
逻辑单元 | 业务进程或线程 | 逻辑服务器
网络存储单元 | 本地数据库、文件或缓存 | 数据库服务器
请求队列 | 各单元之间的通信方式 | 各服务器之间的永久TCP连接

* I/O 处理单元是服务器管理客户连接的模块。它通常要完成以下工作：等待并接受新的客户连接，接收客户数据，将服务器响应数据返回给客户端。但是数据的收发不一定在 I/O 处理单元中执行，也可能在逻辑单元中执行，具体在何处执行取决于事件处理模式。

* 一个逻辑单元通常是一个进程或线程。它分析并处理客户数据，然后将结果传递给 I/O 处理单元或者直接发送给客户端（具体使用哪种方式取决于事件处理模式）。服务器通常拥有多个逻辑单元，以实现对多个客户任务的并发处理。

* 网络存储单元可以是数据库、缓存和文件，但不是必须的。

* 请求队列是各单元之间的通信方式的抽象。I/O 处理单元接收到客户请求时，需要以某种方式通知一个逻辑单元来处理该请求。同样，多个逻辑单元同时访问一个存储单元时，也需要采用某种机制来协调处理竞态条件。请求队列通常被实现为池的一部分。

8.两种高效的事件处理模式

服务器程序通常需要处理三类事件：I/O 事件、信号及定时事件。有两种高效的事件处理模式：Reactor 和 Proactor，同步 I/O 模型通常用于实现 Reactor 模式，异步 I/O 模型通常用于实现 Proactor 模式。
可以认为，同步I/O向应用程序通知的是I/O就绪事件，而异步I/O向应用程序通知的是I/O完成时间。

9.Reactor模式  

要求主线程（I/O处理单元）只负责监听文件描述符上是否有事件发生，有的话就立即将该事件通知工作线程（逻辑单元），将 socket 可读可写事件放入请求队列，交给工作线程处理。除此之外，主线程不做任何其他实质性的工作。读写数据，接受新的连接，以及处理客户请求均在工作线程中完成。

使用同步 I/O（以 epoll_wait 为例）实现的 Reactor 模式的工作流程是：
* 主线程往 epoll 内核事件表中注册 socket 上的读就绪事件。
* 主线程调用 epoll_wait 等待 socket 上有数据可读。
* 当 socket 上有数据可读时， epoll_wait 通知主线程。主线程则将 socket 可读事件放入请求队列。
* 睡眠在请求队列上的某个工作线程被唤醒，它从 socket 读取数据，并处理客户请求，然后往 epoll 内核事件表中注册该 socket 上的写就绪事件。
* 当主线程调用 epoll_wait 等待 socket 可写。
* 当 socket 可写时，epoll_wait 通知主线程。主线程将 socket 可写事件放入请求队列。
* 睡眠在请求队列上的某个工作线程被唤醒，它往 socket 上写入服务器处理客户请求的结果。

10.Proactor模式  

Proactor 模式将所有 I/O 操作都交给主线程和内核来处理（进行读、写），工作线程仅仅负责业务逻辑。使用异步 I/O 模型（以 aio_read 和 aio_write 为例）实现的 Proactor 模式的工作流程是：
* 主线程调用 aio_read 函数向内核注册 socket 上的读完成事件，并告诉内核用户读缓冲区的位置，以及读操作完成时如何通知应用程序（这里以信号为例）。
* 主线程继续处理其他逻辑。
* 当 socket 上的数据被读入用户缓冲区后，内核将向应用程序发送一个信号，以通知应用程序数据已经可用。
* 应用程序预先定义好的信号处理函数选择一个工作线程来处理客户请求。工作线程处理完客户请求后，调用 aio_write 函数向内核注册 socket 上的写完成事件，并告诉内核用户写缓冲区的位置，以及写操作完成时如何通知应用程序。
* 主线程继续处理其他逻辑。
* 当用户缓冲区的数据被写入 socket 之后，内核将向应用程序发送一个信号，以通知应用程序数据已经发送完毕。
* 应用程序预先定义好的信号处理函数选择一个工作线程来做善后处理，比如决定是否关闭 socket。

11.同步 I/O 模拟 Proactor 模式 

使用同步 I/O 方式模拟出 Proactor 模式。原理是：主线程执行数据读写操作，读写完成之后，主线程向工作线程通知这一"完成事件"。那么从工作线程的角度来看，它们就直接获得了数据读写的结果，接下来要做的只是对读写的结果进行逻辑处理。

使用同步 I/O 模型（以 epoll_wait为例）模拟出的 Proactor 模式的工作流程如下：
* 主线程往 epoll 内核事件表中注册 socket 上的读就绪事件。
* 主线程调用 epoll_wait 等待 socket 上有数据可读。
* 当 socket 上有数据可读时，epoll_wait 通知主线程。主线程从 socket 循环读取数据，直到没有更多数据可读，然后将读取到的数据封装成一个请求对象并插入请求队列。
* 睡眠在请求队列上的某个工作线程被唤醒，它获得请求对象并处理客户请求，然后往 epoll 内核事件表中注册 socket 上的写就绪事件。
* 主线程调用 epoll_wait 等待 socket 可写。
* 当 socket 可写时，epoll_wait 通知主线程。主线程往 socket 上写入服务器处理客户请求的结果。


12.两种高效的事件处理模式

服务器主要有两种并发编程模式：半同步/半异步模式 和 领导者/追随者模式。
* 半同步/半异步模式
    - 在I/O模型中，“同步”和“异步”区分的是内核向应用程序通知的是何种I/O事件(是就绪事件还是完成事件)，以及该由谁来完成I/O读写(是应用程序还是内核)。
    - 在并发模式中，“同步”指的是程序完全按照代码序列的顺序执行；“异步”指的是程序的执行需要由系统事件(例如中断、信号等)来驱动。
    - 半同步/半异步模式中，同步线程用于处理客户逻辑；异步线程用于处理I/O事件。
* 领导者/追随者模式
    - 领导者/追随者模式是多个工作线程轮流获得事件源集合，轮流监听、分发并处理事件的一种模式。
    - 在任意时间点，程序都仅有一个领导者线程，它负责监听I/O事件。而其他线程则都是追随者，它们休眠在线程池中等待成为新的领导者。
    - 当前领导者如果检测到I/O事件，首先要从线程池中推选出新的领导者线程，然后处理I/O事件。此时，新的领导者等待新的I/O事件，而原来的领导者则处理I/O事件，二者实现了并发。

13.线程池
* 线程池是由服务器预先创建的一组子线程，线程池中的线程数量应该和 CPU 数量差不多。线程池中的所有子线程都运行着相同的代码。当有新的任务到来时，主线程将通过某种方式选择线程池中的某一个子线程来为之服务。相比与动态的创建子线程，选择一个已经存在的子线程的代价显然要小得多。
* 至于主线程选择哪个子线程来为新任务服务，则有多种方式：
    - 主线程使用某种算法来主动选择子线程。最简单、最常用的算法是随机算法和 Round Robin（轮流选取）算法，但更优秀、更智能的算法将使任务在各个工作线程中更均匀地分配，从而减轻服务器的整体压力。
    - 主线程和所有子线程通过一个共享的工作队列来同步，子线程都睡眠在该工作队列上。当有新的任务到来时，主线程将任务添加到工作队列中。这将唤醒正在等待任务的子线程，不过只有一个子线程将获得新任务的”接管权“，它可以从工作队列中取出任务并执行之，而其他子线程将继续睡眠在工作队列上。

线程池中的线程数量最直接的限制因素是中央处理器(CPU)的处理器(processors/cores)的数量N ：如果你的CPU是4-cores的，对于CPU密集型的任务(如视频剪辑等消耗CPU计算资源的任务)来说，那线程池中的线程数量最好也设置为4（或者+1防止其他因素造成的线程阻塞）；对于IO密集型的任务，一般要多于CPU的核数，因为线程间竞争的不是CPU的计算资源而是IO，IO的处理一般较慢，多于cores数的线程将为CPU争取更多的任务，不至在线程处理IO的过程造成CPU空闲导致资源浪费。

* 空间换时间，浪费服务器的硬件资源，换取运行效率。
* 池是一组资源的集合，这组资源在服务器启动之初就被完全创建好并初始化，这称为静态资源。
* 当服务器进入正式运行阶段，开始处理客户请求的时候，如果它需要相关的资源，可以直接从池中获取，无需动态分配。
* 当服务器处理完一个客户连接后，可以把相关的资源放回池中，无需执行系统调用释放资源。

**线程的处理函数在C++中是静态的(在C中用的是全局)，而静态函数不能使用类里面的非静态的成员变量，解决的方案是通过处理函数的参数传this指针进去。**

**模板定义声明最好在一个文件，不然可能会报错**

14.有限状态机

逻辑单元内部的一种高效编程方法：有限状态机（finite state machine）。

15.EPOLLONESHOT事件

即使可以使用 ET 模式，一个socket 上的某个事件还是可能被触发多次。这在并发程序中就会引起一个问题。比如一个线程在读取完某个 socket 上的数据后开始处理这些数据，而在数据的处理过程中该socket 上又有新数据可读（EPOLLIN 再次被触发），此时另外一个线程被唤醒来读取这些新的数据。于是就出现了两个线程同时操作一个 socket 的局面。一个socket连接在任一时刻都只被一个线程处理，可以使用 epoll 的 EPOLLONESHOT 事件实现。

对于注册了 EPOLLONESHOT 事件的文件描述符，操作系统最多触发其上注册的一个可读、可写或者异常事件，且只触发一次，除非我们使用 epoll_ctl 函数重置该文件描述符上注册的 EPOLLONESHOT 事件。这样，当一个线程在处理某个 socket 时，其他线程是不可能有机会操作该 socket 的。但反过来思考，注册了 EPOLLONESHOT 事件的 socket 一旦被某个线程处理完毕， 该线程就应该立即重置这个socket 上的 EPOLLONESHOT 事件，以确保这个 socket 下一次可读时，其 EPOLLIN 事件能被触发，进而让其他工作线程有机会继续处理这个 socket。

**注意，监听socket是不能注册EPOLLONESHOT事件的，否则应用程序只能处理一个客户连接！因为后续的客户连接请求将不再触发监听socket上的EPOLLIN事件。**

16.服务器压力测试  
Webbench 是 Linux 上一款知名的、优秀的 web 性能压力测试工具。它是由Lionbridge公司开发。  

    * 测试处在相同硬件上，不同服务的性能以及不同硬件上同一个服务的运行状况。
    * 展示服务器的两项内容：每秒钟响应请求数和每秒钟传输数据量。

基本原理：Webbench 首先 fork 出多个子进程，每个子进程都循环做 web 访问测试。子进程把访问的结果通过pipe 告诉父进程，父进程做最终的统计结果。

```
// 测试示例
webbench -c 1000 -t 30 http://192.168.110.129:10000/index.html
参数：
-c 表示客户端数
-t 表示访问时间，单位秒。即30s内有1000个客户端请求。
```


## 七、Linux服务器程序规范

1.大部分后台进程都在/var/log目录下拥有自己的日志目录。

2.绝大多数服务器程序都有配置文件，并存放在/etc目录下。

3.Linux服务器进程通常会在启动的时候生成一个PID文件并存入/var/run目录中，以记录该后台进程的PID。

4.大部分服务器必须以root身份启动但不能以root身份运行。

5.Linux提供一个守护进程来处理系统日志——syslogd，现在使用的已经是它的升级版——rsyslogd。

rsyslogd守护进程既能接收用户进程输出的日志，又能接收内核日志。
```
#include <syslog.h>
// 应用程序使用syslog函数与rsyslogd守护进程通信。
void syslog(int priority, const char* message, ...);
    
// 下面这个函数可以改变syslog的默认输出方式：
void openlog(const char* ident, int logopt, int facility);
    - ident指定的字符串将被添加到日志消息的日期和时间之后，它通常被设置为程序的名字。
    - logopt参数对后续syslog调用的行为进行配置。
    - facility参数可用来修改syslog函数中的默认设施值。

// 下面这个函数用于设置syslog的日志掩码
int setlogmask(int maskpri);
    - maskpri参数指定日志掩码值。

// 最后不要忘了关闭日志功能：
void closelog();
```

6.下面这一组函数可以获取和设置当前进程的真实用户ID(UID)、有效用户ID(EUID)、真实组ID(GID)和有效组ID(EGID)。
```
#include <sys/types.h>
#include <unistd.h>
uid_t getuid();                 // 获取真实用户ID
uid_t geteuid();                // 获取有效用户ID
gid_t getgid();                 // 获取真实组ID
gid_t getegid();                // 获取有效组ID
int setuid(uid_t uid);          // 设置真实用户ID
int seteuid(uid_t uid);         // 设置有效用户ID
int setgid(gid_t gid);          // 设置真实组ID
int setegid(gid_t gid);         // 设置有效组ID
```
* 一个进程拥有两个用户ID：UID和EUID。EUID存在的目的是方便资源访问：它使得运行程序的用户拥有该程序的有效用户的权限。

```
// 如果将以root身份启动的进程切换为以一个普通用户身份运行。
static bool switch_to_user(uid_t user_id, gid_t gp_id)
{
    // 先确保目标用户不是root
    if(0 == user_id && 0 == gp_id)
        return false;
    
    // 确保当前用户是合法用户：root或者目标用户
    gid_t gid = getgid();
    uid_t uid = getuid();
    if((0 != gid || 0!= uid) && (gid != gp_id || uid != user_id))
        return false;
    
    // 如果不是root，则已经是目标用户
    if(0 != uid)
        return false;
    
    // 切换到目标用户
    if(setgid(gp_ud) < 0 || setuid(user_id) < 0)
        return false;
        
    return true;
}
```

7.Linux系统资源限制可以用个如下一对函数来读取和设置：
```
#include <sys/resource.h>
int getrlimit(int resource, struct rlimit *rlim);
int setrlimit(int resource, const struct rlimit *rlim);

struct rlimit
{
    rlimt_t rlim_cur;   // 指定资源的软限制
    rlimt_t rlim_max;   // 指定资源的硬限制
}
```
此外，我们可以使用ulimit命令修改当前shell环境下的资源限制(软限制或/和硬限制)。

8.高性能服务器应该避免不必要的数据复制，尤其是当数据复制发生在用户代码和内核之间的时候。例如可以用零拷贝函数去实现某些功能。

用户代码内部(不访问内核)的数据复制也是应该避免的。例如，使用共享内存来共享这些数据，而不是使用管道或消息队列传递数据。

9.strace命令能跟踪程序执行时调用的系统调用和接收到的信号。

10.两种高效的管理定时器的容器：时间轮和时间堆。

Linux提供了三种定时方法，它们是：
* socket选项SO_RCVTIMEO和SO_SNDTIMEO
* SIGALRM信号
* I/O复用系统调用的超时参数
