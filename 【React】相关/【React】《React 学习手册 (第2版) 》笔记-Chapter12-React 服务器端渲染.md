###### 十二、React 服务器端渲染

1. 我们可以采用同构方式渲染 React，以便支持浏览器以外的平台。这意味着，我们可以在服务器端渲染 UI，然后再发给浏览器。借助服务器端渲染，可以提升性能、增进可移植性、提高安全性。

2. 同构（isomorphic）和普适（universal）这两个术语通常都指在客户端和服务器端都可以运行的应用。虽然二者往往可以互换，但还是有点儿区别的。同构应用指可在多个平台上渲染的应用，而普适代码指同一套代码可在多个环境中运行。

3. 有了 Node.js，我们可以在其他应用，例如服务器端应用、CLI 应用，甚至是原生应用中重用为浏览器编写的代码。

4. 不是所有 JavaScript 代码不经一点改动就能在两端运行。
    - Node.js 和浏览器不同，没有内置 fetch 函数。
    - 在 Node.js 中，我们可以使用 npm 中的 isomorphic-fetch，也可以使用内置的 https 模块。
        ```
        // 既然我们已经使用了 fetch 句法，那就用 isomorphic-fetch：
        npm install isomorphic-fetch
        ```
    - 然后，只需导入 isomorphic-fetch，其他代码无需改动：
        ```
        const fetch = require("isomorphic-fetch");

        const userDetails = response => {
            const login = response.login;
            console.log(login);
        };

        fetch("https://api.github.com/users/moonhighway")
            .then(res => res.json())
            .then(userDetails);
        ```

5. JSX 会编译成 JavaScript。Star 组件就是一个函数而已。这个组件可以直接在浏览器渲染，也可以捕获输出的 HTML 字符串，在其他环境中渲染。ReactDOM 提供的 renderToString 方法可以把 UI 渲染为 HTML 字符串：
    ```
    // 在浏览器中直接渲染 HTML
    ReactDOM.render(<Star />);

    // 把 HTML 渲染为字符串
    let html = ReactDOM.renderToString(<Star />);
    ```

6. 我们可以构建同构应用，在不同的平台中渲染组件，还可以通过一定的架构方式，跨多个平台全局重用 JavaScript 代码。另外，使用其他编程语言，例如 Go 或 Python 也可以构建同构应用，不限于只能使用 Node.js。

7. 使用 ReactDOM.renderToString 方法可以在服务器端渲染 UI。

8. 构建应用时，客户端渲染通常是首选方案。我们伺服 Create React App 生成的 build 文件夹，在浏览器中运行 HTML，通过 script.js 文件加载所需的 JavaScript 代码。
    - 但是这个方案有点儿耗时。用户可能要等一会儿才能看到内容加载出来，具体等待事件取决于网络速度。
    - Create React App 搭配 Express 服务器可以实现一种混合客户端和服务端渲染的体验。

9. 改动客户端的代码，首先要做的改动是，把 ReactDOM.render 换成 ReactDOM.hydrate。
    - 这两个函数的作用相同，只不过 hydrate 把内容添加到一个由 ReactDOMServer 渲染的容器中。操作顺序是这样的：
        1. 渲染应用的静态版本，让用户知道应用在做事，“加载了”页面。
        2. 请求动态 JavaScript 代码。
        3. 把静态内容替换为动态内容。
        4. 用户点击，交互正常。
    - 这其实就是在服务端渲染之后把内容再输送给应用。再输送是指先加载静态 HTML 内容，然后加载 JavaScript。这样做能让用户在感官上体验到性能的顺畅。

10. 接下来要设置项目的服务器。我们将使用轻量级 Node 服务器 Express。
    - 先安装：
        ```
        npm install express
        ```
    - 然后，创建一个服务器文件夹，命名为 server，再新建一个文件 index.js，保存在该文件夹中。我们在这个文件中构建一个服务器，让它伺服 build 文件夹，另外预先加载静态 HTML 内容。
        ```
        import express from "express";
        const app = express();

        app.use(express.static("./build"));
        ```
        - 这段代码导入服务器，静态伺服 build 文件夹。
    - 接着，使用 ReactDOM 提供的 renderToString 函数，以静态 HTML 字符串的形式渲染应用：
        ```
        import React from "react"; 
        import ReactDOMServer from "react-dom/server";
        import { Menu } from "../src/Menu.js";

        const PORT = process.env.PORT || 4000;

        app.get("/*", (req, res) => {
            const app = ReactDOMServer.renderToString(
                <Menu />
            );

            const indexFile = path.resolve(
                "./build/index.html"
            );

            fs.readFile(indexFile, "utf8", (err, data) => {
                return res.send(
                data.replace(
                    '<div id="root"></div>',
                    `<div id="root">${app}</div>`
                )
                );
            });
        });

        app.listen(PORT, () =>
            console.log(
                `Server is listening on port ${PORT}`
            )
        );
        ```
        - 我们要把 Menu 组件传给 renderToString 函数，因为这是我们想静态渲染的组件。
        - 然后，我们要从构建好的客户端应用中读取静态的 index.html 文件，把应用的内容写入 div 元素，并把这部分内容作为请求的响应。

11. 完成以上操作后，我们要配置 webpack 和 Babel。虽然 Create React App 自身就能编译和构建，但是我们要为服务器端设置和实施一些不同的规则。
    - 首先安装几个依赖：
        ```
        npm install @babel/core @babel/preset-env babel-loader nodemon npm-run-all webpack webpack-cli webpack-node-externals
        ```
    - 安装好 Babel 之后，创建 .babelrc 文件，写入下述预设规则：
        ```
        {
            "presets": ["@babel/preset-env", "react-app"]
        }
        ```
    - 接下来，为服务器增加一个 webpack 配置文件，名为 webpack.server.js：
        ```
        const path = require("path");
        const nodeExternals = require("webpack-node-externals");

        module.exports = {
            entry: "./server/index.js",

            target: "node",

            externals: [nodeExternals()],

            output: {
                path: path.resolve("build-server"),
                filename: "index.js"
            },

            module: {
                rules: [
                    {
                        test: /\.js$/,
                        use: "babel-loader"
                    }
                ]
            }
        };
        ```
    - babel-loader 的作用是转换 JavaScript 文件，而 nodeExternals 则负责扫描 node_modules 文件夹，找到所有 node_modules 名称，构建一个外部函数，告诉 webpack 不要打包这些模块及其子模块。
    - 另外，你可能会遇到一个有关 webpack 的错误，这是由 Create React App 安装的版本与我们自己安装的版本发生冲突导致的。这个冲突的解决方法是，在项目根目录中创建 .env 文件，写入下述内容：
        ```
        SKIP_PREFLIGHT_CHECK=true
        ```
    - 最后，可以添加几个 npm 脚本，运行开发相关的命令：
        ```
        "scripts": {
            // ...
            "dev:build-server": "NODE_ENV=development webpack --config webpack.server.js --mode=development -w",
            "dev:start": "nodemon ./server-build/index.js",
            "dev": "npm-run-all --parallel build dev:*"
        },
        ```
        - dev:build-server：设置环境变量 development，使用新服务器配置运行 webpack。
        - dev:start：使用 nodemon 运行服务器文件，监听变动。
        - dev：并行运行前两个进程。
    - 现在，执行 npm run dev 命令，两个进程都会运行。你会看到应用运行在 localhost:4000 地址上。这个应用将按顺序加载内容，先预渲染 HTML，再渲染 JavaScript 构建包。

12. 在服务器端渲染生态环境中，还有一个使用广泛的强大工具，即 Next.js。Next.js 旨在帮助工程人员更轻松地编写服务器端渲染的应用。这个工具提供的功能有直观的路由、静态优化、自动分拆代码等。

13. 基于 React 的另一个受欢迎的网站生成工具是 Gatsby。Gatsby 为创建内容驱动型网站提供了一种简单直观的方式。Gatsby 的默认设置足够智能，性能、可访问性、图像处理等都不用我们自己操心。
    - Gatsby 可以创建的项目类型很多，不过通常用于构建内容驱动型网站。也就是说，对博客或静态内容网站而言，Gatsby 是个不错的选择，尤其是对 React 熟悉的人。此外，Gatsby 也可以处理动态内容，例如从 API 中加载数据、与框架集成等。

14. 如果你想构建移动应用，可以使用 React Native；如果你想使用声明式句法获取数据，可以使用 GraphQL；如果你想构建内容类网站，可以进一步探索 Next.js 和 Gatsby。
