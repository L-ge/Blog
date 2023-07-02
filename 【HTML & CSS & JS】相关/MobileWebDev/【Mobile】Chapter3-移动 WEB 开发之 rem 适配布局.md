# 站在巨人的肩膀上

> [黑马程序员pink老师前端入门教程，零基础必看的h5(html5)+css3+移动端前端视频教程](https://www.bilibili.com/video/BV14J4114768/)

> [MurphyChen's Notes](https://github.com/Hacker-C/notes)


# （三）移动 WEB 开发之 rem 适配布局

## 1. rem 基础

`rem`（root em）是一个相对单位，类似于 `em`，`em` 是父元素字体大小。

`em` 是相对于父元素的字体大小来说的，比如父元素的 font-size: 12px，那么如果子元素设置了 width: 10em，就是设置了 width: 120px。

不同的是 `rem` 的基准是 **相对于 html 元素的字体大小**。  

比如，根元素（html）设置 `font-size: 12px`，非根元素设置 `width: 2rem`，则换成 `px` 就是 `24px`。  
rem的优势：父元素文字大小可能不一致，但是整个页面只有一个 html，可以很好来控制整个页面的元素大小
。

```css
/* 根 html 为 12px */
html {
    font-size: 12px;
}
/* 此时 div 的字体大小就是 24px */
div {
    font-size: 2rem;
}
```

## 2. 媒体查询

### 2.1 什么是媒体查询

媒体查询（Media Query）是 CSS3 的新语法。
- 使用 `@media` 查询，可以针对不同的媒体类型定义不同的样式。
- `@media` **可以针对不同的屏幕尺寸设置不同的样式**。
- 当你重置浏览器大小的过程中，页面也会根据浏览器的宽度和高度重新渲染页面。
- 目前针对很多苹果手机、Android 手机、平板等设备都用得到多媒体查询。

### 2.2 语法规范

```css
@media mediatype and|not|only (media feature) {
    CSS3-Code;
}
```
- 以 `@media` 开头，注意 @ 符号。
- mediatype 是媒体类型。
- 关键字 `and`、`not`、`only`。
- `media feature` 媒体特性，必须有小括号包含。

#### 2.2.1 mediatype 查询类型

将不同的终端设备划分成不同的类型，称为媒体类型。

| 值       | 解释说明                           |
| -------- | ---------------------------------- |
| `all`    | 用于所有设备                       |
| `print`  | 用于打印机和打印浏览               |
| `screen` | 用于电脑屏幕，平板电脑，智能手机等 |

#### 2.2.2 关键字

关键字将媒体类型或多个媒体特性连接到一起做为媒体查询的条件。

- `and`：可以将多个媒体特性连接到一起，相当于 “且” 的意思。
- `not`：排除某个媒体类型，相当于 “非” 的意思，可以省略。
- `only`：指定某个特定的媒体类型，可以省略。

#### 2.2.3 媒体特性

每种媒体类型都具体各自不同的特性，根据不同媒体类型的媒体特性设置不同的展示风格。我们暂且了解三个。

注意他们要加小括号包含。

| 值          | 解释说明                           |
| ----------- | ---------------------------------- |
| `width`     | 定义输出设备中页面可见区域的宽度   |
| `min-width` | 定义输出设备中页面最小可见区域宽度 |
| `max-width` | 定义输出设备中页面最大可见区域宽度 |

**媒体查询的主要价值：媒体查询可以根据不同的屏幕尺寸来改变不同的样式。**

例子：
```css
/* 在屏幕上并且最大的宽度是800像素，设置我们想要的样式 */
@media screen and (max-width: 800px) {
    body {
        background-color: pink;
    }
}

/* 在屏幕上并且最小的宽度是540像素并且最大的宽度是969像素（540<=width<=969），设置我们想要的样式 */
@media screen and (min-width: 540px) and (max-width: 969px) {
    body {
        background-color: purple;
    }
}
```

注意：为了防止混乱，媒体查询我们要按照从小到大或者从大到小的顺序来写，但是我们最喜欢的还是从小到大来写，这样代码更简洁。

### 2.3 媒体查询 + rem 实现元素动态大小变化

`rem` 单位是跟着 `html` 来走的，有了 `rem` 页面元素可以设置不同大小尺寸。

媒体查询可以根据不同设备宽度来修改样式。 

**媒体查询 + rem** 就可以实现不同设备宽度，实现页面元素大小的动态变化。

例子：
```css
@media screen and (min-width: 320px) {
    html {
        font-size: 50px;
    }
}
@media screen and (min-width: 640px) {
    html {
        font-size: 100px;
    }
}

.top {
    height: 1rem;
    font-size: .5rem;
    background-color: green;
    color: #fff;
    text-align: center;
    line-height: 1rem;
}
```

### 2.4 引入资源（理解）

当样式比较繁多的时候，我们可以针对不同的媒体使用不同 stylesheets（样式表）。

原理，**就是直接在 link 中判断设备的尺寸，然后引用不同的 css 文件**。

#### 2.4.1 语法规范

```html
<link rel="stylesheet" media="mediatype and|not|only (media feature)" href="mystylesheet.css">
```

#### 2.4.2 示例

```html
<link rel="stylesheet" href="style320.css" media="screen and (min-width: 320px)">
<link rel="stylesheet" href="style640.css" media="screen and (min-width: 640px)">
```
- 引入资源就是针对于不同的屏幕尺寸，调用不同的css文件

## 3. Less 基础

### 3.1 维护 CSS 的弊端

CSS 是一门非程序式语言，没有变量、函数、SCOPE（作用域）等概念。

- CSS 需要书写大量看似没有逻辑的代码，CSS 冗余度是比较高的。
- 不方便维护及扩展，不利于复用。
- CSS 没有很好的计算能力。
- 非前端开发工程师来讲，往往会因为缺少 CSS 编写经验而很难写出组织良好且易于维护的 CSS 代码项目。

### 3.2 Less 介绍

Less（Leaner Style Sheets 的缩写）是一门 CSS 扩展语言，也称为 CSS 预处理器。  

作为 CSS 的一种形式的扩展，它并没有减少 CSS 的功能，而是在现有的 CSS 语法上，为 CSS 加入程序式语言的特性。  

它在 CSS 的语法基础之上，**引入了变量，Mixin（混入），运算以及函数等功能，大大简化了 CSS 的编写，并且降低了 CSS 的维护成本**，就像它的名称所说的那样，Less 可以让我们用更少的代码做更多的事情。  

Less 中文网址: http://lesscss.cn  

常见的 CSS 预处理器：Sass、Less、Stylus  

一句话：**Less 是一门 CSS 预处理语言，它扩展了 CSS 的动态特性。**

### 3.3 Less 安装（注意如果使用 vscode 无需安装 less）

1. 安装 nodejs，可选择版本(8.0)，网址：http://nodejs.cn/download/
2. 检查是否安装成功，使用 cmd 命令（win10 是 window + r 打开运行输入cmd） 
    - 输入“node –v”查看版本即可。
3. 基于 nodejs 在线安装 Less，使用 cmd 命令“npm install -g less”即可。
4. 检查是否安装成功，使用 cmd 命令“lessc -v”查看版本即可。

Less 使用：我们首先新建一个后缀名为 less 的文件，在这个 less 文件里面书写 less 语句。

- Less 变量
- Less 编译
- Less 嵌套
- Less 运算

### 3.4 Less 变量

变量是指没有固定的值，可以改变的。因为我们 CSS 中一些颜色和数值等经常使用。

```less
@变量名:值;
```

例子：
```less
// 定义一个粉色变量
@color: pink;
@font14: 14px;
body {
    background-color: @color;
}
div {
    background-color: @color;
    font-size: @font14;
}
```

#### 3.4.1 变量命名规范

- 必须有 `@` 为前缀
- 不能包含特殊字符
- 不能以数字开头
- 大小写敏感

```less
@color: pink;
```

#### 3.4.2 变量使用规范

```less
//直接使用
body{
    color:@color;
}
a:hover{
    color:@color;
}
```

### 3.5 Less 编译

本质上，Less 包含一套自定义的语法及一个解析器，用户根据这些语法定义自己的样式规则，这些规则最终会通过解析器，编译生成对应的 CSS 文件。  

所以，我们需要把我们的 less 文件，编译生成为 css 文件，这样我们的 html 页面才能使用。  

vscode Less 插件：
- Easy LESS 插件用来把 less 文件编译为 css 文件。
- 安装完毕插件，重新加载下 vscode。
- 只要保存一下 Less 文件，会自动生成 CSS 文件。

在 VS Code 中，**使用 Easy Less 插件** 可以即时编译生成 CSS 文件，再引入即可。

### 3.6 Less 嵌套

我们经常用到选择器的嵌套。
```css
.header .logo {
    width: 300px;
}
```

Less 嵌套写法：
```less
.header {
    .logo {
        width: 100px;
    }
}
```

等同于：
```css
.header .logo {
     width: 100px;
}
```

如果遇见（交集|伪类|伪元素选择器）
- 内层选择器的前面没有 & 符号，则它被解析为父选择器的后代；
- 如果有 & 符号，它就被解析为父元素自身或父元素的伪类。

```css
a:hover{
    color:red;
}
```

Less 嵌套写法:
```less
a{
    &:hover{
        color:red;
    }
}
```

### 3.7 Less 运算

任何数字、颜色或者变量都可以参与运算。Less 提供了加（`+`）、减（`-`）、乘（`*`）、除（`/`）算数运算。

```
/*Less 里面写*/
@witdh: 10px + 5;
div {
    border: @witdh solid red;
}
/*生成的css*/
div {
    border: 15px solid red;
}
/*Less 甚至还可以这样 */
width: (@width + 5) * 2;
```

注意：
- 乘号（*）和除号（/）的写法。
- 运算符中间左右都有个空格隔开。
- 对于两个不同的单位的值之间的运算，运算结果的值取第一个值的单位。
- 如果两个值之间只有一个值有单位，则运算结果就取该单位。

## 4. rem 适配方案

1. 让一些不能等比自适应的元素，达到当设备尺寸发生改变的时候，等比例适配当前设备。
2. 使用媒体查询根据不同设备按比例设置 html 的字体大小，然后页面元素使用 rem 做尺寸单位，当 html 字体大小变化元素尺寸也会发生变化，从而达到等比缩放的适配。

### 4.1 rem 实际开发适配方案

1. 按照设计稿与设备宽度的比例，动态计算并设置 html 根标签的 font-size 大小;（媒体查询）
2. CSS 中，设计稿元素的宽、高、相对位置等取值，按照同等比例换算为 rem 为单位的值。

### 4.2 rem 适配方案技术使用（市场主流）

技术方案1：
- less
- 媒体查询
- rem

技术方案2（推荐）：
- flexible.js
- rem

总结：
1. 两种方案现在都存在。
2. 方案 2 更简单，现阶段大家无需了解里面的 js 代码。

### 4.3 rem 实际开发适配方案1

rem + 媒体查询 + less技术

#### 4.3.1 设计稿常见尺寸宽度

| 设备             | 常见宽度   |
| ---------------- | ---------- |
| iphone 4 5       | 640px      |
| **iphone 6 7 8** | **750px**  |
| Android          | 常见320px、360px、375px、384px、400px、414px、500px、720px。**大部分4.7~5寸的安卓设备为720px** |

一般情况下，我们以一套或两套效果图适应大部分的屏幕，放弃极端屏或对其优雅降级，牺牲一些效果。 

**现在基本以 750 为准**。

#### 4.3.2 动态设置 html 标签 font-size 大小

1. 假设设计稿是750px  
2. 假设我们把整个屏幕划分为15等份（划分标准不一，可以是20份也可以是10等份）
3. 每一份作为 html 字体大小，这里就是 50px  
4. 那么在 320px 设备的时候，字体大小为 320/15 就是 21.33px
5. 用我们页面元素的大小除以不同的 html 字体大小会发现他们比例还是相同的
6. 比如我们以 750 为标准设计稿
7. 一个 100*100 像素的页面元素在 750 屏幕下，就是 100/50，转换为 rem 是 2rem * 2rem，比例是 1 比 1
8. 320 屏幕下，html 字体大小为 21.33，则 2rem = 42.66px，此时宽和高都是42.66，但是宽和高的比例还是 1 比 1
9. 但是已经能实现不同屏幕下，页面元素盒子等比例缩放的效果

#### 4.3.3 元素大小取值方法

- 最后的公式：页面元素的rem值 = 页面元素值（px）/（屏幕宽度 / 划分的份数）
- 屏幕宽度 / 划分的份数 就是 html font-size 的大小
- 或者：页面元素的 rem 值 = 页面元素值（px） / html font-size 字体大小

具体步骤：
1. 首先选一套标准尺寸，例如以 750 为准
2. 用 **屏幕尺寸** 除以 **划分的份数**，得到 html 里面的文字大小。但是我们知道，不同屏幕下得到的文字大小是不一样的。
3. **页面元素的 rem 值** = **页面元素在750像素下的px值** / **html里面的文字大小**。

## 5. 苏宁网首页案例制作

访问地址：m.suning.com

### 5.1 技术选型

- 方案：我们采取单独制作移动页面方案
- 技术：布局采用 `rem` 适配布局（less + rem + 媒体查询）
- 设计图：本设计图采用 750px 设计尺寸

### 5.2 搭建相关文件夹结构

- css 文件夹
- images 文件夹
- upload 文件夹
- index.html

### 5.3 设置视口标签以及引入初始化样式

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no, maximum-scale=1.0, minimum-sacle=1.0">
<link rel="stylesheet" href="css/normalize.css">
```

### 5.4 设置公共 common.less 文件

1. 新建 common.less 设置好最常见的屏幕尺寸，利用媒体查询设置不同的 html 字体大小。因为除了首页其他页面也需要，所以写成公共样式。
2. 我们关心的尺寸有320px、360px、375px、384px、400px、414p、424px、480px、540px、720px、750px
3. 划分的份数我们定为15等份
4. 因为我们 pc 端也可以打开我们苏宁移动端首页，我们默认 html 字体大小为 50px，**注意这句话写到最上面**

```less
// 设置常见屏幕尺寸，修改 html 里面的文字大小
// PC屏幕下：
// 一定要写在最上面（因为样式有层叠性）
html {
    font-size: 50px;
}
// 划分份数：15
// 适配屏幕：320px、360px、375px、384px、400px、414p、424px、480px、540px、720px、750px
@num: 15;
// 320
@media screen and (min-width: 320px) {
    html {
        font-size: (320px / @num);
    }
}
// 360
@media screen and (min-width: 360px) {
    html {
        font-size: (360px / @num);
    }
}
// 375
...
```

### 5.5 新建 index.less 文件

1. 新建 index.less，写首页样式
2. 将刚才设置好的 common.less 引入到 index.less 里面，语法如下：
    ```less
    // 在 index.less 中导入 common.less
    @import "common";
    ```
3. 生成 index.css 引入到 index.html 里面（common.less 生成的 common.css 就不用引入到 index.html 里面了）

### 5.6 body 样式

```css
body {
    min-width: 320px;
    width:15rem;
    margin: 0 auto;
    line-height: 1.5;
    font-family: Arial,Helvetica;
    background: #F2F2F2;
}
```

## 6. rem 适配方案2

### 6.1 简洁高效的 rem 适配方案 flexible.js

技术方案1（less+媒体查询+rem）效果很好，但是过于繁琐。因此介绍第二种 rem 方案。

手机淘宝团队出的简洁高效移动端适配库。  

我们再也 **不需要在写不同屏幕的媒体查询**，因为里面 js 做了处理。  

它的原理是把 **当前设备划分为10等份**，但是不同设备下，比例还是一致的。  

我们要做的，**就是确定好我们当前设备的 html 文字大小就可以了**。  

比如当前设计稿是750px，那么我们只需要把 html 文字大小设置为 75px(750px/10) 就可以里面页面元素 rem 值：页面元素的px值/ 75。

剩余的，**让flexible.js来去算**。

github 地址：https://github.com/amfe/lib-flexible

### 6.2 使用适配方案2制作苏宁移动端首页

#### 6.2.1 技术选型

- 方案：我们采取单独制作移动页面方案
- 技术：布局采取rem适配布局2（flexible.js + rem）
- 设计图：本设计图采用 750px 设计尺寸

#### 6.2.2 搭建相关文件夹结构

- css 文件夹
- images 文件夹
- js 文件夹
- upload 文件夹
- index.html

#### 6.2.3 设置视口标签以及引入初始化样式还有js文件

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no, maximum-scale=1.0, minimum-sacle=1.0">
<link rel="stylesheet" href="css/normalize.css">
<link rel="stylesheet" href="css/index.css">
```

我们页面需要引入这个 js 文件：
```
在 index.html 中 引入 flexible.js 这个文件
<script src=“js/flexible.js”></script>
```

#### 6.2.4 body样式

```css
body {
    min-width: 320px;
    max-width: 750px;   // 按 750 的设计稿。
    width: 10rem;       // flexible 给我们划分了 10 等份，按 750 的设计稿，750px 也就是 10rem。
    margin: 0 auto;
    line-height: 1.5;
    font-family: Arial,Helvetica;
    background: #F2F2F2;
}
```

#### 6.2.5 注意事项

flexible.js 按照屏幕尺寸修改 html 的 `font-size` 大小，当处于 PC 窗口时，宽度会过大。需要额外设置一个媒体查询：
```css
/* 若设备屏幕超过 750px，则按照 750 设计稿布局，不会让页面超过 750px */
@media screen and (min-width: 750px) {
    html {
        font-size: 75px!important;
    }
}
```

### 6.3 VSCode px 转换 rem 插件 cssrem

自动转化，在输入的时候，下拉框选择即可。（这个插件像是增加了代码提示一样）

但要注意，该插件默认的 html 文字大小 cssroot 是 16px。按需修改（设置 html 字体大小基准值：打开设置（快捷键是 ctrl + 逗号），然后点击右边更多设置，然后选择打开 settings.json，搜索 cssroot。修改完成后重启 vscode。也可以通过更改该插件的配置）。
