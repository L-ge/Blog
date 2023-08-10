# （六）Dom 获取&属性操作

以下的变量可以将 let 改为 const：
```js
let arr = ['red', 'green']
arr.push('pink')
console.log(arr)    // ['red', 'green', 'pink']

let person = {
    uname: 'pink老师',
    age: 18,
    gender: '女'
}
person.address = '武汉黑马'
console.log(person)
```
- const 声明的值不能更改，而且 const 声明变量的时候需要里面进行初始化
- 但是对于引用数据类型，const 声明的变量，里面存的不是值，是地址！

```
// 以下写法错误，因为他们地址不一样：
const names = []
names = [1,2,3]

const obj = {}
obj = {
    uname: 'pink老师'
}

// 应该如下写法：
const names = []
names[0] = 1
names[1] = 2
names[2] = 3

const obj = {}
obj.uname = 'pink老师'
```
- 为什么const声明的对象可以修改里面的属性？
    - 因为对象是引用类型，里面存储的是地址，只要地址不变，就不会报错
    - 建议数组和对象使用 const 来声明
- 声明变量先给const，如果发现它后面是要被修改的，再改为let
- 如果基本数据类型的值或者引用类型的地址发生变化的时候，需要用let

## 1. Web API 基本认知

### 1.1 作用和分类

- 作用: 就是使用 JS 去操作 html 和浏览器
- 分类：DOM (文档对象模型)、BOM（浏览器对象模型）

### 1.2 什么是DOM

- DOM（Document Object Model——文档对象模型）是用来呈现以及与任意 HTML 或 XML文档交互的API
- 白话文：DOM是浏览器提供的一套专门用来 操作网页内容 的功能
- DOM作用
    - 开发网页内容特效和实现用户交互

### 1.3 DOM树

- DOM树是什么
    - 将 HTML 文档以树状结构直观的表现出来，我们称之为文档树或 DOM 树
    - 描述网页内容关系的名词
    - 作用：文档树直观的体现了标签与标签之间的关系

### 1.4 DOM对象（重要）

- DOM对象：浏览器根据html标签生成的 JS对象
    - 所有的标签属性都可以在这个对象上面找到
    - 修改这个对象的属性会自动映射到标签身上
- DOM的核心思想
    - 把网页内容当做对象来处理
- document 对象
    - 是 DOM 里提供的一个对象
    - 所以它提供的属性和方法都是用来访问和操作网页内容的
        - 例：document.write()
    - 网页所有内容都在document里面

## 2. 获取DOM对象

查找元素DOM元素就是利用 JS 选择页面中标签元素。

### 2.1 根据CSS选择器来获取DOM元素 (重点)

1. 选择匹配的第一个元素
- 语法：
    ```js
    document.querySelector('css选择器')
    ```
- 参数：
    - 包含一个或多个有效的CSS选择器 字符串
- 返回值：
    - CSS选择器匹配的第一个元素，一个 HTMLElement对象。
    - 如果没有匹配到，则返回null。
- 多参看文档：https://developer.mozilla.org/zh-CN/docs/Web/API/Document/querySelector

2. 选择匹配的多个元素
- 语法：
    ```js
    document.querySelectorAll('css选择器')
    ```
- 参数：
    - 包含一个或多个有效的CSS选择器 字符串
- 返回值：
    - CSS选择器匹配的NodeList 对象集合
- 例如：
    ```js
    document.querySelectorAll('ul li')
    ```

3. `document.querySelectorAll('css选择器')` 得到的是一个伪数组：
    - 有长度有索引号的数组
    - 但是没有 pop() push() 等数组方法
- 想要得到里面的每一个对象，则需要遍历（for）的方式获得。
- 哪怕只有一个元素，通过querySelectAll() 获取过来的也是一个伪数组，里面只有一个元素而已
- querySelector() 只能选择一个元素，可以直接操作。
- querySelectorAll() 可以选择多个元素，得到的是伪数组，需要遍历得到每一个元素。
- 小括号里面的参数的注意事项：
    - 里面写css选择器；
    - 必须是字符串，也就是必须加引号。

### 2.2 其他获取DOM元素方法（了解）

```js
// 根据 id 获取一个元素
document.getElementById('nav')
// 根据标签获取一类元素。获取页面所有div
document.getElementsByTagName('div')
// 根据类名获取元素。获取页面所有类名为w的
document.getElementsByClassName('w')
```

## 3. 操作元素内容

- DOM对象都是根据标签生成的，所以操作标签，本质上就是操作DOM对象。
- 就是操作对象使用的点语法。
- 如果想要修改标签元素的里面的内容，则可以使用如下几种方式：
    - 对象.innerText 属性
    - 对象.innerHTML 属性

### 3.1 元素 innerText 属性

- 将文本内容添加/更新到任意标签位置
- 显示纯文本，不解析标签
- 例子：
    ```js
    const info = document.querySelector('.info')
    // 获取标签内部的文字
    // console.log(info.innerText)
    // 添加/修改标签内部文字内容
    info.innerText = '哈哈,你好...'
    ```

### 3.2 元素 innerHTML 属性

- 将文本内容添加/更新到任意标签位置
- 会解析标签，多标签建议使用模板字符
- 例子：
    ```js
    const info = document.querySelector('.info')
    // 获取标签内部的文字
    // console.log(info.innerHTML)
    // 添加/修改标签内部文字内容
    info.innerHTML = '哈哈,<strong>你好</strong>...'
    ```

## 4. 操作元素属性

### 4.1 操作元素常用属性

- 还可以通过 JS 设置/修改标签元素属性，比如通过 src更换 图片
- 最常见的属性比如： href、title、src 等
- 语法：
    ```
    对象.属性 = 值
    ```    
- 例子：
    ```js
    // 1. 获取元素
    const pic = document.querySelector('img')
    // 2. 操作元素
    pic.src = './images/b02.jpg'
    pic.title = '你好'
    ```

### 4.2 操作元素样式属性

- 还可以通过 JS 设置/修改标签元素的样式属性。
    - 比如通过轮播图小圆点自动更换颜色样式
    - 点击按钮可以滚动图片，这是移动的图片的位置 left 等等
- 学习路径：
    - 通过 style 属性操作CSS
    - 操作类名(className) 操作CSS
    - 通过 classList 操作类控制CSS

1. 通过 style 属性操作CSS
- 语法：
    ```
    对象.style.样式属性 = 值
    ```
- 例子：
    ```js
    const box = document.querySelector('.box')
    // 修改元素样式
    box.style.width = '200px'
    box.style.marginTop = '15px'
    box.style.backgroundColor = 'pink'
    ```
- 注意：
    - 修改样式通过style属性引出
    - 如果属性有-连接符，需要转换为小驼峰命名法
    - 赋值的时候，需要的时候不要忘记加css单位
- 标签选择body时，因为body是唯一的标签，可以直接写 document.body.style。

2. 操作类名(className) 操作CSS
- 如果修改的样式比较多，直接通过style属性修改比较繁琐，我们可以通过借助于css类名的形式。
- 语法：
    ```
    // active 是一个 css 类名
    元素.className = 'active'
    ```
- 注意：
    - 由于class是关键字, 所以使用className去代替
    - className是使用新值换旧值，如果需要添加一个类，需要保留之前的类名

3. 通过 classList 操作类控制CSS
- 为了解决className 容易覆盖以前的类名，我们可以通过classList方式追加和删除类名
- 语法：
    ```js
    // 追加一个类
    元素.classList.add('类名')
    // 删除一个类
    元素.classList.remove('类名')
    // 切换一个类
    元素.classList.toggle('类名')
    ```

### 4.3 操作表单元素属性

- 表单很多情况，也需要修改属性，比如点击眼睛，可以看到密码，本质是把表单类型转换为文本框
- 正常的有属性有取值的 跟其他的标签属性没有任何区别
- 获取: DOM对象.属性名
- 设置: DOM对象.属性名 = 新值

```
表单.value = '用户名'
表单.type = 'password'
```

- 表单属性中添加就有效果,移除就没有效果,一律使用布尔值表示 如果为true 代表添加了该属性 如果是false 代表移除了该属性。比如： disabled、checked、selected

### 4.4 自定义属性

- 标准属性: 标签天生自带的属性 比如class id title等, 可以直接使用点语法操作比如：disabled、checked、selected
- 自定义属性：
    - 在html5中推出来了专门的data-自定义属性
    - 在标签上一律以data-开头
    - 在DOM对象上一律以dataset对象方式获取
- 例子：
    ```html
    <body>
        <div class="box" data-id="10">盒子</div>
        <script>
            const box = document.querySelector('.box')
            console.log(box.dataset.id)
        </script>
    </body>
    ```    

## 5. 定时器-间歇函数

### 5.1 定时器函数介绍

- 网页中经常会需要一种功能：每隔一段时间需要自动执行一段代码，不需要我们手动去触发
- 例如：网页中的倒计时
- 要实现这种需求，需要定时器函数
- 定时器函数有两种，下面先介绍间歇函数

### 5.2 定时器函数基本使用

定时器函数可以开启和关闭定时器

1. 开启定时器
    ```js
    setInterval(函数, 间隔时间)
    ```
    - 作用：每隔一段时间调用这个函数
    - 间隔时间单位是毫秒
- 例子：
    ```js
    funcion repeat() {
        console.log('前端程序员')
    }
    // 每隔一秒调用 repeat 函数
    setInterval(repeat, 1000)
    ```
- 注意：
    - 函数名字不需要加括号
    - 定时器返回的是一个id数字

2. 关闭定时器
    ```
    let 变量名 = setInterval(函数, 间隔时间)
    clearInterval(变量名)
    ```
- 一般不会刚创建就停止，而是满足一定条件再停止。
- 例子：
    ```js
    let timer = setInterval(funcion() {
        console.log('前端程序员')
    }, 1000)
    clearInterval(timer)
    ```
