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
