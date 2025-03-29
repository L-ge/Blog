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

---

###### 二、起步阶段

1. 入口程序
- 在上一讲中我们已经创建了一个名为 main.cpp 的程序文件，这是我们整个程序的入口文件，接下来我们先把这个文件的代码写好：
    ```cpp
    #include <windows.h>
    #include "App.h"
    //整个应用的入口函数
    int APIENTRY wWinMain(_In_ HINSTANCE hInstance, _In_opt_ HINSTANCE hPrevInstance, _In_ LPTSTR lpCmdLine, _In_ int nCmdShow) 
    {
        CefEnableHighDPISupport();
        CefMainArgs main_args(hInstance);
        CefSettings settings;
        int exit_code = CefExecuteProcess(main_args, nullptr, nullptr);
        if (exit_code >= 0) {
            return exit_code;
        }
        CefRefPtr<App> app(new App());
        CefInitialize(main_args, settings, app.get(), nullptr);
        CefRunMessageLoop();
        CefShutdown();
        return 0;
    }
    ```
    - 这是整个应用程序的入口，应用启动后执行的第一段逻辑，我们在这段逻辑中完成了一些初始化工作、为不同的进程分派了不同的处理逻辑、开启了 CEF 的消息循环、在应用退出后释放 CEF 占用的资源等。
- 操作系统入口函数
    - wWinMain是操作系统指定的应用程序的入口函数。这个函数有 4 个参数，参数前有 In 修饰符表示该参数是必填的输入参数，有 In_opt 修饰符的意思是该参数是可选的输入参数。
    - 这 4 个参数都是由操作系统传递给 wWinMain 方法的：
        - hInstance 是应用程序的实例句柄，也叫模块句柄；
        - hPrevInstance 没有实际意义，是老版本 Windows 系统的历史遗留产物；
        - lpCmdLine 是命令行参数；
        - nCmdShow 表示应用程序的窗口是最小化、最大化还是正常显示。
    - 这个方法返回一个 int 类型的数字，操作系统会直接抛弃这个值。但如果有外部应用唤起你这个应用，那么这个返回值对于它来说可能是有意义的，一般返回 0 表示应用程序正常退出，返回其他值表示应用程序因异常而退出。
- 应用程序初始化
    - CefEnableHighDPISupport 方法启用高分屏支持，如果不调用这个方法，你的应用程序在一些高分辨率的屏幕下将显示得很模糊。
    - CefMainArgs 是 CEF 对应用程序实例句柄的包装类，用于多进程启动。这里我们使用 hInstance 实例化了这个类的对象，名为 main_args 。
    - CefSettings 是 CEF 的配置对象，类似日志级别、调试端口等都是通过这个对象设置的，这里我们就全部使用默认值。
- CEF 框架如何启动多个进程
    - CefExecuteProcess 负责启动进程。这里需要详细介绍一下，一般我们启动一个 CEF 应用，你会发现任务管理器里有好几个进程。这些进程中除了主进程是由用户启动的外，其他子进程都是 CEF 框架通过 CefExecuteProcess 方法启动的。
    - 主进程启动后，执行到此方法时，此方法会立即返回 -1 。接下去主进程就会进入 CEF 的消息循环，在适当的时候主进程会以特殊的命令行参数，多次启动你的可执行文件，这样就创建了多个子进程。子进程启动后也会执行到这个 CefExecuteProcess 方法，但子进程执行此方法会被阻塞（不会继续执行后面的逻辑），当子进程执行完它们的任务后，这个方法将返回一个大于等于 0 的值。也就是说子进程在第10行代码处就退出执行了，子进程不会执行 12~18 行代码。
    - CefExecuteProcess 方法的第一个参数就是我们前面介绍的 CefMainArgs 对象，第二个参数可以是不同进程的业务处理对象，也可以为空。基于一切从简的原则，我们这里传递了一个空指针。第三个参数与 Chromium 沙箱有关，此处我们也没有设置。
- CEF 特有的智能指针
    - CefRefPtr 是一个智能指针类型，它内部有一个引用计数，持有这个指针的使用者越多，计数就越高，使用完了之后，计数也会相应地减少；当没有任何使用者之后，指针就会自动释放它指向的对象。
    - 这与现代 C++ 中的智能指针并没有什么明显区别，CEF 框架大量使用了类似的智能指针，减轻了开发者的心智负担。
- CEF 框架初始化及消息循环
    - 接下来主进程会执行 CefInitialize 方法，这个方法负责初始化 CEF 的浏览器进程处理类（注意：后文我们提到的浏览器进程与前文提到的主进程属于同一个进程）。这个方法的第一个参数仍然是我们前面创建的 CefMainArgs 对象，第二个参数是 CefSettings 对象，第三个参数就是 App 对象的指针，这里是通过 CefRefPtr 智能指针的 get 方法获取的。第四个参数与沙箱有关，这里我们依然置空。
    - CefRunMessageLoop 负责开启 CEF 消息循环，这个方法会阻塞后面代码的执行，一直到应用程序的某个地方调用了 CefQuitMessageLoop 方法之后，这个方法才会退出执行。（CefQuitMessageLoop 方法会发射应用程序退出的消息，CefRunMessageLoop 方法会收到这个消息，收到这个消息后就退出方法了。）
    - CefShutdown 方法会结束主进程，释放资源。最后应用程序退出。

2. 浏览器进程入口
- 操作系统调用完程序的入口函数后，CEF 框架就通过其自身的消息循环机制接管了接下来的执行工作，在上一小节中我们提到了自定义的 App 对象，并且把这个对象传递给了 CEF 的 CefInitialize 方法，CEF 框架收到这个对象之后，会把浏览器进程的一些逻辑交给 App 对象执行，也就是说 App 对象就是我们浏览器进程的入口程序。先看一下它的头文件的代码：
    ```cpp
    #pragma once
    #include "include/cef_app.h"
    class App : public CefApp, public CefBrowserProcessHandler
    {
    public:
        App() = default;
        CefRefPtr<CefBrowserProcessHandler> GetBrowserProcessHandler() override { return this; }
        void OnContextInitialized() override;
    private:
        IMPLEMENT_REFCOUNTING(App);
    };
    ```
- 头文件里的宏
    - 头文件中 #pragma once 宏指令告诉编译器这个文件只会被编译一次。这是现代 C++ 编译器新增的一个指令，这个指令出现之前 C++ 开发者都是通过如下方式来保证头文件不会被重复编译的。
        ```cpp
        #ifndef _FileA
        #define _FileA
        // code 
        #endif
        ```
        - 这种方式虽然可以兼容古老的低版本编译器，但书写起来繁琐，编译时要预先分析文件内容，又非常低效，所以推荐使用 #pragma once 指令。
    - App 类的头文件中使用了宏 IMPLEMENT_REFCOUNTING ，这个宏为 App 类附加了一些特殊的方法，这些方法保证 App 类型的对象指针可以被 CefRefPtr 包裹。
- 浏览器进程的行为
    - App 类继承自 CefApp 和 CefBrowserProcessHandler 类，这两个基类提供了对浏览器进程的行为描述，比如：OnContextInitialized（浏览器进程的渲染线程初始化成功后被调用）、OnBeforeCommandLineProcessing （命令行参数被 CEF 和 Chromium 处理之前触发）等。我们在 App 类里只重写了 2 个基类方法，下面我们一个一个来介绍。
    - GetBrowserProcessHandler 方法返回浏览器进程的处理类实例指针，这里我们返回了 App 类的实例指针自身（也就是它自己）。浏览器进程内可能会有多个线程在执行，任何一个线程都有可能调用这个方法。
    - OnContextInitialized 方法在浏览器进程的主线程初始化成功后被调用，代表着浏览器进程已经初始化成功了。
- 窗口创建逻辑
    - 我们在 OnContextInitialized 方法中创建了第一个窗口，这个方法代码在类的源码文件中完成，下面我们来看一下它的代码：
        ```cpp
        //App.cpp
        #include "App.h"
        #include "include/cef_browser.h"
        #include "include/views/cef_browser_view.h"
        #include "include/views/cef_window.h"
        #include "include/wrapper/cef_helpers.h"
        #include "WindowDelegate.h"
        //CEF主进程上下文环境初始化成功
        void App::OnContextInitialized() {
            CEF_REQUIRE_UI_THREAD();
            auto url = "https://www.baidu.com";
            CefBrowserSettings settings;
            CefRefPtr<CefBrowserView> browser_view = CefBrowserView::CreateBrowserView(nullptr, url, settings, nullptr, nullptr, nullptr);
            CefWindow::CreateTopLevelWindow(new WindowDelegate(browser_view));
        }
        ```
    - 这个方法的实现逻辑稍微多一些，我们一步步详细解释这段代码的执行逻辑。
        - CEF_REQUIRE_UI_THREAD() 是一个宏，这个宏保证执行此方法的是浏览器进程的主线程。
        - url 字符串变量用于存放应用的首页地址。
        - settings 对象，是 CEF 配置对象，它的作用我们前文已经说过了，这里同样也是只用它默认的配置。
        - browser_view 是我们通过 CefBrowserView 类的 CreateBrowserView 静态方法创建了一个 BrowserView 对象。这个方法的第一个参数是一个 CefClient 类型的智能指针，它负责处理一切与页面相关的事件，比如下载、拖动、聚焦等，这里我们没有设置它的值，直接传递了一个空指针，要求 CEF 框架按照默认的行为处理页面相关的事件； url 字符串和 settings 对象作为第二个和第三个参数；第四个参数是一个用户自定义的附加信息对象，我们暂时用不到它；第五个参数是一个处理页面请求的对象，我们就使用 CEF 默认的请求处理机制，所以暂时也用不到它；第六个参数是 BrowserView 的代理对象，我们可以通过这个对象处理一些与 BrowserView 相关的事件，比如与该 BrowserView 关联的 Browser 对象创建成功的事件，为了简便，我们也没有用这个参数。
        - 最后我们通过 CefWindow 类的 CreateTopLevelWindow 静态方法创建了一个窗口，这个方法只有一个参数，就是 CefWindowDelegate 对象。WindowDelegate 是我们自定义的一个类型，这个类型继承自 CefWindowDelegate 对象，所以它的实例也是 CefWindowDelegate 类型的对象，我们通过这个 WindowDelegate 对象设置窗口的大小，聚焦窗口内的页面等，接下去我们就介绍它。

3. 窗口代理对象
- 前文我们介绍了当浏览器进程的主线程初始化成功后，App 对象的 OnContextInitialized 方法会被执行，在这个方法的最后，我们通过 CreateTopLevelWindow 方法为 CEF 框架提供了一个窗口代理对象。 CEF 框架会把与窗口创建有关的逻辑交给这个对象来执行，接下来我们就看一下这个对象的头文件代码：
    ```cpp
    // WindowDelegate.h
    #pragma once
    #include "include/views/cef_window.h"
    #include "include/views/cef_browser_view.h"
    class WindowDelegate : public CefWindowDelegate
    {
    public:
        explicit WindowDelegate(CefRefPtr<CefBrowserView> browser_view) : browser_view_(browser_view) {};
        void OnWindowCreated(CefRefPtr<CefWindow> window) override;
        void OnWindowDestroyed(CefRefPtr<CefWindow> window) override;
        CefRect GetInitialBounds(CefRefPtr<CefWindow> window) override;
        WindowDelegate(const WindowDelegate&) = delete;
        WindowDelegate& operator=(const WindowDelegate&) = delete;
    private:
        CefRefPtr<CefBrowserView> browser_view_;
        IMPLEMENT_REFCOUNTING(WindowDelegate);
    };
    ```
    - WindowDelegate 类继承自 CefWindowDelegate ，它的构造函数接收一个 CefBrowserView 类型的智能指针，并把这个智能指针存放到 browser_view_ 私有变量中，以备后续使用。
    - 与 App 类一样，它也使用了 IMPLEMENT_REFCOUNTING 宏，除此之外，它还删除了拷贝和赋值操作（注意头文件中的两个 “...= delete” 语句）。
    - 这个类实现了父类的三个方法，CEF 框架会在适当的时机调用这三个方法，一般我们可以把它理解为窗口生命周期内的事件，它们分别是：设置窗口位置和大小事件、窗口创建成功事件、窗口销毁成功事件。接下来我们看一下它们的实现代码并一一介绍它们的功用。
        ```cpp
        //WindowDelegate.cpp
        #include "WindowDelegate.h"
        #include "include/cef_app.h"
        #include "include/views/cef_display.h"
        //窗口创建成功
        void WindowDelegate::OnWindowCreated(CefRefPtr<CefWindow> window) {
            window->AddChildView(browser_view_);
            window->Show();
            browser_view_->RequestFocus();
            window->SetTitle(L"这是我的窗口标题");
            //window->CenterWindow(CefSize(800, 600));
        }
        //窗口销毁成功
        void WindowDelegate::OnWindowDestroyed(CefRefPtr<CefWindow> window) {
            browser_view_ = nullptr;
            CefQuitMessageLoop();
        }
        //设置窗口位置和大小
        CefRect WindowDelegate::GetInitialBounds(CefRefPtr<CefWindow> window) {
            CefRefPtr<CefDisplay> display = CefDisplay::GetPrimaryDisplay();
            CefRect rect = display->GetBounds();
            rect.x = (rect.width - 800) / 2;
            rect.y = (rect.height - 600) / 2;
            rect.width = 800;
            rect.height = 600;
            return rect;
        }
        ```
- 设置窗口位置和大小
    - 当 App 类 CreateTopLevelWindow 方法被执行后，CEF 框架将创建一个系统窗口，创建这个窗口之前，CEF 框架会调用 GetInitialBounds 方法，在这个方法中我们做了如下几个工作。
        - 设定窗口的尺寸为宽 800 像素，高 600 像素。
        - 通过 CefDisplay 类的静态方法 GetPrimaryDisplay 获取到了用户的主屏幕信息。
        - 根据主屏幕信息及设定的窗口尺寸计算出窗口位于屏幕正中间时窗口的坐标。
        - 通过 CefRect 结构把窗口的坐标及尺寸返回给 CEF 框架。
    - 我们可以在窗口创建成功之后再通过窗口对象的 CenterWindow 方法来把窗口设置到屏幕正中间（同时也可以设置窗口尺寸），但这显然不如在窗口创建之初就明确窗口的位置和尺寸更高效。如果开发者不通过 GetInitialBounds 方法设置窗口的尺寸，还可以通过重写基类的 GetPreferredSize 方法来设置窗口尺寸。 CEF 的示例项目就是这么做的，但我认为还是在 GetInitialBounds 方法中完成这项工作比较好。
- 窗口创建成功事件
    - 当 App 类 CreateTopLevelWindow 方法被执行后，CEF 框架将创建一个系统窗口，当这个窗口成功创建完成后，OnWindowCreated 被调用，我们在这个方法里把 App 类里创建的 BrowserView 对象添加到了这个窗口中（ window->AddChildView ），然后让这个窗口显示出来（ window->Show ），最后用户焦点被聚焦在 BrowserView 上（ browser_view_->RequestFocus ），最后一行注释掉的代码就是把窗口移动到主屏幕中央的代码。
- 窗口销毁成功事件
    - 当窗口被销毁后（可能是用户点击了窗口的关闭按钮，也可能是代码逻辑触发了窗口关闭的方法），OnWindowDestroyed 方法被执行，此处我们把 browser_view_ 指针置空。接着我们执行了 CefQuitMessageLoop 方法，这个方法会在 CEF 的消息循环中插入一个退出消息，CEF 框架收到这个消息后，会退出消息循环，清理资源，退出应用。

4. 在这一节我们通过创建一个精简的 CEF 应用程序，来带领你熟悉 CEF 框架的运作机制，可总结为如下：
    - 操作系统调用应用程序入口函数；
    - 入口函数初始化 CEF 框架并进入消息循环；
    - CEF 框架初始化浏览器进程并把操作逻辑转交给 BrowserProcessHandler 对象；
    - BrowserProcessHandler 对象创建 BrowserView 和 WindowDelegate 对象；
    - 第一个窗口创建成功后，CEF 会通知 WindowDelegate 对象，并由它把 BrowserView 对象附加到窗口上。

5. 为了解决加载本地页面的问题， CEF 提供了 CefSchemeHandlerFactory 和 CefResourceHandler 等类型和方法支持用户自定义协议。下面我们就介绍第一个方案，如何使用 CEF 内置的协议加载本地页面。

6. 注册内置协议处理工厂
- 要想让应用能通过自定义的内置协议处理本地页面，首先需要为 CEF 框架注册一个自定义协议处理工厂，这个工作我们在 App 类中的 OnContextInitialized 方法内完成，代码所示：
    ```cpp
    void App::OnContextInitialized() {
        CEF_REQUIRE_UI_THREAD();
        //下面两行代码为新增代码
        CefRegisterSchemeHandlerFactory("https", "bread", new HttpSchemeFactory());
        std::string url = "https://bread/index.html?a=123";
        CefBrowserSettings settings;
        CefRefPtr<CefBrowserView> browser_view = CefBrowserView::CreateBrowserView(nullptr, url, settings, nullptr, nullptr, nullptr);
        CefWindow::CreateTopLevelWindow(new WindowDelegate(browser_view));
    }
    ```
    - 从上面的代码可以看出，我们在浏览器主线程初始化成功后，调用了这个 CEF 提供的 API ：CefRegisterSchemeHandlerFactory。这个方法为应用注册一个 HttpSchemeFactory 对象，注册成功则返回 true ，失败则返回 false 。
    - 这个方法的第一个参数是开发者要注册的协议名称，此时我们设置的是一个 CEF 框架内置支持的协议： HTTPS （其他的内置协议还有 HTTP 、 File 、 FTP 、 About 和 Data 等。另外值得注意的是，在这里使用 HTTPS 协议是无需配置证书的）。
    - 第二个参数是域名，这里的域名不一定以 .com 或者 .cn 结尾，你可以随意输入一个小写字符串，如果输入的是空字符串，那么 CEF 框架将为你匹配所有的域名。
    - 第三个参数是自定义协议工厂，这是我们自己实现的类，名为：HttpSchemeFactory。
    - 当应用成功注册 HttpSchemeFactory 后，接下来我们把第一个窗口的页面地址设置成 https://bread/index.html?a=123 ，虽然这个地址以 https scheme 开头，但它显然不是一个互联网地址（它没有域名后缀），接下来我们就介绍如何通过自定义的 HttpSchemeFactory 类型处理这个请求。

7. 接管并处理请求
- HttpSchemeFactory 类的工作很简单，当浏览器以指定的自定义协议加载页面时，它负责接管并处理请求。我们先来看一下这个类的头文件代码：
    ```cpp
    //文件名为：HttpSchemeFactory.h
    #pragma once
    #include "include/cef_app.h"
    class HttpSchemeFactory : public CefSchemeHandlerFactory
    {
    public:
        HttpSchemeFactory() = default;    
        //删除拷贝函数
        HttpSchemeFactory(const HttpSchemeFactory&) = delete;
        //删除赋值函数
        HttpSchemeFactory& operator=(const HttpSchemeFactory&) = delete;
        //处理请求的方法定义
        CefRefPtr<CefResourceHandler> Create(CefRefPtr<CefBrowser> browser, CefRefPtr<CefFrame> frame, const CefString& scheme_name, CefRefPtr<CefRequest> request) override;
    private:
        IMPLEMENT_REFCOUNTING(HttpSchemeFactory);
    };
    ```
    - 虽然 CEF 示例项目是使用 DISALLOW_COPY_AND_ASSIGN 宏来完成类拷贝和赋值函数的删除的，但这样做并不理想，而且这个宏已经被标记为弃用了。建议大家按照本文介绍的方法来禁用一个类的赋值和拷贝方法。
    - 但有些类涉及到一些资源操作或跨进程异步操作，不希望编译器提供这两种行为的默认实现，比如：C++ 标准库中的 iostream 类就阻止了拷贝行为，以避免多个对象写入或读取相同的 IO 缓冲。还有一些类持有的数据特别多，这就会导致拷贝或赋值非常损耗性能，基于这方面的理由，它们也会阻止拷贝和赋值行为。
- HttpSchemeFactory 这个类继承自 CefSchemeHandlerFactory 基类，这个基类只有一个虚方法就是Create，当浏览器核心以用户自定义的协议发起请求时，CEF 框架会调用这个方法，我们在这个方法里接管了浏览器核心的请求，并给出响应。以下是这个方法的实现代码：
    ```cpp
    //文件名为：HttpSchemeFactory.cpp
    #include "HttpSchemeFactory.h"
    #include "include/wrapper/cef_helpers.h"
    #include "include/wrapper/cef_stream_resource_handler.h"
    #include <fstream>
    #include <filesystem>
    #include <iostream>
    //处理请求的方法实现
    CefRefPtr<CefResourceHandler> HttpSchemeFactory::Create(CefRefPtr<CefBrowser> browser, 
                                                            CefRefPtr<CefFrame> frame, 
                                                            const CefString& scheme_name, 
                                                            CefRefPtr<CefRequest> request)
    {
        CEF_REQUIRE_IO_THREAD();
        std::string url = request->GetURL().ToString();
        std::string urlPrefix = "https://bread/";
        url.erase(0, urlPrefix.size());
        size_t paramIndex = url.find_first_of('?');
        if (paramIndex != std::string::npos) {
            url.erase(paramIndex);
        }
        TCHAR buffer[MAX_PATH] = { 0 };
        GetModuleFileName(NULL, buffer, MAX_PATH);
        std::filesystem::path targetPath(buffer);
        targetPath = targetPath.parent_path().append("html").append(url);
        if (!std::filesystem::exists(targetPath)) {
            DLOG(INFO) << L"试图加载：" << targetPath.generic_wstring() << L"，但找不到这个文件";
        }
        std::string ext = targetPath.extension().generic_string();
        std::string mime_type_ = ext == ".html"? "text/html":"*";
        auto stream = CefStreamReader::CreateForFile(targetPath.generic_wstring());
        return new CefStreamResourceHandler(mime_type_, stream);
    };
    ```
    - 这个方法主要完成了以下几项工作：
        - 使用 CEF_REQUIRE_IO_THREAD 宏来确保此方法的逻辑运行在 IO 线程上，这类线程只负责处理 IPC 消息和网络消息（ IPC 消息是 CEF 框架用于进程间通信的消息），开发者应该避免在此类线程上执行阻塞性工作。
        - 从 request 参数中获取浏览器请求的 url 路径，稍后我们会把窗口加载的页面地址设置为：https://bread/index.html?a=123，并去掉了路径中协议和域名的部分（也就是https://bread/），如果路径中存在 url 参数的话也会被去掉（也就是 ?a=123 ）。这样就得到了被请求的资源的相对路径，也就是 index.html。
        - 通过 GetModuleFileName 方法得到当前可执行文件的路径（注意，这是一个 Windows 平台独有的方法，Mac 平台下无法使用此方法），并通过这个路径得到静态资源所在目录（是一个名为 html 的子目录），接着拼接上被请求的资源的相对路径，这样就得到了被请求资源的绝对路径（在我的电脑上这个路径为：E:\project\cef_test\x64\Debug\html）。
        - 获取被请求资源的扩展名，然后通过扩展名确定 MimeType，这里我们只给 html 文件设置了 MimeType ，其他文件的 MimeType 都以 * 代替，Chromium 浏览器核心能识别常见的静态资源的 MimeType 类型。
        - 通过 CefStreamReader 类的静态方法 CreateForFile 得到了一个 CefStreamReader 对象，最终把这个对象连同 MimeType 一起，封装到了 CefStreamResourceHandler 对象中，并把这个对象返回给 CEF 框架。 CEF 框架会把本地文件的数据响应给 Chromium 浏览器核心。
    - Create 方法的第一个参数是 CefBrowser 类型的对象。目前为止我们可以把它理解为浏览器实例，当前我们使用的这个窗口，就持有这么一个浏览器实例。
    - 第二个参数是 CefFrame 类型的对象，它对应到页面中的 iframe 子页面，通过它可以对页面中的 iframe 进行操作。第三个参数是协议类型字符串，在本示例中就是 https 。第四个参数是请求对象，它包括浏览器的请求头、请求来源等信息，开发者也可以通过这个对象修改浏览器的请求信息。
    - 字符串的 erase 方法负责清除字符串中指定位置处的内容。

8. 验证自定义协议
- 为了验证自定义协议，我们需要在我们的工程中添加两个静态文件 a.js 和 index.html ，并分别设置它们的属性，让 Visual Studio 在启动工程前把它们复制到输出目录下。
- 这两个文件的代码比较简单， index.html 的代码为：
    ```html
    <html>
      <head>
        <title>第一个本地页面</title>
        <meta charset="utf-8" />
      </head>
      <body>
        这是第一个本地页面<br />
        它的地址是：
        <script src="/a.js"></script>
      </body>
    </html>
    ```
- a.js 的代码是：
    ```js
    document.write(location.href);
    ``` 
    - 这行代码的意思是在页面加载成功后显示它当前的页面地址。
- 这个程序说明了两个问题：
    - 我们的自定义内置协议 https://bread 可以正常使用；
    - 我们在页面中可以使用根目录路径 /a.js 加载资源（当浏览器请求 a.js 时，HttpSchemeFactory 的 Create 方法也会被执行，以响应 a.js 的内容）。

9. 被 HttpSchemeFactory 工厂类处理的请求除了要匹配 https scheme 之外，还要匹配我们注册的域名： bread 。另外注册的 HTTPS 协议处理工厂只对当前应用生效，不会影响用户操作系统内的其他应用，不用担心应用的兼容性和安全性问题。所以，这个方案适应性还是比较广泛的。

10. 前面介绍了如何使用内置的 HTTPS 协议加载本地页面，实际上所有的内置协议 FTP、 Data 等都可以使用类似的方式加载本地页面。但这也仅仅局限在内置协议的范畴，如果我们要注册一个完全自定义的协议，这种做法就行不通了。本讲中我就带领你注册一个完全自定义的协议。

11. 分离进程处理类
- 使用自定义的协议处理请求，必须要在应用的每个进程下注册这个自定义的协议，所以在完成这项工作前，必须要把不同进程的处理逻辑剥离出来。
- 前面介绍了应用的入口函数，在入口函数中我们通过调用 CefExecuteProcess 方法来启动不同的进程，这个方法的第二个参数就是对应进程的处理逻辑，由于我们当时并没有用到它，所以传递了一个空指针，让 CEF 框架帮我们处理不同进程的业务逻辑。
- 现在我们就要使用它来接管这些进程的处理逻辑，为入口函数增加如下代码：
    ```cpp
    CefMainArgs main_args(hInstance);
    CefRefPtr<CefCommandLine> command_line = CefCommandLine::CreateCommandLine();
    command_line->InitFromString(::GetCommandLineW());
    CefRefPtr<CefApp> app;
    if (!command_line->HasSwitch("type")) {
        app = new App();
    }
    else if (command_line->GetSwitchValue("type").ToString() == "renderer") {
        app = new Renderer();
    }
    else {
        app = new Other();
    }
    int exit_code = CefExecuteProcess(main_args, app, nullptr);
    ```
    - 在这段代码中，我们根据命令行参数来判断当前进程属于什么类型的进程，如果命令行参数中不包含 type 参数，那么当前进程就是浏览器进程，我们为这个进程创建了 App 类型的实例，让它接管浏览器进程的处理逻辑。
    - 如果命令行参数 type 的值为 renderer ，说明正在启动的进程为渲染进程，那么我们创建自定义类型 Renderer 的实例。
    - 如果命令行参数 type 的值不是 renderer ，那么我们创建了自定义类型 Other 的实例，此时有可能是 GPU 进程，也有可能是辅助进程，但我们用同一个类接管这类进程的处理逻辑。
    - 无论是 App 、 Renderer 还是 Other ，这三个自定义类型都继承自 CEF 的 CefApp 类，最终这三个类的实例都作为第二个参数传递给了 CefExecuteProcess 方法（这个参数的类型就是 CefApp ）。

12. 为不同进程注册协议
- 分离出不同进程的处理类之后，为了完成自定义协议的注册工作，我们需要在这三个类中重写 CefApp 类的虚方法 OnRegisterCustomSchemes，不过它们的重写逻辑都是相同的（注意，方法前缀 App:: 要改为对应的类名），代码如下：
    ```cpp
    void App::OnRegisterCustomSchemes(CefRawPtr<CefSchemeRegistrar> registrar) {
        registrar->AddCustomScheme("my", CEF_SCHEME_OPTION_STANDARD | CEF_SCHEME_OPTION_CORS_ENABLED);
    }
    ```
- 实际上开发者也可以定义一个类型，比如说叫 MyApp，然后让 App 、 Renderer 和 Other 这三个类都继承自 MyApp（不必再继承自 CefApp 了），而 MyApp 则继承自 CefApp，这样只要在 MyApp 类中重写这个 OnRegisterCustomSchemes 方法就可以了。最终达到的效果就是 App 、 Renderer 和 Other 三个类型都拥有 CefApp 的"身份"，并且它们都实现了 OnRegisterCustomSchemes 方法。而且我们将来还可以为 App 、 Renderer 和 Other 三个类型扩展更多的方法，让它们拥有更多的能力。
- CEF 框架允许我们在 OnRegisterCustomSchemes 方法被调用时，为不同的进程注册协议。第一个参数就是协议的名称，如你所见，我们自定义的协议名称为 my ；第二个参数为处理这些协议的方式，其中： CEF_SCHEME_OPTION_CORS_ENABLED 代表着允许跨域请求。 CEF_SCHEME_OPTION_STANDARD 代表着标准协议处理方式，也就是要遵循如下这种类型的 URL 格式：
    ```
    [scheme]://[username]:[password]@[host]:[port]/[url-path]
    ```
- 为每个进程注册这个自定义协议后，我们把 App 类中的 OnContextInitialized 方法内注册协议的名称改为 my ，接着再把 HttpSchemeFactory 类中 Create 方法内的 urlPrefix 改为 "my://bread/" 就可以加载如下 url 地址啦：
    ```
    my://bread/index.html?a=123
    ```
- 此时，App 类的 OnContextInitialized 方法被修改为：
    ```cpp
    void App::OnContextInitialized() {
       CEF_REQUIRE_UI_THREAD();
       CefRegisterSchemeHandlerFactory("my", "bread", new HttpSchemeFactory());
       std::string url = "my://bread/index.html?a=123";
       CefBrowserSettings settings;
       CefRefPtr<CefBrowserView> browser_view = CefBrowserView::CreateBrowserView(nullptr, url, settings, nullptr, nullptr, nullptr);
       CefWindow::CreateTopLevelWindow(new WindowDelegate(browser_view));
    }
    ```
- 虽然这个协议并非 Chromium 内置支持的协议，但它处理静态资源的方式和内置协议的方式一模一样（上一节定义的 HttpSchemeFactory 类在这里仍然工作得很好），所以我们仍然可以把 "my://bread/" 这个域视作一个常规的互联网域。同样，这个方案也能解决 File 协议带来的问题。

13. 自定义资源处理类
- 前面提到的两个方案都只能加载本地的文件资源，如果我们的 HTML、JS、CSS 等静态文件是加密存储的，或者是从二进制资源文件中获取的，又该如何加载呢？接下来我们就介绍如何实现一个 ResourceHandler 来处理这种需求。
- 为了更自由地加载本地资源，首先我们模仿 HttpSchemeFactory 类，创建一个 CustomSchemeFactory 类，它的 Create 方法没有太多逻辑，只是返回了一个自定义类型 CustomSchemeHandler 的实例，实现代码如下（头文件的代码与 HttpSchemeFactory 头文件代码类似）：
    ```cpp
    CefRefPtr<CefResourceHandler> CustomSchemeFactory::Create(CefRefPtr<CefBrowser> browser, 
                                                              CefRefPtr<CefFrame> frame, 
                                                              const CefString& scheme_name, 
                                                              CefRefPtr<CefRequest> request)
    {
        CEF_REQUIRE_IO_THREAD();
        return new CustomSchemeHandler();
    };
    ```
- 这个 CustomSchemeHandler 就是我们的资源处理类，代码如下所示
    ```cpp
    // CustomSchemeHandler.h
    #pragma once
    #include "include/cef_app.h"
    #include "include/cef_resource_handler.h"
    #include "include/wrapper/cef_helpers.h"
    class CustomSchemeHandler : public CefResourceHandler
    {
    public:
        CustomSchemeHandler() : offset_(0) {}
        CustomSchemeHandler(const CustomSchemeHandler&) = delete;
        CustomSchemeHandler& operator=(const CustomSchemeHandler&) = delete;
        //请求发生时触发的事件
        bool Open(CefRefPtr<CefRequest> request, bool& handle_request, CefRefPtr<CefCallback> callback) override;
        //发送响应头之前触发的事件
        void GetResponseHeaders(CefRefPtr<CefResponse> response, int64& response_length, CefString& redirectUrl) override;
        //请求被取消时触发的事件
        void Cancel() override { CEF_REQUIRE_IO_THREAD(); }
        //响应数据时触发的事件
        bool Read(void* data_out, int bytes_to_read, int& bytes_read, CefRefPtr<CefResourceReadCallback> callback) override;
    private:
        std::string data_;
        size_t offset_;
        IMPLEMENT_REFCOUNTING(CustomSchemeHandler);
    };
    ```
    - 这个类继承自 CefResourceHandler 类型，当请求发生时，CEF 框架会调用这个类的 Open 方法。当处理响应头时，CEF 框架会调用这个类的 GetResponseHeaders 方法；当处理响应体时，CEF 框架会调用这个类的 Read 方法（如果响应的数据比较多，那么这个方法可能会被多次调用，每次调用响应一部分数据内容）。 当请求被取消时，Cancel 方法被执行，虽然这个方法没什么实现逻辑，但我们必须重写它，如果不重写它，CEF 框架则认为这个类型是不完整的。
- 在这个类型实例化时，我们初始化了一个名为 offset_ 的变量，这个变量负责记录响应请求时响应数据块的位置。
- 接下来我们看一下这个类描述的方法的实现代码：
    ```cpp
    // CustomSchemeHandler.cpp
    #include "CustomSchemeHandler.h"
    #include "include/wrapper/cef_helpers.h"
    #include <fstream>
    #include <filesystem>
    #include <iostream>
    //请求发生时触发的事件
    bool CustomSchemeHandler::Open(CefRefPtr<CefRequest> request, bool& handle_request, CefRefPtr<CefCallback> callback) {
        DCHECK(!CefCurrentlyOn(TID_UI) && !CefCurrentlyOn(TID_IO));
        handle_request = true;
        this->data_ = "<html><head><title>这是我自己的页面</title></head><body>这是我自己的内容</body></html>";
        return true;
    }
    //发送响应头之前触发的事件
    void CustomSchemeHandler::GetResponseHeaders(CefRefPtr<CefResponse> response, int64& response_length, CefString& redirectUrl) {
        CEF_REQUIRE_IO_THREAD();
        DCHECK(!data_.empty());
        response->SetMimeType("text/html");
        response->SetStatus(200);
        response_length = data_.length();
    }
    //响应数据时触发的事件
    bool CustomSchemeHandler::Read(void* data_out, int bytes_to_read, int& bytes_read, CefRefPtr<CefResourceReadCallback> callback) {
        DCHECK(!CefCurrentlyOn(TID_UI) && !CefCurrentlyOn(TID_IO));
        bool has_data = false;
        bytes_read = 0;
        if (offset_ < data_.length()) {
            int transfer_size = std::min(bytes_to_read, static_cast<int>(data_.length() - offset_));
            memcpy(data_out, data_.c_str() + offset_, transfer_size);
            offset_ += transfer_size;
            bytes_read = transfer_size;
            has_data = true;
        }
        return has_data;
    }
    ```
    - 在请求发生时，也就是在 Open 方法执行时，我们就准备好了待响应的数据，目前我们只设置了一个字符串，你可以写代码读取本地静态文件或读取保存在二进制资源文件中的静态文件，甚至可以考虑在这里把读取来的文件缓存到内存中，应用下次请求同一个路径的静态资源时，就可以在缓存中获取了（你应该仔细衡量哪些资源该这么做，毕竟 Chromium 也内置了缓存机制）。
    - 在 Open 方法执行时，还把参数 handle_request 的值被设置为 true ，并且返回值也是 true ，说明我们已经接管了这个请求。如果 handle_request 的值为 false，并且返回值为 true ，那么开发者需要使用 callback 参数来决定什么时候继续执行（或取消执行）本次请求。如果开发者想立即取消本次请求，那么只需要把 handle_request 设置为 true ，返回 false 即可。
    - 我们在 GetResponseHeaders 方法中设置了响应的 MimeType 和状态码，并通过设置 response_length 告知 CEF 框架我们打算响应的数据的长度。
    - Read 方法可能会被反复执行多次（并不一定是在同一个线程内被反复调用），每执行一次，我们就会响应一部分数据，直到把所有的数据都输出给客户端为止。
    - Read 方法的第一个参数是一个数据指针，我们通过memcpy方法向这个指针指向的地址写入数据，随后 CEF 会把这里的数据渲染到界面上。
    - Read 方法的第二个参数 bytes_to_read 是 CEF 框架指定的本次输出的数据的长度，不过有可能待输出的数据并没有这么多，所以我们通过 std::min 方法得到一个精确的待输出数据长度值。
    - Read 方法的第三个参数 bytes_read 是本次真实输出的数据的长度，是我们的业务代码告诉 CEF 框架的，CEF 框架会把这个真实的长度传递给 Chromium 渲染引擎。
    - 如果开发者想在未来的某个时候再响应数据，那么可以保存一下 data_out 指针，设置 bytes_read 的值为 0 并返回 true ，这样就可以在未来的某个时候通过第四个参数callback响应数据了。数据处理完成后，只要把 bytes_read 设置为 0 并返回 false 即可。 

---

###### 三、窗口控制

1. 新窗口没有大小
- 如果在我的第一个页面中，增加一个 target 为 \_blank 的链接（只有把 target 属性设置为 \_blank 时，用户点击链接才会打开新窗口），如下代码所示：
    ```html
    <a href="my://bread/index.html?a=123" target="_blank">打开一个新窗口</a>
    ```
    - 注意，这里我们尝试打开的是我们自定义协议所指向的页面地址，而且这个地址就是当前页面的地址，也就是说打开的新窗口加载的也是当前页面，而且在新窗口也可以通过点击这个连接打开另一个新窗口。
    - 运行程序，点击这个连接，虽然可以打开一个新窗口，但这个窗口却没有尺寸。
- 实际上，如果我们没有为第一个窗口提供 WindowDelegate 对象的话，我们的第一个窗口也会是这样的，但第一个窗口的 WindowDelegate 对象是我们在 CreateTopLevelWindow 时手动附加上去的，那么类似这种用户自己打开的窗口我们该如何动态把 WindowDelegate 对象附加上去呢？     
- 用户打开新窗口这个事件是发生在 ViewDelegate 对象上的，我们在前面的示例中并没有定义这个对象，所以就没有机会把 WindowDelegate 附加到新窗口上去。接下来我们就定义一下这个对象。

2. 创建新窗口的事件
- 这个 ViewDelegate 对象是在 App 类的 OnContextInitialized 方法内创建的，如下代码所示：
    ```cpp
    CefRefPtr<ViewDelegate> viewDelegate(new ViewDelegate());
    CefRefPtr<CefBrowserView> browserView = CefBrowserView::CreateBrowserView(pageHandler, url, settings, nullptr, nullptr, viewDelegate);
    ```
- ViewDelegate 是我们自定义的一个类型，它继承自 CefBrowserViewDelegate 类，我们通过 CreateBrowserView 方法的最后一个参数（之前这个参数传递的是 nullptr ）把这个对象传递给 CEF 框架，当 CEF 框架成功创建出 browserView 对象之后，这个 browserView 对象就和 viewDelegate 对象建立了联系。接下来我们看一下 ViewDelegate 的头文件代码：
    ```cpp
    #pragma once
    #include "include/views/cef_browser_view.h"
    #include "include/views/cef_window.h"
    class ViewDelegate : public CefBrowserViewDelegate
    {
    public:
        ViewDelegate() = default;
        ViewDelegate(const ViewDelegate&) = delete;
        ViewDelegate& operator=(const ViewDelegate&) = delete;
        // 当前页面弹出新窗口时此方法被执行
        bool OnPopupBrowserViewCreated(CefRefPtr<CefBrowserView> browser_view, CefRefPtr<CefBrowserView> popup_browser_view, bool is_devtools) override;
    private:
        IMPLEMENT_REFCOUNTING(ViewDelegate);
    };
    ```
    - CefBrowserViewDelegate 类声明了很多与 BrowserView 相关的事件方法，这里我们只实现了它的 OnPopupBrowserViewCreated 方法，当页面请求弹出一个新页面时，这个方法被调用。
    - 方法的第一个参数是弹窗来源页面对应的 BrowserView，第二个参数是被弹出的页面对应的 BrowserView ，第三个参数是一个布尔值，代表着被弹出的页面是不是开发者调试工具（开发者调试工具也是一个页面）。
- 接下来我们看一下这个方法的实现代码：
    ```cpp
    #include "ViewDelegate.h"
    #include "WindowDelegate.h"
    // 当前页面弹出新窗口时此方法被执行
    bool ViewDelegate::OnPopupBrowserViewCreated(CefRefPtr<CefBrowserView> browserView, CefRefPtr<CefBrowserView> popupBrowserView, bool isDevtools)
    {
        CefWindow::CreateTopLevelWindow(new WindowDelegate(popupBrowserView));
        return true;
    }
    ```
    - 实现逻辑很简单，仅仅是通过 CefWindow 类的静态方法 CreateTopLevelWindow 创建了一个顶级窗口，创建这个窗口的时候我们把自定义的 WindowDelegate 对象附加到目标 BrowserView 上了，这与我们在 App 类中创建第一个窗口的逻辑没什么两样。
    - 值得注意的是，这个方法传入的 popupBrowserView 对象，默认也是被 ViewDelegate 控制的，也就是说在一个弹出窗口的页面上，通过点击连接再弹出一个窗口，也会进入 OnPopupBrowserViewCreated 事件。这就使得整个应用中所有的页面都具备了弹出新窗口这项能力。

3. 最后一个窗口关闭时退出应用
- 虽然程序可以成功地打开页内弹窗了，而且还可以在新弹窗里点击连接打开更多的弹窗，但你会发现我们的应用打开多个窗口之后，无法通过关闭窗口退出应用了。这是因为我们退出应用的逻辑是放置在窗口代理类的 OnWindowDestroyed 事件中的（我们在这个事件中调用了 CefQuitMessageLoop 方法），当这个事件在某一个窗口触发时，其他窗口和页面尚未释放，所以就爆出了异常信息。
- 为了解决这个问题，我们必须把创建的页面管理起来，明确是最后一个窗口关闭时再退出应用。 要想做到这一点就必须知道什么时候页面创建成功了、什么时候页面被销毁了。为此我们引入了一个新的类型：PageHandler。这个类型负责处理与页面有关的事件，同样它也是在 App 类的 OnContextInitialized方法中创建并与 BrowserView 进行关联的，代码如下所示：
    ```cpp
    CefRefPtr<PageHandler> pageHandler(new PageHandler());
    CefRefPtr<ViewDelegate> viewDelegate(new ViewDelegate());
    CefRefPtr<CefBrowserView> browserView = CefBrowserView::CreateBrowserView(pageHandler, url, settings, nullptr, nullptr, viewDelegate);
    ```
    - 上面这段代码中，我们把 PageHandler 的对象作为第一个参数传递给了 CreateBrowserView 方法，CEF 会把 PageHandler 的对象与我们创建的 BrowserView 对象关联起来。当页面创建成功时，PageHandler 类内定义的方法将被 CEF 调用。
- PageHandler 的头文件的代码如下：
    ```cpp
    #pragma once
    #include "include/cef_app.h"
    #include <list>
    class PageHandler : public CefClient, public CefLifeSpanHandler
    {
    public:
        PageHandler() = default;
        //获取LifeSpanHandler对象
        virtual CefRefPtr<CefLifeSpanHandler> GetLifeSpanHandler() override { return this; }
        //页面创建成功
        void OnAfterCreated(CefRefPtr<CefBrowser> browser) override;
        //页面即将关闭
        void OnBeforeClose(CefRefPtr<CefBrowser> browser) override;
    private:
        IMPLEMENT_REFCOUNTING(PageHandler);
        std::list<CefRefPtr<CefBrowser>> browsers;
    };
    ```
    - 这个类的第一个基类是 CefClient， CefClient 类定义了一系列的页面行为接口，并通过这些接口返回具体某项事务的处理对象，比如打印处理对象、展示处理对象、菜单处理对象等，在本案例中我们用到了它的 GetLifeSpanHandler 接口。这个接口返回一个 CefLifeSpanHandler 类型的实例指针，用于处理页面生命周期中的事件。
    - 为了简单，我们直接让 PageHandler 也继承自 CefLifeSpanHandler 类（第二个基类），这样在 GetLifeSpanHandler 方法时只要返回它自身就可以了。当然你也可以创建一个新的继承自 CefLifeSpanHandler 的类，在 GetLifeSpanHandler 方法中返回这个新类的实例指针。
    - 在这个类中我们实现了 CefLifeSpanHandler 父类的两个方法：OnAfterCreated 方法和 OnBeforeClose 方法。当页面被成功创建后，CEF 框架会主动调用 OnAfterCreated 方法；当页面被关闭之前，CEF框架会主动调用 OnBeforeClose 方法。这两个方法的入参都是一个 CefBrowser 指针。
    - 另外，我们还在这个类中创建了一个名为 browsers 的私有变量，这个变量是一个标准 C++ 的 list 容器，用于存放 CefBrowser 类型的指针。
- 下面我们来看一下 PageHandler 这个类的实现代码：
    ```cpp
    #include "PageHandler.h"
    #include "include/wrapper/cef_helpers.h"
    //页面创建成功
    void PageHandler::OnAfterCreated(CefRefPtr<CefBrowser> browser) {
        CEF_REQUIRE_UI_THREAD();    
        browsers.push_back(browser);
    }
    //页面即将关闭
    void PageHandler::OnBeforeClose(CefRefPtr<CefBrowser> browser) {
        CEF_REQUIRE_UI_THREAD();
        std::list<CefRefPtr<CefBrowser>>::iterator bit = browsers.begin();
        for (; bit != browsers.end(); ++bit) {
            if ((*bit)->IsSame(browser)) {
                browsers.erase(bit);
                break;
            }
        }
        if (browsers.empty()) {
            CefQuitMessageLoop();
        }
    }
    ```
    - 当页面被成功创建后，我们把得到的 browser 对象存入了 browsers 容器。当页面被关闭之前，我们遍历这个容器，并把即将被关闭的页面从容器中删除。最后判断容器是否为空，如果容器已经被清空了，说明所有的页面都关闭了，那么此时发射退出应用的消息。
    - 最后不要忘记修改 WindowDelegate 类的 OnWindowDestroyed 方法，把我们之前写的 CefQuitMessageLoop 语句删掉。

4. 页面阻止窗口关闭
- 阻止窗口关闭并请求用户确认的 JavaScript 代码如下所示：
    ```js
    window.onbeforeunload = function(){
        return "askForClose"
    }
    ```
    - 然而把上面这段代码加入到我们的 JavaScript 文件中却起不到任何阻止窗口关闭的效果。接下来我们就介绍 CEF 框架是如何让这段代码生效的。
    - 注意，弹窗中提示的内容与我们在 onbeforeunload 事件中返回的内容是不相同的，弹窗中的内容是由 Chromium 控制的。
- 首先，我们要在前文介绍过的WindowDelegate类中增加一个方法重载：
    ```cpp
    bool CanClose(CefRefPtr<CefWindow> window) override;
    ```
    - 这是它的父类CefWindowDelegate描述的方法，CEF 框架在窗口关闭前调用这个方法，如果窗口可以被关闭，那么这个方法返回 true ，如果窗口不能被关闭，那么这个方法返回 false 。如果开发者不重写这个方法，它永远返回 true，这就是我们无法阻止窗口关闭的原因。我们重写这个方法的代码如下：
        ```
        bool WindowDelegate::CanClose(CefRefPtr<CefWindow> window) {
            bool result = true;
            CefRefPtr<CefBrowser> browser = browserView->GetBrowser();
            if (browser) {
                result = browser->GetHost()->TryCloseBrowser();
            }
            return result;
        }
        ```
        - 在这段代码中，我们先通过 browserView 对象（这个对象是我们实例化 WindowDelegate 时传进来的，它的类型是 CefBrowserView ）获取到与它关联的 CefBrowser 对象，然后通过 CefBrowser 对象获取到 CefBrowserHost 对象，这个对象持有一些处理 CefBrowser 对象的方法。
- 接下来我们分两种情况介绍 CEF 窗口关闭的逻辑。
    - 如果页面没有注册 onbeforeunload 事件，那么当执行 CefBrowserHost 对象的 TryCloseBrowser 方法时，得到的结果为 true ，也就是说可以关闭窗口。 CEF 框架将再次调用 CanClose 方法，第二次调用此方法时 TryCloseBrowser 方法依然返回 true ，窗口关闭成功。
    - 如果页面注册了 onbeforeunload 事件，那么当执行 CefBrowserHost 对象的 TryCloseBrowser 方法时，得到的结果为 false ，也就是说页面在阻止窗口关闭，此时页面会弹出询问对话框，当用户选择了 Cancel 后，窗口关闭行为被成功阻止，CEF 框架也不会再次调用 CanClose 方法；当用户选择了 OK 后，CEF 框架将再次调用 CanClose 方法，此时 TryCloseBrowser 方法返回 true，窗口关闭成功。
- 这里有一点值得注意，新版本 Chromium 要求只有当用户与页面发生了交互行为，才会弹出这个询问对话框。也就是说，如果窗口创建成功后，不操作任何界面元素，马上点击窗口的关闭按钮，这个对话框是不会出现的，但这个时候 CanClose 方法依然被调用了两次，第一次返回 false （也就是说 TryCloseBrowser 方法返回了 false），第二次返回 true。
- 这就是 CEF 框架阻止窗口关闭的实现逻辑，但这个询问对话框无法更改里面的提示文字，很难满足产品的要求。

5. 自定义阻止页面关闭的对话框
- 要想自定义阻止页面关闭的对话框，就必须知道这个对话框是在什么时候创建的， CEF 框架为了允许开发者自定义 JavaScript 对话框，特地公开了这些对话框的创建事件，允许开发者在这个时机阻止默认对话框的创建，并显示自己的对话框。
- 首先，我们再为上一节的 PageHandler 类型添加一个基类：CefJSDialogHandler，并在 PageHandler 的头文件中加入如下代码：
    ```cpp
    // 现在 PageHandler 类的继承关系如下：
    // class PageHandler : public CefClient, public CefLifeSpanHandler, public CefJSDialogHandler
    CefRefPtr<CefJSDialogHandler> GetJSDialogHandler() override { return this; }
    bool OnBeforeUnloadDialog(CefRefPtr<CefBrowser> browser, const CefString& message_text, bool is_reload, CefRefPtr<CefJSDialogCallback> callback) override;
    ```
    - GetJSDialogHandler 返回的就是与 JavaScript 对话框相关的事务的处理对象。
    - OnBeforeUnloadDialog方法就是在 CefJSDialogHandler 基类中定义的，当阻止页面关闭的对话框弹出之前，CEF 框架将执行我们自定义的 OnBeforeUnloadDialog 方法。
    - 这个方法的第一个参数为浏览器对象，第二个参数为提示文本字符串，这个字符串并不是 JavaScript 代码中onbeforeunload事件的返回值，而是 CEF 定死的一个值：Is it OK to leave/refresh this page（这是 Chromium 规定的行为，所以即使我们自定义阻止页面关闭对话框，也无法显示 JavaScript 代码提供的阻止信息）。第三个参数代表着当前关闭页面的行为是不是刷新行为。
    - 如果这个方法返回 false ，那么就使用 CEF 内置的对话框，这也是 CEF 的默认行为。如果这个方法返回 true，那么就需要开发者自定义对话框，一旦开发者使用自己定义的对话框，那么必须要调用第四个参数 callback 对应的 Continue 方法，把用户选择的结果通知 CEF 框架。
- OnBeforeUnloadDialog 方法的实现代码如下所示：
    ```cpp
    bool PageHandler::OnBeforeUnloadDialog(CefRefPtr<CefBrowser> browser, const CefString& message_text, bool is_reload, CefRefPtr<CefJSDialogCallback> callback) {
        HWND hwnd = browser->GetHost()->GetWindowHandle();
        int msgboxID = MessageBox(hwnd, L"您编辑的内容尚未保存.\n确定要关闭窗口吗?", L"系统提示", MB_ICONEXCLAMATION | MB_OKCANCEL);
        if (msgboxID == IDOK) {
            callback->Continue(true, CefString());
        }
        else {
            callback->Continue(false, CefString());
        }
        return true;
    }
    ```
    - 在这段代码中，我们完成了如下三项工作：
        - 通过browser对象获取当前窗口的窗口句柄。接下来我们显示模态对话框时要用到这个窗口句柄。
        - 调用了一个 Windows API ：MessageBox来创建对话框。MessageBox 的第一个参数就是当前窗口的窗口句柄，这样做可以使我们弹出的窗口显示为当前窗口的模态窗口。第二个参数是对话框显示的内容，第三个参数是对话框的标题，第四个参数中 MB_ICONEXCLAMATION 代表对话框左侧显示一个警示图标（可选的图标还有很多，比如错误图标 MB_ICONERROR、询问图标 MB_ICONQUESTION 等），MB_OKCANCEL 代表对话框右下角显示确定和取消按钮。
        - 用户点击了确定按钮后，传一个真值和一个空字符串给 CEF 框架，允许关闭窗口。如果用户点击了取消按钮，那么传一个假值和一个空字符串给 CEF 框架，阻止窗口关闭。
    - 在为 MessageBox 方法提供中文字符串输入参数时，我们为这些字符串添加了L前缀，这个前缀可以将 ANSI 字符串转换成 unicode 的字符串，unicode 每个字符占用两个字节。可以用来表示中文。字符串中\n代表换行。
    - 当一个窗口弹出了一个模态对话框后，用户就不能再与这个窗口进行交互了，必须先完成模态对话框内的操作，关闭模态对话框之后，才能再与这个窗口进行交互。 
    - 上面的代码中，MessageBox 函数执行完成后，模态对话框就被弹出来了，此时代码将阻塞在这里，直到用户点击了确定或取消按钮，模态对话框被关闭后才会继续执行。MessageBox 方法的返回值就是用户最终点击的那个按钮的 ID 值。
    - 模态对话框弹出后，不但 C++ 代码不会再继续执行，页面中的 JavaScript 脚本也会被阻塞，这个行为是符合预期的，即使我们开发纯前端页面，弹出 JavaScript 对话框后，页面的脚本也会被阻塞。你可以在弹出对话框前用 setInterval 开启一个 JavaScript 定时器，看对话框弹出后，定时器的回调方法是不是还在按时执行。

6. 自定义 alert 与 confirm 对话框
- 同样的，要想自定义alert与confirm对话框（包括下一节要介绍的prompt对话框），就必须知道什么时候弹出这些 JavaScript 对话框。CEF 框架把弹出这些对话框的时机封装到同一个事件中，这个事件就是OnJSDialog，我们直接看一下它的实现代码：
    ```cpp
    //头文件中要加入对应的方法声明
    //bool OnJSDialog(CefRefPtr<CefBrowser> browser, const CefString& origin_url,JSDialogType dialog_type,const CefString& message_text,const CefString& default_prompt_text,CefRefPtr<CefJSDialogCallback> callback,bool& suppress_message) override;
    bool PageHandler::OnJSDialog(CefRefPtr<CefBrowser> browser,
                                 const CefString& origin_url,
                                 JSDialogType dialog_type,
                                 const CefString& message_text,
                                 const CefString& default_prompt_text,
                                 CefRefPtr<CefJSDialogCallback> callback,
                                 bool& suppress_message) {
        suppress_message = false;
        HWND hwnd = browser->GetHost()->GetWindowHandle();
        if (dialog_type == JSDialogType::JSDIALOGTYPE_ALERT) {        
            MessageBox(hwnd, message_text.c_str(), L"系统提示", MB_ICONEXCLAMATION | MB_OK);
            callback->Continue(true, CefString()); 
        }
        else if(dialog_type == JSDialogType::JSDIALOGTYPE_CONFIRM){
            int msgboxID = MessageBox(hwnd, message_text.c_str(), L"系统提示", MB_ICONEXCLAMATION | MB_YESNO);
            callback->Continue(msgboxID == IDYES, CefString()); 
        }
        else if (dialog_type == JSDialogType::JSDIALOGTYPE_PROMPT){
           //这部分逻辑稍后讲解
        }    
        return true;
    }
    ```
    - 先来按顺序介绍一下这个方法的七个参数：
        - browser：浏览器对象。
        - origin_url：弹出对话框的页面的 url 路径。
        - dialog_type：弹出对话框的类型，我们将使用这个参数判断即将弹出的对话框是 alert 对话框还是其他对话框。
        - message_text：弹出对话框所承载的文本信息，这是由 JavaScript 代码提供的。
        - default_prompt_text：这个参数专门为 prompt 对话框服务，显示在 prompt 对话框中输入框的默认值。
        - callback：自定义对话框被用户关闭后，需要通过这个对象的内置方法 Continue 把用户做出的选择通知 CEF 框架。与前文介绍阻止页面关闭的对话框的 callback 参数作用一致。
        - suppress_message：代表着要不要屏蔽默认对话框。如果把这个参数设置为 true，然后整个函数的返回值为 false，那么将不会显示对话框；如果这个参数被设置为 false，整个函数返回 false 时，将使用 CEF 框架默认的对话框；如果这个参数被设置为 false，整个方法返回 true，那么将使用开发者自定义的对话框。
    - 同样的，OnJSDialog 方法运行之初，我们先通过 browser 对象获取到当前窗口的窗口句柄，接下来我们显示模态对话框时要用到这个窗口句柄。
    - 然后通过dialog_type判断当前弹窗的类型。dialog_type是一个JSDialogType的枚举类型，这个枚举类型就只定义了三种弹窗类型。如下代码所示：
        ```cpp
        typedef enum {
            JSDIALOGTYPE_ALERT = 0,     //alert弹窗
            JSDIALOGTYPE_CONFIRM,       //confirm弹窗
            JSDIALOGTYPE_PROMPT,        //prompt弹窗
        } cef_jsdialog_type_t;
        ```
    - 如果弹出的是 alert 弹窗，则通过 MessageBox 显示只有一个“确定”按钮的对话框。如果是 confirm 弹窗，则显示包含“是”和“否”按钮的弹窗（你也可以用MB_OKCANCEL来让对话框显示“确定”与“取消”按钮）。如果是 prompt 弹窗，则显示自定义的 prompt 对话框，这部分内容我们将在下一节讲解。显示这三个对话框时，都使用了当前窗口的句柄，所以这三个弹窗都是模态的。
    - alert 与 confirm 这两个弹窗内显示的信息都是通过 message_text 参数获得的，这个参数的类型是 CefString 类型，它是 CEF 框架内置的字符串类型，我们不能把它直接传递给 MessageBox 方法，所以这里调用了 CefString 的 c_str 方法，得到它的原始字符数组后就可以传递给 MessageBox 方法了。
    - 由于 alert 与 confirm 这两个弹窗也是模态对话框，所以 MessageBox 方法把弹窗弹出后，代码将不再继续往下执行，直到用户做出选择后，callback 的 Continue 方法才被执行，此时我们才把用户的选择传递给 CEF 框架。
    - 在 confirm 对话框弹出后，当用户点击“是”按钮时，callback 的 Continue 方法第一个参数被设置为 true；如果用户点击的是“否”按钮，callback 的 Continue 方法第一个参数被设置为 false。callback 的 Continue 方法执行完成后，JavaScript 代码的 confirm 方法将得到返回值。
- 接下来在我们的 JavaScript 文件中加入如下代码以验证：
    ```js
    let flag = confirm("这是一个confirm对话框 \n做出选择之后我会把你的选择alert出来");
    alert("用户选择了：" + (flag?"确认":"取消"));
    ```

7. prompt 对话框不能再通过 MessageBox 定义，因为 prompt 对话框是一个请求用户输入的对话框，这个对话框内包含一个文本输入框，而 MessageBox 创建的任何样式的对话框都没有文本输入框，所以只能自己设计并实现这么一个对话框。

8. 自定义对话框
- 要想实现一个完全自定义的对话框，首先需要在 Visual Studio 中添加一个对话框资源（Dialog 类型的资源），并为这个对话框添加相应的元素。
- 我们使用一个Static Text控件作为提示信息的显示控件， JavaScript 调用 prompt 方法时传入的第一个参数对应的字符串将显示在这个控件上。使用一个Edit Control控件作为文本输入框控件，JavaScript 调用 prompt 方法时传入的第二个参数对应的字符串将显示在这个控件上，它是作为输入框的默认值呈现的，如果用户没有为 prompt 方法提供第二个参数，那么这个文本输入框将不显示默认值，对话框的“确认”和“取消”按钮是对话框资源自带的，不需要我们手动添加。
- 资源添加成功之后，项目中会多出两个文件：ceftest.rc 是资源描述文件，resource.h 是资源的头文件。这个资源头文件中应该包含如下代码：
    ```cpp
    //对话框ID
    #define IDD_PROMPT                      101 
    //文本框ID
    #define IDC_INPUT                       1001
    //提示信息显示控件的ID
    #define IDC_LABEL                       1002
    ```
    - 这些代码是对话框及对话框内控件的ID定义，后文我们将通过这些 ID 获取到对话框内的控件元素，并设置它们显示的文本信息。这些 ID 的值是与 ceftest.rc 文件内的描述信息有关联的，不要轻易手动修改他们的值。

9. 使用对话框资源
- 开发者可以通过如下 Windows API 来使用这个对话框资源：
    ```cpp
    DialogBox(hinst, MAKEINTRESOURCE(IDD_PROMPT), hwnd, (DLGPROC)PromptProc)
    ```
    - 这个 API 与上一节中我们讲过的 MessageBox API 类似，但 MessageBox 方法无法使用自定义的对话框资源，DialogBox 方法却可以。
    - DialogBox 方法的四个参数的意义如下：
        - 第一个参数 hinst 是应用程序的实例句柄，也叫模块句柄，整个应用的入口函数的第一个参数就是它，当然我们也可以在运行过程中通过 GetModuleHandle 这个 Windows API 获得这个句柄。
        - 第二个参数是一个资源指针，MAKEINTRESOURCE 资源转化宏可以把对话框 ID 转化成对话框资源对应的内存指针。此处使用的就是这个指针。
        - hwnd 是当前窗口的句柄。
        - PromptProc 是对话框的消息循环处理函数的函数指针，类型为 DLGPROC，这个函数用于处理对话框创建成功事件、对话框关闭事件等，接下来我们就介绍这个消息循环处理函数的实现逻辑。
    - 操作系统会把对话框运行过程中产生的事件消息封装到指针中传递给对话框消息处理函数，供开发者在指定的事件发生时完成相应的业务逻辑。实际上不单单是对话框，正常的 Windows 窗口也有类似的消息循环处理函数。
- 这个消息循环处理函数的实现逻辑代码如下：
    ```cpp
    // 这段代码被放置在 PageHandler.cpp 文件中
    // 要使用自定义对话框资源必须提前 #include "resource.h"
    namespace{
        LPWSTR infoValue;
        LPWSTR userInputValue;
        LPWSTR defaultValue;
        BOOL CALLBACK PromptProc(HWND hwndDlg, UINT message, WPARAM wParam, LPARAM lParam) {
            BOOL result = FALSE;
            switch (message) {
                case WM_INITDIALOG:{ //对话框创建成功的消息
                    SetDlgItemText(hwndDlg, IDC_LABEL, infoValue); //设置提示信息控件的文本信息
                    SetDlgItemText(hwndDlg, IDC_INPUT, defaultValue); //设置文本输入框的默认文本信息
                    HWND hwndInput = GetDlgItem(hwndDlg, IDC_INPUT); //得到文本输入框的窗口句柄
                    SetFocus(hwndInput); //把鼠标光标聚焦在文本输入框内
                    SendMessage(hwndInput, EM_SETSEL, 0, -1); //发送选中文本输入框内的所有内容的消息
                    break;
                }
                case WM_COMMAND: {
                    switch (LOWORD(wParam)) {
                        case IDOK: { //用户点击确定按钮的消息
                            HWND hwndInput = GetDlgItem(hwndDlg, IDC_INPUT);
                            int outLength = GetWindowTextLength(hwndInput) + 1;
                            userInputValue = new TCHAR[outLength];
                            GetDlgItemText(hwndDlg, IDC_INPUT, userInputValue, outLength);
                            EndDialog(hwndDlg, wParam);
                            result = TRUE;
                            break;
                        }
                        case IDCANCEL: { //用户点击取消按钮的消息
                            EndDialog(hwndDlg, wParam);
                            result = FALSE;
                            break;
                        }                        
                    }
                    break;
                }
            }
            return result;
        }
    }
    ```
    - 这段代码被放置在 PageHandler 类的实现文件中，并且被包含在一个匿名的名称空间 namespace 内，在这个匿名名称空间内定义的变量和方法只能被 PageHandler 类所访问，其他类无法访问。
    - C++ 编译器在编译匿名名称空间时，会为这个名称空间生成一个唯一的名字，并附加上对应的 using 指令，以达到只允许在某个文件内访问这个名称空间内部变量的效果。
    - 在这段代码中我们定义了三个中间变量以及一个对话框消息处理函数，他们分别是：
        - infoValue 用于存放对话框的提示信息，它的值将被显示在 Static Text 控件上。
        - userInputValue 用于存放用户在输入框输入的信息。
        - defaultValue 用于存放 JavaScript 为 prompt 对话框提供的默认值，在用户尚未输入任何信息时，这个默认值将显示在输入框内。
        - PromptProc 是对话框的消息处理函数。
    - 对话框创建成功后，操作系统将向对话框的消息处理函数发送 WM_INITDIALOG 消息，我们在这个消息处理逻辑中完成了如下工作：
        - 设置提示信息控件的文本信息。
        - 设置文本输入框的默认文本信息。
        - 得到文本输入框的窗口句柄（没错，文本输入框也是一个窗口，也有窗口句柄）。
        - 把光标聚焦在文本输入框内。
        - 选中文本输入框内的所有内容，这里选中的就是文本输入框内的默认文本信息，这样做主要是为了方便用户输入自己的信息，提升用户体验。
    - 在这段代码中 SetDlgItemText 和 GetDlgItem 都是 Windows API 。它们的第一个参数都是 hwndDlg，hwndDlg 是对话框的窗口句柄，它们的第二个参数是目标控件的 ID（resource.h 资源头文件中定义的 ID）， SetDlgItemText 的第三个参数是具体要设置的文本信息。
    - 当用户点击“确定”按钮或“取消”按钮后，操作系统将向对话框消息处理函数发送 WM_COMMAND 消息，收到这个消息后我们通过 LOWORD 宏取出 wParam 参数的有意义的部分，这个有意义的部分就是用户点击了哪个按钮的 ID 值。
    - 如果用户点击的是确定按钮，那么我们获取到文本输入框内的文本信息，并把它赋值给前面定义的 userInputValue 中间变量，然后关闭对话框。如果用户点击的是取消按钮，直接关闭对话框即可。对话框关闭后，调用者将收到DialogBox 方法的返回值。这个返回值是一个 BOOL 类型的返回值。用户点击了“确定”按钮时，这个值是 TRUE，用户点击了“取消”按钮时，这个值时 FALSE。
    - 在获取输入框的文本信息时，首先通过 GetDlgItem 获取到文本框控件的窗口句柄，然后通过 GetWindowTextLength 获取文本框内文本信息的长度（长度值需要加1，因为还要为字符串的结尾符号/0留下空间），然后为 userInputValue 变量开辟指定长度的 TCHAR 类型的数组空间。最后通过 GetDlgItemText 把文本框内的数据拷贝到这个空间中。
    - userInputValue 变量的类型是 LPWSTR ，这是 Windoiws API 定义的一个类型，实际上它就是 TCHAR 类型的数组指针。所以我们这里直接把 TCHAR 类型的数组指针赋值给 userInputValue 变量并没有任何问题。

10. 自定义 prompt 对话框
- 接下来我们看一下 prompt 对话框的处理逻辑：
    ```cpp
    // 之前的逻辑在上一节中已经详细讲述了
    else if (dialog_type == JSDialogType::JSDIALOGTYPE_PROMPT){
        HINSTANCE hinst = GetModuleHandle(NULL);
        defaultValue = (LPWSTR)default_prompt_text.c_str();
        infoValue = (LPWSTR)message_text.c_str();
        BOOL result = DialogBox(hinst, MAKEINTRESOURCE(IDD_PROMPT), hwnd, (DLGPROC)PromptProc);
        if (result == 1) {            
            callback->Continue(result, CefString(userInputValue));
            delete []userInputValue;
        }
        else{
            callback->Continue(result, CefString());
        }
    }
    ```
    - 在这段代码中，我们完成了如下几项工作：
        - 通过 GetModuleHandle 这个 Windows API 获取应用程序的实例句柄。
        - 通过 OnJSDialog 方法的 default_prompt_text 参数获取 JavaScript 代码为 prompt 弹窗提供的默认值，并把它赋值给 defaultValue 中间变量。
        - 通过 OnJSDialog 方法的 message_text 参数获取 JavaScript 代码为 prompt 弹窗提供的提示信息字符串，并把它赋值给 infoValue 中间变量。
        - 使用 Windows API DialogBox 创建自定义对话框，注意这也是一个模态窗口。
        - 当对话框关闭后，如果用户点击了“确定”按钮，我们通过 callback 的 Continue 方法把用户输入的信息返回给了 JavaScript 代码。 同时释放了 userInputValue 变量所指向的字符数组空间。
        - 当对话框关闭后，如果用户点击了“取消”按钮，我们通过 callback 的 Continue 方法把一个空字符串返回给了 JavaScript 代码，由于用户点击的是取消按钮，并没有为 userInputValue 变量创建新的字符数组空间，所以就没必要释放 userInputValue 变量所指向的字符数组空间了。
- 接下来再在我们的 JavaScript 文件中增加一段验证代码，运行程序并查看效果。
    ```js
    let userInput = prompt("请您输入一段文字", "这是一段文字的默认值");
    alert("用户输入了：" + userInput);
    ```

11. 自定义右键菜单
- 要想打开窗口页面的开发者调试工具 DevTools ，必须先为页面创建一个“调试页面”的右键菜单项，想要创建这个菜单项，就必须知道 CEF 框架在何时创建右键菜单。
- 与上两节介绍的 CefJSDialogHandler 相同，CEF 框架也为我们提供了一个与右键菜单有关的处理类：CefContextMenuHandler，这个类型描述了与右键菜单相关的行为。同样与上两节介绍的内容相同，我们让 PageHandler 类继承这个 CefContextMenuHandler 类型，为 PageHandler 类的头文件添加如下代码：
    ```cpp
    //PageHandler类的集成关系目前为：
    //class PageHandler:public CefClient, public CefLifeSpanHandler, public CefJSDialogHandler, public CefContextMenuHandler
    CefRefPtr<CefContextMenuHandler> GetContextMenuHandler() override { return this; }
    virtual void OnBeforeContextMenu(CefRefPtr<CefBrowser> browser, CefRefPtr<CefFrame> frame, CefRefPtr<CefContextMenuParams> params, CefRefPtr<CefMenuModel> model) override;
    virtual bool OnContextMenuCommand(CefRefPtr<CefBrowser> browser, CefRefPtr<CefFrame> frame, CefRefPtr<CefContextMenuParams> params, int command_id, EventFlags event_flags) override;
    ```
    - 在这段代码中，GetContextMenuHandler 方法是 CefClient 基类定义的，它负责返回右键菜单的处理类对象，由于我们的 PageHandler 类就继承自 CefContextMenuHandler ，所以这里就返回了 PageHandler 自己的对象指针。
    - 当用户在页面上点击鼠标右键时， CEF 框架将执行 OnBeforeContextMenu 方法，这个方法的第一个参数是 browser 对象，第二个参数是 frame 对象。第三个参数是一个 CefContextMenuParams 类型的对象，我们可以通过它获得用户当前鼠标所在的位置、鼠标所在位置的页面元素的类型、当前页面的路径等信息。第四个参数是 CefMenuModel 类型的菜单对象，这个对象里包含 CEF 框架默认的菜单项，我们可以通过这个对象删除掉 CEF 的默认菜单项并添加自己的菜单项。
    - 当用户点击了某个菜单项时（无论用户点击的是 CEF 默认菜单项还是开发者自定义的菜单项）， CEF 框架将执行 OnContextMenuCommand 方法。这个方法的前三个参数与 OnBeforeContextMenu 方法的前三个参数一致。第四个参数是菜单ID：command_id，开发者可以通过这个参数来获取用户点击了具体的哪个菜单项。第五个参数是一个事件标记：event_flags，开发者可以通过它获取用户按下鼠标右键时，是不是同时按下了 Ctrl 键或者 Alt 键等信息。
- OnBeforeContextMenu 方法的实现代码：
    ```cpp
    void PageHandler::OnBeforeContextMenu(CefRefPtr<CefBrowser> browser, CefRefPtr<CefFrame> frame, CefRefPtr<CefContextMenuParams> params, CefRefPtr<CefMenuModel> model)
    {
        model->Clear();
        model->AddItem(MENU_ID_USER_FIRST, L"打开开发者调试工具");
        CefRefPtr<CefMenuModel> subModel = model->AddSubMenu(MENU_ID_USER_FIRST + 1, L"这是一个包含子菜单的测试菜单");
        subModel->AddItem(MENU_ID_USER_FIRST + 2, L"这是子菜单1");
        subModel->AddItem(MENU_ID_USER_FIRST + 3, L"这是子菜单2");
        model->AddSeparator();
        model->AddCheckItem(MENU_ID_USER_FIRST + 4, L"这是一个包含复选框的菜单");
        model->SetChecked(MENU_ID_USER_FIRST + 4,true);
        model->AddRadioItem(MENU_ID_USER_FIRST + 5, L"这是一个包含复选框的菜单",888);
        model->AddRadioItem(MENU_ID_USER_FIRST + 6, L"这是一个包含单选框的菜单", 888);
        model->SetChecked(MENU_ID_USER_FIRST + 6, true);
    }
    ```
    - 在这个方法中我们做了以下几项工作：
        - 使用 CEF 提供的菜单对象 model 的 Clear 方法清除掉默认的菜单项。
        - 菜单对象的 AddItem 方法可以为菜单添加一个新的普通菜单项，这个方法的第一个参数是菜单的 CommandID，接下来选中菜单项以及点击菜单项时，都要用到这个 CommandID 。它应该被设置为 MENU_ID_USER_FIRST（26500） 和 MENU_ID_USER_LAST（28500） 之间的值，这些值不会与 CEF 或 Chromium 默认的菜单项重复。
        - 菜单对象的 AddSubMenu 方法可以添加一个具备子菜单的菜单项，这个方法也返回一个菜单对象，开发者可以使用这个返回的菜单对象添加子菜单。
        - 菜单对象的 AddSeparator 方法可以为菜单添加一个分割符。
        - 菜单对象的 AddCheckItem 方法和 AddRadioItem 方法可以为菜单添加复选菜单项和单选菜单项。
        - 菜单对象的 SetChecked 方法可以选中某个复选菜单项或单选菜单项。
- 注意：我们设置的菜单是可以浮出窗口之外的，这个特性是使用HTML元素模拟菜单无法实现的。
- 开发者仍然可以使用 document.oncontextmenu = () => { return false } 这样的 JavaScript 代码来屏蔽掉我们自定义的菜单项。

12. 处理菜单点击事件
- 当用户点击某个菜单项时 OnContextMenuCommand 方法会被 CEF 框架调用。我们可以在这个方法中处理菜单点击事件，先来看一下这个方法的实现代码：
    ```cpp
    bool PageHandler::OnContextMenuCommand(CefRefPtr<CefBrowser> browser, CefRefPtr<CefFrame> frame, CefRefPtr<CefContextMenuParams> params, int command_id, EventFlags event_flags)
    {
        switch (command_id)
        {
            case MENU_ID_USER_FIRST: {
                //这部分内容稍后讲解
            }
            default: {
                std::wstring msg = L"你点击的标签ID：" + std::to_wstring(command_id);
                MessageBox(NULL, (LPWSTR)msg.c_str(), L"系统提示", MB_ICONEXCLAMATION | MB_OKCANCEL);
                break;
            }
        }
        return true;
    }
    ```
    - 如果这个方法返回 false 说明我们要把菜单点击事件的处理权限交给 CEF 框架，如果返回 true ，说明我们已经处理了菜单点击事件，不需要 CEF 框架再帮忙了。（如果用户点击的是开发者自定义的菜单项，那么这类事件 CEF 框架也处理不了），由于我们屏蔽了所有 CEF 的内置菜单项，所以这里就直接返回 true 了。
    - 这个方法只做了两件事：
        - 用户点击第一个菜单项的时候弹出 DevTools 窗口，这部分内容稍后讲解。
        - 用户点击其他菜单项的时候，我们弹出了一个对话框，并让这个对话框显示菜单项的 CommandID 。
- 我们使用宽字节字符串在弹窗内显示信息，实际上在开发 CEF 桌面应用过程中很多地方都会使用宽字节字符串处理信息，比如后面的 C++ 与 JavaScript 交互的信息，如果不使用宽字节字符串，界面就会显示乱码。C++ 的标准库<string>， CEF 的字符串类型 CefString 都有对宽字符的支持。这里我们用到的 std:wstring 就是标准库对宽字节字符串的支持类型。
- 虽然我们创建了一个具备选中能力的菜单项，但我们并没有完成相关的选中和取消选中的功能，这并不困难，开发者只要在相应的菜单项点击事件中，记下菜单项的 CommandID 和选中状态，在下一次用户点击右键并创建菜单项时，使用相应的选中状态创建菜单项就可以了。

13. 打开开发者调试窗口
- 设置菜单项时，我们把打开开发者调试工具的菜单项的 CommandID 设置为 MENU_ID_USER_FIRST 了，所以这里我们就把打开 DevTools 窗口的处理逻辑放置在 MENU_ID_USER_FIRST 这个 Command 消息的 switch case 中，代码如下：
    ```cpp
    case MENU_ID_USER_FIRST: {
        CefBrowserSettings browserSettings;
        CefWindowInfo windowInfo;
        CefPoint mousePoint(params->GetXCoord(), params->GetYCoord());
        browser->GetHost()->ShowDevTools(windowInfo, this, browserSettings, mousePoint);
        break;
    }
    ```
    - 在这段代码中，我们首先定义了browserSettings和windowInfo两个对象，这两个对象都是配置对象，我们都使用它们的默认值，并未对它们做多余的设置。
    - 接着通过 OnContextMenuCommand 方法的第三个参数params获取到了当前用户鼠标所在的位置的坐标。并使用这个坐标信息初始化了一个 CefPoint 类型的对象。
    - 最后使用 BrowserHost 对象的 ShowDevTools 方法显示 DevTools 窗口，这个方法的第一个和第三个参数就是我们前面初始化的 browserSettings 和 windowInfo 两个对象，第二个参数是一个 CefClient 类型的对象，实际上它就是我们的 PageHandler 对象，所以这里我们把第二个参数设置为 this 。最后一个参数是鼠标位置坐标对象，它的作用是使 DevTools 窗口的 Elements 面板定位到该坐标所在的元素上。
- 如果当前窗口已经打开了一个 DevTools 窗口，再次点击页面的“打开开发者调试工具”菜单项，是不会打开一个新的 DevTools 窗口的，原来的 DevTools 窗口将会被聚焦，并且 Elements 面板会定位到鼠标所指向的新的元素上。
- 在前面介绍了 OnPopupBrowserViewCreated 方法，如果此时你在这个方法中设置一个断点，你会发现点击页面的“打开开发者调试工具”菜单项，这个断点会命中。而且 OnPopupBrowserViewCreated 方法的最后一个参数 isDevtools 的值为 true 。说明 DevTools 窗口也是受ViewDelegate 管控的。
- 实际上 DevTools 窗口也是被 WindowDelegate 管控的。

14. 自定义的标题栏效果都是通过屏蔽掉系统默认标题栏，然后在一个无标题栏的窗口内实现一个自定义的标题栏实现的。

15. 无边框窗口
- 要想实现自定义的窗口标题栏，首先需要把系统默认的窗口标题栏屏蔽掉，这个工作需要在 WindowDelegate 类中实现，WindowDelegate 类继承自 CefWindowDelegate 类，用于控制一个窗口的行为。目前为止我们应用中的所有窗口都受它的控制，所以只要在这个类里屏蔽掉系统默认的标题栏，那么应用程序所有窗口的标题栏就都被屏蔽掉了。
- 然而我们并不希望开发者工具 DevTools 窗口的标题栏也被屏蔽（没错，这个特殊的窗口也受 WindowDelegate 类的控制）。所以我们要在 WindowDelegate 类中记录当前窗口的类型，为此我们决定为 WindowDelegate 类增加一个 bool 类型的变量isDevTool来记录窗口的类型。
- 另外，只要让 WindowDelegate 类重写基类 CefWindowDelegate 的 IsFrameless 方法就可以屏蔽掉系统默认的标题栏，如下就是修改过的 WindowDelegate 类的头文件：
    ```cpp
    #pragma once
    #include "include/views/cef_window.h"
    #include "include/views/cef_browser_view.h"
    class WindowDelegate : public CefWindowDelegate
    {
    public:
        //构造函数增加了一个参数，用于初始化isDevTool
        explicit WindowDelegate(CefRefPtr<CefBrowserView> browser_view,bool dev_tool) : browserView(browser_view),isDevTool(dev_tool) {};
        // 无边框窗口，屏蔽掉系统默认的标题栏
        bool IsFrameless(CefRefPtr<CefWindow> window) override;
        void OnWindowCreated(CefRefPtr<CefWindow> window) override;
        void OnWindowDestroyed(CefRefPtr<CefWindow> window) override;
        bool CanClose(CefRefPtr<CefWindow> window) override;
        CefRect GetInitialBounds(CefRefPtr<CefWindow> window) override;
        WindowDelegate(const WindowDelegate&) = delete;
        WindowDelegate& operator=(const WindowDelegate&) = delete;
    private:
        // 当前窗口是否为开发者工具窗口
        bool isDevTool;
        CefRefPtr<CefBrowserView> browserView;
        IMPLEMENT_REFCOUNTING(WindowDelegate);
    };
    ```
    - 前面已经讲解过这个类型的大部分代码了，只有 3 处内容是本节新增加的。
        - 声明重写基类的 IsFrameless 方法，这个方法可以使当前窗口变成无边框窗口，无边框窗口是没有系统默认标题栏的。
        - 创建了一个私有变量isDevTool，用于记录当前窗口是不是开发者工具窗口。
        - 我们为构造函数新增加了一个参数dev_tool，用于初始化 isDevTool 私有变量。
- 由于改动了 WindowDelegate 类的构造函数，所以有两处地方需要跟着修改：
    - 第一处是App类的OnContextInitialized方法，我们在这个方法内创建了应用程序的第一个窗口，显然这个窗口不是开发者工具窗口，所以这里修改后的代码为：
        ```cpp
        CefWindow::CreateTopLevelWindow(new WindowDelegate(browserView,false));
        ```
    - 第二处为ViewDelegate类的OnPopupBrowserViewCreated方法，这个方法的最后一个参数就标记着即将创建的窗口是不是开发者工具窗口，所以我们在这里创建 WindowDelegate 对象时就直接使用了这个参数，代码如下:
        ```cpp
        bool ViewDelegate::OnPopupBrowserViewCreated(CefRefPtr<CefBrowserView> browserView, CefRefPtr<CefBrowserView> popupBrowserView, bool isDevtools)
        {
            CefWindow::CreateTopLevelWindow(new WindowDelegate(popupBrowserView,isDevtools));
            return true;
        }
        ```
- 有了这些工作后，在 CEF 框架执行IsFrameless方法时，我们就知道当前窗口是不是开发者工具窗口了，所以 IsFrameless 方法的实现代码为：
    ```cpp
    bool WindowDelegate::IsFrameless(CefRefPtr<CefWindow> window) {
        if (isDevTool) {
            return false;
        }
        return true;
    }
    ```
    - 这段代码的意义就是说，如果窗口是开发者工具窗口，那么就使用系统标题栏，如果不是，那么就屏蔽系统标题栏。
- 虽然 CEF 框架给这样的窗口叫无边框窗口，但这样称呼是不科学的，因为它还是有边框的，而且可以通过拖动边框改变窗口的大小，也就是说这个窗口只是屏蔽了系统标题栏，边框还是在的，这也正是我们期望的样子。如果你希望制作一个真正无边框的窗口（不能通过拖拽边框改变窗口的大小），那么你应该尝试重写 CefWindowDelegate 类的 CanResize 方法，一旦这个方法返回false，那么你的窗口的边框就彻底没有了，非但如此，甚至连窗口的阴影效果也没有了。此时你可能需要考虑自己实现窗口的阴影效果。

16. 自定义标题栏
- 第一个窗口的标题栏是我们用 HTML 、 CSS 实现的。其中 HTML 代码如下所示：
    ```html
    <html>
    <head>
        <title>第一个本地页面</title>
        <meta charset="utf-8" />
        <link rel="stylesheet" type="text/css" href="icon/iconfont.css" />
        <style>
            /*CSS代码放在此处，稍后会介绍*/
        </style>
    </head>
      <body>
        <div class="titleBar">
            <div class="titleContent">这是我的第一个窗口</div>
            <div class="toolBox">
                <div id="minimizeBtn"> <i class="icon icon-minimize"></i> </div>
                <div id="maximizeBtn"> <i class="icon icon-maximize"></i> </div>
                <div id="restoreBtn" style="display:none"> <i class="icon icon-restore"></i> </div>
                <div id="closeBtn"> <i class="icon icon-close"></i> </div>
            </div>
        </div>
        <div class="content">
            这是窗口的内容区域<br />
            <a href="my://bread/index.html?a=123" target="_blank">打开一个新窗口</a>
        </div>
      </body>
    </html>
    ```
    - 这段 HTML 代码把页面切分成上下两个部分，上部titleBar区域是窗口的标题栏区域，在这部分区域我们除了设置窗口标题外，还使用字体图标设置了窗口的工具按钮（设置、最小化、最大化、还原（还原按钮默认处于隐藏状态）、关闭）。下部为窗口的内容区域。
- 可以在 https://www.iconfont.cn/ 网站找到并下载的这些字体图标资源，同时通过设置这些资源文件在 VisualStudio 中的属性，让它们复制到输出目录，这样就可以在渲染页面时成功加载这些资源了。
- 这段 HTML 代码的样式代码如下所示：
    ```css
    html, body {
        margin: 0px;
        padding: 0px;
        height: 100%;
        width: 100%;
    }
    body {
        user-select: none;
        font-size: 13px;
        color: #333;
        font-family: -apple-system, 'Microsoft Yahei', Ubuntu, sans-serif;
        display: flex;
        flex-direction: column;
    }
    .titleBar {
        height: 38px;
        background: #ffd6e7;
        line-height: 38px;
        padding-left: 16px;
        display: flex;
    }
    .titleContent {
        flex: 1;
        -webkit-app-region: drag;
    }
    .toolBox {
        width: 180px;
        display:flex;
    }
    .toolBox div{
        flex:1;
        text-align:center;
    }
    .toolBox div:hover {
        background: #ffadd2;
    }
    #closeBtn:hover {
        background: #ff7875;
        color:#fff;
    }
    .toolBox div i {
        font-size: 12px;
    }
    .content {
        flex: 1;
        text-align: center;
        padding-top: 120px;
    }
    ```
    - 我们为 body 设置了user-select: none样式，禁止用户选择页面上的文本节点，这也符合大部分桌面应用的行为表现，如果某个区域需要允许用户选中文本节点，那么再为这个区域设置 user-select: text 样式即可。
    - 我们为 titleContent 元素设置了 -webkit-app-region: drag 样式，标志着我们希望用户在这个区域拖拽鼠标时可以移动窗口。假设你的 titleContent 元素内还有某个子元素是不希望具备这个能力的，那么你可以使用-webkit-app-region: no-drag 样式来修饰这个子元素，以屏蔽掉它继承来的这个能力。
    - 目前还存在一个大问题，就是无法通过拖拽标题栏左侧区域（ titleContent 所在区域）来移动窗口，这是因为虽然我们设置了-webkit-app-region样式，但还没有在 C++ 代码中注册可拖拽区域导致的。

17. 可拖拽区域
- 有一个专门的类型，CefDragHandler 负责注册在页面中定义的 -webkit-app-region 区域。接下来我们再为 PageHandler 类添加一个基类 CefDragHandler ，同时为 PageHandler 类的头文件增加如下代码：
    ```cpp
    // PageHandler 类的继承关系如下：
    //class PageHandler : public CefClient, public CefLifeSpanHandler, public CefJSDialogHandler, public CefContextMenuHandler, public CefDragHandler
    CefRefPtr<CefDragHandler> GetDragHandler() override { return this; }
    virtual void OnDraggableRegionsChanged(CefRefPtr<CefBrowser> browser, CefRefPtr<CefFrame> frame, const std::vector<CefDraggableRegion>& regions) override;
    ```
- CefClient 基类描述的 GetDragHandler 方法会把 PageHandler 类的实例返回给 CEF 框架。 CefDragHandler 类描述的 OnDraggableRegionsChanged 方法负责注册页面中 -webkit-app-region 样式标记的区域。当页面渲染成功后，CEF 框架会主动调用这个方法。这个方法的实现代码如下：
    ```cpp
    void PageHandler::OnDraggableRegionsChanged(CefRefPtr<CefBrowser> browser, CefRefPtr<CefFrame> frame, const std::vector<CefDraggableRegion>& regions)
    {
        CefRefPtr<CefBrowserView> browser_view = CefBrowserView::GetForBrowser(browser);
        if (browser_view)
        {
            CefRefPtr<CefWindow> window = browser_view->GetWindow();
            if (window) window->SetDraggableRegions(regions);
        }
    }
    ```
    - 在这段代码中，我们先使用 CefBrowserView 的静态方法 GetForBrowser 获取到 browser_view 对象，然后再通过 browser_view 对象的 GetWindow 方法获取到窗口对象 window，然后再通过 window 对象的 SetDraggableRegions 方法为这个窗口添加可拖拽区域。这里可拖拽区域可能是由多个不连续的区域构成的，这些区域被放置在一个 C++ 标准容器 vector 中。
    - 现在再运行程序，我们就可以通过拖拽标题栏左侧区域来移动窗口了。
- 如果页面中有 JavaScript 脚本为某个元素动态添加了 -webkit-app-region 样式， CEF 框架并不会再次调用 OnDraggableRegionsChanged 方法，所以动态设置元素的 -webkit-app-region 样式并不能扩展可拖拽区域。 这应该是 Chromium 的一个 BUG，或许在未来的版本中得以修复。如果你想动态的添加可拖拽区域，需要使用后续章节讲解的知识才行。

---

###### 四、进程通信

1. 为 JavaScript 全局对象注册方法
- 要想实现窗口标题栏内的按钮的功能，就必须在用户点击这些按钮的时候让 JavaScript 代码访问 C++ 代码，为了达到这个目的，我们用 C++ 代码为 JavaScript 的全局对象（也就是window对象）注册了一个方法，当 JavaScript 代码调用这个方法时，实际上执行的是 C++ 代码，这就是我们让 JavaScript 调用 C++ 原生方法的实现思路。
- 前面我们分离出了浏览器进程处理类、渲染进程处理类、工具进程处理类等不同进程的处理类，现在要为 JavaScript 全局对象注册方法就是在渲染进程的处理类（Renderer）中完成的。
- 首先在 Renderer 类的头文件中加入如下代码：
    ```cpp
    // 这个成员方法是public类型的
    void OnContextCreated(CefRefPtr<CefBrowser> browser, CefRefPtr<CefFrame> frame, CefRefPtr<CefV8Context> context) override;
    // 这个成员变量是private类型的
    CefRefPtr<V8Handler> v8Handler;
    ```
- OnContextCreated 方法是在 Renderer 类的基类CefRenderProcessHandler中定义的，在一个页面的 JavaScript 执行上下文创建完成的时候， CEF 框架会调用这个方法。如果页面还包括iframe子页面，那么子页面的 JavaScript 执行上下文创建完成时也会调用这个方法。也就是说我们在这个方法里为 JavaScript 注册的全局对象，存在于父页面中，也存在于子页中。
- v8Handler 成员变量是我们自定义的一个类型对象，它可以接收 JavaScript 代码的调用，并执行其内部定义的 C++ 方法。
- OnContextCreated 方法的实现代码如下所示：
    ```cpp
    void Renderer::OnContextCreated(CefRefPtr<CefBrowser> browser, CefRefPtr<CefFrame> frame, CefRefPtr<CefV8Context> context) {
        CefRefPtr<CefV8Value> globalObject = context->GetGlobal();
        v8Handler = new V8Handler();
        CefRefPtr<CefV8Value> nativeCall = CefV8Value::CreateFunction("nativeCall", v8Handler);
        globalObject->SetValue("nativeCall", nativeCall, V8_PROPERTY_ATTRIBUTE_READONLY);     
    }
    ```
    - 通过 context 参数的 GetGlobal 方法获取 JavaScript 的全局对象，这个 JavaScript 的全局对象就是我们在页面中写 JavaScript 代码时常用的 window 对象， context 参数就是 JavaScript 的执行上下文对象。
    - 通过 CefV8Value 的 CreateFunction 方法创建一个 JavaScript 的原生方法。这个原生方法的处理逻辑被封装在一个名为 V8Handler 的自定义类中，我们稍后再讲这个类的实现细节。
    - 为 JavaScript 的全局对象附加一个只读的方法属性，这个属性名为 nativeCall，这个属性的值就是我们前面创建的原生方法，这个工作执行完成之后，JavaScript 代码就可以通过 window.nativeCall() 来调用我们的 C++ 代码了。

2. 使用 V8 处理类处理 JavaScript 方法
- 要想让一个 C++ 类型可以处理 JavaScript 的方法逻辑，那么这个类型必须继承自 CefV8Handler 基类。V8Handler 类就是这么做的，它的头文件代码如下所示：
    ```cpp
    #pragma once
    #include "include/cef_v8.h"
    class V8Handler : public CefV8Handler
    {
    public:
        V8Handler() = default;
        virtual bool Execute(const CefString& name, CefRefPtr<CefV8Value> object, const CefV8ValueList& arguments, CefRefPtr<CefV8Value>& retval, CefString& exception) override;
    private:
        IMPLEMENT_REFCOUNTING(V8Handler);
    };
    ```
    - V8Handler 类在这个头文件中声明重写基类的 Execute 方法，也就是说当 window.nativeCall() 这句 JavaScript 代码执行时，Execute 方法内的逻辑被执行。
    - 这个方法的 5 个参数的含义如下：
        - name 为方法的名字，它的值就是 nativeCall 字符串。
        - object 对象是 JavaScript 调用这个方法时的 this 对象。
        - arguments 对象是 JavaScript 调用这个方法时传递的参数，这是个 CEF 框架定义的一个容器对象。
        - retval 对象是方法执行完成后 C++ 代码返回给 JavaScript 代码的返回值，这是一个输出参数。
        - exception 是方法执行失败时返回的错误信息，JavaScript 会把这个错误信息封装成一个 Error 对象抛出来。
- Execute 这个方法的实现代码如下所示：
    ```cpp
    #include "V8Handler.h"
    bool V8Handler::Execute(const CefString& name, CefRefPtr<CefV8Value> object, const CefV8ValueList& arguments, CefRefPtr<CefV8Value>& retval, CefString& exception)
    {
        auto msgName = arguments[0]->GetStringValue();
        CefRefPtr<CefProcessMessage> msg = CefProcessMessage::Create(msgName);
        CefRefPtr<CefV8Context> context = CefV8Context::GetCurrentContext();
        context.get()->GetFrame()->SendProcessMessage(PID_BROWSER, msg);
        return true;
    };
    ```
    - 从参数容器中获取第一个参数，这个参数是一个字符串类型的数据，形如：window_minimize，这是我们自己定义的一种格式，window 代表着操作的类型，minimize 代表着操作的名称。我们后文还会介绍为什么要做这样的定义。
    - 通过 CefProcessMessage 类型的静态方法 Create 创建一个进程间消息，这个消息的名称就是我们前面获取到的第一个字符串参数。这个消息将会被发送给浏览器进程（要时刻记得现在这段逻辑是在渲染进程中执行的）。
    - 接着通过 JavaScript 的上下文对象 context 获取到当前的 Frame 对象，然后通过 Frame 对象的 SendProcessMessage 方法把前面创建的进程间消息发送给浏览器进程，由主进程执行具体的操作。
    - 最后方法执行完成，返回 true，代表着我们已经成功的处理了这个 JavaScript 方法 nativeCall 。
- nativeCall 是一个异步方法，方法执行完成后，实际的任务还没执行完。
- 完成这些工作后，运行程序，打开开发者工具，在 console 面板输入window.nativeCall，按下回车，看看是否得到如下结果：
    ```cmd
    > window.nativeCall
    < ƒ nativeCall() { [native code] }
    ```
    - 如果是的话，说明你的 JavaScript 全局对象中已经具备了 nativeCall 方法，并且这个方法是原生代码实现的（native code），调试人员看不到这个方法的实现逻辑。
    - 现在我们可以在页面的 JavaScript 代码中调用这个 nativeCall 方法了。

3. 标题栏按钮的前端逻辑
- 我们在实现标题栏按钮的处理逻辑时，为每个标题栏按钮注册了点击事件，并在点击事件的处理函数中调用了 nativeCall 方法，如下代码所示：
    ```js
    let browserWindow = {
        getMsgName(args) {
            return `window_${args.callee.name}`
        },
        minimize() {
            let msgName = this.getMsgName(arguments);
            window.nativeCall(msgName);
        },
        maximize() {
            let msgName = this.getMsgName(arguments);
            window.nativeCall(msgName);
        },
        close() {
            let msgName = this.getMsgName(arguments);
            window.nativeCall(msgName);
        },
        restore() {
            let msgName = this.getMsgName(arguments);
            window.nativeCall(msgName);
        },
    }
    let minimizeBtn = document.querySelector("#minimizeBtn");
    let maximizeBtn = document.querySelector("#maximizeBtn");
    let restoreBtn = document.querySelector("#restoreBtn");
    let closeBtn = document.querySelector("#closeBtn");
    minimizeBtn.addEventListener("click", () => browserWindow.minimize());
    closeBtn.addEventListener("click", () => browserWindow.close())
    maximizeBtn.addEventListener("click", () => {
        browserWindow.maximize();
        maximizeBtn.setAttribute("style", "display:none");
        restoreBtn.removeAttribute("style");
    })
    restoreBtn.addEventListener("click", () => {
        browserWindow.restore();
        restoreBtn.setAttribute("style", "display:none");
        maximizeBtn.removeAttribute("style");
    })
    ```
    - 我们把 nativeCall 方法封装到一个 JavaScript 对象 browserWindow 中，这样只要在点击事件的处理函数中调用 browserWindow 对象的方法就可以了，不必在每个点击事件中实现很多相似的逻辑。
    - 在一个 JavaScript 方法内（箭头函数除外），可以通过 arguments.callee.name 获取的方法的名称，我们使用这个技巧来组装 nativeCall 方法的参数，在 minimize 方法内调用 this.getMsgName(arguments) ，将得到： window_minimize 字符串。
    - 用户点击完最大化按钮后，窗口状态变为最大化状态，这个时候不能再显示最大化按钮，而应该显示还原按钮。用户点击还原按钮后，窗口状态变为非最大化状态，这个时候不能再显示还原按钮，而应该显示最大化按钮。为了满足这个需求，我们在最大化和还原按钮的点击事件中增加了相应的按钮的显隐处理逻辑。
    - 要想把上述代码写的更精简高效，开发者还可以考虑如下这种实现方式：
        - 不在具体的按钮元素上监听点击事件，而在这些按钮的父元素上监听点击事件，当点击事件触发时，从点击事件的event对象中获取到具体点击了哪个按钮，然后通过按钮的 id ，生成消息的名字，再传递给 nativeCall 方法。
        - 这样做只要监听一个点击事件就可以了，而且我们也不必为 browserWindow 对象实现那几个方法，虽然即精简又高效，但可读性就差了很多，为了更容易理解，我们选择了这种实现方案。
- 完成这些工作后，用户点击标题栏按钮时，就会调用我们的 nativeCall 方法，比如点击的是最小化按钮，最终执行的是:window.nativeCall('window_minimize'); 在这行代码执行时，此前我们定义的 V8Handler 的 Execute 方法就会被执行。

4. 调试渲染进程代码逻辑
- VisualStudio 默认仅调试应用程序主进程的代码，要想调试渲染进程的代码，就必须让 VisualStudio 的调试器附加到渲染进程，才能调试渲染进程的代码。
- 点击 VisualStudio 调试菜单下的附加到进程菜单，打开附加到进程窗口，这个窗口会把系统中的所有进程罗列出来，此时我们通过过滤器找出名为 ceftest.exe 的进程，仍然有 6 个，没关系，我们把处于活动状态的 5 个全部选中，然后再点击附加按钮。这时再在 V8Handler 类的 Execute 方法内下个断点，点击窗口标题栏内的某个按钮，看看断点是不是命中了呢。
- 如果想调试 OnContextCreated 方法内的代码，只要在工程的配置属性中为可执行程序增加一个命令行参数：--renderer-startup-dialog，这样在应用程序创建渲染进程前，CEF框架会弹出一个对话框，阻止渲染进程的创建工作，我们可以在这个时机让 VisualStudio 附加进程。附加好进程并设置好断点之后，点击对话框的确定按钮，关闭对话框，CEF框架会继续执行创建渲染进程的工作，这时当 OnContextCreated 方法执行时，你的断点就会命中了。

5. 我们为窗口标题栏的按钮添加了点击事件的处理逻辑，而且在执行这些点击事件的时候，我们成功的通过 JavaScript 代码调用了在渲染进程中实现的 C++ 代码。但这还远远不够，用户点击这些标题栏按钮的时候，窗口状态仍旧没有变化，这是因为我们虽然把控制消息发送给了浏览器进程，但还没有为浏览器进程撰写接收消息和处理消息的逻辑。

6. 主进程接收并处理渲染进程消息
- 主进程的PageHandler类继承自CefClient基类，CefClient基类定义了一个方法：OnProcessMessageReceived，这个方法就负责接收渲染进程发来的消息。
- 在 PageHandler 类的头文件中声明重写 OnProcessMessageReceived 方法，代码如下：
    ```CPP
    bool OnProcessMessageReceived(CefRefPtr<CefBrowser> browser, CefRefPtr<CefFrame> frame, CefProcessId source_process, CefRefPtr<CefProcessMessage> message) override;
    ```
    - 第三个参数是来源进程的 ID ，它的值就是PID_RENDERER的枚举值，代表着消息从渲染进程发来。
    - 第四个参数就是我们使用 JavaScript 的输入参数构造的进程间消息，实际上这个消息仅通过消息名携带了一个有用的数据（以最小化按钮的点击事件来说，这个消息名字符串为：window_minimize）。
- OnProcessMessageReceived 方法的实现代码：
    ```cpp
    bool PageHandler::OnProcessMessageReceived(CefRefPtr<CefBrowser> browser, CefRefPtr<CefFrame> frame, CefProcessId source_process, CefRefPtr<CefProcessMessage> message)
    {
        CEF_REQUIRE_UI_THREAD();
        std::string messageName = message->GetName();
        std::vector<std::string> arr = split(messageName, '_');    
        if (arr.at(0) == "window"){
            CefRefPtr<CefBrowserView> browserView = CefBrowserView::GetForBrowser(browser);
            CefRefPtr<CefWindow> window = browserView->GetWindow();
            if (arr.at(1) == "minimize") {
                window->Minimize();
            }
            else if (arr.at(1) == "maximize") {
                window->Maximize();
            }
            else if (arr.at(1) == "close") {
                window->Close();
            }
            else if (arr.at(1) == "restore") {
                window->Restore();
            }
        }
        return true;
    }
    ```
    - 获取进程间消息的消息名称，并以 _ 字符拆分这个消息名称字符串，拆分后得到一个子字符串容器对象，拆分方法是我们自定义的一个方法，稍后再介绍。
    - 判断消息名称的第一个子字符串是否为 window ，如果是，说明这个消息是用来控制窗口的。第一个子字符串用于区分消息的分类，目前虽然我们只有一个控制窗口的消息类型，将来我们还会增加其他的消息类型，用于处理文件读写，发送系统通知等。
    - 进入控制窗口的处理逻辑后，我们就通过 CefBrowserView 类的静态方法 GetForBrowser 获取到了页面的 browserView 对象，再通过 browserView 对象的 GetWindow 方法获取到了窗口对象。
    - 接下来用第二个子字符串区分具体的操作名称，比如，点击最小化按钮时，消息的操作名称为 minimize，点击最大化按钮时，消息的操作名称为 maximize 等。此处我们通过这些操作名称，来判断应该对窗口执行什么操作。这些操作都是通过前面获取到的 window 对象完成的。
    - 最后返回 true ，告知 CEF 框架我们已经成功的处理了这个进程间消息。
- 分割字符串的代码如下所示：
    ```cpp
    std::vector<std::string> split(const std::string& s, char delim) {
        std::vector<std::string> elems;
        std::istringstream iss(s);
        std::string item;
        while (std::getline(iss, item, delim)) {
            elems.push_back(item);
        }
        return elems;
    }
    ```
    - 在这个方法中我们把目标字符串转化为istringstream类型，这样我们就可以使用标准库的getline方法从目标字符串中一个一个的获取子字符串了，这些子字符串是以 delim 代表的字符进行分割的，得到子字符串之后，我们就把这些字符串按顺序放入了vector容器内，最后把这个 vector 容器返回给了调用者。

7. 引入事件处理器
- 虽然功能完成了，但有一个隐藏的比较深的 bug：我们的窗口的最大化和还原按钮的显隐状态不会跟着切换。为了解决这个 bug，我们决定引入一个事件处理器。让这个事件处理器来监听窗口最大化状态变更的事件。如下代码所示：
    ```js
    let eventer = {
        //事件容器
        dic: {},
        //监听事件
        on(eventName, callBack) {
            if (!this.dic[eventName]) this.dic[eventName] = [callBack]
            else this.dic[eventName].push(callBack)
        },
        //发射事件
        emit(eventName,...obj) {
            if (!this.dic[eventName]) {
                console.warn(`没有找到该事件的监听函数：${eventName}`)
                return;
            }
            this.dic[eventName].forEach((func) => func(...obj))
        },    
        //取消监听事件
        off(eventName, callBack) {
            if (!this.dic[eventName]) return
            if (!callBack) {
                delete this.dic[eventName]
                return
            }
            let index = this.dic[eventName].findIndex((v) => v == callBack)
            if (index >= 0) this.dic[eventName].splice(index, 1)
            if (this.dic[eventName].length < 1) delete this.dic[eventName];
        }
        //监听一次性事件
        once(eventName, callBack) {
            let callBackWrap = (...obj) => {
                let index = this.dic[eventName].findIndex((v) => v == callBackWrap)
                if (index >= 0) this.dic[eventName].splice(index, 1)
                if (this.dic[eventName].length < 1) delete this.dic[eventName];
                callBack(...obj)
            }
            if (!this.dic[eventName]) this.dic[eventName] = [callBackWrap]
            else this.dic[eventName].push(callBackWrap)
        },
    }
    ```
    - 在这个事件处理器对象中我们实例化了一个事件容器dic，当用户使用这个事件处理器的on方法监听一个事件的时候，我们首先判断以前有没有监听过同名事件，如果有，说明事件容器中存在一个以该事件名命名的属性，这个属性的值是一个数组，数组中存放的是以前监听该事件时注册的回调函数，那此时只要把本次监听该事件的回调函数添加到这个数组的末尾就可以了。如果以前没有监听过同名事件，那么就为事件容器创建一个以该事件名命名的属性，属性的值是一个数组，数组中只有一个值，就是这次监听该事件的回调函数。
    - 当业务代码在页面的某处发射一个事件的时候（也就是调用这个事件处理器的emit方法时），我们会查找事件容器的属性列表，看它是否存在一个与事件名同名的属性，如果存在，就找出这个属性对应的值，也就是注册事件时创建的数组，然后依次执行这个数组中的回调函数。在业务代码发射事件时可能携带了一些事件数据，这些数据也将以参数的形式传递给事件的回调函数。如果事件容器中不存在与事件名同名的属性，说明以前没有监听过这个事件，那么就打印一行警告信息。
    - 有的时候我们监听了一个事件，又希望取消监听这个事件，此时就需要用到事件处理器的off方法，如果取消注册事件时，只给 off 方法传递了事件名称，那么我们就会把事件容器中保存的所有回调方法都删掉。如果不但给 off 方法传递了事件名称，还传递了事件的回调函数，那么我们就会在事件容器中找到与这个事件对应的回调方法，仅仅删除这个回调方法。这样做可以保证其他同名事件不受影响。
    - 有的时候我们需要监听一个一次性事件，也就是说这个事件的回调函数只可能被执行一次，监听这类事件可以调用事件处理器的once方法，在这个方法中，我们对事件的回调函数进行了一次包装，生成了一个包装函数，并把包装函数添加到事件容器中了。当一次性事件发生时，先执行包装函数，包装函数会把自身从事件容器中删除，删除之后再执行原始回调函数。因为事件容器中已经没有了包装函数，所以即使再次发射同名事件，第一次注册的回调函数也不会再被执行了。此处充分的利用了 JavaScript 的闭包特性。

8. 监听窗口最大化和还原事件
- 为了解决这个问题，我们在页面加载成功后就使用事件处理器监听了两个事件：window_maximize和window_unMaximize，我们期望在窗口最大化时触发window_maximize事件，窗口还原时触发window_unMaximize事件。 如下代码所示：
    ```js
    eventer.on("window_maximize", () => {
        maximizeBtn.setAttribute("style", "display:none");
        restoreBtn.removeAttribute("style");
    })
    eventer.on("window_unMaximize", () => {
        restoreBtn.setAttribute("style", "display:none");
        maximizeBtn.removeAttribute("style");
    })
    ```
    - 我们在这两个事件的回调函数中完成了最大化按钮和还原按钮的显示和隐藏的逻辑。
    - 这个工作完成后，原先我们在最大化按钮和还原按钮的点击事件中撰写的显隐代码就可以删掉了。
- 由于窗口状态的变化属于窗口的逻辑，因此我们把监控窗口状态变化的逻辑放置在上一节中讲解的 browserWindow 对象内。如下代码所示。
    ```js
    maximized: false,
    isMaximized() {
        let hSpan = window.outerHeight - screen.availHeight
        let wSpan = window.outerWidth - screen.availWidth
        return Math.abs(hSpan) < 2 && Math.abs(wSpan) < 2
    },
    init() {
        window.addEventListener('resize', () => {
            let curState = this.isMaximized()
            let oldState = this.maximized
            this.maximized = curState
            if (oldState && !curState) eventer.emit(`window_unMaximize`)
            else if (!oldState && curState) eventer.emit(`window_maximize`)
        })
    }
    ```
    - 当页面加载成功后，我们将执行 browserWindow.init(); 方法来监听窗口大小改变事件。
    - 在窗口大小改变的事件中我们监控窗口的状态的改变，如果窗口从非最大化状态变为最大化状态，则通过事件处理器发射 window_maximize 事件，如果窗口状态从最大化状态变为非最大化状态，则通过事件处理器发射 window_unMaximize 事件。
    - 为了监控监控窗口的状态的改变，我们定义了一个方法isMaximized来判断当前窗口是否处于最大化状态，这里使用 window 对象的 outerHeight 和 outerWidth 属性与 screen 对象的 availHeight 和 availWidth 属性比较，如果两者相差都小于2个像素的话，我们就认为窗口已经是最大化状态了（此处2个像素的偏差是必要的，不然在特殊屏幕下会判断失败）。
    - 我们使用 maximized 属性来记录窗口的当前状态，如果通过isMaximized获取到的状态与当前状态不一致，那么就说明窗口不是从最大化状态变成了非最大化状态，就是从非最大化状态变成了最大化状态。 使用判断语句区分这两种情况并触发对应的事件就能让事件接收者收到通知了。也就是说当事件发生时，我们前面注册的事件的回调函数就会被执行。相应的最大化和还原按钮就跟着完成了切换。

9. JavaScript 封装跨进程异步 API
- 获取操作系统版本号的请求是由 JavaScript 代码在前端页面发起的，在发起这个请求之前，我们对原生代码的调用逻辑进行了简单的封装，如下所示：
    ```js
    let native = {
      randomNum(len = 12) {
        return Math.floor(Math.pow(10, len) * Math.random());
      },
      call(msgName, ...params) {
        return new Promise((resolve, reject) => {
          let eventName = `${msgName}_${this.randomNum()}`;
          eventer.once(eventName, (obj) => {
            resolve(obj);
          });
          window.nativeCall(eventName, ...params);
        });
      },
      init() {
        window.nativeCall(`native_registe_callback`, (msgName, ...otherParams) => {
          eventer.emit(msgName, ...otherParams);
        });
      },
    };
    native.init();
    ```
    - 我们把 JavaScript 访问 C++ 代码的逻辑封装到了一个名为 native 的对象中，有了这个对象，我们以后就可以不再直接使用 window.nativeCall 方法访问 C++ 代码了，而是改用 native.call 方法来访问 C++ 代码。
    - native.call 方法返回一个 Promise 对象，当 window.nativeCall 执行完之后，Promise 对象处于 pendding 状态，一旦某个特殊的事件发生后，这个 Promise 对象才成功 resolve 掉。
    - 业务代码调用 native.call 方法时，将传递一个事件名，这个事件名是类似 system_getOSVersion 这样的事件名，在 native.call 方法内部我们又为这个事件名附加了一个随机字符串尾缀，最终的事件名类似这样：system_getOSVersion_780967123914。
    - 生成了这个事件名之后，我们就用上一节介绍的事件处理器 eventer，监听了与这个事件名同名的一次性事件。当这个一次性事件发生时，Promise 对象就成功 resolve 了。这里需要注意的是发送给 C++ 代码的事件名和事件处理器监听的事件名是相同的。
    - 在业务代码使用 native 对象之前，我们先执行了它的 init 方法，这个方法非常重要，它向渲染进程的 C++ 代码进行了一次名为 native_registe_callback 的访问，并且给这次访问传递了一个回调方法。我们预期 C++ 代码有消息需要通知 JavaScript 代码时，这个回调方法将被执行。
    - 如果 C++ 代码在执行这个回调方法时，传递来的消息名称正是我们前文介绍的那个字符串（system_getOSVersion_780967123914），这时事件处理器就会发射这个字符串所指代的事件，一旦这个事件发射之后，我们前面在 native.call 方法内注册的一次性监听的回调函数就会被调用，此时 Promise 对象就成功 resolve 掉了。
    - 使用 native.call 方法访问 C++ 代码时，除了事件名之外，还传递了业务参数 params，C++代码执行 native_registe_callback 的回调方法时，也传递了事件名和业务参数，也就是说现在 JavaScript 可以向 C++ 传递更多数据了，C++ 也可以把数据传递给 JavaScript 了。
- 要为事件名加一个随机字符串尾缀的原因：假设用户在极短的时间内多次请求了 system_getOSVersion 原生 API，由于跨进程操作都是异步的，所以可能同时有多个请求处于 pendding 状态，一旦某个请求完成后， eventer.emit 方法就会被执行，凡注册了 system_getOSVersion 事件名的回调函数都会被执行，这是不符合预期的，我们期望请求和回调是成对出现的。有了这个随机字符串尾缀就能很好的把同名请求隔离开，不会再发生某个请求的响应扰乱其他请求的现象。
- 为获取系统版本号定义一个新的对象：system，代码如下：
    ```js
    let system = {
      getMsgName(args) {
        return `system_${args.callee.name}`;
      },
      async getOSVersion() {
        let msgName = this.getMsgName(arguments);
        let osVersion = await native.call(msgName);
        return osVersion;
      },
    };
    ```
    - 这段代码和 browserWindow 对象的代码很相似，唯一不同的就是我们在 getOSVersion 方法内不再直接使用 window.nativeCall 方法访问 C++ 代码，而是使用我们前面定义的 native 对象的 call 方法访问 C++代码。
    - 由于 native.call 方法返回的是一个 Promise 对象，所以这里我们使用了 async 和 await 关键字来处理这种异步操作。

10. 在渲染进程中注册回调函数
- 现在一个重要的问题就是在执行 native.init 方法时，我们是如何把 JavaScript 的回调函数告诉 C++ 代码的，C++ 代码又是如何调用这个回调函数的。
- 在 JavaScript 调用 native.init 方法时，实际上还是调用的是 window.nativeCall 方法，这个方法最终会调用渲染进程中 V8Handler 类中的 Execute 方法。不过以前我们调用这个方法时，除了传递事件名称外，并没有传递其他数据，现在我们为它传递了一个回调函数。
- 增加了以下逻辑让 Execute 方法处理这个回调方法：
    ```cpp
    bool V8Handler::Execute(const CefString& name, CefRefPtr<CefV8Value> object, const CefV8ValueList& arguments, CefRefPtr<CefV8Value>& retval, CefString& exception)
    {
        auto msgName = arguments[0]->GetStringValue();
        if (msgName == "native_registe_callback") {
            callBack = arguments[1];
            return true;
        }
        CefRefPtr<CefProcessMessage> msg = CefProcessMessage::Create(msgName);
        CefRefPtr<CefV8Context> context = CefV8Context::GetCurrentContext();
        context.get()->GetFrame()->SendProcessMessage(PID_BROWSER, msg);
        return true;
    };
    ```
    - 在上面的代码中，我们验证事件名称，检查其是否为 native_registe_callback 事件，如果是，我们就获取这次请求的第二个参数（也就是回调方法），然后把它保存在一个公开变量 callBack 中，实际上就是保存了回调方法的引用。
    - native_registe_callback 事件是一个特殊的请求，其目的就是注册回调方法，这个工作完成后，就可以直接返回了，不必再发任何消息给主进程。
- 公开变量 callBack 是在 V8Handler 的头文件中定义的，其代码如下所示：
    ```cpp
    public:
	    CefRefPtr<CefV8Value> callBack;
    ```
    - 它是一个 CefV8Value 的对象指针，实际上 JavaScript 与 C++ 交互的任何对象、字符串、函数都可以使用这个类型来保存。
- 当需要执行这个回调方法时，我们只要执行如下代码即可（obj 是传递给回调函数的的参数）：
    ```cpp
    v8Handler->callBack->ExecuteFunction(nullptr, obj);
    ```
- 现在我们在渲染进程中保留了一个 JavaScript 回调函数的引用，我们前文介绍的 native.call 返回的 Promise 对象，就全靠这个回调函数来 resolve。（因为这个回调函数 emit 了消息）
-  我们知道 native.call('system_getOSVersion') 语句最终执行的是 window.nativeCall('system_getOSVersion_780967123914') 语句，渲染进程的 C++ 代码收到这个请求之后，会封装一个名为'system_getOSVersion_780967123914'的消息发送给浏览器进程，接下来我们就看一下浏览器进程如何接收这个事件、如何获取操作系统的版本号信息，如何把这个信息发送给渲染进程。
- 调用关系：前端页面(js)->渲染进程(C++)->浏览器进程(主进程)调用操作系统API拿到版本号(C++)->渲染进程调用 JavaScript 回调函数的引用->前端页面(js)继续往下执行。

11. 主进程获取版本号信息
- 浏览器进程在 PageHandler 类的 OnProcessMessageReceived 方法中接收并处理渲染进程发来的消息，在上一节中我们只处理了 window 类型的消息。
- 接下来我们为获取系统版本号另开辟一个新的消息类型：system，在收到这类消息时，我们将完成获取版本号的工作并把版本号信息返回给渲染进程，如下代码所示：
    ```cpp
    else if(arr.at(0) == "system") {
        // messageName是在进入OnProcessMessageReceived方法时就通过如下代码获得了，它就是渲染进程发来的消息名
        // std::string messageName = message->GetName();
        CefRefPtr<CefProcessMessage> msg = CefProcessMessage::Create(messageName);
        CefRefPtr<CefListValue> msgArgs = msg->GetArgumentList();
        if (arr.at(1) == "getOSVersion") {
            std::string version = getOSVersion();
            msgArgs->SetString(0, version);
        }
        frame->SendProcessMessage(PID_RENDERER, msg);
    }
    ```
    - 使用 CefProcessMessage 类的 Create 静态方法创建了一个进程间消息，没错，在渲染进程中我们也是使用它创建的进程间消息。这个方法既可以在渲染进程中使用，也可以在浏览器进程中使用。
    - 这个消息的名字是与渲染进程发来的消息的名字一模一样的，就是'system_getOSVersion_780967123914'，这个名字是在 JavaScript 代码中定义的。
    - getOSVersion 方法是我们自定义的一个方法，用于获取操作系统的版本号，我们稍后会介绍这个方法。
    - 获取到版本号信息后，我们把这个信息插入到进程间消息的消息体内，这个工作是通过操作消息体的 CefListValue 属性完成的。开发者可以为这个属性添加多个不同类型的数据值，这些数据会随着消息体被发送到目标进程，无论是浏览器进程发消息给渲染进程还是渲染进程发消息给浏览器进程都可以这么做。
    - 使用 frame 对象的 SendProcessMessage 方法把消息发送给渲染进程，即使在同一个窗口内，不同的 frame 对象也可能对应着不同的渲染进程处理对象，这里我们的处理原则是：谁发消息给浏览器进程了，我们就向谁回传消息。
- getOSVersion 这个工具函数的代码如下所示：
    ```cpp
    const std::string getOSVersion()
    {
        NTSTATUS(WINAPI* RtlGetVersion)(LPOSVERSIONINFOEXW);
        OSVERSIONINFOEXW osInfo;
        std::string result;
         *(FARPROC*)&RtlGetVersion = GetProcAddress(GetModuleHandle(L"ntdll"), "RtlGetVersion");
        if (nullptr != RtlGetVersion) {
            osInfo.dwOSVersionInfoSize = sizeof osInfo;
            RtlGetVersion(&osInfo);
            result = std::to_string(osInfo.dwMajorVersion) +"." + std::to_string(osInfo.dwMinorVersion) +"." + std::to_string(osInfo.dwBuildNumber);
        }
        return result;
    }
    ```
    - 在这段代码中调用 Windows 操作系统的 API，来获取操作系统的版本信息。
- 在我的电脑上，这个 API 获取到的版本信息为：10.0.16299，开发者可以到这个页面：https://docs.microsoft.com/en-us/windows/win32/sysinfo/operating-system-version 查询这个版本号对应的具体系统名称。获取其他类型的系统信息，请参考这个页面提供的 API：https://docs.microsoft.com/en-us/windows/win32/sysinfo/getting-system-information。

12. 渲染进程接收消息并传递给 JavaScript
- 渲染进程的 Renderer 类继承自 CefRenderProcessHandler 基类，这个 CefRenderProcessHandler 基类描述了一个方法：OnProcessMessageReceived（没错，这个方法与浏览器进程接收进程间消息的方法同名），我们只要重写这个方法就可以在渲染进程中接收消息了，如下代码所示：
    ```cpp
    bool Renderer::OnProcessMessageReceived(CefRefPtr<CefBrowser> browser, CefRefPtr<CefFrame> frame, CefProcessId source_process, CefRefPtr<CefProcessMessage> message)
    {
        CefString messageName = message->GetName();
        CefRefPtr<CefListValue> args = message->GetArgumentList();
        CefString result = args->GetString(0);
        CefRefPtr<CefV8Value> messageNameV8 = CefV8Value::CreateString(messageName);
        CefRefPtr<CefV8Value> resultV8 = CefV8Value::CreateString(result);
        CefV8ValueList argsForJs;
        argsForJs.push_back(messageNameV8);
        argsForJs.push_back(resultV8);
        CefRefPtr<CefV8Context> context = frame->GetV8Context();
        context->Enter();
        v8Handler->callBack->ExecuteFunction(nullptr, argsForJs);
        context->Exit();
        return true;
    }
    ```
    - 这个方法的最后一个参数就是主进程发来的消息对象。
    - 获取消息的名称（'system_getOSVersion_780967123914'），并保存在 messageName 变量中。
    - 获取消息体内的第一个数据，并保存在 result 变量中，这个数据就是操作系统的版本号字符串。
    - 使用 CefV8Value 类的静态方法 CreateString 创建两个 V8 字符串，一个用于保存消息名称，一个用于保存系统版本号。
    - 把前面创建的两个 V8 字符串添加到一个 CefV8ValueList 类型的对象 argsForJs 中。
    - 使用 frame 对象的 GetV8Context 方法获取到 JavaScript 的执行上下文。
    - 进入 JavaScript 的执行上下文环境，并执行 v8Handler 对象保存的回调方法 callBack，然后退出 JavaScript 的执行上下文环境。
    - 执行 callBack 方法时把 argsForJs 对象传递给这个方法，argsForJs 里保存的两个数据（消息名称和版本号）将用作 callBack 对应的 JavaScript 方法的入参。
    - 返回 true，告知 CEF 框架，我们已经成功的处理了主进程发来的消息。
- 至此 JavaScript 获取信息，C++返回信息的需求我们就正式完成了。
- 整个过程中有两个数据一直没有变：
    - 第一个是事件名称：事件名称从 JavaScript 传递给渲染进程，再从渲染进程传递给浏览器进程，再从浏览器进程返回给渲染进程，再从渲染进程返回给 JavaScript，整个过程都没有变化，我们可以把它当做一个业务 ID，无论系统中有多少个事件正在发生，只要这个业务 ID 不变，事件和事件之间就不会有任何干扰和冲突。
    - 第二个是 frame 对象，自从渲染进程收到了 JavaScript 请求之后，渲染进程就知道是哪个 frame 再向自己发出请求，而且渲染进程也是通过这个 frame 向主进程发送消息的。主进程收到消息时就知道是渲染进程的哪个 frame 在向自己发送消息，而且主进程向渲染进程回传消息时，也是通过这个 frame 完成工作的。渲染进程收到主进程发来的消息后，又通过同一个 frame 执行回调函数。这个逻辑保证了消息从谁那儿来，响应就发送给谁。无论系统中有多少个页面实例，浏览器进程与渲染进程之间的消息流转都不会发生错乱。

---

###### 五、本地数据

1. JavaScript 传递复杂数据
- 路径对话框有很多可配置项，比如窗口标题，过滤字符串，默认路径，是否允许多选等。
- 我们计划在 JavaScript 代码中把这些配置项封装到一个 Json 对象中，然后把这个 Json 对象传递给渲染进程，再由渲染进程把这个 Json 对象传递给主进程，再由主进程使用这个 Json 对象携带的数据来打开路径对话框，当用户做出路径选择后，再由主进程封装一个新的 Json 对象并发送给渲染进程，渲染进程收到这个对象之后再传递给 JavaScript 代码，这样 JavaScript 代码就得到了用户选择的路径了。
- 计划确定之后，我们先从 JavaScript 代码着手开展工作，首先封装一个新的对象 dialog，代码如下：
    ```js
    let dialog = {
      getMsgName(args) {
        return `dialog_${args.callee.name}`;
      },
      async openFile(param) {
        let msgName = this.getMsgName(arguments);
        let resultStr = await native.call(msgName, JSON.stringify(param));
        return JSON.parse(resultStr);
      },
      async openFolder(param) {
        let msgName = this.getMsgName(arguments);
        let resultStr = await native.call(msgName, JSON.stringify(param));
        return JSON.parse(resultStr);
      },
    };
    ```
    - dialog 对象的两个方法 openFile 和 openFolder 都向渲染进程传递了一个 Json 字符串，而且这两个方法都返回了一个 Json 对象，返回的 Json 对象我们稍后再介绍。先看着两个 Json 字符串是从何而来的。
- 这两个 Json 字符串是从一个 Json 配置对象中序列化来的，这个配置对象用于控制路径对话框的外观和行为，它是在按钮点击事件中定义的，代码如下所示：
    ```js
    let fileOpenBtn = document.querySelector("#fileOpenBtn");
    fileOpenBtn.addEventListener("click", async () => {
      let param = {
        title: "这是打开文件对话框的标题",
        defaultPath: "C:\\Program Files",
        filters: ["image/*", "text/*"],
        filterIndex: 1,
        multiSelections: true,
      };
      let files = await dialog.openFile(param);
      console.log(files);
    });
    let dirOpneBtn = document.querySelector("#dirOpneBtn");
    dirOpneBtn.addEventListener("click", async () => {
      let param = {
        title: "这是打开文件夹对话框的标题",
        defaultPath: "C:\\Program Files",
      };
      let files = await dialog.openFolder(param);
      console.log(files);
    });
    ```
    - Json 对象中 title 是对话框的标题，defaultPath 是对话框的默认路径，filters 是对话框文件过滤器数组，filterIndex 为默认对话框文件过滤器的下标，multiSelections 为是否允许用户在对话框中选择多个文件。
    - 当用户点击按钮的时候，这个配置对象将被序列化成字符串再发送到渲染进程，接下来我们就看一下渲染进程的实现逻辑。

2. 渲染进程中转复杂数据
- JavaScript 传递数据给渲染进程时最终执行的是渲染进程的 V8Handler 类的 Execute 方法，这个方法的 arguments 参数携带了 JavaScript 传递过来的参数，接下来我们看一下这个方法的代码：
    ```cpp
    bool V8Handler::Execute(const CefString& name, CefRefPtr<CefV8Value> object, const CefV8ValueList& arguments, CefRefPtr<CefV8Value>& retval, CefString& exception)
    {
        auto msgName = arguments[0]->GetStringValue();
        if (msgName == "native_registe_callback") {
            callBack = arguments[1];
            return true;
        }
        CefRefPtr<CefProcessMessage> msg = CefProcessMessage::Create(msgName);
        CefRefPtr<CefListValue> msgBody =  msg->GetArgumentList();
        if (arguments.size() > 1 && arguments[1]->IsString()) {
            msgBody->SetString(0, arguments[1]->GetStringValue());
        }
        CefRefPtr<CefV8Context> context = CefV8Context::GetCurrentContext();
        context.get()->GetFrame()->SendProcessMessage(PID_BROWSER, msg);
        return true;
    };
    ```
    - 在这段代码中增加了一个判断逻辑，如果 JavaScript 传递过来了 2 个以上的参数，并且第二个参数是字符串类型的话，那么我们就把这个参数附加到发送给主进程的消息体内。注意这里并没有做类型转换，JavaScript 传递来的是字符串，我们就原封不动把这个字符串转发给了浏览器进程。
- 浏览器进程才是处理配置对象的核心场所。

3. 主进程处理复杂数据
- 我们知道浏览器进程是通过 PageHandler 类的 OnProcessMessageReceived 方法来处理渲染进程发来的消息的，前面我们已经完成了 window、system 的处理逻辑，接下来我们就看一下 dialog 的处理逻辑，代码如下所示：
    ```cpp
    else if (arr.at(0) == "dialog") {
        CefRefPtr<CefListValue> msgBody = message->GetArgumentList();
        nlohmann::json param = nlohmann::json::parse(msgBody->GetString(0).ToString());
        std::wstring title = convertStr(param["title"].get<std::string>());
        std::wstring defaultPath = convertStr(param["defaultPath"].get<std::string>());
        CefRefPtr<CefRunFileDialogCallback> dcb = new DialogHandler(messageName, frame);
        CefBrowserHost::FileDialogMode mode;
        std::vector<CefString> fileFilters;
        int filterIndex = 0;
        if (arr.at(1) == "openFile") {
            for (const std::string& var : param["filters"]) {
                fileFilters.push_back(var);
            }
            filterIndex = param["filterIndex"].get<int>();
            mode = param["multiSelections"].get<bool>() ? FILE_DIALOG_OPEN_MULTIPLE : FILE_DIALOG_OPEN;
            browser->GetHost()->RunFileDialog(mode, title, defaultPath, fileFilters, filterIndex, dcb);
        }
        else if (arr.at(1) == "openFolder") {
            mode = FILE_DIALOG_OPEN_FOLDER;
            browser->GetHost()->RunFileDialog(mode, title, defaultPath, fileFilters, filterIndex, dcb);
        }
    }
    ```
    - 在这段代码中，我们获取到渲染进程发来的消息的第一项数据，也就 JavaScript 传来的 Json 字符串，并把这个 Json 字符串转化为一个 Json 对象。
    - 得到可用的配置数据之后，我们创建了一个自定义类型 DialogHandler 的对象。需要注意的是我们实例化 DialogHandler 对象时，把 frame 和 messageName 传递给了这个对象，当用户在路径对话框内做出选择之后（无论是选择了路径还是直接点击了取消按钮），CEF 框架会调用 DialogHandler 对象的 OnFileDialogDismissed 方法，在这个方法中我们使用 frame 和 messageName 向渲染进程发送消息，这部分逻辑我们稍后还会有详细介绍。
    - FileDialogMode 是 CEF 定义的对话框类型枚举，CEF 定义了很多对话框类型，但这里我们只用到了三种：打开单个文件对话框、打开多个文件对话框、打开路径对话框。这个枚举的值是通过消息名称和配置对象的 multiSelections 属性共同决定的。
    - fileFilters 是存储文件过滤器的容器，我们通过 param["filters"] 获取到 JavaScript 传递过来的过滤器数组之后，就把这个数组内的值填充到这个 C++ 容器中了。
    - 准备好所有的对话框配置项之后，我们就通过 BrowserHost 对象的 RunFileDialog 方法打开了对话框，这个方法所需的参数就是我们前面准备好的配置项。
- 转化过程是我们通过一个第三方 C++ 库完成的，这个库的开源地址为：https://github.com/nlohmann/json，只要把它的 single_include/nlohmann 目录下的 json.hpp 文件拷贝到自己的工程中，就可以通过 #include "Helper/json.hpp" 使用它了。
- nlohmann::json::parse 方法可以把一个字符串转化为一个 nlohmann::json 类型的 C++ 对象，转化成 C++ 对象之后就可以使用 param["title"].get<std::string>() 这样的方式获取对象中的数据了。
- 由于这个库使用 utf-8 的编码格式反序列化 Json 字符串，所以包含中文的数据要经过一次转码才能使用（不然会出现乱码），convertStr 方法就是负责转码的工具函数，代码如下：
    ```cpp
    std::wstring convertStr(const std::string& str)
    {
        static std::wstring_convert<std::codecvt_utf8<wchar_t>> utf8_conv;
        return utf8_conv.from_bytes(str);
    }
    ```
    - 这个方法负责把 std::string 类型的数据转化为 std::wstring 类型的数据。
- 代码运行至此，路径对话框就被打开了，这里需要注意的是打开路径对话框之后我们并没有发送任何消息给渲染进程，也就是说此时 JavaScript 的 Promise 对象尚未成功 resolve。只有在用户做出选择（或者取消选择）之后，Promise 对象才会成功 resolve，而且此时 JavaScript 代码会得到用户选择的具体路径信息。（这些工作都是在 DialogHandler 类中完成的）

4. 路径对话框处理类
- DialogHandler 类继承自 CefRunFileDialogCallback 基类，这个类只有一个有价值的方法：OnFileDialogDismissed，当路径对话框被关闭时，CEF 框架会主动调用这个方法，DialogHandler 类的代码如下所示：
    ```cpp
    #pragma once
    #include "include/wrapper/cef_message_router.h"
    #include "include/cef_browser.h"
    #include "Helper/json.hpp"
    using nlohmann::json;
    class DialogHandler : public CefRunFileDialogCallback
    {
    public:
    	DialogHandler(std::string& msgName, CefRefPtr<CefFrame> frame) :msgName(msgName), frame(frame) {};
    	void OnFileDialogDismissed(int selected_accept_filter, const std::vector<CefString>& file_paths) override {
    		json result;
    		result["success"] = true;
    		result["data"] = {};
    		for (size_t i = 0; i < file_paths.size(); i++)
    		{
    			result["data"].push_back(file_paths[i].ToString());
    		}
    		CefRefPtr<CefProcessMessage> msgBack = CefProcessMessage::Create(msgName);
    		CefRefPtr<CefListValue> msgArgs = msgBack->GetArgumentList();
    		std::string dataStr = result.dump();
    		msgArgs->SetString(0, dataStr);
    		frame->SendProcessMessage(PID_RENDERER, msgBack);
    	}
    	DialogHandler(const DialogHandler&) = delete;
    	DialogHandler& operator=(const DialogHandler&) = delete;
    private:
    	std::string msgName;
    	CefRefPtr<CefFrame> frame;
    	IMPLEMENT_REFCOUNTING(DialogHandler);
    };
    ```
    - 当 OnFileDialogDismissed 方法被执行时，我们创建了一个 nlohmann::json 类型的对象，用户选择的文件路径或目录路径将以字符串的形式存放在这个对象的 data 属性内。data 属性是一个数组类型的值，可以存放多个数据。我们可以使用这个 Json 对象的 dump 方法把这个 Json 对象序列化成字符串。
    - 得到用户选择的数据之后，我们就把这个数据存放到进程间消息 msgBack 中了。最后通过 frame 对象把消息发回给渲染进程。frame 对象和 msgName 字符串都是在实例化 DialogHandler 对象时，通过构造函数传进来的。
- 渲染进程接到浏览器进程的消息之后，就会把这个消息发送给 JavaScript，由于 msgName 是在 JavaScript 发起异步请求前定义的，所以 JavaScript 收到这个返回的消息后，Promise 对象就会被成功 resolve 了，而且 resolve 得到的结果就是由用户选择的文件路径组成的 Json 字符串，接着我们使用 JSON.parse 方法把这个字符串序列化成 Json 对象，这就是前文 dialog.openFile 和 dialog.openFolder 方法的返回值。
- 值得注意的是我们实现的路径的对话框是模态的，也就是说当这个对话框弹出后，浏览器进程的所有操作都会被阻塞（渲染进程与 JavaScript 执行线程不会被阻塞）。
- 实际上我们在浏览器进程完成的几项任务都是在浏览器进程的主线程中完成的，也就是说这几项任务都是阻塞的，只不过获取版本号或者控制窗口这类操作在极短的时间内就完成了，所以我们感觉不到阻塞的影响。

5. 在 JavaScript 中分片接收文件数据
- 为了在 JavaScript 中发起读文件的操作，我们创建了一个新的对象 file，代码如下所示：
    ```js
    let file = {
      getMsgName(args) {
        return `file_${args.callee.name}_${this.randomNum()}`;
      },
      randomNum(len = 12) {
        return Math.floor(Math.pow(10, len) * Math.random());
      },
      readFile(param) {
        let msgName = this.getMsgName(arguments);
        let onData = (obj) => {
          if (param.onData) {
            param.onData(obj);
          }
        };
        let onFinish = (obj) => {
          if (param.onFinish) {
            param.onFinish(obj);
          }
          eventer.off(`${msgName}_data`, onData);
        };
        eventer.on(`${msgName}_data`, onData);
        eventer.once(`${msgName}_finish`, onFinish);
        window.nativeCall(msgName, JSON.stringify(param));
      },
    };
    ```
    - 这个对象的 readFile 方法负责发起读取文件的操作，我们并没有在这个方法中继续使用 native 对象的 call 方法来向渲染进程发起请求，这主要是基于以下两个原因的考虑结果：
        - 读取超大文件不应该把文件数据一次性全部读取到内存中，因为用户的内存是有限的，如果用户内存不够用的话可能会导致我们的应用程序崩溃，所以要分片读取文件数据，这就要求开发者提供一个 onData 的回调方法，当读取完一片数据之后，onData 方法被调用一次，直到整个文件全部读取完成为止，然而 native.call 方法内注册的是一个一次性的回调事件，执行一次之后就自己把自己删除了，所以它无法满足我们的要求。
        - 当然，我们也可以为 native 对象提供新的方法来实现需求，但这样会导致代码变得复杂，不利于理解本节的重点知识，所以我们就没有对 native 对象进行扩展。
- 在 file.readFile 方法执行时，我们依然通过包含随机字符串尾缀的事件名向渲染进程发起请求，而且在请求时还传送了一个 Json 字符串（这个字符串中包含文件路径信息）。
- 在向渲染进程发起请求前，我们注册了两个事件，一个是file_readFile_49471465148_data（普通事件），一个是file_readFile_49471465148_finish（一次性事件），我们预期每读取一片数据就触发一次 file_readFile_49471465148_data 事件，当所有数据都读取完成后，触发一次 file_readFile_49471465148_finish 事件。
- 如果开发者在调用 file.readFile 方法时，传递的参数 param 中包含 onData 和 onFinish 回调函数，那么我们就会在 file_readFile_49471465148_data 事件和 file_readFile_49471465148_finish 事件发生时，调用这两个回调函数。而且当 onFinish 回调函数被执行完成之后，我们还会把 file_readFile_49471465148_data 事件从事件对象中移除。
- 用户虽然在 param 参数中传递了回调函数，但这个回调函数并没有被发送给渲染进程的 C++ 代码，因为 JSON.stringify 序列化工作把这些函数给屏蔽掉了。
- file.readFile 方法的使用者的实现逻辑如下代码所示：
    ```js
    let readFileBtn = document.querySelector("#readFileBtn");
    readFileBtn.addEventListener("click", async () => {
      let result = "";
      let param = {
        filePath: "E:\\project\\bread\\14.md",
        onData(chunk) {
          let decoder = new TextDecoder("utf-8", { ignoreBOM: true });
          let str = decoder.decode(chunk);
          result += str;
        },
        onFinish(data) {
          console.log("文件读取完成");
          console.log(result);
        },
      };
      file.readFile(param);
    });
    ```
    - 这段代码并不复杂，在用户点击 readFileBtn 按钮的时候，会发起一个读取 E:\\\project\\\bread\\\14.md 文件的请求，在 onData 回调方法被执行时，也就是获取到一片数据之后，我们使用 TextDecoder 对象解码数据内容，并把它存储到字符串中。在 onFinish 回调方法被执行时，也就是文件读取完成后，我们把文件的所有内容打印到控制台上。
    - 这段代码有三点需要注意：
        - 这里我们读取的是一个文本文件，并以 utf-8 格式解码文件内容，实际开发过程中可能读取的是其他类型的文件，处理数据的逻辑要相应的做出调整。
        - 虽然我们前面提到超大文件可能会导致内存泄漏，但这里我们仍然把文件的所有内容都存储到 result 变量中，这主要是为了演示，并不应该用于实际生产工作中，实际生产中应该在得到一片数据之后就处理这一片数据，处理完成之后再索取下一片数据。
        - chunk 是一个 ArrayBuffer 类型的数据，由于一个中文会占用两个字节，所以某片数据的结尾字符可能正好是一个中文字的一半，为了不影响主题，我们并未处理这个问题，我们应该明确应用程序读取的是什么类型的文件，读取文本文件时应该考虑在 C++ 代码内就完成转码工作，直接向 JavaScript 输出转码后的字符串。

6. 在浏览器进程中使用新线程读取数据
- 浏览器进程的 PageHandler::OnProcessMessageReceived 方法会处理渲染进程发来的消息，我们在本节处理的是 file 类型的消息，如下代码所示：
    ```cpp
    // 需要引入如下两个头文件
    // #include "include/cef_task.h"
    // #include "include/wrapper/cef_closure_task.h"
    else if (arr.at(0) == "file") {
        if (arr.at(1) == "readFile") {
            CefRefPtr<CefListValue> msgBody = message->GetArgumentList();
            nlohmann::json param = nlohmann::json::parse(msgBody->GetString(0).ToString());
            std::string filePath = param["filePath"].get<std::string>();
            CefPostTask(TID_FILE_BACKGROUND, base::BindOnce(&ReadFileByBlocks, filePath,messageName, frame));
        }
    }
    ```
    - 在这个逻辑分支下，我们获取了 JavaScript 传递过来的 Json 字符串，并把这个字符串序列化成 nlohmann::json 类型的对象，然后从这个对象中得到 filePath 路径，也就是我们要读取的文件的路径。
    - 最后通过 CefPostTask 方法把读取文件的工作放置到一个新的线程中完成，这行代码有以下几点需要注意：
        - CefPostTask 方法的第一个参数 TID_FILE_BACKGROUND 是一个线程标记，它指代的线程就是文件读写后台线程，在这个线程下执行的任务不会阻塞浏览器主线程的操作，浏览器进程的主线程的标记为 TID_UI，渲染进程主线程的标记为 PID_RENDERER。
        - CefPostTask 方法的第二个参数是由 base::BindOnce 创建的一个任务对象，这个对象的类型是 base::OnceClosure，我们用这个对象包装了 ReadFileByBlocks 方法和它的参数。
    - 也就是说 CefPostTask 方法会在 TID_FILE_BACKGROUND 线程上调用 ReadFileByBlocks 方法，调用这个方法时还把主线程里获取到的数据传递给了这个方法。
- 接下来我们就看一下 ReadFileByBlocks 方法的代码实现：
    ```cpp
    // 需要引入如下两个标准库
    // #include <iostream>
    // #include <fstream>
    void ReadFileByBlocks( const std::string& filePath,const std::string& msgName, CefRefPtr<CefFrame> frame) {
        std::ifstream fileStream(filePath, std::ifstream::binary);
        if (fileStream) {
            fileStream.seekg(0, fileStream.end);
            long length = fileStream.tellg();
            fileStream.seekg(0, fileStream.beg);
            long size = 1024;
            long position = 0;
            while (position != length) {
                long leftSize = length - position;
                if (leftSize < size) {
                    size = leftSize;
                }
                char* buffer = new char[size];
                fileStream.read(buffer, size);
                position = fileStream.tellg();
                CefRefPtr<CefBinaryValue> data = CefBinaryValue::Create(buffer, size);
                CefRefPtr<CefProcessMessage> msgBack = CefProcessMessage::Create(msgName+"_data");
                CefRefPtr<CefListValue> msgArgs = msgBack->GetArgumentList();
                msgArgs->SetBinary(0, data);
                frame->SendProcessMessage(PID_RENDERER, msgBack);
                delete[] buffer;
            }
            fileStream.close();
            CefRefPtr<CefProcessMessage> msgBack = CefProcessMessage::Create(msgName + "_finish");
            frame->SendProcessMessage(PID_RENDERER, msgBack);
        }
    }
    ```
    - 在这个方法中，我们使用 C++标准库完成了文件的分批次读取工作。
    - 以二进制资源的方式实例化 fileStream 对象，这个对象用于读取文件。
    - 通过把文件读取指针定位到最后一个字符来获取文件大小length。
    - 再次把文件读取指针定位到第一个字符，开始循环读取文件。
    - 每次尝试读取 1024 字节的内容，如果剩余内容不足 1024 字节，那么就仅仅读取剩余内容。
    - 通过 CefBinaryValue::Create 方法创建一个 CefBinaryValue 类型的对象，这个对象持有读取到的文件内容，这个对象被封装到消息体内发送给渲染进程。渲染进程的消息名为当前消息名再附加一个"_data"字符串尾缀。渲染进程收到这个消息后会发射 JavaScript 的同名事件，此时前文介绍的 onData 回调方法就会被执行。
    - 当读完文件的所有内容后，再创建一个名为 msgName + "_finish"的消息发送给渲染进程，以告知 JavaScript 读取工作宣告完成。
- 我们在发送文件内容的消息中使用了二进制数据对象 CefBinaryValue，渲染进程要撰写额外的逻辑才能处理这种类型的数据，接下来我们就看一下渲染进程是如何处理这些二进制数据的。

7. 渲染进程向 JavaScript 传递二进制数据
- 我们在渲染进程的 Renderer::OnProcessMessageReceived 方法中处理主进程发来的消息，这个方法的大部分逻辑我们前面已经介绍过了，本节我们只增加了处理二进制数据的逻辑，如下代码所示：
    ```cpp
    bool Renderer::OnProcessMessageReceived(CefRefPtr<CefBrowser> browser, CefRefPtr<CefFrame> frame, CefProcessId source_process, CefRefPtr<CefProcessMessage> message)
    {
        CefString messageName = message->GetName();
        CefRefPtr<CefListValue> args = message->GetArgumentList();
        CefRefPtr<CefV8Context> context = frame->GetV8Context();
        context->Enter();
        CefRefPtr<CefV8Value> messageNameV8 = CefV8Value::CreateString(messageName);
        CefV8ValueList argsForJs;
        argsForJs.push_back(messageNameV8);
        if (args->GetType(0) == CefValueType::VTYPE_STRING) {
            CefString result = args->GetString(0);
            CefRefPtr<CefV8Value> resultV8 = CefV8Value::CreateString(result);
            argsForJs.push_back(resultV8);
        } //下面else if分支是我们增加的内容
        else if (args->GetType(0) == CefValueType::VTYPE_BINARY) {
            CefRefPtr<CefBinaryValue> data = args->GetBinary(0);
            size_t size = data->GetSize();
            unsigned char* result = new unsigned char[size];
            data->GetData(result, size, 0);
            CefRefPtr<CefV8ArrayBufferReleaseCallback> cb = new ReleaseCallback();
            CefRefPtr<CefV8Value> resultV8 = CefV8Value::CreateArrayBuffer(result, size, cb);
            argsForJs.push_back(resultV8);
        }
        v8Handler->callBack->ExecuteFunction(nullptr, argsForJs);
        context->Exit();
        return true;
    }
    ```
    - 当获取到的消息体中第一项数据为 CefValueType::VTYPE_BINARY 类型时，我们将对消息体内的数据执行特殊的处理。这部分逻辑主要完成了如下几项工作：
        - 读取消息体的数据，并把它存储在无符号数组 result 中。
        - 使用 CefV8Value::CreateArrayBuffer 方法把无符号数组 result 封装到一个 CefV8Value 对象 resultV8 中，这个对象就是 JavaScript 中的 ArrayBuffer 对象。
        - 把 resultV8 传递给我们的回调方法（在 JavaScript 中使用 native_registe_callback 注册的回调方法）。
        - 由于 JavaScript 将继续持有 resultV8 包装的二进制数据，所以我们不能在 OnProcessMessageReceived 方法执行完成前释放这部分内存，所以我们为 resultV8 对象提供了一个 ReleaseCallback 对象，当 JavaScript 不再使用 resultV8 所引用的二进制数据时，这个 ReleaseCallback 对象的 ReleaseBuffer 方法将被执行，我们将在这个方法中释放 resultV8 所引用的二进制数据。
- ReleaseCallback 是一个自定义类型，它的代码如下所示：
    ```cpp
    #pragma once
    #include "V8Handler.h"
    class ReleaseCallback : public CefV8ArrayBufferReleaseCallback {
    public:
        void ReleaseBuffer(void* buffer) override {
            std::free(buffer);
        }
        IMPLEMENT_REFCOUNTING(ReleaseCallback);
    };
    ```
    - 这个类继承自 CefV8ArrayBufferReleaseCallback 基类，但凡在 C++ 中创建 ArrayBuffer 对象，都应该使用这个类来释放 ArrayBuffer 所引用的资源，不然会造成内存泄漏的问题。

8. 在 JavaScript 中操作数据库
- 按照惯例，我们还是先在 JavaScript 中创建一个数据库操作对象db，如下代码所示：
    ```js
    let db = {
      getMsgName(args) {
        return `db_${args.callee.name}`;
      },
      async open(param) {
        let msgName = this.getMsgName(arguments);
        let result = await native.call(msgName, JSON.stringify(param));
        return JSON.parse(result);
      },
      async close() {
        let msgName = this.getMsgName(arguments);
        let result = await native.call(msgName);
        return JSON.parse(result);
      },
      async execute(param) {
        let msgName = this.getMsgName(arguments);
        let result = await native.call(msgName, JSON.stringify(param));
        return JSON.parse(result);
      },
    };
    ```
    - 这个对象提供了三个 async 方法。
    - open 方法用于打开数据库，一个数据库只有处于打开状态才能被 SQL 指令操作，这个方法要求使用者传递一个数据库文件的路径字符串，这个字符串是从 JSON 对象 param 中获取的。
    - close 方法用于关闭数据库，关闭数据库应该与打开数据库成对出现，一般在应用初始化成功之后打开数据库，应用退出之前关闭数据库。
    - execute 方法用于执行 SQL 语句，SQL 语句是从 JSON 对象 param 中获取的。
    - 这三个方法仍然使用我们之前封装的 native.call 方法与 C++ 代码交互。这三个方法均返回一个 JSON 对象。
- 我们把打开数据库和关闭数据库的操作放在一个按钮点击事件中，代码如下所示：
    ```js
    let dbBtn = document.querySelector("#dbBtn");
    dbBtn.addEventListener("click", async () => {
      if (dbBtn.innerHTML === "打开数据库") {
        let result = await db.open({
          dbPath: "E:\\project\\cef-in-action\\test.db",
        });
        console.log(result);
        dbBtn.innerHTML = "关闭数据库";
      } else if (dbBtn.innerHTML === "关闭数据库") {
        let result = await db.close();
        console.log(result);
        dbBtn.innerHTML = "打开数据库";
      }
    });
    ```
    - 打开数据库时，我们传递了一个写死的数据库路径，这个数据库是我们提前创建并设计好的，推荐大家使用 SQLite Expert（http://www.sqliteexpert.com/） 这个工具来管理 SQLite 数据库。
- 在我们的示例数据库中只有一张数据表：Message，它是通过如下 SQL 语句在 SQLite Expert 中创建的。
    ```sql
    CREATE TABLE Message(Message TEXT NOT NULL, fromUser CHAR(60) NOT NULL, toUser CHAR(60)  NOT NULL, sendTime TIMESTAMP DEFAULT CURRENT_TIMESTAMP);
    ```
- 实际项目中数据库的路径可以让用户自己指定，这就需要用到我们之前章节中介绍的路径对话框了。数据库结构也可以动态创建，你可以利用本节介绍的程序来执行上述建表语句。
- 当我们调用 open 方法打开数据库之后，我们就可以通过点击另一个按钮来向数据库插入 60 行数据，代码如下所示：
    ```js
    let insertBtn = document.querySelector("#insertBtn");
    insertBtn.addEventListener("click", async () => {
      //
      let msgs = [
        {
          message: `天接云涛连晓雾`,
          fromUser: `李清照`,
          toUser: "辛弃疾",
        },
        {
          message: `醉里挑灯看剑`,
          fromUser: `辛弃疾`,
          toUser: "李清照",
        },
      ];
      let sqls = [];
      for (let i = 0; i < 60; i++) {
        let msg = msgs[i % 2];
        sqls.push(
          `insert into Message(Message, fromUser, toUser) values ('${msg.message}','${msg.fromUser}','${msg.toUser}');`
        );
      }
      let result = await db.execute({ sql: sqls.join("") });
      console.log(result);
    });
    ```
    - 在这段代码中，我们向 Message 表插入了 60 行数据，每一行数据对应一条 SQL 语句，60 行 SQL 语句通过分号分割开，最终被拼装到一起发送给浏览器进程。
- 在向数据库 Message 表插入数据之后，我们就可以从数据库中检索数据了，如下代码所示：
    ```js
    let selectBtn = document.querySelector("#selectBtn");
    selectBtn.addEventListener("click", async () => {
      let sql = `select rowid,* from  Message limit 16;`;
      let result = await db.execute({ sql });
      console.log(result);
    });
    ```
    - 这段代码从数据库中检索出 16 行数据，并显示在开发者工具控制台上。
- 更新数据库中第一行数据的代码如下所示：
    ```js
    let updateBtn = document.querySelector("#updateBtn");
    updateBtn.addEventListener("click", async () => {
      let obj = {
        message:
          "怒发冲冠",
        fromUser: "岳飞",
        toUser: "辛弃疾",
      };
      let sql = `update Message set Message = '${obj.message}',fromUser = '${obj.fromUser}',toUser='${obj.toUser}' where rowid in  (select rowid from Message limit 1);`;
      let result = await db.execute({ sql });
      console.log(result);
    });
    ```
    - 这里用到了一个较为复杂的 where 子句，这个 where 子句负责检索出第一行数据的 rowid，更多关于 SQL 的语法请参考：https://www.w3school.com.cn/sql/index.asp
- 删除数据库中第一行数据的代码如下所示：
    ```js
    let deleteBtn = document.querySelector("#deleteBtn");
    deleteBtn.addEventListener("click", async () => {
      let sql = `delete from Message where rowid in  (select rowid from Message limit 1);`;
      let result = await db.execute({ sql });
      console.log(result);
    });
    ```

9. 在主进程中引入 SQLite 数据库
- SQLite 数据库是使用 C 语言创建的，所以在 C++应用中集成 SQLite 数据库非常简单，只要到 SQLite 官网（https://www.sqlite.org/download.html）下载它的源码文件并把这些源码文件复制到你的工程中即可。
- 在使用 SQLite 数据库之前需要引用 SQLite 数据库的头文件 #include "SQLite/sqlite3.h"，引入 SQLite 头文件之后，我们就可以对数据库进行操作了。
- 我们首先声明一个全局静态变量 db：static sqlite3* db;，后面所有关于数据库的操作都是针对这个变量完成的。
- 在 PageHandler::OnProcessMessageReceived 方法中，我们接收并处理了数据库的打开、关闭、执行 SQL 指令这三类消息，代码如下所示：
    ```cpp
    else if (arr.at(0) == "db") {
        CefRefPtr<CefListValue> msgBody = message->GetArgumentList();
        if (arr.at(1) == "open") {
            nlohmann::json param = nlohmann::json::parse(msgBody->GetString(0).ToString());
            std::string dbPath = param["dbPath"].get<std::string>();
            int rc = sqlite3_open(dbPath.c_str(), &db);
            CefRefPtr<CefProcessMessage> msg = CefProcessMessage::Create(messageName);
            CefRefPtr<CefListValue> msgArgs = msg->GetArgumentList();
            json result;
            result["success"] = rc == 0;
            msgArgs->SetString(0, result.dump());
            frame->SendProcessMessage(PID_RENDERER, msg);
        }
        else if(arr.at(1) == "close") {
            int rc = sqlite3_close(db);
            CefRefPtr<CefProcessMessage> msg = CefProcessMessage::Create(messageName);
            CefRefPtr<CefListValue> msgArgs = msg->GetArgumentList();
            json result;
            result["success"] = rc == 0;
            msgArgs->SetString(0, result.dump());
            frame->SendProcessMessage(PID_RENDERER, msg);
        }
        else if (arr.at(1) == "execute") {
            nlohmann::json param = nlohmann::json::parse(msgBody->GetString(0).ToString());
            std::string sqlStr = param["sql"].get<std::string>();
            CefPostTask(TID_FILE_BACKGROUND, base::BindOnce(&ExecuteSql, sqlStr, messageName, frame));
        }
    }
    ```
    - sqlite3_open 方法负责打开数据库，它的第一个参数是我们通过 JSON 传递过来的数据库路径，第二个参数就是我们前面声明的全局静态变量 db。这个方法返回一个整形的值，我们应该判断 rc 是否与 SQLITE_OK（值为 0）相等来处理异常。
    - sqlite3_close 方法负责关闭数据库，它只有一个参数就是我们前面声明的全局静态变量 db。
    - sqlite3_open 和 sqlite3_close 执行完成后，我们都向渲染进程发回了一个消息，这个消息仅包含一项数据 success，操作执行成功，它的值是 true，操作执行失败，它的值是 false。
    - 虽然打开和关闭 SQLite 数据库的工作只要调用一个 SQLite 的 API 就能完成，但执行 SQL 指令的工作就要复杂的多了，因为 SQL 指令非常丰富，接下来我们就看一下该如何在异步线程中处理 SQL 指令。

10. 处理 SQL 指令
- 我们把执行 SQL 指令的操作放在一个后台线程中完成，因为这项工作有可能会造成浏览器进程的阻塞。 执行 SQL 指令的方法如下所示：
    ```cpp
    void ExecuteSql(const std::string& sqlStr, const std::string& msgName, CefRefPtr<CefFrame> frame) {
        json result;
        result["data"] = {};
        const char* zTail = sqlStr.c_str();
        sqlite3_exec(db, "BEGIN TRANSACTION;", NULL, NULL, NULL);
        while (strlen(zTail) != 0) {
            sqlite3_stmt* stmt = NULL;
            //判断 prepareResult == SQLITE_OK并处理异常
            int prepareResult = sqlite3_prepare_v2(db, zTail, -1, &stmt, &zTail);
            int stepResult = sqlite3_step(stmt);
            while (stepResult == SQLITE_ROW) {
                json row;
                int columnCount = sqlite3_column_count(stmt);
                for (size_t i = 0; i < columnCount; i++) {
                    std::string columnName = sqlite3_column_name(stmt, i);
                    int type = sqlite3_column_type(stmt, i);
                    if (type == SQLITE_INTEGER) {
                        row[columnName] = sqlite3_column_int(stmt, i);
                    }
                    else if (type == SQLITE3_TEXT)
                    {
                        const unsigned char* val = sqlite3_column_text(stmt, i);
                        row[columnName] = reinterpret_cast<const char*>(val);
                    }
                    //这里只处理了两种数据类型是不足以满足生产条件的
                }
                result["data"].push_back(row);
                stepResult = sqlite3_step(stmt);
            }
            sqlite3_finalize(stmt);
        }
        sqlite3_exec(db, "END TRANSACTION;", NULL, NULL, NULL);
        CefRefPtr<CefProcessMessage> msg = CefProcessMessage::Create(msgName);
        CefRefPtr<CefListValue> msgArgs = msg->GetArgumentList();
        result["success"] = true;
        msgArgs->SetString(0, result.dump());
        frame->SendProcessMessage(PID_RENDERER, msg);
    }
    ```
    - 声明一个 JSON 对象 result，待 SQL 指令执行完成之后，把它返回给渲染进程，这个对象的 data 属性用于存储 SQL 指令执行后返回的数据（如果有的话）。
    - 把 std::string 类型的 SQL 语句转型为 char* 类型并存储在 zTail 变量中，用于传递给 SQLite 的接口，注意，这里的 SQL 语句可能是多条 SQL 指令组合而成的，SQL 指令之间必须以分号分割。
    - 使用 sqlite3_exec 方法执行 BEGIN TRANSACTION; 指令，开启 SQLite 的事务处理过程，当所有 SQL 指令处理完成之后，再使用 END TRANSACTION; 指令提交事务。如果处理 SQL 指令时发生了异常，应该使用 ROLLBACK; 指令回滚事务。对于处理多条 SQL 语句的场景，开启事务会大大提高 SQLite 的处理效率。
    - 使用 sqlite3_prepare_v2 方法编译 SQL 指令，zTail 参数就是我们前面传入的 SQL 语句，如果 zTail 中包含多条 SQL 指令，那么这个方法一次只能编译一条，编译完一条 SQL 指令之后，这条语句就会从 zTail 中移除，当 zTail 的长度为 0 时，就代表着所有的 SQL 指令都被编译完成了，所以我们使用 while (strlen(zTail) != 0) 来循环处理 SQL 语句中的指令（zTail 中只包含一条 SQL 指令的场景也能应对）。
    - stmt 是 SQL 指令编译完成之后生成的对象句柄，我们可以使用它获取 SQL 指令执行后的数据。
    - sqlite3_step 用于一步一步处理 SQL 指令的数据结果，如果是增、删、改类的指令，那么只要调用一次这个 API 就可以了，如果是查询类的指令，数据结果就可能会有多个，所以这里我们用 while (stepResult == SQLITE_ROW)循环语句来处理这类 SQL 指令（增、删、改类的指令执行完成后得到的 stepResult 不是 SQLITE_ROW 所以不会进入这个循环）。
    - 我们在循环体内处理查询指令返回的多行数据，每行数据都会被格式化成 json 对象（row），这些 json 对象最终以数组的形式被存储在 result["data"] 中。
    - 由于我们无法预知查询语句对应的表拥有什么样的表结构，所以我们使用 sqlite3_column_count 方法得到目标表一共有多少个列，使用 sqlite3_column_name 方法得到列名，使用 sqlite3_column_type 方法得到列的数据类型，然后利用这三个信息遍历每行数据中的所有属性，最终把这些属性的值存储到返回结果中。
    - 一行数据处理完之后，再调用 sqlite3_step 处理下一行数据，直到所有的数据结果都处理完为止。
    - 一条 SQL 指令执行完之后，我们要调用 sqlite3_finalize 方法释放 stmt 句柄，让它对应的内存空间为下一条 SQL 指令提供服务。
    - 当所有 SQL 指令执行完成后，把得到的数据结果发送给渲染进程。
- 虽然这个方法可以从容的应对绝大多数增、删、改、查类的 SQL 指令，但还是难以满足一些特殊的 SQLite 操作需求（比如支持中文的 SQLite 全文索引），遇到类似的需求，开发者应该考虑开放另外的接口供 JavaScript 调用。

---

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

---

###### 七、其他

1. 如何寻求帮助
- 翻阅官方示例代码：https://bitbucket.org/chromiumembedded/cef/src/master/
- 翻阅官方文档：https://magpcss.org/ceforum/apidocs3/
- 到官方论坛提问：http://www.magpcss.org/ceforum/
- 向 CEF 的作者提交 Issue：https://bitbucket.org/chromiumembedded/cef/issues?status=new&status=open

2. 操作系统的各种 API 地址
- Windows 操作系统 API：https://docs.microsoft.com/zh-cn/windows/win32/apiindex/windows-api-list
- Mac 操作系统 API：https://developer.apple.com/cn/documentation/
- 由于 Linux 各个发行版之间的差异较大，这里还是建议大家直接到各发行版的官网查阅 API 资料。

3. 成为一个桌面应用开发高手
- 掌握多线程知识，比如创建和运用多线程知识执行异步任务，使用线程锁控制异步任务的执行顺序等。
- 掌握本地数据库的分库、分表技术，这样可以应对在客户端电脑上存储海量数据的需求。
- 掌握多进程知识，这样可以让你的应用可以很好地和第三方应用进行交互、通信，是集成第三方应用的一个很好途径。
- 掌握各种日志记录技术，本地日志、服务端日志、业务日志、异常日志、用户操作日志等，日志是你提升产品质量的很好助手。
- 掌握版本控制知识，对于一个商业产品来说，你应该考虑如何灰度发布你的产品，如何进行 AB 测试，如何紧急情况下让客户端进行降级，而这些都是需要版本控制技术才能做到的。
- 掌握设计模式与架构相关的知识，比如分层设计、分模块设计、代理模式、单例模式等，对于一个复杂的业务系统来说，拥有这些知识，你就可以很好地控制复杂的业务，让更多的团队成员很好地分工协作。

4. 哪些技术值得关注
- Flutter Desktop。谷歌在技术上的持续性和投入力度都是令人钦佩的，虽然这项技术目前还不是很成熟，但非常值得关注。
- Qt。Qt 技术发展了很久，生态建设和成熟度都非常好，其内部也有 QWebEngin 模块与 CEF 框架类似，目前 Qt6.3.x 也逐渐成熟了，是 CEF 的强有力竞争者。
- Electron。Electron 杀手级应用非常多，而且有 Node.js 的加持，让很多前端从业者都杀入了桌面应用开发领域，是低成本开发桌面应用的不二之选。
