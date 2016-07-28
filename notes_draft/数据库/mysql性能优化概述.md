SQL优化主要有SQL语句及索引优化，数据库结构优化，系统配置优化以及服务器硬件优化

### SQL优化

#### 慢查询
1、查看mysql是否开启慢查询日志
```mysql
show variables like 'slow_query_log';
set global slow_query_log=on;
```

2、设置没有索引的记录到慢查询日志
```mysql
show variables like 'log_queries_not_using_indexes';
set global log_queries_not_using_indexes=on;
```

3、查看超过多长时间的sql进行记录到慢查询日志
```mysql
show variables like 'long_query_time';
set global long_query_time = 1; # (1秒)
```

4、查看慢查询记录文件位置
show variables like 'slow_query_log_file';
set global slow_query_log_file='D:/mysql/data/mysql-slow.log';

>查看数据库系统属性值：show variables like '%log%';

