##### 一、QA：

1. connect 做了什么事情？
- 通过 sender 和 signal 得到 signal_index。
- slot 被包装成 QtPrivate::QSlotObjectBase 对象 slotObj。
- 将 signal_index 和 slotObj 等信息包装成 QObjectPrivate::Connection 对象 c，然后把对象 c addConnection 到 sender 里面去。
- 包装成 QObjectPrivate::Connection 对象 c 并 addConnection 这个过程是线程安全的。
- 其中，包装的信息有：
    ```cpp
    c->sender = s;
    c->signal_index = signal_index;
    QThreadData *td = r->d_func()->threadData;
    td->ref();
    c->receiverThreadData.storeRelaxed(td);
    c->receiver.storeRelaxed(r);
    c->slotObj = slotObj;
    c->connectionType = type;
    c->isSlotObject = true;
    if (types) {
        c->argumentTypes.storeRelaxed(types);
        c->ownArgumentTypes = false;
    }
    ```

2. connect 保存的数据结构，在信号发出的时候，是怎么用上这个保存的数据结构的？
- 首先，编译出来的 moc_xxx.cpp 里面有个 staticMetaObject，通过它求出发送的信号的下标 signal_index。
- 然后通过 signal_index 得到该信号绑定的 ConnectionList。
- 然后遍历得到的 ConnectionList，从中取出每一个 Connection 进行处理。
- 通过判断每一个 Connection 的 connectionType 和 receiverThreadData 的 threadId 分别进行相应的处理。

3. connect 的第 5 个参数分别有什么？分别作用是什么？使用场景是怎样的？
- 有如下类型：
    ```cpp
    enum ConnectionType {
        AutoConnection,
        DirectConnection,
        QueuedConnection,
        BlockingQueuedConnection,
        UniqueConnection =  0x80
    };
    ```
- AutoConnection：默认值。使用这个值时，则连接类型会在信号发送的时候确定（根据是否发送者和接收者是否在同一个线程决定是 DirectConnection 还是 QueuedConnection）。
- DirectConnection：直连的形式。槽函数会在信号发送的时候直接被调用。这和直接调槽函数其实没有很大区别。
- QueuedConnection：队列绑定的形式。信号发出后，信号会被放到一个消息队列中，等到接收者所属线程的事件循环取得控制权时取得该信号后，执行和信号关联的槽函数，这种方式既可以在同一线程内传递消息，也可以跨线程传递消息。槽函数在接收者所依附线程执行。
- BlockingQueuedConnection：不允许在信号的发送者和接收者在同一个线程中，否则就是 Dead lock。槽函数的调用时机与 QueuedConnection 一样，不过发送完信号后，发送者所在线程会阻塞，直到槽函数运行完。在多线程间需要同步的场合可能需要用到。
- UniqueConnection：它可以通过按位或的形式与以上四个类型结合在一起使用。如果这个类型被设置，当某个信号和槽已经连接时，再进行重复的连接就会失败，即避免重复连接。

4. 队列绑定是怎么实现的？信号槽跨线程是怎么实现的？它怎么知道是否在同一个线程？
- 改变了 threadData 这个结构体的数据，例如 threadid。
- 在 QObject.cpp 里面有个函数叫 doActivate，该函数通过在 connect 的时候包装的 QObjectPrivate::Connection 信息来判断是否在同一个线程。
- Connection 里面的 connectionType 如果是 Qt::QueuedConnection 或者是 Qt::AutoConnection 且不在同一个线程，那么会当前信号调用封装成 QMetaCallEvent 对象，并 postEvent 到接受者的事件队列去。
- 不在同一个线程是通过 Connection 的 receiverThreadData 的 threadId （信号接收者） 和 QThread::currentThreadId()（发送信号者）对比得知。

---

##### 二、source code：

```cpp
// /Qt/5.15.2/Src/qtbase/src/corelib/kernel/qobject.h
    //Connect a signal to a pointer to qobject member function
    template <typename Func1, typename Func2>
    static inline QMetaObject::Connection connect(const typename QtPrivate::FunctionPointer<Func1>::Object *sender, Func1 signal,
                                     const typename QtPrivate::FunctionPointer<Func2>::Object *receiver, Func2 slot,
                                     Qt::ConnectionType type = Qt::AutoConnection)
    {
        typedef QtPrivate::FunctionPointer<Func1> SignalType;
        typedef QtPrivate::FunctionPointer<Func2> SlotType;

        Q_STATIC_ASSERT_X(QtPrivate::HasQ_OBJECT_Macro<typename SignalType::Object>::Value,
                          "No Q_OBJECT in the class with the signal");

        //compilation error if the arguments does not match.
        Q_STATIC_ASSERT_X(int(SignalType::ArgumentCount) >= int(SlotType::ArgumentCount),
                          "The slot requires more arguments than the signal provides.");
        Q_STATIC_ASSERT_X((QtPrivate::CheckCompatibleArguments<typename SignalType::Arguments, typename SlotType::Arguments>::value),
                          "Signal and slot arguments are not compatible.");
        Q_STATIC_ASSERT_X((QtPrivate::AreArgumentsCompatible<typename SlotType::ReturnType, typename SignalType::ReturnType>::value),
                          "Return type of the slot is not compatible with the return type of the signal.");

        const int *types = nullptr;
        if (type == Qt::QueuedConnection || type == Qt::BlockingQueuedConnection)
            types = QtPrivate::ConnectionTypes<typename SignalType::Arguments>::types();

        return connectImpl(sender, reinterpret_cast<void **>(&signal),
                           receiver, reinterpret_cast<void **>(&slot),
                           new QtPrivate::QSlotObject<Func2, typename QtPrivate::List_Left<typename SignalType::Arguments, SlotType::ArgumentCount>::Value,
                                           typename SignalType::ReturnType>(slot),
                            type, types, &SignalType::Object::staticMetaObject);
    }
```
- 关键语句：new QtPrivate::QSlotObject<Func2, typename QtPrivate::List_Left<typename SignalType::Arguments, SlotType::ArgumentCount>::Value,
                                           typename SignalType::ReturnType>(slot),

```cpp
/*!
    \internal

    Implementation of the template version of connect

    \a sender is the sender object
    \a signal is a pointer to a pointer to a member signal of the sender
    \a receiver is the receiver object, may not be \nullptr, will be equal to sender when
                connecting to a static function or a functor
    \a slot a pointer only used when using Qt::UniqueConnection
    \a type the Qt::ConnectionType passed as argument to connect
    \a types an array of integer with the metatype id of the parameter of the signal
             to be used with queued connection
             must stay valid at least for the whole time of the connection, this function
             do not take ownership. typically static data.
             If \nullptr, then the types will be computed when the signal is emit in a queued
             connection from the types from the signature.
    \a senderMetaObject is the metaobject used to lookup the signal, the signal must be in
                        this metaobject
 */
QMetaObject::Connection QObject::connectImpl(const QObject *sender, void **signal,
                                             const QObject *receiver, void **slot,
                                             QtPrivate::QSlotObjectBase *slotObj, Qt::ConnectionType type,
                                             const int *types, const QMetaObject *senderMetaObject)
{
    if (!signal) {
        qWarning("QObject::connect: invalid nullptr parameter");
        if (slotObj)
            slotObj->destroyIfLastRef();
        return QMetaObject::Connection();
    }

    int signal_index = -1;
    void *args[] = { &signal_index, signal };
    for (; senderMetaObject && signal_index < 0; senderMetaObject = senderMetaObject->superClass()) {
        senderMetaObject->static_metacall(QMetaObject::IndexOfMethod, 0, args);
        if (signal_index >= 0 && signal_index < QMetaObjectPrivate::get(senderMetaObject)->signalCount)
            break;
    }
    if (!senderMetaObject) {
        qWarning("QObject::connect: signal not found in %s", sender->metaObject()->className());
        slotObj->destroyIfLastRef();
        return QMetaObject::Connection(nullptr);
    }
    signal_index += QMetaObjectPrivate::signalOffset(senderMetaObject);
    return QObjectPrivate::connectImpl(sender, signal_index, receiver, slot, slotObj, type, types, senderMetaObject);
}

/*!
    \internal

    Internal version of connect used by the template version of QObject::connect (called via connectImpl) and
    also used by the QObjectPrivate::connect version used by QML. The signal_index is expected to be relative
    to the number of signals.
 */
QMetaObject::Connection QObjectPrivate::connectImpl(const QObject *sender, int signal_index,
                                             const QObject *receiver, void **slot,
                                             QtPrivate::QSlotObjectBase *slotObj, Qt::ConnectionType type,
                                             const int *types, const QMetaObject *senderMetaObject)
{
    if (!sender || !receiver || !slotObj || !senderMetaObject) {
        const char *senderString = sender ? sender->metaObject()->className()
                                          : senderMetaObject ? senderMetaObject->className()
                                          : "Unknown";
        const char *receiverString = receiver ? receiver->metaObject()->className()
                                              : "Unknown";
        qWarning("QObject::connect(%s, %s): invalid nullptr parameter", senderString, receiverString);
        if (slotObj)
            slotObj->destroyIfLastRef();
        return QMetaObject::Connection();
    }

    QObject *s = const_cast<QObject *>(sender);
    QObject *r = const_cast<QObject *>(receiver);

    QOrderedMutexLocker locker(signalSlotLock(sender),
                               signalSlotLock(receiver));

    if (type & Qt::UniqueConnection && slot && QObjectPrivate::get(s)->connections.loadRelaxed()) {
        QObjectPrivate::ConnectionData *connections = QObjectPrivate::get(s)->connections.loadRelaxed();
        if (connections->signalVectorCount() > signal_index) {
            const QObjectPrivate::Connection *c2 = connections->signalVector.loadRelaxed()->at(signal_index).first.loadRelaxed();

            while (c2) {
                if (c2->receiver.loadRelaxed() == receiver && c2->isSlotObject && c2->slotObj->compare(slot)) {
                    slotObj->destroyIfLastRef();
                    return QMetaObject::Connection();
                }
                c2 = c2->nextConnectionList.loadRelaxed();
            }
        }
        type = static_cast<Qt::ConnectionType>(type ^ Qt::UniqueConnection);
    }

    std::unique_ptr<QObjectPrivate::Connection> c{new QObjectPrivate::Connection};
    c->sender = s;
    c->signal_index = signal_index;
    QThreadData *td = r->d_func()->threadData;
    td->ref();
    c->receiverThreadData.storeRelaxed(td);
    c->receiver.storeRelaxed(r);
    c->slotObj = slotObj;
    c->connectionType = type;
    c->isSlotObject = true;
    if (types) {
        c->argumentTypes.storeRelaxed(types);
        c->ownArgumentTypes = false;
    }

    QObjectPrivate::get(s)->addConnection(signal_index, c.get());
    QMetaObject::Connection ret(c.release());
    locker.unlock();

    QMetaMethod method = QMetaObjectPrivate::signal(senderMetaObject, signal_index);
    Q_ASSERT(method.isValid());
    s->connectNotify(method);

    return ret;
}
```
- 关键语句：signal_index += QMetaObjectPrivate::signalOffset(senderMetaObject);
- 关键语句：QObjectPrivate::get(s)->addConnection(signal_index, c.get());

---

##### 三、信号槽的过程（单线程）：

1. 追溯源码发现，起作用的原来是 c->slotObj。回到 connect 的时候看 slotObj 的诞生，即 `new QtPrivate::QSlotObject<Func2, typename QtPrivate::List_Left<typename SignalType::Arguments, SlotType::ArgumentCount>::Value, typename SignalType::ReturnType>(slot) `
    - 可以看到 Func2 也就是类 QSlotObject 的 Func，而 slot 传入 QSlotObject 的构造函数并赋值为 function。

2. 断点调试追踪的函数栈：
```cpp
emit sigMySignal();
void TestClassA::sigMySignal()
QMetaObject::activate(this, &staticMetaObject, 0, nullptr);
doActivate<false>(sender, signal_index, argv);
template <bool callbacks_enabled>
void doActivate(QObject *sender, int signal_index, void **argv)
obj->call(receiver, argv);
inline void call(QObject *r, void **a)  { m_impl(Call,    this, r, a, nullptr); }
FuncType::template call<Args, R>(static_cast<QSlotObject*>(this_)->function, static_cast<typename FuncType::Object *>(r), a);
FunctorCall<typename Indexes<ArgumentCount>::Value, SignalArgs, R, Function>::call(f, o, arg);
(o->*f)((*reinterpret_cast<typename RemoveRef<SignalArgs>::Type *>(arg[II+1]))...), ApplyReturnValue<R>(arg[0]);
void TestClassA::sltMySlot()
```

3. 测试代码：
```cpp
// testclassa.h
#ifndef TESTCLASSA_H
#define TESTCLASSA_H

#include <QObject>

class TestClassA : public QObject
{
    Q_OBJECT
public:
    explicit TestClassA(QObject* parent = nullptr);
    void testFunc();

signals:
    void sigMySignal();

public slots:
    void sltMySlot();
};

#endif // TESTCLASSA_H

// testclassa.cpp
#include "testclassa.h"
#include <QDebug>

TestClassA::TestClassA(QObject *parent)
    : QObject(parent)
{
    connect(this, &TestClassA::sigMySignal, this, &TestClassA::sltMySlot);
}

void TestClassA::testFunc()
{
    emit sigMySignal();
}

void TestClassA::sltMySlot()
{
    qDebug() << "slot call...";
}

// main.cpp
#include <QCoreApplication>
#include "testclassa.h"

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    TestClassA t;
    t.testFunc();

    return a.exec();
}

// moc_testclassa.cpp
/****************************************************************************
** Meta object code from reading C++ file 'testclassa.h'
**
** Created by: The Qt Meta Object Compiler version 67 (Qt 5.15.2)
**
** WARNING! All changes made in this file will be lost!
*****************************************************************************/

#include <memory>
#include "../SignalAndSlot/testclassa.h"
#include <QtCore/qbytearray.h>
#include <QtCore/qmetatype.h>
#if !defined(Q_MOC_OUTPUT_REVISION)
#error "The header file 'testclassa.h' doesn't include <QObject>."
#elif Q_MOC_OUTPUT_REVISION != 67
#error "This file was generated using the moc from 5.15.2. It"
#error "cannot be used with the include files from this version of Qt."
#error "(The moc has changed too much.)"
#endif

QT_BEGIN_MOC_NAMESPACE
QT_WARNING_PUSH
QT_WARNING_DISABLE_DEPRECATED
struct qt_meta_stringdata_TestClassA_t {
    QByteArrayData data[4];
    char stringdata0[34];
};
#define QT_MOC_LITERAL(idx, ofs, len) \
    Q_STATIC_BYTE_ARRAY_DATA_HEADER_INITIALIZER_WITH_OFFSET(len, \
    qptrdiff(offsetof(qt_meta_stringdata_TestClassA_t, stringdata0) + ofs \
        - idx * sizeof(QByteArrayData)) \
    )
static const qt_meta_stringdata_TestClassA_t qt_meta_stringdata_TestClassA = {
    {
QT_MOC_LITERAL(0, 0, 10), // "TestClassA"
QT_MOC_LITERAL(1, 11, 11), // "sigMySignal"
QT_MOC_LITERAL(2, 23, 0), // ""
QT_MOC_LITERAL(3, 24, 9) // "sltMySlot"

    },
    "TestClassA\0sigMySignal\0\0sltMySlot"
};
#undef QT_MOC_LITERAL

static const uint qt_meta_data_TestClassA[] = {

 // content:
       8,       // revision
       0,       // classname
       0,    0, // classinfo
       2,   14, // methods
       0,    0, // properties
       0,    0, // enums/sets
       0,    0, // constructors
       0,       // flags
       1,       // signalCount

 // signals: name, argc, parameters, tag, flags
       1,    0,   24,    2, 0x06 /* Public */,

 // slots: name, argc, parameters, tag, flags
       3,    0,   25,    2, 0x0a /* Public */,

 // signals: parameters
    QMetaType::Void,

 // slots: parameters
    QMetaType::Void,

       0        // eod
};

void TestClassA::qt_static_metacall(QObject *_o, QMetaObject::Call _c, int _id, void **_a)
{
    if (_c == QMetaObject::InvokeMetaMethod) {
        auto *_t = static_cast<TestClassA *>(_o);
        Q_UNUSED(_t)
        switch (_id) {
        case 0: _t->sigMySignal(); break;
        case 1: _t->sltMySlot(); break;
        default: ;
        }
    } else if (_c == QMetaObject::IndexOfMethod) {
        int *result = reinterpret_cast<int *>(_a[0]);
        {
            using _t = void (TestClassA::*)();
            if (*reinterpret_cast<_t *>(_a[1]) == static_cast<_t>(&TestClassA::sigMySignal)) {
                *result = 0;
                return;
            }
        }
    }
    Q_UNUSED(_a);
}

QT_INIT_METAOBJECT const QMetaObject TestClassA::staticMetaObject = { {
    QMetaObject::SuperData::link<QObject::staticMetaObject>(),
    qt_meta_stringdata_TestClassA.data,
    qt_meta_data_TestClassA,
    qt_static_metacall,
    nullptr,
    nullptr
} };


const QMetaObject *TestClassA::metaObject() const
{
    return QObject::d_ptr->metaObject ? QObject::d_ptr->dynamicMetaObject() : &staticMetaObject;
}

void *TestClassA::qt_metacast(const char *_clname)
{
    if (!_clname) return nullptr;
    if (!strcmp(_clname, qt_meta_stringdata_TestClassA.stringdata0))
        return static_cast<void*>(this);
    return QObject::qt_metacast(_clname);
}

int TestClassA::qt_metacall(QMetaObject::Call _c, int _id, void **_a)
{
    _id = QObject::qt_metacall(_c, _id, _a);
    if (_id < 0)
        return _id;
    if (_c == QMetaObject::InvokeMetaMethod) {
        if (_id < 2)
            qt_static_metacall(this, _c, _id, _a);
        _id -= 2;
    } else if (_c == QMetaObject::RegisterMethodArgumentMetaType) {
        if (_id < 2)
            *reinterpret_cast<int*>(_a[0]) = -1;
        _id -= 2;
    }
    return _id;
}

// SIGNAL 0
void TestClassA::sigMySignal()
{
    QMetaObject::activate(this, &staticMetaObject, 0, nullptr);
}
QT_WARNING_POP
QT_END_MOC_NAMESPACE
```

4. 相关源码：
```cpp
// ./Qt/5.15.2/Src/qtbase/src/corelib/kernel/qobject.cpp
void QMetaObject::activate(QObject *sender, const QMetaObject *m, int local_signal_index,
                           void **argv)
{
    int signal_index = local_signal_index + QMetaObjectPrivate::signalOffset(m);

    if (Q_UNLIKELY(qt_signal_spy_callback_set.loadRelaxed()))
        doActivate<true>(sender, signal_index, argv);
    else
        doActivate<false>(sender, signal_index, argv);
}
```

```cpp
// ./Qt/5.15.2/Src/qtbase/src/corelib/kernel/qobject.cpp
template <bool callbacks_enabled>
void doActivate(QObject *sender, int signal_index, void **argv)
{
    QObjectPrivate *sp = QObjectPrivate::get(sender);

    if (sp->blockSig)
        return;

    Q_TRACE_SCOPE(QMetaObject_activate, sender, signal_index);

    if (sp->isDeclarativeSignalConnected(signal_index)
            && QAbstractDeclarativeData::signalEmitted) {
        Q_TRACE_SCOPE(QMetaObject_activate_declarative_signal, sender, signal_index);
        QAbstractDeclarativeData::signalEmitted(sp->declarativeData, sender,
                                                signal_index, argv);
    }

    const QSignalSpyCallbackSet *signal_spy_set = callbacks_enabled ? qt_signal_spy_callback_set.loadAcquire() : nullptr;

    void *empty_argv[] = { nullptr };
    if (!argv)
        argv = empty_argv;

    if (!sp->maybeSignalConnected(signal_index)) {
        // The possible declarative connection is done, and nothing else is connected
        if (callbacks_enabled && signal_spy_set->signal_begin_callback != nullptr)
            signal_spy_set->signal_begin_callback(sender, signal_index, argv);
        if (callbacks_enabled && signal_spy_set->signal_end_callback != nullptr)
            signal_spy_set->signal_end_callback(sender, signal_index);
        return;
    }

    if (callbacks_enabled && signal_spy_set->signal_begin_callback != nullptr)
        signal_spy_set->signal_begin_callback(sender, signal_index, argv);

    bool senderDeleted = false;
    {
    Q_ASSERT(sp->connections.loadAcquire());
    QObjectPrivate::ConnectionDataPointer connections(sp->connections.loadRelaxed());
    QObjectPrivate::SignalVector *signalVector = connections->signalVector.loadRelaxed();

    const QObjectPrivate::ConnectionList *list;
    if (signal_index < signalVector->count())
        list = &signalVector->at(signal_index);
    else
        list = &signalVector->at(-1);

    Qt::HANDLE currentThreadId = QThread::currentThreadId();
    bool inSenderThread = currentThreadId == QObjectPrivate::get(sender)->threadData.loadRelaxed()->threadId.loadRelaxed();

    // We need to check against the highest connection id to ensure that signals added
    // during the signal emission are not emitted in this emission.
    uint highestConnectionId = connections->currentConnectionId.loadRelaxed();
    do {
        QObjectPrivate::Connection *c = list->first.loadRelaxed();
        if (!c)
            continue;

        do {
            QObject * const receiver = c->receiver.loadRelaxed();
            if (!receiver)
                continue;

            QThreadData *td = c->receiverThreadData.loadRelaxed();
            if (!td)
                continue;

            bool receiverInSameThread;
            if (inSenderThread) {
                receiverInSameThread = currentThreadId == td->threadId.loadRelaxed();
            } else {
                // need to lock before reading the threadId, because moveToThread() could interfere
                QMutexLocker lock(signalSlotLock(receiver));
                receiverInSameThread = currentThreadId == td->threadId.loadRelaxed();
            }


            // determine if this connection should be sent immediately or
            // put into the event queue
            if ((c->connectionType == Qt::AutoConnection && !receiverInSameThread)
                || (c->connectionType == Qt::QueuedConnection)) {
                queued_activate(sender, signal_index, c, argv);
                continue;
#if QT_CONFIG(thread)
            } else if (c->connectionType == Qt::BlockingQueuedConnection) {
                if (receiverInSameThread) {
                    qWarning("Qt: Dead lock detected while activating a BlockingQueuedConnection: "
                    "Sender is %s(%p), receiver is %s(%p)",
                    sender->metaObject()->className(), sender,
                    receiver->metaObject()->className(), receiver);
                }
                QSemaphore semaphore;
                {
                    QBasicMutexLocker locker(signalSlotLock(sender));
                    if (!c->receiver.loadAcquire())
                        continue;
                    QMetaCallEvent *ev = c->isSlotObject ?
                        new QMetaCallEvent(c->slotObj, sender, signal_index, argv, &semaphore) :
                        new QMetaCallEvent(c->method_offset, c->method_relative, c->callFunction,
                                           sender, signal_index, argv, &semaphore);
                    QCoreApplication::postEvent(receiver, ev);
                }
                semaphore.acquire();
                continue;
#endif
            }

            QObjectPrivate::Sender senderData(receiverInSameThread ? receiver : nullptr, sender, signal_index);

            if (c->isSlotObject) {
                c->slotObj->ref();

                struct Deleter {
                    void operator()(QtPrivate::QSlotObjectBase *slot) const {
                        if (slot) slot->destroyIfLastRef();
                    }
                };
                const std::unique_ptr<QtPrivate::QSlotObjectBase, Deleter> obj{c->slotObj};

                {
                    Q_TRACE_SCOPE(QMetaObject_activate_slot_functor, obj.get());
                    obj->call(receiver, argv);
                }
            } else if (c->callFunction && c->method_offset <= receiver->metaObject()->methodOffset()) {
                //we compare the vtable to make sure we are not in the destructor of the object.
                const int method_relative = c->method_relative;
                const auto callFunction = c->callFunction;
                const int methodIndex = (Q_HAS_TRACEPOINTS || callbacks_enabled) ? c->method() : 0;
                if (callbacks_enabled && signal_spy_set->slot_begin_callback != nullptr)
                    signal_spy_set->slot_begin_callback(receiver, methodIndex, argv);

                {
                    Q_TRACE_SCOPE(QMetaObject_activate_slot, receiver, methodIndex);
                    callFunction(receiver, QMetaObject::InvokeMetaMethod, method_relative, argv);
                }

                if (callbacks_enabled && signal_spy_set->slot_end_callback != nullptr)
                    signal_spy_set->slot_end_callback(receiver, methodIndex);
            } else {
                const int method = c->method_relative + c->method_offset;

                if (callbacks_enabled && signal_spy_set->slot_begin_callback != nullptr) {
                    signal_spy_set->slot_begin_callback(receiver, method, argv);
                }

                {
                    Q_TRACE_SCOPE(QMetaObject_activate_slot, receiver, method);
                    QMetaObject::metacall(receiver, QMetaObject::InvokeMetaMethod, method, argv);
                }

                if (callbacks_enabled && signal_spy_set->slot_end_callback != nullptr)
                    signal_spy_set->slot_end_callback(receiver, method);
            }
        } while ((c = c->nextConnectionList.loadRelaxed()) != nullptr && c->id <= highestConnectionId);

    } while (list != &signalVector->at(-1) &&
        //start over for all signals;
        ((list = &signalVector->at(-1)), true));

        if (connections->currentConnectionId.loadRelaxed() == 0)
            senderDeleted = true;
    }
    if (!senderDeleted) {
        sp->connections.loadRelaxed()->cleanOrphanedConnections(sender);

        if (callbacks_enabled && signal_spy_set->signal_end_callback != nullptr)
            signal_spy_set->signal_end_callback(sender, signal_index);
    }
}
```

```cpp
// ./Qt/5.15.2/Src/qtbase/src/corelib/kernel/qobjectdefs_impl.h
    // internal base class (interface) containing functions required to call a slot managed by a pointer to function.
    class QSlotObjectBase {
        QAtomicInt m_ref;
        // don't use virtual functions here; we don't want the
        // compiler to create tons of per-polymorphic-class stuff that
        // we'll never need. We just use one function pointer.
        typedef void (*ImplFn)(int which, QSlotObjectBase* this_, QObject *receiver, void **args, bool *ret);
        const ImplFn m_impl;
    protected:
        enum Operation {
            Destroy,
            Call,
            Compare,

            NumOperations
        };
    public:
        explicit QSlotObjectBase(ImplFn fn) : m_ref(1), m_impl(fn) {}

        inline int ref() noexcept { return m_ref.ref(); }
        inline void destroyIfLastRef() noexcept
        { if (!m_ref.deref()) m_impl(Destroy, this, nullptr, nullptr, nullptr); }

        inline bool compare(void **a) { bool ret = false; m_impl(Compare, this, nullptr, a, &ret); return ret; }
        inline void call(QObject *r, void **a)  { m_impl(Call,    this, r, a, nullptr); }
    protected:
        ~QSlotObjectBase() {}
    private:
        Q_DISABLE_COPY_MOVE(QSlotObjectBase)
    };
```

```cpp
// ./Qt/5.15.2/gcc_64/include/QtCore/qobjectdefs_impl.h
    // implementation of QSlotObjectBase for which the slot is a pointer to member function of a QObject
    // Args and R are the List of arguments and the return type of the signal to which the slot is connected.
    template<typename Func, typename Args, typename R> class QSlotObject : public QSlotObjectBase
    {
        typedef QtPrivate::FunctionPointer<Func> FuncType;
        Func function;
        static void impl(int which, QSlotObjectBase *this_, QObject *r, void **a, bool *ret)
        {
            switch (which) {
            case Destroy:
                delete static_cast<QSlotObject*>(this_);
                break;
            case Call:
                FuncType::template call<Args, R>(static_cast<QSlotObject*>(this_)->function, static_cast<typename FuncType::Object *>(r), a);
                break;
            case Compare:
                *ret = *reinterpret_cast<Func *>(a) == static_cast<QSlotObject*>(this_)->function;
                break;
            case NumOperations: ;
            }
        }
    public:
        explicit QSlotObject(Func f) : QSlotObjectBase(&impl), function(f) {}
    };
```

```cpp
// ./Qt/5.15.2/gcc_64/include/QtCore/qobjectdefs_impl.h
template<class Obj, typename Ret, typename... Args> struct FunctionPointer<Ret (Obj::*) (Args...)>
    {
        typedef Obj Object;
        typedef List<Args...>  Arguments;
        typedef Ret ReturnType;
        typedef Ret (Obj::*Function) (Args...);
        enum {ArgumentCount = sizeof...(Args), IsPointerToMemberFunction = true};
        template <typename SignalArgs, typename R>
        static void call(Function f, Obj *o, void **arg) {
            FunctorCall<typename Indexes<ArgumentCount>::Value, SignalArgs, R, Function>::call(f, o, arg);
        }
    };
```

```cpp
// ./Qt/5.15.2/gcc_64/include/QtCore/qobjectdefs_impl.h
    template <int... II, typename... SignalArgs, typename R, typename... SlotArgs, typename SlotRet, class Obj>
    struct FunctorCall<IndexesList<II...>, List<SignalArgs...>, R, SlotRet (Obj::*)(SlotArgs...)> {
        static void call(SlotRet (Obj::*f)(SlotArgs...), Obj *o, void **arg) {
            (o->*f)((*reinterpret_cast<typename RemoveRef<SignalArgs>::Type *>(arg[II+1]))...), ApplyReturnValue<R>(arg[0]);
        }
    };
```

---

##### 四、信号槽的过程（多线程）：

1. 断点调试追踪信号发送的函数栈：
```cpp
emit sigMySignal2();
void TestClassA::sigMySignal2()
QMetaObject::activate(this, &staticMetaObject, 1, nullptr);
doActivate<false>(sender, signal_index, argv);
queued_activate(sender, signal_index, c, argv);
QCoreApplication::postEvent(c->receiver.loadRelaxed(), ev);
data->postEventList.addEvent(QPostEvent(receiver, event, priority));
void addEvent(const QPostEvent &ev)
append(ev);
```

2. 断点调试追踪槽函数运行的函数栈：
```cpp
int QThread::exec()	int returnCode = eventLoop.exec();
int QEventLoop::exec(ProcessEventsFlags flags)	processEvents(flags | WaitForMoreEvents | EventLoopExec);
bool QEventDispatcherGlib::processEvents(QEventLoop::ProcessEventsFlags flags)	bool result = g_main_context_iteration(d->mainContext, canWait);
static gboolean postEventSourceDispatch(GSource *s, GSourceFunc, gpointer)	QCoreApplication::sendPostedEvents();
void QCoreApplication::sendPostedEvents(QObject *receiver, int event_type)	QCoreApplicationPrivate::sendPostedEvents(receiver, event_type, data);
void QCoreApplicationPrivate::sendPostedEvents(QObject *receiver, int event_type, QThreadData *data)	QCoreApplication::sendEvent(r, e);
bool QCoreApplication::sendEvent(QObject *receiver, QEvent *event)	return notifyInternal2(receiver, event);
bool QCoreApplication::notifyInternal2(QObject *receiver, QEvent *event)	return self->notify(receiver, event);
bool QCoreApplication::notify(QObject *receiver, QEvent *event)	return doNotify(receiver, event);
static bool doNotify(QObject *receiver, QEvent *event)	return receiver->isWidgetType() ? false : QCoreApplicationPrivate::notify_helper(receiver, event);
bool QObject::event(QEvent *e)	mce->placeMetaCall(this);
template<typename Func, typename Args, typename R> class QSlotObject : public QSlotObjectBase	FuncType::template call<Args, R>(static_cast<QSlotObject*>(this_)->function, static_cast<typename FuncType::Object *>(r), a);
template<class Obj, typename Ret, typename... Args> struct FunctionPointer<Ret (Obj::*) (Args...)>	FunctorCall<typename Indexes<ArgumentCount>::Value, SignalArgs, R, Function>::call(f, o, arg);
struct FunctorCall<IndexesList<II...>, List<SignalArgs...>, R, SlotRet (Obj::*)(SlotArgs...)>	(o->*f)((*reinterpret_cast<typename RemoveRef<SignalArgs>::Type *>(arg[II+1]))...), ApplyReturnValue<R>(arg[0]);
void TestClassThread::sltMySlotThread()
```
