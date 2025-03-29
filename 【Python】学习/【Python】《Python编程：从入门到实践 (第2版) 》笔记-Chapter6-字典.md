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
