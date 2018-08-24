### es结构
![ES架构](https://static.tmaczhao.cn/note/180824/8L76I4762g.png)


### 建立文档 
#### 减少副本数量
>对于每个索引请求，Elasticsearch需要将文档写入主分片和所有副本分片。显然，副本过多会减慢索引速度，但另一方面，这将提高搜索性能。

![副本数量和吞吐量关系](https://static.tmaczhao.cn/note/180824/A30EghckGA.png)
我们可以看到，随着副本数增加，吞吐量下降，响应时间增加。

#### refresh_interval
>_bulk/index/update?refresh=true  强制在请求之后刷新indexBuffer到缓存

每次刷新事件发生时，Elasticsearch都会创建一个新的segment，并在稍后进行合并。增加刷新时间间隔将降低创建和合并的开销。
![refresh_interval和吞吐量关系](https://static.tmaczhao.cn/note/180824/m6KbG6j6mJ.png)
我们可以看到，随着刷新时间间隔的增加，吞吐量增加，响应时间减少。

#### 优化建议
 - 使用批量请求，对于大量的数据请求使用批量接口，减少网络损耗
 - 使用线程池发送请求，提升应用的并发能力
 - 线程池隔离，对于创建/查询/删除等的不同请求进行线程池隔离，防止某一类请求问题拖垮其他的请求 
 - 字段不可枚举，路由
 - 字段可枚举，拆分多个索引 - http://localhost:9200/shanghai,beijing/_search?routing=food
 - 按日期建立索引
 - 注意分片数据平衡 - index.routing_partition_size
 - 确保分片均匀分布在节点上



### 搜索文档
#### Filter 替代 Query
![mark](https://static.tmaczhao.cn/note/180824/kiL15laIKj.png)

#### 增加更多的副本数量
![mark](https://static.tmaczhao.cn/note/180824/jbA8HBJLaC.png)

#### 其他
 - 检索必要的字段 - stored_fields
 - 避免搜索一些停用词，比如助词/副词，可能导致返回的文档过多
 - 如果文档返回顺序没有要求，则按 _doc 排序，默认使用“_score”
 - 避免通配符查询或者脚本查询 - （"source":"doc['num'].value.startsWith（'1234'）"）
 - 查询缓存，节点缓存/分片缓存/scroll深分页
 - https://www.ebayinc.com/stories/blogs/tech/elasticsearch-performance-tuning-practice-at-ebay/


### lucene原理
#### Posting List
假设有这么几条数据(为了简单，去掉about, interests这两个field):
| ID | Name | Age | Sex    |
| ---|  --- | --- |  ---   |
| 1  | Kate | 24  | Female |
| 2  | John | 24  | Male   |
| 3  | Bill | 29  | Male   |
ID是Elasticsearch自建的文档id，那么Elasticsearch建立的索引如下:

Name:
| Term | Posting List |
| ---  |---|
| Kate | 1 |
| John | 2 |
| Bill | 3 |

Age:
| Term | Posting List |
| ---  |---|
| 24   | [1,2] |
| 29   | 3 |

Sex:
| Term   | Posting List |
| ---    |---|
| Female | 1 |
| Male   | [2,3] |


#### Term Dictionary
Elasticsearch为了能快速找到某个term，将所有的term排个序，二分法查找term，logN的查找效率，就像通过字典查找一样，这就是Term Dictionary。


#### Term Index
B-Tree通过减少磁盘寻道次数来提高查询性能，Elasticsearch也是采用同样的思路，直接通过内存查找term，不读磁盘，但是如果term太多，term dictionary也会很大，放内存不现实，于是有了Term Index，就像字典里的索引页一样，A开头的有哪些term，分别在哪页，可以理解term index是一颗树：
![Term Index](https://static.tmaczhao.cn/note/180824/L1aaEGL4bd.png)

这棵树不会包含所有的term，它包含的是term的一些前缀。通过term index可以快速地定位到term dictionary的某个offset，然后从这个位置再往后顺序查找。
![Term Index](https://static.tmaczhao.cn/note/180824/fIifcLc49C.png)

>Term Index仅仅存储了一些前缀和Term Dictionary的block之间的关系，实际存储时使用FST(Finite State Transducer，有限状态机)存储。
![mark](https://static.tmaczhao.cn/note/180824/E59632k34f.png)

#### 压缩技巧
Elasticsearch里除了上面说到用FST压缩term index外，对posting list也有压缩技巧。

>如果Elasticsearch需要对同学的性别进行索引(这时传统关系型数据库已经哭晕在厕所……)，会怎样？如果有上千万个同学，而世界上只有男/女这样两个性别，每个posting list都会有至少百万个文档id。 Elasticsearch是如何有效的对这些文档id压缩的呢？

>增量编码压缩，将大数变小数，按字节存储

首先，Elasticsearch要求posting list是有序的(为了提高搜索的性能，再任性的要求也得满足)，这样做的一个好处是方便压缩，看下面这个图例：
![mark](https://static.tmaczhao.cn/note/180824/B49CkA8Ilj.png)
原理就是通过增量，将原来的大数变成小数仅存储增量值，再精打细算按bit排好队，最后通过字节存储，而不是大大咧咧的尽管是2也是用int(4个字节)来存储。


#### Roaring bitmaps
说到Roaring bitmaps，就必须先从bitmap说起。Bitmap是一种数据结构，假设有某个posting list：[1,3,4,7,10]
对应的bitmap就是：[1,0,1,1,0,0,1,0,0,1]
非常直观，用0/1表示某个值是否存在，比如10这个值就对应第10位，对应的bit值是1，这样用一个字节就可以代表8个文档id，旧版本(5.0之前)的Lucene就是用这样的方式来压缩的，但这样的压缩方式仍然不够高效，如果有1亿个文档，那么需要12.5MB的存储空间，这仅仅是对应一个索引字段(我们往往会有很多个索引字段)。于是有人想出了Roaring bitmaps这样更高效的数据结构。

Bitmap的缺点是存储空间随着文档个数线性增长，Roaring bitmaps需要打破这个魔咒就一定要用到某些指数特性：
将posting list按照65535为界限分块，比如第一块所包含的文档id范围在0~65535之间，第二块的id范围是65536~131071，以此类推。再用<商，余数>的组合表示每一组id，这样每组里的id范围都在0~65535内了，剩下的就好办了，既然每组id不会变得无限大，那么我们就可以通过最有效的方式对这里的id存储。
![mark](https://static.tmaczhao.cn/note/180824/89eBad4g6E.png)

>程序员的世界里除了1024外，65535也是一个经典值，因为它=2^16-1，正好是用2个字节能表示的最大数，一个short的存储单位，注意到上图里的最后一行“If a block has more than 4096 values, encode as a bit set, and otherwise as a simple array using 2 bytes per value”，如果是大块，用节省点用bitset存，小块就豪爽点，2个字节我也不计较了，用一个short[]存着方便。
>那为什么用4096来区分采用数组还是bitmap的阀值呢？这个是从内存大小考虑的，当block块里元素超过4096后，用bitmap更剩空间： 采用bitmap需要的空间是恒定的: 65536/8 = 8192bytes 而如果采用short[]，所需的空间是: 2*N(N为数组元素个数)，N=4096刚好是边界:


#### 联合索引
上面说了半天都是单field索引，如果多个field索引的联合查询，倒排索引如何满足快速查询的要求呢？
 - 利用跳表(Skip list)的数据结构快速做“与”运算，或者
 - 利用上面提到的bitset按位“与”

![](https://raw.githubusercontent.com/Neway6655/neway6655.github.com/master/images/elasticsearch-study/combineIndex.png)
如果使用跳表，对最短的posting list中的每个id，逐个在另外两个posting list中查找看是否存在，最后得到交集的结果。
如果使用bitset，就很直观了，直接按位与，得到的结果就是最后的交集。




### 参考文档
https://www.ebayinc.com/stories/blogs/tech/elasticsearch-performance-tuning-practice-at-ebay/
https://www.elastic.co/guide/en/elasticsearch/reference/current/tune-for-indexing-speed.html#_additional_optimizations
https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-refresh.html#docs-refresh
https://blog.csdn.net/u011686226/article/details/78436573

