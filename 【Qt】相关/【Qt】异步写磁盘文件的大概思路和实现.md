大概思路：直接看代码吧，逻辑很简单。

bool ASaveFile::writeFile(const QByteArray &baSend)：对外提供的接口.

void ASaveFile::sltWriteFile()：发送sigWriteFile信号后执行该槽函数(运行于独立线程).

下面代码一共有三个文件，分别是main.cpp、asavefile.h、asavefile.cpp.

其中main.cpp是用于测试ASaveFile的。ASaveFile运行于线程中。
```
// asavefile.h的源代码
#ifndef ASAVEFILE_H
#define ASAVEFILE_H

#include <QObject>
#include <QThread>
#include <QMutex>

class ASaveFile : QObject
{
    Q_OBJECT
public:
    explicit ASaveFile();
    virtual ~ASaveFile();

    bool writeFile(const QByteArray &baSend);

signals:
    void sigWriteFile();

public slots:
    void sltWriteFile();

private:
    QThread m_thread;
    QMutex m_mtxFile;

    QList<QByteArray> m_lstCache;
};

#endif // ASAVEFILE_H
```

```
// asavefile.cpp的源代码
#include "asavefile.h"
#include <QDebug>

ASaveFile::ASaveFile()
{
    qDebug() << QThread::currentThreadId() << "ASaveFile ctor";
    connect(this, SIGNAL(sigWriteFile()), this, SLOT(sltWriteFile()));
    m_thread.start();
    this->moveToThread(&m_thread);
}

ASaveFile::~ASaveFile()
{
    qDebug() << QThread::currentThreadId() << "ASaveFile dector";
    m_thread.quit();
    m_thread.wait();
    m_thread.deleteLater();
}

bool ASaveFile::writeFile(const QByteArray &baSend)
{
    qDebug() << QThread::currentThreadId() << "ASync writeFile";
    {
        QMutexLocker locker(&m_mtxFile);
        m_lstCache.append(baSend);
    }
    emit sigWriteFile();
}

void ASaveFile::sltWriteFile()
{
    QList<QByteArray> lstCacheTmp;
    {
        QMutexLocker locker(&m_mtxFile);
        lstCacheTmp.swap(m_lstCache);
        m_lstCache.clear();
    }

    for(int i=0; i<lstCacheTmp.size(); ++i)
    {
        // TODO: write disk
        qDebug() << QThread::currentThreadId() << "Write Disk : " << lstCacheTmp.at(i);
        QThread::sleep(1);
    }
}
```

```
// main.cpp的源代码
#include <QCoreApplication>
#include <QDebug>
#include "asavefile.h"

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    {
        ASaveFile s;
        int i = 10;
        while(i--)
        {
            s.writeFile(QString::number(i).toLocal8Bit());
        }

        //QThread::sleep(15);    // 该语句测试用.s析构后,线程终止
    }

    qDebug() << QThread::currentThreadId() << "mainthread fin";


    return a.exec();
}
```