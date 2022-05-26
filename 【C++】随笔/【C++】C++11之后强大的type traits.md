在我自己写的类没有继承任何库里面的类或者组合任何库里面的类的情况下，惊讶于C++11它知道我自己写的类的那么多"隐私"。

下面，直接看示例代码和输出结果。
```
#include <QCoreApplication>
#include <QDebug>
#include <functional>

class TestTypeTraits
{
public:
    TestTypeTraits(int nValue) : m_nValue(nValue) { }
    TestTypeTraits(TestTypeTraits&&) = default;
    TestTypeTraits& operator=(const TestTypeTraits&&) = delete;
    virtual ~TestTypeTraits(){ }
private:
    int m_nValue;
};

template<typename T>
void typeTraitsOutput(const T& x)
{
    qDebug() << " typeid.name = " << typeid(T).name();
    qDebug() << " is_class? " << std::is_class<T>::value;
    qDebug() << " is_polymorphic? " << std::is_polymorphic<T>::value;
    qDebug() << " is_default_constructible? " << std::is_default_constructible<T>::value;
    qDebug() << " is_move_constructible? " << std::is_move_constructible<T>::value;
    qDebug() << " is_move_assignable? " << std::is_move_assignable<T>::value;

    // is_trivial的意思是：是不重要的吗？
    qDebug() << " is_trivial? " << std::is_trivial<T>::value;
}

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    typeTraitsOutput(TestTypeTraits(1));

    qDebug() << "=============================";

    typeTraitsOutput(int(1));

    return a.exec();
}
```
```
 // 下面是输出结果
 typeid.name =  14TestTypeTraits
 is_class?  true
 is_polymorphic?  true
 is_default_constructible?  false
 is_move_constructible?  true
 is_move_assignable?  false
 is_trivial?  false
=============================
 typeid.name =  i
 is_class?  false
 is_polymorphic?  false
 is_default_constructible?  true
 is_move_constructible?  true
 is_move_assignable?  true
 is_trivial?  true
```

它是怎么做到的呢？

例如const属性，是通过模板的偏特化去实现的。

```
// 相关源代码在.\include\c++\tr1\type_traits

// remove_const
template<typename _Tp>
struct remove_const
{
    typedef _Tp type;
}
template<typename _Tp>
struct remove_const<_Tp const>
{
    typedef _Tp type;
}

// is_const
template<typename>
struct is_const
    : public false_type { };

template<typename _Tp>
struct is_const<_Tp const>
    : public true_type { };
```

但一个东西是否is_class、is_enum等，可能不是由偏特化实现的，可能是从编译器得到的。