# 第14章　逐步改进

本章研究一个逐步改进的案例。你将看到一个开始还不错，但规模扩大后即出问题的模块。你还将看到这个模块是如何被重构得整洁起来的。

我们中的大多数人都会遇到解析命令行参数的情况。如果没有就手的工具，就得遍历传入`main`函数的字符串数组。有一些不同来源的好工具，但没有一个是最符合要求的。所以，我当然要自己写一个，我把它叫作`Args`。

`Args`非常易于使用。你只要简单地用输入参数和格式化字符串构造`Args`类，再向`Args`实体询问参数值即可。看看代码清单14-1中给出的简单例子。

代码清单14-1　Args的简单用法

```
public static void main(String[]args) {
  try {
    Argsarg = new Args("l,p#,d*", args);
    boolean logging = arg.getBoolean('l');
    int port = arg.getInt('p');
    String directory = arg.getString('d');
    executeApplication(logging, port, directory);
  } catch (ArgsException e) {
    System.out.printf("Argument error:%s\n", e.errorMessage());
  }
}
```

可以看到这有多简单。我们只是用两个参数创建了`Args`类的一个实体。第一个参数是格式字符串，或范式字符串`"l,p#,d*"`。它定义了3个命令行参数。第一个，`-l`，是一个布尔值参数。第二个，`-p`，是一个整数参数。第三个，`-d`，是一个字符串参数。向`Args`构造器传入的第二个参数就是向`main`传入的命令行参数数组。

如果构造器正常返回，没有抛出`ArgsException`异常，则命令行参数已传入，`Args`实体随时待命。使用`getBoolean`、`getInteger`和`getString`等方法，可以用参数名称获得参数值。

不管是格式化字符串还是命令行参数出现问题，都会抛出一个`ArgsException`异常。可以从该异常的`errorMessage`中获得关于错误的描述。

## 14.1　Args的实现

代码清单14-2是`Args`类的实现。请仔细阅读。我在代码风格和结构上花了大力气，使之值得仿效。

代码清单14-2　Args.java

```
package com.objectmentor.utilities.args;

import static com.objectmentor.utilities.args.ArgsException.ErrorCode.*;
import java.util.*;

public class Args {
  private Map<Character, ArgumentMarshaler> marshalers;

  private Set<Character> argsFound;
  private ListIterator<String> currentArgument;

  public Args(String schema, String[] args) throws ArgsException {
    marshalers = new HashMap<Character, ArgumentMarshaler>();
    argsFound = new HashSet<Character>();

    parseSchema(schema);
    parseArgumentStrings(Arrays.asList(args));
  }

  private void parseSchema(String schema) throws ArgsException {
    for (String element : schema.split(","))
      if (element.length() > 0)
        parseSchemaElement(element.trim());
  }

  private void parseSchemaElement(String element) throws ArgsException {
    char elementId = element.charAt(0);
    String elementTail = element.substring(1);
    validateSchemaElementId(elementId);
    if (elementTail.length() == 0)
      marshalers.put(elementId, new BooleanArgumentMarshaler());
    else if (elementTail.equals("*"))
      marshalers.put(elementId, new StringArgumentMarshaler());
    else if (elementTail.equals("#"))
      marshalers.put(elementId, new IntegerArgumentMarshaler());
    else if (elementTail.equals("##"))
      marshalers.put(elementId, new DoubleArgumentMarshaler());
    else if (elementTail.equals("[*]"))
      marshalers.put(elementId, new StringArrayArgumentMarshaler());
    else
      throw new ArgsException(INVALID_ARGUMENT_FORMAT, elementId, elementTail);
  }

  private void validateSchemaElementId(char elementId) throws ArgsException {
    if (!Character.isLetter(elementId))
      throw new ArgsException(INVALID_ARGUMENT_NAME, elementId, null);
  }

  private void parseArgumentStrings(List<String> argsList) throws ArgsException 
  {
    for (currentArgument = argsList.listIterator(); currentArgument.hasNext();) 
    {
      String argString = currentArgument.next();
      if (argString.startsWith("-")) {
        parseArgumentCharacters(argString.substring(1));
      } else {
        currentArgument.previous();
        break;
      }
    }
  }

  private void parseArgumentCharacters(String argChars) throws ArgsException {
    for (int i = 0; i < argChars.length(); i++)
      parseArgumentCharacter(argChars.charAt(i));
  }

  private void parseArgumentCharacter(char argChar) throws ArgsException {
    ArgumentMarshaler m = marshalers.get(argChar);
    if (m == null) {
      throw new ArgsException(UNEXPECTED_ARGUMENT, argChar, null);
    } else {
      argsFound.add(argChar);
      try {
        m.set(currentArgument);
      } catch (ArgsException e) {
        e.setErrorArgumentId(argChar);
        throw e;
      }
    }
  }

  public boolean has(char arg) {
    return argsFound.contains(arg);
  }

  public int nextArgument() {
    return currentArgument.nextIndex();
  }

  public boolean getBoolean(char arg) {
    return BooleanArgumentMarshaler.getValue(marshalers.get(arg));
  }

  public String getString(char arg) {
    return StringArgumentMarshaler.getValue(marshalers.get(arg));
  }

  public int getInt(char arg) {
    return IntegerArgumentMarshaler.getValue(marshalers.get(arg));
  }

  public double getDouble(char arg) {
    return DoubleArgumentMarshaler.getValue(marshalers.get(arg));
  }

  public String[] getStringArray(char arg) {
    return StringArrayArgumentMarshaler.getValue(marshalers.get(arg));
  }
}
```

注意，你可以从上到下阅读这些代码，不用跳来跳去，也不用先看后面的部分。唯一需要先看的是`ArgumentMarshaler`的定义，这部分我有意省略了。仔细看这段代码，你应该能理解`ArgumentMarshaler`接口是什么，其派生类做什么。下面我将向你展示一部分（如代码清单14-3～代码清单14-6所示）。

代码清单14-3　ArgumentMarshaler.java

```
public  interface ArgumentMarshaler  {
   void  set(Iterator<String>  currentArgument) throws ArgsException;
}
```

代码清单14-4　BooleanArgumentMarshaler.java

```
public class BooleanArgumentMarshaler implements ArgumentMarshaler {
  private boolean booleanValue = false;

  public void set(Iterator<String> currentArgument) throws ArgsException {
    booleanValue = true;
  }

  public static boolean getValue(ArgumentMarshaler am) {
    if (am != null && am instanceof BooleanArgumentMarshaler)
      return ((BooleanArgumentMarshaler) am).booleanValue;
    else
      return false;
  }
}
```

代码清单14-5　StringArgumentMarshaler.java

```
import static com.objectmentor.utilities.args.ArgsException.ErrorCode.*;

public class StringArgumentMarshaler implements ArgumentMarshaler {
  private String stringValue = "";

  public void set(Iterator<String> currentArgument) throws ArgsException {
    try {
      stringValue = currentArgument.next();
    } catch (NoSuchElementException e) {
      throw new ArgsException(MISSING_STRING);
    }
  }

  public static String getValue(ArgumentMarshaler am) {
    if (am != null && am instanceof StringArgumentMarshaler)
      return ((StringArgumentMarshaler) am).stringValue;
    else
      return "";
  }
}
```

代码清单14-6　IntegerArgumentMarshaler.java

```
import static com.objectmentor.utilities.args.ArgsException.ErrorCode.*;

public class IntegerArgumentMarshaler implements ArgumentMarshaler {
  private int intValue = 0;

  public void set(Iterator<String> currentArgument) throws ArgsException {
    String parameter = null;
    try {
      parameter = currentArgument.next();
      intValue = Integer.parseInt(parameter);
    } catch (NoSuchElementException e) {
      throw new ArgsException(MISSING_INTEGER);
    } catch (NumberFormatException e) {
      throw new ArgsException(INVALID_INTEGER, parameter);
    }
  }

  public static int getValue(ArgumentMarshaler am) {
    if (am != null && am instanceof IntegerArgumentMarshaler)
      return ((IntegerArgumentMarshaler) am).intValue;
    else
      return 0;
  }
}
```

`ArgumentMarshaler`的其他派生类以同样的模式处理`double`和`String`数组，一一列出反而阻碍行文。你可以练习自己实现它们。

还有些信息可能会困扰你：错误码常量的定义。这些是在`ArgsException`类（代码清单14-7）中定义的。

代码清单14-7　ArgsException.java

```
import static com.objectmentor.utilities.args.ArgsException.ErrorCode.*;

public class ArgsException extends Exception {
  private char errorArgumentId = '\0';
  private String errorParameter = null;
  private ErrorCode errorCode = OK;

  public ArgsException() {}

  public ArgsException(String message) {  super(message);}

  public ArgsException(ErrorCode errorCode) {
    this.errorCode = errorCode;
  }

  public ArgsException(ErrorCode errorCode, String errorParameter) {
    this.errorCode = errorCode;
    this.errorParameter = errorParameter;
  }

  public ArgsException(ErrorCode errorCode, 
                       char errorArgumentId,String errorParameter) {
    this.errorCode = errorCode;
    this.errorParameter = errorParameter;
    this.errorArgumentId = errorArgumentId;
  }

  public char getErrorArgumentId() {
    return errorArgumentId;
  }

  public void setErrorArgumentId(char errorArgumentId) {
    this.errorArgumentId = errorArgumentId;
  }

  public String getErrorParameter() {
    return errorParameter;
  }

  public void setErrorParameter(String errorParameter) {
    this.errorParameter = errorParameter;
  }

  public ErrorCode getErrorCode() {
    return errorCode;
  }

  public void setErrorCode(ErrorCode errorCode) {
    this.errorCode = errorCode;
  }

  public String errorMessage() {
    switch (errorCode) {
      case OK:
        return "TILT:  Should  not  get  here.";
      case UNEXPECTED_ARGUMENT:
        return String.format("Argument  -%c  unexpected.", errorArgumentId);
      case MISSING_STRING:
        return String.format("Could not find string parameter for -%c.",
                             errorArgumentId);
      case INVALID_INTEGER:
        return String.format("Argument  -%c expects an  integer  but  was  '%s'.",
                             errorArgumentId, errorParameter);
      case MISSING_INTEGER:
        return String.format("Could not find integer parameter for -%c.",
                             errorArgumentId);
      case INVALID_DOUBLE:
        return String.format("Argument -%c expects a double but was '%s'.",
                             errorArgumentId, errorParameter);
      case MISSING_DOUBLE:
        return String.format("Could not find double parameter for -%c.",
                             errorArgumentId);
      case INVALID_ARGUMENT_NAME:
        return String.format("'%c' is not a valid argument name.",
                             errorArgumentId);

      case INVALID_ARGUMENT_FORMAT:
        return String.format("'%s' is not a valid argument format.",
                             errorParameter);
    }
    return "";
  }

  public enum ErrorCode {
    OK, INVALID_ARGUMENT_FORMAT, UNEXPECTED_ARGUMENT, INVALID_ARGUMENT_NAME,
    MISSING_STRING, 
    MISSING_INTEGER, INVALID_INTEGER, 
    MISSING_DOUBLE, INVALID_DOUBLE
}
```

为了充实这么一个简单概念的细节，需要如此多代码，这很值得注意。原因之一是我们使用了Java这种唠叨型语言。作为一种静态类型语言，需要大量语句才能满足类型系统的要求。在Ruby、Python或Smalltalk等语言中，程序会短很多。

请再次阅读这段代码。特别注意命名方式、函数大小和代码格式。如果你是经验丰富的程序员，可能会对风格或结构有着这样或那样的不同观点。不过，希望你认为这段程序总体上编写良好，有着整洁的结构。

例如，如何增加新参数类型，如日期或复杂数字参数。其实现手段很清楚，而且只需要花一点点力气即可。简言之，只需要从`ArgumentMarshaler`派生一个新类，写一个新的`getXXX`函数，在`parseSchemaElement`函数中添加一个新的`case`语句。可能还需要添加新的`ArgsException.ErrorCode`和新错误信息。

### 我怎么做的

先放松一下神经。这段程序并非从一开始就写成了现在的样子。更重要的是，我也没指望你能够一次就写出整洁、漂亮的程序。如果说我们从过去几十年里学到什么东西的话，那就是编程是一种技艺甚于科学的东西。要编写整洁代码，必须先写肮脏代码，然后再清理它。

你应该不会对此感到惊讶。我们在小学就学过这条真理了。那时，老师（通常是徒劳地）努力让我们写作文草稿。他们告诉我们，我们应该先写草稿，再写二稿，一次又一次地草撰，直至写出终稿。他们尽力告诉我们，写出好作文是一个逐步改进的过程。

多数新手程序员（就像多数小学生一样）没有特别认真地遵循这个建议。他们相信，首要任务是写出能工作的程序。只要程序“能工作”，就转移到下一个任务上，而那个“能工作”的程序就留在了最后那个所谓“能工作”的状态。多数有经验的程序员都知道，这是一种自毁行为。

## 14.2　Args：草稿

代码清单14-8展示了Args类的一个早期版本。它“能工作”，但却很烂。

代码清单14-8　Args.java（初稿）

```
import java.text.ParseException;
import java.util.*;

public class Args {
  private String schema;
  private String[] args;
  private boolean valid = true;
  private Set<Character> unexpectedArguments = new TreeSet<Character>();
  private Map<Character, Boolean> booleanArgs = 
    new HashMap<Character, Boolean>();
  private Map<Character, String> stringArgs = new HashMap<Character, String>();
  private Map<Character, Integer> intArgs = new HashMap<Character, Integer>();
  private Set<Character> argsFound = new HashSet<Character>();
  private int currentArgument;
  private char errorArgumentId = '\0';
  private String errorParameter = "TILT";
  private ErrorCode errorCode = ErrorCode.OK;

  private enum ErrorCode {
    OK, MISSING_STRING, MISSING_INTEGER, INVALID_INTEGER, UNEXPECTED_ARGUMENT}

  public Args(String schema, String[] args) throws ParseException {
    this.schema = schema;
    this.args = args;
    valid = parse();
  }

  private boolean parse() throws ParseException {
    if (schema.length() == 0 && args.length == 0)
      return true;
    parseSchema();
    try {
      parseArguments();
    } catch (ArgsException e) {
    }
    return valid;
  }

  private boolean parseSchema() throws ParseException {
    for (String element : schema.split(",")) {

      if (element.length() > 0) {
        String trimmedElement = element.trim();
        parseSchemaElement(trimmedElement);
      }
    }
    return true;
  }

  private void parseSchemaElement(String element) throws ParseException {
    char elementId = element.charAt(0);
    String elementTail = element.substring(1);
    validateSchemaElementId(elementId);
    if (isBooleanSchemaElement(elementTail))
      parseBooleanSchemaElement(elementId);
    else if (isStringSchemaElement(elementTail))
      parseStringSchemaElement(elementId);
    else if (isIntegerSchemaElement(elementTail)) {
      parseIntegerSchemaElement(elementId);
    } else {
      throw new ParseException
        (String.format("Argument:  %c has invalid format: %s.",
                       elementId,elementTail),0);
    }
  }

  private void validateSchemaElementId(char elementId) throws ParseException {
    if (!Character.isLetter(elementId)) {
      throw new ParseException(
        "Bad character:" + elementId + "in Args format: "+schema,0);
    }
  }

  private void parseBooleanSchemaElement(char elementId) {
    booleanArgs.put(elementId, false);
  }

  private void parseIntegerSchemaElement(char elementId) {
    intArgs.put(elementId, 0);
  }

  private void parseStringSchemaElement(char elementId) {
    stringArgs.put(elementId, "");
  }

  private boolean isStringSchemaElement(String elementTail) {
    return elementTail.equals("*");
  }

  private boolean isBooleanSchemaElement(String elementTail) {
    return elementTail.length() == 0;
  }

  private boolean isIntegerSchemaElement(String elementTail) {
    return elementTail.equals("#");
  }

  private boolean parseArguments() throws ArgsException {
    for (currentArgument = 0; currentArgument < args.length; currentArgument++) {

      String arg = args[currentArgument];
      parseArgument(arg);
    }
    return true;
  }

  private void parseArgument(String arg) throws ArgsException {
    if (arg.startsWith("-"))
      parseElements(arg);
  }

  private void parseElements(String arg) throws ArgsException {
    for (int i = 1; i < arg.length(); i++)
      parseElement(arg.charAt(i));
  }

  private void parseElement(char argChar) throws ArgsException {
    if (setArgument(argChar))
      argsFound.add(argChar);
    else {
      unexpectedArguments.add(argChar);
      errorCode = ErrorCode.UNEXPECTED_ARGUMENT;
      valid = false;
    }
  }

  private boolean setArgument(char argChar) throws ArgsException {
    if (isBooleanArg(argChar))
      setBooleanArg(argChar, true);
    else if (isStringArg(argChar))
      setStringArg(argChar);
    else if (isIntArg(argChar))
      setIntArg(argChar);
    else
      return false;

    return true;
  }

  private boolean isIntArg(char argChar) {return intArgs.containsKey(argChar);}

  private void setIntArg(char argChar) throws ArgsException {
    currentArgument++;
    String parameter = null;
    try {
      parameter = args[currentArgument];
      intArgs.put(argChar, new Integer(parameter));
    } catch (ArrayIndexOutOfBoundsException e) {
      valid = false;
      errorArgumentId = argChar;
      errorCode = ErrorCode.MISSING_INTEGER;

      throw new ArgsException();
    } catch (NumberFormatException e) {
      valid = false;
      errorArgumentId = argChar;
      errorParameter = parameter;
      errorCode = ErrorCode.INVALID_INTEGER;
      throw new ArgsException();
    }
  }

  private void setStringArg(char argChar) throws ArgsException {
    currentArgument++;
    try {
      stringArgs.put(argChar, args[currentArgument]);
    } catch (ArrayIndexOutOfBoundsException e) {
      valid = false;
      errorArgumentId = argChar;
      errorCode = ErrorCode.MISSING_STRING;
      throw new ArgsException();
    }
  }

  private boolean isStringArg(char argChar) {
    return stringArgs.containsKey(argChar);
  }

  private void setBooleanArg(char argChar, boolean value) {
    booleanArgs.put(argChar, value);
  }

  private boolean isBooleanArg(char argChar) {
    return booleanArgs.containsKey(argChar);
  }

  public int cardinality() {
    return argsFound.size();
  }

  public String usage() {
    if (schema.length() > 0)
      return "-[" + schema + "]";
    else
      return "";
  }

  public String errorMessage() throws Exception {
    switch (errorCode) {
      case OK:
        throw new Exception("TILT:  Should  not  get  here.");
      case UNEXPECTED_ARGUMENT:
        return unexpectedArgumentMessage();
      case MISSING_STRING:
        return String.format("Could  not  find  string  parameter  for  -%c.",
                             errorArgumentId);

      case INVALID_INTEGER:
        return String.format("Argument  -%c expects an  integer  but  was  '%s'.",
                             errorArgumentId, errorParameter);
      case MISSING_INTEGER:
        return String.format("Could  not  find  integer  parameter  for  -%c.",
                             errorArgumentId);
    }
    return "";
  }

  private String unexpectedArgumentMessage() {
    StringBuffer message = new StringBuffer("Argument(s)  -");
    for (char c : unexpectedArguments) {
      message.append(c);
    }
    message.append("  unexpected.");

    return message.toString();
  }

  private boolean falseIfNull(Boolean b) {
    return b != null && b;
  }

  private int zeroIfNull(Integer i) {
    return i == null ? 0 : i;
  }

  private String blankIfNull(String s) {
    return s == null ? "" : s;
  }

  public String getString(char arg) {
    return blankIfNull(stringArgs.get(arg));
  }

  public int getInt(char arg) {
    return zeroIfNull(intArgs.get(arg));
  }

  public boolean getBoolean(char arg) {
    return falseIfNull(booleanArgs.get(arg));
  }

  public boolean has(char arg) {
    return argsFound.contains(arg);
  }

  public boolean isValid() {
    return valid;
  }

  private class ArgsException extends Exception {
  }
}
```

希望你看到这段乱七八糟的代码时，第一反应是“他没就此罢手，真令人高兴！”如果你这么想，不如想想其他人对你留置在草稿形态的代码的想法吧。

实际上，“草稿”大概会是你对这段代码的最高评价。它显然还需打磨。实体变量的数量多到吓人，诸如`"TILT"`之类奇怪的字符串、`HashSets`和`TreeSets`，还有那些`try-catch-catch`代码块，组成了一个烂摊子。

我不想写出一个烂摊子。我也一直想保持一切有序。从函数和变量命名，以及程序的粗略架构中，你可以看出这一点。不过，显然我没能做到。

混乱是逐渐产生的。更早的版本并不如此肮脏。例如，代码清单14-9展示了一个早期版本代码，那时只支持`Boolean`类型参数。

代码清单14-9　Args.java（只支持布尔类型）

```
package com.objectmentor.utilities.getopts;

import java.util.*;

public class Args {
  private String schema;
  private String[] args;
  private boolean valid;
  private Set<Character> unexpectedArguments = new TreeSet<Character>();
  private Map<Character, Boolean> booleanArgs = 
    new HashMap<Character, Boolean>();
  private int numberOfArguments = 0;

  public Args(String schema, String[] args) {
    this.schema = schema;
    this.args = args;
    valid = parse();
  }

  public boolean isValid() {
    return valid;
  }

  private boolean parse() {
    if (schema.length() == 0 && args.length == 0)
      return true;
    parseSchema();
    parseArguments();
    return unexpectedArguments.size() == 0;
  }

  private boolean parseSchema() {
    for (String element : schema.split(",")) {
      parseSchemaElement(element);
    }

    return true;
  }

  private void parseSchemaElement(String element) {
    if (element.length() == 1) {
      parseBooleanSchemaElement(element);
    }
  }

  private void parseBooleanSchemaElement(String element) {
    char c = element.charAt(0);
    if (Character.isLetter(c)) {
      booleanArgs.put(c, false);
    }
  }

  private boolean parseArguments() {
    for (String arg : args)
      parseArgument(arg);
    return true;
  }

  private void parseArgument(String arg) {
    if (arg.startsWith("-"))
      parseElements(arg);
  }

  private void parseElements(String arg) {
    for (int i = 1; i < arg.length(); i++)
      parseElement(arg.charAt(i));
  }

  private void parseElement(char argChar) {
    if (isBoolean(argChar)) {
      numberOfArguments++;
      setBooleanArg(argChar, true);
    } else
      unexpectedArguments.add(argChar);
  }

  private void setBooleanArg(char argChar, boolean value) {
    booleanArgs.put(argChar, value);
  }

  private boolean isBoolean(char argChar) {
    return booleanArgs.containsKey(argChar);
  }

  public int cardinality() {
    return numberOfArguments;
  }

  public String usage() {
    if (schema.length() > 0)
       return "-["+schema+"]";
    else
      return "";
  }

  public String errorMessage() {
    if (unexpectedArguments.size() > 0) {
      return unexpectedArgumentMessage();
    } else
      return "";
  }

  private String unexpectedArgumentMessage() {
    StringBuffer message = new StringBuffer("Argument(s)  -");
    for (char c : unexpectedArguments) {
      message.append(c);
    }
    message.append("  unexpected.");

    return message.toString();
  }

  public boolean getBoolean(char arg) {
    return booleanArgs.get(arg);
  }
}
```

尽管你可能对这段代码很不满意，但是其实它并非如此糟糕。它精练、简单，易于理解。然而，在这段代码中很容易找到后面烂摊子的根源，能很清楚看到小问题是如何变成大混乱的。

注意，后面的混乱代码只比这个版本多支持两种参数类型：`String`和`integer`。只增加两种参数类型支持，就对代码产生了如此巨大的负面影响。它从某种可维护之物变成了满是缺陷的东西。

我逐步添加了对这两种参数类型的支持。首先，我添加对`String`参数的支持，如代码清单14-10所示。

代码清单14-10　Args.java（Boolean和String）

```
package com.objectmentor.utilities.getopts;

import java.text.ParseException;
import java.util.*;

public class Args {
  private String schema;
  private String[] args;
  private boolean valid = true;
  private Set<Character> unexpectedArguments = new TreeSet<Character>();
  private Map<Character, Boolean> booleanArgs = 
    new HashMap<Character, Boolean>();

  private Map<Character, String> stringArgs = 
    new HashMap<Character, String>();
  private Set<Character> argsFound = new HashSet<Character>();
  private int currentArgument;
  private char errorArgument = '\0';

  enum ErrorCode {
    OK, MISSING_STRING}

  private ErrorCode errorCode = ErrorCode.OK;

  public Args(String schema, String[] args) throws ParseException {
    this.schema = schema;
    this.args = args;
    valid = parse();
  }

  private boolean parse() throws ParseException {
    if (schema.length() == 0 && args.length == 0)
      return true;
    parseSchema();
    parseArguments();
    return valid;
  }

  private boolean parseSchema() throws ParseException {
    for (String element : schema.split(",")) {
      if (element.length() > 0) {
        String trimmedElement = element.trim();
        parseSchemaElement(trimmedElement);
      }
    }
    return true;
  }

  private void parseSchemaElement(String element) throws ParseException {
    char elementId = element.charAt(0);
    String elementTail = element.substring(1);
    validateSchemaElementId(elementId);
    if (isBooleanSchemaElement(elementTail))
      parseBooleanSchemaElement(elementId);
    else if (isStringSchemaElement(elementTail))
      parseStringSchemaElement(elementId);
  }

  private void validateSchemaElementId(char elementId) throws ParseException {
    if (!Character.isLetter(elementId)) {
      throw new ParseException(
        "Bad  character:" + elementId + "in Args  format:  " + schema, 0);
    }

  }

  private void parseStringSchemaElement(char elementId) {
    stringArgs.put(elementId, "");
  }

  private boolean isStringSchemaElement(String elementTail) {
    return elementTail.equals("*");
  }

  private boolean isBooleanSchemaElement(String elementTail) {
    return elementTail.length() == 0;
  }

  private void parseBooleanSchemaElement(char elementId) {
    booleanArgs.put(elementId, false);
  }

  private boolean parseArguments() {
    for (currentArgument = 0; currentArgument < args.length; currentArgument++) 
    {
      String arg = args[currentArgument];
      parseArgument(arg);
    }
    return true;
  }

  private void parseArgument(String arg) {
    if (arg.startsWith("-"))
      parseElements(arg);
  }

  private void parseElements(String arg) {
    for (int i = 1; i < arg.length(); i++)
      parseElement(arg.charAt(i));
  }

  private void parseElement(char argChar) {
    if (setArgument(argChar))
      argsFound.add(argChar);
    else {
      unexpectedArguments.add(argChar);
      valid = false;
    }
  }

  private boolean setArgument(char argChar) {
    boolean set = true;
    if (isBoolean(argChar))
      setBooleanArg(argChar, true);
    else if (isString(argChar))
      setStringArg(argChar, "");
    else
      set = false;

    return set;
  }

  private void setStringArg(char argChar, String s) {
    currentArgument++;
    try {

      stringArgs.put(argChar, args[currentArgument]);
    } catch (ArrayIndexOutOfBoundsException e) {
      valid = false;
      errorArgument = argChar;
      errorCode = ErrorCode.MISSING_STRING;
    }
  }

  private boolean isString(char argChar) {
    return stringArgs.containsKey(argChar);
  }

  private void setBooleanArg(char argChar, boolean value) {
    booleanArgs.put(argChar, value);
  }

  private boolean isBoolean(char argChar) {
    return booleanArgs.containsKey(argChar);
  }

  public int cardinality() {
    return argsFound.size();
  }

  public String usage() {
    if (schema.length() > 0)
      return "-[" + schema + "]";
    else
      return "";
  }

  public String errorMessage() throws Exception {
    if (unexpectedArguments.size() > 0) {
      return unexpectedArgumentMessage();
    } else
      switch (errorCode) {
        case MISSING_STRING:
          return String.format("Could  not  find  string  parameter  for  -%c.", 
                               errorArgument);
        case OK:
          throw new Exception("TILT:  Should  not  get  here.");
      }
    return "";
  }

  private String unexpectedArgumentMessage() {
    StringBuffer message = new StringBuffer("Argument(s)  -");
    for (char c : unexpectedArguments) {
      message.append(c);
    }
    message.append("  unexpected.");

    return message.toString();
  }

  public boolean getBoolean(char arg) {
    return falseIfNull(booleanArgs.get(arg));
  }

  private boolean falseIfNull(Boolean b) {
    return b == null ? false : b;
  }

  public String getString(char arg) {
    return blankIfNull(stringArgs.get(arg));
  }

  private String blankIfNull(String s) {
    return s == null ? "" : s;
  }

  public boolean has(char arg) {
    return argsFound.contains(arg);
  }

  public boolean isValid() {
    return valid;
  }
}
```

你可以看到，代码开始失去控制，这还算不上可怕，可怕的是混乱已经开始生长，已经出现了一堆东西，不过还没烂掉。增加对整数参数类型的支持后，那堆东西才真的变质腐烂了。

### 14.2.1　所以我暂停了

还有至少两种参数类型要添加，而且情形一定会更加糟糕。如果一味蛮干，大概也能让它工作，不过就会留下一大堆要调整的混乱代码。如果希望代码结构一直可维护，现在正是调整的时机。

所以我暂停添加特性，开始重构。由于刚添加了`String`类型和`integer`类型参数，我知道每种参数类型都需要在3个主要位置增加新代码。首先，每种参数类型都要解析其范式元素，从而为该种类型选择`HashMap`的方法。其次，每种参数类型都需要在命令行字符串中解析，然后再转换为真实类型。最后，每种参数类型都需要一个`getXXX`方法，并按照其真实类型向调用者返回参数值。

许多种不同类型，类似的方法——听起来像是一个类。`ArgumentMarshaler`的概念就是这样产生的。

### 14.2.2　渐进

毁坏程序的最好方法之一就是以改进之名大动其结构。有些程序永远不能从这种所谓“改进”中恢复过来。问题在于，很难让程序以“改进”之前的方式工作。

为了避免这种状况发生，我采用了测试驱动开发的规程，这种方法的核心原则之一是保持系统始终能运行。换言之，采用TDD，我不会允许做出破坏系统的修改，每次修改都必须保证系统能像以前一样工作。

我需要一套能随需运行，确保系统行为不会改动的自动化测试。在我搞出那个烂摊子的同时，也为Args类创建了一套单元测试和验收测试。单元测试用Java写成，采用JUnit管理。验收测试用FitNesse以wiki页形式写成。我可以随时运行这些测试，如果测试通过，就能打包票说系统能以我期望的方式工作。

于是我开始做出大量小规模修改。每次修改都将系统结构向`ArgumentMarshaler`概念的方向推动，而且每次修改后，系统都要能正常工作。第一个修改是在烂摊子末尾添加`ArgumentMarshaler`的轮廓，如代码清单14-11所示。

代码清单14-11　向Args.java添加ArgumentMarshaler

```
private  class ArgumentMarshaler  {
  private  boolean  booleanValue  =  false;

  public  void  setBoolean(boolean  value)  {
    booleanValue  =  value;
  }

  public  boolean  getBoolean()  {return  booleanValue;}
}

private  class  BooleanArgumentMarshaler extends ArgumentMarshaler  {
}

private  class  StringArgumentMarshaler extends ArgumentMarshaler  {
}

private  class  IntegerArgumentMarshaler extends ArgumentMarshaler  {
}
```

显然，这什么也不会破坏。于是我做了一点最简单的、破坏性尽可能小的修改。我修改了`HashMap`，采用`ArgumentMarshaler`，使之支持`Boolean`类型参数。

```
private  Map<Character, ArgumentMarshaler>  booleanArgs  =
  new  HashMap<Character, ArgumentMarshaler>();
```

这个修改影响少数语句，我很快就修正了。

```
...
  private  void  parseBooleanSchemaElement(char elementId)  {
    booleanArgs.put(elementId, new BooleanArgumentMarshaler());
  }
...
  private  void  setBooleanArg(char argChar, boolean  value)  {
    booleanArgs.get(argChar).setBoolean(value);
  }
...
  public  boolean  getBoolean(char arg)  {
    return  falseIfNull(booleanArgs.get(arg).getBoolean());
}
```

注意，这些修改正是在我之前提到的那些区域之内所做的：参数类型的`parse`、`set`和`get`操作。不幸的是，即便修改如此细微，有些测试也还是会失败。仔细看`getBoolean`，可以看到如果用`y`去调用，但没有`y`这个参数，则`booleanArgs.get('y')`就会返回`null`值，函数将抛出`NullPointerException`异常。函数`falseIfNull`用以防止这种状况发生，但我做出的修改却导致该函数无所作为。

渐进主义要求我在做其他修改之前迅速修正这个问题。修正并不费劲。我只是把对`null`值的检查移了个位置。再也不用检测`bollean`是否为`null`，而是检查`ArgumentMarshaler`是否为`null`。

首先，我移除了`getBoolean`函数中的`falseIfNull`调用。现在它没什么用了，所以我也删除了这个函数。测试还是以同样的方式失败，所以我确定没有引入新的错误。

```
public  boolean  getBoolean(char arg)  {
  return  booleanArgs.get(arg).getBoolean();
}
```

下一步，我把函数拆解为两行，并把`ArgumentMarshaler`放到它自己的名为`argumentMarshaler`的变量中（即创建一个类型为`ArgumentMarshaler`的对象实体）。我不在意变量名太长，但它确实有点啰唆，把函数搞得支离破碎。所以我把变量名缩短为`am`[N5]。

```
public  boolean  getBoolean(char arg)  { 
  Args.ArgumentMarshaler am  =  booleanArgs.get(arg); 
  return am.getBoolean();
}
```

然后再放入检测null值的逻辑。

```
public  boolean  getBoolean(char arg)  {
  Args.ArgumentMarshaler am  =  booleanArgs.get(arg); 
  return am != null && am.getBoolean();
}
```

## 14.3　字符串类型参数

添加`String`类型参数和添加`boolean`类型参数非常像。我要修改`HashMap`，让`parse`、`set`和`get`函数能工作。跟着就是按部就班，但我似乎该把所有的marshalling（编组）实现放到`ArgumentMarshaler`基类而不是派生类中。

```
  private  Map<Character, ArgumentMarshaler>  stringArgs  =
      new  HashMap<Character, ArgumentMarshaler>();
...
  private  void  parseStringSchemaElement(char elementId)  {
    stringArgs.put(elementId, new StringArgumentMarshaler());
  }
...
  private  void  setStringArg(char argChar) throws ArgsException  {
    currentArgument++;
    try  {
      stringArgs.get(argChar).setString(args[currentArgument]);
    } catch (ArrayIndexOutOfBoundsException e)  {
      valid  =  false;
      errorArgumentId  = argChar;
      errorCode  =  ErrorCode.MISSING_STRING;
      throw  new ArgsException();
    }
  }
...
  public  String  getString(char arg)  { 
    Args.ArgumentMarshaler am  =  stringArgs.get(arg); 
    return am == null ? ""  : am.getString();
  }
...
  private  class ArgumentMarshaler  {
    private  boolean  booleanValue  =  false; 
    private String stringValue;

    public  void  setBoolean(boolean  value)  {
      booleanValue  =  value;
    }

    public  boolean  getBoolean()  {
      return  booleanValue;
    }

    public void setString(String s) {
     stringValue = s;
    }

    public String getString() {
      return stringValue == null ? "" : stringValue;
    }
  }
```

同样，也是每次修改一个地方，持续运行测试。如果测试出错，则在做下一个修改前确保通过。

现在你应该明白我的意图了。一旦我将当前的编组行为放到`ArgumentMarshaler`基类中，就会开始往派生类推入该行为。这样，在我逐渐修改程序的形状时，还能保持一切正常。

下一步显而易见，把`int`参数的相关功能放到`ArgumentMarshaler`里面。同样，也是照方抓药。

```
private  Map<Character, ArgumentMarshaler>  intArgs  =
    new  HashMap<Character, ArgumentMarshaler>();
...
  private  void  parseIntegerSchemaElement(char elementId)  {
    intArgs.put(elementId, new IntegerArgumentMarshaler());
  }
...
  private  void  setIntArg(char argChar) throws ArgsException  {
    currentArgument++;
    String  parameter  =  null;
    try  {
      parameter  = args[currentArgument];
      intArgs.get(argChar).setInteger(Integer.parseInt(parameter));
    } catch (ArrayIndexOutOfBoundsException e)  {
      valid  =  false;
      errorArgumentId  = argChar;
      errorCode  =  ErrorCode.MISSING_INTEGER;
      throw  new ArgsException();
    } catch (NumberFormatException e)  {
      valid  =  false; 
      errorArgumentId  = argChar; 
      errorParameter  =  parameter;
      errorCode  =  ErrorCode.INVALID_INTEGER;
      throw  new ArgsException();
    }
  }
...
  public  int  getInt(char arg)  { 
   Args.ArgumentMarshaler am  =  intArgs.get(arg); 
    return am == null ? 0 : am.getInteger();
  }
...
  private  class ArgumentMarshaler  { 
    private  boolean  booleanValue  =  false; 
    private  String  stringValue;
    private int integerValue;

    public  void  setBoolean(boolean  value)  {
      booleanValue  =  value;
    }

    public  boolean  getBoolean()  {
      return  booleanValue;
    }

    public  void  setString(String  s)  {
      stringValue  =  s;
    }

    public  String  getString()  {
      return  stringValue  ==  null ? ""  :  stringValue;
    }

    public void setInteger(int i) {
      integerValue = i;
    }

    public int getInteger() {
      return integerValue;
    }
  }
```

当所有的编组操作都放到了`ArgumentMarshaler`中，我开始向派生类移植功能。第一步是把`setBoolean`函数放到`BooleanArgumentMarshaler`中，确保它能被正确调用。所以我创建了一个抽象的`set`方法。

```
private abstract  class ArgumentMarshaler  { 
  protected  boolean  booleanValue  =  false;
  private  String  stringValue;
  private  int  integerValue;

  public  void  setBoolean(boolean  value)  {
    booleanValue  =  value;
  }

  public  boolean  getBoolean()  {
    return  booleanValue;
  }

  public  void  setString(String  s)  {
    stringValue  =  s;
  }

  public  String  getString()  {
    return  stringValue  ==  null ? ""  :  stringValue;
  }

  public  void  setInteger(int  i)  {
    integerValue  =  i;
  }

  public  int  getInteger()  {
    return  integerValue;
  }

  public abstract void set(String s);
}
```

然后在`BooleanArgumentMarshaler`中实现`set`方法。

```
private  class  BooleanArgumentMarshaler extends ArgumentMarshaler  {
  public void set(String s) {
 booleanValue = true;
 }
}
```

最后，通过调用`set`，替换对`setBoolean`函数的调用。

```
private  void  setBooleanArg(char argChar, boolean  value)  {
  booleanArgs.get(argChar).set("true");
}
```

测试仍然全部通过。因为这次修改将`set`函数放到了`BooleanArgumentMarshaler`里面，所以我从`ArgumentMarshaler`基类删除了`setBoolean`方法。

注意，抽象函数`set`有一个`String`类型参数，但其在`BooleanArgumentMarshaler`中的实现却没有使用这个参数。之所以在这里放这个参数，是因为我知道`StringArgumentMarshaler`和`IntegerArgumentMarshaler`可能会使用它。

接着，我打算把`get`方法放到`BooleanArgumentMarshaler`中。这有点难看，因为返回类型必须是`Object`，且在这里需要转换为`Boolean`类型值。

```
public  boolean  getBoolean(char arg)  { 
  Args.ArgumentMarshaler am  =  booleanArgs.get(arg); 
  return am  !=  null  && (Boolean)am.get();
}
```

为了编译通过，我把`get`函数加到`ArgumentMarshaler`中。

```
private abstract  class ArgumentMarshaler  {
  ...

  public Object get() {
   return null;
  }
}
```

这样一来，虽然可以编译，但却无法通过测试。只要将`get`修改为抽象方法，并在`BooleanArgumentMarshaler`中实现，就能重新通过测试。

```
private abstract  class ArgumentMarshaler  {
  protected  boolean  booleanValue  =  false;
  ...

  public abstract  Object  get();
}

  private  class  BooleanArgumentMarshaler extends ArgumentMarshaler  {
   public  void  set(String  s)  {
    booleanValue  = true;
    }

    public Object get() {
     return booleanValue;
    }
  }
```

测试又通过了。`get`方法和`set`方法都已部署到`BooleanArgumentMarshaler`中！这样我就可以从`ArgumentMarshaler`里面移除旧的`getBoolean`函数，把受保护的`booleanValue`变量向下移动到`BooleanArgumentMarshaler`，并将其设置为`private`。

对于`String`类型也照此处理，即修改`set`和`get`的部署方式，删除无用的函数，并移动了变量。

```
private  void  setStringArg(char argChar) throws ArgsException  {
 currentArgument++;
 try  {
 stringArgs.get(argChar).set(args[currentArgument]);
 } catch (ArrayIndexOutOfBoundsException e)  {
 valid  =  false;
 errorArgumentId  = argChar;
 errorCode  =  ErrorCode.MISSING_STRING;
 throw  new ArgsException();
  }
}
...
  public  String  getString(char arg)  { 
   Args.ArgumentMarshaler am  =  stringArgs.get(arg); 
   return am  ==  null ? ""  : (String) am.get();
  }
...
  private abstract  class ArgumentMarshaler  {
   private  int  integerValue;

   public  void  setInteger(int  i)  {
    integerValue  =  i;
    }

   public  int  getInteger()  {
    return  integerValue;
   }

   public abstract  void  set(String  s);

   public abstract  Object  get();
  }

  private  class  BooleanArgumentMarshaler extends ArgumentMarshaler  {
   private boolean booleanValue = false;

    public  void  set(String  s)  {
    booleanValue  = true;
   }

   public  Object  get()  {
      return  booleanValue;
   }
  }

  private  class  StringArgumentMarshaler extends ArgumentMarshaler  {
   private String stringValue = "";

   public  void  set(String  s)  {
    stringValue = s;
   }

   public  Object  get()  {
     return stringValue;
    }
  }

  private  class  IntegerArgumentMarshaler extends ArgumentMarshaler  {

    public  void  set(String  s)  {

    }

    public  Object  get()  {
      return  null;
    }
  }
}
```

最后，我为`integer类`型参数重复上述过程。这稍稍复杂一点，因为`integer`需要解析，而`parse`操作会抛出异常。不过结果会更好，因为`NumberFormatException`的概念在`IntegerArgumentMarshaler`中隐藏了。

```
private  boolean  isIntArg(char argChar)  {return  intArgs.containsKey(argChar);}

  private  void  setIntArg(char argChar) throws ArgsException  {
    currentArgument++;
    String  parameter  =  null;
    try  {
      parameter  = args[currentArgument];
      intArgs.get(argChar).set(parameter);
    } catch (ArrayIndexOutOfBoundsException e)  {
      valid  =  false;
      errorArgumentId  = argChar;
      errorCode  =  ErrorCode.MISSING_INTEGER;
      throw  new ArgsException();
    } catch (ArgsException e)  { 
      valid  =  false; 
      errorArgumentId  = argChar;
      errorParameter  =  parameter;
      errorCode  =  ErrorCode.INVALID_INTEGER;
      throw e;
    }
  }
...
  private  void  setBooleanArg(char argChar)  { 
   try {
     booleanArgs.get(argChar).set("true");
    } catch (ArgsException e) {
    }
  }
...
  public  int  getInt(char arg)  { 
    Args.ArgumentMarshaler am  =  intArgs.get(arg); 
    return am  ==  null ? 0  : (Integer) am.get();
  }
...
  private abstract  class ArgumentMarshaler  {
    public abstract  void  set(String  s) throws ArgsException;
    public abstract  Object  get();
  }
...
  private  class  IntegerArgumentMarshaler extends ArgumentMarshaler  {
   private int intValue = 0;

   public  void  set(String  s) throws ArgsException {
     try {
      intValue = Integer.parseInt(s);
     } catch (NumberFormatException e) {
       throw new ArgsException();
     }
    }

   public  Object  get()  {
     return intValue;
   }
  }
```

测试当然会继续通过。下一步，我要删除算法顶端的3种不同映射。这样，整个系统就变得更通用了。不过，如果只是删除它们则无法达到目的，因为那样会破坏系统。取而代之的是，我为`ArgumentMarshaler`添加一个新的映射，然后再逐个修改那些方法，让那些方法调用这个新映射。

```
public  class Args  {
...
  private Map<Character, ArgumentMarshaler> booleanArgs =
    new HashMap<Character, ArgumentMarshaler>();
  private Map<Character, ArgumentMarshaler> stringArgs =
    new HashMap<Character, ArgumentMarshaler>(); 
  private Map<Character, ArgumentMarshaler> intArgs = 
    new HashMap<Character,ArgumentMarshaler>();
 private Map<Character, ArgumentMarshaler> marshalers =
 new HashMap<Character, ArgumentMarshaler>();
...
 private  void  parseBooleanSchemaElement(char elementId)  { 
 ArgumentMarshaler m = new BooleanArgumentMarshaler();
    booleanArgs.put(elementId, m);
 marshalers.put(elementId, m);
 }

 private  void  parseIntegerSchemaElement(char elementId)  {
 ArgumentMarshaler m = new IntegerArgumentMarshaler(); 
 intArgs.put(elementId, m);
 marshalers.put(elementId, m);
 }

 private  void  parseStringSchemaElement(char elementId)  { 
 ArgumentMarshaler m = new StringArgumentMarshaler(); 
    stringArgs.put(elementId,m);
 marshalers.put(elementId, m);
 }
```

当然，测试还是通过了。接着，我把isBooleanArg

```
private boolean isBooleanArg(char argChar) {
  return booleanArgs.containsKey(argChar);
}
```

修改成这样：

```
private boolean isBooleanArg(char argChar) { 
 ArgumentMarshaler m = marshalers.get(argChar); 
 return m instanceof BooleanArgumentMarshaler;
}
```

测试仍然通过。于是我修改了`isIntArg`和`isStringArg`：

```
private boolean isIntArg(char argChar) { 
 ArgumentMarshaler m = marshalers.get(argChar); 
 return m instanceof IntegerArgumentMarshaler;
}

private boolean isStringArg(char argChar) {
 ArgumentMarshaler m = marshalers.get(argChar); 
 return m instanceof StringArgumentMarshaler;
}
```

测试继续通过。我接着删除对`marshaler.get`的重复调用：

```
private boolean setArgument(char argChar) throws ArgsException {
 ArgumentMarshaler m = marshalers.get(argChar);
 if (isBooleanArg(m)) 
 setBooleanArg(argChar);
 else if (isStringArg(m)) 
 setStringArg(argChar);
 else if (isIntArg(m)) 
    setIntArg(argChar);
 else
 return false;

 return true;
}

private boolean isIntArg(ArgumentMarshaler m) {
 return m instanceof IntegerArgumentMarshaler;
}

private boolean isStringArg(ArgumentMarshaler m) {
 return m instanceof StringArgumentMarshaler;
}

private boolean isBooleanArg(ArgumentMarshaler m) {
 return m instanceof BooleanArgumentMarshaler;
}
```

存在3个`isxxxArg`方法毫无道理，所以我做了内联修改：

```
private boolean setArgument(char argChar) throws ArgsException {
 ArgumentMarshaler  m = marshalers.get(argChar);
 if (m instanceof BooleanArgumentMarshaler)
 setBooleanArg(argChar);
 else if (m instanceof StringArgumentMarshaler)
 setStringArg(argChar);
 else if (m instanceof IntegerArgumentMarshaler)
 setIntArg(argChar);
 else
 return false;

 return true;
}
```

下一步，我开始在`set`函数中使用`marshalers`映射，停止使用另外3个映射。从`boolean`开始：

```
  private boolean setArgument(char argChar) throws ArgsException {
    ArgumentMarshaler m = marshalers.get(argChar);
    if (m instanceof BooleanArgumentMarshaler)
      setBooleanArg(m);
    else if (m instanceof StringArgumentMarshaler)
      setStringArg(argChar);
    else if (m instanceof IntegerArgumentMarshaler)
      setIntArg(argChar);
    else
      return false;

    return true;
  }
...
  private void setBooleanArg(ArgumentMarshaler m) {
    try {
      m.set("true"); // was: booleanArgs.get(argChar).set("true");
    } catch (ArgsException e) {
    }
  }
```

测试通过，于是我如法炮制`String`类型和`Integer`类型参数。这样我就能把有些丑陋的异常管理代码整合到`setArgument`函数中。

```
private boolean setArgument(char argChar) throws ArgsException { 
  ArgumentMarshaler m = marshalers.get(argChar);
  try {
    if (m instanceof BooleanArgumentMarshaler)
      setBooleanArg(m);
    else if (m instanceof StringArgumentMarshaler)
      setStringArg(m);
    else if (m instanceof IntegerArgumentMarshaler)
      setIntArg(m);
    else
      return false;
  } catch (ArgsException e) {
 valid = false; 
 errorArgumentId = argChar;
 throw e;
 }
  return true;
}

private void setIntArg(ArgumentMarshaler m) throws ArgsException {
  currentArgument++;
  String parameter = null;
  try {
    parameter = args[currentArgument];
    m.set(parameter);
  } catch (ArrayIndexOutOfBoundsException e) { 
    errorCode = ErrorCode.MISSING_INTEGER; 
    throw new ArgsException();
  } catch (ArgsException e) {
    errorParameter = parameter;
    errorCode = ErrorCode.INVALID_INTEGER;
    throw e;
  }
}

private void setStringArg(ArgumentMarshaler m) throws ArgsException {
  currentArgument++;
  try {
    m.set(args[currentArgument]);
  } catch (ArrayIndexOutOfBoundsException e) {
    errorCode = ErrorCode.MISSING_STRING;
    throw new ArgsException();
  }
}
```

离彻底删除那3个旧映射的时机越来越近了。首先，我需要修改`getBoolean`函数：

```
public boolean getBoolean(char arg) { 
  Args.ArgumentMarshaler am = booleanArgs.get(arg); 
  return am != null && (Boolean) am.get();
}
```

修改成这样：

```
public boolean getBoolean(char arg) { 
  Args.ArgumentMarshaler am = marshalers.get(arg);
  boolean b = false;
  try {
    b = am != null && (Boolean) am.get();
  } catch (ClassCastException e) {
    b = false;
  }
  return b;
}
```

最后这个修改可能令人吃惊。为什么我会突然决定对付`ClassCastException`？原因是我有一组单元测试，还有用FitNesse编写的一组验收测试。FitNesse测试确认，如果用非布尔值参数调用`getBoolean`，应该返回`false`。可单元测试的结果不是这样。而到此时为止，我一直只调用单元测试。

这次修改把另一个对`boolean`映射的使用抽离了：

```
private void parseBooleanSchemaElement(char elementId) { 
  ArgumentMarshaler m = new BooleanArgumentMarshaler();
 booleanArgs.put(elementId, m);   marshalers.put(elementId, m);
}
```

如此我们就能删除`boolean`映射。

```
public class Args {
...
 private Map<Character, ArgumentMarshaler> booleanArgs =
 new HashMap<Character, ArgumentMarshaler>();   private Map<Character, ArgumentMarshaler> stringArgs =
  new HashMap<Character, ArgumentMarshaler>();
  private Map<Character, ArgumentMarshaler> intArgs =
  new HashMap<Character, ArgumentMarshaler>();
  private Map<Character, ArgumentMarshaler> marshalers =
  new HashMap<Character, ArgumentMarshaler>();
...
```

接下来，我用同样的方法处理`String`类型和`Integer`类型参数，对`boolean`参数做一点清理工作。

```
  private void parseBooleanSchemaElement(char elementId) {
    marshalers.put(elementId, new BooleanArgumentMarshaler());
  }

  private  void parseIntegerSchemaElement(char elementId) {
    marshalers.put(elementId, new IntegerArgumentMarshaler());
  }

  private void parseStringSchemaElement(char elementId) {
    marshalers.put(elementId, new StringArgumentMarshaler());
  }
...
  public String getString(char arg) { 
    Args.ArgumentMarshaler am = marshalers.get(arg); 
    try {
      return am == null ? "" : (String) am.get();
    } catch (ClassCastException e) {
      return "";
    }
  }

  public int getInt(char arg) {
    Args.ArgumentMarshaler am = marshalers.get(arg);
   try {
      return am == null ? 0 : (Integer) am.get();
    } catch (Exception e) {
      return 0;
    }
  }
...
public class Args {
...
 private Map<Character, ArgumentMarshaler> st ringArgs =    new HashMap<Character, ArgumentMarshaler>();  private Map<Character, ArgumentMarshaler> intArgs =   new HashMap<Character, ArgumentMarshaler>();
  private Map<Character, ArgumentMarshaler> marshalers =
  new HashMap<Character, ArgumentMarshaler>();
...
```

接着，由于那些`parse`方法没有太多事可做，我对它们进行了内联修改：

```
private void parseSchemaElement(String element) throws ParseException {
  char elementId = element.charAt(0);
  String elementTail = element.substring(1);
  validateSchemaElementId(elementId);
  if (isBooleanSchemaElement(elementTail)) 
    marshalers.put(elementId, new BooleanArgumentMarshaler()); 
  else if (isStringSchemaElement(elementTail)) 
    marshalers.put(elementId, new StringArgumentMarshaler()); 
  else if (isIntegerSchemaElement(elementTail)) { 
   marshalers.put(elementId, new IntegerArgumentMarshaler());
  } else {
    throw new ParseException(String.format(
    "Argument: %c has invalid format: %s.", elementId, elementTail), 0);
  }
}
```

行了，下面来看看全貌吧。代码清单14-12展示了`Args`类的现状。

代码清单14-12　Args.java（首次重构后）

```
package com.objectmentor.utilities.getopts;

import java.text.ParseException;
import java.util.*;

public class Args { 
  private String schema; 
  private String[] args;
  private boolean valid = true;
  private Set<Character> unexpectedArguments = new TreeSet<Character>();
  private Map<Character, ArgumentMarshaler> marshalers = 
  new HashMap<Character, ArgumentMarshaler>();
  private Set<Character> argsFound = new HashSet<Character>();
  private int currentArgument;
  private char errorArgumentId = '\0';
  private String errorParameter = "TILT";
  private ErrorCode errorCode = ErrorCode.OK;

  private enum ErrorCode {
    OK, MISSING_STRING, MISSING_INTEGER, INVALID_INTEGER, UNEXPECTED_ARGUMENT}

  public Args(String schema, String[] args) throws ParseException {
    this.schema = schema; 
    this.args = args; 
    valid = parse();
  }

  private boolean parse() throws ParseException { 
    if (schema.length() == 0 && args.length == 0) 
      return true;
    parseSchema();
    try {
      parseArguments();
    } catch (ArgsException e) {
    }
    return valid;
  }

  private boolean parseSchema() throws ParseException {
    for (String element : schema.split(",")) {
      if (element.length() > 0) {
        String trimmedElement = element.trim();
        parseSchemaElement(trimmedElement);
      }
    }
    return true;
  }

  private void parseSchemaElement(String element) throws ParseException {
    char elementId = element.charAt(0);
    String elementTail = element.substring(1);
    validateSchemaElementId(elementId);
    if (isBooleanSchemaElement(elementTail)) 
      marshalers.put(elementId, new BooleanArgumentMarshaler()); 
    else if (isStringSchemaElement(elementTail)) 
      marshalers.put(elementId, new StringArgumentMarshaler());

      else if (isIntegerSchemaElement(elementTail)) {
        marshalers.put(elementId, new IntegerArgumentMarshaler());
      } else {
        throw new ParseException(String.format(
        "Argument: %c has invalid format: %s.", elementId, elementTail), 0);
      }
  }

  private void validateSchemaElementId(char elementId) throws ParseException {
    if (!Character.isLetter(elementId)) {
      throw new ParseException(
      "Bad character:" + elementId + "in Args format: " + schema, 0);
    }
  }

  private boolean isStringSchemaElement(String elementTail) {
    return elementTail.equals("*");
  }

  private boolean isBooleanSchemaElement(String elementTail) {
    return elementTail.length() == 0;
  }

  private boolean isIntegerSchemaElement(String elementTail) {
    return elementTail.equals("#");
  }

  private boolean parseArguments() throws ArgsException {
    for (currentArgument=0; currentArgument<args.length; currentArgument++) {
      String arg = args[currentArgument];
      parseArgument(arg);
    }
    return true;
  }

  private void parseArgument(String arg) throws ArgsException {
    if (arg.startsWith("-"))
      parseElements(arg);
  }

  private void parseElements(String arg) throws ArgsException { 
    for (int i = 1; i < arg.length(); i++) 
      parseElement(arg.charAt(i));
  }

  private void parseElement(char argChar) throws ArgsException {
    if (setArgument(argChar)) 
      argsFound.add(argChar); 
    else {
      unexpectedArguments.add(argChar);
      errorCode = ErrorCode.UNEXPECTED_ARGUMENT;
      valid = false;
    }
  }

  private boolean setArgument(char argChar) throws ArgsException { 
    ArgumentMarshaler m = marshalers.get(argChar);
    try {
      if (m instanceof BooleanArgumentMarshaler)
        setBooleanArg(m);
      else if (m instanceof StringArgumentMarshaler)
        setStringArg(m);
      else if (m instanceof IntegerArgumentMarshaler)
        setIntArg(m);
      else
        return false;
    } catch (ArgsException e) { 
      valid = false; 
      errorArgumentId = argChar; 
      throw e;
    }
    return true;
  }

  private void setIntArg(ArgumentMarshaler m) throws ArgsException {
    currentArgument++;
    String parameter = null;
    try {
      parameter = args[currentArgument];
      m.set(parameter);
    } catch (ArrayIndexOutOfBoundsException e) {
      errorCode = ErrorCode.MISSING_INTEGER; 
      throw new ArgsException();
    } catch (ArgsException e) {
      errorParameter = parameter;
      errorCode = ErrorCode.INVALID_INTEGER;
      throw e;
    }
  }

  private void setStringArg(ArgumentMarshaler m) throws ArgsException {
    currentArgument++;
    try {
      m.set(args[currentArgument]);
    } catch (ArrayIndexOutOfBoundsException e) {
      errorCode = ErrorCode.MISSING_STRING;
      throw new ArgsException();
    }
  }

  private void setBooleanArg(ArgumentMarshaler m) {
    try {
      m.set("true");
    } catch (ArgsException e) {
    }
  }

  public int cardinality() {
    return argsFound.size();
  }

  public String usage() {
    if (schema.length() > 0) 
      return "-[" + schema + "]";
    else
      return "";
  }

  public String errorMessage() throws Exception {
    switch (errorCode) {
      case OK:
        throw new Exception("TILT: Should not get here.");
      case UNEXPECTED_ARGUMENT:
        return unexpectedArgumentMessage();
      case MISSING_STRING:
        return String.format("Could not find string parameter for -%c.",
                             errorArgumentId);
      case INVALID_INTEGER:
        return String.format("Argument -%c expects an integer but was '%s'.",
                             errorArgumentId, errorParameter);
      case MISSING_INTEGER:
        return String.format("Could not find integer parameter for -%c.",
                             errorArgumentId);
    }
    return "";
  }

  private String unexpectedArgumentMessage() {
    StringBuffer message = new StringBuffer("Argument(s) -");
    for (char c : unexpectedArguments) {
      message.append(c);
    }
    message.append(" unexpected.");

    return message.toString();
  }

  public boolean getBoolean(char arg) {
    Args.ArgumentMarshaler am = marshalers.get(arg); 
    boolean b = false;
    try {
      b = am != null && (Boolean) am.get();
    } catch (ClassCastException e) {
      b = false;
    }
    return b;
  }

  public String getString(char arg) { 
    Args.ArgumentMarshaler am = marshalers.get(arg);
    try {
      return am == null ? "" : (String) am.get();
    } catch (ClassCastException e) {
      return "";
    }
  }

  public int getInt(char arg) {
    Args.ArgumentMarshaler am = marshalers.get(arg);
    try {
      return am == null ? 0 : (Integer) am.get();
    } catch (Exception e) {
      return 0;
    }
  }

  public boolean has(char arg) {
    return argsFound.contains(arg);
  }

  public boolean isValid() {
    return valid;
  }

  private class ArgsException extends Exception {
  }

  private abstract class ArgumentMarshaler {
    public abstract void set(String s) throws ArgsException;
    public abstract Object get();
  }

  private class BooleanArgumentMarshaler extends ArgumentMarshaler {
    private boolean booleanValue = false;

    public void set(String s) {
      booleanValue = true;
    }

    public Object get() {
      return booleanValue;
    }
  }

  private class StringArgumentMarshaler extends ArgumentMarshaler {
    private String stringValue = "";

    public void set(String s) {
      stringValue = s;
    }

    public Object get() {
      return stringValue;
    }
  }

  private class IntegerArgumentMarshaler extends ArgumentMarshaler {
    private int intValue = 0;

    public void set(String s) throws ArgsException {
      try {
        intValue = Integer.parseInt(s);
      } catch (NumberFormatException e) {
        throw new ArgsException();
      }
    }

    public Object get() {
      return intValue;
    }
  }
}
```

功夫费尽，还是有点失望。程序结构好了一点，但在代码顶端还是有那一堆变量；在`setArgument`里面还是有那么恐怖的类型转换操作；而且那些`set`函数真的很丑陋。就别提那些错误处理操作了。后面要做的事还很多。

我真是想删掉`setArgument`里面那些类型转换操作[G23]。我希望`setArgument`只简单地调用`ArgumentMarshaler.set`。这意味着我需要将`setIntArg`、`setStringArg`和`setBooleanArg`推到合适的`ArgumentMarshaler`派生类里面。不过这里有一个问题。

仔细看`setIntArg`，你会发现，它使用了两个实体变量`args`和`currentArgument`。为了把`setIntArg`移到`IntegerArgumentMarshaler`里面，我得把这两个变量都作为函数参数传递过去，这种做法太烂了[F1]，因为我只想传递一个参数。幸运的是，有一个简单的解决方法，可以把`args`数组转换为一个`list`，并向`set`函数传递一个`Iterator`。这需要经过10步，每步都通过了测试。不过我只向你展示结果，你应该能看出每个小修改步骤。

```
public class Args {
  private String schema;
 private String[] args;   private boolean valid = true;
  private Set<Character> unexpectedArguments = new TreeSet<Character>();
  private Map<Character, ArgumentMarshaler> marshalers =
   new HashMap<Character, ArgumentMarshaler>();
  private Set<Character> argsFound = new HashSet<Character>();
  private Iterator<String> currentArgument; 
  private char errorArgumentId = '\0';
  private String errorParameter = "TILT";
  private ErrorCode errorCode = ErrorCode.OK;
  private List<String> argsList;

  private enum ErrorCode {
    OK, MISSING_STRING, MISSING_INTEGER, INVALID_INTEGER, UNEXPECTED_ARGUMENT}

  public Args(String schema, String[] args) throws ParseException {
    this.schema = schema;
    argsList = Arrays.asList(args);
    valid = parse();
  }

  private boolean parse() throws ParseException {
    if (schema.length() == 0 && argsList.size() == 0)
      return true;
    parseSchema(); 
    try {
      parseArguments();
    } catch (ArgsException e) {
    }
    return valid;
  }
---
  private boolean parseArguments() throws ArgsException {
    for (currentArgument = argsList.iterator();currentArgument.hasNext();) {
      String arg = currentArgument.next();
      parseArgument(arg);
    }

    retun true;
  }
---
  private void setIntArg(ArgumentMarshaler m) throws ArgsException {
    String parameter = null;
    try {
      parameter = currentArgument.next();
      m.set(parameter);
    } catch (NoSuchElementException e) {
      errorCode = ErrorCode.MISSING_INTEGER;
      throw new ArgsException();
    } catch (ArgsException e) {
      errorParameter = parameter;
      errorCode = ErrorCode.INVALID_INTEGER;
      throw e;
    }
  }

  private void setStringArg(ArgumentMarshaler m) throws ArgsException {
    try {
      m.set(currentArgument.next());
    } catch (NoSuchElementException e) {
      errorCode = ErrorCode.MISSING_STRING;
      throw new ArgsException();
    }
  }
```

是这些简单的修改让测试保持通过。现在我们可以开始把`set`函数移植到合适的派生类中了。第一步，我要在`setArgument`中做以下修改：

```
private boolean setArgument(char argChar) throws ArgsException {
  ArgumentMarshaler m = marshalers.get(argChar);
  if (m == null)
    return false;
  try {
    if (m instanceof BooleanArgumentMarshaler)
      setBooleanArg(m);
    else if (m instanceof StringArgumentMarshaler)
      setStringArg(m);
    else if (m instanceof IntegerArgumentMarshaler)
      setIntArg(m);
 else  return false;   } catch (ArgsException e) {
    valid = false;
    errorArgumentId = argChar; 
    throw e;
  }
  return true;
}
```

这个修改很重要，因为我们想要彻底删除那条`if-else`链。所以，需要把错误条件抽离。

现在可以开始移动`set`函数了。`setBooleanArg`函数很小，就从它开始，目标是让`setBooleanArg`函数只与`BooleanArgumentMarshaler`有关。

```
private boolean setArgument(char argChar) throws ArgsException  { 
    ArgumentMarshaler m = marshalers.get(argChar);
    if (m == null) 
      return false; 
    try {
      if (m instanceof BooleanArgumentMarshaler)
        setBooleanArg(m, currentArgument);
      else if (m instanceof StringArgumentMarshaler)
        setStringArg(m);
      else if (m instanceof IntegerArgumentMarshaler)
        setIntArg(m);

    } catch (ArgsException e) {
      valid = false; 
      errorArgumentId = argChar; 
      throw e;
    }
    return true;
  }
---
  private void setBooleanArg(ArgumentMarshaler m, 
                             Iterator<String> currentArgument)
                             throws ArgsException  {
 try {      m.set("true");
 catch (ArgsException e) {  } 
}
```

我们不是刚把那个异常处理放进去吗？放进和拿出是重构过程中常见的事。小步幅修改和保持测试通过，意味着你会不断移动各种东西。重构有点儿像是解魔方，需要经过许多小步骤，才能达到较大目标。每一步都是下一步的基础。

为什么要在`setBooleanArg`根本不需要的情况下向其传递`iterator`呢？因为`setIntArg`和`setStringArg`需要！还因为我打算通过`ArgumentMarshaler`中的抽象方法部署这3个函数，需要将其传递给`setBooleanArg`。

现在`setBooleanArg`没用了。如果`ArgumentMarshaler`中有一个`set`函数，我们就可以直接调用它。是时候打造这个函数了！第一步，在`ArgumentMarshaler`中添加抽象方法。

```
private abstract class ArgumentMarshaler {
  public abstract void set(Iterator<String> currentArgument) 
 throws ArgsException;
  public abstract void set(String s) throws ArgsException;
  public abstract Object get();
}
```

当然，这会影响到所有派生类。所以，要逐个实现新方法。

```
private class BooleanArgumentMarshaler extends ArgumentMarshaler {
  private boolean booleanValue = false;

  public void set(Iterator<String> currentArgument) throws ArgsException {
 booleanValue = true;
  }

  public void set(String s) {
 booleanValue = true;
  }

  public Object get() {
    return booleanValue;
  }
}

private class StringArgumentMarshaler extends ArgumentMarshaler {
  private String stringValue = "";

  public void set(Iterator<String> currentArgument) throws ArgsException {
  }

  public void set(String s) {
    stringValue = s;
  }

  public Object get() {
    return stringValue;
  }
  }

  private class IntegerArgumentMarshaler extends ArgumentMarshaler {
    private int intValue = 0;

   public void set(Iterator<String> currentArgument) throws ArgsException {
  }

    public void set(String s) throws ArgsException {
      try  
        intValue = Integer.parseInt(s);
      } catch (NumberFormatException e) {
        throw new ArgsException();
      }
    }

    public Object get() {
      return intValue;
    }
  }
```

现在可以删除`setBooleanArg`了！

```
private boolean setArgument(char argChar) throws ArgsException { 
  ArgumentMarshaler m = marshalers.get(argChar);
  if (m == null) 
    return false;
  try {
    if (m instanceof BooleanArgumentMarshaler)
      m.set(currentArgument);
    else if (m instanceof StringArgumentMarshaler)
      setStringArg(m);
    else if (m instanceof IntegerArgumentMarshaler)
      setIntArg(m);

  } catch (ArgsException e) { 
    valid = false; 
    errorArgumentId = argChar; 
    throw e;
  }
  return true;
}
```

测试全都通过，而且`set`函数也部署到`BooleanArgumentMarshaler`里面了！

现在就能对`String`类型和`Integer`类型参数的处理做同样的修改。

```
  private boolean setArgument(char argChar) throws ArgsException {
    ArgumentMarshaler m = marshalers.get(argChar);
    if (m == null) 
      return false;
    try {
      if (m instanceof BooleanArgumentMarshaler)
        m.set(currentArgument);
      else if (m instanceof StringArgumentMarshaler)
        m.set(currentArgument);
      else if (m instanceof IntegerArgumentMarshaler)
        m.set(currentArgument);

    } catch (ArgsException e) { 
      valid = false; 
      errorArgumentId = argChar; 
      throw e;
    }
    return true;
  }
---
  private class StringArgumentMarshaler extends ArgumentMarshaler {
    private String stringValue = "";

    public void set(Iterator currentArgument) throws ArgsException {
      try {
   stringValue = currentArgument.next();
   } catch (NoSuchElementException e) {
   errorCode = ErrorCode.MISSING_STRING;
   throw new ArgsException();
   }
    }

    public void set(String s) {
    }

    public Object get() {
      return stringValue;
    }
  }

  private class IntegerArgumentMarshaler extends ArgumentMarshaler {
    private int intValue = 0;

  public void set(Iterator currentArgument) throws ArgsException {
    String parameter = null;
 try {
 parameter = currentArgument.next();
 set(parameter);
 } catch (NoSuchElementException e) { 
 errorCode = ErrorCode.MISSING_INTEGER; 
 throw new ArgsException();
 } catch (ArgsException e) {
 errorParameter = parameter;
 errorCode = ErrorCode.INVALID_INTEGER;
 throw e;
 }
  }

    public void set(String s) throws ArgsException {
      try {
        intValue = Integer.parseInt(s);
      } catch (NumberFormatException e) {
        throw new  ArgsException();
      }
    }

    public Object get() {
      return intValue;
    }
  }
```

最后一击：可以移除类型转换了！看招！

```
private boolean setArgument(char argChar) throws ArgsException { 
  ArgumentMarshaler m = marshalers.get(argChar);
  if (m == null)
    return false; 
  try {
    m.set(currentArgument);
    return true;
  } catch (ArgsException e) { 
    valid = false; 
    errorArgumentId = argChar; 
    throw e;
  }
}
```

现在可以删除`IntegerArgumentMarshaler`中那些过时的函数，做一下清理了。

```
private class IntegerArgumentMarshaler extends ArgumentMarshaler {
  private int intValue = 0

  public void set(Iterator currentArgument) throws ArgsException {
    String parameter = null;
    try {
      parameter = currentArgument.next();
      intValue = Integer.parseInt(parameter);
    } catch (NoSuchElementException e) { 
      errorCode = ErrorCode.MISSING_INTEGER; 
      throw new ArgsException();
    } catch (NumberFormatException e) {
      errorParameter = parameter;
      errorCode = ErrorCode.INVALID_INTEGER;
      throw new ArgsException();
    }
  }

  public Object get() {
    return  intValue;
  }
}
```

还可以把`ArgumentMarshaler`修改为接口：

```
private interface ArgumentMarshaler {
  void set(Iterator<String> currentArgument) throws ArgsException; 
  Object get();
}
```

现在来看看往这个结构中添加新的参数类型有多容易。只需要做少量修改，而且修改是被隔离的。首先，增加一个新的测试用例，检测`double`参数是否正常工作。

```
public void testSimpleDoublePresent() throws Exception { 
  Args args = new Args("x##", new String[] {"-x","42.3"}); 
  assertTrue(args.isValid());
  assertEquals(1, args.cardinality());
  assertTrue(args.has('x'));
  assertEquals(42.3, args.getDouble('x'), .001);
}
```

然后清理范式解析代码，为`double`参数类型添加##监测。

```
private void parseSchemaElement(String element) throws ParseException {
  char elementId = element.charAt(0);
  String elementTail = element.substring(1);
  validateSchemaElementId(elementId);
  if (elementTail.length() == 0)
    marshalers.put(elementId, new BooleanArgumentMarshaler());
  else if (elementTail.equals("*"))
    marshalers.put(elementId, new  StringArgumentMarshaler());
  else if (elementTail.equals("#"))
    marshalers.put(elementId, new  IntegerArgumentMarshaler());

  else if (elementTail.equals("##"))
 marshalers.put(elementId, new DoubleArgumentMarshaler());
  else
    throw new ParseException(String.format(
      "Argument: %c has invalid format: %s.", elementId, elementTail), 0);
}
```

下一步，编写`DoubleArgumentMarshaler`类：
```
private class DoubleArgumentMarshaler implements ArgumentMarshaler {
 private double doubleValue = 0;

  public void set(Iterator<String> currentArgument) throws ArgsException {
    String parameter = null;
 try {
 parameter = currentArgument.next();
 doubleValue = Double.parseDouble(parameter);
 } catch (NoSuchElementException e) { 
   errorCode = ErrorCode.MISSING_DOUBLE; 
   throw new ArgsException();
 } catch (NumberFormatException e) {
 errorParameter = parameter;
 errorCode = ErrorCode.INVALID_DOUBLE;
 throw new ArgsException();
 }
 }

 public Object get() {
 return doubleValue;
 }
}
```

然后就得添加一个新的`ErrorCode`：

```
private enum ErrorCode {
 OK, MISSING_STRING, MISSING_INTEGER, INVALID_INTEGER, UNEXPECTED_ARGUMENT,
 MISSING_DOUBLE, INVALID_DOUBLE}
```

还需要一个getDouble函数：

```
public double getDouble(char arg) { 
 Args.ArgumentMarshaler am = marshalers.get(arg); 
 try {
 return am == null ? 0 : (Double) am.get();
 } catch (Exception e) {
 return 0.0;
 }
}
```

全部测试都通过了！完全无痛。再来确保全部错误处理代码正确工作。下一个测试用例用来检测在向`##`参数传递一个不可解析的字符串时是否会返回错误。

```
public void testInvalidDouble() throws Exception {
  Args args = new  Args("x##", new String[] {"-x","Forty two"});
  assertFalse(args.isValid()); 
  assertEquals(0, args.cardinality()); 
  assertFalse(args.has('x')); 
  assertEquals(0, args.getInt('x'));
  assertEquals("Argument -x expects a double but was 'Forty two'.", 
               args.errorMessage());

 }
---
 public String errorMessage() throws Exception {
   switch (errorCode) {
     case OK:
       throw new Exception("TILT: Should not get here.");
     case UNEXPECTED_ARGUMENT:
       return unexpectedArgumentMessage();
     case MISSING_STRING:
       return String.format("Could not find string parameter for -%c.",
                            errorArgumentId);
     case INVALID_INTEGER:
       return String.format("Argument -%c expects an integer but was '%s'.",
                             errorArgumentId, errorParameter);
     case MISSING_INTEGER:
       return String.format("Could not find integer parameter for -%c.",
                             errorArgumentId);
    case INVALID_DOUBLE:
 return String.format("Argument -%c expects a double but was '%s'.",
 errorArgumentId, errorParameter);
 case MISSING_DOUBLE:
 return String.format("Could not find double parameter for -%c.",
 errorArgumentId);
   }
   return  "";
 }
```

测试通过。下一个测试确保我们正确检测到遗漏的`double`参数：

```
public void testMissingDouble() throws Exception { 
 Args args = new Args("x##", new String[]{"-x"}); 
 assertFalse(args.isValid());
 assertEquals(0, args.cardinality());
 assertFalse(args.has('x'));
 assertEquals(0.0, args.getDouble('x'), 0.01);
 assertEquals("Could not find double parameter for -x.",
 args.errorMessage());
}
```

测试如愿通过。我们只是为了保持一切完整而编写这个测试。

异常代码很丑陋，不该在`Args`类中存在，我们也抛出`ParseException`，但那并不真的属于我们自己。因此我们把所有异常都塞到`ArgsException`类中，并将其移到它自己的模块里面。

```
public class ArgsException extends Exception {
 private char errorArgumentId = '\0';
 private String errorParameter = "TILT";
 private ErrorCode errorCode = ErrorCode.OK;

 public ArgsException() {}

 public ArgsException(String message) {super(message);}

 public enum ErrorCode {
 OK, MISSING_STRING, MISSING_INTEGER, INVALID_INTEGER, UNEXPECTED_ARGUMENT,
 MISSING_DOUBLE, INVALID_DOUBLE}
}
---

public class Args {
  ...
  private char errorArgumentId = '\0';
  private String errorParameter = "TILT";
  private ArgsException.ErrorCode errorCode = ArgsException.ErrorCode.OK;
  private List<String> argsList;

  public Args(String schema, String[] args) throws ArgsException {
    this.schema = schema;
    argsList = Arrays.asList(args);
    valid = parse();
  }

  private boolean parse() throws ArgsException {
    if (schema.length() == 0 && argsList.size() == 0)
      return true; 
    parseSchema(); 
    try {
      parseArguments();
    } catch (ArgsException e) {
    }
    return valid;
  }

  private boolean parseSchema() throws ArgsException {
    ...
  }

  private void parseSchemaElement(String element) throws ArgsException {
    ... 
    else
      throw new ArgsException(
        String.format("Argument: %c has invalid format: %s.", 
                      elementId,elementTail));
  }

  private void validateSchemaElementId(char elementId) throws ArgsException {
    if (!Character.isLetter(elementId)) {
      throw new ArgsException(
        "Bad character:" + elementId + "in Args format: " + schema);
    }
  }

  ...

  private void parseElement(char argChar) throws ArgsException {
    if (setArgument(argChar)) 
      argsFound.add(argChar); 
    else {
      unexpectedArguments.add(argChar);
      errorCode = ArgsException.ErrorCode.UNEXPECTED_ARGUMENT;
      valid = false;
    }
  }

  ...

  private class StringArgumentMarshaler implements ArgumentMarshaler {
    private String stringValue = "";

    public void set(Iterator<String> currentArgument) throws ArgsException {
      try {
        stringValue = currentArgument.next();
      } catch (NoSuchElementException e) {
        errorCode = ArgsException.ErrorCode.MISSING_STRING;
        throw new ArgsException();
      }
    }

    public Object get() {
      return stringValue;
    }
  }

  private class IntegerArgumentMarshaler implements ArgumentMarshaler {
    private int intValue = 0;

    public void set(Iterator<String> currentArgument) throws ArgsException { 
      String parameter = null;
      try {
        parameter = currentArgument.next();
        intValue = Integer.parseInt(parameter);
      } catch (NoSuchElementException e) {
        errorCode = ArgsException.ErrorCode.MISSING_INTEGER;
        throw new ArgsException();
      } catch (NumberFormatException e) {
        errorParameter = parameter;
        errorCode = ArgsException.ErrorCode.INVALID_INTEGER;
        throw new ArgsException();
      }
    }

    public Object get() {
      return intValue;
    }
  }

  private class DoubleArgumentMarshaler implements ArgumentMarshaler {
    private double doubleValue = 0;

    public void set(Iterator<String> currentArgument) throws ArgsException { 
      String parameter = null;
      try {
        parameter = currentArgument.next();
        doubleValue = Double.parseDouble(parameter);
      } catch (NoSuchElementException e) {
        errorCode = ArgsException.ErrorCode.MISSING_DOUBLE;
        throw new ArgsException();
      } catch (NumberFormatException e) {
        errorParameter = parameter;
        errorCode = ArgsException.ErrorCode.INVALID_DOUBLE;
        throw new ArgsException();
      }
    }

    public Object get() {
      return doubleValue;
    }
  }
}
```

很好。现在，`Args`抛出的唯一一个异常是`ArgsException`。把`ArgsException`移到它自己的模块中，这意味着我们能把大量杂七杂八的错误支持代码从`Args`模块转移到这个模块。

现在我们完全把异常和错误代码从`Args`模块中隔离出来了。（如代码清单14-13～代码清单14-16所示。）为达到这一目标，大概做了30次小修改，每次修改都保持测试通过。

代码清单14-13　ArgsTest.java

```
package com.objectmentor.utilities.args;

import junit.framework.TestCase;

public class ArgsTest extends TestCase {
  public void testCreateWithNoSchemaOrArguments() throws Exception { 
    Args args = new Args("", new String[0]);
    assertEquals(0, args.cardinality());
  }

  public void testWithNoSchemaButWithOneArgument() throws Exception {
    try {
      new Args("", new String[]{"-x"});
      fail();
    } catch (ArgsException e) {
      assertEquals(ArgsException.ErrorCode.UNEXPECTED_ARGUMENT,
                   e.getErrorCode());
      assertEquals('x', e.getErrorArgumentId());
    }
  }

  public void testWithNoSchemaButWithMultipleArguments() throws Exception {
    try {
      new Args("", new String[]{"-x", "-y"});
      fail();
    } catch (ArgsException e) {
      assertEquals(ArgsException.ErrorCode.UNEXPECTED_ARGUMENT, 
                   e.getErrorCode());
      assertEquals('x', e.getErrorArgumentId());
    }
　
  }

  public void testNonLetterSchema() throws Exception {
    try {
      new Args("*", new String[]{});
      fail("Args  constructor  should have thrown exception");
    } catch (ArgsException e) {
      assertEquals(ArgsException.ErrorCode.INVALID_ARGUMENT_NAME,
                   e.getErrorCode());
      assertEquals('*', e.getErrorArgumentId());
    }
  }

  public void testInvalidArgumentFormat() throws Exception {
    try {
      new Args("f~", new  String[]{});
      fail("Args constructor should have throws exception");
    } catch (ArgsException e) { 
      assertEquals(ArgsException.ErrorCode.INVALID_FORMAT, e.getErrorCode()); 
      assertEquals('f', e.getErrorArgumentId());
    }
  }

  public void testSimpleBooleanPresent() throws Exception { 
    Args args = new  Args("x", new String[]{"-x"}); 
    assertEquals(1, args.cardinality());
    assertEquals(true, args.getBoolean('x'));
  }

  public void testSimpleStringPresent() throws Exception { 
    Args args = new Args("x*", new String[]{"-x", "param"}); 
    assertEquals(1, args.cardinality()); 
    assertTrue(args.has('x'));
    assertEquals("param", args.getString('x'));
  }

  public void testMissingStringArgument() throws Exception {
    try {
      new Args("x*", new String[]{"-x"});
      fail();
    } catch (ArgsException e) { 
      assertEquals(ArgsException.ErrorCode.MISSING_STRING, e.getErrorCode()); 
      assertEquals('x', e.getErrorArgumentId());
    }
  }

  public void testSpacesInFormat() throws Exception { 
    Args args = new  Args("x, y", new String[]{"-xy"}); 
    assertEquals(2, args.cardinality()); 
    assertTrue(args.has('x')); 
    assertTrue(args.has('y'));
  }

  public void testSimpleIntPresent() throws Exception { 
    Args args = new Args("x#", new String[]{"-x", "42"}); 
    assertEquals(1, args.cardinality());
    assertTrue(args.has('x'));
    assertEquals(42, args.getInt('x'));
  }

  public void testInvalidInteger() throws Exception {
    try {
      new Args("x#", new String[]{"-x","Forty two"});
      fail();
    } catch (ArgsException e) { 
      assertEquals(ArgsException.ErrorCode.INVALID_INTEGER, e.getErrorCode()); 
      assertEquals('x', e.getErrorArgumentId());
      assertEquals("Forty two", e.getErrorParameter());
    }
  }

  public void testMissingInteger() throws Exception {
    try {
      new Args("x#", new String[]{"-x"});
      fail();
    } catch (ArgsException e) { 
      assertEquals(ArgsException.ErrorCode.MISSING_INTEGER, e.getErrorCode()); 
      assertEquals('x', e.getErrorArgumentId());
    }
  }

  public void testSimpleDoublePresent() throws Exception { 
    Args args = new Args("x##", new String[]{"-x", "42.3"}); 
    assertEquals(1, args.cardinality());
    assertTrue(args.has('x'));
    assertEquals(42.3, args.getDouble('x'), .001);
  }

  public void testInvalidDouble() throws Exception {
    try {
      new Args("x##", new  String[]{"-x", "Forty two"});
      fail();
    } catch (ArgsException e) {
      assertEquals(ArgsException.ErrorCode.INVALID_DOUBLE, e.getErrorCode()); 
      assertEquals('x', e.getErrorArgumentId());
      assertEquals("Forty two", e.getErrorParameter());
    }
  }

  public void testMissingDouble() throws Exception {
    try {
      new Args("x##", new String[]{"-x"});
      fail();
    } catch (ArgsException e) { 
      assertEquals(ArgsException.ErrorCode.MISSING_DOUBLE, e.getErrorCode()); 
      assertEquals('x', e.getErrorArgumentId());
    }
  }
}
```

代码清单14-14　ArgsExceptionTest.java

```
public class ArgsExceptionTest extends TestCase {
  public void testUnexpectedMessage() throws Exception { 
    ArgsException e = 
    new ArgsException(ArgsException.ErrorCode.UNEXPECTED_ARGUMENT,
                      'x', null);
    assertEquals("Argument -x unexpected.", e.errorMessage());
  }

  public void testMissingStringMessage() throws Exception {
    ArgsException e = new ArgsException(ArgsException.ErrorCode.MISSING_STRING,
                                        'x', null);
    assertEquals("Could not find string parameter for -x.", e.errorMessage());
  }

  public void testInvalidIntegerMessage() throws Exception { 
    ArgsException e =
      new ArgsException(ArgsException.ErrorCode.INVALID_INTEGER, 'x', "Forty two");
    assertEquals("Argument -x expects an integer but was 'Forty two'.",
                 e.errorMessage());
  }

  public  void testMissingIntegerMessage() throws Exception { 
    ArgsException e = 
      new ArgsException(ArgsException.ErrorCode.MISSING_INTEGER, 'x', null);
    assertEquals("Could not find integer parameter  for -x.", e.errorMessage());
  }

  public void testInvalidDoubleMessage() throws Exception {
    ArgsException e = new ArgsException(ArgsException.ErrorCode.INVALID_DOUBLE,
                                        'x', "Forty two");
    assertEquals("Argument -x expects a double but was 'Forty two'.",
                 e.errorMessage());
  }

  public void testMissingDoubleMessage() throws Exception {
    ArgsException e = new ArgsException(ArgsException.ErrorCode.MISSING_DOUBLE,
                                        'x', null);
    assertEquals("Could not find double parameter  for  -x.", e.errorMessage());
  }
}
```

代码清单14-15　ArgsException.java

```
public class ArgsException extends  Exception {
  private char errorArgumentId = '\0';
  private String errorParameter = "TILT";
  private ErrorCode errorCode = ErrorCode.OK;

  public ArgsException() {}

  public ArgsException(String  message) {super(message);}

  public ArgsException(ErrorCode errorCode) {
    this.errorCode = errorCode;
  }

  public ArgsException(ErrorCode errorCode, String errorParameter) {
    this.errorCode = errorCode;
    this.errorParameter = errorParameter;
  }

  public ArgsException(ErrorCode errorCode, char errorArgumentId,
                       String errorParameter) {
    this.errorCode = errorCode; 
    this.errorParameter = errorParameter;
    this.errorArgumentId = errorArgumentId;
  }

  public char getErrorArgumentId() {
    return errorArgumentId;
  }

  public void setErrorArgumentId(char errorArgumentId) {
    this.errorArgumentId = errorArgumentId;
  }

  public String getErrorParameter() {
    return errorParameter;
  }

  public void setErrorParameter(String errorParameter) {
    this.errorParameter = errorParameter;
  }

  public ErrorCode getErrorCode() {
    return errorCode;
  }

  public void setErrorCode(ErrorCode errorCode) {
    this.errorCode = errorCode;
  }

  public String errorMessage() throws Exception {
    switch (errorCode) {
      case OK:
        throw new Exception("TILT: Should not get here.");
      case  UNEXPECTED_ARGUMENT:
        return String.format("Argument -%c unexpected.", errorArgumentId);
      case MISSING_STRING:
        return String.format("Could not find string parameter for -%c.",
                              errorArgumentId);
      case INVALID_INTEGER:
        return  String.format("Argument -%c expects an integer but was '%s'.",
                              errorArgumentId, errorParameter);
      case MISSING_INTEGER:
        return  String.format("Could not find integer parameter for -%c.",
                              errorArgumentId);
      case INVALID_DOUBLE:
        return  String.format("Argument -%c expects a double but was '%s'.",
                              errorArgumentId, errorParameter);

      case MISSING_DOUBLE:
        return  String.format("Could not find double parameter for -%c.",
                              errorArgumentId);
    }
    return  "";
  }

  public enum  ErrorCode {
    OK, INVALID_FORMAT, UNEXPECTED_ARGUMENT, INVALID_ARGUMENT_NAME, 
    MISSING_STRING,
    MISSING_INTEGER, INVALID_INTEGER, 
    MISSING_DOUBLE, INVALID_DOUBLE}
}
```

代码清单14-16　Args.java

```
public class Args {
  private String schema;
  private Map<Character, ArgumentMarshaler> marshalers =
    new HashMap<Character, ArgumentMarshaler>();
  private Set<Character> argsFound = new HashSet<Character>();
  private Iterator<String> currentArgument;
  private List<String> argsList;

  public Args(String schema, String[] args) throws ArgsException {
    this.schema = schema;
    argsList = Arrays.asList(args);
    parse();
  }

  private void parse() throws ArgsException {
    parseSchema();
    parseArguments();
  }

  private boolean parseSchema() throws ArgsException {
    for (String element : schema.split(",")) { 
      if (element.length() > 0) { 
        parseSchemaElement(element.trim());
      }
    }
    return true;
  }

  private void parseSchemaElement(String element) throws ArgsException {
    char elementId = element.charAt(0);
    String elementTail = element.substring(1);
    validateSchemaElementId(elementId);
    if (elementTail.length() == 0)
      marshalers.put(elementId, new BooleanArgumentMarshaler());
    else if (elementTail.equals("*"))
      marshalers.put(elementId, new  StringArgumentMarshaler());
    else if (elementTail.equals("#"))
      marshalers.put(elementId, new IntegerArgumentMarshaler());
    else if (elementTail.equals("##"))
      marshalers.put(elementId, new  DoubleArgumentMarshaler());
    else
      throw new ArgsException(ArgsException.ErrorCode.INVALID_FORMAT,
                              elementId, elementTail);
  }

  private void validateSchemaElementId(char elementId) throws ArgsException {
    if (!Character.isLetter(elementId)) {
      throw  new ArgsException(ArgsException.ErrorCode.INVALID_ARGUMENT_NAME,
                               elementId, null);
    }
  }

  private void parseArguments() throws ArgsException {
    for (currentArgument = argsList.iterator();  currentArgument.hasNext();) { 
      String arg = currentArgument.next();
      parseArgument(arg);
    }
  }

  private void parseArgument(String arg) throws ArgsException {
    if (arg.startsWith("-"))
      parseElements(arg);
  }

  private void parseElements(String arg) throws ArgsException { 
    for (int i = 1; i < arg.length(); i++) 
      parseElement(arg.charAt(i));
  }

  private void parseElement(char argChar) throws ArgsException {
    if (setArgument(argChar)) 
      argsFound.add(argChar); 
    else {
      throw new ArgsException(ArgsException.ErrorCode.UNEXPECTED_ARGUMENT,
                              argChar, null);
    }
  }

  private boolean setArgument(char argChar) throws ArgsException { 
    ArgumentMarshaler m = marshalers.get(argChar);
    if (m == null)
      return false; 
    try {
      m.set(currentArgument);
      return true;
    } catch (ArgsException e) { 
      e.setErrorArgumentId(argChar); 
      throw e;
    }
  }

  public int cardinality() {
    return argsFound.size();
  }

  public String usage() {
    if (schema.length() > 0) 
      return "-[" + schema + "]"; 
    else
      return "";
  }

  public boolean getBoolean(char arg) { 
    ArgumentMarshaler am = marshalers.get(arg); 
    boolean b = false;
    try {
      b = am != null && (Boolean) am.get();
    } catch (ClassCastException e) {
      b = false;
    }
    return b;
  }

  public String getString(char arg) { 
    ArgumentMarshaler am = marshalers.get(arg); 
    try {
      return am == null ? "" : (String) am.get();
    } catch (ClassCastException e) {
      return "";
    }
  }

  public int getInt(char arg) {
    ArgumentMarshaler am = marshalers.get(arg);
    try {
      return am == null ? 0 : (Integer) am.get();
    } catch (Exception e) {
      return 0;
    }
  }

  public double getDouble(char arg) { 
    ArgumentMarshaler am = marshalers.get(arg); 
    try {
      return am == null ? 0 : (Double) am.get();
    } catch (Exception e) {
      return  0.0;
    }
  }

  public boolean has(char arg) {
    return argsFound.contains(arg);
  }
}
```

对`Args`类所做的最主要的修改是在监测部分。从`Args`里面取出了大量代码，放到`ArgsException`中。很好。我们还把全部`ArgumentMarshaler`转移到了它们自己的文件中。更好！

优秀的软件设计，大都关乎分隔——创建合适的空间放置不同种类的代码。对关注面的分隔让代码更易于理解和维护。

特别有意思的是`ArgsException`中的`errorMessage`方法。显然，把错误信息格式化操作放在`Args`里面，违反了SRP。`Args`应该只处理参数，而不该去管错误信息的格式。然而，把错误信息格式化代码放到`ArgsException`中是否有道理呢？

实话说，这是一种折中的做法。不打算用`ArgsException`提供的错误信息的用户会想自己写错误信息。但如果有备好的错误信息，其方便之处也并非鲜见。

显然，现在我们完成的工作已经非常接近本章开始处所展示的最终解决方案了。最后的工作留给你来练习完成。

## 14.4　小结

代码能工作还不够。能工作的代码经常会严重崩溃。满足于仅仅让代码能工作的程序员不够专业，他们害怕没时间改进代码的结构和设计，我不敢苟同。没什么能比糟糕的代码给开发项目带来更深远和长期的损害了。进度可以调整，需求可以重新定义，团队可以动态修正，但糟糕的代码只会一直腐败发酵，无情地拖着团队的后腿。我无数次看到开发团队蹒跚前行，就是因为他们匆匆搞出一片代码沼泽，从此之后命运再也不受自己控制。

当然，糟糕的代码可以清理，不过成本高昂。随着代码腐败下去，模块之间互相渗透，出现大量隐藏纠缠的依赖关系，找到和破除陈旧的依赖关系又费时间又费劲。然而，保持代码整洁却相对容易。早晨在模块中制造出一堆混乱，下午就能轻易清理掉。更好的情况是，5分钟之前制造出混乱，马上就能很容易地清理掉。

所以，解决之道就是保持代码持续整洁和简单，永不让“腐坏”有机会开始。
