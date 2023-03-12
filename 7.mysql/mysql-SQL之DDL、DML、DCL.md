# SQL之DDL、DML、DCL

- **DDL（Data Definition Languages）语句：**数据定义语言，这些语句定义了不同的数据段、数据库、表、列、索引等数据库对象的定义。常用的语句关键字主要包括 create、drop、alter等。
- **DML（Data Manipulation Language）语句：**数据操纵语句，用于添加、删除、更新和查询数据库记录，并检查数据完整性，常用的语句关键字主要包括 insert、delete、udpate 和select 等。(增添改查）
- **DCL（Data Control Language）语句：**数据控制语句，用于控制不同数据段直接的许可和访问级别的语句。这些语句定义了数据库、表、字段、用户的访问权限和安全级别。主要的语句关键字包括 grant、revoke 等。

## DDL

### MySQL的数据类型

| 分类             | 数据类型                                                     |
| ---------------- | ------------------------------------------------------------ |
| 整数类型         | TINYINT、SMALLINT、MEDIUMINT、INT(或INTGER)、BIGINT          |
| 浮点类型         | FLOAT、DOUBLE                                                |
| 定点数类型       | DECIMAL                                                      |
| 位类型           | BIT                                                          |
| 日期时间类型     | YEAR、TIME、DATE、DATETIME、TIMESTAMP                        |
| 文本字符串类型   | CHAR、VHARCHAR、TINTEXT、TEXT、MEDIUMTEXT、LONGTEXT          |
| 枚举类型         | ENUM                                                         |
| 集合类型         | SET                                                          |
| 二进制字符串类型 | BINARY、VARBINARY、TINYBLOB、BLOB、MEDIUMBLOB、LONGBLOB      |
| JSON类型         | JSON对象、JSON数组                                           |
| 空间数组类型     | 单值：GEOMETRY、POINT、LINESTRING、POLYGON <br />集合：MULTIPOINT、MULTILINESTRING、MULTIPOLYOGON、GEOMETRYCOLLECTION |

- 其中，常用的几种类型

| 数据类型      | 描述                                                         |
| ------------- | ------------------------------------------------------------ |
| INT           | 从-2^31^到2^31^-1的整形数据，存储大小为4个字节               |
| CHAR(size)    | 定长字符数据。若未指定，默认为1个字符，最大长度255           |
| VARCHAR(size) | 可变长字符数据，根据字符串实际长度保存，必须指定长度         |
| FLOAT(M,D)    | 单精度，占用4个字节，M=整数位+小数位，D=小数位。D <= M <= 255,0 <= D <= 30,默认M + D <= 6 |
| DOUBLE(M,D)   | 双精度，占用8个字节，D <= M <= 255, 0 <= D <= 30，默认 M + D <= 15 |
| DECIMAL(M,D)  | 高精度小数，占用M+2个字节，D <= M <= 65, 0 <= D <= 30,最大取值范围与DOUBLE相同。 |
| DATE          | 日期型数据，格式 'YYYY-MM-DD'                                |
| BLOB          | 二进制形式的长文本数据，最大可达4G                           |
| TEXT          | 长文本数据，最大可达4G                                       |

#### 常见的数据类型的属性

| MySQL关键字        | 含义                     |
| ------------------ | ------------------------ |
| NULL               | 数据列可包含NULL值       |
| NOT NULL           | 数据列不允许包含NULL值   |
| DEFAULT            | 默认值                   |
| PRIMARY KEY        | 主键                     |
| AUTO_INCREMENT     | 自动递增，适用于整数类型 |
| UNSIGNED           | 无符号                   |
| CHARACTER SET name | 指定一个字符集           |

#### 整数类型范围和字节介绍

| 整数类型     | 字节 | 有符号数取值范围        | 无符号取值范围 |
| ------------ | ---- | ----------------------- | -------------- |
| TINYINT      | 1    | -128~127                | 0~255          |
| SMALLINT     | 2    | -32768~32768            | 0~65535        |
| MEDIUMINT    | 3    | -8388608~8388607        | 0~16777215     |
| INT、INTEGER | 4    | -21474832648~2147483647 | 0~4294967295   |
| BIGINT       | 8    | -2^63^~2^63^-1          | 0~2^64^-1      |

#### 整数类型可选属性

- M:表示显示宽度，M的取值范围是(0,255)。例如，int(5)：当数据宽度小于5位的时候在数字前面需要用字符填充宽度。该项功能需要配合"ZEROFILL"使用，表示用"0"填满宽度，否则指定显示宽度无效。
- UNSIGNED：无符号类型(非负)，所有的整数类型都有一个可选的属性UNSIGNED(无符号属性)，无符号整数类型的最小取值为0
- ZEROFILL：0填充

#### 浮点数类型

| 类型   | 区别         | 占用字节数 |
| ------ | ------------ | ---------- |
| FLOAT  | 单精度浮点数 | 4          |
| DOUBLE | 双精度浮点数 | 8          |

- FLOAT和DOUBLE类型在不指定(M,D)时，默认会按照实际的精度(由实际的硬件和操作系统决定)来显示

#### 定点数类型

| 数据类型                 | 字节数  | 含义               |
| ------------------------ | ------- | ------------------ |
| DECIMAL(M,D),DEC,NUMERIC | M+2字节 | 有效返回由M和D决定 |

- 使用DECIMAL(M,D)的方式表示高精度小数。其中，M被称为精度，D被称为标度。0 <= M <= 65,0 <= D <= 30,D < M.例如，定义DECIMAL(5,2)的类型，表示该列取值范围时-999.99~999.99.
- DECIMAL(M,D)总共占用的存储空间为M+2个字节。定点数在MySQL内部是以字符串形式存储，这就决定了它一定是精确的。负号和小数点各占一个字节，M占M个字节
- 当DECIAML类型不指定精度和标度时，其默认为DECIAML(10,0),当数据的精度超出了定点数类型的精度范围时，则MySQL同样会进行四舍五入处理。

#### 位类型

| 二进制字符串类型 | 长度 | 长度范围     | 占用空间                |
| ---------------- | ---- | ------------ | ----------------------- |
| BIT(M)           | M    | 1 <= M <= 64 | 约为 (M + 7) / 8 个字节 |

- BIT类型，如果没有指定(M)，默认是1位，这个1位，表示只能存1位的二进制值。这里(M)是表示二进制的位数，位数最小值为1，最大值为64。

#### 日期与时间类型

| 类型      | 名称     | 字节 | 日期格式            | 最小值                  | 最大值                  |
| --------- | -------- | ---- | ------------------- | ----------------------- | ----------------------- |
| YEAR      | 年       | 1    | YYYY或YY            | 1901                    | 2155                    |
| TIME      | 时间     | 3    | HH:MM:SS            | -838:59:59              | 838:59:59               |
| DATE      | 日期     | 3    | YYYY-MM-DD          | 1000-01-01              | 9999-12-03              |
| DATETIME  | 日期时间 | 8    | YYYY-MM-DD HH:MM:SS | 1000-01-01 00:00:00     | 9999-12-31 23:59:59     |
| TIMESTAMP | 日期时间 | 4    | YYYY-MM-DD HH:MM:SS | 1970-01-01 00:00:00 UTC | 2038-01-19 03:14:07 UTC |

- YEAR：以4位字符串或数字格式表示YEAR类型，其格式为YYYY
- DATE：以YYYY-MM-DD格式或者YYYYMMDD格式表示的字符串日期
- TIME：HH:MM:SS格式表示TIME类型，其中，HH表示小时，MM表示分钟，SS表示秒
- DATETIME类型：以YYYY-MM-DD HH:MM:SS格式或者 YYYYMMDDHHMMSS 格式字符串插入DATETIME类型的字段
  - 以 YYYYMMDDHHMMSS 格式的数字插入DATETIME类型的字段时，会被转化为 YYYY-MM-DD HH:MM:SS 格式
- TIMESTAMP：格式和DATETIME类型相同，但是存储的是UTC时间，UTC表示世界统一时间，也叫作世界标准时间
  - 存储数据的时候需要对当前时间所在的时区进行转换，查询数据的时候再将时间转换回当前的时区。因此，使用TIMESTAMP存储的同一个时间值，在不同的时区会显示不同的时间。

#### CHAR和VARCHAR类型

| 字符串(文本)类型 | 特点     | 长度 | 长度范围        | 占用的存储空间      |
| ---------------- | -------- | ---- | --------------- | ------------------- |
| CHAR(M)          | 固定长度 | M    | 0 <= M <= 255   | M个字节             |
| VARCHAR(M)       | 可变长度 | M    | 0 <= M <= 65535 | (实际长度 +1)个字节 |

- CHAR(M)类型一般需要预先定义字符串长度。如果不指定(M)，则表示长度默认是1个字符
- CHAR类型保存时，数据的实际长度如果比CHAR类型声明的长度小，则会在右侧填充空格以达到指定的长度。当MySQL检索CHAR类型的数据时，CHAR类型的字段会去除尾部的空格。
- VARCHAR：MySQL4.0版本以下,varchar(20):指的是20字节，如果存放UTF8汉字时，只能存6个(每个汉字3字节)；MySQL5.0版本以上，varchar(20)：指的是20字符
- 检索VARCHAR类型的字段数据时，会保留数据尾部的空格。VARCHAR类型的字段所占用的存储空间为字符实际长度加1个字节。多出来1个字节，用于存储信息长度

#### TEXT类型

- 由于实际存储的长度不确定，MySQL不允许TEXT类型的字段做主键。
- 开发经验：
  - TEXT文本类型，可以存比较大的文本段，搜索速度稍慢，因此如果不是特别大的内容，建议使用CHAR，VARCHAR来代替。text和blob类型的数据删除后容易导致“空洞”，使得文件随便较多，所以最好单独用一个表

#### JSON类型

- JSON是一种轻量级的数据交换格式。JSON可以将JavaScript对象中表示的一组数据转换为字符串，然后就可以在网络或者程序之间轻松地传递这个字符串，并在需要的时候将它还原为各编程语言所支持的数据格式
- 通过“->”和"->>"符号，从JSNO字段中正确查询出了指定的JSNO数据的值

~~~sql
INSERT INTO test_json(js)
VALUES ('{"name":"sry","age":18,"address":{"province":"shanghai","city":"shanghai"}}');

SELECT * FROM test_json;

SELECT 
  js -> '$.name' AS NAME, js -> '$.age' AS age
FROM test_json; 
~~~



### 创建和管理数据库

~~~SQL
CREATE DATABASE 数据库名;  //方式1：创建数据库

CREATE DATATBASE 数据库名 CHARACTER SET 字符集; //方式2：创建数据库并指定字符集

CREATE DATABASE IF NOT EXISTS 数据库名; //方式3：判断数据库是否已经存在，不存在则创建数据库(推荐)
~~~

### 使用数据库

~~~SQL
SHOW DATABASES; //有一个S，代表多个数据库

SELECT DATABASE(); //使用的一个mysql中的全局函数

SHOW TABLES FROM 数据库名; //查看指定库下所有的表

SHOW CREATE DATABASE 数据库名; //查看数据库的创建信息

USE 数据库名; //使用/切换数据库
~~~

### 修改数据库

~~~SQL
ALTER DATABASE 数据库名 CHARACTER SET 字符集; //更改数据库字符集

DROP DATABASE 数据库名; //删除指定的数据库

DROP DATABASE IF EXISTS 数据库名 ; //删除指定的数据库(推荐)
~~~

### 创建和管理表

### 创建表方式一

~~~SQL
CREATE TABLE [IF NOT EXISTS] 表名 (
    字段1，数据类型 [约束条件] [默认值],
    字段2，数据类型 [约束条件] [默认值],
    字段3，数据类型 [约束条件] [默认值],
    ....
    [表约束条件]
);
~~~

### 创建表方式二

~~~SQL
CREATE TABLE table[ (column,column...)] AS subquery
~~~

- 指定的列和子查询中的列要一一对应
- 通过列名和默认值定义列

### 查看表结构

~~~SQL
DESCRIBE 表名;
SHOW CREATE TABLE 表名;
~~~

### 修改表

- 向已有的表中添加列

~~~SQL
ALTER TABLE ADD [COLUMN] 字段名 字段类型 [约束条件] [FIRST | AFTER 字段名]; 
//FIRST 表头添加字段
//AFTER 表尾添加字段
~~~

- 修改现有表中的列

~~~SQL
ALTER TABLE 表名 MODIFY [COLUMN] 字段名1 字段类型 [DEFAULT 默认值] [FIRST | AFTER 字段名2];
~~~

- 删除现有表中的列

~~~SQL
ALTER TABLE 表名 DROP [COLUMN] 字段名;
~~~

- 重命名现有表中的列

~~~SQL
ALTER TABLE 表名 CHANGE [column] 列名 新列名 新数据类型;
~~~

### 清空表

~~~SQL
TRUNCATE TABLE detail_dept;
//TRUNCATE 语句不能回滚，而使用 DELETE 语句删除数据，可以回滚
~~~

拓展：MySQL8新特性-DDL的原子化

## DML

### 插入数据

~~~SQL
INSERT INTO 表名 VALUES (value1,value2,....); //为表的所有字段按默认顺序插入数据

INSERT INTO 表名 (column1 [, column2,...,columnn]) VALUES (value1 [, value2,...,.valuen]); 
//为表的指定字段插入数据,就是在INSERT语句中只向部分字段中插入值,而其他字段的值为表定义时的默认值

INSERT INTO table_name VALUES 
(value1 [, value2,...,valuen]),
(value1 [,value2,....,valuen]),
...
(value1 [,value2,....,valuen]);
或者
INSERT INTO table_name(column1 [, column2, ...,column]) VALUES
(value1 [, value2,...,valuen]),
(value1 [, value2,...,valuen]),
...
(value1 [, value2,...valuen]);
//INSERT语句可以同时向数据表中插入多条记录，插入时指定多个值列表，每个值列表之间用逗号分割开

INSERT INTO 目标表名 (tar_colum1 [, tar_colum2,...,tar_columnn])
SELECT (src_column [, src_column2,...,src_columnn])
FROM 源表名
[WHERE condition]
//INSERT还可以将SELECT语句查询的结果插入到表中
//在INSERT语句中加入子查询
//不必数据VALUES子句
//子查询中的值列表应与INSERT子句中的列名对应
~~~

### 更新数据

~~~SQL
UPDATE table_name
SET column1=value1,column2=value2,column=valuen
[WHERE condition]
//可以一次更新多条数据,如果需要回滚数据，需要保证在DML前，进行设置：SET AUTOCOMMIT = FALSE;
~~~

### 删除数据

~~~sql
DELETE FROM table_name [WHERE <condition>];
table_name指定要执行删除操作的表;"[WHERE]"为可选参数，指定删除条件，如果没有WHERE子句，DELETE语句将删除表中的所有记录
~~~

### MySQL新特性:计算列

- 计算列：某一列的值是通过别的列计算得来的。在MySQL8中，CREATE TABLE 和 ALTER TABLE 中都支持增加计算列

~~~
CREATE TABLE tb1(
id INT,
a INT,
b INT,
c INT GENERATED ALWAYS AS (a+b) VIRTUAL
)
~~~

## 约束

- 数据完整性：指数据的精确性(Accuracy)和可靠性(Reliability)

- 从以下四个方面考虑

  - 实体完整性：例如，同一个表中，不能存在两条完全相同无法区分的记录。
  - 域完整性：例如，年龄范围0~120，性别范围"男/女"
  - 引用完整性：例如：员工所在部分，在部门表中要能找到这个部门
  - 用户自定义完整性：例如：用户名唯一、密码不能为空，本部门经理地工资不得高于本部门职工的平均工资的5倍。

- 约束是表级的强制规定：可以在创建表时规定约束(通过CREATE TABLE 语句)，或者在表创建之后通过 ALTER TABLE 语句规定约束

- 约束的分类

  - 根据约束数据列的限制
    - 单列约束：每个约束只约束一列
    - 多列约束：每个约束可约束多列数据
    
  - 根据约束的作用范围：
    - 列级约束：只能作用在一个列上，跟在列的定义后面
    - 表级约束：可以作用在多个列上，不与列一起，而是单独定义
    
    |          | 位置         | 支持的约束类型             | 是否可以起约束名     |
    | -------- | ------------ | -------------------------- | -------------------- |
    | 列级约束 | 列的后面     | 语法都支持，但外键没有效果 | 不可以               |
    | 表级约束 | 所有列的下面 | 默认和非空不支持，其他支持 | 可以（主键没有效果） |
    
    
    
  - 根据约束的作用：
    - NOT NULL：非空约束，规定某个字段不能为空
    - UNIQUE：唯一约束，规定某个字段在整个表中是唯一的
    - PRIMARY KEY：主键（非空且唯一）约束
    - FOREIGN KEY：外键约束
    - CHECK：检查约束
    - DEFAULT：默认值约束
    
  - 查看某个表已有的约束
  
  ~~~SQL
  SELECT * FROM information_schema.table_constraints where table_name = '表名称';
  ~~~

### 非空约束

- 作用：限定某个字段/某列的值不允许为空

- 关键字：NOT NULL

- 特点

  - 默认，所有的类型的值都可以是NULL，包括INT、FLOAT等数据类型
  - 非空约束只能出现在表对象的列上，只能某个列单独限定非空、不能组合非空
  - 一个表可以由很多列都分别限定了非空

- 添加非空约束

  - 建表

  ~~~SQL
  CREATE TABLE 表名称(
      字段名 数据类型,
      字段名 数据类型 NOT NULL,
      字段名 数据类型 NOT NULL
  );
  ~~~

  - 建表后

  ~~~SQL
  ALTER TABLE 表名称 MODIFY 字段名 数据类型 NOT NULL
  ~~~

  - 删除非空约束

  ~~~SQL
  ALTER TABLE 表名称 MODIFY 字段名 数据类型 NULL;
  
  ALTER TABLE 表名称 MODIFY 字段名 数据类型;
  ~~~

### 唯一性约束

- 作用：用来限制某个字段/某列的值不能重复
- 关键字：UNIQUE
- 特点：
  - 同一个表可以有多个唯一约束
  - 唯一约束可以是某一个列的值唯一，也可以多个列组合的值唯一
  - 唯一性约束允许列值为空
  - 在创建唯一约束的时候，如果不给唯一约束命名，就默认和列明相同。
  - MySQL会给唯一约束的列上默认创建一个唯一索引。
- 建表

~~~
CREATE TABLE 表名称 (
	字段名 数据类型,
	字段名 数据类型 UNIQUE,
	字段名 数据类型 UNIQUE KEY,
	字段名 数据类型,
	字段名 数据类型,
	[CONSTRAINT 约束名] UNIQUE KEY(字段名)
);
~~~

- 建表后指定唯一键约束

~~~SQL
ALTER TABLE 表名称 ADD UNIQUE KEY(字段列表);

ALTER TABLE 表名称 MODIFY 字段名 字段类型 UNIQUE;
##字段列表中如果是一个字段，表示该列的值唯一，如果是两个或更多个字段，那么复合唯一，即多个字段的组合是唯一的
~~~

- 删除唯一约束
  -  添加唯一性约束的列上会自动创建唯一索引
  - 删除唯一约束只能通过删除唯一索引的方式删除
  - 删除时需要指定唯一索引名，唯一索引名就和唯一约束名一样
  - 如果创建唯一约束时未指定名称，如果是单列，就默认和列名相同；如果是组合列，那么默认和()中排在第一个的列名相同。也可以自定义唯一性约束名。

~~~SQL
SELECT * FROM information_schema.table_constraints WHERE TABLE_NAME = '表名'; ##查看都有哪些约束

SHOW INDEX FROM 表名称; ##查看表的索引

ALTER TABLE USER DROP INDEX uk_name_pwd;
~~~

### 主键

- 作用：用来唯一标识表中的一行记录
- 关键字：PRIMARY KEY
- 特点：
  - 主键约束相当于唯一约束+非空约束的组合，主键约束列不允许重复，也不允许出现空值
  - 一个表最多只能有一个主键约束，建立主键约束可以在列级别创建，也可以在表级别上创建
  - 主键约束对应着表中的一列或者多列（复合主键）
  - 如果是多列组合的复合主键约束，那么这些列都不允许为空值，并且组合的值不允许重复
  - MySQL的主键名总是PRIMAY
  - 当创建主键约束时，系统默认会在所在的列或组合上建立对应的主键索引（能够根据主键查询的，就根据主键查询，效率更高）。如果删除主键约束了，主键约束对应的索引就自动删除了。
  - 需要注意的一点是，不要修改主键字段的值。因为主键是数据记录的唯一标识，如果修改了主键的值，就有可能会破环数据的完整性。
- 添加主键约束

~~~SQL
CREATE TABLE 表名称 (
	字段名 数据类型,
	字段名 数据类型 PRIMARY KEY,
	字段名 数据类型
	[CONSTRAINT 约束名] PRIMARY KEY(字段名1,字段名2) ##表级模式
)
~~~

- 建表后添加主键约束

~~~SQL
ALTER TABLE 表名称 ADD PRIMARY KEY(字段列表); ##字段列表可以是一个字段，也可以是多个字段，如果是多个字段的话，是复合主键
~~~

- 删除主键约束

~~~SQL
ALTER TABLE 表名称 DROP PRIMARY KEY;
~~~

### 自增列

- 作用：某个字段的值自增
- 关键字：AUTO_INCREMENT
- 特点
  - 一个表最多只能有一个自增长列
  - 当需要产生唯一标识符或顺序值时，可设置自增长
  - 自增长列约束的列必须是键列（主键列，唯一键列）
  - 自增约束的列的数据类型必须是整数类型
  - 如果自增列指定了0和null，会在当前最大值的基础上自增；如果自增列手动指定了具体值，直接赋值为具体值
  - 自增列可以指定自增开始值
- 建表

~~~SQL
CREATE TABLE 表名称 (
	字段名 数据类型 PRIMARY KEY AUTO_INCREMENT,
	字段名 数据类型 UNIQUE KEY NOT NULL
	字段名 数据类型
)AUTO_INCREMENT = 100
~~~

- 建表后

~~~SQL
ALTER TABLE 表名称 MODIFY 字段名 数据类型 AUTO_INCREMENT;

ALTER TABLE 表名称 AUTO_INCREMENT=100;
~~~

- 如何删除自增约束

~~~SQL
ALTER TABLE 表名称 MODIFY 字段名 数据类型;
~~~

- 如何增加自增约束

~~~SQL
ALTER TABLE 表名称 MODIFY 字段名 数据类型 AUTO_INCREMENT;
~~~

- 重置自增列数据

~~~SQL
##删除自增字段，添加自增字段，并重新赋值
ALTER TABLE 表名称 DROP 主键;
ALTER TABLE 表名称 ADD 列名 INT NOT NULL PRIMARY KEY AUTO_INCREMENT FIRST;

##删除部分数据，然后重置id
DELETE FROM 表名称 WHERE;
ALTER TABLE 表名称 AUTO_INCREMENT=100;
~~~

### FOREIGN KEY约束

- 作用：限定某个表的某个字段的引用完整性
- 关键字：FOREIGN KEY
- 主表和从表
  - 主表：被引用的表，被参考的表
  - 从表：引用别人的表，参考别人的表
- 特点
  - 从表的外键列，必须引用/参考主表的主键或唯一约束的列
  - 在创建外键约束时，如果不给外键约束名，默认名不是列名，而是自动产生一个外键名
  - 创建表时就指定外键约束的话，先创建主表，再创建从表
  - 删表时，先删从表（或先删除外键约束），再删除主表
  - 当主表的记录被从表参照时，主表的记录将不允许删除，如果删除数据，需要先删除从表中依赖该记录的数据，然后才可以删除主表的数据
  - 在"从表"中指定外键约束，并且一个表可以建立多个外键约束
  - 从表的外键列与主表被参照的列名字可以不相同，但是数据类别必须一致，逻辑意义一致。如果类型不一样，创建子表时，就会出现错误
  - 删除外键约束后，必须手动删除对应的索引
- 添加外键约束
- 建表

~~~SQL
CREATE TABLE 主表名称(
	字段1 数据类型 PRIMARY KEY,
	字段2 数据类型
);

CREATE TABLE 从表名称(
	字段1 数据类型 PRIMARY KEY,
	字段2 数据类型,
	[CONSTRAINT <外键约束名称>] FOREIGN KEY (从表的某个字段) references 主表名(能参考字段)
)
~~~

- 建表后

~~~SQL
ALTER TABLE 从表名 ADD [CONSTRAINT 约束名] FOREIGN KEY(从表的字段) REFERENCES 主表名 (被引用字段) [on update xx] [on delete xx];
~~~

- 外键约束关系
  - 添加了外键约束后，主表的修改和删除数据受约束
  - 添加了外键约束后，从表的添加和修改数据受约束
  - 在从表上建立外键，要求主表必须存在
  - 删除主表时，要求从表先删除，或将从表中外键引用该主表的关系先删除
- 约束等级
  - Cascade方式：在父表上update/delete记录时，同步update/delete子表的匹配记录
  - Set null方式：在父表上update/delete记录时，将子表上匹配记录的列设为null，但要注意子表的外键列不能为not null
  - No action方式：如果子表中有匹配的记录，则不允许对父表对应候选键进行update/delete操作
  - Restrict方式：同no action 方式，都是立即检查外键约束
  - Set default方式：父表有变更时，子表将外键列设置成一个默认的值，但Innodb不能识别

- 删除外键约束

~~~SQL
## 第一步 先查看约束名和删除外键约束
SELECT * FROM information_schema.table_constraints WHERE table_name = '表名称'; ##查案某个表的约束名

ALTER TABLE 从表名 DROP FOREIGN KEY 外键约束名;

## 第二步 查看索引名和删除索引。（注意，只能手动删除）
SHOW INDEX FROM 表名称;

ALTER TABLE 从表名 DROP INDEX 索引名;
~~~

### CHECK约束

- 作用：检查某个字段的值是否符合xx要求，一般指的是值的范围
- 关键字：CHECK
- 创建表

~~~SQL
CREATE TABLE 表名(
	字段名 数据类型,
	字段名 数据类型,
	字段名 数据类型 CHECK(约束条件) [[NOT] ENFORCED]; ##ENFORCED表示是否强制，默认是强制的，即会对改变的数据进行约束，NOT ENFORCED表示check约束不作用
);
~~~

- 添加CHECK约束

~~~SQL
ALTER TABLE `表名` ADD [CONSTRAINT 约束名] CHECK(约束条件) [[NOT] ENFORCED];
~~~

- 删除CHECK约束

~~~SQL
ALTER TABLE `表名` DROP CHECK 约束名;
~~~

- 修改check约束的强制性

~~~SQL
ALTER TABLE `表名` ALTER CHECK 约束名 [NOT] ENFORCED;
~~~

### DEFAULT约束

- 作用：给某个字段/某列指定默认值，一旦设置默认值，再插入数据时，如果此字段没有显示赋值，则赋值为默认值
- 关键字：DEFAULT
- 建表

~~~SQL
CREATE TABLE 表名称 (
	字段名 数据类型 PRIMARY KEY,
	字段名 数据类型 UNIQUE KEY NOT NULL,
	字段名 数据类型 UNIQUE KEY,
	字段名 数据类型 NOT NULL DEFAULT 默认值
);
~~~

- 建表后

~~~SQL
ALTER TABLE 表名称 MODIFY 字段名 数据类型 DEFAULT 默认值;
~~~

- 删除默认值约束

~~~SQL
ALTER TABLE MODIFY 字段名 数据类型; ## 删除默认值约束，也不保留非空约束

ALTER TABLE MODIFY 字段名 数据类型 NOT NULL;## 删除默认值约束，保留非空约束
~~~



