# 站在巨人的肩膀上

> [C++高性能分布式服务器框架](https://github.com/sylar-yin/sylar)

> [从零开始重写sylar C++高性能分布式服务器框架](https://www.midlane.top/wiki/pages/viewpage.action?pageId=10060952)


# 概述

- 二进制数组（序列化/反序列化）模块。
- 字节数组容器，提供基础类型的序列化与反序列化功能。
- ByteArray 的底层存储是固定大小的块，以链表形式组织。每次写入数据时，将数据写入到链表最后一个块中，如果最后一个块不足以容纳数据，则分配一个新的块并添加到链表结尾，再写入数据。ByteArray 会记录当前的操作位置，每次写入数据时，该操作位置按写入大小往后偏移，如果要读取数据，则必须调用 setPosition 重新设置当前的操作位置。
- ByteArray 支持基础类型的序列化与反序列化功能，并且支持将序列化的结果写入文件，以及从文件中读取内容进行反序列化。
- ByteArray 支持以下类型的序列化与反序列化：
	- 固定长度的有符号/无符号8位、16位、32位、64位整数。
    - 不固定长度的有符号/无符号32位、64位整数。
    - float、double 类型。
    - 字符串，包含字符串长度，长度范围支持16位、32位、64位。
    - 字符串，不包含长度。
	- 以上所有的类型都支持读写。
- ByteArray 还支持设置序列化时的大小端顺序。


# ByteArray

- 二进制数组，提供基础类型的序列化，反序列化功能。
- 内含一个 Node 类，作为 ByteArray 的存储节点。
- 相当于一个链表的数据结构。


# 其他说明

- ByteArray 在序列化不固定长度的有符号/无符号32位、64位整数时使用了 zigzag 算法。zigzag 算法参考： [小而巧的数字压缩算法：zigzag](https://blog.csdn.net/zgwangbo/article/details/51590186)
- ByteArray 在序列化字符串时使用 TLV 编码结构中的 Length 和 Value。TLV编码结构用于序列化和消息传递，指 Tag（类型），Length（长度），Value（值），参考：[TLV编码通信协议设计](https://www.wtango.com/tlv%E7%BC%96%E7%A0%81%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE%E8%AE%BE%E8%AE%A1/)


# 待完善

- 感觉这个模块有两个地方有bug。
	- ByteArray::getReadBuffers 方法：
	```C++
	uint64_t ByteArray::getReadBuffers(std::vector<iovec>& buffers, uint64_t len, uint64_t position) const
	{
    // 这里的判断好像没啥用，因为下面用的position是外面传进来的，
    // 与m_position是没有关系的，bug???
    len = len > getReadSize() ? getReadSize() : len;
    ...
	}
	```
	- ByteArray::writeToFile 方法：
	```C++
	bool ByteArray::writeToFile(const std::string& name) const
	{
    ...
    int64_t read_size = getReadSize();
    int64_t pos = m_position;
    Node* cur = m_cur;
    while(read_size > 0)
    {
        int diff = pos % m_baseSize;    // 当前块不可读(已经读完)的字节数
	
        // 1、要读的内容比一整块要大，则读完一整块剩下的那些
        // 2、要读的内容比一整块少，则读？？？
        // 假设一块10字节，可读9字节，不可读是3字节，则读9-3=6字节，bug???
        // 假设一块10字节，可读1字节，不可读是3字节，则读1-3=-5字节，bug???
        // 应该是：
        // int64_t len = 0;
        // if(read_size > (int64_t)m_baseSize - diff)
        //      len = (int64_t)m_baseSize - diff;
        // else
        //      len = read_size;
        int64_t len = (read_size > (int64_t)m_baseSize ? m_baseSize : read_size) - diff;
        ofs.write(cur->ptr + diff, len);
        cur = cur->next;
        pos += len;
        read_size -= len;
    }
	
    return true;
	}
	```


# 部分相关代码
```C++
/**
 * @filename    bytearray.h
 * @brief   ByteArray 模块
 * @author  L-ge
 * @version 0.1
 * @modify  2022-07-10
 */
#ifndef __SYLAR_BYTEARRAY_H__
#define __SYLAR_BYTEARRAY_H__

#include <memory>
#include <string>
#include <vector>
#include <stdint.h>
#include <sys/types.h>
#include <sys/socket.h>

namespace sylar
{

/**
 * @brief   二进制数组，提供基础类型的序列化和反序列化功能
 */
class ByteArray
{
public:
    typedef std::shared_ptr<ByteArray> ptr;

    /**
     * @brief  存储节点 
     */
    struct Node
    {
        /**
         * @brief  构造指定大小的内存块 
         *
         * @param   s   内存块字节数
         */
        Node(size_t s);
        Node();
        ~Node();

        /// 内存块地址指针
        char* ptr;
        /// 下一个内存块地址
        Node* next;
        /// 内存块大小
        size_t size;
    };

    /**
     * @brief   按指定大小的内存块构造
     *
     * @param   base_size   内存块大小
     */
    ByteArray(size_t base_size = 4096);
    ~ByteArray();

    /**
     * @brief  写入固定长度类型的整型数据 
     */
    void writeFint8(int8_t value);
    void writeFuint8(uint8_t value);
    void writeFint16(int16_t value);
    void writeFuint16(uint16_t value);
    void writeFint32(int32_t value);
    void writeFuint32(uint32_t value);
    void writeFint64(int64_t value);
    void writeFuint64(uint64_t value);

    /**
     * @brief  写入可压缩(Zigzag算法)的字符串数据 
     */
    void writeInt32(int32_t value);
    void writeUint32(uint32_t value);
    void writeInt64(int64_t value);
    void writeUint64(uint64_t value);
    
    void writeFloat(float value);
    void writeDouble(double value);

    /**
     * @brief  写入前面带长度(长度所占字节数固定)的字符串数据(长度+实际数据) 
     */
    void writeStringF16(const std::string& value);
    void writeStringF32(const std::string& value);
    void writeStringF64(const std::string& value);
    
    /**
     * @brief  写入前面带长度的数据(实际长度+实际数据) 
     *         长度所占字节数为可压缩的uint64的实际大小
     */
    void writeStringVint(const std::string& value);

    /**
     * @brief   写入不带长度的字符串数据
     */
    void writeStringWithoutLength(const std::string& value);

    int8_t readFint8();
    uint8_t readFuint8();
    int16_t readFint16();
    uint16_t readFuint16();
    int32_t readFint32();
    uint32_t readFuint32();
    int64_t readFint64();
    uint64_t readFuint64();
    
    int32_t readInt32();
    uint32_t readUint32();
    int64_t readInt64();
    uint64_t readUint64();

    float readFloat();
    double readDouble();

    std::string readStringF16();
    std::string readStringF32();
    std::string readStringF64();
    std::string readStringVint();

    void clear();
    void write(const void* buf, size_t size);
    void read(void* buf, size_t size);

    /**
     * @brief  读取size长度的数据 
     *
     * @param   position    读取开始的位置
     */
    void read(void* buf, size_t size, size_t position) const;
    
    /**
     * @brief  返回ByteArray的当前位置 
     */
    size_t getPosition() const { return m_position; }
    void setPosition(size_t v);
    
    /**
     * @brief  把ByteArray的数据写入到文件中 
     */
    bool writeToFile(const std::string& name) const;
    bool readFromFile(const std::string& name);
    
    /**
     * @brief  返回内存块的大小 
     */
    size_t getBaseSize() const { return m_baseSize; }
    
    /**
     * @brief   返回可读取数据的大小
     */
    size_t getReadSize() const { return m_size - m_position; }

    bool isLittleEndian() const;
    void setIsLittleEndian(bool val);

    std::string toString() const;
    std::string toHexString() const;

    /**
     * @brief   获取可读取的缓存，保存到iovec数组中 
     */
    uint64_t getReadBuffers(std::vector<iovec>& buffers, uint64_t len = ~0ull) const;
  
    /**
     * @brief   获取可读取的缓存，保存到iovec数组中，从position位置开始
     */
    uint64_t getReadBuffers(std::vector<iovec>& buffers, uint64_t len, uint64_t position) const;
    
    /**
     * @brief   获取可写入的缓存，保存到iovec数组中
     */
    uint64_t getWriteBuffers(std::vector<iovec>& buffers, uint64_t len);

    /**
     * @brief   返回数据的长度(当前总数据量)
     */
    size_t getSize() const { return m_size; }

private:
    /**
     * @brief   扩容ByteArray，使其可以容纳size个数据 
     */
    void addCapacity(size_t size);
    
    /**
     * @brief   获取当前的可写入容量
     */
    size_t getCapacity() const { return m_capacity - m_position; }

private:
    /// 内存块的大小 
    size_t m_baseSize;
    /// 当前操作位置
    size_t m_position;
    /// 当前的总容量
    size_t m_capacity;
    /// 当前总数据量
    size_t m_size;
    /// 字节序,默认大端
    int8_t m_endian;
    /// 第一个内存块的指针
    Node* m_root;
    /// 当前操作内存块的指针
    Node* m_cur;
};

}
#endif


#include "bytearray.h"
#include <fstream>
#include <sstream>
#include <string.h>
#include <iomanip>
#include <math.h>
#include "log.h"
#include "endian.h"

namespace sylar
{
static sylar::Logger::ptr g_logger = SYLAR_LOG_NAME("system");

ByteArray::Node::Node(size_t s)
    : ptr(new char[s])
    , next(nullptr)
    , size(s)
{

}

ByteArray::Node::Node()
    : ptr(nullptr)
    , next(nullptr)
    , size(0)
{

}

ByteArray::Node::~Node()
{
    if(ptr)
    {
        delete[] ptr;
    }
}

ByteArray::ByteArray(size_t base_size)
    : m_baseSize(base_size)
    , m_position(0)
    , m_capacity(base_size)
    , m_size(0)
    , m_endian(SYLAR_BIG_ENDIAN)
    , m_root(new Node(base_size))
    , m_cur(m_root)
{

}

ByteArray::~ByteArray()
{
    Node* tmp = m_root;
    while(tmp)
    {
        m_cur = tmp;
        tmp = tmp->next;
        delete m_cur;   // 调用Node的析构函数，里面再delete掉一个节点所占的char数组
    }
}

void ByteArray::writeFint8(int8_t value)
{
    write(&value, sizeof(value));
}

void ByteArray::writeFuint8(uint8_t value)
{
    write(&value, sizeof(value));
}

void ByteArray::writeFint16(int16_t value)
{
    if(m_endian != SYLAR_BYTE_ORDER)
    {
        value = byteswap(value);
    }
    write(&value, sizeof(value));
}

void ByteArray::writeFuint16(uint16_t value)
{
    if(m_endian != SYLAR_BYTE_ORDER)
    {
        value = byteswap(value);
    }
    write(&value, sizeof(value));
}

void ByteArray::writeFint32(int32_t value)
{
    if(m_endian != SYLAR_BYTE_ORDER)
    {
        value = byteswap(value);
    }
    write(&value, sizeof(value));
}

void ByteArray::writeFuint32(uint32_t value)
{
    if(m_endian != SYLAR_BYTE_ORDER)
    {
        value = byteswap(value);
    }
    write(&value, sizeof(value));
}

void ByteArray::writeFint64(int64_t value)
{
    if(m_endian != SYLAR_BYTE_ORDER)
    {
        value = byteswap(value);
    }
    write(&value, sizeof(value));
}

void ByteArray::writeFuint64(uint64_t value)
{
    if(m_endian != SYLAR_BYTE_ORDER)
    {
        value = byteswap(value);
    }
    write(&value, sizeof(value));
}

static uint32_t EncodeZigzag32(const int32_t& v)
{
    if(v < 0)
    {
        return ((uint32_t)(-v)) * 2 - 1;
    }
    else
    {
        return v * 2;
    }
}

static uint64_t EncodeZigzag64(const int64_t& v)
{
    if(v < 0)
    {
        return ((uint64_t)(-v)) * 2 - 1;
    }
    else
    {
        return v * 2;
    }
}

static int32_t DecodeZigzag32(const uint32_t& v)
{
    return (v >> 1) ^ -(v & 1);
}

static int64_t DecodeZigzag64(const uint64_t& v)
{
    return (v >> 1) ^ -(v & 1);
}

void ByteArray::writeInt32(int32_t value)
{
    writeUint32(EncodeZigzag32(value));
}

void ByteArray::writeUint32(uint32_t value)
{
    uint8_t tmp[5];
    uint8_t i = 0;
    while(value >= 0x80)
    {
        tmp[i++] = (value & 0x7F) | 0x80;
        value >>= 7;
    }
    tmp[i++] = value;
    write(tmp, i);
}

void ByteArray::writeInt64(int64_t value)
{
    writeUint64(EncodeZigzag64(value));
}

void ByteArray::writeUint64(uint64_t value)
{
    uint8_t tmp[10];
    uint8_t i = 0;
    while(value >= 0x80)
    {
        tmp[i++] = (value & 0x7F) | 0x80;
        value >>= 7;
    }
    tmp[i++] = value;
    write(tmp, i);
}

void ByteArray::writeFloat(float value)
{
    uint32_t v;
    memcpy(&v, &value, sizeof(value));  // 当作固定长度的uint32_t写入
    writeFuint32(v);
}

void ByteArray::writeDouble(double value)
{
    uint64_t v;
    memcpy(&v, &value, sizeof(value));  // 当作固定长度的uint64_t写入
    writeFuint64(v);
}

void ByteArray::writeStringF16(const std::string& value)
{
    writeFuint16(value.size());
    write(value.c_str(), value.size());
}

void ByteArray::writeStringF32(const std::string& value)
{
    writeFuint32(value.size());
    write(value.c_str(), value.size());
}

void ByteArray::writeStringF64(const std::string& value)
{
    writeFuint64(value.size());
    write(value.c_str(), value.size());
}

void ByteArray::writeStringVint(const std::string& value)
{
    writeUint64(value.size());
    write(value.c_str(), value.size());
}

void ByteArray::writeStringWithoutLength(const std::string& value)
{
    write(value.c_str(), value.size());
}

int8_t ByteArray::readFint8()
{
    int8_t v;
    read(&v, sizeof(v));
    return v;
}

uint8_t ByteArray::readFuint8()
{
    uint8_t v;
    read(&v, sizeof(v));
    return v;
}

#define XX(type) \
    type v; \
    read(&v, sizeof(v)); \
    if(m_endian == SYLAR_BYTE_ORDER) \
    { \
        return v; \
    } \
    else \
    { \
        return byteswap(v); \
    }

int16_t ByteArray::readFint16()
{
    XX(int16_t);
}

uint16_t ByteArray::readFuint16()
{
    XX(uint16_t);
}

int32_t ByteArray::readFint32()
{
    XX(int32_t);
}

uint32_t ByteArray::readFuint32()
{
    XX(uint32_t);
}

int64_t ByteArray::readFint64()
{
    XX(int64_t);
}

uint64_t ByteArray::readFuint64()
{
    XX(uint64_t);
}

#undef XX

int32_t ByteArray::readInt32()
{
    return DecodeZigzag32(readUint32());
}

uint32_t ByteArray::readUint32()
{
    uint32_t result = 0;
    for(int i=0; i<32; i+=7)
    {
        uint8_t b = readFuint8();
        if(b < 0x80)
        {
            result |= ((uint32_t)b) << i;
            break;
        }
        else
        {
            result |= (((uint32_t)(b & 0x7f)) << i);
        }
    }
    return result;
}

int64_t ByteArray::readInt64()
{
    return DecodeZigzag64(readUint64());
}

uint64_t ByteArray::readUint64()
{
    uint64_t result = 0;
    for(int i=0; i<64; i+=7)
    {
        uint8_t b = readFuint8();
        if(b < 0x80)
        {
            result |= ((uint64_t)b) << i;
            break;
        }
        else
        {
            result |= (((uint64_t)(b & 0x7f)) << i);
        }
    }
    return result;
}

float ByteArray::readFloat()
{
    uint32_t v = readFuint32();
    float value;
    memcpy(&value, &v, sizeof(v));
    return value;
}

double ByteArray::readDouble()
{
    uint64_t v = readFuint64();
    double value;
    memcpy(&value, &v, sizeof(v));
    return value;
}

std::string ByteArray::readStringF16()
{
    uint16_t len = readFuint16();    // 先把长度读出来，再读实际数据(字符串)
    std::string buff;
    buff.resize(len);
    read(&buff[0], len);
    return buff;
}

std::string ByteArray::readStringF32()
{
    uint32_t len = readFuint32();
    std::string buff;
    buff.resize(len);
    read(&buff[0], len);
    return buff;
}

std::string ByteArray::readStringF64()
{
    uint64_t len = readFuint64();
    std::string buff;
    buff.resize(len);
    read(&buff[0], len);
    return buff;
}

std::string ByteArray::readStringVint()
{
    uint64_t len = readUint64();
    std::string buff;
    buff.resize(len);
    read(&buff[0], len);
    return buff;
}

void ByteArray::clear()
{
    m_position = m_size = 0;
    m_capacity = m_baseSize;
    Node* tmp = m_root->next;   // 保留根节点
    while(tmp)
    {
        m_cur = tmp;
        tmp = tmp->next;
        delete m_cur;
    }
    m_cur = m_root;
    m_root->next = NULL;
}

void ByteArray::write(const void* buf, size_t size)
{
    if(size == 0)
    {
        return;
    }
    addCapacity(size);      // 先根据需要扩充容量

    size_t npos = m_position % m_baseSize;  // 当前内存块所用的字节数
    size_t ncap = m_cur->size - npos;       // 当前内存块所剩下的字节数
    size_t bpos = 0;            // 已经分给使用者的内存字节数
    while(size > 0)
    {
        if(ncap >= size)  // 当前内存块所剩下的内存够用
        {
            memcpy(m_cur->ptr + npos, (const char*)buf + bpos, size);
            if(m_cur->size == (npos + size)) // 当前内存块已用完，更新当前可用内存块位置
            {
                m_cur = m_cur->next;
            }
            m_position += size;     // 当前位置变更
            bpos += size;           // 分配给buf的位置更新
            size = 0;               // 所要的内存字节数变0，也就是满足需求了
        }
        else
        {
            memcpy(m_cur->ptr + npos, (const char*)buf + bpos, ncap); // 先分完当前的内存块
            m_position += ncap;     // 当前位置变更，加上已经分的字节数
            bpos += ncap;           // 已经分给使用者ncap个字节
            size -= ncap;           // 需求量减少ncap个字节
            m_cur = m_cur->next;    // 当前内存块已经用完，更新当前可用内存块
            ncap = m_cur->size;     // 下一个内存块的所剩下字节数，也就是下一块内存块的大小
            npos = 0;               // 下一个内存块是刚刚才next到的，所用字节数肯定是0
        }
    }

    // 当前位置比当前使用内存量大，则更新当前内存使用量
    if(m_position > m_size)
    {
        m_size = m_position;
    }
}

void ByteArray::read(void* buf, size_t size)
{
    // 需要读取的比可读取的还要大，则认为异常
    if(size > getReadSize())
    {
        throw std::out_of_range("not enough len");
    }

    // 下面读取的代码和写入的代码的区别只是在memcpy的时候，
    // 读取时 memcpy源地址和目的地址 与 写入时调转
    size_t npos = m_position % m_baseSize;
    size_t ncap = m_cur->size - npos;
    size_t bpos = 0;
    while(size > 0)
    {
        if(ncap >= size)
        {
            memcpy((char*)buf + bpos, m_cur->ptr + npos, size);
            if(m_cur->size == (npos + size))
            {
                m_cur = m_cur->next;
            }
            m_position += size;
            bpos += size;
            size = 0;
        }
        else
        {
            memcpy((char*)buf + bpos, m_cur->ptr + npos, ncap);
            m_position += ncap;
            bpos += ncap;
            size -= ncap;
            m_cur = m_cur->next;
            ncap = m_cur->size;
            npos = 0;
        }
    }
}

void ByteArray::read(void* buf, size_t size, size_t position) const
{
    // 总使用量减去要读取的位置，也就是能读到的字节数。
    // 需要读取的比能读到的要大，则认为异常。
    if(size > (m_size - position))
    {
        throw std::out_of_range("not enough len");
    }

    // 下面代码和不指定位置读取的代码的区别是：
    // 1、不使用m_position；2、没有副作用，不直接改m_cur
    size_t npos = position % m_baseSize;    
    size_t ncap = m_cur->size - npos;
    size_t bpos = 0;
    Node* cur = m_cur;      // 无副作用，不影响原有数据结构
    while(size > 0)
    {
        if(ncap >= size)
        {
            memcpy((char*)buf + bpos, cur->ptr + npos, size);
            if(cur->size == (npos + size))
            {
                cur = cur->next;
            }
            position += size;
            bpos += size;
            size = 0;
        }
        else
        {
            memcpy((char*)buf + bpos, cur->ptr + npos, ncap);
            position += ncap;
            bpos += ncap;
            size -= ncap;
            cur = cur->next;
            ncap = cur->size;
            npos = 0;
        }
    }
}

void ByteArray::setPosition(size_t v)
{
    // 要设置的当前位置比容量还大，则认为异常
    if(v > m_capacity)
    {
        throw std::out_of_range("set_position out of range");
    }
    m_position = v;
    
    if(m_position > m_size)
    {
        m_size = m_position;
    }

    // 因为当前位置变了，那么指向当前内存块的指针也要变
    m_cur = m_root;
    while(v > m_cur->size)
    {
        v -= m_cur->size;
        m_cur = m_cur->next;
    }
    if(v == m_cur->size)
    {
        m_cur = m_cur->next;
    }
}

bool ByteArray::writeToFile(const std::string& name) const
{
    std::ofstream ofs;
    // trunc覆盖写，binary以二进制模式打开
    ofs.open(name, std::ios::trunc | std::ios::binary);
    if(!ofs)
    {
        SYLAR_LOG_ERROR(g_logger) << "writeToFile name=" << name
            << " error, errno=" << errno << " errstr=" << strerror(errno);
        return false;
    }

    int64_t read_size = getReadSize();
    int64_t pos = m_position;
    Node* cur = m_cur;
    while(read_size > 0)
    {
        int diff = pos % m_baseSize;    // 当前块不可读(已经读完)的字节数

        // 1、要读的内容比一整块要大，则读完一整块剩下的那些
        // 2、要读的内容比一整块少，则读？？？
        // 假设一块10字节，可读9字节，不可读是3字节，则读9-3=6字节，bug???
        // 假设一块10字节，可读1字节，不可读是3字节，则读1-3=-5字节，bug???
        // 应该是：
        // int64_t len = 0;
        // if(read_size > (int64_t)m_baseSize - diff)
        //      len = (int64_t)m_baseSize - diff;
        // else
        //      len = read_size;
        int64_t len = (read_size > (int64_t)m_baseSize ? m_baseSize : read_size) - diff;
        ofs.write(cur->ptr + diff, len);
        cur = cur->next;
        pos += len;
        read_size -= len;
    }

    return true;
}

bool ByteArray::readFromFile(const std::string& name)
{
    std::ifstream ifs;
    ifs.open(name, std::ios::binary);
    if(!ifs)
    {
        SYLAR_LOG_ERROR(g_logger) << "readFromFile name=" << name
            << " error, errno=" << errno << " errstr=" << strerror(errno);
        return false;
    }

    std::shared_ptr<char> buff(new char[m_baseSize], [](char* ptr) { delete[] ptr; });
    while(!ifs.eof())
    {
        ifs.read(buff.get(), m_baseSize);   // 每次读取一块
        write(buff.get(), ifs.gcount());    // gcount()返回最近无格式输入操作所提取的字符数
    }
    return true;
}

bool ByteArray::isLittleEndian() const
{
    return m_endian == SYLAR_LITTLE_ENDIAN;
}

void ByteArray::setIsLittleEndian(bool val)
{
    if(val)
    {
        m_endian = SYLAR_LITTLE_ENDIAN;
    }
    else
    {
        m_endian = SYLAR_BIG_ENDIAN;
    }
}

std::string ByteArray::toString() const
{
    std::string str;
    str.resize(getReadSize());
    if(str.empty())
    {
        return str;
    }
    read(&str[0], str.size(), m_position);
    return str;
}

std::string ByteArray::toHexString() const
{
    std::string str = toString();
    std::stringstream ss;
    for(size_t i=0; i<str.size(); ++i)
    {
        if(i > 0 && i % 32 == 0)
        {
            ss << std::endl;
        }
        ss << std::setw(2) << std::setfill('0') << std::hex
           << (int)(uint8_t)str[i] << " ";
    }
    return ss.str();
}

uint64_t ByteArray::getReadBuffers(std::vector<iovec>& buffers, uint64_t len) const
{
    len = len > getReadSize() ? getReadSize() : len;
    if(len == 0)
    {
        return 0;
    }

    uint64_t size = len;
    size_t npos = m_position % m_baseSize;
    size_t ncap = m_cur->size - npos;
    struct iovec iov;
    Node* cur = m_cur;

    while(len > 0)
    {
        if(ncap >= len)
        {
            iov.iov_base = cur->ptr + npos;
            iov.iov_len = len;
            len = 0;
        }
        else
        {
            iov.iov_base = cur->ptr + npos;
            iov.iov_len = ncap;
            len -= ncap;
            cur = cur->next;
            ncap = cur->size;
            npos = 0;
        }
        buffers.push_back(iov);
    }
    return size;
}

uint64_t ByteArray::getReadBuffers(std::vector<iovec>& buffers, uint64_t len, uint64_t position) const
{
    // 这里的判断好像没啥用，因为下面用的position是外面传进来的，
    // 与m_position是没有关系的，bug???
    len = len > getReadSize() ? getReadSize() : len;
    if(len == 0)
    {
        return 0;
    }

    uint64_t size = len;
    size_t npos = position % m_baseSize;
    size_t count = position / m_baseSize;
    Node* cur = m_root;
    while(count > 0)
    {
        cur = cur->next;
        --count;
    }
    
    size_t ncap = cur->size - npos;
    struct iovec iov;
    while(len > 0)
    {
        if(ncap >= len)
        {
            iov.iov_base = cur->ptr + npos;
            iov.iov_len = len;
            len = 0;
        }
        else
        {
            iov.iov_base = cur->ptr + npos;
            iov.iov_len = ncap;
            len -= ncap;
            cur = cur->next;
            ncap = cur->size;
            npos = 0;
        }
        buffers.push_back(iov);
    }
    return size;
}

uint64_t ByteArray::getWriteBuffers(std::vector<iovec>& buffers, uint64_t len)
{
    if(len == 0)
    {
        return 0;
    }
    addCapacity(len);

    uint64_t size = len;
    size_t npos = m_position % m_baseSize;
    size_t ncap = m_cur->size - npos;
    struct iovec iov;
    Node* cur = m_cur;

    while(len > 0)
    {
        if(ncap >= len)
        {
            iov.iov_base = cur->ptr + npos;
            iov.iov_len = len;
            len = 0;
        }
        else
        {
            iov.iov_base = cur->ptr + npos;
            iov.iov_len = ncap;
            len -= ncap;
            cur = cur->next;
            ncap = cur->size;
            npos = 0;
        }
        buffers.push_back(iov);
    }
    return size;
}

void ByteArray::addCapacity(size_t size)
{
    if(size == 0)
    {
        return;
    }

    size_t old_cap = getCapacity();
    if(old_cap >= size)     // 剩余容量还够，无需扩充
    {
        return;
    }

    size = size - old_cap;  // 得到需要扩充的容量大小
    // ceil(x) 返回大于或者等于x的最小整数
    size_t count = ceil(1.0 * size / m_baseSize);   // 得到需要扩充的块数
    
    // 遍历得到最后一个节点
    Node* tmp = m_root;
    while(tmp->next)
    {
        tmp = tmp->next;
    }

    Node* first = NULL;     // 记录扩充的第一块内存块的位置
    for(size_t i=0; i<count; ++i)
    {
        tmp->next = new Node(m_baseSize);
        if(first == NULL)
        {
            first = tmp->next;
        }
        tmp = tmp->next;
        m_capacity += m_baseSize;   // 容量加上一个内存块
    }

    // 如果剩余容量已经是0了，那么扩充的第一块内容块就是当前可用内存块所在的位置
    if(old_cap == 0)        
    {
        m_cur = first;
    }
}

}
```


# 广告时间：基于sylar框架实现的小demo(希望给个star)

> [基于redis的参数查询服务](https://github.com/L-ge/awesome-sylar/tree/main/projects/paramquery)
