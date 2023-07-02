# MySQL

## MySQL的基本概念

- MySQL目前属于Oracle，是关系型数据库
- MySQL区别于其他关系型数据库很大的一个特点就是支持插件式的存储引擎，例如InnoDB,MyISAM,Memory等
- MySQL设计成C/S模型 mysql client , mysql server
- MySQL的服务器模型采用的是IO复用+可伸缩的线程池，是实现网络服务器的经典模型 select + 线程池
  - 为什么不使用高性能的epoll IO复用呢？
  - 因为磁盘IO速度较慢(业务处理较慢)，使用较低性能的select正好使性能相匹配
- MySQL默认的工作端口是3306



### MySQL的数据类型

**数值类型** 

|    整数类型    |   字节   |
| :------------: | :------: |
|    TINYINT     |    1     |
|    SMALLINT    |    2     |
|   MEDIUMINT    |    3     |
|  INT、INTEGER  |    4     |
|     BIGINT     |    8     |
| **浮点数类型** | **字节** |
|     FLOAT      |    4     |
|     DOUBLE     |    8     |

**常用的字符串类型**

char 

varchar

**日期时间类型**

| 日期时间类型 | 字节 |
| :----------: | :--: |
|     DATE     |  4   |
|   DATETIME   |  8   |
|  TIMESTAMP   |  4   |
|     TIME     |  3   |
|     YEAR     |  1   |

**enum和set**

这两个类型都是限制该字段只能取固定的值，但是枚举字段只能取一个唯一的值，而集合字段可以取任意个值

 

### MySQL完整性约束

1. 主键约 primary key
2. 自增键约束 auto_increment
3. 唯一键约束 unique
4. 非空约束 not null
5. 默认值约束 default
6. 外键约束 foreign key



## MySQL数据库设计

一个表对应一个实体，表与表之间的关系就是实体与实体之间的关系

**表设计原则**

1. 一对一：设计子表，子表中增加一列关联父表的主键

   ![](https://img1.imgtp.com/2023/06/26/SaMb4o4A.png)

2. 一对多：设计成子表，在子表中条件一列关联父表的主键

   ![](https://img1.imgtp.com/2023/06/26/SaMb4o4A.png)

3. 多对多：增加中间表

![](https://img1.imgtp.com/2023/06/26/I8iy2KQP.png)

### 数据库范式

**优点**

1. 减少数据冗余(最主要的好处，其他好处都是由此附带的)
2. 消除异常(插入异常、更新一样， 删除异常)
3. 让数据组织的更加和谐

数据库范式不是越高越好，范式越高，意味着表越多，多表联合查询的几率就越大，SQL的效率就越低



**第一范式(1NF)**

每一列保持原子特性。表中每一列都是基本数据项，不能够再进行分割，否则设计成为一对多的实体关系。例如表中的地址字段，可以再细分为省市区等不可再分割(即原子特性)的字段

例如：需要查询某个省份的订单，直接查询就行，不需要按照格式进行分割

**第二范式(2NF)**

属性完全依赖于主键-主要针对联合主键。非主属性完全依赖于主关键字，如果不是完全依赖主键，应该拆分成为新的实体，设计成为一对多的实体关系

例如：选课关系表selectCourse(学号，课程名称，成绩，学分)，其中学号与课程名称为联合主键，但是学分字段只与课程名称有关，与学号无关，相当于只依赖联合主键的其中一个字段，不适合第二范式。

**第三范式(3NF)**

属性不依赖于其他非主属性。要求一个数据库表中不包含已在其他表中已包含的非主关键信息

例如：学生关系表Student（学号，姓名，年龄，学院，学院地址）,学号为主键，但是学院地址只依赖学院，并不依赖主键学号，此外，学院表中包含了学院地址，不符合三范式。应该设计两张表，学生表与学院表，两个是一对多的关系。

一般关系型数据库满足第三范式就可以了

**BC范式(BCNF)**

每个表中只有一个候选键。简单来说，BC范式是在三范式基础上的一种特殊情况，即每个表中只有一个候选键(在一个数据库中每行的值都不相同，则可成为候选键)。

例如，学生表(学号，姓名，年龄，电话)中，每个同学的电话都是唯一的，可以设计为候选键单独建表存储

**第四范式**

消除表中的多值依赖。也就是说可以减少维护数据一致性的工作。

例如：学生表（学号，姓名，技能），有的学生技能描述"c++, mysql",有人描述"C++,MySQL"，虽然表达意思一致，但是数据不一致，解决办法就是将多值属性存放到一个新表(学号， 技能)



## 结构化查询语句SQL

1. DDL (Data Definition Languages）语句。数据定义语言，这些语句定义了不同的数据库、表、列、索引等数据库对象的定义。常用的语句关键字主要包括create、drop、alter等。
2. DML (Data Manipulation Language)语句。数据操纵语句，用于添加、删除、更新和查询数据库记录，并检查数据完整性，常用的语句关键字主要包括insert、delete、update和select等。
3. DcL (Data Control Language）语句。数据控制语句，用于控制不同的许可和访问级别的语句。这些语句定义了数据库、表、字段、用户的访问权限和安全级别。主要的语句关键字包括grant、revoke等。
   

### SQL语句的执行过程

用户写的每一条语句都是属于MySQL Client， 最终交给MySQL Server执行，**每一条sql语句都会执行下面的操作**。

1. client 与 server进行tcp三次握手

2. client发送sql到server上，server接收并处理，返回处理结果

3. server和client断开连接，tcp四次挥手

   

### 库操作

查询数据库

```mysql
show databases;
```

创建数据库

```mysql
create database chat;
```

删除数据库

```mysql
drop database chat;
```

选择数据库

```mysql
use mysql; 
```



### 表操作

```mysql
/* 创建表 */
CREATE TABLE USER(
	id INT UNSIGNED PRIMARY KEY AUTO_INCREMENT,  -- （）内的数字不是指定字节，而是显示的宽度, AUTO INCREMENT自增键
    name VARCHAR(50) UNIQUE NOT NULL,
    age TINYINT NOT NULL,
    sex enum('M', 'W') NOT NULL
) ENGINE=INNODB DEFAULT CHARSET='UTF8';
/* 
windows下mysql的配置文件路径：my.ini 
linux下 /etc/my.cnf
*/
USE SCHOOL;
SHOW TABLES;
DROP TABLE USER;
SHOW CREATE TABLE USER; 
```



### CURD操作

**insert增加**

```mysql
INSERT INTO USER(name, age, sex) VALUES ('zhangsan', 20, 'M');
INSERT INTO USER(name, age, sex) VALUES ('lisi', 21, 'M');
INSERT INTO USER(name, age, sex) VALUES ('feiyu', 22, 'W');
INSERT INTO USER(name, age, sex) VALUES ('anna', 20, 'M');
INSERT INTO USER(name, age, sex) VALUES ('ikun', 20, 'M');
INSERT INTO USER(name, age, sex) VALUES ('amagi', 20, 'W');

INSERT INTO USER(name, age, sex) VALUES ('zhangsan', 20, 'M'), ('lisi', 21, 'M'), 
										('feiyu', 22, 'W'), ('anna', 20, 'M'),
										('ikun', 20, 'M'), ('amagi', 20, 'W');
/*
 * 上面两种插入方式最终的结果是相同的
 * 但是第一种插入方式建立了6次tcp连接，下面建立了1次tcp连接
 */


```

**delete删除**

```mysql
delete from USER where id=1;
INSERT INTO USER(name, age, sex) VALUES ('zhangsan', 20, 'M');
delete from USER;
```

**undate更新**

```mysql
UPDATE USER SET age=age+1;
UPDATE USER SET age=age+1 WHERE id=3;
```

**select查询**

```mysql
/* 单表查询 */
SELECT * FROM USER;
SELECT name FROM USER;
SELECT name FROM USER WHERE age>21;
SELECT name FROM USER WHERE age>21 AND age<25;  
-- SELECT name FROM USER WHERE age BETWEEN 21 AND 25;
-- SELECT name FROM USER WHERE age IN(21, 25);
SELECT name FROM USER WHERE name LIKE 'ZHANG%'
SELECT DISTICT AGE FROM USER;  -- distinct去重
-- ALL表示查看重复值，默认进行去重
SELECT name, age, sex FROM USER WHERE age>=21 UNION ALL SELECT name, age, sex FROM USER WHERE sex='M';


/* 分页查询 */
SELECT * FROM USER LIMIT 3;  -- 偏移0
SELECT * FROM USER LIMIT 1, 3;  -- 偏移1
SELECT * FROM USER LIMIT 3 OFFSET 1;

/* 一个用于快速装填测试数据的存储过程 */
DELIMITER $
CREATE PROCEDURE addTEA(IN n INT)
BEGIN
DECLARE i INT;
SET i = 0;
WHILE i < n DO
INSERT INTO TEA VALUES(NULL, CONCAT(i+1, '@FYBF.COM'),i+1);
SET i = i + 1;
END WHILE;
END$

DELIMITER;
	
CALL addTEA(200);
```

- SELECT name FROM USER WHERE name='zhangsan';
  - 由于name字段加了索引，只会扫描一行就能够返回结果
  - SELECT name FROM USER WHERE age=20;会扫描整表



EXPLAIN SELECT name FROM USER WHERE name='zhangsan'

EXPLAIN用于查看sql语句的执行计划

当我们使用select查询一个非索引字段时，不加limit会扫描整表，浪费时间，使用limit能够节省时间

**limit分页与偏移**

1. 对于同样的偏移量来说，选择1个字段比选择多个字段效率更高

2. LIMIT分页的效率低在OFFSET偏移上，可以通过主键索引过滤掉偏移

3. 当我们只知道偏移量而不知道具体的索引时，可以通过**内连接**进行优化

   - ```mysql
     SELECT A.ID, A.EMAIL FROM TEA A INNER JOIN (SELECT ID FROM TEA LIMIT 15000, 10);
     ```

4. 

```mysql
SELECT * FROM USER LIMIT (PAGENO - 1) * PAGENUM, PAGENUM;  -- 查询指定页数的记录
-- LIMIT分页的效率低在OFFSET偏移上，可以通过主键索引过滤掉偏移，如下：
SELECT * FROM USER LIMIT 20 OFFSET 10000000;  -- 偏移一千万浪费时间
SELECT * FROM USER WHILE ID > 10000000 LIMIT 20 OFFSET 0;  -- id为主键，查找效率高
```

**order by排序**

asc升序

desc降序

排序的性能优化：



**group by分组**

通常与聚合函数一起使用，group by分组后会自动排序

group by后面的字段如果是非索引字段可能涉及到外排序与临时表，效率较低，但是使用索引字段的话效率就会高一点

```mysql
SELECT SEX FROM USER GROUP BY SEX;
SELECT COUNT(ID), SEX FROM USER GROUP BY SEX;
SELECT COUNT(ID), AGE FROM USER GROUP BY AGE HAVING AGE>20;  -- HAVING用于分组后的条件过滤
```



**试题：**

![1687943804174.png](https://img1.imgtp.com/2023/06/28/RawVo0jE.png)

1. 统计表中缴费的总比数与总金额
2. 给出一个sql，按照网点和日期统计每个网点每天的营业额，并进行倒序排序

```mysql
SELECT COUNT(*), SUM(amount) FROM bank_bill;
SELECT SUM(amount) FROM (SELECT * FROM bank_bill GROUP BY brno) GROUP BY date ORDER BY amount DESC;
```



## 连接查询

**连接查询**

- 内连接查询
- 外连接查询
  - 左连接查询
  - 右连接查询

### 内连接查询

1. ON A.UID=C.UID 区分大表和小表(数据量区分) ，小表永远是整表扫描，然后去大表搜索。从STUDENT表(小表)中取出所有A.UID，然后拿着这些UID去EXAME大表中搜索
2. 对于inner join 内连接，过滤条件写在where后面和on后面效果是一样的

```mysql
CREATE TABLE STUDENT(
	UID INT UNSIGNED PRIMARY KEY AUTO_INCREMENT,
    NAME VARCHAR(10) NOT NULL,
    AGE TINYINT UNSIGNED NOT NULL,
    SEX ENUM('M', 'W') NOT NULL
);

CREATE TABLE COURSE(
	CID INT UNSIGNED PRIMARY KEY AUTO_INCREMENT,
    CNAME VARCHAR(10) NOT NULL,
    CREDIT TINYINT UNSIGNED NOT NULL
);

CREATE TABLE EXAME(
    UID INT UNSIGNED NOT NULL,
	CID INT UNSIGNED NOT NULL,
    E_TIME DATE,
	SCORE FLOAT NOT NULL,
    PRIMARY KEY(UID, CID)
);

insert into STUDENT(NAME, AGE, SEX) values('zhangsan', 18, 'M'), ('gaoyang', 20, 'w'),
										  ('chenwei', 22, 'M'), ('linfeng', 21, 'w'),  ('liuxiang', 19, 'W');



insert into COURSE (CNAME , CREDIT) values('C++基础课程', 5), ( 'C++高级课程', 10),
										  ('C++项目开发', 8 ), ( 'C++算法课程', 12);
										  
insert into EXAME(UID, CID, E_TIME, SCORE) values
(1, 1, '2021-04-09', 99.0),
(1, 2, '2021-04-10', 80.0),
(2, 2, '2021-04-10', 90.0),
(2, 3, '2021-04-12', 85.0),
(3, 1, '2021-04-09', 56.0),
(3, 2, '2021-04-10', 93.0),
(3, 3, '2021-04-12', 89.0),
(3, 4, '2021-04-11', 100.0),
(4, 4, '2021-04-11', 99.0),
(5, 2, '2021-04-10', 59.0),
(5, 3, '2021-04-12', 94.0),
(5, 4, '2021-04-11', 95.0);

-- 查看zhangsan同学的课程成绩
/* 
 * ON A.UID=C.UID 区分大表和小表(数据量区分) ，小表永远是整表扫描，然后去大表搜索
 * 从STUDENT表(小表)中取出所有A.UID，然后拿着这些UID去EXAME大表中搜索
 */
SELECT A.UID, A.NAME, A.AGE, A.SEX, C.SCORE FROM STUDENT A INNER JOIN EXAME C 
	ON A.UID=C.UID WHERE A.NAME='zhangsan';

SELECT A.UID, A.NAME, A.AGE, A.SEX, B.CID, B.CNAME, B.CREDIT, C.SCORE
	FROM EXAME C
	INNER JOIN STUDENT A ON C.UID=A.UID
	INNER JOIN COURSE B ON C.CID=B.CID
	WHERE C.UID=1 AND C.CID=2;
```



### 外连接查询

#### 左连接查询

```mysql
-- 把LEFT这边的表中的数据全部显示，在右表中不存在响应数据则显示NULL
SELECT A.* FROM USER A LEFT JOIN EXAME B ON A.UID=B.UID;
```



#### 右连接查询

```mysql
-- 把RIGHT这边的表中的数据全部显示，在左表中不存在响应数据则显示NULL
SELECT A.* FROM USER A RIGHT JOIN EXAME B ON A.UID=B.UID;
```

当过滤条件写在where后面和on后面效果是不一样的。

当我们使用外连接查询带有一定的限制条件加到on的连接条件后面





## 存储引擎

存储引擎直接影响表的结构，数据，索引等等

不同的存储引擎数据的存储方式是不同的

InnoDB

- .frm（存储表的定义） 
- .ibd（存储数据和索引）

MyISAM

- .frm（存储表定义）
-  .MYD（MYData，存储数据） 
- .MYI （ MYIndex，存储索引）



## MySQL索引

当表中的数据量到达几十万甚至上百万的时候，SQL查询所花费的时间会很长，导致业务超时出错，此 时就需要用索引来加速SQL查询。 由于索引也是需要存储成索引文件的，因此对索引的使用也会涉及磁盘I/O操作。

如果索引创建过多， 使用不当，会造成SQL查询时，进行大量无用的磁盘I/O操作，降低了SQL的查询效率，适得其反，因此 掌握良好的索引创建原则非常重要！即索引也是磁盘IO事件

一次sql查询只能用到一个索引

1. 优点：提高查询效率
2. 缺点：索引并非越多越好，过多的索引会导致CPU使用率居高不下，由于数据的改变会造成索引文件的改动，过多的磁盘IO造成CPU负荷太重。

### 索引分类

物理上：

- 聚集索引
- 非聚集索引

逻辑上：

- 普通索引（二级索引）：没有限制条件，可以给任何类型的字段创建普通索引(创建新表&&已经存在的表，数量不限，一张表的一次sql查询，只能用一个**索引**，使用距离where最近的)

- 唯一性索引：使用UNIQUE修饰的字段，值不能够重复，主键索引就隶属于唯一性索引
- 主键索引：使用PRIMARY KEY会自动创建索引(InnoDB会，MyISAM不会)
- 单列索引：在一个字段上创建索引
- 多列索引：在表的多个字段上创建索引(联合主键生成的多列索引，多列索引必须使用到第一个列才能使用，如uid+cid,不使用uid这个列就无法使用多列索引)
- 全文索引：使用FULLTEXT参数可以设置全文索引，只支持CHAR,VARCHAR和TEXT类型的字段上，常用于数据量较大的字符串类型上，可以提高查询速度。



### 索引应用

**创建表的时候指定索引**

```sql
CREATE TABLE INDEX1(
    ID INT,
	NAME VARCHAR(20),
    SEX ENUM("F", "M"),
    INDEX(ID);  -- 普通索引
)
```

在已经创建的表上添加索引

```mysql
CREATE [UNIQUE] INDEX 索引名 ON 表名(属性名(length)) [ASC | DESC];
-- 这个length指定前length建立索引
```

删除索引

```mysql
DROP INDEX 索引名 ON 表名
```

### 索引的执行过程

使用explain查看执行过程



### 索引的优化

1. 给经常作为where过滤条件的字段考虑添加索引
2. 给字符串创建索引时，尽量减少作为索引的字符串长度
3. 当where后面的过滤条件涉及类型转换和MySQL的函数时，不会使用索引



### 索引的底层实现原理

> 数据库索引是存储在磁盘上的，当数据量大时，就不能把整个索引全部加载到内存，只能逐一加载每个磁盘块(**对应索引树的节点， 索引树高度越低，越矮胖，磁盘IO次数就少**)

> 通过select查询做整表搜索时，有索引时，MySQL的server会把索引文件加载进内存。索引文件(不同存储引擎)，索引的数据结构入手。



```mysql
-- 场景1，只有uid为主键
select * from student;  -- 整表搜索 不会遍历树，而是遍历有序链表

select * from student where uid = 5;  -- 等值查询，二分法搜索B+树

select * from student where uid < 5;  -- 范围搜索，遍历有序链表

select * from student where name-"zhang san";  -- 整表搜索

-- 场景2，uid作为主键， name创建了普通索引(二级索引)
select name from student where name = "zhangsan";  -- 二级索引树
select uid, name from student where name = "zhangsan";  -- 二级索引树

-- 回表，搜索name的二级索引树，找到zhangsan对应的主键uid，在拿uid=4回表在索引树上搜索uid=4那一行的记录
select * from student where name = "zhangsan";  

-- 场景三
-- 优化：把age和name作为多列索引就不会涉及外部排序,先按age排序，age相同使用name排序
select * from student where age = 10 order by name; 
explain select * from student where age = 10 order by name;  


```





### **B-树**(balance树）

> B树是平衡树，AVL是平衡二叉树。AVL的一个节点是左孩子，数据域，右孩子组成，但是B树的孩子在300~500(有300~500个指针域， -1个数据域)之间，极大的减少数的高度。进而减少了节点对应的磁盘块加载进入内存的次数,每加载一个节点就是一次磁盘IO，但是磁盘IO效率低。
>
> B树具有AVL树的特性，左节点值 < 根节点值 < 右节点值，因此我们使用二分搜索加快查找效率

![1688194287360.png](https://img1.imgtp.com/2023/07/01/WixZUsku.png)



**为什么MySQL(MyISAM和InnoDB)索引底层选择B+树而不是B树**

**缺点**

1. 索引+数据内容分散在不同节点上，距离节点近，搜索就快，反之就慢
2. 每一个非叶子节点，不仅存储索引，还要存储索引值所在的那一行的data数据，一个节点所能存放的索引Key的值的个数比只存储key的值的节点个数要少的多！(就是非叶子节点存放的key值过少)
3. B树不方便做范围搜索，整表搜索也不方便



### **B+树**

> 1. 每一个非叶子节点只存放key，不存储data(一个节点存放的key值更多，理论上讲层数会更低一些，搜索的效率更好一些)
> 2. 叶子节点上存储了所有的索引值和其对应的data(搜索每一个对需要跑到叶子节点上，这样每一行记录搜索的时间是非常平均的)
> 3. 叶子节点被串在一个链表中，形成了一个有序链表，如果要进行索引树的搜索或者整表搜索，直接遍历叶子节点的有序链表即可！或者做范围查询的时候，也直接遍历有序链表就行

![1688198530303.png](https://img1.imgtp.com/2023/07/01/lYNGXXLm.png)



### 辅助(二级)索引树

对于`InnoDB`存储引擎，底层数据结构也是B+树，不过节点中data里面存放的是所在记录行的主键值

对于`MyISAM`存储引擎，底层数据结构也是B+树，不过节点中data里面存放的还是数据在磁盘上的地址(主键索引树也存放数据的地址)。

主键索引树的索引不能重复，二级索引树可以重复

`聚集索引`：数据和索引一起存储

`非聚集所引`：数据和索引分开存储



### 哈希索引

memory存储引擎支持哈希索引

1. 搜索的效率好O(1)
2. 磁盘IO花费更少



> 哈希索引的底层数据结构是哈希表，位与内存中。
>
> 当创建哈希索引时，被创建索引的字段会作为哈希函数的参数哈希 hashkey  = hash(name) % bucket_num（存在哈希冲突通过加链解决）
>
> 经过哈希过后，哈希表上的元素没有任何顺序可言，**因此只能进行等值查询**!!!**不适合用于范围查询**



**缺点**

1. 没办法处理磁盘上的数据，加载进内存上构建高效的搜索数据结构(没办法减少磁盘IO的次数)
2. 只适合做等值查询，其他的范围、排序不合适。



#### InnoDB的自适应哈希索引

InnoDB存储引擎会监测到同样的二级索引不断被使用，那么存储引擎会根据这个索引在内存上根据二级索引树上的二级索引值在内存上构建一个哈希索引来加速搜索(只能用于等值查询)

较少了一个二级索引回表操作的优化

自适应哈希在MySQL5.7以后是线程安全的。不同线程操作哈希表中的不同链表不需要加锁



> 自适应哈希索引本身的数据维护需要耗费性能，也就是说并不是自适应哈希索引在任何情况下都会提升二级索引的查询性能
>
> show engine innodb status；能够得到两个重要信息
>
> 1. RW-latch：等待的线程数量(默认八个分区)，每个分区独立，线程安全
> 2. MySQL Server只用自适应哈希索引的频率没有使用二级索引树的频率高，推荐关闭自适应哈希索引



### 索引的常见问题

