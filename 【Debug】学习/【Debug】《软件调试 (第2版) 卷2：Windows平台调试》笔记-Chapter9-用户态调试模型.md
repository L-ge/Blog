# 第三篇　操作系统的调试支持

操作系统（Operating System，OS）是计算机系统中的基本软件，它负责统一管理系统中的软硬件资源，为系统中运行的应用软件（application）提供服务，是应用软件运行的基础。操作系统所提供的服务因其设计目的和使用环境不同而有所差异，但通常都包括文件管理、内存管理、进程管理、打印管理、网络管理等基本功能。除了这些功能，如何支持调试也是操作系统设计的一项根本任务，从被调试对象的角度来看，可以把操作系统的调试支持分为以下3个方面。

第一，对应用程序调试（application debugging）的支持，即如何简单高效地调试运行在系统中的各种应用程序。应用程序通常在操作系统分配的较低优先级下运行，其代码属于操作系统不信赖的代码。

第二，对设备驱动程序调试（device driver debugging）的支持，设备驱动程序或其他运行在内核模式的模块是操作系统的可信赖代码，通常与操作系统运行在同一个优先级下和同一个地址空间中，因此调试这些模块通常与调试应用程序有很大不同。

第三，对操作系统自身调试的支持，即如何调试操作系统的各个组成部分。例如调试正在开发的系统模块，以及定位产品发布后出现的系统故障。

调试器（debugger）是软件调试最重要的工具，使用调试器调试是解决复杂软件问题的首选途径。但是在某些情况（比如产品发布后在用户环境中出现问题）下，并不具备使用调试器调试的条件，因此就必须考虑如何在没有调试器的情况下进行调试。从这个意义上讲，对于以上每类调试对象（任务），还必须考虑两种情况。

第一，使用调试器的调试，即通过有效的模型和系统机制来支持调试器软件操纵和访问被调试对象。

第二，不使用调试器的调试，即通过操作系统的基础服务，支持软件实现各种不依赖于调试器的调试途径，比如错误提示、事件追踪、日志和错误报告等。

综合以上分析，可以把操作系统的调试支持归纳为如下表所示的6个问题。

| 对比项目 | 使用调试器的调试 | 不使用调试器的调试 |
| --- | --- | --- |
| 调试应用程序 | 如何使系统中的应用程序可以被调试器调试？（问题A） | 如何使系统中的应用程序在没有调试器时也具有很好的可调试性？（问题B） |
| 调试驱动程序 | 如何使系统中的驱动程序可以被调试器调试？（问题C） | 如何使系统中的驱动程序在没有调试器时也具有很好的可调试性？（问题D） |
| 调试操作系统自身 | 如何使操作系统自身的代码可以被调试器调试？（问题E） | 如何使系统本身在没有调试器时也具有很好的可调试性？（问题F） |


本篇以Windows操作系统为例详细探讨操作系统对软件调试的支持，主要分为如下5个板块。

- 第9章和第10章着重讨论问题A，包括支持应用程序调试的用户态调试子系统、调试会话的建立过程及调试事件的产生和分发机制等。 

- 与CPU异常相呼应，第11章和第12章从操作系统的层面分析异常的分发和处理机制及系统处置未处理异常的方法。该板块的主题是异常处理，它与6个问题都相关。 

- 第13～17章把视线转向调试器之外的辅助调试机制（问题B、D和F）。第13章分析Windows系统提供的错误提示机制。第14章介绍Windows XP系统引入的错误报告（WER）机制。第15章分析Windows的事件日志（event log）机制。第16章分析Windows事件追踪（ETW）机制。 

- 内核调试对于解决系统级的问题和学习操作系统有着非常重要的意义，因此，第18章深入介绍Windows系统的内核调试引擎。 

- 第19章介绍用于提高测试和调试效率的验证机制，包括应用程序验证和驱动程序验证。 


# 第9章　用户态调试模型

应用程序（application program或application）是指能够解决某一个问题或满足某一种应用的特定程序，它是相对系统程序而言的。从用户的角度来看，大多数用户使用计算机是为了使用应用程序，系统程序的作用是为应用程序提供服务。以操作系统为例，用户购买和安装操作系统的目的主要是在其上运行应用程序。换言之，操作系统存在的意义在于通过应用程序满足用户的使用需要。因此，是否具有丰富的应用程序是操作系统成功的一个关键。

如何才能有丰富的应用程序呢？除了要有一套强大而且易用的应用程序编程接口（Application Programming Interface，API），还需要有高效的开发和调试环境。提高应用程序开发效率的一个重要课题就是提高应用程序调试（application debugging）的效率，因为应用程序调试很多时候耗费了比设计编码还要多的时间。

支持应用程序调试一直是操作系统设计中的一项重要任务。为了与调试运行在内核模式下的驱动程序或操作系统自身的代码相区分，通常把调试应用程序称为用户态调试（user mode debugging）。

本章将介绍Windows操作系统中用于支持用户态调试的模型和各种基础设施，包括内核中的调试支持例程、调试子系统和调试API等。

需要说明的一点是，本章和第10章所说的调试都是指使用调试器进行的调试，而不是广义上的软件纠错行为。

## 9.1　概览

回想我们在调试器（如WinDBG）中调试程序（如HelloWorld程序）的情景，可以随时将被调试程序中断到调试器中，然后观察变量信息或跟踪执行，仿佛一切都在调试器中进行。事实上，从进程的角度来看，WinDBG和HelloWorld是两个分别运行的进程，分别有自己的进程空间；而且根据我们在第8章对进程和进程空间的介绍，每个进程的空间都是受到严密的系统机制所保护的。那么，调试器进程是如何“轻而易举”地观察和控制被调试进程的呢？简单的回答是使用调试API。但要深入理解调试API是如何工作的，就必须挖掘调试子系统在调试中所起的作用。

事实上，当我们使用WinDBG调试HelloWorld进程时，除了WinDBG进程和HelloWorld进程，还有一些重要角色积极地参与到这个过程中，正是它们的“努力工作”，才使得调试过程如此得心应手。它们是谁呢？在这一节中，我们就来概览参与用户态调试的各个角色。

### 9.1.1　参与者

原书P168的图9-1显示了在Windows系统下进行用户态调试时参与调试过程的各个角色，包括调试器进程（debugger process）、被调试进程（debuggee process）、调试子系统、调试API，以及位于NTDLL.DLL和内核中的调试支持函数。

首先，调试器进程是调试过程的主导者，它负责发起调试对话，读取和处理调试事件，并通过用户界面接受调试人员下达的指令，然后执行。调试器进程通过调试API（9.8节）与系统的调试支持函数和调试子系统交互，这样不仅简化了调试器的开发工作，而且大大降低了调试器与调试子系统的耦合度。当对调试子系统进行革新时，只要保持调试API不变就可以保证调试器依然可以工作。

被调试进程是调试的目标。为了降低海森伯效应（28.6节），应尽可能少地向被调试进程中加入支持调试的设施，以避免影响问题的重现和分析。但某些调试功能需要在被调试进程中做少量标记或执行简单的动作（后文详述）。

调试子系统是沟通被调试进程和调试器进程的桥梁，它的职责是帮助调试器完成各种调试功能，比如控制和访问被调试进程，管理和分发调试事件，接受和处理调试器的服务请求。从内部来看，调试器大多时候是在和调试子系统对话。

### 9.1.2　调试子系统

调试子系统主要由3个部分——位于NTDLL.DLL中的支持函数、位于内核文件中的支持函数，以及调试子系统服务器组成。

NTDLL.DLL中的调试支持函数主要分为3类。第一类是以DbgUi开头的，供调试器使用；第二类是以DbgSs开头的，供调试子系统使用，这一部分在Windows 2000之后被移除；第三类是以Dbg开头（非前两种）的，用于实现调试API，如DbgBreakpoint是DebugBreak API的实现。根据第3章对NTDLL.DLL的介绍，尤其是图3-1，我们知道，NTDLL.DLL是所有用户态进程都使用的一个模块，因此放在这个模块中的函数具有共享性。这也是要把以上3类调试函数放在NTDLL.DLL中的一个原因。

内核文件中的调试支持函数负责采集和传递调试事件，以及控制被调试进程。这些内核函数都是以Dbgk开头的，我们在9.2节和9.3节详细介绍它们。

调试子系统服务器的主要职能是管理调试会话和调试事件，是调试消息（事件）的集散地，也是所有调试设施的核心。

调试子系统是操作系统的一个组成部分，其实现因操作系统的不同而不同。对于同一操作系统，不同的版本也可能包含不同的实现。Windows NT（3.x和4.0）与Windows 2000的调试子系统基本一致。Windows XP做了较大改进，增加了专门用于应用程序调试的内核对象（Debug Object），并将调试子系统服务器从用户模式移入内核模式中，但是在API层仍与以前兼容。Windows Vista沿用了Windows XP改进后的实现。

### 9.1.3　调试事件驱动

与Windows程序的消息驱动机制类似，Windows用户态调试是通过调试事件来驱动的。调试器程序在与被调试进程建立调试会话（10.3节和10.4节）后，调试器进程便进入所谓的调试事件循环（debug event loop），等待调试事件的发生，然后处理，再等待，直到调试会话终止。其核心代码如下。

```
while (WaitForDebugEvent(&DbgEvt, INFINITE)) // 等待事件
{
   // 处理等待得到的事件
   // 处理后，恢复调试目标继续执行
   ContinueDebugEvent(DbgEvt.dwProcessId, DbgEvt.dwThreadId, 
       dwContinueStatus); 
}
```

其中，`WaitForDebugEvent`用于等待和接收调试事件，收到调试事件后，调试器便根据事件的类型（事件ID）来分发和处理，并根据情况决定是否要通知用户并进入交互式调试（命令模式，29.7节）。在处理调试事件的过程中，被调试进程是处于挂起状态的（9.3节）。处理调试事件后，调试器调用`ContinueDebugEvent`将处理结果回复给调试子系统，让被调试程序继续运行，调试器则再次调用`WaitForDebugEvent`等待下一个调试事件。`WaitForDebugEvent`和`ContinueDebugEvent`都是Windows提供的调试API，我们在9.8节详细介绍。

调试子系统的内核部分设计了一系列函数用来采集调试事件，并以一个消息结构发送给调试子系统，使其保存在调试子系统的调试消息队列中。调试子系统和调试器之间是靠一个内核对象来同步的。当有调试消息需要读取时，调试子系统服务器会设置这个同步对象，使等待这个对象的调试器线程被唤醒。

在内核中，调试事件有时也称为调试消息，并使用一个名为`DBGKM_APIMSG`的结构来描述。在发送给调试器时，调试API使用的是一个名为`DEBUG_EVENT`的结构（10.5.1节）。因为这两个结构是不同的，所以需要一个转化过程，这个工作是由调试子系统服务器和NTDLL.DLL中的用户态函数来完成的。简单来说，子系统服务器会将自己使用的结构转化为NTDLL.DLL使用的`DBGUI_WAIT_STATE_CHANGE`，NTDLL.DLL再将这个结构转化为调试器使用的`DEBUG_EVENT`结构。

本节概括性地介绍了用户态调试的所有参与者，以及它们之间是如何以调试事件为纽带而协同工作的。接下来的几节将围绕调试事件做进一步介绍。

## 9.2　采集调试消息

为了能了解到调试有关的系统动作，调试子系统的内核部分对外公开了一系列函数，供内核的其他部分调用，以便得到“知情”和处理机会。这些函数都是以Dbgk（不是Dbgkp，p代表内部过程）开头的，是调试子系统公开给内核其他部件的接口函数，我们将它们简称为Dbgk采集例程。
 
### 9.2.1　消息常量

Dbgk采集例程将所有调试事件（消息）分为8种类型，并使用以下常量来代表不同类型的调试消息。

```
typedef enum _DBGKM_APINUMBER
{
    DbgKmExceptionApi = 0,        //异常
    DbgKmCreateThreadApi = 1,     //创建线程
    DbgKmCreateProcessApi = 2,    //创建进程
    DbgKmExitThreadApi = 3,       //线程退出
    DbgKmExitProcessApi = 4,      //进程退出
    DbgKmLoadDllApi = 5,          //映射DLL
    DbgKmUnloadDllApi = 6,        //反映射DLL
    DbgKmErrorReportApi = 7,      //内部错误
    DbgKmMaxApiNumber = 8,        //这组常量的最大值
} DBGKM_APINUMBER;
```

其中`DbgKmErrorReportApi`用来报告调试子系统内部的错误，目前已经不再使用。下面将分别介绍其他几种调试消息的采集过程。

### 9.2.2　进程和线程创建消息

操作系统的一大核心任务就是管理系统中运行的各个进程和线程，包括创建新的进程和线程，调度等待运行的线程，负责进程间通信，终止进程和线程，以及分配、回收资源等，这些任务通常统称为进程管理。操作系统的很多模块与进程管理有关，但进程管理的最核心部分是由位于Windows执行体（NTOSKRNL.EXE的上半部分，图3-1）中的一系列函数完成的，这些函数大多以Ps或Psp开头，比如`PsCreateSystemThread ()`、`PspShutdownThread ()`等。为了描述简便，通常把这些函数（以及这些函数使用的数据结构）泛称为进程管理器（process manager）。

当进程管理器创建新的用户态Windows线程时，它首先要为该线程建立必要的内核对象与数据结构，并分配栈（stack）空间，这些工作完成后，该线程就处于挂起（CREATE_SUSPEND）状态；而后进程管理器会通知环境子系统，子系统会做必要的设置和登记；最后进程管理器会调用`PspUserThreadStartup`例程，准备启动该线程。为了支持调试，`PspUserThreadStartup`总是会调用调试子系统的内核函数`DbgkCreateThread`，以便让调试子系统得到处理机会。

`DbgkCreateThread`会检查新创建线程所在的进程是否正在被调试（根据DebugPort是否为空）。如果不是，便立即返回；如果是（DebugPort不为空），则会继续检查该进程的用户态运行时间（UserTime）是否为0，目的是判断该线程是否是进程中的第一个线程。如果是，则通过`DbgkpSendApiMessage ()`函数向DebugPort发送`DbgKmCreateProcessApi`消息；如果不是，则发送`DbgKmCreateThreadApi`消息。调试器收到的进程创建（`CREATE_PROCESS_DEBUG_EVENT`，值为3）和线程创建（`CREATE_THREAD_DEBUG_EVENT`，值为2）事件就是源于这两个消息的。

### 9.2.3　进程和线程退出消息

进程管理器的`PspExitThread`函数负责线程的退出和清除。为了支持调试，在销毁该线程的结构和资源之前，`PspExitThread`会调用调试子系统的函数以便让调试器（如果有）得到处理机会。如果正在退出的不是进程中的最后一个线程，那么`PspExitThread`会调用`DbgkExitThread`函数通知指定线程要退出；如果是最后一个线程，那么`PspExitThread`便会调用`DbgkExitProcess`函数通知指定进程要退出。

`DbgkExitThread`会检查进程的DebugPort是否为0，如果不为0，则会先将该进程挂起，然后通过`DbgkpSendApiMessage`函数向DebugPort发送`DbgKmExitThreadApi`消息，待发送函数返回后再恢复进程运行。

`DbgkExitProcess`的执行过程非常类似，只不过发送的是`DbgKmExitProcessApi`消息，而且没必要执行挂起和恢复动作，因为进程管理器已经对该线程做了删除标记。

调试器接收到的线程退出事件（`EXIT_THREAD_DEBUG_EVENT`，值为4）和进程退出（`EXIT_PROCESS_DEBUG_EVENT`，值为5）是源于这两个消息的。

### 9.2.4　模块映射和反映射消息

自Windows诞生以来，动态链接库（Dynamic Link Library，DLL）一直是Windows系统中使用最多的技术之一。Windows操作系统内核、Windows API和无以计数的Windows应用程序都普遍使用了DLL技术。比如，Windows的内核文件NTOSKRNL.EXE虽然以EXE为后缀，其实质就是一个DLL；NTDLL.DLL是连接用户态和操作系统内核的桥梁（图3-1），用户态的代码通过它访问内核服务；Windows子系统DLL（KERNEL32.DLL、ADVAPI32.DLL、USER32.DLL、GDI32.DLL）是Windows API的载体，当使用Windows API的应用程序正在执行时都离不开这些DLL。观察Windows的系统目录（winnt\system32）还会看到很多其他的DLL。除了Windows系统自带的DLL，应用软件本身经常有自己的DLL。DLL不可以独立拥有进程和运行，但可以被EXE程序加载到其所属的进程空间，并被调用和执行。

很多工具可以用来观察进程中的DLL使用情况。运行记事本程序，启动VS，通过选择“调试”→“附加到进程”菜单弹出Attach Process对话框，然后选择notepad。而后再通过选择“调试”→“窗口”→“模块”菜单弹出的“模块”窗口，便可以看到notepad进程中的DLL了。第二列是该模块在进程空间中的地址（虚拟地址，均小于0x80000000，可见这些模块都是位于用户空间中的）。

在上面“模块”窗口中可以看到Windows子系统DLL等，重复上面的步骤观察其他的Windows进程，通常也会看到这些DLL。那么，这些存在于多个进程空间中的DLL是不是要重复占用内存呢？答案是否定的，当Windows系统的DLL加载函数（`LoadLibrary()`和`LoadLibraryEx()` API及未公开的用户态函数和内核API）要加载一个DLL时，会首先判断该DLL是否已经加载过，如果是，则不会重复加载，只要将该DLL对应的内存页面映射（map）到目标进程的内存空间，并把该DLL的引用次数加1。当一个进程退出或调用`FreeLibrary()` API卸载一个DLL时，Windows会从进程的虚拟内存空间中把该DLL的映射删除（unmap），并递减该DLL的引用次数，如果引用次数变为0，那么该DLL会被彻底移出内存。

Windows内核中的内存管理器（memory manager）负责DLL的映射和反映射。在内部，内存管理器使用Section对象（Windows子系统称为file mapping object，即文件映射对象）来表示一块可被多个进程共享的内存区域，并设计了一系列内核服务和函数来实现各种映射与反映射任务。使用WinDBG的`x`命令可以看到这些内核函数。

```
lkd> x nt!Nt*mapvie*
805a6526 nt!NtMapViewOfSection = <no type information>
805a733c nt!NtUnmapViewOfSection = <no type information>
```

其中`NtMapViewOfSection`是用来映射模块的内核服务的，`NtUnmapViewOfSection`是用来反映射的。

为了支持调试，当`NtMapViewOfSection`把一个模块映像（表示为section对象）成功映射到指定进程的空间中（使用`MmMapViewOfSection`）时，`NtMapViewOfSection`会调用调试子系统的`DbgkMapViewOfSection`函数通知调试子系统。清单9-1所示的函数调用序列显示了进程初始化期间加载DLL文件的整个过程。

清单9-1　加载DLL文件的整个过程

```
kd> kn
 # ChildEBP RetAddr  
00 bcb59abc 80494840 nt!LpcRequestWaitReplyPort           // 使用LPC发送并等待回复
01 bcb59bdc 8052a3cb nt!DbgkpSendApiMessage+0x43          // 发送调试消息
02 bcb59ca8 804b61a8 nt!DbgkMapViewOfSection+0xe8         // 通知调试子系统
03 bcb59d34 80461691 nt!NtMapViewOfSection+0x333          // 系统服务的内核函数
04 bcb59d34 77f86839 nt!KiSystemService+0xc4              // 系统服务分发例程
05 0006f7dc 77f94020 ntdll!ZwMapViewOfSection+0xb         // 调用系统服务
06 0006f870 77f85478 ntdll!LdrpMapDll+0x199               // 加载器的模块映射函数
07 0006f8a4 77f95f18 ntdll!LdrpLoadImportModule+0x62      // 加载依赖的模块
08 0006f8fc 77f8548a ntdll!LdrpWalkImportDescriptor+0x96  // 遍历模块的输入表
09 0006f920 77f95f18 ntdll!LdrpLoadImportModule+0x70      // 加载依赖的模块
0a 0006f978 77f8651e ntdll!LdrpWalkImportDescriptor+0x96  // 遍历模块的输入表
0b 0006fc98 77f96416 ntdll!LdrpInitializeProcess+0x70a
0c 0006fd1c 77f9fb67 ntdll!LdrpInitialize+0x175           // 加载器的初始化函数
0d 00000000 00000000 ntdll!KiUserApcDispatcher+0x7        // 异步过程调用到用户空间
```

由下而上，栈帧#0c～#06是NTDLL.DLL中的映像加载函数，它们都以`Ldr`（Loader）开头，是Windows系统的加载器函数，#0a是遍历模块的输入表，#09准备加载一个模块，#08是遍历这个要加载模块的输入表，#06是用户模式的DLL映射函数，它在内部调用系统服务`ZwMapViewOfSection`，之后便进入内核模式。

`DbgkMapViewOfSection`被调用后会检查当前进程的DebugPort是否为空，如果不为空，则通过`DbgkpSendApiMessage`函数发送`DbgKmLoadDllApi`消息。

类似地，当内存管理器反映射一个模块映像时，`MmUnmapViewOfSection`函数会调用调试子系统的`DbgkUnMapViewOfSection`函数。该函数在检测到当前进程的DebugPort不为空后，会发送`DbgKmUnloadDllApi`消息。

调试器接收到的模块映射事件（`LOAD_DLL_DEBUG_EVENT`，值为6）和反映射（`UNLOAD_DLL_DEBUG_EVENT`，值为7）是源于这两个消息的。

### 9.2.5　异常消息

异常与调试有着密不可分的关系，很多软件错误就是与异常有关的，因此调试器应该能够知道并控制被调试程序中的异常。另外，很多调试机制是以异常机制为基础的，比如断点和单步执行分别是依靠断点异常（#BP）和调试异常（#DB）来工作的。

为了支持调试，系统会把被调试程序中发生的所有异常发送给调试器。第11章会详细介绍异常的分发过程，在此只需要知道内核中的`KiDispatchException`函数是分发异常的枢纽，它会给每个异常安排最多两轮被处理的机会。对于每一轮处理机会，它都会调用调试子系统的`DbgkForwardException`函数来通知调试子系统。

`DbgkForwardException`函数既可以向进程的异常端口发送消息，也可以向调试端口发送消息，`KiDispatchException`函数在调用它时会通过一个布尔类型的参数来指定。如果要向调试端口发送消息，那么`DbgkForwardException`函数会判断进程的`DebugPort`字段是否为空，如果不为空，便通过`DbgkpSendApiMessage`函数发送`DbgKmExceptionApi`消息。

调试器收到的异常事件（`EXCEPTION_DEBUG_EVENT`，值为1）和输出调试字符串（`OUTPUT_DEBUG_STRING_EVENT`，值为8），都是源于`DbgKmExceptionApi`消息的。我们在10.7节详细介绍输出调试字符串的细节。

前面分别介绍了各种调试消息的采集过程。简单来说，系统的进程管理器、内存管理器和异常分发函数会调用调试子系统的Dbgk采集例程，向调试子系统通报调试消息。这些例程被调用后会根据当前进程的`DebugPort`字段来判断当前进程是否处于被调试状态。如果不是，便忽略这次调用，直接返回；如果是，便产生一个`DBGKM_APIMSG`结构，然后调用9.3节介绍的`DbgkpSendApiMessage`函数来发送调试消息。

## 9.3　发送调试消息

9.2节介绍了调试子系统的内核函数采集调试事件的方法和过程。本节将继续介绍调试消息发送给调试子系统服务器的过程。

### 9.3.1　调试消息结构

首先，调试子系统的内核函数使用以下结构来描述和传递调试消息。

```
typedef struct _DBGKM_APIMSG 
{
    PORT_MESSAGE h;                  //LPC端口消息结构，Windows XP之前使用
    DBGKM_APINUMBER ApiNumber;       //消息类型
    NTSTATUS ReturnedStatus;         //调试器的回复状态
    union {                          //具体描述消息详情的联合结构
        DBGKM_EXCEPTION Exception;               //异常
        DBGKM_CREATE_THREAD CreateThread;        //创建线程
        DBGKM_CREATE_PROCESS CreateProcessInfo;   //创建进程
        DBGKM_EXIT_THREAD ExitThread;             //线程退出
        DBGKM_EXIT_PROCESS ExitProcess;           //进程退出
        DBGKM_LOAD_DLL LoadDll;                   //映射DLL
        DBGKM_UNLOAD_DLL UnloadDll;               //反映射DLL
    } u;
} DBGKM_APIMSG, *PDBGKM_APIMSG;
```

其中，`ApiNumber`就是我们在9.2节介绍过的枚举常量，用来表示消息的类型；`ReturnedStatus`用来存放调试器的回复信息；联合体`u`的内容因为消息类型的不同而不同，用来描述消息的参数和详细信息。例如，当`ApiNumber`等于`DbgKmExceptionApi(0)`时，联合部分是一个`DBGKM_EXCEPTION`结构，其余的以此类推。

调试消息采集函数在确认需要向调试子系统报告消息后，它会填写`DBGKM_APIMSG`结构，然后将其作为参数传给`DbgkpSendApiMessage`函数。

### 9.3.2　DbgkpSendApiMessage函数

`DbgkpSendApiMessage`函数用来将一条调试消息发送到调试子系统服务器。

```
NTSTATUS DbgkpSendApiMessage(
    IN OUT PDBGKM_APIMSG ApiMsg,
    IN PVOID Port,
    IN BOOLEAN SuspendProcess)
```

其中`ApiMsg`用来描述消息的详细信息，`Port`用来指定要发往的端口，大多数时候就是EPROCESS结构中的`DebugPort`字段的值，偶尔是进程中的异常端口，即`ExceptionPort`字段。

如果SuspendProcess为真，那么这个函数会先调用下面将要介绍的`DbgkpSuspendProcess`函数挂起当前进程，然后发送消息，等收到消息回复后再调用`DbgkpResumeProcess`函数唤醒当前进程。

因为Windows NT和Windows 2000的调试子系统服务器处于用户模式，所以在这些系统中，`DbgkpSendApiMessage`会通过LPC机制来发送调试消息。这时`Port`参数指定的是一个LPC端口，这个端口的监听者通常是Windows环境子系统的服务器进程，即CSRSS。CSRSS收到消息后再发给位于会话管理器进程中的调试子系统服务器，后者再通知等候调试事件的调试器。因为`DbgkpSendApiMessage`内部是调用`LpcRequestWaitReplyPort`函数来完成具体的LPC收发任务的，而且`LpcRequestWaitReplyPort`函数是阻塞的，所以只有在收到回复后，`LpcRequestWaitReplyPort`才会返回。

在Windows XP及其后续的Windows系统中，调试子系统服务器被移到内核空间中，因此这些版本中的`DbgkpSendApiMessage`改为通过调用`DbgkpQueueMessage`来发送消息。`DbgkpQueueMessage`会根据参数决定是否需要等待，如果需要，它会调用等待函数，直到收到调试器的回复后才返回。不需要等待的情况只用于发送杜撰消息（9.4节）等特殊情况，因此如不特别说明，我们讨论的都是需要等待的情况。

### 9.3.3　控制被调试进程

调试子系统设计了两个内核函数来控制被调试进程，它们是`DbgkpSuspendProcess`和`DbgkpResumeProcess`。

在调试子系统向调试器发送调试事件之前，通常会先调用`DbgkpSuspendProcess`函数。这个函数会在内部调用`KeFreezeAllThreads()`函数冻结（freeze）被调试进程中除调用线程之外的所有线程。进一步讲，在调用`DbgkpResumeProcess`函数后，被调试进程中便只有当前线程（发送调试消息的这个线程）还在活动，接下来它会执行实际的消息发送函数，在Windows XP之前调用LPC函数，从Windows XP开始调用`DbgkpQueueMessage`函数。无论哪个函数，对于大多数调试事件，它们都是堵塞的，也就是会调用等待函数（`KeWaitForSingleObject`等）来使当前线程进入等待状态。因为当前进程的其他线程已被此前的`DbgkpSuspendProcess`调用所挂起，所以一旦当前线程进入等待，那么整个被调试进程的所有线程便都不再执行。这可以解释为什么当被调试进程被中断到调试器时，被调试程序没有响应。

接下来的任务就是调试子系统服务器通知调试器读取调试消息，调试器进行处理后回复给调试子系统，后者再唤醒被调试进程的等待线程（发送调试消息的那个）。等待线程醒来后，再执行`DbgkpResumeProcess`函数，后者在内部会调用`KeThawAllThreads()`恢复（unfreeze）被调试进程中的所有线程。

从数据结构的角度来看，在每个Windows线程的KTHREAD结构中，有两个与线程执行状态密切相关的字段，一个叫`FreezeCount`，一个叫`SuspendCount`。对于可调度执行的线程来说，这两个字段的值都为0。`KeFreezeAllThreads()`函数和`KeThawAllThreads()`操作的是`FreezeCount`字段，而`SuspendThread ()`和`ResumeThread ()`API（对应于NtSuspendThread内核服务和KeSuspendThread）操作的是`SuspendCount`字段。

当被调试进程中断到调试器中时，它当前线程的FreezeCount通常为0，其他线程的FreezeCount通常为1。因为`KeFreezeAllThreads`不会冻结当前线程，包括WinDBG在内的调试器在收到调试事件后，会对被调试进程中的所有线程依次调用SuspendThread API，这样所有线程的SuspendCount计数通常都为1。例如，当使用WinDBG调试包含两个线程的MulThreads程序时，当设置在`kernel32!SleepEx`处的断点命中后，使用本地内核调试会话可以观察到MulThreads进程内所有线程的详细信息，如清单9-2所示（为节约篇幅，格式进行了一些调整，并增加了行号）。

清单9-2　MulThreads进程内所有线程的详细信息

```
1 lkd> !PROCESS 88a3a600 2    ** 2代表只显示线程状态信息
2 PROCESS 88a3a600  SessionId: 0  ...     Image: MulThrds.exe
3 THREAD 881fb020  Cid 1e40.1e44  Teb: 7ffdf000 Win32Thread: e4ff3eb0 WAIT: 
4 (Suspended) KernelMode Non-Alertable SuspendCount 1 FreezeCount 1
5             881fb1bc  Semaphore Limit 0x2
6 THREAD 87716ce0  Cid 1e40.1e48  Teb: 7ffde000 Win32Thread: e11a1af8 WAIT: 
7 (Executive) KernelMode Non-Alertable SuspendCount 1
8            a9a1d7d4  SynchronizationEvent
```

从第4行可以看到，线程`1e44`的`SuspendCount`和`FreezeCount`都是`1`。从第7行看，线程`1e48`的`SuspendCount`为`1`，`FreezeCount`为`0`（大于0才显示）。这是因为线程`1e48`中发生了断点异常，调试事件的发送过程是发生在这个线程中的，所以当`KeFreezeAllThreads`执行时，没有冻结这个线程。WinDBG的断点和调试异常处理函数（`ProcessBreakpointOrStepException`）会对所有线程调用SuspendThread API，因此，这两个线程的`SuspendCount`都为`1`。

WinDBG的`～n`和`～m`命令允许用户调整被调试线程的SuspendCount，这两个命令实际上调用的是SuspendThread和ResumeThread的API。

我们前面说过，Windows XP对调试子系统做了重大改变，特别是子系统服务器部分。因此，Windows XP版本之前和之后（包括Windows XP）的调试子系统服务器是不同的。接下来的两节将分别介绍这两种子系统服务器。

## 9.4　调试子系统服务器（Windows XP之后）

与Windows 2000和Windows NT相比，Windows XP对用户态调试子系统做了重大改进，将调试子系统服务器由用户模式移入内核模式，Windows Vista沿用了这一改动。新的子系统服务器是以新引入的内核对象DebugObejct为核心的。本节将围绕这个内核对象介绍Windows XP和Windows Vista的调试子系统服务器。

### 9.4.1　DebugObject

`DebugObject`内核对象是专门用于用户态调试的，它不仅承担了同步调试器和调试子系统的功能，而且也是调试器和调试子系统之间传递数据的重要纽带，取代了调试子系统各部件间本来使用的LPC通信方式。以下是`DebugObejct`对象的内部结构。

```
typedef struct _DEBUG_OBJECT
{
    KEVENT EventsPresent;                //+0x00,用于指示有调试事件发生的事件对象 
    FAST_MUTEX Mutex;                    //+0x10,用于同步的互斥对象
    LIST_ENTRY StateEventListEntry;      //+0x30,保存调试事件的链表
    ULONG Flags;                         //+0x38,标志位，见下文
} DEBUG_OBJECT, *PDEBUG_OBJECT;
```

其中最值得关注的就是`StateEventListEntry`字段，它是一个用来存储调试事件的链表，我们将其称为调试消息队列。`EventsPresent`用来同步调试器进程和被调试进程，调试子系统服务器通过设置此事件来通知调试器读取消息队列中的调试消息。调试器进程通过`WaitForDebugEvent`API来等待调试事件，而`WaitForDebugEvent`API对应的`NtWaitForDebugEvent`内核服务内部实际上等待的就是这个`EventsPresent`对象。互斥对象（`Mutex`）用来锁定对这个数据结构的访问，以防止多个线程同时读写造成数据错误。Flags字段包含多个标志位，比如，位1代表结束调试会话时是否终止被调试进程（KillProcessOnExit），`DebugSetProcessKillOnExit`API实际上设置的就是这个标志位。

### 9.4.2　创建调试对象

内核服务`NtCreateDebugObject`用来创建调试对象。当调试器与调试子系统建立连接时（10.3和10.4节），调试子系统会为其创建一个调试对象，并将其保存在调试器当前线程的线程环境块的`DbgSsReserved`字段中。`DbgSsReserved`字段中保存的调试对象是这个调试器线程区别于其他普通线程的重要标志，详见10.1.3节。

### 9.4.3　设置调试对象

调试对象通常是在调试器进程中创建的，为了起到联系被调试进程和调试进程的作用，需要将其设置到被调试进程的EPROCESS结构的`DebugPort`字段中。

建立应用程序调试对话有两种典型的情况：一种是在调试器中启动被调试程序；另一种是把调试器附加到一个已经运行的进程中。对于前一种情况，系统在创建进程时，会把调试器线程TEB结构的`DbgSsReserved`字段中保存的调试对象句柄传递给创建进程的内核服务，然后内核中的进程创建函数会将这个句柄所对应的对象指针赋给新创建进程的EPROCESS结构的`DebugPort`字段。前面介绍过，`DebugPort`字段是系统判断一个进程是否正在被调试的标志。收集调试消息的Dbgk函数通过判断`DebugPort`字段来决定是否要产生和发送调试消息。对于后一种情况，系统会调用内核中的`DbgkpSetProcessDebugObject`函数来将一个创建好的调试对象附加到其参数所指定的进程中，也就是要被调试的进程。

`DbgkpSetProcessDebugObject`函数内部除了将调试对象赋给EPROCESS结构的`DebugPort`字段，还会调用`DbgkpMarkProcessPeb`函数设置进程环境块（PEB）的`BeingDebugged`字段。

### 9.4.4　传递调试消息

`DbgkpQueueMessage`函数用于向一个调试对象的消息队列中追加调试事件。指定调试对象的方法有两个：一是直接在参数中指定调试对象；二是指定EPROCESS结构，`DbgkpQueueMessage`函数会使用这个结构中的`DebugPort`字段所代表的调试对象。

调试消息队列的每个节点是一个名为`DEBUG_EVENT`的数据结构，其名称与调试API中的`DEBUG_EVENT`结构同名，但内容完全不同，为了避免混淆，本书将内核中的`DEBUG_EVENT`结构称为`DBGKM_DEBUG_EVENT`。根据参考资料（Alex Ionescu. Kernel User-Mode Debugging Support (Dbgk).），`DBGKM_DEBUG_EVENT`结构的定义如下。

```
typedef struct _DBGKM_DEBUG_EVENT
{
    LIST_ENTRY EventList;       //与兄弟节点相互链接的节点结构
    KEVENT ContinueEvent;       //用于等待调试器回复的事件对象
    CLIENT_ID ClientId;         //调试事件所属的线程ID和进程ID
    PEPROCESS Process;          //被调试进程的EPROCESS结构地址
    PETHREAD Thread;            //被调试进程中触发调试事件的线程的ETHREAD地址
    NTSTATUS Status;            //对调试事件的处理结果
    ULONG Flags;                //标志
    PETHREAD BackoutThread;     //产生杜撰消息（faked message）的线程
    DBGKM_MSG ApiMsg;           //调试事件的详细信息
} DBGKM_DEBUG_EVENT, *PDBGKM_DEBUG_EVENT;
```

其中`ClientId`字段是一个`CLIENT_ID`结构，包含两个DWORD，分别代表调试事件所属的进程ID和线程ID。

在给`DBGKM_DEBUG_EVENT`结构赋值后，`DbgkpQueueMessage`函数会将其插入调试对象的消息链表（StateEventListEntry）中。

而后`DbgkpQueueMessage`函数会根据参数中是否指定了不需等待（NOWAIT，值为2）的标志，决定是否要立刻通知调试器来读取消息。如果指定了，便返回；如果没有指定，便设置调试对象的EventsPresent对象，通知调试器有消息需要读取，然后调用`KeWaitForSingleObject`等待`DEBUG_EVENT`结构中的`ContinueEvent`对象，等待调试器的回复。

在调试器处理好调试事件后，它会通过`ContinueDebugEvent`API间接调用或直接调用`nt!NtDebugContinue`内核服务。`NtDebugContinue`会根据参数中指定的`CLIENT_ID`结构找到要恢复的调试事件结构，然后设置它的`ContinueEvent`事件对象，使处于等待的被调试线程被唤醒而继续执行。

清单9-3所示的函数调用序列记录了断点异常的分发和发送过程，从触发异常开始（栈帧#07），到放入调试事件队列（栈帧#01），再到设置`EventsPresent`对象（栈帧#00）。该线程便是所谓的RemoteBreakin线程（10.6.4节），是调试器（WinDBG）在被调试进程中创建的，用于产生断点异常，以响应中断（break）到调试器的命令。

清单9-3　函数调用序列

```
kd> kn
 # ChildEBP RetAddr  
00 f5fb87bc 805bd170 nt!KeSetEvent+0x1                [设置事件]
01 f5fb8894 805bc37a nt!DbgkpQueueMessage+0x13f       [放入调试事件链表]
02 f5fb88b4 805bc505 nt!DbgkpSendApiMessage+0x43      [格式化为消息结构]
03 f5fb8940 804feb8a nt!DbgkForwardException+0x8d     [转发给调试子系统]
04 f5fb8cf4 804dab0e nt!KiDispatchException+0x150     [异常分发]
05 f5fb8d5c 804db119 nt!CommonDispatchException+0x4d  [建立异常结构]
06 f5fb8d5c 77f767ce nt!KiTrap03+0x97                 [执行INT3 异常的处理例程]
07 0079ffc8 77f7285c ntdll!DbgBreakPoint+0x1          [执行INT 3指令，产生断点异常]
08 0079fff4 00000000 ntdll!DbgUiRemoteBreakin+0x36    [调用DbgUi的中断函数]
```

9.1节简要介绍过，建立调试会话后，调试器工作线程便进入调试事件循环，等待调试事件，这实际上就是调用`NtWaitForDebugEvent`内核服务等待调试对象中的`EventsPresent`对象。因此，当被调试进程中设置了这个对象时，调试器的工作线程就会被唤醒，并开始读取调试对象中的消息队列（StateEventListEntry）。读到一个调试事件后，`NtWaitForDebugEvent`会调用`DbgkpConvertKernelToUserStateChange`函数将`DBGKM_DEBUG_EVENT`结构转换为用户模式下使用的`DBGUI_WAIT_STATE_CHANGE`结构。清单9-4显示了调试器线程等待和读取调试事件的完整过程，从线程启动（BaseThreadStart）到进入调试消息循环（EngineLoop），到等待调试事件（`ZwWaitForDebugEvent`），再到得到通知去读取和转换调试事件（`DbgkpConvert-KernelToUserStateChange`）。

清单9-4　调试器线程等待和读取调试事件的完整过程

```
kd> k
ChildEBP RetAddr  
f9afac80 805bce25 nt!DbgkpConvertKernelToUserStateChange    [读取调试事件]
f9afad4c 804da140 nt!NtWaitForDebugEvent+0x1b8   [NtWaitForDebugEvent内核服务]
f9afad4c 7ffe0304 nt!KiSystemService+0xc4             [内核服务分发]
00a1fd00 77f766fc SharedUserData!SystemCallStub+0x4    [系统调用]
00a1fd04 02242d3e ntdll!ZwWaitForDebugEvent+0xc        [调用等待调试事件的内核服务]
00a1fda0 02107d23 dbgeng!LiveUserDebugServices::WaitForEvent+0x12e
00a1ff10 020a3c3f dbgeng!LiveUserTargetInfo::WaitForEvent+0x3b3
00a1ff34 020a401e dbgeng!WaitForAnyTarget+0x5f        [依次等待每个调试目标]
00a1ff80 020a4290 dbgeng!RawWaitForEvent+0x2ae        [调试引擎的内部函数]
00a1ff98 0102925f dbgeng!DebugClient::WaitForEvent+0xb0     [调试引擎的等待接口]
00a1ffb4 77e7d33b WinDBG!EngineLoop+0x13f            [调试循环]
00a1ffec 00000000 kernel32!BaseThreadStart+0x37        [调试线程启动]
```

读取一个调试事件后，`NtWaitForDebugEvent`会在`DBGKM_DEBUG_EVENT`这个事件结构的Flags字段中设置一个已读标志。

如上文曾提到的，在调试器处理好一个调试事件后，它会调用`ContinueDebugEvent`API让被调试进程继续运行。这个API内部会调用NtDebugContinue内核服务。这个内核服务会遍历调试对象的消息队列，找到匹配的调试事件后调用`DbgkpWakeTarget`函数，来设置ContinueEvent对象唤醒等待的被调试线程。

### 9.4.5　杜撰的调试消息

当将调试器附加到一个已经运行的进程时，为了向调试器报告以前发生的但目前仍有意义的调试事件，调试子系统会“捏造”一些调试消息来模拟过去的调试事件，这样的调试消息称为杜撰的调试消息（faked debug message）。

`NtDebugActiveProcess`是用来与已经运行的进程建立调试会话的内核服务，它在调用`DbgkpSetProcessDebugObject`将调试对象设置到要调试的进程之前，会调用调试子系统的`DbgkpPostFakeProcessCreateMessages`函数。

`DbgkpPostFakeProcessCreateMessages`会先调用`DbgkpPostFakeThreadMessages`，后者会遍历被调试进程的所有线程，以向调试对象的消息队列中投放杜撰的进程和线程来创建消息。而后`DbgkpPostFakeProcessCreateMessages`会调用`DbgkpPostFakeModuleMessages`来投放杜撰的模块加载消息。`DbgkpPostFakeThreadMessages`和`DbgkpPostFakeModuleMessages`都是调用`DbgkpQueueMessage`来向消息队列添加调试消息的，因为在参数中指定了不需等待的标志（NOWAIT），所以`DbgkpQueueMessage`将事件放入队列后便会返回，不会设置`EventsPresent`对象以避免它通知调试器来读取。

`DbgkpPostFakeProcessCreateMessages`返回后，`NtDebugActiveProcess`会调用`DbgkpSetProcessDebugObject`函数来将调试对象设置到要调试的进程。`DbgkpSetProcessDebugObject`内部在成功设置调试对象后，会遍历事件队列中的所有事件，并会设置调试对象的`EventsPresent`字段。这样，`NtDebugActiveProcess`服务返回后，当调试器再调用`NtWaitForDebugEvent`时，它便可以立刻等待成功并读取到事件队列中的调试事件。清单9-5所示的栈回溯序列显示了向消息队列中投递杜撰的调试消息的执行过程。

清单9-5　栈回溯序列

```
kd> kn
# ChildEBP RetAddr  
00 f50cbc18 805bd26b nt!DbgkpQueueMessage                      [放入消息队列]
01 f50cbcec 805bc143 nt!DbgkpPostFakeThreadMessages+0x155      [杜撰线程创建消息]
02 f50cbd30 805bcf2c nt!DbgkpPostFakeProcessCreateMessages+0x2a
03 f50cbd54 804da140 nt!NtDebugActiveProcess+0x8d              [调试已运行进程]
04 f50cbd54 7ffe0304 nt!KiSystemService+0xc4                   [系统服务分发函数]
05 00a1fcb4 00000000 SharedUserData!SystemCallStub+0x4         [系统调用]
```

值得注意的是，以上过程是在调试器进程中执行的。清单9-3所示的对`DbgkpQueueMessage`的调用是发生在被调试进程中的。

### 9.4.6　清除调试对象

当调试结束后需要撤销调试会话时，系统会调用`DbgkClearProcessDebugObject`将被调试进程的`DebugPort`字段恢复为NULL。恢复时，这个函数会遍历调试对象的消息队列，将关于这个进程的调试事件清除。这个函数并不破坏调试对象，因为一个调试器可以同时调试多个被调试进程，这个调试对象可能还在被其他被调试进程所使用。

### 9.4.7　内核服务

配合调试子系统的改变，Windows XP引入了8个新的内核服务。表9-1列出了这些服务的名称和主要功能。

以上内核服务大多是通过调试API提供给运行在用户态的调试器程序的，我们将在9.8节介绍调试API。

表9-1　Windows XP中支持用户态调试的内核服务

| 服 务 名 称 | 描　　述 |
| --- | --- |
| NtCreateDebugObject | 创建调试对象 |
| NtRemoveProcessDebug | 分离调试对话 |
| NtDebugActiveProcess | 与已经运行的进程建立调试会话 |
| NtSetInformationDebugObject | 设置调试对象的属性 |
| NtDebugContinue | 回复调试事件，恢复被调试进程 |
| NtWaitForDebugEvent | 等待调试事件 |
| NtQueryDebugFilterState | 查询调试信息输出的过滤级别 |
| NtSetDebugFilterState | 设置调试信息输出的过滤级别 |


### 9.4.8　全景

原书P180的图9-3画出了Windows XP改进后的用户态调试子系统的完整模型。图中虚线矩形框代表调试对象；双向链表是用来临时存储调试事件的消息队列，即DebugObject的`StateEventListEntry`字段；小旗帜代表调试对象中的EventsPresent事件对象。

原书P180的图9-3中左侧的表格代表的是IDT，左下方的异常分发过程将在第11章中详细讨论。图9-3中还列出了部分Dbgk例程的名字。表9-2归纳了支持用户态调试的内核函数。

表9-2　支持用户态调试的内核函数

| No | 名　　称 | 是否是Windows XP中引入的 | 描　　述 |
| --- | --- | --- | --- |
| 1 | nt!DbgkCreateThread | 否 | 采集线程创建事件 |
| 2 | nt!DbgkClearProcessDebugObject | 是 | 将调试对象从指定进程中分离 |
| 3 | nt!DbgkpConvertKernelToUserStateChange | 是 | 供NtWaitForDebugEvent使用，将DBGKM_DEBUG_EVENT结构转换为DBGUI_WAIT_STATE_CHANGE结构 |
| N/A | nt!DbgkDebugObjectType | 是 | 调试对象类型（type）的全局指针 |
| 4 | nt!DbgkpMarkProcessPeb | 是 | 当建立和解除调试会话时修改被调试进程中PEB的BeingDebugged字段 |
| 5 | nt!DbgkpSetProcessDebugObject | 是 | 当建立调试会话时将调试对象写入被调试进程（EPROCESS）的DebugPort字段 |
| 6 | nt!DbgkMapViewOfSection | 否 | 采集模块映射事件 |
| 7 | nt!DbgkExitProcess | 否 | 采集进程退出事件 |
| 8 | nt!DbgkOpenProcessDebugPort | 是 | 访问指定进程中DebugPort字段指定的调试对象 |
| 9 | nt!DbgkpWakeTarget | 是 | 设置ContinueEvent对象，唤醒等待调试器回复的线程 |
| 10 | nt!DbgkpQueueMessage | 是 | 向调试事件队列中加入消息 |
| 11 | nt!DbgkpResumeProcess | 否 | 恢复执行被调试进程 |
| 12 | nt!DbgkpOpenHandles | 是 | 打开进程、线程对象，增加应用计数 |
| N/A | nt!DbgkInitialize | 是 | 初始化调试对象，在系统启动早期被调用 |
| 13 | nt!DbgkpFreeDebugEvent | 是 | 释放调试事件 |
| 14 | nt!DbgkUnMapViewOfSection | 否 | 采集模块反映射事件 |
| 15 | nt!DbgkForwardException | 否 | 向调试子系统通报异常 |
| 16 | nt!DbgkpPostFakeProcessCreateMessages | 是 | 向调试子系统发送杜撰的进程创建消息 |
| 17 | nt!DbgkpPostFakeThreadMessages | 是 | 向调试子系统发送杜撰的线程创建消息 |
| 18 | nt!DbgkpSendApiMessageLpc | 是 | 主要用于向当前进程的异常端口（ExceptionPort字段）发送异常的第二轮处理机会 |
| 19 | nt!DbgkpCloseObject | 是 | 关闭调试对象，枚举系统内的所有进程，如果发现某个进程的DebugPort字段的值与要关闭的对象相同，则将其置为0 |
| 20 | nt!DbgkpPostFakeModuleMessages | 是 | 向调试子系统发送杜撰的模块消息 |
| 21 | nt!DbgkpDeleteObject | 是 | 目前没有使用 |
| 22 | nt!DbgkpSendApiMessage | 否 | 发送调试事件，在Windows XP中，调用DbgkpQueueMessage |
| 23 | nt!DbgkExitThread | 否 | 采集线程退出事件 |
| N/A | nt!DbgkpProcessDebugPortMutex | 是 | 全局的互斥量对象，用于保护对EPROCESS结构中DebugPort字段的访问 |
| 24 | nt!DbgkCopyProcessDebugPort | 是 | 当创建新的进程时，根据需要将父进程的DebugPort对象复制到新的进程中 |
| 25 | nt!DbgkpSectionToFileHandle | 否 | 取得Section对象对应的文件句柄 |
| 26 | nt!DbgkpSuspendProcess | 否 | 挂起被调试进程 |


表9-2中很多新引入的函数所实现的功能在Windows XP之前是在用户模式实现的。比如，`DbgkpPostFakeXXXMessages`函数所实现的投递杜撰消息的功能是在CSRSS进程中实现的。下一节将详细介绍Windows XP之前的调试子系统服务器。

## 9.5　调试子系统服务器（Windows XP之前）

本节将介绍Windows 2000和Windows NT中的调试子系统服务器，我们将其简称为Windows XP之前的调试子系统服务器。与9.4节介绍的Windows XP改进过的调试子系统服务器相比，Windows XP之前的调试子系统服务器有两个显著特征，分别是在用户态实现，使用LPC（Local Procedure Call）机制传递调试事件。

为了表达的简洁性，如不特别指出，本节下文中的Windows系统就是指Windows 2000或Windows NT（3.1～4.0）。

### 9.5.1　概览

原书P182的图9-4展示了Windows 2000/NT的用户态调试模型，其中展示了参与调试的所有成员，并简单地表示了这些角色之间的通信和协作关系，图中的圆柱代表的是LPC。

在详细介绍每个角色之前，我们先做一些简单介绍。

（1）调试子系统内核例程：调试子系统的内核部分，负责采集异常调试事件，以及控制（如挂起和恢复）被调试进程。这一部分与9.4节介绍的Windows XP开始的情况是一样的。

（2）会话管理器（Session Manager，SMSS.EXE）进程：如果把调试器看作请求调试服务的客户（client），那么SMSS.EXE便是提供服务的服务器。调试器通过LPC端口与SMSS.EXE通信，发送请求和接收调试事件。

（3）Windows环境子系统服务器进程（CSRSS.EXE）：尽管调试器是与SMSS.EXE直接通信的，但是SMSS.EXE通常并不真正处理请求，而是把请求转发给相应的子系统服务器进程。CSRSS便是Windows子系统的服务器进程。

在对参与调试的各个角色有了基本印象后，下面将对其分别做介绍。

### 9.5.2　Windows会话管理器

会话管理器（Session Manager，SMSS.EXE）是Windows系统启动后创建的第一个用户态进程。它负责启动和监护Windows环境子系统服务器进程（CSRSS.EXE）和WinLogon进程，对系统的正常运行起着重要的作用。SMSS.EXE在用户态调试中占据着核心地位（Windows XP之前），负责创建和维护调试子系统与调试器进行通信的LPC端口，是调试子系统的服务器（server）。

原书P183的图9-5所示的注册表表项定义了系统中的各个环境子系统，每次启动时，SMSS.EXE根据这里的定义决定加载哪些子系统。

```
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SessionManager\SubSystems
```

从图9-5中可以看到，除了Windows、Posix等定义子系统进程的表项，里面还有一个Required项。

Required项的类型是REG_MULTI_SZ，即可以包含多个字符串。通常这里的定义都是Debug Windows。毋庸置疑，Windows代表的是要SMSS启动Windows子系统。那么Debug项的含义是什么呢？答案是建立调试子系统，更准确地说是建立用户态调试子系统的服务器，包括LPC端口、链表结构和服务线程等，分别叙述如下。

（1）\DbgUiApiPort端口，这是调试子系统与调试器之间的通信通道。当调试器建立会话时，不论其使用的是`OpenPorcess(…, DEBUG_PROCESS, …)`还是`DebugActiveProcess()`API，其内部都会调用`DbgUiConnectToDbg()`函数（位于NTDLL.DLL中）。`DbgUiConnectToDbg()`函数的操作步骤之一，就是使用`NtConnectPort`函数与\DbgUiApiPort端口连接。

（2）\DbgSsApiPort端口，这是调试子系统与各个环境子系统进行通信的LPC端口。SMSS.EXE接收到来自该端口的事件并对其过滤后通过\DbgUiApiPort端口分发给等待的调试器进程。通常环境子系统服务器会连接该端口，以便把本系统的调试事件发给SMSS.EXE（调试子系统服务器进程）。因此可以说，\DbgSsApiPort端口是SMSS与环境子系统服务器进程之间的联系通道。

（3）\SmApiPort端口，该端口是SMSS.EXE对外提供服务的LPC通道。SMSS用该端口来接收系统内的其他进程发给它的各种服务请求（API）。在调试方面，当需要调试环境子系统进程（比如CSRSS）自身时，需要使用该端口（稍后继续讨论）。使用Process Explorer工具可以观察到SMSS进程中的各个LPC端口。

（4）监听\DbgUiApiPort端口的DbgUiThread的线程，DbgUiThread接收来自调试器的消息请求，如下所示。

    - 由于DbgUiConnectToDbg()而触发的连接请求。DbgUiConnectToDbg是调试器用来与调试子系统建立连接的关键函数，其内部主要通过调用NtConnectPort与\DbgUiApiPort端口建立连接。DbgUiThread收到连接请求后会创建一个用于通知调试状态变化的信号量，并分配一个数据结构，用于记录请求进程（即调试器进程）的ID和这个信号量。该信号量会被作为连接应答信息（ConnectionInfo）返回给DbgUiConnectToDbg函数，DbgUiConnectToDbg会将此信号量保存在线程环境块（TEB）结构中的DbgSsReserved[0]字段中，WaitForDebugEvent函数就是使用该信号量来等待调试事件的。DbgUiConnectToDbg将NtConnectPort函数返回的LPC句柄保存到TEB的DbgSsReserved[1]字段中。有一点需要说明的是，对于Windows 2000及其后的Windows系统，LPC端口就是可以等待的对象。那么为什么还要再使用一个信号量呢？这主要是为了与Windows NT 4.0兼容，因为在NT 4.0中，端口不是可以等待的内核对象。

    - 由于调试器进程调用WaitForDebugEvent()而触发的提取调试状态变化消息的请求（DbgUiStateChangeMsg）。DbgUiThread收到该请求后会将最近一次调试事件的详细信息返回给调试进程。WaitForDebugEvent() API内部会等待通过DbgUiConnectToDbg()获得的信号量（保存在线程环境块中的DbgSsReserved[0]）。在等到该信号后，便会发送DbgUiStateChangeMsg消息提取调试事件。

    - 由于调试器调用ContinueDebugEvent而触发的继续消息（DbgUiContinueMsg）。通常该消息中包含了调试器对最近一次调试事件的处理结果（response），DbgUiThread收到该消息后会通过 \DbgSsApiPort端口将应答转发给环境子系统进程。

（5）监听\DbgSsApiPort端口的DbgSsThread线程，主要接收来自环境子系统的连接请求和汇报调试事件的各种消息，包括异常、线程创建和退出、进程创建和退出、DLL映射（map）与反映射（unmap）等。

（6）用以维护已经注册的调试器进程和被调试进程的信息列表。该列表用于查找调试事件（被调试进程）和调试器进程间的对应关系，在分发调试事件和终止调试对话时起着重要作用。

### 9.5.3　Windows环境子系统服务器进程

Windows环境子系统服务器进程的映像文件名是CSRSS.EXE，因此常简称为CSRSS。尽管从Windows NT4开始，窗口管理（包括屏幕输出、用户输入和消息传递）与GDI的主体实现被移入内核模式的win32k.sys中，但CSRSS仍然是Windows子系统的灵魂，它监管系统内运行的所有Windows进程和线程，每个进程在创建后都要在这里注册、登记后方能运行，退出时也要到此报告、注销。除了掌管着各个进程的“生死存亡”，CSRSS在桌面管理、终端登录、控制台管理、HardError报告和DOS虚拟机等方面也起着重要作用。在调试方面（Windows XP之前），CSRSS也承担着很多重要职责，如下所示。

（1）创建和维护\Windows\ApiPort端口，该端口是CSRSS对外服务的“窗口”，系统的其他进程可以通过该端口向CSRSS发送服务请求。在调试方面，该端口是传递调试事件的重要通道。被调试进程的DebugPort字段所记录的通常就是这个端口（请参见下文）。

（2）调用`DbgSsInitialize()`函数（位于NTDLL.DLL中）向调试子系统服务器（SMSS）注册。DbgSsInitialize()会通过\DbgSsApiPort端口与调试子系统服务器建立连接，并将返回的句柄以全局变量的形式记录下来（名为`ntdll!DbgSspApiPort`）。接下来，`DbgSsInitialize()`会启动一个线程专门监听这个端口，用于接收SMSS发起的消息。一旦收到消息便将其发送到\Windows\ApiPort端口（为了使用方便，`DbgSsInitialize()`会将该端口的句柄赋给名为`ntdll!DbgSspKmReplyPort`的全局变量）。因为CSRSS与SMSS之间的通信大多是异步的，所以SMSS发起的消息通常是对前面发生的调试事件的应答，是由调试器通过调用`ContinueDebugEvent`发起的。

（3）启动线程用来监听\Windows\ApiPort端口，该线程的名字通常是`CsrApiRequestThread`（其实现位于CSRSRV.DLL中）。当`CsrApiRequestThread`接收到消息类型为调试事件的LPC消息（`LPC_DEBUG_EVENT`）后，它会调用`DbgSsHandleKmApiMsg()`函数（位于NTDLL.DLL中）。`DbgSsHandleKmApiMsg ()`函数会根据调试事件的类型将调试事件分发给具体的事件处理函数，如`DbgSspException`、`DbgSspCreateThread`等。这些函数会将调试事件格式化为`DBGSS_APIMSG`结构，并填写合适的枚举结构，然后将`DBGSS_APIMSG`结构通过\DbgSsApiPort端口转发给SMSS进程。全局变量`DbgSspApiPort`记录着CSRSS与\DbgSsApiPort端口的连接句柄。

（4）CSRSS的另一个调试支持就是它的仿真系统（emulation system），可模拟并发送过去发生的调试事件，即杜撰的调试事件，这对于把调试器附加到已经运行的进程很有用。

除了以上功能，CSRSS的子系统服务中还包含了专门用于调试的服务。在介绍这些服务前，让我们先来看一下每个Windows进程是如何调用它们的子系统服务的。

### 9.5.4　调用CSRSS的服务

CSRSS是Windows子系统的服务器进程，系统内的其他进程可以通过\Windows\ApiPort端口向其发送服务请求。事实上，每个Windows进程在启动阶段就已经做好了与CSRSS通信的准备。下面通过一个小实验来证明这一点。

启动WinDBG，然后打开一个可执行文件（选择File → Open Executable）或附加到某个已经运行的进程。

输入“`x ntdll!csr*`”，观察NTDLL.DLL中包含的以CSR开头的符号。

```
0:000> x ntdll!csr*
...
77fc4710 ntdll!CsrPortName = <no type information>
77fc46ac ntdll!CsrServerProcess = <no type information>
77fc46b4 ntdll!CsrServerApiRoutine = <no type information>
77fc46a4 ntdll!CsrProcessId = <no type information>
77fc46b0 ntdll!CsrPortHandle = <no type information>
77f5ec1e ntdll!CsrClientConnectToServer = <no type information>
77f5e9ca ntdll!CsrpConnectToServer = <no type information>
77f5ee8a ntdll!CsrClientCallServer = <no type information>
77fc46c0 ntdll!CsrInitOnceDone = <no type information>
...
```

注意上面的输出结果。其中`ntdll!CsrPortName`是个UNICODE字符串，使用`dS`（`S`要大写）可以显示其内容。

```
0:000> dS 77fc4710
00141ea0  "\Windows\ApiPort"
```

可见，`CsrPortName`变量记录的就是我们刚才介绍的CSRSS进程所公开的LPC端口名。

`ntdll!CsrPortHandle`中包含了当前进程与CSRSS的ApiPort连接所得到的句柄。

```
0:000> !handle 7ec
Handle 7ec
  Type            Port
```

`ntdll!CsrProcessId`变量包含了CSRSS进程的ID。

```
0:000> dd 77fc46a4 l1
77fc46a4  000004cc
```

将十六进制的`4cc`转换为十进制，然后打开Task Manager，可以发现转换结果与CSRSS进程的ID是一致的。

`ntdll!CsrInitOnceDone`变量用来保证与CSRSS建立连接的服务只运行一次，其值为1，表示初始化已经完毕。

变量`ntdll!CsrServerProcess`用来标记当前进程是否就是服务器（CSRSS）进程，因为CSRSS 进程本身也会加载和使用NTDLL.DLL。如果是，就直接将CSRSS的服务例程（位于CSRSRV.DLL中）的函数地址放入`ntdll!CsrServerApiRoutine`数组中，这样只需直接通过服务的索引找到这个数组中的函数指针，就可以调用服务了，省去了通过LPC通信的过程。如果调试CSRSS进程，我们可以看到`ntdll!CsrServerProcess`和`ntdll!CsrServerApiRoutine`变量为非零值，在普通的进程中，它们都是0。

```
0:000> dd ntdll!CsrServerProcess l1
77fc46ac  00000000
0:000> dd ntdll!CsrServerApiRoutine l1
77fc46b4  00000000
```

有了以上的准备工作，每个Windows进程就可以很方便地请求CSRSS的服务。为了进一步简化这一操作，NTDLL.DLL中包含了一个名为`CsrClientCallServer()`的未公开API。这个API封装了通过LPC端口发送请求和接收应答的细节，使客户进程只要通过调用该函数便可以“享受”CSRSS提供的服务，不用关心通信的细节。事实上，`CsrClientCallServer`内部向全局变量`ntdll!CsrPortName`记录的端口（\ApiPort端口）发送LPC消息，监听在这个端口的CSRSS的工作线程`CsrApiRequestThread`会收到这个消息，然后根据请求中的API编号，分发给真正的服务处理函数，最后再把应答发回给请求者。

```
NTSTATUS NTAPI  CsrClientCallServer(
   struct _CSR_API_MESSAGE *Request, 
   struct _CSR_CAPTURE_BUFFER *CaptureBuffer, 
   ULONG ApiNumber, 
   ULONG RequestLength)
```

真正完成各种CSRSS服务的各个函数主要位于BASESRV.DLL、CSRSRV.DLL和WINSRV.DLL这3个DLL模块中。通过Dependency Walker工具，可以观察到CSRSRV.DLL模块中所包含的函数信息。

### 9.5.5　CsrCreateProcess服务

CSRSRV模块中包含了很多与调试有关的函数，如`CsrCreateProcess`、`CsrDebugProcess`和`CsrDebugProcessStop`等。我们先来看一下`CsrCreateProcess`。

我们知道，大多数Windows进程是通过CreateProcess系列API创建的。在这个API成功调用`NtCreateProcess`或`NtCreateProcessEx`内核服务完成新进程的内核部分创建工作后，它会通过`CsrClientCallServer`向CSRSS请求CreateProcess服务。该服务的主要目的是向子系统服务器报告新进程的产生，这有点像新生儿的登记。CSRSS的`CsrApiRequestThread`线程收到CreateProcess请求后，会将该请求分发给`CsrCreateProcess`函数来处理。`CsrCreateProcess`函数所做的处理主要包括以下内容。

（1）为新进程分配一个`CSR_PROCESS`结构，用来登记新进程的各种信息。

（2）调用`NtSetInformationProcess`，将新进程的EPROCESS结构的ExceptionPort字段设置为\Windows\ApiPort端口对象。

（3）若进程的创建标志设置了调试标志（DEBUG_PROCESS），则调用`NtSetInformationProcess`将新进程的EPROCESS结构的`DebugPort`字段设置为\Windows\ApiPort端口对象。这里不必判断正在创建的进程是否为CSRSS进程，因为`CsrDebugPorcess`函数一定是在CSRSS进程创建后才工作的。

（4）将填写完整的`CSR_PROCESS`结构插入用来记录子系统内所有进程的一个全局链表。

### 9.5.6　CsrDebugProcess服务

除了`CsrCreateProcess`，另一个与调试密切相关的CSRSS服务就是`CsrDebugPorcess`。

当调试器使用`DebugActiveProcess`附加到一个已经运行的程序时，`DebugActiveProcess`函数便会通过`CsrClientCallServer`向CSRSS请求DebugProcess服务。请求的消息结构中包含了当前进程（即调试器进程）的ID和要调试进程的ID。CSRSS的`CsrApiRequestThread`线程收到请求后，便会将该请求分发给`CsrDebugProcess`函数来处理。`CsrDebugProcess`函数所做的处理主要包括以下内容。

（1）检查被调试进程是否是CSRSS本身。如果是，那么通过调用`RtlGetNtGlobalFlags`得到系统的全局标志。然后检查全局标志中是否设置了`FLG_ENABLE_CSRDEBUG`（0x20000）标志位，如果该位为0，那么不允许调试CSRSS，`CsrDebugProcess`会返回`STATUS_ACCESS_DENIED`（访问被拒绝）。可以使用GFlags工具（gflags /r +20000）或修改注册表表项HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Session Manager的GlobalFlag（REG_DWORD）值设置`FLG_ENABLE_CSRDEBUG`标志位。

（2）从包含子系统的所有进程的链表中找到被调试进程所对应的节点。CSRSS为每个进程分配并维护一个CSR_PROCESS结构，并以链表的形式存储起来。找到被调试进程的CSR_PROCESS结构后，`CsrDebugProcess`将调试器进程（也就是发起请求者）的ID、调试器进程发起调试的线程ID，以及该进程正在被调试等属性记录在CSR_PROCESS结构中。

（3）挂起被调试进程。

（4）如果被调试进程不是CSRSS自己，调用`NtSetInformationProcess`将被调试进程的EPROCESS结构的`DebugPort`字段设置为连接到\Windows\ApiPort端口的LPC句柄。

（5）调用`DbgSsHandleKmApiMsg()`函数，依次向调试器发送杜撰的进程创建、线程创建和DLL加载事件。因为要附加到已经运行的进程，所以这些事件事实上是以前发生的，现在CSRSS根据记录重新“播放”给调试器，这一功能又称为CSRSS的调试事件“仿真”功能。

（6）唤醒被调试进程。

（7）如果被调试进程是CSRSS本身，调用`NtSetInformationProcess`将其EPROCESS结构的`DebugPort`字段设置为连接到\SmApiPort端口的句柄。之所以调试CSRSS进程时要将DebugPort设置为\SmApiPort端口，是因为\SmApiPort端口是由SMSS进程创建并维护的，而\Windows\ApiPort端口是由CSRSS进程维护的。如果调试CSRSS进程时仍使用\Windows\ApiPort端口，那么CSRSS被中断到调试器后，就“无人”监听和维护该端口了，这样会导致死锁。

上面简要介绍了Windows 2000和Windows NT的调试子系统服务器，包括组成部分和每一部分所承担的功能。下面简要介绍这些部件的协作方法。调试器通过`DbgUiConnectToDbg`函数连接到位于SMSS的调试子系统服务器，也就是\DbgUiApiPort端口。如果调试新创建的进程，那么CSRSS的创建进程服务（CsrCreateProcess）会将新创建进程的DebugPort设置为连接到CSRSS的LPC端口（\Windows\ApiPort）。如果调试已经运行的程序，那么CSRSS的调试服务（CsrDebugProcess）会将\Windows\ApiPort端口设置为要调试进程的DebugPort字段。

当内核中的`DbgkForwardException`收集到异常事件时，它会调用`DbgkpSendApiMessage`，后者再调用`LpcRequestWaitReplyPort`将消息发到DebugPort所标识的LPC端口，即 \Windows\ApiPort。这样，CSRSS.EXE便收到了调试消息，它会将其转发给\DbgSsApiPort。守候\DbgSsApiPort端口的会话管理器进程（SMSS.EXE）收到调试消息后会根据LPC消息的AppClientId检查是否有调试器进程与其匹配。如果有，便释放与这个被调试进程相关的信号量`NtReleaseSemaphore`（`StateChangeSemaphore`），通知调试器有事件发生。与Windows XP中一样，调试器建立调试对话后便使用`WaitForDebugEvent`API等待调试事件，不过这个API在Windows XP之前的实现是在用户模式调用`NtWaitForSingleObject`服务等待`DbgStateChangeSemaphore`信号量。调试器得到信号量后，便通过`NtRequestWaitReplyPort(…, DbgUiWaitStateChangeApi, …)`函数向\DbgUiApiPort端口读取消息。会话管理器发现来自\DbgUiApiPort的`DbgUiWaitStateChangeApi`请求后，便会将属于该调试器的消息发送给调试器。

## 9.6　比较两种模型

前面两节分别介绍了Windows XP之后（包括Windows XP）和之前的用户态调试模型，二者的主要差异是调试子系统服务器的位置。本节对这两种模型做一个简单的总结和比较。

### 9.6.1　Windows 2000调试子系统的优点

总体来说，Windows 2000调试子系统的设计是非常优秀的，主要表现在以下几个方面。

（1）架构非常清晰。整个子系统以调试事件的产生、传递、分发和处理为线索，清楚地划分为职责明确的几大模块——内核例程、NTDLL.DLL中的支持例程和调试子系统服务器。

（2）接口非常简单。尽管整个调试系统包含了很多个进程和模块，每个模块内部的功能都比较复杂，但是部件之间的接口非常简洁。调试器只需要与调试子系统服务器通信；内核例程只需要向被调试进程的DebugPort发送信息，根本不关心这个端口到底是谁创建的，谁在监听。这些精湛的低耦合设计带来了极强的灵活性和可扩展性，可以方便地扩充和修改子系统内某一部分的设计，同时保证其他部分能照常工作。举例来说，Windows XP对调试子系统的服务器部分做了较大改动，但是针对Windows 2000或Windows NT设计的调试器依然可以一如既往地工作，旧的应用程序也仍然可以被调试。

（3）代码复用方面做得非常好，把需要共享的调试支持例程（DbgUi系列、DbgSs系列和Csr系列）放到NTDLL.DLL中，因为NTDLL.DLL是Windows内包括环境子系统服务器在内的所有程序都加载和使用的模块，这样便可以让这些代码得到最有效的复用。比如，DbgSsInitialize是用来完成与调试子系统服务器（SMSS）建立连接等初始化工作的，现在看来这部分代码只有CSRSS需要，没有必要放在NTDLL.DLL中。但是我们要知道，Windows 2000和Windows NT设计时都曾考虑到要支持多个环境子系统，如果把DbgSsInitialize放到CSRSS模块中，那么POSIX子系统服务器就要重复编写这部分代码了。

### 9.6.2　Windows 2000调试子系统的安全问题

金无足赤，Windows 2000调试子系统模型（事实上Windows NT 4就使用这样的模型）在应用了多年之后，2002年3月有人（EliCZ）在互联网上公开了它的一个安全漏洞，取名为DebPloit（Deb代表调试）。攻击这个漏洞的基本步骤如下。

（1）使用`DbgUiConnectToDbg`与调试子系统服务器的UI端口建立连接，这是调试器通常采取的动作。

（2）通过ZwConnectPort与调试子系统服务器的LPC端口（\DbgSsApiPort）建立连接，这是环境子系统服务器程序（如CSRSS）通常采取的动作。主要的漏洞出现在这里—— \DbgSsApiPort端口在创建时所设定的安全权限使所有权限等级的进程都可以连接。设计者之所以这么做，应该是为了照顾POSIX和OS/2等子系统服务器进程，使它们在非管理员账号下也能成功连接调试服务器。

（3）成功完成以上两步后，对于调试服务器（SMSS）而言，攻击程序便既有了调试器身份，又有了环境子系统身份。于是它便可利用环境子系统身份来模拟调试事件，欺骗SMSS为其服务，然后再利用调试器身份接收服务结果。接下来，攻击程序模拟一个调试器附加到被攻击进程的调试事件，将要攻击的其他系统进程的ID（或线程ID）和假调试器（攻击程序自身）的进程ID放到消息结构中，发给SMSS，目的是让调试子系统为假调试器与被攻击进程建立调试关系。

（4）一旦调试关系建立，攻击程序便可以控制和操作被攻击进程了。最简单的一种攻击就是直接退出“调试器”，因为调试器退出会导致被调试进程也退出，这样攻击程序的退出便可能使重要的系统进程随着终止。重要系统进程（如WinLogon和Lsass等）的终止会导致系统崩溃。

修正以上漏洞的方法之一是拒绝没有管理员权限的进程连接 \DbgSsApiPort端口，也就是只要在创建该端口时将安全描述符设置为NULL（使用SMSS默认的管理员权限，使低于该权限的进程无法访问）。

2002年5月22日，微软公布了关于这个漏洞的安全公告（security bulletin），代号为MS02-024，同时提供了修正这个问题的HotFix，此后的Windows 2000 Service Pack（SP3或更高版本）都包含了这个修正。Windows XP操作系统将调试子系统服务器移入内核模式中，不再使用 \DbgSsApiPort 端口，因此不再存在这个漏洞。

### 9.6.3　Windows XP的调试模型的优点

Windows XP的调试模型以调试对象（DebugObejct）为核心，去除了原本利用LPC多级通信的相对复杂的通信模型。以前，调试消息要通过CSRSS的ApiPort、SMSS的DbgSsApiPort和DbgUiApiPort传递，显得有些累赘。新的模型利用调试对象中的事件链表直接通信，使通信过程大大简化。

从进程角度来看，Windows XP的调试模型更加简洁，在调试事件的驱动下，调试器进程和被调试进程以调试对象为纽带相互通信。

从功能上看，Windows XP的调试模型的改变为增加下面将介绍的新调试功能创造了便利条件。

### 9.6.4　Windows XP引入的新调试功能

因为使用了DebugObject内核对象和新的调试模型（把调试子系统服务器移入内核），Windows XP的调试子系统支持跨Windows登录会话（Windows session）进行调试。举例来说，用户A可以登录系统并调试用户B运行的程序。登录的方式可以是通过终端服务（terminal service）或使用Windows XP的快速用户切换（fast user switch）功能。跨Windows登录会话调试对于Windows 2000的调试模型来说是做不到的，因为不同的登录会话（session）会启动不同的CSRSS进程。这意味着用户A的调试器进程与用户B的进程分别属于不同的Windows子系统，二者无法建立调试对话。

分离调试会话并保持被调试进程继续运行是Windows XP之后的调试子系统的一个新功能。此前结束调试会话时，被调试进程随之终止，不过，这一功能在以前的模型中是可以实现的。

因为调试API（`DebugAtiveProcess、WaitForDebugEvent`）的功能和函数原型没有任何改变，所以以上两种模型在API层是兼容的。也就是说，旧的调试器无须做任何修改仍能非常好地工作在新的系统中，感觉不到调试模型的变化。

从数据结构角度来看，新旧模型保持了很好的兼容性。比如，在调试器一侧，仍然使用调试器线程TEB的`DbgSsReserved[2]`数组，来记录用来与调试子系统通信的重要信息。以前`DbgSsReserved[1]`记录的是LPC端口句柄（DbgUiApiPort），`DbgSsReserved[0]`记录的是等待调试消息的事件对象。现在`DbgSsReserved[1]`记录的是调试对象（DebugObject）句柄，`DbgSsReserved[0]`改为他用（10.1.3节），因为只要直接等待调试对象就可以了。

## 9.7　NTDLL.DLL中的调试支持例程

3.6节简要介绍过NTDLL.DLL（简称NTDLL），它是Windows系统中一个很特别的模块，不仅所有的Windows应用程序都与它有依赖关系，而且系统的进程也要使用它。NTDLL.DLL的重要性首先体现在它所包含的用于访问系统服务的残根（stub）函数上。例如NTDLL.DLL中的`NtCreateFile`函数是内核中真正的`NtCreateFile`函数的残根函数，这些残根函数是用户模式的应用程序访问内核服务的唯一正式方法。此外，NTDLL.DLL的重要性还体现在它内部包含了很多关键的支持函数，比如9.5节讲到的`CsrClientCallServer`是调用Windows子系统服务的重要函数。

在调试方面，NTDLL.DLL中包含了很多非常重要的函数，可以把这些函数分为如下3类——DbgUi函数、DbgSs函数和Dbg函数，下面分别进行介绍。

### 9.7.1　DbgUi函数

为了让调试器程序可以方便地使用调试子系统所定义的功能，NTDLL.DLL中设计了一系列函数，它们都是以DbgUi开头的，我们将其称为DbgUi函数。在Windows 2000的NTDLL.DLL中，包含了3个DbgUi函数，分别是用于和调试子系统建立连接的`DbgUiConnectToDbg`，用于等待调试事件的`DbgUiWaitStateChange`，以及用于继续调试事件的`DbgUiContinue`函数。Windows XP进一步丰富了DbgUi函数，其数量从原来的3增加到10。表9-3列出了所有这些函数，包括Windows 2000中已经存在的，第二列说明该函数是否是Windows XP版本新引入的。

表9-3　NTDLL.DLL中的DbgUi函数

| 函　数　名 | 是否是Windows XP中新引入的 | 说　 明 |
| --- | --- | --- |
| DbgUiDebugActiveProcess | 是 | DebugActiveProcess API的实现，KERNEL32中在将进程ID转为句柄后调用DbgUiDebugActiveProcess，后者再调用内核服务NtDebugActiveProcess。Windows 2000是直接在KERNEL32.DLL中实现这个API的 |
| DbgUiConnectToDbg | 否 | 连接调试子系统，Windows XP中主要调用ZwCreateDebugObject |
| DbgUiConvertStateChangeStructure | 是 | 将DBGUI_WAIT_STATE_CHANGE结构转换为调试器所需要的DEBUG_EVENT结构 |
| DbgUiGetThreadDebugObject | 是 | 从调试器工作线程TEB中（偏移0xf24处）读取调试对象 |
| DbgUiSetThreadDebugObject | 是 | 将调试对象记录到TEB中（偏移0xf24处）。建立调试对话时，先调用NtCreateDebugObject创建调试对象，然后再记录到TEB中 |
| DbgUiIssueRemoteBreakin | 是 | 在被调试进程中创建远程线程以使其中断到调试器，是DebugBreakProcess API（KERNEL32.DLL输出）的实现 |
| DbgUiContinue | 否 | 恢复被调试进程，Windows XP中调用系统服务NtDebug Continue，Windows 2000中通过LPC回复消息给调试子系统 |
| DbgUiWaitStateChange | 否 | 等待调试事件，Windows XP中调用系统服务NtWaitForDebugEvent，后者再等待保存在TEB的DbgSsReserved[0]中的DebugObject对象，Windows 2000就在该函数中使用NtWaitForSingleObject等待保存在DbgSsReserved[0]中的信号量，等待成功后，通过NtRequestWaitReplyPort()读取调试事件 |
| DbgUiStopDebugging | 是 | 停止调试，调用系统服务NtRemoveProcessDebug |


在Windows 2000中，`DbgUiConnectToDbg`主要调用`NtConnectPort()`与会话管理器（SMSS.EXE）的\DbgUiApiPort连接。返回的端口句柄和用于等待端口数据的信号量（semaphore）被保存在调用线程的线程环境块中（Teb()->DbgSsReserved[1]和Teb()->DbgSsReserved[0]）。调试器等待调试事件（执行DbgUiWaitStateChange）实际就是等待保存在Teb中的信号量。

从软件架构的角度来讲，DbgUi函数是调试子系统向外（调试器）提供的接口。

### 9.7.2　DbgSs函数

在Windows XP以前，调试子系统服务其实是在会话管理器（SMSS）中实现的，为了让各个环境子系统可以方便地与调试子系统建立联系，并支持调试功能，NTDLL.DLL实现了一系列以DbgSs开头的函数，我们将其称为DbgSs函数。例如`DbgSsInitialize()`函数是供环境子系统初始化调试支持的，包括连接调试子系统服务器的\DbgSsApiPort端口和注册回复端口（KmReplyPort）。在WinDBG中使用符号搜索命令可以很容易列出这些函数（清单9-6）。

清单9-6　NTDLL中包含的供环境子系统使用的调试支持函数（Windows 2000）

```
0:000> x ntdll!DbgSs*
77f9ae47 ntdll!DbgSsInitialize = <no type information>        [初始化]
77fcf2a4 ntdll!DbgSspKmApiMsgFilter = <no type information>
77f9b04f ntdll!DbgSspSrvApiLoop = <no type information>
77f9ae8d ntdll!DbgSsHandleKmApiMsg = <no type information>    [见正文]
77f9adad ntdll!DbgSspLoadDll = <no type information>          [报告模块映射消息]
77f9acba ntdll!DbgSspCreateProcess = <no type information>    [报告进程创建消息]
77f9ac1c ntdll!DbgSspException = <no type information>        [报告异常消息]
77f9adfd ntdll!DbgSspUnloadDll = <no type information>        [报告反映射消息]
77fcd144 ntdll!DbgSspUiLookUpRoutine = <no type information>  [函数指针]
77f9ad63 ntdll!DbgSspExitProcess = <no type information>      [报告进程退出消息]
77f9ac6a ntdll!DbgSspCreateThread = <no type information>     [报告线程创建消息]
77f9abde ntdll!DbgSspConnectToDbg = <no type information>   
77f9ad19 ntdll!DbgSspExitThread = <no type information>       [报告线程退出消息]
77fcd1f0 ntdll!DbgSspApiPort = <no type information>[保存连接SMSS端口的全局变量]
77fcd158 ntdll!DbgSspKmReplyPort = <no type information>      [全局变量]
77fcf2a8 ntdll!DbgSspSubsystemKeyLookupRoutine = <no type information>
```

其中，`DbgSsHandleKmApiMsg()用于`供环境子系统处理（包装并发送给调试服务器）收到的调试消息（LPC_DEBUG_EVENT）。以DbgSsp开头的符号是调试子系统的内部函数或全局变量。

Windows XP将调试子系统移到内核中，因此其NTDLL.DLL中不再存在以上DbgSs函数。

### 9.7.3　Dbg函数

除了以上两类函数，NTDLL.DLL中还有一系列以Dbg开头（非DbgUi或DbgSs）的调试支持函数，我们将其称为Dbg函数。例如用于触发断点事件的DbgBreakPoint，它是调试API DebugBreak的实现，事实上，在x86平台中这个函数的内部就是一条INT 3指令。

```
0:001> uf ntdll!DbgBreakPoint
ntdll!DbgBreakPoint:
7c901230 cc              int     3
7c901231 c3              ret
```

此外，还有DbgUserBreakPoint（断点指令）、DbgPrint（打印调试信息）、DbgPrompt（提示输入）、DbgPrintReturnControlC等。Windows XP又引入了DbgBreakPointWithStatus、DbgSetDebugFilterState（设置调试信息输出的过滤级别，内部调用NtSetDebugFilterState内核服务）、DbgQueryDebugFilterState和DbgPrintEx。

## 9.8　调试API

Windows SDK中公开了一系列API供调试器与调试子系统交互以实现各种调试功能。大多数SDK中公开的调试API是从KERNEL32.DLL中导出的。其中有些就是在KERNEL32.DLL中实现的，有些是用来调用9.7节介绍的NTDLL.DLL中的调试支持函数的。表9-4列出了目前SDK中已经文档化的调试API。

表9-4　SDK中已经文档化的调试API

| API | 版本 | 描　　述 | 实　　现 |
| --- | --- | --- | --- |
| BOOL CheckRemoteDebuggerPresent (HANDLE hProcess, PBOOL pbDebuggerPresent) | XP SP1 | 判断指定的进程是否处于被调试状态 | 调用NtQueryInformationProcess查询进程环境块（PEB） |
| BOOL ContinueDebugEvent (DWORD dwProcessId, DWORD dwThreadId, DWORD dwContinueStatus) | 9x，NT | 供调试器恢复被调试进程运行，回复调试事件 | 调用NTDLL.DLL中的DbgUiContinue() |
| BOOL DebugActiveProcess (DWORD dwProcessId) | 9x，NT | 供调试器附加到已经运行的进程 | 调用NTDLL.DLL中的DbgUiDebugActiveProcess |
| BOOL DebugActiveProcessStop (DWORD dwProcessId) | XP | 分离调试会话 | 将进程ID转换为句柄后调用NTDLL.DLL中的DbgUiStopDebugging |
| void DebugBreak(void) | 9x，NT | 在当前进程中产生断点异常 | 调用NTDLL.DLL中的DbgBreakpoint |
| BOOL DebugBreakProcess (HANDLE Process) | XP | 在指定进程中产生断点异常 | 调用NTDLL.DLL中的DbgUiIssueRemoteBreakin |
| BOOL DebugSetProcessKillOnExit (BOOL KillOnExit) | XP | 指定调试器线程退出时是否终止被调试进程 | 使用DbgUiGetThreadDebugObject和NtSetInformationDebugObject实现 |
| void FatalExit (int ExitCode) | 9x，NT | 16位Windows系统遗留下来的API。最初供调试应用程序强制中断到调试器 | 目前Windows 2000、Windows XP实现的只是调用ExitProcess |
| BOOL FlushInstructionCache (HANDLE hProcess, LPCVOID lpBaseAddress, SIZE_T dwSize) | 9x，NT | 当调试器修改代码段时，可以使用此API冲转（flush）缓存 | 调用NtFlushInstructionCache |
| BOOL GetThreadContext (HANDLE hThread, LPCONTEXT lpContext) | 9x，NT | 取得指定线程的上下文（CONTEXT）结构 | 调用NtGetContextThread |
| BOOL GetThreadSelectorEntry (HANDLE hThread, DWORD dwSelector, LPLDT_ENTRY lpSelectorEntry) | 9x，NT | 从指定线程的局部描述符表（LDT）中取得指定选择子所对应的表项Entry | 调用NtQueryInformationThread |
| BOOL IsDebuggerPresent(void) | 9x，NT | 判断调用进程是否在被调试 | 检查PEB的BeingDebugged字段 |
| void OutputDebugString (LPCTSTR lpOutputString) | 9x，NT | 供应用程序输出调试信息。当被调试时，这些信息会显示到调试器。参见后文 | 通过产生异常实现：RaiseException（DBG_PRINTE- XCEPTION_C,0,2,ExceptionArgu-ments），详见10.7节 |
| BOOL ReadProcessMemory (HANDLE hProcess, LPCVOID lpBaseAddress, LPVOID lpBuffer, SIZE_T nSize, SIZE_T* lpNumberOfBytesRead) | 9x，NT | 读取指定进程空间中的指定内存区域 | 调用NtReadVirtualMemory |
| BOOL SetThreadContext (HANDLE hThread, const CONTEXT* lpContext) | 9x，NT | 设置指定线程的CONTEXT信息 | 调用NtSetContextThread |
| BOOL WaitForDebugEvent (LPDEBUG_EVENT lpDebugEvent, DWORD dwMilliseconds) | 9x，NT | 供调试器的工作线程等待调试事件 | 调用NTDLL.DLL中的DbgUiWaitStateChange |
| BOOL WriteProcessMemory (HANDLE hProcess, LPVOID lpBaseAddress, LPCVOID lpBuffer, SIZE_T nSize, SIZE_T* lpNumberOfBytesWritten); | 9x，NT | 向指定进程空间中的指定内存区域写入数据 | 调用NtWriteVirtualMemory |


表9-4中，第2列给出的是支持该API的Windows最低版本，最后一列描述该API的主要实现方法。大家在使用这些API前，应该进一步查阅SDK文档以了解其详细的用法。

## 9.9　本章总结

本章比较详细地介绍了Windows操作系统用户态调试模型及用于实现这一模型的各个模块和函数。9.1节是概要介绍，9.2节和9.3节分别介绍了调试消息的采集和发送过程，而后介绍了调试模型的核心部分——调试子系统服务器（9.4节至9.6节）。9.7节介绍了NTDLL.DLL中的调试支持例程，9.8节简要介绍了调试API。

总的来说，本章介绍了Windows系统中用于支持用户态调试的基础设施，第10章将进一步介绍这些设施是如何协同配合完成各种调试功能的。如果说本章是为Windows系统的调试设施拍一幅静态的照片，那么在第10章中，我们将让这些设施动起来。
