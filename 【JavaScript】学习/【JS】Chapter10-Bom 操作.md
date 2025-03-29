# 站在巨人的肩膀上

> [黑马程序员前端JavaScript入门到精通全套视频教程，javascript核心进阶ES6语法、API、js高级等基础知识和实战教程](https://www.bilibili.com/video/BV1Y84y1L7Nn/)


# （十）Bom 操作

## 1. Window对象

### 1.1 BOM(浏览器对象模型)

- BOM(Browser Object Model) 是浏览器对象模型
- window对象是一个全局对象，也可以说是JavaScript中的顶级对象
- 像document、alert()、console.log()这些都是window的属性，基本BOM的属性和方法都是window的。
- 所有通过var定义在全局作用域中的变量、函数都会变成window对象的属性和方法
- window对象下的属性和方法调用的时候可以省略window

### 1.2 定时器-延时函数

- JavaScript 内置的一个用来让代码延迟执行的函数，叫 setTimeout
- 语法：
    ```
    setTimeout(回调函数, 等待的毫秒数)
    ```    
- setTimeout 仅仅只执行一次，所以可以理解为就是把一段代码延迟执行, 平时省略window
- 清除延时函数：
    ```
    let timer = setTimeout(回调函数, 等待的毫秒数)
    clearTimeout(timer)
    ```
- 注意点
    - 延时器需要等待,所以后面的代码先执行
    - 每一次调用延时器都会产生一个新的延时器
- 两种定时器对比：执行的次数
    - 延时函数：执行一次
    - 间歇函数：每隔一段时间就执行一次,除非手动清除

### 1.3 JS执行机制

- 经典面试题1：
```
console.log(1111)
setTimeout(function() {
    console.log(2222)
}, 1000)
console.log(3333)

// 输出结果是
1111
3333
2222
```

- 经典面试题2：
```
console.log(1111)
setTimeout(function() {
    console.log(2222)
}, 0)
console.log(3333)

// 输出结果也是
1111
3333
2222
```

- JavaScript 语言的一大特点就是单线程，也就是说，同一个时间只能做一件事。
- 这是因为 Javascript 这门脚本语言诞生的使命所致——JavaScript 是为处理页面中用户的交互，以及操作 DOM 而诞生的。比如我们对某个 DOM 元素进行添加和删除操作，不能同时进行。应该先进行添加，之后再删除。
- 单线程就意味着，所有任务需要排队，前一个任务结束，才会执行后一个任务。这样所导致的问题是：如果 JS 执行的时间过长，这样就会造成页面的渲染不连贯，导致页面渲染加载阻塞的感觉。

- 为了解决这个问题，利用多核 CPU 的计算能力，HTML5 提出 Web Worker 标准，允许 JavaScript 脚本创建多个线程。于是，JS 中出现了同步和异步。
- 同步：前一个任务结束后再执行后一个任务，程序的执行顺序与任务的排列顺序是一致的、同步的。比如做饭的同步做法：我们要烧水煮饭，等水开了（10分钟之后），再去切菜，炒菜。
- 异步：你在做一件事情时，因为这件事情会花费很长时间，在做这件事的同时，你还可以去处理其他事情。比如做饭的异步做法，我们在烧水的同时，利用这10分钟，去切菜，炒菜。
- 同步和异步，他们的本质区别：这条流水线上各个流程的执行顺序不同。

- 同步任务：同步任务都在主线程上执行，形成一个执行栈。
- 异步任务：
    - JS 的异步是通过回调函数实现的。
    - 一般而言，异步任务有以下三种类型:
        1. 普通事件，如 click、resize 等
        2. 资源加载，如 load、error 等
        3. 定时器，包括 setInterval、setTimeout 等
    - 异步任务相关添加到任务队列中（任务队列也称为消息队列）。

- 执行机制：
    1. 先执行执行栈中的同步任务。
    2. 异步任务放入任务队列中。
    3. 一旦执行栈中的所有同步任务执行完毕，系统就会按次序读取任务队列中的异步任务，于是被读取的异步任务结束等待状态，进入执行栈，开始执行。

- 由于主线程不断的重复获得任务、执行任务、再获取任务、再执行，所以这种机制被称为事件循环（event loop）。

### 1.4 location对象

- location 的数据类型是对象，它拆分并保存了 URL 地址的各个组成部分
- 常用属性和方法：
    - href 属性获取完整的 URL 地址，对其赋值时用于地址的跳转
        ```js
        // 可以得到当前文件URL地址
        console.log(location.href)
        // 可以通过js方式跳转到目标地址
        location.href = 'http://www.itcast.cn'
        ```
    - search 属性获取地址中携带的参数，符号 ？后面部分
        ```js
        console.log(location.search)
        ```
    - hash 属性获取地址中的啥希值，符号 # 后面部分
        ```js
        console.log(location.hash)
        ```
        - 后期vue路由的铺垫，经常用于不刷新页面，显示不同页面，比如 网易云音乐
    - reload 方法用来刷新当前页面，传入参数 true 时表示强制刷新
        ```html
        <button>点击刷新</button>
        <script>
            let btn = document.querySelector('button')
            btn.addEventListener('click', function() {
                location.reload(true)
                // 强制刷新，类型ctrl+f5
            })
        </script>
        ```

### 1.5 navigator对象

- navigator 的数据类型是对象，该对象下记录了浏览器自身的相关信息
- 常用属性和方法：
    - 通过 userAgent 检测浏览器的版本及平台
        ```js
        // 检测 userAgent（浏览器信息）
        !(function() {
            const userAgent = navigator.userAgent
            // 验证是否为Android或iPhone
            const android = userAgent.match(/(Android);?[\s\/]+([\d.]+)?/)
            const iphone = userAgent.match(/(iPhone\sOS)\s([\d_]+)/)
            // 如果是Android或iPhone，则跳转至移动站点
            if (android || iphone) {
                location.href = 'http://m.itcast.cn'
            }
        })()
        ```

### 1.6 histroy对象

- history 的数据类型是对象，主要管理历史记录，该对象与浏览器地址栏的操作相对应，如前进、后退、历史记录等
- 常用属性和方法
    
    | history对象方法 | 作用 |
    | --- | --- |
    | back() | 可以后退功能 | 
    | forward() | 前进功能 |
    | go(参数) | 前进后退功能。参数如果是1,则前进1个页面;如果是-1,则后退1个页面 |

- history 对象一般在实际开发中比较少用，但是会在一些 OA 办公系统中见到。

## 2. 本地存储

### 2.1 本地存储介绍

为了满足各种各样的需求，会经常性在本地存储大量的数据，HTML5规范提出了相关解决方案。
    - 数据存储在用户浏览器中
    - 设置、读取方便、甚至页面刷新不丢失数据
    - 容量较大，sessionStorage和localStorage约 5M 左右
    - 常见的使用场景：
        - https://todomvc.com/examples/vanilla-es6/ 页面刷新数据不丢失

### 2.2 本地存储分类

#### 2.2.1 localStorage

- 使用 localStorage 把数据存储的浏览器中
- 作用: 可以将数据永久存储在本地(用户的电脑), 除非手动删除，否则关闭页面也会存在
- 特性：
    - 可以多窗口（页面）共享（同一浏览器可以共享）
    - 以键值对的形式存储使用
- 语法：
    ```
    // 存储数据：
    localStorage.setItem(key, value)
    
    // 获取数据：
    localStorage.getItem(key)

    // 删除数据：
    localStorage.removeItem(key)
    ```
- 浏览器查看本地数据：F12->Application->Storage->Local Storage

#### 2.2.2 sessionStorage

- sessionStorage 特性：
    - 生命周期为关闭浏览器窗口
    - 在同一个窗口(页面)下数据可以共享
    - 以键值对的形式存储使用
    - 用法跟 localStorage 基本相同

### 2.3 存储复杂数据类型

- 本地只能存储字符串,无法存储复杂数据类型.
- 解决：需要将复杂数据类型转换成JSON字符串,再存储到本地
- 语法：JSON.stringify(复杂数据类型)
- 例子：
    ```js
    const goods = {
        name: '小米10',
        price: 1999
    }
    localStorage.setItem('goods', JSON.stringify(goods))
    ```
    - 将复杂数据转换成JSON字符串存储本地存储中.

- 因为本地存储里面取出来的是字符串，不是对象，无法直接使用
- 解决：把取出来的字符串转换为对象
- 语法：JSON.parse(JSON字符串)
- 例子：
    ```
    const obj = JSON.parse(localStorage.getItem('goods'))
    console.log(obj)
    ```
    - 将JSON字符串转换成对象取出时候使用.

## 3. 综合案例

### 3.1 数组中map方法迭代数组

- map 也称为映射。映射是个术语，指两个元素的集之间元素相互“对应”的关系.
- 作用：map 迭代数组
- 语法：
    ```
    const arr = ['pink', 'red', 'blue']
    arr.map(function(item, index) {
        console.log(item)   // item 得到数组元素 'pink' 'red' 'blue'
        console.log(index)  // index 得到索引号 0 1 2
    })
    ```
- 使用场景：
    - map 可以处理数据，并且返回新的数组
- 例子：
    ```js
    const arr = ['pink', 'red', 'blue']
    const newArr = arr.map(function(item, index) {
        console.log(item)   // item 得到数组元素 'pink' 'red' 'blue'
        console.log(index)  // index 得到索引号 0 1 2
        return item + '老师'
    })
    console.log(newArr)     // ['pink老师', 'red老师', 'blue老师']
    ```

### 3.2 数组中join方法

- 作用：join() 方法用于把数组中的所有元素转换一个字符串
- 语法：
    ```
    const arr = ['pink老师', 'red老师', 'blue老师']
    console.log(arr.join(''))   // pink老师red老师blue老师
    ```
- 参数：
    - 数组元素是通过参数里面指定的分隔符进行分隔的
