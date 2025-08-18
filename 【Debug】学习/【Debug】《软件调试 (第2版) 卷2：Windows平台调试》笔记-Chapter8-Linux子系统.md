# 第8章　Linux子系统

操作系统是软件世界中的管理机构，好比现实世界中的政府和国家机器，其地位和价值不言而喻。Linux操作系统自1991年问世后，很快得到了全美达（Transmeta）、英特尔等芯片公司的大力支持。对于芯片公司来说，很多新的功能如果没有操作系统的支持，上层软件就没有办法使用。另外，从政治因素、信息产业格局等角度思考，很多人、公司和国家也是不希望单一的操作系统产品垄断市场的。开源的Linux内核刚好满足了这些需求，于是在众多力量的推动下，迅猛发展。

在2001年前后，微软意识到了Linux操作系统对Windows操作系统的威胁，开始从多方面考察Linux系统，并且采取了一些措施，试图抑制Linux系统的发展。

2016年11月，微软宣布加入Linux基金会（Foundation），成为铂金级别的会员，这标志着微软彻底转变了对待Linux操作系统的态度，从最初的抑制和被动接受到拥抱与合作。

在2016年3月30至4月1日举行的Build大会上，微软宣布Windows操作系统将加入一个全新的子系统，在里面可以运行原生的Linux应用程序，不需要重新编译。这个子系统的名字叫“Windows的Linux环境子系统”（Windows Subsystem for Linux，WSL）。大约一周后，2016年4月7日，微软向Windows 10 Insider订阅者们推送的14328预览版本中包含了Bash on Ubuntu on Windows功能，WSL首次亮相。

今天，WSL已经成为Windows 10的一部分，并且开始与Windows平台的开发工具、调试工具深度融合。比如，VS中已经加入了WSL支持，可以在同一台机器上调试Linux程序，非常方便快捷。

WSL的出现标志着Windows和Linux两大流行的系统步入了互通融合的新阶段。

## 8.1　源于Drawbridge

从技术实现的角度来讲，WSL来源于微软研究院（MSR）的一个操作系统原型项目，名叫Drawbridge，时间在2011年左右。

简单来说，Drawbridge的目标是重构Windows 7操作系统的代码，让其可以运行在一个沙箱性质的进程空间里，像虚拟机那样。但是Drawbridge项目使用的不是虚拟机技术，而是经典的“库OS”（Library OS）思想。

所谓“库OS”就是把一个OS做成软件库的形式，它的所有特性都实现为可动态加载的库模块，整个OS可以以容器和沙箱的形式运行在另一个宿主OS上。原书P156的图8-1是Drawbridge的架构，是笔者根据Drawbridge项目论文中的架构图重新绘制的。

在图 8-1 中，下面是宿主 OS内核，是一个普通的Windows 7，左上部分是一个运行在库OS中的应用，中心虚线部分便是所谓的库OS，是重构过的Windows 7。右上部分是传统的Windows 7用户空间进程。

与虚拟机方式相比，库OS的优点是轻便灵活，需要使用时只要动态加载启动，不需要使用时，几乎没有任何开销。此外，因为没有虚拟机技术（VT）那样基于硬件的地址空间隔离，所以执行效率也更高。Drawbridge的英文本意是“吊桥”，需要时放下，不需要时拉起来，灵活自如，这个项目的名字取得非常妙。

实现方面，Drawbridge使用了第一篇介绍过的Pico进程技术，让“库OS”和应用程序以Pico进程的形式运行在NT内核之上，既有明确的进程身份，可以与系统中的其他软件和谐共处，又标志了它的特殊性，外表与大家一样，但是内部很特别。

## 8.2　融入NT

微软在 .NET 方面的一个惨痛教训就是在开发新技术的时候没有很好地利用Windows平台的原有设施，最典型的例子便是 .NET 的调试模型，使用了一套基于调试辅助线程的方法，既费力，又不稳定，到了 .NET 4.0 时进行重构，改为使用Windows系统原生的用户态调试设施，走了很长一段弯路。

从这个角度看，WSL的设计者们显然高明得多。对于Windows系统，Linux系统当然是新的来客，但是这并不意味着就要为此把所有东西都新建一套，那样不仅费时费力，还会导致两个部分格格不入，也失去了WSL的意义。因为没有WSL，通过虚拟机技术，用户本来就可以在同一台硬件上既运行Windows系统又运行Linux系统。或者说，WSL的价值就是要提供一种与旧有虚拟机和容器方案不同的方法。

2014年12月，著名IT媒体ZDNet的Windows方面资深记者玛丽·乔·弗莱（Mary Jo Foley）从微软得到神秘信息，撰文称微软在开发一个模拟器，可以在Windows系统上运行安卓应用程序。随后越来越多的消息传到外界，这个项目的名字叫Astoria。但在2016年2月，多家媒体报道说微软停止了Astoria项目。而不到两个月后，微软便宣布了WSL。

在笔者看来，从设计思想来看，WSL源于经典的库OS思想和我们上一节介绍的Drawbridge原型项目。从工程实现的角度看，WSL的实现和很多代码来自很多人误以为废弃的Astoria项目。最大的证据在WSL的内核空间核心模块lxcore中有100多个函数的名字都是以Adss开头的。名字的起始部分代表模块身份，这是NT内核的悠久传统。Adss这4个字符含义深刻，前两个字符Ad可能是Android的缩写，也可能是Astoria Driver之类，我们无法确定，但是都与Astoria有关。后两个字符ss，笔者推测它们是子系统（sub-system）的缩写，代表了Astoria和WSL的设计架构（8.3节）。

## 8.3　总体架构

概括地讲，WSL的架构是基于NT上经典的“环境子系统”技术设计的。在第一篇曾介绍过子系统的概念，其设计初衷就是让不同类型的应用程序都可以运行在NT内核之上。WSL利用NT的这个古老特征，让Linux应用程序运行在NT内核之上，让老技术焕发新光彩，为Windows平台添丁加户，可谓功莫大焉。

原书P157的图8-2是WSL的架构，图中下半部分是内核空间，其中的LxSS.sys和LxCore.sys是WSL的两个内核空间模块，以驱动程序的形式融入NT内核。上半部分为用户空间，其中的LxssManager为Linux子系统的服务进程，右侧的Init和Bash是WSL程序，以Pico进程的形式在运行。下面将分别介绍WSL的各个组成部分。

## 8.4　子系统内核模块

lxss是WSL的子系统内核模块，其角色与Windows子系统的内核模块win32k.sys相同。从历史角度来讲，它源于我们前面提到的Astoria项目中的adss.sys。

lxss是以驱动程序的身份存在的，其可执行文件位于system32\drivers目录下，并且在注册表中登记为内核空间的服务模块（Type为1代表SERVICE_KERNEL_DRIVER），位于`计算机\HKEY_LOCAL_MACHINE\SYSTEM\ControlSet001\Services\lxss`。

在对应的注册表中，有一个细节值得注意，那就是Start表键的值为0，代表SERVICE_BOOT_START。这意味着lxss是以启动类型的驱动随同NT内核一起加载进内存的。这意味着，在NT内核启动早期，lxss就得到了执行权，初始化好了。

lxss会创建一个名为\dev\lxss的设备对象，用于与用户空间的子系统服务进程进行通信，充当对外服务的接口。

## 8.5　微软版Linux内核

LxCore是WSL的另一个内核空间模块，其个头比lxss大很多，后者大约有40KB，前者接近1MB。对于NT内核来说，LxCore的身份是Pico进程的提供器。所谓Pico进程就是一种特殊的最小进程，特殊的地方就是它具有与其关联的内核空间提供器。如果把Pico进程比喻为孩子的话，那么Pico进程提供器就是家长。当Pico进程发起系统调用或者进程内发生异常时，NT内核就会找它的提供器，把这些难处理的问题都转给它的家长。

举例来说，如果在内核调试会话中为LxCore模块中的LxpSyscall_READ设置断点，然后启动一个WSL进程，那么这个断点很快就会命中。WSL进程调用系统服务的过程如清单8-1所示。

清单8-1　WSL进程调用系统服务的过程

```
# Call Site
00 LXCORE!LxpSyscall_READ
01 LXCORE!LxpSysDispatch
02 LXCORE!PicoSystemCallDispatch
03 nt!PsPicoSystemCallDispatch
04 nt!KiSystemServiceUser
05 0x0
06 0x0
```

栈帧#03中的PsPicoSystemCallDispatch是进程管理器的转发函数，它会把WSL进程的系统调用转给LxCore。

在LxCore中，像LxpSyscall_READ这样的函数有247个（16299版本），应该囊括了当前Linux系统的大多数系统服务。这意味着，LxCore承当着一个非常重要的功能，那就是为WSL的用户空间程序提供系统调用服务。从这个角度来说，LxCore是一个特殊的Linux内核，它不是Linux基金会的Linux内核，而是微软实现的“Linux内核”。

为了让应用程序觉察不到它们是运行在微软的Linux内核之上，LxCore要尽可能完全地模拟真实的Linux内核，外部行为要一致。

举个有趣的例子，在Linux的reboot API参数中，定义了4个魔码（magic code），代表着Linux内核的创始者Linus先生和他3个女儿的生日。在Linux内核的对应系统服务中，会检验这几个魔码。为了保持与Linux内核兼容，这几个魔码也出现在了微软的Linux内核中，比如下面是LxCore的REBOOT系统服务中检查第一个魔码的汇编指令。

```
LXCORE!LxpSyscall_REBOOT+0x4c:
fffff805`39c658ac 81fd69191228    cmp     ebp,28121969h
```

除了提供系统服务，LxCore还承担很多其他重要的角色，比如处理异常，名为 PicoDispatchException的函数负责从NT内核接管异常。比如，下面的栈回溯记录的便是NT内核把页错误异常转给LxCore的过程。

```
00 LXCORE!PicoDispatchException
01 nt!KiDispatchException
02 nt!KiExceptionDispatch
03 nt!KiPageFault
```

此外，LxCore中还要模拟Linux的伪文件系统，比如ProcFS、SysFS等。再举个有趣的例子，LxCore中定义了一个特殊的字符串，用来存放假Linux内核的版本信息。

```
3: kd> da LXCORE!ProcFsRootVersionBuffer
fffff805`39c21ca0  "Linux version 4.4.0-43-Microsoft"
fffff805`39c21cc0  " (Microsoft@Microsoft.com) (gcc "
fffff805`39c21ce0  "version 5.4.0 (GCC) ) #1-Microso"
fffff805`39c21d00  "ft Wed Dec 31 14:42:53 PST 2014."
fffff805`39c21d20  ""
```

当我们在内核调试会话中为这个变量设置硬件断点后，再在WSL中执行`cat /proc/version`，那么这个断点就会命中，其执行过程如清单8-2所示。

清单8-2　执行过程

```
00 LXCORE!memcpy
01 LXCORE!LxpUtilWriteToUser
02 LXCORE!LxpPseudoFsFileRead
03 LXCORE!VfsFileRead
04 LXCORE!LxpSyscall_READ
05 LXCORE!LxpSysDispatch
06 LXCORE!PicoSystemCallDispatch
07 nt!PsPicoSystemCallDispatch
08 nt!KiSystemServiceUser
```

栈帧#03中的VfsFileRead代表着LxCore对Linux内核的VFS（Virtual File Switch/System）设施的模拟。

## 8.6　Linux子系统服务器

我们知道CSRSS是Windows子系统的服务器进程。WSL的服务进程名叫LxssManager，它被编译为DLL，运行在SvcHost进程中。在任务管理器的服务列表中，可以找到它。

从实现技术角度讲，LxssManager使用了微软的组件对象模型（COM）技术，以进程外COM形式提供服务。LxssManager实现的最重要的接口名叫LxssUserSession，这个接口的方法主要有GetCurrentInstance()、StartDefaultInstance()、SetState()、QueryState()、InitializeFileSystem()等。使用微软经典的COM工具OleView可以观察到LxssUserSession组件。

很可能出于安全方面的顾虑，目前版本的WSL服务进程是不允许使用用户空间的调试器调试的。但是，使用内核调试器仍可以调试这个进程。例如清单8-3就是在内核调试会话中观察到的WSL服务进程在响应创建进程请求时的执行过程。

清单8-3　WSL服务进程在响应创建进程请求时的执行过程

```
00 LXCORE!AdssBusIoctl
01 LXCORE!LxpControlDeviceIoctlAdssBusInstance
02 LXCORE!LxpControlDeviceIoctlServerPort
03 nt!IofCallDriver
【省略数行】
08 ntdll!NtDeviceIoControlFile
09 lxssmanager!AdssBusClientpIoctl
0a lxssmanager!LxssInstance::_TranslateNtPath
0b lxssmanager!LxssInstance::_TranslateNtPathEnvironment
0c lxssmanager!LxssInstance::_AppendNtPath
0d lxssmanager!LxssInstance::CreateLxProcess
0e RPCRT4!Invoke
0f RPCRT4!Ndr64StubWorker
10 RPCRT4!NdrStubCall3
11 combase!CStdStubBuffer_Invoke
```

栈帧#0e~#11表示通过RPC接收到客户端（下一节将介绍的启动器程序）的请求，栈帧#09~#0d记录的是在用户空间处理这个请求，栈帧#00~#03记录的是请求内核空间模块LXCORE的文件系统服务，做路径转换。

## 8.7　WSL启动器

为了让Linux系统能更好地融入Windows系统，WSL公开了一套API，名字就叫WSL API。使用这套API，开发者可以配置和管理WSL实例，设置默认的WSL实例，以及启动Linux程序。这样的程序有个通用的名字，一般叫WSL启动器（launcher）。

值得说明的是，WSL启动器是纯粹的Windows程序，只不过它调用了WSL API来与WSL交互。

举例来说，在启动器程序中可以通过下面这个WslLaunch API来启动指定的Linux进程。

```
HRESULT WslLaunch(
  PCWSTR distributionName,
  PCWSTR command,
  BOOL   useCurrentWorkingDirectory,
  HANDLE stdIn,
  HANDLE stdOut,
  HANDLE stdErr,
  HANDLE *process
);
```

第一个参数用于指定系统中的WSL实例名字，第二个参数用于指定Linux程序的命令行，第三个参数用来指定是否使用当前目录作为工作目录，接下来的3个参数用来指定新进程使用的标准设备文件，最后一个参数用来接收创建好的进程句柄。

WSL自带了一个启动器，名字就叫wsl.exe，位于system32目录下。执行它，便会在默认的WSL实例中执行指定的命令，例如下面的命令会在默认的WSL实例中执行ls命令。

```
wsl ls /
```

WSL配备了一个名为wslconfig的小工具（位于system32目录下），使用它可以列出系统中安装的所有WSL实例，比如：

```
C:\wd10x64>wslconfig /l
kali-linux (默认)
Ubuntu
```

也可以使用/s选项来设置默认的WSL实例，比如，以下命令把默认实例改为Ubuntu。

```
wslconfig /s Ubuntu
```

通常每个WSL发行版都会配备一个启动器程序，比如从Windows商店下载的“WSL版Ubuntu”的启动器程序，名为ubuntu.exe。当我们在“开始”菜单旁边的搜索框中输入Ubuntu并选择执行时，系统启动的便是这个程序。

## 8.8　交叉开发

有了WSL，让开发和调试Linux程序多了一种非常便捷的方案，那就是使用“Visual Studio或者VS Code集成环境+WSL”来编辑、编译和调试Linux程序，只需要一个系统，无缝衔接。我们姑且把这种方式称为“V+W”交叉开发。

以下是使用Visual Studio 2019（简称VS2019）在Windows 10 17134版本上开发Linux程序的基本步骤，供大家参考，详细步骤可以查阅参考资料。

首先启用WSL和安装你喜欢的发行版，比如Ubuntu，然后使用如下命令安装常用的工具：

```
sudo apt install g++ gdb make rsync zip
```

其次要为VS2019安装“使用C++的Linux开发”组件。

接下来在VS2019中创建Linux程序，比如HiWSL。

接下来，在项目属性的“平台工具集”选项中选择WSL_1_0。

如果你的系统中有多个WSL实例，那么要指定你想使用实例的启动器程序。也就是在项目属性的“平台工具集”选项中的“`WSL*.exe`完整路径”选项（这个选项的名字有些别扭）。

以上工作做好后，就可以开始调试了，设置断点后单击工具栏上的GDB调试程序，VS2019便会通过WSL API启动Ubuntu中的gdb程序，开始远程调试。断点立刻命中，熟悉的VS窗口中呈现出Linux世界的内容。

简单解释一下原书P162的图8-9中的几个子窗口，右下角是VS的“输出”窗口，目前显示的是来自GDB的输出。GDB是Linux平台上最常用的调试工具，VS使用GDB的远程调试模式来调试HiWSL程序。

与“输出”窗口并列的“模块”窗口显示的是HiWSL进程中已经加载的两个动态库模块（so）。上面是Linux平台的模块加载器（loader），简称ld，它是创建进程时，由内核映射到新进程用户空间的，其角色与NTDLL中的加载器部分类似。另一个模块是著名的glibc，是由GNU组织开发的。在Linux平台上，它承担着多个关键角色，首先是GCC编译器的运行时模块，然后还是应用程序与内核的接口，大多时候，应用程序通过libc来调用系统服务。还有，大多数Linux程序使用的堆也是实现在libc中的。在本书的Linux分卷中，我们将详细地探讨GDB、ld和glibc。

在与图8-9相同的时间点打开任务管理器，可以看到运行在WSL中的GDB进程。任务管理器中还显示了正在被调试的HiWSL进程（可执行文件名为HiWSL.out），以及Linux系统的init进程。在这个截图中，传统的Windows进程与新的Linux进程并肩运行在同一个NT内核上，而且紧密协作完成复杂的交叉调试功能，让我们感受到了WSL的强大。WSL把经典的“库OS”思想与NT的环境子系统功能结合，让Linux应用程序融入NT大家庭中，是近年来Windows系统中难得一见的一个精妙设计。

## 8.9　WSL2

在2019年5月6～8日举行的Build 2019大会上，微软宣布了第二代的WSL，简称WSL2。在微软提供给Windows Insider订阅者的预览版本18917中，第一次包含了WSL2。相对于WSL2，第一代WSL便简称为WSL1。在笔者写作本节内容时，WSL2还没有正式发布，还处于测试阶段。本节基于微软公开的信息和预览版本做简单介绍，着重比较WSL2与WSL1的差异。

与WSL1相比，WSL2的最大差异是引入了原生的Linux内核来替代WSL1的LxCore功能。从根本上讲，WSL1使用的是NT内核，用NT内核的系统服务来模拟Linux内核系统服务，为Linux应用程序服务。而WSL2使用的是Linux内核，基于Linux基金会的Linux内核做修改，构建出一个特殊的版本为WSL2中的Linux程序提供服务。

如何在NT系统中运行Linux内核呢？我们知道，两个内核是无法并列运行在同一个系统中的。如果要并列，那么就要使用虚拟机技术。在微软的官方资料中，确认WSL2使用了虚拟机技术。在WSL2的预览版本中，可以看到使用的是微软的Hyper-V。这意味着，WSL2把定制过的Linux内核运行在一个Hyper-V虚拟机中。

表面上看，WSL2通过引入原生Linux内核解决了WSL1的一些问题，比如兼容性不够好，文件系统的性能不够好等。

但实质上，因为WSL2是基于虚拟机技术的，背离了WSL的设计初衷，脱离了根本，所以是个倒退。

在WSL1中，WSL中的Linux进程（简称WSL进程）与普通Windows进程是同一个内核上的进程，它们之间的边界是“进程边界”。在WSL2中，WSL进程运行在单独的虚拟机中，与Windows进程的边界是“机器边界”。进程边界和机器边界是有根本差异的。前者是机器内部的边界，后者是机器之间的边界。在WSL1中，WSL进程和Windows进程可以通过共享内存与内核对象等本机通信方式进行快速高效的协作。使用了虚拟机之后，便要跨越机器边界通信了。

更大的现实问题是，WSL2的虚拟机依赖，要求用户开启Hyper-V，这导致NT内核也要运行在虚拟机中。这样做会损伤NT系统的性能，并导致某些设备驱动程序发生故障，是很多人所厌恶的。

概而言之，WSL1的魅力是轻，其以非常低的开销把Linux应用融入NT系统中，Linux进程和Windows进程都直接运行在真实硬件之上。WSL2的特点是重，不仅把Linux进程运行在虚拟机中，还拖累本来的Windows进程也要运行在虚拟机中。从另一个角度来看，在Windows系统上以虚拟机的方式运行Linux系统，久已有之，而且方案众多，现在又多了一种方式，叫WSL2，不知其前途如何。

## 8.10　本章总结

NT内核的第一个版本NT 3.1发布于1993年，正值信息产业如朝阳般升起的20世纪90年代。20多年过去了，出于种种原因，NT内核在某些方面呈现“老态”，比如系统庞大、沉重，资源消耗多等。WSL1巧妙地让Linux应用可以直接运行在NT内核之上，发挥了NT内核的技术优势，让其焕发新的活力。WSL2使用笨重的虚拟机技术把Linux内核生硬地搬进Windows系统，导致磁盘、内存等方面的开销大大增加，让整个系统变得更复杂和臃肿，而且与虚拟机技术同质化，笔者觉得是误入歧途。

本章是本篇的最后一章，我们特意选择“Linux子系统”这个主题，它代表着当前阶段Windows和Linux两大平台并存的现状，也代表着两大平台相互借鉴、相互融合的大趋势。
