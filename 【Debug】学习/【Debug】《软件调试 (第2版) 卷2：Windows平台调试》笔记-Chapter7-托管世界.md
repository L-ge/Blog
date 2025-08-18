# 第7章　托管世界

与传统的 C/C++ 语言相比，Java 和 .NET 等语言代表着软件的一个新方向。当把它们放在一起来讨论时，经常把 C/C++ 语言叫静态语言，而把 Java 和 .NET 语言称为动态语言。两类语言的核心差异是在哪个阶段把程序的内存布局等关键属性固定下来。对于 C/C++ 语言，程序编译链接好了，它的机器码、数据结构和内存布局就都确定下来了，因此称为静态语言。对于 Java 和 .NET 语言，开发阶段一般只把源代码编译为中间代码，到程序运行时再使用及时（JIT）编译技术把中间代码编译为目标代码。概而言之，静态语言在编译期就把机器码和内存布局确定下来了，而动态语言把很多事情留到运行期才确定。

二者相比较，静态语言具有针对特定硬件优化能力强、速度快的优点，缺点是开发速度慢、程序运行久时内存空间碎片化、难以调试等。动态语言则相反，具有容易跨平台，生产率高，程序中不直接使用内存地址，堆上的对象可以移动，内存空间不会碎片化，运行期仍有类型信息，容易调试等优点，缺点是执行效率不如静态语言，速度慢。

静态语言和动态语言各有优缺点，各有用武之地。很长一段时间里，它们一定会同时存在。为此，本书特开辟一章，对动态语言略作介绍。

## 7.1　简要历史

微软在2002年2月正式发布第一代动态语言开发技术，即 .NET 1.0，包括运行时框架以及配套的开发工具Visual Studio.NET。这意味着，一场既影响微软公司命运又影响整个Windows平台命运的.NET征程正式开始了。

.NET 1.0推出后的一段时间里，微软公司内还进行着一个更庞大的项目，那就是Windows Vista。Vista是下一代的操作系统，.NET 是未来的开发语言。如果二者能密切配合，相互促进，那么微软在软件行业的地位就可以继续在新世纪里所向无敌，笑傲江湖了。不知道微软的高层是否真的这样想，但是可以确定，微软为Vista和.NET规划了非常宏伟的蓝图。

欲速则不达，Vista的开发非常不顺利，一再延期，很多计划的功能不得不裁剪或者砍掉。新的 .NET 规划当然也受了影响。

无论如何，2006年11月，Windows Vista和.NET 3.0终于发布了。.NET 3.0引入新的图形框架WPF（Windows Presentation Foundation）、新的通信框架WCF（Windows Communication Foundation）和工作流框架（Workflow Foundation）。

.NET 3.0的新框架可以帮助软件开发者快速开发出漂亮的界面。但一个显著的问题就是性能较差。于是，后续版本开始从不同角度进行重构和优化。这个过程至今仍在继续。

2017年6月，微软正式发布了可以支持Linux系统和macOS的 .NET Core 1.0，代表着 .NET 技术开始走向更广阔的天地，进入一个新的阶段。今天，.NET Core 和普通的 .NET Framework 平行发展，同门两兄弟一起征战多样化的软件世界。

## 7.2　宏伟蓝图

使用 C# 或者 Visual Basic .NET 等编程语言编写的 .NET 源程序在编译时，默认编译为微软的中间语言（Microsoft Intermediate Language），简称MSIL，有时也称CIL，全称为公共中间语言（Common Intermediate Language）。.NET 的一个设计思想是在下面设计一套公共的基础设施，上面可以支持各式各样的编程语言，如下所示。

```
C#	Visual Basic .Net	C++
↓
CTS
↓
CIL
↓
CLR	及时编译器
↓
本地代码
↓
操作系统服务
```

下面的基础设施除了中间语言，还包括一套公共类型系统（Common Type System，CTS），以及一套公共的运行时，简称CLR。CIL、CTS和CLR统称为CLI（公共语言基础设施）。这些名字中都包含“公共”（Common）字样，这是为了强调它们是跨语言的，是抽象出来的公共设施，它们肩负着微软帝国的期望，要支持已有的和未来的各种顶层语言。

进一步讲，CIL是一种面向对象的、基于栈的字节码（byte code）。与CPU的指令集类似，CIL定义了200多条指令，分为10个组。以x86的如下加法指令为例：

```
add eax, edx//把EDX累加到EAX
```

对应的CIL指令为：

```
ldloc.0    // 把局部变量0压入栈
ldloc.1    // 把局部变量1压入栈
add        // 把栈顶的两项相加，结果压入栈
stloc.0    // 把栈顶那项弹出栈并保存到局部变量0中
```

微软的 .NET 框架中包含了两个CIL工具——ILASM和ILDASM，前者用于把CIL指令汇编为CIL字节码，后者反之。

在 .NET 世界中，还有一个在传统程序中没有的重要概念，那就是元数据（meta data）。在传统程序中，编译器在把源代码编译为目标代码的过程中，只把CPU执行时需要的信息（比如机器码和变量的位置）放到目标代码中。CPU不需要的那些源程序信息（比如结构名称、函数名称等）要么被丢弃，要么单独以调试信息的形式保存起来。而在 .NET 中，源程序中诸如类型定义、名称等信息是以元数据的形式保存下来的，在运行期仍可以使用。

.NET 给由CIL字节码和元数据组合起来的程序文件取了一个新的名字，叫程序集（assembly）。

理论上，.NET 程序集可以轻松跨平台，可以运行在任何硬件和操作系统之上，条件是只要在那上面有公共语言运行时，也就是CLR。从这个角度来思考，.NET 世界里便流行着另一个术语——托管（managed）。开发 .NET 程序的 C# 等语言叫托管语言，使用 .NET 技术开发出的程序叫托管程序，运行托管程序的CLR有时也叫托管运行时。编译好的C/C++程序，一切都很固定，可以直接给CPU去执行，好比一个成年人外出，不需要托管和监护。而 .NET 程序则不然，很多东西还要在运行期确定，不能直接交给CPU执行，还需要一个特殊的运行环境来照应，来看护，就好比把一个孩子送到托儿所一样。如此想来，托管一词真是贴切，遂以此命名本章。

.NET 程序运行的时候，系统会自动加载它依赖的MSCOREE.dll，这个DLL位于Windows系统目录下，是和 .NET 版本无关的。它会解析 .NET 程序中的信息，为其加载合适版本的CLR，其执行过程如清单7-1所示。

清单7-1　执行过程

```
09 KERNELBASE!LoadLibraryExW
0a mscoreei!RuntimeDesc::LoadLibrary
0b mscoreei!RuntimeDesc::LoadMainRuntimeModuleHelper
0c mscoreei!RuntimeDesc::LoadMainRuntimeModule
0d mscoreei!RuntimeDesc::EnsureLoaded
0e mscoreei!RuntimeDesc::GetProcAddressInternal
0f mscoreei!CLRRuntimeInfoImpl::GetProcAddress
10 mscoreei!GetCorExeMainEntrypoint
11 mscoreei!_CorExeMain
12 MSCOREE!ShellShim__CorExeMain
13 MSCOREE!_CorExeMain_Exported
14 KERNEL32!BaseThreadInitThunk
15 ntdll!__RtlUserThreadStart
16 ntdll!_RtlUserThreadStart
```

上面的清单中，MSCOREE中的EE是执行引擎（Execution Engine）的缩写。MSCOREE会通过注册表确定所需版本CLR的位置，然后加载它的mscoreei模块，后者名字中的i应该代表接口模块，它收到调用后，再加载自己的CLR模块。对于本章的示例项目HiDotnet，所用mscoreei的全路径如下。

```
C:\Windows\Microsoft.NET\Framework\v4.0.30319\mscoreei.dll
```

要加载的CLR主模块全路径如下。

```
C:\Windows\Microsoft.NET\Framework\v4.0.30319\clr.dll
```

值得提醒的是，大家不要被路径中的版本号所蒙蔽，很多时候，它并不代表真正的版本号，比如，观察文件属性，实际的版本信息如下。

```
ProductVersion:   4.8.3801.0
FileVersion:      4.8.3801.0 built by: NET48REL1LAST_B
PrivateBuild:     DDBLD435
```

也就是说，这里实际使用的CLR是最新的4.8版本，虽然路径是4.0。导致这个问题的原因是最近一些年来，.NET 运行时经常做所谓的原位更新（in place update），也就是直接在老的目录里更新文件，这样一来，文件的版本号和目录名中的版本信息就不匹配了。

如果使用2.0版本的 .NET，那么主运行时模块的名字叫mscorwks.dll或者mscorsvr.dll，后者是服务器版本，前者是桌面和工作站版本。

## 7.3　类和方法表

.NET 世界是完全面向对象的，所有东西都是以类或者对象的形式设计和存在的。换句话来说，在 C# 这样的托管语言里，不可以像 C/C++ 语言那样，孤零零地写个全局函数在那里。无论如何，必须要套个类的外壳才行。

以程序入口为例，C/C++ 中，只要有个约定好的入口函数就可以了。在 .NET 中，作为用户代码入口的Main方法，也必须写在一个类中。本章的示例程序CliHello是使用 C# 语言编写的控制台类型程序，它的Main方法是静态的，包含在一个简单的类中（清单7-2）。

清单7-2　CliHello的主要代码

```
namespace CliHello
{
    public class CliHello2
    {
        static void Main(string[] args)
        {
            Console.WriteLine("C#/CLI Hello, World!");
            Console.ReadLine();
        }
    }
}
```

值得说明的是，C# 编译器会自动寻找Main方法，所以包含Main方法的类名是可以任意设计的。如果找遍整个项目仍找不到合适的Main方法，那么编译器会报告如下错误。

```
1>CSC : error CS5001: 程序不包含适合于入口点的静态 "Main" 方法
```

顺便说一下，也可以在项目属性里来设置启动对象，也就是指定包含Main方法的命名空间和类名，如果不设置，编译器会自动寻找。

那么，为什么必须把Main方法写在某个类里呢？这里面有个深层的原因，那就是在托管世界里，类是组织代码和程序的一个基本单位。MSIL中的很多指令就是针对类和对象来定义的。或者说，在托管世界里，要执行一段代码前，必须先加载它所属的类。

接下来，我们以清单7-2中的CliHello为例，用调试的方法来介绍托管世界中组织和管理“类”的方法。

在WinDBG中打开CliHello.exe，待初始断点命中后，执行sxe ld:clr让WinDBG收到clr模块的加载事件时中断下来。收到模块加载事件后，再执行bp clr!RunMain命令埋伏断点。

断点命中后，执行.loadby sos clr来加载观察托管世界的WinDBG扩展命令模块sos（7.6节）。

接下来，先使用!name2ee命令找到CliHello2类的执行引擎（EE）信息，即：

```
0:000> !name2ee CliHello.exe CliHello.CliHello2
Module:      010c403c
Assembly:    CliHello.exe
Token:       02000002
MethodTable: 010c4d68
EEClass:     010c1278
Name:        CliHello.CliHello2
```

倒数第2行描述类的EEClass信息地址，倒数第3行是这个类的方法表的地址，有了这两个地址后，便可以分别用!dumpclass或者!dumpmt命令来观察进一步的信息了。清单7-3是使用!DumpMT命令观察到的记录。

清单7-3　记录

```
0:000> !DumpMT -MD 010c4d68
EEClass:         010c1278
Module:          010c403c
Name:            CliHello.CliHello2
mdToken:         02000002
File:            C:\sdbg2e\ch207\CliHello\CliHello\bin\Debug\CliHello.exe
BaseSize:        0xc
ComponentSize:   0x0
Slots in VTable: 6
Number of IFaces in IFaceMap: 0
--------------------------------------
MethodDesc Table
   Entry MethodDe    JIT Name
6d0e87b8 6ccec838 PreJIT System.Object.ToString()
6d0e86a0 6ce28568 PreJIT System.Object.Equals(System.Object)
6d0f1140 6ce28588 PreJIT System.Object.GetHashCode()
6d0a3f2c 6ce28590 PreJIT System.Object.Finalize()
02b40440 010c4d60   NONE CliHello.CliHello2..ctor()
02b40438 010c4d54   NONE CliHello.CliHello2.Main(System.String[])
```

清单7-3所示的方法表信息值得我们慢慢咀嚼，里面包含了托管世界的很多奥秘。我们重点看后面的方法表，也就是最后7行所描绘的表格。这个表格的第一行是列标题，下面6行中的每一行代表类的一个方法。

在我们的源代码中（清单7-2），只有一个方法Main，这里为什么是6个呢？另外5个方法的来源是这样的。一个是编译器自动产生的构造函数。另外4个是编译器自动赋予的基类System.Object的方法。在 .NET 世界中，有个强硬的约定，所有类都一定要从System.Object派生而来，如果程序员不写，那么编译器会自动补上。

表格一共有4列，最后一列是方法名。第二列是方法的描述，可以使用!dumpmd命令来显示，但主要信息已经显示出来了。第三列是编译状态，PreJIT代表方法已经预编译为机器码，None表示还没有编译，JIT表示已经及时编译过。第一列是方法的入口，如果这个方法已经编译为机器码，那么这个入口就是机器码的起始地址。如果这个方法还没有编译，那么它指向的是一段桩代码，以Main方法为例，这个方法还没有编译过，对入口做反汇编。

```
0:000> u 02b40438 
02b40438 e8b3ec666b      call    clr!PrecodeFixupThunk (6e1af0f0)
```

其中的PrecodeFixupThunk就是用来触发及时编译的桩代码。执行sxe ld:clrjit让WinDBG收到加载用于及时编译的clrjit模块时中断。然后执行g命令恢复执行，很快又中断下来，WinDBG显示加载模块。

```
ModLoad: 6cc50000 6ccd9000   C:\Windows\Microsoft.NET\Framework\v4.0.30319\clrjit.dll
```

观察栈回溯，可以看到因为要运行Main方法而触发加载及时编译模块的过程，如清单7-4所示。

清单7-4　加载及时编译模块的过程

```
09 KERNELBASE!LoadLibraryExW
0a mscoreei!RuntimeDesc::LoadLibrary
0b mscoreei!CLRRuntimeInfoImpl::LoadLibrary
0c clr!LoadAndInitializeJIT
0d clr!EEJitManager::LoadJIT
0e clr!UnsafeJitFunction
0f clr!MethodDesc::MakeJitWorker
10 clr!MethodDesc::DoPrestub
11 clr!PreStubWorker
12 clr!ThePreStub
13 clr!CallDescrWorkerInternal
14 clr!CallDescrWorkerWithHandler
15 clr!MethodDescCallSite::CallTargetWorker
16 clr!RunMain
17 clr!Assembly::ExecuteMainMethod
```

此时使用sos的bpmd命令对Main方法设置断点。

```
0:000> !bpmd -MD 010c4d54
MethodDesc = 010c4d54
Adding pending breakpoints...
```

因为方法尚未编译，所以显式增加了悬而未决（pending）的断点。

执行g命令恢复目标执行，很快看到WinDBG中出现如下信息。

```
(3bac.4060): CLR notification exception - code e0444143 (first chance)
JITTED CliHello!CliHello.CliHello2.Main(System.String[])
Setting breakpoint: bp 02B40864 [CliHello.CliHello2.Main(System.String[])]
Breakpoint 1 hit
```

上面的信息很值得玩味，第一行代表WinDBG收到了一个CLR通知异常，异常代码为e0444143，对应的ASCII码为.DAC，是Data Access Component（数据访问组件）的缩写。简单来说，DAC是 .NET 框架中的一个组件，供各种工具来访问 .NET 的内部世界。这里的CLR通知异常是及时编译器编译了Main方法后故意触发的，目的就是通知调试器，编译了一个方法，如果有关于这个方法的悬而未决断点，就赶紧落实。WinDBG心领神会，第三行信息告诉我们它在地址02B40864处设置了一个断点。

第四行信息显示断点命中。此时再重复清单7-3中的命令（!DumpMT -MD 010c4d68）观察CliHello2的方法表，可以看到Main方法已经编译好了。

```
02b40848 010c4d54    JIT CliHello.CliHello2.Main(System.String[])
```

入口地址也与之前不同了，不再指向触发PrecodeFixupThunk的代码，而是指向及时编译器新产生的代码，如清单7-5所示。

清单7-5　及时编译器为Main方法产生的机器码

```
02b40848 55              push    ebp
02b40849 8bec            mov     ebp,esp
02b4084b 83ec08          sub     esp,8
02b4084e 33c0            xor     eax,eax
02b40850 8945f8          mov     dword ptr [ebp-8],eax
02b40853 894dfc          mov     dword ptr [ebp-4],ecx
02b40856 833de8420c0100  cmp     dword ptr ds:[10C42E8h],0
02b4085d 7405            je      02b40864  Branch
02b4085f e83ceda06b      call    clr!JIT_DbgIsJustMyCode (6e54f5a0)
02b40864 90              nop
02b40865 8b0d3423d803    mov     ecx,dword ptr ds:[3D82334h]
02b4086b e86434656a      call    mscorlib_ni+0x4b3cd4 (6d193cd4)
02b40870 90              nop
02b40871 e81af3d26a      call    mscorlib_ni+0xb8fb90 (6d86fb90)
02b40876 8945f8          mov     dword ptr [ebp-8],eax
02b40879 90              nop
02b4087a 90              nop
02b4087b 8be5            mov     esp,ebp
02b4087d 5d              pop     ebp
02b4087e c3              ret
```

之所以列出Main方法的机器码，一是作为实例，二是为了解决某些细心读者的困惑。仔细观察，大家会发现，WinDBG设置断点的地址02B40864和方法表中的入口地址02b40848是不一样的，为什么呢？观察清单7-5中的汇编，容易看出，二者的差异就是前面的9条汇编指令，这9条指令会判断一个全局标志，如果标志不为0，则调用JIT_DbgIsJustMyCode来调用调试接口，不然就跳到02B40864。简单来说，前9条指令是为了支持所谓的“只调试我自己的代码”功能而附加的一个前置处理，WinDBG设置断点时跳过了这个部分。

断点命中时，如果直接用k命令观察栈回溯，那么可以看到如下调用过程。

```
00 0x2b40864
01 clr!CallDescrWorkerInternal
02 clr!CallDescrWorkerWithHandler
03 clr!MethodDescCallSite::CallTargetWorker
04 clr!RunMain
```

栈帧#01~#03是CLR通过所谓的“方法描述”来调用一个 .NET 方法的标准过程。

在sos中也提供了两个观察栈回溯的命令。一个叫!clrstack，它只显示 .NET 世界里的方法。

```
0:000> !clrstack
OS Thread Id: 0x4060 (0)
Child SP       IP Call Site
00aff1f0 02b40864 CliHello.CliHello2.Main(System.String[])-[E:...\Program.cs-@-10]
00aff370 6e1af016 [GCFrame: 00aff370]
```

另一个是!dumpstack，它既显示 .NET 方法，又显示非托管的函数。结果非常长，为节约篇幅，省去不录。

## 7.4　辅助调试线程

当一个普通的Windows本地程序开始运行时，操作系统会自动为其创建一个线程，通常称为初始线程（initial thread），应用程序的主函数（main或者WinMain）便是在这个线程中执行的。当应用程序需要启动更多线程时，它可以调用CreateThread或者CreateThreadEx这样的API。如果应用程序自己没有调用这些API来创建其他线程，也没有调用会创建线程的其他函数（如RPC），那么进程中便始终只有一个线程。这种说法对于托管程序来说也成立吗？答案是否定的。因为在托管程序初始化期间，.NET 运行时（runtime）会自动创建两个工作线程。这意味着，即使像上一节介绍的只打印一句话的Hello World小程序，在运行时也一定是多线程的。

那么这两个工作线程分别是做什么的呢？简单来说，一个是与托管程序的内存回收机制密切相关的终结器线程（finalizer thread），另一个便是用来支持调试的辅助调试线程（debug helper thread）。本节将深入介绍这个辅助调试线程，我们先从托管调试的基本模型说起，然后再谈调试辅助线程的作用和可能导致的问题。

### 7.4.1　托管调试模型

原书P146的图7-2显示了微软在 .NET 4.0 之前定义的用于调试托管程序的基本模型，这幅图根据MSDN中的架构图重绘。图中左侧是被调试进程（debuggee process），右侧是调试器进程（debugger process）。

我们先来看被调试进程，注意其中的3个矩形。最上面的代表非托管代码；中间的矩形代表托管代码；最下面的矩形代表支持托管调试的运行时控制器（runtime controller），我们前面提到的调试辅助线程便是这个控制器的工作线程，出于这个原因，调试辅助线程很多时候简称为RCThread。

下面我们再看右侧，上面的矩形代表非托管调试器，也就是调试本地代码的本地调试器；中间的矩形代表托管调试器；最下面是托管调试API和调试接口，其实就是一组COM接口，核心是ICorDebug，在MSDN中有详细介绍。

最后我们再来看图中的线条，这些线条代表了调试托管程序的3种模式。

第一种是使用非托管调试器来调试托管程序中的非托管代码，图中连接左右两个进程的上面一条横线代表的就是这种调试模式。因为托管代码最终也要编译为本地代码来执行，所以使用这种方式既可以调试托管进程中的本地模块，也可以调试及时编译后的托管代码。但是因为这种调试是把托管进程当作普通本地程序来调试的，从本地代码层次访问和控制目标进程，所以需要借助扩展插件才能观察托管世界中的名称和数据结构。

第二种是使用托管调试器通过托管调试接口调试托管代码，图中连接左右两个进程的下面一条横线加上托管调试器与托管调试API之间的竖线所代表的是这种调试模式。具体来说，托管调试器通过进程内的托管调试API与被调试进程中的运行时控制器通信来访问要调试的托管代码和数据。因为使用这种方式调试时，只能跟踪托管代码，不能跟踪本地代码，所以这种方式常称为纯托管调试。

第三种是使用一个调试器同时调试托管代码和非托管代码，这种情况相当于把上面两种调试模式加起来，调试器在使用本地调试机制的同时，还使用托管调试API，因此又叫混合调试。混合调试的优点是既可以调试托管代码，又可以调试本地代码。

举例来说，使用WinDBG调试托管程序属于第一种情况。使用Visual Studio（以下简称VS）集成开发环境（IDE）既可以做纯托管调试，又可以做混合调试。在原书P147的图7-3所示界面的“调试”选项卡中选中“启用本地代码调试（T）”选项后便启用了混合调试，否则使用纯托管调试。

综上所述，因为托管程序中既有托管代码，又有非托管代码，所以在调试时有3种典型的调试模式。第一种是“相对底层”的方法，灵活性大，但是不容易观察顶层的托管语义，适合调试复杂的问题。第二种是最常用的，适合做开发阶段的源代级调试。第三种存在一些局限，比如使用这种方式时，EnC（Edit and Continue）功能便不能使用了，而且在单步跟踪时，速度会比较慢（在老的Visual Studio .NET 中这一点尤其显著），因此通常只有在调试托管代码与非托管代码交互调用问题时才使用混合调试。

### 7.4.2　RCThread

下面我们把注意力集中到调试辅助线程，也就是RCThread。通过前面的介绍，我们知道，在第二种和第三种调试模式中，RCThread都起着重要作用。调试器是通过这个线程与被调试进程通信的，没有这个线程，调试器就无法工作。

不妨做个小实验。在VS中打开前面提到过的CliHello项目，选择菜单Debug → Start Debugging开始调试。在CliHello运行到Console.ReadLine()等待键盘输入时，启动WinDBG，将WinDBG附加到CliHello进程中，因为VS是以纯托管调试器的身份运行的，所以WinDBG还可以以本地调试器的方式附加上去。在WinDBG中使用`～* k`命令列出所有线程的栈回溯，找到清单7-6所示的包含DebuggerRCThread的线程便是RCThread。

清单7-6　RCThread的栈回溯

```
00 ntdll!NtWaitForMultipleObjects
01 KERNELBASE!WaitForMultipleObjectsEx
02 clr!DebuggerRCThread::MainLoop
03 clr!DebuggerRCThread::ThreadProc
04 clr!DebuggerRCThread::ThreadProcStatic
```

接下来执行`～1n（n之前是RCThread的线程号）`将RCThread挂起，然后执行g命令恢复CliHello进程运行。

通过上面的操作，我们相当于在VS不知晓的情况下，偷偷地将被调试进程中的调试辅助线程（RCThread）挂了起来。接下来，在VS中试着执行某种调试功能，比如选择Debug → Break All，这样操作后，VS的界面会立刻失去响应，像挂死了一样，过了差不多1分钟后，才显示出错误对话框。

此后，如果再执行其他调试命令，VS会提示不支持该操作，即使执行分离调试目标命令，也会失败。

通过上面的实验，我们知道，RCThread在托管调试中起着重要的纽带作用，如果RCThread被挂起或者意外退出，那么托管调试器将无法继续工作。

因为RCThread以及位于被调试进程内部的其他内部类和函数在托管调试中起着重要作用，所以它们经常被称为调试器的左端（Left Side，LS）。相对地，把位于调试器进程中的通信和接口函数称为调试器右端（Right Side，RS）。

调试器的左端与右端是使用进程间通信（Inter-Process Communication）机制来协作的。具体来说，主要使用了下面几个命名的事件对象和内存映射对象：

```
"CorDBIPCSetupSyncEvent_%d"
"CorDBIPCLSEventAvailName_%d"
"CorDBIPCLSEventReadName_%d"
"CorDBDebuggerAttachedEvent_%d"
"Cor_Private_IPCBlock_%d"
"Cor_Public_IPCBlock_%d"
"CLR_PRIVATE_RS_IPCBlock_%d"
"CLR_PUBLIC_IPCBlock_%d"
```

以上名字中的%d代表进程ID，在实际使用时会被替换为实际的进程ID值。以使用VS（进程号为4428）调试进程号为3340的CliHello为例，这些全局对象的实际名称如下。

```
CorDBIPCSetupSyncEvent_3340
Cor_Private_IPCBlock_3340
Cor_Public_IPCBlock_3340
CorDBIPCLSEventAvailName_3340
CorDBIPCLSEventReadName_3340
CorDBIPCLSetupSyncEvent_4428
Cor_Private_IPCBlock_4428
Cor_Public_IPCBlock_4428
```

对于希望更详细了解这些内核对象用法的读者，可以阅读SSCLI源代码。SSCLI的全称是Shared Source Common Language Infrastructure，是微软公开的CLI源程序，又称ROTOR，可以自由下载。

ROTOR源代码中包含了上文介绍的RCThread和IPC通信机制的源代码。以SSCLI 2.0为例，sscli20\clr\src\ipcman目录中包含了IPC使用的共享内存块有关的源文件。sscli20\clr\src\debug是调试支持的一个核心目录，又分为多个子目录，ee目录下的rcthread.cpp包含了实现辅助线程的主要代码，shell和di子目录中包含了调试接口和右端的代码，cordbg子目录中是一个命令行方式的简单调试器CorDbg。

### 7.4.3　刺探线程

在调试时，调试器左端和RCThread可能需要访问托管程序中的资源，访问之前可能需要先获取保护这些资源的保护锁，如果要获取的某个锁由托管程序中的普通线程所拥有，那么就可能导致死锁，因为普通线程需要由RCThread唤醒后才可能释放锁。为了防止这样的死锁发生，RCThread会创建另外一个线程，用来帮助RCThread刺探信息，以便RCThread可以知道哪些情况下需要获取锁，哪些情况下不要去冒险。这个用来刺探信息的线程名叫“刺探者”（canary）。清单7-7所示的栈回溯描述的便是刺探线程的执行经历。当时VS在调试这个进程，在将CliHello中断到调试器中一次后刺探线程便出现了。

清单7-7　刺探线程的栈回溯

```
# Call Site
00 ntdll!NtWaitForSingleObject
01 KERNELBASE!WaitForSingleObjectEx
02 clr!HelperCanary::ThreadProc
03 clr!HelperCanary::ThreadProc
04 KERNEL32!BaseThreadInitThunk
05 ntdll!RtlUserThreadStart
```

根据我们上面的介绍，调试辅助线程、刺探线程和调试器左端都是工作在被调试进程中的，它们与应用程序代码工作在一个进程空间中，使用的内存区域相互可见并可能相互影响。因此，托管调试模型具有较大的海森伯效应（详见本书卷1第15章），这是这种模型的不足之处。

## 7.5　CLR4的调试模型重构

我们在7.4节介绍的调试模型具有多方面的不足，因此，.NET 4.0引入了一种新的调试方式，一般就按版本号称其为v4.0调试方式，我们将其称为CLR4调试模型。

重构后使用的方法很简单，就是使用Windows系统中现成的用户态调试API。我们将在第三篇详细介绍Windows系统用于支持应用程序调试的调试设施。这套设施扎根内核，在系统中享有诸多特殊照顾，它依赖内核中的用户态调试子系统（DbgK）来协调调试事件，而不是普通的进程间通信，调试器使用调试端口（DebugPort）与被调试进程建立调试会话，这一关系登记在进程的核心数据结构中，受到内核的照顾。

原书P150的图7-8画出了重构后的CLR4调试模型。图中左侧是调试器进程，以VS的集成调试器（VSIDE）为例，右侧是使用 .NET 4.0（CLR4）的被调试进程。二者使用Windows系统的本地用户态调试设施建立起正式的调试关系。正如这次重构的主要贡献者Rick Byers在他的名为“ICorDebug re-architecture in CLR 4.0”的博客文章中所说的：“Under the hood we’re built on the native debugging pipeline”（揭开表面，新的调试模型是建立在本地调试流水线之上的）。

重构之后，当调试器要附加到要调试的托管进程时，调试器会调用操作系统的本地用户态调试API——DebugActiveProcess，建立真正的调试会话。

值得说明的是，当观察 .NET 4.0 的托管进程时，我们仍然可以看到调试辅助线程。保留调试辅助线程主要是为了兼容老版本的调试器，以便可以用CLR4以前的调试器来调试CLR4程序，这种调试模式简称为v2模式。对于支持CLR 4的VS 2010这样的调试器来说，它支持多种调试模式，新的模式称为v4模式，在开始调试时可以选择。

概而言之，CLR4重构后的调试模型就使用了Windows系统早已有之的调试API，可谓绕了一圈弯路后又回归到了本地调试使用多年的成熟方法。

## 7.6　SOS扩展

前面两节分别介绍了用于托管程序的两种调试模型。本节将介绍使用WinDBG调试器和SOS扩展模块来调试托管程序的方法。使用这种方法可以跟踪包括托管运行时自身代码在内的本地代码，也可以调试托管代码，又可以看到托管世界的执行引擎（EE）数据，特别适合调试复杂的托管程序问题。

与NT内核和KD的关系类似，SOS和 .NET 的关系也可谓是一衣带水，休戚相关。在开发 .NET 时，人们就开发了一个NTSD的扩展模块用来调试正在开发的代码，这个扩展模块有个响亮的名字，叫Strike。

Strike很有用，不但对开发 .NET 的内部工程师有用，而且对于做产品支持和服务（Product Support Service，PSS）的人也有用。但是开发团队不想把自己使用的Strike直接给别人用，觉得那没有必要，Strike显示的高级信息可能不被人理解。于是，便安排人开发了一个简化的Strike，取名为Son Of Strike（Strike之子），简称SOS。而SOS刚好又代表国际通用的船舶呼救信号（Save Our Ship），用在这里可以表示这个模块可以在危急时刻派上用场。

.NET已经走过了很多年，从1.0到1.1、2.0、3.0、3.5、4.0、4.5、4.8等。当然，在每个版本中都有SOS，或者说，有了SOS的帮助，才有一个又一个的新版本。在这个过程中，SOS也在发展和成熟。

### 7.6.1　加载SOS

要使用SOS调试托管程序，当然要有WinDBG。下载和安装WinDBG不在话下。当使用WinDBG来调试托管程序时，WinDBG是将托管程序当作一个“略微有些特别”的本地程序来对待的。之所以这样说，是因为大多时候WinDBG将被调试的托管进程与普通的Windows进程按一样的方式处理，只是在个别时候会使用调试引擎中的CLR特殊支持（稍后介绍）。

与调试本地程序一样，可以使用两种方式来建立调试会话，一种是在WinDBG中启动托管程序的EXE文件，另一种是附加到一个托管进程中。我们先来谈后一种情况。

首先，不同版本的运行时有不同版本的SOS，应该根据被调试程序所使用的 .NET 运行时来加载对应版本的SOS。

如果被调试程序使用的是 .NET 1.0 运行时，那么它的SOS模块位于WinDBG程序目录中的clr10子目录下。具体来说，可以使用如下命令。

```
.load clr10\sos.dll
```

从 .NET Framework 1.1 开始，.NET 运行时中就包含了与其配套的SOS.dll，比如下面便是两个与运行时放在一起的sos.dll。

```
C:\WINDOWS\Microsoft.NET\Framework\v1.1.4322\sos.dll
C:\WINDOWS\Microsoft.NET\Framework\v2.0.50727\sos.dll
```

如果使用.load命令加载上面的sos.dll，路径很长，不容易记住。这时可以通过观察运行时核心模块mscorwks.dll的位置来得到CLR的位置，然后复制并粘贴。

```
0:000> lmvm mscorwks
start    end        module name
791b0000 79419000   mscorwks   (pdb symbols)          C:\WINDOWS\Microsoft.NET\Framework\v1.1.4322\mscorwks.dll
```

但是这样做有点麻烦，更简洁的方法是使用.loadby命令，即：

```
.loadby sos mscorwks
```

意思是加载与mscorwks模块相同位置的sos扩展模块，如果执行时没有任何提示信息，那么便执行成功了。

无论使用以上哪种方法加载，加载后，都可以使用.chain命令来观察已经加载的扩展模块。

```
0:000> .chain
Extension DLL chain:
C:\WINDOWS\Microsoft.NET\Framework\v1.1.4322\sos: API 1.0.0, built Thu Jul 15  
10:46:07 2004 [path: C:\WINDOWS\Microsoft.NET\Framework\v1.1.4322\sos.dll]
[以上信息做过删减]
```

加载成功后，可以执行!help命令来显示SOS的帮助信息。对于2.0或者更高版本的SOS，可以在!help后面加上要了解的命令来得到关于某个命令的详细解释。

下面再讲另一种情况，也就是在使用WinDBG打开EXE时该如何加载SOS扩展。因为当被调试进程加载了CLR后，SOS才能工作和有意义，所以应该在CLR的核心模块MSCORWKS.dll或者clr.dll加载后，再加载SOS。那么如何知道MSCORWKS/clr何时被加载呢？这可以通过定制WinDBG模块加载事件的方式来实现，执行sxe命令。

```
sxe ld:mscorwks.dll 或者sxe ld:clr.dll
```

上述命令的含义是让WinDBG收到被调试进程加载mscorwks.dll/clr.dll模块的事件时，中断下来。中断下来后，便可以执行.loadby sos mscorwks或者.loadby sos clr命令了。

也可以使用下面这条命令把上面所说的两个动作合在一起。

```
sxe -c ".loadby sos mscorwks;g" ld mscorwks.dll
```

其含义是当收到加载mscorwks.dll的事件后执行-c后面跟的命令，也就是双引号内的内容——先加载sos，然后恢复目标继续运行（g）。

### 7.6.2　设置断点

可以使用SOS中的BPMD命令来针对托管代码设置断点，它有两种格式。先来看第一种。

```
!bpmd <module name> <method name>
```

第一个参数是模块名，第二个参数是完整描述的方法名。例如使用下面的命令可以为CliHello模块中CliHello名称空间的CliHello类的Main方法设置断点。

```
!bpmd CliHello CliHello.CliHello.Main
```

其中，模块名可以不区分大小写，但是命名空间、类名和方法名必须严格区分大小写。图7-12所示的截图便是设置以上断点后这个断点命中时的场景。

注意，在原书P153的图7-12中，!bpmd执行时，SOS显示找到了一个匹配的方法，并增加了一个“延迟的”断点，这是因为当时被设置断点的方法还没有编译。在g命令恢复目标继续执行后，我们看到WinDBG收到了一个CLR通知异常，告诉WinDBG，CliHello.CliHello.Main方法已经被及时编译了，于是WinDBG向及时编译后的本地代码位置（0x00dc0070）设置了断点，也就是向那里写入断点指令，x86平台上即INT 3，机器码为0xCC（本书卷1有详细介绍）。使用bl命令，可以看到WinDBG设置的断点。

```
!bpmd命令的另一种格式是：
!bpmd -md <MethodDesc>
```

这种形式需要知道方法的MethodDesc结构的地址，这个地址可以通过!name2ee命令来得到。比如：

```
0:000> !name2ee clihello CliHello.CliHello.Main
...
MethodDesc: 00a73020
Name: CliHello.CliHello.Main(System.String[])
JITTED Code Address: 00dc0070
0:000> !bpmd -md 00a73020
MethodDesc = 00a73020
```

SOS目前不支持使用源文件位置设置断点，但是可以使用一个名为SOSEX的扩展模块来设置这样的断点，比如：

```
0:000> !mbp Program.cs 11
Breakpoint set at CliHello.CliHello.Main(System.String[])
```

可以从网上免费下载这个扩展模块，下载后复制到WinDBG的winext目录后，可以使用.load命令来加载。

### 7.6.3　简要原理

SOS主要使用操作系统提供的机制来访问被调试进程，比如使用ReadProcessMemory和WriteProcessMemory这两个API来读写被调试进程的内存。但是只用这两个API是不够的，还要知道要读写的位置，也就是被调试进程中的运行时数据结构。这是如何做的呢？简单来说是从目标进程中读取一个运行时信息表，这个信息表中有CLR运行时的关键数据结构（比如类信息、托管堆等）的位置。再启动一个WinDBG，附加到刚刚加载了SOS的WinDBG上，针对ReadTableInfo方法设置一个断点后，恢复前一个WinDBG运行并执行SOS扩展命令，断点命中后执行kn命令，便可以看到SOS模块初始化信息表的过程。

SOS把读到的信息保存在自己的全局变量中，执行`x sos!g_*`便可以看到这些全局变量。

```
0:008> x sos!g_*
602bf4a0 sos!g_cClassInfo = <no type information>
602bf4ad sos!g_fInfoTableReadAlreadyAttempted = <no type information>
602bf494 sos!g_rgGlobalInfo = <no type information>
602bf4a4 sos!g_rgMemberInfo = <no type information>
602bf49c sos!g_rgClassInfo = <no type information>
602bf490 sos!g_pTableInfo = <no type information>
602bf498 sos!g_cGlobalInfo = <no type information>
602bf4ac sos!g_fInfoTableValid = <no type information>
...
```

上面介绍的是SOS 1.1版本的基本工作原理。对于2.0或者更高版本，SOS会使用我们前面提到的DAC（mscordacwks.dll）模块来访问被调试进程，这个模块在执行某些操作时可能与辅助调试线程通信，因为篇幅所限，细节从略。如果读者感兴趣，可以继续用上面的调试方法来跟踪分析，或者阅读ROTOR中debug\daccess目录下的源代码。

## 7.7　本章总结

坦率地讲，微软对 .NET 寄予了很高的期望，但是实际取得的成果远远不如预期，经过十几年的努力，始终没有出现微软所期望的“.NET技术流行天下”的局面。直至今天，在意高生产率的开发人员热衷于Java或者Python，在意高效率的开发人员仍使用`C/C++`。使用 .NET 的团队当然也有一些，但是谈不上流行。但无论如何，经过了十几年，.NET 确确实实已经成为Windows平台的一部分，不少软件已经 .NET 化，比如VS，比如微软的Office。另外，代表移动互联网特色的很多Windows Store程序也是基于 .NET 的。概而言之，在今天的Windows系统中，一般总是有一些 .NET 程序在运行着。某些程序在这一刻还是普通的Windows程序，但是过一会儿，它可能因为加载了 .NET 模块，于是就有 .NET 运行时入驻，也时不时地执行托管代码了。

概而言之，不管我们喜欢还是不喜欢，.NET 已经成为Windows平台上的一种客观存在。调试时，我们至少要认识它。
