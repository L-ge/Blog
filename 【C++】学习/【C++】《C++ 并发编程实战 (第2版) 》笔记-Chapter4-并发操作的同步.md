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
