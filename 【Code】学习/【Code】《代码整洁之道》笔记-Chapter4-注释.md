# 第4章　注释

什么也比不上放置良好的注释来得有用。什么也不会比乱七八糟的注释更有本事搞乱一个模块。什么也不会比陈旧、提供错误信息的注释更有破坏性。

注释并不像辛德勒的名单。它们并不“纯然地好”。实际上，注释最多也就是一种必需的恶。若编程语言足够有表达力，或者我们长于用这些语言来表达意图，就不那么需要注释——也许根本不需要。

注释的恰当用法是弥补我们在用代码表达意图时遭遇的失败。注意，我用了“ 失败”一词，我是说真的，注释总是一种失败。我们总无法找到不用注释就能表达自我的方法，所以总要有注释，这并不值得庆贺。

如果你发现自己需要写注释，就再想想看是否有办法翻盘，用代码来表达。每次用代码表达，你都该夸奖一下自己。每次写注释，你都该做个鬼脸，感受自己在表达能力上的失败。

我为什么要极力贬低注释？因为注释会撒谎，也不是说总是如此或有意如此，但出现得实在太频繁。注释存在的时间越久，离其所描述的代码就越远，就越来越变得全然错误。原因很简单：程序员不能坚持维护注释。

代码在变动，在演化，从这里移到那里，彼此分离、重造又合到一处。很不幸，注释并不总是随之变动—— 不能总是跟着代码走。注释常常会与其所描述的代码分隔开来，孑然飘零，越来越不准确。例如，看看以下注释以及它本来要描述的代码行变成了什么样子：

```
MockRequest request;
private final String HTTP_DATE_REGEXP = 
  "[SMTWF][a-z]{2}\\,\\s[0-9]{2}\\s[JFMASOND][a-z]{2}\\s"+
  "[0-9]{4}\\s[0-9]{2}\\:[0-9]{2}\\:[0-9]{2}\\sGMT";
private Response response;
private FitNesseContext context;
private FileResponder responder;
private Locale saveLocale;
// Example: "Tue, 02 Apr 2003 22:18:49 GMT"
```

在`HTTP_DATE_REGEXP`常量及其注释之间，有可能插入其他实体变量。

程序员应当负责将注释保持在可维护、有关联、精确的高度。我同意这种说法，但我更主张把力气用在写清楚代码上，这样可以直接保证无须编写注释。

不准确的注释要比没注释糟糕得多。它们满口胡言，它们预期的东西永不能实现，它们设定了无须也不应再遵循的旧规则。

真实只在一处地方有：代码。只有代码能忠实地告诉你它做的事。那是唯一真正准确的信息来源。所以，尽管有时也需要注释，但是我们也该多花心思尽量减少注释量。

## 4.1　注释不能美化糟糕的代码

写注释的常见动机之一是糟糕的代码的存在。我们编写一个模块，发现它令人困扰、乱七八糟。我们知道，它烂透了。我们告诉自己：“喔，最好写点注释！”不！最好是把代码弄干净！

带有少量注释的整洁而有表达力的代码，要比带有大量注释的零碎而复杂的代码像样得多。与其花时间编写解释你写出的糟糕的代码的注释，不如花时间清理那堆糟糕的代码。

## 4.2　用代码来阐述

有时，代码本身不足以解释其行为。不幸的是，许多程序员据此以为代码很少——如果有的话——能做好解释工作。这种观点纯属错误。你愿意看到这个：

```
// Check to see if the employee is eligible for full benefits 
if ((employee.flags & HOURLY_FLAG) && 
    (employee.age > 65))
```

还是下面这个？

```
if (employee.isEligibleForFullBenefits())
```

只要想上几秒，就能用代码解释你大部分的意图。很多时候，简单到只需要创建一个描述了与注释所言同一事物的函数即可。

## 4.3　好注释

有些注释是必需的，也是有利的。来看看一些我认为值得写的注释。不过要记住，唯一真正好的做法是你想办法不去写注释。

### 4.3.1　法律信息

有时，公司代码规范要求编写与法律有关的注释。例如，版权及著作权声明就是必须和有理由在每个源文件开头注释处放置的内容。

下例是我们在FitNesse项目每个源文件开头放置的标准注释。我可以很开心地说，IDE自动卷起这些注释，这样就不会显得凌乱了。

```
// Copyright (C) 2003,2004,2005 by Object Mentor, Inc. All rights reserved.
// Released under the terms of the GNU General Public License version 2 or later.
```

这类注释不应是合同或法典。只要有可能，就指向一份标准许可或其他外部文档，而不要把所有条款放到注释中。

### 4.3.2　提供信息的注释

有时，用注释来提供基本信息也有其用处。例如，以下注释解释了某个抽象方法的返回值：

```
// Returns an instance of the Responder being tested.
protected abstract Responder responderInstance();
```

这类注释有时管用，但更好的方式是尽量利用函数名称传达信息。比如，在本例中，只要把函数重新命名为`responderBeingTested`，注释就是多余的了。

下例稍好一些：

```
// format matched kk:mm:ss EEE, MMM dd, yyyy
Pattern timeMatcher = Pattern.compile(
  "\\d*:\\d*:\\d* \\w*, \\w* \\d*, \\d*");
```

在本例中，注释说明，该正则表达式意在匹配一个经由`SimpleDateFormat.format`函数利用特定格式字符串格式化的时间和日期。同样，如果把这段代码移到某个转换日期和时间格式的类中，就会更好、更清晰，而注释也就变得多此一举了。

### 4.3.3　对意图的解释

有时，注释不仅提供了有关实现的有用信息，而且还提供了某个决定后面的意图。在下例中，我们看到注释反映出来的一个有趣决定。在对比两个对象时，作者决定将他的类放置在比其他东西更高的位置。

```
public int compareTo(Object o)
{
  if(o instanceof WikiPagePath)
  {
    WikiPagePath p = (WikiPagePath) o;
    String compressedName = StringUtil.join(names, "");
    String compressedArgumentName = StringUtil.join(p.names, "");
    return compressedName.compareTo(compressedArgumentName);
  }
  return 1; // we are greater because we are the right type.
}
```

下面的例子更好。你也许不同意程序员给这个问题提供的解决方案，但至少你知道他想干什么。

```
public void testConcurrentAddWidgets() throws Exception {
  WidgetBuilder widgetBuilder = 
    new WidgetBuilder(new Class[]{BoldWidget.class});
  String text = "'''bold text'''";
  ParentWidget parent = 
    new BoldWidget(new MockWidgetRoot(), "'''bold text'''");
  AtomicBoolean failFlag = new AtomicBoolean();
  failFlag.set(false);

  //This is our best attempt to get a race condition 
  //by creating large number of threads.
  for (int i = 0; i < 25000; i++) {
    WidgetBuilderThread widgetBuilderThread = 
      new WidgetBuilderThread(widgetBuilder, text, parent, failFlag);
    Thread thread = new Thread(widgetBuilderThread);
    thread.start();
  }
  assertEquals(false, failFlag.get());
}
```

### 4.3.4　阐释

有时，注释把某些晦涩难明的参数或返回值的意义翻译为某种可读形式，也是有用的。通常，更好的方法是尽量让参数或返回值本身就足够清楚；但如果参数或返回值是某个标准库的一部分，或是你不能修改的代码，帮助阐释其含义的代码就会有用。

```
public void testCompareTo() throws Exception
{
  WikiPagePath a = PathParser.parse("PageA");
  WikiPagePath ab = PathParser.parse("PageA.PageB");
  WikiPagePath b = PathParser.parse("PageB");
  WikiPagePath aa = PathParser.parse("PageA.PageA");
  WikiPagePath bb = PathParser.parse("PageB.PageB");
  WikiPagePath ba = PathParser.parse("PageB.PageA");

  assertTrue(a.compareTo(a) == 0);    // a == a
  assertTrue(a.compareTo(b) != 0);    // a != b
  assertTrue(ab.compareTo(ab) == 0);  // ab == ab
  assertTrue(a.compareTo(b) == -1);   // a < b
  assertTrue(aa.compareTo(ab) == -1); // aa < ab
  assertTrue(ba.compareTo(bb) == -1); // ba < bb
  assertTrue(b.compareTo(a) == 1);    // b > a
  assertTrue(ab.compareTo(aa) == 1);  // ab > aa
  assertTrue(bb.compareTo(ba) == 1);  // bb > ba
}
```

当然，这也会冒阐释性注释本身就不正确的风险。回头看看上例，你会发现想要确认注释的正确性有多难。这一方面说明了阐释有多必要，另一方面也说明了它有风险。所以，在写这类注释之前，考虑一下是否还有更好的办法，然后再加倍小心地确认注释的正确性。

### 4.3.5　警示

有时，用于警示其他程序员可能会出现某种后果的注释也是有用的。例如，下面的注释解释了为什么要关闭某个特定的测试用例：

```
// Don't run unless you
// have some time to kill. 
public void _testWithReallyBigFile()
{
  writeLinesToFile(10000000);

  response.setBody(testFile);
  response.readyToSend(this);
  String responseString = output.toString();
  assertSubString("Content-Length: 1000000000", responseString);
  assertTrue(bytesSent > 1000000000);
}
```

当然，如今我们在多数情况下会利用附上恰当解释性字符串的`@Ignore`属性来关闭测试用例。比如`@Ignore("Takes too long to run"`)。但在JUnit4之前的日子里，惯常的做法是在方法名前面加上下划线。如果注释足够有说服力，就会很有用了。

这里有一个更麻烦的例子：

```
public static SimpleDateFormat makeStandardHttpDateFormat()
{
  //SimpleDateFormat is not thread safe, 
  //so we need to create each instance independently.
  SimpleDateFormat df = new SimpleDateFormat("EEE, dd MMM  yyyy HH:mm:ss z");
  df.setTimeZone(TimeZone.getTimeZone("GMT"));
  return df;
}
```

你也许会抱怨说，还会有更好的解决方法。我大概会同意。不过上面的注释绝对有道理，它能阻止某位急切的程序员以效率之名使用静态初始器。

### 4.3.6　TODO注释

有时，有理由用`//TODO`形式在源代码中放置要做的工作列表。在下例中，`TODO`注释解释了为什么该函数的实现部分无所作为，将来应该是怎样。

```
//TODO-MdM these are not needed
// We expect this to go away when we do the checkout model
protected VersionInfo makeVersion() throws Exception
{
  return null;
}
```

`TODO`是一种程序员认为应该做，但由于某些原因目前还没做的工作。它可能是要提醒删除某个不必要的特性，或者要求他人注意某个问题。它可能是恳请别人起个好名字，或者提示对依赖某个计划的事件的修改。无论`TODO`的目的如何，它都不是在系统中留下糟糕的代码的借口。

如今，大多数好IDE都提供了特别的手段来定位所有`TODO`注释，这些注释看来丢不了。你不会愿意代码因为`TODO`的存在而变成一堆垃圾，所以要定期查看，删除不再需要的`TODO`注释。

### 4.3.7　放大

注释可以用来放大某种看来不合理之物的重要性。

```
String listItemContent = match.group(3).trim();
// the trim is real important. It removes the starting 
// spaces that could cause the item to be recognized
// as another list.
new ListItemWidget(this, listItemContent, this.level + 1);
return buildList(text.substring(match.end()));
```

### 4.3.8　公共API中的Javadoc

没有什么比描述良好的公共API更有用和令人满意的了。标准Java库中的Javadoc就是一例。没有它们，写Java程序就会变得很难。

如果你在编写公共API，就该为它编写良好的Javadoc。不过要记住本章中的其他建议。就像其他注释一样，Javadoc也可能误导、不适用或者提供错误信息。

## 4.4　坏注释

大多数注释都属此类。通常，坏注释都是糟糕的代码的支撑或借口，或者是对错误决策的修正，基本上等于程序员自说自话。

### 4.4.1　喃喃自语

如果只是因为你觉得应该或者因为过程需要就添加注释，那就是无谓之举。如果你决定写注释，就要花必要的时间确保写出最好的注释。

例如，我在FitNesse中找到的这个例子，例子中的注释大概确实有用。不过，作者太着急，或者没太花心思。他的喃喃自语变成了一个谜团。

```
public void loadProperties()
{
  try
  {
    String propertiesPath = propertiesLocation + "/" + PROPERTIES_FILE;
    FileInputStream propertiesStream = new FileInputStream(propertiesPath);
    loadedProperties.load(propertiesStream);
  }
  catch(IOException e)
  {
    // No properties files means all defaults are loaded
  }
}
```

`catch`代码块中的注释是什么意思呢？显然对于作者有其意义，不过并没有好到足够的程度。很明显，如果出现`IOException`，就表示没有属性文件，在那种情况下，载入默认设置。但谁来装载默认设置呢？会在对`loadProperties.load`调用之前装载吗？抑或`loadProperties.load`捕获异常、装载默认设置、再向上传递异常以忽略它？再或`loadProperties.load`在尝试载入文件前就装载所有默认设置？还是作者只是在安慰自己别在意`catch`代码块的留空？或者——这种可能最可怕——作者是想告诉自己，将来再回过头来写装载默认设置的代码？

我们唯有检视系统其他部分的代码，弄清事情原委。任何迫使读者查看其他模块的注释，都没能与读者沟通好，不值所费。

### 4.4.2　多余的注释

代码清单4-1展示的简单函数，其头部位置的注释全属多余。读这段注释花的时间没准比读代码花的时间还要长。

代码清单4-1　waitForClose

```
// Utility method that returns when this.closed is true. Throws an exception
// if the timeout is reached.
public synchronized void waitForClose(final long timeoutMillis) 
throws Exception
{
  if(!closed)
  {
    wait(timeoutMillis);
    if(!closed)
      throw new Exception("MockResponseSender could not be closed");
  }
}
```

这段注释起了什么作用呢？它并不能比代码本身提供更多的信息。它没有证明代码的意义，也没有给出代码的意图或逻辑。读它并不比读代码更容易。事实上，它不如代码精确，误导读者接受不精确的信息，而不是正确地理解代码。它就像个自来熟的二手车贩子，满口保证你不用打开发动机盖查验。

来看看代码清单4-2中摘自Tomcat项目的无用而多余的Javadoc吧。这些注释只是一味将代码搞得含混不清，完全没有文档的价值。下面只列出了靠前面的一些代码，后续模块中还有许多类似情况。

代码清单4-2　ContainerBase.java（Tomcat）

```
public abstract class ContainerBase
  implements Container, Lifecycle, Pipeline, 
  MBeanRegistration, Serializable {

  /**
   * The processor delay for this component.
   */
  protected int backgroundProcessorDelay = -1;


  /**
   * The lifecycle event support for this component.
   */
  protected LifecycleSupport lifecycle = 
    new LifecycleSupport(this);


  /**
   * The container event listeners for this Container.
   */
  protected ArrayList listeners = new ArrayList();


  /**
   * The Loader implementation with which this Container is
   * associated.
   */
  protected Loader loader = null;


  /**
   * The Logger implementation with which this Container is    
   * associated.
   */
  protected Log logger = null;


  /**
   * Associated logger name.
   */
  protected String logName = null;


  /**
   * The Manager implementation with which this Container is 
   * associated.
   */
  protected Manager manager = null;


  /**
   * The cluster with which this Container is associated.
   */
  protected Cluster cluster = null;


  /**
   * The human-readable name of this Container.
   */
  protected String name = null;


  /**
   * The parent Container to which this Container is a child.
   */
  protected Container parent = null;


  /**
   * The parent class loader to be configured when we install a 
   * Loader.
   */
  protected ClassLoader parentClassLoader = null;


  /**
   * The Pipeline object with which this Container is 
   * associated.
   */
  protected Pipeline pipeline = new StandardPipeline(this);


  /**
   * The Realm with which this Container is associated.
   */
  protected Realm realm = null;


  /**
   * The resources DirContext object with which this Container 
   * is associated.
   */
  protected DirContext resources = null;
```

### 4.4.3　误导性注释

有时，尽管初衷可嘉，但是程序员还是会写出不够精确的注释。想想代码清单4-1中那些多余而又有误导嫌疑的注释吧。

你有没有发现那样的注释是如何误导读者的？在`this.closed`变为`true`的时候，方法并没有返回。方法只在判断到`this.closed`为`true`的时候才返回，否则，就只是等待遥遥无期的超时，然后如果判断`this.closed`还是非`true`，就抛出一个异常。

这一细微的误导信息，放在比代码本身更难阅读的注释里面，有可能导致其他程序员快活地调用这个函数，并期望在`this.closed`变为`true`时立即返回。那位可怜的程序员将会发现自己陷于调试困境之中，拼命想找出代码执行得如此之慢的原因。

### 4.4.4　循规式注释

所谓每个函数都要有Javadoc或每个变量都要有注释的规矩全然是愚蠢可笑的。这类注释徒然让代码变得散乱，满口胡言，令人迷惑不解。

例如，要求每个函数都要有Javadoc，就会得到类似代码清单4-3那样面目可憎的代码。这类废话只会搞乱代码，有可能会误导读者。

代码清单4-3

```
/**
 * 
 * @param title The title of the CD
 * @param author The author of the CD
 * @param tracks The number of tracks on the CD
 * @param durationInMinutes The duration of the CD in minutes
 */
public void addCD(String title, String author, 
                   int tracks, int durationInMinutes) {
  CD cd = new CD();
  cd.title = title;
  cd.author = author;
  cd.tracks = tracks;
  cd.duration = durationInMinutes;
  cdList.add(cd);
}
```

### 4.4.5　日志式注释

有人会在每次编辑代码时，在模块开始处添加一条注释。这类注释就像是一种记录每次修改的日志。我见过满篇尽是这类日志的代码模块。

```
* Changes (from 11-Oct-2001)
* --------------------------
* 11-Oct-2001 : Re-organised the class and moved it to new package 
*               com.jrefinery.date (DG);
* 05-Nov-2001 : Added a getDescription() method, and eliminated NotableDate 
*               class (DG);
* 12-Nov-2001 : IBD requires setDescription() method, now that NotableDate 
*               class is gone (DG);  Changed getPreviousDayOfWeek(), 
*               getFollowingDayOfWeek() and getNearestDayOfWeek() to correct 
*               bugs (DG);
* 05-Dec-2001 : Fixed bug in SpreadsheetDate class (DG);
* 29-May-2002 : Moved the month constants into a separate interface 
*               (MonthConstants) (DG);
* 27-Aug-2002 : Fixed bug in addMonths() method, thanks to N???levka Petr (DG);
* 03-Oct-2002 : Fixed errors reported by Checkstyle (DG);
* 13-Mar-2003 : Implemented Serializable (DG);
* 29-May-2003 : Fixed bug in addMonths method (DG);
* 04-Sep-2003 : Implemented Comparable.  Updated the isInRange javadocs (DG);
* 05-Jan-2005 : Fixed bug in addYears() method (1096282) (DG);
```

如果在很久以前，在模块开始处创建并维护这些记录还算有道理，因为那时我们还没有源代码控制系统可用，但是如今，这种冗长的记录只会让模块变得凌乱不堪，应当全部删除。

### 4.4.6　废话注释

有时，你会看到纯然是废话的注释。它们对于显然之事喋喋不休，毫无新意。

```
/**
*  Default constructor.
*/
protected AnnualDateRule() {
}
```

对吧？再看看这个：

```
/** The day of the month. */
 private int dayOfMonth;
```

还有这样的废话模范：

```
/**
* Returns the day of the month.
*
* @return the day of the month.
*/
public int getDayOfMonth() {
  return dayOfMonth;
}
```

这类注释废话连篇，我们都学会了视而不见。读代码时，眼光不会停留在它们上面。最终，当代码修改之后，这类注释就变作了谎言一堆。

代码清单4-4中的第一条注释貌似还行。它解释了`catch`代码块为何被忽略。不过第二条注释就纯是废话了。显然，该程序员对编写函数中那些`try/catch`代码块感到沮丧。

代码清单4-4　startSending

```
private void startSending()
{
  try
  {
    doSending();
  }
  catch(SocketException e)
  {
    // normal. someone stopped the request.
  }
  catch(Exception e)
  {
    try
    {
      response.add(ErrorResponder.makeExceptionString(e));
      response.closeAll();
    }
    catch(Exception e1)
    {
      // Give me a break!
    }
  }
}
```

程序员与其纠缠毫无价值的废话注释，不如意识到，他的挫败感可以由改进代码结构而消除。他应该把力气花在将最末一个`try/catch`代码块拆解到单独的函数中，如代码清单4-5所示。

代码清单4-5　startSending（重构之后）

```
private void startSending()
{
  try
  {
    doSending();
  }
  catch(SocketException e)
  {
    //normal. someone stopped the request.
  }
  catch(Exception e)
  {
    addExceptionAndCloseResponse(e);
  }
}

private void addExceptionAndCloseResponse(Exception e)
{
  try
  {
    response.add(ErrorResponder.makeExceptionString(e));
    response.closeAll();
  }
  catch(Exception e1)
  {
  }
}
```

用整理代码的决心替代创造废话的冲动吧，这样你就会发现自己将成为更优秀、更快乐的程序员。

### 4.4.7　可怕的废话

Javadoc也可能是废话。下列Javadoc（来自某知名开源库）的目的是什么？答案：无。它们只是源自某种提供文档的不当愿望的废话注释。

```
/** The name. */
private String name;

/** The version. */
private String version;

/** The licenceName. */
private String licenceName;

/** The version. */
private String info;
```

再仔细读读这些注释。你是否发现了剪切-粘贴错误？如果作者在写（或粘贴）注释时都没花心思，又怎么能指望读者从中获益呢？

### 4.4.8　能用函数或变量时就别用注释

看看以下代码概要：

```
// does the module from the global list <mod> depend on the
// subsystem we are part of?
if (smodule.getDependSubsystems().contains(subSysMod.getSubSystem()))
```

可以改成以下没有注释的版本：

```
ArrayList moduleDependees = smodule.getDependSubsystems(); 
String ourSubSystem = subSysMod.getSubSystem();
if (moduleDependees.contains(ourSubSystem))
```

代码原作者可能（不太像）是先写注释再编写代码的。不过，作者应该重构代码，如我所做的那样，从而删掉注释。

### 4.4.9　位置标记

有时，程序员喜欢在源代码中标记某个特别位置。例如，最近我在程序中看到这样一行：

```
// Actions //////////////////////////////////
```

把特定函数趸放在这种标记栏下面，多数时候实属无理。鸡零狗碎，理当删除——特别是尾部那一长串无用的斜杠。

这么说吧，如果标记栏不多，它就会显而易见。所以，尽量少用标记栏，只在特别有价值的时候用。如果滥用标记栏，就会沉没在背景噪音中而被忽略。

### 4.4.10　括号后面的注释

有时，程序员会在括号后面放置特殊的注释，如代码清单4-6所示。尽管这对于含有深度嵌套结构的长函数可能有意义，但只会给我们更愿意编写的短小、封装的函数带来混乱。如果你发现自己想标记右括号，其实应该做的是缩短函数。

代码清单4-6　wc.java

```
public class wc {
  public static void main(String[] args) {
    BufferedReader in = new BufferedReader(new InputStreamReader(System.in));
    String line;
    int lineCount = 0;
    int charCount = 0;
    int wordCount = 0;
    try {
      while ((line = in.readLine()) != null) {
        lineCount++;
        charCount += line.length();
        String words[] = line.split("\\W");
        wordCount += words.length;
     } //while
      System.out.println("wordCount = " + wordCount);
      System.out.println("lineCount = " + lineCount);
      System.out.println("charCount = " + charCount);
	} // try
    catch (IOException e) {
      System.err.println("Error:" + e.getMessage());
	} //catch
  } //main
}
```

### 4.4.11　归属与署名

```
/* Added by Rick  */
```

源代码控制系统非常善于记住是谁在何时添加了什么。没必要用那些小小的签名搞脏代码。你也许会认为，这种注释大概有助于他人了解应该和谁讨论这段代码。不过，事实却是注释在那儿放了一年又一年，越来越不准确，越来越和原作者没关系。

重申一下，源代码控制系统是这类信息最好的归属地。

### 4.4.12　注释掉的代码

直接把代码注释掉是讨厌的做法。别这么干！

```
 InputStreamResponse response = new InputStreamResponse();
 response.setBody(formatter.getResultStream(), formatter.getByteCount());
// InputStream resultsStream = formatter.getResultStream();
// StreamReader reader = new StreamReader(resultsStream);
// response.setContent(reader.read(formatter.getByteCount()));
```

其他人不敢删除注释掉的代码。他们会想，代码依然放在那儿，一定有其原因，而且这段代码很重要，不能删除。注释掉的代码堆积在一起，就像破酒瓶底的渣滓一般。

看看以下来自Apache公共库的代码：

```
this.bytePos = writeBytes(pngIdBytes, 0);
//hdrPos = bytePos;
writeHeader();
writeResolution();
//dataPos = bytePos;
if (writeImageData()) {
  writeEnd();
  this.pngBytes = resizeByteArray(this.pngBytes, this.maxPos);
}
else{
  this.pngBytes=null;
}
return this.pngBytes;
```

这两行代码为什么要注释掉？它们重要吗？它们搁在那儿，是为了给未来的修改做提示吗？或者，只是某人在多年以前注释掉、懒得清理的过时玩意？

20世纪60年代，曾经有一段时间，注释掉的代码可能有用。但我们已经拥有优良的源代码控制系统如此之久，这些系统可以为我们记住不要的代码。我们无须再用注释来标记，删掉即可，它们丢不了。我担保。

### 4.4.13　HTML注释

源代码注释中的HTML标记是一种厌物，如你在下面代码中所见。编辑器/IDE中的代码本来易于阅读，却因为HTML注释的存在而变得难以卒读。如果注释将由某种工具（例如Javadoc）抽取出来，呈现到网页，那么该是工具而非程序员来负责给注释加上合适的HTML标签。

```
/**
 * Task to run fit tests. 
 * This task runs fitnesse tests and publishes the results.
 * <p/>
 * <pre>
 * Usage:
 * &lt;taskdef name=&quot;execute-fitnesse-tests&quot; 
 *     classname=&quot;fitnesse.ant.ExecuteFitnesseTestsTask&quot; 
 *     classpathref=&quot;classpath&quot; /&gt;
 * OR
 * &lt;taskdef classpathref=&quot;classpath&quot; 
 *             resource=&quot;tasks.properties&quot; /&gt;
 * <p/>
 * &lt;execute-fitnesse-tests 
 *     suitepage=&quot;FitNesse.SuiteAcceptanceTests&quot; 
 *     fitnesseport=&quot;8082&quot; 
 *     resultsdir=&quot;${results.dir}&quot; 
 *     resultshtmlpage=&quot;fit-results.html&quot; 
 *     classpathref=&quot;classpath&quot; /&gt;
 * </pre>
 */
```

### 4.4.14　非本地信息

假如你一定要写注释，请确保它描述了离它最近的代码。别在本地注释的上下文环境中给出系统级的信息。以下面的Javadoc注释为例，除了可怕的冗余，它还给出了有关默认端口的信息。不过该函数完全没控制那个所谓的默认值。这个注释并未描述该函数，而是在描述系统中远在其他地方的其他函数。当然，也无法确保在包含那个默认值的代码修改之后，这里的注释也会随之修改。

```
/**
 * Port on which fitnesse would run. Defaults to <b>8082</b>.
 *
 * @param fitnessePort
 */
public void setFitnessePort(int fitnessePort)
{
  this.fitnessePort = fitnessePort;
}
```

### 4.4.15　信息过多

别在注释中添加有趣的历史性话题或者无关的细节描述。下列注释来自某个用来测试base64编解码函数的模块。除RFC文档编号之外，注释中的其他细节信息对于读者完全没有必要。

```
/*
   RFC 2045 - Multipurpose Internet Mail Extensions (MIME) 
   Part One: Format of Internet Message Bodies
   section 6.8.  Base64 Content-Transfer-Encoding
   The encoding process represents 24-bit groups of input bits as output 
   strings of 4 encoded characters. Proceeding from left to right, a 
   24-bit input group is formed by concatenating 3 8-bit input groups. 
   These 24 bits are then treated as 4 concatenated 6-bit groups, each 
   of which is translated into a single digit in the base64 alphabet. 
   When encoding a bit stream via the base64 encoding, the bit stream 
   must be presumed to be ordered with the most-significant-bit first. 
   That is, the first bit in the stream will be the high-order bit in 
   the first 8-bit byte, and the eighth bit will be the low-order bit in 
   the first 8-bit byte, and so on.
 */
```

### 4.4.16　不明显的联系

注释及其描述的代码之间的联系应该显而易见。如果你不嫌麻烦要写注释，至少让读者能看到注释和代码，并且理解注释所谈何物。

以来自Apache公共库的这段注释为例：

```
/*
 * start with an array that is big enough to hold all the pixels
 * (plus filter bytes), and an extra 200 bytes for header info
 */
this.pngBytes = new byte[((this.width + 1) * this.height * 3) + 200];
```

过滤器字节是什么？与那个`+1`有关系吗？或与`*3`有关？还是与两者皆有关？为什么用`200`？注释的作用是解释未能自行解释的代码。如果注释本身还需要解释，就太遗憾了。

### 4.4.17　函数头

短函数不需要太多描述。为只做一件事的短函数选个好名字，通常要比写函数头注释好。

### 4.4.18　非公共代码中的Javadoc

虽然Javadoc对于公共API非常有用，但对于不打算作公共用途的代码就令人厌恶了。为系统中的类和函数生成Javadoc页并非总有用，而对Javadoc注释额外的形式要求几乎等同于八股文章。

### 4.4.19　范例

我曾为首个XP Immersion课程编写了代码清单4-7列出的模块。这个模块几乎是糟糕的代码和坏注释风格的典范。后来Kent Beck当着几十位满腔热情的学生的面重构了这些代码，将其变得令人愉悦。后来，我在拙著《敏捷软件开发：原则、模式与实践》（Agile Software Development, Principles, Patterns, and Practices）和Software Development杂志的“技艺”专栏的第一篇文章中引用了这个例子。

代码清单4-7　GeneratePrimes.java

```
/**
 * This class Generates prime numbers up to a user specified
 * maximum.  The algorithm used is the Sieve of Eratosthenes.
 * <p>
 * Eratosthenes of Cyrene, b. c. 276 BC, Cyrene, Libya --
 * d. c. 194, Alexandria.  The first man to calculate the
 * circumference of the Earth.  Also known for working on
 * calendars with leap years and ran the library at Alexandria.
 * <p>
 * The algorithm is quite simple.  Given an array of integers
 * starting at 2.  Cross out all multiples of 2.  Find the next
 * uncrossed integer, and cross out all of its multiples.
 * Repeat untilyou have passed the square root of the maximum
 * value.
 *
 * @author Alphonse
 * @version 13 Feb 2002 atp
 */
import java.util.*;

public class GeneratePrimes
{
  /**
   * @param maxValue is the generation limit.
   */
  public static int[] generatePrimes(int maxValue)
  {
    if (maxValue >= 2) // the only valid case
    {
      // declarations
      int s = maxValue + 1; // size of array
      boolean[] f = new boolean[s];
      int i;
      // initialize array to true.
      for (i = 0; i < s; i++)
        f[i] = true;
      // get rid of known non-primes
      f[0] = f[1] = false;

      // sieve
      int j;
      for (i = 2; i < Math.sqrt(s) + 1; i++)
      {
        if (f[i]) // if i is uncrossed, cross its multiples.
        {
          for (j = 2 * i; j < s; j += i)
            f[j] = false; // multiple is not prime
        }
      }

      // how many primes are there?
      int count = 0;
      for (i = 0; i < s; i++)
      {
        if (f[i])
          count++; // bump count.
      }

      int[] primes = new int[count];

      // move the primes into the result
      for (i = 0, j = 0; i < s; i++)
      {
        if (f[i])             // if prime
          primes[j++] = i;
      }

      return primes;  // return the primes      
    }
    else // maxValue < 2
      return new int[0]; // return null array if bad input.
  }
}
```

这个模块最迷人的地方是，有那么一阵，我们中的许多人都认为它“文档做得很好”。如今，我们认为它是一小团乱麻。看看你能发现多少个不同的注释问题吧。

在代码清单4-8中，你可以看到该模块重构后的版本。注意，注释的使用被明显地限制了。在整个模块中只有两个注释。每个注释都足具说明意义。

代码清单4-8　PrimeGenerator.java（重构后）

```
/**
 * This class Generates prime numbers up to a user specified
 * maximum.  The algorithm used is the Sieve of Eratosthenes.
 * Given an array of integers starting at 2:
 * Find the first uncrossed integer, and cross out all its
 * multiples.  Repeat until there are no more multiples
 * in the array.
 */

public class PrimeGenerator
{
  private static boolean[] crossedOut;
  private static int[] result;

  public static int[] generatePrimes(int maxValue)
  {
    if (maxValue < 2)
      return new int[0];
    else
    {
      uncrossIntegersUpTo(maxValue);
      crossOutMultiples();
      putUncrossedIntegersIntoResult();
      return result;
    }
  }

  private static void uncrossIntegersUpTo(int maxValue)
  {
    crossedOut = new boolean[maxValue + 1];
    for (int i = 2; i < crossedOut.length; i++)
      crossedOut[i] = false;
  }

  private static void crossOutMultiples()
  {
    int limit = determineIterationLimit();
    for (int i = 2; i <= limit; i++)
      if (notCrossed(i))
        crossOutMultiplesOf(i);
  }

  private static int determineIterationLimit()
  {
    // Every multiple in the array has a prime factor that
    // is less than or equal to the root of the array size,
    // so we don't have to cross out multiples of numbers
    // larger than that root.
    double iterationLimit = Math.sqrt(crossedOut.length);
    return (int) iterationLimit;
  }

  private static void crossOutMultiplesOf(int i)
  {
    for (int multiple = 2*i;
         multiple < crossedOut.length;
         multiple += i)
      crossedOut[multiple] = true;
  }

  private static boolean notCrossed(int i)
  {
    return crossedOut[i] == false;
  }

  private static void putUncrossedIntegersIntoResult()
  {
    result = new int[numberOfUncrossedIntegers()];
    for (int j = 0, i = 2; i < crossedOut.length; i++)
      if (notCrossed(i))
        result[j++] = i;
  }

  private static int numberOfUncrossedIntegers()
  {
    int count = 0;
    for (int i = 2; i < crossedOut.length; i++)
      if (notCrossed(i))
        count++;

    return count;
  }
}
```

很容易说明，第一个注释完全是多余的，因为它读起来非常像是`generatePrimes`函数自身。不过，我认为这段注释还是省了读者去读具体算法的精力，所以我倾向于保留它。

第二个注释显然很有必要。它解释了平方根作为循环限制的理由。我找不到能说明白这个问题的简单变量名或者其他编程结构。另外，对平方根的使用可能也有点武断。通过限制平方根循环，我是否真节省了许多时间？计算平方根所花的时间会不会比省下的时间还要多？这些都值得考虑。使用平方根作为循环限制，满足了我这种旧式C语言和汇编语言黑客，不过我可不敢说抵得上其他人为理解它而花的时间和精力。
