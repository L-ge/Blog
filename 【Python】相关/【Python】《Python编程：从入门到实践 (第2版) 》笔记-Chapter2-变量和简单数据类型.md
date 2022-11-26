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
