# 深入浅出MySQL

# 一、基础篇

## 1.SQL基础

### 1.1 SQL分类 

##### SQL 语句主要可以划分为以下 3 个类别。

 􏰁 DDL(Data Definition Languages)语句:数据定义语言，这些语句定义了不同的数据段、 数据库、表、列、索引等数据库对象的定义。常用的语句关键字主要包括 create、drop、alter 等。
 􏰁 DML(Data Manipulation Language)语句:数据操纵语句，用于添加、删除、更新和查 询数据库记录，并检查数据完整性，常用的语句关键字主要包括 insert、delete、udpate 和 select 等。
 􏰁 DCL(Data Control Language)语句:数据控制语句，用于控制不同数据段直接的许可和 访问级别的语句。这些语句定义了数据库、表、字段、用户的访问权限和安全级别。主要的 语句关键字包括 grant、revoke 等。 

### 1.2 DDL语句

​	**DDL 是数据定义语言的缩写，简单来说，就是对数据库内部的对象进行创建、删除、修改的操作语言。它和 DML 语言的最大区别是 DML 只是对表内部数据的操作，而不涉及到表的定义、结构的修改，更不会涉及到其他对象。**	

1.删除数据库：drop database dbname;	(开发中不要轻易使用删除语句，否则可能会导致一些数据缺失)

2.查看表的定义：DESC tablename

3.查看创建表的 SQL 语句：show create table emp \G;	(可以看到表定义以外，还可以看到表的 engine(存储引擎)和 charset(字符集)等信息。“\G”选项的含义是使得记录能够按照字段竖着排列，对于内容比较长的记录更易于显示。

4.删除表：DROP TABLE tablename

5.修改表：

5.1 修改表类型：ALTER TABLE tablename MODIFY [COLUMN] column_definition [FIRST | AFTER col_name]；如：修改表 emp 的 ename 字段定义，将 varchar(10)改为 varchar(20)                                  	 alter table emp modify ename varchar(20);

5.2 增加表字段：ALTER TABLE tablename ADD [COLUMN] column_definition [FIRST | AFTER col_name]

5.3 删除表字段：ALTER TABLE tablename DROP [COLUMN] col_name

5.4 字段改名(可以修改字段类型和名字)：![image-20190108230228581](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190108230228581-6959748.png)

5.5 修改字段排列顺序：前面介绍的的字段增加和修改语法(ADD/CNAHGE/MODIFY)中，都有一个可选项 first|after column_name，这个选项可以用来修改字段在表中的位置，默认 ADD 增加的新字段是加在
表的最后位置，而 CHANGE/MODIFY 默认都不会改变字段的位置。                                                                        		

​	如：alter table emp add birth date after ename;                                                                                                   

​	修改字段 age，将它放在最前面:alter table emp modify age int(3) first;

5.6 表改名：ALTER TABLE tablename RENAME [TO] new_tablename

### 1.3 DML语句

​	**DML 操作是指对数据库中表记录的操作，主要包括表记录的插入(insert)、更新(update)、删除(delete)和查询(select)，是开发人员日常使用最频繁的操作。**

#### 1.插入记录：

INSERT INTO tablename (field1,field2,......fieldn) VALUES(value1,value2,......valuesn);

**注意：对于含可空字段、非空但是含有默认值的字段、自增字段，可以不用在 insert 后的字段列表里面出现，values 后面只写对应字段名称的 value，这些没写的字段可以自动设置为 NULL、默认值、自增的下一个数字，这样在某些情况下可以大大缩短 SQL 语句的复杂性。**

1.1 一次插入多条记录(用逗号隔开)：

INSERT INTO tablename (field1, field2,......fieldn) 

VALUES
 (record1_value1, record1_value2,......record1_valuesn), (record2_value1,record2_value2,......record2_valuesn), 

...... 

(recordn_value1, recordn_value2,......recordn_valuesn) ; 

如：insert into dept values(5,'dept5'),(6,'dept6');

#### 2.更新记录：

UPDATE tablename SET field1=value1，field2.=value2，......fieldn=valuen [WHERE CONDITION]

2.1 更新多条记录：

UPDATE t1,t2...tn set t1.field1=expr1,tn.fieldn=exprn [WHERE CONDITION]

如：update emp a,dept b set a.sal=a.sal*b.deptno, b.deptname=a.ename where  a.deptno=b.deptno;

##### 注意:多表更新的语法更多地用在了根据一个表的字段，来动态的更新另外一个表的字段

#### 3.删除记录

DELETE FROM tablename [WHERE CONDITION]

3.1 删除多条记录

DELETE t1,t2...tn FROM t1,t2...tn [WHERE CONDITION]

如：delete a,b from emp a,dept b where a.deptno=b.deptno and a.deptno=3;

##### 注意:不管是单表还是多表，不加 where 条件将会把表的所有记录删除，所以操作时一定要小心。

#### 4.查询记录

SELECT * FROM tablename [WHERE CONDITION]

##### (1)查询不重复的记录。

用 distinct 关键字来实现，如：select distinct deptno from emp;选择不重复的deptno

##### (2)排序和限制。

用关键字 ORDER BY 来实现：

![image-20190108232903393](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190108232903393-6961343.png)

​	**其中，DESC 和 ASC 是排序顺序关键字，DESC 表示按照字段进行降序排列，ASC 则表示升序排列，如果不写此关键字默认是升序排列。ORDER BY后面可以跟多个不同的排序字段，并且每个排序字段可以有不同的排序顺序。**==如果排序字段的值一样，则值相同的字段按照第二个排序字段进行排序，以此类推。==如果只有一个排序字段，则这些字段相同的记录将会无序排列。

如：select * from emp order by deptno,sal desc;先按deptno默认排序(升序)，然后再按sal降序排序。

对于排序后的记录，如果希望只显示一部分，而不是全部，这时，就可以使用 LIMIT 关键字来实现：

​	**SELECT ......[LIMIT offset_start,row_count]**

​	==其中 offset_start 表示记录的起始偏移量，row_count 表示显示的行数。==
在默认情况下，起始偏移量为 0，只需要写记录行数就可以，这时候，**显示的实际就是前 n条记录**，

如显示 emp 表中按照 sal 排序后从第二条记录开始，显示 3 条记录：select * from emp order by sal limit 1,3;

##### (3)聚合。

![image-20190109111428759](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190109111428759-7003668.png)

对其参数进行以下说明。
**􏰁 fun_name表示要做的聚合操作，也就是聚合函数，常用的有sum(求和)、count(*)(记录数)、max(最大值)、min(最小值)。**
**􏰁 GROUP BY 关键字表示要进行分类聚合的字段，比如要按照部门分类统计员工数量，部门就应该写在 group by 后面。**
**􏰁 WITH ROLLUP 是可选语法，表明是否对分类聚合后的结果进行再汇总。**
**􏰁 HAVING关键字表示对分类后的结果再进行条件的过滤。**

==注意:==having 和 where 的区别在于 having 是对聚合后的结果进行条件的过滤，而 where 是在聚合前就对记录进行过滤，如果逻辑允许，我们尽可能用 where 先过滤记录，这样因为结果集减小，将对聚合的效率大大提高，最后再根据逻辑看是否用 having 进行再过滤。

##### (4)表连接。

​	当需要同时显示多个表中的字段时，就可以用表连接来实现这样的功能。**从大类上分，表连接分为内连接和外连接，它们之间的最主要区别是內连接仅选出两张表中互相匹配的记录，而外连接会选出其他不匹配的记录。**	

​	外连接有分为左连接和右连接：

• 左连接:包含所有的左边表中的记录甚至是右边表中没有和它匹配的记录 

• 右连接:包含所有的右边表中的记录甚至是左边表中没有和它匹配的记录 

##### (5)子查询

​	某些情况下，当我们查询的时候，需要的条件是另外一个 select 语句的结果，这个时候，就要用到子查询。用于子查询的关键字主要包括 in、not in、=、!=、exists、not exists 等。	

如：select * from emp where deptno in(select deptno from dept);

​	select * from emp where deptno = (select deptno from dept limit 1);  子查询结果集返回多条记录时，可以用limit关键字进行限制一条记录。

##### (6)记录联合

​	将两个表的数据按照一定的查询条件查询出来后，将结果合并到一起显示出来，这个时候，就需要用 union 和 union all 关键字来实现这样的功能：

SELECT * FROM t1 

UNION|UNION ALL 

SELECT * FROM t2 

...... 

UNION|UNION ALL 

SELECT * FROM tn; 

​	**UNION 和 UNION ALL 的主要区别是 UNION ALL 是把结果集直接合并在一起，而 UNION 是将UNION ALL 后的结果进行一次 DISTINCT，去除重复记录后的结果。**

如：select deptno from emp union all select deptno from dept; 显示emp表和dept表中所有的deptno字段的值。

### 1.4 DCL语句

​	DCL 语句主要是 DBA 用来管理系统中的对象权限时所使用，一般的开发人员很少使用。下面通过一个例子来简单说明一下。	

创建一个数据库用户 z1，具有对 sakila 数据库中所有表的 SELECT/INSERT 权限:

 **grant select,insert on sakila.* to 'z1'@'localhost' identified by '123';**
**其中，'z1'表示用户名，'123'表示密码，'localhost'表示数据库的ip地址。**

由于权限变更，需要将 z1 的权限变更，收回 INSERT，只能对数据进行 SELECT 操作:

revoke insert on sakila.* from 'z1'@'localhost';

### 1.5 帮助语句

​	可以用“? contents”的方式查询相关命令的使用方法或相关数据类型等，比如：

? show  查询show命令的作用		? create table  参看 CREATE TABLE 的语法

## 2.数据类型

### 2.1数值类型

​	MySQL 支持所有标准 SQL 中的数值类型，其中包括严格数值类型(INTEGER、SMALLINT、 DECIMAL和NUMERIC)，以及近似数值数据类型(FLOAT、REAL和DOUBLE PRECISION)，并在此基础上做了扩展。扩展后增加了 TINYINT、MEDIUMINT 和 BIGINT 这 3 种长度不同的整型，并增加了 BIT 类型，用来存放位数据。

![image-20190109114619839](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190109114619839-7005579.png)

​	在整数类型中，按照取值范围和存储方式不同，分为 tinyint、smallint、mediumint、int、bigint 这 5 个类型。如果超出类型范围的操作，会发生“Out of range”错误􏰀示。	

​	对于整型数据，MySQL 还支持在类型名称后面的小括号内指定显示宽度，例如 int(5)表示当数值宽度小于 5 位的时候在数字前面填满宽度，**==如果不显示指定宽度则默认为 int(11)。==**一般配合 zerofill 使用，顾名思义，zerofill 就是用“0”填充的意思，也就是在数字位数不够的空间用字符“0”填满。以下几个例子分别􏰄述了填充前后的区别。	

如：alter table t1 modify id1 int zerofill;加入zerofill参数之后，通过select语句就可以看到补充的0了，但是不影响插入的数据宽度。

​	**所有的整数类型都有一个可选属性 UNSIGNED(无符号)，**如果需要在字段里面保存非负数或者需要较大的上限值时，可以用此选项，它的取值范围是正常值的下限取 0，上限取 原值的 2 倍，例如，tinyint 有符号范围是-128~+127，而无符号范围是 0~255。**如果一个列 指定为 zerofill，则 MySQL 自动为该列添加 UNSIGNED 属性。** 

​	另外，整数类型还有一个属性:AUTO_INCREMENT。**在需要产生唯一标识符或顺序值时，可利用此属性，这个属性只用于整数类型。**AUTO_INCREMENT 值一般从 1 开始，每行增加 1。在插入 NULL 到一个 AUTO_INCREMENT 列时，MySQL 插入一个比该列中当前最大值大 1 的值。==一个表中最多只能有一个AUTO_INCREMENT列。对于任何想要使用AUTO_INCREMENT 的列，应该定义为NOT NULL，并定义为PRIMARY KEY或定义为UNIQUE键。==	

如以下任何一种都可以：

```sql
CREATE TABLE AI (ID INT AUTO_INCREMENT NOT NULL PRIMARY KEY); 
CREATE TABLE AI(ID INT AUTO_INCREMENT NOT NULL ,PRIMARY KEY(ID));
CREATE TABLE AI (ID INT AUTO_INCREMENT NOT NULL ,UNIQUE(ID));
```

​	对于小数的表示，**MySQL 分为两种方式:浮点数和定点数。**浮点数包括 float(单精度) 和 double(双精度)，而**定点数则只有 decimal 一种表示。**==定点数在 MySQL 内部以字符串形式存放，比浮点数更精确，适合用来表示货币等精度高的数据。==

​	浮点数和定点数都可以用类型名称后加“(M,D)”的方式来进行表示，**“(M,D)”表示该 值一共显示 M 位数字(整数位+小数位)，其中 D 位位于小数点后面，M 和 D 又称为精度和标度。**例如，定义为 float(7,4)的一个列可以显示为-999.9999。**MySQL 保存值时进行四舍五 入，因此如果在 float(7,4)列内插入 999.00009，近似结果是 999.0001。**值得注意的是，浮点 数后面跟“(M,D)”的用法是非标准用法，如果要用于数据库的迁移，则最好不要这么使用。 **float 和 double 在不指定精度时，默认会按照实际的精度(由实际的硬件和操作系统决定) 来显示，而 decimal 在不指定精度时，默认的整数位为 10，默认的小数位为 0，即只会插入整数。** 

注意:在今后关于浮点数和定点数的应用中，用户要考虑到以下几个原则:

- 浮点数存在误差问题;
- 对货币等对精度敏感的数据，应该用定点数表示或存储;
- 在编程中，如果用到浮点数，要特别注意误差问题，并尽量避免做浮点数比较;
- 要注意浮点数中一些特殊值的处理。

### 2.2 日期时间类型

​	MySQL5.0版本：

![image-20190110102356344](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190110102356344-7087036.png)

这些数据类型的主要区别如下:

- 􏰃  如果要用来表示年月日，通常用DATE来表示。 

- 􏰃  如果要用来表示年月日时分秒，通常用DATETIME表示。 

- 􏰃  如果只用来表示时分秒，通常用TIME来表示。 

- 􏰃  如果需要经常插入或者更新日期为当前系统时间，则通常使用 TIMESTAMP 来表示。 TIMESTAMP值返回后显示为“**==YYYY-MM-DD HH:MM:SS==**”格式的字符串，显示宽度固定 为 19 个字符。如果想要获得数字值，应在 TIMESTAMP 列添加+0。 

- 􏰃  如果只是表示年份，可以用YEAR来表示，它比DATE占用更少的空间。YEAR有2位或 4 位格式的年。默认是 4 位格式。在 4 位格式中，允许的值是 1901~2155 和 0000。在 2位格式中，允许的值是70~69，表示从1970~2069年。MySQL以YYYY格式显示YEAR 

  值。 

TIMESTAMP还有一个重要特点，就是和时区相关。当插入日期时，会先转换为本地时区后存放;而从数据库里面取出时，也同样需要将日期转换为本地时区后显示。这样，两个不同时区的用户看到的同一个日期可能是不一样的，

**查看当前时区：show variables like 'time_zone';**

**修改时区为东九区(+9:00)：set time_zone='+9:00';**

TIMESTAMP和DATETIME的表示方法非常类似，区别主要有以下 几点。 

- 􏰅 TIMESTAMP支持的时间范围较小，其取值范围从19700101080001到2038年的某个 时间，而DATETIME是从1000-01-01 00:00:00到9999-12-31 23:59:59，范围更大。 
- 􏰅 表中的第一个TIMESTAMP列自动设置为系统时间。如果在一个TIMESTAMP列中插入 NULL，则该列值将自动设置为当前的日期和时间。在插入或更新一行但不明确给 TIMESTAMP列赋值时也会自动设置该列的值为当前的日期和时间，当插入的值超出 取值范围时，MySQL认为该值溢出，使用“0000-00-00 00:00:00”进行填补。 
- TIMESTAMP的插入和查询都受当地时区的影响，更能反应出实际的日期。而 DATETIME则只能反应出插入时当地的时区，其他时区的人查看数据必然会有误差 的。 
- 􏰅  TIMESTAMP的属性受MySQL版本和服务器SQLMode的影响很大， 

##### 日期类型的数据插入格式(以 DATETIME 为例)：

- 􏰃 YYYY-MM-DDHH:MM:SS或YY-MM-DDHH:MM:SS格式的字符串。允许“不严格”语法:任何标点符都可以用做日期部分或时间部分之间的间割符。例如，“98-12-31 11:30:45”、“98.12.31 11+30+45”、“98/12/31 11*30*45”和“98@12@31 11^30^45”是等价的。对于包括日期部分间割符的字符串值，如果日和月的值小于 10，不需要指定两位数。“1979-6-9”与“1979-06-09”是相同的。同样，对于包括时间部分间割符的字符串值，如果时、分和秒的值小于 10，不需要指定两位数。“1979-10-30 1:2:3”与“1979-10-30 01:02:03”相同。

- YYYYMMDDHHMMSS或YYMMDDHHMMSS格式的没有间割符的字符串，假定字符 串对于日期类型是有意义的。例如，“19970523091528”和“970523091528”被解 释为“1997-05-23 09:15:28”，但“971122129015”是不合法的(它有一个没有意 义的分钟部分)，将变为“0000-00-00 00:00:00”。 
- YYYYMMDDHHMMSS或YYMMDDHHMMSS格式的数字，假定数字对于日期类型是 有意义的。例如，19830905132800和830905132800被解释为“1983-09-05 13:28:00”。 数字值应为 6、8、12 或者 14 位长。如果一个数值是 8 或 14 位长，则假定为 YYYYMMDD 或 YYYYMMDDHHMMSS 格式，前 4 位数表示年。如果数字 是 6 或 12 位长，则假定为 YYMMDD 或 YYMMDDHHMMSS 格式，前 2 位数表示年。其他数字 被解释为仿佛用零填充到了最近的长度。 
- 函数返回的结果，其值适合DATETIME、DATE或者TIMESTAMP上下文，例如NOW() 或 CURRENT_DATE。 

![image-20190110104251498](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190110104251498-7088171.png)

##### 日期类型的选择：

- 􏰃根据实际需要选择能够满足应用的最小存储的日期类型。如果应用只需要记录“年 份”，那么用 1 个字节来存储的 YEAR 类型完全可以满足，而不需要用 4 个字节来 存储的 DATE 类型。这样不仅仅能节约存储，更能够􏰀高表的操作效率。 
- 如果要记录年月日时分秒，并且记录的年份比较久远，那么最好使用 DATETIME， 而不要使用 TIMESTAMP。因为 TIMESTAMP 表示的日期范围比 DATETIME 要短得多。 􏰃 
- 如果记录的日期需要让不同时区的用户使用，那么最好使用 TIMESTAMP，因为日期类型中只有它能够和实际时区相对应。

### 2.3 字符串类型

​	以 5.0 版本为例，MySQL 包括了 CHAR、VARCHAR、BINARY、VARBINARY、BLOB、TEXT、ENUM 和 SET 等多种字符串类型。	

![image-20190110104339978](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190110104339978-7088220.png)

#### 2.3.1 CHAR和VARCHAR类型

​	CHAR 和 VARCHAR 很类似，都用来保存 MySQL 中较短的字符串。**二者的主要区别在于存储方式的不同:CHAR 列的长度固定为创建表时声明的长度，长度可以为从 0~255 的任何值;而VARCHAR列中的值为可变长字符串，长度可以指定为0~25(5 5.0.3以前)或者6553(5 5.0.3以后)之间的值。**==在检索的时候，CHAR 列删除了尾部的空格，而 VARCHAR 则保留这些空格。==

如：向数据库中两个类型的字段插入“ab   ”，char类型显示2个长度，而varchar类型显示4个长度。

​	**由于 CHAR 是固定长度的，所以它的处理速度比 VARCHAR 快得多，但是其缺点是浪费存储空间，程序需要对行尾空格进行处理，所以对于那些长度变化不大并且对查询速度有较高要求的数据可以考虑使用 CHAR 类型来存储。**

在 MySQL 中，不同的存储引擎对 CHAR 和 VARCHAR 的使用原则有所不同：

- 􏰃MyISAM存储引擎:建议使用固定长度的数据列代替可变长度的数据列。 
- MEMORY存储引擎:目前都使用固定长度的数据行存储，因此无论使用 CHAR 或 VARCHAR 列都没有关系。两者都是作为 CHAR 类型处理。 
- InnoDB 存储引擎:建议使用 VARCHAR 类型。**对于 InnoDB 数据表，内部的行存储格式没有区分固定长度和可变长度列(所有数据行都使用指向数据列值的头指针)，因此在本质上，使用固定长度的 CHAR 列不一定比使用可变长度 VARCHAR 列性能要好**。因而，主要的性能因素是数据行使用的存储总量。==由于 CHAR 平均占用的空间多于 VARCHAR，因此使用 VARCHAR 来最小化需要处理的数据行的存储总量和磁盘 I/O 是比较好的。==

#### 2.3.2 BINARY和VARBINARY类型

​	BINARY 和 VARBINARY 类似于 CHAR 和 VARCHAR，不同的是它们包含二进制字符串而不包含非二进制字符串。

#### 2.3.3 ENUM类型

​	ENUM 中文名称叫枚举类型，它的值范围需要在创建表时通过枚举方式显式指定，对 1~255 个成员的枚举需要 1 个字节存储;对于 255~65535 个成员，需要 2 个字节存储。最多允许有 65535 个成员。	

如：create table t (gender enum('M','F'));

​	INSERT INTO t VALUES('M'),('1'),('f'),(NULL);

结果：![image-20190110105507190](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190110105507190-7088907.png)

​	从上面的例子中，可以看出 **ENUM 类型是忽略大小写的，对'M'、'f'在存储的时候将它们都转成了大写，还可以看出对于插入不在 ENUM 指定范围内的值时，并没有返回警告，而是==插入了 enum('M','F')的第一值'M'==，**这点用户在使用时要特别注意。另外，==ENUM 类型只允许从值集合中选取单个值，而不能一次取多个值。==

#### 2.3.4 SET类型

​	Set 和 ENUM 类型非常类似，也是一个字符串对象，里面可以包含 0~64 个成员。根据
成员的不同，存储上也有所不同。
􏰃 1~8成员的集合，占1个字节。
􏰃 9~16成员的集合，占2个字节。
􏰃 17~24成员的集合，占3个字节。
􏰃 25~32成员的集合，占4个字节。
􏰃 33~64成员的集合，占8个字节。

​	Set 和 ENUM 除了存储之外，最主要的区别在于 Set 类型一次可以选取多个成员，而 ENUM则只能选一个。

![image-20190110105901728](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190110105901728-7089141.png)

#### 2.3.5 TEXT与BLOB

​	一般在保存少量字符串的时候，我们会选择 CHAR 或者 VARCHAR;而在保存较大文本时，通常会选择使用 TEXT 或者 BLOB，**二者之间的主要差别是 BLOB 能用来保存二进制数据，比如照片;而 TEXT 只能保存字符数据，比如一篇文章或者日记。**TEXT 和 BLOB 中有分别包括TEXT、MEDIUMTEXT、LONGTEXT 和 BLOB、MEDIUMBLOB、LONGBLOB 3 种不同的类型，**它们之间的主要区别是存储文本长度不同和存储字节不同**。

##### BLOB 和 TEXT 可能存在的一些常见问题：

- **BLOB和TEXT值会引起一些性能问题，特别是在执行了大量的删除操作时。**
  ​       删除操作会在数据表中留下很大的“空洞”，以后填入这些“空洞”的记录在插入的性能上会有影响。为了􏰀高性能，建议定期使用==OPTIMIZE TABLE==功能对这类表进行碎片整理，避免因为“空洞”导致性能问题。如：OPTIMIZE TABLE_name t。
- **可以使用合成的(Synthetic)索引来􏰀高大文本字段(BLOB 或 TEXT)的查询性能。**
  ​        简单来说，**合成索引就是根据大文本字段的内容建立一个散列值，并把这个值存储在单独的数据列中，接下来就可以通过检索散列值找到数据行了。**但是，要注意这种技术只能用于精确匹配的查询(散列值对于类似<或>=等范围搜索操作符是没有用处的)。==可以使用 MD5()函数生成散列值，也可以使用 SHA1()或 CRC32()，或者使用自己的应用程序逻辑来计算散列值。==请记住数值型散列值可以很高效率地存储。同样，**如果散列算法生成的字符串带有尾部空格，就不要把它们存储在 CHAR 或 VARCHAR 列中，它们会受到尾部空格去除的影响。**合成的散列索引对于那些 BLOB 或 TEXT 数据列特别有用。用散列标识符值查找的速度比搜索BLOB 列本身的速度快很多。
  - 如：create table t (id varchar(100),context blob,hash_value varchar(40));![image-20190110213249706](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190110213249706-7127169.png)
  - ![image-20190110213339985](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190110213339985-7127220.png)								   		   

上面的例子展示了合成索引的用法，由于这种技术只能用于精确匹配，在一定程度上减少 I/O，从而􏰀高查询效率。**如果需要对 BLOB 或者 CLOB 字段进行模糊查询，MySQL 􏰀供了前缀索引，也就是只为字段的前 n 列创建索引。**![image-20190110213510733](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190110213510733-7127310.png)

​	可以发现，对 context 前 100 个字符进行模糊查询，就可以用到前缀索引。**==请注意，这里的查询条件中，“%”不能放在最前面，否则索引将不会被使用。==**

- **在不必要的时候避免检索大型的 BLOB 或 TEXT 值。**
  例如，SELECT * 查询就不是很好的想法，除非能够确定作为约束条件的 WHERE 子句只会找 到所需要的数据行。否则，很可能毫无目的地在网络上传输大量的值。这也是 BLOB 或 TEXT 标识符信息存储在合成的索引列中对用户有所帮助的例子。**用户可以搜索索引列，决定需要 的哪些数据行，然后从符合条件的数据行中检索 BLOB 或 TEXT 值。** 

- **把 BLOB 或 TEXT 列分离到单独的表中。** 

  在某些环境中，如果把这些数据列移动到第二张数据表中，可以把原数据表中的数据列转换为固定长度的数据行格式，那么它就是有意义的。这会减少主表中的碎片，可以得到固定长 度数据行的性能优势。它还可以使主数据表在运行 SELECT * 查询的时候不会通过网络传输 大量的 BLOB 或 TEXT 值。 

## 3.运算符

### 1 算术运算符

​	MySQL 支持的算术运算符包括加、减、乘、除和模运算。它们是最常使用、最简单的一类运算符。	

![image-20190110110158402](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190110110158402-7089318.png)

/ 运算符用一个值除以另一个值得到商，可以是小数

􏰁% 运算符用一个值除以另外一个值得到余数。

**除法运算和模运算中，如果除数为 0，将是非法除数，返回结果为 NULL**

对于模运算，还有另外一种表达方式，使用 MOD(a,b)函数与 a%b 效果一样

### 2 比较运算符

​	当使用 SELECT 语句进行查询时，MySQL允许用户对表达式的左边操作数和右边操作数进行比较，比较结果为真，则返回 1，为假则返回 0，比较结果不确定则返回 NULL。	

![image-20190110110355981](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190110110355981-7089436.png)

​	比较运算符可以用于比较数字、字符串和表达式。数字作为浮点数比较，而字符串以不区分大小写的方式进行比较。	

​	“<=>”安全的等于运算符，和“=”类似，在操作数相等时值为 1，不同之处在于即使操作的值为 NULL 也可以正确比较。

​	“BETWEEN”运算符的使用格式为“a BETWEEN min AND max”，当 a 大于等于 min 并且小于等于 max，则返回值为 1，否则返回 0;当操作数 a、min、max 类型相同时，此表达式等价于(a>=min and a<=max)，当操作数类型不同时，比较时会遵循类型转换原则进行转换后，再进行比较运算。

​	**“LIKE”运算符的使用格式为“a LIKE %123%”,当a中含有字符串“123”时，则返回 值为 1，否则返回 0。** 

​	“REGEXP”运算符的使用格式为“str REGEXP str_pat”,当 str 字符串中含有 str_pat 相匹配的字符串时，则返回值为 1，否则返回 0。 

##### 数据类型的选择原则：

- 􏰃  **对于字符类型，要根据存储引擎来进行相应的选择。** 
- 􏰃  **对精度要求较高的应用中，建议使用定点数来存储数值，以保证结果的准确性。** 
- **􏰃  对含有 TEXT 和 BLOB 字段的表，如果经常做删除和修改记录的操作要定时执行 OPTIMIZE TABLE 功能对表进行碎片整理。** 
- **􏰃  日期类型要根据实际需要选择能够满足应用的最小存储的日期类型。** 

### 3 逻辑运算符

​	逻辑运算符又称为布尔运算符，用来确认表达式的真和假。

| 运算符     | 作用     |
| ---------- | -------- |
| NOT 或!    | 逻辑非   |
| AND 或&&   | 逻辑与   |
| OR 或 \|\| | 逻辑或   |
| XOR        | 逻辑异或 |

​	“**XOR”表示逻辑异或。当任意一个操作数为 NULL 时，返回值为 NULL。对于非 NULL 的操作数，如果两个的逻辑真假值相异，则返回结果 1;否则返回 0。**

### 4 位运算符

​	位运算是将给定的操作数转化为二进制后，对各个操作数每一位都进行指定的逻辑运算，得到的二进制结果转换为十进制数后就是位运算的结果。	![image-20190110111133716](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190110111133716-7089893.png)

### 5 运算符的优先级![image-20190110111331302](image-20190110111331302.png)

## 4.常用函数

### 1 字符串函数

| 函数                  | 功能                                                         |
| --------------------- | ------------------------------------------------------------ |
| CANCAT(S1,S2,...Sn)   | 连接 S1,S2,...Sn 为一个字符串                                |
| INSERT(str,x,y,instr) | 将字符串 str 从第 x 位置开始，y 个字符长的子串替换为字符串 instr |
| LOWER(str)            | 将字符串 str 中所有字符变为小写                              |
| UPPER(str)            | 将字符串 str 中所有字符变为大写                              |
| LEFT(str ,x)          | 返回字符串 str 最左边的 x 个字符  ，如果第二个参数是 NULL，那么将不返回任何字符串。下面的	RIGHT(str,x)同理 |
| RIGHT(str,x)          | 返回字符串 str 最右边的 x 个字符                             |
| LPAD(str,n ,pad)      | 用字符串 pad 对 str 最左边进行填充，直到长度为 n 个字符长度  |
| RPAD(str,n,pad)       | 用字符串 pad 对 str 最右边进行填充，直到长度为 n 个字符长度  |
| LTRIM(str)            | 去掉字符串 str 左侧的空格                                    |

如：把字符串“beijing2008you”中的从第 12 个字符开始以后的 3 个字符替换成“me”。

​	select INSERT('beijing2008you',12,3, 'me') ;	输出：beijing2008me

​	显示了字符串“beijing”加空格进行过滤后的结果。

​	select ltrim(' |beijing'),rtrim('beijing| ');		输出：|beijing和beijing|

### 2 数值函数

| 函数          | 功能                                                         |
| ------------- | ------------------------------------------------------------ |
| ABS(x)        | 返回 x 的绝对值                                              |
| CEIL(x)       | 返回大于 x 的最大整数值                                      |
| FLOOR(x)      | 返回小于 x 的最大整数值                                      |
| MOD(x，y)     | 返回 x/y 的模                                                |
| RAND()        | 返回0到1内的随机值                                           |
| ROUND(x,y)    | 返回参数 x 的四舍五入的有 y 位小数的值，如果是整数，将会保留 y 位数量的 0;如果不写 y，则默认 y 为 0，即将 x 四舍五入后取整。适合于将所有数字保留同样小数位的情况。 |
| TRUNCATE(x,y) | 返回数字 x 截断为 y 位小数的结果                             |

如：需要产生 0~100 内的任意随机整数

​	select ceil(100*rand()),ceil(100*rand());

##### 注意 TRUNCATE 和 ROUND 的区别在于 TRUNCATE 仅仅是截断，而不进行四舍五入。

### 3 日期和时间函数

| 函数                              | 功能                                                         |
| --------------------------------- | ------------------------------------------------------------ |
| CURDATE()                         | 返回当前日期，只包含年月日                                   |
| CURTIME()                         | 返回当前时间，只包含时分秒                                   |
| NOW()                             | 返回当前的日期和时间，年月日时分秒都包含                     |
| UNIX_TIMESTAMP(date)              | 返回日期 date 的 UNIX 时间戳                                 |
| FROM_UNIXTIME                     | 返回 UNIX 时间戳的日期值                                     |
| WEEK(date)                        | **返回日期 date 为一年中的第几周**                           |
| YEAR(date)                        | 返回日期 date 的年份                                         |
| HOUR(time)                        | 返回 time 的小时值                                           |
| MINUTE(time)                      | 返回 time 的分钟值                                           |
| MONTHNAME(date)                   | 返回 date 的月份名                                           |
| DATE_FORMAT(date,fmt)             | 返回按字符串 fmt 格式化日期 date 值，此函数能够按指定的格式显示日期，可以用到的格式符如下表 |
| DATE_ADD(date,INTERVAL expr type) | 返回一个日期或时间值加上一个时间间隔的时间值                 |
| DATEDIFF(expr,expr2)              | 返回起始时间 expr 和结束时间 expr2 之间的天数                |

![image-20190110152815637](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190110152815637-7105295.png)

![image-20190110152827674](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190110152827674-7105307.png)

### 4 流程函数

| 函数                                                       | 功能                                                  |
| ---------------------------------------------------------- | ----------------------------------------------------- |
| IF(value,t f)                                              | 如果 value 是真，返回 t;否则返回 f                    |
| IFNULL(value1,value2)                                      | 如果 value1 不为空返回 value1，否则返回 value2        |
| CASE WHEN [value1] THEN[result1]...ELSE[default]END        | 如果 value1 是真，返回 result1，否则返回 default      |
| CASE [expr] WHEN [value1] THEN[result1]...ELSE[default]END | 如果 expr 等于 value1，返回 result1，否则返回 default |

### 5 其他常用函数

| 函数           | 功能                      |
| -------------- | ------------------------- |
| DATABASE()     | 返回当前数据库名          |
| VERSION()      | 返回当前数据库版本        |
| USER()         | 返回当前登录用户名        |
| INET_ATON(IP)  | 返回 IP 地址的数字表示    |
| INET_NTOA(num) | 返回数字代表的 IP 地址    |
| PASSWORD(str)  | 返回字符串 str 的加密版本 |
| MD5()          | 返回字符串 str 的 MD5 值  |

​	通过select加对应的函数名称，即可以得到相应的返回值。

# 二、开发篇

## 1.表类型(存储引擎)的选择

### 1 MySQL存储引擎概述

​	MySQL 5.0 支持的存储引擎包括 MyISAM、InnoDB、BDB、MEMORY、MERGE、EXAMPLE、NDB Cluster、ARCHIVE、CSV、BLACKHOLE、FEDERATED等，其中InnoDB和BDB􏰀供事务安全表，其他存储引擎都是非事务安全表。	

​	默认情况下，创建新表不指定表的存储引擎，则新表是默认存储引擎的，如果需要修改默认的存储引擎，则可以在参数文件中设置 default-table-type。	

查看当前的默认存储引擎：show variables like 'table_type';

### 2 各种存储引擎的特性

##### 常用存储引擎的对比

| 特点           | MyISAM | InnoDB | MEMORY | MERGE | NDB  |
| -------------- | ------ | ------ | ------ | ----- | ---- |
| 存储限制       | 有     | 64TB   | 有     | 没有  | 有   |
| 事务安全       |        | 支持   |        |       |      |
| 锁机制         | 表锁   | 行锁   | 表锁   | 表锁  | 行锁 |
| B 树索引       | 支持   | 支持   | 支持   | 支持  | 支持 |
| 哈希索引       |        |        | 支持   |       | 支持 |
| 全文索引       | 支持   |        |        |       |      |
| 集群索引       |        | 支持   |        |       |      |
| 数据缓存       |        | 支持   | 支持   |       | 支持 |
| 索引缓存       | 支持   | 支持   | 支持   | 支持  | 支持 |
| 数据可压缩     | 支持   |        |        |       |      |
| 空间使用       | 低     | 高     | N/A    | 低    | 低   |
| 内存使用       | 低     | 高     | 中等   | 低    | 高   |
| 批量插入的速度 | 高     | 低     | 高     | 高    | 高   |
| 支持外键       |        | 支持   |        |       |      |

#### 2.1 MyISAM

​	**MyISAM 是 MySQL 的默认存储引擎(5.7之后默认引擎改为InnoDB)。**MyISAM 不支持事务、也不支持外键，其优势是访问的速度快，对事务完整性没有要求或者以 SELECT、INSERT 为主的应用基本上都可以使用这个引擎来创建表。	

​	每个 MyISAM 在磁盘上存储成 3 个文件，其文件名都和表名相同，但扩展名分别是:frm(存储表定义);
MYD(MYData，存储数据);MYI (MYIndex，存储索引)。	

**数据文件和索引文件可以放置在不同的目录，平均分布 IO，获得更快的速度。**

​	MyISAM 的表又支持 3 种不同的存储格式，分别是:静态(固定长度)表; 动态表;压缩表。

​	其中，静态表是默认的存储格式。静态表中的字段都是非变长字段，这样每个记录都是 固定长度的，这种存储方式的优点是存储非常迅速，容易缓存，出现故障容易恢复;缺点是 占用的空间通常比动态表多。静态表的数据在存储的时候会按照列的宽度定义补足空格，但 是在应用访问的时候并不会得到这些空格，这些空格在返回给应用之前已经去掉。 

​	动态表中包含变长字段，记录不是固定长度的，这样存储的优点是占用的空间相对较少，但是频繁地更新删除记录会产生碎片，需要定期执行OPTIMIZE TABLE语句或myisamchk -r命令来改善性能，并且出现故障的时候恢复相对比较困难。

​	压缩表由 myisampack 工具创建，占据非常小的磁盘空间。因为每个记录是被单独压缩的，所以只有非常小的访问开支。

#### 2.2 InnoDB	

​	**InnoDB 存储引擎􏰀供了具有提􏰀交、回滚和崩溃恢复能力的事务安全。但是对比 MyISAM的存储引擎，InnoDB 写的处理效率差一些并且会占用更多的磁盘空间以保留数据和索引。**	

##### 1、自动增长列

​	**InnoDB 表的自动增长列可以手工插入，但是==插入的值如果是空或者 0，则实际插入的将是自动增长后的值。==**	

​	可以通过**“ALTER TABLE *** AUTO_INCREMENT = n;”语句强制设置自动增长列的初识值**， 默认从 1 开始，但是该强制的默认值是保留在内存中的，如果该值在使用之前数据库重新启 动，那么这个强制的默认值就会丢失，就需要在数据库启动以后重新设置。 

​	可以使用 LAST_INSERT_ID()查询当前线程最后插入记录使用的值。如果一次插入了多条记录，那么返回的是第一条记录使用的自动增长值。

​	**对于 InnoDB 表，自动增长列必须是索引。如果是组合索引，也必须是组合索引的第一列**，但是对于 MyISAM 表，自动增长列可以是组合索引的其他列，这样插入记录后，自动增长列是按照组合索引的前面几列进行排序后递增的。	

##### 2、外键约束

​	MySQL 支持外键的存储引擎只有 InnoDB，在创建外键的时候，要求父表必须有对应的索引，子表在创建外键的时候也会自动创建对应的索引。	

当某个表被其他表创建了外键参照，那么该表的对应索引或者主键禁止被删除。

##### 3、存储方式

InnoDB 存储表和索引有以下两种方式。 

- 􏰃 ==使用共享表空间存储==，这种方式创建的表的表结构保存在.frm文件中，数据和索引保存在innodb_data_home_dir 和 innodb_data_file_path 定义的表空间中，可以是多个文件。 
- 􏰃  ==使用多表空间存储==，这种方式创建的表的表结构仍然保存在.frm文件中，但是每个表的数据和索引单独保存在.ibd 中。如果是个分区表，则每个分区对应单独的.ibd 文件，文件名是“表名+分区名”，可以在创建分区的时候指定每个分区的数据文件 的位置，以此来将表的 IO 均匀分布在多个磁盘上。 

#### 2.3 MEMORY

​	MEMORY 存储引擎使用存在内存中的内容来创建表。每个 MEMORY 表只实际对应一个磁盘文件，格式是.frm。MEMORY 类型的表访问非常得快，因为它的数据是放在内存中的，并且默认使用 HASH 索引，但是一旦服务关闭，表中的数据就会丢失掉。	

​	MEMORY 类型的存储引擎主要用在那些内容变化不频繁的代码表，或者作为统计操作的中间结果表，便于高效地对中间结果进行分析并得到最终的统计结果。对 MEMORY 存储引擎的表进行更新操作要谨慎，因为数据并没有实际写入到磁盘中，所以一定要对下次重新启动服务后如何获得这些修改后的数据有所考虑。	

#### 2.4 MERGE

​	MERGE 存储引擎是一组 MyISAM 表的组合，这些 MyISAM 表必须结构完全相同，MERGE 表本身并没有数据，对 MERGE 类型的表可以进行查询、更新、删除的操作，这些操作实际 上是对内部的实际的 MyISAM 表进行的。对于 MERGE 类型表的插入操作，是通过 INSERT_METHOD 子句定义插入的表，可以有 3 个不同的值，使用 FIRST 或 LAST 值使得插入 操作被相应地作用在第一或最后一个表上，不定义这个子句或者定义为 NO，表示不能对这 个 MERGE 表执行插入操作。 

​	可以对 MERGE 表进行 DROP 操作，这个操作只是删除 MERGE 的定义，对内部的表没有 任何的影响。 

​	MERGE 表在磁盘上保留两个文件，文件名以表的名字开始，一个.frm 文件存储表定义， 另一个.MRG 文件包含组合表的信息，包括 MERGE 表由哪些表组成、插入新的数据时的依据。 可以通过修改.MRG 文件来修改 MERGE 表，但是修改后要通过 FLUSH TABLES 刷新。 

### 3 如何选择合适的存储引擎

- 􏰁 MyISAM:默认的 MySQL 插件式存储引擎(5.1.6之前)。如果应用是以读操作和插入操作为主， 只有很少的更新和删除操作，并且对事务的完整性、并发性要求不是很高，那么选择这个存 储引擎是非常适合的。MyISAM 是在 Web、数据仓储和其他应用环境下最常使用的存储引擎 之一。 
- InnoDB:用于事务处理应用程序，支持外键。**如果应用对事务的完整性有比较高的 要求，在并发条件下要求数据的一致性，数据操作除了插入和查询以外，还包括很多的更新、 删除操作，那么 InnoDB 存储引擎应该是比较合适的选择。**InnoDB 存储引擎除了有效地降低由于删除和更新导致的锁定，还可以确保事务的完整提􏰀交(Commit)和回滚(Rollback)， 对于类似计费系统或者财务系统等对数据准确性要求比较高的系统，InnoDB 都是合适的选择。 
- MEMORY:将所有数据保存在RAM中，在需要快速定位记录和其他类似数据的环境 下，可􏰀供极快的访问。MEMORY 的缺陷是对表的大小有限制，太大的表无法 CACHE 在内 存中，其次是要确保表的数据可以恢复，数据库异常终止后表中的数据是可以恢复的。 MEMORY 表通常用于更新不太频繁的小表，用以快速得到访问结果。 
- MERGE:用于将一系列等同的MyISAM表以逻辑方式组合在一起，并作为一个对象 引用它们。MERGE 表的优点在于可以突破对单个 MyISAM 表大小的限制，并且通过将不同 的表分布在多个磁盘上，可以有效地改善 MERGE 表的访问效率。这对于诸如数据仓储等 VLDB 环境十分适合。 

## 2.字符集

### 2.1 MySQL字符集的设置

​	MySQL 的字符集和校对规则有 4 个级别的默认设置:服务器级、数据库级、表级和字段级。

#### 2.2.1 服务器字符集和校对规则

![image-20190110224610410](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190110224610410-7131570.png)

​	如果没有特别的指定服务器字符集，默认使用 latin1 作为服务器字符集。上面 3 种设置 的方式都只指定了字符集，没有指定校对规则，这样是使用该字符集默认的校对规则，如果 要使用该字符集的非默认校对规则，则需要在指定字符集的同时指定校对规则。 

**可以用“show variables like 'character_set_server';”命令查询当前服务器的字符集和校 对规则。** 

#### 2.2.2 字符集的修改步骤

​	如果在应用开始阶段没有正确的设置字符集，在运行一段时间以后才发现存在不能满足要求需要调整，又不想丢弃这段时间的数据，那么就需要进行字符集的修改。字符集的修改不能直接通过“alter database character set ***”或者“alter table tablename character set ***”命令进行，这两个命令都没有更新已有记录的字符集，而只是对新创建的表或者记录生效。已有记录的字符集调整，需要先将数据导出，经过适当的调整重新导入后才可完成。以下模拟的是将 latin1 字符集的数据库修改成 GBK 字符集的数据库的过程。

(1)导出表结构:	mysqldump -uroot -p --default-character-set=gbk -d databasename> createtab.sql

**​	其中--default-character-set=gbk 表示设置以什么字符集连接，-d 表示只导出表结构， 不导出数据。** 

(2)手工修改 createtab.sql 中表结构定义中的字符集为新的字符集。
(3)确保记录不再更新，导出所有记录。

​	mysqldump -uroot -p --quick --no-create-info --extended-insert --default-character-set=latin1 databasename> data.sql	

􏰁 --quick:该选项用于转储大的表。它强制mysqldump从服务器一次一行地检索表中 的行而不是检索所有行，并在输出前将它缓存到内存中。
 􏰁 --extended-insert:使用包括几个VALUES列表的多行INSERT语法。这样使转储文件 更小，重载文件时可以加速插入。 

􏰁 --no-create-info:不写重新创建每个转储表的 CREATE TABLE 语句。
 􏰁 --default-character-set=latin1:按照原有的字符集导出所有数据，这样导出的文件中， 所有中文都是可见的，不会保存成乱码。 

(4)打开 data.sql，将 SET NAMES latin1 修改成 SET NAMES gbk。 

(5)使用新的字符集创建新的数据库。 

​	create database databasename default charset gbk;

(6)创建表，执行 createtab.sql。
 	mysql -uroot -p databasename < createtab.sql 

(7)导入数据，执行 data.sql。 mysql -uroot -p databasename < data.sql 

**注意**:选择目标字符集的时候，要注意最好是源字符集的超级，或者确定比源字符集的字库更大，否则如果目标字符集的字库小于源字符集的字库，那么目标字符集中不支持的字符倒入后会变成乱码，丢失一部分数据。例如，GBK 字符集的字库大于 GB2312 字符集，那么 GBK 字符集的数据，如果导入 GB2312 数据库中，就会丢失 GB2312 中不支持的那部分汉字的数据。

## 3.索引的设计和使用

### 1 索引概述

​	所有 MySQL 列类型都可以被索引，对相关列使用索引是提􏰀高 SELECT 操作性能的最佳途 径。**根据存储引擎可以定义每个表的最大索引数和最大索引长度，每种存储引擎(如 MyISAM、 InnoDB、BDB、MEMORY 等)对每个表至少支持 16 个索引，总索引长度至少为 256 字节。 大多数存储引擎有更高的限制。** 

​	**MyISAM 和 InnoDB 存储引擎的表默认创建的都是 BTREE 索引。**MySQL 支持前缀索引，即对索引字段的前 N 个字符创建索引。**前缀索引的长度跟存 储引擎相关，**对于 MyISAM 存储引擎的表，索引的前缀长度可以达到 1000 字节长，而对于 InnoDB 存储引擎的表，索引的前缀长度最长是 767 字节。请注意前缀的限制应以字节为单 位进行测量，而CREATE TABLE语句中的前缀长度解释为字符数。在为使用多字节字符集的 列指定前缀长度时一定要加以考虑。 

​	MySQL 中还支持全文本(FULLTEXT)索引，该索引可以用于全文搜索，只限于 CHAR、VARCHAR 和 TEXT列。索引总是对整个列进行的，不支持局部(前缀)索引。 

​	也可以为空间列类型创建索引，但是只有 MyISAM 存储引擎支持空间类型索引，且索引 的字段必须是非空的。 

​	默认情况下，MEMORY 存储引擎使用 HASH 索引，但也支持 BTREE 索引。 索引在创建表的时候可以同时创建，也可以随时增加新的索引。创建新索引的语法为: 

![image-20190110230852063](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190110230852063-7132932.png)

如：为 city 表创建了 10 个字节的前缀索引：create index cityname on city (city(10));

索引的删除语法为: 

​	DROP INDEX index_name ON tbl_name 

### 2 设计索引的原则

- **搜索的索引列，不一定是所要选择的列。**换句话说，==最适合索引的列是出现在 WHERE 子句中的列，或连接子句中指定的列，而不是出现在 SELECT 关键字后的选择列表中的列==。 
- **使用惟一索引。考虑某列中值的分布。**索引的列的基数越大，索引的效果越好。例如，存放出生日期的列具有不同值，很容易区分各行。而用来记录性别的列，只含有“ M” 和“F”，则对此列进行索引没有多大用处，因为不管搜索哪个值，都会得出大约一半的行。 
- **使用短索引。**如果对字符串列进行索引，应该指定一个前缀长度，只要有可能就应 该这样做。例如，如果有一个 CHAR(200)列，**如果在前 10 个或 20 个字符内，多数值是惟一 的，那么就不要对整个列进行索引。**对前 10 个或 20 个字符进行索引能够节省大量索引空间， 也可能会使查询更快。==较小的索引涉及的磁盘 IO 较少，较短的值比较起来更快。更为重要 的是，对于较短的键值，索引高速缓存中的块能容纳更多的键值，因此，MySQL 也可以在 内存中容纳更多的值。这样就增加了找到行而不用读取索引中较多块的可能性。== 
- **利用最左前缀。**在创建一个n列的索引时，实际是创建了MySQL可利用的n个索引。 多列索引可起几个索引的作用，**因为可利用索引中最左边的列集来匹配行。这样的列集称为 最左前缀。** 
- **不要过度索引。**不要以为索引“越多越好”，什么东西都用索引是错误的。每个额 外的索引都要占用额外的磁盘空间，并降低写操作的性能。在修改表的内容时，索引必须进 行更新，有时可能需要重构，因此，索引越多，所花的时间越长。如果有一个索引很少利用 或从不使用，那么会不必要地减缓表的修改速度。此外，MySQL 在生成一个执行计划时， 要考虑各个索引，这也要花费时间。创建多余的索引给查询优化带来了更多的工作。索引太 多，也可能会使 MySQL 选择不到所要使用的最好索引。只保持所需的索引有利于查询优化。 
- 对于 InnoDB 存储引擎的表，记录默认会按照一定的顺序保存，如果有明确定义的主键，则按照主键顺序保存。**如果没有主键，但是有唯一索引，那么就是按照唯一索引的顺序 保存。**如果既没有主键又没有唯一索引，那么表中会自动生成一个内部列，按照这个列的顺 序保存。==按照主键或者内部列进行的访问是最快的，==所以 InnoDB 表尽量自己指定主键，当表中同时有几个列都是唯一的，都可以作为主键的时候，**要选择最常作为访问条件的列作为 主键，􏰀**高查询的效率。另外，还需要注意，InnoDB 表的普通索引都会保存主键的键值， 所以主键要尽可能选择较短的数据类型，可以有效地减少索引的磁盘占用，􏰀高索引的缓存 效果。 

### 3 BTREE索引与HASH索引

​	MEMORY 存储引擎的表可以选择使用 BTREE 索引或者 HASH 索引，两种不同类型的索引各有其不同的适用范围。HASH 索引有一些重要的特征需要在使用的时候特别注意，如下所示。

- 􏰃  **只用于使用=或<=>操作符的等式比较。** 

- **􏰃  优化器不能使用 HASH 索引来加速 ORDER BY 操作。** 

- 􏰃  **MySQL不能确定在两个值之间大约有多少行。如果将一个MyISAM表改为HASH索引的 MEMORY 表，会影响一些查询的执行效率。** 

- 􏰃  **只能使用整个关键字来搜索一行。**

  对于 BTREE 索引，当使用>、<、>=、<=、BETWEEN、!=或者<>，或者 LIKE 'pattern'(其中'pattern'不以通配符开始)操作符时，都可以使用相关列上的索引。 

下列范围查询适用于 BTREE 索引和 HASH 索引:

SELECT * FROM t1 WHERE key_col = 1 OR key_col IN (15,18,20); 

下列范围查询只适用于 BTREE 索引:
 ![image-20190111105224960](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190111105224960-7175145.png)

##### 	当对索引字段进行范围查询的时候，只有 BTREE 索引可以通过索引访问， HASH 索引实际上是全表扫􏰄。

​	当使用 MEMORY 表的时候，如果是默认创建的 HASH索引，就要注意 SQL 语句的编写，确保可以使用上索引，**如果一定要使用范围查询，那么在创建索引的时候，就应该选择创建成 BTREE 索引。**

## 4.视图

### 1 什么是视图 

​	视图(View)是一种**虚拟存在的表**，对于使用视图的用户来说基本上是透明的。视图并不在数据库中实际存在，**行和列数据来自定义视图的查询中使用的表，并且是在使用视图时动态生成的。** 

  视图相对于普通的表的优势主要包括以下几项。

- 􏰃  简单:使用视图的用户完全不需要关心后面对应的表的结构、关联条件和筛选条件，**对用户来说已经是过滤好的复合条件的结果集。**
- 􏰃  安全:使用视图的用户只能访问他们被允许查询的结果集，对表的权限管理并不能限制到某个行某个列，但是通过视图就可以简单的实现。 
- 􏰃  数据独立:一旦视图的结构确定了，可以屏蔽表结构变化对用户的影响，源表增加列对视图没有影响;源表修改列名，则可以通过修改视图来解决，不会造成对访问 者的影响。 

### 2 视图操作 

#### 2.1 创建或者修改视图 

![image-20190111110750532](https://raw.githubusercontent.com/JDawnF/learning_note/master/images/image-20190111110750532-7176070.png)

​	MySQL 视图的定义有一些限制，例如，**在 FROM 关键字后面不能包含子查询，**这和其他数 据库是不同的，如果视图是从其他数据库迁移过来的，那么可能需要因此做一些改动，可以 将子查询的内容先定义成一个视图，然后对该视图再创建视图就可以实现类似的功能了。 

视图的可更新性和视图中查询的定义有关系，以下类型的视图是不可更新的。

- 包含以下关键字的 SQL 语句:聚合函数(SUM、MIN、MAX、COUNT 等)、DISTINCT、GROUP BY、HAVING、UNION 或者 UNION ALL。
- 常量视图。
- SELECT中包含子查询。
- JION。
- FROM一个不能更新的视图。
- WHERE字句的子查询引用了FROM字句中的表。

**WITH [CASCADED | LOCAL] CHECK OPTION 决定了是否允许更新数据使记录不再满足视图的条件。**这个选项与 Oracle 数据库中的选项是类似的，其中:􏰁LOCAL是只要满足本视图的条件就可以更新; CASCADED则是必须满足所有针对该视图的所有视图的条件才可以更新。
**如果没有明确是 LOCAL 还是 CASCADED，则默认是 CASCADED。**

#### 2.2 删除视图

用户可以一次删除一个或者多个视图，前􏰀是必须有该视图的 DROP 权限。

​	**DROP VIEW [IF EXISTS] view_name [, view_name] ...[RESTRICT | CASCADE]**

#### 2.3 查看视图

​	从MySQL 5.1版本开始，使用SHOW TABLES命令的时候不仅显示表的名字，同时也会显示视图的名字，而不存在单独显示视图的 SHOW VIEWS 命令。 

## 5.存储过程和函数

### 1 什么是存储过程和函数

​	**存储过程和函数是事先经过编译并存储在数据库中的一段 SQL 语句的集合，**调用存储过程和函数可以简化应用开发人员的很多工作，减少数据在数据库和应用服务器之间的传输，对于􏰀高数据处理的效率是有好处的。
​	==存储过程和函数的区别在于函数必须有返回值，而存储过程没有，存储过程的参数可以使用IN、OUT、INOUT 类型，而函数的参数只能是 IN 类型的。==如果有函数从其他类型的数据库迁移到 MySQL，那么就可能因此需要将函数改造成存储过程。

### 2 存储过程和函数的相关操作

​	在对存储过程或函数进行操作时，需要首先确认用户是否具有相应的权限。

#### 2.1 创建、修改存储过程或者函数

**创建、修改存储过程或者函数的语法:**

```sql
CREATE PROCEDURE sp_name ([proc_parameter[,...]]) [characteristic ...] routine_body
CREATE FUNCTION sp_name ([func_parameter[,...]]) RETURNS type
[characteristic ...] routine_body
proc_parameter:
[ IN | OUT | INOUT ] param_name type		--表示输入，输出，输出和输出参数
func_parameter: param_name type
type:
Any valid MySQL data type
characteristic: LANGUAGE SQL
| [NOT] DETERMINISTIC
| { CONTAINS SQL | NO SQL | READS SQL DATA | MODIFIES SQL DATA } | SQL SECURITY { DEFINER | INVOKER }
| COMMENT 'string'
routine_body:
Valid SQL procedure statement or statements
ALTER {PROCEDURE | FUNCTION} sp_name [characteristic ...]
characteristic:
{ CONTAINS SQL | NO SQL | READS SQL DATA | MODIFIES SQL DATA }
| SQL SECURITY { DEFINER | INVOKER } | COMMENT 'string'
```

调用过程的语法如下：	**CALL  sp_name([parameter[,...]])**

​	MySQL 的存储过程和函数中允许包含 DDL 语句，也允许在存储过程中执行􏰀交(Commit，即确认之前的修改)或者回滚(Rollback，即放弃之前的修改)，但是存储过程和函数中不允许执行 LOAD DATA INFILE 语句。此外，存储过程和函数中可以调用其他的过程或者函数。

下面对 characteristic 特征值的部分进行简单的说明。 

- 􏰃  LANGUAGESQL:说明下面过程的BODY是使用SQL语言编写，这条是系统默认的， 为今后 MySQL 会支持的除 SQL 外的其他语言支持的存储过程而准备。 

- 􏰃  [NOT]DETERMINISTIC:DETERMINISTIC确定的，即每次输入一样输出也一样的程序， NOT DETERMINISTIC 非确定的，默认是非确定的。当前，这个特征值还没有被优化 程序使用。 

- 􏰃  { CONTAINS SQL | NO SQL | READS SQL DATA | MODIFIES SQL DATA }:这些特征值􏰀供 

  子程序使用数据的内在信息，这些特征值目前只是􏰀供给服务器，并没有根据这些 特征值来约束过程实际使用数据的情况。CONTAINS SQL表示子程序不包含读或写 数据的语句。NO SQL 表示子程序不包含 SQL 语句。READS SQL DATA 表示子程序包 含读数据的语句，但不包含写数据的语句。MODIFIES SQL DATA 表示子程序包含写 数据的语句。如果这些特征没有明确给定，默认使用的值是 CONTAINS SQL。 

- 􏰃  SQL SECURITY { DEFINER | INVOKER }:可以用来指定子程序该用创建子程序者的许 可来执行，还是使用调用者的许可来执行。默认值是 DEFINER。 

- 􏰃  COMMENT 'string':存储过程或者函数的注释信息。

#### 2.2 删除存储过程或者函数

​	一次只能删除一个存储过程或者函数，删除存储过程或者函数需要有该过程或者函数的ALTER ROUTINE 权限，具体语法如下:	**DROP {PROCEDURE | FUNCTION} [IF EXISTS] sp_name**

#### 2.3 查看存储过程或者函数

##### 1.查看存储过程或者函数的状态

SHOW {PROCEDURE | FUNCTION} STATUS [LIKE 'pattern']

##### 2.查看存储过程或者函数的定义

SHOW CREATE {PROCEDURE | FUNCTION} sp_name

##### 3、通过查看information_schema. Routines了解存储过程和函数的信息









































































































































































































































































