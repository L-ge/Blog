如何发送按键事件到当前焦点所在的窗体？
```
QWidget* obj = QApplication::focusWidget();
// 要注意判空,否则会崩溃.
// 因为不管是子窗口还是主窗口,都需要当前widget是能够捕获到焦点的.
if(nullptr != obj)
{
	// 当前焦点所在位置(也即接收该按键事件的是哪个widget)
	qDebug() << "ObjectName:" << obj->objectName();
	// 如果没有后面第四个参数,那么当发送键盘事件给QLineEdit的时候,显示不出东西.
	// 发送键盘事件给QLineEdit,那么QLineEdit显示heiheihei.
	QKeyEvent keyPress(QEvent::KeyPress, Qt::Key_0, Qt::NoModifier, "heiheihei");
	QApplication::sendEvent(obj, &keyPress);
}
```

附上一下帮助文档：

①QKeyEvent的构造函数：

QKeyEvent::QKeyEvent(Type type, int key, Qt::KeyboardModifiers modifiers, const QString &text = QString(), bool autorep = false, ushort count = 1)

Constructs a key event object.

The type parameter must be QEvent::KeyPress, QEvent::KeyRelease, or QEvent::ShortcutOverride.

Int key is the code for the Qt::Key that the event loop should listen for. If key is 0, the event is not a result of a known key; for example, it may be the result of a compose sequence or keyboard macro. The modifiers holds the keyboard modifiers, and the given text is the Unicode text that the key generated. If autorep is true, isAutoRepeat() will be true. count is the number of keys involved in the event.


②QApplication::focusWidget()的解释：

[static] QWidget *QApplication::focusWidget()

Returns the application widget that has the keyboard input focus, or 0 if no widget in this application has the focus.