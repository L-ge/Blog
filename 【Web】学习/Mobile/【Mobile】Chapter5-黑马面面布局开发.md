# 站在巨人的肩膀上

> [黑马程序员pink老师前端入门教程，零基础必看的h5(html5)+css3+移动端前端视频教程](https://www.bilibili.com/video/BV14J4114768/)

> [MurphyChen's Notes](https://github.com/Hacker-C/notes)


# （五）黑马面面布局开发

## 1. 目的

1. 了解移动端页面开发流程
2. 掌握移动端常见布局思路

### 1.1  技术方案

1. 弹性盒子 + rem + LESS 
2. 最小适配设备为 iphone5 320px。最大设配设备为 iphone8 plus(ipad 能正常查看内容即可)

### 1.2 代码规范

1. 类名语义化，尽量精短、明确，必须以字母开头命名，且全部字母为小写，单词之间统一使用下划线 “_” 连接。
2. 类名嵌套层次尽量不超过三层。
3. 尽量避免直接使用元素选择器。
4. 属性书写顺序：
    - 布局定位属性：display / position / float / clear / visibility / overflow
    - 尺寸属性：width / height / margin / padding / border / background
    - 文本属性：color / font / text-decoration / text-align / vertical-align
    - 其他属性（CSS3）：content / cursor / border-radius / box-shadow / text-shadow
5. 避免使用 id 选择器。
6. 避免使用通配符 * 和 !important。

### 1.3 目录规范

```
项目文件夹：heimamm
	样式文件夹：css
	业务类图片文件夹：images
	样式类图片文件夹：icons
	字体类文件夹：fonts
```

## 2. 流程开发

### 2.1 蓝湖/摹客协作平台

- UI设计师 psd 效果图完成后，会上传到蓝湖或摹客里面，同时会拉前端工程师进入开发。
- 大部分情况下，UI会把图片按照前端设计要求给切好。
- UI设计师上传到蓝湖或摹客（了解）。

具体步骤：
1. 摹客官网地址：https://www.mockplus.cn，注册一个账号
2. 下载 moke 的 ps 插件 
3. PS 安装摹客或蓝湖插件
4. 打开 PS 摹客或蓝湖插件
5. 上传（需要切图，需要先标注切图）
6. 查看项目
7. 邀请成员进入（分享按钮，链接地址）

前端设计师可以直接摹客或蓝湖测量取值。

### 2.2 适配方案

- flex 布局  
- 百分比布局
- rem 布局
- vw/vh布局
- 响应式布局
- 本次案例使用 flex + rem + flexible.js +  LESS   

### 2.3  初始化文件

- 引入 normalize.css
    ```html
    <head>
        ...
        <link rel="stylesheet" href="./css/normalize.css" />
        ...
    </head>
    ```
- less 中初始化body样式
    ```css
    body {
        min-width: 320px;
        max-width: 750px;
        margin: 0 auto;
    }
    ```
- 引入 flexible.js
    ```html
    <body>
        <script src="./js/flexible.js"></script>
        ...
    </body>
    ```
- 约束范围:
    ```less
    // 约束当屏幕大于 750px 的时候，html 字体大小就不变化了。
    @media screen and (min-width: 750px) {
        html {
            font-size: 37.5px !important;
        }
    }
    ```

### 2.4 swiper 插件使用

官网地址：https://www.swiper.com.cn

- 下载需要的css和js文件，然后在 html 页面中引入相关文件。
    - swiper.min.css
    - swiper.min.js
- 官网找到类似案例，然后右键查看网页源代码，然后复制html结构、css样式和js语法。
- 根据需求定制修改模块。

### 2.5 图标字体上传下载

上传步骤：
1. 让UI美工准备好图标字体（必须是svg格式）
2. 点上传按钮（保留颜色并提交）
3. 生成之后加入购物车即可
4. 点击下载，下载代码。

小技巧：如何批量下载全部字体图标呢？
```js
var span = document.querySelectorAll('.icon-cover');
for (var i = 0, len = span.length; i < len; i++) {
    console.log(span[i].querySelector('span').click());
}
```

### 2.6 上传码云并发布部署静态网站

准备工作：需要下载git软件、需要码云注册账号

git 可以把我们的本地网站提交上传到远程仓库（码云 gitee）里面，类似以前的 ftp。

码云，就是远程仓库，类似服务器。 

1. 码云创建新的仓库，heimamm。  
2. 利用 git 提交，把本地网站提交到码云新建的仓库里面。
    - 在网站根目录右键 Git Bash Here。
    - 如果是第一次利用 git 提交，请配置好全局选项
        ```
        git config --global user.name "用户名"
        git config --global user.email "你的邮箱地址"
        ```
   - 初始化仓库
        ```
        git init
        ```
   - 把本地文件放到暂存区
        ```
        git add .
        ```
   - 把本地文件放到本地仓库里面
        ```
        git commit -m '提交黑马面面网站'
        ```
   - 链接远程仓库
        ```
        git remote add origin 你新建的仓库地址
        ```
   - 把本地仓库的文件推送到远程仓库 push
        ```
        git push -u origin master
        ```
3. 码云部署发布静态网站

- 在码云当前仓库中，点击“服务”菜单 
- 选择 Gitee Pages
- 选择 “启动” 按钮
- 稍等之后，会拿到地址，就可以利用这个地址来预览网页了!
- 当然你也可以利用“草料二维码”把网址生成二维码。
    - https://cli.im
- 注意仓库必须要有 index.html 首页。 

最后，如果提交网站，你不愿意用 git 提交，可以直接找到仓库，里面有文件，选择上传本地文件即可。但是，1个小时内只能上传 20 个以内的文件。
