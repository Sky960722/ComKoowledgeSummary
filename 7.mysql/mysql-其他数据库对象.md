# SQL之其他数据库对象

## 常见的数据库对象

| 对象                | 描述                                                         |
| ------------------- | ------------------------------------------------------------ |
| 表(TABLE)           | 表是存储数据的逻辑单元，以行和列的形式存在，列就是子u但，行就是记录 |
| 数据字典            | 就是系统表，存放数据库相关信息的表。系统表的数据通常由数据库系统维护，程序员通常不应该修改，只可查看 |
| 约束(CONSTRAINT)    | 执行数据校验的规则，用于保证数据完整性的规则                 |
| 视图(VIEW)          | 一个或者多个数据表里的数据的逻辑显示，视图并不存储数据       |
| 索引(INDEX)         | 用于提高查询性能，相当于书的目录                             |
| 存储过程(PROCEDURE) | 用于完成一次完整的业务处理，没有返回值，但可通过传出参数将多个值传给调用环境 |
| 存储函数(FUNCTION)  | 用于完成一次特定的计算，具有一个返回值                       |
| 触发器(TRIGGER)     | 相当于一个事件监听器，当数据库发生特定事件后，触发器被触发，完成相应的处理 |

## 视图

### 视图的理解

- 视图是一种虚拟表，本身是不具有数据的，占用很少的内存空间，它是SQL中的一个重要概念

- 视图建立在已有表的基础上，视图赖以建立的这些表称为基表

- 视图的创建和删除只影响视图本身，不影响对应的基表。但是当对视图中的数据进行增加、删除和修改操作时，数据表中的数据会相应地发生变化，反之亦然

- 向视图提供数据内容的语句为SELECT语句，可以将视图理解为存储起来的SELECT语句

  - 在数据库中，视图不会保存数据，数据真正保存在数据表中。当对视图中的数据进行增加、删除和修改操作时，数据表中的数据会相应地发生变化，反之亦然

- 视图，是向用户提供基表数据的另一种表现形式。通常情况下，小型项目的数据库可以不使用视图，但是在大型项目中，以及数据表比较复杂的情况下，视图的价值就凸显出来了，它可以帮助我们把经常查询的结果集放到虚拟表中，提升使用效率。理解和使用起来都非常方便。

### 创建视图

~~~SQL
CREATE [OR REPLACE]
[ALGORITHM = {UNDEFINED | MERGE | TEMPTABLE}]
VIEW 视图名称 [(字段列表)]
AS 查询语句
[WITH [CASCADED | LOCAL] CHECK OPTION]

##ALGORITHM:可选的ALGORITHM子句是对标准SQL的MySQL扩展
##可取三个值：MERGE、TEMPTABLE或UNDEFINED。如果没有ALGORITHM子句，默认算法是UNDEFINED(未定义的)。算法会影响MySQL处理视图的方式。
##对于MERGE，会将引用视图的语句的文本与视图定义合并起来，使得视图定义的某一部分取代语句的对应部分。
##对于TEMPTABLE，视图的结果将被置于临时表中，然后使用它执行语句。
##对于UNDEFINED，MySQL将选择所要使用的算法。如果可能，它倾向于MERGE而不是TEMPTABLE，这是因为MERGE通常更有效，而且如果使用了临时表，视图是不可更新的。
##明确选择TEMPTABLE的1个原因在于，创建临时表之后、并在完成语句处理之前，能够释放基表上的锁定。与MERGE算法相比，锁定释放的速度更快，这样，使用视图的其他客户端不会被屏蔽过长时间。
##CASCADED ：级联，满足与该视图有关的的所有相关视图和表的条件
##LOCAL : 可选，满足该视图本身定义即可

##精简版
CREATE VIEW 视图名称
AS 查询语句
~~~

### 查看视图

~~~SQL
##查看你数据库的表对象、视图对象
SHOW TABLES;

##查看视图的结构
DESC / DESCRIBE 视图名称;

##查看视图信息(显示数据表的存储引擎、版本、数据行数和数据大小等)
SHOW TABLES STATUS LIKE '视图名称';

##查看视图的详细定义信息
SHOW CREATE VIEW 视图名称;
~~~

### 更新视图的数据

- 一般情况

  - MySQL支持使用INSERT、UPDATE和DELETE语句对视图中的数据进行插入、更新和删除操作。当视图中的数据发生变化时，数据表中的数据也会发生变化，反之亦然

- 不可更新的视图

  - 要使视图可更新，视图中的行和底层基本表中的行之间必须存在一对一的关系。另外当视图定义出现如下情况时，视图不支持更新操作：
    - 在定义视图的时候指定了"ALGORITHM = TEMPTABLE"，视图将不支持INSERT和DELETE操作
    - 视图中不包含基表中所有被定义为非空又未指定默认值的列，视图将不支持INSERT操作
    - 在定义视图的SELECT语句中使用了JOIN联合查询，视图将不支持INSERT和DELETE操作
    - 在定义视图的SELECT语句后的字段列表中使用了数学表达式或子查询，视图将不支持INSERT，也不支持UPDATE使用了数学表达式，子查询的字段值
    - 在定义视图的SELECT语句后的字段列表中使用DISTINCT、聚合函数、GROUP BY、HAVING、UNION等，视图将不支持INSERT、UPDATE、DELETE；
    - 在定义视图的SELECT语句中包含了子查询，而子查询中引用了FROM后面的表，视图将不支持INSERT，UPDATE，DELETE；
    - 视图定义基于一个不可更新视图
    - 常量视图

### 修改视图

- 方法一：CREATE OR REPLACE VIEW
- 方法二：

~~~SQL
ALTER VIEW 视图名称
AS
查询语句
~~~

### 删除视图

- 删除视图只是删除视图的定义，并不会删除基表的数据
- 删除视图的语法是：

~~~SQL
DROP VIEW IF EXISTS 视图名称;
~~~

### 总结

- 视图优点
  - 操作简单
  - 减少数据冗余
  - 数据安全
  - 适应灵活多变的需求
  - 能够分解复杂的查询逻辑
- 视图缺点
  - 如果在实际数据表的基础上创建了视图，那么，如果实际数据表的结构变更了，我们就需要及时对相关的视图进行相应的维护。特别是嵌套的视图，维护会变得比较复杂

## 存储过程与函数

### 存储过程

- 含义：英文 Stored Procedure。是一组经过预先编译的SQL语句的封装
- 执行过程：存储过程预先存储在MySQL服务器上，需要执行的时候，客户端只需要向服务器端发出调用存储过程的命令，服务器端就可以把预先存储好的这一系列SQL语句全部执行
- 好处：
  - 简化操作，提高了sql语句的重用性，减少了开发程序员的压力
  - 减少操作过程中的失误，提高效率
  - 减少网络传输量（客户端不需要把所有的SQL语句通过网络发给服务器）
  - 减少了SQL语句暴露在网络上的风险，也提高了数据查询的安全性
- 差异：
  - 一旦存储过程被创建出来，使用它就像使用函数一样简单，我们直接通过调用存储过程名即可。相较于函数，存储过程是没有返回值的
- 存储过程的参数
  - 没有参数
  - 只有 IN 参数（有参数无返回）
  - 只有OUT类型（无参数有返回）
  - 即带IN又带OUT（有参数有返回）
  - 带INOUT（有参数有返回）
  - 注意：IN、OUT、INOUT都可以在一个存储过程中带多个
- 创建存储过程

~~~SQL
CREATE PROCEDURE 存储过程名(IN | OUT | INOUT 参数名 参数类型,...) 
[characteristics ...]
BEGIN
   存储过程体

END
~~~

- 说明

  1. 参数前面的符号的意思
     - IN：当前参数为输入参数，也就是表示入参
       - 存储过程知识读取这个参数的值。如果没有定义参数种类，默认就是 IN，表示输入参数
     - OUT：当前参数为输出参数，也就是表示出参
       - 执行完成之后，调用这个存储过程的客户端或者应用程序就可以读取这个参数返回的值了
     - INOUT：当前参数可以为输入参数，也可以为输出参数
  2. 形参类型可以是MySQL数据库中的任意类型
  3. characteristics 表示创建存储过程时指定的对存储过程的约束条件，其取值信息如下：

  ~~~SQL
  LANGUAGE SQL
  | [NOT] DETERMINISTIC
  | ( CONTAINS SQL | NO SQL | READS SQL DATA | MODIFIES SQL DATA)
  | SQL SECURITY ( DEFINER | INVOKER )
  | COMMENT 'string'
  ~~~

  - LANGUAGE SQL：说明存储过程执行是由SQL语句组成的，当前系统支持的语言为SQL
  - [NOT] DETERMINISTIC：指明存储过程执行的结果是否确定
    - DETERMINISTIC表示结果是确定的
    - NOT DETERMINISTIC表示结果是不确定的，默认为NOT DETERMINISTIC
  - (CONTAINS SQL | NO SQL | READS SQL DATA | MODIFIES SQL DATA)：指明子程序使用SQL语句的限制
    - CONTAINS SQL表示当前存储过程中的子程序包含SQL语句，但是并不包含读写数据的SQL语句
    - NO SQL表示当前存储过程的子程序中不包含任何SQL语句
    - READS SQL DATA表示当前存储过程的子程序中包含写数据的SQL语句
    - MODIFIES SQL DATA表示当前存储过程中的子程序中包含写数据的SQL语句
  - SQL SECURITY { DEFINER | INVOKER }：执行当前存储过程的权限，即指明哪些用户能够执行当前存储过程
    - DEFINER：表示只有当前存储过程的创建者或者定义这才能执行当前存储过程
    - INVOKER：表示拥有当前存储过程的访问权限的用户能够执行当前存储过程
  - COMMENT：表示注释信息
  
  4. 存储过程中可以有多条SQL语句，如果仅仅一条SQL语句，则可以省略BEGIN和END
  5. 需要设置新的结束标记
  
  ~~~SQL
  DELIMITER 新的结束标记
  ~~~
  
  因为MySQL默认的语句结束符号为分号';'。为了避免与存储过程中SQL语句结束符相冲突，需要使用DELIMITER改变存储过程的结束符
  
  示例：
  
  ~~~SQL
  DELIMITER $
  
  CREATE PROCEDURE 存储过程名(IN | OUT | INOUT 参数名 参数类型,...) 
  [characteristics ...]
  BEGIN
     sql语句1;
     sql语句2;
  END $
  ~~~
### 调用存储过程

- 调用格式

~~~SQL
CALL 存储过程名(实参列表)
~~~

1. 调用 IN 模式参数

~~~SQL
CALL sp1('值');
~~~

2. 调用 OUT 模式的参数

~~~SQL
SET @name;
CALL sp1(@name);
SELECT @name;
~~~

3. 调用 INOUT 模式的参数

~~~SQL
SET @name=值;
CALL sp1(@name);
SELECT @name
~~~

### 存储函数

- 用处：MySQL支持自定义函数，定义好之后，调用方式与调用MySQL预定义的系统函数一样

- 创建

~~~SQL
CREATE FUNCTION 函数名 (参数名 参数类型,...)
RETURNS 返回值类型
[characteristics...]
BEGIN
   函数体 #函数体中肯定有 RETURN 语句
END
~~~

- 说明

  1. 参数列表：指定参数为IN、OUT或INOUT只对PROCEDURE是合法的，FUNCTION中总是默认为IN参数

  2. RETURNS type 语句表示函数返回数据的类型

     RETURNS子句只能对FUNCTION做指定，对函数而言这是强制的。它用来指定函数的返回类型，而且函数体必须包含一个 RETURN value 语句

  3. characteristic 创建函数时指定的对函数的约束。取值与创建存储过程时相同

  4. 函数体也可以用BEGIN...END来表示SQL代码的开始和结束。如果函数体只有一条语句，也可以省略BEGIN...END

- 调用

~~~SQL
SELECT 函数名（实参列表）
~~~

### 存储过程与存储函数对比

|          | 关键字    | 调用语法        | 返回值            | 应用场景                       |
| -------- | --------- | --------------- | ----------------- | ------------------------------ |
| 存储过程 | PROCEDURE | CALL 存储过程() | 理解为有0个或多个 | 一般用于更新                   |
| 存储函数 | FUNCTION  | SELECT 函数()   | 只能是一个        | 一般用于查询结果为一个值并返回 |

- 总结
- 存储过程的功能更加强大，包括能够执行对表的操作（比如创建表，删除表等）和事务操作，这些功能是存储函数不具备的

### 存储过程和函数的查看、修改、删除

- 查看

  1. 使用SHOW CREATE语句查看存储过程和函数的创建信息

  ~~~SQL
  SHOW CREATE (PROCEDURE | FUNCTION) 存储过程名或函数名
  ~~~

  2. 使用 SHOW STATUS语句查看存储过程和函数的状态信息

  ~~~sql
  SHOW (PROCEDURE | FUNCTION) STATUS [LIKE 'pattern']
  ~~~

  3. 从information_schema.Routindes表中查看存储过程和函数的信息

  ~~~sql
  SELECT * FROM information_schema.Routines WHERE ROUTINE_NAME = '存储过程或函数的名' [AND ROUTINE_TYPE = { 'PROCEDURE | FUNCTION'}];
  ~~~

- 修改

~~~SQL
ALTER ( PROCEDURE | FRUNCTION ) 存储过程或函数的名 [characteristic...]

##characteristic 同上面存储过程
~~~

- 删除

~~~sql
DROP {PROCEDURE | FUNCTION} [IF EXISTS]
~~~

### 存储过程优点

1. 存储过程可以一次编译多次使用
2. 可以减少开发工具量
3. 存储过程的安全性强
4. 可以减少网络传输量
5. 良好的封装性

### 存储过程缺点

1. 可移植性差
2. 调试困难
3. 存储过程的版本管理很困难
4. 它不适合高并发的场景

## 变量、流程控制与游标

### 变量

1. MySQL数据库中，变量分为系统变量以及用户自定义变量

#### 系统变量

- 变量由系统定义，不是用户定义，属于服务器层面。这些系统变量的值要么是编译MySQL时参数的默认值，要么是配置文件（例如 my.ini 等）中的参数值。通过网址https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html查看MySQL文档的系统变量

##### 分类

- 全局系统变量（global 关键字）。静态变量（在 MySQL服务实例运行期间它们的值不能使用 set 动态修改）属于特殊的全局系统变量
- 会话系统变量（session 关键字），如果不写，默认会话级别
- 区别：每一个MySQL客户机成功连接MySQL服务器后，就会产生与之对应的会话。会话期间，MySQL服务实例会在MySQL服务器内存中生成与该会话对应的会话系统变量，这些会话系统变量的初始值是全局系统变量值的复制。

##### 查看系统变量

~~~sql
#查看所有全局变量
SHOW GLOBAL VARIABLES;

#查看所有会话变量
SHOW SESSION VARIABLES；
或
SHOW VARIABLES;

#查看满足条件的部分系统变量
SHOW GLOBAL VARIABLES LIKE '%标识符%';

#查看满足条件的部分会话变量
SHOW SESSION VARIABLES LIKE '%标识符%';

#查看指定系统变量
#作为MySQL编码规范，MySQL中的系统变量以两个"@"开头，其中"@@global"仅用于标记全局系统变量，"@@session"仅用于标记会话系统变量。"@@"首先标记会话系统变量，如果会话系统变量不存在，则标记全局系统变量
#查看指定的系统变量的值
SELECT @@global.变量名;

#查看指定的会话变量的值
SELECT @@session.变量名;
#或者
SELECT @@变量名;
~~~

##### 修改系统变量的值

- 方法1：修改MySQL 配置文件，继而修改MySQL系统变量的值
- 方法2：在MySQL服务运行期间，使用"set" 命令重新设置系统变量的值

~~~sql
SET @@global.变量名=变量值;
SET GLOBAL 变量名=变量值;

SET @@session.变量名=变量值;
SET SESSION 变量名=变量值;
~~~

#### 用户变量

##### 用户变量分类

- 用户变量是用户自己定义的，作为MySQL编码规范，MySQL中的用户变量以一个 "@"开头。根据作用范围不同，又分为 会话用户变量 和 局部变量
- 会话用户变量：作用域和会话变量一样，只对当前连接会话有效
- 局部变量：只在BEGIN 和 END 语句块中有效。局部变量只能在存储过程和函数中使用。

##### 会话用户变量

~~~SQL
# 方式1， "=" 或 ":="
SET @用户变量 = 值;
SET @用户变量 := 值;

#方式2： ":=" 或 INTO 关键字
SELECT @用户变量 := 表达式 [FROM 等子句];
SELECT 表达式 INTO @用户变量 [FROM 等字句];
~~~

- 查看用户变量的值（查看、比较、运算等）

~~~sql
SELECT @用户变量
~~~

##### 局部变量

- 定义：可以使用 DECLARE 语句定义一个局部变量
- 作用域：仅仅在定义它的 BEGIN ... END 中有效
- 位置：只能放在 BEGIN ... END 中，而且只能放在第一句

~~~sql
BEGIN
	#声明局部变量
	DECLARE 变量名1 变量数据类型 [DEFAULT 变量默认值];
	DECLARE 变量名2,变量名3，。。。 变量数据类型 [DEFAULT 变量默认值];
	
	##为局部变量赋值
	SET 变量名1 = 值
	SELECT 值 INTO 变量名2 [FROM 子句];
	
	##查看局部变量的值
	SELECT 变量1,变量2，变量3
END
~~~

##### 会话用户变量与局部变量对比

|              | 作用域              | 定义位置            | 语法                      |
| ------------ | ------------------- | ------------------- | ------------------------- |
| 会话用户变量 | 当前会话            | 会话的任何地方      | 加@符号，不用指定类型     |
| 局部变量     | 定义它的BEGIN END中 | BEGIN END的第一句话 | 一般不用加@，需要指定类型 |

### 定义条件与处理程序

- 定义条件：事先定义程序执行过程中可能遇到的问题
- 处理程序 定义了在遇到问题时应当采取的处理方式，并且保证存储过程或函数在遇到警告或错误时能继续执行

#### 定义条件

- 定义条件就是给MySQL中的错误码命名，这有助于存储的程序diamagnetic更清晰。它将一个错误名字和指定的错误条件关联起来。这个名字可以随后被用在定义处理程序的DECLARE HANDLER语句中

~~~sql
DECLARE 错误名称 CONDITION FOR MSQL_error_code （或错误条件）;

DECLARE 错误名称 CONDITION FOR SQLSTATE sqlstate_value ;
~~~

- 错误码的说明：
  - MSQL_error_code 和 sqlstate_value 都可以表示MySQL的错误
    - MySQL_error_code 是数值类型错误代码
    - sqlstate_value 是长度为5的字符串类型错误代码
  - 例如：在ERROR 1418(HY000)中，1418是MySQL_error_code,'HY000‘是sqlstate_value

#### 处理程序

- 可以为SQL执行过程中发生的某种类型的错误定义特殊的处理程序。定义处理程序时，使用DELARE语句

~~~sql
DELCLARE 处理方式 HANDLER FOR 错误类型 处理语句
~~~

- 处理方式：处理方式有三个取值：CONTINUE，EXIT，UNDO
  - CONTINUE：表示遇到错误不处理，继续执行
  - EXIT：表示遇到错误马上退出
  - UNDO：表示遇到错误后撤回之前的操作。MySQL中暂时不支持这样的操作
- 错误类型 （即条件）可以有如下取值：
  - SQLSTATE '字符串错误码'：表示长度为5的sqlstate_value类型的错误代码；
  - MySQL_error_code：匹配数值类型错误代码
  - 错误名称：表示DECLARE...CONDITION定义的错误条件名称
  - SQLWARNING：匹配所有以01开头的SQLSTATE错误代码
  - NOT FOUND：匹配所有以02开头的SQLSTATE错误代码
  - SQLEXCEPTION：匹配所有没有被SQLWARING 或 NOT FOUND捕获的SQLSTATE错误代码
- 处理语句：如果出现上述条件之一，则采用对应的处理方式，并执行指定的处理语句。语句可以是像 "SET 变量 = 值"这样的简单语句，也可以是使用 BEGIN ... END 编写的复合语句

### 流程控制

- 解决复杂问题不可能通过一个SQL语句完成，我们需要执行多个SQL操作。流程控制语句的作用就是控制存储过程中SQL语句的执行顺序，是我们完成复杂操作必不可少的一部分。分为三大类：

  - 顺序结构
  - 分支结构
  - 循环结构

- 针对MySQL的流程控制语句主要有3类。

  - 条件判断语句：IF语句和CASE语句

  - 循环语句：LOOP、WHILE 和 REPEAT 语句

  - 跳转语句：ITERATE 和 LEAVE 语句

#### 分支结构之IF
- IF语句的语法结构是：

~~~sql
IF 表达式1 THEN 操作1
[ELSE IF 表达式2 THEN 操作2].....
[ELSE 操作N]
END IF
~~~

#### 分支结构之CASE

- CASE 语句的语法结构1：

~~~sql
#情况一：类似于switch
CASE 表达式
WHEN 值1 THEN 结果1或语句1（如果是语句，需要加分号）
WHEN 值2 THEN 结果2或语句2（如果是语句，需要加分号）
...
ELSE 结果n或语句n（如果是语句，需要加分号）
END [case] (如果是放在beging end中需要加上case，如果放在select后面不需要);

#情况二：类似于多重if
CASE
WHEN 条件1 THEN 结果1或语句1（如果是语句，需要加分号）
WHEN 条件2 THEN 结果2或语句2 （如果是语句，需要加分号）
...
ELSE 结果n或语句n（如果是语句，需要加分号）
END [case] (如果放在begin end 中需要加上case，如果放在select后面不需要)
~~~

#### 循环结构之LOOP







