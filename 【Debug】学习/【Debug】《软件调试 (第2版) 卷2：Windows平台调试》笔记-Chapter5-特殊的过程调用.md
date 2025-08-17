# 第二篇 探　　微

本书前两篇的目标是帮助读者快速认识Windows系统。如果说上一篇的目标是尝试“致广大”，那么本篇的目标便是“尽精微”了。

第5章探索Windows系统里几个独特的函数调用机制——APC、DPC、LPC和RPC。选择这个内容的原因是多方面的，一方面，它们是Windows系统里常用的机制，调试时经常遇到，而且常常成为拦路虎，追踪到这里就难以推进了。另一方面，理解它们有较大的难度，已有的资料大多比较抽象晦涩。我们使用不同的方法，通过具体实例，上调试器，让抽象的概念具体化。

第6章探索的是Windows系统中神秘的垫片（shim）机制。自从Windows XP引入这个机制后，它便成为Windows系统中解决各种软件兼容问题的核心机制。今天，随着向64位Windows系统过渡和Windows on ARM（WoA）的发展，垫片机制的应用范围更加广泛。很多时候，会看到垫片模块的身影。垫片模块会改变软件的原本执行路径，让本来已经充满不确定性的软件世界变得更加飘忽不定。

不管你是否喜欢.NET，它都已经成为Windows系统的一部分。既然无法回避，就面对它。因此，本书第7章让你快速了解托管世界的核心技术和关键细节。

Windows 10引入的WSL（Windows Subsystem for Linux）技术让Linux应用程序可以运行在NT内核之上。这代表着Windows和Linux两大软件平台从对立走向合作，从各行其道到交叉和融合。WSL给经典的NT内核添丁增户，让它在新时代焕发活力。第8章探索WSL的架构、组件和关键细节。

# 第5章　特殊的过程调用

在软件世界里，函数、子函数、过程和子过程是很常用的几个术语。在某些语境中，它们有细微的区别，但是大多数情况下，它们的意思是相同的，代表相对独立、可以完成某个功能的代码。本章等同对待它们，不作区分。在x86这样的现代CPU中，CPU内部就设计了用于调用函数的CALL指令，这将在第22章详细介绍。简单来说，CALL指令所执行的操作就是先把下一条指令的地址压入当前线程的栈，为函数返回做好准备，然后便跳转到操作数所指向的子函数。子函数的末尾一般是一条RET指令，它要执行的主要动作便是从栈中弹出返回地址，然后转移到这个地址继续执行。

从线程的角度来看，CPU所提供的CALL和RET指令可以完成同一线程在同一特权级内的普通函数调用。因为CALL和RET指令必须使用相同的栈，所以对于不同线程之间或者同一线程里跨越特权级别的调用（比如用户空间和内核空间之间代码的调用），直接使用CALL和RET指令就不行了。

我们把跨越空间、跨越特权级别、跨越线程、跨越进程或者跨越机器的过程调用称为特殊的过程调用（Procedure Call）。为了支持特殊的过程调用，Windows操作系统设计了丰富的系统设施，这些设施中，有些是供操作系统自己使用的，有些是供Windows平台的开发者使用的。这些设施的名字很类似，分别叫异步过程调用（APC）、延迟过程调用（DPC）、本地过程调用（LPC）和远程过程调用（RPC），看起来就像同门兄弟。这些设施有很多共同点，比如，都用于支持跨越某种边界的特殊调用；都在Windows系统中享有重要的地位，离开任意一个过程调用，系统都无法运行；都有些难度，不太好理解。

## 5.1　异步过程调用

普通的函数（过程）调用是阻塞的，直到子函数执行完毕和返回后，父函数才能继续执行。如果子函数所用的时间很久（比如调用ReadFile函数读串口，但是对方暂时没有发送数据），那么这次函数调用便会导致线程阻塞在这个函数内部。对于典型的Windows GUI程序，如果这样的阻塞发生在UI线程中，那么便会导致界面无法更新，失去响应。为了避免类似这样的问题，Windows系统的很多API支持以异步方式工作，它们所依赖的便是NT内核的异步过程调用（Asynchronous Procedure Call，APC）机制。

以ReadFile API为例，最后一个参数就是用于支持异步方式的。

```
BOOL ReadFile(
  HANDLE       hFile,
  LPVOID       lpBuffer,
  DWORD        nNumberOfBytesToRead,
  LPDWORD      lpNumberOfBytesRead,
  LPOVERLAPPED lpOverlapped
);
```

简单来说，如果打开文件时指定了`FILE_FLAG_OVERLAPPED`标志，而且在调用ReadFile时传递了一个OVERLAPPED结构，那么当要读的数据还没有就绪时（比如串口或者管道通信时，发送方还没有发送），ReadFile便会立刻返回，等数据就绪时，系统会设置OVERLAPPED结构中指定的事件对象（hEvent）。

为了让编程更方便，Windows XP引入了增强版本的ReadFile函数，名为ReadFileEx，增加了一个参数，可以指定一个完成函数（complete routine），供系统回调。

```
BOOL ReadFileEx(
  HANDLE                          hFile,
  LPVOID                          lpBuffer,
  DWORD                           nNumberOfBytesToRead,
  LPOVERLAPPED                    lpOverlapped,
  LPOVERLAPPED_COMPLETION_ROUTINE lpCompletionRoutine
);
```

最后一个参数lpCompletionRoutine便是用于指定回调函数的。

GeAPC小程序是笔者专门为了调试异步文件I/O工作过程而开发的靶子程序。

当单击Write Long按钮时，清单5-1中的WriteLong方法会执行，其内部会以异步方式调用WriteFileEx。因为指定了异步方式，WriteFileEx会立刻返回TRUE，代表操作成功。

当单击界面上的Alertable按钮时，GeAPC会执行如下语句，调用SleepEx函数，让当前线程进入所谓的可接警状态。

```
SleepEx(1, TRUE);
```

此时，清单5-1中的回调函数会被调用，向列表框中打印出信息。

清单5-1　GeAPC程序的核心代码

```
VOID WINAPI GeCompletedWriteRoutine(DWORD dwErr, DWORD cbWritten,
    LPOVERLAPPED lpOverLap)
{
    CGeAPCDlg * pDlg = (CGeAPCDlg *)((LPGEOVERLAP)lpOverLap)->pUserData;
    BOOL fRead = FALSE;

    pDlg->D4D(_T("Complete Routine is called with %d bytes written."), cbWritten);
}

HRESULT CGeAPCDlg::WriteLong(LPCTSTR szFileName, int nBytes)
{
    if(m_hFile == INVALID_HANDLE_VALUE)
    { 
        m_hFile = CreateFile(szFileName, GENERIC_WRITE, 0, NULL, CREATE_ALWAYS,
            FILE_ATTRIBUTE_NORMAL, NULL);
        if (m_hFile == INVALID_HANDLE_VALUE)
        {
            D4D(_T("Failed to create file %s, gle %d"), szFileName, GetLastError());
            return E_FAIL;
        }
        m_lpBuffer = (LPBYTE)malloc(nBytes);
        memset(&m_Overlapped, 0, sizeof(m_Overlapped));
        m_Overlapped.pUserData = this;
    }
    else
        m_Overlapped.oOverlap.Offset += nBytes;

    BOOL bWrite = WriteFileEx(
        m_hFile,
        m_lpBuffer,
        nBytes,
        (LPOVERLAPPED)&m_Overlapped,
        (LPOVERLAPPED_COMPLETION_ROUTINE)GeCompletedWriteRoutine);
    D4D(_T("Writing %d bytes returned %d, gle=%d"), nBytes, bWrite, GetLastError());

    return bWrite ? S_OK : E_FAIL;
    }
```

那么系统到底是如何调用这个回调函数的呢？

首先，当I/O管理器完成I/O请求时，如果发现这个请求是异步的，那么便会创建一个APC对象，并调用内核函数KeInsertQueueApc将其放入内核队列中。

回到例子，当NTFS完成GeAPC小程序提交的写操作时，会调用I/O管理器的IofComplete Request函数报告完成I/O请求包（IRP）。当I/O管理器的内部函数IopCompleteRequest检查到与对应的IRP操作有关联的用户态完成函数时，便会创建APC，其过程如清单5-2所示。

清单5-2　I/O管理器完成I/O动作后向队列中插入APC对象的过程

```
# Call Site
00 nt!KiInsertQueueApc
01 nt!KeInsertQueueApc
02 nt!IopCompleteRequest
03 nt!IopfCompleteRequest
04 nt!IofCompleteRequest
05 NTFS!NtfsExtendedCompleteRequestInternal
06 NTFS!NtfsCommonWrite
07 NTFS!NtfsFsdWrite
```

创建的APC被放入APC队列中。在内核调试会话中，可以使用!apc命令来观察APC队列。!apc默认会显示系统中所有进程的所有APC，可以使用!apc proc <进程的EPROCESS地址>的方式来只显示指定进程的APC队列。比如，下面是观察GeAPC进程的结果。

```
4: kd> !apc proc ffffc10f`8fe5a080
Process ffffc10f8fe5a080 GeAPC.exe
    Thread ffffc10f82020080 ApcStateIndex 0 ApcListHead ffffc10f82020128 [USER]
        KAPC @ ffffc10f8fa82b38
          Type           12
          KernelRoutine  fffff80278596840 nt!IopUserCompletion+0
          RundownRoutine fffff80278596840 nt!IopUserCompletion+0
```

可以看到，GeAPC进程的ffffc10f82020080线程（即0号线程，唯一的线程）有一个APC等待执行。使用!thread ffffc10f82020080观察这个线程的详情，可以看到线程的状态如下。

```
WAIT: (WrUserRequest) UserMode Non-Alertable
```

这意味着，虽然此时线程进入内核空间且处于等待状态，但是它处于不可接警（NonAlertable）状态，因为GeAPC的0号线程执行Write Long动作后又进入消息循环，调用NtUserGetMessage等待窗口消息。

按照NT系统的规则，只有线程处于可接警状态时才可以向其投递（deliver）APC。目前0号线程处于不可接警状态，所以虽然它有APC，但是也不能向其投递APC。

那么如何让线程进入可接警状态呢？微软的文档提供了多种方法。一种简单的方法是调用SleepEx，调用时把第二个参数指定为TRUE。出于这个原因，我们特意为GeAPC小程序设计了一个Alertable按钮，单击这个按钮后，GeAPC会调用SleepEx让线程进入可接警状态。进一步来说，SleepEx会调用内核服务NtDelayExecution，当这个服务返回时，内核中的系统服务返回函数KiSystemServiceExit会检查是否需要投递APC。如果需要而且当前线程处于可接警状态，那么便会调用KiDeliverApc来投递APC，其过程如清单5-3所示。

清单5-3　投递APC

```
# Call Site
00 nt!KiDeliverApc
01 nt!KiInitiateUserApc
02 nt!KiSystemServiceExit
03 ntdll!NtDelayExecution
04 KERNELBASE!SleepEx
05 mfc140u
```

投递APC的技术细节过于冗长，简单来说，内核空间会保存线程的当前状态，然后修改内存中的寄存器上下文，让程序指针指向事先准备好的APC分发函数。做好这些准备工作后，当系统服务返回时，CPU便开始执行APC分发函数了，其调用过程如清单5-4所示。

清单5-4　APC的调用过程

```
# Call Site
00 GeAPC!GeCompletedWriteRoutine
01 ntdll!KiUserApcDispatch
02 ntdll!NtDelayExecution
03 KERNELBASE!SleepEx
04 mfc140u
```

位于NTDLL.DLL中的KiUserApcDispatch负责在用户空间中分发APC，它会调用我们在OVERLAPPED结构中指定的回调函数。

归纳一下，APC机制所要解决的核心问题是跨越线程调用函数。因为调用发起者和被调用者可以不属于同一个线程，所以需要APC这样的系统机制让内核来帮助做这件事。如果不管被调用线程做什么而都中断它，强制其执行APC，那么可能会导致死锁等问题。因此，要把APC先放到队列中，等待被调用线程进入可接警状态后才能向其投递APC。形象地理解，APC机制提供了一种很优雅的跨线程协作方式，一个线程可以让另一个线程在它方便的时候做某件事。

除了上面介绍的因为异步文件操作引发的APC，微软还公开了一个名为QueueUserAPC的API。

```
DWORD QueueUserAPC( PAPCFUNC  pfnAPC,
  HANDLE    hThread, ULONG_PTR dwData );
```

通过这个APC，应用程序开发者可以让指定的目标线程执行指定的函数。目标线程可以是当前进程中的，也可以是其他进程中的。更有趣的是，目标线程也可以是发起线程。例如，在GeAPC小程序中，单击Queue APC按钮后，按钮消息的处理函数便会像下面这样调用QueueUserAPC。

```
void CGeAPCDlg::OnBnClickedQueueapc()
{
    DWORD dwRet = QueueUserAPC(Papcfunc, GetCurrentThread(), (ULONG_PTR)this);
    D4D(_T("QueueUserAPC returned 0x%x, gle = %d"), dwRet, GetLastError());
}
```

和前面的情况类似，单击Alertable按钮后，系统便会投递APC，Papcfunc函数会被调用。如果连续单击两次Queue APC按钮，那么系统便会把两个APC插入队列中，单击Alertable按钮后，一个APC投递后，会投递第二个。

## 5.2　中断请求级别

在微软的很多技术文档中经常会出现一个叫作中断请求级别（Interrupt ReQuest Level，IRQL）的术语。比如，在提供给驱动程序开发者的DDK/WDK文档中，每个内核API的描述里都有一栏叫作IRQL，描述使用这个API时一定要遵守的IRQL规则。如果违反这些规则，后果会非常严重，可能导致整个系统挂死或者蓝屏崩溃。翻阅描述蓝屏崩溃原因的文档，可以看到有十几种蓝屏崩溃都与IRQL有关，例如`RQL_NOT_LESS_OR_EQUAL`、`IRQL_GT_ZERO_AT_SYSTEM_SERVICE`、`IRQL_NOT_DISPATCH_LEVEL`等。

毫不夸张地说，IRQL是Windows系统中最重要的系统机制之一。同时，它也是NT内核中非常有特色的一个设计，在UNIX和Linux等系统中没有同类的概念，因此，可以说IRQL是NT内核的一个独特基因。

IRQL的字面意思是中断请求级别，不过这个名字并不十分准确。虽然它确实与硬件中断有着密切的关系，但其实它的影响范围不仅仅是中断处理。下面我们先介绍IRQL的设计初衷，然后再介绍它的工作原理，最后再讨论几个有关的问题。

### 5.2.1　设计初衷

在生活中，当需要专心处理一件重要的任务时，我们可能把自己关在一个房间里，门口挂上“请勿打扰”的标志。在软件世界里，也有这样的情况，比如操作系统内核执行某些原子操作时是不希望被打断的。解决这个问题的一种方法是屏蔽CPU的中断，拒绝打扰。但是这样做会带来下面所示的一些问题。

（1）问题A。如果关闭中断后执行的代码中存在瑕疵，执行很慢或者发生了死循环，那么CPU便可能长时间处于关闭中断的状态，对于单CPU的系统，这会导致整个系统出现卡顿甚至挂死的症状。对于多CPU的系统，如果其他CPU需要与这个CPU通信（一般通过IPI中断），那么可能也会受牵连，导致连锁反应，最后也可能导致系统瘫痪。

（2）问题B。如果系统中发生更重要的事情，比如电源故障或者硬件错误，那么内核就不能及时跳过去处理更重要的事务。

如何解决上述问题呢？一种思路是能够通过一种机制标识出CPU当前所执行代码的重要程度。当它做这件事时，如果有比这件事优先级更高的事件发生，可以让CPU跳过去执行更高优先级的；如果有比这件事优先级低的事件发生，那么就不要打扰CPU，让它先把高优先级的事情做完。这便是IRQL的设计初衷。

### 5.2.2　基本原理

简单说，IRQL机制就是为CPU要做的所有事情分类并定义每一类事情的优先级。每个优先级用一个数字来表示，从0开始，0代表最低优先级，数字越大，优先级越高。这个用数字表示的优先级便是IRQL。把所有IRQL列出来，便形成一张IRQL表（表5-1）。

表5-1　IRQL表

| IRQL | x86 | AMD64 | IA64 | 描　　述 |
| --- | --- | --- | --- | --- |
| PASSIVE_LEVEL | 0 | 0 | 0 | 被动级别，用于所有用户线程和内核空间的普通操作 |
| APC_LEVEL | 1 | 1 | 1 | 异步过程调用和处理页错误 |
| DISPATCH_LEVEL | 2 | 2 | 2 | 线程调度器和延迟过程调用 |
| CMC_LEVEL | N/A | N/A | 3 | 可纠正的机器检查（correctable machine-check）异常 |
| DIRQL | 3~26 | 3~11 | 4~11 | 用于处理硬件设备的中断 |
| PC_LEVEL | N/A | N/A | 12 | 性能计数器 |
| PROFILE_LEVEL | 27 | 15 | 15 | 性能分析使用的定时器（profiling timer，Windows 2000之前） |
| SYNCH_LEVEL | 27 | 13 | 13 | 跨处理器的代码和指令流同步 |
| CLOCK_LEVEL | N/A | 13 | 13 | 时钟定时器 |
| CLOCK2_LEVEL | 28 | N/A | N/A | x86硬件的时钟定时器 |
| IPI_LEVEL | 29 | 14 | 14 | 处理器间的中断 |
| POWER_LEVEL | 30 | 14 | 15 | 电源故障（power failure） |
| HIGH_LEVEL | 31 | 15 | 15 | 机器检查异常、蓝屏和转储，以及其他不可打断的重要操作 |


因为CPU要做的具体事情是和硬件平台相关的，所以IRQL的定义也是和平台相关的，表5-1中列出了常用的32位x86架构、64位x86（AMD64）以及安腾64位架构的IRQL。表中第一列是IRQL的名字（宏），接下来的三列是IRQL在对应平台上的取值，最后一列是描述。

在表5-1中，优先级最低的是`PASSIVE_LEVEL`，也叫`LOW_LEVEL`，优先级最高的叫`HIGH_LEVEL`，前者代表被动级别，意思是它级别最低，永远没机会打断其他级别的操作，只能被动地等待CPU把高优先级的事情全部做完后再来执行它。当CPU在执行用户空间的普通线程时，它的IRQL是`LOW_LEVEL`，这意味着，一旦有任何中断发生，CPU便会跳过去处理中断和其他高优先级的事情。当CPU的IRQL是`HIGH_LEVEL`时，代表这个CPU不再响应任何硬件中断请求，不可屏蔽中断（NMI）除外。

根据表5-1，某些操作必须在某个IRQL进行，否则就违反了IRQL规则，后果便是蓝屏崩溃，比如本节开头提到的蓝屏停止码`IRQL_NOT_DISPATCH_LEVEL`就用于“抱怨”当前的IRQL不对，不是要求的`DISPATCH_LEVEL`。再比如，停止码`IRQL_GT_ZERO_AT_SYSTEM_SERVICE`的意思是CPU执行系统服务后即将返回用户空间时，IRQL还大于0。而停止码`IRQL_NOT_LESS_OR_EQUAL`的意思是IRQL没有小于或者等于要求的值，也就是太大了。

### 5.2.3　析疑

首先值得强调的是，理解IRQL的一个关键是要清楚IRQL是内核赋予CPU的一个属性，用于描述CPU目前所做事情的重要程度。或者说，在多CPU系统中，每个CPU都有自己的IRQL。进一步说，NT内核会为系统中的每个处理器建立一个名为KPCR的结构，在这个结构中有一个名为IRQL的字段，它便是用于记录CPU的IRQL。

在内核调试会话中，可以使用!pcr命令获取当前CPU的KPCR结构地址，然后使用类似下面这样的命令读取IRQL。

```
0: kd> dt nt!_KPCR fffff803077dc000 -y IRQL
   +0x050 Irql : 0 ''
```

因此，上文故意避免使用任务这样的词汇，以避免大家把IRQL与线程的优先级混淆。区分二者的关键就是线程优先级是线程的属性，IRQL是CPU的属性。CPU在执行同一个线程的过程中，它的IRQL是可能变化的。举例来说，当CPU在用户空间执行一个普通线程时，IRQL为0，如果来了一个键盘中断，那么CPU跳过去处理中断，它的IRQL便会升高到DIRQL（D代表设备，具体值因中断而异），处理中断后，如果继续执行用户线程，那么IRQL又降低到0。

理解IRQL的另一个难点在于有些同行缺少底层硬件的知识，不知道内核是如何做到优先级控制的。简单说，这是依赖系统硬件内的中断控制器来实现的，即所谓的可编程中断控制器（Programmable Interrupt Controller，PIC）。以常见的英特尔架构为例，每个CPU内部都集成了一个高级可编程中断空间器（APIC），一般简称本地APIC（Local APIC），在系统的芯片组内也集成了中断控制器，一般称为I/O APIC。系统中的多个APIC相互协作，一起管理系统中的各种中断请求。APIC中实现了一系列寄存器，与系统软件接口，比如任务优先级寄存器（Task Priority Register，TPR）便是用来标识任务优先级的，可读可写。在NT内核和HAL模块中，实现了一系列函数，把NT世界的IRQL逻辑“贯彻”到APIC硬件中。实现细节从略，感兴趣的读者可以参考英特尔的APIC文档和Linux内核的开源代码。

最后再举一个例子来帮助大家深化对IRQL的理解。在驱动程序中，可以很方便地获取和改变IRQL。比如在笔者开发的RealBug驱动程序中，有下面这样一个函数，它把IRQL提升到指定的较高级别后，故意进入死循环，在高IRQL上“游荡”。

```
VOID RoamAtIRQL(ULONG ulIRQL)
{
    KIRQL OldIrql;
    KeRaiseIrql((KIRQL)ulIRQL, &OldIrql);
    while(1) 
        _asm{hlt};

    KeLowerIrql(OldIrql);
}
```

如果在配套的ImBuggy应用程序中以26为参数（x86系统）调用上面这个函数，那么会导致单CPU系统立刻挂死。对于多CPU系统，也会导致系统缓慢，或者出现其他怪异症状。原因就是CPU被黏滞在了高IRQL，如果IRQL不降下来，那么CPU就不会执行普通线程。有趣的是，如果此时使用内核调试器附加到挂死的系统，中断下来，在调试器中修改程序指针（r命令），让程序指针指向死循环下面的指令，然后恢复执行，让CPU执行循环下面的KeLowerIrql函数，把IRQL降下来，那么系统便恢复正常了。

既然CPU已经黏滞在IRQL 26上，它怎么还会响应内核调试器的中断（break）请求呢？因为内核调试器的中断请求依赖的是时钟定时器中断。仔细观察表5-1，在x86系统中，时钟定时器的中断优先级更高。也就是说，当CPU黏滞在IRQL 26上时，每当有时钟定时器中断时，CPU仍会跳过去处理，在处理这个时钟定时器中断时，NT内核会检查有没有内核调试器的中断请求。通过这个例子，也可以看出NT内核设计的IRQL机制很合理，是非常先进的。

## 5.3　延迟过程调用

5.2节介绍的IRQL机制为Windows系统中的软件行为做了分类，并定义了严格的优先级。以IRQL为核心的优先级规则是NT内核中的根本法规，其影响是深远而且广泛的。如此森严的“等级制度”是一把双刃剑，它在带来好处的同时，也带来了一些问题。

IRQL机制的一条核心规则是如果CPU有高IRQL的事情没有做完，那么就不去做低IRQL的事情。换句话来说，如果CPU的IRQL为N，那么这个CPU就不回去执行IRQL小于N的操作。5.2节末尾举的例子中，有不好的代码把IRQL升上去，不降回来，那么系统便会出现严重的故障。

这意味着，如果CPU在高IRQL上运行，出问题时产生的影响更大，系统的风险更高。这有点像现实世界中站在平地上摔个跟头和在摩天大楼的楼顶滑一跤是完全不一样的。换句话来说，在高IRQL与低IRQL时所执行的操作有很大不同，这也是为什么WDK文档为每个DDI（设备驱动程序接口）函数严格指定IRQL要求。举例来说，当IRQL高于`DISPATCH_LEVEL`时，是不可以调用KeWaitForSingleObject这样的等待函数的。一个简单的原因就是在高IRQL上等待可能导致严重的系统级别死锁。出于种种原因，大多数内核函数只可以在`PASSIVE_LEVEL`调用，可以在`APC_LEVEL (1)`级别调用的内核函数也不多，能够在`DISPATCH_LEVEL (2)`和更高级别调用的就更少了。

概而言之，等级森严的IRQL规则决定了在高IRQL时的操作要快速完成，对应的代码要简单短小。那如果确实需要执行比较长的操作怎么办呢？为了解决这个问题，NT内核设计了一个专门的机制，叫延迟过程调用（Deffered Procedure Call，DPC）。

### 5.3.1　使用模式

使用DPC的一个典型场景是设备驱动程序的中断处理函数（ISR）。ISR一般在DIRQL级别运行，不适宜做复杂的操作，如果需要复杂的操作，那么便应该使用DPC。一般把事先已经初始化的一个DPC对象插入某个CPU的DPC队列中。可以使用DDI中的KeInsertQueueDpc函数来向DPC队列插入DPC。在WDF中，封装了一个名为WdfInterruptQueueDpcForIsr的函数来简化操作。希望了解代码细节的读者可以阅读WDK中的示例程序，路径如下。

```
Windows-driver-samples/general/PLX9x5x/sys/IsrDpc.c
```

DPC队列也是与CPU相关的，NT内核为系统中的每个CPU维护一个DPC队列，在内核调试会话中，可以使用!pcr或者!dpcs命令来观察DPC队列。比如：

```
0: kd> !dpcs
CPU Type      KDPC       Function
 0: Normal  : 0xfffff80307799420 0xfffff80307482cf0 nt!PpmCheckPeriodicStart
 0: Normal  : 0xffffe0009aa42738 0xfffff80186ea9480 dxgkrnl!DpiFdoDpcForIsr
```

每个DPC对象是一个KDPC结构，可以使用dt命令来观察，比如，第二个DPC的详细属性如下。

```
0: kd> dt _KDPC 0xffffe0009aa42738 
ntdll!_KDPC
   +0x000 TargetInfoAsUlong : 0x113
   +0x000 Type              : 0x13 ''
   +0x001 Importance        : 0x1 ''
   +0x002 Number            : 0
   +0x008 DpcListEntry      : _SINGLE_LIST_ENTRY
   +0x010 ProcessorHistory  : 1
   +0x018 DeferredRoutine    : 0xfffff801`86ea9480  void  dxgkrnl!DpiFdoDpcForIsr+0
   +0x020 DeferredContext   : 0xffffe000`9aa42160 Void
   +0x028 SystemArgument1   : (null) 
   +0x030 SystemArgument2   : (null) 
   +0x038 DpcData           : 0xffffd001`62f871f0 Void
```

其中的DeferredRoutine是函数指针，指向这个DPC关联的回调函数。

那么，系统什么时候会清理DPC队列呢？文档上的描述是IRQL降低到DISPATCH_LEVEL或者以下。清理DPC队列的实际情况有多种，清单5-5显示了比较多见的一种。

清单5-5　清理DPC队列

```
# Call Site
00 nt!KeInsertQueueApc
01 afd!AfdTLConnectComplete2
02 afd!AfdTLConnectComplete
03 tcpip!TcpCreateAndConnectTcbComplete
04 tcpip!TcpShutdownTcb
05 tcpip!TcpAbortTcbDelivery
06 tcpip!TcpRetransmitTimeout
07 tcpip!TcpProcessExpiredTcbTimers
08 tcpip!TcpPeriodicTimeoutHandler
09 nt!KiExecuteAllDpcs
0a nt!KiRetireDpcList
0b nt!KiIdleLoop
```

从清单5-5最下面的KiIdleLoop函数可以看出，这次清理DPC队列发生在CPU的空闲线程中。从栈帧08可以看出，正在清理的DPC是属于tcpip内核驱动程序的，它在执行了一系列操作后，调用AFD（NT网络栈的内核空间接口模块）驱动程序的AfdTLConnectComplete完成函数，这个函数插入了一个APC对象，应该用于回调用户空间回调函数，如5.1节所讲。

内核在调用DPC的回调函数时，把CPU的IRQL设置为DISPATCH_LEVEL。这样做的好处是，可以让DPC的回调函数拥有较高的优先级，优先执行。使用DPC和APC典型过程如表5-2所示。

表5-2　使用DPC和APC的典型过程

| 步　 骤 | 操　　作 | IRQL |
| --- | --- | --- |
| 1 | 应用程序发起异步I/O请求，比如读数据 | 0 |
| 2 | 设备驱动程序收到请求后向硬件下发操作指令 | 0 |
| 3 | 硬件准备好数据后，通过中断通知驱动程序 | N/A |
| 4 | 驱动程序的ISR简单处理后，向DPC队列插入一个DPC | DIRQL |
| 5 | DPC的回调函数被执行，对数据做处理后，插入APC通知应用程序 | 2 |
| 6 | 系统投递APC，应用程序的回调函数被调用，得到通知和数据 | 1/0 |


观察表5-2的IRQL列，可以看到IRQL的数值先由小到大，再由大到小。驱动程序和应用程序分工协作，IRQL有序起伏，先后关系很合理。

### 5.3.2　黏滞在DPC

在DISPATCH_LEVEL执行DPC函数也意味着风险。如果在DPC函数中操作不当，发生意外，那么也可能导致严重的系统故障。比如清单 5-6 所示的便是笔者亲历的因为CPU黏滞在DPC上而导致的系统挂死。

清单5-6　CPU黏滞在DPC上而导致的系统挂死

```
00 nt!KeBugCheckEx
01 i8042prt!I8xProcessCrashDump
02 i8042prt!I8042KeyboardInterruptService
03 nt!KiInterruptDispatch
04 hal!KeStallExecutionProcessor
05 usbehci!EHCI_RH_PortResetComplete
06 USBPORT!USBPORT_AsyncTimerDpc
07 nt!KiTimerListExpire
08 nt!KiTimerExpiration
09 nt!KiRetireDpcList
0a nt!KiIdleLoop
```

从栈帧05可以看出，有问题的是USB 2.0主机控制器（EHCI）驱动的`EHCI_RH_PortResetComplete`函数，它是用于完成USB端口复位操作的。在这个函数中，它进入一个循环，反复写USB控制器的端口状态和控制寄存器，写了后等待一会儿再读，读了很多次后，如果没有看到希望的值，再重复写，如此循环，把CPU黏滞在DISPATCH_LEVEL了，这导致系统中唯一的CPU没有机会执行普通任务，所有界面没有响应。栈帧#00～#03表示笔者按下了触发蓝屏的快捷键，键盘驱动的ISR被执行，调用KeBugCheckEx产生蓝屏和转储。《格蠹汇编》一书详细地介绍了这个案例，在此不再赘述。

清单5-6所示的问题发生在Windows XP系统中，在Windows 10系统中，如果发生类似的黏滞在DPC上的情况，那么系统中的DPC看门狗（Watch dog）会触发系统蓝屏崩溃，停止码为DPC_WATCHDOG_VIOLATION（0x133）。搜索网络，可以看到很多用户遇到过这个蓝屏。这侧面反映了DPC机制在NT内核中的重要地位。

本节收尾之际，略作归纳。如果说APC机制是为了提供一种优雅的机制来跨越线程调用一个函数，那么DPC便提供了一种系统机制来跨越森严的IRQL等级界限来调用一个函数。如果说APC机制是很礼貌地请另一个线程在它方便的时候做件事，那么DPC机制就是请CPU在从高空走下来后，在不十分危险的时候从容地完成刚才不方便做的事。

## 5.4　本地过程调用

在Windows系统中，广泛使用着一种基于端口和消息的通信机制，它有个不是很恰当的名字，叫作本地过程调用（Local Procedure Call）或者轻量级过程调用（Light-weight Procedure Call，LPC）。事实上，它是一种使用“客户端／服务器”模型的跨进程通信进制。因此对LPC全称的更恰当说法是本地跨进程通信（Local Inter-Process Communication）。

使用LPC通信的一般过程如下所示。

```
服务器端调用 NtCreatePort 创建服务端口
↓
服务器端调用 NtListenPort 监听端口
↓
客户端调用 NtConnectPort 连接端口
↓
服务器端调用 NtAcceptConnectPort 接受连接
↓
服务器端调用 NtCompleteConnectPort 完成连接
↓
客户端调用 NtRequestWaitReplyPort 发送消息和等待回复
↓
服务器端调用 NtReplyWaitReceivePort 接收数据和发送回复
```

Windows Vista对LPC的实现做了增强，并且使用了一个新的名字，叫作ALPC，关于缩写A的全称有两种说法，一种是Advanced，另一种是Asynchronous。本书仍使用LPC一词泛指传统的LPC和改进后的ALPC。

与DPC和APC机制类似，LPC机制在NT内核中也是根深蒂固的，影响广泛，很多地方可以看到它的身影。

在内核调试会话中，可以非常方便地观察LPC有关的信息，下面分别举一些例子。

首先使用列进程命令找到希望观察的进程，比如，我们选择会话管理器进程。

```
0: kd> !process 0 0 smss.exe
PROCESS ffffc48d86b00640
```

然后便可以使用!alpc /lpp命令加进程的EPROCESS地址列出指定进程的所有LPC端口了。

```
0: kd> !alpc /lpp ffffc48d86b00640
Ports created by the process ffffc48d86b00640:
    ffffc48d8895de20('SmApiPort') 0, 3 connections
        ffffc48d8b63bbd0 0 -> ffffc48d8b63be20 0 ffffc48d869fd640('csrss.exe')
        ffffc48d8b6f9e20 0 -> ffffc48d8b6fabb0 0 ffffc48d8b679640('csrss.exe')
        ffffc48d8b83bad0 0 -> ffffc48d8b83bd20 0 ffffc48d8b7fb080('svchost.exe')
Ports the process ffffc48d86b00640 is connected to:
    ffffc48d8b63ae20 0 -> ffffc48d8b63cb70('SbApiPort') 0 ffffc48d869fd640('csrss.exe')
    ffffc48d8b6f9bd0 0 -> ffffc48d8b6fae20('SbApiPort') 0 ffffc48d8b679640('csrss.exe')
```

上面的结果包含两个部分，上半部分是SMSS进程创建的端口，属于服务器端，名为'SmApiPort'，它有3个连接。下半部分是SMSS进程作为客户端连接的服务器端口。注意，虽然名字都是'SbApiPort'，但是其实是不同的两个服务器端口，属于两个Windows子系统服务器（CSRSS）进程。

端口名前面的地址就是LPC端口对象，可以使用!alpc /p加这个地址来观察其详情。

```
0: kd> !alpc /p ffffc48d8895de20
Port  ffffc48d8895de20
  Type                      : ALPC_CONNECTION_PORT
  CommunicationInfo         : ffffa4805c1ca610
    ConnectionPort          : ffffc48d8895de20 (SmApiPort)
    ClientCommunicationPort : 0000000000000000
    ServerCommunicationPort : 0000000000000000
  OwnerProcess              : ffffc48d86b00640 (smss.exe)
  SequenceNo                : 0x00000005 (5)
  CompletionPort            : ffffc48d888ee1c0
  CompletionList            : 0000000000000000
  ConnectionPending         : No
  ConnectionRefused         : No
  Disconnected              : No
  Closed                    : No
  FlushOnClose              : Yes
  ReturnExtendedInfo        : No
  Waitable                  : No
  Security                  : Static
  Wow64CompletionList       : No

  1 thread(s) are registered with port IO completion object:
    THREAD ffffc48d8895b080  Cid 01d8.0208  Teb: 00000063218f8000 Win32Thread:  
 0000000000000000 WAIT
  Main queue is empty.
  Direct message queue is empty.
  Large message queue is empty.
  Pending queue is empty.
```

空行上面的部分是LPC对象的基本属性。空行下面是列表部分，第一个列表是为这个端口服务的服务线程，目前只有一个，就是SMSS进程中提供会话服务的工作线程。线程列表后面是几个队列的状态，目前都为空，说明这个端口很空闲。对于繁忙的端口或者当服务线程出现故障时，队列里可能有很多消息在排队。

进一步分析拥堵原因的一般方法就是看服务线程的状态，看它们为什么没有处理消息。

对于处于排队状态的消息，可以使用!alpc /m命令来观察每个消息的详情，比如：

```
0: kd> !alpc /m ffffa480730b6890

Message ffffa480730b6890
  MessageID             : 0x59C8 (22984)
  CallbackID            : 0x24AE18E (38461838)
  SequenceNumber        : 0x00000207 (519)
  Type                  : LPC_REQUEST
  DataLength            : 0x00A4 (164)
  TotalLength           : 0x00CC (204)
  Canceled              : No
  Release               : No
  ReplyWaitReply        : No
  Continuation          : Yes
  OwnerPort             : ffffc48da7067070 [ALPC_CLIENT_COMMUNICATION_PORT]
  WaitingThread         : ffffc48da6de0700
  QueueType             : ALPC_MSGQUEUE_MAIN
  QueuePort             : ffffc48da9178090 [ALPC_CONNECTION_PORT]
  QueuePortOwnerProcess : ffffc48d958a1080 (audiodg.exe)
【省略一些行】
```

其中的WaitingThread描述的是正在等待这个消息的客户线程。值得说明的是，WinDBG在显示线程的状态时，如果这个线程在等待ALPC消息，那么在!thread命令的结果中也会明确显示这个信息，比如，使用!thread观察上面显示的等待线程。

```
0: kd> !thread ffffc48da6de0700
THREAD ffffc48da6de0700  Cid 0994.4fcc  Teb: 0000005902f10000 Win32Thread:   
0000000000000000 WAIT: (Suspended) KernelMode Non-Alertable
SuspendCount 1
FreezeCount 1
    ffffc48da6de09e0  NotificationEvent
Waiting for reply to ALPC Message ffffa480730b6890 : queued at port  
ffffc48da9178090 : owned by process ffffc48d958a1080
```

可以看到WinDBG非常清晰地显示了这个线程在等待ALPC消息，这个消息在端口xxx排队，拥有这个端口的进程是xxx，真可谓无微不至。

概而言之，LPC为Windows系统提供了一种高效的跨进程通信方式，可以使两个进程的用户空间代码相互通信，也可以使内核空间的代码和用户空间通信。严格来说，LPC不是像APC、DPC和5.5节将介绍的RPC那样的函数调用机制，只是一种消息通信机制，但因为其名字与APC、DPC和RPC类似而且也在调试时经常遇到，所以一并在本章中介绍。

## 5.5　远程过程调用

5.5节介绍的LPC机制是供Windows系统内部使用的，没有公开给开发者。一种间接的方式是通过微软公开的RPC编程接口来使用LPC。

RPC的全称是远程过程调用（Remote Procedure Call），其定位是为Windows平台提供一套强大的跨进程通信机制，让构建分布式计算软件更容易。RPC的基本思想是既能提供强大的远程调用能力，又不损失本地调用的语义简洁性。或者说，RPC技术的目标是让开发者像调用本地函数一样调用远程函数，开发者不必关心底层的通信细节，只要按设计好的函数原型发起调用，就既可以调用本地的函数实现，也可以调用远程的函数实现。为了实现这个目标，RPC技术提供了一种透明调用机制让使用者不必显式地区分本地调用和远程调用。

举例来说，如果服务器端实现了如下函数：

```
void HelloProc(unsigned char * pszString)
```

那么客户端只要像下面这样调用：

```
HelloProc(pszString);  // make call with user message
```

### 5.5.1　工作模型

如下显示了RPC的工作模型，左侧是客户端进程，右侧是服务器进程。当客户进程调用某个函数时，它首先执行的是这个函数的桩（stub），而不是真正的函数实现。当编译和链接客户端程序时，编译器链接的便是桩函数。

```
客户端应用程序
1↓   ↑14
客户端端桩（Client Stub）
2↓   ↑13
客户端运行时库
3↓   ↑12
传输层（客户端）
4↓   ↑11
传输层（服务器端）
5↓   ↑10
服务器端运行时库
6↓   ↑9
服务器端桩（Server Stub）
7↓   ↑8
服务器端应用程序
```

桩函数接到调用后，负责收集参数，并把参数翻译为可以在网络上传输的标准数据包，称为网络数据表示（Network Data Representation，NDR）。在封装好参数后，桩函数会调用RPC的客户端运行时函数，后者通过传输层把数据发给服务器端。

举例来说，对于上面提到的HelloProc，编译工具会为其产生下面这样的桩函数：

```
void HelloProc( 
    /* [string][in] */ unsigned char *pszString)
{
    NdrClientCall2(
        ( PMIDL_STUB_DESC  )&hello_StubDesc,
        (PFORMAT_STRING) &__MIDL_ProcFormatString.Format[0],
        pszString);    
}
```

其中的NdrClientCall2就是RPC运行时中的重要函数，供客户端发起调用。

服务器端的运行时函数收到数据包后，调用服务端的桩函数，服务器桩函数把NDR数据解开，转换为普通形式的参数，然后调用函数的真正实现。如果函数有返回值或者返回类型的参数，那么服务器桩函数会把返回数据打包，然后调用服务器端的RPC运行时函数将其发回给客户端。

### 5.5.2　RPC子系统服务

在今天的Windows系统中，有几个后台服务是必须启动而且不允许禁止的，其中之一便是所谓的RPC子系统（Remote Process Control Subsystem）服务，简称RpcSs。

从历史角度看，微软在实现RPC技术的时候，同时也在开发COM（组件对象模型）和DCOM技术。因此，从实现的角度来看，RPC与COM和DCOM技术有很多交叉和复用，很多开发工具和基础设施也是共享的。比如，在RPC中，也使用COM和DCOM中使用的IDL语言来描述接口。又如，在RpcSs中，既有对RPC的全局支持，也有对COM和DCOM的关键支持。因为Windows操作系统的很多内部机制是离不开COM/DCOM和RPC的，所以RpcSs是不允许禁止的。

试图使用WinDBG等调试器附加到RpcSs所在的进程也是非常危险的，当你感觉似乎附加成功了的时候，WinDBG可能很快就挂死了，因为WinDBG加载符号时可能要访问网络，访问网络时可能就要触发RPC调用，于是很多东西便锁在一起了。当你忍无可忍、想强制关闭WinDBG时，你很可能难以做到，也可能做到了，但是立刻得到一个蓝屏崩溃，停止码为0xF4，代表CRITICAL_OBJECT_TERMINATION。

如果你很想用调试方法了解RpcSs，那么一种比较温和而且安全的方式是在任务管理器里找到RpcSs所对应的进程，然后产生转储文件，再使用WinDBG打开转储文件。另一种方法是使用内核调试会话。

下面是rpcss模块中的部分重要函数。

```
rpcss!GetEndpoint
rpcss!ActivateRemote
rpcss!CRemoteMachine::Activate
rpcss!CProcess::ActivateProcess
rpcss!BreakOnDebuggedClsid
rpcss!CWinRTActivationStoreCatalog::CServerInfo::GetDebuggerCommandLine
```

其中，最后两个是用于支持调试的。

### 5.5.3　端点和协议串

端点是RPC中的一个关键概念。简单来说，端点是RPC的服务器端与客户端建立联系和通信的纽带。RPC服务器端启动后，它便会监听一个或者多个端点，客户端启动后，也是通过端点与服务器端建立联系的。

每个端点具有固定的传输层。RPC的传输层有很多种选择，比如命名管道、TCP、UDP、IPX（Internet Packet Exchange）、SPX（Sequenced Packet Exchange）、LPC等。

在传输层上面，RPC层存在多种协议，目前，微软支持以下3类RPC协议。

（1）面向连接的（connection-oriented）协议，全称为面向连接的网络计算架构（Network Computing Architecture Connection Oriented，NCACN）。

（2）数据报文协议，全称为数据报文网络计算架构（Network Computing Architecture Datagram，NCADG）。

（3）本地的远程过程调用，全称为本地远程过程调用网络计算架构（Network Computing Architecture Local Remote Procedure Call，NCALRPC）。

这意味着，在使用RPC时，RPC双方必须明确RPC协议和传输层协议。在RPC中，专门使用一个固定的字符串来标识不同的协议组合，并给这样的字符串取了个专门的名字，叫作协议串（protocol sequence），有时简写为ProtoSeq。

举例来说，`ncacn_nb_tcp`代表的RPC协议为ncacn，即面向连接的RPC协议，传输层使用的是NetBIOS TCP。微软的文档中有完整的协议串列表，在此从略。

在Windows系统中，有一个系统服务，用于匹配RPC端点，名叫EP Mapper。端点匹配服务使用的服务模块与RPC子系统服务是一样的，都是RPCSS.dll。

### 5.5.4　蜂巢

当RPC设施工作时，有一种特别的方式来存储各种类型的RPC对象，称为蜂巢（cell）。每个蜂巢都有自己的ID，一般表示为x.y的形式。RPC运行时专门维护一个单独的堆来分配和释放蜂巢，这个堆便叫蜂巢堆（cell heap）。在RPC运行时里，有专门的函数来从蜂巢堆上分配蜂巢，比如RPCRT4!CellHeap::AllocateCell。

蜂巢堆上存储的RPC对象有如下几种。

（1）端点（endpoint）。

（2）线程（thread），比如RPC服务器端的工作线程。

（3）连接，即客户端和服务器端的连接实例。

（4）服务器端调用（Server Call），简称SCALL。

（5）客户端调用（Client Call），简称CCALL。

蜂巢里保存着每个RPC对象的属性，在调试RPC时，这些属性的价值是非常高的。为此，我们特意把这些属性的详细信息整理在表5-3中。

表5-3　保存在蜂巢中的RPC对象的属性

| 对 象 | 属 性 | 说 明 |
| --- | --- | --- |
| 端点 | ProtseqType | 该端点的协议串类型 |
| 端点 | Status | 端点的状态，有3种值——allocated（已分配）、active（活跃）和inactive（不活跃） |
| 端点 | EndpointName | 端点名称的前28个字符 |
| 线程 | Status | 线程状态，有4种值——processing、dispatched、allocated、idle |
| 线程 | LastUpdateTime | 上次更新时间（系统启动后的毫秒数） |
| 线程 | TID | 线程ID |
| 连接 | Flags | 标志，用来指示是否为互斥模式（exclusive/non-exclusive）、认证级别和认证服务状态 |
| 连接 | LastTransmitFragmentSize | 通过这个连接传输的上一个数据片的大小 |
| 连接 | Endpoint | 所属端点的蜂巢ID |
| 连接 | LastSendTime | 上次发送时间 |
| 连接 | LastReceiveTime | 上次接收时间 |
| 服务器端调用（SCALL） | Status | 调用状态，有3种值——已分配（allocated）、活跃（active，RPC运行时正在处理）和已分发（dispatched，服务函数已经调用但还没有返回） |
| 服务器端调用（SCALL） | ProcNum | 过程序号（procedure number），要调用函数在接口中的序号，从0开始 |
| 服务器端调用（SCALL） | InterfaceUUIDStart | 接口的第一个DWORD |
| 服务器端调用（SCALL） | ServicingTID | 服务线程的蜂巢ID，如果调用状态不是活跃或者已分配，那么此信息可能有误 |
| 服务器端调用（SCALL） | CallFlags | 调用标志，指示是否为被缓存的调用（cached call）、异步调用、管道调用、LRPC或者OSF调用 |
| 服务器端调用（SCALL） | LastUpdateTime | 上次更新时间 |
| 服务器端调用（SCALL） | PID | 调用者的进程ID，仅当LRPC时有效 |
| 服务器端调用（SCALL） | TID | 调用者的线程ID，仅当LRPC时有效 |
| 客户端调用（CCALL） | ProcNum | 被调用方法的过程序号，与服务器端调用的ProcNum属性含义相同 |
| 客户端调用（CCALL） | ServicingThread | 发起调用的工作线程的蜂巢ID |
| 客户端调用（CCALL） | IfStart | 接口UUID的第一个DWORD |
| 客户端调用（CCALL） | Endpoint | 这个调用的服务器端端点名字的前12个字符 |
| 客户端调用（CCALL） | ProtocolSequence | 协议串 |
| 客户端调用（CCALL） | LastUpdateTime | 上次更新时间 |
| 客户端调用（CCALL） | TargetServer | 服务器名称的前24个字符 |


使用后面介绍的RPC调试工具可以读取上表中的属性，稍后会继续介绍。

RPC调试的一大难点便是调试工具的输出信息常常包含各种简写和晦涩术语，特汇集于一表备查。

### 5.5.5　案例和调试方法

举例来说，笔者曾经遇到Windows版本的微信程序在接听语音呼叫时挂死，上调试器，发现UI线程因为调用waveInOpen API而陷入等待。

```
00 ntdll!NtWaitForSingleObject
…
07 wdmaud!widMessage
08 winmmbase!waveInOpen
```

继续追查进程中的其他线程，发现语音API的工作线程通过RPC调用系统中的语音服务（清单5-7）。

清单5-7　API的工作线程通过RPC调用系统中的语音服务

```
0:031> kc
 # 
00 ntdll!NtAlpcSendWaitReceivePort
01 rpcrt4!LRPC_BASE_CCALL::DoSendReceive
02 rpcrt4!NdrClientCall2
03 rpcrt4!NdrClientCall4
04 AudioSes!CAudioClient::InitializeAudioServer
05 AudioSes!CAudioClient::InitializeInternal
06 AudioSes!CAudioClient::Initialize
07 wdmaud!CWaveHandle::_Open
08 wdmaud!CWaveOutHandle::_Open
09 wdmaud!`CWaveHandle::Open'::`2'::COpenJob::Work
0a wdmaud!CWorker::_ThreadProc
0b wdmaud!CWorker::_StaticThreadProc
0c kernel32!BaseThreadInitThunk
0d ntdll!__RtlUserThreadStart
0e ntdll!_RtlUserThreadStart
```

注意观察上面的栈回溯，其中栈帧03中的rpcrt4便是RPC的运行时模块，NdrClientCall4是客户端发起调用的函数。栈帧01中的`LRPC_BASE_CCALL`表示使用LPC作为传输层。栈帧00表示调用NtAlpcSendWaitReceivePort系统服务进入内核空间。

对于使用LPC作为传输层的RPC阻塞，继续追查的最好方法便是使用内核调试会话，比如借助LiveKD工具（第18章）到内核空间观察这个线程的详情，一般就会看到这个线程所等待的LPC消息和服务进程，像5.4节（使用 LPC 通信的一般过程）所描述的那样，本节不再重复。下面继续介绍使用其他传输层的情况。

如果RPC的传输层使用的不是LPC，而是网络或者命名管道等方式，那么一般使用如下调试方法：先启用RPC服务的调试支持，让其保存状态信息，再使用RPC调试工具来获取状态信息。

首先要启用RPC的调试支持。执行gpedit.msc启动本地组策略编辑器，然后依次选择“计算机配置”→“管理模板”→“系统”→“远程过程调用”找到关于RPC的配置页面。

然后双击右侧的“维护RPC疑难解答状态信息”（这个中文翻译很别扭）。在接下来打开的配置页面中先选择“已启用”启用RPC状态保存，再在下面关于保存信息多少的选项中选择“完全”。做了这个改动后要重启系统才能生效。

做好上面的准备后，有两个工具可供选择，一个是WinDBG的扩展命令模块rpcexts.dll，另一个是WinDBG工具集附带的工具DbgRPC.exe。二者用法大同小异，我们以前者为例介绍基本过程。

下面通过一个实例来介绍。在微软的平台SDK示例程序中，有一个名为Hello的示例，其路径为Samples\NetDS\Hello。笔者对其做了一些改进，在HelloProc函数中加入代码，让其可以模拟服务器端挂死。

```
void HelloProc(unsigned char * pszString)
{
    printf("GeRPC HelloProc got called. I will sleep %d seconds \n", 
    nMimicHang);
    Sleep(nMimicHang*1000);
    printf("%s\n", pszString);
}
```

编译好修改后的代码，以如下命令启动服务器端。

```
C:\sdbg2e\ch205\GeRPC>hellos -h 10000
```

服务器端会打印消息，显示进入监听状态。接下来，执行helloc启动客户端。收到客户端的连接后，服务器端故意睡觉不回复，于是导致双方都僵持在那里。

如何调试这样的问题呢？我们假定不知道服务器端是哪个进程，在哪里。

以管理员身份启动WinDBG，附加到挂死的客户端进程。注意，一定要以管理员身份执行WinDBG，不然稍后执行rpcexts的命令时会因为权限不足而失败。

执行～0s切换到挂死的0号线程，执行k命令可以看到客户端调用RPC函数的执行经过（清单5-8）。

清单5-8　客户端调用RPC函数的经过

```
# Call Site
00 ntdll!NtWaitForSingleObject
01 KERNELBASE!WaitForSingleObjectEx
02 RPCRT4!UTIL_WaitForSyncIO
03 RPCRT4!UTIL_GetOverlappedResultEx
04 RPCRT4!NMP_SyncSendRecv
05 RPCRT4!OSF_CCONNECTION::TransSendReceive
06 RPCRT4!OSF_CCONNECTION::SendFragment
07 RPCRT4!OSF_CCALL::SendNextFragment
08 RPCRT4!OSF_CCALL::FastSendReceive
09 RPCRT4!OSF_CCALL::SendReceiveHelper
0a RPCRT4!OSF_CLIENT_MESSAGE_SENDER::SendReceive
0b RPCRT4!NdrpClientCall2
0c RPCRT4!NdrClientCall2
0d helloc!main
0e helloc!mainCRTStartup
0f KERNEL32!BaseThreadInitThunk
10 ntdll!RtlUserThreadStart
```

因为被调试的helloc程序是发布版本，所以栈回溯中，没有显示出桩函数的名字HelloProc，这给调试增加了难度。那么，该如何找到出现问题的调用，以及这次调用没有返回的原因呢？接近目标的一个快捷方法是使用!rpcexts.getclientcallinfo命令，让其打印出客户端调用列表。

客户端调用列表显示了一个客户端调用，以表格形式显示了它的详细信息。表格中的各个部分的含义可以参考上文中的表5-3。PID代表进程ID，即当前客户端进程的进程ID。接下来是这个客户端调用对象的蜂巢ID。再接下来的PNO是ProcNo的简写，代表被调用过程的序号。查看接口的IDL文件，可以看到所有方法的列表，其序号从0开始，比如下面是我们的Hello例子使用的接口，来自hello.idl文件。

```
interface hello
{
void HelloProc([in, string] unsigned char * pszString);
void Shutdown(void);
}
```

随后的IFSTART代表接口UUID的起始部分。接下来的TIDNUMBER代表调用线程的ID（calling thread ID）。后面的CALLID是这次调用的ID。接下来的LASTTIME是上次更新时间，可以使用后面介绍的getdbgcell命令将其翻译为秒数，最后一个部分是这个调用的端点名称，即\pipe\hello。

上面的信息中，最有价值的是端点名称，因为有了这个名称，使用getendpointinfo命令就可以获取到服务端的信息。

```
0:001> !getendpointinfo \pipe\hello
Searching for endpoint info ...
PID  CELL ID   ST PROTSEQ        ENDPOINT                    
-------------------------------------------------------------
06cc 0000.0002 01            NMP \pipe\hello
```

上面结果的第一列便是服务进程的ID，而后是蜂巢ID，ST列用来描述端点的状态，1代表活跃状态（active），0表示已分配（allocated）。随后一列是协议串，NMP代表命名管道，最后一列仍是端点名称。顺便说一下，如果不指定端点名，那么这条命令会列出系统中的所有端点。

有了服务端的进程ID后，便可以使用WinDBG附加到服务进程，追查进一步的原因了。仍以管理员身份运行WinDBG，附加到ID为6cc的进程，其实就是Hellos进程。观察0号线程，可以看到它在监听RPC连接（清单5-9）。

清单5-9　监听RPC连接

```
00 ntdll!NtWaitForSingleObject
01 KERNELBASE!WaitForSingleObjectEx
02 RPCRT4!EVENT::Wait
03 RPCRT4!RPC_SERVER::WaitForStopServerListening
04 RPCRT4!RPC_SERVER::ServerListen
05 RPCRT4!RpcServerListen
06 hellos!main
07 hellos!mainCRTStartup
08 KERNEL32!BaseThreadInitThunk
09 ntdll!RtlUserThreadStart
```

继续观察1号线程，可以看到它正在处理RPC（清单5-10）。

清单5-10　处理RPC

```
# Call Site
00 ntdll!NtDelayExecution
01 KERNELBASE!SleepEx
02 hellos!HelloProc
03 RPCRT4!Invoke
04 RPCRT4!NdrStubCall2
05 RPCRT4!NdrServerCall2
06 RPCRT4!DispatchToStubInCNoAvrf
07 RPCRT4!RPC_INTERFACE::DispatchToStubWorker
08 RPCRT4!OSF_SCALL::DispatchHelper
09 RPCRT4!OSF_SCALL::ProcessReceivedPDU
0a RPCRT4!OSF_SCONNECTION::ProcessReceiveComplete
0b RPCRT4!CO_ConnectionThreadPoolCallback
0c RPCRT4!CO_NmpThreadPoolCallback
0d KERNELBASE!BasepTpIoCallback
0e ntdll!TppIopExecuteCallback
0f ntdll!TppWorkerThread
10 KERNEL32!BaseThreadInitThunk
11 ntdll!RtlUserThreadStart
```

在清单5-10中，可以看到RPC运行时模块（RPCRT4）在调用RPC的目标函数HelloProc，HelloProc在调用Sleep，导致这个RPC调用黏滞在这里，无法返回。

最后再介绍几条有用的命令。getclientcallinfo用来显示客户端调用信息，也就是清单5-8中桩函数调用NdrClientCall2的信息。类似地，getcallinfo用来显示服务器端调用的情况，也就是服务线程调用NdrServerCall2的信息（清单5-10）。不带任何参数会显示很多内容，通常要加过滤条件，比如`!rpcexts.getcallinfo 0 0 0 6cc`命令会显示6cc这个服务进程的调用。

对于每次调用，可以使用getdbgcell命令获取其详细信息，例如：

```
0:004> !rpcexts.getdbgcell 6cc 0.4
Getting cell info ...
Call
Status: Active
Procedure Number: 0
Interface UUID start (first DWORD only): 0
Call ID: 0x0 (0)
Servicing thread identifier: 0x0.0
Call Flags: cached
Last update time (in seconds since boot):37790.343 (0x939E.157)
Owning connection identifier: 0x0.3
```

可以看到这个调用处于活跃状态。倒数第二行是它的上次更新时间。使用rpctime命令可以获取当前时间。

```
0:004> !rpcexts.rpctime
Current time is: 054706.609 (0x00d5b2.261)
```

二者相减，可以看到已经过去几小时（(54706—37790)/3600≈4.7），这个调用一定是出问题了。

## 5.6　本章总结

本章介绍了Windows操作系统中的APC、DPC和RPC。概而言之，它们都是为了实现特殊的函数调用，为了跨越某种边界做调用。简单来说，APC跨越的是线程边界，DPC跨越的是IRQL边界，RPC跨越的是进程边界。

LPC不仅名字与APC、DPC和RPC相近，而且它在Windows系统中的重要性与其他三者相比也是不相上下的，特别是与RPC经常相伴使用。因此我们也在本章对其做了介绍。

本章的目的有多个。一方面是因为这些特殊的过程调用和IRQL是NT的基因，了解它们对理解NT的价值很大。另一方面，因为理解这些概念有较大的难度，调试时又经常遇到，所以我们特别拿出来讲解，帮助大家攻克难关，增强信心。还有，我们在介绍时，故意使用了调试的方法来帮助学习，这也是在传达我们“以调试之剑征服软件世界”的思想，提高大家学习调试技术的兴趣。
