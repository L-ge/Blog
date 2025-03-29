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
