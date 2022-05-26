客户端通信流程:

①创建套接字：new QTcpSocket。

②连接服务器：可以使用QTcpSocket类的connectToHost()函数来连接服务器。

③向服务器发送或者接受数据。


下面代码一共有三个文件，分别是main.cpp、atcpclient.h、atcpclient.cpp.

其中main.cpp是用于测试ATcpClient的。封装好了的ATcpClient运行于线程中，可用于其他小项目中。
```
// atcpclient.h

#ifndef ATCPCLIENT_H
#define ATCPCLIENT_H

#include <QObject>
#include <QTcpSocket>
#include <QThread>
#include <QTimer>

class ATcpClient : public QObject
{
    Q_OBJECT
public:
    ATcpClient(QString sServerIp, int nPort, QObject *parent = 0);
    virtual ~ATcpClient();

    int  sendData(QByteArray baData);
    void setReConnectInterval(int nMSec);

protected:
    virtual void initInThread();
    virtual void deinitInThread();

    virtual void doReConnect();
    virtual void setConnectStatus(bool bIsConnect);

    virtual int doSendDataImp(QByteArray baData);
    virtual int doRecvDataImp(const QByteArray &baRecvData);

signals:
    void sigSendData(QByteArray baData);
    void sigOnDeinit();

private slots:
    void sltSendData(QByteArray baData);
    void sltReadyRead();

    void sltOnConnected();
    void sltOnDisonnected();
    void sltOnError(QAbstractSocket::SocketError nErrCode);
    void sltOnStateChanged(QAbstractSocket::SocketState nSocketState);

private:
    QString m_sServerIp;
    int m_nPort;
    int m_nInterval;
    bool m_bConnected{false};

    QThread* m_pThread{nullptr};
    QTimer* m_pTimer{nullptr};
    QTcpSocket* m_pSocketClient{nullptr};
};

#endif // ATCPCLIENT_H
```

```
// atcpclient.cpp

#include "atcpclient.h"
#include <QHostAddress>
#include <QTimer>
#include <QEventLoop>

ATcpClient::ATcpClient(QString sServerIp, int nPort, QObject *parent)
    : m_sServerIp(sServerIp)
    , m_nPort(nPort)
    , m_nInterval(50)
    , QObject(parent)
{
    qDebug() << QString::fromLocal8Bit("ATcpClient::ATcpClient(%1,%2)").arg(m_sServerIp).arg(m_nPort);

    m_pThread = new QThread();
    m_pThread->setObjectName("TcpClient");
    m_pThread->start();
    this->moveToThread(m_pThread);

    //不要直接调用initInThread(),否则并非在线程内初始化
    QTimer::singleShot(0,this, &ATcpClient::initInThread);
}

ATcpClient::~ATcpClient()
{
    qDebug() << "ATcpClient::~ATcpClient()";

    //两种错误的反初始化方法,会导致无法kill掉QTimer.
    //deinitInThread();
    //QTimer::singleShot(0, this, &ATcpClient::deinitInThread);

    //正确的反初始化方法: 在线程内反初始化
    QEventLoop loop;
    connect(this, SIGNAL(sigOnDeinit()), &loop, SLOT(quit()));
    QTimer::singleShot(0, this, &ATcpClient::deinitInThread);
    loop.exec();

    if (nullptr != m_pThread)
    {
        m_pThread->quit();
        m_pThread->wait();
        m_pThread->deleteLater();
    }
}

int ATcpClient::sendData(QByteArray baData)
{
    emit sigSendData(baData);
    return baData.length();
}

void ATcpClient::setReConnectInterval(int nMSec)
{
    m_nInterval = nMSec;
}

void ATcpClient::initInThread()
{
    if(nullptr == m_pTimer)
    {
        m_pTimer = new QTimer(this);
        connect(m_pTimer, &QTimer::timeout, this, &ATcpClient::doReConnect);
    }
    m_pTimer->start(m_nInterval);

    if(nullptr == m_pSocketClient)
    {
        m_pSocketClient = new QTcpSocket(this);
        connect(m_pSocketClient, SIGNAL(connected()), this, SLOT(sltOnConnected()));
        connect(m_pSocketClient, SIGNAL(disconnected()), this, SLOT(sltOnDisonnected()));
        connect(m_pSocketClient, SIGNAL(stateChanged(QAbstractSocket::SocketState)), this, SLOT(sltOnStateChanged(QAbstractSocket::SocketState)));
        connect(m_pSocketClient, SIGNAL(error(QAbstractSocket::SocketError)), this, SLOT(sltOnError(QAbstractSocket::SocketError)));
        connect(m_pSocketClient, SIGNAL(readyRead()), this, SLOT(sltReadyRead()));

        m_pSocketClient->setSocketOption(QAbstractSocket::LowDelayOption, 1);// 禁用Nagle算法.
        m_pSocketClient->setSocketOption(QAbstractSocket::KeepAliveOption, 1);
        m_pSocketClient->connectToHost(QHostAddress(m_sServerIp), m_nPort);
    }

    connect(this, SIGNAL(sigSendData(QByteArray)), this, SLOT(sltSendData(QByteArray)));
}

void ATcpClient::deinitInThread()
{
    if(nullptr != m_pTimer)
    {
        m_pTimer->stop();
        delete m_pTimer;
        m_pTimer = nullptr;
    }

    if(nullptr != m_pSocketClient)
    {
	m_pSocketClient->disconnectFromHost();
	delete m_pSocketClient;
	m_pSocketClient = NULL;
    }

    emit sigOnDeinit();
}

void ATcpClient::doReConnect()
{
    if(false == m_bConnected && nullptr != m_pSocketClient)
    {
        m_pSocketClient->disconnectFromHost();
        m_pSocketClient->connectToHost(QHostAddress(m_sServerIp), m_nPort);
    }
}

void ATcpClient::setConnectStatus(bool bIsConnect)
{
    if(m_bConnected != bIsConnect)
    {
        m_bConnected = bIsConnect;
    }
}

int ATcpClient::doSendDataImp(QByteArray baData)
{
    if(nullptr == m_pSocketClient || false == m_bConnected)
    {
        qDebug() << "TcpClient socket not connect.";
        return -1;
    }

    int nWriteSize = m_pSocketClient->write(baData);
    if(m_pSocketClient->flush())
    {
        qDebug() << "[" << QString::fromLocal8Bit("%1:%2").arg(m_sServerIp).arg(m_nPort)
                 << "] Write["<< nWriteSize
                 <<"]:"<< QString::fromLocal8Bit(baData);
    }
    else
    {
        qDebug() << "[" << QString::fromLocal8Bit("%1:%2").arg(m_sServerIp).arg(m_nPort)
                 << "] Write Fail["<< nWriteSize
                 << "]:"<< QString::fromLocal8Bit(baData);
    }

    return nWriteSize;
}

int ATcpClient::doRecvDataImp(const QByteArray &baRecvData)
{
    qDebug() << "[" << QString::fromLocal8Bit("%1:%2").arg(m_sServerIp).arg(m_nPort)
             << "] Recv: " << QString::fromLocal8Bit(baRecvData);
}

void ATcpClient::sltSendData(QByteArray baData)
{
    doSendDataImp(baData);
}

void ATcpClient::sltReadyRead()
{
    QByteArray baRecv = m_pSocketClient->readAll();
    doRecvDataImp(baRecv);
}

void ATcpClient::sltOnConnected()
{
    setConnectStatus(true);
    qDebug() << "[" << QString::fromLocal8Bit("%1:%2").arg(m_sServerIp).arg(m_nPort)
             <<"] Connect success.";
}

void ATcpClient::sltOnDisonnected()
{
    setConnectStatus(false);
    qDebug() << "[" << QString::fromLocal8Bit("%1:%2").arg(m_sServerIp).arg(m_nPort)
             << "] Disconnect success.";
}

void ATcpClient::sltOnError(QAbstractSocket::SocketError nErrCode)
{
    qDebug() << "[" << QString::fromLocal8Bit("%1:%2").arg(m_sServerIp).arg(m_nPort)
             << "] socket error,code:" << nErrCode;
	m_pSocketClient->disconnectFromHost();
    setConnectStatus(false);
}

void ATcpClient::sltOnStateChanged(QAbstractSocket::SocketState nSocketState)
{
    qDebug() << "[" << QString::fromLocal8Bit("%1:%2").arg(m_sServerIp).arg(m_nPort)
             << "] socket state change:" << nSocketState;
}
```

```
// main.cpp-用于测试

#include <QCoreApplication>
#include "atcpclient.h"

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    {
        ATcpClient tcpClient("127.0.0.1", 9999);

        for(int i=0; i<5; ++i)
        {
           QThread::sleep(1);
           tcpClient.sendData("aaaa");
        }
    }

    return a.exec();
}
```