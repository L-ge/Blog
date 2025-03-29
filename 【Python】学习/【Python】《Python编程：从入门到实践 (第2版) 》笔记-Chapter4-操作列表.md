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
