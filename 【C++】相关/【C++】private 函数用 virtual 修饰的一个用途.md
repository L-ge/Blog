private 函数用 virtual 修饰时，可以用于“不想对外暴露 virtual 函数，而提供一个 public 的非虚函数给外界访问”的情形。具体看如下代码：
```
#include <QCoreApplication>
#include <QDebug>

class Father
{
public:
    void func()
    {
        vfunc();
    }

private:
    virtual void vfunc()
    {
        qDebug() << "I am Father";
    }
};

class Son : public Father
{
private:
    void vfunc() override
    {
        qDebug() << "I am Son";
    }
};

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    Father* pFather = new Father();
    pFather->func();            // "I am Father"

    Father* pSon = new Son();
    pSon->func();               // "I am Son"


    return a.exec();
}
```