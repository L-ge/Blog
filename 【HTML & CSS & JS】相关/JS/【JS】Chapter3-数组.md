# 站在巨人的肩膀上

> [黑马程序员前端JavaScript入门到精通全套视频教程，javascript核心进阶ES6语法、API、js高级等基础知识和实战教程](https://www.bilibili.com/video/BV1Y84y1L7Nn/)


# （三）数组

## 1. 循环-for

### 1.1 for循环基本使用

1. for循环语法
- 作用：重复执行代码
- 好处：把声明起始值、循环条件、变化值写到一起，让人一目了然，它是最常使用的循环形式
- 语法：
    ```js
    for(变量起始值; 终止条件; 变量变化量) {
        // 循环体
    }
    ```

2. 退出循环
-  continue 退出本次循环，一般用于排除或者跳过某一个选项的时候，可以使用continue
- break 退出整个for循环，一般用于结果已经得到，后续的循环不需要的时候可以使用

了解：
1. while(true) 来构造“无限”循环，需要使用break退出循环。
2. for(;;) 也可以来构造“无限”循环，同样需要使用break退出循环。

for循环和while循环有什么区别呢：
- 当如果明确了循环的次数的时候推荐使用for循环
- 当不明确循环的次数的时候推荐使用while循环

### 1.2 for 循环嵌套

```js
for(外部声明记录循环次数的变量; 循环条件; 变化值) {
    for(内部声明记录循环次数的变量; 循环条件; 变化值) {
        循环体
    }
}
```
- 一个循环里再套一个循环，一般用在for循环里

## 2. 数组

### 2.1 数组是什么

- 数组：(Array)是一种可以按顺序保存数据的数据类型
- 为什么要数组？
    - 思考：如果我想保存一个班里所有同学的姓名怎么办？
    - 场景：如果有多个数据可以用数组保存起来，然后放到一个变量中，管理非常方便

### 2.2 数组的基本使用

1. 声明语法
    ```js
    let 数组名 = [数据1, 数据2, ..., 数据n]
    let names = ['小明', '小刚', '小红', '小丽', '小米']
    ```
- 数组是按顺序保存，所以每个数据都有自己的编号
- 计算机中的编号从0开始，所以小明的编号为0，小刚编号为1，以此类推
- 在数组中，数据的编号也叫索引或下标
- 数组可以存储任意类型的数据

2. 取值语法
    ```js
    数组名[下标]
    
    let names = ['小明', '小刚', '小红', '小丽', '小米']
    names[0]    // 小明
    names[1]    // 小刚
    ```
- 通过下标取数据
- 取出来是什么类型的，就根据这种类型特点来访问

3. 一些术语：
    ```js
    let names = ['小明', '小刚', '小红', '小丽', '小米']
    console.log(names.length)   // 5
    ```
- 元素：数组中保存的每个数据都叫数组元素
- 下标：数组中数据的编号
- 长度：数组中数据的个数，通过数组的length属性获得

4. 遍历数组(重点)
- 用循环把数组中每个元素都访问到,一般会用for循环遍历
- 语法：
    ```js
    for(let i=0; i<数组名.length; i++) {
        数组名[i]
    }
    ```

### 2.3 操作数组

数组本质是数据集合, 操作数据无非就是 增 删 改 查。

语法：
- 查询数组数据
    - 数组[下标]
    - 或者我们称为访问数组数据
- 重新赋值
    - 数组[下标] = 新值
- 数组添加新的数据
    - arr.push(新增的内容)
    - arr.unshift(新增的内容)
- 删除数组中数据
    - arr.pop()
    - arr.shift()
    - arr.splice(操作的下标,删除的个数)

#### 2.3.1 操作数组-新增

- 数组.push() 方法将一个或多个元素添加到数组的末尾，并返回该数组的新长度 (重点)
    - 语法：
        ```js
        arr.push(元素1, ..., 元素n)
        ```
    - 例子：
        ```js
        let arr = ['red', 'green']
        arr.push('pink', 'hotpink')
        console.log(arr)   // ['red', 'green', 'pink', 'hotpink']
        ```
- arr.unshift(新增的内容) 方法将一个或多个元素添加到数组的开头，并返回该数组的新长度
    - 语法：
        ```js
        arr.unshift(元素1, ..., 元素n)
        ```
    - 例子：
        ```js
        let arr = ['red', 'green']
        arr.unshift('pink', 'hotpink')
        console.log(arr)   // ['pink', 'hotpink', 'red', 'green']
        ```

#### 2.3.2 操作数组-删除

- 数组.pop() 方法从数组中删除最后一个元素，并返回该元素的值
    - 语法：
        ```js
        arr.pop()
        ```
- 数组.shift() 方法从数组中删除第一个元素，并返回该元素的值
    - 语法：
        ```js
        arr.shift()
        ```
- 数组.splice() 方法删除指定元素
    - 语法：
        ```js
        arr.splice(start, deleteCount)
        arr.splice(起始位置, 删除几个元素)
        ```
    - start 起始位置:
        - 指定修改的开始位置（从0计数）
    - deleteCount:
        - 表示要移除的数组元素的个数
        - 可选的。如果省略则默认从指定的起始位置删除到最后

### 2.4 数组案例

冒泡排序：
- 冒泡排序是一种简单的排序算法。
- 它重复地走访过要排序的数列，一次比较两个元素，如果他们的顺序错误就把他们交换过来。走访数列的工作是重复地进行直到没有再需要交换，也就是说该数列已经排序完成。
- 这个算法的名字由来是因为越小的元素会经由交换慢慢“浮”到数列的顶端。
- 比如数组 [2,3,1,4,5] 经过排序成为了 [1,2,3,4,5] 或者 [5,4,3,2,1]

数组排序：
- 数组.sort() 方法可以排序
    - 语法：
        ```js
        let arr = [4, 2, 5, 1, 3]
        // 1. 升序排列写法
        arr.sort(function (a, b) {
            return a - b
        })
        console.log(arr) // [1, 2, 3, 4, 5]
        // 2. 降序排列写法
        arr.sort(function (a, b) {
            return b - a
        })
        console.log(arr) // [5, 4, 3, 2, 1]
        ```

## 3. 综合案例

无。
