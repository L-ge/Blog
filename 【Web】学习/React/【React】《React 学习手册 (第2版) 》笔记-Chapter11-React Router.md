###### 十一、React Router

1. 单页应用的一切内容都在同一个页面中，由 JavaScript 负责加载信息和变换 UI。如果没有路由方案，浏览器的很多功能，例如历史记录、收藏夹及前进和后退按钮都无法使用。路由为客户端请求定义端点，这些端点兼容浏览器的地址和历史记录对象。端点用于标识请求的内容，以便 JavaScript 加载和渲染相应的用户界面。

2. 路由器的作用是为网站中的各部分设置路由。一个路由就是一个端点，即可在浏览器的地址栏中输入的地址。请求某个路由，应用将渲染相应的内容。

3. 使用 React Router，我们要安装 React Router 和 React Router DOM。React Router DOM 供使用 DOM 的常规 React 应用使用。如果你编写的是 React Native 应用，要使用 react-router-native。

4. 现在，我们不再渲染 App 组件，而是渲染 Router 组件。Router 组件把当前地址的有关信息传给内部嵌套的子组件。Router 组件只能使用一次，而且应该放在组件树中靠近根的位置上：
    ```
    import React from "react";
    import { render } from "react-dom";
    import App from "./App";

    import { BrowserRouter as Router } from "react-router-dom";

    render(
        <Router>
            <App />
        </Router>,
        document.getElementById("root")
    );
    ```

5. 接下来要配置路由。我们把路由配置放在 App.js 文件中。要渲染的各个路由嵌套在名为 Routes 的组件中。在 Routes 中，一个页面使用一个 Route 组件设置。
    ```
    import React from "react";
    import {
        Routes,
        Route,
        Redirect
    } from "react-router-dom";
    import {
        Home,
        About,
        Events,
        Products,
        Contact,
        Whoops404,
        Services,
        History,
        Location
    } from "./pages";

    function App() {
        return (
            <div>
                <Routes>
                    <Route path="/" element={<Home />} />
                    <Route path="about" element={<About />}>
                        <Route
                            path="services"
                            element={<Services />}
                        />
                        <Route
                            path="history"
                            element={<History />}
                        />
                        <Route
                            path="location"
                            element={<Location />}
                        />
                    </Route>
                    <Route
                        path="events"
                        element={<Events />}
                    />
                    <Route
                        path="products"
                        element={<Products />}
                    />
                    <Route
                        path="contact"
                        element={<Contact />}
                    />
                    <Redirect
                        from="services"
                        to="about/services"
                    />
                    <Route path="*" element={<Whoops404 />} />
                </Routes>
            </div>
        );
    }

    export default App;
    ```
    - 这些路由告诉 React Router 在地址发送变化后渲染什么组件。每个 Route 组件都 path 和 element 属性。当浏览器中的地址匹配某个 path 时，显示对应的 element。
    - 可以在 Route 组件中嵌套 Route 组件（比如 About）。
    - 可以利用 Redirect 组件进行重定向（比如把访问 http://localhost:3000/services 的用户重定向到正确的路由 http://localhost:3000/about/services 上）。

6. 我们可以使用 react-router-dom 提供的 Link 组件创建链接。下面来修改首页，加入指向各个路由的导航链接目录：
    ```
    import { Link } from "react-router-dom";

    export function Home() {
        return (
            <div>
                <h1>[Company Website]</h1>
                <nav>
                    <Link to="about">About</Link>
                    <Link to="events">Events</Link>
                    <Link to="products">Products</Link>
                    <Link to="contact">Contact Us</Link>
                </nav>
            </div>
        );
    }
    ```

7. 我们可以使用当前地址值显示用户访问的路由：
    ```
    import { useLocation } from "react-router-dom";

    export function Whoops404() {
        let location = useLocation();
        console.log(location);
        return (
            <div>
                <h1>
                    Resource not found at {location.pathname}
                </h1>
            </div>
        );
    }
    ```

8. Router 组件只能使用一次，要路由的组件都嵌套在这个组件中。所有 Route 组件都嵌套在 Routes 组件中，根据浏览器中的当前地址选择渲染哪个组件。我们可以 Link 组件实现导航链接。

9. Route 组件只在匹配特定的 URL 时显示对应的内容。利用这个功能可以按照严格的层次结构组织 Web 应用中的不同页面，促进内容重用。

10. 嵌套路由时，若想显示嵌套内的组件（比如上面的 History），我们要使用 React Router DOM 的另一个特性：Outlet 组件。嵌套组件就使用 Outlet 组件渲染。想渲染子页面内容使用这个组件即可。
    ```
    import { Outlet } from "react-router-dom";

    export function About() {
        return (
            <div>
                <h1>[About]</h1>
                <Outlet />
            </div>
        );
    }

    export function History() {
        return (
            <section>
                <h2>Our History</h2>
                <p>
                    Lorem ipsum dolor sit amet, consectetur
                    adipiscing elit. Integer nec odio.
                    Praesent libero. Sed cursus ante dapibus
                    diam. Sed nisi. Nulla quis sem at nibh
                    elementum imperdiet. Duis sagittis ipsum.
                    Praesent mauris. Fusce nec tellus sed
                    augue semper porta. Mauris massa.
                    Vestibulum lacinia arcu eget nulla. Class
                    aptent taciti sociosqu ad litora torquent
                    per conubia nostra, per inceptos
                    himenaeos. Curabitur sodales ligula in
                    libero.
                </p>
            </section>
        );
    }
    ```
    - 现在，“About”页面都将重用 About 组件，分别显示各个嵌套的组件。应用通过地址判断该显示哪个子部分。例如，访问 http://localhost:3000/about/history 地址，渲染的是 About 组件内部的 History 组件。

11. Redirect 组件的作用是把用户重定向到指定的路由。

12. 也可以使用 useRoutes 钩子配置应用的路由。
    ```
    import React from "react";
    import { useRoutes } from "react-router-dom";
    import {
        Home,
        About,
        Events,
        Products,
        Contact,
        Whoops404,
        Services,
        History,
        Location
    } from "./pages";

    function App() {
        let element = useRoutes([
            { path: "/", element: <Home /> },
            {
                path: "about",
                element: <About />,
                children: [
                    {
                        path: "services",
                        element: <Services />
                    },
                    { path: "history", element: <History /> },
                    {
                        path: "location",
                        element: <Location />
                    }
                ]
            },
            { path: "events", element: <Events /> },
            { path: "products", element: <Products /> },
            { path: "contact", element: <Contact /> },
            { path: "*", element: <Whoops404 /> },
            {
                path: "services",
                redirectTo: "about/services"
            }
        ]);
        return element;
    }
    ```
    - Route 是对 useRoutes 的包装，其实使用哪种句法都可以。

13. React Router 的另一个有用特性是为路由设置参数。路由参数是一种变量，作用是从 URL 中获取相应的值。路由参数在数据驱动型 Web 应用中特别有用，可用于筛选内容或管理显示偏好设置。下面举个例子：
    - 首先，打开 index.js 文件，设置路由器。
        ```
        import React from "react";
        import { render } from "react-dom";
        import { BrowserRouter as Router } from "react-router-dom";
        import App from "./Colors";

        render(
            <Router>
                <App />
            </Router>,
            document.getElementById("root")
        );
        ```
        - 嵌套 App 组件之后，路由的所有属性都将传给它及它的子组件。
    - 然后，在 App 中添加路由。
        ```
        import React from "react";
        import { Routes, Route } from "react-router-dom";
        import AddColorForm from "./AddColorForm";
        import ColorList from "./ColorList";
        import { ColorDetails } from "./ColorDetails";
        import "./Colors.css";
        import { ColorProvider } from "./hooks";
        export * from "./hooks";

        export default function App() {
            return (
                <ColorProvider>
                    <AddColorForm />
                    <Routes>
                        <Route
                            path="/"
                            element={<ColorList />}
                        ></Route>
                        <Route
                            path=":id"
                            element={<ColorDetails />}
                        />
                    </Routes>
                </ColorProvider>
            );
        }
        ```
    - ColorDetails 组件根据颜色的 id 动态显示内容。例如，localhost:3000/00fdb4c5-c5bd-4087-a48f-4ff7a9d90af8。
        ```
        import React from "react";
        import { useColors } from "./";
        import { useParams } from "react-router-dom";

        export function ColorDetails() {
            let { id } = useParams();
            let { colors } = useColors();

            let foundColor = colors.find(
                color => color.id === id
            );

            return (
                <div>
                <div
                    style={{
                    backgroundColor: foundColor.color,
                    height: 100,
                    width: 100
                    }}
                ></div>
                <h1>{foundColor.title}</h1>
                <h1>{foundColor.color}</h1>
                </div>
            );
        }
        ```
14. 我们想为这个颜色管理应用添加的另一个功能是，点击颜色列表中的某个颜色后打开 ColorDetails 页面。下面在 Colors 组件中添加这个功能。为此，我们要使用另一个路由器钩子 useNavigate，在用户点击该组件时打开详情页面。
    - 先从 react-router-dom 中导入这个钩子：
        ```
        import { useNavigate } from "react-router-dom";
        ```
    - 然后，调用 useNavigate 钩子，返回一个函数，导航到另一个页面：
        ```
        let navigate = useNavigate();
        ```
    - 接下来，为 section 元素添加一个 onClick 处理函数，根据颜色的 id 导航到正确的路由：
        ```
        let navigate = useNavigate();

        return (
            <section
                className="color"
                onClick={() => navigate(`/${id}`)}
            >
                // Color 组件
            </section>
        );
        ```
    - 现在，单击这个 section 元素就可以转到正确的页面了。

15. 路由参数是获取影响用户界面呈现内容所需数据的理想工具。然而，我们应该只在希望用户从 URL 中捕获这些信息时才使用。
