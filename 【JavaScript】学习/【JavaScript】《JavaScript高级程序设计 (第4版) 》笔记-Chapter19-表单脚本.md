###### 十九、表单脚本

1. 表单脚本
    - JavaScript 较早的一个用途是承担一部分服务器端表单处理的责任。虽然 Web 和 JavaScript 都已经发展了很多年，但 Web 表单的变化不是很大。由于不能直接使用表单解决问题，因此开发者不得不使用JavaScript 既做表单验证，又用于增强标准表单控件的默认行为。

2. 表单基础
    - Web 表单在 HTML 中以`<form>`元素表示，在 JavaScript 中则以 HTMLFormElement 类型表示。HTMLFormElement 类型继承自 HTMLElement 类型，因此拥有与其他 HTML 元素一样的默认属性。不过，HTMLFormElement 也有自己的属性和方法。
        - acceptCharset：服务器可以接收的字符集，等价于 HTML 的 accept-charset 属性。
        - action：请求的 URL，等价于 HTML 的 action 属性。
        - elements：表单中所有控件的 HTMLCollection。
        - enctype：请求的编码类型，等价于 HTML 的 enctype 属性。
        - length：表单中控件的数量。
        - method：HTTP 请求的方法类型，通常是"get"或"post"，等价于 HTML 的 method 属性。
        - name：表单的名字，等价于 HTML 的 name 属性。
        - reset()：把表单字段重置为各自的默认值。
        - submit()：提交表单。
        - target：用于发送请求和接收响应的窗口的名字，等价于 HTML 的 target 属性。
    - 有几种方式可以取得对`<form>`元素的引用。最常用的是将表单当作普通元素为它指定一个 id 属性，从而可以使用 getElementById()来获取表单，比如：
        ```
        let form = document.getElementById("form1");
        ```
    - 此外，使用 document.forms 集合可以获取页面上所有的表单元素。然后，可以进一步使用数字索引或表单的名字（name）来访问特定的表单。比如：
        ```
        // 取得页面中的第一个表单
        let firstForm = document.forms[0]; 
        // 取得名字为"form2"的表单
        let myForm = document.forms["form2"];
        ```
    - 较早的浏览器，或者严格向后兼容的浏览器，也会把每个表单的 name 作为 document 对象的属性。例如，名为"form2"的表单可以通过 document.form2 来访问。不推荐使用这种方法，因为容易出错，而且这些属性将来可能会被浏览器删除。
    - 注意，表单可以同时拥有 id 和 name，而且两者可以不相同。

3. 提交表单
    - 表单是通过用户点击提交按钮或图片按钮的方式提交的。提交按钮可以使用 type 属性为"submit"的`<input>`或`<button>`元素来定义，图片按钮可以使用 type 属性为"image"的`<input>`元素来定义。点击下面例子中定义的所有按钮都可以提交它们所在的表单：
        ```
        <!-- 通用提交按钮 --> 
        <input type="submit" value="Submit Form"> 
        <!-- 自定义提交按钮 --> 
        <button type="submit">Submit Form</button> 
        <!-- 图片按钮 --> 
        <input type="image" src="graphic.gif">
        ```
    - 如果表单中有上述任何一个按钮，且焦点在表单中某个控件上，则按回车键也可以提交表单。（textarea 控件是个例外，当焦点在它上面时，按回车键会换行。）注意，没有提交按钮的表单在按回车键时不会提交。
    - 以这种方式提交表单会在向服务器发送请求之前触发 submit 事件。这样就提供了一个验证表单数据的机会，可以根据验证结果决定是否真的要提交。阻止这个事件的默认行为可以取消提交表单。例如，下面的代码会阻止表单提交：
        ```
        let form = document.getElementById("myForm"); 
        form.addEventListener("submit", (event) => { 
            // 阻止表单提交
            event.preventDefault(); 
        });
        ```
    - 调用 preventDefault()方法可以阻止表单提交。通常，在表单数据无效以及不应该发送到服务器时可以这样处理。
    - 当然，也可以通过编程方式在 JavaScript 中调用 submit()方法来提交表单。可以在任何时候调用这个方法来提交表单，而且表单中不存在提交按钮也不影响表单提交。下面是一个例子：
        ```
        let form = document.getElementById("myForm"); 
        // 提交表单
        form.submit();
        ```
    - 通过 submit()提交表单时，submit 事件不会触发。因此在调用这个方法前要先做数据验证。
    - 表单提交的一个最大的问题是可能会提交两次表单。如果提交表单之后没有什么反应，那么没有耐心的用户可能会多次点击提交按钮。结果是很烦人的（因为服务器要处理重复的请求），甚至可能造成损失（如果用户正在购物，则可能会多次下单）。解决这个问题主要有两种方式：在表单提交后禁用提交按钮，或者通过 onsubmit 事件处理程序取消之后的表单提交。

4. 重置表单
    - 用户单击重置按钮可以重置表单。重置按钮可以使用 type 属性为"reset"的`<input>`或`<button>`元素来创建，比如：
        ```
        <!-- 通用重置按钮 --> 
        <input type="reset" value="Reset Form"> 
        <!-- 自定义重置按钮 --> 
        <button type="reset">Reset Form</button>
        ```
    - 这两种按钮都可以重置表单。表单重置后，所有表单字段都会重置回页面第一次渲染时各自拥有的值。如果字段原来是空的，就会变成空的；如果字段有默认值，则恢复为默认值。
    - 用户单击重置按钮重置表单会触发 reset 事件。这个事件为取消重置提供了机会。例如，以下代码演示了如何阻止重置表单：
        ```
        let form = document.getElementById("myForm"); 
        form.addEventListener("reset", (event) => { 
            event.preventDefault(); 
        });
        ```
    - 与表单提交一样，重置表单也可以通过 JavaScript 调用 reset()方法来完成，如下面的例子所示：
        ```
        let form = document.getElementById("myForm"); 
        // 重置表单
        form.reset();
        ```
    - 与 submit()方法的功能不同，调用 reset()方法会像单击了重置按钮一样触发 reset 事件。
    - 注意，表单设计中通常不提倡重置表单，因为重置表单经常会导致用户迷失方向，如果意外触发则会令人感到厌烦。实践中几乎没有重置表单的需求。一般来说，提供一个取消按钮，让用户点击返回前一个页面，而不是恢复表单中所有的值来得更直观。

5. 表单字段
    - 表单元素可以像页面中的其他元素一样使用原生 DOM 方法来访问。此外，所有表单元素都是表单elements 属性（元素集合）中包含的一个值。这个 elements 集合是一个有序列表，包含对表单中所有字段的引用，包括所有`<input>`、`<textarea>`、`<button>`、`<select>`和`<fieldset>`元素。elements集合中的每个字段都以它们在 HTML 标记中出现的次序保存，可以通过索引位置和 name 属性来访问。以下是几个例子：
        ```
        let form = document.getElementById("form1"); 
        // 取得表单中的第一个字段
        let field1 = form.elements[0]; 
        // 取得表单中名为"textbox1"的字段
        let field2 = form.elements["textbox1"]; 
        // 取得字段的数量
        let fieldCount = form.elements.length;
        ```
    - 如果多个表单控件使用了同一个 name，比如像单选按钮那样，则会返回包含所有同名元素的HTMLCollection。比如，来看下面的 HTML 代码片段：
        ```
        <form method="post" id="myForm"> 
            <ul> 
                <li><input type="radio" name="color" value="red">Red</li> 
                <li><input type="radio" name="color" value="green">Green</li> 
                <li><input type="radio" name="color" value="blue">Blue</li> 
            </ul> 
        </form>
        ```
    - 这个 HTML 中的表单有 3 个单选按钮的 name 是"color"，这个名字把它们联系在了一起。在访问elements["color"]时，返回的 NodeList 就包含这 3 个元素。而在访问 elements[0]时，只会返回第一个元素。比如：
        ```
        let form = document.getElementById("myForm"); 
        let colorFields = form.elements["color"]; 
        console.log(colorFields.length); // 3 
        let firstColorField = colorFields[0]; 
        let firstFormField = form.elements[0]; 
        console.log(firstColorField === firstFormField); // true
        ```
    - 以上代码表明，使用 form.elements[0]获取的表单的第一个字段就是 form.elements["color"]中包含的第一个元素。
    - 注意，也可以通过表单属性的方式访问表单字段，比如 form[0]这种使用索引和 form["color"]这种使用字段名字的方式。访问这些属性与访问 form.elements 集合是一样的。这种方式是为向后兼容旧版本浏览器而提供的，实际开发中应该使用 elements。

6. 表单字段的公共属性
    - 除`<fieldset>`元素以外，所有表单字段都有一组同样的属性。由于`<input>`类型可以表示多种表单字段，因此某些属性只适用于特定类型的字段。除此之外的属性可以在任何表单字段上使用。以下列出了这些表单字段的公共属性和方法。
        - disabled：布尔值，表示表单字段是否禁用。
        - form：指针，指向表单字段所属的表单。这个属性是只读的。
        - name：字符串，这个字段的名字。
        - readOnly：布尔值，表示这个字段是否只读。
        - tabIndex：数值，表示这个字段在按 Tab 键时的切换顺序。
        - type：字符串，表示字段类型，如"checkbox"、"radio"等。
        - value：要提交给服务器的字段值。对文件输入字段来说，这个属性是只读的，仅包含计算机上某个文件的路径。
    - 这里面除了 form 属性以外，JavaScript 可以动态修改任何属性。来看下面的例子：
        ```
        let form = document.getElementById("myForm"); 
        let field = form.elements[0]; 
        // 修改字段的值
        field.value = "Another value"; 
        // 检查字段所属的表单
        console.log(field.form === form); // true 
        // 给字段设置焦点
        field.focus(); 
        // 禁用字段
        field.disabled = true; 
        // 改变字段的类型（不推荐，但对<input>来说是可能的）
        field.type = "checkbox";
        ```
    - 这种动态修改表单字段属性的能力为任何时候以任何方式修改表单提供了方便。举个例子，Web 表单的一个常见问题是用户常常会点击两次提交按钮。在涉及信用卡扣款的情况下，这是个严重的问题，可能会导致重复扣款。对此，常见的解决方案是第一次点击之后禁用提交按钮。可以通过监听 submit事件来实现。比如下面这个例子：
        ```
        // 避免多次提交表单的代码
        let form = document.getElementById("myForm"); 
        form.addEventListener("submit", (event) => { 
            let target = event.target; 
            // 取得提交按钮
            let btn = target.elements["submit-btn"]; 
            // 禁用提交按钮
            btn.disabled = true; 
        });
        ```
    - 以上代码在表单的 submit 事件上注册了一个事件处理程序。当 submit 事件触发时，代码会取得提交按钮，然后将其 disabled 属性设置为 true。**注意，这个功能不能通过直接给提交按钮添加onclick 事件处理程序来实现，原因是不同浏览器触发事件的时机不一样。有些浏览器会在触发表单的 submit 事件前先触发提交按钮的 click 事件，有些浏览器则会后触发 click 事件。对于先触发click 事件的浏览器，这个按钮会在表单提交前被禁用，这意味着表单就不会被提交了。因此最好使用表单的 submit 事件来禁用提交按钮。但这种方式不适用于没有使用提交按钮的表单提交。如前所述，只有提交按钮才能触发 submit 事件。**
    - type 属性可以用于除`<fieldset>`之外的任何表单字段。对于`<input>`元素，这个值等于 HTML的 type 属性值。对于其他元素，这个 type 属性的值按照下表设置。

        描述 | 示例 HTML | 类型的值
        ---|---|---
        单选列表 | `<select>...</select>` | "select-one"
        多选列表 | `<select multiple>...</select>` | "select-multiple"
        自定义按钮 | `<button>...</button>` | "submit"
        自定义非提交按钮 | `<button type="button">...</button>` | "button"
        自定义重置按钮 | `<button type="reset">...</button>` | "reset"
        自定义提交按钮 | `<button type="submit">...</button>` | "submit"

    - 对于`<input>`和`<button>`元素，可以动态修改其 type 属性。但`<select>`元素的 type 属性是只读的。

7. 表单字段的公共方法
    - 每个表单字段都有两个公共方法：focus()和 blur()。focus()方法把浏览器焦点设置到表单字段，这意味着该字段会变成活动字段并可以响应键盘事件。例如，文本框在获得焦点时会在内部显示闪烁的光标，表示可以接收输入。focus()方法主要用来引起用户对页面中某个部分的注意。比如，在页面加载后把焦点定位到表单中第一个字段就是很常见的做法。实现方法是监听 load 事件，然后在第一个字段上调用 focus()，如下所示：
        ```
        window.addEventListener("load", (event) => { 
            document.forms[0].elements[0].focus(); 
        });
        ```
    - 注意，如果表单中第一个字段是 type 为"hidden"的`<input>`元素，或者该字段被 CSS 属性display 或 visibility 隐藏了，以上代码就会出错。
    - HTML5 为表单字段增加了 autofocus 属性，支持的浏览器会自动为带有该属性的元素设置焦点，而无须使用 JavaScript。比如：
        ```
        <input type="text" autofocus>
        ```
    - 为了让之前的代码在使用 autofocus 时也能正常工作，必须先检测元素上是否设置了该属性。如果设置了 autofocus，就不再调用 focus()：
        ```
        window.addEventListener("load", (event) => { 
            let element = document.forms[0].elements[0]; 
            if (element.autofocus !== true) { 
                element.focus(); 
                console.log("JS focus"); 
            } 
        });
        ```
    - 因为 autofocus 是布尔值属性，所以在支持的浏览器中通过 JavaScript访问表单字段的 autofocus属性会返回 true（在不支持的浏览器中是空字符串）。上面的代码只会在 autofocus 属性不等于 true时调用 focus()方法，以确保向前兼容。大多数现代浏览器支持 autofocus 属性，只有 iOS Safari、Opera Mini 和 IE10 及以下版本不支持。
    - 注意，默认情况下只能给表单元素设置焦点。不过，通过将 tabIndex 属性设置为–1 再调用 focus()，也可以给任意元素设置焦点。只有 Opera 不支持这个技术。
    - focus()的反向操作是 blur()，其用于从元素上移除焦点。调用 blur()时，焦点不会转移到任何特定元素，仅仅只是从调用这个方法的元素上移除了。在浏览器支持 readonly 属性之前，Web 开发者通常会使用这个方法创建只读字段。现在很少有用例需要调用 blur()，不过如果需要是可以用的。下面是一个例子：
        ```
        document.forms[0].elements[0].blur();
        ```

8. 表单字段的公共事件
    - 除了鼠标、键盘、变化和 HTML 事件外，所有字段还支持以下 3 个事件。
        - blur：在字段失去焦点时触发。
        - change：在`<input>`和`<textarea>`元素的 value 发生变化且失去焦点时触发，或者在`<select>`元素中选中项发生变化时触发。
        - focus：在字段获得焦点时触发。
    - blur 和 focus 事件会因为用户手动改变字段焦点或者调用 blur()或 focus()方法而触发。这两个事件对所有表单都会一视同仁。change 事件则不然，它会因控件不同而在不同时机触发。对于`<input>`和`<textarea>`元素，change 事件会在字段失去焦点，同时 value 自控件获得焦点后发生变化时触发。对于`<select>`元素，change 事件会在用户改变了选中项时触发，不需要控件失去焦点。
    - focus 和 blur 事件通常用于以某种方式改变用户界面，以提供可见的提示或额外功能（例如在文本框下面显示下拉菜单）。change 事件通常用于验证用户在字段中输入的内容。比如，有的文本框可能只限于接收数值。focus 事件可以用来改变控件的背景颜色以便更清楚地表明当前字段获得了焦点。blur 事件可以用于去掉这个背景颜色。而 change 事件可以用于在用户输入了非数值时把背景改为红色。以下代码展示了上述操作：
        ```
        let textbox = document.forms[0].elements[0]; 
        textbox.addEventListener("focus", (event) => { 
            let target = event.target; 
            if (target.style.backgroundColor != "red") { 
                target.style.backgroundColor = "yellow"; 
            } 
        }); 
        textbox.addEventListener("blur", (event) => { 
            let target = event.target; 
            target.style.backgroundColor = /[^\d]/.test(target.value) ? "red" : ""; 
        }); 
        textbox.addEventListener("change", (event) => { 
            let target = event.target; 
            target.style.backgroundColor = /[^\d]/.test(target.value) ? "red" : ""; 
        });
        ```
    - 这里的 onfocus 事件处理程序会把文本框的背景改为黄色，更清楚地表明它是当前活动字段。onblur 和 onchange 事件处理程序会在发现非数值字符时把背景改为红色。为测试非数值字符，这里使用了一个简单的正则表达式来检测文本框的 value。这个功能必须同时在 onblur 和 onchange 事件处理程序上实现，以确保无论文本框是否改变都能执行验证。
    - 注意，blur 和 change 事件的关系并没有明确定义。在某些浏览器中，blur 事件会先于change 事件触发；在其他浏览器中，触发顺序则相反。因此不能依赖这两个事件触发的顺序，必须区分时要多加注意。

9. 文本框编程
    - 在 HTML 中有两种表示文本框的方式：单行使用`<input>`元素，多行使用`<textarea>`元素。这两个控件非常相似，大多数时候行为也一样。不过，它们也有非常重要的区别。
    - 默认情况下，`<input>`元素显示为文本框，省略 type 属性会以"text"作为默认值。然后可以通过size 属性指定文本框的宽度，这个宽度是以字符数来计量的。而 value 属性用于指定文本框的初始值，maxLength 属性用于指定文本框允许的最多字符数。因此要创建一个一次可显示 25 个字符，但最多允许显示 50 个字符的文本框，可以这样写：
        ```
        <input type="text" size="25" maxlength="50" value="initial value">
        ```
    - `<textarea>`元素总是会创建多行文本框。可以使用 rows 属性指定这个文本框的高度，以字符数计量；以 cols 属性指定以字符数计量的文本框宽度，类似于`<input>`元素的 size 属性。与`<input>`不同的是，`<textarea>`的初始值必须包含在`<textarea>`和`</textarea>`之间，如下所示：
        ```
        <textarea rows="25" cols="5">initial value</textarea>
        ```
    - 同样与`<input>`元素不同的是，`<textarea>`不能在 HTML 中指定最大允许的字符数。
    - 除了标记中的不同，这两种类型的文本框都会在 value 属性中保存自己的内容。通过这个属性，可以读取也可以设置文本模式的值，如下所示：
        ```
        let textbox = document.forms[0].elements["textbox1"]; 
        console.log(textbox.value); 
        textbox.value = "Some new value";
        ```
    - 应该使用 value 属性，而不是标准 DOM 方法读写文本框的值。比如，不要使用 setAttribute()设置`<input>`元素 value 属性的值，也不要尝试修改`<textarea>`元素的第一个子节点。对 value 属性的修改也不会总体现在 DOM 中，因此在处理文本框值的时候最好不要使用 DOM 方法。

10. 选择文本
    - 两种文本框都支持一个名为 select()的方法，此方法用于全部选中文本框中的文本。大多数浏览器会在调用 select()方法后自动将焦点设置到文本框（Opera 例外）。这个方法不接收参数，可以在任何时候调用。下面来看一个例子：
        ```
        let textbox = document.forms[0].elements["textbox1"]; 
        textbox.select();
        ```
    - 在文本框获得焦点时选中所有文本是非常常见的，特别是在文本框有默认值的情况下。这样做的出发点是让用户能够一次性删除所有默认内容。可以通过以下代码来实现：
        ```
        textbox.addEventListener("focus", (event) => { 
            event.target.select(); 
        });
        ```
    - 把以上代码应用到文本框之后，只要文本框一获得焦点就会自动选中其中的所有文本。这样可以极大提升表单易用性。

11. select 事件
    - 与 select()方法相对，还有一个 select 事件。当选中文本框中的文本时，会触发 select 事件。这个事件确切的触发时机因浏览器而异。在 IE9+、Opera、Firefox、Chrome 和 Safari 中，select 事件会在用户选择完文本后立即触发；在 IE8 及更早版本中，则会在第一个字符被选中时触发。另外，调用select()方法也会触发 select 事件。下面来看一个例子：
        ```
        let textbox = document.forms[0].elements["textbox1"]; 
        textbox.addEventListener("select", (event) => {
            console.log(`Text selected: ${textbox.value}`); 
        });
        ```

12. 取得选中文本
    - 虽然 select 事件能够表明有文本被选中，但不能提供选中了哪些文本的信息。HTML5对此进行了扩展，以方便更好地获取选中的文本。扩展为文本框添加了两个属性：selectionStart 和 selectionEnd。这两个属性包含基于 0 的数值，分别表示文本选区的起点和终点（文本选区起点的偏移量和文本选区终点的偏移量）。因此，要取得文本框中选中的文本，可以使用以下代码：
        ```
        function getSelectedText(textbox){ 
            return textbox.value.substring(textbox.selectionStart, 
                textbox.selectionEnd); 
        }
        ```
    - 因为 substring()方法是基于字符串偏移量的，所以直接传入 selectionStart 和 selectionEnd就可以取得选中的文本。
    - 这个扩展在 IE9+、Firefox、Safari、Chrome 和 Opera 中都可以使用。IE8 及更早版本不支持这两个属性，因此需要使用其他方式。
    - 老版本 IE 中有一个包含整个文档中文本选择信息的 document.selection 对象。这意味着无法确定选中的文本在页面中的什么位置。不过，在与 select 事件一起使用时，可以确定是触发这个事件文本框中选中的文本。为取得这些选中的文本，必须先创建一个范围，然后再从中提取文本，如下所示：
        ```
        function getSelectedText(textbox){ 
            if (typeof textbox.selectionStart == "number"){ 
                return textbox.value.substring(textbox.selectionStart, 
                                            textbox.selectionEnd); 
            } else if (document.selection){ 
                return document.selection.createRange().text; 
            } 
        }
        ```
    - 这个修改后的函数兼容在 IE 老版本中取得选中文本。注意 document.selection 是根本不需要textbox 参数的。

13. 部分选中文本
    - HTML5 也为在文本框中选择部分文本提供了额外支持。现在，除了 select()方法之外，Firefox最早实现的 setSelectionRange()方法也可以在所有文本框中使用。这个方法接收两个参数：要选择的第一个字符的索引和停止选择的字符的索引（与字符串的 substring()方法一样）。下面是几个例子：
        ```
        textbox.value = "Hello world!" 
        // 选择所有文本
        textbox.setSelectionRange(0, textbox.value.length); // "Hello world!" 
        // 选择前 3 个字符
        textbox.setSelectionRange(0, 3); // "Hel" 
        // 选择第 4~6 个字符
        textbox.setSelectionRange(4, 7); // "o w"
        ```
    - 如果想看到选择，则必须在调用 setSelectionRange()之前或之后给文本框设置焦点。这个方法在 IE9、Firefox、Safari、Chrome 和 Opera 中都可以使用。
    - IE8 及更早版本支持通过范围部分选中文本。这也就是说，要选择文本框中的部分文本，必须先使用 IE 在文本框上提供的 createTextRange()方法创建一个范围，并使用 moveStart()和 moveEnd()范围方法把这个范围放到正确的位置上。不过，在调用这两个方法前需要先调用 collapse()方法把范围折叠到文本框的开始。接着，moveStart()可以把范围的起点和终点都移动到相同的位置，再给moveEnd()传入要选择的字符总数作为参数。最后一步是使用范围的 select()方法选中文本，如下面的例子所示：
        ```
        textbox.value = "Hello world!"; 
        var range = textbox.createTextRange(); 
        // 选择所有文本
        range.collapse(true); 
        range.moveStart("character", 0); 
        range.moveEnd("character", textbox.value.length); // "Hello world!" 
        range.select(); 
        // 选择前 3 个字符
        range.collapse(true); 
        range.moveStart("character", 0); 
        range.moveEnd("character", 3); 
        range.select(); // "Hel" 
        // 选择第 4~6 个字符
        range.collapse(true); 
        range.moveStart("character", 4); 
        range.moveEnd("character", 6); 
        range.select(); // "o w"
        ```
    - 与其他浏览器一样，如果想要看到选中的效果，则必须让文本框获得焦点。
    - 部分选中文本对自动完成建议项等高级文本输入框是很有用的。

14. 输入过滤
    - 不同文本框经常需要保证输入特定类型或格式的数据。或许数据需要包含特定字符或必须匹配某个特定模式。由于文本框默认并未提供什么验证功能，因此必须通过 JavaScript 来实现这种输入过滤。组合使用相关事件及 DOM 能力，可以把常规的文本框转换为能够理解自己所收集数据的智能输入框。

15. 屏蔽字符
    - 有些输入框需要出现或不出现特定字符。例如，让用户输入手机号的文本框就不应该出现非数字字符。我们知道 keypress 事件负责向文本框插入字符，因此可以通过阻止这个事件的默认行为来屏蔽非数字字符。比如，下面的代码会屏蔽所有按键的输入：
        ```
        textbox.addEventListener("keypress", (event) => { 
            event.preventDefault(); 
        });
        ```
    - 运行以上代码会让文本框变成只读，因为所有按键都被屏蔽了。如果想只屏蔽特定字符，则需要检查事件的 charCode 属性，以确定正确的回应方式。例如，下面就是只允许输入数字的代码：
        ```
        textbox.addEventListener("keypress", (event) => { 
            if (!/\d/.test(String.fromCharCode(event.charCode))){ 
                event.preventDefault();
            } 
        });
        ```
    - 这个例子先用String.fromCharCode()把事件的charCode 转换为字符串，再用正则表达式/\d/来测试。这个正则表达式匹配所有数字字符，如果测试失败就调用 preventDefault()屏蔽事件默认行为。这样就可以让文本框忽略非数字输入。
    - 虽然 keypress 事件应该只在按下字符键时才触发，但某些浏览器会在按下其他键时也触发这个事件。Firefox 和 Safari（3.1 之前）会在按下上、下箭头键、退格键和删除键时触发 keypress 事件。Safari 3.1 及之后版本对这些键则不会再触发 keypress 事件。这意味着简单地屏蔽所有非数字字符还不够好，因为这样也屏蔽了上述这些非常有用的且必要的键。好在我们可以轻松检测到是否按下了这些键。在Firefox 中，所有触发 keypress 事件的非字符键的 charCode 都是 0，而在 Safari 3 之前这些键的charCode 都是 8。综合考虑这些情况，就是不能屏蔽 charCode 小于 10 的键。为此，上面的函数可以改进为：
        ```
        textbox.addEventListener("keypress", (event) => { 
            if (!/\d/.test(String.fromCharCode(event.charCode)) && 
              event.charCode > 9){ 
                event.preventDefault(); 
            } 
        });
        ```
    - 这个事件处理程序可以在所有浏览器中使用，屏蔽非数字字符但允许同样会触发 keypress 事件的所有基础按键。
    - 还有一个问题需要处理：复制、粘贴及涉及 Ctrl 键的其他功能。在除 IE 外的所有浏览器中，前面代码会屏蔽快捷键 Ctrl+C、Ctrl+V 及其他使用 Ctrl 的组合键。因此，最后一项检测是确保没有按下 Ctrl键，如下面的例子所示：
        ```
        textbox.addEventListener("keypress", (event) => { 
            if (!/\d/.test(String.fromCharCode(event.charCode)) && 
              event.charCode > 9 && 
              !event.ctrlKey){ 
                event.preventDefault(); 
            }
        });
        ```
    - 最后这个改动可以确保所有默认的文本框行为不受影响。这个技术可以用来自定义是否允许在文本框中输入某些字符。

16. 处理剪贴板
    - IE 是第一个支持剪贴板相关事件及通过 JavaScript 访问剪贴板数据的浏览器。IE 的实现成为了事实标准，这是因为 Safari、Chrome、Opera 和 Firefox 都实现了相同的事件和剪贴板访问机制，后来 HTML5也增加了剪贴板事件 。以下是与剪贴板相关的 6 个事件。
        - beforecopy：复制操作发生前触发。
        - copy：复制操作发生时触发。
        - beforecut：剪切操作发生前触发。
        - cut：剪切操作发生时触发。
        - beforepaste：粘贴操作发生前触发。
        - paste：粘贴操作发生时触发。
    - 这是一个比较新的控制剪贴板访问的标准，事件的行为及相关对象会因浏览器而异。在 Safari、Chrome 和 Firefox 中，beforecopy、beforecut 和 beforepaste 事件只会在显示文本框的上下文菜单（预期会发生剪贴板事件）时触发，但 IE 不仅在这种情况下触发，也会在 copy、cut 和 paste 事件之前触发。无论是在上下文菜单中做出选择还是使用键盘快捷键，copy、cut 和 paste 事件在所有浏览器中都会按预期触发。
    - 通过 beforecopy、beforecut 和 beforepaste 事件可以在向剪贴板发送或从中检索数据前修改数据。不过，取消这些事件并不会取消剪贴板操作。要阻止实际的剪贴板操作，必须取消 copy、cut和 paste 事件。
    - 剪贴板上的数据可以通过 window 对象（IE）或 event 对象（Firefox、Safari 和 Chrome）上的clipboardData 对象来获取。在 Firefox、Safari 和 Chrome 中，为防止未经授权访问剪贴板，只能在剪贴板事件期间访问 clipboardData 对象；IE 则在任何时候都会暴露 clipboardData 对象。为了跨浏览器兼容，最好只在剪贴板事件期间使用这个对象。
    - clipboardData 对象上有 3 个方法：getData()、setData()和 clearData()，其中 getData()方法从剪贴板检索字符串数据，并接收一个参数，该参数是要检索的数据的格式。IE 为此规定了两个选项："text"和"URL"。Firefox、Safari 和 Chrome 则期待 MIME 类型，不过会将"text"视为等价于"text/plain"。
    - setData()方法也类似，其第一个参数用于指定数据类型，第二个参数是要放到剪贴板上的文本。同样，IE 支持"text"和"URL"，Safari 和 Chrome 则期待 MIME 类型。不过，与 getData()不同的是，Safari 和 Chrome 不认可"text"类型。只有在 IE8 及更早版本中调用 setData()才有效，其他浏览器会忽略对这个方法的调用。为抹平差异，可以使用以下跨浏览器的方法：
        ```
        function getClipboardText(event){ 
            var clipboardData = (event.clipboardData || window.clipboardData); 
            return clipboardData.getData("text"); 
        } 
        function setClipboardText (event, value){ 
            if (event.clipboardData){ 
                return event.clipboardData.setData("text/plain", value); 
            } else if (window.clipboardData){ 
                return window.clipboardData.setData("text", value); 
            } 
        }
        ```
    - 这里的 getClipboardText()函数相对简单，它只需要知道 clipboardData 对象在哪里，然后便可以通过"text"类型调用 getData()。相应的，setClipboardText()函数则要复杂一些。在确定clipboardData 对象的位置之后，需要根据实现以相应的类型（Firefox、Safari 和 Chrome 是"text/plain"，而 IE 是"text"）调用 setData()。
    - 如果文本框期待某些字符或某种格式的文本，那么从剪贴板中读取文本是有帮助的。比如，如果文本框只允许输入数字，那么就必须检查粘贴过来的值，确保其中只包含数字。在 paste 事件中，可以确定剪贴板上的文本是否无效，如果无效就取消默认行为，如下面的例子所示：
        ```
        textbox.addEventListener("paste", (event) => { 
            let text = getClipboardText(event); 
            if (!/^\d*$/.test(text)){ 
                event.preventDefault();
            } 
        });
        ```
    - 这个 onpaste 事件处理程序确保只有数字才能粘贴到文本框中。如果剪贴板中的值不符合指定模式，则取消粘贴操作。Firefox、Safari 和 Chrome 只允许在 onpaste 事件处理程序中访问 getData()方法。
    - 因为不是所有浏览器都支持剪贴板访问，所以有时候更容易屏蔽一个或多个剪贴板操作。在支持copy、cut 和 paste 事件的浏览器（IE、Safari、Chrome 和 Firefox）中，很容易阻止事件的默认行为。在 Opera 中，则需要屏蔽导致相应事件的按键，同时阻止显示相应的上下文菜单。

17. 自动切换
    - JavaScript 可以通过很多方式来增强表单字段的易用性。最常用的是在当前字段完成时自动切换到下一个字段。对于要收集数据的长度已知（比如电话号码）的字段是可以这样处理的。在美国，电话号码通常分为 3 个部分：区号、交换局号，外加 4 位数字。在网页中，可以通过 3 个文本框来表示这几个部分，比如：
        ```
        <input type="text" name="tel1" id="txtTel1" maxlength="3"> 
        <input type="text" name="tel2" id="txtTel2" maxlength="3"> 
        <input type="text" name="tel3" id="txtTel3" maxlength="4">
        ```
    - 为增加这个表单的易用性并加速数据输入，可以在每个文本框输入到最大允许字符数时自动把焦点切换到下一个文本框。因此，当用户在第一个文本框中输入 3 个字符后，就把焦点移到第二个文本框，当用户在第二个文本框中输入 3 个字符后，把焦点再移到第三个文本框。这种自动切换文本框的行为可以通过如下代码实现：
        ```
        <script> 
            function tabForward(event){ 
                let target = event.target; 
                if (target.value.length == target.maxLength){ 
                    let form = target.form; 
                    for (let i = 0, len = form.elements.length; i < len; i++) { 
                        if (form.elements[i] == target) { 
                            if (form.elements[i+1]) { 
                                form.elements[i+1].focus(); 
                            } 
                            return; 
                        } 
                    } 
                } 
            } 
            let inputIds = ["txtTel1", "txtTel2", "txtTel3"]; 
            for (let id of inputIds) { 
                let textbox = document.getElementById(id); 
                textbox.addEventListener("keyup", tabForward); 
            } 
            let textbox1 = document.getElementById("txtTel1"); 
            let textbox2 = document.getElementById("txtTel2"); 
            let textbox3 = document.getElementById("txtTel3"); 
        </script>
        ```
    - 这个 tabForward()函数是实现自动切换的关键。它通过比较用户输入文本的长度与 maxlength属性的值来检测输入是否达到了最大长度。如果两者相等（因为浏览器会强制最大字符数，所以不可能出现多的情况），那么就要通过循环表单中的元素集合找到当前文本框，并把焦点设置到下一个元素。这个函数接着给每一个文本框都指定了 onkeyup 事件处理程序。因为 keyup 事件会在每个新字符被插入到文本框中时触发，所以此时应该是检测文本框内容长度的最佳时机。在填写这个简单的表单时，用户不用按 Tab 键切换字段和提交表单。
    - 不过要注意，上面的代码只适用于之前既定的标记，没有考虑可能存在的隐藏字段。

18. HTML5 约束验证 API
    - HTML5为浏览器新增了在提交表单前验证数据的能力。这些能力实现了基本的验证，即使JavaScript不可用或加载失败也没关系。这是因为浏览器自身会基于指定的规则进行验证，并在出错时显示适当的错误消息（无须 JavaScript）。这些能力只有支持 HTML5 这部分的浏览器才有，包括所有现代浏览器（除了 Safari）和 IE10+。
    - 验证会根据某些条件应用到表单字段。可以使用 HTML 标记指定对特定字段的约束，然后浏览器会根据这些约束自动执行表单验证。

19. 必填字段
    - 第一个条件是给表单字段添加 required 属性，如下所示：
        ```
        <input type="text" name="username" required>
        ```
    - 任何带有 required 属性的字段都必须有值，否则无法提交表单。这个属性适用于`<input>`、`<textarea>`和`<select>`字段（Opera 直到版本 11 都不支持`<select>`的 required 属性）。可以通过JavaScript 检测对应元素的 required 属性来判断表单字段是否为必填：
        ```
        let isUsernameRequired = document.forms[0].elements["username"].required;
        ```
    - 还可以使用下面的代码检测浏览器是否支持 required 属性：
        ```
        let isRequiredSupported = "required" in document.createElement("input"); 
        ```
    - 这行代码使用简单的特性检测来确定新创建的`<input>`元素上是否存在 required 属性。
    - 注意，不同浏览器处理必填字段的机制不同。Firefox、Chrome、IE 和 Opera 会阻止表单提交并在相应字段下面显示有帮助信息的弹框，而 Safari 什么也不做，也不会阻止提交表单。

20. 更多输入类型
    - HTML5 为`<input>`元素增加了几个新的 type 值。这些类型属性不仅表明了字段期待的数据类型，而且也提供了一些默认验证，其中两个新的输入类型是已经得到广泛支持的"email"和"url"，二者都有浏览器提供的自定义验证。比如：
        ```
        <input type="email" name="email"> 
        <input type="url" name="homepage">
        ```
    - "email"类型确保输入的文本匹配电子邮件地址，而"url"类型确保输入的文本匹配 URL。注意，浏览器在匹配模式时都存在问题。最明显的是文本"-@-"会被认为是有效的电子邮件地址。浏览器厂商仍然在解决这些问题。
    - 要检测浏览器是否支持这些新类型，可以在 JavaScript 中新创建一个输入元素并将其类型属性设置为"email"或"url"，然后再读取该元素的值。老版本浏览器会自动将未知类型值设置为"text"，而支持的浏览器会返回正确的值。比如：
        ```
        let input = document.createElement("input"); 
        input.type = "email"; 
        let isEmailSupported = (input.type == "email");
        ```
    - 对于这两个新类型，除非应用了 required 属性，否则空字段是有效的。另外，指定一个特殊输入类型并不会阻止用户输入无效的值。新类型只是会应用一些默认验证。

21. 数值范围
    - 除了"email"和"url"，HTML5 还定义了其他几种新的输入元素类型，它们都是期待某种数值输入的，包括："number"、"range"、"datetime"、"datetime-local"、"date"、"month"、"week"和"time"。并非所有主流浏览器都支持这些类型，因此使用时要当心。浏览器厂商目前正致力于解决兼容性问题和提供更逻辑化的功能。本节内容更多地是介绍未来趋势，而不是讨论当前就能用的功能。
    - 对上述每种数值类型，都可以指定 min 属性（最小可能值）、max 属性（最大可能值），以及 step属性（从 min 到 max 的步长值）。例如，如果只允许输入 0 到 100 中 5 的倍数，那么可以这样写：
        ```
        <input type="number" min="0" max="100" step="5" name="count">
        ```
    - 根据浏览器的不同，可能会也可能不会出现旋转控件（上下按钮）用于自动增加和减少。
    - 上面每个属性在 JavaScript 中也可以通过对应元素的 DOM 属性来访问和修改。此外，还有两个方法，即 stepUp()和 stepDown()。这两个方法都接收一个可选的参数：要从当前值加上或减去的数值。（默认情况下，步长值会递增或递减 1。）虽然浏览器还没有实现这些方法，但可以先看一下它们的用法：
        ```
        input.stepUp(); // 加 1 
        input.stepUp(5); // 加 5 
        input.stepDown(); // 减 1 
        input.stepDown(10); // 减 10
        ```

22. 输入模式
    - HTML5 为文本字段新增了 pattern 属性。这个属性用于指定一个正则表达式，用户输入的文本必须与之匹配。例如，要限制只能在文本字段中输入数字，可以这样添加模式：
        ```
        <input type="text" pattern="\d+" name="count">
        ```
    - 注意模式的开头和末尾分别假设有^和$。这意味着输入内容必须从头到尾都严格与模式匹配。
    - 与新增的输入类型一样，指定 pattern 属性也不会阻止用户输入无效内容。模式会应用到值，然后浏览器会知道值是否有效。通过访问 pattern 属性可以读取模式：
        ```
        let pattern = document.forms[0].elements["count"].pattern;
        ```
    - 使用如下代码可以检测浏览器是否支持 pattern 属性：
        ```
        let isPatternSupported = "pattern" in document.createElement("input");
        ```

23. 检测有效性
    - 使用 checkValidity()方法可以检测表单中任意给定字段是否有效。这个方法在所有表单元素上都可以使用，如果字段值有效就会返回 true，否则返回 false。判断字段是否有效的依据是本节前面提到的约束条件，因此必填字段如果没有值就会被视为无效，而字段值不匹配 pattern 属性也会被视为无效。比如：
        ```
        if (document.forms[0].elements[0].checkValidity()){ 
            // 字段有效，继续
        } else { 
            // 字段无效
        }
        ```
    - 要检查整个表单是否有效，可以直接在表单上调用 checkValidity()方法。这个方法会在所有字段都有效时返回 true，有一个字段无效就会返回 false：
        ```
        if(document.forms[0].checkValidity()){ 
            // 表单有效，继续
        } else { 
            // 表单无效
        }
        ```
    - checkValidity()方法只会告诉我们字段是否有效，而 validity 属性会告诉我们字段为什么有效或无效。这个属性是一个对象，包含一系列返回布尔值的属性。
        - customError：如果设置了 setCustomValidity()就返回 true，否则返回 false。
        - patternMismatch：如果字段值不匹配指定的 pattern 属性则返回 true。
        - rangeOverflow：如果字段值大于 max 的值则返回 true。
        - rangeUnderflow：如果字段值小于 min 的值则返回 true。
        - stepMisMatch：如果字段值与 min、max 和 step 的值不相符则返回 true。
        - tooLong：如果字段值的长度超过了 maxlength 属性指定的值则返回 true。某些浏览器，如Firefox 4 会自动限制字符数量，因此这个属性值始终为 false。
        - typeMismatch：如果字段值不是"email"或"url"要求的格式则返回 true。
        - valid：如果其他所有属性的值都为 false 则返回 true。与 checkValidity()的条件一致。
        - valueMissing：如果字段是必填的但没有值则返回 true。
    - 因此，通过 validity 属性可以检查表单字段的有效性，从而获取更具体的信息，如下面的代码所示：
        ```
        if (input.validity && !input.validity.valid){ 
            if (input.validity.valueMissing){ 
                console.log("Please specify a value.") 
            } else if (input.validity.typeMismatch){ 
                console.log("Please enter an email address."); 
            } else { 
                console.log("Value is invalid."); 
            } 
        }
        ```
        
24. 禁用验证
    - 通过指定 novalidate 属性可以禁止对表单进行任何验证：
        ```
        <form method="post" action="/signup" novalidate> 
            <!-- 表单元素 --> 
        </form>
        ```
    - 这个值也可以通过 JavaScript 属性 noValidate 检索或设置，设置为 true 表示属性存在，设置为false 表示属性不存在：
        ```
        document.forms[0].noValidate = true; // 关闭验证
        ```
    - 如果一个表单中有多个提交按钮，那么可以给特定的提交按钮添加 formnovalidate 属性，指定通过该按钮无须验证即可提交表单：
        ```
        <form method="post" action="/foo"> 
            <!-- 表单元素 --> 
            <input type="submit" value="Regular Submit"> 
            <input type="submit" formnovalidate name="btnNoValidate" 
                value="Non-validating Submit"> 
        </form>
        ```
    - 在这个例子中，第一个提交按钮会让表单像往常一样验证数据，第二个提交按钮则禁用了验证，可以直接提交表单。我们也可以使用 JavaScript 来设置这个属性：
        ```
        // 关闭验证
        document.forms[0].elements["btnNoValidate"].formNoValidate = true;
        ```

25. 选择框编程
    - 选择框是使用`<select>`和`<option>`元素创建的。为方便交互，HTMLSelectElement 类型在所有表单字段的公共能力之外又提供了以下属性和方法。
        - add(newOption, relOption)：在 relOption 之前向控件中添加新的`<option>`。
        - multiple：布尔值，表示是否允许多选，等价于 HTML 的 multiple 属性。
        - options：控件中所有`<option>`元素的 HTMLCollection。
        - remove(index)：移除给定位置的选项。
        - selectedIndex：选中项基于 0 的索引值，如果没有选中项则为–1。对于允许多选的列表，始终是第一个选项的索引。
        - size：选择框中可见的行数，等价于 HTML 的 size 属性。选择框的 type 属性可能是"select-one"或"select-multiple"，具体取决于 multiple 属性是否存在。当前选中项根据以下规则决定选择框的 value 属性。
        - 如果没有选中项，则选择框的值是空字符串。
        - 如果有一个选中项，且其 value 属性有值，则选择框的值就是选中项 value 属性的值。即使value 属性的值是空字符串也是如此。
        - 如果有一个选中项，且其 value 属性没有指定值，则选择框的值是该项的文本内容。
        - 如果有多个选中项，则选择框的值根据前两条规则取得第一个选中项的值。
    - 来看下面的选择框：
        ```
        <select name="location" id="selLocation"> 
            <option value="Sunnyvale, CA">Sunnyvale</option> 
            <option value="Los Angeles, CA">Los Angeles</option> 
            <option value="Mountain View, CA">Mountain View</option> 
            <option value="">China</option> 
            <option>Australia</option> 
        </select>
        ```
    - 如果选中这个选择框中的第一项，则字段的值就是"Sunnyvale, CA"。如果文本为"China"的项被选中，则字段的值是一个空字符串，因为该项的 value 属性是空字符串。如果选中最后一项，那么字段的值是"Australia"，因为该`<option>`元素没有指定 value 属性。
    - 每个`<option>`元素在 DOM 中都由一个 HTMLOptionElement 对象表示。HTMLOptionElement类型为方便数据存取添加了以下属性。
        - index：选项在 options 集合中的索引。
        - label：选项的标签，等价于 HTML 的 label 属性。
        - selected：布尔值，表示是否选中了当前选项。把这个属性设置为 true 会选中当前选项。
        - text：选项的文本。
        - value：选项的值（等价于 HTML 的 value 属性）。
    - 大多数`<option>`属性是为了方便存取选项数据。可以使用常规 DOM 功能存取这些信息，只是效率比较低，如下面的例子所示：
        ```
        let selectbox = document.forms[0].elements["location"]; 
        // 不推荐
        let text = selectbox.options[0].firstChild.nodeValue; // 选项文本
        let value = selectbox.options[0].getAttribute("value"); // 选项值
        ```
    - 以上代码使用标准的 DOM 技术获取了选择框中第一个选项的文本和值。下面再比较一下使用特殊选项属性的代码：
        ```
        let selectbox = document.forms[0].elements["location"]; 
        // 推荐
        let text = selectbox.options[0].text; // 选项文本
        let value = selectbox.options[0].value; // 选项值
        ```
    - 在操作选项时，最好使用特定于选项的属性，因为这些属性得到了跨浏览器的良好支持。在操作DOM 节点时，与表单控制实际的交互可能会因浏览器而异。不推荐使用标准 DOM 技术修改`<option>`元素的文本和值。
    - 最后强调一下，选择框的 change 事件与其他表单字段是不一样的。其他表单字段会在自己的值改变后触发 change 事件，然后字段失去焦点。而选择框会在选中一项时立即触发 change 事件。
    - 注意，不同浏览器返回的 value 属性可能会有差异。JavaScript 中的 value 属性始终等于HTML 中的 value 属性。但在 HTML 中没有指定 value 属性的情况下，IE8 及早期版本会返回空字符串，而 IE9 及之后版本、Safari、Firefox、Chrome 和 Opera 会返回与 text相同的值。

26. 选项处理
    - 对于只允许选择一项的选择框，获取选项最简单的方式是使用选择框的 selectedIndex 属性，如下面的例子所示：
        ```
        let selectedOption = selectbox.options[selectbox.selectedIndex];
        ```
    - 这样可以获取关于选项的所有信息，比如：
        ```
        let selectedIndex = selectbox.selectedIndex; 
        let selectedOption = selectbox.options[selectedIndex]; 
        console.log(`Selected index: ${selectedIndex}\n` + 
                `Selected text: ${selectedOption.text}\n` + 
                `Selected value: ${selectedOption.value}`);
        ```
    - 以上代码打印出了选中项的索引及其文本和值。
    - 对于允许多选的选择框，selectedIndex 属性就像只允许选择一项一样。设置 selectedIndex会移除所有选项，只选择指定的项，而获取 selectedIndex 只会返回选中的第一项的索引。
    - 选项还可以通过取得选项的引用并将其 selected 属性设置为 true 来选中。例如，以下代码会选中选择框中的第一项：
        ```
        selectbox.options[0].selected = true;
        ```
    - 与 selectedIndex 不同，设置选项的 selected 属性不会在多选时移除其他选项，从而可以动态选择任意多个选项。如果修改单选框中选项的 selected 属性，则其他选项会被移除。要注意的是，把selected 属性设置为 false 对单选框没有影响。
    - 通过 selected 属性可以确定选择框中哪个选项被选中。要取得所有选中项，需要循环选项集合逐一检测 selected 属性，比如：
        ```
        function getSelectedOptions(selectbox){ 
            let result = new Array(); 
            for (let option of selectbox.options) { 
                if (option.selected) { 
                    result.push(option); 
                } 
            } 
            return result; 
        }
        ```
    - 这个函数会返回给定选择框中所有选中项的数组。首先创建一个包含结果的数组，然后通过 for 循环迭代所有选项，检测每个选项的 selected 属性。如果选项被选中，就将其添加到 result 数组。最后是返回选中项数组。这个 getSelectedOptions()函数可以用于获取选中项的信息，比如：
        ```
        let selectbox = document.getElementById("selLocation"); 
        let selectedOptions = getSelectedOptions(selectbox); 
        let message = ""; 
        for (let option of selectedOptions) { 
            message += 'Selected index: ${option.index}\n' + 
                        'Selected text: ${option.text}\n' + 
                        'Selected value: ${option.value}\n' 
        } 
        console.log(message);
        ```
    - 这个例子先检索了一个选择框的所有选中项。然后通过 for 循环构建包含所有选中项信息的字符串，包括每项的索引、文本和值。以上代码既适用于单选框也适用于多选框。

27. 添加选项
    - 可以使用 JavaScript 动态创建选项并将它们添加到选择框。首先，可以使用 DOM 方法，如下所示：
        ```
        let newOption = document.createElement("option"); 
        newOption.appendChild(document.createTextNode("Option text")); 
        newOption.setAttribute("value", "Option value"); 
        selectbox.appendChild(newOption);
        ```
    - 以上代码创建了一个新的`<option>`元素，使用文本节点添加文本，设置其 value 属性，然后将其添加到选择框。添加到选择框之后，新选项会立即显示出来。
    - 另外，也可以使用 Option 构造函数创建新选项，这个构造函数是 DOM 出现之前就已经得到浏览器支持的。Option 构造函数接收两个参数：text 和 value，其中 value 是可选的。虽然这个构造函数通常会创建 Object 的实例，但 DOM 合规的浏览器都会返回一个`<option>`元素。这意味着仍然可以使用 appendChild()方法把这样创建的选项添加到选择框。比如下面的例子：
        ```
        let newOption = new Option("Option text", "Option value"); 
        selectbox.appendChild(newOption); // 在 IE8 及更低版本中有问题
        ```
    - 这个方法在除 IE8 及更低版本之外的所有浏览器中都没有问题。由于实现问题，IE8 及更低版本在这种情况下不能正确设置新选项的文本。
    - 另一种添加新选项的方式是使用选择框的 add()方法。DOM 规定这个方法接收两个参数：要添加的新选项和要添加到其前面的参考选项。如果想在列表末尾添加选项，那么第二个参数应该是 null。IE8 及更早版本对 add()方法的实现稍有不同，其第二个参数是可选的，如果要传入则必须是一个索引值，表示要在其前面添加新选项的选项。DOM 合规的浏览器要求必须传入第二个参数，因此在跨浏览器方法中不能只使用一个参数（IE9 是符合 DOM 规范的）。此时，传入 undefined 作为第二个参数可以保证在所有浏览器中都将选项添加到列表末尾。下面是一个例子：
        ```
        let newOption = new Option("Option text", "Option value"); 
        selectbox.add(newOption, undefined); // 最佳方案
        ```
    - 以上代码可以在所有版本的 IE 及 DOM 合规的浏览器中使用。如果不想在最后插入新选项，则应该使用 DOM 技术和 insertBefore()。
    - 注意，跟在 HTML 中一样，选项的值不是必需的。Option 构造函数也可以只接收一个参数（选项的文本）。

28. 移除选项
    - 与添加选项类似，移除选项的方法也不止一种。
    - 第一种方式是使用 DOM 的 removeChild()方法并传入要移除的选项，比如：
        ```
        selectbox.removeChild(selectbox.options[0]); // 移除第一项
        ```
    - 第二种方式是使用选择框的 remove()方法。这个方法接收一个参数，即要移除选项的索引，比如：
        ```
        selectbox.remove(0); // 移除第一项
        ```
    - 最后一种方式是直接将选项设置为等于 null。这同样也是 DOM 之前浏览器实现的方式。下面是一个例子：
        ```
        selectbox.options[0] = null; // 移除第一项
        ```
    - 要清除选择框的所有选项，需要迭代所有选项并逐一移除它们，如下面例子所示：
        ```
        function clearSelectbox(selectbox) { 
            for (let option of selectbox.options) { 
                selectbox.remove(0); 
            } 
        }
        ```
        - 这个函数可以逐一移除选择框中的每一项。因为移除第一项会自动将所有选项向前移一位，所以这样就可以移除所有选项。

29. 移动和重排选项
    - 在 DOM 之前，从一个选择框向另一个选择框移动选项是非常麻烦的，要先从第一个选择框移除选项，然后以相同文本和值创建新选项，再将新选项添加到第二个选择框。DOM 方法则可以直接将某个选项从第一个选择框移动到第二个选择框，只要对相应选项使用 appendChild()方法即可。如果给这个方法传入文档中已有的元素，则该元素会先从其父元素中移除，然后再插入指定位置。例如，下面的代码会从选择框中移除第一项并插入另一个选择框：
        ```
        let selectbox1 = document.getElementById("selLocations1"); 
        let selectbox2 = document.getElementById("selLocations2"); 
        selectbox2.appendChild(selectbox1.options[0]);
        ```
    - 移动选项和移除选项都会导致每个选项的 index 属性重置。
    - 重排选项非常类似，DOM 方法同样是最佳途径。要将选项移动到选择框中的特定位置，insertBefore()方法是最合适的。不过，要把选项移动到最后，还是 appendChild()方法比较方便。下面的代码演示了将一个选项在选择框中前移一个位置：
        ```
        let optionToMove = selectbox.options[1]; 
        selectbox.insertBefore(optionToMove, 
                selectbox.options[optionToMove.index-1]);
        ```
    - 这个例子首先获得要移动选项的索引，然后将其插入之前位于它前面的选项之前，其中第二行代码适用于除第一个选项之外的所有选项。下面的代码则可以将选项向下移动一个位置：
        ```
        let optionToMove = selectbox.options[1]; 
        selectbox.insertBefore(optionToMove, 
                            selectbox.options[optionToMove.index+2]);
        ```
    - 以上代码适用于选择框中的所有选项，包括最后一个。

30. 表单序列化
    - 随着 Ajax（第 21 章会进一步讨论）的崭露头角，表单序列化（form serialization）已经成为一个常见需求。表单在 JavaScript 中可以使用表单字段的 type 属性连同其 name 属性和 value 属性来进行序列化。在写代码之前，我们需要理解浏览器如何确定在提交表单时要把什么发送到服务器。
        - 字段名和值是 URL 编码的并以和号（&）分隔。
        - 禁用字段不会发送。
        - 复选框或单选按钮只在被选中时才发送。
        - 类型为"reset"或"button"的按钮不会发送。
        - 多选字段的每个选中项都有一个值。
        - 通过点击提交按钮提交表单时，会发送该提交按钮；否则，不会发送提交按钮。类型为"image"的`<input>`元素视同提交按钮。
        - `<select>`元素的值是被选中`<option>`元素的 value 属性。如果`<option>`元素没有 value 属性，则该值是它的文本。
    - 表单序列化通常不包含任何按钮，因为序列化得到的字符串很可能以其他方式提交。除此之外其他规则都应该遵循。最终完成表单序列化的代码如下：
        ```
        function serialize(form) { 
            let parts = []; 
            let optValue; 
            for (let field of form.elements) { 
                switch(field.type) { 
                    case "select-one": 
                    case "select-multiple": 
                        if (field.name.length) { 
                            for (let option of field.options) { 
                                if (option.selected) { 
                                    if (option.hasAttribute) { 
                                        optValue = (option.hasAttribute("value") ? 
                                            option.value : option.text); 
                                    } else { 
                                        optValue = (option.attributes["value"].specified ? 
                                            option.value : option.text); 
                                    } 
                                    parts.push(encodeURIComponent(field.name) + "=" + 
                                                encodeURIComponent(optValue)); 
                                } 
                            } 
                        } 
                        break; 
                    case undefined: // 字段集
                    case "file": // 文件输入
                    case "submit": // 提交按钮
                    case "reset": // 重置按钮
                    case "button": // 自定义按钮
                        break; 
                    case "radio": // 单选按钮
                    case "checkbox": // 复选框
                        if (!field.checked) { 
                            break; 
                        } 
                    default: 
                        // 不包含没有名字的表单字段
                        if (field.name.length) { 
                            parts.push('${encodeURIComponent(field.name)}=' + 
                                    '${encodeURIComponent(field.value)}'); 
                        } 
                }
            } 
            return parts.join("&"); 
        }
        ```
    - 这个 serialize()函数一开始定义了一个名为 parts 的数组，用于保存要创建字符串的各个部分。接下来通过 for 循环迭代每个表单字段，将字段保存在 field 变量中。获得一个字段的引用后，再通过 switch 语句检测其 type 属性。最麻烦的是序列化`<select>`元素，包括单选和多选两种模式。在遍历选择框的每个选项时，只要有选项被选中，就将其添加到结果字符串。单选控件只会有一个选项被选中，多选控件则可能有零或多个选项被选中。同样的代码适用于两种选择类型，因为浏览器会限制可选项的数量。找到选中项时，需要确定使用哪个值。如果不存在 value 属性，则应该以选项文本代替，不过 value 属性为空字符串是完全有效的。为此需要使用 DOM 合规的浏览器支持的 hasAttribute()方法，而在 IE8 及更早版本中要使用值的 specified 属性。
    - 表单中如果有`<fieldset>`元素，它就会出现在元素集合中，但应该没有 type 属性。因此，如果type 属性是 undefined，则不必纳入序列化。各种类型的按钮以及文件输入字段也是如此。（文件输入字段在提交表单时包含文件的内容，但这些字段通常无法转换，因而也要排除在序列化之外。）对于单选按钮和复选框，会检测其 checked 属性。如果值为 false 就退出 switch 语句；如果值为 true，则继续执行 default 分支，将字段的名和值编码后添加到 parts 数组。注意，所有没有名字的表单字段都不会包含在序列化结果中以模拟浏览器的表单提交行为。这个函数的最后一步是使用 join()通过和号把所有字段的名值对拼接起来。serialize()函数返回的结果是查询字符串格式。如果想要返回其他格式，修改起来也很简单。

31. 富文本编辑
    - 在网页上编写富文本内容是 Web 应用开发中很常见的需求。富文本编辑也就是所谓的“所见即所得”（WYSIWYG，What You See Is What You Get）编辑。虽然没有规范定义，但源自 IE 的一套事实标准已经被 Opera、Safari、Chrome 和 Firefox 所支持。基本的技术就是在空白 HTML 文件中嵌入一个iframe。通过 designMode 属性，可以将这个空白文档变成可以编辑的，实际编辑的则是`<body>`元素的 HTML。designMode 属性有两个可能的值："off"（默认值）和"on"。设置为"on"时，整个文档都会变成可以编辑的（显示插入光标），从而可以像使用文字处理程序一样编辑文本，通过键盘将文本标记为粗体、斜体，等等。
    - 作为 iframe 源的是一个非常简单的空白 HTML 页面。下面是一个例子：
        ```
        <!DOCTYPE html> 
        <html> 
            <head> 
                <title>Blank Page for Rich Text Editing</title> 
            </head> 
            <body> 
            </body> 
        </html>
        ```
    - 这个页面会像其他任何页面一样加载到 iframe 里。为了可以编辑，必须将文档的 designMode 属性设置为"on"。不过，只有在文档完全加载之后才可以设置。在这个包含页面内，需要使用 onload事件处理程序在适当时机设置 designMode，如下面的例子所示：
        ```
        <iframe name="richedit" style="height: 100px; width: 100px"></iframe> 
        <script> 
            window.addEventListener("load", () => { 
                frames["richedit"].document.designMode = "on"; 
            }); 
        </script>
        ```
    - 以上代码加载之后，可以在页面上看到一个类似文本框的区域。这个框的样式具有网页默认样式，不过可以通过 CSS 调整。

32. 使用 contenteditable
    - 还有一种处理富文本的方式，也是 IE 最早实现的，即指定 contenteditable 属性。可以给页面中的任何元素指定 contenteditable 属性，然后该元素会立即被用户编辑。这种方式更受欢迎，因为不需要额外的 iframe、空页面和 JavaScript，只给元素添加一个 contenteditable 属性即可，比如：
        ```
        <div class="editable" id="richedit" contenteditable></div>
        ```
    - 元素中包含的任何文本都会自动被编辑，元素本身类似于`<textarea>`元素。通过设置contentEditable 属性，也可以随时切换元素的可编辑状态：
        ```
        let div = document.getElementById("richedit"); 
        richedit.contentEditable = "true";
        ```
    - contentEditable 属性有 3 个可能的值："true"表示开启，"false"表示关闭，"inherit"表示继承父元素的设置（因为在 contenteditable 元素内部会创建和删除元素）。IE、Firefox、Chrome、Safari 和 Opera 及所有主流移动浏览器都支持 contentEditable 属性。
    - 注意，contenteditable 是一个非常多才多艺的属性。比如，访问伪 URL data:text/html, `<html contenteditable>`可以把浏览器窗口转换为一个记事本。这是因为这样会临时创建 DOM 树并将整个文档变成可编辑区域。

33. 与富文本交互
    - 与富文本编辑器交互的主要方法是使用 document.execCommand()。这个方法在文档上执行既定的命令，可以实现大多数格式化任务。document.execCommand()可以接收 3 个参数：要执行的命令、表示浏览器是否为命令提供用户界面的布尔值和执行命令必需的值（如果不需要则为 null）。为跨浏览器兼容，第二个参数应该始终为 false，因为 Firefox 会在其为 true 时抛出错误。
    - 不同浏览器支持的命令也不一样。下表列出了最常用的命令。

        命令 | 值（第三个参数） | 说明
        ---|---|---
        backcolor | 颜色字符串 | 设置文档背景颜色
        bold | null | 切换选中文本的粗体样式
        copy | null | 将选中文本复制到剪贴板
        createlink | URL 字符串 | 将当前选中文本转换为指向给定 URL 的链接
        cut | null | 将选中文本剪切到剪贴板
        delete | null | 删除当前选中的文本
        fontname | 字体名 | 将选中文本改为使用指定字体
        fontsize | 1~7 | 将选中文本改为指定字体大小
        forecolor | 颜色字符串 | 将选中文本改为指定颜色
        formatblock | HTML 标签，如`<h1>` | 将选中文本包含在指定的 HTML 标签中
        indent | null | 缩进文本
        inserthorizontalrule | null | 在光标位置插入`<hr>`元素
        insertimage | 图片 URL | 在光标位置插入图片
        insertorderedlist | null | 在光标位置插入`<ol>`元素
        insertparagraph | null | 在光标位置插入`<p>`元素
        insertunorderedlist | null | 在光标位置插入`<ul>`元素
        italic | null | 切换选中文本的斜体样式
        justifycenter | null | 在光标位置居中文本块
        justifyleft | null | 在光标位置左对齐文本块
        outdent | null | 减少缩进
        paste | null | 在选中文本上粘贴剪贴板内容
        removeformat | null | 移除包含光标所在位置块的 HTML 标签。这是 formatblock 的反操作
        selectall | null | 选中文档中所有文本
        underline | null | 切换选中文本的下划线样式
        unlink | null | 移除文本链接。这是 createlink 的反操作

    - 剪贴板相关的命令与浏览器关系密切。虽然这些命令并不都可以通过 document.execCommand()使用，但相应的键盘快捷键都是可以用的。
    - 这些命令可以用于修改内嵌窗格（iframe）中富文本区域的外观，如下面的例子所示：
        ```
        // 在内嵌窗格中切换粗体文本样式
        frames["richedit"].document.execCommand("bold", false, null); 
        // 在内嵌窗格中切换斜体文本样式
        frames["richedit"].document.execCommand("italic", false, null); 
        // 在内嵌窗格中创建指向 www.wrox.com 的链接
        frames["richedit"].document.execCommand("createlink", false, 
        "http://www.wrox.com"); 
        // 在内嵌窗格中为内容添加<h1>标签
        frames["richedit"].document.execCommand("formatblock", false, "<h1>");
        ```
    - 同样的方法也可以用于页面中添加了 contenteditable 属性的元素，只不过要使用当前窗口而不是内嵌窗格中的 document 对象：
        ```
        // 切换粗体文本样式
        document.execCommand("bold", false, null); 
        // 切换斜体文本样式
        document.execCommand("italic", false, null); 
        // 创建指向 www.wrox.com 的链接
        document.execCommand("createlink", false, "http://www.wrox.com"); 
        // 为内容添加<h1>标签
        document.execCommand("formatblock", false, "<h1>");
        ```
    - 注意，即使命令是所有浏览器都支持的，命令生成的 HTML 通常差别也很大。例如，为选中文本应用 bold 命令在 IE 和 Opera 中会使用`<strong>`标签，在 Safari 和 Chrome 中会使用`<b>`标签，而在Firefox 中会使用`<span>`标签。在富文本编辑中，不能依赖浏览器生成的 HTML，因为命令实现和格式转换都是通过 innerHTML 完成的。
    - 还有与命令相关的其他一些方法。第一个方法是 queryCommandEnabled()，此方法用于确定对当前选中文本或光标所在位置是否可以执行相关命令。它只接收一个参数，即要检查的命令名。如果可编辑区可以执行该命令就返回 true，否则返回 false。来看下面的例子：
        ```
        let result = frames["richedit"].document.queryCommandEnabled("bold");
        ```
    - 以上代码在当前选区可以执行"bold"命令时返回 true。不过要注意，queryCommandEnabled()返回 true 并不代表允许执行相关命令，只代表当前选区适合执行相关命令。在 Firefox 中，queryCommandEnabled("cut")即使默认不允许剪切也会返回 true。
    - 另一个方法 queryCommandState()用于确定相关命令是否应用到了当前文本选区。例如，要确定当前选区的文本是否为粗体，可以这样：
        ```
        let isBold = frames["richedit"].document.queryCommandState("bold");
        ```
    - 如果之前给文本选区应用过"bold"命令，则以上代码返回 true。全功能富文本编辑器可以利用这个方法更新粗体、斜体等按钮。
    - 最后一个方法是 queryCommandValue()，此方法可以返回执行命令时使用的值（即前面示例的execCommand()中的第三个参数）。如果对一段选中文本应用了值为 7 的"fontsize"命令，则如下代码会返回 7：
        ```
        let fontSize = frames["richedit"].document.queryCommandValue("fontsize");
        ```
    - 这个方法可用于确定如何将命令应用于文本选区，从而进一步决定是否需要执行下一个命令。

34. 富文件选择
    - 在内嵌窗格中使用 getSelection()方法，可以获得富文本编辑器的选区。这个方法暴露在document 和 window 对象上，返回表示当前选中文本的 Selection 对象。每个 Selection 对象都拥有以下属性。
        - anchorNode：选区开始的节点。
        - anchorOffset：在 anchorNode 中，从开头到选区开始跳过的字符数。
        - focusNode：选区结束的节点。
        - focusOffset：focusNode 中包含在选区内的字符数。
        - isCollapsed：布尔值，表示选区起点和终点是否在同一个地方。
        - rangeCount：选区中包含的 DOM 范围数量。
    - Selection 的属性并没有包含很多有用的信息。好在它的以下方法提供了更多信息，并允许操作选区。
        - addRange(range)：把给定的 DOM 范围添加到选区。
        - collapse(node, offset)：将选区折叠到给定节点中给定的文本偏移处。
        - collapseToEnd()：将选区折叠到终点。
        - collapseToStart()：将选区折叠到起点。
        - containsNode(node)：确定给定节点是否包含在选区中。
        - deleteFromDocument()：从文档中删除选区文本。与执行 execCommand("delete", false, null)命令结果相同。
        - extend(node, offset)：通过将 focusNode 和 focusOffset 移动到指定值来扩展选区。
        - getRangeAt(index)：返回选区中指定索引处的 DOM 范围。
        - removeAllRanges()：从选区中移除所有 DOM 范围。这实际上会移除选区，因为选区中至少要包含一个范围。
        - removeRange(range)：从选区中移除指定的 DOM 范围。
        - selectAllChildren(node)：清除选区并选择给定节点的所有子节点。
        - toString()：返回选区中的文本内容。
    - Selection 对象的这个方法极其强大，充分利用了 DOM 范围来管理选区。操纵 DOM 范围可以实现比 execCommand()更细粒度的控制，因为可以直接对选中文本的 DOM 内容进行操作。来看下面的例子：
        ```
        let selection = frames["richedit"].getSelection(); 
        // 取得选中的文本
        let selectedText = selection.toString(); 
        // 取得表示选区的范围
        let range = selection.getRangeAt(0); 
        // 高亮选中的文本
        let span = frames["richedit"].document.createElement("span"); 
        span.style.backgroundColor = "yellow"; 
        range.surroundContents(span);
        ```
    - 以上代码会在富文本编辑器中给选中文本添加黄色高亮背景。实现方式是在默认选区使用 DOM 范围，用 surroundContents()方法给选中文本添加背景为黄色的`<span>`标签。
    - getSelection()方法在 HTML5 中进行了标准化，IE9 以及 Firefox、Safari、Chrome 和 Opera 的所有现代版本中都实现了这个方法。
    - IE8 及更早版本不支持 DOM 范围，不过它们允许通过专有的 selection 对象操作选中的文本。如本章前面所讨论的，这个 selection 对象是 document 的属性。要取得富文本编辑器中选中的文本，必须先创建一个文本范围，然后再访问其 text 属性：
        ```
        let range = frames["richedit"].document.selection.createRange(); 
        let selectedText = range.text;
        ```
    - 使用 IE 文本范围执行 HTML 操作不像使用 DOM 范围那么可靠，不过也是可以做到的。要实现与使用 DOM 范围一样的高亮效果，可以组合使用 htmlText 属性和 pasteHTML()方法：
        ```
        let range = frames["richedit"].document.selection.createRange(); 
        range.pasteHTML( 
            '<span style="background-color:yellow">${range.htmlText}</span>');
        ```
    - 以上代码使用 htmlText 取得了当前选区的 HTML，然后用一个`<span>`标签将其包围起来并通过pasteHTML()再把它插入选区中。

35. 通过表单提交富文本
    - 因为富文本编辑是在内嵌窗格中或通过为元素指定 contenteditable 属性实现的，而不是在表单控件中实现，所以富文本编辑器技术上与表单没有关系。这意味着要把富文本编辑的结果提交给服务器，必须手工提取 HTML 并自己提交。通常的解决方案是在表单中添加一个隐藏字段，使用内嵌窗格或contenteditable 元素的 HTML 更新它的值。在表单提交之前，从内嵌窗格或 contenteditable 元素中提取出 HTML 并插入隐藏字段中。例如，以下代码在使用内嵌窗格实现富文本编辑时，可以用在表单的 onsubmit 事件处理程序中：
        ```
        form.addEventListener("submit", (event) => { 
            let target = event.target; 
            target.elements["comments"].value = 
                    frames["richedit"].document.body.innerHTML; 
        });
        ```
    - 这里，代码使用文档主体的 innerHTML 属性取得了内嵌窗格的 HTML，然后将其插入名为"comments"的表单字段中。这样做可以确保在提交表单之前给表单字段赋值。如果使用 submit()方法手工提交表单，那么要注意在提交前先执行上述操作。对于 contenteditable 元素，执行这一操作的代码是类似的：
        ```
        form.addEventListener("submit", (event) => { 
            let target = event.target; 
            target.elements["comments"].value = 
                    document.getElementById("richedit").innerHTML; 
        });
        ```

36. 小结
    - 尽管 HTML 和 Web 应用自诞生以来已经发生了天翻地覆的变化，但 Web 表单几乎从来没有变过。JavaScript 可以增加现有的表单字段以提供新功能或增强易用性。为此，表单字段也暴露了属性、方法和事件供 JavaScript 使用。以下是本章介绍的一些概念。
        - 可以使用标准或非标准的方法全部或部分选择文本框中的文本。
        - 所有浏览器都采用了 Firefox 操作文本选区的方式，使其成为真正的标准。
        - 可以通过监听键盘事件并检测要插入的字符来控制文本框接受或不接受某些字符。
    - 所有浏览器都支持剪贴板相关的事件，包括 copy、cut 和 paste。剪贴板事件在不同浏览器中的实现有很大差异。
    - 在文本框只限某些字符时，可以利用剪贴板事件屏幕粘贴事件。
    - 选择框也是经常使用 JavaScript 来控制的一种表单控件。借助 DOM，操作选择框比以前方便了很多。使用标准的 DOM 技术，可以为选择框添加或移除选项，也可以将选项从一个选择框移动到另一个选择框，或者重排选项。
    - 富文本编辑通常以使用包含空白 HTML 文档的内嵌窗格来处理。通过将文档的 designMode 属性设置为"on"，可以让整个页面变成编辑区，就像文字处理软件一样。另外，给元素添加 contenteditable属性也可以将元素转换为可编辑区。默认情况下，可以切换文本的粗体、斜体样式，也可以使用剪贴板功能。JavaScript 通过 execCommand()方法可以执行一些富文本编辑功能，通过 queryCommandEnabled()、queryCommandState()和 queryCommandValue()方法则可以获取有关文本选区的信息。由于富文本编辑区不涉及表单字段，因此要将富文本内容提交到服务器，必须把 HTML 从 iframe 或 contenteditable元素中复制到一个表单字段。
