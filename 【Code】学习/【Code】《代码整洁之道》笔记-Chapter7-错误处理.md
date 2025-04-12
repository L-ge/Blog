# 第7章　错误处理

在一本有关整洁代码的书中，居然有讨论错误处理的章节，看起来有些突兀。错误处理只不过是编程时必须要做的事之一。输入可能出现异常，设备可能失效。简言之，可能会出错，当错误发生时，程序员就有责任确保代码照常工作。

然而，应该弄清楚错误处理与整洁代码的关系。许多程序完全由错误处理所占据。所谓占据，并不是说错误处理就是全部。我的意思是几乎无法看明白代码所做的事，因为到处都是凌乱的错误处理代码。错误处理很重要，但如果它搞乱了代码逻辑，就是错误的做法。

在本章中，我将概要列出编写既整洁又强固的代码——雅致地处理错误代码的一些技巧和思路。

## 7.1　使用异常而非返回码

在很久以前，许多语言都不支持异常，这些语言汇报和处理错误的手段都有限。你要么设置一个错误标识，要么返回给调用者检查的错误码。代码清单7-1中的代码展示了这些手段。

代码清单7-1　DeviceController.java

```
public class DeviceController {
  ...
  public void sendShutDown() {
    DeviceHandle handle = getHandle(DEV1);
    // Check the state of the device
    if (handle != DeviceHandle.INVALID) {
      // Save the device status to the record field
      retrieveDeviceRecord(handle);
      // If not suspended, shut down
      if (record.getStatus() != DEVICE_SUSPENDED) {
        pauseDevice(handle);
        clearDeviceWorkQueue(handle);
        closeDevice(handle);
      } else {
        logger.log("Device suspended.  Unable to shut down");
      }
    } else {
      logger.log("Invalid handle for: " + DEV1.toString());
    }
  }
  ...
}
```

这类手段的问题在于，它们搞乱了调用者代码，调用者必须在调用之后即刻检查错误。不幸的是，这个步骤很容易被遗忘。所以，遇到错误时，最好抛出一个异常，这样调用代码会很整洁，其逻辑不会被错误处理搞乱。

代码清单7-2展示了在方法中遇到错误时抛出异常的情形。

代码清单7-2　DeviceController.java（采用异常处理）

```
public class DeviceController {
  ...

  public void sendShutDown() {
    try {
      tryToShutDown();
    } catch (DeviceShutDownError e) {
      logger.log(e);
    }
  }

  private void tryToShutDown() throws DeviceShutDownError {
    DeviceHandle handle = getHandle(DEV1);
    DeviceRecord record = retrieveDeviceRecord(handle);

    pauseDevice(handle);
    clearDeviceWorkQueue(handle);
    closeDevice(handle);
  }

  private DeviceHandle getHandle(DeviceID id) {
    ...
    throw new DeviceShutDownError("Invalid handle for: " + id.toString());
    ...
  }

  ...
}
```

注意，这段代码整洁了很多，这不仅关乎美观。这段代码更好一些，因为之前相互纠结的两个主题，设备关闭算法和错误处理，现在被隔离了。你可以查看其中任一主题，分别理解它。

## 7.2　先写try-catch-finally语句

异常的妙处之一是，它们在程序中定义了范围。执行`try-catch-finally`语句中`try`部分的代码时，你是在表明可随时取消执行，并在`catch`语句中接续。

在某种意义上，`try`代码块就像是事务。`catch`代码块将程序维持在一种持续状态，无论`try`代码块中发生了什么均如此。所以，在编写可能抛出异常的代码时，最好先写出`try-catch-finally`语句。这能帮你定义该代码的用户应该期待什么，无论`try`代码块中执行的代码出什么错都一样。

来看个例子。我们要编写访问某个文件并读出一些序列化对象的代码。

先写一个单元测试，其中显示当文件不存在时将得到一个异常：

```
@Test(expected = StorageException.class)
public void retrieveSectionShouldThrowOnInvalidFileName() {
  sectionStore.retrieveSection("invalid - file");
}
```

该测试令我们创建以下占位代码：

```
public List<RecordedGrip> retrieveSection(String sectionName) {
  // dummy return until we have a real implementation
  return new ArrayList<RecordedGrip>();
}
```

测试失败了，因为以上代码并未抛出异常。下一步，修改实现代码，尝试访问非法文件。该操作抛出一个异常：

```
public List<RecordedGrip> retrieveSection(String sectionName) {
  try {
    FileInputStream stream = new FileInputStream(sectionName)
  } catch (Exception e) {
    throw new StorageException("retrieval error", e);
  }
  return new ArrayList<RecordedGrip>(); 
}
```

这次测试通过了，因为我们捕获了异常，此时就可以重构了。我们可以缩小异常类型的范围，使之符合`FileInputStream`构造器真正抛出的异常，即`FileNotFound Exception`：

```
public List<RecordedGrip> retrieveSection(String sectionName) {
  try {
    FileInputStream stream = new FileInputStream(sectionName);
    stream.close();
  } catch (FileNotFoundException e) {
    throw new StorageException("retrieval error", e);
  }
  return new ArrayList<RecordedGrip>();
}
```

如此一来，我们就用`try-catch`结构定义了范围，可以继续用测试驱动开发（TDD）方法构建剩余的代码逻辑。这些代码逻辑将在`FileInputStream`和`close`之间添加，装作一切正常的样子。

尝试编写强行抛出异常的测试，再往处理器中添加行为，使之满足测试要求。结果就是你要先构造`try`代码块的事务范围，而且也会帮助你维护好该范围的事务特征。

## 7.3　使用未检异常

争辩业已结束。多年来，Java程序员们一直在争论已检异常（checked exception）的利与弊。Java的第一个版本中引入已检异常时，已检异常看似是一个极好的点子。每个方法的签名都列出它可能传递给调用者的异常。而且，这些异常就是方法类型的一部分。如果签名与代码实际所做之事不符，代码在字面上就无法编译。

那时，我们认为已检异常是个绝妙的主意，而且，它也有所裨益。然而，现在已经很清楚，对于强固软件的生产，它并非必需的。`C#`不支持已检异常。尽管做过勇敢的尝试，`C++`最后也不支持已检异常。`Python`和`Ruby`同样如此。不过，用这些语言也有可能写出强固的软件。我们得决定——的确如此——已检异常是否值回票价。

代价是什么？已检异常的代价就是违反开放/闭合原则。如果你在方法中抛出已检异常，而`catch`语句在3个层级之上，你就得在`catch`语句和抛出异常处之间的每个方法签名中声明该异常。这意味着对软件中较低层级的修改，都将波及较高层级的签名。修改好的模块必须重新构建、发布，即便它们自身所关注的任何东西都没改动过。

以某个大型系统的调用层级为例。顶端函数调用它们之下的函数，逐级向下。假设某个位于最低层级的函数被修改为抛出一个异常。如果该异常是已检的，则函数签名就要添加`throw`子句。这意味着每个调用该函数的函数都要被修改，捕获新异常，或在其签名中添加合适的`throw`子句。以此类推。最终得到的就是一个从软件最底端贯穿到最顶端的修改链！封装被打破了，因为在抛出路径中的每个函数都要去了解下一层级的异常细节。既然异常旨在让你能在较远处处理错误，那么已检异常以这种方式破坏封装简直就是一种耻辱。

如果你在编写一套关键代码库，则已检异常有时也会有用：你必须捕获异常。但对于一般的应用开发，其依赖成本要高于收益。

## 7.4　给出异常发生的环境说明

你抛出的每个异常，都应当提供足够的环境说明，以便判断错误的来源和位置。在Java中，你可以从任何异常里得到栈踪迹（stack trace），然而，栈踪迹却无法告诉你该失败操作的初衷。

应创建信息充分的错误消息，并和异常一起传递出去，在消息中，应包括失败的操作和失败类型。如果你的应用程序有日志系统，可以传递足够的信息给`catch`块，并记录下来。

## 7.5　依调用者需要定义异常类

对异常分类有很多方式。可以依其来源分类：是来自组件还是其他地方？也可以依其类型分类：是设备错误、网络错误还是编程错误？不过，当我们在应用程序中定义异常类时，最重要的考虑应该是它们如何被捕获。

我们来看一个不太好的异常分类的例子。下面的`try-catch-finally`语句是对某个第三方代码库的调用。它覆盖了该调用可能抛出的所有异常：

```
ACMEPort port = new ACMEPort(12);

try {
  port.open();
} catch (DeviceResponseException e) {
  reportPortError(e);
  logger.log("Device response exception", e);
} catch (ATM1212UnlockedException e) {
  reportPortError(e);
  logger.log("Unlock exception", e);
} catch (GMXError e) {
  reportPortError(e);
  logger.log("Device response exception");
} finally {
  …
}
```

语句包含了一大堆重复代码，这并不出奇。在大多数异常处理中，不管真实原因如何，我们总是做相对标准的处理。我们得记录错误，确保能继续工作。

在本例中，既然知道我们所做的事不外如此，就可以通过打包调用API，确保它返回通用异常类型，从而简化代码。

```
LocalPort port = new LocalPort(12);
try {
  port.open();
} catch (PortDeviceFailure e) {
  reportError(e);
  logger.log(e.getMessage(), e);
} finally {
  ...
}
```

`LocalPort`类就是一个简单的打包类，它捕获并翻译由`ACMEPort`类抛出的异常：

```
public class LocalPort {
  private ACMEPort innerPort;

  public LocalPort(int portNumber) {
    innerPort = new ACMEPort(portNumber);
  }

  public void open() {
    try {
      innerPort.open();
    } catch (DeviceResponseException e) {
      throw new PortDeviceFailure(e);
    } catch (ATM1212UnlockedException e) {
      throw new PortDeviceFailure(e);
    } catch (GMXError e) {
      throw new PortDeviceFailure(e);
    }
  }
  ...
}
```

类似于我们为`ACMEPort`定义的这种打包类非常有用。实际上，将第三方API打包是个良好的实践手段。当你打包一个第三方API，你就降低了对它的依赖：未来你可以不太痛苦地改用其他代码库。在你测试自己的代码时，打包也有助于模拟第三方调用。

打包的好处还在于你不必绑死在某个特定厂商的API设计上，你可以定义自己感觉舒服的API。在上例中，我们为`port`设备错误定义了一个异常类型，然后发现这样能写出更整洁的代码。

对于代码的某个特定区域，单一异常类通常可行。伴随异常发送出来的信息能够区分不同错误。如果你想要捕获某个异常，并且放过其他异常，就使用不同的异常类。

## 7.6　定义常规流程

如果你遵循前文提及的建议，在业务逻辑和错误处理代码之间就会有良好的区隔。大量代码会开始变得像是整洁而简朴的算法。然而，这样做却把错误检测推到了程序的边缘地带。你打包了外部API以抛出自己的异常，你在代码的顶端定义了一个处理器来应对任何失败了的运算。在大多数时候，这种手段很棒，不过有时你也许不愿这么做。

我们来看一个例子。下面的笨代码来自某个记账应用的开支总计模块：

```
try {
  MealExpenses expenses = expenseReportDAO.getMeals(employee.getID());
  m_total += expenses.getTotal();
} catch(MealExpensesNotFound e) {
  m_total += getMealPerDiem();
}
```

业务逻辑是：如果消耗了餐食，则计入总额中；如果没有消耗，则员工得到当日餐食补贴。异常打断了业务逻辑。如果不去处理特殊情况会不会好一些？那样的话代码看起来会更简洁，就像这样：

```
MealExpenses expenses = expenseReportDAO.getMeals(employee.getID());
m_total += expenses.getTotal();
```

能把代码写得那样简洁吗？能。可以修改`ExpenseReportDAO`，使其总是返回`MealExpense`对象。如果没有消耗餐食，就返回一个返回餐食补贴的`MealExpense`对象。

```
public class PerDiemMealExpenses implements MealExpenses {
  public int getTotal() {
    // return the per diem default
  }
}
```

这种手法叫作特例模式（SPECIAL CASE PATTERN [Fowler]）。创建一个类或配置一个对象，用来处理特例。你来处理特例，客户代码就不用应对异常行为了。异常行为被封装到特例对象中。

## 7.7　别返回null值

我认为，要讨论错误处理，就一定要提及那些容易引发错误的做法。第一项就是返回`null`值。我不想去计算曾经见过多少个几乎每行代码都在检查`null`值的应用程序。下面就是其中的一个例子：

```
public void registerItem(Item item) {
  if (item != null) {
    ItemRegistry registry = peristentStore.getItemRegistry();
    if (registry != null) {
      Item existing = registry.getItem(item.getID());
      if (existing.getBillingPeriod().hasRetailOwner()) {
        existing.register(item);
      }
    }
  }
}
```

这种代码看似不坏，其实糟透了！返回`null`值，基本上是在给自己增加工作量，也是在给调用者添乱。只要有一处没检查`null`值，应用程序就会失控。

你有没有注意到，嵌套`if`语句的第二行没有检查`null`值？如果在运行时`persistentStore`为`null`会发生什么事？我们会在运行时得到一个`NullPointer-Exception`异常，也许有人在代码顶端捕获这个异常，也可能没有捕获。两种情况都很糟糕。对于从应用程序深处抛出的`NullPointerException`异常，你到底该作何反应呢？

可以敷衍说上列代码的问题是少做了一次`null`值检查，其实问题多多。如果你打算在方法中返回`null`值，不如抛出异常，或是返回特例对象。如果你在调用某个第三方API中可能返回`null`值的方法，可以考虑用新方法打包这个方法，在新方法中抛出异常或返回特例对象。

在许多情况下，特例对象都是爽口良药。设想有以下一段代码：

```
List<Employee> employees = getEmployees();
if (employees != null) {
  for(Employee e : employees) {
    totalPay += e.getPay();
  }
}
```

现在，`getExployees`可能返回`null`值，但是否一定要这么做呢？如果修改`getEmployees`，返回空列表，就能使代码整洁起来：

```
List<Employee> employees = getEmployees();
for(Employee e : employees) {
  totalPay += e.getPay();
}
```

所幸Java有`Collections.emptyList()`方法，该方法返回一个预定义的不可变列表，可用于达到这种目的：

```
public List<Employee> getEmployees() {
  if( .. there are no employees .. ) 
    return Collections.emptyList();
}
```

这样编码，就能尽量避免`NullPointerException`的出现，代码也更整洁了。

## 7.8　别传递null值

在方法中返回`null`值是糟糕的做法，将`null`值传递给其他方法就更糟糕了。除非API要求你向它传递`null`值，否则就要尽可能避免传递`null`值。

举例说明原因。用下面这个简单的方法计算两点间的一种度量：

```
public class MetricsCalculator 
{
  public double xProjection(Point p1, Point p2) {
    return (p2.x – p1.x) * 1.5;
  }
  ...
}
```

如果有人传入`null`值会怎样？

```
calculator.xProjection(null, new Point(12, 13));
```

当然，我们会得到一个`NullPointerException`异常。

如何修正？可以创建一个新异常类型并抛出：

```
public class MetricsCalculator 
{
  public double xProjection(Point p1, Point p2) {
    if (p1 == null || p2 == null) {
      throw InvalidArgumentException(
        "Invalid argument for MetricsCalculator.xProjection");
    }
    return (p2.x – p1.x) * 1.5;
  }
}
```

这样做好些吗？可能比`null`指针异常好一些，但要记住，我们还得为`InvalidArgumentException`异常定义处理器。这个处理器该做什么？还有更好的做法吗？

还有替代方案。可以使用一组断言（assertion）：

```
public class MetricsCalculator 
{
  public double xProjection(Point p1, Point p2) {
    assert p1 != null : "p1 should not be null";
    assert p2 != null : "p2 should not be null";
    return (p2.x – p1.x) * 1.5;
  }
}
```

看上去很美，但仍未解决问题。如果有人传入`null`值，还是会得到运行时错误。

在大多数编程语言中，没有良好的方法能应对由调用者意外传入的`null`值。事已至此，恰当的做法就是禁止传入`null`值。这样，你在编码的时候，就会时时记住参数列表中的`null`值意味着出问题了，从而大量避免这种无心之失。

## 7.9　小结

整洁代码是可读的，但也要强固。可读与强固并不冲突。如果将错误处理隔离看待，独立于主要逻辑之外，就能写出强固而整洁的代码。做到这一步，我们就能单独处理它，也可以极大地提升代码的可维护性。
