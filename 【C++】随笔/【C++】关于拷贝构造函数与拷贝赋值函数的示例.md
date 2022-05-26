放在前面的一句话是：函数的返回值是类的对象时，GCC编译器会进行优化，不一定会调用拷贝构造函数！！！

通过示例去说明两个知识点：

① DemoClass t1 = *this; // 这个写法是直接调用拷贝构造函数,而不是拷贝赋值函数.

②在构造函数通过初值列的方式去初始化成员变量，是一次的拷贝构造函数和一次析构函数完成。而另一种方式是一次构造函数+一次拷贝赋值函数+两次析构函数。

```
#include <QCoreApplication>
#include <QDebug>

class DemoClass
{
public:
    DemoClass(){
        qDebug() << "Ctor";
    }
    ~DemoClass()
    {
        qDebug() << "DeCtor";
    }
    DemoClass(const DemoClass &demoClassTmp)
    {
        qDebug() << "Copy Ctor";
    }
    DemoClass operator=(const DemoClass &demoClassTmp)
    {
        qDebug() << "Copy Assignment";
    }
    DemoClass test1()
    {
        qDebug() << "t1 begin";
        DemoClass t1 = *this;
        qDebug() << "t1 end";
    }
};

class Demo1
{
public:
    Demo1(const DemoClass &demo) : m_demo(demo)
    {

    }

private:
    DemoClass m_demo;
};

class Demo2
{
public:
    Demo2(const DemoClass &demo)
    {
        m_demo = demo;
    }

private:
    DemoClass m_demo;
};

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    {
        DemoClass t1;
        t1.test1();
    }

    qDebug() << "=======================================";

    {
        DemoClass t1;
        DemoClass t2;
        t2 = t1;
    }

    qDebug() << "=======================================";

    {
        DemoClass t1;
        Demo1 d1(t1);
    }

    qDebug() << "=======================================";

    {
        DemoClass t1;
        Demo2 d1(t1);
    }

    return a.exec();
}
```
```
Ctor
t1 begin
Copy Ctor
t1 end
DeCtor
DeCtor
DeCtor
=======================================
Ctor
Ctor
Copy Assignment
DeCtor
DeCtor
DeCtor
=======================================
Ctor
Copy Ctor
DeCtor
DeCtor
=======================================
Ctor
Ctor
Copy Assignment
DeCtor
DeCtor
DeCtor
```