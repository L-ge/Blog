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
