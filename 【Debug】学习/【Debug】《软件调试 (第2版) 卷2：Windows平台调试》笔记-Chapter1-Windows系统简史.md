# 前言

阅读本书的最好方法是根据书中的提示，调试各种代码，多动手，多实践。本书的配套网站（http://advdbg.org）上有本书示例程序的源代码和编译好的二进制文件。

# 第一篇　大 局 观

# 第1章　Windows系统简史

## 1.1　源于DOS

直到今天，在每个Windows系统里，还都保存着DOS的痕迹。例如，在所有Windows系统的可执行文件（称为PE格式）的开头，一般都有一个DOS头。DOS头总是以MZ（十六进制4D 5A）开头，代表一位DOS开发者的名字Mark Zbikowski。DOS头后面一般还有一小段16位的DOS程序，当用户在DOS下执行这个文件时，这一小段程序会提醒用户该程序不可以运行在DOS模式下。

Windows NT（New Technology）系列，包括Windows NT 3.1（1993年7月发布）、Windows NT 4.0（1996年7月发布）、Windows 2000（1999年12月发布）、Windows XP（2001年10月发布）、Windows Server 2003、Windows Vista、Windows Server 2008、Windows 7、Windows Server 2012、Windows 8、Windows 10等，这些版本都源于Windows NT内核。

## 1.2　功在NT

略。

## 1.3　Windows 2000彰显实力

略。

## 1.4　巅峰之作：Windows XP和Windows Server 2003

略。

## 1.5　Windows Vista折戟沙场

从Vista实际发布的功能来看，比较显著的有以下几项。

（1）重新规划和设计的GPU软件栈和Windows显示驱动程序模型（Windows Display Driver Model，WDDM）。WDDM实现了针对GPU的很多高级功能，比如显存虚拟化、GPU任务的并行和抢先式调度、多GPU支持等。直到今天，这些功能仍是Windows系统领先于Linux系统的地方。笔者认为，WDDM可以算是Vista对NT系统所做的最大贡献。

（2）重构的Windows驱动程序模型——Windows Driver Foundation（WDF）支持使用C++编写用户态的驱动程序，使用面向对象技术对原来的WDDM驱动模型做了一些封装，本意是简化驱动程序开发，但是增加了一层封装后，也增加了模糊性和开发者要学习的内容。另一个副作用是，原来DDK中的一些经典例子被删除了，可能是因为难以转换为WDF风格。

（3）安全方面，引入了用户账号控制（User Account Control，UAC），限定普通应用程序只有标准用户权限，提升特权时需要用户同意。因为牵涉用户交互和程序界面，所以UAC对Windows系统的影响是广泛和深远的。

## 1.6　Windows 7享利中兴

Windows 7还引入了与MinWin有关的一项变化，即对Kernel32.DLL做了重构，引入Kernelbase.DLL，把原本实现在Kernel32.DLL中的逻辑移到Kernelbase中，Kernel32只保留接口，这样修改后，负责用户空间开发的团队只需要使用稳定版本的Kernel32.DLL，不需要频繁更新，负责内核空间的团队如果对底层做修改，一般只需要修改Kernelbase.DLL，不需要更新Kernel32.DLL，两个团队之间的相互牵制大大减少。

因为这一改动，在Windows 7或者更高版本的Windows系统中，Kernel32.DLL中的API入口大多只剩一条无条件跳转指令（jmp），比如ReadFile API：

```
0:011> u kernel32!readfile
KERNEL32!ReadFile:
00007ffb`f64d0cc0 ff25e26d0500    jmp     qword ptr [KERNEL32!_imp_ReadFile  
(00007ffb`f6527aa8)]
```

上面的`_imp_ReadFile`是导入表项，相当于函数指针，jmp指令跳转的目标是指针的内容，使用`ln`命令观察这个指针。

```
0:011> ln poi(KERNEL32!_imp_ReadFile)
(00007ffb`f5282ac0)   KERNELBASE!ReadFile   Exact matches:
    KERNELBASE!ReadFile (void)
```

可以看到，`_imp_ReadFile`指向的目标就是kernelbase中的ReadFile函数。

Windows 7所做的更多是对Vista引入的新功能进行改进和优化，把慢的加快，把不方便的改得方便，把瑕疵去掉，把棱角打平。举例来说，安装Windows 7时，Windows 7的安装程序就会在硬盘上建立一个特殊的分区，安装一个用于修复故障的简易Windows系统，称为Windows恢复环境（Windows Recovery Environment，WRE）。在系统盘上，可以看到一个名为Recovery的隐藏目录，里面放着包括WRE磁盘映像在内的一些文件。当正常的Windows系统无法启动时，Windows 7可以让用户从高级启动选项（按F8键）中选择“Repair Your Computer”，进入WRE。

## 1.7　Windows 8革新受挫

在Windows 8或者更高版本的Windows系统中，在“开始”菜单处输入bootim并执行它，就会出现启动菜单画面，一般只包含一个“关闭电脑”选项，看到这个画面，你可能被吓一跳，以为系统突然重启了，其实不必害怕，它只是一个全屏显示的普通应用，按`Alt + Tab`组合键就可以把它切换到后台了。

另外，可能是受iPad“无须关机”特征的影响，Windows 8的“开始”菜单中也删除了关机功能。这让很多老用户很不习惯，因为已经习惯了不用计算机就要关机，现在找不到“关机”菜单，怎不让用户着急？于是很多网站发表文章教用户如何寻找Windows 8的“关机”菜单。

其实，即使用户找到并选择了关机，Windows 8实际执行的也不再是传统的关机过程，而是改进了的休眠过程，把系统内核的执行状态保存在磁盘上。当用户下一次开机时，Windows 8会从磁盘上恢复上次的执行状态，迅速地显示出桌面，让用户感觉启动速度非常快，这个功能称为“混合启动”（hybrid boot）。

## 1.8　Windows 10何去何从

从功能角度来看，Windows 10引入的最大新功能莫过于适用于Linux的Windows子系统（Windows Subsystem for Linux，WSL）。启用了WSL功能后，用户可以在Windows系统中运行原生的Linux程序，但这些程序依赖的不是Linux内核，而是经典的NT内核，某种程度上说，这称得上是NT内核的一次重生。

另外一个非常大的新功能便是VBS，全称是基于虚拟化的安全（Virtualization-Based Security）。启用VBS功能后，Windows 10会启动集成的Hyper-V的虚拟机监视器（VMM），然后启动两个虚拟机，一个运行NT内核，另一个运行专门用于安全目的的安全内核（SecureKernel.EXE）。

在安全内核上面，运行着一些特殊的应用程序，这些程序是使用特殊工具开发的，运行在隔离的用户空间（Isolated User Mode，IUM）中，普通内核上面即使有恶意软件，也很难攻击到IUM中的程序。从安全的角度来看，安全内核提供了一个隔离的环境，执行证书验证等安全有关的功能操作，这是有价值的。但是，从用户的角度来看，把普通的程序和NT内核运行在虚拟机里，是会损失性能和降低用户体验的。

## 1.9　本章总结

表1-3　源于NT内核的Windows操作系统

| 产品名称 | 内部版本号 | 发布日期 | 备　　注 |
| --- | --- | --- | --- |
| Windows NT 3.1 | 3.1 | 1993年7月 | — |
| Windows NT 3.5 | 3.5 | 1994年9月 | — |
| Windows NT 3.51 | 3.51 | 1995年5月 | — |
| Windows NT 4.0 | 4.0 | 1996年7月 | — |
| Windows 2000 | 5.0 | 1999年12月 | 分为台式机版本（Windows 2000 Professional）和服务器版本 |
| Windows XP | 5.1 | 2001年8月 | 巅峰的桌面版本 |
| Windows Server 2003 | 5.2 | 2003年3月 | 巅峰的服务端版本 |
| Windows Vista | 6.0 | 2006年11月 | — |
| Windows Server 2008 | 6.0 | 2008年2月 | — |
| Windows 7 | 6.1 | 2009年7月 | — |
| Windows Server 2012 | 6.2 | 2012年8月 | — |
| Windows 8 | 6.2 | 2012年10月 | — |
| Windows Server 2012 R2 | 6.2 | 2013年10月 | — |
| Windows 10 | 10.0 | 2015年7月 | — |
| Windows Server 2016 | 10.0 | 2016年9月 | — |
| Windows Server 2019 | 10.0 | 2018年10月 | — |

