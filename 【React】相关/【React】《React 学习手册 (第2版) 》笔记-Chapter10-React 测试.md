###### 十、React 测试

1. 保证质量的一个重要工具是单元测试（unit testing）。单元测试可以确认应用的每一个功能或单元能按预期工作。

2. 函数式编程技术的一个优势是写出的代码易于测试。纯函数生而便于测试。不可变性易于测试。使用专注特定任务的小型函数构织应用，实现的功能或代码单元易于测试。

3. 有些工具可以帮助我们分析代码，确保代码遵守一定的格式化指导方针。

4. 分析 JavaScript 代码的过程称为 hinting 和 linting。最早用来分析 JavaScript 代码的工具是 JSHint 和 JSLint，二者可以给我们提供格式方面的反馈。ESLint（https://eslint.org）是一个新近代码 lint 工具，支持最新的 JavaScript 句法。而且，ESLint 支持插件。我们可以创建和分享自己的插件，为 ESLint 增加配置项目，扩展 ESLint 的功能。

5. Create React App 原生支持 ESLint，我们在控制台中已经见过它发出的 lint 提醒和错误。

6. 我们将使用一个名为 eslint-plugin-react（https://www.npmjs.com/package/eslint-plugin-react）的插件。这个插件除了分析 JavaScript 代码之外，还能分析 JSX 和 React 句法。

7. 下面安装 eslint，作为一个开发依赖。eslint 可使用 npm 安装：
    ```
    npm install eslint --save-dev
    或
    yarn add eslint --dev
    ```

8. 使用 ESLint 之前，我们要配置一些应遵守的规则。这些规则在项目根目录中的一个配置文件里定义。这个文件可以使用 JSON 格式，也可以使用 YAML 格式。YAML 格式是一种数据序列化格式，与 JSON 类似，不过句法更少，便于人类阅读。

9. ESLint 自带一个创建配置的工具。我们可以在一些公司创建的 ESLint 配置文件的基础上修改，也可以自己动手创建。

10. 运行 eslint --init 命令，回答一些有关代码风格的问题，创建一个 ESLint 配置文件。
    ```
    npx eslint --init
    ...
    ```

11. npx eslint --init 命令做了三件事：
    - 在项目的 ./node_modules 文件夹中安装 eslint-plugin-react。
    - 自动把相关依赖添加到 package.json 文件中。
    - 在项目的根目录中创建配置文件 .eslintrc.json。

12. ESLint 对代码做静态分析，根据配置报告问题。
    - ESLint 不会自动引入 Node.js 全局变量，所以使用 __filename 和 __dirname 这些属性会报告问题。

13. eslint . 命令对整个目录执行 lint 操作。此时，我们往往希望 ESLint 忽略一些 JavaScript 文件。请把想忽略的文件或目录添加到 .eslintignore 文件中。例如下面这个 .eslintignore 文件：
    ```
    dist/assets/
    sample.js
    ```
    - 让 ESLint 忽略前面创建的 sample.js 文件，以及 dist/assets 文件夹中的所有文件。
    - 如果不忽略 assets 文件夹，ESLint 将分析供客户端使用的 bundle.js 文件，有可能会发现大量问题。

14. 可以在 package.json 文件中添加一个脚本，用于运行 ESLint：
    ```
    {
        "scripts": {
            "lint": "eslint ."
        }
    }
    ```
    - 执行 npm run lint 命令即可运行 ESLint，分析项目中除了被忽略以外的所有文件。

15. ESLint 插件众多，我们可以把插件添加到 ESLint 配置中，辅助我们编写代码。对 React 项目来说，绝对应该安装 eslint-plugin-react-hooks 插件（https://reactjs.org/docs/hooks-rules.html），实施 React Hooks 相关的规则。这个包由 React 团队发布，目的是帮助我们修正钩子相关的 bug。
    ```
    // 1. 先来安装
    npm install eslint-plugin-react-hooks --save-dev
    或
    yarn add eslint-plugin-react-hooks --dev

    // 2. 然后，打开 .eslintrc.json 文件，添加下述内容：
    {
        "plugins": [
            //...
            "react-hooks"
        ],
        "rules": {
            "react-hooks/rules-of-hooks": "error",
            "react-hooks/exhaustive-deps": "warn"
        }
    }
    ```
    - 这个插件检查名称以“use”开头的函数（假定为钩子），确保遵守钩子相关的规则。

16. 建议在项目中使用的另一个 ESLint 插件是 eslint-pluginjsx-a11y。a11y 是一个带数字的缩略词，表示在 accessibility 这个单词中，字符“a”和“y”之间由11个字母。可访问性指在构建工具、网站或技术时要考虑残障人士能不能使用。
    - 这个插件分析代码，确保没有违背任何可访问性规则。我们每个人都应该关注可访问性，这个插件可以帮助我们提升 React 应用的可访问性。
    - 这个插件也可以使用 npm 或 yarn 安装：
        ```
        npm install eslint-pluginjsx-a11y
        或
        yarn add eslint-pluginjsx-a11y
        ```
    - 然后添加到 .eslintrc.json 配置文件中：
        ```
        {
            "extends": [
                // ...
                "plugins: jsx-a11y/recommended"
            ],
            "plugins": [
                //...
                "jsx-a11y"
            ],
        }
        ```
    - 比如，我们添加一个没有 alt 属性的 image 标签。而要想通过 lint 检查，图像必须有 alt 属性，如果图像不影响用户对内容的理解，alt 属性的值可以为空字符串。 

17. 如果你想进一步完善自己的 ESLint 配置，可以参考 Awesome ESLint 仓库（https://github.com/dustinspecker/awesome-eslint）中众多的优秀资源。

18. Prettier 是一个代码格式化工具，适用于各种不同的项目。
    - 另外，如果你使用 Prettier 格式化过 Markdown 表格，干净利落的格式化肯定让你惊叹不已。
    - 以前，很多项目使用 ESLint 格式化代码，不过现在职责划分得很清晰，ESLint 负责保证代码质量，而 Prettier 负责处理代码格式化。
    - 为了使 Prettier 能与 ESLint 协同工作，我们要稍微调整一下项目的配置。首先，全局安装 Prettier：
        ```
        sudo npm install -g prettier
        ```
        - 现在，不管项目保存在何处，都可以使用 Prettier。

19. 为了在项目中配置 Prettier，要创建一个 .prettierrc 文件。我们在这个文件中设置项目的默认配置：
    ```
    {
        "semi": true,
        "trailingComma": none,
        "singleQuote": false,
        "printWidth": 80
    }
    ```
    - 这是笔者选用的默认配置，当然，你要根据自己的需求选择合适的配置。Prettier 支持的格式化选项参见文档（https://prettier.io/docs/en/options.html）。

20. 下面我们把 sample.js 文件中的内容替换为需要格式化的代码：
	- 首先，我们要安装一个配置工具和一个插件，把 ESLint 和 Prettier 集成在一起：
		```
		npm install eslint-config-prettier eslint-plugin-prettier --save-dev
		```
		- 我们安装的配置工具（eslint-config-prettier）禁用与 Prettier 有冲突的 ESLint 规则，安装的插件（eslint-plugin-prettier）则把 Prettier 规则集成到 ESLint 规则中。如此一来，运行 lint 脚本时也将运行 Prettier。
	- 然后，在 .eslintrc.json 文件中设置这些工具：
		```
		{
			"extends": [
				// ...
				"plugin:prettier/recommended"
			],
			"plugins": [
				//,
				"prettier"
			],
			"rules": {
				// ...
				"prettier/prettier": "error"
			}
		}
		```
	- 现在，运行 Prettier 的写命令，扫除一个文件中的格式化问题：
		```
		prettier --write "sample.js"
		```
		- 或者，针对指定文件夹中的所有 JavaScript 文件：
			```
			prettier --write "src/*.js"
			```

21. 在 VSCode 中使用 Prettier。
	- 首先要安装 Prettier 的 VSCode 插件。
	- 安装之后，按 Ctrl+Shift+P 组合键即可手动格式化一个文件或选中的部分代码。更好的做法是在保存时自动格式化代码。为此，要添加一些 VSCode 配置。
	- 依次选择菜单“Code-Preferences-Settings”，然后点击右上角那个小纸片图标，以 JSON 格式打开 VSCode 设置。在设置中添加下述键值对：
		```
		{
			"editor.formatOnSave": true
		}
		```
		- 读者备注：VSCode 设置即 C:\Users\Administrator\AppData\Roaming\Code\User\settings.json
	- 现在，每次保存文件，Prettier 都会根据 .prettierrc 文件中的默认配置格式化代码。如果希望在项目中没有 .prettierrc 配置文件时也格式化代码，可以把 Prettier 的配置添加到编辑器的设置中。

22. 对 React 应用做类型检查主要有三种方案：prop-type 库、Flow 和 TypeScript。

23. 若想在应用中使用 PropTypes，要安装 prop-type 库：
	```
	npm install prop-type --save-dev
	```
	- 为属性提供错误类型的值导致的提醒只在开发模式下显示，在生产环境中不显示，也不会导致渲染中断。

24. PropTypes 支持的类型检查包括：
	- PropTypes.array
	- PropTypes.object
	- PropTypes.bool
	- PropTypes.func
	- PropTypes.number
	- PropTypes.string
	- PropTypes.symbol

25. 另外，如果想确定有没有提供值，可以在上述检查的后面串上 .isRequired。比如说，要求必须提供字符串，可以这样检查：
	```
	App.propTypes = {
		name: PropTypes.string.isRequired
	};
	```

26. 还有些情况，我们不关系提供的是什么值，只要提供一个值就可以。此时，使用 any。例如：
	```
	App.propTypes = {
		name: PropTypes.any.isRequired
	};
	````
	- 这意味着，我们可以提供布尔值、字符串、数字等类型的值，只要 name 不是 undefined，类型检查就能通过。

27. 假设我们要做字符串类型的枚举检查。枚举类型是一个受限的选项列表，相应的字段或属性只能使用其中的值。我们要像下面这样调整 propTypes 对象：
	```
	App.propTypes = {
		status: PropTypes.oneOf(["Open", "Closed"])
	};
	```

28. 在 React 应用中， PropTypes 可用的全部配置选项参见文档（https://legacy.reactjs.org/docs/typechecking-with-proptypes.html）

29. Flow 是一个类型检查库（https://flow.org/en/docs/getting-started/），由 Facebook Open Source 小组开发和维护。这个工具通过静态类型注解检查错误。也就是说，我们在创建变量时指定其为特定的类型，Flow 就按照这一点检查使用的值是不是正确类型。

30. Flow 的一大特色是可以采用渐进的方式逐步应用 Flow。为整个项目添加类型检查功能是一个大工程。而使用 Flow，无须一步到位。我们只需在想做类型检查的文件顶部加上 //@flow，这样 Flow 便会自动检查有这一行内容的文件。

31. 在 React 应用中做类型检查，TypeScript 也是一个使用广泛的工具。开源的 TypeScript 是 JavaScript 的超集，为 JavaScript 语言添加了一些额外的特性。TypeScript 由 Microsoft 开发，旨在帮助开发者找出大型应用的 bug，并快速迭代项目。

32. 如何在应用中使用 TypeScript：
	- 首先，使用 Create React App 生成一个应用。这一次使用不同的标志：
		```
		npx create-react-app my-type --template typescript
		```
	- 我们来看一下生成的项目骨架。注意 src 目录中的文件，现在的扩展名是 .ts 或 .tsx。此外，还有一个 .tsconfig.json 文件，存储所有 TypeScript 设置。
	- 另外，打开 package.json 文件，你会发现列出了一些与 TypeScript 有关的新依赖，包括 TypeScript 库本身和 Jest、React 和 ReactDOM 等的类型定义。以 @types/ 开头的依赖就是某个库的类型定义。有了这些依赖，库中的函数和方法就有了类型，我们不用再自己动手设定类型。
	

33. 如果你生成的项目不含 TypeScript 功能，可能是使用的 Create React App 版本较旧。解决方法是执行 npm uninstall -g create-react-app 命令。

34. TypeScript 的作用不止验证属性。我们可以使用 TypeScript 的类型推导（type inference）对钩子值做类型检查。
	- 关于 TypeScript 的更多信息，参阅官方文档（https://www.typescriptlang.org/zh/docs/）和 GitHub 中的 React+TypeScript Cheatsheets（https://github.com/typescript-cheatsheets/react）。

35. 测试驱动开发（Test-driven development，TDD）是一种实践方式，而不是一项工程技术。TDD 也不仅仅是为应用编写测试，而是由测试驱动开发工作的实践过程。践行 TDD 要遵守下述步骤。
	- 先编写测试：
		- 这是最重要的步骤。首先通过测试声明要构建的功能，指明功能应该怎样运行。测试的步骤是遇红、变绿和镀金。
	- 运行测试，看着测试失败（遇红）：
		- 在开始编写代码之前运行测试，看着测试失败。
	- 编写最少量的代码，让测试通过（变绿）：
		- 一次关注一个测试，让测试通过。不要添加超出测试覆盖范畴的功能。
	- 重构代码和测试（镀金）：
		- 测试通过后再重新审视代码和测试，尽量使用简洁优雅的方式编写代码。

36. TDD 特别适合用于开发 React 应用，尤其方便测试钩子。在真正动手编写代码之前想好钩子的作用往往能达到事半功倍的效果。践行 TDD 可以让我们脱离 UI 去思考如何构建和验证一项功能或整个应用的数据结构。　

37. 着手编写测试之前，我们要选择一个测试框架。为 React 编写测试可以使用任何 JavaScript 测试框架，不过 React 官方文档推荐使用 Jest。这是一个 JavaScript 测试运行程序，可以通过 JSDOM 访问 DOM。不要小看访问 DOM 这个功能，因为我们要检查 React 渲染的内容，确保应用运作正常。

38. 使用 Create React App 生成的项目已经安装了 jest 包。我们可以使用 Create React App 再创建一个项目：
	```
	npx create-react-app testing
	```

38. Create React App 和测试：下面通过一个简单的例子说明如何测试。
	- 在 src 文件夹中新建两个文件：function.js 和 function.test.js。注意，Create React App 已经安装和配置好了 Jest，我们什么都不用做，直接编写测试即可。
	- 下面在 function.test.js 文件中拟定测试，即表达出我们希望函数应该具有什么功能。
	- 我们希望这个函数接受一个值，返回该值的两倍。测试就要把这个要求表达出来。test 是 Jest 提供的函数，用于测试某个具体功能。
		```
		// function.test.js
		test("Multiplies by two", () => {
			expect();
		});
		```
		- 第一个参数（Multiplies by two）是测试的名称。第二个参数是一个函数，提供要测试的内容。第三个参数可选，指定超时时间。默认的超时时间为 5 秒。
	- 接下来，我们要初步定义这个函数，求一个数的两倍是多少。下文把这个函数称为被测系统（system under test, SUT）。在 function.js 文件中定义这个函数：
		```
		export default function timesTwo() {...}
		```
	- 为了在测试中使用这个被测系统，要把它导出。在测试文件中，我们要导入这个函数，然后使用 expect 编写一个断言（assertion）。在断言中，我们表达的意思是，如果把 4 传给 timesTwo 函数，预期返回 8：
		```
		import { timeTwo } from "./functions";
		
		test("Multiplies by two", () => {
			expect(timesTwo(4)).toBe(8);
		});
		```
	- 在 Jest 中，expect 函数返回一个“匹配器”（matcher），用于验证结果。测试这个函数，使用的是 .toBe 匹配器，确认得到的对象与发给 .toBe 的参数是一致的。
	- 现在，执行 npm test 或 npm run test 命令，运行测试，看着测试失败。Jest 为每个失败测试提供了详细信息，包括堆栈跟踪。
	- 花时间编写测试，然后运行测试，看着测试失败，这个过程表明测试真正起作用了。失败反馈就是我们的待办事项清单。我们要编写最少量的代码，让测试通过。
	- 现在，打开 function.js 文件，添加适当的功能，让测试通过：
		```
		export function timesTwo(a) {
			return a * 2;
		}
		```
	
39. .toBe 匹配器测试的是与单个值是否相等。如果想测试一个对象或一个数组，要使用 .toEqual。

40. 另外一个常用的 Jest 函数是 describe()。这个函数通常用于组织多个相关的测试。比如说，我们可以使用 describe 语句把几个针对类似函数的测试组织在一起：
	```
	describe("Math functions", () => {
		test("Multiplies by two", () => {
			expect(timesTwo(4)).toBe(8);
		});
		test("Adds two numbers", () => {
			expect(sum(4, 2)).toBe(6);
		});
		test("Subtracts two numbers", () => {
			expect(subtract(4, 2)).toBe(2);
		});
	});
	```
	- 使用 describe 语句把测试组织在一起之后，测试运行程序会把这些测试归在一起，终端输出的测试报告更有条理，更方便阅读。

41. React 组件是对 React 所下的指令，用于创建和更新 DOM。因此，对组件的测试首先要渲染组件，然后检查得到的 DOM。
	- 组件测试不在浏览器中运行，而是在终端里使用 Node.js 运行。Node.js 没有提供与各浏览器兼容的 DOM API，不过 Jest 集成的 jsdom 包可以在 Node.js 中模拟浏览器环境，这是测试 React 组件必不可少的。

42. 组件测试基本上都要把 React 组件树渲染成 DOM 元素。下面以 Star.js 文件中的 Star 组件为例说明整个流程。
	- Start 组件：
		```
		import { FaStar } from "react-icons/fa";

		export default function Star({ selected = false }) {
			return (
				<FaStar color={selected ? "red" : "grey"} id="star" />
			);
		}
		```
	- 然后，在 index.js 文件中导入并渲染星标：
		```
		import Star from "./Star";

		ReactDOM.render(
			<Star />,
			document.getElementById("root")
		);
		```
	- 接下来编写测试。星标的代码已经编写好了，因此这里采用的不是 TDD 技术。新建 Star.test.js 文件，导入 React、ReactDOM 和 Star 组件：
		```
		import React from "react";
		import ReactDOM from "react-dom";
		import Star from "./Star";

		test("renders a star", () => {
			const div = document.createElement("div");
			ReactDOM.render(<Star />, div);
			expect(div.querySelector("svg")).toBeTruthy();
		});
		```
		- 注意，提供给 test 函数的第一个参数是测试的名称。然后，我们做些准备工作，创建一个 div 元素，供 ReactDOM.render 渲染星标。创建好元素之后，编写断言。
		- 我们预计在这个 div 元素中可以找到一个 svg 元素。

43. 文档（https://github.com/testing-library/jest-dom#custom-matchers）对各个自定义匹配器做了详细说明，请仔细阅读，了解在测试中有哪些匹配器可用。

44. 生成 React 项目时你可能注意到了，除了基本的 React 和 ReactDOM 之外，还安装了一些 @testing-library 名下的包。React Testing Library 旨在帮助我们实施好的测试实践，增强 React 生态环境中有关测试工具的功能。Testing Library 是一个综合项目，不仅对 React，还为 Vue、Svelte、Reason、Angular 等库提供了测试增强包。
	- React Testing Library 吸引人的因素之一是测试失败消息更详尽。

45. 下面添加 React Testing Library，增强功能。这个库在使用 Create React App 生成项目时已经安装了。
	- 首先，从 @testing-library/jest-dom 中导入 toHaveAttribute 函数：
		```
		import { toHaveAttribute } from '@testing-library/jest-dom';
		```
	- 然后，增强 expect 的功能，包含这个函数：
		```
		expect.extend({ toHaveAttribute });
		```
	- 现在，我们不再使用错误消息含糊其词的 toBeTruthy，转而使用 toHaveAttribute：
		```
		test("renders a star", () => {
			const div = document.createElement("div");
			ReactDOM.render(<Star />, div);
			expect(
				div.querySelector("svg")
			).toHaveAttribute("id", "hotdog");
		});
		```
	- 然后运行测试，你会发现错误消息指出了问题的真正所在。 

46. 如果想使用其他自定义匹配器，过程是一样的，即导入、扩展和使用：
	```
	import { toHaveAttribute, toHaveClass } from '@testing-library/jest-dom';

	expect.extend({ toHaveAttribute, toHaveClass });

	expect(you).toHaveClass("evenALittle");
	```
	- 不过，还有个简便的方法。如果导入的匹配器太多，不想一一列出或记住，可以导入 extend-expect 库：
		```
		import '@testing-library/jest-dom/extend-expect';

		// 删除这一行 --> expect.extend({ toHaveAttribute, toHaveClass });
		```

47. React Testing Library 的另一个功能是查询，即根据条件匹配。
	- 下面编写一种常见的测试——根据文本匹配：
		```
		export default function Star({ selected = false }) {
			return (
				<>
					<h1>Great Star</h1>
					<FaStar 
					 	id="star"
						color={selected ? "red" : "grey"} 
					/>
				</>
			);
		}
		```
	- 我们要渲染组件，然后测试 h1 中是否包含正确的文本。React Testing Library 提供的 render 函数正好能为我们提供帮助。我们将使用 render 替代 ReactDOM.render()，因此测试有些许差别。
	- 先从 React Testing Library 中导入 react：
		```
		import { render } from '@testing-library/react';
		```
	- render 接受一个参数，即要渲染的组件或元素。该函数返回一个查询对象，我们就通过这个对象检查组件或元素中的值。我们要使用的查询是 getByText，通过它找到匹配查询的第一个节点；如果没有元素匹配，抛出一个错误。如果想返回匹配的所有结点，使用返回一个数组的 getAllBy。
		```
		test("renders an h1", () => {
			const { getByText } = render(<Star />);
			const h1 = getByText(/Great Star/);
			expect(h1).toHaveTextContent("Great Star");
		});
		```
	- getByText 通过传入的正则表达式查找 h1 元素。然后，使用 Jest 匹配器 toHaveTextContent 指明 h1 中应该有什么文本。

48. 测试往往还涉及组件中的事件。
	- 以第 7 章创建的 Checkbox 组件为例：
		```
		export function Checkbox() {
			const [checked, setChecked] = useReducer(
				checked => !checked,
				false
			);
			return (
				<>
					<label>
						{checked ? "checked" : "not checked"}
						<input
							type="checkbox"
							value={checked}
							onChange={setChecked}
						/>
					</label>
				</>
			);
		}
		```
		- 这个组件使用 useReducer 改变复选框的勾选状态。这里，我们想通过一个自动化测试点击该复选框，把 checked 的值由默认的 false 改为 true。编写检查这个复选框的测试也能触发 useReducer，从而测试该钩子。
	- 我们要做的第一件事是选择想触发事件的元素。也就是说，指明让自动化测试点击哪个元素。我们将使用 Testing Library 提供的一个查询查找元素。由于目标输入框有一个标注（label），因此我们可以使用 getByLabelText()。
		```
		import { render } from '@testing-library/react';
		import Checkbox from "./Checkbox";

		test("Selecting the checkbox should change the value of checked to true", () => {
			const { getByLabelText } = render(<Checkbox />);
			const checkbox = getByLabelText(/not checked/);
		});
		```
		- 这个组件首次渲染时，标注文本是 not checked，因此我们可以通过正则表达式查找匹配的字符串。
		- 这里使用的正则表达式区分大小写，如果不需要区分，可以在正则表达式末尾加上 i。
			```
			const checkbox = getByLabelText(/not checked/i);
			```
	- 现在，目标复选框找到了。接下来，我们只需触发事件，编写一个断言，确定点击该复选框后 checked 属性的值变成了 true。
		```
		import { render, fireEvent } from '@testing-library/react';
		import Checkbox from "./Checkbox";

		test("Selecting the checkbox should change the value of checked to true", () => {
			const { getByLabelText } = render(<Checkbox />);
			const checkbox = getByLabelText(/not checked/i);
			fireEvent.click(checkbox);
			expect(checkbox.checked).toEqual(true);
		});
		```
	- 我们还可以反过来测试，检查再次触发事件后 checked 属性的值有没有被设为 false。我们修改了测试的名称，更准确地表明测试的意图：
		```
		test("Selecting the checkbox should toggle its value", () => {
			const { getByLabelText } = render(<Checkbox />);
			const checkbox = getByLabelText(/not checked/i);
			fireEvent.click(checkbox);
			expect(checkbox.checked).toEqual(true);
			fireEvent.click(checkbox);
			expect(checkbox.checked).toEqual(false);
		});
		```

49. 在不太容易访问 DOM 元素的情况下，我们可以使用 Testing Library 提供的另一个函数搜索 DOM 元素。
	- 首先，为要选择的元素添加一个属性：
		```
		<input
			type="checkbox"
			value={checked}
			onChange={setChecked}
			date-testid="checkbox"	// 添加 date-testid= 属性
		/>
		```
	- 然后使用 getByTestId 查询：
		```
		test("Selecting the checkbox should change the value of checked to true", () => {
			const { getByTestId } = render(<Checkbox />);
			const checkbox = getByTestId("checkbox");
			fireEvent.click(checkbox);
			expect(checkbox.checked).toEqual(true);
		});
		```

50. 代码覆盖度（code coverage）指具体对多少行代码做了测试的统计，通过这个指标可以判断有没有编写足够的测试。
	- Jest 提供的 Istanbul 是一个 JavaScript 工具，用于审查测试，生成一份报告，指明覆盖了多少语句、分支、函数和代码行。
	- 若想让 Jest 检查代码覆盖度，在运行 test 命令时加上 coverage 标志即可：
		```
		npm test -- --coverage
		```
	- 生成的报告将指出在测试过程中各个文件（包括导入测试的所有文件）中的代码执行了多少。
	- Jest 还会生成一份可在浏览器中查看的报告，进一步说明测试覆盖了哪些代码。Jest 生成覆盖度报告后，根目录中会出现一个 coverage 文件夹。在浏览器中打开 /coverage/lcov-report/index.html 文件，这是一份交互式报告，具体说明代码覆盖度。
