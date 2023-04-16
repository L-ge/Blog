##### 一、QA：

1. deleteLater 是线程安全的吗？
- 线程安全的。

2. deleteLater 是怎么实现的？
- 它并不马上 delete 掉该对象，而是会包装成一个延迟删除事件 QDeferredDeleteEvent，然后 post 到事件循环里面去。
- 最后在 Qt 的事件循环中，取出该事件，然后 qDeleteInEventHandler(this)，从而实现最后的 delete 工作。

3. 为什么需要 deleteLater？
- 在 QObject 的析构函数 ~QObject() 的 cpp 实现上面，Qt 有句注释“Use deleteLater() instead, which will cause the event loop to delete the object after all pending events have been delivered to it.”，百度翻译为“改为使用deleteLater()，这将导致事件循环在所有挂起的事件都传递到对象之后删除该对象。”。
- 因此，如果使用传统的马上 delete 该对象，那么该对象在事件队列的那些事件将无法正常传递和处理。

---

##### 二、source code：

```cpp
// ./Qt/5.15.2/Src/qtbase/src/corelib/kernel/qobject.cpp
/*!
    \threadsafe

    Schedules this object for deletion.

    The object will be deleted when control returns to the event
    loop. If the event loop is not running when this function is
    called (e.g. deleteLater() is called on an object before
    QCoreApplication::exec()), the object will be deleted once the
    event loop is started. If deleteLater() is called after the main event loop
    has stopped, the object will not be deleted.
    Since Qt 4.8, if deleteLater() is called on an object that lives in a
    thread with no running event loop, the object will be destroyed when the
    thread finishes.

    Note that entering and leaving a new event loop (e.g., by opening a modal
    dialog) will \e not perform the deferred deletion; for the object to be
    deleted, the control must return to the event loop from which deleteLater()
    was called. This does not apply to objects deleted while a previous, nested
    event loop was still running: the Qt event loop will delete those objects
    as soon as the new nested event loop starts.

    \b{Note:} It is safe to call this function more than once; when the
    first deferred deletion event is delivered, any pending events for the
    object are removed from the event queue.

    \sa destroyed(), QPointer
*/
void QObject::deleteLater()
{
    QCoreApplication::postEvent(this, new QDeferredDeleteEvent());
}
```

```cpp
// ./Qt/5.15.2/Src/qtbase/src/corelib/kernel/qcoreapplication.cpp
/*!
    \since 4.3

    Adds the event \a event, with the object \a receiver as the
    receiver of the event, to an event queue and returns immediately.

    The event must be allocated on the heap since the post event queue
    will take ownership of the event and delete it once it has been
    posted.  It is \e {not safe} to access the event after
    it has been posted.

    When control returns to the main event loop, all events that are
    stored in the queue will be sent using the notify() function.

    Events are sorted in descending \a priority order, i.e. events
    with a high \a priority are queued before events with a lower \a
    priority. The \a priority can be any integer value, i.e. between
    INT_MAX and INT_MIN, inclusive; see Qt::EventPriority for more
    details. Events with equal \a priority will be processed in the
    order posted.

    \threadsafe

    \sa sendEvent(), notify(), sendPostedEvents(), Qt::EventPriority
*/
void QCoreApplication::postEvent(QObject *receiver, QEvent *event, int priority)
{
    Q_TRACE_SCOPE(QCoreApplication_postEvent, receiver, event, event->type());

    if (receiver == nullptr) {
        qWarning("QCoreApplication::postEvent: Unexpected null receiver");
        delete event;
        return;
    }

    auto locker = QCoreApplicationPrivate::lockThreadPostEventList(receiver);
    if (!locker.threadData) {
        // posting during destruction? just delete the event to prevent a leak
        delete event;
        return;
    }

    QThreadData *data = locker.threadData;

    // if this is one of the compressible events, do compression
    if (receiver->d_func()->postedEvents
        && self && self->compressEvent(event, receiver, &data->postEventList)) {
        Q_TRACE(QCoreApplication_postEvent_event_compressed, receiver, event);
        return;
    }

    if (event->type() == QEvent::DeferredDelete)
        receiver->d_ptr->deleteLaterCalled = true;

    if (event->type() == QEvent::DeferredDelete && data == QThreadData::current()) {
        // remember the current running eventloop for DeferredDelete
        // events posted in the receiver's thread.

        // Events sent by non-Qt event handlers (such as glib) may not
        // have the scopeLevel set correctly. The scope level makes sure that
        // code like this:
        //     foo->deleteLater();
        //     qApp->processEvents(); // without passing QEvent::DeferredDelete
        // will not cause "foo" to be deleted before returning to the event loop.

        // If the scope level is 0 while loopLevel != 0, we are called from a
        // non-conformant code path, and our best guess is that the scope level
        // should be 1. (Loop level 0 is special: it means that no event loops
        // are running.)
        int loopLevel = data->loopLevel;
        int scopeLevel = data->scopeLevel;
        if (scopeLevel == 0 && loopLevel != 0)
            scopeLevel = 1;
        static_cast<QDeferredDeleteEvent *>(event)->level = loopLevel + scopeLevel;
    }

    // delete the event on exceptions to protect against memory leaks till the event is
    // properly owned in the postEventList
    QScopedPointer<QEvent> eventDeleter(event);
    Q_TRACE(QCoreApplication_postEvent_event_posted, receiver, event, event->type());
    data->postEventList.addEvent(QPostEvent(receiver, event, priority));
    eventDeleter.take();
    event->posted = true;
    ++receiver->d_func()->postedEvents;
    data->canWait = false;
    locker.unlock();

    QAbstractEventDispatcher* dispatcher = data->eventDispatcher.loadAcquire();
    if (dispatcher)
        dispatcher->wakeUp();
}
```
- 关键语句：receiver->d_ptr->deleteLaterCalled = true;
- 关键语句：data->postEventList.addEvent(QPostEvent(receiver, event, priority));

```cpp
// ./Qt/5.15.2/Src/qtbase/src/corelib/kernel/qcoreapplication.cpp
int QCoreApplication::exec()
{
    ...
    int returnCode = eventLoop.exec();
    ...
}

// ./Qt/5.15.2/Src/qtbase/src/corelib/kernel/qeventloop.cpp
int QEventLoop::exec(ProcessEventsFlags flags)
{
   ...
   processEvents(flags | WaitForMoreEvents | EventLoopExec);
   ...
}

// ./Qt/5.15.2/Src/qtbase/src/corelib/kernel/qeventdispatcher_glib.cpp
bool QEventDispatcherGlib::processEvents(QEventLoop::ProcessEventsFlags flags)
{
    ...
    bool result = g_main_context_iteration(d->mainContext, canWait);
    ...
}

// ./Qt/5.15.2/Src/qtbase/src/corelib/kernel/qeventdispatcher_glib.cpp
static gboolean postEventSourceDispatch(GSource *s, GSourceFunc, gpointer)
{
    ...
    QCoreApplication::sendPostedEvents();
    ...
}

// ./Qt/5.15.2/Src/qtbase/src/corelib/kernel/qcoreapplication.cpp
void QCoreApplication::sendPostedEvents(QObject *receiver, int event_type)
{
    ...
    QCoreApplicationPrivate::sendPostedEvents(receiver, event_type, data);
}

// ./Qt/5.15.2/Src/qtbase/src/corelib/kernel/qcoreapplication.cpp
void QCoreApplicationPrivate::sendPostedEvents(QObject *receiver, int event_type,
                                               QThreadData *data)
{
    ...
    while (i < data->postEventList.size()) {
        ...

        // after all that work, it's time to deliver the event.
        QCoreApplication::sendEvent(r, e);
    }
    ...
}

// ./Qt/5.15.2/Src/qtbase/src/corelib/kernel/qcoreapplication.cpp
bool QCoreApplication::sendEvent(QObject *receiver, QEvent *event)
{
    ...
    return notifyInternal2(receiver, event);
}

// ./Qt/5.15.2/Src/qtbase/src/corelib/kernel/qcoreapplication.cpp
bool QCoreApplication::notifyInternal2(QObject *receiver, QEvent *event)
{
    ...
    return self->notify(receiver, event);
}

// ./Qt/5.15.2/Src/qtbase/src/corelib/kernel/qcoreapplication.cpp
bool QCoreApplication::notify(QObject *receiver, QEvent *event)
{
    ...
    return doNotify(receiver, event);
}

// ./Qt/5.15.2/Src/qtbase/src/corelib/kernel/qcoreapplication.cpp
static bool doNotify(QObject *receiver, QEvent *event)
{
    ...
    return receiver->isWidgetType() ? false : QCoreApplicationPrivate::notify_helper(receiver, event);
}

// ./Qt/5.15.2/Src/qtbase/src/corelib/kernel/qobject.cpp
bool QObject::event(QEvent *e)
{
    ...
    case QEvent::DeferredDelete:
        qDeleteInEventHandler(this);
        break;
    ...
}

~TestClass()
{
    qDebug() << "TestClass dector";
}
```

---

##### 三、demo code：

```cpp
#include <QCoreApplication>
#include <QObject>
#include <QDebug>

class TestClass : public QObject
{
public:
    explicit TestClass(QObject *parent = nullptr)
        : QObject(parent)
    {}
    ~TestClass()
    {
        qDebug() << "TestClass dector";	// 打断点在这行
    }
};

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    TestClass* t = new TestClass();
    t->deleteLater();

    return a.exec();
}
```
