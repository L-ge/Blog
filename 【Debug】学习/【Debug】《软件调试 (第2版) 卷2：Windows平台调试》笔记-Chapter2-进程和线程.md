# 第2章　进程和线程

## 2.1　任务

从操作系统的角度来讲，每个进程又称为一个任务（task），Windows任务管理器的名称即由此而来。但值得说明的是，“任务”这个词是不精确的，在不同上下文中的含义可能是不同的，可能是指进程，也可能是指线程。举例来说，在操作系统层面，“任务”一词常常是代表进程的，比如Windows是典型的多任务操作系统，是指系统中可以同时运行多个进程。在CPU手册中，很多时候是使用“任务”来指代线程的，比如著名的任务状态段（Task State Segment，TSS）就是用来记录每个线程的状态的。从这个角度来说，CPU一级的任务很多时候相当于进程中的一个线程，操作系统中的任务是指系统中运行着的各个进程，操作系统的每个任务对应于CPU的一个或多个任务。

## 2.2　进程资源

在Windows操作系统中，每个进程都拥有如下资源。

（1）一个虚拟的地址空间，一般称为进程空间，稍后将详细介绍。

（2）全局唯一的进程ID（又称客户ID，Client ID），简称PID。

（3）一个可执行映像（image），也就是该进程的程序文件（可执行文件）在内存中的表示。

（4）一个或多个线程。

（5）一个位于内核空间中的名为EPROCESS（executive process block，进程执行块）的数据结构，用以记录该进程的关键信息，包括进程的创建时间、映像文件名称等，详见下文。

（6）一个位于内核空间中的对象句柄表，用以记录和索引该进程所创建/打开的内核对象。操作系统根据该表格将用户模式下的句柄翻译为指向内核对象的指针。

（7）一个用于描述内存目录表起始位置的基地址，简称页目录基地址（DirBase），当CPU切换到该进程/任务时，会将该地址加载到页表基地址寄存器（x86之CR3，ARM之TTBR），这样当前进程的虚拟地址才会被翻译为正确的物理地址（卷1之2.7节和2.9节）。

（8）一个位于用户空间中的进程环境块（Process Environment Block，PEB），详见下文。

（9）一个访问令牌（access token），用于表示该进程的用户、安全组以及优先级。

上面列出的只是所有进程资源中比较常用的部分，本节后面会详细介绍与调试密切相关的资源。为了更好地理解每个项目，我们将结合WinDBG的进程观察命令（!process）来介绍，这样既易于理解，又可以帮助大家熟悉调试器的用法。也建议大家按下面的步骤亲自动手做一下。

格物

首先启动WinDBG程序，选择`File → Open Crash Dump`，然后选择本书实验文件中的`dumps\w10x64k.dmp`文件。在调试会话建立后，先执行`.symfix c:\symbols`，设置符号服务器，在有互联网的情况下执行`.reload`，加载模块和符号。

成功加载符号后，执行`!process 0 0`命令，列出系统内的所有进程。

```
6: kd> !process 0 0
**** NT ACTIVE PROCESS DUMP ****
PROCESS ffff84898203c440
    SessionId: none  Cid: 0004    Peb: 00000000  ParentCid: 0000
    DirBase: 001ad002  ObjectTable: ffffe18f2b814040  HandleCount: 2564.
    Image: System

PROCESS ffff8489820c6040
    SessionId: none  Cid: 0078    Peb: 00000000  ParentCid: 0004
    DirBase: 99d00002  ObjectTable: ffffe18f2b825b80  HandleCount:   0.
    Image: Registry
【省略很多行】
```

`!process 0 0`命令的第一个参数用来指定要显示的进程ID，0代表所有进程。第二个参数用来指定要显示的进程属性，0代表只显示最基本的进程属性。

可以在上面的命令后面加上程序文件名作为过滤条件，比如，下面的命令只显示wermgr进程的属性。

```
6: kd> !process 0  0 wermgr.exe
PROCESS ffff84899855d580
    SessionId: 0  Cid: 1840    Peb: cd79837000  ParentCid: 0644
    DirBase: 173e00002  ObjectTable: ffffe18f347c0d80  HandleCount:  16.
    Image: wermgr.exe
```

在以上命令结果中，第一行显示的是进程的EPROCESS结构的地址，接下来的三行显示的是进程的关键属性，下面将分别介绍。

进程的SessionId是指该进程所在的Windows会话（session）的ID号。当有多个用户同时登录时，Windows会为每个登录用户建立一个会话，每个会话有自己的WorkStation和Desktop。这样大家便可以工作在不同的“会话”中共用同一个Windows系统。对于典型的XP系统，当只有一个用户登录时，用户启动的程序和系统服务都运行在Session 0。当切换到另一个用户账号（Switch User，不是Log off）时，系统会建立Session 1，以此类推。为了提高系统服务的安全性，从Windows Vista开始，只允许系统服务运行在Session 0，系统启动后便会自动创建。当用户登录后，会创建另一个会话，一般为Session 1。因此，当用户登录到系统中后，总是会看到至少有两个会话，有两个Windows子系统进程（CSRSS）在运行。系统启动早期创建的几个特殊进程不属于任何会话，因此它们的SessionId为空（none），例如系统（System）进程便是如此。

Cid即进程ID，又叫Client ID（客户ID）。进程ID是标识进程的一个整数，很多用户态的函数用它作为标识进程的参数。在内核空间的代码里，主要使用EPROCESS指针来标识一个进程。

Parent Cid是父进程的进程ID，即创建该进程的那个进程的进程ID。

DirBase描述的是该进程顶级页表的位置，即当CPU切换到该进程执行时，CR3寄存器（对于x86 CPU）的内容。页目录基地址是将虚拟地址转换为物理地址的必需参数。我们在本书卷1中介绍如何将虚拟地址转换为物理地址时曾经提到过页目录基地址。在x86 CPU经典的32位分页模式中，顶级的页表叫页目录表，所以这个字段的名字就叫页目录基地址。对于64位分页模式，顶级页表有新的名字（页映射表，参阅卷1第2章），但这个字段的名字未变。

DirBase字段的位定义与当前使用的分页模式有关。在经典的32位分页模式中，DirBase的低12位总是0，高20位是该进程的页目录的页帧编号（Page Frame Number，PFN）。例如，如果DirBase的值是0x1f350000，那么它的PFN便是0x1f350。

在IA32e分页模式中，CR3的低12位的含义因CR4的PCIDE位（第17位）而不同。PCIDE是Process-Context Identifiers Enable的缩写，其为1时，CPU会缓存多个进程的页表信息，CR3的低12位是进程上下文ID号。在2018年后的Windows 10版本中，为了应对CPU的熔断（Meltdown）和幽灵（Spectry）漏洞，NT内核引入了名为KVA影子（Kernel Virtual Address Shadowing）的安全补丁，在这个补丁中会使用CPU的PCID功能。

在前面“格物”实验中使用的转储文件，已经启用了这个安全功能。

```
6: kd> dd nt!KiKvaShadow L1
fffff800`06465840  00000001
```

KVA影子有两种工作模式，在该转储文件中，使用的是模式1。

```
6: kd> dd nt!KiKvaShadowMode L1
fffff800`0644e4f8  00000001
```

模式1要求CPU具有PCID功能支持，从以下内核变量和cr4寄存器的位17都可以看出。

```
6: kd> dd nt!KiFlushPcid L1
fffff800`0644e249  00000001
6: kd> .formats cr4
Evaluate expression:
  Hex:     00000000`00170778
  Binary:  【省略全为0的高32位】00000000 00010111 00000111 01111000
```

PFN代表着物理内存页的编号，加上低12位便是物理地址。WinDBG的一些内存命令是使用PFN作为参数的，例如使用`!ptov`扩展命令加上PFN，便可以列出对应进程中所有物理地址到虚拟地址间的映射，如`!ptov 1f350`（输出结果非常长，从略）。

ObjectTable的含义是该进程的内核对象和句柄表格。Windows系统使用这个表格将句柄翻译为指向内核对象的指针。使用`!handle`命令可以查看句柄和对象信息。

在内核调试对话中，该命令的格式如下。

```
!handle [要显示的句柄索引 [显示标志 [进程ID或EPROCESS指针 [类型]]]]
```

比如`!handle 0 0 86a7d030`会显示出86a7d030进程的所有句柄概况。

在用户调试对话中，命令格式如下。

```
!handle [要显示的句柄索引 [显示标志 [类型]]]
```

使用`!object`命令可以进一步查看内核对象的信息。

HandleCount即该进程所使用的句柄个数，也就是ObjectTable所包含的表项数。

## 2.3　进程空间

为了保证系统中每个任务或进程的安全，Windows为不同的进程分配了独立的进程地址空间（process address space），常常简称为进程空间。进程空间是操作系统分配给每个进程的虚拟地址空间（virtual address space），每个进程运行在这个受操作系统保护的虚拟空间之中，它的地址指针指向的都是这个空间中的虚拟地址，根本无法指到另一个进程空间中，这样便保证了一个进程的数据和代码不会轻易受到其他进程的侵害，一个进程内的错误也不会波及同一系统内运行着的其他进程。或者说每个进程都在操作系统分配给它的虚拟空间中运行，它无法直接访问其他进程的空间，也不必担心自己的空间会被其他进程所侵占。

### 2.3.1　32位进程空间

在不同系统中，进程空间的大小可能不同。对于32位的Windows系统，每个进程的进程空间是4GB，即地址0x00000000到地址0xFFFFFFFF。为了高效地调用和执行操作系统的各种服务，Windows会把操作系统的内核数据和代码映射到所有进程的进程空间中。因此，4GB的进程空间总是被划分为两个区域：用户空间和内核空间，内核空间有时也被称作系统空间。

在32位系统中，用户空间和内核空间的默认大小各为2GB，低2GB为用户空间，高2GB为内核空间。Windows 2000 Advanced Server、Windows 2000 Datacenter Server、Windows XP和Windows Server 2003支持“3GB”启动选项使用户空间为3GB，以便满足数据库系统等某些特殊应用程序的需要。要使用该功能，除了要在启动配置文件（boot.ini）中设置“/3GB”启动选项，还需要在可执行映像的头信息中设置大用户空间标志（`IMAGE_FILE_LARGE_ADDRESS_AWARE` flag）。Windows XP和Windows Server 2003还支持/USERVA选项，该选项可以设定一个介于2GB～3GB（以MB为单位）的值用于定义用户空间的大小。由于对于大多数32位系统和大多数进程，用户空间大小是2GB，因此如不特别指出，本书讨论的都是用户空间为2GB的情况，也就是地址0x00000000到地址0x7FFFFFFF为用户空间，地址0x80000000到地址0xFFFFFFFF为内核空间。

简单来说，用户空间是给应用程序的模块使用的，内核空间是给操作系统内核使用的。考虑到所有的应用程序都需要使用内核提供的服务，所以内核空间是统一的，只有一个，会映射到所有应用程序的进程空间中。从这个角度来讲，用户空间是独立的，每个进程都有自己的一个空间，而内核空间是共享的，所有进程共享一个空间。

因为内核空间是共享的，要为系统中的所有进程服务，所以不允许被某个进程任意访问和破坏。为了保护内核空间，操作系统会利用CPU的硬件保护机制。从特权级别的角度看，内核空间和用户空间具有不同的特权级别，内核空间的特权级别高于用户空间。

用户空间和内核空间的另一个差别是，在用户空间中运行的应用程序代码可以很容易地显示出界面，与用户交互。而内核空间中的代码，出于多种原因，一般是不能直接显示信息的（崩溃和启动时可以显示有限的信息）。因此，简单地说，用户空间是有界面和可见的，而内核空间是没有界面和不可见的。

### 2.3.2　64位进程空间

64位系统下，用户空间和内核空间都增大了很多，具体数值因硬件平台和系统版本而不同。比如在IA-64（安腾）平台上，用户空间为7152GB（约7TB），内核空间为6144GB。

对于 x64 CPU，在早期的 64 位Windows系统中使用的是44位线性地址，进程地址空间的总大小为16TB，用户空间的范围是0x0～0x7FF FFFFFFFF，大小为8192GB（8TB），内核空间的范围是0xFFFFF800 00000000～0xFFFFFFFF FFFFFFFF，大小也是8TB。大约从Windows 7的x64版本开始，进程空间的总大小扩大为256TB（48位线性地址），用户空间和内核空间各为128TB，用户空间的范围为0x0～0x7FFF FFFFFFFF。

## 2.4　EPROCESS结构

从数据结构的角度来看，NT内核使用一个名为EPROCESS的庞大结构来描述进程。在Windows系统中，每个进程都会有一个EPROCESS结构。

在前面的`!process 0 0`命令的执行结果中，每个进程有3行信息，第一行信息中，PROCESS后面的地址指向的便是进程的EPROCESS结构，使用dt命令（显示类型结构命令）可以观察该结构的各个字段和取值。清单2-1列出了在32位Windows XP内核中EPROCESS结构的定义（//后是注释）。

清单2-1　EPROCESS结构的定义

```
lkd> dt _EPROCESS 86a7d030
   +0x000 Pcb                   : _KPROCESS  // 内核进程块，用来记录与任务调度有关的信息
   +0x06c ProcessLock           : _EX_PUSH_LOCK
   +0x070 CreateTime           : _LARGE_INTEGER 0x1c6aec5`7d9a68a0  // 创建时间
   +0x078 ExitTime              : _LARGE_INTEGER 0x0 // 退出时间      
   +0x080 RundownProtect        : _EX_RUNDOWN_REF   
   +0x084 UniqueProcessId       : 0x00000f20   // 进程ID
   +0x088 ActiveProcessLinks   : _LIST_ENTRY [ 0x87a5f0b8 - 0x878b80b8 ]
   +0x090 QuotaUsage            : [3] 0xa50
   +0x09c QuotaPeak             : [3] 0xa50
   +0x0a8 CommitCharge          : 0x15a
   +0x0ac PeakVirtualSize       : 0x1e61000
   +0x0b0 VirtualSize           : 0x1cc5000   // 
   +0x0b4 SessionProcessLinks   : _LIST_ENTRY [ 0x87a5f0e4 - 0x861080e4 ]
   +0x0bc DebugPort             : (null)     // 用户态调试端口，参见下文
   +0x0c0 ExceptionPort         : 0xe27767a0   // 异常端口
   +0x0c4 ObjectTable           : 0xe1771668 _HANDLE_TABLE  // 对象句柄表
   +0x0c8 Token                 : _EX_FAST_REF   // 访问令牌
   +0x0cc WorkingSetLock        : _FAST_MUTEX
   +0x0ec WorkingSetPage        : 0x653
   +0x0f0 AddressCreationLock   : _FAST_MUTEX
   +0x110 HyperSpaceLock        : 0
   +0x114 ForkInProgress        : (null) 
   +0x118 HardwareTrigger       : 0
   +0x11c VadRoot               : 0x871126d0   // 虚拟地址描述符二叉树的根节点
   +0x120 VadHint               : 0x86dc1f78 
   +0x124 CloneRoot             : (null) 
   +0x128 NumberOfPrivatePages  : 0x110
   +0x12c NumberOfLockedPages   : 0
   +0x130 Win32Process          : 0xe13b8de8 
   +0x134 Job                   : (null) 
   +0x138 SectionObject         : 0xe1d16900 
   +0x13c SectionBaseAddress   : 0x01000000 
   +0x140 QuotaBlock            : 0x8768c480 _EPROCESS_QUOTA_BLOCK
   +0x144 WorkingSetWatch       : (null) 
   +0x148 Win32WindowStation   : 0x00000030 
   +0x14c InheritedFromUniqueProcessId: 0x00000d98 
   +0x150 LdtInformation        : (null) 
   +0x154 VadFreeHint           : (null) 
   +0x158 VdmObjects            : (null) 
   +0x15c DeviceMap             : 0xe27058d0 
   +0x160 PhysicalVadList       : _LIST_ENTRY [ 0x86a7d190 - 0x86a7d190 ]
   +0x168 PageDirectoryPte      : _HARDWARE_PTE_X86
   +0x168 Filler                : 0
   +0x170 Session               : 0xf7c9b000   // 所属会话对象
   +0x174 ImageFileName         : [16]  "notepad.exe"
   +0x184 JobLinks              : _LIST_ENTRY [ 0x0 - 0x0 ]
   +0x18c LockedPagesList       : (null) 
   +0x190 ThreadListHead  : _LIST_ENTRY [ 0x861fe25c - 0x861fe25c ] // 线程列表
   +0x198 SecurityPort          : (null) 
   +0x19c PaeTop                : (null) 
   +0x1a0 ActiveThreads         : 1
   +0x1a4 GrantedAccess         : 0x1f0fff
   +0x1a8 DefaultHardErrorProcessing: 1   // 参见13.1节
   +0x1ac LastThreadExitStatus  : 0
   +0x1b0 Peb                   : 0x7ffdf000 _PEB   // 进程环境块
   +0x1b4 PrefetchTrace         : _EX_FAST_REF
   +0x1b8 ReadOperationCount    : _LARGE_INTEGER 0x1
   +0x1c0 WriteOperationCount   : _LARGE_INTEGER 0x0
   +0x1c8 OtherOperationCount   : _LARGE_INTEGER 0x9a
   +0x1d0 ReadTransferCount    : _LARGE_INTEGER 0x46da
   +0x1d8 WriteTransferCount   : _LARGE_INTEGER 0x0
   +0x1e0 OtherTransferCount   : _LARGE_INTEGER 0x1e6
   +0x1e8 CommitChargeLimit    : 0
   +0x1ec CommitChargePeak      : 0x23a
   +0x1f0 AweInfo               : (null) 
   +0x1f4 SeAuditProcessCreationInfo: _SE_AUDIT_PROCESS_CREATION_INFO
   +0x1f8 Vm                    : _MMSUPPORT
   +0x238 LastFaultCount        : 0
   +0x23c ModifiedPageCount   : 0
   +0x240 NumberOfVads          : 0x42
   +0x244 JobStatus             : 0
   +0x248 Flags                 : 0xd0840
   +0x248 CreateReported        : 0y0
   +0x248 NoDebugInherit        : 0y0
   +0x248 ProcessExiting        : 0y0   // 正在退出标志
   +0x248 ProcessDelete         : 0y0   // 删除标志
   +0x248 Wow64SplitPages       : 0y0 
   +0x248 VmDeleted             : 0y0
   +0x248 OutswapEnabled        : 0y1
   +0x248 Outswapped            : 0y0
   +0x248 ForkFailed               : 0y0
   +0x248 HasPhysicalVad           : 0y0
   +0x248 AddressSpaceInitialized  : 0y10
   +0x248 SetTimerResolution       : 0y0
   +0x248 BreakOnTermination       : 0y0
   +0x248 SessionCreationUnderway  : 0y0
   +0x248 WriteWatch               : 0y0
   +0x248 ProcessInSession         : 0y1
   +0x248 OverrideAddressSpace     : 0y0
   +0x248 HasAddressSpace          : 0y1
   +0x248 LaunchPrefetched         : 0y1
   +0x248 InjectInpageErrors       : 0y0
   +0x248 Unused                   : 0y00000000000 (0)
   +0x24c ExitStatus               : 0x103
   +0x250 NextPageColor            : 0x5504
   +0x252 SubSystemMinorVersion     : 0 ''
   +0x253 SubSystemMajorVersion     : 0x4 ''
   +0x252 SubSystemVersion          : 0x400   // 环境子系统版本号
   +0x254 PriorityClass             : 0x2 ''
   +0x255 WorkingSetAcquiredUnsafe    : 0 ''
```

从清单2-1中可以看到，EPROCESS结构几乎包括了进程的所有关键信息和重要“资产”，比如与调试密切相关的`DebugPort`和`ExceptionPort`（见第9章），指向进程的虚拟地址描述符（VAD）二叉树根节点的`VadRoot`（使用`!vad` 命令可以列出这些描述符），以及指向进程内所有线程列表表头的`ThreadListHead`，进程环境块地址等。我们将在下文和以后的章节中逐步介绍其中的重要字段。

在WinDBG中，可以用`!process`命令加上EPROCESS结构的地址来显示该进程的关键信息，比如：

```
lkd> !process 86a7d030
PROCESS 86a7d030  SessionId: 0  Cid: 0f20    Peb: 7ffdf000  ParentCid: 0d98
    DirBase: 1f350000  ObjectTable: e1771668  HandleCount:  33.
    Image: notepad.exe
    VadRoot 871126d0 Vads 66 Clone 0 Private 272. Modified 0. Locked 0.
    DeviceMap e27058d0
    Token                           e24d8510
    ElapsedTime                     00:10:35.754
    UserTime                        00:00:00.010
    KernelTime                     00:00:00.060
    QuotaPoolUsage[PagedPool]      30276
    QuotaPoolUsage[NonPagedPool]   2640
    Working Set Sizes (now,min,max)  (966, 50, 345) (3864KB, 200KB, 1380KB)
    PeakWorkingSetSize             966
    VirtualSize                    28 Mb
    PeakVirtualSize                30 Mb
    PageFaultCount                 996
    MemoryPriority                 BACKGROUND
    BasePriority                   8
    CommitCharge                   346

    THREAD 861fe030  Cid 0f20.0f90  Teb: 7ffde000 Win32Thread: e1d1d650 WAIT: 
(WrUserRequest) UserMode Non-Alertable
            86f45030  SynchronizationEvent
        Not impersonating
        DeviceMap              e27058d0
        Owning Process         86a7d030    Image: notepad.exe
        Wait Start TickCount   26300875    Ticks: 62664 (0:00:10:27.542)
        Context Switch Count   424         LargeStack
        UserTime               00:00:00.0000
        KernelTime             00:00:00.0060
        Start Address kernel32!BaseProcessStartThunk (0x77e813f2)
        Win32 Start Address WinDBG!`string' (0x01006ae0)
        Stack Init f7921000 Current f7920c20 Base f7921000 Limit f791b000 Call 0
        Priority 10 BasePriority 8 PriorityDecrement 0 DecrementCount 16
        Kernel stack not resident.
```

随着NT内核的不断发展，EPROCESS结构的大小也不断增大。在上面的Windows 10转储文件和笔者写作时使用的Windows 10中，这个结构的大小都是2120（0x848）字节，在WinDBG中，可以使用`?? sizeof(_EPROCESS)`命令来观察。为了节约篇幅，我们省去了当前版本EPROCESS结构的详细定义，但在以后的内容中还会经常谈到其中的字段，希望大家在阅读到那些内容时，可以使用WinDBG的dt命令来观察。

EPROCESS结构中的`Token`字段记录这个进程的TOKEN结构的地址，进程的很多与安全有关的信息都是记录在`Token`结构中的。在清单2-1所示的显示结果中找到`Token`字段的值，然后使用`!Token`命令便可以观察`Token`的详细信息（清单2-2）。

清单2-2　Token的详细信息

```
lkd> !Token e24d8510
_TOKEN e24d8510
TS Session ID: 0
User: S-1-5-21-1757981266-725345543-1404487317-19316
Groups: 
 00 S-1-5-21-1757981266-725345543-1404487317-513
    Attributes - Mandatory Default Enabled 
 01 S-1-1-0
    Attributes - Mandatory Default Enabled 
 …
Primary Group: S-1-5-21-1757981266-725345543-1404487317-513
Privs: 
 00 0x000000017 SeChangeNotifyPrivilege                Attributes - Enabled Default
 01 0x000000008 SeSecurityPrivilege                   Attributes - 
 02 0x000000011 SeBackupPrivilege                     Attributes - 
 03 0x000000012 SeRestorePrivilege                    Attributes - 
 04 0x00000000c SeSystemtimePrivilege                 Attributes - 
 05 0x000000013 SeShutdownPrivilege                   Attributes - 
 06 0x000000018 SeRemoteShutdownPrivilege             Attributes - 
 07 0x000000009 SeTakeOwnershipPrivilege              Attributes - 
 08 0x000000014 SeDebugPrivilege                      Attributes - 
 09 0x000000016 SeSystemEnvironmentPrivilege          Attributes - 
 10 0x00000000b SeSystemProfilePrivilege              Attributes - 
 11 0x00000000d SeProfileSingleProcessPrivilege       Attributes - 
 12 0x00000000e SeIncreaseBasePriorityPrivilege       Attributes - 
 13 0x00000000a SeLoadDriverPrivilege                 Attributes - Enabled 
 14 0x00000000f SeCreatePagefilePrivilege             Attributes - 
 15 0x000000005 SeIncreaseQuotaPrivilege              Attributes - 
 16 0x000000019 SeUndockPrivilege                     Attributes - Enabled 
 17 0x00000001c SeManageVolumePrivilege               Attributes - 
Authentication ID:      (0,1b496)
Impersonation Level:    Anonymous
TokenType:               Primary
Source: User32           TokenFlags: 0x9 ( Token in use )
Token ID: 14cb4cc       ParentToken ID: 0
Modified ID:             (0, 14cb4ce)
RestrictedSidCount: 0    RestrictedSids: 00000000
```

也可以使用`dt nt!_TOKEN`加上令牌对象的地址（如e24d8510）来观察令牌对象。详细解释令牌有关的内容超出了本书的范围，感兴趣的读者可以阅读参考资料（Mark E. Russinovich and David A. Solomon. Windows Internals 4th Edition. Microsoft Press, 2005.）的第8章。

## 2.5　PEB

PEB（Process Environment Block）的全称是进程环境块，它包含了进程的大多数用户模式信息。与EPROCESS结构位于内核空间中不同，PEB是在内核模式建立后映射到用户空间的。因此，在一个系统中，多个进程的PEB地址可能是同一个值。

使用`dt _PEB`命令可以显示出PEB结构的字段及其当前值。因为PEB的地址位于用户空间，所以既可以在内核调试会话也可以在用户调试会话中观察PEB。在内核调试会话中观察PEB时，应该先用`.process`命令设置当前进程（清单2-3）。

清单2-3　PEB结构

```
lkd> .process 86a7d030
Implicit process is now 86a7d030
lkd> dt _PEB 7ffdf000
   +0x000 InheritedAddressSpace : 0 ''
   +0x001 ReadImageFileExecOptions : 0 ''
   +0x002 BeingDebugged       : 0 ''         // 是否在被调试
   +0x003 SpareBool         : 0 ''
   +0x004 Mutant            : 0xffffffff 
   +0x008 ImageBaseAddress      : 0x01000000      // 执行映像（EXE）的基地址
   +0x00c Ldr                   : 0x00191e90 _PEB_LDR_DATA
   +0x010 ProcessParameters   : 0x00020000 _RTL_USER_PROCESS_PARAMETERS
   +0x014 SubSystemData         : (null) 
   +0x018 ProcessHeap           : 0x00090000      // 进程堆，参见23.1节
   +0x01c FastPebLock           : 0x77fc49e0 _RTL_CRITICAL_SECTION
   +0x020 FastPebLockRoutine   : 0x77f5b2a0 
   +0x024 FastPebUnlockRoutine   : 0x77f5b380 
   +0x028 EnvironmentUpdateCount   : 1
   +0x02c KernelCallbackTable   : 0x77d429b8 
   +0x030 SystemReserved        : [1] 0
   +0x034 ExecuteOptions        : 0y00
   +0x034 SpareBits             : 0y000000000000000000000000000000 (0)
   +0x038 FreeList              : (null) 
   +0x03c TlsExpansionCounter   : 0
   +0x040 TlsBitmap             : 0x77fc4680 
   +0x044 TlsBitmapBits         : [2] 0x7ffff
   +0x04c ReadOnlySharedMemoryBase   : 0x7f6f0000 
   +0x050 ReadOnlySharedMemoryHeap   : 0x7f6f0000 
   +0x054 ReadOnlyStaticServerData   : 0x7f6f0688  -> (null) 
   +0x058 AnsiCodePageData      : 0x7ffa0000 
   +0x05c OemCodePageData       : 0x7ffa0000 
   +0x060 UnicodeCaseTableData   : 0x7ffd1000 
   +0x064 NumberOfProcessors   : 1          // CPU个数
   +0x068 NtGlobalFlag          : 0x24400       // 全局标志
   +0x070 CriticalSectionTimeout   : _LARGE_INTEGER 0xffffe86d`079b8000
   +0x078 HeapSegmentReserve   : 0x100000     // 默认进程堆的总保留空间，1MB
   +0x07c HeapSegmentCommit   : 0x2000        // 默认进程堆的已提交空间
   +0x080 HeapDeCommitTotalFreeThreshold : 0x10000
   +0x084 HeapDeCommitFreeBlockThreshold : 0x1000
   +0x088 NumberOfHeaps         : 0xc       //堆的个数
   +0x08c MaximumNumberOfHeaps   : 0x10       //堆的最多个数
   +0x090 ProcessHeaps          : 0x77fc5a80 -> 0x00090000  //保存堆句柄的数组地址
   +0x094 GdiSharedHandleTable   : 0x003b0000   //GDI共享句柄表
   +0x098 ProcessStarterHelper   : (null) 
   +0x09c GdiDCAttributeList   : 0x14
   +0x0a0 LoaderLock            : 0x77fc1774 
   +0x0a4 OSMajorVersion        : 5      //操作系统主版本号
   +0x0a8 OSMinorVersion        : 1      //操作系统子版本号
   +0x0ac OSBuildNumber         : 0xa28    //操作系统构建号，即2600
   +0x0ae OSCSDVersion          : 0x100    //Service Pack版本号
   +0x0b0 OSPlatformId          : 2      //系统类别，2代表NT，1代表9x，3代表Windows CE
   +0x0b4 ImageSubsystem        : 2      //环境子系统ID
   +0x0b8 ImageSubsystemMajorVersion: 4       //环境子系统主版本号
   +0x0bc ImageSubsystemMinorVersion: 0xa     //环境子系统子版本号
   +0x0c0 ImageProcessAffinityMask: 0
   +0x0c4 GdiHandleBuffer       : [34] 0
   +0x14c PostProcessInitRoutine   : (null) 
   +0x150 TlsExpansionBitmap   : 0x77fc4660 
   +0x154 TlsExpansionBitmapBits   : [32] 0
   +0x1d4 SessionId             : 0      //所属会话的ID
   +0x1d8 AppCompatFlags        : _ULARGE_INTEGER 0x0
   +0x1e0 AppCompatFlagsUser   : _ULARGE_INTEGER 0x0
   +0x1e8 pShimData             : (null) 
   +0x1ec AppCompatInfo         : (null) 
   +0x1f0 CSDVersion            : _UNICODE_STRING "Service Pack 1"
   +0x1f8 ActivationContextData   : 0x00080000 
   +0x1fc ProcessAssemblyStorageMap         : 0x000930f8 
   +0x200 SystemDefaultActivationContextData   : 0x00070000 
   +0x204 SystemAssemblyStorageMap   : (null) 
   +0x208 MinimumStackCommit      : 0
```

也可以使用`!peb`扩展命令来观察进程环境块，比如使用`!peb 7ffdf000`命令便可以显示位于`0x 7ffdf000`处的PEB结构。

在调试应用程序时，常常通过观察PEB来了解进程的很多全局信息，比如进程是否在被调试、进程的默认堆、进程中的模块列表、进程的命令行等。

## 2.6　内核模式和用户模式

根据前面的介绍，NT内核会把操作系统的代码和数据映射到系统中所有进程的内核空间中。这样，每个进程内的应用程序代码便可以很方便地调用内核空间中的系统服务。这里的“很方便”有多层含义，一方面是内核代码和用户代码在一个地址空间中，应用程序调用系统服务时不需要切换地址空间，另一方面是整个系统中内核空间的地址是统一的，编写内核空间的代码时会简单很多。但是，如此设计也带来一个很大的问题，那就是用户空间中的程序指针可以指向内核空间中的数据和代码，因此必须防止用户代码破坏内核空间中的操作系统。怎么做呢？答案是利用权限控制来实现对内核空间的保护。

### 2.6.1　访问模式

Windows定义了两种访问模式（access mode）——用户模式（user mode，也称为用户态）和内核模式（kernel mode，也称为内核态）。应用程序（代码）运行在用户模式下，操作系统代码运行在内核模式下。内核模式对应于处理器的最高权限级别（不考虑虚拟机情况），在内核模式下执行的代码可以访问所有系统资源并具有使用所有特权指令的权利。相对而言，用户模式对应于较低的处理器优先级，在用户模式下执行的代码只可以访问系统允许其访问的内存空间，并且没有使用特权指令的权利。

本书卷1介绍过，IA-32处理器定义了4种特权级别（privilege level），或者称为环（ring），分别为0、1、2、3，优先级0（环0）的特权级别最高。处理器在硬件一级保证高优先级的数据和代码不会被低优先级的代码破坏。Windows系统使用了IA-32处理器所定义的4种优先级中的两种，优先级3（环3）用于用户模式，优先级0用于内核模式。之所以只使用了其中的两种，主要是因为有些处理器只支持两种优先级，比如Compaq Alpha处理器。值得说明的是，对于x86处理器来说，并没有任何寄存器表明处理器当前处于何种模式（或优先级）下，优先级只是代码或数据所在的内存段或页的一个属性，参见卷1的2.6节和2.7节。

因为内核模式下的数据和代码具有较高的优先级，所以用户模式下的代码不可以直接访问内核空间中的数据，也不可以直接调用内核空间中的任何函数或例程。任何这样的尝试都会导致保护性错误。也就是说，即使用户空间中的代码指针正确指向了要访问的数据或代码，但一旦访问发生，那么处理器会检测到该访问是违法的，会停止该访问并产生保护性异常（#GP）。

虽然不可以直接访问，但是用户程序可以通过调用系统服务来间接访问内核空间中的数据或间接调用、执行内核空间中的代码。当调用系统服务时，主调线程会从用户模式切换到内核模式，调用结束后再返回到用户模式，也就是所谓的模式切换。在线程的KTHREAD结构中，定义了UserTime和KernelTime两个字段，分别用来记录这个线程在用户模式和内核模式的运行时间（以时钟中断次数为单位）。模式切换是通过软中断或专门的快速系统调用（fast system call）指令来实现的。下面通过一个例子来分别介绍这两种切换机制。

### 2.6.2　使用INT 2E切换到内核模式

下面展示了在Windows 2000中通过INT 2E从应用程序调用ReadFile() API的过程。因为ReadFile() API是从Kernel32.dll导出的，所以我们看到该调用首先转到Kernel32.dll中的ReadFile()函数，ReadFile()函数在对参数进行简单检查后便调用NtDll.dll中的NtReadFile()函数。

```
Win32应用程序：包含ReadFile()调用的函数
↓
Kernel32.dll：Kernel32!ReadFile()
↓
NtDll.dll：NtDll!NtReadFile()

用户模式：↓ INT 2E
内核模式：↑ IRET
↓
NtOSKrnl.exe：KiSystemService()
↓
NtOSKrnl.exe：Nt!NtReadFile()
```

通过反汇编可以看到，NtDll.dll中的`NtReadFile()`函数非常简短，首先将`ReadFile()`对应的系统服务号（0xa1，与版本有关）放入EAX寄存器中，将参数指针放入EDX寄存器中，然后便通过INT n指令发出调用。这里要说明的一点是，虽然每个系统服务都具有唯一的号码，但微软公司没有公开这些服务号，也不保证这些号码在不同的Windows版本中会保持一致。

```
ntdll!NtReadFile: // Windows 2000
77f8fb5d b8a1000000    mov      eax,0xa1
77f8fb62 8d542404      lea      edx,[esp+0x4]
77f8fb66 cd2e          int      2e
77f8fb68 c22400        ret      0x24
```

在WinDBG下通过`!idt 2e`命令可以看到2e号向量对应的服务例程是`KiSystemService()`。`KiSystemService()`是内核态中专门用来分发系统调用的例程。

```
lkd> !idt 2e
Dumping IDT:
2e:   804db1ed nt!KiSystemService
```

Windows将2e号向量专门用于系统调用，在启动早期初始化中断描述符表（Interrupt Descriptor Table，IDT）时（见第11章）便注册好了合适的服务例程。因此当NTDll.DLL中的NtReadFile()发出INT 2E指令后，CPU便会通过IDT找到`KiSystemService()`函数。因为`KiSystemService()`函数是位于内核空间的，所以CPU在把执行权交给`KiSystemService()`函数前，会做好从用户模式切换到内核模式的各种工作，包括：

（1）权限检查，即检查源位置和目标位置所在的代码段权限，核实是否可以转移；

（2）准备内核模式使用的栈，为了保证内核安全，所有线程在内核态执行时都必须使用位于内核空间的内核栈（kernel stack），内核栈的大小一般为8KB或12KB。

`KiSystemService()`会根据服务ID从系统服务分发表（System Service Dispatch Table）中查找到要调用的服务函数地址和参数描述，然后将参数从用户态栈复制到该线程的内核栈中，最后`KiSystemService()`调用内核中真正的`NtReadFile()`函数，执行读文件的操作，操作结束后会返回到`KiSystemService()`，`KiSystemService()`会将操作结果复制回该线程用户态栈，最后通过IRET指令将执行权交回给NtDll.dll中的`NtReadFile()`函数（继续执行INT 2E后面的那条指令）。

通过INT 2E进行系统调用时，CPU必须从内存中分别加载门描述符和段描述符才能得到`KiSystemService()`的地址，即使门描述符和段描述符已经在高速缓存中，CPU也需要通过“内存读（memory read）”操作从高速缓存中读出这些数据，然后进行权限检查。

### 2.6.3　快速系统调用

因为系统调用是非常频繁的操作，所以如果能减少这些开销还是非常有意义的。可以从两个方面来降低开销：一是把系统调用服务例程的地址放到寄存器中以避免读IDT这样的内存操作，因为读寄存器的速度比读内存的速度要快很多；二是避免权限检查，也就是使用特殊的指令让CPU省去那些对系统服务调用来说根本不需要的权限检查。奔腾II处理器引入的SYSENTER/SYSEXIT指令正是按这一思路设计的。AMD K7引入的SYSCALL/SYSRETURN指令也是为这一目的而设计的。相对于INT 2E，使用这些指令可以加快系统调用的速度，因此利用这些指令进行的系统调用称为快速系统调用。

下面我们介绍Windows系统是如何利用IA-32处理器的SYSENTER/SYSEXIT指令（从奔腾II开始）实现快速系统调用的。首先，Windows 2000或之前的Windows系统不支持快速系统调用，它们只能使用前面介绍的INT 2E方式进行系统调用。Windows XP和Windows Server 2003或更新的版本在启动过程中会通过CPUID指令检测CPU是否支持快速系统调用指令（EDX寄存器的SEP标志位）。如果CPU不支持这些指令，那么仍使用INT 2E方式。如果CPU支持这些指令，那么Windows系统便会决定使用新的方式进行系统调用，并做好如下准备工作。

（1）在全局描述符表（GDT）中建立4个段描述符，分别用来描述供SYSENTER指令进入内核模式时使用的代码段（CS）和栈段（SS），以及SYSEXIT指令从内核模式返回用户模式时使用的代码段和栈段。这4个段描述符在GDT中的排列应该严格按照以上顺序，只要指定一个段描述符的位置便能计算出其他的。

（2）设置表2-1中专门用于系统调用的MSR（关于MSR的详细介绍见卷1的2.4.3节），`SYSENTER_EIP_MSR`用于指定新的程序指针，也就是SYSENTER指令要跳转到的目标例程地址。Windows系统会将其设置为KiFastCallEntry的地址，因为KiFastCallEntry例程是Windows内核中专门用来受理快速系统调用的。`SYSENTER_CS_MSR`用来指定新的代码段，也就是KiFastCallEntry所在的代码段。`SYSENTER_ESP_MSR`用于指定新的栈指针（ESP）。新的栈段是由`SYSENTER_CS_MSR`的值加8得来的。

（3）将一小段名为SystemCallStub的代码复制到SharedUserData内存区，该内存区会被映射到每个Win32进程的进程空间中。这样当应用程序每次进行系统调用时，NTDll.DLL中的残根（stub）函数便调用这段SystemCallStub代码。SystemCallStub的内容因系统硬件的不同而不同，对于IA-32处理器，该代码使用SYSENTER指令，对于AMD处理器，该代码使用SYSCALL指令。

表2-1　供SYSENTER指令使用的MSR

| MSR名称 | MSR地址 | 用　　途 |
| --- | --- | --- |
| `SYSENTER_CS_MSR` | 174h | 目标代码段的CS选择子 |
| `SYSENTER_ESP_MSR` | 175h | 目标ESP |
| `SYSENTER_EIP_MSR` | 176h | 目标EIP |


例如在配有Pentium M CPU的Windows XP系统上，以上3个寄存器的值分别为：

```
lkd> rdmsr 174
msr[174] = 00000000 00000008
lkd> rdmsr 175
msr[175] = 00000000 bacd8000
lkd> rdmsr 176
msr[176] = 00000000 8053cad0
```

其中`SYSENTER_CS_MSR`的值为8，这是Windows系统的内核代码段的选择子，即常量`KGDT_R0_CODE`的值。WinDBG帮助文件中关于dg命令的说明中列出了这个常量。`SYSENTER_EIP_MSR`的值是8053cad0，检查nt内核中`KiFastCallEntry`函数的地址。

```
lkd> x nt!KiFastCallEntry
8053cad0 nt!KiFastCallEntry = <no type information>
```

可见，Windows把快速系统调用的目标指向内核代码段中的`KiFastCallEntry`函数。

通过反汇编Windows XP下NTDll.DLL中的`NtReadFile()`函数，可以看到SystemCallStub被映射到进程的0x7ffe0300位置。与前面Windows 2000下的版本相比，容易看到该服务的系统服务号码在这两个版本间是不同的。

```
kd> u ntdll...
ntdll!NtReadFile: // Windows XP
77f5bfa8 b8b7000000       mov     eax,0xb7
77f5bfad ba0003fe7f       mov     edx,0x7ffe0300
77f5bfb2 ffd2             call edx {SharedUserData!SystemCallStub (7ffe0300)}
77f5bfb4 c22400           ret     0x24
77f5bfb7 90               nop
```

观察本段下面反汇编SystemCallStub的结果，它只包含3条指令，分别用于将栈指针（ESP寄存器）放入EDX寄存器中、执行sysenter指令和返回。第一条指令有两个用途：一是向内核空间传递参数；二是指定从内核模式返回时的栈地址。因为笔者使用的是英特尔奔腾M处理器，所以此处是sysenter指令，对于AMD处理器，此处应该是syscall指令。

```
kd> u...
SharedUserData!SystemCallStub:
7ffe0300 8bd4             mov     edx,esp
7ffe0302 0f34             sysenter
7ffe0304 c3               ret
```

下面让我们看一下KiFastCallEntry例程，其清单如下所示。

```
kd> u nt!KiFastCallEntry L20
nt!KiFastCallEntry:
804db1bb 368b0d40f0dfff   mov      ecx,ss:[ffdff040]
804db1c2 368b6104         mov      esp,ss:[ecx+0x4]
804db1c6 b90403fe7f       mov      ecx,0x7ffe0304
804db1cb 3b2504f0dfff     cmp      esp,[ffdff004]
804db1d1 0f84cc030000     je       nt!KiServiceExit2+0x13f (804db5a3)
804db1d7 6a23             push     0x23
804db1d9 52               push     edx
804db1da 83c208           add      edx,0x8
804db1dd 6802020000       push     0x202
804db1e2 6a02             push     0x2
804db1e4 9d               popfd
804db1e5 6a1b             push     0x1b
804db1e7 51               push     ecx // Fall Through，自然进入KiSystemService函数
nt!KiSystemService:
804db1e8 90               nop
804db1e9 90               nop
804db1ea 90               nop
804db1eb 90               nop
804db1ec 90                nop
nt!KiSystemService:
804db1ed 6a00             push      0x0
804db1ef 55               push      ebp
```

显而易见，`KiFastCallEntry`在做了些简单操作后，便下落（fall through）到`KiSystemService`函数了，也就是说，快速系统调用和使用INT 2E进行的系统调用在内核中的处理绝大部分是一样的。另外，请注意ecx寄存器，`mov ecx,0x7ffe0304`将其值设为`0x7ffe0304`，也就是`SharedUserData`内存区里`SystemCallStub`例程中`ret`指令的地址（参见上文的`SystemCallStub`代码）。在进入`nt!KiSystemService`之前，ecx连同其他一些参数被压入栈中。事实上，ecx用来指定SYSEXIT返回用户模式时的目标地址。当使用INT 2E进行系统调用时，由于INT n指令会自动将中断发生时的CS和EIP寄存器压入栈中，当中断处理例程通过执行iretd返回时，iretd指令会使用栈中保存的CS和EIP值返回合适的位置。因为sysenter指令不会向栈中压入要返回的位置，所以sysexit指令必须通过其他机制知道要返回的位置。这便是压入ECX寄存器的原因。通过反汇编`KiSystemCallExit2`例程，我们可以看到在执行sysexit指令之前，ecx寄存器的值又从栈中恢复出来了。

```
kd> u nt!KiSystemCallExit l20
nt!KiSystemCallExit:
804db3b4 cf            iretd
nt!KiSystemCallExit2:
804db3b5 5a            pop      edx
804db3b6 83c408        add      esp,0x8
804db3b9 59            pop      ecx
804db3ba fb            sti
804db3bb 0f35          sysexit
nt!KiSystemCallExit3:
804db3bd 59            pop      ecx
804db3be 83c408        add      esp,0x8
804db3c1 5c            pop      esp
804db3c2 0f07          sysret
```

以上代码中包含了3个从系统调用返回的例程，即`KiSystemCallExit`、`KiSystemCallExit2`和`KiSystemCallExit3`，它们分别对应于使用INT 2E、sysenter和syscall发起的系统调用，如表2-2所示。

表2-2　系统调用

| 发起系统调用的方法 | 入口内核例程 | 返回时的指令 | 返回时使用的内核例程 |
| --- | --- | --- | --- |
| INT 2E | KiSystemService | iret | KiSystemCallExit |
| sysenter | KiFastCallEntry | sysexit | KiSystemCallExit2 |
| syscall | KiFastCallEntry | sysret | KiSystemCallExit3 |


下面展示了使用sysenter/sysexit指令对进行系统调用的完整过程（以调用ReadFile服务为例）。

```
Win32应用程序：包含ReadFile()调用的函数
↓
Kernel32.dll：Kernel32!ReadFile()
↓
NtDll.dll：NtDll!NtReadFile()
↓
固定内存区：SharedUserData!SystemCallStub
↓↑
用户模式：↓ sysenter
内核模式：↑ sysexit
↓↑
NtOSKrnl.exe：↓ KiFastCallEntry
NtOSKrnl.exe：↑ KiSystemCallExit2
↓
NtOSKrnl.exe：KiSystemService
↓
NtOSKrnl.exe：Nt!NtReadFile()
```

格物

下面通过一个小的实验来加深大家对系统调用的理解。首先启动WinDBG程序，选择`File → Open Crash Dump`，然后选择本书实验文件中的dumps\w732cf4.dmp文件。在调试会话建立后，先执行.`symfix c:\symbols`和`.reload`加载模块与符号，再执行k命令，便得到清单2-4所示的完美栈回溯。

第22章将详细讲解栈回溯的原理，现在大家只要知道栈上记录着函数相互调用时的参数和返回地址等信息。栈回溯是从栈上找到这些信息，然后显示出来的过程，是追溯线程执行轨迹的一种便捷方法。

清单2-4还显示了任务管理器程序（taskmgr）调用NtTerminateProcess系统服务时的执行过程。栈回溯的结果包含4列，第一列是序号，第二列是每个函数的栈帧基地址，第三列是返回地址，第四列是使用“函数名+字节偏移量”形式表达的执行位置。以00栈帧为例，它对应的函数是著名的蓝屏函数KeBugCheckEx，它的栈帧基地址是9796fb9c，它的返回地址是82b1ab51，翻译成符号便是PspCatchCriticalBreak+0x71。

清单2-4　完美栈回溯

```
# ChildEBP RetAddr  
00 9796fb9c 82b1ab51 nt!KeBugCheckEx+0x1e
01 9796fbc0 82a6daa8 nt!PspCatchCriticalBreak+0x71
02 9796fbf0 82a605b6 nt!PspTerminateAllThreads+0x2d
03 9796fc24 8287c87a nt!NtTerminateProcess+0x1a2
04 9796fc24 77da7094 nt!KiFastCallEntry+0x12a
05 001df4dc 77da68d4 ntdll!KiFastSystemCallRet
06 001df4e0 76193c82 ntdll!NtTerminateProcess+0xc
07 001df4f0 00bf57b9 KERNELBASE!TerminateProcess+0x2c
08 001df524 00bf67ec taskmgr!CProcPage::KillProcess+0x116
09 001df564 00bebc96 taskmgr!CProcPage::HandleWMCOMMAND+0x10f
0a 001df5d8 76abc4e7 taskmgr!ProcPageProc+0x275
0b 001df604 76ad5b7c USER32!InternalCallWinProc+0x23
0c 001df680 76ad59f3 USER32!UserCallDlgProcCheckWow+0x132
0d 001df6c8 76ad5be3 USER32!DefDlgProcWorker+0xa8
0e 001df6e4 76abc4e7 USER32!DefDlgProcW+0x22
0f 001df710 76abc5e7 USER32!InternalCallWinProc+0x23
10 001df788 76ab5294 USER32!UserCallWinProcCheckWow+0x14b
11 001df7c8 76ab5582 USER32!SendMessageWorker+0x4d0
12 001df7e8 74e94601 USER32!SendMessageW+0x7c
13 001df808 74e94663 COMCTL32!Button_NotifyParent+0x3d
14 001df824 74e944ed COMCTL32!Button_ReleaseCapture+0x113
15 001df884 76abc4e7 COMCTL32!Button_WndProc+0xa18
16 001df8b0 76abc5e7 USER32!InternalCallWinProc+0x23
17 001df928 76abcc19 USER32!UserCallWinProcCheckWow+0x14b
18 001df988 76abcc70 USER32!DispatchMessageWorker+0x35e
19 001df998 76ab41eb USER32!DispatchMessageW+0xf
1a 001df9bc 00be16fc USER32!IsDialogMessageW+0x588
1b 001dfdac 00be5384 taskmgr!wWinMain+0x5d1
1c 001dfe40 76bbed6c taskmgr!_initterm_e+0x1b1
1d 001dfe4c 77dc377b kernel32!BaseThreadInitThunk+0xe
1e 001dfe8c 77dc374e ntdll!__RtlUserThreadStart+0x70
1f 001dfea4 00000000 ntdll!_RtlUserThreadStart+0x1b
```

仔细观察清单2-4中的地址部分，很容易看出用户空间和内核空间的分界，也就是在栈帧04和栈帧05之间。栈帧05中的KiFastSystemCallRet函数属于ntdll模块，位于用户空间。栈帧04中的KiFastCallEntry函数属于nt模块，位于内核空间。栈帧04的基地址是9796fc24，属于内核空间；栈帧05的基地址是001df4dc，属于用户空间。它们分别来自这个线程的内核态栈和用户态栈。WinDBG的k命令穿越两个空间，遍历两个栈，显示出线程在用户空间和内核空间执行的完整过程，能产生如此完美的栈回溯显示了WinDBG的强大。

### 2.6.4　逆向调用

前文介绍了从用户模式进入内核模式的两种方法，通过这两种方法，用户模式的代码可以“调用”位于内核模式的系统服务。那么内核模式的代码是否可以主动调用用户模式的代码呢？答案是肯定的，这种调用通常称为逆向调用（reverse call）。

简单来说，逆向调用的过程是这样的。首先内核代码使用内核函数`KiCallUserMode`发起调用。接下来的执行过程与从系统调用返回（KiServiceExit）类似，不过进入用户模式时执行的是NTDll.DLL中的`KiUserCallbackDispatcher`。而后KiUserCallbackDispatcher会调用内核希望调用的用户态函数。当用户模式的工作完成后，执行返回动作的函数会执行INT 2B指令，也就是触发一个0x2B异常。这个异常的处理函数是内核模式的`KiCallbackReturn`函数。于是，通过INT 2B异常，CPU又跳回内核模式继续执行了。

```
lkd> !idt 2b
Dumping IDT:
2b:   8053d070 nt!KiCallbackReturn
```

以上是使用WinDBG的`!idt`命令观察到的0x2B异常的处理函数。

### 2.6.5　实例分析

下面通过一个实际例子来进一步展示系统调用和逆向调用的执行过程。清单2-5显示了使用WinDBG的内核调试会话捕捉到的记事本进程发起系统调用进入内核和内核函数执行逆向调用的全过程（栈回溯）。

清单2-5　记事本进程从发起系统调用进入内核和内核函数逆向调用的全过程

```
kd> kn
 # ChildEBP RetAddr  
00 0006fe94 77fb4da6 USER32!XyCallbackReturn
01 0006fe94 8050f8ae ntdll!KiUserCallbackDispatcher+0x13
02 f4fc19b4 80595d2c nt!KiCallUserMode+0x4
03 f4fc1a10 bf871e98 nt!KeUserModeCallback+0x87
04 f4fc1a90 bf8748d4 win32k!SfnDWORD+0xa0
05 f4fc1ad8 bf87148d win32k!xxxSendMessageToClient+0x174
06 f4fc1b24 bf8714d3 win32k!xxxSendMessageTimeout+0x1a6
07 f4fc1b44 bf8635f6 win32k!xxxSendMessage+0x1a
08 f4fc1b74 bf84a620 win32k!xxxMouseActivate+0x22d
09 f4fc1c98 bf87a0c1 win32k!xxxScanSysQueue+0x828
0a f4fc1cec bf87a8ad win32k!xxxRealInternalGetMessage+0x32c
0b f4fc1d4c 804da140 win32k!NtUserGetMessage+0x27
0c f4fc1d4c 7ffe0304 nt!KiSystemService+0xc4
0d 0006feb8 77d43a21 SharedUserData!SystemCallStub+0x2
0e 0006febc 77d43c95 USER32!NtUserGetMessage+0xc
0f 0006fed8 010028e4 USER32!GetMessageW+0x31
10 0006ff1c 01006c54 notepad!WinMain+0xe3
11 0006ffc0 77e814c7 notepad!WinMainCRTStartup+0x174
12 0006fff0 00000000 kernel32!BaseProcessStart+0x23
```

根据执行的先后顺序，最下面一行（帧#12）对应的是进程的启动函数`BaseProcessStart`，而后是编译器生成的进程启动函数`WinMainCRTStartup`，以及记事本程序自己的入口函数`WinMain`。帧#0f表示记事本程序在调用GetMessage API进入消息循环。接下来GetMessage API调用Windows子系统服务的残根函数`NtUserGetMessage`。从第2列的栈帧基地址都小于0x800000000可以看出，帧#12～#0d都是在用户模式执行的。帧#0d执行我们前面分析过的`SystemCallStub`，而后（帧#0c）便进入了内核模式的`KiSystemService`。`KiSystemService`根据系统服务号码，将调用分发给Windows子系统内核模块win32k中的`NtUserGetMessage`函数。

帧#0a～#05表示内核模式的窗口消息函数在工作。帧#07～#05表示要把一个窗口消息发送到用户态。帧#04的SfnDWORD表示在将消息组织好后调用`KeUserModeCallback`函数，发起逆向调用。帧#02表明在执行`KiCallUserMode`函数，帧#01表明已经在用户模式下执行，这两行之间的部分过程没有显示出来。同样，帧#01 和帧#00 之间执行用户模式函数的过程没有完全体现出来。`XyCallbackReturn`函数是用于返回内核模式的，它的代码很简单，只有如下几条指令。

```
USER32!XyCallbackReturn:
001b:77d44168 8b442404     mov   eax,dword ptr [esp+4] ss:0023:0006fe84=00000000
001b:77d4416c cd2b          int   2Bh
001b:77d4416e c20400        ret   4
```

第1行把用户模式函数的执行结果赋给EAX寄存器，第2行执行INT 2B指令。执行过INT 2B后，CPU便转去执行异常处理程序KiCallbackReturn，回到了内核模式。

## 2.7　线程

通常，一个进程内有一个或者多个线程。但是在某些特殊情况下，比如进程创建初期或者进程退出和销毁的过程中，进程内也可能没有任何线程。

### 2.7.1　ETHREAD

与使用EPROCESS结构来描述进程类似，NT内核使用ETHREAD结构来描述线程。在内核代码中，大多时候使用ETHREAD结构的地址来索引线程。在调试时，也常常这样使用。比如在内核调试会话中，执行`.thread`命令，便会显示出当前线程的ETHREAD结构地址。

```
kd> .thread
Implicit thread is now 873e9500
```

然后执行如下命令便可以观察ETHREAD结构的内容。

```
kd> dt _ETHREAD 873e9500
ntdll!_ETHREAD
   +0x000 Tcb                     : _KTHREAD
   +0x200 CreateTime              : _LARGE_INTEGER 0x01d4dbe4`94abfe4e
   +0x208 ExitTime                : _LARGE_INTEGER 0x873e9708`873e9708
   +0x208 KeyedWaitChain          : _LIST_ENTRY [ 0x873e9708 - 0x873e9708 ]
   +0x210 ExitStatus              : 0n0
   +0x214 PostBlockList           : _LIST_ENTRY [ 0x0 - 0x77da7078 ]
   +0x214 ForwardLinkShadow       : (null) 
   +0x218 StartAddress            : 0x77da7078 Void
【省略多行】
   +0x290 AlpcMessageId           : 0
   +0x294 AlpcMessage             : (null) 
   +0x294 AlpcReceiveAttributeSet : 0
   +0x298 AlpcWaitListEntry     : _LIST_ENTRY [ 0x0 - 0x8b9d7764 ]
   +0x2a0 CacheManagerCount     : 0
   +0x2a4 IoBoostCount          : 0
   +0x2a8 IrpListLock           : 0
   +0x2ac ReservedForSynchTracking : (null) 
   +0x2b0 CmCallbackListHead     : _SINGLE_LIST_ENTRY
```

ETHREAD结构也很庞大，包含着线程的各种属性。特别值得说明的是，ETHREAD开头的512字节是一个KTHREAD结构，也称为线程控制块（TCB），里面的字段主要是供内核调度线程时使用的。

只要把上面命令中的第一个E字符改为K便可以观察KTHREAD结构了。

```
kd> dt _KTHREAD 873e9500
ntdll!_KTHREAD
   +0x000 Header               : _DISPATCHER_HEADER
   +0x010 CycleTime            : 0x5a2bcb8f
   +0x018 HighCycleTime        : 0
   +0x020 QuantumTarget        : 0x60941ef2
   +0x028 InitialStack         : 0x9796fed0 Void
   +0x02c StackLimit           : 0x9796d000 Void
   +0x030 KernelStack          : 0x9796fac8 Void
   +0x034 ThreadLock           : 0
   +0x038 WaitRegister         : _KWAIT_STATUS_REGISTER
   +0x039 Running              : 0x1 ''
   +0x03a Alerted              : [2]  ""
   +0x03c KernelStackResident     : 0y1
   +0x03c ReadyTransition      : 0y0
【省略多行】
   +0x1f0 SListFaultAddress    : (null) 
   +0x1f4 ThreadCounters       : (null) 
   +0x1f8 XStateSave           : (null)
```

其中的Header字段是`DISPATCHER_HEADER`类型，DISPATCHER是NT内核的线程调度器的别名，代表分发CPU时间片的意思。

因为ETHREAD结构字段众多，而且缺少详细的文档描述，所以在调试时，一般不直接观察结构，而使用WinDBG的扩展命令`!thread`，让这个扩展命令以比较友好的方式显示线程属性。

表2-3　线程的状态

| 状　　态 | 值 | 含　　义 |
| --- | --- | --- |
| Initialized | 0 | 正在创建和初始化 |
| Ready | 1 | 就绪，可以被分发器调度运行 |
| Running | 2 | 正在某个CPU上运行 |
| Standby | 3 | 待命，每个CPU只有一个线程处于此状态，代表下一个要执行的线程 |
| Terminated | 4 | 结束执行 |
| Waiting | 5 | 等待，通常意味着线程调用了睡眠（Sleep）函数、取消息函数或者等待同步对象的各种函数，主动放弃执行机会 |
| Transition | 6 | 过渡状态，一般是因为线程已经可以运行，但是它的内核态栈被交换出了内存，一旦栈被交换回内存，便进入就绪状态 |
| DeferredReady | 7 | 延迟就绪，为了缩短扫描调度数据库时的加锁时间，内核把就绪的线程先设置为此状态 |
| GateWait | 8 | 门等待，在等待门分发器对象时，进入此状态 |


在内核调试会话中，可以直接观察State字段的值。

```
kd> dt _KTHREAD 873e9500 -y state
ntdll!_KTHREAD
   +0x068 State : 0x2 ''
```

执行`!ready`命令可以显示所有处于就绪状态的线程。

```
kd> !ready
Processor 0: Ready Threads at priority 14
    THREAD 8ba7d238  Cid 01b4.01f0  Teb: 7ffd9000 Win32Thread: ffb13008 READY on  
 processor 0
    THREAD 87007a90  Cid 01b4.01e4  Teb: 7ffde000 Win32Thread: ffa99dd8 READY on  
 processor 0
Processor 0: Ready Threads at priority 13
    THREAD 872bcd48  Cid 0314.0624  Teb: 7ffde000 Win32Thread: fe4eb438 READY on  
 processor 0
Processor 0: Ready Threads at priority 11
    THREAD 87288758  Cid 0314.0460  Teb: 7ffd9000 Win32Thread: ffa97408 READY on  
 processor 0
Processor 0: Ready Threads at priority 10
    THREAD 87271490  Cid 0438.06b8  Teb: 7ffd8000 Win32Thread: ffb746b8 READY on  
 processor 0
Processor 0: Ready Threads at priority 8
    THREAD 87328088  Cid 039c.085c  Teb: 7ffd7000 Win32Thread: 00000000 READY on  
 processor 0
```

观察上面各个线程的State字段，值为1，代表就绪。

```
kd> dt _KTHREAD 8ba7d238 -y state
ntdll!_KTHREAD
   +0x068 State : 0x1 ''
```

NT内核为每个CPU定义了一个名为处理器控制块（Processor Control Block，PRCB）的庞大结构，在这个结构中有一个名为DispatcherReadyListHead的数组，包含32个元素，代表线程的32个优先级，每个元素是个LIST_ENTRY结构，起链表头的作用，用来挂接对应优先级的就绪线程。

```
kd> dt _KPRCB 82969d20 -y Dispatcher
ntdll!_KPRCB
   +0x3220 DispatcherReadyListHead : [32] _LIST_ENTRY [ 0x8296cf40 - 0x8296cf40 ]
```

上面的`!ready`命令便是从这个链表数组中读取信息，然后显示出各个优先级的就绪线程的。

原书P41的图2-7显示了各个线程状态之间的切换关系，以及部分切换条件。

如果线程处于等待状态，那么`!thread`命令会显示出等待原因，这对于调试线程死锁问题是非常有价值的。KTHREAD结构中有一个名为WaitReason的字段，用来记录线程的等待原因，它的长度只有1字节，是枚举类型，名为`KWAIT_REASON`，在公开的PDB符号文件中，包含了这个枚举类型的定义，因此，很容易在WinDBG中观察它，比如：

```
kd> dt _KTHREAD 8b658c88 -y WaitReason
ntdll!_KTHREAD
   +0x187 WaitReason : 0x6 ''
```

WaitReason字段的值为6，代表用户代码主动请求等待（UserRequest）。

因为一些公开的内核函数的参数中也使用了`KWAIT_REASON`，比如KeWaitForSingleObject等，所以在驱动开发包（DDK/WDK）的头文件（wdm.h）中也包含这个枚举的定义，但是没有描述每个值的含义。表2-4列出了`KWAIT_REASON`的所有可能值，并且对每种原因做了说明。

表2-4　KWAIT_REASON的所有可能值

| 原　　因 | 值 | 含　　义 |
| --- | --- | --- |
| Executive | 0 | 公开的文档建议驱动程序调用等待函数时应该指定此原因 |
| FreePage | 1 | 等待空闲页 |
| PageIn | 2 | 等待把交换出去的内存页换回内存 |
| PoolAllocation | 3 | — |
| DelayExecution | 4 | 延迟执行，一般是因为调用了Sleep或者NtDelayExecution等函数 |
| Suspended | 5 | 线程被挂起，调用KiSuspendThread时，这个函数内部会使用Suspended常量来调用等待函数，放弃执行权 |
| UserRequest | 6 | 公开的文档建议如果驱动程序代表应用代码调用等待函数，那么应该指定此原因 |
| WrExecutive | 7 | 某些LPC函数会使用这个常量调用等待函数 |
| WrFreePage | 8 | 等待空闲页，例如CcQueueLazyWriteScanThread和MiModifiedPageWriter会使用此常量 |
| WrPageIn | 9 | 等待把交换出去的内存页换回内存 |
| WrPoolAllocation | 10 | 从名字看与内核池有关，未找到使用实例 |
| WrDelayExecution | 11 | 推迟执行，与DelayExecution含义相同 |
| WrSuspended | 12 | 线程挂起，与Suspended含义相同 |
| WrUserRequest | 13 | 与UserRequest含义相同 |
| WrEventPair | 14 | 当服务端和客户端使用一对时间对象时，使用此常量调用等待函数 |
| WrQueue | 15 | 等待队列对象，比如调用KeRemoveQueueEx时可能进入 |
| WrLpcReceive | 16 | 使用LPC通信时，因为要接收数据而等待对方发送 |
| WrLpcReply | 17 | 使用LPC通信时，因为要发送数据而等待对方接收 |
| WrVirtualMemory | 18 | 内存管理器使用这个常量调用等待函数 |
| WrPageOut | 19 | 当内存管理器冲洗缓冲区并把内存中的数据写入磁盘时，会使用此常量来调用等待函数 |
| WrRendezvous | 20 | 较少使用 |
| WrKeyedEvent | 21 | — |
| WrTerminated | 22 | — |
| WrProcessInSwap | 23 | — |
| WrCpuRateControl | 24 | — |
| WrCalloutStack | 25 | — |
| WrKernel | 26 | — |
| WrResource | 27 | — |
| WrPushLock | 28 | — |
| WrMutex | 19 | 等待互斥量 |
| WrQuantumEnd | 30 | 时间片用完 |
| WrDispatchInt | 31 | — |
| WrPreempted | 32 | 被剥夺执行权 |
| WrYieldExecution | 33 | 主动放弃执行权 |
| WrFastMutex | 34 | 等待Windows 2000引入的高速互斥量 |
| WrGuardedMutex | 35 | 等待Windows Server 2003引入的被保护互斥量（Guarded Mutex） |
| WrRundown | 36 | — |
| WrAlertByThreadId | 37 | — |
| WrDeferredPreempt | 38 | — |
| WrPhysicalFault | 39 | — |
| MaximumWaitReason | 40 | 最大的有效值，Windows 10中的值，Windows 7中该值为37 |


值得说明一下，表2-4列出的枚举常量主要是为软件调试服务的。当线程因为进入等待状态而不执行时，可以通过这些常量查找进入等待状态的原因，搜索发起等待的代码，对于NT内核中的等待函数来说，并不关心KWAIT_REASON参数的内容。

### 2.7.2　TEB

与描述进程用户空间信息的PEB类似，NT内核定义了线程环境块（Thread Environment Block，TEB）来描述线程的用户空间信息，包括用户态栈、异常处理、错误码、线程局部存储等。

在通过WinDBG调试应用程序时，可以使用`!teb`显示当前线程的TEB结构位置，比如：

```
0:000> !teb
TEB at 000000f0023f4000
    ExceptionList:        0000000000000000
    StackBase:            000000f002130000
    StackLimit:           000000f00211f000
    SubSystemTib:         0000000000000000
    FiberData:            0000000000001e00
    ArbitraryUserPointer: 0000000000000000
    Self:                 000000f0023f4000
    EnvironmentPointer:   0000000000000000
    ClientId:             000000000000415c . 0000000000004e1c
    RpcHandle:            0000000000000000
    Tls Storage:          0000026cd1deb020
    PEB Address:          000000f0023f3000
    LastErrorValue:       0
    LastStatusValue:      c0000034
    Count Owned Locks:    0
    HardErrorMode:        0
```

本书卷1在介绍段机制时，曾经提到过TEB，介绍了NT内核会使用CPU的硬件机制来快速定位当前线程的TEB。也因为此，内核在创建线程时，就会分配专门的内存页用作TEB，将其地址记录在KTHREAD中，所以TEB的地址总是按页对齐的（低12位为0）。

也可以使用dt命令直接观察TEB结构。

```
0:000> dt _TEB 000000f0023f4000
ntdll!_TEB
   +0x000 NtTib                : _NT_TIB
   +0x038 EnvironmentPointer   : (null) 
   +0x040 ClientId             : _CLIENT_ID
   +0x050 ActiveRpcHandle      : (null) 
   +0x058 ThreadLocalStoragePointer : 0x0000026c`d1deb020 Void
   +0x060 ProcessEnvironmentBlock : 0x000000f0`023f3000 _PEB
   +0x068 LastErrorValue       : 0
【省略很多行】
   +0x1230 glSection            : (null) 
   +0x1238 glTable              : (null) 
   +0x1240 glCurrentRC          : (null) 
   +0x1248 glContext            : (null) 
【省略很多行】
   +0x1778 ThreadPoolData       : (null) 
   +0x1780 TlsExpansionSlots    : (null) 
   +0x17ee SafeThunkCall        : 0y0
   +0x17ee InDebugPrint         : 0y0
   +0x17ee HasFiberData         : 0y0
   +0x17ee SkipThreadAttach     : 0y0
   +0x1818 ReservedForWdf       : (null) 
   +0x1820 ReservedForCrt       : 0
   +0x1828 EffectiveContainerId    : _GUID {00000000-0000-0000-0000-000000000000}
```

可以看到，TEB包含着各种各样的信息，是一个庞大的结构，而且还在随着NT内核的发展而增长。最后一个字段EffectiveContainerId（有效容器ID）显然是在容器技术流行后新增的，可谓与时俱进。

## 2.8　WoW进程

今天的大多数Windows系统是64位的，运行在支持64位的CPU上，比如笔者现在写作使用的便是64位的Windows 10。在64位的Windows系统中，内核空间的代码都是64位的，而用户空间的代码却不一定如此。为了兼容老的32位应用程序，64位的Windows系统上可以运行32位的应用程序，这样运行在64位内核上的32位进程有一个专门的名字，叫作WoW64（Windows 32 on Windows 64）进程，常常简称为WoW进程。

### 2.8.1　架构

```
32位的可执行文件
↓
32位DLL
↓
转接层（64位）
↓
64位的内核
```

上面展示了WoW进程的架构和工作原理。上面是32位的可执行文件和32位的动态链接库（DLL），下面是64位的内核。32位的代码是不能直接与64位的内核交互的，中间的转接层就是为了解决这个问题而设计的。转接层本身是64位的模块，它给32位的应用程序营造一个32位的环境。这个环境有点像虚拟机，但没有虚拟机技术那么复杂，它的工作简单很多，主要负责指针长度的转换和解决API兼容等问题。

因为32位的程序需要使用老的32位Win 32API和一些库函数，所以在64位Windows系统的目录里，总是有一个名为SysWoW64的子目录，里面放着32位版本的程序文件和动态库。与SysWoW64并列的还有一个System32目录，里面放着内核和64位的各种程序文件。简而言之，SysWoW64中放的是32位的内容，System32中放的是64位的内容，目录名字是有些误导的，大家不要上当。为什么这样呢？一种说法是为了兼容应用程序，保持系统程序主目录的System32之名不变，而SysWOW64中的64来自Windows 32 on Windows 64。

### 2.8.2　工作过程

既可以使用32位版本的WinDBG调试WoW进程，也可以使用64位版本的WinDBG来调试。前者的好处是比较简单，仿佛调试普通32位程序一样；后者的好处是既可以调试32位代码，也可以调试64位的转接层。可以使用`.effmach`命令在两种代码间切换。

格物

在64位的Windows系统中，使用资源管理器浏览Windows\SysWoW64文件夹，找到notepad.exe（32位版本），通过双击执行它。

启动64位的WinDBG，选择`File → Attach to a process`，附加到刚刚启动的notepad进程中。执行如下命令先切换到0号线程，再切换到64位模式，然后观察栈回溯。

```
~0s
.effmach amd64
0:000> k
 # Child-SP          RetAddr           Call Site
00 00000000`002ae4b8 00000000`776152c0 wow64win!NtUserGetMessage+0x14
01 00000000`002ae4c0 00000000`77697913 wow64win!whNtUserGetMessage+0x30
02 00000000`002ae520 00000000`776f1913 wow64!Wow64SystemServiceEx+0x153
03 00000000`002aede0 00000000`776f1389 wow64cpu!ServiceNoTurbo+0xb
04 00000000`002aee90 00000000`7769cec6 wow64cpu!BTCpuSimulate+0x9
05 00000000`002aeed0 00000000`7769cdb0 wow64!RunCpuSimulation+0xa
06 00000000`002aef00 00007ffd`6e2af637 wow64!Wow64LdrpInitialize+0x120
07 00000000`002af1b0 00007ffd`6e29fa45 ntdll!LdrpInitializeProcess+0x1887
08 00000000`002af5d0 00007ffd`6e254feb ntdll!_LdrpInitialize+0x4aa45
09 00000000`002af670 00007ffd`6e254f9e ntdll!LdrpInitialize+0x3b
0a 00000000`002af6a0 00000000`00000000 ntdll!LdrInitializeThunk+0xe
```

阅读上面的栈回溯，可以看到64位转接层的执行过程，其中的wow64和wow64win都是转接层的核心模块。

执行如下命令切换到32位模式并观察32位代码的执行情况：

```
0:000> .effmach x86
Effective machine: x86 compatible (x86)
0:000:x86> k
 # ChildEBP RetAddr  
00 002efa4c 770ba850 win32u!NtUserGetMessage+0xc
01 002efa88 012e72d8 USER32!GetMessageW+0x30
02 002efb0c 012fb400 notepad!WinMain+0x18e
03 002efba0 75f08494 notepad!__mainCRTStartup+0x146
04 002efbb4 777641c8 KERNEL32!BaseThreadInitThunk+0x24
05 002efbfc 77764198 ntdll_77700000!__RtlUserThreadStart+0x2f
06 002efc0c 00000000 ntdll_77700000!_RtlUserThreadStart+0x1b
```

值得说明的是，在WoW进程中，总是有两个ntdll模块，一个是64位的，另一个是32位的。因为二者的名字相同，为了区别它们，WinDBG会给后加载进进程的32位版本的模块名加上基地址，即像ntdll_77700000这样。

仔细观察上面k命令的结果，容易看出两个结果中的栈地址相差悬殊，其实它们来自两个栈。进一步说，WoW进程中，很多东西是双份的，每个进程有两个PEB，每个线程有两个TEB，有两个栈。WinDBG的wow64exts扩展模块专门是为调试WoW进程而设计的，它的info命令可以显示WoW进程的双份资产。

```
0:000:x86> !wow64exts.info

Guest (WoW) PEB: 0x5cb000
Native      PEB: 0x5ca000

Wow64 information for current thread:
Guest (WoW) TEB: 0x5ce000
Native      TEB: 0x5cc000

Guest (WoW), StackBase   : 0x2f0000
        StackLimit  : 0x2df000
        Deallocation: 0x2b0000
Native, StackBase   : 0x2afd20
        StackLimit  : 0x2a8000
        Deallocation: 0x270000
```

上面的结果中，使用了虚拟机的术语，Guest指的是32位代码，Native指的是64位代码。

下面再做一个小实验来演示WoW线程中系统调用的执行过程。执行如下命令为32位版本的NTDLL.DLL中的NtReadFile函数设置一个断点。

```
bp ntdll_77700000!NtReadFile
```

执行g命令恢复目标执行，切换到记事本程序的窗口，选择`File → Open`，触发文件操作，断点会命中，切换回WinDBG，执行u命令反汇编断点附近的代码。

```
0:000:x86> u
ntdll_77700000!NtReadFile:
7776a910 b806001a00    mov     eax,1A0006h
7776a915 ba60f17777 mov edx,offset ntdll_77700000!Wow64SystemServiceCall (7777f160)
7776a91a ffd2          call    edx
7776a91c c22400        ret     24h
```

按F11键单步跟踪执行，跟踪进入下面call指令调用的函数：

```
7776a91a ffd2      call   edx {ntdll_77700000!Wow64SystemServiceCall (7777f160)}
```

进入这个函数后，可以看到，它只有一条无条件跳转指令，即：

```
ntdll_77700000!Wow64SystemServiceCall:
7777f160 ff2528b28177    jmp     dword ptr [ntdll_77700000!Wow64Transition  
(7781b228)] ds:002b:7781b228={wow64cpu!KiFastSystemCall (776f7000)}
```

继续单步跟踪，进入wow64cpu模块：

```
wow64cpu!KiFastSystemCall:
776f7000 ea09706f773300  jmp     0033:776F7009
```

这又是一条无条件跳转。注意，这条指令的跳转目标中特别指定了段选择子33，它指向的是64位的段。这正是CPU手册中所描述的从32位兼容模式过渡到64位模式的方法。再单步跟踪一次，便进入64位代码。

```
0:014:x86> t
wow64cpu!KiFastSystemCall+0x9:
00000000`776f7009 41ffa7f8000000  jmp     qword ptr [r15+0F8h] ds:00000000`  
776f4718={wow64cpu!CpupReturnFromSimulatedCode (00000000`776f18d2)}
```

通过上面的实验可以看出，WoW进程中的32位版本NTDLL.DLL在执行系统调用时，会调用特殊的Wow64SystemServiceCall函数，切换到64位的WoW转接层。

### 2.8.3　注册表重定向

在调试与WoW进程有关的问题时，如果需要查看或者修改注册表，那么需要特别注意。出于多种原因，64位Windows系统会对WoW进程的注册表访问实施重定向。比如，如果程序中访问的路径为`HKEY_LOCAL_MACHINE\Software`，那么会被重定向到`HKEY_LOCAL_MACHINE\Software\Wow6432Node`。

使用注册表编辑器时，如果要查看WoW进程的设置，那么也应该查看Wow6432Node表键下的。

有了这个重定向机制，可以认为，很多注册表表键有两份，一份供WoW进程使用，另一份供64位程序使用。不过，一部分表键是两类程序共享的，`HKEY_LOCAL_MACHINE\SOFTWARE\Policies`表键便是如此。而且有些表键的情况是因Windows版本不同而不同的，比如`HKEY_LOCAL_MACHINE\SOFTWARE\Classes`表键，在Windows 7或者更高版本中是共享的，在老的版本中是重定向的。概而言之，注册表早因为臃肿杂乱而成为Windows系统的一个负担，有了WoW后，问题变得更加复杂，遇到问题时，建议仔细查阅官方文档，因地制宜。

### 2.8.4　注册表反射

考虑到有些COM组件既有32位版本也有64位版本，为了让用户在一个版本中所做的设置在另一个版本中也有效，Windows实现了一种名为注册表反射（registry reflection）的机制，对于某些与COM组件有关的表键，来自一边的修改会自动更新到另一边。这样的表键主要有以下几个：

（1）HKLM\Software\Classes；

（2）HKLM\Software\Ole；

（3）HKLM\Software\Rpc；

（4）HKLM\Software\Com3；

（5）HKLM\Software\EventSystem；

（6）HKLM\Software\CLSID（只用于进程外组件）。

### 2.8.5　文件系统重定向

如上文所讲，在WoW进程中，有两个NTDLL.dll，一个是64位的，另一个是32位的。64位版本的NTDLL.DLL位于`%windir%\System32`目录中，32位版本的NTDLL.DLL位于`%windir%\SysWOW64`目录中。

在Windows on ARM系统中，还有一个`%windir%\SysArm32`目录，里面放的是32位的ARM版本系统文件。

为了让不同类型的程序可以取到自己需要的系统文件，64位的Windows系统设计了名为文件系统重定向的机制，当32位的WoW进程访问系统文件目录时，会被自动重定向到SysWOW64或SysArm32目录。

## 2.9　创建进程

无论我们使用哪种方式打开一个新的程序，大多数时候，Windows操作系统使用一套标准的流程来创建一个新进程。创建新进程的过程比较复杂，一般分为如下6个阶段。

- 阶段1：在父进程的用户空间中打开要执行的映像文件，确定其名称、类型和系统对它的设置选项。

- 阶段2：进入父进程的内核空间，为新进程创建EPROCESS结构、进程地址空间、KPROCESS结构和PEB。 

- 阶段3：创建初始线程，但是创建时指定了挂起（suspend）标志，它并不会立刻开始运行。 

- 阶段4：通知子系统服务程序。对于Windows程序，通知Windows子系统服务进程，即CSRSS。

- 阶段5：初始线程开始在内核空间执行。 

- 阶段6：通过APC机制（第5章），在新进程自己的用户空间中执行初始化动作。这一步最重要的工作就是通过NTDLL.DLL中的加载器，加载进程所依赖的DLL文件。 

在上面6个阶段中，前4个都是在父进程或者子系统服务进程中完成的。这样做的原因是新进程创建之初，进程内的设施还不完善，执行某些任务可能有困难或者不可行。

Windows Internals一书很详细地描述了上面每个阶段所做的工作，为了避免重复，本书只介绍其梗概。

## 2.10　最小进程和Pico进程

为了满足虚拟化、容器和适用于Linux系统的Windows子系统（Windows Subsystem for Linux，WSL）等需求，除了用以运行Windows程序的普通进程（称为NT进程），今天的NT内核还支持两种特殊类型的进程——最小进程（minimal process）和Pico进程。

如前面各节所讲，对于普通的NT进程，NT内核会自动创建一些设施，并将这些设施映射到进程的用户空间中，比如描述进程属性的进程环境块（PEB），描述线程属性的线程环境块（TEB）等。另外，考虑到NTDLL.DLL的特殊性，NT内核也会自动将NTDLL.DLL映射到普通进程的用户模式空间中。但对于某些特殊情况，这些动作不但是不必要的，而且是多余和有副作用的，最小进程和Pico进程就是为了解决这个问题而设计的。

### 2.10.1　最小进程

所谓最小进程，就是在创建进程时，指定一个特殊的标志，告诉NT内核，只创建进程的空间，不要自动向进程空间中添加内容。

目前，微软没有对外公开用于创建最小进程的接口，只供内部使用。根据有限的资料，Windows 10的内存压缩技术和基于虚拟化的安全（VBS）功能都使用了最小进程。

下面以内存压缩进程为例来加深一下大家对最小进程的理解。在Windows 10系统中，以管理员身份启动一个PowerShell窗口，执行Enable-MMAgent–mc，启用内存压缩功能，重启后，在内核调试会话中执行`!process 0 0`，列出所有进程，然后找到MemCompression进程。

```
PROCESS ffffbd039cacd040
    SessionId: none  Cid: 0a00    Peb: 00000000  ParentCid: 0004
    DirBase: 20fd1e000  ObjectTable: ffffac8e2f236740  HandleCount:   0.
    Image: MemCompression
```

可以看到，这个进程的Peb值为0，这是区别于普通NT进程的一个显著标志。

继续执行如下的dt命令观察进程结构中的Flags字段：

```
1: kd> dt _EPROCESS ffffbd039cacd040 -y Flags
ntdll!_EPROCESS
   +0x300 Flags2 : 0xd000
   +0x304 Flags : 0x14440c01
   +0x6cc Flags3 : 1
```

其中，Flags3为1，意味着代表最小进程的Minimal标志位（Bit 0）为1。

如果执行`!process ffffbd039cacd040`继续观察这个进程的详细信息，可以看到它有很多个线程，但都是系统线程。简单理解，这个进程的进程空间就是一个内存仓库（Store），用于存放原本应该交换到磁盘上的内存。为了减少这个内存仓库所占用的物理内存，这个进程的系统线程会压缩仓库里的内存页。

从2017年年底发布的17063版本开始，Windows 10引入了一个新的最小进程，名叫Registry，我们将其称为注册表进程。与内存压缩进程用于缓存内存数据类似，注册表进程的主要作用是缓存注册表数据，以便提高访问注册表数据的效率，降低与注册表有关的内存开销。

在WinDBG的内核调试会话中，可以使用`!process 0 0 Registry`命令找到注册表进程：

```
0: kd> !process 0 0 Registry
PROCESS ffff8682decc5040
    SessionId: none  Cid: 0078    Peb: 00000000  ParentCid: 0004
    DirBase: 03e00002  ObjectTable: ffff9a8e06c25b40  HandleCount:   0.
    Image: Registry
```

然后，可以使用EPROCESS结构的地址作为参数观察它的详细信息。

```
0: kd> !PROCESS ffff8682decc5040
PROCESS ffff8682decc5040
    SessionId: none  Cid: 0078    Peb: 00000000  ParentCid: 0004
    DirBase: 03e00002  ObjectTable: ffff9a8e06c25b40  HandleCount:   0.
    Image: Registry
    VadRoot ffff8682df0bb6c0 Vads 182 Clone 0 Private 361. Modified 99491. Locked 0.
    DeviceMap ffff9a8e06c04bb0
    Token                             ffff9a8e06c23700
    ElapsedTime                       00:51:06.419
    UserTime                          00:00:00.000
    KernelTime                        00:00:02.140
    QuotaPoolUsage[PagedPool]         318904
    QuotaPoolUsage[NonPagedPool]      24752
    Working Set Sizes (now,min,max)  (3502, 50, 345) (14008KB, 200KB, 1380KB)
    PeakWorkingSetSize                55844
    VirtualSize                       153 Mb
    PeakVirtualSize                   220 Mb
    PageFaultCount                    126591
    MemoryPriority                    BACKGROUND
    BasePriority                      8
    CommitCharge                      365
```

一般Registry进程有3个线程，其中两个线程的入口函数都是CmpLazyWriteWorker，该函数应该是用于把修改过的注册表数据成批写回磁盘的工作线程。另一个线程名叫CmpDummyThreadRoutine。这个线程的代码看起来有些古怪，线程启动后就一直在等待一个名为CmpDummyThreadEvent的事件，如果等待成功，便调用KeBugCheckEx函数触发蓝屏崩溃。其真实用途是“占位”，始终等待一个内核空间的事件对象，让内存管理器不要把它的内存页交换出去。

关于在任务管理器中是否显示最小进程，目前的Windows 10版本似乎有些混乱。根据笔者的观察，内存压缩进程总是不显示，而注册表进程则有时显示有时不显示。后文介绍任务管理器使用的Windows 10 版本的任务管理器中包含了注册表进程（其进程ID似乎总为120）。

### 2.10.2　Pico进程

Pico进程是“最小进程”的一个子类。在英文中，Pico一般用作前缀，代表10-12，有“微小”之意。

与普通的“最小进程”相比，Pico进程的特点是它通过所谓的Pico提供器与NT内核协作。简单理解，“最小进程”是一个不希望内核干预太多的容器，而Pico进程则可以通过一组接口与NT内核交互。

NT内核为Pico提供器新增了一个名为PsRegisterPicoProvider的接口，供注册使用。例如，在启用了WSL的Windows 10内核启动早期，WSL的子系统核心驱动LXCORE就会调用PsRegisterPicoProvider函数注册Pico提供器，其过程如下：

```
# Call Site
00 nt!PsRegisterPicoProvider
01 LXCORE!LxInitialize
02 lxss!DriverEntry
03 lxss!GsDriverEntry
04 nt!IopInitializeBuiltinDriver
05 nt!PnpInitializeBootStartDriver
06 nt!PipInitializeCoreDriversByGroup
07 nt!PipInitializeCoreDriversAndElam
08 nt!IopInitializeBootDrivers
09 nt!IoInitSystemPreDrivers
0a nt!IoInitSystem
0b nt!Phase1Initialization
0c nt!PspSystemThreadStartup
0d nt!KiStartSystemThread
```

PsRegisterPicoProvider函数有两个参数，都是结构指针，结构的第一个字段表示大小，后面是函数指针。例如，下面是LXCORE注册时第一个参数的内容。

```
7: kd> dqs rcx
fffffa82`8f607440  00000000`00000058
fffffa82`8f607448  fffff806`1b14aaa0 LXCORE!PicoSystemCallDispatch
fffffa82`8f607450  fffff806`1b14aad0 LXCORE!PicoThreadExit
fffffa82`8f607458  fffff806`1b14a970 LXCORE!PicoProcessExit
fffffa82`8f607460  fffff806`1b14a700 LXCORE!PicoDispatchException
fffffa82`8f607468  fffff806`1b14aa40 LXCORE!PicoProcessTerminate
fffffa82`8f607470  fffff806`1b14ac80 LXCORE!PicoWalkUserStack
fffffa82`8f607478  fffff806`1b0e3fa0 LXCORE!LxpProtectedRanges
fffffa82`8f607480  fffff806`1b14aca0 LXCORE!PicoGetAllocatedProcessImageName
fffffa82`8f607488  00100801`00101081
fffffa82`8f607490  00000000`00000001
```

中间部分便是LXCORE提供给内核的回调函数，例如，当Pico进程执行系统调用时，NT内核便会调用PicoSystemCallDispatch，转交给LXCORE继续分发和处理。

```
00 LXCORE!PicoSystemCallDispatch
01 nt!PsPicoSystemCallDispatch
02 nt!KiSystemServiceUser
```

当Pico进程内发生异常时，NT内核的异常分发函数KiDispatchException会调用PicoDispatchException，例如：

```
00 LXCORE!PicoDispatchException
01 nt!KiDispatchException
02 nt!KiExceptionDispatch
03 nt!KiPageFault
```

注册成功后，NT内核也会返回一个类似的结构，包含一组函数供Pico提供器调用。

```
7: kd> dqs fffff806`1b10b0a0
fffff806`1b10b0a0  00000000`00000060
fffff806`1b10b0a8  fffff802`5af1a8c0 nt!PspCreatePicoProcess
fffff806`1b10b0b0  fffff802`5af1ab40 nt!PspCreatePicoThread
fffff806`1b10b0b8  fffff802`5ad96c50 nt!PspGetPicoProcessContext
fffff806`1b10b0c0  fffff802`5ad96c60 nt!PspGetPicoThreadContext
fffff806`1b10b0c8  fffff802`5acf3e20 nt!PspGetContextThreadInternal
fffff806`1b10b0d0  fffff802`5acf3c20 nt!PspSetContextThreadInternal
fffff806`1b10b0d8  fffff802`5acc1390 nt!PspTerminateThreadByPointer
fffff806`1b10b0e0  fffff802`5acc1930 nt!PsResumeThread
fffff806`1b10b0e8  fffff802`5aa506c0 nt!PspSetPicoThreadDescriptorBase
fffff806`1b10b0f0  fffff802`5acc1f10 nt!PsSuspendThread
fffff806`1b10b0f8  fffff802`5af1afb0 nt!PspTerminatePicoProcess
```

在启用了WSL后，LINUX子系统中的每个Linux进程都是一个Pico进程，比如下面是top进程的EPROCESS结构的地址和概要信息。

```
PROCESS ffffc202da46b080
    SessionId: 1  Cid: 2a80    Peb: 00000000  ParentCid: 2ec0
    DirBase: 1988db000  ObjectTable: ffffda8331c03540  HandleCount:   0.
    Image: System Process
```

注意，其中Peb值为00000000，可执行文件名显示为“System Process”。

Pico进程的EPROCESS结构还有两个特征：首先，Flags2的PicoCreated标志位（第10位）为1；其次，PicoContext字段是一个指针，指向的是Pico提供器使用的Pico上下文结构。

## 2.11　任务管理器

观察进程的一个更简单的方法就是使用Windows操作系统自带的任务管理器。有3种方法可以启动任务管理器：按Ctrl+Shift+Esc组合键；在任务栏上右击，然后选择任务管理器；按Ctrl+Alt+Del组合键，然后选择任务管理器。

“任务管理器”窗口中有多个选项卡，我们重点介绍调试时常用的“详细信息”（老版本为“进程”）选项卡。这个选项卡的核心内容是进程列表，列表的每一行描述一个进程（系统中断行除外），每一列描述进程的一个属性。列的内容是可定制的，默认显示的是常用的进程属性。定制列的方法都是通过“选择列”对话框实现的。在老版本中，弹出这个对话框的方法与在新版本中不同，老版本中是通过View菜单中的Select Columns弹出的，新版本中是通过右击表格的列标题，激活右键菜单弹出的。

无论对于调试高手还是初学者，任务管理器都是一个非常好的帮手。下面分享笔者使用任务管理器的一些经验和技巧。

系统空闲（IDLE）进程的进程ID总是为0，它的线程数就等于系统中的总CPU个数。例如，在笔者现在使用的系统中，一共有 8 个逻辑CPU（四核，启用了超线程），那“线程”刚好为“8”。

“CPU时间”列显示的是CPU的净时间，也就是CPU在该进程上运行的总时间。观察该列，可以知道CPU的时间都花在哪里了。一般来说，一个进程的累计CPU时间达到分钟级别就算比较多了，如果达到小时级别，就代表比较重的进程了。当没有任务执行时，CPU就会执行空闲进程，所以空闲进程的总时间大约等于系统的总开机时间乘以CPU个数。或者说空闲进程的总时间除以CPU个数约等于系统的总开机时间。

当分析高CPU占用率的问题时，“CPU”列显示的是上1s的CPU占用率，是针对系统中所有CPU计算的百分比。这意味着，对于笔者使用的8个CPU的系统，如果“CPU”列的数值始终在12（单位是秒）左右，就代表对应进程中可能有一个线程陷入死循环了。

当分析磁盘有关的问题时，可以选择以I/O开头的多个列，比如“I/O读取”“I/O写入”代表的是I/O次数，如果希望知道访问的字节数，则可以选择“I/O读取字节”列、“I/O写入字节”列。

如果分析内存有关的问题，可以通过“工作集（内存）”“峰值工作集（内存）”和“工作集增量（内存）”来了解物理内存的使用情况，通过“提交大小”了解虚拟内存的使用情况，通过“页面错误”和“页面错误增量”来了解触发页面错误的情况。

任务管理器的默认更新间隔是1s，所以凡是增量性质的数据都是上1s内的变化情况。

除了任务管理器之外，其他一些工具也可用于观察进程的内部信息，比如Process and Thread Status（PStat.exe）、Process Tree（PTree.exe）、Process Explorer（ProcExp.exe）、Process Viewer（PView.exe）和Task List（TList.exe）等。

## 2.12　本章总结

进程和线程分别代表着软件的空间与生命，是极其重要的两个概念。本章从进程和线程的概念开始，先详细介绍进程（2.2～2.6节），再过渡到线程（2.7节），然后按照从一般到特殊的规律，介绍了用于兼容32位程序的WoW进程以及满足特殊用途的最小进程和Pico进程，最后介绍了观察进程和线程属性的最常用工具——任务管理器。
