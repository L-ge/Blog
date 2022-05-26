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
