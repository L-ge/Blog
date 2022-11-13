##### 一、创建型模式

1. 工厂模式
- 作用：
    1. 定义创建对象的接口，封装了对象的创建。
    2. 使得具体化类的工作延迟到了子类中。
- 其他：
    1. 如果为每一个具体的 ConcreteProduct 类的实例化提供一个函数体，那么我们可能不得不在 ConcreteFactory 中添加一个方法来处理这个新建的 ConcreteProduct。
    2. Factory 模式对于对象的创建给予开发人员提供了很好的实现策略，但是 Factory 模式仅仅局限于一类类（就是说 Product 是一类，有一个共同的基类），如果我们要为不同类的类提供一个对象创建的接口，那就要用 AbstractFactory 了。
- 代码演示：
    ```
    class Product
    {
    public:
        virtual ~Product() = 0;
    }
    
    class ConcreteProduct : public Product      // 具体工厂类
    { 
    public: 
        ConcreteProduct(); 
        ~ConcreteProduct(); 
    }; 
    
    class Factory
    {
    public:
        virtual ~Factory() = 0;
        virtual Product* CreateProduct() = 0; 
    };
    
    class ConcreteFactory : public Factory      // 具体工厂类
    { 
    public: 
        ConcreteFactory(); 
        ~ConcreteFactory(); 
        
        Product* CreateProduct()
        {
            return new ConcreteProduct();
        }
    }; 
    
    Factory* f = new ConcreteFactory(); 
    Product* p = fac->CreateProduct(); 
    ```

2. 抽象工厂模式
- 作用：
    1. 创建一组相关或者相互依赖的对象。
- 其他：
    1. AbstractFactory模式是为创建一组（有多类）相关或依赖的对象提供创建接口，而Factory模式是为一类对象提供创建接口或延迟对象的创建到子类中实现。
- 代码演示：
    ```
    class AbstractApple 
    {
    public:
        virtual void showName() = 0;
    };
    
    class ChinaApple : public AbstractApple
    {
    public:
        virtual void showName() 
        {
            cout << "中国苹果" << endl;
        }
    };
    
    class JapanApple : public AbstractApple 
    {
    public:
        virtual void showName() 
        {
            cout << "日本苹果" << endl;
        }
    };
    
    class AbstractBanana 
    {
    public:
        virtual void showName() = 0;
    };
    
    class ChinaBanana : public AbstractBanana
    {
    public:
        virtual void showName() 
        {
            cout << "中国香蕉" << endl;
        }
    };
    
    class JapanBanana : public AbstractBanana 
    {
    public:
        virtual void showName() 
        {
            cout << "日本香蕉" << endl;
        }
    };
    
    class AbstractFactory 
    {
    public:
        virtual AbstractApple*  CreateApple() = 0;
        virtual AbstractBanana* CreateBanana() = 0;
    };
    
    class ChinaFactory : public AbstractFactory 
    {
        virtual AbstractApple* CreateApple() 
        {
            return new ChinaApple;
        }
        virtual AbstractBanana* CreateBanana() 
        {
            return new ChinaBanana;
        }
    };
    
    class JapanFactory : public AbstractFactory 
    {
        virtual AbstractApple* CreateApple() 
        {
            return new JapanApple;
        }
        virtual AbstractBanana* CreateBanana()
        {
            return new JapanBanana;
        }
    };
    
    AbstractFactory* factory = new ChinaFactory;
    AbstractApple* apple = factory->CreateApple();
    AbstractBanana* banana = factory->CreateBanana();
    apple->showName();
    banana->showName();
    ```

3. 单例模式
- 作用：
    1. Singleton 模式就是一个类只创建一个唯一的对象，即一次创建多次使用。
- 其他：
    1. Singleton 模式经常和 Factory（AbstractFactory）模式在一起使用，因为系统中工厂对象一般来说只要一个。
- 代码演示：
    ```
    template<class T>
    class Singleton
    {
    public:
        static T* GetInstance()
        {
            static T v;
            return &v;
        }
    };
    
    class TestClass
    {
    public:
        static TestClass & getInstance()
        {
            static TestClass instance;
            return instance;
        }
    };
    ```

4. 建造者模式
- 作用：
    1. 当我们要创建的对象很复杂的时候（通常是由很多其他的对象组合而成），我们要将复杂对象的创建过程和这个对象的表示（展示）分离开来，这样做的好处就是通过一步步地进行复杂对象的构建，由于在每一步的构造过程中可以引入参数，使得经过相同的步骤创建最后得到的对象的展示不一样。
- 其他：
    1. Builder 模式的关键是其中的 Director 对象并不直接返回对象，而是通过一步步（BuildPartA，BuildPartB，BuildPartC）来一步步进行对象的创建。
    2. Builder 模式的示例代码中，BuildPart 的参数是通过客户程序员传入的，这里使用“user-defined”代替，实际的可在 Construct 方法中传入这 3 个参数，这样就可以得到不同的细微差别的复杂对象了。
    3. Builder 模式和 AbstractFactory 模式在功能上很相似，因为都是用来创建大的复杂的对象，它们的区别是 Builder 模式强调的是一步步创建对象，并通过相同的创建过程可以获得不同的结果对象，一般来说 Builder 模式中对象不是直接返回的。而在 AbstractFactory 模式中对象是直接返回的，AbstractFactory 模式强调的是为创建多个相互依赖的对象提供一个同一的接口。
    4. 当一个类的构造函数参数个数超过4个，而且这些参数有些是可选的参数，考虑使用构造者模式。
- 代码演示：
    ```
    class Product           // 最终要生成的对象
    { 
    public: 
        Product(); 
        ~Product(); 
        void ProducePart(); 
    }; 
    
    class Builder           // 建造者抽象类
    { 
    public: 
        virtual ~Builder(); 
        
        // 构建 Product 的抽象步骤
        virtual void BuildPartA(const string& buildPara) = 0;
        virtual void BuildPartB(const string& buildPara) = 0; 
        virtual void BuildPartC(const string& buildPara) = 0; 
        
        // 返回最终 Product 的方法
        virtual Product* GetProduct() = 0;
    protected: 
        Builder(); 
    }; 
    
    class ConcreteBuilder : public Builder      // 具体建造者
    { 
    public: 
        ConcreteBuilder(); 
        ~ConcreteBuilder(); 
        void BuildPartA(const string& buildPara);
        void BuildPartB(const string& buildPara);
        void BuildPartC(const string& buildPara);
        Product* GetProduct()
        {
            return new Product(); 
        }
    }; 
    
    class Director          // 指导者类。决定如何构建最终 Product
    { 
    public: 
        Director(); 
        ~Director(); 
        
        // 通过调用 builder 的方法，就可以设置builder，
        // 等设置完成后，就可以通过 builder 的 getProduct() 方法获得最终的 Product
        void Construct(Builder* b)    
        {
            b->BuildPartA("user-defined"); 
            b->BuildPartB("user-defined"); 
            b->BuildPartC("user-defined");  
        }
    }; 
    
    Builder* b = new ConcreteBuilder()
    Director* d = new Director(); 
    d->Construct(b);
    Product* p = b->GetProduct();
    ```

5. 原型模式
- 作用：
    1. Prototype 模式提供了自我复制的功能，就是说新对象的创建可以通过已有对象进行创建。（深拷贝）
    2. Prototype 模式提供了一个通过已存在对象进行新对象创建的接口（Clone），Clone接口在 C++ 中我们将通过拷贝构造函数实现。
- 其他：
    1. Prototype 模式通过复制原型（Prototype）而获得新对象创建的功能，这里 Prototype 本身就是“对象工厂”（因为能够生产对象），实际上 Prototype 模式和 Builder 模式、AbstractFactory 模式都是通过一个类（对象实例）来专门负责对象的创建工作（工厂对象），它们之间的区别是：Builder 模式重在复杂对象的一步步创建（并不直接返回对象），AbstractFactory 模式重在产生多个相互依赖类的对象，而 Prototype 模式重在从自身复制自己创建新类。
    - 原型模式的本质就是 clone，可以解决构建复杂对象的资源消耗问题，能在某些场景中提升构建对象的效率；还有一个重要的用途就是保护性拷贝，可以通过返回一个拷贝对象的形式，实现只读的限制。
- 代码演示：
    ```C++
    class Prototype 
    { 
    public: 
        virtual ~Prototype(); 
        virtual Prototype* Clone() const = 0; 
    protected: 
        Prototype(); 
    }; 
    
    class ConcretePrototype : public Prototype 
    { 
    public: 
        ConcretePrototype(); 
        ConcretePrototype(const ConcretePrototype& cp); 
        ~ConcretePrototype(); 
        
        Prototype* Clone() const
        {
            return new ConcretePrototype(*this); 
        }
    }; 
    Prototype* p = new ConcretePrototype(); 
    Prototype* p1 = p->Clone(); 
    ```

---

##### 二、结构型模式

1. 桥接模式
- 作用：
    1. 
- 其他：
    1. 桥接模式实现了抽象化与实现化的脱耦。他们两个互相独立，不会影响到对方。
    2. 对于两个独立变化的维度，使用桥接模式再适合不过了。
    3. 分离抽象接口及其实现部分，提高了比继承更好的解决方案。
- 代码演示：
    ```
    class Abstraction 
    { 
    public: 
        virtual ~Abstraction(); 
        virtual void Operation() = 0; 
    }; 
    
    class RefinedAbstraction : public Abstraction 
    { 
    public: 
        RefinedAbstraction::RefinedAbstraction(AbstractionImp* imp)
        { 
            _imp = imp; 
        } 
         
        ~RefinedAbstraction(); 
        void Operation()
        {
            _imp->Operation(); 
        }
     
    private: 
        AbstractionImp* _imp; 
    }; 
    
    class AbstractionImp 
    { 
    public: 
        virtual ~AbstractionImp(); 
        virtual void Operation() = 0; 
    protected: 
        AbstractionImp(); 
    };
    
    class ConcreteAbstractionImpA : public AbstractionImp
    { 
    public: 
        ConcreteAbstractionImpA(); 
        ~ConcreteAbstractionImpA(); 
        virtual void Operation();
    }; 
    
    class ConcreteAbstractionImpB : public AbstractionImp
    { 
    public: 
        ConcreteAbstractionImpB(); 
        ~ConcreteAbstractionImpB(); 
        virtual void Operation();
    }; 
    
    AbstractionImp* imp = new ConcreteAbstractionImpA(); 
    Abstraction* abs = new RefinedAbstraction(imp); 
    abs->Operation(); 
    ```

2. 适配器模式
- 作用：
    1. 
- 其他：
    1. 适配器模式分为类模式和对象模式。
- 代码演示：
    ```
    class Target 
    { 
    public: 
        Target(); 
        virtual ~Target();
        virtual void Request(); 
    };
    
    class Adaptee
    { 
    public: 
        Adaptee();
        ~Adaptee();
        void SpecificRequest(); 
    };
    
    class Adapter : public Target 
    { 
    public: 
        Adapter(Adaptee* ade)
        { 
            this->_ade = ade;
        } 
        ~Adapter(); 
        void Request()
        {
            _ade->SpecificRequest(); 
        }
    private: 
        Adaptee* _ade;
    };
    
    Adaptee* ade = new Adaptee;
    Target* adt = new Adapter(ade);
    adt->Request();
    ```

3. 装饰器模式
- 作用：
    1. 对类的功能进行扩展（代替继承）。可以动态的给类增加功能。
    2. 装饰器模式（Decorator Pattern）允许向一个现有的对象添加新的功能，同时又不改变其结构。
        - Component（被装饰对象的基类）：定义一个对象接口，可以给这些对象动态地添加职责。
        - ConcreteComponent（具体被装饰对象）：定义一个对象，可以给这个对象添加一些职责。
        - Decorator（装饰者抽象类）：维持一个指向 Component 实例的引用，并定义一个与 Component 接口一致的接口。
        - ConcreteDecorator（具体装饰者）：具体的装饰对象，给内部持有的具体被装饰对象，增加具体的职责。
- 其他：
    1. Decorator 模式除了采用组合的方式取得了比采用继承方式更好的效果，Decorator 模式还给设计带来一种“即用即付”的方式来添加职责。在 OO 设计和分析经常有这样一种情况：为了多态，通过父类指针指向其具体子类，但是这就带来另外一个问题，这样处于高层的父类就承载了太多的特征（方法），并且继承自这个父类的所有子类都不可避免继承了父类的这些接口，但是可能这并不是这个具体子类所需要的。这样父类就太过臃肿了，那么当需要添加一个操作的时候就可以通过 Decorator 模式来解决。
- 代码演示：
    ```
    //我们需要为英雄增加额外的功能或者属性，但是又不能影响Hero类，这个时候就可以考虑装饰器模式
    
    class AbstractHero      //抽象的英雄
    {
    public:
    	virtual void showStatus() = 0;
    
    	int mHp;
    	int mMp;
    	int mAt;
    	int mDf;
    };
    
    class HeroA : public AbstractHero
    {
    public:
    	HeroA()
    	{
    		mHp = 0;
    		mMp = 0;
    		mAt = 0;
    		mDf = 0;
    	}
    
    	virtual void showStatus()
    	{
    		cout << "血量：" << mHp << endl;
    		cout << "魔法：" << mMp << endl;
    		cout << "攻击：" << mAt << endl;
    		cout << "防御：" << mDf << endl;
    	}
    };
    
    //英雄穿上某个装饰物，那么也还是一个英雄
    class AbstractEquipment : public AbstractHero
    {
    public:
    	AbstractEquipment(AbstractHero* hero)
    	{
    		this->pHero = hero;
    	}
    
    	virtual void showStatus()
    	{
    	}
    
    	AbstractHero* pHero;
    };
    
    //给英雄穿上狂徒铠甲
    class KuangtuEquipment : public AbstractEquipment
    {
    public:
    	KuangtuEquipment(AbstractHero* hero) : AbstractEquipment(hero){}
    
    	//增加额外的功能
    	void AddKuangtu()
    	{
    		cout << "英雄穿了狂徒之后" << endl;
    		this->mHp = this->pHero->mHp;
    		this->mMp = this->pHero->mMp;
    		this->mAt = this->pHero->mAt;
    		this->mDf = this->pHero->mDf+30;
    	}
    
    	virtual void showStatus()
    	{
    		AddKuangtu();
    		cout << "血量：" << mHp << endl;
    		cout << "魔法：" << mMp << endl;
    		cout << "攻击：" << mAt << endl;
    		cout << "防御：" << mDf << endl;
    	}
    };
    
    //无尽之刃
    class Wujinzhireng : public AbstractEquipment
    {
    public:
    	Wujinzhireng(AbstractHero* hero) : AbstractEquipment(hero) {}
    
    	//增加额外的功能
    	void AddKuangtu()
    	{
    		cout << "英雄穿了无尽之刃后" << endl;
    		this->mHp = this->pHero->mHp;
    		this->mMp = this->pHero->mMp;
    		this->mAt = this->pHero->mAt+80;
    		this->mDf = this->pHero->mDf;
    	}
    
    	virtual void showStatus()
    	{
    		AddKuangtu();
    		cout << "血量：" << mHp << endl;
    		cout << "魔法：" << mMp << endl;
    		cout << "攻击：" << mAt << endl;
    		cout << "防御：" << mDf << endl;
    	}
    };
    
	AbstractHero* hero = new HeroA;
	hero->showStatus();
	//给英雄穿上狂徒
	hero = new KuangtuEquipment(hero);
	hero->showStatus();
	hero->mAt = 33;
	//给英雄装备武器
	hero = new Wujinzhireng(hero);
	hero->showStatus();
    ```

4. 组合实体模式
- 作用：
    1. 将对象组合成树形结构以表示“部分-整体”的层次结构。Composite 使得用户对单个对象和组合对象的使用具有一致性。
- 其他：
    1. 
- 代码演示：
    ```
    class Component     // 组合中的抽象基类
    { 
    public: 
        Component(){} 
        virtual ~Component(){} 
    
        // 纯虚函数,只提供接口,没有默认的实现   
        virtual void Operation() = 0;   
        
        // 虚函数,提供接口,有默认的实现就是什么都不做
        virtual void Add(Component* pChild); 
        virtual void Remove(Component* pChild); 
        virtual Component* GetChild(int nIndex); 
    }; 
    
    class Leaf : public Component   // 派生自 Component,是其中的叶子组件的基类
    { 
    public: 
        Leaf(){} 
        virtual ~Leaf(){} 
        virtual void Operation(); 
    }; 
    
    class Composite : public Component  // 派生自 Component,是其中的含有子件的组件的基类 
    { 
    public: 
        Composite(){}
        
        virtual ~Composite()
        {
            std::list<Component*>::iterator iter1, iter2, temp; 
            for (iter1 = m_ListOfComponent.begin(), iter2 = m_ListOfComponent.end(); iter1 != iter2; ) 
            { 
                temp = iter1; 
                ++iter1; 
                delete (*temp); 
            } 
        }
        
        virtual void Operation()
        {
             std::list<Component*>::iterator iter1, iter2; 
             for(iter1 = m_ListOfComponent.begin(), iter2 = m_ListOfComponent.end(); iter1 != iter2; ++iter1) 
             { 
                (*iter1)->Operation(); 
             } 
        }
        
        virtual void Add(Component* pChild)
        {
            m_ListOfComponent.push_back(pChild); 
        }
        
        virtual void Remove(Component* pChild)
        {
            std::list<Component*>::iterator iter = find(m_ListOfComponent.begin(), m_ListOfComponent.end(), pChild); 
            if (m_ListOfComponent.end() != iter) 
            { 
                m_ListOfComponent.erase(iter); 
            } 
        }
        
        virtual Component* GetChild(int nIndex)
        {
            if(nIndex <= 0 || nIndex > m_ListOfComponent.size()) 
                return NULL; 
            std::list<Component*>::iterator iter1, iter2; 
            int i; 
            for(i = 1, iter1 = m_ListOfComponent.begin(), iter2 = m_ListOfComponent.end(); iter1 != iter2; ++iter1, ++i) 
            { 
                if(i == nIndex) 
                    break; 
            } 
            return *iter1; 
        }
        private: 
            // 采用 list 容器去保存子组件
            std::list<Component*> m_ListOfComponent; 
    }; 
    
    Leaf *pLeaf1 = new Leaf(); 
    Leaf *pLeaf2 = new Leaf(); 
    Composite* pComposite = new Composite; 
    pComposite->Add(pLeaf1); 
    pComposite->Add(pLeaf2); 
    pComposite->Operation(); 
    pComposite->GetChild(2)->Operation(); 
    delete pComposite; 
    ```

5. 享元模式
- 作用：
    1. 适用性: 一个应用程序使用了大量对象；完全由于使用大量的对象造成很大的存储开销；对象的大多数状态都可变为外部状态；如果删除对象的外部状态，那么可以使用相对较少的共享对象取代外部对象。
- 其他：
    1. Flyweight 模式中有一个类似 Factory 模式的对象构造工厂 FlyweightFactory，当客户程序员（Client）需要一个对象时候就会向 FlyweightFactory 发出请求对象的消息 GetFlyweight() 消息，FlyweightFactory 拥有一个管理、存储对象的“仓库”（或者叫对象池，vector 实现），GetFlyweight() 消息会遍历对象池中的对象，如果已经存在则直接返回给 Client，否则创建一个新的对象返回给 Client。
- 代码演示：
    ```
    class Flyweight 
    { 
    public: 
        virtual ~Flyweight(){} 
        std::string GetIntrinsicState()
        {
            return m_State;
        }
        virtual void Operation(std::string& ExtrinsicState) = 0; 
    protected: 
        Flyweight(const std::string& state) : m_State(state) 
        { 
        } 
    private: 
        std::string m_State; 
    }; 
    
    class FlyweightFactory 
    { 
    public: 
        FlyweightFactory(){} 
        ~FlyweightFactory()
        {
            std::list<Flyweight*>::iterator iter1, iter2, temp; 
            for(iter1 = m_listFlyweight.begin(), iter2 = m_listFlyweight.end(); iter1 != iter2; ) 
            { 
                temp = iter1; 
                ++iter1; 
                delete (*temp); 
            } 
            m_listFlyweight.clear(); 
        }
        Flyweight* GetFlyweight(const std::string& key)
        {
            std::list<Flyweight*>::iterator iter1, iter2; 
            for(iter1 = m_listFlyweight.begin(), iter2 = m_listFlyweight.end(); iter1 != iter2; ++iter1) 
            { 
                if ((*iter1)->GetIntrinsicState() == key) 
                {
                    return (*iter1); 
                } 
            }
            
            Flyweight* flyweight = new ConcreateFlyweight(key); 
            m_listFlyweight.push_back(flyweight); 
        }
    private: 
        std::list<Flyweight*> m_listFlyweight; 
    }; 
    
    class ConcreateFlyweight : public Flyweight 
    { 
    public: 
        ConcreateFlyweight(const std::string& state) 
            : Flyweight(state) 
        { 
        } 
        virtual ~ConcreateFlyweight(){} 
        virtual void Operation(std::string& ExtrinsicState); 
    };
    
    FlyweightFactory flyweightfactory; 
    flyweightfactory.GetFlyweight("hello"); 
    flyweightfactory.GetFlyweight("world"); 
    flyweightfactory.GetFlyweight("hello"); 
    ```

6. 外观模式
- 作用：
    1. 外观模式是一个很重要、平常也用得很多的一个设计模式。
- 其他：
    1. 
- 代码演示：
    ```
    class Television {      //电视机类
    public:
        void on() {
            cout << "电视机打开" << endl;
        }
        void off() {
            cout << "电视机关闭" << endl;
        }
    };
    
    class Light {           //灯类
    public:
        void on() {
            cout << "灯打开" << endl;
        }
        void off() {
            cout << "灯关闭" << endl;
        }
    };
    
    class Audio {           //音响
    public:
        void on() {
            cout << "音响打开" << endl;
        }
        void off() {
            cout << "音响关闭" << endl;
        }
    };
    
    class Microphone {      //麦克风
    public:
        void on() {
            cout << "麦克风打开" << endl;
        }
        void off() {
            cout << "麦克风关闭" << endl;
        }
    };
    
    class DVDplayer {       //DVD
    public:
        void on() {
            cout << "DVD打开" << endl;
        }
        void off() {
            cout << "DVD关闭" << endl;
        }
    };
    
    class GameMachine {     //游戏机
    public:
        void on() {
            cout << "游戏机打开" << endl;
        }
        void off() {
            cout << "游戏机关闭" << endl;
        }
    };
    
    class KTVMode {
    public:
        KTVMode() {
            pTV = new Television;
            pLight = new Light;
            pAudio = new Audio;
            pMicrophone = new Microphone;
            pDvd = new DVDplayer;
        }
    
        void onKTV() {
            pTV->on();
            pLight->off();          // 注意
            pAudio->on();
            pMicrophone->on();
            pDvd->on();
        }
    
        void offKTV() {
            pTV->off();
            pLight->on();           // 注意
            pAudio->off();
            pMicrophone->off();
            pDvd->off();
        }
    
        ~KTVMode() {
            delete pTV;
            delete pLight;
            delete pAudio;
            delete pMicrophone;
            delete pDvd;
        }
    private:
        Television* pTV;
        Light* pLight;
        Audio* pAudio;
        Microphone* pMicrophone;
        DVDplayer* pDvd;
    };
    
    KTVMode* ktv = new KTVMode;
    ktv->onKTV();       //KTV包厢来人了：灯关闭，其他设备开启。
    ```

7. 代理模式
- 作用：
    1. 代理模式：提供一种代理来控制其他对象的访问。
    2. 代理模式（Proxy Pattern）是指为其他对象提供一种代理，以控制对这个对象的访问。代理对象在客服端和目标对象之间起到中介作用。
- 其他：
    1. 和适配器模式的区别：适配器模式主要改变所考虑对象的接口，而代理模式不能改变所代理类的接口。
    2. 和装饰器模式的区别：装饰器模式为了增强功能，而代理模式是为了加以控制。
- 代码演示：
    ```C++
    class AbstractCommonInterface {
    public:
        virtual void run() = 0;
    };
    
    //下面是操作系统类
    class MySystem : public AbstractCommonInterface {
    public:
        virtual void run() {
            cout << "系统启动" << endl;
        }
    };
    
    //代理：启动系统必须要权限验证,不是所以的人都可以来启动我的系统,必须要提供用户名和密码
    class MySystemProxy : public AbstractCommonInterface {
    public:
        MySystemProxy(string userName, string password) {
            this->mUserName = userName;
            this->mPassword = password;
            pMySystem = new MySystem;
        }   
        
        ~MySystemProxy() {
            if (pMySystem != NULL) {
                delete pMySystem;
            }
        }
    
        bool checkUserNameAndPassword() {
            if(mUserName == "admin" && mPassword == "admin") {
                return true;
            }
            return false;
        }
    
        virtual void run() {
            if(checkUserNameAndPassword() == true) {
                cout << "启动成功" << endl;
                this->pMySystem->run();
            }
            else {
                cout << "用户名或者密码错误,权限不足" << endl;
            }
        }
    
    private:
        string mUserName;
        string mPassword;
        MySystem* pMySystem;
    };
    
    MySystemProxy* proxy = new MySystemProxy("admin", "admin");
    proxy->run();
    ```

---

##### 三、行为模式

1. 模板模式
- 作用：
    1. 在面向对象系统的分析与设计过程中经常会遇到这样一种情况：对于某一个业务逻辑（算法实现）在不同的对象中有不同的细节实现，但是逻辑（算法）的框架（或通用的应用算法）是相同的。Template 提供了这种情况的一个实现框架。Template 模式是采用继承的方式实现这一点：将逻辑（算法）框架放在抽象基类中，并定义好细节的接口，子类中实现细节。
    2. 模板方法模式：在父类中定义一个方法的抽象，由它的子类来实现细节的处理，在子类实现详细的处理算法时并不会改变算法中的执行次序。
- 其他：
    1. 可以看到 Template 模式获得一种反向控制结构效果，这也是面向对象系统的分析和设计中一个原则，依赖倒置原则。其含义就是父类调用子类的操作（高层模块调用低层模块的操作），低层模块实现高层模块声明的接口。这样控制权在父类（高层模块），低层模块反而要依赖高层模块。
    2. 但是也有局限性，因为模板模式采用继承这种强制约束关系，导致复用性不高。假如我接下来要做冷饮，那么 Make 函数中的煮水操作其实可以省去直接冲泡即可，但是我们就无法再次使用这个模板了，这样就导致了不能复用子类的实现步骤。
- 代码演示：
    ```C++
    //做饮料模板
    class DrinkTemplate {
    public:
        //煮水
        virtual void BoildWater() = 0;
        //冲泡
        virtual void Brew() = 0;
        //倒入杯中
        virtual void PourInCup() = 0;
        //加辅助材料
        virtual void AddSomething() = 0;
    
        //模板方法
        void Make() {
            BoildWater();
            Brew();
            PourInCup();
            AddSomething();
        }
    };
    
    //做咖啡：实现做饮料模板
    class Coffee : public DrinkTemplate {
        virtual void BoildWater() {
            cout << "煮山泉水" << endl;
        }
        virtual void Brew() {
            cout << "冲泡咖啡" << endl;
        }
        virtual void PourInCup() {
            cout << "咖啡倒入杯中" << endl;
        }
        virtual void AddSomething() {
            cout << "加糖，加牛奶" << endl;
        }
    };
    
    //做茶：实现做饮料模板
    class Tea : public DrinkTemplate {
        virtual void BoildWater() {
            cout << "煮自来水" << endl;
        }
        virtual void Brew() {
            cout << "冲泡铁观音" << endl;
        }
        virtual void PourInCup() {
            cout << "茶水倒入杯中" << endl;
        }
        virtual void AddSomething() {
            cout << "加糖,加大蒜" << endl;
        }
    };
    
    Tea* tea = new Tea;
    tea->Make();
    
    Coffee* coffee = new Coffee;
    coffee->Make();
    ```

2. 策略模式
- 作用：
    1. Strategy 模式和 Template 模式要解决的问题是相同（类似）的，都是为了给业务逻辑（算法）具体实现和抽象接口之间的解耦。
    2. 简而言之，Strategy 模式是对算法的封装。处理一个问题的时候可能有多种算法,这些算法的接口（输入参数，输出参数等）都是一致的，那么可以考虑采用 Strategy 模式对这些算法进行封装，在基类中定义一个函数接口就可以了。
- 其他：
    1. Character 类是通过组合 WeaponStrategy 的形式调用武器使用的，这就比继承要好得多，因为在面向对象的设计中的有一条很重要的原则就是：优先使用对象组合，而非类继承。
    2. 优点：
        - 算法可以自由切换。
        - 避免使用多重条件判断。
        - 扩展性良好。
    3. 缺点：
        - 策略类会增多。
        - 所有策略类都需要对外暴露。
- 代码演示：
    ```
    // 抽象武器，策略基类（抽象的策略）
    class WeaponStrategy {
    public:
        virtual void UseWeapon() = 0;
    };
    
    // 具体的策略，使用匕首作为武器
    class Knife : public WeaponStrategy {
    public:
        virtual void UseWeapon() {
            cout << "使用匕首" << endl;
        }
    };
    
    //具体的策略，使用AK47作为武器
    class AK47 : public WeaponStrategy {
    public:
        virtual void UseWeapon() {
            cout << "使用AK47" << endl;
        }
    };
    
    //具体使用策略的角色 
    class Character {
    public:
        WeaponStrategy* pWeapon;
        
        void setWeapon(WeaponStrategy* pWeapon) {
            this->pWeapon = pWeapon;
        }
    
        void ThrowWeapon() {
            this->pWeapon->UseWeapon();
        }
    };
    
    Character* character = new Character;
    WeaponStrategy* knife = new Knife;
    WeaponStrategy* ak47 = new AK47;
    
    //用匕首当作武器
    character->setWeapon(knife);
    character->ThrowWeapon();
    
    character->setWeapon(ak47);
    character->ThrowWeapon();
    
    delete ak47;
    delete knife;
    delete character;
    ```

3. 状态模式
- 作用：
    1. 每个人、事物在不同的状态下会有不同表现（动作），而一个状态又会在不同的表现下转移到下一个不同的状态（State）。
    2. 主要解决：
        - 当状态数目不是很多的时候，Switch/Case 可能可以搞定。但是当状态数目很多的时候（实际系统中也正是如此），维护一大组的 Switch/Case 语句将是一件异常困难并且容易出错的事情。
        - 状态逻辑和动作实现没有分离。在很多的系统实现中，动作的实现代码直接写在状态的逻辑当中。这带来的后果就是系统的扩展性和维护得不到保证。
- 其他：
    1. 优点：
        - 封装了转换规则。
        - 枚举可能的状态，在枚举状态之前需要确定状态种类。
        - 将所有与某个状态有关的行为放到一个类中，并且可以方便地增加新的状态，只需要改变对象状态即可改变对象的行为。
        - 允许状态转换逻辑与状态对象合成一体，而不是某一个巨大的条件语句块。
        - 可以让多个环境对象共享一个状态对象，从而减少系统中对象的个数。
- 代码演示：
    ```
    class State
    {
    public:
    	// 扣除50积分
    	virtual void deductMeney() = 0;
    
    	// 是否中奖
    	virtual bool raffle() = 0;
    
    	// 发放奖品
    	virtual void dispensePrize() = 0;
    };
    
    // 能抽奖状态
    class CanRaffleState : public State
    {
    public:
    	CanRaffleState(RaffleActivity* activity)
    	{
        	srand(time(NULL));
        	this->activity = activity;
    	}
    	
    	// 已经扣除积分了，不能再扣了
    	void deductMeney() override
    	{}
    	
    	// 可以抽奖，根据抽奖情况改成新的状态
    	bool raffle() override
    	{
    	    cout << "正在抽奖" << endl;
        	int result = rand() % 10;
        	if(0 == result)
        	{
        		// 将activity的状态设置为发放奖品的状态
        		activity->setState(activity->getDispenseState());
        		return true;
        	}
        	else
        	{
        		cout << "很遗憾没有抽中奖品" << endl;
        		// 将activity的状态设置为不能抽奖的状态
        		activity->setState(activity->getNoRaffleState());
        		return false;
        	}
    	}
    	
    	void dispensePrize() override
    	{}
    private:
    	RaffleActivity* activity;
    };
    
    // 奖品发送完毕状态
    class DispenseOutState : public State
    {
    public:
    	DispenseOutState(RaffleActivity* activity);
    	{
    	    this->activity = activity;
    	}
    	
    	void deductMeney() override
    	{}
    	
    	bool raffle() override;
    	{}
    	
    	// 发放奖品
    	void dispensePrize() override
    	{}
    private:
    	RaffleActivity* activity;
    };
    
    // 发放奖品的状态
    class DispenseState :public State
    {
    public:
    	DispenseState(RaffleActivity* activity)
    	{
    	    this->activity = activity;
    	}
    	
    	void deductMeney() override
    	{}
    	
    	bool raffle() override
    	{}
    	
    	// 发放奖品
    	void dispensePrize() override
    	{
    	    if(activity->getCount() > 0)
        	{
        		cout << "送出奖品" << endl;
        		activity->setState(activity->getNoRaffleState());
        	}
        	else
        	{
        		cout << "很遗憾,奖品发完了" << endl;
        		// 奖品已经发完，后面的人就不能抽奖了
        		activity->setState(activity->getDispenseOutState());
        	}
    	}
    private:
    	RaffleActivity* activity;
    };
    
    // 不能抽奖状态
    class NoRaffleState :public State
    {
    public:
    	NoRaffleState(RaffleActivity* activity)
    	{
    	    this->activity = activity;
    	}
    	
    	//在不能抽奖状态是可以扣积分的，扣除积分后设置成可以抽奖状态
    	void deductMeney() override
    	{
    	    activity->setState(activity->getCanRaffleState());
    	}
    	
    	bool raffle() override
    	{}
    	
    	void dispensePrize() override
    	{}
    private:
    	//初始化时传入活动，扣除积分后改变其状态
    	RaffleActivity* activity;
    };
    
    class RaffleActivity {
    public:
    	State* getState() const{
    		return state;
    	}
    	
    	void setState(State* const state) {
    		this->state = state;
    	}
    
    	int getCount() {
    		return count--;
    	}
    
    	void setCount(const int count) {
    		this->count = count;
    	}
    
    	State* getNoRaffleState() const {
    		return noRaffleState;
    	}
    
    	void setNoRaffleState(State* const noRaffleState) {
    		this->noRaffleState = noRaffleState;
    	}
    
    	State* getCanRaffleState() const {
    		return canRaffleState;
    	}
    
    	void setCanRaffleState(State* const canRaffleState) {
    		this->canRaffleState = canRaffleState;
    	}
    
    	State* getDispenseState() const {
    		return dispenseState;
    	}
    
    	void setDispenseState(State* const dispenseState) {
    		this->dispenseState = dispenseState;
    	}
    
    	State* getDispenseOutState() const {
    		return dispenseOutState;
    	}
    
    	void setDispenseOutState(State* const dispenseOutState) {
    		this->dispenseOutState = dispenseOutState;
    	}
    
    	RaffleActivity(int count) {
    		//初始化当前状态为 不能抽奖状态
    		this->state = getNoRaffleState();
    		//初始化奖品数量
    		this->count = count;
    	}
    
    	//扣分，调用当前状态的deductMoney
    	void deductMoney() {
    		state->deductMeney();
    	}
    
    	//抽奖
    	void raffle() {
    		//如果抽中奖了，则领奖品
    		if (state->raffle()) {
    			state->dispensePrize();
    		}
    	}
    
    private:
    	//state表示活动当前的状态
    	State* state = nullptr;
    	//奖品数量
    	int count = 0;
    	//四个属性 表示四种状态
    	State* noRaffleState = new NoRaffleState(this);
    	State* canRaffleState = new CanRaffleState(this);
    	State* dispenseState = new DispenseState(this);
    	State* dispenseOutState = new DispenseOutState(this);
    };
    
	RaffleActivity* activity = new RaffleActivity(1);
	for(int i=0; i<50; i++)
	{
		cout << "第" << i << "次抽奖" << endl;
		activity->deductMoney();
		activity->raffle();
		cout << endl;
	}
    ```

4. 观察者模式
- 作用：
    1. Observer 模式要解决的问题为：建立一个一（Subject）对多（Observer）的依赖关系，并且做到当“一”变化的时候，依赖这个“一”的多也能够同步改变。最常见的一个例子就是：对同一组数据进行统计分析时候，我们希望能够提供多种形式的表示（例如以表格进行统计显示、柱状图统计显示、百分比统计显示等）。指多个对象间存在一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都得到通知并被自动更新。
    2. Subject 提供依赖于它的观察者 Observer 的注册（registerObserver）和注销（remove）操作，并且提供了使得依赖于它的所有观察者同步的操作（notifyObserver），观察者 Observer 则提供一个 Update 操作，注意这里的 Observer 的 Update 操作并不在 Observer 改变了 Subject 目标状态的时候就对自己进行更新，这个更新操作要延迟到 Subject 对象发出 Notify 通知所有 Observer 进行修改（调用 Update）。
- 其他：
    1. 优点：
        - 观察者和被观察者是抽象耦合的。
        - 建立一套触发机制。
    2. 缺点：
        - 如果在观察者和观察目标之间有循环依赖的话，观察目标会触发它们之间进行循环调用，可能导致系统崩溃。
        - 观察者模式没有相应的机制让观察者知道所观察的目标对象是怎么发生变化的，而仅仅只是知道观察目标发生了变化。
- 代码演示：
    ```
    // 抽象的英雄 抽象的观察者  Observer
    class AbstractHero {
    public:
        virtual void Update() = 0;
    };
    
    //具体的英雄 具体的观察者 
    class HeroA : public AbstractHero {
    public:
        HeroA() {
            cout << "英雄A正在鲁BOSS" << endl;
        }
        virtual void Update() {
            cout << "英雄A停止撸，待机状态" << endl;
        }
    };
    class HeroB : public AbstractHero {
    public:
        HeroB() {
            cout << "英雄B正在鲁BOSS" << endl;
        }
        virtual void Update() {
            cout << "英雄B停止撸，待机状态" << endl;
        }
    };
    class HeroC : public AbstractHero {
    public:
        HeroC() {
            cout << "英雄C正在鲁BOSS" << endl;
        }
        virtual void Update() {
            cout << "英雄C停止撸，待机状态" << endl;
        }
    };
    class HeroD : public AbstractHero {
    public:
        HeroD() {
            cout << "英雄D正在鲁BOSS" << endl;
        }
        virtual void Update() {
            cout << "英雄D停止撸，待机状态" << endl;
        }
    };
    class HeroE : public AbstractHero {
    public:
        HeroE() {
            cout << "英雄E正在鲁BOSS" << endl;
        }
        virtual void Update() {
            cout << "英雄E停止撸，待机状态" << endl;
        }
    };
    
    // 定义抽象的观察目标  Subject
    class AbstractBoss {
    public:
        // 添加观察者
        virtual void addHero(AbstractHero* hero) = 0;
        // 删除观察者
        virtual void deleteHero(AbstractHero* hero) = 0;
        // 通知所有观察者
        virtual void notify() = 0;
    };
    
    // 相对于 concreteSubject
    class BOSSA : public AbstractBoss {
    public:
        // 添加观察者
        virtual void addHero(AbstractHero* hero) 
        {
            pHeroList.push_back(hero);
        }
        // 删除观察者
        virtual void deleteHero(AbstractHero* hero)
        {
            pHeroList.remove(hero);
        }
        // 通知所有观察者
        virtual void notify() 
        {
            for(auto it = pHeroList.begin(); it != pHeroList.end(); it++) {
                (*it)->Update();
            }
        }
    private:
        list<AbstractHero*> pHeroList;
    };
    
    // 创建观察者
    AbstractHero* heroA = new HeroA;
    AbstractHero* heroB = new HeroB;
    AbstractHero* heroC = new HeroC;
    AbstractHero* heroD = new HeroD;
    AbstractHero* heroE = new HeroE;
    
    // 创建观察目标
    AbstractBoss* bossA = new BOSSA;
    bossA->addHero(heroA);
    bossA->addHero(heroB);
    bossA->addHero(heroC);
    bossA->addHero(heroD);
    bossA->addHero(heroE);
    
    cout << "heroC阵亡" << endl;
    bossA->deleteHero(heroC);
    
    cout << "Boss死了，通知其他英雄停止攻击。。。" << endl;
    bossA->notify();
    ```

5. 备忘录模式
- 作用：
    1. 我们对一些关键性的操作肯定需要提供诸如撤销（Undo）的操作。
    2. Memento 模式的关键就是要在不破坏封装行的前提下，捕获并保存一个类的内部状态，这样就可以利用该保存的状态实施恢复操作。
- 其他：
    1. 优点：
        - 给用户提供了一种可以恢复状态的机制，可以使用户能够比较方便地回到某个历史的状态。
        - 实现了信息的封装，使得用户不需要关心状态的保存细节。
    2. 缺点：
        - 消耗资源。如果类的成员变量过多，势必会占用比较大的资源，而且每一次保存都会消耗一定的内存。
- 代码演示：
    ```
    class Originator        // Originator 原始的状态
    {
    public:
    	std::string getState() const
    	{
    		return state;
    	}
    
    	void setState(const std::string& state)
    	{
    		this->state = state;
    	}
    
    	// 保存一个状态对象 Memento，即用当前状态的值去创建一个 Memento 然后将其返回
    	Memento SaveStateMemento()
    	{
    		return Memento(state);
    	}
    
    	// 通过备忘录对象，恢复状态
    	void getStateFromMemento(Memento memento)
    	{
    		state = memento.getState();
    	}
    private:
    	std::string state;
    };
    
    class Memento           // 包含了要被恢复的对象的状态
    {
    public:
    	explicit Memento(const std::string& state)
    		: state(state)
    	{
    	}
    
    	std::string getState() const
    	{
    		return state;
    	}
    private:
    	std::string state;
    };
    
    class Caretaker         // 用于保存所有的memento的一个管理类
    {
    public:
    	void add(Memento memento)
    	{
    		mementoList.push_back(memento);
    	}
    
    	//获取到第 index 个 Originator 的备忘录对象（即备忘录状态）
    	Memento get(int index)
    	{
    		return mementoList[index];
    	}
    private:
    	// 在mementoList中会有很多备忘录对象
    	std::vector<Memento> mementoList;
    };
    
	Originator originator;
	Caretaker caretaker;
	originator.setState("状态1，攻击力为100");
    
	//保存当前状态
	caretaker.add(originator.SaveStateMemento());
    
	//受到debuff，攻击力下降
	originator.setState("状态2，攻击力为80");
	//保存状态
	caretaker.add(originator.SaveStateMemento());
    
	originator.getStateFromMemento(caretaker.get(0));
	cout << "恢复到状态1后的状态：" << originator.getState();
	return 0;
    ```

6. 中介者模式
- 作用：
    1. 在面向对象系统的设计和开发过程中，对象之间的交互和通信是最为常见的情况，因为对象间的交互本身就是一种通信。在系统比较小的时候，可能对象间的通信不是很多、对象也比较少，我们可以直接硬编码到各个对象的方法中。但是当系统规模变大，对象的量变引起系统复杂度的急剧增加，对象间的通信也变得越来越复杂，这时候我们就要提供一个专门处理对象间交互和通信的类，这个中介者就是 Mediator 模式。所以 Mediator 模式的实现关键就是将对象 Colleague 之间的通信封装到一个类中单独处理。
- 其他：
    1. 何时使用：多个类相互耦合，形成了网状结构，需要将上述网状结构分离为星型结构的时候。
    2. 中介者承担了较多的责任，一旦中介者出现问题，整个系统都会收到影响。
    3. 其他说明：
        - Meadiator 是抽象中介者，定义了同事对象到中介者对象的接口。
        - Colleague 是抽象同事类。
        - ConcreteMediator 是具体的中介者对象，实现抽象方法，他需要知道所有的具体的同事类，用一个集合来管理它们，并接收某个同事的消息，完成相应的任务。
        - ConcreteColleague 是具体的同事类，会有很多，每个同事只知道自己的行为，而不了解其他同事的行为，它们都依赖中介者对象。
- 代码演示：
    ```
    class Colleague
    {
    public:
        Colleague(Mediator* mediator)
        {
            this->mediator = mediator;
	        this->mediator->add(this);
        }
    
    	Mediator* getMediator()
    	{
    	    return mediator;
    	}
    	
    	void setMediator(Mediator* const mediator)
    	{
    	    this->mediator = mediator;
    	}
    	
    	virtual void Notify(std::string message) = 0;
    private:
    	Mediator* mediator;
    };
    
    class ConcreteColleague1 : public Colleague
    {
    public:
    	ConcreteColleague1(Mediator* mediator) : Colleague(mediator)
    	{
    	}
    
    	void send(std::string message)
    	{
    		getMediator()->send(message, this);
    	}
    
    	void Notify(std::string message)
    	{
    		std::cout << "同事1收到消息：" + message<<std::endl;
    	}
    };
    
    class ConcreteColleague2 : public Colleague
    {
    public:
    	ConcreteColleague2(Mediator* mediator) : Colleague(mediator)
    	{
    	}
    
    	void send(std::string message)
    	{
    		getMediator()->send(message, this);
    	}
    
    	void Notify(std::string message)
    	{
    		std::cout << "同事2收到消息：" + message << std::endl;
    	}
    };
    
    class Mediator	
    {
    public:
        virtual void add(Colleague* colleague) = 0;
    	virtual void send(std::string message, Colleague* colleague) = 0;
    };
    
    class ConcreteMediator : public Mediator
    {
    public:
    	void add(Colleague* colleague)
    	{
    		colleaguesList.push_back(colleague);
    	}
    
    	void send(std::string message, Colleague* colleague)
    	{
    		for(auto value : colleaguesList)
    		{
    			if(value != colleague)
    			{
    				value->Notify(message);
    			}
    		}
    	}
    private:
    	std::vector<Colleague*> colleaguesList;
    };
    
	Mediator* mediator = new ConcreteMediator();
	ConcreteColleague1* colleague1 = new ConcreteColleague1(mediator);
	ConcreteColleague2* colleague2 = new ConcreteColleague2(mediator);
    
	colleague1->send("早上好啊！");
	colleague2->send("早安！");
    ```

7. 命令模式
- 作用：
    1. 请求以命令的形式包裹在对象中，并传给调用对象。调用对象寻找可以处理该命令的合适的对象，并把该命令传给相应的对象，该对象执行命令。
    2. Command 模式的最终目的就是将一个请求封装成一个对象，从而使您可以用不同的请求对客户进行参数化。
- 其他：
    1. 其他说明：
        - Invoker 是调用者角色。
        - Command 是命令角色，需要执行的所有命令都在这里。
        - Receiver 是接受者角色，知道如何实施和执行一个请求相关的操作。
        - ConcreteCommand 将接受者对象与一个动作绑定，调用接受者相应的操作。
- 代码演示：
    ```
    class HandleClientProtocal      // 协议处理类（请求类）
    {
    public:
    	// 处理增加金币
    	void AddMoney()
    	{
    		cout << "给玩家增加金币" << endl;
    	}
    	// 处理增加钻石
    	void AddDiamond()
    	{
    		cout << "给玩家增加钻石" << endl;
    	}
    	// 处理玩家装备
    	void AddEquipment()
    	{
    		cout << "给玩家穿装备" << endl;
    	}
    	// 玩家升级
    	void AddLevel()
    	{
    		cout << "给玩家升级" << endl;
    	}
    };
    
    class AbstractCommand    // 命令接口
    {
    public:
    	// 相当于 execute()
    	virtual void handle() = 0;
    };


    // 下面是把每个功能都封装为一个请求对象
    class AddMoneyCommand : public AbstractCommand      //处理增加金币请求
    {
    public:
    	AddMoneyCommand(HandleClientProtocal* protocal)
    	{
    		this->pProtocol = protocal;
    	}
    	virtual void handle()
    	{
    		this->pProtocol->AddMoney();
    	}
    public:
    	HandleClientProtocal* pProtocol;
    };
    
    class AddDiamondCommand : public AbstractCommand    // 处理增加钻石请求
    {
    public:
    	AddDiamondCommand(HandleClientProtocal* protocal)
    	{
    		this->pProtocol = protocal;
    	}
    	virtual void handle()
    	{
    		this->pProtocol->AddDiamond();
    	}
    public:
    	HandleClientProtocal* pProtocol;
    };
    
    class AddEquipmentCommand : public AbstractCommand    // 处理玩家装备请求
    {
    public:
    	AddEquipmentCommand(HandleClientProtocal* protocal)
    	{
    		this->pProtocol = protocal;
    	}
    	virtual void handle()
    	{
    		this->pProtocol->AddEquipment();
    	}
    public:
    	HandleClientProtocal* pProtocol;
    };
    
    class AddLevelCommand : public AbstractCommand    // 处理玩家升级请求
    {
    public:
    	AddLevelCommand(HandleClientProtocal* protocal)
    	{
    		this->pProtocol = protocal;
    	}
    	virtual void handle()
    	{
    		this->pProtocol->AddLevel();
    	}
    public:
    	HandleClientProtocal* pProtocol;
    };
    
    class Server    // 服务器程序（命令调用类）
    {
    public:
    	//将请求对象放入处理队列
    	void addRequest(AbstractCommand* command)
    	{
    		mCommands.push(command);
    	}
    
    	//启动处理程序
    	void startHandle()
    	{
    		while (!mCommands.empty())
    		{
    			Sleep(2000);
    			AbstractCommand* command = mCommands.front();
    			command->handle();
    			mCommands.pop();
    		}
    	}
    private:
    	queue<AbstractCommand*> mCommands;
    };
    
    HandleClientProtocal* protocal = new HandleClientProtocal;
    //客户端增加金币的请求
    AbstractCommand* addmoney = new AddMoneyCommand(protocal);
    //客户端增加钻石的请求
    AbstractCommand* adddiamond = new AddDiamondCommand(protocal);
    //客户端穿装备的请求
    AbstractCommand* addequipment = new AddEquipmentCommand(protocal);
    //客户端升级请求
    AbstractCommand* addlevel = new AddLevelCommand(protocal);
    
    //将客户端的请求加入到请求队列中
    Server* server = new Server;
    server->addRequest(addmoney);
    server->addRequest(adddiamond);
    server->addRequest(addequipment);
    server->addRequest(addlevel);
    
    //服务器开始处理请求
    server->startHandle();
    ```

8. 访问者模式
- 作用：
    1. 访问者模式（VisitorPattern），封装一些作用于某种数据结构的各元素的操作，它可以在不改变数据结构的前提下定义作用于这些元素的新的操作。
    2. 主要将数据结构与数据操作分离，解决数据结构和操作耦合性问题。
    3. 访问者模式的基本工作原理是：在被访问的类里面加一个对外提供接待访问者的接口。
    4. 访问者模式主要应用场景是：需要对一个对象结构中的对象进行很多不同操作（这些操作彼此没有关联），同时需要避免让这些操作“污染”这些对象的类，可以选用访问者模式解决。
- 其他：
    1. 优点：
        - 访问者模式符合单一职责原则、让程序具有优秀的扩展性、灵活性非常高。
        - 访问者模式可以对功能进行统一，可以做报表、UI、拦截器与过滤器，适用于数据结构相对稳定的系统。
    2. 缺点：
        - 具体元素对访问者公布细节，也就是说访问者关注了其他类的内部细节，这是迪米特法则所不建议的，这样造成了具体元素变更比较困难。
        - 违背了依赖倒转原则。访问者依赖的是具体元素，而不是抽象元素。
        - 因此，如果一个系统有比较稳定的数据结构，又有经常变化的功能需求，那么访问者模式就是比较合适的。
- 代码演示：
    ```
    class Action
    {
    public:
    	virtual void getManResult(Man* man) = 0;
    	virtual void getWomanResult(Woman* woman) = 0;
    };
    
    class Success : public Action
    {
    public:
    	void getManResult(Man* man) override
    	{
    		cout << "男人的给的评价该歌手是很成功" << endl;
    	}
    
    	void getWomanResult(Woman* woman) override
    	{
    		cout << "女人的给的评价该歌手是很成功" << endl;
    	}
    };
    
    class Fail : public Action
    {
    public:
    	void getManResult(Man* man) override
    	{
    		cout << "男人的给的评价该歌手是失败" << endl;
    	}
    
    	void getWomanResult(Woman* woman) override
    	{
    		cout << "女人的给的评价该歌手是失败" << endl;
    	}
    };
    
    class Person
    {
    public:
    	// 提供一个方法让访问者可以访问
    	virtual void accept(Action* action) = 0;
    };
    
    // 这里用到了双分派，即首先在客户端程序中，将具体的状态作为参数传递进 Man 中（第一次分派），
    // 然后在 Man 中调用作为参数的“具体方法” getManResult()，同时将自己作为参数传入完成第二次分派。
    class Man : public Person
    {
    public:
    	void accept(Action* action) override
    	{
    		action->getManResult(this);
    	}
    };
    
    class Woman : public Person
    {
    public:
    	void accept(Action* action) override
    	{
    		action->getWomanResult(this);
    	}
    };
    
    // 数据结构，管理很多人（Man、Woman）用来装符合某一个评价的所有人
    class ObjectStructure
    {
    public:
    	//添加到list
    	void attach(Person* p)
    	{
    		persons.push_back(p);
    	}
    	
    	//移除
    	void detach(Person* p)
    	{
    		persons.remove(p);
    		delete p;
    	}
    
    	//显示测评情况
    	void display(Action* action)
    	{
    		for(auto value : persons)
    		{
    			value->accept(action);
    		}
    	}
    private:
    	list<Person*> persons;
    };
    
	//创建 ObjectStructure（可以创建很多个不同的 ObjectStructure 来代表不同评价，然后把同样评价的人放到一个 ObjectStructure 中）
	ObjectStructure* objectStructure = new ObjectStructure;
	objectStructure->attach(new Man);
	objectStructure->attach(new Woman);
    
	//假如歌手获得成功
	Success* success = new Success;
	objectStructure->display(success);
	
	//假如歌手失败了
	Fail* fail = new Fail;
	objectStructure->display(fail);
    ```

9. 责任链模式
- 作用：
    1. 责任链模式（Chain of Responsibility Pattern）为请求创建了一个接收者对象的链。这种模式给予请求的类型，对请求的发送者和接收者进行解耦。这种类型的设计模式属于行为型模式。
    2. 在这种模式中，通常每个接收者都包含对另一个接收者的引用。如果一个对象不能处理该请求，那么它会把相同的请求传给下一个接收者，依此类推。
    3. 主要解决：职责链上的处理者负责处理请求，客户只需要将请求发送到职责链上即可，无须关心请求的处理细节和请求的传递，所以职责链将请求的发送者和请求的处理者解耦了。
    4. 何时使用：在处理消息的时候以过滤很多道。
- 其他：
    1. 将请求和处理分开，实现解耦，提高系统的灵活性。
    2. 简化了对象，使对象不需要知道链的结构。
    3. 调试不方便。采用了类似递归的方式，调试时逻辑可能比较复杂。
    4. 最佳应用场景：有多个对象可以处理同一个请求时，比如：多级请求、请假/加薪等审批流程、Java Web 中 Tomcat 对 Encoding 的处理、拦截器。
- 代码演示：
    ```
    class PurchaseRequest       //请求类
    {
    public:
    	int getType() const {
    		return type;
    	}
    
    	float getPrice() const {
    		return price;
    	}
    
    	int getId() const {
    		return id;
    	}
    
    	PurchaseRequest(const int type, const float price, const int id)
    		: type(type),
    		  price(price),
    		  id(id)
    	{}
    private:
    	int type;               //请求类型
    	float price = 0.f;      //价格
    	int id = 0;
    };
    
    class Approver
    {
    public:
    	void setApprover(Approver* const approver) {
    		this->approver = approver;
    	}
    
    	explicit Approver(const string& name)
    		: name(name)
    	{}
    
    	//处理审批请求的方法，得到一个请求，处理是子类完成的，因此该方法写成纯虚方法
    	virtual void processRequest(PurchaseRequest* purchaseRequest) = 0;
    protected:
    	Approver* approver;     // 下一个处理者
    	string name;            // 名字
    };
    
    
    class DepartmentApprover : public Approver      // 教学主任
    {
    public:
    	explicit DepartmentApprover(const string& name)
    		: Approver(name)
    	{}
    
    	void processRequest(PurchaseRequest* purchaseRequest) override
    	{
    		if(purchaseRequest->getPrice() <= 5000)
    		{
    			cout << "请求编号id=" << purchaseRequest->getId() << "被" << this->name << "处理" << endl;
    		}
    		else
    		{
    			approver->processRequest(purchaseRequest);
    		}
    	}
    };
    
    class CollegeApprover :public Approver          // 院长
    {
    public:
    	explicit CollegeApprover(const string& name)
    		: Approver(name)
    	{}
    
    	void processRequest(PurchaseRequest* purchaseRequest) override
    	{
    		if(purchaseRequest->getPrice() > 5000 && purchaseRequest->getPrice() <= 10000)
    		{
    			cout << "请求编号id=" << purchaseRequest->getId() << "被" << this->name << "处理" << endl;
    		}
    		else
    		{
    			approver->processRequest(purchaseRequest);
    		}
    	}
    };
    
    class ViceSchoolMasterApprover : public Approver     // 副校长
    {
    public:
    	explicit ViceSchoolMasterApprover(const string& name)
    		: Approver(name)
    	{}
    
    	void processRequest(PurchaseRequest* purchaseRequest) override
    	{
    		if(purchaseRequest->getPrice() > 10000 && purchaseRequest->getPrice() <= 30000)
    		{
    			cout << "请求编号id=" << purchaseRequest->getId() << "被" << this->name << "处理" << endl;
    		}
    		else
    		{
    			approver->processRequest(purchaseRequest);
    		}
    	}
    };
    
    class SchoolMasterApprover : public Approver         // 校长
    {
    public:
    	explicit SchoolMasterApprover(const string& name)
    		: Approver(name)
    	{
    	}
    
    	void processRequest(PurchaseRequest* purchaseRequest) override
    	{
    		if (purchaseRequest->getPrice() > 30000)
    		{
    			cout << "请求编号id=" << purchaseRequest->getId() << "被" << this->name << "处理" << endl;
    		}
    		else
    		{
    			approver->processRequest(purchaseRequest);
    		}
    	}
    };
    
    //创建一个请求
    PurchaseRequest* purchaseRequest = new PurchaseRequest(1, 1000, 1);
    
    //创建相关的审批人
    DepartmentApprover* department = new DepartmentApprover("张主任");
    CollegeApprover* college = new CollegeApprover("李院长");
    ViceSchoolMasterApprover* viceSchoolMaster = new ViceSchoolMasterApprover("王副校长");
    SchoolMasterApprover* schoolMaster = new SchoolMasterApprover("佟校长");
    
    // 需要将各个审批级别的下一个人设置好(处理人构成一个环装就可以从任何一个人开始处理了)
    // 否则必须从第一个处理人开始
    department->setApprover(college);
    college->setApprover(viceSchoolMaster);
    viceSchoolMaster->setApprover(schoolMaster);
    schoolMaster->setApprover(department);
    //开始处理请求
    viceSchoolMaster->processRequest(purchaseRequest);
    ```

10. 迭代器模式
- 作用：
    1. 迭代器模式，提供一种遍历集合元素的统一接口，用一致的方法遍历集合元素，不需要知道集合对象的底层表示，即：不暴露其内部的结构。而且不管这些对象是什么都需要遍历的时候，就应该选择使用迭代器模式。
- 其他：
    1. 提供一个统一的方法遍历对象，客户不用再考虑聚合的类型，使用一种方法就可以遍历对象了。
    2. 隐藏了聚合的内部结构，客户端要遍历聚合的时候只能取到迭代器，而不会知道聚合的具体组成。
    3. 提供了一种设计思想，就是一个类应该只有一个引起变化的原因（叫做单一责任原则）。在聚合类中，我们把迭代器分开，就是要把管理对象集合和遍历对象集合的责任分开，这样一来集合改变的话，只影响到聚合对象。而如果遍历方式改变的话，只影响到了迭代器。
    4. 当要展示一组相似对象，或者遍历一组相同对象时使用, 适合使用迭代器模式。
- 代码演示：
    ```
    // 迭代抽象类，用于定义得到开始对象、得到下一个对象、判断是否到结尾、当前对象等抽象方法，统一接口
    class Iterator
    {
    public:
        Iterator() {};
        virtual ~Iterator() {};
        virtual string First() = 0;
        virtual string Next() = 0;
        virtual string CurrentItem() = 0;
        virtual bool IsDone() = 0;
    };
    
    class Aggregate         // 聚集抽象类
    {
    public:
        virtual int Count() = 0;
        virtual void Push(const string& strValue) = 0;
        virtual string Pop(const int nIndex) = 0;
        virtual Iterator* CreateIterator() = 0;
    };
    
    //具体迭代器类，继承Iterator 实现开始、下一个、是否结尾、当前对象等方法
    class ConcreteIterator : public Iterator
    {
    public:
        ConcreteIterator(Aggregate* pAggregate) : m_nCurrent(0), Iterator()
        {
            m_Aggregate = pAggregate;
        }
        string First()
        {
            return m_Aggregate->Pop(0);
        }
        string Next()
        {
            string strRet;
            m_nCurrent++;
            if(m_nCurrent < m_Aggregate->Count())
            {
                strRet = m_Aggregate->Pop(m_nCurrent);
            }
            return strRet;
        }
        string CurrentItem()
        {
            return m_Aggregate->Pop(m_nCurrent);
        }
        bool IsDone()
        {
            return ((m_nCurrent >= m_Aggregate->Count()) ? true : false);
        }
    private:
        Aggregate* m_Aggregate;
        int m_nCurrent;
    };
    
    //具体聚集类 继承
    class ConcreteAggregate : public Aggregate
    {
    public:
        ConcreteAggregate() : m_pIterator(NULL)
        {
            m_vecItems.clear();
        }
        ~ConcreteAggregate()
        {
            if(NULL != m_pIterator)
            {
                delete m_pIterator;
                m_pIterator = NULL;
            }
        }
        Iterator* CreateIterator()
        {
            if(NULL == m_pIterator)
            {
                m_pIterator = new ConcreteIterator(this);
            }
            return m_pIterator;
        }
        int Count()
        {
            return m_vecItems.size();
        }
        void Push(const string& strValue)
        {
            m_vecItems.push_back(strValue);
        }
        string Pop(const int nIndex)
        {
            string strRet;
            if(nIndex < Count())
            {
                strRet = m_vecItems[nIndex];
            }
            return strRet;
        }
    private:
        vector<string> m_vecItems;
        Iterator* m_pIterator;
    };
    
    ConcreteAggregate* pName = NULL;
    pName = new ConcreteAggregate();
    if(NULL != pName)
    {
        pName->Push("hello");
        pName->Push("word");
        pName->Push("cxue");
    }
    Iterator* iter = NULL;
    iter = pName->CreateIterator();
    if(NULL != iter)
    {
        string strItem = iter->First();
        while (!iter->IsDone())
        {
            cout << iter->CurrentItem() << " is ok" << endl;
            iter->Next();
        }
    }
    ```

11. 解释器模式
- 作用：
    1. 在编译原理中，一个算术表达式通过词法分析器形成词法单元，而后这些词法单元再通过语法分析器构建语法分析树，最终形成一颗抽象的语法分析树。这里的词法分析器和语法分析器都可以看做是解释器。
    2. 解释器模式（Interpreter Pattern）：是指给定一个语言（表达式），定义它的文法的一种表示，并定义一个解释器，使用该解释器来解释语言中的句子（表达式）。
    3. 应用场景：
        - 应用可以将一个需要解释执行的语言中的句子表示为一个抽象语法树。
        - 一些重复出现的问题可以用一种简单的语言来表达。
        - 一个简单语法需要解释的场景。
        - 这样的例子还有，比如编译器、运算表达式计算、正则表达式、机器人等。
- 其他：
    1. 优点：
        - 可扩展性比较好，灵活。
        - 增加了新的解释表达式的方式。
        - 易于实现简单文法。
    2. 缺点：
        - 可利用场景比较少。
        - 对于复杂的文法比较难维护。
        - 解释器模式会引起类膨胀。
        - 解释器模式采用递归调用方法。
- 代码演示：
    ```
    class Context
    {
    public:
        Context(int num) {
            m_num = num;
        }
    
        void setNum(int num) {
            m_num = num;
        }
    
        int getNum() {
            return m_num;
        }
    
        void setRes(int res) {
            m_res = res;
        }
    
        int getRes() {
            return m_res;
        }
    
    private:
        int m_num;
        int m_res;
    };
    
    class Expression
    {
    public:
        virtual void interpreter(Context *context) = 0;
    };
    
    class PlusExpression : public Expression
    {
    public:
        virtual void interpreter(Context *context)
        {
            int num = context->getNum();
            num++;
            context->setNum(num);
            context->setRes(num);
        }
    };
    
    class MinusExpression : public Expression
    {
    public:
        virtual void interpreter(Context *context)
        {
            int num = context->getNum();
            num--;
            context->setNum(num);
            context->setRes(num);
        }
    };
    
    Context *pcxt = new Context(10);
    
    Expression *e1 = new PlusExpression();
    e1->interpreter(pcxt);
    cout << "PlusExpression: " << pcxt->getRes() << endl;
    
    Expression *e2 = new MinusExpression();
    e2->interpreter(pcxt);
    cout << "MinusExpression: " << pcxt->getRes() << endl;
    
    delete e1;
    delete e2;
    delete pcxt;
    ```
