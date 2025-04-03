###### 十三、客户端检测

1. 要检测当前的浏览器有很多方法，每一种都有各自的长处和不足。问题的关键在于知道客户端检测
应该是解决问题的最后一个举措。任何时候，只要有更普适的方案可选，都应该毫不犹豫地选择。首先
要设计最常用的方案，然后再考虑为特定的浏览器进行补救。

2. 能力检测（又称特性检测）即在 JavaScript 运行时中使用一套简单的检测逻辑，测试浏览器是否支
持某种特性。这种方式不要求事先知道特定浏览器的信息，只需检测自己关心的能力是否存在即可。
	- 能力检测的基本模式如下：
		```
		if (object.propertyInQuestion) { 
			// 使用 object.propertyInQuestion 
		}
		```
	- 比如，IE5 之前的版本中没有 document.getElementById()这个 DOM 方法，但可以通过 document.all 属性实现同样的功能。为此，可以进行如下能力检测：
		```
		function getElement(id) { 
			if (document.getElementById) { 
				return document.getElementById(id); 
			} else if (document.all) { 
				return document.all[id]; 
			} else { 
				throw new Error("No way to retrieve element!"); 
			} 
		}
		```
		- 这个 getElement()函数的目的是根据给定的 ID 获取元素。因为标准的方式是使用 document. getElementById()，所以首先测试它。如果这个函数存在（不是 undefined），那就使用这个方法；否则检测 document.all 是否存在，如果存在则使用。如果这两个能力都不存在（基本上不可能），则抛出错误说明功能无法实现。
	- 能力检测的关键是理解两个重要概念。首先，如前所述，应该先检测最常用的方式。在前面的例子中就是先检测 document.getElementById()再检测 document.all。测试最常用的方案可以优化代码执行，这是因为在多数情况下都可以避免无谓检测。
	- 其次是必须检测切实需要的特性。某个能力存在并不代表别的能力也存在。比如下面的例子：
		```
		function getWindowWidth() { 
			if (document.all) { // 假设 IE 
				return document.documentElement.clientWidth; // 不正确的用法！
			} else { 
				return window.innerWidth; 
			} 
		}
		```
		- 这个例子展示了不正确的能力检测方式。getWindowWidth()函数首先检测 document.all 是否存在，如果存在则返回 document.documentElement.clientWidth，理由是 IE8 及更低版本不支持window.innerWidth。这个例子的问题在于检测到 document.all 存在并不意味着浏览器是 IE。事实，也可能是某个早期版本的 Opera，既支持 document.all 也支持 windown.innerWidth。

3. 能力检测最有效的场景是检测能力是否存在的同时，验证其是否能够展现出预期的行为。
	- 前一节中的例子依赖将测试对象的成员转换类型，然后再确定它是否存在。虽然这样能够确定检测的对象成员存在，但不能确定它就是你想要的。来看下面的例子，这个函数尝试检测某个对象是否可以排序：
		```
		// 不要这样做！错误的能力检测，只能检测到能力是否存在
		function isSortable(object) { 
			return !!object.sort; 
		}
		```
		- 这个函数尝试通过检测对象上是否有 sort()方法来确定它是否支持排序。问题在于，即使这个对象有一个 sort 属性，这个函数也会返回 true：
			```
			let result = isSortable({ sort: true });
			```
	- 简单地测试到一个属性存在并不代表这个对象就可以排序。更好的方式是检测 sort 是不是函数：
		```
		// 好一些，检测 sort 是不是函数
		function isSortable(object) { 
			return typeof object.sort == "function"; 
		}
		```
		- 上面的代码中使用的 typeof 操作符可以确定 sort 是不是函数，从而确认是否可以调用它对数据进行排序。

	- 进行能力检测时应该尽量使用 typeof 操作符，但光有它还不够。尤其是某些宿主对象并不保证对 typeof 测试返回合理的值。最有名的例子就是 Internet Explorer（IE）。在多数浏览器中，下面的代码都会在 document.createElement()存在时返回 true：
		```
		// 不适用于 IE8 及更低版本
		function hasCreateElement() { 
			return typeof document.createElement == "function"; 
		}
		```
		- 但在 IE8 及更低版本中，这个函数会返回 false。这是因为 typeof document.createElement返回"object"而非"function"。前面提到过，DOM 对象是宿主对象，而宿主对象在 IE8 及更低版本中是通过 COM 而非 JScript 实现的。因此，document.createElement()函数被实现为 COM 对象，typeof 返回"object"。IE9 对 DOM 方法会返回"function"。
	- 要深入了解 JavaScript 能力检测，推荐阅读 Peter Michaux 的文章“Feature Detection—State of the Art Browser Scripting”。

4. 虽然可能有人觉得能力检测类似于黑科技，但恰当地使用能力检测可以精准地分析运行代码的浏览器。使用能力检测而非用户代理检测的优点在于，伪造用户代理字符串很简单，而伪造能够欺骗能力检测的浏览器特性却很难。

5. 检测特性。可以按照能力将浏览器归类。如果你的应用程序需要使用特定的浏览器能力，那么最好集中检测所有能力，而不是等到用的时候再重复检测。比如：
	```
	// 检测浏览器是否支持 Netscape 式的插件
	let hasNSPlugins = !!(navigator.plugins && navigator.plugins.length); 
	
	// 检测浏览器是否具有 DOM Level 1 能力
	let hasDOM1 = !!(document.getElementById && document.createElement && 
			document.getElementsByTagName);
	```
	- 这个例子完成了两项检测：一项是确定浏览器是否支持 Netscape 式的插件，另一项是检测浏览器是否具有 DOM Level 1 能力。保存在变量中的布尔值可以用在后面的条件语句中，这样比重复检测省事多了。

6. 检测浏览器
	- 可以根据对浏览器特性的检测并与已知特性对比，确认用户使用的是什么浏览器。这样可以获得比用户代码嗅探（稍后讨论）更准确的结果。但未来的浏览器版本可能不适用于这套方案。
	- 下面来看一个例子，根据不同浏览器独有的行为推断出浏览器的身份。这里故意没有使用 navigator.userAgent 属性，后面会讨论它：
		```
		class BrowserDetector { 
			constructor() { 
				// 测试条件编译
				// IE6~10 支持
				this.isIE_Gte6Lte10 = /*@cc_on!@*/false; 
				
				// 测试 documentMode 
				// IE7~11 支持
				this.isIE_Gte7Lte11 = !!document.documentMode;
				
				// 测试 StyleMedia 构造函数
				// Edge 20 及以上版本支持
				this.isEdge_Gte20 = !!window.StyleMedia; 
			 
				// 测试 Firefox 专有扩展安装 API 
				// 所有版本的 Firefox 都支持
				this.isFirefox_Gte1 = typeof InstallTrigger !== 'undefined'; 
			 
				// 测试 chrome 对象及其 webstore 属性
				// Opera 的某些版本有 window.chrome，但没有 window.chrome.webstore 
				// 所有版本的 Chrome 都支持
				this.isChrome_Gte1 = !!window.chrome && !!window.chrome.webstore; 
			 
				// Safari 早期版本会给构造函数的标签符追加"Constructor"字样，如：
				// window.Element.toString(); // [object ElementConstructor] 
				// Safari 3~9.1 支持
				this.isSafari_Gte3Lte9_1 = /constructor/i.test(window.Element); 
			 
				// 推送通知 API 暴露在 window 对象上
				// 使用默认参数值以避免对 undefined 调用 toString() 
				// Safari 7.1 及以上版本支持
				this.isSafari_Gte7_1 = 
					(({pushNotification = {}} = {}) => 
						pushNotification.toString() == '[object SafariRemoteNotification]' 
					)(window.safari); 
			 
				// 测试 addons 属性
				// Opera 20 及以上版本支持
				this.isOpera_Gte20 = !!window.opr && !!window.opr.addons; 
		} 
		isIE() { return this.isIE_Gte6Lte10 || this.isIE_Gte7Lte11; } 
		isEdge() { return this.isEdge_Gte20 && !this.isIE(); } 
		isFirefox() { return this.isFirefox_Gte1; } 
		isChrome() { return this.isChrome_Gte1; } 
		isSafari() { return this.isSafari_Gte3Lte9_1 || this.isSafari_Gte7_1; } 
		isOpera() { return this.isOpera_Gte20; } 
		}
		```
		- 这个类暴露的通用浏览器检测方法使用了检测浏览器范围的能力测试。随着浏览器的变迁及发展，可以不断调整底层检测逻辑，但主要的 API 可以保持不变。

7. 能力检测的局限。
	- 通过检测一种或一组能力，并不总能确定使用的是哪种浏览器。
	- 注意，能力检测最适合用于决定下一步该怎么做，而不一定能够作为辨识浏览器的标志。

8. 用户代理检测
	- 用户代理检测通过浏览器的用户代理字符串确定使用的是什么浏览器。用户代理字符串包含在每个HTTP 请求的头部，在 JavaScript 中可以通过 navigator.userAgent 访问。在服务器端，常见的做法是根据接收到的用户代理字符串确定浏览器并执行相应操作。而在客户端，用户代理检测被认为是不可靠的，只应该在没有其他选项时再考虑。
	- 用户代理字符串最受争议的地方就是，在很长一段时间里，浏览器都通过在用户代理字符串包含错误或误导性信息来欺骗服务器。

9. 浏览器分析
	- 想要知道自己代码运行在什么浏览器上，大部分开发者会分析 window.navigator.userAgent返回的字符串值。所有浏览器都会提供这个值，如果相信这些返回值并基于给定的一组浏览器检测这个字符串，最终会得到关于浏览器和操作系统的比较精确的结果。
	- 相比于能力检测，用户代理检测还是有一定优势的。能力检测可以保证脚本不必理会浏览器而正常执行。

10. 伪造用户代理
	- 通过检测用户代理来识别浏览器并不是完美的方式，毕竟这个字符串是可以造假的。只不过实现window.navigator 对象的浏览器（即所有现代浏览器）都会提供 userAgent 这个只读属性。因此，简单地给这个属性设置其他值不会有效：
		```
		console.log(window.navigator.userAgent); 
		// Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/65.0.3325.181 Safari/537.36 
		
		window.navigator.userAgent = 'foobar'; 
		
		console.log(window.navigator.userAgent); 
		// Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/65.0.3325.181 Safari/537.36
		```
	- 不过，通过简单的办法可以绕过这个限制。比如，有些浏览器提供伪私有的__defineGetter__方法，利用它可以篡改用户代理字符串：
		```
		console.log(window.navigator.userAgent); 
		// Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/65.0.3325.181 Safari/537.36 
		
		window.navigator.__defineGetter__('userAgent', () => 'foobar'); 
		
		console.log(window.navigator.userAgent); 
		// foobar
		```
		- 对付这种造假是一件吃力不讨好的事。检测用户代理是否以这种方式被篡改过是可能的，但总体来看还是一场猫捉老鼠的游戏。
		- 与其劳心费力检测造假，不如更好地专注于浏览器识别。如果相信浏览器返回的用户代理字符串，那就可以用它来判断浏览器。如果怀疑脚本或浏览器可能篡改这个值，那最好还是使用能力检测。

11. 分析浏览器
	- 通过解析浏览器返回的用户代理字符串，可以极其准确地推断出下列相关的环境信息：
		- 浏览器
		- 浏览器版本
		- 浏览器渲染引擎
		- 设备类型（桌面/移动）
		- 设备生产商
		- 设备型号
		- 操作系统
		- 操作系统版本
	- 当然，新浏览器、新操作系统和新硬件设备随时可能出现，其中很多可能有着类似但并不相同的用户代理字符串。因此，用户代理解析程序需要与时俱进，频繁更新，以免落伍。自己手写的解析程序如果不及时更新或修订，很容易就过时了。相反，这里推荐一些 GitHub 上维护比较频繁的第三方用户代理解析程序：
		- Bowser 
		- UAParser.js 
		- Platform.js 
		- CURRENT-DEVICE 
		- Google Closure 
		- Mootools
	- Mozilla 维基有一个页面“Compatibility/UADetectionLibraries”，其中提供了用户代理解析程序的列表，可以用来识别 Mozilla 浏览器（甚至所有主流浏览器）。这些解析程序是按照语言分组的。这个页面好像维护不频繁，但其中给出了所有主流的解析库。（注意JavaScript 部分包含客户端库和 Node.js 库。）GitHub 上的文章“Are We Detectable Yet?”中还有一张可视化的表格，能让我们对这些库的检测能力一目了然。

12. 软件与硬件检测。现代浏览器提供了一组与页面执行环境相关的信息，包括浏览器、操作系统、硬件和周边设备信息。这些属性可以通过暴露在 window.navigator 上的一组 API 获得。不过，这些 API 的跨浏览器支持还不够好，远未达到标准化的程度。注意，强烈建议在使用这些 API 之前先检测它们是否存在，因为其中多数都不是强制性的，且很多浏览器没有支持。另外，本节介绍的特性有时候不一定可靠。

13. 识别浏览器与操作系统
	- 特性检测和用户代理字符串解析是当前常用的两种识别浏览器的方式。而 navigator 和 screen对象也提供了关于页面所在软件环境的信息。
	- navigator.oscpu 属性是一个字符串，通常对应用户代理字符串中操作系统/系统架构相关信息。根据 HTML 实时标准：oscpu 属性的获取方法必须返回空字符串或者表示浏览器所在平台的字符串，比如"Windows NT 10.0; Win64; x64"或"Linux x86_64"。
	- navigator.vendor 属性是一个字符串，通常包含浏览器开发商信息。返回这个字符串是浏览器navigator 兼容模式的一个功能。根据 HTML 实时标准：navigator.vendor 返回一个空字符串，也可能返回字符串"Apple Computer, Inc."或字符串"Google Inc."。
	- navigator.platform 属性是一个字符串，通常表示浏览器所在的操作系统。根据 HTML 实时标准：navigator.platform 必须返回一个字符串或表示浏览器所在平台的字符串，例如"MacIntel"、"Win32"、"FreeBSD i386"或"WebTV OS"。
	- screen.colorDepth 和 screen.pixelDepth 返回一样的值，即显示器每像素颜色的位深。根据CSS 对象模型（CSSOM）规范：screen.colorDepth 和 screen.pixelDepth 属性应该返回输出设备中每像素用于显示颜色的位数，不包含 alpha 通道。
	- screen.orientation 属性返回一个 ScreenOrientation 对象，其中包含 Screen Orientation API定义的屏幕信息。
		- 这里面最有意思的属性是 angle 和 type，前者返回相对于默认状态下屏幕的角度，后者返回以下 4 种枚举值之一：
			- portrait-primary
			- portrait-secondary
			- landscape-primary
			- landscape-secondary
		- 例如，在 Chrome 移动版中，screen.orientation 返回的信息如下：
			```
			// 垂直看
			console.log(screen.orientation.type); // portrait-primary 
			console.log(screen.orientation.angle); // 0 
			
			// 向左转
			console.log(screen.orientation.type); // landscape-primary 
			console.log(screen.orientation.angle); // 90 
			
			// 向右转
			console.log(screen.orientation.type); // landscape-secondary 
			console.log(screen.orientation.angle); // 270
			```
			- 根据规范，这些值的初始化取决于浏览器和设备状态。因此，不能假设 portrait-primary 和 0始终是初始值。这两个值主要用于确定设备旋转后浏览器的朝向变化。

14. 浏览器元数据。navigator 对象暴露出一些 API，可以提供浏览器和操作系统的状态信息。

15. Geolocation API
	- navigator.geolocation 属性暴露了 Geolocation API，可以让浏览器脚本感知当前设备的地理位置。这个 API 只在安全执行环境（通过 HTTPS 获取的脚本）中可用。
	- 这个 API 可以查询宿主系统并尽可能精确地返回设备的位置信息。根据宿主系统的硬件和配置，返回结果的精度可能不一样。手机 GPS 的坐标系统可能具有极高的精度，而 IP 地址的精度就要差很多。
	- 根据 Geolocation API 规范：地理位置信息的主要来源是 GPS 和 IP 地址、射频识别（RFID）、Wi-Fi 及蓝牙 Mac 地址、GSM/CDMA 蜂窝 ID 以及用户输入等信息。
	- 注意，浏览器也可能会利用 Google Location Service（Chrome 和 Firefox）等服务确定位置。有时候，你可能会发现自己并没有 GPS，但浏览器给出的坐标却非常精确。浏览器会收集所有可用的无线网络，包括 Wi-Fi 和蜂窝信号。拿到这些信息后，再去查询网络数据库。这样就可以精确地报告出你的设备位置。
	- 要获取浏览器当前的位置，可以使用 getCurrentPosition()方法。这个方法返回一个Coordinates 对象，其中包含的信息不一定完全依赖宿主系统的能力：
		```
		// getCurrentPosition()会以 position 对象为参数调用传入的回调函数
		navigator.geolocation.getCurrentPosition((position) => p = position);
		```
		- 这个 position 对象中有一个表示查询时间的时间戳，以及包含坐标信息的 Coordinates 对象：
			```
			console.log(p.timestamp); // 1525364883361 
			console.log(p.coords); // Coordinates {...}
			```
	- Coordinates 对象中包含标准格式的经度和纬度，以及以米为单位的精度。精度同样以确定设备位置的机制来判定。
		```
		console.log(p.coords.latitude, p.coords.longitude); // 37.4854409, -122.2325506 
		console.log(p.coords.accuracy); 					// 58
		```
	- Coordinates 对象包含一个 altitude（海拔高度）属性，是相对于 1984 世界大地坐标系（World Geodetic System，1984）地球表面的以米为单位的距离。此外也有一个 altitudeAccuracy 属性，这个精度值单位也是米。为了取得 Coordinates 中包含的这些信息，当前设备必须具备相应的能力（比如 GPS 或高度计）。很多设备因为没有能力测量高度，所以这两个值经常有一个或两个是空的。
		```
		console.log(p.coords.altitude); // -8.800000190734863 
		console.log(p.coords.altitudeAccuracy); // 200
		```
	- Coordinates 对象包含一个 speed 属性，表示设备每秒移动的速度。还有一个 heading（朝向）属性，表示相对于正北方向移动的角度（0 ≤ heading < 360）。为获取这些信息，当前设备必须具备相应的能力（比如加速计或指南针）。很多设备因为没有能力测量高度，所以这两个值经常有一个是空的，或者两个都是空的。
	- 注意，设备不会根据两点的向量来测量速度和朝向。不过，如果可能的话，可以尝试基于两次连续的测量数据得到的向量来手动计算。当然，如果向量的精度不够，那么计算结果的精度肯定也不够。
	- 获取浏览器地理位置并不能保证成功。因此 getCurrentPosition()方法也接收失败回调函数作为第二个参数，这个函数会收到一个 PositionError 对象。在失败的情况下，PositionError 对象中会包含一个 code 属性和一个 message 属性，后者包含对错误的简短描述。code 属性是一个整数，表示以下 3 种错误。
		- PERMISSION_DENIED：浏览器未被允许访问设备位置。页面第一次尝试访问 Geolocation API时，浏览器会弹出确认对话框取得用户授权（每个域分别获取）。如果返回了这个错误码，则要么是用户不同意授权，要么是在不安全的环境下访问了 Geolocation API。message 属性还会提供额外信息。
		- POSITION_UNAVAILABLE：系统无法返回任何位置信息。这个错误码可能代表各种失败原因，但相对来说并不常见，因为只要设备能上网，就至少可以根据 IP 地址返回一个低精度的坐标。
		- TIMEOUT：系统不能在超时时间内返回位置信息。关于如何配置超时，会在后面介绍。
			```
			// 浏览器会弹出确认对话框请用户允许访问 Geolocation API 
			// 这个例子显示了用户拒绝之后的结果 
			navigator.geolocation.getCurrentPosition( 
				() => {}, 
				(e) => { 
					console.log(e.code); 	// 1 
					console.log(e.message); // User denied Geolocation 
				} 
			); 
			
			// 这个例子展示了在不安全的上下文中执行代码的结果
			navigator.geolocation.getCurrentPosition( 
				() => {}, 
				(e) => { 
					console.log(e.code); 	// 1 
					console.log(e.message); // Only secure origins are allowed 
				} 
			);
			```
	- Geolocation API 位置请求可以使用 PositionOptions 对象来配置，作为第三个参数提供。这个对象支持以下 3 个属性。
		- enableHighAccuracy：布尔值，true 表示返回的值应该尽量精确，默认值为 false。默认情况下，设备通常会选择最快、最省电的方式返回坐标。这通常意味着返回的是不够精确的坐标。比如，在移动设备上，默认位置查询通常只会采用 Wi-Fi 和蜂窝网络的定位信息。而在enableHighAccuracy 为 true 的情况下，则会使用设备的 GPS 确定设备位置，并返回这些值的混合结果。使用 GPS 会更耗时、耗电，因此在使用 enableHighAccuracy 配置时要仔细权衡一下。
		- timeout：毫秒，表示在以 TIMEOUT 状态调用错误回调函数之前等待的最长时间。默认值是0xFFFFFFFF（232 – 1）。0 表示完全跳过系统调用而立即以 TIMEOUT 调用错误回调函数。
		- maximumAge：毫秒，表示返回坐标的最长有效期，默认值为 0。因为查询设备位置会消耗资源，所以系统通常会缓存坐标并在下次返回缓存的值（遵从位置缓存失效策略）。系统会计算缓存期，如果 Geolocation API 请求的配置要求比缓存的结果更新，则系统会重新查询并返回值。0 表示强制系统忽略缓存的值，每次都重新查询。而 Infinity 会阻止系统重新查询，只会返回缓存的值。JavaScript 可以通过检查 Position 对象的 timestamp 属性值是否重复来判断返回的是不是缓存值。

16. Connection State 和 NetworkInformation API 
	- 浏览器会跟踪网络连接状态并以两种方式暴露这些信息：连接事件和 navigator.onLine 属性。在设备连接到网络时，浏览器会记录这个事实并在 window 对象上触发 online 事件。相应地，当设备断开网络连接后，浏览器会在 window 对象上触发 offline 事件。任何时候，都可以通过 navigator.onLine 属性来确定浏览器的联网状态。这个属性返回一个布尔值，表示浏览器是否联网。
		```
		const connectionStateChange = () => console.log(navigator.onLine); 
		window.addEventListener('online', connectionStateChange); 
		window.addEventListener('offline', connectionStateChange); 
		
		// 设备联网时：
		// true 
		
		// 设备断网时：
		// false
		```
	- 当然，到底怎么才算联网取决于浏览器与系统实现。有些浏览器可能会认为只要连接到局域网就算“在线”，而不管是否真正接入了互联网。
	- navigator 对象还暴露了 NetworkInformation API，可以通过 navigator.connection 属性使用。这个 API 提供了一些只读属性，并为连接属性变化事件处理程序定义了一个事件对象。
	- 以下是 NetworkInformation API 暴露的属性。
		- downlink：整数，表示当前设备的带宽（以 Mbit/s 为单位），舍入到最接近的 25kbit/s。这个值可能会根据历史网络吞吐量计算，也可能根据连接技术的能力来计算。
		- downlinkMax：整数，表示当前设备最大的下行带宽（以 Mbit/s 为单位），根据网络的第一跳来确定。因为第一跳不一定反映端到端的网络速度，所以这个值只能用作粗略的上限值。
		- effectiveType：字符串枚举值，表示连接速度和质量。这些值对应不同的蜂窝数据网络连接技术，但也用于分类无线网络。这个值有以下 4 种可能。
			- slow-2g
				- 往返时间 ＞ 2000ms 
				- 下行带宽 ＜ 50kbit/s 
			- 2g
				- 2000ms ＞ 往返时间 ≥ 1400ms 
				- 70kbit/s ＞ 下行带宽 ≥ 50kbit/s 
			- 3g
				- 1400ms ＞ 往返时间 ≥ 270ms 
				- 700kbit/s ＞ 下行带宽 ≥ 70kbit/s 
			- 4g
				- 270ms ＞ 往返时间 ≥ 0ms 
				- 下行带宽 ≥ 700kbit/s 
		- rtt：毫秒，表示当前网络实际的往返时间，舍入为最接近的 25 毫秒。这个值可能根据历史网络吞吐量计算，也可能根据连接技术的能力来计算。
		- type：字符串枚举值，表示网络连接技术。这个值可能为下列值之一。
			- bluetooth：蓝牙。
			- cellular：蜂窝。
			- ethernet：以太网。
			- none：无网络连接。相当于 navigator.onLine === false。
			- mixed：多种网络混合。
			- other：其他。
			- unknown：不确定。
			- wifi：Wi-Fi。
			- wimax：WiMAX。
		- saveData：布尔值，表示用户设备是否启用了“节流”（reduced data）模式。
		- onchange：事件处理程序，会在任何连接状态变化时激发一个 change 事件。可以通过 navigator.connection.addEventListener('change',changeHandler)或 navigator.connection.onchange = changeHandler 等方式使用。

17. Battery Status API
	- 浏览器可以访问设备电池及充电状态的信息。navigator.getBattery()方法会返回一个期约实例，解决为一个 BatteryManager 对象。
		```
		navigator.getBattery().then((b) => console.log(b)); 
		// BatteryManager { ... }
		```
	- BatteryManager 包含 4 个只读属性，提供了设备电池的相关信息。
		- charging：布尔值，表示设备当前是否正接入电源充电。如果设备没有电池，则返回 true。
		- chargingTime：整数，表示预计离电池充满还有多少秒。如果电池已充满或设备没有电池，则返回 0。
		- dischargingTime：整数，表示预计离电量耗尽还有多少秒。如果设备没有电池，则返回 Infinity。
		- level：浮点数，表示电量百分比。电量完全耗尽返回 0.0，电池充满返回 1.0。如果设备没有电池，则返回 1.0。
	- 这个 API 还提供了 4 个事件属性，可用于设置在相应的电池事件发生时调用的回调函数。可以通过给 BatteryManager 添加事件监听器，也可以通过给事件属性赋值来使用这些属性。
		- onchargingchange
		- onchargingtimechange
		- ondischargingtimechange
		- onlevelchange
	- 例子：
		```
		navigator.getBattery().then((battery) => { 
			// 添加充电状态变化时的处理程序
			const chargingChangeHandler = () => console.log('chargingchange'); 
			battery.onchargingchange = chargingChangeHandler; 
			// 或
			battery.addEventListener('chargingchange', chargingChangeHandler); 
			
			// 添加充电时间变化时的处理程序
			const chargingTimeChangeHandler = () => console.log('chargingtimechange'); 
			battery.onchargingtimechange = chargingTimeChangeHandler; 
			// 或
			battery.addEventListener('chargingtimechange', chargingTimeChangeHandler); 
			
			// 添加放电时间变化时的处理程序
			const dischargingTimeChangeHandler = () => console.log('dischargingtimechange'); 
			battery.ondischargingtimechange = dischargingTimeChangeHandler; 
			// 或
			battery.addEventListener('dischargingtimechange', dischargingTimeChangeHandler);
			
			// 添加电量百分比变化时的处理程序
			const levelChangeHandler = () => console.log('levelchange'); 
			battery.onlevelchange = levelChangeHandler; 
			// 或
			battery.addEventListener('levelchange', levelChangeHandler); 
		});
		```

18. 硬件。
	- 浏览器检测硬件的能力相当有限。不过，navigator 对象还是通过一些属性提供了基本信息。
	- 处理器核心数。navigator.hardwareConcurrency 属性返回浏览器支持的逻辑处理器核心数量，包含表示核心数的一个整数值（如果核心数无法确定，这个值就是 1）。关键在于，这个值表示浏览器可以并行执行的最大工作线程数量，不一定是实际的 CPU 核心数。
	- 设备内存大小。navigator.deviceMemory 属性返回设备大致的系统内存大小，包含单位为 GB 的浮点数（舍入为最接近的 2 的幂：512MB 返回 0.5，4GB 返回 4）。
	- 最大触点数。navigator.maxTouchPoints 属性返回触摸屏支持的最大关联触点数量，包含一个整数值。

19. 总结
	- 客户端检测是 JavaScript 中争议最多的话题之一。因为不同浏览器之间存在差异，所以经常需要根据浏览器的能力来编写不同的代码。客户端检测有不少方式，但下面两种用得最多。
		- 能力检测，在使用之前先测试浏览器的特定能力。例如，脚本可以在调用某个函数之前先检查它是否存在。这种客户端检测方式可以让开发者不必考虑特定的浏览器或版本，而只需关注某些能力是否存在。能力检测不能精确地反映特定的浏览器或版本。
		- 用户代理检测，通过用户代理字符串确定浏览器。用户代理字符串包含关于浏览器的很多信息，通常包括浏览器、平台、操作系统和浏览器版本。用户代理字符串有一个相当长的发展史，很多浏览器都试图欺骗网站相信自己是别的浏览器。用户代理检测也比较麻烦，特别是涉及 Opera会在代理字符串中隐藏自己信息的时候。即使如此，用户代理字符串也可以用来确定浏览器使用的渲染引擎以及平台，包括移动设备和游戏机。
	- 在选择客户端检测方法时，首选是使用能力检测。特殊能力检测要放在次要位置，作为决定代码逻辑的参考。用户代理检测是最后一个选择，因为它过于依赖用户代理字符串。
	- 浏览器也提供了一些软件和硬件相关的信息。这些信息通过 screen 和 navigator 对象暴露出来。利用这些 API，可以获取关于操作系统、浏览器、硬件、设备位置、电池状态等方面的准确信息。
