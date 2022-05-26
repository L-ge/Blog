1、STL六大部件：容器、分配器、算法、迭代器、适配器、仿函数。

2、begin()指向第一个元素，end()函数指向最后一个元素的下一个位置。

迭代器概念上讲就是泛化的指针。
```
Container<T> c;
...
写法①：
Container<T>::iterator ite = c.begin();
for(; ite!=c.end(); ++ite)
    ...

写法②(since C++11)：
for(int i : { 2, 3, 5, 7, 9 })
    ...

写法③(since C++11)：
std::vector<double> vec;
...
for(auto elem : vec)
    ...
for(auto& elem : vec)   // 拿到是元素的引用
    ...
```

3、容器不一定是连续的空间。

4、容器分为序列式容器、关联式容器。
- 序列式容器：Array-数组、Vector-可变大小数组、Deque-双向队列、List-双向链表、Forward-List-单向链表。
- 关联式容器：Set-元素不能重复/MultiSet-元素可重复、Map-key不可重复/Multimap-key可重复。
- 无序的容器：哈希表实现。
- 红黑树：高度平衡的二叉树。Set和Map一般用红黑树实现。

5、C++11支持std::array<int, 10> arr;这种形式的数组。

6、大容量的元素插入，可能会导致抛出std::bad_alloc异常。因此最好要加try-catch.
```
try
{
    ...
}
catch(exception& e)
{
    std::cout << e.what();
}
```

7、vector扩展的方式是2倍扩展(其实不一定，要看环境，有些可能是1.5倍)。

8、list容器有max_size()函数，它不是根据内存大小决定的吗？？

9、当容器(例如list)自己有sort函数的时候，应该用自带的，而不是用STL的(全局的))。
```
lst.sort();
::sort(lst.begin(), lst.end());
```

10、forward_list和slist是一样的用途。

`slist是在头文件#include <ext/slist>中，用法是__gnu_cxx::slist<std::string> c;.`

11、stack(先进后出)和queue(先进先出)基于deque实现，但没有提供iterator操作。

12、deque的内存结构其实一段段的(每一段的指针放在一个vector中)，一段称为一个buffer，一个buffer存放多个元素(GNU2.9可以指定放多少个元素、GNU4.9.3就已经不能指定大小了)。

deque表面是连续的，实质是分段连续。

13、关联式容器：元素插入比较慢，但查找非常快。

14、multimap不可以使用[]做插入。
```
multimap<long, string> c;
c.insert(pair<long, string>(i, buf));

auto pItem = c.find(i);
std::cout << (*pItem).second;
```

15、pair这个结构是没有重载<<运算符的，因此不能直接输出整个元素。

16、unordered_set/unordered_multiset、unordered_map/unordered_multimap用哈希表实现。哈希表的篮子的个数一定比元素多。

17、容器利用分配器来支持它对内存的使用。

18、Qt Creator测试以下程序-Based on Qt 5.7.0 (MSVC 2013, 32 bit)，编译器是mingw530_32。
```
qHash不支持string？
    {
        using namespace std;
        set<int> setTmp;
        clock_t timeStart = clock();
        for(int i=0; i<10000000; i++)
            setTmp.insert(i);
        cout << "set msecs: " << (clock()-timeStart) << endl;   // 输出 9034
    }

    {
        using namespace std;
        QSet<int> setTmp;
        clock_t timeStart = clock();
        for(int i=0; i<10000000; i++)
            setTmp.insert(i);
        cout << "QSet msecs: " << (clock()-timeStart) << endl;  // 输出 1233
    }

    {
        using namespace std;
        map<int, int> mapTmp;
        clock_t timeStart = clock();
        for(int i=0; i<10000000; i++)
            mapTmp[i] = i;
        cout << "map msecs: " << (clock()-timeStart) << endl;   // 输出 9853
    }

    {
        using namespace std;
        QMap<int, int> mapTmp;
        clock_t timeStart = clock();
        for(int i=0; i<10000000; i++)
            mapTmp[i] = i;
        cout << "QMap msecs: " << (clock()-timeStart) << endl;   // 输出 5976
    }
```

19、使用分配器(不建议直接使用分配器)：
```
int* p;
allocator<int> alloc1;
p = alloc1.allocato(1);     // 1是一个元素的意思.
alloc1.deallocate(p, 1);
```
而malloc的时候要指定大小，free的时候却是不用。

20、泛型编程将容器和算法分开来，两者通过Iterator(迭代器)沟通。

`在泛型编程中，操作符重载扮演着非常重要的角色。`

21、所有算法其内最终涉及元素本身的操作，无非就是比较大小。

`例如，是否相等，也就是a不比b大且a不比b小，那就是等于。`

22、完整特化-全特化；局部特化-偏特化。

`除了个数上的偏特化，也可以在范围上偏特化，例如，T偏特化为T*或const T*.`

23、下面是C++ Builder 6.0的new.h的源代码，operator new里面用到了malloc函数。
```
#ifdef __BORLANDC__
// Prototypes for the standard global new & delete operators
void * _RTLENTRY _EXPFUNC operator new (size_t);
void   _RTLENTRY _EXPFUNC operator delete (void *);

// inline versions of the nothrow_t versions of new & delete operators
inline void * _RTLENTRY operator new (size_t size, const std::nothrow_t &)
{
    size = size ? size : 1;
    return malloc(size);
}
```

24、VC6和BorlandC++和GCC2.9的allocator都只是以::operator new和::operator delete完成allocate()和deallocate()，没有任何特殊设计。

25、每次malloc，都会有一些额外的内存开销(Cookie，记录着这块内存的大小)。例如，要分配100万个元素，就有100万次额外的开销，因此分配器的实现非常重要。

容器里面每个元素的大小都是一样的，新增一个元素都会用cookie来记录这块内存的大小是没有必要的。

26、下面是gcc2.9的defalloc.h的源代码(但它自己没有用这个，它用了另外一个alloc，在stl_alloc.h文件中).
```
// DO NOT USE THIS FILE unless you have an old container implementation
// that requires an allocator with the HP-style interface.  
//
// Standard-conforming allocators have a very different interface.  The
// standard default allocator is declared in the header <memory>.
template <class T>
inline T* allocate(ptrdiff_t size, T*) {
    set_new_handler(0);
    T* tmp = (T*)(::operator new((size_t)(size * sizeof(T))));
    if (tmp == 0) {
	cerr << "out of memory" << endl; 
	exit(1);
    }
    return tmp;
}


template <class T>
inline void deallocate(T* buffer) {
    ::operator delete(buffer);
}
```

27、gcc2.9在stl_alloc.h文件中有更好的分配器alloc(大概就是16条链表，每个链表挂着一串相同大小的内存块)。
- 但是在gcc4.9用的默认分配器还是用回了gcc2.9defalloc.h的形式，定义在_gnu_cxx::new_allocator.
- gcc4.9在gcc2.9的stl_alloc.h文件中的那种分配器alloc定义在_gnu_cxx::_pool_alloc.

28、注意以下这种情况是拷贝构造，不是拷贝赋值。

其实这里是唤醒拷贝构造函数用以创建tmp并以*this为初值。

self tmp = *this;   // 这种是拷贝构造.

29、C++ 不允许 i++++ 这种操作，即不允许后 ++ 两次。（但是允许 ++++i ，所以迭代器重载的前 ++ 版本返回值是引用）

self operator++(int){ ... } // 所以迭代器重载的后++版本返回值是值，而不是引用。

30、list的大小在2.9版本是4(一个指针，指向一个空虚的节点)，在4.9版本是8(两个指针，分别指向2.9版本里面那个空虚节点的前后指针).

31、Iterator一定会提供5种相应的类型associated types。可以看看源代码<stl_iterator.h>。
```
以list举例:
template<class T, class Ref, class Ptr>
struct __list_iterator
{
    typedef bidirectional_itetator_tag iterator_category;   // 双向链表的意思
    typedef T value_type;
    typedef Ptr pointer;
    typedef Ref reference;
    typedef ptrdiff_t different_type; 
    ...
}

下面是iterators的三个associated types:
iterator_traits<_Iter>::iterator_category               // 分类，Iterator的移动性质(有的迭代器可以++、有的可以跳着走等等)
iterator_traits<RandomAccessIterator>::different_type   // 两个Iterator的距离
iterator_traits<RandomAccessIterator>::value_type       // Iterator指向的元素的类型
```

32、non-class iterators亦即native pointers，无法定义associated types。

class iterators都有能力定义自己的associated types.

所以有一个中介层(iterator_traits)用以分离class iterators和non-class iterators. 

33、value_type的主要目的是用来声明变量，而声明一个无法被赋值的变量没有用，所以iterator(即便是constant iterator)的value type不应加上const。

iterator若是const int*，其value_type应该是int，而非const int.

34、一个vector的大小是12，存放着三个指针(指向头start、指向最后一个元素下一个位置finish、指向尾end_of_range)。

35、public继承表现出的是is-a的关系。

36、只要是连续空间的容器(例如array、vector)，它的迭代器就可以用单纯的指针来表现，不需要另外实现class。

deque的迭代器是class.

37、一个deque的大小是40.(GNU2.9版本和GNU4.9版本)

38、queue(先进先出)和stack(先进后出)是默认选择deque作为底层的，其实也可以选择list。
- stack或queue都不允许遍历，也不提供iterator。
- stack可以选择vector作为底层结构。
- queue不可选择vector作为底层结构，因为vector没有pop_front()函数。
- stack或queue都不可选择set或map作为底层结构。

编译器不会对模板进行全面的检查，是用多少，检查多少。如果queue没有用到pop函数(也即vector的pop_front()函数)，则是可以使用的，编译器不会报错。

39、红黑树是平衡二分搜寻树。
- 平衡二分搜寻树的特征：排列规则有利于查找和插入。
- 红黑树按正常规则遍历，便能获得排序状态。
- 我们不应使用红黑树的iterators改变元素值(因为元素有其严谨排列规则)。编程层面并未阻绝此事。如此设计是正确的，因为红黑树即将为set和map服务(作为其底部支持)，而map支持元素的data被改变，只有元素的key才是不可被改变的。

40、编译器对于大小为0的类，创造出来对象的大小是1.

41、面向对象有一种思想是 在一个class里面有一个指针(或者一个独立的东西)来表现它的实现手法。

`这个class基本什么都没有做，主要的实现都在它那个指针(或者一个独立的东西)的类中。`

42、set/multiset/map/multimap以rb-tree为底层结构，因此有元素自动排序特性。

`排序的依据是key。按正常规则(++ite)遍历，便能获得排序状态。`

43、我们无法使用set/multiset的iterators改变元素值。

`set/multiset的iterator是其底部RB tree的const_iterator，就是为了禁止user对元素赋值。`

44、我们无法使用map/multimap的iterators改变元素的key，但可以用它来改变元素的data。

`因此，map/multimap内部自动将user指定的key_type设为const，如此便能禁止user对元素的key赋值。`

45、set/map元素的key必须独一无二，因此其insert()用的是rb_tree的inser_unique();

`multiset/multimap元素的key可以重复，因此其insert()用的是rb_tree的inser_equal().`

46、multimap不可以使用[]做insert。

47、map的[]插入方法比insert慢一点点，因为[]插入会先调用lower_bound方法找到元素插入的位置，然后再调用insert进行插入。
```
mapTest.insert(pair<int, string>(i, buf));
mapTest[i] = buf;
```

48、hashtable当元素的个数等于篮子的个数bucket_count时，就会打散rehashing(GNU C2.9一开始的篮子是53，然后是97,193,389...，但在4.9的时候已经不是这样了)。

49、hash funciton的目的就是希望根据元素值算出一个hash code（一个可进行modulus运算的值），使得元素经hash code映射之后能够很杂乱很随机地放进hashtable内。越是杂乱，越不容易发生碰撞。

50、打印类型的名称。
```
#include <typeinfo>
typeid(itr).name(); // 编译器给类定义的名字，比如一个类叫 ABC, 则可能打印 xxxABCxxxxxx
```

51、 下面几个分类具有继承的关系：
```
random_access_iterator_tag继承自bidirectional_access_iterator_tag、
bidirectional_access_iterator_tag继承自forward_access_iterator_tag、
forward_access_iterator_tag继承自input_access_iterator_tag.
其中，
random_access_iterator_tag、        // 随机访问的
bidirectional_access_iterator_tag、 // 双向链表
forward_access_iterator_tag、       // 单向链表
input_access_iterator_tag、         // 输入
```

52、memmove函数：
```
/* memmove example */
#include <stdio.h>
#include <string.h>
int main ()
{
  char str[] = "1234567890";
  memmove (str+5,str+2,1);
  puts (str);       // 1234537890
  return 0;
}
```

53、函数对象(仿函数functors)，它是一个类或者一个结构体，只为算法而服务:
```
struct myClass	// 这里如果没有继承就没有融入到STL
{
    bool operator()(int x, int y) { return (i<j); }	// 必须是重载这个运算符，()称为function call operator
} myObj;

sort(vec.begin(), vec.end(), myObj);

template<class T>
struct myClass : public binary_function<T, T, bool>		// binary_function类里面其实只有三个typedef.
{
    bool operator()(const T& x, const T& y) { return x<y; }
};
```

54、list和forward_list容器带有sort成员函数。

list和forward_list不要去调用算法的sort函数，因为算法的sort函数仅支持random_access_iterator。

55、{10、20、20、20、30}，用lower_bound找20，找到的是第二个元素的位置。用upper_bound找20，找到的是第四个元素的位置。

56、没有数据的类，理论上大小应该为0，其实是1。但当它作为父类的时候，它的大小就是0.
```
class TestSize
{
typedef QString MyString;
public:
    void show(){}
    void showImp(){}
};

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    qDebug() << "size of TestSize:" << sizeof(TestSize);	// 输出大小是1

    return a.exec();
}
```

57、类模板是没有实参推导的，函数模板有。

58、typename关键字，告诉编译器这是一个类型。

59、.\mingw530_32\i686-w64-mingw32\include\c++\backward\backward_warning.h有以下内容，告诉大家哪些东西是已经过时的了，现在用的是啥。
```
/*
  A list of valid replacements is as follows:

  Use:					Instead of:
  <sstream>, basic_stringbuf	   	<strstream>, strstreambuf
  <sstream>, basic_istringstream	<strstream>, istrstream
  <sstream>, basic_ostringstream	<strstream>, ostrstream
  <sstream>, basic_stringstream		<strstream>, strstream
  <unordered_set>, unordered_set     	<ext/hash_set>, hash_set
  <unordered_set>, unordered_multiset	<ext/hash_set>, hash_multiset
  <unordered_map>, unordered_map	<ext/hash_map>, hash_map
  <unordered_map>, unordered_multimap	<ext/hash_map>, hash_multimap
  <functional>, bind			<functional>, binder1st
  <functional>, bind			<functional>, binder2nd
  <functional>, bind			<functional>, bind1st
  <functional>, bind			<functional>, bind2nd
  <memory>, unique_ptr       		<memory>, auto_ptr
*/
```

60、使用占位符需要using namespace std::placeholders; // _1、_2、_3这种
```
double myDivide(double x, double y)
{ return x / y; }
或者使用函数对象：std::divides<double> myDivide;

auto fnTest1 = bind(myDivide, _1, 2);
cout << fnTest1(10);		// 5
auto fnTest2 = bind(myDivide, _2, _1);
cout << fnTest2(10, 2);		// 0.2
auto fnTest3 = bind<int> (myDivide, _1, _2);	// 绑定int,绑定的是返回类型,return int(x/y);
cout << fnTest3(10, 3);		// 3
```

61、bind也可以用于成员函数或数据成员。
```
struct MyPair
{
    double a, b;
    double multiply() { return a*b; }   // 成员函数其实有个参数this
}

MyPair ten_two{10,2};
auto test1 = bind(&MyPair::multiply, _1);    // return x.multiply()
cout << test1(ten_two);     // 20
auto test2 = bind(&MyPair::a, ten_two); // return ten_two.a
cout << test2();            // 10
auto test3 = bind(&MyPair::b, _1);
cout << test3(ten_two);     // 2
```

62、bind的使用举例：
```
vector<int> vec{1,2,3,4,5};
int n1 = count_if(vec.cbegin(), vec.cend(), bind2nd(less<int>(), 5)); // n1=4

auto test = bind(less<int>(), _1, 5);
int n2 = count_if(vec.cbegin(), vec.cend(), test); // n2=4
int n3 = count_if(vec.cbegin(), vec.cend(), bind(less<int>(), _1, 5)); // n3=4
```

63、对逆向迭代器(例如rbegin()、rend())取值就是对正向迭代器(例如begin()、end())退一位取值。

对逆向迭代器前进(++)就是对正向迭代器后退(--).

64、advance(ite, 2); // 迭代器ite前进三个位置

65、迭代器适配器inserter可以用于插入元素，而无需过多关心目标容器是否会越界。

66、tr1提供了一个万用的hash function，最底层计算hashCode的函数名为hash_combine().

67、GNU 4.9为string提供了计算hashCode的方法，为struct hash提供偏特化(放在std名称空间内)。

相关源代码在.\mingw530_32\i686-w64-mingw32\include\c++\bits\basic_string.h
```
// std::hash specialization for string.
template<>
struct hash<string>
    : public __hash_base<size_t, string>
{
    size_t
    operator()(const string& __s) const noexcept
    { return std::_Hash_impl::hash(__s.data(), __s.length()); }
};
```

68、tuple可以认为是允许放任何类型的一个集合。

```
例如，auto t1 = make_tuple(22, 4.4, "hello");

cout << get<0>(t1); // 22
cout << get<1>(t1); // 4.4
cout << get<2>(t1); // hello

int n1;
float f1;
string s1;
tie(n1, f1, s1) = t1;
```

69、tuple可以比较、可以赋值、可以取其中的元素、可以对其中的元素进行赋值、可以用cout进行输出 等等。

70、tuple类是继承模板参数少一个的自己(其实就像递归)。
它的尾部_M_tail返回的是*this，然后转型为_Inherited&，也就是它的父类。
```
 /**
   * Recursive tuple implementation. Here we store the @c Head element
   * and derive from a @c Tuple_impl containing the remaining elements
   * (which contains the @c Tail).
  */
template<int _Idx, typename _Head, typename... _Tail>
struct _Tuple_impl<_Idx, _Head, _Tail...>
    : public _Tuple_impl<_Idx + 1, _Tail...>
{
    typedef _Tuple_impl<_Idx + 1, _Tail...> _Inherited;

    _Inherited& _M_tail() { return *this; }
    ... 
};

template<typename... _Elements> 
class tuple : public _Tuple_impl<0, _Elements...>
{
    typedef _Tuple_impl<0, _Elements...> _Inherited;
    ...
};
```

71、C++是不能拒绝被继承的。

`一个类如果它会被继承，那么它就应该有虚析构函数。`

72、Since C++11，对于type traits的测试:
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

    // 是不重要的吗？
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

73、通过模板的偏特化去掉const属性。也能通过模板的偏特化让type traits知道是不是具有某个属性。相关源代码在.\mingw530_32\i686-w64-mingw32\include\c++\tr1\type_traits
但一个东西是否is_class，可能不是由偏特化实现的，可能是从编译器得到的。
```
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

74、如果自己写的类型需要cout来输出，那么需要重载<<运算符。

75、拷贝构造和移动构造，性能差异很大。假设M是个容器：
```
M c1;
M c2(c1);
M c3(std::move(c1));    // 注意，必须要确保以后不会再使用c1(因为move有点像浅拷贝)
```

76、vector因为有扩充，2倍2倍地扩充，因此插入100个元素会不止100次调用构造函数。

77、移动构造函数和移动赋值函数举例(联想一下深拷贝和浅拷贝)：
```
class MyString
{
private:
    char* _data;
    size_t _len;

public:
    MyString(MyString&& str) noexcept
    : _data(str._data)
    , _len(str._len)
    {
        str._len = 0;
        str._data = NULL;   // 避免delete
    }

    MyString& operator=(MyString&& str)
    {
        if(this != &str)
        {
            if(_data)
                delete _data;
            _len = str._len;
            _data = str._data;  // move
            str._len = 0;
            str._data = NULL;   // 避免delete
        }
    }

    virtual ~MyString()
    {
        if(_data)
            delete _data;
    }
}
```

78、std::string也具有移动构造函数和移动赋值函数。
