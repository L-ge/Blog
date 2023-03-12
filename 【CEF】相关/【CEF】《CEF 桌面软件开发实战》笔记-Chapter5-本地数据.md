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
