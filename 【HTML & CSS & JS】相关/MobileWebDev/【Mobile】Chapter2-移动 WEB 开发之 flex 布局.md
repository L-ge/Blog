# 站在巨人的肩膀上

> [黑马程序员pink老师前端入门教程，零基础必看的h5(html5)+css3+移动端前端视频教程](https://www.bilibili.com/video/BV14J4114768/)

> [MurphyChen's Notes](https://github.com/Hacker-C/notes)


# （二）移动 WEB 开发之 flex 布局

## 1. flex 布局体验

### 1.1 传统布局与 flex 布局

#### 1.1.1 传统布局

- 兼容性好
- 布局繁琐
- 局限性，不能在移动端很好地布局

#### 1.1.2 flex 弹性布局

- 操作方便，布局极为简单，移动端应用很广泛
- PC 端浏览器支持情况较差
- IE11 或更低版本，不支持或仅部分支持

#### 1.1.3 建议

1. 如果是 PC 端页面布局，我们还是传统布局。
2. 如果是移动端布局或者不考虑兼容性的 PC 端页面布局，我们还是使用 flex 弹性布局。

## 2. flex 布局原理

flex 是 flexible Box 的缩写，意为“弹性布局”，用来为盒状模型提供最大的灵活性，**任何一个容器都可以指定为 flex 布局**。

- 当我们为父盒子设为 `flex` 布局以后，子元素的 `float`、`clear` 和 `vertical-align` 属性将失效。
- 伸缩布局 = 弹性布局 = 伸缩盒布局 = 弹性盒布局 = flex 布局。

采用 Flex 布局的元素，称为 **Flex容器**（flex container），简称“容器”。它的所有子元素自动成为容器成员，称为 Flex 项目（flex item），简称 “项目”。

- 子容器可以横向排列也可以纵向排列。

总结 flex 布局原理：**就是通过给父盒子添加 flex 属性，来控制子盒子的位置和排列方式**。

## 3. flex 布局父项常见属性

### 3.1 常见父项属性

以下有 6 个属性是对父元素设置的。
- `flex-direction`：设置主轴的方向。
- `justify-content`：设置主轴上的子元素排列方式。
- `flex-wrap`：设置子元素是否换行。
- `align-content`：设置侧轴上的子元素的排列方式（多行）。
- `align-items`：设置侧轴上的子元素排列方式（单行）。
- `flex-flow`：复合属性，相当于同时设置了 `flex-direction` 和 `flex-wrap`。

### 3.2 flex-direction 设置主轴的方向

#### 3.2.1 主轴与侧轴

在 flex 布局中，是分为主轴和侧轴两个方向，同时的叫法有：行和列、x 轴和 y 轴。
- 默认主轴方向就是 x 轴方向，水平向右。
- 默认侧轴方向就是 y 轴方向，垂直向下。

#### 3.2.2 属性值

`flex-direction` 属性决定主轴的方向（即项目的排列方向）。  

注意：主轴和侧轴是会变化的，就看 `flex-direction` 设置谁为主轴，剩下的就是侧轴。而我们的子元素是跟着主轴来排列的。

| 属性值         | 说明             |
| -------------- | ---------------- |
| row            | 默认值，从左到右 |
| row-reverse    | 从右到左         |
| column         | 从上到下         |
| column-reverse | 从下到上         |

### 3.3 justify-content 设置主轴上的子元素排列方式  

`justify-content` 属性定义了项目在主轴上的对齐方式。

**注意：使用这个属性之前一定要确定好主轴是哪个。**

| 属性值        | 说明                                                               |
| ------------  | ------------------------------------------------------------------ |
| flex-start    | 默认值。从头部开始，如果主轴是 x 轴，则从左到右                    |
| flex-end      | 从尾部开始排列（顺序不变，只是贴着尾部了，和 row-reverse 是不同的）|
| center        | 在主轴居中对齐（如果主轴是 x 轴，则水平居中）                      |
| space-around  | 平分剩余空间                                                       |
| space-between | **先两边贴边，再平分剩余空间（重要）**                             |

### 3.4 flex-wrap 设置子元素是否换行

默认情况下，项目都排在一条线（又称“轴线”）上。`flex-wrap` 属性定义，flex 布局中默认是不换行的。  

如果父盒子一行上装不开，则会缩小子元素的宽度，从而仍然一行显示。

| 属性值 | 说明           |
| ------ | -------------- |
| nowrap | 默认值，不换行 |
| wrap   | 换行           |

### 3.5 align-items 设置侧轴上的子元素排列方式（单行）

该属性是控制子项在侧轴（默认是 y 轴）上的排列方式，在子项为单项（单行）的时候使用。

| 属性值     | 说明                    |
| ---------- | ----------------------- |
| flex-start | 从上到下                |
| flex-end   | 从下到上                |
| center     | 挤在一起居中（垂直居中）|
| strech     | 拉伸（默认值）          |

strech：拉伸，但是子盒子不要给高度（侧轴是 y 轴的情况下），子元素会拉伸到和父元素一样高。

### 3.6 align-content 设置侧轴上的子元素的排列方式（多行）

设置子项在侧轴上的排列方式并且只能用于子项出现换行的情况（多行），在单行下是没有效果的。

| 属性值        | 说明                                   |
| ------------- | -------------------------------------- |
| flex-start    | 默认值。在侧轴的头部开始排列           |
| flex-end      | 在侧轴的尾部开始排列                   |
| center        | 在侧轴中间显示                         |
| space-around  | 子项在侧轴平分剩余空间                 |
| space-between | 子项在侧轴先分布在两头，再平分剩余空间 |
| strech        | 设置子项元素高度平分父元素高度         |

### 3.7 align-content 和 align-items 区别

- `align-items` 适用于单行情况下，只有上对齐、下对齐、居中和拉伸。
- `align-content` 适用于换行（多行）的情况下（单行情况下无效），可以设置上对齐、下对齐、居中、拉伸以及平均分配剩余空间等属性值。
- 总结就是单行找 `align-items`，多行找 `align-content`。

### 3.8 flex-flow

`flex-flow` 属性是 `flex-direction` 和 `flex-wrap` 属性的复合属性。

```css
flex-flow: row wrap;
```

- `flex-direction`：设置主轴的方向。
- `justify-container`：设置主轴上的子元素排列方式。
- `flex-wrap`：设置子元素是否换行。
- `align-content`：设置侧轴上的子元素的排列方式（多行）。
- `align-items`：设置侧轴上的子元素排列方式（单行）。
- `flex-flow` ：复合属性，相当于同时设置了 `flex-direction` 和 `flow-wrap`。

## 4. flex 布局子项常见属性

- flex 子项目占的份数。
- align-self 控制子项自己在侧轴的排列方式。
- order 属性定义子项的排列顺序（前后顺序）。

### 4.1 flex 属性

`flex` 属性定义子项目分配剩余空间，用 `flex` 表示占多少份数。

```css
.item {
    flex: <number>;     /*默认值 0*/
}
```
- 也可以写百分比（相对于父级来说，比如20%，也就是分成了5份）。

例如要平分一个盒子，则不给定子元素宽度（高度），然后给每一个子元素设置属性：`flex: 1`。（即如果有3个子盒子，则总共是3份，每个盒子占1份，也就是平均分配了）

### 4.2 align-self 控制子项自己在侧轴上的排列方式

`align-self` 属性允许单个项目有与其他项目不一样的对齐方式，可覆盖 `align-items` 属性。

默认值为 `auto`，表示继承父元素的 `align-items` 属性，如果没有父元素，则等同于 `strech`。

```css
span:nth-child(2) {
    /* 设置自己在侧轴上的排列方式 */
    align-self: flex-end;
}
```

### 4.3 order 属性定义项目的排列顺序

数值越小，排列越靠前，默认为 `0`。

**注意：和 `z-index` 不一样。**

```css
.item {
    order: <number>;
}
```

## 5. 携程网首页案例制作

访问地址：m.ctrip.com

### 5.1 技术选型

- 方案：我们采取单独制作移动页面方案
- 技术：布局采用 flex 布局

### 5.2 设置视口标签以及引入初始化样式

```html
<meta name="viewport" content="width=device-width, user-scalable=no, 
initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
<link rel="stylesheet" href="css/normalize.css">
<link rel="stylesheet" href="css/index.css">
```

### 5.3 常用初始化样式

```css
body {
    max-width: 540px;
    min-width: 320px;
    margin: 0 auto;
    font: normal 14px/1.5 Tahoma,"Lucida Grande",Verdana,"Microsoft Yahei",STXihei,hei;
    color: #000;
    background: #f2f2f2;
    overflow-x: hidden;
    -webkit-tap-highlight-color: transparent;
}
```
- overflow-x：水平滚动条

其他知识点：
- 固定的盒子应该有宽度。
- 固定定位跟父级没有关系，它以屏幕为准。

### 5.4 常见 flex 布局思路

上下结构的布局（上面是一张小图片，下面是文字）：
- 把主轴设为 y 轴；
- 把侧轴设为居中对齐。

```html
    <ul class="local-nav">
        <li>
        <a href="#" title="景点·玩乐">
            <span class="local-nav-icon-icon1"></span>
            <span>景点·玩乐</span>
        </a>
    </li>
    <li>
        <a href="#" title="景点·玩乐">
            <span class="local-nav-icon-icon2"></span>
            <span>景点·玩乐</span>
        </a>
    </li>
    <li>
        <a href="#" title="景点·玩乐">
            <span class="local-nav-icon-icon3"></span>
            <span>景点·玩乐</span>
        </a>
    </li>
</ul>
```

```css
.local-nav {
    display: flex;
    height: 64px;
    margin: 3px 4px;
    background-color: #fff;
    border-radius: 8px;
}

.local-nav li {
    flex: 1;
}

.local-nav a {
    display: flex;
    flex-direction: column;
    /* 侧轴居中对齐 因为是单行 */
    align-items: center;
    font-size: 12px;
}

.local-nav li [class^="local-nav-icon"] {
    width: 32px;
    height: 32px;
    background-color: pink;
    margin-top: 8px;
    background: url(../images/localnav_bg.png) no-repeat 0 0;
    background-size: 32px auto;
}

.local-nav li .local-nav-icon-icon2 {
    background-position: 0 -32px;
}
```
- 这里的小图片是精灵图。

### 5.5 背景线性渐变

语法1：
```css
background: linear-gradient(起始方向, 颜色1, 颜色2, ...);
background: -webkit-linear-gradient(left, red, blue);
background: -webkit-linear-gradient(top left, red, blue);
```
- 背景渐变必须添加浏览器私有前缀。
- 起始方向可以是：方位名词或者度数。如果省略，默认就是 top。
