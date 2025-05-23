# 第2章　有意义的命名

## 2.1　介绍

本章列出了起个好名字应遵从的几条简单规则。

## 2.2　名副其实

变量、函数或类的名称应该已经答复了所有的大问题。它该告诉你，它为什么会存在，它做什么事，应该怎么用。如果名称需要注释来补充，那就不算是名副其实。

```
int d;  // elapsed time in days

```

名称`d`什么也没说明。它没有引起读者对时间消逝的感觉，更别说以日计了。我们应该选择指明了计量对象和计量单位的名称：

```
int elapsedTimeInDays;
int daysSinceCreation;
int daysSinceModification;
int fileAgeInDays;

```

选择体现本意的名称能让人更容易理解和修改代码。下列代码的目的何在？

```
public List<int[]> getThem() {
  List<int[]> list1 = new ArrayList<int[]>();
  for (int[] x : theList) 
    if (x[0] == 4) 
      list1.add(x);
  return list1;
}

```

为什么难以说明上述代码要做什么事？里面并没有复杂的表达式，空格和缩进中规中矩，只用到三个变量和两个常量，甚至没有涉及任何其他类或多态方法，只是（或者看起来是）一个数组的列表而已。

问题不在于代码的简洁度，而在于代码的模糊度：即上下文在代码中未被明确体现的程度。上述代码要求我们了解类似以下问题的答案：

（1）`theList`中是什么类型的东西？

（2）`theList`零下标条目的意义是什么？

（3）值`4`的意义是什么？

（4）我怎么使用返回的列表？

问题的答案没体现在代码段中，可代码段就是它们该在的地方。比方说，我们在开发一种扫雷游戏，我们发现，盘面是名为`theList`的单元格列表，那就将其名称改为`gameBoard`。

盘面上每个单元格都用一个简单数组表示。我们还发现，零下标条目是一种状态值，而该种状态值为`4`表示“已标记”。只要改为有意义的名称，代码就会得到相当程度的改进：

```
public List<int[]> getFlaggedCells()  {
  List<int[]> flaggedCells = new ArrayList<int[]>();
  for (int[] cell : gameBoard)
    if (cell[STATUS_VALUE] == FLAGGED)
      flaggedCells.add(cell);
  return flaggedCells;
}

```

注意，代码的简洁性并未被触及。运算符和常量的数量全然保持不变，嵌套数量也全然保持不变，但代码变得明确多了。

还可以更进一步，不用`int`数组表示单元格，而是另写一个类。该类包括一个名副其实的函数（称为`isFlagged`），从而掩盖住那个魔术数（即表示已标记的`4`）。于是得到函数的新版本：

```
public List<Cell> getFlaggedCells()  {
  List<Cell> flaggedCells = new ArrayList<Cell>();
  for (Cell cell : gameBoard) 
    if (cell.isFlagged()) 
      flaggedCells.add(cell); 
  return flaggedCells;
}

```

只要简单改一下名称，就能轻易知道发生了什么。这就是选用好名称的力量。

## 2.3　避免误导

程序员必须避免留下掩藏代码本意的错误线索。应当避免使用与本意相悖的词，例如，`hp`、`aix`和`sco`都不该用作变量名，因为它们都是Unix平台或类Unix平台的专有名称。即便你是在编写三角计算程序，`hp`看起来是一个不错的缩写（即`hypotenuse`的缩写），但那也可能会提供错误信息。

别用`accountList`来指称一组账号，除非它真的是`List`类型。`List`一词对程序员有特殊意义。如果包纳账号的容器并非真是一个`List`，就会引起错误的判断（如后文提到的，即便容器就是一个`List`，最好也别在名称中写出容器类型名）。所以，用`accountGroup`或`bunchOfAccounts`，甚至直接用`accounts`都会好一些。

提防使用外形相似度较高的名称。例如，想区分模块中某处的`XYZControllerFor-EfficientHandlingOfStrings`和另一处的`XYZControllerForEfficientStorage-OfStrings`，会花多长时间呢？这两个词的外形实在太相似了。

以同样的方式拼写出同样的概念才是信息。拼写前后不一致就是误导。我们很享受现代Java编程环境的自动代码完成特性。键入某个名称的前几个字母，按一下某个热键组合（如果有的话），就能得到一列该名称的可能形式。假如相似的名称依字母顺序放在一起，且差异很明显，那就会相当有助益，因为程序员多半会压根不看你的详细注释，甚至不看该类的方法列表就直接看名字挑一个对象。

误导性名称真正可怕的例子，是用小写字母l和大写字母O作为变量名，尤其是在组合使用的时候。当然，问题在于它们看起来完全像是常量“壹”和“零”。

```
int a = l;
if (O == l)
  a = O1;
else
  l = 01;

```

读者可能会认为这纯属虚构，但我们确曾见过充斥这类名称的代码。有一次，代码作者建议用不同字体写变量名，好显得更清楚些，但前提是这种方案得要通过口头和书面传递给未来所有的开发者才行。后来，只是做了简单的重命名操作，就解决了问题，而且也没引起别的问题。

## 2.4　做有意义的区分

如果程序员只是为满足编译器或解释器的需要而写代码，就会制造麻烦。例如，因为同一作用范围内两样不同的东西不能重名，你可能会随手改掉其中一个的名称，有时干脆以错误的拼写充数，结果就会出现在更正拼写错误后导致编译器出错的情况。

光是添加数字系列或是废话远远不够，即便这足以让编译器满意。如果名称必须相异，那么其意思也应该不同才对。

以数字系列命名（`a1`、`a2`…`aN`）是依义命名的对立面。这样的名称纯属误导——完全没有提供正确信息，没有提供导向作者意图的线索。试看：

```
public static void copyChars(char a1[], char a2[]) {
  for (int i = 0; i < a1.length; i++) {
    a2[i] = a1[i];
  }
}

```

如果参数名改为`source`和`destination`，这个函数就会像样许多。

废话是另一种没意义的区分。假设你有一个`Product`类，如果还有一个名为`ProductInfo`或`ProductData`的类，那它们的名称虽然不同，意思却无区别。`Info`和`Data`就像`a`、`an`和`the`一样，是意义含混的废话。

注意，只要体现出有意义的区分，使用`a`和`the`这样的前缀就没错。例如，你可能把`a`用在域内变量，而把`the`用于函数参数。但如果你已经有一个名为`zork`的变量，又想调用一个名为`theZork`的变量，麻烦就来了。

废话都是冗余。`variable`一词永远不应当出现在变量名中。`table`一词永远不应当出现在表名中。`NameString`会比`Name`好吗？难道`Name`会是一个浮点数？如果是这样，就违反了关于误导的规则。设想有一个名为`Customer`的类，还有一个名为`CustomerObject`的类，它们的区别何在呢？哪一个是表示客户历史支付情况的最佳方式？

有一个应用反映了这种状况。为当事者讳，我们改了一下，不过犯错的代码的确就是这个样子：

```
getActiveAccount(); 
getActiveAccounts(); 
getActiveAccountInfo();

```

程序员怎么知道该调用哪个函数呢？

如果缺少明确约定，那么变量`moneyAmount`与`money`就没区别，`customerInfo`与`customer`没区别，`accountData`与`account`没区别，`theMessage`也与`message`没区别。要区分名称，就要以读者能鉴别不同之处的方式来区分。

## 2.5　使用读得出来的名称

人类长于记忆和使用单词。大脑的相当一部分就是用来容纳和处理单词的。单词能读得出来。人类的大脑中有那么大的一块地方用来处理言语，若不善加利用，实在是种耻辱。

如果名称读不出来，讨论的时候就会像个傻鸟。“哎，这儿，鼻涕阿三喜摁踢（bee cee arr three cee enn tee）上头，有个皮挨死极翘（pee ess zee kyew）整数，看见没？”这不是小事，因为编程本就是一种社会活动。

有一家公司，程序里面写了一个`genymdhms`（生成日期，年、月、日、时、分、秒），他们一般读作“gen why emm dee aich emm ess”。我有见字照拼读的恶习，于是开口就念“gen-yah-mudda-hims”。后来好些设计师和分析师都有样学样，听起来傻乎乎的。我们知道典故，所以会觉得很搞笑。搞笑归搞笑，实际是在强忍糟糕的命名。在给新开发者解释变量名的意义时，他们总是读出傻乎乎的自造词，而非恰当的英语词。比较

```
class DtaRcrd102 {
  private Date genymdhms;
  private Date modymdhms;
  private final String pszqint = "102";
  /*  ...  */
};

```

和

```
class Customer {
  private Date generationTimestamp; 
  private Date modificationTimestamp;
  private final String recordId = "102";
  /*  ...  */
};

```

现在读起来就像人话了：“喂，Mikey，看看这条记录！生成时间戳（generation timestamp）被设置为明天了！不能这样吧？”

## 2.6　使用可搜索的名称

对于单字母名称和数字常量，有一个问题，就是很难在一大篇文字中找出来。

找`MAX_CLASSES_PER_STUDENT`很容易，但想找数字7就麻烦了，它可能是某些文件名或其他常量定义的一部分，出现在因不同意图而采用的各种表达式中。如果该常量是个长数字，又被人错改过，就会逃过搜索，从而造成错误。

同样，`e`也不是一个便于搜索的好变量名，它是英文中最常用的字母，在每个程序、每段代码中都有可能出现。由此而见，长名称胜于短名称，搜得到的名称胜于用自造编码代写就的名称。

窃以为单字母名称仅用于短方法中的本地变量。名称长短应与其作用域大小相对应 [N5]。若变量或常量可能在代码中多处使用，则应赋予其便于搜索的名称。再比较

```
for (int j=0; j<34; j++) {
  s += (t[j]*4)/5;
}

```

和

```
int realDaysPerIdealDay = 4;
const int WORK_DAYS_PER_WEEK = 5;
int sum = 0;
for (int j=0; j < NUMBER_OF_TASKS; j++) {
  int realTaskDays = taskEstimate[j] * realDaysPerIdealDay;
  int realTaskWeeks = (realTaskdays / WORK_DAYS_PER_WEEK);
  sum += realTaskWeeks;
}

```

注意，上面代码中的`sum`并非特别有用的名称，不过至少搜得到它。采用能表达意图的名称，貌似拉长了函数代码，但要想想看，`WORK_DAYS_PER_WEEK`比数字5好找得多，而列表中也只剩下了体现作者意图的名称。

## 2.7　避免使用编码

编码已经太多，无谓再自找麻烦。把类型或作用域编进名称里面，徒然增加了解码的负担。没理由要求每位新人都在弄清要应付的代码之外（那算是正常的），还要再搞懂另一种编码“语言”。这对解决问题而言，纯属多余的负担。带编码的名称通常也不便发音，容易打错。

### 2.7.1　匈牙利语标记法

在往昔名称长短很重要的时代，我们毫无必要地破坏了不编码的规矩，如今后悔不迭。Fortran语言要求首字母体现出类型，导致了编码的产生。BASIC语言的早期版本只允许使用一个字母再加上一位数字。匈牙利语标记法（Hungarian Notation，HN）将这种态势愈演愈烈。

Charles Simonyi在微软任首席架构师时推广了这种命名法。Simonyi是匈牙利人，在匈牙利语中，姓置于名前。匈牙利语命名法明确写出类型，并将类型置于实际名称前，以大写字母间隔，就像匈牙利语姓名的排列一般。例如，bBusy代表一个类型为boolean、名称为Busy的变量。

在Windows的C语言API的时代，HN相当重要，那时所有名称要么是一个整数句柄，要么是一个长指针或者void指针，要不然就是string的几种实现（有不同的用途和属性）之一。那时候编译器并不做类型检查，程序员需要匈牙利语标记法来帮助自己记住类型。

现代编程语言具有更丰富的类型系统，编译器也记得并强制使用类型。而且，程序员趋向于使用更小的类、更短的方法，好让每个变量的定义都在视野范围之内。

Java程序员不需要类型编码，因为对象是强类型的，代码编辑环境已经先进到在编译开始前就能监测到类型错误的程度！所以，如今HN和其他的类型编码形式都纯属多余。它们增加了修改变量、函数或类的名称或类型的难度，它们增加了阅读代码的难度，它们制造了让编码系统误导读者的可能性。

```
PhoneNumber  phoneString;
//  name not changed when type changed！

```

### 2.7.2　成员前缀

也不必用`m_`前缀来标明成员变量。应当把类和函数做得足够小，以消除对成员前缀的需要。你应当使用某种可以高亮或用颜色标出成员的编辑环境。

```
public class Part {
  private String m_dsc; // The textual description
  void setName(String name) {
    m_dsc = name;
  }
}
--------------------------------------------------------------------------------------
public class Part { 
  String description;
  void setDescription(String description) {
    this.description = description;
  }
}

```

此外，人们会很快学会无视前缀（或后缀），而只看到名称中有意义的部分。代码读得越多，眼中就越没有前缀。最终，前缀变作了不入法眼的废料，变作了旧代码的标志物。

### 2.7.3　接口和实现

有时也会出现采用编码的特殊情形。比如，你在做一个创建形状用的抽象工厂（Abstract Factory），该工厂是一个接口，要用具体类来实现。你怎么来命名工厂和具体类呢？`IShapeFactory`和`ShapeFactory`吗？我喜欢不加修饰的接口。前导字母`I`被滥用到了说好听点儿是干扰，说难听点儿根本就是废话的程度。我不想让用户知道我给他们的是接口，而就想让他们知道那是一个`ShapeFactory`。如果在接口和实现中必须选其一来编码的话，我宁肯选择实现。`ShapeFactoryImp`，甚至是丑陋的`CShapeFactory`，都比对接口名称编码好。

## 2.8　避免思维映射

不应当让读者在脑中把你的名称翻译为他们熟知的名称，这种问题经常出现在选择是使用问题领域术语还是解决方案领域术语时。

单字母变量名就是个问题。在作用域较小、没有名称冲突时，循环计数器自然有可能被命名为`i`、`j`或`k`。（但千万别用字母`l`！）这是因为传统上惯用单字母名称做循环计数器。然而，在多数其他情况下，单字母名称不是个好选择；读者必须在脑中将它映射为真实概念。仅仅是因为有了`a`和`b`，就要起名为`c`，这实在不是像样的理由。

程序员通常都是聪明人。聪明人有时会借脑筋急转弯炫耀其聪明。总而言之，假使你记得`r`代表不包含主机名和模式（scheme）的小写字母版url的话，那你真是太聪明了。

聪明程序员和专业程序员之间的区别在于，专业程序员了解，明确是王道。专业程序员善用其能，编写其他人能理解的代码。

## 2.9　类名

类名和对象名应该是名词或名词短语，例如`Customer、WikiPage`、`Account`或`AddressParser`。应避免使用`Manager`、`Processor`、`Data`或`Info`这样的类名。类名不应当是动词。

## 2.10　方法名

方法名应当是动词或动词短语，如`postPayment`、`deletePage`或`save`。属性访问器（accessor）、修改器（mutator）和断言（predicate）应该根据其值命名，并依Javabean标准加上前缀`get`、`set`和`is`。

```
string name = employee.getName();
customer.setName("mike");
if (paycheck.isPosted())...

```

重载构造器时，使用描述了参数的静态工厂方法名。例如，

```
Complex fulcrumPoint = Complex.FromRealNumber(23.0);

```

通常好于

```
Complex fulcrumPoint = new Complex(23.0);

```

可以考虑将相应的构造器设置为`private`，强制使用这种命名手段。

## 2.11　别抖机灵

如果名称太耍宝，那就只有同作者一般有幽默感的人才能记得住，而且还是在他们记得那个笑话的时候才行。谁会知道名为`HolyHand-Grenade`的函数是用来做什么的呢？没错，这名字挺有趣，不过`DeleteItems`或许是更好的名称。宁可明确，毋为好玩。

抖机灵在代码中经常体现为使用俗话或俚语。例如，别用`whack()`来表示`kill()`。别用`eatMyShorts()`这类与文化紧密相关的笑话来表示`abort()`。

言到意到。意到言到。

## 2.12　每个概念对应一个词

给每个抽象概念选一个词，并且一以贯之。例如，使用`fetch`、`retrieve`和`get`来给在多个类中的同种方法命名。你怎么记得住是哪个类中的哪个方法呢？很悲哀，你总得记住编写库或类的公司、机构或个人，才能想得起来用的是哪个术语。否则，就得耗费大把时间浏览各个文件头及前面的代码。

Eclipse和IntelliJ之类现代编程环境提供了与环境相关的线索，比如某个对象能调用的方法列表。不过要注意，列表中通常不会给出你为函数名和参数列表编写的注释。如果参数名称来自函数声明，你就太幸运了。函数名称应当独一无二，而且要保持一致，这样你才能不借助多余的浏览就找到正确的方法。

同样，如果在同一堆代码中有`controller`，有`manager`，还有`driver`，就会令人困惑。`DeviceManager`和`Protocol Controller`之间有何根本区别？为什么不全用`controller`或`manager`呢？它们都是`Drivers`吗？这种名称，让人觉得这两个对象是不同类型的，也分属不同的类。

对于那些会用到你代码的程序员，一以贯之的命名法简直就是天降福音。

## 2.13　别用双关语

避免将同一单词用于不同目的。同一术语用于不同概念，基本上就是双关语了。如果遵循“一词一义”规则，可能在好多个类里面都会有`add`方法。只要这些`add`方法的参数列表和返回值在语义上等价，就一切顺利。

但是，可能会有人决定为“保持一致”而使用`add`这个词来命名，即便并非真的想表示这种意思。比如，在多个类中都有`add`方法，该方法通过增加或连接两个现存值来获得新值。假设要写一个新类，该类中有一个方法，把单个参数放到群集（collection）中。该把这个方法叫作`add`吗？这样做貌似和其他`add`方法保持了一致，但实际上语义却不同，应该用`insert`或`append`之类的词来命名才对。把该方法命名为`add`，就是双关语了。

代码作者应尽力写出易于理解的代码。我们想把代码写得让别人能一目了然，而不必殚精竭虑地研究。我们想要那种大众化的作者尽责写清楚的平装书模式，而不想要那种学者挖地三尺才能明白个中意义的学院派模式。

## 2.14　使用解决方案领域名称

记住，只有程序员才会读你的代码。所以，尽管用那些计算机科学（Computer Science，CS）术语、算法名、模式名、数学术语吧。依据问题所涉领域来命名可不算是聪明的做法，因为不该让协作者老是跑去问客户每个名称的含义，其实他们早该通过另一名称了解这个概念了。

对于熟悉访问者（VISITOR）模式的程序员来说，名称`AccountVisitor`富有意义。哪个程序员会不知道`JobQueue`的意思呢？程序员要做太多技术性工作，给这些事起个技术性的名称，通常是最靠谱的做法。

## 2.15　使用源自所涉问题领域的名称

如果不能用程序员熟悉的术语来给手头的工作命名，就采用从所涉问题领域而来的名称吧。至少，负责维护代码的程序员就能去请教领域专家了。

优秀的程序员和设计师，其工作之一就是分离解决方案领域和问题领域的概念。与所涉问题领域更为贴近的代码，应当采用源自问题领域的名称。

## 2.16　添加有意义的语境

很少有名称是能自我说明的——多数都不能。反之，你需要用命名良好的类、函数或名称空间来放置名称，给读者提供语境。如果没这么做，给名称添加前缀就是最后一招了。

设想你有名为`firstName`、`lastName`、`street`、`houseNumber`、`city`、`state`和`zipcode`的变量。把它们放一块儿的时候，很明显构成了一个地址。但是，假使只是在某个方法中看见孤零零的一个`state`变量呢？你会理所当然地推断那是某个地址的一部分吗？

可以添加前缀`addrFirstName`、`addrLastName`、`addrState`等，以此提供语境。至少，读者会明白这些变量是某个更大结构的一部分。当然，更好的方案是创建名为`Address`的类。这样，即便是编译器也会知道这些变量隶属某个更大的概念了。

看看代码清单2-1中的方法，以下变量是否需要更有意义的语境呢？函数名仅给出了部分语境；算法提供了剩下的部分。遍览函数后，你会知道`number`、`verb`和`pluralModifier`这3个变量是“测估”信息的一部分。不幸的是这样的语境得靠读者推断出来。第一眼看到这个方法时，这些变量的含义完全不清楚。

代码清单2-1　语境不明确的变量

```
private void printGuessStatistics(char candidate, int count) { 
  String number;
  String verb;
  String pluralModifier; 
  if (count == 0) { 
    number = "no";
    verb = "are";
    pluralModifier = "s";
  } else if (count == 1) {
    number = "1";
    verb = "is";
    pluralModifier = "";
  } else {
    number = Integer.toString(count);
    verb = "are";
    pluralModifier = "s";
  }
  String guessMessage = String.format(
    "There %s %s %s%s", verb, number, candidate, pluralModifier
  );
  print(guessMessage);
}

```

上述函数有点儿过长，变量的使用贯穿始终。要分解这个函数，需要创建一个名为`GuessStatisticsMessage`的类，把3个变量做成该类的成员字段，这样它们就在定义上变作了`GuessStatisticsMessage`的一部分。语境的增强也让算法能够通过分解为更小的函数而变得更为干净利落（如代码清单2-2所示）。

代码清单2-2　有语境的变量

```
public class GuessStatisticsMessage {
  private String number;
  private String verb;
  private String pluralModifier;

  public String make(char candidate, int count) { 
    createPluralDependentMessageParts(count); 
    return String.format(
      "There %s %s %s%s",
       verb, number, candidate, pluralModifier );
  }

  private void createPluralDependentMessageParts(int count) {
    if (count == 0) {
      thereAreNoLetters();
    } else if (count == 1) {
      thereIsOneLetter();
    } else {
      thereAreManyLetters(count);
    }
  }

  private void thereAreManyLetters(int count) {
    number = Integer.toString(count);
    verb = "are";
    pluralModifier = "s";
  }

  private void thereIsOneLetter() {
    number = "1";
    verb = "is";
    pluralModifier = "";
  }

  private void thereAreNoLetters() {
    number = "no";
    verb = "are";
    pluralModifier = "s";
  }
}

```

## 2.17　不要添加没用的语境

假若有一个名为“加油站豪华版”（Gas Station Deluxe）的应用，在其中给每个类添加`GSD`前缀就不是什么好点子。说白了，你是在和自己在用的工具过不去。输入`G`，按下自动完成键，结果会得到系统中全部类的列表，列表恨不得有一英里那么长。这样做聪明吗？为什么要搞得IDE没法帮助你？

再比如，你在`GSD`应用程序中的记账模块创建了一个表示邮件地址的类，然后给该类命名为`GSDAccountAddress`。稍后，你的客户联络应用中需要用到邮件地址，你会用`GSDAccountAddress`吗？这名字听起来没问题吗？在这17个字母里面，有10个字母纯属多余，和当前语境毫无关联。

只要短名称足够清楚，就比长名称好。别给名称添加不必要的语境。

对Address类的实体来说，`accountAddress`和`customerAddress`都是不错的名称，不过用在类名上就不太好了。`Address`是一个好类名。如果需要与邮政编码MAC地址和Web地址相区分，我会考虑使用`PostalAddress`、`MAC`和`URI`。这样的名称更为精确，而精确正是命名的要点。

## 2.18　最后的话

起好名字最难的地方在于需要良好的描述技巧和共有文化背景。与其说这是一种技术、商业或管理问题，还不如说是一种教学问题。其结果是，这个领域内的许多人都没能学会做得很好。

我们有时会担心其他开发者反对重命名。讨论一下就会知道，如果名称改得更好，那大家真的会感激你。多数时候我们并不记忆类名和方法名，而使用现代工具应对这些细节，好让自己集中精力把代码写得像词句篇章，至少像是表和数据结构（词句并非总是呈现数据的最佳手段）。改名可能会让某些人吃惊，就像你做到其他代码改善工作一样。别让这种事阻碍你前进的步伐。

不妨试试上面这些规则，看看你的代码可读性是否有所提升。如果你是在维护别人写的代码，使用重构工具来解决问题，效果会立竿见影，而且会持续下去。
