# 第6章　垫片

在Windows操作系统里，也有一种著名的系统机制，叫垫片（shim）。概而言之，它的作用是用来解决软件兼容问题的。

在软件兼容方面，Windows平台一直表现卓越，有着非常好的口碑。举例来说，笔者很喜欢的Visual C++ 6.0（简称VC6）是在1998年发布的，当时主流的Windows桌面版本还是基于16位代码的Windows 9x。20多年过去了，Windows系统演进了很多代，而VC6依然可以在今天的Windows 10中较好地运行。

与Linux平台上流行的源代码兼容方式不同，Windows系统上一直奉行的是二进制兼容。二进制兼容的一个关键问题是应用程序编程接口（API）兼容。20多年里，Windows系统做了无数次重构和升级，Windows系统的开发团队换了一批又一批，但是始终保持着二进制兼容的传统。

为了保持这个传统，微软做了非常多的工作，垫片机制就是其中之一。与机械世界中的垫片的功能类似，Windows的垫片机制也是为了让两个软件模块可以对接在一起。如果没有垫片，它们之间可能是有冲突的。有了垫片后，它们就可以一起工作了。

Windows XP引进了垫片机制，当时，这个机制只是针对用户空间的，旨在解决用户空间的第三方代码兼容问题，更正式的名字叫应用程序兼容引擎（Application Compatibility Engine，ACE）。Windows 8.1把这个机制扩展到内核空间，用来解决设备驱动程序有关的兼容问题，称为内核垫片引擎（Kernel Shim Engine，KSE）。本章先介绍ACE和KSE都依赖的垫片数据库（SDB），然后再分别介绍ACE和KSE。

## 6.1　垫片数据库

垫片机制是Windows系统的一个神秘机制，公开的资料很少。我们的“探索垫片机制之旅”从认识神秘的垫片数据库开始。

### 6.1.1　认识SDB文件

从Windows XP开始，Windows系统的主文件夹下便有一个名为apppatch的子文件夹，里面放着若干个以.sdb为后缀的二进制文件，它们便是神秘的垫片数据库文件，有时也叫应用程序兼容数据库（compatibility database）。

微软没有公开SDB文件的格式定义，使用一个名为sdb2xml的免费工具可以把SDB的内容输出为XML格式。清单6-1包含了从sysmain.sdb产生的sysmain.xml文件的一部分（完整文件在本书配套电子资源的ch206文件夹中）。

清单6-1　转化为XML的SDB信息（局部）

```
<EXE>
<NAME type="xs:string">Womcc.exe</NAME>
<APP_NAME type="xs:string">Windows优化大师</APP_NAME>
<VENDOR type="xs:string">鲁锦</VENDOR>
<EXE_ID type="xs:string" baseType="xs:base64Binary">{bfbb948e-2d4e-4b43-
b9d5-6af14ecd2be8}</EXE_ID>
<APP_ID type="xs:base64Binary"/>
<RUNTIME_PLATFORM type="xs:int">37</RUNTIME_PLATFORM>
<MATCHING_FILE>
<NAME type="xs:string">*</NAME>
<COMPANY_NAME type="xs:string">鲁锦</COMPANY_NAME>
<PRODUCT_NAME type="xs:string">Windows优化大师</PRODUCT_NAME>
<UPTO_BIN_PRODUCT_VERSION type="xs:long">1970324836974591</UPTO_BIN_PRODUCT_VERSION>
</MATCHING_FILE>
<SHIM_REF>
<NAME type="xs:string">WinXPSP2VersionLie</NAME>
<SHIM_TAGID type="xs:int">191322</SHIM_TAGID>
</SHIM_REF>
</EXE>
```

清单6-1中的信息来自SDB中的可执行（EXE）文件表，大半部分描述的是可执行文件的属性，比如文件名为Womcc.exe，所属的应用软件名为“Windows优化大师”，开发商名为“鲁锦”等。随后的MATCHINGFILE部分定义了匹配这个文件的规则，或者说过滤条件，其中的`UPTO_BIN_PRODUCT_VERSION`定义的是截至目前的最高版本号。

清单6-1的SHIM_REF部分描述的是应该为这个EXE应用的垫片，垫片的名称为WinXPSP2 VersionLie，垫片的ID为191322，使用这个ID可以在专门描述垫片的表里找到这个垫片的详细信息。

从微软网站可以免费下载一个名为应用兼容工具包（Application Compatibility Toolkit，ACT）的软件包，使用其中的兼容管理器也可以浏览SDB数据库的内容。找到与清单6-1相对应的内容后，可以看到垫片WinXPSP2VersionLie的详细描述。根据描述，可以知道WinXPSP2VersionLie垫片的作用是挂接GetVersion和GetVersionEx两个API，让它们总是返回Windows XP SP2的版本信息。

通过上面一个例子，我们对SDB文件的作用有了一些了解。概括来说，SDB文件中描述了数以千计的应用程序，有微软的，也有第三方的。这些程序出现在SDB中意味着它们在新版本的Windows系统中有这样那样的问题。由于这些软件已经发布出去，可能已经在用户手中，因此很难把它们全部升级和替换成新版本。因此，只能用垫片机制来缓解问题，当用户使用这些有问题的软件时，修补加载到内存中的程序。为此，垫片机制还有一个名字，叫“微软内存修正补丁”（Microsoft Fix-It In-memory Patch），与更换磁盘文件的一般补丁相区别。

因为这样的兼容性问题可能随时都会新增，所以使用SDB这样的数据库文件来存储，可以很方便地增添新的记录。仔细观察本机的C:\Windows\apppatch，可以看到sysmain.sdb的更新时间为2019年6月，距离笔者写作时间相距只有一个多月，这意味着这个文件最近被更新过。

C:\Windows\apppatch中，有多个SDB文件，还有子目录。简单说，根目录下的xxxmain.sdb是微软官方维护的，sysmain.sdb用于解决用户空间的应用程序问题，drvmain.sdb用于内核空间，msimain.sdb用于MSI安装包，pcamain.sdb供程序兼容助理（program compatibility assistant）使用。

### 6.1.2　定制的SDB文件

使用ACT工具集中的兼容管理员（compatibility administrator）程序可以创建新的SDB文件，这样的SDB文件一般称为用户定制的SDB文件。

创建了一个定制的SDB文件后，便可以向其中添加修补（fix）信息，也就是指定要修补的程序，为它确定匹配条件，选择修补方案。举例来说，从菜单栏选择Database → Create New → Application Fix，便可以调出Create new Application Fix向导。

使用向导很快便为本书配套的BadBoy小程序创建了一个定制数据库，我们选择了4个修补方案，每个方案可以看作一个垫片。

定制的SDB文件需要安装、注册才能生效，注册表的位置如下。

```
HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\AppCompatFlags\Custom
HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\AppCompatFlags\InstalledSDB
```

定制SDB文件的默认安装位置为C:\Windows\AppPatch\Custom\SDB和C:\Windows\AppPatch\Custom\Custom64\。

在微软的ACT工具集中，有一个名为sdbinst的命令行工具，用于安装SDB数据库，例如使用如下命令便将我们刚刚建立的gesdb数据库安装到系统中了。

```
C:\Windows\system32>sdbinst c:\dbglabs\gesdb.sdb
Installation of gesdb complete.
```

安装后，在上面的提到的AppPatch\CustomSDB目录下会新增一个以GUID命名的文件。

值得说明的是，因为垫片机制可能被黑客所利用，产生严重的安全问题，所以在Windows 10上运行上面的“兼容管理员”程序时，系统会显示“Application Compatibility Toolkit 无法在 Windows 上运行（此应用可能会降低电脑安全性或性能。请检查在此版本的Windows上运行的更新应用）”的提示信息并阻止运行。

不过，在具有管理员权限的命令行中，使用sdbinst命令可以很顺利地把在Windows 7上创建的SDB文件安装到Windows 10系统中。其实，我们前面描述的sdbinst gesdb.sdb命令就是在Windows 10系统上执行的。执行成功后，在CustomSDB目录下新增了下面这样一个新文件。

```
{c403b64f-e663-4de7-9823-32aa50943d34}.sdb
```

### 6.1.3　修补模式

在为有问题的程序定义修补方案时，除了可以像上面描述的一个个选择修补垫片，还可以选择包含一组垫片的修补模式。每个修补模式针对某个常见的应用场景而设计，包含若干个修补垫片。例如在原书P123的图6-6中，左侧的树形控件中列出了4个修补模式。其中，展开的Win2000Sp3模式包含了16个垫片（图中只列出部分）。为应用程序启用这个修补模式后，它就仿佛回到了老的Windows 2000 SP3环境中运行。

有多种方式可以启用修补模式。除了可以在ACT工具中启用，还可以不用任何特殊工具，在资源管理器的文件属性对话框中启用，只要勾选“以兼容模式运行这个程序”复选框即可。

归纳一下，本节从保存垫片信息的SDB文件开始，介绍了垫片机制的数据部分。概而言之，垫片信息是存储在专门的SDB文件中的。SDB文件中包含了很多个具有各类修补功能的垫片，还包含了被修补程序的信息，以及二者的关联信息。后续几节将继续介绍垫片机制的程序模块，也就是垫片机制的代码部分，旨在理解系统是如何从SDB文件中读取垫片信息并让其发挥作用的。

## 6.2　AppHelp

在今天的Windows程序进程中，经常可以看到一个名叫AppHelp的DLL。这个模块的命名模式与DbgHelp类似。从字面理解，DbgHelp是帮助调试的，AppHelp是帮助应用程序的。相对而言，AppHelp针对的麻烦比较大，要帮助应用程序解决兼容性的问题，没有这个帮助，程序可能就崩溃或者无法运行。简单说，AppHelp的名字取得有些低调，它的作用很大，要为应用程序提供“救命”服务。从软件架构的角度来讲，它是垫片机制的核心模块，内部包含了很多必需的功能，本节将分别加以介绍。

### 6.2.1　SDB功能

在AppHelp模块中，包含了300多个以Sdb开头的函数，它们就是操作垫片数据库（SDB）的数据库引擎。表6-1按照操作数据库的一般过程，列出了部分主要函数。

表6-1　AppHelp中的SDB访问函数

| 操　　作 | 函　数　名 |
| --- | --- |
| 打开数据库 | SdbOpenDatabase，SdbOpenDatabaseEx，SdbOpenDbFromGuid，SdbOpenLocalDatabase，
SdbOpenApphelpDetailsDatabase |
| 查询信息 | SdbQueryModule，SdbQueryName，SdbQueryApphelpInformation，SdbQueryContext |
| 读取标签和数据 | SdbReadBYTETag，SdbReadStringTag，很多SdbReadXXXTag，SdbReadApphelpData |
| 写标签和数据 | SdbWriteWORDTag，很多SdbWriteXXXTag |
| 关闭数据库 | SdbCloseDatabase，SdbCloseLocalDatabase |


SDB数据库是以标签的形式来组织数据的，与XML方式类似，所以SDB文件可以很方便地转化为XML，我们在6.1节使用的sdb2xml工具便是如此。

### 6.2.2　垫片引擎

在AppHelp中还包含了垫片设施的核心引擎，一般称为应用程序垫片引擎（Application Shim Engine，ASE），或者SE。

值得说明的是，在Windows 10的系统目录中，有一个名为SHIMENG.DLL的文件，从文件名和文件属性中的文件说明来看，它更像是真正的垫片引擎模块。

但其实，它只是一个空的外壳。在这个文件属性中，可以看到它的原始文件名为Shim Engine DLL（IAT）。名字后括号中的IAT是导入地址表（Import Address Table）的缩写，它道出了这个假垫片引擎的真实身份，它内部只是包含了一个导入表，没有实际实现，使用经典的Depends工具观察这个模块，可以看到这个模块的所有导出函数（Function列）的入口都位于AppHelp中（最后一列）。

垫片引擎的对外导出函数以SE_开头，其他函数都模仿NT内核的命名习惯，垫片引擎自己的内部函数以Sep开头，小写的p代表内部过程，公开给其他子功能使用的变量或者函数都以Se开头。表6-2按照功能列出了垫片引擎的重要函数。

从表6-2中可以看出垫片引擎定义了不同角色的“经理（Manager）”，帮助它管理某一个方面，比如挂钩管理、垫片管理、文件重定向管理、模块追踪、标志管理等。

名为g_Engine的全局变量记录着初始化后的垫片引擎结构。全局变量g_ShimDebugLog是与调试信息输出有关的，另一个全局变量g_ShimDebugLogLevel用于记录信息输出的级别。可以通过设置如下环境变量来开启垫片引擎的调试信息。

```
SHIMENG_DEBUG_LEVEL=9
```

成功启用调试信息输出后，使用DbgView或者WinDBG便可以接收到垫片工作函数打印的调试信息。

表6-2　垫片引擎的重要函数

| 操　　作 | 函　数　名 |
| --- | --- |
| 初始化 | SE_InitializeEngine |
| 查询信息 | SE_GetShimId，SE_GetHookAPIs，SE_GetMaxShimCount，SE_GetShimCount |
| 重定向器（Redirector） | SeFindRedirector，SeInitializeRedirectors |
| 挂钩管理器 | SeHookManagerCreate，SeHookManagerAddHooks，SeHookManagerResolveHooks |
| 垫片管理器 | SeShimManagerCreate，SeShimManagerAddShim，SeShimManagerGetShimDllList |
| 标志管理器 | SeFlagManagerCreate，SeFlagManagerExecute，SeFlagManagerAddFlag，
SeFlagManagerDelete |
| 怪癖（Quirk）管理器 | SeQuirkManagerCreate，SeQuirkManagerExecute，SeQuirkManagerDumpState |
| 模块追踪器 | SeModuleTrackerCreate，SeModuleTrackerLookup，SeModuleTrackerDelete |
| 常用操作 | SepIatPatch（修补IAT） |


### 6.2.3　AD挂钩

随着安全形势的日益严峻，Windows系统的安全防范也越来越多。粗略地说，系统管得越来越严，在老版本中可以访问的，在新版本中可能就不能访问了，这可能导致老的应用程序访问某些资源时被拒绝，收到`STATUS_ACCESS_DENIED`错误，无法工作。自从Windows Vista引入UAC（用户访问控制）机制后，这样的问题就更多了。

为了修补有这样问题的程序，AppHelp中专门设计了一个子模块，名叫AD，是ACCESS DENIED的缩写。

这个子模块里有一系列以`AdHook_`开头的函数，比如apphelp!AdHook_RegOpenKeyA，应该对RegOpenKey函数做修补，当应用程序调用这个API而收到AD错误时，它会采取补救措施。

如果补救措施被使用而且启用了调试信息输出，那么可以看到类似下面这样的调试信息。

```
"Access denied detector fires,%s"
```

全局变量`g_AdState`记录着AD挂钩的状态，另一个全局变量`g_AdFireState`用于记录挂钩被触发的情况，目的是评估挂钩的效果。

### 6.2.4　穿山甲挂钩

在AppHelp还可以看到一种名字很新颖的挂钩技术，名叫穿山甲挂钩（Armadillo Hook）。从名字来看，该技术可能用来处理普通挂钩难以应付的情况。全局数组g_ArmadilloHooks记录着这种挂钩。目前要挂接的函数有GetModuleFileName和GetModuleHandle。

概而言之，AppHelp是应用程序垫片设施的核心模块，它内部不但包括承担核心管理角色的应用程序垫片引擎，以及承担各种管理角色的管理器，而且包含了操作垫片数据库的数据库引擎。

## 6.3　垫片动态库

虽然在AppHelp中包含了一部分垫片的实现，但只是一少部分。更多的垫片是以动态库的形式实现的，根据需要动态加载。我们把专门实现垫片的DLL称为垫片动态库，它们大多以Ac开头，代表着它们的统一目标：为实现应用程序兼容而奋斗，让Windows应用程序可以持续运行“千秋万代”。本节将分别介绍Windows 10系统内建的垫片动态库。

### 6.3.1　AcLayers.DLL

这个垫片动态库中实现了容错堆（Fault Tolerant Heap，FTH）、虚拟注册表（Virtual Registry）、谎报版本（Version Lie），以及一系列与显示模式有关的垫片。下面分别介绍。

容错堆是Windows 7引入的一个兼容功能，目的是修补内存堆有关的错误。比如，有些程序可能多次释放内存，导致崩溃。当启用这个垫片后，应用程序调用HeapFree API 释放内存时会被重定向到垫片的钩子函数`NS_FaultTolerantHeap::APIHook_RtlFreeHeap`。

垫片的钩子函数得到调用后，会检查HeapFree的参数。如果发现释放的内存块地址有问题，则会报告如下格式的错误。

```
"bogus address: %p, %p."
```

如果发现这个内存块已经释放过，则会报告如下格式的错误。

```
"double free: %p, %p."
```

无论哪种错误，FTH都可以根据当前设置的策略来灵活应对，比如可以忽略错误，让程序继续运行。

值得说明的是，实现在AcLayers.DLL模块中的只是FTH的客户端，即NS_FaultTolerantHeap::FthClient。在系统的WDI服务中，运行着FTH的服务端。服务端运行着两个工作线程，分别如下。

```
FthServerMainThreadFunction
FthServerTrackingThreadFunction
```

FTH的客户端会通过下面名字的命名管道与服务端进行通信。

```
\Device\NamedPipe\ProtectedPrefix\LocalService\FTHPIPE
```

基于这样的“客户端/服务器”模型，FTH客户端工作在应用程序现场，在一线作战，服务器端工作在后台，二者配合，可以对应用程序实施很复杂的拯救行为。

除了容错堆，AcLayers.DLL中还实现了虚拟注册表，用于模拟“老的”注册表行为。比如，下面这些函数是用来建立不同版本的“假”注册表的：

```
AcLayers!NS_VirtualRegistry::BuildWin98SE
AcLayers!NS_VirtualRegistry::BuildArm64WOW
AcLayers!NS_VirtualRegistry::BuildIE60
AcLayers!NS_VirtualRegistry::BuildNT50
【省略很多】
```

AcLayers.DLL中还有很多垫片都属于同一类别，它们的目的就是对版本有关的API挂钩，谎报版本号，让应用程序取得老的版本号。比如下面是用于谎报Windows XP SP2版本的两个钩子函数。

```
AcLayers!NS_WinXPSP2VersionLie::APIHook_GetVersion
AcLayers!NS_WinXPSP2VersionLie::APIHook_GetVersionExA
```

类似的函数有很多，比如：

```
AcLayers!NS_WinXPSP3VersionLie::APIHook_GetVersion
AcLayers!NS_VistaRTMVersionLie::APIHook_GetVersionExA
```

此外，AcLayers.DLL还有与图形模式有关的一些垫片，比如NS_Force640x480，不再赘述。

### 6.3.2　AcGenral.DLL和AcSpecfc.DLL

从名字来看，AcGenral.DLL里面的垫片是比较通用的，与另一个垫片模块AcSpecfc.DLL是相对关系。以笔者写作本部分内容时使用的Windows 10为例，系统中有两个版本的AcSpecfc.DLL，一个是64位的，另一个是WoW版本的。前者较小，有339KB；后者较大，有2MB多（00253000）。

AcGenral.DLL内部包含了很多种垫片，比如针对堆错误的模拟堆`AcGenral!NS_EmulateHeap`，针对字体问题的`AcGenral!NS_FontMigration`，也有很多个用于谎报版本的VersionLie，比如`AcGenral!NS_Win2000SP1VersionLie`。

与针对比较普遍问题的AcGenral.DLL不同，AcSpecfc.DLL用来实现针对特殊问题的垫片，细节从略。

### 6.3.3　其他垫片模块

除了上面提到的垫片模块，Windows系统中内建的垫片模块还有几个，比如针对外部问题的AcXtrnal.dll，针对Windows 8引入的Windows Runtime API的AcWinRT.dll，以及针对UAC问题的AcLua.dll。所有垫片DLL都至少实现了两个导出函数，分别是GetHookAPIs和NotifyShims。下一节我们将介绍这些垫片模块的工作过程。

## 6.4　应用程序垫片的工作过程

前面3节分别介绍了垫片机制的数据库文件、核心引擎以及实现垫片的各个代码模块，前者是数据部分，后二者是代码部分，本节将继续介绍代码与数据如何结合到一起，发挥作用。

### 6.4.1　在父进程中准备垫片数据

一个新进程的生命是从父进程开始的，在父进程着手创建子进程的第一阶段（见2.9节）中，会为新进程准备各种材料。这其中就包括为新进程准备垫片数据。清单6-2所示的栈回溯记录了创建子进程的CreateProcessInternalW函数调用BasepGetAppCompatData获取垫片数据的过程。

清单6-2　栈回溯记录

```
00 apphelp!SdbInitDatabaseEx
01 apphelp!ApphelpCreateAppcompatData
02 KERNEL32!BaseGenerateAppCompatData
03 KERNEL32!BasepGetAppCompatData
04 KERNELBASE!CreateProcessInternalW
05 KERNELBASE!CreateProcessW
```

在清单6-2中，CreateProcessW是著名的CreateProcess API的Unicode版本，它调用创建进程的内部函数CreateProcessInternalW。后者在为子进程准备创建材料时，调用BasepGetAppCompatData获取应用程序兼容数据，也就是垫片数据。栈帧#01中的ApphelpCreateAppcompatData是我们介绍过的垫片引擎的接口函数，它再调用内部的SdbInitDatabaseEx来打开垫片数据库文件。

准备好的垫片数据会与新进程的其他数据一起提交给内核，内核会将这个数据映射到新进程的用户空间中，并将这个数据的起始地址通过PEB结构的pShimData字段传递给新进程。举例来说，下面是在WinDBG调试器里观察到的Badboy进程的结果，上面3行用于观察PEB结构中的pShimData字段，得到其内容为0x001b0000。后面3行是观察垫片数据块的结果。

```
0:000> dt _PEB 00304000 -y pShimData
ntdll!_PEB
   +0x1e8 pShimData : 0x001b0000 Void
0:000> dd 0x001b0000 
001b0000  00000fb8 ac0dedab 00000001 0000014c
001b0010  3000012e 00000000 00000000 00000000
```

### 6.4.2　在新进程中加载和初始化垫片引擎

在创建新进程的最后一个阶段，也就是新进程开始在自己的用户空间执行初始化工作时，NTDLL.DLL中的模块加载器（loader）会初始化垫片引擎，如清单6-3所示。

清单6-3　模块加载器初始化垫片引擎

```
00 ntdll!LdrpInitShimEngine
01 ntdll!LdrpInitializeProcess
02 ntdll!_LdrpInitialize
03 ntdll!LdrInitializeThunk
```

清单6-3中的LdrpInitializeProcess是个很长的函数，内部包含了很多逻辑，包括整理进程的参数，初始化异常信息，初始化用于记录关键操作过程的栈回溯数据库（RtlpInitializeStackTraceDatabase） （第23章），初始化堆管理器（RtlInitializeHeapManager），创建进程的默认堆，发起初始断点（LdrpDoDebuggerBreak）等。当然，还有我们现在正介绍的初始化垫片设施。

清单6-3中的LdrpInitShimEngine函数是NTDLL.DLL中发起初始化垫片引擎的重要函数，为了行文方便，我们给它取个简短的名字，叫LISE。

LISE内部会加载我们在6.2节介绍的AppHelp.DLL，加载成功后，会把这个模块的句柄（起始地址）记录在全局变量ntdll!g_pShimEngineModule中。

加载了垫片引擎模块后，LISE会调用LdrpGetShimEngineInterface动态获取垫片引擎的接口函数。后者会反复调用LdrGetProcedureAddressEx来获取垫片引擎模块内部的接口函数地址，主要是下面这些函数——SE_InstallBeforeInit、SE_InstallAfterInit、SE_DllLoaded、SE_DllUnloaded、SE_LdrEntryRemoved、SE_ProcessDying、SE_LdrResolveDllName、SE_GetProcAddressForCaller和ApphelpCheckModule。

从名字来看，这些接口函数中的大部分具有事件触发性质，也就是让NTDLL.DLL在某些事件发生前后来调用垫片引擎，给它机会施加“垫片”。

这些接口函数的地址保存到NTDLL.DLL中以g_pfn开头的一系列全局变量（即ntdll!g_pfnApphelpCheckModuleProc、ntdll!g_pfnSE_LdrResolveDllName、ntdll!g_pfnSE_LdrEntryRemoved、ntdll!g_pfnSE_GetProcAddressForCaller、ntdll!g_pfnSE_DllUnloaded、ntdll!g_pfnSE_InstallBeforeInit、ntdll!g_pfnSE_InitializeEngine、ntdll!g_pfnSE_ShimDllLoaded、ntdll!g_pfnSE_InstallAfterInit、ntdll!g_pfnSE_ProcessDying和ntdll!g_pfnSE_DllLoaded）中。

接下来，LISE会调用SE_InitializeEngine来给垫片引擎初始化的机会。后者会执行一系列初始化工作，包括调用SeProcessAttach准备具有全局性的层（layer）特征垫片，比如修改环境变量、模拟注册表等，如清单6-4所示。

清单6-4　垫片引擎初始化

```
00 apphelp!SeProcessAttach
01 apphelp!SE_InitializeEngine
02 ntdll!LdrpInitShimEngine
03 ntdll!LdrpInitializeProcess
04 ntdll!_LdrpInitialize
05 ntdll!LdrInitializeThunk
```

如果启用了垫片引擎的信息输出，那么可能看到类似下面这样的输出。

```
TRACE,SeSdbProcessLayers,481,Resetting layer env variable
```

### 6.4.3　加载垫片模块

在初始化垫片引擎后，LISE的下一个重大动作是加载具体的垫片模块。LISE会调用LdrpLoadShimEngine函数来做这个工作。LdrpLoadShimEngine也是垫片机制的重要函数，为了行文方便，我们将其简称为LLSE。需要说明的是，LLSE这个函数的名字有些误导，表面上看它加载垫片引擎，其实此时垫片引擎已经加载好了，它要加载的是我们在6.3节中介绍的垫片工作模块（AcGenral.DLL等）并发起部署垫片。

### 6.4.4　落实挂钩

如前几节所介绍，垫片的种类很多，但大多数垫片的基本原理是通过某种方式拦截API调用，改变程序的执行轨迹，实施修补策略。如何拦截到API呢？最主要的方法就是修补模块的导入地址表（IAT）。每个使用了API的模块（EXE或者DLL）都会有导入表，链接时建立好这张表，每个表项描述一个函数，地址部分虚在那里，留待运行期落实（resolve）。NTDLL.DLL中的加载器（LDR）部件负责在运行期落实IAT中的地址部分。在普通情况下，IAT的地址部分指向实际的API入口，在有垫片机制时，NTDLL.DLL会调用垫片引擎的SE_DllLoaded接口，让垫片模块“篡改”IAT，把表项中的地址指向垫片模块中的“假API”入口。

考虑到IAT操作的复杂度，让每个垫片模块分别修改IAT会导致代码重复，可靠性差。为此，垫片引擎会先收集各个垫片模块的挂接信息，然后统一实施挂接。收集动作是在LLSE函数调用垫片引擎的SE_InstallBeforeInit接口的时候完成的，如清单6-5所示。

清单6-5　收集垫片模块的挂钩信息

```
00 AcGenral!ShimLib::GetHookAPIs
01 apphelp!SeEngineInstallHooks
02 apphelp!SE_InstallBeforeInit
03 ntdll!LdrpLoadShimEngine
04 ntdll!LdrpInitShimEngine
05 ntdll!LdrpInitializeProcess
06 ntdll!_LdrpInitialize
07 ntdll!LdrInitializeThunk
```

值得说明一下，`SE_InstallBeforeInit`中的Init不是指垫片引擎的初始化，而是指NTDLL.DLL加载每个DLL后，会调用DLL的入口函数，让它执行自己的初始化动作，也就是所谓的DLL Init。清单6-5显示的是在系统调用每个DLL的初始化代码前，先通知垫片引擎，让它执行前期动作，此时，DLL自己的代码还没有执行过。稍后，NTDLL.DLL会调用`SE_InstallAfterInit`做后期动作。

调用了垫片引擎的`SE_InstallBeforeInit`接口后，LLSE会发送模块加载通知，反复调用垫片引擎的`SE_DllLoaded`接口，通知每个加载的DLL。在这个时机，垫片引擎会使用收集好的挂钩信息，实施挂钩。清单6-6记录的便是LDR中的LLSE函数调用垫片引擎（AppHelp）的模块加载接口、落实挂钩的过程。

清单6-6　加载接口、落实挂钩的过程

```
00 apphelp!SepIatPatch
01 apphelp!SepRouterHookImportedApi
02 apphelp!SepRouterHookIAT
03 apphelp!SE_DllLoaded
04 ntdll!LdrpSendShimEngineInitialNotifications
05 ntdll!LdrpSendShimEngineInitialNotifications
06 ntdll!LdrpLoadShimEngine
07 ntdll!LdrpInitShimEngine
08 ntdll!LdrpInitializeProcess
09 ntdll!_LdrpInitialize
0a ntdll!LdrInitializeThunk
```

落实挂钩后，如果再观察模块的导入表，便会看到其中的很多函数入口已经指向了垫片函数，清单6-7显示的是观察msvcrt.dll的导入表的部分结果。

清单6-7　落实挂钩后的导入表（部分）

```
0:000> dds msvcrt!_imp__HeapAlloc
73e5813c  78719c00 AcGenral!NS_EmulateHeap::APIHook_RtlAllocateHeap
73e58140  78719450 AcGenral!NS_EmulateHeap::APIHook_HeapValidate
73e58144  787190b0 AcGenral!NS_EmulateHeap::APIHook_HeapCompact
73e58148  78719530 AcGenral!NS_EmulateHeap::APIHook_HeapWalk
73e5814c  78718a00 AcGenral!NS_EmulateHeap::APIHook_GetProcessHeap
73e58150  78719e60 AcGenral!NS_EmulateHeap::APIHook_RtlSizeHeap
73e58154  78719cc0 AcGenral!NS_EmulateHeap::APIHook_RtlFreeHeap
```

在落实挂钩时，有个有趣的细节，垫片模块自己的DLL是不该安装挂钩的，否则便可能导致递归调用等故障，因此，垫片引擎会处理这种情况，排除自己的模块，不做挂钩，如果启用信息输出，那么可以看到下面这样的信息。

```
TRACE,SeRouterCanHookModule,1001,Module AcLayers.DLL is a shim module
TRACE,SepRouterHookIAT,1103,Excluding AcLayers.DLL from hooking
```

落实挂钩后，NTDLL.DLL中名为g_ShimsEnabled的全局变量会被设置为真（1），初始值为0。

接下来，LDR会调用DLL的初始化函数，这是Windows系统写在DLL协议里的标准动作，例如，清单6-8显示的是调用msvcrt.dll的初始化函数的过程。

清单6-8　调用msvcrt.dll的初始化函数的过程

```
00 msvcrt!__CRTDLL_INIT
01 ntdll!LdrxCallInitRoutine
02 ntdll!LdrpCallInitRoutine
03 ntdll!LdrpInitializeNode
04 ntdll!LdrpInitializeGraphRecurse
05 ntdll!LdrpInitializeShimDllDependencies
06 ntdll!LdrpLoadShimEngine
07 ntdll!LdrpInitShimEngine
08 ntdll!LdrpInitializeProcess
09 ntdll!_LdrpInitialize
0a ntdll!LdrInitializeThunk
```

执行了每个DLL的初始化过程后，LDR还会调用垫片引擎的SE_InstallAfterInit接口，再次给垫片引擎执行机会，做初始化后期的工作（清单6-9）。

清单6-9　调用垫片引擎的SE_InstallAfterInit接口

```
00 apphelp!SE_InstallAfterInit
01 ntdll!LdrpInitializeProcess
02 ntdll!_LdrpInitialize
03 ntdll!LdrInitializeThunk
```

### 6.4.5　执行垫片

有了前面描述的准备工作后，一个个垫片就都部署在它的岗位上了，好像渔夫下好了网，猎户挖好了坑（陷阱），就等着“猎物”进来了。

举例来说，当SHELL32模块调用GetVersionEx API时，它将要执行下面这条call指令。

```
call    dword ptr [SHELL32!_imp__GetVersionExW (75130d80)]
```

其中的`SHELL32!_imp__GetVersionExW`是编译器产生的导入符号，相当于函数指针，它本来指向的是KERNELBASE!GetVersionExW。

```
0:015> ln poi(75130d80)
(765ebc90)   KERNELBASE!GetVersionExW
```

现在因为启用了版本谎报垫片，它已经指向了AcLayers模块中的`NS_WinXPVersionLie::APIHook_GetVersionExW`。

```
0:000> ln poi(75130d80)
(009752b0)   AcLayers!NS_WinXPVersionLie::APIHook_GetVersionExW
```

于是，执行call指令后，CPU便开始执行垫片函数了，即：

```
00 AcLayers!NS_WinXPVersionLie::APIHook_GetVersionExW
01 SHELL32!AutoProviderRegistrar::AutoProviderRegistrar
```

## 6.5　内核垫片引擎

自从Windows XP引入垫片机制后，它便成为Windows操作系统中解决应用程序兼容问题的一个关键设施，取得了非常好的效果。Windows 8.1把垫片机制扩展到内核空间，用来解决设备驱动程序有关的兼容问题，称为内核垫片引擎（Kernel Shim Engine，KSE）。

### 6.5.1　数据和配置

从数据角度来说，KSE也使用SDB，主要的SDB文件叫drvmain.sdb，其默认路径如下。

```
C:\Windows\apppatch\drvmain.sdb
```

与用于应用程序的垫片数据库类似，我们仍然可以使用sdb2xml工具将二进制的SDB文件转化为xml格式，清单6-10截取了其中的一小部分。

清单6-10　驱动程序垫片数据举例

```
<KDRIVER>
    <NAME type="xs:string">qcmbb8960.sys</NAME>
    <APP_NAME type="xs:string">Qualcomm Wireless Network Device</APP_NAME>
    <VENDOR type="xs:string">Qualcomm</VENDOR>
    <EXE_ID type="xs:string" baseType="xs:base64Binary">{0d6e8f4f-11d1-4746-  
b724- 5122b14f2d7a}</EXE_ID>
    <MATCHING_FILE>
        <NAME type="xs:string">*</NAME>
        <UPTO_BIN_PRODUCT_VERSION type="xs:long">281479271677951</UPTO_BIN_PRODUCT_  
VERSION>
    </MATCHING_FILE>
    <KSHIM_REF>
        <NAME type="xs:string">SkipDriverUnload</NAME>
        <FIX_ID type="xs:base64Binary"/>
        <FLAGS type="xs:int">0</FLAGS>
        <MODULE type="xs:string">NT kernel component</MODULE>
    </KSHIM_REF>
</KDRIVER>
```

清单6-10中描述的是高通（Qualcomm）公司的无线网卡驱动——qcmbb8960.sys，MATCHING_FILE节点描述了匹配条件，KSHIM_REF节点描述了为这个驱动配置的垫片——SkipDriverUnload。从名字可以看出，这个垫片是要跳过驱动的卸载函数，也就是在卸载这个驱动程序时，不要执行驱动程序的卸载回调函数，可能一调用就有蓝屏之类的严重问题。

在系统启动时，Windows系统的加载程序WinLoad会通过OslpLoadMiscModules函数来加载SDB文件，并把加载到内存中的数据通过LOADER_PARAMETER_BLOCK_EXTENSION结构的DrvDBImage和DrvDBSize字段传递给内核。

除了SDB文件，KSE还支持从注册表中获取垫片数据，注册表的路径为\Registry\Machine\System\CurrentControlSet\Control\Compatibility\Device以及\Registry\Machine\System\CurrentControlSet\Control\Compatibility\Driver。

简单来说，Device表键用来针对设备来启用垫片，每个子键以设备的硬件ID命名，描述一个设备的垫片信息。Driver表键用来针对驱动程序启用垫片。举例来说，原书P133的图6-11所示的注册表信息是笔者写作本内容时所用Windows 10系统的默认设置。

在图6-11中，左侧的键名即代表驱动程序storahci.sys，右侧的Shims键值用来描述要启用的垫片，可以写多个，但是目前只指定了Srbshim。Srb是SCSI Request Block的缩写，代表存储设备驱动经常要处理的通信数据块。

### 6.5.2　初始化

在第4章中，我们详细介绍过Windows系统的启动过程，特别是内核部分的初始化过程。启动时，系统会分多个阶段来初始化执行体。I/O管理器是系统中非常庞大而且复杂的一个执行体。在I/O管理器执行阶段1初始化的时候，它会初始化KSE，其过程如清单6-11所示。

清单6-11　I/O管理器在阶段1初始化时初始化KSE

```
# Call Site
00 nt!KsepEngineInitialize
01 nt!KseInitialize
02 nt!IoInitSystemPreDrivers
03 nt!IoInitSystem
04 nt!Phase1Initialization
05 nt!PspSystemThreadStartup
06 nt!KiStartSystemThread
```

清单6-11中，栈帧#04中的Phase1Initialization是执行体阶段1初始化的“导演”函数，它内部会依次调用每个执行体的初始化函数，IoInitSystem便是I/O执行体的初始化函数。如我们在第4章所讲，I/O初始化是内核启动过程中用时最长的部分，有很多事情要做。栈帧#02中的IoInitSystemPreDrivers表示在做加载驱动之前的初始化，这与上一节介绍的DLL Init前期有些类似。栈帧#01中的KseInitialize便是KSE的初始化函数，它进一步调用内部函数KsepEngineInitialize。

除了初始化KSE本身，KseInitialize还会调用KseShimDatabaseBootInitialize来初始化SDB。

### 6.5.3　KSE垫片结构

微软定义了一个名为KSE_SHIM的结构来描述KSE垫片，其定义如下。

```
typedef struct _KSE_SHIM {
 _In_ SIZE_T Size; 
_In_ PGUID ShimGuid; 
_In_ PWCHAR ShimName; 
_Out_ PVOID KseCallbackRoutines; 
_In_ PVOID ShimmedDriverTargetedNotification; 
_In_ PVOID ShimmedDriverUntargetedNotification; 
_In_ PVOID HookCollectionsArray; // _KSE_HOOK_COLLECTION数组
} KSE_SHIM, *PKSE_SHIM;
```

在笔者调试的Windows 10 16299版本中，这个结构的大小为56字节，即Size字段的值0x38，共包含7个字段，每个字段都为8字节。

其中的HookCollectionsArray字段指向一个数组，每个元素的类型如下。

```
typedef struct _KSE_HOOK_COLLECTION { 
ULONG Type; // 0表示NT Export, 1表示HAL Export, 2表示Driver Export, 3表示Callback
PWCHAR ExportDriverName; // 若Type == 2 
PVOID HookArray;
} KSE_HOOK_COLLECTION, *PKSE_HOOK_COLLECTION;
```

其中的HookArray也指向一个数组，每个元素的类型如下。

```
typedef struct _KSE_HOOK { 
_In_ ULONG Type; // 1表示Function, 2表示IRP Callback
union {
  _In_ PCHAR FunctionName; // 若Type == 1 
  _In_ ULONG CallbackId; // 若Type == 2 
}; 
_In_ PVOID HookFunction; 
_Outopt_ PVOID OriginalFunction; // 若Type == 1 
} KSE_HOOK, *PKSE_HOOK;
```

根据以上结构定义，可以在调试器里观察每个垫片的详细信息。以Win7VersionLieShim为例，先用dq nt!Win7VersionLieShim命令观察KSE_SHIM结构。

```
1: kd> dq nt!Win7VersionLieShim 
fffff800`abde20f8  00000000`00000038 fffff800`abde5a90
fffff800`abde2108  fffff800`abd560c0 fffff800`abe14e38
fffff800`abde2118  00000000`00000000 00000000`00000000
fffff800`abde2128  fffff800`abde2c48 00000000`00000038
```

得到HookCollectionsArray的地址fffff800 abde2c48。使用dq命令观察。

```
1: kd> dq fffff800`abde2c48
fffff800`abde2c48  00000000`00000000 00000000`00000000
fffff800`abde2c58  fffff800`abde28f0 00000000`00000004
```

得到HookArray的地址fffff800 abde28f0。继续观察。

```
1: kd> dq fffff800`abde28f0
fffff800`abde28f0  00000000`00000000 fffff800`abd560b0
fffff800`abde2900  fffff800`ac1680e0 00000000`00000000
fffff800`abde2910  00000000`00000000 fffff800`abd560e8
fffff800`abde2920  fffff800`ac168050 00000000`00000000
```

于是可以使用da命令来显示被挂钩函数的名字。

```
1: kd> da fffff800`abd560b0
fffff800`abd560b0  "RtlGetVersion"
```

再用ln命令观察，替代这个API的垫片函数如下。

```
1: kd> ln fffff800`ac1680e0 
(fffff800`ac1680e0)   nt!Win7RtlGetVersion
```

这意味着，对某个驱动程序启用这个垫片后，当它调用RtlGetVersion时，会被重定向到Win7RtlGetVersion。

### 6.5.4　注册垫片

NT内核中以全局变量的形式包含了一些垫片，某些驱动程序中也实现了垫片，表6-3列出了目前为止的大部分内核垫片。

表6-3　内核垫片

| 提　供　者 | 垫片对象/名称 | 描　　述 |
| --- | --- | --- |
| 内核内建的 | nt!KseSkipDriverUnloadShim | 调用驱动程序的Unload |
| 内核内建的 | nt!KseDsShim | 用于驱动程序回调函数的垫片 |
| 内核内建的 | nt!Win7VersionLieShim、nt!Win81VersionLieShim等 | 版本谎报 |
| Storport.sys | StorPort、DeviceIdShim、Srbshim | 磁盘存储有关的垫片 |
| Usbd.sys | Usbshim | USB设备驱动的垫片 |
| Ndis.sys | NdisGetVersion640Shim | 网络设备驱动的垫片 |


解释一下表6-3中的KseDsShim，其中的DS是driverscope的缩写，这种垫片的目标是拦截驱动程序的回调函数，比如各种IRP（I/O Request Packet）的处理函数。

KSE提供了一个名为KseRegisterShimEx的函数，用来注册垫片，其原型如下。

```
NTSTATUS KseRegisterShimEx( KSE_SHIM *pShim, PVOID ignored, ULONG flags,  
DRIVER_OBJECT *pDrv_Obj);
```

在KseInitialize函数中，它会反复调用这个注册函数来注册垫片。一个名为KseEngine的全局变量记录着已经注册的所有垫片。

值得注意的是，Windows系统中的某些驱动程序也会注册垫片，比如清单6-12中记录的便是磁盘端口驱动程序storport的StorpRegisterShim函数注册垫片的过程。

清单6-12　磁盘端口驱动程序的StorpRegisterShim函数注册垫片的过程

```
# Call Site
00 nt!KseRegisterShimEx
01 nt!KseRegisterShim
02 storport!StorpRegisterShim
03 storport!DllInitialize
04 nt!MmCallDllInitialize
05 nt!PipInitializeDriverDependentDLLs
06 nt!IopInitializeBootDrivers
07 nt!IoInitSystemPreDrivers
08 nt!IoInitSystem
09 nt!Phase1Initialization
0a nt!PspSystemThreadStartup
0b nt!KiStartSystemThread
```

调试时，可以这样观察注册垫片的名称。当设置在KseRegisterShimEx的断点命中时，放在rcx寄存器中的第一个参数便是KSE_SHIM指针，先由`dq @rcx`显示这个结构，然后dU命令显示结构中第三个字段的内容。例如：

```
0: kd> dq @rcx
fffff807`ca0474c0  00000000`00000038 fffff807`ca047a40
fffff807`ca0474d0  fffff807`ca0337a0 00000000`00000000
0: kd> dU fffff807`ca0337a0
fffff807`ca0337a0  "NdisGetVersion640Shim"
```

### 6.5.5　部署垫片

初始化KSE后，I/O管理器会先对启动类型的驱动程序部署垫片，也就是针对每个已经加载的驱动对象，依次调用KseDriverLoadImage。

可以把KseDriverLoadImage看作KSE公开给NT内核的模块加载接口，用来接收模块加载事件，为了行文方便，我们将其简称为KDLI。

KDLI内部会先调用KsepGetShimsForDriver，为当前的驱动程序寻找匹配的垫片，其执行过程如下。

```
00 nt!KsepGetShimsForDriver
01 nt!KseDriverLoadImage
02 nt!IopInitializeBuiltinDriver
03 nt!PnpInitializeBootStartDriver
```

在KsepGetShimsForDriver内部会调用nt!KsepResolveApplicableShimsForDriver来具体匹配适用的垫片。

找到匹配的垫片后，KDLI会调用KsepApplyShimsToDriver来应用垫片，对于钩子类型的垫片，会调用KsepPatchDriverImportsTable函数来修补IAT（清单6-13），与用户态的情况很类似。

清单6-13　修补IAT

```
00 nt!KsepPatchDriverImportsTable
01 nt!KsepApplyShimsToDriver
02 nt!KseDriverLoadImage
03 nt!IopInitializeBuiltinDriver
```

### 6.5.6　执行垫片

垫片的执行过程因垫片类型而不同，对于函数钩子类型的垫片，与用户空间的情况类似，在此从略。

下面介绍两种特别针对驱动程序行为的垫片执行过程，一种是上面提到的“跳过Unload垫片”，其执行过程如清单6-14所示。

清单6-14　执行过程

```
00 nt!KsepIsModuleShimmed
01 nt!KseDriverUnloadImage
02 nt!MiUnloadSystemImage
03 nt!MmUnloadSystemImage
04 nt!IopDeleteDriver
```

清单6-14中，下面的栈帧是典型的删除驱动对象和卸载驱动程序的过程，栈帧#01~#02表示内存管理器的MiUnloadSystemImage函数调用KSE的驱动程序卸载回调函数KseDriverUnloadImage，不妨将其简称为KDUI。

KDUI内部会调用KsepIsModuleShimmed来判定正在卸载的驱动程序是否启用了垫片，这个信息也记录在前面提到过的KseEngine全局变量中。如果正在卸载的驱动程序没有启用垫片，那么KDUI会立刻返回0。如果启用了“跳过Unload垫片”，那么它会执行垫片，包括更新统计信息和输出下面这样的日志。

```
fffff800`ac078b50  "KSE: Shimmed driver unload notification processed."
```

针对驱动程序的另一类常用垫片便是“杜撰”设备数据，“哄骗”老的驱动程序。举例来说，清单6-15展示了微软的USBXHCI（USB 3.0控制器驱动）调用KseQueryDeviceFlags接口获取模拟设备信息的过程。

清单6-15　获取模拟设备信息的过程

```
00 nt!KsepShimDbChanged
01 nt!KseQueryDeviceData
02 nt!KseQueryDeviceFlags
03 USBXHCI!Controller_PopulateDeviceFlagsFromKse
04 USBXHCI!Controller_PopulateDeviceFlags
05 USBXHCI!Controller_Create
06 USBXHCI!Controller_WdfEvtDeviceAdd
07 Wdf01000!FxDriverDeviceAdd::Invoke
08 Wdf01000!FxDriver::AddDevice
09 Wdf01000!FxDriver::AddDevice
0a nt!PpvUtilCallAddDevice
0b nt!PnpCallAddDevice
0c nt!PipCallDriverAddDevice
0d nt!PipProcessDevNodeTree
0e nt!PiProcessStartSystemDevices
0f nt!PnpDeviceActionWorker
10 nt!ExpWorkerThread
```

清单6-15所示的调用过程发生在PnP（即插即用）管理器的工作线程中，栈帧#0b~#0c表示发现了新设备，准备调用驱动程序的AddDevice（增加设备）回调。栈帧#07~#09是WDF框架的AddDevice方法，栈帧#03~#06转到USBXHCI驱动程序的函数，并调用KSE的接口函数KseQueryDeviceFlags，目的是让KSE得到执行机会，执行垫片逻辑。

最后说明一下，在安全模式启动时，或者启用了驱动的验证机制后，KSE会被禁止。

## 6.6　本章总结

本章深入挖掘了Windows操作系统中非常有特色的垫片机制，介绍了垫片机制使用的数据库文件、旨在解决应用程序兼容问题的应用程序垫片，以及旨在解决驱动程序兼容问题的内核垫片。

因为垫片机制可能被黑客所利用，破坏系统安全，所以微软一直没有公开垫片机制的技术细节。本章使用调试方式探微索隐，比较全面地介绍了垫片机制的配置数据、关键模块和工作原理。

某种程度上说，垫片机制是软件社会发展到一定阶段才有的一种技术。一方面，它有助于解决用户非常关心的软件兼容问题；另一方面，它也增加了软件行为的不确定性，有时可能给软件测试和调试带来意外的结果，比如，系统可能自动启用垫片机制，让某个Bug突然不见了。这意味着有了垫片机制后，定位软件问题的难度加大了，对软件工程师提出了更高的要求。

本章继续演示了“以调试之剑征服软件世界”的思想方法。如果大家阅读时遇到某些不认识的调试命令，不要紧，后面的章节会详细介绍。
