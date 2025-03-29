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
