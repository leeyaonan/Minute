# SQL语句

> 摘自泰山版《Java开发手册》-MySQL数据库

1. 【强制】不要使用 count(列名)或 count(常量)来替代 count(*)，count(*)是 SQL92 定义的标

准统计行数的语法，跟数据库无关，跟 NULL 和非 NULL 无关。

> 说明：count(*)会统计值为 NULL 的行，而 count(列名)不会统计此列为 NULL 值的行。

2. 【强制】count(distinct col) 计算该列除 NULL 之外的不重复行数，注意 count(distinct col1, 

col2) 如果其中一列全为 NULL，那么即使另一列有不同的值，也返回为 0。

3. 【强制】当某一列的值全是 NULL 时，count(col)的返回结果为 0，但 sum(col)的返回结果为

NULL，因此使用 sum()时需注意 NPE 问题。

正例：可以使用如下方式来避免 sum 的 NPE 问题：SELECT IFNULL(SUM(column), 0) FROM table;

4. 【强制】使用 ISNULL()来判断是否为 NULL 值。

> 说明：NULL 与任何值的直接比较都为 NULL。 

1） NULL<>NULL 的返回结果是 NULL，而不是 false。 

2） NULL=NULL 的返回结果是 NULL，而不是 true。 

3） NULL<>1 的返回结果是 NULL，而不是 true。

反例：在 SQL 语句中，如果在 null 前换行，影响可读性。select * from table where column1 is null and 

column3 is not null; 而`ISNULL(column)`是一个整体，简洁易懂。从性能数据上分析，`ISNULL(column)`

执行效率更快一些。

5. 【强制】代码中写分页查询逻辑时，若 count 为 0 应直接返回，避免执行后面的分页语句。

6. 【强制】不得使用外键与级联，一切外键概念必须在应用层解决。

> 说明：（概念解释）学生表中的 student_id 是主键，那么成绩表中的 student_id 则为外键。如果更新学

生表中的 student_id，同时触发成绩表中的 student_id 更新，即为级联更新。外键与级联更新适用于单机

低并发，不适合分布式、高并发集群；级联更新是强阻塞，存在数据库更新风暴的风险；外键影响数据库

的插入速度。

7. 【强制】禁止使用存储过程，存储过程难以调试和扩展，更没有移植性。

8. 【强制】数据订正（特别是删除或修改记录操作）时，要先 select，避免出现误删除，确认无

误才能执行更新语句。

 39/57Java 开发手册

9. 【强制】对于数据库中表记录的查询和变更，只要涉及多个表，都需要在列名前加表的别名（或

表名）进行限定。

> 说明：对多表进行查询记录、更新记录、删除记录时，如果对操作列没有限定表的别名（或表名），并且

操作列在多个表中存在时，就会抛异常。

正例：select t1.name from table_first as t1 , table_second as t2 where t1.id=t2.id;

反例：在某业务中，由于多表关联查询语句没有加表的别名（或表名）的限制，正常运行两年后，最近在

某个表中增加一个同名字段，在预发布环境做数据库变更后，线上查询语句出现出 1052 异常：Column 

'name' in field list is ambiguous。

10.【推荐】SQL 语句中表的别名前加 as，并且以 t1、t2、t3、...的顺序依次命名。

> 说明：1）别名可以是表的简称，或者是根据表出现的顺序，以 t1、t2、t3 的方式命名。2）别名前加 as

使别名更容易识别。

正例：select t1.name from table_first as t1, table_second as t2 where t1.id=t2.id;

11.【推荐】in 操作能避免则避免，若实在避免不了，需要仔细评估 in 后边的集合元素数量，控

制在 1000 个之内。

12.【参考】因国际化需要，所有的字符存储与表示，均采用 utf8 字符集，那么字符计数方法需 

要注意。

> 说明：

 SELECT LENGTH("轻松工作")； 返回为 12

 SELECT CHARACTER_LENGTH("轻松工作")； 返回为 4

 如果需要存储表情，那么选择 utf8mb4 来进行存储，注意它与 utf8 编码的区别。

13.【参考】TRUNCATE TABLE 比 DELETE 速度快，且使用的系统和事务日志资源少，但 TRUNCATE

无事务且不触发 trigger，有可能造成事故，故不建议在开发代码中使用此语句。

> 说明：TRUNCATE TABLE 在功能上与不带 WHERE 子句的 DELETE 语句相同。