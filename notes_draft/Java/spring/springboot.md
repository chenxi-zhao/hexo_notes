# Spring-Boot 1.5 学习笔记
使用Spring Boot很容易创建一个独立运行（运行jar,内嵌Servlet容器）、准生产级别的基于Spring框架的项目，使用Spring Boot你可以不用或者只需要很少的Spring配置。

Spring将很多魔法带入了Spring应用程序的开发之中，其中最重要的是以下四个核心。
- 自动配置：针对很多Spring应用程序常见的应用功能，Spring Boot能自动提供相关配置
- 起步依赖：告诉Spring Boot需要什么功能，它就能引入需要的库。
- 命令行界面：这是Spring Boot的可选特性，借此你只需写代码就能完成完整的应用程序，无需传统项目构建。
- Actuator：让你能够深入运行中的Spring Boot应用程序，一探究竟。

Java版本：推荐使用java8

构建一个Sping Boot的Maven项目，强烈推荐Spring Initializr,它从本质上来说就是一个Web应用程序，它能为你生成Spring Boot项目结构。
Spring Initializr有几种用法：
- 1、通过Web界面使用，访问：[http://start.spring.io/](http://start.spring.io/)
- 2、通过ide相关插件创建

@SpringBootApplication是Sprnig Boot项目的核心注解，主要目的是开启自动配置。
使用命令 mvn spring-boot:run”在命令行启动该应用



## 配置文件
spring-boot配置文件application.properties支持的属性列表：[http://docs.spring.io/spring-...](http://docs.spring.io/spring-boot/docs/current/reference/html/common-application-properties.html)
1、直接在要使用的地方通过注解@Value(value=”${configName}”)就可以绑定到你想要的属性上面。
2、在application.properties中的各个参数之间也可以直接引用来使用。
```
com.name="111"
com.want="222"
com.dudu.yearhope=${com.name}-${com.want}
```

3、有时候属性太多了，一个个绑定到属性字段上太累，官方提倡绑定一个对象的bean，这里我们建一个`ConfigBean.java`类，顶部需要使用注解`@ConfigurationProperties(prefix = “com.xxx”)`来指明使用哪个.这点可以参考：org.springframework.boot.autoconfigure.jdbc.DataSourceProperties类的写法

这里配置完还需要在spring Boot入口类加上@EnableConfigurationProperties并指明要加载哪个bean
比如：@EnableConfigurationProperties(DataSourceProperties.class)

4、有时候我们不希望把所有配置都放在application.properties里面，这时候我们可以另外定义一个，如test.properties,路径跟也放在src/main/resources下面。
我们新建一个bean类,如下：
```java
@Configuration
@ConfigurationProperties(prefix = "com.md")
@PropertySource("classpath:test.properties")
public class ConfigTestBean {
    private String name;
    private String want;
    // 省略getter和setter
}
```

5、随机值配置
配置文件中${random} 可以用来生成各种不同类型的随机值，从而简化了代码生成的麻烦，例如 生成 int 值、long 值或者 string 字符串。
```
dudu.number=${random.int}
dudu.uuid=${random.uuid}
dudu.number.less.than.ten=${random.int(10)}
dudu.number.in.range=${random.int[1024,65536]}
```

6、外部配置-命令行参数配置
如java -jar xx.jar --server.port=9090
其中server.port是application.properties里面的选项

7、Profile-多环境配置
在Spring Boot中多环境配置文件名需要满足application-{profile}.properties的格式，其中{profile}对应你的环境标识，比如：
- application-dev.properties：开发环境
- application-prod.properties：生产环境

想要使用对应的环境，有两种方式：
- 只需要在application.properties中使用`spring.profiles.active`属性来设置，值对应上面提到的{profile}，这里就是指dev、prod这2个。
- 当然你也可以用命令行启动的时候带上参数：-jar xxx.jar --spring.profiles.active=dev，还可以像这样设置SPRING_PROFILES_ACTIVE环境变量：$ export SPRING_PROFILES_ACTIVE=production

在代码里，我们还可以直接用`@Profile`注解来进行配置
如下：
```
/**
  * 测试数据库
  */
@Component
@Profile("testdb")
public class TestDBConnector implements DBConnector {
    @Override
    public void configure() {
        System.out.println("testdb");
    }
}
/**
 * 生产数据库
 */
@Component
@Profile("devdb")
public class DevDBConnector implements DBConnector {
    @Override
    public void configure() {
        System.out.println("devdb");
    }
}
```
通过在配置文件激活具体使用哪个实现类`spring.profiles.active=testdb`



## 启动原理解析
```
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

### @SpringBootApplication与@EnableAutoConfiguration
@SpringBootApplication背后：打开源码看看，有三个Annotation原注解：
- @Configuration（@SpringBootConfiguration点开查看发现里面还是应用了@Configuration）
- @EnableAutoConfiguration
- @ComponentScan(如果不指定basePackages等属性，则默认Spring框架实现会从声明@ComponentScan所在类的package进行扫描。)

`@EnableAutoConfiguration`这个Annotation最为重要(点开源码看看),Spring框架提供的各种名字为@Enable开头的Annotation定义，比如@EnableScheduling、@EnableCaching、@EnableMBeanExport等，@EnableAutoConfiguration的理念和做事方式其实一脉相承，简单概括一下就是，**借助@Import的支持，收集和注册特定场景相关的bean定义。**
- @EnableScheduling是通过@Import将Spring调度框架相关的bean定义都加载到IoC容器。
- @EnableMBeanExport是通过@Import将JMX相关的bean定义加载到IoC容器。
- **而@EnableAutoConfiguration也是借助@Import的帮助，将所有符合自动配置条件的bean定义加载到IoC容器，仅此而已！**

借助于Spring框架原有的一个工具类：SpringFactoriesLoader的支持，@EnableAutoConfiguration可以智能的自动配置
`SpringFactoriesLoader`属于Spring框架私有的一种扩展方案，其主要功能就是从指定的配置文件`META-INF/spring.factories`加载配置。
@EnableAutoConfiguration自动配置的魔法骑士是从classpath中搜寻所有的META-INF/spring.factories配置文件，并将其中org.springframework.boot.autoconfigure.EnableAutoConfiguration对应的配置项(主要在SpringBoot的autoconfigure依赖包中)通过反射（Java Refletion）实例化为对应的标注了@Configuration的JavaConfig形式的IoC容器配置类，然后汇总为一个并加载到IoC容器。

### SpringApplication执行流程
1. 如果使用的是SpringApplication.run静态方法，那么，这个方法里面首先要创建一个SpringApplication实例，在初始化的时候，它会提前做几件事情：
- 根据classpath里面是否存在某个特征类（org.springframework.web.context.ConfigurableWebApplicationContext）来决定是否应该创建一个为Web应用使用的ApplicationContext类型。
- 使用SpringFactoriesLoader在应用的classpath中查找并加载所有可用的`ApplicationContextInitializer,ApplicationListener`

2. 执行run方法的逻辑，首先遍历执行所有通过SpringFactoriesLoader可以查找到并加载的SpringApplicationRunListener。调用它们的started()方法。然后创建并配置当前Spring Boot应用将要使用的Environment（包括配置要使用的PropertySource以及Profile）。然后遍历调用所有SpringApplicationRunListener的environmentPrepared()的方法。之后，如果SpringApplication的showBanner属性被设置为true，则打印banner。

3. 根据用户是否明确设置了applicationContextClass类型以及初始化阶段的推断结果，决定该为当前SpringBoot应用创建什么类型的ApplicationContext并创建完成，然后根据条件决定是否添加ShutdownHook，决定是否使用自定义的BeanNameGenerator，决定是否使用自定义的ResourceLoader，当然，最重要的，将之前准备好的Environment设置给创建好的ApplicationContext使用。

4. ApplicationContext创建好之后，遍历调用先前找到的ApplicationContextInitializer的initialize（applicationContext）方法来对已经创建好的ApplicationContext进行进一步的处理。
5. 遍历调用所有SpringApplicationRunListener的contextPrepared()方法。
6. 最核心的一步，将之前通过@EnableAutoConfiguration获取的所有配置以及其他形式的IoC容器配置加载到已经准备完毕的ApplicationContext。
7. 遍历调用所有SpringApplicationRunListener的contextLoaded()方法。
8. 调用ApplicationContext的refresh()方法，完成IoC容器可用的最后一道工序。
9. 查找当前ApplicationContext中是否注册有CommandLineRunner，如果有，则遍历执行它们。
10. 正常情况下，遍历执行SpringApplicationRunListener的finished()方法.



## 模板引擎
Spring Boot为Spring MVC提供适用于多数应用的自动配置功能。在Spring默认基础上，自动配置添加了以下特性：
- 引入ContentNegotiatingViewResolver和BeanNameViewResolver beans。
- 对静态资源的支持，包括对WebJars的支持。
- 自动注册Converter，GenericConverter，Formatter beans。
- 对HttpMessageConverters的支持。
- 自动注册MessageCodeResolver。
- 对静态index.html的支持。
- 对自定义Favicon的支持。

如果想全面控制Spring MVC，你可以添加自己的@Configuration，并使用@EnableWebMvc对其注解。如果想保留Spring Boot MVC的特性，并只是添加其他的MVC配置(拦截器，formatters，视图控制器等)，你可以添加自己的`WebMvcConfigurerAdapter`类型的@Bean（不使用@EnableWebMvc注解）
例如：配置一个拦截器
```java
@Configuration
public class WebConfiguration extends WebMvcConfigurerAdapter {
    @Bean
    public RemoteIpFilter remoteIpFilter() {
        return new RemoteIpFilter();
    }-
    @Bean
    public LocaleChangeInterceptor localeChangeInterceptor() {
        return new LocaleChangeInterceptor();
    }
    @Override
    public void addInterceptors(InterceptorRegistry registry {
        registry.addInterceptor(localeChangeInterceptor());
    }
}
```

Spring Boot 默认为我们提供了静态资源处理，使用 WebMvcAutoConfiguration 中的配置各种属性。
建议大家使用Spring Boot的默认配置方式，提供的静态资源映射如下:
- classpath:/META-INF/resources
- classpath:/resources
- classpath:/static
- classpath:/public

这使用了Spring MVC的ResourceHttpRequestHandler.

Spring Boot支持多种模版引擎包括：
- FreeMarker
- Groovy
- Thymeleaf(官方推荐)
- Mustache

JSP技术Spring Boot官方是不推荐的，原因有三：
- tomcat只支持war的打包方式，不支持可执行的jar。
- Jetty 嵌套的容器不支持jsp
- 创建自定义error.jsp页面不会覆盖错误处理的默认视图，而应该使用自定义错误页面

当你使用上述模板引擎中的任何一个，它们默认的模板配置路径为：src/main/resources/templates。当然也可以修改这个路径


### 配置错误页面
Spring Boot自动配置的默认错误处理器会查找名为`error`的视图，如果找不到就用默认的白标错误视图，如图3-1所示。因此，最简单的方法就是创建一个自定义视图，让解析出的视图名为
error。
这一点归根到底取决于错误视图解析时的视图解析器。
- 实现了Spring的View接口的Bean，其 ID为error（由Spring的BeanNameViewResolver所解析）。
- 如果配置了Thymeleaf，则有名为error.html的Thymeleaf模板。
- 如果配置了FreeMarker，则有名为error.ftl的FreeMarker模板。
- 如果配置了Velocity，则有名为error.vm的Velocity模板。
- 如果是用JSP视图，则有名为error.jsp的JSP模板。

Spring Boot会为错误视图提供如下错误属性。
- timestamp：错误发生的时间。
- status：HTTP状态码。
- error：错误原因。
- exception：异常的类名。
- message：异常消息（如果这个错误是由异常引起的）。
- errors：BindingResult异常里的各种错误（如果这个错误是由异常引起的）。
- trace：异常跟踪信息（如果这个错误是由异常引起的）。
- path：错误发生时请求的URL路径。



## 默认日志logback配置
spring-boot-starter-logging
根据不同的日志系统，你可以按如下规则组织配置文件名，就能被正确加载：
- Logback：logback-spring.xml, logback-spring.groovy, logback.xml
- Log4j：log4j-spring.properties, log4j-spring.xml, log4j.properties, log4j.xml
- Log4j2：log4j2-spring.xml, log4j2.xml

如果你不想用logback.xml作为Logback配置的名字，可以通过logging.config属性指定自定义的名字：
logging.config=classpath:logging-config.xml

### 多环境日志输出
据不同环境（prod:生产环境，test:测试环境，dev:开发环境）来定义不同的日志输出，在 logback-spring.xml中使用 springProfile 节点来定义，方法如下：
`文件名称不是logback.xml，想使用spring扩展profile支持，要以logback-spring.xml命名`
```properties
<!-- 测试环境+开发环境. 多个使用逗号隔开. -->
<springProfile name="test,dev">
    <logger name="com.dudu.controller" level="info" />
</springProfile>
<!-- 生产环境. -->
<springProfile name="prod">
    <logger name="com.dudu.controller" level="ERROR" />
</springProfile>
```
可以启动服务的时候指定 profile （如不指定使用默认），如指定prod 的方式为：
`java -jar xxx.jar –spring.profiles.active=prod`


## 数据源与事务配置
1、使用普通jdbc
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
```

application.properties中配置数据源信息。
```properties
spring.datasource.url = jdbc:mysql://localhost:3306/spring?useUnicode=true&characterEncoding=utf-8
spring.datasource.username = root
spring.datasource.password = root
spring.datasource.driver-class-name = com.mysql.jdbc.Driver
```

**自定义数据源**
```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.0.19</version>
</dependency>
```

```java
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

    @Autowired
    private Environment env;

    //destroy-method="close"的作用是当数据库连接不使用的时候,就把该连接重新放到数据池中,方便下次使用调用.
    @Bean(destroyMethod =  "close")
    public DataSource dataSource() {
        DruidDataSource dataSource = new DruidDataSource();
        dataSource.setUrl(env.getProperty("spring.datasource.url"));
        dataSource.setUsername(env.getProperty("spring.datasource.username"));//用户名
        dataSource.setPassword(env.getProperty("spring.datasource.password"));//密码
        dataSource.setDriverClassName(env.getProperty("spring.datasource.driver-class-name"));
        dataSource.setInitialSize(2);//初始化时建立物理连接的个数
        dataSource.setMaxActive(20);//最大连接池数量
        dataSource.setMinIdle(0);//最小连接池数量
        dataSource.setMaxWait(60000);//获取连接时最大等待时间，单位毫秒。
        dataSource.setValidationQuery("SELECT 1");//用来检测连接是否有效的sql
        dataSource.setTestOnBorrow(false);//申请连接时执行validationQuery检测连接是否有效
        dataSource.setTestWhileIdle(true);//建议配置为true，不影响性能，并且保证安全性。
        dataSource.setPoolPreparedStatements(false);//是否缓存preparedStatement，也就是PSCache
        return dataSource;
    }
}
```

## 覆盖Spring Boot 配置

Spring Boot自动配置自带了很多配置类，每一个都能运用
在你的应用程序里。它们都使用了Spring 4.0的条件化配置，可以在运行时判断这个配置是该被运用，还是该被忽略。如:
```java
@Bean
@ConditionalOnMissingBean(JdbcOperations.class)
public JdbcTemplate jdbcTemplate() {
    return new JdbcTemplate(this.dataSource);
}
```

条件注解有如下：
- @ConditionalOnBean 配置了某个特定Bean
- @ConditionalOnMissingBean 没有配置特定的Bean
- @ConditionalOnClass Classpath里有指定的类
- @ConditionalOnMissingClass Classpath里缺少指定的类
- @ConditionalOnExpression 给定的Spring Expression Language（SpEL）表达式计算结果为true
- @ConditionalOnJava Java的版本匹配特定值或者一个范围值
- @ConditionalOnJndi 参数中给定的JNDI位置必须存在一个，如果没有给参数，则要有JNDI
- InitialContext
- @ConditionalOnProperty 指定的配置属性要有一个明确的值
- @ConditionalOnResource Classpath里有指定的资源
- @ConditionalOnWebApplication 这是一个Web应用程序
- @ConditionalOnNotWebApplication 这不是一个Web应用程序

## 单元测试
```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = App.class)
public MyTest{
    @Test
    public void test1(){

    }
}
```

### 测试Web 应用程序
1、普通测试
```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
@WebAppConfiguration
public class MockMvcWebTests {
    @Autowired
    private WebApplicationContext webContext;
    private MockMvc mockMvc;
    @Before
    public void setupMockMvc() {
        mockMvc = MockMvcBuilders.webAppContextSetup(webContext).build();
    }
    @Test
    public void homePage() throws Exception {
        mockMvc.perform(MockMvcRequestBuilders.get("/readingList"))
                .andExpect(MockMvcResultMatchers.status().isOk())
                .andExpect(MockMvcResultMatchers.view().name("readingList"))
                .andExpect(MockMvcResultMatchers.model().attributeExists("books"))
                .andExpect(MockMvcResultMatchers.model().attribute("books",
            Matchers.is(Matchers.empty())));
    }
}
```
首先向/readingList发起一个GET请求，接下来希望该请求处理成功（isOk()会判断HTTP 200响应码），并且视图的逻辑名称为readingList。测试还要断定模型包含一个名为books的属性，该属性是一个空集合。所有的断言都很直观。

2、测试运行中的应用程序
~~在测试类上添加`@Web-IntegrationTest`注解，可以声明你不仅希望Spring Boot为测试创建应用程序上下文，还要启动一个嵌入式的Servlet容器。一旦应用程序运行在嵌入式容器里，你就可以发起真实的HTTP请求，断言结果了。
这里采用@WebIntegration-Test，在服务器里启动了应用程序~~, 以Spring的`RestTemplate`对应用程序发起HTTP请求。
Spring-boot 1.4以后推荐采用**SpringBootTest(webEnvironment=WebEnvironment.DEFINED_PORT) (or RANDOM_PORT)**来代替@WebIntegrationTest,而此类在1.5已经被移除了。[https://github.com/spring-pro...](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-1.4-Release-Notes)
```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = App.class,webEnvironment=WebEnvironment.RANDOM_PORT)
public class WebInRuntimeTest {
    @Value("${local.server.port}")
    private int port;
    @Autowired
    private TestRestTemplate template;
    @Test
    public void test1(){
//        Assert.state(true,"测试成功");
        System.out.println(port);
        ResponseEntity<String> response=template.getForEntity("/test/111", String.class);
        System.out.println(response.getStatusCodeValue());
        System.out.println(response.getHeaders());

        System.out.println(response.getBody());
        Assert.hasText("111","true");
    }
}
```

用随机端口启动服务器,@WebIntegrationTest的value属性接受一个String数组，数组中的每项都是键值对，形如name=value，用来设置测试中使用的属性。要设置server.port，你可以这样做：**@WebIntegrationTest("server.port:0") //使用0表示端口号随机，也可以具体指定如8888这样的固定端口.
或者直接这样@WebIntegrationTest(randomPort=true)**

Spring Boot将local.server.port的值设置为了选中的端口。我们只需使用Spring的@Value注解将其注入即可：
```java
@Value("${local.server.port}")
private int port;
```

3、使用Selenium 测试HTML 页面
```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = App.class)
@WebIntegrationTest(randomPort=true)
public class ServerWebTests {
    private static FirefoxDriver browser;
    @Value("${local.server.port}")
    private int port;
    @BeforeClass
    public static void openBrowser() {
        browser = new FirefoxDriver();
        browser.manage().timeouts().implicitlyWait(10, TimeUnit.SECONDS);
    }
    @Test
    public void addBookToEmptyList() {
        String baseUrl = "http://localhost:" + port;
        browser.get(baseUrl);
        assertEquals("You have no books in your book list",browser.findElementByTagName("div").getText());
        browser.findElementByName("title").sendKeys("BOOK TITLE");
        browser.findElementByTagName("form").submit();
        WebElement dl =
            browser.findElementByCssSelector("dt.bookHeadline");
        assertEquals("BOOK TITLE by BOOK AUTHOR (ISBN: 1234567890)",dl.getText());
    }
    @AfterClass
    public static void closeBrowser() {
        browser.quit();
    }
}
```

## 外部tomcat部署war配置
当我们不想使用内嵌tomcat部署时，我们也可以使用外部tomcat，并打包成war：
1、继承SpringBootServletInitializer
外部容器部署的话，就不能依赖于Application的main函数了，而是要以类似于web.xml文件配置的方式来启动Spring应用上下文，此时我们需要在启动类中继承SpringBootServletInitializer并实现configure方法：
```java
public class Application extends SpringBootServletInitializer {
    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
        return application.sources(Application.class);
    }
}
```

这个类的作用与在web.xml中配置负责初始化Spring应用上下文的监听器作用类似，只不过在这里不需要编写额外的XML文件了。

SpringBootServletInitializer是一个支持Spring WebApplicationInitializer实现。除了配置Spring的Dispatcher-Servlet，还会在Spring应用程序上下文里查找**Filter、Servlet或ServletContextInitializer类型的Bean**，把它们绑定到Servlet容器里。

还有一点值得注意：就算我们在构建的是WAR文件，这个文件仍旧可以脱离应用服务器直接运行。如果你没有删除Application里的main()方法，构建过程生成的WAR文件仍可直接运行，一如可执行的JAR文件：
>$ java -jar readinglist-0.0.1-SNAPSHOT.war

这样一来，同一个部署产物就能有两种部署方式了！

2、pom.xml修改tomcat相关的配置
如果要将最终的打包形式改为war的话，还需要对pom.xml文件进行修改，因为spring-boot-starter-web中包含内嵌的tomcat容器，所以直接部署在外部容器会冲突报错。所以需要进行依赖排除
```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.apache.tomcat.embed</groupId>
    <artifactId>tomcat-embed-jasper</artifactId>
    <scope>provided</scope>
</dependency>
```

## 开发常用的热部署方式汇总
- Spring Loaded
- spring-boot-devtools
- IDE的JRebel插件

1、Spring Loaded 实现热部署
Spring Loaded是一个用于在JVM运行时重新加载类文件更改的JVM代理,Spring Loaded允许你动态的新增/修改/删除某个方法/字段/构造方法,同样可以修改作用在类/方法/字段/构造方法上的注解.也可以新增/删除/改变枚举中的值。

spring-loaded是一个开源项目,项目地址:[https://github.com/spring-pro...](https://github.com/spring-projects/spring-loaded)

Spring Loaded有两种方式实现，分别是Maven引入依赖方式或者添加启动参数方式
```xml
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <dependencies>
        <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>springloaded</artifactId>
        <version>1.2.6.RELEASE</version>
        </dependency>
    </dependencies>
</plugin>
```

启动：mvn spring-boot:run
注意：maven依赖的方式只适合spring-boot:run的启动方式，右键那种方式不行。

出现如下配置表实配置成功：
>[INFO] Attaching agents: [C:Userstengj.m2repositoryorgspringframeworkspringloaded1.2.6.R

添加启动参数方式
这种方式是右键运行启动类.首先先下载对应的springloaded-xxx.RELEASE.jar，可以去上面提到的官网获取,在VM options中输入
>-javaagent:< pathTo >/springloaded-{VERSION}.jar

上面2种方式随便选择一种即可,当系统通过 mvn spring-boot:run启动或者 右键application debug启动Java文件时，系统会监视classes文件，当有classes文件被改动时，系统会重新加载类文件，不用重启启动服务。
**注：IDEA下需要重新编译文件 Ctrl+Shift+F9或者编译项目 Ctrl+F9**

在 Spring Boot，模板引擎的页面默认是开启缓存，如果修改页面内容，刷新页面是无法获取修改后的页面内容，所以，如果我们不需要模板引擎的缓存，可以进行关闭。
```properties
spring.freemarker.cache=false
spring.thymeleaf.cache=false
spring.velocity.cache=false
spring.mustache.cache=false
```

不过还是有一些情况下需要重新启动，不可用的情况如下：
- 1：对于一些第三方框架的注解的修改，不能自动加载，比如：spring mvc的@RequestMapping
- 2：application.properties的修改也不行
- 3：log4j的配置文件的修改不能即时生效

2、spring-boot-devtools 实现热部署
spring-boot-devtools为应用提供一些开发时特性，包括默认值设置，自动重启，livereload等。
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <optional>true</optional>
    </dependency>
</dependencies>
```
将依赖关系标记为可选<optional\>true</optional\>是一种最佳做法，可以防止使用项目将devtools传递性地应用于其他模块。在Spring Boot集成Thymeleaf时，spring.thymeleaf.cache属性设置为false可以禁用模板引擎编译的缓存结果。

现在，devtools会自动帮你做到这些，禁用所有模板的缓存，包括Thymeleaf, Freemarker, Groovy Templates, Velocity, Mustache等。

自动重启的原理在于spring boot使用两个classloader：不改变的类（如第三方jar）由base类加载器加载，正在开发的类由restart类加载器加载。应用重启时，restart类加载器被扔掉重建，而base类加载器不变，这种方法意味着应用程序重新启动通常比“冷启动”快得多，因为base类加载器已经可用并已填充。

所以，当我们开启devtools后，classpath中的文件变化会导致应用自动重启。当然不同的IDE效果不一样，Eclipse中保存文件即可引起classpath更新(注：需要打开自动编译)，从而触发重启。而IDEA则需要自己手动CTRL+F9重新编译一下（感觉IDEA这种更好，不然每修改一个地方就重启，好蛋疼）

**排除静态资源文件**
静态资源文件在改变之后有时候没必要触发应用程序重启，例如thymeleaf模板文件就可以实时编辑，默认情况下，更改/META-INF/maven, /META-INF/resources ,/resources ,/static ,/public 或/templates下的资源不会触发重启，而是触发live reload（devtools内嵌了一个LiveReload server，当资源发生改变时，浏览器刷新,需要浏览器插件支持）。

可以使用spring.devtools.restart.exclude属性配置，例如
```
spring.devtools.restart.exclude=static/**,public/**
```

如果想保留默认配置，同时增加新的配置，则可使用
spring.devtools.restart.additional-exclude属性

观察额外的路径
如果你想观察不在classpath中的路径的文件变化并触发重启，则可以配置 spring.devtools.restart.additional-paths 属性。

不在classpath内的path可以配置spring.devtools.restart.additionalpaths属性来增加到监视中，同时配置spring.devtools.restart.exclude可以选择这些path的变化是导致restart还是live reload。

关闭自动重启
设置 spring.devtools.restart.enabled 属性为false，可以关闭该特性。可以在application.properties中设置，也可以通过设置环境变量的方式。
```java
public static void main(String[] args) {
    System.setProperty("spring.devtools.restart.enabled", "false");
    SpringApplication.run(MyApp.class, args);
}
```

**使用一个触发文件**
若不想每次修改都触发自动重启，可以设置spring.devtools.restart.trigger-file指向某个文件，只有更改这个文件时才触发自动重启。

自定义自动重启类加载器
默认时，IDE中打开的项目都会由restart加载器加载，jar文件由Base加载器加载，但是若你使用multi-module的项目，并且不是所有模块都被导入到IDE中，此时会导致加载器不一致。这时你可以创建META-INF/spring-devtools.properties文件，并增加restart.exclude.XXX，restart.include.XXX来配置哪些jar被restart加载，哪些被base加载。如：
```
restart.include.companycommonlibs=/mycorp-common-[\\w-]+\.jar
restart.include.projectcommon=/mycorp-myproj-[\\w-]+\.jar
```

如果您不想在应用程序运行时启动LiveReload服务器，则可以将spring.devtools.livereload.enabled属性设置为false。
一次只能运行一个LiveReload服务器。开始应用程序之前，请确保没有其他LiveReload服务器正在运行。
如果你的IDE启动多个应用程序，则只有第一个应用程序将支持LiveReload。

3、JRebel插件方式 ：略



## 数据库迁移库支持
Spring Boot为两款流行的数据库迁移库提供了自动配置支持。
- Flyway（[http://flywaydb.org](http://flywaydb.org)）
- Liquibase（[http://www.liquibase.org](http://www.liquibase.org)）

1. 用Flyway定义数据库迁移过程
```xml
<dependency>
<groupId>org.flywayfb</groupId>
<artifactId>flyway-core</artifactId>
</dependency>
```

Flyway是一个非常简单的开源数据库迁移库，使用SQL来定义迁移脚本。它的理念是，每个脚本都有一个版本号，Flyway会顺序执行这些脚本，让数据库达到期望的状态。它也会记录已执行的脚本状态，不会重复执行。Flyway脚本就是SQL。让其发挥作用的是其在Classpath里的位置和文件名。Flyway脚本都遵循一个命名规范，V版本号__描述.sql,如`V1__initdb.sql`。

Flyway脚本需要放在src/main/resources/db/migration里。你还需要将spring.jpa.hibernate.ddl-auto设置为none，由此告知Hibernate不要创建数据表。

原理：在应用程序部署并运行起来后，Spring Boot会检测到Classpath里的Flyway，自动配置所需的Bean。Flyway会依次查看/db/migration里的脚本，如果没有执行过就运行这些脚本。每个脚本都执行过后，向schema_version表里写一条记录。应用程序下次启动时，Flyway会先看schema_version里的记录，跳过那些脚本。

2. 用Liquibase定义数据库迁移过程
相比Flyway的优点，数据库无关性，脚本不是用sql写，而是支持yaml,json,xml等格式。
```
<dependency>
<groupId>org.liquibase</groupId>
<artifactId>liquibase-core</artifactId>
</dependency>
```
具体介绍：略。



## Actuator监控应用程序状态
运行中的应用程序就像礼物盒。你可以刺探它，作出合理的推测，猜测它的运行情况。但如
何了解真实的情况呢？有没有一种办法能让你深入应用程序内部一窥究竟，了解它的行为，检查
它的健康状况，甚至触发一些操作来影响应用程序呢？
Spring Boot的Actuator。它提供了很多生产级的特性，比如监控和度
量Spring Boot应用程序。Actuator的这些特性可以通过众多REST端点、远程shell和JMX获得。

```xml
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

1、Actuator的Web端点
端点可以分为三大类：配置端点、度量端点和其他端点
- GET /autoconfig 提供了一份自动配置报告，记录哪些自动配置条件通过了，哪些没通过
- GET /configprops 描述配置属性（包含默认值）如何注入Bean
- GET /beans 描述应用程序上下文里全部的Bean，以及它们的关系
- GET /dump 获取线程活动的快照
- GET /env 获取全部环境属性
- GET /env/{name} 根据名称获取特定的环境属性值
- GET /health 报告应用程序的健康指标，这些值由HealthIndicator的实现类提供
- GET /info 获取应用程序的定制信息，这些信息由info打头的属性提供
- GET /mappings 描述全部的URI路径，以及它们和控制器（包含Actuator端点）的映射关系
- GET /metrics 报告各种应用程序度量信息，比如内存用量和HTTP请求计数
- GET /metrics/{name} 报告指定名称的应用程序度量值
- POST /shutdown 关闭应用程序，要求endpoints.shutdown.enabled设置为true
- GET /trace 提供基本的HTTP请求跟踪信息（时间戳、HTTP头等）

/beans端点产生的报告能告诉你Spring应用程序上下文里都有哪些Bean。
/autoconfig端点能告诉你为什么会有这个Bean，或者为什么没有这个Bean。

Spring Boot自动配置构建于Spring的条件化配置之上。它提供了众多带有@Conditional注解的配置类，根据条件决定是否要自动配置这些Bean。/autoconfig端点提供了一个报告，列出了计算过的所有条件，根据条件是否通过进行分组。

/env端点会生成应用程序可用的所有环境属性的列表，无论这些属性是否用到。这其中包括
环境变量、JVM属性、命令行参数，以及applicaition.properties或application.yml文件提供的属性。

/metrics端点提供了一些针对Web请求的基本计数器和计时器，但那些度量值缺少详细信息。知道所处理请求的更多信息是很有帮助的，尤其是在调试时，所以就有了/trace这个端点。

/trace端点能报告所有Web请求的详细信息，包括请求方法、路径、时间戳以及请求和响应的
头信息

2、使用:CRaSH shell

3、通过JMX 监控应用程序

## 参考：

Book: Spring-Boot In Action
[嘟嘟独立博客](http://tengj.top/categories/Spring-Boot%E5%B9%B2%E8%B4%A7%E7%B3%BB%E5%88%97/)
[https://github.com/tengj/Spri...](https://github.com/tengj/SpringBootDemo)
