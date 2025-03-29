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
