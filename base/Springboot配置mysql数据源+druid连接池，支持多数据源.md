jSpringboot配置mysql数据源+druid连接池，支持多数据源

一、配置mysql+druid

1、导入数据源所需jar，此处只导入了必要的，其他工具自行配置

```java
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
            <version>1.1.21</version>
        </dependency>
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
            <version>1.9.4</version>
        </dependency>    
```



2、配置yml，若只需要配置单数据源，自行删除相关数据库配置即可。

```java
spring:
  # jackson
  jackson:
    date-format: yyyy-MM-dd HH:mm:ss
    time-zone: GMT+8

  #多数据源需要配置
  main:
    allow-bean-definition-overriding: true
  # druid数据源配置
  datasource:
    name: druidDataSource
    type: com.alibaba.druid.pool.DruidDataSource
    druid:
      first:
        driver-class-name: com.mysql.cj.jdbc.Driver
        url: jdbc:mysql://ip:port/db_name?serverTimezone=GMT%2B8&zeroDateTimeBehavior=convertToNull&autoReconnect=true
        username: username
        password: password
      second:
         url: jdbc:mysql://ip:port/db_name?serverTimezone=GMT%2B8&zeroDateTimeBehavior=convertToNull&autoReconnect=true
        username: username
        password: password
      third:
         url: jdbc:mysql://ip:port/db_name?serverTimezone=GMT%2B8&zeroDateTimeBehavior=convertToNull&autoReconnect=true
        username: username
        password: password


      filters: stat,wall,config
      max-active: 100
      initial-size: 1
      max-wait: 60000
      min-idle: 1
      time-between-eviction-runs-millis: 60000
      min-evictable-idle-time-millis: 300000
      validation-query: select 'x'
      test-while-idle: true
      test-on-borrow: false
      test-on-return: false
      pool-prepared-statements: true
      max-open-prepared-statements: 50
      max-pool-prepared-statement-per-connection-size: 20
      web-stat-filter:
        # 添加过滤规则
        url-pattern: /*
        # 忽略过滤格式
        exclusions: "*.js,*.gif,*.jpg,*.png,*.css,*.ico,/druid/*"
      stat-view-servlet:
        # 设置ip白名单
        allow: 127.0.0.1
        # 设置ip黑名单，优先级高于白名单
        deny:
        # 设置控制台管理用户
        #        login-username: root
        #        login-password: root
        # 是否可以重置数据
        reset-enable: false
        # 开启druid监控页面
        enabled: true

```



二、配置多数据源

1、配置多数据源名称的枚举接口，方便管理

```java
/**
 * @ClassName : DataSourceConfig
 * @Description : 多数据源名称
 * @Author : Jinwei
 * @Date: 2020-05-29 09:59
 */
public interface DataSourceNames {

    String FIRST = "first";
    String SECOND = "second";
    String THIRD = "third";

}

```



2、配置注解，在使用数据源时，直接在需要的配置的方法上面加上数据源注解即可。

```java
/**
 * @ClassName : DataSource
 * @Description : 多数据源注解
 * @Author : Jinwei
 * @Date: 2020-05-29 10:12
 */

@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface DataSource {
    String name() default "";
}
```



3、配置多数据源实例

```java
/**
 * @ClassName : DynamicDataSourceConfig
 * @Description : 多数据源配置
 * @Author : Jinwei
 * @Date: 2020-05-29 10:20
 */
@Configuration
public class DynamicDataSourceConfig {
    //对应yml文件中的数据源名称
    @Bean
    @ConfigurationProperties("spring.datasource.druid.first")
    public DataSource firstDataSource() {
        return DruidDataSourceBuilder.create().build();
    }

    @Bean
    @ConfigurationProperties("spring.datasource.druid.second")
    public DataSource secondDataSource() {
        return DruidDataSourceBuilder.create().build();
    }

    @Bean
    @ConfigurationProperties("spring.datasource.druid.third")
    public DataSource thirdDataSource() {
        return DruidDataSourceBuilder.create().build();
    }

    //对应yml文件中的数据源名称，以及枚举类中的名称
    @Bean
    @Primary
    public DynamicDataSource dataSource(DataSource firstDataSource, DataSource secondDataSource, DataSource thirdDataSource) {
        Map<Object, Object> targetDataSources = new HashMap<>();
        targetDataSources.put(DataSourceNames.FIRST, firstDataSource);
        targetDataSources.put(DataSourceNames.SECOND, secondDataSource);
        targetDataSources.put(DataSourceNames.THIRD, thirdDataSource);
        return new DynamicDataSource(firstDataSource, targetDataSources);
    }
}
```



4、配置动态数据源

```java
/**
 * @ClassName : DynamicDataSource
 * @Description : 动态数据源
 * @Author : Jinwei
 * @Date: 2020-05-29 10:22
 */
public class DynamicDataSource extends AbstractRoutingDataSource {
    private static final ThreadLocal<String> contextHolder = new ThreadLocal<>();

    public DynamicDataSource(DataSource defaultTargetDataSource, Map<Object, Object> targetDataSources) {
        super.setDefaultTargetDataSource(defaultTargetDataSource);
        super.setTargetDataSources(targetDataSources);
        super.afterPropertiesSet();
    }

    @Override
    protected Object determineCurrentLookupKey() {
        return getDataSource();
    }

    public static void setDataSource(String dataSource) {
        contextHolder.set(dataSource);
    }

    public static String getDataSource() {
        return contextHolder.get();
    }

    public static void clearDataSource() {
        contextHolder.remove();
    }

}
```



5、配置多数据源切面处理

```java

/**
 * @ClassName : DataSourceAspect
 * @Description : 多数据源切面处理类
 * @Author : Jinwei
 * @Date: 2020-05-29 10:25
 */
@Aspect
@Component
public class DataSourceAspect {
    
    @Pointcut("@annotation(com.kalvin.kvf.modules.zax.common.dateSourceConfig.DataSource) " +
            "|| @within(com.kalvin.kvf.modules.zax.common.dateSourceConfig.DataSource)")
    public void dataSourcePointCut() {

    }

    //此处可以获取到类和方法上面的注解，大多数情况下，只需要使用类上面的，或者方法上面的。可以自行删减一个。
    @Around("dataSourcePointCut()")
    public Object around(ProceedingJoinPoint point) throws Throwable {
        //获得类上的数据源注解
        Class clazz = point.getSignature().getDeclaringType();
        DataSource dataClass = (DataSource) clazz.getAnnotation(DataSource.class);
        //获得方法上的数据源注解
        MethodSignature signature = (MethodSignature) point.getSignature();
        Method method = signature.getMethod();
        DataSource dsMethod = method.getAnnotation(DataSource.class);

        if (dataClass != null) {
            dsMethod = dataClass;
        }

        if (dsMethod == null) {
            DynamicDataSource.setDataSource(DataSourceNames.FIRST);
            logger.debug("set datasource is " + DataSourceNames.FIRST);
        } else {
            DynamicDataSource.setDataSource(dsMethod.name());
            logger.debug("set datasource is " + dsMethod.name());
        }

        try {
            return point.proceed();
        } finally {
            DynamicDataSource.clearDataSource();
            logger.debug("clean datasource");
        }
    }

}
```



三、使用多数据源、增加多数据源

1、使用多数据源

只需要在你所需要使用多数据源的数据访问层（即service）层，对应的实现类上面，或者实现类中对应的方法上面，增加注解：

```java
@DataSource(name = DataSourceNames.FIRST)
@DataSource(name = DataSourceNames.SECOND)
@DataSource(name = DataSourceNames.THIRD)
```



2、增加多数据源

增加多数据源，按照次流程，依次在yml文件、数据源名称枚举类、多数据源实例类，增加对应的代码。使用时与其他的一样，都是添加注解。



四、问题解决

1、多数据源切面无法进入

方案：在切面上，添加注解：

```java
   @Pointcut("@annotation(com.kalvin.kvf.modules.zax.common.dateSourceConfig.DataSource) " +
            "|| @within(com.kalvin.kvf.modules.zax.common.dateSourceConfig.DataSource)")
    public void dataSourcePointCut() {

    }
```



2、多数据源在service中，或是在循环中，只生效一次。

剖析：测试中发现，如果在service方法上添加数据源，在方法内写循环，写mapper，此时可能出现指定数据源只出现一次的情况。

方案：需要写一个公共的service，此service专用于查询mapper，每个mapper一个方法，每个方法一个注解。







