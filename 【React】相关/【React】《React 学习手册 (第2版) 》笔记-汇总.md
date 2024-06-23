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


---


###### 二、JavaScript 新特性

1. 可以将浏览器尚不支持的新句法转换成可以正确识别的旧句法。

2. kangax 编辑的兼容表（http://kangax.github.io/compat-table/esnext）是一个很好的参考资源，可以查明最新的 JavaScript 特性在各浏览器中的支持程度。

3. 使用 let 关键字，可以把变量的作用域限定在任何代码块中。

4. 模板字符串为字符串拼接提供了一种新的方式，还可以在字符串中插入变量。
	```
	console.log(lastName + ", " + firstName + " " + middleName);
	console.log(`${lastName}, ${firstName} ${middleName}`);
	```

5. 任何返回一个值的 JavaScript 代码都可以放入 ${ }，添加到模板字符串中。

6. 现在，一个字符串可以横跨多行。

7. 函数在声明之后便可以调用、执行。

8. 函数声明的作用域会被提升，而函数表达式不会。换句话说，在编写函数声明之前可以调用函数，而在创建函数表达式之前不能调用函数，否则会报错。
	```
	// 在声明函数之前调用函数
	hey();
	
	// 函数声明
	function hey() {
		alert("hey!");
	}
	```
	- 这样是可以的。因为函数的作用域被提升了，移到了文件作用域的顶部。
	
	```
	// 在声明函数之前调用函数
	hey();
	
	// 函数表达式
	const hey = function() {
		alert("hey!");
	}
	TypeError: hey is not a function
	```
	- 错误。在项目中导入文件和函数时偶尔也会遇到这种 TypeError。遇到这种错误，重构一下，改成函数声明即可。

9. 默认参数是 ES6 规范引入的特性，如果没有为参数提供值，将使用默认值。

10. 箭头函数是 ES6 新增的一个特性。有了箭头函数，创建函数不再需要使用 function 关键字了。另外，也经常不使用 return 关键字。
	```
	const lordify = firstName => `${firstName} of Canterbury`;
	
	const lordify = (firstName, land) => `${firstName} of ${land}`;
	```
	- 箭头指明的就是要返回的值。
	- 这种写法的另一个好处是，如果函数只接受一个参数，可以省略参数两侧的圆括号。不过，参数超过一个时要放入圆括号中。
	- 上述示例函数可以写在一行内，毕竟只需要返回一个语句。如果有多行，要使用花括号。

11. 注意，返回对象时，把要返回的对象放在圆括号中。
	```
	const person = (firstName, lastName) => ({
		first: firstName,
		last: lastName
	});
	```

12. 常规的函数不限定 this 的作用域。
	```
	// 例子1：
	const tahoe = {
		mountains: ["Freel", "Rose", "Tallac", "Rubicon", "Silver"],
		print: function(delay = 1000) {
			setTimeout(function() {
				console.log(this.mountains.join(", "));
			}, delay);
		}
	};
	
	tahoe.print();	// Uncaught TypeError: Cannot read property 'join' of undefined
	```
	- 出现这个错误的原因在于试图在 this 上调用 .join 方法。在控制台中输出 this，可以看到它引用的是 Window 对象。
	- 为了解决这个问题，我们可以使用箭头函数句法保全 this 的作用域。
		```
		// 例子2：
		const tahoe = {
			mountains: ["Freel", "Rose", "Tallac", "Rubicon", "Silver"],
			print: function(delay = 1000) {
				setTimeout(() => {
					console.log(this.mountains.join(", "));
				}, delay);
			}
		};
		```
	
13. 务必时刻考虑作用域。箭头函数不限定 this 的作用域。
	```
	// 例子3：
	const tahoe = {
		mountains: ["Freel", "Rose", "Tallac", "Rubicon", "Silver"],
		print: (delay = 1000) => {
			setTimeout(() => {
				console.log(this.mountains.join(", "));
			}, delay);
		}
	};
	
	tahoe.print();	// Uncaught TypeError: Cannot read property 'join' of undefined
	```
	- 把 print 函数改成箭头函数后，this 引用的就是窗口了。
	
14. 读者笔记，来源于 ChatGpt 的知识点：
	- 箭头函数（Arrow Function）在 JavaScript 中确实不绑定自己的 this，它们会捕获自己创建时的上下文的 this 值，作为自己的 this 值。这是箭头函数的一个重要特性，也是它们与普通函数的一个主要区别。
	- 在普通函数中，this 的值取决于函数如何被调用。例如，如果函数作为对象的方法被调用，this 将指向该对象。如果函数只是被简单地调用（不是作为对象的方法），this 通常指向全局对象（在浏览器中是 window），除非在严格模式下，this 会是 undefined。
	- 然而，在箭头函数中，this 的值在函数被创建时就已经确定了，并且不能通过函数的调用方式来改变。箭头函数会捕获它被创建时的上下文的 this 值，这通常是在包含箭头函数的外部函数中的 this。
	- 箭头函数不限定 this 的作用域，而是继承父级作用域的 this 值。
	- 因此：
		- 例子1由于 setTimeout 里面是回调，回调函数只是被简单地调用，因此里面 this 的作用域是 Window 对象。
		- 例子2由于 setTimeout 里面的回调改成了箭头函数，箭头函数绑定了它被创建时的上下文的 this 值，也即 print 的 function 的 this，而在普通函数中，this 的值取决于函数如何被调用，因此 print 的 function 的 this 就是 tahoe。
		- 例子3由于 print 也成了箭头函数，箭头函数绑定了它被创建时的上下文的 this 值，也即 tahoe 所在的 this，即 window 对象。
		
15. 为了确保你写出的代码能正常在浏览器中运行，要把带啊吗转换成兼容性最好的版本。这个过程叫作编译。最常用于编译 JavaScript 代码的工具之一是 Babel（http://www.babeljs.io）
- 借助 Babel 可以立即就使用最新的 JavaScript 特性，而不用等浏览器支持。
- 这与常规的编译不一样，代码不被编译成二进制文件，而是转换成更多浏览器可以解析的句法。
- 编译 JavaScript 代码的过程通常由 webpack 或 Parcel 等构建工具自动操作。

16. 在函数内部，可以通过 arguments 数组获取函数参数。 

17. 下面这段代码从对象中取出 bread 和 meat，创建两个局部变量。另外，由于这两个析构的变量是使用 let 声明的，因此 bread 和 meat 的变化对原 sandwich 对象没有影响。
	```
	const sandwich =  {
      bread: "dutch crunch",
      meat: "tuna",
      cheese: "swiss",
      toppings: ["lettuce", "tomato", "mustard"]
    };

    let {bread, meat} = sandwich;

	bread = "garlic";
    meat = "turkey";
	
    console.log(bread)	// garlic
	console.log(meat)	// turkey
    
    console.log(sandwich.bread, sandwich.meat)	// dutch crunch tuna
	```

18. 可以析构传给函数的参数。也可以使用冒号和嵌套的花括号进行析构。
	```
	const lordify = ({ firstname }) => {
      console.log(`${firstname} of Canterbury`);
	};
	
	const lordify1 = ({ spouse: { firstname } }) => {
      console.log(`${firstname} of Canterbury`);
	};

    const regularPerson = {
      firstname: "Bill",
      lastname: "Wilson",
	  spouse: {
		firstName: "Phil",
		lastname: "Wilson",
	  }
    };

    lordify(regularPerson);	// Bill of Canterbury
	```

19. 值可以从数组中析构出来。还可以使用逗号跳过不需要的值，这叫列表匹配。
	```
	const [firstResort] = ["Kirkwood", "Squaw", "Alpine"]
    console.log(firstResort);	// Kirkwood
	
	var [,,thirdResort] = ["Kirkwood", "Squaw", "Alpine"]
    console.log(thirdResort);	// Alpine
	```

20. 对象字面量增强与析构相反，它指把对象重新组合成一体。还可以使用对象字面量增强或重组创建对象方法。
	```
	const name = "Tallac";
    const elevation = 9738;
    const funHike = { name, elevation };
    console.log(funHike);	// {name: 'Tallac', elevation: 9738}
	
	
	const name = "Tallac";
    const elevation = 9738;
    const print = function() {
      console.log(`Mt. ${this.name} is ${this.elevation} feet tall`);
    }
    const funHike = { name, elevation, print };
    funHike.print();	// Mt. Tallac is 9738 feet tall
	```
	- 注意，我们使用 this 访问对象的键。

21. 定于对象的方法时，无须使用 function 关键字。
	```
	// 旧句法
    var skier = {
      name: name,
      sound: sound,
      powderYell: function() {
		var yell = this.sound.toUpperCase();
		console.log(`${yell} ${yell} ${yell}!!!`);
	  },
      speed: function(mph) {
        this.speed = mph;
        console.log('speed:', mph);
      }
    };
	
	// 新句法
    const skier = {
      name,
      sound,
      powderYell() {
        let yell = this.sound.toUpperCase();
        console.log(`${yell} ${yell} ${yell}!!!`);
      },
      speed(mph) {
        this.speed = mph;
        console.log('speed:', mph);
      }
    };
	```
	- 使用对象字面量增强可以把全局变量放到对象中，还可以省略 function 关键字。

22. 可以使用展开运算符合并数组的内容。
	```
	const peaks = ["Tallac", "Ralston", "Rose"];
    const canyons = ["Ward", "Blackwood"];
    const tahoe = [...peaks, ...canyons];
    console.log(tahoe.join(', '));	// Tallac, Ralston, Rose, Ward, Blackwood
	```

23. reverse 函数会改变原数组，而在有展开运算符的情况下，我们无须改变原数组，而是创建一个副本，再做反转。
	```
	// 获取数组中的最后一个元素
	const peaks = ["Tallac", "Ralston", "Rose"];
    const [last] = [...peaks].reverse();
    console.log(last);	// Rose
    console.log(peaks.join(', '));	// Tallac, Ralston, Rose
	```

24. 展开运算符也可用于获取数组中剩余的元素。
	```
	const lakes = ["Donner", "Marlette", "Fallen Leaf", "Cascade"];
    const [first, ...rest] = lakes;
    console.log(rest.join(", "));	// Marlette, Fallen Leaf, Cascade
	```

25. 还可以把使用三个点号句法把函数的参数收集到一个数组中。在函数中，这叫剩余参数。
	```
	function directions(...args) {
      var [start, ...remaining] = args;
      var [finish, ...stops] = remaining.reverse();

      console.log(`drive through ${args.length} towns`);		// drive through 5 towns
      console.log(`start in ${start}`);							// start in Truckee
      console.log(`the destination is ${finish}`);				// the destination is Tahoma
      console.log(`stopping ${stops.length} times in between`);	// stopping 3 times in between
    }

    directions("Truckee", "Tahoe City", "Sunnyside", "Homewood", "Tahoma");
	```

26. 展开运算符还可用于处理对象。
	```
	const morning = {
      breakfast: "oatmeal",
      lunch: "peanut butter and jelly"
    };

    const dinner = "mac and cheese";

    const backpackingMeals = {
      ...morning,
      dinner
    };

    console.log(backpackingMeals);	// {breakfast: 'oatmeal', lunch: 'peanut butter and jelly', dinner: 'mac and cheese'}
	```

27. promise 是一个对象，表示异步操作的状态是挂起、已完成或是失败。
	- 挂起的 promise 表示获取数据之前的状态。我们要串接一个名为 .then() 的函数。这个函数接受一个回调函数，在前一步操作执行成功后运行。
	- then 方法在 promise 得到成功处理后调用回调函数。不管回调函数返回什么，都将作为下一个 then 函数的参数。因此，我们可以串接多个 then 函数处理成功的 promise。
	- 例子：
		```
		fetch("https://api.randomuser.me/?nat=US&results=1")
        .then(res => res.json())
		.then(json => json.results)
        .then(console.log)
        .catch(console.error);
		```
		
28. 处理 promise 的另一种常用方式是创建异步函数。此时，我们不等待 promise 成功处理后返回结果再调用一系列 then 函数了，而是让异步函数暂停执行函数中的代码，直到 promise 处理成功。

29. 下面例子中，getFakePerson 函数是使用 async 关键字声明的。这样定义的才是异步函数，才会在成功处理 promise 之前暂停执行后续代码。promise 调用前面要加上 await 关键字，让异步函数等待 promise 得到成功处理。这段代码完成的任务与上面使用 then 函数的那段代码完全一样。
	```
	const getFakePerson = async () => {
		try {
			let res = await fetch("https://api.randomuser.me/?nat=US&results=1");
			let { results } = res.json();
			console.log(results);
		} catch (error) {
			console.error(error);
		}
	};
	getFakePerson();
	```
	- 使用 async 和 await 时，要把 promise 调用放在 try...catch 块中，处理 promise 失败时可能出现的错误。
	
30. 下面 getFakeMembers 函数返回一个新的 promise。这个 promise 向 API 发起请求，如果成功，加载数据；如果不成功，返回错误。
	```
	const getFakeMembers = count => new Promise((resolves, rejects) => {
      const api = `https://api.randomuser.me/?nat=US&results=${count}`;
      const request = new XMLHttpRequest();
      request.open('GET', api);
      request.onload = () =>
           (request.status === 200)
		   ? resolves(JSON.parse(request.response).results)
		   : rejects(Error(request.statusText));
      request.onerror = (err) => rejects(err);
      request.send();
    });
	
	getFakeMembers(5)
	  .then(members => console.log(members))
	  .catch(error => console.error(`getFakeMembers failed: ${error.message}`));
	```
	
31. JavaScript 使用原型继承实现一种类似面向对象的结构。ES2015 引入了类声明句法，可现实是，JavaScript 的运作方式并没有变。函数还是对象，继承仍通过原型处理。类就像是在糟糕的原型句法上套了一层句法糖。
	```
	// ES2015 之前
	function Vacation(destination, length) {
      this.destination = destination;
      this.length = length;
    }

    Vacation.prototype.print = function() {
      console.log(this.destination + " will take " + this.length + " days");
    }

    const maui = new Vacation("Maui", 7);
    maui.print();
	
	// ES2015
	class Vacation {

      constructor(destination, length) {
        this.destination = destination;
        this.length = length;
      }

      print() {
        console.log(`${this.destination} will take ${this.length} days.`);
      }
    }

    const trip = new Vacation("Santiago, Chile", 9);
    trip.print();
	```
	
32. JavaScript 模块（module）是一组可重用的代码，方便插入其他 JavaScript 文件而不产生变量冲突。JavaScript 模块保存在单独的文件中，一个文件一个模块。创建和导出模块有两种选择：从一个模块中导出多个 JavaScript 对象，或者从一个模块中导出一个 JavaScript 对象。

33. export 可以导出供其他模块使用的任何 JavaScript 类型。

34. 模块也可以只导出一个主变量。此时，使用 export default。例如，export default new MyClass("xx", 123);
	- 如果只想导出一个类型，就可以把 export 换成 export default。
	- 读者笔记，来源于 ChatGpt 的知识点：一个模块中只能有一个 export default。
	- export 和 export default 都可用于导出任何 JavaScript 类型：原始对象、对象、数组和函数。

35. 在其他文件中使用一个模块，通过 import 语句导入。导出多个对象的模块可以充分利用对象析构。使用 export default 的模块导入为一个变量。
	```
	import { print, log } from "./text-helpers";
	import freel as f from "./mt-freel";
	import * as fns from "./text-helpers";
	```
	- 导入的模块变量在当前模块中可以放在其他变量名下。
	- 也可以使用 \* 把一切都导入为一个变量。
	
36. import 和 export 句法还没有得到所有浏览器及 Node 的完全支持。然而，与其他新兴的 JavaScript 句法一样，Babel 支持。这意味着，你可以在自己的源码中使用这些语句，Babel 知道在何处寻找你要使用的模块，把找到的模块添加到编译得到的 JavaScript 代码中。

37. CommonJS 是所有 Node 版本都支持的模块模式。Babel 和 webpack 也支持这种模块。在 CommonJS 中，JavaScript 对象使用 module.export 导出。
	```
	const print(message) => log(message, new Date())
	const log(message, timestamp) => console.log(`${timestamp.toString()}: ${message}`);
	
	module.export = {print, log}
	```

38. CommonJS 不支持 import 语句，模块使用 require 函数导入。
	```
	const { log, print } = require("./txt-helpers");
	```

39. 如果要了解最新的兼容信息，请查看 ESNext 兼容表。


---


###### 三、JavaScript 函数式编程

1. 函数可以使用 var、let 或 const 关键字声明，就像声明字符串、数字等变量一样。
	```
	var log = function(message) {
		console.log(message);
	}
	
	const log = message => {
		console.log(message);
	};
	```
	
2. 由于函数是变量，那就可以把函数添加到对象中。
	```
	const obj = {
		message: "xx",
		log(message) {
			console.log(message);
		}
	};
	obj.log(obj.message);
	```

3. 还可以把函数添加到 JavaScript 数组中。
	```
	const message = [
		"xx",
		message => console.log(message),
		"yy",
		message => console.log(message)
	];
	message[1](message[0]);
	message[3](message[2]);
	```

4. 函数可以像常规变量一样作为参数发给其他函数。
	```
	const insideFn = logger => {
        logger("They can be sent to other functions as arguments");
	}

    insideFn(message => console.log(message));
	```

5. 函数也可以像变量一样由其他函数返回。
	```
	const createScream = function(logger) {
        return function(message) {
			logger(message.toUpperCase() + "!!!");
        };
    };

    const scream = createScream(message => console.log(message));

    scream("functions can be returned from other functions");
    scream("createScream returns a function");
    scream("scream invokes that returned function");
	```
	- 高阶函数 createScream 还可以用箭头句法表示：
		```
		const createScream = logger => message => {
			logger(message.toUpperCase() + "!!!");
		};

		const scream = createScream(message =>
			console.log(message)
		);

		scream("ES6 can createScream with less");
		```

6. 函数式编程是一个更大编程范式的一部分，即声明式编程。声明式编程是一种编程风格，采用这种风格开发的应用有个显著特点：重点描述该做什么，而不管怎么做。

7. React 是声明式的。

8. 函数式编程的一些核心概念：不可变性、纯函数、数据转换、高阶函数和递归。

9. 在采用函数式风格的程序中，数据是不可变的，永不更改。我们不直接更改原始数据结构，而是创建数据结构的副本，所有操作都使用副本。

10. 在 JavaScript 中，函数的参数是对真正数据的引用。
	```
	// 例子1：
	let color_lawn = {
		title: "lawn",
        color: "#00FF00",
        rating: 0
    };

    function rateColor(color, rating) {
        color.rating = rating;
        return color;
    }

    console.log(rateColor(color_lawn, 5).rating);	// 5
    console.log(color_lawn.rating);					// 5
	
	// 例子2：
	let color_lawn = {
        title: "lawn",
        color: "#00FF00",
        rating: 0
    };

    const rateColor = function(color, rating) {
        return Object.assign({}, color, { rating: rating });
    };

    console.log(rateColor(color_lawn, 5).rating);	// 5
    console.log(color_lawn.rating);					// 0
	
	// 例子3：
	const rateColor = (color, rating) => ({
        ...color,
        rating
    });
	```
	- 例子2中，我们使用了 Object.assign，它创建了一个空对象，复制颜色对象，然后覆盖副本的评分。
	- 例子3中，使用箭头函数句法和对象展开运算符编写 rateColor 函数。使用展开运算符把颜色对象复制到一个新对象中，然后覆盖评分。
	- 注意例子3中，返回的对象在一对圆括号中。使用箭头函数时一定要这样做，因为箭头不能直接指向一个对象的花括号。

11. Array.push 不是一个不可变函数。如需不更改原数组，必须使用 Array.concat。Array.concat 的作用是拼接数组。
	```
	// 例子1：
	let list = [
      { title: "Rad Red" },
      { title: "Lawn" },
      { title: "Party Pink" }
    ];

    var addColor = function(title, colors) {
      colors.push({ title: title });
      return colors;
    };

    console.log(addColor("Glam Green", list).length);	// 4
    console.log(list.length);	// 4
	
	// 例子2：
	let list = [
      { title: "Rad Red" },
      { title: "Lawn" },
      { title: "Party Pink" }
    ];

    const addColor = (title, array) => array.concat({ title });

    console.log(addColor("Glam Green", list).length);	// 4
    console.log(list.length);	// 3
	```
	- 例子2中，concat 方法是接受一个由新颜色的标题构成的对象，把它添加到原数组的副本中。

12. 与复制对象一样，还可以使用展开运算符拼接数组。
	```
	let list = [
      { title: "Rad Red" },
      { title: "Lawn" },
      { title: "Party Pink" }
    ];

    const addColor = (title, list) => [
      ...list,
      { title }
    ];

    console.log(addColor("Glam Green", list).length);	// 4
    console.log(list.length);	// 3
	```
	- 把原列表复制到一个新数组中，然后把一个包含颜色标题的对象添加到副本中。这就体现了不可变性。
	
13. 纯函数指基于参数做计算，并返回一个值的函数。纯函数至少接受一个参数，而且始终返回一个值或另一个函数。这种函数没有副作用、不设置全局变量，也不更改应用的状态。纯函数把参数视为不可变数据。
	```
	var frederick = {
      name: "Frederick Douglass",
      canRead: false,
      canWrite: false
    };

    const selfEducate = person => ({
      ...person,
      canRead: true,
      canWrite: true
    });

    console.log(selfEducate(frederick));	// {name: 'Frederick Douglass', canRead: true, canWrite: true}
    console.log(frederick);					// {name: 'Frederick Douglass', canRead: false, canWrite: false}
	```
	- 这里的 selfEducate 是纯函数。该函数基于传入的值（即 person）做计算，返回一个新 person 对象，而不更改传入的参数，因此没有副作用。
	
14. 在 React 中，UI 使用纯函数表达。

15. 建议遵守下面三条规则编写函数：
	- 函数应该至少接受一个参数。
	- 函数应该返回一个值或另一个函数。
	- 函数不应该更改任何参数。

16. 在函数式编程中，数据从一种格式变成另一种格式。我们要做的式使用函数产出转换后的副本。这样的函数减少了代码的命令式属性，从而能降低复杂度。**若想精通 JavaScript 函数式编程，必须掌握两个核心函数，即 Array.map 和 Array.reduce。**

17. 我们可以使用 Array.join 函数得到一个以逗号分割的清单或其他格式的字符串。Array.join 是 JavaScript 内置的数组方法，作用是从数组中提取出以指定符号分割的字符串。
	```
	const schools = [
        "Yorktown",
        "Washington & Lee",
        "Wakefield"
    ];

    console.log(schools.join(", "));	// Yorktown, Washington & Lee, Wakefield
	```

18. 如果想定义一个函数，创建一个新数组，存储名称以”W“开头的学校，可以使用 Array.filter 方法。Array.filter 是 JavaScript 内置的函数，根据源数组产出一个新数组。该函数只接受一个参数，这个参数是一个断言，即一个始终返回布尔值 true 或 false 的函数。Array.filter 在数组的每一个元素上调用断言，元素作为参数传给断言，返回值决定是否把元素添加到新建的数组中，
	```
	const schools = [
      "Yorktown",
      "Washington & Lee",
      "Wakefield"
    ]

    const wSchools = schools.filter(school => school[0] === "W")
    console.log( wSchools )		// ['Washington & Lee', 'Wakefield']
	```

19. 若想从数组红删除元素，不建议使用 Array.pop 或 Array.splice，应该使用 Array.filter，因为 Array.filter 执行的是不可变操作。

20. Array.map 函数接受的参数是一个函数，这个函数是数组的每一个元素上调用一次，不管返回什么都添加到新数组中。
	```
	const schools = [
      "Yorktown",
      "Washington & Lee",
      "Wakefield"
    ]

    const highSchools = schools.map(school => ({ name: school }))
    console.log(highSchools)	// [{name: 'Yorktown'}, {name: 'Washington & Lee'}, {name: 'Wakefield'}]
	```
	- map 函数可以产出任何 JavaScript 类型的数组，包括对象数组、值数组、嵌套数组和函数数组。

21. 如果想把数组转换成对象，可以结合 Array.map 和 Object.keys。Object.keys 方法从对象中获取所有键，返回由键构成的数组。
	```
	const schools = {
      "Yorktown": 10,
      "Washington & Lee": 2,
      "Wakefield": 5
    }

    const schoolArray = Object.keys(schools).map(key => ({
      name: key,
      wins: schools[key]
    }))
	
    console.log(schoolArray)
	```

22. reduce 和 reduceRight 函数可用于把数组转换成任何值，包括数组、字符串、布尔值、对象，甚至是函数。
	```
	const ages = [21,18,42,40,64,63,34]

    const maxAge = ages.reduce((max, age) => {
      console.log(`${age} > ${max} = ${age > max}`)
      if (age > max) {
          return age
      } else {
          return max
      }
    }, 0)

    console.log('maxAge', maxAge)
	
	21 > 0 = true
	18 > 21 = false
	42 > 21 = true
	40 > 42 = false
	64 > 42 = true
	63 > 64 = false
	34 > 64 = false
	maxAge 64
	
	// 可简化为：
	const max = ages.reduce((max, value) => (value > max) ? value : max, 0)
	```
	- ages 数组被归约为一个值了，即最大年龄64。
	- **reduce 函数接受两个参数：一个回调函数和一个初始值。在这个示例中，初始值是0，即先把最大值设为0。回调函数在数组中的每个元素上调用一次。首次调用回调函数时，age等于21，即数组中的第一个值，max等于0，即设置的初始值。回调函数返回的这两个数中较大的那一个，即21，下次迭代将把这个值赋为max。每次迭代比较各个age值与max值，返回二者中较大的那个。最后，比较数组中最后一个数和前一次调用回调函数返回的数。**

23. Array.reduceRight 的作用与 Array.reduce 相同，只不过不从数组开头开始归约，而是从数组末尾开始。

24. 把数组转换成对象。
	```
	const colors = [
        {
            id: '-xekare',
            title: "rad red",
            rating: 3
        },
        {
            id: '-jbwsof',
            title: "big blue",
            rating: 2
        },
        {
            id: '-prigbj',
            title: "grizzly grey",
            rating: 5
        },
        {
            id: '-ryhbhsl',
            title: "banana",
            rating: 1
        }
    ]

    const hashColors = colors.reduce((hash, {id, title, rating}) => {
        hash[id] = {title, rating}
        return hash
    }, {})

    console.log(hashColors)
	```
	- 在这个示例中，传给 reduce 函数的第二个参数是一个空对象。这是创建散列的初始值。每次迭代，回调函数使用方括号句法为散列添加一个键，把键的值设为数组中 id 字段的值。

25. 使用 reduce 甚至可以把数组转换成完全不同的数组。
	```
	const colors = ["red", "red", "green", "blue", "green"];

    const uniqueColors = colors.reduce(
        (unique, color) =>
            (unique.indexOf(color) !== -1) ? unique : [...unique, color],
        []
    )

    console.log(distinctColors)
	```
	- 这个示例把 colors 数组归约为一个没有重复值的数组。

26. 高阶函数指用于处理其他函数的函数，其参数可以是函数，也可以返回函数，或者二者兼而有之。

27. 第一类高阶函数是接受其他函数为参数的函数。Array.map、Array.filter 和 Array.reduce 都接受函数作为参数，因此它们是高阶函数。
	```
	const invokeIf = (condition, fnTrue, fnFalse) => 
        (condition) ? fnTrue() : fnFalse()

    const showWelcome = () => console.log("Welcome!!!")

    const showUnauthorized = () => console.log("Unauthorized!!!")

    invokeIf(true, showWelcome, showUnauthorized)	// Welcome!!!
    invokeIf(false, showWelcome, showUnauthorized)	// Unauthorized!!!
	```

28. 返回一个函数的高阶函数有助于我们处理 JavaScript 复杂的异步操作，创建我们在必要时需要使用或重用的函数。
	```
	const getFakeMembers = count => new Promise((resolves, rejects) => {
      const api = `https://api.randomuser.me/?nat=US&results=${count}`
      const request = new XMLHttpRequest()
      request.open('GET', api)
      request.onload = () =>
           (request.status === 200) ?
            resolves(JSON.parse(request.response).results) :
            reject(Error(request.statusText))
      request.onerror = (err) => rejects(err)
      request.send()
    })

    const userLogs = userName => message =>
        console.log(`${userName} -> ${message}`)

    const log = userLogs("grandpa23")

    log("attempted to load 20 fake members")
    getFakeMembers(20).then(
        members => log(`successfully loaded ${members.length} members`),
        error => log("encountered an error loading members")
    )
	
	grandpa23 -> attempted to load 20 fake members
	grandpa23 -> successfully loaded 20 members
	或
	grandpa23 -> attempted to load 20 fake members
	grandpa23 -> encountered an error loading members
	```
	- userLogs 函数保存一些信息，返回一个函数，以便在其他信息可用时使用或重用。
	- userLogs 是高阶函数。log 函数由 userLogs 得来，每次调用 log 函数都会在消息前面加上“grandpa23”。

29. 递归技术指创建重新调用自身的函数。如果要解决的问题涉及循环，往往就可以转而使用递归函数。
	```
	const countdown = (value, fn) => {
        fn(value)
        return (value > 0) ? countdown(value-1, fn) : value
    }

    countdown(10, value => console.log(value))
	```
	- countdown 是递归函数。

30. 递归模式特别适合在异步操作中使用。函数在做好准备时调用本身，例如在数据可用时，或者当计时器结束时。
    ```
    const countdown = (value, fn, delay=1000) => {
        fn(value)
        return (value > 0) ?
            setTimeout(() => countdown(value-1, fn), delay) : value
    }

    const log = value => console.log(value)
    countdown(10, log)
    ```
    - 修改后的版本可用于创建倒计时钟表。

31. 递归技术适合在搜索数据结构中使用。可以递归迭代子文件夹，直到找出仅包含文件的文件夹为止。也可以递归迭代 HTML DOM，直到找到一个没有子节点的元素为止。
    ```
    const dan = {
        type: "person",
        data: {
          gender: "male",
          info: {
            id: 22,
            fullname: {
              first: "Dan",
              last: "Deacon"
            }
          }
        }
      }

    const deepPick = (fields, object={}) => {
       const [first, ...remaining] = fields.split(".")
       return (remaining.length) 
            ? deepPick(remaining.join("."), object[first])
            : object[first]
    }

    console.log( deepPick("type", dan) )    // person
    console.log( deepPick("data.info.fullname.first", dan) )    // Dan
    ```
    - 通过数组析构把第一个值与其余的值分开。

32. 函数式程序是把逻辑拆分成一系列关注特定任务的小型纯函数。最终，你要把这些小型函数整合到一起。合成的方式、模式和技术有很多种，你可能熟悉的一种是串接。在 JavaScript 中，函数可以使用点号记法串接在一起，依次处理前一个函数的返回值。

33. 字符串有 replace 方法。replace 方法返回一个模块字符串，而模板字符串也有 replace 方法。因此，可以使用点号记法把 replace 方法串接在一起处理字符串。
    ```
    const template = "hh:mm:ss tt"
    const clockTime = template.replace("hh", "03")
          .replace("mm", "33")
          .replace("ss", "33")
          .replace("tt", "PM")

    console.log(clockTime)  // 03:33:33 PM
    console.log(template)   // hh:mm:ss tt
    ```
    - template 自身完好无损，可以继续用于创建要显示的其他时间。

34. 当一个函数充当了在两个函数之间传递数据的管道，但像下面的例子，这样的句法难以理解，也不易维护和扩充。
    ```
    const createClockTime = date => ({date})

    const appendAMPM = ({date}) =>
        ({
            date,
            ampm: (date.getHours() >= 12) ? "PM" : "AM"
        })

    const civilianHours = clockTime => {
      const hours = clockTime.date.getHours()
      return {
        ...clockTime,
        hours: (hours > 12) ?
          hours - 12 :
          hours
      }
    }

    const removeDate = clockTime => {
      let newTime = {...clockTime}
      delete newTime.date
      return newTime
    }

    // Not the best way to compose all of these into one function

    const oneFunction = date =>
        removeDate(
            civilianHours(
                appendAMPM(
                    createClockTime(date)
                )
            )
        )

    console.log(oneFunction(new Date()))
    ```

35. 一种更为优雅的方法是创建一个高阶函数，把多个函数合成为一个大型函数。
    ```
    const createClockTime = date => ({date})

    const appendAMPM = ({date}) =>
      ({
          date,
          ampm: (date.getHours() >= 12) ? "PM" : "AM"
      })

    const civilianHours = clockTime => {
    const hours = clockTime.date.getHours()
      return {
        ...clockTime,
        hours: (hours > 12) ?
          hours - 12 :
          hours
      }
    }

    const removeDate = clockTime => {
      let newTime = {...clockTime}
      delete newTime.date
      return newTime
    }

    // A more elegant approach to composition

    const compose = (...fns) =>
      (arg) =>
      fns.reduce(
        (composed, f) => f(composed),
        arg
    )

    const oneFunction = compose(
      createClockTime,
      appendAMPM,
      civilianHours,
      removeDate
    )

    console.log(oneFunction(new Date()))   // { "ampm": "PM", "hours": 8 }
    ```
    - 这样的写法易于扩充，随时都可以添加更多函数。
    - 另外，这种方法也方便修改组合函数的顺序。
    - compose 函数是一个高阶函数，接受的参数是函数，只返回一个值。
    - compose 接受的参数是函数，返回一个函数。在上述实现中，我们使用展开运算符把通过参数传入的函数转换成一个名为 fns 的数组。返回的函数接受一个参数，即 arg。调用这个函数时，首先填充 fns 数组的是想通过该函数发送的参数。这个参数作为 compose 的初始值，随后各值是每次迭代 reduce 的回调返回的值。
    - 注意，回调函数接受两个参数：composed 和一个函数 f。调用每个函数时传入的参数是 composed，即前一个函数的输出结果。最终，将调用最后一个函数，返回最后的结果。 

36. 在函数式程序中，应该尽量使用函数，而非值。需要时，我们可以调用函数来获取值。
    ```
    const oneSecond = () => 1000
    const getCurrentTime = () => new Date()
    const clear = () => console.clear()
    const log = message => console.log(message)
    ```


---


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


---


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


---


###### 六、React 状态管理

1. React 应用的状态在数据的驱动下改变。

2. react-icons 是一个 npm 库，以 React 组件的形式分发，包含数百个 SVG 图标。安装这个库之后，我们便可以使用数个流行的图标库中数百个 SVG 图标。这个库中包含的全部图标可以到它的网站中查看（https://react-icons.netlify.com）
    ```
    npm i react-icons

    // 例子：
    import React from "react";
    import { FaStar } from "react-icons/fa";

    export default function StarRating() {
        return [
            <FaStar color="red" />,
            <FaStar color="red" />,
            <FaStar color="red" />,
            <FaStar color="grey" />,
            <FaStar color="grey" />
        ];
    }
    ```

3. 状态通过 React 的 Hooks（钩子）特性纳入函数组件。Hooks 是一组可重用的代码逻辑，放在组件树之外，用于接入组件的功能。React 自带多个钩子，可以直接使用。这里，我们想为 React 组件添加状态，要使用的是 React 钩子是 useState。这个钩子在 react 包中，导入即可：
    ```
    import React, { useState } from "react";
    import { FaStar } from "react-icons/fa";

    const createArray = length => [...Array(length)];

    const Star = ({ selected = false }) => (
        <FaStar color={selected ? "red" : "grey"} />
    );

    export default function StarRating({ totalStars = 5 }) {
        const [selectedStars] = useState(3);
        return (
            <>
            {createArray(totalStars).map((n, i) => (
                <Star key={i} selected={selectedStars > i} />
            ))}
            <p>
                {selectedStars} of {totalStars} stars
            </p>
            </>
        );
    }
    ```
    - createArray 函数：我们提供一个长度就能创建该长度的数组。
    - useState 钩子是一个函数，调用后返回一个数组。数组中的第一个值是我们想使用的状态变量。
    - 既然 useState 返回一个数组，那我们就可以利用数组析构把状态变量命名为任何名称。
    - 我们传给 useState 函数的值是状态变量的默认值。

4. 下面例子中，onSelect 属性是一个函数，这个属性的默认值是 f => f，这是一个虚假函数，什么也不做，只是返回传入的参数。然而，倘若不设置一个默认函数，onSelect 属性就是未定义的。虽然 f => f 什么也没做，但它仍是一个函数，调用时不会报错。
    ```
    import React from "react";
    import { FaStar } from "react-icons/fa";

    export default function Star({ selected = false, onSelect = f => f }) {
        return <FaStar color={selected ? "red" : "grey"} onClick={onSelect} />;
    }
    ```

5. useState 钩子返回的数组中的第二个元素是一个函数，可用于修改状态值。同样，通过析构数组，我们可以把那个函数命名为任何名称。

6. 使用钩子时有一件事一定要记住：钩子会导致所在的组件重新渲染。钩子中的数据发生变化后，钩子会使用新数据重新渲染所在的组件。

7. React 开发者工具会显示某个组件接入了哪些钩子。

8. 在 React 的旧版本中（v16.8.0之前），为组件添加状态只能使用类组件。这不仅需要更多的句法，还不便于在组件之间重用功能。钩子的出现就是为了解决类组件存在的问题，提供了一种把功能接入函数组件的方案。
    - 类组件在使用 this 关键字和函数绑定上还容易让人摸不着头脑。

9. 所有 React 元素都支持 style 属性，很多组件也有 style 属性。
    ```
    import React, { useState } from "react";
    import Star from "./Star";
    import { createArray } from "./lib";

    export default function StarRating({ style = {}, totalStars = 5, ...props }) {
    const [selectedStars, setSelectedStars] = useState(0);
        return (
            <div style={{ padding: 5, ...style }} {...props}>
                {createArray(totalStars).map((n, i) => (
                    <Star
                        key={i}
                        selected={selectedStars > i}
                        onSelect={() => setSelectedStars(i + 1)}
                    />
                ))}
                <p>
                    {selectedStars} of {totalStars} stars
                </p>
            </div>
        );
    }
    ```
    - 我们为这个 div 元素设置了默认 5px 的内边距，然后使用展开运算符从 style 对象中获取余下的样式属性，应用到 div 的样式上。
    - 我们通过展开运算符，即 ...props 收集用户想传给 StarRating 组件的所有属性。然后，把余下的所有属性传给 div 元素：{...props}。

10. 把状态集中管理的第一种做法是在状态树的根部存储状态，通过属性把状态传给子组件。

11. 纯组件指不含状态的函数组件，而且对给定的属性始终渲染得到相同的用户界面。

12. 沿组件树向上发送交互：我们要收集子组件的交互，沿组件树向上发送给根组件，在那里修改状态。
    - 就像数据可以通过属性沿着组件树向下传递一样，交互可以通过函数属性连同数据一起沿着组件树向上传递。

13. 在 DOM 中可用的 HTML 表单元素全部都有对应的 React 元素。

14. 在 React 中构建表单组件有好几种模式可用。在其中一个模式中，要使用 React 的一项特性直接访问 DOM 节点。这个特性称为 ref。在 React 中，ref 是一个对象，存储着一个组件整个生命期内的值。ref 适合在几个场景下使用，

15. 我们可以使用 React 提供的 useRef 钩子构建 ref。
    ```
    import React, { useRef } from "react";

    export default function AddColorForm({ onNewColor = f => f }) {
        const txtTitle = useRef();
        const hexColor = useRef();

        const submit = e => {
            e.preventDefault();
            const title = txtTitle.current.value;
            const color = hexColor.current.value;
            onNewColor(title, color);
            txtTitle.current.value = "";
            hexColor.current.value = "";
        };

        return (
            <form onSubmit={submit}>
            <input ref={txtTitle} type="text" placeholder="color title..." required />
            <input ref={hexColor} type="color" required />
            <button>ADD</button>
            </form>
        );
    }
    ```
    - 我们想使用基本的 HTML 表单验证，因此把两个输入框都标记为必须的（required）。
    - 我们在 JSX 中的输入框元素上添加了 ref 属性，用以设定 txtTitle 和 hexColor 两个 ref 的值。如此，在 ref 对象上就创建了一个 current 字段，直接引用 DOM 元素。
    - 默认情况下，提交 HTML 表单后向当前 URL 发送 POST 请求，表单元素的值存储在请求主体中。我们不想用这种行为，因此 submit 函数的第一行代码是 e.preventDefault()，即阻止浏览器使用 POST 请求提交表单。
    - 我们直接修改了 DOM 节点的 value 属性，把值设为空字符串。这是命令式代码。现在，AddColorForm 变成了不受控组件，因为它使用 DOM 保存表单值。有时，使用不受控组件可以帮助我们解决棘手的问题，例如，你可能想让 React 之外的代码访问表单及其值。然而，使用受控组件是更好的方案。

16. 在受控组件中，表单值由 React 管理，而非 DOM。此时，无须使用 ref，不用编写命令式代码，添加可靠的表单验证等功能也要容易得多。
    ```
    import React, { useState } from "react";

    export default function AddColorForm({ onNewColor = f => f }) {
        const [title, setTitle] = useState("");
        const [color, setColor] = useState("#000000");

        const submit = e => {
            e.preventDefault();
            onNewColor(title, color);
            setTitle("");
            setColor("");
        };

        return (
            <form onSubmit={submit}>
            <input
                value={title}
                onChange={event => setTitle(event.target.value)}
                type="text"
                placeholder="color title..."
                required
            />
            <input
                value={color}
                onChange={event => setColor(event.target.value)}
                type="color"
                required
            />
            <button>ADD</button>
            </form>
        );
    }
    ```
    - event.target 是对 DOM 元素的引用，因此我们可以使用 event.target.value 获取元素的当前值。
    - 这个组件之所以叫作受控组件，是因为 React 控制着表单的状态。值得注意的是，受控的表单组件经常重新渲染。

17. 我们可以把创建受控表单组件的详细过程独立打包，自定义为一个钩子。我们可以自己定义一个 useInput 钩子，去除创建受控的表单输入框过程中涉及的重复。
    ```
    // 当一个表单有大量 input 元素，下面两个语句将频繁地复制粘贴，再调整变量名称
    // value={color}
    // onChange={event => setColor(event.target.value)}
    import { useState } from "react";

    export const useInput = initialValue => {
        const [value, setValue] = useState(initialValue);
        return [
            { value, onChange: e => setValue(e.target.value) },
            () => setValue(initialValue)
        ];
    };
    ```
    - 返回一个数组，数组中的第一个值是一个对象，包含我们之前复制粘贴的属性：状态中的 value 和用于修改状态值的 onChange 函数属性。数组中的第二个值是一个函数，它的作用是把 value 重置为初始值。
    - 应用：
        ```
        import React from "react";
        import { useInput } from "./hooks";

        export default function AddColorForm({ onNewColor = f => f }) {
            const [titleProps, resetTitle] = useInput("");
            const [colorProps, resetColor] = useInput("#000000");

            const submit = e => {
                e.preventDefault();
                onNewColor(titleProps.value, colorProps.value);
                resetTitle();
                resetColor();
            };

            return (
                <form onSubmit={submit}>
                <input
                    {...titleProps}
                    type="text"
                    placeholder="color title..."
                    required
                />
                <input {...colorProps} type="color" required />
                <button>ADD</button>
                </form>
            );
        }
        ```
    - useState 钩子封装在我们自己定义的 useInput 钩子内。
    - 展开从自定义的钩子中获取的属性比直接复制粘贴属性有趣多了。
    - 在我们自定义的这个钩子中，如果状态发生了变化，仍会导致 AddColorForm 重新渲染。

18. 钩子应在 React 组件内部使用。在自定义的钩子内可以使用其他钩子，毕竟钩子最终要在组件内使用。

19. 我们可以使用 uuid 包中的 v4 函数为新颜色生成一个唯一的 id 值。
    ```
    import { v4 } from "uuid";
    ...
    const createColor = (title, color) => {
        const newColors = [
            ...colors,
            {
                id: v4(),
                rating: 0,
                title,
                color
            }
        ];
        setColors(newColors);
    };
    ...
    ```

20. 在 React 的早期版本中，把状态集中管理放在组件树的根部是一个重要的模式，但是在复杂的应用中集中于组件树的根部维护状态不是一件容易的事。
    - 每个组件都要接收只传给子组件的属性。
    - 状态数据通过属性经由一个个组件传递，一直传到需要使用状态的组件。

21. 为了把数据放入 React 上下文，我们要创建上下文供应组件（context provider）。这是一种 React 组件，可以包含整个组件树，也可以包含组件树的特定部分。
    - 使用上下文不妨碍我们集中在一处存储状态数据，只是不用再经过一堆用不到状态的组件传递数据了。

22. 在 React 中使用上下文，首先要把数据放入上下文供应组件，并把供应组件添加到组件树中。我们使用 React 提供的 createContext 函数创建上下文对象。这个对象包含两个组件：一个上下文 Provider 和一个 Consumer。
    ```
    import React, { createContext } from "react";
    import colors from "./color-data";
    import { render } from "react-dom";
    import App from "./App";

    export const ColorContext = createContext();

    render(
        <ColorContext.Provider value={{ colors }}>
            <App />
        </ColorContext.Provider>,
        document.getElementById("root")
    );
    ```
    - 我们要使用供应组件把颜色放到状态中。把数据添加到上下文中的方法是为 Provider 的 value 属性设值。这里，我们把一个包含 colors 的对象添加到上下文中。
    - 由于我们把整个 App 组件都放在供应组件内，因此 colors 数组可供整个组件树中的任何上下文消费组件使用。
    - 注意，我们还导出了 ColorContext。这是必须的一步，因为从上下文中获取 colors 时需要使用 ColorContext.Consumer。

23. 上下文供应组件不是总要包含整个应用。有时也会包含部分特定的组件，这样效率更高。供应组件只为所含的子组件提供上下值。
    - 一个应用中可以有多个上下文供应组件。
    - 很多支持 React 的 npm 包在背后就使用上下文。

24. 现在，我们在上下文中提供了 colors 值，App 组件无须再持有状态，并通过属性向下传给子组件了。如此一来，App 组件彻底不用接触颜色了，这很好，毕竟 App 组件本身也用不到颜色。

25. useContext 钩子用于从上下文中获取值，它从上下文 Consumer 中获取我们需要的值。
    ```
    import React, { useContext } from "react";
    import { ColorContext } from "./";
    import Color fron "./Color";

    export default function ColorList() {
        const { colors } = useContext(ColorContext);
        ...
    }
    ```
    - 为了从上下文中获取值，useContext 钩子要用到上下文实例。为此，我们从创建上下文及把供应组件添加到组件树中的 index.js 文件里导入了 ColorContext 实列。

26. Consumer 用 useContext 钩子访问，这意味着我们不用直接处理消费组件了。在钩子出现之前，若想从上下文中获取颜色，要在上下文消费组件中使用一种称为“渲染属性”（render props）的模式。渲染属性作为参数传给一个子函数。

27. 上下文供应组件可以把对象放入上下文，但是它自己无法修改上下文中的值，需要父级组件的协助。具体做法是创建一个有状态的组件，让它渲染上下文供应组件。当有状态的组件的状态发生变化时，将使用新的上下文数据重新渲染上下文供应组件。上下文供应组件的所有子组件也将使用新的上下文数据重新渲染。

28. 渲染上下文供应组件的有状态的组件是一个自定义的供应组件。我们将把 App 放在这个供应组件中。
    ```
    import React, { createContext, useState, useContext } from "react";
    import colorData from "./color-data.json";
    import { v4 } from "uuid";

    const ColorContext = createContext();
    export const useColors = () => useContext(ColorContext);

    export default function ColorProvider({ children }) {
        const [colors, setColors] = useState(colorData);

        const addColor = (title, color) =>
            setColors([
                ...colors,
                {
                    id: v4(),
                    rating: 0,
                    title,
                    color
                }
            ]);

        const rateColor = (id, rating) =>
            setColors(
                colors.map(color => (color.id === id ? { ...color, rating } : color))
            );

        const removeColor = id => setColors(colors.filter(color => color.id !== id));

        return (
            <ColorContext.Provider value={{ colors, addColor, removeColor, rateColor }}>
                {children}
            </ColorContext.Provider>
        );
    }
    ```
    - ColorProvider 组件负责渲染 ColorContext.Provider。在这个组件中，我们使用 useState 钩子创建了状态变量 colors。
    - 通过 ColorContext.Provider 的 value 属性把状态中的 colors 添加到上下文中。
    - ColorProvider 的所有子组件都将放入 ColorContext.Provider 中，因此可以在上下文中访问 colors 数组。
    - 我们也把 addColor 等函数添加到了上下文中了。这样，上下文消费组件便可以修改颜色数组的值。只要调用 setColors，colors 数组就会发生变化，从而导致 ColorProvider 重新渲染。
    - 引入钩子之后，我们完全不用把上下文开放给消费组件。我们可以把上下文包装到一个自定义的钩子中。我们不再对外开放 ColorContext 实例，而是创建一个名为 useColors 的钩子，从上下文中返回颜色。我们把渲染和处理有状态的颜色所需的全部功能都打包到一个 JavaScript 模块中，上下文隐藏在这个模块中，通过一个钩子对外开放。

29. 现在 Color 组件不用再通过函数属性把事件传给父级组件，它能访问上下文中的 rateColor 和 removeColor 函数。这两个函数通过 useColors 钩子就能轻易获取。
    ```
    import React from "react";
    import StarRating from "./StarRating";
    import { FaTrash } from "react-icons/fa";
    import { useColors } from "./ColorProvider";

    export default function Color({ id, title, color, rating }) {
        const { rateColor, removeColor } = useColors();
        return (
            <section>
            <h1>{title}</h1>
            <button onClick={() => removeColor(id)}>
                <FaTrash />
            </button>
            <div style={{ height: 50, backgroundColor: color }} />
            <StarRating
                selectedStars={rating}
                onRate={rating => rateColor(id, rating)}
            />
            </section>
        );
    }
    ```


---


###### 七、使用钩子增强组件

1. alert 是阻塞线程的一种好方式。
    ```
    import React, { useState } from "react";

    function Checkbox() {
        const [checked, setChecked] = useState(false);

        alert(`checked: ${checked.toString()}`);

        return (
            <>
            <input
                type="checkbox"
                value={checked}
                onChange={() => setChecked(checked => !checked)}
            />
            {checked ? "checked" : "not checked"}
            </>
        );`
    }

    export default function App() {
        return <Checkbox />;
    }

    ```
    - 我们把 alert 添加到渲染操作之前，阻塞渲染。在用户单击弹出框上的“确定”按钮之前，这个组件不会渲染。由于 alert 阻塞了线程，在单击“确定”按钮之前，复选框的下一个状态不会重新渲染。
    - 这可不是我们想要的效果，把 alert 放在 return 语句之后也不行，因为代码根本执行不到那一行。

2. 为了确保能正常看到弹出框，可以使用 useEffect。把 alert 放在 useEffect 函数中，渲染之后就会被调用了，这是一个副作用。
    ```
    import React, { useState, useEffect } from "react";

    function Checkbox() {
        const [checked, setChecked] = useState(false);

        useEffect(() => {
            alert(`checked: ${checked.toString()}`);
        });

        return (
            <>
            <input
                type="checkbox"
                value={checked}
                onChange={() => setChecked(checked => !checked)}
            />
            {checked ? "checked" : "not checked"}
            </>
        );
    }

    export default function App() {
        return <Checkbox />;
    }
    ```
    - 想让渲染产生副作用就使用 useEffect。副作用可以理解为函数在返回之外所做的事情。
    - 想让组件在返回 UI 之外去做的其他事情就叫作“效应”（useEffect 中的“effect”）。

3. alert、console.log，或者与浏览器或原生 API 的交互都不能作为渲染操作的一部分，不能写在 return 语句中。然而，在 React 应用中，渲染会影响这些事件的结果。useEffect 的作用是等待渲染结束，把值提供给 alert 或 console.log。

4. 我们还可以使用 useEffect 让添加到 DOM 中的某个文本输入框获得焦点。React 先渲染输出，再调用 useEffect，让元素获得焦点：
    ```
    useEffect(() => {
        txtInputRef.current.focus();
    });
    ```
    - 渲染之后，txtInputRef 就有值了，因此在这段代码中可以访问它的值，让元素获得焦点。每次渲染之后，useEffect 都能访问属性、状态、ref 等的最新值。

5. useEffect 相当于是在渲染之后调用的一个函数。渲染之后，我们便可以访问组件中当前的状态值，用这些值做其他事情。如果重新渲染了，一切都重来一遍。新值，新渲染，新效应。

6. 我们可以使用依赖数组，把 useEffect 钩子与特定的数据变化关联起来。依赖数组可以控制在什么时候调用 useEffect。
    ```
    useEffect(() => {
        console.log(`typing "${val}"`);
    }, [val]);

    useEffect(() => {
        console.log(`saved phrase: "${phrase}"`);
    }, [phrase]);
    ```
    - 第一个效应只在 val 的值发生变化时调用，第二个效应只在 phrase 的值发生变化时调用。

7. 依赖数组是一个数组，因此可以检查多个值。
    ```
    useEffect(() => {
        ...
    }, [val, phrase]);
    ```
    - 如果两个值中有一个发生了变化，就会调用这个效应。

8. useEffect 函数的第二个参数可以是一个空数组，此时只在首次渲染后调用效应。
    ```
    useEffect(() => {
        ...
    }, []);
    ```
    - 由于数组中没有依赖，因此这个效应只在首次渲染时调用。
    - 没有依赖意味着没有变化，所以这个效应以后都不会再调用。
    - 只在首次渲染时调用的效应特别适合用于初始化。

9. 如果效应返回一个函数，该函数将在把组件从组件树上移除时调用：
    ```
    useEffect(() => {
        welcomeChime.play();
        return () => goodbyeChime.play();
    }, []);
    ```
    - 这意味着，我们可以使用 useEffect 做事前设置和事后清理。
    - 我们提供的是一个空数组，因此欢迎乐只在首次渲染时播放。然后，返回一个函数，做清理工作，在从组件树上删除组件时播放送别乐。

10. 把功能分散到多个 useEffect 调用中通常是不错的注意。

11. 在 JavaScript 中，对数组、对象和函数来说，仅当完全是同一个实例时才相等。

12. 下面构建一个钩子，不管按什么键都渲染组件。
    ```
    import React, { useState, useEffect } from "react";

    const useAnyKeyToRender = () => {
        const [, forceRender] = useState();

        useEffect(() => {
            window.addEventListener("keydown", forceRender);
            return () => window.removeEventListener("keydown", forceRender);
        }, []);
    };

    export default function App() {
        useAnyKeyToRender();

        useEffect(() => {
            console.log("fresh render");
        });

        return <h1>Open the console</h1>;
    }
    ```
    - 只需调用改变状态的函数就能强制渲染。
    - 我们不关心状态的值，有改变状态的 forceRender 函数就行（鉴于此，使用数组析构时添加了一个逗号）。
    - 这个组件首次渲染时，监听按键事件。发现按键后，调用 forceRender，强制组件渲染。
    - 按照前面的做法，我们返回了一个清理函数，用于停止监听按键事件。
    - 为了验证该功能，每次渲染 App 组件都通过 useEffect 在控制台输出“fresh render”。

13. 下面例子中，只在首次渲染之后和 word 的值有变化时才调用 useEffect。这个值是不变的，因此后面不会重新渲染。在依赖数组中添加原始类型或数字均是如此，这个效应只被调用一次。
    ```
    const word = "gnar";
    useEffect(() => {
        console.log("fresh render");
    }, [word]);
    ```

14. words 变量的值是一个数组。由于每次渲染都新声明一个数组，因此在 JavaScript 看来，words 是有变化的，所以每次都会调用“fresh render”效应。每一次数组都是一个新实例，这被视为可触发重新渲染的变动。
    ```
    const words = ["sick", "powder", "day"];
    useEffect(() => {
        console.log("fresh render");
    }, [words]);
    ```

15. 在 App 的作用域之外声明 words 就可以解决上述问题。
    ```
    const words = ["sick", "powder", "day"];

    function App() {
        useAnyKeyToRender();

        useEffect(() => {
            console.log("fresh render");
        }, [words]);

        return <h1>Open the console</h1>;
    }
    ```
    - 这里的依赖数组引用的是在组件函数外部声明的同一个 words 实例。首次渲染之后，“fresh render”效应不再被调用，因为 words 实例始终不变。

16. 然而并不是所有变量都适合（或建议）在组件函数外部定义，有时传给依赖数组的值必须在作用域内定义。
    ```
    import React, { useEffect, useState, useMemo } from "react";

    function WordCount({ children = "" }) {
        useAnyKeyToRender();

        // const words = children.split(" ");
        const words = useMemo(() => children.split(" "), [children]);

        useEffect(() => {
            console.log("fresh render");
        }, [words]);

        return (
            <>
                <p>{children}</p>
                <p>
                    <strong>{words.length} - words</strong>
                </p>
            </>
        );
    }
    ```
    - useMemo 调用一个函数，计算得到一个备忘值。在计算机科学中，备忘技术一般用于提升性能。对支持备忘的函数来说，调用的函数得到的结果会被保存并缓存起来。以后如果使用相同的输入调用函数，返回的是缓存的值。在 React 中，我们使用 useMemo 比较缓存的值与当前值，判断值是不是真的变了。
    - 使用 useMemo 时要传入一个函数，用于计算并创建备忘值。仅当有依赖发生变化时，useMemo 才重新计算值。
    - useMemo 调用传给它的函数，把 words 设为该函数的返回值。与 useEffect 一样，useMemo 根据一个依赖数组做判断。
    - 如果没有为 useMemo 提供依赖数组，每次渲染都会计算 words 的值。依赖数组控制着何时调用回调函数。传给 useMemo 函数的第二个参数是依赖数组。
    - words 数组依赖 children 属性。如果 children 发生了变化，应该重新计算 words 的值，体现变化。useMemo 在组件首次渲染和 children 属性发生变化时重新计算 words 的值。

17. useCallback 的作用和 useMemo 类似，不过备忘的是函数而不是值。
    ```
    const fn = useCallback(() => {
        console.log("hello");
        console.log("world");
    }, []);

    useEffect(() => {
        console.log("fresh render");
        fn();
    }, [fn]);
    ```
    - useCallback 将备忘 fn 函数的值。与 useMemo 和 useEffect 一样，useCallback 的第二个参数也是一个依赖数组。这里，这个备忘回调只创建了一次，因为依赖数组是空。

18. React Profiler 是一个浏览器扩展，用于测试性能和检测 React 组件的渲染情况。

19. 我们知道，渲染始终发生在 useEffect 之前，渲染在前，然后各个效应按顺序运行，而且效应可以访问渲染后所有值。

20. useLayoutEffect 在渲染循环的特定时刻调用。这一系列事件是按照下述顺序发生的：
    1. 渲染。
    2. 调用 useLayoutEffect。
    3. 浏览器绘制，即把组件元素添加到 DOM 中。
    4. 调用 useEffect。

21. useLayoutEffect 在渲染之后，浏览器绘制变化之前调用。多数情况下，你需要的是 useEffect，但是如果要实现的效果对浏览器绘制很重要（屏幕上 UI 元素的外观或位置），那就要使用 useLayoutEffect。比如说，我们想要调整窗口的大小之后获取元素的宽度和高度：
    ```
    function useWindowSize() {
        const [width, setWidth] = useState(0);
        const [height, setHeight] = useState(0);

        const resize = () => {
            setWidth(window.innerWidth);
            setHeight(window.innerHeight);
        };

        useLayoutEffect(() => {
            window.addEventListener("resize", resize);
            resize();
            return () => window.removeEventListener("resize", resize);
        }, []);

        return [width, height];
    }
    ```
    - 你的组件可能想要在浏览器开始绘制之前知道窗口的 width 和 height，因此我们使用 useLayoutEffect 在绘制之前计算窗口的 width 和 height。

22. 跟踪鼠标的位置也要使用 useLayoutEffect，例如：
    ```
    function useMousePosition() {
        const [x, setX] = useState(0);
        const [y, setY] = useState(0);

        const setPosition = ({ x, y }) => {
            setX(x);
            setY(y);
        };

        useLayoutEffect(() => {
            window.addEventListener("mousemove", setPosition);
            return () => window.removeEventListener("mousemove", setPosition);
        }, []);

        return [x, y];
    }
    ```
    - 在绘制屏幕时很有可能需要使用鼠标的 x 和 y 坐标位置。在绘制之前，可以使用 useLayoutEffect 精确计算这两个坐标位置。

23. 钩子只在组件的作用域中运行：钩子只能在 React 组件中调用。钩子也可以添加到自定义的钩子中，不过最终也是添加到组件中。钩子不是常规的 JavaScript 代码，而是一种 React 模式，不过其他库也开始使用了。

24. 建议把功能分解到多个钩子中：这些写出的代码更易于阅读。此外还有一个好处，由于钩子是按顺序调用的，因此最好让钩子保持小的体量。调用钩子后，React 在一个数组中保存钩子的值，以便跟踪值。
    ```
    function Counter() {
        const [count, setCount] = useState(0);
        const [checked, toggle] = useState(false);

        useEffect(() => {
            ...
        }, [checked]);

        useEffect(() => {
            ...
        }, []);

        useEffect(() => {
            ...
        }, [count]);

        return (...)
    }
    ```
    - 每一次渲染钩子的调用顺序都是一样的：[count, checked, DependencyArray, DependencyArray, DependencyArray]

25. 钩子只应该在顶层代码中调用：钩子只应该在 React 函数的顶层代码中使用，不能放在条件语句、循环或嵌套函数中。
    ```
    function Counter() {
        const [count, setCount] = useState(0);
        if (count > 5) {
            const [checked, toggle] = useState(false);
        }

        useEffect(() => {
            ...
        });

        if (count > 5) {
            useEffect(() => {
                ...
            });
        }

        useEffect(() => {
            ...
        });

        return (...)
    }
    ```
    - 放在 if 语句中的 useState，意思是当 count 值大于 5 时调用钩子。这样钩子便游离在数组值之外了。有时数组是[count, checked, DependencyArray, 0, DependencyArray]，有时是[count, DependencyArray, 1]。效应在这个数组中的索引对 React 是很重要的，值就是按索引保存的。

26. 但是我们可以在钩子中嵌套 if 语句、循环和其他条件语句：
    ```
     function Counter() {
        const [count, setCount] = useState(0);
        if (count > 5) {
            const [checked, toggle] = useState(false);
        }

        const [checked, toggle] = useState(count => (count < 5) ? undefined : !c, (count < 5) ? undefined);

        useEffect(() => {
            ...
        });

        useEffect(() => {
            if (count > 5) return;
            ...
        });

        useEffect(() => {
            ...
        });

        return (...)
    }
    ```
    - 像这样把条件语句嵌套在钩子中，钩子依然在顶层，而最终的效果是类似的。
    - 这样可以确定钩子数组的值保持不变，始终为[countValue, checkedValue, DependencyArray, DependencyArray, DependencyArray]。

27. 与条件逻辑一样，异步操作也要嵌套到钩子中。useEffect 的第一个参数是一个函数，而不是一个 promise。因此，第一个参数不能是异步函数，例如 useEffect(async () => {})。然后，在嵌套的函数中可以创建异步函数，如下：
    ```
    useEffect(() => {
        const fn = async () => {
            await SomePromise();
        };
        fn();
    });
    ```
    - 我们创建变量 fn 处理 async/await，然后在函数结尾调用这个函数。
    - 我们可以为异步函数命名，也可以使用匿名函数：
        ```
        useEffect(() => {
            (async () => {
                await SomePromise();
            })();
        });
        ```

28. 遵守上述 23-27 点规则可以避免一些常见的 React 钩子陷阱。Create React App 包含一个名为 eslint-plugin-react-hoots 的 ESLint 插件，如果你违反了这些规则它会提醒你。

29. reducer 函数最简单的定义是，接收当前状态并返回新状态的函数。如果 checked 为 false，应该返回相反值 true。我们不再把这个行为硬编码在 onChange 事件中，而是提取到一个 reducer 函数中，始终生成相同的结果。
    ```
    function Checkbox() {
        const [checked, toggle] = useReducer(checked => !checked, false);

        return (
            <>
                <input type="checkbox" value={checked} onChange={toggle} />
                {checked ? "checked" : "not checked"}
            </>
        );
    }
    ```
    - useReducer 接受的参数为 reducer 函数和初始状态 false。然后，把 onChange 属性设为 toggle，调用 reducer 函数。

30. 如果为函数提供相同的输入，得到的输入也应该相同。这个概念源自 JavaScript 中的 Array.reduce。reduce 的作用与 reducer 函数基本相同：接收一个函数（把全部值归约为一个值）和一个初始值，返回一个值。

31. Array.reduce 接收一个 reducer 函数和一个初始值。numbers 数组中的各个值依次传给 reducer 函数，直到最终返回一个值。
    ```
    const numbers = [28, 34, 67, 68];
    numbers.reduce((number, nextNumber) => number + nextNumber, 0);  // 197
    ```
    - 这里，传给 Array.reduce 的 reducer 函数接收两个参数。此外，reducer 函数还可以接收更多参数。

32. 下面例子中，每次点击 h1，在总数上加 30。
    ```
    function Numbers() {
        const [number, setNumber] = useReducer(
            (number, newNumber) => number + newNumber,
            0
        );

        return <h1 onClick={() => setNumber(30)}>{number}</h1>;
    }
    ```

33. 管理状态时一个常见的错误是覆盖状态：
    ```
    const firstUser = {
        id: "0001",
        firstName: "xx",
        LastName: yy",
        admin: false
    }
    ...
    <button
        onClick{() => {
            setUser({admin: true});
        }}
    >
        Make Admin
    </button>
    ```
    - 这样做将覆盖 firstUser 状态，把状态替换为发给 setUser 函数的值，即{admin: true}。
    - 正确的做法是展开用户对象的当前值，然后覆盖 admin 值：
        ```
        <button
            onClick{() => {
                setUser({ ...user, admin: true});
            }}
        >
        Make Admin
        </button>
        ```
        - 现在是在初始状态的基础上增加新的键值对{admin: true}。

34. 我们可以把新状态值 newDetails 发给 reducer 函数，让它把新值推送进对象。
    ```
    const [user, setUser] = useReducer(
        (user, newDetails) => ({ ...user, ...newDetails }),
        firstUser
    );

    <button
        onClick={() => {
            setUser({ admin: true });
        }}
    >
        Make Admin
    </button>
    ```
    - 如果状态有多个子值，或者下一个状态依赖于前一个状态，就可以使用这种模式。

35. 在 React 之前的版本中，使用 setState 函数更新状态，初始状态要在构造方法中使用对象赋值。

36. 以前 setState 合并状态值。useReducer 也是如此。
    ```
    const [state, setState] = useReducer(
        (state, newState) => ({ ...state, ...newState }),
        initialState
    );
    ```
    - 如果你喜欢这个模式，可是使用 npm 包 legacy-set-state 或 useReducer。

37. 在 React 应用中，组件渲染的次数不在少数。提升组件性能涉及两方面，一是避免不必要的渲染，二是减少渲染传播的时间。React 自身提供了一些可用来避免非必要渲染的工具：memo、useMemo 和 useCallback。

38. memo 函数用于创建纯组件。在 React 中，对给定的属性，纯组件始终渲染相同的输出。
    ```
    const Cat = ({ name }) => {
        console.log(`readering ${name}`);
        return <p>{name}</p>
    };

    function App() {
        const [cats, setCats] = useState(["xx", "yy", "zz"]);
        return (
            <>
                {
                    cats.map((name, i) => (<Cat key={i} name={name} />));
                    <button onClick={() => setCats([...cats, prompt("New a cat")])}>
                        Add a Cat
                    </button>
                }
            </>
        );
    }
    ```
    - 首次渲染后，控制台中将输出：
        ```
        readering xx
        readering yy
        readering zz
        ```
    - 单击“Add a Cat”按钮将弹出一个窗口，让用户添加一个猫，假如添加一个名为“bb”的猫，渲染 Cat 组件输出的结果为：
        ```
        readering xx
        readering yy
        readering zz
        readering aa
        ```
    - 这段代码可以正常运行，因为 prompt 会阻塞代码执行。这里只是举例子，在真实的应用中不要使用 prompt。
    - 每增加一个猫，Cat 组件就多渲染一次，可 Cat 是纯组件，对给定的属性来说，输出并没有变化，不是每次都要渲染。

39. 使用 memo 函数可以创建只在属性有变化时渲染的组件。
    ```
    import React, { useState, memo } from "react";
    ...
    const PureCat = memo(Cat);
    ...
    cats.map((name, i) => <PureCat key={i} name={name} />);
    ```
    - 只有属性发生变化时，PureCat 才会渲染 Cat。
    - 现在，新增一只猫的名称后，只会看到 Cat 的一次渲染。由于其他猫的名称没有变化，因此不需要渲染对应 Cat 组件。
        ```
        readering aa
        ```

40. 要是为 Cat 组件增加一个函数属性 meow，PureCat 不能像设想中那样使用了，即使 name 属性保持不变，每个 Cat 组件还是全部渲染。每次定义的 meow 函数属性都是一个新函数。对 React 来说，meow 属性发生了变化，因此要重新渲染组件。

41. 我们可以使用 memo 函数定义更具体的规则，指明何时重新渲染组件：
    ```
    const PureCat = memo(
        Cat,
        (prevProps, nextProps) => prevProps.name === nextProps.name
    );
    ```
    - 传给 memo 函数的第二个参数是一个断言，即一个只返回 true 或 false 的函数。返回 false，重新渲染 Cat 组件；返回 true，不重新渲染 Cat 组件。无论如何，Cat 组件至少会渲染一次。
    - 断言函数能接收到前一组属性和下一组属性，我们就通过这两个对象比较 name 属性。

42. 在 React 之前的版本中，有个名为 shouldComponentUpdate 的方法。如果在组件中定义了这个方法，React 将据此判断在什么情况下应该更新组件。shouldComponentUpdate 定义哪些属性或状态发生变化时才应该重新渲染组件。由于 shouldComponentUpdate 在引入 React 库之后太受欢迎，React 团队甚至推出了一种创建类组件的新方式。
    - 类组件像下面这样定义：
        ```
        class Cat extends React.Component {
            render() {
                return (
                    {name} is a good cat!
                )
            }
        }
        ```
    - PureComponent 像下面这样定义：
        ```
        class Cat extends React.PureComponent {
            render() {
                return (
                    {name} is a good cat!
                )
            }
        }
        ```
        - PureComponent 的作用与 React.memo 相同，不过前者只适用类组件，而后者只适用函数组件。

43. useCallback 和 useMemo 可用于备忘对象和函数属性。
    ```
    const PureCat = memo(Cat);
    function App() {
        const meow = useCallback(name => console.log(`${name} has meowed`, []));
        return <PureCat name="aa", meow={meow} />
    }
    ```
    - 这里，我们没有为 memo(Cat) 提供检查属性的断言，而是使用 useCallback 确保 meow 函数没有变化。使用这些函数可以减少组件树种重新渲染的次数。

44. 可以使用 React Profiler 衡量各个组件的性能。React 开发者工具也带有分析程序。


---


###### 八、处理数据

1. 在 JavaScript 中，最常使用 fetch 发起 HTTP 请求。
    ```
    fetch(`https://api.github.com/users/moonhighway`)
        .then(response => response.json())
        .then(console.log)
        .catch(console.error);
    ```
    - fetch 函数返回一个 promise。
    - 这里，我们向指定的 URL 发起一个异步请求。
    - 传送完毕后，信息通过 .then(callback) 方法传给一个回调。
    - GitHub API 响应的数据是 JSON 格式，而且包含中 HTTP 响应的主体中，因此我们调用 response.json() 获取数据，把数据解析为 JSON 格式。
    - 获取响应信息后，把数据输出到控制台中。
    - 如果有什么地方出错了，把错误传给 console.error 方法。

2. promise 还可以使用 async/await 处理。由于 fetch 返回一个 promise，因此我们可以在 async 函数中异步等待（await）请求：
    ```
    async function requestGithubUser(githubLogin) {
        try {
            const response = await fetch(
                `https://api.github.com/users/${githubLogin}`
            );
            const userData = await response.json();
            console.log(userData);
        } catch (error) {
            console.error(error);
        }
    }

    requestGithubUser("moonhighway");
    ```
    - 这与前面串接 .then 函数处理请求的效果是完全一样的。
    - 在异步等待一个 promise 时，下一行代码直到 promise 得到解决后才执行。
    - 以这种方式在代码中处理 promise 比较简洁。

3. 一般来说，创建数据使用 POST 请求，修改数据使用 PUT 请求。fetch 函数的第二个参数接收一个选项对象，供 fetch 创建 HTTP 请求。
    ```
    fetch("/create/user", {
        method: "POST",
        body: JSON.stringify({ username, password, bio })
    })
    ```
    - JSON.stringify 是 JavaScript 中的一个函数，用于将一个 JavaScript 对象或值转换成一个 JSON 格式的字符串。
    - 这个请求使用 POST 方法创建一个新用户。username、password 和用户的 bio 以字符串的形式在请求的 body 中发送。

4. 上传文件需要使用一种不同的 HTTP 请求：multipart-formdata 请求。这种请求告诉服务器，请求的主体中有一个或多个文件。在 JavaScript 中发起这种请求，只需在请求的主体中传送一个 FormData 对象。
    ```
    const formData = new FormData();
    formData.append("username", "xx");
    formData.append("fullname", "yy");
    formData.append("avatar", imgFile);

    fetch("/create/user", {
        method: "POST",
        body: formData
    })
    ```
    - 通过一个 formData 对象随请求一起传送 username、fullname 和 avatar 图像。

5. 有时，我们要获得授权才能发起请求。通常，授权意味着获取个人或敏感数据。而且基本上要求用户通过 POST、PUT 或 DELETE 请求在服务器上执行一定的操作。

6. 一般情况下，我们在每个请求中添加一个唯一的令牌（token），供服务识别用户的身份。这个令牌通常在 Authorization 首部中添加。
    ```
    fetch(`https://api.github.com/users/${login}`, {
        method: "POST",
        headers: {
            Authorization: `Bearer ${token}`
        }
    });
    ```
    - 令牌通常在用户使用用户名和密码登录服务后获取。此外，也可以使用开放标准协议 OAuth 从第三方（例如 GitHub 或 Facebook）获取。

7. GitHub 可为你生成一个个人用户令牌。首先登录 GitHub，依次进入 Settings -> Developer Settings -> Personal Access Tokens。然后创建具有特定读写规则的令牌，最后使用令牌从 GitHub API 中获取个人信息。随 fetch 请求一起发送 Personal Access Tokens，GitHub 将提供更多有关账户的隐私信息。

8. 在 React 组件中获取数据要合理使用 useState 和 useEffect 钩子。前者的作用是把响应存储在状态中，后者的作用是发起 fetch 请求。
    ```
    import React, { useState, useEffect } from "react";

    function GitHubUser({ login }) {
        const [data, setData] = useState();

        useEffect(() => {
            if (!login) return;
            fetch(`https://api.github.com/users/${login}`)
                .then(response => response.json())
                .then(setData)
                .catch(console.error);
        }, [login]);

        if (data) 
            return <pre>{JSON.stringify(data, null, 2)}</pre>;

        return null;
    }

    export default function App() {
        return <GitHubUser login="moonhighway" />;
    }
    ```
    - 首次渲染时，由于 data 的初始值是 null，所以组件返回 null。组件返回 null 的意思是告诉 React，什么也不渲染。这样做没有问题，只是会看到一个黑屏。
    - JSON.stringify 方法接受三个参数：要转换成字符串的 JSON 数据，用于替换 JSON 对象中属性的函数，以及格式化数据时使用的空格数量。
    - 这里，我们把替换函数设为 null，即不做任何替换。2 表示格式化代码时使用的空格数量，即以两个空格缩进 JSON 字符串。
    - 使用 pre 元素的目的是保留空白，确保最终渲染出来的 JSON 字符串具有良好的阅读性。

9. 我们可以使用 Web Storage API 把数据存储在本地浏览器中。保存数据可以使用 window.localStorage 或 window.sessionStorage 对象。sessionStorage API 只把数据保存到用户会话中，关闭标签页或重启浏览器，保存的数据都将清空。而 localStorage 将无限期保存数据，除非主动删除。

10. JSON 数据应该以字符串的形式保存到浏览器存储空间中。这意味着，保存时要把对象转换成 JSON 对象，而在加载时则要把字符串解析为 JSON。下面是可用于保存和加载 JSON 数据的函数：
    ```
    const loadJSON = key => key && JSON.parse(localStorage.getItem(key));
    const saveJSON = (key, data) => localStorage.setItem(key, JSON.stringify(data));
    ```
    - 如果没有数据，loadJSON 函数返回 null。
    - 从 Web Storage 中加载数据、把数据保存到 Web Storage 中、把数据转换成字符串，以及解析 JSON 字符串，统统都是异步任务。loadJSON 和 saveJSON 都是异步函数。因此请注意，如果频繁调用这两个函数处理大量数据，可能会导致性能问题。为了性能，通常最好限制这两个函数的使用频率。

11. 例子：
    ```
    import React, { useState, useEffect } from "react";

    const loadJSON = key => key && JSON.parse(localStorage.getItem(key));
    const saveJSON = (key, data) => localStorage.setItem(key, JSON.stringify(data));

    function GitHubUser({ login }) {
        const [data, setData] = useState(loadJSON(`user:${login}`));

        useEffect(() => {
            if (!data) return;
            if (data.login === login) return;
            const { name, avatar_url, location } = data;
            saveJSON(`user:${login}`, {
                name,
                login,
                avatar_url,
                location
            });
        }, [data]);

        useEffect(() => {
            if (!login) return;
            if (data && data.login === login) return;
            fetch(`https://api.github.com/users/${login}`)
                .then(response => response.json())
                .then(setData)
                .catch(console.error);
        }, [login]);

        if (data) return <pre>{JSON.stringify(data, null, 2)}</pre>;

        return null;
    }

    export default function App() {
        return <GitHubUser login="moonhighway" />;
    }
    ```
    - loadJSON 是异步函数，因此调用 useState 时可以使用它设定状态数据的初始值。

12. 清空存储空间：
    ```
    localStorage.clear();
    ```

13. sessionStorage 和 localStorage 均是 Web 开发者的得力武器。
    - 离线时，我们可以使用本地数据，减少网络请求数，从而提升应用的性能。然而，一定要合理使用。实现离线存储增加了应用的复杂度，而且在开发的过程中很难处理。
    - 另外，请勿使用 Web Storage 缓存数据。如果发现性能有波动，可以试着让 HTTP 处理缓存。添加 Cache-Control: max-age=<EXP_DATE> 首部后，浏览器将自动缓存内容。EXP_DATE 定义内容的失效日期。

14. HTTP 请求和 promise 均有三种状态：待定、成功（完成）和失败（被拒）。发送请求后等待响应期间，请求处于待定状态。响应只有两种可能，成功或失败。成功的响应表示顺利连接上服务器，收到了数据。对 promise 来说，成功的响应表示 promise 得到解决了。倘若在请求的过程中有什么地方出错了，那就可以说 HTTP 请求失败，或者 promise 被拒了。在这两种情况下，我们将收到一个 error 对象，说明具体情况。

15. 发起 HTTP 请求时，这三种状态都要处理。
    ```
    import { useState, useEffect } from "react";

    function GitHubUser({ login }) {
        const [data, setData] = useState();
        const [error, setError] = useState();
        const [loading, setLoading] = useState(false);

        useEffect(() => {
            if (!login) return;
            setLoading(true);
            fetch(`https://api.github.com/users/${login}`)
                .then((data) => data.json())
                .then(setData)
                .then(() => setLoading(false))
                .catch(setError);
        }, [login]);

        if (loading) return <h1>loading...</h1>;
        if (error) return <pre>{JSON.stringify(error, null, 2)}</pre>;
        if (!data) return null;

        return (
            <div className="githubUser">
            <img src={data.avatar_url} alt={data.login} style={{ width: 200 }} />
            <div>
                <h1>{data.login}</h1>
                {data.name && <p>{data.name}</p>}
                {data.location && <p>{data.location}</p>}
            </div>
            </div>
        );
    }

    export default function App() {
        return <GitHubUser login="moonhighway" />;
    }
    ```
    - 请求处理待定状态时，我们可以显示一个“loading...”消息；如果出错了，那就渲染 error 对象中的详情。
    - 在生产环境中，我们可以进一步处理错误，比如说把跟踪到的错误记录到日志中，或者尝试再次发起请求。在开发环境中，只渲染错误详情没什么问题，反而能让开发者立即得到反馈。
    - 有些服务器会在成功的响应中发送额外的失败消息。

16. 在异步组件中，为了最大程度提升可重用性，经常利用渲染属性模式。采用这种模式创建组件，可以把开发应用所需的复杂机制或单调的样板代码抽离出来。
    ```
    import React from "react";

    const tahoe_peaks = [
        { name: "Freel Peak", elevation: 10891 },
        { name: "Monument Peak", elevation: 10067 },
        { name: "Pyramid Peak", elevation: 9983 },
        { name: "Mt. Tallac", elevation: 9735 }
    ];

    function List({ data = [], renderItem, renderEmpty }) {
        return !data.length ? (
            renderEmpty
        ) : (
            <ul>
            {data.map((item, i) => (
                <li key={i}>{renderItem(item)}</li>
            ))}
            </ul>
        );
    }

    export default function App() {
        return (
            <List
            data={tahoe_peaks}
            renderEmpty={<p>This list is empty</p>}
            renderItem={item => (
                <>
                {item.name} - {item.elevation.toLocaleString()}
                </>
            )}
            />
        );
    }
    ```

17. 生产环境中的应用通常要渲染大量数据，但是又不能一次性全部渲染。浏览器的渲染能力是有限的。渲染要耗费时间、占据处理能力及消耗内存，这三项都是有限制的。

18. 在用户滚动界面的过程中，我们要消除用户已经看过的结果，并渲染位于屏幕范围以外的新结果，随时准备展示。这种解决方案一次只渲染11个元素，其他数据排队等候，后面再渲染。这种技术称为虚拟化。使用这种技术滚动特大型列表，数据量无穷尽时，浏览器也不会崩溃。

19. 构建虚拟化列表组件要考虑很多事情。幸好，我们不用从头开始动手，社区已经开发了很多虚拟化列表组件，直接拿来使用即可。对浏览器渲染来说，最受欢迎的是 react-window 和 react-virtualized。虚拟化列表十分重要，React Native 甚至自带了一个这样的组件，即 FlatList。多数时候，我们无须自己动手构建虚拟化列表组件，知道如何使用就可以了。

20. 为了实现虚拟化列表，我们需要大量数据。这里指的是大量虚拟数据。
    ```
    npm i faker
    ```
    - 安装 faker 之后，我们可以创建一组大型的虚拟数据。
        ```
        import React from "react";
        import faker from "faker";

        const bigList = [...Array(5000)].map(() => ({
            name: faker.name.findName(),
            email: faker.internet.email(),
            avatar: faker.internet.avatar()
        }));

        function List({ data = [], renderItem, renderEmpty }) {
            return !data.length ? (
                renderEmpty
            ) : (
                <ul>
                {data.map((item, i) => (
                    <li key={i}>{renderItem(item)}</li>
                ))}
                </ul>
            );
        }

        export default function App() {
            const renderItem = item => (
                <div style={{ display: "flex" }}>
                <img src={item.avatar} alt={item.name} width={50} />
                <p>
                    {item.name} - {item.email}
                </p>
                </div>
            );

            return <List data={bigList} renderItem={renderItem} />;
        }
        ```
        - 我们映射一个有五千个空值的数组，把空值替换成虚拟用户的信息，创建 bigList 变量。

21. 下面使用 react-window 渲染这个虚拟用户列表：
    ```
    npm -i react-window
    ```
    - react-window 库提供了多个用于渲染虚拟列表的组件。
    - 下面的示例中，我们使用 react-window 提供的 FixedSizeList 组件：
        ```
        import React from "react";
        import { FixedSizeList } from "react-window";
        import faker from "faker";

        const bigList = [...Array(5000)].map(() => ({
            name: faker.name.findName(),
            email: faker.internet.email(),
            avatar: faker.internet.avatar()
        }));

        export default function App() {
            const renderRow = ({ index, style }) => (
                <div style={{ ...style, ...{ display: "flex" } }}>
                <img
                    src={bigList[index].avatar}
                    alt={bigList[index].name}
                    width={50}
                />
                <p>
                    {bigList[index].name} - {bigList[index].email}
                </p>
                </div>
            );

            return (
                <FixedSizeList
                    height={window.innerHeight}
                    width={window.innerWidth - 20}
                    itemCount={bigList.length}
                    itemSize={50}
                >
                    {renderRow}
                </FixedSizeList>
            );
        }
        ```
        - 通过 itemSize 属性指定每一行占据的像素数。
        - 渲染属性通过 children 属性传给 FixedSizeList。这种渲染属性模式时常用到。

22. 一个请求有三种状态：待定、成功或失败。为了重用 fetch 请求的这个逻辑，我们可以自定义一个钩子。在整个应用中，只要想发起 fetch 请求，就可以在组件中使用这个钩子。下面创建 useFetch 钩子：
    ```
    import React, { useState, useEffect } from "react";

    function useFetch(uri) {
        const [data, setData] = useState();
        const [error, setError] = useState();
        const [loading, setLoading] = useState(true);

        useEffect(() => {
            if (!uri) return;
            fetch(uri)
                .then(data => data.json())
                .then(setData)
                .then(() => setLoading(false))
                .catch(setError);
        }, [uri]);

        return {
            loading,
            data,
            error
        };
    }


    ```

23. 钩子最大的作用是跨组件重用的功能。有时，在不同的组件中需要重复渲染同样的内容。一个应用中，fetch 请求的错误处理方式或许也应该保持一致。下面创建一个 Fetch 组件：
    ```
    export default function Fetch({
        uri,
        renderSuccess,
        loadingFallback = <p>loading...</p>,
        renderError = error => <pre>{JSON.stringify(error, null, 2)}</pre>
    }) {
        const { loading, data, error } = useFetch(uri);
        if (loading) return loadingFallback;
        if (error) return renderError(error);
        if (data) return renderSuccess({ data });
    }

    // 如何使用 Fetch 组件如下：
    function GitHubUser({ login }) {
        return (
            <Fetch
            uri={`https://api.github.com/users/${login}`}
            renderSuccess={UserDetails}
            />
        );
    }

    function UserDetails({ data }) {
        return (
            <div className="githubUser">
            <img src={data.avatar_url} alt={data.login} style={{ width: 200 }} />
            <div>
                <h1>{data.login}</h1>
                {data.name && <p>{data.name}</p>}
                {data.location && <p>{data.location}</p>}
            </div>
            <UserRepositories
                login={data.login}
                onSelect={(repoName) => console.log(`${repoName} selected`)}
            />
            </div>
        );
    }
    ```
    - 自定义钩子 useFetch 是一层抽象，抽离了发起 fetch 请求的机制。Fetch 组件又是一层抽象，抽离了处理渲染什么的机制。

24. 下面创建一个名为 useIterator 的自定义钩子，迭代任意类型的对象数组：
    ```
    export const useIterator = (items = [], initialValue = 0) => {
        const [i, setIndex] = useState(initialValue);

        const prev = useCallback(() => {
            if (i === 0) return setIndex(items.length - 1);
            setIndex(i - 1);
        }, [i]);

        const next = useCallback(() => {
            if (i === items.length - 1) return setIndex(0);
            setIndex(i + 1);
        }, [i]);

        const item = useMemo(() => items[i], [i]);

        return [item || items[0], prev, next];
    };
    ```
    - 使用这个钩子可以遍历任何数组。由于这个钩子返回数组中的元素，因此我们可以利用数组析构为值提供有意义的名称：
        ```
        const [letter, previous, next] = useIterator([
            "a",
            "b",
            "c"
        ])
        ```
        - 这里，letter 的初始值是“a”。如果调用 next，组件将重新渲染，letter 的值变成“b”。再调用两次next，letter 的值又变成“a”，因为这个迭代器不会让 index 超出界限，绕了一圈又回到了数组的第一个元素。
    - 这里，prev 和 next 都使用 useCallback 钩子创建。这样可以确保 prev 函数始终保持不变，除非 i 的值发生变化。同样，item 的值也始终指向同一个元素对象，除非 i 的值发生变化。
    - 备忘这些值不会大幅提升性能，至少不足以让我们下决心增加代码的复杂度。然而，使用 useIterator 钩子时，备忘的值始终指向相同的对象和函数，这样方便比较值或者在依赖数组中使用。

25. 下面例子中，我们使用 useIterator 钩子，让用户遍览仓库列表：
    ```
    import React from "react";
    import { useIterator } from "./hooks";
    import RepositoryReadme from "./RepositoryReadme";

    export default function RepoMenu({ repositories, login }) {
        const [{ name }, previous, next] = useIterator(repositories);

        return (
            <>
            <div style={{ display: "flex" }}>
                <button onClick={previous}>&lt;</button>
                <p>{name}</p>
                <button onClick={next}>&gt;</button>
            </div>
            <RepositoryReadme login={login} repo={name} />
            </>
        );
    }
    ```
    - `&lt;` 是小于号的实体，显示一个小于号“<”。
    - 注意，使用数组析构可以为数组元素任意命名。即使在钩子中我们把函数命名为 prev 和 next，但是在使用钩子时，我们可以修改名称，改为 previous 和 next。
    - RepositoryReadme 组件在下面第 28 点有说明。

26. 瀑布式请求：一个接着一个地发起请求，前后请求之后有依赖关系，如果前一个请求出错了，后一个请求就不会再发起。

27. 仓库的 README 文件是使用 Markdown 编写，这是一种文本格式，可使用 ReactMarkdown 组件渲染为 HTML。安装 react-markdown 包：
    ```
    npm i react-markdown
    ```

28. 请求仓库的 README 文件内容也涉及瀑布式请求。首先，要向仓库 README 文件的路由发起数据请求。GitHub 对这个路由的响应是关于仓库 README 文件的详情，而不是文件内容。不过，响应中有个 download_url，我们可以通过它请求 README 文件的内容。可见，为了获取 Markdown 内容，要多发起一次请求。这里两个请求可以在同一个异步函数中发起：
    ```
    import React, { useState, useEffect, useCallback } from "react";

    import ReactMarkdown from "react-markdown";

    export default function RepositoryReadme({ repo, login }) {
        const [loading, setLoading] = useState(false);
        const [error, setError] = useState();
        const [markdown, setMarkdown] = useState("");

        const loadReadme = useCallback(async (login, repo) => {
            setLoading(true);
            const uri = `https://api.github.com/repos/${login}/${repo}/readme`;
            const { download_url } = await fetch(uri).then(res => res.json());
            const markdown = await fetch(download_url).then(res => res.text());
            setMarkdown(markdown);
            setLoading(false);
        }, []);

        useEffect(() => {
            if (!repo || !login) return;
            loadReadme(login, repo).catch(setError);
        }, [repo]);

        if (error) return <pre>{JSON.stringify(error, null, 2)}</pre>;
        if (loading) return <p>Loading...</p>;

        return <ReactMarkdown source={markdown} />;
    }
    ```
    - 使用 useCallback 钩子把 loadReadme 函数添加到组件中，在首次渲染组件时备忘该函数。

29. 所有请求都可在开发者工具的“Network”标签页中查看。在这个标签页中，你可以查看每一个请求，还可以限制网络速度，考察请求在较慢的网速下表现如何。

30. 在 Google Chrome 中如果想限制网络速度，单击“Online”旁边的箭头，在打开的菜单中，你可以选择不同的速度，选择“Fast 3G”和“Slow 3G”对网络请求的速度限制较大。另外，“Network”标签页还显示有全部 HTTP 请求的时间线。你可以筛选时间线，只查看“XHR”请求，即只显示 fetch 发出的请求。
    - 注意，加载图的标题是“Waterfall”。（读者笔记：除了名称、状态、类型、启动器、大小、时间之外，还可以像 Windows 任务管理器一样，勾选瀑布等其他列）

31. 有时候，可以一次发送全部请求，提升应用的速度。这种情况下，不再像瀑布式那样一个接一个发起请求，而是并行（或同时）发起所有请求。前面的例子之所以发送瀑布式请求，是因为组件是一个套一个渲染的，前一个组件未渲染之前不能发起下一个请求。如果在同一级渲染这三个组件，所有请求同时并行发起。

32. 我们并不能始终猜测出一开始要渲染什么数据。遇到这种情况，我们干脆不渲染组件，等到获得所需的数据再渲染。
    ```
    export default function App() {
        const [login, setLogin] = useState();
        const [repo, setRepo] = useState();
        return (
            <>
                <SearchForm value={login} onSearch={setLogin} />
                {login && <GitHubUser login={login} />}
                {login && (
                    <UserRepositoties
                        login={login}
                        repo={repo}
                        onSelect={setRepo}
                    />
                )}
                {login && repo && (
                    <RepositoryReadme login={login} repo={repo} />
                )}
            </SearchForm>
        );
    }
    ```
    - 这里，在所需的属性获得值之前，不渲染对应的组件。

33. 当 fetch 请求返回响应时，如果组件已经卸载完毕，试图修改已被卸载的组件的状态值会在控制台报错。
    - 只要用户在较慢的网速下加载数据，就有可能遇到这个错误。
    - 其实我们可以做些防护措施。首先，我们可以创建一个钩子，获知当前的组件有没有挂载：
        ```
        export function useMountedRef() {
            const mounted = useRef(false);
            useEffect(() => {
                mounted.current = true;
                return () => (mounted.current = false);
            });
            return mounted;
        }
        ```
        - 如果组件被卸载了，状态虽被清除了，但是 ref 还在。
        - 上面的 useEffect 没有依赖数组，每一次渲染组件都会调用，确保 ref 的值 true。只要卸载了组件，调用 useEffect 导致函数返回，ref 的值变成 false。
    - 使用例子：
        ```
        const mounted = useMountedRef();
        const loadReadme = useCallback(async (login, repo) => {
            setLoading(true);
            const uri = `https://api.github.com/repos/${login}/${repo}/readme`;
            const { download_url } = await fetch(uri).then(res => res.json());
            const markdown = await fetch(download_url).then(res => res.text());
            if (mounted.current) {
                setMarkdown(markdown);
                setLoading(false);
            }
        }, []);
        ```
        - 现在，我们可以通过一个 ref 判断组件有没有挂载。

34. React 为构建用户界面提供了一种声明式方案，GraphQL 为与 API 通信提供一种声明式方案。并行发起数据请求时，我们希望能同时获得所需的全部数据。GraphQL 就是为此而设计的。
    - 为了从 GraphQL API 获取数据，我们仍然要向指定的 URI 发起 HTTP 请求。不过，还要随请求一起发送查询。GraphQL 查询是对想要请求的数据的一种声明式描述。服务负责解析这个描述，把请求的所有数据打包进一个响应中。

35. 若想在 React 应用中使用 GraphQL，与之通信的后端服务要按照 GraphQL 规范构建。幸好，GitHub 也开放了 GraphQL API。多数 GraphQL 服务都提供有探索 GraphQL API 的方式。GitHub 提供的是 GraphQL Explorer（https://developer.github.com/v4/explorer）。

36. GraphQL 请求是在请求主体中包含查询的 HTTP 请求。我们可以使用 fetch 发起 GraphQL 请求。此外，也有一些专门的库和框架能辅助我们处理这类请求各方面的细节。
    - GraphQL 不限于 HTTP。GraphQL 是一个规范，规定如何通过网络发起数据请求。理论上，GraphQL 适用于任何网络协议。而且，GraphQL 不限定必须使用某种编程语言。

37. 安装 graphql-request 库：
    ```
    npm i graphql-request
    ```


---


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


---


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


---


###### 十一、React Router

1. 单页应用的一切内容都在同一个页面中，由 JavaScript 负责加载信息和变换 UI。如果没有路由方案，浏览器的很多功能，例如历史记录、收藏夹及前进和后退按钮都无法使用。路由为客户端请求定义端点，这些端点兼容浏览器的地址和历史记录对象。端点用于标识请求的内容，以便 JavaScript 加载和渲染相应的用户界面。

2. 路由器的作用是为网站中的各部分设置路由。一个路由就是一个端点，即可在浏览器的地址栏中输入的地址。请求某个路由，应用将渲染相应的内容。

3. 使用 React Router，我们要安装 React Router 和 React Router DOM。React Router DOM 供使用 DOM 的常规 React 应用使用。如果你编写的是 React Native 应用，要使用 react-router-native。

4. 现在，我们不再渲染 App 组件，而是渲染 Router 组件。Router 组件把当前地址的有关信息传给内部嵌套的子组件。Router 组件只能使用一次，而且应该放在组件树中靠近根的位置上：
    ```
    import React from "react";
    import { render } from "react-dom";
    import App from "./App";

    import { BrowserRouter as Router } from "react-router-dom";

    render(
        <Router>
            <App />
        </Router>,
        document.getElementById("root")
    );
    ```

5. 接下来要配置路由。我们把路由配置放在 App.js 文件中。要渲染的各个路由嵌套在名为 Routes 的组件中。在 Routes 中，一个页面使用一个 Route 组件设置。
    ```
    import React from "react";
    import {
        Routes,
        Route,
        Redirect
    } from "react-router-dom";
    import {
        Home,
        About,
        Events,
        Products,
        Contact,
        Whoops404,
        Services,
        History,
        Location
    } from "./pages";

    function App() {
        return (
            <div>
                <Routes>
                    <Route path="/" element={<Home />} />
                    <Route path="about" element={<About />}>
                        <Route
                            path="services"
                            element={<Services />}
                        />
                        <Route
                            path="history"
                            element={<History />}
                        />
                        <Route
                            path="location"
                            element={<Location />}
                        />
                    </Route>
                    <Route
                        path="events"
                        element={<Events />}
                    />
                    <Route
                        path="products"
                        element={<Products />}
                    />
                    <Route
                        path="contact"
                        element={<Contact />}
                    />
                    <Redirect
                        from="services"
                        to="about/services"
                    />
                    <Route path="*" element={<Whoops404 />} />
                </Routes>
            </div>
        );
    }

    export default App;
    ```
    - 这些路由告诉 React Router 在地址发送变化后渲染什么组件。每个 Route 组件都 path 和 element 属性。当浏览器中的地址匹配某个 path 时，显示对应的 element。
    - 可以在 Route 组件中嵌套 Route 组件（比如 About）。
    - 可以利用 Redirect 组件进行重定向（比如把访问 http://localhost:3000/services 的用户重定向到正确的路由 http://localhost:3000/about/services 上）。

6. 我们可以使用 react-router-dom 提供的 Link 组件创建链接。下面来修改首页，加入指向各个路由的导航链接目录：
    ```
    import { Link } from "react-router-dom";

    export function Home() {
        return (
            <div>
                <h1>[Company Website]</h1>
                <nav>
                    <Link to="about">About</Link>
                    <Link to="events">Events</Link>
                    <Link to="products">Products</Link>
                    <Link to="contact">Contact Us</Link>
                </nav>
            </div>
        );
    }
    ```

7. 我们可以使用当前地址值显示用户访问的路由：
    ```
    import { useLocation } from "react-router-dom";

    export function Whoops404() {
        let location = useLocation();
        console.log(location);
        return (
            <div>
                <h1>
                    Resource not found at {location.pathname}
                </h1>
            </div>
        );
    }
    ```

8. Router 组件只能使用一次，要路由的组件都嵌套在这个组件中。所有 Route 组件都嵌套在 Routes 组件中，根据浏览器中的当前地址选择渲染哪个组件。我们可以 Link 组件实现导航链接。

9. Route 组件只在匹配特定的 URL 时显示对应的内容。利用这个功能可以按照严格的层次结构组织 Web 应用中的不同页面，促进内容重用。

10. 嵌套路由时，若想显示嵌套内的组件（比如上面的 History），我们要使用 React Router DOM 的另一个特性：Outlet 组件。嵌套组件就使用 Outlet 组件渲染。想渲染子页面内容使用这个组件即可。
    ```
    import { Outlet } from "react-router-dom";

    export function About() {
        return (
            <div>
                <h1>[About]</h1>
                <Outlet />
            </div>
        );
    }

    export function History() {
        return (
            <section>
                <h2>Our History</h2>
                <p>
                    Lorem ipsum dolor sit amet, consectetur
                    adipiscing elit. Integer nec odio.
                    Praesent libero. Sed cursus ante dapibus
                    diam. Sed nisi. Nulla quis sem at nibh
                    elementum imperdiet. Duis sagittis ipsum.
                    Praesent mauris. Fusce nec tellus sed
                    augue semper porta. Mauris massa.
                    Vestibulum lacinia arcu eget nulla. Class
                    aptent taciti sociosqu ad litora torquent
                    per conubia nostra, per inceptos
                    himenaeos. Curabitur sodales ligula in
                    libero.
                </p>
            </section>
        );
    }
    ```
    - 现在，“About”页面都将重用 About 组件，分别显示各个嵌套的组件。应用通过地址判断该显示哪个子部分。例如，访问 http://localhost:3000/about/history 地址，渲染的是 About 组件内部的 History 组件。

11. Redirect 组件的作用是把用户重定向到指定的路由。

12. 也可以使用 useRoutes 钩子配置应用的路由。
    ```
    import React from "react";
    import { useRoutes } from "react-router-dom";
    import {
        Home,
        About,
        Events,
        Products,
        Contact,
        Whoops404,
        Services,
        History,
        Location
    } from "./pages";

    function App() {
        let element = useRoutes([
            { path: "/", element: <Home /> },
            {
                path: "about",
                element: <About />,
                children: [
                    {
                        path: "services",
                        element: <Services />
                    },
                    { path: "history", element: <History /> },
                    {
                        path: "location",
                        element: <Location />
                    }
                ]
            },
            { path: "events", element: <Events /> },
            { path: "products", element: <Products /> },
            { path: "contact", element: <Contact /> },
            { path: "*", element: <Whoops404 /> },
            {
                path: "services",
                redirectTo: "about/services"
            }
        ]);
        return element;
    }
    ```
    - Route 是对 useRoutes 的包装，其实使用哪种句法都可以。

13. React Router 的另一个有用特性是为路由设置参数。路由参数是一种变量，作用是从 URL 中获取相应的值。路由参数在数据驱动型 Web 应用中特别有用，可用于筛选内容或管理显示偏好设置。下面举个例子：
    - 首先，打开 index.js 文件，设置路由器。
        ```
        import React from "react";
        import { render } from "react-dom";
        import { BrowserRouter as Router } from "react-router-dom";
        import App from "./Colors";

        render(
            <Router>
                <App />
            </Router>,
            document.getElementById("root")
        );
        ```
        - 嵌套 App 组件之后，路由的所有属性都将传给它及它的子组件。
    - 然后，在 App 中添加路由。
        ```
        import React from "react";
        import { Routes, Route } from "react-router-dom";
        import AddColorForm from "./AddColorForm";
        import ColorList from "./ColorList";
        import { ColorDetails } from "./ColorDetails";
        import "./Colors.css";
        import { ColorProvider } from "./hooks";
        export * from "./hooks";

        export default function App() {
            return (
                <ColorProvider>
                    <AddColorForm />
                    <Routes>
                        <Route
                            path="/"
                            element={<ColorList />}
                        ></Route>
                        <Route
                            path=":id"
                            element={<ColorDetails />}
                        />
                    </Routes>
                </ColorProvider>
            );
        }
        ```
    - ColorDetails 组件根据颜色的 id 动态显示内容。例如，localhost:3000/00fdb4c5-c5bd-4087-a48f-4ff7a9d90af8。
        ```
        import React from "react";
        import { useColors } from "./";
        import { useParams } from "react-router-dom";

        export function ColorDetails() {
            let { id } = useParams();
            let { colors } = useColors();

            let foundColor = colors.find(
                color => color.id === id
            );

            return (
                <div>
                <div
                    style={{
                    backgroundColor: foundColor.color,
                    height: 100,
                    width: 100
                    }}
                ></div>
                <h1>{foundColor.title}</h1>
                <h1>{foundColor.color}</h1>
                </div>
            );
        }
        ```
14. 我们想为这个颜色管理应用添加的另一个功能是，点击颜色列表中的某个颜色后打开 ColorDetails 页面。下面在 Colors 组件中添加这个功能。为此，我们要使用另一个路由器钩子 useNavigate，在用户点击该组件时打开详情页面。
    - 先从 react-router-dom 中导入这个钩子：
        ```
        import { useNavigate } from "react-router-dom";
        ```
    - 然后，调用 useNavigate 钩子，返回一个函数，导航到另一个页面：
        ```
        let navigate = useNavigate();
        ```
    - 接下来，为 section 元素添加一个 onClick 处理函数，根据颜色的 id 导航到正确的路由：
        ```
        let navigate = useNavigate();

        return (
            <section
                className="color"
                onClick={() => navigate(`/${id}`)}
            >
                // Color 组件
            </section>
        );
        ```
    - 现在，单击这个 section 元素就可以转到正确的页面了。

15. 路由参数是获取影响用户界面呈现内容所需数据的理想工具。然而，我们应该只在希望用户从 URL 中捕获这些信息时才使用。


---


###### 十二、React 服务器端渲染

1. 我们可以采用同构方式渲染 React，以便支持浏览器以外的平台。这意味着，我们可以在服务器端渲染 UI，然后再发给浏览器。借助服务器端渲染，可以提升性能、增进可移植性、提高安全性。

2. 同构（isomorphic）和普适（universal）这两个术语通常都指在客户端和服务器端都可以运行的应用。虽然二者往往可以互换，但还是有点儿区别的。同构应用指可在多个平台上渲染的应用，而普适代码指同一套代码可在多个环境中运行。

3. 有了 Node.js，我们可以在其他应用，例如服务器端应用、CLI 应用，甚至是原生应用中重用为浏览器编写的代码。

4. 不是所有 JavaScript 代码不经一点改动就能在两端运行。
    - Node.js 和浏览器不同，没有内置 fetch 函数。
    - 在 Node.js 中，我们可以使用 npm 中的 isomorphic-fetch，也可以使用内置的 https 模块。
        ```
        // 既然我们已经使用了 fetch 句法，那就用 isomorphic-fetch：
        npm install isomorphic-fetch
        ```
    - 然后，只需导入 isomorphic-fetch，其他代码无需改动：
        ```
        const fetch = require("isomorphic-fetch");

        const userDetails = response => {
            const login = response.login;
            console.log(login);
        };

        fetch("https://api.github.com/users/moonhighway")
            .then(res => res.json())
            .then(userDetails);
        ```

5. JSX 会编译成 JavaScript。Star 组件就是一个函数而已。这个组件可以直接在浏览器渲染，也可以捕获输出的 HTML 字符串，在其他环境中渲染。ReactDOM 提供的 renderToString 方法可以把 UI 渲染为 HTML 字符串：
    ```
    // 在浏览器中直接渲染 HTML
    ReactDOM.render(<Star />);

    // 把 HTML 渲染为字符串
    let html = ReactDOM.renderToString(<Star />);
    ```

6. 我们可以构建同构应用，在不同的平台中渲染组件，还可以通过一定的架构方式，跨多个平台全局重用 JavaScript 代码。另外，使用其他编程语言，例如 Go 或 Python 也可以构建同构应用，不限于只能使用 Node.js。

7. 使用 ReactDOM.renderToString 方法可以在服务器端渲染 UI。

8. 构建应用时，客户端渲染通常是首选方案。我们伺服 Create React App 生成的 build 文件夹，在浏览器中运行 HTML，通过 script.js 文件加载所需的 JavaScript 代码。
    - 但是这个方案有点儿耗时。用户可能要等一会儿才能看到内容加载出来，具体等待事件取决于网络速度。
    - Create React App 搭配 Express 服务器可以实现一种混合客户端和服务端渲染的体验。

9. 改动客户端的代码，首先要做的改动是，把 ReactDOM.render 换成 ReactDOM.hydrate。
    - 这两个函数的作用相同，只不过 hydrate 把内容添加到一个由 ReactDOMServer 渲染的容器中。操作顺序是这样的：
        1. 渲染应用的静态版本，让用户知道应用在做事，“加载了”页面。
        2. 请求动态 JavaScript 代码。
        3. 把静态内容替换为动态内容。
        4. 用户点击，交互正常。
    - 这其实就是在服务端渲染之后把内容再输送给应用。再输送是指先加载静态 HTML 内容，然后加载 JavaScript。这样做能让用户在感官上体验到性能的顺畅。

10. 接下来要设置项目的服务器。我们将使用轻量级 Node 服务器 Express。
    - 先安装：
        ```
        npm install express
        ```
    - 然后，创建一个服务器文件夹，命名为 server，再新建一个文件 index.js，保存在该文件夹中。我们在这个文件中构建一个服务器，让它伺服 build 文件夹，另外预先加载静态 HTML 内容。
        ```
        import express from "express";
        const app = express();

        app.use(express.static("./build"));
        ```
        - 这段代码导入服务器，静态伺服 build 文件夹。
    - 接着，使用 ReactDOM 提供的 renderToString 函数，以静态 HTML 字符串的形式渲染应用：
        ```
        import React from "react"; 
        import ReactDOMServer from "react-dom/server";
        import { Menu } from "../src/Menu.js";

        const PORT = process.env.PORT || 4000;

        app.get("/*", (req, res) => {
            const app = ReactDOMServer.renderToString(
                <Menu />
            );

            const indexFile = path.resolve(
                "./build/index.html"
            );

            fs.readFile(indexFile, "utf8", (err, data) => {
                return res.send(
                data.replace(
                    '<div id="root"></div>',
                    `<div id="root">${app}</div>`
                )
                );
            });
        });

        app.listen(PORT, () =>
            console.log(
                `Server is listening on port ${PORT}`
            )
        );
        ```
        - 我们要把 Menu 组件传给 renderToString 函数，因为这是我们想静态渲染的组件。
        - 然后，我们要从构建好的客户端应用中读取静态的 index.html 文件，把应用的内容写入 div 元素，并把这部分内容作为请求的响应。

11. 完成以上操作后，我们要配置 webpack 和 Babel。虽然 Create React App 自身就能编译和构建，但是我们要为服务器端设置和实施一些不同的规则。
    - 首先安装几个依赖：
        ```
        npm install @babel/core @babel/preset-env babel-loader nodemon npm-run-all webpack webpack-cli webpack-node-externals
        ```
    - 安装好 Babel 之后，创建 .babelrc 文件，写入下述预设规则：
        ```
        {
            "presets": ["@babel/preset-env", "react-app"]
        }
        ```
    - 接下来，为服务器增加一个 webpack 配置文件，名为 webpack.server.js：
        ```
        const path = require("path");
        const nodeExternals = require("webpack-node-externals");

        module.exports = {
            entry: "./server/index.js",

            target: "node",

            externals: [nodeExternals()],

            output: {
                path: path.resolve("build-server"),
                filename: "index.js"
            },

            module: {
                rules: [
                    {
                        test: /\.js$/,
                        use: "babel-loader"
                    }
                ]
            }
        };
        ```
    - babel-loader 的作用是转换 JavaScript 文件，而 nodeExternals 则负责扫描 node_modules 文件夹，找到所有 node_modules 名称，构建一个外部函数，告诉 webpack 不要打包这些模块及其子模块。
    - 另外，你可能会遇到一个有关 webpack 的错误，这是由 Create React App 安装的版本与我们自己安装的版本发生冲突导致的。这个冲突的解决方法是，在项目根目录中创建 .env 文件，写入下述内容：
        ```
        SKIP_PREFLIGHT_CHECK=true
        ```
    - 最后，可以添加几个 npm 脚本，运行开发相关的命令：
        ```
        "scripts": {
            // ...
            "dev:build-server": "NODE_ENV=development webpack --config webpack.server.js --mode=development -w",
            "dev:start": "nodemon ./server-build/index.js",
            "dev": "npm-run-all --parallel build dev:*"
        },
        ```
        - dev:build-server：设置环境变量 development，使用新服务器配置运行 webpack。
        - dev:start：使用 nodemon 运行服务器文件，监听变动。
        - dev：并行运行前两个进程。
    - 现在，执行 npm run dev 命令，两个进程都会运行。你会看到应用运行在 localhost:4000 地址上。这个应用将按顺序加载内容，先预渲染 HTML，再渲染 JavaScript 构建包。

12. 在服务器端渲染生态环境中，还有一个使用广泛的强大工具，即 Next.js。Next.js 旨在帮助工程人员更轻松地编写服务器端渲染的应用。这个工具提供的功能有直观的路由、静态优化、自动分拆代码等。

13. 基于 React 的另一个受欢迎的网站生成工具是 Gatsby。Gatsby 为创建内容驱动型网站提供了一种简单直观的方式。Gatsby 的默认设置足够智能，性能、可访问性、图像处理等都不用我们自己操心。
    - Gatsby 可以创建的项目类型很多，不过通常用于构建内容驱动型网站。也就是说，对博客或静态内容网站而言，Gatsby 是个不错的选择，尤其是对 React 熟悉的人。此外，Gatsby 也可以处理动态内容，例如从 API 中加载数据、与框架集成等。

14. 如果你想构建移动应用，可以使用 React Native；如果你想使用声明式句法获取数据，可以使用 GraphQL；如果你想构建内容类网站，可以进一步探索 Next.js 和 Gatsby。
