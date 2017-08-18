### JDBC的数据操作
```java
// 加载jdbc驱动
Class.forName("com.mysql.jdbc.Driver");
// 获取jdbc连接
Connection conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/test", "root", "admin");
String sql = "SELECT * FROM STUDENTS WHERE STUD_ID=?";
//创建PreparedStatement
PreparedStatement pstmt = conn.prepareStatement(sql);
//设置输入参数
pstmt.setInt(1, studId);
ResultSet rs = pstmt.executeQuery();
//从数据库中取出结果并生成 Java 对象
if(rs.next()){
    student = new Student();
    student.setStudId(rs.getInt("stud_id"));
    student.setName(rs.getString("name"));
    student.setEmail(rs.getString("email"));
    student.setDob(rs.getDate("dob"));
}
```
>上述的每个方法中有大量的重复代码：创建一个连接，创建一个Statement对象，设置输入参数，关闭资源（如connection，statement，resultSet）。MyBatis抽象了上述的这些相同的任务，如准备需要被执行的SQLstatement对象并且将Java对象作为输入数据传递给statement对象的任务，进而开发人员可以专注于真正重要的方面。

`另外，MyBatis自动化了将从输入的Java对象中的属性设置成查询参数、从SQL结果集上生成Java对象这两个过程。`

>- hibernate映射Java对象到数据库表上，
- mybatis则是主要映射查询结果集合以及查询参数，这使得MyBatis可以很好地与传统数据库协同工作。你可以根据面相对象的模型创建Java域对象，执行传统数据库的查询，然后将结果映射到对应的Java对象上。


### mybatis特性
>当前有很多Java实现的持久化框架，而MyBatis流行起来有以下原因：
- 它消除了大量的JDBC冗余代码
- 它有低的学习曲线
- 它能很好地与传统数据库协同工作
- 它可以接受SQL语句
- 它提供了与Spring 和Guice框架的集成支持
- 它提供了与第三方缓存类库的集成支持
- 它引入了更好的性能


### mybatis配置文件详解
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

    <!-- 引入外部配置文件 http://images2015.cnblogs.com/blog/664931/201602/664931-20160225094128786-1384508381.png-->
    <!-- 如果<properties>中定义的元素和属性文件定义元素的key值相同，它们会被属性文件中定义的值覆盖。 -->
    <properties resource="mysql.properties">
        <property name="username" value="db_user" />
        <property name="password" value="verysecurepwd" />
    </properties>


    <!-- settings是 MyBatis 中极为重要的调整设置，它们会改变 MyBatis 的运行时行为。 -->
    <settings>
        <!-- 该配置影响的所有映射器中配置的缓存的全局开关。默认值true -->
        <setting name="cacheEnabled" value="true"/>

        <!--延迟加载的全局开关。当开启时，所有关联对象都会延迟加载。 特定关联关系中可通过设置fetchType属性来覆盖该项的开关状态。默认值false  -->
        <setting name="lazyLoadingEnabled" value="true"/>

        <!--当对象使用延迟加载时 属性的加载取决于能被引用到的那些延迟属性,否则,按需加载(需要的是时候才去加载)-->
        <setting name="aggressiveLazyLoading" value="true"/>

        <!-- 是否允许单一语句返回多结果集（需要兼容驱动）。 默认值true -->
        <setting name="multipleResultSetsEnabled" value="true"/>

        <!-- 使用列标签代替列名。不同的驱动在这方面会有不同的表现， 具体可参考相关驱动文档或通过测试这两种不同的模式来观察所用驱动的结果。默认值true -->
        <setting name="useColumnLabel" value="true"/>

        <!-- 允许 JDBC 支持自动生成主键，需要驱动兼容。 如果设置为 true 则这个设置强制使用自动生成主键，尽管一些驱动不能兼容但仍可正常工作（比如 Derby）。 默认值false  -->
        <setting name="useGeneratedKeys" value="false"/>

        <!-- 指定MyBatis应如何自动映射列到字段或属性。NONE表示取消自动映射；PARTIAL只会自动映射没有定义嵌套结果集映射的结果集。 FULL会自动映射任意复杂的结果集（无论是否嵌套）。-->
        <!-- 默认值PARTIAL -->
        <setting name="autoMappingBehavior" value="PARTIAL"/>
        <setting name="autoMappingUnknownColumnBehavior" value="WARNING"/>

        <!--  配置默认的执行器。SIMPLE 就是普通的执行器；REUSE 执行器会重用预处理语句（prepared statements）； BATCH 执行器将重用语句并执行批量更新。默认SIMPLE  -->
        <setting name="defaultExecutorType" value="SIMPLE"/>

        <!-- 设置超时时间，它决定驱动等待数据库响应的秒数。 -->
        <setting name="defaultStatementTimeout" value="25"/>

        <!--设置默认每次拉取的数据条数-->
        <setting name="defaultFetchSize" value="100"/>

        <!-- 允许在嵌套语句中使用分页（RowBounds）默认值False -->
        <setting name="safeRowBoundsEnabled" value="false"/>

        <!-- 是否开启自动驼峰命名规则（camel case）映射，即从经典数据库列名 A_COLUMN 到经典 Java 属性名 aColumn 的类似映射。  默认false -->
        <setting name="mapUnderscoreToCamelCase" value="false"/>

        <!-- MyBatis 利用本地缓存机制（Local Cache）防止循环引用（circular references）和加速重复嵌套查询。
               默认值为 SESSION，这种情况下会缓存一个会话中执行的所有查询。
              若设置值为 STATEMENT，本地会话仅用在语句执行上，对相同 SqlSession 的不同调用将不会共享数据。  -->
        <setting name="localCacheScope" value="SESSION"/>

        <!-- 当没有为参数提供特定的 JDBC 类型时，为空值指定 JDBC 类型。 某些驱动需要指定列的 JDBC 类型，多数情况直接用一般类型即可，比如 NULL、VARCHAR 或 OTHER。  -->
        <setting name="jdbcTypeForNull" value="OTHER"/>

        <!--   指定哪个对象的方法触发一次延迟加载。  -->
        <setting name="lazyLoadTriggerMethods" value="equals,clone,hashCode,toString"/>
    </settings>



    <!--无论是 MyBatis 在预处理语句（PreparedStatement）中设置一个参数时，还是从结果集中取出一个值时，-->
    <!--都会用类型处理器将获取的值以合适的方式转换成 Java 类型。下表描述了一些默认的类型处理器。-->
    <!--可以重写类型处理器或创建自己的类型处理器来处理不支持的或非标准的类型。具体的做法为：实现org.apache.ibatis.type.TypeHandler接口，-->
    <!--或继承一个很便利的类org.apache.ibatis.type.BaseTypeHandler,然后可以选择性地将它映射到一个JDBC类型。-->
    <!--<typeHandlers>-->
        <!--<typeHandler handler="org.mybatis.example.ExampleTypeHandler"/>-->
        <!--<package name="org.mybatis.example"/>-->
    <!--</typeHandlers>-->


    <!-- 为JAVA Bean起类别名 -->
    <typeAliases>
        <!-- 别名方式1，一个一个的配置 type中放置的是类的全路径，alias中放置的是类别名
        <typeAliase type="com.chenxi.mybatis.beans.UserBean" alias="UserBean"/> -->
        <!-- 别名方式2，自动扫描，将JAVA类的类名作为类的类别名 -->
        <package name="com.chenxi.mybatis.beans"/>
    </typeAliases>


    <!-- 配置mybatis运行环境 -->
    <!-- 根据environment id创建不同的SqlSessionFactory -->
    <!-- new SqlSessionFactoryBuilder().build(inputStream, "development"); -->
    <environments default="mybatis">
        <environment id="development">
            <!-- MyBatis 支持两种类型的事务管理器： JDBC and MANAGED. -->
            <!-- type="JDBC" 使用JDBC的提交和回滚来管理事务，-->
            <!-- JDBC事务管理器被用作当应用程序负责管理数据库连接的生命周期（提交、回退等等）的时候。例如，部署到Apache Tomcat的应用程序，需要应用程序自己管理事务。使用JdbcTransactionFactory类创建TransactionManager。 -->
            <!-- type="MANAGED" 事务管理器是当由应用服务器负责管理数据库连接生命周期的时候使用。 -->
            <!-- Managed是托管的意思，即是应用本身不去管理事务，而是把事务管理交给应用所在的服务器进行管理。如JBoss，WebLogic，GlassFish。使用ManagedTransactionFactory 类创建事务管理器TransactionManager。 -->
            <transactionManager type="JDBC"/>

            <!-- mybatis提供了3种数据源类型，分别是：POOLED,UNPOOLED,JNDI -->
            <!-- POOLED 表示支持JDBC数据源连接池。连接池中的连接将会被用作数据库操作。数据库操作完成时会将连接返回给连接池。-->
            <!-- UNPOOLED 表示不支持数据源连接池，MyBatis会为每一个数据库操作创建一个新的连接，并关闭它，不推荐-->
            <!-- JNDI MyBatis从在应用服务器向配置好的JNDI数据源dataSource获取数据库连接。生产环境-->
            <dataSource type="POOLED">
                <property name="driver" value="${jdbc.driver}"/>
                <property name="url" value="${jdbc.url}"/>
                <property name="username" value="${jdbc.username}"/>
                <property name="password" value="${jdbc.password}"/>
            </dataSource>
        </environment>

        <environment id="production">
            <transactionManager type="MANAGED" />
            <dataSource type="JNDI">
                <property name="data_source" value="java:comp/jdbc/MyBatisDemoDS" />
            </dataSource>
        </environment>
    </environments>


    <mappers>
        <!-- 告知映射文件方式1，一个一个的配置
        <mapper resource="com/chenxi/mybatis/mapper/UserMapper.xml"/>-->
        <!-- url属性用来通过完全文件系统路径或者web URL地址来指向mapper文件 -->
        <mapper url="file:///D:/mybatisdemo/app/mappers/TutorMapper.xml" />
        <!-- class属性用来指向一个mapper接口 -->
        <mapper class="com.mybatis3.mappers.TutorMapper" />
        <!-- 告知映射文件方式2，自动扫描包内的Mapper接口与配置文件 -->
        <package name="com/chenxi/mybatis/mapper"/>
    </mappers>
</configuration>
```

### 基于Java API的配置
#### 创建SessionFactory
```java
SqlSessionFactory sqlSessionFactory = null;
try{
    // 获取数据源
    DataSource dataSource = DataSourceFactory.getDataSource();
    // 获取事务管理器工厂
    TransactionFactory transactionFactory = new JdbcTransactionFactory();
    // TransactionFactory txnFactory = new ManagedTransactionFactory();
    // 获取运行环境
    Environment environment = new Environment("development", transactionFactory, dataSource);
    // 配置mybatis详细配置
    Configuration configuration = new Configuration(environment);
    // 注册别名
    configuration.getTypeAliasRegistry().registerAlias("student", Student.class);
    configuration.getTypeAliasRegistry().registerAlias("Student", "com.mybatis3.domain.Student");
    // 根据默认的别名规则，使用一个类的首字母小写、非完全限定的类名作为别名注册
    configuration.getTypeAliasRegistry().registerAlias(Student.class);
    configuration.getTypeAliasRegistry().registerAliases("com.mybatis3.domain");
    configuration.getTypeAliasRegistry().registerAliases("com.mybatis3.domain", Identifiable.class);
    // 注册类型处理器
    configuration.getTypeHandlerRegistry().register(PhoneNumber. class, PhoneTypeHandler.class);
    configuration.getTypeHandlerRegistry().register("com.mybatis3.typehandlers");
    // 添加mapper
    configuration.addMapper(StudentMapper.class);
    configuration.addMappers("com.mybatis3.mappers");
    // 添加所有com.mybatis3.mappers包中的拓展了特定Mapper接口的Maper接口
    configuration.addMappers("com.mybatis3.mappers", BaseMapper.class);
    sqlSessionFactory = new SqlSessionFactoryBuilder().build(configuration);
}catch (Exception e){
    throw new RuntimeException(e);
}
return sqlSessionFactory;
```
#### 创建数据源
```java
public class DataSourceFactory{
    public static DataSource getDataSource() {
        String driver = "com.mysql.jdbc.Driver";
        String url = "jdbc:mysql://localhost:3306/mybatisdemo";
        String username = "root";
        String password = "admin";
        PooledDataSource dataSource = new PooledDataSource(driver, url, username, password);
        return dataSource;
    }

    public static DataSource getJNDIDataSource(){
        String jndiName = "java:comp/env/jdbc/MyBatisDemoDS";
        try{
            InitialContext ctx = new InitialContext();
            DataSource dataSource = (DataSource) ctx.lookup(jndiName);
            return dataSource;
        }catch (NamingException e){
            throw new RuntimeException(e);
        }
    }
}
```
>还可以使用第三方库创建数据源，c3p0、druid等

#### 全局设置
```java
configuration.setCacheEnabled(true);
configuration.setLazyLoadingEnabled(false);
configuration.setMultipleResultSetsEnabled(true);
configuration.setUseColumnLabel(true);
configuration.setUseGeneratedKeys(false);
configuration.setAutoMappingBehavior(AutoMappingBehavior.PARTIAL);
configuration.setDefaultExecutorType(ExecutorType.SIMPLE);
configuration.setDefaultStatementTimeout(25);
configuration.setSafeRowBoundsEnabled(false);
configuration.setMapUnderscoreToCamelCase(false);
configuration.setLocalCacheScope(LocalCacheScope.SESSION);
configuration.setAggressiveLazyLoading(true);
configuration.setJdbcTypeForNull(JdbcType.OTHER);
Set<String> lazyLoadTriggerMethods = new HashSet<String>();
lazyLoadTriggerMethods.add("equals");
lazyLoadTriggerMethods.add("clone");
lazyLoadTriggerMethods.add("hashCode");
lazyLoadTriggerMethods.add("toString");
configuration.setLazyLoadTriggerMethods(lazyLoadTriggerMethods );
```

### mappper.xml
该文件中的主要几个顶级元素
- cache – 配置给定命名空间的缓存。
- cache-ref – 从其他命名空间引用缓存配置。
- resultMap – 最复杂,也是最有力量的元素,用来描述如何从数据库结果集中来加载你的对象。
- ~~parameterMap – 已经被废弃了!老式风格的参数映射。内联参数是首选,这个元素可能在将来被移除。这里不会记录。~~
- sql – 可以重用的 SQL 块,也可以被其他语句引用。
- insert – 映射插入语句
- update – 映射更新语句
- delete – 映射删除语句
- select – 映射查询语句

#### select元素
```xml
<select id="selectPerson" parameterType="int" parameterMap="deprecated" resultType="hashmap" resultMap="personResultMap" flushCache="false" useCache="true" timeout="10000" fetchSize="256"  statementType="PREPARED" resultSetType="FORWARD_ONLY">
SELECT * FROM PERSON WHERE ID = #{id}
</select>
```
![](http://static.tmaczhao.cn/images/1d3cc794a4c1f5095b04de1b421b1f6e.jpg)

上面的代码约等于jdbc中的下述代码：
```java
//Similar JDBC code, NOT MyBatis…
String selectPerson = "SELECT * FROM PERSON WHERE ID=?";
PreparedStatement ps = conn.prepareStatement(selectPerson);
ps.setInt(1,id);
// ...
```

#### insert, update and delete
数据变更语句 insert,update 和 delete 在它们的实现中非常相似:
```xml
<insert id="insertAuthor" parameterType="domain.blog.Author" flushCache="true" statementType="PREPARED" keyProperty="" keyColumn="" useGeneratedKeys="" timeout="20000" />
insert into Author (id,username,password,email,bio) values (#{id},#{username},#{password},#{email},#{bio})
</insert>

<update id="updateAuthor" parameterType="domain.blog.Author" flushCache="true" statementType="PREPARED"   timeout="20000">
update Author set username=#{username},password=#{password},email=#{email},bio=#{bio} where id=#{id}
</update>

<delete id="deleteAuthor" parameterType="int" flushCache="true" statementType="PREPARED" timeout="20000"/>
delete from Author where id=#{id}
</delete>
```

其中的属性解释如下：
| 属性 | 解释 |
| --- | --- |
| id | 在命名空间中唯一的标识符,可以被用来引用这条语句。|
| parameterType | 将会传入这条语句的参数类的完全限定名或别名。|
| ~~parameterMap~~ | 这是引用外部 parameterMap 的已经被废弃的方法。使用内联参数映射和 parameterType 属性。|
| flushCache | 将其设置为 true,不论语句什么时候被带哦用,都会导致缓存被清空。默认值:false。|
| timeout | 这个设置驱动程序等待数据库返回请求结果, 并抛出异常时间的最大等待值。默认不设置(驱动自行处理)。|
| statementType | STATEMENT，PREPARED或CALLABLE的一种。这会让MyBatis使用选择使用Statement，PreparedStatement或CallableStatement。默认值:PREPARED。 |
| useGeneratedKeys | (仅对insert有用) 这会告诉MyBatis使用JDBC的getGeneratedKeys方法来取出由数据(比如:像MySQL和SQLServer这样的数据库管理系统的自动递增字段)内部生成的主键。默认值:false。|
| keyProperty | (仅对insert有用)标记一个属性，MyBatis会通过getGeneratedKeys或者通过insert语句的selectKey子元素设置它的值。默认: 不设置。 |
| keyColumn | （仅对insert有用）标记一个属性，MyBatis会通过getGeneratedKeys或者通过insert语句的selectKey子元素设置它的值。默认: 不设置。 |

#### sql
```xml
<sql id="userColumns"> id,username,password </sql>
<select id="selectUsers" parameterType="int" resultType="hashmap">
 select <include refid="userColumns"/> from some_table where id = #{id}
 </select>
```

#### property
```xml
\#{property, mode=OUT,javaType=int,jdbcType=NUMERIC, mode=OUT}
```

#### resultMap
```xml
<resultMap type="CoursesBean" id="coursesMap">
    <!--   在默认情况下，mybatis会自动在TypeAliasRegistry初始化的时候挂在很多jdk常用类，所以javaType="java.lang.Integer"可以写成javaType="Integer"-->
    <id property="id" column="id" javaType="java.lang.Integer"/>
    <result property="name" column="courses_name" javaType="java.lang.String"  typeHandler="" jdbcType=""/>
</resultMap>

<resultMap type="CoursesBean" id="couAndStu">
    <id property="id" column="id" javaType="java.lang.Integer"/>
    <result property="name" column="courses_name" javaType="java.lang.String"/>
    <!-- 对于一个属性的类型是一个集合，就使用collection 对于一个属性的类型是一个类，就使用association   -->
    <!-- 嵌套查询 -->
    <collection property="student" column="id" select="findStudentByCourses"/>
    <!-- 结果集查询，用于表连接直接把表字段都查询出来的情况 -->
    <collection property="student" column="id" resultMap="coursesMap"/>
    <!-- <association property="wife" column="id" javaType="WifeBean" select="com.chenxi.mybatis.mapper.WifeMapper.selectWifeByHusbandId"/> -->
</resultMap>

<resultMap id="detailedBlogResultMap" type="Blog">
     <!-- 类在实例化时,用来注入结果到构造方法中 -->
    <constructor>
        <idArg column="blog_id" javaType="int"/>
    </constructor>
    <!-- 注入到字段或 JavaBean 属性的普通结果 -->
    <result property="title" column="blog_title"/>
    <!-- 一个复杂的类型关联;许多结果将包成这种类型 -->
    <association property="author" javaType="Author">
        <!-- 一个 ID 结果;标记结果作为 ID 可以帮助提高整体效能 -->
        <id property="id" column="author_id"/>
        <result property="username" column="author_username"/>
        <result property="password" column="author_password"/>
        <result property="email" column="author_email"/>
        <result property="bio" column="author_bio"/>
        <result property="favouriteSection" column="author_favourite_section"/>
    </association>
    <!-- 嵌入结果映射 – 结果映射自身的集,或者参考一个 -->
    <collection property="posts" ofType="Post">
        <id property="id" column="post_id"/>
        <result property="subject" column="post_subject"/>
        <association property="author" javaType="Author"/>
        <collection property="comments" ofType="Comment">
            <id property="id" column="comment_id"/>
        </collection>
        <collection property="tags" ofType="Tag" >
           <id property="id" column="tag_id"/>
        </collection>
        <!-- 使用结果值来决定使用哪个结果映射 -->
        <discriminator javaType="int" column="draft">
            <case value="1" resultType="DraftPost"/>
        </discriminator>
    </collection>
</resultMap>
```

#### 动态Sql语句
>mybatis通过ONGL（Object Graph Nagigation Language）表达式来构建动态sql语句

#####  If条件
```xml
<resultMap type="Course" id="CourseResult">
  <id column="course_id" property="courseId" />
  <result column="name" property="name" />
  <result column="description" property="description" />
  <result column="start_date" property="startDate" />
  <result column="end_date" property="endDate" />
</resultMap>
<!-- 参数满足条件的时候添加语句 -->
<select id="searchCourses" parameterType="hashmap" resultMap="CourseResult"></select>
    SELECT * FROM COURSES
        WHERE TUTOR_ID= #{tutorId}
    <if test="courseName != null">
        AND NAME LIKE #{courseName}
    </if>
    <if test="startDate != null">
        AND START_DATE >= #{startDate}
    </if>
    <if test="endDate != null">
        AND END_DATE <= #{endDate}
    </if>
</select>
```

##### choose,when 和otherwise 条件
```xml
<select id="searchCourses" parameterType="hashmap" resultMap="CourseResult">
    SELECT * FROM COURSES
    <choose>
        <when test="searchBy == 'Tutor'">
            WHERE TUTOR_ID= #{tutorId}
        </when>
        <when test="searchBy == 'CourseName'">
            WHERE name like #{courseName}
        </when>
        <otherwise>
            WHERE TUTOR start_date >= now()
        </otherwise>
    </choose>
</select>
```

##### where、set条件
```xml
<!-- 内部条件判断有任何内容返回时，他会插入SET SQL片段。 -->
<select id="searchCourses" parameterType="hashmap"resultMap="CourseResult">
    SELECT * FROM COURSES
    <where>
        <if test=" tutorId != null ">
            TUTOR_ID= #{tutorId}
        </if>
        <if test="courseName != null">
            AND name like #{courseName}
        </if>
        <if test="startDate != null">
            AND start_date >= #{startDate}
        </if>
        <if test="endDate != null">
            AND end_date <= #{endDate}
        </if>
    </where>
</select>
<!-- 内部条件判断有任何内容返回时，他会插入SET SQL片段。 -->
<update id="updateStudent" parameterType="Student">
    update students
    <set>
    <if test="name != null">name=#{name},</if>
    <if test="email != null">email=#{email},</if>
    <if test="phone != null">phone=#{phone},</if>
    </set>
    where stud_id=#{id}
</update>
```

##### <trim>条件
```xml
<select id="searchCourses" parameterType="hashmap" resultMap="CourseResult">
SELECT * FROM COURSES
<trim prefix="WHERE" prefixOverrides="AND | OR">
    <if test=" tutorId != null ">
        TUTOR_ID= #{tutorId}
    </if>
    <if test="courseName != null">
        AND name like #{courseName}
    </if>
</trim>
</select>
```

##### foreach循环
```xml
<!-- where子句中一直通过Or连接 -->
<select id="searchCoursesByTutors" parameterType="map"
resultMap="CourseResult">
    SELECT * FROM COURSES
    <if test="tutorIds != null">
        <where>
            <foreach item="tutorId" collection="tutorIds">
            OR tutor_id=#{tutorId}
            </foreach>
        </where>
    </if>
</select>
<!-- In子句的实现 -->
<select id="searchCoursesByTutors" parameterType="map" resultMap="CourseResult">
    SELECT * FROM COURSES
    <if test="tutorIds != null">
        <where>
        tutor_id IN
            <foreach item="tutorId" collection="tutorIds"
            open="(" separator="," close=")">
            #{tutorId}
            </foreach>
        </where>
    </if>
</select>
```


#### 缓存
MyBatis包含一个非常强大的查询缓存特性，它可以非常方便地配置和定制。默认情况下是没有开启缓存的，除了局部的session缓存，可以增强变现而且处理循环依赖也是必须的。要开启二级缓存,你需要在你的 SQL 映射文件中添加一行:
><cache />

这个简单语句的效果如下:
- 映射语句文件中的所有select语句将会被缓存。
- 映射语句文件中的所有insert，update和delete语句会刷新缓存。
- 缓存会使用 Least Recently Used(LRU,最近最少使用的)算法来收回。
- 根据时间表(比如 no Flush Interval,没有刷新间隔), 缓存不会以任何时间顺序来刷新。
- 缓存会存储列表集合或对象(无论查询方法返回什么)的1024个引用。
- 缓存会被视为是read/write(可读/可写)的缓存，意味着对象检索不是共享的，而且可以安全地被调用者修改，而不干扰其他调用者或线程所做的潜在修改。

所有的这些属性都可以通过缓存元素的属性来修改。比如:
```xml
<!-- 这个配置创建了一个FIFO缓存，并每隔60秒刷新，存数结果对象或列表的512个引用，而且返回的对象被认为是只读的，因此在不同线程中的调用者之间修改它们会导致冲突。 -->
<!-- 收回策略有LRU（最少使用的被回收，默认设置）、FIFO（先进先出，按缓存顺序）、SOFT（软引用:移除基于垃圾回收器状态和软引用规则的对象）、WEAK（弱引用:更积极地移除基于垃圾收集器状态和弱引用规则的对象） -->
<!-- flushInterval(刷新间隔)可以被设置为任意的正整数,而且它们代表一个合理的毫秒形式的时间段。默认情况是不设置,也就是没有刷新间隔,缓存仅仅调用语句时刷新。 -->
<!-- size(引用数目)可以被设置为任意正整数,要记住你缓存的对象数目和你运行环境的可用内存资源数目。默认值是 1024。 -->
<!-- readOnly(只读)属性可以被设置为 true 或 false。只读的缓存会给所有调用者返回缓存对象的相同实例。因此这些对象不能被修改。这提供了很重要的性能优势。可读写的缓存会返回缓存对象的拷贝(通过序列化) 。这会慢一些,但是安全,因此默认是 false。 -->
<cache eviction="FIFO" flushInterval="60000" size="512" readOnly="true"/>
```

##### 自定义缓存
```xml
<cache type="com.domain.something.MyCustomCache"/>
<!-- 参照缓存 -->
<cache-ref namespace="com.someone.application.data.SomeMapper"/>
```
type指定的类实现org.mybatis.cache.Cache接口。这个接口是MyBatis框架中很多复杂的接口之一,但是简单给定它做什么就行。
```java
public interface Cache{
    String getId();
    int getSize();
    void putObject(Object key, Object value);
    Object getObject(Object key);
    boolean hasKey(Object key);
    Object removeObject(Object key);
    void clear();
    ReadWriteLock getReadWriteLock();
}
```


























