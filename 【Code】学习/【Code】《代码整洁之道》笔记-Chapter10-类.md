# 第10章　类

到目前为止，本书一直在讨论如何编写良好的代码行和代码块。我们深入研究了函数的恰当构成，以及函数之间如何互相关联。尽管讨论了这么多关于代码语句及由代码语句形成的函数的表达力，但是，除非我们将注意力放到代码组织的更高层面，否则始终不能得到整洁的代码。

## 10.1　类的组织

遵循标准的Java约定，类应该从一组变量列表开始。如果有公共静态常量，应该先出现。然后是私有静态变量，以及私有实体变量。很少会有公共变量。

公共函数应跟在变量列表之后。我们喜欢把由某个公共函数调用的私有工具函数紧随在该公共函数后面。这符合自上向下原则，让程序读起来就像一篇报纸文章。

#### 封装

我们喜欢保持变量和工具函数的私有性，但并不执着于此。有时，我们也需要用到受保护的（protected）变量或工具函数，好让测试可以访问到。对我们来说，测试说了算。若同一程序包内的某个测试需要调用一个函数或变量，我们就会将该函数或变量置为受保护或在整个程序包内可访问。然而，我们首先会想办法使之保有隐私。放松封装总是下策。

## 10.2　类应该短小

关于类的第一条规则是类应该短小。第二条规则是还要更短小。不，我们并不是要重谈“函数”一章的论调。就像函数一样，在设计类时，首要规则就是要更短小。和函数一样，马上有个问题出现，那就是“多小合适呢？”

对于函数，我们通过计算代码行数衡量其大小。对于类，我们采用不同的衡量方法，即计算其权责（responsibility）。

代码清单10-1给出了某个类的轮廓。`SuperDashboard`类曝露大概70个公共方法，其中大多数开发者都会同意，这实在是太长了。有些开发者或许会将`SuperDashboard`类称为“神类”。

代码清单10-1　权责太多

```
public class SuperDashboard extends JFrame implements MetaDataUser 
   public String getCustomizerLanguagePath() 
   public void setSystemConfigPath(String systemConfigPath) 
   public String getSystemConfigDocument() 
   public void setSystemConfigDocument(String systemConfigDocument) 
   public boolean getGuruState() 
   public boolean getNoviceState() 
   public boolean getOpenSourceState() 
  public void showObject(MetaObject object) 
  public void showProgress(String s)
  public boolean isMetadataDirty() 
  public void setIsMetadataDirty(boolean isMetadataDirty) 
  public Component getLastFocusedComponent() 
  public void setLastFocused(Component lastFocused) 
  public void setMouseSelectState(boolean isMouseSelected)
  public boolean isMouseSelected() 
  public LanguageManager getLanguageManager() 
  public Project getProject() 
  public Project getFirstProject() 
  public Project getLastProject() 
  public String getNewProjectName()
  public void setComponentSizes(Dimension dim)
  public String getCurrentDir() 
  public void setCurrentDir(String newDir) 
  public void updateStatus(int dotPos, int markPos) 
  public Class[] getDataBaseClasses() 
  public MetadataFeeder getMetadataFeeder() 
  public void addProject(Project project)
  public boolean setCurrentProject(Project project)
  public boolean removeProject(Project project) 
  public MetaProjectHeader getProgramMetadata()
  public void resetDashboard()
  public Project loadProject(String fileName, String projectName)
  public void setCanSaveMetadata(boolean canSave)
  public MetaObject getSelectedObject() 
  public void deselectObjects() 
  public void setProject(Project project) 
  public void editorAction(String actionName, ActionEvent event)
  public void setMode(int mode) 
  public FileManager getFileManager() 
  public void setFileManager(FileManager fileManager) 
  public ConfigManager getConfigManager() 
  public void setConfigManager(ConfigManager configManager) 
  public ClassLoader getClassLoader() 
  public void setClassLoader(ClassLoader classLoader) 
  public Properties getProps() 
  public String getUserHome() 
  public String getBaseDir() 
  public int getMajorVersionNumber() 
  public int getMinorVersionNumber() 
  public int getBuildNumber() 
  public MetaObject pasting(
    MetaObject target, MetaObject pasted, MetaProject project)
  public void processMenuItems(MetaObject metaObject)
  public void processMenuSeparators(MetaObject metaObject)
  public void processTabPages(MetaObject metaObject)
  public void processPlacement(MetaObject object) 
  public void processCreateLayout(MetaObject object) 
  public void updateDisplayLayer(MetaObject object, int layerIndex)
  public void propertyEditedRepaint(MetaObject object) 
  public void processDeleteObject(MetaObject object) 
  public boolean getAttachedToDesigner() 
  public void processProjectChangedState(boolean hasProjectChang)
  public void processObjectNameChanged(MetaObject object) 
  public void runProject() 
  public void setAllowDragging(boolean allowDragging) 
  public boolean allowDragging() 
  public boolean isCustomizing() 
  public void setTitle(String title) 
  public IdeMenuBar getIdeMenuBar() 
  public void showHelper(MetaObject metaObject, String propertyName)
  // ... many non-public methods follow ...
}
```

如果`SuperDashboard`类只包括代码清单10-2中的方法呢？

代码清单10-2　足够短小了吗？

```
public class SuperDashboard extends JFrame implements MetaDataUser{
  public Component getLastFocusedComponent() 
  public void setLastFocused(Component lastFocused) 
  public int getMajorVersionNumber() 
  public int getMinorVersionNumber() 
  public int getBuildNumber() 
}
```

5个方法不算多，在这里，虽然方法数量较少，但是`SuperDashboard`还是拥有太多权责。

类的名称应当描述其权责。实际上，命名正是帮助判断类的长度的第一个手段。如果无法为某个类命以精确的名称，那么这个类大概就太长了。类名越含糊，该类越有可能拥有过多权责。例如，如果类名中包含含义模糊的词，如`Processor`、`Manager`或`Super`，那么这种现象往往说明有不恰当的权责聚集情况存在。

我们也应该能够用大概25个单词简要描述一个类，且不用“若”（if）“与”（and）“或”（or）或者“但”（but）等词汇。我们该如何描述`SuperDashboard`类呢？“`SuperDashboard`类提供了对最后拥有焦点的组件的访问能力，我们还能通过它跟踪版本号和构建序列号。”“还能”二字恰好提示了`SuperDashboard`类有太多权责。

### 10.2.1　单一权责原则

单一权责原则（SRP）认为，类或模块应该有且只有一条加以修改的理由。该原则既给出了权责的定义，又是关于类的长度的指导方针。类只应有一个权责——只有一条修改的理由。

代码清单10-2中貌似很小的`SuperDashboard`类有两条加以修改的理由。第一，它跟踪大概会随软件每次发布而更新的版本信息。第二，它管理Java Swing组件（派生自`JFrame`，顶层GUI窗口的Swing表现形态）。每次修改Swing代码时，无疑都要更新版本号，但反之未必可行：也可能依据系统中其他代码的修改而更新版本信息。

鉴别权责（修改的理由）常常帮助我们在代码中认识到并创建出更好的抽象。可以轻易地将全部3个处理版本信息的`SuperDashboard`方法拆解到名为`Version`的类中（如代码清单10-3所示）。`Version`类是个极有可能在其他应用程序中得到复用的构造！

代码清单10-3　单一权责类

```
public class Version {
  public int getMajorVersionNumber() 
  public int getMinorVersionNumber() 
  public int getBuildNumber() 
}
```

SRP是面向对象设计中最为重要的概念之一，也是较为容易理解和遵循的概念之一。奇怪的是SRP往往也是最容易被破坏的类设计原则。经常会遇到做太多事的类，为什么呢？

让软件能工作和让软件保持整洁，是两种截然不同的工作。我们中的大多数人脑力有限，只能更多地把精力放在让代码能工作上，而不是放在保持代码有组织和整洁上，这全然正确。分而治之在编程行为中的重要程度，等同于其在程序中的重要程度。

问题是太多人在程序能工作时就以为万事大吉了。我们没能把思维转向有关代码组织和整洁的部分。我们直接转向了下一个问题，而没能回过头将臃肿的类切分为只有单一权责的去耦式单元。

与此同时，许多开发者害怕数量巨大的短小单一目的类会导致难以一目了然抓住全局。他们认为，要搞清楚一件较大工作如何完成，就得在类与类之间找来找去。

然而，有大量短小类的系统并不比有少量庞大类的系统拥有更多移动部件，其数量大致相等。问题是：你是想把工具归置到有许多抽屉、每个抽屉中装有定义和标记良好的组件的工具箱中呢？还是想要少数几个能随便把所有东西扔进去的抽屉？

每个达到一定规模的系统都会包括大量逻辑和复杂性。管理这种复杂性的首要目标就是加以组织，以便开发者知道到哪儿能找到东西，并且在某个特定时间只需要理解直接相关的复杂性。反之，拥有巨大、多目的类的系统，总是让我们在目前并不需要了解的一大堆东西中艰难跋涉。

再强调一下：系统应该由许多短小的类而不是少量巨大的类组成，每个小类封装一个权责，只有一个修改的原因，并与少数其他类一起协同达成期望的系统行为。

### 10.2.2　内聚

类应该只有少数实体变量。类中的每个方法都应该操作一个或多个这种变量。通常而言，方法操作的变量越多，就越黏聚到类上。如果一个类中的每个变量都被每个方法所使用，则该类具有最大的内聚性。

一方面，一般来说，创建这种极大化内聚类是既不可取也不可能的；另一方面，我们希望内聚性保持在较高位置。内聚性高，意味着类中的方法和变量互相依赖、互相结合成一个逻辑整体。

看看代码清单10-4中一个`Stack`类的实现方式。这个类内聚性非常高。在3个方法中，只有`size()`方法没有使用所有两个变量。

代码清单10-4　Stack.java（一个内聚类）

```
public class Stack {
  private int topOfStack = 0;
  List<Integer> elements = new LinkedList<Integer>();

  public int size() {
    return topOfStack;
  }

  public void push(int element) {
    topOfStack++;
    elements.add(element);
  }

  public int pop() throws PoppedWhenEmpty {
    if (topOfStack == 0)
      throw new PoppedWhenEmpty();
    int element = elements.get(--topOfStack);
    elements.remove(topOfStack);
    return element;
  }
}
```

保持函数和参数列表短小的策略，有时会导致为一组子集方法所用的实体变量数量增加。出现这种情况时，往往意味着至少有一个类要从大类中挣扎出来。你应当尝试将这些变量和方法分拆到两个或多个类中，让新的类更为内聚。

### 10.2.3　保持内聚性就会得到许多短小的类

仅仅是将较大的函数切割为小函数，就将导致更多的类出现。想想看一个有许多变量的大函数。你想把该函数中某一小部分拆解成单独的函数。不过，你想要拆出来的代码使用了该函数中声明的4个变量。是否必须将这4个变量都作为参数传递到新函数中去呢？

完全没必要！只要将4个变量提升为类的实体变量，完全无须传递任何变量就能拆解代码了。应该很容易将函数拆分为小块。

可惜这也意味着类丧失了内聚性，因为堆积了越来越多只为允许少量函数共享而存在的实体变量。等一下！如果有些函数想要共享某些变量，为什么不让它们拥有自己的类呢？当类丧失了内聚性，就拆分它！

所以，将大函数拆分为许多小函数时，往往也是将类拆分为多个小类的时机。程序会更加有组织，也会拥有更为透明的结构。

为了说明我的意思，不如从Knuth的名著Literate Programming中摘取一个经过时间考验的例子。代码清单10-5展示了Knuth的`PrintPrimes`程序的Java版本。为示公平，以下程序并非Knuth原版，而是用他的Web工具输出的版本。采用它作为例子的目的，是因为它是展示如何将较大的函数分解为多个较小的函数和类的极好入手点。

代码清单10-5　PrintPrimes.java

```
package literatePrimes;

public class PrintPrimes {
  public static void main(String[] args) {
    final int M = 1000;
    final int RR = 50;
    final int CC = 4;
    final int WW = 10;
    final int ORDMAX = 30;
    int P[] = new int[M + 1];
    int PAGENUMBER;
    int PAGEOFFSET;
    int ROWOFFSET;
    int C;
    int J;
    int K;
    boolean JPRIME;
    int ORD;
    int SQUARE;
    int N;
    int MULT[] = new int[ORDMAX + 1];

    J = 1;
    K = 1;
    P[1] = 2;
    ORD = 2;
    SQUARE = 9;

    while (K < M) {
      do {
        J = J + 2;
        if (J == SQUARE) {
          ORD = ORD + 1;
          SQUARE = P[ORD] * P[ORD];
          MULT[ORD - 1] = J;
        }
        N = 2;
        JPRIME = true;
        while (N < ORD && JPRIME) {
          while (MULT[N] < J)
            MULT[N] = MULT[N] + P[N] + P[N];
          if (MULT[N] == J)
            JPRIME = false;
          N = N + 1;
        }
      } while (!JPRIME);
      K = K + 1;
      P[K] = J;
    }
    {
      PAGENUMBER = 1;
      PAGEOFFSET = 1;
      while (PAGEOFFSET <= M) {
        System.out.println("The First " + M + 
                           " Prime Numbers --- Page " + PAGENUMBER);
        System.out.println("");
        for (ROWOFFSET = PAGEOFFSET; ROWOFFSET < PAGEOFFSET + RR; ROWOFFSET++){
          for (C = 0; C < CC;C++)
            if (ROWOFFSET + C * RR <= M)
              System.out.format("%10d", P[ROWOFFSET + C * RR]);
          System.out.println("");
        }
        System.out.println("\f");
        PAGENUMBER = PAGENUMBER + 1;
        PAGEOFFSET = PAGEOFFSET + RR * CC;
      }
    }
  }
}
```

该程序只有一个大函数，简直一团糟，它拥有很深的缩进结构，冗余的变量和紧密耦合的结构。至少应该将其拆分为数个较小的函数。

从代码清单10-6到代码清单10-8，展示了将代码清单10-5中的代码拆分为较小的类和函数，并为这些类、函数和变量起个好名字后的结果。

代码清单10-6　PrimePrinter.java（重构后）

```
package literatePrimes;

public class PrimePrinter {
  public static void main(String[] args) {
    final int NUMBER_OF_PRIMES = 1000;
    int[] primes = PrimeGenerator.generate(NUMBER_OF_PRIMES);

    final int ROWS_PER_PAGE = 50;
    final int COLUMNS_PER_PAGE = 4;
    RowColumnPagePrinter tablePrinter =
      new RowColumnPagePrinter(ROWS_PER_PAGE,
                               COLUMNS_PER_PAGE,
                               "The First " + NUMBER_OF_PRIMES + 
                               " Prime Numbers");
    tablePrinter.print(primes);
  }
}
```

代码清单10-7　RowColumnPagePrinter.java

```
package literatePrimes;

import java.io.PrintStream;

public class RowColumnPagePrinter {
  private int rowsPerPage;
  private int columnsPerPage;
  private int numbersPerPage;
  private String pageHeader;
  private PrintStream printStream;

  public RowColumnPagePrinter(int rowsPerPage,
                              int columnsPerPage,
                              String pageHeader) {
    this.rowsPerPage = rowsPerPage;
    this.columnsPerPage = columnsPerPage;
    this.pageHeader = pageHeader;
    numbersPerPage = rowsPerPage * columnsPerPage;
    printStream = System.out;
  }

  public void print(int data[]) {
    int pageNumber = 1;
    for (int firstIndexOnPage = 0;
         firstIndexOnPage < data.length;
         firstIndexOnPage += numbersPerPage) {
      int lastIndexOnPage = 
        Math.min(firstIndexOnPage + numbersPerPage - 1, 
                 data.length - 1);
      printPageHeader(pageHeader, pageNumber);
      printPage(firstIndexOnPage, lastIndexOnPage, data);
      printStream.println("\f");
      pageNumber++;
    }
  }

  private void printPage(int firstIndexOnPage,
                         int lastIndexOnPage,
                         int[] data) {
    int firstIndexOfLastRowOnPage = 
      firstIndexOnPage + rowsPerPage - 1;
    for (int firstIndexInRow = firstIndexOnPage; 
         firstIndexInRow <= firstIndexOfLastRowOnPage; 
         firstIndexInRow++) {
      printRow(firstIndexInRow, lastIndexOnPage, data);
      printStream.println("");
    }
  }

  private void printRow(int firstIndexInRow, 
                        int lastIndexOnPage, 
                        int[] data) {
    for (int column = 0; column < columnsPerPage; column++) {
      int index = firstIndexInRow + column * rowsPerPage;
      if (index <= lastIndexOnPage)
        printStream.format("%10d", data[index]);
    }
  }

  private void printPageHeader(String pageHeader, 
                               int pageNumber) {
    printStream.println(pageHeader + " --- Page " + pageNumber);
    printStream.println("");
  }

  public void setOutput(PrintStream printStream) {
    this.printStream = printStream;
  }
}
```

代码清单10-8　PrimeGenerator.java

```
package literatePrimes;

import java.util.ArrayList;

public class PrimeGenerator {
  private static int[] primes;
  private static ArrayList<Integer> multiplesOfPrimeFactors;

  protected static int[] generate(int n) {
    primes = new int[n];
    multiplesOfPrimeFactors = new ArrayList<Integer>();
    set2AsFirstPrime();
    checkOddNumbersForSubsequentPrimes();
    return primes;
  }

  private static void set2AsFirstPrime() {
    primes[0] = 2;
    multiplesOfPrimeFactors.add(2);
  }

  private static void checkOddNumbersForSubsequentPrimes() {
    int primeIndex = 1;
    for (int candidate = 3;
         primeIndex < primes.length;
         candidate += 2) {
      if (isPrime(candidate))
        primes[primeIndex++] = candidate;
    }
  }

  private static boolean isPrime(int candidate) {
    if (isLeastRelevantMultipleOfNextLargerPrimeFactor(candidate)) {
      multiplesOfPrimeFactors.add(candidate);
      return false;
    }
    return isNotMultipleOfAnyPreviousPrimeFactor(candidate);
  }

  private static boolean
  isLeastRelevantMultipleOfNextLargerPrimeFactor(int candidate) {
    int nextLargerPrimeFactor = primes[multiplesOfPrimeFactors.size()];
    int leastRelevantMultiple = nextLargerPrimeFactor * nextLargerPrimeFactor;
    return candidate == leastRelevantMultiple;
  }

  private static boolean 
  isNotMultipleOfAnyPreviousPrimeFactor(int candidate) {
    for (int n = 1; n < multiplesOfPrimeFactors.size(); n++) {
      if (isMultipleOfNthPrimeFactor(candidate, n))
        return false;
    }
    return true;
  }

  private static boolean 
  isMultipleOfNthPrimeFactor(int candidate, int n) {
    return 
      candidate == smallestOddNthMultipleNotLessThanCandidate(candidate, n);
  }

  private static int 
  smallestOddNthMultipleNotLessThanCandidate(int candidate, int n) {
    int multiple = multiplesOfPrimeFactors.get(n);
    while (multiple < candidate)
      multiple += 2 * primes[n];
    multiplesOfPrimeFactors.set(n, multiple);
    return multiple;
  }
}
```

你可能注意到的第一件事就是程序比原来长了许多，从1页多增加到了将近3页。这有几个原因：其一，重构后的程序采用了更长、更有描述性的变量名；其二，重构后的程序将函数和类声明当作是给代码添加注释的一种手段；其三，我们采用了空格和格式技巧让程序更可读。

注意程序是如何被拆分为3个主要权责的。`PrimePrinter`类中只有主程序。主程序的权责是处理执行环境。如果调用方式改变，它也会随之改变。例如，如果程序被转换为SOAP服务，则该类也会受影响。

`RowColumnPagePrinter`类懂得如何将数字列表格式化到有着固定行数、列数的页面上。若输出格式需要改动，则该类也会受影响。

`PrimeGenerator`类懂得如何生成素数列表。注意，这并不意味着要实体化为对象。该类就是一个有用的作用域，在其中声明并隐藏变量。如果计算素数的算法发生改动，则该类也会改动。

这并不算是重写！因为我们没有从头开始写一遍程序。实际上，如果你仔细看上述两个不同的程序，就会发现它们采用了同样的算法和机制来完成工作。

我们通过编写验证第一个程序的精确行为的用例来实现修改。然后，我们做了许多小改动，每次改动一处。每改动一次，就执行一次，确保程序的行为没有变化。一小步接着一小步，第一个程序就这样被逐渐清理和转换为第二个程序。

## 10.3　为了修改而组织

对于多数系统，修改将一直持续。每处修改都让我们冒着系统其他部分不能如期望般工作的风险。在整洁的系统中，我们对类加以组织，以降低修改的风险。

代码清单10-9中的`Sql`类用来生成提供恰当元数据的SQL格式化字符串。这个类还没写完，所以暂时不支持`update`语句等SQL功能。当需要`Sql`类支持`update`语句时，我们就得“打开”这个类进行修改。打开类带来的问题是风险随之而来。对类的任何修改都有可能破坏类中的其他代码。因此必须全面重新测试。

代码清单10-9　一个必须打开修改的类

```
public class Sql {
  public Sql(String table, Column[] columns)
  public String create()
  public String insert(Object[] fields)
  public String selectAll()
  public String findByKey(String keyColumn, String keyValue)
  public String select(Column column, String pattern)
  public String select(Criteria criteria)
  public String preparedInsert()
  private String columnList(Column[] columns)
  private String valuesList(Object[] fields, final Column[] columns)
  private String selectWithCriteria(String criteria)
  private String placeholderList(Column[] columns)
}
```

当增加一种新类型语句时，就要修改`Sql`类。改动单个语句类型时，也要进行修改，比如，打算让`select`功能支持子查询。如果存在两个修改的理由，就说明`Sql`违反了SRP原则。

可以从简单的组织性观点发现对SRP的违反。`Sql`的方法概述显示，存在类似于`selectWithCriteria`等只与`select`语句有关的私有方法。

出现了只与类的一小部分有关的私有方法行为，就意味着存在改进空间。然而，展开行动的基本动因却应该是系统的变动。若我们认为`Sql`类在逻辑上已具足，则无须担心对权责的拆分。如果在可预见的未来无须增加`update`功能，就不该去修改`Sql`类。不过，一旦打开了类，就应当修正设计方案。

代码清单10-10中的解决方式如何呢？代码清单10-9中`Sql`类的每个接口方法都重构到从`Sql`类派生出来的类中了。注意那些私有方法，如`valuesList`，直接移到了需要用它们的地方。公共私有行为被划分到独立的两个工具类`Where`和`ColumnList`中。

代码清单10-10　一组封闭类

```
abstract public class Sql {
  public Sql(String table, Column[] columns)
  abstract public String generate();
}

public class CreateSql extends Sql {
  public CreateSql(String table, Column[] columns)
  @Override public String generate()
}

public class SelectSql extends Sql {
  public SelectSql(String table, Column[] columns)
  @Override public String generate()
}

public class InsertSql extends Sql {
  public InsertSql(String table, Column[] columns, Object[] fields)
  @Override public String generate()
  private String valuesList(Object[] fields, final Column[] columns)
}

public class SelectWithCriteriaSql extends Sql {
  public SelectWithCriteriaSql(
    String table, Column[] columns, Criteria criteria)
  @Override public String generate()
}

public class SelectWithMatchSql extends Sql {
  public SelectWithMatchSql(
    String table, Column[] columns, Column column, String pattern)
  @Override public String generate()
}

public class FindByKeySql extends Sql{
  public FindByKeySql(
    String table, Column[] columns, String keyColumn, String keyValue)
  @Override public String generate()
}

public class PreparedInsertSql extends Sql {
  public PreparedInsertSql(String table, Column[] columns)
  @Override public String generate(){
  private String placeholderList(Column[] columns)
}

public class Where {
  public Where(String criteria)
  public String generate()
}
public class ColumnList {
  public ColumnList(Column[] columns)
  public String generate()
}
```

每个类中的代码都变得极为简单。理解每个类花费的时间缩减到近乎为零。函数对其他函数造成毁坏的风险也变得几近于无。从测试的角度看，验证方案中每一处逻辑都成了极为简单的任务，因为类与类之间相互隔离了。

当需要增加`update`语句时，现存类无须做任何修改，这也同样重要！我们在`Sql`类的新子类`UpdateSql`中构建`update`语句的逻辑，系统中的其他代码都不会因为这个修改而被破坏。

重新架构的`Sql`逻辑百利而无一弊。它符合SRP，也符合其他面向对象设计的关键原则，如开放闭合原则（OCP）：类应当对扩展开放，对修改封闭。通过子类化手段，重新架构的`Sql`类对添加新功能是开放的，而且可以同时不触及其他类，只要将`UpdateSql`类放置到位就行了。

我们希望精心组织系统，从而在添加或修改特性时尽可能少惹麻烦。在理想系统中，我们通过扩展系统而非修改现有代码来添加新特性。

### 隔离修改

需求会改变，所以代码也会改变。在OO 101中，我们学习到，具体类包含实现细节（代码），而抽象类则只呈现概念。依赖具体细节的客户类，当细节改变时，就会有风险。我们可以借助接口和抽象类来隔离这些细节带来的影响。

对具体细节的依赖给对系统的测试带来了挑战。如果我们构建一个依赖外部`TokyoStockExchange`API的`Portfolio`类以代表投资组合的价值，则测试用例就会受到价值查询的连带影响。如果每隔5分钟就有新说法，就很难写出测试来。

与其设计直接依赖`TokyoStockExchange`的`Portfolio`类，不如创建`StockExchange`接口，其中只声明一个方法：

```
public interface StockExchange {
  Money currentPrice(String symbol);
}
```

我们设计`TokyoStockExchange`类来实现这个接口。我们还要确保`Portfolio`的构造器接受作为参数的`StockExchange`引用：

```
public  Portfolio  {
  private  StockExchange  exchange;
  public  Portfolio(StockExchange  exchange)  {
    this.exchange  =  exchange;
  }
  //  ...
}
```

现在就可以为`StockExchange`接口创建可测试的尝试性实现了，该尝试性实现将返回固定的现值。如果测试中购买了5股微软股票，则尝试性实现总是返回每股100美元的现值。对于`StockExchange`接口的尝试性实现简化为简单的表格查找。然后再编写一个总投资价值为500美元的测试。

```
public class PortfolioTest {
  private FixedStockExchangeStub exchange;
  private Portfolio portfolio;

  @Before
  protected void setUp() throws Exception {
    exchange = new FixedStockExchangeStub();
    exchange.fix("MSFT", 100);
    portfolio = new Portfolio(exchange);
  }

  @Test
  public void GivenFiveMSFTTotalShouldBe500() throws Exception {
    portfolio.add(5, "MSFT");
    Assert.assertEquals(500, portfolio.value());
  }
}
```

如果系统解耦到这样测试的程度，也就更加灵活，更加可复用。部件之间的解耦代表着系统中的元素互相隔离得很好。隔离也让对系统每个元素的理解变得更加容易。

通过降低连接度，我们的类就遵循了另一条类设计原则，即依赖倒置原则（Dependency Inversion Principle，DIP）。本质而言，DIP认为类应当依赖抽象而不是依赖具体细节。

我们的`Portfolio`类不再依赖`TokyoStockExchange`类的实现细节，而是依赖`StockExchange`接口。`StockExchange`接口呈现的是有关询问某只股票价格的抽象概念，这种抽象隔离了所有询价的特定细节，其中包括价格数据来自何处的类。
