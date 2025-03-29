函数的返回值是类的对象时，GCC编译器会进行优化(不调用拷贝构造函数或移动构造函数)，而MSVC编译器不会。

以下例子分别在下面环境运行通过：
- ①Qt5.7.0，mingw5.3.0，debug模式；
- ②Visual Studio 2015，debug模式.

**理论可供学习研究，但是具体情况还是要靠实践！**
```
#include <map>
#include <iostream>

using namespace  std;

template <class Key, class T>
class MyMap : public map<Key,T>
{
public:
    MyMap() : map<Key,T>()
    {
        cout << "Ctor" << endl;
    }

    ~MyMap()
    {
        cout << "DeCtor" << endl;
    }

    MyMap(const MyMap<Key, T> &other) : map<Key,T>(other)
    {
        cout << "Copy Ctor" << endl;
    }

    MyMap& operator=(const MyMap<Key, T> &demoClassTmp)
    {
        cout << "Copy Assignment" << endl;
        return map<Key,T>::operator =(demoClassTmp);;
    }

    MyMap(const MyMap<Key, T> &&other) : map<Key,T>(other)
    {
        cout << "Move Ctor" << endl;
    }

    MyMap& operator=(const MyMap<Key, T> &&demoClassTmp)
    {
        cout << "Move Assignment" << endl;
        return map<Key,T>::operator =(demoClassTmp);;
    }
};

MyMap<int,int> getMapCopy()
{
    MyMap<int,int> m;
    m[1] = 1;
    return m;
}

int main(int argc, char *argv[])
{
	cout << "=================map====================" << endl;

	{
		MyMap<int, int> d = getMapCopy();
	}

	cout << "=======================================" << endl;

	{
		MyMap<int, int> d = std::move(getMapCopy());
	}

	system("pause");
	return 0;
}
```

```
①GCC编译器，当MyMap有移动构造和移动赋值函数时，输出结果为：
=================map====================
Ctor
DeCtor
=======================================
Ctor
Move Ctor
DeCtor
DeCtor

①GCC编译器，当MyMap没有移动构造和移动赋值函数时，输出结果为：
=================map====================
Ctor
DeCtor
=======================================
Ctor
Copy Ctor
DeCtor
DeCtor

③MSVC编译器，当MyMap有移动构造和移动赋值函数时，输出结果为：
=================map====================
Ctor
Move Ctor
DeCtor
DeCtor
=======================================
Ctor
Move Ctor
DeCtor
Move Ctor
DeCtor
DeCtor

④MSVC编译器，当MyMap没有移动构造和移动赋值函数时，输出结果为：
=================map====================
Ctor
Copy Ctor
DeCtor
DeCtor
=======================================
Ctor
Copy Ctor
DeCtor
Copy Ctor
DeCtor
DeCtor
```
