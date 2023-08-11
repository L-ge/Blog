# （十三）构造函数&数据常用函数

## 1. 深入对象

### 1.1 创建对象三种方式

1. 利用对象字面量创建对象
    ```js
    const o = {
        name: '佩奇'
    }
    ```
2. 利用 new Object 创建对象
    ```js
    const o = new Object({ name: '佩奇' })
    console.log(o)  // {name: '佩奇'}
    ```
3. 利用构造函数创建对象

### 1.2 构造函数

- 构造函数：是一种特殊的函数，主要用来初始化对象
- 使用场景：常规的 {...} 语法允许创建一个对象。比如我们创建了佩奇的对象，继续创建乔治的对象还需要重新写一遍，此时可以通过构造函数来快速创建多个类似的对象。
- 例子：
    ```js
    function Pig(name, age, gender) {
        this.name = name
        this.age = age
        this.gener = gender 
    }
    // 创建佩奇对象
    const Peppa = new Pig('佩奇', 6, '女')
    // 创建乔治对象
    const George = new Pig('乔治', 3, '男')
    console.log(Peppa)  // {name: '乔治', age: 6, gener: '女'}
    ```
- 构造函数在技术上是常规函数。
- 不过有两个约定：
    1. 它们的命名以大写字母开头。
    2. 它们只能由 "new" 操作符来执行。
- 构造函数语法：大写字母开头的函数
- 说明：
    1. 使用 new 关键字调用函数的行为被称为实例化
    2. 实例化构造函数时没有参数时可以省略 ()
    3. 构造函数内部无需写return，返回值即为新创建的对象
    4. 构造函数内部的 return 返回的值无效，所以不要写return
    5. new Object()、new Date() 也是实例化构造函数
- 实例化执行过程说明：
    1. 创建新对象
    2. 构造函数this指向新对象
    3. 执行构造函数代码，修改this，添加新的属性
    4. 返回新对象

### 1.3 实例成员&静态成员

- 实例成员：通过构造函数创建的对象称为实例对象，实例对象中的属性和方法称为实例成员。
- 实例成员说明：
    1. 实例对象的属性和方法即为实例成员
    2. 为构造函数传入参数，动态创建结构相同但值不同的对象
    3. 构造函数创建的实例对象彼此独立互不影响。
- 静态成员：构造函数的属性和方法被称为静态成员
- 静态成员说明：
    1. 构造函数的属性和方法被称为静态成员
    2. 一般公共特征的属性或方法设置为静态成员
    3. 静态成员方法中的 this 指向构造函数本身

## 2. 内置构造函数

- 在 JavaScript 中最主要的数据类型有 6 种：
    - 基本数据类型：字符串、数值、布尔、undefined、null
    - 引用类型: 对象
- 但是，我们会发现有些特殊情况：
    ```js
    // 普通字符串
    const str = 'andy'
    console.log(str.length) // 4
    ```
    - 其实字符串、数值、布尔、等基本类型也都有专门的构造函数，这些我们称为包装类型。
    - JS中几乎所有的数据都可以基于构成函数创建。
- 引用类型
    - Object，Array，RegExp，Date 等
- 包装类型
    - String，Number，Boolean 等

### 2.1 Object

- Object 是内置的构造函数，用于创建普通对象。
- 例子：
    ```js
    // 通过构造函数创建普通对象
    const user = new Object({name: '小明', age: 15})
    ```
- 推荐使用字面量方式声明对象，而不是 Object 构造函数

- 有三个常用静态方法（静态方法就是只有构造函数Object可以调用的）:
    ```js
    // 想要获取对象里面的属性和值的做法：
    const o = { name: '佩奇', age: 6 }
    
    // 传统做法：
    for(let k in o) {
        console.log(k)      // 属性 name age
        console.log(o[k])   // 值 佩奇 6
    }
    ```

1. Object.keys 静态方法获取对象中所有属性（键）
    ```js
    const o = { name: '佩奇', age: 6 }
    
    // 获取对象的所有键，并且返回是一个数组
    const arr = Object.keys(0)
    console.log(arr)    // ['name', 'age']
    ```
    - 注意：返回的是一个数组

2. Object.values 静态方法获取对象中所有属性值
    ```js
    const o = { name: '佩奇', age: 6 }
    
    // 获取对象的所有值，并且返回是一个数组
    const arr = Object.values(0)
    console.log(arr)    // ['佩奇', 6]
    ```
    - 注意：返回的是一个数组

3. Object. assign 静态方法常用于对象拷贝
    ```js
    // 拷贝对象，把 o 拷贝给 obj
    const o = { name: '佩奇', age: 6 }
    const obj = {}
    Object.assign(obj, o)
    console.log(obj)    // { name: '佩奇', age: 6 }
    
    const o = { name: '佩奇', age: 6 }
    Object.assign(o, { gender: '女' })
    console.log(o)    // { name: '佩奇', age: 6, gender: '女' }
    ```
    - 经常使用的场景给对象添加属性。

### 2.2 Array

- Array 是内置的构造函数，用于创建数组
    ```js
    const arr = new Array(3. 5)
    console.log(arr)    // [3, 5]
    ```
- 创建数组建议使用字面量创建，不用 Array 构造函数创建

#### 2.2.1 数组常见实例方法-核心方法

| 方法 | 作用 | 说明 |
| ---  | ---  | ---  |
| forEach | 遍历数组 | 不返回，用于不改变值，经常用于查找打印输出值 |
| filter | 过滤数组 | 筛选数组元素，并生成新数组 |
| map | 迭代数组 | 返回新数组，新数组里面的元素是处理之后的值，经常用于处理数据 |
| reduce | 累计器 | 返回函数累计处理的结果，经常用于求和等 |

1. reduce 返回函数累计处理的结果，经常用于求和等
- 基本语法：
    ```
    arr.reduce(function(){}, 起始值)
    ```
- 参数：起始值可以省略，如果写就作为第一次累计的起始值
- 语法：
    ```
    arr.reduce(function(累计值, 当前元素 [,索引号][,源数组]){}, 起始值)
    ```
- 累计值参数：
    1. 如果有起始值，则以起始值为准开始累计， 累计值 = 起始值
    2. 如果没有起始值， 则累计值以数组的第一个数组元素作为起始值开始累计
    3. 后面每次遍历就会用后面的数组元素 累计到 累计值 里面 （类似求和里面的 sum ）
- 使用场景：求和运算：
    ```js
    const arr = [1, 5, 9]
    const count = arr.reduce((prev, item) => prev + item)
    console.log(count)
    ```

#### 2.2.2 数组常见方法-其他方法

- 实例方法 `join` 数组元素拼接为字符串，返回字符串(重点)
- 实例方法  `find`  查找元素，返回符合测试条件的第一个数组元素值，如果没有符合条件的则返回 undefined(重点)
- 实例方法 `every` 检测数组所有元素是否都符合指定条件，如果**所有元素**都通过检测返回 true，否则返回 false(重点)
- 实例方法 `some` 检测数组中的元素是否满足指定条件，**如果数组中有**元素满足条件返回 true，否则返回 false
- 实例方法 `concat` 合并两个数组，返回生成新数组
- 实例方法 `sort` 对原数组单元值排序
- 实例方法 `splice` 删除或替换原数组单元
- 实例方法 `reverse` 反转数组
- 实例方法 `findIndex` 查找元素的索引值

#### 2.2.3 数组常见方法-伪数组转换为真数组

- 静态方法 Array.from()

### 2.3 String

- 在 JavaScript 中的字符串、数值、布尔具有对象的使用特征，如具有属性和方法
- 例子：
    ```js
    // 字符串类型
    const str = 'hello world!'
    // 统计字符的长度（字符数量）
    console.log(str.length)
      
    // 数值类型
    const price = 12.345
    // 保留两位小数
    price.toFixed(2) // 12.34
    ```
- 之所以具有对象特征的原因是字符串、数值、布尔类型数据是 JavaScript 底层使用 Object 构造函数“包装”来的，被称为包装类型。

#### 2.3.1 常见实例方法

1. 实例属性 `length` 用来获取字符串的长度(重点)
2. 实例方法 `split('分隔符')` 用来将字符串拆分成数组(重点)
3. 实例方法 `substring（需要截取的第一个字符的索引[,结束的索引号]）` 用于字符串截取(重点)
4. 实例方法 `startsWith(检测字符串[, 检测位置索引号])` 检测是否以某字符开头(重点)
5. 实例方法 `includes(搜索的字符串[, 检测位置索引号])` 判断一个字符串是否包含在另一个字符串中，根据情况返回 true 或 false(重点)
6. 实例方法 `toUpperCase` 用于将字母转换成大写
7. 实例方法 `toLowerCase` 用于将字母转换成小写
8. 实例方法 `indexOf`  检测是否包含某字符
9. 实例方法 `endsWith` 检测是否以某字符结尾
10. 实例方法 `replace` 用于替换字符串，支持正则匹配
11. 实例方法 `match` 用于查找字符串，支持正则匹配

### 2.4 Number

- Number 是内置的构造函数，用于创建数值
- 常用方法：
    - toFixed() 设置保留小数位的长度
- 例子：
    ```js
    // 数值类型
    const price = 12.345
    // 保留两位小数，四舍五入
    console.log(price.toFixed(2)) // 12.34
    ```
