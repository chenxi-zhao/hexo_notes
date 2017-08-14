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

| 命令 | 示例 | 解释 |
| --- | --- | --- |
| set | set name tmac | 设置键值对name=tmac |
| mset | mset key1 1 key2 2 | 设置多个键值对 |
| setnx | setnx name me | 当name不存在时设置 |
| msetnx | msetnx key1 1 key2 2 | 设置多个不存在的键值对，存在全部回滚 |
| setex | setex email 10 11@qq.com | 设置该键值对有效期10s |
| setrange | setrange email 3 126.com | 从email第三位开始更改 |
| setbit | setbit bit 100 1 | 设置bit第100位为1 |
| psetex | psetex email 10 11@qq.com | 同setex,时间单位为毫秒 |
| get | get name | 获取键为name的value |
| getset | getset name chenxi | 获取name值返回并设置新值 |
| mget | mget email name | 获取多个键对应的值 |
| getrange | getrange name 1 3 | 获取name第1到3位 |
| getbit | get bit 100 | 获取bit第100位的值 |
| incr | incr age | age必须为int，age+1 |
| incrby | incrby age 5 | age+5，非int报错 |
| incrbyfloat | incrbyfloat float 0.1 | age+0.1，类型同步为float |
| decr | decr age | age-1 |
| decrby | decrby age 5 | age-5 |
| append | append name zhao | 在键对应的值后面连接字符串 |
| strlen | strlen name | 键对应的值长度 |


#### Hash（哈希）
Redis hash 是一个键值对集合。
Redis hash是一个string类型的field和value的映射表，hash特别适合用于存储对象。
>hash-max-zipmap-entries 64 #配置字段最多64 个
hash-max-zipmap-value 512 #配置value 最大为512 字节

| 命令 | 示例 | 解释 |
| --- | --- | --- |
| hset | hset obj name tmac | 设置hash field为指定值，不存在obj先创建 |
| hmset | hmset obj key1 1 key2 2 | 为obj设置多个键值对 |
| hsetnx | hsetnx obj name me | 当name不存在时设置 |
| hget | hget obj name | 获取obj中键为name的value |
| hmget | hmget obj email name | 获取多个键对应的值 |
| hincrby | hincrby obj age 5 | age+5，非int报错 |
| hincrbyfloat | incrbyfloat obj float 0.1 | age+0.1，类型同步为float |
| hlen | hlen obj | hash中的field数量 |
| hexists | hexists obj name | 测试指定field是否存在 |
| hdel | hdel obj name | 删除指定的field |
| hkeys | hkeys obj | hash的所有field |
| hvals | hvals obj | hash中的所有value |
| hgetall | hgetall | 获取某个hash中全部的filed及value |
| hscan | | |

#### lists（链表）
list 是一个链表结构，主要功能是push、pop、获取一个范围的所有值等等，操作中key理解为链表的名字。

Redis的list类型其实就是一个每个子元素都是string 类型的双向链表。链表的最大长度是(2的32次方)。我们可以通过push,pop 操作从链表的头部或者尾部添加删除元素。这使得list既可以用作栈，也可以用作队列。

| 命令 | 示例 | 解释 |
| --- | --- | --- |
| lpush | lpush mylist World | 将一个或多个值插入到列表头部 |
| lpushx | lpushx mylist Hello there | 将一个或多个值插入到已存在的列表头部 |
| rpush | rpush mylist man | 在列表尾部添加一个或多个值 |
| rpushx | rpush mylist !!!! | 在已存在列表尾部添加一个或多个值 |
| lpop | lpop mylist| 移出并获取列表的第一个元素 |
| rpop | rpop mylist | 移除并获取列表最后一个元素 |
| lrange | lrange mylist 0 -1 | 获取列表指定范围内的元素 |
| linsert | linsert mylist before/after "World" there | 在列表的元素前或者后插入元素 |
| lrem | lrem mylist 1 there | 移除count个值为value的元素，count>0从头到尾，count<0反之，等于0全部移除 |
| lset | lset mylist 0 hello| 通过索引设置列表元素的值 |
| ltrim | ltrim mylist 1 -1| 让列表只保留指定区间内的元素 |
| rpoplpush | rpoplpush mylist5 mylist6 | 移除列表的尾部元素，并添加到另一个列表头 |
| lindex | lindex mylist 1 | 通过索引获取列表中的元素 |
| llen | llen | 获取列表长度 |
| blpop | blpop list1 .. listn timeout | 移出并获取列表的头部元素，没有则阻塞或等待元素或者超时 |
| brpop | blpop list1 .. listn timeout | 移出并获取列表的尾部元素，没有则阻塞或等待元素或者超时 |
| brpoplpush | brpoplpush msg reciver 500 | 移除列表的尾部元素，并添加到另一个列表头，阻塞同上 |


#### sets（集合）
Redis的set是string类型的无序集合。set元素最大可以包含(2的32次方)个元素。
set的是通过hashtable实现的，所以添加、删除和查找的复杂度都是O(1)。hashtable会随着添加或者删除自动的调整大小。

| 命令 | 示例 | 解释 |
| --- | --- | --- |
| sadd | sadd myset hello world | 向集合添加一个或多个成员 |
| smembers | smembers myset | 返回集合中所有元素 |
| srem | srem myset hello | 删除myset中的hello元素 |
| spop | spop myset| 随机弹出myset中一个元素 |
| sdiff | sdiff myset myset2| 返回给定集合的差集(myset中不同的) |
| sdiffstore | sdiff storeset myset myset2 | 返回给定集合的差集并存储 |
| sinter | sinter myset myset2| 返回给定集合的交集 |
| sinterstore | sinter storeset myset myset2 | 返回给定集合的交集并存储 |
| sunion | sunion myset myset2 | 返回给定集合的并集 |
| sunionstore | sunion storeset myset myset2 | 返回给定集合的并集并存储 |
| smove | smove myset myset2 hello| 将hello从myset移动到myset2 |
| scard | scard myset| 获取集合的成员数 |
| sismember | sismember myset hello| 判断hello是否是集合myset的成员 |
| srandmember | srandmember myset 2| 随机返回myset中的2个元素，不删除 |
| sscan | | |


#### sorted set（有序集合）
sorted set是set的一个升级版本，它在set的基础上增加了一个顺序属性，这一属性在添加修改元素的时候可以指定，每次指定后，zset会自动重新按新的值调整顺序。可以理解为有两列的mysql表，一列存value，一列存顺序。操作中key理解为zset的名字。
和set一样sorted set也是string类型元素的集合，不同的是每个元素都会关联一个double类型的score。sorted set的实现是skip list和hash table的混合体。

| 命令 | 示例 | 解释 |
| --- | --- | --- |
| zadd | zadd key sco1 mem1 [sco2 mem2] |向有序集合添加一个或多个成员，或者更新已存在成员的分数|
| zcard | zcard key | 获取有序集合的成员数 |
| zcount | zcount key sco-min sco-max | 计算在有序集合中指定区间分数的成员数 |
| zincrby | zincrby key incr mem | 有序集合中对指定成员的分数加上增量incr |
| zscore | zscore key mem | 返回有序集合中mem元素的score |
| zlexcount | zlexcount key lex-min lex-max | 计算有序集合中指定字典区间的成员数 |
| zrange | zrange key idx1 idx2 [withscores] | 返回索引区间内的所有元素，分数从低到高 |
| zrangebyscore | zrangebyscore key min max [withscores] [limit] | 返回指定分数区间元素 |
| zrangebylex | zrangebylex key min max [limit offset count] | 返回指定字典区间的元素 |
| zreverange | zrevrange key idx1 idx2 [withscores] | 返回指定区间元素，分数从高到低 |
| zrevrangebyscore | zrevrangebyscore key max min [withscores] | 返回指定从高到低分数区间元素 |
| zrem | zrem key mem1 [mem2]| 删除有序集合中指定元素 |
| zremrangebylex | zremrangebylex key min max  | 删除指定字典区间的元素 |
| zremrangebyrank | zremrangebyrank key start stop | 删除指定索引区间的元素 |
| zremrangebyscore | zremrangebyscore key min max  | 删除指定分数区间的元素 |
| zrank | zrank key member | 返回集合中元素索引，按分数从小到大排序 |
| zrevrank | zrevrank key member | 返回集合中元素索引，分数从大到小排序 |
| zunionstore | zunionstore destination numkeys key [key ...] | 计算指定有序集合的并集，并存储 |
| zinterstore | zinterstore destination numkeys key [key ...]  | 计算制定有序集合的交集，并存储 |
| zscan | | |

#### key（Redis键命令）
| 命令 | 解释 |
| --- | --- |
| DEL key | 该命令用于在key存在是删除key。 |
| DUMP key | 序列化给定key，并返回被序列化的值。 |
| EXISTS key | 检查给定key是否存在。 |
| EXPIRE key seconds | 为给定key设置过期时间。 |
| EXPIREAT key timestamp | 和EXPIRE类似，不同在于EXPIREAT接受的时间参数是UNIX时间戳(unix timestamp)|
| PEXPIRE key milliseconds  | 设置 key 的过期时间亿以毫秒计。 |
| PEXPIREAT key milliseconds-timestamp  | 设置 key 过期时间的时间戳(unix timestamp) 以毫秒计 |
| KEYS pattern  | 查找所有符合给定模式( pattern)的 key 。 |
| MOVE key db  | 将当前数据库的 key 移动到给定的数据库 db 当中。
| PERSIST key  | 移除 key  |的过期时间，key 将持久保持。
| PTTL key  | 以毫秒为单位返回 key 的剩余的过期时间。 |
| TTL key  | 以秒为单位，返回给定 key 的剩余生存时间(TTL, time to live)。 |
| RANDOMKEY  | 从当前数据库中随机返回一个 key 。 |
| RENAME key newkey  | 修改 key 的名称 |
| RENAMENX key newkey  | 仅当 newkey 不存在时，将 key 改名为 newkey 。 |
| TYPE key  | 返回 key 所储存的值的类型。 |

#### HyperLogLog
[redis数据结构HyperLogLog](http://www.cnblogs.com/ysuzhaixuefei/p/4052110.html)

#### 发布和订阅
| 命令 | 解释 |
| --- | --- |
| psubscribe pattern [pattern ...] | 订阅一个或多个符合给定模式的频道。 |
| pubsub subcommand [argument [argument ...]] | 查看订阅与发布系统状态。 |
| publish channel message | 将信息发送到指定的频道。 |
| punsubscribe [pattern [pattern ...]] | 退订所有给定模式的频道。 |
| subscribe channel [channel ...] | 订阅给定的一个或多个频道的信息。 |
| unsubscribe [channel [channel ...]] | 指退订给定的频道。 |

```
//订阅频道
redis 127.0.0.1:6379> SUBSCRIBE redisChat

//向频道发送消息
redis 127.0.0.1:6379> PUBLISH redisChat "Redis is a great caching technique"
(integer) 1
redis 127.0.0.1:6379> PUBLISH redisChat "Learn redis by w3cschool.cc"
(integer) 1
```

#### 事务
| 命令 | 解释 |
| --- | --- |
| discard | 取消事务，放弃执行事务块内的所有命令。 |
| exec | 执行所有事务块内的命令。 |
| multi | 标记一个事务块的开始。 |
| unwatch | 取消 watch 命令对所有 key 的监视。 |
| watch key [key ...] | 监视一个(或多个)key，如果在事务执行前这些key被其他命令改动，那么事务将被打断。 |

```
redis 127.0.0.1:6379> multi
OK
redis 127.0.0.1:6379> set age 10
QUEUED
redis 127.0.0.1:6379> set age 20
QUEUED
redis 127.0.0.1:6379> exec (执行)/ discard（取消）
...
```

#### 持久化
##### snapshotting
快照是默认的持久化方式。这种方式是就是将内存中数据以快照的方式写入到二进制文件中,默认的文件名为dump.rdb。可以通过配置设置自动做快照持久化的方式。我们可以配置redis在n秒内如果超过m个key被修改就自动做快照，下面是默认的快照保存配置：
>save 900 1 #900 秒内如果超过1 个key 被修改，则发起快照保存
save 300 10 #300 秒内容如超过10 个key 被修改，则发起快照保存
save 60 10000

##### aof(Append-Only file)
在使用aof 持久化方式时,redis会将每一个收到的写命令都通过write 函数追加到文件中(默认是appendonly.aof)。当redis重启时会通过重新执行文件中保存的写命令来在内存中重建整个数据库的内容。当然由于os会在内核中缓存write做的修改，所以可能不是立即写到磁盘上。这样aof方式的持久化也还是有可能会丢失部分修改。不过我们可以通过配置文件告诉redis我们想要通过fsync函数强制os写入到磁盘的时机。有三种方式如下（默认是：每秒fsync一次）
```
appendonly yes //启用aof 持久化方式
# appendfsync always //收到写命令就立即写入磁盘，最慢，但是保证完全的持久化
appendfsync everysec //每秒钟写入磁盘一次，在性能和持久化方面做了很好的折中
# appendfsync no //完全依赖os，性能最好,持久化没保证
```
aof的方式也同时带来了另一个问题，持久化文件会变的越来越大。为了压缩aof的持久化文件，redis提供了bgrewriteaof命令。收到此命令redis 将使用与快照类似的方式将内存中的数据以命令的方式保存到临时文件中，最后替换原来的文件。

#### Pipeline管道技术
打包redis-cli到redis-server的tcp请求，交给server处理，然后合并处理结果到同一个tcp报文中，以此提高执行效率。

```java
public class TestPipeline {
    public static void main(String[] args) {
        long start = System.currentTimeMillis();
        //采用pipeline 方式发送指令
        usePipeline();
        long end = System.currentTimeMillis();
        System.out.println("用pipeline 方式耗时：" + (end - start) + "毫秒");
        start = System.currentTimeMillis();
        //普通方式发送指令
        withoutPipeline();
        end = System.currentTimeMillis();
        System.out.println("普通方式耗时：" + (end - start) + "毫秒");
    }
    //采用pipeline 方式发送指令
    private static void usePipeline() {
        try {
            ConnectionSpec spec = DefaultConnectionSpec.newSpec(
            "192.168.115.170", 6379, 0, null);
            JRedis jredis = new JRedisPipelineService(spec);
            for (int i = 0; i < 100000; i++) {
                jredis.incr("test2");
            }
            jredis.quit();
        } catch (Exception e) {
        }
    }
    //普通方式发送指令
    private static void withoutPipeline() {
        try {
            JRedis jredis = new JRedisClient("192.168.115.170", 6379);
            for (int i = 0; i < 100000; i++) {
                jredis.incr("test2");
            }
            jredis.quit();
        } catch (Exception e) {
        }
    }
}
```

#### 虚拟内存技术
vm-enabled yes #开启vm 功能
vm-swap-file /tmp/redis.swap #交换出来的value 保存的文件路径
vm-max-memory 1000000 #redis 使用的最大内存上限
vm-page-size 32 #每个页面的大小32 个字节
vm-pages 134217728 #最多使用多少页面
vm-max-threads 4 #用于执行value 对象换入换出的工作线程数量

#### 主从复制
##### redis主从复制特点:
- master可以拥有多个slave
- 多个slave可以连接同一个master外，还可以连接到其他slave
- 主从复制不会阻塞master，在同步数据时，master可以继续处理client请求
- 提高系统的伸缩性

##### redis主从复制过程:
当配置好slave后，slave与master建立连接，然后发送sync命令。无论是第一次连接还是重新连接，master都会启动一个后台进程，将数据库快照保存到文件中，同时master主进程会开始收集新的写命令并缓存。后台进程完成写文件后，master就发送文件给slave，slave将文件保存到硬盘上，再加载到内存中，接着master就会把缓存的命令转发给slave，后续master将收到的写命令发送给slave。如果master同时收到多个slave发来的同步连接命令，master只会启动一个进程来写数据库镜像，然后发送给所有的slave。
```
slaveof 192.168.1.1 6379 #指定master 的ip 和端口
```





















