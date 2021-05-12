---
title: 初学SpringBoot
tag: 
  - java
  - springboot
typora-root-url: ../
---

<!--more-->

## 一、SpringBoot配置

### 1、配置文件

spring boot使用一个全局的配置文件，配置文件名是固定的

* application.properties

* application.yml

配置文件的作用： 修改spring boot配置的默认值（比如改端口号什么的



YAML：标记语言，以数据为中心，比json、xml更适合做配置文件

YAML配置示例

```yaml
server:
  port: 8081
```

### 2、YAML语法

#### 1、基本语法

key: value (空格必须有)

以**空格**的缩进来空值层级关系（类python

属性和值是大小写敏感的

### 2、值的写法

#### 字面量：普通的值（数字，字符串，布尔）：

​	k: v //字面量直接来写，字符串不用加上单引号双引号

​	"": 双引号：不会转义字符串里面的特殊字符

​	'': 单引号：会转义特殊字符，特殊字符最终只是一个普通的字符串数据

#### 对象（键值对）：

​	k: v //在下一行来写对象的属性和值的关系

​	对象还是键值对

```yaml
people:
	name: zhangsan
	age: 18
```

​	行内写法

```yaml
people: {name: zhangsan, age: 18}
```



#### 数组（List、set):

用- v来表示数组中的一个元素

```yaml
pets:
 - cat
 - dog
 - pig
```

行内写法

```yaml
pets: [cat, dog, pog]
```

### 3、配置文件注入

配置文件

```yaml
person:
  name: 张三
  age: 18
  boss: false
  birthday: 2019/3/27

  maps: {k1: v1, k2: v2}
  lists:
    - 李四
    - 赵六
  dog:
    name: yyy
    age: 2
```

javaBean

```java
/**
 * 将配置文件中每一个属性的值，映射到这个组建中
 * @ConfigurationProperties 告诉SpringBoot将本类中所有属性和配置文件中的值进行绑定
 *
 * 只有这个组件是容器中的组件，才能使用容器提供的功能类，比如快速绑定
 * @Component 加入到容器中
 */
@Component
@ConfigurationProperties(prefix = "person") //定位到配置文件中的person
public class Person {
    private String name;
    private Integer age;
    private Boolean boss;
    private Date birthday;

    private Map<String, Object> maps;
    private List<Object> lists;
    private Dog dog;

    /*setter getter toString*/
}
```

我们可以导入配置文件处理器，以后写配置就有提示 

```xml
<!--导入配置文件处理器，配置文件进行绑定就会有提示-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>
        </dependency>
```

#### @Value获取值和@ConfigurationProperties获取值比较

|                      | @ConfigurationProperties                                     | @Value     |
| -------------------- | ------------------------------------------------------------ | ---------- |
| 功能                 | 批量注入配置文件中的属性                                     | 一个个指定 |
| 松散绑定（松散语法） | 支持                                                         | 不支持     |
| SpEl                 | 不支持                                                       | 支持       |
| JSR303数据校验       | 支持（比如数据注明邮箱格式后编译器会检查格式合法性（@Validated | 不支持     |
| 复杂类型封装         | 支持                                                         | 不支持     |

如果说只是在某个业务逻辑中需要获取一下配置文件中的某项值，那么@Value即可

如果说是专门的JavaBean和配置文件进行映射，嗯

#### @PropertySource和@ImportResource

@PropertySource加载指定配置文件

```java
@Component
@ConfigurationProperties(prefix = "person") //定位到配置文件中的person 默认是全局配置文件
@PropertySource(value = {"classpath:person.properties"}) //加上可以指定配置文件
public class Person {

//    @Value("${person.name}") //通过Value注解获得值
    private String name;
    private Integer age;
    private Boolean boss;
    private Date birthday;

    private Map<String, Object> maps;
    private List<Object> lists;
    private Dog dog;
}
```

@ImportSource导入Spring配置文件，让配置文件中的内容生效   xxxx.xml

Spring Boot中替代地使用@Bean给容器添加组件

```java
/**
 * @Configuration 指明当前类是一个配置类，就是用来替代之前的spring配置
 * 以前在配置文件中是用<bean></bean>标签来加组件，
 */
@Configuration
public class MyAppConfig {

    /**
     *
     * @return 将方法的返回值添加到容器中 容器中组件默认的id就是方法名
     */
    @Bean
    public HelloService helloService(){
        System.out.println("配置类@Bean给容器中添加组件了");
        return new HelloService();
    }

}
```

### 4、配置文件占位符

#### 1、随机数

* random.value 

* {random.int} 

* ${random.long} 

* random.int(10) 

* {random.int[1024, 65535]}

#### 2、占位符获取之前配置的值，如果没有可以使用冒号指定默认值

### 5、Profile

Profile是Spring对不同环境提供不同配置功能的支持，可通过激活、指定参数等方式快速切换环境。（比如开发环境和生产环境之间的切换）

#### 1、多profile文件

我们在这配置文件编写的时候，文件名可以是 application-{profile}.properties/yml

默认使用application.properties

#### 2、yml支持多文档块方式

用`---`分割文档块

#### 3、激活指定profile

1. 配置文件中指定 `spring.profiles.active=dev`
2. 命令行参数 `--spring.profiles.active=dev`
3. jar包运行时 同样可通过命令行指定
4. 虚拟机参数 `-Dspring.profiles.active=dev`

### 6、配置文件加载位置

spring boot启动会扫描一下位置的application.properties or application.yml文件作为Spring Boot的默认配置文件

* file: ./config/
* file: ./
* classpath:/config
* classpath:/

以上按照**优先级从高到低顺序**，所有位置文件都会被加载，**高优先级覆盖低优先级**，Spring Boot会加载四个位置全部的配置文件，并形成互补配置

也可以要通过配置spring.config.location改变默认配置，项目打包好以后，可以使用命令行参数的形式，启动项目的时候来指定配置文件的新位置；指定配置文件和默认加载的配置文件一起生效

### 7、外部配置加载顺序

懒得写了，好多啊

### 8、自动配置原理

配置文件配置属性参照[官方文档](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/reference/html/common-application-properties.html)

**自动配置原理**

1. Spring Boot启动的时候加载主配置类，开启了自动配置功能@EnableAutoConfiguration

2. @EnableAutoConfiguration作用 ：

   1. 利用`AutoConfigurationImportSelector`给容器中导入组件
   2. 查看`selectImports()`，有一个`List<String> configurations = this.getCandidateConfigurations(annotationMetadata, attributes);`获取候选的配置
   3. 查看`getCandidateConfigurations()`有一个`SpringFactoriesLoader.loadFactoryNames()`发现时从类路径下得到一个资源`classLoader.getResources("META-INF/spring.factories")`
   4. 扫描所有jar包类路径下 `META-INF/spring.factories`，把扫描到的文件的内容包装成properties对象
   5. 从properties中获取到`EnableAutoConfiguration.class`类（类名）对应的值，然后把他们添加在容器中 （**将类路径下`META-INF/spring.factories`里面配置的所有`EnableAutoConfiguration.class`的值加入到了容器中**）

   ```properties
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
   org.springframework.boot.autoconfigure.data.mongo.MongoDataAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.mongo.MongoReactiveDataAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.mongo.MongoReactiveRepositoriesAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.mongo.MongoRepositoriesAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.neo4j.Neo4jDataAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.neo4j.Neo4jRepositoriesAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.solr.SolrRepositoriesAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.redis.RedisAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.redis.RedisReactiveAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.redis.RedisRepositoriesAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.rest.RepositoryRestMvcAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.web.SpringDataWebAutoConfiguration,\
   org.springframework.boot.autoconfigure.elasticsearch.jest.JestAutoConfiguration,\
   org.springframework.boot.autoconfigure.elasticsearch.rest.RestClientAutoConfiguration,\
   org.springframework.boot.autoconfigure.flyway.FlywayAutoConfiguration,\
   org.springframework.boot.autoconfigure.freemarker.FreeMarkerAutoConfiguration,\
   org.springframework.boot.autoconfigure.gson.GsonAutoConfiguration,\
   org.springframework.boot.autoconfigure.h2.H2ConsoleAutoConfiguration,\
   org.springframework.boot.autoconfigure.hateoas.HypermediaAutoConfiguration,\
   org.springframework.boot.autoconfigure.hazelcast.HazelcastAutoConfiguration,\
   org.springframework.boot.autoconfigure.hazelcast.HazelcastJpaDependencyAutoConfiguration,\
   org.springframework.boot.autoconfigure.http.HttpMessageConvertersAutoConfiguration,\
   org.springframework.boot.autoconfigure.http.codec.CodecsAutoConfiguration,\
   org.springframework.boot.autoconfigure.influx.InfluxDbAutoConfiguration,\
   org.springframework.boot.autoconfigure.info.ProjectInfoAutoConfiguration,\
   org.springframework.boot.autoconfigure.integration.IntegrationAutoConfiguration,\
   org.springframework.boot.autoconfigure.jackson.JacksonAutoConfiguration,\
   org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration,\
   org.springframework.boot.autoconfigure.jdbc.JdbcTemplateAutoConfiguration,\
   org.springframework.boot.autoconfigure.jdbc.JndiDataSourceAutoConfiguration,\
   org.springframework.boot.autoconfigure.jdbc.XADataSourceAutoConfiguration,\
   org.springframework.boot.autoconfigure.jdbc.DataSourceTransactionManagerAutoConfiguration,\
   org.springframework.boot.autoconfigure.jms.JmsAutoConfiguration,\
   org.springframework.boot.autoconfigure.jmx.JmxAutoConfiguration,\
   org.springframework.boot.autoconfigure.jms.JndiConnectionFactoryAutoConfiguration,\
   org.springframework.boot.autoconfigure.jms.activemq.ActiveMQAutoConfiguration,\
   org.springframework.boot.autoconfigure.jms.artemis.ArtemisAutoConfiguration,\
   org.springframework.boot.autoconfigure.groovy.template.GroovyTemplateAutoConfiguration,\
   org.springframework.boot.autoconfigure.jersey.JerseyAutoConfiguration,\
   org.springframework.boot.autoconfigure.jooq.JooqAutoConfiguration,\
   org.springframework.boot.autoconfigure.jsonb.JsonbAutoConfiguration,\
   org.springframework.boot.autoconfigure.kafka.KafkaAutoConfiguration,\
   org.springframework.boot.autoconfigure.ldap.embedded.EmbeddedLdapAutoConfiguration,\
   org.springframework.boot.autoconfigure.ldap.LdapAutoConfiguration,\
   org.springframework.boot.autoconfigure.liquibase.LiquibaseAutoConfiguration,\
   org.springframework.boot.autoconfigure.mail.MailSenderAutoConfiguration,\
   org.springframework.boot.autoconfigure.mail.MailSenderValidatorAutoConfiguration,\
   org.springframework.boot.autoconfigure.mongo.embedded.EmbeddedMongoAutoConfiguration,\
   org.springframework.boot.autoconfigure.mongo.MongoAutoConfiguration,\
   org.springframework.boot.autoconfigure.mongo.MongoReactiveAutoConfiguration,\
   org.springframework.boot.autoconfigure.mustache.MustacheAutoConfiguration,\
   org.springframework.boot.autoconfigure.orm.jpa.HibernateJpaAutoConfiguration,\
   org.springframework.boot.autoconfigure.quartz.QuartzAutoConfiguration,\
   org.springframework.boot.autoconfigure.reactor.core.ReactorCoreAutoConfiguration,\
   org.springframework.boot.autoconfigure.security.servlet.SecurityAutoConfiguration,\
   org.springframework.boot.autoconfigure.security.servlet.SecurityRequestMatcherProviderAutoConfiguration,\
   org.springframework.boot.autoconfigure.security.servlet.UserDetailsServiceAutoConfiguration,\
   org.springframework.boot.autoconfigure.security.servlet.SecurityFilterAutoConfiguration,\
   org.springframework.boot.autoconfigure.security.reactive.ReactiveSecurityAutoConfiguration,\
   org.springframework.boot.autoconfigure.security.reactive.ReactiveUserDetailsServiceAutoConfiguration,\
   org.springframework.boot.autoconfigure.sendgrid.SendGridAutoConfiguration,\
   org.springframework.boot.autoconfigure.session.SessionAutoConfiguration,\
   org.springframework.boot.autoconfigure.security.oauth2.client.servlet.OAuth2ClientAutoConfiguration,\
   org.springframework.boot.autoconfigure.security.oauth2.client.reactive.ReactiveOAuth2ClientAutoConfiguration,\
   org.springframework.boot.autoconfigure.security.oauth2.resource.servlet.OAuth2ResourceServerAutoConfiguration,\
   org.springframework.boot.autoconfigure.security.oauth2.resource.reactive.ReactiveOAuth2ResourceServerAutoConfiguration,\
   org.springframework.boot.autoconfigure.solr.SolrAutoConfiguration,\
   org.springframework.boot.autoconfigure.task.TaskExecutionAutoConfiguration,\
   org.springframework.boot.autoconfigure.task.TaskSchedulingAutoConfiguration,\
   org.springframework.boot.autoconfigure.thymeleaf.ThymeleafAutoConfiguration,\
   org.springframework.boot.autoconfigure.transaction.TransactionAutoConfiguration,\
   org.springframework.boot.autoconfigure.transaction.jta.JtaAutoConfiguration,\
   org.springframework.boot.autoconfigure.validation.ValidationAutoConfiguration,\
   org.springframework.boot.autoconfigure.web.client.RestTemplateAutoConfiguration,\
   org.springframework.boot.autoconfigure.web.embedded.EmbeddedWebServerFactoryCustomizerAutoConfiguration,\
   org.springframework.boot.autoconfigure.web.reactive.HttpHandlerAutoConfiguration,\
   org.springframework.boot.autoconfigure.web.reactive.ReactiveWebServerFactoryAutoConfiguration,\
   org.springframework.boot.autoconfigure.web.reactive.WebFluxAutoConfiguration,\
   org.springframework.boot.autoconfigure.web.reactive.error.ErrorWebFluxAutoConfiguration,\
   org.springframework.boot.autoconfigure.web.reactive.function.client.ClientHttpConnectorAutoConfiguration,\
   org.springframework.boot.autoconfigure.web.reactive.function.client.WebClientAutoConfiguration,\
   org.springframework.boot.autoconfigure.web.servlet.DispatcherServletAutoConfiguration,\
   org.springframework.boot.autoconfigure.web.servlet.ServletWebServerFactoryAutoConfiguration,\
   org.springframework.boot.autoconfigure.web.servlet.error.ErrorMvcAutoConfiguration,\
   org.springframework.boot.autoconfigure.web.servlet.HttpEncodingAutoConfiguration,\
   org.springframework.boot.autoconfigure.web.servlet.MultipartAutoConfiguration,\
   org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration,\
   org.springframework.boot.autoconfigure.websocket.reactive.WebSocketReactiveAutoConfiguration,\
   org.springframework.boot.autoconfigure.websocket.servlet.WebSocketServletAutoConfiguration,\
   org.springframework.boot.autoconfigure.websocket.servlet.WebSocketMessagingAutoConfiguration,\
   org.springframework.boot.autoconfigure.webservices.WebServicesAutoConfiguration,\
   org.springframework.boot.autoconfigure.webservices.client.WebServiceTemplateAutoConfiguration
   ```

   每一个这样的xxxAutoConfiguration类都是容器中的一个组件，都加入到容器中，用他们来做自动配置；

3. 每一个自动配置类进行自动配置功能；以`HttpEncodingAutoConfiguration`类为例：

   ```java
   @Configuration //表示这是一个配置类，和以前写的配置文件一样，也可以给容器中添加组件
   @EnableConfigurationProperties({HttpProperties.class}) //启动指定类的ConfigurationProperties功能；将配置文件中的值和HttpProperties.class绑定起来，并把HttpProperties.class加入到ioc容器中
   @ConditionalOnWebApplication(
       type = Type.SERVLET
   ) //Spring底层注解@Conditional，根据不同的条件，如果满足指定的条件，整个配置类里面的配置就会生效（这里判断当前应用是不是web应用
   @ConditionalOnClass({CharacterEncodingFilter.class}) //判断当前项目有没有这个类：CharacterEncodingFilter.class SpringMVC中进行乱码解决的过滤器
   @ConditionalOnProperty(
       prefix = "spring.http.encoding",
       value = {"enabled"},
       matchIfMissing = true
   ) //判断配置文件中是否存在某个配置（这里是spring.http.encoding.enabled）,如果不存在判断也成立的（matchIfMissing）。结果：即使我们不配置spring.http.encoding.enabled=true也是默认生效的
   public class HttpEncodingAutoConfiguration {
       
       // 这个已经和Spring Boot中的配置文件映射了
       private final Encoding properties;
       
       //只有一个有参构造器的情况下，参数的值就会从容器中拿
       public HttpEncodingAutoConfiguration(HttpProperties properties) {
           this.properties = properties.getEncoding();
       }
       
       @Bean //给容器中添加一个组件，这个组件的某些值需要从properties中获取
       @ConditionalOnMissingBean
       public CharacterEncodingFilter characterEncodingFilter() {
           CharacterEncodingFilter filter = new OrderedCharacterEncodingFilter();
           filter.setEncoding(this.properties.getCharset().name());
                  filter.setForceRequestEncoding(this.properties.shouldForce(org.springframework.boot.autoconfigure.http.HttpProperties.Encoding.Type.REQUEST));
           filter.setForceResponseEncoding(this.properties.shouldForce(org.springframework.boot.autoconfigure.http.HttpProperties.Encoding.Type.RESPONSE));
           return filter;
       }
   }
   
   /* 总结：根据当前不同的条件，决定这个配置类是否生效。
   	一旦这个配置类生效，这个配置类就会给容器中添加各种组件；这些组件的属性是从对应的properties类中获取的，而这些类里面的每一个属性又是和配置文件绑定的 */
   
   /* 最终总结：properties文件 --> properties类 --> 配置类操作组件（自动），组件从类中获取属性 --> 组件被添加到ioc --> ioc给应用提供服务  */
   
   ```

   所有在配置文件中能配置的属性都是在xxxProperties类中封装过的；配置文件能配置什么东西即可参照这个属性类

   ```java
   @ConfigurationProperties(
       prefix = "spring.http"
   ) //从配置文件中获取指定的值和Bean的属性进行绑定
   public class HttpProperties {} //这是一个Bean类，里面有啥 你就能配啥
   ```


**精髓：**

 	1. **SpringBoot启动会加载大量的自动配置类**
 	2. **我们看我们需要的功能有没有Spring Boot写好的自动配置类**
 	3. **我们再来看自动配置类中配置了哪些组件，只要我们要用的组件有，就不需要再来配置了**
 	4. **给容器中自动配置类添加组件的时候，会从properties类中获取某些属性。因此我们就可以在配置文件中指定这些属性的值**

xxxAutoConfiguration：自动配置类；给容器中添加组件

xxxProperties：封装配置文件中相关属性；

细节：

  @Conditional注解会给配置类加许多生效条件，**自动配置类只在满足条件的情况下才生效**

  我们怎么知道哪些自动配置类生效了？开启Spring Boot的debug模式（`debug=true`），让控制台打印自动配置报告	

## 二、SpringBoot与日志

### 1、SLF4j（一种日志框架的抽象层）使用

#### 1、如何在系统中使用SLF4j

日志记录方法的掉用，不应该来直接调用日志的实现类，而是调用日志抽象层里面的方法。

给系统导入slf4j的jar和logback的实现jar 

```java
/**
 * Example
 */
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class HelloWorld {
  public static void main(String[] args) {
    Logger logger = LoggerFactory.getLogger(HelloWorld.class);
    logger.info("Hello World");
  }
}
```

每一个日志的实现框架都有自己的配置文件，**使用slf4j后，配置文件还是做成日志实现框架的配置文件**；

#### 2、解决遗留问题

排除Spring中的其他日志实现，使用中间包替换之，并使用slf4j的实现

**如果我们引入了其他框架，一定要把这个框架的默认日志框架移除掉**

### 2、日志使用

```java
//记录器
Logger logger = LoggerFactory.getLogger(getClass());

@Test
public void contextLoads() {

    //日志的级别，由低到高
    //便于调整日志级别，使只显示特定级别以上的信息
    logger.trace("这是trace日志");
    logger.debug("这是debug日志");
    //SpringBoot默认使用info级别，可给指定包指定级别
    logger.info("这是info日志");
    logger.warn("这是warn日志");
    logger.error("这是error日志");

}
```

修改默认配置示例

```properties
logging.level.com.vwmin = trace

# 在当前磁盘（注意不是当前文件夹）的根路径下创建spring文件夹和log文件夹，使用spring.log作为默认日志文件
logging.path=/spring/log

# 在控制台输出的日志的格式
#logging.pattern.console=

# 文件中日志输出的格式
#logging.pattern.file=

# 在指定路径下生成日志文件
#logging.file=springboot.log
```

指定配置文件，resources下放logback.xml(其他框架放各自的配置文件)

logback.xml会直接被logback识别，改为logback-spring.xml可使用SpringBoot提供的高级功能（使配置只在指定环境下生效）

```xml
<springProfile name="dev">
	your configuration.
</springProfile>
```

## 三、SpringBoot与WEB

### 1、SpringBoot对静态资源的映射规则

```java
@ConfigurationProperties(
    prefix = "spring.resources",
    ignoreUnknownFields = false
)
public class ResourceProperties {}
// 可以设置和静态资源有关的参数，比如缓存时间等
```

```java
//from WebMvcAutoConfiguration.class
public void addResourceHandlers(ResourceHandlerRegistry registry) {
    if (!this.resourceProperties.isAddMappings()) {
        logger.debug("Default resource handling disabled");
    } else {
        Duration cachePeriod = this.resourceProperties.getCache().getPeriod();
        CacheControl cacheControl = this.resourceProperties.getCache().getCachecontrol().toHttpCacheControl();
        if (!registry.hasMappingForPattern("/webjars/**")) {
            this.customizeResourceHandlerRegistration(registry.addResourceHandler(new String[]{"/webjars/**"}).addResourceLocations(new String[]{"classpath:/META-INF/resources/webjars/"}).setCachePeriod(this.getSeconds(cachePeriod)).setCacheControl(cacheControl));
        }

        String staticPathPattern = this.mvcProperties.getStaticPathPattern();
        if (!registry.hasMappingForPattern(staticPathPattern)) {
            this.customizeResourceHandlerRegistration(registry.addResourceHandler(new String[]{staticPathPattern}).addResourceLocations(getResourceLocations(this.resourceProperties.getStaticLocations())).setCachePeriod(this.getSeconds(cachePeriod)).setCacheControl(cacheControl));
        }

    }
}

//欢迎页映射； HandlerMapping保存每一个请求谁来处理
@Bean
public WelcomePageHandlerMapping welcomePageHandlerMapping(ApplicationContext applicationContext){
    return new WelcomePageHandlerMapping(new TemplateAvailabilityProviders(applicationContext), applicationContext, this.getWelcomePage(), this.mvcProperties.getStaticPathPattern());
}

// 图标哒哟
	  @Configuration
        @ConditionalOnProperty(
            value = {"spring.mvc.favicon.enabled"},
            matchIfMissing = true
        )
        public static class FaviconConfiguration implements ResourceLoaderAware {
            private final ResourceProperties resourceProperties;
            private ResourceLoader resourceLoader;

            public FaviconConfiguration(ResourceProperties resourceProperties) {
                this.resourceProperties = resourceProperties;
            }

            public void setResourceLoader(ResourceLoader resourceLoader) {
                this.resourceLoader = resourceLoader;
            }

            @Bean
            public SimpleUrlHandlerMapping faviconHandlerMapping() {
                SimpleUrlHandlerMapping mapping = new SimpleUrlHandlerMapping();
                mapping.setOrder(-2147483647);
                mapping.setUrlMap(Collections.singletonMap("**/favicon.ico", this.faviconRequestHandler()));
                return mapping;
            }

            @Bean
            public ResourceHttpRequestHandler faviconRequestHandler() {
                ResourceHttpRequestHandler requestHandler = new ResourceHttpRequestHandler();
                requestHandler.setLocations(this.resolveFaviconLocations());
                return requestHandler;
            }

            private List<Resource> resolveFaviconLocations() {
                String[] staticLocations = WebMvcAutoConfiguration.WebMvcAutoConfigurationAdapter.getResourceLocations(this.resourceProperties.getStaticLocations());
                List<Resource> locations = new ArrayList(staticLocations.length + 1);
                Stream var10000 = Arrays.stream(staticLocations);
                ResourceLoader var10001 = this.resourceLoader;
                this.resourceLoader.getClass();
                var10000.map(var10001::getResource).forEach(locations::add);
                locations.add(new ClassPathResource("/"));
                return Collections.unmodifiableList(locations);
            }
        }
```

1. 所有`/webjars/**`，都去`classpath:/META-INF/resources/webjars/`找资源。

   [webjars](https://www.webjars.org/)：以jar包的形式引入静态资源

   在pox.xml导入

   [点点看？](http://localhost:8080/webjars/jquery/3.3.1/jquery.js)

   ```xml
   <dependency><!--Jquery组件（前端）-->
         <groupId>org.webjars</groupId>
         <artifactId>jquery</artifactId>
         <version>3.3.1</version>
   </dependency>
   ```

2. “/**”访问当前项目的任何资源（静态资源的文件夹 

   ```java
   "classpath:/META-INF/resources/"
   "classpath:/resources/"
   "classpath:/static/"
   "classpath:/public/"
    "/"
   ```

   localhost:8080/xxx 等价于去以上路径查找这个静态资源

3. 欢迎页：静态资源文件夹下的所有`index.html`页面；被 `/**`映射

   localhost:8080/ 等价于进入index.html页面

4. 所有的 `**/favicon.ico`都是在静态资源文件夹找

5. 自定义静态资源路径
   ```properties
   spring.resources.static-locations=classpath:/hello/, ...
   ```

### 2、模板引擎

JSP、Velocity、Freemarker、Thymeleaf

#### 1、引入thymeleaf

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

#### 2、thymeleaf使用和语法

```java
@ConfigurationProperties(
    prefix = "spring.thymeleaf"
)
public class ThymeleafProperties {
    private static final Charset DEFAULT_ENCODING;
    //按照这样的格式放置文件thymeleaf就能自动渲染
    public static final String DEFAULT_PREFIX = "classpath:/templates/";
    public static final String DEFAULT_SUFFIX = ".html";
}
```

1. 导入语法提示（thymeleaf名称空间）`<html lang="en" xmlns:th="http://www.thymeleaf.org">`

2. 使用thymeleaf语法 `th:任意属性`来替换原生HTML属性

   ![语法]((/CXM(U7`315[FXIXS[TZF46.png)

3. ```properties
   Simple expressions:
       Variable Expressions: ${...} :获取变量值：OGNL表达式
       	#1、获取对象的属性、调用方法
       	#2、使用内置的基本对象 看文档把
       	#3、内置的一些工具对象
       Selection Variable Expressions: *{...} :选择表达式，和${...}在功能上相同
       	#补充配合th:object使用
       	 <div th:object="${session.user}">
               <p>Name: <span th:text="*{firstName}">Sebastian</span>.</p>
               <p>Surname: <span th:text="*{lastName}">Pepper</span>.</p>
               <p>Nationality: <span th:text="*{nationality}">Saturn</span>.</p>
             </div>
       Message Expressions: #{...} :获取国际化内容
       Link URL Expressions: @{...} :定义URl
       	#eg: @{/order/process(execId=${execId},execType='FAST')}
       Fragment Expressions: ~{...} :片段引用表达式
   ```

### 3、使用Spring MVC

自动配置内容

####  eg：Spring MVC Auto-configuration

Spring Boot provides auto-configuration for Spring MVC that works well with most applications.

The auto-configuration adds the following features on top of Spring’s defaults:

- Inclusion of `ContentNegotiatingViewResolver` and `BeanNameViewResolver` beans. 
  - 自动配置了ViewResolver：视图解析器：根据方法得返回值得到试图对象，试图对象据欸的那个如何渲染（转发？重定向？
  - `ContentNegotiatingViewResolver`组合所有视图解析器的视图解析器
  - 如何定制：我们可以自己给容器中添加一个视图解析器， 上面这个类就会自动的将其组合进来，效果如下
  - ![我是效果]($6%@96}[(/_IF66%OMV$~F4N.png)
- Support for serving static resources, including support for WebJars (covered [later in this document](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/reference/html/boot-features-developing-web-applications.html#boot-features-spring-mvc-static-content))).（静态资源文件夹）
- Automatic registration of `Converter`, `GenericConverter`, and `Formatter` beans.
  - `Converter` 转换器：将网页提交？的类型进行转换
  - `Formatter` 格式化器：比如按照地区 转化时间格式
  - 结论：自己添加的格式化器、转换器只需要放进容器中即可
- Support for `HttpMessageConverters` (covered [later in this document](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/reference/html/boot-features-developing-web-applications.html#boot-features-spring-mvc-message-converters)).
  - `HttpMessageConverter`SpringMVC用来转换http请求和响应的
  - `HttpMessageConverters`是从容器中确定、获取所有的`HttpMessageConverter`
  - 结论：自定义的这啥只需要注册进容器中即可工作（@Bean，@Component
- Automatic registration of `MessageCodesResolver` (covered [later in this document](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/reference/html/boot-features-developing-web-applications.html#boot-features-spring-message-codes)).（定义错误代码生成规则）
- Static `index.html` support.（静态首页访问）
- Custom `Favicon` support (covered [later in this document](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/reference/html/boot-features-developing-web-applications.html#boot-features-spring-mvc-favicon)).（自定义图标）
- Automatic use of a `ConfigurableWebBindingInitializer` bean (covered [later in this document](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/reference/html/boot-features-developing-web-applications.html#boot-features-spring-mvc-web-binding-initializer)).
  - 也是从容器中尝试取出的，因此也是可自定义的
  - 初始化数据绑定器，将web请求转化为JavaBean，这里还会用到上面提到的组件

**org.springframework.boot.autoconfigure.web: WEB的所有自动场景**

If you want to keep Spring Boot MVC features and you want to add additional [MVC configuration](https://docs.spring.io/spring/docs/5.1.5.RELEASE/spring-framework-reference/web.html#mvc) (interceptors, formatters, view controllers, and other features), you can add your own `@Configuration` class of type `WebMvcConfigurer` but **without** `@EnableWebMvc`. If you wish to provide custom instances of `RequestMappingHandlerMapping`, `RequestMappingHandlerAdapter`, or `ExceptionHandlerExceptionResolver`, you can declare a `WebMvcRegistrationsAdapter` instance to provide such components.

（**扩展SpringMVC：编写一个WebMvcConfigurer类型的配置类(@Configuration)，并且不能标注@EnableWebMvc**

这部分听得好迷，多看看把

If you want to take complete control of Spring MVC, you can add your own `@Configuration` annotated with `@EnableWebMvc`.

### 4、如何修改SpringBoot的默认配置

模式：

       	1. SpringBoot在自动配置很多组件的时候都是先看容器中有没有用户自定义，如果没有才加入默认的；如果有些组件可以有多个，则是将榕湖配置的和自己默认的组合起来。
                        	2. 在SpringBoot中有非常多的xxxConfigurer接口类帮助扩展配置 
                        	3. 也有很多的xxxCustomizer来帮助定制配置

### 5、RestfulCRUD

加载静态资源文件时不要强行把路径写全，比如`../static/img/xxx.jpg`直接写 `img/xxx.jpg`就够了

资源引用时注意相对路径和绝对路径

模板页面修改后要想实时生效

1. 禁用模板引擎的缓存 `spring.thymeleaf.cache=false`
2. 页面修改完成以后crtl + f9

登错错误的消息提示

```html
    <p style="color: red;" th:text="${msg}" th:if="${not #strings.isEmpty(msg)}"></p>
```

放置F5刷新重复提交表单 ---> 重定向

使用拦截器进行登陆检查

CRUD员工列表

1. restfulCRUD：

   URI：/资源名称/资源标识    HTTP请求方式区分对资源CRUD操作

   |      | 普通CRUD（uri区分操纵 | RestfulCRUD       |
   | ---- | --------------------- | ----------------- |
   | 查询 | getEmp                | emp---GET         |
   | 添加 | addEmp?xxx            | emp---POST        |
   | 修改 | updateEmp?xxx=        | emp/{xx}---PUT    |
   | 删除 | deleteEmp?xxx=        | emp/{xx}---DELETE |

2. 实验的请求架构：

   |                                    | 请求URI  | 请求方式 |
   | ---------------------------------- | -------- | -------- |
   | 查询所有员工                       | emps     | GET      |
   | 查询某个员工                       | emp/{id} | GET      |
   | 进入添加页面                       | emp      | GET      |
   | 添加员工                           | emp      | POST     |
   | 来到修改页面（查出员工进行信息回显 | emp/{id} | GET      |
   | 修改员工                           | emp      | PUT      |
   | 删除员工                           | emp/{id} | DELETE   |

3. 员工列表

   thymeleaf公共页面抽取

   ```html
   1. 抽取公共页面
   <div th:fragment="copy">yyy</div>
   
   2. 引入公共片段
   <div th:insert="~{footer  ::  copy}"></div>
   ~{templatename::selector} 模板名::选择器
   ~{templatename::fragmentname} 模板名::片段名
   
   
   3. 默认效果：
   insert的公共片段在div标签中
   ```

   三种引入公共片段的th属性

   * `th:insert`抽取出来的片段会被包含在语句所在的div中
   * `th:replace`抽取出来的片段会整个替换掉语句所在div
   * `th:include`抽取出来的片段会被去掉外层标签然后被包含在div中

4. 员工添加

   格式问题：特别是日期

   日期的格式化：SpringMVC将页面提交的值转化为指定的类型

### 6、错误处理机制

默认处理参照 `ErrorMvcAutoConfiguration` 给容器添加了以下组件：

1. `DefaultErrorAttributes` 帮我们在页面共享信息

2. `BasicErrorController`

   ```java
   @Controller
   @RequestMapping({"${server.error.path:${error.path:/error}}"})
   public class BasicErrorController extends AbstractErrorController {
       //产生HTML页面
       @RequestMapping(produces = {"text/html"})
       public ModelAndView errorHtml(HttpServletRequest request, HttpServletResponse response) {
           HttpStatus status = this.getStatus(request);
           Map<String, Object> model = Collections.unmodifiableMap(this.getErrorAttributes(request, this.isIncludeStackTrace(request, MediaType.TEXT_HTML)));
           response.setStatus(status.value());
           ModelAndView modelAndView = this.resolveErrorView(request, response, status, model);
           return modelAndView != null ? modelAndView : new ModelAndView("error", model);
       }
   
       //产生json数据
       @RequestMapping
       public ResponseEntity<Map<String, Object>> error(HttpServletRequest request) {
           Map<String, Object> body = this.getErrorAttributes(request, this.isIncludeStackTrace(request, MediaType.ALL));
           HttpStatus status = this.getStatus(request);
           return new ResponseEntity(body, status);
       }
   } //处理默认/error请求
   ```

3. `ErrorPageCustomizer`

   ```java
   @Value("${error.path:/error}")
   private String path = "/error"; //系统出现错误后来到error请求进行处理
   ```

4. `DefaultErrorViewResolver`

步骤：

​	一旦系统出现4xx或5xx系列错误，`ErrorPageCustomizer`就会生效（定制错误的响应规则）然后来到 `/error`请求，被`BasicErrorController`处理，根据请求头响应页面或是json    **后面没听进去嘤嘤嘤**

如何定制错误页面

1. 有模板页面时：找 `/error/(status_code).html`
2. 有模板页面，未找到确定状态码：找 `/error/4xx.html` 5xx同理
3. 没有使用thymeleaf，静态资源文件夹下找
4. 页面能获取的信息 `timestamp 时间戳` `status 状态码` `error 错误提示` `exception 异常对象` `message 异常消息` `errors JSR303数据校验的错误`

如何定制JSON错误信息

1. 自定义异常处理并返回定制json数据

   ```java
       //1、捕获并处理用户不存在这个特定的异常，浏览器服务器返回的都是json
   //    @ResponseBody
   //    @ExceptionHandler(UserNotExistException.class)
   //    public Map<String, Object> handleException(Exception e){
   //        Map<String, Object> map = new HashMap<>();
   //        map.put("code", "user.not_exist");
   //        map.put("message", e.getMessage());
   //        return map;
   //    }
   ```

2. 转发到error进行自适应

   ```java
   @ExceptionHandler(UserNotExistException.class)
   public String handleException(Exception e, HttpServletRequest request){
   
       //传入自己定义的错误状态码，否则不能进入自定义错误页面
       request.setAttribute("javax.servlet.error.status_code", 444);
   
       Map<String, Object> map = new HashMap<>();
       map.put("code", "user.not_exist");
       map.put("message", e.getMessage());
       return "forward:/error"; //转发到/error请求
   }
   ```

3. 将定制数据携带出去
   1. 完全来编写一个ErrorController的实现类，或是编写AbstractErrorController的子类，放在容器中 
   2. 页面上能用的数据或者json返回的能用的数据都是由`getErrorAttributes()`得到的（源码中 `ErrorController`规定的方法），容器中`DefaultErrorAttributes.getErrorAttributes()`默认进行数据处理

如果设置了一个自定义的状态码，会因为默认状态码里找不到这个而被统一显示成500，当然你改的确实生效了

总结：在异常handler里面更改状态码，并转发至 `/error`，使页面能够进入到自定义错误页面。重写`DefaultErrorAttributes`的`getAttributes`方法，自定义要显示的信息

### 7、配置嵌入式Servlet容器

SpringBoot使用的使嵌入式的Servlet容器（Tomcat）

问题：

1. 如何定制和修改这个Servlet容器的相关配置？

   1. 在properties中修改和server相关--->通用的Servlet配置，修改和tomcat相关`server.tomcat`

   2. 编写一个EmbeddedServletContainerCustomizer：嵌入式的Servlet容器定制器

   3. 注册Servlet三大组件【servlet、filter、listener】

      注册三大组件以以下方式

      * `ServletRegistrationBean`
      * `FilterRegistrationBean`
      * `ServletListenerRegistrationBean`

2. 能不能支持其他的Servlet？能，嗯。

**Servlet的内容从P48开始跳过了，特记**

### 8、一些注解的解释

`@RequestParam`和 `@PathVariable`

```java
在spring MVC中，两者的作用都是将request里的参数的值绑定到contorl里的方法参数里的，区别在于，URL写法不同。

使用@RequestParam时，URL是这样的：http://host:port/path?参数名=参数值
使用@PathVariable时，URL是这样的：http://host:port/path/参数值
```

`@RequestParam`、`@RequestBody`和`@ModelAttribute`

[头皮发麻](https://www.cnblogs.com/zeroingToOne/p/8992746.html)

## 四、SpringBoot与Docker

### 1、简介

Docker是一个开源的应用容器引擎；就是将你已经安装配置好的软件边一成一个镜像，然后镜像发布出去就能够直接使用，免去配置。

### 2、核心概念

* docker主机（Host）：安装了Docker程序的机器，docker是直接安装在操作系统之上的
* docker客户端（client）：链接docker主机进行操作
* docker仓库（Registy）：用来保存各种打包好的软件镜像
* docker镜像（images）：软件打包好的镜像，放在docker仓库中
* docker容器（Container）：镜像启动后的示例成为一个容器；容器是独立运行的一个或一组应用程序

使用docker的步骤：

1. 安装docker
2. 去docker仓库找到这个软件对应的镜像
3. 使用docker运行这个镜像，这个镜像就会生成一个docker容器
4. 对容器的启动停止就是对软件的启动停止

docker常操：

1. `docker search xxx`
2. `docker pull xxx`  
3. `docker images` 查看所有本地镜像
4. `docker rmi image-id` 删除指定的本地镜像

容器常操

| 操作     | 命令                                      | 说明                                           |
| -------- | ----------------------------------------- | ---------------------------------------------- |
| 运行     | `docker run --name 自定义命名 -d 镜像名`  | `--name`：自定义容器名；`-d`后台运行           |
| 列表     | `docker ps`                               | 查看运行中容器；加上`-a`可查看所有容器         |
| 停止     | `docker stop 自定义命名/container-id`     | 启动同理                                       |
| 删除     | `docker rm container-id`                  | 只接受容器id？                                 |
| 端口映射 | `-p 8080:8080`                            | 添加在运行命令，将机器的端口映射到docker的端口 |
| 容器日志 | `docker logs container-name/container-id` | 嗯， 啊。                                      |

一个镜像可以启动成多个容器

### 3、mysql

运行 `docker run --name mymysql -d mysql`结果运行失败，查看日志，提示需要指定密码项

查看使用文档，指定密码参数 `docker run --name mymysql -e MYSQL_ROOT_PASSWORD=justmiss -d mysql`

依然失败，需要端口映射

其他操作：

```shell
docker run --name mymysql -v /自定义路径/:/etc/mysql/conf.d -e MYSQL_ROOT_PASSWORD=justmiss -d mysql #把主机的指定文件夹挂载到docker容器的那个文件夹里面，就是说可以通过主机文件夹下的配置文件影响mysql容器的配置

#还可不指定配置文件，yong--来额外指定一些参数
```

## 五、SpringBoot与数据访问

### 1、JDBC

访问配置

```yml
spring:
  datasource:
    username: root
    password: justmiss
    url: jdbc:mysql://159.65.73.6:3306/JDBC
    driver-class-name: com.mysql.cj.jdbc.Driver
```

默认使用Hikari作为数据源；

操作数据库：自动配置了jdbcTemplate操作数据库

### 2、引入Druid数据源

```yml
spring:
  datasource:
    username: root
    password: justmiss
    url: jdbc:mysql://159.65.73.6:3306/JDBC
    driver-class-name: com.mysql.cj.jdbc.Driver
    type: com.alibaba.druid.pool.DruidDataSource #使用了druid数据源
#    schema:
#      - classpath:department.sql
#    initialization-mode: always

    #其他配置，不能映射到默认的配置文件，需要创建设置类
    initialSize: 5
    minIdle: 5
    maxActive: 20
    maxWait: 60000
    timeBetweenEvictionRunsMillis: 60000
    minEvictableIdleTimeMillis: 300000
    validationQuery: SELECT 1 FROM DUAL
    testWhileIdle: true
    testOnBorrow: false
    testOnReturn: false
    poolPreparedStatement: true
    #配置监控统计拦截的filters，去点后监控界面sql无法统计，wall用于防火墙
    filters: stat,wall
    maxPoolPreparedStatementPerConnectionSize: 20
    useGlobalDataSourceStat: true
    connectionProperties: druid.stat.mergeSql=true;druid.stat.slowSqlMillis=500
```

设置Servlet和filter

```java
@Configuration
public class DruidConfig {
    
    @Bean
    @ConfigurationProperties(prefix = "spring.datasource") //将这些配置绑定进来
    public DataSource druid(){
        return new DruidDataSource();
    }

    //配置Druid的监控
    //1、配置一个管理后台的Servlet
    @Bean
    public ServletRegistrationBean statViewServlet(){
        //处理druid下的所有请求
        ServletRegistrationBean bean =  new ServletRegistrationBean(new StatViewServlet(), "/druid/*");
        Map<String, String> initParams = new HashMap<>();

        initParams.put("loginUsername", "admin"); //可配置参数详见ResourceServlet
        initParams.put("loginPassword", "justmiss");
//        initParams.put("allow", ""); //允许哪些访问
//        initParams.put("deny", ""); //拒绝哪些访问
        initParams.put("resetEnable", "false");
        bean.setInitParameters(initParams);
        return bean;
    }


    //2、配置一个web监控的filter
    @Bean
    public FilterRegistrationBean webStatFilter(){
        FilterRegistrationBean bean = new FilterRegistrationBean(new WebStatFilter());

        Map<String, String> initParams = new HashMap<>();   //可用参数详见WebStatFilter
        initParams.put("exclusions", "*.js, *.css, /druid/*"); //忽略各种资源文件的访问
        bean.setInitParameters(initParams);
        bean.addUrlPatterns("/*"); //设置过滤器过滤路径
        return bean;
    }
}
```

### 3、整合MyBatis

使用 `@Mapper`注解将接口映射到表来操作

```java
//指定这是一个操作数据库的Mapper
@Mapper
public interface DepartmentMapper {


    @Select("select * from department where dept_id=#{dept_id}")
    public Department getDeptById(Integer dept_id);

    @Delete("delete * from department where dept_id=#{dept_id}")
    public int deleteDeptById(Integer dept_id);

    @Options(useGeneratedKeys = true, keyProperty = "dept_id") //使用自动生成的主键，需要注明主键
    @Insert("insert into department(dept_name) values(#{dept_name})")
    public int insertDept(Department department);

    @Update("update department set dept_name=#{dept_name} where dept_id=#{dept_id}")
    public int updateDeptById(Department department);

}

```

不写配置文件的情况下配置MyBatis属性：给容器中添加ConfigurationCustomizer

```java
@org.springframework.context.annotation.Configuration
public class MyBatisConfig {
  
    @Bean
    public ConfigurationCustomizer configurationCustomizer(){
        return new ConfigurationCustomizer(){

            @Override
            public void customize(Configuration configuration) {
                // 开启驼峰命名法
                configuration.setMapUnderscoreToCamelCase(true);
            }
        };
    }
}
```

`@MapperScan(value = "com.vwmin.springboot.mapper")`扫描整个包，所有接口都作为Mapper

### 4、SpringData JPA（JAVA 持久化API

JPA：ORM（Object Relational Mapping）

1. 编写一个实体类（Bean）和数据表进行映射，并且配置好映射关系

   ```java
   //使用JPA注解配置映射关系
   @Entity //告诉JPA是一个实体类，即和数据表映射的类
   @Table(name = "user") //来指定和哪个数据表对应，如果省略，默认表明就是类名小写
   public class User {
   
       @Id //标明主键
       @GeneratedValue(strategy = GenerationType.IDENTITY) //设置自增组件
       private Integer id;
       @Column //数据表对应的列（name（自定义列名）= xxx, length = xxx）
       private String lastName;
       @Column //省略的话默认列名就是属性名
       private String email;
       
       /*setter and getter*/
   }
   ```

2. 编写一个Dao接口来操作实体类对应的数据表（Repository）

   ```java
   /**
    * 继承对JpaRepository来完成对数据库的操作
    * 第一个参数接收要操作的数据类型，第二个接收主键类型
    */
   public interface UserRepository extends JpaRepository<User, Integer> {
   
   
   }
   ```

3. 基本的配置

   ```yml
   jpa:
     hibernate:
       ddl-auto: update #定义数据表的生成策略，更新或者创建数据表
     show-sql: true #在控制台显示sql
   ```



### 5、JDBC与JPA

   [区别](https://blog.csdn.net/AAAAABBBBBYYYYY/article/details/76422695)

## 六、SpringBoot启动配置原理

几个重要的事件回调机制

`ApplicationContextInitializer`

`SpringApplicationRunListener`

`ApplicationRunner`

`CommandLineRunner`



启动流程：

1. 创建SpringApplication对象
   1. 调用initialize方法创建对象
2. 运行run方法

**溜了溜了**

## 七、SpringBoot与缓存

### 1、JSR107



### 2、Spring缓存抽象

| Cache          | 缓存接口，定义缓存操作。实现有：RedisCache, EhCacheCache, etc. |
| -------------- | ------------------------------------------------------------ |
| CacheManager   | 缓存管理器，管理各种缓存(Cache)组件                          |
| @Cacheable     | 主要针对方法配置，能够根据方法的请求参数对其结果进行缓存     |
| @CacheEvict    | 清空缓存                                                     |
| @CachePut      | 保证方法配调用，右希望结果被缓存（更新缓存                   |
| @EnableCaching | 开启基于注解的缓存                                           |
| keyGenerator   | 缓存数据时生成key的策略                                      |
| serialize      | 缓存数据时value序列化策略                                    |

简单体验（默认使用 `ConcurrentMapCacheManager` ）：

```java
@Service
public class EmployeeService {
    @Autowired
    EmployeeMapper employeeMapper;

    Logger logger = LoggerFactory.getLogger(getClass());


    /**
     * 将方法的运行结果进行缓存
     * 几个属性：
     *      cacheName/value: 指定缓存的名字，唯一的，是缓存管理组件管理的标记
     *      key: 缓存数据使用的key，用它来指定(可使用SpEL)，默认使用方法参数来指定
     *      keyGenerator: key的生成器，可以自己指定key的生成器的组件id。与上条仅存其一
     *      cacheManager: 指定缓存管理器。
     *      cacheResolver: 指定缓存解析器，与上条仅存其一
     *      condition: 指定符合条件的情况下才缓存   
     *      unless: 满足条件时不缓存，可以取结果使用
     *      sync: 是否使用异步，使用异步时unless不可使用
     *
     */
    @Cacheable(cacheNames = "emp", key = "#emp_id", condition = "#emp_id>0", unless = "#result == null")
    public Employee getEmp(Integer emp_id){
        logger.info("查询"+emp_id+"号员工");
        return employeeMapper.getEmpById(emp_id);
    }

}
```

运行流程：

1. 方法运行之前，CacheManager按照cacheName获取Cache（缓存组件），第一次获取缓存会自动创建这个Cache
2.  去Cache中按照key查找缓存
3. 没有查到缓存就调用目标方法，将方法返回的结果加入缓存

key(SimplekeyGenerator)默认的生成策略：

* 没有参数：`key = new SimpleKey()` 
* 有一个参数：`key = args[0]`
* 有多个参数：`key = new SimpleKey(args)`

自定义KeyGenerator

```java
/**
 * 自定义KeyGenerator
 */
@Configuration
public class MyCacheConfig {

    @Bean("myKeyGenerator")
    public KeyGenerator keyGenerator(){
        return new KeyGenerator(){
            @Override
            public Object generate(Object traget, Method method, Object... args) {
                return method.getName()+"["+args[0].toString()+"]";
            }
        };
    }
}
```

更新数据并更新缓存（注意保持key不变）：

```java
@CachePut(cacheNames = "emp", key = "#result.emp_id")
public Employee update(Employee employee){
    logger.info("更新"+employee.getEmp_id()+"号员工");
    employeeMapper.updateEmpById(employee);
    return employee;
}
```

删除缓存：

```java
/**
     * allEntries = true: 删干净
     */
    @CacheEvict(cacheNames = "emp", key = "#emp_id")
    public void deleteEmp(Integer emp_id){
        logger.info("删除"+emp_id+"号员工");
//        employeeMapper.deleteEmpById(emp_id);
    }
```

复杂缓存规则 

```java
@Caching(
        cacheable = {
                @Cacheable(value = "emp", key = "#emp_lastName")
        },
        put = {
                @CachePut(value = "emp", key = "#result.dept_id"),
                @CachePut(value = "emp", key = "#result.emp_email")
        }
)
public Employee getEmpByLastName(String emp_lastName){
    logger.info("查询名为"+emp_lastName+"的员工");
    return employeeMapper.getEmpByLastName(emp_lastName);
}
```

使用类名注解 `@CacheConfig` 简化配置

### 3、整合Redis

引入redis后，优先配置了`RedisCacheManager` 并创建 `RedisCache`作为缓存组件(自动)

操作类：

* `RedisTemplate`键值对`<Object, Object>`
* `StringRedisTemplate`操作字符串 `<String, String>`

常见数据类型

- String：`StringRedisTemplate.opsForValue()`
- list: `StringRedisTemplate.opsForList()`
- set: `StringRedisTemplate.opsForSet()`
- hash: `StringRedisTemplate.opsForHash()`
- ZSet（有序集合: `StringRedisTemplate.opsForZSet()`

关于保存对象：

1. `RedisManager`使用`RedisTemplate<Object, Object>`进行缓存，对象需要序列化，`RedisTemplate<Object, Object>`默认使用的是jdk的序列化器
2. 自定义`RedisCacheManager`  

## 八、SpringBoot与消息

### 1、应用场景

* 异步处理
* 应用解耦（发布与订阅
* 流量削峰（比如秒杀业务

### 2、重要概念

消息代理

目的地

1. 队列（点对点消息通信）：
   * 消息发送者发送消息，消息代理者将其放入一个队列，消息接收者从队列中去除消息内容，消息被读取后移出队列
   * 消息只有唯一的发送者和接受者，并不是说只能有一个接收者
2. 主题（发布/订阅消息通信）：发送者发送消息到主题，多个监听者订阅这个主题，就会在消息到达的同时收到消息

### 3、RabbiteMQ

#### 核心概念

* **message**就类似于HTTP response呗
* **publisher**消息的生产者，一个向交换器发布消息的客户端应用程序
* **exchange**交换器，用来接收生产者发送的消息并将这些消息路由给服务器中的消息队列
* **queue**消息队列，用来保存消息直到发送给消费者
* **binding**绑定，用于消息队列和交换器之间的关联。一个绑定就是基于路由键将交换器和消息队列连接起来的路由规则，所以可以将将交换器理解成一个由绑定构成的路由表
* **connect**没什么好说的
* **channel**信道，信道是建立在真是TCP链接里面的虚拟链接，AMQP命令都是通过信道发送
* **consumer**信息的消费者，表示一个从消息队列中取出消息的应用程序
* **virtual host**虚拟主机，表示一批交换器、消息队列和相关对象
* **broker**表示消息队列服务器实体

#### 运行机制

![1554897801732]({{ site.url }}/assets/2020-03-13-spring-boot.assets/1554897801732.png)

#### spring自动配置

1. 自动配置了连接工厂Connection Factory
2. RabbitProperties封装了RabbitMQ的配置
3. RabbitMQTemplate给RabbitMQ发送和接收消息的
4. AmpqAdmin：RabbitMQ系统管理组件，生成队列什么的（自动注入）
5. @EnableRabbit 开启基于注解的rabbitMQ
6. @RabbitListener监听队列内容

## 九、SpringBoot与搜索

`sudo docker run --name mysearch -e ES_JAVA_OPTS="-Xms512m -Xmx512m" -d -p 9200:9200 -p 9300:9300 02982be5777d` 

![1555051392074]({{ site.url }}/assets/2020-03-13-spring-boot.assets/1555051392074.png)

### 整合进SpringBoot

Spring Boot默认支持两种技术来和ES交互

1. Jest

   需要导入工具包

   Bean类如有主键需要指定`@JestId`

   ```java
   @Autowired
   JestClient jestClient;
   
   // 1、给ES中索引（保存
   @Test
   public void contextLoads() {
       Article article = new Article(1, "vwmin", "yyy", "yyy");
       // 构建一个索引
       Index index = new Index.Builder(article).index("vwmin").type("article").build();
       // 执行
       try {
           jestClient.execute(index);
       } catch (IOException e) {
           e.printStackTrace();
       }
   
   }
   
   // 2、搜索
   @Test
   public void search(){
       String json = "{\n" +
               "\t\"query\": {\n" +
               "\t\t\"match\": {\n" +
               "\t\t\t\"content\": \"yyy\"\n" +
               "\t\t}\n" +
               "\t}\n" +
               "}";
   
       Search search = new Search.Builder(json).addIndex("vwmin").addType("article").build();
       try {
           SearchResult result =  jestClient.execute(search);
           System.out.println(result.getJsonString());
       } catch (IOException e) {
           e.printStackTrace();
       }
   }
   ```

2. SpringData ElasticSearch

   1. `Client` 需要指定`clusterNodes` `clusterName`

   2. `ElasticSearchTemplate`

   3. `ElasticSearchRepository`类似JPA 

      Bean类需要注明`@Document`

## 十、SpringBoot与任务

### 1、异步任务

`@EnableAsync`开启异步注解

`@Async`于方法名

### 2、定时任务

 `@EnableScheduling`开启基于注解的定时任务

`@Scheduled`于方法名

![1555071446534](/assets/2020-03-13-spring-boot.assets/1555071446534.png)

### 3、事件

```java
//事件推送者
private final ApplicationEventPublisher publisher;
publisher.pulish(myEvent);
```

```java
//创建一个自定义事件
public class MyEvent extends ApplicationEvent {
    public MyEvent(Object source) {
        super(source);
    }
}
```

```java
//创建一个事件接收者
@Component
public class NewWorksListener implements ApplicationListener<NewWorksEvent> {

    @Override
    public void onApplicationEvent(MyEvent event) {
        // 收到事件
    }
    
}
```

事件推送后，默认时同步等待事件处理的

对事件处理方法标记异步，来使事件异步执行

## 十一、SpringBoot与安全

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

编写配置类：

```java
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    //定义认证规则
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
//        super.configure(auth); //有一些默认的授权规则
        auth.xxx();//设置认证信息的来源，配置用户角色
    }
    //定制请求的授权规则
    @Override
    protected void configure(HttpSecurity http) throws Exception {
//        super.configure(http);
        http.authorizeRequests().antMatchers("/").permitAll()；//针对资源设置不同的权限需求

        //开启自动配置的登陆功能，带登陆界面 效果：如果未登陆会来到登陆页面
        http.formLogin();
        /**
     	* 1. 访问/login会来到自动配置好的登陆界面
     	* 2. 登陆错误会重定向到/login?error
     	*/
        
        //开启自动配置的注销功能
        http.logout();
        /**
        * 1. 访问/logout表示注销等钱用户，清空session
        * 2. 登出后重定向至/login?logout，可定制
        */
        
        http.rememberMe(); //开启记住我功能
        /**
        * 开启后，登陆界面会多一个rememberMe勾选框，选中后登陆应用会向浏览器发送一个cookie
        * 这个cookie的默认生命周期为14天
        * 注销会删除这个cookie
        */
    }
}
```

引入SpringSecurity和Thymeleaf的整合模块

```xml
<!-- https://mvnrepository.com/artifact/org.thymeleaf.extras/thymeleaf-extras-springsecurity4 -->
<dependency>
    <groupId>org.thymeleaf.extras</groupId>
    <artifactId>thymeleaf-extras-springsecurity5</artifactId>
    <version>3.0.4.RELEASE</version>
</dependency>
```

添加工具 `xmlns:sec="http://www.thymeleaf.org/thymeleaf-extras-springsecurity4"`

常用函数：

```C++
sec:authorize="!isAuthenticated()" //判断用户是否已认证
sec:authentication="principal.authorities" //用户拥有的角色集
sec:authorize="hasRole('VIP1')" //判断用户是否拥有某一角色
```

## 其他

### 动态创建Bean

```java
DefaultListableBeanFactory beanFactory = (DefaultListableBeanFactory) applicationContext.getBeanFactory();
        BeanDefinitionBuilder beanDefinitionBuilder = BeanDefinitionBuilder.genericBeanDefinition(obj.getClass());
        beanFactory.registerBeanDefinition("BeanName", beanDefinitionBuilder.getBeanDefinition());
```

### 动态加入Bean

```java
applicationContext.getBeanFactory().registerSingleton("Beanname", obj);
```

L(7x?_d*tw+qAXHD