###### 二十五、客户端存储

1. 客户端存储
    - 随着 Web 应用程序的出现，直接在客户端存储用户信息的需求也随之出现。这背后的想法是合理的：与特定用户相关的信息应该保存在用户的机器上。无论是登录信息、个人偏好，还是其他数据，Web 应用程序提供者都需要有办法把它们保存在客户端。对该问题的第一个解决方案就是 cookie，cookie由古老的网景公司发明，由一份名为 Persistent Client State: HTTP Cookies 的规范定义。今天，cookie 只是在客户端存储数据的一个选项。

2. cookie
    - HTTP cookie 通常也叫作 cookie，最初用于在客户端存储会话信息。这个规范要求服务器在响应HTTP 请求时，通过发送 Set-Cookie HTTP 头部包含会话信息。例如，下面是包含这个头部的一个 HTTP响应：
        ```
        HTTP/1.1 200 OK 
        Content-type: text/html 
        Set-Cookie: name=value 
        Other-header: other-header-value
        ```
    - 这个 HTTP 响应会设置一个名为"name"，值为"value"的 cookie。名和值在发送时都会经过 URL编码。浏览器会存储这些会话信息，并在之后的每个请求中都会通过 HTTP 头部 cookie 再将它们发回服务器，比如：
        ```
        GET /index.jsl HTTP/1.1 
        Cookie: name=value 
        Other-header: other-header-value
        ```
    - 这些发送回服务器的额外信息可用于唯一标识发送请求的客户端。

3. 限制
    - cookie 是与特定域绑定的。设置 cookie 后，它会与请求一起发送到创建它的域。这个限制能保证cookie 中存储的信息只对被认可的接收者开放，不被其他域访问。
    - 因为 cookie 存储在客户端机器上，所以为保证它不会被恶意利用，浏览器会施加限制。同时，cookie也不会占用太多磁盘空间。
    - 通常，只要遵守以下大致的限制，就不会在任何浏览器中碰到问题：
        - 不超过 300 个 cookie；
        - 每个 cookie 不超过 4096 字节；
        - 每个域不超过 20 个 cookie；
        - 每个域不超过 81 920 字节。
    - 每个域能设置的 cookie 总数也是受限的，但不同浏览器的限制不同。例如：
        - 最新版 IE 和 Edge 限制每个域不超过 50 个 cookie；
        - 最新版 Firefox 限制每个域不超过 150 个 cookie；
        - 最新版 Opera 限制每个域不超过 180 个 cookie；
        - Safari 和 Chrome 对每个域的 cookie 数没有硬性限制。
    - 如果 cookie 总数超过了单个域的上限，浏览器就会删除之前设置的 cookie。IE 和 Opera 会按照最近最少使用（LRU，Least Recently Used）原则删除之前的 cookie，以便为新设置的 cookie 腾出空间。Firefox好像会随机删除之前的 cookie，因此为避免不确定的结果，最好不要超出限制。
    - 浏览器也会限制 cookie 的大小。大多数浏览器对 cookie 的限制是不超过 4096 字节，上下可以有一个字节的误差。为跨浏览器兼容，最好保证 cookie 的大小不超过 4095 字节。这个大小限制适用于一个域的所有 cookie，而不是单个 cookie。
    - 如果创建的 cookie 超过最大限制，则该 cookie 会被静默删除。注意，一个字符通常会占 1 字节。如果使用多字节字符（如 UTF-8 Unicode 字符），则每个字符最多可能占 4 字节。

4. cookie 的构成
    - cookie 在浏览器中是由以下参数构成的。
        - 名称：唯一标识 cookie 的名称。cookie 名不区分大小写，因此 myCookie 和 MyCookie 是同一个名称。不过，实践中最好将 cookie 名当成区分大小写来对待，因为一些服务器软件可能这样对待它们。cookie 名必须经过 URL 编码。
        - 值：存储在 cookie 里的字符串值。这个值必须经过 URL 编码。
        - 域：cookie 有效的域。发送到这个域的所有请求都会包含对应的 cookie。这个值可能包含子域（如www.wrox.com），也可以不包含（如.wrox.com 表示对 wrox.com 的所有子域都有效）。如果不明确设置，则默认为设置 cookie 的域。
        - 路径：请求 URL 中包含这个路径才会把 cookie 发送到服务器。例如，可以指定 cookie 只能由http://www.wrox.com/books/访问，因此访问 http://www.wrox.com/下的页面就不会发送 cookie，即使请求的是同一个域。
        - 过期时间：表示何时删除 cookie 的时间戳（即什么时间之后就不发送到服务器了）。默认情况下，浏览器会话结束后会删除所有 cookie。不过，也可以设置删除 cookie 的时间。这个值是 GMT 格式（Wdy, DD-Mon-YYYY HH:MM:SS GMT），用于指定删除 cookie 的具体时间。这样即使关闭浏览器 cookie 也会保留在用户机器上。把过期时间设置为过去的时间会立即删除 cookie。
        - 安全标志：设置之后，只在使用 SSL 安全连接的情况下才会把 cookie 发送到服务器。例如，请求 https://www.wrox.com 会发送 cookie，而请求 http://www.wrox.com 则不会。
    - 这些参数在 Set-Cookie 头部中使用分号加空格隔开，比如：
        ```
        HTTP/1.1 200 OK 
        Content-type: text/html 
        Set-Cookie: name=value; expires=Mon, 22-Jan-07 07:10:24 GMT; domain=.wrox.com 
        Other-header: other-header-value
        ```
    - 这个头部设置一个名为"name"的 cookie，这个 cookie 在 2007 年 1 月 22 日 7:10:24 过期，对www.wrox.com 及其他 wrox.com 的子域（如 p2p.wrox.com）有效。
    - 安全标志 secure 是 cookie 中唯一的非名/值对，只需一个 secure 就可以了。比如：
        ```
        HTTP/1.1 200 OK 
        Content-type: text/html 
        Set-Cookie: name=value; domain=.wrox.com; path=/; secure 
        Other-header: other-header-value
        ```
    - 这里创建的 cookie 对所有 wrox.com 的子域及该域中的所有页面有效（通过 path=/指定）。不过，这个 cookie 只能在 SSL 连接上发送，因为设置了 secure 标志。
    - 要知道，域、路径、过期时间和 secure 标志用于告诉浏览器什么情况下应该在请求中包含 cookie。这些参数并不会随请求发送给服务器，实际发送的只有 cookie 的名/值对。

5. JavaScript 中的 cookie
    - 在 JavaScript 中处理 cookie 比较麻烦，因为接口过于简单，只有 BOM 的 document.cookie 属性。根据用法不同，该属性的表现迥异。要使用该属性获取值时，document.cookie 返回包含页面中所有有效 cookie 的字符串（根据域、路径、过期时间和安全设置），以分号分隔，如下面的例子所示：
        ```
        name1=value1;name2=value2;name3=value3
        ```
    - 所有名和值都是 URL 编码的，因此必须使用 decodeURIComponent()解码。
    - 在设置值时，可以通过 document.cookie 属性设置新的 cookie 字符串。这个字符串在被解析后会添加到原有 cookie 中。设置 document.cookie 不会覆盖之前存在的任何 cookie，除非设置了已有的cookie。设置 cookie 的格式如下，与 Set-Cookie 头部的格式一样：
        ```
        name=value; expires=expiration_time; path=domain_path; domain=domain_name; secure
        ```
    - 在所有这些参数中，只有 cookie 的名称和值是必需的。下面是个简单的例子：
        ```
        document.cookie = "name=Nicholas";
        ```
    - 这行代码会创建一个名为"name"的会话 cookie，其值为"Nicholas"。这个 cookie 在每次客户端向服务器发送请求时都会被带上，在浏览器关闭时就会被删除。虽然这样直接设置也可以，因为不需要在名称或值中编码任何字符，但最好还是使用 encodeURIComponent()对名称和值进行编码，比如：
        ```
        document.cookie = encodeURIComponent("name") + "=" + 
                            encodeURIComponent("Nicholas");
        ```
    - 要为创建的 cookie 指定额外的信息，只要像 Set-Cookie 头部一样直接在后面追加相同格式的字符串即可：
        ```
        document.cookie = encodeURIComponent("name") + "=" + 
                    encodeURIComponent("Nicholas") + "; domain=.wrox.com; path=/";
        ```
    - 因为在 JavaScript 中读写 cookie 不是很直观，所以可以通过辅助函数来简化相应的操作。与 cookie相关的基本操作有读、写和删除。这些在 CookieUtil 对象中表示如下：
        ```
        class CookieUtil { 
            static get(name) { 
                let cookieName = `${encodeURIComponent(name)}=`, 
                    cookieStart = document.cookie.indexOf(cookieName), 
                    cookieValue = null; 
                if (cookieStart > -1){ 
                    let cookieEnd = document.cookie.indexOf(";", cookieStart); 
                    if (cookieEnd == -1){ 
                        cookieEnd = document.cookie.length; 
                    } 
                    cookieValue = decodeURIComponent(document.cookie.substring(cookieStart 
                        + cookieName.length, cookieEnd)); 
                } 
                return cookieValue; 
            } 
            static set(name, value, expires, path, domain, secure) { 
                let cookieText = 
                    `${encodeURIComponent(name)}=${encodeURIComponent(value)}` 
                if (expires instanceof Date) { 
                    cookieText += `; expires=${expires.toGMTString()}`; 
                } 
                if (path) { 
                    cookieText += `; path=${path}`; 
                } 
                if (domain) { 
                    cookieText += `; domain=${domain}`; 
                } 
                if (secure) { 
                    cookieText += "; secure"; 
                } 
                document.cookie = cookieText; 
            } 
            static unset(name, path, domain, secure) { 
                CookieUtil.set(name, "", new Date(0), path, domain, secure); 
            } 
        };
        ```
    - CookieUtil.get()方法用于取得给定名称的 cookie 值。为此，需要在 document.cookie 返回的字符串中查找是否存在名称后面加上等号。如果找到了，则使用 indexOf()再查找该位置后面的分号（表示该 cookie 的末尾）。如果没有找到分号，说明这个 cookie 在字符串末尾，因此字符串剩余部分都是cookie 的值。取得 cookie 值后使用 decodeURIComponent()解码，然后返回。如果没有找到 cookie，则返回 null。
    - CookieUtil.set()方法用于设置页面上的 cookie，接收多个参数：cookie 名称、cookie 值、可选的 Date 对象（表示何时删除 cookie）、可选的 URL 路径、可选的域以及可选的布尔值（表示是否添加 secure 标志）。这些参数以它们的使用频率为序，只有前两个是必需的。在方法内部，使用了encodeURIComponent()对名称和值进行编码，然后再依次检查其他参数。如果 expires 参数是 Date对象，则使用 Date 对象的 toGMTString()方法添加一个 expires 选项来获得正确的日期格式。剩下的代码就是简单地追加 cookie 字符串，最终设置给 document.cookie。
    - 没有直接删除已有 cookie 的方法。为此，需要再次设置同名 cookie（包括相同路径、域和安全选项），但要将其过期时间设置为某个过去的时间。CookieUtil.unset()方法实现了这些处理。这个方法接收4 个参数：要删除 cookie 的名称、可选的路径、可选的域和可选的安全标志。
    - 这些参数会传给 CookieUtil.set()，将 cookie 值设置为空字符串，将过期时间设置为 1970 年1 月 1 日（以 0 毫秒初始化的 Date 对象的值）。这样可以保证删除 cookie。
    - 可以像下面这样使用这些方法：
        ```
        // 设置 cookie 
        CookieUtil.set("name", "Nicholas"); 
        CookieUtil.set("book", "Professional JavaScript"); 
        // 读取 cookie 
        alert(CookieUtil.get("name")); // "Nicholas" 
        alert(CookieUtil.get("book")); // "Professional JavaScript" 
        // 删除 cookie 
        CookieUtil.unset("name"); 
        CookieUtil.unset("book"); 
        // 设置有路径、域和过期时间的 cookie 
        CookieUtil.set("name", "Nicholas", "/books/projs/", "www.wrox.com", 
        new Date("January 1, 2010")); 
        // 删除刚刚设置的 cookie 
        CookieUtil.unset("name", "/books/projs/", "www.wrox.com"); 
        // 设置安全 cookie 
        CookieUtil.set("name", "Nicholas", null, null, null, true);
        ```
    - 这些方法通过处理解析和 cookie 字符串构建，简化了使用 cookie 存储数据的操作。

6. 子 cookie
    - 为绕过浏览器对每个域 cookie 数的限制，有些开发者提出了子 cookie 的概念。子 cookie 是在单个cookie 存储的小块数据，本质上是使用 cookie 的值在单个 cookie 中存储多个名/值对。最常用的子 cookie模式如下：
        ```
        name=name1=value1&name2=value2&name3=value3&name4=value4&name5=value5
        ```
    - 子 cookie 的格式类似于查询字符串。这些值可以存储为单个 cookie，而不用单独存储为自己的名/值对。结果就是网站或 Web 应用程序能够在单域 cookie 数限制下存储更多的结构化数据。
    - 要操作子 cookie，就需要再添加一些辅助方法。解析和序列化子 cookie 的方式不一样，且因为对子cookie 的使用而变得更复杂。比如，要取得某个子 cookie，就需要先取得 cookie，然后在解码值之前需要先像下面这样找到子 cookie：
        ```
        class SubCookieUtil { 
            static get(name, subName) { 
                let subCookies = SubCookieUtil.getAll(name); 
                return subCookies ? subCookies[subName] : null;
            } 
            static getAll(name) { 
                let cookieName = encodeURIComponent(name) + "=", 
                    cookieStart = document.cookie.indexOf(cookieName), 
                    cookieValue = null, 
                    cookieEnd, 
                    subCookies, 
                    parts, 
                    result = {}; 
                if (cookieStart > -1) { 
                    cookieEnd = document.cookie.indexOf(";", cookieStart); 
                    if (cookieEnd == -1) { 
                        cookieEnd = document.cookie.length; 
                    } 
                    cookieValue = document.cookie.substring(cookieStart + cookieName.length, cookieEnd); 
                    if (cookieValue.length > 0) { 
                        subCookies = cookieValue.split("&"); 
                        for (let i = 0, len = subCookies.length; i < len; i++) { 
                            parts = subCookies[i].split("="); 
                            result[decodeURIComponent(parts[0])] = decodeURIComponent(parts[1]); 
                        } 
                        return result; 
                    } 
                } 
                return null; 
            } 
            // 省略其他代码
        };
        ```
    - 取得子 cookie 有两个方法：get()和 getAll()。get()用于取得一个子 cookie 的值，getAll()用于取得所有子 cookie，并以对象形式返回，对象的属性是子 cookie 的名称，值是子 cookie 的值。get()方法接收两个参数：cookie 的名称和子 cookie 的名称。这个方法先调用 getAll()取得所有子 cookie，然后返回要取得的子 cookie（如果不存在则返回 null）。
    - SubCookieUtil.getAll()方法在解析 cookie 值方面与 CookieUtil.get()方法非常相似。不同的是 SubCookieUtil.getAll()方法不会立即解码 cookie 的值，而是先用和号（&）拆分，将所有子cookie 保存到数组。然后，再基于等号（=）拆分每个子 cookie，使 parts 数组的第一个元素是子 cookie的名称，第二个元素是子 cookie 的值。两个元素都使用 decodeURIComponent()解码，并添加到 result对象，最后返回 result 对象。如果 cookie 不存在则返回 null。
    - 可以像下面这样使用这些方法：
        ```
        // 假设 document.cookie=data=name=Nicholas&book=Professional%20JavaScript 
        // 取得所有子 cookie 
        let data = SubCookieUtil.getAll("data"); 
        alert(data.name); // "Nicholas" 
        alert(data.book); // "Professional JavaScript"
        // 取得个别子 cookie 
        alert(SubCookieUtil.get("data", "name")); // "Nicholas" 
        alert(SubCookieUtil.get("data", "book")); // "Professional JavaScript"
        ```
    - 要写入子 cookie，可以使用另外两个方法：set()和 setAll()。这两个方法的实现如下：
        ```
        class SubCookieUtil { 
            // 省略之前的代码
            static set(name, subName, value, expires, path, domain, secure) { 
                let subcookies = SubCookieUtil.getAll(name) || {}; 
                subcookies[subName] = value; 
                SubCookieUtil.setAll(name, subcookies, expires, path, domain, secure); 
            } 
            static setAll(name, subcookies, expires, path, domain, secure) { 
                let cookieText = encodeURIComponent(name) + "=", 
                    subcookieParts = new Array(), 
                    subName; 
                for (subName in subcookies){ 
                    if (subName.length > 0 && subcookies.hasOwnProperty(subName)){ 
                        subcookieParts.push( 
                            '${encodeURIComponent(subName)}=${encodeURIComponent(subcookies[subName])}'); 
                    } 
                } 
                if (cookieParts.length > 0) { 
                    cookieText += subcookieParts.join("&"); 
                    if (expires instanceof Date) { 
                        cookieText += `; expires=${expires.toGMTString()}`; 
                    } 
                    if (path) { 
                        cookieText += `; path=${path}`; 
                    } 
                    if (domain) { 
                        cookieText += `; domain=${domain}`; 
                    } 
                    if (secure) { 
                        cookieText += "; secure"; 
                    } 
                } else { 
                    cookieText += `; expires=${(new Date(0)).toGMTString()}`; 
                } 
                document.cookie = cookieText; 
            } 
            // 省略其他代码
        };
        ```
    - set()方法接收 7 个参数：cookie 的名称、子 cookie 的名称、子 cookie 的值、可选的 Date 对象用于设置 cookie 的过期时间、可选的 cookie 路径、可选的 cookie 域和可选的布尔值 secure 标志。所有可选的参数都作用于 cookie 本身，而不是子 cookie。为了在同一个 cookie 中存储多个子 cookie，路径、域和 secure 标志也必须相同。过期时间作用于整个 cookie，可以在写入个别子 cookie 时另行设置。在这个方法内部，第一步是取得给定 cookie 名称下包含的所有子 cookie。逻辑或操作符（||）在这里用于在 getAll()返回 null 的情况下将 subcookies 设置为新对象。然后，在 subcookies 上设置完子cookie 的值，再将参数传给 setAll()。
    - setAll()方法接收 6 个参数：cookie 的名称、包含所有子 cookie 的对象，然后是 set()方法中使用的 4 个可选参数。这个方法会在 for-in 循环中迭代第二个参数的属性。为保证只存储合适的数据，这里使用了 hasOwnProperty()方法确保只有实例属性才会序列化为子 cookie。因为存在属性名等于空字符串的可能，所以在添加到 subcookieParts 数组之前也要检查属性名的长度。subcookieParts数组包含了子 cookie 的名/值对，这样我们可以方便地使用 join()方法用和号将它们拼接成字符串。剩下的逻辑与 CookieUtil.set()一样。
    - 可以像下面这样使用这些方法：
        ```
        // 假设 document.cookie=data=name=Nicholas&book=Professional%20JavaScript 
        // 设置两个子 cookie 
        SubCookieUtil.set("data", "name", "Nicholas"); 
        SubCookieUtil.set("data", "book", "Professional JavaScript"); 
        // 设置所有子 cookie 并传入过期时间
        SubCookieUtil.setAll("data", { name: "Nicholas", book: "Professional JavaScript" }, new Date("January 1, 2010")); 
        // 修改"name"的值并修改整个 cookie 的过期时间
        SubCookieUtil.set("data", "name", "Michael", new Date("February 1, 2010"));
        ```
    - 最后一组子 cookie 相关的方法是要删除子 cookie 的。常规 cookie 可以通过直接设置过期时间为某个过去的时间删除，但删除子 cookie 没有这么简单。为了删除子 cookie，需要先取得所有子 cookie，把要删除的那个删掉，然后再把剩下的子 cookie 设置回去。下面是相关方法的实现：
        ```
        class SubCookieUtil { 
            // 省略之前的代码
            static unset(name, subName, path, domain, secure) { 
                let subcookies = SubCookieUtil.getAll(name); 
                if (subcookies){ 
                    delete subcookies[subName]; // 删除
                    SubCookieUtil.setAll(name, subcookies, null, path, domain, secure); 
                } 
            } 
            static unsetAll(name, path, domain, secure) { 
                SubCookieUtil.setAll(name, null, new Date(0), path, domain, secure); 
            } 
        }
        ```
    - 这里定义的这两个方法有两个不同的目的。unset()方法用于从 cookie 中删除一个子 cookie，其他子 cookie 不受影响；而 unsetAll()方法与 CookieUtil.unset()一样，会删除整个 cookie。与 set()和 setAll()一样，路径、域和 secure 标志必须与创建 cookie 时使用的一样。可以像下面这样使用这两个方法：
        ```
        // 只删除"name"子 cookie 
        SubCookieUtil.unset("data", "name"); 
        // 删除整个 cookie 
        SubCookieUtil.unsetAll("data");
        ```
    - 如果实际开发中担心碰到每个域的 cookie 限制，则可以考虑使用子 cookie 这个方案。此时要特别注意 cookie 的大小，不要超过对单个 cookie 大小的限制。

7. 使用 cookie 的注意事项
    - 还有一种叫作 HTTP-only 的 cookie。HTTP-only 可以在浏览器设置，也可以在服务器设置，但只能在服务器上读取，这是因为 JavaScript 无法取得这种 cookie 的值。
    - 因为所有 cookie 都会作为请求头部由浏览器发送给服务器，所以在 cookie 中保存大量信息可能会影响特定域浏览器请求的性能。保存的 cookie 越大，请求完成的时间就越长。即使浏览器对 cookie 大小有限制，最好还是尽可能只通过 cookie 保存必要信息，以避免性能问题。
    - 对 cookie 的限制及其特性决定了 cookie 并不是存储大量数据的理想方式。因此，其他客户端存储技术出现了。
    - 注意，不要在 cookie 中存储重要或敏感的信息。cookie 数据不是保存在安全的环境中，因此任何人都可能获得。应该避免把信用卡号或个人地址等信息保存在 cookie 中。

8. Web Storage
    - Web Storage 最早是网页超文本应用技术工作组（WHATWG，Web Hypertext Application Technical Working Group）在 Web Applications 1.0 规范中提出的。这个规范中的草案最终成为了 HTML5 的一部分，后来又独立成为自己的规范。Web Storage 的目的是解决通过客户端存储不需要频繁发送回服务器的数据时使用 cookie 的问题。
    - Web Storage 规范最新的版本是第 2 版，这一版规范主要有两个目标：
        - 提供在 cookie 之外的存储会话数据的途径；
        - 提供跨会话持久化存储大量数据的机制。
    - Web Storage 的第 2 版定义了两个对象：localStorage 和 sessionStorage。localStorage 是永久存储机制，sessionStorage 是跨会话的存储机制。这两种浏览器存储 API 提供了在浏览器中不受页面刷新影响而存储数据的两种方式。2009 年之后所有主要供应商发布的浏览器版本在 window 对象上支持 localStorage 和 sessionStorage。
    - 注意，Web Storage 第 1 版曾使用过 globalStorage，不过目前 globalStorage 已废弃。

9. Storage 类型
    - Storage 类型用于保存名/值对数据，直至存储空间上限（由浏览器决定）。Storage 的实例与其他对象一样，但增加了以下方法。
        - clear()：删除所有值；不在 Firefox 中实现。
        - getItem(name)：取得给定 name 的值。
        - key(index)：取得给定数值位置的名称。
        - removeItem(name)：删除给定 name 的名/值对。
        - setItem(name, value)：设置给定 name 的值。
    - getItem()、removeItem()和 setItem()方法可以直接或间接通过 Storage 对象调用。因为每个数据项都作为属性存储在该对象上，所以可以使用点或方括号操作符访问这些属性，通过同样的操作来设置值，也可以使用 delete 操作符删除属性。即便如此，通常还是建议使用方法而非属性来执行这些操作，以免意外重写某个已存在的对象成员。
    - 通过 length 属性可以确定 Storage 对象中保存了多少名/值对。我们无法确定对象中所有数据占用的空间大小，尽管 IE8 提供了 remainingSpace 属性，用于确定还有多少存储空间（以字节计）可用。
    - 注意，Storage 类型只能存储字符串。非字符串数据在存储之前会自动转换为字符串。注意，这种转换不能在获取数据时撤销。

10. sessionStorage 对象
    - sessionStorage 对象只存储会话数据，这意味着数据只会存储到浏览器关闭。这跟浏览器关闭时会消失的会话 cookie 类似。存储在 sessionStorage 中的数据不受页面刷新影响，可以在浏览器崩溃并重启后恢复。（取决于浏览器，Firefox 和 WebKit 支持，IE 不支持。）
    - 因为 sessionStorage 对象与服务器会话紧密相关，所以在运行本地文件时不能使用。存储在sessionStorage 对象中的数据只能由最初存储数据的页面使用，在多页应用程序中的用处有限。
    - 因为 sessionStorage 对象是 Storage 的实例，所以可以通过使用 setItem()方法或直接给属性赋值给它添加数据。下面是使用这两种方式的例子：
        ```
        // 使用方法存储数据
        sessionStorage.setItem("name", "Nicholas"); 
        // 使用属性存储数据
        sessionStorage.book = "Professional JavaScript";
        ```
    - 所有现代浏览器在实现存储写入时都使用了同步阻塞方式，因此数据会被立即提交到存储。具体API 的实现可能不会立即把数据写入磁盘（而是使用某种不同的物理存储），但这个区别在 JavaScript 层面是不可见的。通过 Web Storage 写入的任何数据都可以立即被读取。
    - 老版 IE 以异步方式实现了数据写入，因此给数据赋值的时间和数据写入磁盘的时间可能存在延迟。对于少量数据，这里的差别可以忽略不计，但对于大量数据，就可以注意到 IE 中 JavaScript 恢复执行的速度比其他浏览器更快。这是因为实际写入磁盘的进程被转移了。在 IE8 中可以在数据赋值前调用begin()、之后调用 commit()来强制将数据写入磁盘。比如：
        ```
        // 仅适用于 IE8 
        sessionStorage.begin(); 
        sessionStorage.name = "Nicholas"; 
        sessionStorage.book = "Professional JavaScript"; 
        sessionStorage.commit();
        ```
    - 以上代码确保了"name"和"book"在 commit()调用之后会立即写入磁盘。调用 begin()是为了保证在代码执行期间不会有写入磁盘的操作。对于少量数据，这个过程不是必要的，但对于较大的数据量，如文档，则可以考虑使用这种事务性方法。
    - 对存在于 sessionStorage 上的数据，可以使用 getItem()或直接访问属性名来取得。下面是使用这两种方式的例子：
        ```
        // 使用方法取得数据
        let name = sessionStorage.getItem("name"); 
        // 使用属性取得数据
        let book = sessionStorage.book;
        ```
    - 可以结合 sessionStorage 的 length 属性和 key()方法遍历所有的值：
        ```
        for (let i = 0, len = sessionStorage.length; i < len; i++){ 
            let key = sessionStorage.key(i); 
            let value = sessionStorage.getItem(key); 
            alert(`${key}=`${value}`); 
        }
        ```
    - 这里通过 key()先取得给定位置中的数据名称，然后使用该名称通过 getItem()取得值，可以依次访问 sessionStorage 中的名/值对。
    - 也可以使用 for-in 循环迭代 sessionStorage 的值：
        ```
        for (let key in sessionStorage){ 
            let value = sessionStorage.getItem(key); 
            alert(`${key}=${value}`); 
        }
        ```
    - 每次循环，key 都会被赋予 sessionStorage 中的一个名称；这里不会返回内置方法或 length属性。
    - 要从 sessionStorage 中删除数据，可以使用 delete 操作符直接删除对象属性，也可以使用removeItem()方法。下面是使用这两种方式的例子：
        ```
        // 使用 delete 删除值
        delete sessionStorage.name; 
        // 使用方法删除值
        sessionStorage.removeItem("book");
        ```
    - sessionStorage 对象应该主要用于存储只在会话期间有效的小块数据。如果需要跨会话持久存储数据，可以使用 globalStorage 或 localStorage。

11. localStorage 对象
    - 在修订的 HTML5 规范里，localStorage 对象取代了 globalStorage，作为在客户端持久存储数据的机制。要访问同一个 localStorage 对象，页面必须来自同一个域（子域不可以）、在相同的端口上使用相同的协议。
    - 因为 localStorage 是 Storage 的实例，所以可以像使用 sessionStorage 一样使用localStorage。比如下面这几个例子：
        ```
        // 使用方法存储数据
        localStorage.setItem("name", "Nicholas"); 
        // 使用属性存储数据
        localStorage.book = "Professional JavaScript"; 
        // 使用方法取得数据
        let name = localStorage.getItem("name"); 
        // 使用属性取得数据
        let book = localStorage.book;
        ```
    - 两种存储方法的区别在于，存储在 localStorage 中的数据会保留到通过 JavaScript 删除或者用户清除浏览器缓存。localStorage 数据不受页面刷新影响，也不会因关闭窗口、标签页或重新启动浏览器而丢失。

12. 存储事件
    - 每当 Storage 对象发生变化时，都会在文档上触发 storage 事件。使用属性或 setItem()设置值、使用 delete 或 removeItem()删除值，以及每次调用 clear()时都会触发这个事件。这个事件的事件对象有如下 4 个属性。
        - domain：存储变化对应的域。
        - key：被设置或删除的键。
        - newValue：键被设置的新值，若键被删除则为 null。
        - oldValue：键变化之前的值。
    - 可以使用如下代码监听 storage 事件：
        ```
        window.addEventListener("storage", 
            (event) => alert('Storage changed for ${event.domain}'));
        ```
    - 对于 sessionStorage 和 localStorage 上的任何更改都会触发 storage 事件，但 storage 事件不会区分这两者。

13. 限制
    - 与其他客户端数据存储方案一样，Web Storage 也有限制。具体的限制取决于特定的浏览器。一般来说，客户端数据的大小限制是按照每个源（协议、域和端口）来设置的，因此每个源有固定大小的数据存储空间。分析存储数据的页面的源可以加强这一限制。
    - 不同浏览器给 localStorage 和 sessionStorage 设置了不同的空间限制，但大多数会限制为每个源 5MB。关于每种媒介的新配额限制信息表，可以参考 web.dev 网站上的文章“Storage for the Web”。
    - 要了解关于 Web Storage 限制的更多信息，可以参考 dev-test.nemikor 网站的“Web Storage Support Test”页面。

14. IndexedDB
    - Indexed Database API 简称 IndexedDB，是浏览器中存储结构化数据的一个方案。IndexedDB 用于代替目前已废弃的 Web SQL Database API。IndexedDB 背后的思想是创造一套 API，方便 JavaScript 对象的存储和获取，同时也支持查询和搜索。
    - IndexedDB 的设计几乎完全是异步的。为此，大多数操作以请求的形式执行，这些请求会异步执行，产生成功的结果或错误。绝大多数 IndexedDB 操作要求添加 onerror 和 onsuccess 事件处理程序来确定输出。
    - 2017 年，新发布的主流浏览器（Chrome、Firefox、Opera、Safari）完全支持 IndexedDB。IE10/11和 Edge 浏览器部分支持 IndexedDB。

15. 数据库
    - IndexedDB 是类似于 MySQL 或 Web SQL Database 的数据库。与传统数据库最大的区别在于，IndexedDB 使用对象存储而不是表格保存数据。IndexedDB 数据库就是在一个公共命名空间下的一组对象存储，类似于 NoSQL 风格的实现。
    - 使用 IndexedDB 数据库的第一步是调用 indexedDB.open()方法，并给它传入一个要打开的数据库名称。如果给定名称的数据库已存在，则会发送一个打开它的请求；如果不存在，则会发送创建并打开这个数据库的请求。这个方法会返回 IDBRequest 的实例，可以在这个实例上添加 onerror 和onsuccess 事件处理程序。举例如下：
        ```
        let db, 
            request, 
            version = 1; 
        request = indexedDB.open("admin", version); 
        request.onerror = (event) => 
            alert(`Failed to open: ${event.target.errorCode}`); 
        request.onsuccess = (event) => { 
            db = event.target.result; 
        };
        ```
    - 以前，IndexedDB 使用 setVersion()方法指定版本号。这个方法目前已废弃。如前面代码所示，要在打开数据库的时候指定版本。这个版本号会被转换为一个 unsigned long long 数值，因此不要使用小数，而要使用整数。
    - 在两个事件处理程序中，event.target 都指向 request，因此使用哪个都可以。如果 onsuccess事件处理程序被调用，说明可以通过 event.target.result 访问数据库（IDBDatabase）实例了，这个实例会保存到 db 变量中。之后，所有与数据库相关的操作都要通过 db 对象本身来进行。如果打开数据库期间发生错误，event.target.errorCode 中就会存储表示问题的错误码。
    - 注意，以前，出错时会使用 IDBDatabaseException 表示 IndexedDB 发生的错误。目前它已被标准的 DOMExceptions 取代。

16. 对象存储
    - 建立了数据库连接之后，下一步就是使用对象存储。如果数据库版本与期待的不一致，那可能需要创建对象存储。不过，在创建对象存储前，有必要想一想要存储什么类型的数据。
    - 假设要存储包含用户名、密码等内容的用户记录。可以用如下对象来表示一条记录：
        ```
        let user = { 
            username: "007", 
            firstName: "James", 
            lastName: "Bond", 
            password: "foo" 
        };
        ```
    - 观察这个对象，可以很容易看出最适合作为对象存储键的 username 属性。用户名必须全局唯一，它也是大多数情况下访问数据的凭据。这个键很重要，因为创建对象存储时必须指定一个键。
    - 数据库的版本决定了数据库模式，包括数据库中的对象存储和这些对象存储的结构。如果数据库还不存在，open()操作会创建一个新数据库，然后触发 upgradeneeded 事件。可以为这个事件设置处理程序，并在处理程序中创建数据库模式。如果数据库存在，而你指定了一个升级版的版本号，则会立即触发 upgradeneeded 事件，因而可以在事件处理程序中更新数据库模式。
    - 下面的代码演示了为存储上述用户信息如何创建对象存储：
        ```
        request.onupgradeneeded = (event) => { 
            const db = event.target.result; 
            // 如果存在则删除当前 objectStore。测试的时候可以这样做
            // 但这样会在每次执行事件处理程序时删除已有数据
            if (db.objectStoreNames.contains("users")) { 
                db.deleteObjectStore("users"); 
            } 
            db.createObjectStore("users", { keyPath: "username" }); 
        };
        ```
    - 这里第二个参数的 keyPath 属性表示应该用作键的存储对象的属性名。

17. 事务
    - 创建了对象存储之后，剩下的所有操作都是通过事务完成的。事务要通过调用数据库对象的transaction()方法创建。任何时候，只要想要读取或修改数据，都要通过事务把所有修改操作组织起来。最简单的情况下，可以像下面这样创建事务：
        ```
        let transaction = db.transaction();
        ```
    - 如果不指定参数，则对数据库中所有的对象存储有只读权限。更具体的方式是指定一个或多个要访问的对象存储的名称：
        ```
        let transaction = db.transaction("users");
        ```
    - 这样可以确保在事务期间只加载 users 对象存储的信息。如果想要访问多个对象存储，可以给第一个参数传入一个字符串数组：
        ```
        let transaction = db.transaction(["users", "anotherStore"]);
        ```
    - 如前所述，每个事务都以只读方式访问数据。要修改访问模式，可以传入第二个参数。这个参数应该是下列三个字符串之一："readonly"、"readwrite"或"versionchange"。比如：
        ```
        let transaction = db.transaction("users", "readwrite");
        ```
    - 这样事务就可以对 users 对象存储读写了。
    - 有了事务的引用，就可以使用 objectStore()方法并传入对象存储的名称以访问特定的对象存储。然后，可以使用 add()和 put()方法添加和更新对象，使用 get()取得对象，使用 delete()删除对象，使用 clear()删除所有对象。其中，get()和 delete()方法都接收对象键作为参数，这 5 个方法都创建新的请求对象。来看下面的例子：
        ```
        const transaction = db.transaction("users"), 
            store = transaction.objectStore("users"), 
            request = store.get("007"); 
        request.onerror = (event) => alert("Did not get the object!"); 
        request.onsuccess = (event) => alert(event.target.result.firstName);
        ```
    - 因为一个事务可以完成任意多个请求，所以事务对象本身也有事件处理程序：onerror 和 oncomplete。这两个事件可以用来获取事务级的状态信息：
        ```
        transaction.onerror = (event) => { 
            // 整个事务被取消
        }; 
        transaction.oncomplete = (event) => { 
            // 整个事务成功完成
        };
        ```
    - 注意，不能通过 oncomplete 事件处理程序的 event 对象访问 get()请求返回的任何数据。因此，仍然需要通过这些请求的 onsuccess 事件处理程序来获取数据。

18. 插入对象
    - 拿到了对象存储的引用后，就可以使用 add()或 put()写入数据了。这两个方法都接收一个参数，即要存储的对象，并把对象保存到对象存储。这两个方法只在对象存储中已存在同名的键时有区别。这种情况下，add()会导致错误，而 put()会简单地重写该对象。更简单地说，可以把 add()想象成插入新值，而把 put()想象为更新值。因此第一次初始化对象存储时，可以这样做：
        ```
        // users 是一个用户数据的数组
        for (let user of users) { 
            store.add(user); 
        }
        ```
    - 每次调用 add()或 put()都会创建对象存储的新更新请求。如果想验证请求成功与否，可以把请求对象保存到一个变量，然后为它添加 onerror 和 onsuccess 事件处理程序：
        ```
        // users 是一个用户数据的数组
        let request, 
            requests = []; 
        for (let user of users) { 
            request = store.add(user); 
            request.onerror = () => { 
                // 处理错误
            }; 
            request.onsuccess = () => { 
                // 处理成功
            }; 
            requests.push(request); 
        }
        ```
    - 创建并填充了数据后，就可以查询对象存储了。

19. 通过游标查询
    - 使用事务可以通过一个已知键取得一条记录。如果想取得多条数据，则需要在事务中创建一个游标。游标是一个指向结果集的指针。与传统数据库查询不同，游标不会事先收集所有结果。相反，游标指向第一个结果，并在接到指令前不会主动查找下一条数据。
    - 需要在对象存储上调用 openCursor()方法创建游标。与其他 IndexedDB操作一样，openCursor()方法也返回一个请求，因此必须为它添加 onsuccess 和 onerror 事件处理程序。例如：
        ```
        const transaction = db.transaction("users"), 
            store = transaction.objectStore("users"), 
            request = store.openCursor(); 
        request.onsuccess = (event) => { 
            // 处理成功
        }; 
        request.onerror = (event) => { 
            // 处理错误
        };
        ```
    - 在调用 onsuccess 事件处理程序时，可以通过 event.target.result 访问对象存储中的下一条记录，这个属性中保存着 IDBCursor 的实例（有下一条记录时）或 null（没有记录时）。这个 IDBCursor实例有几个属性。
        - direction：字符串常量，表示游标的前进方向以及是否应该遍历所有重复的值。可能的值包括：NEXT("next")、NEXTUNIQUE("nextunique")、PREV("prev")、PREVUNIQUE("prevunique")。
        - key：对象的键。
        - value：实际的对象。
        - primaryKey：游标使用的键。可能是对象键或索引键（稍后讨论）。
    - 可以像下面这样取得一个结果：
        ```
        request.onsuccess = (event) => { 
            const cursor = event.target.result; 
            if (cursor) { // 永远要检查
                console.log(`Key: ${cursor.key}, Value: ${JSON.stringify(cursor.value)}`); 
            } 
        };
        ```
    - 注意，这个例子中的 cursor.value 保存着实际的对象。正因为如此，在显示它之前才需要使用JSON 来编码。
    - 游标可用于更新个别记录。update()方法使用指定的对象更新当前游标对应的值。与其他类似操作一样，调用 update()会创建一个新请求，因此如果想知道结果，需要为 onsuccess 和 onerror 赋值：
        ```
        request.onsuccess = (event) => { 
            const cursor = event.target.result; 
            let value, 
                updateRequest; 
            if (cursor) { // 永远要检查
                if (cursor.key == "foo") { 
                    value = cursor.value; // 取得当前对象
                    value.password = "magic!"; // 更新密码
                    updateRequest = cursor.update(value); // 请求保存更新后的对象
                    updateRequest.onsuccess = () => { 
                        // 处理成功
                    }; 
                    updateRequest.onerror = () => { 
                        // 处理错误
                    }; 
                } 
            } 
        };
        ```
    - 也可以调用 delelte()来删除游标位置的记录，与 update()一样，这也会创建一个请求：
        ```
        request.onsuccess = (event) => { 
            const cursor = event.target.result; 
            let value, 
                deleteRequest; 
            if (cursor) { // 永远要检查
                if (cursor.key == "foo") { 
                    deleteRequest = cursor.delete(); // 请求删除对象
                    deleteRequest.onsuccess = () => { 
                        // 处理成功
                    }; 
                    deleteRequest.onerror = () => { 
                        // 处理错误
                    }; 
                } 
            } 
        };
        ```
    - 如果事务没有修改对象存储的权限，update()和 delete()都会抛出错误。
    - 默认情况下，每个游标只会创建一个请求。要创建另一个请求，必须调用下列中的一个方法。
        - continue(key)：移动到结果集中的下一条记录。参数 key 是可选的。如果没有指定 key，游标就移动到下一条记录；如果指定了，则游标移动到指定的键。
        - advance(count)：游标向前移动指定的 count 条记录。
    - 这两个方法都会让游标重用相同的请求，因此也会重用 onsuccess 和 onerror 处理程序，直至不再需要。例如，下面的代码迭代了一个对象存储中的所有记录：
        ```
        request.onsuccess = (event) => { 
            const cursor = event.target.result; 
            if (cursor) { // 永远要检查
                console.log(`Key: ${cursor.key}, Value: ${JSON.stringify(cursor.value)}`); 
                cursor.continue(); // 移动到下一条记录
            } else { 
                console.log("Done!"); 
            } 
        };
        ```
    - 调用 cursor.continue()会触发另一个请求并再次调用 onsuccess 事件处理程序。在没有更多记录时，onsuccess 事件处理程序最后一次被调用，此时 event.target.result 等于 null。

20. 键范围
    - 使用游标会给人一种不太理想的感觉，因为获取数据的方式受到了限制。使用键范围（key range）可以让游标更容易管理。键范围对应 IDBKeyRange 的实例。有四种方式指定键范围，第一种是使用only()方法并传入想要获取的键：
        ```
        const onlyRange = IDBKeyRange.only("007");
        ```
    - 这个范围保证只获取键为"007"的值。使用这个范围创建的游标类似于直接访问对象存储并调用get("007")。
    - 第二种键范围可以定义结果集的下限。下限表示游标开始的位置。例如，下面的键范围保证游标从"007"这个键开始，直到最后：
        ```
        // 从"007"记录开始，直到最后
        const lowerRange = IDBKeyRange.lowerBound("007");
        ```
    - 如果想从"007"后面的记录开始，可以再传入第二个参数 true：
        ```
        // 从"007"的下一条记录开始，直到最后
        const lowerRange = IDBKeyRange.lowerBound("007", true);
        ```
    - 第三种键范围可以定义结果集的上限，通过调用 upperBound()方法可以指定游标不会越过的记录。下面的键范围保证游标从头开始并在到达键为"ace"的记录停止：
        ```
        // 从头开始，到"ace"记录为止
        const upperRange = IDBKeyRange.upperBound("ace");
        ```
    - 如果不想包含指定的键，可以在第二个参数传入 true：
        ```
        // 从头开始，到"ace"的前一条记录为止
        const upperRange = IDBKeyRange.upperBound("ace", true);
        ```
    - 要同时指定下限和上限，可以使用 bound()方法。这个方法接收四个参数：下限的键、上限的键、可选的布尔值表示是否跳过下限和可选的布尔值表示是否跳过上限。下面是几个例子：
        ```
        // 从"007"记录开始，到"ace"记录停止
        const boundRange = IDBKeyRange.bound("007", "ace"); 
        // 从"007"的下一条记录开始，到"ace"记录停止
        const boundRange = IDBKeyRange.bound("007", "ace", true); 
        // 从"007"的下一条记录开始，到"ace"的前一条记录停止
        const boundRange = IDBKeyRange.bound("007", "ace", true, true); 
        // 从"007"记录开始，到"ace"的前一条记录停止
        const boundRange = IDBKeyRange.bound("007", "ace", false, true);
        ```
    - 定义了范围之后，把它传给 openCursor()方法，就可以得到位于该范围内的游标：
        ```
        const store = db.transaction("users").objectStore("users"), 
            range = IDBKeyRange.bound("007", "ace"); 
            request = store.openCursor(range); 
        request.onsuccess = function(event){ 
            const cursor = event.target.result; 
            if (cursor) { // 永远要检查
                console.log(`Key: ${cursor.key}, Value: ${JSON.stringify(cursor.value)}`); 
                cursor.continue(); // 移动到下一条记录
            } else { 
                console.log("Done!"); 
            } 
        };
        ```
    - 这个例子只会输出从键为"007"的记录开始到键为"ace"的记录结束的对象，比上一节的例子要少。

21. 设置游标方向
    - openCursor()方法实际上可以接收两个参数，第一个是 IDBKeyRange 的实例，第二个是表示方向的字符串。通常，游标都是从对象存储的第一条记录开始，每次调用 continue()或 advance()都会向最后一条记录前进。这样的游标其默认方向为"next"。如果对象存储中有重复的记录，可能需要游标跳过那些重复的项。为此，可以给 openCursor()的第二个参数传入"nextunique"：
        ```
        const transaction = db.transaction("users"), 
            store = transaction.objectStore("users"), 
            request = store.openCursor(null, "nextunique");
        ```
    - 注意，openCursor()的第一个参数是 null，表示默认的键范围是所有值。此游标会遍历对象存储中的记录，从第一条记录开始迭代，到最后一条记录，但会跳过重复的记录。
    - 另外，也可以创建在对象存储中反向移动的游标，从最后一项开始向第一项移动。此时需要给openCursor()传入"prev"或"prevunique"作为第二个参数（后者的意思当然是避免重复）。例如：
        ```
        const transaction = db.transaction("users"), 
            store = transaction.objectStore("users"), 
            request = store.openCursor(null, "prevunique"); 
        ```
    - 在使用"prev"或"prevunique"打开游标时，每次调用 continue()或 advance()都会在对象存储中反向移动游标。

22. 索引
    - 对某些数据集，可能需要为对象存储指定多个键。例如，如果同时记录了用户 ID 和用户名，那可能需要通过任何一种方式来获取用户数据。为此，可以考虑将用户 ID 作为主键，然后在用户名上创建索引。
    - 要创建新索引，首先要取得对象存储的引用，然后像下面的例子一样调用 createIndex()：
        ```
        const transaction = db.transaction("users"), 
            store = transaction.objectStore("users"), 
            index = store.createIndex("username", "username", { unique: true });
        ```
    - createIndex()的第一个参数是索引的名称，第二个参数是索引属性的名称，第三个参数是包含键 unique 的 options 对象。这个选项中的 unique 应该必须指定，表示这个键是否在所有记录中唯一。因为 username 可能不会重复，所以这个键是唯一的。
    - createIndex()返回的是 IDBIndex 实例。在对象存储上调用 index()方法也可以得到同一个实例。例如，要使用一个已存在的名为"username"的索引，可以像下面这样：
        ```
        const transaction = db.transaction("users"), 
            store = transaction.objectStore("users"), 
            index = store.index("username");
        ```
    - 索引非常像对象存储。可以在索引上使用 openCursor()方法创建新游标，这个游标与在对象存储上调用 openCursor()创建的游标完全一样。只是其 result.key 属性中保存的是索引键，而不是主键。下面看一个例子：
        ```
        const transaction = db.transaction("users"), 
            store = transaction.objectStore("users"), 
            index = store.index("username"), 
            request = index.openCursor(); 
        request.onsuccess = (event) => { 
            // 处理成功
        };
        ```
    - 使用 openKeyCursor()方法也可以在索引上创建特殊游标，只返回每条记录的主键。这个方法接收的参数与 openCursor()一样。最大的不同在于，event.result.key 是索引键，且 event.result.value是主键而不是整个记录。
        ```
        const transaction = db.transaction("users"), 
            store = transaction.objectStore("users"), 
            index = store.index("username"), 
            request = index.openKeyCursor(); 
        request.onsuccess = (event) => { 
            // 处理成功
            // event.result.key 是索引键，event.result.value 是主键
        };
        ```
    - 可以使用 get()方法并传入索引键通过索引取得单条记录，这会创建一个新请求：
        ```
        const transaction = db.transaction("users"), 
            store = transaction.objectStore("users"), 
            index = store.index("username"), 
            request = index.get("007"); 
        request.onsuccess = (event) => { 
            // 处理成功
        }; 
        request.onerror = (event) => { 
            // 处理错误
        };
        ```
    - 如果想只取得给定索引键的主键，可以使用 getKey()方法。这样也会创建一个新请求，但result.value 等于主键而不是整个记录：
        ```
        const transaction = db.transaction("users"), 
            store = transaction.objectStore("users"), 
            index = store.index("username"), 
            request = index.getKey("007"); 
        request.onsuccess = (event) => { 
            // 处理成功
            // event.target.result.key 是索引键，event.target.result.value 是主键
        };
        ```
    - 在这个 onsuccess 事件处理程序中，event.target.result.value 中应该是用户 ID。
    - 任何时候，都可以使用 IDBIndex 对象的下列属性取得索引的相关信息。
        - name：索引的名称。
        - keyPath：调用 createIndex()时传入的属性路径。
        - objectStore：索引对应的对象存储。
        - unique：表示索引键是否唯一的布尔值。
    - 对象存储自身也有一个 indexNames 属性，保存着与之相关索引的名称。使用如下代码可以方便地了解对象存储上已存在哪些索引：
        ```
        const transaction = db.transaction("users"), 
            store = transaction.objectStore("users"), 
            indexNames = store.indexNames;
        for (let indexName in indexNames) { 
            const index = store.index(indexName); 
            console.log(`Index name: ${index.name} 
                        KeyPath: ${index.keyPath} 
                        Unique: ${index.unique}`); 
        }
        ```
    - 以上代码迭代了每个索引并在控制台中输出了它们的信息。
    - 在对象存储上调用 deleteIndex()方法并传入索引的名称可以删除索引：
        ```
        const transaction = db.transaction("users"), 
            store = transaction.objectStore("users"), 
            store.deleteIndex("username");
        ```
    - 因为删除索引不会影响对象存储中的数据，所以这个操作没有回调。

23. 并发问题
    - IndexedDB 虽然是网页中的异步 API，但仍存在并发问题。如果两个不同的浏览器标签页同时打开了同一个网页，则有可能出现一个网页尝试升级数据库而另一个尚未就绪的情形。有问题的操作是设置数据库为新版本，而版本变化只能在浏览器只有一个标签页使用数据库时才能完成。
    - 第一次打开数据库时，添加 onversionchange 事件处理程序非常重要。另一个同源标签页将数据库打开到新版本时，将执行此回调。对这个事件最好的回应是立即关闭数据库，以便完成版本升级。例如：
        ```
        let request, database; 
        request = indexedDB.open("admin", 1); 
        request.onsuccess = (event) => { 
            database = event.target.result; 
            database.onversionchange = () => database.close(); 
        };
        ```
    - 应该在每次成功打开数据库后都指定 onversionchange 事件处理程序。记住，onversionchange有可能会被其他标签页触发。
    - 通过始终都指定这些事件处理程序，可以保证 Web 应用程序能够更好地处理与 IndexedDB 相关的并发问题。

24. 限制
    - IndexedDB 的很多限制实际上与 Web Storage 一样。首先，IndexedDB 数据库是与页面源（协议、域和端口）绑定的，因此信息不能跨域共享。这意味着 www.wrox.com 和 p2p.wrox.com 会对应不同的数据存储。
    - 其次，每个源都有可以存储的空间限制。当前 Firefox 的限制是每个源 50MB，而 Chrome 是 5MB。移动版 Firefox 有 5MB 限制，如果用度超出配额则会请求用户许可。
    - Firefox 还有一个限制——本地文本不能访问 IndexedDB 数据库。Chrome 没有这个限制。因此在本地运行本书示例时，要使用 Chrome。

25. 小结
    - Web Storage 定义了两个对象用于存储数据：sessionStorage 和 localStorage。前者用于严格保存浏览器一次会话期间的数据，因为数据会在浏览器关闭时被删除。后者用于会话之外持久保存数据。
    - IndexedDB 是类似于 SQL 数据库的结构化数据存储机制。不同的是，IndexedDB 存储的是对象，而不是数据表。对象存储是通过定义键然后添加数据来创建的。游标用于查询对象存储中的特定数据，而索引可以针对特定属性实现更快的查询。
    - 有了这些存储手段，就可以在客户端通过使用 JavaScript 存储可观的数据。因为这些数据没有加密，所以要注意不能使用它们存储敏感信息。
