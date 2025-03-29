##### 一、QA：

1. 指定了 QObject 对象作为父类对象，我还需要手动 delete 这个对象吗？
- 不需要。指定了 QObject 对象作为父类对象之后，在该父类对象析构的时候，该父类对象的父类 QObject 的析构函数会帮该父类对象 delete 掉所有的子类对象。

2. 指定了 QObject 对象作为父类对象，我手动 delete 了这个对象，会发生重复析构的问题吗？
- 不会。在该对象析构的时候，其父类 QObject 的析构函数会把该对象从其父类对象的子对象列表中删掉。

3. QObject 中的子对象列表是一个怎样的数据结构，是树吗？
- 不是。是 QList<QObject*>。

4. QObject::~QObject() 是线程安全的吗？
- 不是。比如对子对象列表 children 的操作没有加锁。

5. 指定 QObject 对象作为父类对象的子类对象们，析构顺序和构造顺序相反吗？
- 不是，先指定的，先被析构。因为指定 QObject 对象作为父类对象时，是 append 到 QList<QObject*> 里面的，而析构时是 for (int i = 0; i < children.count(); ++i) 的遍历顺序进行 delete。

---

##### 二、source code：

```cpp
// ./Qt/5.15.2/Src/qtbase/src/corelib/kernel/qobject.cpp
/*!
    \internal
 */
QObject::QObject(QObjectPrivate &dd, QObject *parent)
    : d_ptr(&dd)
{
    Q_ASSERT_X(this != parent, Q_FUNC_INFO, "Cannot parent a QObject to itself");

    Q_D(QObject);
    d_ptr->q_ptr = this;
    auto threadData = (parent && !parent->thread()) ? parent->d_func()->threadData.loadRelaxed() : QThreadData::current();
    threadData->ref();
    d->threadData.storeRelaxed(threadData);
    if (parent) {
        QT_TRY {
            if (!check_parent_thread(parent, parent ? parent->d_func()->threadData.loadRelaxed() : nullptr, threadData))
                parent = nullptr;
            if (d->isWidget) {
                if (parent) {
                    d->parent = parent;
                    d->parent->d_func()->children.append(this);
                }
                // no events sent here, this is done at the end of the QWidget constructor
            } else {
                setParent(parent);
            }
        } QT_CATCH(...) {
            threadData->deref();
            QT_RETHROW;
        }
    }
#if QT_VERSION < 0x60000
    qt_addObject(this);
#endif
    if (Q_UNLIKELY(qtHookData[QHooks::AddQObject]))
        reinterpret_cast<QHooks::AddQObjectCallback>(qtHookData[QHooks::AddQObject])(this);
    Q_TRACE(QObject_ctor, this);
}

/*!
    Destroys the object, deleting all its child objects.

    All signals to and from the object are automatically disconnected, and
    any pending posted events for the object are removed from the event
    queue. However, it is often safer to use deleteLater() rather than
    deleting a QObject subclass directly.

    \warning All child objects are deleted. If any of these objects
    are on the stack or global, sooner or later your program will
    crash. We do not recommend holding pointers to child objects from
    outside the parent. If you still do, the destroyed() signal gives
    you an opportunity to detect when an object is destroyed.

    \warning Deleting a QObject while pending events are waiting to
    be delivered can cause a crash. You must not delete the QObject
    directly if it exists in a different thread than the one currently
    executing. Use deleteLater() instead, which will cause the event
    loop to delete the object after all pending events have been
    delivered to it.

    \sa deleteLater()
*/

QObject::~QObject()
{
    Q_D(QObject);
    d->wasDeleted = true;
    d->blockSig = 0; // unblock signals so we always emit destroyed()

    QtSharedPointer::ExternalRefCountData *sharedRefcount = d->sharedRefcount.loadRelaxed();
    if (sharedRefcount) {
        if (sharedRefcount->strongref.loadRelaxed() > 0) {
            qWarning("QObject: shared QObject was deleted directly. The program is malformed and may crash.");
            // but continue deleting, it's too late to stop anyway
        }

        // indicate to all QWeakPointers that this QObject has now been deleted
        sharedRefcount->strongref.storeRelaxed(0);
        if (!sharedRefcount->weakref.deref())
            delete sharedRefcount;
    }

    if (!d->isWidget && d->isSignalConnected(0)) {
        emit destroyed(this);
    }

    if (d->declarativeData) {
        if (static_cast<QAbstractDeclarativeDataImpl*>(d->declarativeData)->ownedByQml1) {
            if (QAbstractDeclarativeData::destroyed_qml1)
                QAbstractDeclarativeData::destroyed_qml1(d->declarativeData, this);
        } else {
            if (QAbstractDeclarativeData::destroyed)
                QAbstractDeclarativeData::destroyed(d->declarativeData, this);
        }
    }

    QObjectPrivate::ConnectionData *cd = d->connections.loadRelaxed();
    if (cd) {
        if (cd->currentSender) {
            cd->currentSender->receiverDeleted();
            cd->currentSender = nullptr;
        }

        QBasicMutex *signalSlotMutex = signalSlotLock(this);
        QBasicMutexLocker locker(signalSlotMutex);

        // disconnect all receivers
        int receiverCount = cd->signalVectorCount();
        for (int signal = -1; signal < receiverCount; ++signal) {
            QObjectPrivate::ConnectionList &connectionList = cd->connectionsForSignal(signal);

            while (QObjectPrivate::Connection *c = connectionList.first.loadRelaxed()) {
                Q_ASSERT(c->receiver.loadAcquire());

                QBasicMutex *m = signalSlotLock(c->receiver.loadRelaxed());
                bool needToUnlock = QOrderedMutexLocker::relock(signalSlotMutex, m);
                if (c->receiver.loadAcquire()) {
                    cd->removeConnection(c);
                    Q_ASSERT(connectionList.first.loadRelaxed() != c);
                }
                if (needToUnlock)
                    m->unlock();
            }
        }

        /* Disconnect all senders:
         */
        while (QObjectPrivate::Connection *node = cd->senders) {
            Q_ASSERT(node->receiver.loadAcquire());
            QObject *sender = node->sender;
            // Send disconnectNotify before removing the connection from sender's connection list.
            // This ensures any eventual destructor of sender will block on getting receiver's lock
            // and not finish until we release it.
            sender->disconnectNotify(QMetaObjectPrivate::signal(sender->metaObject(), node->signal_index));
            QBasicMutex *m = signalSlotLock(sender);
            bool needToUnlock = QOrderedMutexLocker::relock(signalSlotMutex, m);
            //the node has maybe been removed while the mutex was unlocked in relock?
            if (node != cd->senders) {
                // We hold the wrong mutex
                Q_ASSERT(needToUnlock);
                m->unlock();
                continue;
            }

            QObjectPrivate::ConnectionData *senderData = sender->d_func()->connections.loadRelaxed();
            Q_ASSERT(senderData);

            QtPrivate::QSlotObjectBase *slotObj = nullptr;
            if (node->isSlotObject) {
                slotObj = node->slotObj;
                node->isSlotObject = false;
            }

            senderData->removeConnection(node);
            if (needToUnlock)
                m->unlock();

            if (slotObj) {
                locker.unlock();
                slotObj->destroyIfLastRef();
                locker.relock();
            }
        }

        // invalidate all connections on the object and make sure
        // activate() will skip them
        cd->currentConnectionId.storeRelaxed(0);
    }
    if (cd && !cd->ref.deref())
        delete cd;
    d->connections.storeRelaxed(nullptr);

    if (!d->children.isEmpty())
        d->deleteChildren();

#if QT_VERSION < 0x60000
    qt_removeObject(this);
#endif
    if (Q_UNLIKELY(qtHookData[QHooks::RemoveQObject]))
        reinterpret_cast<QHooks::RemoveQObjectCallback>(qtHookData[QHooks::RemoveQObject])(this);

    Q_TRACE(QObject_dtor, this);

    if (d->parent)        // remove it from parent object
        d->setParent_helper(nullptr);
}
```
- 关键语句：d->parent->d_func()->children.append(this);
- 关键语句：d->deleteChildren();
- 关键语句：d->setParent_helper(nullptr);

```cpp
// ./Qt/5.15.2/Src/qtbase/src/corelib/kernel/qobject.cpp
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
    currentChildBeingDeleted = nullptr;
    isDeletingChildren = false;
}
```
- 关键语句：delete currentChildBeingDeleted;

```cpp
// ./Qt/5.15.2/Src/qtbase/src/corelib/kernel/qobject.cpp
void QObjectPrivate::setParent_helper(QObject *o)
{
    Q_Q(QObject);
    Q_ASSERT_X(q != o, Q_FUNC_INFO, "Cannot parent a QObject to itself");
#ifdef QT_DEBUG
    const auto checkForParentChildLoops = qScopeGuard([&](){
        int depth = 0;
        auto p = parent;
        while (p) {
            if (++depth == CheckForParentChildLoopsWarnDepth) {
                qWarning("QObject %p (class: '%s', object name: '%s') may have a loop in its parent-child chain; "
                         "this is undefined behavior",
                         q, q->metaObject()->className(), qPrintable(q->objectName()));
            }
            p = p->parent();
        }
    });
#endif

    if (o == parent)
        return;

    if (parent) {
        QObjectPrivate *parentD = parent->d_func();
        if (parentD->isDeletingChildren && wasDeleted
            && parentD->currentChildBeingDeleted == q) {
            // don't do anything since QObjectPrivate::deleteChildren() already
            // cleared our entry in parentD->children.
        } else {
            const int index = parentD->children.indexOf(q);
            if (index < 0) {
                // we're probably recursing into setParent() from a ChildRemoved event, don't do anything
            } else if (parentD->isDeletingChildren) {
                parentD->children[index] = 0;
            } else {
                parentD->children.removeAt(index);
                if (sendChildEvents && parentD->receiveChildEvents) {
                    QChildEvent e(QEvent::ChildRemoved, q);
                    QCoreApplication::sendEvent(parent, &e);
                }
            }
        }
    }
    parent = o;
    if (parent) {
        // object hierarchies are constrained to a single thread
        if (threadData != parent->d_func()->threadData) {
            qWarning("QObject::setParent: Cannot set parent, new parent is in a different thread");
            parent = nullptr;
            return;
        }
        parent->d_func()->children.append(q);
        if(sendChildEvents && parent->d_func()->receiveChildEvents) {
            if (!isWidget) {
                QChildEvent e(QEvent::ChildAdded, q);
                QCoreApplication::sendEvent(parent, &e);
            }
        }
    }
    if (!wasDeleted && !isDeletingChildren && declarativeData && QAbstractDeclarativeData::parentChanged)
        QAbstractDeclarativeData::parentChanged(declarativeData, q, o);
}
```
- 关键语句：parentD->children.removeAt(index);

---

##### 三、demo code：
```cpp
// iner.h
#ifndef INER_H
#define INER_H

#include <QObject>

class Iner : public QObject
{
public:
    explicit Iner(QObject *parent = nullptr);
    ~Iner();
};

#endif // INER_H


// iner.cpp
#include "iner.h"
#include <QDebug>

Iner::Iner(QObject *parent)
    : QObject(parent)
{

}

Iner::~Iner()
{
    qDebug() << "Iner dector";
}


// outer.h
#ifndef OUTER_H
#define OUTER_H

#include <QObject>
#include "iner.h"

class Outer : public QObject
{
public:
    explicit Outer(QObject *parent = nullptr);
    ~Outer();

private:
    Iner* m_pIner;
};

#endif // OUTER_H


// outer.cpp
#include "outer.h"

Outer::Outer(QObject *parent)
    : QObject(parent)
{
    m_pIner = new Iner(this);
}

Outer::~Outer()
{
    // 由于 Iner 构造的时候指定了父对象，因此在 Outer 的析构函数中，是否 delete m_pIner 都无所谓。
    // 如果不 delete m_pIner，那么在 pOuter 对象析构的时候，QObject 的析构函数（先调用子类 Outer 析构函数，再调用 QObject 父类的析构函数）会帮你 delete 掉所有的子类对象。
    // 如果 delete m_pIner，那么在 m_pIner 对象析构的时候，QObject 的析构函数（先调用子类 Iner 析构函数，再调用 QObject 父类的析构函数）会把 m_pIner 对象从其父类对象 pOuter 的子对象列表中删掉。这样就不会造成重复析构的情况了。
    delete m_pIner;
    m_pIner = nullptr;
}


// main.cpp
#include <QCoreApplication>
#include "outer.h"

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    Outer* pOuter = new Outer();

    delete pOuter;

    return a.exec();
}
```
