@CacheNamespace：用于类，相当于<cache \>
为给定的命名空间（比如类）配置缓存。属性：implemetation,eviction,flushInterval,size和readWrite 。

@CacheNamespaceRef：用于类，相当于<cacheRef \>
参照另外一个命名空间的缓存来使用。属性：value，也就是类的完全限定名。

@ConstructorArgs：用于方法，相当于<constructor\>
收集一组结果传递给对象构造方法。属性：value，是形式参数的数组

@Arg：用于方法，相当于<arg\>、<idArg\>
单独的构造方法参数，是ConstructorArgs集合的一部分。属性：id,column,javaType，typeHandler。
id属性是布尔值，来标识用于比较的属性，和<idArg>XML元素相似


@TypeDiscriminator：用于方法，相当于<discriminator\>
一组实例值被用来决定结果映射的表现。属性：Column,javaType,jdbcType typeHandler，cases。cases属性就是实例的数组。

@Case：用于方法，相当于<case\>
单独实例的值和它对应的映射。属性：value，type，results 。
Results属性是结果数组，因此这个注解和实际的ResultMap很相似，由下面的Results注解指定

@Results：用于方法，相当于<resultMap\>
结果映射的列表，包含了一个特别结果列如何被映射到属性或字段的详情。属性：value，是@Result的数组

@Result：用于方法，对应的xml<result\>、<id\>
在列和属性或字段之间的单独结果映射。属性：id，column，property，javaType，jdbcType，type Handler，one，many。
id属性是一个布尔值，表示了应该被用于比较的属性。one属性是单独的联系，和<association\>相似，而many属性是对集合而言的，和<collection\>相似。

@One：用于方法，相当于<association\>
复杂类型的单独属性值映射。属性：select，已映射语句（也就是映射器方法）的完全限定名，它可以加载合适类型的实例。注意：联合映射在注解API中是不支持的。

@Many：用于方法，相当于<collection\>
复杂类型的集合属性映射。属性：select，是映射器方法的完全限定名，它可加载合适类型的一组实例。注意：联合映射在Java注解中是不支持的。

@Options：用于方法映射语句的属性
这个注解提供访问交换和配置选项的宽广范围，它们通常在映射语句上作为属性出现。而不是将每条语句注解变复杂，Options注解提供连贯清晰的方式来访问它们。属性：useCache=true，flushCache=false，resultSetType=FORWARD_ONLY，statementType=PREPARED，fetchSize=-1，timeout=-1，useGeneratedKeys=false，keyProperty=”id“。
理解Java 注解是很重要的，因为没有办法来指定“null”作为值。因此，一旦你使用了Options注解，语句就受所有默认值的支配。要注意什么样的默认值来避免不期望的行为

@Insert、@Update、@Delete：用于方法，相当于<insert\>、<update\>、<delete\>
这些注解中的每一个代表了执行的真实 SQL。它们每一个都使用字符串数组（或单独的字符串）。如果传递的是字符串数组，它们由每个分隔它们的单独空间串联起来。属性：value，这是字符串数组用来组成单独的SQL语句

@Insert/Update/Delete/SelectProvider，用于方法、<insert\>、<update\>、<delete\>、<select\>
允许创建动态SQL。这些可选的SQL注解允许你指定一个类名和一个方法在执行时来返回运行的SQL。
基于执行的映射语句，MyBatis会实例化这个类，然后执行由provider指定的方法。
这个方法可以选择性的接受参数对象作为它的唯一参数，但是必须只指定该参数或者没有参数。
属性：type，method。type 属性是类的完全限定名。method是该类中的那个方法名。

@Param：参数，当映射器方法需多个参数，这个注解可以被应用于映射器方法参数来给每个参数一个名字。否则，多参数将会以它们的顺序位置来被命名。比如#{1}，#{2} 等，这是默认的。使用@Param(“person”)，SQL中参数应该被命名为#{person}。
