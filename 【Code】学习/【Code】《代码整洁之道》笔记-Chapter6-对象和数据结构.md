# 第6章　对象和数据结构

我们将变量设置为私有（private）有一个理由：不想让其他人依赖这些变量。我们还想在心血来潮时能自由修改其类型或实现。那么，为什么还是有那么多程序员不假思索就给对象添加赋值器（setter）和取值器（getter），将私有变量公之于众，如同它们是公共变量一般呢？

## 6.1　数据抽象

看看代码清单6-1和代码清单6-2之间的区别。每段代码都表示笛卡儿平面上的一个点。不过，其中一个曝露了其实现，而另一个则完全隐藏了其实现。

代码清单6-1　具象点

```
public class Point {
  public double x;
  public double y;
}
```

代码清单6-2　抽象点

```
public interface Point {
  double getX();
  double getY();
  void setCartesian(double x, double y);
  double getR();
  double getTheta();
  void setPolar(double r, double theta);
}
```

代码清单6-2的漂亮之处在于，你不知道该实现会是在矩形坐标系中还是在极坐标系中。可能两个都不是！然而，该接口还是明白无误地呈现了一种数据结构。

不过它呈现的还不止是一个数据结构。那些方法固化了一套存取策略，你可以单独读取某个坐标，但必须通过一次原子操作设定所有坐标。

而代码清单6-1则非常清楚地表明是在矩形坐标系中实现的，并要求我们单个操作那些坐标。这就曝露了实现。实际上，即便变量都是私有的，而且我们也通过变量取值器和赋值器使用变量，其实现也被曝露了。

隐藏实现并非只是在变量之间放上一个函数层那么简单。隐藏实现关乎抽象！类并不简单地用取值器和赋值器将其变量推向外界，而是曝露抽象接口，以便用户无须了解数据的实现就能操作数据本体（essence）。

看看代码清单6-3和代码清单6-4。前者采用具象手段与机动车的燃料层通信，而后者则采用百分比抽象。你能确定前者里面都是一些变量存取器，而却无法得知后者中的数据形态。

代码清单6-3　具象机动车

```
public interface Vehicle {
  double getFuelTankCapacityInGallons();
  double getGallonsOfGasoline();
}
```

代码清单6-4　抽象机动车

```
public interface Vehicle {
  double getPercentFuelRemaining();
}
```

以上两段代码以后者为佳。我们不愿意曝露数据细节，而更愿意以抽象形态表述数据。这并不意味着只是用接口和/或赋值器、取值器就万事大吉。要以最好的方式呈现某个对象包含的数据，需要进行严肃的思考。随意乱加取值器和赋值器是最坏的选择。

## 6.2　数据、对象的反对称性

这两个例子展示了对象与数据结构之间的差异。对象把数据隐藏于抽象之后，曝露操作数据的函数；而数据结构曝露其数据，没有提供有意义的函数。回过头再读一遍，注意这两种定义的本质，其实它们是对立的。这种差异貌似微小，但却有深远的含义。

例如，代码清单6-5中的过程式形状代码范例。`Geometry`类操作3个形状类。形状类都是简单的数据结构，没有任何行为。所有行为都在`Geometry`类中。

代码清单6-5　过程式形状代码

```
public class Square {
  public Point topLeft;
  public double side;
}

public class Rectangle {
  public Point topLeft;
  public double height;
  public double width;
}

public class Circle {
  public Point center;
  public double radius;
}

public class Geometry {
  public final double PI = 3.141592653589793;

  public double area(Object shape) throws NoSuchShapeException 
  {
    if (shape instanceof Square) {
      Square s = (Square)shape;
      return s.side * s.side;
    }
    else if (shape instanceof Rectangle) {
      Rectangle r = (Rectangle)shape;
      return r.height * r.width;
    }
    else if (shape instanceof Circle) {
      Circle c = (Circle)shape;
      return PI * c.radius * c.radius;
    }
    throw new NoSuchShapeException();
  }
}
```

面向对象编程的程序员可能会对此嗤之以鼻，抱怨说这是过程式代码——他们大概是对的，不过这种嘲笑并不完全正确。想想看，如果给`Geometry`类添加一个`primeter()`函数会怎样？那些形状类根本不会因此而受影响！另一方面，如果要添加一个新形状，就得修改`Geometry`中的所有函数来处理它。再读一遍代码。注意，这两种情形也是直接对立的。

现在来看看代码清单6-6中的面向对象方案。这里，`area()`方法是多态的，不需要有`Geometry`类。所以，如果要添加一个新形状，现有的函数中没有一个会受到影响，而当添加新函数时所有的形状都得做修改！

代码清单6-6　多态式形状

```
public class Square implements Shape {
  private Point topLeft;
  private double side;

  public double area() {
    return side*side;
  }
}

public class Rectangle implements Shape {
  private Point topLeft;
  private double height;
  private double width;

  public double area() {
    return height * width;
  }
}

public class Circle implements Shape {
  private Point center;
  private double radius;
  public final double PI = 3.141592653589793;

  public double area() {
    return PI * radius * radius;
  }
}
```

我们再次看到这两种定义的本质：它们是截然对立的。这说明了对象与数据结构之间的二分原理：

过程式代码（使用数据结构的代码）便于在不改动既有数据结构的前提下添加新函数；面向对象代码便于在不改动既有函数的前提下添加新类。

反过来讲也说得通：

过程式代码难以添加新数据结构，因为必须修改所有函数；面向对象代码难以添加新函数，因为必须修改所有类。

所以，对于面向对象较难完成的事，对于过程式代码却较容易，反之亦然！

在任何一个复杂系统中，都会有需要添加新数据类型而不是新函数的时候。这时，对象和面向对象就比较适合。另一方面，也会有想要添加新函数而不是数据类型的时候。在这种情况下，过程式代码和数据结构就更适合。

老练的程序员知道，一切都是对象的说法只是一个传说。有时候你真的想要在简单数据结构上做一些过程式的操作。

## 6.3　得墨忒耳律

著名的得墨忒耳律（The Law of Demeter）认为，模块不应了解它所操作对象的内部情形。如上节所见，对象隐藏数据，曝露操作。这意味着对象不应通过存取器曝露其内部结构，因为这样更像是曝露而非隐藏其内部结构。

更准确地说，得墨忒耳律认为，类C的方法f只应该调用以下对象的方法：

-　C；

-　由f创建的对象；

-　作为参数传递给f的对象；

-　由C的实体变量持有的对象。

方法不应调用由任何函数返回的对象的方法。换言之，只跟朋友谈话，不与陌生人谈话。

下列代码违反了得墨忒耳律（除违反其他规则之外），因为它调用了`getOptions()`返回值的`getScratchDir()`函数，又调用了`getScratchDir()`返回值的`getAbsolutePath()`方法。

```
final String outputDir = ctxt.getOptions().getScratchDir().getAbsolutePath();
```

### 6.3.1　火车失事

这类代码常被称作火车失事，因为它看起来就像是一列火车。这类连串的调用通常被认为是肮脏的风格，应该避免[G36]。最好做类似如下的切分：

```
Options opts = ctxt.getOptions();
File scratchDir = opts.getScratchDir();
final String outputDir = scratchDir.getAbsolutePath();
```

上述代码是否违反了得墨忒耳律呢？当然，模块知道`ctxt`对象包含多个选项，每个选项中都有一个临时目录，而每个临时目录都有一个绝对路径。对于一个函数，这些知识真够丰富的。调用函数懂得如何在一大堆不同对象间浏览。

这些代码是否违反得墨忒耳律，取决于`ctxt`、`Options`和`ScratchDir`是对象还是数据结构。如果是对象，则它们的内部结构应当隐藏而不曝露，而有关其内部细节的知识就明显违反了得墨忒耳律。如果`ctxt`、`Options`和`ScratchDir`只是数据结构，没有任何行为，则它们自然会曝露其内部结构，得墨忒耳律也就不适用了。

属性访问器函数的使用把问题搞复杂了。如果像下面这样写代码，我们大概就不会提及是否违反得墨忒耳律。

```
final String outputDir = ctxt.options.scratchDir.absolutePath;
```

如果数据结构只简单地拥有公共变量，没有函数，而对象则拥有私有变量和公共函数，那么这个问题就没那么复杂。然而，有些框架和标准甚至要求最简单的数据结构也要有访问器和改值器。

### 6.3.2　混杂

这种混杂有时会不幸地导致混合结构，即一半是对象，另一半是数据结构。这种结构拥有执行操作的函数，也有公共变量或公共访问器及改值器。无论出于怎样的初衷，公共访问器及改值器都把私有变量公开化，诱导外部函数以过程式程序使用数据结构的方式使用这些变量。

此类混杂增加了添加新函数的难度，也增加了添加新数据结构的难度，两头不讨好。应避免创造这种结构。它们的出现，展示了一种乱七八糟的设计，其作者不确定——或者更糟糕，完全无视——他们是否需要函数或类型的保护。

### 6.3.3　隐藏结构

假使`ctxt`、`Options`和`ScratchDir`是拥有真实行为的对象又怎样呢？由于对象应隐藏其内部结构，我们就不该看到内部结构。这样一来，如何才能取得临时目录的绝对路径呢？

```
ctxt.getAbsolutePathOfScratchDirectoryOption();
```

或者

```
ctx.getScratchDirectoryOption().getAbsolutePath()
```

第一种方案可能导致`ctxt`对象中方法的曝露。第二种方案是在假设`getScratchDirectoryOption()`返回一个数据结构而非对象。感觉两种方案都不好。

如果`ctxt`是一个对象，就应该要求它做点儿什么，而不该要求它给出内部情形。那我们为何还要得到临时目录的绝对路径呢？我们要它做什么？来看看同一模块（许多行之后）的这段代码：

```
String outFile = outputDir + "/" + className.replace('.', '/') + ".class";
FileOutputStream fout = new FileOutputStream(outFile);
BufferedOutputStream bos = new BufferedOutputStream(fout);
```

这种不同层级细节的混杂（[G34][G6]）有点麻烦。句点、斜杠、文件扩展名和`File`对象不该如此随便地混杂到一起。不过，撇开这些毛病，我们发现，取得临时目录绝对路径的初衷是为了创建指定名称的临时文件。

所以，直接让`ctxt`对象来做这事如何？

```
BufferedOutputStream bos = ctxt.createScratchFileStream(classFileName);
```

这下看起来像是对象做的事了！`ctxt`隐藏了其内部结构，防止当前函数因浏览它不该知道的对象而违反得墨忒耳律。

## 6.4　数据传送对象

最为精练的数据结构，是一个只有公共变量、没有函数的类。这种数据结构有时被称为数据传送对象（Data Transfer Objects，DTO）。DTO是非常有用的结构，尤其是在与数据库通信或解析套接字传递的消息之类的场景中，在应用程序代码里一系列将原始数据转换为数据库的翻译过程中，它们往往是排头兵。

更常见的是如代码清单6-7所示的“bean”结构。“bean”结构拥有由赋值器和取值器操作的私有变量。对“bean”结构的半封装会让某些面向对象纯化论者感觉舒服些，不过通常没有其他好处。

代码清单6-7　address.java

```
public class Address {
  private String street;
  private String streetExtra;
  private String city;
  private String state;
  private String zip;

  public Address(String street, String streetExtra, 
                 String city, String state, String zip) {
    this.street = street;
    this.streetExtra = streetExtra;
    this.city = city;
    this.state = state;
    this.zip = zip;
  }

  public String getStreet() {
    return street;
  }

  public String getStreetExtra() {
    return streetExtra;
  }

  public String getCity() {
    return city;
  }

  public String getState() {
    return state;
  }

  public String getZip() {
    return zip;
  }
}
```

### Active Record

Active Record是一种特殊的DTO形式。它们是拥有公共（或可“bean”式访问的）变量的数据结构，但通常也会拥有类似`save`和`find`这样的可浏览方法。Active Record一般是对数据库表或其他数据源的直接翻译。

我们不幸经常发现开发者往这类数据结构中塞进业务规则方法，把这类数据结构当成对象来用。这是不智的行为，因为它导致了数据结构和对象的混杂体。

当然，解决方案就是把Active Record当作数据结构，并创建包含业务规则、隐藏内部数据（可能就是Active Record的实体）的独立对象。

## 6.5　小结

对象曝露行为，隐藏数据，便于添加新对象类型而无须修改既有行为，同时难以在既有对象中添加新行为；数据结构曝露数据，没有明显的行为，便于向既有数据结构添加新行为，同时难以向既有函数添加新数据结构。

在任何系统中，我们有时会希望能够灵活地添加新数据类型，所以更喜欢在这部分使用对象。另外一些时候，我们希望能灵活地添加新行为，这时我们更喜欢使用数据类型和过程。优秀的软件开发者不带成见地了解这种情形，并依据手边工作的性质选择其中一种适合的手段。
