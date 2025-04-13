# 第11章　系统

“复杂要人命。它消磨开发者的生命，让产品难以规划、构建和测试。”

## 11.1　如何建造一个城市

你能自己掌管一切细节吗？大概不行。即便是管理一个既存的城市，也是靠单人能力无法做到的。不过，城市还是在运转（多数时候）。这是因为每个城市都有各种组织管理不同的部分，如供水系统、供电系统、交通、执法、立法，诸如此类。有些人负责全局，有些人负责细节。

城市能运转，还因为它演化出恰当的抽象等级和模块，好让个人和其所管理的“组件”即便在不了解全局时也能有效地运转。

尽管软件团队往往也是这样组织起来的，但他们所致力的工作却常常没有同样的关注面切分及抽象层级，而整洁的代码可以帮助我们在较低的抽象层级上达成这一目标。本章将讨论如何在较高的抽象层级（系统层级）上保持整洁。

## 11.2　将系统的构造与使用分开

首先，构造与使用是非常不一样的过程。当我走笔至此，投目窗外的芝加哥，看到有一家酒店正在建设。今天，那只是个框架结构，起重机和升降机布设在外面。忙碌的人们身穿工作服，头戴安全帽。大概一年之后，酒店就将建成。起重机和升降机都会消失无踪。建筑物变得整洁，覆盖着玻璃幕墙和漂亮的漆色。在其中工作和住宿的人，会看到完全不同的景象。

软件系统应将起始过程和起始过程之后的运行时逻辑分离开，在起始过程中构建应用对象，也会存在互相缠结的依赖关系。

每个应用程序都该留意起始过程。那也是本章中我们首先要考虑的问题。将关注的方面分离开，是软件技能中最古老也最重要的设计技能。

不幸的是，多数应用程序都没有做分离处理。起始过程代码很特殊，被混杂到运行时逻辑中，下例就是典型的这种情形：

```
public Service getService() {
  if (service == null)
    service = new MyServiceImpl(...);  // Good enough default for most cases?
  return service;
}
```

这就是所谓延迟初始化/赋值，它有一些好处，即在真正用到对象之前，无须操心这种架空构造，起始时间也会更短，而且还能保证永远不会返回`null`值。

然而，我们也得到了`MyServiceImpl`及其构造器所需一切（我省略了那些代码）的硬编码依赖。不搞好这些被依赖项，程序就无法编译，即便在运行时永不使用这种类型的对象！

测试也会是问题。如果`MyServiceImpl`是一个重型对象，则我们必须确保在单元测试调用该方法之前，就给服务指派恰当的测试替身（TEST DOUBLE）或仿制对象（MOCK OBJECT）。由于构造逻辑与运行过程相混杂，我们必须测试所有的执行路径（例如，`null`值测试及其代码块）。有了这些权责，说明方法做了不止一件事，这样就略微违反了单一权责原则。

最糟糕的大概是，我们不知道`MyServiceImpl`在所有情形中是否都是正确的对象。我在代码注释中做了暗示。为什么该方法所属类必须知道全局情景？我们是否真能知道在这里要用到的正确对象？是否真有可能存在一种放之四海而皆准的类型？

当然，仅出现一次的延迟初始化不算是严重问题。不过，在应用程序中往往有许多种类似的情况出现。于是，全局设置策略（如果有的话）在应用程序中四散分布，缺乏模块组织性，通常也会有许多重复代码。

如果我们勤于打造有着良好格式并且强固的系统，就不该让这类就手小技巧破坏模块组织性，对象构造的起始和设置过程也不例外。应当将这个过程从正常的运行时逻辑中分离出来，确保拥有解决主要依赖问题的全局性一贯策略。

### 11.2.1　分解main

将构造与使用分开的方法之一是将全部构造过程搬迁到`main`或被称之为`main`的模块中，设计系统的其余部分时，假设所有对象都已正确构造和设置。

控制流程很容易理解。`main`函数创建系统所需的对象，再传递给应用程序，应用程序只管使用。注意，看横贯`main`与应用程序之间隔离的依赖箭头的方向，它们都从`main`函数向外走，这表示应用程序对`main`或者构造过程一无所知，它只是简单地指望一切已齐备。

### 11.2.2　工厂

当然，有时应用程序也要负责确定何时创建对象。例如，在某个订单处理系统中，应用程序必须创建`LineItem`实体，添加到`Order`对象。在这种情况下，我们可以使用抽象工厂模式让应用自行控制何时创建`LineItem`，但构造的细节却隔离于应用程序代码之外。

再注意一下，所有依赖都是从`main`指向`OrderProcessing`应用程序的，这表示应用程序与如何构建`LineItem`的细节是分离开来的。其中构建能力由`LineItemFactoryImplementation`持有，而`LineItemFactoryImplementation`又是在`main`这一边的，但应用程序能完全控制`LineItem`实体何时构建，甚至能传递应用程序特定的构造器参数。

### 11.2.3　依赖注入

有一种强大的机制可以实现分离构造与使用，那就是依赖注入（Dependency Injection，DI）。控制反转（Inversion of Control，IoC）在依赖管理中的一种应用手段。控制反转将第二权责从对象中拿出来，转移到另一个专注于此的对象中，从而遵循了单一权责原则。在依赖管理情景中，对象不应负责实体化对自身的依赖，反之，它应当将这份权责移交给其他“有权力”的机制，从而实现控制的反转。因为初始设置是一种全局问题，所以通常这种授权机制要么是`main`例程，要么是有特定目的的容器。

JNDI查找是DI的一种“部分”实现。在JNDI中，对象请求目录服务器提供一种符合某个特定名称的“服务”。

```
MyService myService = (MyService)(jndiContext.lookup("NameOfMyService"));
```

调用对象并不控制真正返回对象的类别（当然前提是它实现了恰当的接口），但调用对象仍然主动解决了依赖问题。

真正的依赖注入还要更进一步。类并不直接解决依赖问题，而是保持完全被动。它提供可用于注入依赖的赋值器方法或构造器参数（或二者皆有）。在构造过程中，DI容器实体化需要的对象（通常按需创建），并使用构造器参数或赋值器方法将依赖连接到一起。至于哪个依赖对象真正得到使用，是通过配置文件或在一个有特殊目的的构造模块中编程决定的。

Spring框架提供了最有名的Java DI容器。用户在XML配置文件中定义互相关联的对象，然后用Java代码请求特定的对象。稍后我们就会看到相关例子。

但延后初始化的好处是什么呢？这种手段在DI中也有其作用。首先，多数DI容器在需要对象之前并不构造对象。其次，许多这类容器提供调用工厂或构造代理的机制，而这种机制可为延迟赋值或类似的优化处理所用。

## 11.3　扩容

城市由城镇而来，城镇由乡村而来。一开始，道路狭窄，几乎无人涉足，随后逐渐拓宽。小型建筑和空地渐渐被更大的建筑所取代，一些地方最终矗立起摩天大楼。

一开始，供电、供水、下水、互联网（哇！）等服务全部欠奉。随着人口和建筑密度的增加，这些服务也开始出现。

这种成长并非全无阵痛。想想看你有多少次开着车，艰难穿行一个“道路改善”工程时，是否问过自己：“他们为什么不一开始就修条够宽的路呢？！”

不过那无论如何不可能实现。谁敢打包票说在小镇里修建一条六车道的公路并不浪费呢？谁会想要这么一条穿过他们小镇的路呢？

“一开始就做对系统”纯属神话。反之，我们应该只去实现今天的用户故事，然后重构，明天再扩展系统、实现新的用户故事。这就是迭代和增量敏捷的精髓所在。测试驱动开发、重构以及它们打造出的整洁代码，在代码层面保证了这个过程的实现。

但在系统层面又如何呢？难道系统架构不需要预先做好计划吗？系统理所当然不可能从简单递增到复杂，它能行吗？

与物理系统相比，软件系统比较独特。软件系统的架构可以递增式地增长，只要我们持续将关注面恰当地切分。

如我们将见到的那样，软件系统短生命周期的本质使这一切变得可行。我们先来看一个没有充分隔离关注问题的架构反例。

初始的EJB1和EJB2架构没有恰当地切分关注面，从而给有机增长压上了不必要的负担。比如一个持久`Bank`类的Entity Bean。Entity Bean是关系数据在内存中的体现，换言之，是表格的一行。

首先，你要定义一个本地（进程内）或远程（分离的JVM）接口，供客户代码使用。代码清单11-1就是一种可能的本地接口：

代码清单11-1　Bank EJB的EJB2本地接口

```
package com.example.banking;
import java.util.Collections;
import javax.ejb.*;

public interface BankLocal extends java.ejb.EJBLocalObject {
  String getStreetAddr1() throws EJBException;
  String getStreetAddr2() throws EJBException;
  String getCity() throws EJBException;
  String getState() throws EJBException;
  String getZipCode() throws EJBException;
  void setStreetAddr1(String street1) throws EJBException;
  void setStreetAddr2(String street2) throws EJBException;
  void setCity(String city) throws EJBException;
  void setState(String state) throws EJBException;
  void setZipCode(String zip) throws EJBException;
  Collection getAccounts() throws EJBException;
  void setAccounts(Collection accounts) throws EJBException;
  void addAccount(AccountDTO accountDTO) throws EJBException;
}
```

上面列出了银行地址的几个属性，和一组该银行拥有的账户，其中每个账户的数据都由单独的`Account` EJB所持有。代码清单11-2展示了`Bank` Bean的相应实现类。

代码清单11-2　相应的EJB2 Entity Bean实现

```
package com.example.banking;
import java.util.Collections;
import javax.ejb.*;

public abstract class Bank implements javax.ejb.EntityBean {
  // Business logic...
  public abstract String getStreetAddr1();
  public abstract String getStreetAddr2();
  public abstract String getCity();
  public abstract String getState();
  public abstract String getZipCode();
  public abstract void setStreetAddr1(String street1);
  public abstract void setStreetAddr2(String street2);
  public abstract void setCity(String city);
  public abstract void setState(String state);
  public abstract void setZipCode(String zip);
  public abstract Collection getAccounts();
  public abstract void setAccounts(Collection accounts);
  public void addAccount(AccountDTO accountDTO) {
    InitialContext context = new InitialContext();
    AccountHomeLocal accountHome = context.lookup("AccountHomeLocal");
    AccountLocal account = accountHome.create(accountDTO);
    Collection accounts = getAccounts();
    accounts.add(account);
  }
  // EJB container logic
  public abstract void setId(Integer id);
  public abstract Integer getId();
  public Integer ejbCreate(Integer id) { ... }
  public void ejbPostCreate(Integer id) { ... }
  // The rest had to be implemented but were usually empty:
  public void setEntityContext(EntityContext ctx) {} 
  public void unsetEntityContext() {}
  public void ejbActivate() {}
  public void ejbPassivate() {}
  public void ejbLoad() {}
  public void ejbStore() {}
  public void ejbRemove() {}
}
```

我没有列出对应的`LocalHome`接口，该接口基本上是用来创建对象的，也没有列出你可能添加的`Bank`查找器（查询）。

最后，你要编写一个或多个XML部署说明，将对象相关映射细节指定给某个持久化存储空间，说明期望的事物行为、安全约束等。

业务逻辑与EJB2应用“容器”紧密耦合。你必须子类化容器类型，必须提供许多个该容器所需要的生命周期方法。

由于存在这种与重量级容器的紧耦合，隔离单元测试就很困难。有必要模拟出容器（这很难），或者花费大量时间在真实服务器上部署EJB和测试。也由于耦合的存在，在EJB2架构之外的复用实际上变得不可能。

最终，连面向对象编程本身也被侵蚀。bean不能继承自另一个bean。留意添加新账号的逻辑。在EJB2 bean中，定义一种本质上是无行为struct的“数据传输对象”（DTO）很常见。这往往会导致拥有同样数据的冗余类型出现，而且也需要在对象之间复制数据的八股式代码。

#### 横贯式关注面

在某些领域，EBJ2架构已经很接近真正的关注面切分。例如，在与源代码分离的部署描述中声明了期待的事务、安全及部分持久化行为。

注意，持久化之类关注面倾向于横贯某个领域的天然对象边界。你会想用同样的策略来持久化所有对象，例如，使用DBMS而非平面文件，表名和列名遵循某种命名约定，采用一致的事务语义，等等。

原则上，你可以从模块、封装的角度推理持久化策略。但在实践上，你却不得不将实现了持久化策略的代码铺展到许多对象中。我们用术语横贯式关注面（Cross-Cutting Concern）来形容这类情况。同样，对于持久化框架和领域逻辑，如果我们孤立地看也可以是模块化的。问题在于横贯这些领域的情形。

实际上，EJB架构处理持久化、安全和事务的方法要早于面向方面编程（aspect-oriented programming，AOP），而AOP是一种恢复横贯式关注面模块化的普适手段。

在AOP中，被称为方面（aspect）的模块构造说明了系统中哪些点的行为会以某种一致的方式被修改，从而支持某种特定的场景。这种说明是用某种简洁的声明或编程机制来实现的。

以持久化为例，可以声明哪些对象和属性（或其模式）应当被持久化，然后将持久化任务委托给持久化框架。行为的修改由AOP框架以无损方式在目标代码中进行。下面来看看Java中的3种方面或类似方面的机制。

## 11.4　Java代理

Java代理适用于简单的情况，例如在单独的对象或类中包装方法调用。然而，JDK提供的动态代理仅能与接口协同工作。对于代理类，你得使用字节码操作库，比如CGLIB、ASM或Javassist。

代码清单11-3展示了为我们的`Bank`应用程序提供持久化支持的JDK代理，代码仅覆盖设置和取得账号列表的方法。

代码清单11-3　JDK代理范例

```
// Bank.java (suppressing package names...)
import java.util.*;

// The abstraction of a bank.
public interface Bank {
  Collection<Account> getAccounts();
  void setAccounts(Collection<Account> accounts);
}

// BankImpl.java
import java.utils.*;

// The "Plain Old Java Object" (POJO) implementing the abstraction.
public class BankImpl implements Bank {
  private List<Account> accounts;

  public Collection<Account> getAccounts() { 
    return accounts; 
  }

  public void setAccounts(Collection<Account> accounts) { 
    this.accounts = new ArrayList<Account>(); 
    for (Account account: accounts) {
      this.accounts.add(account);
    }
  }
}

// BankProxyHandler.java
import java.lang.reflect.*;
import java.util.*;

// "InvocationHandler" required by the proxy API.
public class BankProxyHandler implements InvocationHandler {
  private Bank bank;

  public BankHandler (Bank bank) {
    this.bank = bank;
  }

  // Method defined in InvocationHandler
  public Object invoke(Object proxy, Method method, Object[] args) 
      throws Throwable {
    String methodName = method.getName();
    if (methodName.equals("getAccounts")) {
      bank.setAccounts(getAccountsFromDatabase());
      return bank.getAccounts();
    } else if (methodName.equals("setAccounts")) {
      bank.setAccounts((Collection<Account>) args[0]);
      setAccountsToDatabase(bank.getAccounts());
      return null;
    } else {
      ...
    }
  }

  // Lots of details here:
  protected Collection<Account> getAccountsFromDatabase() { ... }
  protected void setAccountsToDatabase(Collection<Account> accounts) { ... }
}

// Somewhere else...

Bank bank = (Bank) Proxy.newProxyInstance(
  Bank.class.getClassLoader(), 
  new Class[] { Bank.class },
  new BankProxyHandler(new BankImpl()));
```

我们定义了将被代理包装起来的接口`Bank`，还有旧式的Java对象（Plain-Old Java Object，POJO）`BankImpl`，该对象实现业务逻辑（稍后再来看POJO）。

Proxy API需要一个`InvocationHandler`对象，用来实现对代理的全部`Bank`方法调用。`BankProxyHandler`使用Java反射API将一般方法调用映射到`BankImpl`中的对应方法，以此类推。

即便对于这样简单的例子，也有许多相对复杂的代码。使用那些字节操作类库也同样具有挑战性。代码量和复杂度是代理的两大弱点，创建整洁代码变得很难！另外，代理也没有提供在系统范围内指定执行点的机制，而那正是真正的AOP解决方案所必需的。

## 11.5　纯Java AOP框架

幸运的是，编程工具能自动处理大多数代理模板代码。在数个Java框架中，代理都是内嵌的，如Spring AOP和JBoss AOP等，从而能够以纯Java代码实现面向方面编程。在Spring中，你将业务逻辑编码为旧式Java对象。POJO自扫门前雪，并不依赖于企业框架（或其他域），因此，它在概念上更简单、更易于测试驱动，相对简单，也较易于保证正确地实现相应的用户故事，并为未来的用户故事维护和改进代码。

通过使用描述性配置文件或API，你可以把需要的应用程序构架组合起来，包括持久化、事务、安全、缓存、恢复等横贯性问题。在许多情况下，你实际上只是指定Spring或Jboss类库，框架以对用户透明的方式处理使用Java代理或字节代码库的机制。这些声明驱动了依赖注入（DI）容器，DI容器再实体化主要对象，并按需将对象连接起来。

代码清单11-4展示了Spring V2.5配置文件app.xml的典型片段。

代码清单11-4　Spring 2.x的配置文件

```
<beans>
  ...
  <bean id="appDataSource"
  class="org.apache.commons.dbcp.BasicDataSource" 
  destroy-method="close"
  p:driverClassName="com.mysql.jdbc.Driver" 
  p:url="jdbc:mysql://localhost:3306/mydb" 
  p:username="me"/>

  <bean id="bankDataAccessObject"
  class="com.example.banking.persistence.BankDataAccessObject"
  p:dataSource-ref="appDataSource"/>

  <bean  id="bank"
  class="com.example.banking.model.Bank"
  p:dataAccessObject-ref="bankDataAccessObject"/>
  ...
</beans>
```

每个bean就像是嵌套“俄罗斯套娃”中的一个，每个由数据存取器对象（DAO）代理（包装）的`Bank`都有一个域对象，而bean本身又是由JDBC驱动程序数据源代理。

客户代码以为调用的是`Bank`对象的`getAccount()`方法，其实它是在与一组扩展`Bank` POJO基础行为的DECORATOR对象中最外面的那个沟通。

在应用程序中，只添加了少数几行代码，用来向DI容器请求系统中的顶层对象，如XML文件中所定义的那样。

```
XmlBeanFactory bf =
  new XmlBeanFactory(new ClassPathResource("app.xml", getClass()));
Bank bank = (Bank) bf.getBean("bank");
```

只需区区几行与Spring相关的Java代码，应用程序就几乎与Spring完全分离了，消除了EJB2之类系统中那种紧耦合问题。

尽管XML可能会冗长且难以阅读，配置文件中定义的“策略”还是要比那种隐藏在幕后自动创建的复杂的代理和方面逻辑来得简单。这种类型的架构是如此引人注目，Spring之类的框架最终导致了EJB标准在第3版的彻底变化。使用XML配置文件和/或Java 5注解，EJB3很大程度上遵循了Spring通过描述性手段支持横贯式关注面的模型。

代码清单11-5展示了用EJB3重写的`Bank`对象。

代码清单11-5　EJB3版本的Bank

```
package com.example.banking.model;
import javax.persistence.*;
import java.util.ArrayList;
import java.util.Collection;

@Entity
@Table(name = "BANKS")
public class Bank implements java.io.Serializable {
  @Id @GeneratedValue(strategy=GenerationType.AUTO)
  private int id;

  @Embeddable // An object "inlined" in Bank’s DB row
  public class Address { 
    protected String streetAddr1; 
    protected String streetAddr2; 
    protected String city; 
    protected String state; 
    protected String zipCode; 
  }

  @Embedded
  private Address address;

  @OneToMany(cascade = CascadeType.ALL, fetch = FetchType.EAGER, 
             mappedBy = "bank")
  private Collection<Account> accounts = new ArrayList<Account>();

  public int getId() {
    return id;
  }

  public void setId(int id) {
    this.id = id;
  }

  public void addAccount(Account account) {
    account.setBank(this);
    accounts.add(account);
  }

  public Collection<Account> getAccounts() {
    return accounts;
  }

  public void setAccounts(Collection<Account> accounts) {
    this.accounts = accounts;
  }
}
```

上述代码要比原本的EJB2代码整洁多了。有些实体细节仍然在注解中存在。不过，因为没有任何信息超出注解之外，代码依然整洁、清晰，也因此而易于测试驱动、易于维护。

如果愿意的话，注解中的一些或全部持久化信息可以转移到XML部署描述中，只留下真正的纯POJO。如果持久化映射细节不会频繁改动，许多团队可能会选择保留注解，但与EJB2那种侵害性相比还是少了很多问题。

## 11.6　AspectJ的方面

通过方面来实现关注面切分的功能最全的工具是AspectJ语言，它提供“一流的”将方面作为模块构造处理支持的Java扩展。在80%～90%用到方面特性的情况下，Spring AOP和JBoss AOP提供的纯Java实现手段足够使用。然而，AspectJ却提供了一套用以切分关注面的丰富而强有力的工具。AspectJ的弱势在于，需要采用几种新工具，学习新语言构造和使用方式。

借由AspectJ近期引入的“annotation form”（使用Java 5注解定义纯Java代码的方面），采用新工具的问题大大减少。另外，Spring框架也有一些让拥有较少AspectJ经验的团队更容易组合基于注解的方面的特性。

关于AspectJ的全面探讨已经超出本书范围。更多信息可参见[AspectJ]、[Colyer]和[Spring]。

## 11.7　测试驱动系统架构

通过方面式的手段切分关注面的威力不可低估。假使你能用POJO编写应用程序的领域逻辑，在代码层面与架构关注面分离开，就有可能真正地用测试来驱动架构。采用一些新技术，就能将架构按需从简单演化到精细。没必要先做大设计（Big Design Up Front，BDUF）。实际上，BDUF甚至是有害的，它阻碍改进，因为心理上会抵制丢弃既成之事，也因为架构上的方案选择影响后续的设计思路。

建筑设计师不得不做BDUF，因为一旦建造过程开始，就不可能对大型物理建筑的结构做根本性改动。尽管软件也有物理的一面，但只要软件的构架有效切分了各个关注面，还是有可能做根本性改动的。

这意味着我们可以从“简单自然”但切分良好的架构开始做软件项目，快速交付可开展工作的用户故事，随着规模的增加添加更多基础架构。有些世界上最大的网站采用了精密的数据缓存、安全、虚拟化等技术，获得了极高的可用性和性能，在每个抽象层和范围之内，那些最小化耦合的设计都简单到位，效率和灵活性也随之而来。

当然，这不是说要毫无准备地进入一个项目。对于总的覆盖范围、目标、项目进度和最终系统的总体构架，我们会有所预期。不过，我们必须有能力随机应变。

EJB早期架构就是一种著名的过度工程化而没能有效切分关注面的API。在没能真正得到使用时，设计得再好的API也等于是杀鸡用牛刀。优秀的API在大多数时间都该在视线之外，这样团队才能将创造力集中在要实现的用户故事上。否则，架构上的约束就会妨碍向客户交付优化价值的软件。

概言之，

最佳的系统架构由模块化的关注面领域组成，每个关注面均用纯Java（或其他语言）对象实现。不同的领域之间用最不具有侵害性的方面或类方面工具整合起来。这种架构能测试驱动，就像代码一样。

## 11.8　优化决策

模块化和关注面切分成就了分散化管理和决策。在巨大的系统中，不管是一座城市或一个软件项目，无人能做所有决策。

众所周知，对于决策最好是授权给最有资格的人。但我们常常忘记了，延迟决策至最后一刻也是好手段，这不是懒惰或不负责，而是让我们能够基于最有可能的信息做出选择。提前决策是一种预备知识不足的决策。如果决策太早，就会缺少太多客户反馈、关于项目的思考和实施经验。

拥有模块化关注面的POJO系统提供的敏捷能力，允许我们基于最新的知识做出优化的、时机刚好的决策。决策的复杂性也降低了。

## 11.9　明智使用添加了可论证价值的标准

建筑构造大有可观，既因为新建筑的构建过程（即便是在隆冬季节），也因为那些现今科技所能实现的超凡设计。建筑业是一个成熟行业，有着高度优化的部件、方法和久经岁月历练的标准。

即便是轻量级和更直截了当的设计已足敷使用，许多团队也还是采用了EJB2架构，这是因为EJB2是一个标准。我见过一些团队，纠缠于这个或那个名声大噪的标准，却丧失了对为客户实现价值的关注。

有了标准，就更易复用想法和组件、雇用拥有相关经验的人才、封装好点子，以及将组件连接起来。不过，创立标准的过程有时却漫长到行业等不及的程度，有些标准没能与它要服务的采用者的真实需求相结合。

## 11.10　系统需要领域特定语言

建筑，与大多数其他领域一样，发展出一套丰富的语言，有词汇、熟语和清晰而简洁地表达基础信息的句式。在软件领域，领域特定语言（Domain-Specific Language，DSL）最近重受关注。DSL是一种单独的小型脚本语言或以标准语言写就的API，领域专家可以用它编写读起来像是组织严谨的散文一般的代码。

优秀的DSL填平了领域概念和实现领域概念的代码之间的“壕沟”，就像敏捷实践优化了开发团队和甲方之间的沟通一样。如果你用与领域专家使用的同一种语言来实现领域逻辑，就会降低不正确地将领域翻译为实现的风险。

DSL在有效使用时能提升代码惯用法和设计模式之上的抽象层次，它允许开发者在恰当的抽象层级上直指代码的初衷。

领域特定语言允许所有抽象层级和应用程序中的所有领域，从高级策略到底层细节，使用POJO来表达。

## 11.11　小结

系统也应该是整洁的。侵害性架构会湮灭领域逻辑，冲击敏捷能力。如果领域逻辑受到困扰，质量就会堪忧，因为缺陷更易隐藏，用户故事更难实现。当敏捷能力受到损害时，生产力也会降低，TDD的好处遗失殆尽。

在所有的抽象层级上，意图都应该清晰可辨。只有在编写POJO并使用类方面的机制来无损地组合其他关注面时，这种事情才会发生。

无论是设计系统还是单独的模块，别忘了使用大概可开展工作的最简单方案。
