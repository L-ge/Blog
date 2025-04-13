# 第12章　迭进

## 12.1　通过迭进设计达到整洁目的

假使有4条简单的规则，跟着做就能帮助你创建优良的设计，会如何？假使遵循这些规则，你就能洞见代码的结构和设计，更能轻易地应用SRP和DIP之类的原则，便会如何？假使这4条规则有利于良好的设计“浮现”出来，又会如何？

我们中的许多人认为，Kent Beck关于简单设计的4条规则，对于创建具有良好设计的软件有着莫大的帮助。

据Kent所述，只要遵循以下规则，设计就能变得“简单”：

- 运行所有测试；

- 不可重复；

- 表达了程序员的意图；

- 尽可能减少类和方法的数量。

以上规则按其重要程度排列。

## 12.2　简单设计规则1：运行所有测试

设计必须制造出如预期一般工作的系统，这是首要因素。系统也许有一套绝佳设计，但如果缺乏验证系统是否真按预期那样工作的简单方法，那就无异于纸上谈兵。

全面测试并持续通过所有测试的系统，就是可测试的系统。这看似浅显，却很重要。不可测试的系统同样不可验证。不可验证的系统，绝不应部署。

幸运的是，只要系统可测试，就会导向保持类短小且目的单一的设计方案。遵循SRP的类，测试起来较为简单。测试编写得越多，就越能持续走向编写较易测试的代码。所以，确保系统完全可测试能帮助我们创建更好的设计。

紧耦合的代码难以编写测试。同样，编写测试越多，就越会遵循DIP之类的规则，从而越会使用依赖注入、接口和抽象等工具尽可能减少耦合，如此一来，设计就会有长足进步。

遵循有关编写测试并持续运行测试的简单、明确的规则，系统就会更贴近面向对象低耦合度、高内聚度的目标。编写测试将会引致更好的设计。

## 12.3　简单设计规则2～4：重构

有了测试，就能保持代码和类的整洁，方法就是递增式地重构代码。添加了几行代码后，就要暂停，琢磨一下变化了的设计。设计退步了吗？如果是，就要清理它，并且运行测试，保证没有破坏任何东西。测试消除了对清理代码就会破坏代码的恐惧。

在重构过程中，可以应用有关优秀软件设计的一切知识，提升内聚性，降低耦合度，切分关注面，模块化系统性关注面，缩小函数和类的尺寸，选用更好的名称，如此等等。这也是应用简单设计后3条规则的地方：消除重复，保证表达力，尽可能减少类和方法的数量。

## 12.4　不可重复

重复是拥有良好设计的系统的大敌。它代表着额外的工作、额外的风险和额外且不必要的复杂度。重复有多种表现。极其雷同的代码行当然是重复。类似的代码往往可以调整得更相似，这样就能更容易地进行重构。重复也有实现上的重复等其他一些形态，例如，在某个群集类中可能会有以下两个方法：

```
int size()  {}
boolean isEmpty()  {}
```

这两个方法可以分别实现。`isEmpty`方法跟踪一个布尔值，而`size`方法则跟踪一个计数器。或者，也可以通过在`isEmpty`方法中使用`size`方法来消除重复：

```
boolean isEmpty() {
  return 0 == size();
}
```

要想创建整洁的系统，需要有消除重复的意愿，即便对于短短几行也是如此。例如，以下代码：

```
 public void scaleToOneDimension(
     float desiredDimension, float imageDimension) {
   if (Math.abs(desiredDimension - imageDimension) < errorThreshold)
      return;
   float scalingFactor = desiredDimension / imageDimension;
   scalingFactor = (float)(Math.floor(scalingFactor * 100) * 0.01f);

   RenderedOp newImage = ImageUtilities.getScaledImage(
      image, scalingFactor, scalingFactor);
   image.dispose();
   System.gc();
   image = newImage;
 }
 public synchronized void rotate(int degrees) {
   RenderedOp newImage = ImageUtilities.getRotatedImage(
     image, degrees);
   image.dispose();
   System.gc();
   image = newImage;
}
```

要保持系统整洁，应该消除`scaleToOneDimension`方法和`rotate`方法里面的少量重复：

```
public void scaleToOneDimension(
    float desiredDimension, float imageDimension) {
  if (Math.abs(desiredDimension - imageDimension) < errorThreshold)
     return;
   float scalingFactor = desiredDimension / imageDimension;
   scalingFactor = (float)(Math.floor(scalingFactor * 100) * 0.01f);
   replaceImage(ImageUtilities.getScaledImage(
 image, scalingFactor, scalingFactor)); 
}

public synchronized void rotate(int degrees) {
  replaceImage(ImageUtilities.getRotatedImage(image, degrees)); 
}

private void replaceImage(RenderedOp newImage) {
 image.dispose();
 System.gc();
 image = newImage;
}
```

做了一点点共性抽取后，我们意识到已经违反了SRP原则。所以，可以把一个新方法分解到另外的类中，从而提升其可见性。团队中的其他成员也许会发现进一步抽象新方法的机会，并且在其他场景中复用之。“小规模复用”可大量降低系统复杂性。要想实现大规模复用，必须理解如何实现小规模复用。

模板方法模式（Template Method）是一种移除高层级重复的通用技巧。例如：

```
public class VacationPolicy {
  public void accrueUSDivisionVacation() {
    // code to calculate vacation based on hours worked to date
    // ...
    // code to ensure vacation meets US minimums
    // ...
    // code to apply vaction to payroll record
    // ...
  }

   public void accrueEUDivisionVacation() {
     // code to calculate vacation based on hours worked to date
     // ...
     // code to ensure vacation meets EU minimums
     // ...
     // code to apply vaction to payroll record
     // ...
   }
 }
```

除了计算法定最少数量假期的部分，`accrueUSDivisionVacation`和`accrue-EuropeanDivisionVacation`中有大量代码雷同。那部分的算法，依据员工类型而变。

可以通过应用模板方法模式来消除明显的重复。

```
abstract public class VacationPolicy {
   public void accrueVacation() {
     calculateBaseVacationHours(); 
 alterForLegalMinimums(); 
 applyToPayroll(); 
   }

   private void calculateBaseVacationHours() { /* ... */ };
   abstract protected void alterForLegalMinimums();
   private void applyToPayroll() { /* ... */ };
}

public class USVacationPolicy extends VacationPolicy {
  @Override protected void alterForLegalMinimums() {
    // US specific logic
  }
}

public class EUVacationPolicy extends VacationPolicy {
  @Override protected void alterForLegalMinimums() {
    // EU specific logic
  }
}
```

子类填充了`accrueVacation`算法中的“空洞”，提供不重复的信息。

## 12.5　表达力

我们中的大多数人都经历过费解代码的纠缠。我们中的许多人自己就编写过费解的代码。写出自己能理解的代码很容易，因为在写这些代码时，我们正深入于要解决的问题中。代码的其他维护者不会那么深入，也就不易理解代码。

软件项目的主要成本在于长期的维护。为了在修改时尽量降低出现缺陷的可能性，很有必要理解系统是做什么的。当系统变得越来越复杂，开发者就需要越来越多的时间来理解它，而且也极有可能误解。所以，代码应当清晰地表达其作者的意图。作者把代码写得越清晰，其他人花在理解代码上的时间也就越少，从而减少缺陷，缩减维护成本。

可以通过选用好名称来表达。我们想要听到好类名和好函数名，而且在查看其权责时不会大吃一惊。

也可以通过保持函数和类尺寸短小来表达。短小的类和函数通常易于命名，易于编写，易于理解。

还可以通过采用标准命名法来表达。例如，设计模式很大程度上关乎沟通和表达。通过在实现这些模式的类的名称中采用标准模式名，如COMMAND或VISITOR，就能充分地向其他开发者描述你的设计。

编写良好的单元测试也具有表达性。测试的主要目的之一就是通过实例起到文档的作用。读到测试的人应该能很快理解某个类是做什么的。

不过，做到有表达力的最重要方式却是尝试。有太多时候，我们一旦写出能工作的代码，就转移到下一个问题上，而没有下足功夫调整代码，让后来者易于阅读。记住，下一位读代码的人最有可能是你自己。

所以，多少尊重一下你的手艺吧。花一点点时间在每个函数和类上。选用较好的名称，将大函数切分为小函数，时时照拂自己创建的东西。用心是最珍贵的资源。

## 12.6　尽可能少的类和方法

即便是消除重复、代码表达力和SRP等最基础的概念，也会被过度使用。为了保持类和函数短小，我们可能会造出太多的细小类和方法。所以这条规则也主张函数和类的数量要少。

类和方法的数量太多，有时是由毫无意义的教条主义导致的。例如，某个编码标准就坚称应当为每个类创建接口。也有开发者认为，字段和行为必须切分到数据类和行为类中。应该抵制这类教条，采用更实用的手段。

我们的目标是在保持函数和类短小的同时，保持整个系统短小精悍。不过要记住，这在关于简单设计的4条规则里面是优先级最低的一条。所以，尽管使类和函数的数量尽量少是很重要的，但更重要的却是测试、消除重复和表达力。

## 12.7　小结

有没有能替代经验的一套简单实践手段呢？当然不会有。另外，本章中写到的实践来自本书作者数十年经验的精练总结。遵循简单设计的实践手段，开发者不必经年学习就能掌握好的原则和模式。
