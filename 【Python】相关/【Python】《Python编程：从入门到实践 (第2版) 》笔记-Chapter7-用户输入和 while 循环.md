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
