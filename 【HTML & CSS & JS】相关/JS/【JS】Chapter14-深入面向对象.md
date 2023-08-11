# （十四）深入面向对象

## 1. 编程思想

### 1.1 面向过程介绍

- 面向过程就是分析出解决问题所需要的步骤，然后用函数把这些步骤一步一步实现，使用的时候再一个一个的依次调用就可以了。
- 面向过程，就是按照我们分析好了的步骤，按照步骤解决问题。

### 1.2 面向对象介绍

- 面向对象是把事务分解成为一个个对象，然后由对象之间分工与合作。
- 面向对象是以对象功能来划分问题，而不是步骤。
- 在面向对象程序开发思想中，每一个对象都是功能中心，具有明确分工。
- 面向对象编程具有灵活、代码可复用、容易维护和开发的优点，更适合多人合作的大型软件项目。
- 面向对象的特性：
    - 封装性
    - 继承性
    - 多态性

### 1.3 面向过程和面向对象的对比

- 面向过程编程：
    - 优点：性能比面向对象高，适合跟硬件联系很紧密的东西，例如单片机就采用的面向过程编程。
    - 缺点：没有面向对象易维护、易复用、易扩展
- 面向对象编程：
    - 优点：易维护、易复用、易扩展，由于面向对象有封装、继承、多态性的特性，可以设计出低耦合的系统，使系统 更加灵活、更加易于维护
    - 缺点：性能比面向过程低

## 2. 构造函数

- 封装是面向对象思想中比较重要的一部分，js面向对象可以通过构造函数实现的封装。
- 同样的将变量和函数组合到了一起并能通过 this 实现数据的共享，所不同的是借助构造函数创建出来的实例对象之间是彼此不影响的
- 总结：
    1. 构造函数体现了面向对象的封装特性
    2. 构造函数实例创建的对象彼此独立、互不影响
- 封装是面向对象思想中比较重要的一部分，js面向对象可以通过构造函数实现的封装。
- 面向对象编程的特性：比如封装性、继承性等，可以借助于构造函数来实现
- 前面我们学过的构造函数方法很好用，但是存在浪费内存的问题：
    ```js
    function Star(uname, age) {
        this.uname = uname;
        this.age = age;
        this.sing = function() {
            console.log('我会唱歌');
        }
    }
    const ldh = new Star('刘德华', 18)
    const zxy = new Star('张学友', 19)
    console.log(ldh.sing === zxy.sing)  // 结果为 false，说明俩函数不一样
    ```
    - 我们希望所有的对象使用同一个函数，这样就比较节省内存，那么我们要怎样做呢？

## 3. 原型

### 3.1 原型

- 构造函数通过原型分配的函数是所有对象所共享的。
- JavaScript 规定，每一个构造函数都有一个 prototype 属性，指向另一个对象，所以我们也称为原型对象
- 这个对象可以挂载函数，对象实例化不会多次创建原型上函数，节约内存
- 我们可以把那些不变的方法，直接定义在 prototype 对象上，这样所有对象的实例就可以共享这些方法。
- **构造函数和原型对象中的 this 都指向实例化的对象。**
- 例子：
    ```js
    function Star(uname, age) {
        this.uname = uname;
        this.age = age;
    }
    console.log(Star.prototype)     // 返回一个对象称为原型对象
    Star.prototype.sing = function() {
        console.log('我会唱歌');
    }
    const ldh = new Star('刘德华', 18)
    const zxy = new Star('张学友', 19)
    console.log(ldh.sing === zxy.sing)  // 结果为 true，说明俩函数一样，共享
    ```

### 3.2 constructor 属性

- 每个原型对象里面都有个 constructor 属性（constructor 构造函数）
- 作用：该属性指向该原型对象的构造函数。
- 使用场景：
    - 如果有多个对象的方法，我们可以给原型对象采取对象形式赋值.
    - 但是这样就会覆盖构造函数原型对象原来的内容，这样修改后的原型对象 constructor 就不再指向当前构造函数了
    - 此时，我们可以在修改后的原型对象中，添加一个 constructor 指向原来的构造函数。
- 例子：
    ```js
    function Star(name) {
        this.name = name
    }
    Star.prototype = {
        sing: function() { console.log('唱歌') },
        dance: function() { console.log('跳舞') }
    }
    console.log(Star.prototype.constructor)     // 指向 Object
    
    function Star(name) {
        this.name = name
    }
    Star.prototype = {
        // 手动利用 constructor 指回 Star 构造函数
        constructor: Star,
        sing: function() { console.log('唱歌') },
        dance: function() { console.log('跳舞') }
    }
    console.log(Star.prototype.constructor)     // 指向 Star
    ```

### 3.3 对象原型

- 对象都会有一个属性 __proto__指向构造函数的 prototype 原型对象，之所以我们对象可以使用构造函数 prototype 原型对象的属性和方法，就是因为对象有 __proto__原型的存在。
- 注意：
    - __proto__是JS非标准属性
    - [[prototype]]和__proto__意义相同
    - 用来表明当前实例对象指向哪个原型对象prototype
    - __proto__对象原型里面也有一个 constructor属性，指向创建该实例对象的构造函数

### 3.4 原型继承

- 继承是面向对象编程的另一个特征，通过继承进一步提升代码封装的程度，JavaScript 中大多是借助原型对象实现继承的特性。
- 例子：
    ```js
    function Man() {
        this.head = 1;
        this.eyes = 2;
        this.legs = 2;
        this.say = function() {},
        this.eat = function() {}
    }
    const pink = new Man()
    
    function Woman() {
        this.head = 1;
        this.eyes = 2;
        this.legs = 2;
        this.say = function() {},
        this.eat = function() {},
        this.baby = function() {}
    }
    const red = new Woman()
    ```
- 对上面例子进行改造：
    1. 封装-抽取公共部分
        - 把男人和女人公共的部分抽取出来放到人类里面。
    2. 继承-让男人和女人都能继承人类的一些属性和方法
        - 把男人女人公共的属性和方法抽取出来 People
        - 然后赋值给Man的原型对象，可以共享这些属性和方法
        - 注意让constructor指回Man这个构造函数
    3. 问题：
        - 如果我们给男人添加了一个吸烟的方法，发现女人自动也添加这个方法。
        - 原因：男人和女人都同时使用了同一个对象，根据引用类型的特点，他们指向同一个对象，修改一个就会都影响
        - 解决：男人和女人不要使用同一个对象，但是不同对象里面包含相同的属性和方法。
    4. 继承写法完善
- 最后代码：
    ```js
    const People = {
        head: 1;
        eyes: 2;
        legs: 2;
        say: function() {},
        eat: function() {}
    }
    
    // 男人
    function Man() {
    }
    // 用 new Person() 替换刚才的固定对象
    Man.prototype = new Person()
    // 注意让原型里面的constructor重新指回Man
    Man.prototype.constructor = Man
    const pink = new Man()
    Man.prototype.smoking = function() {}
    console.log(pink)
    
    // 女人
    function Woman() {
        this.bady = function() {}
    }
    // 用 new Person() 替换刚才的固定对象
    Woman.prototype = new Person()
    // 注意让原型里面的constructor重新指回Woman
    Woman.prototype.constructor = Woman
    const red = new Woman()
    console.log(red)
    ```
- 思路：真正做这个案例，我们的思路应该是先考虑大的，后考虑小的
    1. 人类共有的属性和方法有那些，然后做个构造函数，进行封装，一般公共属性写到构造函数内部，公共方法，挂载到构造函数原型身上。
    2. 男人继承人类的属性和方法，之后创建自己独有的属性和方法
    3. 女人同理

### 3.5 原型链

- 基于原型对象的继承使得不同构造函数的原型对象关联在一起，并且这种关联的关系是一种链状的结构，我们将原型对象的链状结构关系称为原型链
- 原型链-查找规则：
    1. 当访问一个对象的属性（包括方法）时，首先查找这个对象自身有没有该属性。
    2. 如果没有就查找它的原型（也就是 __proto__指向的 prototype 原型对象）
    3. 如果还没有就查找原型对象的原型（Object的原型对象）
    4. 依此类推一直找到 Object 为止（null）
    5. __proto__对象原型的意义就在于为对象成员查找机制提供一个方向，或者说一条路线
    6. 可以使用 instanceof 运算符用于检测构造函数的 prototype 属性是否出现在某个实例对象的原型链上
