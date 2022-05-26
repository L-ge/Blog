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
