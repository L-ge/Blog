- 结论：如下面代码运行结果所示，智能指针的实现如果是 void 指针，将会调用不了管理的对象的析构函数。
- 原因：应该是转化成了 void*，编译器找不到该对象的析构函数了。
- 影响：如果管理对象的类，里面有容器的话，将会造成较大的内存泄漏（注意容器的元素是在堆内存的，需要析构函数去 free 掉内存）。
- 其他：如果管理的对象没有被调用析构函数，那么其成员函数也就不会被调用析构函数（先析构成员变量，再析构自己。这个和构造的顺序是相反的）。

```C++
#include <QCoreApplication>
#include <QDebug>

class MyTest
{
public:
    MyTest()
    {
        qDebug() << "MyTest ctor";
    }

    ~MyTest()
    {
        qDebug() << "MyTest dector";
    }
};

class Test
{
public:
    Test(void* ptr)
        : m_ptr(ptr)
    {
        qDebug() << "Test ctor";
    }

    ~Test()
    {
        delete m_ptr;
        qDebug() << "Test dector";
    }

    void* m_ptr;
};

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    {
        MyTest *pTest = new MyTest();
        Test test(pTest);
    }

    qDebug() << "Fin";

    return a.exec();
}
```
```C++
// 运行结果：

MyTest ctor
Test ctor
Test dector
Fin
```

