---
layout:     post
title:      "SQL注入常见的类型及原理"
subtitle:   "SQL Injection——常年登顶OWASP TOP10的漏洞"
date:       2020-03-14 14:50:00
author:     "Lindada"
header-img: "img/post-bg-rwd.jpg"
catalog: true
tags:
    - SQL注入
    - Web安全
---

>本文介绍了一些测试网站是否存在SQL注入漏洞的技巧，总结了几种SQL注入方式，并介绍了一些简单的绕过方式。文中没有插图片，因为主要是自己的学习总结，以写作方便和自己能看懂为主。

### SQL注入介绍
#### 漏洞原理
SQL注入是指Web应用对用户输入的数据过滤不严，使得从前端传入后端的参数是攻击者可控的，并且参数带进了数据库查询。攻击者可以利用SQL注入漏洞，通过构造不同的payload来实现对数据库的任意操作。

存在SQL注入的两个前提：
- 参数是用户可控的
- 参数会被带进数据库查询

以常用的SQL查询语句为例：
`select * from user where id = '$_GET['id']'`
在这个查询语句中，`$_GET['id']`是PHP通过GET请求接收的用户所输入的数据，被带到SQL数据库进行查询，当id为不同的值时，服务器将返回不同的内容。
如果用户输入的是一段SQL语句，则这段语句也将会被带进来查询，从而实现了SQL注入。比如用户输入的id数据为：`1' or 1=1# '`，查询语句将变为：
`select * from user where id = '1' or 1=1# ''`
该语句将返回所有的查询结果。
上面#号是SQL中的注释符，类似的还有`-- `，`/**/`

上述前提是id是**字符型**变量，如果是整型变量，则输入参数应为：`1 or 1=1# '` ，即不需要闭合前面的单引号。

以上都是一些SQL注入应该知道的基本概念，这里解释得不是很完整，不清楚的读者可以自行搜索资料了解。

#### 与注入相关的MySQL重要知识点
**(1) order by 语法**

order by 语句用于根据指定的列（比如第四列）对结果集进行排序，默认为升序。

如：
```
select * from user order by 1  -- 按照第一列升序进行排序，返回结果
select * from user order by 1 desc -- 按照第一列升序进行排序，返回结果
select * from user order by 1 asc  -- 按照第一列降序进行排序，返回结果
```  
在SQL注入测试中常用来测试表中含有的字段数：观察order by 1，order by 2，...，知道页面返回结果发生明显变化，则说明到达了临界点，即得出该表的字段数。

**(2)union 联合查询**

union 操作符用于合并两个或多个 select 语句的结果集，即，可以同时查询两个表，并返回两个结果。注意：这两个表的列数应该相同。

如：
`SELECT * FROM table_users where id = 1 UNION SELECT * FROM table_admins where id = 1`

**也可构造相同列数的表格用于联合查询**：比如知道users中有3列数据，分别是id，name，和password，我们也可以构造一个3列的数组用于联合查询：
```SELECT * FROM table_users where id = 1 UNION SELECT 1,2,3``` 。这里用select构造了一个（1,2,3）数组，这个技巧常用于测试页面中的**显示位**。

**(3)information_schema数据库**

在MySQL5.0版本以后，会默认存在一个'information_schema'的数据库，它记录了用户创建的所有数据库库名、表名、字段名。我们实现SQL的注入后，可以通过查询这个数据库中的表来获取这个主机上所有的数据库信息。

需要记住的三个表：
- SCHEMATA :  *存放库名*
- TABLES : *存放表名，及对应的库名*
- COLUMNS : *存放字段名，及对应的表名、库名*

**(4)几个重要的函数**

- database(): 当前网站使用的数据库
- version(): 当前MySQL版本
- user(): 当前MySQL用户

**(5)快速测试是否存在注入**

1. 多打一个单引号，如：`www.yoursite.com/test.php?id=1'`
2. 如上面语句发现网站返回页面不一样，则说明可能存在SQL注入，继续测试：`www.yoursite.com/test.php?id=1' and 1=1 #'`
3. 如返回结果正常，则说明存在SQL注入

**(6)常用测试套路**

1. 用order by确定列数
2. 用union确定网站显示位
	-	令第一条语句不成立，在第二条语句构造一个数组
3. 利用显示位显示查询结果
	-	查询当前database
	-	查询information_schema数据库
	-	可利用concat拼接查询结果

### 常见的几种注入方式
#### Union注入

因为Union语句可以实现两条语句的查询，我们可以在参数后面加上union语句，并使得前一条语句的结果为假，才能让union的语句的查询结果得到返回（一般网站只返回第一个查询结果）。如：`select * from users where id = '-1' union select 1,2,3` 由于id=-1在数据库中不存在，会返回select 1,2,3的结果，我们可以观察网站上显示出来的数字，判断users表中第几列是被显示出来的。**注：要构造数组时先要知道列数，可以通过order by爆破出来。**

比如我们登录网站时，右上角经常会显示用户的用户名，我们希望知道这个用户名信息到底存在数据库表中的第几列，如果知道了，我们将该列要显示的信息替换成我们想知道的内容，即可达到探测信息的效果。在上述例子中，name处于第2列，由于union注入的存在，我们将用户名的内容改成了“2”，因此便知道，网站上显示的是第2个字段（name）。

知道了显示位是第2位，我们可以在构造数组的时候，对第2位进行特别关照。假如我们想知道当前用的是哪个数据库，则可以将union语句构造为：
`union select 1, database() ,3`

还可以进一步打探消息：
```
-- 返回sql数据库中的第一个表
union select 1, (select table_name from information_schema.tables where table_schema='sql' limit 0,1;) ,3

-- 返回sql数据库中的所有表（拼接成一行返回）
union select 1, concat((select table_name from information_schema.tables where table_schema='sql';)) ,3
```

以上是构建的union语句，使用时需要在前面接上参数，如`id=1 union ...`

**总结：**

利用union注入，我们可以获取目标网站数据库中的重要信息，包括数据库名、表名、字段、甚至内容。操作步骤为：
1. 利用order by爆破出列数
2. 利用union注入试出显示位
3. 构造union语句，使显示位展示你想要的信息

这需要满足以下几点条件：
- 首先是存在注入漏洞，即对方过滤不严
- 存在显示位，即能让我们把信息显示出来
- 目标MySQL数据库在5.0以上

#### 布尔注入
布尔注入，顾名思义对应的情况是 YES/NO ，即网站不返回具体的信息，只返回是与否。比如说网站提供的服务是查询某个用户名是否被注册，则它返回的信息只会是已注册和未被注册。这时候我们不能使用union注入，因为它没有返回具体信息，即没有显示位。

在这种情况，我们可以利用布尔注入来逐步探测出数据库中的信息。打个比方就是，对方只会回答是/否，我们通过不断地问对方问题，来获取关于对方的信息。如：你的年龄是在10-20之间吗？你喜欢的颜色是黄色吗？

布尔注入的payload构造例子如下，id=
`1' and length(database())>=1--+` 其中，+是空格，id=1是你已经知道的回答为yes的案例。

知道当前数据库名长度之和，我们就可以猜啦，假如是3位，每一位有26个字母，则最多试78次就可以知道库名**（可用Burpsuite爆破）**：id=
`1' and substr(database(),1,1)='a' --+`

**总结**

布尔注入用在网站响应为YES/NO的时候，通过不断地“问”网站问题达到信息嗅探的目的，步骤：
- 直接构造payload，一般要用到爆破

需要满足条件：
- 存在布尔注入漏洞，即对方过滤不严
- 语句运行正确与否，网站上要有看得出来的区别（数据包有区别也可以）

#### 报错注入

报错注入主要利用网站执行SQL出错会显示出来的漏洞进行注入：为了调试方便，程序员可能会将SQL语句执行出错时产生的报错信息返回到网页前端，如果我们能在这些报错中“加入”我们想要知道的敏感信息，则达到注入的效果。但是在实际中，很少会出现直接把SQL报错信息直接返回前端的场景。

常被利用在报错注入的函数有：
- updatexml()
- floor()
- extractvalue()
- exp()

下面只介绍updatexml()，因为我只用过这个...其他的应该大同小异
```
UPDATEXML (XML_document, XPath_string, new_value); 
第一个参数：XML_document是String格式，为XML文档对象的名称 
第二个参数：XPath_string (Xpath格式的字符串) ，如果不了解Xpath语法，可以在网上查找教程。 
第三个参数：new_value，String格式，替换查找到的符合条件的数据 
```
应该是将XML_document中的XPath_string替换成new_value的意思？

构造payload：id=
`1 and updatexml(1,concat(0x7e,(select user()),0x7e),1)--+`

其中，0x7e是波浪号~，因为以~~包裹的信息不符合xml语法，因此会报错，报错的时候顺带把user()的内容告诉给我们了，所以我们就知道了当前MySQL的用户名。类似union注入，我们可以继续探测出更多的信息：库名、存在的所有表名、字段名等等。

**总结**

报错注入利用了网站会显示SQL错误信息的漏洞，构造一个payload，使得报错信息中包含重要信息，步骤：
- 直接构造payload

需要满足条件：
- 存在报错注入漏洞，即对方过滤不严
- SQL错误信息要回显在网页上

#### 时间盲注

时间盲注跟布尔注入类似，每次只能知道答案“是”或“否”，不同点是布尔注入会在网页上显示结果，当网页上连“是”或“否”都不告诉你的时候，时间盲注是一个不错的选择。时间盲注在payload中加入延时函数sleep()，为语句执行结果为true和false设置不同的延时，根据页面的响应状况可以判断执行结果是true还是false。

时间注入用到了一个关键的函数：if(expr1,expr2,expr3)。在SQL中if可以看做一个三目运算符，当expr1语句为true时，返回expr2，否则返回expr3.因此可以构建payload：id=
`1' and if((length(database())>1,sleep(5),1) --+`
当数据库长度大于1时，页面响应时间大概有5秒，而小于1时，很快就能返回页面了。根据返回时间的不同，可以获取我们想要知道的信息。其他操作思路与布尔注入类似。

**注：BurpSuite中可以查看页面响应时间**

**总结**

时间盲注应用场景更广泛，它不需要页面显示任何内容，通过观察页面的响应时间来判断“是”或“否”，步骤：
- 直接构造payload，一般要用到爆破

需要满足条件：
- 存在时间注入漏洞，即对方过滤不严（一般服务器会过滤sleep函数）

#### 宽字节注入

一般我们可以利用union注入进行信息获取，但当服务器对我们的单引号进行转义之后（' -> \'），我们一般是没有办法进行SQL注入的。但有一个特例：当该数据库使用GBK编码的时候，可以使用宽字节注入。

如果该数据库使用了GBK编码，因为反斜杠\的编码是%5c，我们可以在反斜杠\前面加上一个%df，这样%df%5c会构成繁体字的“連”，从而达到转义。构造payload：id=
`1%df' and 1=1%23` 其中%23是注释符#的编码

其他的操作可以参考union注入，比如爆破列数、获取当前database，当前user等，但如果想利用information_schema获取其他信息的时候，就不能像union注入一样：id=
```
1%df' and union select 1, (select table_name from information_schema.tables where table_schema='test' limit 0,1;) ,3
``` 
因为test的单引号会被转义成\' ，这时候可以采取嵌套查询：

```
select table_name from information_schema.tables where table_schema=(select database()) limit 0,1;
```
其中database()就是当前的数据库test。如果想查询其他表名，可用limit一个个试：


查询字段名
```
select column_name from information_schema.column where table_schema=(select database()) and table_name=
(select table_name from information_schema.tables where table_schema=(select database()) limit 0,1;)
limit 0,1
```

**总结**

宽字节注入适用于有显示信息的页面，但是单引号'被转义了的情况。步骤
- 跟union注入类似，但要在‘前面加上%df

需要满足条件：
- 存在漏洞，即服务端过滤不严
- 数据库是GBK编码


#### base64注入

base64注入是针对传递的参数被base64加密后的注入点进行注入。除了数据被加密以外，其中注入方式与常规注入一般无二。
即参数是经过base64加密传输的，一般可能出现在cookie中，我们可以通过抓包，获取加密后的cookie，然后base64解密，加上我们的注入语句，再base64加密，最后替换原来的cookie值发送给服务器，以实现注入。

具体可参考：[SQL注入之BASE64变形注入](https://blog.csdn.net/qq_35569814/article/details/100274561)


#### 总结

总的来说，不同的SQL注入方法原理大同小异，都是通过在参数中携带注入语句，尝试让服务器带到数据库中查询。根据网站的情况不同：是否有信息显示（原始数据还是布尔数据）、是否有报错信息显示、是否存在加密传输等采取不同的注入方法。通常，服务器也会做一定的防御措施，比如过滤危险的函数名、对一些特殊符号进行转义等。我们最后稍微讨论一下几种简单的绕过方式。


### 绕过方式
#### 大小写绕过

服务器如果只是简单地过滤关键字，比如select、order等，我们可以尝试大小写混搭能否绕过，比如"OrdeR".

#### 双写绕过

难点在于判断服务器对哪个单词进行了过滤，如果通过不断的测试发现了的话，双写该单词即可，比如服务器对and进行了过滤，那么我们只需要将and携程aandnd即可。

#### 编码绕过

比如对payload进行2次URL全编码，因为服务器会自动对URL进行一次解码

#### 内联注释绕过

用`/!*  */`包裹关键字

更多绕过方式可参考：[SQL注入绕过技巧](https://www.cnblogs.com/Vinson404/p/7253255.html)


### 更多阅读
- [Web安全攻防：渗透测试实战指南](https://book.douban.com/subject/30280378/)
- [SQL注入之BASE64变形注入](https://blog.csdn.net/qq_35569814/article/details/100274561)
- [SQL注入绕过技巧](https://www.cnblogs.com/Vinson404/p/7253255.html)













































