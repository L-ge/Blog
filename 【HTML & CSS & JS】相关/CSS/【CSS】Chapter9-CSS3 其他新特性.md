# 站在巨人的肩膀上

> [黑马程序员pink老师前端入门教程，零基础必看的h5(html5)+css3+移动端前端视频教程](https://www.bilibili.com/video/BV14J4114768/)

> [MurphyChen's Notes](https://github.com/Hacker-C/notes)


# （九）CSS3 其他新特性

## 1. 2D 转换

转换（`transform`）是 CSS3 中具有颠覆性的特征之一，可以实现元素的位移、旋转、缩放等效果。

转换可以简单理解为变形。
- 移动：`translate`
- 旋转：`rotate`
- 缩放：`scale`

### 1.1 二维坐标系

2D 转换是改变标签在二维平面上的位置和形状的一种技术。

### 1.2 2D 转换之移动 translate

2D 移动是 2D 转换里面的一种功能，可以改变元素在页面中的位置，类似定位。

移动盒子的位置的3个方法：定位、盒子的外边距、2D 转换移动。

语法：
```css
transform: translate(x, y);     // 或者分开写     
transform: translateX(n);
transform: translateY(n);
```
- translate 里面的参数可以用 %。
- 如果里面的参数是 %，移动的距离是盒子自身的宽度或高度来对比的。

重点：
- 定义 2D 转换中的移动，沿着 X 和 Y 轴移动元素。
- translate 最大的优点：不会影响到其他元素的位置。
- translate 中的百分比单位是相对于自身元素的 `trainslate:(50%, 50%)`。
- 对行内标签没有效果。

### 1.3 让一个盒子水平垂直居中

```css
/*子绝父相*/
position: absolute;
top: 50%;
left: 50%;
transform: translate(-50%, -50%);   // 盒子往上走自己高度的一半、往左走自己宽度的一半。
```

### 1.4 2D 转换之旋转 rotate

2D 旋转指的是让元素在2维平面内顺时针旋转或者逆时针旋转。

语法：
```css
transform: rotate(度数);
```

重点：
- rotate 里面跟度数，单位是 deg，比如 rotate(45deg)。
- 角度为正时，顺时针；负时，为逆时针。
- 默认旋转的中心点是元素的中心点。

### 1.5 2D 转换中心点 transform-origin

我们可以设置元素转换的中心点。

语法：
```css
transform-origin: x y;
```

重点：
- 注意后面的参数 x 和 y 用空格隔开。
- x y 默认转换的中心点是元素的中心点 (50% 50%)。
- 还可以给 x y 设置像素或者方位名词（top bottom left right center）。
    ```css
    transform-origin: left bottom;      // 左下角
    transform-origin: center center;    // 中心点。等价于 50% 50%。
    ```

### 1.6 2D 转换之缩放 scale

缩放，顾名思义，可以放大和缩小。只要给元素添加上了这个属性就能控制它放大还是缩小。

```css
transform: scale(x, y);
```

注意：
- 注意其中的 x 和 y 用逗号分隔。
- transform: scale(1, 1); 宽和高都放大一倍，相对于没有放大。
- transform: scale(2, 2); 宽和高都放大2倍
- transform: scale(2); 只写一个参数，第二个参数则和第一个参数一样，相当于 scale(2, 2)。
- transform: scale(0.5, 0.5); 缩小。
- scale 缩放最大的优势：可以设置转换中心点缩放，默认以中心点缩放的，而且不影响其他盒子。

注意，scale 会让阴影变大，它放大的是元素。

### 1.7 2D 转换综合写法

注意：
1. 同时使用多个转换，其格式为：`transform: translate() rotate() scale() ...` 等。
2. 其顺序会影转换的效果。（先旋转会改变坐标轴方向）
3. 当我们同时有位移和其他属性的时候，记得要将位移放到最前。

### 1.8 2D 转换总结

- 转换 transform，我们简单理解就是变形，有 2D 和 3D 之分。
- 我们暂且学了三个，分别是位移、旋转和缩放。
- 2D 移动 translate(x,y) 最大的优势是不影响其他盒子，里面参数用%，是相对于自身宽度和高度来计算的。
- 可以分开写，比如 translateX(x) 和 translateY(y)。
- 2D 旋转 rotate(度数) 可以实现旋转元素，度数的单位是deg。
- 2D 缩放 scale(x,y) 里面参数是数字，不跟单位，可以是小数，最大的优势是不影响其他盒子。
- 设置转换中心点 transform-origin: x y; 参数可以百分比、像素或者是方位名词。
- 当我们进行综合写法，同时有位移和其他属性的时候，记得要将位移放到最前。

## 2. 动画

动画（animation）是 CSS3 中具有颠覆性的特征之一，可通过设置多个节点来精确控制一个或一组动画，常用来实现复杂的动画效果。

相比较过渡，动画可以实现更多变化，更多控制，连续自动播放等效果。

### 2.1 动画的基本使用

制作动画分为两步：
1. 先定义动画 (动画序列 `%α`)
2. 再使用（调用）动画

#### 2.1.1 用 keyframes 定义动画（类似定义类选择器）

```css
@keyframes 动画名称 {
    0% {
        width: 100px;
    }
    100% {
        width: 200px;
    }
}
```

动画序列：
- 0% 是动画的开始，100% 是动画的完成。这样的规则就是动画序列。
- 在 @keyframes 中规定某项 CSS 样式，就能创建由当前样式逐渐改为新样式的动画效果。
- 动画是使元素从一种样式逐渐变化为另一种样式的效果。您可以改变任意多的样式任意多的次数。
- 请用百分比来规定变化发生的时间，或用关键词“from”和“to”，等同于 0% 和 100%。

#### 2.1.2 元素使用动画

```css
div {
    width: 200px;
    height: 200px;
    background-color: aqua;
    margin: 100px auto;
    /* 调用动画 */
    animation-name: 动画名称;
    /* 持续时间 */
    animation-duration: 持续时间;
}
```

例子：
```css
 /* 我们想页面一打开，一个盒子就从左边走到右边 */
 /* 1. 定义动画 */
@keyframes move {
    /* 开始状态 */
    0% {
        transform: translateX(0px);
    }
    /* 结束状态 */
    100% {
        transform: translateX(1000px);
    }
}
div {
    width: 200px;
    height: 200px;
    background-color: pink;
    /* 2. 调用动画 */
    /* 动画名称 */
    animation-name: move;
    /* 持续时间 */
    animation-duration: 5s;
}
```

注意:
1. 可以做多个状态的变化。（`keyframes`：关键帧）
2. 里面的百分比必须是整数。
3. 里面的百分比是总的时间 `animation-duration` 的划分。

### 2.2 动画常用属性

| 属性                        | 描述                                                                |
| --------------------------- | ------------------------------------------------------------------- |
| `keyframes`                 | 规定动画。                                                          |
| `animation`                 | 所有动画属性的简写属性，除了 animation-play-state 属性。            |
| `animation-name`            | 规定 @keyframes 动画的名称。（必须的）                              |
| `animation-duration`        | 规定动画完成一个周期所花费的秒或毫秒，默认是0。（必须的）           |
| `animation-timing-function` | 规定动画的速度曲线，默认是“ease”。                                  |
| `animation-delay`           | 规定动画何时开始，默认是0。                                         |
| `animation-iteration-count` | 规定动画被播放的次数，默认是1，还有 infinite。                      |
| `animation-direction`       | 规定动画是否在下一周期逆向播放，默认是"normal"，alternate 逆播放。  |
| `animation-play-state`      | 规定动画是否正在运行或暂停。默认是"running"，还有"paused"。         |
| `animation-fill-mode`       | 规定动画结束后状态。保持 forwards；回到起始 backwards（默认值）。   |

### 2.3 动画简写属性

```css
animation: 动画名称 持续时间 运动曲线 何时开始 播放次数 是否反方向 动画起始或者结束的状态;
```

```css
animation: myfirst 5s linear 2s infinite alternate;
```

- 简写属性里面不包含 `animation-play-state`
- 暂停动画: `animation-play-state: puased;` 经常和鼠标经过等其他配合使用。
- 想要动画走回来，而不是直接跳回来: `animation-direction: alternate`。
- 盒子动画结束后，停在结束位置: ` animation-fill-mode: forwards`。

### 2.4 速度曲线细节

animation-timing-function：规定动画的速度曲线，默认是“ease”。

| 值          | 描述                                            |
| ----------- | ----------------------------------------------- |
| linear      | 动画从头到尾的速度是相同的。匀速。              |
| ease        | 默认。动画以低速开始，然后加快，在结束前变慢。  |
| ease-in     | 动画以低速开始。                                |
| ease-out    | 动画以低速结束。                                |
| ease-in-out | 动画以低速开始和结束。                          |
| steps()     | 指定了时间函数中的间隔数量（步长）。            |

元素可以添加多个动画，用逗号分隔。

## 3. 3D 转换

我们生活的环境是 3D 的，照片就是 3D 物体在 2D 平面呈现的例子。

有什么特点？
- 近大远小。
- 物体后面遮挡不可见。

当我们在网页上构建 3D 效果的时候，参考这些特点就能产生 3D 效果。

## 3.1 三维坐标系

三维坐标系其实就是指立体空间，立体空间是由 3 个轴共同组成的。

- x轴：水平向右。注意：x 右边是正值，左边是负值。
- y轴：垂直向下。注意，y 下面是正值，上面是负值。
- z轴：垂直屏幕。注意：往外面是正值，往里面是负值。

3D 转换，我们主要学习工作中最常用的 3D 位移和 3D 旋转。

主要知识点：
- 3D 位移：translate3d(x,y,z)
- 3D 旋转：rotate(x,y,z)
- 透视：perspective
- 3D 呈现：transform-style

## 3.2 3D 移动 translate3d

3D 移动在 2D 移动的基础上多加了一个可以移动的方向，就是 z 轴方向。

- transform:translateX(100px)：仅仅是在 x 轴上移动。
- transform:translateY(100px)：仅仅是在 y 轴上移动。
- transform:translateZ(100px)：仅仅是在 z 轴上移动（注意：translateZ 一般用 px 单位）。
- transform:translate3d(x,y,z)：其中 x、y、z 分别指要移动的轴的方向的距离。

## 3.3 透视 perspective

在 2D 平面产生近大远小视觉立体，但是只是效果二维的。

- 如果想要在网页产生 3D 效果需要透视（理解成 3D 物体投影在 2D 平面内）。
- 模拟人类的视觉位置，可认为安排一只眼睛去看。
- 透视我们也称为视距，视距就是人的眼睛到屏幕的距离。
- 距离视觉点越近的在电脑平面成像越大，越远成像越小。
- 透视的单位是像素。

透视写在被观察元素的父盒子上面的。
d：就是视距，视距就是一个距离人的眼睛到屏幕的距离。
z：就是 z 轴，物体距离屏幕的距离，z 轴越大（正值）我们看到的物体就越大。

```
眼睛 <------> 物体 <---z---> 屏幕
眼睛 <----------d----------> 屏幕
```

## 3.4 translateZ

transform:translateZ(100px)：仅仅是在 z 轴上移动。

有了透视，就能看到 translateZ 引起的变化了。

## 3.5 3D 旋转 rotate3d

3D 旋转指可以让元素在三维平面内沿着 x 轴、y 轴、z 轴或者自定义轴进行旋转。

- transform:rotateX(45deg)：沿着 x 轴正方向旋转 45 度。
- transform:rotateY(45deg)：沿着 y 轴正方向旋转 45 度。
- transform:rotateZ(45deg)：沿着 z 轴正方向旋转 45 度。
- transform:rotate3d(x,y,z,deg)：沿着自定义轴旋转，deg 为角度（了解即可）。
    - xyz是表示旋转轴的矢量，是标示你是否希望沿着该轴旋转，最后一个标示旋转的角度。
    - transform:rotate3d(1,0,0,45deg) 就是沿着 x 轴旋转 45deg。
    - transform:rotate3d(1,1,0,45deg) 就是沿着对角线旋转 45deg。

对于元素旋转的方向的判断，我们需要先学习一个左手准则。

左手准则（x 轴）：
- 首先，眼睛看着左手的手掌背，拇指伸直指向右，其他手指弯曲。
- 左手的手拇指指向 x 轴的正方向。
- 其余手指的弯曲方向就是该元素沿着 x 轴旋转的方向。

左手准则（y 轴）：
- 首先，眼睛看着左手的手掌背，拇指伸直指向下，其他手指弯曲。
- 左手的手拇指指向 y 轴的正方向。
- 其余手指的弯曲方向就是该元素沿着 y 轴旋转的方向（正值）。

z 轴旋转是顺时针旋转（正值）。

## 3.6 3D 呈现 transform-style

- 控制子元素是否开启三维立体环境。
- transform-style: flat; 子元素不开启 3d 立体空间，默认的。
- transform-style: preserve-3d; 子元素开启立体空间。
- 代码写给父级，但是影响的是子盒子。
- **这个属性很重要，后面必用。**

## 4. 浏览器私有前缀

浏览器私有前缀是为了兼容老版本的写法，比较新版本的浏览器无须添加。

1. 私有前缀
    - `-moz-`：代表 firefox 浏览器私有属性。
    - `-ms-`：代表 ie 浏览器私有属性。
    - `-webkit-`：代表 safari、chrome 私有属性。
    - `-o-`：代表 Opera 私有属性。

2. 提倡的写法
    ```css
    -moz-border-radius: 10px;
    -webkit-border-radius: 10px;
    -o-border-radius: 10px;
    border-radius: 10px;
    ```
