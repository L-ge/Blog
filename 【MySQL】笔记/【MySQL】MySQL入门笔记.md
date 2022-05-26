## 前言：
**《MySQL必知必会》**

# 一、MySQL安装
Ubuntu 16.04 安装 MySQL
```
sudo apt-get update
sudo apt-get upgrade
sudo apt install mysql-server
sudo apt install mysql-client
sudo apt install libmysqlclient-dev
```
验证是否成功，若 MySQL 节点处于 LISTEN 状态表示成功 
```
sudo netstat -tap | grep mysql
```

(看需要开启)设置mysql允许远程访问的方法：  
注释掉 bind-address=127.0.0.1 那行：  
sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf

mysql -uroot -p
然后提示输入你的密码，例如mysql -uroot -proot
```
新版有可能在安装时没有提示你输入用户名和密码，则就没有root这个用户，则需要
sudo cat /etc/mysql/debian.cnf
mysql -u debian-sys-maint -p        // debian-sys-maint 这个名字和密码是 debian.cnf 里面的
-> use mysql;
-> update mysql.user set authentication_string=password('root') where user='root' and Host='localhost';
-> update user set plugin='mysql_native_password';
-> flush privileges;
-> quit;
sudo service mysql restart      // 重启mysql
mysql -u root -p                // 上面设置用户名和密码都为 root
```

```
执行SQL脚本语句：  
create database student;  
use student;  
source xxx.sql  
```

# 二、MySQL工具
1、mysql命令行实用程序
- 命令用;或\g结束，换句话说，仅按Enter不执行命令；
- 输入help或\h获得帮助，也可以输入更多的文本获得特定命令的帮助；（例如，输入help select获得使用SELECT语句的帮助）；
- 输入quit或exit退出命令行实用程序。

# 三、了解数据库和表
1、执行SQL脚本语句
```
mysql> create database student;    // 创建数据库student
mysql> use student;                // 选择数据库student
mysql> source xxx.sql              // 执行当前目录下xxx.sql脚本
```

2、show命令
```
mysql> show databases;              // 返回可用数据库的一个列表
mysql> show tables;                 // 获得当前选择的数据库内可用表的列表
mysql> show columns from xxxxx;     // 显示 xxxxx 表的所有列
mysql> describe xxxxx;              // 显示 xxxxx 表的所有列(是show columns的一种快捷方式)
mysql> show status;                 // 显示广泛的服务器状态信息
mysql> show create database xxx;    // 显示创建特定数据库xxx的MySQL语句
mysql> show create table xxx;       // 显示创建特定表xxx的MySQL语句
mysql> show grants;                 // 显示授予用户(所有用户或特定用户)的安全权限
mysql> show errors;                 // 显示服务器错误信息
mysql> show warnings;               // 显示服务器警告信息
mysql> help show;                   // 显示show命令的帮助信息
```

# 四、检索数据
1、select语句
```
mysql> select prod_name from products;                      // 从products表中检索出一个名为prod_name的列(返回的结果是可能并不是插入表的顺序)
mysql> select prod_id, prod_name, prod_price from products; // 从products表中检索出一个名为prod_id, prod_name, prod_price的列，列名之间用逗号分隔
mysql> select * from products;                              // 通过通配符（*）返回表中的所有列
mysql> select distinct vend_id from products;               // 关键字distinct告诉MySQL只返回不同的值，distinct关键字必须直接放在列名的前面
mysql> select prod_name from products limit 5;              // limit 5指示MySQL返回不多于5行
mysql> select prod_name from products limit 3,4;            // limit 3,4 指示MySQL返回从行3开始的4行。第一个数为开始的位置，第二个数为要检索的行数。
mysql> select prod_name from products limit 4 offset 3;     // limit 4 offset 3 指示MySQL返回从行3开始的4行。第一个数为开始的位置，第二个数为要检索的行数。
mysql> select products.prod_name from products;             // 从products表中检索出一个名为prod_name的列。使用完全限定的名字来引用列（同时使用表名和列字）
```
- 不能部分使用 distinct：distinct关键字应用于所有列而不仅是前置它的列。
- 带一个值的limit总是从第一行开始，给出的数为返回的行数。带两个值的limit可以指定从行号为第一个值的位置开始。
- 行0检索出来的第一行为行0而不是行1。因此，limit 1,1将检索出第二行而不是第一行。
- 列名和表名都可以是完全限定的。

# 五、排序数据
1、order by

为了明确地排序用select语句检索出的数据，可使用 order by 子句。
order by 子句取一个或多个列的名字，据此对输出进行排序。
```
mysql> select prod_name from products order by prod_name;  // 指示MySQL对prod_name列以字母顺序排序数据。
mysql> select prod_id, prod_price, prod_name from products order by prod_price, prod_name;   // 按其中两个列对结果进行排序。仅在多个行具有相同的prod_price值时才对产品按prod_name进行排序。如果prod_price列中所有的值都是唯一的，则不会按prod_name排序。

mysql> select prod_id, prod_price, prod_name from products order by prod_price desc;   //  按prod_price以降序排序产品
mysql> select prod_id, prod_price, prod_name from products order by prod_price desc, prod_name;   //  按prod_price以降序排序产品，然后再按prod_name排序
mysql> select prod_price from products order by prod_price desc limit 1;    // 找出价格最高的物品
```
- 通常，order by子句中使用的列将是为显示所选择的列。但是，用非检索的列排序数据也是完全合法的。
- 数据排序默认是升序排序，也可以用降序进行排序。为了进行降序排序，必须指定 desc 关键字。升序是关键字asc，但是asc没有多大用处，因为默认就是升序。
- desc关键字只应用到直接位于其前面的列名。所以，如果想在多个列上进行降序排序，必须对每个列指定desc关键字。
- 字母排序是按字典序排的，因此A和a是相同的，所以order by子句是无法区分大小写字母的。
- 使用order by和limit的组合，可以找出一个列中最高或最低的值。
- 在给出order by子句时，应该保证它位于from子句之后。如果使用limit，它必须位于order by之后。使用子句的次序不对将产生错误信息。
- order by子句必须是select语句中的最后一条语句。

# 六、过滤数据

1、where
```
mysql> select prod_name, prod_price from products where prod_price=2.50;    // 只返回prod_price值为2.50的行

mysql> select prod_name, prod_price from products where prod_name='fuses';  // 返回prod_name的值为Fuses的一行（没有区分大小写）。

mysql> select vend_id, prod_name from products where vend_id <> 1003;   // 列出vend_id不是1003的所有产品

mysql> select vend_id, prod_name from products where vend_id != 1003;   // 列出vend_id不是1003的所有产品

mysql> select prod_name, prod_price from products where prod_price between 5 and 10;    // 返回prod_price在5和10之间的产品。注意是闭区间[5,10]

mysql> select cust_id from customers where cust_email is null;
```
- where子句在表名（from子句）之后给出。
- 在同时使用order by和where子句时，应该让order by位于where之后，否则将会产生错误。
- MySQL在执行匹配时默认不区分大小写。
- 单引号用来限定字符串。如果将值与串类型的列进行比较，则需要限定引号。用来与数值列进行比较的值不用引号。
- between匹配范围中所有的值，包括指定的开始值和结束值。
- 在创建表时，表设计人员可以指定其中的列是否可以不包含值。在一个列不包含值时，称其为包含空值NULL。
- NULL：无值，它与字段包含0、空字符串或仅仅包含空格不同。
- select语句有一个特殊的where子句，可用来检查具有NULL值的列。这个where子句就是is null子句。
- 在通过过滤条件选择出不具有特定值的行时，返回不了具有NULL值的行。因为未知具有特殊的含义，数据库不知道它们是否匹配，所以在匹配过滤或不匹配过滤时不返回它们。因此，在过滤数据时，一定要验证返回数据中确实给出了被过滤列具有NULL的行。

2、MySQL支持的所有条件操作符

操作符 | 说明
---|---
= | 等于
<> | 不等于
!= | 不等于
< | 小于
<= | 小于等于
> | 大于
>= | 大于等于
BETWEEN | 在指定的两个值之间

# 七、数据过滤

1、组合where子句
```
mysql> select prod_id, prod_price, prod_name from products where vend_id=1003 and prod_price<=10;
    
mysql> select prod_name, prod_price from products where vend_id=1002 or vend_id=1003;
```

- 操作符：用来联结或改变where子句中的子句的关键字。也称为逻辑操作符。
- AND：用在where子句中的关键字，用来指示检索满足所有给定条件的行。
- OR：where子句中使用的关键字，用来表示检索匹配任一给定条件的行。
- AND在计算次序中优先级更高，and比or优先级高。但是圆括号具有较and或or操作符高的计算次序。
- 任何时候使用具有and和or操作符的where子句，都应该使用圆括号明确地分组操作符，不要过分依赖默认计算次序。

2、in操作符
```
mysql> select prod_name, prod_price from products
    -> where vend_id in (1002,1003) order by prod_name;
    
mysql> select prod_name, prod_price from products
    -> where vend_id=1002 or vend_id=1003 order by prod_name;   // 功能同上 
```

- in操作符用来指定条件范围，范围中的每个条件都可以进行匹配。
- in取合法值的由逗号分隔的清单，全都括在圆括号中。
- in：where子句中用来指定要匹配值的清单的关键字，功能与or相当。

为什么要使用IN操作符？其优点具体如下：
- 在使用长的合法选项清单时，in操作符的语法更清楚且更直观；
- 在使用in时，计算的次序更容易管理（因为使用的操作符更少）；
- in操作符一般比or操作符清单执行更快；
- in的最大优点是可以包含其他select语句，使得能够更动态地建立where子句。

3、not操作符

where子句中的not操作符有且只有一个功能，那就是否定它之后所跟的任何条件。

```
mysql> select prod_name, prod_price from products
    -> where vend_id not in (1002,1003) order by prod_name;
```
- NOT：where子句中用来否定后跟条件的关键字。
- MySQL中的not：MySQL支持使用not对in、between和exists子句取反，这与多数其他DBMS允许使用not对各种条件取反有很大的差别。


# 八、用通配符进行过滤

1、like操作符
```
mysql> select prod_id, prod_name from products where prod_name like 'jet%'; // 检索任意以jet起头的词。%告诉MySQL接受jet之后的任何字符，不管它有多少字符。

mysql> select prod_id, prod_name from products where prod_name like '%anvil%';  // 匹配任何位置包含文本anvil的值，而不论它之前或之后出现什么字符。

mysql> select prod_id, prod_name from products where prod_name like '_ ton anvil';
```

- 通配符：用来匹配值的一部分的特殊字符。
- 搜索模式：由字面值、通配符或两者组合构成的搜索条件。
- 为在搜索子句中使用通配符，必须使用LIKE操作符。LIKE指示MySQL，后跟的搜索模式利用通配符而不是直接相等匹配进行比较。
- 从技术上讲，LIKE是谓词而不是操作符。
- 在搜索串中，%表示任何字符出现任意次数（可以是0次）。
- 根据MySQL的配置方式，搜索可以是区分大小写的。
- 通配符可在搜索模式中任意位置使用，并且可以使用多个通配符。
- 尾空格可能会干扰通配符匹配。例如，在保存词anvil时，如果它后面有一个或者多个空格，则子句where prod_name like '%anvil'将不会匹配它们，因为在最后的l后有多余的字符。解决这个问题的一个简单的方法是在搜索模式最后附加一个%。一个更好的办法是使用函数去掉首尾空格。
- 虽然似乎%通配符可以匹配任何东西，但有一个例外，即NULL。即使是where prod_name like '%'也不能匹配用值NULL作为产品名的行。
- 通配符下划线只匹配一个字符而不是多个字符。

使用通配符的技巧：
- 不要过度使用通配符。如果其他操作符能达到相同的目的，应该使用其他操作符。
- 在确实需要使用通配符时，除非绝对有必要，否则不要把它们用在搜索模式的开始处。把通配符置于搜索模式的开始处，搜索起来是最慢的。
- 仔细注意通配符的位置。如果放错地方，可能不会返回想要的数据。


# 九、用正则表达式进行搜索

1、MySQL仅支持多数正则表达式实现的一个很小的子集。
```
mysql> select prod_name from products where prod_name regexp '1000' order by prod_name;  // 检索列prod_name包含文本1000的所有行

mysql> select prod_name from products where prod_name regexp '.000' order by prod_name;

mysql> select prod_name from products where prod_name regexp '1000|2000' order by prod_name;

mysql> select prod_name from products where prod_name regexp '[123] Ton' order by prod_name;  // [123]定义一组字符，它的意思是匹配1或2或3，

mysql> select prod_name from products where prod_name regexp '[1-5] Ton' order by prod_name;  // [1-5]定义了一个范围，这个表达式意思是匹配1到5。由于5 ton匹配，所以.5 ton也会返回。

mysql> select vend_name from vendors where vend_name regexp '\\.' order by vend_name;  // 用\\进行转义，匹配含有 . 的字符串。

mysql> select prod_name from products where prod_name regexp '\\([0-9] sticks?\\)' order by prod_name; // \\(匹配(，[0-9]匹配任意数字，sticks?匹配stick和sticks(s后的?使s可选，因为?匹配它前面的任何字符的0次或1次出现)，\\)匹配)。如果没有?，匹配stick和sticks会非常困难。

mysql> select prod_name from products where prod_name regexp '[[:digit:]]{4}' order by prod_name; // [:digit:]匹配任意数字，因而它为数字的一个集合。{4}要求它前面的字符(任意数字)出现4次，所以[[:digit:]]{4}匹配连在一起的任意4位数字。

mysql> select prod_name from products where prod_name regexp '[0-9][0-9][0-9][0-9]' order by prod_name; // 同上，匹配连在一起的任意4位数字。

mysql> select prod_name from products where prod_name regexp '^[0-9\\.]' order by prod_name; // ^匹配串的开始。因此，^[0-9\\.]只在.或任意数字为串中第一个字符时才匹配它们。如果没有^，则还要多检索出那些中间有数字的行。
```

- regexp后所跟的东西为正则表达式。
- .是正则表达式语言中一个特殊的字符。它表示匹配任意一个字符。
- MySQL中的正则表达式匹配（自版本3.23.4后）不区分大小写（即，大写和小写都匹配）。为区分大小写，可使用binary关键字，如where prod_name regexp binary 'JetPack .000'。
- |为正则表达式的or操作符。它表示匹配其中之一。使用|从功能上类似于在select语句中使用or语句，多个or条件可并入单个正则表达式。
- 可以给出两个以上的or条件。例如，'1000|2000|3000'将匹配1000或2000或3000.
- []是另一种形式的or语句。事实上，正则表达式 [123] Ton 为 [1|2|3] Ton 的缩写。
- 字符集合也可以被否定，在集合的开始处放置一个^即可。例如[123]匹配字符1、2或3，但[^123]却匹配除这些字符外的任何东西。
- 集合可用来定义要匹配的一个或多个字符。
- 为了匹配特殊字符，必须用\\\为前导。\\\\-表示查找-，\\\\.表示查找.。
- 为了匹配反斜杠(\\)字符本身，需要使用\\\\\。
- 多数正则表达式实现使用单个反斜杠转义特殊字符，以便能使用这些字符本身。但MySQL要求两个反斜杠(MySQL自己解释一个，正则表达式库解释另一个)。

2、like与regexp之间的一个重要的差别
```
mysql> select prod_name from products where prod_name like '1000' order by prod_name; 

mysql> select prod_name from products where prod_name regexp '1000' order by prod_name; 
```
- like匹配整个列。如果被匹配的文本在列值中出现，like将不会找到它，相应的行也不被返回（除非使用通配符）。
- 而regexp在列值内进行匹配，如果被匹配的文本在列值中出现，regexp将会找到它，相应的行将被返回。

3、\\\也用来引用元字符(具有特殊含义的字符)。

元字符 | 说明
---|---
\\\f | 换页
\\\n | 换行
\\\r | 回车
\\\t | 制表
\\\v | 纵向制表

4、匹配字符类

类 | 说明
---|---
[:alnum:] | 任意字母和数字（同[a-zA-Z0-9]）
[:alpha:] | 任意字符（同[a-zA-Z]）
[:blank:] | 空格和制表符（同[\\\t]）
[:cntrl:] | ACSII控制字符（ASCII0到31和127）
[:digit:] | 任意数字（同[0-9]）
[:graph:] | 与[:print:]相同，但不包括空格
[:lower:] | 任意小写字母（同[a-z]）
[:print:] | 任意可打印字符
[:punct:] | 既不在[:alnum:]又不在[:cntrl:]中的任意字符
[:space:] | 包括空格在内的任意空白字符（同[\\\f\\\n\\\r\\\t\\\v]）
[:upper:] | 任意大写字母（同[A-Z]）
[:xdigit:] | 任意十六进制数字（同[a-fA-F0-9]）

5、正则表达式的重复元字符

元字符 | 说明
---|---
* | 0个或多个匹配
+ | 1个或多个匹配（等于{1,}）
? | 0个或1个匹配（等于{0,1}）
{n} | 指定数目的匹配
{n,} | 不少于指定数目的匹配
{n,m} | 匹配数目的范围（m不超过255）

6、定位元字符

元字符 | 说明
---|---
^ | 文本的开始
$ | 文本的结尾
[[:<:]] | 词的开始
[[:>:]] | 词的结尾

7、^有两种用法。在集合中(用[和]定义)，用它来否定该集合；否则，用来指串的开始处。

8、like和regexp的不同在于，like匹配整个串而regexp匹配子串。利用定位符，通过用^开始每个表达式，用$结束每个表达式，可以使regexp的作用与like一样。

9、可以在不使用数据库表的情况下用select来简单测试正则表达式。 
- regexp检查总是返回0（没有匹配）或1（匹配）。  
- 可以用带文字串的regexp来测试表达式，并试验它们。  
- 相应的语法如下：  
    select 'hello' regexp '[0-9]';
    这个例子显然将返回0（因为文本hello中没有数字）。


# 十、创建计算字段

1、计算字段是运行时在select语句内创建的。

2、拼接：将值联结到一起构成单个值。
在MySQL的select语句中，可使用Concat()函数来拼接两个列。
** 注意：多数DBMS使用+或||来实现拼接，MySQL则使用Concat()函数来实现。当把SQL语句转换成MySQL语句时一定要把这个区别铭记在心。 **

```
mysql> select concat(vend_name, ' (', vend_country, ')') from vendors order by vend_name;

mysql> select concat(rtrim(vend_name), ' (', rtrim(vend_country), ')') from vendors order by vend_name;
```
- Concat()拼接串，即把多个串连接起来形成一个较长的串，各个串之间用逗号分隔。
- 使用MySQL的RTrim()函数来完成删除数据右侧多余的空格。
- 使用MySQL的LTrim()函数来完成删除数据左侧多余的空格。
- 使用MySQL的Trim()函数来完成删除数据两侧多余的空格。

3、别名是一个字段或值的替换名。别名用AS关键字赋予。方便客户机按名引用这个列。
别名有时也称为导出列。
```
mysql> select concat(rtrim(vend_name), ' (', rtrim(vend_country), ')') as vend_title from vendors order by vend_name;
```

4、计算字段的另一常见用途是对检索出的数据进行算术计算。
```
mysql> select prod_id, quantity, item_price, quantity*item_price as expanded_price from orderitems where order_num=20005;
```
MySQL算术操作符
操作符 | 说明
---|---
+ | 加
- | 减
* | 乘
/ | 除

5、select提供了测试和试验函数与计算的一个很好的方法。

虽然select通常用来从表中检索数据，但可以省略from子句以便简单地访问和处理表方式。
例如，select 3*2; 将返回6，select trim('abc'); 将返回abc。
而select now()利用Now函数返回当前日期和时间。

# 十一、使用数据处理函数

1、函数没有SQL的可移植性强。几乎每种主要的DBMS的实现都支持其他实现不支持的函数，而且有时候差异还很大。

2、大多数SQL实现支持以下类型的函数。
- 用于处理文本串（如删除或填充值，转换值为大写或小写）的文本函数。
- 用于在数值数据上进行算术操作（如返回绝对值，进行代数计算）的数值函数。
- 用于处理日期和时间值并从这些值中提取特定成分（例如，返回两个日期之差，检查日期有效性等）的日期和时间函数。
- 返回DBMS正使用的特殊信息（如返回用户登录信息，检查版本细节）的系统函数。

3、文本处理函数
```
mysql> select vend_name, upper(vend_name) as vend_name_upcase from vendors order by vend_name;
```

- Upper()将文本转换为大写。


4、常用的文本处理函数

函数 | 说明
---|---
Left() | 返回串左边的字符
Length() | 返回串的长度
Locate() | 找出串的一个子串
Lower() | 将串转换为小写
LTrim() | 去掉串左边的空格
Right() | 返回串右边的字符
RTrim() | 去掉串右边的空格
Soundex() | 返回串的SOUNDEX值
SubString() | 返回子串的字符
Upper() | 将串转换为大写

注：SOUNDEX是一个将任何文本串转换为描述其语音表示的字母数字模式的算法。
SOUNDEX考虑了类似的发音字符和音节，使得能对串进行发音比较而不是字母比较。
虽然SOUNDEX不是SQL概念，但MySQL(就像多数DBMS一样)都提供对SOUNDEX的支持。
```
mysql> select cust_name, cust_contact from customers where cust_contact='Y.lie';
mysql> select cust_name, cust_contact from customers where soundex(cust_contact)=soundex('Y.lie'); // 匹配所有发音类似于Y.Lie的联系名
```

5、常用的日期和时间处理函数

函数| 说明
---|---
AddDate() | 增加一个日期(天、周等)
AddTime() | 增加一个时间(时、分等)
CurDate() | 返回当前日期
CurTime() | 返回当前时间
Date() | 返回日期时间的日期部分
DateDiff() | 计算两个日期之差
Date_Add() | 高度灵活的日期运算函数
Date_Format() | 返回一个格式化的日期或时间串
Day() | 返回一个日期的天数部分
DayOfWeek() | 对于一个日期，返回对应的星期几
Hour() | 返回一个时间的小时部分
Minute() | 返回一个时间的分钟部分
Month() | 返回一个日期的月份部分
Now() | 返回当前日期和时间
Second() | 返回一个时间的秒部分
Time() | 返回一个日期时间的时间部分
Year() | 返回一个日期的年份部分

```
mysql> select cust_id, order_num from orders where order_date='2005-09-01'; // order_date的值为2005-09-01 00:00:00，但如果order_date的值为2005-09-01 11:30:05，则匹配不上了。
mysql> select cust_id, order_num from orders where date(order_date)='2005-09-01';   // 基于上面的原因，因此要使用Date()函数。

mysql> select cust_id, order_num from orders where date(order_date) between '2005-09-01' and '2005-09-30';   // 输出和下面一样。
mysql> select cust_id, order_num from orders where year(order_date)=2005 and month(order_date)=9;   // 输出和上面一样。
```
- MySQL首选的日期格式为yyyy-mm-dd。
- Date()和Time()都是在MySQL 4.1.1中第一次引入的。

6、常用的数值处理函数

在主要DBMS的函数中，数值函数是最一致最统一的函数。

函数 | 说明
---|---
Abs() | 返回一个数绝对值
Cos() | 返回一个角度的余弦
Exp() | 返回一个数的指数值
Mod() | 返回除操作的余数
Pi() | 返回圆周率
Rand() | 返回一个随机数
Sin() | 返回一个角度的正弦
Sqrt() | 返回一个数的平方根
Tan() | 返回一个角度的正切


# 十二、汇总数据

1、聚集函数

聚集函数：运行在行组上，计算和返回单个值的函数。

MySQL还支持一系列的标准偏差聚集函数。

SQL聚集函数 | 说明 
---|---
AVG() | 返回某列的平均值
COUNT() | 返回某列的行数
MAX() | 返回某列的最大值
MIN() | 返回某列的最小值
SUM() | 返回某列值之和

```
mysql> select avg(prod_price) as avg_price from products;

mysql> select avg(prod_price) as avg_price from products where vend_id=1003;

mysql> select count(*) as num_cust from customers;

mysql> select count(cust_email) as num_cust from customers;

mysql> select max(prod_price) as max_price from products;

mysql> select min(prod_price) as min_price from products;

mysql> select sum(quantity) as items_ordered from orderitems where order_num=20005;

mysql> select sum(item_price*quantity) as total_price from orderitems where order_num=20005;
```
- AVG()通过对表中行数计数并计算特定列值之和，求得该列的平均值。AVG()可用来返回所有列的平均值，也可以用来返回特定列或行的平均值。
- AVG()只能用来确定特定数值列的平均值，而且列名必须作为函数参数给出。为了获得多个列的平均值，必须使用多个AVG()函数。
- AVG()函数忽略列值为NULL的行。
- 可利用COUNT()函数确定表中行的数目或符合特定条件的行的数目。
- COUNT()函数有两种使用方式：①使用COUNT(*)对表中行的数目进行计数，不管表列中包含的是空值(NULL)还是非空值。②使用COUNT(column)对特定列中具有值的行进行计数，忽略NULL值。
- 如果指定列名，则指定列的值为空的行被COUNT()函数忽略，但如果COUNT()函数中用的是星号(*)，则不忽略。
- MAX()返回指定列中的最大值。MAX()要求指定列名。MIN()返回指定列中的最小值。MIN()要求指定列名。
- 虽然MAX()一般用来找出最大的数值或日期值，但MySQL允许将它用来返回任意列中的最大值，包括返回文本列中的最大值。在用于文本数据时，如果数据按相应的列排序，则MAX()返回最后一行，MIN()返回最前面的行。
- MAX()函数忽略列值为NULL的行。MIN()函数忽略列值为NULL的行。
- SUM()用来返回指定列值的和(总和)。SUM()也可以用来合计计算值。
- 利用标准的算术操作符，所有的聚集函数都可用来执行多个列上的计算。
- SUM()函数忽略列值为NULL的行。

2、聚集不同值

以上5个聚集函数都可以如下使用：
- 对所有的行执行计算，指定ALL参数或不给参数（因为ALL是默认行为）；
- 只包含不同的值，指定DISTINCT参数。

```
mysql> select avg(distinct prod_price) as avg_price from products where vend_id=1003;

mysql> select count(*) as num_items, min(prod_price) as price_min, max(prod_price) as price_max, avg(prod_price) as price_avg from products;  
```

- 如果指定列名，则DISTINCT只能用于COUNT(列名)。DISTINCT不能用于COUNT(*)，因此不允许使用CONUNT(DISTINCT *)，否则会产生错误。类似地，DISTINCT必须使用列名，不能用于计算或表达式。
- 虽然DISTINCT从技术上可用于MIN()和MAX()，但这样做实际上没有价值。一个列中的最小值和最大值不管是否包含不同值都是相同的。
- 在指定别名以包含某个聚集函数的结果时，不建议使用表中实际的列名。


# 十三、分组数据

1、创建分组

```
mysql> select vend_id, count(*) as num_prods from products group by vend_id;    // group by子句指示MySQL按vend_id排序并分组数据。 

mysql> select vend_id, count(*) as num_prods from products group by vend_id with rollup;    // 比上面语句多出统计NULL值的一组
```

- 分组允许把数据分为多个逻辑组，以便能对每个组进行聚集计算。
- 分组是在select语句的group by子句中建立的。
- group by子句指示MySQL分组数据，然后对每个组而不是整个结果集进行聚集。
- 使用with rollup关键字，可以得到每个分组以及每个分组汇总级别(针对每个分组)的值。


2、在具体使用group by子句前，需要知道一些重要的规定：
- group by子句可以包含任意数目的列。这使得能对分组进行嵌套，为数据分组提供更细致的控制。
- 如果在group by子句中嵌套了分组，数据将在最后规定的分组上进行汇总。换句话说，在建立分组时，指定的所有列在一起计算(所以不能从个别的列取回数据)。
- group by子句中列出的每个列都必须是检索列或有效的表达式(但不能是聚集函数)。如果在select中使用表达式，则必须在group by子句中指定相同的表达式。不能使用别名。
- 除聚集计算语句外，select语句中的每个列都必须在group by子句中给出。
- 如果分组列中具有NULL值，则NULL将作为一个分组返回。如果列中有多行NULL值，它们将分为一组。
- group by子句必须出现在where子句之后，order by子句之前。

3、过滤分组

```
mysql> select cust_id, count(*) as orders from orders group by cust_id having count(*)>=2;  // 这里的having子句过滤两个以上的订单的那些分组

mysql> select vend_id, count(*) as num_prods from products where prod_price>=10 group by vend_id having count(*)>=2;    // 先用where子句过滤行，然后按vend_id分组，having子句过滤分组。
```

- having子句非常类似于where。事实上，目前为止所学过的所有类型的where子句都可以用having来替代。唯一的差别是where过滤行，而having过滤分组。
- having支持所有where操作符。
- 过滤是基于分组聚集值而不是特定行值的。
- having和where的差别：where在数据分组前进行过滤，having在数据分组后进行过滤。这是一个重要的区别，where排除的行不包括在分组中。这可能会改变计算值，从而影响having子句中基于这些值过滤掉的分组。


4、分组和排序

order by与group by的区别：

order by | group by
---|---
排序产生的输出 | 分组行。但输出可能不是分组的顺序
任意列都可以使用(甚至非选择的列也可以使用) | 只可能使用选择列和表达式列，而且必须使用每个选择列表达式
不一定需要 | 如果与聚集函数一起使用列(或表达式)，则必须使用

一般在使用group by子句时，应该也给出order by子句。这是保证数据正确排序的唯一方法。千万不要仅依赖group by排序数据。

```
mysql> select order_num, sum(quantity*item_price) as ordertotal from orderitems group by order_num having sum(quantity*item_price)>=50;

mysql> select order_num, sum(quantity*item_price) as ordertotal from orderitems group by order_num having sum(quantity*item_price)>=50 order by ordertotal;   // group by子句用来按订单号分组数据，以便sum(*)函数能够返回总计订单价格。having子句过滤数据，使得只返回总订单价格大于等于50的订单。最后order by子句排序输出。
```

5、select子句及其顺序

子句 | 说明 | 是否必须使用
---|---|---
select | 要返回的列或表达式 | 是
from | 从中检索数据的表 | 仅在从表选择数据时使用
where | 行级过滤 | 否
group by | 分组说明 | 仅在按组计算聚集时使用
having | 组级过滤 | 否
order by | 输出排序顺序 | 否
limit | 要检索的行数 | 否


# 十四、使用子查询

1、利用子查询进行过滤
```
mysql> select order_num from orderitems where prod_id='TNT2';
mysql> select cust_id from orders where order_num in (20005, 20007);

mysql> select cust_id from orders where order_num in (select order_num from orderitems where prod_id='TNT2');   // 在select语句中，子查询总是从内向外处理。
```

- 格式化SQL: 包含子查询的select语句难以阅读和调试，特别是它们较为复杂时更是如此。应该适当地进行缩进。
- 对于能嵌套的子查询的数目没有限制，不过在实际使用时由于性能的限制，不能嵌套太多的子查询。
- 在where子句中使用子查询，应该保证select语句具有与where子句中相同数目的列。通常，子查询将返回单个列并且与单个列匹配，但如果需要也可以使用多个列。
- 虽然子查询一般与IN操作符结合使用，但也可以用于测试等于(=)、不等于(<>)等。


2、作为计算字段使用子查询

```
mysql> select cust_name,
              cust_state,
              (select count(*)
               from orders
               where orders.cust_id = customers.cust_id) as orders
       from customers
       order by cust_name;  // orders是一个计算字段，它是由圆括号中的子查询建立的。该子查询对检索出的每个客户执行一次。
```

- 使用子查询的另一方法是创建计算字段。
- "where orders.cust_id = customers.cust_id"：这种类型的子查询称为相关子查询。任何时候只要列名可能有多义性，就必须使用这种语句(表名和列名由一个句点分隔)。
- 用子查询建立(和测试)查询的最可靠的方法是逐渐进行。首先，建立和测试最内层的查询，然后用硬编码数据建立和测试外层查询，并且仅在确认它正常后才嵌入子查询。
- 子查询最常见的使用是在where子句的in操作符中，以及用来填充计算列。


# 十五、联结表

1、SQL最强大的功能之一就是能在数据检索查询的执行中联结表。

- 外键：外键为某个表中的一列，它包含另一个表的主键值，定义了两个表之间的关系。
- 联结是一种机制，用来在一条select语句中关联表，因此称之为联结。
- 维护引用完整性：通过在表的定义中指定主键和外键来实现的。

```
mysql> select vend_name, prod_name, prod_price 
       from vendors, products 
       where vendors.vend_id = products.vend_id
       order by vend_name, prod_name;
       
mysql> select vend_name, prod_name, prod_price 
       from vendors inner join products
       on vendors.vend_id = products.vend_id;   // 执行结果与上一句相同。
       // 这里，两个表之间的关系是from子句的组成部分，以inner join指定。
       // 在使用这种语法时，联结条件用特定的on子句而不是where子句给出。传递给on的实际条件与传递给where的相同。
       
mysql> select prod_name, vend_name, prod_price, quantity
       from orderitems, products, vendors
       where products.vend_id = vendors.vend_id
         and orderitems.prod_id = products.prod_id
         and order_num = 20005;
```
- 完全限定列名：在引用的列可能出现二义性时，必须使用完整限定列名(用一个点分隔的表名和列名)。如果引用一个没有用表名限制的具有二义性的列名，MySQL将返回错误。
- 在联结两个表时，你实际上做的是将第一个表中的每一行与第二个表中的每一行配对。where子句作为过滤条件，它只包含那些匹配给定条件(这里是联结条件)的行。没有where子句，第一个表中的每个行将与第二个表中的每个行配对，而不管它们逻辑上是否可以配在一起。
- 应该保证所有联结都有where子句，否则MySQL将返回比想要的数据多得多的数据。
- 目前为止所用的联结称为等值联结，它基于两个表之间的相等测试。这种联结也称为内部联结。
- ANSI SQL规范首选inner join语法。
- SQL对一条select语句中可以联结的表的数目没有限制。
- MySQL在运行时关联指定的每个表以处理联结。这种处理可能是非常耗费资源的，因此不要联结不必要的表。联结的表越多，性能下降越厉害。


# 十六、创建高级联结

1、SQL允许给表名起别名。这样做有两个主要理由：一是缩短SQL语句；二是允许在单条select语句中多次使用相同的表。
```
mysql> select cust_name, cust_contact 
       from customers as c, orders as o, orderitems as oi
       where c.cust_id = o.cust_id
         and oi.order_num = o.order_num
         and prod_id = 'TNT2';
```
- 应该注意，表别名只在查询执行中使用。与列别名不一样，表别名不返回到客户机。

2、使用不同类型的联结

```
mysql> select prod_id, prod_name
       from products
       where vend_id = (select vend_id from products where prod_id = 'DTNTR');  // 使用子查询
       
mysql> select p1.prod_id, p1.prod_name
       from products as p1, products as p2
       where p1.vend_id = p2.vend_id
         and p2.prod_id = 'DTNTR';  // 使用自联结
       // 此查询用到的两个表实际上是相同的表，所以对products的引用具有二义性。因此使用了表别名来解决该问题。


mysql> select c.*, o.order_num, o.order_date, oi.prod_id, oi.quantity, oi.item_price
       from customers as c, orders as o, orderitems as oi
       where c.cust_id = o.cust_id
         and oi.order_num = o.order_num
         and prod_id = 'FB';    // 这里，通配符只对第一表使用。所有其他列明确列出，所以没有重复的列被检索出来。


mysql> select customers.cust_id, orders.order_num 
       from customers inner join orders
       on customers.cust_id = orders.cust_id;
       
mysql> select customers.cust_id, orders.order_num 
       from customers left outer join orders
       on customers.cust_id = orders.cust_id;   //  比上一语句执行结果多出NULL的一行 
       
mysql> select customers.cust_id, orders.order_num 
       from customers right outer join orders
       on orders.cust_id = customers.cust_id;   //  比上上语句执行结果相同
```
- 3种其他联结：自联结、自然联结和外部联结。
- 使用表别名的主要原因之一是能在单条select语句中不止一次引用相同的表。
- 自联结通常作为外部语句用来替代从相同表中检索数据时使用的子查询语句。有时候使用联结远比处理子查询快得多。
- 自然联结是这样一种联结，其中你只能选择那些唯一的列。这一般是通过对表使用通配符(select *)，对所有其他表的列使用明确的子集来完成的。
- 联结包含了那些在相关表中没有关联行的行，这种类型的联结称为外部联结。
- 在使用outer join语法时，必须使用right或left关键字指定包括其所有行的表(right指出的是outer join右边的表，而left指出的是outer join左边的表)。
- MySQL不支持简化字符*=和=*的使用，这两种操作符在其他DBMS种是很流行的。
- 存在两种基本的外部联结形式：左外部联结和右外部联结。它们之间的唯一差别是所关联的表的顺序不同。换句话说，左外部联结可通过颠倒from或where子句中表的顺序转换为右外部联结。因此，两种类型的外部联结可互换使用。

3、使用带聚集函数的联结
```
mysql> select customers.cust_name, customers.cust_id, count(orders.order_num) as num_ord
       from customers inner join orders on customers.cust_id = orders.cust_id
       group by customers.cust_id;
       
mysql> select customers.cust_name, customers.cust_id, count(orders.order_num) as num_ord
       from customers left outer join orders on customers.cust_id = orders.cust_id
       group by customers.cust_id;  // 比上一语句执行结果多出0个订单的行
```

4、使用联结和联结条件

- 注意所使用的联结类型。一般我们使用内部联结，但使用外部联结也是有效的。
- 保证使用正常的联结条件，否则将返回不正确的数据。
- 应该总是提供联结条件，否则会得出笛卡尔积。
- 在一个联结汇总可以包含多个表，甚至对于每个联结可以采用不同的联结类型。虽然这种做是合法的，一般也很有用，但应该在一起测试它们前，分别测试每个联结。这将使故障排除更为简单。


# 十七、组合查询

1、MySQL允许执行多个查询(多条select语句)，并将结果作为单个查询结果集返回。这些组合查询通常称为并(union)或复合查询(compound query)。

有两种基本情况，其中需要使用组合查询：
- 在单个查询中从不同的表返回类似结构的数据；
- 对单个表执行多个查询，按单个查询返回数据。

多数情况下，组合相同表的两个查询完成的工作与具有多个where子句条件的单条查询完成的工作相同。换句话说，任何具有多个where子句的select语句都可以作为一个组合查询给出。

2、可用union操作符来组合数条SQL查询。
```
mysql> select vend_id, prod_id, prod_price
       from products
       where prod_price <=5
       union
       select vend_id, prod_id, prod_price
       from products
       where vend_id in (1001,1002);    // union指示MySQL执行两条select语句，并把输出组合成单个查询结果集。
       
mysql> select vend_id, prod_id, prod_price
       from products
       where prod_price <=5 or vend_id in (1001,1002);  // 和上面语句执行结果一样
       
mysql> select vend_id, prod_id, prod_price
       from products
       where prod_price <=5
       union
       select vend_id, prod_id, prod_price
       from products
       where vend_id in (1001,1002)
       order by vend_id, prod_price;   
       // 虽然order by子句似乎只是最后一条select语句的组成部分，但实际上MySQL将用它来排序所有select语句返回的所有结果。
```
在进行并时要注意以下几条规则：
- union必须由两条或两条以上的select语句组成，语句之间用关键字union分隔。
- union中的每个查询必须包含相同的列、表达式或聚集函数(不过各个列不需要以相同的次序列出)。
- 列数据类型必须兼容：类型不必完全相同，但必须是DBMS可以隐含地转换的类型(例如，不同的数值类型或不同的日期类型)。

其他要点：
- union从查询结果集中自动去除了重复的行(换句话说，它的行为与单条select语句中使用多个where子句条件一样)。
这是union的默认行为，可以使用union all返回所有匹配行。
union all完成了where子句完成不了的工作。

- 在用union组合查询时，只能使用一天order by子句，它必须出现在最后一条select语句之后。因为对于结果集，不存在用一种方式排序一部分，而又用另一种方式排序另一部分的情况，因此不允许使用多条order by子句。

- union的组合查询可以应用不同的表。

- 使用union可极大地简化复杂的where子句，简化从多个表中检索数据的工作。


# 十八、全文本检索

1、MySQL支持几种基本的数据库引擎。并非所有的引擎都支持本书所描述的全文本搜索。两个最常使用的引擎为MyISAM和InnoDB，前者支持全文本搜索，而后者不支持。

在使用全文本搜索时，MySQL不需要分别查看每个行，不需要分别分析和处理每个词。MySQL创建指定列中各词的一个索引，搜索可以针对这些词进行。这样，MySQL可以快速有效地决定哪些词匹配(哪些行包含它们)，哪些词不匹配，它们匹配的频率，等等。

为了进行全文本搜索，必须索引被搜索的列，而且要随着数据的改变不断地重新索引。在对表列进行适当设计后，MySQL会自动进行所有的索引和重新索引。
在索引之后，select可与Match()和Against()一起使用以实际执行搜索。

```
CREATE TABLE productnotes
(
  note_id    int           NOT NULL AUTO_INCREMENT,
  prod_id    char(10)      NOT NULL,
  note_date datetime       NOT NULL,
  note_text  text          NULL ,
  PRIMARY KEY(note_id),
  FULLTEXT(note_text)
) ENGINE=MyISAM;
```

- 一般在创建表时启用全文本搜索。create table语句接受fulltext子句，它给出被索引列的一个逗号分隔的列表。
- 在定义之后，MySQL自动维护该索引。在增加、更新或删除行时，索引随之自动更新。
- 可以在创建表时指定fulltext，或者在稍后指定(在这种情况下所有已有数据必须立即索引)。
- 如果正在导入数据到一个新表，此时不应该启用fulltext索引。应该首先导入所有数据，然后再修改表，定义fulltext。这样有助于更快地导入数据(而且使索引数据的总时间小于在导入每行时分别进行索引所需的总时间)。

2、使用全文本搜索

```
mysql> select note_text from productnotes where match(note_text) against('rabbit');
mysql> select note_text from productnotes where note_text like '%rabbit%';
// 前者(使用全文本搜索)返回以文本匹配的良好程序排序的数据。两个行都包含词rabbit，但包含词rabbit作为第3个词的行的等级比作为第20个词的行高。这很重要。全文本搜索的一个重要部分就是对结果排序。具有较高等级的行先返回(因为这些行很可能是你真正想要的行)。

mysql> select note_text, 
              match(note_text) against('rabbit') as rank
       from productnotes;   // 因为没有where子句，所以所有行都被返回。
       // Match()和Against()用来建立一个计算列(别名为rank)，此列包含全文本搜索计算出的等级值。不包含词rabbit的行等级为0。文本中词靠前的行的等级值比词靠后的行的等级值高。
```

- 在索引之后，使用两个函数Match()和Against()执行全文本搜索，其中Match()指定被搜索的列，Against()指定要使用的搜索表达式。
- 传递给match()的值必须与fulltext()定义中的相同。如果指定多个列，则必须列出它们(而且次序正确)。
- 除非使用binary方式，否则全文本搜索不区分大小写。
- 等级由MySQL根据行中词的数目、唯一词的数目、整个索引中词总数以及包含该词的行的数目计算出来。
- 如果指定多个搜索项，则包含多数匹配词的那些行将具有比包含较少词(或仅有一个匹配)的那些行高的等级值。
- 全文本搜索提供了简单like搜索不能提供的功能。而且，由于数据是索引的，全文本搜索还相当快。

3、利用查询扩展，能找出可能相关的结果，即使它们并不精确包含所查找的词。

在使用查询扩展时，MySQL对数据和索引进行两遍扫描来完成搜索：
- 首先，进行一个基本的全文本搜索，找出与搜索条件匹配的所有行；
- 其次，MySQL检查这些匹配行并选择所有有用的词;
- 再其次，MySQL再次进行全文本搜索，这次不仅使用原来的条件，而且还使用所有有用的词。

```
mysql> select note_text from productnotes where match(note_text) against('anvils'); // 只有一行包含词anvils，因此只返回一行。

mysql> select note_text from productnotes where match(note_text) against('anvils' with query expansion);    // 这次返回了7行。
```
- 查询扩展极大地增加了返回的行数，但这样做也增加了你实际上并不想要的行的数目。
- 表中的行越多(这些行中的文本就越多)，使用查询扩展返回的结果越好。


4、布尔文本搜索

MySQL支持全文本搜索的另外一种形式，称为布尔方式。以布尔方式，可以提供关于如下内容的细节：
- 要匹配的词；
- 要排斥的词(如果某行包含这个词，则不返回该行，即使它包含其他指定的词也是如此)；
- 排序提示(指定某些词比其他词更重要，更重要的词等级更高)；
- 表达式分组；
- 另外一些内容。

布尔方式不同于迄今为止使用的全文本搜索语法的地方在于，即使没有定义fulltext索引，也可以使用它。但这是一种非常缓慢的操作(其性能将随着数据量的增加而降低)。

```
mysql> select note_text from productnotes where match(note_text) against('heavy' in boolean mode); // 这里使用了关键字in boolean mode，但实际上没有指定布尔操作符。因此，其结果与没有指定布尔方式的结果相同。

mysql> select note_text from productnotes where match(note_text) against('heavy -rope*' in boolean mode);   // 匹配包含heavy但不包含任意以rope开始的词的行 
```

5、全文本布尔操作符

布尔操作符 | 说明
---|---
+ | 包含，词必须存在
- | 排除，词必须不出现
> | 包含，而且增加等级值
< | 包含，而且减少等级值
() | 把词组成子表达式(允许这些子表达式作为一个组被包含、排序、排列等)
~ | 取消一个词的排序值
* | 词尾的通配符
"" | 定义一个短语(与单个词的列表不一样，它匹配整个短语以便包含或排除这个短语)

```
mysql> select note_text from productnotes where match(note_text) against('rabbit bait' in boolean mode); // 没有指定操作符，这个搜索匹配包含rabbit和bait中的至少一个词的行

mysql> select note_text from productnotes where match(note_text) against('"rabbit bait"' in boolean mode); // 这个搜索匹配短语rabbit bait而不是匹配两个词rabbit和bait。

mysql> select note_text from productnotes where match(note_text) against('>rabbit <carrot' in boolean mode); // 匹配rabbit和carrot，增加前者的等级，降低后者的等级

mysql> select note_text from productnotes where match(note_text) against('+safe +(<combination)' in boolean mode);  // 这个搜索匹配词safe和combination，降低后者的等级
```

- 在布尔方式中，不按等级值降序排序返回的行。

6、全文本搜索的某些重要说明
- 在索引全文本数据时，短词被忽略且从索引中排除。短词定义为那些具有3个或3个以下字符的词(如果需要，这个数目可以更改)。
- MySQL带有一个内建的非用词(stopword)列表，这些词在索引全文本数据时总是被忽略。如果需要，可以覆盖这个列表。
- 许多词出现的频率很高，搜索它们没有用处(返回太多的结果)。因此，MySQL规定了一条50%规则，如果一个词出现在50%以上的行中，则将它作为一个非用词忽略。50%规则不用于in boolean mode。
- 如果表中的行数少于3行，则全文本搜索不返回结果(因为每个词或者不出现，或者至少出现在50%的行中)。
- 忽略词中的单引号。例如，don't索引为dont。
- 不具有词分割符(包括日语和汉语)的语言不能恰当地返回全文本搜索结果。
- 仅在MyISAM数据库引擎中支持全文本搜索。

邻近搜索是许多全文本搜索支持的一个特性，它能搜索相邻的词(在相同的句子中、相同的段落中或者在特定数目的词的部分中，等等)。MySQL全文本搜索现在还不支持邻近操作符，不过未来的版本有支持这种操作符的计划。


# 十九、插入数据

1、insert是用来插入(或添加)行到数据库表的。插入可以用几种方式使用：
- 插入完整的行；
- 插入行的一部分；
- 插入多行；
- 插入某些查询的结果。

可针对每个表或每个用户，利用MySQL的安全机制禁止使用insert语句。

2、插入完整的行
```
mysql> insert into customers 
       values(NULL,
          'pep E.Lapew',
          '100 Main Street',
          'Los Angeles',
          'CA', 
          '90046', 
          'USA',
          NULL,
          NULL);    // 每个列必须以它们在表定义中出现的次序填充。
          
mysql> insert into customers(cust_name,
          cust_address,
          cust_city,
          cust_state,
          cust_zip,
          cust_country,
          cust_contact,
          cust_email) 
       values('pep E.Lapew',
          '100 Main Street',
          'Los Angeles',
          'CA', 
          '90046', 
          'USA',
          NULL,
          NULL);    // 在插入行时，MySQL将用values列表中的相应值填入列表中的对应项。
```
- insert语句一般不会产生输出。
- 一般不要使用没有明确给出列的列表的insert语句。使用列的列表能使SQL代码继续发挥作用，即使表结构发生了变化。
- 不管使用哪种insert语法，都必须给出values的正确数目。如果不提供列名，则必须给每个表列提供一个值。如果提供列名，则必须对每个列出的列给出一个值。如果不这样，将产生一条错误信息，相应的行插入不成功。
- 如果表的定义允许，则可以在insert操作中省略某些列。省略的列必须满足以下某个条件：一是该列定义为允许NULL值(无值或空值)；二是在表定义中给出默认值。这表示如果不给出值，将使用默认值。
- 如果对表中不允许NULL值且没有默认值的列不给出值，则MySQL将产生一条错误信息，并且相应的行插入不成功。
- insert操作可能是很耗时(特别是有很多索引需要更新时)，而且它可能降低等待处理的select语句的性能。如果数据检索是最重要的(通常是这样)，则你可以通过在insert和into之间添加关键字low_priority，指示MySQL降低insert语句的优先级。例如 insert low_priority into xxxxxx。该关键字也同时适用于update和delete语句。

3、插入多个行
```
mysql> insert into customers(cust_name,
          cust_address,
          cust_city,
          cust_state,
          cust_zip,
          cust_country) 
        values('Pep E. LaPew',
                '100 Main Street',
                'Los Angeles',
                'CA',
                '90046',
                'USA'
               ),
               ('M. Martian',
                '42 Galaxy Way',
                'New York',
                'NY',
                '11213',
                'USA'); 
```
- 可以使用多条insert语句，甚至一次提交它们，每条语句用一个分号结束。
- 或者，只要每条insert语句中的列名(和次序)相同，可以在values字段后面接上多组值，每组值用一对圆括号括起来，用逗号分隔。
- MySQL用单条insert语句处理多个插入比使用多条insert语句快。

4、插入检索出的数据
```
mysql> insert into customers(cust_id,
           cust_contact,
           cust_email,
           cust_name,
           cust_address,
           cust_city,
           cust_state,
           cust_zip,
           cust_country) 
        select cust_id,
            cust_contact,
            cust_email,
            cust_name,
            cust_address,
            cust_city,
            cust_state,
            cust_zip,
            cust_country 
        FROM custnew;       // 使用insert select从custnew中将所有数据导入customers中。
```
- select中列出的每个列对应于customers表名后所跟的列表中的每个列。
- 为简单起见，这个例子在insert和select语句中使用了相同的列名。但是，不一定要求列名匹配。事实上，MySQL甚至不关心select返回的列名。它使用的是列的位置，因此select中的第一列(不管其列名)将用来填充表列中指定的第一个列，以此类推。这对于从使用不同列名的表中导入数据是非常有用的。
- insert select中的select语句可包含where子句以过滤插入的数据。

# 二十、更新数据

1、为了更新(修改)表中的数据，可使用update语句。可采用两种方式使用update：更新表中特定行、更新表中所有行。
- 不要省略where子句：在使用update时一定要注意细心，因为稍不注意，就会更新表中所有行。
- update与安全；可以限制和控制update语句的使用。

基本的update语句由3部分组成，分别是：
- 要更新的表；
- 列名和它们的新值；
- 确定要更新行的过滤条件。

```
mysql> update customers set cust_email = 'elmer@fudd.com' where cust_id = 10005;    // 没有where子句，MySQL将会用这个电子邮件更新customers表中所有行。

mysql> update customers set cust_name = 'The Fudds', cust_email = 'elmer@fudd.com' where cust_id = 10005;

mysql> update customers set cust_email = NULL where cust_id = 10005;    // 其中NULL用来去除cust_email列中的值。
```
- 在更新多个列时，只需要使用单个set命令，每个“列=值”对之间用逗号分隔(最后一列之后不用逗号)。
- update语句中可以使用子查询，使得能用select语句检索出的数据更新列数据。
- 如果用update语句更新多行，并且在更新这些行中的一行或多行时出现一个错误，则整个update操作被取消(错误发生前更新的所有行将恢复到它们原来的值)。为即使是发生错误，也继续进行更新，可使用ignore关键字，如下所示：update ignore customers xxxxx.
- 为了删除某个列的值，可设置它为NULL(假如表定义允许NULL值)。


2、为了从一个表中删除(去掉)数据，使用delete语句。可以两种方式使用delete：从表中删除特定的行、从表中删除所有行。
- 不要省略where子句：在使用delete时一定要注意细心，因为稍不注意，就会错误地删除表中所有行。
- delete与安全；可以限制和控制delete语句的使用。

```
mysql> delete from customers where cust_id = 10006; // delete from要求指定从中删除数据的表名。where子句过滤要删除的行。
```
- delete不需要列名或通配符。delete删除整行而不是删除列。为了删除指定的列，请使用update语句。
- delete语句从表中删除行，甚至是删除表中所有行。但是，delete不删除表本身。
- 如果想从表中删除所有行，不要使用delete。可使用truncate table语句，它完成相同的工作，但速度更快(truncate实际是删除原来的表并重新创建了一个表，而不是逐行删除表中的数据)。

3、更新和删除的指导原则
- 除非确实打算更新和删除每一行，否则绝对不要使用不带where子句的update或delete语句。
- 保证每个表都有主键，尽可能像where子句那样使用它(可以指定各主键、多个值或值的范围)。
- 在对update或delete语句使用where子句前，应该先用select进行测试，保证它过滤的是正确的记录，以防编写的where子句不正确。
- 使用强制实施引用完整性的数据库，这样MySQL将不允许删除具有与其他表相关联的数据的行。

注意：MySQL没有撤销(undo)按钮。应该非常小心地使用update和delete，否则你会发现自己更新或删除了错误的数据。

# 二十一、创建和操纵表

1、一般有两种创建表的方法：
- 使用具有交互式创建和管理表的工具；
- 表也可以直接用MySQL语句操纵。

为利用create table创建表，必须给出下列信息：
- 新表的名字，在关键字create table之后给出；
- 表列的名字和定义，用逗号分隔。

```
mysql> create table orders20220227
       (
         order_num int not null auto_increment,
         order_date datetime not null,
         cust_id int not null,
         order_name char(50) null,
         primary key (order_num)
       ) engine = innodb;
       
mysql> create TABLE orderitems20220227
       (
         order_num int NOT NULL,
         order_item int  NOT NULL,
         prod_id char(10) NOT NULL,
         quantity int NOT NULL DEFAULT 1,
         item_price decimal(8,2) NOT NULL,
         PRIMARY KEY(order_num,order_item)
        ) ENGINE = InnoDB;

```
- 每列的定义以列名(它在表中必须是唯一的)开始，后跟列的数据类型。
- 表的主键可以在创建表时用primary key关键字指定。
- 在创建新表时，指定的表名必须不存在，否则将出错。
- 如果你仅想在一个表不存在时创建它，应该在表名后给出if not exists。这样做不检查已有表的模式是否与你打算创建的表模式相匹配。它只是查看表名是否存在，并且仅在表名不存在时创建它。
- NULL值就是没有值或缺值。允许NULL值的列也允许在插入行时不给出该列的值。不允许NULL值的列不接受该列没有值的行，换句话说，在插入或更新行时，该列必须有值。
- 每个表列或者是NULL列，或者是NOT NULL列，这种状态在创建时由表的定义规定。
- NULL为默认设置，如果不指定not null，则认为指定的是null。
- 不要把NULL值与空串相混淆。NULL值是没有值，它不是空串。如果指定''(两个单引号，其间没有字符)，这在not null列中是允许的。空串是一个有效的值，它不是无值。NULL值用关键字NULL而不是空串指定。
- 主键值必须唯一。即，表中的每个行必须具有唯一的主键值。如果主键使用单个列，则它的值必须唯一。如果使用多个列，则这些列的组合值必须唯一。
- 主键可以在创建表时定义，或者在创建表之后定义。
- 主键中只能使用不允许NULL值的列。允许NULL值的列不能作为唯一标识。
- auto_increment告诉MySQL，本列每当增加一行时自动增量。每次执行一个insert操作时，MySQL自动对该列增量(从而才有这个关键字auto_increment)，给该列赋予下一个可用的值。这样给每个行分配一个唯一的order_num，从而可以作为主键值。
- 每个表只允许一个auto_increment列，而且它必须被索引(如，通过使它成为主键)。
- 覆盖auto_increment：可以简单地在insert语句中指定一个值，只要它是唯一的(至今尚未使用过)即可，该值将被用来替代自动生成的值。后续的增量将开始使用该手工插入的值。
- 通过使用last_insert_id()函数获得使用auto_increment列时的值。例如 select last_insert_id() 此语句返回最后一个auto_increment值，然后可以将它用于后续的MySQL语句。
- MySQL允许指定在插入行没有给出值时使用默认值，默认值用create table语句的列定义中的default关键字指定。
- 与大多数DBMS不一样，MySQL不允许使用函数作为默认值，它只支持常量。

2、几个需要知道的引擎
- InnoDB是一个可靠的事务处理引擎，它不支持全文本搜索；
- MEMORY在功能等同于MyISAM，但由于数据存储在内存(不是磁盘)中，速度很快(特别适合于临时表)；
- MyISAM是一个性能极高的引擎，它支持全文本搜索，但不支持事务处理。

一个数据库中，引擎类型可以混用。但是外键不能跨引擎，即是用一个引擎的表不能引用具有使用不同引擎的表的外键。

3、为更新表定义，可使用alter table语句。但是理想状态下，当表中存储数据以后，该表就不应该再被更新。

为了使用alter table更改表结构，必须给出下面的信息：
- 在alter table之后给出要更改的表名(该表必须存在，否则将出错)；
- 所做更改的列表。

```
mysql> alter table vendors add vend_phone char(20); // 添加一个列

mysql> alter table vendors drop column vend_phone;  // 删除上面添加的列

mysql> ALTER TABLE orderitems ADD CONSTRAINT fk_orderitems_orders FOREIGN KEY (order_num) REFERENCES orders (order_num);    // alter table的一种常见用途是定义外键
```


复杂的表结构更改一般需要手动删除过程，它涉及以下步骤：
- 用新的列布局创建一个新表；
- 使用INSERT SELECT语句从旧表复制数据到新表。如果有必要，可使用转换函数和计算字段；
- 检验包含所需数据的新表；
- 重命名旧表（如果确定，可以删除它）；
- 用旧表原来的名字重命名新表；
- 根据需要，重新创建触发器、存储过程、索引和外键。

注意：使用alter table要极为小心，应该在进行改动前做一个完整的备份(模式和数据的备份)。数据库表的更改不能撤销，如果增加了不需要的列，可能不能删除它们。类似地，如果删除了不应该删除的列，可能会丢失该列中的所有数据。

4、删除表(删除整个表而不是其内容)使用drop table语句。
```
mysql> drop table orders20220227;
```
- 删除表没有确认，也不能撤销，执行这条语句将永久删除该表。

5、使用rename table语句可以重命名一个表。
```
mysql> rename table orderitems20220227 to orderitems20220227001;

mysql> rename table orderitems20220227 to orderitems20220227001,
                    orders20220227 to orders20220227001;
```


# 二十二、使用视图

1、MySQL 5添加了对视图的支持。

视图是虚拟的表。与包含数据的表不一样，视图只包含使用时动态检索数据的查询。

视图的一些常见应用：
- 重用SQL语句。
- 简化复杂的SQL操作。在编写查询后，可以方便地重用它而不必知道它的基本查询细节。
- 使用表的组成部分而不是整个表。
- 保护数据。可以给用户授予表的特定部分的访问权限而不是整个表的访问权限。
- 更改数据格式和表示。视图可返回与底层表的表示和格式不同的数据。

在视图创建之后，可以用与表基本相同的方式利用它们。可以对视图执行select操作，过滤和排序数据，将视图联结到其他视图或表，甚至能添加和更新数据(添加和更新数据存在某些限制)。

重要的是知道视图仅仅是用来查看存储在别处的数据的一种设施。视图本身不包含数据，因此它们返回的数据是从其他表中检索出来的，在添加或更改这些表中的数据时，视图将返回改变过的数据。

因为视图不包含数据，所以每次使用视图时，都必须处理查询执行时所需的任一个检索。如果你用多个联结和过滤创建了复杂的视图或者嵌套了视图，可能会发现性能下降得很厉害。因此，在部署使用了大量视图的应用前，应该进行测试。

2、关于视图创建和使用的一些最常见的规则和限制：
- 和表一样，视图必须唯一命名(不能给视图取与别的视图或表相同的名字)。
- 对于可以创建的视图数目没有限制。
- 为了创建视图，必须具有足够的访问权限。这些限制通常由数据库管理人员授予。
- 视图可以嵌套，即可以利用从其他视图中检索数据的查询来构造一个视图。
- order by可以用在视图中，但如果从该视图检索数据select中也含有order by，那么该视图中的order by将被覆盖。
- 视图不能索引，也不能有关联的触发器或默认值。
- 视图可以和表一起使用。例如，编写一条联结表和视图的select语句。

3、使用视图

视图的创建：
- 视图用create view语句来创建。
- 使用show create view viewname;来查看创建视图的语句。
- 用drop删除视图，其语法为drop view viewname;。
- 更新视图时，可以先用drop再用create，也可以直接用create or replace view。如果要更新的视图不存在，则第2条更新语句会创建一个视图；如果要更新的视图存在，则第2条更新语句会替换原有视图。

```
mysql> select cust_name, cust_contact 
       from customers, orders, orderitems
       where customers.cust_id = orders.cust_id
         and orderitems.order_num = orders.order_num
         and prod_id = 'TNT2';      // 没有创建视图的联结查询
         
mysql> create view productcustomers as 
       select cust_name, cust_contact, prod_id      // 这里必须要有prod_id，否则下面查询时会报没有prod_id列
       from customers, orders, orderitems
       where customers.cust_id = orders.cust_id
         and orderitems.order_num = orders.order_num;
mysql> select cust_name, cust_contact 
       from productcustomers
       where prod_id = 'TNT2';      // 创建视图的联结查询
```
- 可以看出，视图极大地简化了复杂SQL语句的使用。利用视图，可一次性编写基础的SQL，然后根据需要多次使用。
- 创建不受特定数据限制的视图是一种好办法。扩展视图的范围不仅使得它能被重用，而且甚至更有用。这样做不需要创建和维护多个类似视图。


4、视图的另一常见用途是重新格式化检索出的数据。

```
mysql> select concat(rtrim(vend_name), ' (', rtrim(vend_country), ')') as vend_title from vendors order by vend_name;

mysql> create view vendorlocations as select concat(rtrim(vend_name), ' (', rtrim(vend_country), ')') as vend_title from vendors order by vend_name;
mysql> select * from vendorlocations;
```

5、用视图过滤不想要的数据

视图对于应该普通的where子句也很有用。

```
mysql> create view customeremaillist as select cust_id, cust_name, cust_email from customers where cust_email is not null;

mysql> select * from customeremaillist;
```
- 如果从视图检索数据时使用了一条where子句，则两组子句(一组在视图中，另一组是传递给视图的)将自动组合。


6、使用视图与计算字段

```
mysql> select prod_id, quantity, item_price, quantity*item_price as expanded_price from orderitems where order_num=20005;

mysql> create view orderitemsexpanded as select order_num, prod_id, quantity, item_price, quantity*item_price as expanded_price from orderitems; 
mysql> select * from orderitemsexpanded where order_num=20005;
```
- 可以看出，视图非常容易创建，而且很好使用。正常使用，视图可极大地简化复杂的数据处理。

7、通常，视图是可更新的(即，可以对它们使用insert、update和delete)。
更新一个视图将更新其基表(视图本身没有数据)。如果你对视图增加或删除行，实际上是对其基表增加或删除行。

但是，并非所有视图都是可更新的。基本上可以说，如果MySQL不能正确地确定被更新的基数据，则不允许更新(包括插入和删除)。这实际上意味着，如果视图定义中有以下操作，则不能进行视图的更新(下面这些限制自MySQL 5以来是正确的，不过未来可能会取消某些限制)：
- 分组(使用group by和having)；
- 联结；
- 子查询；
- 并；
- 聚集函数(Min()、Count()、Sum()等)；
- distinct；
- 导出(计算)列。

视图主要用于数据检索。一般，应该将视图用于检索(select语句)而不用于更新(insert、update和delete)。

视图为虚拟的表。它们包含的不是数据而是根据需要检索数据的查询。视图提供了一种MySQL的select语句层次的封装，可用来简化数据处理以及重新格式化基础数据或保护基础数据。


# 二十三、使用存储过程

1、MySQL 5添加了对存储过程的支持。

存储过程就是为以后的使用而保存的一条或多条MySQL语句的集合。可将其视为批文件，虽然它们的作用不仅限于批处理。

2、为什么要使用存储过程?
- 通过把处理封装在容易使用的单元中，简化复杂的操作。
- 由于不要求反复建立一系列处理步骤，这保证了数据的完整性。
- 简化对变动的管理。
- 提供性能。因为使用存储过程比使用单独的SQL语句要快。
- 存在一些只能用在单个请求中的MySQL元素和特性，存储过程可以使用它们来编写功能更强更灵活的代码。

换句话说，使用存储过程有3个主要的好处，即简单、安全、高性能。
不过，在将SQL代码转换为存储过程前，也必须知道它的一些缺陷：
- 一般来说，存储过程的编写比基本SQL语句复杂，编写存储过程需要更高的技能，更丰富的经验。
- 你可能没有创建存储过程的安全访问权限。
尽管有这些缺陷，存储过程还是非常有用的，并且应该尽可能地使用。


3、使用存储过程

MySQL称存储过程的执行为调用，因此MySQL执行存储过程的语句为CALL。
CALL接受存储过程的名字以及需要传递给它的任意参数。
```
mysql> call productpricing(@pricelow, @pricehigh, @priceaverage);   // 执行名为productpricing的存储过程，它计算并返回产品的最低、最高和平均价格。

mysql> create procedure productpricing()        // 如果存储过程接受参数，它们将在()中列举出来。
       begin
          select avg(prod_price) as priceaverage
          from products;
       end;     // 该语句没有返回数据，因为这段代码并未调用存储过程，这里只是为以后使用而创建它。
```
- 存储过程可以显示结果，也可以不显示结果。
- begin和end语句用来限定存储过程体，过程体本身仅是一个简单的select语句。

4、mysql命令行客户机的分割符

默认的MySQL语句分隔符为;。mysql命令行实用程序也使用;作为语句分隔符。  
如果命令行实用程序要解释存储过程自身内的;字符，则它们最终不会成为存储过程的成分，这会使存储过程中的SQL出现句法错误。  
解决方法是临时更改命令行实用程序的语句分隔符，如下所示：
```
DELIMITER //
create procedure productpricing()
begin
   select avg(prod_price) as priceaverage
   from products;
end //
DELIMITER ;
```
- 如果存储过程接受参数，它们将在()中列举出来。
其中，DELIMITER //告诉命令行实用程序使用//作为新的语句结束分隔符，可以看到标志存储过程结束的end定义为end //而不是end;。这样，存储过程体内的;仍然保持不动，并且正确地传递给数据库引擎。最后，为恢复为原来的语句分隔符，可使用DELIMITER ;。
除\符号外，任何字符都可以用作语句分隔符。

5、
```
mysql> call productpricing();           // 使用上面创建的存储过程

mysql> drop procedure productpricing;   // 删除存储过程
```
- 因为存储过程实际上是一种函数，所以存储过程名后需要有()符号。
- 存储过程在创建之后，被保存在服务器上以供使用，直至被删除。
- 删除存储过程语句后面没有使用()，只给出存储过程名即可。
- 如果指定的过程不存在，则drop procedure将产生一个错误。当过程存在想删除它时(如果过程不存在也不产生错误)可使用drop procedure if exists。
- 一般，存储过程并不显示结果，而是把结果返回给你指定的变量。
- 变量：内存中一个特定的位置，用来临时存储数据。

```
mysql> create procedure productpricing(
        out pl decimal(8,2),
        out ph decimal(8,2),
        out pa decimal(8,2)
       )
       begin
        select Min(prod_price)
        into pl
        from products;
        select Max(prod_price)
        into ph
        from products;
        select Avg(prod_price)
        into pa
        from products;
       end;
mysql> call productpricing(@pricelow, @pricehigh, @priceaverage);   // 该语句并不显示任何数据，它返回以后可以显示的变量
mysql> select @pricehigh,  @pricelow, @priceaverage;        // 显示出结果

mysql> create procedure ordertotal(
        in onumer int,
        out ototal decimal(8,2)
       ) comment 'this is comment...'
       begin
        declare aaa boolean default 6;   -- 定义了一个变量aaa
        select sum(item_price*quantity)
        from orderitems
        where order_num = onumer
        into ototal;
        
        if aaa then
            -- TODO 如果aaa为真
        end if;
       end;
mysql> call ordertotal(20005, @total);
mysql> select @total;

mysql> show create procedure ordertotal;    // 显示用来创建一个存储过程的create语句。

mysql> show procedure status;   // 获得包括何时、由谁创建等详细信息的存储过程列表。
mysql> show procedure status like 'ordertotal';   // 为限制其输出，可使用like指定一个过滤模式。
```
- 每个参数必须指定的类型，这里使用十进制值。
- 关键字out指出相应的参数用来从存储过程传出一个值(返回给调用者)。
- MySQL支持in(传递给存储过程)、out(从存储过程传出)和inout(对存储过程传入和传出)类型的参数。
- 通过指定into关键字保存到相应的变量。
- 存储过程的参数允许的数据类型与表中使用的数据类型相同。注意，记录集不是允许的类型，因此，不能通过一个参数返回多个行和列。
- 变量名：所有MySQL变量都必须以@开始。
- 注释：前面放置--。
- 用declare语句定义局部变量。declare要求指定变量名和数据类型，它也支持可选的默认值。
- if语句检查aaa是否为真。
- comment关键字：非必需的，但如果给出，将在show procedure status的结果中显示。
- boolean值指定为1表示真，指定为0表示假(实际上，非零值都考虑为真，只有0被视为假)。
- if语句还支持elseif和else子句(前者还使用then子句，后者不使用)。


# 二十四、使用游标

1、MySQL 5添加了对游标的支持。

- 游标是一个存储在MySQL服务器上的数据库查询，它不是一条select语句，而是被该语句检索出来的结果集。  
- 在存储了游标之后，应用程序可以根据需要滚动或浏览其中的数据。  
- 游标主要用于交互式应用，其中用户需要滚动屏幕上的数据，并对数据进行浏览或做出更改。  
- 不像多数DBMS，MySQL游标只能用于存储过程(和函数)。

2、使用游标

使用游标涉及几个明确的步骤：
- 在能够使用游标前，必须声明(定义)它。这个过程实际上没有检索数据，它只是定义要使用select语句。
- 一旦声明后，必须打开游标以供使用。这个过程用前面定义的select语句把数据实际检索出来。
- 对于填有数据的游标，根据需要取出(检索)各行。
- 在结束游标使用前，必须关闭游标。

游标用declare语句创建。declare命名游标，并定义相应的select语句，根据需要带where和其他子句。

```
mysql> create procedure processorders()     // declare语句用来定义和命名游标。存储过程处理完成后，游标就消失(因为它局限于存储过程)。
       begin
        declare ordernumbers cursor
        for 
        select order_num from orders;
       end;
mysql> open ordernumbers;
mysql> close ordernumbers;

mysql> create procedure processorders()     // 这个存储过程声明、打开和关闭一个游标。但对检索出的数据什么也没做。
       begin
        -- Declare the cursor
        declare ordernumbers cursor
        for 
        select order_num from orders;
        
        -- Open the cursor
        open ordernumbers;
        
        -- Close the cursor
        close ordernumbers;
       
       end;
```
- 游标用open cursor语句来打开。
- 在处理open语句时执行查询，存储检索出的数据以供浏览和滚动。
- close释放游标使用的所有内部内存和资源，因此在每个游标不再需要时都应该关闭。
- 在一个游标关闭后，如果没有重新打开，则不能使用它。但是，使用声明过的游标不需要再次声明，用open语句打开它就可以了。
- 如果你不明确关闭游标，MySQL将会在到达END语句时自动关闭它。

3、使用游标数据

在一个游标被打开后，可以使用fetch语句分别访问它的每一行。fetch指定检索什么数据(所需的列)，检索出来的数据存储在什么地方。它还向前移动游标中的内部行指针，使下一条fetch语句检索下一行(不重复读取同一行)。

```
// 从游标中检索单个行(第一行)。其中fetch用来检索当前行的order_num列(将自动从第一行开始)到一个名为o的局部声明的变量中。
mysql> create procedure processorders()
       begin
        -- Declare local variables
        declare o int;
        
        -- Declare the cursor
        declare ordernumbers cursor
        for
        select order_num from orders;
        
        -- Open the cursor
        open ordernumers;
        
        -- Get order numer
        fetch ordernumers into o;       
        
        -- Close the cursor
        close ordernumers;
       end;

// 循环检索数据，从第一行到最后一行。
// 如果调用这个存储过程，它将定义几个变量和一个continue handler，定义并打开一个游标，重复读取所有行，然后关闭游标。
// 如果一切正常，你可以在循环内放入任意需要的处理(在fetch语句之后，循环结束之前)。
mysql> create procedure processorders()
       begin
        -- Declare local variables
        declare done boolean default 0;
        declare o int;
        
        -- Declare the cursor
        declare ordernumbers cursor
        for
        select order_num from orders;
        
        -- Declare continue handler
        declare continue handler for sqlstate '02000' set done = 1; 
        
        -- Open the cursor
        open ordernumers;
        
        -- Loop through all rows
        repeat
            -- Get order number
            fetch ordernumers into o;
        
        -- End of loop
        until done end repeat;
        
        -- Close the cursor
        close ordernumers;
       end;
       
// 此存储过程不返回数据，但它能够创建和填充另一个表。       
mysql> create procedure processorders()
       begin
        -- Declare local variables
        declare done boolean default 0;
        declare o int;
        declare t decimal(8,2);
        
        -- Declare the cursor
        declare ordernumbers cursor
        for
        select order_num from orders;
        
        -- Declare continue handler
        declare continue handler for sqlstate '02000' set done = 1; 
        
        -- Create a table to store the results
        create table if not exists ordertotals
            (order_num int, total decimal(8,2));
        
        -- Open the cursor
        open ordernumers;
        
        -- Loop through all rows
        repeat
            -- Get order number
            fetch ordernumers into o;
        
            -- Get the total for this order
            call ordertotal(o, 1, t);           // 存储过程ordertotal在前一章中创建的
            
            -- Insert order and total into ordertotals
            insert into ordertotals(order_num, total)
            values(o, t);
            
        -- End of loop
        until done end repeat;
        
        -- Close the cursor
        close ordernumers;
       end;
mysql> select * from ordertotals;
```
- sqlstate '02000'是一个未找到条件，当repeat由于没有更多的行供循环而不能继续时，出现这个条件。
- declare语句的发布存在特定的次序。用declare语句定义的局部变量必须在定义任意游标或句柄之前定义，而句柄必须在游标之后定义。不遵守此顺序将产生错误消息。
- 除上面例子2使用的repeat语句外，MySQL还支持循环语句，它可用来重复执行代码，直到使用leave语句手动退出为止。通常repeat语句的语法使它更适合于对游标进行循环。


# 二十五、使用触发器

1、对触发器的支持是在MySQL 5中增加的。

触发器：需要在某个表发生更改时自动处理。

触发器是MySQL响应以下任意语句而自动执行的一条MySQL语句(或位于begin和end语句之间的一组语句)：
- delete；
- insert；
- update。
其他MySQL语句不支持触发器。

2、创建触发器

在创建触发器时，需要给出4条信息：
- 唯一的触发器名；
- 触发器关联的表；
- 触发器应该响应的活动(delete、insert或update)；
- 触发器何时执行(处理之前或之后)。

在MySQL 5中，触发器名必须在每个表中唯一，但不是在每个数据库中唯一。
这表示同一数据库中的两个表可具有相同名字的触发器。这在其他每个数据库触发器名必须唯一的DBMS中不允许的，而且以后的MySQL版本很可能会使命名规则更为严格。因此，现在最好是在数据库范围内使用唯一的触发器名。



```
// create trigger用来创建名为newproduct的新触发器。
// 此触发器将在insert语句成功执行后执行。
// 这个触发器还指定for each row，因此代码对每个插入行执行。文本Product added将对每个插入的行显示一次。
// 使用insert语句添加一行或多行到products中，将看到对每个成功的插入，显示Product added消息。
mysql> create trigger newproduct after insert on products
       for each row select 'Product added';

mysql> 
```
- 触发器用create trigger语句创建。
- 触发器可在一个操作发送之前或之后执行。
- 只有表才支持触发器，视图不支持(临时表也不支持)。
- 触发器按每个表每个事件每次地定义，每个表每个事件每次只允许一个触发器。因此，每个表最多支持6个触发器(每条insert、update和delete的之前和之后)。
- 单一触发器不能与多个事件或多个表关联，所以，如果你需要一个对insert和update操作执行的触发器，则应该定义两个触发器。
- 如果before触发器失败，则MySQL将不执行请求的操作。此外，如果before触发器或语句本身失败，MySQL将不执行after触发器(如果有的话)。

3、删除触发器

```
mysql> drop trigger newproduct;
```
- 触发器不能更新或覆盖。为了修改一个触发器，必须先删除它，然后再重新创建。

4、使用触发器

insert触发器在insert语句执行之前或之后执行。需要知道以下几点：
- 在insert触发器代码内，可引用一个名为NEW的虚拟表，访问被插入的行；
- 在before insert触发器中，NEW中的值也可以被更新(允许更改被插入的值)；
- 对于auto_increment列，NEW在insert执行之前包含0，在insert执行之后包含新的自动生成值。
```
// 在插入一个新订单到orders表时，MySQL生成一个新订单号并保存到order_num中。触发器从NEW.order_num取得这个值并返回它。
// 此触发器必须按照after insert执行，因为在before insert语句执行之前，新order_num还没有生成。
// 对于orders的每次插入使用这个触发器将总是返回新的订单号。
mysql> create trigger neworder after insert on orders
       for each row select NEW.order_num;

mysql> insert into orders(order_date, cust_id) values(Now(), 10001);    // 执行后自动返回order_num
```
- 通常，将before用于数据验证和净化(目的是保证插入表中的数据确实是需要的数据)。本用途也适用于update触发器。


delete触发器在delete语句执行之前或之后执行。需要知道以下两点：
- 在delete触发器代码内，你可以引用一个名为OLD的虚拟表，访问被删除的行；
- OLD的值全都是只读的，不能更新。
```
// 在任意订单被删除前将执行此触发器。它使用一条insert语句将OLD中的值(要被删除的订单)保存到一个名为archive_orders的存档表中(为实际使用这个例子，你需要用与orders相同的列创建一个名为archive_orders的表)。
mysql> create trigger deleteorder before delete on orders
       for each row
       begin
        insert into archive_orders(order_num, order_date, cust_id)
        values(OLD.order_num, OLD.order_date, OLD.cust_id);
       end;
```
- 使用before delete触发器的优点(相对于after delete触发器来说)为，如果由于某种原因，订单不能存档，delete本身将被放弃。
- 触发器deleteorder使用begin和end语句标记触发器体。使用begin end块的好处是触发器能容纳多条SQL语句(在begin end块中一条挨着一条)。


update触发器在update语句执行之前或之后执行。需要知道以下几点：
- 在update触发器代码中，你可以引用一个名为OLD的虚拟表访问以前(update语句前)的值，引用一个名为NEW的虚拟表访问新更新的值；
- 在before update触发器中，NEW中的值可能也被更新(允许更改将要用于update语句中的值)；
- OLD中的值全都是只读的，不能更新。
```
// 每次更新一个行时，NEW.vend_state中的值(将用来更新表行的值)都用Upper(NEW.vend_state)替换。
mysql> create trigger updatevendor before update on vendors
       for each row set NEW.vend_state = Upper(NEW.vend_state);
```
- 任何数据净化都需要在update语句之前进行。


5、一些使用触发器时需要记住的重点。
- 与其他DBMS相比，MySQL 5中支持的触发器相当初级。未来的MySQL版本中有一些改进和增强触发器支持的计划。
- 创建触发器可能需要特殊的安全访问权限，但是，触发器的执行是自动的。如果insert、update或delete语句能够执行，则相关的触发器也能执行。
- 应该用触发器来保证数据的一致性(大小写、格式等)。在触发器中执行这种类型的处理的优点是它总是进行这种处理，而且是透明地进行，与客户机应该无关。
- 触发器的一种非常有意义的使用是创建审计跟踪。使用触发器，把更改(如果需要，甚至还有之前和之后的状态)记录到另一个表非常容易。
- 遗憾的是，MySQL触发器中不支持CALL语句。这表示不能从触发器内调用存储过程。所需的存储过程代码需要复制到触发器内。


# 二十六、管理事务处理

1、MySQL支持几种基本的数据库引擎。并非所有引擎都支持明确的事务处理管理。MyISAM和InnoDB是两种最常使用的引擎。前者不支持明确的事务处理管理，而后者支持。

事务处理可以用来维护数据库的完整性，它保证成批的MySQL操作要么完全执行，要么完全不执行。

事务处理是一种机制，用来管理必须成批执行的MySQL操作，以保证数据库不包括不完整的操作结果。
利用事务处理，可以保证一组操作不会中途停止，它们或者作为整体执行，或者完全不执行(除非明确指示)。如果没有错误发生，整组语句提交给(写到)数据库表。如果发生错误，则进行回退(撤销)以恢复数据库到某个已知且安全的状态。


关于事务处理的几个术语：
- 事务指一组SQL语句；
- 回退指撤销指定SQL语句的过程；
- 提交指将未存储的SQL语句结果写入数据库表；
- 保留点指事务处理中设置的临时占位符，你可以对它发布回退(与回退整个事务处理不同)。


2、控制事务处理

管理事务处理的关键在于将SQL语句组分解为逻辑块，并明确规定数据何时应该回退，何时不应该回退。

```
mysql> select * from ordertotals;
       start transaction;           // 标识事务的开始
       delete from ordertotals;
       select * from ordertotals;
       rollback;                    // 回退start transaction之后的所有语句
       select * from ordertotals;
       
```
- rollback只能在一个事务处理内使用(在执行一条start transaction命令之后)。
- 事务处理用来管理insert、update和delete语句。你不能回退select语句。你不能回退create或drop语句。事务处理块中可以使用这两条语句，但如果你执行回退，它们不会被撤销。


一般的MySQL语句都是直接针对数据库表执行和编写的。这就是所谓的隐含提交，即提交(写或保存)操作是自动进行的。
但是，在事务处理块中，提交不会隐含地进行。为进行明确的提交，使用commit语句。
```
start transaction;
delete from orderitems where order_num = 20010;
delete from orders where order_num = 20010;
commit; // 仅在不出错时写出更改。如果第一条delete起作用，但第二条失败，则delete不会提交(实际上，它是被自动撤销的)。
```
- 当commit或rollback语句执行后，事务会自动关闭(将来的更改会隐含提交)。


为了支持回退部分事务处理，必须能在事务处理块中合适的位置放置占位符。这样，如果需要回退，可以回退到某个占位符。这样的占位符称为保留点。
```
savepoint deletel;      // 创建占位符
rollback to deletel;    // 回退到上面给出的保留点
```
- 每个保留点都取标识它的唯一名字，以便在回退时，MySQL知道要回退到何处。
- 可以在MySQL代码中设置任意多的保留点，越多越好，便于灵活地进行回退。
- 保留点在事务处理完成(执行一条rollback或commit)后自动释放。自MySQL 5以来，也可以用release savepoint明确地释放保留点。

3、更改默认的提交行为

默认的MySQL行为是自动提交所有更改。
为指示MySQL不自动提交更改，需要使用以下语句：
```
set autocommit=0;
```
- autocommit标志决定是否自动提交更改，不管有没有commit语句。
- 设置autocommit为0(假)指示MySQL不自动提交更改(直到autocommit被设置为真为止)。
- autocommit标志是针对每个连接而不是服务器的。


# 二十七、全球化和本地化

1、不同的语言和字符集需要以不同的方式存储和检索。

几个术语：
- 字符集为字母和符合的集合；
- 编码为某个字符集成员的内部表示；
- 校对为规定字符如何比较的指令。
```
mysql> show character set;      // 显示所有可用的字符集以及每个字符集的描述和默认校对

mysql> show collation;          // 显示所有可用的校对以及它们适用的字符集。

mysql> show variables like 'character%';    // 确定所用的字符集

mysql> show variables like 'collation%';    // 确定所用的校对

mysql> create table mytable
       (
        columnn1 int,
        columnn2 varchar(10)
       ) default character set hebrew
         collate hebrew_general_ci;
         
mysql> create table mytable
       (
        columnn1 int,
        columnn2 varchar(10),
        column3  varchar(10) character set latin1 collate latin1_general_ci
       ) default character set hebrew
         collate hebrew_general_ci;
         
mysql> select * from customers
       order by lastname, firstname collate latin1_general_cs;  // 指定校对顺序排序
```
- 
- 有的字符集具有不止一种校对。例如，latin1对不同的欧洲语言有几种校对，而且许多校对出现两次，一次区分大小写(由_cs表示)，一次不区分大小写(由_ci表示)。
- 通常系统管理在安装时定义一个默认的字符集和校对。此外，也可以在创建数据库时，指定默认的字符集和校对。
- 不同的表，甚至不同的列都可能需要不同的字符集，而且两者都可以在创建表时指定。
- 校对在对用order by子句检索出来的数据排序时起重要的作用。
- 除了在order by子句中使用以外，collcate还可以用于group by、having、聚集函数、别名等。
- 如果绝对需要，串可以在字符集之间进行转换。为此，使用Cast()或Convert()函数。

一般，MySQL如下确定使用什么样的字符集和校对:
- 如果指定character set和collation两者，则使用这些值；
- 如果只指定character set，则使用此字符集及其默认的校对(如show character set的结果中所示)。
- 如果既不指定character set，也不指定collation，则使用数据库默认。


# 二十八、安全管理 

1、MySQL Administrator提供了一个图形用户界面，可用来管理用户及账号权限。


2、MySQL用户账号和信息存储在名为mysql的MySQL数据库中。

```
mysql> user mysql;
mysql> select user from user;       、// 获得所有用户账号列表
```

3、
```
mysql> create user ben identified by 'p@$$w0rd';    // 创建一个新用户账号

mysql> rename user ben to bforta;   // 重命名一个用户账号

mysql> drop user bforta;    // 删除一个用户账号(以及相关的权限)

mysql> show grants for bforta;  // 显示用户账号的权限
```
- identified by指定的口令为纯文本，MySQL将在保存到user表之前对其进行加密。为了作为散列值指定口令，使用identified by password。
- grant语句也可以创建用户账号，但一般来说create user是最清楚和最简单的句子。
- 仅MySQL 5或之后的版本支持rename user。为了在以前的MySQL中重命名一个用户，可使用update直接更新user表。
- 自MySQL 5以来，drop user删除用户账号和所有相关的账号权限。在MySQL 5以前，drop user只能用来删除用户账号，不能删除相关的权限。因此，如果使用旧版本的MySQL，需要先用revoke删除与账号相关的权限，然后再用drop user删除账号。
- 在创建用户账号后，必须接着分配访问权限。新创建的用户账号没有访问权限。它们能登录MySQL，但不能看到数据，不能执行任何数据库操作。
- usage表示根本没有权限，所以，此结果表示在任意数据库和任意表上对任何东西没有权限。
- 用户定义为user@host：MySQL的权限用用户名和主机名结合定义。如果不指定主机名，则使用默认的主机名%(授予用户访问权限而不管主机名)。


为设置权限，使用grant语句。grant要求你至少给出以下信息：
- 要授予的权限；
- 被授予访问权限的数据库或表；
- 用户名。
```
mysql> grant select on crashcourse.* to bforta; // 允许用户在crashcourse.*(crashcourse数据库的所有表)上使用select、

mysql> revoke select on crashcourse.* from bforta;
```
- 每个grant添加(或更新)用户一个权限。MySQL读取所有授权，并根据它们确定权限。
- grant的反操作为revoke，用它来撤销特定的权限。
- 被撤销的访问权限必须存在，否则会出错。

grant和revoke可在几个层次上控制访问权限：
- 整个服务器，使用grant all和revoke all；
- 整个数据库，使用on database.*；
- 特定的表，使用on database.table；
- 特定的列；
- 特定的存储过程。

4、可以授予或撤销的每个权限

权限 | 说明
---|---
all | 除grant option外的所有权限
alter | 使用alter table
alter routine | 使用alter procedure和drop procedure
create | 使用create table
create routine | 使用create procedure
create temporary tables | 使用create temporary table
create user | 使用create user、drop user、rename user和revoke all privileges
create view | 使用create view
delete | 使用delete
drop | 使用drop table
execute | 使用call和存储过程
file | 使用select into outfile和load data infile
grant option | 使用grant和revoke
index | 使用create index和drop index
insert | 使用insert
lock tables | 使用lock tables
process | 使用show full processlist
reload | 使用flush
replication client | 服务器位置的访问
replication slave | 由复制从属使用
select | 使用select
show databases | 使用show databases
show view | 使用show create view
shutdown | 使用mysqladmin shudown(用来关闭MySQL)
super | 使用change master、kill、logs、purge、master和set global。还允许mysqladmin调试登录
update | 使用update
usage | 无访问权限

 
- 在使用grant和revoke时，用户账号必须存在，但对所涉及的对象没有这个要求。这允许管理员在创建数据库和表之前设计和实现安全措施。这样做的副作用是，当某个数据库或表被删除时(用drop语句)，相关的访问权限仍然存在。而且，如果将来重新创建数据库或表，这些权限仍然起作用。
- 可通过列出各权限并用逗号分隔，将多条grant语句串在一起，如：grant select, insert on crashcourse.* to bforta; 

5、更改口令
```
mysql> set password for bforta = password('n3w p@$$w0rd');

mysql> set password = password('n3w p@$$w0rd');
```
- set password更新用户口令。新口令必须传递到password()函数进行加密。
- set password还可以用来设置自己的口令。在不指定用户名时，set password更新当前登录用户的口令。


# 二十九、数据库维护

1、备份数据

- 使用命令行实用程序mysqldump转储所有数据库内容到某个外部文件。在进行常规备份前这个实用程序应该正常运行，以便能正常地备份转储文件。
- 可用命令行实用程序mysqlhotcopy从一个数据库复制所有数据(并非所有数据库引擎都支持这个实用程序)。
- 可以用MySQL的backup table或select into outfile转储所有数据到某个外部文件。这两条语句都接受将要创建的系统文件名，此系统文件必须不存在，否则会出错。数据可以用restore table来复原。

为了保证所有数据被写到磁盘(包括索引数据)，可能需要在进行备份前使用flush tables语句。


2、进行数据库维护
```
mysql> analyze table orders;

mysql> check table orders, orderitems;
```
- analyze table，用来检查表键是否正确。
- check table用来针对许多问题对表进行检查。在MyISAM表上还对索引进行检查。check table支持一系列的用于MyISAM表的方式。
- changed检查自最后一次检查以来改动过的表。
- extended执行最彻底的检查，fast只检查未正常关闭的表，medium检查所有被删除的链接并进行键检验，quick只进行快速扫描。
- 如果MyISAM表访问产生不正确和不一致的结果，可能需要用repair table来修复相应的表。这条语句不应该经常使用，如果需要经常使用，可能会有更大的问题要解决。
- 如果从一个表中删除大量数据，应该使用optimize table来收回所用的空间，从而优化表的性能。


3、诊断启动问题

在排序系统启动问题时，首先应该尽量用手动启动服务器。
MySQL服务器自身通过在命令行上执行mysqld启动。
下面是几个重要的mysqld命令行选项：
- --help显示帮助——一个选项列表；
- --safe-mode装载减去某些最佳配置的服务器；
- --verbose显示全文本消息(为获得更详细的帮助信息与--help联合使用)。
- --version显示版本信息然后退出。

4、查看日志文件

主要的日志文件有以下几种：
- 错误日志。它包含启动和关闭问题以及任意关键错误的细节。此日志通常名为hostname.err，位于data目录中。此日志名可用--log-error命令行选项更改。
- 查询日志。它记录所有MySQL活动，在诊断问题时非常有用。此日志文件可能会很快地变得非常大，因此不应该长期使用它。此日志通常名为hostname.log，位于data目录中。此日志名可用--log命令行选项更改。
- 二进制日志。它记录更新过数据(或者可能更新过数据)的所有语句。此日志通常名为hostname.bin，位于data目录中。此名字可用--log-bin命令行选项更改。注意，这个日志文件是MySQL 5中添加的，以前的MySQL版本中使用的是更新日志。
- 缓慢查询日志。顾名思义，此日志记录执行缓慢的任何查询。这个日志在确定数据库何处需要优化很有用。此日志通常名为hostname-slow.log，位于data目录中。此名字可用--log-slow-queries命令行选项更改。

在使用日志时，可用flush logs语句来刷新和重新开始所有日志文件。


# 三十、改善性能

1、回顾一下前面各章的重点，提供进行性能优化探讨和分析的一个出发点。
- 首先， MySQL（与所有DBMS一样）具有特定的硬件建议。在学习和研究MySQL时，使用任何旧的计算机作为服务器都可以。但对用于生产的服务器来说，应该坚持遵循这些硬件建议。
- 一般来说，关键的生产DBMS应该运行在自己的专用服务器上。
- MySQL是用一系列的默认设置预先配置的，从这些设置开始通常是很好的。但过一段时间后你可能需要调整内存分配、缓冲区大小等。（为查看当前设置，可使用SHOW VARIABLES;和SHOW STATUS;。）
- MySQL一个多用户多线程的DBMS，换言之，它经常同时执行多个任务。如果这些任务中的某一个执行缓慢，则所有请求都会执行缓慢。如果你遇到显著的性能不良，可使用SHOW PROCESSLIST显示所有活动进程（以及它们的线程ID和执行时间）。你还可以用KILL命令终结某个特定的进程（使用这个命令需要作为管理员登录）。
- 总是有不止一种方法编写同一条SELECT语句。应该试验联结、并、子查询等，找出最佳的方法。
- 使用EXPLAIN语句让MySQL解释它将如何执行一条SELECT语句。
- 一般来说，存储过程执行得比一条一条地执行其中的各条MySQL语句快。
- 应该总是使用正确的数据类型。
- 决不要检索比需求还要多的数据。换言之，不要用SELECT *（除非你真正需要每个列）。
- 有的操作（包括INSERT）支持一个可选的DELAYED关键字，如果使用它，将把控制立即返回给调用程序，并且一旦有可能就实际执行该操作。
- 在导入数据时，应该关闭自动提交。你可能还想删除索引（包括FULLTEXT索引），然后在导入完成后再重建它们。
- 必须索引数据库表以改善数据检索的性能。确定索引什么不是一件微不足道的任务，需要分析使用的SELECT语句以找出重复的WHERE和ORDER BY子句。如果一个简单的WHERE子句返回结果所花的时间太长，则可以断定其中使用的列（或几个列）就是需要索引的对象。
- 你的SELECT语句中有一系列复杂的OR条件吗？通过使用多条SELECT语句和连接它们的UNION语句，你能看到极大的性能改进。
- 索引改善数据检索的性能，但损害数据插入、删除和更新的性能。如果你有一些表，它们收集数据且不经常被搜索，则在有必要之前不要索引它们。（索引可根据需要添加和删除。）
- LIKE很慢。一般来说，最好是使用FULLTEXT而不是LIKE。
- 数据库是不断变化的实体。一组优化良好的表一会儿后可能就面目全非了。由于表的使用和内容的更改，理想的优化和配置也会改变。
- 最重要的规则就是，每条规则在某些条件下都会被打破


# 三十一、附录：MySQL数据类型

1、串数据类型

数据类型 | 说明
---|---
char | 1~255个字符的定长串。它的长度必须在创建时指定，否则MySQL假定为char(1)
enum | 接受最多64K个串组成的一个预定义集合的某个串
longtext | 与text相同，但最大长度为4GB
mediumtext | 与text相同，但最大长度为16K
set | 接受最多64个串组成的一个预定义集合的零个或多个串
text | 最大长度为64K的变长文本
tinytext | 与text相同，但最大长度为255字节
varchar | 长度可变，最多不超过255字节。如果在创建时指定为varchar(n)，则可存储0到n个字符的变长串(其中n≤255)

不管使用何种形式的串数据类型，串值都必须括在引号内(通常单引号更好)。

2、数值数据类型

数据类型 | 说明
---|---
bit | 位字段，1~64位。(在MySQL 5之前，bit在功能上等价于tinyint)
bigint | 整数值，支持-9223372036854775808\~9223372036854775807(如果是unsigned，为0\~18446744073709551615)的数
boolean(或bool) | 布尔标志，或者为0或者为1，主要用于开/关(on/off)标志
decimal(或dec) | 精度可变的浮点值
double | 双精度浮点值
float | 单精度浮点值
int(或integer) | 整数值，支持-2147483648\~2147483647（如果是unsigned，为0\~4294967295）的数
mediumint | 整数值，支持-8388608\~8388607（如果是unsigned，为0\~16777215）的数
real | 4字节的浮点值
smallint | 整数值，支持-32768\~32767（如果是unsigned，为0\~65535）的数
tinyint | 整数值，支持-128\~127（如果是unsigned，为0\~255）的数

3、日期和时间数据类型

数据类型 | 说明
---|---
date | 表示1000-01-01~9999-12-31的日期，格式为YYYY-MM-DD
datatime | date和time的组合
timestamp | 功能和datetime相同(但范围较小)
time | 格式为HH:MM:SS
year | 用2位数字表示，范围是70(1970年)\~69(2069年)，用4位数字表示，范围是1901年\~2155年

4、二进制数据

数据类型 | 说明
---|---
blob | blob最大长度为64KB
mediumblob | blob最大长度为16MB
longblob | blob最大长度为4GB
tinyblob | blob最大长度为255字节
