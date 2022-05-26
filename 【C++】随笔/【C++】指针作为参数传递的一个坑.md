```
#include <QCoreApplication>
#include <QDebug>

void func1(int *pCnt)
{
    pCnt = new int(1);
}

void func2(int* &pCnt)
{
    pCnt = new int(2);
}

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    int *pCnt1 = nullptr;
    func1(pCnt1);
    if(nullptr != pCnt1)
        qDebug() << "*pCnt1: " << *pCnt1;
    else
        qDebug() << "pCnt1 nullptr";        // 输出这句


    int *pCnt2 = nullptr;
    func2(pCnt2);
    if(nullptr != pCnt2)
        qDebug() << "*pCnt2: " << *pCnt2;   // 输出这句
    else
        qDebug() << "pCnt2 nullptr";

    return a.exec();
}
```
```
// 运行结果：

pCnt1 nullptr
*pCnt2:  2
```