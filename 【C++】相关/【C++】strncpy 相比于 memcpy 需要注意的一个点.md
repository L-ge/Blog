1. 结论：当 strncpy 函数的 src 参数的字符串长度小于指定的长度 n 时，strncpy 函数将会在 dest 后面补 0，而不是像 memcpy 那样强制复制 src 字符串后面的 n 个字符。

2. 测试代码：
```cpp
#include <QCoreApplication>
#include <QDebug>

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    QByteArray ba;
    ba.append('a');
    ba.append(1, 0x00);
    ba.append('b');

    QString qstr = QString(ba);
    std::string sstr = ba.toStdString();

    std::vector<char> buffer1(10, 0);
    strcpy(buffer1.data(), sstr.c_str());

    std::vector<char> buffer2(10, 0);
    strncpy(buffer2.data(), sstr.c_str(), sstr.size());

    std::vector<char> buffer3(10, 0);
    memcpy(buffer3.data(), sstr.c_str(), sstr.size());

    qDebug() << "Finish Test.";

    return a.exec();
}
```
- 打断点调试时，可以看到 buffer1[2] 是 '\0'，buffer2[2] 也是 '\0'，而 buffer3[2] 是 'b'。

3. 文档说明：
```
$ man 3 strncpy
STRCPY(3)                  Linux Programmer's Manual                 STRCPY(3)

NAME
       strcpy, strncpy - copy a string

SYNOPSIS
       #include <string.h>

       char *strcpy(char *dest, const char *src);

       char *strncpy(char *dest, const char *src, size_t n);

DESCRIPTION
       The  strcpy()  function  copies the string pointed to by src, including
       the terminating null byte ('\0'), to the buffer  pointed  to  by  dest.
       The  strings  may  not overlap, and the destination string dest must be
       large enough to receive the copy.  Beware  of  buffer  overruns!   (See
       BUGS.)

       The  strncpy()  function is similar, except that at most n bytes of src
       are copied.  Warning: If there is no null byte among the first n  bytes
       of src, the string placed in dest will not be null-terminated.

       If  the  length of src is less than n, strncpy() writes additional null
       bytes to dest to ensure that a total of n bytes are written.
setJointAngle
       A simple implementation of strncpy() might be:

           char *
           strncpy(char *dest, const char *src, size_t n)
           {
               size_t i;

               for (i = 0; i < n && src[i] != '\0'; i++)
                   dest[i] = src[i];
               for ( ; i < n; i++)
                   dest[i] = '\0';

               return dest;
           }

RETURN VALUE
       The strcpy() and strncpy() functions return a pointer to  the  destina‐
       tion string dest.
...
```
- 注意“If  the  length of src is less than n, strncpy() writes additional null bytes to dest to ensure that a total of n bytes are written.” 这句话。
