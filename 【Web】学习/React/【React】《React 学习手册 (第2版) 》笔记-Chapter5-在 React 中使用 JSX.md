###### 五、在 React 中使用 JSX

1. JSX 是 JavaScript 中的 JS 和 XML 中的 X 的综合体，是对 JavaScript 的扩展，使用一种基于标签的句法直接在 JavaScript 代码中定义 React 元素。JSX 也是一种创建 React 元素的方式。

2. 在 JSX 中，元素的类型通过标签指定，标签的属性表示元素的属性，子元素添加在起始和结束标签之间。
	- 子元素也可以是其他 JSX 元素。
	- JSX 还可以与组件一起使用，只需使用类名定义组件即可。
		```
		React.createElement(IngredientsList, {list: [...]}, null)		// IngredientsList 是自定义组件
		
		<IngredientsList list={[...]}>
		```
		- 把数组传给组件时，要把数组放在一对花括号内。这叫 JavaScript 表达式，通过属性把 JavaScript 值传给组件时，必须这么做。
		- 组件的属性有两种类型：字符串或 JavaScript 表达式。JavaScript 表达式可以是数组、对象，甚至是函数。传入的表达式必须放在一对花括号内。

3. JSX 看着眼熟，而且多数指令得到的句法都与 HTML 类似。然而，使用 JSX 时还是要注意一些事项。
	- JSX 支持把组件添加为组件的子元素。
	- 由于 class 是 JavaScript 的保留字，因此定义 class 属性时要使用 className。
	- JavaScript 表达式放在一对花括号内，表示要对变量求值，并返回得到的值。字符串以外的值都要通过 JavaScript 表达式传入。
	- 一对花括号之间的 JavaScript 代码会做求值。这意味着，拼接或相加等操作都会正确执行。这也意味着，JavaScript 表达式中的函数也会被调用。

4. JSX 是 JavaScript，可以直接在 JavaScript 函数中使用。例如，可以把数组映射为 JSX 元素：
	```
	<ul>
		{props.ingredients.map((ingredient, i) => (
			<li key="{i}">{ingredient}</li>
		))}
	</ul>
	```

5. JSX 简洁，可读性高，但是浏览器无法解释。所有 JSX 都要转换成 createElement 调用。幸好，有一个工具可以胜任这项工作：Babel。

6. JavaScript 是一门解释型语言，浏览器把代码解释为文本，无须编译。然而，不是所有浏览器都支持最新的 JavaScript 句法，而且没有浏览器支持 JSX 句法。 由于我们想在 JSX 中使用最新的 JavaScript 特性，那就需要找到一种方式把我们写出的源码转换成浏览器可以解释的版本。这个过程叫作编译，Babel（https://babeljs.io）就是做这项工作的。

7. 使用 Babel 的方式有很多种，最容易上手的是直接在 HTML 中通过 CDN 引入 Babel，编译类型为“text/babel”的脚本块中的代码。Babel 在客户端运行源码之前编译代码。这虽然不是在生产环境中使用 Babel 的最佳方式，却是开始着手使用 JSX 的好方法。
	```
	<!DOCTYPE html>
	<html>
	  <head>
		<meta charset="utf-8" />
		<title>React Examples</title>
	  </head>
	  <body>
		<div id="root"></div>
		<script src="https://unpkg.com/react@16/umd/react.development.js"></script>
		<script src="https://unpkg.com/react-dom@16/umd/react-dom.development.js"></script>
		<script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
		
		<script type="text/babel">
			// 在这里编写 JSX 代码，或者链接保存 JSX 的 JavaScript 文件
		</script>
	  </body>
	</html>
	```
	- 直接在浏览器内使用转换程序，你会看到一个警告，要求我们事先编译好脚本，供生产环境使用。对于小 demo 来说，不要在意这个警告。

8. JSX 的展开运算符用起来与对象的展开运算符一样。展开运算符将把 recipe 对象的每个字段作为属性传给 Recipe 组件。
	```
	<div className="recipes">
      {recipes.map((recipe, i) => (
        <Recipe key={i} {...recipe} />
      ))}
    </div>
	```
	- 注意，上述简短的句法即可以把所有属性提供给 Recipe 组件。这样做是不错，但是提供给组件的属性有时太多了。

9. 我们可以使用对象析构限定传给函数的变量。这样一来，我们便可以直接访问 title 和 recipes 变量，不用再加上 props 前缀。
	```
	const Menu = (props) => (
		...
	);
	与
	const Menu = ({ title, recipes }) => (
		...
	);
	```
	- 使用对象析构可以让组件把这些字段的名称限定在本地作用域中，以便直接访问，而不用再使用 props.title，props.recipes。

10. 下面这个表达式把菜谱的名称转换成小写字符串，再把空格全局替换成连字符。
	```
	<section id={name.toLowerCase().replace(/ /g, "-")}>
  	</section>
	```

11. 在浏览器的 React 开发者工具中，可以查看得到的虚拟 DOM。如果修改 recipes 数组的内容，然后重新渲染 Menu 组件，React 将尽可能高效地修改 DOM。

12. React 不在一个组件中渲染两个或以上毗邻元素或同辈元素，必须把这些元素放在一个标签内，例如一个 div 内。然而，这样会导致创建很多非必需的标签，产生很多没什么用的容器。使用 React 片段，我们可以模拟容器的行为，但不真正创建新标签。
	```
	function Cat({ name }) {
		return (
			<React.Fragment>
				<h1>xxx</h1>
				<p>yyy</p>
			</React.Fragment>
		);
	}
	```
	- 添加 `<React.Fragment>` 这个标签后，控制台警告就没有了。
	- 我们还可以使用片段的简写句法进一步精简代码：
		```
		function Cat({ name }) {
			return (
				<>
					<h1>xxx</h1>
					<p>yyy</p>
				</>
			);
		}
		```
	- 查看 DOM，你会发现 DOM 树中没有这个片段：
		```
		<div id="root">
			<h1>xxx</h1>
			<p>yyy</p>
		</div>
		```
	- 片段是 React 一个相对较新的特性，免除了可能扰乱 DOM 的额外的容器。

13. webpack 在宣传上称自己为模块打包工具，负责把各种文件（JavaScript、LESS、CSS、JSX、ESNext 等）汇集成一个文件。
	- 打包的好处主要体现在两方面：模块化和网络性能。
	- 模块化指把源码分解为不同的部分，即模块，这样更便于展开工作，尤其是在团队中。
	- 网络性能的提升体现在浏览器只加载一个依赖，即构建包。每个 script 发起一个 HTTP 请求，而每个 HTTP 请求都有延迟损耗。把所有依赖打包到一个文件中，所有依赖通过一个 HTTP 请求就能加载，规避了额外的延迟。

14. 除了编译代码之外，webpack 还可以处理以下事项。
	- 代码分拆：把代码分成一块块，在需要时加载。这个过程有时也叫作汇总（rollup）或者分层（layer），目的是按照不同页面或设备的需求打散代码。
	- 代码简化：去掉空白、换行符、较长的变量名称和不必要的代码，减少文件大小。
	- 特性标记：把代码发给一个或多个（但不是所有）环境，测试特性。
	- 模块热替换：监视源码的变化，立即更新有变化的模块。

15. 使用 webpack 这类工具静态构建客户端 JavaScript，方便团队协同开发大型 Web 应用。采用模块打包工具 webpack 还可以获得以下好处：
	- 模块化：使用模块模式导出的模块可在应用的其他部分导入或引入，源码浅显易懂。模块化方便开发团队协同工作，各自负责编写自己的文件，在上线之前再合并成一个静态文件。
	- 合成：借助模块，我们可以构建简单、可重用的小型 React 组件，再高效组合在一起，构成完整的应用。小型组件易于理解、测试和重用，需要增强应用的功能时也方便替换。
	- 速度：把应用中的所有模块和依赖打包成一个客户端构建包能减少应用的加载时间，因为每个 HTTP 请求都有延迟。把一切打包进一个文件，客户端就只需发起一个请求。简化构建包中的代码也能减少加载时间。
	- 一致性：webpack 可以把 JSX 编译成 JavaScript，现在就可以使用未来的 JavaScript 句法。Babel 支持大量 ESNext 句法，我们无须担心浏览器是否支持我们写出的代码。开发者可以始终使用最新的 JavaScript 句法。

16. 有个可以自动生成预先配置好一切的工具，名为 CreateReact App。不过，这个工具隐藏了具体的步骤，在使用它之前我们要了解一下背后的机制。

17. 如何从头开始设置一个 React 项目。
	1. 创建项目
		- 使用 npm 创建项目和 package.json 文件，传入 -y 标志，一切都采用默认值。然后安装 webpack、webpack-cli、react 和 react-dom。
			```
			npm init -y
			npm install react react-dom serve
			```
		- 如果使用的是 npm 5，安装各个包时无须传入 --save 标志。
		- React 项目中文件的组织方式没有统一标准。
	2. 按模块划分组件
		- 更符合函数式编程风格的做法是把 Recipe 组件拆解成多个功能更专一的小型组件，然后再组合在一起。
		- 我们需要使用 ReactDOM 渲染 Menu 组件。
			```
			import React from "react";
			import ReactDOM from "react-dom";
			import data from "../data/recipes.json";
			import Menu from "./components/Menu";

			ReactDOM.render(<Menu recipes={data} />, document.getElementById("root"));
			```
			- 前四个语句导入这个应用所需的模块。我们没有通过 script 标签加载 react 和 react-dom，而是导入它们，这样 webpack 便可以把它们添加到构建包中。
	3. 使用 webpack 构建
		- 为了使用 webpack 创建静态构建过程，我们要安装一些包。我们所需的一切都可以使用 npm 安装。
			```
			npm install --save-dev webpack webpack-cli
			```
		- 为了让这个模块化的菜谱应用能顺利运行，我们要告诉 webpack 如何把源码打包成一个文件。从 4.0.0 版开始，webpack 无须配置文件就可以打包项目。没有配置文件，webpack 就使用默认设置打包代码。不过，使用配置文件可以定制打包过程。而且，我们也可以窥见被 webpack 隐藏起来的部分神秘操作。webpack 默认使用的配置文件名为 webpack.config.js。
		- 只要 webpack 发现 import 语句，就会在文件系统中寻找对应的模块，把它放到构建包中。
		- webpack 将沿着导入树，把用到的所有模块都放入构建包中。遍历这些文件得到一个依赖图。依赖指应用需要用到的什么东西，可以是一个组件文件、一个库（例如 React）或一个图像。整个依赖图就是构建包。
		- webpack.config.js 文件其实也是一个模块，导出一个 JavaScript 字面量对象，描述 webpack 要执行的操作。这个配置文件应该保存在项目的根文件夹中，与 index.js 文件同级。
			```
			var path = require("path");

			module.exports = {
				entry: "./src/index.js",
				output: {
					path: path.join(__dirname, "dist", "assets"),
					filename: "bundle.js",
				},
				module: {
					rules: [
						{
							test: /\.js$/,
							exclude: /(node_modules)/,
							use: {
								loader: "babel-loader",
								options: {
									presets: ["@babel/preset-env", "@babel/preset-react"],
								},
							},
						},
						{
							test: /\.css$/i,
							use: ["style-loader", "css-loader"],
						},
					],
				},
			};
			```
			- 首先，告诉 webpack，客户端的入口文件为 ./src/index.js。webpack 根据这个文件中的 import 语句自动构建依赖图。
			- 然后，指明我们想把打包后的 JavaScript 输出到 ./dist/bundle.js 文件中。webpack 最终打包好的 JavaScript 将保存在这个文件中。
			- 接下来，我们要设置的 webpack 指令是一些 loader，在特定的模块上运行。这部分指令添加到配置文件中的 module 字段名下。
			- rules 字段的值是一个数组，因为 webpack 支持的 loader 有好多种。每个 loader 在一个 JavaScript 对象中指定。test 字段的值是一个正则表达式，匹配模块的文件路径，执行相关的操作。这里，我们在所有导入的 JavaScript 文件上运行 babel-loader，但把 node_modules 文件夹排除在外。
		- 下面我们要安装所需的 Babel 依赖：babel-loader 和 @babel/core。
			```
			npm install babel-loader @babel/core --save-dev
			```
		- 现在，我们要指定运行 Babel 的预设（preset）。Babel 通过预设设置执行什么转换操作。先来安装预设：
			```
			npm install @babel/preset-env @babel/preset-react --save-dev
			```
		- 然后在项目的根文件夹中新建一个文件，命名为 .babelrc。
			```
			{
				"presets": ["@babel/preset-env", "@babel/preset-react"]
			}
			```
	4. 下面来运行 webpack，确保一切正常。
		- webpack 是静态运行的。通常，构建包在把应用部署到服务器之前创建。webpack 可以在命令行中使用 npx 运行：
			```
			npx webpack --mode development
			```
		- webpack 在成功运行后构建一个构建包，倘若失败会报错，多数错误与导入目标失效有关。调试 webpack 错误时，请特别留意 import 语句中的文件名和路径。
	5. 你还可以在 package.json 文件中添加一个 npm 脚本，创建一个快捷命令：
		```
		"scripts": {
			"build": "webpack --mode production"
		},
		```
		- 这样就可以运行快捷命令生成构建包了：
			```
			npm run build
			```
	
18. 我们使用的 import 语句，目前多数浏览器和 Node.js 都不支持。但是，我们依然可以使用，因为在最终的代码中 Babel 将把 import 语句转换成 require('module/path');。CommonJS 模块通常使用 require 函数。 

19. 我们把构建包导出到 dist 文件夹中，这个文件夹中就是要在 Web 服务器中运行的文件。index.html 文件应该放在 dist 文件夹中。这个文件中应该有一个 div 元素，用于挂载 React Menu 组件。此外，还要有一个 script 标签，加载打包好的 JavaScript。
	```
	<!DOCTYPE html>
	<html>
		<head>
			<meta
			name="viewport"
			content="minimum-scale=1.0, width=device-width, maximum-scale=1.0, user-scalable=no"
			/>
			<meta charset="utf-8" />
			<title>React Recipes App</title>
		</head>
		<body>
			<div id="root"></div>
			<script src="assets/bundle.js"></script>
		</body>
	</html>
	```
	- 这是应用的首页，通过一个 HTTP 请求从一个文件中加载所有代码，即 bundle.js。
	- 你要把这些文件部署到 Web 服务器中，或者使用 Node.js 或 Ruby on Rails 构建一个 Web 服务器端应用，伺服这些文件。

20. 把代码打包进一个文件，在浏览器中调试应用可能有些不便。为了缓解这方面的问题，我们可以提供源码映射（source mapping）。源码映射把构建包映射到源文件上。使用 webpack，我们只需要在 webpack.config.js 文件中添加几行代码：
	```
	// 在 webpack.config.js 中设置源码映射
	module.exports = {
		...
		devtool: "#source-map"	// 添加这个选项，开启源码映射
	}
	```
	- 把 devtool 属性设为 '#source-map'，这样 webpack 便知道我们想使用源码映射。再次运行 webpack，你会看到 dist 文件夹中生成了两个文件：老面孔 bundle.js 和新出现的 bundle.js.map。
	
21. 源码映射的作用是方便我们使用源码文件调试。在浏览器开发者工具的“Sources”标签页中，你会看到一个名为 webpack:// 的文件夹。这个文件夹中是构建包里的所有源码文件。你可以使用浏览器中的单步调试器调试这些文件。任何点击一个行号即可添加一个断点。刷新浏览器，JavaScript 将在运行到源码文件中的断点处暂停。你可以在“Scope”面板中查看作用域中的变量，或者在“Watch”面板中添加要监视的变量。 

22. Create React App（https://github.com/facebook/create-react-app） 是一个命令行工具，能自动生成 React 项目。Create React App 目的是帮助开发者快速创建 React 项目，不用再自己动手配置 webpack、Babel、ESLint 等相关的工具。
	- 首先，全局安装 Create React App 包：
		```
		npm install -g create-react-app 
		```
	- 然后，执行下述命令，指定用于存放项目的文件夹的名称：
		```
		create-react-app my-project
		```
	- 如果不想全局安装，也可以使用 npx 运行 Create React App。只需执行 npx create-react-app my-project。
	- 上述命令在指定的目录中创建一个 React 项目。仅有三个依赖：React、ReactDOM 和 react-scripts。react-scripts 负责安装 Babel、ESLint、webpack 等，免去了我们自己动手配置的麻烦。在生成的项目文件夹中有个 src 文件夹，其中有个 APP.js 文件。我们可以编辑这个文件，修改根组件及导入其他组件文件。
	- 进入 my-project 文件夹，执行 npm start。如果愿意，也可以执行 yarn start。这两个命令在 3000 端口上启动应用。
	- 你还可以执行 npm test 或 yarm test 命令运行测试。这两个命令在非交互模式下运行项目中所有测试文件。
	- 另外，也可以执行 npm run build 命令。使用 yarn 的话，执行 yarn build 命令。这两个命令创建供生成环境使用的构建包，经过了转换和简化。

23. 学习 React 时，如果不想自己定制 webpack 构建过程，还可以使用 CodeSandbox。这是一个在线 IDE，地址为 https://codesandbox.io。
