1、下载hiredis

`https://github.com/redis/hiredis`

得到hiredis-master.zip，解压后，得到hiredis-master目录，可以看到CMakeLists.txt。

2、下载CMake

`https://cmake.org/download/`

选择 cmake-3.23.0-windows-x86_64.msi

具体自行百度 Windows下CMake的下载与安装详解

3、打开cmd，进入hiredis-master目录。

```
; 查看CMake版本号
cmake --version

; 进入hiredis-master目录
cd D:\MyCode\OwnCode\redis\test\tools\redis\hiredis-master

; 创建build目录
mkdir build

; 进入build目录
cd build

; 运行cmake命令进行解析，检查依赖项（需要安装VS Build Tools)，我本机安装了VS 2015
cmake ..

; 运行编译命令即可自动开始编译，这里的Target与vcxproj后缀的文件保持同名
cmake --build . --target hiredis --config Release

```
最后会在./hiredis-master/build/Release目录下，生成hiredis.dll

4、编译其他小工具

`https://github.com/microsoftarchive/redis`

得到redis-3.0.zip。解压后，进入./redis-3.0/msvs，然后用vs打开RedisServer.sln项目，编译全部。

编译完成后在./redis-3.0/msvs/x64/Release里面找5个.exe文件，分别是redis-benchmark.exe，redis-check-aof.exe，redis-check-dump.exe，redis-cli.exe，redis-server.exe。

- redis-server：Redis服务器的daemon启动程序
- redis-cli：Redis命令行操作工具。也可以用telnet根据其纯文本协议来操作
- redis-benchmark：Redis性能测试工具，测试Redis在当前系统下的读写性能
- redis-check-aof：数据修复
- redis-check-dump：检查导出工具


5、测试Demo(Qt 5.7.0 + mingw5.3.0 + Windows10)

其他IDE也行，关键是链接hiredis.dll动态库。

```
// Pro文件如下：
QT += core
QT -= gui

CONFIG += c++11

TARGET = test
CONFIG += console
CONFIG -= app_bundle

TEMPLATE = app

SOURCES += main.cpp

unix|win32: LIBS += -L$$PWD/tools/redis/hiredis-master/build/Release -lhiredis
INCLUDEPATH += $$PWD/tools/redis/hiredis-master
DEPENDPATH += $$PWD/tools/redis/hiredis-master


// main.cpp如下：
#include <QCoreApplication>
#include <tools/redis/hiredis-master/hiredis.h>
#include <QDebug>

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    redisContext* pRedis = redisConnect("127.0.0.1", 6379);

    if(nullptr == pRedis || 0 != pRedis->err)
    {
        qDebug() << pRedis->errstr << endl;
        return -1;
    }
    else
    {
        qDebug() << "connect successfully" << endl;
    }

    QString sSetCmd = QString::fromUtf8("set hello redisWorld!");
    redisReply* pReplySet = (redisReply*)redisCommand(pRedis, sSetCmd.toStdString().c_str());
    qDebug() << "set cmd: " << sSetCmd;
    qDebug() << "set reply: " << pReplySet->str << endl;
	freeReplyObject(pReplySet);

    QString sGetCmd = QString::fromUtf8("get hello");
    redisReply* pReplyGet = (redisReply*)redisCommand(pRedis, sGetCmd.toStdString().c_str());
    qDebug() << "get cmd: " << sGetCmd;
    qDebug() << "get reply: " << pReplyGet->str << endl;
	freeReplyObject(pReplyGet);

    return a.exec();
}

```

启动第4步编译出来的redis-server.exe

然后启动Demo程序编译出来的test.exe

```
// 运行test.exe的结果如下：
connect successfully

set cmd:  "set hello redisWorld!"
set reply:  OK

get cmd:  "get hello"
get reply:  redisWorld!
```

