###### 四、React 运行机制

1. 使用 React 构建应用几乎离不开 JSX。这是一种基于标签的 JavaScript 句法，看起来很像 HTML。

2. 为了在浏览器中使用 React，我们要引入两个库：React 和 ReactDOM。前者用于创建视图，后者则具体负责在浏览器中渲染 UI。这两个库都可以通过 unpkg CDN 引用（链接见下述代码片段）。
	```
	<!DOCTYPE html>
	<html>
	  <head>
		<meta charset="utf-8" />
		<title>React Samples</title>
	  </head>
	  <body>
		<!-- 目标容器 -->
		<div id="root"></div>

		<!-- React 和 ReactDOM 库（开发版本）-->
		<script src="https://unpkg.com/react@16/umd/react.development.js"></script>
		<script src="https://unpkg.com/react-dom@16/umd/react-dom.development.js"></script>
		<script>
		  // 纯 React 和 JavaScript 代码
		</script>
	  </body>
	</html>
	```
	- 这是在浏览器中使用 React 需要做的最少工作。
	- 你可以把 JavaScript 代码放在单独的文件中，不过在页面中加载时必须放到 React 后面。
	- 为了在浏览器控制台中看到所有错误消息和警告，我们使用的是 React 的开发版本。你也可以使用 react.production.min.js 和 react-dom.production.min.js，换成供生产环境使用的简化版本，只是看不到警告消息了。

3. 简单来说，HTML 是一系列指令，让浏览器构建 DOM。浏览器加载 HTML 并渲染用户页面时，构成 HTML 文档的元素变成 DOM 元素。
	- 在 HTML 中，元素之间呈现一种层次结构。
	- 过去，网站由相互独立的 HTML 界面组成。用户访问这些页面时，浏览器请求并加载各 HTML 文档。AJAX 的发明引入了单页应用的概念。由于浏览器可以通过 AJAX 请求并加载少量数据之后，整个 Web 应该便可以精简成一个页面，依靠 JavaScript 更新用户界面。
	- 在单页应用中，浏览器只在开始时加载一个 HTML 文档。在用户浏览网站的过程中，始终待在同一个页面。用户不断与应用产生交互，JavaScript 则不断销毁和创建新的用户页面。这给人的感觉像是从一个页面跳转到另一个页面，但其实你仍然在相同的 HTML 页面上，背后繁重的工作都交给 JavaScript 了。

4. DOM API 是一系列对象，供 JavaScript 与浏览器交互，修改 DOM。如果你曾用过 document.createElement 或 document.appendChild，那你就用过 DOM API。
	- React 是代我们更新浏览器 DOM 的一个库。

5. 浏览器 DOM 由 DOM 元素组成，类似地，React DOM 由 React 元素组成。DOM 元素和 React 元素看着类似，其实区别很大。React 元素是对真正的 DOM 元素的描述。换句话说，React 元素是如何创建浏览器 DOM 的命令。

6. 我们可以使用 React.createElement 创建一个表示 h1 的 React 元素：
	```
	<!DOCTYPE html>
	<html>
	<head>
		<meta charset="utf-8">
		<title>React Elements</title>
	</head>
	<body>

	<!-- Target Container -->
	<div id="react-container"></div>

	<!-- React Library & React DOM-->
	<script src="https://unpkg.com/react@16/umd/react.development.js"></script>
	<script src="https://unpkg.com/react-dom@16/umd/react-dom.development.js"></script>

	<script>

		const dish = React.createElement(
		  "h1",
		  {id: "recipe-0", 'data-type': "title"},
		  "Baked Salmon"
		)

		ReactDOM.render(
		  dish,
		  document.getElementById('react-container')
		)

		console.log('dish', dish)

	</script>

	</body>
	</html>
	```
	- 第一个参数指定想创建的元素的类型。
	- 第二个参数设定元素的属性。
	- 第三个参数指定元素的子节点，即插入起始和结束标签之间的节点（这里只插入了一些文本）。
	- 渲染时，React 将把这个元素转换成真正的 DOM 元素。
	- React 组件其实就是 JavaScript 字面量，告知 React 如何构建 DOM 元素。
	- 在控制台中输出这个元素，你会看到如下内容：
		```
		{ 
			$$typeof: Symbol(React.element), 
			type: "h1", 
			key: null, 
			ref: null, 
			props: {id: 'recipe-0', data-type: 'title', children: 'Baked Salmon'}, 
			_owner: null, 
			_store: {}, 
		}
		```
		- 这是一个 React 元素的结构，其中一些字段是供 React 使用的，包括 _owner、_store 和 $$typeof。key 和 ref 字段对 React 元素很重要。
		- React 元素的 type 属性告诉 React 要创建的是什么类型的 HTML 或 SVG 元素。props 属性表示构建一个 DOM 元素所需的数据和子元素。children 属性则表示嵌套显示在元素内的文本。
		- 这里给出的是 React.createElement 返回的对象。我们其实不会自己动手创建 React 元素，而是使用 React.createElement 函数。
	- ReactDOM 提供了在浏览器中渲染 React 元素所需的工具。渲染所需的 reader 方法在 ReactDOM 中。
	- React 元素，包括它的子元素，使用 ReactDOM.reader 在 DOM 中渲染。我们想渲染的元素通过第一个参数传入，第二参数是目标节点，即指明在哪里渲染元素。
		
7. 在 DOM 中与渲染元素有关的一切功能都在 ReactDOM 包中。

8. 在 React 16 之前的版本中，只能在 DOM 中渲染一个元素。而现在，还可以渲染数组。
	```
	const dish = React.createElement("h1", null, "Baked Salmon"};
	const dessert = React.createElement("h2", null, "Coconut Cream Pie"};
	ReactDOM.render([dish, dessert], document.getElementById('root'));
	```
	- 上述代码在 root 容器中把两个元素渲染为同辈元素。

9. React 使用 props.children 渲染子元素。可以把其他 React 元素渲染为子元素，创建一个元素树。
	```
	const dish = React.createElement("section", {id: "baked-salmon"},
        React.createElement("h1", null, "Baked Salmon"),
        React.createElement("ul", {"className": "ingredients"},
            React.createElement("li", null, "1 lb Salmon"),
            React.createElement("li", null, "1 cup Pine Nuts"),
            React.createElement("li", null, "2 cups Butter Lettuce"),
            React.createElement("li", null, "1 Yellow Squash"),
            React.createElement("li", null, "1/2 cup Olive Oil"),
            React.createElement("li", null, "3 cloves of Garlic")
        ),
        React.createElement("section", {"className": "instructions"},
            React.createElement("h2", null, "Cooking Instructions"),
            React.createElement("p", null, "Preheat the oven to 350 degrees."),
            React.createElement("p", null, "Spread the olive oil around a glass baking dish."),
            React.createElement("p", null, "Add the salmon, Garlic, and pine..."),
            React.createElement("p", null, "Bake for 15 minutes."),
            React.createElement("p", null, "Add the Butternut Squash and put..."),
            React.createElement("p", null, "Remove from oven and let cool for 15 ....")
        )
    )

    ReactDOM.render(dish, document.getElementById('react-container'))

    console.log('dish element', dish)
	```
	- 传给 createElement 函数的各个额外参数就是一个子元素。React 将创建一个由子元素构成的数组，把 props.children 的值设为这个数组。
	- 审查得到的 React 元素，你会看到每个列表项目 li 都是一个 React 元素，出现在 props.children 名下的数组里。
	- 有 class 属性的 HTML 元素，在 React 中使用的是 className，而不是 class。这是因为 class 是 JavaScript 的保留字，所以不得不使用 class Name 定义 HTML 元素的 class 属性。

10. React 应用是由一系列 React 元素构成的树状结构，一切都发端于一个根元素。

11. React 的主要优势是能把数据与 UI 元素分开。由于 React 本身是 JavaScript，我们可以借助 JavaScript 逻辑构建 React 组件树。
	```
	const items = [
        "1 lb Salmon",
        "1 cup Pine Nuts",
        "2 cups Butter Lettuce",
        "1 Yellow Squash",
        "1/2 cup Olive Oil",
        "3 cloves of Garlic"
    ]

    const ingredients = React.createElement(
        "ul",
        { className: "ingredients" },
		// items.map((ingredient, i) => React.createElement("li", null, ingredient))
        items.map((ingredient, i) => React.createElement("li", { key: i }, ingredient))
    )

    ReactDOM.render(
        ingredients,
        document.getElementById('react-container')
    )

    console.log('ingredients', ingredients)
	```
	- "React.createElement("li", null, ingredient))"时，控制台会发出一个警告。通过迭代数组构建一组子元素时，React 预期每个元素都有 key 属性。由了 key 属性，React 才能高效率地更新 DOM。为每个列表项目元素添加一个独一无二的 key 属性后，这个警告便会消失。

12. 在 React 中，组件（component）方便重用相同的结构，只需在结构中填充不同的数据即可。
	-  使用 React 构建用户界面时，应该尽量考虑把元素分解成可重用的片段。
	- 例子：
		```
		function IngredientsList({ items }) {
			return React.createElement("ul", {className: "ingredients"},
				items.map((ingredient, i) =>
					React.createElement("li", { key: i }, ingredient)
				)
			);
		}
		```
		
13. 在出现函数组件之前，创建组件有几种其他方式。
	- createClass：2013年，React 刚开源时，创建组件使用 createClass。
		```
		const IngredientsList = React.createClass({
			displayName: "IngredientsList",
			render() {
				return React.createElement("ul", {className: "ingredients"},
				  this.props.items.map((ingredient, i) =>
					  React.createElement("li", { key: i }, ingredient)
				  )
				);
			}
		});
		```
		- 使用 createClass 创建的组件需要一个 render() 方法，指明返回及渲染什么 React 元素。组件背后的思想还是一样的，即描述一些可重用的 UI 供渲染。
		- 在 React 15.5（2017年4月）中，React 开始在使用 createClass 的地方发出警告。在 React 16（2017年9月）中，React.createClass 正式被弃用，移到了单独的包 create-react-class 中。 
	- 类组件：ES2015 为 JavaScript 添加类句法后，React 引入了一种创建 React 组件的新方法。React.Component API 支持使用类句法创建组件实列：
		```
		class IngredientsList extends React.Component {
			render() {
			  return React.createElement("ul", {className: "ingredients"},
				  this.props.items.map((ingredient, i) =>
					  React.createElement("li", { key: i }, ingredient)
				  )
			  )
			}
		}
		
		const items = [
		  "1 lb Salmon",
		  "1 cup Pine Nuts",
		  "2 cups Butter Lettuce",
		  "1 Yellow Squash",
		  "1/2 cup Olive Oil",
		  "3 cloves of Garlic"
		]

		ReactDOM.render(
			React.createElement(IngredientsList, {items}, null),
			document.getElementById('react-container')
		)
		```
		- 现在依然可以使用类句法创建 React 组件，但是笔者要提前告诉你，React.Component 即将被弃用。
