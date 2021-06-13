---
title: MySQL 索引
date: 2021-01-06 19:41:23
author: noslime
top: false
cover: false
categories: 数据库
tags: 
	- MYSQL
keywords:  Mysql index
summary: 本文总结下Mysql索引相关的知识，增强对索引的认识，以便于高效利用索引来提高我的数据库查询。
---

# MySQL 索引简介

## 什么是索引

- 官方定义：一种帮助MySQL提高查询效率的数据结构
- 索引数据结构：MySQL中索引的存储类型有两种：BTREE和HASH，具体和表的而存储引擎相关：MyINSAM和InnoDB存储引擎只支持BETREE索引；MEMEORY/HEAP 存储引擎可以支持HASH和BTREE索引。
- 索引的优点：
  1. 大大加速数据查询速度，也是创建索引的主要原因。
  2. 加速表和表之间的连接，特别是在实现数据的参考完整性方面特别有意义
  3. 使用分组和排序子句进行数据查询时，显著减少查询中分组和排序的时间。
- 索引的缺点：
  1. 维护索引需要耗费数据库资源
  2. 索引需要占用磁盘空间
  3. 会降低更新表的速度，因为要维护索引，增加了数据得维护速度。

## 索引的分类

- 主键索引

  设定为主键后数据库会==自动==创建索引，innodb为聚簇索引，不能有空值。是一种特殊的唯一索引。

- 普通索引

  即一个索引只包含单个列，一个表中可以有多个普通索引，允许空值与重复值。

- 唯一索引

  索引列的值必须唯一，但允许有空值

- 组合索引

  即一个索引包含多个列

- 全文索引（5.7之前只有MyISAM引擎支持，自5.7版本开始，InnoDB也支持了）

  全文索引类型为FULLTEXT，在定音索引的列上支持值得全文索引，允许这些索引列中插入重复值和空值。全文索引可以在char、varchar、text类型列上创建。MySQL只有MYISAM引擎支持全文索引。

- 空间索引

  空间索引是对空间数据类型的字段建立的索引，MySQL种的空间类型有4种，分别是：GEOMETRY、POINT、LINESTRING和POLYGON。MySQL使用SAPTIAL关键字进行扩展。空间索引的列必须声明为NOT NULL。

## 索引的基本操作

### 创建索引

#### 创建表的时候创建索引

- 基本语法格式

  ```sql
  CREATE TABLE table_name [col_name data_type] [UNIQUE|FULLTEXT|SPATIAL] [INDEX|KEY] [index_name](col_name[length]) [ASC|DESC]
  ```

  其中UNIQUE、FULLTEXT、SAPTIAL为可选参数，分别表示唯一索引、全文索引和空间索引；INDEX和KEY为同义词，作用相同，用来指定创建索引；index_name指定索引的名称，为可选参数，若不指定，则默认col_name为索引值；col_name为需要创建索引的字段列；length为可选参数，表示索引的长度，只有字符串类型的字段才能指定索引的长度；ASC或DESC指定升序或者降序的索引值存储。

- 示例

  1. 创建普通索引

     ```sql
     CREATE TABLE `t_user01` (
       `id` int NOT NULL,
       `NAME` varchar(255) NOT NULL,
       KEY(`NAME`)
     ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
     ```

  2. 创建唯一索引

     ```sql
     CREATE TABLE `t_user02` (
       `ID` int NOT NULL,
       `NAME` varchar(255) NOT NULL,
       UNIQUE KEY `UniqIdx` (`ID`)
     ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
     ```

  3. 创建组合索引

     ```sql
     CREATE TABLE `t_user03` (
       `id` int NOT NULL,
       `name` char(30) NOT NULL,
       `age` int NOT NULL,
       `info` varchar(255) DEFAULT NULL,
       KEY `MultiIdx` (`id`,`name`,`age`)
     ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
     ```

     注意：组合索引遵循左前缀原则，并且mysql引擎在查询为了更好利用索引，查询过程会动态调整查询字段顺序以便利用索引。

  4. 创建全文索引

     ```sql
     CREATE TABLE `t_user04` (
       `id` int NOT NULL,
       `name` char(30) NOT NULL,
       `age` int NOT NULL,
       `info` varchar(255) DEFAULT NULL,
       FULLTEXT KEY `FullTxtIdx` (`info`)
     ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
     ```

  5. 创建空间索引

     ```sql
     CREATE TABLE `t_space` (
       `p` point NOT NULL,
       SPATIAL KEY `spatIdx` (`p`)
     ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
     ```

#### 在已存在的表上创建索引

- 使用[ALTER语句](https://dev.mysql.com/doc/refman/5.7/en/alter-table.html)创建

  - 语法格式

    ```text
    ALTER TABLE tbl_name
    ADD {UNIQUE | FULLTEXT | SPATIAL} [INDEX | KEY] [index_name]
            (key_part,...) [index_option] ...
    ```

  - 示例

    ```sql
    ALTER TABLE t_user01 ADD KEY(`name`);
    ALTER TABLE t_user02 ADD UNIQUE KEY `UniqIdx`(`ID`);
    ALTER TABLE t_user03 ADD KEY `MultiIdx`(`id`,`name`,`age`);
    ALTER TABLE t_user04 ADD FULLTEXT KEY `FullTxtIdx` (`info`);
    ALTER TABLE t_space ADD SAPTIAL KEY `spatIdx` (`p`);
    ```

  

- 使用[CREATE语句](https://dev.mysql.com/doc/refman/5.7/en/create-index.html)创建

  - 语法格式

    ```text
    CREATE [UNIQUE | FULLTEXT | SPATIAL] INDEX index_name
        [index_type]
        ON tbl_name (key_part,...)
        [index_option]
        [algorithm_option | lock_option] ...
    ```

  - 示例

    ```sql
    CREATE INDEX `name` ON t_user01(`name`);
    CREATE UNIQUE INDEX `UniqIdx` ON t_user02(`ID`);
    CREATE INDEX `MultiIdx` ON t_user03(`id`,`name`,`age`);
    CREATE FULLTEXT INDEX `FullTxtIdx` ON t_user04(`info`);
    CREATE SAPTIAL INDEX `spatIdx` ON t_space(`p`);
    ```

### 删除索引

删除索引语法格式如下

```sql
ALTER TABLE table_name DROP INDEX index_name;
DROP INDEX index_name on table_name;
```

## 索引数据结构

​	考虑到磁盘IO是非常高昂的操作，计算机操作系统做了一些优化，当一次IO时，不光把当前磁盘地址的数据，而是把相邻的数据也都读取到内存缓冲区内，因为局部预读性原理告诉我们，当计算机访问一个地址的数据的时候，与其相邻的数据也会很快被访问到。每一次IO读取的数据我们称之为一页(page)。具体一页有多大数据跟操作系统有关，一般为4k或8k，也就是我们读取一页内的数据时候，实际上才发生了一次IO，这个理论对于索引的数据结构设计非常有帮助。

​	而在MySQL数据库中一页一般为16kb，MySQL基于页的形式进行索引的管理，结构大致如下图所示，这种数据结构叫做**B+Tree**，B+Tree只有叶子存储实际数据信息，其他节点只存储键值信息，大大降低了树的高度，从而减少了磁盘IO的次数。

![](C:\Document\hexo\blog\source\images\mysql_index01.png)

​																				图 一

InnoDB存储引擎中页的大小为16KB，一般若表的主键类型为INT（4个字节）或BIGINT（8个字节），指针类型也一般为4或8个字节，也就是说一个叶节点中大概存储16KB/(8B+8B)=1K个键值，若数据记录为也为16B，则一个深度为3的B+Tree索引理论上可以维护```10^3*10^3*500```=5亿条记录。因而在数据库中，B+Tree的高度一般都在2\~4层，而MySQL的InnoDB存储引擎在在设计时是将**根节点常驻内存**的，也就是说，查找某一键值的行记录最多只需要1\~3次IO操作。

所以一般来说，普通项目B+Tree的高度2层基本够用了。

### 补充 

#### B-树的概念简介

B-树是一种多路平衡查找树。每一个节点最多包含K个孩子，而K又被称为是B树的阶。这个阶取决于磁盘页大小。它类似普通的平衡二叉树，不同的一点是B-树允许每个节点有更多的子节点。

#### B+同B-树的不同

B+树是B-树的变体，也是一种多路搜索树, 它与 B- 树的不同之处在于:

1. 非叶子节点只存储键值信息，数据记录都存放在叶子节点中
2. 为所有叶子结点增加了一个链指针
3. B+树只有在叶子节点才会存在数据，所以同样的情况下，一次性能够放入更多的数据进入内存，因此减少了磁盘io，效率更高些。

4. B+树内节点不存储数据，所有 data 存储在叶节点导致查询时间复杂度固定为 log n。而B-树查询时间复杂度不固定，与 key 在树中的位置有关，最好为O(1)。

## 聚簇索引与非聚簇索引

聚簇索引：将数据存储与索引放到了一块，索引结构的==叶子节点保存了数据==。

非聚簇索引：将数据与索引分开存储，索引结构的叶子节点指向了数据所在的位置。

> ​	在innodb中，在聚簇索引之上创建的索引称之为辅助索引，**非聚簇索引都是辅助索引**，像复合索引、前缀索引、唯一索引。辅助索引叶子节点存储的不再是行的位置，而是主键值，辅助索引访问数据总是需要二次查找。

1. InnoDB

![](https://cdn.jsdelivr.net/gh/noslime/noslime.github.io@master/source/images/2106131655_mysql_index.png)

​																		图 二  InnoDB 索引检索过程

如图二所示，InnoDB 默认使用主键作为聚簇索引，如果直接通过主键检索数据，只需检索一次即可获取数据；若通过辅助索引进行检索数据，则要多上一个步骤：即先在辅助索引中检索到树的主键，然后根据主键去主键索引中查找真正数据所在。

在InnoDB中，主键默认是聚簇索引，若没有指定主键，则会选择一个非空且唯一的索引代替。如果没有这样的索引，InnoDB会隐式定义一个主键来作为聚簇索引。如果已经设置了主键为聚簇索引有希望再单独设置聚簇索引，必须先删除主键，添加后，再恢复主键。

2. MyISAM

![](https://cdn.jsdelivr.net/gh/noslime/noslime.github.io@master/source/images/2106131718_mysql_index.png)

​												图三  MyISAM 索引检索过程

MyISAM使用的是非聚簇索引，如图三所示，主键索引与辅助索引结构一致，存储的只是数据的地址，而数据存储在专门的位置，所以访问没有区别。

### 聚簇索引的优势

1. 由于行数据和聚簇索引的叶子节点存储在一起，同一页有多条数据，访问同一页数据不同记录时，由于已经加载到缓存中了，直接在内存中拿即可，访问速度更快。
2. 当行数据发生变化时，索引树的节点也需要分裂变化；或者发生新的IO读数据时，可以避免对辅助索引的维护工作。另外辅助索引放地主键值比放地址更省空间。

### 使用聚簇索引需要注意什么

1. 最好不要使用UUID作为主键，因为太过离散，且可能出现新增加的uuid插入到树的中间，导致索引树调整的复杂度增大。
2. 建议使用int类型的自增，方便排序，并且默认会在索引树的末尾增加主键值，对索引树结构影响最小。且主键值越大，辅助索引保存的也要跟着增大。
3. 其中主键自增的好处是，只要索引是相邻的，那么数据一般也在磁盘上相邻。避免了不必要的调整物理地址，分页等，总之就是磁盘碎片化低。





