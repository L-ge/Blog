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
