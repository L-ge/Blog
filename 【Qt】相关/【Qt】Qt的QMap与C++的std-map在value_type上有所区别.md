通过源码或示例去说明三个知识点：

①QMap的value_type是T，也就是key-value中的value；

②std::map的value_type是pair<const _Key, _Tp>，也就是由key和value组成的pair；

③如何使用for-rangement(for的范围循环)和std::for_each循环去遍历QMap和std::map.

```
// QMap源码片段
template <class Key, class T>
class QMap
{
    ...
    class iterator
    {
        ...
    public:
        typedef std::bidirectional_iterator_tag iterator_category;
        typedef qptrdiff difference_type;
        typedef T value_type;
        typedef T *pointer;
        typedef T &reference;

        ...
    };
    ...
};
```

```
// STL的map源代码片段
template <class _Key, class _Tp, class _Compare, class _Alloc>
class map {
public:
  ...

  typedef _Key                  key_type;
  typedef _Tp                   data_type;
  typedef _Tp                   mapped_type;
  typedef pair<const _Key, _Tp> value_type;
  typedef _Compare              key_compare;
    
  ...
};
```

```
// 测试Demo
#include <QCoreApplication>
#include <QMap>
#include <map>
#include <QDebug>
#include <iostream>
#include <algorithm>

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    QMap<int, int> qMap;
    qMap[1] = 10;
    qMap[2] = 20;
    qMap[3] = 30;

    std::map<int, int> stdMap;
    stdMap[1] = 10;
    stdMap[2] = 20;
    stdMap[3] = 30;

    for(auto& item : qMap)
    {
        //std::cout << item.first; // 编译报错:request for member 'first' in 'item', which is of non-class type 'int'
    }

    for(auto& item : stdMap)
    {
        qDebug() << item.first;     // 逐行输出1 2 3
    }

    std::for_each(qMap.cbegin(), qMap.cend(), [](auto& item)    // C++14在lambda支持auto
    {
        qDebug() << item;           // 逐行输出10 20 30
    });

    std::for_each(stdMap.cbegin(), stdMap.cend(), [](auto& item)     // C++14在lambda支持auto
    {
        qDebug() << item;           // 逐行输出std::pair(1,10) std::pair(2,20) std::pair(3,30)
    });


    return a.exec();
}
```
