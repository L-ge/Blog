1. 现象：在 MyLog 调用析构函数（释放了它管理的资源 m_pStr）之后，在 TestLog 的析构函数中再去访问 MyLog 类型的局部静态变量 log（它是指针类型的），此时发现仍然能调用它的成员函数。

2. 结论：
    - 先说明一句话：析构函数负责释放对象管理的资源，而销毁一个对象负责释放这个对象。
    - 出现这种现象，猜测是 log 指向的内存还没有被覆盖掉（按理来说，这种指针的用法很容易就崩溃掉了）。
    - 万一说，log 并非指针类型，而是简简单单的局部静态变量，那么猜测为 log 虽然是已经调用了析构函数，但是 log 变量所在的内存区域（.data 数据段？）还没有被回收。
    - C++ 全局静态变量的析构销毁顺序是未定义的，应该与不同编译模块的链接顺序有关。
    - 建议用指针的方式，例如在 ~MyLog() 将 m_pStr delete 并赋值为 nullptr，在 void writeLog() 函数里面先判断 m_pStr 是否不为 nullptr 再继续使用。

3. 测试代码：
```cpp
#include <QCoreApplication>
#include <QDebug>

class MyLog : public QObject
{
public:
    MyLog(QObject *parent = nullptr)
        : QObject(parent)
    {
        m_pStr = new std::string("AAA");
        qDebug() << "MyLog ctor";
    }
    ~MyLog()
    {
        delete m_pStr;
        m_pStr = nullptr;
        qDebug() << "MyLog dector";
    }
    static MyLog* getInstance(QObject *parent = 0)
    {
        static MyLog* log = new MyLog(parent);
        return log;
    }
    void writeLog()
    {
        //qDebug() << m_pStr->size();
        qDebug() << "MyLog writeLog..." << m_pStr;
    }

private:
    std::string *m_pStr;
};

class TestLog : public QObject
{
public:
    TestLog(QObject *parent = nullptr)
        : QObject(parent)
    {
        qDebug() << "TestLog ctor";
    }
    ~TestLog()
    {
        MyLog::getInstance()->writeLog();
        qDebug() << "TestLog dector";
    }
    static TestLog* getInstance(QObject *parent = 0)
    {
        static TestLog* testlog = new TestLog(parent);
        return testlog;
    }
};

class Test : public QObject
{
public:
    Test(QObject *parent = nullptr)
        : QObject(parent)
    {
        qDebug() << "Test ctor";
    }
    ~Test()
    {
        qDebug() << "Test dector";
    }
};

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    {
        Test t;
        MyLog::getInstance(&t);
        TestLog::getInstance(&t);
    }

    return a.exec();
}
```

3. 执行结果：
```
Test ctor
MyLog ctor
TestLog ctor
Test dector
MyLog dector
MyLog writeLog... 0x0
TestLog dector
```
