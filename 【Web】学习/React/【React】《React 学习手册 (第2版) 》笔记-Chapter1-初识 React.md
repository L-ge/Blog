###### 一、初识 React

1. 在 package.json 文件中提供具体的版本信息，以便安装各个包的正确版本。

2. 官方文档：https://zh-hans.react.dev/

3. 建议安装 React 开发者工具，来辅助开发 React 项目。这些工具可通过扩展 Chrome 和 Firefox 中安装，可作为 Safari、IE 和 React Native 的独立应用程序使用。借助开发者工具可以审查 React 组件树、查看属性和状态详情，甚至还可以判断哪些线上网址使用的是 React。
- 若想安装，请访问这个 Github 仓库（https://github.com/facebook/react-devtools）。仓库中有 Chrome（https://chromewebstore.google.com/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi）和 Firefox 扩展（https://addons.mozilla.org/zh-CN/firefox/addon/react-devtools/）的链接。
- 安装之后，就可以看出哪些网站正在使用 React 了。只要浏览器工具栏中 React 的图标点亮了，就说明那个网站使用的是 React。

4. 使用 React 必须安装 Node。安装 Node.js 顺带就安装了 npm，即 Node 包管理器。

5. 你见到的多数 JavaScript 项目中除了一系列文件之外还有一个 package.json 文件。这个文件用于描述项目自身及其依赖。在含有 package.json 文件的文件夹中执行 npm install 命令将安装项目中列出的所有包。

6. 常用命令：
	```
	// 初始化项目，创建 package.json 文件
	npm install -y

	// 安装包
	npm install package-name

	// 删除某个包
	npm remove package-name
	```

7. npm 的一个替代品是 Yarn。使用 npm 全局安装 Yarn：npm install -g yarn。

8. 凡是含有 yarn.lock 文件的项目，使用的就是 Yarn。
