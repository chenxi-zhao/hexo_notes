### MyBatis的框架设计
![](http://static.tmaczhao.cn/images/df45418c35ef3d9e2372dc42ecc7e1d4.jpg)

#### 接口层---和数据库交互的方式
MyBatis和数据库的交互有两种方式：
- 使用传统的MyBatis提供的API（selectOne、selectList）；
- 使用Mapper接口（面向接口的配置）

##### 使用传统的MyBatis提供的API
![](http://static.tmaczhao.cn/images/0487eecbc252def5f3b1a8a86261fcac.jpg)

这是传统的传递Statement Id和查询参数给SqlSession对象，使用SqlSession对象完成和数据库的交互；MyBatis提供了非常方便和简单的API，供用户实现对数据库的增删改查数据操作，以及对数据库连接信息和MyBatis自身配置信息的维护操作。

##### 使用Mapper接口
![](http://static.tmaczhao.cn/images/671a8abe8f6136ff75a7cc9be050bed6.jpg)

根据MyBatis的配置规范配置好后，通过SqlSession.getMapper(XXXMapper.class)方法，MyBatis会根据相应接口声明的方法信息，通过动态代理机制生成一个Mapper实例。
>Mybatis通过动态代理为每一个接口生成一个MapperProxy的代理类，通过在其中对于MapperMethod的调用最终还是会调用selectList这类方法获取结果集。

我们使用Mapper接口的某一个方法时，MyBatis会根据这个方法的方法名和参数类型，确定Statement Id，底层还是通过SqlSession.select("statementId",parameterObject);或者SqlSession.update("statementId",parameterObject);等等来实现对数据库的操作。


#### 数据处理层
数据处理层可以说是MyBatis的核心，从大的方面上讲，它要完成两个功能：
- 通过传入参数构建动态SQL语句；
- SQL语句的执行以及封装查询结果集成List<E>

1. 参数映射和动态SQL语句生成
- 动态语句生成可以说是MyBatis框架非常优雅的一个设计，MyBatis通过传入的参数值，使用`Ognl`来动态地构造SQL语句，使得MyBatis有很强的灵活性和扩展性。
- 参数映射指的是对于Java数据类型和jdbc数据类型之间的转换：这里有包括两个过程：查询阶段，我们要将java类型的数据，转换成jdbc类型的数据，通过preparedStatement.setXXX()来设值；另一个就是对resultset查询结果集的jdbcType数据转换成java数据类型。

2. SQL语句的执行以及封装查询结果集成List<E>
动态SQL语句生成之后，MyBatis将执行SQL语句，并将可能返回的结果集转换成List<E> 列表。MyBatis在对结果集的处理中，支持结果集关系一对多和多对一的转换，并且有两种支持方式，一种为嵌套查询语句的查询，还有一种是嵌套结果集的查询。


#### 框架支撑层
- 事务管理机制
- 连接池管理机制
- 缓存机制
- SQL语句的配置方式(面向xml配置以及面向接口编程)

#### MyBatis的主要构件及其相互关系
![](http://static.tmaczhao.cn/images/5cc78d05f4eb8bea5024fb82b7dba2b4.jpg)




### Mybatis初始化机制
MyBatis采用了一个非常直白和简单的方式---使用`org.apache.ibatis.session.Configuration`对象作为一个所有配置信息的容器，Configuration对象的组织结构和XML配置文件的组织结构几乎完全一样（当然，Configuration对象的功能并不限于此，它还负责创建一些MyBatis内部使用的对象，如Executor等，这将在后续的文章中讨论）。如下图所示：
![](http://static.tmaczhao.cn/images/d4319a76a48d8349ffec968e9c5db197.jpg)

>可以这么说，MyBatis初始化的过程，就是创建Configuration对象的过程。
- 基于XML配置文件：基于XML配置文件的方式是将MyBatis的所有配置信息放在XML文件中，MyBatis通过加载并XML配置文件，将配置文信息组装成内部的Configuration对象
- 基于Java API：这种方式不使用XML配置文件，需要MyBatis使用者在Java代码中，手动创建Configuration对象，然后将配置参数set进入Configuration对象中

![](http://static.tmaczhao.cn/images/3b82c5ad13cd62a628f66fa07eb59647.jpg)
由上图所示，mybatis初始化要经过简单的以下几步：
1. 调用SqlSessionFactoryBuilder对象的build(inputStream)方法；
2. SqlSessionFactoryBuilder会根据输入流inputStream等信息创建XMLConfigBuilder对象;
3. SqlSessionFactoryBuilder调用XMLConfigBuilder对象的parse()方法；
4. XMLConfigBuilder对象返回Configuration对象；
5. SqlSessionFactoryBuilder根据Configuration对象创建一个DefaultSessionFactory对象；
6. SqlSessionFactoryBuilder返回 DefaultSessionFactory对象给Client，供Client使用。


### mybatis数据源
mybatis中将数据源分成三种：
- UNPOOLED    不使用连接池的数据源
- POOLED      使用连接池的数据源
- JNDI        使用JNDI实现的数据源

![](http://static.tmaczhao.cn/images/377e1e12f0200e9e0e74de7c76a59b99.jpg)

MyBatis在初始化时，解析配置文件，根据<dataSource\>的type属性来创建相应类型的的数据源DataSource。

>MyBatis是通过工厂方法模式来创建数据源DataSource对象的，MyBatis定义了抽象的工厂接口:org.apache.ibatis.datasource.DataSourceFactory,通过其getDataSource()方法返回数据源DataSource。

![](http://static.tmaczhao.cn/images/c2f6a6d86104109ed9e783dd38acf4f5.jpg)

#### UnpooledDataSource
![](http://static.tmaczhao.cn/images/4b721a95a83c26aedbd1cbc7b5c6f9ca.jpg)
```java
/*
UnpooledDataSource的getConnection()实现
*/
public Connection getConnection() throws SQLException{
    return doGetConnection(username, password);
}

private Connection doGetConnection(String username, String password) throws SQLException{
    //封装username和password成properties
    Properties props = new Properties();
    if (driverProperties != null){
        props.putAll(driverProperties);
    }
    if (username != null){
        props.setProperty("user", username);
    }
    if (password != null){
        props.setProperty("password", password);
    }
    return doGetConnection(props);
}

/*
 *  获取数据连接
 */
private Connection doGetConnection(Properties properties) throws SQLException{
    //1.初始化驱动
    initializeDriver();
    //2.从DriverManager中获取连接，获取新的Connection对象
    Connection connection = DriverManager.getConnection(url, properties);
    //3.配置connection属性
    configureConnection(connection);
    return connection;
}
```

#### PooledDataSource
![](http://static.tmaczhao.cn/images/4f93eaf7f9ca6f4e31107ae52c4a1a5c.jpg)

PooledDataSource维护了一个UnpooledDataSource对象和一个PooledState对象。
- UnpooledDataSource主要用来创建新的Connection
- PooledState对象则是用来保存连接池的信息，该对象中包含了DataSource、以及一个idleConnections和一个activeConnections列表，获取连接的机制如上。

通过该数据源获取的连接并不是纯粹的Connection，而是它的一个包装代理类PooledConnection，这个类的作用除了保存连接的时间、使用等状态外，还代理了原连接的方法。其作用主要是为了在调用关闭连接时不真的关闭它，而是将其放入连接池中去。
```java
@Override
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    String methodName = method.getName();
    if (CLOSE.hashCode() == methodName.hashCode() && CLOSE.equals(methodName)) {
      dataSource.pushConnection(this);
      return null;
    } else {
      try {
        if (!Object.class.equals(method.getDeclaringClass())) {
          // issue #579 toString() should never fail
          // throw an SQLException instead of a Runtime
          checkConnection();
        }
        return method.invoke(realConnection, args);
      } catch (Throwable t) {
        throw ExceptionUtil.unwrapThrowable(t);
      }
    }
}
```


### MyBatis事务管理机制
对数据库的事务而言，应该具有以下几点：创建（create）、提交（commit）、回滚（rollback）、关闭（close）。对应地，MyBatis将事务抽象成了Transaction接口：其接口定义如下：
```java
public interface Transaction {

  Connection getConnection() throws SQLException;

  void commit() throws SQLException;

  void rollback() throws SQLException;

  void close() throws SQLException;

  Integer getTimeout() throws SQLException;

}
```
MyBatis的事务管理分为两种形式：
- 使用JDBC的事务管理机制：即利用java.sql.Connection对象完成对事务的提交（commit()）、回滚（rollback()）、关闭（close()）等
- 使用MANAGED的事务管理机制：这种机制MyBatis自身不会去实现事务管理，而是让程序的容器如（JBOSS，Weblogic）来实现对事务的管理

MyBatis事务的创建是交给TransactionFactory事务工厂来创建的：
- 如果我们将<transactionManager\>的type配置为"JDBC"，那么，在MyBatis初始化解析<environment\>节点时，会根据type="JDBC"创建一个JdbcTransactionFactory工厂。
- 如果type="MANAGED"，则MyBatis会创建一个MangedTransactionFactory.class实例。

#### 事务工厂TransactionFactory
事务工厂Transaction定义了创建Transaction的两个方法：一个是通过指定的Connection对象创建Transaction，另外是通过数据源DataSource来创建Transaction。

与JDBC和MANAGED两种Transaction相对应，TransactionFactory默认有两个对应的实现的子类：如下所示：
![](http://static.tmaczhao.cn/images/20211c8f9545ac4901ef926cb31478f9.jpg)

#### 创建Transaction
JDBCTransaction的创建流程：
```java
public class JdbcTransactionFactory implements TransactionFactory {

    public void setProperties(Properties props) {
    }

    /**
     * 根据给定的数据库连接Connection创建Transaction
     */
    public Transaction newTransaction(Connection conn) {
        return new JdbcTransaction(conn);
    }

    /**
     * 根据DataSource、隔离级别和是否自动提交创建Transacion
     */
    public Transaction newTransaction(DataSource ds, TransactionIsolationLevel level, boolean autoCommit) {
        return new JdbcTransaction(ds, level, autoCommit);
    }
}
```

#### JdbcTransaction
JdbcTransaction直接使用JDBC的提交和回滚事务管理机制 。它依赖与从dataSource中取得的连接connection 来管理transaction 的作用域，connection对象的获取被延迟到调用getConnection()方法。
>如果autocommit设置为on，开启状态的话，它会忽略commit和rollback。

```java
    /**
     * 获取数据库连接
     */
    public Connection getConnection() throws SQLException {
        if (connection == null) {
            // connection = dataSource.getConnection();
            openConnection();
        }
        return connection;
    }

    /**
     * commit()功能 使用connection的commit()
     * @throws SQLException
     */
    public void commit() throws SQLException {
        if (connection != null && !connection.getAutoCommit()) {
            if (log.isDebugEnabled()) {
                log.debug("Committing JDBC Connection [" + connection + "]");
            }
            connection.commit();
        }
    }

    /**
     * rollback()功能 使用connection的rollback()
     * @throws SQLException
     */
  public void rollback() throws SQLException {
    if (connection != null && !connection.getAutoCommit()) {
      if (log.isDebugEnabled()) {
        log.debug("Rolling back JDBC Connection [" + connection + "]");
      }
      connection.rollback();
    }
  }

    /**
     * close()功能 使用connection的close()
     * @throws SQLException
     */
  public void close() throws SQLException {
    if (connection != null) {
      resetAutoCommit();
      if (log.isDebugEnabled()) {
        log.debug("Closing JDBC Connection [" + connection + "]");
      }
      connection.close();
    }
  }

  protected void openConnection() throws SQLException {
    if (log.isDebugEnabled()) {
      log.debug("Opening JDBC Connection");
    }
    connection = dataSource.getConnection();
    if (level != null) {
      connection.setTransactionIsolation(level.getLevel());
    }
    setDesiredAutoCommit(autoCommmit);
  }
```

#### ManagedTransaction
ManagedTransaction让容器来管理事务Transaction的整个生命周期，意思就是说，使用ManagedTransaction的commit和rollback功能不会对事务有任何的影响，它什么都不会做，它将事务管理的权利移交给了容器来实现。
>ManagedTransaction中的commit和rollback为空，也就是说如果容器不接管事务处理功能，则update等操作对于数据库来说无效。



### 缓存
#### 一级缓存
每当我们使用MyBatis开启一次和数据库的会话，MyBatis会创建出一个SqlSession对象表示一次数据库会话。

在对数据库的一次会话中，MyBatis会在表示会话的SqlSession对象中建立一个简单的缓存，将每次查询到的结果结果缓存起来，当下次查询的时候，如果判断先前有个完全一样的查询，会直接从缓存中直接将结果取出，返回给用户，不需要再进行一次数据库查询了。这个过程在Executor中执行。

**`对于会话（Session）级别的数据缓存，我们称之为一级数据缓存，简称一级缓存。`**

##### MyBatis中的一级缓存是怎样组织的
实际上,SqlSession只是一个MyBatis对外的接口，SqlSession将它的工作交给了Executor执行器这个角色来完成，负责完成对数据库的各种操作。当创建了一个SqlSession对象时，MyBatis会为这个SqlSession对象创建一个新的Executor执行器，而缓存信息就被维护在这个Executor执行器中，MyBatis将缓存和对缓存相关的操作封装成了Cache接口中。SqlSession、Executor、Cache之间的关系如下列类图所示：
![](http://static.tmaczhao.cn/images/45b52ea15c05b4139b3ac307bb14eeb3.jpg)

>PerpetualCache内部通过一个简单的HashMap实现

##### 一级缓存的生命周期
1. MyBatis在开启一个数据库会话时，会创建一个新的SqlSession对象，SqlSession对象中会有一个新的Executor对象，Executor对象中持有一个新的PerpetualCache对象；当会话结束时，SqlSession对象及其内部的Executor对象还有PerpetualCache对象也一并释放掉。
2. 如果SqlSession调用了close()方法，会释放掉一级缓存PerpetualCache对象，一级缓存将不可用；
3. 如果SqlSession调用了clearCache()，会清空PerpetualCache对象中的数据，但是该对象仍可使用；
4.SqlSession中执行了任何一个数据更新操作（update、insert等），都会清空PerpetualCache对象的数据，但是该对象可以继续使用；

##### 一级缓存的工作流程
1. 对于某个查询，根据statementId,params,rowBounds来构建一个key值，根据这个key值去缓存Cache中取出对应的key值存储的缓存结果；
2. 判断从Cache中根据特定的key值取的数据数据是否为空，即是否命中；
3. 如果命中，则直接将缓存结果返回；
4. 如果没命中：
    - 4.1  去数据库中查询数据，得到查询结果；
    - 4.2  将key和查询到的结果分别作为key,value对存储到Cache中；
    - 4.3. 将查询结果返回；
5. 结束。

>CacheKey由以下条件决定：`statementId  + rowBounds  + 传递给JDBC的SQL  + 传递给JDBC的参数值`

#### 二级缓存
MyBatis的缓存机制整体设计以及二级缓存的工作模式如下：
![](http://static.tmaczhao.cn/images/69b697971802c6a3af1044b03c6f6bb8.jpg)

如上图所示，当开一个会话时，一个SqlSession对象会使用一个Executor对象来完成会话操作，MyBatis的二级缓存机制的关键就是对这个Executor对象做文章。

如果用户配置了`"cacheEnabled=true"`，那么MyBatis在为SqlSession对象创建Executor对象时，会对Executor对象加上一个装饰者：`CachingExecutor`，这时SqlSession使用`CachingExecutor`对象来完成操作请求。

`CachingExecutor`对于查询请求，会先判断该查询请求在Application级别的二级缓存中是否有缓存结果，如果有查询结果，则直接返回缓存结果；如果缓存中没有，再交给真正的Executor对象来完成查询操作，之后CachingExecutor会将真正Executor返回的查询结果放置到缓存中，然后在返回给用户。

>CachingExecutor是Executor的装饰者，以增强Executor的功能，使其具有缓存查询的功能，这里用到了设计模式中的装饰者模式。


MyBatis并不是简单地对整个Application就只有一个Cache缓存对象，它将缓存划分的更细，即是Mapper级别的，即每一个Mapper都可以拥有一个Cache对象，具体如下：
- 为每一个Mapper分配一个Cache缓存对象（使用<cache\>节点配置）；
- 多个Mapper共用一个Cache缓存对象（使用<cache-ref\>节点配置）；

>虽然在Mapper中配置了<cache>,并且为此Mapper分配了Cache对象，这并不表示我们使用Mapper中定义的查询语句查到的结果都会放置到Cache对象之中，我们必须指定Mapper中的某条选择语句是否支持缓存，即如下所示，在<select\>节点中配置useCache="true"，Mapper才会对此Select的查询支持缓存特性，否则，不会对此Select查询，不会经过Cache缓存。

总之，要想使某条Select查询支持二级缓存，你需要保证：
    1.  MyBatis支持二级缓存的总开关：全局配置变量参数   cacheEnabled=true
    2. 该select语句所在的Mapper，配置了<cache\> 或<cached-ref\>节点，并且有效
    3. 该select语句的参数 useCache=true

>二级缓存中关于各种过期时间，过期策略，大小等功能通过装饰者模式形成了一条委托链

![](http://static.tmaczhao.cn/images/aaafd4fd349a33f2b9ac14aa0a5251d2.jpg)
![](http://static.tmaczhao.cn/images/fbf96bd4a5588a5223a4897539573fc1.jpg)

























