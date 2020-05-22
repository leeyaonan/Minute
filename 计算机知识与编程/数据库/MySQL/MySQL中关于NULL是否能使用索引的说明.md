# MySQL关于NULL能否使用索引的研究

网上流传着这么一个说法：

> MySQL的where子句中包含IS NULL、IS NOT NULL、!=这些条件时索引会失效，只能使用全表扫描

下面对这个说法进行一个验证：

还使用上面的user表，补充一些数据后：

![image-20200522153056145](D:\PersonalFiles\微云同步助手\图片\Typora图床\20200522153057.png)

分别执行如下SQL，查看执行计划

```mysql
explain select * from `user` where `phone` is null;

explain select * from `user` where `phone` is not null;

explain select * from `user` where `phone` != '00000000000';
```

![image-20200522153511649](D:\PersonalFiles\微云同步助手\图片\Typora图床\20200522153512.png)

这里可以看到索引并没有全部失效，再把数据的规模扩大点，插入900多个为NULL的数据，数据规模扩充至1000，再次执行计划

```mysql
INSERT INTO `rotli`.`user`(`username`, `password`, `phone`) VALUES ( '张四山', '123456', NULL);
```



![image-20200522154157905](D:\PersonalFiles\微云同步助手\图片\Typora图床\20200522154159.png)

![image-20200522154356326](D:\PersonalFiles\微云同步助手\图片\Typora图床\image-20200522154356326.png)

发现这里的执行计划和上面相反了

首先可以确定的是上面的说法是错误的，下面分析一下原理

## NULL值是怎么存储在数据库中的？（了解）

以下面的表为例：

```mysql
CREATE TABLE s1 (
    id INT NOT NULL AUTO_INCREMENT,
    key1 VARCHAR(100),
    key2 VARCHAR(100),
    key3 VARCHAR(100),
    key_part1 VARCHAR(100),
    key_part2 VARCHAR(100),
    key_part3 VARCHAR(100),
    common_field VARCHAR(100),
    PRIMARY KEY (id),
    KEY idx_key1 (key1),
    KEY idx_key2 (key2),
    KEY idx_key3 (key3),
    KEY idx_key_part(key_part1, key_part2, key_part3)
) Engine=InnoDB CHARSET=utf8;
```

这个表里有10000条记录

在MySQL中，每一条记录都有它固定的格式，我们以`InnoDB`存储引擎的`Compact`行格式为例，来看一下`NULL`值是怎样存储的。在`Compact`行格式下，一条记录是由下边这几个部分构成的（图片来自网络）：

![img](D:\PersonalFiles\微云同步助手\图片\Typora图床\20200522154850.webp)

存储NULL值得过程如下：

1. 首先统计表中允许存储`NULL`的列有哪些。

   我们前边说过，主键列、被`NOT NULL`修饰的列都是不可以存储`NULL`值的，所以在统计的时候不会把这些列算进去。

2. 如果表中没有允许存储`NULL`的列，则`NULL值列表`也不存在了，否则将每个允许存储`NULL`的列对应一个二进制位，二进制位按照列的顺序逆序排列，二进制位表示的意义如下：

   因为表`record_format_demo`有3个值允许为`NULL`的列，所以这3个列和二进制位的对应关系就是这样：

   ![img](D:\PersonalFiles\微云同步助手\图片\Typora图床\20200522155547.webp)

   

   再一次强调，二进制位按照列的顺序逆序排列，所以第一个列`c1`和最后一个二进制位对应。

3. - 二进制位的值为`1`时，代表该列的值为`NULL`。
   - 二进制位的值为`0`时，代表该列的值不为`NULL`。

4. `InnoDB`规定`NULL值列表`必须用整数个字节的位表示，如果使用的二进制位个数不是整数个字节，则在字节的高位补0。

   表`record_format_demo`只有3个值允许为`NULL`的列，对应3个二进制位，不足一个字节，所以在字节的高位补0，效果就是这样：

   ![img](D:\PersonalFiles\微云同步助手\图片\Typora图床\20200522155607.webp)

   

   以此类推，如果一个表中有9个允许为`NULL`，那这个记录的`NULL值列表`部分就需要2个字节来表示了。

假设我们现在向`record_format_demo`表中插入一条记录：

```mysql
INSERT INTO record_format_demo(c1, c2, c3, c4) VALUES('eeee', 'fff', NULL, NULL);
```

这条记录的`c1`、`c3`、`c4`这3个列中`c3`和`c4`的值都为`NULL`，所以这3个列对应的二进制位的情况就是：



![img](D:\PersonalFiles\微云同步助手\图片\Typora图床\20200522155620.webp)



所以这记录的`NULL值列表`用十六进制表示就是：`0x06`。

## 键值为NULL的记录是怎么在B+树种存放的？（了解）

对于InnoDB存储引擎来说，记录都是存储在页面中的（一个页面默认是16KB大小），这些页面可以作为`B+`树的节点而组成一个索引，类似这种样子（只是用下边的图举个B+树的例子而已，跟我们上边列举的表没关系）：



![img](D:\PersonalFiles\微云同步助手\图片\Typora图床\20200522155753.webp)



聚簇索引和二级索引都对应着像上图一样的`B+`树（也就是说有多少个索引就有多少棵对应的`B+`树），不过：

- 对于聚簇索引来说，页面中的记录是按照主键值进行排序的；而对于二级索引来说，页面中的记录是按照给定的索引列的值进行排序的。
- 对于聚簇索引来说，B+树每一层节点（页面）都是按照页中记录的主键值大小进行排序的；而对于二级索引来说，B+树每一层节点（页面）都是按照页中记录的给定的索引列的值进行排序的。
- 对于聚簇索引来说，B+树叶子节点对应的页面中存储的是完整的用户记录（就是一条记录中包含我们定义的所有列值，还包含一些InnoDB自己添加的一些隐藏列）；而对于二级索引来说，B+树叶子节点对应的页面中存储的只是`索引列的值 + 主键值`。

按规定，一条记录的主键值不允许存储`NULL`值，所以下边语句中的WHERE子句结果肯定为`FALSE`：

```mysql
SELECT * FROM tbl_name WHERE primary_key IS NULL;
```

像这样的语句优化器自己就能判定出WHERE子句必定为NULL，所以压根儿不会去执行它，不信我们看（Extra信息提示WHERE子句压根儿不成立）：



![img](D:\PersonalFiles\微云同步助手\图片\Typora图床\20200522155758.webp)



对于二级索引来说，索引列的值可能为`NULL`。那对于索引列值为`NULL`的二级索引记录来说，它们被放在`B+`树的哪里呢？答案是：放在B+树的最左边。比方说我们有如下查询语句：

```mysql
SELECT * FROM s1 WHERE key1 IS NULL;
```

那它的查询示意图就如下所示：



![img](D:\PersonalFiles\微云同步助手\图片\Typora图床\20200522155800.webp)



从图中可以看出，对于`s1`表的二级索引`idx_key1`来说，值为`NULL`的二级索引记录都被放在了`B+`树的最左边，这是因为·InnoDB`有这样的规定：

> We define the SQL null to be the smallest possible value of a field.

也就是说他们把SQL中的`NULL`值认为是列中最小的值。

在通过二级索引`idx_key1`对应的`B+`树快速定位到叶子节点中符合条件的最左边的那条记录后，也就是本例中`id`值为`521`的那条记录之后，就可以顺着每条记录都有的`next_record`属性沿着由记录组成的单向链表去获取记录了，直到某条记录的`key1`列不为NULL。

> 小贴士： 通过B+树快速定位到叶子节点的记录的过程是靠一个所谓的页目录（Page Directory）做到的，不过这不是本文的重点，大家可以到小册中翻看，都有详细解释。

## 使不使用索引的依据到底是什么？（结论）

那既然`IS NULL`、`IS NOT NULL`、`!=`这些条件都可能使用到索引，那到底什么时候索引，什么时候采用全表扫描呢？

答案就是**成本**。对于使用二级索引进行查询来说，成本组成主要有两个方面：

- 读取二级索引记录的成本
- 将二级索引记录执行回表操作，也就是到聚簇索引中找到完整的用户记录的操作所付出的成本。

很显然，要扫描的二级索引记录条数越多，那么需要执行的回表操作的次数也就越多，达到了某个比例时，使用二级索引执行查询的成本也就超过了全表扫描的成本（举一个极端的例子，比方说要扫描的全部的二级索引记录，那就要对每条记录执行一遍回表操作，自然不如直接扫描聚簇索引来的快）。

所以MySQL优化器在真正执行查询之前，对于每个可能使用到的索引来说，都会预先计算一下需要扫描的二级索引记录的数量，比方说对于下边这个查询：

```mysql
SELECT * FROM s1 WHERE key1 IS NULL;
```

优化器会分析出此查询只需要查找`key1`值为`NULL`的记录，然后访问一下二级索引`idx_key1`，看一下值为`NULL`的记录有多少（如果符合条件的二级索引记录数量较少，那么统计结果是精确的，如果太多的话，会采用一定的手段计算一个模糊的值），这种在查询真正执行前优化器就率先访问索引来计算需要扫描的索引记录数量的方式称之为`index dive`。当然，对于某些查询，比方说WHERE子句中有IN条件，并且IN条件中包含许多参数的话，比方说这样：

```mysql
SELECT * FROM s1 WHERE key1 IN ('a', 'b', 'c', ... , 'zzzzzzz');
```

这样的话需要统计的`key1`值所在的区间就太多了，这样就不能采用`index dive`的方式去真正的访问二级索引`idx_key1`，而是需要采用之前在背地里产生的一些统计数据去估算匹配的二级索引记录有多少条（很显然根据统计数据去估算记录条数比`index dive`的方式精确性差了很多）。

反正不论采用`index dive`还是依据统计数据估算，最终要得到一个需要扫描的二级索引记录条数，如果这个条数占整个记录条数的比例特别大，那么就趋向于使用全表扫描执行查询，否则趋向于使用这个索引执行查询。

理解了这个也就好理解为什么在WHERE子句中出现`IS NULL`、`IS NOT NULL`、`!=`这些条件仍然可以使用索引，本质上都是优化器去计算一下对应的二级索引数量占所有记录数量的比值而已。

---

文章来源：

>https://mp.weixin.qq.com/s/Fl96NNuFfZKj55bz2XDCTw

