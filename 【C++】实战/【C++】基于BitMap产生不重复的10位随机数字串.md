```
#include <QCoreApplication>
#include <vector>
#include <chrono>
#include <iostream>

// 位图的每一位的0,1标志这个数存在或不存在的状态
class BitMap
{
public:
    BitMap(size_t nSize = 1024)
    {
        vecArray.resize(nSize/32+1);    // 加1是为了防止除不尽的时候外加一个size_t来保存位
    }

    // 将这个数的状态位置为1
    void set(const size_t &nValue)
    {
        size_t index = nValue >> 5;     // 除以32，取nValue在数组中的索引
        size_t bit = nValue % 32;       // 判断在32位中的哪一位
        vecArray[index] |= (1<<bit);
    }

    // 将这个数的状态位置为0
    void reset(const size_t &nValue)
    {
        size_t nIndex = nValue >> 5;
        size_t nBit = nValue % 32;
        vecArray[nIndex] &= (~(1<<nBit));   // ~按位取反
    }

    // 这个数的状态位是否为1
    bool test(const size_t &nValue)
    {
        size_t nIndex = nValue >> 5;
        size_t nBit = nValue % 32;
        return (vecArray[nIndex] & (1<<nBit));
    }

private:
    std::vector<size_t> vecArray;
};

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    // obtain a seed from the system clock:
    unsigned int seed = std::chrono::system_clock::now().time_since_epoch().count();
    std::mt19937_64 rng(seed);
    std::uniform_int_distribution<uint64_t> distribution(0, size_t(-1));   // 产生0-4294967295的随机整数

    BitMap bm(size_t(-1));
    std::cout << "Num: " << size_t(-1) << std::endl;
    for(size_t i=0; i<10000; )       // 产生10000个
    {
        std::string sIdTmp = std::string(10,'0') + std::to_string(distribution(rng));   // 产生的随机数不够10位,左边补0
        std::string sId = sIdTmp.substr(sIdTmp.length()-10);        // 取右边10位
        unsigned long nId = std::stoul(sId);

        if(false == bm.test(nId))
        {
            std::cout << "Id: " << sId << std::endl;
            bm.set(nId);
            ++i;
        }
    }

    return a.exec();
}
```
