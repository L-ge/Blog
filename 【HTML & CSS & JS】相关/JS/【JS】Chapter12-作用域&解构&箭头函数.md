# 站在巨人的肩膀上

> [黑马程序员前端JavaScript入门到精通全套视频教程，javascript核心进阶ES6语法、API、js高级等基础知识和实战教程](https://www.bilibili.com/video/BV1Y84y1L7Nn/)


# （十二）作用域&解构&箭头函数

## 1. 作用域

- 使用闭包函数创建隔离作用域避免全局变量污染。
- 作用域（scope）规定了变量能够被访问的“范围”，离开了这个“范围”变量便不能被访问。
- 作用域分为：
    - 局部作用域
    - 全局作用

### 1.1 局部作用域

- 局部作用域分为函数作用域和块作用域。

1. 函数作用域：
- 在函数内部声明的变量只能在函数内部被访问，外部无法直接访问。
- 总结：
    1. 函数内部声明的变量，在函数外部无法被访问
    2. 函数的参数也是函数内部的局部变量
    3. 不同函数内部声明的变量无法互相访问
    4. 函数执行完毕后，函数内部的变量实际被清空了

2. 块作用域：
- 在 JavaScript 中使用 { } 包裹的代码称为代码块，代码块内部声明的变量外部将【有可能】无法被访问。
- 总结：
    1. let 声明的变量会产生块作用域，var 不会产生块作用域
    2. const 声明的常量也会产生块作用域
    3. 不同代码块之间的变量无法互相访问
    4. 推荐使用 let 或 const

### 1.2 全局作用域

- <script> 标签和 .js 文件的【最外层】就是所谓的全局作用域，在此声明的变量在函数内部也可以被访问。
- 全局作用域中声明的变量，任何其它作用域都可以被访问
- 注意：
    1. 为 window 对象动态添加的属性默认也是全局的，不推荐！
    2. 函数中未使用任何关键字声明的变量为全局变量，不推荐！！！
    3. 尽可能少的声明全局变量，防止全局变量被污染

### 1.3 作用域链

- 作用域链本质上是底层的变量查找机制。
    - 在函数被执行时，会优先查找当前函数作用域中查找变量
    - 如果当前作用域查找不到则会依次逐级查找父级作用域直到全局作用域
- 总结：
    1. 嵌套关系的作用域串联起来形成了作用域链
    2. 相同作用域链中按着从小到大的规则查找变量
    3. 子作用域能够访问父作用域，父级作用域无法访问子级作用域

### 1.4 JS垃圾回收机制

#### 1.4.1 什么是垃圾回收机制？

- 垃圾回收机制(Garbage Collection) 简称 GC
- JS中内存的分配和回收都是自动完成的，内存在不使用的时候会被垃圾回收器自动回收。
- 正因为垃圾回收器的存在，许多人认为JS不用太关心内存管理的问题
- 但如果不了解JS的内存管理机制，我们同样非常容易成内存泄漏（内存无法被回收）的情况
- 不再用到的内存，没有及时释放，就叫做内存泄漏

#### 1.4.2 内存的生命周期

- JS环境中分配的内存, 一般有如下生命周期：
    1. 内存分配：当我们声明变量、函数、对象的时候，系统会自动为他们分配内存
    2. 内存使用：即读写内存，也就是使用变量、函数等
    3. 内存回收：使用完毕，由垃圾回收自动回收不再使用的内存
    4. 说明：
        - 全局变量一般不会回收(关闭页面回收)；
        - 一般情况下局部变量的值, 不用了, 会被自动回收掉

#### 1.4.3 拓展-JS垃圾回收机制-算法说明

堆栈空间分配区别：
1. 栈（操作系统）: 由操作系统自动分配释放函数的参数值、局部变量等，基本数据类型放到栈里面。
2. 堆（操作系统）: 一般由程序员分配释放，若程序员不释放，由垃圾回收机制回收。复杂数据类型放到堆里面。

下面介绍两种常见的浏览器垃圾回收算法: 引用计数法和标记清除。

1. 引用计数法：
- IE采用的引用计数算法，定义“内存不再使用”，就是看一个对象是否有指向它的引用，没有引用了就回收对象
- 算法：
    1. 跟踪记录被引用的次数
    2. 如果被引用了一次，那么就记录次数1,多次引用会累加 ++
    3. 如果减少一个引用就减1 --
    4. 如果引用次数是0 ，则释放内存
- 但它却存在一个致命的问题：嵌套引用（循环引用）
- 如果两个对象相互引用，尽管他们已不再使用，垃圾回收器不会进行回收，导致内存泄露。
- 因为他们的引用次数永远不会是0。这样的相互引用如果说很大量的存在就会导致大量的内存泄露：
    ```
    function fn() {
        let o1 = {}
        let o2 = {}
        o1.a = o2
        o2.a = o1
        return '引用计数无法回收'
    }
    fn()
    ```

2. 标记清除法
- 现代的浏览器已经不再使用引用计数算法了。
- 现代浏览器通用的大多是基于标记清除算法的某些改进算法，总体思想都是一致的。
- 核心：
    1. 标记清除算法将“不再使用的对象”定义为“无法达到的对象”。
    2. 就是从根部（在JS中就是全局对象）出发定时扫描内存中的对象。凡是能从根部到达的对象，都是还需要使用的。
    3. 那些无法由根部出发触及到的对象被标记为不再使用，稍后进行回收。

### 1.5 闭包

- 概念：一个函数对周围状态的引用捆绑在一起，内层函数中访问到其外层函数的作用域
- 简单理解：闭包 = 内层函数 + 外层函数的变量
- 闭包作用：封闭数据，提供操作，外部也可以访问函数内部的变量
- 闭包的基本格式:
    ```js
    function outer() {
        let i = 1
        fucntion fn() {
            console.log(i)
        }
        return fn
    }
    const fun = outer()
    fun()       // 1
    // 外层函数使用内部函数的变量
    
    // 简约写法
    function outer() {
        let i = 1
        return fucntion () {
            console.log(i)
        }
    }
    const fun = outer()
    fun()       // 调用fun  1
    // 外层函数使用内部函数的变量
    ```
- 闭包应用：实现数据的私有。
- 比如，我们要做个统计函数调用次数，函数调用一次，就++
    - 不推荐例子：
        ```js
        let count = 1
        function fn() {
            count++
            console.log('函数被调用${count}次')
        }
        fn()    // 2
        fn()    // 3
        ```
        - 但是，这个 count 是个全局变量，很容易被修改。
    - 推荐例子：
        ```js
        function fn() {
            let count = 1
            function fun() {
                count++
                console.log('函数被调用${count}次')
            }
            return fun
        }
        const result = fn()
        result()    // 2
        result()    // 3
        ```    
        - 这样实现了数据私有，无法直接修改 count。
- 闭包的作用？
    - 封闭数据，实现数据私有，外部也可以访问函数内部的变量
    - 闭包很有用，因为它允许将函数与其所操作的某些数据（环境）关联起来
- 闭包可能引起的问题？
    - 内存泄漏。

### 1.6 变量提升

- 变量提升是 JavaScript 中比较“奇怪”的现象，它允许在变量声明之前即被访问（仅存在于var声明变量）
- 注意：
    1. 变量在未声明即被访问时会报语法错误
    2. 变量在var声明之前即被访问，变量的值为 undefined
    3. let/const 声明的变量不存在变量提升
    4. 变量提升出现在相同作用域当中
    5. 实际开发中推荐先声明再访问变量
- JS初学者经常花很多时间才能习惯变量提升，还经常出现一些意想不到的bug，正因为如此，ES6 引入了块级作用域，用 let 或者 const 声明变量，让代码写法更加规范和人性化。

- 用 var 关键字声明变量会有变量提升
- 变量提升是什么流程？
    - 先把 var 变量提升到当前作用域于最前面
    - 只提升变量声明，不提升变量赋值
    - 然后依次执行代码
- 我们不建议使用var声明变量

## 2. 函数进阶

### 2.1 函数提升

- 函数提升与变量提升比较类似，是指函数在声明之前即可被调用。
- 例子：
    ```js
    // 调用函数
    foo()
    // 声明函数
    function foo() {
        console.log('声明之前即被调用...')
    }
    
    // 不存在提升现象
    bar()   // 错误
    var var = function() {
        console.log('函数表达式不存在提升现象...')
    }
    ```
- 总结：
    1. 函数提升能够使函数的声明调用更灵活
    2. 函数表达式不存在提升的现象
    3. 函数提升出现在相同作用域当中

### 2.2 函数参数

1. 动态参数
- arguments 是函数内部内置的伪数组变量，它包含了调用函数时传入的所有实参
- 总结：
    1. arguments 是一个伪数组，只存在于函数中
    2. arguments 的作用是动态获取函数的实参
    3. 可以通过for循环依次得到传递过来的实参
- 例子：
    ```js
    function sum() {
        // console.log(arguments)
        let s = 0
        for(let i = 0; i < arguments.length; i++) {
            s += arguments[i]
        }
        console.log(s)
    }
    // 调用求和函数
    sum(5, 10)
    sum(1, 2, 4)
    ```
- 当不确定传递多少个实参的时候，我们使用 arguments 动态参数。

2. 剩余参数
- 剩余参数允许我们将一个不定数量的参数表示为一个数组
- 例子：
    ```js
    function getSum(...other) {
        // other 得到 [1,2,3]
        console.log(other)
    }
    getSum(1, 2, 3)
    ```
- 和 arguments 有什么不同：  
    1. ... 是语法符号，置于最末函数形参之前，用于获取多余的实参
    2. 借助 ... 获取的剩余实参，是个真数组
    3. 开发中，还是提倡多使用剩余参数
- 例子：
    ```js
    function config(baseURL, ...other) {
        console.log(baseURL)    // 得到 http://baidu.com
        console.log(other)      // 得到 ['get', 'json']
    }
    config('http://baidu.com', 'get', 'json');
    ```

3. 展开运算符
- 展开运算符(…)，将一个数组进行展开
- 例子：
    ```js
    const arr = [1, 5, 3, 8, 2]
    console.log(...arr)     // 1 5 3 8 2
    ```
    - 不会修改原数组
- 典型运用场景：求数组最大值(最小值)、合并数组等
- 例子：
    ```js
    const arr = [1, 5, 3, 8, 2]
    console.log(Math.max(...arr))     // 8
    console.log(Math.min(...arr))     // 1
    
    // 合并数组
    const arr1 = [1, 2, 3]
    const arr2 = [4 ,5, 6]
    const arr3 = [...arr1, ...arr3]
    console.log(arr3)   // [1,2,3,4,5,6]
    ```
- 展开运算符 or 剩余参数：
    - 剩余参数：函数参数使用，得到真数组
    - 展开运算符：数组中使用，数组展开

### 2.3 箭头函数（重要）

- 引入箭头函数的目的是更简短的函数写法并且不绑定this，箭头函数的语法比函数表达式更简洁
- 使用场景：箭头函数更适用于那些本来需要匿名函数的地方

#### 2.3.1 基本语法

1. 语法1：基本写法
    ```js
    // 普通函数
    const fn = function() {
        console.log('我是普通函数')
    }
    fn()
    
    // 箭头函数
    const fn = () => {
        console.log('我是箭头函数')
    }
    fn()
    ```

2. 语法2：只有一个参数可以省略小括号
    ```js
    // 普通函数
    const fn = function(x) {
        return x + x
    }
    console.log(fn(1))  // 2
    
    // 箭头函数
    const fn = x => {
        return x + x
    }
    console.log(fn(1))  // 2
    ```

3. 语法3：如果函数体只有一行代码，可以写到一行上，并且无需写 return 直接返回值
    ```js
    // 普通函数
    const fn = function(x, y) {
        return x + y
    }
    console.log(fn(1, 2))  // 3
    
    // 箭头函数
    const fn = (x, y) => x + y
    console.log(fn(1, 2))  // 3
    
    // 更简洁的语法：
    const form = document.querySelector('form')
    form.addEventListener('click', ev => ev.preventDefault())
    ```

4. 语法4：加括号的函数体返回对象字面量表达式
    ```js
    const fn1 = uname => ({uname: uname})
    console.log(fn1('pink老师'))    // {uname: 'pink老师'}
    ```

5. 总结
- 箭头函数属于表达式函数，因此不存在函数提升
- 箭头函数只有一个参数时可以省略圆括号 ()
- 箭头函数函数体只有一行代码时可以省略花括号 {}，并自动做为返回值被返回
- 加括号的函数体返回对象字面量表达式

#### 2.3.2 箭头函数参数

- 普通函数有 arguments 动态参数
- 箭头函数没有 arguments 动态参数，但是有剩余参数 ...arg
- 例子：
    ```js
    const getSum = (...arg) => {
        let sum = 0
        for(let i = 0; i < args.lenth; i++) {
            sum += args[i]
        }
        return sum      // 注意函数体有多行代码需要 return
    }
    console.log(getSum(1, 2, 3))    // 6
    ```

#### 2.3.3 箭头函数this

- 在箭头函数出现之前，每一个新函数根据它是被如何调用的来定义这个函数的this值，非常令人讨厌。
- 箭头函数不会创建自己的this，它只会从自己的作用域链的上一层沿用this。
- 例子1：
    ```js
    console.log(this)   // 此处为 window
    const sayHi = function() {
        console.log(this)   // 普通函数指向调用者，此处为 window
    }
    btn.addEventListener('click', function() {
        console.log(this)   // 当前 this 指向 btn  
    })
    ```
- 例子2：
    ```js
    console.log(this)   // 此处为 window
    // 箭头函数
    const sayHi = () => {
        console.log(this)   // 箭头函数此处为 window
    }
    btn.addEventListener('click', () => {
        console.log(this)   // 当前 this 指向 window  
    })
    
    const user = {
        name: '小明',
        // 该箭头函数中的 this 为函数声明环境中 this 一致
        walk: () => {
            console.log(this)   // 指向 window，不是 user
        }
    }
    user.walk()
    ```
- 例子3：
    ```js
    const user = {
        name: '小明',
        sleep: function () {
            console.log(this)   // 指向 user
            const fn = () => {
                console.log(this)   // 指向 user，该箭头函数中的 this 与 sleep 中的 this 一致
            }
            // 调用箭头函数
            fn()
        }
    }
    user.sleep()
    ```
- 在开发中【使用箭头函数前需要考虑函数中 this 的值】，事件回调函数使用箭头函数时，this 为全局的 window，因此 DOM 事件回调函数为了简便，还是不太推荐使用箭头函数

## 3. 解构赋值

- 使用解构简洁语法快速为变量赋值。
- 例子：
    ```js
    const [max, min, avg] = [100, 60, 80]
    console.log(max)    // 100
    console.log(min)    // 60
    console.log(avg)    // 80
    ```
- 解构赋值是一种快速为变量赋值的简洁语法，本质上仍然是为变量赋值。分为：
    - 数组解构
    - 对象解构

### 3.1 数组解构

- 数组解构是将数组的单元值快速批量赋值给一系列变量的简洁语法。
- 基本语法：
    1. 赋值运算符 = 左侧的 [] 用于批量声明变量，右侧数组的单元值将被赋值给左侧的变量
    2. 变量的顺序对应数组单元值的位置依次进行赋值操作
- 例子：
    ```js
    // 普通的数组
    const arr = [1, 2, 3]
    // 批量声明变量 a b c
    // 同时将数组单元值 1 2 3 依次赋值给变量 a b c
    const [a, b, c] = arr
    console.log(a)  // 1
    console.log(b)  // 2
    console.log(c)  // 3
    ```
- 典型应用：交互2个变量
    ```js
    let a = 1
    let b = 3;      // 这里必须有分号！！！
    [b, a] = [a, b]
    console.log(a)  // 3
    console.log(b)  // 1
    ```
- **js前面必须加分号情况：**
    ```js
    // 1. 立即执行函数
    (function t() {})();
    // 或者
    ;(function t() {})()
    
    // 2. 数组解构
    // 数组开头的，特别是前面有语句的一定注意加分号
    ;[b, a] = [a, b]
    ```
- 变量多，单元值少的情况：
    ```js
    const [a, b, c, d] = ['小米', '苹果', '华为']
    console.log(a)  // 小米
    console.log(b)  // 苹果
    console.log(c)  // 华为
    console.log(d)  // undefined
    ```
    - 变量的数量大于单元值数量时，多余的变量将被赋值为 undefined
- 变量少，单元值多的情况：
    ```js
    const [a, b, c] = ['小米', '苹果', '华为', '格力']
    console.log(a)  // 小米
    console.log(b)  // 苹果
    console.log(c)  // 华为
    ```
- 利用剩余参数解决变量少但单元值多的情况：
    ```js
    const [a, b, ...tel] = ['小米', '苹果', '华为', '格力', 'vivo']
    console.log(a)      // 小米
    console.log(b)      // 苹果
    console.log(tel)    // ['华为', '格力', 'vivo']
    ```
    - 剩余参数返回的还是一个数组。
- 防止有undefined传递单元值的情况，可以设置默认值：
    ```js
    const [a = '手机', b = '华为'] = ['小米']
    console.log(a)      // 小米
    console.log(b)      // 华为
    ```
    - 允许初始化变量的默认值，且只有单元值为 undefined 时默认值才会生效
- 按需导入，忽略某些返回值：
    ```js
    const [a, , c, d] = ['小米', '苹果', '华为', '格力']
    console.log(a)      // 小米
    console.log(c)      // 华为
    console.log(d)      // 格力
    ``` 
- 支持多维数组的结构：
    ```js
    const [a, b] = ['苹果', ['小米', '华为']]
    console.log(a)      // 苹果
    console.log(b)      // ['小米', '华为']
    
    const [a, [b, c]] = ['苹果', ['小米', '华为']]
    console.log(a)      // 苹果
    console.log(b)      // 小米
    console.log(c)      // 华为
    ```

### 3.2 对象解构

- 对象解构是将对象属性和方法快速批量赋值给一系列变量的简洁语法
- 基本语法：
    1. 赋值运算符 = 左侧的 {} 用于批量声明变量，右侧对象的属性值将被赋值给左侧的变量
    2. 对象属性的值将被赋值给与属性名相同的变量
    3. 注意解构的变量名不要和外面的变量名冲突否则报错
    4.对象中找不到与变量名一致的属性时变量值为 undefined
- 例子：
    ```js
    // 普通对象
    const user = {
        name: '小明',
        age: 18
    };
    // 批量声明变量 name age
    // 同时将数组单元值 小明 18 依次赋值给变量 name age
    const [name, age] = user
    
    console.log(name)   // 小明
    console.log(age)    // 18
    ```
- 给新的变量名赋值：可以从一个对象中提取变量并同时修改新的变量名
    ```js
    // 普通对象
    const user = {
        name: '小明',
        age: 18
    };
    // 把原来的name变量重新命名为uname
    const [name: uname, age] = user
    
    console.log(uname)   // 小明
    console.log(age)     // 18
    ```
- 数组对象解构：
    ```js
    const pig = [
        {
            name: '佩奇',
            age: 6
        }
    ]
    
    const [{name, age}] = pig
    console.log(name, age)
    ```
- 多级对象解构：
    ```js
    // 依次打印家庭成员
    const pig = {
        name: '佩奇',
        family: {
            mother: '猪妈妈',
            father: '猪爸爸',
            sister: '乔治'
        },
        age: 6
    }
    const { name, family: {mother, father, sister} } = pig
    console.log(name)       // 佩奇
    console.log(mother)     // 猪妈妈
    console.log(father)     // 猪爸爸
    console.log(sister)     // 乔治
    
    // 这是后台传递过来的数据，请选出data里面的数据方便后面渲染
    const msg = {
        "code": 200,
        "msg": "获取新闻列表成功",
        "data": [
            {
                "id": 1,
                "title": "xxx",
                "count": 58
            },
            {
                "id": 2,
                "title": "yyy",
                "count": 56
            },
            {
                "id": 3,
                "title": "zzz",
                "count": 1800
            }
        ]
    }
    const {data} = msg
    console.log(data)
    
    // 将上面对象只选出data数据，传递给另外一个函数
    function render({ data }) {
        // 内部处理
        console.log(data)
    }
    render(msg)
    
    // 为了防止msg里面的data名字混淆，渲染函数里面的数据名改为 myData
    function render({ data: myData }) {
        // 内部处理
        console.log(myData)
    }
    render(msg)
    ```

### 3.3 遍历数组 forEach 方法（重点）

- forEach() 方法用于调用数组的每个元素，并将元素传递给回调函数
- 主要使用场景：遍历数组的每个元素
- 语法：
    ```
    被遍历的数组.forEach(function (当前数组元素, 当前元素索引号) {
        // 函数体
    })
    ```
- 例如：
    ```js
    const arr = ['pink', 'red', 'green']
    arr.forEach(function (item, index) {
        console.log('当前数组元素是：${item}')          // 依次打印数组每一个元素
        console.log('当前数组元素的索引是：${index}')   // 依次打印数组每一个元素的索引
    })
    ```
- 注意：
    1. forEach 主要是遍历数组
    2. 参数当前数组元素是必须要写的，索引号可选

## 4. 综合案例

### 4.1 筛选数组 filter 方法（重点）

- filter() 方法创建一个新的数组，新数组中的元素是通过检查指定数组中符合条件的所有元素
- 主要使用场景：筛选数组符合条件的元素，并返回筛选之后元素的新数组
- 语法：
    ```
    被遍历的数组.filter(function (currentValue, index) {
        return 筛选条件
    })
    ```
- 例子：
    ```js
    // 筛选数组中大于30的元素
    const score = [10, 50, 3, 40, 33]
    const re = score.filter(function (item) {
        return ietm > 30
    })
    console.log(re)     // [50, 40, 33]
    ```
- filter() 筛选数组
- 返回值：返回数组，包含了符合条件的所有元素。如果没有符合条件的元素则返回空数组
- 参数：currentValue 必须写， index 可选
- 因为返回新数组，所以不会影响原数组
