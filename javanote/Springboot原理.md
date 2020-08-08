# Springboot原理

## 源码分析

### 1.SpringBootApplication

主要代码如下：

```java
// 指明批注类型是自动继承的
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
// 自定义的TypeFilter类.会被过滤掉,不会加入bean容器
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
public @interface SpringBootApplication {
    @AliasFor(
        annotation = EnableAutoConfiguration.class
    )
    Class<?>[] exclude() default {};
	// 这里使用别名，可以理解为在@SpringbootApplication注解中引入@EnableAutoConfiguration注解（继承）
    @AliasFor(
        annotation = EnableAutoConfiguration.class
    )
    String[] excludeName() default {};
	// 这里使用别名，可以理解为@SpringbootApplication(scanBasePackages="")会使用@ComponentScan的basePacksges别名
    @AliasFor(
        annotation = ComponentScan.class,
        attribute = "basePackages"
    )
    String[] scanBasePackages() default {};

    @AliasFor(
        annotation = ComponentScan.class,
        attribute = "basePackageClasses"
    )
    Class<?>[] scanBasePackageClasses() default {};
}
```

**@Inherited**：指示批注类型是自动继承的。

**@SpringBootConfiguration**：标注这个类是个配置类，只是@Configuration注解的派生注解，跟@Configuration注解的功能一致，标注这个类是一个配置类，只不过@SpringBootConfiguration是springboot的注解，而@Configuration是spring的注解

**@ComonentScan**用于类或接口上，主要是指定扫描路径，spring会把指定路径下带有@Controller、@Service、@Component、@Repository等等注解的类自动装配到bean容器。

​	**exclideFilters**：指定排除Filter条件的类

​	**basePackages、value**：指定扫描路径，如果为空则以@ComponentScan注解所在的包为基本的扫描路径

​	**basePackageClasses**：指定具体扫描的类

​	**includeFilters**：指定满足Filter条件的类

**@AliasFor**：

​	(1).同个注解中的两个属性互为别名

​	(2).继承父注解的属性，使其拥有更强大的功能

- ```java
  @RunWith(SpringJUnit4ClassRunner.class)
  @ContextConfiguration(classes = AopConfig.class)
  public class AopUtilsTest {
  ```

  每次写[@ContextConfiguration](https://github.com/ContextConfiguration)(classes = AopConfig.class)太麻烦了，我想写得简单一点，我就可以定义一个这样的标签：

  ```java
  @Retention(RetentionPolicy.RUNTIME)
  @ContextConfiguration
  public @interface STC {
   
      @AliasFor(value = "classes", annotation = ContextConfiguration.class)
      Class<?>[] cs() default {};
   
  }
  ```

  我觉得classes属性太长了，所以我创建了一个cs属性，为了让这个属性等同于[@ContextConfiguration](https://github.com/ContextConfiguration)属性中的classes属性，我使用了[@AliasFor](https://github.com/AliasFor)标签，分别设置了value（即作为哪个属性的别名）和annotation（即作为哪个注解）；

  使用我们的STC：

  ```java
  
  @RunWith(SpringJUnit4ClassRunner.class)
  @STC(cs = AopConfig.class)
  public class AopUtilsTest {
   
      @Autowired
      private IEmployeeService service;
  ```

  [参考1](https://www.cnblogs.com/sandyflower/p/10877291.html)        [参考2](https://blog.csdn.net/wolfcode_cn/article/details/80654730)

由上可知核心功能是由EnableAutoConfiguration提供的



### 2.EnableAutoConfiguration

主要代码如下：

```java
@AutoConfigurationPackage
 // 将AutoConfigurationImportSelector导入ioc容器
@Import({AutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {
    String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

    Class<?>[] exclude() default {};

    String[] excludeName() default {};
}

```

**@AutoConfigurationPackage**：注解的作用是将 **添加该注解的类所在的package** 作为 **自动配置package** 进行管理。@SpringBootApplication -> @EnableAutoConfiguration -> @AutoConfigurationPackage。也就是说当SpringBoot应用启动时默认会将启动类所在的package作为自动配置的package。

**@Import**：@Import只能用在类上 ，@Import通过快速导入的方式实现把实例加入spring的IOC容器中

由上可知EnableAutoConfiguration将AutoConfigurationImportSelector导入ioc容器

### 3.AutoConfigurationImportSelector

核心代码

![](https://kulalasmile.oss-cn-hangzhou.aliyuncs.com/PicGo20200805100621.png)

selectImports方法里调用`List<String> configurations = getCandidateConfigurations(annotationMetadata,attributes);`

![](https://kulalasmile.oss-cn-hangzhou.aliyuncs.com/PicGo20200805100825.png)

getCandidateConfigurations又调用了`List<String> configurations = SpringFactoriesLoader.loadFactoryNames(getSpringFactoriesLoaderFactoryClass(), getBeanClassLoader());`

![](https://kulalasmile.oss-cn-hangzhou.aliyuncs.com/PicGo20200805101015.png)



![](https://kulalasmile.oss-cn-hangzhou.aliyuncs.com/PicGo20200805101147.png)

```xml
# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration,\
org.springframework.boot.autoconfigure.batch.BatchAutoConfiguration,\
org.springframework.boot.autoconfigure.cache.CacheAutoConfiguration,\
org.springframework.boot.autoconfigure.cassandra.CassandraAutoConfiguration,\
org.springframework.boot.autoconfigure.cloud.CloudServiceConnectorsAutoConfiguration,\
org.springframework.boot.autoconfigure.context.ConfigurationPropertiesAutoConfiguration,\
org.springframework.boot.autoconfigure.context.MessageSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.context.PropertyPlaceholderAutoConfiguration,\
org.springframework.boot.autoconfigure.couchbase.CouchbaseAutoConfiguration,\
org.springframework.boot.autoconfigure.dao.PersistenceExceptionTranslationAutoConfiguration,\
org.springframework.boot.autoconfigure.data.cassandra.CassandraDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.cassandra.CassandraReactiveDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.cassandra.CassandraReactiveRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.cassandra.CassandraRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.couchbase.CouchbaseDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.couchbase.CouchbaseReactiveDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.couchbase.CouchbaseReactiveRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.couchbase.CouchbaseRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchAutoConfiguration,\
org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.jdbc.JdbcRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.jpa.JpaRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.ldap.LdapRepositoriesAutoConfiguration,\

```



### 4.xxAutoConfiguration

```java
@Configuration
@ConditionalOnClass({JobLauncher.class, DataSource.class, JdbcOperations.class})
@AutoConfigureAfter({HibernateJpaAutoConfiguration.class})
@ConditionalOnBean({JobLauncher.class})
@EnableConfigurationProperties({BatchProperties.class})
@Import({BatchConfigurerConfiguration.class})
public class BatchAutoConfiguration {
```

```java
@Configuration
@ConditionalOnClass({EnableAspectJAutoProxy.class, Aspect.class, Advice.class, AnnotatedElement.class})
@ConditionalOnProperty(
        prefix = "spring.aop",
    name = {"auto"},
    havingValue = "true",
    matchIfMissing = true
)
public class AopAutoConfiguration {
```

```java
@Configuration
@ConditionalOnClass({Cluster.class})
@EnableConfigurationProperties({CassandraProperties.class})
public class CassandraAutoConfiguration {
```

打开spring.factories任意一个AutoConfiguration文件，一般都有下面的条件注解（在spring-boot-autoconfigure-x.jar的org.springframwork,boot.autoconfigure.condition包下）：

**@ConditionalOnBean**：当容器里有指定的Bean的条件下。

**@ConditionalOnClass**：当类路径下有指定的类的条件下

**@ConditionalOnExpression**：基于SpEL表达式作为判断条件

**@ConditionalOnJava**：基于JVM版本进行判断

**@ConditionalOnProperty**：指定的路径是否有指定的值

它的作用是按照一定的条件进行判断，满足条件给容器注册bean。

例如：

```java
@Configuration
@AutoConfigureOrder(-2147483648)
@ConditionalOnClass({ServletRequest.class})
@ConditionalOnWebApplication(
    type = Type.SERVLET
)
@EnableConfigurationProperties({ServerProperties.class})
@Import({ServletWebServerFactoryAutoConfiguration.BeanPostProcessorsRegistrar.class, EmbeddedTomcat.class, EmbeddedJetty.class, EmbeddedUndertow.class})
public class ServletWebServerFactoryAutoConfiguration {
```

在ServletWebServerFactoryAutoConfiguration类上，有一个@EnableConfigurationProperties注解：**开启配置属性**，而它后面的参数是一个ServerProperties类，这就是习惯优于配置的最终落地点

```java
@ConfigurationProperties(
    prefix = "server",
    ignoreUnknownFields = true
)
public class ServerProperties {
    private Integer port;
    private InetAddress address;
    @NestedConfigurationProperty
    private final ErrorProperties error = new ErrorProperties();
    private Boolean useForwardHeaders;
    private String serverHeader;
```

在这个类上，我们看到了一个非常熟悉的注解：**@ConfigurationProperties**，它的作用就是从配置文件中绑定属性到对应的bean上，而**@EnableConfigurationProperties**负责导入这个已经绑定了属性的bean到spring容器中（见上面截图）。那么所有其他的和这个类相关的属性都可以在全局配置文件中定义，也就是说，真正“限制”我们可以在全局配置文件中配置哪些属性的类就是这些**XxxxProperties**类，它与配置文件中定义的prefix关键字开头的一组属性是唯一对应的。

至此，我们大致可以了解。在全局配置的属性如：server.port等，通过@ConfigurationProperties注解，绑定到对应的XxxxProperties配置实体类上封装为一个bean，然后再通过@EnableConfigurationProperties注解导入到Spring容器中。

而诸多的XxxxAutoConfiguration自动配置类，就是Spring容器的JavaConfig形式，作用就是为Spring 容器导入bean，而所有导入的bean所需要的属性都通过xxxxProperties的bean来获得。

可能到目前为止还是有所疑惑，但面试的时候，其实远远不需要回答的这么具体，你只需要这样回答：

> Spring Boot启动的时候会通过@EnableAutoConfiguration注解找到META-INF/spring.factories配置文件中的所有自动配置类，并对其进行加载，而这些自动配置类都是以AutoConfiguration结尾来命名的，它实际上就是一个JavaConfig形式的Spring容器配置类，它能通过以Properties结尾命名的类中取得在全局配置文件中配置的属性如：server.port，而XxxxProperties类是通过@ConfigurationProperties注解与全局配置文件中对应的属性进行绑定的。



### 5.手写自动配置

引入pom.xml

```xml
 <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-autoconfigure</artifactId>
 </dependency>
```

新建xxxProperties类用来配置属性

```java
@ConfigurationProperties(prefix = "hello")
public class HelloProperties {
    private static final String MSG = "world";

    private String msg = MSG;

    public void setMsg(String msg) {
        this.msg = msg;
    }

    public String getMsg() {
        return msg;
    }
}
```

新建判断依据类，根据此类的存在与否来创建这个类的Bean，这个类可以是第三方类库的类

```java
public class HelloService {

    private String msg;

    public String sayHello(){
        return "hello" + msg;
    }

    public String getMsg() {
        return msg;
    }

    public void setMsg(String msg) {
        this.msg = msg;
    }
}
```

新建自动配置类，根据HelloServiceProperties提供的参数，并通过@ConditionOnClass判断HelloService这个类在类路径中是否存在

```java
@Configuration
@EnableConfigurationProperties(HelloProperties.class)
@ConditionalOnClass(HelloService.class)
@ConditionalOnProperty(prefix = "hello", value = "enable", matchIfMissing = true)
public class HelloServiceAutoConfiguration {
    @Autowired
    private HelloProperties helloProperties;

    @Bean
    @ConditionalOnMissingBean(HelloService.class)
    public HelloService helloService(){
        HelloService helloService = new HelloService();
        helloService.setMsg(helloProperties.getMsg());
        return helloService;
    }

}

```

新建注册配置，在resource下新建MATE-INF/spring-factories

```
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
    com.yvon.study.HelloServiceAutoConfiguration

```

新建springboot项目，导入依赖

```xml
<dependency>
            <groupId>com.yvon</groupId>
            <artifactId>study</artifactId>
            <version>0.0.1-SNAPSHOT</version>
        </dependency>
```

在代码中直接注入HelloService这个Bean

```java
@SpringBootApplication
@RestController
public class HelloApplication {

    @Autowired
    HelloService helloService;

    @RequestMapping("/")
    public String index() {
        return helloService.sayHello();
    }
    public static void main(String[] args) {
        SpringApplication.run(HelloApplication.class, args);
    }

}
```

结果：

![](https://kulalasmile.oss-cn-hangzhou.aliyuncs.com/PicGo20200805114637.png)

配置文件

![](https://kulalasmile.oss-cn-hangzhou.aliyuncs.com/PicGo20200805114955.png)

结果：

![](https://kulalasmile.oss-cn-hangzhou.aliyuncs.com/PicGo20200805115024.png)

