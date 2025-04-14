# 第17章　味道与启发

Martin Fowler在其大作《重构：改善既有代码的设计》（Refactoring: Improving the Design of Existing Code）中指出了许多不同的“代码的坏味道”。下面的清单包括很多Martin指出的“坏味道”，还添加了更多我自己提出的“坏味道”，也包括我借以历练本业的其他珍宝与启发。

我借由遍览和重构几个不同的程序总结出这个清单。每次修改，我都问自己为什么要这样改，把修改的原因写下来，结果就得到相当长的清单，该清单列出了在读代码时让我闻起来不舒服的味道。

清单应按顺序阅读，并作为一种参考来使用。

## 17.1　注释

#### C1：不恰当的信息

让注释传达本该更好地在源代码控制系统、问题追踪系统或任何其他记录系统中保存的信息，是不恰当的。例如，修改历史记录只会用大量过时而无趣的文本搞乱源代码文件。通常，作者、最后修改时间、SPR数等元数据不该在注释中出现。注释只应该描述有关代码和设计的技术性信息。

#### C2：废弃的注释

过时、无关或不正确的注释就是废弃的注释。注释会很快过时。最好别编写将被废弃的注释。如果发现废弃的注释，最好尽快更新或删除。废弃的注释会远离它们曾经描述的代码，变成代码中无关和误导阅读者的浮岛。

#### C3：冗余注释

如果注释描述的是某种充分自我描述了的东西，那么注释就是多余的。例如：

```
i++; // increment i
```

另一个例子是除函数签名之外什么也没多说（或少说）的Javadoc：

```
/**
 * @param sellRequest
 * @return
 * @throws ManagedComponentException
 */
public SellResponse beginSellItem(SellRequest sellRequest) 
  throws ManagedComponentException
```

注释应该谈及代码自身没提到的东西。

#### C4：糟糕的注释

值得编写的注释，也值得好好写。如果要编写一条注释，就花时间保证写出最好的注释，字斟句酌，使用正确的语法和拼写，别闲扯，别画蛇添足，要保持简洁。

#### C5：注释掉的代码

看到被注释掉的代码会令我抓狂。谁知道它有多旧？谁知道它有没有意义？没人会删除它，因为大家都假设别人需要它或是有进一步计划。

那样的代码就这样腐烂掉，随着时间推移，它与系统越来越没关系。它调用不复存在的函数，它使用已改名的变量，它遵循已被废弃的约定，它污染了所属的模块，分散了想要读它的人的注意力。注释掉的代码纯属厌物。

看到注释掉的代码，就删除它！别担心，源代码控制系统还会记得它。如果有人真的需要，可以签出较旧的版本。别被它搞得死去活来。

## 17.2　环境

#### E1：需要多步才能实现的构建

构建系统应该是单步的小操作。不应该从源代码控制系统中一小点一小点签出代码。不应该需要一系列神秘指令或环境依赖脚本来构建单个元素。不应该四处寻找额外的小JAR、XML文件和其他系统所需的杂物。你应当能够用单个命令签出系统，并用单个指令构建它。

```
svn get mySystem 
cd mySystem
ant all
```

#### E2：需要多步才能做到的测试

你应当能够发出单个指令就可以运行全部单元测试。能够运行全部测试是如此基础和重要，应该快速、轻易和直截了当地做到。

## 17.3　函数

#### F1：过多的参数

函数的参数量应该少。没参数最好，一个次之，两个、三个再次之。三个以上的参数非常值得质疑，应坚决避免。（参见3.6节。）

#### F2：输出参数

输出参数违反直觉，因为读者期望参数用于输入而非输出。如果函数非要修改什么东西的状态，就修改它所在对象的状态好了。（参见3.7节。）

#### F3：标识参数

布尔值参数大声宣告函数做了不止一件事。它们令人迷惑，应该被消灭掉。（参见3.6.2节。）

#### F4：死函数

永不被调用的函数应该被丢弃。保留死函数纯属浪费，别害怕删除死函数，记住，源代码控制系统还会记得它。

## 17.4　一般性问题

#### G1：一个源文件中存在多种语言

当今的现代编程环境允许在单个源文件中存在多种不同语言。例如，Java源文件可能还包括XML、HTML、YAML、Javadoc、英文、JavaScript等语言。再如，JSP文件可能还包括HTML、Java、标签库语法、英文注释、Javadoc、XML、JavaScript等。往好处说是令人迷惑，往坏处说就是粗心大意、驳杂不精。

理想的源文件包括且只包括一种语言。现实中，我们可能会不得不使用多于一种语言，但应该尽力缩小源文件中额外语言的数量和范围。

#### G2：明显的行为未被实现

遵循“最小惊异原则”，函数或类应该实现其他程序员有理由期待的行为。例如，考虑一个将日期名称翻译为表示该日期的枚举的函数。

```
Day day = DayDate.StringToDay(String dayName);
```

我们期望字符串`Monday`翻译为`Day.MONDAY`，也期望常用缩写形式能被翻译出来，还期望函数忽略大小写。

如果明显的行为未被实现，读者和用户就不能再依靠他们对函数名称的直觉。他们不再信任原作者，不得不阅读代码细节。

#### G3：不正确的边界行为

代码应该有正确行为，这话看似明白。问题是我们很难明白正确行为有多复杂。开发者常常写出他们以为能工作的函数，信赖自己的直觉，而不是努力去证明代码在所有的角落和边界情形下都真正能工作。

没什么可以替代谨小慎微。每种边界条件、每种极端情形、每个异常都代表了某种可能搞乱优雅而直白的算法的东西。别依赖直觉。追索每种边界条件，并编写测试。

#### G4：忽视安全

切尔诺贝利核电站崩塌了，因为电厂经理一再忽视安全机制。遵守安全规则就不便于做试验，结果就造成试验未能运行，全世界都目睹了首个民用核电站的大灾难。

忽视安全相当危险。手工控制`serialVersionUID`可能有必要，但总会有风险。关闭某些编译器警告（或者全部警告！）可能有助于构建成功，但可能会陷于无穷无尽的调试中。关闭失败测试、告诉自己过后再处理，这和假装刷信用卡不用还钱一样存在安全隐患。

#### G5：重复

有一条本书提到过的最重要的规则之一，你应该非常严肃地对待它，实际上，每位编写有关软件设计的作者都提到过这条规则，Dave Thomas和Andy Hunt称之为DRY原则（Don’t Repeat Yourself，别重复自己）。Kent Beck将它列为极限编程核心原则之一，并称之为“一次，也只一次”。Ron Jeffries将这条规则列在第二位，地位仅次于通过所有测试。

每次看到重复代码，都代表遗漏了抽象。重复的代码可能成为子程序或干脆是另一个类。将重复代码叠放进类似的抽象，增加了你的设计语言的词汇量。其他程序员可以用到你创建的抽象设施。编码变得越来越快，错误越来越少，因为你提升了抽象层级。

重复最明显的形态是你不断看到明显一样的代码，就像是某位程序员疯狂地用鼠标不断复制粘贴代码。可以用单一方法来替代之。

较隐蔽的形态是在不同模块中不断重复出现、检测同一组条件的`switch/case`或`if/else`链。可以用多态来替代之。

更隐蔽的形态是采用类似算法但具体代码行不同的模块。这也是一种重复，可以使用模板方法模式或策略模式来修正。

的确，过去15年内出现的多数设计模式都是消除重复的有名手段。Codd范式（Codd Normal Forms）是消除数据库规划中的重复的策略。面向对象自身也是组织模块和消除重复的策略。毫不出奇，结构化编程也是。

重点已经在那里了。尽可能找到并消除重复。

#### G6：在错误的抽象层级上的代码

创建分离较高层级一般性概念与较低层级细节概念的抽象模型，这很重要。有时，我们创建抽象类来容纳较高层级概念，创建派生类来容纳较低层级概念。这样做的时候，需要确保分离完整。所有较低层级概念放在派生类中，所有较高层级概念放在基类中。

例如，只与细节实现有关的常量、变量或工具函数不应该在基类中出现，基类应该对这些东西一无所知。

这条规则对于源文件、组件和模块也适用。良好的软件设计要求分离位于不同层级的概念，将它们放到不同容器中。这些容器有时是基类或派生类，有时是源文件、模块或组件。无论哪种情况，分离都要完整。较低层级概念和较高层级概念不应混杂在一起。看看下面的代码：

```
public interface Stack {
  Object pop() throws EmptyException;
  void push(Object o) throws FullException;
  double percentFull();
  class EmptyException extends Exception {}
  class FullException extends Exception {}
}
```

函数`percentFull`位于错误的抽象层级。尽管存在许多在其中“充满”（fullness）概念有意义的`Stack`的实现，但也有其他不能知道自己有多满的实现存在。所以，最好将该函数放在类似于`BoundedStack`之类的派生接口中。

你或许会认为，如果栈无边界，实现可以返回0，但问题是，不存在真的无边界的栈。你不能真的避免在做以下检查时出现`OutOfMemoryException`异常：

```
stack.percentFull() < 50.0.
```

实现返回0的函数可能是在撒谎。

要点是你不能就错误放置的抽象模型撒谎。孤立抽象是软件开发者最难做到的事之一，而且一旦做错就没有快捷的修复手段。

#### G7：基类依赖派生类

将概念分解到基类和派生类的最普遍的原因是，较高层级基类概念可以不依赖较低层级派生类概念。这样，如果看到基类而提到派生类名称，就可能发现了问题。通常来说，基类对派生类应该一无所知。

当然也有例外。有时，派生类数量严格固定，而基类中拥有在派生类之间选择的代码。在有限状态机的实现中这种情形很多见。然而，在那种情况下，派生类和基类紧密耦合，总是在同一个jar文件中部署。一般情况下，我们会想要把派生类和基类部署到不同的jar文件中。

将派生类和基类部署到不同的jar文件中，确保基类jar文件对派生类jar文件的内容一无所知，我们就能把系统部署为分散和独立的组件。修改了这些组件时，不必重新部署基组件就能部署它们。这意味着修改产生的影响极大地降低了，而维护系统也变得更加简单。

#### G8：信息过多

设计良好的模块有着非常小的接口，让你事半功倍。设计低劣的模块有着广阔、深入的接口，你不得不事倍功半。设计良好的接口并不提供许多需要依赖的函数，所以耦合度也较低。设计低劣的接口提供大量你必须调用的函数，耦合度较高。

优秀的软件开发人员要学会限制类或模块中暴露的接口数量。类中的方法越少越好，函数知道的变量越少越好，类拥有的实体变量越少越好。

隐藏你的数据，隐藏你的工具函数，隐藏你的常量和你的临时变量。不要创建拥有大量方法或大量实体变量的类，不要为子类创建大量受保护变量和函数。尽力保持接口紧凑。通过限制信息来控制耦合度。

#### G9：死代码

死代码就是不执行的代码。可以在检查不会发生的条件的`if`语句体中找到，可以在从不抛出异常的`try`语句的`catch`块中找到，可以在从不被调用的小工具方法中找到，也可以在永不会发生的`switch/case`条件中找到。

死代码的问题是过不久它就会发出“坏味道”。时间越久，味道就越酸臭。这是因为，在设计改变时，死代码不会随之更新。它还能通过编译，但并不会遵循较新的约定或规则。编写它的时候，系统是另一番模样。如果你找到死代码，就体面地埋葬它，将它从系统中删除掉。

#### G10：垂直分隔

变量和函数应该在靠近被使用的地方定义。本地变量应该正好在其首次被使用的位置上面声明，垂直距离要短。本地变量不该在距离其被使用之处几百行以外的位置声明。

私有函数应该刚好在其首次被使用的位置下面定义。私有函数属于整个类，但我们还是要限制调用和定义之间的垂直距离。找一个私有函数，应该只是从其首次被使用处往下一点儿的位置那么简单。

#### G11：前后不一致

从一而终。这可以追溯到最小惊异原则。小心选择约定，一旦选中，就小心持续遵循。

如果在特定函数中用名为`response`的变量来持有`HttpServletResponse`对象，则在其他用到`HttpServletResponse`对象的函数中也用同样的变量名。如果将某个方法命名为`processVerificationRequest`，则给处理其他请求类型的方法起类似的名字，例如`processDeletionRequest`。

如此简单的前后一致，一旦坚决贯彻，就能让代码更加易于阅读和修改。

#### G12：混淆视听

没有实现的默认构造器有何用处呢？它只会用无意义的杂碎搞乱对代码的理解。没有用到的变量，从不调用的函数，没有信息量的注释，等等，这些都是应该移除的废物。保持源文件整洁，组织良好，不被搞乱。

#### G13：人为耦合

不互相依赖的东西不该耦合。例如，普通的`enum`不应被包括在特殊类中，因为这样一来应用程序就要了解这些更为特殊的类。对于在特殊类中声明一般目的的`static`函数也是如此。

一般来说，人为耦合是指两个没有直接目的的模块之间的耦合。其根源是将变量、常量或函数不恰当地放在临时方便的位置。这是一种漫不经心的偷懒行为。

花点儿时间研究应该在什么地方声明函数、常量和变量。不要为了方便随手放置，然后置之不理。

#### G14：特性依恋

这是Martin Fowler提出的代码的“坏味道”之一。类的方法只应对其所属类中的变量和函数感兴趣，不该垂青其他类中的变量和函数。当方法通过某个其他对象的访问器和修改器来操作该对象内部数据时，它就依恋该对象所属类的范围。它期望自己在那个类里面，这样就能直接访问它操作的变量。例如：

```
public class HourlyPayCalculator {
  public Money calculateWeeklyPay(HourlyEmployee e) {
    int tenthRate = e.getTenthRate().getPennies();
    int tenthsWorked = e.getTenthsWorked();
    int straightTime = Math.min(400, tenthsWorked);
    int overTime = Math.max(0, tenthsWorked - straightTime);
    int straightPay = straightTime * tenthRate;
    int overtimePay = (int)Math.round(overTime*tenthRate*1.5); 
    return new Money(straightPay + overtimePay);
  }
}
```

方法`calculateWeeklyPay`深入到`HourlyEmployee`对象，获取要操作的数据。方法`calculateWeeklyPay`依恋`HourlyEmployee`的作用范围，它“期望”自己能够在`HourlyEmployee`中。

同样情况下，我们要消除特性依恋，因为它将一个类的内部情形暴露给了另外一个类。不过，有时特性依恋是一种有必要的恶行。看一下下面的代码：

```
public class HourlyEmployeeReport {
  private HourlyEmployee employee ;

  public HourlyEmployeeReport(HourlyEmployee e) {
    this.employee = e;
  }

  String reportHours() {
    return String.format(
      "Name: %s\tHours:%d.%1d\n",
      employee.getName(), 
      employee.getTenthsWorked()/10,
      employee.getTenthsWorked()%10);
  }
}
```

显然，`reportHours`方法依恋`HourlyEmployee`类。另外，我们并不想要`HourlyEmployee`得知报告的格式。把格式化字符串移到`HourlyEmployee`会破坏好几种面向对象设计原则（具体是单一权责原则、开放闭合原则和公共关闭原则）。它将把`HourlyEmployee`与报告的格式耦合起来，向该格式的修改暴露这个类。

#### G15：选择算子参数

没有什么比在函数调用末尾遇到一个false参数更为可憎的事情了，这个`false`是什么意思呢？如果它是`true`，会有什么变化吗？不仅一个选择算子（selector）参数的目的令人难以记住，而且每个选择算子参数将多个函数绑到了一起。选择算子参数只是一种避免把大函数切分为多个小函数的偷懒做法。考虑下面这段代码：

```
public int calculateWeeklyPay(boolean overtime) {
  int tenthRate = getTenthRate();
  int tenthsWorked = getTenthsWorked();
  int straightTime = Math.min(400, tenthsWorked);
  int overTime = Math.max(0, tenthsWorked - straightTime);
  int straightPay = straightTime * tenthRate;
  double overtimeRate = overtime ? 1.5 : 1.0 * tenthRate;
  int overtimePay = (int)Math.round(overTime*overtimeRate);
  return straightPay + overtimePay;
}
```

当加班时间以一倍半计算薪资时，用`true`调用这个函数，`false`则表示直接计算。每次用到这个函数，你都得记住`calculateWeeklyPay(false)`表示什么，这已经足够糟糕了，但这种函数真正的坏处在于作者错过了这样写的机会：

```
public int straightPay() {
  return getTenthsWorked() * getTenthRate();
}

public int overTimePay() {
  int overTimeTenths = Math.max(0, getTenthsWorked() - 400);
  int overTimePay = overTimeBonus(overTimeTenths);
  return straightPay() + overTimePay;
}

private int overTimeBonus(int overTimeTenths) {
  double bonus = 0.5 * getTenthRate() * overTimeTenths;
  return (int) Math.round(bonus);
}
```

当然，选择算子不一定是`boolean`类型，而可能是枚举元素、整数或任何一种用于选择函数行为的参数。使用多个函数，通常优于向单个函数传递某些代码来选择函数行为。

#### G16：晦涩的意图

代码要尽可能具有表达力。联排表达式、匈牙利语标记法和魔术数都遮蔽了作者的意图。例如，下面是`overTimePay`函数可能的一种表现形式：

```
public int m_otCalc() {
  return iThsWkd * iThsRte +
    (int) Math.round(0.5 * iThsRte *
      Math.max(0, iThsWkd - 400)
    );
}
```

它既短小又紧凑，但实际上不可捉摸。值得花时间将代码的意图呈现给读者。

#### G17：位置错误的权责

软件开发者做出的最重要决定之一就是在哪里放代码。例如，`PI`常量放在何处？是应该放在`Math`类中吗？或者应该属于`Trigonometry`类？还是属于`Circle`类？

最小惊异原则在这里起作用了。代码应该放在读者自然而然期待它所在的地方。`PI`常量应该出现在声明三角函数的地方。`OVERTIME_RATE`常量应该在`HourlyPayCalculator`类中声明。

有时，我们“聪明”地知道在何处放置功能代码，我们会将其放在自己方便而读者不能随直觉找到的地方。例如，也许我们需要打印出某个雇员的总工作时间的报表。我们可以在打印报表的代码中做工作时间统计，或者我们可以在接受工作时间卡的代码中保留一份工作时间记录。

做这个决定的途径之一是看函数名称。例如，报表模块有一个名为`getTotalHours`的函数。接受时间卡的模块有一个`saveTimeCard`函数。顾名思义，哪个名称暗示了函数会计算总时间呢？答案显而易见。

显然，对于总时间应该在接受时间卡的时候计算，而不是在打印报表时计算，这里面有些性能上的考量。没问题，但函数名称应该反映这种考虑。例如，应该在时间卡模块中有个`computeRunningTotalOfHours`函数。

#### G18：不恰当的静态方法

`Math.max(double a, double)`是一个良好的静态方法。它并不在单个实体上操作。的确，不得不写`new Math().max(a,b)`甚至`a.max(b)`实在愚蠢。那个`max`用到的全部数据来自其两个参数，而不是来自“所属”对象，而且，我们也没机会用到`Math.max`的多态特征。

不过，我们有时也编写不该是静态的静态方法。例如：

```
HourlyPayCalculator.calculatePay(employee, overtimeRate).
```

这看起来像是一个有道理的`static`函数。它并不在任何特定对象上操作，而且从参数中获得全部数据。然而，我们却有理由希望这个函数是多态的。我们可能希望为计算每小时支付的工资实现几种不同算法，如`OvertimeHourlyPayCalculator`和`StraightTimeHourlyPayCalculator`。所以，在这种情况下，该函数就不该是静态的。它应该是`Employee`的非静态成员函数。

通常应该倾向于选用非静态方法。如果有疑问，就用非静态函数。如果的确需要静态函数，确保没机会打算让它有多态行为。

#### G19：使用解释性变量

Kent Beck在其巨著Smalltalk Best Practice Patterns和Implementation Patterns中都写到这个。让程序可读的最有力方法之一就是将计算过程打散成在用有意义的单词命名的变量中放置的中间值。

看看来自FitNesse的这个例子：

```
Matcher match = headerPattern.matcher(line);
if(match.find())
{
  String key = match.group(1);
  String value = match.group(2);
  headers.put(key.toLowerCase(), value);
}
```

解释性变量的这种简单用法，说明了第一个匹配组是`key`，而第二个匹配组是`value`。

这事很难做过火。解释性变量多比少好。只要把计算过程打散成一系列命名良好的中间值，不透明的模块就会突然变得透明，这很值得注意。

#### G20：函数名称应该表达其行为

看看这行代码：

```
Date newDate = date.add(5);
```

你会期望它向日期添加5天吗？或者是5星期？5小时？该`date`实体会变化吗？或者该函数只返回一个新的`Date`实体，并不改动旧的？从函数调用中看不出函数的行为。

如果函数向日期添加5天并且修改该日期，就该命名为`addDaysTo`或`increaseByDays`。如果函数返回一个表示5天后的日期，而不修改日期实体，就该叫作`daysLater`或`daysSince`。

如果你必须查看函数的实现（或文档）才知道它是做什么的，就该换个更好的函数名，或者重新安排功能代码，放到有较好名称的函数中。

#### G21：理解算法

好多可笑代码的出现，是因为人们没花时间去理解算法。他们硬塞进足够多的`if`语句和标识，从不真正停下来考虑发生了什么，勉强让系统能工作。

编程常常是一种探险。你以为自己知道某事的正确算法，然后就卷起袖子瞎干一气，搞到“可以工作”为止。你怎么知道它“可以工作”？因为它通过了你能想到的单元测试。这种做法没错。实际上，这也是让函数按你设想的方式执行的唯一途径。不过，“可以工作”周围的引号可不能一直保留。

在你认为自己完成某个函数之前，确认自己理解了它是怎么工作的。通过全部测试还不够好。你必须知道解决方案是正确的。

获得这种知识和理解的最好途径，往往是重构函数，得到某种整洁而足具表达力、清楚呈示如何工作的东西。

#### G22：把逻辑依赖改为物理依赖

如果某个模块依赖另一个模块，依赖就该是物理上的而不是逻辑上的。依赖者模块不应对被依赖者模块有假定（换言之，逻辑依赖），它应当明确地询问后者全部信息。

例如，想象你在编写一个打印出雇员工作时长的纯文本报表的函数。有一个名为`HourlyReporter`的类把数据收集为某种方便的形式，传递到`HourlyReportFormatter`中，再打印出来。（如代码清单17-1所示。）

代码清单17-1　HourlyReporter.java

```
public class HourlyReporter {
  private HourlyReportFormatter formatter;
  private List<LineItem> page;
  private final int PAGE_SIZE = 55;

  public HourlyReporter(HourlyReportFormatter formatter) {
    this.formatter = formatter;
    page = new ArrayList<LineItem>();
  }

  public void generateReport(List<HourlyEmployee> employees) {
    for (HourlyEmployee e : employees) {
      addLineItemToPage(e);
      if (page.size() == PAGE_SIZE)
        printAndClearItemList();
    }
    if (page.size() > 0)
      printAndClearItemList();
  }

  private void printAndClearItemList() {
    formatter.format(page);
    page.clear();
  }

  private void addLineItemToPage(HourlyEmployee e) {
    LineItem item = new LineItem();
    item.name = e.getName();
    item.hours = e.getTenthsWorked() / 10;
    item.tenths = e.getTenthsWorked() % 10;
    page.add(item);
  }

  public class LineItem {
    public String name;
    public int hours;
    public int tenths;
  }
}
```

这段代码有尚未物理化的逻辑依赖。你能指出来吗？那就是常量`PAGE_SIZE`。`HourlyReporter`为什么要知道页面尺寸？页面尺寸只该是`HourlyReportFormatter`的权责。

`PAGE_SIZE`在`HourlyReporter`中声明，代表了一种位置错误的权责[G17]，导致`HourlyReporter`假定它知道页面尺寸。这类假设是一种逻辑依赖。`HourlyReporter`依赖`HourlyReportFormatter`能应对55的页面尺寸。如果`HourlyReportFormatter`的某些实现不能处理这样的尺寸，就会出错。

可以通过创建`HourlyReport`中名为`getMaxPageSize()`的新方法来物理化这种依赖。`HourlyReporter`将调用这个方法，而不是使用`PAGE_SIZE`常量。

#### G23：用多态替代If/Else或Switch/Case

有了第6章谈及的主题，这条建议看似奇怪。在第6章中，我提出在添加新函数甚于添加新类型的系统中，`switch`语句是恰当的。

首先，多数人使用`switch`语句，因为它是最直截了当又有力的方案，而不是因为它适合当前情形。这给我们的启发是在使用`switch`之前，先考虑使用多态。

其次，函数变化甚于类型变化的情形相对罕见。每个`switch`语句都值得怀疑。

我使用所谓“单个`switch`”规则：对于给定的选择类型，不应有多于一个的`switch`语句。在那个`switch`语句中的多个`case`，必须创建多态对象，取代系统中其他类似的`switch`语句。

#### G24：遵循标准约定

每个团队都应遵循基于通用行业规范的一套编码标准。编码标准应指定诸如在何处声明实体变量，如何命名类、方法和变量，在何处放置括号，等等。团队不应用文档描述这些约定，因为代码本身提供了范例。

团队中的每个成员都应遵循这些约定。这意味着每个团队成员必须成熟到能了解只要全体同意在何处放置括号，那么在哪里放置都无关紧要。

如果你想知道我遵循哪些约定，可以查看代码清单B-7至代码清单B-14中重构之后的代码。

#### G25：用命名常量替代魔术数

这大概是软件开发中最古老的规则之一了。我记得，在20世纪60年代介绍COBOL、FORTRAN和PL/1的手册中就读到过。在代码中出现原始形态数字通常来说是坏现象。应该用命名良好的常量来隐藏它。

例如，数字86400应当藏在常量`SECONDS_PER_DAY`后面。如果每页打印55行，则常数55应该藏在常量`LINES_PER_PAGE`后面。

有些常量与非常具有自我解释能力的代码协同工作时，如此易于识别，也就不必总是需要命名常量来隐藏了。例如：

```
double milesWalked = feetWalked/5280.0;
int dailyPay = hourlyRate * 8;
double circumference = radius * Math.PI * 2;
```

在上例中，我们真需要常量`FEET_PER_MILE`、`WORK_HOURS_PER_DAY`和`TWO`吗？显然，最后那个很可笑。有些情况下，常量直接写作原始形态数字会更好。你可能会质疑`WORK_HOURS_PER_DAY`，因为约定规则可能会改变。另外，在这里直接用数字8读起来很舒服，也就没必要非用17个额外的字母来加重读者负担。对于`FEET_PER_MILE`，数字5280众人皆知，意义独特，即便没有上下文环境，读者也能识别它。

3.141 592 653 589 793之类的常数也众所周知，很容易识别。不过，如果直接使用原始形式，则很有可能出错。每次有人看到3.141 592 653 589 793，都会知道那是π值，从而不会去仔细查看。（你发现那个错误的数字了吗？）我们不想要人们使用3.14、3.14 159或3.142等。所以，为我们定义好`Math.PI`是一件好事。

术语“魔术数”不仅指数字，它还泛指任何不能自我描述的符号。例如：

```
assertEquals(7777, Employee.find("John Doe").employeeNumber());
```

上述断言中有两个魔术数。第一个魔术数显然是7777，它的意义并不明确。第二个魔术数是`"John Doe"`，因为其意图不明显。

`"John Doe"`是开发团队创建的测试数据中编号为#7777的雇员。团队中每个成员都知道，当连接到数据库时，里面已经有数个雇员信息，其值和属性都是大家熟知的。所以，这个测试应该读作：

```
assertEquals(
  HOURLY_EMPLOYEE_ID,
  Employee.find(HOURLY_EMPLOYEE_NAME).employeeNumber());
```

#### G26：准确

期望某个查询的第一次匹配就是唯一匹配可能过于天真。用浮点数表示货币几近于犯罪。因为你不想做并发更新就避免使用锁和/或事务管理往好处说也是一种懒惰行为。在可以用`List`的时候非要把变量声明为`ArrayList`就过于拘束了。把所有变量设置为`protected`却不够自律。

在代码中做决定时，确认自己足够准确。明确自己为何要这么做，如果遇到异常情况如何处理。别懒得理会决定的准确性。如果你打算调用可能返回`null`的函数，确认自己检查了`null`值。如果查询你认为是数据库中唯一的记录，确保代码检查不存在其他记录。如果要处理货币数据，使用整数，并恰当地处理四舍五入。如果可能有并发更新，确认你实现了某种锁定机制。

代码中的含糊和不准确要么是意见不同的结果，要么源于懒惰。无论原因是什么，都要消除。

#### G27：结构甚于约定

坚守结构甚于约定的设计原则。命名约定很好，但却次于强制性的结构。例如，用到命名良好的枚举的`switch/case`要弱于拥有抽象方法的基类。没人会被强迫每次都以同样方式实现`switch/case`语句，但基类却让具体类必须实现所有抽象方法。

#### G28：封装条件

如果没有`if`或`while`语句的上下文，布尔逻辑就难以理解。应该把解释了条件意图的函数抽离出来。

例如：

```
if (shouldBeDeleted(timer))
```

要好于

```
if (timer.hasExpired() && !timer.isRecurrent())
```

#### G29：避免否定性条件

否定式要比肯定式难明白一些。所以，尽可能将条件表示为肯定形式。例如：

```
if (buffer.shouldCompact())
```

要好于

```
if (!buffer.shouldNotCompact())
```

#### G30：函数只该做一件事

编写执行一系列操作的包括多段代码的函数常常是诱人的。这类函数做了不只一件事，应该转换为多个更小的函数，每个小函数只做一件事。

例如：

```
public void pay() {
  for (Employee e : employees) {
    if (e.isPayday()) {
      Money pay = e.calculatePay();
      e.deliverPay(pay);
    }
  }
}
```

这段代码做了3件事。它遍历所有雇员，检查是否该给雇员付工资，然后支付薪水。代码可以写得更好，如：

```
public void pay() {
  for (Employee e : employees)
    payIfNecessary(e);
}

private void payIfNecessary(Employee e) {
  if (e.isPayday())
    calculateAndDeliverPay(e);
}

private void calculateAndDeliverPay(Employee e) {
  Money pay = e.calculatePay();
  e.deliverPay(pay);
}
```

上列每个函数都只做一件事。（见3.2节。）

#### G31：掩蔽时序耦合

常常有必要使用时序耦合，但你不应该掩蔽它。排列函数参数，好让它们被调用的次序显而易见。看下列代码：

```
public class MoogDiver {
  Gradient gradient;
  List<Spline> splines;

  public void dive(String reason) {
    saturateGradient();
    reticulateSplines();
    diveForMoog(reason);
  }
  ...
}
```

3个函数的次序很重要。捕鱼之前先织网，织网之前先编绳。不幸的是，代码并没有强制这种时序耦合。其他程序员可以在调用`saturateGradient`之前调用`reticulateSplines`，从而导致抛出`UnsaturatedGradientException`异常。更好的方式是：

```
public class MoogDiver {
  Gradient gradient;
  List<Spline> splines;

  public void dive(String reason) {
    Gradient gradient = saturateGradient();
    List<Spline> splines = reticulateSplines(gradient);
    diveForMoog(splines, reason);
  }
  ...
}
```

这样就通过创建顺序队列暴露了时序耦合。每个函数都产生出下一个函数所需的结果，这样一来就没理由不按顺序调用了。

你可能会抱怨这增加了函数的复杂度，没错，不过这点额外的复杂度却曝露了该种情况真正的时序复杂性。

注意，我保留了那些实体变量。我假设类中的私有方法可能会用到它们。即便如此，我还是希望参数能让时序耦合变得可见。

#### G32：别随意

构建代码需要理由，而且理由应与代码结构相契合。如果结构显得太随意，其他人就会想修改它。如果结构自始至终保持一致，其他人就会使用它，并且遵循其约定。例如，我最近对FitNesse做合并修改，发现有位贡献者这么做：

```
public class AliasLinkWidget extends ParentWidget
{
  public static class VariableExpandingWidgetRoot {
    ...

  ...
}
```

问题在于，`VariableExpandingWidgetRoot`没必要在`AliasLinkWidget`作用范围之内。而且，其他无关的类也用到`AliasLinkWidget.VariableExpandingWidgetRoot`，这些类没必要了解`AliasLinkWidget`。

或许那位程序员只是循例把`VariableExpandingWidgetRoot`放到`AliasWidget`里面，或者他真认为这么做是对的。不管是什么原因，结果都显得随意。不作为类工具的公共类，不应该放到其他类里面，惯例是将它置为`public`，并且放在代码包的顶部。

#### G33：封装边界条件

边界条件难以追踪。把处理边界条件的代码集中到一处，不要散落于代码中。我们不想见到四处散见的`+1`和`−1`字样。看看这个来自FIT的简单例子：

```
if(level + 1 < tags.length)
{
  parts = new Parse(body, tags, level + 1, offset + endTag);
  body = null;
}
```

注意，`level+1`出现了两次。这是一个应该封装到名为`nextLevel`之类的变量中的边界条件。

```
int nextLevel = level + 1;
if(nextLevel < tags.length)
{
  parts = new Parse(body, tags, nextLevel, offset + endTag);
  body = null;
}
```

#### G34：函数应该只在一个抽象层级上

函数中的语句应该在同一抽象层级上，该层级应该是函数名所示操作的下一层。这可能是最难理解和遵循的启发。尽管概念足够直白，但是人们还是很容易混淆抽象层级。例如，请看下面来自FitNesse的例子：

```
public String render() throws Exception
{
  StringBuffer html = new StringBuffer("<hr");
  if(size > 0)
    html.append(" size=\"").append(size + 1).append("\"");
  html.append(">");

  return html.toString();
}
```

稍微研究一下，你就会看到发生了什么。该函数构建了绘制横贯页面线条的HTML标记。线条高度在`size`变量中指定。

再看一遍。方法混杂了至少两个抽象层级，第一个是横线有尺寸这个概念，第二个是`hr`标记自身的语法。这段代码来自FitNesse的`HruleWidget`模块。该模块检测一行4个或更多个破折号，并将其转换为恰当的`hr`标记。破折号越多，尺寸越大。

我重构了这段代码。注意，我修改了`size`字段的名称，新名称反映其真正目的，表示额外的破折号的数量。

```
public String render() throws Exception
{
  HtmlTag hr = new HtmlTag("hr");
  if (extraDashes > 0)
    hr.addAttribute("size", hrSize(extraDashes));
  return hr.html();
}

private String hrSize(int height)
{
  int hrSize = height + 1;
  return String.format("%d", hrSize);
}
```

这次修改很好地拆开了两个抽象层级。函数`render`只构造一个`hr`标记，不去管该标记的HTML语法。而`HtmlTag`模块则照管所有这些肮脏的语法问题。

做出修改时，我发现了一处微小的错误。原始代码没有加上`hr`标记的结束斜线符，而XHTML标准要求这样做。（换言之，代码使用了`<hr>`而不是`<hr />`。）`HtmlTag`模块很早就改造成符合XHTML标准了。

拆分不同抽象层级是重构的最重要功能之一，也是最难实现的一个功能。以下面的代码为例。这是我第一次尝试拆分`HruleWidget.rendermethod`中的抽象层级的结果。

```
public String render() throws Exception
{
  HtmlTag hr = new HtmlTag("hr");
  if (size > 0) {
    hr.addAttribute("size", ""+(size+1));
  }
  return hr.html();
}
```

此时，我的目的是做必要的拆分，并让测试通过。我轻易达到了这一目的，但结果是该函数仍然混杂了多个抽象层级。此时，混杂的层级是`hr`标记的构建，以及`size`变量的翻译和格式化。这说明当你循抽象界线拆解函数时，经常会挖出原本被之前的结构所掩蔽的新抽象界线。

#### G35：在较高层级放置可配置数据

如果你有一个已知并该在较高抽象层级的默认常量或配置值，不要将它埋藏到较低层级的函数中。把它作为较高层级函数调用较低层级函数时的一个参数。看看以下来自FitNesse的代码：

```
    public static void main(String[] args) throws Exception
    {
      Arguments arguments = parseCommandLine(args);
      ...
    }

public class Arguments
{
  public static final String DEFAULT_PATH = ".";
  public static final String DEFAULT_ROOT = "FitNesseRoot";
  public static final int DEFAULT_PORT = 80;
  public static final int DEFAULT_VERSION_DAYS = 14;
  ...
}
```

命令行参数在FitNesse中的第一行可执行代码得到解析。这些参数的默认值在`Argument`类的顶部指定。你不必到系统的较低层级去查看类似的语句：

```
if (arguments.port == 0) // use 80 by default
```

位于较高层级的配置性常量易于修改，它们向下贯穿应用程序。应用程序的较低层级并不拥有这些常量的值。

#### G36：避免传递浏览

通常我们不想让某个模块了解太多其协作者的信息。更具体地说，如果`A`与`B`协作，`B`与`C`协作，我们不想让使用`A`的模块了解`C`的信息。（例如，我们不想写类似于`a.getB().getC().doSomething()`的代码。）

这就是所谓的得墨忒耳律。《程序员修炼之道》（The Pragmatic Programmers）称之为“编写害羞代码”。两者都归结为确保模块只了解其直接协作者，而不了解整个系统的游览图。

如果有多个模块使用类似`a.getB().getC()`这样的语句形式，就难以修改设计和架构，要在`B`和`C`之间插进一个`Q`，就得找到`a.getB().getC()`出现的所有地方，并将其改为`a.getB().getQ().getC()`。系统就会变得缺乏柔韧性。太多的模块了解了太多有关架构的信息。

正确的做法是让直接协作者提供所需的全部服务，而不必逛遍系统的对象全图，搜寻我们要调用的方法。只要简单地写：

```
myCollaborator.doSomething().
```

## 17.5　Java

#### J1：通过使用通配符避免过长的导入清单

如果使用了来自同一程序包的两个或多个类，用以下语句导入整个包：

```
import package.*;
```

过长的导入清单令读者望而却步。我们不想用80行导入语句搞乱模块顶部位置。我们想要导入语句简约地列出我们要使用的包。

指定导入包是一种硬依赖，而通配符导入则不是。如果你具体指定导入某个类，那么该类必须存在。但如果你用通配符导入某个包，则不需要存在具体的类。导入语句只在搜寻名称时把这个包列入查找路径。所以，这种导入并未构成真正的依赖，也就让我们的模块较少耦合。

有时，长长的具体导入清单也会有用。例如，如果你在处理遗留下来的代码，想要找出需要为哪些类构造替身类和占位代码，就可以遍历导入清单，找出这些类的真名，再恰当地放置占位代码。不过，这种用法很罕见。而且，多数现代IDE允许你用一个命令就把通配符导入语句转换为指定导入清单。所以，即便在处理遗留代码时，最好也用通配符导入。

通配符导入有时会导致名称冲突和歧义。两个同名但位于不同包中的类需要指名导入，或至少在使用时指定名称。这种情形的确讨厌，不过很罕见，所以使用通配符导入通常仍优于指定名称导入。

#### J2：不要继承常量

我见过这种情况好几次，它总是让我面露苦笑。某个程序在接口中放了一些常量，再通过继承结构来访问这些常量。看看以下代码：

```
public class HourlyEmployee extends Employee {
  private int tenthsWorked;
  private double hourlyRate;

  public Money calculatePay() {
    int straightTime = Math.min(tenthsWorked, TENTHS_PER_WEEK);
    int overTime = tenthsWorked - straightTime;
    return new Money(
      hourlyRate * (tenthsWorked + OVERTIME_RATE * overTime)
    );
  }
  ...
}
```

常量`TENTHS_PER_WEEK`和`OVERTIME_RATE`来自何方？它们可能来自`Employee`类。来看看：

```
public abstract class Employee implements PayrollConstants {
  public abstract boolean isPayday();
  public abstract Money calculatePay();
  public abstract void deliverPay(Money pay);
}
```

不，不在那儿，但在哪儿呢？再仔细看`Employee`类，它实现了`PayrollConstants`接口。

```
public interface PayrollConstants {
  public static final int TENTHS_PER_WEEK = 400;
  public static final double OVERTIME_RATE = 1.5;
}
```

真是丑陋不堪！常量躲在了继承结构的最顶端。呸！别利用继承欺骗编程语言的作用范围规则，而应该用静态导入。

```
import static PayrollConstants.*;

public class HourlyEmployee extends Employee {
  private int tenthsWorked;
  private double hourlyRate;

  public Money calculatePay() {
    int straightTime = Math.min(tenthsWorked, TENTHS_PER_WEEK);
    int overTime = tenthsWorked - straightTime;
    return new Money(
      hourlyRate * (tenthsWorked + OVERTIME_RATE * overTime)
    );
  }
  ...
}
```

#### J3：常量和枚举

现在`enum`已经加入Java语言（Java 5），放心用吧！别再用那个`public static final int`老花招，那样做`int`的意义就丧失了，而用`enum`则不然，因为它隶属于有名称的枚举。

而且，仔细研究`enum`的语法，它可以拥有方法和字段，从而成为能比`int`提供更多表达力和灵活性的强有力工具。看看以下发薪代码中的不同做法：

```
public class HourlyEmployee extends Employee {
  private int tenthsWorked;
  HourlyPayGrade grade;

  public Money calculatePay() {
    int straightTime = Math.min(tenthsWorked, TENTHS_PER_WEEK);
    int overTime = tenthsWorked - straightTime;
    return new Money(
      grade.rate() * (tenthsWorked + OVERTIME_RATE * overTime)
    );
  }
  ...
}

public enum HourlyPayGrade {
  APPRENTICE {
    public double rate() {
      return 1.0;
    }
  },
  LIEUTENANT_JOURNEYMAN {
    public double rate() {
      return 1.2;
    }
  },
  JOURNEYMAN {
    public double rate() {
      return 1.5;
    }
  },
  MASTER {
    public double rate() {
      return 2.0;
    }
  };

  public abstract double rate();
}
```

## 17.6　名称

#### N1：采用描述性名称

不要太快起名。确认名称具有描述性。记住，事物的意义随着软件的演化而变化，所以，要经常性地重新估量名称是否恰当。

这不仅是一条“感觉良好式”建议。软件中的名称对于软件可读性有90%的作用。你要花时间明智地起名，保持名称有关。名称太重要了，不可随意对待。

看看以下代码。这段代码是做什么的？用了好名称的代码一目了然，而这样的代码却是符号和魔术数的大杂烩。

```
public int x() {
  int q = 0;
  int z = 0;
  for (int kk = 0; kk < 10; kk++) {
    if (l[z] == 10)
    {
      q += 10 + (l[z + 1] + l[z + 2]);
      z += 1;
    }
    else if (l[z] + l[z + 1] == 10)
    {
      q += 10 + l[z + 2];
      z += 2;
    } else {
      q += l[z] + l[z + 1];
      z += 2;
    }
  }
  return q;
}
```

下面是这段代码应该写成的样子。代码片段实际上不如上段完整，但你还是能马上推断出它要做什么，而且很有可能依据推断出的意思写出遗漏的函数。魔术数不复神秘，算法的结构也足具描述性。

```
public int score() {
  int score = 0;
  int frame = 0;
  for (int frameNumber = 0; frameNumber < 10; frameNumber++) {
    if (isStrike(frame)) {
      score += 10 + nextTwoBallsForStrike(frame);
      frame += 1;
    } else if (isSpare(frame)) {
      score += 10 + nextBallForSpare(frame);
      frame += 2;
    } else {
      score += twoBallsInFrame(frame);
      frame += 2;
    }
  }
  return score;
}
```

仔细起好的名称的威力在于，它用描述性信息覆盖了代码。这种信息覆盖设定了读者对于模块中其他函数行为的期待。看看上面的代码，你就能推断出`isStrike()`的实现。读到`isStrick`方法时，它“深合你意”。

```
private boolean isStrike(int frame) {
  return rolls[frame] == 10;
}
```

#### N2：名称应与抽象层级相符

不要起沟通实现的名称，而起反映类或函数抽象层级的名称。这样做不容易。人们擅长于混杂抽象层级。每次浏览代码，你总会发现有些变量的名称层级太低。你应当趁机为之改名。要让代码可读，需要持续不断的改进。看看下面的`Modem`接口：

```
public interface Modem {
  boolean dial(String phoneNumber);
  boolean disconnect();
  boolean send(char c);
  char recv();
  String getConnectedPhoneNumber();
}
```

粗看还行。函数看来都很合适，对多数应用程序来说是这样。不过，想想看某个应用中有些调制解调器并不用拨号连接的情形，有些用线缆直连，有些通过向USB口发送端口信息连接。显然，有关电话号码的信息就是位于错误的抽象层级了。对于这种情形，更好的命名策略可能是：

```
public interface Modem {
  boolean connect(String connectionLocator);
  boolean disconnect();
  boolean send(char c);
  char recv();
  String getConnectedLocator();
}
```

现在名称不再与电话号码有关系，并且还是可以用于用电话号码的情形，也可以用于其他连接策略。

#### N3：尽可能使用标准命名法

如果名称基于既存约定或用法，就比较易于理解。例如，如果你采用油漆工模式，就该在给油漆类命名时用上`Decorator`字样。例如，`AutoHangupModemDecorator`可能是某个给`Modem`类刷上在会话结束时自动挂机的能力的类的名称。

模式只是标准的一种。例如，在Java中，将对象转换为字符串的函数通常命名为`toString`。最好是遵循这些约定，而不是自己创造命名法。

对于特定项目，开发团队常常发明自己的命名标准系统。Eric Evans称之为项目的共同语言。代码应该使用来自这种语言的术语。简言之，具有与项目有关的特定意义的名称用得越多，读者就越容易明白你的代码是做什么的。

#### N4：无歧义的名称

选用不会混淆函数或变量意义的名称。看看来自FitNesse的这个例子：

```
private String doRename() throws Exception
{
  if(refactorReferences)
    renameReferences();
  renamePage();

  pathToRename.removeNameFromEnd();
  pathToRename.addNameToEnd(newName);
  return PathParser.render(pathToRename);
}
```

该函数的名称含混不清，没有说明函数的作用。由于在`doRename`函数里面还有一个名为`renamePage`的函数，这就更不明白了！这些名称有没有说明两个函数之间的区别呢？没有。

更好的函数名称应该是`renamePageAndOptionallyAllReferences`。看似太长，的确是很长，不过它只在模块中的一处被调用，所以其解释性的好处大过了长度的坏处。

#### N5：为较大作用范围选用较长名称

名称的长度应与作用范围的广泛度相关。对于较小的作用范围，可以用很短的名称，而对于较大的作用范围，就该用较长的名称。

类似`i`和`j`之类的变量名对于作用范围在5行之内的情形没问题。看看以下来自老“标准保龄球游戏”的代码片段：

```
private void rollMany(int n, int pins)
{
  for (int i=0; i<n; i++)
    g.roll(pins);
}
```

这段代码很明白，如果用`rollCount`之类烦人的名称代替变量`i`，反而是徒增混乱。另外，在较长距离上，使用短名称的变量和函数会丧失其含义。名称的作用范围越大，名称就该越长、越准确。

#### N6：避免编码

不应在名称中包括类型或作用范围信息。在如今的开发环境中，`m_`或`f`之类的前缀完全无用。类似`vis_`（表示图形系统）之类的项目或子系统名称也属多余。当今的开发环境不用纠缠于名称也能提供这些信息。不要用匈牙利语命名法污染你的名称。

#### N7：名称应该说明副作用

名称应该说明函数、变量或类的一切信息。不要用名称掩蔽副作用。不要用简单的动词来描述做了不止一个简单动作的函数。例如，请看以下来自`TestNG`的代码：

```
public ObjectOutputStream getOos() throws IOException {
  if (m_oos == null) {
    m_oos = new ObjectOutputStream(m_socket.getOutputStream());
  }
  return m_oos;
}
```

该函数不只是获取一个“oos”，如果“oos”不存在，还会创建一个“oos”。所以，更好的名称大概是`createOrReturnOos`。

## 17.7　测试

#### T1：测试不足

一套测试中应该有多少个测试？不幸的是，许多程序员的衡量标准是“看起来够了”。一套测试应该测到所有可能失败的东西。只要还有没被测试探测过的条件，或是还有没被验证过的计算，测试就还不够。

#### T2：使用覆盖率工具

覆盖率工具能汇报你测试策略中的缺口。使用覆盖率工具能更容易地找到测试不足的模块、类和函数。多数IDE都能给出直观的指示，用绿色标记测试覆盖了的代码行，而未覆盖的代码行则是红色。这样就能又快又容易地找到尚未检测过的`if`或`catch`语句。

#### T3：别略过小测试

小测试易于编写，其文档上的价值高于编写成本。

#### T4：被忽略的测试就是对不确定事物的疑问

有时，我们会因为需求不明而不能确定某个行为细节。可以用注释掉的测试或者用`@Ignore`注解的测试来表达我们对于需求的疑问。使用哪种方式，取决于该不确定性所涉及的代码是否要编译。

#### T5：测试边界条件

特别注意测试边界条件。算法的中间部分正确但边界判断错误的情形很常见。

#### T6：全面测试相近的缺陷

缺陷趋向于扎堆。在某个函数中发现一个缺陷时，最好全面测试那个函数。你可能会发现缺陷不止一个。

#### T7：测试失败的模式有启发性

有时，你可以通过找到测试用例失败的模式来诊断问题所在。这也是尽可能编写足够完整的测试用例的理由之一。完整的测试用例，按合理的顺序排列，能暴露出模式。

简单举例，假设你注意到所有长于5个字符的输入，或者向函数的第二个参数传入负数都会导致测试失败。有时，只要看看测试报告的红绿模式，就足以绽放出那句带来解决方法的“啊哈！”回头看看第16章中的有趣例子吧。

#### T8：测试覆盖率的模式有启发性

查看被或未被已通过的测试执行的代码，往往能发现失败的测试为何失败的线索。

#### T9：测试应该快速

慢速的测试是不会被运行的测试。时间一紧，较慢的测试就会被摘掉。所以，竭尽所能让测试够快。

## 17.8　小结

这份关于启发与坏味道的清单很难说已完备无缺。我不能确定这样一份清单会不会完备无缺。但或许完整性不该是目标，因为该清单确实给出了一套价值体系。

这套价值体系才该是目标，也是本书的主题所在。整洁代码并非遵循一套规则写就。学习一系列启发并不足以让你成为软件匠人。专业性和技艺来自驱动规程的价值观。
