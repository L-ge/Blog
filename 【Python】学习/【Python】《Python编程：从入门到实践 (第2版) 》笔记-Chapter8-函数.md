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
