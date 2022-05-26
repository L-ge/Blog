我猜，同一个进程加载同一个动态库两次，用的其实是同一份内存空间。下面直接根据代码看结果。

```
// 下面的测试动态库Test.dll的代码，就一个头文件dll_global.h和一个main.cpp.

#ifndef DLL_GLOBAL_H
#define DLL_GLOBAL_H

#ifdef __cplusplus
#define D_EXTERN_C extern "C"
#else
#define D_EXTERN_C
#endif

#ifdef __SHARE_EXPORT
#define D_SHARE_EXPORT D_DECL_EXPORT
#else
#define D_SHARE_EXPORT D_DECL_IMPORT
#endif

#if defined(WIN32) || defined(_WIN32) || defined(__WIN32) || defined(__WIN32__)
#define D_CALLTYPE __stdcall
#define D_DECL_EXPORT __declspec(dllexport)
#define D_DECL_IMPORT
#else
#define __stdcall
#define D_CALLTYPE
#define D_DECL_EXPORT __attribute__((visibility("default")))
#define D_DECL_IMPORT __attribute__((visibility("default")))
#endif

D_EXTERN_C D_SHARE_EXPORT void D_CALLTYPE Dll_SetValue(int value);
D_EXTERN_C D_SHARE_EXPORT int D_CALLTYPE Dll_GetValue();

#endif // DLL_GLOBAL_H

============================================================================ 

#include "dll_global.h"

int g_value = 0;

//---------------------------------------------------------------------------
void Dll_SetValue(int value)
{
    g_value = value;
}
//---------------------------------------------------------------------------
int Dll_GetValue()
{
    return g_value;
}
//---------------------------------------------------------------------------
```

```
// 下面是测试工具的代码 
#include <QCoreApplication>
#include <QString>
#include <QLibrary>
#include <QDebug>

typedef int (_stdcall *Dll_SetValue)(int value);
typedef int (_stdcall *Dll_GetValue)();

int loadDll_SameDllTwice()
{
    qDebug() << "===========loadDll_SameDllTwice=========";
    Dll_SetValue pDll_SetValue1{nullptr};
    Dll_GetValue pDll_GetValue1{nullptr};
    {
        QLibrary qLib(QString("./TestDll.dll"));
        if(false == qLib.load())
            return -1;

        pDll_SetValue1 = (Dll_SetValue)qLib.resolve("Dll_SetValue");
        if(nullptr == pDll_SetValue1)
            return -2;

        pDll_GetValue1 = (Dll_GetValue)qLib.resolve("Dll_GetValue");
        if(nullptr == pDll_GetValue1)
            return -3;
    }

    Dll_SetValue pDll_SetValue2{nullptr};
    Dll_GetValue pDll_GetValue2{nullptr};
    {
        QLibrary qLib(QString("./TestDll.dll"));
        if(false == qLib.load())
            return -1;

        pDll_SetValue2 = (Dll_SetValue)qLib.resolve("Dll_SetValue");
        if(nullptr == pDll_SetValue2)
            return -2;

        pDll_GetValue2 = (Dll_GetValue)qLib.resolve("Dll_GetValue");
        if(nullptr == pDll_GetValue2)
            return -3;
    }

    qDebug() << "pDll_SetValue1: " << &pDll_SetValue1;
    qDebug() << "pDll_SetValue2: " << &pDll_SetValue2;
    qDebug() << "pDll_GetValue1: " << &pDll_GetValue1;
    qDebug() << "pDll_GetValue2: " << &pDll_GetValue2;

    pDll_SetValue1(123);
    qDebug() << "pDll_SetValue1(123), pDll_GetValue2=" << pDll_GetValue2() << endl;

    return 0;
}

int loadDll_DiffDllOnce()
{
    qDebug() << "===========loadDll_DiffDllOnce=========";
    Dll_SetValue pDll_SetValue1{nullptr};
    Dll_GetValue pDll_GetValue1{nullptr};
    {
        QLibrary qLib(QString("./TestDll1.dll"));
        if(false == qLib.load())
            return -1;

        pDll_SetValue1 = (Dll_SetValue)qLib.resolve("Dll_SetValue");
        if(nullptr == pDll_SetValue1)
            return -2;

        pDll_GetValue1 = (Dll_GetValue)qLib.resolve("Dll_GetValue");
        if(nullptr == pDll_GetValue1)
            return -3;
    }

    Dll_SetValue pDll_SetValue2{nullptr};
    Dll_GetValue pDll_GetValue2{nullptr};
    {
        QLibrary qLib(QString("./TestDll2.dll"));
        if(false == qLib.load())
            return -1;

        pDll_SetValue2 = (Dll_SetValue)qLib.resolve("Dll_SetValue");
        if(nullptr == pDll_SetValue2)
            return -2;

        pDll_GetValue2 = (Dll_GetValue)qLib.resolve("Dll_GetValue");
        if(nullptr == pDll_GetValue2)
            return -3;
    }

    qDebug() << "pDll_SetValue1: " << &pDll_SetValue1;
    qDebug() << "pDll_SetValue2: " << &pDll_SetValue2;
    qDebug() << "pDll_GetValue1: " << &pDll_GetValue1;
    qDebug() << "pDll_GetValue2: " << &pDll_GetValue2;

    pDll_SetValue1(234);
    qDebug() << "pDll_SetValue1(234), pDll_GetValue2=" << pDll_GetValue2() << endl;

    return 0;
}

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    loadDll_SameDllTwice();

    loadDll_DiffDllOnce();

    return a.exec();
}
```
```
===========loadDll_SameDllTwice=========
pDll_SetValue1:  0x60fe14
pDll_SetValue2:  0x60fe1c
pDll_GetValue1:  0x60fe18
pDll_GetValue2:  0x60fe20
pDll_SetValue1(123), pDll_GetValue2= 123

===========loadDll_DiffDllOnce=========
pDll_SetValue1:  0x60fe14
pDll_SetValue2:  0x60fe1c
pDll_GetValue1:  0x60fe18
pDll_GetValue2:  0x60fe20
pDll_SetValue1(234), pDll_GetValue2= 0
```