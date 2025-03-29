###### 八、处理数据

1. 在 JavaScript 中，最常使用 fetch 发起 HTTP 请求。
    ```
    fetch(`https://api.github.com/users/moonhighway`)
        .then(response => response.json())
        .then(console.log)
        .catch(console.error);
    ```
    - fetch 函数返回一个 promise。
    - 这里，我们向指定的 URL 发起一个异步请求。
    - 传送完毕后，信息通过 .then(callback) 方法传给一个回调。
    - GitHub API 响应的数据是 JSON 格式，而且包含中 HTTP 响应的主体中，因此我们调用 response.json() 获取数据，把数据解析为 JSON 格式。
    - 获取响应信息后，把数据输出到控制台中。
    - 如果有什么地方出错了，把错误传给 console.error 方法。

2. promise 还可以使用 async/await 处理。由于 fetch 返回一个 promise，因此我们可以在 async 函数中异步等待（await）请求：
    ```
    async function requestGithubUser(githubLogin) {
        try {
            const response = await fetch(
                `https://api.github.com/users/${githubLogin}`
            );
            const userData = await response.json();
            console.log(userData);
        } catch (error) {
            console.error(error);
        }
    }

    requestGithubUser("moonhighway");
    ```
    - 这与前面串接 .then 函数处理请求的效果是完全一样的。
    - 在异步等待一个 promise 时，下一行代码直到 promise 得到解决后才执行。
    - 以这种方式在代码中处理 promise 比较简洁。

3. 一般来说，创建数据使用 POST 请求，修改数据使用 PUT 请求。fetch 函数的第二个参数接收一个选项对象，供 fetch 创建 HTTP 请求。
    ```
    fetch("/create/user", {
        method: "POST",
        body: JSON.stringify({ username, password, bio })
    })
    ```
    - JSON.stringify 是 JavaScript 中的一个函数，用于将一个 JavaScript 对象或值转换成一个 JSON 格式的字符串。
    - 这个请求使用 POST 方法创建一个新用户。username、password 和用户的 bio 以字符串的形式在请求的 body 中发送。

4. 上传文件需要使用一种不同的 HTTP 请求：multipart-formdata 请求。这种请求告诉服务器，请求的主体中有一个或多个文件。在 JavaScript 中发起这种请求，只需在请求的主体中传送一个 FormData 对象。
    ```
    const formData = new FormData();
    formData.append("username", "xx");
    formData.append("fullname", "yy");
    formData.append("avatar", imgFile);

    fetch("/create/user", {
        method: "POST",
        body: formData
    })
    ```
    - 通过一个 formData 对象随请求一起传送 username、fullname 和 avatar 图像。

5. 有时，我们要获得授权才能发起请求。通常，授权意味着获取个人或敏感数据。而且基本上要求用户通过 POST、PUT 或 DELETE 请求在服务器上执行一定的操作。

6. 一般情况下，我们在每个请求中添加一个唯一的令牌（token），供服务识别用户的身份。这个令牌通常在 Authorization 首部中添加。
    ```
    fetch(`https://api.github.com/users/${login}`, {
        method: "POST",
        headers: {
            Authorization: `Bearer ${token}`
        }
    });
    ```
    - 令牌通常在用户使用用户名和密码登录服务后获取。此外，也可以使用开放标准协议 OAuth 从第三方（例如 GitHub 或 Facebook）获取。

7. GitHub 可为你生成一个个人用户令牌。首先登录 GitHub，依次进入 Settings -> Developer Settings -> Personal Access Tokens。然后创建具有特定读写规则的令牌，最后使用令牌从 GitHub API 中获取个人信息。随 fetch 请求一起发送 Personal Access Tokens，GitHub 将提供更多有关账户的隐私信息。

8. 在 React 组件中获取数据要合理使用 useState 和 useEffect 钩子。前者的作用是把响应存储在状态中，后者的作用是发起 fetch 请求。
    ```
    import React, { useState, useEffect } from "react";

    function GitHubUser({ login }) {
        const [data, setData] = useState();

        useEffect(() => {
            if (!login) return;
            fetch(`https://api.github.com/users/${login}`)
                .then(response => response.json())
                .then(setData)
                .catch(console.error);
        }, [login]);

        if (data) 
            return <pre>{JSON.stringify(data, null, 2)}</pre>;

        return null;
    }

    export default function App() {
        return <GitHubUser login="moonhighway" />;
    }
    ```
    - 首次渲染时，由于 data 的初始值是 null，所以组件返回 null。组件返回 null 的意思是告诉 React，什么也不渲染。这样做没有问题，只是会看到一个黑屏。
    - JSON.stringify 方法接受三个参数：要转换成字符串的 JSON 数据，用于替换 JSON 对象中属性的函数，以及格式化数据时使用的空格数量。
    - 这里，我们把替换函数设为 null，即不做任何替换。2 表示格式化代码时使用的空格数量，即以两个空格缩进 JSON 字符串。
    - 使用 pre 元素的目的是保留空白，确保最终渲染出来的 JSON 字符串具有良好的阅读性。

9. 我们可以使用 Web Storage API 把数据存储在本地浏览器中。保存数据可以使用 window.localStorage 或 window.sessionStorage 对象。sessionStorage API 只把数据保存到用户会话中，关闭标签页或重启浏览器，保存的数据都将清空。而 localStorage 将无限期保存数据，除非主动删除。

10. JSON 数据应该以字符串的形式保存到浏览器存储空间中。这意味着，保存时要把对象转换成 JSON 对象，而在加载时则要把字符串解析为 JSON。下面是可用于保存和加载 JSON 数据的函数：
    ```
    const loadJSON = key => key && JSON.parse(localStorage.getItem(key));
    const saveJSON = (key, data) => localStorage.setItem(key, JSON.stringify(data));
    ```
    - 如果没有数据，loadJSON 函数返回 null。
    - 从 Web Storage 中加载数据、把数据保存到 Web Storage 中、把数据转换成字符串，以及解析 JSON 字符串，统统都是异步任务。loadJSON 和 saveJSON 都是异步函数。因此请注意，如果频繁调用这两个函数处理大量数据，可能会导致性能问题。为了性能，通常最好限制这两个函数的使用频率。

11. 例子：
    ```
    import React, { useState, useEffect } from "react";

    const loadJSON = key => key && JSON.parse(localStorage.getItem(key));
    const saveJSON = (key, data) => localStorage.setItem(key, JSON.stringify(data));

    function GitHubUser({ login }) {
        const [data, setData] = useState(loadJSON(`user:${login}`));

        useEffect(() => {
            if (!data) return;
            if (data.login === login) return;
            const { name, avatar_url, location } = data;
            saveJSON(`user:${login}`, {
                name,
                login,
                avatar_url,
                location
            });
        }, [data]);

        useEffect(() => {
            if (!login) return;
            if (data && data.login === login) return;
            fetch(`https://api.github.com/users/${login}`)
                .then(response => response.json())
                .then(setData)
                .catch(console.error);
        }, [login]);

        if (data) return <pre>{JSON.stringify(data, null, 2)}</pre>;

        return null;
    }

    export default function App() {
        return <GitHubUser login="moonhighway" />;
    }
    ```
    - loadJSON 是异步函数，因此调用 useState 时可以使用它设定状态数据的初始值。

12. 清空存储空间：
    ```
    localStorage.clear();
    ```

13. sessionStorage 和 localStorage 均是 Web 开发者的得力武器。
    - 离线时，我们可以使用本地数据，减少网络请求数，从而提升应用的性能。然而，一定要合理使用。实现离线存储增加了应用的复杂度，而且在开发的过程中很难处理。
    - 另外，请勿使用 Web Storage 缓存数据。如果发现性能有波动，可以试着让 HTTP 处理缓存。添加 Cache-Control: max-age=<EXP_DATE> 首部后，浏览器将自动缓存内容。EXP_DATE 定义内容的失效日期。

14. HTTP 请求和 promise 均有三种状态：待定、成功（完成）和失败（被拒）。发送请求后等待响应期间，请求处于待定状态。响应只有两种可能，成功或失败。成功的响应表示顺利连接上服务器，收到了数据。对 promise 来说，成功的响应表示 promise 得到解决了。倘若在请求的过程中有什么地方出错了，那就可以说 HTTP 请求失败，或者 promise 被拒了。在这两种情况下，我们将收到一个 error 对象，说明具体情况。

15. 发起 HTTP 请求时，这三种状态都要处理。
    ```
    import { useState, useEffect } from "react";

    function GitHubUser({ login }) {
        const [data, setData] = useState();
        const [error, setError] = useState();
        const [loading, setLoading] = useState(false);

        useEffect(() => {
            if (!login) return;
            setLoading(true);
            fetch(`https://api.github.com/users/${login}`)
                .then((data) => data.json())
                .then(setData)
                .then(() => setLoading(false))
                .catch(setError);
        }, [login]);

        if (loading) return <h1>loading...</h1>;
        if (error) return <pre>{JSON.stringify(error, null, 2)}</pre>;
        if (!data) return null;

        return (
            <div className="githubUser">
            <img src={data.avatar_url} alt={data.login} style={{ width: 200 }} />
            <div>
                <h1>{data.login}</h1>
                {data.name && <p>{data.name}</p>}
                {data.location && <p>{data.location}</p>}
            </div>
            </div>
        );
    }

    export default function App() {
        return <GitHubUser login="moonhighway" />;
    }
    ```
    - 请求处理待定状态时，我们可以显示一个“loading...”消息；如果出错了，那就渲染 error 对象中的详情。
    - 在生产环境中，我们可以进一步处理错误，比如说把跟踪到的错误记录到日志中，或者尝试再次发起请求。在开发环境中，只渲染错误详情没什么问题，反而能让开发者立即得到反馈。
    - 有些服务器会在成功的响应中发送额外的失败消息。

16. 在异步组件中，为了最大程度提升可重用性，经常利用渲染属性模式。采用这种模式创建组件，可以把开发应用所需的复杂机制或单调的样板代码抽离出来。
    ```
    import React from "react";

    const tahoe_peaks = [
        { name: "Freel Peak", elevation: 10891 },
        { name: "Monument Peak", elevation: 10067 },
        { name: "Pyramid Peak", elevation: 9983 },
        { name: "Mt. Tallac", elevation: 9735 }
    ];

    function List({ data = [], renderItem, renderEmpty }) {
        return !data.length ? (
            renderEmpty
        ) : (
            <ul>
            {data.map((item, i) => (
                <li key={i}>{renderItem(item)}</li>
            ))}
            </ul>
        );
    }

    export default function App() {
        return (
            <List
            data={tahoe_peaks}
            renderEmpty={<p>This list is empty</p>}
            renderItem={item => (
                <>
                {item.name} - {item.elevation.toLocaleString()}
                </>
            )}
            />
        );
    }
    ```

17. 生产环境中的应用通常要渲染大量数据，但是又不能一次性全部渲染。浏览器的渲染能力是有限的。渲染要耗费时间、占据处理能力及消耗内存，这三项都是有限制的。

18. 在用户滚动界面的过程中，我们要消除用户已经看过的结果，并渲染位于屏幕范围以外的新结果，随时准备展示。这种解决方案一次只渲染11个元素，其他数据排队等候，后面再渲染。这种技术称为虚拟化。使用这种技术滚动特大型列表，数据量无穷尽时，浏览器也不会崩溃。

19. 构建虚拟化列表组件要考虑很多事情。幸好，我们不用从头开始动手，社区已经开发了很多虚拟化列表组件，直接拿来使用即可。对浏览器渲染来说，最受欢迎的是 react-window 和 react-virtualized。虚拟化列表十分重要，React Native 甚至自带了一个这样的组件，即 FlatList。多数时候，我们无须自己动手构建虚拟化列表组件，知道如何使用就可以了。

20. 为了实现虚拟化列表，我们需要大量数据。这里指的是大量虚拟数据。
    ```
    npm i faker
    ```
    - 安装 faker 之后，我们可以创建一组大型的虚拟数据。
        ```
        import React from "react";
        import faker from "faker";

        const bigList = [...Array(5000)].map(() => ({
            name: faker.name.findName(),
            email: faker.internet.email(),
            avatar: faker.internet.avatar()
        }));

        function List({ data = [], renderItem, renderEmpty }) {
            return !data.length ? (
                renderEmpty
            ) : (
                <ul>
                {data.map((item, i) => (
                    <li key={i}>{renderItem(item)}</li>
                ))}
                </ul>
            );
        }

        export default function App() {
            const renderItem = item => (
                <div style={{ display: "flex" }}>
                <img src={item.avatar} alt={item.name} width={50} />
                <p>
                    {item.name} - {item.email}
                </p>
                </div>
            );

            return <List data={bigList} renderItem={renderItem} />;
        }
        ```
        - 我们映射一个有五千个空值的数组，把空值替换成虚拟用户的信息，创建 bigList 变量。

21. 下面使用 react-window 渲染这个虚拟用户列表：
    ```
    npm -i react-window
    ```
    - react-window 库提供了多个用于渲染虚拟列表的组件。
    - 下面的示例中，我们使用 react-window 提供的 FixedSizeList 组件：
        ```
        import React from "react";
        import { FixedSizeList } from "react-window";
        import faker from "faker";

        const bigList = [...Array(5000)].map(() => ({
            name: faker.name.findName(),
            email: faker.internet.email(),
            avatar: faker.internet.avatar()
        }));

        export default function App() {
            const renderRow = ({ index, style }) => (
                <div style={{ ...style, ...{ display: "flex" } }}>
                <img
                    src={bigList[index].avatar}
                    alt={bigList[index].name}
                    width={50}
                />
                <p>
                    {bigList[index].name} - {bigList[index].email}
                </p>
                </div>
            );

            return (
                <FixedSizeList
                    height={window.innerHeight}
                    width={window.innerWidth - 20}
                    itemCount={bigList.length}
                    itemSize={50}
                >
                    {renderRow}
                </FixedSizeList>
            );
        }
        ```
        - 通过 itemSize 属性指定每一行占据的像素数。
        - 渲染属性通过 children 属性传给 FixedSizeList。这种渲染属性模式时常用到。

22. 一个请求有三种状态：待定、成功或失败。为了重用 fetch 请求的这个逻辑，我们可以自定义一个钩子。在整个应用中，只要想发起 fetch 请求，就可以在组件中使用这个钩子。下面创建 useFetch 钩子：
    ```
    import React, { useState, useEffect } from "react";

    function useFetch(uri) {
        const [data, setData] = useState();
        const [error, setError] = useState();
        const [loading, setLoading] = useState(true);

        useEffect(() => {
            if (!uri) return;
            fetch(uri)
                .then(data => data.json())
                .then(setData)
                .then(() => setLoading(false))
                .catch(setError);
        }, [uri]);

        return {
            loading,
            data,
            error
        };
    }


    ```

23. 钩子最大的作用是跨组件重用的功能。有时，在不同的组件中需要重复渲染同样的内容。一个应用中，fetch 请求的错误处理方式或许也应该保持一致。下面创建一个 Fetch 组件：
    ```
    export default function Fetch({
        uri,
        renderSuccess,
        loadingFallback = <p>loading...</p>,
        renderError = error => <pre>{JSON.stringify(error, null, 2)}</pre>
    }) {
        const { loading, data, error } = useFetch(uri);
        if (loading) return loadingFallback;
        if (error) return renderError(error);
        if (data) return renderSuccess({ data });
    }

    // 如何使用 Fetch 组件如下：
    function GitHubUser({ login }) {
        return (
            <Fetch
            uri={`https://api.github.com/users/${login}`}
            renderSuccess={UserDetails}
            />
        );
    }

    function UserDetails({ data }) {
        return (
            <div className="githubUser">
            <img src={data.avatar_url} alt={data.login} style={{ width: 200 }} />
            <div>
                <h1>{data.login}</h1>
                {data.name && <p>{data.name}</p>}
                {data.location && <p>{data.location}</p>}
            </div>
            <UserRepositories
                login={data.login}
                onSelect={(repoName) => console.log(`${repoName} selected`)}
            />
            </div>
        );
    }
    ```
    - 自定义钩子 useFetch 是一层抽象，抽离了发起 fetch 请求的机制。Fetch 组件又是一层抽象，抽离了处理渲染什么的机制。

24. 下面创建一个名为 useIterator 的自定义钩子，迭代任意类型的对象数组：
    ```
    export const useIterator = (items = [], initialValue = 0) => {
        const [i, setIndex] = useState(initialValue);

        const prev = useCallback(() => {
            if (i === 0) return setIndex(items.length - 1);
            setIndex(i - 1);
        }, [i]);

        const next = useCallback(() => {
            if (i === items.length - 1) return setIndex(0);
            setIndex(i + 1);
        }, [i]);

        const item = useMemo(() => items[i], [i]);

        return [item || items[0], prev, next];
    };
    ```
    - 使用这个钩子可以遍历任何数组。由于这个钩子返回数组中的元素，因此我们可以利用数组析构为值提供有意义的名称：
        ```
        const [letter, previous, next] = useIterator([
            "a",
            "b",
            "c"
        ])
        ```
        - 这里，letter 的初始值是“a”。如果调用 next，组件将重新渲染，letter 的值变成“b”。再调用两次next，letter 的值又变成“a”，因为这个迭代器不会让 index 超出界限，绕了一圈又回到了数组的第一个元素。
    - 这里，prev 和 next 都使用 useCallback 钩子创建。这样可以确保 prev 函数始终保持不变，除非 i 的值发生变化。同样，item 的值也始终指向同一个元素对象，除非 i 的值发生变化。
    - 备忘这些值不会大幅提升性能，至少不足以让我们下决心增加代码的复杂度。然而，使用 useIterator 钩子时，备忘的值始终指向相同的对象和函数，这样方便比较值或者在依赖数组中使用。

25. 下面例子中，我们使用 useIterator 钩子，让用户遍览仓库列表：
    ```
    import React from "react";
    import { useIterator } from "./hooks";
    import RepositoryReadme from "./RepositoryReadme";

    export default function RepoMenu({ repositories, login }) {
        const [{ name }, previous, next] = useIterator(repositories);

        return (
            <>
            <div style={{ display: "flex" }}>
                <button onClick={previous}>&lt;</button>
                <p>{name}</p>
                <button onClick={next}>&gt;</button>
            </div>
            <RepositoryReadme login={login} repo={name} />
            </>
        );
    }
    ```
    - `&lt;` 是小于号的实体，显示一个小于号“<”。
    - 注意，使用数组析构可以为数组元素任意命名。即使在钩子中我们把函数命名为 prev 和 next，但是在使用钩子时，我们可以修改名称，改为 previous 和 next。
    - RepositoryReadme 组件在下面第 28 点有说明。

26. 瀑布式请求：一个接着一个地发起请求，前后请求之后有依赖关系，如果前一个请求出错了，后一个请求就不会再发起。

27. 仓库的 README 文件是使用 Markdown 编写，这是一种文本格式，可使用 ReactMarkdown 组件渲染为 HTML。安装 react-markdown 包：
    ```
    npm i react-markdown
    ```

28. 请求仓库的 README 文件内容也涉及瀑布式请求。首先，要向仓库 README 文件的路由发起数据请求。GitHub 对这个路由的响应是关于仓库 README 文件的详情，而不是文件内容。不过，响应中有个 download_url，我们可以通过它请求 README 文件的内容。可见，为了获取 Markdown 内容，要多发起一次请求。这里两个请求可以在同一个异步函数中发起：
    ```
    import React, { useState, useEffect, useCallback } from "react";

    import ReactMarkdown from "react-markdown";

    export default function RepositoryReadme({ repo, login }) {
        const [loading, setLoading] = useState(false);
        const [error, setError] = useState();
        const [markdown, setMarkdown] = useState("");

        const loadReadme = useCallback(async (login, repo) => {
            setLoading(true);
            const uri = `https://api.github.com/repos/${login}/${repo}/readme`;
            const { download_url } = await fetch(uri).then(res => res.json());
            const markdown = await fetch(download_url).then(res => res.text());
            setMarkdown(markdown);
            setLoading(false);
        }, []);

        useEffect(() => {
            if (!repo || !login) return;
            loadReadme(login, repo).catch(setError);
        }, [repo]);

        if (error) return <pre>{JSON.stringify(error, null, 2)}</pre>;
        if (loading) return <p>Loading...</p>;

        return <ReactMarkdown source={markdown} />;
    }
    ```
    - 使用 useCallback 钩子把 loadReadme 函数添加到组件中，在首次渲染组件时备忘该函数。

29. 所有请求都可在开发者工具的“Network”标签页中查看。在这个标签页中，你可以查看每一个请求，还可以限制网络速度，考察请求在较慢的网速下表现如何。

30. 在 Google Chrome 中如果想限制网络速度，单击“Online”旁边的箭头，在打开的菜单中，你可以选择不同的速度，选择“Fast 3G”和“Slow 3G”对网络请求的速度限制较大。另外，“Network”标签页还显示有全部 HTTP 请求的时间线。你可以筛选时间线，只查看“XHR”请求，即只显示 fetch 发出的请求。
    - 注意，加载图的标题是“Waterfall”。（读者笔记：除了名称、状态、类型、启动器、大小、时间之外，还可以像 Windows 任务管理器一样，勾选瀑布等其他列）

31. 有时候，可以一次发送全部请求，提升应用的速度。这种情况下，不再像瀑布式那样一个接一个发起请求，而是并行（或同时）发起所有请求。前面的例子之所以发送瀑布式请求，是因为组件是一个套一个渲染的，前一个组件未渲染之前不能发起下一个请求。如果在同一级渲染这三个组件，所有请求同时并行发起。

32. 我们并不能始终猜测出一开始要渲染什么数据。遇到这种情况，我们干脆不渲染组件，等到获得所需的数据再渲染。
    ```
    export default function App() {
        const [login, setLogin] = useState();
        const [repo, setRepo] = useState();
        return (
            <>
                <SearchForm value={login} onSearch={setLogin} />
                {login && <GitHubUser login={login} />}
                {login && (
                    <UserRepositoties
                        login={login}
                        repo={repo}
                        onSelect={setRepo}
                    />
                )}
                {login && repo && (
                    <RepositoryReadme login={login} repo={repo} />
                )}
            </SearchForm>
        );
    }
    ```
    - 这里，在所需的属性获得值之前，不渲染对应的组件。

33. 当 fetch 请求返回响应时，如果组件已经卸载完毕，试图修改已被卸载的组件的状态值会在控制台报错。
    - 只要用户在较慢的网速下加载数据，就有可能遇到这个错误。
    - 其实我们可以做些防护措施。首先，我们可以创建一个钩子，获知当前的组件有没有挂载：
        ```
        export function useMountedRef() {
            const mounted = useRef(false);
            useEffect(() => {
                mounted.current = true;
                return () => (mounted.current = false);
            });
            return mounted;
        }
        ```
        - 如果组件被卸载了，状态虽被清除了，但是 ref 还在。
        - 上面的 useEffect 没有依赖数组，每一次渲染组件都会调用，确保 ref 的值 true。只要卸载了组件，调用 useEffect 导致函数返回，ref 的值变成 false。
    - 使用例子：
        ```
        const mounted = useMountedRef();
        const loadReadme = useCallback(async (login, repo) => {
            setLoading(true);
            const uri = `https://api.github.com/repos/${login}/${repo}/readme`;
            const { download_url } = await fetch(uri).then(res => res.json());
            const markdown = await fetch(download_url).then(res => res.text());
            if (mounted.current) {
                setMarkdown(markdown);
                setLoading(false);
            }
        }, []);
        ```
        - 现在，我们可以通过一个 ref 判断组件有没有挂载。

34. React 为构建用户界面提供了一种声明式方案，GraphQL 为与 API 通信提供一种声明式方案。并行发起数据请求时，我们希望能同时获得所需的全部数据。GraphQL 就是为此而设计的。
    - 为了从 GraphQL API 获取数据，我们仍然要向指定的 URI 发起 HTTP 请求。不过，还要随请求一起发送查询。GraphQL 查询是对想要请求的数据的一种声明式描述。服务负责解析这个描述，把请求的所有数据打包进一个响应中。

35. 若想在 React 应用中使用 GraphQL，与之通信的后端服务要按照 GraphQL 规范构建。幸好，GitHub 也开放了 GraphQL API。多数 GraphQL 服务都提供有探索 GraphQL API 的方式。GitHub 提供的是 GraphQL Explorer（https://developer.github.com/v4/explorer）。

36. GraphQL 请求是在请求主体中包含查询的 HTTP 请求。我们可以使用 fetch 发起 GraphQL 请求。此外，也有一些专门的库和框架能辅助我们处理这类请求各方面的细节。
    - GraphQL 不限于 HTTP。GraphQL 是一个规范，规定如何通过网络发起数据请求。理论上，GraphQL 适用于任何网络协议。而且，GraphQL 不限定必须使用某种编程语言。

37. 安装 graphql-request 库：
    ```
    npm i graphql-request
    ```
