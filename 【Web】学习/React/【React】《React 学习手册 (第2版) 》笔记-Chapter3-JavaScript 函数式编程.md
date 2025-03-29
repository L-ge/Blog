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
