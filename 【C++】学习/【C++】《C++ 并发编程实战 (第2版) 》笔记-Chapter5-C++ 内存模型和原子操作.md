###### 五、C++ 内存模型和原子操作

1. 对象和内存区域 
    1. C++ 标准只将“对象”定位为“某一存储范围”。
    2. 位域有一项重要的性质：尽管相邻的位域分属不同对象，但照样算作同一内存区域。
    3. 整个结构体就是一个对象，它由几个子对象构成，每个数据成员即为一个子对象。
        ```
        struct my_data
        {
            int i;
            double d;
            unsigned bf1:10;
            int bf2:25;
            int bf3:0;
            int bf4:9;
            int i2;
            char c1, c2;
            std::string s;
        };
        ```
        - 位域 bf1 和 bf2 共用同一块内存区域，std::string 对象 s 则由几块内存区域构成，别的数据成员都有各自的内存区域。
        - bf3 是 0 宽度位域（其变量名被注释掉，因为 0 宽度位域必须匿名），与 bf4 彻底分离，将 bf4 排除在 bf3 的内存区域之外，但 bf3 实际上并不占用任何内存区域。
        - 若变量属于内建基本类型（如 int 或 char），则不论其大小，都占用一块内存区域（且仅此一块），即便它们的位置相邻或它们是数列中的元素；相邻的位域属于同一内存区域。

2. 改动序列
    1. 在一个 C++ 程序中，每个对象都具有一个改动序列，它由所有线程在对象上的全部写操作构成，其中第一个写操作即为对象的初始化。
    2. 大部分情况下，这个序列会随程序的多次运行而发生变化，但是在程序的任意一次运行过程中，所含的全部线程都必须形成相同的改动序列。
    3. 在不同的线程上观察属于同一变量的序列，如果所见各异，就说明出现了数据竞争和未定义行为。若我们采用了原子操作，那么编译器有责任保证必要的同步操作有效、到位。

3.  原子操作是不可分割的操作。

4. 标准原子类型
    1. 标准原子类型的定义位于头文件<atomic>内。这些类型的操作全是原子化的，并且，根据语言的定义，C++ 内建的原子操作也仅仅支持这些类型，尽管通过采用互斥，我们能够令其他操作实现原子化。
    2. 事实上，我们可以凭借互斥保护，模拟出标准的原子类型：它们全部（几乎）都具备成员函数 is_lock_free()，准许使用者判定某一给定类型上的操作是能由原子指令直接实现（x.is_lock_free() 返回 true），还是要借助编译器和程序库的内部锁来实现（x.is_lock_free() 返回 false）。
    3. 原子操作的关键用途是取代需要互斥的同步方式。
    4. 从 C++ 17 开始，全部原子类型都含有一个静态常量表达式成员变量，形如 X::is_always_lock_free，功能与那些宏相同：考察编译生成的一个特定版本的程序，当且仅当在所有支持该程序运行的硬件上，原子类型 X 全都以无锁结构形式实现，该成员变量的值才为 true。
    5. 若在任意给定的目标硬件上，std::atomic<int> 都以无锁结构形式实现，std::atomic<int>::is_always_lock_free 的值会是 true。
    6. 上述的宏是 ATOMIC_BOOL_LOCK_FREE、ATOMIC_CHAR_LOCK_FREE、ATOMIC_CHAR16_T_LOCK_FREE、ATOMIC_CHAR32_T_LOCK_FREE、ATOMIC_WCHAR_T_LOCK_FREE、ATOMIC_SHORT_LOCK_FREE、ATOMIC_INT_LOCK_FREE、ATOMIC_LONG_LOCK_FREE、ATOMIC_LLONG_LOCK_FREE 和 ATOMIC_POINTER_LOCK_FREE。
    7. 依据 C++ 某些内建整数类型和相应的无符号类型，std::atomic<> 特化成各种原子类型，上面的宏按名字与之一一对应，功能是标示该类型是否属于无锁结构（LLONG 代表 long long，而 POINTER 代表所有指针类型）。
    8. 假设某原子类型从来不属于无锁结构，那么，对应的宏取值为0；若它一直都属于无锁结构，则宏取值为2；如果像前文所述，等到运行时才能确定该原子类型是否属于无锁结构，就取值为1。
    9. 只有一个原子类型不提供 is_lock_free() 成员函数：std::atomic_flag。它是简单的布尔标志，因此必须采取无锁操作。只要利用这种简单的无锁布尔标志，我们就能实现一个简易的锁，进而基于该锁实现其他所有原子类型。
    10. 其余的原子类型都是通过类模板 std::atomic<> 特化得出的，功能更加齐全，但可能不属于无锁结构。各种原子类型依据内建类型特化而成（如 std::atomic<int> 和 std::atomic<void*>）。
    11. 对于类型 T 的标准 typedef，只要为相同的名称冠以前缀“atomic_”，写出 atomic_T，即为相应的原子类型。该模式同样适用于内建类型，只不过 signed 缩略成字母 s，unsigned 缩略成字母 u，而 long long 缩略成 llong。
    12. 由于不具备拷贝构造函数或拷贝赋制操作符，因此按照传统做法，标准的原子类型对象无法复制，也无法赋值。然而，它们其实可以接受内建类型赋值，也支持隐式地转换成内建类型，还可以直接经由成员函数处理，如 load() 和 store()、exchange()、compare_exchange_weak() 和 compare_exchange_strong() 等。它们还支持复合赋值操作，如 +=、-=、*= 和 |= 等。而且，整数和指针的 std::atomic<> 特化都支持 ++ 和 -- 运算。这些操作符有对应的具名成员函数，fetch_add() 和 fetch_or() 等。赋值操作符的返回值是存入的值，而具名成员函数的返回值则是操作前的值。习惯上，C++ 的赋值操作符通常返回引用，指向接受赋值的对象，但原子类型的设计与此有别，要防止暗藏错误。否则，为了从引用获取存入的值，代码须执行单独的读取操作，使赋值和读取操作之间存在间隙，让其他线程有机可乘，得以改动该值，结果形成条件竞争。
    13. 当然，类模板 std::atomic<> 并不局限于上述的特化类型。它其实具有泛化模板（根据 C++ 标准，相对全特化模板和偏特化模板，普通模板称为泛化模板，又称主模板、通例模板），可依据用户自定义类型创建原子类型的变体。该泛化模板所具备的操作仅限于以下几种：load()、store()（接受用户自定义类型的赋值，以及转换为用户自定义类型）、exchange()、compare_exchange_weak()、compare_exchange_strong()。
    14. 对于原子类型上的每一种操作，我们都可以提供额外的参数，从枚举类 std::memory_order 取值，用于设定所需的内存次序语义（memory-ordering semantics）。枚举类 std::memory_order 具有 6 个可能的值，包括 std::memory_order_relaxed、std::memory_order_acquire、std::memory_order_consume、std::memory_order_acq_rel、std::memory_order_release 和 std::memory_order_seq_cst。
    15. 操作的类别决定了内存次序所准许的取值。若我们没有把内存次序显式设定成上面的值，则默认采用最严格的内存次序，即 std::memory_order_seq_cst。目前，知道操作被划分 3 类就已经足够：
        - 存储（store）操作，可选用的内存次序有 std::memory_order_relaxed、std::memory_order_release 或 std::memory_order_seq_cst。
        - 载入（load）操作，可选用的内存次序有 std::memory_order_relaxed、std::memory_order_consume、std::memory_order_acquire 或 std::memory_order_seq_cst。
        - “读-改-写”（read-modify-write）操作，可选用的内存次序有 std::memory_order_relaxed、std::memory_order_consume、std::memory_order_acquire、std::memory_order_release、std::memory_order_acq_rel 或 std::memory_order_seq_cst。

5. 操作 std::atomic_flag
    1. std::atomic_flag 是最简单的标准原子类型，表示一个布尔标志。该类型的对象只有两种状态：成立或置零。二者必居其一。
    2. std::atomic_flag 类型的对象必须由宏 ATOMIC_FLAG_INIT 初始化，它把标志初始化为置零状态：std::atomic_flag f = ATOMIC_FLAG_INIT。无论在哪里声明，也无论处于什么作用域，std::atomic_flag 对象永远以置零状态开始，别无他选。全部原子类型中，只有 std::atomic_flag 必须采取这种特殊的初始化处理，它也是唯一保证无锁的原子类型。如果 std::atomic_flag 对象具有静态存储期，它就会保证以静态方式初始化，从而避免初始化次序的问题。
    3. 完成 std::atomic_flag 对象的初始化后，我们只能执行 3 种操作：销毁、置零、读取原有的值并设置标志成立。这分别对应于析构函数、成员函数 clear()、成员函数 test_and_set()。我们可以为 clear() 和 test_and_set() 指定内存次序。clear() 是存储操作，因此无法采用 std::memory_order_acquire、std::memory_order_acq_rel 内存次序，而 test_and_set() 是“读-改-写”操作，因此能采用任何内存次序。对于上面两个原子操作，默认内存次序都是 std::memory_order_seq_cst。
    4. 我们无法从 std::atomic_flag 对象拷贝构造出另一个对象，也无法向另一个对象拷贝赋值，这两个限制并非 std::atomic_flag 独有，所有原子类型都同样受限。原因是按定义，原子类型上的操作全都是原子化的，但拷贝赋值和拷贝构造都涉及两个对象，而牵涉两个不同对象的单一操作却无法原子化。所以，原子对象禁止拷贝赋值和拷贝构造。
    5. 正因为 std::atomic_flag 功能有限，所以它可以完美扩展成自旋锁互斥（spin-lock mutex）。最开始令原子标志置零，表示互斥没有加锁。我们反复调用 test_and_set() 试着锁住互斥，一旦读取的值变成 false，则说明线程已将标志设置成立（其新值为 true），则循环终止。而简单地将标志置零即可解锁互斥。
        ```
        // 采用 std::atomic_flag 实现自旋锁互斥
        class spinlock_mutex
        {
            std::atomic_flag flag;
        public:
            spinlock_mutex()
                : flag(ATOMIC_FLAG_INIT)
            {}
            void lock()
            {
                while(flag.test_and_set(std::memory_order_acquire));
            }
            void unlock()
            {
                flag.clear(std::memory_order_release);
            }
        }
        ```
        - 从本质上讲，上述自旋锁互斥在 lock() 函数内忙等。
        - 由于 std::atomic_flag 严格受限，甚至不支持单纯的无修改查值操作，无法用作普通的布尔标志，因此最好还是使用 std::atomic<bool>。

6. 操作 std::atomic<bool>
    1. std::atomic<bool> 是基于整数的最基本的原子类型。
    2. 按 C++ 惯例，赋值操作符通常返回一个引用，指向接受赋值的目标对象（等号左侧的对象）。而非原子布尔量也可以向 std::atomic<bool> 赋值，但该赋值操作符的行为有别于惯常做法：它直接返回赋予的布尔值。这是原子类型的又一个常见模式：它们所支持的赋值操作符不返回引用，而是按值返回（该值属于对应的非原子类型）。
    3. std::atomic<bool> 还支持单纯的读取（没有伴随的修改行为）：隐式做法是将实例转换为普通布尔值，显式做法则是调用 load()。
    4. 不难看出，store() 是存储操作，而 load() 是载入操作，但 exchange() 是“读-改-写”操作。
        ```
        std:atomic<bool> b;
        bool x = b.load(std::memory_order_acquire);
        b.store(true);
        x = b.exchange(false, std::memory_order_acq_rel);
        ```
    5. std::atomic<bool> 还引入了一种操作：若原子对象当前的值符合预期，就赋予新值。它与 exchange() 一样，同为“读-改-写”操作。
    6. 依据原子对象当前的值决定是否保存新值，这一新操作被称为“比较-交换”，实现方式是成员函数 compare_exchange_weak() 和 compare_exchange_strong()。比较-交换操作是原子类型的编程基石。使用者给定一个期望值，原子变量将它和自身的值比较，如果相等，就存入另一既定的值，否则，更新期望值所属的变量，向它赋予原子变量的值。比较-交换函数返回布尔类型，如果完成了保存动作（前提是两值相等），则操作成功，函数返回 true；反之操作失败，函数返回 false。
    7. 对于 compare_exchange_weak()，即使原子变量的值等于期望值，保存动作还是有可能失败，在这种情形下，原子变量维持原值不变，compare_exchange_weak() 返回 false。原子化的比较-交换必须由一条指令单独完成，而某些处理器没有这种指令，无从保证该操作按原子化方式完成。要实现比较-交换，负责的线程则须改为连续运行一系列指令，但在这些计算机上，只要出现线程数量多于处理器数量的情形，线程就有可能执行到中途因系统调度而切出，导致操作失败。这种计算机最有可能引发上述的保存失败，我们称之为佯败（spurious failure）。其败因不是变量值本身存在问题，而是函数执行时机不对。因为 compare_exchange_weak() 可能佯败，所以它往往必须配合循环使用。
        ```
        bool expected = false;
        extern atomic<bool> b;  // 由其他源文件的代码设定变量的值
        while(!b.compare_exchange_weak(expected, true) && !expected);
        ```
        - 此例中，只要 expected 变量还是 false，就说明 compare_exchange_weak() 调用发送佯败，我们就继续循环。
    8. 另一方面，只有当原子变量的值不符合预期时，compare_exchange_strong() 才返回 false。这让我们得以明确知悉变量是否成功修改，或者是否存在另一线程抢先切入而导致佯败，从而摆脱上例所示的循环。
    9. 假如经过简单计算就能得出要保存的值，而在某些硬件平台上，虽然使用 compare_exchange_weak() 可能导致佯败，但改用 compare_exchange_strong() 却会形成双重嵌套循环（因为 compare_exchange_strong() 自身内部含有一个循环），那么采用 compare_exchange_weak() 比较有利于性能。反之，如果存入的值需要耗时的值，选择 compare_exchange_strong() 则更加合理。因为只要预期值没有变化，就可避免重复计算。
    10. 比较-交换函数还有一个特殊之处：它们接受两个内存次序参数。这使程序能区分成功和失败两种情况，采用不同的内存次序语义。合适的做法是：若操作成功，就采用 std::memory_order_acq_rel 内存次序，否则就改用 std::memory_order_relaxed 内存次序。失败操作没有存储行为，所以不可能采用 std::memory_order_release 内存次序或 std::memory_order_acq_rel 内存次序。因而这两种内存次序不准用作失败操作的参数。失败操作设定的内存次序不能比成功操作的更严格；若将失败操作的内存次序指定为 std::memory_order_acquire 或 std::memory_order_seq_cst，则要向成功操作设定同样的内存次序。
    11. 如果没有设定失败操作的内存次序，那么编译器就假定它和成功操作具有同样的内存次序，但其中的释放语义会被移除：memory_order_release 会变成 std::memory_order_relaxed，而 std::memory_order_acq_rel 会变成 std::memory_order_acquire。若成功和失败两种情况都未设定内存次序，则它们采用默认内存次序 std::memory_order_seq_cst，依照完全顺次的方式读写内存。
    12. 选择不同的内存次序会使运行结果各异。
    13. std::atomic<bool> 和 std::atomic_flag 的另一不同点是，前者有可能不具备无锁结构，它的实现可能需要在内部借用互斥，以保证操作的原子性。这通常不成问题，但保险起见，我们还可以调用成员函数 is_lock_free()，检查 std::atomic<bool> 是否具备真正的无锁操作。除 std::atomic_flag 之外，所有原子类型都提供这项检查功能。
    
7. 操作 std::atomic<T*>: 算术形式的指针运算
    1. 它与 std::atomic<bool> 相似，同样不能拷贝复制或拷贝赋值。然而，只要依据适合的指针，就可以创建该原子类型的对象，接受其赋值。
    2. fetch_add() 和 fetch_sub() 返回原来的地址。
    3. 由于 fetch_add() 和 fetch_sub() 都是“读-改-写”操作，因此可以为其选取任何内存次序，它们还能参与释放次序。

8. 操作标准整数原子类型
    1. 在 std::atomic<int> 和 std::atomic<unsigned long long> 这样的整数原子类型上，我们可以执行的操作颇为齐全：既包括常用的原子操作（load()、store()、exchange()、compare_exchange_weak() 和 compare_exchange_strong()），也包括原子运算（fetch_add()、fetch_sub()、fetch_and()、fetch_or()、fetch_xor()），以及这些运算的复合赋值形式（+=、-=、&=、|= 和 ^=），还有前后缀形式的自增和自减（++ x、x++、--x、x--）。
    2. 实际上，整数原子类型往往只用作计数器或位掩码。若有必要，我们还能利用 compare_exchange_weak() 配合循环轻松实现更多操作。
    3. 上述操作的语义非常接近 std::atomic<T*> 类型的 fetch_add() 和 fetch_sub()：具名函数按原子化方式执行操作，并返回原子对象的旧值，然后复合赋值运算符则返回新值。前后缀形式的自增和自减都按既有方式工作：++ x 令变量自增，并返回新值；x++ 也令变量自增、但返回旧值。

9. 泛化的 std::atomic<> 类模板
    1. 对于某个自定义类型 UDT，要满足一定条件才能具现化出 std::atomic<UDT>：必须具备平实拷贝赋值操作符，它不得含有任何虚函数，也不可以从虚基类派生得出，还必须由编译器代其隐式生成拷贝赋值操作符；另外，若自定义类型具有基类或非静态数据成员，则它们同样具备平实拷贝赋值操作符。由于以上限制，赋值操作不涉及任何用户编写的代码，因此编译器可借用 memcpy() 或采取与之等效的行为完成它。
    2. “平实拷贝赋值操作符”的实质含义是：平直、简单的原始内存赋值及其等效操作。
    3. 比较-交换操作所采用的是逐位比较运算，效果等同于直接使用 memcmp() 函数。即使 UDT 自行定义了比较运算符，在这项操作中也会被忽略。若自定义类型含有填充位，却不参与普通比较操作，那么即使 UDT 对象的值相等，比较-交换操作还是会失败。
    4. 编译器根据类定义的 alignas 说明符或编译命令，可能会在对象内各数据成员后方特意留出间隙，令它们按 2/4/8 字节或其他 2 次幂倍乘数字对齐内存地址，从而加速内存读写操作。这些间隙即为填充位，它们不具名，对使用者不可见。 
    5. 即使原子类型内本来存有的值与比较的值相等，若两个值表示方式不同（根据 IEEE 754 浮点数标准，容许采用多种形式表示同一数值），依然会令函数操作失败。请注意，浮点值的算术原子运算并不存在。
    6. 假设我们用某个自定义类型特化 std::atomic<>，而该类型定义了自己的等值比较运算符重载，它的判断方式与 memcmp() 不同，如果我们就这一特化调用 compare_exchange_strong()，那么该函数的行为与处理浮点原子类型的情况类似，虽然参与比较的两个值相等，但比较-交换操作还是会因表示方式不同而失败。
    7. 若某类型单纯含有其中一种数据（计数器、标志、指针、简单数据元素的数组等），我们还是能依据它将 std::atomic<> 具现化。

10. 原子操作的非成员函数
    1. 大部分非成员函数依据对应的成员函数命名，只不过冠以前缀“atomic_”（如 std::atomic_load()），它们还针对各原子类型进行了重载。只要有可能指定内存次序，这些函数就衍化出两个变体：一个带有后缀“_explicit”，接收更多参数以指定内存次序，而另一个则不带后缀也不接收内存次序参数，如 std::atomic_store_explicit(&atomic_var, new_value, std::memory_order_release) 与 std::atomic_store(&atomic_var, new_value)。成员函数的调用会隐式地操作原子对象，但所有非成员函数的第一个参数都是指针，指向所要操作的目标原子对象。
    2. 根据 C++ 标准的设计，这些非成员函数要兼容 C 语言，所以它们全都只接受指针，而非引用。例如，成员函数 compare_exchange_weak() 和 compare_exchange_strong() 的第一个参数都是引用（期望值），而非成员函数 std::atomic_compare_exchange_weak() 的第二个参数与之对应，却是指针（其第一个参数是目标原子对象的指针）。负责比较-交换的成员函数都具有两个重载，一个版本只接受一种内存次序（默认参数值为 std::memory_order_seq_cst），而另一版本则接受两种内存次序，分别用于成功和失败的情况。但非成员函数 std::atomic_compare_exchange_weak_explicit() 则没有重载版本，须同时为两种情况各自设定内存次序。
    3. 操作 std::atomic_flag 的非成员函数是 std::atomic_flag_test_and_set() 和 std::atomic_flag_clear()，它们并未严格遵从上述规则，在名字中加入了“_flag”。它们也具有其他变体，以后缀“_explicit”结尾，用于指定内存次序，如 std::atomic_flag_test_and_set_explicit() 和 std::atomic_flag_clear_explicit()。
    4. 标准库给出了共享指针的原子操作（载入、存储、交换和比较-交换），它们与标准原子类型上的操作一样，都是对应的同名函数的重载，而且第一个参数都属于 std::shard_ptr<>* 类型（该参数是形式上的双重指针。参数本身是一个指向 shared_ptr 对象的指针，而该 shared_ptr 对象指向最终目标）。
        ```
        std::shared_ptr<my_data> p;
        void process_global_data()
        {
            std::shared_ptr<my_data> local = std::atomic_load(&p);
            process_data();
        }
        void update_global_data()
        {
            std::shared_ptr<my_data> local(new my_data);
            std::atomic_store(&p, local);
        }
        ```
    5. atomic_shared_ptr<> 被设计成一个独立类型，因为按照这种形式，它有机会通过无锁方式实现，而且比起普通 std::shared_ptr 对象，它没有增加额外开销。但是在目标硬件平台上，我们仍需查验它是否属于无锁实现，这可以由成员函数 is_lock_free() 判定，与类模板 std::atomic<> 上的做法相同。
    6. 在多线程环境下处理共享指针，我们要避免采用普通的 std::shared_ptr 类型，也不要通过非成员原子函数对其进行操作（开发者很容易忘记这么做而误用普通函数），类型 std::experimental::atomic_shared_ptr 应予优先采用（就算它不是无锁实现）。原因是，后者可以使代码更加清晰，并确保全部访问都按原子化方式进行，还能预防误用普通函数，最终避免数据竞争。

11. 同步操作和强制次序
    1. 两个线程同时读写变量
        ```
        // 利用原子操作强制其他非原子操作遵从预定次序
        #include <vector>
        #include <atomic>
        #include <iostream>
        std::vector<int> data;
        std::atomic<bool> data_ready(false);
        void reader_thread()
        {
            while(!data_read.load())    // ①
            {
                std::this_thread::sleep(std::chrono::milliseconds(1));
            }
            std::cout << "The answer = " << data[0] << "\n";    // ②
        }
        void writer_thread()
        {
            data.push_back(42);     // ③
            data_ready = true;      // ④
        }
        ```
        - 原子变量 data_ready 的操作提供了所需的强制次序，它属于 std::atomic<bool> 类型，凭借两种内存模型关系“先行”和“同步”，这些操作确定了必要的次序。数据写出 ③ 在标志变量 data_ready 设置为成立 ④ 之前发生，标志判别 ① 在数据读取 ② 之前发生。等到从变量 data_ready 读取的值变成 true，写出动作与该读取动作即达成同步，构成先行关系。所以，这些操作被强制施行了预定的次序：数据写出在数据读取操作前面发生。
    2. 同步关系只存在于原子类型的操作之间。如果一种数据结构含有原子类型，并且其整体操作都涉及恰当的内部原子操作，那么该数据结构的多次操作之间（如锁定互斥）就可能存在同步关系。但同步关系从根本上来说来自原子类型的操作。
    3. 同步关系的基本思想是：对变量 x 执行原子写操作 W 和原子读操作 R，且两者都有适当的标记。只要满足下面其中一点，它们即彼此同步。
        - R 读取了 W 直接存入的值。
        - W 所属线程随后还执行了另一原子写操作，R 读取了后面存入的值。
        - 任意线程执行一连串“读-改-写”操作（如 fetch_add() 或 compare_exchange_weak()），而其中第一个操作读取的值由 W 写出。
    4. 先行关系和严格先行关系是在程序中确立操作次序的基本要素；它们的用途是清楚界定哪些操作能看见哪些操作产生的结果。
    5. 函数调用需要求出所传入参数的值，但求值运算次序并不明确。
        ```
        #include <iostream>
        void foo(int a, int b)
        {
            std::cout << a << ", " << b << std::endl;
        }
        int get_num()
        {
            static int i = 0;
            return ++i;
        }
        int main()
        {
            foo(get_num(), get_num());  // get_num() 发生两次调用，但没有明确的先后次序
        }
        ```
    6. 在线程间先行关系和先行关系中，各种操作都被标记为 memory_order_consume，而严格先行关系则无此标记。
    
12. 原子操作的内存次序——概述
    1. 原子类型上操作服从 6 种内存次序：memory_order_relaxed、memory_order_consume、memory_order_acquire、memory_order_release、memory_order_acq_rel 和 memory_order_seq_cst。其中，memory_order_seq_cst 是可选的最严格的内存次序，各种原子类型的所有操作都默认遵从该次序，除非我们特意为某项操作另行指定。
    2. 虽然内存次序共有 6 种，但它们只代表 3 种模式：先后一致次序（memory_order_seq_cst）、获取-释放次序（memory_order_consume、memory_order_acquire、memory_order_release 和 memory_order_acq_rel）、宽松次序（memory_order_relaxed）。
    3. 在不同的 CPU 架构上，这几种内存模型也许会有不同的运行开销。一些场景对性能不构成关键影响，普通开发者则能采取默认方式，按先后一致性次序执行原子操作（比起其他内存序，它分析起来要容易很多）。

13. 原子操作的内存次序——先后一致次序
    1. 只要服从该次序，全部线程所见的一切操作都必须服从相同的次序，这是目前最容易理解的内存次序，它因此被选作默认内存次序。
    2. 若其他线程使用的原子操作服从宽松次序，那么这种不起作用：它们依然有可能看见操作按不同的次序发生。因此，要保持绝对先后一致，所有线程都必须采用保序原子操作。
    3. 若某项操作标记为 memory_order_seq_cst，则编译器和 CPU 须严格遵循源码逻辑流程的先后顺序。在相同的线程上，以该项操作为界，其后方的任何操作不得重新编排到它前面，而前方的任何操作不得重新编排到它的后面，其中“任何”是指带有任何内存标记的任何变量之上的任何操作。
    4. 先后一致次序是最直观、最符合直觉的内存次序，但由于它要求所有线程间进行全局同步，因此也是代价最高的内存次序。在多处理器系统中，处理器之间也许为此而需要频繁通信。

14. 原子操作的内存次序——非先后一致次序
    1. 线程之间不必就事件发生次序达成一致。
    2. 如果没有指定程序服从哪种内存次序，则采用默认内存次序。它仅仅要求一点：全部线程在每个独立变量上都达成一致的修改序列。不同变量上的操作构成其特有的序列，假如各种操作都受施加的内存次序约束，若线程都能看到变量的值相应地保持一致，就容许这个操作序列在各线程中出现差别。

15. 原子操作的内存次序——宽松次序
    1. 如果采用宽松次序，那么原子类型上的操作不存在同步关系。在单一线程内，同一个变量上的操作仍然服从先行关系，但几乎不要求线程间存在任何次序关系。该内存次序的唯一要求是，在一个线程内，对相同变量的访问次序不得重新编排。
    2. memory_order_relaxed 次序无须任何额外的同步操作，线程间仅存的共有信息是每个变量的改动序列。
    3. 如果一个变量的存储操作和载入操作分属不同线程，那么因为采用了宽松次序，所以后者不一定能见到前者执行产生的效果，即存储的新值 true 还停留在 CPU 缓存中，而读取的 false 值是来自内存的旧值。

16. 原子操作的内存次序——理解宽松次序
    1. 交换操作：告诉我列表最后的值，再记录一个新值。
    2. compare_exchange_strong() 函数：若我给出的值与列表最后的值相等，就写下这个值；否则，请告诉我最后的值是什么。
    3. 除非万不得已，强烈建议避免使用宽松原子操作，即便要用，也请保持十二分警惕。

17. 原子操作的内存次序——获取-释放次序
    1. 获取-释放次序比宽松次序严格一些，它会产生一定程序的同步效果形成服从先后一致次序的全局总操作序列。在该内存模型中，原子化载入即为获取操作（memory_order_acquire），原子化存储即为释放操作（memory_order_release），而原子化“读-改-写”操作（像 fetch_add() 和 exchange()）则为获取或释放操作，或二者皆是（memory_order_acq_rel）。这种内存次序在成对的读写线程之间起到同步作用。释放与获取操作构成同步关系，前者写出的值由后者读取。换言之，若多个现场服从获取-释放次序，则其所见的操作序列可能各异，但其差异的程度和方式都受到一定条件的制约。
    2. 若某些操作标记为 memory_order_acquire 或 memory_order_release，那么编译器和 CPU 的重新编排行为将受其约束，根据代码逻辑流程的先后顺序，在相同的线程上，以该项被标记的操作为界，memory_order_acquire 限令后方的任何操作不得重新编排到它前面，memory_order_release 则限令前方的任何操作不得编排到它后面，其中“任何”是指带有任何内存标记的任何变量之上的任何操作。
    3. 获取和释放操作唯有成对才可以产生同步。释放操作所存储的值必须为获取操作所见，才会产生有效同步。
    4. 线程间的先行关系可传递。按此定义，获取-释放次序可用于多线程之间的数据同步，即使“过渡线程”的操作不涉及目标数据，也照样可行。

18. 原子操作的内存次序——通过获取-释放次序传递同步
    1. 运用获取-释放次序传递同步
        ```
        std::atomic<int> data[5];
        std::atomic<bool> sync1(false), sync2(false);
        void thread_1()
        {
            data[0].store(42, std::memory_order_relaxed);
            data[1].store(97, std::memory_order_relaxed);
            data[2].store(17, std::memory_order_relaxed);
            data[3].store(-141, std::memory_order_relaxed);
            data[4].store(2003, std::memory_order_relaxed);
            sync1.store(true, std::memory_order_release);   // ① 设置 sync1 成立
        }
        void thread_2()
        {
            while(!sync1.load(std::memory_order_acquire));  // ② 一直循环，到 sync1 成立为止
            sync2.store(true, std::memory_order_release);   // ③ 设置 sync2 成立
        }
        void thread_3()
        {
            while(!sync2.load(std::memory_order_acquire));  // ④ 一直循环，到 sync2 成立为止
            assert(data[0].load(std::memory_order_relaxed) == 42);
            assert(data[1].load(std::memory_order_relaxed) == 97);
            assert(data[2].load(std::memory_order_relaxed) == 17);
            assert(data[3].load(std::memory_order_relaxed) == -141);
            assert(data[4].load(std::memory_order_relaxed) == 2003);
        }
        ```
        - 尽管线程 thread_2 只接触过变量 sync1② 和 sync2③，但这足以同步线程 thread_1 和 thread_3，从而保证每个断言都不会触发。
        - 首先，线程 thread_1 对各数组的 data 和变量 sync1 执行操作，因同属一个线程，故各 data 元素的存储均在变量 sync1 的存储①之前发生。而线程 thread_2 运用 while 循环反复载入变量 sync1②，因此该操作最终会见到线程 thread_1 所存储的值，这两项操作形成配对，服从获取-释放次序。
    2. 采用  memory_order_acquire 次序的 fetch_sub() 不会与任何操作同步，因为它不是释放操作。类似地，存储操作无法与采用 memory_order_release 次序的 fetch_or() 同步，因为 fetch_or() 所含的读取行为并不是获取操作。若“读-改-写”操作采用 memory_order_acq_rel 次序，则其行为是获取和释放的结合，因此前方的存储操作会与之同步，而它也会与后方的载入操作同步。
    3. 若将获取-释放操作与保序操作交错混杂，那么保序载入的行为就与服从获取语义的载入相同，保序存储的行为则与服从释放语义的存储相同。如果“读-改-写”操作采用保序语义，则其行为是获取和释放的结合。混杂期间的宽松操作仍旧宽松，但由于获取-释放语义引入了同步关系（也附带引入了先行关系），这些操作的宽松程序因此受到限制。
    4. 如果原子操作对先后一致的要求不是很严格，那么由成对的获取-释放操作实现同步，开销会远低于由保序操作实现的全局一致顺序。

19. 原子操作的内存次序——获取-释放次序和 memory_order_consume 次序造成的数据依赖
    1. memory_order_consume 次序相当特别：它完全针对数据依赖，引入了线程间先行关系中的数据依赖细节。其特别之处还在于，C++ 17 标准明确建议我们对其不予采用。
    2. 这种内存次序有一个重要用途：按原子化方式载入某份数据的指针。我们把存储操作设定成 memory_order_release 次序，而将后面的读取操作设定为 memory_order_consume 次序，即可保证所指向的目标数据得到正确同步，而无须对任何非独立数据施加同步措施。
    3. 若代码中有大量携带依赖，则会造成额外开销。我们并不想编译器面对依赖而束手无策，而希望它将值缓存在 CPU 寄存器中，并重新编排指令进行优化。这时，我们可以运用 std::kill_dependency() 显式打断依赖链。std::kill_dependency() 是一个简单的函数模板，它复制调用者给出的参数，直接将其作为返回值，借此打断依赖链。假设有一个只读的全局数组，其索引值由其他线程给出，而我们采用 std::memory_order_consume 次序接收该值，那么我们就可以运用 std::kill_dependency() 告知编译器，无需重读数组元素。
    4. 在实际代码中，凡是要用到 memory_order_consume 次序的情形，我们应该一律改用 memory_order_acquire 次序，而使用 std::kill_dependency() 是没有必要的。

20. 释放序列和同步关系
    1. 如果存储操作的标记是 memory_order_release、memory_order_acq_rel 或 memory_order_seq_cst，而载入操作则以 memory_order_consume、memory_order_acquire 或 memory_order_seq_cst 标记，这些操作前后相扣成链，每次载入的值都源自前面的存储操作，那么该操作链由一个释放序列组成。若最后的载入操作服从内存次序 memory_order_acquire 或 memory_order_seq_cst，则最初的存储操作与它构成同步关系。但如果该载入操作服从的内存次序是 memory_order_consume，那么两者构成前序依赖关系。操作链中，每个“读-改-写”操作都可选用任意内存次序，甚至也能选用 memory_order_relaxed 次序。
    2. 原子类型上的操作具有各自内存次序语义，大多数同步关系据此形成。尽管如此，我们还可以使用栅栏引入别的次序约束。

21. 栅栏
    1. 栅栏具备多种操作，用途是强制施加内存次序，却无须改动任何数据。通常，它们与服从 memory_order_relaxed 次序的原子操作组合使用。
    2. 栅栏操作全部通过全局函数执行。
    3. 当线程运行至栅栏处时，它便对线程中其他原子操作的次序产生作用。栅栏也常常被称作“内存卡”或“内存屏障”，其得名原因是它们在代码中划出界线，限定某些操作不得通行。
    4. 针对不同变量上的宽松操作，编译器或硬件往往可以自主对其进行重新编排。栅栏限制了这种重新编排。在一个多线程程序中，可能原来并非处处具备先行关系和同步关系，栅栏则在欠缺之处引入这两种关系。
    5. 栅栏可以令宽松操作服从一定的次序
        ```
        #include <atomic>
        #include <thread>
        #include <assert.h>
        std::atomic<bool> x,y;
        std::atomic<int> z;
        void write_x_then_y()
        {
            x.store(true, std::memory_order_relaxed);               // ①
            std::atomic_thread_fence(std::memory_order_release);    // ②
            y.store(true, std::memory_order_relaxed);               // ③
        }
        void read_y_then_x()
        {
            while(!y.load(std::memory_order_relaxed));              // ④
            std::atomic_thread_fence(std::memory_order_acquire);    // ⑤
            if(x.load(std::memory_order_relaxed))                   // ⑥
                ++z;
        }
        int main()
        {
            x = false;
            y = false;
            z = 0;
            std::thread a(write_x_then_y);
            std::thread b(read_y_then_x);
            a.join();
            b.join();
            assert(z.load() != 0);                                  // ⑦
        }
        ```
        - ② 处加入的释放栅栏和 ⑤ 处的获取栅栏形成同步。
        - 请注意，我们加入的两个栅栏都有必要：一个线程需要进行释放操作，另一个线程则需进行获取操作，唯有配对才可以构成同步关系。
        - 上例中，释放栅栏 ② 的作用是令变量 y 的存储操作不再服从 memory_order_relaxed 次序，如改用了 memory_order_release 一样。类似地，我们还加入了获取栅栏 ⑤，变量 y 的载入操作遂如改用了次序 memory_order_acquire 一样。
        - 栅栏的整体运作思路是：若存储操作处于释放栅栏后面，而存储操作的结果为获取操作所见，则该释放栅栏与获取操作同步；若载入操作处于获取栅栏前面，而载入操作见到了释放操作的结果，则该获取栅栏与释放栅栏同步。
        - 尽管栅栏之间的同步取决于其前后的读写操作，但我们一定要明白，同步点是栅栏本身。栅栏只有放置在变量 x 和 y 的存储操作之间，才会强制这两个操作服从先后次序。就其他原子操作之间的先行关系而言，栅栏存在与否并不影响已经加诸其上的次序。
    6. 我们利用原子操作强制施行内存次序，其中真正的奥妙在于它们可以强制非原子操作服从一定的内存次序，并避免因数据竞争而引发的未定义行为。    

22. 强制非原子操作服从内存次序
    1. 先行关系中蕴含着控制流程的先后执行顺序。
    2. lock() 的实现方式是在循环中反复调用 flag.test_and_set()，其中所采用的次序为 std::memory_order_acquire；unlock() 实质上则是服从 std::memory_order_release 次序的 flag.clear() 操作。
        1. 第一个线程调用 lock() 时，标志 flag 正处于置零状态，test_and_set() 的第一次调用会设置标志成立并返回 false，表示负责执行的线程已获取了锁，遂循环结束，互斥随即生效。该线程可修改受其保护的数据而不受干扰。此时标志已设置成立，如果任何其他线程再调用 lock()，都会在 test_and_set() 所在的循环中阻塞。
        2. 当持锁线程完成了受保护数据的改动，就调用 unlock()，再进一步按 std::memory_order_release 次序语义执行 flag.clear()。
        3. 若第二个线程因调用 lock() 而反复执行 flag.test_and_set()，又因该操作采用了 std::memory_order_acquire 次序语义，故标志 flag 上的这两项操作形成同步。
    3. 尽管其他互斥实现的内部操作各有不同，但其基本原理都一样：lock() 与 unlock() 都是某内部内存区域之上的操作，前者是获取操作，后者则是释放操作。
    4. 条件变量并不提供任何同步关系。它们本质上是忙等循环的优化，其所有同步功能都由关联的互斥提供。
