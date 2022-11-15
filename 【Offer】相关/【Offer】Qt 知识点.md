1. movetothread怎么实现的？
- 改变了threadData这个结构体的数据，例如 threadid。
- 不能对有父对象（parent 属性）的对象使用QObject::moveToThread()函数

2. QList 和 std::list 的底层实现有什么区别？
- QList 好像是基于 QByteArray 的，好像没有next指针这个东西。

3. Qt的内存管理是怎么做的？
- 半自动内存管理机制。有个 QObject 对象树。
- QObject及其派生类的对象，如果其parent非0，那么其parent析构时会析构该对象。

4. 继承QObject的类怎么析构？
- 析构的时候，将其从对象树摘掉。

5. moc是什么？
- 元对象系统。

6. QVector的底层数据结构是什么？
- QByteArray，字节数组。

7. C++二进制兼容性问题？Qt的d指针。
- 具体就是不能改动态库接口。

8. connect第5个参数，分别都有什么用？队列绑定是怎么实现的？信号槽跨线程是怎么实现的？它怎么知道是否在同一个线程？
- 改变了threadData这个结构体的数据，例如 threadid。

9. deletelater是怎么实现的？为什么要这么做？
- 不马上delete，会包装成一个事件，扔到事件循环里面去。

10. Windows的消息机制、Qt的消息机制？
- 每个线程有自己的事件循环。

11. QObject对象的析构是线程安全的吗？
- 不是？

12. QString 和 std::string 的底层实现有什么区别？


13. std::shared_ptr 是线程安全的吗？
不是。引用计数是线程安全的，因为用的都是原子变量。

14. 静态变量析构的时候，调用另一个静态变量怎么办？静态变量的析构顺序？
- C++全局静态变量的析构销毁顺序是未定义的，特别是在多线程环境，多dll环境下，特别复杂。
- 如果是单线程的话，建议用指针的方式，在AppInit的时候构造，在AppExit的时候delete，并设置为0。别的地方引用的时候先判断。
- 保证构造顺序？

15. std::unique_ptr有时候sizeof是4个字节大小？底层实现是怎样的？
- tuple实现？成员变量是个tuple？

16. Windows系统求CPU占用率是没有系统API的，那怎么求？
- 计算用户态时间和内核态消耗的时间，除以时钟周期？

17. 信号槽传参，如果要传递自己定义的结构体/类/对象，需要做些什么？
- 在main()函数中注册这种类型：qRegisterMetaType(“MyClass”);
- Q_DECLARE_METATYPE(MyClass); //在自定义类或者结构体等声明后紧接着用宏Q_DECLARE_METATYPE声明自定义自定义数据类型 
- 为了QMetaType直接能够识别，自己进加到他的元对象系统？

18. stdcall 与 cdcall 的区别？
- __stdcall的全称是standard call。是C++的标准调用方式。函数参数的入栈顺序为从右到左入栈。函数返回时使用ret x指令，其中x为调整堆栈的字节数。这种方式叫做自动清栈。即被调用的函数的参数个数是固定的，调用者必须严格按照定义传递参数，一个不多，一个不少。
- __cdecl的全称是C Declaration，即C语言默认的函数调用方式。函数参数的入栈顺序为从右到左入栈。函数返回时作用ret指令。由调用者手动清栈。被调用的函数支持可变参数。调用者根据调用时传入参数的个数，手动平衡堆栈。
- 两者的相同点与不同点
	- 相同点
		- 参数入栈顺序相同：从右到左
	- 不同点
		- 堆栈平衡方式不同：__stdcall自动清栈，__cdecl手动清栈。
		- 返回指令不同：_stdcall使用retn x, __cdecl使用ret
		- 编译后函数的修饰名不同： 假设有函数int foo(int a, int b), 采用__stdcall编译后的函数名为_foo@8，而采用__cdecl编译后的函数名为_foo。
	- 从函数调用看，b和a依次被push进堆栈，而在函数中又通过相对于ebp(即刚进函数时的堆栈指针）的偏移量存取参数。函数结束后，ret 8表示清理8个字节的堆栈，函数自己恢复了堆栈。
- __fastcall：快速调用方式。所谓快速，这种方式选择将参数优先从寄存器传入（ECX和EDX），剩下的参数再从右向左从栈传入。因为栈是位于内存的区域，而寄存器位于CPU内，故存取方式快于内存，故其名曰“__fastcall”。

19. shared_from_this()的实现原理
- enable_shared_from_this的一种实现方法是，其内部有一个weak_ptr类型的成员变量_Wptr，当shared_ptr构造的时候，如果其模板类型继承了enable_shared_from_this，则对_Wptr进行初始化操作，这样将来调用shared_from_this函数的时候，就能够通过weak_ptr构造出对应的shared_ptr。

20. Qt的自定义控件，有搞过吗？

21. Qt除了QThread外的线程库有用过吗？
QThreadPool、QtConcurrent




排序算法有哪些？
基数排序有了解吗？
快速排序是怎样的？（冒泡排序、快速排序、归并排序出现的频率都很高）
图的算法？
红黑树如何保持平衡？
有了解过其他平衡树吗？
std::sort是怎么实现的？

大小堆是怎么调整的？
一个exe的加载执行过程？
main函数前后做了什么？
Windows内核对象是什么？
TCP和UDP可以同时使用同一个端口号吗？
从汇编的角度讲一下函数调用的过程？

