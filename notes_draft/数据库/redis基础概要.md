### Redis适用场景
1. 取最新N个数据的操作
2. 排行榜应用，取TOP N操作
3. 需要精确设定过期时间的应用
4. 计数器应用
5. Uniq操作，获取某段时间所有数据排重值
6. 实时系统，反垃圾系统
7. Pub/Sub构建实时消息系统——消息的发布与订阅
8. 构建队列系统
9. 缓存
### redis数据类型
Redis支持五种数据类型：string（字符串），hash（哈希），list（列表），set（集合）及zset(sorted set：有序集合)

#### String（字符串）
string是redis最基本的类型，你可以理解成与Memcached一模一样的类型，一个key对应一个value。
string类型是二进制安全的。意思是redis的string可以包含任何数据。比如jpg图片或者序列化的对象 。
string类型是Redis最基本的数据类型，一个键最大能存储512MB。
```redis
127.0.0.1:6379> SET key "value"
OK
127.0.0.1:6379> GET key
"value"
```
#### Hash（哈希）
Redis hash 是一个键值对集合。
Redis hash是一个string类型的field和value的映射表，hash特别适合用于存储对象。

