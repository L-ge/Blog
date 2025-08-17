# 第4章　启动过程

前面两章从空间角度浏览了Windows系统的软件世界，或者说我们给Windows系统拍了一张照片，并介绍了照片里的空间布局和人物角色。本章将介绍Windows系统的启动过程，目的是从时间角度描述Windows世界的搭建过程。如果把Windows系统的每个部件比作一个演员，那么本章将介绍这些演员如何快速适应演出环境，在尽可能短的时间里，披挂整齐，各就各位，开始表演。

我们将按时间顺序，分别介绍正常启动时的每个阶段，从BootMgr接管执行权到创建用户空间，每个阶段对应一节（4.1～4.6节）。

## 4.1　BootMgr

系统上电后，CPU首先执行固化在系统主板上的固件（firmware）代码，目的是检测和初始化基本的硬件，包括CPU、内存这样的核心硬件，以及键盘、显卡和磁盘等基本的输入/输出设备。

固件程序完成基本的初始化工作后，会加载操作系统的启动程序，然后把执行权移交给后者。在Windows Vista之前，承担这个任务的是NTLDR，其名字是NT Loader的缩写，意思是NT操作系统的加载程序。从Windows Vista开始，NTLDR的职责被拆分为3个模块——BootMgr、WinLoad和WinResume。简单来说，BootMgr负责从固件接管执行权和管理协调系统中的多个启动项，WinLoad负责加载指定启动项定义的操作系统实例，WinResume负责从上次休眠时产生的休眠文件恢复运行。

### 4.1.1　工作过程

BootMgr（即Windows Boot Manager）会从系统的引导配置数据（Boot Configuration Data，BCD）中读取启动设置信息，如果有多个启动选项，那么它会根据规则做选择，或者显示出启动菜单让用户来选择。

对于 Windows 8 或者更高版本的Windows系统，因为引入了蹩脚的图形界面启动程序（BootIM.exe），默认可能禁止了经典启动菜单，但是可以通过在具有管理员权限的命令行窗口中执行如下命令来启用：

```
bcdedit /set bootmenupolicy          Legacy
```

清单4-1中的栈回溯是BootMgr显示启动菜单后等待用户输入的情景。

清单4-1　栈回溯

```
kd> kn
 # ChildEBP RetAddr  
00 00061e34 00432655 bootmgr!DbgBreakPoint
01 00061e44 00431c24 bootmgr!BlXmlConsole::getInput+0xe
02 00061e90 00402e8f bootmgr!OsxmlBrowser::browse+0xe0
03 00061e98 00402b5e bootmgr!BmDisplayGetBootMenuStatus+0x13
04 00061f10 004017ce bootmgr!BmDisplayBootMenu+0x174
05 00061f6c 00401278 bootmgr!BmpGetSelectedBootEntry+0xf8
06 00061ff0 00020a9a bootmgr!BmMain+0x278
WARNING: Frame IP not in any known module. Following frames may be wrong.
07 00000000 f000ff53 0x20a9a
08 00000000 00000000 0xf000ff53
```

栈帧#06中的BmMain便是BootMgr的32位代码的入口函数，栈帧04中的BmDisplayBootMenu是显示启动菜单的函数，栈帧#07和#08是在实模式中执行时的痕迹。

当用户选择一个启动选项后，BootMgr会开始准备引导对应的操作系统。如果计算机上有Windows XP或者更老的Windows系统，而且用户选择了这些项，那么BootMgr会加载NTLDR来启动它们。如果用户选择的是Windows Vista或者更高版本的Windows系统，那么BootMgr会寻找和加载WinLoad.exe。如果没有找到或者在检查文件的完整性时发现问题，那么BootMgr会显示出错误界面。

在成功加载WinLoad.exe后，BootMgr会为其做一系列准备工作，包括启用新的GDT和IDT，然后调用平台的相关控制权移交函数把执行权移交给WinLoad。在x86平台中，完成这一任务的是Archx86TransferTo32BitApplicationAsm函数。移交执行权后，BootMgr完成使命，WinLoad开始工作。

### 4.1.2　调试方法

不管是Checked版本还是Free版本，Windows Vista的BootMgr和WinLoad程序内部都集成了调试引擎。对于Free版本，默认是禁止的，使用时需要开启，具体做法如下。

如果要启用BootMgr中的调试引擎，那么应该在一个具有管理员权限的控制台窗口中执行如下命令。

```
bcdedit /set {bootmgr} bootdebug on
bcdedit /set {bootmgr} debugtype serial
bcdedit /set {bootmgr} debugport 1
bcdedit /set {bootmgr} baudrate 115200
```

以上命令使用1号串口作为主机和目标机之间的通信方式，如果使用其他方式，那么应该设置对应的参数。

如果要启用WinLoad程序中的调试引擎，那么应该先找到它所对应的引导项的GUID值，然后执行如下命令。

```
bcdedit /set {GUID} bootdebug on
```

启用调试引擎并连接通信电缆后，在主机端运行WinDBG工具，便可以进行调试了，可以像调试NT内核那样使用栈回溯、内存访问、寄存器访问等调试命令。

在Windows Vista之前，NTLDR是Windows操作系统的加载程序。因为只有Checked版本的NTLDR才支持调试，所以如果要调试加载阶段的问题，应该先将NTLDR替换为Checked版本。DDK中通常包含Checked版本的NTLDR程序。记住，在替换前，应该先去除NTLDR文件的系统、隐藏和只读属性，在替换后，要加上这些属性，否则引导扇区中的代码会报告NTLDR is missing错误，无法继续启动。

除了加载内核和引导类型的驱动程序，NTLDR会调用NTDETECT.COM来做基本的硬件检查并收集硬件信息。NTDETECT会把收集到的信息存放到注册表中。如果找不到NTDETECT.COM，那么通常会直接重启。如果NTDETECT发现系统缺少必须的硬件或固件支持，比如ACPI支持，那么会显示因为硬件配置问题而无法启动。对于这样的问题，可以尝试更改BIOS选项来解决，或者通过调试NTLDR来进一步定位错误原因。

## 4.2　WinLoad

WinLoad的主要任务是把操作系统内核加载到内存中，并为它做好“登基”准备。WinLoad首先要做的一件事就是启用CPU的分页机制，进一步改善运行环境。然后初始化自己的支持库，如果启用了引导调试支持（4.2节），那么它会初始化调试引擎。

接下来WinLoad会读取启动参数，决定是否显示“高级启动”菜单，高级菜单中含有“以安全模式启动”等选项。如果用户按了F8键或者上次没有正常关机，那么WinLoad便会显示“高级启动”菜单。

接下来要做的一个重要工作是读取和加载注册表的System Hive，因为其中包含了更多的系统运行参数，负责这项工作的是OslpLoadSystemHive函数。

做好以上工作后，WinLoad开始完成它的核心任务，那就是加载操作系统的内核文件和引导类型的设备驱动程序。它首先加载的是NTOSKRNL.EXE，这个文件包含了Windows操作系统的内核和执行体。此时真正的磁盘和文件系统驱动程序还没有加载进来，所以WinLoad是使用它自己的文件访问函数来读取文件的。例如，FileIoOpen函数是用来打开文件的，如果FileIoOpen打开文件失败，那么调用它的BlpFileOpen 函数会返回错误码0C000000Dh；否则，返回0，代表成功。

接下来加载的是硬件抽象层模块HAL.DLL、支持调试的KDCOM.DLL，以及它们所依赖的模块。如上一章所介绍的，内核直接依赖的模块一般包括PSHED.DLL（第17章）、BOOTVID.DLL（用于引导期间和发生蓝屏时的显示）、CLFS.SYS（支持日志的内核模块）和CI.DLL（用于检查模块的完整性）。

加载系统模块后，WinLoad还需要加载引导类型（Boot Type）的设备驱动程序。在安装驱动程序时，每个驱动程序都会指定启动类型（Start Type），这个设置决定了驱动程序的加载时机，引导类型的驱动程序是由WinLoad加载到内存中的。

如果在加载以上程序模块或者注册表的过程中找不到需要的文件或者在检查文件的完整性时发现异常，那么WinLoad便会提示错误，停止继续加载。

完成模块加载后，WinLoad开始准备把执行权移交给内核，包括为内核准备新的GDT和IDT（OslArchpKernelSetupPhase0），以及建立内存映射（OslBuildKernelMemoryMap）等。所有准备工作做完后，WinLoad调用OslArchTransferToKernel函数把供内核使用的GDT和IDT地址加载到CPU中，然后调用内核的入口函数，正式把控制权移交给内核。

Linux系统常用的BootLoader是Grub，只能靠打印消息调试，NTLDR虽然支持调试器，但很难使用，BootMgr和WinLoad将强大的KD集成进来，让调试BootLoader变得非常轻松惬意。

## 4.3　内核初始化

前面两节分别介绍了Windows系统的启动管理程序BootMgr和操作系统加载程序（OS Loader）WinLoad。简单说，BootMgr会根据规则或者用户选择加载合适的WinLoad，WinLoad会把内核初始化早期所需的NT内核模块、内核模块的依赖模块以及引导类型的驱动程序等模块加载到内存中，并为内核开始执行做好准备。一切准备就绪后，WinLoad会把执行权移交给内核模块的入口函数，于是NT内核模块就开始执行了。内核模块开始执行，标志着计算机系统的统帅走马上任，“漫长的”启动过程进入了一个新的阶段。虽然前面已经做了很多准备工作，但是对于一个典型的多任务操作系统来说，要搭建一个可以运行各种应用程序的多任务环境，还有很多事情要做。接下来我们将分别介绍。

### 4.3.1　NT的入口函数

NT内核模块的入口函数名叫KiSystemStartup，意思是系统将从这里起步，从无到有，逐渐成长。

当调用KiSystemStartup时，WinLoad会将启动选项以一个名为LOADERPARAMETER BLOCK的结构传递给KiSystemStartup函数。微软在某些内核版本公开的符号文件中包含了这个结构的符号，在内核调试时可以观察到这个结构的详细定义，如清单4-2所示。

清单4-2　 `_LOADER_PARAMETER_BLOCK`结构的定义

```
0: kd> dt nt!_LOADER_PARAMETER_BLOCK
   +0x000 OsMajorVersion               : Uint4B
   +0x004 OsMinorVersion               : Uint4B
   +0x008 Size                         : Uint4B
   +0x00c OsLoaderSecurityVersion      : Uint4B
   +0x010 LoadOrderListHead            : _LIST_ENTRY
   +0x020 MemoryDescriptorListHead     : _LIST_ENTRY
   +0x030 BootDriverListHead           : _LIST_ENTRY
   +0x040 EarlyLaunchListHead          : _LIST_ENTRY
   +0x050 CoreDriverListHead           : _LIST_ENTRY
   +0x060 CoreExtensionsDriverListHead    : _LIST_ENTRY
   +0x070 TpmCoreDriverListHead        : _LIST_ENTRY
   +0x080 KernelStack                  : Uint8B
   +0x088 Prcb                         : Uint8B
   +0x090 Process                      : Uint8B
   +0x098 Thread                       : Uint8B
   +0x0a0 KernelStackSize              : Uint4B
   +0x0a4 RegistryLength               : Uint4B
   +0x0a8 RegistryBase                 : Ptr64 Void
   +0x0b0 ConfigurationRoot        : Ptr64 _CONFIGURATION_COMPONENT_DATA
   +0x0b8 ArcBootDeviceName        : Ptr64 Char
   +0x0c0 ArcHalDeviceName         : Ptr64 Char
   +0x0c8 NtBootPathName           : Ptr64 Char
   +0x0d0 NtHalPathName            : Ptr64 Char
   +0x0d8 LoadOptions              : Ptr64 Char
   +0x0e0 NlsData                  : Ptr64 _NLS_DATA_BLOCK
   +0x0e8 ArcDiskInformation       : Ptr64 _ARC_DISK_INFORMATION
   +0x0f0 Extension                : Ptr64 _LOADER_PARAMETER_EXTENSION
   +0x0f8 u                        : <unnamed-tag>
   +0x108 FirmwareInformation      : _FIRMWARE_INFORMATION_LOADER_BLOCK
   +0x148 OsBootstatPathName       : Ptr64 Char
   +0x150 ArcOSDataDeviceName      : Ptr64 Char
   +0x158 ArcWindowsSysPartName    : Ptr64 Char
```

KiSystemStartup开始执行后，首先会将参数结构的地址保存到全局变量KeLoaderBlock中，但值得说明的是，当内核启动结束后，内核会释放参数结构，并将KeLoaderBlock的值设置为0。因此，当系统启动后，再观察这个全局变量时，会看到这个变量的值为0。

```
0: kd> dq nt!KeLoaderBlock L1
fffff801`a705d2a8  00000000`00000000
```

接下来，KiSystemStartup需要进一步完善基本的执行环境，比如检测CPU特征和初始化CPU，设置中断描述符表，建立和初始化处理器控制区（PCR），建立任务状态段（TSS），设置用户调用内核服务的MSR等。在Windows 2000时代，上述这些初始化操作中，一部分是直接在KiSystemStartup函数体内做的，另一部分是调用HalInitializeProcessor函数完成的。在后来的版本中，这些初始化操作都被封装到一个名为KiInitializeBootStructures的函数中，这样调整后，KiSystemStartup的代码变得更加简洁，如清单4-3所示。

清单4-3　KiSystemStartup的代码（伪代码）

```
void KiSystemStartup(LOADER_PARAMETER_BLOCK* pLoaderParaBlock)
{
    KeLoaderBlock = pLoaderParaBlock;

    KiInitializeBootStructures(KeLoaderBlock);

    KdInitSystem();

    KiInitializeKernel();

    ExpSecurityCookieRandomData = <RDTSC>;

    KiIdleLoop();
}
```

在KiInitializeBootStructures中，会第一次调用内存管理器的初始化函数MmInitSystem，以-1为参数，代表只做最基本的初始化以满足启动阶段的内存分配需求。当KiInitializeBootStructures返回后，内核的执行环境已经改善了很多，不但可以动态分配内存，而且已经初始化了PCR、GDT、IDT、TSS等数据结构。有了这些基础后，接下来会调用KdInitSystem来初始化内核调试引擎。如果在内核调试时按Ctrl+Alt+K快捷键启用了初始断点，那么内核调试引擎初始化时便会中断到调试器，如清单4-4所示，这也意味着从此便可以使用内核调试器来调试后面的启动过程了。

清单4-4　内核调试引擎初始化时主动中断到调试器

```
kd> kc
# Call Site
00 nt!DebugService2
01 nt!DbgLoadImageSymbols
02 nt!KdInitSystem
03 nt!KiSystemStartup
```

### 4.3.2　内核初始化

做好以上基础工作后，接下来出场的是一个重量级的函数，名叫KiInitializeKernel，意思是初始化内核。这个名字虽然看起来有些夸大，但是事实上不算过分，因为后续的启动过程确实是由它发起和“导演”的。

因为内核调试引擎已经准备好了，所以在初始断点命中时，便可以执行`bp nt!KiInitializeKernel`以设置断点，然后执行g命令以恢复执行，断点很快会命中，而后便可以单步跟踪这个“导演”的行动了。

值得强调的是，在多CPU系统中，每个CPU唤醒后，都会执行KiInitializeKernel，但是最先执行的0号CPU（启动启动处理器）执行的动作会多一些。以笔者近期开发，使用的Windows 10 16299版本为例，KiInitializeKernel第一次执行时所做的主要工作如下。

（1）调用HvlPhase0Initialize检查是否运行在虚拟机中。如果运行在虚拟机中，那么会进一步检查虚拟机监视器（VMM）是否是微软公司的HyperV。如果是，会检查版本号，以便可以得到VMM的优待（HvlEnlightenments），以提高性能。

（2）调用KiDetectFpuLeakage检查与浮点指令有关的安全漏洞。

（3）调用KiSetPageAttributesTable初始化页机制使用的页属性表。

（4）调用KiConfigureInitialNodes和KiConfigureProcessorBlock检查系统的拓扑结构，配置初始节点和处理器块。

（5）调用KeAddProcessorAffinityEx。

（6）调用KeCompactServiceTable初始化系统服务表。

（7）调用KiSetCacheInformation初始化CPU的高速缓存信息。

（8）调用KiInitSystem初始化内核部件（微观意义的内核）的一些全局数据结构，包括蓝屏回调函数链表（KeBugCheckCallbackListHead）、性能勘查器列表（KiProfileListHead），以及各种同步对象。

（9）调用HviGetHypervisorFeatures获取VMM的特征。

（10）调用KeInitializeProcess初始化全局结构KiInitialProcess所描述的初始进程。

（11）调用KiEnableXSave，通过设置IA32_XSS（索引为0xDA0）寄存器，设置CPU的浮点协处理器状态保存选项。

（12）调用KiInitializeIdleThread创建空闲线程。

（13）调用HalInitSystem为当前CPU做硬件抽象层的初始化。

（14）调用InitBootProcessor执行只需要启动处理器（0号）执行的动作。这个函数内部会做很多重要的准备工作，并且会依次调用每个执行体的初始化函数，让每个执行体都做基本的初始化，称为阶段0初始化，这将在4.4节中详细描述。

（15）调用KiCompleteKernelInit，内部执行多个动作，包括把一个初始化线程附加到系统进程（nt!PsInitialSystemProcess）中，以便等当前线程跳入空闲循环后，CPU会转去执行新的系统线程，开始新一轮的执行体初始化工作。此外，这个函数还通过KeInitializeTimer2注册KiForegroundTimerCallback定时器回调，通过KeInitializeDpc初始化两个延迟过程调用（Deferred Procedure Call，DPC）对象——KiProcessPendingForegroundBoosts和KiTriggerForegroundBoostDpc。

（16）调用KiIdleLoop进入空闲循环，永远不再返回。

上面的介绍包含了较多细节。为了帮助大家理解关键脉络，我们再简单归纳一下内核启动的关键过程。CPU进入内核的入口函数KiSystemStartup后，先做基本的初始化，然后调用KdInitSystem初始化内核调试引擎，而后调用KiInitializeKernel开始内核启动之旅。当0号CPU执行KiInitializeKernel时，KiInitializeKernel会调用KiInitSystem来初始化系统的全局数据结构，调用InitBootProcessor做只需要执行一遍的动作，包括创建和初始化空闲进程与系统进程。InitBootProcessor函数返回后，KiInitializeKernel的工作基本结束，调用KiCompleteKernelInit做收场工作，附加系统线程到系统进程中让其准备接班，自己便调用KiIdleLoop进入空闲循环休息了。原书P79的图4-3归纳了NT内核的启动过程，分为左右两个部分。图4-3（a）为发生在初始进程中的过程，这个初始进程就是后来的空闲进程。图4-3（b）为发生在系统（System）进程中的过程，即所谓的执行体阶段1初始化，这将在后面单独介绍。

从线程的角度来看，启动早期还不具备多线程执行条件，很多初始化工作必须在设施很不完善的情况下来做，因此放在初始进程中，只让0号CPU以单线程方式来执行。内核中的执行体有很多个，分别承担某一方面的任务，做好自己的工作。所谓的执行体初始化，就是依次调用每个执行体的初始化函数，让它们做好准备。考虑到执行体之间可能是有依赖的，所以NT内核设计了分阶段初始化的策略，让执行体逐步初始化。所谓阶段0初始化，就是做基本的初始化，执行的动作尽可能不要依赖其他执行体。

阶段0初始化完成后，已经有了系统进程，并且具备了多线程执行能力，之后复杂的阶段1初始化动作便在系统进程中以系统线程的方式并行执行，这也是为了加快启动速度。

## 4.4　执行体的阶段0初始化

4.3节对整个内核初始化过程进行了宏观介绍，本节回过头来进一步介绍执行体的阶段0初始化。

首先，这个工作只需要执行一次，用于为后面的阶段1初始化做必需的准备动作，所以它是而且只是由启动CPU执行的。

根据第3章，如果把操作系统看作一个国家，那么执行体便是这个国家的各个行政机构。典型的执行体部件有进程管理器、对象管理器、内存管理器、I/O管理器等。

考虑到各个执行体之间可能有相互依赖关系，所以每个执行体会有多次初始化机会，一般是两次。第一次通常是做不依赖其他执行体的基本初始化，第二次做可能依赖其他执行体的动作。通常把前者叫阶段0初始化，后者叫阶段1初始化。

### 4.4.1　总体过程

在Windows XP时代，内核中有一个ExpInitializeExecutive函数，它内部依次调用各个执行体的初始化函数，比如调用MmInitSystem构建页表和内存管理器的基本数据结构，调用ObInitSystem建立名称空间，调用SeInitSystem初始化令牌（token）对象，调用PsInitSystem对进程管理器做阶段0初始化，调用PpInitSystem让即插即用管理器初始化设备链表。在Windows 10中，InitBootProcessor 取代了ExpInitializeExecutive，比如，清单4-5显示的便是InitBootProcessor调用进程管理器的阶段0初始化函数（PspInitPhase0）时的情景。

清单4-5　InitBootProcessor调用进程管理器的阶段0初始化函数时的情景

```
# Call Site
00 nt!PspInitPhase0
01 nt!InitBootProcessor
02 nt!KiInitializeKernel
03 nt!KiSystemStartup
```

内核的全局变量InitializationPhase用来记录当前是哪一阶段的初始化，0即代表阶段0初始化。

```
dd nt!InitializationPhase L1
kd> dd nt!InitializationPhase L1
fffff801`ab006468  00000000
```

接下来，为了避免歧义，仍以上一节使用的Windows 10 16299版本为例来介绍。在调试器中可以观察到InitBootProcessor所做的主要动作如下。

（1）解析内核启动参数字符串，寻找是否包含用于测试和验证使用的选项，比如“PERFMEM”“BURNMEMORY”“FORCEGROUPAWARE”等。

（2）调用RtlInitNlsTable和RtlResetRtlTranslations初始化支持多语言的设施。

（3）调用WheaInitializeServices初始化WHEA服务，第三篇将详细介绍WHRA。

（4）以参数0调用HalInitSystem。

（5）调用KeInitializeClock，设置CPU时钟中断，判断是否启用动态时钟。如果不启用，会把原因写在KiDynamicTickDisableReason变量中。时钟中断是系统的脉搏，是内核执行很多常规操作所依赖的，比如检查是否有内核调试中断请求、是否需要切换线程等。动态时钟的目的是让系统空闲时减少时钟中断的次数，以便降低系统的功耗，与Linux系统中的Zero Tick或者Tickless技术类似。

（6）调用PsInitializeQuotaSystem初始化配额管理设施。

（7）调用CmGetSystemControlValues读取注册表的系统控制数据。

（8）调用KeInitializeTimerTable初始化定时器表。

（9）调用ExComputeTickCountMultiplier计算时钟计数器的换算因子。

（10）调用ExInitSystem对执行体的运行时库做初始化。

（11）调用KeNumaInitialize初始化NUMA（非对称内存架构）有关的设施。

（12）调用VerifierInitSystem初始化内核验证器（详细介绍在第三篇）。

（13）再次调用MmInitSystem，让内存管理器进行初始化。在这次初始化过程中，内存管理器会调用MiReloadBootLoadedDrivers来“重新”加载WinLoad加载到内存中的引导类型驱动程序。

（14）调用HalInitializeBios初始化与固件有关的信息。

（15）调用InbvDriverInitialize初始化系统自带（inbox）的显示驱动程序。

（16）如果启用了内核调试，则反复调用DbgLoadImageSymbols向内核调试器发送关于WinLoad所加载模块的加载通知。

（17）调用HeadlessInit。

（18）调用BootApplicationPersistentDataInitialize处理早期启动程序（比如固件）希望持久化的数据。因为某些启动程序没有磁盘这样的持久存储设施，所以在ACPI标准中定义了接口让操作系统来帮助固件保存某些信息。

（19）调用HalQueryMaximumProcessorCount查询当前处理器组所支持的逻辑处理器总数。

（20）调用ObInitSystem初始化对象管理器。

（21）调用SeInitSystem，让负责管理系统安全的安全管理器初始化。

（22）调用PspInitPhase0，让进程管理器做阶段0初始化，考虑到这个函数对学习操作系统和调试的重要性，我们将在下文单独介绍它。

（23）调用DbgkInitialize初始化用于支持用户态调试的内核设施。

### 4.4.2　创建特殊进程

上面按执行时间顺序介绍了InitBootProcessor所导演的执行体阶段0初始化过程，下面再进一步介绍一下其中的进程管理器的阶段0初始化，也就是PspInitPhase0的工作过程。首先，PspInitPhase0会注册一系列回调函数，目的是在对象管理器创建进程和线程等内核对象时得到执行机会。然后，会调用PsChangeQuantumTable调整线程调度器使用的时间片信息。

接下来，PspInitPhase0会初始化用于记录系统中所有进程的链表结构，并将这个链表的头结构地址记录到全局变量PsActiveProcessHead中。这一步完成后，我们才能在调试器中通过!process命令观察进程列表。

接着，PspInitPhase0会创建进程和线程对象类型。注意，这里创建的是内核对象类型。在NT内核中，每个内核对象都属于某一种类型。创建类型有点像是注册一个对象工厂，有了对象类型这个工厂后，后面才能创建指定类型的对象。

需要说明一下，在今天的Windows 10内核中，以全局变量的形式定义了一个进程“对象”，名叫KiInitialProcess。之所以给对象二字加上引号，是因为这个对象不是使用标准的对象创建方法创建的，而是直接使用定义静态变量的方法定义的。在4.3节介绍的内核初始化过程中，KiInitializeKernel会调用KeInitializeProcess函数初始化这个结构。清单4-6显示了在创建进程对象类型前观察KiInitialProcess时的情景。

清单4-6　创建进程对象类型前观察KiInitialProcess时的情景

```
kd> !process nt!KiInitialProcess
PROCESS fffff80022c23b40
    SessionId: none  Cid: 0000    Peb: 00000000  ParentCid: 0000
    DirBase: 001aa000  ObjectTable: ffffb481c2814040  HandleCount:   0.
    Image: System Process
    VadRoot ffffca0f66843df0 Vads 2 Clone 0 Private 8. Modified 0. Locked 0.
    DeviceMap 0000000000000000
    Token                             ffffb481c2817040
    ElapsedTime                       00:00:00.000
    UserTime                          00:00:00.000
    KernelTime                        00:00:00.000
    QuotaPoolUsage[PagedPool]         0
    QuotaPoolUsage[NonPagedPool]      136
    Working Set Sizes (now,min,max)  (8, 50, 450) (32KB, 200KB, 1800KB)
    PeakWorkingSetSize                2
    VirtualSize                       0 Mb
    PeakVirtualSize                   0 Mb
    PageFaultCount                    8
    MemoryPriority                    BACKGROUND
    BasePriority                      0
    CommitCharge                      13

        THREAD fffff80022c25380  Cid 0000.0000  Teb: 0000000000000000   
Win32Thread: 0000000000000000 RUNNING on processor 0
        Not impersonating
        Owning Process            fffff80022c23b40  Image:      System Process
        Attached Process          N/A            Image:         N/A
        Wait Start TickCount      0              Ticks: 35 (0:00:00:00.546)
        Context Switch Count      0              IdealProcessor: 0     
        UserTime                  00:00:00.000
        KernelTime                00:00:00.000
        Win32 Start Address nt!KiIdleLoop (0xfffff80022977a60)
        Stack Init fffff80024a6cb90 Current fffff80024a6cb20
        Base fffff80024a6d000 Limit fffff80024a66000 Call 0000000000000000
        Priority 127 BasePriority 0 PriorityDecrement 0 IoPriority 0 PagePriority 5
        Child-SP          RetAddr           Call Site
        fffff800`24a6c320 fffff800`23032a45 nt!PspInitPhase0+0x258
        fffff800`24a6c490 fffff800`22c2c803 nt!InitBootProcessor+0x6a5
        fffff800`24a6c6d0 fffff800`22c2b1cf nt!KiInitializeKernel+0x433
        fffff800`24a6c9d0 00000000`00000000 nt!KiSystemStartup+0x1bf
```

在真正创建进程之前，PspInitPhase0会继续做一些初始化工作，包括PspInitializeJobStructures、PspInitializeSiloStructures、ExCreateHandleTable和PspInitializeSystem PartitionPhase0。

以上工作都准备就绪后，PspInitPhase0 才调用PspCreateProcess创建第一个真正的进程对象，创建的进程ID总是4，代表系统进程。过程如下。

```
# Call Site
00 nt!PspCreateProcess
01 nt!PspInitPhase0
02 nt!InitBootProcessor
03 nt!KiInitializeKernel
04 nt!KiSystemStartup
```

清单4-7显示在调试器中观察到的刚刚创建的系统进程。

清单4-7　刚刚创建的系统进程

```
kd> !process ffffca0f668c3040
PROCESS ffffca0f668c3040
    SessionId: none  Cid: 0004    Peb: 00000000  ParentCid: 0000
    DirBase: 001aa000  ObjectTable: ffffb481c2814040  HandleCount:   1.
    Image: System Process
    VadRoot ffffca0f668562d0 Vads 2 Clone 0 Private 8. Modified 0. Locked 0.
    DeviceMap 0000000000000000
    Token                             ffffb481c2817040
    ElapsedTime                       00:00:00.000
    UserTime                          00:00:00.000
    KernelTime                        00:00:00.000
    QuotaPoolUsage[PagedPool]         0
    QuotaPoolUsage[NonPagedPool]      136
    Working Set Sizes (now,min,max)  (8, 50, 450) (32KB, 200KB, 1800KB)
    PeakWorkingSetSize                0
    VirtualSize                       0 Mb
    PeakVirtualSize                   0 Mb
    PageFaultCount                    0
    MemoryPriority                    BACKGROUND
    BasePriority                      8
    CommitCharge                      1

No active threads
```

接下来，PspInitPhase0会给这个新的进程赋予一个特殊的名字，叫System，并把这个进程对象的地址赋值给全局变量PsInitialSystemProcess，然后把KiInitialProcess的地址赋给PsIdleProcess。

```
fffff801`b744eb53 48890d8685bcff  mov     qword ptr [nt!PsInitialSystemProcess   
(fffff801`b70170e0)],rcx
fffff801`b744eb66 488b0dbb85bcff  mov     rcx,qword ptr [nt!PsIdleProcess (fffff801  
`b7017128)] ds:002b:fffff801`b7017128={nt!KiInitialProcess (fffff801`b7030b40)}
```

刚刚创建的系统进程中没有任何线程（注意上面信息中的No active threads）。接下来，PspInitPhase0会为该进程创建第一个线程，并将Phase1Initialization函数作为线程的起始地址。

```
fffff800`23041bed 488d051c78d6ff  lea     rax,[nt!Phase1Initialization (fffff800  
`22da9410)]
fffff800`23041bf4 baffff1f00   mov   edx,1FFFFFh
fffff800`23041bf9 4889442428   mov   qword ptr [rsp+28h],rax
fffff800`23041bfe 488d4de8     lea   rcx,[rbp-18h]
fffff800`23041c02 4c89742420   mov   qword ptr [rsp+20h],r14
fffff800`23041c07 e884c5d0ff   call  nt!PsCreateSystemThread (fffff800`22d4e190)
```

注意理解这一步，因为它衔接着系统启动的下一个阶段，即执行体的阶段1初始化，但是这里并没有直接调用阶段1的初始化函数，而是将它作为新创建的系统线程的入口函数。此时由于当前的中断请求级别（IRQL）比较高，因此这个线程还得不到机会执行。在KiInitializeKernel函数返回后，KiSystemStartup函数将当前CPU的IRQL降低到DISPATCH_LEVEL，然后跳转到KiIdleLoop()，退化为空闲进程中的第一个空闲线程。这样，当下次时钟中断发生、内核调度线程时，便会执行刚刚创建的系统线程，于是阶段1初始化便开始运行了，我们将在下一节继续介绍。

最后说明一下，虽然NT内核已经进入老年，但是像启动过程这样的逻辑始终还在不断调整。举例来说，在Windows XP时代，会在进程管理器的阶段0初始化（PsInitSystem）中为空闲进程创建一个真正的进程对象并把地址保存在PsIdleProcess变量中。但是在今天的Windows 10中，不再为空闲进程创建真正的进程对象，而是让其复用启动阶段使用的KiInitialProcess结构。出于这个原因，在调试器中，先使用!pcr命令从当前CPU的处理器控制区中得到空闲线程的ETHREAD结构地址，再使用!thread观察空闲线程的信息，可以看到它是附加在系统进程中的。

```
7: kd> !pcr
IdleThread: ffffb700cd64ccc0
7: kd> !thread ffffb700cd64ccc0
THREAD ffffb700cd64ccc0  Cid 0000.0000  Teb: 0000000000000000 Win32Thread: 0000000000000000 RUNNING on processor 7
Not impersonating
DeviceMap                 ffffa70dc4018b60
Owning Process            fffff8015622fb40       Image:         Idle
Attached Process          ffffdf08234cf480       Image:         System
```

这意味着，虽然空闲线程属于空闲进程，但是它是生活在系统进程的进程空间中的。从多个方面考虑，这个改动都是合理的，代表着NT内核还在持续的改进之中。

## 4.5　执行体的阶段1初始化

宏观来看，执行体的阶段1初始化是启动过程中花时间最多的部分。为了加快启动速度，这个部分是以多线程并行的方式运行的。初始的线程便是系统进程中的第一个线程，它的工作函数名为Phase1Initialization。

### 4.5.1　Phase1Initialization

可以说，Phase1Initialization是执行体阶段1初始化的总导演。在Windows XP时代，这个函数非常冗长，内部要依次调用很多函数。在较新的NT内核中，把很多冗长的代码都转移到了一个新增的nt!Phase1InitializationDiscard中，目的是启动后这个函数所占的内存可以释放和回收。

重构后，在Phase1Initialization函数中，会依次调用如下几个函数。

```
nt!Phase1InitializationDiscard
nt!IoInitSystem
nt!Phase1InitializationIoReady
nt!MmFreeBootDriverInitializationCode
```

从CPU角度来看，最初执行Phase1Initialization函数的仍是0号CPU，而且目前只有一个CPU，调用过程如下。

```
00 nt!Phase1Initialization
01 nt!PspSystemThreadStartup
02 nt!KiStartSystemThread
```

栈帧01～02表示在系统线程中执行。Phase1InitializationDiscard被调用后，它会先做一些初始化工作，比如调用HalInitSystem又一次给HAL初始化的机会，调用KeInitializeClock初始化时钟，以及初始化系统时间等。而后，一个大的动作开始了，那便是唤醒其他沉睡的CPU。

### 4.5.2　唤醒其他CPU

在前面两节介绍的各个过程中，不管系统中实际安装了多少个CPU，只有其中的0号CPU在工作。简单来说，在启动早期，只有0号CPU一个处理器在执行启动任务，其他处理器处于睡眠状态，因此，0号CPU也常称为启动处理器（boot processor）。

为什么如此设计呢？一个简单的原因就是在内核启动早期，负责管理多任务的任务管理器还没准备好，不具备并行运行多个线程的基本条件，有多个CPU，也没办法同时工作。

当系统已经完成了内核初始化和执行体的阶段0初始化后，系统中已经有了空闲进程和系统进程，也有了系统线程。多任务并行执行的条件成熟，到了唤醒其他CPU的时刻。在NT内核中，执行这个唤醒同伴任务的函数叫作KeStartAllProcessors（KSAP），调用过程如下。

```
# Call Site
00 nt!KeStartAllProcessors
01 nt!Phase1InitializationDiscard
02 nt!Phase1Initialization
03 nt!PspSystemThreadStartup
04 nt!KiStartSystemThread
```

KSAP调用HAL的HalEnumerateProcessors来枚举系统里的所有处理器。对于枚举到的其他每个处理器，启动处理器作为老大哥会为即将走上工作岗位的小弟准备好几样必备的家当，如下所示。

（1）用于处理异常和中断的IDT。

（2）用于记录每个CPU状态和重要属性的处理器控制区，一般称为PCR（Processor Control Region），或者PRCB（Processor Resource Control Block）。这个内存区好比CPU随身携带的背包，里面放着自己最常用的各种信息。在调试内核时，我们可以使用!pcr命令观察当前CPU的这个特殊区域。

（3）负责处理NMI、双误、机器检查异常等特殊中断或者异常的专用栈。因为当发生这些事件时，CPU要切换到一个全新的线程上下文（使用新的栈）来执行。这个栈称为中断服务例程（Interrupt Service Routine，ISR）栈。

做好上述准备后，KSAP调用HalStartNextProcessor来唤醒一个新的CPU。然后重复上述过程，直到把所有应该唤醒的CPU都唤醒。可唤醒的CPU总数受当前系统的版本、许可协议等因素限制，在此不去深究。

### 4.5.3　非启动CPU的起步路线

值得说明的是，每个CPU都会从内核的入口处开始执行，也都会执行KiInitializeKernel这样的内核初始化函数，但只有第一个CPU会执行其中的所有初始化逻辑，包括全局性的初始化，其他CPU只执行与单个CPU相关的部分。比如只有0号CPU会调用和执行KiInitSystem。另外，初始化空闲进程的工作也只由0号CPU执行，因为只需要一个空闲进程，但因为每个CPU都需要一个空闲线程，所以每个CPU都会执行初始化空闲线程的代码。KiInitializeKernel函数通过参数来知道当前的CPU号。举例来说，在下面的过程中，1号CPU（1：kd中的1代表CPU编号，从0开始）被唤醒后，从KiSystemStartup开始执行，然后执行KiInitializeKernel，再调用KiInitializeIdleThread为自己创建空闲线程。

```
1: kd> kc
# Call Site
00 nt!KiInitializeIdleThread
01 nt!KiInitializeKernel
02 nt!KiSystemStartup
```

全局变量KeNumberProcessors用来维护系统中的CPU个数，其初始值为0。当0号CPU执行KiSystemStartup函数时，KeNumberProcessors的值刚好是当前的CPU号。初始化一个CPU后，这个全局变量会递增1。于是，当第二个CPU开始运行时，KiSystemStartup函数仍然可以从这个全局变量了解到CPU号，以此类推，直到所有CPU都开始运行。ExpInitializeExecutive函数的第一个参数也是CPU号，这个函数中的大多数代码块只需为0号CPU执行，或者说大多数执行体的阶段0初始化逻辑只要0号CPU执行一次就可以了，不需要其他CPU重复执行。

宏观来看，0号CPU先跑，跑到阶段1的初始化阶段，开始唤醒其他CPU，其他CPU醒来后，也从内核的入口开始跑，也会经过KiInitializeKernel这样的“关键路标”，但只是走马观花，不会像0号CPU那样面面俱到。比如，只有0号CPU会创建入口为Phase1Initialization的系统线程，其他CPU不会操心这件事，很快就返回KiInitializeKernel，然后又返回KiSystemStartup，之后就通过一个长跳转指令跳到KiIdleLoop，去执行它自己的空闲线程了。这时，0号CPU执行的Phase1Initialization通常已经创建了很多等待执行的线程，所以其他CPU并不会在空闲线程中停留多久，当它收到时钟中断并执行线程调度逻辑时，就会“奔赴前线”去执行其他线程了。此后，多个CPU便并肩战斗，一起“打理”这个系统了。

### 4.5.4　漫长的I/O初始化

虽然阶段1初始化要做很多事情，但是最花费时间的是I/O初始化，从某种程度上说，也就是建立设备树的过程，包括枚举系统中的各种设备——真实的硬件设备和虚拟的软件设备，并且为找到的设备加载和初始化驱动程序。对于总线驱动程序，I/O初始化会进一步枚举自己的子设备，然后加载和初始化驱动程序。

在调试内核时，可以使用!devnode命令来观察设备树，起初它只有根节点。

```
7: kd> !devnode
Dumping IopRootDeviceNode (= 0xffffcd8cd5df0d20)
DevNode 0xffffcd8cd5df0d20 for PDO 0xffffcd8cd5df1e40
  Parent 0000000000   Sibling 0000000000   Child 0xffffcd8cd5debd20   
  InstancePath is "HTREE\ROOT\0"
```

设备树的一级子节点是根据注册表中的配置创建的，注册表的路径如下。

```
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Enum\Root
```

今天的Windows系统要求固件支持ACPI（Advanced Configuration and Power Interface）标准，因此一级子节点中总是有ACPI_HAL。

ACPI_HAL的Service（服务）是\Driver\ACPI_HAL，这告诉了I/O管理器为这个“设备”安装ACPI驱动程序（ACPI.sys）。ACPI.sys上岗后，会读取系统固件中的设备表，枚举其中的所有设备，分别安装驱动程序，其中最重要的就是PCI总线设备，会触发加载PCI总线的驱动程序（PCLsys）。PCI驱动程序加载后，便会按照PCI协议枚举所有PCI设备，包括下面的子总线控制器，比如USB总线控制器。随着新设备的不断加入，设备树不断添枝加叶，长高长大。限于篇幅，本书不再详细讨论。

导演I/O阶段1初始化的函数名叫IoInitSystem，它内部又把任务分派给两个子函数——IoInitSystemPreDrivers和IopInitializeSystemDrivers，前者负责初始化内建的和启动类型的驱动程序，后者负责初始化系统类型的驱动程序。

### 4.5.5　更新进度

在Windows 2000时代，内核启动时，屏幕下方有一个进度条，表示启动的总进程。到了Windows XP时代，进度条不见了，但是内部仍会调用名为nt!InbvIndicateProgress的函数。这个函数的第一个参数便是完成的百分比，每次递增5%。

Windows 10（或者8）对上述逻辑做了重构。重构后，InbvIndicateProgress内部会判断一个函数指针，如果指针为空，则直接返回。这个函数的参数也改变了，不再传递进度的百分比。

在执行体的阶段1初始化结束前，Phase1Initialization会创建第一个使用映像文件创建的进程，即会话管理器进程（SMSS.EXE）。该进程肩负着建设一个繁荣美丽的用户空间的伟大使命。

## 4.6　创建用户空间

前面介绍了内核和执行体的初始化过程，讨论了执行体的阶段0初始化和阶段1初始化。至此，内核和执行体已经初始化了，驱动程序也已经加载了很多，“内核空间”可以说是准备好了。但是准备好这些对于最终用户来说还不够，因为光秃秃的内核是没办法直接用的，还需要用户接口，需要用户可以操作的界面。或者说，内核设施建立好后，还需要构建“用户环境”，这个环境可以接受用户输入和向用户输出运行结果，也就是要提供人机接口。在Windows系统中，通常把这样的一套人机交互环境称为一个登录会话（Logon Session），有时干脆简称为会话。Windows支持多个会话，每个会话有自己的输入/输出设备和“桌面”。本节将继续介绍NT系统建立会话的过程。

### 4.6.1　创建会话管理器进程

在执行体的阶段1初始化末期，负责这一工作的Phase1Initialization函数会着手创建会话管理器进程。这将是系统根据可执行文件创建的第一个进程，因为此前的空闲进程和系统进程都是根据进程的模样“捏造”出来的，而且它们只在内核模式运行，没有用户态部分，不是完整的进程。

Phase1Initialization首先要做的是为要创建的进程准备命令行参数，包括系统的路径、系统盘的盘符等。在Windows XP时代，Phase1Initialization会根据全局变量NtInitialUserProcessBuffer得到会话管理器进程的路径和文件名。在内核调试会话中，可以通过WinDBG的du命令来观察这个全局变量的内容：

```
lkd> du nt!NtInitialUserProcessBuffer
80544ef0  "\SystemRoot\System32\smss.exe"
```

准备工作做好后，Phase1Initialization会调用RtlCreateUserProcess发起创建进程的过程。当设置在RtlCreateUserProcess函数处的断点命中时，使用kn命令可以观察到Phase1Initialization调用这个函数创建会话管理器进程的过程（清单4-8）。

清单4-8　创建会话管理器进程的过程（Windows XP版本）

```
kd> kn
# ChildEBP RetAddr  
00 fc8d3818 806a3d9a nt!RtlCreateUserProcess
01 fc8d3dac 80582fed nt!Phase1Initialization+0x1059
02 fc8d3ddc 804ff477 nt!PspSystemThreadStartup+0x34
03 00000000 00000000 nt!KiThreadStartup+0x16
```

在Windows 10中，内核新增了一个名为StartFirstUserProcess的函数，它会创建会话管理器进程，过程如清单4-9所示。

清单4-9　创建会话管理器进程的过程（Windows 10）

```
00 nt!RtlpCreateUserProcess
01 nt!RtlCreateUserProcessEx
02 nt!StartFirstUserProcess
03 nt!Phase1InitializationIoReady
04 nt!Phase1Initialization
05 nt!PspSystemThreadStartup
06 nt!KiStartSystemThread
```

如果创建会话管理器进程失败，那么会引发0x6F号蓝屏（SESSION3INITIALIZATION FAILED）。

接下来，内核通过等待SMSS进程句柄的方式来确保它在持续运行，如果等待超时，那么表明SMSS进程还在运行；否则，可能SMSS进程意外退出了，Phase1Initialization会通过引发0x71号蓝屏（SESSION5_INITIALIZATION_FAILED）进行报告。在等待SMSS几秒并且没有发现它退出后，Phase1Initialization便确信SMSS进程正常启动了。

至此，Phase1Initialization完成阶段1初始化的导演工作，可以功成身退了。在Windows XP时代，Phase1Initialization完成使命后并不会返回，因为一旦返回，便退出这个线程了。系统线程会调用MmZeroPageThread开始另一个事业，专门负责把已经释放的空闲内存页清零。这有点像一个从战场上退役的军人，改变角色后继续发光发热。出于这个原因，在启动的系统中，我们始终可以观察到当初导演阶段1初始化的系统线程，它一直是系统进程的第一个线程，清单4-10显示了这个线程的执行过程。

清单4-10　第一个系统线程的执行过程

```
80d86c00 8309fc6d nt!KiSwapContext+0x26 (FPO: [Uses EBP] [0,0,4])
80d86c38 8309ead3 nt!KiSwapThread+0x266
80d86c60 8309e7b1 nt!KiCommitThreadWait+0x1df
80d86cbc 830d3c99 nt!KeDelayExecutionThread+0x2aa
80d86d44 831c7452 nt!MmZeroPageThread+0x1e5
80d86d50 8322bd6e nt!Phase1Initialization+0x14
80d86d90 830cd159 nt!PspSystemThreadStartup+0x9e
00000000 00000000 nt!KiThreadStartup+0x19
```

Windows 10改变了上述行为，Phase1Initialization完成任务后便返回，导致这个线程退出，在系统中消失。

会话管理器程序的名字叫SMSS，全称是Session Manager SubSystem（会话管理器子系统）。

### 4.6.2　建立环境子系统

SMSS进程的启动代表着一个新的阶段开始了，它肩负着内核的委托，要营造出一个繁荣的“用户”世界。SMSS在完成了自身的初始化工作后，会创建一个\SmApiPort的LPC端口对象，用于对外提供服务。切换会话和建立新会话的请求都是通过这个端口发送给SMSS的。

接下来SMSS要做的是完成注册表中为其安排的任务，主要有两类。第一类是完成悬而未决的文件删除和改名任务。注册表中用于配置会话管理器的表键，即My Computer\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Session Manager。

左侧的PendingFileRenameOperations子键就是用来存放需要删除的文件清单的。杀毒软件和反安装程序经常使用这个表键来删除难以直接删除的病毒程序或者当时在使用的模块，因为SMSS启动时，其他用户态进程还没有运行，所以不会因为其他程序使用某个模块而无法删除它。

第二类是执行BootExecute表键下定义的命令，通常这里定义的是磁盘检查程序，即autochk.exe。清单4-11所示的栈回溯记录了SMSS创建AUTOCHK进程的过程。

清单4-11　栈回溯

```
# ChildEBP RetAddr  
00 fc13dd3c 804e406b nt!NtCreateProcess
01 fc13dd3c 7c90eb94 nt!KiFastCallEntry+0xf8
02 0015f788 7c90d760 ntdll!KiFastSystemCallRet
03 0015f78c 7c93134b ntdll!NtCreateProcess+0xc
04 0015f830 4858690c ntdll!RtlCreateUserProcess+0x125
05 0015f8b0 48586b79 smss!SmpExecuteImage+0x97
06 0015fe0c 48588588 smss!SmpInvokeAutoChk+0x12c
07 0015fe48 48588c4e smss!SmpExecuteCommand+0x53
08 0015fecc 48588f27 smss!SmpLoadDataFromRegistry+0x2f1
09 0015ff18 48589bfc smss!SmpInit+0x1bd
0a 0015ffa8 4858ad97 smss!main+0x68
0b 0015fff4 00000000 smss!NtProcessStartup+0x1d2
```

接下来SMSS要做的一项重要任务是建立虚拟内存机制所需的页面交换文件（paging file）。

之后，SMSS会开始建立环境子系统（enviroment subsystem）。子系统的信息定义在SubSystems表键下，通常包含Posix和Windows等子系统，其中Windows子系统是必须启动的。于是，SMSS会先加载Windows子系统的内核部分，即Win32K.sys，然后根据SubSystems表键下Windows键值的内容来创建Windows子系统的服务器进程，即CSRSS.exe。这个键值的典型格式如下。

```
C:\WINDOWS\system32\csrss.exe ObjectDirectory=\Windows SharedSection=1024,3072,512  
Windows=On SubSystemType=Windows ServerDll=basesrv,1 ServerDll=winsrv:UserServer  
DllInitialization,3 ServerDll=winsrv:ConServerDllInitialization,2 ProfileControl=  
Off MaxRequestThreads=16
```

ObjectDirectory开始的部分是传递给CSRSS进程的命令行参数，其中ServerDll是CSRSS进程要加载的服务模块。某些病毒和恶意软件可能利用这个特征把自己的模块加进来，以便加载到CSRSS进程中。

### 4.6.3　创建窗口站和桌面

建立Windows子系统后，SMSS会创建另一个关键的进程——WinLogon，它是系统中负责安全登录工作的核心部件，掌控着登录、重启和关机等重要的系统行为，屏幕保护程序也是由它来启动的。

WinLogon首先要做的一个重要工作就是创建0号窗口站（WinStat0）和默认的桌面（default desktop）对象。窗口站是会话的下一层组织结构。一个会话中可以有多个窗口站，但同一时刻每个会话中只能有一个窗口站可以与用户交互。每个窗口站有自己的剪切板，可以有多个桌面。原书P91的图4-7显示了一个典型的Windows XP系统中的窗口站和桌面信息，图中的信息是使用Desktop Heap Monitor工具产生的，可以从微软网站下载这个工具。

WinLogon需要调用Windows子系统内核模块（Win32K）的服务来创建窗口站。清单4-12中的栈回溯就是WinLogon进程调用Win32K的NtUserCreateWindowStation函数创建窗口站的过程。

清单4-12　栈回溯

```
# ChildEBP RetAddr  
00 fc68bd40 804e406b win32k!NtUserCreateWindowStation
01 fc68bd40 7c90eb94 nt!KiFastCallEntry+0xf8
02 0006f618 77d6a7d7 ntdll!KiFastSystemCallRet
03 0006f96c 77d6a5b8 USER32!NtUserCreateWindowStation+0xc
04 0006f98c 01030abc USER32!CreateWindowStationW+0x26
05 0006fcfc 0103112e winlogon!CreatePrimaryTerminal+0x130
```

创建窗口站后，WinLogon会调用NtUserCreateDesktop来创建桌面。它首先会创建一个名为WinLogon的桌面供自己使用，然后创建一个名为Default的桌面供应用程序使用。其执行过程如下。

```
# ChildEBP RetAddr  
00 fc68bd48 804e406b win32k!NtUserCreateDesktop
01 fc68bd48 7c90eb94 nt!KiFastCallEntry+0xf8
02 0006f914 77d6a898 ntdll!KiFastSystemCallRet
03 0006f94c 77d6a821 USER32!NtUserCreateDesktop+0xc
04 0006f984 01030ae9 USER32!CreateDesktopW+0x42
05 0006fcfc 0103112e winlogon!CreatePrimaryTerminal+0x15d
```

在创建桌面后，WinLogon会调用SetActiveDesktop函数将供自己使用的桌面设置为当前的活动桌面，于是登录桌面便呈现在用户面前。

当用户退出或者锁定屏幕（按Ctrl+Alt+Del组合键）时，WinLogon也会将自己的桌面切换到前台（设置为活动的），以防有人侵犯应用程序使用的桌面。因此，WinLogon自己使用的桌面又称为安全桌面。

创建桌面后，WinLogon会创建用于管理系统服务的服务管理器（Services.exe）和本地安全认证子系统（LSASS.exe）。服务管理器会启动系统中登记的各种服务程序。当所有需要启动的服务都启动后，系统中已经有很多个进程在运行了。

### 4.6.4　用户登录

如何接收用户的登录信息（比如用户名、密码或者指纹等）呢？简单来说，Windows Vista之前使用的是图形识别与验证（Graphical Identification and Authentication，GINA）模块，Vista引入了称为Credential Provider的模型，取代了GINA模块。不论是哪种方式，都需要与前面提到的LSASS进程进行交互，本书跳过其细节。当验证用户的登录信息后，WinLogon会将应用程序桌面激活。在Windows XP时代，应用程序桌面上默认会显示当前的壁纸图片，而在Windows 10中，什么都不显示。

接下来WinLogon会引发所谓的用户初始化动作，也就是执行注册表中以下键值中定义的命令：

```
HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon\UserInit
```

通常这个键值的内容是c:\windows\system32\userinit.exe，也就是UserInit程序。

UserInit启动后，会运行HKCU\SOFTWARE\Policies\Microsoft\Windows\System\Scripts和HKLM\SOFTWARE\Policies\Microsoft\Windows\System\Scripts表键下定义的登录脚本。接下来UserInit会启动操作系统的外壳（Shell）程序。它首先会在HKCU\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon表键中寻找Shell键值，如果没有找到，那么它会在HKEY_LOCAL_MACHINE下寻找。默认情况下，Shell键值的内容是Explorer.exe，也就是资源管理器程序。“开始”菜单和任务栏的界面对象都是资源管理器程序创建与维护的。Explorer运行后，一个图形化的操作界面便展现在用户面前了。

## 4.7　本章总结

本章介绍了Windows操作系统的启动过程，让前面几章介绍的系统部件一个个走上场，活动起来。原书P93的图4-10归纳了从CPU复位上电到操作系统外壳就绪的全过程。
