1. 使用PreparedStatment
PreparedStatment可以通过预编译的方式避免我们在拼接SQL时造成SQL注入，以及缓存编译后的sql，提高执行效率。

2. 使用ConnectionPool（连接池）
使用连接的原因简单的说就是因为创建JDBC连接耗时比较长，如果每次查询都重新打开一个连接， 然后关闭，性能将会非常低，而如果事先创建好一批连接缓存起来，使用的时候取出， 不使用的时候仍不关闭，将会节省大量的创建关闭连接的时间。

3. 禁用自动提交
这个最佳实践在我们使用JDBC的批量提交的时候显得非常有用，将自动提交禁用后，你可以将一组数据库操作放在一个事务中，而自动提交模式每次执行SQL语句都将执行自己的事务，并且在执行结束提交。

4. 使用Batch Update
JDBC的API提供了通过addBatch()方法向batch中添加SQL查询，然后通过executeBatch()执行批量的查询。
JDBC batch update可以减少数据库数据传输的往返次数，从而提高性能。

5. 使用列名获取ResultSet中的数据，从而避免invalidColumIndexError
JDBC中的查询结果封装在ResultSet中，我们可以通过列名和列序号两种方式获取查询的数据，可能通过序号访问列要比列名访问快一些，确实是这样，但是为了程序的健壮性、可读性，我还是更推荐你使用列名来访问。

6. 使用变量绑定而不是字符串拼接
在第一条最佳实践中，我们已经说过要使用PreparedStatment可以防止注入，而使用？或者其他占位符也会提升性能，因为这样数据库就可以使用不同的参数执行相同的查询， 这个最佳实践带来更高的性能的同时也防止了SQL注入。

7. 要记住关闭Statement、PreparedStatement和Connection
通常的做法是在finally块中关闭它们，这样做的好处是不论语句执行正确与否，不管是否有异常抛出，都能保证资源被释放。在Java7中，可以通过Automatic Resource Management Block来自动的关闭资源。

8. 选择合适的JDBC驱动

9. 尽量使用标准的SQL语句，从而在某种程度上避免数据库对SQL支持的差异

10. 使用正确的getXXX()方法
当从ResultSet中读取数据的时候，虽然JDBC允许你使用getString()和getObject()方法获取任何数据类型，推荐使用正确的getXXX方法，这样可以避免数据类型转换。
