# 第9章　单元测试

过去十年以来，编程专业领域进步很大。1997年时，没人听说过测试驱动开发。对于我们之中的大多数人来说，单元测试是那种用来确保程序“可运行”的用过即扔的短代码。我们辛勤地编写类和方法，再弄出一些特殊代码来测试它们。通常这些代码会是一种简单的驱动式程序，让我们能够手工与自己编写的程序交互。

我记得在20世纪90年代曾为一套嵌入式实时系统编写过C++程序。该程序是一个简单的计时器，有如下签名：

```
void Timer::ScheduleCommand(Command* theCommand, int milliseconds)
```

想法很简单；到达指定毫秒数时，在一个新线程中执行`Command`的`execute`方法。问题在于如何测试它。

我随便写了个简单的驱动式程序，聆听来自键盘的动作。键盘输入一个字符时，它就安排5秒之后输出同样的字符。我输入了一句带节奏的歌词，然后等着5秒之后它在屏幕上重现出来。

“I . . . want-a-girl . . . just . . . like-the-girl-who-marr . . . ied . . . dear . . . old . . . dad.”[1]

在按下那些“.”键时，我真的在哼着那段旋律，当那些句点出现在屏幕上时，我又哼了一次。

那就是我的测试！我看到这法子可行，演示给同事们看，然后就把代码扔掉了。

如前文所述，我们的专业领域进步甚多。如今，我会编写测试，确保代码中每个犄角旮旯都如我所愿地工作。我会将代码和操作系统隔离开，而不是直接调用标准计时功能。我会伪造一套计时函数，这样就能全面控制时间。我会安排一些设置布尔值标识的命令，往前步进时间，查看这些标识，确保它们在我将时间调到正确值时由`false`变为`true`。

有了一套运行通过的测试，我会确保任何需要用到代码的人都能方便地使用这些测试。我会确保测试和代码一起签入同一个代码包。

对，我们进步甚多，但还有很长的路要走。敏捷和TDD运动鼓舞了许多程序员编写自动化单元测试，每天还有更多人加入这个行列。但是，在争先恐后将测试加入规程中时，许多程序员遗漏了一些关于编写好测试的更细微但却重要的要点。

## 9.1　TDD三定律

谁都知道TDD要求我们在编写生产代码前先编写单元测试。但这条规则只是冰山之巅。看看下列3条定律。

第一定律　在编写不能通过的单元测试前，不可编写生产代码。

第二定律　只可编写刚好无法通过的单元测试，不能编译也算不通过。

第三定律　只可编写刚好足以通过当前失败测试的生产代码。

这3条定律将你限制在大概30秒一个的循环中。测试与生产代码一起编写，测试只比生产代码早写几秒。

这样写程序，我们每天就会编写数十个测试，每个月编写数百个测试，每年编写数千个测试。这样写程序，测试将覆盖所有生产代码。测试代码量足以匹敌生产代码量，导致令人生畏的管理问题。

## 9.2　保持测试整洁

几年前，有人请我去指导一个开发团队。那个团队认定，测试代码的维护不应遵循生产代码的质量标准。他们彼此默许在单元测试中破坏规矩。“速而不周”成了团队格言，即变量命名不用很好，测试函数不必短小和具有描述性，测试代码不必做良好设计和仔细划分，只要测试代码还能工作，只要还覆盖着生产代码，就足够好。

有些读者可能会同意这种做法。或许，在很久以前，你也用过我为那个`Timer`类写测试的方法。从编写那种用后即扔的测试到编写全套自动化单元测试是一大进步。所以，就像那个我指导过的团队一样，你或许也会认为脏测试好于没测试。

这个团队没有意识到的是，脏测试等同于——如果不是坏于的话——没测试。问题在于，测试必须随生产代码的演进而修改。测试越脏，就越难修改。测试代码越缠结，你就越有可能花更多时间塞进新测试，而不是编写新的生产代码。修改生产代码后，旧测试就会开始失败，而测试代码中乱七八糟的东西将阻碍代码再次通过。于是，测试变得就像是不断翻番的债务。

随着版本递进，团队维护测试代码组的代价也在上升，最终，这样的代价变成了开发者最大的抱怨对象。当经理们问及为何超支如此巨大，开发者们就归咎于测试。最后，他们只能扔掉整个测试代码组。

但是，没有了测试代码组，他们就失去了确保对代码的改动能如愿工作的能力。没有了测试代码组，他们就无法确保对系统某个部分的修改不会影响系统的其他部分。故障率开始上升。随着并非出自有意的故障越来越多，他们开始害怕做改动。他们不再清理生产代码，因为他们害怕修改带来的损害多于收益。生产代码开始腐坏。最后，他们只剩下没有测试、纷乱而缺陷缠身的生产代码，沮丧的客户，还有对测试的失望。

在某种意义上，他们说对了。测试的确让他们失望。不过是他们自己决定让测试变得乱七八糟的，而那正是失败的根源。如果他们保持测试整洁，测试就不会令他们失望，我可以拍着胸脯这么说，因为我曾经参与并指导了多个凭借整洁单元测试获得成功的团队。

故事的寓意很简单：测试代码和生产代码一样重要。测试代码可不是二等公民，它需要被思考、被设计和被照料，它该像生产代码一般保持整洁。

### 测试带来一切好处

如果测试不能保持整洁，你就会失去它们。没有了测试，你就会失去保证生产代码可扩展的一切要素。你没看错，正是单元测试让你的代码可扩展、可维护、可复用。原因很简单。有了测试，你就不用担心对代码的修改！没有测试，每次修改都可能带来缺陷。无论架构多有扩展性，无论设计划分得有多好，如果没有了测试，你就很难做改动，因为你担忧改动会引入不可预知的缺陷。

有了测试，愁云一扫而空。测试覆盖率越高，你就越不用担心。哪怕是对于那种架构并不优秀、设计晦涩纠缠的代码，你也能近乎没有后患地做修改。实际上，你甚至能毫无顾虑地改进架构和设计！

所以，覆盖了生产代码的自动化单元测试程序组能尽可能地保持设计和架构的整洁。测试带来了一切好处，因为测试使改动变得可能。

如果测试不干净，你改动自己代码的能力就会有所限制，而你也会开始失去改进代码结构的能力。测试越脏，代码就会变得越脏。最终，你丢失了测试，代码开始腐坏。

## 9.3　整洁的测试

整洁的测试有哪些要素呢？有3个要素：可读性、可读性和可读性。在单元测试中，可读性甚至比在生产代码中还重要。测试如何才能做到可读？和在其他代码中一样：明确，简洁，并有足够的表达力。在测试中，你要以尽可能少的文字表达大量内容。

我们来看看代码清单9-1中来自FitNesse的代码。这3个测试很难读懂，显然有改善空间。首先，其中有数量巨大的重复代码[G5]调用`addPage`和`assertSubString`。更重要的是，代码中充满干扰测试表达力的细节。

代码清单9-1　SerializedPageResponderTest.java

```
public void testGetPageHieratchyAsXml() throws Exception
{
  crawler.addPage(root, PathParser.parse("PageOne"));
  crawler.addPage(root, PathParser.parse("PageOne.ChildOne"));
  crawler.addPage(root, PathParser.parse("PageTwo"));

  request.setResource("root");
  request.addInput("type", "pages");
  Responder responder = new SerializedPageResponder();
  SimpleResponse response = 
    (SimpleResponse) responder.makeResponse(
       new FitNesseContext(root), request);
  String xml = response.getContent();

  assertEquals("text/xml", response.getContentType());
  assertSubString("<name>PageOne</name>", xml);
  assertSubString("<name>PageTwo</name>", xml);
  assertSubString("<name>ChildOne</name>", xml);
}

public void testGetPageHieratchyAsXmlDoesntContainSymbolicLinks() 
throws Exception
{
  WikiPage pageOne = crawler.addPage(root, PathParser.parse("PageOne"));
  crawler.addPage(root, PathParser.parse("PageOne.ChildOne"));
  crawler.addPage(root, PathParser.parse("PageTwo"));

  PageData data = pageOne.getData();
  WikiPageProperties properties = data.getProperties();
  WikiPageProperty symLinks = properties.set(SymbolicPage.PROPERTY_NAME);
  symLinks.set("SymPage", "PageTwo");
  pageOne.commit(data);

  request.setResource("root");
  request.addInput("type", "pages");
  Responder responder = new SerializedPageResponder();
  SimpleResponse response = 
    (SimpleResponse) responder.makeResponse(
       new FitNesseContext(root), request);
  String xml = response.getContent();

  assertEquals("text/xml", response.getContentType());
  assertSubString("<name>PageOne</name>", xml);
  assertSubString("<name>PageTwo</name>", xml);
  assertSubString("<name>ChildOne</name>", xml);
  assertNotSubString("SymPage", xml);
}

public void testGetDataAsHtml() throws Exception
{
  crawler.addPage(root, PathParser.parse("TestPageOne"), "test page");

  request.setResource("TestPageOne");
  request.addInput("type", "data");
  Responder responder = new SerializedPageResponder();
  SimpleResponse response = 
    (SimpleResponse) responder.makeResponse(
       new FitNesseContext(root), request);
  String xml = response.getContent();

  assertEquals("text/xml", response.getContentType());
  assertSubString("test page", xml);
  assertSubString("<Test", xml);
}
```

请看对`PathParser`的那些调用，它们将字符串转换为供爬虫使用的`PagePath`实体。转换与测试毫无关系，徒然混淆了代码的意图。与创建`responder`相关的细节，还有`response`的收集与转换也尽是噪声，此外还有从`resource`和参数构造请求URL的笨手段。（这些代码我有幸参与编写，所以可以敞开来批评。）

最终，这段代码不是设计来给人看的。可怜的读者淹没在细节的汪洋大海中，在真正用到测试之前，还得理解这些细节。

现在看看代码清单9-2中改进了的测试。这些测试还是做同样的事，不过已经被重构为更整洁和更有表达力的形式。

代码清单9-2　SerializedPageResponderTest.java（重构后）

```
public void testGetPageHierarchyAsXml() throws Exception {
  makePages("PageOne", "PageOne.ChildOne", "PageTwo");

  submitRequest("root", "type:pages");

  assertResponseIsXML();
  assertResponseContains(
    "<name>PageOne</name>", "<name>PageTwo</name>", "<name>ChildOne</name>"
  );
}

public void testSymbolicLinksAreNotInXmlPageHierarchy() throws Exception {
  WikiPage page = makePage("PageOne");
  makePages("PageOne.ChildOne", "PageTwo");

  addLinkTo(page, "PageTwo", "SymPage");

  submitRequest("root", "type:pages");

  assertResponseIsXML();
  assertResponseContains(
    "<name>PageOne</name>", "<name>PageTwo</name>", "<name>ChildOne</name>"
  );
  assertResponseDoesNotContain("SymPage");
}

public void testGetDataAsXml() throws Exception {
  makePageWithContent("TestPageOne", "test page");

  submitRequest("TestPageOne", "type:data");

  assertResponseIsXML();
  assertResponseContains("test page", "<Test");
}
```

这些测试显然呈现了构造-操作-检验（BUILD-OPERATE-CHECK）模式。每个测试都清晰地拆分为3个环节。第一个环节构造测试数据，第二个环节操作测试数据，第三个环节部分检验操作是否得到期望的结果。

注意，大部分恼人的细节消失了。测试直达目的，只用到那些真正需要的数据类型和函数。读测试的人应该都能够很快搞清楚状况，而不至于被细节误导或吓倒。

### 9.3.1　面向特定领域的测试语言

代码清单9-2中的测试展示了为测试构造一种面向特定领域的语言的技巧。我们没有直接使用程序员用来对系统进行操作的API，而是打造了一套包装这些API的函数和工具代码，这样就能更方便地编写测试，写出来的测试也更便于阅读。那正是一种测试语言，可以帮助程序员编写自己的测试，也可以帮助后来者阅读测试。

这种测试API并非起初就设计出来的，而是在对那些充满令人迷惑细节的测试代码进行后续重构时逐渐演进的。如同你看见我将代码清单9-1重构为代码清单9-2一般，守规矩的开发者也将他们的测试代码重构为更简洁和更具表达力的形式。

### 9.3.2　双重标准

在某种意义上，本章开始处提到的那个团队的做法是正确的。测试API中的代码与生产代码相比，的确有一套不同的工程标准。测试代码应当简单、精悍、足具表达力，但它该和生产代码一般有效。毕竟它是在测试环境而非生产环境中运行的，这两种环境有着截然不同的需求。

请看代码清单9-3中的测试。在为某个环境控制系统设计原型时，我写了这个测试。无须深入细节，你就能说出该测试是在“温度太低”时检验温度警报器、加热器和送风机是否全部打开。

代码清单9-3　EnvironmentControllerTest.java

```
@Test
public void turnOnLoTempAlarmAtThreashold() throws Exception {
  hw.setTemp(WAY_TOO_COLD);
  controller.tic();
  assertTrue(hw.heaterState());
  assertTrue(hw.blowerState());
  assertFalse(hw.coolerState());
  assertFalse(hw.hiTempAlarm());
  assertTrue(hw.loTempAlarm());
}
```

当然，这里也有许多细节。例如，`tic`函数是做什么的？实际上，在读测试时你可以不用担心这些问题，你只需考虑是否同意系统最终状态与“温度太低”的情况相符。

当你阅读这个测试时，可以留意到自己的眼光得在被检验的状态的名称与状态的意义之间来回跳转。你看到`heaterState`，眼光向左滑到`assertTrue`。你看到`coolerState`，眼光向左看`assertFalse`。这个过程既乏味又不可靠，它让测试变得难以阅读。

我大幅改进了测试的可读性，得到代码清单9-4。

代码清单9-4　EnvironmentControllerTest.java（重构后）

```
@Test
public void turnOnLoTempAlarmAtThreshold() throws Exception {
  wayTooCold();
  assertEquals("HBchL", hw.getState());
}
```

当然，我创建了一个`wayTooCold`函数，隐藏了`tic`函数的细节。不过要注意的是，`assertEquals`中的那个奇怪的字符串，大写表示“打开”，小写表示“关闭”，那些字符遵循以下次序：`{heater, blower, cooler, hi-temp-alarm, lo-temp-alarm}`。

尽管这破坏了思维映射的规则，但看来它在这种情况下还是适用的。只要你明白其含义，你就能一眼看到那个字符串，并迅速译解出结果，如代码清单9-5所示。

代码清单9-5　EnvironmentControllerTest.java（扩展到更大范围）

```
@Test
public void turnOnCoolerAndBlowerIfTooHot() throws Exception {
  tooHot();
  assertEquals("hBChl", hw.getState());
}

@Test
public void turnOnHeaterAndBlowerIfTooCold() throws Exception {
  tooCold();
  assertEquals("HBchl", hw.getState());
}

@Test
public void turnOnHiTempAlarmAtThreshold() throws Exception {
  wayTooHot();
  assertEquals("hBCHl", hw.getState());
}

@Test
public void turnOnLoTempAlarmAtThreshold() throws Exception {
  wayTooCold();
  assertEquals("HBchL", hw.getState());
}
```

代码清单9-6中给出了`getState`函数，注意，它的代码效率不是非常高。要提升效率，可能应该使用`StringBuffer`。

代码清单9-6　MockControlHardware.java

```
public String getState() {
  String state = "";
  state += heater ? "H" : "h";
  state += blower ? "B" : "b";
  state += cooler ? "C" : "c";
  state += hiTempAlarm ? "H" : "h";
  state += loTempAlarm ? "L" : "l";
  return state;
}
```

`StringBuffer`有点儿丑陋。即便在生产代码中，就算代价较小，我也会避免使用`StringBuffer`；而且你可以看到，代码清单9-6中代码的代价的确很小。这套应用显然是嵌入式实时系统，计算机和内存资源都很有限。不过，测试环境大概完全不必做限制。

这就是双重标准。有些事你大概永远不会在生产环境中做，而在测试环境中做却完全没问题。通常这关乎内存或CPU效率的问题，不过却永远不会与整洁有关。

## 9.4　每个测试一个断言

有一个流派认为，JUnit中每个测试函数都应该有且只有一个断言语句。这条规则看似过于苛刻，但其好处却可以在代码清单9-5中看到。这些测试都归结为一个可快速方便地理解的结论。

代码清单9-2又如何？我们能将关于输出是XML的断言与输出包含某些子字符串的断言轻易地组合到一起，不过这样做看来毫无道理。然而，我们可以将测试分解为两个单独的测试，每个测试都有各自的断言，如代码清单9-7所示。

代码清单9-7　SerializedPageResponderTest.java（单个断言的版本）

```
public void testGetPageHierarchyAsXml() throws Exception {
  givenPages("PageOne", "PageOne.ChildOne", "PageTwo");

  whenRequestIsIssued("root", "type:pages");

  thenResponseShouldBeXML();
}

public void testGetPageHierarchyHasRightTags() throws Exception {
  givenPages("PageOne", "PageOne.ChildOne", "PageTwo");

  whenRequestIsIssued("root", "type:pages");

  thenResponseShouldContain(
    "<name>PageOne</name>", "<name>PageTwo</name>", "<name>ChildOne</name>"
  );
}
```

注意，我修改了那些函数的名称，以符合given-when-then约定。这让测试更易阅读。不幸的是，如此分解测试，导致了许多重复代码的出现。

可以利用模板方法（TEMPLATE METHOD）模式，将given/when部分放到基类中，将then部分放到派生类中，消除代码重复问题。或者，我们也可以创建一个完整的单独测试类，把given和when部分放到`@Before`函数中，把when部分放到每个@Test函数中，但对于这个小问题，这种做法看来有点儿机械。最后，我还是保留了代码清单9-2那种多个断言的形式。

我认为，单个断言是一个好准则。我通常都会创建支持这条准则的特定领域测试语言，如代码清单9-5所示。不过，我也不害怕在单个测试中放入多于一个的断言。我认为最好的说法是，单个测试中的断言数量应该最小化。

### 每个测试一个概念

更好一些的规则或许是每个测试函数中只测试一个概念。我们不想要超长的测试函数，测试完这个又测试那个。代码清单9-8就是那样一种测试的例子。这个测试应当拆解为3个单独测试，因为它测试了3件不同的事。如果把3件事混到一起，读者就不得不猜想每段代码出现的理由，以及那段代码到底要测试什么。

代码清单9-8

```
/**
 * Miscellaneous tests for the addMonths() method.
 */
public void testAddMonths() {
  SerialDate d1 = SerialDate.createInstance(31, 5, 2004);

  SerialDate d2 = SerialDate.addMonths(1, d1);
  assertEquals(30, d2.getDayOfMonth());
  assertEquals(6, d2.getMonth());
  assertEquals(2004, d2.getYYYY());

  SerialDate d3 = SerialDate.addMonths(2, d1);
  assertEquals(31, d3.getDayOfMonth());
  assertEquals(7, d3.getMonth());
  assertEquals(2004, d3.getYYYY());

  SerialDate d4 = SerialDate.addMonths(1, SerialDate.addMonths(1, d1));
  assertEquals(30, d4.getDayOfMonth());
  assertEquals(7, d4.getMonth());
  assertEquals(2004, d4.getYYYY());
}
```

这3个测试函数大概应该像下面这个样子。

- 对于某个有31天的月份（如5月）的最后一天 
    - 增加一个该月最末一天为30日的月份（如6月）时，日期应该是该月的30日而非31日。
    - 增加最末月有31天的两个月时，日期应该是31日。 
- 对于某个有30天的月份（如6月）的最后一天
    - 增加一个有31天的月份时，日期应该是30日而非31日。

这样一来，你可以看到，在这些混杂的测试当中，隐藏有一条普遍规则。增加月数时，日期不能大于该月的最末一天。这意味着在2月28日增加月份数，就会得到3月28日。而这个测试应该有用，但被遗漏了。

并非是由于代码清单9-8中每个段落的多重断言导致问题。问题在于，有多个概念被测试，所以，最佳规则也许是应该尽可能减少每个概念的断言数量，每个测试函数只测试一个概念。

## 9.5　F.I.R.S.T.

整洁的测试还遵循以下5条规则，这5条规则的首字母构成了本节标题。

- 快速（Fast）。测试应该够快。测试应该能快速运行。测试运行缓慢，你就不会想要频繁地运行它。如果你不频繁运行测试，就不能尽早发现问题，也无法轻易修正，从而也不能轻而易举地清理代码。最终，代码就会腐坏。

- 独立（Independent）。测试应该相互独立。某个测试不应为下一个测试设定条件。你应该可以单独运行每个测试，以及以任何顺序运行测试。当测试互相依赖时，头一个测试没通过就会导致一连串的测试失败，使问题诊断变得困难，隐藏了下级错误。

- 可重复（Repeatable）。测试应当可以在任何环境中重复通过。你应该能够在生产环境、质检环境中运行测试，也能够在无网络的列车上用笔记本电脑运行测试。如果测试不能在任意环境中重复，你就总会有个解释其失败的借口。当环境条件不具备时，你也会无法运行测试。

- 自足验证（Self-Validating）。测试应该有布尔值输出。无论是通过或失败，你都不应该通过查看日志文件来确认测试是否通过。你不应该手工对比两个不同的文本文件来确认测试是否通过。如果测试不能自足验证，对失败的判断就会变得依赖主观，而运行测试也需要更长的手工操作时间。

- 及时（Timely）。测试应及时编写。单元测试应该恰好在使其通过的生产代码之前编写。如果在编写生产代码之后编写测试，你会发现生产代码难以测试。你可能会因为某些生产代码本身难以测试而不去设计可测试的代码。

## 9.6　小结

我们只是触及了这个话题的表面。实际上，我认为应该为整洁的测试写上一整本书。对于项目的健康度，测试和生产代码同等重要。或许测试更为重要，因为它保证和增强了生产代码的可扩展性、可维护性和可复用性。所以，保持测试整洁吧。让测试具有表达力并短小精悍。发明作为面向特定领域语言的测试API，帮助自己编写测试。

如果你坐视测试腐坏，那么代码也会跟着腐坏。保持测试整洁吧。
