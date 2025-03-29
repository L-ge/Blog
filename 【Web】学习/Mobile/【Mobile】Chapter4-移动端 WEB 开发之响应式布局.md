# 站在巨人的肩膀上

> [黑马程序员pink老师前端入门教程，零基础必看的h5(html5)+css3+移动端前端视频教程](https://www.bilibili.com/video/BV14J4114768/)

> [MurphyChen's Notes](https://github.com/Hacker-C/notes)


# （四）移动端 WEB 开发之响应式布局

## 1. 响应式开发

### 1.1 响应式开发原理

就是使用媒体查询针对不同宽度的设备进行布局和样式的设置，从而适配不同设备的目的。

| 设备划分                  | 尺寸区间              |
| ------------------------- | --------------------- |
| 超小屏幕（手机）          | `w < 768px`           |
| 小屏设备（平板）          | `768px <= w < 992px`  |
| 中等屏幕（桌面显示器）    | `992px <= w < 1200px` |
| 宽屏设备（大桌面显示器）  | `w >= 1200px`         |

### 1.2 响应式布局容器

响应式 **需要一个父级作为布局容器，来配合子级元素来实现变化效果。**  

原理就是在不同屏幕下，通过媒体查询来改变这个布局容器的大小，再改变里面子元素的排列方式和大小，从而实现不同屏幕下，看到不同的页面布局和样式变化。

平时我们的响应式尺寸划分（但是我们也可以根据实际情况自己定义划分）：
- 超小屏幕（手机，小于768px）：设置宽度为 `100%`
- 小屏幕（平板，大于等于768px）：设置宽度为 `750px`
- 中等屏幕（桌面显示器，大于等于992px）：宽度设置为 `970px`
- 大屏幕（大桌面显示器，大于等于1200px）：宽度设置为 `1170px`

代码实现：  

HTML部分：
```html
<!-- 布局容器 -->
<div class="container"></div>
```

CSS部分：
```css
.container {
    height: 150px;
    margin: 0 auto;
    background-color: pink;
}
/* 超小屏幕 小于 768，布局容器的宽度为 100% */
@media screen and (max-width: 767px) {
    .container {
        width: 100%;
    }
}
/* 小屏幕 大于等于 768，布局容器 750px */
@media screen and (min-width: 768px) {
    .container {
        width: 750px;
    }
}
/* 中等屏幕 */
@media screen and (min-width: 992px) {
    .container {
        min-width: 970px;
    }
}
/* 大屏幕 */
@media screen and (min-width: 1200px) {
    .container {
        width: 1170px;
    }
}
```

### 1.3 案例：响应式导航

- 当我们屏幕大于等于 768 像素，我们给布局容器 `container` 宽度为 750px。
- `container` 里面包含 8 个小 `li` 盒子，每个盒子的宽度定为 93.75px（750/8），高度为30px，浮动一行显示。
- 当我们屏幕缩放，宽度小于 768 像素的时候，`container` 盒子宽度修改为 100% 宽度。
- 此时里面的 8 个小 `li`，宽度修改为 33.33%，这样一行就只能显示 3 个小 `li`，剩余下行显示。

代码实现：

HTML部分:
```html
<div class="container">
    <ul>
        <li>导航栏</li>
        <li>导航栏</li>
        <li>导航栏</li>
        <li>导航栏</li>
        <li>导航栏</li>
        <li>导航栏</li>
        <li>导航栏</li>
        <li>导航栏</li>
    </ul>
</div>
```

CSS部分：
```css
* {
    padding: 0;
    margin: 0;
}

ul {
    list-style: none;
}

.container {
    width: 750px;
    margin: 0 auto;
}

.container ul li {
    float: left;
    width: 93.75px;
    height: 30px;
    background-color: green;
}

@media screen and (max-width: 767px) {
    .container {
        width: 100%;
    }
    .container ul li {
        width: 33.33%;
    }
}
```

## 2. Bootstrap 前端开发框架

### 2.1 Bootstrap 简介

Bootstrap 来自 Twitter（推特），是目前最受欢迎的前端框架。Bootstrap 是基于 HTML、CSS 和 JAVASCRIPT 的，它简洁灵活，使得 Web 开发更加快捷。

- 中文官网：http://www.bootcss.com/
- 官网：http://getbootstrap.com/
- 推荐使用：http://bootstrap.css88.com/

框架：顾名思义就是一套架构，它有一套比较完整的网页功能解决方案，而且控制权在框架本身，有预制样式库、组件和插件。使用者要按照框架所规定的某种规范进行开发。  

Bootstrap 优点：
- 标准化的 html+css 编码规范
- 提供了一套简洁、直观、强悍的组件
- 有自己的生态圈，不断的更新迭代
- 让开发更简单，提高了开发的效率

版本：
- 2.x.x：停止维护，兼容性好，代码不够简洁，功能不够完善。
- 3.x.x：目前使用最多，稳定，但是放弃了 IE6-IE7。对 IE8 支持但是界面效果不好，偏向用于开发响应式布局、移动设备优先的 WEB 项目。
- 4.x.x：最新版，目前还不是很流行。

### 2.2 Bootstrap 使用

在现阶段我们还没有接触 JS 相关课程，所以我们只考虑使用它的样式库。

控制权在框架本身，使用者要按照框架所规定的某种规范进行开发。

Bootstrap 使用四步曲：
1. 创建文件夹结构
2. 创建 html 骨架结构
3. 引入相关样式文件
4. 书写内容

### 2.2.1 创建文件夹结构
    ```
    bootstrap
        css
        fonts
        js
    css
    images
    index.html
    ```

### 2.2.2 创建 html 骨架结构
    ```html
    <!--要求当前网页使用IE浏览器最高版本的内核来渲染-->
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <!--视口的设置：视口的宽度和设备一致，默认的缩放比例和PC端一致，用户不能自行缩放-->
    <meta name="viewport" content="width=device-width, initial-scale=1, user-scalable=0">
    <!--[if lt IE 9]>
        <!-- 解决ie9以下浏览器对html5新增标签的不识别，并导致CSS不起作用的问题 -->
        <script src="https://oss.maxcdn.com/html5shiv/3.7.2/html5shiv.min.js"></script>
        <!--解决ie9以下浏览器对css3 Media Query 的不识别-->
        <script src="https://oss.maxcdn.com/respond/1.4.2/respond.min.js"></script>
    <![endif]-->
    ```

### 2.2.3 引入相关样式文件
    ```html
    <!-- Bootstrap 核心样式-->
    <link rel="stylesheet" href="bootstrap/css/bootstrap.min.css">
    ```

### 2.2.4 书写内容

- 直接拿 Bootstrap 预先定义好的样式来使用
- 修改 Bootstrap 原来的样式，注意权重问题
- 学好 Bootstrap 的关键在于知道它定义了哪些样式，以及这些样式能实现什么样的效果

### 2.3 布局容器

Bootstrap 需要为页面内容和栅格系统包裹一个 `.container` 容器，Bootstrap 预先定义好了这个类，叫 `.container`。它提供了两个作此用处的类：
1. `container` 类
    - 响应式布局的容器。固定宽度
    - 大屏( >=1200px ) 宽度定为 1170px
    - 中屏( >=992px ) 宽度定为 970px
    - 小屏( >=768px ) 宽度定为 750px
    - 超小屏(100%)
2. `container-fluid` 类
    - 流式布局容器。百分百宽度
    - 占据全部视口（viewport）的容器。
    - 适合于单独做移动端开发。

## 3. Bootstrap 栅格系统

### 3.1 栅格系统简介

栅格系统英文为 “grid systems”，也有人翻译为“网格系统”，它是指将页面布局划分为等宽的列，然后通过列数的定义来模块化页面布局。  

Bootstrap 提供了一套响应式、移动设备优先的流式栅格系统，随着屏幕或视口（viewport）尺寸的增加，系统会自动分为最多 12 列。

Bootstrap 里面的 container 宽度是固定的，但是在不同屏幕下，container 的宽度不同，我们再把 container 划分为 12 等份。

### 3.2 栅格选型参数

栅格系统用于通过一系列的行（row）与列（column）的组合来创建页面布局，你的内容就可以放入这些创建好的布局中。

| -                     | 超小屏幕（手机）`< 768px` | 小屏设备（平板）`>= 768px` | 中等屏幕（桌面显示器）`>= 992px` | 宽屏设备（大桌面显示器）`>= 1200px` |
| --------------------- | ------------------------- | -------------------------- | -------------------------------- | ----------------------------------- |
| `.container` 最大宽度 | 自动（100%）              | 750 px                     | 970 px                           | 1170 px                             |
| 类前缀                | `.col-xs-`                | `.col-sm-`                 | `.col-md-`                       | `.col-lg-`                          |   
| 列（column）数        | 12                        | 12                         | 12                               | 12                                  |  

- 行（row）必须放到 container 布局容器里面。
- 我们实现列的平均划分，需要给列添加类前缀。
- 按照不同屏幕划分为 1~12 等份。
- 行（row）可以去除父容器作用 15px 的边距。
- xs-extra small：超小；sm-small：小；md-medium：中等；lg-large：大。
- 列（column）大于12，多余的 “列（column）”所在的元素将被作为一个整体另起一行排列。
- 每一列默认有左右15像素的 `padding`。
- 可以同时为一列指定多个设备的类名，以便划分不同份数。例如 `class="col-md-4 col-sm-6"`。

```html
<!-- 如果孩子的份数相加等于12，则孩子能占满整个的 container 的宽度 -->
<!-- 有12个（1+2+3+6=12），则可以占满一行，不同份数表示了所占比例 -->
<div class="row">
    <div class="col-lg-1">1</div>
    <div class="col-lg-2">2</div>
    <div class="col-lg-3">3</div>
    <div class="col-lg-6">4</div>
</div>
<!-- 如果孩子的份数相加小于12，则占不满整个的 container 的宽度，会有空白 -->
<!-- 不足12个，则空出多余 -->
<div class="row">
    <div class="col-lg-3">1</div>
    <div class="col-lg-3">2</div>
    <div class="col-lg-3">3</div>
    <div class="col-lg-2">4</div>
</div>
<!-- 如果孩子的份数相加大于12，则多于的那一列会另起一行显示 -->
<!-- 超出12个，则放到下一行 -->
<div class="row">
    <div class="col-lg-3">1</div>
    <div class="col-lg-3">2</div>
    <div class="col-lg-3">3</div>
    <div class="col-lg-4">4</div>
</div>

<!-- `col-lg-3 col-md-4 col-sm-6 col-xm-12` 表示随着屏幕尺寸的缩小，每一行能放的盒子变为 4、3、2、1。-->
<!-- 有12个，则可以占满一行 -->
<div class="row">
    <div class="col-lg-3 col-md-4 col-sm-6 col-xs-12">1</div>
    <div class="col-lg-3 col-md-4 col-sm-6 col-xs-12">2</div>
    <div class="col-lg-3 col-md-4 col-sm-6 col-xs-12">3</div>
    <div class="col-lg-3 col-md-4 col-sm-6 col-xs-12">4</div>
</div>
```

### 3.3 列嵌套

栅格系统内置的栅格系统将内容再次嵌套。简单理解就是一个列内再分成若干份小列。我们可以通过添加一个新的 `.row` 元素和一系列 `.col-sm-*` 元素到已经存在的 `.col-sm-*` 元素内。

我们列嵌套最好加1个行row，这样可以取消父元素的padding值，而且高度自动和父级一样高。

```html
<!-- 列嵌套 -->
<div class="col-sm-4">
    <div class="row">
        <div class="col-sm-6">小列</div>
        <div class="col-sm-6">小列</div>
    </div>
</div>
```

```html
<div class="container">
    <div class="row">
        <div class="col-md-4">
            <!-- 列嵌套最好加1个行row，这样可以去掉父元素的padding值，而且高度自动和父级一样高 -->
            <div class="row">
                <div class="col-md-4">第一小列</div>
                <div class="col-md-8">第二小列</div>
            </div>
        </div>
        <div class="col-md-4">第二列</div>
        <div class="col-md-4">第三列</div>
    </div>
</div>
```

### 3.4 列偏移

使用 `.col-md-offset-*` 类可以将列向右侧偏移。这些类实际是通过使用 `*` 选择器为当前元素增加了左侧的边距（margin）。

```html
<!-- 列偏移 -->
<div class="row">
    <div class="col-lg-4">1</div>
    <div class="col-lg-4 col-lg-offset-4">2</div>
</div>
```

```html
<div class="container">
    <div class="row">
        <div class="col-md-4">左侧</div>
        <div class="col-md-4 col-md-offset-4">右侧</div>
    </div>
    <div class="row">
        <div class="col-md-8 col-md-offset-2">中间盒子</div>
    </div>
</div>
```

### 3.5 列排序

通过使用 `.col-md-push-*` 和 `.col-md-pull-*` 类就可以很容易地改变列（`column`）的顺序。

```html
<!-- 列排序 -->
<div class="row">
    <div class="col-lg-4 col-lg-push-8">左侧</div>
    <div class="col-lg-8 col-lg-pull-4">右侧</div>
</div>
```

```html
<div class="container">
    <div class="row">
        <div class="col-md-4">左侧盒子</div>
        <div class="col-md-8">右侧盒子</div>
    </div>
    <div class="row">
        <div class="col-md-4 col-md-push-8">左侧盒子</div>
        <div class="col-md-8 col-md-pull-4">右侧盒子</div>
    </div>
</div>
```

### 3.6 响应式工具

为了加快对移动设备友好的页面开发工作，利用媒体查询功能，并使用这些工具类可以方便的针对不同设备展示或隐藏页面内容。

| 类名         | 超小屏 | 小屏 | 中屏 | 大屏 |
| ------------ | ------ | ---- | ---- | ---- |
| `.hidden-xs` | 隐藏   | 可见 | 可见 | 可见 |
| `.hidden-sm` | 可见   | 隐藏 | 可见 | 可见 |
| `.hidden-md` | 可见   | 可见 | 隐藏 | 可见 |
| `.hidden-lg` | 可见   | 可见 | 可见 | 隐藏 | 

与之相反的是 visible-xs、visible-sm、visible-md、visible-lg 是显示某个页面内容。

Bootstrap 其他（按钮、表单、表格） 请参考 Bootstrap 文档。

## 4. 阿里百秀首页案例

### 4.1 技术选型

方案：我们采取响应式页面开发方案
技术：bootstrap 框架
设计图：本设计图采用 1280px 设计尺寸

### 4.2 屏幕划分分析

- 屏幕缩放发现中屏幕和大屏幕布局是一致的。因此我们列定义为 col-md- 就可以了， md 是大于等于 970 以上的
- 屏幕缩放发现小屏幕布局发生变化，因此我们需要为小屏幕根据需求改变布局。
- 屏幕缩放发现超小屏幕布局又发生变化，因此我们需要为超小屏幕根据需求改变布局
- 策略：我们先布局 md 以上的 pc 端布局，最后根据实际需求在修改小屏幕和超小屏幕的特殊布局样式。

### 4.3 container 宽度修改

因为本效果图采取 1280 的宽度，而 Bootstrap 里面 container 宽度最大为 1170px，因此我们需要手动改下 container 宽度。
```css
/* 利用媒体查询修改 container 宽度适合效果图宽度 */
@media screen and (min-width: 1280px) { 
    .container { 
        width: 1280px; 
    } 
}
```

## 5. 移动端布局总结

移动端技术选型

- 流式布局（百分比布局）
- flex 弹性布局（推荐）
- rem 适配布局（推荐）
- 响应式布局

建议：我们选取一种主要技术选型，其他技术做为辅助，这种混合技术开发。
