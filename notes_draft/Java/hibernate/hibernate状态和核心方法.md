对象状态转移图
![](http://static.tmaczhao.cn/images/215e5c24705e22ad984f82bbefcdb109.jpg)


### 持久化对象的状态
站在持久化的角度，Hibernate把对象分为4种状态：持久化状态，临时状态，游离状态，删除状态。Session的特定方法能使对象从一个状态转换到另一个状态。
- 临时对象（Transient）：OID通常为null，不处于Session的缓存中，在数据库中没有对应的记录。

- 持久化对象(也叫”托管”)（Persist）：OID不为null，位于Session缓存中，若在数据库中已经有和其对应的记录，持久化对象和数据库中的相关记录对应，Session在flush缓存时，会根据持久化对象的属性变化，来同步更新数据库，在同一个Session实例的缓存中，数据库表中的每条记录只对应唯一的持久化对象。

- 删除对象(Removed)：在数据库中没有和其OID对应的记录，不再处于Session缓存中，一般情况下，应用程序不该再使用被删除的对象。

- 游离对象(也叫”脱管”)（Detached）：OID不为null，不再处于Session缓存中，一般情况需下，游离对象是由持久化对象转变过来的，因此在数据库中可能还存在与它对应的记录。


### 核心方法
#### Session的save()方法
>Session的save()方法使一个临时对象转变为持久化对象

Session的save()方法完成以下操作：
- 把New的对象加入到Session缓存中，使它进入持久化状态
- 选用映射文件指定的标识符生成器，为持久化对象分配唯一的OID。在使用代理主键的情况下，setId()方法为News对象设置OID是无效的。
- 计划执行一条insert语句：在flush缓存的时候
- Hibernate 通过持久化对象的 OID 来维持它和数据库相关记录的对应关系。当 News 对象处于持久化状态时，不允许程序随意修改它的 ID

persist()和save()区别：
当对一个OID不为Null的对象执行save()方法时，会把该对象以一个新的oid保存到数据库中；但执行persist()方法时会抛出一个异常。

#### get和load方法
- 相同点：
    都可以根据跟定的 OID 从数据库中加载一个持久化对象
- 区别：
    1. 当数据库中不存在与OID对应的记录时，load()方法抛出ObjectNotFoundException异常，而get()方法返回null。
    2. 两者采用不同的延迟检索策略：load默认使用懒加载策略，除非将相应对象的获取策略设为即时加载。而get只要调用了，都会立即从数据库中加载数据。

#### Session的update()方法
Session的update()方法使一个游离对象转变为持久化对象，并且计划执行一条update语句。
- 当update()方法关联一个游离对象时，如果在Session的缓存中已经存在相同OID的持久化对象，会抛出异常
- 当update()方法关联一个游离对象时，如果在数据库中不存在相应的记录，也会抛出异常

#### saveOrUpdate方法
Session的saveOrUpdate()可以看成是save和update的整合,两者形成有效互补。
- 当操作对象是瞬时状态时，保存对象并将对象缓存到session中去。
- 当操作对象是游离状态，即有标识符（IDENTIFIER,id）时，若id数据库中存在则更新，不存在抛错。
>当对象oid为空或者oid取值与unsaved-value属性值匹配时认定为瞬时状态。

#### Session的merge()方法
![](http://static.tmaczhao.cn/images/ae5b2182c3c91be29966b5c0c6ba032a.jpg)

#### Session 的 delete() 方法
**Session的delete()方法既可以删除一个游离对象，也可以删除一个持久化对象**

Session的delete()方法处理过程：
- 计划执行一条delete语句（在flush时正式执行）
- 把对象从Session缓存中删除，该对象进入删除状态。

Hibernate的cfg.xml配置文件中有一个hibernate.use_identifier_rollback属性，其默认值为false，若把它设为true，将改变delete()方法的运行行为：delete()方法会把持久化对象或游离对象的OID设置为null，使它们变为临时对象。这样程序就可以重复利用这些对象了


### Hibernate 与触发器协同工作
Hibernate与数据库中的触发器协同工作时, 会造成两类问题：
1. 触发器使Session的缓存中的持久化对象与数据库中对应的数据不一致：触发器运行在数据库中，它执行的操作对Session是透明的
2. Session的update()方法盲目地激发触发器：无论游离对象的属性是否发生变化，都会执行update语句，而update语句会激发数据库中相应的触发器

解决方案：
在执行完Session的相关操作后，立即调用Session的flush()和refresh()方法，迫使Session的缓存与数据库同步(refresh()方法重新从数据库中加载对象)
![](http://static.tmaczhao.cn/images/89f039a76db2275b675c69e08fe2e7bb.jpg)

>在映射文件的的<class>元素中设置select-before-update属性：当Session update或saveOrUpdate()方法更新一个游离对象时，会先执行Select语句，获得当前游离对象在数据库中的最新数据，只有在不一致的情况下才会执行update语句。







