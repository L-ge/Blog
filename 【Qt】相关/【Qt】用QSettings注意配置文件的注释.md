使用QSettings后，配置文件的注释要用分号，不要用双反斜杠。
```
#include <QCoreApplication>
#include <QSettings>
#include <QDebug>

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    QSettings setting("./MyConfig.ini", QSettings::IniFormat);
    QString sUseBackslash = setting.value("Test/useBackslash", "").toString();
    QString sUseSemicolon= setting.value("Test/useSemicolon", "").toString();

    qDebug() << sUseBackslash;
    qDebug() << sUseSemicolon;

    return a.exec();
}
```
```
// 下面是./MyConfig.ini的内容
[Test]
useBackslash = 1234 //1234
useSemicolon = 1234 ;1234
```
```
// 下面是测试程序的输出
"1234 //1234"
"1234"
```

