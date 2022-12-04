###### 一、起步

1. 本书编写期间的最新版本为Python 3.7，但只要你安装了Python 3.6或更高的版本，就能运行本书中的所有代码。

2. 运行Python代码片段
- Python自带一个在终端窗口中运行的解释器，让你无须保存并运行整个程序就能尝试运行Python代码片段。
    ```
    C:\> python --version
    Python 3.7.0
    C:\> python
    Python 3.7.0 (v3.7.0:1bf9cc5093, Jun 27 2018, 04:59:51) [MSC v.1914 64 bit (AMD64)] on win32
    Type "help", "copyright", "credits" or "license" for more information.  
    >>> 
    >>> print("Hello Python interpreter!") 
    Hello Python interpreter!
    >>> 
    ```
    - 提示符 >>> 表明正在使用终端窗口。

3. 在Windows系统中搭建Python编程环境
- 请务必选中复选框Add Python（版本号）to PATH，这让你能够更轻松地配置系统。
- 每当要运行Python代码片段时，都请打开一个命令窗口并启动Python终端会话。要关闭该终端会话，可按Ctrl + Z、再按回车键，也可执行命令exit() 。

4. 文件名和文件夹名称最好使用小写字母，并使用下划线代替空格，因为Python采用了这些命名约定。

5. Windows 系统使用命令dir（表示directory，即目录）可以显示当前目录中的所有文件。

6. 从终端运行Python程序
```
C:\> cd Desktop\python_work
C:\Desktop\python_work> dir 
hello_world.py
C:\Desktop\python_work> python hello_world.py 
Hello Python world!
```
- 要运行Python程序，只需使用命令python（或python3）即可。

*** 

###### 二、变量和简单数据类型

1. 在程序中可随时修改变量的值，而Python将始终记录变量的最新值。
```
message = "Hello Python world!" 
print(message)

message = "Hello Python Crash Course world!" 
print(message)
```

2. 变量的命名规则
- 变量名只能包含字母、数字和下划线。变量名能以字母或下划线打头，但不能以数 字打头。例如，可将变量命名为message_1 ，但不能将其命名为1_message 。
- 变量名不能包含空格，但能使用下划线来分隔其中的单词。例如，变量名 greeting_message 可行，但变量名 greeting message 会引发错误。
- 不要将Python关键字和函数名用作变量名，即不要使用Python保留用于特殊用途的 单词，如 print。
- 变量名应既简短又具有描述性。例如，name 比 n 好，student_name 比 s_n 好，name_length 比 length_of_persons_name 好。
- 慎用小写字母 l 和大写字母 O，因为它们可能被人错看成数字 1 和 0。
- 注意：就目前而言，应使用小写的Python变量名。虽然在变量名中使用大写字母不会导致错误，但是大写字母在变量名中有特殊含义。

3. 变量名称错误通常意味着两种情况：要么是使用变量前忘记给它赋值，要么是输  入变量名时拼写不正确。

4. 变量是可以赋给值的标签，也可以说变量指向特定的值。

5. 在Python中，用引号括起的都是字符串，其中的引号可以是单引号，也可以是双引号。这种灵活性让你能够在字符串中包含引号和撇号：
```
'I told my friend, "Python is my favorite language!"'
"The language 'Python' is named after Monty Python, not the snake." "One of Python's strengths is its diverse and supportive community."
```

6. 方法title() 以首字母大写的方式显示每个单词，即将每个单词的首字母都改为大写。
```
name = "ada lovelace" 
print(name.title())         // Ada Lovelace
print(name.upper())         // ADA LOVELACE
print(name.lower())         // ada lovelace
```
- 存储数据时，方法lower() 很有用。很多时候，你无法依靠用户来提供正确的大小写，因此需要将字符串先转换为小写，再存储它们。以后需要显示这些信息时，再将其转换为最合适的大小写方式。

7. 要在字符串中插入变量的值，可在前引号前加上字母f，再将要插入的变量放在花括号内。这样，当Python显示字符串时，将把每个变量都替换为其值。这种字符串名为f字符串。f是format（设置格式）的简写，因为Python通过把花括号内的变量替换为其值来设置字符串的格式。
```
first_name = "ada" 
last_name = "lovelace"
full_name = f"{first_name} {last_name}" print(full_name)

first_name = "ada" last_name = "lovelace"
full_name = f"{first_name} {last_name}"
print(f"Hello, {full_name.title()}!")
```

8. 注意 f 字符串是Python 3.6引入的。如果你使用的是Python 3.5或更早的版本， 需要使用format() 方法，而非这种f语法。要使用方法format()，可在圆括号内列出要在字符串中使用的变量。对于每个变量，都通过一对花括号来引用。这样将按顺序将这些花括号替换为圆括号内列出的变量的值，
```
full_name = "{} {}".format(first_name, last_name)
```

9. 在编程中，空白泛指任何非打印字符，如空格、制表符和换行符。要在字符串中添加制表符，可使用字符组合\t。要在字符串中添加换行符，可使用字符组合\n。
```
>>> print("Python") 
Python
>>> print("\tPython") 
    Python
>>> print("Languages:\nPython\nC\nJavaScript") 
Languages:
Python 
C
JavaScript
```

10. Python能够找出字符串开头和末尾多余的空白。要确保字符串末尾没有空白，可使用方法rstrip() 。还可以剔除字符串开头的空白，或者同时剔除字符串两边的空白。为此，可分别使用方法lstrip() 和strip()。
```
>>> favorite_language = 'python '
>>> favorite_language
'python '
>>> favorite_language.rstrip() 
'python'
>>> favorite_language 
'python '

>>> favorite_language = 'python '
>>> favorite_language = favorite_language.rstrip()
>>> favorite_language 
'python'

>>> favorite_language = ' python '
>>> favorite_language.rstrip() 
' python'
>>> favorite_language.lstrip() 
'python '
>>> favorite_language.strip() 
'python'
```
- 调用方法rstrip() 后，这个多余的空格被删除了。然而，这种删除只是暂时的，要永久删除这个字符串中的空白，必须将删除操作的结果关联到变量。

11. 整数
- 在Python中，可对整数执行加（+）减（-）乘（*）除（/）运算。
- Python使用两个乘号表示乘方运算。
- Python还支持运算次序，因此可在同一个表达式中使用多种运算。还可以使用圆括号来修改运算次序，让Python按你指定的次序执行运算。
- 空格不影响Python计算表达式的方式。

12. 从很大程度上说，使用浮点数时无须考虑其行为。你只需输入要使用的数，Python通常会按你期望的方式处理它们。但需要注意的是，结果包含的小数位数可能是不确定的。
```
>>> 0.2 + 0.1
0.30000000000000004
>>> 3 * 0.1
0.30000000000000004
```

13. 将任意两个数相除时，结果总是浮点数，即便这两个数都是整数且能整除。在其他任何运算中，如果一个操作数是整数，另一个操作数是浮点数，结果也总是浮点数。
```
>>> 4/2
2.0
>>> 1 + 2.0
3.0
>>> 2 * 3.0
6.0
>>> 3.0 ** 2
9.0
```
- 无论是哪种运算，只要有操作数是浮点数，Python默认得到的总是浮点数，即便结果原本为整数也是如此。

14. 书写很大的数时，可使用下划线将其中的数字分组，使其更清晰易读。当你打印这种使用下划线定义的数时，Python不会打印其中的下划线。
```
>>> universe_age = 14_000_000_000
>>> print(universe_age) 
14000000000
```
- 因为存储这种数时，Python会忽略其中的下划线。
- 将数字分组时，即便不是将每三位分成一组，也不会影响最终的值。在Python看来，1000 与1_000 没什么不同，1_000与10_00也没什么不同。
- **这种表示法适用于整数和浮点数，但只有Python 3.6和更高的版本支持。**

15. 可在一行代码中给多个变量赋值，这有助于缩短程序并提高其可读性。这种做法最常用于将一系列数赋给一组变量。
```
>>> x, y, z = 0, 0, 0
```
- 这样做时，需要用逗号将变量名分开；对于要赋给变量的值，也需同样处理。Python将按顺序将每个值赋给对应的变量。只要变量和值的个数相同，Python就能正确地将它们关联起来。

16. Python没有内置的常量类型，但Python程序员会使用全大写来指出应将某个变量视为常量，其值应始终不变。
```
MAX_CONNECTIONS = 5000
```

16. 在Python中，注释用井号（#）标识。井号后面的内容都会被Python解释器忽略。

17. 要获悉有关编写优秀Python代码的指导原则，只需在解释器中执行命令import this。

***

###### 三、列表介绍

1. 列表由一系列按特定顺序排列的元素组成。在Python中，用方括号（[]）表示列表，并用逗号分隔其中的元素。如果让Python将列表打印出来，Python将打印列表的内部表示，包括方括号。
```
bicycles = ['trek', 'cannondale', 'redline', 'specialized'] 
print(bicycles)

['trek', 'cannondale', 'redline', 'specialized']
```

2. 列表是有序集合，因此要访问列表的任意元素，只需将该元素的位置（索引）告诉Python即可。要访问列表元素，可指出列表的名称，再指出元素的索引，并将后者放在方括号内。
```
bicycles = ['trek', 'cannondale', 'redline', 'specialized']
print(bicycles[0])
print(bicycles[0].title())

trek
Trek
```

3. 在Python中，第一个列表元素的索引为0，而不是1。

4. Python为访问最后一个列表元素提供了一种特殊语法。通过将索引指定为-1，可让Python返回最后一个列表元素。这种约定也适用于其他负数索引。例如，索引-2返回倒数第二个列表元素，索引-3返回倒数第三个列表元素，依此类推。

5. 要修改列表元素，可指定列表名和要修改的元素的索引，再指定该元素的新值。
```
motorcycles[0] = 'ducati' 
```

6. Python提供了多种在既有列表中添加新数据的方式。
```
1. 在列表末尾添加元素
motorcycles = []                // 创建一个空列表
motorcycles.append('ducati')

2. 在列表中插入元素
motorcycles.insert(0, 'ducati')
```
- 在列表中添加新元素时，最简单的方式是将元素附加（append）到列表。给列表附加元素时，它将添加到列表末尾。
- 使用方法insert()可在列表的任何位置添加新元素。为此，你需要指定新元素的索引和值。

7. 从列表中删除元素
```
1. 使用del语句删除元素
del motorcycles[0]

2. 使用方法pop()删除元素
popped_motorcycle = motorcycles.pop()

3. 弹出列表中任何位置处的元素
first_owned = motorcycles.pop(0)

4. 根据值删除元素
motorcycles.remove('ducati') 
```
- 可以根据位置或值来删除列表中的元素。
- 如果知道要删除的元素在列表中的位置，可使用del 语句。
- 方法pop()删除列表末尾的元素，并让你能够接着使用它。
- 实际上，可以使用pop()来删除列表中任意位置的元素，只需在圆括号中指定要删除元素的索引即可。
- 每当你使用pop()时，被弹出的元素就不再在列表中了。
- 如果你要从列表中删除一个元素，且不再以任何方式使用它，就使用del语句；如果你要在删除元素后还能继续使用它，就使用方法pop() 。
- 如果只知道要删除的元素的值，可使用方法remove()。
- 注意，方法remove()只删除第一个指定的值。如果要删除的值可能在列表中出现多次，就需要使用循环来确保将每个值都删除。

8. 使用方法sort()对列表永久排序
```
cars = ['bmw', 'audi', 'toyota', 'subaru']
cars.sort() 
print(cars)

cars.sort(reverse=True)
```
- 方法sort()永久性地修改列表元素的排列顺序。
- 还可以按与字母顺序相反的顺序排列列表元素，只需向sort()方法传递参数reverse=True即可。

9. 使用函数sorted()对列表临时排序
```
print(sorted(cars))
```
- 函数sorted()让你能够按特定顺序显示列表元素，同时不影响它们在列表中的原始排列顺序。
- 如果要按与字母顺序相反的顺序显示列表，也可向函数sorted()传递参数reverse=True。
- 注意，在并非所有的值都是小写时，按字母顺序排列列表要复杂些。决定排列顺序时，有多种解读大写字母的方式，要指定准确的排列顺序，可能比我们这里所做的要复杂。

10. 要反转列表元素的排列顺序，可使用方法reverse()。
```
cars.reverse() 
```
- 注意，reverse()不是按与字母顺序相反的顺序排列列表元素，而只是反转列表元素的排列顺序。
- 方法reverse()永久性地修改列表元素的排列顺序，但可随时恢复到原来的排列顺序，只需对列表再次调用reverse()即可。

11. 使用函数len()可快速获悉列表的长度。
```
>>> cars = ['bmw', 'audi', 'toyota', 'subaru']
>>> len(cars) 
4
```
- 注意，Python计算列表元素数时从1开始。

12. 每当需要访问最后一个列表元素时，都可使用索引-1。这在任何情况下都行之有效，即便你最后一次访问列表后，其长度发生了变化。索引-1总是返回最后一个列表元素。**仅当列表为空时，这种访问最后一个元素的方式才会导致错误。**

***

###### 四、操作列表

1. 使用for循环遍历整个列表
```
magicians = ['alice', 'david', 'carolina']
for magician in magicians:
    print(magician)
    print(f"I can't wait to see your next trick, {magician.title()}.\n")
```
- 在for 循环中，想包含多少行代码都可以。在代码行for magician in magicians后面，每个缩进的代码行都是循环的一部分，将针对列表中的每个值都执行一次。

2. Python根据缩进来判断代码行与前一个代码行的关系。如果你不小心缩进了无须缩进的代码行，Python将指出这一点。

3. for语句末尾的冒号告诉Python，下一行是循环的第一行。如果不小心遗漏了冒号，将导致语法错误。

4. 列表非常适合用于存储数字集合，而Python提供了很多工具，可帮助你高效地处理数字列表。

5. Python函数range()让你能够轻松地生成一系列数。
```
for value in range(1, 5): 
    print(value)

1
2
3
4
```
- 函数range()让Python从指定的第一个值开始数，并在到达你指定的第二个值时停止。因为它在第二个值处停止，所以输出不包含该值（这里为5）。
- 调用函数range()时，也可只指定一个参数，这样它将从0开始。例如，range(6)返回数0～5。

6. 要创建数字列表，可使用函数list()将range()的结果直接转换为列表。如果将range()作为list()的参数，输出将是一个数字列表。
```
numbers = list(range(1, 6)) 
print(numbers)                  // [1, 2, 3, 4, 5]

// 在这个示例中，函数range()从2开始数，然后不断加2，直到达到或超过终值
even_numbers = list(range(2, 11, 2))
print(even_numbers)             // [2, 4, 6, 8, 10]

// 创建一个列表，其中包含前10个整数（1～10）的平方
squares = []
for value in range(1, 11):
	square = value ** 2
	squares.append(square)
print(squares)      // [1, 4, 9, 16, 25, 36, 49, 64, 81, 100]

```
- 使用函数range()时，还可指定步长。为此，可给这个函数指定第三个参数，Python将根据这个步长来生成数。

7. 有几个专门用于处理数字列表的Python函数。
```
>>> digits = [1, 2, 3, 4, 5, 6, 7, 8, 9, 0]
>>> min(digits)
0
>>> max(digits)
9
>>> sum(digits)
45
```

8. 列表解析将for循环和创建新元素的代码合并成一行，并自动附加新元素。
```
squares = [value**2 for value in range(1, 11)] 
print(squares)      // [1, 4, 9, 16, 25, 36, 49, 64, 81, 100]
```
- 要使用这种语法，首先指定一个描述性的列表名，如squares。然后，指定一个左方括号，并定义一个表达式，用于生成要存储到列表中的值。在这个示例中，表达式为value\*\*2，它计算平方值。接下来，编写一个for循环，用于给表达式提供值，再加上右方括号。在这个示例中，for循环为for value in range(1,11)，它将值1～10提供给表达式value\*\*2。**请注意，这里的for 语句末尾没有冒号**。

9. 还可以处理列表的部分元素，Python称之为切片。要创建切片，可指定要使用的第一个元素和最后一个元素的索引。与函数range()一样，Python在到达第二个索引之前的元素后停止。要输出列表中的前三个元素，需要指定索引0和3，这将返回索引为0、1和2的元素。
```
players = ['charles', 'martina', 'michael', 'florence', 'eli']
print(players[0:3])     // ['charles', 'martina', 'michael']

players = ['charles', 'martina', 'michael', 'florence', 'eli'] 
print(players[:4])      // ['charles', 'martina', 'michael', 'florence']

players = ['charles', 'martina', 'michael', 'florence', 'eli'] 
print(players[2:])      // ['michael', 'florence', 'eli']

players = ['charles', 'martina', 'michael', 'florence', 'eli'] 
print(players[-3:])     // ['michael', 'florence', 'eli']
```
- 如果没有指定第一个索引，Python将自动从列表开头开始。要让切片终止于列表末尾，也可使用类似的语法。
- 负数索引返回离列表末尾相应距离的元素，因此你可以输出列表末尾的任意切片。
- 注意	可在表示切片的方括号内指定第三个值。这个值告诉Python在指定范围内每隔多少元素提取一个。

10. 如果要遍历列表的部分元素，可在for循环中使用切片。
```
for player in players[:3]: 
    print(player.title())
```

11. 要复制列表，可创建一个包含整个列表的切片，方法是同时省略起始索引和终止索引（[:]）。这让Python创建一个始于第一个元素、终止于最后一个元素的切片，即整个列表的副本。
```
my_foods = ['pizza', 'falafel', 'carrot cake']
friend_foods = my_foods[:]

friend_foods.append('ice cream')

# 这行不通:
friend_foods = my_foods
my_foods.append('cannoli')
friend_foods.append('ice cream')
print(my_foods)     // ['pizza', 'falafel', 'carrot cake', 'cannoli', 'ice cream']
```
- 如果只是将my_foods 赋给friend_foods，就不能得到两个列表。这里将my_foods赋给friend_foods，而不是将my_foods的副本赋给friend_foods。这种语法实际上是让Python将新变量friend_foods关联到已与my_foods相关联的列表，因此这两个变量指向同一个列表。

12. 列表非常适合用于存储在程序运行期间可能变化的数据集。

13. Python将不能修改的值称为不可变的，而不可变的列表被称为元组。

14. 元组看起来很像列表，但使用圆括号而非中括号来标识。定义元组后，就可使用索引来访问其元素，就像访问列表元素一样。
```
dimensions = (200, 50)
print(dimensions[0]) 
print(dimensions[1])

my_t = (3,)
```
- 严格地说，元组是由逗号标识的，圆括号只是让元组看起来更整洁、更清晰。**如果你要定义只包含一个元素的元组，必须在这个元素后面加上逗号。**
- 创建只包含一个元素的元组通常没有意义，但自动生成的元组有可能只有一个元素。

15. 像列表一样，也可以使用for循环来遍历元组中的所有值。
```
dimensions = (200, 50)
for dimension in dimensions: 
    print(dimension)
```

16. 虽然不能修改元组的元素，但可以给存储元组的变量赋值。
```
dimensions = (200, 50)
print("Original dimensions:") 
for dimension in dimensions:
    print(dimension)

dimensions = (400, 100)
print("\nModified dimensions:") 
for dimension in dimensions:
    print(dimension)
```
- 给元组变量重新赋值是合法的。


17. 如果需要存储的一组值在程序的整个生命周期内都不变，就可以使用元组。

18. 要提出Python语言修改建议，需要编写Python改进提案（Python Enhancement Proposal，PEP）。PEP 8是最古老的PEP之一，向Python程序员提供了代码格式设置指南。
- PEP 8建议每级缩进都使用四个空格（并非制表符）。
- 很多Python程序员建议每行不超过80字符。
- PEP 8还建议注释的行长不应超过72字符，因为有些工具为大型项目自动生成文档时，会在每行注释开头添加格式化字符。
- 要将程序的不同部分分开，可使用空行。

***

###### 五、if 语句

1. 每条if语句的核心都是一个值为True或False的表达式，这种表达式称为条件测试。Python根据条件测试的值为True还是False来决定是否执行if语句中的代码。如果条件测试的值为True，Python就执行紧跟在if语句后面的代码；如果为False，Python就忽略这些代码。

2. 最简单的条件测试检查变量的值是否与特定值相等。
```
>>> car == 'bmw' 
True
```

3. 在Python中检查是否相等时区分大小写。
```
>>> car = 'Audi'
>>> car == 'audi' 
False

>>> car = 'Audi'
>>> car.lower() == 'audi' 
True
>>> car 
'Audi'
```
- 函数lower()不会修改最初赋给变量car的值。

4. 要判断两个值是否不等，可结合使用惊叹号和等号（!=），其中的惊叹号表示不。
```
requested_topping = 'mushrooms'

if requested_topping != 'anchovies': 
    print("Hold the anchovies!")
```
- 你编写的大多数条件表达式检查两个值是否相等，但有时候检查两个值是否不等的效率更高。

5. 数值比较
```
>>> age = 18
>>> age == 18 
True
```

6. 条件语句中可包含各种数学比较，如小于、小于等于、大于、大于等于。
```
>>> age = 19
>>> age < 21 
True
>>> age <= 21 
True
>>> age > 21 
False
>>> age >= 21
False
```

7. 要检查是否两个条件都为True，可使用关键字and将两个条件测试合而为一。如果每个测试都通过了，整个表达式就为True；如果至少一个测试没有通过，整个表达式就为False。
```
>>> age_0 = 22
>>> age_1 = 18
>>> age_0 >= 21 and age_1 >= 21  // 或 (age_0 >= 21) and (age_1 >= 21)
False
```
- 为改善可读性，可将每个测试分别放在一对圆括号内，但并非必须这样做。

8. 关键字or也能够让你检查多个条件，但只要至少一个条件满足，就能通过整个测试。仅当两个测试都没有通过时，使用or的表达式才为False 。
```
>>> age_0 = 22
>>> age_1 = 18
>>> age_0 >= 21 or age_1 >= 21 
True
```

9. 要判断特定的值是否已包含在列表中，可使用关键字in。
```
>>> requested_toppings = ['mushrooms', 'onions', 'pineapple']
>>> 'mushrooms' in requested_toppings 
True
```

10. 可使用关键字not in确定特定的值未包含在列表中。
```
banned_users = ['andrew', 'carolina', 'david'] 
user = 'marie'

if user not in banned_users:
    print(f"{user.title()}, you can post a response if you wish.")
```

11. 与条件表达式一样，布尔表达式的结果要么为True，要么为False。布尔值通常用于记录条件。
```
game_active = True 
can_edit = False
```

12. 在if语句中，缩进的作用与在for循环中相同。如果测试通过了，将执行if语句后面所有缩进的代码行，否则将忽略它们。
```
age = 19
if age >= 18:
    print("You are old enough to vote!") 
    print("Have you registered to vote yet?")
```

13. 当需要在条件测试通过时执行一个操作，在没有通过时执行另一个操作，可使用Python提供的if-else语句。

14. 我们经常需要检查超过两个的情形，为此可使用Python提供的if-elif-else结构。Python只执行if-elif-else结构中的一个代码块。它依次检查每个条件测试，直到遇到通过了的条件测试。
```
// 例子1：
age = 12

if age < 4:
    print("Your admission cost is $0.")
elif age < 18:
    print("Your admission cost is $25.")
else:
    print("Your admission cost is $40.")
        
// 例子2：
age = 12

if age < 4:
    price = 0
elif age < 18:
    price = 25
else:
    price = 40
print(f"Your admission cost is ${price}.")
```

15. 可根据需要使用任意数量的elif代码块。

16. Python并不要求if-elif结构后面必须有else代码块。在有些情况下，else代码块很有用；而在其他一些情况下，使用一条elif语句来处理特定的情形更清晰。
```
age = 12

if age < 4:
    price = 0 
elif age < 18:
    price = 25
elif age < 65: 
    price = 40
elif age >= 65:
    price = 20

print(f"Your admission cost is ${price}.")
```
- else是一条包罗万象的语句，只要不满足任何if或elif中的条件测试，其中的代码就会执行。这可能引入无效甚至恶意的数据。如果知道最终要测试的条件，应考虑使用一个elif代码块来代替else代码块。这样就可以肯定，仅当满足相应的条件时，代码才会执行。

17. 如果只想执行一个代码块，就使用if-elif-else结构；如果要执行多个代码块，就使用一系列独立的if语句。

18. 在运行for循环前确定列表是否为空很重要。
```
requested_toppings = []

if requested_toppings:
    for requested_topping in requested_toppings: 
        print(f"Adding {requested_topping}.")
    print("\nFinished making your pizza!")
else:
    print("Are you sure you want a plain pizza?")
```
- 在if语句中将列表名用作条件表达式时，Python将在列表至少包含一个元素时返回True，并在列表为空时返回False。

19. 在条件测试的格式设置方面，PEP 8提供的唯一建议是，在诸如==、>=和<=等比较运算符两边各添加一个空格。

***

###### 六、字典

1. 在Python中，字典是一系列键值对。每个键都与一个值相关联，你可使用键来访问相关联的值。与键相关联的值可以是数、字符串、列表乃至字典。事实上，可将任何Python对象用作字典中的值。
```
alien_0 = {'color': 'green', 'points': 5}

print(alien_0['color'])         // green 
print(alien_0['points'])        // 5
```
- 在Python中，字典用放在花括号（{}）中的一系列键值对表示。
- 键和值之间用冒号分隔，而键值对之间用逗号分隔。在字典中，想存储多少个键值对都可以。
- 要获取与键相关联的值，可依次指定字典名和放在方括号内的键。

2. 字典是一种动态结构，可随时在其中添加键值对。要添加键值对，可依次指定字典名、用方括号括起的键和相关联的值。
```
alien_0 = {'color': 'green', 'points': 5}
print(alien_0)

alien_0['x_position'] = 0
alien_0['y_position'] = 25 
print(alien_0)  // {'color': 'green', 'points': 5, 'x_position': 0, 'y_position': 25}
```
- 注意，在Python 3.7中，字典中元素的排列顺序与定义时相同。如果将字典打印出来或遍历其元素，将发现元素的排列顺序与添加顺序相同。

3. 可使用一对空花括号定义一个字典，再分行添加各个键值对。
```
alien_0 = {}

alien_0['color'] = 'green' 
alien_0['points'] = 5
```
- 使用字典来存储用户提供的数据或在编写能自动生成大量键值对的代码时，通常需要先定义一个空字典。

4. 要修改字典中的值，可依次指定字典名、用方括号括起的键，以及与该键相关联的新值。
```
alien_0 = {'color': 'green'}
alien_0['color'] = 'yellow'
```

5. 对于字典中不再需要的信息，可使用del语句将相应的键值对彻底删除。使用del语句时，必须指定字典名和要删除的键。
```
alien_0 = {'color': 'green', 'points': 5}
del alien_0['points'] 
```
- 注意，删除的键值对会永远消失。

6. 确定需要使用多行来定义字典时，要在输入左花括号后按回车键。在下一行缩进四个空格，指定第一个键值对，并在它后面加上一个逗号。此后再按回车键时，文本编辑器将自动缩进后续键值对，且缩进量与第一个键值对相同。定义好字典后，在最后一个键值对的下一行添加一个右花括号，并缩进四个空格，使其与字典中的键对齐。一种不错的做法是，在最后一个键值对后面也加上逗号，为以后在下一行添加键值对做好准备。
```
favorite_languages = {
    'jen': 'python',
    'sarah': 'c',
    'edward': 'ruby',
    'phil': 'python',
    }
```
- 注意，对于较长的列表和字典，大多数编辑器提供了以类似方式设置格式的功能。对于较长的字典，还有其他一些可行的格式设置方式，因此在你的编辑器或其他源代码中，你可能会看到稍微不同的格式设置方式。

7. 使用放在方括号内的键从字典中获取感兴趣的值时，可能会引发问题：如果指定的键不存在就会出错。但就字典而言，可使用方法get()在指定的键不存在时返回一个默认值，从而避免这样的错误。方法get()的第一个参数用于指定键，是必不可少的；第二个参数为指定的键不存在时要返回的值，是可选的。
```
alien_0 = {'color': 'green', 'speed': 'slow'}

point_value = alien_0.get('points', 'No point value assigned.') 
print(point_value)
```
- 如果指定的键有可能不存在，应考虑使用方法get()，而不要使用方括号表示法。
- 注意，调用get()时，如果没有指定第二个参数且指定的键不存在，Python将返回值None。这个特殊值表示没有相应的值。None并非错误，而是一个表示所需值不存在的特殊值。

8. 字典可用于以各种方式存储信息，因此有多种遍历方式：可遍历字典的所有键值对，也可仅遍历键或值。

9. 使用for循环来遍历字典
```
user_0 = {
    'username': 'efermi', 
    'first': 'enrico',
    'last': 'fermi',
    }

for key, value in user_0.items():
    print(f"\nKey: {key}")
    print(f"Value: {value}")
```
- 编写遍历字典的for循环，可声明两个变量，用于存储键值对中的键和值。这两个变量可以使用任意名称。
- 方法items()返回一个键值对列表。

10. 在不需要使用字典中的值时，方法keys()很有用。
```
favorite_languages = { 
    'jen': 'python',
    'sarah': 'c',
    'edward': 'ruby',
    'phil': 'python',
    }

for name in favorite_languages.keys():
    print(name.title())
```
- 遍历字典时，会默认遍历所有的键。因此，如果将上述代码中的：`for name in favorite_languages:`替换为：`for name in favorite_languages.keys():`输出将不变。显式地使用方法keys()可让代码更容易理解，你可以选择这样做，但是也可以省略它。

11. 方法keys()并非只能用于遍历：实际上，它返回一个列表，其中包含字典中的所有键。
```
favorite_languages = { 
    'jen': 'python',
    'sarah': 'c',
    'edward': 'ruby',
    'phil': 'python',
    }

if 'erin' not in favorite_languages.keys():
    print("Erin, please take our poll!")
```

12. 从Python 3.7起，遍历字典时将按插入的顺序返回其中的元素。要以特定顺序返回元素，一种办法是在for循环中对返回的键进行排序。为此，可使用函数sorted()来获得按特定顺序排列的键列表的副本：
```
favorite_languages = { 
    'jen': 'python',
    'sarah': 'c',
    'edward': 'ruby',
    'phil': 'python',
    }
    
for name in sorted(favorite_languages.keys()): 
    print(f"{name.title()}, thank you for taking the poll.")
```

13. 可使用方法values()来返回一个值列表，不包含任何键。
```
favorite_languages = { 
    'jen': 'python',
    'sarah': 'c',
    'edward': 'ruby',
    'phil': 'python',
    }
    
for language in favorite_languages.values():
    print(language.title())

for language in set(favorite_languages.values()): 
    print(language.title())
```
- 为剔除重复项，可使用集合（set）。集合中的每个元素都必须是独一无二的。
- 通过对包含重复元素的列表调用set()，可让Python找出列表中独一无二的元素，并使用这些元素来创建一个集合。

14. 可使用一对花括号直接创建集合，并在其中用逗号分隔元素。
```
>>> languages = {'python', 'ruby', 'python', 'c'}
>>> languages
{'ruby', 'python', 'c'}
```
- 集合和字典很容易混淆，因为它们都是用一对花括号定义的。当花括号内没有键值对时，定义的很可能是集合。不同于列表和字典，集合不会以特定的顺序存储元素。

15. 字典列表
```
// 例子1：
alien_0 = {'color': 'green', 'points': 5} 
alien_1 = {'color': 'yellow', 'points': 10} 
alien_2 = {'color': 'red', 'points': 15}

aliens = [alien_0, alien_1, alien_2] 

for alien in aliens:
    print(alien)
  
// 例子2：  
# 创建一个用于存储外星人的空列表。
aliens = []

# 创建30个绿色的外星人。
for alien_number in range(30):
    new_alien = {'color': 'green', 'points': 5, 'speed': 'slow'}
    aliens.append(new_alien)

# 显示前5个外星人。
for alien in aliens[:5]: 
    print(alien)
print("...")

# 显示创建了多少个外星人。
print(f"Total number of aliens: {len(aliens)}")
```
- range()返回一系列数，其唯一的用途是告诉Python要重复这个循环多少次。

16. 在字典中存储列表
```
# 存储所点比萨的信息。
pizza = {
    'crust': 'thick',
    'toppings': ['mushrooms', 'extra cheese'],
    }

# 概述所点的比萨。
print(f"You ordered a {pizza['crust']}-crust pizza " 
    "with the following toppings:")

for topping in pizza['toppings']:
    print(f"\t{topping}")
```
- 在这个字典中，一个键是'crust'，与之相关联的值是字符串'thick'；下一个键是'toppings'，与之相关联的值是一个列表，
- 如果函数调用print()中的字符串很长，可以在合适的位置分行。只需要在每行末尾都加上引号，同时对于除第一行外的其他各行，都在行首加上引号并缩进。这样，Python将自动合并圆括号内的所有字符串。
- 每当需要在字典中将一个键关联到多个值时，都可以在字典中嵌套一个列表。
- 可在遍历字典的for循环开头添加一条if语句，通过查看len(languages)的值来确定当前的被调查者喜欢的语言是否有多种。
- 注意，列表和字典的嵌套层级不应太多。如果嵌套层级比前面的示例多得多，很可能有更简单的解决方案。

17. 可在字典中嵌套字典，但这样做时，代码可能很快复杂起来。
```
users = {
    'aeinstein': {
        'first': 'albert',
        'last': 'einstein',
        'location': 'princeton',
        },
    'mcurie': {
        'first': 'marie',
        'last': 'curie',
        'location': 'paris',
        },
    }
for username, user_info in users.items():
    print(f"\nUsername: {username}")
    full_name = f"{user_info['first']} {user_info['last']}"
    location = user_info['location']
    print(f"\tFull name: {full_name.title()}")
    print(f"\tLocation: {location.title()}")
```
- 注意，表示每位用户的字典都具有相同的结构。虽然Python并没有这样的要求，但这使得嵌套的字典处理起来更容易。

***

###### 七、用户输入和 while 循环

1. 函数input()让程序暂停运行，等待用户输入一些文本。获取用户输入后，Python将其赋给一个变量，以方便你使用。
```
message = input("Tell me something, and I will repeat it back to you: ") 
print(message)

prompt = "If you tell us who you are, we can personalize the messages you see." 
prompt += "\nWhat is your first name? "
name = input(prompt) 
print(f"\nHello, {name}!")
```
- 函数input()接受一个参数——要向用户显示的提示或说明，让用户知道该如何做。
- 注意，Sublime Text等众多编辑器不能运行提示用户输入的程序。你可以使用Sublime Text来编写提示用户输入的程序，但必须从终端运行它们。
- 运算符+=在前面赋给变量prompt的字符串末尾附加一个字符串。
- **使用函数input()时，Python将用户输入解读为字符串。**

2. 可使用函数int()，它让Python将输入视为数值。函数int()将数的字符串表示转换为数值表示。
```
>>> age = input("How old are you? ") 
How old are you? 21
>>> age = int(age)
>>> age >= 18 
True
```
- 用户根据提示输入21后，Python将这个数解读为字符串，但随后int()将这个字符串转换成了数值表示。
- 将数值输入用于计算和比较前，务必将其转换为数值表示。

3. 处理数值信息时，求模运算符（%）是个很有用的工具，它将两个数相除并返回余数。如果一个数可被另一个数整除，余数就为0，因此求模运算将返回0。可利用这一点来判断一个数是奇数还是偶数。
```
number = input("Enter a number, and I'll tell you if it's even or odd: ") 
number = int(number)

if number % 2 == 0:
    print(f"\nThe number {number} is even.") 
else:
    print(f"\nThe number {number} is odd.")
```

4. for循环用于针对集合中的每个元素都执行一个代码块，而while循环则不断运行，直到指定的条件不满足为止。

5. 可使用while循环来数数。
```
current_number = 1
while current_number <= 5: 
    print(current_number) 
    current_number += 1
```

6. 可以使用while循环让程序在用户愿意时不断运行。
```
prompt = "\nTell me something, and I will repeat it back to you:" 
prompt += "\nEnter 'quit' to end the program. "
message = ""
while message != 'quit': 
    message = input(prompt)
    if message != 'quit':
        print(message)
```

7. 在要求很多条件都满足才继续运行的程序中，可定义一个变量，用于判断整个程序是否处于活动状态。这个变量称为标志（flag），充当程序的交通信号灯。可以让程序在标志为True时继续运行，并在任何事件导致标志的值为False时让程序停止运行。这样，在while语句中就只需检查一个条件：标志的当前值是否为True。然后将所有其他测试（是否发生了应将标志设置为False的事件）都放在其他地方，从而让程序更整洁。
```
prompt = "\nTell me something, and I will repeat it back to you:" 
prompt += "\nEnter 'quit' to end the program. "

active = True
while active:
    message = input(prompt)
    if message == 'quit': 
        active = False
    else:
        print(message)
```

8. 要立即退出while循环，不再运行循环中余下的代码，也不管条件测试的结果如何，可使用break语句。break语句用于控制程序流程，可用来控制哪些代码行将执行、哪些代码行不执行，从而让程序按你的要求执行你要执行的代码。
```
prompt = "\nPlease enter the name of a city you have visited:" 
prompt += "\n(Enter 'quit' when you are finished.) "

while True:
    city = input(prompt)

    if city == 'quit': 
        break
    else:
        print(f"I'd love to go to {city.title()}!")
```
- 注意，在任何Python循环中都可使用break语句。例如，可使用break语句来退出遍历列表或字典的for循环。

9. 要返回循环开头，并根据条件测试结果决定是否继续执行循环，可使用continue语句，它不像break语句那样不再执行余下的代码并退出整个循环。
```
current_number = 0
while current_number < 10:
    current_number += 1
    if current_number % 2 == 0: 
        continue
    print(current_number)
```

10. 如果程序陷入无限循环，可按Ctrl + C，也可关闭显示程序输出的终端窗口。注意，Sublime Text等一些编辑器内嵌了输出窗口，这可能导致难以结束无限循环，不得不通过关闭编辑器来结束。在这种情况下，可在输出窗口中单击鼠标，再按Ctrl + C，这样应该能够结束无限循环。

11. for循环是一种遍历列表的有效方式，但不应在for循环中修改列表，否则将导致Python难以跟踪其中的元素。要在遍历列表的同时对其进行修改，可使用while循环。

12. 在列表之间移动元素
```
# 首先，创建一个待验证用户列表
# 和一个用于存储已验证用户的空列表。
unconfirmed_users = ['alice', 'brian', 'candace']
confirmed_users = []

# 验证每个用户，直到没有未验证用户为止。
# 将每个经过验证的用户都移到已验证用户列表中。
while unconfirmed_users:
    current_user = unconfirmed_users.pop()
    print(f"Verifying user: {current_user.title()}")
    confirmed_users.append(current_user)
 
# 显示所有已验证的用户。
print("\nThe following users have been confirmed:")
for confirmed_user in confirmed_users:
    print(confirmed_user.title())
```
- 方法pop()以每次一个的方式从列表unconfirmed_users末尾删除未验证的用户。

13. 如果要删除列表中所有为特定值的元素，可不断运行一个while循环，直到列表中不再包含特定值。
```
pets = ['dog', 'cat', 'dog', 'goldfish', 'cat', 'rabbit', 'cat'] 
print(pets)

while 'cat' in pets: 
    pets.remove('cat')

print(pets)
```

14. 可使用while循环提示用户输入任意多的信息。
```
responses = {}

# 设置一个标志，指出调查是否继续。
polling_active = True

while polling_active:
    # 提示输入被调查者的名字和回答。
    name = input("\nWhat is your name? ")
    response = input("Which mountain would you like to climb someday? ")
 
    # 将回答存储在字典中。
    responses[name] = response
 
    # 看看是否还有人要参与调查。
    repeat = input("Would you like to let another person respond? (yes/ no) ")
    if repeat == 'no':
        polling_active = False
    
# 调查结束，显示结果。
print("\n--- Poll Results ---")
for name, response in responses.items():
    print(f"{name} would like to climb {response}.")
```

***

###### 八、函数

1. 使用关键字def来告诉Python，你要定义一个函数。这是函数定义，向Python指出了函数名，还可能在圆括号内指出函数为完成任务需要什么样的信息。
```
def greet_user():
    """显示简单的问候语。"""    // A处
    print("Hello!")

greet_user()
```
- 在这里，函数名为greet_user()，它不需要任何信息就能完成工作，因此括号是空的（即便如此，括号也必不可少）。最后，定义以冒号结尾。
- 紧跟在def greet_user(): 后面的所有缩进行构成了函数体。
- A处的文本是称为文档字符串（docstring）的注释，描述了函数是做什么的。文档字符串用三引号括起，Python使用它们来生成有关程序中函数的文档。
- 要调用函数，可依次指定函数名以及用圆括号括起的必要信息。

2. 向函数传递信息
```
def greet_user(username): 
    """显示简单的问候语。"""
    print(f"Hello, {username.title()}!") 

greet_user('jesse')
```
- 在函数greet_user()的定义中，变量username是一个形参（parameter），即函数完成工作所需的信息。在代码greet_user('jesse')中，值'jesse'是一个实参（argument），即调用函数时传递给函数的信息。调用函数时，将要让函数使用的信息放在圆括号内。在greet_user('jesse')中，将实参'jesse'传递给了函数greet_user()，这个值被赋给了形参username。

3. 向函数传递实参的方式很多：可使用位置实参，这要求实参的顺序与形参的顺序相同；也可使用关键字实参，其中每个实参都由变量名和值组成；还可使用列表和字典。

4. 调用函数时，Python必须将函数调用中的每个实参都关联到函数定义中的一个形参。为此，最简单的关联方式是基于实参的顺序。这种关联方式称为位置实参。
```
def describe_pet(animal_type, pet_name): 
    """显示宠物的信息。"""
    print(f"\nI have a {animal_type}.")
    print(f"My {animal_type}'s name is {pet_name.title()}.")

describe_pet('hamster', 'harry')
describe_pet('dog', 'willie')
```
- 在函数中，可根据需要使用任意数量的位置实参，Python将按顺序将函数调用中的实参关联到函数定义中相应的形参。
- 使用位置实参来调用函数时，如果实参的顺序不正确，结果可能出乎意料。

5. 关键字实参是传递给函数的名称值对。因为直接在实参中将名称和值关联起来，所以向函数传递实参时不会混淆。关键字实参让你无须考虑函数调用中的实参顺序，还清楚地指出了函数调用中各个值的用途。
```
def describe_pet(animal_type, pet_name): 
    """显示宠物的信息。"""
    print(f"\nI have a {animal_type}.")
    print(f"My {animal_type}'s name is {pet_name.title()}.") 
    
describe_pet(animal_type='hamster', pet_name='harry')
```
- 调用这个函数时，向Python明确地指出了各个实参对应的形参。
- 关键字实参的顺序无关紧要，因为Python知道各个值该赋给哪个形参。
- 注意，使用关键字实参时，务必准确指定函数定义中的形参名。

6. 编写函数时，可给每个形参指定默认值。在调用函数中给形参提供了实参时，Python将使用指定的实参值；否则，将使用形参的默认值。因此，给形参指定默认值后，可在函数调用中省略相应的实参。使用默认值可简化函数调用，还可清楚地指出函数的典型用法。
```
def describe_pet(pet_name, animal_type='dog'): 
    """显示宠物的信息。"""
    print(f"\nI have a {animal_type}.")
    print(f"My {animal_type}'s name is {pet_name.title()}.") 
    
describe_pet(pet_name='willie')
describe_pet('willie')

// 等效的函数调用
describe_pet(pet_name='harry', animal_type='hamster')
describe_pet(animal_type='hamster', pet_name='harry')
describe_pet('harry', 'hamster') 
```
- 请注意，在这个函数的定义中，修改了形参的排列顺序。因为给animal_type指定了默认值，无须通过实参来指定动物类型，所以在函数调用中只包含一个实参——宠物的名字。然而，Python依然将这个实参视为位置实参，因此如果函数调用中只包含宠物的名字，这个实参将关联到函数定义中的第一个形参。这就是需要将pet_name放在形参列表开头的原因。
- 注意，使用默认值时，必须先在形参列表中列出没有默认值的形参，再列出有默认值的实参。这让Python依然能够正确地解读位置实参。

7. 函数并非总是直接显示输出，它还可以处理一些数据，并返回一个或一组值。函数返回的值称为返回值。在函数中，可使用return语句将值返回到调用函数的代码行。
```
def get_formatted_name(first_name, last_name):
    """返回整洁的姓名。"""
    full_name = f"{first_name} {last_name}"
    return full_name.title()

musician = get_formatted_name('jimi', 'hendrix') 
print(musician)     // Jimi Hendrix
```

8. 让实参变成可选的
```
def get_formatted_name(first_name, last_name, middle_name=''):
    """返回整洁的姓名。"""
	if middle_name:
        full_name = f"{first_name} {middle_name} {last_name}"
	else:
        full_name = f"{first_name} {last_name}" 
    return full_name.title()

musician = get_formatted_name('jimi', 'hendrix') 
print(musician)

musician = get_formatted_name('john', 'hooker', 'lee') 
print(musician)
```
- 在函数体中，检查是否提供了中间名。Python将非空字符串解读为True。

9. 函数可返回任何类型的值，包括列表和字典等较复杂的数据结构。
```
// 例子1：
def build_person(first_name, last_name):     
    """返回一个字典，其中包含有关一个人的信息。"""
    person = {'first': first_name, 'last': last_name}
    return person

musician = build_person('jimi', 'hendrix')
print(musician)     // {'first': 'jimi', 'last': 'hendrix'}

// 例子2：
def build_person(first_name, last_name, age=None): 
    """返回一个字典，其中包含有关一个人的信息。"""
    person = {'first': first_name, 'last': last_name}
    if age:
        person['age'] = age 
    return person

musician = build_person('jimi', 'hendrix', age=27) 
print(musician)
```
- 在函数定义中，新增了一个可选形参age，并将其默认值设置为特殊值None（表示变量没有值）。可将None视为占位值。在条件测试中，None相当于False。

10. 向函数传递列表
```
def greet_users(names):
    """向列表中的每位用户发出简单的问候。""" 
    for name in names:
        msg = f"Hello, {name.title()}!" 
        print(msg)

usernames = ['hannah', 'ty', 'margot'] 
greet_users(usernames)
```

11. 将列表传递给函数后，函数就可对其进行修改。**在函数中对这个列表所做的任何修改都是永久性的。**
```
def print_models(unprinted_designs, completed_models): 
    """
    模拟打印每个设计，直到没有未打印的设计为止。
    打印每个设计后，都将其移到列表completed_models中。
    """
    while unprinted_designs:
        current_design = unprinted_designs.pop() 
        print(f"Printing model: {current_design}") 
        completed_models.append(current_design)

def show_completed_models(completed_models): 
    """显示打印好的所有模型。"""
    print("\nThe following models have been printed:") 
    for completed_model in completed_models:
        print(completed_model)

unprinted_designs = ['phone case', 'robot pendant', 'dodecahedron']
completed_models = []

print_models(unprinted_designs, completed_models) 
show_completed_models(completed_models)
```

12. 如果需要禁止函数修改列表，可向函数传递列表的副本而非原件。
```
function_name(list_name_[:])

例如 unprinted_designs 需要传递副本：
print_models(unprinted_designs[:], completed_models)
```
- 虽然向函数传递列表的副本可保留原始列表的内容，但除非有充分的理由，否则还是应该将原始列表传递给函数。这是因为让函数使用现成的列表可避免花时间和内存创建副本，从而提高效率，在处理大型列表时尤其如此。

13. 传递任意数量的实参
```
def make_pizza(*toppings): 
    """打印顾客点的所有配料。""" 
    print(toppings)

make_pizza('pepperoni')         // ('pepperoni',)
make_pizza('mushrooms', 'green peppers', 'extra cheese')    // ('mushrooms', 'green peppers', 'extra cheese')
```
- 有时候，预先不知道函数需要接受多少个实参，好在Python允许函数从调用语句中收集任意数量的实参。
- 形参名*toppings中的星号让Python创建一个名为toppings的空元组，并将收到的所有值都封装到这个元组中。
- 注意，Python将实参封装到一个元组中，即便函数只收到一个值：

14. 如果要让函数接受不同类型的实参，必须在函数定义中将接纳任意数量实参的形参放在最后。Python先匹配位置实参和关键字实参，再将余下的实参都收集到最后一个形参中。
```
def make_pizza(size, *toppings): 
    """概述要制作的比萨。"""
    print(f"\nMaking a {size}-inch pizza with the following toppings:") 
    for topping in toppings:
        print(f"- {topping}")

make_pizza(16, 'pepperoni')
make_pizza(12, 'mushrooms', 'green peppers', 'extra cheese')
```
- 基于上述函数定义，Python将收到的第一个值赋给形参size，并将其他所有值都存储在元组toppings中。
- 注意，你经常会看到通用形参名*args，它也收集任意数量的位置实参。

15. 使用任意数量的关键字实参
```
def build_profile(first, last, **user_info):
    """创建一个字典，其中包含我们知道的有关用户的一切。"""
    user_info['first_name'] = first 
    user_info['last_name'] = last 
    return user_info

user_profile = build_profile('albert', 'einstein',
                             location='princeton', 
                             field='physics')
print(user_profile)     // {'location': 'princeton', 'field': 'physics', 'first_name': 'albert', 'last_name': 'einstein'}
```
- 有时候，需要接受任意数量的实参，但预先不知道传递给函数的会是什么样的信息。在这种情况下，可将函数编写成能够接受任意数量的键值对——调用语句提供了多少就接受多少。
- 形参**user_info 中的两个星号让Python创建一个名为user_info的空字典，并将收到的所有名称值对都放到这个字典中。
- 注意，你经常会看到形参名**kwargs，它用于收集任意数量的关键字实参。

16. 将函数存储在模块中
- 使用函数的优点之一是可将代码块与主程序分离。
- 将函数存储在称为模块的独立文件中，再将模块导入到主程序中。import语句允许在当前运行的程序文件中使用模块中的代码。
- 将函数存储在独立文件中后，可与其他程序员共享这些文件而不是整个程序。

17. 导入整个模块
- 要让函数是可导入的，得先创建模块。模块是扩展名为.py的文件，包含要导入到程序中的代码。
```
// pizza.py 文件
def make_pizza(size, *toppings): 
    """概述要制作的比萨。"""
    print(f"\nMaking a {size}-inch pizza with the following toppings:") 
    for topping in toppings:
        print(f"- {topping}")
        
// 在pizza.py所在的目录中创建一个名为making_pizzas.py的文件。这个文件导入刚创建的模块，再调用make_pizza() 两次：
import pizza

pizza.make_pizza(16, 'pepperoni')
pizza.make_pizza(12, 'mushrooms', 'green peppers', 'extra cheese')
```
- 代码行import pizza让Python打开文件pizza.py，并将其中的所有函数都复制到这个程序中。
- 要调用被导入模块中的函数，可指定被导入模块的名称pizza和函数名make_pizza()，并用句点分隔。
- 这种导入方法只需编写一条import语句并在其中指定模块名，就可在程序中使用该模块中的所有函数。

18. 导入模块中的特定函数
```
from pizza import make_pizza

make_pizza(16, 'pepperoni')
make_pizza(12, 'mushrooms', 'green peppers', 'extra cheese')
```
- 使用这种语法时，调用函数时无须使用句点。由于在import 语句中显式地导入了函数make_pizza()，调用时只需指定其名称即可。

19. 使用as给函数指定别名。
```
from pizza import make_pizza as mp

mp(16, 'pepperoni')
mp(12, 'mushrooms', 'green peppers', 'extra cheese')
```
- 如果要导入函数的名称可能与程序中现有的名称冲突，或者函数的名称太长，可指定简短而独一无二的别名：函数的另一个名称，类似于外号。要给函数取这种特殊外号，需要在导入它时指定。

20. 使用as 给模块指定别名
```
import pizza as p

p.make_pizza(16, 'pepperoni')
p.make_pizza(12, 'mushrooms', 'green peppers', 'extra cheese')
```
- 还可以给模块指定别名。通过给模块指定简短的别名（如给模块pizza指定别名p），让你能够更轻松地调用模块中的函数。

21. 导入模块中的所有函数
```
from pizza import *

make_pizza(16, 'pepperoni')
make_pizza(12, 'mushrooms', 'green peppers', 'extra cheese')
```
- 使用星号（*）运算符可让Python导入模块中的所有函数。
- import语句中的星号让Python将模块pizza中的每个函数都复制到这个程序文件中。由于导入了每个函数，可通过名称来调用每个函数，而无须使用句点表示法。然而，使用并非自己编写的大型模块时，**最好不要采用这种导入方法**。这是因为如果模块中有函数的名称与当前项目中使用的名称相同，可能导致意想不到的结果：Python可能遇到多个名称相同的函数或变量，进而覆盖函数，而不是分别导入所有的函数。
- **最佳的做法是，要么只导入需要使用的函数，要么导入整个模块并使用句点表示法。**

22. 函数编写指南
- 应给函数指定描述性名称，且只在其中使用小写字母和下划线。
- 每个函数都应包含简要地阐述其功能的注释。该注释应紧跟在函数定义后面，并采用文档字符串格式。
- 给形参指定默认值时，等号两边不要有空格。对于函数调用中的关键字实参，也应遵循这种约定。
    ```
    def function_name(parameter_0, parameter_1='default value')
    function_name(value_0, parameter_1='value')
    ```
- PEP 8建议代码行的长度不要超过79字符，这样只要编辑器窗口适中，就能看到整行代码。如果形参很多，导致函数定义的长度超过了79字符，可在函数定义中输入左括号后按回车键，并在下一行按两次Tab键，从而将形参列表和只缩进一层的函数体区分开来。
    ```
    def function_name(
            parameter_0, parameter_1, parameter_2, 
            parameter_3, parameter_4, parameter_5):
    function body...
    ```
    - 大多数编辑器会自动对齐后续参数列表行，使其缩进程度与你给第一个参数列表行指定的缩进程度相同。
- 如果程序或模块包含多个函数，可使用两个空行将相邻的函数分开，这样将更容易知道前一个函数在什么地方结束，下一个函数从什么地方开始。
- 所有import 语句都应放在文件开头。唯一例外的情形是，在文件开头使用了注释来描述整个程序。

***

###### 九、类

1. 一个简单的Dog类
```
class Dog:
    """一次模拟小狗的简单尝试。"""
    def __init__(self, name, age):
        """初始化属性name和age。"""
        self.name = name
        self.age = age

    def sit(self):
        """模拟小狗收到命令时蹲下。"""
        print(f"{self.name} is now sitting.")
 
    def roll_over(self):
        """模拟小狗收到命令时打滚。"""
    print(f"{self.name} rolled over!")
```
- 根据约定，在Python中，首字母大写的名称指的是类。这个类定义中没有圆括号，因为要从空白创建这个类（读者备注：“空白”指的是没有父类要继承）。
- 类中的函数称为方法。
- \_\_init\_\_()是一个特殊方法，每当你根据Dog类创建新实例时，Python都会自动运行它。在这个方法的名称中，开头和末尾各有两个下划线，这是一种约定，旨在避免Python默认方法与普通方法发生名称冲突。务必确保\_\_init\_\_()的两边都有两个下划线，否则当你使用类来创建实例时，将不会自动调用这个方法，进而引发难以发现的错误。
- 我们将方法\_\_init\_\_()定义成包含三个形参：self、name和age。在这个方法的定义中，形参self必不可少，而且必须位于其他形参的前面。为何必须在方法定义中包含形参self呢？因为Python调用这个方法来创建Dog实例时，将自动传入实参self。
- 每个与实例相关联的方法调用都自动传递实参self，它是一个指向实例本身的引用，让实例能够访问类中的属性和方法。创建Dog实例时，Python将调用Dog类的方法\_\_init\_\_()。我们将通过实参向Dog()传递名字和年龄，self会自动传递，因此不需要传递它。每当根据Dog类创建实例时，都只需给最后两个形参（name和age）提供值。
- 以self为前缀的变量可供类中的所有方法使用，可以通过类的任何实例来访问。self.name = name 获取与形参name相关联的值，并将其赋给变量name，然后该变量被关联到当前创建的实例。

2. 根据类创建实例
```
my_dog = Dog('Willie', 6)       // A处

print(f"My dog's name is {my_dog.name}.")   // B处
print(f"My dog is {my_dog.age} years old.")

my_dog.sit()
my_dog.roll_over()
```
- 遇到A处这行代码时，Python使用实参'Willie'和6调用Dog类的方法\_\_init\_\_()。方法\_\_init\_\_()创建一个表示特定小狗的实例，并使用提供的值来设置属性name和age。接下来，Python返回一个表示这条小狗的实例，而我们将这个实例赋给了变量my_dog。在这里，命名约定很有用：通常可认为首字母大写的名称（如Dog）指的是类，而小写的名称（如my_dog）指的是根据类创建的实例。
- 要访问实例的属性，可使用句点表示法。
- 在B处这里，Python先找到实例my_dog，再查找与该实例相关联的属性name。在Dog类中引用这个属性时，使用的是self.name。
- 使用句点表示法来调用Dog类中定义的任何方法。

3. 创建实例时，有些属性无须通过形参来定义，可在方法\_\_init\_\_()中为其指定默认值。
```
class Car:
    def init (self, make, model, year): 
        """初始化描述汽车的属性。""" 
        self.make = make
        self.model = model 
        self.year = year 
        self.odometer_reading = 0
```

4. 我们能以三种方式修改属性的值：直接通过实例进行修改，通过方法进行设置，以及通过方法进行递增（增加特定的值）。
```
1、直接修改属性的值
my_new_car.odometer_reading = 23 

2、通过方法修改属性的值
class Car:
    ...
    def update_odometer(self, mileage): 
        """将里程表读数设置为指定的值。""" 
        self.odometer_reading = mileage

my_new_car.update_odometer(23) 

3、通过方法对属性的值进行递增
class Car:
    ...
    def increment_odometer(self, miles): 
        """将里程表读数增加指定的量。""" 
        self.odometer_reading += miles

my_used_car.increment_odometer(100)
```
- 可对方法update_odometer()进行扩展，使其在修改里程表读数时做些额外的工作。
- 可以轻松地修改increment_odometer()方法，以禁止增量为负值，从而防止有人利用它来回调里程表。
- 注意，你可以使用类似于上面的方法来控制用户修改属性值（如里程表读数）的方式，但能够访问程序的人都可以通过直接访问属性来将里程表修改为任何值。要确保安全，除了进行类似于前面的基本检查外，还需特别注意细节。

5. 编写类时，并非总是要从空白开始。如果要编写的类是另一个现成类的特殊版本，可使用继承。一个类继承另一个类时，将自动获得另一个类的所有属性和方法。原有的类称为父类，而新类称为子类。子类继承了父类的所有属性和方法，同时还可以定义自己的属性和方法。

6. 在既有类的基础上编写新类时，通常要调用父类的方法\_\_init\_\_()。这将初始化在父类\_\_init\_\_()方法中定义的所有属性，从而让子类包含这些属性。
```
class Car:
    """一次模拟汽车的简单尝试。"""
    def __init__(self, make, model, year):
        self.make = make
        self.model = model
        self.year = year
        self.odometer_reading = 0
 
class ElectricCar(Car):
    """电动汽车的独特之处。"""
    def __init__(self, make, model, year):
        """
        初始化父类的属性。
        再初始化电动汽车特有的属性。
        """
        super().__init__(make, model, year)
        self.battery_size = 75

```
- 创建子类时，父类必须包含在当前文件中，且位于子类前面。
- 定义子类时，必须在圆括号内指定父类的名称。
- super()是一个特殊函数，让你能够调用父类的方法。这行代码让Python调用Car类的方法\_\_init\_\_()，让ElectricCar实例包含这个方法中定义的所有属性。父类也称为超类（superclass），名称super由此而来。

7. 重写父类的方法
- 对于父类的方法，只要它不符合子类模拟的实物的行为，都可以进行重写。为此，可在子类中定义一个与要重写的父类方法同名的方法。这样，Python将不会考虑这个父类方法，而只关注你在子类中定义的相应方法。
```
class ElectricCar(Car):
    --snip--
    def fill_gas_tank(self):
        """电动汽车没有油箱。"""
        print("This car doesn't need a gas tank!")
```
- 假设Car 类有一个名为fill_gas_tank()的方法，它对全电动汽车来说毫无意义，因此你可能想重写它。现在，如果有人对电动汽车调用方法fill_gas_tank()，Python将忽略Car类中的方法fill_gas_tank()，转而运行上述代码。使用继承时，可让子类保留从父类那里继承而来的精华，并剔除不需要的糟粕。

8. 将实例用作属性
```
class Battery:
    """一次模拟电动汽车电瓶的简单尝试。"""
    def __init__(self, battery_size=75):
        """初始化电瓶的属性。"""
        self.battery_size = battery_size

class ElectricCar(Car):
    """电动汽车的独特之处。"""
    def __init__(self, make, model, year):
        """
        初始化父类的属性。
        再初始化电动汽车特有的属性。
        """
        super().__init__(make, model, year)
        self.battery = Battery()
```

9. Python允许将类存储在模块中，然后在主程序中导入所需的模块。

10. 导入单个类
```
// car.py 文件
"""一个可用于表示汽车的类。"""
class Car:
    """一次模拟汽车的简单尝试。"""
    
    def __init__(self, make, model, year):
        """初始化描述汽车的属性。"""
        self.make = make
        self.model = model
        self.year = year
        self.odometer_reading = 0
    ...

// my_car.py 文件
from car import Car

my_new_car = Car('audi', 'a4', 2019)
```
- car.py 文件第一行包含一个模块级文档字符串，对该模块的内容做了简要的描述。你应为自己创建的每个模块编写文档字符串。
- my_car.py 文件第一行的import语句让Python打开模块car并导入其中的Car类。

11. 在一个模块中存储多个类
```
// car.py 文件
"""一组用于表示燃油汽车和电动汽车的类。"""
class Car:
    --snip--

class Battery:
    """一次模拟电动汽车电瓶的简单尝试。"""
    
    def __init__(self, battery_size=75):
        """初始化电瓶的属性。"""
        self.battery_size = battery_size

class ElectricCar(Car):
    """模拟电动汽车的独特之处。"""
    
    def __init__(self, make, model, year):
        """
        初始化父类的属性。
        再初始化电动汽车特有的属性。
        """
        super().__init__(make, model, year)
        self.battery = Battery()

// my_electric_car.py 文件
from car import ElectricCar

my_tesla = ElectricCar('tesla', 'model s', 2019)
```

12. 从一个模块中导入多个类
```
from car import Car, ElectricCar
```
- 可根据需要在程序文件中导入任意数量的类。从一个模块中导入多个类时，用逗号分隔了各个类。

13. 导入整个模块
```
import car

my_beetle = car.Car('volkswagen', 'beetle', 2019)
print(my_beetle.get_descriptive_name())

my_tesla = car.ElectricCar('tesla', 'roadster', 2019)
print(my_tesla.get_descriptive_name())
```
- 还可以导入整个模块，再使用句点表示法访问需要的类。这种导入方式很简单，代码也易于阅读。因为创建类实例的代码都包含模块名，所以不会与当前文件使用的任何名称发生冲突。

14. 导入模块中的所有类
```
from module_name import *
```
- 不推荐使用这种导入方式，原因有二。第一，如果只看文件开头的import语句，就能清楚地知道程序使用了哪些类，将大有裨益。然而这种导入方式没有明确地指出使用了模块中的哪些类。第二，这种方式还可能引发名称方面的迷惑。如果不小心导入了一个与程序文件中其他东西同名的类，将引发难以诊断的错误。
- 需要从一个模块中导入很多类时，最好导入整个模块，并使用module_name.ClassName语法来访问类。这样做时，虽然文件开头并没有列出用到的所有类，但你清楚地知道在程序的哪些地方使用了导入的模块。这也避免了导入模块中的每个类可能引发的名称冲突。

15. 在一个模块中导入另一个模块
```
// car.py 文件
"""一个可用于表示汽车的类。"""
class Car:
    --snip--

// electric_car.py 文件
"""一组可用于表示电动汽车的类。"""
from car import Car

class Battery:
    --snip--

class ElectricCar(Car):
    --snip--
```
- ElectricCar类需要访问其父类Car，因此直接将Car类导入该模块中。

16. 导入类时，也可为其指定别名。
```
// 在 import 语句中给 ElectricCar 指定一个别名：
from electric_car import ElectricCar as EC

my_tesla = EC('tesla', 'roadster', 2019)
```

17. Python标准库是一组模块，我们安装的Python都包含它。可以使用标准库中的任何函数和类，只需在程序开头包含一条简单的import语句即可。
```
>>> from random import randint
>>> randint(1, 6)
3

>>> from random import choice
>>> players = ['charles', 'martina', 'michael', 'florence', 'eli']
>>> first_up = choice(players)
>>> first_up
'florence'
```
- randint()将两个整数作为参数，并随机返回一个位于这两个整数之间（含）的整数。
- choice()将一个列表或元组作为参数，并随机返回其中的一个元素。

18. 类编码风格
- 类名应采用驼峰命名法，即将类名中的每个单词的首字母都大写，而不使用下划线。实例名和模块名都采用小写格式，并在单词之间加上下划线。
- 对于每个类，都应紧跟在类定义后面包含一个文档字符串。这种文档字符串简要地描述类的功能，并遵循编写函数的文档字符串时采用的格式约定。每个模块也都应包含一个文档字符串，对其中的类可用于做什么进行描述。
- 可使用空行来组织代码，但不要滥用。在类中，可使用一个空行来分隔方法；而在模块中，可使用两个空行来分隔类。
- 需要同时导入标准库中的模块和你编写的模块时，先编写导入标准库模块的import语句，再添加一个空行，然后编写导入你自己编写的模块的import语句。在包含多条import语句的程序中，这种做法让人更容易明白程序使用的各个模块都来自何处。

***

###### 十、文件和异常

1. 读取整个文件
```
with open('pi_digits.txt') as file_object:
    contents = file_object.read()
print(contents)
```
- 要以任何方式使用文件，那怕仅仅是打印其内容，都得先打开文件，才能访问它。函数open()接受一个参数：要打开的文件的名称。Python在当前执行的文件所在的目录中查找指定的文件。
- 函数open() 返回一个表示文件的对象。
- 关键字with在不再需要访问文件后将其关闭(让Python去确定：你只管打开文件，并在需要时使用它，Python自会在合适的时候自动将其关闭)。
- 在这个程序中，注意到我们调用了open()，但没有调用close()。也可以调用open()和close()来打开和关闭文件，但这样做时，如果程序存在bug导致方法close()未执行，文件将不会关闭。
- 使用方法read()读取这个文件的全部内容。
- 相比于原始文件，该输出唯一不同的地方是末尾多了一个空行。为何会多出这个空行呢？因为read()到达文件末尾时返回一个空字符串，而将这个空字符串显示出来时就是一个空行。要删除多出来的空行，可在函数调用print()中使用rstrip()：
    ```
    with open('pi_digits.txt') as file_object:
        contents = file_object.read()
    print(contents.rstrip())
    ```
    - Python方法rstrip()删除字符串末尾的空白。

2. 要让Python打开不与程序文件位于同一个目录中的文件，需要提供文件路径，让Python到系统的特定位置去查找。
```
with open('text_files/filename.txt') as file_object:

file_path = '/home/ehmatthes/other_files/text_files/_filename_.txt'
with open(file_path) as file_object:
```
- 相对文件路径让Python到指定的位置去查找，而该位置是相对于当前运行的程序所在目录的。
- 第一行代码让Python到文件夹python_work下的文件夹text_files中去查找指定的.txt文件。
- 注意，显示文件路径时，Windows系统使用反斜杠（\）而不是斜杠（/），但在代码中依然可以使用斜杠。
- 通过使用绝对路径，可读取系统中任何地方的文件。就目前而言，最简单的做法是，要么将数据文件存储在程序文件所在的目录，要么将其存储在程序文件所在目录下的一个文件夹（如text_files）中。
- 注意，如果在文件路径中直接使用反斜杠，将引发错误，因为反斜杠用于对字符串中的字符进行转义。例如，对于路径"C:\path\to\file.txt"，其中的\t将被解读为制表符。如果一定要使用反斜杠，可对路径中的每个反斜杠都进行转义，如"C:\\\path\\\to\\\file.txt" 。

3. 要以每次一行的方式检查文件，可对文件对象使用for循环。
```
filename = 'pi_digits.txt'
with open(filename) as file_object:
    for line in file_object:
        print(line)
```
- 打印每一行时，发现空白行更多了。为何会出现这些空白行呢？因为在这个文件中，每行的末尾都有一个看不见的换行符，而函数调用print()也会加上一个换行符，因此每行末尾都有两个换行符：一个来自文件，另一个来自函数调用print()。要消除这些多余的空白行，可在函数调用print()中使用rstrip()。
    ```
    filename = 'pi_digits.txt'
    with open(filename) as file_object:
        for line in file_object:
            print(line.rstrip())
    ```

4. 使用关键字with时，open()返回的文件对象只在with代码块内可用。如果要在with代码块外访问文件的内容，可在with代码块内将文件的各行存储在一个列表中，并在with代码块外使用该列表：可以立即处理文件的各个部分，也可以推迟到程序后面再处理。
```
3.1415926535
 8979323846
 2643383279

filename = 'pi_digits.txt'
with open(filename) as file_object:
    lines = file_object.readlines()

for line in lines:
    print(line.rstrip())
    
pi_string = ''
for line in lines:
    pi_string += line.strip()      // 删除每行末尾的换行符和原来位于每行左边的空格

print(pi_string)
print(len(pi_string))
```
- 方法readlines()从文件中读取每一行，并将其存储在一个列表中。
- 因为列表lines的每个元素都对应于文件中的一行，所以输出与文件内容完全一致。
- 注意，读取文本文件时，Python将其中的所有文本都解读为字符串。如果读取的是数，并要将其作为数值使用，就必须使用函数int()将其转换为整数或使用函数float()将其转换为浮点数。
- 对于可处理的数据量，Python没有任何限制。只要系统的内存足够多，你想处理多少数据都可以。

5. 要将文本写入文件，你在调用open()时需要提供另一个实参，告诉Python你要写入打开的文件。
```
filename = 'programming.txt'
with open(filename, 'w') as file_object:
    file_object.write("I love programming.")
```
- 实参（'w'）告诉Python，要以写入模式打开这个文件。打开文件时，可指定读取模式（'r'）、写入模式（'w'）、附加模式（'a'）或读写模式（'r+'）。如果省略了模式实参，Python将以默认的只读模式打开文件。如果以只读模式打开文件，当文件不存在时，则会抛异常。
- 如果要写入的文件不存在，函数open()将自动创建它。然而，以写入模式（'w'）打开文件时千万要小心，因为如果指定的文件已经存在，Python将在返回文件对象前清空该文件的内容。
- 注意，Python只能将字符串写入文本文件。要将数值数据存储到文本文件中，必须先使用函数str()将其转换为字符串格式。

6. 函数write()不会在写入的文本末尾添加换行符。要让每个字符串都单独占一行，需要在方法调用write()中包含换行符。
```
filename = 'programming.txt'
with open(filename, 'w') as file_object:
    file_object.write("I love programming.\n")
    file_object.write("I love creating new games.\n")
```

7. 如果要给文件添加内容，而不是覆盖原有的内容，可以以附加模式打开文件。以附加模式打开文件时，Python不会在返回文件对象前清空文件的内容，而是将写入文件的行添加到文件末尾。如果指定的文件不存在，Python将为你创建一个空文件。

8. Python使用称为异常的特殊对象来管理程序执行期间发生的错误。每当发生让Python不知所措的错误时，它都会创建一个异常对象。如果你编写了处理该异常的代码，程序将继续运行；如果未对异常进行处理，程序将停止并显示traceback，其中包含有关异常的报告。
- 异常是使用try-except代码块处理的。try-except代码块让Python执行指定的操作，同时告诉Python发生异常时怎么办。使用try-except代码块时，即便出现异常，程序也将继续运行：显示你编写的友好的错误消息，而不是令用户迷惑的traceback。

9. 当你认为可能会发生错误时，可编写一个try-except代码块来处理可能引发的异常。
```
try:
    print(5/0)
except ZeroDivisionError:
    print("You can't divide by zero!")
```
- 如果try代码块中的代码运行起来没有问题，Python将跳过except代码块；如果try代码块中的代码导致了错误，Python将查找与之匹配的except代码块并运行其中的代码。

10. else代码块
```
--snip--
while True:
    --snip--
    if second_number == 'q':
        break
    try:
        answer = int(first_number) / int(second_number)
    except ZeroDivisionError:
        print("You can't divide by 0!")
    else:
        print(answer)
```
- 依赖try代码块成功执行的代码都应放到else代码块中。在本例中，如果除法运算成功，就使用else代码块来打印结果。
- try-except-else代码块的工作原理大致如下：Python尝试执行try代码块中的代码，只有可能引发异常的代码才需要放在try语句中。有时候，有一些仅在try代码块成功执行时才需要运行的代码，这些代码应放在else代码块中。except代码块告诉Python，如果尝试运行try代码块中的代码时引发了指定的异常该怎么办。

11. 处理FileNotFoundError异常
```
filename = 'alice.txt'
try:
    with open(filename, encoding='utf-8') as f:
        contents = f.read()
except FileNotFoundError:
    print(f"Sorry, the file {filename} does not exist.")
```
- 给参数encoding指定了值，在系统的默认编码与要读取文件使用的编码不一致时，必须这样做。

12. 方法split()，它能根据一个字符串创建一个单词列表。
```
>>> title = "Alice in Wonderland"
>>> title.split()
['Alice', 'in', 'Wonderland']
```
- 方法split()以空格为分隔符将字符串分拆成多个部分，并将这些部分都存储到一个列表中。结果是一个包含字符串中所有单词的列表，虽然有些单词可能包含标点。

12.要让程序静默失败，可像通常那样编写try代码块，但在except代码块中明确地告诉Python什么都不要做。Python有一个pass语句，可用于让Python在代码块中什么都不要做。
```
def count_words(filename):
    """计算一个文件大致包含多少个单词。"""
    try:
        --snip--
    except FileNotFoundError:
        pass
    else:
        --snip--
```
- pass语句还充当了占位符，提醒你在程序的某个地方什么都没有做，并且以后也许要在这里做些什么。

13. 模块json让你能够将简单的Python数据结构转储到文件中，并在程序再次运行时加载该文件中的数据。你还可以使用json在Python程序之间分享数据。更重要的是，JSON数据格式并非Python专用的，这让你能够将以JSON格式存储的数据与使用其他编程语言的人分享。
- 注意，JSON（JavaScript Object Notation）格式最初是为JavaScript开发的，但随后成了一种常见格式，被包括Python在内的众多语言采用。

14. 函数json.dump()接受两个实参：要存储的数据，以及可用于存储数据的文件对象。
```
import json

numbers = [2, 3, 5, 7, 11, 13]

filename = 'numbers.json'
with open(filename, 'w') as f:
    json.dump(numbers, f)       // 写入后，文件内容为：[2, 3, 5, 7, 11, 13]
```
- 通常使用文件扩展名.json来指出文件存储的数据为JSON格式。

15. 使用json.load()将列表读取到内存中。
```
import json

filename = 'numbers.json'
with open(filename) as f:
    numbers = json.load(f)

print(numbers)      // [2, 3, 5, 7, 11, 13]
```

16. 保存和读取用户生成的数据
```
import json
# 如果以前存储了用户名，就加载它。
# 否则，提示用户输入用户名并存储它。
filename = 'username.json'
try:
    with open(filename) as f:
        username = json.load(f)
except FileNotFoundError:
    username = input("What is your name? ")
    with open(filename, 'w') as f:
        json.dump(username, f)              # 文件内容为："xxx"
        print(f"We'll remember you when you come back, {username}!")
else:
    print(f"Welcome back, {username}!")
```

17. 代码能够正确地运行，但通过将其划分为一系列完成具体工作的函数，还可以改进。这样的过程称为重构。
```
import json

def get_stored_username():
    """如果存储了用户名，就获取它。"""
    filename = 'username.json'
    try:
        with open(filename) as f:
            username = json.load(f)
    except FileNotFoundError:
        return None
    else:
        return username

def get_new_username():
    """提示用户输入用户名。"""
    username = input("What is your name? ")
    filename = 'username.json'
    with open(filename, 'w') as f:
        json.dump(username, f)
    return username

def greet_user():
    """问候用户，并指出其名字。"""
    username = get_stored_username()
    if username:
        print(f"Welcome back, {username}!")
    else:
        username = get_new_username()
        print(f"We'll remember you when you come back, {username}!")

greet_user()
```

***

###### 十一、测试代码

1. Python标准库中的模块unittest提供了代码测试工具。单元测试用于核实函数的某个方面没有问题。测试用例是一组单元测试，它们一道核实函数在各种情形下的行为都符合要求。良好的测试用例考虑到了函数可能收到的各种输入，包含针对所有这些情形的测试。全覆盖的测试用例包含一整套单元测试，涵盖了各种可能的函数使用方式。

2. 要为函数编写测试用例，可先导入模块unittest和要测试的函数，再创建一个继承unittest.TestCase的类，并编写一系列方法对函数行为的不同方面进行测试。
```
// 要测试的函数，在 name_function.py 文件中
def get_formatted_name(first, last):
    """生成整洁的姓名。"""
    full_name = f"{first} {last}"
    return full_name.title()

// test_name_function.py 文件
import unittest
from name_function import get_formatted_name

class NamesTestCase(unittest.TestCase):
    """测试name_function.py。"""
    
    def test_first_last_name(self):
        """能够正确地处理像Janis Joplin这样的姓名吗？"""
        formatted_name = get_formatted_name('janis', 'joplin')
        self.assertEqual(formatted_name, 'Janis Joplin')
        
    def test_first_last_middle_name(self):
        """能够正确地处理像Wolfgang Amadeus Mozart这样的姓名吗？"""
        formatted_name = get_formatted_name('wolfgang', 'mozart', 'amadeus')
        self.assertEqual(formatted_name, 'Wolfgang Amadeus Mozart')

if __name__ == '__main__':      // A处
    unittest.main()

// 运行结果如下：    
.
----------------------------------------------------------------------
Ran 1 test in 0.000s
OK
```
- 运行test_name_function.py时，所有以test_打头的方法都将自动运行。方法名必须以test_打头，这样它才会在我们运行test_name_function.py时自动运行。
- 断言方法核实得到的结果是否与期望的结果一致。
- 很多测试框架都会先导入测试文件再运行。导入文件时，解释器将在导入的同时执行它。
- A处的if 代码块检查特殊变量\_\_name\_\_，这个变量是在程序执行时设置的。如果这个文件作为主程序执行，变量\_\_name\_\_将被设置为'\_\_main\_\_'。在这里，调用unittest.main()来运行测试用例。如果这个文件被测试框架导入，变量\_\_name\_\_的值将不是'\_\_main\_\_'，因此不会调用unittest.main()。
- 运行结果的第一行的句点表明有一个测试通过了。接下来的一行指出Python运行了一个测试，消耗的时间不到0.001秒。最后的OK表明该测试用例中的所有单元测试都通过了。
- 可以在TestCase类中使用很长的方法名，而且这些方法名必须是描述性的，这样你才能看懂测试未通过时的输出。这些方法由Python自动调用，你根本不用编写调用它们的代码。

3. Python在unittest.TestCase类中提供了很多断言方法。下表描述了unittest模块中6个常用的断言方法。使用这些方法可核实返回的值等于或不等于预期的值，返回的值为True或False，以及返回的值在列表中或不在列表中。只能在继承unittest.TestCase的类中使用这些方法。

方法 | 用途
---|---
assertEqual(a, b)       | 核实a == b
assertNotEqual(a, b)    | 核实a != b
assertTrue(x)           | 核实x 为True
assertFalse(x)          | 核实x 为False
assertIn(item, list)    | 核实 item 在 list 中
assertNotIn(item, list) | 核实 item 不在 list 中

4. 测试类
```
// survey.py 文件，一个要测试的类
class AnonymousSurvey:
    """收集匿名调查问卷的答案。"""
    
    def __init__(self, question):
        """存储一个问题，并为存储答案做准备。"""
        self.question = question
        self.responses = []
    
    def show_question(self):
        """显示调查问卷。"""
        print(self.question)

    def store_response(self, new_response):
        """存储单份调查答卷。"""
        self.responses.append(new_response)

    def show_results(self):
        """显示收集到的所有答卷。"""
        print("Survey results:")
        for response in self.responses:
            print(f"- {response}")

// test_survey.py
import unittest
from survey import AnonymousSurvey

class TestAnonymousSurvey(unittest.TestCase):
    """针对AnonymousSurvey类的测试。"""
    def test_store_single_response(self):
        """测试单个答案会被妥善地存储。"""
        question = "What language did you first learn to speak?"
        my_survey = AnonymousSurvey(question)
        my_survey.store_response('English')
        self.assertIn('English', my_survey.responses)

    def test_store_three_responses(self):
        """测试三个答案会被妥善地存储。"""
        question = "What language did you first learn to speak?"
        my_survey = AnonymousSurvey(question)
        responses = ['English', 'Spanish', 'Mandarin']
        for response in responses:
            my_survey.store_response(response)

        for response in responses:
            self.assertIn(response, my_survey.responses)

if __name__ == '__main__':
    unittest.main()
```

5. unittest.TestCase类包含的方法setUp()让我们只需创建这些对象一次，就能在每个测试方法中使用。如果在TestCase类中包含了方法setUp()，Python将先运行它，再运行各个以test_打头的方法。这样，在你编写的每个测试方法中，都可使用在方法setUp()中创建的对象。
```
import unittest
from survey import AnonymousSurvey
    
class TestAnonymousSurvey(unittest.TestCase):
    """针对AnonymousSurvey类的测试。"""
    
    def setUp(self):
        """
        创建一个调查对象和一组答案，供使用的测试方法使用。
        """
        question = "What language did you first learn to speak?"
        self.my_survey = AnonymousSurvey(question)
        self.responses = ['English', 'Spanish', 'Mandarin']

    def test_store_single_response(self):
        """测试单个答案会被妥善地存储。"""
        self.my_survey.store_response(self.responses[0])
        self.assertIn(self.responses[0], self.my_survey.responses)
    
    def test_store_three_responses(self):
        """测试三个答案会被妥善地存储。"""
        for response in self.responses:
            self.my_survey.store_response(response)
        for response in self.responses:
            self.assertIn(response, self.my_survey.responses)

if __name__ == '__main__':
    unittest.main()
```
- 方法setUp()做了两件事情：创建一个调查对象，以及创建一个答案列表。存储这两样东西的变量名包含前缀self（即存储在属性中），因此可在这个类的任何地方使用。
- 测试自己编写的类时，方法setUp()让测试方法编写起来更容易：可在setUp()方法中创建一系列实例并设置其属性，再在测试方法中直接使用这些实例。相比于在每个测试方法中都创建实例并设置其属性，这要容易得多。

6. 运行测试用例时，每完成一个单元测试，Python都打印一个字符：测试通过时打印一个句点，测试引发错误时打印一个E，而测试导致断言失败时则打印一个F。这就是你运行测试用例时，在输出的第一行中看到的句点和字符数量各不相同的原因。如果测试用例包含很多单元测试，需要运行很长时间，就可通过观察这些结果来获悉有多少个测试通过了。
