1. 结论：通过 engine.addImageProvider 添加的裸指针，在其所在类的析构函数中不要对其析构。因为在 addImageProvider() 的实现里面已经用了一个智能指针对其进行管理。如果在其所在类的析构函数中再次对其进行析构，则程序退出时将会崩溃。直接看代码。
```
// 报错信息如下：
The inferior stopped because it received a signal from the operating system.

Signal name :       SIGSEGV
Signal meaning :    Segmentation fault

// Debug 调试模式时，报错位置如下：
void QHashData::free_helper(void (*node_delete)(Node *))
{
    if (node_delete) {
        Node *this_e = reinterpret_cast<Node *>(this);
        Node **bucket = reinterpret_cast<Node **>(this->buckets);

        int n = numBuckets;
        while (n--) {
            Node *cur = *bucket++;
            while (cur != this_e) {
                Node *next = cur->next;
                node_delete(cur);                   // 指向这行代码！
                freeNode(cur);
                cur = next;
            }
        }
    }
    delete [] buckets;
    delete this;
}
```

2. 代码：
```cpp
// testclass.h
#ifndef TESTCLASS_H
#define TESTCLASS_H

#include <QObject>
#include <QDebug>
#include <QQuickImageProvider>

#define SINGLETON(x)     \
private: \
    x(x const&); \
    void operator=(x const&); \
public: \
static x* getInstance(QObject *parent = 0) \
{ \
   static bool first=true; \
   if ((parent == 0) && (first == true)) { qCritical("Incorrect Initialisation - no parent and first"); } \
   if ((parent != 0) && (first == false)) { qCritical("Incorrect Initialisation - parent and not first"); } \
   first = false; \
   static x *instance = new x(parent); \
   return instance; \
} \
private:

class ImageProvider : public QQuickImageProvider
{
public:
    explicit ImageProvider() : QQuickImageProvider(QQuickImageProvider::Image) {}
};

class TestClass : public QObject
{
    SINGLETON(TestClass)
public:
    explicit TestClass(QObject* parent = nullptr);
    ~TestClass();

    ImageProvider* m_pVal;
};

#endif // TESTCLASS_H


// testclass.cpp
#include "testclass.h"

TestClass::TestClass(QObject *parent)
    : QObject(parent)
    , m_pVal(new ImageProvider())
{
    qDebug() << "TestClass ctor";
}

TestClass::~TestClass()
{
    qDebug() << "TestClass dector";

    // 此处不能再次析构.
//    if(nullptr != m_pVal)
//    {
//        delete m_pVal;
//        m_pVal = nullptr;
//    }
}


// main.cpp
#include <QGuiApplication>
#include <QQmlApplicationEngine>
#include "testclass.h"

int main(int argc, char *argv[])
{
#if QT_VERSION < QT_VERSION_CHECK(6, 0, 0)
    QCoreApplication::setAttribute(Qt::AA_EnableHighDpiScaling);
#endif
    QGuiApplication app(argc, argv);

    QQmlApplicationEngine engine;
    const QUrl url(QStringLiteral("qrc:/main.qml"));
    QObject::connect(&engine, &QQmlApplicationEngine::objectCreated,
                     &app, [url](QObject *obj, const QUrl &objUrl) {
        if (!obj && url == objUrl)
            QCoreApplication::exit(-1);
    }, Qt::QueuedConnection);

    // 用 TestClass::getInstance() 不会崩溃是因为实现的单例模式是不会自动析构的，
    // 如果传入了 &engine，也就是指定了父类指针 QObject，此时就会被析构（因为 QObject 的析构机制）。
    // engine.addImageProvider(QLatin1String("drawImg"), TestClass::getInstance()->m_pVal);
    engine.addImageProvider(QLatin1String("drawImg"), TestClass::getInstance(&engine)->m_pVal);

    engine.load(url);

    return app.exec();
}
```

3. 相关源码：
```cpp
// 源码在：./Qt/5.15.2/Src/qtdeclarative/src/qml/qml/qqmlengine.cpp
// 注意 QSharedPointer<QQmlImageProviderBase> sp(provider); 这行代码！
void QQmlEngine::addImageProvider(const QString &providerId, QQmlImageProviderBase *provider)
{
    Q_D(QQmlEngine);
    QString providerIdLower = providerId.toLower();
    QSharedPointer<QQmlImageProviderBase> sp(provider);
    QMutexLocker locker(&d->mutex);
    d->imageProviders.insert(std::move(providerIdLower), std::move(sp));
}
QHash<QString,QSharedPointer<QQmlImageProviderBase> > imageProviders;
```
