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
