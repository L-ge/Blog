###### 九、Suspense

1. 设计 Suspense 的目的是解决 Factbook 在弹性伸缩中遇到的特定问题。
	- 事实上，Suspense 相关的概念大都涉及使用钩子。
	- Suspense 涉及的多数机制都可以通过钩子抽象出来。

2. 错误边界组件可以防止整个应用崩溃。另外，借此还可以在生产环境中渲染合理的错误消息。由于错误集中在一个组件处理，我们可以跟踪应用中的错误，把错误发给问题管理系统。

3. 当下，创建错误边界组件的唯一方法是使用类组件。未来，错误边界组件或许能够通过钩子或者其他不用类的方案创建。
	```
	import React, { Component } from "react";

	export default class ErrorBoundary extends Component {
		state = { error: null };

		static getDerivedStateFromError(error) {
			return { error };
		}

		render() {
			const { error } = this.state;
			const { children, fallback } = this.props;

			if (error) return <fallback error={error} />;
			return children;
		}
	}
	```
	- 这是一个类组件，以不同的方式存储状态，而且没有使用钩子。
	- 在类组件中，我们要定义一些特定的方法，在组件生命周期的不同时刻调用。getDerivedStateFromError 就是其中一个方法。这个方法在渲染过程中子组件发生错误时调用。一旦发生错误，将为 state.error 设值。只要有错误，就通过属性把错误传给 fallback 组件，渲染该组件。
	- 我们可以使用这个组件捕获组件树中的错误，倘若发生错误，渲染 fallback 组件。例如，我们可以把整个应用放在一个错误边界组件中；也可以把 App 中的各个组件分别放到 ErrorBoundary 组件中。
	- 如果把两个同辈组件分别放到 ErrorBoundary 组件中，ErrorBoundary 组件会在恰当的位置渲染，两个错误发生在各自的区域中。错误边界组件就像一道墙一样，能防止错误侵袭应用中的其他区域。尽管我们故意抛出了两个错误，区域中的内容依然能正常渲染。

4. 代码拆分技术可以把代码基分解为便于管理的小块，然后按需加载各小块。
	```
	import React, { useState } from "react";
	import Agreement from "./Agreement";
	import Main from "./Main";
	import "./SiteLayout.css";

	export default function App() {
		const [agree, setAgree] = useState(false);

		if (!agree) 
			return <Agreement onAgree={() => setAgree(true)} />;

		return <Main />;
	}
	```
	- 首先渲染 Agreement，等待用户同意。用户同意后，卸载 Agreement 组件，再渲染 Main 组件。
	- 一开始，唯一渲染的组件是 Agreement。用户同意后，agree 的值变成 true，此时渲染 Main 组件。现在问题是，Main 组件及其所有子组件都打包在一个 JavaScript 文件中。这意味着，用户要等待代码基下载完毕后才能开始渲染 Agreement 组件。（读者笔记，这里没有笔误，就是渲染 Agreement 组件）

5. 为了在渲染时才加载 Main 组件，我们可以使用 React.lazy 声明，而不是直接导入：
	```
	const Main = lazy(() => import("./Main"));
	```
	- 这行代码让 React 在首次渲染 Main 组件时再加载对应的代码基，即在渲染时再使用 import 函数导入 Main 组件。
	- 在运行时导入代码与从互联网中加载内容是一样的。首先，对 JavaScript 代码的请求处于等待状态，真正加载时要么成功，返回一个 JavaScript 文件，要么失败，抛出错误。加载数据时我们要让用户知道数据正在加载中，加载代码也一样，要让用户知道代码正在加载中。

6. 我们又一次进入管理异步请求的情况之中。这一次，我们有 Suspense 组件的协助。Suspense 组件的用法和 ErrorBoundary 组件类似，即把组件树中特定的的组件嵌套其中。只不过，ErrorBoundary 组件在出现错误时回落到一个错误消息，而 Suspense 组件在惰性加载组件时渲染一个消息，提示正在加载中。
	```
	import React, { useState, Suspense, lazy } from "react";
	import Agreement from "./Agreement";
	import ClimbingBoxLoader from "react-spinners/ClimbingBoxLoader";

	const Main = lazy(() => import("./Main"));

	export default function App() {
		const [agree, setAgree] = useState(false);

		if (!agree) return <Agreement onAgree={() => setAgree(true)} />;

		return (
			<Suspense fallback={<ClimbingBoxLoader />}>
				<Main />
			</Suspense>
		);
	}
	```
	- 现在，应用最初只加载 React、Agreement 组件和 ClimbingBoxLoader，等用户同意协议后再加载 Main 组件。
	- Main 组件嵌套在 Suspense 组件中。一旦用户同意协议，立即开始加载 Main 组件的代码基。由于对这部分代码基的请求目前处于等待状态，因此 Suspense 组件将在 Main 组件的位置渲染 ClimbingBoxLoader，直到成功加载 Main 组件。加载成功后，Suspense 组件将卸载 ClimbingBoxLoader，转而渲染 Main 组件。
	- React Spinners 是一个动画加载框库，指明正在加载什么东西或应用正在处理中。使用之前请安装：
		```
		npm i react-spinners
		```
	- 如果在加载 Main 组件之前网络连接中断了，这是一个错误，要做处理。我们可以把 Suspense 组件嵌套在 ErrorBoundary 组件中。
		```
		<ErrorBoundary fallback={ErrorScreen}>
			<Suspense fallback={<ClimbingBoxLoader />}>
				<Main />
			</Suspense>
		</ErrorBoundary>
		```
		- 这三个组件组合在一起可以处理多数异步请求。我们可以处理等待状态：Suspense 组件在请求源码的过程中渲染一个加载框。我们可以处理失败状态：在加载 Main 组件的过程中如果出错了，由 ErrorBoundary 组件捕获并处理。我们还可以处理成功状态：请求成功后渲染 Main 组件。

7. Suspense 模式也可用于加载数据。只要在 React 应用中抛出一个 promise，就需要一个 Suspense 组件负责渲染回落组件。
	```
	const loadStatus = () => {
		throw new Promise(resolves => null);
	};

	function Status() {
		const status = loadStatus();
		return <h1>status: {status}</h1>;
	}

	export default function App() {
		return (
			<Suspense fallback={<GridLoader />}>
				<ErrorBoundary>
					<Status />
				</ErrorBoundary>
			</Suspense>
		);
	}
	```
	- 如果没有用 Suspense 组件，如果在 loadStatus 函数中抛出一个 promise，在浏览器中将看到一个特殊的错误。错误消息指出我们触发了待定状态，但是组件树中更高层级上没有配置 Suspense 组件。
	- 抛出 promise，React 就知道现在处于待定状态，从而渲染回落的 GridLoader 组件。
	- 如果 loadStatus 成功返回一个结果，正常渲染 Status 组件。倘若出错了（比如 loadStatus 抛出一个错误），ErrorBoundary 便发挥作用。假使 loadStatus 抛出一个 promise，触发待定状态，则由 Suspense 组件处理。

8. 严格来说，JavaScript 中的 throw 关键字用于抛出错误。
	```
	throw new Error("inspecting errors");
	```
	- 这行代码触发一个错误。倘若不对错误做任何处理，将导致整个应用崩溃。
	- 在开发模式中，未处理的错误被捕获后都直接显示在屏幕上。
	- 未处理的错误在控制台中始终可见。在控制台中看到的所有红色文本都是有关错误的信息。
	- 在 JavaScript 中可以抛出任何类型：
		```
		throw "inspecting errors";	
		```
		- 这里抛出一个字符串。浏览器会提醒我们有什么东西未捕获，但不是错误。
		- 这一次，我们抛出的是一个字符串，浏览器没有渲染 Create React App 的错误界面。React 知道错误和字符串是有区别的。
	- 也可以抛出一个 promise：
		```
		throw new Promise(resolves => null);
		```
		- 现在，浏览器会告诉我们有什么东西未捕获，但不是错误，而是一个 promise。

9. 通常，无穷循环并不是我们想要的效果，在 React 中也是如此。我们要知道的是，抛出的 promise 会被 Suspense 组件捕获，随之进入待定状态，直到 promise 得到解决。

10. 兼容 Suspense 的数据源要提供一个函数处理加载数据相关的全部状态，即待定、成功和出错。loadStatus 函数现在一次只能返回或抛出一个类型。我们需要在 loadStatus 函数在加载数据时抛出一个 promise，成功加载数据后返回一个响应，出错时抛出一个错误。
	```
	// 例子1：
	function loadStatus() {
		if (error) throw error;
		if (response) return response;
		throw promise;
	}

	// 例子2：
	const loadStatus = (function() {
		let error, response;
		const promise = new Promise(resolves => 
			setTimeout(resolves, 3000)
		)
			.then(() => (response = "success"))
			.catch(e => (error = e));
		return function() {
			if (error) throw error;
			if (response) return response;
			throw promise;
		};
	})();
	```
	- 我们要在某个地方声明 error、response 和 promise 变量，并确保这些变量位于恰当的作用域内，以免与其他请求冲突。为此，我们要使用闭包（closure）定义 loadStatus 函数。
	- 例子2中，这是一个闭包。error、promise 和 response 的作用域在 loadStatus 函数的定义体外部将关闭。我们声明的 loadStatus 是一个匿名函数，而且立即调用：fn() 的作用与 (fn)() 相同。loadStatus 函数的返回值是内部返回的那个函数。loadStatus 函数可以访问 error、promise 和 response 变量，但是其余的 JavaScript 代码不可以。
	- 例子2中的 loadStatus 函数就是兼容 Suspense 的数据源，能与 Suspense 架构互通状态。

11. 创建兼容 Suspense 的数据源只需要一个 promise，因此我们可以定义一个函数，接受一个 promise 为参数，返回一个兼容 Suspense 的数据源。在下面的示例中，我们把这个函数命名为 createResource：
	```
	const resource = createResource(promise);
	const result = resource.read();
	```
	- 这段代码假定 createResource(promise) 能成功创建一个 resource 对象。该对象有个 read 函数，可以调用任意多次。promise 得到解决后，read 返回最终的数据。在 promise 处于待定状态时，read 抛出 promise。如果有什么地方出错了，read 将抛出错误。这个数据源兼容 Suspense。

12. createResource 函数与前面定义的那个匿名函数很想：
	```
	function createResource(pending) {
		let error, response;
		pending.then(r => (response = r)).catch(e => (error = e));
		return {
			read() {
				if (error) throw error;
				if (response) return response;
				throw pending;
			}
		};
	}
	```
	- createResource 函数返回一个资源对象，该对象有个名为 read 的函数。

13. 以上是目前为止 Suspense 的运作方式，任何类型的异步资源都可以像前文所述那样使用 Suspense 组件处理。本章的主旨是让你知道，React 一直在努力提升 React 的运行速度。这个特性背后涉及很多 React 自身的运行机制，尤其是调和算法 Fiber。 

14. 发生变化时，React 以 JavaScript 对象的形式创建一份组件树副本，找出组件树中需要改动的部分，只改动那些部分。修改完毕后，使用副本（称为 work-in-progress tree）替换现有组件树。强调一点，React 将继续使用组件树中现存的部分。比如说我们要把列表中的 red 项改为 green：
	```
	<ul>
		<li>blue</li>
		<li>green</li>
		<li>red</li>
	</ul>
	```
	- React 不会删掉第三个li，而是把子节点（red文本）替换为green文本。这样更新效率高，自问世以来 React 就是这样更新 DOM 的。然而，这里隐藏一个问题。更新 DOM 是一项耗时的任务，因为这个操作是同步的。我们要等到所有更新都到位并渲染出来之后，在主线程中才能执行其他任务。也就是说，我们要等到所有更新都结束，这回影响用户体验，好似没有反应。为了解决这个问题，React 开发团队彻底重写了 React 调和算法，名为 Fiber。

15. Fiber 在 16.0 版中发布，重写了 DOM 的更新机制，采用一种异步方式。16.0 版引入的第一处改动是分离渲染器（renderer）和调和器（reconciler）。渲染器是 React 库中处理渲染的部分，调和器是 React 库中管理更新的部分。

16. 调和算法保留在 React Core 中（要使用 React 就安装这个包），而各个渲染目标则负责具体渲染。也就是说，ReactDOM、React Native、React 360 等负责处理渲染逻辑，可以接入 React 的核心调和算法。

17. React Fiber 的另一项重要改动是调整了调和算法。前面说到，耗时的 DOM 更新操作会阻塞主线程。长时间阻塞的更新操作叫作“work”，React Fiber 把 work 分解为较小的单元，叫作“fiber”。一个 fiber 是一个 JavaScript 对象，负责记录正在调和什么，及其在更新循环中的什么位置。

18. 当一个 fiber（work 单元）结束后，React 检查主线程中有没有什么重要的事情要做。如果有重要的工作，React 将交出主线程的控制权。当重要的工作做完之后，React 再继续更新。如果主线程接下来没有重要的任务，React 转到下一个 work 单元，然后在 DOM 中渲染改动。

19. Fiber 把一项工作分拆成多个小块，这样优先级高的任务就可以插队，让主线程先行处理。这样实现的用户体验给人的感觉更流畅。
