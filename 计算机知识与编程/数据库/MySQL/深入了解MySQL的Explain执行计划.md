# MySQL Explain详解[转载]

> Explain（执行计划）可以模拟优化器执行sql查询语句，常用来对某些慢查询语句进行分析和优化，本文对Explain的用法进行详细的研究

[TOC]

## Explain概述

Explain语句的使用很简单，就是在待分析的SQL语句前面加上EXPLAIN并执行即可

```sql
EXPLAIN + 待分析的SQL语句
```

执行结果以查询结果集的形式展示，共有12列，各字段释义如下

* **id**：选择标识符（表的读取顺序）
* **select_type**：表示查询的类型（数据读取操作的操作类型）
* **table**：输出结果集的表
* **partitions**：匹配的分区
* **type**：表示表的连接类型
* **possible_keys**：表示查询时，可能使用的索引（哪些索引可以被使用）
* **key**：表示实际使用的索引
* **key_len**：索引字段的长度
* **ref**：列与索引的比较（表直接的引用）
* **rows**：扫描出的行数（估算的行数）
* **filtered**：按表条件过滤的行百分比
* **Extra**：执行情况的描述和说明

下面对这些字段进行详细的解释

## 执行计划详细的字段说明

### 1. id

select查询的序列号，包含一组数字，表示查询中执行select子句或操作表的顺序，该字段通常与table字段搭配来分析。

包含下面三种情况：

* id相同，执行顺序从上到下
* id不同，如果是子查询，id的序号会递增，id值越大执行优先级越高，越先被执行
* id相同不同，同时存在
  * id如果相同，可认为是同一组，执行顺序从上到下
  * 在所有组中，id值越大执行优先级越高

### 2. select_type

查询的类型，主要用于区别普通查询、联合查询、子查询等复杂的查询。其值主要有六个：

* **SIMPLE**：简单的select查询，查询中不包含子查询或union查询。
* **PRIMARY**：查询中若包含任何复杂的子部分，最外层查询则被标记为主查询（PRIMARY），也就是最后加载的就是PRIMARY。
* **SUBQUERY**：在select或where列表中包含了子查询，就为被标记为SUBQUERY。
* **DERIVED**：在from列表中包含的子查询会被标记为DERIVED(衍生)，MySQL会递归执行这些子查询，将结果放在临时表中。
* **UNION**：若第二个select出现在union后，则被标记为UNION，若union包含在from子句的子查询中，外层select将被标记为DERIVED。
* **UNION RESULT**：从union表获取结果的select。

### 3. table

表示数据来自哪张表

### 4. partitions

官方定义为The matching partitions（匹配的分区）

### 5. type

表示查询所使用的访问类型，type的值主要有八种，该值表示查询的sql语句好坏，从最好到最差依次为：

**NULL > system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > ALL**

下面详细分析这些type对应的情况及性能

#### 5.1 常见type的具体解释

| type        | 解释                                                         |
| ----------- | ------------------------------------------------------------ |
| NULL        | MySQL能够在优化阶段分析查询语句，在执行阶段不用再访问表或者索引 |
| system      | 表只有一行记录（等于系统表），这是const类型的特例，很少见可以忽略 |
| const       | 表示通过索引一次就找到了，const用于比较primary key或者unique索引，因为只匹配一行数据，所以很快，如果主键置于where列表中，MySQL就能将该查询转换为一个常量 |
| eq_ref      | 唯一性索引扫描，对于每个索引键，表中只有一条记录与之匹配，常见于主键或唯一索引扫描 |
| ref         | 非唯一性索引扫描，返回匹配某个单独值得所有行。本质上也是一种索引访问，返回所有匹配某个单独值得行。然而可能会招到多个符合条件的行，应该属于查找和扫描的混合体 |
| ref_or_null | 类似ref，但是可以搜索值为NULL的行                            |
| index_merge | 表示使用了索引合并的优化方法                                 |
| range       | 只检索给定范围的行，示用一个索引来选择行，key列显示使用了哪个索引。一般就是在你的where语句中出现between、<>、in等的查询。这种范围扫描索引比全表扫描要好，因为只需要开始于索引的某一点，而结束另一点，不用扫描全部索引 |
| index       | Full index Scan、Index与ALL的区别：index只遍历索引树，通常比ALL快，因为索引文件通常比数据文件小（也就是虽然all和index都是读全表，但index是从索引中读取的，而ALL是从硬盘中读的） |
| ALL         | Full Table Scan，将遍历全表以找到匹配行                      |



### 6. possible_keys

显示可能应用在这张表中的索引，一个或多个查询涉及到的字段若存在索引，则该索引将被列出，但不一定被实际使用

### 7. key

实际使用到的索引，如果为NULL，则没有使用索引；查询中若使用了覆盖索引（查询的列刚好是索引），则该索引仅出现在key列表

### 8. key_len

表示索引中使用的字节数，可通过该列计算查询中使用的索引的长度，在不损失精确度的情况下，长度越短越好，key_len显示的值为索引字段最大的可能长度，而非实际使用长度，即key_len是根据定义计算而得，不是通过表内检索出的

### 9. ref

显示索引的哪一列被使用了，如果可能的话，是一个常数，哪些列或常量被用于查找索引列上的值

### 10. rows

根据表统计信息及索引选用情况，大致估算出找到所需的记录所需读取的行数

### 11. filtered

查询的表行占表的百分比

### 12. Extra

定义：包含不适合在其他列表中显示但是十分重要的额外信息

| 字段                         | 解释                                                         |
| ---------------------------- | ------------------------------------------------------------ |
| Using filesort               | 说明MySQL会对数据使用一个外部的索引排序，而不是按照表内的索引顺序进行读取，MySQL中无法利用索引完成的排序操作称为“文件排序” |
| Using temporary              | 使用了临时表保存中间结果，MySQL在对结果排序时使用临时表，常见于排序order by和分组查询group by |
| Using index                  | 表示相应的select操作中使用了覆盖索引（Covering Index），避免访问了表的数据行，效率不错。如果同时出现using where，表明索引被用来执行索引键值的查找。如果没有同时出现using where，表明索引用来读取数据而非执行查找动作。 |
| Using where                  | 使用了where条件                                              |
| Using join buffer            | 使用了连接缓存                                               |
| impossible where             | where子句的值总是false，不能用来获取任何元组                 |
| distinct                     | 一旦MySQL找到了与行相联合匹配的行，就不再搜索了              |
| Select tables optimized away | Select操作已经优化到不能再优化了（MySQL根本没有遍历表或者索引就返回数据了） |

> 覆盖索引：select的数据列只用从索引中就能够取得，不必读取数据行，MySQL可以利用索引返回select列表中的字段，而不必根据索引再次读取数据文件，即查询列要被所建的索引覆盖



## 知识点思维导图

![preview](https://typora-1301192342.cos.ap-guangzhou.myqcloud.com/img/20200520115503.png)

---

### 参考文章：

1. https://blog.csdn.net/lvhaizhen/article/details/90763799
2. https://segmentfault.com/a/1190000021458117?utm_source=tag-newest

