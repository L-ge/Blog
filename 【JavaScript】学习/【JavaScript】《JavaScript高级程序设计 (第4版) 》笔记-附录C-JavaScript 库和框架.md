###### 附录C、JavaScript 库和框架

1. JavaScript 库和框架
    - JavaScript 库帮助弥合浏览器之间的差异，能够简化浏览器复杂特性的使用。库主要分两种形式：通用和专用。通用 JavaScript 库支持常用的浏览器功能，可以作为网站或 Web 应用程序开发的基础。专用 JavaScript 库支持特定功能，只适合网站或 Web 应用程序的一部分。本附录会从整体上介绍这些库及其功能，并提供相关参考资源。

2. 框架
    - “框架”（framework）涵盖各种不同的模式，但各自具有不同的组织形式，用于搭建复杂应用程序。使用框架可以让代码遵循一致的约定，能够灵活扩展规模和复杂性。框架对常见的任务提供了稳健的实现机制，比如组件定义及重用、控制数据流、路由，等等。
    - JavaScript 框架越来越多地表现为单页应用程序（SPA，Single Page Application）。SPA 使用 HTML5浏览器历史 API，在只加载一个页面的情况下通过 URL 路由提供完整的应用程序用户界面。框架在应用程序运行期间负责管理应用程序的状态以及用户界面组件。大多数流行的 SPA 框架有坚实的开发者社区和大量第三方扩展。

3. React
    - React 是 Facebook 开发的框架，专注于模型视图控制器（MVC，Model-View-Controller）模型中的“视图”。专注的范围让它可以与其他框架或 React 扩展合作，实现 MVC 模式。React 使用单向数据流，是声明性和基于组件的，基于虚拟 DOM 高效渲染页面，提供了在 JavaScript 包含 HTML 标记的 JSX语法。Facebook 也维护了一个 React 的补充框架，叫作 Flux。
        - 许可：MIT

4. Angular
    - 谷歌在 2010 年首次发布的 Angular 是基于模型视图视图模型（MVVM）架构的全功能 Web 应用程序框架。2016 年，这个项目分叉为两个分支：Angular 1.x 和 Angular 2。前者是最初的 AngularJS 项目，后者则是基于 ES6 语法和 TypeScript 完全重新设计的框架。这两个版本的最新发布版都是指令和基于组件的实现，两个项目都有稳健的开发者社区和第三方扩展。
        - 许可：MIT

5. Vue
    - Vue 是类似 Angular 的全功能 Web 应用程序框架，但更加中立化。自 2014 年 Vue 发布以来，它的开发者社区发展迅猛，很多开发者因为其高性能和易组织，同时不过于主观而选择了 Vue。
        - 许可：MIT

6. Ember 
    - Ember 与 Angular 非常相似，都是 MVVM 架构，并使用首选的约定来构建 Web 应用程序。2015 年发布的 2.0 版引入了很多 React 框架的行为。
        - 许可：MIT

7. Meteor 
    - Meteor 与前面的框架都不一样，因为它是同构的 JavaScript 框架，这意味着客户端和服务器共享一套代码。Meteor 也使用实时数据更新协议，持续从 DB 向客户端推送新数据。虽然 Meteor 是一个极为主观的框架，但好处是可以使用其稳健的开箱即用特性快速开发应用程序。
        - 许可：MIT

8. Backbone.js 
    - Backbone.js 是构建于 Underscore.js 之上的一个最小化 MVC 开源库，为 SPA 做了大量优化，可以方便地更新应用程序状态。
        - 许可：MIT

9. 通用库
    - 通用 JavaScript 库提供适应任何需求的功能。所有通用库都致力于通过将常用功能封装为新 API，来补偿浏览器接口、弥补实现差异。其中有些 API 与原生功能相似，而另一些 API 则完全不同。通用库通常会提供与 DOM 的交互，对 Ajax 的支持，还有辅助常见任务的实用方法。

10. jQuery 
    - jQuery 是为 JavaScript 提供函数式编程接口的开源库。该库的核心是通过 CSS 选择符匹配 DOM 元素，通过调用链，jQuery 代码看起来更像描述故事情节而不是 JavaScript 代码。这种代码风格在设计师和原型设计者中非常流行。
        - 许可：MIT 或 GPL

11. Google Closure Library
    - Google Closure Library 是通用 JavaScript 工具包，与 jQuery 在很多方面都很像。这个库包含非常多的模块，涵盖底层操作和高层组件和部件。Google Closure Library 可以按需加载模块，并使用 Google Closure Compiler（附录 D 会介绍）构建。
        - 许可：Apache 2.0

12. Underscore.js 
    - Underscore.js 并不是严格意义上的通用库，但提供了 JavaScript 函数式编程的额外能力。它的文档将 Underscore.js 看成 jQuery 的组件，但提供了更多底层能力，用于操作对象、数组、函数和其他 JavaScript数据类型。
        - 许可：MIT

13. Lodash
    - 与 Underscore.js 一样，Lodash 也是实用库，用于扩充 JavaScript 工具包。Lodash 提供了很多操作原生类型，如数组、对象、函数和原始值的增强方法。
        - 许可：MIT

14. Prototype 
    - Prototype 是对常见 Web 开发任务提供简单 API 的开源库。Prototype 最初是为了 Ruby on Rails 开发者开发的，由类驱动，旨在为 JavaScript 提供类定义和继承。为此，Prototype 提供了大量的类，将常用和复杂的功能封装为简单的 API 调用。Prototype 包含在一个文件里，可以轻松地插入页面中使用。
        - 许可：MIT 及 CC BY-SA 3.0

15. Dojo Toolkit 
    - Dojo Toolkit 是以包系统为基础的开源库，将功能分门别类地划分为包，可以按需加载。Dojo 支持各种配置选项，几乎涵盖了使用 JavaScript 所需的一切。
        - 许可：“新”BSD 许可或 Academic Free License 2.1

16. MooTools 
    - MooTools 是简洁、优化的开源库，为原生 JavaScript 对象添加方法，在熟悉的接口上提供新功能。由于体积小、API 简单，MooTools 在 Web 开发者中很受欢迎。
        - 许可：MIT

17. qooxdoo
    - qooxdoo 是致力于全周期支持 Web 应用程序开发的开源库。通过实现自己的类和接口，qooxdoo 创建了类似传统面向对象编程语言的模型。这个库包含完整的 GUI 工具包和编译器，用于简化前端构建过程。qooxdoo 最初是网站托管公司 1&1 的内部库，后来基于开源许可对外发布。
        - 许可：LGPL 或 EPL

18. 动画与特效
    - 动画与特效是 Web 开发中越来越重要的一部分。在网站中创造流畅的动画并不容易。为此，不少库开发者已开发了包含各种动画和特效的库。前面提到的不少 JavaScript 库也包含动画特性。

19. D3 
    - 数据驱动文档（D3，Data Driven Documents）是非常流行的动画库，也是今天非常稳健和强大的JavaScript 数据可视化工具。D3 提供了全面完整的特性，涵盖 canvas、SVG、CSS 和 HTML5 可视化。使用 D3 可以极为精准地控制最终渲染的输出。
        - 许可：BSD

20. three.js 
    - three.js 是当前非常流行的 WebGL 库。它提供了轻量级 API，可以实现复杂 3D 渲染与动效。
        - 许可：MIT

21. moo.fx 
    - moo.fx 是基于 Prototype 或 MooTools 使用的开源动画库。它的目标是尽可能小（最新版 3KB），并使开发者只写尽可能少的代码。moo.fx 默认包含 MooTools，也可以单独下载，与 Prototype 一起使用。
        - 许可：MIT

22. Lightbox 
    - Lightbox 是创建简单图像覆盖特效的 JavaScript 库，依赖 Prototype 和 script.aculo.us 实现特效。其基本思想是可以使用户在当前页面的一个覆盖层中查看一个图像或多个图像。可以自定义覆盖层的外观和过渡。
        - 许可：Creative Commons Attribution 2.5
