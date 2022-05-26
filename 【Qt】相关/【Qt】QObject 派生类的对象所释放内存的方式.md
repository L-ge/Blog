###### 1、在 Qt 中，有时候 new 出的对象可以不用亲自去 delete —— QObject 及其派生类的对象，如果其 parent 非0，那么其 parent 析构时会析构该对象。
`在 Qt 中，每个 QObject 内部都有一个 list，用来保存所有的 children，当它自己析构时，它同时也会析构掉所有的 children。`

###### 2、下面是测试代码：

下面的测试程序在 ATestClass 构造函数中的 m_pTimer 变量 new 的时候有没有传入 this 指针是会不会内存泄漏的关键。如果没有传入 this 指针，则会内存泄漏。

根据第一点的描述，new 出的对象是 m_pTimer，其 paren t是 pTestClass，当 pTestClass 析构的时候，m_pTimer 会自动被析构。所以 ATestClass 无需声明和定义析构函数去析构 m_pTimer。

```
// 下面是 atestclass.h
#ifndef ATESTCLASS_H
#define ATESTCLASS_H

#include <QTimer>
#include <QObject>

class ATestClass : public QObject
{
    Q_OBJECT
public:
    ATestClass(QObject* parent = nullptr);
    
protected:
    void sltTimeOut();

private:
    QTimer* m_pTimer;
};

#endif // ATESTCLASS_H

// 下面是 atestclass.cpp
#include "atestclass.h"

ATestClass::ATestClass(QObject *parent)
    : QObject(parent)
{
    m_pTimer = new QTimer(this);    // 有没有this很重要
    m_pTimer->setInterval(1000);
    connect(m_pTimer, &QTimer::timeout, this, &ATestClass::sltTimeOut);
    m_pTimer->start();
}

void ATestClass::sltTimeOut()
{

}

// 下面是测试的 main.cpp
#include <QCoreApplication>
#include <QDebug>
#include <QThread>

#include "atestclass.h"

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    qDebug() << "begin";

    QThread::sleep(5);

    qDebug() << "begin--";

    for(int i=0; i<1000000; i++)
    {
        ATestClass* pTestClass = new ATestClass();
        delete pTestClass;
    }

    qDebug() << "end---";

    QThread::sleep(5);

    qDebug() << "end";

    return a.exec();
}
```

###### 3、相关Qt源码

可以在 QObject 的析构函数中看到 d->deleteChildren(); 这句代码。
```
QObject::~QObject()
{
    ...

    if (!d->children.isEmpty())
        d->deleteChildren();

    qt_removeObject(this);
    if (Q_UNLIKELY(qtHookData[QHooks::RemoveQObject]))
        reinterpret_cast<QHooks::RemoveQObjectCallback>(qtHookData[QHooks::RemoveQObject])(this);

    if (d->parent)        // remove it from parent object
        d->setParent_helper(0);
}

void QObjectPrivate::deleteChildren()
{
    Q_ASSERT_X(!isDeletingChildren, "QObjectPrivate::deleteChildren()", "isDeletingChildren already set, did this function recurse?");
    isDeletingChildren = true;
    // delete children objects
    // don't use qDeleteAll as the destructor of the child might
    // delete siblings
    for (int i = 0; i < children.count(); ++i) {
        currentChildBeingDeleted = children.at(i);
        children[i] = 0;
        delete currentChildBeingDeleted;
    }
    children.clear();
    currentChildBeingDeleted = 0;
    isDeletingChildren = false;
}
```
