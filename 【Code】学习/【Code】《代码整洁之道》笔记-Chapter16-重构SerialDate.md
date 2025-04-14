# 第16章　重构SerialDate

如果你找到JCommon类库，深入该类库，其中有个名为`org.jfree.date`的程序包。在该程序包中，有个名为`SerialDate`的类，我们即将剖析这个类。

`SerialDate`的作者是David Gilbert。David显然是一位经验丰富、能力很强的程序员。如我们将看到的，他在代码中展示了极高的专业性和原则性。无论怎么说，`SerialDate`都是“好代码”，而我将把它撕成碎片。

这并非恶意的行为，我也不认为自己比David强许多，有权对他的代码说三道四。其实，如果你看过我的代码，我敢说你也会发现好些该埋怨的东西。

不，这也并非傲慢无礼的行为。我所要做的，只是一种专业眼光的检视，不多也不少，那是我们都该坦然接受的做法。那是我们应该欢迎别人对自己做的事。只有通过这样的批评，我们才能学到东西。医生就是这样做的，飞行员就是这样做的，律师就是这样做的，我们程序员也需要学习如何这样做。

多说一句关于David Gilbert的事：David不仅是一位优秀的程序员，他还有着将代码免费呈献给社区的勇气和好心。他公开代码，让所有人都能看到，邀请大众使用并审查。做得真好！

`SerialDate`（见代码清单B-1）是一个用Java呈现日期的类。为什么在Java已经有`java.util.Date`和`java.util.Calendar`的情况下，还需要一个呈现日期的类呢？作者编写这个类，是为了解除我自己也常感到的痛苦。在开放的Javadoc（第67行）中，他很好地解释了原因。我们可以质疑他的初衷，但我的确有处理这个问题的需要，而且我也欢迎有一个关乎日期甚于关乎时间的类存在。

## 16.1　首先，让它能工作

在一个名为`SerialDateTests`的类（见代码清单B-2）中，有一些单元测试。测试都通过了，但不幸的是，快览一遍测试，发现它们并没有测试所有东西[T1]。例如，用“查找使用”搜索方法`MonthCodeToQuarter`（第356行），会发现没有被用过[F4]。因此，单元测试并没有测试这个方法。

所以，我用Clover来检查单元测试覆盖了哪些代码。Clover报告说，在`SerialDate`的185个可执行语句中，单元测试只执行了91个（约50%）[T2]。覆盖图看起来像是一床满是补丁的棉被，整个类上布满大块的未执行代码。

我的目标是完整地理解和重构这个类，如果没有好得多的测试覆盖率，就达不到目标。所以，我完全重起炉灶编写了自己的单元测试（见代码清单B-4）。

在阅读这些测试时，你可以看到，其中许多注释掉了。这些测试不能通过。它们代表了我以为`SerialDate`应该有的行为。在我重构`SerialDate`时，也将让这些测试通过。

即便有些测试被注释掉，Clover也还是会报告新的单元测试执行了185个可执行语句中的170个（92%）。这样就好多了，而且我想我们可以把这个覆盖率提高些。

前几个注释掉的测试（第23～63行）是我一厢情愿。程序并没有设计为通过这些测试，但对我来说它们代表的行为显而易见[G2]。我不太确定`StringToWeekdayCode`方法为何要写成那样，不过既然它已经在那儿，显然不该是区分大小写的。编写这些测试是区区小事[T3]，通过测试更加容易。我只修改了第259行和第263行，就能使用`equalsIgnoreCase`了。

我注释掉了第32行和第45行的测试，因为我不太明确是否应该支持tues和thurs缩写。

第153行和第154行的测试不能通过。显然，它们本该通过[G2]。我们可以轻易地修正，只要对`stringToMonthCode`作出以下修改就行，对于第163行和第213行的测试也一样。

```
457         if ((result < 1) || (result > 12)) {
              result = -1;
458           for (int i = 0; i < monthNames.length; i++) {
459             if (s.equalsIgnoreCase(shortMonthNames[i])) {
460               result = i + 1;
461               break;
462             }
463             if (s.equalsIgnoreCase(monthNames[i])) {
464               result = i + 1;
465               break;
466             }
467           }
468         }
```

第318行注释掉的测试暴露了`getFollowingDayOfWeek`方法中的一个缺陷（第672行）。2004年12月25日是周六。下一个周六是2005年1月1日。然而，运行测试时，会看到`getFollowingDayOfWeek`返回12月25日之后的周六还是12月25日。显然这不对[G3] [T1]。我们看到问题在第685行。那是个典型的边界条件错误[T5]。应该是这样：

```
685         if (baseDOW >= targetWeekday) {
```

很有意思，这个函数是之前一次修改的结果。修改记录（第43行）显示，`getPrevious- DayOfWeek`、`getFollowingDayOfWeek`和`getNearestDayOfWeek`中的“缺陷”已被修正[T6]。

测试`getNearestDayOfWeek`（第705行）的单元测试`testGetNearestDayOfWeek`（第329行）之前的版本不像现在一样没有遗漏。我添加了大量测试用例，因为初始的测试用例并没有全部通过[T6]。查看哪些测试用例被注释掉，你可以看到失败的模式，这很有启发。如果最近的日期是在未来，算法就会失败。显然存在某种边界条件错误[T5]。

Clover汇报的测试覆盖模式也很有趣[T8]。第719行根本没有执行！这意味着第718行的`if`语句总是得到`false`的结果。没错，看一眼代码就知道是这样。变量`adjust`总是为负，所以不会大于或等于4。所以，算法错了。

正确的算法如下所示：

```
int delta = targetDOW - base.getDayOfWeek();
int positiveDelta = delta + 7;
int adjust = positiveDelta % 7;
if (adjust > 3)
  adjust -= 7;

return SerialDate.addDays(adjust, base);
```

最后，只要简单地抛出`IllegalArgumentException`异常而不是从`weekInMonthToString`和`relativeToString`返回错误字符串，第417行和第429行的测试就能通过。

做出这些修改后，所有的单元测试都通过了，我确信`SerialDate`现在可以工作。是时候让它“做对”了。

## 16.2　让它做对

我们将从头到尾遍历`SerialDate`，同时加以改进。尽管在本章的讨论中你看不到这个过程，在每次做修改后，我还是要运行全部`JCommon`单元测试，包括我为`SerialDate`改进的那些单元测试。所以，后面你看到的所有修改，对于`JCommon`都是可工作的。

从第1行开始，我看到大量有关许可、版权、作者和修改历史的注释。我明白，的确有些法律事宜要说明，所以版权和许可信息应该保留。另外，修改历史是产生于19世纪60年代的古董，现今源代码控制工具可以帮我们做到这个。应该删掉修改历史[C1]。

从第61行开始的导入列表应该通过使用`java.text.*`和`java.util.*`来缩短。[J1]

Javadoc的HTML格式化工作（第67行）令我畏惧。一个源文件里面有多种语言，我有点发怵。这条注释有4种语言：Java、英文、Javadoc和html[G1]。有那么多语言，注释就很难直截了当。例如，生成Javadoc后，第71行和第72行原本很好的位置就丢失了，而且谁想在源代码中看到`<ul>`和`<li>`这样的东西呢？更好的策略可能是用`<pre>`标签把整个注释部分包围起来，这样，对于源代码的格式化只会限于Javadoc之内（更好的解决方案是让Javadoc不对注释做格式化，这样注释在代码和文档中就会是同一种样式）。

第86行是类声明。这个类为何要命名为`SerialDate`呢？Serial一词有什么妙处吗？是不是因为该类派生自`Serializable`？看来不是这样的。

别猜了，我知道为什么（或者我认为自己知道）要用Serial一词。线索就在第98行和第101行的常量`SERIAL_LOWER_BOUND`和`SERIAL_UPPER_BOUND`。更好的线索在从第830行开始的注释中。该类被命名为`SerialDate`，是因为它用“序列数”（serial number）来实现，该序列数恰好是从1899年12月30日后的天数。

对此我有两个问题。首先，术语“序列数”并不真对。可能有点诡辩，但其呈现方式却更接近相对偏移。术语“序列数”更多地用于产品版本标识，而非日期标识。我没发现这个名称特别有描述力[N1]。更有描述力的术语大概是“顺序”（ordinal）。

第二个问题更突出。名称`SerialDate`暗示了一种实现。该类是一个抽象类，没必要暗示任何有关实现的事。实际上，没理由隐藏实现！我发现这个名称放在了不正确的抽象层级上[N2]。以我之见，该类的名称应该就是简单的`Date`。

不幸的是，Java类库里面有太多名为`Date`的类了，所以这大概也不是最好的名称。因为这个类关于日期而非时间，所以我想将其命名为`Day`，但Day这个名字也在多处被滥用。最后，我选了`DayDate`作为最佳折中方案。

从现在起，我将使用术语`DayDate`。请记住，你读到的代码清单，还是用的`SerialDate`。

我理解为何`DayDate`继承自`Comparable`和`Serializable`。不过，为什么它要继承自`MonthConstants`呢？类`MonthConstants`（见代码清单B-3）只是一大堆定义了月份的静态常量。从常量类继承是Java程序员用的一种老花招，这样他们就能避免形如`MonthConstants.January`的表达式，不过这是一个坏主意[J2]。`MonthConstants`其实应该是一个枚举。

```
public abstract class DayDate implements Comparable,
                                         Serializable {
  public static enum Month {
    JANUARY(1),
    FEBRUARY(2),
    MARCH(3),
    APRIL(4),
    MAY(5),
    JUNE(6),
    JULY(7),
    AUGUST(8),
    SEPTEMBER(9),
    OCTOBER(10),
    NOVEMBER(11),
    DECEMBER(12);

    Month(int index) {
      this.index = index;
    }

    public static Month make(int monthIndex) {
      for (Month m : Month.values()) {
        if (m.index == monthIndex)
          return m;
      }
      throw new IllegalArgumentException("Invalid month index " + monthIndex);
    }
    public final int index;
  }
```

把`MonthConstants`改成枚举，导致对`DayDate`类和所有用到这个类的代码的一些修改。我花了一小时来改代码。不过，原来以`int`为月份类型的函数，现在都用上`Month`枚举元素了。这意味着我们可以去除`isValidMonthCode`方法（第326行），以及`monthCodeToQuarter`等位置的月份代码错误检查（第356行）了[G5]。

下一步，我们看到第91行的`serialVersionUID`。该变量用于控制序列号。如果我们修改了它，那么用这个软件编写的旧版本`DayDate`都将不再可用，而是返回一个`InvalidClassException`异常。如果你没有声明`serialVersionUID`变量，则编译器会自动生成一个，每次修改模块时都会得到不一样的值。我知道，所有的文档都建议手工控制这个变量，但对我来说自动控制序列号安全得多[G4]。我宁可调试`InvalidClassException`，也不愿意面对因忘记修改`serialVersionUID`引起的后续工作。所以，我要删除这个变量——至少暂时这么做[2]。

我发现第93行的注释是多余的。这正是谎言和误导信息所在之地[C2]。所以我要删掉它和它的同类。

第97行和第100行的注释有关序列数，我之前已经讨论过这个问题[C1]，它们描述的变量是`DayDate`能够描述的最早和最晚的日期。这可以搞得更清楚些[N1]。

```
public static final int EARLIEST_DATE_ORDINAL = 2;     // 1/1/1900
public static final int LATEST_DATE_ORDINAL = 2958465; // 12/31/9999
```

我不太清楚为什么`EARLIEST_DATE_ORDINAL`的值是2而不是0。在第829行的注释中有提示，说明这与用Microsoft Excel展示日期的方式有关。在`DayDate`的派生类`SpreadsheetDate`中能看得更深入（见代码清单B-5）。第71行的注释很好地描述了这个问题。

我的问题是，这看来应该与`SpreadsheetDate`有关，而与`DayDate`无关才对。所以，`EARLIEST_DATE_ORDINAL`和`LATEST_DATE_ORDINAL`实在不该属于`DayDate`，而应该移到`SpreadsheetDate`中[G6]。

的确，搜索一下代码就知道，这些变量值仅在`SpreadsheetDate`中用到，在`DayDate`中没用到，在`JCommon`框架的其他类中也没用到。所以，我将把这些变量值向下移到`SpreadsheetDate`中。

下面的两个变量，`MINIMUN_YEAR_SUPPORTED`和`MAXIMUM_YEAR_SUPPORTED`（第104行和第107行）地位尴尬。显然，如果`DayDate`是一个没有提供实现铺垫的抽象类，它就不该告知我们有关最小年份和最大年份的信息。同样，我很想把这些变量向下移到`SpreadsheetDate`中[G6]。然而，如果快速查找这些变量的使用情况，会发现另一个类也在用：`RelativeDayOfWeekRule`（见代码清单B-6）。在第177行和第178行的`getDate`函数中，它们被用来检查`getDate`的年份参数是否有效，而抽象类的用户需要得知其实现信息，这是一个矛盾。

我们要做的是既提供信息，又不污染`DayDate`。通常，我们会从派生类实体中获取实现信息。不过，我们并未向`getDate`函数传入`DayDate`的实体，反而返回了一个`DayDate`的实体，这意味着必须在某处创建这个实体。第187～205行提供了线索。`DayDate`实体是在`getPreviousDayOfWeek`、`getNearestDayOfWeek`或`getFollowingDayOfWeek`这3个函数其中之一里面创建的。看一下`DayDate`代码清单，我们看到，这些函数（第638～724行）全都返回了由`addDays`（第571行）创建的日期实体，`addDays`调用`CreateInstance`（第808行），创建出一个`SpreadsheetDate！`[G7]。

通常来说，基类不宜了解其派生类的情况。为了修正这个毛病，我们应该利用抽象工厂模式（ABSTRACT FACTORY），创建一个`DayDateFactory`，该工厂类将创建我们所需要的`DayDate`的实体，并回答有关实现的问题，例如最大和最小日期之类。

```
public abstract class DayDateFactory {
  private static DayDateFactory factory = new SpreadsheetDateFactory();
  public static void setInstance(DayDateFactory factory) {
    DayDateFactory.factory = factory;
  }

  protected abstract DayDate _makeDate(int ordinal);
  protected abstract DayDate _makeDate(int day, DayDate.Month month, int year);
  protected abstract DayDate _makeDate(int day, int month, int year);
  protected abstract DayDate _makeDate(java.util.Date date);
  protected abstract int _getMinimumYear();
  protected abstract int _getMaximumYear();

  public static DayDate makeDate(int ordinal) {
    return factory._makeDate(ordinal);
  }

  public static DayDate makeDate(int day, DayDate.Month month, int year) {
    return factory._makeDate(day, month, year);
  }

  public static DayDate makeDate(int day, int month, int year) {
    return factory._makeDate(day, month, year);
  }

  public static DayDate makeDate(java.util.Date date) {
    return factory._makeDate(date);
  }

  public static int getMinimumYear() {
    return factory._getMinimumYear();
  }

  public static int getMaximumYear() {
    return factory._getMaximumYear();
  }
}
```

该工厂类用`makeDate`方法替代了`createInstance`方法，前者的名称稍好一些[N1]。在初始状态下，它使用`SpreadsheetDateFactory`，但随时可以使用其他工厂。委托到抽象方法的静态方法混合采用了单件模式（SINGLETON）、油漆工模式和抽象工厂模式，我发现这种手段很有用。

`SpreadsheetDateFactory`看起来像这个样子：

```
public class SpreadsheetDateFactory extends DayDateFactory {
  public DayDate _makeDate(int ordinal) {
    return new SpreadsheetDate(ordinal);
  }

  public DayDate _makeDate(int day, DayDate.Month month, int year) {
    return new SpreadsheetDate(day, month, year);
  }

  public DayDate _makeDate(int day, int month, int year) {
    return new SpreadsheetDate(day, month, year);
  }

  public DayDate _makeDate(Date date) {
    final GregorianCalendar calendar = new GregorianCalendar();
    calendar.setTime(date);
    return new SpreadsheetDate(
      calendar.get(Calendar.DATE),
      DayDate.Month.make(calendar.get(Calendar.MONTH) + 1),
      calendar.get(Calendar.YEAR));
  }

  protected int _getMinimumYear() {
    return SpreadsheetDate.MINIMUM_YEAR_SUPPORTED;
  }

  protected int _getMaximumYear() {
    return SpreadsheetDate.MAXIMUM_YEAR_SUPPORTED;
  }
}
```

如你所见，我已经把变量`MINIMUM_YEAR_SUPPORTED`和`MAXIMUM_YEAR_SUPPORTED`移到了它们该在的`SpreadsheetDate`中[G6]。

`DayDate`的下一个问题是第109行的日期常量。这些常量其实应该是枚举类型[J3]。我们之前见过这种模式，不再赘述。你可以在最终的代码清单中看到。

接着，我们看到第140行中一系列以`LAST_DAY_OF_MONTH`开头的数组。首先，描述这些数组的注释全属多余[C3]，光看名称就足够了，所以我要删除这些注释。

这个数组没理由不是私有的[G8]，因为有一个静态函数`lastDayOfMonth`提供同样的数据。

下一个数组`AGGREGATE_DAYS_TO_END_OF_MONTH`更神秘一些，在`JCommon`框架中根本没用到它[G9]。所以我直接删除了。

对`LEAP_YEAR_AGGREGATE_DAYS_TO_END_OF_MONTH`也一样。

`AGGREGATE_DAYS_TO_END_OF_PRECEDING_MONTH`只在`SpreadsheetDate`中用到（第434行和第473行）。是否把它移到`SpreadsheetDate`中是一个问题。不转移的理由是，该数组并不专属于任何特定的实现[G6]，另外，实际上并不存在`SpreadsheetDate`之外的实现，所以，数组应该移到靠近其使用位置的地方[G10]。

说服我的理由是保持一致[G11]，数组应该私有，并通过类似于`julianDateOfLastDayOfMonth`这样的函数来暴露。看来没人需要那样的函数。而且，如果有新的`DayDate`实现需要该数组，可以轻易地把它移回到`DayDate`中去。所以我就把它移到`SpreadsheetDate`里面了。

对`LEAP_YEAR_AGGREGATE_DAYS_TO_END_OF_MONTH`也一样。

接着，我们看到3组可以转换为枚举的常量（第162～205行）。第一个用来选择月份中的一周，我将其转换为名为`WeekInMonth`的枚举。

```
public enum WeekInMonth {
  FIRST(1), SECOND(2), THIRD(3), FOURTH(4), LAST(0);
  public final int index;

  WeekInMonth(int index) {
    this.index = index;
  }
}
```

第二组常量（第177～187行）有点麻烦。常量`INCLUDE_NONE`、`INCLUDE_FIRST`、`INCLUDE_SECOND`和`INCLUDE_BOTH`用于描述某个范围的终止日是否包含在该范围之内。数学上，用术语“开放区间”“半开放区间”和“闭合区间”来表示。我想，用数学术语来命名会更清晰[N3]，所以就将其转换为枚举`DateInterval`，其中包括`CLOSED`、`CLOSED_LEFT`、`CLOSED_RIGHT`和`OPEN`枚举元素。

第三组常量（第189～205行）描述了是否该在最后、下一个或最近的日期实体中呈现对某个星期中的特定一天的查找结果。怎么命名是一个难题。最终，我给`WeekdayRange`设定了`LAST`、`NEXT`和`NEAREST`枚举元素。

你也许不会同意我起的名字。对我而言这些名字有意义，但对你可能不然。要点是它们眼下变成了易于修改的形式[J3]，不再以整数形式传递，而是作为符号传递。我可以用IDE的“修改名称”功能来改动名称或类型，无须担忧漏掉代码中某处`−1`或`2`之类的数字，也不必担忧某些int参数声明处于描述不佳的状态。

第208行的描述字段看来没有任何地方用到，我把它及其取值器和赋值器都删掉了。

我还删除了第213行的默认构造器[G12]，编译器会为我们自动生成的。

略过`isValidWeekdayCode`方法（第216～238行），在创建`Day`枚举时已经把它删掉了。

于是来到`stringToWeekdayCode`方法（第242～270行）。没有方法签名增添价值的Javadoc都是废话[C3]、[G12]，唯一的价值是对返回值−1的描述。然而，我们改用了`Day`枚举，所以这条注释完全错误了[C2]。该方法现在抛出一个`IllegalArgumentException`异常，所以我删除了Javadoc。

我还删除了参数和变量声明中的全部`final`关键字，我敢说，它们毫无价值，只会混淆视听[G12]。删除这些`final`，不合乎某些成例。例如，Robert Simmons就强烈建议我们“……在代码中遍布`final`。”我不能苟同。我认为，`final`有少数的好用法，例如，偶尔使用的`final`常量，但除此之外该关键字利小于弊。我之所以这么认为，或许是因为`final`可能捕获到的那些错误类型，早已被我编写的单元测试捕获了。

我不喜欢`for`循环（第259行和第263行）中的那些`if`语句[G5]，所以我利用“`||`”操作符把它们连接为单个`if`语句。我还使用`Day`枚举整理`for`循环，做了一些装饰性的修改。

我认为，这个方法并不真属于`DayDate`类，它其实是`Day`的一个解析函数。所以，我将它移到`Day`枚举中。不过，那样`Day`枚举就会变得太大。因为`Day`的概念并不依赖`DayDate`，所以我把`Day`枚举移到`DayDate`类之外，放到它自己的源代码文件中。

我还把下一个函数`weekdayCodeToString`（第272～286行），移植到`Day`枚举中，称其为`toString`。

```
public enum Day {
  MONDAY(Calendar.MONDAY),
  TUESDAY(Calendar.TUESDAY),
  WEDNESDAY(Calendar.WEDNESDAY),
  THURSDAY(Calendar.THURSDAY),
  FRIDAY(Calendar.FRIDAY),
  SATURDAY(Calendar.SATURDAY),
  SUNDAY(Calendar.SUNDAY);

  public final int index;
  private static DateFormatSymbols dateSymbols = new DateFormatSymbols();

  Day(int day) {
    index = day;
  }

  public static Day make(int index) throws IllegalArgumentException {
    for (Day d : Day.values())
      if (d.index == index)
        return d;
    throw new IllegalArgumentException(
      String.format("Illegal day index: %d.", index));
  }

  public static Day parse(String s) throws IllegalArgumentException {
    String[] shortWeekdayNames =
      dateSymbols.getShortWeekdays();
    String[] weekDayNames =
      dateSymbols.getWeekdays();

    s = s.trim();
    for (Day day : Day.values()) {
      if (s.equalsIgnoreCase(shortWeekdayNames[day.index]) ||
          s.equalsIgnoreCase(weekDayNames[day.index])) {
        return day;
      }
    }
    throw new IllegalArgumentException(
      String.format("%s is not a valid weekday string", s));
  }

  public String toString() {
    return dateSymbols.getWeekdays()[index];
  }
}
```

有两个`getMonth`函数（第288～316行）。第一个函数调用第二个函数。第二个函数只被第一个函数调用。所以，我把这两个函数合二为一，而且极大地简化之[G9][G12][F4]。最后，我把名称修改得更具描述力[N1]。

```
public static String[] getMonthNames() {
  return dateFormatSymbols.getMonths();
}
```

由于有了`Month`枚举，函数`isValidMonthCode`（第326～346行）就变得没什么用，因此我把它删除了[G9]。

函数`monthCodeToQuarter`（第356～375行）有特性依恋（FEATURE ENVY）的味道，可以是`Month`枚举中的一个名为`quarter`的方法，我就这么办了。

```
public int quarter() {
  return 1 + (index-1)/3;
}
```

这样一来，`Month`枚举就大到需要放到自己的类中了。我把它从`DayDate`中移出来，与`Day`枚举保持一致[G11][G13]。

后面两个方法被命名为`monthCodeToString`（第377～426行）。我们再次看到其中一个方法使用标识调用其兄弟方法的模式。将标识作为参数传递给函数的做法通常不太好，尤其是当该标识只是有关其输出格式时[G15]。我重命名、简化、重新构架了这些函数，并把它们移到`Month`枚举中[N1][N3][G14]。

```
public String toString() {
  return dateFormatSymbols.getMonths()[index - 1];
}

public String toShortString() {
  return dateFormatSymbols.getShortMonths()[index - 1];
}
```

下一个方法是`stringToMonthCode`（第428～472行）。我重新为它命名，转移到`Month`枚举中，并且简化之[N1][N3][C3][G14][G12]。

```
public static Month parse(String s) {
  s = s.trim();
  for (Month m : Month.values())
    if (m.matches(s))
      return m;

  try {
    return make(Integer.parseInt(s));
  }
  catch (NumberFormatException e) {}
  throw new IllegalArgumentException("Invalid month " + s);
}

private boolean matches(String s) {
  return s.equalsIgnoreCase(toString()) ||
         s.equalsIgnoreCase(toShortString());
}
```

方法`isLeapYear`（第495～517行）可以写得更具表达力一些[G16]。

```
public static boolean isLeapYear(int year) {
  boolean fourth = year % 4 == 0;
  boolean hundredth = year % 100 == 0;
  boolean fourHundredth = year % 400 == 0;
  return fourth && (!hundredth || fourHundredth);
}
```

下一个函数`leapYearCount`（第519～536行）并不真属于`DayDate`。除了`SpreadsheetDate`中的两个方法，没有其他调用者，所以我将它往下放。

函数`lastDayOfMonth`（第538～560行）使用了`LAST_DAY_OF_MONTH`数组，该数组应该隶属于`Month`枚举[G17]，所以我就把它移到那儿去了。我还简化了这个函数，使其更具表达力[G16]。

```
public static int lastDayOfMonth(Month month, int year) {
  if (month == Month.FEBRUARY && isLeapYear(year))
    return month.lastDay() + 1;
  else
    return month.lastDay();
}
```

现在，事情变得比较有趣了。下一个函数是`addDays`（第562～576行）。首先，由于该函数对`DayDate`的变量进行操作，它就不该是静态的[G18]，因此，我把它修改为实体方法。其次，它调用了函数`toSerial`，这个函数应该重新命名为`toOrdinal`[N1]。最后，该方法可以简化。

```
public DayDate addDays(int days) {
  return DayDateFactory.makeDate(toOrdinal() + days);
}
```

对于`addMonth`（第578～602行）也一样。它应该是一个实体方法[G18]，算法过于复杂，所以我利用解释临时变量模式（EXPLAINING TEMPORARY VARIABLES）来使其更为透明。我还将方法`getYYY`重命名为`getYear`[N1]。

```
public DayDate addMonths(int months) {
  int thisMonthAsOrdinal = 12 * getYear() + getMonth().index - 1;
  int resultMonthAsOrdinal = thisMonthAsOrdinal + months;
  int resultYear = resultMonthAsOrdinal / 12;
  Month resultMonth = Month.make(resultMonthAsOrdinal % 12 + 1);
  int lastDayOfResultMonth = lastDayOfMonth(resultMonth, resultYear);
  int resultDay = Math.min(getDayOfMonth(), lastDayOfResultMonth);
  return DayDateFactory.makeDate(resultDay, resultMonth, resultYear);
}
```

对函数`addYear`（第604～626行）也照方办理。

```
public DayDate plusYears(int years) {
  int resultYear = getYear() + years;
  int lastDayOfMonthInResultYear = lastDayOfMonth(getMonth(), resultYear);
  int resultDay = Math.min(getDayOfMonth(), lastDayOfMonthInResultYear);
  return DayDateFactory.makeDate(resultDay, getMonth(), resultYear);
}
```

把这些方法从静态方法变为实体方法，让我有点心头发痒。用`date.addDays(5)`这样的表达方法，是不是明确地表示`date`对象并没变动，以及返回了一个`DayDate`的新实体呢？或者，它只是错误地暗示我们往`date`对象添加了5天呢？你可能不会认为这是一个大问题，但下列代码可能会有欺骗性。

```
DayDate date = DateFactory.makeDate(5, Month.DECEMBER, 1952);
date.addDays(7); // bump date by one week
```

有些读到这段代码的人会认为`addDays`在修改`date`对象。所以，我们需要消除这种歧义[N4]。我把名称改为`plusDays`和`plusMonths`。我认为，方法的初衷很清楚地被

```
DayDate date = oldDate.plusDays(5);
```

所体现，不过下列代码对认为`date`对象被修改的读者来说，看起来并不那么顺畅：

```
date.plusDays(5);
```

算法越来越有趣，`getPreviousDayOfWeek`（第628～660行）可以工作，不过有点复杂了。经过一番思考，了解到它的功能后[G21]，我就能够使用解释临时变量模式来简化它[G19]，使其更为清晰。我还将它从静态方法改为实体方法[G18]，并删除了重复的实体方法[G5]（第997～1008行）。

```
public DayDate getPreviousDayOfWeek(Day targetDayOfWeek) {
  int offsetToTarget = targetDayOfWeek.index - getDayOfWeek().index;
  if (offsetToTarget >= 0)
    offsetToTarget -= 7;
  return plusDays(offsetToTarget);
}
```

对`getFollowingDayOfWeek`（第662～693行）也如法炮制：

```
public DayDate getFollowingDayOfWeek(Day targetDayOfWeek) {
  int offsetToTarget = targetDayOfWeek.index - getDayOfWeek().index;
  if (offsetToTarget <= 0)
    offsetToTarget += 7;
  return plusDays(offsetToTarget);
}
```

下一个函数是我们之前修改过的`getNearestDayOfWeek`（第695～726行）。我之前所做的修改和前两个函数没有保持一致[G11]，所以我将它改为和这两个函数保持一致，并且使用解释临时变量模式[G19]来阐明算法。

```
public DayDate getNearestDayOfWeek(final Day targetDay) {
  int offsetToThisWeeksTarget = targetDay.index - getDayOfWeek().index;
  int offsetToFutureTarget = (offsetToThisWeeksTarget + 7) % 7;
  int offsetToPreviousTarget = offsetToFutureTarget - 7;

  if (offsetToFutureTarget > 3)
    return plusDays(offsetToPreviousTarget);
  else
    return plusDays(offsetToFutureTarget);
}
```

方法`getEndOfCurrentMonth`（第728～740行）有点奇怪，因为它获取了`DayDate`参数，从而成为一个依恋[G14]其自身类的实体方法。我将其改为真正的实体方法，并修改了几个名称。

```
public DayDate getEndOfMonth() {
  Month month = getMonth();
  int year = getYear();
  int lastDay = lastDayOfMonth(month, year);
  return DayDateFactory.makeDate(lastDay, month, year);
}
```

重构`weekInMonthToString`（第742～761行）的过程非常有趣。利用IDE的重构工具，我先将其移到我之前创建的`WeekInMonth`枚举中，再将其重命名为`toString`。接着，我把它从静态方法改为实体方法。所有的测试都通过了。（你能猜出来我打算做什么吗？）

接下来，我删掉了整个方法！有5个断言失败了（第411～415行，见代码清单B-4）。我改动了这些代码行，让它们使用枚举元素的名称（`FIRST`、`SECOND`……）。全部测试都通过了。你知道为什么吗？你是否知道为什么这些步骤都是必要的吗？重构工具确保之前对`weekInMonthToString`方法的调用现在都调用`weekInMonth`枚举元素的`toString`方法，全部枚举元素都以返回其名称的形式实现了`toString`方法……

我不幸有点聪明过头了。经过这一套美妙的重构，我终于意识到，这个函数的唯一调用者，就是我刚修改的测试，所以我删除了这些测试。

愚我一次，是你之耻。愚我两次，是我之耻！所以，在判定除测试之外没有人调用过`relativeToString`（第765～781行）后，我就删除了该函数及其测试。

我最后将其改为这个抽象类的抽象方法。第一个函数保持了原样：`toSerial`（第838～844行），前文我曾把其名称改为`toOrdinal`，以现在的情形看，我决定把名称改为`getOrdinalDay`。

下一个抽象方法是`toDate`（第838～844行）。它将`DayDate`转换为`java.util.Date`。这个方法为何是抽象的？查看其在`SpreadsheetDate`中的实现（第198～207行，见代码清单B-5），可以看到它并不依赖该类的实现[G6]，所以，我把它往上推了。

方法`getYYYY`、`getMonth`和`getDayOfMonth`已经是抽象方法。不过，`getDayOfWeek`方法是另一个应该从`SpreadsheetDate`中提出来的方法，因为它不依赖`DayDate`之外的东西[G6]。是这样吗？

仔细阅读代码清单B-5第247行，可以发现该算法暗中依赖顺序日期的起点（换言之，第0天的星期日数）。所以，即便该方法没有物理上的依赖，也不能移到DayDate中，因为它的确有逻辑上的依赖。

这样的逻辑依赖困扰了我[G22]。如果有什么东西在逻辑上依赖实现的话，也该有物理上的依赖存在。我也认为，算法本身也该有一小部分依赖实现。

所以我在`DayDate`中创建了一个名为`getDayOfWekForOrdinalZero`的抽象方法，并在`SpreadsheetDate`中实现它，返回`Day.SATURDAY`。然后我把`getDayOfWeek`上移到`DayDate`中，并调用`getOrdinalDay`和`getDayOfWeekForOrdinalZero`。

```
public Day getDayOfWeek() {
  Day startingDay = getDayOfWeekForOrdinalZero();
  int startingOffset = startingDay.index - Day.SUNDAY.index;
  return Day.make((getOrdinalDay() + startingOffset) % 7 + 1);
}
```

顺便说一句，请仔细阅读第895～899行的注释。这样的重复有必要吗？通常，我会删除这类注释。

下一个方法是`compare`（第902～913行）。同样，该抽象方法是不恰当的[G6]，我将其实现上移到`DayDate`，其名称也不足够有沟通意义[N1]。方法实际上返回的是自参数日期以来的天数，所以我把名称改为`daysSince`。我还注意到该方法没有测试，就为它编写了测试。

下面6个函数（第915～980行）全都是应该在`DayDate`中实现的抽象方法。我把它们全都从`SpreadsheetDate`中抽出来了。

最后一个函数`isInRange`（第982～995行）也需要推到上一层并重构之。那个`switch`语句有点儿丑陋[G23]，可以把那些条件判断移到`DateInterval`枚举中去。

```
public enum DateInterval {
  OPEN {
    public boolean isIn(int d, int left, int right) {
      return d > left && d < right;
    }
  },
  CLOSED_LEFT {
    public boolean isIn(int d, int left, int right) {
      return d >= left && d < right;
    }
  },
  CLOSED_RIGHT {
    public boolean isIn(int d, int left, int right) {
      return d > left && d <= right;
    }
  },
  CLOSED {
    public boolean isIn(int d, int left, int right) {
      return d >= left && d <= right;
    }
  };

  public abstract boolean isIn(int d, int left, int right);
}

public boolean isInRange(DayDate d1, DayDate d2, DateInterval interval) {
  int left = Math.min(d1.getOrdinalDay(), d2.getOrdinalDay());
  int right = Math.max(d1.getOrdinalDay(), d2.getOrdinalDay());
  return interval.isIn(getOrdinalDay(), left, right);
}
```

我们来到了`DayDate`的末尾。现在我们要从头到尾再过一次，看看整个重构过程是怎样良好执行的。

首先，开端注释过时已久，我缩短并改进了它[C2]。

然后，我把全部枚举移到它们自己的文件中[G12]。

接着，我把静态变量（`dateFormatSymbols`）和3个静态方法（`getMonthNames`、`isLeapYear`和`lastDayOfMonth`）移到名为`DateUtil`的新类中[G6]。

我把那些抽象方法上移到它们该在的顶层类中[G24]。

我把`Month.make`改为`Month.fromInt` [N1]，并如法炮制所有其他枚举。我还为全部枚举创建了`toInt()`访问器，把`index`字段改为私有。

在`plusYears`和`plusMonths`中存在一些有趣的重复[G5]，我通过抽离出名为`correctLastDayOfMonth`的新方法消解了重复，使这3个方法清晰多了。

我消除了魔术数1 [G25]，用`Month.JANUARY.toInt()`或`Day.SUNDAY.toInt()`做了恰当的替换。我在`SpreadsheetDate`上花了点儿时间，清理了一下算法。最终结果在代码清单B-7～代码清单B-16中。

有趣的是，`DayDate`的代码覆盖率降低到了84.9%！这并不是因为测试到的功能减少了，而是因为该类缩减得太多，导致少量未覆盖到的代码行拥有了更大权重。`DayDate`的53个可执行语句中有45个得到测试覆盖。未覆盖的代码行微细到不值得测试。

## 16.3　小结

我们再一次遵从了童子军军规。我们签入的代码，要比签出时整洁了一点。虽然花了点儿时间，不过很值得。测试覆盖率提升了，修改了一些缺陷，代码清晰并缩短了。后来者有望比我们更容易地应对这些代码，他们也有可能把代码整理得更干净些。
