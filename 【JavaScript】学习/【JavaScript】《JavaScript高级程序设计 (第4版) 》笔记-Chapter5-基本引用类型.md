###### 五、基本引用类型

1. 引用值（或者对象）是某个特定引用类型的实例。
    - 在 ECMAScript 中，引用类型是把数据和功能组织到一起的结构，经常被人错误地称作“类”。
    - 虽然从技术上讲 JavaScript 是一门面向对象语言，但 ECMAScript 缺少传统的面向对象编程语言所具备的某些基本结构，包括类和接口。
    - 引用类型有时候也被称为对象定义，因为它们描述了自己的对象应有的属性和方法。
    - 注意，引用类型虽然有点像类，但跟类并不是一个概念。
    - 对象被认为是某个特定引用类型的实例。新对象通过使用 new 操作符后跟一个构造函数（constructor）来创建。构造函数就是用来创建新对象的函数，比如下面这行代码：
        ```
        let now = new Date(); 
        ```
        - 这行代码创建了引用类型 Date 的一个新实例，并将它保存在变量 now 中。Date()在这里就是构造函数，它负责创建一个只有默认属性和方法的简单对象。ECMAScript 提供了很多像 Date 这样的原生引用类型，帮助开发者实现常见的任务。
    - 注意，函数也是一种引用类型。

2. Date
    - ECMAScript 的 Date 类型参考了 Java 早期版本中的 java.util.Date。为此，Date 类型将日期保存为自协调世界时（UTC，Universal Time Coordinated）时间 1970 年 1 月 1 日午夜（零时）至今所经过的毫秒数。使用这种存储格式，Date 类型可以精确表示 1970 年 1 月 1 日之前及之后 285 616 年的日期。
    - 要创建日期对象，就使用 new 操作符来调用 Date 构造函数：
        ```
        let now = new Date(); 
        ```
        - 在不给 Date 构造函数传参数的情况下，创建的对象将保存当前日期和时间。
    - 要基于其他日期和时间创建日期对象，必须传入其毫秒表示（UNIX 纪元 1970 年 1 月 1 日午夜之后的毫秒数）。ECMAScript为此提供了两个辅助方法：Date.parse()和 Date.UTC()。
    - Date.parse()方法接收一个表示日期的字符串参数，尝试将这个字符串转换为表示该日期的毫秒数。ECMA-262 第 5 版定义了 Date.parse() 应该支持的日期格式，填充了第 3 版遗留的空白。所有实现都必须支持下列日期格式：
        - “月/日/年”，如"5/23/2019"；
        - “月名 日, 年”，如"May 23, 2019"；
        - “周几 月名 日 年 时:分:秒 时区”，如"Tue May 23 2019 00:00:00 GMT-0700"；
        -  ISO 8601 扩展格式“YYYY-MM-DDTHH:mm:ss.sssZ”，如 2019-05-23T00:00:00（只适用于兼容 ES5 的实现）
    - 比如，要创建一个表示“2019 年 5 月 23 日”的日期对象，可以使用以下代码：
        ```
        let someDate = new Date(Date.parse("May 23, 2019")); 
        ```
    - 如果传给 Date.parse()的字符串并不表示日期，则该方法会返回 NaN。如果直接把表示日期的字符串传给 Date 构造函数，那么 Date 会在后台调用 Date.parse()。
        ```
        let someDate = new Date("May 23, 2019");
        ```
        - 下面这行代码跟前面那行代码是等价的，这两行代码得到的日期对象相同。
    - 注意，不同的浏览器对 Date 类型的实现有很多问题。比如，很多浏览器会选择用当前日期替代越界的日期，因此有些浏览器会将"January 32, 2019"解释为"February 1, 2019"。Opera 则会插入当前月的当前日，返回"January 当前日, 2019"。就是说，如果是在 9 月 21 日运行代码，会返回"January 21, 2019"。
    - Date.UTC()方法也返回日期的毫秒表示，但使用的是跟 Date.parse()不同的信息来生成这个值。传给 Date.UTC()的参数是年、零起点月数（1 月是 0，2 月是 1，以此类推）、日（1~31）、时（0~23）、分、秒和毫秒。这些参数中，只有前两个（年和月）是必需的。如果不提供日，那么默认为 1 日。其他参数的默认值都是 0。下面是使用 Date.UTC()的两个例子：
        ```
        // GMT 时间 2000 年 1 月 1 日零点
        let y2k = new Date(Date.UTC(2000, 0)); 
        
        // GMT 时间 2005 年 5 月 5 日下午 5 点 55 分 55 秒
        let allFives = new Date(Date.UTC(2005, 4, 5, 17, 55, 55));
        ```
    - 与 Date.parse()一样，Date.UTC()也会被 Date 构造函数隐式调用，但有一个区别：这种情况下创建的是本地日期，不是 GMT 日期。不过 Date 构造函数跟 Date.UTC()接收的参数是一样的。
        ```
        // 本地时间 2000 年 1 月 1 日零点
        let y2k = new Date(2000, 0);        // console.log(y2k); Sat Jan 01 2000 00:00:00 GMT+0800 (中国标准时间)
        
        // 本地时间 2005 年 5 月 5 日下午 5 点 55 分 55 秒
        let allFives = new Date(2005, 4, 5, 17, 55, 55);
        ```
        - 以上代码创建了与前面例子中相同的两个日期，但这次的两个日期是（由于系统设置决定的）本地时区的日期。
    - ECMAScript 还提供了 Date.now()方法，返回表示方法执行时日期和时间的毫秒数。
    - 读者备注：（来源GPT3.5）GMT，即格林尼治标准时间（Greenwich Mean Time），是以英国伦敦郊区的皇家格林尼治天文台为零度经线的地方时为标准的时间系统。GMT日期和时间是相对于格林尼治标准时间的值，它可能与您所在的时区不同。当需要计算不同时区之间的时间差时，GMT可以作为一个基准点。

3. Date 继承的方法
    - 与其他类型一样，Date 类型重写了 toLocaleString()、toString()和 valueOf()方法。但与其他类型不同，重写后这些方法的返回值不一样。Date 类型的 toLocaleString()方法返回与浏览器运行的本地环境一致的日期和时间。这通常意味着格式中包含针对时间的 AM（上午）或 PM（下午），但不包含时区信息（具体格式可能因浏览器而不同）。toString()方法通常返回带时区信息的日期和时间，而时间也是以 24 小时制（0~23）表示的。下面给出了 toLocaleString()和 toString()返回的 2019 年 2 月 1 日零点的示例（地区为"en-US"的 PST，即 Pacific Standard Time，太平洋标准时间）：
        ```
        toLocaleString() - 2/1/2019 12:00:00 AM 
        toString() - Thu Feb 1 2019 00:00:00 GMT-0800 (Pacific Standard Time)
        ```
        - 现代浏览器在这两个方法的输出上已经趋于一致。在比较老的浏览器上，每个方法返回的结果可能在每个浏览器上都是不同的。这些差异意味着 toLocaleString()和 toString()可能只对调试有用，不能用于显示。
    - Date 类型的 valueOf()方法根本就不返回字符串，这个方法被重写后返回的是日期的毫秒表示。因此，操作符（如小于号和大于号）可以直接使用它返回的值。
        ```
        let date1 = new Date(2019, 0, 1); // 2019 年 1 月 1 日
        let date2 = new Date(2019, 1, 1); // 2019 年 2 月 1 日
        console.log(date1 < date2); // true 
        console.log(date1 > date2); // false
        ```

4. 日期格式化方法
    - Date 类型有几个专门用于格式化日期的方法，它们都会返回字符串：
        - toDateString()显示日期中的周几、月、日、年（格式特定于实现）；
        - toTimeString()显示日期中的时、分、秒和时区（格式特定于实现）；
        - toLocaleDateString()显示日期中的周几、月、日、年（格式特定于实现和地区）；
        - toLocaleTimeString()显示日期中的时、分、秒（格式特定于实现和地区）；
        - toUTCString()显示完整的 UTC 日期（格式特定于实现）。
    - 这些方法的输出与 toLocaleString()和 toString()一样，会因浏览器而异。因此不能用于在用户界面上一致地显示日期。
    - 注意，还有一个方法叫 toGMTString()，这个方法跟 toUTCString()是一样的，目的是为了向后兼容。不过，规范建议新代码使用 toUTCString()。

5. 日期/时间组件方法。Date 类型剩下的方法（见下表）直接涉及取得或设置日期值的特定部分。注意表中“UTC 日期”，指的是没有时区偏移（将日期转换为 GMT）时的日期。
    
    | 方法 | 说明 |
    | --- | --- |
    | getTime() | 返回日期的毫秒表示；与 valueOf()相同 |
    | setTime(milliseconds) | 设置日期的毫秒表示，从而修改整个日期 |
    | getFullYear() | 返回 4 位数年（即 2019 而不是 19） |
    | getUTCFullYear() | 返回 UTC 日期的 4 位数年 |
    | setFullYear(year) | 设置日期的年（year 必须是 4 位数） |
    | setUTCFullYear(year) | 设置 UTC 日期的年（year 必须是 4 位数） |
    | getMonth() | 返回日期的月（0 表示 1 月，11 表示 12 月） |
    | getUTCMonth() | 返回 UTC 日期的月（0 表示 1 月，11 表示 12 月） |
    | setMonth(month) | 设置日期的月（month 为大于 0 的数值，大于 11 加年） |
    | setUTCMonth(month) | 设置 UTC 日期的月（month 为大于 0 的数值，大于 11 加年） |
    | getDate() | 返回日期中的日（1~31） |
    | getUTCDate() | 返回 UTC 日期中的日（1~31） |
    | setDate(date) | 设置日期中的日（如果 date 大于该月天数，则加月） |
    | setUTCDate(date) | 设置 UTC 日期中的日（如果 date 大于该月天数，则加月） |
    | getDay() | 返回日期中表示周几的数值（0 表示周日，6 表示周六） |
    | getUTCDay() | 返回 UTC 日期中表示周几的数值（0 表示周日，6 表示周六） |
    | getHours() | 返回日期中的时（0~23） |
    | getUTCHours() | 返回 UTC 日期中的时（0~23） |
    | setHours(hours) | 设置日期中的时（如果 hours 大于 23，则加日） |
    | setUTCHours(hours) | 设置 UTC 日期中的时（如果 hours 大于 23，则加日） |
    | getMinutes() | 返回日期中的分（0~59） |
    | getUTCMinutes() | 返回 UTC 日期中的分（0~59） |
    | setMinutes(minutes) | 设置日期中的分（如果 minutes 大于 59，则加时） |
    | setUTCMinutes(minutes) | 设置 UTC 日期中的分（如果 minutes 大于 59，则加时） |
    | getSeconds() | 返回日期中的秒（0~59） |
    | getUTCSeconds() | 返回 UTC 日期中的秒（0~59） |
    | setSeconds(seconds) | 设置日期中的秒（如果 seconds 大于 59，则加分） |
    | setUTCSeconds(seconds) | 设置 UTC 日期中的秒（如果 seconds 大于 59，则加分） |
    | getMilliseconds() | 返回日期中的毫秒 |
    | getUTCMilliseconds() | 返回 UTC 日期中的毫秒 |
    | setMilliseconds(milliseconds) | 设置日期中的毫秒 |
    | setUTCMilliseconds(milliseconds) | 设置 UTC 日期中的毫秒 |
    | getTimezoneOffset() | 返回以分钟计的 UTC 与本地时区的偏移量（如美国 EST 即“东部标准时间”返回 300，进入夏令时的地区可能有所差异） |

6. ECMAScript 通过 RegExp 类型支持正则表达式。
    - 正则表达式使用类似 Perl 的简洁语法来创建：
        ```
        let expression = /pattern/flags;
        ```
        - 这个正则表达式的 pattern（模式）可以是任何简单或复杂的正则表达式，包括字符类、限定符、分组、向前查找和反向引用。每个正则表达式可以带零个或多个 flags（标记），用于控制正则表达式的行为。
    - 下面给出了表示匹配模式的标记：
        - g：全局模式，表示查找字符串的全部内容，而不是找到第一个匹配的内容就结束。
        - i：不区分大小写，表示在查找匹配时忽略 pattern 和字符串的大小写。
        - m：多行模式，表示查找到一行文本末尾时会继续查找。
        - y：粘附模式，表示只查找从 lastIndex 开始及之后的字符串。
        - u：Unicode 模式，启用 Unicode 匹配。
        - s：dotAll 模式，表示元字符.匹配任何字符（包括\n 或\r）。
    - 使用不同模式和标记可以创建出各种正则表达式，比如：
        ```
        // 匹配字符串中的所有"at" 
        let pattern1 = /at/g; 
        
        // 匹配第一个"bat"或"cat"，忽略大小写
        let pattern2 = /[bc]at/i; 
        
        // 匹配所有以"at"结尾的三字符组合，忽略大小写
        let pattern3 = /.at/gi;
        ```
    - 与其他语言中的正则表达式类似，所有元字符在模式中也必须转义，包括：
        ```
        ( [ { \ ^ $ | ) ] } ? * + .
        ```
    - 元字符在正则表达式中都有一种或多种特殊功能，所以要匹配上面这些字符本身，就必须使用反斜杠来转义。下面是几个例子：
        ```
        // 匹配第一个"bat"或"cat"，忽略大小写
        let pattern1 = /[bc]at/i; 
        
        // 匹配第一个"[bc]at"，忽略大小写
        let pattern2 = /\[bc\]at/i; 
        
        // 匹配所有以"at"结尾的三字符组合，忽略大小写
        let pattern3 = /.at/gi; 
        
        // 匹配所有".at"，忽略大小写
        let pattern4 = /\.at/gi;
        ```
    - 正则表达式也可以使用 RegExp 构造函数来创建，它接收两个参数：模式字符串和（可选的）标记字符串。任何使用字面量定义的正则表达式也可以通过构造函数来创建，比如：
        ```
        // 匹配第一个"bat"或"cat"，忽略大小写
        let pattern1 = /[bc]at/i; 
        
        // 跟 pattern1 一样，只不过是用构造函数创建的
        let pattern2 = new RegExp("[bc]at", "i");
        ```
        - 这里的 pattern1 和 pattern2 是等效的正则表达式。
    - 注意，RegExp 构造函数的两个参数都是字符串。因为 RegExp 的模式参数是字符串，所以在某些情况下需要二次转义。**所有元字符都必须二次转义，包括转义字符序列，如\n（\转义后的字符串是\\\，在正则表达式字符串中则要写成\\\\\\\）。**
    - 下表展示了几个正则表达式的字面量形式，以及使用 RegExp 构造函数创建时对应的模式字符串。

        | 字面量模式 | 对应的字符串 |
        | --- | --- |
        | /\\[bc\\]at/          | "\\\\[bc\\\\]at" | 
        | /\\.at/               | "\\\\.at" | 
        | /name\\/age/          | "name\\\\/age" | 
        | /\d.\d{1,2}/          | "\\\\d.\\\\d{1,2}" | 
        | /\w\\\\hello\\\\123/  | "\\w\\\\\\\\hello\\\\\\\\123" | 

    - 此外，使用 RegExp 也可以基于已有的正则表达式实例，并可选择性地修改它们的标记：
        ```
        const re1 = /cat/g; 
        console.log(re1); // "/cat/g" 
        
        const re2 = new RegExp(re1); 
        console.log(re2); // "/cat/g" 
        
        const re3 = new RegExp(re1, "i"); 
        console.log(re3); // "/cat/i"
        ```

7. RegExp 实例属性
    - 每个 RegExp 实例都有下列属性，提供有关模式的各方面信息：
        - global：布尔值，表示是否设置了 g 标记。
        - ignoreCase：布尔值，表示是否设置了 i 标记。
        - unicode：布尔值，表示是否设置了 u 标记。
        - sticky：布尔值，表示是否设置了 y 标记。
        - lastIndex：整数，表示在源字符串中下一次搜索的开始位置，始终从 0 开始。
        - multiline：布尔值，表示是否设置了 m 标记。
        - dotAll：布尔值，表示是否设置了 s 标记。
        - source：正则表达式的字面量字符串（不是传给构造函数的模式字符串），没有开头和结尾的斜杠。
        - flags：正则表达式的标记字符串。始终以字面量而非传入构造函数的字符串模式形式返回（没有前后斜杠）。
    - 通过这些属性可以全面了解正则表达式的信息，不过实际开发中用得并不多，因为模式声明中包含这些信息。下面是一个例子：
        ```
        let pattern1 = /\[bc\]at/i; 
        console.log(pattern1.global);       // false 
        console.log(pattern1.ignoreCase);   // true 
        console.log(pattern1.multiline);    // false 
        console.log(pattern1.lastIndex);    // 0 
        console.log(pattern1.source);       // "\[bc\]at" 
        console.log(pattern1.flags);        // "i" 
        
        let pattern2 = new RegExp("\\[bc\\]at", "i"); 
        console.log(pattern2.global);       // false 
        console.log(pattern2.ignoreCase);   // true 
        console.log(pattern2.multiline);    // false 
        console.log(pattern2.lastIndex);    // 0 
        console.log(pattern2.source);       // "\[bc\]at" 
        console.log(pattern2.flags);        // "i"
        ```
        - 注意，虽然第一个模式是通过字面量创建的，第二个模式是通过 RegExp 构造函数创建的，但两个模式的 source 和 flags 属性是相同的。source 和 flags 属性返回的是规范化之后可以在字面量中使用的形式。

8. RegExp 实例方法
    - RegExp 实例的主要方法是 exec()，主要用于配合捕获组使用。这个方法只接收一个参数，即要应用模式的字符串。如果找到了匹配项，则返回包含第一个匹配信息的数组；如果没找到匹配项，则返回null。返回的数组虽然是 Array 的实例，但包含两个额外的属性：index 和 input。index 是字符串中匹配模式的起始位置，input 是要查找的字符串。这个数组的第一个元素是匹配整个模式的字符串，其他元素是与表达式中的捕获组匹配的字符串。如果模式中没有捕获组，则数组只包含一个元素。来看下面的例子：
        ```
        let text = "mom and dad and baby"; 
        let pattern = /mom( and dad( and baby)?)?/gi; 
        
        let matches = pattern.exec(text); 
        console.log(matches.index);     // 0 
        console.log(matches.input);     // "mom and dad and baby" 
        console.log(matches[0]);        // "mom and dad and baby" 
        console.log(matches[1]);        // " and dad and baby" 
        console.log(matches[2]);        // " and baby"
        ```
        - 在这个例子中，模式包含两个捕获组：最内部的匹配项" and baby"，以及外部的匹配项" and dad"或" and dad and baby"。
        - 调用 exec()后找到了一个匹配项。因为整个字符串匹配模式，所以 matchs 数组的 index 属性就是 0。数组的第一个元素是匹配的整个字符串，第二个元素是匹配第一个捕获组的字符串，第三个元素是匹配第二个捕获组的字符串。
    - 如果模式设置了全局标记，则每次调用 exec()方法会返回一个匹配的信息。如果没有设置全局标记，则无论对同一个字符串调用多少次 exec()，也只会返回第一个匹配的信息。
        ```
        // 例子1：
        let text = "cat, bat, sat, fat"; 
        let pattern = /.at/; 
        
        let matches = pattern.exec(text); 
        console.log(matches.index);         // 0 
        console.log(matches[0]);            // cat 
        console.log(pattern.lastIndex);     // 0 
        
        matches = pattern.exec(text); 
        console.log(matches.index);         // 0 
        console.log(matches[0]);            // cat 
        console.log(pattern.lastIndex);     // 0
        
        // 例子2：
        let text = "cat, bat, sat, fat"; 
        let pattern = /.at/g; 
        let matches = pattern.exec(text); 
        console.log(matches.index);         // 0 
        console.log(matches[0]);            // cat 
        console.log(pattern.lastIndex);     // 3 
        
        matches = pattern.exec(text); 
        console.log(matches.index);         // 5 
        console.log(matches[0]);            // bat 
        console.log(pattern.lastIndex);     // 8 
        
        matches = pattern.exec(text); 
        console.log(matches.index);         // 10 
        console.log(matches[0]);            // sat 
        console.log(pattern.lastIndex);     // 13
        ```
        - 例子1中的模式没有设置全局标记，因此调用 exec()只返回第一个匹配项（"cat"）。lastIndex在非全局模式下始终不变。
        - 如果在这个模式上设置了 g 标记，则每次调用 exec()都会在字符串中向前搜索下一个匹配项。
        - 例子2设置了全局标记，因此每次调用 exec()都会返回字符串中的下一个匹配项，直到搜索到字符串末尾。注意模式的 lastIndex 属性每次都会变化。在全局匹配模式下，每次调用 exec()都会更新 lastIndex 值，以反映上次匹配的最后一个字符的索引。
    - 如果模式设置了粘附标记 y，则每次调用 exec()就只会在 lastIndex 的位置上寻找匹配项。粘附标记覆盖全局标记。
        ```
        let text = "cat, bat, sat, fat"; 
        let pattern = /.at/y; 
        
        let matches = pattern.exec(text); 
        console.log(matches.index);         // 0 
        console.log(matches[0]);            // cat 
        console.log(pattern.lastIndex);     // 3 
        
        // 以索引 3 对应的字符开头找不到匹配项，因此 exec()返回 null 
        // exec()没找到匹配项，于是将 lastIndex 设置为 0 
        matches = pattern.exec(text); 
        console.log(matches);               // null 
        console.log(pattern.lastIndex);     // 0 
        
        // 向前设置 lastIndex 可以让粘附的模式通过 exec()找到下一个匹配项：
        pattern.lastIndex = 5; 
        matches = pattern.exec(text); 
        console.log(matches.index);         // 5 
        console.log(matches[0]);            // bat 
        console.log(pattern.lastIndex);     // 8
        ```
    - 正则表达式的另一个方法是 test()，接收一个字符串参数。如果输入的文本与模式匹配，则参数返回 true，否则返回 false。这个方法适用于只想测试模式是否匹配，而不需要实际匹配内容的情况。test()经常用在 if 语句中：
        ```
        let text = "000-00-0000"; 
        let pattern = /\d{3}-\d{2}-\d{4}/; 
        if (pattern.test(text)) { 
            console.log("The pattern was matched."); 
        }
        ```
    - 无论正则表达式是怎么创建的，继承的方法 toLocaleString()和 toString()都返回正则表达式的字面量表示。比如：
        ```
        let pattern = new RegExp("\\[bc\\]at", "gi"); 
        console.log(pattern.toString());        // /\[bc\]at/gi 
        console.log(pattern.toLocaleString());  // /\[bc\]at/gi
        ```
    - 注意，正则表达式的 valueOf()方法返回正则表达式本身。

9. RegExp 构造函数属性
    - RegExp 构造函数本身也有几个属性。（在其他语言中，这种属性被称为静态属性。）这些属性适用于作用域中的所有正则表达式，**而且会根据最后执行的正则表达式操作而变化。**这些属性还有一个特点，就是可以通过两种不同的方式访问它们。换句话说，每个属性都有一个全名和一个简写。
    - 下表列出了 RegExp 构造函数的属性：
    
        | 全名 | 简写 | 说明 |
        | --- | --- | --- |
        | input | $_ | 最后搜索的字符串（非标准特性）
        | lastMatch | $& | 最后匹配的文本
        | lastParen | $+ | 最后匹配的捕获组（非标准特性）
        | leftContext | $` | input 字符串中出现在 lastMatch 前面的文本
        | rightContext | $' | input 字符串中出现在 lastMatch 后面的文本
    
    - 通过这些属性可以提取出与 exec()和 test()执行的操作相关的信息。来看下面的例子：
        ```
        let text = "this has been a short summer"; 
        let pattern = /(.)hort/g; 
        if (pattern.test(text)) { 
            console.log(RegExp.input);          // this has been a short summer 
            console.log(RegExp.leftContext);    // this has been a 
            console.log(RegExp.rightContext);   // summer 
            console.log(RegExp.lastMatch);      // short 
            console.log(RegExp.lastParen);      // s 
        }
        
        let text = "this has been a short summer"; 
        let pattern = /(.)hort/g; 
        /* 
         * 注意：Opera 不支持简写属性名
         * IE 不支持多行匹配
         */ 
        if (pattern.test(text)) { 
            console.log(RegExp.$_);             // this has been a short summer 
            console.log(RegExp["$`"]);          // this has been a 
            console.log(RegExp["$'"]);          // summer 
            console.log(RegExp["$&"]);          // short 
            console.log(RegExp["$+"]);          // s 
        }
        ```
        - 以上代码创建了一个模式，用于搜索任何后跟"hort"的字符，并把第一个字符放在了捕获组中。
        - 这些属性名也可以替换成简写形式，只不过要使用中括号语法来访问，因为大多数简写形式都不是合法的 ECMAScript 标识符。
    - RegExp 还有其他几个构造函数属性，可以存储最多 9 个捕获组的匹配项。这些属性通过 RegExp.$1~RegExp.$9 来访问，分别包含第 1~9 个捕获组的匹配项。在调用 exec()或 test()时，这些属性就会被填充，然后就可以像下面这样使用它们：
        ```
        let text = "this has been a short summer"; 
        let pattern = /(..)or(.)/g; 
        if (pattern.test(text)) { 
            console.log(RegExp.$1); // sh 
            console.log(RegExp.$2); // t 
        }
        ```
        - 在这个例子中，模式包含两个捕获组。调用 test()搜索字符串之后，因为找到了匹配项所以返回true，而且可以打印出通过 RegExp 构造函数的$1 和$2 属性取得的两个捕获组匹配的内容。
    - **注意，RegExp 构造函数的所有属性都没有任何 Web 标准出处，因此不要在生产环境中使用它们。**

10. 模式局限。下列特性目前还没有得到 ECMAScript 的支持（想要了解更多信息，可以参考 Regular-Expressions.info
网站）：
    - \A 和\Z 锚（分别匹配字符串的开始和末尾）
    - 联合及交叉类
    - 原子组
    - x（忽略空格）匹配模式
    - 条件式匹配
    - 正则表达式注释

11. 原始值包装类型
    - 为了方便操作原始值，ECMAScript 提供了 3 种特殊的引用类型：Boolean、Number 和 String。这些类型具有本章介绍的其他引用类型一样的特点，但也具有与各自原始类型对应的特殊行为。每当用到某个原始值的方法或属性时，后台都会创建一个相应原始包装类型的对象，从而暴露出操作原始值的各种方法。来看下面的例子：
        ```
        let s1 = "some text"; 
        let s2 = s1.substring(2);
        ```
        - 原始值本身不是对象，因此逻辑上不应该有方法。而实际上这个例子又确实按照预期运行了。这是因为后台进行了很多处理，从而实现了上述操作。具体来说，当第二行访问 s1 时，是以读模式访问的，也就是要从内存中读取变量保存的值。在以读模式访问字符串值的任何时候，后台都会执行以下 3 步：
            - (1) 创建一个 String 类型的实例；
            - (2) 调用实例上的特定方法；
            - (3) 销毁实例。
        - 可以把这 3 步想象成执行了如下 3 行 ECMAScript 代码：
            ```
            let s1 = new String("some text"); 
            let s2 = s1.substring(2); 
            s1 = null;
            ```
            - 这种行为可以让原始值拥有对象的行为。对布尔值和数值而言，以上 3 步也会在后台发生，只不过使用的是 Boolean 和 Number 包装类型而已。
    - **引用类型与原始值包装类型的主要区别在于对象的生命周期。在通过 new 实例化引用类型后，得到的实例会在离开作用域时被销毁，而自动创建的原始值包装对象则只存在于访问它的那行代码执行期间。这意味着不能在运行时给原始值添加属性和方法。**比如下面的例子：
        ```
        let s1 = "some text"; 
        s1.color = "red"; 
        console.log(s1.color); // undefined
        ```
        - 第二行代码运行时会临时创建一个 String 对象，而当第三行代码执行时，这个对象已经被销毁了。实际上，第三行代码在这里创建了自己的 String 对象，但这个对象没有 color 属性。
    - 可以显式地使用 Boolean、Number 和 String 构造函数创建原始值包装对象。不过应该在确实必要时再这么做，否则容易让开发者疑惑，分不清它们到底是原始值还是引用值。在原始值包装类型的实例上调用 typeof 会返回"object"，所有原始值包装对象都会转换为布尔值 true。
    - 另外，Object 构造函数作为一个工厂方法，能够根据传入值的类型返回相应原始值包装类型的实例。比如：
        ```
        let obj = new Object("some text"); 
        console.log(obj instanceof String); // true
        ```
        - 如果传给 Object 的是字符串，则会创建一个 String 的实例。如果是数值，则会创建 Number 的实例。布尔值则会得到 Boolean 的实例。
    - 注意，使用 new 调用原始值包装类型的构造函数，与调用同名的转型函数并不一样。例如：
        ```
        let value = "25"; 
        let number = Number(value);     // 转型函数
        console.log(typeof number);     // "number" 
        let obj = new Number(value);    // 构造函数
        console.log(typeof obj);        // "object"
        ```
        - 在这个例子中，变量 number 中保存的是一个值为 25 的原始数值，而变量 obj 中保存的是一个 Number 的实例。
    - 虽然不推荐显式创建原始值包装类型的实例，但它们对于操作原始值的功能是很重要的。每个原始值包装类型都有相应的一套方法来方便数据操作。

12. Boolean 是对应布尔值的引用类型。
    - 要创建一个 Boolean 对象，就使用 Boolean 构造函数并传入 true 或 false。
    - Boolean 的实例会重写 valueOf()方法，返回一个原始值 true 或 false。toString()方法被调用时也会被覆盖，返回字符串"true"或"false"。不过，Boolean 对象在 ECMAScript 中用得很少。不仅如此，它们还容易引起误会，尤其是在布尔表达式中使用 Boolean 对象时，比如：
        ```
        let falseObject = new Boolean(false); 
        let result = falseObject && true; 
        console.log(result);    // true 
        
        let falseValue = false; 
        result = falseValue && true; 
        console.log(result);    // false
        ```
        - **所有对象在布尔表达式中都会自动转换为 true，因此 falseObject 在这个表达式里实际上表示一个 true 值。**那么 true && true 当然是 true。
    - 除此之外，原始值和引用值（Boolean 对象）还有几个区别。首先，typeof 操作符对原始值返回"boolean"，但对引用值返回"object"。同样，Boolean 对象是 Boolean 类型的实例，在使用 instaceof 操作符时返回 true，但对原始值则返回 false，如下所示：
        ```
        console.log(typeof falseObject);                // object 
        console.log(typeof falseValue);                 // boolean 
        console.log(falseObject instanceof Boolean);    // true 
        console.log(falseValue instanceof Boolean);     // false 
        ```
    - 理解原始布尔值和 Boolean 对象之间的区别非常重要，**强烈建议永远不要使用后者。**

13. Number 是对应数值的引用类型。
    - 要创建一个 Number 对象，就使用 Number 构造函数并传入一个数值。
    - 与 Boolean 类型一样，Number 类型重写了 valueOf()、toLocaleString()和 toString()方法。valueOf()方法返回 Number 对象表示的原始数值，另外两个方法返回数值字符串。toString()方法可选地接收一个表示基数的参数，并返回相应基数形式的数值字符串，如下所示：
        ```
        let num = 10; 
        console.log(num.toString());    // "10" 
        console.log(num.toString(2));   // "1010" 
        console.log(num.toString(8));   // "12" 
        console.log(num.toString(10));  // "10" 
        console.log(num.toString(16));  // "a"
        ```
    - 除了继承的方法，Number 类型还提供了几个用于将数值格式化为字符串的方法。
    - toFixed()方法返回包含指定小数点位数的数值字符串，如：
        ```
        let num = 10; 
        console.log(num.toFixed(2)); // "10.00"
        
        let num = 10.005; 
        console.log(num.toFixed(2)); // "10.01"
        ```
        - 如果数值本身的小数位超过了参数指定的位数，则四舍五入到最接近的小数位。
    - toFixed()自动舍入的特点可以用于处理货币。不过要注意的是，多个浮点数值的数学计算不一定得到精确的结果。比如，0.1 + 0.2 = 0.30000000000000004。
    - toFixed()方法可以表示有 0~20 个小数位的数值。某些浏览器可能支持更大的范围，但这是通常被支持的范围。
    - 另一个用于格式化数值的方法是 toExponential()，返回以科学记数法（也称为指数记数法）表示的数值字符串。与 toFixed()一样，toExponential()也接收一个参数，表示结果中小数的位数。来看下面的例子：
        ```
        let num = 10; 
        console.log(num.toExponential(1)); // "1.0e+1"
        ```
    - 如果想得到数值最适当的形式，那么可以使用 toPrecision()。toPrecision()方法会根据情况返回最合理的输出结果，可能是固定长度，也可能是科学记数法形式。这个方法接收一个参数，表示结果中数字的总位数（不包含指数）。来看几个例子：
        ```
        let num = 99; 
        console.log(num.toPrecision(1)); // "1e+2" 
        console.log(num.toPrecision(2)); // "99" 
        console.log(num.toPrecision(3)); // "99.0"
        ```
        - 在这个例子中，要用 1 位数字表示数值 99，得到"1e+2"，也就是 100。因为 99 不能只用 1 位数字来精确表示，所以这个方法就将它舍入为 100，这样就可以只用 1 位数字（及其科学记数法形式）来表示了。
        - 本质上，toPrecision()方法会根据数值和精度来决定调用 toFixed()还是 toExponential()。为了以正确的小数位精确表示数值，这 3 个方法都会向上或向下舍入。
    - toPrecision()方法可以表示带 1~21 个小数位的数值。某些浏览器可能支持更大的范围，但这是通常被支持的范围。
    - 并不建议直接实例化 Number 对象。在处理原始数值和引用数值时，typeof 和 instacnceof 操作符会返回不同的结果，如下所示：
        ```
        let numberObject = new Number(10); 
        let numberValue = 10; 
        console.log(typeof numberObject);               // "object" 
        console.log(typeof numberValue);                // "number" 
        console.log(numberObject instanceof Number);    // true 
        console.log(numberValue instanceof Number);     // false
        ```
        - 原始数值在调用 typeof 时始终返回"number"，而 Number 对象则返回"object"。类似地，Number对象是 Number 类型的实例，而原始数值不是。
    - ES6 新增了 Number.isInteger()方法，用于辨别一个数值是否保存为整数。有时候，小数位的 0 可能会让人误以为数值是一个浮点值：
        ```
        console.log(Number.isInteger(1));       // true 
        console.log(Number.isInteger(1.00));    // true 
        console.log(Number.isInteger(1.01));    // false
        ```
    - IEEE 754 数值格式有一个特殊的数值范围，在这个范围内二进制值可以表示一个整数值。这个数值范围从 Number.MIN_SAFE_INTEGER（-2^53 + 1）到 Number.MAX_SAFE_INTEGER（2^53 - 1）。对超出这个范围的数值，即使尝试保存为整数，IEEE 754 编码格式也意味着二进制值可能会表示一个完全不同的数值。为了鉴别整数是否在这个范围内，可以使用 Number.isSafeInteger()方法：
        ```
        console.log(Number.isSafeInteger(-1 * (2 ** 53)));      // false 
        console.log(Number.isSafeInteger(-1 * (2 ** 53) + 1));  // true 
        
        console.log(Number.isSafeInteger(2 ** 53));             // false 
        console.log(Number.isSafeInteger((2 ** 53) - 1));       // true
        ```

14. String 是对应字符串的引用类型。
    - 要创建一个 String 对象，使用 String 构造函数并传入一个数值。
    - String 对象的方法可以在所有字符串原始值上调用。3个继承的方法 valueOf()、toLocaleString()和 toString()都返回对象的原始字符串值。
    - 每个 String 对象都有一个 length 属性，表示字符串中字符的数量。
        ```
        let stringValue = "hello world"; 
        console.log(stringValue.length); // "11"
        ```
        - 注意，即使字符串中包含双字节字符（而不是单字节的 ASCII 字符），也仍然会按单字符来计数。

15. String 的 JavaScript 字符
    - JavaScript 字符串由 16 位码元（code unit）组成。对多数字符来说，每 16 位码元对应一个字符。换句话说，字符串的 length 属性表示字符串包含多少 16 位码元。
    - 此外，charAt()方法返回给定索引位置的字符，由传给方法的整数参数指定。具体来说，这个方法查找指定索引位置的 16 位码元，并返回该码元对应的字符：
        ```
        let message = "abcde"; 
        console.log(message.charAt(2)); // "c"
        ```
    - JavaScript 字符串使用了两种 Unicode 编码混合的策略：UCS-2 和 UTF-16。对于可以采用 16 位编码的字符（U+0000~U+FFFF），这两种编码实际上是一样的。
    - 使用 charCodeAt()方法可以查看指定码元的字符编码。这个方法返回指定索引位置的码元值，索引以整数指定。比如：
        ```
        let message = "abcde"; 
        
        // Unicode "Latin small letter C"的编码是 U+0063 
        console.log(message.charCodeAt(2)); // 99 
        
        // 十进制 99 等于十六进制 63 
        console.log(99 === 0x63);           // true
        ```
    - fromCharCode()方法用于根据给定的 UTF-16 码元创建字符串中的字符。这个方法可以接受任意多个数值，并返回将所有数值对应的字符拼接起来的字符串：
        ```
        // Unicode "Latin small letter A"的编码是 U+0061 
        // Unicode "Latin small letter B"的编码是 U+0062 
        // Unicode "Latin small letter C"的编码是 U+0063 
        // Unicode "Latin small letter D"的编码是 U+0064 
        // Unicode "Latin small letter E"的编码是 U+0065 
        console.log(String.fromCharCode(0x61, 0x62, 0x63, 0x64, 0x65)); // "abcde" 
        // 0x0061 === 97 
        // 0x0062 === 98 
        // 0x0063 === 99 
        // 0x0064 === 100 
        // 0x0065 === 101 
        console.log(String.fromCharCode(97, 98, 99, 100, 101));         // "abcde"
        ```
    - 对于 U+0000~U+FFFF 范围内的字符，length、charAt()、charCodeAt()和 fromCharCode()返回的结果都跟预期是一样的。这是因为在这个范围内，每个字符都是用 16 位表示的，而这几个方法也都基于 16 位码元完成操作。只要字符编码大小与码元大小一一对应，这些方法就能如期工作。这个对应关系在扩展到 Unicode 增补字符平面时就不成立了。问题很简单，即 16 位只能唯一表示65 536 个字符。这对于大多数语言字符集是足够了，在 Unicode 中称为基本多语言平面（BMP）。为了表示更多的字符，Unicode 采用了一个策略，即每个字符使用另外 16 位去选择一个增补平面。这种每个字符使用两个 16 位码元的策略称为代理对。
    - 在涉及增补平面的字符时，前面讨论的字符串方法就会出问题。比如，下面的例子中使用了一个笑脸表情符号，也就是一个使用代理对编码的字符：
        ```
        // "smiling face with smiling eyes" 表情符号的编码是 U+1F60A 
        // 0x1F60A === 128522 
        let message = "ab☺de"; 
        console.log(message.length);                                        // 6 
        console.log(message.charAt(1));                                     // b
        console.log(message.charAt(2));                                     // <?> 
        console.log(message.charAt(3));                                     // <?> 
        console.log(message.charAt(4));                                     // d 
        console.log(message.charCodeAt(1));                                 // 98 
        console.log(message.charCodeAt(2));                                 // 55357 
        console.log(message.charCodeAt(3));                                 // 56842 
        console.log(message.charCodeAt(4));                                 // 100 
        console.log(String.fromCodePoint(0x1F60A));                         // ☺
        console.log(String.fromCharCode(97, 98, 55357, 56842, 100, 101));   // ab☺de
        ```
        - 这些方法仍然将 16 位码元当作一个字符，事实上索引 2 和索引 3 对应的码元应该被看成一个代理对，只对应一个字符。
        - fromCharCode()方法仍然返回正确的结果，因为它实际上是基于提供的二进制表示直接组合成字符串。浏览器可以正确解析代理对（由两个码元构成），并正确地将其识别为一个 Unicode 笑脸字符。
    - **为正确解析既包含单码元字符又包含代理对字符的字符串，可以使用 codePointAt()来代替charCodeAt()。**跟使用 charCodeAt()时类似，codePointAt()接收 16 位码元的索引并返回该索引位置上的码点（code point）。码点是 Unicode 中一个字符的完整标识。比如，"c"的码点是 0x0063，而"☺"的码点是 0x1F60A。**码点可能是 16 位，也可能是 32 位，而 codePointAt()方法可以从指定码元位置识别完整的码点。**
        ```
        let message = "ab☺de"; 
        console.log(message.codePointAt(1)); // 98 
        console.log(message.codePointAt(2)); // 128522 
        console.log(message.codePointAt(3)); // 56842 
        console.log(message.codePointAt(4)); // 100
        ```
    - 注意，如果传入的码元索引并非代理对的开头，就会返回错误的码点。这种错误只有检测单个字符的时候才会出现，可以通过从左到右按正确的码元数遍历字符串来规避。迭代字符串可以智能地识别代理对的码点：
        ```
        console.log([..."ab☺de"]); // ["a", "b", "☺", "d", "e"]
        ```
    - 与 charCodeAt()有对应的 codePointAt()一样，fromCharCode()也有一个对应的 fromCodePoint()。这个方法接收任意数量的码点，返回对应字符拼接起来的字符串：
        ```
        console.log(String.fromCharCode(97, 98, 55357, 56842, 100, 101)); // ab☺de 
        console.log(String.fromCodePoint(97, 98, 128522, 100, 101)); // ab☺de
        ```

16. String 的 normalize()方法
    - 某些 Unicode 字符可以有多种编码方式。有的字符既可以通过一个 BMP 字符表示，也可以通过一个代理对表示。比如：
        ```
        // U+00C5：上面带圆圈的大写拉丁字母 A 
        console.log(String.fromCharCode(0x00C5)); // Å 
        // U+212B：长度单位“埃”
        console.log(String.fromCharCode(0x212B)); // Å 
        // U+004：大写拉丁字母 A 
        // U+030A：上面加个圆圈
        console.log(String.fromCharCode(0x0041, 0x030A)); // Å
        
        let a1 = String.fromCharCode(0x00C5), 
            a2 = String.fromCharCode(0x212B), 
            a3 = String.fromCharCode(0x0041, 0x030A); 
        console.log(a1, a2, a3);    // Å, Å, Å 
        console.log(a1 === a2);     // false 
        console.log(a1 === a3);     // false 
        console.log(a2 === a3);     // false
        ```
        - 比较操作符不在乎字符看起来是什么样的，因此这 3 个字符互不相等。
    - 为解决这个问题，Unicode提供了 4种规范化形式，可以将类似上面的字符规范化为一致的格式，无论底层字符的代码是什么。这 4种规范化形式是：NFD（Normalization Form D）、NFC（Normalization Form C）、NFKD（Normalization Form KD）和 NFKC（Normalization Form KC）。可以使用 normalize()方法对字符串应用上述规范化形式，使用时需要传入表示哪种形式的字符串："NFD"、"NFC"、"NFKD"或"NFKC"。
    - 通过比较字符串与其调用 normalize()的返回值，就可以知道该字符串是否已经规范化了。
        ```
        let a1 = String.fromCharCode(0x00C5), 
            a2 = String.fromCharCode(0x212B), 
            a3 = String.fromCharCode(0x0041, 0x030A); 
        
        // U+00C5 是对 0+212B 进行 NFC/NFKC 规范化之后的结果
        console.log(a1 === a1.normalize("NFD"));    // false 
        console.log(a1 === a1.normalize("NFC"));    // true 
        console.log(a1 === a1.normalize("NFKD"));   // false 
        console.log(a1 === a1.normalize("NFKC"));   // true 
        
        // U+212B 是未规范化的
        console.log(a2 === a2.normalize("NFD"));    // false 
        console.log(a2 === a2.normalize("NFC"));    // false 
        console.log(a2 === a2.normalize("NFKD"));   // false 
        console.log(a2 === a2.normalize("NFKC"));   // false 
        
        // U+0041/U+030A 是对 0+212B 进行 NFD/NFKD 规范化之后的结果
        console.log(a3 === a3.normalize("NFD"));    // true 
        console.log(a3 === a3.normalize("NFC"));    // false 
        console.log(a3 === a3.normalize("NFKD"));   // true 
        console.log(a3 === a3.normalize("NFKC"));   // false
        ```
    - 选择同一种规范化形式可以让比较操作符返回正确的结果：
        ```
        let a1 = String.fromCharCode(0x00C5), 
            a2 = String.fromCharCode(0x212B), 
            a3 = String.fromCharCode(0x0041, 0x030A); 
        console.log(a1.normalize("NFD") === a2.normalize("NFD")); // true 
        console.log(a2.normalize("NFKC") === a3.normalize("NFKC")); // true 
        console.log(a1.normalize("NFC") === a3.normalize("NFC")); // true
        ```

17. 字符串操作方法
    - concat()，用于将一个或多个字符串拼接成一个新字符串。
        ```
        let stringValue = "hello "; 
        let result = stringValue.concat("world"); 
        console.log(result); // "hello world" 
        console.log(stringValue); // "hello"
        
        let stringValue = "hello "; 
        let result = stringValue.concat("world", "!"); 
        console.log(result); // "hello world!" 
        console.log(stringValue); // "hello"
        ``` 
        - concat()方法可以接收任意多个参数，因此可以一次性拼接多个字符串。
    - 虽然 concat()方法可以拼接字符串，但更常用的方式是使用加号操作符（+）。而且多数情况下，对于拼接多个字符串来说，使用加号更方便。
    - ECMAScript 提供了 3 个从字符串中提取子字符串的方法：slice()、substr()和 substring()。这3个方法都返回调用它们的字符串的一个子字符串，而且都接收一或两个参数。第一个参数表示子字符串开始的位置，第二个参数表示子字符串结束的位置。对 slice()和 substring()而言，第二个参数是提取结束的位置（即该位置之前的字符会被提取出来）。对 substr()而言，第二个参数表示返回的子字符串数量。任何情况下，省略第二个参数都意味着提取到字符串末尾。**与 concat()方法一样，slice()、substr()和 substring()也不会修改调用它们的字符串，而只会返回提取到的原始新字符串值。**
        ```
        let stringValue = "hello world"; 
        console.log(stringValue.slice(3));          // "lo world" 
        console.log(stringValue.substring(3));      // "lo world" 
        console.log(stringValue.substr(3));         // "lo world" 
        console.log(stringValue.slice(3, 7));       // "lo w" 
        console.log(stringValue.substring(3,7));    // "lo w" 
        console.log(stringValue.substr(3, 7));      // "lo worl"
        ```
    - 当某个参数是负值时，这 3 个方法的行为又有不同。比如，slice()方法将所有负值参数都当成字符串长度加上负参数值。而 substr()方法将第一个负参数值当成字符串长度加上该值，将第二个负参数值转换为 0。substring()方法会将所有负参数值都转换为 0。
        ```
        let stringValue = "hello world"; 
        console.log(stringValue.slice(-3));         // "rld" 
        console.log(stringValue.substring(-3));     // "hello world" 
        console.log(stringValue.substr(-3));        // "rld" 
        console.log(stringValue.slice(3, -4));      // "lo w" 
        console.log(stringValue.substring(3, -4));  // "hel" 
        console.log(stringValue.substr(3, -4));     // "" (empty string)
        ```
        - 在第二个参数是负值时，slice()方法将第二个参数转换为 7，实际上相当于调用 slice(3, 7)，因此返回"lo w"。
        - 在第二个参数是负值时，substring()方法会将第二个参数转换为 0，**相当于调用substring(3, 0)，等价于 substring(0, 3)，这是因为这个方法会将较小的参数作为起点，将较大的参数作为终点。**
        - 在第二个参数是负值时，对 substr()来说，第二个参数会被转换为 0，意味着返回的字符串包含零个字符，因而会返回一个空字符串。

18. 字符串位置方法。有两个方法用于在字符串中定位子字符串：indexOf()和 lastIndexOf()。这两个方法从字符串中搜索传入的字符串，并返回位置（如果没找到，则返回-1）。两者的区别在于，indexOf()方法从字符串开头开始查找子字符串，而 lastIndexOf()方法从字符串末尾开始查找子字符串。
    ```
    let stringValue = "hello world"; 
    console.log(stringValue.indexOf("o"));      // 4 
    console.log(stringValue.lastIndexOf("o"));  // 7
    
    let stringValue = "hello world"; 
    console.log(stringValue.indexOf("o", 6));       // 7 
    console.log(stringValue.lastIndexOf("o", 6));   // 4
    
    let stringValue = "Lorem ipsum dolor sit amet, consectetur adipisicing elit"; 
    let positions = new Array(); 
    let pos = stringValue.indexOf("e"); 
    while(pos > -1) { 
        positions.push(pos); 
        pos = stringValue.indexOf("e", pos + 1); 
    } 
    console.log(positions); // [3,24,32,35,52]
    ```
    - 如果字符串中只有一个"o"，则 indexOf()和 lastIndexOf()返回同一个位置。
    - 这两个方法都可以接收可选的第二个参数，表示开始搜索的位置。这意味着，indexOf()会从这个参数指定的位置开始向字符串末尾搜索，忽略该位置之前的字符；lastIndexOf()则会从这个参数指定的位置开始向字符串开头搜索，忽略该位置之后直到字符串末尾的字符。
    - **像这样使用第二个参数并循环调用indexOf()或 lastIndexOf()，就可以在字符串中找到所有的目标子字符串。**

19. 字符串包含方法
    - ECMAScript 6 增加了 3 个用于判断字符串中是否包含另一个字符串的方法：startsWith()、endsWith()和 includes()。这些方法都会从字符串中搜索传入的字符串，并返回一个表示是否包含的布尔值。它们的区别在于，startsWith()检查开始于索引 0 的匹配项，endsWith()检查开始于索引(string.length - substring.length)的匹配项，而 includes()检查整个字符串：
        ```
        let message = "foobarbaz"; 
        console.log(message.startsWith("foo"));     // true 
        console.log(message.startsWith("bar"));     // false 
        console.log(message.endsWith("baz"));       // true 
        console.log(message.endsWith("bar"));       // false 
        console.log(message.includes("bar"));       // true 
        console.log(message.includes("qux"));       // false
        ```
    - startsWith()和 includes()方法接收可选的第二个参数，表示开始搜索的位置。如果传入第二个参数，则意味着这两个方法会从指定位置向着字符串末尾搜索，忽略该位置之前的所有字符。
    - endsWith()方法接收可选的第二个参数，表示应该当作字符串末尾的位置。如果不提供这个参数，那么默认就是字符串长度。如果提供这个参数，那么就好像字符串只有那么多字符一样：
        ```
        let message = "foobarbaz"; 
        console.log(message.endsWith("bar"));       // false 
        console.log(message.endsWith("bar", 6));    // true
        ```

20. 字符串的 trim() 方法
    - ECMAScript 在所有字符串上都提供了 trim()方法。这个方法会创建字符串的一个副本，删除前、后所有空格符，再返回结果。
    - 由于 trim()返回的是字符串的副本，因此原始字符串不受影响，即原本的前、后空格符都会保留。
    - 另外，trimLeft()和 trimRight()方法分别用于从字符串开始和末尾清理空格符。

21. 字符串的 repeat() 方法。ECMAScript 在所有字符串上都提供了 repeat()方法。这个方法接收一个整数参数，表示要将字符串复制多少次，然后返回拼接所有副本后的结果。
    ```
    let stringValue = "na "; 
    console.log(stringValue.repeat(16) + "batman"); 
    // na na na na na na na na na na na na na na na na batman
    ```
    
22. 字符串的 padStart() 和 padEnd() 方法。padStart()和 padEnd()方法会复制字符串，如果小于指定长度，则在相应一边填充字符，直至满足长度条件。这两个方法的第一个参数是长度，第二个参数是可选的填充字符串，默认为空格（U+0020）。
    ```
    let stringValue = "foo"; 
    console.log(stringValue.padStart(6));       // " foo" 
    console.log(stringValue.padStart(9, "."));  // "......foo" 
    console.log(stringValue.padEnd(6));         // "foo " 
    console.log(stringValue.padEnd(9, "."));    // "foo......"
    
    let stringValue = "foo"; 
    console.log(stringValue.padStart(8, "bar"));    // "barbafoo" 
    console.log(stringValue.padStart(2));           // "foo" 
    console.log(stringValue.padEnd(8, "bar"));      // "foobarba" 
    console.log(stringValue.padEnd(2));             // "foo"
    ```
    - 可选的第二个参数并不限于一个字符。如果提供了多个字符的字符串，则会将其拼接并截断以匹配指定长度。
    - 此外，如果长度小于或等于字符串长度，则会返回原始字符串。

23. 字符串迭代与解构
    - 字符串的原型上暴露了一个@@iterator 方法，表示可以迭代字符串的每个字符。可以像下面这样手动使用迭代器：
        ```
        let message = "abc"; 
        let stringIterator = message[Symbol.iterator](); 
        console.log(stringIterator.next()); // {value: "a", done: false} 
        console.log(stringIterator.next()); // {value: "b", done: false} 
        console.log(stringIterator.next()); // {value: "c", done: false} 
        console.log(stringIterator.next()); // {value: undefined, done: true}
        ```
    - 在 for-of 循环中可以通过这个迭代器按序访问每个字符：
        ```
        for (const c of "abcde") { 
            console.log(c); 
        } 
        // a 
        // b 
        // c 
        // d 
        // e
        ```
    - 有了这个迭代器之后，字符串就可以通过解构操作符来解构了。比如，可以更方便地把字符串分割为字符数组：
        ```
        let message = "abcde"; 
        console.log([...message]); // ["a", "b", "c", "d", "e"]
        ```

24. 字符串大小写转换。下一组方法涉及大小写转换，包括 4 个方法：toLowerCase()、toLocaleLowerCase()、toUpperCase()和toLocaleUpperCase()。toLowerCase()和toUpperCase()方法是原来就有的方法，与 java.lang.String 中的方法同名。toLocaleLowerCase()和 toLocaleUpperCase()方法旨在基于特定地区实现。在很多地区，地区特定的方法与通用的方法是一样的。但在少数语言中（如土耳其语），Unicode 大小写转换需应用特殊规则，要使用地区特定的方法才能实现正确转换。下面是几个例子：
    ```
    let stringValue = "hello world"; 
    console.log(stringValue.toLocaleUpperCase()); // "HELLO WORLD" 
    console.log(stringValue.toUpperCase()); // "HELLO WORLD" 
    console.log(stringValue.toLocaleLowerCase()); // "hello world" 
    console.log(stringValue.toLowerCase()); // "hello world"
    ```
    - 通常，如果不知道代码涉及什么语言，则最好使用地区特定的转换方法。

25. 字符串模式匹配方法
    - String 类型专门为在字符串中实现模式匹配设计了几个方法。
    - 第一个就是 match()方法，这个方法本质上跟 RegExp 对象的 exec()方法相同。match()方法接收一个参数，可以是一个正则表达式字符串，也可以是一个 RegExp 对象。来看下面的例子：
        ```
        let text = "cat, bat, sat, fat"; 
        let pattern = /.at/; 
        // 等价于 pattern.exec(text) 
        let matches = text.match(pattern); 
        console.log(matches.index);     // 0 
        console.log(matches[0]);        // "cat" 
        console.log(pattern.lastIndex); // 0
        ```
        - match()方法返回的数组与 RegExp 对象的 exec()方法返回的数组是一样的：第一个元素是与整个模式匹配的字符串，其余元素则是与表达式中的捕获组匹配的字符串（如果有的话）。
    - 另一个查找模式的字符串方法是 search()。这个方法唯一的参数与 match()方法一样：正则表达式字符串或 RegExp 对象。这个方法返回模式第一个匹配的位置索引，如果没找到则返回-1。search()始终从字符串开头向后匹配模式。看下面的例子：
        ```
        let text = "cat, bat, sat, fat"; 
        let pos = text.search(/at/); 
        console.log(pos);   // 1
        ```
        - 这里，search(/at/)返回 1，即"at"的第一个字符在字符串中的位置。
    - 为简化子字符串替换操作，ECMAScript 提供了 replace()方法。这个方法接收两个参数，第一个参数可以是一个 RegExp 对象或一个字符串（这个字符串不会转换为正则表达式），第二个参数可以是一个字符串或一个函数。如果第一个参数是字符串，那么只会替换第一个子字符串。要想替换所有子字符串，第一个参数必须为正则表达式并且带全局标记，如下面的例子所示：
        ```
        let text = "cat, bat, sat, fat"; 
        let result = text.replace("at", "ond"); 
        console.log(result); // "cond, bat, sat, fat" 
        
        result = text.replace(/at/g, "ond"); 
        console.log(result); // "cond, bond, sond, fond"
        ```
    - replace()方法第二个参数是字符串的情况下，有几个特殊的字符序列，可以用来插入正则表达式操作的值。ECMA-262 中规定了下表中的值：
    
        | 字符序列 | 替换文本 |
        | --- | --- |
        | $$ | $ |
        | $& | 匹配整个模式的子字符串。与 RegExp.lastMatch 相同 |
        | $' | 匹配的子字符串之前的字符串。与 RegExp.rightContext 相同 |
        | $` | 匹配的子字符串之后的字符串。与 RegExp.leftContext 相同 |
        | $n | 匹配第 n 个捕获组的字符串，其中 n 是 0~9。比如，$1 是匹配第一个捕获组的字符串，$2 是匹配第二个捕获组的字符串，以此类推。如果没有捕获组，则值为空字符串 |
        | $nn | 匹配第 nn 个捕获组字符串，其中 nn 是 01~99。比如，$01 是匹配第一个捕获组的字符串，$02 是匹配第二个捕获组的字符串，以此类推。如果没有捕获组，则值为空字符串 |

    - 使用这些特殊的序列，可以在替换文本中使用之前匹配的内容，如下面的例子所示：
        ```
        let text = "cat, bat, sat, fat"; 
        result = text.replace(/(.at)/g, "word ($1)"); 
        console.log(result); // word (cat), word (bat), word (sat), word (fat)
        ```
        - 这里，每个以"at"结尾的词都会被替换成"word"后跟一对小括号，其中包含捕获组匹配的内容$1。
        - replace()的第二个参数可以是一个函数。在只有一个匹配项时，这个函数会收到 3 个参数：与整个模式匹配的字符串、匹配项在字符串中的开始位置，以及整个字符串。在有多个捕获组的情况下，每个匹配捕获组的字符串也会作为参数传给这个函数，但最后两个参数还是与整个模式匹配的开始位置和原始字符串。这个函数应该返回一个字符串，表示应该把匹配项替换成什么。使用函数作为第二个参数可以更细致地控制替换过程，如下所示：
            ```
            function htmlEscape(text) { 
                return text.replace(/[<>"&]/g, function(match, pos, originalText) { 
                    switch(match) { 
                        case "<": 
                            return "&lt;"; 
                        case ">": 
                            return "&gt;"; 
                        case "&": 
                            return "&amp;"; 
                        case "\"": 
                            return "&quot;"; 
                    } 
                }); 
            } 
            console.log(htmlEscape("<p class=\"greeting\">Hello world!</p>")); 
            // "&lt;p class=&quot;greeting&quot;&gt;Hello world!</p>"
            ```
            - 这里，函数 htmlEscape()用于将一段 HTML 中的 4 个字符替换成对应的实体：小于号、大于号、和号，还有双引号（都必须经过转义）。
            - 实现这个任务最简单的办法就是用一个正则表达式查找这些字符，然后定义一个函数，根据匹配的每个字符分别返回特定的 HTML 实体。
        - 最后一个与模式匹配相关的字符串方法是 split()。这个方法会根据传入的分隔符将字符串拆分成数组。作为分隔符的参数可以是字符串，也可以是 RegExp 对象。（字符串分隔符不会被这个方法当成正则表达式。）还可以传入第二个参数，即数组大小，确保返回的数组不会超过指定大小。
            ```
            let colorText = "red,blue,green,yellow"; 
            let colors1 = colorText.split(",");         // ["red", "blue", "green", "yellow"] 
            let colors2 = colorText.split(",", 2);      // ["red", "blue"] 
            let colors3 = colorText.split(/[^,]+/);     // ["", ",", ",", ",", ""]
            ```
            - 最后，使用正则表达式可以得到一个包含逗号的数组。注意在最后一次调用 split()时，返回的数组前后包含两个空字符串。这是因为正则表达式指定的分隔符出现在了字符串开头（"red"）和末尾（"yellow"）。
            - 读者注：^匹配输入字符串的开始位置，除非在方括号表达式中使用，当该符号在方括号表达式中使用时，表示不接受该方括号表达式中的字符集合。要匹配 ^ 字符本身，请使用 \^。所以上面最后一句返回了5个元素。
            
26. 字符串的 localeCompare() 方法
    - localeCompare() 方法比较两个字符串，返回如下 3 个值中的一个：
        - 如果按照字母表顺序，字符串应该排在字符串参数前头，则返回负值。（通常是-1，具体还要看与实际值相关的实现。）
        - 如果字符串与字符串参数相等，则返回 0。
        - 如果按照字母表顺序，字符串应该排在字符串参数后头，则返回正值。（通常是 1，具体还要看与实际值相关的实现。）
    - 例子：
        ```
        let stringValue = "yellow"; 
        console.log(stringValue.localeCompare("brick"));    // 1 
        console.log(stringValue.localeCompare("yellow"));   // 0 
        console.log(stringValue.localeCompare("zoo"));      // -1
        ```
        - "brick"按字母表顺序应该排在"yellow"前头，因此 localeCompare()返回 1。"zoo"在"yellow"后面，因此 localeCompare()返回-1。
    - 因为返回的具体值可能因具体实现而异，所以最好像下面的示例中一样使用 localeCompare()：
        ```
        function determineOrder(value) { 
            let result = stringValue.localeCompare(value); 
            if (result < 0) { 
                console.log(`The string 'yellow' comes before the string '${value}'.`); 
            } else if (result > 0) { 
                console.log(`The string 'yellow' comes after the string '${value}'.`); 
            } else { 
                console.log(`The string 'yellow' is equal to the string '${value}'.`); 
            } 
        } 
        determineOrder("brick"); 
        determineOrder("yellow"); 
        determineOrder("zoo");
        ```
        - 这样一来，就可以保证在所有实现中都能正确判断字符串的顺序了。
    - localeCompare()的独特之处在于，实现所在的地区（国家和语言）决定了这个方法如何比较字符串。在美国，英语是 ECMAScript 实现的标准语言，localeCompare()区分大小写，大写字母排在小写字母前面。但其他地区未必是这种情况。

27. 字符串的 HTML 方法。早期的浏览器开发商认为使用 JavaScript 动态生成 HTML 标签是一个需求。因此，早期浏览器扩展了规范，增加了辅助生成 HTML 标签的方法。不过，这些方法基本上已经没有人使用了，因为结果通常不是语义化的标记。

28. 单例内置对象
    - ECMA-262 对内置对象的定义是“任何由 ECMAScript 实现提供、与宿主环境无关，并在 ECMAScript程序开始执行时就存在的对象”。这就意味着，开发者不用显式地实例化内置对象，因为它们已经实例化好了。
    - 我们已经接触了大部分内置对象，包括 Object、Array 和 String。本节介绍 ECMA-262 定义的另外两个单例内置对象：Global 和 Math。

29. Global
    - Global 对象是 ECMAScript 中最特别的对象，因为代码不会显式地访问它。
    - ECMA-262 规定 Global 对象为一种兜底对象，它所针对的是不属于任何对象的属性和方法。事实上，不存在全局变量或全局函数这种东西。在全局作用域中定义的变量和函数都会变成 Global 对象的属性。
    - isNaN()、isFinite()、parseInt()和 parseFloat()，实际上都是 Global 对象的方法。除了这些，Global 对象上还有另外一些方法。

30. Global 的 URL 编码方法
    - encodeURI()和 encodeURIComponent()方法用于编码统一资源标识符（URI），以便传给浏览器。有效的 URI 不能包含某些字符，比如空格。使用 URI 编码方法来编码 URI 可以让浏览器能够理解它们，同时又以特殊的 UTF-8 编码替换掉所有无效字符。
    - encodeURI()方法用于对整个 URI 进行编码，比如"www.wrox.com/illegal value.js"。而encodeURIComponent()方法用于编码 URI 中单独的组件，比如前面 URL 中的"illegal value.js"。这两个方法的主要区别是，encodeURI()不会编码属于 URL 组件的特殊字符，比如冒号、斜杠、问号、井号，而 encodeURIComponent()会编码它发现的所有非标准字符。
        ```
        let uri = "http://www.wrox.com/illegal value.js#start"; 
        
        // "http://www.wrox.com/illegal%20value.js#start" 
        console.log(encodeURI(uri)); 
        
        // "http%3A%2F%2Fwww.wrox.com%2Fillegal%20value.js%23start" 
        console.log(encodeURIComponent(uri));
        ```
        - 这里使用 encodeURI()编码后，除空格被替换为%20 之外，没有任何变化。而 encodeURIComponent()方法将所有非字母字符都替换成了相应的编码形式。这就是使用 encodeURI()编码整个URI，但只使用 encodeURIComponent()编码那些会追加到已有 URI 后面的字符串的原因。
    - 注意，一般来说，使用 encodeURIComponent()应该比使用 encodeURI()的频率更高，这是因为编码查询字符串参数比编码基准 URI 的次数更多。
    - 与 encodeURI()和 encodeURIComponent()相对的是 decodeURI()和 decodeURIComponent()。decodeURI()只对使用 encodeURI()编码过的字符解码。例如，%20 会被替换为空格，但%23 不会被替换为井号（#），因为井号不是由 encodeURI()替换的。类似地，decodeURIComponent()解码所有被 encodeURIComponent()编码的字符，基本上就是解码所有特殊值。
        ```
        let uri = "http%3A%2F%2Fwww.wrox.com%2Fillegal%20value.js%23start"; 
        
        // http%3A%2F%2Fwww.wrox.com%2Fillegal value.js%23start 
        console.log(decodeURI(uri)); 
        
        // http:// www.wrox.com/illegal value.js#start 
        console.log(decodeURIComponent(uri));
        ```
    - 注意，URI方法 encodeURI()、encodeURIComponent()、decodeURI()和 decodeURIComponent()取代了 escape()和 unescape()方法，后者在 ECMA-262 第 3 版中就已经废弃了。URI 方法始终是首选方法，因为它们对所有 Unicode 字符进行编码，而原来的方法只能正确编码 ASCII 字符。不要在生产环境中使用 escape()和 unescape()。

31. Global 的 eval()方法
    - eval() 方法就是一个完整的 ECMAScript 解释器，它接收一个参数，即一个要执行的 ECMAScript（JavaScript）字符串。
        ```
        eval("console.log('hi')"); 
        上面这行代码的功能与下一行等价：
        console.log("hi");
        ```
    - 当解释器发现 eval()调用时，会将参数解释为实际的 ECMAScript 语句，然后将其插入到该位置。通过 eval()执行的代码属于该调用所在上下文，被执行的代码与该上下文拥有相同的作用域链。这意味着定义在包含上下文中的变量可以在 eval()调用内部被引用。类似地，可以在 eval()内部定义一个函数或变量，然后在外部代码中引用。
        ```
        let msg = "hello world"; 
        eval("console.log(msg)"); // "hello world"
        
        eval("function sayHi() { console.log('hi'); }"); 
        sayHi();
        
        eval("let msg = 'hello world';"); 
        console.log(msg); // Reference Error: msg is not defined
        ```
        - 第二行代码会被替换成一行真正的函数调用代码。
        - 函数 sayHi()是在 eval()内部定义的。因为该调用会被替换为真正的函数定义，所以才可能在下一行代码中调用 sayHi()。对于变量也是一样的。
        - 通过 eval()定义的任何变量和函数都不会被提升，这是因为在解析代码的时候，它们是被包含在一个字符串中的。它们只是在 eval()执行的时候才会被创建。
        - 在严格模式下，在 eval()内部创建的变量和函数无法被外部访问。换句话说，最后两个例子会报错。
        - 同样，在严格模式下，赋值给 eval 也会导致错误：
            ```
            "use strict"; 
            eval = "hi"; // 导致错误
            ```

32. Global 对象属性。像 undefined、NaN 和 Infinity 等特殊值都是 Global 对象的属性。此外，所有原生引用类型构造函数，比如 Object 和 Function，也都是 Global 对象的属性。下表列出了所有这些属性：

    | 属性 | 说明 |
    | --- | --- |
    | undefined | 特殊值 undefined
    | NaN | 特殊值 NaN
    | Infinity | 特殊值 Infinity
    | Object | Object 的构造函数
    | Array | Array 的构造函数
    | Function | Function 的构造函数
    | Boolean | Boolean 的构造函数
    | String | String 的构造函数
    | Number | Number 的构造函数
    | Date | Date 的构造函数
    | RegExp | RegExp 的构造函数
    | Symbol | Symbol 的伪构造函数
    | Error | Error 的构造函数
    | EvalError | EvalError 的构造函数
    | RangeError | RangeError 的构造函数
    | ReferenceError | ReferenceError 的构造函数
    | SyntaxError | SyntaxError 的构造函数
    | TypeError | TypeError 的构造函数
    | URIError | URIError 的构造函数

33. window 对象
    - 虽然 ECMA-262 没有规定直接访问 Global 对象的方式，但浏览器将 window 对象实现为 Global 对象的代理。因此，所有全局作用域中声明的变量和函数都变成了 window 的属性。
        ```
        var color = "red"; 
        function sayColor() { 
            console.log(window.color); 
        } 
        window.sayColor(); // "red"
        ```
        - 这里定义了一个名为color的全局变量和一个名为sayColor()的全局函数。在sayColor()内部，通过 window.color 访问了 color 变量，说明全局变量变成了 window 的属性。接着，又通过 window对象直接调用了 window.sayColor()函数，从而输出字符串。
    - 注意，window 对象在 JavaScript 中远不止实现了 ECMAScript 的 Global 对象那么简单。
    - 另一种获取 Global 对象的方式是使用如下的代码：
        ```
        let global = function() { 
            return this; 
        }();
        ```
        - 这段代码创建一个立即调用的函数表达式，返回了 this 的值。如前所述，**当一个函数在没有明确（通过成为某个对象的方法，或者通过 call()/apply()）指定 this 值的情况下执行时，this 值等于 Global 对象。因此，调用一个简单返回 this 的函数是在任何执行上下文中获取 Global 对象的通用方式。**

34. Math
    - ECMAScript 提供了 Math 对象作为保存数学公式、信息和计算的地方。
    - Math 对象上提供的计算要比直接在 JavaScript 实现的快得多，因为 Math 对象上的计算使用了 JavaScript 引擎中更高效的实现和处理器指令。但使用 Math 计算的问题是精度会因浏览器、操作系统、指令集和硬件而异。

35. Math 对象有一些属性，主要用于保存数学中的一些特殊值。

    | 属性 | 说明 |
    | --- | --- |
    | Math.E | 自然对数的基数 e 的值
    | Math.LN10 | 10 为底的自然对数
    | Math.LN2 | 2 为底的自然对数
    | Math.LOG2E | 以 2 为底 e 的对数
    | Math.LOG10E | 以 10 为底 e 的对数
    | Math.PI | π 的值
    | Math.SQRT1_2 | 1/2 的平方根
    | Math.SQRT2 | 2 的平方根

36. Math 的 min()和 max()方法
    - min()和 max()方法用于确定一组数值中的最小值和最大值。这两个方法都接收任意多个参数。
        ```
        let max = Math.max(3, 54, 32, 16); 
        console.log(max); // 54 
        
        let min = Math.min(3, 54, 32, 16); 
        console.log(min); // 3
        ```
    - 使用这两个方法可以避免使用额外的循环和 if 语句来确定一组数值的最大最小值。
    - 要知道数组中的最大值和最小值，可以像下面这样使用扩展操作符：
        ```
        let values = [1, 2, 3, 4, 5, 6, 7, 8]; 
        let max = Math.max(...val);
        ```

37. Math 的 舍入方法
    - 用于把小数值舍入为整数的 4 个方法：Math.ceil()、Math.floor()、Math.round()和 Math.fround()。这几个方法处理舍入的方式如下所述：
        - Math.ceil()方法始终向上舍入为最接近的整数。
        - Math.floor()方法始终向下舍入为最接近的整数。
        - Math.round()方法执行四舍五入。
        - Math.fround()方法返回数值最接近的单精度（32 位）浮点值表示。
    - 例子：
        ```
        console.log(Math.ceil(25.9)); // 26 
        console.log(Math.ceil(25.5)); // 26 
        console.log(Math.ceil(25.1)); // 26 
        
        console.log(Math.round(25.9)); // 26 
        console.log(Math.round(25.5)); // 26 
        console.log(Math.round(25.1)); // 25 
        
        console.log(Math.fround(0.4));  // 0.4000000059604645 
        console.log(Math.fround(0.5));  // 0.5 
        console.log(Math.fround(25.9)); // 25.899999618530273 
        
        console.log(Math.floor(25.9)); // 25 
        console.log(Math.floor(25.5)); // 25 
        console.log(Math.floor(25.1)); // 25
        ```

38. Math 的 random()方法
    - Math.random()方法返回一个 0~1 范围内的随机数，其中包含 0 但不包含 1。
    - 可以基于如下公式使用 Math.random()从一组整数中随机选择一个数：
        ```
        number = Math.floor(Math.random() * total_number_of_choices + first_possible_value)
        ```
    - 例子：
        ```
        let num = Math.floor(Math.random() * 10 + 1);   // 从 1~10 范围内随机选择一个数
        let num = Math.floor(Math.random() * 9 + 2);    // 选择一个 2~10 范围内的值
        ```
        - 2~10 只有 9 个数，所以可选总数（total_number_of_choices）是 9，而最小可能的值（first_possible_value）是 2。
    - 很多时候，通过函数来算出可选总数和最小可能的值可能更方便：
        ```
        function selectFrom(lowerValue, upperValue) {
            let choices = upperValue - lowerValue + 1; 
            return Math.floor(Math.random() * choices + lowerValue); 
        } 
        let num = selectFrom(2,10); 
        console.log(num);   // 2~10 范围内的值，其中包含 2 和 10
        
        let colors = ["red", "green", "blue", "yellow", "black", "purple", "brown"]; 
        let color = colors[selectFrom(0, colors.length-1)];
        ```
    - **注意，如果是为了加密而需要生成随机数（传给生成器的输入需要较高的不确定性），那么建议使用 window.crypto.getRandomValues()。**

39. Math 的其他方法

    | 方法 | 说明 |
    | --- | --- |
    | Math.abs(x) | 返回 x 的绝对值
    | Math.exp(x) | 返回 Math.E 的 x 次幂
    | Math.expm1(x) | 等于 Math.exp(x) - 1
    | Math.log(x) | 返回 x 的自然对数
    | Math.log1p(x) | 等于 1 + Math.log(x)
    | Math.pow(x, power) | 返回 x 的 power 次幂
    | Math.hypot(...nums) | 返回 nums 中每个数平方和的平方根
    | Math.clz32(x) | 返回 32 位整数 x 的前置零的数量
    | Math.sign(x) | 返回表示 x 符号的 1、0、-0 或-1
    | Math.trunc(x) | 返回 x 的整数部分，删除所有小数
    | Math.sqrt(x) | 返回 x 的平方根
    | Math.cbrt(x) | 返回 x 的立方根
    | Math.acos(x) | 返回 x 的反余弦
    | Math.acosh(x) | 返回 x 的反双曲余弦
    | Math.asin(x) | 返回 x 的反正弦
    | Math.asinh(x) | 返回 x 的反双曲正弦
    | Math.atan(x) | 返回 x 的反正切
    | Math.atanh(x) | 返回 x 的反双曲正切
    | Math.atan2(y, x) | 返回 y/x 的反正切
    | Math.cos(x) | 返回 x 的余弦
    | Math.sin(x) | 返回 x 的正弦
    | Math.tan(x) | 返回 x 的正切

    - 即便这些方法都是由 ECMA-262 定义的，对正弦、余弦、正切等计算的实现仍然取决于浏览器，因为计算这些值的方式有很多种。结果，这些方法的精度可能因实现而异。

40. 本章总结
    - JavaScript 中的对象称为引用值。引用值与传统面向对象编程语言中的类相似，但实现不同。
    - **JavaScript 比较独特的一点是，函数实际上是 Function 类型的实例，也就是说函数也是对象。因为函数也是对象，所以函数也有方法，可以用于增强其能力。**
    - **由于原始值包装类型的存在，JavaScript 中的原始值可以被当成对象来使用。**有 3 种原始值包装类型：Boolean、Number 和 String。它们都具备如下特点：
        - 每种包装类型都映射到同名的原始类型。
        - 以读模式访问原始值时，后台会实例化一个原始值包装类型的对象，借助这个对象可以操作相应的数据。
        - 涉及原始值的语句执行完毕后，包装对象就会被销毁。
    - 当代码开始执行时，全局上下文中会存在两个内置对象：Global 和 Math。其中，Global 对象在大多数 ECMAScript 实现中无法直接访问。不过，浏览器将其实现为 window 对象。所有全局变量和函数都是 Global 对象的属性。Math 对象包含辅助完成复杂计算的属性和方法。
