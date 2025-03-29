###### 一、C++ 并发世界

1. 同一进程内的所有进程都共用相同的地址空间，且所有线程都能直接访问大部分数据。
2. 每个线程都需要独立的栈空间，如果线程太多，就可能耗尽所属进程的可用内存或地址空间。
3. 管控线程的函数和类在<thread>中声明，而有关共享数据保护的声明则位于别的头文件中。
4. 每个线程都需要一个起始函数，新线程从这个函数开始执行。就应用程序的起始线程而言，该函数是main()。对于别的线程，其起始函数需要在std::thread对象的构造函数中指明。
5. 新线程启动后，起始线程继续执行。如果起始线程不等待新线程结束，就会一路执行，直到main()结束，甚至很可能直接终止整个程序，新线程根本没有机会启动。这正是要调用join()的原因，该调用会令主线程等待子线程。

---

###### 二、线程管控

1. 每个 C++ 程序都含有至少一个线程，即运行main()的线程，它由 C++ 运行时（C++ runtime）系统启动。

2. 发起线程
    1. 任何可调用类型都适用于std::thread。包括函数指针、函数对象、lambda等，能让使用者对其进行函数调用操作。
        ```
        class Func
        {
        public:
            void operator() () const
            {
                do_something();
            }
        };
        Func f;
        std::thread myThread(f);
        
        // std::thread myThread(Func());    // 会被解释成函数声明(接收的参数是函数指针，所指向的函数没有参数传入，返回Func对象)
        std::thread myThread((Func()));
        std::thread myThread{Func()};
        
        std::thread myThread([]{
            do_something();
        });
        ```
        - 上述代码提供了函数对象f作为参数，它被复制到属于新线程的存储空间中，并在那里被调用，由新线程执行。
        - 针对存在二义性的 C++ 语句，只要它有可能被解释成函数声明，编译器就肯定将其解释成函数声明。为临时函数对象命名即可解决问题，做法是多用一对圆括号，或采用新式的统计初始化语法(列表初始化)。
    2. 调用 detach() 明确设定不等待新线程结束。
        - 主线程的函数退出后，新线程可能仍在运行，而主线程的函数却已结束。
        - 意图在函数中创建线程，并让线程访问函数的局部变量。除非线程肯定会在该函数退出前结束，否则切勿这么做。

3. 等待线程完成
    1. 若需等待线程完成，那么可以在与之关联的 std::thread 实例上，通过调用成员函数 join() 实现。
    2. 对于某个给定的线程，join() 仅能调用一次；只要 std::thread 对象曾经调用过 join()，线程就不再可汇合（joinable），成员函数 joinable() 将返回 false。

4. 在出现异常的情况下等待
    1. 在 std::thread 对象被销毁前，我们需确保已经调用 join() 或 detach()。
    2. 利用 RAII 过程等待线程完结
        ```
        class thread_guard
        {
            std::thread& t;
        public:
            explicit thread_guard(std::thread& t_) : t(t_) {}
            ~thread_guard()
            {
                if(t.joinable())
                    t.join();
            }
            thread_guard(thread_guard const&) = delete;
            thread_guard& operator=(thread_guard const&) = delete;
        };
        struct func     // 函数对象
        {
            int& i;
            func(int& i_) : i(i_) {}
            void operator() ()
            {
                for(unsigned j=0; j<1000000; ++j)
                {
                    do_something(i);
                }
            }
        }
        void f()
        {
            int some_local_state = 0;
            func my_func(some_local_state);
            std::thread t(my_func);
            thread_guard g(t);
            do_something_in_current_thread();
        }
        ```
        - 当主线程执行到 f() 末尾时，按构建的逆序，全体局部对象都会被销毁。因此类型 thread_guard 对象 g 首先被销毁，在其析构函数中，新线程汇合。即便 do_something_in_current_thread() 抛出异常，函数 f() 退出，以上行为仍然会发生。
        - 分离操作会切断线程和 std::thread 对象间的关联。这样，std::thread 对象在销毁时肯定不调用 std::terminate()，即便线程还在后台运行，也不会调用。

5. 在后台运行线程
    1. 分离的线程确实仍在后台运行，其归属权和控制权都转移给 C++运行时库（runtime library，又名运行库），由此保证，一旦线程退出，与之关联的资源都会被正确回收。
    2. 只有当 t.joinable() 返回 true 时，我们才能调用 t.detach()。

6. 向线程函数传递参数
    1. 线程具有内部存储空间，参数会按照默认方式先复制到该处，新创建的执行线程才能直接访问它们。然后，这些副本被当成临时变量，以右值形式传给新线程上的函数或可调用对象。即便函数的相关参数按设想应该是引用，上述过程依然会发生。
    2. 右值即 rvalue，是一类表达式，本质是固定不变的“值”（相对变量之“可变”），不接受赋值，只出现在赋值表达式等号右方。它的特点是没有标识符，无法提取地址，往往是临时变量或匿名变量，仅在其所属语句中有效，满足移动语义。
        ```
        void f(int i, std::string const& s);            // 注意这里是 const
        void not_oops(int some_param)
        {
            char buffer[1024];
            sprintf(buffer, "%i", some_param);
            std::thread t(f, 3, std::string(buffer));   // 使用 std:string 避免悬空指针
            t.detach();
        }
        ```
    3. 若需按引用方式传递参数，只要用 std::ref() 函数加以包装即可。
        ```
        void update_data_for_widget(widget_id w, widget_data& data);
        void oops_again(widget_id w)
        {
            widget_data data;
            // std::thread t(update_data_for_widget, w, data);      编译失败
            std::thread t(update_data_for_widget, w, std::ref(data));
            t.join();
        }
        ```
        - 根据 update_data_for_widget() 函数的声明，第二个采纳数会以引用方式传入，可是 std::thread 的构造函数对此却并不知情，它全然忽略 update_data_for_widget() 函数所期望的参数类型，直接复制我们提供的值。然而，线程库的内部代码会把参数的副本（t构造时由对象 data 复制得出，位于新线程的内部存储空间）当成 move-only 型别（只能移动，不可复制），并以右值的形式传递。最终，update_data_for_widget()函数调用会收到一个右值作为参数，这会导致编译失败。
        - std::ref(data) 包装之后，传入 update_data_for_widget() 函数的就不是变量data的临时副本，而是指向变量data的引用，代码遂能成功编译。
    4. 若要将某个类的成员函数设定为线程函数，我们则应传入一个函数指针，指向该成员函数。此外，我们还要给出合适的对象指针，作为函数的第一个参数。
        ```
        class X
        {
        public:
            void do_lengthy_work();
        };
        X my_x;
        std::thread t(&X::do_lengthy_work, &my_x);
        ```
    5. 通过移动构造函数和移动赋值操作符，对象的归属权就得以在多个 std::unique_ptr 实例间转移。若移动行为令 std::unique_ptr 的源对象的值变成 NULL 指针。函数可以接收这种类型的对象作为参数，也能将它作为返回值，充分发挥其可移动特性，以提升性能。若源对象是临时变量，移动就会自动发生。若源对象是具名变量，则必须通过调用 std::mpve() 直接请求转移。
        ```
        // 向线程转移动态对象的归属权：
        void process_big_object(std::unique_ptr<big_object>);
        std::unique_ptr<big_object> p(new big_object);
        p->prepare_data(42);
        std::thread t(process_big_object, std::move(p));
        ```
        - std::move(p) 未进行移动操作，仅仅将 p 转移成右值；创建线程时，发生一次转移（由线程库内部进行复制）；process_big_object() 开始执行时，再次转移（函数调用复制参数）。
    6. 一般地，在调用类的非静态成员函数时，编译器会隐式添加一参数，它是所操作对象的地址，用于绑定对象和成员函数，并且位于所有其他实际参数之前。例如，类 example 具有成员函数 func(int x)，而 obj 是该类的对象，则调用 obj.func(2) 等价于调用 example::func(&obj, 2)。
    7. 因为 std::thread 类的实例能够移动却不能赋值，故此线程的归属权可以在其实例之间转移。这就保证了，对于任一特定的执行线程，任何时候都只有唯一的 std::thread 对象与之关联，还准许程序员在其对象之间转移线程归属权。

7. 移交线程归属权
    1. 线程归属权在实例之间多次转移的例子：
        ```
        void some_function();
        void some_other_function();
        std::thread t1(some_function);      // 启动新线程，并使之关联t1
        std::thread t2 = std::move(t1);
        t1 = std::thread(some_other_function);  // 下面说明①
        std::thread t3;
        t3 = std::move(t2);
        t1 = std::move(t3);     // 该赋值操作会终止整个程序，下面说明②
        ```
        - 构造t2，在其初始化过车中调用 std::move()，将新线程的归属权显式地转移给t2。std::move()仅仅将t1强制转换成右值，但没有进行移动；真正触发移动行为的是t2的初始化。
        - ①处：启动新线程，它与一个 std::thread 类型的临时对象关联。新线程的归属权随即转移给t1。这里无须显示调用std::move()，因为新线程本来就由临时变量持有，而源自临时对象的移动操作会自动地隐式进行。
        - ②处：在转移之时，t1已经关联运行 some_other_function() 的线程。因此 std::terminate() 会被调用，终止整个程序。该调用在 std::thread 的析构函数中发生，目的是保持一致性。在 std::thread 对象析构前，我们必须明确：是等待线程完成还是要与之分离。不然，便会导致关联的线程终结。只要 std::thread 对象正管控着一个线程，就不能简单地向它赋新值，否则该线程会因此被遗弃。
        - std::thread 支持移动操作的意义是，函数可以便捷地向外部转移线程的归属权。
    2. 只要其执行析构函数，线程即能自动汇合。如下 joining_thread 类：
        ```
        class joining_thread
        {
            std::thread t;
        public:
            joining_thread() noexcept = default;
            template<typename Callable, typename ... Args>
            explicit joining_thread(Callable&& func, Args&& ... args)
                : t(std::forward<Callable>(func), std::forward<Args>(args)...)
            {}
            
            explicit joining_thread(std::thread t_) noexcept
                : t(std::move(t_))
            {}
            
            joining_thread(joining_thread&& other) noexcept
                : t(std::move(other.t))
            {}
            
            joining_thread& operator=(joining_thread&& other) noexcept
            {
                if(joinable())
                    join();
                t = std::move(other.t);
                return *this;
            }
            
            joining_thread& operator=(std::thread other) noexcept
            {
                if(joinable())
                    join();
                t = std::move(other);
                return *this;
            }
            
            ~joining_thread() noexcept
            {
                if(joinable())
                    join();
            }
            
            void swap(joining_thread& other) noexcept
            {
                t.swap(other.t);
            }
            
            std::thread::id get_id() const noexcept
            {
                return t.get_id();
            }
            
            bool joinable() const noexcept
            {
                return t.joinable();
            }
            
            void join()
            {
                t.join();
            }
            
            void detach()
            {
                t.detach();
            }
            
            std::thread& as_thread() noexcept
            {
                return t;
            }
            
            const std::thread& as_thread() const noexcept
            {
                return t;
            }
        };
        ```
    3. 生成多个线程，并等待它们完成运行
        ```
        void do_work(unsigned id);
        void f()
        {
            std::vector<std::thread> threads;
            for(unsigned i=0; i<20; ++i)
            {
                threads.emplace_back(do_work, i);
            }
            for(auto& entry : threads)
                entry.join();           // 依次在各线程上调用join()函数
        }
        ```
        - 若要运行多线程切分某算法的运算任务，往往要采取以上方式；必须等所有线程完成运行后，运行流程才能返回到调用者。

8. C++ 标准库的 std::thread::hardware_concurrency() 函数，它的返回值是一个指标，表示程序在各次运行中可真正并发的线程数量。这仅仅是一个指标，若信息无法获取，该函数则可能返回0。

9. 识别线程
    1. 线程 ID 所属型别是 std::thread::id，它有两种获取方法。首先，在与线程关联的 std::thread 对象上调用成员函数 get_id()，即可得到该线程的 ID。如果 std::thread 对象没有关联任何执行线程，调用 get_id() 则返回一个 std::thread::id 对象，它按默认构造方式生成，表示“线程不存在”。其次，当前线程的 ID 可以通过调用 std::this_thread::get_id() 获得，函数定义位于头文件<thread>内。
    2. 如果两个 std::thread::id 型别的对象相等，则它们表示相同的线程，或者它们的值都表示“线程不存在”。

---

###### 三、在线程间共享数据

1. 用互斥保护共享数据
    1. 在 C++ 中，我们通过构造 std::mutex 的实例来创建互斥，调用成员函数 lock() 对其加锁，调用 unlock() 解锁。
    2. C++ 标准库提供了类模板 std::lock_guard<>，针对互斥类融合实现了 RAII 手法：在构造时给互斥加锁，在析构时解锁，从而保证互斥总被正确解锁。
    3. std::mutex 和 std::lock_guard<> 都在头文件<mutex>里声明。
    4. C++17 引入了一个新特性，名为类模板参数推断，对于 std::lock_guard<> 这种简单的类模板，模板参数列表可以忽略。例如 “std::lock_guard<std::mutex> guard(some_mutex);”可以简化成“std::lock_guard guard(some_mutex);”。
    5. C++17 还引入了 std::scoped_lock，它是增强版的 lock_guard。例如，“std::scoped_lock guard(some_mutex);”。
    6. 如果成员函数返回指针或引用，指向受保护的共享数据，那么即便成员函数全都按良好、有序的方式锁定互斥，仍然无济于事，因为保护已被打破，出现了大漏洞。只要存在任何能访问该指针和引用的代码，它就可以访问受保护的共享数据(也可以修改)，而无须锁定互斥。 
    7. 不得向锁所在的作用域之外传递指针和引用，指向受保护的共享数据，无论是通过函数返回值将它们保存到对外可见的内存，还是将它们作为参数传递给使用者提供的函数。
    8. 在空栈上调用 top() 会导致未定义行为。
    9. 利用 std::is_nothrow_copy_constructible 和 std::is_nothrow_move_constructible 这两个型别特征（type trait），便可在编译期对某个型别进行判断，确定它是否含有不抛出异常的拷贝构造函数，以及是否含有不抛出异常的移动构造函数。
    10. std::shared_ptr<> 在<memory>头文件内定义。

2. 线程安全的栈容器类
    ```
    #include <exception>
    #include <memory>
    #include <mutex>
    #include <stack>
    struct empty_stack : std::exception
    {
        const char* what() const throw();
    }
    template<typename T>
    class threadsafe_stack
    {
    private:
        std::stack<T> data;
        mutable std::mutex m;
    public:
        threadsafe_stack() {}
        threadsafe_stack(const threadsafe_stack& other)
        {
            std::lock_guard<std::mutex> lock(other.m);
            data = other.data;  // 在构造函数的函数体内进行复制操作
        }
        threadsafe_stack& operator=(const threadsafe_stack&) = delete;
        void push(T new_value)
        {
            std::lock_guard<std::mutex> lock(m);
            data.push(std::move(new_value));
        }
        std::shared_ptr<T> pop()
        {
            std::lock_guard<std::mutex> lock(m);
            if(data.empty()) throw empty_stack();
            std::shared_ptr<T> const res(std::make_shared<T>(data.top()));
            data.pop();
            return res;
        }
        void pop(T& value)
        {
            std::lock_guard<std::mutex> lock(m);
            if(data.empty()) throw empty_stack();
            value = data.top();
            data.pop();
        }
        bool empty() const
        {
            std::lock_guard<std::mutex> lock(m);
            return data.empty();
        }
    }
    ```
    - 我们没采用成员初始化列表，而是在构造函数的函数体内进行复制操作，从而保证互斥的锁定会横跨整个复制过程。
    - 若要保护同一个类的多个独立的实例，则应该分别使用多个互斥。

3. 防范死锁的建议通常是，始终按相同顺序对两个互斥加锁。

4. 运用 std::lock() 函数和 std::lock_guard<> 类模板，进行内部数据的互换操作
    ```
    class some_big_object;
    void swap(some_big_object& lhs, some_big_object& rhs);
    class X
    {
    private:
        some_big_object some_detail;
        std::mutex m;
    public:
        X(some_big_object const& sd) : some_detail(sd) {}
        friend void swap(X& lhs, X& rhs)
        {
            if(&lhs == &rhs)
                return;
            std::lock(lhs.m, rhs.m);
            std::lock_guard<std::mutex> lock_a(lhs.m, std::adopt_lock);
            std::lock_guard<std::mutex> lock_b(rhs.m, std::adopt_lock);
            swap(lhs.some_detail, rhs.some_detail);
        }
    }
    
    // C++17 的写法：
    void swap(X& lhs, X& rhs)
    {
        if(&lhs == &rhs)
            return;
        std::scoped_lock guard(lhs.m, rhs.m);   // 类模板参数推断
        swap(lhs.some_detail, rhs.some_detail);
    }
    ```
    - C++ 标准库提供了 std::lock() 函数，它可以同时锁住多个互斥，而没有发生死锁的风险。
    - 若我们已经在某个 std::mutex 对象上获取锁，那么再次试图从该互斥获取锁将导致未定义行为（std::recursive_mutex 类型的互斥准许同一线程重复加锁）。
    - 提供 std::adopt_lock 对象以指明互斥已被锁住，即互斥上有锁存在，std::lock_guard 实例应当据此接收锁的归属权，不得在构造函数内试图另行加锁。
    - C++17 还进一步提供了新的 RAII 类模板 std::scoped_lock<>。std::scoped_lock<> 和 std::lock_guard<> 完全等价，只不过前者是可变参数模板，接收各种互斥型别作为模板参数列表，还以多个互斥对象作为构造函数的参数列表。
    - C++17 具有隐式类模板参数推导机制，依据传入构造函数的参数对象自动匹配，选择正确的互斥型别。

5. 防范死锁的补充准则
    1. 避免嵌套锁
        - 万一确有需要获取多个锁，我们应采用 std::lock() 函数，借单独的调用动作一次获取全部锁来避免死锁。
    2. 一旦持锁，就须避免调用由用户提供的程序接口
    3. 依从固定顺序获取锁
        - 如果多个锁是绝对必要的，却无法通过 std::lock() 在一步操作中全部获取，我们只能退而求其次，在每个线程内部都依次固定顺序获取这些锁。
    4. 按层级加锁
        - 锁的层级划分就是按特定方式规定加锁次序，在运行期据此查验加锁操作是否遵从预设规则。
        - 若某线程已对低层级互斥加锁，则不准它再对高层级互斥加锁。具体做法是将层级的编号赋予对应用级应用程序上的互斥，并记录各线程分别锁定了哪些互斥。
        - 层级互斥之间不可能发生死锁，因为互斥自身已经被强制限定了加锁次序。
        - 举例示范两个线程如何运用层级互斥：
            ```
            hierarchical_mutex high_level_mutex(10000);
            hierarchical_mutex low_level_mutex(5000);
            hierarchical_mutex other_mutex(6000);
            
            int do_low_level_stuff();
            int low_level_func()
            {
                std::lock_guard<hierarchical_mutex> lk(low_level_mutex);
                return do_low_level_stuff();
            }
            void high_level_stuff(int some_param);
            int high_level_func()
            {
                // 先对 high_level_mutex 加锁，随后调用 low_level_func()，这两步符合加锁规则，因为互斥 high_level_mutex 所在的层级 10000 高于 low_level_mutex 所在的层级 5000。
                std::lock_guard<hierarchical_mutex> lk(high_level_mutex);
                return high_level_stuff(low_level_func());
            }
            void thread_a()
            {
                high_level_func();
            }
            
            void do_other_stuff();
            void other_stuff()
            {
                high_level_func();
                do_other_stuff();
            }
            void thread_b() // 线程 b 在运行期出错。
            {
                std::lock_guard<hierarchical_mutex> lk(other_mutex);
                other_stuff();
            }
            ```
            - 依据用户自定义的互斥型别，将类模板 std::lock_guard<> 具体化。尽管 hierarchical_mutex 类属于用户自定义的型别，但也能与 std::lock_guard<> 结合使用，因其实现了3个成员函数——lock()、unlock()和try_lock()，满足了互斥概念所具备的操作。
            - try_lock()：若另一线程已在目标互斥上持有锁，则函数立即返回 false，完全不等待。
        - 简单的层级互斥的代码：
            ```
            class hierarchical_mutex
            {
                std:mutex internal_mutex;
                unsigned long const hierarchy_value;
                unsigned long previous_hierarchy_value;
                static thread_local unsigned long this_thread_hierarchy_value;
                void check_for_hierarchy_violation()
                {
                    if(this_thread_hierarchy_value <= hierarchy_value)
                    {
                        throw std::logic_error("mutex hierarchy violated");
                    }
                }
                
                void update_hierarchy_value()
                {
                    previous_hierarchy_value = this_thread_hierarchy_value;
                    this_thread_hierarchy_value = hierarchy_value;
                }
                
            public:
                explicit hierarchical_mutex(unsigned long value)
                    : hierarchy_value(value)
                    , previous_hierarchy_value(0)
                {}
                void lock()
                {
                    check_for_hierarchy_violation();
                    internal_mutex.lock();
                    update_hierarchy_value();
                }
                void unlock()
                {
                    if(this_thread_hierarchy_value != hierarchy_value)
                        throw std::logic_error("mutex hierarchy violated");
                    this_thread_hierarchy_value = previous_hierarchy_value; // 线程的层级按保存的值复原。
                    internal_mutex.unlock();
                }
                bool try_lock()
                {
                    check_for_hierarchy_violation();
                    if(!internal_mutex.try_lock())
                        return false;
                    update_hierarchy_value();
                    return true;
                }
            };
            thread_local unsigned long hierarchical_mutex::this_thread_hierarchy_value(ULONG_MAX);
            ```
            - 使用线程专属的变量(名为 this_thread_hierarchy_value，以关键字 thread_local 修饰)表示当前线程的层级编号。线程自身不属于任何层级，this_thread_hierarchy_value 的准确意义是，当前线程最后一次加锁操作所牵涉的层级编号。
    5. 将准则推广到锁操作以外
        - 任何同步机制导致的循环等待都会导致死锁出现。

6. 运用 std::unique_lock<> 灵活加锁
    1. std::unique_lock 对象不一定始终占用与之关联的互斥。首先，其构造函数接收第二个参数：我们可以传入 std::adopt_lock 实例，借此指明 std::unique_lock 对象管理互斥上的锁；也可以传入 std::defer_lock 实例，从而使互斥在完成构造时处于无锁状态，等以后有需要时才在 std::unique_lock 对象（不是互斥对象）上调用 lock() 而获取锁，或把 std::unique_lock 对象交给 std::lock() 函数加锁。
    2. std::unique_lock 占用更多的空间，也比 std::lock_guard 略慢。但 std::unique_lock 对象可以不占有关联的互斥，具备这份灵活性需要付出代价：需要存储并且更新互斥信息。
        ```
        void swap(X& lhs, X& rhs)
        {
            if(&lhs == &rhs)
                return;
            // 实例 std::defer_lock 将互斥保留为无锁状态
            std::unique_lock<std::mutex> lock_a(lhs.m, std::defer_lock);
            std::unique_lock<std::mutex> lock_b(rhs.m, std::defer_lock);
            std::lock(lock_a, lock_b);  // 到这里才对互斥加锁
            swap(lhs.some_detail, rhs.some_detail);
        }
        ```
    3. 因为 std::unique_lock 类具有成员函数 lock()、try_lock() 和 unlock()，所以它的实例得以传给 std::lock() 函数。std::unique_lock 实例还含有一个内部标志，亦随着这些函数的执行而更新，以表明关联的互斥目前是否正被该类的实例占据。这一标志必须存在，作用是保证析构函数正确调用 unlock()。此标志可以通过调用成员函数 owns_lock() 查询。不过，若条件允许，最好还是采用 C++17 所提供的变参模板类 std::scoped_lock，除非我们必须采用 std::unique_lock 类进行某些操作，如转移锁的归属权。
    4. 如果 std::lock_guard 已经能满足所需，建议优先采用。若需要延时加锁或者需要从某些作用域转移锁的归属权到其他作用域，则 std::unique_lock 类更为合适。

7. 在不同作用域之间转移互斥归属权
    1. 因为 std::unique_lock 实例不占有与之关联的互斥，所以随着其实例的转移，互斥的归属权可以在多个 std::unique_lock 实例之间转移。
    2. 转移会在某些情况下自动发生，譬如从函数返回实例时，但我们须针对别的情形显式调用 std::move()。本质上，这取决于移动数据的来源到底是左值还是右值。若是左值（lvalue，实实在在的变量或指向真实变量的引用），则必须显式转移，以免归属权意外地转移到别处；如果是右值（rvalue，某种形式的临时变量），归属权转移便会自动发生。
    3. std::unique_lock 属于可移动却不可复制的型别。
    4. 转移有一种用途：准许函数锁定互斥，然后把互斥的归属权转移给函数调用者，好让他在同一个锁的保护下执行其他操作。
        ```
        std::unique_lock<std::mutex> get_lock()
        {
            extern std::mutex some_mutex;
            std::unique_lock<std::mutex> lk(some_mutex);
            prepare_data();
            return lk;
        }
        void process_data()
        {
            std::unique_lock<std::mutex> lk(get_lock());
            do_something();
        }
        ```
        - 由于锁 lk 是 get_lock() 函数中声明的 std::unique_lock 局部变量，因此代码无须调用 std::move() 就能把它直接返回，编译器会妥善调用移动构造函数。
    5. 通道（gate way）类是一种利用锁转移的具体形式，锁的角色是其数据成员，用于保证只有正确加锁才能够访问受保护数据，而不再充当函数的返回值。这样，所有数据必须通过通道类访问：若想访问数据，则需先取得通道类的实例（由函数调用返回，如上例中的 get_lock()），再借它执行加锁操作，然后通过通道对象的成员函数才得以访问数据。我们在访问完成后销毁通道对象，锁便随之释放，别的线程遂可以重新访问受保护的数据。这类通道对象几乎是可移动的（只有这样，函数才有可能向外转移归属权），因此锁对象作为其数据成员也必须是可移动的。
    6. std::unique_lock 类十分灵活，允许它的实例在被销毁前解锁。其成员函数 unlock() 负责解锁操作，这与互斥一致。

8. 按适合的粒度加锁
    1. std::unique_lock 应用：假如代码不再需要访问共享数据，那我们就调用 unlock() 解锁；若以后需重新访问，则调用 lock() 加锁。
    2. 一般地，若要执行某项操作，那我们应该只在所需的最短时间内持锁。换言之，除非绝对必要，否则不得在持锁期间进行耗时的操作，如等待 I/O 完成或获取另一个锁（即便我们知道不会死锁）。

9. 在初始化过程中保护共享数据
    1. 令所有线程共同调用 std::call_once() 函数，从而确保在该调用返回时，指针初始化由其中某线程安全且唯一地完成（通过适合的同步机制）。必要的同步数据则由 std::once_flag 实例存储，每个 std::once_flag 实例对应一次不同的初始化。相比显式使用互斥，std::call_once() 函数的额外开销往往更低，特别是在初始化已经完成的情况下，所以如果功能符合需求就应优先使用。
        ```
        // 用互斥实现线程安全的延迟初始化：
        std::shared_ptr<some_resource> resource_ptr;
        std::mutex resource_mutex;
        void foo()
        {
            std::unique_lock<std::mutex> lk(resource_mutex);
            if(!resource_ptr)
            {
                resource_ptr.reset(new some_resource);
            }
            lk.unlock();
            resource_ptr->do_something();
        }
        
        // std::once_flag 类和 std::call_once() 函数的方式：
        std::shared_ptr<some_resource> resource_ptr;
        std::once_flag resource_flag;
        void init_resource()
        {
            resource_ptr.reset(new some_resource);
        }
        void foo()
        {
            std::call_once(resource_flag, init_resource);   // 初始化函数准确地被唯一一次调用
            resource_ptr->do_something();
        }
        ```
    2. 利用 std::call_once() 函数对类 X 的数据成员实施线程安全的延迟初始化
        ```
        class X
        {
        private:
            connection_info connection_details;
            connection_handle connection;
            std::once_flag connection_init_flag;
            void open_connection()
            {
                connection = connection_manager.open(connection_details);
            }
        public:
            X(connection_info const& connection_details_)
                : connection_details(connection_details_)
            {}
            void send_data(data_packet const& data)
            {
                std::call_once(connection_init_flag, &X::open_connection, this);
            }
            data_packet receive_data()
            {
                std::call_once(connection_init_flag, &X::open_connection, this);
                return connection.receive_data();
            }
        }
        ```
        - 借助成员函数 open_connection() 初始化数据，而该函数必须用到 this 指针，所以要向其传入 this 指针。标准库的某些函数接收可调用对象，如 std::thread 的构造函数和 std::bind() 函数，std::call_once() 同样如此。
        - std::once_flag 的实例既不可复制也不可移动，这与 std::mutex 类似。

    3. 如果把局部变量声明成静态数据，那样便有可能让初始化过程出现条件竞争。根据 C++ 标准规定，只要控制流程第一次遇到静态数据的声明语句，变量即进行初始化。若多个线程同时调用同一函数，而它含有静态数据，则任意线程均可能首先到达其声明处，这就形成了条件竞争的隐患。C++11 解决了这个问题，规定初始化只会在某一线程上单独发生，在初始化完成之前，其他线程不会越过静态数据的声明而继续运行。
    
    4. 某些类的代码只需用到唯一一个全局实例，这种情形可用以下方法代替 std::call_once():
        ```
        class my_class;
        my_class& get_my_class_instance()   // 线程安全的初始化，C++11 标准保证其正确性
        {
            static my_class instance;
            return instance;
        }
        ```
        - 多个线程可以安全地调用 get_my_class_instance()，而无须担忧初始化的条件竞争。
    
10. 保护甚少更新的数据结构
    1. 读写互斥：允许单独一个“写线程”进行完全排他的访问，也允许多个“读线程”共享数据或并发访问。
    2. C++ 17 标准库提供了两种新的互斥：std::shared_mutex 和 std::shared_timed_mutex。C++ 14 标准库只有 std::shared_timed_mutex，而 C++ 11 标准库都没有。std::shared_mutex 和 std::shared_timed_mutex 的区别在于，后者支持更多操作。所以，若无须进行额外操作，则应选用 std::shared_mutex，其在某些平台上可能会带来性能增益。
    3. 更新操作可用 std::lock_guard<std::shared_mutex> 和 std::unique_lock<std::shared_mutex> 锁定，代替对应的 std::mutex 特化。它们和 std::mutex 一样，都保证了访问的排他性质。对于那些无须更新数据结构的线程，可以另行改用共享锁 std::shared_lock<std::shared_mutex> 实现共享访问。
    4. C++ 14 引入了共享锁的类模板，其工作原理是 RAII 过程，使用方式则与 std::unique_lock 相同，只不过多个线程能够同时锁住同一个 std::shared_mutex。共享锁仅有一个限制，即假设它已被某些线程所持有，若别的线程试图获取排他锁，就会发生阻塞，直到那些线程全都释放该共享锁。反之，如果任一线程持有排他锁，那么其他线程全都无法获取共享锁或排他锁，直到持锁线程将排他锁释放为止。
    5. 共享锁即读锁，对应 std::shared_lock<std::shared_mutex>；排他锁即写锁，对应 std::lock_guard<std::shared_mutex> 和 std::unique_lock<std::shared_mutex>。

11. 递归加锁
    1. C++ 标准库提供了 std::recursive_mutex，其工作方式与 std::mutex 相似，不同之处是，其允许同一线程对某互斥的同一实例多次加锁。我们必须先释放全部的锁，才可以让另一线程锁住该互斥。只要正确地使用 std::lock_guard<std::recursive_mutex> 和 std::unique_lock<std::recursive_mutex>，它们便会处理好递归锁的余下细节。
    2. 若要设计一个类以支持多线程并发访问，它就需包含互斥来保护数据成员，递归互斥常常用于这种情形。每个公有函数都需先锁住互斥，然后才进行操作，最后解锁互斥。但有时在某些操作过程中，公有函数需要调用另一公有函数。在这种情况下，后者将同样试图锁住互斥，如果采用 std::mutex 便会导致未定义行为。
    3. 但不建议全部用递归互斥代替普通互斥。我们通常可以采取更好的方法：根据这两个公有函数的共同部分，提取出一个新的私有函数，新函数由这两个公有函数调用，而它假定互斥已经被锁住，遂无须重复加锁。（具体改动步骤包括，将该类的内部互斥从 std::recursive_mutex 类型改为 std::mutex 类型，而原来的两个公有函数之间不发生任何调用，都转为调用新提取出的私有函数，并在调用在各自对 std::mutex 成员加锁）

---

###### 四、并发操作的同步

1. 线程间的同步操作很常见，C++ 标准库专门为之提供了处理工具：条件变量和 future。另外，并发技术规约还提供了新式的同步工具：线程闩（latch）和线程卡（barrier）。

2. 让线程休眠100毫秒：
    `std::this_thread::sleep_for(std::chrono::milliseconds(100));`

3. 凭借条件变量等待条件成立
    1. C++ 标准库提供了条件变量的两种实现：std::condition_variable 和 std::condition_variable_any。它们都在标准库的头文件<condition_variable>内声明。两者都需配合互斥，方能提供妥当的同步操作。std::condition_variable 仅限于与 std::mutex 一起使用；然而，只要某一类型符合称为互斥的最低标准，足以充当互斥，std::condition_variable_any 即可与之配合使用，因此它的后缀是“_any”。
    2. 由于 std::condition_variable_any 更加通用，它可能产生额外开销，涉及其性能、自身的体积或系统资源等，因此 std::condition_variable 应予优先采用，除非有必要令程序更灵活。
    3. 用 std::condition_variable 等待处理数据：
        ```
        std::mutex mut;
        std::queue<data_chunk> data_queue;
        std::condition_variable data_cond;
        void data_preparation_thread()      // 由线程乙运行
        {
            while(more_data_to_prepare())
            {
                data_chunk const data = prepare_data();
                {
                    std::lock_guard<std::mutex> lk(mut);
                    data_queue.push(data);
                }
                data_cond.notify_one();
            }
        }
        void data_processing_thread()       // 由线程甲运行
        {
            while(true)
            {
                std::unique_lock<std::mutex> lk(mut);
                data_cond.wait(lk, []{ return !data_queue.empty(); });
                data_chunk data = data_queue.front();
                dara_queue.pop();
                lk.unlock();
                process(data);
                if(is_last_chunk(data))
                    break;
            }
        }
        ```
        - 上述代码中的 lambda 函数用于表达需要等待成立的条件。
        - wait() 在内部调用传入的 lambda 函数，判断条件是否成立：若成立（lambda 函数返回 true），则 wait() 返回；否则（lambda 函数返回 false），wait() 解锁互斥，并令线程进入阻塞状态或等待状态。线程乙将数据准备好后，即调用 notify_one() 通知条件变量，线程甲随之从休眠中觉醒（阻塞解除），重新在互斥上获取锁，再次查验条件：若条件成立，则从 wait() 函数返回，而互斥仍被锁住；若条件不成立，则线程甲解锁互斥，并继续等待。
        - 我们舍弃 std::lock_guard 而采用 std::unique_lock，原因就在这里：线程甲在等待期间，必须解锁互斥，而结束等待之后，必须重新加锁，但 std::lock_guard 无法提供这种灵活性。
        - std::condition_variable::wait 本质上是忙等的优化。
        - std::unique_lock 可灵活解锁，wait() 的调用过程体现了这个特性。
    
4. 利用条件变量构建线程安全的队列
    ```
    #include <queue>
    #include <memory>
    #include <mutex>
    #include <condition_variable>
    template<typename T>
    class threadsafe_queue
    {
    private:
        mutable std::mutex mut;     // 互斥必须用 mutable 修饰（针对 const 对象，准许其数据成员发生变动）
        std::queue<T> data_queue;
        std::condition_variable data_cond;
    public:
        threadsafe_queue()
        {}
        threadsafe_queue(threadsafe_queue const& other)
        {
            std::lock_guard<std::mutex> lk(other.mut);
            data_queue = other.data_queue;
        }
        void push(T new_value)
        {
            std::lock_guard<std::mutex> lk(mut);
            data_queue.push(new_value);
            data_cond.notify_one();
        }
        void wait_and_pop(T& value)
        {
            std::unique_lock<std::mutex> lk(mut);
            data_cond.wait(lk, [this]{ return !data_queue.empty(); });
            value = data_queue.front();
            data_queue.pop();
        }
        std::shared_ptr<T> wait_and_pop()
        {
            std::unique_lock<std::mutex> lk(mut);
            data_cond.wait(lk, [this]{ return !data_queue.empty(); });
            std::shared_ptr<T> res(std::make_shared<T>(data_queue.front());
            data_queue.pop();
            return res;
        }
        bool try_pop(T& value)
        {
            std::lock_guard<std::mutex> lk(mut);
            if(data_queue.empty())
                return false;
            value = data_queue.front();
            data_queue.pop();
            return true;
        }
        std::shared_ptr<T> try_pop()
        {
            std::lock_guard<std::mutex> lk(mut);
            if(data_queue.empty())
                return std::shared_ptr<T>();
            std::shared_ptr<T> res(std::make_shared<T>(data_queue.front());
            data_queue.pop();
            return res;
        }
        bool empty() const
        {
            std::lock_guard<std::mutex> lk(mut);
            return data_queue.empty();
        }
    }
    ```
    - 由于互斥因锁操作而变化，因此它必须用关键字 mutable 修饰，这样才可以在 empty() 函数和拷贝构造函数中锁定。
    -条件变量也适用于多个线程都在等待同一个目标事件的情况。如果多个线程因执行 wait() 而同时等待， 每当有新数据就绪并加入 data_queue（见成员函数push()）时，notify_one() 的调用就会触发其中一个线程去查验条件，让它从 wait() 返回。此方式并不能确定会通知到具体哪个线程，甚至不能保证正好有线程在等待通知，因为可能不巧，负责数据处理的全部线程都尚未完工。
    - 如果几个线程都在等待同一个目标事件，那么还存在另一可能的行为方式：它们全部需要做出响应。以上行为会在两种情形下发生：一是共享数据的初始化，所有负责处理的线程都用到同一份数据，但都需要等待数据初始化完成；二是所有线程都要等待共享数据更新（如定期执行的重新初始化）。尽管条件变量适用于第一种情形，但我们可以选择其他更好的方式，如 std::call_once() 函数。前文中，负责准备的线程原本在条件变量上调用 notify_one()，而这里只需改为调用成员函数 notify_all()。顾名思义，该函数通知当前所有执行 wait() 而正在等待的线程，让它们去查验所等待的条件。

5. 使用 future 等待一次性事件发生
    1. C++ 标准程序库使用 future 来模拟这类一次性事件：若线程需等待某个特定的一次性事件发生，则会以恰当的方式取得一个 future，它代表目标事件；接着，该线程就能一边执行其他任务，一边在 future 上等待；同时，它以短暂的间隔反复查验目标事件是否已经发生。这个线程也可以转换运行模式，先不能等目标事件发生，直接暂缓当前任务，而切换到别的任务，及至必要时，才回头等待 future 准备就绪。future 可能与数据管理，也可能未关联。一旦目标事件发生，其 future 即进入就绪状态，无法重置。
    2. C++ 标准程序库有两种 future，分别由两个类模板实现，其声明位于标准库的头文件<future>内：独占 future（unique future，即 std::future<>）和共享 future（shared future，即 std::shared_future<>）。它们的设计参照了 std::unique_ptr 和 std::shared_ptr。
    3. 同一事件仅仅允许关联唯一一个 std::future 实例，但可以关联多个 std::shared_future 实例。只要目标事件发生，与后者关联的所有实例就会同时就绪，并且，它们全都可以访问与该目标事件关联的任何数据。关联数据正是两种 future 以模板形式实现的原因：模板参数就是关联数据的类型，这与 std::unique_ptr 和 std::shared_ptr 相似。如果没有关联数据，我们应使用特化的模板 std::future<void> 和 std::shared_future<void>。
    4. 虽然 future 能用于线程间通信，但是 future 对象本身不提供同步访问。若多个线程需访问同一个 future 对象，必须使用互斥或其他同步方式进行保护。不过，一个 std::shared_future<> 对象可能派生出多个副本，这些副本都指向同一个异步结果，由多个线程分别独占，它们可访问属于自己的那个副本而无须互相同步。

6. 从后台任务返回值
    1. 只要我们并不急需线程运算的值，就可以使用 std:async() 按异步方式启动任务。我们从 std:async() 函数处获得 std::future 对象（而非 std::thread 对象），运行的函数一旦完成，其返回值就由该对象最后持有。若要用到这个值，只需在 future 对象上调用 get()，当前线程就会阻塞，以便 future 准备妥当并返回该值。
    2. 运用 std::future 取得异步任务的函数返回值：
        ```
        #include <future>
        #include <iostream>
        int find_the_answer_to_ltuae();
        void do_other_stuff();
        int main()
        {
            std::future<int> the_answer = std::async(find_the_answer_to_ltuae);
            do_other_stuff();
            std::cout << "The answer is " << the_answer.get() << std::endl;
        }
        ```
        - 在调用 std:async() 时，它可以接收附加参数，进而传递给任务函数作为其参数，此方式与 std::thread 的构造函数相同。若要异步运行某个类的某成员函数，则 std:async() 的第一个参数应是一个函数指针，指向该类的目标成员函数；第二个参数需要给出相应的对象，以在它之上调用成员函数（这个参数可以是指向对象的指针，或对象本身，或由 std::ref 包装的对象）；余下的 std:async() 的参数会传递给成员函数，用作成员函数的参数。
        - 如果 std:async() 的参数是右值，则通过移动原始参数构建副本，与复制 std::thread 实例相同。这使得仅可移动的类型既能作为函数对象，又能充当 std:async() 的参数。
    3. 通过 std::async() 向任务函数传递参数：
        ```
        #include <string>
        #include <future>
        struct X
        {
            void foo(int, std::string const&);
            std::string bar(std::string const&);
        };
        X x;
        auto f1 = std::async(&X::foo, &x, 42, "hello"); // ①
        auto f2 = std::async(&X::bar, x, "goodbye");    // ②
        struct Y
        {
            double operator() (double);
        }
        Y y;
        auto f3 = std::async(Y(), 3.141);               // ③
        auto f4 = std::async(std::ref(y), 2.718);       // 调用 y(2.718)
        X baz(X&);
        std::async(baz, std::ref(x));                   // 调用 baz(x)
        class move_only
        {
        public:
            move_only();
            move_only(move_only&&);
            move_only(move_only const&) = delete;
            move_only& operator=(move_only&&)；
            move_only& operator=(move_only const&) = delete;
            void operator() ();
        };
        auto f5 = std::async(move_only());              // ④
        
        auto f6 = std::async(std::launch::async, Y(), 1.2);     // 运行新线程
        ayto f7 = std::async(std::launch::deferred, baz, std::ref(x));  // 在 wait() 或 get() 内部运行任务函数
        auto f8 = std::async(std::launch::deferred | std::launch::async, baz, std::ref(x));   // 交由实现自行选择运行方式
        auto f9 = std::async(baz, std::ref(x)); // 交由实现自行选择运行方式
        f7.wait();  // 任务函数调用被延后到这里才运行
        ```
        - ① 调用 p->foo(42, "hello")，其中 p 的值是 &x，即 x 的地址。
        - ② 调用 tmpx.bar("goodbye")，其中 tmpx 是 x 的副本。
        - ③ 调用 tmpy(3.141)。其中，由 Y() 生成一个匿名变量，传递给 std::async()，进而发生移动构造。在 std::async() 内部产生对象 tmpy，在 tmpy 上执行 Y::operator()(3.141)。
        - ④ 调用 tmp()，其中 tmp 等价于 std::move(move_only())，它的产生过车与 ③ 相似。
        - 按默认情况下，std::async() 的具体实现会自行决定——等待future时，是启动新线程，还是同步执行任务。我们也能给 std::async() 补充一个参数，以指定采用哪种运行方式。参数的类型是 std::launch，其值可以是 std::launch::deferred 或 std::launch::async。前者指定在当前线程上延后调用任务函数，等到 future 上调用了 wait() 或 get()，任务函数才会执行；后者指定必须另外开启专属的线程，在其上运行任务函数。该参数的值还可以是 std::launch::deferred | std::launch::async，表示由 std::async() 的实现自行选择运行方式。最后这项是参数的默认值。
        - std::launch 是 C++11 引入的枚举类，std::launch::deferred 和 std::launch::async 是其中的两个枚举常量。
    4. 使 std::future 和任务关联并非唯一的方法：运用类模板 std::packaged_task<> 的实例，我们也能将任务包装起来；又或者，利用 std::promise<> 类模板编写代码，显式地异步求值。与 std::promise 相比，std::packaged_task 的抽象层级更高。

7. 关联 future 实例和任务
    1. std::packaged_task<> 连结了 future 对象与函数（或可调用对象）。std::packaged_task<> 对象在执行任务时，会调用关联的函数（或可调用对象），把返回值保存为 future 的内部数据，并令 future 准备就绪。它可作为线程池的构件单元，亦可用于其他任务关联方案。
    2. std::packaged_task<> 是类模板，其模板参数是函数签名。假设，我们要构建 std::packaged_task<> 实例，那么，由于模板参数先行指定了函数签名，因此传入的函数（或可调用对象）必须与之相符，即它应接收指定类型的参数，返回值也必须可以转换为指定类型。这些类型不必严格匹配，若某函数接收 int 类型参数并返回 float 值，我们则可以为其构建 std::packaged_task<double(double)> 的实例，因为对应的类型可进行隐式转换。
    3. 类模板 std::packaged_task<> 具有成员函数 get_future()，它返回 std::future<> 实例，该 future 的特化类型取决于函数签名所指定的返回值。
    4. std::packaged_task<> 还具备函数调用操作符，它的参数取决于函数签名的参数列表。
        ```
        // 定义特化的 sstd::packaged_task<> 类模板（定义不完全，只列出部分代码）
        template<>
        class packaged_task<std::string(std::vector<char>*, int)>
        {
        public:
            template<typename Callable>
            explicit packaged_task(Callable&& f);
            std::future<std::string> get_future();
            void operator() (std::vector<char>*, int);
        }
        ```
    5. std::packaged_task 对象是可调用对象，我们可以直接调用，还可以将其包装在 std::function 对象内，当作线程函数传递给 std::thread 对象，也可以传递给需要可调用对象的函数。若 std::packaged_task 作为函数对象而被调用，它就会通过函数调用操作符接收参数，并将其进一步传递给包装在内的任务函数，由其异步运行得出结果，并将结果保存到 std::future 对象内部，再通过 get_future() 获取此对象。因此，为了在未来的适当时刻执行某项任务，我们可以将其包装在 std::packaged_task 对象内，取得对应的 future 之后，才把该对象传递给其他线程，由它触发任务执行。等到需要使用结果时，我们静候 future 准备就绪即可。
    6. 使用 std::packaged_task 在 GUI 的线程上运行代码：
        ```
        #include <deque>
        #include <mutex>
        #include <future>
        #include <thread>
        #include <utility>
        
        std::mutex m;
        std::deque<std::packaged_task<void()>> tasks;
        bool gui_shutdown_message_received();
        void get_and_process_gui_message();
        void gui_thread()
        {
            while(!gui_shutdown_message_received())
            {
                get_and_process_gui_message();
                std::packaged_task<void()> task;
                {
                    std::lock_guard<std::mutex> lk(m);
                    if(tasks.empty())
                        continue;
                    task = std::move(tasks.front());
                    tasks.pop_front();
                }
                task();
            }
        }
        std::thread gui_bg_thread(gui_thread);
        template<typename Func>
        std::future<void> post_task_for_gui_thread(Func f)
        {
            std::packaged_task<void()> task(f);
            std::future<void> res = task.get_future();
            std::lock_guard<std::mutex> lk(m);
            tasks.push_back(std::move(task));
            return res;
        }
        ```
        - 我们依据给定的函数创建新任务，将任务包装在内，并随即通过调用成员函数 get_future()，取得与该任务关联的 future。然后将任务放入任务队列，接着向 post_task_for_gui_thread 的调用者返回 future。接下来，有关代码向 GUI 线程投递消息，假如这些代码需判断任务是否完成，以获取结果进而采取后续操作，那么只要等待 future 就绪即可；否则，任务的结果不会派上用场，关联的 future 可被丢弃。
    7. 有一些任务的执行结果可能来自多个部分的代码，这种情况就需要运用第三种方法（第一种是 std::async()，第二种是 std::packaged_task<>）创建 future：借助 std::promise 显式地异步求值。

8. 创建 std::promise
    1. std::promise<T> 给出了一种异步求值的方法（类型为 T)，某个 std::future<T> 对象与结果关联，能延后读出需要求取的值。配对的 std::promise 和 std::future 可实现下面的工作机制：等待数据的线程在 future 上阻塞，而提供数据的线程利用相配的 promise 设定关联的值，使 future 准备就绪。
    2. 若需从给定的 std::promise 实例获取关联的 std::future 对象，调用前者的成员函数 get_future() 即可，这与 std::packaged_task 一样。promise 的值通过成员函数 set_value() 设置，只要设置好，future即准备就绪，凭借它就能获取该值。如果 std::promise 在被销毁时仍未曾设置值，保存的数据则由异常代替。
    3. 利用多个 promise 在单个线程中处理多个连接：
        ```
        #include <future>
        void process_connections(connection_set& connections)
        {
            while(!done(connections))
            {
                for(connection_iterator connection = connections.begin(), end = connections.end(); connection != end; ++connection)
                {
                    if(connection->has_incoming_data())
                    {
                        data_packet data = connection->incoming();
                        std::promise<payload_type>& p = connection->get_promise(data.id);
                        p.set_value(data.payload);
                    }
                    if(connection->has_outgoing_data())
                    {
                        outgoing_packet data = connection->top_of_outgoing_queue();
                        connection->send(data.payload);
                        data.promise.set_value(true);
                    }
                }
            }
        }
        ```
        - 此例运用了一对 std::promise<bool>/std::future<bool>，以确保数据包成功向外发送；与 future 关联的值是一个表示成败的简单标志。对于传入的数据报，与 future 关联的数据则是包内的有效荷载（payload）。
        - 这里，我们假定传入的数据包本身已含有 ID 和荷载数据。令每个 ID 与各 std::promise 对象（它们可能存储到关联容器中，利用查找而实现）一一对应，将其相关值设置为数据包的有效荷载。向外发送的数据包取自发送队列，并通过连接发出。只要发送完成，与向外发送数据关联的 promise 就会被设置为 true，示意数据发送成功。

9. 将异常保存到 future 中
    1. 若经由 std::async() 调用的函数抛出异常，则会被保存到 future 中，代替本该设定的值，future 随之进入就绪状态，等待其成员函数 get() 被调用，存储在内的异常即被重新抛出（C++ 标准没有明确规定应该重新抛出原来的异常，还是其副本；为此，不同的编译器和库有不同的选择）。假如我们把任务函数包装在 std::packaged_task 对象内，也依然如是。若包装的任务函数在执行时抛出异常，则会代替本应求得的结果，被保存到 future 内并使其准备就绪。只要调用 get()，该异常就会被再次抛出。
    2. std::promise 也具有同样的功能，它通过成员函数的显式调用实现。假如我们不想保存值，而想保存异常，就不应调用 set_value()，而应调用成员函数 set_exception()。若算法的并发实现会抛出异常，则该函数通常可用于其 catch 块中，捕获异常并装填 promise。
        ```
        extern std::promise<double> some_promise;
        try
        {
            some_promise.set_value(calculate_value());
        }
        catch(...)
        {
            some_promise.set_exception(std::current_exception());
        }
        ```
        - 这里的 std::current_exception() 用于捕获抛出的异常。此外，我们还能用 std::make_exception_ptr() 直接保存新异常，而不触发抛出行为：  
        `some_promise.set_exception(std::make_exception_ptr(std::logic_error("foo ")));`   
        假定我们能预知异常的类型，那么，相较 try/catch 块，后面的代替方法不仅简化了代码，还更有利于编译器优化代码，因而应优先采用。
    3. 还有另一方法可将异常保存到 future 中：我们不调用 promise 的两个 set() 成员函数，也不执行包装的任务，而直接销毁与 future 关联的 std::promise 对象或 std::packaged_task 对象。如果关联的 future 未能准备就绪，无论销毁两者中的哪一个，其析构函数都会将异常 std::future_error 存储为异步任务的状态数据，它的值是错误代码 std::future_errc::broken_promise。我们一旦创建 future 对象，便是许诺会按异步方式给出值或异常，但刻意销毁产生它们的来源，就无法提供所求的值或出现的异常，导致许诺被破坏。在这种情形下，倘若编译器不向 future 存入任何数据，则等待的线程有可能永远等不到结果。
    4. std::future_error 是一个枚举类，std::future_errc::broken_promise 是其中一个枚举常量。
    5. std::future 自身存在限制，关键问题是：它只容许一个线程等待结果。若我们要让多个线程等待同一个目标事件，则需改用 std::shared_future。

10. 多个线程一起等待 
    1. 只要同步操作是一对一地在线程间传递数据，std::future 就都能处理。然而，对于某个 future 实例，如果其成员函数由不同线程调用，它们却不会自动同步。若我们在多个线程上访问同一个 std::future 对象，而不采取额外的同步措施，将引发数据竞争并导致未定义行为。这是 std::future 特性：它模拟了对异步结果的独占行为，get() 仅能被有效调用唯一一次。这个特性令并发访问失去意义，只有一个线程可以获取目标值，原因是第一次调用 get() 会进行移动操作，之后该值不复存在。
    2. std::future 仅能移动构造和移动赋值，所以归属权可在多个实例之间转移，但在相同时刻，只会有唯一一个 future 实例指向特定的异步结果；std::shared_future 的实例则能复制出副本，因此我们可以持有该类的多个对象，它们全指向同一异步任务的状态数据。
    3. 即便改用 std::shared_future，同一个对象的成员函数却依然没有同步。若我们从多个线程访问同一个对象，就必须采取锁保护以避免数据竞争。首选方式是，向每个线程传递 std::shared_future 对象的副本，它们为各线程独自所有，并被视作局部变量。
    4. future 和 promise 都具备成员函数 valid()，用于判别异步状态是否有效。
    5. std::shared_future 的实例依据 std::future 的实例构造而得，前者所指向的异步状态由后者决定。因为 std::future 对象独占异步状态，其归属权不为其他任何对象所共有，所以若要按默认方式构造 std::shared_future 对象，则须用 std::move 向其默认构造函数传递归属权，这使 std::future 变成空状态（empty state）。
        ```
        std::promise<int> p;
        std::future<int> f(p.get_future());
        assert(f.valid());          // future 对象 f 有效
        std::shared_future<int> sf(std::move(f));
        assert(!f.valid());         // 对象 f 不再有效
        assert(sf.valid());         // 对象 sf 开始生效
        ```
    6. 与其他可移动对象相似，右值对象的归属权会进行隐式转换。因而，依据 std::promise 的成员函数 get_future() 的返回值，我们就能直接构造 std::shard_future 对象：
        ```
        std::promise<std::string> p;
        td::shared_future<std::string> sf(p.get_future());  // 隐式转移归属权
        ```
    7. std::future 还具有另一个特性，可以根据初始化列表自动推断变量的类型，从而使 std::shared_future 更便于使用。std::future 具有成员函数 share()，直接创建新的 std::shared_future 对象，并向它转移归属权。
        ```
        std::promise< std::map<SomeIndexType, SomeDataType, SomeComparator, SomeAllocator>::iterator> p;
        auto sf = p.get_future().share();
        ```

11. 限时等待
    1. 有两种超时机制可供选用：一是迟延超时，线程根据指定的时长而继续等待；二是绝对超时，在某特定时间点来临之前，线程一直等待。
    2. 大部分等待函数都具有变体，专门处理这两种机制的超时。处理迟延超时的函数变体以“_for”为后缀，而处理绝对超时的函数变体以“_until”为后缀。

12. 时钟类
    1. 每种时钟都是一个类，提供 4 项关键信息：
        - 当前时刻；
        - 时间值的类型（从该时钟取得的时间以它为表示形式）；
        - 该时钟的计时单元长度（tick period）；
        - 计时速率是否恒定，即能否将该时钟视为恒稳时钟（steady clock）。
    2. 若要获取某时钟类的当前时刻，调用其静态成员函数 now() 即可，例如，std::chrono::system_clock::now() 可返回系统时钟的当前时刻。
    3. 每个时钟类都具有名为 time_point 的成员类型，它是该时钟类自有的时间点类。据此，some_clock::now() 的返回值的类型就是 some_clock::time_point。
    4. 时钟类的计时单元属于名为 period 的成员类型，它表示为秒的分数形式：若时钟每秒计时 25 次，它的计时单元即为 std::ratio<1,25>；若时钟每隔 2.5 秒计数 1 次，则其计时单元为 std::ratio<5,2>。
    5. 若时钟的计时速率恒定（无论速率是否与计时单元相符）且无法调整，则称之为恒稳时钟。时钟类具有静态数据成员 is_steady，该值在恒稳时钟内为 true，否则为 false。通常，std::chrono::system_clock 类不是恒稳时钟，因为它可调整。即便这种调整自动发生，作用是消除本地系统时钟的偏差，依然可能导致：调用两次 now()，后来返回的时间值甚至早于前一个，结果违反恒定速率的规定。
    6. 恒稳时钟对于超时时限的计算至关重要，因此，C++ 标准库提供了恒稳时钟类 std::chrono::steady_clock。
    7. C++ 标准库还给出了其他时钟类：如前文提到的系统时钟类 std::chrono::system_clock，该类表示系统的“真实时间”，它具备成员函数 from_time_t() 和 to_time_t()，将 time_t 类型的值和自身的 time_point 值互相转化；还有高精度时钟类 std::chrono::high_resolution_clock，在 C++ 标准库提供的全部时钟类里，它具备可能实现的最短计时单元（因而具有可能实现的最高时间精度）。std::chrono::high_resolution_clock 可能不存在独立定义，而是由 typedef 声明的另一时钟类的别名。上述的时钟类都在标准库的头文件 <chrono> 中定义，此文件还包含其他时间工具。这些时钟类是完全相互独立的 4 个类，而不是“同一时钟模板类的 4 个不同具体化”。

13. 时长类
    1. std::chrono::duration<> 是标准库中最简单的时间部件（C++ 标准库用到不少处理时间的工具，它们全都位于名字空间 std::chrono 内）。它是类模板，具有两个模板参数，前者指明采用何种类型表示计时单元的数量（如 int、long 或 double），后者是一个分数，设定该时长类的每一个计时单元代表多少秒。例如，采用 short 值计数的分钟时长类是 std::chrono::duration<short, std::ratio<60,1>>，因为 1 分钟包含 60 秒；采用 double 值计数的毫秒时长类是 std::chrono::duration<double, std::ratio<1,1000>>，因为 1 毫秒是 1/1000 秒。
    2. 标准库在 std::chrono 名字空间中，给出了一组预设的时长类的 typedef 声明：nanoseconds、microseconds、milliseconds、seconds、minutes 和 hours，分别对应纳秒、微妙、毫秒、秒、分钟、小时。它们都采用取值范围足够大的整型表示计数。
    3. 针对国际单位制的词头倍数，头文件 <ratio> 给出了它们全部的 typedef 声明，范围从 std::atto（10^-18）到 std::exa（10^18）（只要平台支持 128 位整型，范围可以更大）。它们可以用于自定义时长，如 std::duration<double, std::centi>，其单元为百分秒，并以 double 值表示计数。
    4. C++14 引入了名字空间 std::chrono_literals，其中预定义了一些字面量后缀运算符。这能够缩短明文写入代码的时长值，例如：
        ```
        using namespace std::chrono_literals;
        auto one_day = 24h;
        auto half_an_hour = 30min;
        auto max_time_between_messages = 30ms;
        ```
        - 如果与整数字面值一起使用，这些后缀就相当于由 typedef 预设的时长类，因此，15ns 和 std::chrono::nanoseconds(15) 是两个相等的值。
    5. 假如不要求截断时长值，它们之间的转化将隐式进行（也就是，小时转化为秒是可行的，但秒转化为小时是不可行的）。显示转换通过 std::chrono::duration_cast<> 完成。
        ```
        std::chrono::milliseconds ms(54802);
        std::chrono::seconds s = std::chrono::duration_cast<std::chrono::seconds>(ms);
        ```
        - 上述的结果被截断，而非四舍五入，故此，时长变量 s 的值是 54。
    6. 时长类支持算术运算，我们将时长乘或除以一个数值（这个数值与该时长类的计数类型相符，即其第一个模板参数），或对两个时长进行加减，就能得出一个新时长。
    7. 计数单元的数量可通过成员函数 count() 获取，因而 std::chrono::milliseconds(1234).count() 得到 1234。
    8. 支持迟延超时的等待需要用到 std::chrono::duration<> 的实例：
        ```
        std::future<int> f = std::async(some_task);
        if(f.wait_for(std::chrono::milliseconds(35)) == std::future_status::ready)
            do_something_with(f.get());
        ```
        - 所有等待函数都返回一个状态值，指明是否超时或目标事件是否已发生。
        - 例子中，我们借助 future 进行等待，所以一旦超时，函数就返回 std::future_status::timeout；假如准备就绪，则函数返回 std::future_status::ready；若 future 的相关任务被延后，函数返回 std::future_status::deferred。
        - 迟延超时的等待需要一个参照标准，它采用了标准库内部的恒稳时钟，只要代码指定了等待 35 毫秒，那现实中等待的时间就是 35 毫秒，即使期间系统时钟发送调整（无论是提前还是延后）。
    
14. 时间点类
    1. 在时钟类中，时间点由类模板 std::chrono::time_point<> 的实例表示，它的第一个模板参数指明所参考的时钟，第二个模板参数指明计时单元（std::chrono::duration<> 的特化）。
    2. 时间点是一个时间跨度，始于一个称为时钟纪元的特定时刻，终于该时间点本身。跨度的值表示某具体时长的倍数。典型的时钟纪元包括 1970 年 1 月 1 日 0 时 0 分 0 秒，或运行应用的计算机的启动时刻。
    3. 时钟类内部具有一个 time_point 成员类型的 typedef。假设两个时钟类都参考同一时钟纪元，则可用该 typedef 指定跟另一个时钟类共用的 time_point 成员类型。
    4. 尽管时钟纪元的时刻无从得知，不过，我们可以在给定的时间点上调用 time_since_epoch()，这个成员函数返回一个时长对象，表示从时钟纪元到该时间点的时间长度。
    5. 我们可将时间点加减时长（即令 std::chrono::time_point<> 的实例加减 std::chrono::duration<> 实例），从而得出新的时间点。据此，std::chrono::high_resolution_clock::now() + std::chrono::nanoseconds(500) 会给出 500 纳秒以后的未来时刻。只要知道运行某段代码所允许的最大时限，我们就能方便地计算出绝对超时的时刻。
    6. 若两个时间点共享同一时钟，我们也可以将它们相减，得到的结果是两个时间点间的时长。这能用于代码计时：
        ```
        auto start = std::chrono::high_resolution_clock::now();
        do_something();
        auto stop = std::chrono::high_resolution_clock::now();
        std::cout << "do_something() took "
                  << std::chrono::duration<double, std::chrono::seconds>(stop-start).count()
                  << " seconds" << std::endl;
        ```
    7. 等待函数若要处理绝对超时，则需接受时间点实例作为参数，该实例的相关时钟会用作参考，计算是否超时。一旦时钟被改动，将产生重大影响，因为等待会跟随时钟变化，若时钟的成员函数 now() 的返回值早于指定的时限，则等待函数不返回。
    8. 时间点用于带有后缀“_until”的等待函数的变体。为了预先安排操作，我们需计算某一具体未来时刻（它对用户可见）。虽然可以用静态函数 std::chrono::system_clock::to_time_point() 转换 time_t 值，从而求出基于系统时钟的时间点，但其实最“地道”的方法是在程序代码中的某个固定位置，将 some_clock::now() 和前向偏移相加得出时间点。
    9. 就条件变量进行限时等待：
        ```
        #include <condition_variable>
        #include <mutex>
        #include <chrono>
        std::condition_variable cv;
        bool done;
        std::mutex m;
        bool wait_stop()
        {
            auto const timeout = std::chrono::steady_clock::now() + std::chrono::milliseconds(500);
            std::unique_lock<std::mutex> lk(m);
            while(!done)
            {
                if(cv.wait_until(lk, timeout) == std::cv_status::timeout)
                    break;
            }
            return done;
        }
        ```
        - 假如循环中使用的是 wait_for()，那么，若在等待时间快消耗完时发生伪唤醒，而我们如果要再次等待，就得重新开始一次完整的迟延等待。这也许会不断重复，令等待变得漫无边际。

15. 接收超时时限的函数
    1. 超时时限的最简单用途是，推迟特定线程的处理过程，若它无所事事，就不会占用其他线程的处理时间。
    2. std::this_thread::sleep_for() 和 std::this_thread::sleep_until，它们的功能就像简单的闹钟：线程或采用 sleep_for() 按指定的时长休眠，或采用 sleep_until() 休眠直到指定时刻为止。
    3. 普通的 std::mutex 和 std::recursive_mutex 不能限制加锁，但 std::timed_mutex 和 std::recursive_timed_mutex 可以。这两种锁都含有成员函数 try_lock_for() 和 try_lock_until()，前者尝试在给定的时长内获取锁，后者尝试在给定的时间点之前获取锁。

16. C++ 标准库中接受超时时限的函数：

类/名字空间 | 函数 | 返回值
---|---|---
std::this_thread 名字空间 | sleep_for(duration)<br>sleep_until(time_point) | 无
std::condition_variable 或 std::condition_variable_any<br>wait_for(lock,duration)<br>wait_until(lock,time_point)| std::cv_status::timeout 或 std::cv_status::no_timeout<br>wait_for(lock,duration,predicate)<br>wait_until(lock,time_point,predicate) | Bool——被唤醒时断言的返回值
std::timed_mutex, std::recursive_timed_mutex 或 std::shared_timed_mutex | try_lock_for(duration)<br>try_lock_until(time_point) | Bool——若获取了锁，则返回 true，否则返回 false
std::shared_timed_mutex | try_lock_shared_for(duration)<br>try_lock_shared_until(time_point) | Bool——若获取了锁，则返回 true，否则返回 false
std::unique_lock<TimedLockable><br>unique_lock(lockable,duration)<br>unique_lock(lockable,time_point) | try_lock_for(duration)<br>try_lock_until(time_point) | 无——如果在新构建的对象上获取了锁，那么 owns_lock() 返回 true，否则返回 false<br>Bool——若获取了锁，则返回 true，否则返回 false
std::shared_lock<TimedLockable><br>shared_lock(lockable,duration)<br>shared_lock(lockable,time_point) | try_lock_for(duration)<br>try_lock_until(time_point) | 无——如果在新构建的对象上获取了锁，那么 owns_lock() 返回 true，否则返回 false<br>Bool——若获取了锁，则返回 true，否则返回 false
std::future<ValueType>或std::shared_future<ValueType> | wait_for(duration)<br>wait_until(time_point) | 如果等待超时则返回 std::future_status::timeout<br>如果 future 已就绪则返回 std::future_status::ready<br>如果 future 上的函数按推迟方式执行，且尚未开始执行，则返回 std::future_status::deferred

17. 运用同步操作简化代码

一种简化代码的途径是：在并发实战中使用非常贴近函数式编程的风格。线程间不会直接共享数据，而是由各任务分别预先备妥自己所需的数据，并借助 future 将结果发送到其他有需要的线程。

18. 利用 future 进行函数式编程
    1. 术语“函数式编程”是指一种编程风格，函数调用的结果完全取决于参数，而不依赖任何外部状态。这源自数学概念中的函数，它的意义是，若我们以相同的参数调用一个函数两次，结果会完全一致。
    2. future 对象可在线程间传递，所以一个计算任务可以依赖另一个任务的结果，却不必显式地访问共享数据。
    3. 函数式编程风格的快速排序：
        ```
        // 快速排序的串行实现
        template<typename T>
        std::list<T> sequential_quick_sort(std::list<T> input)
        {
            if(input.empty())
            {
                return input;
            }
            std::list<T> result;
            result.splice(result.begin(), input, input.begin());
            T const& pivot = *result.begin();
            auto divide_point = std::partition(input.begin(), input.end(), 
                                    [&](T const& t) { return t<pivot; } );
            std::list<T> lower_part;
            lower_part.splice(lower_part.end(), input, input.begin(), divide_point);
            auto new_lower(sequential_quick_sort(std::move(lower_part)));
            auto new_higher(sequential_quick_sort(std::move(input)));
            result.splice(result.end(), new_higher);
            result.splice(result.begin(), new_lower);
            return result;
        }
        ```
        - 我们直接取出原链表的头号元素作为基准元素，用 splice() 将它从原链表最前端切除。
        - 设定切割准则的最简单方式是 lambda 函数；我们按引用方式捕获基准元素，以避免按值复制。
        - std::partition() 在链表内部整理元素的位置，并返回一个迭代器，指向大于等于基准元素的第一个元素。
        - std::partition() 的行为不是排序，而是整理。链表中的一些元素令 lambda 函数返回 true，std::partition() 以此为准，将它们置于链表的前半部分，而其余元素则使 lambda 函数返回 false，遂置于后半部分，仅此而已，前后两部分的元素并未排序。
        
    4. 函数式编程风格的并行快速排序
        ```
        // 运用 future 实现并行快速排序
        template<typename T>
        std::list<T> parallel_quick_sort(std::list<T> input)
        {
            if(input.empty())
            {
                return input;
            }
            std::list<T> result;
            result.splice(result.begin(), input, input.begin());
            T const& pivot = *result.begin();
            auto divide_point = std::partition(input.begin(), input.end(), 
                                    [&](T const& t) { return t<pivot; } );
            std::list<T> lower_part;
            lower_part.splice(lower_part.end(), input, input.begin(), divide_point);
            std::future<std::list<T>> new_lower(std::async(&parallel_quick_sort<T>, std::move(lower_part)));
            auto new_higher(parallel_quick_sort(std::move(input)));
            result.splice(result.end(), new_higher);
            result.splice(result.begin(), new_lower.get());
            return result;
        }
        ```
        - 假设向其他线程传递任务无助于提升性能，那么负责调用 get() 的线程就会亲自执行新任务，而不再发起新线程，从而减小开销。


19. 使用消息传递进行同步
    1. 角色模型是一种程序设计风格：系统中含有一些分散的角色，它们各自在独立线程上运行，它们彼此收发消息以执行手上的任务，还直接通过消息传递状态，但除此以外，它们之间没有共享数据。
    2. “分离关注点”的软件设计原则：通过利用多个线程，整体任务按要求被明确划分。

**以下内容：略**

20. 符合并发技术规约的后续风格并发
21. 后续函数的连锁调用
22. 等待多个 future
23. 运用 std::experimental::when_any() 函数等待多个 future，直到其中之一准备就绪
24. 线程闩和线程卡——并发技术规约提出的新特性
25. 基本的线程闩类 std::experimental::latch
26. 基本的线程卡类 std::experimental::barrier
27. std::experimental::flex_barrier——std::experimental::barrier 的灵活版本

---

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

---

###### 六、设计基于锁的并发数据结构
略。

---

###### 七、设计无锁数据结构
略。

---

###### 八、设计并发代码
略。

---

###### 九、高级线程管理

1. 线程池最简单的实现形式是，采用数目固定的工作线程（往往与 std::thread::hardware_concurrency() 的返回值相等）。

2. 由于 std::packaged_task<> 的实例仅能移动而不可复制，但 std::funtion<> 要求本身所含的函数对象能进行拷贝构造，因此任务队列的元素不能用 std::function<> 充当。我们必须定制自己的类作为代替，以包装函数，并处理只移型别。这个类其实就是一个包装可调用对象的简单类，对外可消除该对象的型别。这个类还具备函数调用操作符。
    ```
    class function_wrapper
    {
        struct impl_base
        {
            virtual void call() = 0;
            virtual ~impl_base() {}
        };
        std::unique_ptr<impl_base> impl;
        template<typename F>
        struct impl_type : impl_base
        {
            F f;
            impl_type(F&& f_)
                : f(std::move(f_))
            {}
            void call()
            {
                f();
            }
        };
    public:
        template<typename F>
        function_wrapper(F&& f)
            : impl(new impl_type<F>(std::move(f)))
        {}
        void operator() ()
        {
            impl->call();
        }
        function_wrapper() = default;
        function_wrapper(function_wrapper&& other)
            : impl(std::move(other.impl))
        {}
        function_wrapper& operator=(function_wrapper&& other)
        {
            impl = std::move(other.impl);
            return *this;
        }
        function_wrapper(const function_wrapper&) = delete;
        function_wrapper(function_wrapper&) = delete;
        function_wrapper& operator=(const function_wrapper&) = delete;
    };
    ```
3. 基于锁的队列，它支持任务窃取
    ```
    class work_stealing_queue
    {
    private:
        typedef function_wrapper data_type;
        std::queue<data_type> the_queue;
        mutable std::mutex the_mutex;
    public:
        work_stealing_queue()
        {}
        work_stealing_queue(const work_stealing_queue& other) = delete;
        work_stealing_queue& operator=(const work_stealing_queue& other) = delete;
        void push(data_type data)
        {
            std::lock_guard<std::mutex> lock(the_mutex);
            the_queue.push_front(std::move(data));
        }
        bool empty() const
        {
            std::lock_guard<std::mutex> lock(the_mutex);
            return the_queue.empty();
        }
        bool try_pop(data_type& res)
        {
            std::lock_guard<std::mutex> lock(the_mutex);
            if(the_queue.empty())
            {
                return false;
            }
            res = std::move(the_queue.front());
            the_queue.pop_front();
            return true;
        }
        bool try_steal(data_type& res)
        {
            std::lock_guard<std::mutex> lock(the_mutex);
            if(the_queue.empty())
            {
                return false;
            }
            res = std::move(the_queue.back());
            the_queue.pop_back();
            return true;
        }
    };
    ```
    - try_steal() 操作队列的后端。

4. 利用任务窃取的线程池
    ```
    class thread_pool
    {
        typedef function_wrapper task_type;
        std::atomic_bool done;
        threadsafe_queue<task_type> pool_work_queue;
        std::vector<std::unique_ptr<work_stealing_queue> > queues;
        std::vector<std::thread> threads;
        join_threads joiner;
        static thread_local work_stealing_queue* local_work_queue;
        static thread_local unsigned my_index;
        void worker_thread(unsigned my_index_)
        {
            my_index = my_index_;
            local_work_queue = queues[my_index].get();
            while(!done)
            {
                run_pending_task();
            }
        }
        bool pop_task_from_local_queue(task_type& task)
        {
            return local_work_queue && local_work_queue->try_pop(task);
        }
        bool pop_task_from_pool_queue(task_type& task)
        {
            return pool_work_queue.try_pop(task);
        }
        bool pop_task_from_other_thread_queue(task_type& task)
        {
            for(unsigned i=0; i<queues.size(); ++i)
            {
                unsigned const index = (my_index+i+1)%queues.size();
                if(queues[index]->try_steal(task))
                {
                    return true;
                }
            }
            return false;
        }
    public:
        thread_pool()
            : done(false)
            , joiner(threads)
        {
            unsigned const thread_count = std::thread::hardware_concurrency();
            try
            {
                for(unsigned i=0; i<thread_count; ++i)
                {
                    queues.push_back(std::unique_ptr<work_stealing_queue>(new work_stealing_queue));
                }
                for(unsigned i=0; i<thread_count; ++i)
                {
                    threads.push_back(std::thread(&thread_pool::worker_thread, this, i));
                }
            }
            catch(...)
            {
                done = true;
                throw;
            }
        }
        ~thread_pool()
        {
            done = true;
        }
        template<typename FunctionType>
        std::future<typename std::result_of<FunctionType()>::type> submit(FunctionType f)
        {
            typedef typename std::result_of<FunctionType()>::type> result_type;
            std::packaged_task<result_type()> task(f);
            std::future<result_type> res(task.get_future());
            if(local_work_queue)
            {
                local_work_queue->push(std::move(task));
            }
            else
            {
                pool_work_queue.push(std::move(task));
            }
            return res;
        }
        void run_pending_task()
        {
            task_type task;
            if(pop_task_from_local_queue(task)
                    || pop_task_from_pool_queue(task)
                    || pop_task_from_other_thread_queue(task))
            {
                task();
            }
            else
            {
                std::this_thread::yield();
            }
        }
    };
    ```
    - 请注意，声明数据成员的先后次序十分重要：done 标志和 pool_work_queue 队列必须位列最前，接着是装载线程的 vector 容器实例 threads，joiner 则必须在最后声明。这是为了确保线程池的数据成员能正确地依次销毁：本例中，线程必须全部终结，任务队列才可以销毁。
    - submit() 函数返回一个 std::future<> 实例，任务的返回值由它持有，调用者凭借该实例即可等待任务完成。
    - 请注意，由于 std::packaged_task<> 不可复制，因此一定要通过 std::move() 把任务压入队列。
    - run_pending_task()函数：如果队列中存在任务，该函数即试图领取，否则就令所属线程让步，以便操作系统重新安排调度。
    - 有一种方法可解决缓存乒乓：为每个线程配备独立的任务队列。各线程只在自己的队列上发布新任务，仅当线程自身的队列没有任务时，才会从全局队列领取任务。
    - 采用 thread_local 变量，从而令每个线程都具有自己的任务队列，线程池本身则再维护一全局队列。
    - 线程局部的任务队列由 std::unique_ptr<> 指针持有。
    - 按此设计，池外的其他无关线程就不必无意义地附带队列。关键字 thread_local 对程序内的全部线程一视同仁，都起作用。所以，无论池内、池外，每个线程都具有自己专属的 std::unique_ptr<> 指针，只不过池外线程不含队列，其指针的目标为空，而池内线程则会分别生成专属的队列，为指针所指向。
    - 局部对象可由普通的 std::queue<> 队列充当，因为它始终只被唯一一个线程访问。
    - 队列在清单中的索引值传给对应的线程函数，池内各线程凭此获取一指针，指向隶属自己的队列。（清单指的是一个 std::vector 容器，内含多个 std::unique_ptr<> 指针，目标是各线程的 work_stealing_queue 队列）
    - pop_task_from_other_thread_queue() 逐个访问池内线程所含全部队列，试图从中窃取任务。这些线程在开始窃取任务时，均根据自己在清单中的索引值，向后偏移一项，以下一线程作为窃取起始点，借此避免了清单中的头一个队列沦为“众矢之的”，而导致每个线程都集中对其下手。

5. 中断线程

> C++20 标准已正式引入了 std::jthread，其下管控的线程可通过这个类接受中断，还能自动汇合。

6. 发起一个线程，以及把它中断
    1. 最基本地，我们希望能在代码中指定某目标位置，说“这里可以发生中断”，即安插一个中断点。为了避免在调用时额外传递数据，从而使之真正实用可行，其具体形式是一个简单函数 interruption_point()，它无须接收任何参数。这就要求，将 thread_local 变量作为中断专门定制的数据结构，在启动线程时将此变量设置妥当，如果当前正在执行的线程调用了 interruption_point() 函数，便会查验该数据结构。
    2. 此处无法用普通的 std::thread 类管控线程，关键原因正是这个 thread_local 变量。它所修饰的变量需按下面的方式分配内存：既能让 interruptible_thread 的实例访问，又能在新启动的线程上直接访问。我们可以先将给定的函数包装好，然后传入 std::thread 的构造函数来发起新线程。
        ```
        class interrupt_flag
        {
        public:
            void set();
            bool is_set() const;
        };
        thread_local interrupt_flag this_thread_interrupt_flag;
        class interruptible_thread
        {
            std::thread internal_thread;
            interrupt_flag* flag;
        public:
            template<typename FunctionType>
            interruptible_thread(FunctionType f)
            {
                std::promise<interrupt_flag*> p;
                internal_thread = std::thread([f, &p]
                {
                    p.set_value(&this_thread_interrupt_flag);
                    f();
                });
                flag = p.get_future().get();
            }
            void interrupt()
            {
                if(flag)
                {
                    flag->set();
                }
            }
        }
        ```
        - interruptible_thread 类的实例管控一个线程，该线程可以受到中断，该中断由别的线程调用 interrupt() 而主动触发。
        - 调用者给出一个函数 f()，它包装在一个 lambda 函数中，后者持有函数 f() 的副本，还有一个指涉局部 promise 的引用。在新启动的线程上，该 lambda 函数将 promise 关联的值设置成 this_thread_interrupt_flag 标志的地址（这一标志的声明以 thread_local 修饰），再根据 f() 的副本来调用给定的函数。接着，发起调用的线程便开始等待，等到与 promise 关联的 future 准备就绪，并将运行结果存储到成员变量 flag 中时，结束等待。
        - 请注意，尽管 lambda 函数在新线程上运行，却有一个指向局部变量 p 的悬空引用，但这并无大碍，因为 interruptible_thread 类的构造函数会一直等待，直到引用 p 不再为新线程所指涉以后，它才返回。
        - 另外请注意，这个实现并不负责汇合线程或分离线程，当线程退出或分离时，我们必须确保清除了 flag 变量，以免指针悬空。

7. 检测线程是否被中断
    1. 最简单的一种方法是直接在 interruption_point() 函数内检测：在可以安全地发生中断的地方，我们调用这个函数，若中断标志已经设置成立，即抛出 thread_interrupted 异常。
        ```
        void interruption_point()
        {
            if(this_thread_interrupt_flag.is_set())
            {
                throw thread_interrupted();
            }
        }
        
        void foo()
        {
            while(!done)
            {
                interruption_point();
                process_next_item();
            }
        }
        ```
        - 若某线程执行含有 interruption_point() 的代码，该线程意在接受中断。
    2. 中断某些线程的最佳时机是在它因等待而发生阻塞之时。

8. 中断条件变量上的等待
    1. 最简单可行的方式是，一旦设置了中断标志成立，即通知条件变量，并且紧随等待调用安插中断点。
    2. 针对条件变量 std::condition_variable 的 interruptible_wait() 函数，它支持超时就停止：
        ```
        class interrupt_flag
        {
            std::atomic<bool> flag;
            std::condition_variable* thread_cond;
            std::mutex set_clear_mutex;
        public:
            interrupt_flag()
                : thread_cond(0)
            {}
            void set()
            {
                flag.store(true, std::memory_order_relaxed);
                std::lock_guard<std::mutex> lk(set_clear_mutex);
                if(thread_cond)
                {
                    thread_cond->notify_all();
                }
            }
            bool is_set()
            {
                return flag.load(std::memory_order_relaxed);
            }
            void set_condition_variable(std::condition_variable& cv)
            {
                std::lock_guard<std::mutex> lk(set_clear_mutex);
                thread_cond = &cv;
            }
            void clear_condition_variable()
            {
                std::lock_guard<std::mutex> lk(set_clear_mutex);
                thread_cond = 0;
            }
            struct clear_cv_on_destruct
            {
                ~clear_cv_on_destruct()
                {
                    this_thread_interrupt_flag.clear_condition_variable();
                }
            };
        };
        
        void interruptible_wait(std::condition_variable& cv, std::unique_lock<std::mutex>& lk)
        {
            interruption_point();
            this_thread_interrupt_flag.set_condition_variable(cv);
            interrupt_flag::clear_cv_on_destruct guard;
            interruption_point();
            cv.wait_for(lk, std::chrono::milliseconds(1));
            interruption_point();
        }
        ```
    3. 如果我们需要等待某个断言成立，那么就可以把 1 毫秒的时限完全融合到断言循环之中。
        ```
        template<typename Predicate>
        void interruptible_wait(std::condition_variable& cv, std::unique_lock<std::mutex>& lk, Predicate pred)
        {
            interruption_point();
            this_thread_interrupt_flag.set_condition_variable(cv);
            interrupt_flag::clear_cv_on_destruct guard;
            while(!this_thread_interrupt_flag.is_set() && !pred())
            {
                cv.wait_for(lk, std::chrono::milliseconds(1));
            }
            interruption_point();
        }
        ```
    
9. 中断条件变量 std::condition_variable_any 上的等待
    1. std::condition_variable_any 和 std::condition_variable 的区别在于，前者可以配合任意型别的锁，而后者仅限于 std::unique_lock<std::mutex>。
    2. 条件变量 std::condition_variable_any 的 interruptible_wait() 函数：
        ```
        class interrupt_flag
        {
            std::atomic<bool> flag;
            std::condition_variable* thread_cond;
            std::condition_variable_any* thread_cond_any;
            std::mutex set_clear_mutex;
        public:
            interrupt_flag()
                : thread_cond(0)
                , thread_cond_any(0)
            {}
            void set()
            {
                flag.store(true, std::memory_order_relaxed);
                std::lock_guard<std::mutex> lk(set_clear_mutex);
                if(thread_cond)
                {
                    thread_cond->notify_all();
                }
                else if(thread_cond_any)
                {
                    thread_cond_any->notify_all();
                }
            }
            template<typename Lockable>
            void wait(std::condition_variable_any& cv, Lockable& lk)
            {
                struct custom_lock
                {
                    interrupt_flag* self;
                    Lockable& lk;
                    custom_lock(interrupt_flag* self_, std::condition_variable_any& cond, Lockable& lk_)
                        : self(self_)
                        , lk(lk_)
                    {
                        self->set_clear_mutex.lock();
                        self->thread_cond_any = &cond;
                    }
                    void unlock()
                    {
                        lk.unlock();
                        self->set_clear_mutex.unlock();
                    }
                    void lock()
                    {
                        std::lock(self->set_clear_mutex, lk);
                    }
                    ~custom_lock()
                    {
                        self->thread_cond_any = 0;
                        self->set_clear_mutex.unlock();
                    }
                };
                custom_lock cl(this, cv, lk);
                interruption_point();
                cv.wait(cl);
                interruption_point();
            }
            // 余下代码和上面 8.2 相同
        };
        
        template<typename Lockable>
        void interruptible_wait(std::condition_variable_any& cv, Lockable& lk)
        {
            this_thread_interrupt_flag.wait(cv, lk);
        }
        ```
        - 在当前线程执行 wait() 期间，若其他线程试图中断，后者就会从互斥 set_clear_mutex 上获取锁，并且查验 thread_cond_any 指针，但在 wait() 调用之前，这两项操作不可能执行。

10. 中断其他阻塞型等待
    1. 一般地，我们需要借助处理 std::condition_variable 所用到的限时功能，因为上述等待行为均不涉及等待某个条件成立，若不从内部修改互斥或 future 就无法将它们中断。但是，我们很清楚另外几种等待的目标条件具体是什么，故可以在 interruptible_wait() 函数中用循环来等待。
        ```
        // 现有一个 interruptible_wait() 函数的重载，它是 std::future<> 的成员函数
        template<typename T>
        void interruptible_wait(std::future<T>& uf)
        {
            while(!this_thread_interrupt_flag.is_set())
            {
                if(uf.wait_for(lk, std::chrono::milliseconds(1)) == std::future_status::ready)
                    break;
            }
            interruption_point();
        }
        ```

11. 处理中断
    1. 从接受中断的线程的视角观察，一次中断本质上就是一个 thread_interrupt 异常，因而中断可依照处理其他异常的方式处理。
        ```
        try{
            do_something();
        }
        catch(thread_interrupted&){
            handle_interruption();
        }
        ```
        - 以上设计的意思是，令目标线程关联某个 interruptible_thread 对象，别的线程在该对象上调用 interrupt() 而引发中断，程序得以捕获这个中断，按某种方式处理，然后继续执行。
    2. std::thread 在构造时设定了线程函数，一旦让异常传播到该函数外，std::terminate() 即会被自动调用，从而终止整个程序。interruptible_thread 其实是个内含 std::thread 的包装类，初始化时同样传入线程函数，且需加入 catch(thread_interrupted&) 的处理代码。为防止忘记这么做，我们可以在这个类的初始化代码中放置 catch 块。如此一来，不处理中断异常、任它随意传播也成了安全行为，因为最后它只会终止一个线程。
        ```
        internal_thread = std::thread([f, &p]
                {
                    p.set_value(&this_thread_interrupt_flag);
                    try{
                        f();
                    }
                    catch(thread_interrupted const&){
                    }
                });
        ```

12. 在应用程序退出时中断后台任务
    1. 当应用程序关闭时，我们需要依次结束后台线程，其中一种做法就是中断它们。
    2. 在后台监控文件系统
        ```
        std::mutex config_mutex;
        std::vector<interruptible_thread> background_threads;
        void background_thread(int disk_id)
        {
            while(true)
            {
                interruption_point();
                fs_change fsc = get_fs_change(disk_id);
                if(fsc.has_changes())
                {
                    update_index(fsc);
                }
            }
        }
        void start_background_processing()
        {
            background_threads.push_back(interruptible_thread(background_thread, disk_1));
            background_threads.push_back(interruptible_thread(background_thread, disk_2));
        }
        int main()
        {
            start_background_processing();
            process_gui_until_exit();
            std::unique_lock<std::mutex> lk(config_mutex);
            for(unsigned i=0; i<background_threads.size(); ++i)
            {
                background_threads[i].interrupt();
            }
            for(unsigned i=0; i<background_threads.size(); ++i)
            {
                background_threads[i].join();
            }
        }
        ```
        - 每轮循环中，它们都调用 interruption_point() 来判别是否发生中断。
        - 先中断全部线程再等它们汇合，而不是中断一个线程就马上等待它汇合，然后再处理下一个的原因是：提高并发性能。线程不太会因为被中断就马上结束，因为它们必须运行到下一个中断点，调用析构函数，或执行处理异常的代码，然后才会退出。因此，中断一个线程并立即等待它汇合，就会令发起中断的线程白白等待，即使它本来可以去做有用的工作——继续中断其他线程。

---

###### 十、并行算法函数
略。

---

###### 十一、多线程应用的测试和除错
略。

---

###### 附录A、C++11 精要：部分语言特性

1. 右值引用
    ```
    int var = 42;
    int& ref = var; // 创建名为 ref 的引用，指向的模目标是变量 var
    
    int &i = 42;    // 无法编译
    int const& i = 42;  // 我们一般都能将右值绑定到 const 左值引用上
    
    int&& i = 42;
    int j = 42;
    int&& k = j;    // 编译失败
    ```
    - 术语右值来自 C 语言，指只能在赋值表达式等号右边出现的元素，如字面值和临时变量。
    - 左值引用只可以绑定左值，而无法与右值绑定。
    - C++11 标准采纳了右值引用这一新特性，它只与右值绑定，而不绑定左值。另外，其声明不再仅仅带有一个“&”，而改为两个“&”。

2. 移动语义
    1. 右值往往是临时变量，故可以自由改变。
        ```
        void process_copy(std::vector<int> const& vec_)
        {
            std::vector<int> vec(vec_);
            vec.push_back(42);
        }
        
        void process_copy(std::vector<int> && vec)
        {
            vec.push_back(42);
        }
        ```
        - void process_copy(std::vector<int> const& vec_) 这个函数接受左值和右值（此处的右值特指前文的绑定常量的引用，而非 C++11 新特性的右值引用）皆可，但都会强制进行复制。
        - 若我们预知原始数据能随意改动，即可重载该函数，编写一个接受右值引用的参数的版本 void process_copy(std::vector<int> && vec)，以此避免复制（这里为了讲解移动语义，刻意采用右值引用传参，但实际上，按传统的非 const 左值引用传参也能避免复制（直接引用原始数据，并不采用移动语义））。
    2. 具备移动构造函数的类
        ```
        class X
        {
        private:
            int* data;
        public:
            X()
             : data(new int[1000000])
            {}
            ~X()
            {
                delete [] data;
            }
            X(const X& other)
             : data(new int[1000000])
            {
                std::copy(other.data, other.data+1000000, data);
            }
            X(X&& other)
             : data(other.data)
            {
                other.data = nullptr;
            }
        }
        ```
        - 移动构造函数，它复制 data 指针，将源实例的 data 指针改为空指针，从而节约了一大块内存，还省去了复制数据本体的时间。
        - 某些类很有必要实现移动构造函数，强令它们实现拷贝构造函数反而不合理。以 std::unique_ptr<> 指针为例，其非空实例必须指向某对象，根据设计意图，它也肯定是指向该对象的唯一指针，故只许移动而不许复制，则拷贝构造函数没有存在的意义。
    3. 假如某个具名对象不再有任何用处，我们想将其移出，因而需要先把它转换成右值，这一操作可通过 static_cast<X&&> 转换或调用 std::move() 来完成。
        ```
        X x1;
        X x2 = std::move(x1);
        X x3 = static_cast<X&&>(x2);
        ```
    4. 尽管右值引用的形参与传入的右值实参绑定，但参数进入函数内部后即被当作左值处理。所以，当我们处理函数的参数的时候，可将其值移入函数局部变量或类的成员变量，从而避免复制整份数据。
        ```
        void do_stuff(X&& x_)
        {
            X a(x_);            // 复制构造
            X b(std::move(x_)); // 移动构造
        }
        do_stuff(X());  // 正确，X() 生成一个匿名临时对象，作为右值与右值引用绑定
        X x;
        do_stuff(x);    // 错误，具名对象 x 是左值，不能与右值引用绑定
        ```
    5. std::thread、std::unique_lock<>、std::future<>、std::promise<> 和 std::packaged_task<> 等类无法复制，但它们都含有移动构造函数，可以在其实例之间转移关联的资源，也能按转移的方式充当函数返回值。
    6. 按照良好的编程实践，类需确保其不变量的成立范围覆盖其“移出状态”。如果 std::thread 的实例作为移动操作的数据源，一旦发生了移动，它就等效于按默认方式构造的线程实例。（按默认方式构造的 std::thread 对象不含实际数据，也不管控或关联任何线程）
    7. 对于 std::string 类，C++ 标准仅要求移动操作在常数复杂度的时间内完成，却没有规定源数据上的实际效用如何。因此，要注意，移动语义可能通过不同方式实现，不一定真正窃取数据，也不一定搬空源对象。
    
3. 右值引用和函数模板
    1. 假如函数的参数是右值引用，目标是模板参数，那么根据模板参数的自动类型推导机制，若我们给出左值作为函数参数，模板参数则会被推导为左值引用；若函数参数是右值，模板参数则会被推导为无修饰型别的普通引用。
        ```
        template<typename T>
        void foo(T&& t)
        {}
        
        foo(42);            // 调用 foo<int>(42)
        foo(3.14159);       // 调用 foo<double>(3.14159)
        foo(std::string()); // 调用 foo<std::string>(std::string())
        
        int i = 42;
        foo(i);             // 调用 foo<int&>(i)
        ```
        - 根据函数声明，其参数型别是 T&&，在本例的情形中会解释成“引用的引用”，所以发生引用折叠（左值的多重引用会引发折叠），编译器将它视为原有型别的普通引用。这里，foo<int&>() 的函数签名是 “void foo<int&>(int& t);”。
        - 利用该特性，同一个函数模板既能接收左值参数，又能接收右值参数。std::thread 的构造函数正是如此。若我们以左值形式提供可调用对象作为参数，它即被复制到相应线程的内部存储空间；若我们以右值形式提供参数，则它会按移动方式传递。

4. 删除函数
    1. 要禁止某个类的复制行为，以前的标准处理手法是将拷贝构造函数和复制赋值操作符声明为私有，且不给出实现。假如有任何外部代码意图复制该类的实例，就会导致编译错误（因为调用私有函数）；若其成员函数或友元函数试图复制它的实例，则会产生链接错误（因为没有提供实现）。
    2. 声明函数的语句只要追加“=delete”修饰，函数即被声明为“删除”。
    3. 若我们在实现某个类的时候，既删除拷贝构造函数和复制赋值操作符，又显式写出移动构造函数和移动赋值操作符，它便成了“只移型别”，该特性与 std::thread 和 std::unique_lock<> 的相似。
    4. 只移对象可以作为参数传入函数，也能充当函数返回值。然而，若要从某个左值移出数据，我们就必须使用 std::move() 或 static_cast<T&&> 显式表达该意图。
    5. 说明符“=delete”可修饰任何函数，而不局限于拷贝构造函数和赋值操作符，其可清楚注明目标函数无效。它还具备别的作用：如果某函数已声明为删除，却按普通方式参与重载解释并且被选定，就会导致编译错误。利用这一特性，我们即能移除特定的重载版本。例如，假如某函数接收 short 型参数，那它也允许传入 int 值，进而将 int 值强制向下转换成 short 值。若要严格杜绝这种情况，我们可以编写一个传入 int 类型参数的重载，并将它声明为删除。

5. 默认函数
    1. 一旦将某函数标注为删除函数，我们就进行了显式声明：它不存在实现。但默认函数则完全相反：它们让我们得以明确指示编译器，按“默认”的实现方式生成目标函数。如果一个函数可以由编译器自动产生，那它才有资格被设为默认：默认构造函数、析构函数、拷贝构造函数、移动构造函数、复制赋值操作符和移动赋值操作符等。
    2. 一般来说，仅当用户自定义构造函数不存在时，编译器才会生成默认构造函数，针对这种情形，添加“=default”修饰即可保证其生成出来。
    3. 令析构函数成为虚拟函数，并托付给编译器生成。
    4. “默认构造函数”特指不接收任何参数的构造函数（或参数全都具备默认值）。
    5. 在同一个类中，若将某些成员函数交由编译器实现，它们便会具备一定的特殊性质，但是让我们自定义实现，这些性质就会丧失。两种实现方式的最大差异是，编译器有可能生成平实函数。
    6. 平实函数即 trivial function，其现实意义是默认构造函数和析构函数不执行任何操作；复制、赋值和移动操作仅仅涉及最简单、直接的按位进行内存复制/内存转移，而没有任何其他行为；若对象所含的默认函数全是平实函数，就可依照 Plain Old Data（POD）方式进行处理。
    7. constexpr 函数所用的字面值型别必须具备平实构造函数、平实拷贝构造函数和平实析构函数。
    8. 若要允许一个类能够被联合体（union）所包含，而后者已具备自定义的构造函数和析构函数，则这个类必须满足：其默认构造函数、拷贝构造函数、复制操作符和析构函数均为平实函数。
    9. 假定某个类充当了类模板 std::atomic<> 的模板参数，那它应当带有平实拷贝赋值操作符，才可能提供该类型值的原子操作。
    10. 一旦函数由用户自己动手显示编写而写，就肯定不是平实函数。
    11. 在同一个类中，某些特定的成员函数既能让编译器生成，又准许用户自行编写，我们继续分析两种实现方式的第二项差异：如果用户没有为某个类提供构造函数，那么它便得以充当聚合体，其初始化过程可依照聚合体初值表达式完成。
    12. 聚合体即 aggregate，是 C++ 11 引入的概念，它通常可以是数组、联合体、结构体或类（不得含有虚函数或自定义的构造函数，亦不得继承自父类的构造函数，还要服从其他限制），其涵盖范围随 C++ 标准的演化而正在扩大。
    13. 如果类 X 的实例在初始化时显示调用了默认构造函数，成员 a 即初始化为 0。
        ```
        struct X
        {
            int a;
        }
        X x1;           // x1.a 的值尚未确定
        X x2 = X();     // x2.a == 0 必然成立
        ```
        - 这个特殊性质还能扩展至基类及内部成员。假定某个类的默认构造函数由编译器产生，而它的每个数据成员与全部基类也同样如此，并且后者两者所含的成员都属于内建型别。那么，这个最外层的类是否显式调用该默认构造函数，就决定其成员是否初始化为尚未确定的值，抑或发生零值初始化。
        - 一旦我们手动实现默认构造函数，它就会丧失这个性质：要是指定了初值或显式地按默认方式构造，数据成员便肯定会进行初始化，否则初始化始终不会发生。
        - 一般情况下，若我们自行编写出任何别的构造函数，编译器就不会再生成默认构造函数。如果我们依然要保留它，就得自己手动编写，但其初始化行为会失去上述特性。然而，将目标构造函数显式声明成“默认”，我们便可强制编译器生成默认构造函数，并且维持该性质。
            ```
            X::X() = default;  // 默认初始化规则对成员 a 起作用
            ```
        - 原子类型正是利用了这个性质将自身的默认构造函数显式声明为“默认”。除去下列几种情况，原子类型的初值只能是未定义：它们具有静态生存期（因此静态初始化成零值）；显式调用默认构造函数，以进行零值初始化；我们明确设定了初值。请注意，各种原子类型均具备一个构造函数，它们单独接受一个参数作为初值，而它们都声明成 constexpr 函数，以准许静态初始化发生。

6. 常量表达式函数
    1. 常量表达式可用于创建常量，进而构建其他常量表达式。一些功能只能靠常量表达式实现。
        ```
        1、设定数组界限：
        int bounds = 99;
        int array[bounds];      // 错误，界限 bounds 不是常量表达式
        const int bounds2 = 99;
        int array2[bounds2];    // 正确，界限 bounds2 是常量表达式
        
        2、设定非类型模板参数的值：
        template<unsigned size>
        struct test
        {};
        test<bounds> ia;        // 错误，界限 bounds 不是常量表达式
        test<bounds2> ia2;      // 正确，界限 bounds2 是常量表达式
        
        3、在定义某个类时，充当静态常量整型数据成员的初始化表达式：
        class X
        {
            static const int the_answer = forty_two;  
        };
        
        4、对于能够进行静态初始化的内建型别和聚合体，我们可以将常量表达式作为其初始化表达式：
        struct my_aggregate
        {
            int a;
            int b;
        }
        static my_aggregate ma1 = {forty_two, 123};     // 静态初始化
        int dummy = 257;
        static my_aggregate ma2 = {dummy, dummy};       // 动态初始化
        
        5、只要采用本例示范的静态初始化方式，即可避免初始化的先后次序问题，从而防止条件竞争。
        ```
        - 静态数据成员 the_answer 由表达式 forty_two 初始化，所在的语句既是声明又是定义。作为静态数据成员，其只许枚举值和整型常量在类定义内部直接定义，而任意其他类型仅能声明，且必须在类定义外部给出定义。
    2. constexpr 关键字的主要功能是充当函数限定符。假如某函数的参数和返回值都满足一定要求，且函数体足够简单，那它就可以声明为 constexpr 函数，进而在常量表达式中使用。
        ```
        constexpr int square(int x)
        {
            return x*x;
        }
        int array[square(5)];
        
        int dummy = 4;
        int array[square(dummy)];   // 错误，dummy 不是常量表达式
        ```

7. constexpr 关键字和用户自定义型别
    1. 若某个类要被划分为字面值型别，则下面条件必须全部成立：
        - 它必须具有平实拷贝构造函数。
        - 它必须具有平实析构函数。
        - 它的非静态数据成员和基类都属于平实型别。
        - 它必须具备平实默认构造函数或常量表达式构造函数（若具备后者，则不得进行拷贝/移动构造）。
    2. 字面值类型是 C++11 引入的新概念，是某些型别的集合，请注意与字面值区分，其是在代码中明确写出的值。
    3. 在 C++ 11 环境中，constexpr 函数的用途仅限于此，即 constexpr 函数只能调用其他 constexpr 函数。C++ 14 则放宽了限制，只要不在 constexpr 函数内部改动非局部变量，我们就几乎可以进行任意操作。
        ```
        class CX
        {
        private:
            int a;
            int b;
        public:
            CX() = default;
            constexpr CX(int a_, int b_)
                : a(a_), b(b_)
            {}
            constexpr int get_a() const
            {
                return a;
            }
            constexpr int get_b()
            {
                return b;
            }
            constexpr int foo()
            {
                return a+b;
            }
        }
        ```
        - 根据 C++11 标准，get_a() 上的 const 现在成了多余的修饰，因其限定作用已经为 constexpr 关键字所蕴含。
    4. 如果只有通过复杂的方法，才可求得某些数组界限或整型常量，那么凭借 constexpr 函数完成任务将省去大量运算。
    5. 一旦涉及用户自定义型别，常量表达式和 constexpr 函数带来的主要好处是：若依照常量表达式初始化字面值型别的对象，就会发生静态初始化，从而避免初始化的条件竞争和次序问题。构造函数同样遵守这条规则。假定构造函数声明成了 constexpr 函数，且它的参数都是常量表达式，那么所属的类就会进行常量初始化，该初始化行为会在程序静态化阶段发生。
    6. 在实践中，常量往往在编译期就完成计算，在运行期直接套用算好的值。
    7. 让用户自定义的构造函数担负起静态初始化工作，而在运行任何其他代码之前，静态初始化肯定已经完成，我们遂能避免任何牵涉初始化的条件竞争。
    8. 若 std::mutex 类的构造函数受条件竞争所累，其全局实例就无法发挥功效，因此我们将它的默认构造函数声明成 constexpr 函数，以确保其初始化总是在静态初始化阶段内完成。
        
8. constexpr 对象
    1. constexpr 限定符会查验对象的初始化行为，核实其所依照的初值是常量表达式、constexpr 构造函数，或由常量表达式构成的聚合体初始化表达式。它还将对象声明为 const 常量。
        ```
        constexpr int i = 45;               // 正确
        constexpr std::string s("hello");   // 错误，std::string 不是字面值型别
        int foo(); 
        constexpr int j = foo();            // 错误，foo() 并未声明为 constexpr 函数
        ```

9. constexpr 函数要符合的条件
    1. C++11 标准对 constexpr 函数的要求如下：
        - 所有参数都必须是字面值型别。
        - 返回值必须是字面值型别。
        - 整个函数体只有一条 return 语句。
        - return 语句返回的表达式必须是常量表达式。
        - 若 return 返回的表达式需要转换为某目标型别的值，涉及的构造函数或转换操作符必须是 constexpr 函数。
    2. C++14 标准大幅度放宽了要求，即 constexpr 函数仍是纯函数，不产生副作用，但其函数体能够包含的内容显著增加。
        - 准许存在多条 return 语句。
        - 函数中创建的对象可被修改。
        - 可以使用循环、条件分支和 switch 语句。
    3. 类所具有的 constexpr 成员函数则需符合更多要求。
        - constexpr 成员函数不能是虚函数。
        - constexpr 成员函数所属的类必须是字面值型别。
    4. constexpr 构造函数需遵守不同的规则：
        - 在 C++ 11 环境下，构造函数的函数体必须为空。而根据 C++ 14 和后来的标准，它必须满足其他要求才可以成为 constexpr 标准。
        - 必须初始化每一个基类。
        - 必须初始化全体非静态数据成员。
        - 在成员初始化列表中，每个表达式都必须是常量表达式。
        - 若数据成员和基类分别调用自身的构造函数进行初始化，则它们所选取执行的必须是 constexpr 构造函数。
        - 假设在构造数据成员和基类时，所依照的初始化表达式为进行类型转换而调用了相关的构造函数或转换操作符，那么执行的必须是 constexpr 函数。
    5. 平实拷贝构造函数是隐式的 constexpr 函数。

10. constexpr 与模板
    1. 如果函数模板与类模板的成员函数加上 constexpr 修饰，而在模板的某个特定的具现化中，其参数和返回值却不属于字面值型别，则 constexpr 关键字会被忽略。该特性让我们可以写出一种函数模板，若选取了恰当的模板参数型别，它就具现化为 constexpr 函数，否则就具现化为普通的 inline 函数。
    2. 具现化的函数模板必须满足前文的全部要求，才可以成为 constexpr 函数。即便是函数模板，一旦它含有多条语句，我们就不能用关键字 constexpr 修饰其声明；这仍将导致编译错误。（此处特指 C++ 11 情形。在 C++ 14 中，constexpr() 函数模板可以合法含有多条语句，前提是符合前文所列要求）

11. lambda 函数
    1. 如果 lambda 函数的函数体仅有一条返回语句，那么 lambda 函数的返回值型别就是表达式的型别。
    2. 假若 lambda 函数的函数体无法仅用一条 return 语句写成，这时就需要明确设定返回值型别。设定返回值型别的方法是在 lambda 函数的参数列表后附上箭头和目标型别。如果 lambda 函数不接收任何参数，而返回值型别却需显式设定，我们依然必须使之包含空参数列表。
        ```
        cond.wait(lk, []()->bool { return data_ready; });
        ```
    3. lambda 函数的真正厉害之处在于捕获本地变量。
    4. 要捕获本地作用域内的全体变量，最简单的方式是改用 lambda 引导符“[=]”。改用该引导符的 lambda 函数从创建开始，即可访问本地变量的副本。
        ```
        std::funtion<int(int)> make_offseter(int offset)
        {
            return [=](int j) { return offset+j; };
        }
        ```
    5. 还可以采用别的手段：按引用的形式捕获全部本地变量。照此处理，一旦 lambda 函数脱离生成函数或所属代码块的作用域，引用的变量即被销毁，若仍然调用 lambda 函数，就会导致未定义行为。
    6. 还有另一种做法：我们可将按引用捕获设定成默认行为，但以复制方式捕获某些特定变量。这种处理方式使用形如“[&]”的 lambda 引导符，并在“&”后面逐一列出需要复制的变量。
    7. 若要我们仅仅想要某几个具体变量，并按引用方式捕获，而非复制，就应该略去上述最开始的等号或“&”，且逐一列出目标变量，再为它们加上“&”前缀。
    8. 当 lambda 函数位于一个类的某成员函数内部时，我们在 lambda 函数中访问类成员时要务必注意。类的数据成员无法直接获取，若想从 lambda 函数内部访问类的数据成员，则须在捕获列表中加上 this 指针以捕获之。
    9. 从 C++ 14 开始，lambda 函数也能有泛型形式，其中的参数型别被声明成 auto，而非具体型别。这么一来，lambda 函数的调用操作符就是隐式模板，参数型别根据运行时外部提供的参数推导得出：
        ```
        auto f = [](auto x) { std::cout << "x=" << x << std::endl; };
        f(42);
        f("hello");
        ```
    10. C++ 14 还加入了广义捕获的概念，我们因此能够捕获表达式的运算结果，而不再限于直接复制或引用本地变量。该特性最常用于以移动方式捕获只移型别，从而避免以引用方式捕获。
        ```
        std::future<int> spawn_async_task()
        {
            std::promise<int> p;
            auto f = p.get_future();
            std::thread t([p = std::move(p)() { p.set_value(find_the_answer()); }]);
            t.detach();
            return f;
        }
        ```
        - 这里的 p = std::move(p) 就是广义捕获行为，它将 promise 实例移入 lambda 函数，因此线程可以安全地分离，我们不必担心本地变量被销毁而形成悬空引用。Lambda 函数完成构建后，原来的实例 p 即进入“移出状态”，因此我们事先从它取得了关联的 future 实例。

12. 变参模板
    1. 变参模板即参数数目可变的模板。
    2. 我们声明变参函数时，需令函数参数列表包含一个省略号（...）。变参模板与之相同，在其声明中，模板参数列表也需带有省略号：
        ```
        template<typename ...ParameterPack>
        class my_template
        {};
        ```
    3. 变参模板的另外两个特性。第一特性相对简单：普通模板参数（ReturnType）和可变参数（Args）能在同一声明内共存。所示的第二特性是，在 packaged_task 的特化版本中，其模板参数列表使用了组合标记“Args...”，当模板具现化时，Args 所含的各种型别均据此列出。这是个偏特化版本，因而它会进行模式匹配：在模板实例化的上下文中，出现的型别被全体捕获并打包成 Args。该可变参数 Args 叫作参数包，应用“Args...”还原参数列表则称为包展开。
        ```
        template<typename ReturnType, typename ...Args>
        class packaged_task<ReturnType(Args...)>;
        ```
    4. 我们可以依照某种模式创建元组，使得其中的成员型别都是普通指针，甚至都是 std::unique_ptr<> 指针，其目标型别对应参数包中的元素。
        ```
        template<typename ...Params>    // ①
        struct dummy3
        {
            std::tuple<Params* ...> pointers;   // ②
            std::tuple<std::unique_ptr<Params> ...> unique_pointers;    // ③
        }
        ```
        - ①处省略号是变参模板声明的语法成分，表示型别参数的数目可变，②③两处的省略号标示出展开模式。②处的模式是型别表达式 Params*，而③处的模式则是型别表达式 std::unique_ptr<Params>。
    5. 我们也可以用某种展开模式来声明函数参数，与前文按模式展开参数包的做法相似。例如，std::thread 类的构造函数正是采取了这种方法，按右值引用的形式接收全部函数参数：
        ```
        template<typename CallableType, typename ...Args>
        thread::thread(CallableType&& func, Args&& ...args);
        ```
    6. 借 std::forward<> 灵活保有函数参数的右值属性。
        ```
        template<typename ...ArgsTypes>
        void bar(ArgsTypes&& ...args)
        {
            foo(std::forward<ArgsTypes>(args)...);
        }
        ```
        - 利用 std::forward<> 完美转发：若是左值，传递之后仍是左值；若是右值，传递之后仍是右值。否则，一个右值引用参数作为函数的形参，在函数内部再转发该参数的时候它已经变成一个左值。
    7. 我们通过 sizeof... 运算符确定参数包大小，写法十分简单：sizeof...(p) 即为参数包 p 所含元素的数目。sizeof... 运算符求得的值是常量表达式，这与普通的 sizeof 运算符一样，故其结果可用于设定数组长度，以及其他合适的场景中。

13. 自动推导变量的型别
    1. 若变量在声明时即进行初始化，所依照的初值与自身型别相同，我们就能以 auto 关键字设定其类型。

14. 线程局部变量 
    1. 在程序中，若将变量声明为线程局部变量，则每个线程上都会存在其独立实例。在声明变量时，只要加入关键字 thread_local 标记，它即成为线程局部变量。
    2. 有 3 种数据能声明为线程局部变量：以名字空间为作用域的变量、类的静态数据成员和普通的局部变量。换言之，它们具有线程存储生存期。
        ```
        thread_local int x;     // 线程局部变量，它以名字空间为作用域
        class X
        {
            static thread_local std::string s;  // 类的静态数据成员，该语句用于声明
        }
        static thread_local std::string X::s;   // 该语句用于定义，类的静态数据成员应在外部另行定义
        void foo()
        {
            thread_local std::vector<int> v;    // 普通的局部变量
        }
        ```
    3. 实际上，在给定的翻译单元中，若所有线程局部变量从未被使用，就无法保证会把它们构造出来。这使得含有线程局部变量的模板得以动态加载，当给定线程初次指涉模板中的线程局部变量时，才进行动态加载，进而构造变量。
    4. 翻译单元，是有关 C++ 代码编译的术语，指当前代码所在的源文件，以及经过预处理后，全部有效包含的头文件和其他源文件。
    5. 对于函数内部声明的线程局部变量，在某个给定的线程上，当控制流程第一次经过其声明语句时，该变量就会初始化。假设某函数在给定的线程上从来都没有被调用，函数中却声明了线程局部变量，那么在该线程上它们均不会发生构造。这一行为规则与静态局部变量相同，但它对每个线程都单独起作用。
    6. 线程局部变量的其他性质与静态变量一致，它们先进行零值初始化，再进行其他变量初始化（如动态初始化）。如果线程局部变量的构造函数抛出异常，程序就会调用 std::terminate() 而完全终止。
    7. 动态初始化，指除非静态初始化（指零值初始化和常量初始化）以外的一切初始化行为。
    8. 给定一个线程，在其线程函数返回之际，该线程上构造的线程局部变量全都会发生析构，它们调用析构函数的次序与调用构造函数的次序相反。由于这些变量的初始化次序并不明确，因此必须保证它们的析构函数间没有相互依赖。若线程局部变量的析构函数因抛出异常而退出，程序则会调用 std::terminate()，与构造函数的情形一样。
    9. 如果线程通过调用 exit() 退出，或从 main() 自然退出（这等价于先取得 main() 的返回值，再以该值调用 std::exit()），那么线程局部变量也会被销毁。当应用程序退出时，如果有其他线程还在运行，则那些线程上的线程局部变量不会发生析构。
    10. 线程局部变量的地址因不同线程而异，但我们依然可以令一个普通指针指向该变量。假定该指针的值源于某线程所执行的取址操作，那么它指涉的目标对象就位于该线程上，其他线程也能通过这一指针访问那个对象。若线程在对象销毁后还试图访问它，将导致未定义行为（向来如此）。所以，若我们向另外一个线程传递指针，其目标是线程局部变量，那就需要确保在目标变量所属的线程结束后，该指针不会再被提取。

15. 类模板的参数推导
    1. C++ 17 拓展了模板参数的自动推导型别的思想：如果我们通过一个模板声明一个对象，那么在大多数情况下，根据该对象的初始化表达式，能推导出模板参数的型别。
    2. 具体来说，若仅凭某个类模板的名字声明对象，却未设定模板参数列表，编译器就会根据对象的初始化表达式，指定调用类模板的某个构造函数，还借以推导模板参数，而函数模板也将发生普通的型别推导，这两个推导机制遵守相同的规则。
        ```
        std::mutex m;
        std::lock_guard guard(m);   // 将推导出 std::lock_guard<std::mutex>
        
        std::mutex m1;
        std::shard_mutex m2;
        std::scoped_lock guard(m1, m2); // 将推导出 std::scoped_lock<std::mutex, std::shard_mutex>
        ```

16. C++11 增加的新特性包括静态断言（static assertion/static_assert）、强类型枚举（strongly typed enumeration/enum class）、委托构造（delegating constructor）函数、Unicode 编码支持、模板别名（template alias）和新式的统一初始化列表（uniform initialization sequence），以及许多相对细小的改变。
