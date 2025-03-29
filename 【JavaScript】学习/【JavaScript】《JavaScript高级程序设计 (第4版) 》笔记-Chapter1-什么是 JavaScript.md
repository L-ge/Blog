###### 一、什么是 JavaScript

1. 虽然 JavaScript 和 ECMAScript（发音为“ek-ma-script”） 基本上是同义词，但 JavaScript 远远不限于 ECMA-262 所定义的那样。没错，完整的 JavaScript 实现包含以下几个部分。
    - 核心（ECMAScript）
    - 文档对象模型（DOM）
    - 浏览器对象模型（BOM）

2. 文档对象模型（DOM，Document Object Model）是一个应用编程接口（API），用于在 HTML 中使用扩展的 XML。DOM 将整个页面抽象为一组分层节点。HTML 或 XML 页面的每个组成部分都是一种节点，包含不同的数据。比如下面的 HTML 页面：
    ```
    <html> 
        <head> 
            <title>Sample Page</title> 
        </head> 
        <body> 
            <p> Hello World!</p> 
        </body> 
    </html> 
    ```
    - 这些代码通过 DOM 可以表示为一组分层节点。
    - DOM 通过创建表示文档的树，让开发者可以随心所欲地控制网页的内容和结构。使用 DOM API，可以轻松地删除、添加、替换、修改节点。

3. IE3 和 Netscape Navigator 3 提供了浏览器对象模型（BOM） API，用于支持访问和操作浏览器的窗口。使用 BOM，开发者可以操控浏览器显示页面之外的部分。

4. 总体来说，BOM 主要针对浏览器窗口和子窗口（frame），不过人们通常会把任何特定于浏览器的扩展都归在 BOM 的范畴内。比如，下面就是这样一些扩展：
    - 弹出新浏览器窗口的能力；
    - 移动、缩放和关闭浏览器窗口的能力；
    - navigator 对象，提供关于浏览器的详尽信息；
    - location 对象，提供浏览器加载页面的详尽信息；
    - screen 对象，提供关于用户屏幕分辨率的详尽信息；
    - performance 对象，提供浏览器内存占用、导航行为和时间统计的详尽信息；
    - 对 cookie 的支持；
    - 其他自定义对象，如 XMLHttpRequest 和 IE 的 ActiveXObject。

5. JavaScript 是一门用来与网页交互的脚本语言，包含以下三个组成部分。
    - ECMAScript：由 ECMA-262 定义并提供核心功能。
    - 文档对象模型（DOM）：提供与网页内容交互的方法和接口。
    - 浏览器对象模型（BOM）：提供与浏览器交互的方法和接口。

6. JavaScript 的这三个部分得到了五大 Web 浏览器（IE、Firefox、Chrome、Safari 和 Opera）不同程度的支持。所有浏览器基本上对 ES5（ECMAScript 5）提供了完善的支持，而对 ES6（ECMAScript 6）和 ES7（ECMAScript 7）的支持度也在不断提升。这些浏览器对 DOM 的支持各不相同，但对 Level 3 的支持日益趋于规范。HTML5 中收录的 BOM 会因浏览器而异，不过开发者仍然可以假定存在很大一部分公共特性。
