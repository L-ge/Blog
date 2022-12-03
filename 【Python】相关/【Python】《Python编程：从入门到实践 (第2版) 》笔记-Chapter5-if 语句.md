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
