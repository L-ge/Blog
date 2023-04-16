##### 一、QA：

1. moveToThread 的限制有哪些？
- 如果对象已经在目标线程了，则不允许 moveToThread。
- 如果对象指定父对象了，则不允许 moveToThread。
- 如果对象是 widget，则不允许 moveToThread。

2. 当一个对象 moveToThread 之后，继承自它的对象也会跟着 moveToThread 吗？
- 会。其所有子类对象都会移到目标线程。

3. moveToThread 函数中主要的逻辑都在 setThreadData_helper 函数，里面做了些什么？
- 当前线程已经 post 的事件全部移动到目标线程，然后目标线程开始事件分发。
- 释放当前线程一些当前正在发送的信号。
- 把当前线程绑定的信号槽的接收者改成目标线程。
- 调用其所有子类的 setThreadData_helper 函数。

4. moveToThread 改变的是什么？
- 改变的是一个类型为 QAtomicPointer<QThreadData> 的变量 threadData。
- QThreadData 是 Qt 自己封装的一个线程结构类，里面有如 threadId 等成员。
- 另外，其实 Qt 的线程是在操作系统线程上面的再一次封装，追踪 threadId 的赋值来源可以知道。

---

##### 二、source code：

```cpp
// /Qt/5.15.2/Src/qtbase/src/corelib/kernel/qobject.cpp
/*!
    Changes the thread affinity for this object and its children. The
    object cannot be moved if it has a parent. Event processing will
    continue in the \a targetThread.

    To move an object to the main thread, use QApplication::instance()
    to retrieve a pointer to the current application, and then use
    QApplication::thread() to retrieve the thread in which the
    application lives. For example:

    \snippet code/src_corelib_kernel_qobject.cpp 7

    If \a targetThread is \nullptr, all event processing for this object
    and its children stops, as they are no longer associated with any
    thread.

    Note that all active timers for the object will be reset. The
    timers are first stopped in the current thread and restarted (with
    the same interval) in the \a targetThread. As a result, constantly
    moving an object between threads can postpone timer events
    indefinitely.

    A QEvent::ThreadChange event is sent to this object just before
    the thread affinity is changed. You can handle this event to
    perform any special processing. Note that any new events that are
    posted to this object will be handled in the \a targetThread,
    provided it is not \nullptr: when it is \nullptr, no event processing
    for this object or its children can happen, as they are no longer
    associated with any thread.

    \warning This function is \e not thread-safe; the current thread
    must be same as the current thread affinity. In other words, this
    function can only "push" an object from the current thread to
    another thread, it cannot "pull" an object from any arbitrary
    thread to the current thread. There is one exception to this rule
    however: objects with no thread affinity can be "pulled" to the
    current thread.

    \sa thread()
 */
void QObject::moveToThread(QThread *targetThread)
{
    Q_D(QObject);

    if (d->threadData.loadRelaxed()->thread.loadAcquire() == targetThread) {
        // object is already in this thread
        return;
    }

    if (d->parent != nullptr) {
        qWarning("QObject::moveToThread: Cannot move objects with a parent");
        return;
    }
    if (d->isWidget) {
        qWarning("QObject::moveToThread: Widgets cannot be moved to a new thread");
        return;
    }

    QThreadData *currentData = QThreadData::current();
    QThreadData *targetData = targetThread ? QThreadData::get2(targetThread) : nullptr;
    QThreadData *thisThreadData = d->threadData.loadRelaxed();
    if (!thisThreadData->thread.loadAcquire() && currentData == targetData) {
        // one exception to the rule: we allow moving objects with no thread affinity to the current thread
        currentData = d->threadData;
    } else if (thisThreadData != currentData) {
        qWarning("QObject::moveToThread: Current thread (%p) is not the object's thread (%p).\n"
                 "Cannot move to target thread (%p)\n",
                 currentData->thread.loadRelaxed(), thisThreadData->thread.loadRelaxed(), targetData ? targetData->thread.loadRelaxed() : nullptr);

#ifdef Q_OS_MAC
        qWarning("You might be loading two sets of Qt binaries into the same process. "
                 "Check that all plugins are compiled against the right Qt binaries. Export "
                 "DYLD_PRINT_LIBRARIES=1 and check that only one set of binaries are being loaded.");
#endif

        return;
    }

    // prepare to move
    d->moveToThread_helper();

    if (!targetData)
        targetData = new QThreadData(0);

    // make sure nobody adds/removes connections to this object while we're moving it
    QMutexLocker l(signalSlotLock(this));

    QOrderedMutexLocker locker(&currentData->postEventList.mutex,
                               &targetData->postEventList.mutex);

    // keep currentData alive (since we've got it locked)
    currentData->ref();

    // move the object
    d_func()->setThreadData_helper(currentData, targetData);

    locker.unlock();

    // now currentData can commit suicide if it wants to
    currentData->deref();
}
```

```cpp
#define Q_D(Class) Class##Private * const d = d_func()

    
class Q_CORE_EXPORT QObjectPrivate : public QObjectData
{
    ...
public:
    ...
    // This atomic requires acquire/release semantics in a few places,
    // e.g. QObject::moveToThread must synchronize with QCoreApplication::postEvent,
    // because postEvent is thread-safe.
    // However, most of the code paths involving QObject are only reentrant and
    // not thread-safe, so synchronization should not be necessary there.
    QAtomicPointer<QThreadData> threadData; // id of the thread that owns the object
    ...
};


void QObjectPrivate::moveToThread_helper()
{
    Q_Q(QObject);
    QEvent e(QEvent::ThreadChange);
    QCoreApplication::sendEvent(q, &e);
    for (int i = 0; i < children.size(); ++i) {
        QObject *child = children.at(i);
        child->d_func()->moveToThread_helper();
    }
}

void QObjectPrivate::setThreadData_helper(QThreadData *currentData, QThreadData *targetData)
{
    Q_Q(QObject);

    // move posted events
    int eventsMoved = 0;
    for (int i = 0; i < currentData->postEventList.size(); ++i) {
        const QPostEvent &pe = currentData->postEventList.at(i);
        if (!pe.event)
            continue;
        if (pe.receiver == q) {
            // move this post event to the targetList
            targetData->postEventList.addEvent(pe);
            const_cast<QPostEvent &>(pe).event = nullptr;
            ++eventsMoved;
        }
    }
    if (eventsMoved > 0 && targetData->hasEventDispatcher()) {
        targetData->canWait = false;
        targetData->eventDispatcher.loadRelaxed()->wakeUp();
    }

    // the current emitting thread shouldn't restore currentSender after calling moveToThread()
    ConnectionData *cd = connections.loadRelaxed();
    if (cd) {
        if (cd->currentSender) {
            cd->currentSender->receiverDeleted();
            cd->currentSender = nullptr;
        }

        // adjust the receiverThreadId values in the Connections
        if (cd) {
            auto *c = cd->senders;
            while (c) {
                QObject *r = c->receiver.loadRelaxed();
                if (r) {
                    Q_ASSERT(r == q);
                    targetData->ref();
                    QThreadData *old = c->receiverThreadData.loadRelaxed();
                    if (old)
                        old->deref();
                    c->receiverThreadData.storeRelaxed(targetData);
                }
                c = c->next;
            }
        }

    }

    // set new thread data
    targetData->ref();
    threadData.loadRelaxed()->deref();

    // synchronizes with loadAcquire e.g. in QCoreApplication::postEvent
    threadData.storeRelease(targetData);

    for (int i = 0; i < children.size(); ++i) {
        QObject *child = children.at(i);
        child->d_func()->setThreadData_helper(currentData, targetData);
    }
}
```
