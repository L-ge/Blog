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
