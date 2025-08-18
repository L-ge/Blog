# 第10章　用户态调试过程

第9章介绍了Windows操作系统中用于支持用户态调试的各种基础设施，描述了这些设施的静态特征和功能。本章将讨论这些设施的动态特征，解析Windows系统中用户态调试的关键过程，特别是调试器、被调试程序及调试子系统这三者是如何相互配合完成各种调试功能的。我们先介绍调试器进程和被调试进程的基本特征（10.1节和10.2节），然后介绍建立调试会话的两种情况——从调试器中启动被调试程序（10.3节）和附加到已运行的进程（10.4节）。10.5节将介绍调试器处理调试事件的基本方法。10.6节介绍被调试进程中断到调试器的典型情况。10.7节介绍OutputDebugString API的工作原理及有关的工具。10.8节介绍调试过程的最后一个步骤，即调试会话的终止和分离。

因为本章讨论的是用户态调试，为了行文简洁，除非特别说明，本章提到的调试器就是指用户态调试器。

## 10.1　调试器进程

调试器进程和被调试进程是调试过程的两个主角。从用户的角度来看，调试过程就是使用调试器进程来控制和观察被调试进程的过程。本节和10.2节将简要描述调试过程中这两个主角的基本特征，以及它们是如何联系起来的。

调试器进程（debugger process）就是指运行着的调试器程序，或者说是调试器程序的运行实例（instance），比如运行着的MSDEV.EXE（VC6的IDE）、DEVENV.EXE（VS2005的IDE）或WinDBG.EXE等。

### 10.1.1　线程模型

可以把调试器的主要功能分成如下两个方面。

（1）人机接口，以某种界面的形式将调试功能呈现给用户，并监听和接收用户的输入（命令），在收到用户输入后进行解析和执行，然后把执行结果显示给用户。

（2）与被调试进程交互，包括与被调试进程建立调试关系，然后监听和处理调试事件，根据需要将被调试进程中断到调试器，读取和修改被调试进程的数据，或者操控它的其他行为。根据上一节的介绍，调试器主要是通过调试子系统与被调试进程交互的。但是从用户的角度来看，可以认为调试器是直接与被调试程序交互的。本章将使用这种粗略的描述方法。

总而言之，调试器进程一方面与用户对话，另一方面与被调试进程对话。为了及时地响应来自每一方面的对话请求，调试器通常会使用两个线程，每个线程负责一个方面的对话。负责与人（用户）对话的称为UI线程，负责与被调试进程对话的称为调试器工作线程（debugger’s worker thread）或调试会话线程，简称DWT。以WinDBG调试器为例，在它启动后，通常只有一个UI线程，即初始线程，在开始调试另一个程序后，那么它便会创建工作线程。清单10-1显示了这两个线程中的函数执行过程。

清单10-1　WinDBG调试器的UI线程和工作线程中的函数执行过程

```
0:001> ~* k
   0  Id: 1774.bd0 Suspend: 1 Teb: 7ffdf000 Unfrozen
ChildEBP RetAddr  
0006df24 7e419408 ntdll!KiFastSystemCallRet         //在内核模式执行系统服务
0006ff7c 0104f252 USER32!NtUserWaitMessage+0xc      //调用等待窗口消息的子系统服务
0006ffc0 7c816ff7 windbg!_wmainCRTStartup+0xfd      //程序的启动函数
0006fff0 00000000 kernel32!BaseProcessStart+0x23    //系统的进程启动函数
// 上面是UI线程，下面是工作线程
#  1  Id: 1774.1358 Suspend: 1 Teb: 7ffde000 Unfrozen
ChildEBP RetAddr  
00cefd04 02242d3e ntdll!ZwWaitForDebugEvent          //调用等待调试事件的内核服务
00cefda0 02107d23 dbgeng!LiveUserDebugServices::WaitForEvent+0x12e
00ceff10 020a3c3f dbgeng!LiveUserTargetInfo::WaitForEvent+0x3b3
00ceff34 020a401e dbgeng!WaitForAnyTarget+0x5f
00ceff80 020a4290 dbgeng!RawWaitForEvent+0x2ae       //调试引擎的内部函数
00ceff98 0102925f dbgeng!DebugClient::WaitForEvent+0xb0      //等待调试事件
00ceffb4 7c80b6a3 windbg!EngineLoop+0x13f            //调试事件循环
00ceffec 00000000 kernel32!BaseThreadStart+0x37      //线程的初始函数
```

可见，以上两个线程分别在等待用户输入和调试事件。

当然，并不是所有调试器都采用双线程模式，有些命令行界面的调试器只使用一个线程，同时负责以上两种对话，这一个线程频繁地在二者间“奔走”，哪一方需要对话便响应哪一方，在响应结束后再看另一方是否有对话请求。据笔者观察，命令行接口的NTSD调试器就是这样设计的。

包括WinDBG在内的大多数调试器使用的是双线程模式，所以本书如不特别说明，讨论的都是这种情况。需要指出的是，双线程模式并不代表调试器进程中只有这两个线程，因为在执行某些调试功能时调试器可能还会创建其他线程。

### 10.1.2　调试器的工作线程

下面我们来看调试器工作线程的主要逻辑。清单10-2显示了调试器工作线程的核心代码。

清单10-2　调试器工作线程的核心代码

```
1  //***********************************************************************
2  // Backbone of a Debugger’s Worker Thread (DWT)
3  // **********************************************************************
4      if(bNewProcess)
5          CreateProcess ( ..., DEBUG_PROCESS ,... );
6      else
7          DebugActiveProcess(dwPID)
8    
9      while ( 1 == WaitForDebugEvent (&DbgEvt, INFINITE) )
10     {
11         switch (DbgEvt.dwDebugEventCode)
12         {
13          case EXIT_PROCESS_DEBUG_EVENT:
14             break;
15          //other cases
16         }
17         ContinueDebugEvent ( ... ) ;
18     }
```

其中，第4～7行用于建立调试对话（后面两节将详细讨论），第9～18行是调试事件循环，类似于Windows程序的消息循环。第9行调用`WaitForDebugEvent`等待调试事件，当被调试程序中有调试事件发生时（线程创建退出、加载DLL或产生异常等），调试子系统便会通过`DEBUG_EVENT`结构通知调试器。接收到调试事件后，第11～16行会根据`DEBUG_EVENT`结构中的事件代码（`dwDebugEventCode`）来判断调试事件的类型，采取适当的动作，并根据需要中断给用户，让用户可以进行各种诊断和分析。在调试器处理好一个事件或接收到用户的恢复执行命令（如WinDBG的`g`命令）后，第17行会向调试子系统发送一个回复命令，典型的回复命令是`DBG_CONTINUE`，即恢复运行被调试程序。接下来，DWT开始等待下一个调试事件，如此往复，直到收到被调试进程退出的事件或发生其他终止情况（10.8节）为止。

### 10.1.3　DbgSsReserved字段

从线程的内部数据结构来看，调试器的UI线程与普通线程没什么特别，但是调试器工作线程与普通线程通常是有所不同的，具体来说，它的线程环境块（TEB）结构的`DbgSsReserved`字段通常与普通线程的取值不同。

第9章已零散地介绍过`DbgSsReserved`字段，简单来说，TEB结构的`DbgSsReserved[2]`数组就是专门用来记录调试器工作线程与调试子系统之间通信用的同步对象和通信对象的。在Windows XP之前，`DbgSsReserved[1]`记录的是LPC端口句柄（DbgUiApiPort），`DbgSsReserved[0]`记录的是等待调试消息的事件对象。

从Windows XP开始，`DbgSsReserved[1]`用来记录调试对象（DebugObject）句柄，`DbgSsReserved[0]`用来记录被调试线程链表的表头，这个链表的每个节点是一个`DBGSS_THREAD_DATA`结构，用来描述被调试进程中的一个线程。

```
typedef struct _DBGSS_THREAD_DATA 
{ 
struct _DBGSS_THREAD_DATA *Next;        //指向下一个节点
HANDLE ThreadHandle;                    //线程句柄（被调试进程中）
HANDLE ProcessHandle;                   //被调试进程的句柄
DWORD ProcessId;                        //被调试进程的ID
DWORD ThreadId;                         //线程ID（被调试进程中）
BOOLEAN HandleMarked;                   //退出标记
} DBGSS_THREAD_DATA, *PDBGSS_THREAD_DATA;
```

例如，以下是使用WinDBG观察MSDEV调试器的DMT所得到的结果。

```
0:011> dt -b ntdll!_Teb 7ffd4000 -y DbgSsReserved
ntdll!_TEB
   +0xf20 DbgSsReserved : 
    [00] 0x001ff878 
    [01] 0x000003b8
```

其中`0x000003b8`是调试对象的句柄，使用`!handle`命令可以确认这一点。

```
0:011> !handle 3b8
Handle 3b8
  Type            DebugObject
```

`0x001ff878`是被调试线程链表的头节点，调试API `WaitForDebugEvent`和`ContinueDebugEvent`会维护这个链表。

Windows XP的NTDLL.DLL中新引入的两个DbgUi函数`DbgUiGetThreadDebugObject`和`DbgUiSetThreadDebugObject`，实际上就是读写`DbgSsReserved[1]`字段的，例如：

```
ntdll!DbgUiGetThreadDebugObject:
7c9506de 64a118000000    mov     eax,dword ptr fs:[00000018h]
7c9506e4 8b80240f0000    mov     eax,dword ptr [eax+0F24h]
7c9506ea c3              ret
```

那么，为什么要使用`DbgSsReserved`数组，而不是把这些信息直接返回给调试器程序来管理呢？这主要是为了简化调试器的设计，使其不用保存和维护这些数据，也不用关心其中的细节。从调试API的函数原型也可以看到这一点，比如`WaitForDebugEvent` API只有两个参数，一个是用来接收调试事件的，另一个是等待的毫秒数。这个API内部会从当前线程的TEB结构中读取要等待对象的句柄。这也是必须在开始调试会话的线程中调用`WaitForDebugEvent` API的原因。

目前版本的WinDBG调试器在Windows XP系统中运行时不使用调试API（`KERNEL32.DLL`输出），而是直接调用NTDLL中的调试支持函数或系统的内核服务，并且自己保存和维护调试对象句柄与线程数据。因此，如果观察其调试器工作线程的TEB结构，那么可能看到`DbgSsReserved`数组的两个元素都为0。为什么说可能呢？这是因为与观察的时机有关，在调用CreateProcess创建调试会话时，CreateProcess内部调用的`DbgUiConnectToDbg`函数会将`DbgSsReserved[1]`设置为非0，但在CreateProcess返回后，WinDBG会调用`DbgUiSetThreadDebugObject`将其设置为0。

本节简要介绍了调试器进程的主要特征，特别是调试器工作线程的属性。最后要说明的一点是，操作系统并不区分调试器进程和普通的进程，一个调试器进程同时也可以被调试，成为被调试进程。笔者在写作此书时便经常启动几个调试器实例，一个调试另一个，以便了解调试器的工作原理。

## 10.2　被调试进程

被调试进程（debuggee process）泛指处于被调试状态的程序运行实例，比如运行在调试器下的控制台程序、窗口程序，或者Windows系统服务，等等。

### 10.2.1　特征

为了不影响问题的重现和分析结果，调试过程本身应该尽可能少地改变被调试进程的属性。也就是说，一个进程在被调试时与没有被调试时越相近越好。但为了实现某些调试功能，系统不得不修改被调试进程的某些属性或在其中执行一些用于调试的代码。概括地说，一个处于被调试状态的Windows进程与普通进程相比，会有如下差异。

（1）进程执行块（Executive Process Block，即EPROCESS结构）的`DebugPort`字段不为空。这是在内核空间中判断一个进程是否正在被调试的主要特征。

（2）进程环境块（Process Enviroment Block，PEB）的`BeingDebugged`字段不等于0。这是用户态判断一个进程是否正在被调试的主要方法。

（3）可能会存在一个由调试器远程启动的线程，这个线程的作用是将被调试进程中断到调试器，我们称之为远程中断线程（10.6.4节）。

（4）响应调试快捷键（F12键），按调试快捷键可以将处于被调试状态的进程中断到调试器，没有被调试的进程通常不响应调试快捷键。

下面我们将深入介绍`DebugPort`字段和`BeingDebugged`字段。10.6节将介绍远程中断线程和调试快捷键。

### 10.2.2　DebugPort字段

进程执行块是所有Windows进程都拥有的一个数据结构，名为EPROCESS。EPROCESS结构位于内核空间中，是系统用来标识和管理每个Windows进程的基本数据结构。EPROCESS结构的具体定义（字段个数和偏移位置）因Windows系统的版本不同而不同，但是都包含一个有关调试的重要字段，即`DebugPort`。如果一个进程不在被调试状态，那么`DebugPort`字段为NULL。如果一个进程在被调试状态，那么`DebugPort`字段是一个指针，指向的内容可能因Windows系统版本的不同而不同。在Windows XP之前，`DebugPort`字段保存着用于接收调试事件的LPC端口对象指针。当有调试事件发生时，系统会向这个端口发送调试消息。因为Windows XP使用了新的专门用于调试的内核对象DebugObject代替LPC端口，所以在Windows XP下， `DebugPort`字段指向的是DebugObject对象。不论是指向LPC端口还是指向调试对象，尽管类型不同，但是它们的作用都是用来传递调试事件的，因此我们将DebugPort中所指向的对象统称为调试端口。

从调试对话的角度来讲，调试端口是联系调试器进程和被调试进程的纽带。被调试进程中的调试事件就是由这个端口发送到调试器进程的。

### 10.2.3　BeingDebugged字段

进程环境块是每个Windows进程的另一个重要数据结构，与EPROCESS不同，PEB结构是位于用户空间中的，而且其地址通常位于用户空间的较高地址区域，例如0x7FFDF000。

```
typedef struct _PEB 
{
/*000*/ BOOLEAN InheritedAddressSpace;
/*001*/ BOOLEAN ReadImageFileExecOptions;
/*002*/ BOOLEAN BeingDebugged;
...
}PEB,*PPEB,**PPPEB;
```

如果一个进程不在被调试状态，那么其PEB结构的`BeingDebugged`字段为0；否则，为1。`IsDebuggerPresent ()` API就是通过判断`BeingDebugged`字段实现的。其汇编指令如下。

```
0:001> uf kernel32!IsDebuggerPresent
kernel32!IsDebuggerPresent:
77e7276b 64a118000000    mov  eax,fs:[00000018]     // 取得当前线程的TEB结构
77e72771 8b4030         mov  eax,[eax+0x30]         // 从TEB中取出PEB指针
77e72774 0fb64002  movzx  eax,byte ptr [eax+0x2]    // 取PEB中的BeingDebugged字段
77e72778 c3         ret
```

### 10.2.4　观察DebugPort字段和BeingDebugged字段

下面通过一个小实验来观察记事本程序在被调试前与被调试时`DebugPort`字段和`BeingDebugged`字段的变化。因为需要内核调试环境，而且Windows 2000不支持本地内核调试，所以我们以Windows XP为例。

先运行notepad程序，启动WinDBG，并开始本地内核调试（选择File→Kernel Debug→Local）。

在WinDBG的命令提示符（lkd）后输入“`!process 0 0`”，列出所有进程，找到关于notepad.exe进程的信息，并记录下它的EPROCESS结构的地址。

```
PROCESS 86f55648  SessionId: 0  Cid: 0eb8    Peb: 7ffdf000  ParentCid: 04d0
    DirBase: 2db71000  ObjectTable: e2a828a8  HandleCount:  38.
    Image: notepad.exe
```

然后使用`dt`命令显示notepad的EPROCESS结构的各个字段值。

```
lkd> dt nt!_EPROCESS 86f55648    // 省略了无关内容
   +0x0bc DebugPort          : (null) 
   +0x0c0 ExceptionPort      : 0xe29fa040
```

可见，此时`DebugPort`字段的值为空，说明该进程还没有被调试。ExceptionPort处已经有值，使用如下命令可以知道它是一个LPC端口的指针。

```
lkd> !lpc port 0xe29fa040

Server connection port e29fa040  Name: ApiPort    // 参见9.5.3节
    Handles: 1   References: 250    
    Server process     : 87122da8 (csrss.exe)    // CSRSS在监听此端口，参见11.3.3节
    Queue semaphore    : 8795d710
    Semaphore state 0 (0x0) 
    The message queue is empty
    The LpcDataInfoChainHead queue is empty
```

输入“`.process 86f55648`”命令，将notepad进程切换为默认进程。然后输入`!peb`命令观察`BeingDebugged`字段。

```
lkd>.process 86f55648    // 观察用户态的内存前，必须切换默认进程
Implicit process is now 86f55648
lkd> !peb
PEB at 7ffdf000    // 省略了无关显示
    BeingDebugged:            No
```

再运行一个WinDBG实例，附加到刚才的notepad进程，选择File→Attatch to a Process，然后输入notepad进程的ID或从列表中选取ID，如果系统中有多个notepad实例，请注意不要选错。

再次使用`dt`命令显示notepad的EPROCESS结构。

```
lkd> dt nt!_EPROCESS 86f55648        // 省略了无关内容
   +0x0bc DebugPort       : 0x87aa58c8 
   +0x0c0 ExceptionPort    : 0xe29fa040
```

可见，`DebugPort`字段的值已经不再为NULL了，使用`!object`命令可以观察到这是一个`DebugObject`对象。

```
lkd> !object 0x87aa58c8
Object: 87aa58c8  Type: (87baa040) DebugObject
    ObjectHeader: 87aa58b0
    HandleCount: 1  PointerCount: 2
```

对于Windows 2000，DebugPort记录的是LPC端口对象，因此可以通过`!lpc port xxxxxxxx`命令来观察。

重复前面观察PEB的步骤或在第二个WinDBG中输入`!peb`命令，可以观察到被调试进程的PEB中的`BeingDebugged`字段已经为Yes。

```
0:001> !peb        // 省略了无关显示
PEB at 7ffdf000
    InheritedAddressSpace:       No
    ReadImageFileExecOptions:    No
    BeingDebugged:               Yes
    ImageBaseAddress:            01000000
```

### 10.2.5　调试会话

我们把调试器进程与被调试进程之间的交互称为调试会话（debugging session）。一次调试会话从建立调试关系开始，直到这种关系解除为止。更准确地说，建立调试关系的标准是被调试进程与调试器进程之间通过调试端口建立的通信连接。调试关系解除的标准是被调试进程的调试端口被清除。

建立调试会话是调试过程的第一步，其建立方式包含两种情况。一种情况是在调试器中启动被调试程序（清单10-2的第4、5行），比如我们在VC6或VS2005（Visual Studio 2005）等集成开发环境（IDE）中开始执行程序，或者在WinDBG中打开一个EXE程序。另一种情况是当开始调试时，被调试程序已经在运行，因此需要把调试器附加（attach）到被调试进程上。后一种情况对于调试系统服务或DLL形式的各种插件非常有用。接下来的两节将分别介绍这两种情况。

## 10.3　从调试器中启动被调试程序

本节将介绍创建调试会话的第一种情况，也就是从调试器进程中创建被调试进程并开始调试。简单来说，这种方式就是当调用创建进程API（如CreateProcess）时指定DEBUG_PROCESS或DEBUG_ONLY_THIS_PROCESS标志。

### 10.3.1　CreateProcess API

Windows操作系统提供了几个API——如`CreateProcess()`、`CreateProcessAsUser()`、`CreateProcessWithTokenW ()`和`CreateProcessWithLogonW ()`用于创建新的进程。因为后面几个API可以看作CreateProcess API的超集，所以我们就以`CreateProcess() API`为例来讨论。它的函数原型如下。

```
BOOL CreateProcess( LPCTSTR lpApplicationName,  LPTSTR lpCommandLine,
  LPSECURITY_ATTRIBUTES lpProcessAttributes,
  LPSECURITY_ATTRIBUTES lpThreadAttributes,
  BOOL bInheritHandles,   DWORD dwCreationFlags,
  LPVOID lpEnvironment,   LPCTSTR lpCurrentDirectory,
  LPSTARTUPINFO lpStartupInfo,   LPPROCESS_INFORMATION lpProcessInformation);
```

其中 dwCreationFlags 参数用于指定创建新进程的选项，可以是一系列标志位的组合。以下两个标志位是专门用于调试的。

```
#define DEBUG_PROCESS              0x00000001      // 调试正在创建的进程和它的子进程
#define DEBUG_ONLY_THIS_PROCESS  0x00000002        // 声明不要调试调试目标的子进程
```

系统在创建进程时，会检查创建标志中是否包含以上标志。如果包含，那么系统会把调用进程当作调试器（debugger）进程，把新创建的进程当作被调试（debuggee）进程，为二者建立起调试关系，主要执行以下3个动作。

（1）在进程创建的早期（执行内核服务NtCreateProcess/NtCreateProcessEx之前），调用`DbgUiConnectToDbg()`使调用线程与调试子系统建立连接。在Windows XP以前，`DbgUiConnectToDbg()`内部会调用`NtConnectPort()`与会话管理器（SMSS.EXE）的\DbgUiApiPort连接，如果连接成功，便将返回的端口句柄和用于等待端口数据的信号量（semaphore）分别存放到当前线程环境块的`DbgSsReserved[1]`和`DbgSsReserved[0]`中。这两个指针有着重要作用，端口对象是与调试子系统进行通信的重要通道，信号对象用来等待调试事件，很多调试API（如`WaitForDebugEvent()`和`ContinueDebugEvent()`）需要这两个指针才能工作。在Windows XP中，`DbgUiConnectToDbg()`内部会调用`ZwCreateDebugObject`创建`DEBUG_OBJECT`内核对象，并将其保存在当前线程环境块的`DbgSsReserved[1]`字段中。需要说明的是，为了防止重复操作，`DbgUiConnectToDbg()`在调用`NtConnectPort`或`ZwCreateDebugObject`之前会先检查`DbgSsReserved[1]`是否为空。完成这一步后，调用线程便由普通线程晋升为调试子系统“眼”里的调试器工作线程了。清单10-3显示了WinDBG的调试器工作线程执行`CreateProcess`API和`DbgUiConnectToDbg`函数的过程。

清单10-3　执行CreateProcess API和DbgUiConnectToDbg函数的过程

```
0:001> k
ChildEBP RetAddr  
0140f198 7c842d54 ntdll!DbgUiConnectToDbg           [连接调试子系统]
0140fbc4 7c80235e kernel32!CreateProcessInternalW+0x12a2 [创建进程的内部函数]
0140fbfc 02241011 kernel32!CreateProcessW+0x2c      [创建进程API]
0140fcb8 020c8109 dbgeng!LiveUserDebugServices::CreateProcessW+0x241   
0140fcf8 0209667f dbgeng!LiveUserTargetInfo::StartCreateProcess+0xf9   
0140fd44 01028bf5 dbgeng!DebugClient::CreateProcessAndAttach2Wide+0xcf   
0140ffa4 0102913b windbg!StartSession+0x445         [开始调试会话]
0140ffb4 7c80b6a3 windbg!EngineLoop+0x1b            [调试事件循环]
0140ffec 00000000 kernel32!BaseThreadStart+0x37     [调试器工作线程的初始函数]
```

（2）当调用进程创建内核服务`NtCreateProcess`或`NtCreateProcessEx`时，将`DbgSsReserved[1]`字段中记录的对象句柄以参数（第7个参数）形式传递给内核中的进程管理器。接下来，内核中的进程创建函数（PspCreateProcess）会检查这个句柄是否为空，如果不为空，会取得它的对象指针，然后设置到进程执行块（EPROCESS结构）的`DebugPort`字段中。因为系统内部以DebugPort是否为0来判断一个进程是否在被调试，并向该端口发送调试消息，所以完成这一步后，新创建进程便由普通的进程晋升为调试子系统“眼”里的被调试进程了。

清单10-4显示了使用内核调试器观察到的WinDBG的调试器工作线程在内核模式中执行进程创建函数的过程。

清单10-4　在内核中执行进程创建函数的过程

```
kd> k
ChildEBP RetAddr  
f8740ce4 805909b2 nt!PspCreateProcess             [进程管理器中的进程创建函数]
f8740d38 804da140 nt!NtCreateProcessEx+0x7e       [内核服务]
f8740d38 7ffe0304 nt!KiSystemService+0xc4   
00c3f1c4 77f75a0f SharedUserData!SystemCallStub+0x4   
00c3f1c8 77e7fe05 ntdll!ZwCreateProcessEx+0xc     [调用内核服务]
00c3fbc4 77e61bb8 kernel32!CreateProcessInternalW+0x1111   
...    [省略用户模式的其他栈帧，参见清单10-2]
```

使用`dd`命令显示`PspCreateProcess`函数的栈帧和参数。

```
kd> dd f8740ce4
f8740ce4  00000286 805909b2 00c3f928 001f0fff
f8740cf4  00000000 ffffffff 00000006 0000020c
f8740d04  000001f4 00000000 00000000 f8740d64
...
```

其中参数000001f4是调试端口句柄，使用`!handle`观察到它对应的是WinDBG进程中的`DebugObject`对象。

```
kd> !handle 000001f4 
processor number 0, process 81e4ba58
PROCESS 81e4ba58  SessionId: 0  Cid: 07fc    Peb: 7ffdf000  ParentCid: 0710
    DirBase: 0be39000  ObjectTable: e1923490  HandleCount: 190.
    Image: windbg.exe

Handle table at e1954000 with 190 Entries in use
01f4: Object: 81e40740  GrantedAccess: 001f000f Entry: e19543e8
Object: 81e40740  Type: (81fc9778) DebugObject
    ObjectHeader: 81e40728 (old version)
        HandleCount: 1  PointerCount: 1
```

（3）当`PspCreateProcess`调用`MmCreatePeb`函数创建新的进程环境块时，MmCreatePeb函数内部会根据EPROCESS结构的`DebugPort`字段设置`BeingDebugged`字段。如果DebugPort不为空，那么BeingDebugged会被设置为真。

### 10.3.2　第一批调试事件

如果以上三步都成功结束（CreateProcess API成功返回），那么调试器与被调试程序的调试对话便建立起来了。调试器线程接下来应该进入调试事件循环来接收调试事件。

一个新创建进程的初始线程是从内核中的KiThreadStartup开始执行的，KiThreadStartup很简短，在将线程的IRQL降低到APC级别后，便将执行权交给了`PspUserThreadStartup`函数。

`PspUserThreadStartup`函数内部会调用DbgkCreateThread向调试子系统通知新线程创建事件。于是如我们在上一章所讲的，调试子系统会向调试事件队列中放入一个进程创建事件，并等待调试器来处理和回复。

结合本节的上下文，当调试器的工作线程等待调试事件时，它会立刻收到进程创建事件。随后，调试器会为调试这个新的进程做一些准备工作，而后它调用ContinueDebugEvent回复调试事件，被调试进程开始继续执行。

新进程会在自己的上下文中执行一系列初始化工作，包括映射和加载映像文件。因此，调试器接下来会收到一系列加载DLL的事件（LOAD_DLL_DEBUG_EVENT）。清单10-5显示了在WinDBG调试器中启动TinyDbge小程序时输出的信息。

清单10-5　在WinDBG调试器启动TinyDbge小程序时输出的信息（节选）

```
1    *** Create process 1470                       [创建进程成功，1470是进程ID]
2    Symbol search path is: SRV*d:\symbols*http://msdl.microsoft.com/download/symbols
3    Executable search path is:                    [未设置可执行文件搜索路径]
4    Process created: 1470.1f9c                    [收到进程创建事件]
5    OUTPUT_PROCESS: *** Create process ***        [进程创建事件触发的输出信息]
6    id: 1470  Handle: 314  index: 0
7      id: 1f9c  hThread: 334  index: 0  addr: 00401120
8    ModLoad: 00400000 0042c000   TinyDbge.exe     [收到模块加载事件]
9    OUTPUT_PROCESS: *** Load dll ***              [模块加载事件触发的信息输出]
10     hFile: 2f0  base: 00400000
11   ModLoad: 7c900000 7c9b0000   ntdll.dll        [映射NTDLL.DLL的事件]
12   OUTPUT_PROCESS: *** Load dll ***
13     hFile: 308  base: 7c900000
14   ModLoad: 7c800000 7c8f5000   C:\WINDOWS\system32\kernel32.dll
15   OUTPUT_PROCESS: *** Load dll ***
16     hFile: 2bc  base: 7c800000  
17   (1470.1f9c): Break instruction exception - code 80000003 (first chance)
18   eax=00241eb4 ebx=7ffd6000 ecx=0 edx=1 esi=00241f48 edi=00241eb4
19   eip=7c901230 esp=0012fb20 ebp=0012fc94 iopl=0    nv up ei pl nz na po nc
20   cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000       
21   efl=00000202
22   Loading symbols for 7c900000        ntdll.dll ->   ntdll.dll
23   ntdll!DbgBreakPoint:                        [发生异常的函数]
24   7c901230 cc              int     3          [触发断点异常的指令]
```

要看到清单10-5中的全部信息，需要按Ctrl+Alt+V组合键或选择View菜单中的Verbose Output让WinDBG“输出详细信息”。第1行显示调试器的工作线程成功调用CreateProcess API，新进程的ID是0x1470。第2、3行显示了当前的符号搜索路径和映像文件搜索路径。第4行显示接收到进程创建事件，实际上，也就是创建初始线程的事件，0x1f9c是初始线程的线程ID。第5～7行输出当前被调试进程的统计信息，下面每收到一个调试事件，会再显示一次（第9、10、12、13、15、16行），为了节约篇幅，我们删除了重复的信息行。

第8行代表WinDBG收到第一个模块加载（LOAD_DLL_DEBUG_EVENT）事件，即程序的EXE模块TinyDbge.exe，冒号后面的两个数字分别是模块的起始地址和结束地址。第11行和第14行分别表示收到映射NTDLL.DLL和KERNEL32.DLL事件。第17行表示收到初始断点异常事件，我们将在下一节讨论。

### 10.3.3　初始断点

当新进程的初始线程在自己的上下文中初始化时，作为进程初始化的一个步骤，NTDLL.DLL中的`LdrpInitializeProcess`函数会检查正在初始化的进程是否处于被调试状态（查询进程环境块的`BeingDebugged`字段）。如果是，它会调用`DbgBreakPoint()`触发一个断点异常，目的是中断到调试器。这实际上相当于系统在新进程中为我们设置了一个断点，这个断点通常称为初始断点。

清单10-5中的第17～24行便是调试器收到初始断点异常事件后所输出的信息。括号中是进程ID和线程ID。`0x80000003`是断点异常的编码，`firstchance`代表异常的第一轮处理机会（第11章）。第18～21行是命中断点时CPU中各个寄存器的值。从EIP指针的值（`eip=7c901230`）可以看出断点指令位于NTDLL.DLL模块中（EIP值介于NTDLL.DLL模块的起止地址7c900000和7c9b0000）。第22行显示调试器在为NTDLL.DLL加载符号文件，因为要寻找位于这个模块中的当前指令所对应的函数符号，即第23行所显示的信息。第24行是触发断点异常的指令地址、机器码和汇编语言表示。

执行栈回溯命令`k`可以看到初始线程调用DbgBreakPoint的完整过程。

```
0:000> k
ChildEBP RetAddr  
0012fb1c 7c93edc0 ntdll!DbgBreakPoint
0012fc94 7c921639 ntdll!LdrpInitializeProcess+0xffa
0012fd1c 7c90eac7 ntdll!_LdrpInitialize+0x183
00000000 00000000 ntdll!KiUserApcDispatcher+0x7
```

可以看出是NTDLL.DLL中的LdrpInitializeProcess调用了DbgBreakPoint。Ldr是Loader的缩写，是NTDLL.DLL中的进程加载器系列函数的前缀。

当命中初始断点时，被调试程序自己的主函数还没开始执行，因此这个调试时间是很早的，对于调试在程序初始化阶段发生的问题是非常有意义的。

包括MSDEV调试器在内的一些调试器不会将初始断点报告给用户，WinDBG默认会向用户报告初始断点，但是可以通过命令行参数-g来忽略初始断点，即收到初始断点后立刻恢复执行，而不是停下来。

### 10.3.4　自动启动调试器

有时，要调试的进程可能是被系统或其他进程动态启动的。在调试器中执行这类进程可能无法提供合适的参数和运行条件。另外，当我们发现这类进程启动并将调试器附加到该进程中时，需要调试的代码可能已经运行结束了。对于这种情况，可以通过在注册表中设置“映像文件执行选项（Image File Execution Options）”来让操作系统先启动调试器，然后再从调试器中启动目标进程。

Windows系统在创建进程时，会在注册表中查询如下表键来读取关于这个程序的执行选项。

```
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options
```

查询的方法就是看以上表键下是否存在以要执行的映像文件名（不包含路径）命名的子键。如果有这样的子键，那么系统会继续查询该子键是否存在名为“Debugger”的键值。

如果Debugger键值存在而且包含内容，那么系统便会先将当前的命令行附加到Debugger键值所定义的内容之后，再将此作为新的命令行来启动。

举例来说，我们先在Image File Execution Options表键下建立一个名为calc.exe（计算器程序）的子键，并在其下加入Debugger键值，内容如下。

```
C:\WinDBG\WinDBG.exe –g
```

也就是WinDBG调试器的主程序的完整路径，加上-g开关，含义是忽略初始断点，让被调试进程启动后就继续运行，而不是直接中断到调试器。如果希望新进程启动后就中断到调试器，那么去掉-g开关。

完成以上修改后，再以某种方式（通过“开始”菜单或运行calc.exe）启动计算器程序，就会发现系统会自动启动WinDBG，因为系统将启动calc.exe的完整路径以命令行参数的形式传给WinDBG，所以WinDBG启动后会立即启动计算器程序。

如果Debugger键值指定的调试器失败，比如我们将刚才的WinDBG路径改为错误的值，那么系统仍会显示错误消息，指出无法发现原来的映像文件。

讲到这里，大家可能会想到如下一个有趣的问题，如果为Debugger键值中定义的调试器映像再定义Debugger执行选项会怎么样呢？比如我们再加入一个名为WinDBG.exe的子键，并加入Debugger键值，其内容为vsjitdebugger.exe（Visual Studio .NET 2003的JIT调试器）。完成这一修改后，再次运行计算器程序，我们得到原书P206的图10-3所示的对话框。尽管其中给出了错误提示，但仔细观察提示中的命令行，可以发现系统是在按我们估计的递归方式工作，启动calc.exe时根据执行选项先启动windbg.exe，启动windbg.exe时执行选项又要先启动vsjitdebugger.exe。但vsjitdebugger.exe出错了，于是就“搁置”在这里了。这颇有点“螳螂捕蝉，黄雀在后”的意味。

根据以上逻辑，如果Debugger键值指定的仍然是同名的一个程序，那么会出现“死循环”。笔者实验的结果是，系统忙碌了数秒后，会出现类似于原书P206的图10-2的对话框。

## 10.4　附加到已经启动的进程中

本节介绍建立调试会话的另一种情况，也就是当要调试的程序已经运行时是如何建立调试对话的，即如何将调试器附加（attach）到已经运行的进程中。简单来说，这是通过`DebugActiveProcess` API来完成的。

### 10.4.1　DebugActiveProcess API

`DebugActiveProcess` API的原型非常简单，即BOOL DebugActiveProcess（DWORD ）。也就是只将被调试进程的ID传递给`DebugActiveProcess` API，系统便会为这个API的进程与`dwProcessId`参数指定的进程建立起调试关系。下面以Windows XP系统中的情况为例，介绍`DebugActiveProcess` API的内部工作过程。

（1）通过`DbgUiConnectToDbg()`使调用进程与调试子系统建立连接，实质上就是获得一个调试通信对象并将它存放在当前TEB结构的`DbgSsReserved`数组中。这与调试一个新进程的第一步相同。

（2）调用`ProcessIdToHandle`函数，获得指定ID的进程句柄，这个函数内部会调用`OpenProcess` API，进而调用NtOpenProcess内核服务。在执行这一步时需要调用进程，它与目标进程有同样或更高的权限，否则这一步便会失败，调用`GetLastError()`返回的错误码通常是5，意思是“Access is denied”，即访问被拒绝。如果调试器进程具有SE_DEBUG_NAME权限，那么它通常有权限调试系统内的任何进程。

（3）调用NTDLL.DLL中的`DbgUiDebugActiveProcess`函数。这个函数内部主要调用NtDebugActiveProcess内核服务，并将要调试进程的句柄（参数1）和调试对象的句柄（参数2）作为参数传递给这个内核服务。NtDebugActiveProcess内部主要执行以下3个动作：根据参数中指定的句柄取得被调试进程的EPROCESS结构和调试对象的对象指针；向调试对象发送杜撰的调试事件（9.4.5节）；调用`DbgkpSetProcessDebugObject`函数，这个函数内部会将调试对象设置到被调试进程的调试端口（`DebugPort`字段），并调用`DbgkpMarkProcessPeb`来设置`BeingDebugged`字段。清单10-6显示了一个名为TinyDbgr的调试器调用`DebugActiveProcess` API和执行`NtDebugActiveProcess`函数的过程。

清单10-6　TinyDbgr调用DebugActiveProcess API和执行NtDebugActiveProcess函数的过程

```
kd> knL
 # ChildEBP RetAddr  
00 f4b92cfc 805bbb48 nt!DbgkpMarkProcessPeb                // 标记PEB
01 f4b92d2c 805bcf39 nt!DbgkpSetProcessDebugObject+0x222   // 设置调试端口
02 f4b92d54 804da140 nt!NtDebugActiveProcess+0x9a          // 系统服务
03 f4b92d54 7ffe0304 nt!KiSystemService+0xc4               // 系统服务分发函数
04 0012fdf8 77f75a96 SharedUserData!SystemCallStub+0x4
05 0012fdfc 77f8f4db ntdll!ZwDebugActiveProcess+0xc        // NTDLL.DLL中的系统服务残根
06 0012fe10 77eaeeae ntdll!DbgUiDebugActiveProcess+0x18    // NTDLL.DLL中的DbgUi函数
07 0012fe20 0040149a kernel32!DebugActiveProcess+0x2e      // 附加到已运行进程的API
08 0012ff80 00401f69 TinyDbgr!main+0xfa                    // 主函数
09 0012ffc0 77e814c7 TinyDbgr!mainCRTStartup+0xe9          // 编译器插入的启动函数
0a 0012fff0 00000000 kernel32!BaseProcessStart+0x23        // 线程启动函数
```

在`NtDebugActiveProcess`成功返回后，`DbgUiDebugActiveProcess`会调用`DbgUiIssueRemoteBreakin`，目的是在远程进程中创建远程中断线程（10.6.4节），使被调试进程中断到调试器中。

以上操作都成功后，`DebugActiveProcess()`会返回真，通知调用进程已经成功建立调试对话。接下来调试器便进入调试事件循环，开始接收和处理调试事件了。它首先会接收到一系列杜撰的调试事件，包括进程创建、模块加载等事件。然后收到远程中断线程产生的断点事件。调试器收到这一事件后，通常会停下来报告给用户。

`DebugActiveProcess API`在Windows 2000中的工作过程与上面介绍的类似，较大的差异是`DbgUiDebugActiveProcess`函数，它内部的主要操作不再是调用系统服务，而是调用CSRSS的CsrDebugProcess服务（9.5.6节）。

至此，我们已经比较详细地介绍了目标程序已经运行和还没有运行两种情况下调试对话的建立过程。下面通过一个演示程序来进一步理解这些内容。

### 10.4.2　示例：TinyDbgr程序

为了演示用户态调试的工作原理，我们使用C++语言编写了一个小型的Windows调试器程序，名为TinyDbgr，意思是微型调试器。清单10-7给出了该程序的主函数和Help函数的源代码。

清单10-7　TinyDbgr程序的主函数和Help函数的源代码

```
void Help()
{
   printf ( "TinyDbgr <PID of Program to Debug>|\n    "
             "<Full Exe File Name> [Prgram Parameters]\n" ) ;
}
int main(int argc, char* argv[])
{
   if(argc<=1) 
   {
      Help();   return -1;
   }
   if (strstr(strupr(argv[1]),".EXE"))
   {
      TCHAR szCmdLine[ MAX_PATH ] ;
      szCmdLine[ 0 ] = '\0' ;

      for ( int i = 1 ; i < argc ; i++ )
      {   
         strcat ( szCmdLine , argv[ i ] ) ;
         if ( i < argc )
         {
            strcat ( szCmdLine , " " ) ;
         }
      }
      if(!DbgNewProcess(szCmdLine))
      {
         return -2;
      }
   }
   else
      if(!DebugActiveProcess(atoi(argv[1])))
      {
         printf("Failed in DebugActiveProcess() with %d.\n",GetLastError());
         return -2;
      }

   return DbgMainLoop();
}
```

从以上代码可以看出，TinyDbgr至少需要一个命令行参数，如果没有，就会显示帮助信息。

TinyDbgr支持两种参数，分别适用于调试已经运行的程序和尚未运行的程序。如果TinyDbgr发现第一个参数中不包含“.EXE”，那么便认为要调试一个已经运行的进程，该参数是要调试进程的ID（十进制整数）；否则，TinyDbgr便会把所有参数当作一个命令行，并试图通过该命令行启动和调试该程序。

让我们先就第一种情况做个实验。启动计算器程序（在“开始”菜单选择“运行”，在“运行”对话框中输入“calc”后按Enter键）。使用任务管理器找到计算器程序的进程ID，当笔者做实验时进程ID为4752。打开控制台窗口，并转到本书附带代码的bin\debug目录，然后输入“tinydbgr 4752”（4752应该换作具体的进程ID）。接下来应该看到TinyDbgr迅速打印出如下结果。

```
Debug event received from process 4752 thread 3632: CREATE_PROCESS_DEBUG_EVENT.
Debug event received from process 4752 thread 3632: LOAD_DLL_DEBUG_EVENT.
[省略多行与上面一行一样的LOAD_DLL_DEBUG_EVENT事件]
Debug event received from process 4752 thread 2672: CREATE_THREAD_DEBUG_EVENT.
Debug event received from process 4752 thread 2672: EXCEPTION_DEBUG_EVENT.
-Debuggee breaks into debugger; press any key to continue.
```

第一行是创建进程的消息，正如我们前面所介绍的，尽管该进程早已经创建，但是调试子系统会模拟出这个以前发生的事件以帮助调试器补充历史信息。接下来是一系列加载或映射DLL的事件，这些也是调试子系统模拟出的历史事件。倒数第三行的CREATE_THREAD_DEBUG_EVENT事件来自一个新的线程，即DbgUiIssueRemoteBreakin所创建的远程线程。最后一行是远程中断线程触发的断点异常（EXCEPTION_DEBUG_EVENT）。

此时试图操作计算器程序，发现无法将其激活，因为它已经中断到调试器中了，也就是被调试子系统挂起了。按任意键让调试器恢复，并让被调试程序继续运行，显示如下内容。

```
Debug event received from process 4752 thread 2672: EXIT_THREAD_DEBUG_EVENT.
```

因为远程中断线程退出了，所以计算器程序可以使用了。

退出计算器程序，显示如下内容。

```
Debug event received from process 4752 thread 3632: EXIT_PROCESS_DEBUG_EVENT.
```

即被调试进程退出了。

下面再根据另一种情况做实验，输入“tinydbgr calc.exe”让TinyDbgr启动一个新的计算器进程并开始调试，显示的结果如下。

```
Debug event received from process 1584 thread 5196: CREATE_PROCESS_DEBUG_EVENT.
Debug event received from process 1584 thread 5196: LOAD_DLL_DEBUG_EVENT.
[省略8行与上面一行一样的LOAD_DLL_DEBUG_EVENT事件]
Debug event received from process 1584 thread 5196: EXCEPTION_DEBUG_EVENT.
-Debuggee breaks into debugger, press any key to continue.
```

这便是10.3.3节介绍的因为`LdrpInitializeProcess`函数调用DbgBreakPoint而产生的初始断点。

按任意键恢复程序运行，会看到类似如下的输出。

```
Debug event received from process 1584 thread 5196: LOAD_DLL_DEBUG_EVENT.
[省略7行与上面一行一样的LOAD_DLL_DEBUG_EVENT事件]
Debug event received from process 1584 thread 5196: UNLOAD_DLL_DEBUG_EVENT.
```

这一行是反映射（卸载）DLL的消息，`FreeLibrary()`可能会导致此动作。

与已经运行程序建立调试对话的一个典型的应用，就是在一个程序发生错误（未处理异常）并即将被系统终止时，将调试器附加到这个进程中，即所谓的JIT调试，这将在第12章详细讨论。

## 10.5　处理调试事件

第9章介绍Windows调试模型的基本特征时（9.1.2节），我们说过Windows系统的用户态调试是通过调试事件来驱动的，而后介绍了调试事件（消息）的采集和传递过程。本节将介绍调试器是如何读取和处理调试事件的。

### 10.5.1　DEBUG_EVENT 结构

在调试API一层，Windows使用名为DEBUG_EVENT的结构来表示调试事件，该结构的定义如下。

```
typedef struct _DEBUG_EVENT {
  DWORD dwDebugEventCode;                          //事件代码
  DWORD dwProcessId;                               //发生调试事件进程的ID
  DWORD dwThreadId;                                //发生调试事件线程的ID
  union {                                          //联合体，用于记录事件的详细信息
    EXCEPTION_DEBUG_INFO Exception;                //异常事件的详细信息
    CREATE_THREAD_DEBUG_INFO CreateThread;          //线程创建事件的详细信息
    CREATE_PROCESS_DEBUG_INFO CreateProcessInfo;    //进程创建事件的详细信息
    EXIT_THREAD_DEBUG_INFO ExitThread;              //线程退出事件的详细信息
    EXIT_PROCESS_DEBUG_INFO ExitProcess;            //进程退出事件的详细信息
    LOAD_DLL_DEBUG_INFO LoadDll;                    //映射DLL事件的详细信息
    UNLOAD_DLL_DEBUG_INFO UnloadDll;                //反映射DLL事件的详细信息
    OUTPUT_DEBUG_STRING_INFO DebugString;           //输出调试字符串事件的详细信息
    RIP_INFO RipInfo;                               //内部错误事件的详细信息
  } u;
} DEBUG_EVENT, *LPDEBUG_EVENT;
```

其中`dwDebugEventCode`用来标识调试事件的类型，其值是表10-1所示的9种调试事件类型常量中的一个。dwProcessId为发生调试事件进程的ID，当调试器同时调试多个进程时，可以使用该ID来区分调试事件来自哪个进程。dwThreadId为发生调试事件线程的ID，该线程属于dwProcessId所指定的进程。接下来是一个联合体（union），定义了描述9种事件详细信息的9个结构。因为使用了联合结构，所以应该根据`dwDebugEventCode`决定实际包含的是哪个结构，其对应关系如表10-1的第4列所示。

表10-1　调试类型常量

| 事件类型（dwDebugEventCode） | 值 | 说　　明 | 详细信息所使用的结构 |
| --- | --- | --- | --- |
| EXCEPTION_DEBUG_EVENT | 1 | 异常 | EXCEPTION_DEBUG_INFO |
| CREATE_THREAD_DEBUG_EVENT | 2 | 创建线程 | CREATE_THREAD_DEBUG_INFO |
| CREATE_PROCESS_DEBUG_EVENT | 3 | 创建进程 | CREATE_PROCESS_DEBUG_INFO |
| EXIT_THREAD_DEBUG_EVENT | 4 | 线程退出 | EXIT_THREAD_DEBUG_INFO |
| EXIT_PROCESS_DEBUG_EVENT | 5 | 进程退出 | EXIT_PROCESS_DEBUG_INFO |
| LOAD_DLL_DEBUG_EVENT | 6 | 映射DLL | LOAD_DLL_DEBUG_INFO |
| UNLOAD_DLL_DEBUG_EVENT | 7 | 反映射DLL | UNLOAD_DLL_DEBUG_INFO |
| OUTPUT_DEBUG_STRING_EVENT | 8 | 输出调试信息 | OUTPUT_DEBUG_STRING_INFO |
| RIP_EVENT | 9 | 内部错误 | RIP_INFO |

第4列中的结构在SDK中有详细的定义和说明，本书从略。

### 10.5.2　WaitForDebugEvent API

Windows系统设计了WaitForDebugEvent API来供调试器等待和接收调试事件。这个API是实现在KERNEL32.DLL中的。

```
BOOL WaitForDebugEvent(LPDEBUG_EVENT lpDebugEvent, DWORD dwMilliSeconds);
```

第一个参数是一个指向`DEBUG_EVENT`结构的指针，用来保存收到的调试事件。第二个参数用来指定要等待的毫秒数，或者使用INFINITE常量（0xFFFFFFFF），意思是无限期等待。

调用`WaitForDebugEvent()`会导致所在线程阻塞，直到有调试事件发生，或等待时间已到或发生错误才返回。这也是大多数调试器使用多线程的原因，可使用其他线程处理UI更新和用户对话。

`WaitForDebugEvent`内部主要完成两项任务，一是调用NTDLL.DLL中的`DbgUiWaitStateChange`函数，二是将这个函数返回的以`DBGUI_WAIT_STATE_CHANGE`结构表示的调试事件转化为`DEBUG_EVENT`结构。从Windows XP开始，转化工作是调用NTDLL.DLL中新增的`DbgUiConvertStateChangeStructure`函数来完成的。

`DBGUI_WAIT_STATE_CHANGE`结构的定义如下。

```
typedef struct _DBGUI_WAIT_STATE_CHANGE {
    DBG_STATE NewState;           // 枚举常量，代表新的调试状态
    CLIENT_ID AppClientId;        //结构，包含进程和线程句柄
    union {                       //描述详细信息的联合体
        DBGKM_EXCEPTION Exception;                  //异常
        DBGUI_CREATE_THREAD CreateThread;           //创建线程
        DBGUI_CREATE_PROCESS CreateProcessInfo;     //创建进程
        DBGKM_EXIT_THREAD ExitThread;         //线程退出
        DBGKM_EXIT_PROCESS ExitProcess;       //进程退出
        DBGKM_LOAD_DLL LoadDll;               //映射模块
        DBGKM_UNLOAD_DLL UnloadDll;           //反映射模块
    } StateInfo;
} DBGUI_WAIT_STATE_CHANGE, *PDBGUI_WAIT_STATE_CHANGE;
```

其中`NewState`是一个枚举常量，其定义如下。

```
typedef enum _DBG_STATE {
DbgIdle,    DbgReplyPending, DbgCreateThreadStateChange,    
DbgCreateProcessStateChange, DbgExitThreadStateChange, DbgExitProcessStateChange,  
DbgExceptionStateChange, DbgBreakpointStateChange, DbgSingleStepStateChange,
DbgLoadDllStateChange, DbgUnloadDllStateChange
} DBG_STATE, *PDBG_STATE;
```

`DBGUI_WAIT_STATE_CHANGE`结构名中的DBGUI代表这是调试子系统向外提供的接口结构。在内核调试中，一个和它作用类似的结构叫`DBGKD_WAIT_STATE_CHANGE`，我们将在第18章介绍。相对而言，`DEBUG_EVENT`是更高一层的结构，即用户API一层。

第9章介绍过，内核中是使用`DBGKM_APIMSG`结构来描述调试事件的，调试API一层是使用`DEBUG_EVENT`结构来描述的，而DbgUi函数是使用`DBGUI_WAIT_STATE_CHANGE`结构的。下面显示了这些结构的使用场合和转换函数。

```
DBGKM_APIMSG（内核）---（DbgkpConvertKernelToUserStateChange()）---> DBGUI_WAIT_STATE_CHANGE（DbgUI） --- （DbgUiConvertStateChangeStructure()） ---> DEBUG_EVENT（调试API）
```

### 10.5.3　调试事件循环

在对调试事件有了比较深入的了解之后，接下来的问题是调试器应该如何接收和处理调试事件，这是设计调试器的一个主要工作。先让我们看一下TinyDbgr程序的简单调试事件循环（清单10-8）。

清单10-8　TinyDbgr程序的简单调试事件循环

```
1    BOOL DbgMainLoop(DWORD dwWaitMS)
2    {
3       DEBUG_EVENT DbgEvt;          // 用来读取调试事件的数据结构 
4       DWORD dwContinueStatus = DBG_CONTINUE; // 恢复继续执行用的状态代码 
5       BOOL bExit=FALSE;            // 退出标志
6   
7       while(!bExit) 
8       {
9          // 等待调试事件发生，第2个参数是等待的毫秒数，如果指定为INFINITE，那么直到 
10         // 有调试事件发生时这个函数才返回
11         if(!WaitForDebugEvent(&DbgEvt, dwWaitMS))
12         {
13            printf("WaitForDebugEvent() returned False %d.\n",GetLastError());
14            bExit=TRUE;
15            continue;
16         }
17    
18         // 以下是处理调试事件的代码
19         printf("Debug event received from process %d thread %d: %s.\n",
20            DbgEvt.dwProcessId,   DbgEvt.dwThreadId,
21            DbgEventName[DbgEvt.dwDebugEventCode>
22               MAX_DBG_EVENT?MAX_DBG_EVENT:
23               DbgEvt.dwDebugEventCode-1]); 
24         switch (DbgEvt.dwDebugEventCode) 
25         { 
26            case EXCEPTION_DEBUG_EVENT: 
27            // 处理异常事件，需要设置继续参数(dwContinueStatus)，ContinueDebugEvent
28            // 函数需要这个参数 
29               printf("-Debuggee breaks into debugger; press any key to   
                continue.\n");
30               getchar();
31               switch (DbgEvt.u.Exception.ExceptionRecord.ExceptionCode) 
32               { 
33                  case EXCEPTION_ACCESS_VIOLATION: 
34                  // 访问违例，对第一轮机会不处理，对最后一轮机会显示错误信息 
35                     break;
36                   case EXCEPTION_BREAKPOINT: 
37                  // 断点异常，对第一轮机会显示当前指令和寄存器值
38                     break;
39                   case EXCEPTION_DATATYPE_MISALIGNMENT: 
40                  // 内存对齐异常，对第一轮机会不处理，对最后一轮机会显示错误信息
41                     break;
42                   case EXCEPTION_SINGLE_STEP: 
43                  // 单步和硬件断点异常，对第一轮机会显示当前指令和寄存器值 
44                     break;
45                   case DBG_CONTROL_C: 
46                  // Ctrl+C，对第一轮机会不处理，对最后一轮机会显示错误 
47                     break;
48                   default:
49                  // 处理错误
50                     break;
51               } 
52             case CREATE_THREAD_DEBUG_EVENT: //线程创建事件
53            // 根据需要可以调用GetThreadContext和SetThreadContext 
54            // API来分析和设置寄存器，调用SuspendThread and ResumeThread API来
55            // 挂起和恢复线程 
56               break;
57            case CREATE_PROCESS_DEBUG_EVENT: //进程创建事件
58            // 根据需要可以调用GetThreadContext和SetThreadContext 
59            // API来访问进程的初始线程，调用ReadProcessMemory和
60            // WriteProcessMemory API来访问内存
61               break;
62            case EXIT_THREAD_DEBUG_EVENT: //线程退出事件
63            // 显示线程的退出代码 
64               break;
65            case EXIT_PROCESS_DEBUG_EVENT: //进程退出事件
66            // 显示进程的退出代码 
67               bExit=TRUE;
68               break;
69            case LOAD_DLL_DEBUG_EVENT: // 加载模块事件
70               break;
71            case UNLOAD_DLL_DEBUG_EVENT: // 模块卸载事件
72            // 显示信息 
73               break;
74            case OUTPUT_DEBUG_STRING_EVENT: 
75            // 从被调试进程读取调试信息字符串，参见10.7节
76               break;
77         } 
78         // 恢复被调试进程运行 
79         ContinueDebugEvent(DbgEvt.dwProcessId, 
80            DbgEvt.dwThreadId, dwContinueStatus); 
81       }
82      return TRUE;
83   }
```

清单10-8中的`DbgMainLoop`函数演示了如何等待和接收调试事件，其代码主要来自MSDN中关于编写调试器主循环（Writing the Debugger’s Main Loop）的示例。尽管它用了80多行代码（包括注释），但是大家可以看到它还没有真正处理各个调试事件，它只是简单地打印出每个调试事件的主要信息（供我们试验使用）。对于一个事件，调试器通常有以下几种处理方式。

（1）什么也不做或只是更新内部状态，用户察觉不到有调试事件发生。比如对C++异常的第一轮处理机会，调试器的默认行为是什么都不做，让系统继续分发异常。

（2）中断给用户，开始交互式调试，直到用户发出继续运行的命令（按F5键或输入g等）。例如，当接收到断点异常时，调试器总是会中断给用户。

（3）输出提示信息。

在处理调试事件后，调试器会调用`ContinueDebugEvent` API来回复调试事件，并让被调试程序继续运行（第79行和第80行）。

### 10.5.4　回复调试事件

调试器在处理好调试事件后，应该调用`ContinueDebugEvent` API来向调试子系统回复处理结果。

```
BOOL ContinueDebugEvent( DWORD dwProcessId, DWORD dwThreadId,
  DWORD dwContinueStatus);
```

`dwProcessId`和`dwThreadId`即收到的调试事件（DEBUG_EVENT）中包含的进程ID与线程ID。`dwContinueStatus`可以为DBG_CONTINUE（0x00010002L）和DBG_EXCEPTION_NOT_HANDLED（0x00010001L）两个常量之一。对于异常事件（EXCEPTION_DEBUG_EVENT）之外的其他所有事件，这两个常量没有差异，调试子系统收到后，都会恢复运行被调试进程（调用DbgkpResumeProcess）。对于异常事件，其差异如下。

DBG_CONTINUE表示调试器处理了该异常。`DbgkForwardException`函数（异常事件便是该函数接收并传给调试子系统的）收到此返回值后会向它的调用者（KiDispatchException）返回真，KiDispatchException看到`DbgkForwardException`函数返回真后，便知道调试器处理了该异常，于是结束对该异常的分发过程。

DBG_EXCEPTION_NOT_HANDLED表示调试器不处理该异常。该回复会导致DbgkForwardException返回假给KiDispatchException。我们前面提到过，KiDispatchException会多次调用DbgkForwardException，分别为了不同轮次的处理机会。对于第一轮处理机会，KiDispatchException在获悉DbgkForwardException返回假后，会继续分发过程，通常寻找异常处理块（第11章）。对于第二轮处理机会，KiDispatchException获悉DbgkForwardException返回假后，会再次调用DbgkForwardException发给异常端口（exception port）。如果这次调用也返回假，则终止该进程，但这通常不会发生。因为对于第二轮处理机会，调试器默认会返回DBG_CONTINUE，也就是“假装”处理了。这会导致异常分发过程结束，产生异常的代码被再次执行，于是又发生异常，如此反复不断。另外，即使调试器对于第二轮机会返回DBG_EXCEPTION_NOT_HANDLED，因为异常端口指定的通常是Windows子系统服务器进程（CSRSS）监听的Windows\ApiPort端口，CSRSS收到此消息后，就会强行终止该进程（end program），并向KiDispatchException回复真。

### 10.5.5　定制调试器的事件处理方式

大多调试器允许调试人员随时修改对每个调试事件的处理和回复方式。比如，当使用VC6调试时，其Debug菜单中便有一个Exceptions项。

对于Exceptions列表框中的每种异常，VC6允许有如下两种选项。

（1）Stop always。该项表示一旦接收到该异常，不论是第一轮处理机会还是第二轮处理机会，都中断（停止）到调试器。如果是第一轮处理机会，当用户按下F5键（让被调试进程继续运行）时，VC6会弹出“如何回复第一轮处理机会”的询问对话框。选择No，则VC6回复DBG_CONTINUE（ContinueDebugEvent），声明已经处理该异常；选择Yes，则VC6回复DBG_EXCEPTION_NOT_HANDLED，也就是自己没有处理该异常，让系统继续分发异常（传递给应用程序）。如果是第二轮处理机会，当用户按下F5键时，VC6不会询问，直接返回DBG_CONTINUE。

（2）Stop if not handled。该项表示对于第一轮处理机会直接返回DBG_EXCEPTION_NOT_HANDLED。对于第二次处理机会才中断到调试器。直接返回的含义是VC6收到此异常会不做任何处理就直接返回DBG_EXCEPTION_NOT_HANDLED，让系统继续分发此异常。因为只有当没有异常处理块处理一个异常时，才会有第二轮分发，这也正是这个选项如此命名的原因，即“如果没有被（应用程序的异常处理块）处理才停止到调试器”（Stop if not handled）。

对于大多数异常，VC6的默认设置是Stop if not handled，即当得到第二轮处理机会时才停止到调试器。但断点事件和单步执行事件除外。

可以使用清单10-9所示的小程序来加深对以上内容的理解。使用VC6打开EvtFilter项目（code\chap10\evtfilter），开始调试（选择Build→Start Debug→Go），在运行到第8行等待输入时，通过Debug→Exception菜单打开Exceptions对话框，从Exceptions列表框中选择Integer Divide by Zero，然后将Action切换到Stop Always，再单击Change按钮。确认列表框中的Integer Divide by Zero的处理方式已经是Stop Always后，单击OK按钮关闭对话框。在运行EvtFilter程序的控制台窗口中按任意键，使其继续。

清单10-9　用来试验调试器异常处理方式的小程序

```
1    #include "stdafx.h"
2    #include <windows.h>
3   
4    int main(int argc, char* argv[])
5    {
6       int i,m=1,n;
7       printf("Hello World!\n");
8       getchar();
9      
10      __try
11      {
12         n=0;
13         i=m/n;
14      }
15      __except(EXCEPTION_EXECUTE_HANDLER)
16      {
17         OutputDebugString("I got the exception.\n");
18         return -1;
19      }
20      return 0;
21   }
```

运行到第13行时，因为除数是0，所以一定会发生异常。由于是在调试器中运行，因此该异常会被发送给调试器。VC6收到异常事件后，根据和Exception结构中的异常代码（DbgEvt.u.Exception.ExceptionRecord.ExceptionCode）可以判断出是整数除以0异常。然后VC6在项目的配置信息中查询该异常的处理方式，在发现是Stop Always后，它会准备中断给用户，因为在第一轮处理机会就中断给用户，所以VC6会显示“这是第一轮处理机会”的对话框。

单击OK按钮后，VC6会显示出当前的代码（第13行）。这时，若不做任何改动直接按F5键继续，则VC6会弹出“如何回复第一轮处理机会”的对话框，询问返回DBG_CONTINUE（如果选No）还是DBG_EXCEPTION_NOT_HANDLED（如果选Yes）。返回DBG_CONTINUE会导致异常代码重新执行，于是再次发生异常，VC6再次显示“这是第一轮处理机会”的对话框。返回DBG_EXCEPTION_NOT_HANDLED会导致系统继续分发该异常，第17～18行的异常处理块被执行。

在第一轮处理机会，如果在VC6中将变量n改为1（或其他非零值）（通过变量观察窗口），然后按F5键，并对“如何回复第一轮处理机会”的对话框选No（已经处理了异常情况，不必继续分发异常），那么第13行也会重新执行，但是此时导致异常的错误情况已经消除，不会再发生异常，程序便顺利执行而正常退出了。

WinDBG允许用户定制包括异常事件在内的所有调试事件的处理方式，我们将在第30章详细讨论。

## 10.6　中断到调试器

我们在介绍`DbgkpSendApiMessage`函数时（9.3.2节），该函数在代表调试子系统把调试事件发给调试器之前会调用DbgkpSuspendProcess挂起当前进程（被调试进程）。这样做是为了防止被调试进程继续运行会发生状态变化，给分析和观察带来困难。从被调试进程的角度来看，一旦它被调试子系统挂起，那么它便“戛然而止”了，代码（用户态的应用程序代码）停止执行，一切状态都被冻结起来，在调试领域，我们将这种现象称为“中断到调试器”（break into debugger）。从调试器的角度来讲，又叫将被调试进程“拉进调试器”（bring debuggee in）。

由于被调试进程内发生调试事件时，它就会中断到调试器，因此，触发调试事件很自然地成为将被调试进程中断到调试器的一种基本途径。因为断点事件很容易被触发，所以在被调试进程中触发断点异常便成为被调试进程中断到调试器的最常用方法。下面我们先介绍被调试进程中断到调试器的典型方法，然后介绍几个有关的问题。

### 10.6.1　初始断点

如我们在10.3.3节所介绍的，一个新创建进程的初始线程在初始化时会检查当前进程是否在被调试，如果是，那么便调用NTDLL.DLL中的`DbgBreakPoint`函数，触发一个断点异常，使新进程中断到调试器中。这通常是当调试器开始调试一个新创建进程时接收到的第一个断点异常，因此称为初始断点。

如果当前进程不在被调试，那么进程的初始化函数不会调用`DbgBreakPoint`，可以正常运行。

### 10.6.2　编程时加入断点

Windows系统的调试API中包含了一个用于产生断点异常的API，名为DebugBreak，它的原型非常简单，没有参数，也没有返回值。

```
void DebugBreak(void);
```

当编写程序时，如果希望在某种情况下中断到调试器中，可以加入如下代码。

```
if (IsDebuggerPresent() && <希望中断的附加条件>)
   DebugBreak();
```

这样，当程序执行到这里时，如果有调试器，并且中断的附加条件成立，那么被调试进程便会中断到调试器中，这对于调试某些复杂的多线程问题或随机发生的问题是很有用的，因为可以在应用程序中检测到希望中断到调试器的条件（包括条件断点难以实现的判断条件），然后中断到调试器中。

事实上，在x86平台上，DebugBreak API等价于一条INT 3指令，所以直接使用如下嵌入式汇编也可以达到同样的效果。

```
_asm{int 3};
```

使用API具有更好的跨平台性，代码看起来也更优雅。

### 10.6.3　通过调试器设置断点

前面介绍的两种方法都是在被调试程序的代码中静态地埋入断点指令。通过调试器的断点功能可以向被调试进程动态地插入断点指令，当被调试进程遇到这些断点指令时，会触发断点异常而中断到调试器中。

### 10.6.4　通过远程线程触发断点异常

前面的3种方法都是在程序的固定位置植入断点指令，只有当被调试进程执行到那里时，才会中断到调试器。如果希望被调试进程立刻中断到调试器，比如按下一个快捷键就中断下来，那么前面的方法就不合适了。这种根据用户的即时需要而将被调试进程中断到调试器的功能通常称为异步阻停（asynchronous stop）。

实现异步阻停的一种方法是利用Windows操作系统的`CreateRemoteThread` API，在被调试进程中创建一个远程线程，让这个线程一运行便执行断点指令，把被调试进程中断到调试器。要做到这一点，我们需要被调试进程中有一个包含断点指令的函数，为了让这个函数可以作为新线程的启动函数，它的函数原型应该符合SDK所定义的线程启动函数原型，即

```
DWORD WINAPI ThreadProc( LPVOID lpParameter);
```

事实上，NTDLL.DLL中已经设计好了这样一个函数，即`DbgUiRemoteBreakin`，它内部不仅会调用`DbgBreakPoint`执行断点指令，而且在一个结构化异常处理（SEH）块中调用，其伪代码如下。

```
DWORD WINAPI DbgUiRemoteBreakin( LPVOID lpParameter)
{
   __try
   {
      if(NtCurrentPeb()->BeingDebugged)
         DbgBreakPoint();
   }
   __except(EXCEPTION_EXECUTE_HANDLER)
   {
      return 1;
   }
   RtlExitUserThread(0); //never return   
}
```

增加异常保护（`__except`）的目的是捕捉断点异常，如果调试器没有处理，那么这个异常处理器会处理它，以防因无人处理而导致整个程序被终止。If语句用来检测当前进程是否在被调试，如果不在被调试，就不要触发断点异常，出于这个原因，向一个不在被调试的进程中创建远程线程并执行这个函数，并不会触发断点异常。倒数第2行用来强制退出当前线程。

介绍到这里，我们明白了可以创建一个远程线程来执行`DbgUiRemoteBreakin`函数，以触发断点异常。为了进一步简化这项任务，Windows XP引入了一个新的API，叫`DebugBreakProcess`，只要调用这个API就可以了。

```
BOOL DebugBreakProcess( HANDLE Process);
```

跟踪这个API的执行过程，可以发现它内部调用NTDLL.DLL中的`DbgUiIssueRemoteBreakin`函数，后者使用`DbgUiRemoteBreakin`作为线程函数创建远程线程。

我们把以上介绍的用于触发断点异常的远程线程称为远程中断线程（remote breakin thread），包括WinDBG在内的很多调试器所提供的Break功能使用了远程中断线程。例如，在WinDBG调试器中，选择Debug菜单中的Break项，或者按Ctrl+Break快捷键便会发出Break命令。WinDBG调试器接到此命令后会通过远程中断线程在被调试进程中产生一个断点异常，使其中断到调试器。明白这个原理后，我们就能理解为什么在被调试进程被中断后，WinDBG总是显示如下的内容。

```
(1e74.1dc4): Break instruction exception - code 80000003 (first chance)
eax=7ffde000 ebx=00000001 ecx=00000002 edx=00000003 esi=00000004 edi=00000005
eip=7c901230 esp=00beffcc ebp=00befff4 iopl=0         nv up ei pl zr na pe nc
cs=001b  ss=0023  ds=0023  es=0023  fs=0038  gs=0000             efl=00000246
ntdll!DbgBreakPoint:
7c901230 cc              int     3
```

所有这些内容都是关于远程中断线程的。此时执行栈回溯命令，会看到这个线程的启动函数`DbgUiRemoteBreakin`调用`DbgBreakPoint`的过程。

```
0:001> k
ChildEBP RetAddr  
00beffc8 7c9507a8 ntdll!DbgBreakPoint
00befff4 00000000 ntdll!DbgUiRemoteBreakin+0x2d
```

此时可以使用线程切换命令来切换到应用程序的线程，比如～0 s切换到0号线程（初始线程）。远程中断线程会在被调试进程恢复运行后很快退出，因此它大多时候不会给调试工作带来副作用。

当调试器附加到一个已经运行的进程（10.4节）时，通常也通过远程中断线程来将被调试进程中断到调试器，这个断点称为调试已运行进程的初始断点。

WinDBG调试器附带了一个名为Breakin.exe的小工具，是以命令行方式运行的，其功能就是向指定进程（通过命令行参数）创建一个远程中断线程。针对一个不在被调试的进程执行这个动作，不会对其造成大的伤害，正如我们前面所说的，`DbgUiRemoteBreakin`函数会先判断所处的进程是否在被调试。

### 10.6.5　在线程当前执行位置设置断点

利用远程线程实现异步阻停的一个明显不足，就是要在被调试进程中启动一个新的线程，这样做对被调试进程的执行环境（对象、句柄、栈和内存等）有较大的影响，而且可能会干扰被调试程序的自身逻辑。实现这一功能的另一种方式是在被调试进程的现有线程中触发断点。其主要步骤如下。

首先，将被调试进程中的所有线程挂起，这可以使用`SuspendThread`API来完成。

然后，取得每个线程的执行上下文（可以调用`GetThreadContext` API），得到线程的程序指针（PC）寄存器的值，并在这个值对应的代码处设置一个断点，也就是把本来的1字节保存起来，并写入断点指令（INT 3）的机器码（0xCC）。因为PC寄存器的值指向的是CPU下次执行这个线程时要执行的指令，所以在这个地方设置断点的目的就是当CPU下一次执行这个线程时就立刻触发断点异常而中断到调试器。对进程中的每个线程都重复此步骤。

最后，恢复所有线程（ResumeThread API），让它们继续执行，以触发断点。一旦有断点被触发，调试器会清除第二步设置的所有断点。

VC6调试器和重构前的WinDBG调试器（第29章）是使用以上方法来实现异步阻停的。

因为当用户发出中断命令时，被调试进程通常在执行某种等待函数（除非它在用户模式下有特别多的运行任务），而且大多数等待函数（GetMessage、Sleep、SleepEx）是调用内核服务而进入内核模式执行的，所以调试器取得的PC值通常指向的是系统服务返回后将执行的`ret`指令。这个指令地址在Windows XP中对应的调试符号是`ntdll!KiFastSystemCallRet`。清单10-10显示了VC6调试器在被调试进程中设置的断点指令。

清单10-10　VC6调试器在被调试进程中设置的断点指令

```
0:000> u 0x7c90eb94
ntdll!KiFastSystemCallRet:
7c90eb94 cc              int     3
```

在以上指令位置本来是`ret`指令，机器码是0xC3。在断点命中后，VC6会将其恢复成本来的指令。

值得注意的是，对于在内核模式执行的线程，动态的断点指令设置在内核服务返回的位置，这意味着只有在内核服务返回后，才能遇到断点指令。

### 10.6.6　动态调用远程函数

与刚才介绍的在程序指针的位置动态替换断点指令类似的一种方法是动态地调用一个函数，这个函数再执行断点指令。Windows 2000的NTDLL.DLL和KERNEL32.DLL中已经包含了这种方法的实现。简单来说，就是利用NTDLL.DLL中的`RtlRemoteCall`函数远程调用被调试进程内位于KERNEL32.DLL中的`BaseAttachComplete`函数。

具体来说，应该先将目标线程挂起，或使其锁定在一个稳定的内核状态，然后调用`RtlRemoteCall`函数，并将KERNEL32.DLL中BaseAttachCompleteThunk的地址作为调用点（CallSite）参数传递给RtlRemoteCall。BaseAttachCompleteThunk是一小段汇编代码，它的作用就是调用`BaseAttachComplete`函数。

RtlRemoteCall内部先通过`NtGetContextThread`内核服务取得目标线程的上下文，得到目标线程的栈地址，然后调整栈指针将取得的上下文（CONTEXT）结构和参数用`NtWriteVirtualMemory`写到目标线程的栈上，接着将线程上下文结构中的程序指针（EIP）寄存器设置为参数指定的调用地址（CallSite）。接下来通过`NtSetContextThread`将修改后的CONTEXT结构设置回目标线程。这些准备工作做好后，便可以调用`NtResumeThread`了，即恢复目标线程运行，而且它一运行便应该执行调用点所指向的代码。

`BaseAttachComplete`函数内部查询当前进程是否在被调试，如果是，便执行`DbgBreakPoint`触发断点。清单10-11使用kn命令显示了被调试的记事本程序中断到调试器（WinDBG）后的栈回溯。

清单10-11　中断到调试器后的栈回溯（Windows 2000）

```
0:000> kn
 # ChildEBP RetAddr  
00 0006fbe4 77e8be83 ntdll!DbgBreakPoint
01 0006fbf4 77e88929 KERNEL32!DebugActiveProcess+0x1e0
02 0006fee4 01002a01 KERNEL32!BaseAttachCompleteThunk+0x13
03 0006ff24 01006576 notepad!WinMain+0x63
04 0006ffc0 77e67903 notepad!WinMainCRTStartup+0x156
05 0006fff0 00000000 KERNEL32!SetUnhandledExceptionFilter+0x5c
```

首先，栈帧#01所对应的函数应该是`BaseAttachComplete`，因为缺少它的符号，所以调试器就以`DebugActiveProcess`函数为参照物。

观察上面的栈回溯，就好像记事本（notepad）程序的WinMain函数调用`BaseAttachCompleteThunk`，但事实上根本不是这样的，WinMain实际调用的是GetMessageW API，这个API进而调用内核模式中的子系统服务。但在内核模式执行（等待）时，`RtlRemoteCall`函数执行了前面描述的动作，使这个线程“飞”到`BaseAttachCompleteThunk`处。

当`DbgBreakPoint`返回后，`BaseAttachComplete`会调用NtContinue，并将其保存在栈上的CONTEXT结构作为参数，这样，这个线程便又被恢复成原来的样子了，前面执行的动作就好像梦游一样被遗忘了。

### 10.6.7　挂起中断

以上3种异步阻停的方法都是希望被调试程序继续执行，然后遇到断点指令就中断到调试器，也就是假定被调试进程依然可以继续执行用户态代码。但是，如果被调试进程出于某种原因不能继续执行用户态代码，那么这3种方法就都不可行了。举例来说，如果被调试进程因为某个同步对象被死锁无法创建或启动新的线程，那么远程中断线程方法就不可行了。对于前面介绍的在内核服务返回处设置的动态断点，如果线程在内核态无限期等待，即所谓的“挂在内核态”（hang in kernel），那么这样的断点也不再会命中了。

针对以上问题，一种替代性的方法是强行将被调试进程的所有线程挂起，然后进入一种准调试状态。我们称这种方法为“挂起中断”（breakin by suspend）。之所以叫准调试状态，是因为通过这种方式中断到调试器后，不可以运行单步执行等跟踪命令。

WinDBG调试器在使用远程线程中断功能超时后会使用挂起中断方式。具体来说，WinDBG在创建远程中断线程后，会等待断点事件发生，等待数秒钟后会显示清单10-12中的前两行信息。再等待30s后，WinDBG就会使用挂起中断方式，并提示第3行和第4行的信息。在将所有线程都挂起后，调试引擎的底层函数会模拟一个唤醒调试器（wake debugger）的异常事件，调试器的事件处理函数收到这个事件后便会中断给用户，提示第5～10行的信息。

清单10-12　WinDBG使用挂起中断方式时输出的提示信息

```
Break-in pending
Break-in sent, waiting 30 seconds...
WARNING: Break-in timed out, suspending.
         This is usually caused by another thread holding the loader lock
(abc.1530): Wake debugger - code 80000007 (first chance)
eax=00000000 ebx=00000000 ecx=00000000 edx=00000000 esi=ffffffff edi=00000001
eip=7c90eb94 esp=0013dbc8 ebp=0013e618 iopl=0         nv up ei ng nz na pe nc
cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000286
ntdll!KiFastSystemCallRet:
7c90eb94 c3              ret
```

因为线程通常挂在内核模式，所以用户模式下看到的程序指针是指向`KiFastSystemCallRet`的。这一点与前一种方法产生的中断类似。使用挂起方式中断后，在调试器中可以执行各种观察和编辑类命令来分析与操作被调试程序的栈回溯、内存和栈等信息。但是不可以执行跟踪类命令，例如，以下是执行p命令时，WinDBG给出的错误提示。

```
0:000> p
Due to the break-in timeout the debugger cannot step or trace
       ^ Operation not supported in current debug session 'p'
```

### 10.6.8　调试快捷键（F12键）

除了在调试器中发出Break命令和使用Breakin这样的小工具之外，还有一种方式可以“激发”被调试进程中断到调试器，那就是向被调试进程按调试快捷键（默认为F12键）。举例来说，当我们使用WinDBG调试计算器程序时，除了通过在WinDBG中按Ctrl+Break组合键将计算器中断到调试器，还可以向计算器程序按F12键（也就是当计算器程序在前台时按F12键）。

在内部，该功能是因为Windows子系统的内核部分接收到此快捷键，然后通过LPC请求CSRSS.EXE中的`SrvActivateDebugger`服务。

`SrvActivateDebugger`首先检查要调试的进程是不是自己。如果是，而且自己处于被调试状态，便调用DbgBreakPoint中断到调试器；如果不是，便试图通过远程方式触发断点事件。触发的方式因为Windows版本的不同而略有不同。

在Windows XP之前，SrvActivateDebugger使用的是动态调用（RtlRemoteCall）远程函数的方法。从Windows XP开始，`SrvActivateDebugger`使用前面介绍的向要调试进程创建新的远程中断线程的方法。从调试器的角度来看，前一种方法的断点异常发生在被调试进程的UI线程（即现有线程）中。后一种方法激发的断点异常发生在新创建的远程中断线程中。

对于Windows XP所使用的方法，因为也是依靠远程中断线程机制工作的，所以在使用这种方法中断后，调试器的当前线程是远程中断线程，这与前面介绍的调试器自己创建远程中断线程的情况是一样的。事实上，这两种方法只是远程线程的创建者有所不同。

可以通过如下注册表表键下的UserDebuggerHotKey选项，来指定其他键作为调试快捷键。

```
HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion\AeDebug
```

### 10.6.9　窗口更新

在调试进程被中断到调试器后，被调试进程是处于“停顿”状态的，直到调试器恢复其运行。在这一阶段，如果被调试进程是窗口程序，那么它的窗口是僵死的，不可移动，不可改变大小，会遮挡住它所对应的桌面区域，对“显示桌面命令”也不响应。另外，被中断到调试器的进程自己也不能刷新窗口内容，所以它的窗口区域处于“无人更新”状态。为了行文方便，我们把中断到调试器的被调试进程的窗口简称为被中断窗口（broke window）。

因为不同版本的Windows系统使用的窗口管理策略有所不同，所以被中断窗口表现出来的“症状”也有所不同。

在Windows 2000中，系统会更新被中断窗口的非用户（non-client）区，包括标题栏和边框等。对于用户区，如果有其他窗口移过这个窗口，那么被中断窗口的对应区域会被擦成背景色（默认白色）。

例如，记事本程序在被WinDBG程序调试，并且被中断到调试器。将WinDBG的窗口（或者其他窗口）在记事本窗口上方移动，记事本的用户区会被涂抹掉，但是标题栏和边框不会，因为系统会更新非用户区。

被中断窗口在Windows XP中的情况，整个被中断窗口都处于被任意涂抹状态，而且会留下移动窗口的痕迹。

有必要说明的是，在Windows XP中，对于未被调试的进程，如果它的主窗口在几秒内没有响应，那么系统会将其视为无响应窗口（not responding window），并为其创建一个所谓的精灵窗口（ghost window）。精灵窗口与原来窗口具有相同的 Z 顺序（Z-Order）、位置、大小，但其窗口标题会包含无响应（Not Responding）字样。用户可以移动精灵窗口、调整其大小，也可以通过窗口标题栏的关闭按钮（默认菜单，或Alt+F4组合键）触发系统终止这个应用程序。

为了更好地理解和感受精灵窗口的特性，我们特意编制了一个小程序HungWin（code\ chap10\HungWin），直接（非调试）启动它，单击File菜单中的Hang，那么它便会调用SuspendThread API将所在线程（该程序的唯一线程）挂起。

```
case IDM_HANG:
   SuspendThread(GetCurrentThread());
```

在几秒后，再单击HungWin程序的窗口，便会发现其菜单不见了，而且标题栏出现了Not Responding字样，这便是所谓的“精灵窗口”。接下来可以感受一下它的特性，比如移动和改变大小等。尝试将精灵窗口移动至中断到调试器的计算器窗口，然后拖动第3个窗口（CMD.EXE）在精灵窗口上面晃动，我们会发现计算器窗口区域留下了很多痕迹，而精灵窗口始终保持着清洁状态。

如果在窗口僵死前就将调试器附加到这个程序上，那么Windows XP就不会对其应用精灵窗口策略，这对于分析问题和软件调试是有利的。

对于Windows Vista，如果使用新引入的Aero风格来显示外观（appearence），那么被中断窗口会始终保持中断前的显示状态。如果使用以前的显示外观，那么被中断窗口的刷新策略与Windows 2000和Windows XP中的情况类似，在此不再赘述。

## 10.7　输出调试字符串

输出调试（debug output）字符串是一种常用的辅助调试手段。在DOS程序或Windows系统的控制台程序中，我们可以使用print（或printf）函数来打印调试字符串。对于Windows系统下的非控制台程序，print函数不再适用。事实上，不论是控制台程序还是窗口程序，Windows系统都可以使用`OutputDebugString()` API来输出调试字符串。

根据Windows SDK的说明，`OutputDebugString`可以把参数指定的字符串发送给调试器。如果程序不在被调试，那么系统调试器（内核调试器）会显示该字符串。如果系统调试器也没有被激活，那么`OutputDebugString`什么也不做。

### 10.7.1　发送调试信息

下面我们先来看一看`OutputDebugString`是如何把参数指定的字符串发送给调试器的，这并不是一个不值一提的问题，因为发送字符串的应用程序和接收字符串的调试器分别属于不同的进程，而且还要保证不论在有无调试器的情况下都能按规定工作，所以就有不少值得探索的细节。

简单来说，`OutputDebugString`利用`RaiseException() API`产生一个特殊的异常，该异常的代码固定为`DBG_PRINTEXCEPTION_C`，ntstatus.h中有其定义。

```
#define DBG_PRINTEXCEPTION_C             ((NTSTATUS)0x40010006L)
```

我们把这个异常称为调试打印（debug print）异常。清单10-13给出了`OutputDebugString`函数调用`RaiseException` API发起调试打印异常的伪代码。

清单10-13　OutputDebugString API发起调试打印异常的伪代码（部分）

```
__try 
{
        ExceptionArguments[0] = strlen (lpOutputString) + 1;
        ExceptionArguments[1] = (ULONG_PTR)lpOutputString;
        RaiseException (DBG_PRINTEXCEPTION_C, 0, 2, ExceptionArguments);
}
__except(EXCEPTION_EXECUTE_HANDLER) 
{
// 异常处理代码
}
```

从上面的代码可以看到，调试信息（lpOutputString）的长度和地址是作为异常的参数传递给`RaiseException` API的。以上代码有两点值得说明。第一，`OutputDebugString`是在KERNEL32.DLL中实现的，分为UNICODE版本（函数名为`OutputDebugStringW`）和ANSI版本（函数名为`OutputDebugStringA`）。与其他API将ANSI版本的参数转换为UNICODE然后调用UNICODE版本的函数不同，`OutputDebugString` API是UNICODE版本的函数，把UNICODE类型的字符串转换为ANSI，然后调用ANSI版本的函数。因此，我们看到以上代码中使用的是适用于ANSI字符串的strlen函数。第二，注意第4行，这里只是把字符串参数（`lpOutputString`）的地址存入`ExceptionArguments[1]`，传递给`RaiseException` API，这意味着，异常信息中将只包含字符串的地址，而不是其实际内容。

`RaiseException` API被调用后，会产生一个标准的异常结构EXCEPTION_RECORD，然后调用内核服务将这个模拟的异常发送到内核中以进行分发。

### 10.7.2　使用调试器接收调试信息

内核中的异常分发函数`KiDispatchException`会按照统一的流程来分发调试打印异常。如第9章所介绍的，`KiDispatchException`会调用支持用户态调试的内核例程`DbgkForwardException`向调试子系统通报异常。`DbgkForwardException`如果检查到当前进程正在被调试，它会将这个异常通过调试子系统服务器发给调试器。

我们知道，调试器工作线程是通过`WaitForDebugEvent` API调用NTDLL.DLL中的`DbgUiWaitStateChange`函数来等待调试事件的。在接收到异常事件后，从Windows XP开始的Windows系统使用`DbgUiConvertStateChangeStructure` 函数将`DBGUI_WAIT_STATE_CHANGE`结构组装成`DEBUG_EVENT`结构。在Windows XP之前，是`WaitForDebugEvent` API自己组装的。但不论哪个函数来做结构组装工作，对于异常事件，如果异常代码等于`DBG_PRINTEXCEPTION_C`，那么它们都会将事件代码字段（`dwDebugEventCode`）设置为`OUTPUT_DEBUG_STRING_EVENT`，而不是`EXCEPTION_DEBUG_EVENT`，并将异常参数中的调试信息填写到`DEBUG_EVENT`结构的`DebugString`子结构中。`DebugString`是一个`OUTPUT_DEBUG_STRING_INFO`结构，其定义如下。

```
typedef struct _OUTPUT_DEBUG_STRING_INFO 
{
  LPSTR lpDebugStringData;     // 调试信息字符串的地址
  WORD fUnicode;                // 是否为UNICODE，参见正文
  WORD nDebugStringLength;     // 调试信息字符串的长度
} OUTPUT_DEBUG_STRING_INFO, *LPOUTPUT_DEBUG_STRING_INFO;
```

如前面所介绍的，对于UNICODE程序，`OutputDebugString` API的UNICODE版本（`OutputDebugStringW`）会将UNICODE格式的字符串先转换为ANSI格式。因此，目前版本的Windows系统中，调试信息都是使用非UNICODE方式传递的，所以上面的`fUnicode`字段会被固定为FALSE。

概括一下，`OUTPUT_DEBUG_STRING_EVENT`事件是异常事件的一个特例，它是以异常事件的形式而产生和分发的，只有在发送给调试器之前，系统（`WaitForDebugEvent` API或`DbgUiConvertStateChangeStructure`）才将其翻译为`OUTPUT_DEBUG_STRING_EVENT`事件。

调试器收到`OUTPUT_DEBUG_STRING_EVENT`事件后，会从其参数中得到调试信息字符串的地址和长度，然后使用内存访问函数从被调试进程的空间中读取调试信息字符串，并显示出来。显示后，调试器默认会立即调用`ContinueDebugEvent`回复此事件，表明自己处理了该异常，于是`KiDispatchException`函数结束异常分发，被调试程序便继续正常运行了。

### 10.7.3　使用工具接收调试信息

接下来看一下应用程序没有被调试的情况。首先我们注意在上面的伪代码中，产生异常的代码（调用RaiseException API）是放在一个异常保护块（`__try`和`__except`，将在第11章详细讨论）中的。

在应用程序没有被调试的情况下，异常不会发给用户态调试器，因此`KiDispatch Exception`函数会继续分发这个异常，也就是寻找异常处理器来处理这个异常。其细节将在第11章介绍，现在我们只要知道，系统会找到并执行异常处理块中的代码，即上面伪代码中`__except`块所对应的内容。

`__except`块中的异常处理代码的主要逻辑是试图将字符串发给DBWIN工具。DBWIN是旧的Windows SDK中包含的一个16位的小工具（由DBWin.DLL和DBWin.EXE组成），用以捕捉应用程序中使用`OutputDebugString` API所输出的信息。目前的SDK不再包含DBWIN，但其中包含的DBMon（Debug Monitor，控制台程序）可以完成类似的任务。可以完成类似任务的工具还有很多，比如Debug View等。为了描述简单，我们仍然使用DBWIN来泛指这类工具。

那么，异常处理块是如何将字符串发给DBWIN程序的呢？简单来说，是通过几个内核对象使用进程间的通信机制来通信的，其主要步骤如下。

首先，`OutputDebugString`会检查静态变量并判断是否创建过名为“DBWinMutex”的互斥（mutex）对象，如果没有，则调用`CreateDBWinMutex`函数来创建，并将其句柄保存在另一个静态变量中。DBWinMutex用于同步`OutputDebugString`所在进程与DBWIN进程间的通信，保证同一时间只能有一个线程与DBWIN通信（传递数据），以避免当有多个线程都包含对`OutputDebugString`的调用时导致数据丢失或其他错误。由于创建DBWinMutex时指定了对象名称，因此对象管理器会保证一旦系统中已经存在具有该名称的对象，那么后来的调用只是打开已经存在的对象。

创建或打开`DBWinMutex`对象后，`OutputDebugString`会使用`WaitForSingleObject`函数无限期地等待`DBWinMutex`对象的使用权。

等待成功后，`OutputDebugString`会通过`OpenFileMapping` API试图打开名为“DBWIN_BUFFER”的内存映射文件（file mapping）对象（section对象）。DBWIN_BUFFER对象是由DBWIN程序通过调用`CreateFileMapping` API创建的，`OutputDebugString`只是调用`OpenFileMapping` API试图打开该对象，如果打开失败，则说明系统内没有DBWIN程序在运行。这时`OutputDebugString`会调用`DbgPrint`函数，试图将字符串打印到内核调试器。

如果成功打开DBWIN_BUFFER，那么`OutputDebugString`会调用`MapViewOfFile` API将映射文件映射到本进程空间中。

接下来，`OutputDebugString`会调用OpenEvent API打开两个事件（event）对象，分别为“DBWIN_BUFFER_READY”和“DBWIN_DATA_READY”。`DBWIN_BUFFER_READY`对象用来供DBWIN程序指示缓冲区是否准备好，OutputDebugString通过等待该事件对象判断是否可以向缓冲区（DBWIN_BUFFER映射文件）写数据。`DBWIN_DATA_READY`供`OutputDebugString`在将数据写到内存映射文件后，通知DBWIN程序来读取数据。当数据量较大时，`OutputDebugString`会分多次发送，每次将数据写到内存映射文件后，便设置DBWIN_DATA_READY通知DBWIN读取数据，然后开始等待`DBWIN_BUFFER_READY`事件。DBWIN得到通知后便读取数据，读取完毕后便设置`DBWIN_BUFFER_READY`事件，`OutputDebugString`得到`DBWIN_BUFFER_READY`事件后，便继续写剩下的数据，如此反复，直到发完所有数据。

当数据传递结束后，`OutputDebugString`会释放`DBWinMutex`互斥对象，以便让其他线程或程序可以与DBWIN通信。

`DBWIN_BUFFER_READY`事件的另一个作用，是供各种DBWIN程序通过检查它的存在来保证系统中只有一个DBWIN程序在运行。DBWIN程序在初始化期间创建`DBWIN_BUFFER_READY`事件对象（使用CreateEvent API），即使成功，也会通过检查GetLastError()的返回值是否为`ERROR_ALREADY_EXISTS`，来判断系统中是否已经存在同名的对象。如果有，则停止继续创建其他对象。例如，当运行DBMon的第二个实例时，它会显示“already running”（已经运行），然后退出。

以上对象除了DBWinMutex之外，其他都是由DBWIN程序创建的，使用WinDBG附加到DBWIN程序（DBMon）上，然后使用`!handle 0 5`命令便可以看到这些对象。清单10-14显示了在DBMon进程中用于与`OutputDebugString`通信的各个对象。

清单10-14　DBMon进程中用于和OutputDebugString通信的各个对象

```
0:000> !handle 0 5   //参数0代表列出所有句柄，参数5代表显示对象名称和类型
...
Handle 30  Type    Event
  Name           \BaseNamedObjects\DBWIN_BUFFER_READY
Handle 34  Type   Event
  Name           \BaseNamedObjects\DBWIN_DATA_READY
Handle 38  Type   Section
  Name           \BaseNamedObjects\DBWIN_BUFFER
```

因为当`OutputDebugString` API与DBWIN程序通信时才需要`DBWinMutex`对象，所以没有调用过`OutputDebugString` API或调用了`OutputDebugString` API但处于调试状态的进程中不会有`DBWinMutex`对象。为了证明这一点，我们特意编写了一个名为DbgString的小程序，清单10-15列出了它的源代码。

清单10-15　用于观察DBWinMutex内核对象的DbgString小程序

```
1  #include "stdafx.h"
2  #include <windows.h>
3 
4  int main(int argc, char* argv[])
5  {
6     printf("Program to test existence of DBWinMutex object.\n");
7     BOOL bDebuggerPresent=IsDebuggerPresent();
8 
9     if(!bDebuggerPresent)
10    {
11       printf("Inspect this process now to verify no DBWinMutex.\n");
12       getchar();
13       OutputDebugString(szMsg);
14       printf("Inspect this process now to verify DBWinMutex exists.\n");
15       getchar();
16    }
17    else
18    {
19       OutputDebugString(szMsg);
20       printf("Inspect this process now to verify no DBWinMutex.\n");
21       getchar();
22    }
23    return 0;
24 }
```

如果在控制台下直接运行该程序（调试版本或非调试版本均可），那么当运行到第12行等待用户按键时，使用WinDBG附加到该进程，然后使用`!handle 0`命令列出所有句柄，我们会发现进程中还不存在`DBWinMutex`对象。输入g命令让DbgString程序运行，并按任意键让`getchar`函数返回，而后DbgString继续运行，调用`OutputDebugString`后停在第15行等待输入，这时按Ctrl+Break快捷键将DbgString中断到调试器。再次执行`!handle`命令，我们会发现进程中还是不存在`DBWinMutex`对象。这是因为有调试器时，调试子系统会通过`OUTPUT_DEBUG_STRING_EVENT`事件将字符串发给调试器，根本不会执行到异常保护块中与DBWIN程序通信的代码。由此我们也很容易理解，为什么当一个程序运行在调试器中时，DBWIN程序就收不到通过`OutputDebugString`输出的信息了。

关闭WinDBG和DbgString，然后再次运行DbgString，等到DbgString第二次等待输入（第15行）时将WinDBG附加到调试器，执行`!handle`命令，我们会看到进程中已经有DBWinMutex对象了。

```
0:000> !handle 0 6 Mutant    // 只显示互斥类型的对象
Handle 10                    // 句柄号
  Attributes      0    
  GrantedAccess   0x120001:    
         ReadControl,Synch    
         QueryState    
  HandleCount     33        // 打开这个互斥对象的句柄总数
  PointerCount    35        // 引用这个对象的指针数量
  Name            \BaseNamedObjects\DBWinMutex    // 对象名称
```

从句柄计数等于0x33判断，系统中有很多其他进程打开过这个对象。

原书P227的图10-11展示了`OutputDebugString()` API与DBWIN程序之间通信的基本流程。

这里有几点值得说明。首先，系统中可能有多个程序调用`OutputDebugString`（图中只画出了一个），但活动的DBWIN程序应该只有一个。当多个进程中的`OutputDebugString`要与DBWIN程序进行通信时，它们会通过等待DBWinMutex互斥对象进行排队，谁获得`DBWinMutex`对象，谁便与DBWIN程序通信。

另外，图中列出的函数调用只用来示意关键参数，其他参数可能没有列出。想自己编程实现DBWIN程序的读者可以参考如下两个例子：（1）MSDN中的DBMON源代码，即接收`OutputDebugString`的小控制台程序的完整代码；（2）Andrew Tucker、Daniel Christian和Dwayne Towell编写的DBWIN32程序，DBWIN32是32位的Windows GUI程序，它在DBMON代码的基础上增加了图形化界面，并继承了16位DBWIN的部分功能，比如通过一个驱动程序输出信息到另一个单色显示器（monochromic monitor）。

最后想说明的一点是，从性能角度来看，`OutputDebugString`是一种执行效率较低的方法。对于效率要求较高的程序，过多使用`OutputDebugString`会影响程序的运行速度。导致`OutputDebugString`效率较低的原因主要有以下几点。

首先，`OutputDebugString`是通过`RaiseException`抛出异常来触发与调试器或DBWIN程序的通信的，`RaiseException`会导致当前线程从用户模式切换到内核模式，然后经由内核模式的异常处理函数进行分发，最后才将信息发送给用户模式的调试器或DBWIN程序。

其次，当以非调试状态执行时，如果系统中有DBWIN程序在运行，那么与DBWIN程序通信需要一定的开销（打开、等待多个内核对象）。如果系统中有大量对`OutputDebugString`的调用，那么DBWIN程序可能很忙碌，调用`OutputDebugString`的线程需要排队等待。

使用`OutputDebugString`输出调试信息的另一个缺点是安全性和可控性差。也就是说，通过`OutputDebugString`输出的信息可以很容易被各种DBWIN程序接收到，因此不该使用`OutputDebugString`输出可能泄露知识产权的信息。可控性差是指`OutputDebugString`函数本身没有提供动态开启或关闭信息输出的机制，如果希望不重新编译就开启或关闭使用`OutputDebugString`输出的信息，那么需要自己编写代码对其进行封装，在封装函数中包含动态控制机制。

尽管`OutputDebugString`有以上不足，但它也有很多优点，比如使用方便、可靠性高、在调试和非调试状态下都可以工作等。

## 10.8　终止调试会话

本节介绍终止调试会话的几种典型情况，探索每种情况的内部过程，并比较它们的异同。除非特别指出，本节主要讨论Windows XP之后（包括XP）的情况。

### 10.8.1　被调试进程退出

不同类型的程序有不同的退出方式、菜单和快捷键等，但无论使用哪种方式，通常都会执行内核中的`PspExitThread`函数来退出线程。

`PspExitThread`函数内部会通过EPROCESS结构的`DebugPort`字段检查当前进程是否在被调试，如果是，它会根据当前线程是否是最后一个线程而调用`DbgkExitProcess`或`DbgkExitThread`来通知调试子系统。清单10-16显示了在使用Alt+F4组合键关闭被调试的记事本程序时的执行过程。

清单10-16　使用Alt+F4组合键关闭被调试的记事本程序时的执行过程

```
kd> k
ChildEBP RetAddr  
f4a56c60 805bbbc4 nt!DbgkExitProcess                       // 通知调试子系统
f4a56d08 8058ac25 nt!PspExitThread+0x2a7                   // 进程管理器的线程退出函数
f4a56d28 80591d13 nt!PspTerminateThreadByPointer+0x50      // 终止线程
f4a56d54 804da140 nt!NtTerminateProcess+0x116              // 终止进程的内核服务
f4a56d54 7ffe0304 nt!KiSystemService+0xc4                  // 内核服务分发函数
0006fdf4 77f7664a SharedUserData!SystemCallStub+0x4        // 调用内核服务
0006fdf8 77e798ec ntdll!NtTerminateProcess+0xc             // 内核服务的残根函数
0006fef0 77e7990f kernel32!_ExitProcess+0x57               // Kernel32的进程退出函数
0006ff04 77c379c8 kernel32!TerminateProcess                // 终止进程API
0006ff0c 77c37ad9 msvcrt!__crtExitProcess+0x2f            // C运行时库的进程退出函数
0006ff18 77c37aea msvcrt!_cinit+0xe4      // 此处的cinit只是参照点，实际应为doexit
0006ff28 01006c65 msvcrt!exit+0xe         // C运行时库的退出函数
0006ffc0 77e814c7 notepad!WinMainCRTStartup+0x185          // 编译器插入的启动函数
0006fff0 00000000 kernel32!BaseProcessStart+0x23           // 进程启动函数
```

如果程序在被调试，那么`DbgkExitProcess`会通过调试子系统向调试器发送进程退出事件。调试器收到此事件后便知道被调试进程正在退出。接下来的处理与调试器的实现和调试器中对进程退出事件的设置有关。对于MSDEV等调试器（VC6的集成调试器），收到进程退出事件后，它们便清理内部状态，结束本次调试了。在WinDBG中，默认情况下，它会向用户报告此事件，如清单10-17所示，并中断下来。

清单10-17　WinDBG收到进程退出事件后输出的内容

```
eax=00000000 ebx=00000000 ecx=7c800000 edx=7c97c080 esi=7c90e88e edi=00000000
eip=7c90eb94 esp=0007fde8 ebp=0007fee4 iopl=0         nv up ei pl zr na pe nc
cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000246
ntdll!KiFastSystemCallRet:
7c90eb94 c3              ret
```

值得说明的是，上面显示的 `KiFastSystemCallRet`是用来调用内核服务的`KiFastSystemCall`函数（2.5.3节）的一部分（返回部分），它是目前程序指针寄存器的值（7c90eb94）所对应的符号。程序指针总是指向将要执行的那条指令，所以，现在还没有执行到这条ret指令。事实上，被调试进程目前是在内核中执行的，因为执行线程退出函数而被调试子系统挂起，目前处于睡眠状态。因为PspExitThread是在释放线程的用户态栈之前通知调试子系统的，所以当收到进程退出事件时，调试器仍然可以观察被调试进程中的信息，包括观察栈、内存、PEB、TEB等结构，这对于调试应用程序意外退出的问题是非常有用的。但是，此时不能再执行让被调试进程运行的p（单步执行）和g等命令，如果执行，WinDBG会提示没有可以运行的被调试程序。

```
0:000> p
       ^ No runnable debuggees error in 'p'
```

此时，进程的句柄和EPROCESS结构仍在，所以在任务管理器中，仍然可以看到这个进程。要想让被调进程继续完成其退出工作，需要触发WinDBG继续调试事件（ContinueDebugEvent），恢复运行被调试进程。这可以通过WinDBG的停止调试功能（Debug → Stop Debugging）或分离调试对话功能（Debug → Detach Debuggee）来完成。`PspExitThread`函数恢复运行后，会继续完成线程的清理工作，释放线程所使用的资源。

进程的最后清理和删除工作是由进程管理器的工作线程执行`PspProcessDelete`函数来完成的。`PspProcessDelete`函数内部会检查进程EPROCESS结构的DebugPort，如果它不为空，会调用`ObDereferenceObject`函数取消引用。而后`PspProcessDelete`调用内存管理器的`MmDeleteProcessAddressSpace`删除进程的地址空间，该进程彻底在系统中消失。

### 10.8.2　调试器进程退出

退出调试器是结束调试会话的另一种简单方式。与其他Windows进程相比，调试器进程退出的大多数步骤是一样的。值得我们注意的便是有关调试对象的操作。

当调试器工作线程退出时，`PspExitThread`函数会检查TEB结构的`DbgSsReserved[1]`字段，如果该字段不为空，会调用`ObCloseHandle`关闭调试对象的句柄。清单10-18显示了调试记事本程序的WinDBG进程退出时（执行`PspExitThread`）通过内核调试器观察到的调试对象句柄的信息。

清单10-18　调试对象句柄的信息

```
kd> !handle 0748
processor number 0, process 81caf958
PROCESS 81caf958  SessionId: 0  Cid: 0268    Peb: 7ffdf000  ParentCid: 065c
    DirBase: 0c884000  ObjectTable: e18f80e8  HandleCount: 139.
    Image: windbg.exe

Handle table at e1288000 with 139 Entries in use
0748: Object: 81f6e8e8  GrantedAccess: 001f000f Entry: e1288e90
Object: 81f6e8e8  Type: (81fc9778) DebugObject
    ObjectHeader: 81f6e8d0 (old version)
        HandleCount: 1  PointerCount: 2
```

空行上方的信息是关于所在进程的，映像文件名为windbg.exe，进程中共有0x139个句柄。空行下方的信息是关于参数（0x748）所指定句柄的，对象的类型为DebugObject，即调试对象，句柄计数目前为1，引用这个对象的指针计数为2。

需要指出的是，因为WinDBG在调用`DbgUiConnectToDbg`后会将`DbgSsReserved[1]`字段设置为0（10.1节），所以在执行`PspExitThread`函数时，如果检测到`DbgSsReserved[1]`字段为0，就不会调用`ObCloseHandle`来关闭调试对象句柄。但这并不要紧，因为后面清理句柄表（handle table）时，会递减句柄计数，相当于执行ObCloseHandle。

清单10-19显示了`PspExitThread`函数调用`ObKillProcess`函数清理句柄表的过程。

清单10-19　清理句柄表的过程

```
kd> kn
 # ChildEBP RetAddr     
00 f4fdb620 805812b4 nt!DbgkpCloseObject                  // 调用对象的关闭函数
01 f4fdb650 80581108 nt!ObpDecrementHandleCount+0x119     // 递减关闭句柄的引用
02 f4fdb678 80591fbb nt!ObpCloseHandleTableEntry+0x14b   // 关闭句柄表中的一项
03 f4fdb694 80592041 nt!ObpCloseHandleProcedure+0x1d      // 关闭句柄表中所有项
04 f4fdb6c4 80591f6e nt!ExSweepHandleTable+0x4d         // 清理句柄表
05 f4fdb6f0 80591e01 nt!ObKillProcess+0x5a     // 对象管理器中针对进程退出所用的函数
06 f4fdb798 8058ac25 nt!PspExitThread+0x5cb                // 线程退出
07 f4fdb7b8 80591d13 nt!PspTerminateThreadByPointer+0x50   // 终止线程
08 f4fdb7e4 804da140 nt!NtTerminateProcess+0x116           // 终止进程内核服务
09 f4fdb7e4 7ffe0304 nt!KiSystemService+0xc4               // 系统服务分发函数
0a 0006c310 77f7664a SharedUserData!SystemCallStub+0x4     // 调用系统服务
0b 0006c314 77e798ec ntdll!NtTerminateProcess+0xc          // 系统服务的残根
0c 0006c40c 77e7990f kernel32!_ExitProcess+0x57      // Kernel32内的进程退出函数
0d 0006c420 0103b76a kernel32!TerminateProcess       // 终止进程API
0e 0006c430 0103aeff windbg!TerminateApplication+0xda  // 终止应用程序
0f 0006cfcc 77d43a68 windbg!FrameWndProc+0x18df      // 窗口过程
…                                                    // 消息分发过程和其他函数省略
```

栈帧#01用于递减关闭句柄的引用。这个函数会调用句柄所对应的对象类型注册的关闭函数，并将对象的句柄计数作为参数传递给关闭函数。调试对象（DebugObject）句柄的关闭函数是`DbgkpCloseObject`。`DbgkpCloseObject`函数共有5个参数，分别是指向EPROCESS结构的指针、指向调试对象的指针、赋给句柄的权限、进程的句柄计数、调试对象的句柄计数。

`DbgkpCloseObject`函数内部会先检测参数中指定的调试对象句柄计数是否大于1，如果大于1，便返回。对于我们讨论的情况，从清单10-18可以看出，引用计数等于1，因为尽管`ObpDecrementHandleCount`会在调用对象关闭函数前递减句柄计数，但是它在递减前会把本来的计数保存起来，并作为参数传递给关闭函数。也就是说，在调用`DbgkpCloseObject`函数时，使用`!object`命令观察到的句柄计数已经是0了。

```
kd> !object 81f6e8e8 
Object: 81f6e8e8  Type: (81fc9778) DebugObject
    ObjectHeader: 81f6e8d0 (old version)
    HandleCount: 0  PointerCount: 2
```

但是在传递给`DbgkpCloseObject`函数的参数中，句柄计数仍然是1。

在`DbgkpCloseObject`确认需要关闭调试对象后，它接下来做的一个主要动作就是列举系统内的所有进程，对于每个进程检查它的`DebugPort`字段，看是否与要关闭的调试对象相同（与参数2比较）。如果相同，那么说明这个进程正在被要退出的调试器进程所调试，于是`DbgkpCloseObject`会对这个被调试进程执行3个动作：第一，将这个进程的`DebugPort`字段设置为空；第二，调用`DbgkpMarkProcessPeb`函数设置这个进程的PEB结构中的`BeingDebugged`字段；第三，检查调试对象的标志（Flags）字段，如果包含KillOnExit标志（位1），那么便调用`PspTerminateProcess`终止这个进程，这便是为什么被调试进程随着调试器的退出而退出。

总之，当退出正在调试记事本程序的WinDBG程序时，发生的主要动作依次如下。

（1）[WinDBG] 调用系统服务NtTerminateProcess，开始终止WinDBG进程。

（2）[WinDBG] 执行PspExitThread函数，WinDBG的调试工作线程退出。

（3）[WinDBG] 执行PspExitThread函数，WinDBG的主线程（UI线程）退出。

（4）[WinDBG] 执行ObKillProcess函数，开始清理WinDBG进程的句柄表。

（5）[WinDBG] 执行DbgkpCloseObject函数，关闭调试对象，将被调试进程的DebugPort字段设置为空，并调用DbgkpMarkProcessPeb设置PEB的BeingDebugged字段，并引发终止被调试进程（记事本进程）。

（6）[记事本]执行PspExitThread，记事本进程的主线程开始退出。

（7）执行`PspProcessDelete`和`MmDeleteProcessAddressSpace`函数，删除记事本与WinDBG进程的内核对象和进程空间。

每一步骤前的方括号内是函数执行的进程上下文，最后一步可能是在系统服务进程（svchost.exe）环境内执行的，也可能是在系统进程内执行的。

### 10.8.3　分离被调试进程

Windows XP允许被调试进程脱离（detach）调试对话并保持运行，也就是在保持调试器进程和被调试进程都不退出的前提下，将二者分离开来。在以前的版本（Windows 2000、Windows NT）中，调试器一旦与被调试进程建立调试关系，那么就只有当二者之一退出时，调试关系才结束，而且调试器退出会强制被调试程序也退出。

调试器可以使用Windows XP新引入的调试DebugActiveProcessStop API来分离调试对话，其原型如下。

```
BOOL DebugActiveProcessStop(DWORD dwProcessId);
```

也就是调试器只要指定要分离的被调试进程的ID，就可以与其终止调试关系了。利用这一特征，调试器可以附加到一个运行的进程，通过生成DUMP文件采集其内部的信息，然后再与其安全分离，也就是在基本上不影响目标进程的条件下采集目标进程的信息。这对于调试需要持续运行的进程（如数据库系统或其他系统服务）来说是非常重要的。

值得说明的是，必须在建立调试会话的线程中调用`DebugActiveProcessStop` API，否则会得到0xC0000022错误（LastError=5，访问被拒绝）。

与其他调试API一样，`DebugActiveProcessStop`函数也是从KERNEL32.DLL中导出的，其内部的工作过程是调用NTDLL.DLL中的`DbgUiStopDebugging`函数，其伪代码如清单10-20所示。

清单10-20　DebugActiveProcessStop函数的伪代码

```
BOOL DebugActiveProcessStop(DWORD dwProcessId)
{
   NTSTATUS nStatus;
   HANDLE hProcess = ProcessIdToHandle(dwProcessId);
   if(hProcess==NULL)
      return FALSE;

   CloseAllProcessHandles(dwProcessId);
   nStatus = DbgUiStopDebugging(hProcess);
   NtClose(hProcess);

   if(NT_SUCCESS(nStatus))
      return TRUE;

   SetLastError(5);
   return FALSE;
}
```

`DbgUiStopDebugging`函数的实现也非常简单，只是调用内核服务`NtRemoveProcessDebug()`。

```
NTSTATUS DbgUiStopDebugging(HANDLE hProcess)
{
    return NtRemoveProcessDebug(hProcess, NtCurrentPeb()-> DbgSsReserved[1]);
}
```

可见，`DbgUiStopDebugging` 向`NtRemoveProcessDebug`传递了两个参数：一个是被调试进程的句柄；另一个是调试器工作线程中保存着的调试对象句柄。`NtRemoveProcessDebug`内部会调用调试子系统的`DbgkClearProcessDebugObject`函数，简言之，后者用于去除调试进程的`DebugPort`字段对调试对象的引用，并将其设置为NULL。将`DebugPort`设置为空后，`DbgkClearProcessDebugObject`会遍历调试对象的调试事件队列，删除有关这个被调试进程的事件。

WinDBG调试器提供的分离被调试（detach debuggee）进程的功能就是使用以上方法实现的。

### 10.8.4　退出时分离

Windows XP引入的与分离调试会话有关的另一个功能就是可以设置当调试器退出时不强制退出被调试进程，也就是当调试器退出时分离被调试进程，而不是将其一起退出，这可以预防调试器意外终止导致被调试进程也被“杀掉”。

在上一段的介绍中，当退出调试器时，被调试的记事本程序也被强制退出了。然而，在调用终止进程的函数前，`DbgkpCloseObject`会检查调试对象的Flags字段是否包含`KillOnExit`标志，如果清除了这个标志，那么便不会退出被调试进程，`DebugSetProcessKillOnExit()`API就是用来设置这个标志的。

```
BOOL DebugSetProcessKillOnExit(BOOL KillOnExit);
```

如果`KillOnExit`参数为真，那么调试器线程的退出会导致系统终止被调试进程；否则，调试器线程的退出不会导致它所调试的被调试进程终止。有3点值得注意：第一，这里说的调试器线程是指调试器进程中启动调试会话的那个线程；第二，对于这里说的是线程，即使调试器进程中有其他线程仍然在运行，只要启动调试会话的线程退出了，那么对应的被调试程序就会退出（假设没有调用过DebugSetProcessKillOnExit）；第三，如果调试器同时调试多个进程，那么这个设置会影响所有进程，因为这个标志是设置在调试对象中的。

## 10.9　本章总结

本章按照创建调试会话、使用调试会话和终止调试会话的顺序深入地介绍了用户态调试的整个过程。我们将在第11章开始本篇的另一个主题，即Windows操作系统中的异常分发和处理。
