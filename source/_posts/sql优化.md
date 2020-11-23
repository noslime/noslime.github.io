---
title: 正确使用SQL既是优化
date: 2020-11-21 16:40:54
author: noslime
tags: 
	- SQL
	- MySQL
categories: SQL
---

## SQL优化小结

### 一、正确使用索引

1. 对查询进行优化，应尽量避免全表扫描，首先应考虑在where及order by 涉及的列上建立索引

2. 应尽量避免在where子句中对字段进行null值判断，否则将导致引擎放弃使用索引而进行全表扫描，如：

   ```SQL
   SELECT ID FROM TEST_INDEX WHERE NUM　IS NULL
   ```

   可以以在num上设置默认值0，确保表中null列没有null值，然后这样查询：

   ```sql
   SELECT ID FROM TEST_INDEX WHERE NUM = 0;
   ```

3. 应尽量避免在WHERE子句中使用!=或<>操作符，否则将引擎放弃使用索引而进行全表扫描

4. 应尽量避免在WHERE子句中使用OR来连接条件，否则只有当条件中的列都是索引字段，且等值比较时候，才能使用索引，如：

   ```sql
   SELECT ID FROM TEST_INDEX WHERE  NUM=10 OR AGE=20; //使用的是index_merge
   SELECT ID FROM TEST_INDEX WHERE  NUM<10 OR AGE>20; //未使用索引
   ```

   可以这样查询：

   ```sql
   SELECT ID FROM TEST_INDEX WHERE NUM=10
   UNION ALL
   SELECT ID FROM TEST_INDEX WHERE NUM=20;
   ```

5. 对于连续的数值能用BETWEEN就不要用IN，否则也会导致全表扫描，如

   ```sql
   SELECT ID FROM TEST_INDEX WHERE NUM　IN(1,2,3);
   ```

   可以改为：

   ```sql
   SELECT ID FROM TEST_INDEX WHERE NUM BETWEEN 1 AND 3;
   ```

   

6. 应尽量避免在WHERE子句等号的左边进行函数、算术运算或其他表达式运算，这将导致引擎放弃使用索引。如

   ```sql
   SELECT ID FROM TEST_INDEX WHERE SUBSTRING(NUM, 1,3)='sql';
   ```

   可以改为：

   ```sql
   SELECT ID FROM TEST_INDEX WHERE NUM LIKE sql%;
   ```

   在使用like关键字进行查询的时候，如果匹配字符串的第一个字符为"%"索引不会器作用。只有"%"不在第一个位置，索引才会起作用。

7. 在使用索引字段作为条件时，如果该索引是复合索引，那么必须使用到该索引中的第一个字段作为条件时才能保证系统使用该索引，否则该索引将不会被使用，并且应让字段顺序与索引顺序相一致

   > 假如对col1、col2、col3创建组合索引，相当于创建了（col1）、（col1，col2）、（col1，col2，col3）3个索引

8. 并不是所有的列都适合做索引，离散型不高的列，即当一列数据有大量重复数据时不适合做索引，如性别、国家等有限数据列

9. 索引并不是越多越好，索引固然可以提高相应的 select 的效率，但同时也降低了 insert 及 update 的效率

   ​	

11. IN和EXIST的选择，in 是把外表和内表作hash 连接，而exists是对外表作loop循环，每次loop循环再对内表进行查询。其中in会使用外层查询表索引，而exist使用内层查询表索引，所以外层查询表小于子查询表，则用exists，外层查询表大于子查询表，则用in，如果外层和子查询表差不多，则爱用哪个用哪个。另外，如果查询语句使用了not in 那么内外表都进行全表扫描，没有用到索引；而not extsts 的子查询依然能用到表上的索引。所以无论那个表大，用not exists都比not in要快。

    

---



### 二、常用SQL操作符

1. JOIN

   包括LEFT JOIN、INNER JOIN、RIGHT JOIN、FULL JOIN等几种，我找来一张网图简单明了了说明了几个操作符的作用范围及基本使用方法，如下：

   ![](https://cdn.jsdelivr.net/gh/noslime/noslime.github.io@master/source/images/sqljoins.png)

   使用时应以小表驱动大表，左连接LEFT JOIN中，左表为驱动表，右表为被驱动表；右连接RIGHT JOIN中，右表为驱动表，左表为被驱动表。其中应尽量在被驱动表上建立索引。

2. UNION 

   UNION 内部的 SELECT 语句必须拥有相同数量的列。列也必须拥有相似的数据类型。同时，每条 SELECT 语句中的列的顺序必须相同。

   ```sql
   SELECT column_name(s) FROM table_name1
   UNION
   SELECT column_name(s) FROM table_name2
   ```

   默认地，UNION 操作符选取不同的值。如果允许重复的值，请使用 UNION ALL。

-----



### 三、表设计

1. 尽量使用数字型字段，若只含数值信息的字段尽量不要设计为字符型，这会降低查询和连接的性能，并会增加存储开销。这是因为引擎在处理查询和连接时会逐个比较字符串中每一个字符，而对于数字型而言只需要比较一次就够了

2. 对于长度基本固定的列，如果该列恰好更新又特别频繁，适合char。

3.  TIMESTAMP和 DATETIME选择

   > DATETIME和 TIMESTAMP类型所占的存储空间不同，前者8字节，后者4字节。前者范围是 
   >
   > 1000-01-01-00:00:00 ~ 9999-12-31 23:59:59，后者范围是 1970-01-01 8:00:01 ~ 2038-01-19 
   >
   > 11:14:07。 所以 TIMESTAMP支持的范围比 DATETIME要小。
   >
   > TIMESTAMP显示与时区有关，它把客户端插入的时间从当前时区转化为UTC（世界标准时间）进行存储。查询时，将其又转化为客户端当前时区进行返回。另外TIMESTAMP每个表允许一个自增时间戳，如：
   >
   > ```sql
   > ALTER TABLE `USER` ADD COLUMN `UTIME` TIMESTAMP NULL DEFAULT NULL ON UPDATE CURRENT_TIMESTAMP; //UTIME字段插入更新数据时不需指定
   > ```
   >
   > 显然存储空间小且会自动切换的TIMESTAMP更方便一些，虽然表示时间范围小好多，但根据IT行业技术换代的速度，相信不用等到2038年，就已经有了更好的替换方案。
   >
   > 

4.  所有表都要指定主键，不要强制使用外键。如果没有主键或者唯一索引，update/delete 是通过所有字段来定位操作的行，相当于每行就是一次全表扫描。即使2个表的字段有明确的外键参考关系，也不建议使用 FOREIGN KEY，因为新纪录会去主键表做校验，影响性能。

5. 对精度有要求可以用decimal。其中decimal(M,D) M表示总长度，D表示小数部分，M范围是1到65，D范围是0到30。

   1. float：浮点型，含字节数为4, 32bit，7个有效位
   2. double：双精度实型，含字节数为8, 64bit，15个有效位
   3. decimal：数字型，128bit，不存在精度损失，常用于银行账目计算，28个有效位。

6. 字段不要过多，取名见名思义，加注释

   字段过多的表可以进行分表，如用户表USER有id、姓名、密码、地址、电话、爱好、备注等字段，可将地址、爱好、备注等不常用字段分解出另一个表USER_DETAIL。

7. 对于经常联合查询的表，可根据需要看是否可以建立中间表存储需要联合查询的数据。

