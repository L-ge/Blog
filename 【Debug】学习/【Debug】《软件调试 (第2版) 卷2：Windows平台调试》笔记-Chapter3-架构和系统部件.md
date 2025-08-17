# 第3章　架构和系统部件

本章将介绍Windows系统的总体架构和重要的系统部件，包括关键的进程和系统文件，以及环境子系统。

## 3.1　系统概览

原书P55的图3-1简要地勾画出了Windows操作系统的基本架构，展示了内核模式下的关键组件及用户态下的重要进程和动态链接库。该图参考了《深入解析Windows操作系统》（第4版）（电子工业出版社，2007年）的图2-3。

以中间的黑线为界，图3-1中上半部分描述的是用户空间，下半部分描述的是内核空间。在第2章中，我们介绍进程空间时已经比较详细地描述了内核空间与用户空间的关系，本章将引领读者进一步加深对这两大空间的理解。

### 3.1.1　内核空间

简单来说，内核空间就是供操作系统内核使用的内存空间。内核的核心任务是管理硬件资源，为上层应用提供服务。对下要面对五花八门的硬件，对上要面对千变万化的应用程序，内核的复杂度可想而知。为了降低复杂度和便于开发维护，内核代码也是模块化的，按照职能分成若干个部分，相互协作。从这个角度来说，内核空间中，除了狭义的内核模块（操作系统核心之核心），还有一些其他模块，概述如下。

（1）硬件抽象层（Hardware Abstraction Layer，HAL），主要作用是隔离硬件差异性，使内核和顶层模块可以通过统一的方式来访问硬件。值得说明的是，HAL负责的硬件类型只限于CPU架构层面的核心硬件，比如中断控制器、固件接口等，HAL并不负责外设类型的硬件，外设硬件的差异性问题是通过I/O管理器加载不同的设备驱动程序来解决的。

（2）操作系统内核，负责线程调度、中断处理、异常分发、多处理器同步等关键任务，是操作系统最核心的部分，我们将其称为微观意义的内核。

（3）执行体（executive），帮助内核行使（执行）某一方面的职能，比如内存管理器负责管理和分配内存资源，进程管理器负责管理系统中的所有进程，输入/输出（I/O）管理器负责协调系统的硬件资源，等等。如果把操作系统内核比喻成系统的最高权力机构，那么执行体便是它的一个个职能部门，负责各方面的事务。

（4）内核态驱动程序，包括文件系统和图形显示驱动程序，以及用于其他硬件的驱动程序，有些是Windows系统自带的，有些是硬件厂商提供的。驱动程序是对内核功能的补充。

（5）Windows子系统驱动程序（Win32K.SYS），包括USER和GDI（Graphics Device Interface，图形设备接口）两大部分，USER部分负责窗口管理、用户输入等，GDI部分负责显示输出和各种图形操作。从Windows Vista开始，名为DXGKRNL的图形核心负责管理GPU有关的核心任务，详见本书卷1的GPU部分。

（6）内核支持模块，包括用于内核调试的KDCOM.DLL，用于启动阶段的显示驱动BOOTVID.DLL，Windows Vista引入的用于检查模块完好性的CI.DLL（CI是Code Integrity的缩写），支持日志功能的CLFS.SYS，支持WHEA的PSHED.DLL。

内核支持模块中的PSHED.DLL将在第17章介绍。3.2节将详细介绍内核文件和HAL文件。

### 3.1.2　用户空间

简单来说，用户空间是应用程序代码和各种用户态模块运行的内存空间。所谓应用程序，就是实现某一方面应用的软件程序，比如办公软件、浏览器、即时通信程序等。

除了用户安装的或者系统预装的普通应用程序，用户空间中还运行着操作系统的一些进程，一般将它们称为常规进程。从职能角度来看，常规进程的角色与内核空间的执行体有些类似，它们分别负责某一方面的职能，帮助内核一起维护系统的秩序。在一个典型的Windows系统中，一般都运行着下面这些常规进程。

（1）会话管理器进程（SMSS.EXE），它是系统中第一个根据映像文件创建的进程，是在系统启动后期由执行体的初始化函数创建的。它运行后，会加载和初始化Win32子系统的内核模块Win32K.SYS，创建Win32子系统服务器进程（CSRSS.EXE），以及登录进程（WinLogon.EXE）。

（2）Windows子系统服务器进程（CSRSS.EXE），负责维护Windows子系统的“日常事务”，为子系统中的各个进程提供服务。例如登记进程和线程，管理控制台窗口，管理DOS程序虚拟机（VDM）进程等。CSRSS是Client/Server Runtime Server Subsystem的缩写，即客户端/服务器运行时子系统。

（3）登录进程（WinLogon.EXE），负责用户登录和安全有关的事务。它启动后，会创建LSASS进程和系统服务管理进程（Services.EXE）。Windows XP的文件保护（Windows File Protection，WFP）功能也是在这个进程中实现的（sfc.dll和sfc_os.dll）。

（4）本地安全和认证进程（LSASS.EXE），负责用户身份验证，LSASS是Local Security Authority Subsystem Service的缩写。

（5）服务管理进程（SERVICES.EXE），负责启动和管理系统服务程序。系统服务程序是按照NT系统服务规范编写的EXE程序，通常没有用户界面，只在后台运行。图3-1中列出了几个常见的系统服务，SpoolSv.exe是打印机脱机服务，WmiPrvSE.exe是WMI提供器管理服务，SvcHost.exe是一个通用的服务宿主程序。

（6）OS/2子系统和POSIX子系统服务进程，用于在Windows系统中运行OS/2和符合POSIX标准的程序。它们只有在需要时才启动，在环境子系统部分将做进一步讨论。

（7）外壳（Shell）程序，默认为Explorer.exe，负责显示“开始”菜单、任务栏和桌面图标等。

与单一的内核空间不同，在一个正常的Windows系统中，用户空间总是有很多个，每个普通的进程都有一个用户空间，它们相互独立，是隔离开的。每个应用程序运行在自己的用户空间中，可以访问连续的地址空间，使用各种硬件资源，“仿佛”拥有整个系统。这里的“仿佛”二字非常关键，对于应用程序而言，它似乎拥有单独的系统，所以从单任务时代发展过来的旧软件，很容易移植到新的系统中；对于应用程序开发者来说，每个程序都有自己的连续内存空间，“似乎”自己有一套完整的内存，写代码和调试都很方便；对于操作系统来说，不仅要把这个“仿佛”实现得比较“真”，或者尽可能接近，还要保证整个系统的安全和稳定。

## 3.2　内核和HAL模块

管理纷繁复杂的软件世界不是一件简单的事。开发操作系统是一项庞大的系统工程。在这项工程中，难度最大的要数开发内核模块和HAL模块了。在整个软件世界中，内核空间是管理中枢，是规则的制定者和秩序的维护者。在内核空间中，最重要的两个模块便是内核和HAL，本节将分别描述。

### 3.2.1　内核文件

从文件角度来看，内核与执行体都位于一个文件中，即通常所说的NT内核文件。NT内核文件有几种版本，它们是使用同一套源代码通过不同编译选项而编译出来的。

（1）针对单处理器系统优化的单处理器版本，在64位的Windows系统中，它的原始文件名为NTOSKRNL.EXE；在32位的Windows系统中，根据是否支持物理地址扩展（Physical Address Extension，PAE），它的原始文件名是NTKRNLPA.EXE（支持PAE）或NTOSKRNL.EXE（不支持PAE）。

（2）可用于多处理器系统的多处理器版本，在64位的Windows系统中，它的原始文件名为NTKRNLMP.EXE；在32位的Windows系统中，支持PAE的版本的原始文件名是NTKRPAMP.EXE，不支持PAE的版本是NTKRNLMP.EXE。

那么，系统是如何决定使用以上版本中的哪一个呢？首先安装程序在安装时会根据系统中的处理器个数，选择单处理器版本或多处理器版本中的一个并复制到用户系统（system32目录）中。如果复制的是多处理器版本，那么会将其改为与单处理器版本相同的名字。这也是我们上面强调“原始文件名”的原因，它是相对于安装在用户系统中的文件名而言的。

在Windows XP时代，因为PAE可以在启动选项中开启或关闭，所以安装程序会将支持PAE的版本和不支持PAE的版本都复制到用户系统（system32目录）中，但是会根据上面的原则选择多处理器版本和单处理器版本中的一种，并将其改为单处理器版本的名字。因此，在安装好的Windows系统中，我们通常可以看到两个NT内核文件——NTOSKRNL.EXE和NTKRNLPA.EXE。在Windows系统启动过程中，系统的加载程序（NTLDR或WinLoad）会根据启动选项中是否启用了PAE加载其中的一个。

上面描述的PAE版本主要用在64位内核流行前的32位Windows系统中，今天，大多数PC或者服务器安装的是64位内核，CPU大多数基于AMD64或者INTEL64的x64 架构，x64可以看作对PAE的进一步扩展（页表由3级扩展到4级，支持更长的线性地址和物理地址）。这意味着，运行在x64硬件上的64位内核都是“PAE”版本的，不再有非PAE版本，因此在这样的系统中，system32目录中只有一个内核文件，而且没有必要提是否是PAE版本了。

需要说明的是，无论是32位内核还是64位内核，它所在的目录都叫system32。上一章介绍WoW时讨论过这个问题，在此不再赘述。

另一方面，多处理器版本也是可以运行在单处理器系统中的，只不过在多处理器系统中必须有的同步措施在单处理器系统中是没有必要的，会牺牲一些性能。随着超线程（Hyper Threading）和多核技术的迅速发展，多处理器系统逐渐成为主流，因此从Windows Vista开始不再安装单处理器版本，无论系统是否有多个处理器，安装程序都会为其安装多处理器版本（NTKRNLMP.EXE或NTKRPAMP.EXE），然后将其改为与单处理器版本一样的名字。

右击文件，选择“属性”，在弹出的对话框中单击“详细信息”选项卡中的“原始文件名”，可以观察到内核文件的原始名称。例如，在笔者写作本书第1版时使用的单处理器的Windows Vista系统中，磁盘上的NTOSKRNL.EXE的原始文件名是NTKRNLMP.EXE，代表多处理器版本，证明了从Vista开始即使系统中只有一个CPU，也会使用多处理器版本。在笔者写作本书第2版时使用的4核8线程Windows 10系统中，内核文件的原始文件名也是NTKRNLMP.EXE。

内核文件的大小从侧面反映了NT内核的发展变化。在Windows XP时代，内核文件的大小只有1MB多，到了Vista时代增大到4MB左右；在开发Windows 7时，进行了旨在减小内核大小的MinWin重构，内核文件的大小回退到4MB以内（32位版本）；进入Windows 10时代后，内核文件的大小又迅猛增长。

Dependency Walker（depends.exe，简称Depends）是了解系统文件用途和相互关系的一个简单而有效的工具，它可以显示EXE和DLL文件的输入/导出（import/export）情况，比如DLL的输出函数列表、文件的依赖关系等。SDK、VC6和VS2003/2005中都包含了该工具。以管理员身份使用Depends一次，它便会自动注册，以后右击DLL或EXE文件时，快捷菜单（shortcut menu）中便会包含View Dependencies项。选择此项，系统便会启动Depends。

Dependency Walker的树控件显示了NTOSKRNL.EXE直接依赖6个模块——PSHED.DLL、HAL.DLL、BOOTVID.DLL、KDCOM.DLL、CLFS.SYS和CI.DLL。

Dependency Walker窗口右侧的两个列表分别是，左侧选中模块包含的被NTOSKRNL.EXE使用的函数列表和选中模块输出的所有函数。

窗口最下部的列表显示的是每个文件的大小、属性、版本等信息。

### 3.2.2　HAL文件

与内核文件类似，硬件抽象层模块也有多种版本，以适用于不同的硬件平台。

（1）hal.dll：支持标准平台。

（2）halacpi.dll：支持符合ACPI标准的硬件平台。

（3）halapic.dll：支持APIC（高级可编程中断控制器）的硬件平台。

（4）halaacpi.dll：同时支持ACPI和APIC的硬件平台。

（5）halmps.dll：系统中有一个多处理器。

（6）halmacpi.dll：支持ACPI的多处理器系统。

当安装Windows系统时，安装程序会根据检测到的情况选择一个合适的hal文件，复制到系统目录中，并改名为hal.dll。同样，使用文件属性对话框可以观察它的原始文件名。在写作本书第1版使用的Windows XP系统中，HAL.DLL的原始文件名是halaacpi.dll，即同时支持ACPI和APIC的版本。在写作本书第2版时使用的Windows 10系统中，通过文件属性看到的原始文件名虽然是hal.dll，但是通过调试符号中的信息，可以知道它的原始名字也是halaacpi.dll。

```
Mapped memory image file: c:\symbols\halaacpi.dll\578997A375000\halaacpi.dll
```

下面基于HAL模块内的几个函数来理解它的功能。第一个例子是用于驱动PC蜂鸣器的HalMakeBeep函数。使用WinDBG的uf命令对这个函数进行反汇编，可以看到其内部使用了很多in/out指令来操作0x42、43和61端口，这几个端口是IBM兼容PC的标准端口。

```
hal!HalMakeBeep+0x5e:
fffff800`1164f4ee ba43000000     mov     edx, 43h
fffff800`1164f4f3 b0b6           mov     al,0B6h
fffff800`1164f4f5 ee              out     dx,al
fffff800`1164f4f6 e8b5a5feff      call    hal!HalpIoDelay (fffff800`11639ab0)
fffff800`1164f4fb b942000000      mov     ecx,42h
```

做过Windows驱动开发的同行都知道中断请求级别（IRQL）是NT内核中的一个重要机制。简单来说，IRQL代表着任务的优先级，如果IRQL操作不当，那么很容易导致系统挂死或者蓝屏。在Windows XP时代的HAL中，有一个名为KfLowerIrql的函数，是驱动开发接口（DDI）KeLowerIrql的实现，用于降低IRQL，其汇编代码如下。

```
kd> uf hal!KfLowerIrql
hal!KfLowerIrql:
806d02d0 33c0            xor     eax,eax
806d02d2 8ac1            mov     al,cl
806d02d4 33c9            xor     ecx,ecx
806d02d6 8a8858026d80    mov     cl,byte ptr hal!HalRequestIpi+0x4e8 (806d0258)[eax]
806d02dc 890d8000feff    mov     dword ptr ds:[0FFFE0080h],ecx
806d02e2 a18000feff      mov     eax,dword ptr ds:[FFFE0080h]
806d02e7 c3              ret
```

上面代码中的FFFE0080h是特殊约定的线性地址，对应的是APIC的任务优先级寄存器（Task Priority Register，TPR）。

在Windows 10中，NT内核中有一个类似功能的函数，直接操作TPR的别名寄存器CR8。

```
0: kd> uf nt!KzLowerIrql
nt!KzLowerIrql:
fffff800`116b2200 0fb6c1          movzx   eax,cl
fffff800`116b2203 440f22c0        mov     cr8,rax
fffff800`116b2207 c3              ret
```

使用特殊编码的控制寄存器访问指令操作CR8要比访问内存速度快，这是因为定义x64架构时，故意对这个常用的重要操作做了优化。在Windows 10版本的hal.dll中，还使用老的方法设置IRQL，其函数名为hal!HalpApicSetPriority。

## 3.3　空闲进程

两个特殊的进程——系统进程（system process）和空闲进程（idle process）。之所以说它们特殊，是因为这两个进程都具有如下特征。

（1）普通的Windows进程都是通过使用CreateProcess或类似的API并指定一个可执行映像文件而创建的，但这两个进程不是，它们没有对应的磁盘映像文件，是在系统启动时“捏造”出来的。

（2）普通的Windows进程都有用户空间和内核空间两个部分，但是这两个进程都只有内核空间，没有用户空间。或者说，这两个进程只在高特权的内核空间中运行。

（3）具有固定的进程ID，空闲进程的ID总是0，系统进程的ID总是8（Windows 2000）或4（Windows XP及之后）。

考虑到系统进程和空闲进程的特殊重要性，本节和3.4节将分别介绍它们。按照系统启动时创建这两个进程的先后顺序，本节先介绍空闲进程。

简单来说，空闲进程是空闲线程的载体。什么是空闲线程呢？用通俗的话来讲，CPU是一种古怪的“动物”，它一上电工作，就要取指令并执行，没有事情做时，也要给它几条指令，组成一个循环，让它在那里空转。因此，在今天的操作系统中，都会设计空闲线程，当CPU没有其他线程需要执行（空闲）时，让它执行空闲线程。

因为系统中每个工作的CPU都需要空闲线程，所以空闲线程的个数与系统中启用的处理器个数是一致的。根据这一原理，我们只要在任务管理器中查看空闲进程的线程个数，就知道系统中的逻辑CPU个数。比如，笔者现在使用的笔记本电脑有8个CPU，空闲进程的线程个数是8。

在NT内核启动时，就会创建空闲进程。更准确地说，是在执行体的阶段0初始化时，进程管理器的初始化函数便会创建空闲进程和第一个空闲线程。可以说，第一个空闲线程是从系统的初始启动线程“蜕化”而来的，是系统中的第一个线程，也是“年龄最大”的线程。

因为空闲进程是系统启动时创建的第一个进程，而且它一直存续，所以可以通过观察它的运行时间来间接推测系统的运行时间，方法是把空闲进程的CPU时间除以CPU个数。比如空闲进程的CPU净时间是116小时，除以CPU个数8，大约为15小时，一般向上取整，因为CPU还用一定的时间执行其他进程。

WinDBG的进程观察命令（`!process`）一般不显示空闲进程，但可以使用如下方法来观察空闲进程。在内核调试会话中执行`!prcb`命令显示处理器控制块（processor control block）。

```
lkd> !prcb
PRCB for Processor 0 at ffdff120:
Threads--  Current 88368470 Next 00000000 Idle 80551d20
Number 0 SetMember 00000001
Interrupt Count -- 04080105
Times --    Dpc        000037a7 Interrupt     00001784 
            Kernel     00725fff User          0005d1be
```

观察`Threads`一行，`Current`字段是当前CPU正在执行的线程的ETHRAD结构（作用相当于进程的EPROCESS结构）的地址，`Next`是等待执行的线程，`Idle`字段的值就是当前CPU的空闲线程的ETHRAD结构地址。使用`!thread`命令可以显示线程的详细信息。

```
lkd> !thread 80551d20
THREAD 80551d20  Cid 0000.0000  Teb: 00000000 Win32Thread: 00000000 RUNNING on  
processor 0
Not impersonating
Owning Process          80551f80     Image:         Idle
Wait Start TickCount     6399946      Ticks: 9878376 (1:18:52:29.625)
Context Switch Count     23038498             
UserTime                00:00:00.000
KernelTime             1 Day 04:18:18.593
Stack Init 80549500 Current 8054924c Base 80549500 Limit 80546500 Call 0
Priority 16 BasePriority 0 PriorityDecrement 0 DecrementCount 0
Unable to get context for thread running on processor 0, HRESULT 0x80004001
```

进程ID（`Cid`）字段为0000.0000，即这个线程属于空闲进程，Owning Process后便是空闲进程的EPROCESS结构地址，`Image`后的映像文件名`Idle`是杜撰的。`UserTime`为00:00:00.000，表示空闲线程只是在内核模式执行的。`KernelTime`是在内核模式的执行时间。

有了空闲进程的EPROCESS结构地址后，便可以使用`!process`命令来观察它了，在此从略。

在上面的运行结果中，最下面一行显示没能读取到空闲线程的上下文，这是因为我们使用的是本地内核调试对话。如果使用真正的双机内核调试或者观察内核转储文件，就可以显示出线程的栈回溯，比如下面是分析转储文件时观察空闲线程的结果。

```
1: kd> knc
# Child-SP           RetAddr           Call Site
00 ffffd001`629c6818 fffff801`890649a1 amdppm!ReadIoMemRaw+0xe8
01 ffffd001`629c6820 fffff801`89062580 amdppm!ReadGenAddr+0x21
02 ffffd001`629c6850 fffff803`07485a05 amdppm!AcpiCStateIdleExecute+0x20
03 ffffd001`629c6880 fffff803`0748526d nt!PpmIdleExecuteTransition+0x5e5
04 ffffd001`629c6b00 fffff803`075c8f4c nt!PoIdle+0x33d
05 ffffd001`629c6c60 00000000`00000000 nt!KiIdleLoop+0x2c
```

从下往上观察上面的栈回溯，最下面的KiIdleLoop便是空闲线程的入口和主函数，它上面的PoIdle是电源执行体（Power）中专门为空闲线程设计的工作函数，它的职能是让CPU执行空闲线程时可以进入合适的电能状态。对于PC系统，这个函数一般会调用CPU的处理器电源管理（Processor Power Management，PPM）模块，进入省电状态。上面调用的是为AMD CPU设计的PPM模块，这个模块导出一系列函数，供内核中的PpmIdleExecuteTransition调用。

CPU在空闲线程中执行时，除了调用PoIdle打盹休息之外，也会做些“零活”。比如调用KiRetireDpcList函数来执行挂在延迟过程调用（Delay Process Call，DPC）队列里的任务。

举例来说，2009年夏季，笔者用于写作本书第1版的笔记本电脑系统有时会僵死，界面没有反应，触发蓝屏崩溃后用调试器分析转储文件，看到如下栈回溯。

```
# ChildEBP RetAddr  
00 80548f8c ba9ca7fa nt!KeBugCheckEx+0x1b
01 80548fa8 ba9ca032 i8042prt!I8xProcessCrashDump+0x237
02 80548ff0 80540add i8042prt!I8042KeyboardInterruptService+0x21c
03 80548ff0 806d6da5 nt!KiInterruptDispatch+0x3d
04 805490a4 bac7d183 hal!KeStallExecutionProcessor+0x1651
05 805490c4 b9123b50 usbehci!EHCI_RH_PortResetComplete+0x61
06 805490e4 804ff550 USBPORT!USBPORT_AsyncTimerDpc+0x7a
07 80549200 804ff667 nt!KiTimerListExpire+0x122
08 8054922c 8054111d nt!KiTimerExpiration+0xaf
09 80549250 80541096 nt!KiRetireDpcList+0x46
0a 80549254 00000000 nt!KiIdleLoop+0x26
```

简言之，上面的栈回溯说明CPU在执行空闲线程时，发现DPC队列里有任务要做，于是去执行，但在执行USB驱动的DPC任务时，因为usbehci驱动中的设计缺欠和硬件老化而陷入死循环，反复调用HAL模块中的KeStallExecutionProcessor函数完成忙等待。栈帧#0～3表示在等待时，发生了键盘中断，因为笔者按下了触发蓝屏的Ctrl + PrintScreen快捷键。

## 3.4　系统进程

简单来说，系统进程是操作系统内核和所有系统线程的宿主，其作用是为操作系统提供独立的进程空间和进程对象。

在内核调试会话中，可以使用`!process 4`命令来观察系统进程的概要信息，例如清单3-1是笔者写作本书第1版时所用Windows XP系统上系统进程的概要信息。

清单3-1　在WinDBG中观察系统进程的概要信息

```
lkd> !process 4 1
Searching for Process with Cid == 4
PROCESS 8a6f2660  SessionId: none  Cid: 0004    Peb: 00000000  ParentCid: 0000
    DirBase: 0072c000  ObjectTable: e1001c38  HandleCount: 705.
    Image: System
    VadRoot 8a6eb870 Vads 4 Clone 0 Private 3. Modified 504318. Locked 0.
    DeviceMap e10087c0
    Token                            e1000728
    ElapsedTime                      2 Days 22:24:34.216 // 约等于系统总运行时间
    UserTime                         00:00:00.000
    KernelTime                       00:03:00.203
    QuotaPoolUsage[PagedPool]        0
    QuotaPoolUsage[NonPagedPool]     0
    Working Set Sizes (now,min,max)    (75, 0, 345) (300KB, 0KB, 1380KB)
    PeakWorkingSetSize               654
    VirtualSize                       1 Mb
    PeakVirtualSize                  3 Mb
    PageFaultCount                   12850
    MemoryPriority                   BACKGROUND
    BasePriority                     8
    CommitCharge                     7
```

注意，系统进程的`Peb`是`0`，在用户空间的运行时间（`UserTime`）也是`0`，这是因为它没有用户空间。它的父进程ID是0，即空闲进程。系统进程的映像文件名是`System`，这个名称是杜撰的，磁盘上并不存在System.exe。

在系统启动阶段，进程管理器在创建空闲进程后，便创建系统进程，因此，系统进程是系统创建的第二个进程。

在一个典型的Windows系统中，系统进程中有几十个乃至数百个系统线程在工作。随着Windows系统的发展，系统线程的个数也在增加。比如当笔者写作本书第1版的本节内容时，系统进程中有102个系统线程。在写作第2版时，系统线程的个数为218。如果把操作系统内核比作软件世界中的“政府”，那么系统线程便是政府中的“公务员”。10年时间里，系统线程数翻了一番，如此多的系统线程一方面表明Windows操作系统的机构和功能在增多，另一方面表示这个系统的固定开销和负担重了很多。

衡量操作系统效率的另一种方法是看系统进程的CPU时间。例如“任务管理器”窗口中，System进程的“CPU时间”接近17小时。虽然从“系统空闲进程”的时间来看，系统运行的总时间比较长，大约为832/4 = 208小时，但二者的比例仍然比较大，大约为(17/208) × 100% = 8.17%。参考清单3-1中的数据，系统总运行时间（70小时）里，System进程的CPU时间仅为3分钟，相应的比例为3/(70 × 60) × 100% = 0.07%，二者相差非常悬殊。

System进程的CPU时间是所有系统线程CPU时间的总和。例如，System进程中有190个系统线程，使用WinDBG可以列出这些线程，然后看每个线程所用时间，如此排查，可以找到排在前面的4个系统线程，如下所示。

（1）用于管理GPU内存的VidMM线程，工作函数为dxgmms2!VidMmWorkerThreadProc。

（2）负责调度GPU任务的VidSch线程，工作函数为dxgmms2!VidSchiWorkerThread，其内部会调用名为dxgmms2!VidSchiRun_PriorityTable的函数。

（3）内存管理器的工作集平衡线程，工作函数为KeBalanceSetManager，这个线程会扫描每个进程的页表，必要时把暂时不用的内存页交换到虚拟内存。

（4）内存管理器清零线程，工作函数为ZeroPageThread，是用来准备清零内存页的。在内核启动时，这个线程发起关键的执行体初始化动作；内核启动后，它退居二线，专门负责把内存页清零，当驱动程序需要已经清零的内存页时，满足需要。

## 3.5　内核空间的其他模块

在内核空间中，除了前面已经提到的NT内核文件、HAL模块以及内核文件直接依赖的KDCOM、CI、CLFS和BOOTVID模块，还有一些其他模块，本节将略作介绍，目的是让大家在调试时可以知道这些模块的大致用途。

Win32k.sys是Windows子系统的内核空间模块，与用户空间的CSRSS进程相互配合一起管理窗口世界，包括窗口对象、用户输入、消息分发、显示输出等。

DxgKrnl.sys是管理GPU的核心模块，其名字是DirectX Kernel的缩写，本书卷1的GPU篇对其有较详细的介绍。

KS.sys是管理流媒体的核心模块，使用图（graph）的模式管理音视频的编解码和处理节点，各个节点之间可以以引脚（pin）方式动态建立连接。

AFD.sys是网络套接字（WinSock）的内核空间接口驱动，是衔接内核空间的网络模块和用户空间网络模块的桥梁。

NDIS.sys是负责管理网卡驱动的核心驱动，与硬件厂商的网卡驱动配合一起管理网卡设备。

Wfplwf.sys是用于管理网络过滤驱动的核心模块，全称为Windows过滤平台（Windows Filtering Platform）轻量过滤器驱动程序（Lightweight Filter Driver）。

ACPI.sys是负责与平台固件接口的内核模块，是用于支持高级配置和电源接口（Advanced Configuration and Power Interface，ACPI）标准的核心驱动程序，其内部除了包含用于解释执行ACPI源语言（ACPI Source Language，ASL）脚本的解释器，还包含了一些ACPI标准设备的驱动程序，比如电源开关、笔记本电脑盖板检测器、风扇等。目前的Windows系统要求硬件平台必须支持ACPI标准。系统启动后，内核中的I/O管理器会根据注册表枚举设备树的一级节点，在一级节点中总是有ACPI节点，触发I/O管理器初始化ACPI驱动程序。ACPI驱动程序会在内存中搜索固件放在内存中的硬件信息表，其中一般都会包含PCI总线，触发I/O管理器初始化PCI总线的驱动程序，进一步枚举系统里的PCI设备。

PCI.sys是PCI总线的核心驱动程序，用于支持PCI总线标准、枚举和管理PCI总线上的设备。每当发现新的设备后，便尝试让即插即用（Plug and Play，PnP）管理器为其加载驱动程序，加载的驱动程序可能是Windows系统自带的（称为inbox），也可能是从硬件厂商那里获得和后来安装的。

NTFS.sys是NTFS文件系统的实现，用于访问NTFS格式的磁盘卷。

从操作系统架构的角度来看，上面介绍的这些模块都是以内核空间驱动程序的形式加载和管理的，广义上都可以称为驱动程序，虽然有些与硬件设备的关系不大。它们的角色都可以看作对内核的扩展，是内核的帮手，帮助内核一起管理软件世界。

## 3.6　NTDLL.DLL

前面几节带我们在内核空间中游历了一番，希望大家能对NT平台的大本营有一个较深的印象。从本节开始，我们将把目光转移到用户空间，介绍NT系统驻扎在用户空间中的“管理团队”。根据图3-1所示的架构，我们仍按横纵两个方向旅行，先横向介绍所有用户空间中都有的NTDLL.DLL，再纵向介绍子系统的关键进程。

### 3.6.1　角色

从操作系统架构设计的角度来看，NTDLL.DLL是NT内核派驻到用户空间的“使领馆”。总体来说，用户空间中运行着不同来源的各种代码，是不可信赖的，让内核直接与五花八门的用户代码交互的风险很大，设计也会复杂。另外，内核存在的意义就是要为用户代码服务，所以又必须与用户空间交互。NTDLL.DLL就是为了解决这个问题而设计的。NTDLL.DLL是沟通用户空间和内核空间的桥梁，用户空间的代码通过这个DLL来调用内核空间的系统服务。同时，NTDLL.DLL也是操作系统内核在用户空间中的“代理”，系统会在启动阶段便把它加载到内存中，并把它映射到所有用户进程的进程空间中，而且映射在相同的位置（虚拟地址）。当内核需要与用户空间配合时，它会使用这个DLL中的函数，因为只有这个DLL才存在于每个用户进程的用户空间的固定位置上。例如，上一节介绍的内核空间调用用户空间的逆向调用机制的用户态着陆点就是位于NTDLL.DLL中的。当内核分发异常时，如果需要在用户空间中寻找异常处理器，那么也会使用特殊的修改程序指针，飞回到NTDLL.DLL中，然后再继续分发，这将在第三篇中详细介绍。

因为NTDLL.DLL的特殊角色，当我们观察Windows系统中的进程时，会发现每个进程中都有这个DLL，而且它的线性地址是相同的。只有NTDLL.DLL具有这个特征，这种特性是NT内核的基因。

### 3.6.2　调用系统服务的桩函数

在NTDLL.DLL中有数百个名为NtXXX的函数，对于笔者使用的Windows 10，有500多个。这些函数有几个共同点——长度都很短，包含的指令也很相似，都是像清单3-2所列指令那样准备和执行系统调用指令，通常我们把这些函数叫作桩（stub）函数。

清单3-2　准备和执行系统调用指令

```
0:022> u ntdll!NtCreateEvent
ntdll!NtCreateEvent:
00007ffc`d442b290 4c8bd1      mov     r10,rcx
00007ffc`d442b293 b848000000  mov     eax,48h
00007ffc`d442b298 f604250803fe7f01 test    byte ptr [SharedUserData+0x308   
(00000000`7ffe0308)],1
00007ffc`d442b2a0 7503        jne   ntdll!NtCreateEvent+0x15 (00007ffc`d442b2a5)
00007ffc`d442b2a2 0f05        syscall
00007ffc`d442b2a4 c3          ret
00007ffc`d442b2a5 cd2e        int     2Eh
00007ffc`d442b2a7 c3          ret
```

第2章已经比较详细地介绍过系统调用，在这里复习一下。清单3-2的指令来自Windows 10版本的NTDLL.DLL，不同版本可能略有差异，但是大同小异，都执行SYSCALL这样的快速系统调用指令，或者传统的INT 2E指令。

在调试时，如果希望拦截系统调用，那么在NTDLL.DLL中设置断点是再合适不过的了。

### 3.6.3　映像文件加载器

在今天的计算机架构中，编译好的程序是存放在磁盘等外部存储器上的，运行时需要先加载到内存中，负责执行这个加载任务的操作系统部件一般称为映像加载器，或者就叫加载器（loader）。在Linux系统中，担任这个角色的是ld.so；在Windows系统中，它是NTDLL.DLL中的一部分，一般简称为LDR。

启动WinDBG，选择`File → Open Executables`，然后随便打开一个可执行文件，开始调试会话，便会看到LDR触发的初始断点。

```
(4f5c.3d78): Break instruction exception - code 80000003 (first chance)
ntdll!LdrpDoDebuggerBreak+0x30:
00007ffc`d445c93c cc              int     3
```

执行k命令观察栈回溯，便可以看到LDR准备为新进程服务的场景（清单3-3）。

清单3-3　LDR为准备新进程服务的场景

```
# Call Site
00 ntdll!LdrpDoDebuggerBreak
01 ntdll!LdrpInitializeProcess
02 ntdll!_LdrpInitialize
03 ntdll!LdrpInitialize
04 ntdll!LdrInitializeThunk
```

创建一个新的进程需要很多步骤，简单来说，早期的进程空间创建等准备工作是在父进程和内核空间中执行的。有了初始线程后，内核使用异步过程调用（Asynchronous Procedure Call，APC）机制让新的线程开始在用户空间运行，目的是在新进程自己的上下文中执行进程初始化工作，而负责这项工作的便是LDR。清单3-3显示的便是LDR准备大干一番的情景。最下面的LdrInitializeThunk是CPU从内核空间切换到用户空间的着陆点，名字中的Thunk代表“转接”之意。#01栈帧中的LdrpInitializeProcess是执行进程初始化的核心函数，它的代码较长，面对崭新的用户空间，有很多工作要做，比如分析程序所依赖的DLL，依次加载等。在大干一番之前，LDR先触发断点，目的是给开发人员一个较早的调试机会。

清单3-3中的函数都是以Ldr或_Ldr开头的，它们都属于LDR，又可分为两类，第4个字符为小写p的代表内部函数，第4个字符为大写形式的代表接口函数。这是NT内核团队经常使用的一种约定，在很多模块中都适用。

### 3.6.4　运行时库

NTDLL.DLL中数量最大的一类函数是运行时库（runtime library），它们都以Rtl开头，一般简称为RTL。在Windows 10版本的NTDLL.DLL中，RTL的函数有2000多个。

RTL的职责是提供基础的函数，包括字符串操作、时间、内存分配等。其中内存分配部分是一项很繁重的任务，一般称为堆管理器，我们将在第四篇中介绍NTDLL.DLL中的堆设施。

### 3.6.5　其他功能

除了上面提到的功能，NTDLL.DLL中还包含了一些其他支持功能，比如异常分发和调试支持等，我们将在本卷第三篇中介绍NTDLL.DLL的异常分发设施和调试支持。

## 3.7　环境子系统

对于大多数计算机用户来说，购买或者安装操作系统的目的是运行应用程序。这意味着，能够支持的应用程序类型越多，市场机会便越多。为了能够在Windows操作系统上运行多种类型的应用软件，Windows设计者在定义Windows架构时便设计了环境子系统（environment subsystem）的概念。不同类型的应用程序运行在不同的环境子系统中。这样，一个Windows系统中可以同时有多个不同的环境子系统，因此可以同时运行不同类型的应用程序。

Windows 2000支持如下3个环境子系统。

（1）POSIX子系统，POSIX是Portable Operating System Interface based on UNIX的缩写，用于运行符合POSIX标准的程序。系统目录（例如c:\winnt\system32）下的PSXSS.EXE（POSIX SubSystem）和PSXDLL.DLL是POSIX子系统的核心文件。

（2）OS/2子系统，支持16位的OS/2（1.2）程序，系统目录下的OS2.EXE、OS2SRV.EXE和OS2SS.EXE是OS/2子系统的核心文件。

（3）Windows子系统，即支持Windows程序运行的子系统，包括Windows子系统服务器进程（CSRSS.EXE）、子系统驱动程序（Win32K.SYS），子系统DLL（KERNEL32.DLL、USER32.DLL、ADVAPI32.DLL、GDI32.DLL），以及和设备相关的显示和打印驱动程序等。

尽管Windows系统的基本设计思想是支持多个独立的环境子系统，但是为了避免多个子系统都重复实现类似的功能，Windows子系统有着与其他子系统不同的地位，Windows子系统可以独立存在而且是系统中不可缺少的部分，其他子系统以Windows子系统为基础，必须依赖于Windows子系统，而且是按需要运行的（当第一次运行相应类型的应用程序时，才启动所需要的子系统）。

表3-1列出了Windows子系统的重要文件，大多数Windows API（Win32 API）是由Windows子系统的DLL输出的。

表3-1　Windows子系统的重要文件

| 名　　称 | 模式 | 描　　述 |
| --- | --- | --- |
| CSRSS.EXE | 用户模式 | Windows子系统服务进程的主程序 |
| ADVAPI32.DLL | 用户模式 | Windows子系统的DLL之一，包含如下API的入口：● 数据加密（以Crpt开头）；● 用户和账号管理（以Lsa开头）；● 注册表操作（以Reg开头）；● WMI（以Wmi开头）；● 终端服务（以Wts开头）。 |
| GDI32.DLL | 用户模式 | Windows子系统的DLL之一，包含各种图形文字绘制API（GDI）的入口，如TextOut()、BitBlt()等。其中大多数API是被转换为系统服务并发给内核模式的Windows子系统驱动程序（Win32K.SYS） |
| KERNEL32.DLL | 用户模式 | Windows子系统的DLL之一，包含如下API的入口：● 进程/线程管理，如CreateThread()；● 调试（以Debug开头）；● 文件操作，包括创建、打开、读写、搜索等；● 内存分配（以Local开头和Global开头）。其中大多数API是被转换为系统服务并发给内核态的执行体 |
| USER32.DLL | 用户模式 | Windows子系统的DLL之一，包含窗口管理、消息处理和用户输入API，如EndDialog()、BeginPaint()、SetWindowPos()、MessageBox()等。其中大多数API是被转换为系统服务并发给内核模式的Windows子系统驱动程序（Win32K.SYS） |


Windows XP去除了OS/2子系统，安装包中也不再包含POSIX子系统，但是可以免费下载。以下注册表表项包含了系统中各个子系统的情况。

```
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Session Manager\SubSystems
```

Windows Vista企业（Enterprise）版带有用于运行UNIX类应用程序的子系统，并提供了免费的SDK供用户开发这样的程序。

Windows 10引入了Linux环境子系统，支持Ubuntu、Fedora、Kali Linux等多种发行版本，可以运行原生的Linux应用程序，这将在第8章做详细介绍。

## 3.8　原生进程

在NT系统中，普通的应用程序都是属于某个环境子系统的，比如，记事本、画笔、WORD这些程序都属于Windows子系统，它们通过所谓的Windows API（Win32 API）与子系统交互。类似地，运行在NT系统中的Linux应用程序通过Linux API与Linux子系统交互。

那么有没有不依赖子系统的进程呢？比如，子系统服务进程是如何工作的呢？答案是肯定的。在NT系统中，有一类特殊的进程，它们不依赖任何子系统，通过特殊的私有接口直接与内核交互，通常把这类进程叫作原生（native）进程。

### 3.8.1　特点

因为依赖很少，所以原生程序在没有创建子系统的时候就能运行，比如著名的磁盘检查程序（autochk.exe）就是这样。但也正因为要能够运行在“简陋”的环境中，与普通的应用程序相比，原生程序有很多独特之处。首先，原生程序的入口是特殊的，不是main或者WinMain这样的入口函数，而是叫作NtProcessStartup的入口函数。其次，原生进程结束时，不能像main函数那样返回运行时，而是要调用NtTerminateProcess把自己结束掉。最后，原生程序如果要输出信息，那么一般只能使用NtDisplayString，其信息会像autochk那样直接打印到屏幕上。下面的简单代码代表了原生程序的框架。

```
void NTAPI NTProcessStartup ( PSTARTUP_ARGUMENT pArgument )
{
  long nResult = 0;
  // 执行具体操作
  NtDisplayString(″Hello Native Process\n″); 
  NtTerminateProcess ( NtCurrentProcess (), nResult );
}
```

另外，不能像开发普通程序那样开发原生程序，而需要使用开发驱动程序的DDK或者WDK环境来构建原生程序。编译好的原生程序也是标准的PE（Portable Execuable）格式，但是头信息中的Subsystem字段是特别的，值为0001（可以使用 PEView 观察原生程序中 Subsystem 字段的值），代表`IMAGE_SUBSYSTEM_NATIVE`。普通Windows GUI程序的类型为2（`IMAGE_SUBSYSTEM_WINDOWS_GUI`），控制台程序的类型为3（`IMAGE_SUBSYSTEM_WINDOWS_CUI`）。

下面分别介绍NT系统中几个重要的原生进程，首先介绍会话管理器。

### 3.8.2　SMSS

会话管理器的全称为会话管理器子系统（Session Manager Sub-System），一般简称为SMSS，其可执行文件为SMSS.exe。

NT系统启动时，当内核空间准备就绪后，内核会着手创建SMSS，让其带领用户空间的建设。清单3-4展示了在内核调试会话中观察到的创建SMSS进程的过程。

清单3-4　创建SMSS进程的过程

```
00 nt!MmCreateProcessAddressSpace
01 nt!PspAllocateProcess
02 nt!NtCreateUserProcess
03 nt!KiFastCallEntry
04 nt!ZwCreateUserProcess
05 nt!RtlpCreateUserProcess
06 nt!RtlCreateUserProcess
07 nt!StartFirstUserProcess
08 nt!Phase1InitializationDiscard
09 nt!Phase1Initialization
0a nt!PspSystemThreadStartup
0b nt!KiThreadStartup
```

观察清单3-4，下面的Phase1Initialization 代表内核启动过程中的执行体阶段1初始化（第4章）。StartFirstUserProcess 代表启动第一个用户进程。诚然，SMSS是NT启动过程中第一个以EXE方式创建的进程，此前的空闲进程和系统进程都是捏造出来的。最上面的MmCreateProcessAddressSpace代表创建进程的地址空间。

SMSS启动后，要执行一系列任务来开创用户世界。我们无法逐一描述，只选择几个比较重要的略作介绍。

（1）执行注册表中登记的程序。

（2）执行登记在PendingFileRenameOperations表键下的延迟文件操作（deferred file operation），比如删除杀毒软件发现但又无法立即删除的病毒程序。

（3）初始化虚拟内存文件（paging file）。

（4）加载Windows子系统的内核模块Win32K.SYS。

（5）创建Windows子系统服务进程CSRSS。

（6）创建显示登录桌面和执行登录过程的WinLogon进程。

其中，BootExecute表键的完整路径如下。

```
HKLM\System\CurrentControlSet\Control\Session Manager\BootExecute
```

其中的内容一般如下。

```
autocheck autochk *
```

这个注册表表键的类型是MULT_SZ，可以包含多行，每一行的第一部分为名字，后面是可执行程序的名称，再后面是命令行参数。SMSS得到这个信息后，便会创建autochk进程，检查磁盘，其过程如清单3-5所示。

清单3-5　SMSS创建磁盘检查进程

```
00 nt!MmCreateProcessAddressSpace
01 nt!PspAllocateProcess
02 nt!NtCreateUserProcess
03 nt!KiFastCallEntry
04 ntdll!KiFastSystemCallRet
05 ntdll!NtCreateUserProcess
06 ntdll!RtlpCreateUserProcess
07 ntdll!RtlCreateUserProcess
08 smss!SmpExecuteImage
09 smss!SmpInvokeAutoChk
0a smss!SmpExecuteCommand
0b smss!SmpLoadDataFromRegistry
0c smss!SmpInit
0d smss!wmain
0e smss!NtProcessStartupW_AfterSecurityCookieInitialized
0f ntdll!__RtlUserThreadStart
10 ntdll!_RtlUserThreadStart
```

上面的清单中，NtProcessStartupW是SMSS的入口。SmpLoadDataFromRegistry函数读取注册表，调用SmpExecuteCommand执行BootExecute表键指定的程序。

系统启动后，SMSS内部一般仍运行着两个线程：一个是主线程，它调用NtWaitForMultipleObjects永久等待CSRSS进程WinLogon进程，监视其是否意外退出，一旦退出，即触发蓝屏崩溃；另一个线程是SMSS的工作线程，它调用NtWaitForWorkViaWorkerFactory等待登录会话有关的任务。

### 3.8.3　CSRSS

CSRSS是Windows子系统的服务进程，是Windows子系统的“大内总管”。它的名字有点晦涩，字面意思是客户端/服务器子系统（Client-Server Sub-system）。这个名字代表了CSRSS内部的工作模式——经典的C/S（客户端/程序发起请求，服务器程序响应请求）模式。

CSRSS进程内会加载多个DLL形式的服务模块，向Windows子系统的程序提供服务。服务模块的数量和名字是可以通过注册表配置的，因此，某些病毒程序会冒充服务模块，目的是能混进可以长时间栖身的CSRSS进程空间。

CSRSS进程的服务除了登记Windows子系统进程的“生老病死”，还要管理窗口和GDI对象，分发窗口消息，打印消息和调试服务等。与清闲的SMSS相比，CSRSS的任务还是挺重的，我们将在第三篇介绍用户态调试模型时继续介绍CSRSS的调试服务。

## 3.9　本章总结

本章介绍了Windows操作系统的架构，带领大家分别在内核空间和用户空间中游历了一番。与第2章内容相呼应，本章前半部分介绍了内核空间，包括内核和HAL模块、空闲进程，系统进程，以及内核空间中的重要驱动程序和子系统，后半部分介绍了用户空间，包括衔接用户空间和内核空间的NTDLL，以及环境子系统和原生进程。
