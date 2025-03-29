包含 acheckmemleak.h 头文件，然后使用 CHECK_MEMLEAK 宏定义就行。具体看如下代码：
```
#ifndef ACHECKMEMLEAK_H
#define ACHECKMEMLEAK_H

#define CHECK_MEMLEAK(nChangeByte) ACheckMemLeak aCheckMemLeak(nChangeByte, __FILE__, __FUNCTION__, __LINE__);

#include <QString>
#include <QDebug>

#ifdef Q_OS_WIN32
    #include <windows.h>
    #include <psapi.h>
#elif defined(Q_OS_LINUX)
    #include <sys/types.h>
    #include <unistd.h>
    #include <sstream>
    #include <fstream>
    #include <string>
    #include <QFile>
#endif

class ACheckMemLeak
{
public:
    explicit ACheckMemLeak(int nChangeByte, const char* pFileName, const char* pFuncName, int nFileLineNo)
        : m_nChangeByte(nChangeByte)
        , m_sFileName(pFileName)
        , m_sFuncName(pFuncName)
        , m_nFileLineNo(nFileLineNo)
    {
        if(0 != m_llMemStart)
        {
            long long llMemEnd = getProcessPhyMem();
            long long llChangeByte = llMemEnd - m_llMemStart;
            qDebug() << QString("[CheckMemLeak-%1-%2-%3]Start:%4B,End:%5B,Change:%6B=%7KB=%8MB)")
                        .arg(m_sFileName.c_str()).arg(m_sFuncName.c_str()).arg(m_nFileLineNo)
                        .arg(m_llMemStart).arg(llMemEnd)
                        .arg(llChangeByte).arg(llChangeByte/1024).arg(llChangeByte/1024/1024);
            if(llChangeByte > m_nChangeByte)
                qWarning() << "Maybe CheckMemLeak...";
            m_llMemStart = llMemEnd;
        }
        else
            m_llMemStart = getProcessPhyMem();
    }

    ~ACheckMemLeak()
    {
        long long llMemEnd = getProcessPhyMem();
        long long llChangeByte = llMemEnd - m_llMemStart;
        qDebug() << QString("[CheckMemLeak-%1-%2-%3]Start:%4B,End:%5B,Change:%6B=%7KB=%8MB)")
                    .arg(m_sFileName.c_str()).arg(m_sFuncName.c_str()).arg(m_nFileLineNo)
                    .arg(m_llMemStart).arg(llMemEnd)
                    .arg(llChangeByte).arg(llChangeByte/1024).arg(llChangeByte/1024/1024);
        if(llChangeByte > m_nChangeByte)
            qWarning() << "Maybe CheckMemLeak...";
        m_llMemStart = llMemEnd;
    }

private:
    long long getProcessPhyMem()
    {
    #ifdef Q_OS_WIN32
        PROCESS_MEMORY_COUNTERS pmc;
        if (GetProcessMemoryInfo(GetCurrentProcess(), &pmc, sizeof(pmc)))
            return pmc.WorkingSetSize;
    #elif defined(Q_OS_LINUX)
        // 结果*4*1024是因为以内存页4KB为单位的.
        uid_t pid = getpid();
        /* C语言方式
        char fileName[100] = {0};
        sprintf(fileName, "/proc/%d/statm", pid);
        FILE* pFile = NULL;
        pFile = fopen(fileName, "r");
        if(NULL == pFile)
            return 0;
        char buff[255] = {0};
        if(NULL != fgets(buff, 255, (FILE*)pFile))
        {
            char *pNeedContent = strtok(buff, " ");
            pNeedContent = strtok(NULL, " ");
            return atoll(pNeedContent)*4*1024;
        }*/

        /* C++方式
        std::stringstream strStream;
        strStream << "/proc/" << pid << "/statm";
        std::string sFileName = strStream.str();
        std::ifstream f(sFileName.c_str(), std::ios::in);
        if(false == f.is_open())
            return 0;
        std::string sContent;
        if(getline(f, sContent))
        {
            char cContentArr[sContent.length()+1];
            bzero(cContentArr, sizeof(cContentArr));
            strcpy(cContentArr, sContent.c_str());
            char* pNeedContent = strtok(cContentArr, " ");
            pNeedContent = strtok(NULL, " ");
            return atoll(pNeedContent)*4*1024;
        }*/

        // Qt方式
        QFile f(QString("/proc/%1/statm").arg(QString::number(pid)));
        if (false == f.open(QIODevice::ReadOnly))
            return 0;
        QByteArray ba = f.readAll();
        f.close();
        QByteArrayList lst = ba.split(' ');
        return lst.at(1).toLongLong()*4*1024;

    #endif
        return 0;
    }

private:
    int m_nChangeByte;
    std::string m_sFileName;
    std::string m_sFuncName;
    int m_nFileLineNo;

    static long long m_llMemStart;
};

long long ACheckMemLeak::m_llMemStart = 0;

#endif // ACHECKMEMLEAK_H
```

下面是测试的代码：
```
#include <QCoreApplication>
#include <QDebug>
#include <QThread>
#include "acheckmemleak.h"

void func()
{
    for(int i=0; i<1000; i++)
        int *p = new int(i);
}

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    for(int i=0; i<10; i++)
    {
        {
            CHECK_MEMLEAK(0)
            func();
        }
        QThread::sleep(1);
    }

    return a.exec();
}
```

```
// 注意链接库文件，下面是 Qt 方式（在.Pro文件加上）
windows{
    LIBS += -lpsapi
}
```