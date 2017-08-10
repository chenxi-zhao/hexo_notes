#### session和cookie的区别和联系，session的生命周期，多个服务部署时session管理。

#### servlet的一些相关问题
![Servlet面试题归纳][http://blog.csdn.net/caohaicheng/article/details/38116481]


#### webservice相关问题
1. 分布式缓存
- redis可以使用jedis进行数据分片，也可以使用twmproxy做分布式代理，后程序也可用于memcached
- Ehcache


#### jdbc连接，forname方式的步骤，怎么声明使用一个事务。举例并具体代码
![JDBC常见面试题集锦][http://www.cnblogs.com/kevinf/p/3705148.html]
![JDBC系列 之 <驱动加载原理全面解析>][http://blog.csdn.net/luanlouis/article/details/29850811]

**`Class.forName()将对应的驱动类加载到内存中，然后执行内存中的static静态代码段，代码段中，会创建一个驱动Driver的实例，放入DriverManager中，供DriverManager使用。`**

>例如，在使用Class.forName() 加载oracle的驱动oracle.jdbc.driver.OracleDriver时，会执行OracleDriver中的静态代码段，创建一个OracleDriver实例，然后调用DriverManager.registerDriver()注册：


#### JDBC的PreparedStatement是什么？
PreparedStatement对象代表的是一个预编译的SQL语句。用它提供的setter方法可以传入查询的变量。
由于PreparedStatement是预编译的，通过它可以将对应的SQL语句高效的执行多次。由于PreparedStatement自动对特殊字符转义，避免了SQL注入攻击，因此应当尽量的使用它。

#### Statement中的setFetchSize和setMaxRows方法有什么用处？
setMaxRows可以用来限制返回的数据集的行数。当然通过SQL语句也可以实现这个功能。比如在MySQL中我们可以用LIMIT条件来设置返回结果的最大行数。

setFetchSize理解起来就有点费劲了，因为你得知道Statement和ResultSet是怎么工作的。当数据库在执行一条查询语句时，查询到的数据是在数据库的缓存中维护的。ResultSet其实引用的是数据库中缓存的结果。

假设我们有一条查询返回了100行数据，我们把fetchSize设置成了10，那么数据库驱动每次只会取10条数据，也就是说得取10次。当每条数据需要处理的时间比较长的时候并且返回数据又非常多的时候，这个可选的参数就变得非常有用了。

我们可以通过Statement来设置fetchSize参数，不过它会被ResultSet对象设置进来的值所覆盖掉。


#### 如何使用JDBC接口来调用存储过程？
存储过程就是数据库编译好的一组SQL语句，可以通过JDBC接口来进行调用。我们可以通过JDBC的CallableStatement接口来在数据库中执行存储过程。初始化CallableStatement的语法是这样的：
```java
CallableStatement stmt = con.prepareCall("{call insertEmployee(?,?,?,?,?,?)}");
stmt.setInt(1, id);
stmt.setString(2, name);
stmt.setString(3, role);
stmt.setString(4, city);
stmt.setString(5, country);
//register the OUT parameter before calling the stored procedure
stmt.registerOutParameter(6, java.sql.Types.VARCHAR);
stmt.executeUpdate();
```

>JDBC通过Statement和PreparedStatement中的addBatch和executeBatch方法来支持批处理。

#### 无框架下配置web.xml的主要配置内容



#### jsp和servlet的区别
Servlet是服务器端的程序，动态生成html页面发送到客户端，但是这样程序里会有很多out.println(),Java与html语言混在一起
很乱，所以后来sun公司推出了JSP.其实JSP就是Servlet，每次运行的时候JSP都首先被编译成servlet文件，然后再被编译成
.class文件运行。有了jsp，在MVC项目中servlet不再负责动态生成页面，转而去负责控制程序逻辑的作用，控制jsp与javabean
之间的流转。


#### JSP中动态INCLUDE和静态INCLUDE有什么区别
动态INCLUDE用jsp:include动作实现 <jsp:include page="included.jsp" flush="true" />它总是会检查所含文件中的变化,适合用于包含动态页面,
并且可以带参数.
静态INCLUDE用include伪码实现,定不会检查所含文件的变化,适用于包含静态页面<%@ include file="included.htm" %>



#### doGet与doPost方法的两个参数是什么
- HttpServletRequest：封装了与请求相关的信息
- HttpServletResponse：封装了与响应相关的信息


#### forward和redirect的区别
转发与重定向
（1）从地址栏显示来说
forward是服务器请求资源，服务器直接访问目标地址的URL，把那个URL的响应内容读取过来，然后把这些内容再发给浏览器。浏览器根本不知道服务器发送的内容从哪里来的，所以它的地址栏还是原来的地址。redirect是服务端根据逻辑，发送一个状态码，告诉浏览器重新去请求那个地址。所以地址栏显示的是新的URL。

（2）从数据共享来说
forward:转发页面和转发到的页面可以共享request里面的数据。
redirect:不能共享数据。
（3）从运用地方来说
forward:一般用于用户登陆的时候,根据角色转发到相应的模块.
redirect:一般用于用户注销登陆时返回主页面和跳转到其它的网站等.
（4）从效率来说
forward:高.
redirect:低.
