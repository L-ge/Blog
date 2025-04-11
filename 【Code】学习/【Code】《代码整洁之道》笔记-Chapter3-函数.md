# 第3章　函数

在编程的早期岁月，系统由程序和子程序组成。后来，到Fortran和PL/1的年代，系统由程序、子程序和函数组成。如今，只有函数存活下来。函数是所有程序中的第一组代码。本章将讨论如何写好函数。

请看代码清单3-1。在FitNesse中，很难找到长函数，不过我还是搜寻到一个。它不光长，而且很复杂，有大量字符串、怪异且不显见的数据类型和API。花3分钟时间，你能读懂多少？

代码清单3-1　HtmlUtil.java（FitNesse 20070619）

```
public static String testableHtml(
  PageData pageData,
  boolean includeSuiteSetup
) throws Exception {
  WikiPage wikiPage = pageData.getWikiPage();
  StringBuffer buffer = new StringBuffer();
  if (pageData.hasAttribute("Test")) {
    if (includeSuiteSetup) {
      WikiPage suiteSetup =
        PageCrawlerImpl.getInheritedPage(
                SuiteResponder.SUITE_SETUP_NAME, wikiPage
        );
      if (suiteSetup != null) {
        WikiPagePath pagePath =
          suiteSetup.getPageCrawler().getFullPath(suiteSetup);
        String pagePathName = PathParser.render(pagePath);
        buffer.append("!include -setup .")
              .append(pagePathName)
              .append("\n");
      }
    }
    WikiPage setup = 
      PageCrawlerImpl.getInheritedPage("SetUp", wikiPage);
    if (setup != null) {
      WikiPagePath setupPath =
        wikiPage.getPageCrawler().getFullPath(setup);
      String setupPathName = PathParser.render(setupPath);
      buffer.append("!include -setup .")
            .append(setupPathName)
            .append("\n");
    }
  }
  buffer.append(pageData.getContent());
  if (pageData.hasAttribute("Test")) {
    WikiPage teardown = 
      PageCrawlerImpl.getInheritedPage("TearDown", wikiPage);
    if (teardown != null) {
      WikiPagePath tearDownPath =
        wikiPage.getPageCrawler().getFullPath(teardown);
      String tearDownPathName = PathParser.render(tearDownPath);
      buffer.append("\n")
            .append("!include -teardown .")
            .append(tearDownPathName)
            .append("\n");
    }
    if (includeSuiteSetup) {
      WikiPage suiteTeardown =
        PageCrawlerImpl.getInheritedPage(
                SuiteResponder.SUITE_TEARDOWN_NAME,
                wikiPage
        );
      if (suiteTeardown != null) {
        WikiPagePath pagePath =
          suiteTeardown.getPageCrawler().getFullPath (suiteTeardown);
        String pagePathName = PathParser.render(pagePath);
        buffer.append("!include -teardown .")
              .append(pagePathName)
              .append("\n");
      }
    }
  }
  pageData.setContent(buffer.toString());
  return pageData.getHtml();
}

```

读懂这个函数了吗？大概没有。有太多事发生，有太多不同层级的抽象。奇怪的字符串和函数调用，混以双重嵌套、用标识来控制的`if`语句等，不一而足。

不过，只要做几个简单的方法抽离和重命名操作，加上一点点重构，就能在9行代码之内解决问题（如代码清单3-2所示）。花3分钟阅读代码清单3-2，看你能理解吗？

代码清单3-2　HtmlUtil.java（重构之后）

```
public static String renderPageWithSetupsAndTeardowns(
  PageData pageData, boolean isSuite
) throws Exception {
  boolean isTestPage = pageData.hasAttribute("Test");
  if (isTestPage) {
    WikiPage testPage = pageData.getWikiPage();
    StringBuffer newPageContent = new StringBuffer();
    includeSetupPages(testPage, newPageContent, isSuite);
    newPageContent.append(pageData.getContent());
    includeTeardownPages(testPage, newPageContent, isSuite);
    pageData.setContent(newPageContent.toString());
  }

  return pageData.getHtml();
}

```

除非你正在研究FitNesse，否则就理解不了所有细节。不过，你大概能明白，该函数包含把一些设置和拆解页放入一个测试页面，再渲染为HTML的操作。如果你熟悉JUnit，或许会想到，该函数归属某个基于Web的测试框架，而且，这当然没错。从代码清单3-2中获得信息很容易，而代码清单3-1则晦涩难明。

是什么让代码清单3-2易于阅读和理解？怎么才能让函数表达其意图？该给函数赋予哪些属性，好让读者一看就明白函数是属于怎样的程序呢？

## 3.1　短小

函数的第一条规则是要短小。第二条规则是还要更短小。我没办法证明这个断言。我给不出任何研究结果来证实小函数更佳。我能说的是，几十年来，我写过各种大小不同的函数。我写过令人憎恶的长达3000行的函数，也写过许多100～300行的函数，还写过20～30行的函数。经过漫长的试错过程，经验告诉我，函数就应该短小。

在20世纪80年代，我们常说函数不该长于一屏。当然，说这话的时候，VT100屏幕只有24行、80列，而编辑器就先占去4行空间放菜单。如今，用上了精致的字体和宽大的显示器，一屏里面可以显示100行，每行能容纳150个字符。每行都不应该有150个字符那么长。函数也不该有100行那么长，20行封顶最佳。

函数到底该有多短？1991年，我去Kent Beck位于俄勒冈州的家中拜访。我们坐到一起写了一些代码。他给我看一个叫作Sparkle（火花闪耀）的有趣的Java/Swing小程序。程序在屏幕上描画电影《灰姑娘》（Cinderella）中仙女用魔棒造出的视觉效果。只要移动鼠标，光标所在处就会爆发出一团令人欣喜的火花，沿着模拟重力场滑落到窗口底部。Kent给我看代码的时候，我惊讶于其中那些函数的尺寸如此之小。我看惯了Swing程序中长度以英里计的函数，而这个程序中的每个函数都只有两行、三行或四行长。每个函数都一目了然。每个函数都只说一件事。而且，每个函数都依序把你带到下一个函数。这就是函数应该达到的短小程度！

函数应该有多短小？通常来说，应该短于代码清单3-2中的函数！代码清单3-2实在应该缩短成代码清单3-3。

代码清单3-3　HtmlUtil.java（再次重构之后）

```
public static String renderPageWithSetupsAndTeardowns(
  PageData pageData, boolean isSuite) throws Exception {
  if (isTestPage(pageData))
    includeSetupAndTeardownPages(pageData, isSuite);
  return pageData.getHtml();
}

```

### 代码块和缩进

`if`语句、`else`语句、`while`语句等，其中的代码块应该只占一行，该行大抵应该是一个函数调用语句。这样不但能保持函数短小，而且，因为块内调用的函数拥有较具说明性的名称，所以增加了文档上的价值。

这也意味着函数不应该大到足以容纳嵌套结构。所以，函数的缩进层级不该多于一层或两层。当然，这样的函数易于阅读和理解。

## 3.2　只做一件事

代码清单3-1显然想做好几件事，它创建缓冲区、获取页面、搜索继承下来的页面、渲染路径、添加神秘的字符串、生成HTML，如此等等。代码清单3-1手忙脚乱。而代码清单3-3则只做一件简单的事，即将设置和拆解功能包纳到测试页面中。

过去几十年以来，以下建议以不同形式一再出现：

#### 函数应该做一件事。做好这件事。只做这一件事。

问题在于很难知道那件该做的事是什么。代码清单3-3只做了一件事，对吧？其实也很容易看作以下是3件事：

（1）判断是否为测试页面；

（2）如果是，则容纳进设置和分拆步骤；

（3）渲染成HTML。

那件事是什么？函数是做了一件事，还是做了3件事呢？注意，这3个步骤均在该函数名下的同一抽象层上。可以用简洁的TO起头段落来描述这个函数：

TO RenderPageWithSetupsAndTeardowns, we check to see whether the page is a test page and if so, we include the setups and teardowns. In either case we render the page in HTML.

（要RenderPageWithSetupsAndTeardowns，检查页面是否为测试页，如果是测试页，就容纳进设置和分拆步骤。无论是否是测试页，都渲染成HTML。）

如果函数只是做了该函数名下同一抽象层上的步骤，则函数还是只做了一件事。编写函数毕竟是为了把较大的概念（换言之，函数的名称）拆分为另一抽象层上的一系列步骤。

代码清单3-1明显包括了多个处于不同抽象层级的步骤。显然，它所做的不止一件事。即便是代码清单3-2也有两个抽象层，这已被我们可将其缩短的能力所证明。但再将代码清单3-3做有意义的缩短却很难。可以将`if`语句拆出来做一个名为`includeSetupsAnd-TeardownsIfTestpage`的函数，但那只是重新诠释代码，并未改变抽象层级。

所以，要判断函数是否不止做了一件事，还有一个方法，就是看它是否能再拆出一个函数，该函数不仅只是单纯地重新诠释其实现[G34]。

#### 函数中的区段

请看代码清单4-7。注意，`generatePrimes`函数被切分为declarations、initializations和sieve等区段。这就是函数做事太多的明显征兆。只做一件事的函数无法被合理地切分为多个区段。

## 3.3　每个函数一个抽象层级

要确保函数只做一件事，函数中的语句就要在同一抽象层级上。一眼就能看出，代码清单3-1违反了这条规则。那里面有`getHtml()`等位于较高抽象层的概念，也有`StringpagePathName = PathParser.render(pagePath)`等位于中间抽象层的概念，还有`.append("\n")`等位于相当低的抽象层的概念。

函数中混杂不同抽象层级，往往会让人迷惑，读者可能无法判断某个表达式是基础概念还是细节。更恶劣的是，就像破损的窗户，一旦细节与基础概念混杂，更多的细节就会在函数中纠结起来。

### 自顶向下读代码：向下规则

我们想要让代码拥有自顶向下的阅读顺序。我们想要让每个函数后面都跟着位于下一抽象层级的函数，这样一来，在查看函数列表时，就能循抽象层级向下阅读了。我把这叫作向下规则。

换一种说法。我们想要这样读程序：程序就像是一系列TO起头的段落，每一段都描述当前抽象层级，并引用位于下一抽象层级的后续TO起头段落。

To include the setups and teardowns, we include setups, then we include the test page content, and then we include the teardowns.（要容纳设置和分拆步骤，就先容纳设置步骤，然后纳入测试页面内容，再纳入分拆步骤。）

To include the setups, we include the suite setup if this is a suite, then we include the regular setup.（要容纳设置步骤，如果是套件，就纳入套件设置步骤，然后再纳入普通设置步骤。）

To include the suite setup, we search the parent hierarchy for the “SuiteSetUp” page and add an include statement with the path of that page.（要容纳套件设置步骤，先搜索“SuiteSetUp”页面的上级继承关系，再添加一个包括该页面路径的语句。）

To search the parent. . . （要搜索……）

程序员往往很难学会遵循这条规则，写出只停留于一个抽象层级上的函数。尽管如此，学习这个技巧还是很重要。这是保持函数短小、确保只做一件事的要诀。让代码读起来像是一系列自顶向下的TO起头段落是保持抽象层级协调一致的有效技巧。

请看本章末尾的代码清单3-7，它展示了遵循这条原则重构的完整`testableHtml`函数。留意每个函数是如何引出下一个函数，并且如何保持在同一抽象层上的。

## 3.4　switch语句

写出短小的`switch`语句很难。即便是只有两种条件的`switch`语句也比我想要的单个代码块或函数大得多。写出只做一件事的`switch`语句也很难。`switch`天生要做N件事。不幸的是，我们总无法避开`switch`语句，不过还是能够确保每个`switch`都埋藏在较低的抽象层级，而且永远不重复。当然，我们可以利用多态来实现这一点。

请看代码清单3-4，它呈现了可能依赖雇员类型的仅仅一种操作。

代码清单3-4　Payroll.java

```
public Money calculatePay(Employee e) 
throws InvalidEmployeeType {
  switch (e.type) {
    case COMMISSIONED:
      return calculateCommissionedPay(e);
    case HOURLY:
      return calculateHourlyPay(e);
    case SALARIED:
      return calculateSalariedPay(e);
    default:
      throw new InvalidEmployeeType(e.type);
  }
}

```

该函数有好几个问题。首先，它太长，当出现新的雇员类型时，还会变得更长。其次，它明显做了不止一件事。第三，它违反了单一权责原则（Single Responsibility Principle，SRP），因为有好几个修改它的理由。第四，它违反了开放闭合原则（Open Closed Principle，OCP），因为每当添加新类型时，就必须修改该函数。不过，该函数最麻烦的可能是到处皆有类似结构的函数。例如，可能会有

```
isPayday(Employee e, Date date),

```

或

```
deliverPay(Employee e, Money pay),

```

如此等等。它们的结构都有同样的问题。

该问题的解决方案（如代码清单3-5所示）是将`switch`语句埋藏到抽象工厂底下，不让任何人看到。该工厂使用`switch`语句为`Employee`的派生物创建适当的实体，而不同的函数，如`calculatePay`、`isPayday`和`deliverPay`等，则借由`Employee`接口多态地接受派遣。

对于`switch`语句，我的规则是，如果只出现一次用于创建多态对象，而且隐藏在某个继承关系中，在系统其他部分看不到，就还能容忍[G23]。当然也要就事论事，有时我也会部分或全部违反这条规则。

代码清单3-5　Employee与工厂

```
public abstract class Employee {
  public abstract boolean isPayday();
  public abstract Money calculatePay();
  public abstract void deliverPay(Money pay);
}
-----------------
public interface EmployeeFactory {
  public Employee makeEmployee(EmployeeRecord r) throws InvalidEmployeeType;
}
-----------------
public class EmployeeFactoryImpl implements EmployeeFactory {
  public Employee makeEmployee(EmployeeRecord r) throws InvalidEmployeeType {
    switch (r.type) {
      case COMMISSIONED:
        return new CommissionedEmployee(r) ;
      case HOURLY:
        return new HourlyEmployee(r);
      case SALARIED:
        return new SalariedEmploye(r);
      default:
        throw new InvalidEmployeeType(r.type);
    }
  }
}
```

## 3.5　使用具有描述性的名称

在代码清单3-7中，我把示例函数的名称从`testableHtml`改为了`SetupTeardownIncluder.render`。这个名称好得多，因为它较好地描述了函数做的事。我也给每个私有方法起个同样具有描述性的名称，如`isTestable`或`includeSetupAndTeardownPages`。好名称的价值怎么好评都不为过。记住Ward原则：“如果每个例程都让你感到深合己意，那就是整洁代码。”要遵循这一原则，泰半工作都在于为只做一件事的小函数起个好名字。函数越短小、功能越集中，就越便于起个好名字。

别害怕长名称。长而具有描述性的名称，要比短而令人费解的名称好。长而具有描述性的名称，要比描述性的长注释好。使用某种命名约定，让函数名称中的多个单词容易阅读，然后使用这些单词给函数起个能说清其功用的名称。

别害怕花时间起名字。你当尝试不同的名称，实测其阅读效果。在Eclipse或IntelliJ等现代IDE中改名称易如反掌。使用这些IDE测试不同名称，直至找到最具有描述性的那一个为止。

选择描述性的名称能理清你关于模块的设计思路，并帮你改进之。追索好名称，往往导致对代码的改善重构。

命名方式要保持一致。使用与模块名一脉相承的短语、名词和动词给函数命名。例如，`includeSetupAndTeardownPages`、`includeSetupPages`、`includeSuiteSetupPage`和`includeSetupPage`等。这些名称使用了类似的措辞，依序讲出一个故事。实际上，假使我只给你看上述函数序列，你就会自问：“`includeTeardownPages`、`includeSuiteTeardownPage`和`includeTeardownPage`又会如何？”这就是所谓“深合己意”了。

## 3.6　函数参数

最理想的参数数量是0（零参数函数），其次是1（单参数函数），再次是2（双参数函数），应尽量避免3（三参数函数）。有足够特殊的理由才能用3个以上参数（多参数函数）——所以无论如何也不要这么做。

参数不易对付。它们带有太多概念性。所以我在代码范例中几乎不加参数。比如，以`StringBuffer`为例，我们可能不把它作为实体变量，而是当作参数来传递，那样的话，读者每次看到它都得要翻译一遍。阅读模块所讲述的故事时，`includeSetupPage()`要比`includeSetupPageInto(newPageContent)`易于理解。参数与函数名处在不同的抽象层级，它要求你了解目前并不特别重要的细节（即那个`StringBuffer`）。

从测试的角度看，参数甚至更叫人为难。想想看，要编写能确保参数的各种组合运行正常的测试用例，是多么困难的事。如果没有参数，就是小菜一碟。如果只有一个参数，也不太困难。有两个参数，问题就麻烦多了。如果参数多于两个，测试覆盖所有可能值的组合简直令人生畏。

输出参数比输入参数还要难以理解。读函数时，我们惯于认为信息通过参数输入函数，通过返回值从函数中输出。我们不太期望信息通过参数输出。所以，输出参数往往让人苦思之后才恍然大悟。

与没有参数相比，只有一个输入参数算是第二好的做法。`SetupTeardownIncluder.render(pageData)`也相当易于理解。很明显，我们将渲染`pageData`对象中的数据。

### 3.6.1　单参数函数的普遍形式

向函数传入单个参数有两种极普遍的理由。你也许会问关于那个参数的问题，就像在`boolean fileExists("MyFile")`中那样；也可能是操作该参数，将其转换为其他的东西，再输出之。例如，`InputStream fileOpen("MyFile")`把`String`类型的文件名转换为`InputStream`类型的返回值。这就是读者看到函数时所期待的东西。你应当选用较能区分这两种理由的名称，而且总在一致的上下文中使用这两种形式。

还有一种虽不那么普遍但仍极有用的单参数函数形式，那就是事件（event）。在这种形式中，有输入参数而无输出参数。程序将函数看作是一个事件，使用该参数修改系统状态，例如`void passwordAttemptFailedNtimes(int attempts)`。请小心使用这种形式。应该让读者很清楚地了解它是一个事件，谨慎地选用名称和上下文语境。

尽量避免编写不遵循这些形式的单参数函数，例如，`void includeSetupPageInto(StringBuffer pageText)`。对于转换，使用输出参数而非返回值会令人迷惑。如果函数要对输入参数进行转换操作，转换结果就该体现为返回值。实际上，`StringBuffer transform(StringBuffer in)`比`void transform(StringBuffer out)`强，即便第一种形式只简单地返回输入参数也是这样，至少，它遵循了转换的形式。

### 3.6.2　标识参数

标识参数丑陋不堪。向函数传入布尔值简直就是骇人听闻的做法。这样做，方法签名会立刻变得复杂起来，这相当于大声宣布本函数不止做一件事，即如果标识为`true`将会这样做，则标识为`false`会那样做！

在代码清单3-7中，我们别无选择，因为调用者已经传入了那个标识，而我想把重构范围限制在该函数及该函数以下的范围之内。方法调用`render(true)`对于可怜的读者来说仍然摸不着头脑。滚动屏幕，看到`render(Boolean isSuite)`，稍许有点帮助，不过仍然不够。应该把该函数一分为二：`reanderForSuite()`和`renderForSingleTest()`。

### 3.6.3　双参数函数

有两个参数的函数要比单参数函数难懂。例如，`writeField(name)`比`writeField(outputStream, name)`好懂。

尽管两种情况下意义都很清楚，但第一个只要扫一眼就能明白，更好地表达了其意义。第二个就得暂停一下才能明白，除非我们学会忽略第一个参数，而且如果这样，最终也会导致问题，因为我们根本就不该忽略任何代码，忽略掉的部分就是缺陷藏身之地。

当然，有些时候两个参数正好。例如，`Point p = new Point(0, 0);`就相当合理，因为笛卡儿点天生拥有两个参数。如果看到`new Point(0)`，我们会倍感惊讶。然而，本例中的两个参数却只是单个值的有序组成部分！而`outputStream`和`name`则既非自然的组合，也不是自然的排序。

即便是如`assertEquals(expected, actual)`这样的双参数函数也有其问题。你有多少次会搞错`actual`和`expected`的位置呢？这两个参数没有自然的顺序。`expected`在前，`actual`在后，只是一种需要学习的约定罢了。

双参数函数不算恶劣，而且你当然也会编写双参数函数。不过，你得小心，使用双参数函数要付出代价。你应该尽量利用一些机制将其转换成单参数函数。例如，可以把`writeField`方法写成`outputStream`的成员之一，从而能这样用：`outputStream.writeField(name)`。或者，也可以把`outputStream`写成当前类的成员变量，从而无须再传递它。还可以分离出类似于`FieldWriter`的新类，在其构造器中采用`outputStream`，并且包含一个`write`方法。

### 3.6.4　三参数函数

有3个参数的函数要比双参数函数难懂得多。排序、琢磨、忽略的问题都会加倍体现。建议你在写三参数函数前一定要想清楚。

例如，设想`assertEquals`有3个参数，该函数形式为`assertEquals(message, expected, actual)`。有多少次，你读到`message`时，会错以为它是`expected`呢？我就常栽在这个三参数函数上。实际上，每次我看到这里，总会绕半天圈子，最后学会了忽略`message`参数。

另外，这里有一个并不那么险恶的三参数函数：`assertEquals(1.0, amount, 0.001)`。虽然也要费点儿神，但还是值得的。得到“浮点值的等值是相对而言”的提示总是好的。

### 3.6.5　参数对象

如果函数看起来需要2个、3个或3个以上参数，就说明其中一些参数应该封装为类了。例如，下面两个声明的差别：

```
Circle makeCircle(double x, double y, double radius);
Circle makeCircle(Point center, double radius);

```

从参数创建对象，从而减少参数数量，看起来像是在作弊，但实则并非如此。当一组参数被共同传递，就像上例中的`x`和`y`那样，往往就是该有自己名称的某个概念的一部分。

### 3.6.6　参数列表

有时，我们想要向函数传入数量可变的参数。例如，`String.format`方法：

```
String.format("%s worked %.2f hours.", name, hours);

```

如果可变参数像上例中那样被同等对待，就和类型为`List`的单个参数没什么两样。这样一来，`String.format`实则是双参数函数。下列`String.format`的声明也很明显是二元的：

```
public String format(String format, Object... args)

```

同理，有可变参数的函数可能是单参数、双参数甚至三参数的。超过这个数量就可能要犯错了。

```
void monad(Integer... args);
void dyad(String name, Integer... args);
void triad(String name, int count, Integer... args);

```

### 3.6.7　动词与关键字

给函数起个好名字，能较好地解释函数的意图，以及参数的顺序和意图。对于单参数函数，函数和参数应当形成一种非常良好的动词/名词对形式。例如，`write(name)`就相当令人认可。不管这个“name”是什么，都要被“write”。更好的名称大概是`writeField(name)`，它告诉我们，“name”是一个“field”。

最后那个例子展示了函数名称的关键字（keyword）形式。使用这种形式，我们把参数的名称编码成了函数名。例如，把`assertEquals`改成`assertExpectedEqualsActual (expected, actual)`可能会好些，这大大减轻了记忆参数顺序的负担。

## 3.7　无副作用

副作用是一种谎言。函数承诺只做一件事，但还是会做其他被藏起来的事。有时，它会对自己类中的变量做出未能预期的改动。有时，它会把变量搞成向函数传递的参数或是系统全局变量。无论哪种情况，都是具有破坏性的，会导致古怪的时序性耦合及顺序依赖。

以代码清单3-6中看似无伤大雅的函数为例。该函数使用标准算法来匹配`userName`和`password`。如果匹配成功，则返回`true`，如果匹配失败，则返回`false`，但它会有副作用。你知道问题所在吗？

代码清单3-6　UserValidator.java

```
public class UserValidator {
  private Cryptographer cryptographer;

  public boolean checkPassword(String userName, String password) {
    User user = UserGateway.findByName(userName);
    if (user != User.NULL) {
      String codedPhrase = user.getPhraseEncodedByPassword();
      String phrase = cryptographer.decrypt(codedPhrase, password);
      if ("Valid Password".equals(phrase)) {
        Session.initialize();
        return true;
      }
    }
    return false;
  }
}

```

显然，副作用就在于对`Session.initialize( )`的调用。`checkPassword`函数，顾名思义，就是用来检查密码的，它的名称并未暗示它会初始化该次会话。所以，当某个误信了函数名的调用者想要检查用户有效性时，就得冒抹除现有会话数据的风险。

这一副作用造成了一次时序性耦合。也就是说，`checkPassword`只能在特定时刻调用（换言之，在初始化会话是安全的时候调用）。如果在不合适的时候调用，会话数据就有可能沉默地丢失。时序性耦合令人迷惑，特别是当它躲在副作用后面时。如果一定要时序性耦合，就应该在函数名称中说明。在本例中，可以将函数重命名为`checkPasswordAndInitializeSession`，虽然那还是违反了“只做一件事”的规则。

### 输出参数

参数多数会被自然而然地看作是函数的输入项。如果你编过好多年程序，我担保你一定被用作输出而非输入的参数迷惑过。例如：

```
appendFooter(s);

```

这个函数是把`s`添加到什么东西后面吗？或者它把什么东西添加到了`s`后面？`s`是输入参数还是输出参数？稍许花点时间看看函数签名：

```
public void appendFooter(StringBuffer report)

```

事情弄清楚了，但付出了检查函数声明的代价。如果你被迫检查函数签名，就得花上一点儿时间。应该避免这种中断思路的事。

在面向对象编程出现之前的岁月里，有时的确需要输出参数。然而，面向对象语言中对输出参数的大部分需求已经消失了，因为`this`也有输出函数的意味。换言之，最好是这样调用`appendFooter`：

```
report.appendFooter();
```

普遍而言，应避免使用输出参数。如果函数必须要修改某种状态，就修改所属对象的状态吧。

## 3.8　分隔指令与询问

函数要么做什么事，要么回答什么事，但二者不可得兼。函数应该修改某对象的状态，或是返回该对象的有关信息。如果两样都干，常会导致混乱。看看下面的例子：

```
public boolean set(String attribute, String value);
```

该函数设置某个指定属性，如果成功，就返回`true`，如果不存在那个属性，就返回`false`。这样就导致了以下语句：

```
if (set("username", "unclebob"))...
```

从读者的角度考虑一下吧。这是什么意思呢？它是在问`username`属性值是否之前已设置为`unclebob`，还是在问`username`属性值是否成功设置为`unclebob`呢？从该行调用语句很难判断其含义，因为`set`是动词还是形容词并不清楚。

作者本意是，`set`是一个动词，但在`if`语句的上下文中，感觉它像是一个形容词。该语句读起来像是在说“如果`username`属性值之前已被设置为`unclebob`”，而不是“设置`username`属性值为`unclebob`，看看是否可行，然后……”。要解决这个问题，可以将`set`函数重命名为`setAndCheckIfExists`，但这对提高`if`语句的可读性帮助不大。真正的解决方案是把指令与询问分隔开来，防止混淆的产生：

```
if (attributeExists("username")) {
  setAttribute("username", "unclebob");
  ...
}
```

## 3.9　使用异常替代返回错误码

从指令式函数返回错误码略微违反了指令与询问分隔的规则。它鼓励了在`if`语句判断中把指令当作表达式使用。

```
if (deletePage(page) == E_OK)
```

这不会引起动词/形容词混淆，但会导致更深层次的嵌套结构。当返回错误码时，就是在要求调用者立刻处理错误。

```
if (deletePage(page) == E_OK) {
  if (registry.deleteReference(page.name) == E_OK) {
    if (configKeys.deleteKey(page.name.makeKey()) == E_OK){
      logger.log("page deleted");
    } else {
      logger.log("configKey not deleted");
    }
  } else {
    logger.log("deleteReference from registry failed");
  }
} else {
  logger.log("delete failed");
  return E_ERROR;
}
```

另外，如果使用异常替代返回错误码，错误处理代码就能从主路径代码中分离出来，从而得到简化：

```
try {
  deletePage(page);
  registry.deleteReference(page.name);
  configKeys.deleteKey(page.name.makeKey());
}
catch (Exception e) {
  logger.log(e.getMessage());
}
```

### 3.9.1　抽离try/catch代码块

`try/catch`代码块丑陋不堪。它们搞乱了代码结构，把错误处理与正常流程混为一谈。最好把`try`和`catch`代码块的主体部分抽离出来，另外形成函数。

```
public void delete(Page page) {
  try {
    deletePageAndAllReferences(page);
  }
  catch (Exception e) {
    logError(e);
  }
}

private void deletePageAndAllReferences(Page page) throws Exception {
  deletePage(page);
  registry.deleteReference(page.name);
  configKeys.deleteKey(page.name.makeKey());
}

private void logError(Exception e) {
  logger.log(e.getMessage());
}
```

在上例中，`delete`函数只与错误处理有关，很容易理解然后可以忽略。`deletePageAndAllReferences`函数只与完全删除一个`page`有关，错误处理可以忽略。有了这样美妙的区隔，代码就更易于理解和修改了。

### 3.9.2　错误处理就是一件事

函数应该只做一件事。错误处理就是一件事。因此，处理错误的函数不该做其他事。这意味着（如上例所示）如果关键字`try`在某个函数中存在，它就应该是这个函数的第一个单词，而且在`catch/finally`代码块后面也不该有其他内容。

### 3.9.3　Error.java依赖磁铁

返回错误码通常暗示某处有个类或是枚举，其定义了所有错误码。

```
public enum Error {
  OK,
  INVALID,
  NO_SUCH,
  LOCKED,
  OUT_OF_RESOURCES, 
  WAITING_FOR_EVENT;
}
```

这样的类就是一块依赖磁铁（dependency magnet），其他许多类都得导入和使用它。当`Error`枚举修改时，其他所有类都需要重新编译和部署。这对`Error`类造成了负面压力。程序员不愿增加新的错误代码，因为如果这样他们就得重新构建和部署所有东西。于是他们就复用旧的错误码，而不添加新的。

使用异常替代错误码，新异常就可以从异常类派生出来，而无须重新编译或重新部署（这也是开放闭合原则（OCP）的一个范例）。

## 3.10　别重复自己

回头仔细看看代码清单3-1，你会注意到，有个算法在`SetUp`、`SuiteSetUp`、`TearDown`和`SuiteTearDown`中总共被重复了4次。识别重复不太容易，因为这4次重复与其他代码混在一起，而且也不完全一样。这样的重复还会导致问题，因为代码因此而臃肿，且当算法改变时需要修改4处地方，就会增加4次放过错误的可能性。

使用代码清单3-7中的`include`方法修正了这些重复。再读一遍那段代码，你会注意到，整个模块的可读性因为重复的消除而得到了提升。

重复可能是软件中一切邪恶的根源。许多原则与实践规则都是为控制与消除重复而创建的。例如，全部Codd数据库范式就是为消除数据重复而服务的。再想想看，面向对象编程如何将代码集中到基类，从而避免了冗余。面向方面编程（Aspect Oriented Programming）、面向组件编程（Component Oriented Programming）多少也都是消除重复的一种策略。看来，自子程序发明以来，软件开发领域的所有创新都是在不断尝试从源代码中消灭重复。

## 3.11　结构化编程

有些程序员遵循Edsger Dijkstra的结构化编程规则。Dijkstra认为，每个函数、函数中的每个代码块都应该有一个入口、一个出口。遵循这些规则，意味着在每个函数中只该有一个`return`语句，循环中不能有`break`或`continue`语句，而且永远不能有任何`goto`语句。

我们赞成结构化编程的目标和规范，但对于小函数，这些规则助益不大。只有在大函数中，这些规则才会有明显的好处。

所以，只要函数保持短小，偶尔出现的`return`、`break`或`continue`语句没有坏处，甚至比单入单出原则更具有表达力。另外，`goto`只在大函数中才有道理，所以应该尽量避免使用。

## 3.12　如何写出这样的函数

写代码和写别的东西很像。在写论文或文章时，你先想什么就写什么，然后再打磨它。初稿也许粗陋无序，你可以对其斟酌推敲，直至达到你心目中的样子。

我写函数时，一开始都冗长而复杂。有太多缩进和嵌套循环，有过长的参数列表。名称是随意起的，也会有重复的代码。不过我会配上一套单元测试，覆盖每行丑陋的代码。

然后我打磨这些代码，分解函数、修改名称、消除重复。我缩短和重新安置方法。有时我还拆解类，同时保持测试通过。

最后，遵循本章列出的规则，我组装好这些函数。

我并不从一开始就按照规则写函数。我想没人做得到。

## 3.13　小结

每个系统都是使用某种领域特定语言搭建的，而这种语言是程序员设计来描述那个系统的。函数是语言的动词，类是名词。这并非是要退回到最初设想的那种认为需求文档中的名词和动词就是系统中类和函数的可怕的旧观念。其实这是个历史更久的真理。编程艺术是且一直是语言设计的艺术。

大师级程序员把系统当作故事来讲，而不是当作程序来写。他们使用选定编程语言提供的工具构建一种更为丰富且更具表达力的语言，用来讲那个故事。那种领域特定语言的一个部分，就是描述在系统中发生的各种行为的函数层级。在一种狡猾的递归操作中，这些行为使用它们定义的与领域紧密相关的语言讲述自己那个小故事。

本章所讲述的是有关编写良好函数的机制。如果你遵循这些规则，函数就会短小、有个好名字，而且被很好地归置。不过永远别忘记，真正的目标在于讲述系统的故事，而你编写的函数必须干净利落地拼装到一起，形成一种精确而清晰的语言，帮助你讲故事。

## 3.14　SetupTeardownIncluder程序

SetupTeardownIncluder程序如代码清单3-7所示。

代码清单3-7　SetupTeardownIncluder.java

```
package fitnesse.html;

import fitnesse.responders.run.SuiteResponder;
import fitnesse.wiki.*;

public class SetupTeardownIncluder {
  private PageData pageData;
  private boolean isSuite;
  private WikiPage testPage;
  private StringBuffer newPageContent;
  private PageCrawler pageCrawler;

  public static String render(PageData pageData) throws Exception {
    return render(pageData, false);
  }

  public static String render(PageData pageData, boolean isSuite)
    throws Exception {
    return new SetupTeardownIncluder(pageData).render(isSuite);
  }

  private SetupTeardownIncluder(PageData pageData) {
    this.pageData = pageData;
    testPage = pageData.getWikiPage();
    pageCrawler = testPage.getPageCrawler();
    newPageContent = new StringBuffer();
  }

  private String render(boolean isSuite) throws Exception {
    this.isSuite = isSuite;
    if (isTestPage())
      includeSetupAndTeardownPages();
    return pageData.getHtml();
  }

  private boolean isTestPage() throws Exception {
    return pageData.hasAttribute("Test");
  }

  private void includeSetupAndTeardownPages() throws Exception {
    includeSetupPages();
    includePageContent();
    includeTeardownPages();
    updatePageContent();
  }

  private void includeSetupPages() throws Exception {
    if (isSuite)
      includeSuiteSetupPage();
    includeSetupPage();
  }

  private void includeSuiteSetupPage() throws Exception {
    include(SuiteResponder.SUITE_SETUP_NAME, "-setup");
  }

  private void includeSetupPage() throws Exception {
    include("SetUp", "-setup");
  }

  private void includePageContent() throws Exception {
    newPageContent.append(pageData.getContent());
  }

  private void includeTeardownPages() throws Exception {
    includeTeardownPage();
    if (isSuite)
      includeSuiteTeardownPage();
  }

  private void includeTeardownPage() throws Exception {
    include("TearDown", "-teardown");
  }

  private void includeSuiteTeardownPage() throws Exception {
    include(SuiteResponder.SUITE_TEARDOWN_NAME, "-teardown");
  }

  private void updatePageContent() throws Exception {
    pageData.setContent(newPageContent.toString());
  }

  private void include(String pageName, String arg) throws Exception {
    WikiPage inheritedPage = findInheritedPage(pageName);
    if (inheritedPage != null) {
      String pagePathName = getPathNameForPage(inheritedPage);
      buildIncludeDirective(pagePathName, arg);
    }
  }

  private WikiPage findInheritedPage(String pageName) throws Exception {
    return PageCrawlerImpl.getInheritedPage(pageName, testPage);
  }

  private String getPathNameForPage(WikiPage page) throws Exception {
    WikiPagePath pagePath = pageCrawler.getFullPath(page);
    return PathParser.render(pagePath);
  }

  private void buildIncludeDirective(String pagePathName, String arg) {
    newPageContent
      .append("\n!include ")
      .append(arg)
      .append(" .")
      .append(pagePathName)
      .append("\n");
  }
}
```
