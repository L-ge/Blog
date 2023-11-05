# 站在巨人的肩膀上

> [黑马程序员前端JavaScript入门到精通全套视频教程，javascript核心进阶ES6语法、API、js高级等基础知识和实战教程](https://www.bilibili.com/video/BV1Y84y1L7Nn/)


# （八）Dom 事件进阶

## 1. 事件流

### 1.1 事件流与两个阶段说明

- 事件流指的是事件完整执行过程中的流动路径
- 说明：假设页面里有个div，当触发事件时，会经历两个阶段，分别是捕获阶段、冒泡阶段
- 简单来说：捕获阶段是从父到子；冒泡阶段是从子到父
- 实际工作都是使用事件冒泡为主

### 1.2 事件捕获

- 事件捕获概念：
    - 从DOM的根元素开始去执行对应的事件 (从外到里)
- 事件捕获需要写对应代码才能看到效果
- 代码：
    ```
    DOM.addEventListener(事件类型, 事件处理函数, 是否使用捕获机制)
    ```
- 说明：
    - addEventListener第三个参数传入 true 代表是捕获阶段触发（很少使用）
    - 若传入false代表冒泡阶段触发，默认就是false
    - 若是用 L0 事件监听，则只有冒泡阶段，没有捕获

### 1.3 事件冒泡

- 事件冒泡概念: 当一个元素的事件被触发时，同样的事件将会在该元素的所有祖先元素中依次被触发。这一过程被称为事件冒泡
- 简单理解：当一个元素触发事件后，会依次向上调用所有父级元素的 同名事件
- 事件冒泡是默认存在的
- L2事件监听第三个参数是 false，或者默认都是冒泡。
- 例子；
    ```js
    const father = document.querySelector('.father')
    const son = document.querySelector('.son')
    document.addEventListener('click', function() {
        alert('我是爷爷')
    })
    fa.addEventListener('click', function() {
        alert('我是爸爸')
    })
    son.addEventListener('click', function() {
        alert('我是儿子')
    })
    ```

### 1.4 阻止冒泡

- 问题：因为默认就有冒泡模式的存在，所以容易导致事件影响到父级元素
- 需求：若想把事件就限制在当前元素内，就需要阻止事件冒泡
- 前提：阻止事件冒泡需要拿到事件对象
- 语法：
    ```
    事件对象.stopPropagation()
    ```
- 注意：此方法可以阻断事件流动传播，不光在冒泡阶段有效，捕获阶段也有效
- 例子：
    ```js
    const father = document.querySelector('.father')
    const son = document.querySelector('.son')
    document.addEventListener('click', function() {
        alert('我是爷爷')
    })
    fa.addEventListener('click', function() {
        alert('我是爸爸')
    })
    // 需要事件对象
    son.addEventListener('click', function(e) {
        alert('我是儿子')
        // 阻止冒泡
        e.stopPropagation()
    })
    ```

我们某些情况下需要阻止默认行为的发生，比如 阻止 链接的跳转，表单域跳转。
- 语法：
    ```
    e.preventDefault()
    ```
- 例子：
    ```html
    <form action="http://www.baidu.com">
        <input type="submit" value="提交">
    </form>
    <script>
        const form = document.querySelector('form')
        form.addEventListener('click', function(e) {
            // 阻止表单默认提交行为
            e.preventDefault()
        })
    </script>
    ```

### 1.5 解绑事件

- on事件方式，直接使用null覆盖偶就可以实现事件的解绑
- 语法：
    ```
    // 绑定事件
    btn.onClick = function() {
        alert('点击了')
    }
    // 解绑事件
    btn.onClick = null
    ```
- addEventListener方式，必须使用：removeEventListener(事件类型, 事件处理函数, [获取捕获或者冒泡阶段])
- 例如：
    ```js
    function fn() {
        alert('点击了')
    }
    // 绑定事件
    btn.addEventListener('click', fn)
    // 解绑事件
    btn.removeEventListener('click', fn)
    ```
- 注意：匿名函数无法被解绑。

#### 1.5.1 鼠标经过事件的区别

- 鼠标经过事件：
    - mouseover 和 mouseout 会有冒泡效果
    - mouseenter 和 mouseleave 没有冒泡效果（推荐）

#### 1.5.2 两种注册事件的区别：

- 传统on注册（L0）
    - 同一个对象,后面注册的事件会覆盖前面注册(同一个事件)
    - 直接使用null覆盖就可以实现事件的解绑
    - 都是冒泡阶段执行的
- 事件监听注册（L2）
    - 语法: addEventListener(事件类型, 事件处理函数, 是否使用捕获)
    - 后面注册的事件不会覆盖前面注册的事件(同一个事件)
    - 可以通过第三个参数去确定是在冒泡或者捕获阶段执行
    - 必须使用removeEventListener(事件类型, 事件处理函数, 获取捕获或者冒泡阶段)
    - 匿名函数无法被解绑

## 2. 事件委托

- 如果同时给多个元素注册事件，我们怎么做的？for循环注册事件：
    ```
    const lis = document.querySelectorAll('ul li')
    for(let i=0; i<lis.length; i++) {
        lis[i].addEventListener('click', function() {
            alert('我被点击了')
        })
    }
    ```
    - 有没有一种技巧注册一次事件就能完成以上效果呢？
- 事件委托是利用事件流的特征解决一些开发需求的知识技巧
    - 优点：减少注册次数，可以提高程序性能
    - 原理：事件委托其实是利用事件冒泡的特点。
    - 给父元素注册事件，当我们触发子元素的时候，会冒泡到父元素身上，从而触发父元素的事件。
    - 实现：事件对象.target.tagName 可以获得真正触发事件的元素
    - 例子：
        ```
        const ul = document.querySelector('ul')
        ul.addEventListener('click', function(e) {
            // console.dir(e.target)
            if(e.target.tagName === 'LI') {
                this.style.color = 'pink'
            }
        })
        ```

## 3. 其他事件

### 3.1 页面加载事件

#### 3.1.1 load 事件

- 加载外部资源（如图片、外联CSS和JavaScript等）加载完毕时触发的事件
- 为什么要学？
    - 有些时候需要等页面资源全部处理完了做一些事情
    - 老代码喜欢把 script 写在 head 中，这时候直接找 dom 元素找不到
- 事件名：load
- 监听页面所有资源加载完毕：
    - 给 window 添加 load 事件：
        ```js
        // 页面加载事件
        windows.addEventListener('load', function() {
            // 执行的操作
        })
        ```
- 注意：不光可以监听整个页面资源加载完毕，也可以针对某个资源绑定load事件

#### 3.1.2 DOMContentLoaded 事件

- 当初始的 HTML 文档被完全加载和解析完成之后，DOMContentLoaded 事件被触发，而无需等待样式表、图像等完全加载
- 事件名：DOMContentLoaded
- 监听页面DOM加载完毕：
    - 给 document 添加 DOMContentLoaded 事件：
        ```js
        document.addEventListener('DOMContentLoaded', function() {
            // 执行的操作
        }
        ```

#### 3.1.3 总结

页面加载事件有哪两个？如何添加？
- load 事件
    - 监听整个页面资源给 window 加
- DOMContentLoaded
    - 给 document 加
    - 无需等待样式表、图像等完全加

### 3.2 元素滚动事件

- 滚动条在滚动的时候持续触发的事件
- 为什么要学？
    - 很多网页需要检测用户把页面滚动到某个区域后做一些处理，比如固定导航栏，比如返回顶部
- 事件名：scroll
- 监听整个页面滚动：
    ```js
    // 页面滚动事件
    windows.addEventListener('scroll', function() {
        // 执行的操作
    }
    ```
    - 给 window 或 document 添加 scroll 事件
- 监听某个元素的内部滚动直接给某个元素加即可
- 使用场景：
    - 我们想要页面滚动一段距离，比如100px，就让某些元素显示隐藏，那我们怎么知道，页面滚动了100像素呢？就可以使用scroll 来检测滚动的距离。

#### 3.2.1 元素滚动事件——获取位置

- scrollLeft和scrollTop （属性）
    - 获取被卷去的大小
    - 获取元素内容往左、往上滚出去看不到的距离
    - 这两个值是可读写的
- 尽量在scroll事件里面获取被卷去的距离
- 例子：
    ```js
    div.addEventListener('scroll', function() {
        console.log(this.scrollTop)
    }
    ```
- 开发中，我们经常检测页面滚动的距离，比如页面滚动100像素，就可以显示一个元素，或者固定一个元素。
    ```js
    // 页面滚动事件
    windows.addEventListener('scroll', function() {
        // document.documentElement 是 html 元素获取方式
        const n = document.documentElement.scrollTop
        console.log(n)
    }
    ```
    - 注意：document.documentElement HTML 文档返回对象为HTML元素

#### 3.2.2 元素滚动事件——滚动到指定的坐标

- scrollTo() 方法可把内容滚动到指定的坐标
- 语法：
    ```
    元素.scrollTo(x, y)
    ```
- 例如:
    ```js
    // 让页面滚动到 y 轴1000像素的位置
    window.scrollTo(0, 1000)
    ```

### 3.3 页面尺寸事件

- 会在窗口尺寸改变的时候触发事件：
    - resize：
        ```js
        windows.addEventListener('resize', function() {
            // 执行的操作
        }
        ```
- 检测屏幕宽度：
    ```js
    windows.addEventListener('resize', function() {
        let w = document.documentElement.clientWidth
        console.log(w)
    }
    ```
- 获取宽高：
    - 获取元素的可见部分宽高（不包含边框，margin，滚动条等）
    - clientWidth和clientHeight

## 4. 元素尺寸与位置

- 使用场景：
    - 前面案例滚动多少距离，都是我们自己算的，最好是页面滚动到某个元素，就可以做某些事。
    - 简单说，就是通过js的方式，得到元素在页面中的位置
    - 这样我们可以做，页面滚动到这个位置，就可以做某些操作，省去计算了
- 获取宽高：
    - 获取元素的自身宽高，包含元素自身设置的宽高、padding、border。
    - offsetWidth和offsetHeight
    - 获取出来的是数值,方便计算
    - 注意: 获取的是可视宽高, 如果盒子是隐藏的,获取的结果是0
- 获取位置：
    - 获取元素距离自己定位父级元素的左、上距离（offsetLeft和offsetTop），注意是只读属性。
- offsetWidth和offsetHeight是得到元素什么的宽高？
    - 内容 + padding + border
- offsetTop和offsetLeft 得到位置以谁为准？
    - 带有定位的父级
    - 如果都没有则以 文档左上角 为准。
- element.getBoundingClientRect()：方法返回元素的大小及其相对于视口的位置。

总结：

| 属性 | 作用 | 说明 |
| --- | --- | --- |
| scrollLeft和scrollTop | 被卷去的头部和左侧 | 配合页面滚动来用，可读写 |  
| clientWidth和clientHeight | 获得元素宽度和高度 | 不包含border、margin、滚动条，用于js获取元素大小，只读属性 |
| offsetWidth和offsetHeight | 获得元素宽度和高度 | 包含border、padding，滚动条等，只读 |
| offsetLeft和offsetTop | 获取元素距离自己定位父级元素的左、上距离 | 获取元素位置的时候使用，只读属性 |
