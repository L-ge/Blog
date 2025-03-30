###### 三、语言基础

1. ECMAScript 的语法很大程度上借鉴了 C 语言和其他类 C 语言，如 Java 和 Perl。ECMAScript 中一切都区分大小写。无论是变量、函数名还是操作符，都区分大小写。

2. 所谓标识符，就是变量、函数、属性或函数参数的名称。标识符可以由一或多个下列字符组成：
    - 第一个字符必须是一个字母、下划线（_）或美元符号（$）；
    - 剩下的其他字符可以是字母、下划线、美元符号或数字。

3. 标识符中的字母可以是扩展 ASCII（Extended ASCII）中的字母，也可以是 Unicode 的字母字符，如 À 和 Æ（但不推荐使用）。

4. 按照惯例，ECMAScript 标识符使用驼峰大小写形式，即第一个单词的首字母小写，后面每个单词的首字母大写。

5. ECMAScript 采用 C 语言风格的注释，包括单行注释和块注释。
    - 单行注释以两个斜杠字符开头，如：
        ```
        // 单行注释
        ```
    - 块注释以一个斜杠和一个星号（/\*）开头，以它们的反向组合（\*/）结尾，如：
        ```
        /* 这是多行
        注释 */
        ```

6. ECMAScript 5 增加了严格模式（strict mode）的概念。严格模式是一种不同的 JavaScript 解析和执行模型，ECMAScript 3 的一些不规范写法在这种模式下会被处理，对于不安全的活动将抛出错误。
    - 要对整个脚本启用严格模式，在脚本开头加上这一行：
        ```
        "use strict";
        ```
    - 虽然看起来像个没有赋值给任何变量的字符串，但它其实是一个预处理指令。任何支持的 JavaScript 引擎看到它都会切换到严格模式。选择这种语法形式的目的是不破坏 ECMAScript 3 语法。
    - 也可以单独指定一个函数在严格模式下执行，只要把这个预处理指令放到函数体开头即可。
        ```
        function doSomething() { 
            "use strict"; 
            // 函数体 
        }
        ```
    - 严格模式会影响 JavaScript 执行的很多方面。
    - 所有现代浏览器都支持严格模式。

7. ECMAScript 中的语句以分号结尾。省略分号意味着由解析器确定语句在哪里结尾。
    - 即使语句末尾的分号不是必需的，也应该加上。
    - 记着加分号有助于防止省略造成的问题，比如可以避免输入内容不完整。
    - 加分号也便于开发者通过删除空行来压缩代码（如果没有结尾的分号，只删除空行，则会导致语法错误）。
    - 加分号也有助于在某些情况下提升性能，因为解析器会尝试在合适的位置补上分号以纠正语法错误。

8. if 之类的控制语句只在执行多条语句时要求必须有代码块。不过，最佳实践是始终在控制语句中使用代码块，即使要执行的只有一条语句。
    ```
    // 有效，但容易导致错误，应该避免
    if (test) 
        console.log(test); 
        
    // 推荐
    if (test) { 
        console.log(test); 
    }
    ```

9. ECMA-262 描述了一组保留的关键字，这些关键字有特殊用途。按照规定，保留的关键字不能用作标识符或属性名。
    - ECMA-262 第 6 版规定的所有关键字如下：
        ```
        break do in typeof 
        case else instanceof var 
        catch export new void 
        class extends return while 
        const finally super with 
        continue for switch yield 
        debugger function this 
        default if throw 
        delete import try
        ```
    - 以下是 ECMA-262 第 6 版为将来保留的所有词汇：
        ```
        始终保留: 
        enum 
        
        严格模式下保留: 
        implements package public 
        interface protected static 
        let private 
        
        模块代码中保留: 
        await
        ```

10. ECMAScript 变量是松散类型的，意思是变量可以用于保存任何类型的数据。每个变量只不过是一个用于保存任意值的命名占位符。有 3 个关键字可以声明变量：var、const 和 let。其中，var 在 ECMAScript 的所有版本中都可以使用，而 const 和 let 只能在 ECMAScript 6 及更晚的版本中使用。

11. var 关键字
    - 要定义变量，可以使用 var 操作符（注意 var 是一个关键字），后跟变量名（即标识符）：
        ```
        var message;
        
        var message = "hi";
        message = 100; // 合法，但不推荐
        ```
        - 第一行代码定义了一个名为 message 的变量，可以用它保存任何类型的值。（不初始化的情况下，变量会保存一个特殊值 undefined）。
        - ECMAScript 实现变量初始化，因此可以同时定义变量并设置它的值。像这样初始化变量不会将它标识为字符串类型，只是一个简单的赋值而已。随后，不仅可以改变保存的值，也可以改变值的类型。
    - 使用 var 操作符定义的变量会成为包含它的函数的局部变量。在函数内定义变量时省略 var 操作符，可以创建一个全局变量。
        ```
        function test() { 
            var message = "hi"; // 局部变量
        } 
        test(); 
        console.log(message); // 出错！
    
        function test() { 
            message = "hi"; // 全局变量
        } 
        test(); 
        console.log(message); // "hi"
        ```
        - 去掉之前的 var 操作符之后，message 就变成了全局变量。只要调用一次函数 test()，就会定义这个变量，并且可以在函数外部访问到。
        - 虽然可以通过省略 var 操作符定义全局变量，但不推荐这么做。在局部作用域中定义的全局变量很难维护，也会造成困惑。这是因为不能一下子断定省略 var 是不是有意而为之。在严格模式下，如果像这样给未声明的变量赋值，则会导致抛出 ReferenceError。
    - 如果需要定义多个变量，可以在一条语句中用逗号分隔每个变量（及可选的初始化）：
        ```
        var message = "hi", 
            found = false, 
            age = 29;
        ```
        - 因为 ECMAScript 是松散类型的，所以使用不同数据类型初始化的变量可以用一条语句来声明。插入换行和空格缩进并不是必需的，但这样有利于阅读理解。
    - 在严格模式下，不能定义名为 eval 和 arguments 的变量，否则会导致语法错误。
    - 使用 var 时，下面的代码不会报错。这是因为使用这个关键字声明的变量会自动提升到函数作用域顶部：
        ```
        function foo() { 
            console.log(age); 
            var age = 26; 
        } 
        foo(); // undefined
        
        // 之所以不会报错，是因为 ECMAScript 运行时把它看成等价于如下代码：
        function foo() { 
            var age; 
            console.log(age); 
            age = 26; 
        } 
        foo(); // undefined
        ```
        - 这就是所谓的“提升”（hoist），也就是把所有变量声明都拉到函数作用域的顶部。（读者注：**只是把声明提前**）
    - 此外，反复多次使用 var 声明同一个变量也没有问题：
        ```
        function foo() { 
            var age = 16; 
            var age = 26; 
            var age = 36; 
            console.log(age); 
        } 
        foo(); // 36
        ```
        - 读者注：（来自GPT3.5）当多次使用var声明同一个变量时，JavaScript只会认为是同一个变量被声明了多次，并不会创建多个不同的变量。这种行为在某些情况下可能会导致意外的结果，尤其是在使用循环或条件语句时。为了避免这种情况，推荐使用let或const关键字来声明变量，因为它们具有块级作用域，可以更好地控制变量的作用域范围。

12. let 声明
    - let 跟 var 的作用差不多，但有着非常重要的区别。最明显的区别是，let 声明的范围是块作用域，而 var 声明的范围是函数作用域。
        ```
        if (true) { 
            var name = 'Matt'; 
            console.log(name); // Matt 
        } 
        console.log(name); // Matt
    
        if (true) { 
            let age = 26; 
            console.log(age); // 26 
        } 
        console.log(age); // ReferenceError: age 没有定义
        ```
        - 读者注：**在JS中，函数作用域和块作用域是分得很清的，var的作用域是函数作用域，var是能越过块作用域的！**
        - 块作用域是函数作用域的子集，因此适用于 var 的作用域限制同样也适用于 let。
    - let 也不允许同一个块作用域中出现冗余声明。这样会导致报错：
        ```
        var name; 
        var name; 
        
        let age; 
        let age; // SyntaxError；标识符 age 已经声明过了
        ```
    - 当然，JavaScript 引擎会记录用于变量声明的标识符及其所在的块作用域，因此嵌套使用相同的标识符不会报错，而这是因为同一个块中没有重复声明。
        ```
        var name = 'Nicholas'; 
        console.log(name);      // 'Nicholas' 
        if (true) { 
            var name = 'Matt'; 
            console.log(name);  // 'Matt' 
        } 
        
        let age = 30; 
        console.log(age);       // 30 
        if (true) { 
            let age = 26; 
            console.log(age);   // 26 
        }
        ```
    - 对声明冗余报错不会因混用 let 和 var 而受影响。这两个关键字声明的并不是不同类型的变量，它们只是指出变量在相关作用域如何存在。
        ```
        var name; 
        let name; // SyntaxError 
        
        let age; 
        var age; // SyntaxError
        ```
    - let 与 var 的另一个重要的区别，就是 let 声明的变量不会在作用域中被提升。
        ```
        // name 会被提升
        console.log(name); // undefined 
        var name = 'Matt'; 
        
        // age 不会被提升
        console.log(age); // ReferenceError：age 没有定义
        let age = 26;
        ```
        - 在解析代码时，JavaScript 引擎也会注意出现在块后面的 let 声明，只不过在此之前不能以任何方式来引用未声明的变量。
        - 在 let 声明之前的执行瞬间被称为“暂时性死区”（temporal dead zone），在此阶段引用任何后面才声明的变量都会抛出 ReferenceError。
    - 与 var 关键字不同，**使用 let 在全局作用域中声明的变量不会成为 window 对象的属性（var 声明的变量则会）**。
        ```
        var name = 'Matt'; 
        console.log(window.name); // 'Matt' 
        
        let age = 26; 
        console.log(window.age); // undefined
        ```
        - 不过，let 声明仍然是在全局作用域中发生的，相应变量会在页面的生命周期内存续。因此，为了避免 SyntaxError，必须确保页面不会重复声明同一个变量。
    - 在使用 var 声明变量时，由于声明会被提升，JavaScript 引擎会自动将多余的声明在作用域顶部合并为一个声明。因为 let 的作用域是块，所以不可能检查前面是否已经使用 let 声明过同名变量，同时也就不可能在没有声明的情况下声明它。
        ```
        <script> 
          var name = 'Nicholas'; 
          let age = 26; 
        </script> 
        <script> 
          // 假设脚本不确定页面中是否已经声明了同名变量
          // 那它可以假设还没有声明过
          
          var name = 'Matt'; 
          // 这里没问题，因为可以被作为一个提升声明来处理
          // 不需要检查之前是否声明过同名变量
          
          let age = 36; 
          // 如果 age 之前声明过，这里会报错
        </script>
        ```
        - 使用 try/catch 语句或 typeof 操作符也不能解决，因为条件块中 let 声明的作用域仅限于该块。
            ```
            <script> 
              let name = 'Nicholas'; 
              let age = 36; 
            </script> 
            
            <script> 
              // 假设脚本不确定页面中是否已经声明了同名变量
              // 那它可以假设还没有声明过
              if (typeof name === 'undefined') { 
                let name; 
              } 
              
              // name 被限制在 if {} 块的作用域内
              // 因此这个赋值形同全局赋值
              name = 'Matt'; 
              
              try { 
                console.log(age); // 如果 age 没有声明过，则会报错
              } 
              catch(error) { 
                let age;
              } 
              // age 被限制在 catch {}块的作用域内
              // 因此这个赋值形同全局赋值
              age = 26; 
            </script>
            ```
            - 为此，对于 let 这个新的 ES6 声明关键字，不能依赖条件声明模式。
            - 不能使用 let 进行条件式声明是件好事，因为条件声明是一种反模式，它让程序变得更难理解。如果你发现自己在使用这个模式，那一定有更好的替代方式。
    - 在 let 出现之前，for 循环定义的迭代变量会渗透到循环体外部。改成使用 let 之后，这个问题就消失了，因为迭代变量的作用域仅限于 for 循环块内部。
        ```
        for (var i = 0; i < 5; ++i) { 
         // 循环逻辑 
        } 
        console.log(i); // 5
        
        for (let i = 0; i < 5; ++i) { 
         // 循环逻辑
        } 
        console.log(i); // ReferenceError: i 没有定义
        ```
    - 在使用 var 的时候，最常见的问题就是对迭代变量的奇特声明和修改：
        ```
        for (var i = 0; i < 5; ++i) { 
            setTimeout(() => console.log(i), 0) 
        } 
        // 你可能以为会输出 0、1、2、3、4 
        // 实际上会输出 5、5、5、5、5
        
        for (let i = 0; i < 5; ++i) { 
            setTimeout(() => console.log(i), 0) 
        } 
        // 会输出 0、1、2、3、4
        ```
        - 之所以会这样，是因为在退出循环时，迭代变量保存的是导致循环退出的值：5。在之后执行超时逻辑时，所有的 i 都是同一个变量，因而输出的都是同一个最终值。
        - 而在使用 let 声明迭代变量时，JavaScript 引擎在后台会为每个迭代循环声明一个新的迭代变量。每个 setTimeout 引用的都是不同的变量实例，所以 console.log 输出的是我们期望的值，也就是循环执行过程中每个迭代变量的值。

13. const 声明
    - const 的行为与 let 基本相同，唯一一个重要的区别是用它声明变量时必须同时初始化变量，且尝试修改 const 声明的变量会导致运行时错误。
        ```
        const age = 26; 
        age = 36; // TypeError: 给常量赋值
        
        // const 也不允许重复声明
        const name = 'Matt'; 
        const name = 'Nicholas'; // SyntaxError 
    
        // const 声明的作用域也是块
        const name = 'Matt'; 
        if (true) { 
            const name = 'Nicholas'; 
        } 
        console.log(name); // Matt
        ```
    - const 声明的限制只适用于它指向的变量的引用。换句话说，**如果 const 变量引用的是一个对象，那么修改这个对象内部的属性并不违反 const 的限制**。 
        ```
        const person = {}; 
        person.name = 'Matt'; // ok
        ```
    - JavaScript 引擎会为 for 循环中的 let 声明分别创建独立的变量实例，虽然 const 变量跟 let 变量很相似，但是不能用 const 来声明迭代变量（因为迭代变量会自增）：
        ```
        for (const i = 0; i < 10; ++i) {} // TypeError：给常量赋值
        ```
        - 如果你只想用 const 声明一个不会被修改的 for 循环变量，那也是可以的。也就是说，每次迭代只是创建一个新变量。这对 for-of 和 for-in 循环特别有意义：
            ```
            let i = 0; 
            for (const j = 7; i < 5; ++i) { 
                console.log(j); 
            } 
            // 7, 7, 7, 7, 7 
            
            for (const key in {a: 1, b: 2}) { 
                console.log(key); 
            } 
            // a, b 
            
            for (const value of [1,2,3,4,5]) { 
                console.log(value); 
            } 
            // 1, 2, 3, 4, 5
            ```

14. 声明风格及最佳实践
    - ECMAScript 6 增加 let 和 const 从客观上为这门语言更精确地声明作用域和语义提供了更好的支持。
    - 不使用 var。有了 let 和 const，大多数开发者会发现自己不再需要 var 了。限制自己只使用 let 和 const 有助于提升代码质量，因为变量有了明确的作用域、声明位置，以及不变的值。
    - const 优先，let 次之。使用 const 声明可以让浏览器运行时强制保持变量不变，也可以让静态代码分析工具提前发现不合法的赋值操作。因此，很多开发者认为应该优先使用 const 来声明变量，只在提前知道未来会有修改时，再使用 let。

15. ECMAScript 有 6 种简单数据类型（也称为原始类型）：Undefined、Null、Boolean、Number、String 和 Symbol。Symbol（符号）是 ECMAScript 6 新增的。还有一种复杂数据类型叫 Object（对象）。Object 是一种无序名值对的集合。因为在 ECMAScript 中不能定义自己的数据类型，所有值都可以用上述 7 种数据类型之一来表示。只有 7 种数据类型似乎不足以表示全部数据。但 ECMAScript 的数据类型很灵活，一种数据类型可以当作多种数据类型来使用。

16. typeof 用来确定任意变量的数据类型。
    - 对一个值使用 typeof 操作符会返回下列字符串之一：
        - "undefined"表示值未定义；
        - "boolean"表示值为布尔值；
        - "string"表示值为字符串；
        - "number"表示值为数值；
        - "object"表示值为对象（而不是函数）或 null；
        - "function"表示值为函数；
        - "symbol"表示值为符号。
    - 使用 typeof 操作符的例子：
        ```
        let message = "some string"; 
        console.log(typeof message);    // "string" 
        console.log(typeof(message));   // "string" 
        console.log(typeof 95);         // "number"
        ```
    - 因为 typeof 是一个操作符而不是函数，所以不需要参数（但可以使用参数）。
    - 注意 typeof 在某些情况下返回的结果可能会让人费解，但技术上讲还是正确的。比如，调用 typeof null 返回的是"object"。这是因为特殊值 null 被认为是一个对空对象的引用。
    - 严格来讲，函数在 ECMAScript 中被认为是对象，并不代表一种数据类型。可是，函数也有自己特殊的属性。为此，就有必要通过 typeof 操作符来区分函数和其他对象。

17. Undefined 类型只有一个值，就是特殊值 undefined。
    - 当使用 var 或 let 声明了变量但没有初始化时，就相当于给变量赋予了 undefined 值：
        ```
        let message; 
        console.log(message == undefined); // true
        ```
        - 变量 message 也可以显式地以 undefined 来初始化。但这是不必要的，因为默认情况下，任何未经初始化的变量都会取得 undefined 值。
    - 一般来说，永远不用显式地给某个变量设置 undefined 值。字面值 undefined 主要用于比较，而且在 ECMA-262 第 3 版之前是不存在的。增加这个特殊值的目的就是为了正式明确空对象指针（null）和未初始化变量的区别。
    - 注意，包含 undefined 值的变量跟未定义变量是有区别的。
    - 对未声明的变量，只能执行一个有用的操作，就是对它调用 typeof。（对未声明的变量调用 delete 也不会报错，但这个操作没什么用，实际上在严格模式下会抛出错误。）
    - 在对未初始化的变量调用 typeof 时，返回的结果是"undefined"，但对未声明的变量调用它时，返回的结果还是"undefined"。
        ```
        let message;    // 这个变量被声明了，只是值为 undefined 
        // 确保没有声明过这个变量
        // let age 
        console.log(typeof message); // "undefined" 
        console.log(typeof age);     // "undefined"
        
        console.log(message);   // "undefined" 
        console.log(age);       // 报错
        ```
        - 无论是声明还是未声明，typeof 返回的都是字符串"undefined"。逻辑上讲这是对的，因为虽然严格来讲这两个变量存在根本性差异，但它们都无法执行实际操作。
        - 即使未初始化的变量会被自动赋予 undefined 值，但我们仍然建议在声明变量的同时进行初始化。这样，当 typeof 返回"undefined"时，你就会知道那是因为给定的变量尚未声明，而不是声明了但未初始化。
    - undefined 是一个假值。因此，如果需要，可以用更简洁的方式检测它。不过要记住，也有很多其他可能的值同样是假值。所以一定要明确自己想检测的就是 undefined 这个字面值，而不仅仅是假值。
        ```
        let message; // 这个变量被声明了，只是值为 undefined 
        // age 没有声明 
        
        if (message) { 
            // 这个块不会执行
        } 
        
        if (!message) { 
            // 这个块会执行
        } 
        
        if (age) { 
            // 这里会报错
        }
        ```

18. Null 类型同样只有一个值，即特殊值 null。
    - 逻辑上讲，null 值表示一个空对象指针，这也是给 typeof 传一个 null 会返回"object"的原因。   
        ```
        let car = null; 
        console.log(typeof car); // "object"
        ```
    - 在定义将来要保存对象值的变量时，建议使用 null 来初始化，不要使用其他值。这样，只要检查这个变量的值是不是 null 就可以知道这个变量是否在后来被重新赋予了一个对象的引用。
        ```
        if (car != null) { 
            // car 是一个对象的引用
        }
        ```
    - undefined 值是由 null 值派生而来的，因此 ECMA-262 将它们定义为表面上相等。用等于操作符（==）比较 null 和 undefined 始终返回 true。
        ```
        console.log(null == undefined);     // true
        console.log(null === undefined);    // false
        ```
    - 即使 null 和 undefined 有关系，它们的用途也是完全不一样的。如前所述，永远不必显式地将变量值设置为 undefined。但 null 不是这样的。任何时候，只要变量要保存对象，而当时又没有那个对象可保存，就要用 null 来填充该变量。这样就可以保持 null 是空对象指针的语义，并进一步将其与 undefined 区分开来。
    - null 是一个假值。因此，如果需要，可以用更简洁的方式检测它。不过要记住，也有很多其他可能的值同样是假值。所以一定要明确自己想检测的就是 null 这个字面值，而不仅仅是假值。
        ```
        let message = null; 
        let age; 
        if (message) { 
            // 这个块不会执行
        } 
        if (!message) { 
            // 这个块会执行
        }
        if (age) { 
            // 这个块不会执行
        } 
        if (!age) { 
            // 这个块会执行
        }
        ```

19. Boolean（布尔值）类型有两个字面值：true 和 false。这两个布尔值不同于数值，因此 true 不等于 1，false 不等于 0。
    - 注意，布尔值字面量 true 和 false 是区分大小写的，因此 True 和 False（及其他大小混写形式）是有效的标识符，但不是布尔值。
    - 虽然布尔值只有两个，但所有其他 ECMAScript 类型的值都有相应布尔值的等价形式。要将一个其他类型的值转换为布尔值，可以调用特定的 Boolean()转型函数：
        ```
        let message = "Hello world!"; 
        let messageAsBoolean = Boolean(message);
        ```
        - Boolean() 转型函数可以在任意类型的数据上调用，而且始终返回一个布尔值。什么值能转换为 true 或 false 的规则取决于数据类型和实际的值。
    - 下表总结了不同类型与布尔值之间的转换规则。理解下表转换非常重要，因为像 if 等流控制语句会自动执行其他类型值到布尔值的转换：
        数据类型 | 转换为 true 的值 | 转换为 false 的值
        ---|---|---
        Boolean | true | false
        String | 非空字符串 | ""（空字符串）
        Number | 非零数值（包括无穷值） | 0、NaN（参见后面的相关内容）
        Object | 任意对象 | null
        Undefined | N/A（不存在） | undefined

20. Number 类型使用 IEEE 754 格式表示整数和浮点值（在某些语言中也叫双精度值）。不同的数值类型相应地也有不同的数值字面量格式。
    - 最基本的数值字面量格式是十进制整数，直接写出来即可。整数也可以用八进制（以 8 为基数）或十六进制（以 16 为基数）字面量表示。对于八进制字面量，第一个数字必须是零（0），然后是相应的八进制数字（数值 0~7）。如果字面量中包含的数字超出了应有的范围，就会忽略前缀的零，后面的数字序列会被当成十进制数。
    - 要创建十六进制字面量，必须让真正的数值前缀 0x（区分大小写），然后是十六进制数字（0~9 以
    及 A~F）。十六进制数字中的字母大小写均可。
        ```
        let intNum = 55; // 整数
        let octalNum1 = 070; // 八进制的 56 
        let octalNum2 = 079; // 无效的八进制值，当成 79 处理
        let octalNum3 = 08; // 无效的八进制值，当成 8 处理
        let hexNum1 = 0xA; // 十六进制 10 
        let hexNum2 = 0x1f; // 十六进制 31
        ```
    - 八进制字面量在严格模式下是无效的，会导致 JavaScript 引擎抛出语法错误。 ECMAScript 2015 或 ES6 中的八进制值通过前缀 0o 来表示；严格模式下，前缀 0 会被视为语法错误，**如果要表示八进制值，应该使用前缀 0o**。
    - 使用八进制和十六进制格式创建的数值在所有数学操作中都被视为十进制数值。
    - 由于 JavaScript 保存数值的方式，实际中可能存在正零（+0）和负零（-0）。正零和负零在所有情况下都被认为是等同的。

21. 浮点值
    - 要定义浮点值，数值中必须包含小数点，而且小数点后面必须至少有一个数字。虽然小数点前面不是必须有整数，但推荐加上。下面是几个例子：
        ```
        let floatNum1 = 1.1; 
        let floatNum2 = 0.1; 
        let floatNum3 = .1;     // 有效，但不推荐
        let floatNum1 = 1.;     // 小数点后面没有数字，当成整数 1 处理
        let floatNum2 = 10.0;   // 小数点后面是零，当成整数 10 处理
        let floatNum = 3.125e7; // 等于 31250000，“以 3.125 作为系数，乘以 10 的 7 次幂。”
        ```
    - 因为存储浮点值使用的内存空间是存储整数值的两倍，所以 ECMAScript 总是想方设法把值转换为整数。在小数点后面没有数字的情况下，数值就会变成整数。类似地，如果数值本身就是整数，只是小数点后面跟着 0（如 1.0），那它也会被转换为整数。
    - 对于非常大或非常小的数值，浮点值可以用科学记数法来表示。科学记数法用于表示一个应该乘以 10 的给定次幂的数值。ECMAScript 中科学记数法的格式要求是一个数值（整数或浮点数）后跟一个大写或小写的字母 e，再加上一个要乘的 10 的多少次幂。
    - 科学记数法也可以用于表示非常小的数值。默认情况下，ECMAScript 会将小数点后至少包含 6 个零的浮点值转换为科学记数法（例如，0.000 000 3 会被转换为 3e-7）。
    - 浮点值的精确度最高可达 17 位小数，但在算术计算中远不如整数精确。例如，0.1 加 0.2 得到的不是 0.3，而是 0.300 000 000 000 000 04。
        ```
        if (a + b == 0.3) { // 别这么干！ 
            console.log("You got 0.3."); 
        }
        ```
        - 这里检测两个数值之和是否等于 0.3。如果两个数值分别是 0.05 和 0.25，或者 0.15 和 0.15，那没问题（读者注：不知道为啥没问题）。但如果是 0.1 和 0.2，如前所述，测试将失败。因此永远不要测试某个特定的浮点值。
        - 之所以存在这种舍入错误，是因为使用了 IEEE 754 数值，这种错误并非 ECMAScript 所独有。其他使用相同格式的语言也有这个问题。

21. 由于内存的限制，ECMAScript 并不支持表示这个世界上的所有数值。ECMAScript 可以表示的最小数值保存在 Number.MIN_VALUE 中，这个值在多数浏览器中是 5e-324；可以表示的最大数值保存在 Number.MAX_VALUE 中，这个值在多数浏览器中是 1.797 693 134 862 315 7e+308。
    - 如果某个计算得到的数值结果超出了 JavaScript 可以表示的范围，那么这个数值会被自动转换为一个特殊的 Infinity（无穷）值。任何无法表示的负数以-Infinity（负无穷大）表示，任何无法表示的正数以 Infinity（正无穷大）表示。
    - 如果计算返回正 Infinity 或负 Infinity，则该值将不能再进一步用于任何计算。这是因为 Infinity 没有可用于计算的数值表示形式。要确定一个值是不是有限大（即介于 JavaScript 能表示的最小值和最大值之间），可以使用 isFinite() 函数。
        ```
        let result = Number.MAX_VALUE + Number.MAX_VALUE; 
        console.log(isFinite(result)); // false
        ```
    - 使用 Number.NEGATIVE_INFINITY 和 Number.POSITIVE_INFINITY 也可以获取正、负 Infinity。没错，这两个属性包含的值分别就是 -Infinity 和 Infinity。

22. NaN
    - 有一个特殊的数值叫 NaN，意思是“不是数值”（Not a Number），用于表示本来要返回数值的操作失败了（而不是抛出错误）。比如，用 0 除任意数值在其他语言中通常都会导致错误，从而中止代码执
    行。但在 ECMAScript 中，0、+0 或 -0 相除会返回 NaN。如果分子是非 0 值，分母是有符号 0 或无符号 0，则会返回 Infinity 或-Infinity。
        ```
        console.log(0/0);       // NaN 
        console.log(-0/+0);     // NaN
        
        console.log(5/0);       // Infinity 
        console.log(5/-0);      // -Infinity
        ```
    - NaN 有几个独特的属性。首先，任何涉及 NaN 的操作始终返回 NaN（如 NaN/10），在连续多步计算时这可能是个问题。其次，NaN 不等于包括 NaN 在内的任何值。例如，下面的比较操作会返回 false：
        ```
        console.log(NaN == NaN); // false
        ```
    - ECMAScript 提供了 isNaN() 函数。该函数接收一个参数，可以是任意数据类型，然后判断这个参数是否“不是数值”。把一个值传给 isNaN()后，该函数会尝试把它转换为数值。某些非数值的值可以直接转换成数值，如字符串"10"或布尔值。任何不能转换为数值的值都会导致这个函数返回 true。
        ```
        console.log(isNaN(NaN));    // true 
        console.log(isNaN(10));     // false，10 是数值
        console.log(isNaN("10"));   // false，可以转换为数值 10 
        console.log(isNaN("blue")); // true，不可以转换为数值
        console.log(isNaN(true));   // false，可以转换为数值 1
        ```
    - 虽然不常见，但 isNaN() 可以用于测试对象。此时，首先会调用对象的 valueOf() 方法，然后再确定返回的值是否可以转换为数值。如果不能，再调用 toString()方法，并测试其返回值。这通常是 ECMAScript 内置函数和操作符的工作方式。

23. 有 3 个函数可以将非数值转换为数值：Number()、parseInt() 和 parseFloat()。Number() 是转型函数，可用于任何数据类型。后两个函数主要用于将字符串转换为数值。对于同样的参数，这 3 个函数执行的操作也不同。

24. Number()函数基于如下规则执行转换。
    - 布尔值，true 转换为 1，false 转换为 0。
    - 数值，直接返回。
    - null，返回 0。
    - undefined，返回 NaN。
    - 字符串，应用以下规则。
        - 如果字符串包含数值字符，包括数值字符前面带加、减号的情况，则转换为一个十进制数值。因此，Number("1")返回 1，Number("123")返回 123，Number("011")返回 11（忽略前面的零）。
        - 如果字符串包含有效的浮点值格式如"1.1"，则会转换为相应的浮点值（同样，忽略前面的零）。
        - 如果字符串包含有效的十六进制格式如"0xf"，则会转换为与该十六进制值对应的十进制整数值。
        - 如果是空字符串（不包含字符），则返回 0。
        - 如果字符串包含除上述情况之外的其他字符，则返回 NaN。
    - 对象，调用 valueOf() 方法，并按照上述规则转换返回的值。如果转换结果是 NaN，则调用 toString()方法，再按照转换字符串的规则转换。
    - 具体例子：
        ```
        let num1 = Number("Hello world!");  // NaN 
        let num2 = Number("");              // 0 
        let num3 = Number("000011");        // 11 
        let num4 = Number(true);            // 1
        ```

25. 考虑到用 Number() 函数转换字符串时相对复杂且有点反常规，通常在需要得到整数时可以优先使用 parseInt() 函数。parseInt() 函数更专注于字符串是否包含数值模式。字符串最前面的空格会被忽略，从第一个非空格字符开始转换。如果第一个字符不是数值字符、加号或减号，parseInt() 立即返回 NaN。这意味着空字符串也会返回 NaN（这一点跟 Number() 不一样，它返回 0）。如果第一个字符是数值字符、加号或减号，则继续依次检测每个字符，直到字符串末尾，或碰到非数值字符。比如，"1234blue"会被转换为 1234，因为"blue"会被完全忽略。类似地，"22.5"会被转换为 22，因为小数点不是有效的整数字符。
    - parseInt()函数也能识别不同的整数格式（十进制、八进制、十六进制）。换句话说，如果字符串以"0x"开头，就会被解释为十六进制整数。如果字符串以"0"开头，且紧跟着数值字符，在非严格模式下会被某些实现解释为八进制整数。
    - 具体例子：
        ```
        let num1 = parseInt("1234blue");    // 1234 
        let num2 = parseInt("");            // NaN 
        let num3 = parseInt("0xA");         // 10，解释为十六进制整数
        let num4 = parseInt(22.5);          // 22 
        let num5 = parseInt("70");          // 70，解释为十进制值
        let num6 = parseInt("0xf");         // 15，解释为十六进制整数
        ```
    - parseInt()也接收第二个参数，用于指定底数（进制数）。
        ```
        let num = parseInt("0xAF", 16); // 175
        
        let num1 = parseInt("AF", 16); // 175 
        let num2 = parseInt("AF"); // NaN，因为检测到第一个字符就是非数值字符，随即自动停止并返回 NaN。
        ```
        - 如果知道要解析的值是十六进制，那么可以传入 16 作为第二个参数，以便正确解析。
        - 如果提供了十六进制参数，那么字符串前面的"0x"可以省掉。
        - 建议始终传给它第二个参数。多数情况下解析的应该都是十进制数，此时第二个参数就要传入 10。

26. parseFloat() 函数的工作方式跟 parseInt() 函数类似，都是从位置 0 开始检测每个字符。同样，它也是解析到字符串末尾或者解析到一个无效的浮点数值字符为止。这意味着第一次出现的小数点是有效的，但第二次出现的小数点就无效了，此时字符串的剩余字符都会被忽略。因此，"22.34.5"将转换成 22.34。
    - parseFloat()函数的另一个不同之处在于，它始终忽略字符串开头的零。这个函数能识别前面讨论的所有浮点格式，以及十进制格式（开头的零始终被忽略）。十六进制数值始终会返回 0。因为 parseFloat()只解析十进制值，因此不能指定底数。最后，如果字符串表示整数（没有小数点或者小数点后面只有一个零），则 parseFloat()返回整数。
    - 具体例子：
        ```
        let num1 = parseFloat("1234blue");  // 1234，按整数解析
        let num2 = parseFloat("0xA");       // 0 
        let num3 = parseFloat("22.5");      // 22.5 
        let num4 = parseFloat("22.34.5");   // 22.34 
        let num5 = parseFloat("0908.5");    // 908.5 
        let num6 = parseFloat("3.125e7");   // 31250000
        ```

27. String（字符串）数据类型表示零或多个 16 位 Unicode 字符序列。字符串可以使用双引号（"）、单引号（'）或反引号（`）标示。
    - 跟某些语言中使用不同的引号会改变对字符串的解释方式不同，ECMAScript 语法中表示字符串的引号没有区别。不过要注意的是，以某种引号作为字符串开头，必须仍然以该种引号作为字符串结尾。
    - 字符串数据类型包含一些字符字面量，用于表示非打印字符或有其他用途的字符。
        字面量 | 含义
        ---|---
        \n | 换行
        \t | 制表
        \b | 退格
        \r | 回车
        \f | 换页
        \\\ | 反斜杠（\）
        \' | 单引号（'），在字符串以单引号标示时使用，例如'He said, \\'hey.\\''
        \" | 双引号（"），在字符串以双引号标示时使用，例如"He said, \\"hey.\\""
        \` | 反引号（\`），在字符串以反引号标示时使用，例如\`He said, \\`hey.\\``
        \xnn | 以十六进制编码 nn 表示的字符（其中 n 是十六进制数字 0~F），例如\x41 等于"A"
        \unnnn | 以十六进制编码 nnnn 表示的 Unicode 字符（其中 n 是十六进制数字 0~F），例如\u03a3 等于希腊字符"Σ"
    - 这些字符字面量可以出现在字符串中的任意位置，且可以作为单个字符被解释：
        ```
        let text = "This is the letter sigma: \u03a3.";
        
        console.log(text.length); // 28
        ```
        - 在这个例子中，即使包含 6 个字符长的转义序列，变量 text 仍然是 28 个字符长。因为转义序列表示一个字符，所以只算一个字符。
        - 字符串的长度可以通过其 length 属性获取。这个属性返回字符串中 16 位字符的个数。
        - 注意，如果字符串中包含双字节字符，那么 length 属性返回的值可能不是准确的字符数。（读者注：为啥不准确？）

28. ECMAScript 中的字符串是不可变的（immutable），意思是一旦创建，它们的值就不能变了。要修改某个变量中的字符串值，必须先销毁原始的字符串，然后将包含新值的另一个字符串保存到该变量。
    ```
    let lang = "Java"; 
    lang = lang + "Script";
    ```
    - 整个过程首先会分配一个足够容纳 10 个字符的空间，然后填充上"Java"和"Script"。最后销毁原始的字符串"Java"和字符串"Script"，因为这两个字符串都没有用了。所有处理都是在后台发生的，而这也是一些早期的浏览器（如 Firefox 1.0 之前的版本和 IE6.0）在拼接字符串时非常慢的原因。这些浏览器在后来的版本中都有针对性地解决了这个问题。

29. 有两种方式把一个值转换为字符串。
    - 首先是使用几乎所有值都有的 toString()方法。这个方法唯一的用途就是返回当前值的字符串等价物。
        ```
        let age = 11; 
        let ageAsString = age.toString();       // 字符串"11" 
        let found = true; 
        let foundAsString = found.toString();   // 字符串"true"
        ```
    - toString()方法可见于数值、布尔值、对象和字符串值。（没错，字符串值也有 toString()方法，该方法只是简单地返回自身的一个副本。）null 和 undefined 值没有 toString() 方法。
    - 多数情况下，toString()不接收任何参数。不过，在对数值调用这个方法时，toString()可以接收一个底数参数，即以什么底数来输出数值的字符串表示。默认情况下，toString()返回数值的十进制字符串表示。而通过传入参数，可以得到数值的二进制、八进制、十六进制，或者其他任何有效基数的字符串表示。默认情况下（不传参数）的输出与传入参数 10 得到的结果相同。
        ```
        let num = 10; 
        console.log(num.toString());    // "10" 
        console.log(num.toString(2));   // "1010" 
        console.log(num.toString(8));   // "12" 
        console.log(num.toString(10));  // "10" 
        console.log(num.toString(16));  // "a"
        ```
    - **如果你不确定一个值是不是 null 或 undefined，可以使用 String()转型函数，它始终会返回表示相应类型值的字符串。**String()函数遵循如下规则。
        - 如果值有 toString()方法，则调用该方法（不传参数）并返回结果。
        - 如果值是 null，返回"null"。
        - 如果值是 undefined，返回"undefined"。
        - 具体例子：
            ```
            let value1 = 10; 
            let value2 = true; 
            let value3 = null; 
            let value4; 
            console.log(String(value1)); // "10" 
            console.log(String(value2)); // "true" 
            console.log(String(value3)); // "null" 
            console.log(String(value4)); // "undefined"
            ```
            - 数值和布尔值的转换结果与调用 toString()相同。因为 null 和 undefined 没有 toString()方法，所以 String()方法就直接返回了这两个值的字面量文本。
    - 注意，用加号操作符给一个值加上一个空字符串""也可以将其转换为字符串。

30. ECMAScript 6 新增了使用模板字面量定义字符串的能力。
    - 与使用单引号或双引号不同，模板字面量保留换行字符，可以跨行定义字符串。
        ```
        let myMultiLineString = 'first line\nsecond line'; 
        let myMultiLineTemplateLiteral = `first line 
        second line`; 
        console.log(myMultiLineString); 
        // first line 
        // second line" 
        console.log(myMultiLineTemplateLiteral); 
        // first line
        // second line 
        console.log(myMultiLineString === myMultiLinetemplateLiteral); // true
        ```
    - 由于模板字面量会保持反引号内部的空格，因此在使用时要格外注意。格式正确的模板字符串看起来可能会缩进不当：
        ```
        // 这个模板字面量在换行符之后有 25 个空格符
        let myTemplateLiteral = `first line 
                                 second line`; 
        console.log(myTemplateLiteral.length); // 47
        
        // 这个模板字面量以一个换行符开头
        let secondTemplateLiteral = ` 
        first line 
        second line`; 
        console.log(secondTemplateLiteral[0] === '\n'); // true
        ```
    - 模板字面量在定义模板时特别有用，比如下面这个 HTML 模板：
        ```
        let pageHTML = ` 
        <div> 
          <a href="#"> 
            <span>Jake</span> 
          </a> 
        </div>`;
        ```

31. 模板字面量最常用的一个特性是支持字符串插值，也就是可以在一个连续定义中插入一个或多个值。技术上讲，模板字面量不是字符串，而是一种特殊的 JavaScript 句法表达式，只不过求值后得到的是字符串。模板字面量在定义时立即求值并转换为字符串实例，任何插入的变量也会从它们最接近的作用域中取值。
    - 字符串插值通过在${}中使用一个 JavaScript 表达式实现：
        ```
        let value = 5; 
        let exponent = 'second'; 
        // 以前，字符串插值是这样实现的：
        let interpolatedString = 
          value + ' to the ' + exponent + ' power is ' + (value * value); 
        
        // 现在，可以用模板字面量这样实现：
        let interpolatedTemplateLiteral = 
          `${ value } to the ${ exponent } power is ${ value * value }`; 
        console.log(interpolatedString); // 5 to the second power is 25 
        console.log(interpolatedTemplateLiteral); // 5 to the second power is 25
        ```
    - 所有插入的值都会使用 toString()强制转型为字符串，而且任何 JavaScript 表达式都可以用于插值。嵌套的模板字符串无须转义：
        ```
        console.log(`Hello, ${ `World` }!`); // Hello, World!
        ```
    - 将表达式转换为字符串时会调用 toString()：
        ```
        let foo = { toString: () => 'World' }; 
        console.log(`Hello, ${ foo }!`); // Hello, World!
        ```
    - 在插值表达式中可以调用函数和方法：
        ```
        function capitalize(word) { 
            return `${ word[0].toUpperCase() }${ word.slice(1) }`; 
        } 
        console.log(`${ capitalize('hello') }, ${ capitalize('world') }!`); // Hello, World!
        ```
    - 模板也可以插入自己之前的值：
        ```
        let value = ''; 
        function append() { 
            value = `${value}abc` 
            console.log(value); 
        } 
        append(); // abc 
        append(); // abcabc 
        append(); // abcabcabc
        ```

32. 模板字面量也支持定义标签函数（tag function），而通过标签函数可以自定义插值行为。标签函数会接收被插值记号分隔后的模板和对每个表达式求值的结果。标签函数本身是一个常规函数，通过前缀到模板字面量来应用自定义行为。标签函数接收到的参数依次是原始字符串数组和对每个表达式求值的结果。这个函数的返回值是对模板字面量求值得到的字符串。
    ```
    let a = 6; 
    let b = 9; 
    function simpleTag(strings, aValExpression, bValExpression, sumExpression) { 
        console.log(strings); 
        console.log(aValExpression); 
        console.log(bValExpression); 
        console.log(sumExpression); 
        return 'foobar'; 
    } 
    let untaggedResult = `${ a } + ${ b } = ${ a + b }`; 
    let taggedResult = simpleTag`${ a } + ${ b } = ${ a + b }`; 
    // ["", " + ", " = ", ""] 
    // 6 
    // 9 
    // 15 
    console.log(untaggedResult); // "6 + 9 = 15" 
    console.log(taggedResult); // "foobar"
    ```
    - 因为表达式参数的数量是可变的，所以通常应该使用剩余操作符（rest operator）将它们收集到一个数组中：
        ```
        let a = 6; 
        let b = 9; 
        function simpleTag(strings, ...expressions) { 
            console.log(strings); 
            for(const expression of expressions) { 
                console.log(expression); 
            } 
            return 'foobar'; 
        } 
        let taggedResult = simpleTag`${ a } + ${ b } = ${ a + b }`; 
        // ["", " + ", " = ", ""] 
        // 6 
        // 9 
        // 15 
        console.log(taggedResult); // "foobar"
        ```
    - 对于有 n 个插值的模板字面量，传给标签函数的表达式参数的个数始终是 n，而传给标签函数的第一个参数所包含的字符串个数则始终是 n+1。因此，如果你想把这些字符串和对表达式求值的结果拼接起来作为默认返回的字符串，可以这样做：
        ```
        let a = 6; 
        let b = 9; 
        function zipTag(strings, ...expressions) { 
            return strings[0] + 
                   expressions.map((e, i) => `${e}${strings[i + 1]}`) 
                              .join(''); 
        } 
        let untaggedResult = `${ a } + ${ b } = ${ a + b }`; 
        let taggedResult = zipTag`${ a } + ${ b } = ${ a + b }`; 
        console.log(untaggedResult); // "6 + 9 = 15" 
        console.log(taggedResult);   // "6 + 9 = 15"
        ```

33. 使用模板字面量也可以直接获取原始的模板字面量内容（如换行符或 Unicode 字符），而不是被转换后的字符表示。
    - 为此，可以使用默认的 String.raw 标签函数：
        ```
        // Unicode 示例
        // \u00A9 是版权符号
        console.log(`\u00A9`);              // © 
        console.log(String.raw`\u00A9`);    // \u00A9 
        
        // 换行符示例
        console.log(`first line\nsecond line`); 
        // first line 
        // second line 
        console.log(String.raw`first line\nsecond line`); // "first line\nsecond line" 
        
        // 对实际的换行符来说是不行的
        // 它们不会被转换成转义序列的形式
        console.log(`first line
        second line`); 
        // first line 
        // second line 
        
        console.log(String.raw`first line 
        second line`); 
        // first line 
        // second line
        ```
    - 另外，也可以通过标签函数的第一个参数，即字符串数组的.raw 属性取得每个字符串的原始内容：
        ```
        function printRaw(strings) { 
            console.log('Actual characters:'); 
            for (const string of strings) { 
                console.log(string); 
            } 
            console.log('Escaped characters;'); 
            for (const rawString of strings.raw) { 
                console.log(rawString); 
            } 
        } 
        
        printRaw`\u00A9${ 'and' }\n`; 
        // Actual characters: 
        // © 
        //（换行符）
        // Escaped characters: 
        // \u00A9 
        // \n
        ```

34. Symbol（符号）是 ECMAScript 6 新增的数据类型。符号是原始值，且符号实例是唯一、不可变的。符号的用途是确保对象属性使用唯一标识符，不会发生属性冲突的危险。
    - 尽管听起来跟私有属性有点类似，但符号并不是为了提供私有属性的行为才增加的（尤其是因为 Object API 提供了方法，可以更方便地发现符号属性）。相反，符号就是用来创建唯一记号，进而用作非字符串形式的对象属性。

35. 符号的基本用法
    - 符号需要使用 Symbol() 函数初始化。因为符号本身是原始类型，所以 typeof 操作符对符号返回 symbol。
        ```
        let sym = Symbol(); 
        console.log(typeof sym); // symbol
        ```
    - 调用 Symbol() 函数时，也可以传入一个字符串参数作为对符号的描述（description），将来可以通过这个字符串来调试代码。但是，这个字符串参数与符号定义或标识完全无关：
        ```
        let genericSymbol = Symbol(); 
        let otherGenericSymbol = Symbol(); 
        let fooSymbol = Symbol('foo'); 
        let otherFooSymbol = Symbol('foo'); 
        console.log(genericSymbol == otherGenericSymbol);   // false
        console.log(fooSymbol == otherFooSymbol);           // false
        ```
    - 符号没有字面量语法，这也是它们发挥作用的关键。按照规范，你只要创建 Symbol() 实例并将其用作对象的新属性，就可以保证它不会覆盖已有的对象属性，无论是符号属性还是字符串属性。
        ```
        let genericSymbol = Symbol(); 
        console.log(genericSymbol); // Symbol() 
        let fooSymbol = Symbol('foo'); 
        console.log(fooSymbol);     // Symbol(foo);
        ```
    - **Symbol()函数不能与 new 关键字一起作为构造函数使用。**这样做是为了避免创建符号包装对象，像使用 Boolean、String 或 Number 那样，它们都支持构造函数且可用于初始化包含原始值的包装对象：
        ```
        let myBoolean = new Boolean(); 
        console.log(typeof myBoolean);  // "object" 
        let myString = new String(); 
        console.log(typeof myString);   // "object" 
        let myNumber = new Number(); 
        console.log(typeof myNumber);   // "object" 
        let mySymbol = new Symbol();    // TypeError: Symbol is not a constructor
        ```
    - 如果你确实想使用符号包装对象，可以借用 Object() 函数：
        ```
        let mySymbol = Symbol(); 
        let myWrappedSymbol = Object(mySymbol); 
        console.log(typeof myWrappedSymbol); // "object"
        ```

36. 使用全局符号注册表
    - 如果运行时的不同部分需要共享和重用符号实例，那么可以用一个字符串作为键，在全局符号注册表中创建并重用符号。为此，需要使用 Symbol.for()方法：
        ```
        let fooGlobalSymbol = Symbol.for('foo'); 
        console.log(typeof fooGlobalSymbol);    // symbol
        ```
    - Symbol.for()对每个字符串键都执行幂等操作。第一次使用某个字符串调用时，它会检查全局运行时注册表，发现不存在对应的符号，于是就会生成一个新符号实例并添加到注册表中。后续使用相同字符串的调用同样会检查注册表，发现存在与该字符串对应的符号，然后就会返回该符号实例。
        ```
        let fooGlobalSymbol = Symbol.for('foo'); // 创建新符号
        let otherFooGlobalSymbol = Symbol.for('foo'); // 重用已有符号
        console.log(fooGlobalSymbol === otherFooGlobalSymbol); // true
        ```
    - 即使采用相同的符号描述，在全局注册表中定义的符号跟使用 Symbol()定义的符号也并不等同：
        ```
        let localSymbol = Symbol('foo'); 
        let globalSymbol = Symbol.for('foo'); 
        console.log(localSymbol === globalSymbol); // false
        ```
    - 全局注册表中的符号必须使用字符串键来创建，因此作为参数传给 Symbol.for()的任何值都会被转换为字符串。此外，注册表中使用的键同时也会被用作符号描述。
        ```
        let emptyGlobalSymbol = Symbol.for(); 
        console.log(emptyGlobalSymbol);     // Symbol(undefined)
        ```
    - 还可以使用 Symbol.keyFor() 来查询全局注册表，这个方法接收符号，返回该全局符号对应的字符串键。如果查询的不是全局符号，则返回 undefined。
        ```
        // 创建全局符号
        let s = Symbol.for('foo'); 
        console.log(Symbol.keyFor(s)); // foo 
        // 创建普通符号
        let s2 = Symbol('bar'); 
        console.log(Symbol.keyFor(s2)); // undefined
        ```
    - 如果传给 Symbol.keyFor() 的不是符号，则该方法抛出 TypeError：
        ```
        Symbol.keyFor(123); // TypeError: 123 is not a symbol
        ```

37. 凡是可以使用字符串或数值作为属性的地方，都可以使用符号。这就包括了对象字面量属性和 Object.defineProperty()/Object.defineProperties() 定义的属性。
    - 对象字面量只能在计算属性语法中使用符号作为属性。
        ```
        let s1 = Symbol('foo'), 
            s2 = Symbol('bar'), 
            s3 = Symbol('baz'), 
            s4 = Symbol('qux'); 
        let o = { 
            [s1]: 'foo val' 
        }; 
        // 这样也可以：o[s1] = 'foo val'; 
        
        console.log(o); 
        // {Symbol(foo): foo val} 
        
        Object.defineProperty(o, s2, {value: 'bar val'}); 
        
        console.log(o); 
        // {Symbol(foo): foo val, Symbol(bar): bar val} 
        
        Object.defineProperties(o, { 
            [s3]: {value: 'baz val'}, 
            [s4]: {value: 'qux val'} 
        }); 
        
        console.log(o); 
        // {Symbol(foo): foo val, Symbol(bar): bar val, 
        // Symbol(baz): baz val, Symbol(qux): qux val}
        ```
    - 类似于 Object.getOwnPropertyNames()返回对象实例的常规属性数组，Object.getOwnPropertySymbols()返回对象实例的符号属性数组。这两个方法的返回值彼此互斥。Object.getOwnPropertyDescriptors()会返回同时包含常规和符号属性描述符的对象。Reflect.ownKeys()会返回两种类型的键： 
        ```
        let s1 = Symbol('foo'), 
            s2 = Symbol('bar'); 
        let o = { 
            [s1]: 'foo val', 
            [s2]: 'bar val', 
            baz: 'baz val', 
            qux: 'qux val' 
        };
        
        console.log(Object.getOwnPropertySymbols(o)); 
        // [Symbol(foo), Symbol(bar)] 
        
        console.log(Object.getOwnPropertyNames(o)); 
        // ["baz", "qux"] 
        
        console.log(Object.getOwnPropertyDescriptors(o)); 
        // {baz: {...}, qux: {...}, Symbol(foo): {...}, Symbol(bar): {...}} 
        console.log(Reflect.ownKeys(o)); 
        // ["baz", "qux", Symbol(foo), Symbol(bar)]
        ```
    - 因为符号属性是对内存中符号的一个引用，所以直接创建并用作属性的符号不会丢失。但是，如果没有显式地保存对这些属性的引用，那么必须遍历对象的所有符号属性才能找到相应的属性键：
        ```
        let o = { 
            [Symbol('foo')]: 'foo val', 
            [Symbol('bar')]: 'bar val' 
        }; 
        
        console.log(o); 
        // {Symbol(foo): "foo val", Symbol(bar): "bar val"} 
        
        let barSymbol = Object.getOwnPropertySymbols(o) 
                        .find((symbol) => symbol.toString().match(/bar/));
                    
        console.log(barSymbol); 
        // Symbol(bar)
        ```

38. 常用内置符号
    - ECMAScript 6 也引入了一批常用内置符号（well-known symbol），用于暴露语言内部行为，开发者可以直接访问、重写或模拟这些行为。这些内置符号都以 Symbol 工厂函数字符串属性的形式存在。
    - 这些内置符号最重要的用途之一是重新定义它们，从而改变原生结构的行为。比如，我们知道 for-of 循环会在相关对象上使用 Symbol.iterator 属性，那么就可以通过在自定义对象上重新定义 Symbol.iterator 的值，来改变 for-of 在迭代该对象时的行为。
    - 这些内置符号也没有什么特别之处，它们就是全局函数 Symbol 的普通字符串属性，指向一个符号的实例。所有内置符号属性都是不可写、不可枚举、不可配置的。
    - 注意 在提到 ECMAScript 规范时，经常会引用符号在规范中的名称，前缀为@@。比如，@@iterator 指的就是 Symbol.iterator。

39. Symbol.asyncIterator
    - 根据 ECMAScript 规范，这个符号作为一个属性表示“一个方法，该方法返回对象默认的 AsyncIterator。由 for-await-of 语句使用”。换句话说，这个符号表示实现异步迭代器 API 的函数。
    - for-await-of 循环会利用这个函数执行异步迭代操作。循环时，它们会调用以 Symbol.asyncIterator 为键的函数，并期望这个函数会返回一个实现迭代器 API 的对象。很多时候，返回的对象是实现该 API的 AsyncGenerator：
        ```
        class Foo { 
            async *[Symbol.asyncIterator]() {} 
        } 
        let f = new Foo(); 
        console.log(f[Symbol.asyncIterator]()); 
        // AsyncGenerator {<suspended>} 
        ```
    - 技术上，这个由 Symbol.asyncIterator 函数生成的对象应该通过其 next()方法陆续返回 Promise 实例。可以通过显式地调用 next()方法返回，也可以隐式地通过异步生成器函数返回：
        ```
        class Emitter { 
            constructor(max) { 
                this.max = max; 
                this.asyncIdx = 0; 
            } 
         
            async *[Symbol.asyncIterator]() { 
                while(this.asyncIdx < this.max) { 
                    yield new Promise((resolve) => resolve(this.asyncIdx++)); 
                } 
            } 
        }
        
        async function asyncCount() { 
            let emitter = new Emitter(5); 
            for await(const x of emitter) { 
                console.log(x); 
            } 
        } 
        
        asyncCount(); 
        // 0 
        // 1 
        // 2 
        // 3 
        // 4 
        ```
    - Symbol.asyncIterator 是 ES2018 规范定义的，因此只有版本非常新的浏览器支持它。
    - 读者备注：（来自GPT3.5）JavaScript的yield关键字用于定义一个generator函数，并指定生成器的返回值。yield关键字在生成器内部使用，用于暂停生成器的执行，并将一个值返回给生成器的调用者。

40. Symbol.hasInstance
    - 根据 ECMAScript 规范，这个符号作为一个属性表示“一个方法，该方法决定一个构造器对象是否认可一个对象是它的实例。instanceof 操作符使用”。instanceof 操作符可以用来确定一个对象实例的原型链上是否有原型。instanceof 的典型使用场景如下：
        ```
        function Foo() {} 
        let f = new Foo(); 
        console.log(f instanceof Foo); // true 
        
        class Bar {} 
        let b = new Bar(); 
        console.log(b instanceof Bar); // true
        ```
    - 在 ES6 中，instanceof 操作符会使用 Symbol.hasInstance 函数来确定关系。以 Symbol.hasInstance 为键的函数会执行同样的操作，只是操作数对调了一下：
        ```
        function Foo() {} 
        let f = new Foo(); 
        console.log(Foo[Symbol.hasInstance](f)); // true 
        
        class Bar {} 
        let b = new Bar(); 
        console.log(Bar[Symbol.hasInstance](b)); // true
        ```
        - 这个属性定义在 Function 的原型上，因此默认在所有函数和类上都可以调用。
    - 由于 instanceof 操作符会在原型链上寻找这个属性定义，就跟在原型链上寻找其他属性一样，因此可以在继承的类上通过静态方法重新定义这个函数：
        ```
        class Bar {} 
        class Baz extends Bar { 
            static [Symbol.hasInstance]() { 
                return false; 
            } 
        } 
        let b = new Baz(); 
        console.log(Bar[Symbol.hasInstance](b));    // true 
        console.log(b instanceof Bar);              // true 
        console.log(Baz[Symbol.hasInstance](b));    // false，因为Baz的方法返回了false
        console.log(b instanceof Baz);              // false，因为Baz的方法返回了false
        ```

41. Symbol.isConcatSpreadable
    - 根据 ECMAScript 规范，这个符号作为一个属性表示“一个布尔值，如果是 true，则意味着对象应该用 Array.prototype.concat() 打平其数组元素”。ES6 中的 Array.prototype.concat() 方法会根据接收到的对象类型选择如何将一个类数组对象拼接成数组实例。覆盖 Symbol.isConcatSpreadable 的值可以修改这个行为。
    - 数组对象默认情况下会被打平到已有的数组，false 或假值会导致整个对象被追加到数组末尾。类数组对象默认情况下会被追加到数组末尾，true 或真值会导致这个类数组对象被打平到数组实例。其他不是类数组对象的对象在 Symbol.isConcatSpreadable 被设置为 true 的情况下将被忽略。
        ```
        let initial = ['foo']; 
        
        let array = ['bar']; 
        console.log(array[Symbol.isConcatSpreadable]);  // undefined 
        console.log(initial.concat(array));             // ['foo', 'bar'] 
        array[Symbol.isConcatSpreadable] = false; 
        console.log(initial.concat(array));             // ['foo', Array(1)]
        
        let arrayLikeObject = { length: 1, 0: 'baz' }; 
        console.log(arrayLikeObject[Symbol.isConcatSpreadable]);    // undefined 
        console.log(initial.concat(arrayLikeObject));               // ['foo', {...}] 
        arrayLikeObject[Symbol.isConcatSpreadable] = true; 
        console.log(initial.concat(arrayLikeObject));               // ['foo', 'baz'] 
        
        let otherObject = new Set().add('qux'); 
        console.log(otherObject[Symbol.isConcatSpreadable]);    // undefined 
        console.log(initial.concat(otherObject));               // ['foo', Set(1)] 
        otherObject[Symbol.isConcatSpreadable] = true; 
        console.log(initial.concat(otherObject));               // ['foo']
        ```

42. Symbol.iterator
    - 根据 ECMAScript 规范，这个符号作为一个属性表示“一个方法，该方法返回对象默认的迭代器。由 for-of 语句使用”。换句话说，这个符号表示实现迭代器 API 的函数。
    - for-of 循环这样的语言结构会利用这个函数执行迭代操作。循环时，它们会调用以 Symbol.iterator 为键的函数，并默认这个函数会返回一个实现迭代器 API 的对象。很多时候，返回的对象是实现该 API 的 Generator：
        ```
        class Foo { 
            *[Symbol.iterator]() {} 
        } 
        let f = new Foo(); 
        console.log(f[Symbol.iterator]()); 
        // Generator {<suspended>}
        ```
    - 技术上，这个由 Symbol.iterator 函数生成的对象应该通过其 next()方法陆续返回值。可以通过显式地调用 next()方法返回，也可以隐式地通过生成器函数返回：
        ```
        class Emitter { 
            constructor(max) { 
                this.max = max; 
                this.idx = 0; 
            } 
            
            *[Symbol.iterator]() { 
                while(this.idx < this.max) { 
                    yield this.idx++; 
                }
            } 
        }
        
        function count() { 
            let emitter = new Emitter(5); 
            for (const x of emitter) { 
                console.log(x); 
            } 
        }
        count(); 
        // 0
        // 1 
        // 2 
        // 3 
        // 4
        ```

43. Symbol.match
    - 根据 ECMAScript 规范，这个符号作为一个属性表示“一个正则表达式方法，该方法用正则表达式去匹配字符串。由 String.prototype.match()方法使用”。String.prototype.match()方法会使用以 Symbol.match 为键的函数来对正则表达式求值。正则表达式的原型上默认有这个函数的定义，因此所有正则表达式实例默认是这个 String 方法的有效参数：
        ```
        console.log(RegExp.prototype[Symbol.match]); 
        // ƒ [Symbol.match]() { [native code] } 
        console.log('foobar'.match(/bar/)); 
        // ["bar", index: 3, input: "foobar", groups: undefined]
        ```
    - 给这个方法传入非正则表达式值会导致该值被转换为 RegExp 对象。如果想改变这种行为，让方法直接使用参数，则可以重新定义 Symbol.match 函数以取代默认对正则表达式求值的行为，从而让match()方法使用非正则表达式实例。Symbol.match 函数接收一个参数，就是调用 match()方法的字符串实例。返回的值没有限制：
        ```
        class FooMatcher { 
            static [Symbol.match](target) { 
                return target.includes('foo'); 
            } 
        }
        
        console.log('foobar'.match(FooMatcher)); // true 
        console.log('barbaz'.match(FooMatcher)); // false 
        
        class StringMatcher { 
            constructor(str) { 
                this.str = str; 
            } 
            
            [Symbol.match](target) { 
                return target.includes(this.str); 
            } 
        }
        
        console.log('foobar'.match(new StringMatcher('foo')));  // true 
        console.log('barbaz'.match(new StringMatcher('qux')));  // false
        ```

44. Symbol.replace
    - 根据 ECMAScript 规范，这个符号作为一个属性表示“一个正则表达式方法，该方法替换一个字符串中匹配的子串。由 String.prototype.replace()方法使用”。String.prototype.replace()方法会使用以 Symbol.replace 为键的函数来对正则表达式求值。正则表达式的原型上默认有这个函数的定义，因此所有正则表达式实例默认是这个 String 方法的有效参数：
        ```
        console.log(RegExp.prototype[Symbol.replace]); 
        // ƒ [Symbol.replace]() { [native code] } 
        console.log('foobarbaz'.replace(/bar/, 'qux')); 
        // 'fooquxbaz'
        ```
    - 给这个方法传入非正则表达式值会导致该值被转换为 RegExp 对象。如果想改变这种行为，让方法直接使用参数，可以重新定义 Symbol.replace 函数以取代默认对正则表达式求值的行为，从而让replace()方法使用非正则表达式实例。Symbol.replace 函数接收两个参数，即调用 replace()方法的字符串实例和替换字符串。返回的值没有限制：
        ```
        class FooReplacer { 
            static [Symbol.replace](target, replacement) { 
                return target.split('foo').join(replacement); 
            } 
        } 
        
        console.log('barfoobaz'.replace(FooReplacer, 'qux')); 
        // "barquxbaz" 
        
        class StringReplacer { 
            constructor(str) { 
                this.str = str; 
            } 
            
            [Symbol.replace](target, replacement) { 
                return target.split(this.str).join(replacement); 
            } 
        }
        
        console.log('barfoobaz'.replace(new StringReplacer('foo'), 'qux')); 
        // "barquxbaz" 
        ```

45. Symbol.search
    - 根据 ECMAScript 规范，这个符号作为一个属性表示“一个正则表达式方法，该方法返回字符串中匹配正则表达式的索引。由 String.prototype.search()方法使用”。String.prototype.search() 方法会使用以 Symbol.search 为键的函数来对正则表达式求值。正则表达式的原型上默认有这个函数的定义，因此所有正则表达式实例默认是这个 String 方法的有效参数：
        ```
        console.log(RegExp.prototype[Symbol.search]); 
        // ƒ [Symbol.search]() { [native code] } 
        
        console.log('foobar'.search(/bar/)); 
        // 3
        ```
    - 给这个方法传入非正则表达式值会导致该值被转换为 RegExp 对象。如果想改变这种行为，让方法直接使用参数，可以重新定义 Symbol.search 函数以取代默认对正则表达式求值的行为，从而让search()方法使用非正则表达式实例。Symbol.search 函数接收一个参数，就是调用 match()方法的字符串实例。返回的值没有限制：
        ```
        class FooSearcher { 
            static [Symbol.search](target) { 
                return target.indexOf('foo'); 
            } 
        } 
    
        console.log('foobar'.search(FooSearcher)); // 0 
        console.log('barfoo'.search(FooSearcher)); // 3 
        console.log('barbaz'.search(FooSearcher)); // -1 
        
        class StringSearcher { 
            constructor(str) { 
                this.str = str; 
            } 
         
            [Symbol.search](target) { 
                return target.indexOf(this.str); 
            } 
        } 
        
        console.log('foobar'.search(new StringSearcher('foo'))); // 0 
        console.log('barfoo'.search(new StringSearcher('foo'))); // 3 
        console.log('barbaz'.search(new StringSearcher('qux'))); // -1 
        ```

46. Symbol.species
    - 根据 ECMAScript 规范，这个符号作为一个属性表示“一个函数值，该函数作为创建派生对象的构造函数”。这个属性在内置类型中最常用，用于对内置类型实例方法的返回值暴露实例化派生对象的方法。用 Symbol.species 定义静态的获取器（getter）方法，可以覆盖新创建实例的原型定义：
        ```
        class Bar extends Array {} 
        class Baz extends Array { 
            static get [Symbol.species]() { 
                return Array; 
            } 
        } 
        
        let bar = new Bar(); 
        console.log(bar instanceof Array);  // true 
        console.log(bar instanceof Bar);    // true 
        bar = bar.concat('bar'); 
        console.log(bar instanceof Array);  // true 
        console.log(bar instanceof Bar);    // true 
        
        let baz = new Baz(); 
        console.log(baz instanceof Array);  // true 
        console.log(baz instanceof Baz);    // true 
        baz = baz.concat('baz');            // concat 之后返回了一个新的 Array 实例，而不是 Baz 实例。这是因为在派生类中覆盖了 Symbol.species 属性，指定了新实例的构造函数为 Array。 
        console.log(baz instanceof Array);  // true 
        console.log(baz instanceof Baz);    // false 
        ```
        - 读者注：concat() 方法用于连接两个或多个数组。

47. Symbol.split
    - 根据 ECMAScript 规范，这个符号作为一个属性表示“一个正则表达式方法，该方法在匹配正则表达式的索引位置拆分字符串。由 String.prototype.split()方法使用”。String.prototype.split()方法会使用以 Symbol.split 为键的函数来对正则表达式求值。正则表达式的原型上默认有这个函数的定义，因此所有正则表达式实例默认是这个 String 方法的有效参数：
        ```
        console.log(RegExp.prototype[Symbol.split]); 
        // ƒ [Symbol.split]() { [native code] } 
        
        console.log('foobarbaz'.split(/bar/)); 
        // ['foo', 'baz'] 
        ```
    - 给这个方法传入非正则表达式值会导致该值被转换为 RegExp 对象。如果想改变这种行为，让方法直接使用参数，可以重新定义 Symbol.split 函数以取代默认对正则表达式求值的行为，从而让 split() 方法使用非正则表达式实例。Symbol.split 函数接收一个参数，就是调用 match()方法的字符串实
    例。返回的值没有限制：
        ```
        class FooSplitter { 
            static [Symbol.split](target) { 
                return target.split('foo'); 
            } 
        } 
        
        console.log('barfoobaz'.split(FooSplitter)); 
        // ["bar", "baz"] 
        
        class StringSplitter { 
            constructor(str) { 
                this.str = str; 
            } 
            
            [Symbol.split](target) { 
                return target.split(this.str); 
            } 
        } 
        
        console.log('barfoobaz'.split(new StringSplitter('foo'))); 
        // ["bar", "baz"] 
        ```

48. Symbol.toPrimitive
    - 根据 ECMAScript 规范，这个符号作为一个属性表示“一个方法，该方法将对象转换为相应的原始值。由 ToPrimitive 抽象操作使用”。很多内置操作都会尝试强制将对象转换为原始值，包括字符串、数值和未指定的原始类型。对于一个自定义对象实例，通过在这个实例的 Symbol.toPrimitive 属性上定义一个函数可以改变默认行为。
    - 根据提供给这个函数的参数（string、number 或 default），可以控制返回的原始值：
        ```
        class Foo {} 
        let foo = new Foo(); 
        
        console.log(3 + foo);       // "3[object Object]" 
        console.log(3 - foo);       // NaN 
        console.log(String(foo));   // "[object Object]" 
        
        class Bar { 
            constructor() { 
                this[Symbol.toPrimitive] = function(hint) { 
                    switch (hint) { 
                        case 'number': 
                            return 3; 
                        case 'string': 
                            return 'string bar'; 
                        case 'default': 
                        default: 
                            return 'default bar'; 
                    } 
                } 
            } 
        } 
        let bar = new Bar(); 
        console.log(3 + bar);       // "3default bar" 
        console.log(3 - bar);       // 0 
        console.log(String(bar));   // "string bar"
        ```

49. Symbol.toStringTag
    - 根据 ECMAScript 规范，这个符号作为一个属性表示“一个字符串，该字符串用于创建对象的默认字符串描述。由内置方法 Object.prototype.toString()使用”。
    - 通过 toString()方法获取对象标识时，会检索由 Symbol.toStringTag 指定的实例标识符，默认为"Object"。内置类型已经指定了这个值，但自定义类实例还需要明确定义：
        ```
        let s = new Set(); 
        
        console.log(s);                     // Set(0) {} 
        console.log(s.toString());          // [object Set] 
        console.log(s[Symbol.toStringTag]); // Set 
        
        class Foo {} 
        let foo = new Foo(); 
        
        console.log(foo);                       // Foo {} 
        console.log(foo.toString());            // [object Object] 
        console.log(foo[Symbol.toStringTag]);   // undefined 
        
        class Bar { 
            constructor() { 
                this[Symbol.toStringTag] = 'Bar'; 
            } 
        } 
        let bar = new Bar(); 
        
        console.log(bar);                       // Bar {} 
        console.log(bar.toString());            // [object Bar] 
        console.log(bar[Symbol.toStringTag]);   // Bar 
        ```

50. Symbol.unscopables
    - 根据 ECMAScript 规范，这个符号作为一个属性表示“一个对象，该对象所有的以及继承的属性，都会从关联对象的 with 环境绑定中排除”。设置这个符号并让其映射对应属性的键值为 true，就可以阻止该属性出现在 with 环境绑定中，如下例所示：
        ```
        let o = { foo: 'bar' }; 
        with (o) { 
            console.log(foo); // bar 
        }
        
        o[Symbol.unscopables] = { 
            foo: true 
        }; 
        
        with (o) { 
            console.log(foo); // ReferenceError 
        }
        ```
    - **不推荐使用 with，因此也不推荐使用 Symbol.unscopables。**

51. Object 类型
    - ECMAScript 中的对象其实就是一组数据和功能的集合。对象通过 new 操作符后跟对象类型的名称来创建。开发者可以通过创建 Object 类型的实例来创建自己的对象，然后再给对象添加属性和方法。
    - ECMAScript 只要求在给构造函数提供参数时使用括号。如果没有参数，那么完全可以省略括号（不推荐）：
        ```
        let o = new Object();
        let o = new Object; // 合法，但不推荐
        ```
    - Object 的实例本身并不是很有用，但理解与它相关的概念非常重要。类似 Java 中的 java.lang.Object，ECMAScript 中的 Object 也是派生其他对象的基类。Object 类型的所有属性和方法在派生的对象上同样存在。
    - 每个 Object 实例都有如下属性和方法：
        - constructor：用于创建当前对象的函数。在前面的例子中，这个属性的值就是 Object() 函数。
        - hasOwnProperty(propertyName)：用于判断当前对象实例（不是原型）上是否存在给定的属性。要检查的属性名必须是字符串（如 o.hasOwnProperty("name")）或符号。
        - isPrototypeOf(object)：用于判断当前对象是否为另一个对象的原型。
        - propertyIsEnumerable(propertyName)：用于判断给定的属性是否可以使用 for-in 语句枚举。与 hasOwnProperty()一样，属性名必须是字符串。
        - toLocaleString()：返回对象的字符串表示，该字符串反映对象所在的本地化执行环境。
        - toString()：返回对象的字符串表示。
        - valueOf()：返回对象对应的字符串、数值或布尔值表示。通常与 toString()的返回值相同。
    - 因为在 ECMAScript 中 Object 是所有对象的基类，所以任何对象都有这些属性和方法。
    - 注意，严格来讲，ECMA-262 中对象的行为不一定适合 JavaScript 中的其他对象。比如浏览器环境中的 BOM 和 DOM 对象，都是由宿主环境定义和提供的宿主对象。而宿主对象不受 ECMA-262 约束，所以它们可能会也可能不会继承 Object。

52. ECMAScript 中的操作符是独特的，因为它们可用于各种值，包括字符串、数值、布尔值，甚至还有对象。在应用给对象时，操作符通常会调用 valueOf()和/或 toString()方法来取得可以计算的值。

53. 只操作一个值的操作符叫一元操作符（unary operator）。

54. 递增/递减操作符
    - 递增和递减操作符直接照搬自 C 语言，但有两个版本：前缀版和后缀版。
    - 无论使用前缀递增还是前缀递减操作符，变量的值都会在语句被求值之前改变。（在计算机科学中，这通常被称为具有副作用。）
        ```
        let age = 29; 
        let anotherAge = --age + 2; 
        console.log(age);           // 28 
        console.log(anotherAge);    // 30
        
        let num1 = 2; 
        let num2 = 20; 
        let num3 = num1-- + num2; 
        let num4 = num1 + num2;
        console.log(num3); // 22 
        console.log(num4); // 21
        ```
    - 前缀递增和递减在语句中的优先级是相等的，因此会从左到右依次求值。
    - 递增和递减的后缀版语法一样（分别是++和--），只不过要放在变量后面。后缀版与前缀版的主要区别在于，后缀版递增和递减在语句被求值后才发生。
    - 这 4 个操作符可以作用于任何值，意思是不限于整数——字符串、布尔值、浮点值，甚至对象都可以。递增和递减操作符遵循如下规则：
        - 对于字符串，如果是有效的数值形式，则转换为数值再应用改变。变量类型从字符串变成数值。
        - 对于字符串，如果不是有效的数值形式，则将变量的值设置为 NaN 。变量类型从字符串变成数值。
        - 对于布尔值，如果是 false，则转换为 0 再应用改变。变量类型从布尔值变成数值。
        - 对于布尔值，如果是 true，则转换为 1 再应用改变。变量类型从布尔值变成数值。
        - 对于浮点值，加 1 或减 1。
        - 如果是对象，则调用其 valueOf() 方法取得可以操作的值。对得到的值应用上述规则。如果是 NaN，则调用 toString()并再次应用其他规则。变量类型从对象变成数值。
    - 下面的例子演示了这些规则：
        ```
        let s1 = "2"; 
        let s2 = "z"; 
        let b = false; 
        let f = 1.1; 
        let o = { 
            valueOf() { 
                return -1; 
            } 
        }; 
        
        s1++;   // 值变成数值 3 
        s2++;   // 值变成 NaN 
        b++;    // 值变成数值 1 
        f--;    // 值变成 0.10000000000000009（因为浮点数不精确）
        o--;    // 值变成-2
        ```

55. 一元加和减
    - 一元加和减操作符在 ECMAScript 中跟在高中数学中的用途一样。一元加由一个加号（+）表示，放在变量前头，对数值没有任何影响：
        ```
        let num = 25; 
        num = +num; 
        console.log(num); // 25
        ```
    - 如果将一元加应用到非数值，则会执行与使用 Number()转型函数一样的类型转换：布尔值 false和 true 转换为 0 和 1，字符串根据特殊规则进行解析，对象会调用它们的 valueOf()和/或 toString()方法以得到可以转换的值。
    - 下面的例子演示了一元加在应用到不同数据类型时的行为：
        ```
        let s1 = "01"; 
        let s2 = "1.1";
        let s3 = "z"; 
        let b = false; 
        let f = 1.1; 
        let o = { 
            valueOf() { 
                return -1; 
            } 
        };
        
        s1 = +s1;   // 值变成数值 1 
        s2 = +s2;   // 值变成数值 1.1 
        s3 = +s3;   // 值变成 NaN 
        b = +b;     // 值变成数值 0 
        f = +f;     // 不变，还是 1.1 
        o = +o;     // 值变成数值-1
        ```
        - 一元加和减操作符主要用于基本的算术，但也可以像上面的例子那样，用于数据类型转换。

56. 位操作符
    - **ECMAScript 中的所有数值都以 IEEE 754 64 位格式存储，但位操作并不直接应用到 64 位表示，而是先把值转换为 32 位整数，再进行位操作，之后再把结果转换为 64 位。**
    - 有符号整数使用 32 位的前 31 位表示整数值。第 32 位表示数值的符号，如 0 表示正，1 表示负。这一位称为符号位（sign bit），它的值决定了数值其余部分的格式。正值以真正的二进制格式存储，即 31位中的每一位都代表 2 的幂。负值以一种称为二补数（或补码）的二进制编码存储。
    - 一个数值的二补数通过如下 3 个步骤计算得到：
        - (1) 确定绝对值的二进制表示（如，对于-18，先确定 18 的二进制表示）；
        - (2) 找到数值的一补数（或反码），换句话说，就是每个 0 都变成 1，每个 1 都变成 0；
        - (3) 给结果加 1。
    - 要注意的是，在处理有符号整数时，我们无法访问第 31 位。
    - 在把负值输出为一个二进制字符串时，我们会得到一个前面加了减号的绝对值，如下所示：
        ```
        let num = -18; 
        console.log(num.toString(2)); // "-10010"
        ```
    - 注意，默认情况下，ECMAScript 中的所有整数都表示为有符号数。不过，确实存在无符号整数。对无符号整数来说，第 32 位不表示符号，因为只有正值。无符号整数比有符号整数的范围更大，因为符号位被用来表示数值了。
    - 在对 ECMAScript 中的数值应用位操作符时，后台会发生转换：64 位数值会转换为 32 位数值，然后执行位操作，最后再把结果从 32 位转换为 64 位存储起来。整个过程就像处理 32 位数值一样，这让二进制操作变得与其他语言中类似。**但这个转换也导致了一个奇特的副作用，即特殊值NaN 和Infinity 在位操作中都会被当成 0 处理。**
    - 如果将位操作符应用到非数值，那么首先会使用 Number()函数将该值转换为数值（这个过程是自动的），然后再应用位操作。最终结果是数值。
    - 按位非操作符用波浪符（~）表示，它的作用是返回数值的一补数。按位非的最终效果是对数值取反并减 1。
    - 按位与操作符用和号（&）表示，有两个操作数。本质上，按位与就是将两个数的每一个位对齐，然后基于真值表中的规则，对每一位执行相应的与操作。
    - 按位或操作符用管道符（|）表示，同样有两个操作数。
    - 按位异或用脱字符（^）表示，同样有两个操作数。按位异或与按位或的区别是，它只在一位上是 1 的时候返回 1（两位都是 1 或 0，则返回 0）
    - 左移操作符用两个小于号（<<）表示，会按照指定的位数将数值的所有位向左移动。注意在移位后，数值右端会空出位。左移会以 0 填充这些空位，让结果是完整的 32 位数值。注意，左移会保留它所操作数值的符号。
    - 有符号右移由两个大于号（>>）表示，会将数值的所有 32 位都向右移，同时保留符号（正或负）。有符号右移实际上是左移的逆运算。同样，移位后就会出现空位。不过，右移后空位会出现在左侧，且在符号位之后。ECMAScript 会用符号位的值来填充这些空位，以得到完整的数值。
    - 无符号右移用 3 个大于号表示（>>>），会将数值的所有 32 位都向右移。对于正数，无符号右移与有符号右移结果相同。对于负数，有时候差异会非常大。与有符号右移不同，无符号右移会给空位补 0，而不管符号位是什么。无符号右移操作符将负数的二进制表示当成正数的二进制表示来处理。因为负数是其绝对值的二补数，所以右移之后结果变得非常之大。

57. 布尔操作符一共有 3 个：逻辑非、逻辑与和逻辑或。

58. 逻辑非操作符由一个叹号（!）表示，可应用给 ECMAScript 中的任何值。这个操作符始终返回布尔值，无论应用到的是什么数据类型。逻辑非操作符首先将操作数转换为布尔值，然后再对其取反。
    - 逻辑非操作符会遵循如下规则：
        - 如果操作数是对象，则返回 false。
        - 如果操作数是空字符串，则返回 true。
        - 如果操作数是非空字符串，则返回 false。
        - 如果操作数是数值 0，则返回 true。
        - 如果操作数是非 0 数值（包括 Infinity），则返回 false。
        - 如果操作数是 null，则返回 true。
        - 如果操作数是 NaN，则返回 true。
        - 如果操作数是 undefined，则返回 true。
    - 以下示例验证了上述行为：
        ```
        console.log(!false);    // true 
        console.log(!"blue");   // false 
        console.log(!0);        // true 
        console.log(!NaN);      // true 
        console.log(!"");       // true 
        console.log(!12345);    // false
        ```
    - 逻辑非操作符也可以用于把任意值转换为布尔值。同时使用两个叹号（!!），相当于调用了转型函数 Boolean()。无论操作数是什么类型，第一个叹号总会返回布尔值。第二个叹号对该布尔值取反，从而给出变量真正对应的布尔值。结果与对同一个值使用 Boolean()函数是一样的：
        ```
        console.log(!!"blue");  // true 
        console.log(!!0);       // false 
        console.log(!!NaN);     // false 
        console.log(!!"");      // false 
        console.log(!!12345);   // true
        ```

59.逻辑与操作符由两个和号（&&）表示，应用到两个值。
    - 逻辑与操作符可用于任何类型的操作数，不限于布尔值。如果有操作数不是布尔值，则逻辑与并不一定会返回布尔值，而是遵循如下规则：
        - 如果第一个操作数是对象，则返回第二个操作数。
        - 如果第二个操作数是对象，则只有第一个操作数求值为 true 才会返回该对象。
        - 如果两个操作数都是对象，则返回第二个操作数。
        - 如果有一个操作数是 null，则返回 null。
        - 如果有一个操作数是 NaN，则返回 NaN。
        - 如果有一个操作数是 undefined，则返回 undefined。
    - 逻辑与操作符是一种短路操作符，意思就是如果第一个操作数决定了结果，那么永远不会对第二个操作数求值。对逻辑与操作符来说，如果第一个操作数是 false，那么无论第二个操作数是什么值，结果也不可能等于 true。
        ```
        let found = true; 
        let result = (found && someUndeclaredVariable); // 这里会出错
        console.log(result); // 不会执行这一行
    
        let found = false; 
        let result = (found && someUndeclaredVariable); // 不会出错
        console.log(result); // 会执行
        ```
        - 例子1：变量 found 的值是 true，逻辑与操作符会继续求值变量 someUndeclaredVariable。但是由于 someUndeclaredVariable 没有定义，不能对它应用逻辑与操作符，因此就报错了。
        - 例子2：即使变量 someUndeclaredVariable 没有定义，由于第一个操作数是 false，逻辑与操作符也不会对它求值，因为此时对&&右边的操作数求值是没有意义的。

60. 逻辑或操作符由两个管道符（||）表示。
    - 与逻辑与类似，如果有一个操作数不是布尔值，那么逻辑或操作符也不一定返回布尔值。它遵循如下规则：
        - 如果第一个操作数是对象，则返回第一个操作数。
        - 如果第一个操作数求值为 false，则返回第二个操作数。
        - 如果两个操作数都是对象，则返回第一个操作数。
        - 如果两个操作数都是 null，则返回 null。
        - 如果两个操作数都是 NaN，则返回 NaN。
        - 如果两个操作数都是 undefined，则返回 undefined。
    - 逻辑或操作符也具有短路的特性。对逻辑或而言，第一个操作数求值为true，第二个操作数就不会再被求值了。**利用这个行为，可以避免给变量赋值 null 或 undefined。**比如：
        ```
        let myObject = preferredObject || backupObject;
        ```
        - 在这个例子中，变量 myObject 会被赋予两个值中的一个。其中，preferredObject 变量包含首选的值，backupObject 变量包含备用的值。如果 preferredObject 不是 null，则它的值就会赋给myObject；如果 preferredObject 是 null，则 backupObject 的值就会赋给 myObject。**这种模式在 ECMAScript 代码中经常用于变量赋值。**

61. ECMAScript 定义了 3 个乘性操作符：乘法、除法和取模。这些操作符跟它们在 Java、C 语言及 Perl中对应的操作符作用一样，但在处理非数值时，它们也会包含一些自动的类型转换。如果乘性操作符有不是数值的操作数，则该操作数会在后台被使用 Number()转型函数转换为数值。这意味着空字符串会被当成 0，而布尔值 true 会被当成 1。

62. 乘法操作符由一个星号（*）表示，可以用于计算两个数值的乘积。其语法类似于 C 语言。不过，乘法操作符在处理特殊值时也有一些特殊的行为：
    - 如果操作数都是数值，则执行常规的乘法运算，即两个正值相乘是正值，两个负值相乘也是正值，正负符号不同的值相乘得到负值。如果 ECMAScript 不能表示乘积，则返回 Infinity 或 -Infinity。
    - 如果有任一操作数是 NaN，则返回 NaN。
    - 如果是 Infinity 乘以 0，则返回 NaN。
    - 如果是 Infinity 乘以非 0的有限数值，则根据第二个操作数的符号返回 Infinity 或-Infinity。
    - 如果是 Infinity 乘以 Infinity，则返回 Infinity。
    - 如果有不是数值的操作数，则先在后台用 Number()将其转换为数值，然后再应用上述规则。

63. 除法操作符由一个斜杠（/）表示，用于计算第一个操作数除以第二个操作数的商。跟乘法操作符一样，除法操作符针对特殊值也有一些特殊的行为：
    - 如果操作数都是数值，则执行常规的除法运算，即两个正值相除是正值，两个负值相除也是正值，符号不同的值相除得到负值。如果ECMAScript不能表示商，则返回Infinity或-Infinity。
    - 如果有任一操作数是 NaN，则返回 NaN。
    - 如果是 Infinity 除以 Infinity，则返回 NaN。
    - 如果是 0 除以 0，则返回 NaN。
    - 如果是非 0 的有限值除以 0，则根据第一个操作数的符号返回 Infinity 或-Infinity。
    - 如果是 Infinity 除以任何数值，则根据第二个操作数的符号返回 Infinity 或-Infinity。
    - 如果有不是数值的操作数，则先在后台用 Number()函数将其转换为数值，然后再应用上述规则。

64. 取模（余数）操作符由一个百分比符号（%）表示。与其他乘性操作符一样，取模操作符对特殊值也有一些特殊的行为：
    - 如果操作数是数值，则执行常规除法运算，返回余数。
    - 如果被除数是无限值，除数是有限值，则返回 NaN。
    - 如果被除数是有限值，除数是 0，则返回 NaN。
    - 如果是 Infinity 除以 Infinity，则返回 NaN。
    - 如果被除数是有限值，除数是无限值，则返回被除数。
    - 如果被除数是 0，除数不是 0，则返回 0。
    - 如果有不是数值的操作数，则先在后台用 Number()函数将其转换为数值，然后再应用上述规则。

65. 指数操作符
    - ECMAScript 7 新增了指数操作符，Math.pow()现在有了自己的操作符**，结果是一样的：
        ```
        console.log(Math.pow(3, 2); // 9 
        console.log(3 ** 2);        // 9 
        
        console.log(Math.pow(16, 0.5);  // 4 
        console.log(16** 0.5);          // 4
        ```
    - 指数操作符也有自己的指数赋值操作符**=，该操作符执行指数运算和结果的赋值操作：
        ```
        let squared = 3; 
        squared **= 2; 
        console.log(squared);   // 9
        
        let sqrt = 16; 
        sqrt **= 0.5; 
        console.log(sqrt);      // 4
        ```

66. 加性操作符，即加法和减法操作符。不过，在 ECMAScript 中，这两个操作符拥有一些特殊的行为。与乘性操作符类似，加性操作符在后台会发生不同数据类型的转换。只不过对这两个操作符来说，转换规则不是那么直观。

67. 加法操作符（+）用于求两个数的和。
    - 如果两个操作数都是数值，加法操作符执行加法运算并根据如下规则返回结果：
        - 如果有任一操作数是 NaN，则返回 NaN；
        - 如果是 Infinity 加 Infinity，则返回 Infinity；
        - 如果是-Infinity 加-Infinity，则返回-Infinity；
        - 如果是 Infinity 加-Infinity，则返回 NaN；
        - 如果是+0 加+0，则返回+0；
        - 如果是-0 加+0，则返回+0；
        - 如果是-0 加-0，则返回-0。
    - 不过，如果有一个操作数是字符串，则要应用如下规则：
        - 如果两个操作数都是字符串，则将第二个字符串拼接到第一个字符串后面；
        - **如果只有一个操作数是字符串，则将另一个操作数转换为字符串，再将两个字符串拼接在一起。**
    - 如果有任一操作数是对象、数值或布尔值，则调用它们的 toString()方法以获取字符串，然后再应用前面的关于字符串的规则。对于 undefined 和 null，则调用 String()函数，分别获取"undefined"和"null"。
        ```
        let result1 = 5 + 5;        // 两个数值
        console.log(result1);       // 10 
        
        let result2 = 5 + "5";      // 一个数值和一个字符串
        console.log(result2);       // "55"，第一个操作数被转换为字符串
        ```
    - **ECMAScript 中最常犯的一个错误，就是忽略加法操作中涉及的数据类型。**比如：
        ```
        let num1 = 5; 
        let num2 = 10; 
        let message = "The sum of 5 and 10 is " + num1 + num2; 
        console.log(message); // "The sum of 5 and 10 is 510"
        
        let message = "The sum of 5 and 10 is " + (num1 + num2); 
        console.log(message); // "The sum of 5 and 10 is 15"
        ```

68. 减法操作符（-）
    - 与加法操作符一样，减法操作符也有一组规则用于处理 ECMAScript 中不同类型之间的转换。
        - 如果两个操作数都是数值，则执行数学减法运算并返回结果。
        - 如果有任一操作数是 NaN，则返回 NaN。
        - 如果是 Infinity 减 Infinity，则返回 NaN。
        - 如果是-Infinity 减-Infinity，则返回 NaN。
        - 如果是 Infinity 减-Infinity，则返回 Infinity。
        - 如果是-Infinity 减 Infinity，则返回-Infinity。
        - 如果是+0 减+0，则返回+0。
        - 如果是+0 减-0，则返回-0。
        - 如果是-0 减-0，则返回+0。
        - **如果有任一操作数是字符串、布尔值、null 或 undefined，则先在后台使用 Number()将其转换为数值，然后再根据前面的规则执行数学运算。**如果转换结果是 NaN，则减法计算的结果是NaN。
        - **如果有任一操作数是对象，则调用其 valueOf()方法取得表示它的数值。**如果该值是 NaN，则减法计算的结果是 NaN。如果对象没有 valueOf()方法，则调用其 toString()方法，然后再将得到的字符串转换为数值。
        - 读者备注：**注意和加法是不一样的。**
    - 以下示例演示了上面的规则：
        ```
        let result1 = 5 - true;     // true 被转换为 1，所以结果是 4 
        let result2 = NaN - 1;      // NaN 
        let result3 = 5 - 3;        // 2 
        let result4 = 5 - "";       // ""被转换为 0，所以结果是 5 
        let result5 = 5 - "2";      // "2"被转换为 2，所以结果是 3 
        let result6 = 5 - null;     // null 被转换为 0，所以结果是 5
        ```

69. 关系操作符执行比较两个值的操作，包括小于（<）、大于（>）、小于等于（<=）和大于等于（>=），用法跟数学课上学的一样。这几个操作符都返回布尔值。
    - 与 ECMAScript 中的其他操作符一样，在将它们应用到不同数据类型时也会发生类型转换和其他行为：
        - 如果操作数都是数值，则执行数值比较。
        - 如果操作数都是字符串，则逐个比较字符串中对应字符的编码。
        - **如果有任一操作数是数值，则将另一个操作数转换为数值，执行数值比较。**
        - **如果有任一操作数是对象，则调用其 valueOf()方法，取得结果后再根据前面的规则执行比较。如果没有 valueOf()操作符，则调用 toString()方法，取得结果后再根据前面的规则执行比较。**
        - 如果有任一操作数是布尔值，则将其转换为数值再执行比较。
    - 对字符串而言，关系操作符会比较字符串中对应字符的编码，而这些编码是数值。比较完之后，会返回布尔值。问题的关键在于，大写字母的编码都小于小写字母的编码。
        ```
        let result = "Brick" < "alphabet";  // true
        let result = "Brick".toLowerCase() < "alphabet".toLowerCase(); // false
        
        let result = "23" < "3"; // true
        let result = "23" < 3;   // false（将字符串"23"转换为数值 23，然后再跟 3 比较）
        ```
        - 字母 B 的编码是 66，字母 a 的编码是 97。
        - 要得到确实按字母顺序比较的结果，就必须把两者都转换为相同的大小写形式（全大写或全小写），然后再比较。
        - 只要是数值和字符串比较，字符串就会先被转换为数值，然后进行数值比较。对于数值字符串而言，这样能保证结果正确。
    - **任何关系操作符在涉及比较 NaN 时都返回 false。**
        ```
        let result = "a" < 3;   // 因为"a"会转换为 NaN，所以结果是 false
        
        let result1 = NaN < 3;  // false 
        let result2 = NaN >= 3; // false
        ```
        - 因为字符"a"不能转换成任何有意义的数值，所以只能转换为 NaN。
        - 在大多数比较的场景中，如果一个值不小于另一个值，那就一定大于或等于它。但在比较 NaN 时，无论是小于还是大于等于，比较的结果都会返回 false。

70. ECMAScript 提供了两组操作符。第一组是等于和不等于，它们在比较之前执行转换。第二组是全等和不全等，它们在比较之前不执行转换。

71. ECMAScript 中的等于操作符用两个等于号（==）表示，如果操作数相等，则会返回 true。不等于操作符用叹号和等于号（!=）表示，如果两个操作数不相等，则会返回 true。这两个操作符都会先进行类型转换（通常称为强制类型转换）再确定操作数是否相等。
    - 在转换操作数的类型时，相等和不相等操作符遵循如下规则：
        - 如果任一操作数是布尔值，则将其转换为数值再比较是否相等。false 转换为 0，true 转换为 1。
        - 如果一个操作数是字符串，另一个操作数是数值，则尝试将字符串转换为数值，再比较是否相等。
        - 如果一个操作数是对象，另一个操作数不是，则调用对象的 valueOf()方法取得其原始值，再根据前面的规则进行比较。
    - 在进行比较时，这两个操作符会遵循如下规则：
        - null 和 undefined 相等。
        - null 和 undefined 不能转换为其他类型的值再进行比较。
        - 如果有任一操作数是 NaN，则相等操作符返回 false，不相等操作符返回 true。记住：**即使两个操作数都是 NaN，相等操作符也返回 false，因为按照规则，NaN 不等于 NaN。**
        - 如果两个操作数都是对象，则比较它们是不是同一个对象。如果两个操作数都指向同一个对象，则相等操作符返回 true。否则，两者不相等。
    - 下表总结了一些特殊情况及比较的结果：

        | 表达式 | 结果 |
        | --- | --- |
        | null == undefined | true |
        | "NaN" == NaN | false |
        | 5 == NaN | false |
        | NaN == NaN | false |
        | NaN != NaN | true |
        | false == 0 | true |
        | true == 1 | true |
        | true == 2 | false |
        | undefined == 0 | false |
        | null == 0 | false |
        | "5" == 5 | true |

72. 全等和不全等操作符与相等和不相等操作符类似，只不过它们在比较相等时不转换操作数。全等操作符由 3 个等于号（\=\=\=）表示，只有两个操作数在不转换的前提下相等才返回 true。不全等操作符用一个叹号和两个等于号（!==）表示，只有两个操作数在不转换的前提下不相等才返回 true。
    ```
    let result1 = ("55" == 55);     // true，转换后相等
    let result2 = ("55" === 55);    // false，不相等，因为数据类型不同
    
    let result1 = ("55" != 55);     // false，转换后相等
    let result2 = ("55" !== 55);    // true，不相等，因为数据类型不同
    ```
    - 虽然 null == undefined 是 true（因为这两个值类似），但 null === undefined 是 false，因为它们不是相同的数据类型。
    - **注意，由于相等和不相等操作符存在类型转换问题，因此推荐使用全等和不全等操作符。这样有助于在代码中保持数据类型的完整性。**

73. 条件操作符是 ECMAScript 中用途最为广泛的操作符之一，语法跟 Java 中一样：
    ```
    variable = boolean_expression ? true_value : false_value;
    
    let max = (num1 > num2) ? num1 : num2;
    ```

74. 赋值操作符
    - 简单赋值用等于号（=）表示，将右手边的值赋给左手边的变量。
        ```
        let num = 10;
        ```
    - 复合赋值使用乘性、加性或位操作符后跟等于号（=）表示。
        ```
        let num = 10; 
        num += 10;
        ```
    - 每个数学操作符以及其他一些操作符都有对应的复合赋值操作符：
        - 乘后赋值（*=）
        - 除后赋值（/=）
        - 取模后赋值（%=）
        - 加后赋值（+=）
        - 减后赋值（-=）
        - 左移后赋值（<<=）
        - 右移后赋值（>>=）
        - 无符号右移后赋值（>>>=）
    - 这些操作符仅仅是简写语法，使用它们不会提升性能。

75. 逗号操作符
    - 逗号操作符可以用来在一条语句中执行多个操作：
        ```
        let num1 = 1, num2 = 2, num3 = 3;
        ```
        - 在一条语句中同时声明多个变量是逗号操作符最常用的场景。
    - 也可以使用逗号操作符来辅助赋值。在赋值时使用逗号操作符分隔值，最终会返回表达式中最后一个值：
        ```
        let num = (5, 1, 4, 8, 0); // num 的值为 0
        ```

76. if 语句
    - 语法如下：
        ```
        if (condition) statement1 else statement2
        ```
        - 这里的条件（condition）可以是任何表达式，并且求值结果不一定是布尔值。ECMAScript 会自动调用 Boolean()函数将这个表达式的值转换为布尔值。
    - 可以像这样连续使用多个 if 语句：
        ```
        if (condition1) statement1 else if (condition2) statement2 else statement3
        
        if (i > 25) { 
            console.log("Greater than 25."); 
        } else if (i < 0) { 
            console.log("Less than 0."); 
        } else { 
            console.log("Between 0 and 25, inclusive."); 
        }
        ```

77. do-while 语句
    - do-while 语句是一种后测试循环语句，即循环体中的代码执行后才会对退出条件进行求值。换句话说，循环体内的代码至少执行一次。
    - 语法如下：    
        ```
        do { 
            statement 
        } while (expression);
        ```
    - 注意，后测试循环经常用于这种情形：循环体内代码在退出前至少要执行一次。

78. while 语句
    - while 语句是一种先测试循环语句，即先检测退出条件，再执行循环体内的代码。因此，while 循环体内的代码有可能不会执行。
    - 语法如下：    
        ```
        while(expression) statement
        ```

79. for 语句
    - for 语句也是先测试语句，只不过增加了进入循环之前的初始化代码，以及循环执行后要执行的表达式，语法如下：
        ```
        for (initialization; expression; post-loop-expression) statement
        ```
    - 无法通过 while 循环实现的逻辑，同样也无法使用 for 循环实现。
    - 初始化、条件表达式和循环后表达式都不是必需的。

80. for-in 语句
    - for-in 语句是一种严格的迭代语句，**用于枚举对象中的非符号键属性，**语法如下：
        ```
        for (property in expression) statement
        ```
    - 例子：
        ```
        for (const propName in window) { 
            document.write(propName); 
        }
        ```
        - 这个例子使用 for-in 循环显示了 BOM 对象 window 的所有属性。每次执行循环，都会给变量 propName 赋予一个 window 对象的属性作为值，直到 window 的所有属性都被枚举一遍。
        - 与 for 循环一样，这里控制语句中的 const 也不是必需的。但为了确保这个局部变量不被修改，推荐使用 const。
    - ECMAScript 中对象的属性是无序的，因此 for-in 语句不能保证返回对象属性的顺序。换句话说，所有可枚举的属性都会返回一次，但返回的顺序可能会因浏览器而异。
    - 如果 for-in 循环要迭代的变量是 null 或 undefined，则不执行循环体。

81. for-of 语句
    - for-of 语句是一种严格的迭代语句，**用于遍历可迭代对象的元素，**语法如下：
        ```
        for (property of expression) statement
        ```
    - 例子：
        ```
        for (const el of [2,4,6,8]) { 
            document.write(el); 
        }
        ```
        - 在这个例子中，我们使用 for-of 语句显示了一个包含 4 个元素的数组中的所有元素。循环会一直持续到将所有元素都迭代完。与 for 循环一样，这里控制语句中的 const 也不是必需的。但为了确保这个局部变量不被修改，推荐使用 const。
    - for-of 循环会按照可迭代对象的 next()方法产生值的顺序迭代元素。
    - 如果尝试迭代的变量不支持迭代，则 for-of 语句会抛出错误。
    - 注意，ES2018 对 for-of 语句进行了扩展，增加了 for-await-of 循环，以支持生成期约（promise）的异步可迭代对象。

82. 标签语句
    - 标签语句用于给语句加标签，语法如下：
        ```
        label: statement
        ```
    - 例子：
        ```
        start: for (let i = 0; i < count; i++) { 
            console.log(i); 
        }
        ```
        - 在这个例子中，start 是一个标签，可以在后面通过 break 或 continue 语句引用。
    - 标签语句的典型应用场景是嵌套循环。

83. break 和 continue 语句
    - break 语句用于立即退出循环，强制执行循环后的下一条语句。而 continue 语句也用于立即退出循环，但会再次从循环顶部开始执行。
    - break 和 continue 都可以与标签语句一起使用，返回代码中特定的位置。这通常是在嵌套循环中，如下面的例子所示：
        ```
        let num = 0; 
        outermost: 
        for (let i = 0; i < 10; i++) { 
            for (let j = 0; j < 10; j++) { 
                if (i == 5 && j == 5) { 
                    break outermost; 
                } 
                num++; 
            } 
        } 
        console.log(num); // 55
        ```
        - 在这个例子中，outermost 标签标识的是第一个 for 语句。
        - break 语句带来了一个变数，即要退出到的标签。添加标签不仅让 break 退出（使用变量 j 的）内部循环，也会退出（使用变量 i 的）外部循环。
    - continue 语句也可以使用标签，如下面的例子所示：
        ```
        let num = 0; 
        outermost: 
        for (let i = 0; i < 10; i++) { 
            for (let j = 0; j < 10; j++) {
                if (i == 5 && j == 5) { 
                    continue outermost; 
                } 
                num++; 
            } 
        } 
        console.log(num); // 95
        ```
        - 这一次，continue 语句会强制循环继续执行，但不是继续执行内部循环，而是继续执行外部循环。当 i 和 j 都等于 5 时，会执行 continue，跳到外部循环继续执行，从而导致内部循环少执行 5 次，结果 num 等于 95。
    - 组合使用标签语句和 break、continue 能实现复杂的逻辑，但也容易出错。注意标签要使用描述性强的文本，而嵌套也不要太深。

84. with 语句
    - with 语句的用途是将代码作用域设置为特定的对象，其语法是：
        ```
        with (expression) statement;
        ```
    - 使用 with 语句的主要场景是针对一个对象反复操作，这时候将代码作用域设置为该对象能提供便利，如下面的例子所示：
        ```
        let qs = location.search.substring(1); 
        let hostName = location.hostname; 
        let url = location.href;
        ```
    - 上面代码中的每一行都用到了 location 对象。如果使用 with 语句，就可以少写一些代码：
        ```
        with(location) { 
            let qs = search.substring(1); 
            let hostName = hostname; 
            let url = href; 
        }
        ```
        - 这里，with 语句用于连接 location 对象。这意味着在这个语句内部，每个变量首先会被认为是一个局部变量。如果没有找到该局部变量，则会搜索 location 对象，看它是否有一个同名的属性。如果有，则该变量会被求值为 location 对象的属性。
    - 严格模式不允许使用 with 语句，否则会抛出错误。
    - 警告，由于 with 语句影响性能且难于调试其中的代码，**通常不推荐在产品代码中使用 with语句。**

85. switch 语句
    - ECMAScript中 switch 语句跟 C 语言中 switch 语句的语法非常相似，如下所示：
        ```
        switch (expression) { 
            case value1: 
                statement
                break; 
            case value2: 
                statement 
                break; 
            case value3: 
                statement 
                break; 
            case value4: 
                statement 
                break; 
            default: 
                statement 
        }
        ```
    - 为避免不必要的条件判断，最好给每个条件后面都加上 break 语句。如果确实需要连续匹配几个条件，那么推荐写个注释表明是故意忽略了 break，如下所示：
        ```
        switch (i) { 
            case 25: 
                /*跳过*/ 
            case 35: 
                console.log("25 or 35"); 
                break; 
            case 45: 
                console.log("45"); 
                break; 
            default:
                console.log("Other"); 
        }
        ```
    - 虽然 switch 语句是从其他语言借鉴过来的，但 ECMAScript 为它赋予了一些独有的特性。首先，switch 语句可以用于所有数据类型（在很多语言中，它只能用于数值），因此可以使用字符串甚至对象。其次，条件的值不需要是常量，也可以是变量或表达式。看下面的例子：
        ```
        switch ("hello world") { 
            case "hello" + " world": 
                console.log("Greeting was found."); 
                break; 
            case "goodbye": 
                console.log("Closing was found."); 
                break; 
            default: 
                console.log("Unexpected message was found."); 
        }
        ```
    - 能够在条件判断中使用表达式，就可以在判断中加入更多逻辑：
        ```
        let num = 25; 
        switch (true) { 
            case num < 0: 
                console.log("Less than 0."); 
                break; 
            case num >= 0 && num <= 10: 
                console.log("Between 0 and 10."); 
                break; 
            case num > 10 && num <= 20: 
                console.log("Between 10 and 20."); 
                break; 
            default: 
                console.log("More than 20."); 
        }
        ```
        - 上面的代码首先在外部定义了变量 num，而传给 switch 语句的参数之所以是 true，就是因为每个条件的表达式都会返回布尔值。条件的表达式分别被求值，直到有表达式返回 true；否则，就会一直跳到 default 语句（这个例子正是如此）。
    - **注意，switch 语句在比较每个条件的值时会使用全等操作符，因此不会强制转换数据类型（比如，字符串"10"不等于数值 10）**

86. ECMAScript 中的函数使用 function 关键字声明，后跟一组参数，然后是函数体。
    - 以下是函数的基本语法：
        ```
        function functionName(arg0, arg1,...,argN) { 
            statements 
        }
        ```
    - 可以通过函数名来调用函数，要传给函数的参数放在括号里（如果有多个参数，则用逗号隔开）。
    - ECMAScript 中的函数不需要指定是否返回值。任何函数在任何时间都可以使用 return 语句来返回函数的值，用法是后跟要返回的值。比如：
        ```
        function sum(num1, num2) { 
            return num1 + num2; 
        }
        
        const result = sum(5, 10);
        ```
        - 注意，除了 return 语句之外没有任何特殊声明表明该函数有返回值。
    - 要注意的是，只要碰到 return 语句，函数就会立即停止执行并退出。因此，return 语句后面的代码不会被执行。
    - 一个函数里也可以有多个 return 语句，例如if-else每个分支都有自己的return语句。
    - **return 语句也可以不带返回值。这时候，函数会立即停止执行并返回 undefined。这种用法最常用于提前终止函数执行，并不是为了返回值。**
    - 注意，最佳实践是函数要么返回值，要么不返回值。只在某个条件下返回值的函数会带来麻烦，尤其是调试时。
    - 严格模式对函数也有一些限制：
        - 函数不能以 eval 或 arguments 作为名称；
        - 函数的参数不能叫 eval 或 arguments；
        - 两个命名参数不能拥有同一个名称。
    - 如果违反上述规则，则会导致语法错误，代码也不会执行。

87. 本章总结
    - JavaScript 的核心语言特性在 ECMA-262 中以伪语言 ECMAScript 的形式来定义。ECMAScript 包含所有基本语法、操作符、数据类型和对象，能完成基本的计算任务，但没有提供获得输入和产生输出的机制。
    - 下面总结一下 ECMAScript 中的基本元素。
        - ECMAScript 中的基本数据类型包括 Undefined、Null、Boolean、Number、String 和 Symbol。
        - 与其他语言不同，ECMAScript 不区分整数和浮点值，只有 Number 一种数值数据类型。
        - Object 是一种复杂数据类型，它是这门语言中所有对象的基类。
        - 严格模式为这门语言中某些容易出错的部分施加了限制。
        - ECMAScript 提供了 C 语言和类 C 语言中常见的很多基本操作符，包括数学操作符、布尔操作符、关系操作符、相等操作符和赋值操作符等。
        - 这门语言中的流控制语句大多是从其他语言中借鉴而来的，比如 if 语句、for 语句和 switch 语句等。
    - ECMAScript 中的函数与其他语言中的函数不一样。
        - 不需要指定函数的返回值，因为任何函数可以在任何时候返回任何值。
        - **不指定返回值的函数实际上会返回特殊值 undefined**。
