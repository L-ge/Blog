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
