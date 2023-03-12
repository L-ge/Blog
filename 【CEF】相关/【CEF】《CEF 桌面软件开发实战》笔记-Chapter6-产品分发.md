###### 六、产品分发

1. 以 Release 模式编译工程
- Debug 模式编译出的二进制文件主要是为开发者调试程序提供帮助的，不适合分发给用户。分发给用户的文件应该是以 Release 模式编译得到的。
- 要想以 Release 模式编译我们的项目，则应该先以 Release 模式编译 CEF 工程得到 CEF 框架的 Release 模式的 dll 和二进制资源。简单起见，我们就直接以 Release 模式编译 CEF 工程下的 cefclient 示例项目即可。
- 编译完成后将在 E:\project\cef\build\tests\cefclient\Release 目录下得到 CEF 框架的 Release 模式的 dll 和二进制资源（你电脑上的 CEF 工程目录可能与我的目录不同），我们把这个目录下除 cefclient.exe、cefclient.exp 和 cefclient.lib 三个文件之外的所有文件都拷贝到我们的工程输出目录下（在我的电脑上这个目录为 E:\project\cef-in-action\x64\Release）。
- 准备好二进制资源和库之后，我们要配置工程的 Release 环境。之前介绍的是为 Debug（x64）环境做的配置，现在要为 Release（x64）环境做配置， 所以要做一些更改。
    - C/C++ -> 优化 -> 优化：最大优化(优选大小) (/O1)
    - C/C++ -> 优化 -> 内敛函数扩展：任何适用项 (/Ob2)
    - C/C++ -> 预处理器定义 -> 预处理器定义：
        ```ini
        WIN32
        _WINDOWS
        NDEBUG
        %(PreprocessorDefinitions)
        __STDC_CONSTANT_MACROS
        __STDC_FORMAT_MACROS
        _WIN32
        UNICODE
        _UNICODE
        WINVER=0x0601
        _WIN32_WINNT=0x601
        NOMINMAX
        WIN32_LEAN_AND_MEAN
        _HAS_EXCEPTIONS=0
        PSAPI_VERSION=1
        CEF_USE_SANDBOX
        CEF_USE_ATL
        _NDEBUG
        CMAKE_INTDIR="Release"
        ```
    - C/C++ -> 代码生成 -> 启用字符串池：是 (/GF)
    - C/C++ -> 代码生成 -> 运行库：多线程 (/MT)
    - C/C++ -> 代码生成 -> 启用函数级连接：是 (/Gy)
    - 连接器 -> 调试 -> 生成调试信息：否
    - 连接器 -> 系统 -> 启用大地址 ：是 (/LARGEADDRESSAWARE)
- 配置好 Release 编译环境后，我们就可以用 Release 模式编译 ceftest 工程了，编译成功后你会发现 E:\project\cef-in-action\x64\Release 输出目录下的 ceftest.exe 比 Debug 模式输出的 ceftest.exe 要小的多，运行速度也明显快很多。这是因为 Visual Studio 在 Release 模式下对工程做了大量的优化导致的。
- 得到 Release 编译的可执行程序和二进制资源后，我们把这些文件拷贝到一个新的目录下，准备使用这些文件制作应用程序的安装包。此时这个目录下包含了我们能运行我们应用程序的所有文件，你可以通过双击ceftest.exe启动程序验证功能是否完整。但这些文件并不全都是运行应用程序必须的，接下来我们就介绍一下这些文件各自的功用。

2. CEF 二进制文件的功用
- 开发者把应用程序分发给用户的时候，有些CEF框架的二进制资源和动态链接库是必须要分发给用户的，缺少这些库你的程序将无法正常运行，有些二进制资源和动态链接库是可以不用分发给用户的，也就是说，如果你没有使用CEF框架的某些功能，那么你就可以不向最终用户提供与这些功能对应的二进制资源和动态链接库。
- 接下来我们就逐一介绍 CEF 的动态链接库和二进制资源的功用。
    - 必须包括在安装包中的动态链接库和二进制资源有如下：
        - libcef.dll：CEF 核心库。
        - chrome_elf.dll：崩溃报告库。
        - icudtl.dat：Unicode 编码支持资源。
        - snapshot_blob.bin：V8 快照数据。
        - v8_context_snapshot.bin：V8 快照数据。
        - LICENSE.txt：这个文件是一定要分发给用户的，这是 CEF 版权要求的。你可以把 CEF 应用到你的闭源商业项目中，你也可以自由的修改 CEF 的源码，但你必须得携带 CEF 的 LICENSE 文件，要不然就是违法的。
    - 可选的动态链接库和二进制资源有如下： 
        - Resources/locales/ 这个目录包含 CEF、Chromium 和 Blink 使用的本地化语言资源。开发者可以通过 CefSettings.locale 设置需要加载哪个国家的语言资源，开发者设置的哪个国家，则相应的 .pak 文件则应该被分发给用户，其他国家的语言资源文件不应该分发给用户，如果开发者没有设置，则默认使用 en-US。
        - Resources/ 这个目录下除了 locales 子目录之外，还有 chrome_100_percent.pak、  chrome_200_percent.pak、resources.pak 等三个文件，这三个文件是 CEF、Chromium 和 Blink 使用的除语言资源外的其他资源。开发者都可以使用 CefSettings.pack_loading_disabled 直接禁用这些资源。如果不分发这些资源，可能导致某些 web 组件显示不正常。
        - 根目录下与 3D 渲染相关的动态链接库 d3dcompiler_47.dll、libEGL.dll 和 libGLESv2.dll，没有这三个库的话，HTML 内的 2D canvas、3D CSS 和 WebGL 的一些功能将不能正常使用。另外，swiftshader 子目录下也有 libEGL.dll、libGLESv2.dll，这两个文件是在 GPU 被禁用时或者没有 GPU 设备时，CEF 为开发者提供的软 3D 渲染的库，如果没有这两个文件，WebGL 相关的 API 将无法在那些设备上使用。
- 知道这些二进制资源和动态链接库的功用之后，我们就可以按照我们的需要来进行安装包的制作工作了。为了保证所有 CEF 的功能都可用，一般情况下开发者不会裁剪 CEF 的动态链接库，仅仅会裁减掉不必要的语言资源。

3. 使用 NSIS 制作安装包
- 我们计划采用 NSIS（https://nsis.sourceforge.io/） 来制作安装包。NSIS 是一个非常强大的安装包制作工具，它的全名是：Nullsoft Scriptable Install System，开发者可以在 NSIS 环境下通过编写特定的脚本来生成应用程序的安装包。
- electron-builder 默认情况下也使用 NSIS 为 Windows 应用程序制作安装包。
- 简单起见，我们直接使用 NIS Edit（https://hmne.sourceforge.net/） 这个可视化 NSIS 脚本制作工具来制作 NSIS 安装包，安装该工具后可以通过新建脚本向导来创建一个打包脚本。
    - 接着脚本向导会要求开发者输入目标程序安装包的基础信息。
    - 再选择安装包应用程序图标、压缩算法及语言。推荐使用 LZMA 压缩算法，该算法压缩比非常高，可以极大地减小你的安装包体积。
    - 接下来选择待打包的目标目录（这里忽略了几步无关紧要的配置）。在这个界面首先把示例中的两个文件删除，接着选择我们上文提到的 win-ia32-unpacked 目录，只选择一个目录路径即可，不必选择这个目录下的所有文件，再次忽略了几步无关紧要的配置。
    - 在这个界面中程序输入框是安装完成后要启动的可执行程序的文件路径，其中 $INSTDIR 代表着用户选择的安装目录，ceftest.exe 是你应用程序的可执行程序文件名。
    - 按要求一步步执行完脚本向导，最终生成一个 .nsi 格式的脚本。
    - 生产脚本后，界面有两个按钮，点击左侧的按钮后，NIS Edit 会把脚本传递给 NSIS，由 NSIS 编译脚本并生成安装包；右侧的按钮非但可以生成安装包，还可以启动安装包，以供开发者测试自己的安装包是否正常。
- 生成的脚本里包含了众多的预定义宏、区段、方法和逻辑，我们简单介绍几个：
    - Section "MainSection" SEC01：安装区段，在应用程序安装时执行，负责释放文件，写注册表等工作。
    - Section Uninstall ：卸载区段，在应用程序卸载时被执行，负责删除客户端电脑上的文件、删除注册表等工作。
    - Section -[SectionName]：隐藏区段（前面带-的都为隐藏区段），在应用程序安装时执行，负责创建桌面图标、注册卸载程序等工作。
- 有了 nsi 脚本，NSIS 就可以按照脚本的逻辑生成安装包，NIS Edit 工具是通过命令行参数的形式把这个脚本传递给 NSIS 的。
- 安装包生成完成后 NIS Edit 会提示生成路径，如 Output: "E:\project\cef-in-action\Setup.exe"。
- 此时你可以双击安装包的可执行程序，观察应用程序是否可以正常安装，安装完成后是否可以自动运行。
- 默认情况下应用程序将被安装到：C:\Program Files (x86)\My application目录下。
- 如果你还没有修改你的可执行程序的图标，那么它一定还是 Windows 应用程序的默认图标，你可以在 VisualStudio 使图标资源，来设置应用程序的默认图标，也可以使用 ResourceHacker（http://angusj.com/resourcehacker/） 这类工具来修改你的应用程序的图标。如果你需要为你的应用程序设置签名，那么你可以考虑使用 signtool（https://learn.microsoft.com/en-us/windows/win32/seccrypto/signtool）这类工具完成这项工作。
