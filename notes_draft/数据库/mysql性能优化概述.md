SQL优化主要有SQL语句及索引优化，数据库结构优化，系统配置优化以及服务器硬件优化

### SQL优化

#### 慢查询
##### 查看mysql是否开启慢查询日志
```mysql
show variables like 'slow_query_log';
set global slow_query_log=on;
```

##### 设置没有索引的记录到慢查询日志
```mysql
show variables like 'log_queries_not_using_indexes';
set global log_queries_not_using_indexes=on;
```

##### 查看超过多长时间的sql进行记录到慢查询日志
```mysql
show variables like 'long_query_time';
set global long_query_time = 1; # (1秒)
```

##### 查看慢查询记录文件位置
show variables like 'slow_query_log_file';
set global slow_query_log_file='D:/mysql/data/mysql-slow.log';

#### explain
EXPLAIN 的每个输出行提供一个表的相关信息,并且每个行包括下面的列:

##### id
MySQL Query Optimizer 选定的执行计划中查询的序列号。表示查询中执行 select 子句或操作表的顺序,id值越大优先级越高,越先被执行。id 相同,执行顺序由上至下。

##### select_type 查询类型
- **SIMPLE**    简单的select查询,不使用union及子查询
- **PRIMARY**    最外层的select查询
- **UNION**    UNION中的第二个或随后的select查询,不依赖于外部查询的结果集
- **DEPENDENT UNION**    UNION中的第二个或随后的select查询,依赖于外部查询的结果集
- **SUBQUERY**    子查询中的第一个select查询,不依赖于外部查询的结果集
- **DEPENDENT SUBQUERY**    子查询中的第一个select查询,依赖于外部查询的结果集
- **DERIVED**    用于from子句里有子查询的情况。MySQL会递归执行这些子查询,把结果放在临时表里。
- **UNCACHEABLE SUBQUERY**    结果集不能被缓存的子查询,必须重新为外层查询的每一行进行评估。
- **UNCACHEABLE UNION**    UNION中的第二个或随后的select查询,属于不可缓存的子查询

##### table 输出行所引用的表

##### type
重要的项,显示连接使用的类型,按最 优到最差的类型排序
- **system**    表仅有一行(=系统表)。这是 const 连接类型的一个特例。
- **const**    const 用于用常数值比较 PRIMARY KEY 时。当查询的表仅有一行时,使用System。
- **eq_ref**    const 用于用常数值比较 PRIMARY KEY 时。当查询的表仅有一行时,使用System。
- **ref**    连接不能基于关键字选择单个行,可能查找到多个符合条件的行。 叫做 ref 是因为索引要 跟某个参考值相比较。这个参考值或者是一个常数,或者是来自一个表里的多表查询的 结果值
- **ref_or_null**    如同ref,但是 MySQL 必须在初次查找的结果里找出 null 条目,然后进行二次查找。
- **index_merge**    说明索引合并优化被使用了。
- **unique_subquery**
在某些 IN 查询中使用此种类型,而不是常规的 ref:value IN (SELECT primary_key FROM single_table WHERE some_expr)
- **index_subquery**
在某些IN查询中使用此种类型,与unique_subquery 类似,但是查询的是非唯一性索引: value IN (SELECT key_column FROM single_table WHERE some_expr)
- **range**
只检索给定范围的行,使用一个索引来选择 行。key 列显示使用了哪个索引。当使用=、 <>、>、>=、<、<=、IS NULL、<=>、BETWEEN 或者 IN 操作符,用常量比较关键字列时,可 以使用 range。
- **index**    全表扫描,只是扫描表的时候按照索引次序 进行而不是行。主要优点就是避免了排序, 但是开销仍然非常大。
- **all**    最坏的情况,从头到尾全表扫描。

##### possible_keys
指出 MySQL 能在该表中使用哪些索引有助于 查询。如果为空,说明没有可用的索引。

##### key
MySQL实际从 possible_key 选择使用的索引。如果为 NULL,则没有使用索引。很少的情况下,MYSQL会选择优化不足的索引。这种情况下,可以在 SELECT 语句中使用 USE INDEX (indexname)来强制使用一个索引或者用 IGNORE INDEX(indexname)来强制 MYSQL 忽略索引

##### key_len
使用的索引的长度。在不损失精确性的情况 下,长度越短越好。

##### ref
显示索引的哪一列被使用了

##### rows
MYSQL 认为必须检查的用来返回请求数据的行数

##### rows
MYSQL 认为必须检查的用来返回请求数据的行数

##### extra
出现以下 2 项意味着 MYSQL 根本不能使用索引,效率会受到重大影响。应尽可能对此进行优化。
- **Using filesort**  表示 MySQL 会对结果使用一个外部索引排序,而不是从表里按索引次序读到相关内容。可能在内存或者磁盘上进行排序。MySQL 中无法利用索引完成的排序操作称为“文件排序”
- **Using temporary**     表示 MySQL 在对查询结果排序时使用临时表。常见于排序 order by 和分组查询 group by。

>查看数据库系统属性值：show variables like '%log%';


#### Count()、Max()、子查询
索引、字段null值影响count(*)和count(id)、子查询/join






























