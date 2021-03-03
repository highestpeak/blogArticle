# 设计理论

**函数依赖：**

（函数依赖是设计理论里最基本的一个描述条件）

- 记 A->B 表示 A 函数决定 B，也可以说 B 函数依赖于 A
- 对于 A->B，如果能找到 A 的真子集 A'，使得 A'-> B，那么 A->B 就是部分函数依赖，否则就是完全函数依赖。
- 对于 A->B，B->C，则 A->C 是一个传递函数依赖。

*<u>todo：这个函数依赖写的还不完全</u>*

**异常：**

- 冗余异常、修改一场、删除异常、插入异常

<u>*todo：异常写的还不完全*</u>

**范式：**

（范式是为了解决四种异常）

- 第一范式：属性不可分

  例如不能含有学生字段，但是可以含有学生姓名、学生性别等

- 第二范式：非主属性必须完全函数依赖于主属性

  学生课程关系中，`{Sno, Cname}` 为键码，有如下函数依赖：

  - `Sno -> Sname, Sdept`
  - `Sdept -> Mname`
  - `Sno, Cname-> Grade`

  `Grade` 完全函数依赖于键码，它没有任何冗余数据，每个学生的每门课都有特定的成绩。`Sname`, `Sdept` 和 `Mname` 都部分依赖于键码，当一个学生选修了多门课时，这些数据就会出现多次，造成大量冗余数据。

  可以分解成两个关系：学生学院关系、学生课程关系

  学生学院关系：键码为 `{Sno,Sdept}`

  - `Sno -> Sname, Sdept`
  - `Sdept -> Mname`

  学生课程关系：键码为 `{Sno,Cname}`

  - `Sno, Cname -> Grade`

- 第三范式：非主属性不传递函数依赖于键码。即不存在传递函数依赖

  - 如上面学生学院关系中存在 `Sno -> Sdept -> Mname`。所以进一步分成两个表，学生学院表，学院院长表

<u>*todo：根据某个场景设计数据表？*</u>

---

表设计：

- 为什么用自增列作为主键

  - InnoDB必须需要一个主键：

    定义了主键(PRIMARY KEY)，那么InnoDB会选择主键作为聚集索引

    如果没有显式定义主键，则InnoDB会选择第一个不包含有NULL值的唯一索引作为主键索引。

    如果也没有这样的唯一索引，则InnoDB会选择内置6字节长的ROWID作为隐含的聚集索引(ROWID随着行记录的写入而主键递增，这个ROWID不像ORACLE的ROWID那样可引用，是隐含的)

  - 记录存储在主索引上，需要按照顺序

  - 如果使用非自增主键（如果身份证号或学号等），由于每次插入主键的值近似于随机，因此每次新纪录都要被插到现有索引页得中间某个位置，这样就会频繁进行页之间的数据移动
  
  - UUID太长且排序性差

---

存储过程、视图、触发器：

- 存储过程是一些预编译的SQL语句，便于模块化设计

  - 优点：执行效率比较高、安全性高，需要有权限、可反复利用
  - 缺点：调试麻烦、移植麻烦

- 视图：

  本质上是一个虚拟表，在物理上是不存在的，其内容与真实的表相似，包含一系列带有名称的列和行数据

  - 视图对背后的表提高了安全性
  - 对视图的更新直接影响基本表
  - 视图来自多个表时，不允许通过视图更新数据
  - 视图会把视图的查询转化成对基本表的查询，可能对于视图是简单的，但是却是对基本表的复杂查询

- 触发器：

  触发器是指一段代码，当触发某个事件时，自动执行这些代码

  - 实现级联修改、监控更改
  - 不要滥用，否则维护困难
  - 触发器形如：Before Insert、After Update等


# SQL必知必会

- <u>*todo：包含下面这本书的介绍内容*</u>  `BenForta. SQL 必知必会 [M]. 人民邮电出版社, 2013.`
- https://github.com/CyC2018/CS-Notes/blob/master/notes/SQL.md

**SQL语句各个部分的执行顺序：**

``` sql
(1) FROM <left_table>
(3) <join_type> JOIN <right_table>
(2) ON <join_condition>
(4) WHERE <where_condition>
(5) GROUP BY <group_by_list>
(6) WITH {CUBE | ROLLUP}
(7) HAVING <having_condition>
(8) SELECT
(9) DISTINCT
(9) ORDER BY <order_by_list>
(10) <TOP_specification> <select_list>
```

- FROM 才是 SQL 语句执行的第一步
- SELECT 是在大部分语句执行了之后才执行的
  - 这就是不能在 WHERE 中使用在 SELECT 中设定别名的字段作为判断条件的原因
  - 也是不能再group by中使用别名的原因
- 大体顺序是：`from==>where==>group by==>聚集函数==>having==>计算表达式==>select==>order by`
- 并非所有SQL都按照上述的顺序进行
- 以上每个步骤都会产生一个虚拟表，该虚拟表被用作下一个步骤的输入。这些虚拟表对调用者(客户端应用程序或者外部查询)不可用。只有最后一步生成的表才会会给调用者

**join：**

来自：

1. [StackOverflow: What's the difference between INNER JOIN, LEFT JOIN, RIGHT JOIN and FULL JOIN?](https://stackoverflow.com/questions/5706437/whats-the-difference-between-inner-join-left-join-right-join-and-full-join)
2. [StackOverflow: What is the difference between “INNER JOIN” and “OUTER JOIN”?](https://stackoverflow.com/questions/38549/what-is-the-difference-between-inner-join-and-outer-join)

![](E:\_data\博文临时库\博文中的图片\SQL的JOIN示意.png)

- 笛卡儿积就是A和B集合中的元素的所有组合情况。
- join就是inner join

**group by和having和聚集函数：**

- 在select指定的字段：
  - 要么包含在Group By语句的后面，作为分组的依据；
  - 要么被包含在聚合函数中
  
- group by不能选择除了上面这一条说的字段之外的字段的原因：

  如果一个字段不在group by后面也没有再聚集函数中的话，那这个字段的值就会有多个，这样数据库就不知道选择字段的哪一个值。

- 常用聚合函数
  
  - `count()` 计数、`sum()` 求和、`avg()` 平均数、`max()` 最大值、`min()` 最小值
  
- group by实现原理？

  知道的不尽详细，但是可以指明一部分原理：

  group by可以使用索引来完成分组，他也受着最左前缀原则的限制。group by是否能利用索引不仅和它使用的字段有关还和聚集函数的字段有关。如果不能使用索引，group by就不得不使用临时表来完成操作。

  总体上是先分组再聚集函数运算。

**分页limit：**

我们使用limit常见的是使用形如 `limit m,n` 这样的写法，在这样的写法中，每次SQL语句都会扫描m+n条数据，在m的数据变得极大之后（例如十万、一千万），这样的速度就会变得很慢。即在获取前几页的数据的时候也没用多长时间，但是越往后需要的时间也就越长。

所以我们就需要优化：

- 先获取指定范围的id，然后根据id去找数据

  原查询：

  ``` sql
  -- 0.24 sec
  select count(1) as num from order_info where channels_id=35;
  -- 0.01 sec
  select * from order_info where channels_id=35 order by order_id desc limit 0,20;
  -- 12.55 sec  即便是第二次查询也用了4.27 sec（mysql自身也会有查询缓存机制）
  select * from order_info where channels_id=35 order by order_id desc limit 1320000,20;
  ```

  优化后：

  ``` sql
  -- 0.00 sec
  select * from order_info,
  (select order_id from order_info where channels_id=35 order by order_id desc limit 0,20) order_info_tmp 
  where order_info.order_id = order_info_tmp.order_id; 
  
  -- 0.47 sec
  select * from order_info,
  (select order_id from order_info where channels_id=35 order by order_id desc limit 1320000,20) order_info_tmp 
  where order_info.order_id = order_info_tmp.order_id; 
  ```

  (这样的做法虽然快了不少，可是依然扫描了很多的数据行，在数据量大的情况下依然会很慢，尤其是在做数据导出的时候)

  - 进一步的可以先在子查询获取指定范围的第一条id，然后再进行limit，例如：

    ``` sql
    SELECT * FROM `cdb_posts` 
    WHERE pid >= 
    	(SELECT pid FROM  `cdb_posts` ORDER BY pid LIMIT 1000000 , 1) 
    LIMIT 30
    ```

- 可以用`between and`

  ``` sql
  select id from A order by id  between 90000 and 90010;
  ```

- 实际上上面这种优化做法是使用了索引来优化了分页

**其他：**

- varchar和char

  - char定长，varchar不定长
  - char适合存储密码的md5值这样的值
  - varchar存储的数据还需要额外的字节来保存长度

- 误操作执行了 drop  语句，如何完整恢复？

  - <u>*todo：基于日志和备份？？*</u>

- DISTINCT和GROUP BY的区别？

  其实二者没有什么可比性，但是对于不包含聚集函数的GROUP BY操作来说，和DISTINCT操作是等价的。不过虽然二者的结果是一样的，但是二者的执行计划并不相同。

  - distinct只是将重复的行从结果中出去；
    group by是按指定的列分组，一般这时在select中会用到聚合函数。

  - distinct是把不同的记录显示出来。
    group by是在查询时先把纪录按照类别分出来再查询。

- count(1)、count(*)、count(column)

  - count(1)：忽略所有列，用1代表代码行，在统计结果的时候，不会忽略列值为NULL
    - 避免使用这样的写法，仅对一个表有效，无法用于多表查询
    - count(1)和count(*)几乎没有任何区别
  - count(*)：所有的列，相当于行数，在统计结果的时候，不会忽略列值为NULL 
  - count(列名)：列名那一列，在统计结果的时候，**会忽略列值为空，即NULL值**（这里的空不是只空字符串或者0，而是表示null）的计数

  <u>*todo： 执行效率上暂且不表，但也和索引有关？？*</u>

# SQL优化

慢SQL该如何分析：

- 通过explain，可以查看SELECT语句的执行情况
- 数据库CPU负载高。一般是查询语句中有很多计算逻辑，导致数据库CPU负载。
- 没有建索引或者索引没有生效
- 并发下锁的开销
- 数据量过大===>分库分表
- 使用缓存，在持久层或持久层之上做缓存，使用redis等做缓存等

**写SQL时我们应该注意**

1. 避免全表扫描：对查询进行优化，应尽可能避免全表扫描
2. 建立索引：首先应考虑在 where 及 order by 涉及的列上建立索引
   1. 减少where 字段值null判断，因为这会导致引擎放弃使用索引而进行全表扫描
   2. 避免在 where 子句中使用!=或<>操作符
   3. 避免在 where 子句中使用 or 来连接条件
3. 不在条件判断时进行 算数运算函数运算等

**常见SQL优化：**

总的优化原则就是使用索引和避免全表扫描

来自：https://juejin.im/post/6844903573935882247

- limit分页优化

  ``` sql
  -- 从
  SELECT id FROM A LIMIT 1000,10; --很快
  SELECT id FROM A LIMIT 90000,10; --很慢
  -- 到
  select id from A order by id limit 90000,10; --方案一
  select id from A order by id between 90000 and 90010; --方案二
  ```

- 利用limit 1 、top 1 取得一行（避免全表扫描）

  ``` sql
  -- 从
  SELECT id FROM A LIKE 'abc%' 
  -- 到
  SELECT id FROM A LIKE 'abc%' limit 1
  ```

- 避免使用`select *`，（<u>*todo：但是这个为什么是全表扫描呢？*</u>）

  ``` sql
  -- 从
  SELECT * FROM A
  -- 到
  SELECT id FROM A 
  ```

- 批量插入优化

  因为批量插入会减少数据库连接、减少事务创建次数、减少日志写入次数

  ``` sql
  -- 从
  INSERT into person(name,age) values('A',24)
  INSERT into person(name,age) values('B',24)
  INSERT into person(name,age) values('C',24)
  -- 到
  INSERT into person(name,age) values('A',24),('B',24),('C',24),
  ```

- like语句的优化

  感觉这其实也是最左前缀的一个应用，由于`abc`前面用了“%”，因此该查询必然走全表查询，不会命中索引。

  ``` sql
  -- 从
  SELECT id FROM A WHERE name like '%abc%'
  -- 到
  SELECT id FROM A WHERE name like 'abc%'
  ```

- where子句使用or的优化

  or不能命中索引

  ``` sql
  -- 从
  SELECT id FROM A WHERE num = 10 or num = 20
  -- 到
  SELECT id FROM A WHERE num = 10 union all SELECT id FROM A WHERE num=20
  ```

- where子句中使用 IS NULL 或 IS NOT NULL 的优化

  在where子句中使用 IS NULL 或 IS NOT NULL 判断，索引将被放弃使用，会进行全表查询。所以应当对相应字段设置默认值，确保表中相应字段没有null值，避免全表扫描。

  ``` sql
  -- 从
  SELECT id FROM A WHERE num IS NULL
  -- 到
  SELECT id FROM A WHERE num=0
  ```

- where子句中对字段进行表达式操作的优化

  如果在where子句中的“=”左边进行函数、算数运算或其他表达式运算，系统将可能无法正确使用索引

  ``` sql
  -- 从
  SELECT id FROM A WHERE datediff(day,createdate,'2019-11-30')=0 
  -- 到
  SELECT id FROM A WHERE createdate>='2019-11-30' and createdate<'2019-12-1'
  ```

- 排序的索引问题

  尽量不要包含多个列的排序，如果需要最好给这些列创建复合索引

- 用 union all 替换 union

  差异主要是前者需要将两个（或者多个）结果集合并后再进行唯一性过滤操作，这就会涉及到排序，增加大量的CPU运算，加大资源消耗及延迟。

  所以当我们可以确认不可能出现重复结果集或者不在乎重复结果集的时候，尽量使用union all而不是union

- Inner join 和 left join、right join、子查询

  - inner join性能比较快，因为inner join是等值连接。能用inner join连接尽量使用inner join连接。

  - 子查询的性能又比外连接性能慢，尽量用外连接来替换子查询。
  - join左边尽量用小表

  ``` sql
  -- 从
  Select* from A where exists (select * from B where id>=3000 and A.uuid=B.uuid);
  -- 到
  Select* from A inner join B ON A.uuid=B.uuid where b.uuid>=3000;
  ```

- exist & in 优化

  - in()适合B表比A表数据小的情况
  - exists()适合B表比A表数据大的情况

  ``` sql
  SELECT * from A WHERE id in ( SELECT id from B )
  SELECT * from A WHERE id EXISTS ( SELECT 1 from A.id= B.id )
  ```

- 

