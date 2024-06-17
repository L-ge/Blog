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
