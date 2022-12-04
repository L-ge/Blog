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
