针对存在二义性的C++语句，只要它有可能被解释成函数声明，编译器就肯定将其解释成函数声明。为临时函数对象命名即可解决问题，做法是多用一对圆括号，或采用新式的统计初始化语法(列表初始化)。
```
#include <QCoreApplication>
#include <QDebug>
#include <thread>

class ATestClass1
{
public:
    static ATestClass1 &getInstance()
    {
        static ATestClass1 instance;
        return instance;
    }

    ~ATestClass1()
    {
        qDebug() << "ATestClass dector";
    }

private:
    ATestClass1()
    {
        qDebug() << "ATestClass ctor";
    }
};

class ATestClass2
{
public:
    void operator() ()
    {
        qDebug() << "operator()";
    }
};

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    ATestClass1 testClass1_1();
    void func1();
    ATestClass1 func2();
    //ATestClass1 testClass1_2;      // error: 'ATestClass1::ATestClass1()' is private

    std::thread thread1(ATestClass2());     // 会被解释成函数声明(接收的参数是函数指针，所指向的函数没有参数传入，返回ATestClass2对象)
    std::thread thread2((ATestClass2()));
    std::thread thread3{ATestClass2()};

    return a.exec();
}
```
