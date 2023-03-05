###### 一、入门知识

1. 基于 CEF 开发桌面应用有其独特的配置要求，比如运行库必须配置为 MTD/MT ，而不能是 MD/MDd 等。

2. CEF 是 Chromium Embedded Framework 的简写，顾名思义，这是一个把 Chromium 嵌入其他应用的框架。官网地址是：https://bitbucket.org/chromiumembedded/cef，这个开源项目是 Marshall Greenblatt 在 2008 年创立的，由 C/C++ 编写而成，它通过提供稳定的 API 来避免开发者被 Blink、V8、Chromium 等复杂的代码逻辑所困扰。CEF 非常注重开发者的使用体验，很多功能都有默认实现方式，遵从约定优于配置的原则，开发者可以很轻松地驾驭 CEF 框架。

3. 选择 CEF SDK
- 下载 CEF 框架：https://cef-builds.spotifycdn.com/index.html#windows64
- 由于 Chromium 和 CEF 都更新得非常频繁，所以即使是当前稳定版本，也难免会有一些问题，如果遇到由于版本太新而导致的问题，可以点击下载页面左下角 Show All Builds 来选择历史版本。
- Debug Symbols 和 Release Symbols 是为调试崩溃报告服务的符号文件，我们暂时用不到这些文件，不用下载。
- Standard Distribution 相对于 Minimal Distribution 来说，多包含了调试文件和示例代码文件，我们这里选择 Standard Distribution。

4. 编译运行 CEF 示例项目
- 按照官方的指导，使用 CMake 工具来编译 CEF。
- CMake 工具内置了 cmake-gui 工具，我们使用这个工具来构建 CEF 的 VisualStudio 工程，在 Where is the source code 和 Where to build the binaries 设置好相应的目录，点击 Configure 按钮，配置工程，再点击 Generate 生成工程文件。
- Generate 完成后会在你配置的 build 目录下（Where to build the binaries）生成 Visual Studio 的工程文件：cef.sln。打开这个文件就启动 Visual Studio 并打开了我们刚刚配置的这个工程，或者点击 cmake-gui 工具中的 Open Project 按钮，也会打开这个工程。
- 在 Visual Studio 的解决方案资源管理器中，把 cefsimple 项目设为启动项目，并启动这个项目的新实例。
- 除此之外，工程内还有另外一个 Demo 项目：cefclient ，你也可以按照上面描述的步骤启动一下看看效果，这个项目就复杂得多了，如果你刚刚入门 CEF，建议你暂时不要读 cefclient 的源码。

5. 搭建全新的 CEF 工程
- 使用 Visual Studio 创建 C++ 项目还是比较简单的，这里简要介绍一下：首先使用 Visual Studio，创建一个全新的 C++ 空项目，我给它取名 ceftest，你可以按照自己的意愿取名；接下来，我们在工程中添加一个文件 main.cpp 的 C++ 程序文件，暂时先不必为 main.cpp 撰写代码，我们先把工程配置好（这个文件对接下来的工程配置是有帮助的，如果工程中没有一个 cpp 文件的话，VisualStudio 是不会出现 C/C++ 配置属性的）。
- 配置工程
    - 先打开项目的属性页面，这里属性窗口有一点需要注意，我们在配置选择框和平台选择框中选择 Debug 和 Release 或者 x64 和 Win32 ，都意味着不同的配置项。也就是说，我们为 Debug 环境做的配置是与 Release 环境的配置不同的，现在我们以 Debug+x64 环境为例介绍配置内容，配置以下几项内容即可。
        - “常规->C++ 语言标准”设置为：ISO C\+\+17 标准 ( /std:c\+\+ 17 )，这个配置可以使我们在工程中使用现代 C\+\+ 的语言特性。
        - “C/C++ ->常规->附加包含目录”设置为： E:\project\cef; ，这个路径是你下载的 CEF 框架所在的路径。注意：在使用 CEF 框架前你必须已经成功编译了 CEF 的示例项目。
        - “C/C++ ->常规->预处理器定义”设置为如下内容：
            ```ini
            WIN32
            _WINDOWS
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
            _HAS_ITERATOR_DEBUGGING=0
            _SILENCE_ALL_CXX17_DEPRECATION_WARNINGS
            ```
            - 这些预处理器定义大部分都是 Windows 平台和 CEF 框架要求的预处理器定义，最后一项是为了屏蔽 CEF 框架中低版本 C++ 警告的预处理器。
        - “C/C++ ->代码生成->运行库”设置为：多线程调试 (/ MTd )，这是为了适配 Chromium 的调试方式而设置的。
        - “链接器->输入->附加依赖项”设置为如下内容：
            ```vbnet
            E:\project\cef\build\libcef_dll_wrapper\Debug\libcef_dll_wrapper.lib
            E:\project\cef\Debug\libcef.lib
            E:\project\cef\Debug\cef_sandbox.lib
            comctl32.lib
            gdi32.lib
            rpcrt4.lib
            shlwapi.lib
            ws2_32.lib
            Advapi32.lib
            dbghelp.lib
            Delayimp.lib
            OleAut32.lib
            PowrProf.lib
            Propsys.lib
            psapi.lib
            ```
            - 这是在编译、链接我们的程序时，使用 CEF 和 Windows 库的配置，其中 E:\project\cef 是你下载的 CEF 框架所在的路径。
            - “链接器->系统->子系统”设置为：窗口 ( /SUBSYSTEM:WINDOWS )，VisualStudio 创建空白项目时，默认是控制台项目，这里我们把它调整为窗口项目。
            - “清单工具->输入和输出->附加清单文件”设置为：$(ProjectDir)ceftest.manifest，这是为了让我们的工程生成的可执行程序兼容不同版本的 Windows 操作系统。设置好此配置之后，需要在工程的根目录下（与 main.cpp 在同一个目录）创建一个名为 ceftest.manifest 的文件，代码如下（此处的文件名你也可以按照自己的想法命名，不过注意配置里的文件名要和实际的文件名一致）：
                ```xml
                <?xml version="1.0" encoding="utf-8"?>
                <assembly xmlns="urn:schemas-microsoft-com:asm.v1" manifestVersion="1.0">
                        <compatibility xmlns="urn:schemas-microsoft-com:compatibility.v1">
                                <application>
                                        <!--The ID below indicates application support for Windows 8.1 -->
                                        <supportedOS Id="{1f676c76-80e1-4239-95bb-83d0f6d0da78}"/>
                                        <!-- 10.0 -->
                                        <supportedOS Id="{8e0f7a12-bfb3-4fe8-b9a5-48fd50a15a9a}"/>
                                </application>
                        </compatibility>
                </assembly>
                ```
- 准备资源
    - 配置好这个全新的工程后，我们还需要把 CEF 的二进制资源拷贝到工程的输出目录下，只有这样这个新工程才能正确调用 CEF 框架提供的API。
    - 在上面“编译运行 CEF 示例项目”小节，已经成功运行了 CEF 框架自带的示例项目，这个示例项目的输出目录下，这里有我们需要的 CEF 二进制资源。
    - 你需要把上述二进制资源文件，拷贝到 “E:\project\ceftest\x64\Debug” 目录下， ceftest 就是我们这个新工程所在的目录，需要注意的是，这个路径目前可能还不存在，你可以自行创建一下这个目录。
