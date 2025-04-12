# 第8章　边界

我们很少能控制系统中的全部软件。有时我们购买第三方程序包或使用开放源代码，有时我们依靠公司中其他团队打造组件或子系统。不管是哪种情况，我们都得将外来代码干净利落地整合进自己的代码中。本章将介绍一些保持软件边界整洁的实践手段和技巧。

## 8.1　使用第三方代码

在接口提供者和使用者之间，存在与生俱来的矛盾。第三方程序包和框架提供者追求普适性，这样就能在多种环境中工作，从而吸引广泛的用户。而使用者则想要得到集中满足特定需求的接口。这种矛盾会导致系统边界上出现问题。

以`java.util.Map`为例，`Map`有着广阔的接口和丰富的功能。当然，这种力量和灵活性很有用，但也要付出代价。比如，应用程序可能构造一个`Map`对象并传递它。我们的初衷可能是`Map`对象的所有接收者都不要删除映射图中的任何东西。但它却正好有一个`clear()`方法。`Map`的任何使用者都能清除映射图。或许设计惯例是`Map`中只能保存特定的类型，但`Map`并不会可靠地约束存于其中的对象的类型。使用者可随意往`Map`中塞入任何类型的条目。

- clear() void – Map 

- containsKey(Object key) boolean – Map 

- containsValue(Object value) boolean – Map 

- entrySet() Set – Map 

- equals(Object o) boolean – Map 

- get(Object key) Object – Map 

- getClass() Class<? extends Object> – Object 

- hashCode() int – Map 

- isEmpty() boolean – Map 

- keySet() Set – Map 

- notify() void – Object 

- notifyAll() void – Object 

- put(Object key, Object value) Object – Map 

- putAll(Map t) void – Map 

- remove(Object key) Object – Map 

- size() int – Map 

- toString() String – Object 

- values() Collection – Map 

- wait() void – Object 

- wait(long timeout) void – Object 

- wait(long timeout, int nanos) void – Object 

如果你的应用程序需要一个包容`Sensor`类对象的`Map`映射图，大概会是这样：

```
Map sensors = new HashMap();
```

当代码的其他部分需要访问这些传感器，就会有这行代码：

```
Sensor s = (Sensor)sensors.get(sensorId);
```

这行代码一再出现。代码的调用端承担了从`Map`中取得对象并将其转换为正确类型的职责。行倒是行，却非整洁代码。而且，这行代码并未说明自己的用途。通过对泛型的使用，这段代码的可读性可以大大提升，如下所示：

```
    Map<String,Sensor> sensors = new HashMap<Sensor>();
...
    Sensor s = sensors.get(sensorId);
```

不过，`Map<String, Sensor>`提供了超出所需/所愿的功能的问题，仍未得到解决。

在系统中不受限制地传递`Map<String, Sensor>`的实体，意味着当`Map`的接口被修改时，有许多地方都要跟着改。你或许会认为这样的改动不太可能发生，不过，当Java 5加入对泛型的支持时，的确发生了改动。我们也的确见到一些系统因为要做大量改动才能自由使用`Map`类，而无法使用泛型。

使用`Map`的更整洁的方式大致如下。`Sensors`的用户不必关心是否用了泛型，那将是（也该是）实现细节才关心的。

```
public class Sensors {
  private Map sensors = new HashMap();

  public Sensor getById(String id) {
    return (Sensor) sensors.get(id);
  }

  // snip
}
```

边界上的接口（`Map`）是隐藏的。它能随来自应用程序其他部分的极小的影响而变动。对泛型的使用不再是个大问题，因为转换和类型管理是在`Sensors`类内部处理的。

该接口也经过仔细修整和归置以适应应用程序的需要。结果就是得到易于理解、难以被误用的代码。`Sensors`类推动了设计和业务的规则的形成。

我们并不建议总是以这种方式封装`Map`的使用。我们建议不要将`Map`（或在边界上的其他接口）在系统中传递。如果你使用类似`Map`这样的边界接口，就把它保留在类或近亲类中。避免从公共API中返回边界接口，或将边界接口作为参数传递给公共API。

## 8.2　浏览和学习边界

第三方代码帮助我们在更少时间内发布更丰富的功能。在利用第三方程序包时，该从何处入手呢？我们没有测试第三方代码的职责，但为要使用的第三方代码编写测试，可能最符合我们的利益。

设想我们对第三方代码库的使用方法并不清楚。我们可能会花上一两天（或者更多）时间阅读文档，决定如何使用。然后，我们会编写使用第三方代码的代码，看看是否如我们所愿地工作。我们会陷入长时间的调试，找出在我们或他们代码中的缺陷，这可不是什么稀罕事。

学习第三方代码很难，整合第三方代码也很难，同时做这两件事难上加难。如果我们采用不同的做法呢？不要在生产代码中试验新东西，而是编写测试来遍览和理解第三方代码。Jim Newkirk把这叫作学习性测试（learning test）。

在学习性测试中，我们就像在应用中那样调用第三方代码。我们基本上是在通过核对试验来检测自己对那个API的理解程度。测试聚焦于我们想从API得到的东西。

## 8.3　学习log4j

比如，我们想使用Apache `log4j`包来代替自定义的日志代码。我们下载了`log4j`，打开介绍文档页。无须看太久，就编写了第一个测试用例，希望它能向控制台输出“hello”字样。

```
@Test
public void testLogCreate() {
  Logger logger = Logger.getLogger("MyLogger");
  logger.info("hello");
}
```

运行上述代码，logger发生了一个错误，告诉我们需要用`Appender`。再多读一点文档，我们发现有个`ConsoleAppender`。于是我们创建了一个`ConsoleAppender`，再看是否能解开向控制台输出日志的秘诀。

```
@Test
public void testLogAddAppender() {
  Logger logger = Logger.getLogger("MyLogger");
  ConsoleAppender appender = new ConsoleAppender();
  logger.addAppender(appender);
  logger.info("hello");
}
```

这回，我们发现`Appender`没有输出流。奇怪，它该有输出流的。在谷歌上得到一点帮助后，我们写了以下代码：

```
@Test
public void testLogAddAppender() {
  Logger logger = Logger.getLogger("MyLogger");
  logger.removeAllAppenders();
  logger.addAppender(new ConsoleAppender(
    new PatternLayout("%p %t %m%n"), 
    ConsoleAppender.SYSTEM_OUT));
  logger.info("hello");
}
```

这回行了，“hello”字样的日志信息出现在控制台上！必须告知`ConsoleAppender`，让它往控制台写字，看起来有点儿奇怪。

很有趣，当我们移除`ConsoleAppender.SystemOut`参数时，那个“hello”字样仍然输出到屏幕上。但如果取走`PatternLayout`，就会出现关于没有输出流的错误信息。这实在太古怪了。

再仔细看看文档，我们看到默认的`ConsoleAppender`构造器是“未配置”的，这看起来并不明显或没什么用，反而像是`log4j`的一个缺陷，或者至少是前后不太一致。

再搜索、阅读、测试，最终我们得到代码清单8-1。我们极大地发掘了`log4j`的工作方式，也将得到的知识融入了一系列简单的单元测试中。

代码清单8-1　LogTest.java

```
public class LogTest {
  private Logger logger;

  @Before
  public void initialize() {
    logger = Logger.getLogger("logger");
    logger.removeAllAppenders();
    Logger.getRootLogger().removeAllAppenders();
  }
  @Test
  public void basicLogger() {
    BasicConfigurator.configure();
    logger.info("basicLogger");
  }

  @Test
  public void addAppenderWithStream() {
    logger.addAppender(new ConsoleAppender(
        new PatternLayout("%p %t %m%n"),
        ConsoleAppender.SYSTEM_OUT));
      logger.info("addAppenderWithStream");
  }

  @Test
  public void addAppenderWithoutStream() {
    logger.addAppender(new ConsoleAppender(
         new PatternLayout("%p %t %m%n")));
      logger.info("addAppenderWithoutStream");
  }
}
```

现在我们知道如何初始化一个简单的控制台日志器，也能把这些知识封装到自己的日志类中，好将应用程序的其他部分与`log4j`的边界接口隔离开来。

## 8.4　学习性测试的好处不只是免费

学习性测试毫无成本。无论如何我们都得学习要使用的API，而编写测试则是获得这些知识的容易而不会影响其他工作的途径。学习性测试是一种精确试验，帮助我们增进对API的理解。

学习性测试不光免费，还在投资上有正向的回报。当第三方程序包发布了新版本，我们可以运行学习性测试，看看程序包的行为有没有改变。

学习性测试确保第三方程序包按照我们想要的方式工作。一旦整合进来，就不能保证第三方代码总与我们的需求兼容。原作者不得不修改代码来满足他们自己的新需求。他们会修正缺陷、添加新功能。风险伴随新版本而来。如果第三方程序包的修改与测试不兼容，我们也能马上发现。

无论你是否需要通过学习性测试来学习，都要有一系列与生产代码中调用方式一致的输出测试来支持整洁的边界。如果不使用这些边界测试来减轻迁移的劳力，我们可能就会超出应有时限，长久地绑在旧版本上面。

## 8.5　使用尚不存在的代码

还有另一种边界，那种将已知和未知分隔开的边界。在代码中总有许多地方是我们的知识未及之处。有时，边界那边就是未知的（至少目前未知）。有时，我们并不往边界那边看过去。

好多年以前，我曾在一个开发无线通信系统软件的团队中工作。该系统有一个子系统`Transmitter`（发送机）。我们对`Transmitter`知之甚少，而该子系统的开发者还没有对接口进行定义。我们不想受这种事阻碍，就从距未知那部分代码很远处开始工作。

对于我们的世界如何结束、新世界如何开始，我们有许多好主意。工作时，我们偶尔会跨越那道边界。尽管云雾遮挡了我们看向边界那边的视线，但是我们还是从工作中了解到我们想要的边界接口是什么样的。我们想要告知发送机一些事：

将发送机置于指定频率，并发出自这个流得到的数据的模拟表示。

我们不知这会如何做到，因为API还没设计出来。所以，我们决定过后再编写细节代码。

为了不受阻碍，我们定义了自己使用的接口。我们给它起了个好记的名字，比如`Transmitter`。我们给它写了个名为`transmit`的方法，获取频率参数和数据流。这就是我们希望得到的接口。

编写我们想得到的接口的好处之一是它在我们控制之下。这有助于保持客户代码更可读，且集中于它该完成的工作。

我们将`CommunicationsController`类从发送器API（Transmitter API，该API不受我们控制，而且还没定义）中隔离出来。通过使用符合应用程序的接口，`CommunicationsController`代码整洁且足以表达其意图。一旦发送器API被定义出来，我们就编写`TransmitterAdapter`来跨接。ADAPTER封装了与API的互动，也提供了一个当API发生变动时唯一需要改动的地方。

这套设计方案为测试提供了一种极为方便的接缝。使用适当的`FakeTransmitter`，我们就能测试`CommunicationsController`类。在拿到`TransmitterAPI`时，我们也能创建确保正确使用API的边界测试。

## 8.6　整洁的边界

边界上会发生有趣的事。改动是其中之一。如果有良好的软件设计，则无须巨大投入和重写即可进行修改。在使用我们控制不了的代码时，必须加倍小心保护投资，确保未来的修改不至于代价太大。

边界上的代码需要清晰的分割和定义了期望的测试。应该避免我们的代码过多地了解第三方代码中的特定信息。依靠你能控制的东西，好过依靠你控制不了的东西，免得日后受它控制。

我们通过代码中少数几处引用第三方边界接口的位置来管理第三方边界。可以像我们对待`Map`那样包装它们，也可以使用ADAPTER模式将我们的接口转换为第三方提供的接口。采用这两种方式，代码都能更好地与我们沟通，如果在边界两边推动内部一致的用法，当第三方代码有改动时修改点就会更少。
