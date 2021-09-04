# nacos

**注册中心**

   1.下载nacos service启动 （修改mode为standalone）

2. nacos pom

   <dependency>
       <groupId>com.alibaba.cloud</groupId>
       <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
   </dependency>

3. yml（properties）配置

   ```
   spring:
     #nacos注册
     cloud:
       nacos:
         discovery:
           server-addr: 127.0.0.1:8848
         config:
           server-addr: 127.0.0.1:8848
     application:
       name: gulimall-coupon
   ```

4.springboot启动项加注解  @EnableDiscoveryClient

5.http://127.0.0.1:8848/nacos/ 账号密码nacos

**配置中心**

1.pom依赖

```
<!--   nacos配置中心-->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
```

**坑点**：不知是springboot还是springcloud的版本问题，bootstrap配置不启动

解决方法 加入依赖：

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bootstrap</artifactId>
    <version>3.0.2</version>
</dependency>
```

2.bootstrap配置

```properties
spring.application.name=gulimall-coupon                     #服务名（有时需要改）
spring.cloud.nacos.config.server-addr=127.0.0.1:8848           #nacos服务地址
spring.cloud.nacos.config.namespace=9b8ec82a-fdef-48aa-b982-d27803d7b115#命名空间
spring.cloud.nacos.config.group=dev                 #group
spring.cloud.nacos.config.file-extension=yml        #如果远程配置用yml 请加这个
```

3.字段上加@Value获取配置信息  controller上加@RefreshScope不用重启springboot也能刷新远程配置信息

# Feign

  1.依赖pom

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

2.写调用服务的接口

![image-20210810150352735](E:\code-personal\note\note\assert\image-20210810150352735.png)

3.加注解

![image-20210810150441499](E:\code-personal\note\note\assert\image-20210810150441499.png)

4.直接调用

![image-20210810150504516](E:\code-personal\note\note\assert\image-20210810150504516.png)



流程：

![image-20210823130335989](E:\code-personal\note\note\assert\image-20210823130335989.png)

<img src="E:\code-personal\note\note\assert\image-20210823130404496.png" alt="image-20210823130404496" style="zoom:150%;" />



# Gateway

1.pom

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
```

2.加@EnableDiscoveryClient （也要加入到nacos配置中心）

3.配置yml 加入nacos（和上面一样）

4.要加入网关的微服务要配置加入nacos

5.配置路由

```yaml
spring:
  application:
    name: gulimall-gateway
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848
    gateway:
      routes:
       - id: admin_route
         uri: lb://renren-fast                             #lb负载均衡
         predicates:
           - Path=/api/**                                    #所有符合(predicates)/api/**的 url 都转发为 uri(lb://renren-fast)
         filters:
           - RewritePath=/api/(?<segment>.*),/renren-fast/$\{segment}  # 把/api/* 改变成 /renren-fast/*
```

# OSS（cloud集成）

1.导入依赖

```java
<!--        阿里云oss云存储-->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alicloud-oss</artifactId>
</dependency>
```

2.注册子账号

3.yml配置账号密码

```java
alicloud:
  access-key: **************
  secret-key: **************
  oss:
    endpoint: **************
```

4.注入oss 上传文件

```java
FileInputStream fileInputStream = new FileInputStream("E:\\code-personal\\note\\note\\assert\\image-20201104225636166.png");
oss.putObject("lyf-gulimall-bucket","image-20201104225636166.png",fileInputStream);
```

5.阿里云设置oss跨域

![image-20210817095401231](E:\code-personal\note\note\assert\image-20210817095401231.png)

6.前端使用upload组件获取签名上传文件



# 逻辑删除（MP）

1.配置全局的逻辑删除规则（省略）

2.配置逻辑删除的组件（省略）

3.给Bean加上逻辑删除注解@TableLogic



# JSR303

1.给bean添加校验注解 @NotNull @Email…………

2.controller方法参数添加@Valid

3.异常处理

**分组校验**

1.在bean中字段的@NutNull @Email…上加入分组

![image-20210817140040197](E:\code-personal\note\note\assert\image-20210817140040197.png)

标识是一个空接口 只是为了标识

![image-20210817140124079](E:\code-personal\note\note\assert\image-20210817140124079.png)

2.在controller上添加@Validated{分组接口.class}  （本来是@Valid）

![image-20210817140248606](E:\code-personal\note\note\assert\image-20210817140248606.png)



**自定义校验注解**

1.写好自定义的注解

![image-20210817145804797](E:\code-personal\note\note\assert\image-20210817145804797.png)

2.写好自定的注解对应的逻辑处理器 与注解关联

![image-20210817145432456](E:\code-personal\note\note\assert\image-20210817145432456.png)

3.使用注解

# 全局异常处理

![image-20210817134817150](E:\code-personal\note\note\assert\image-20210817134817150.png)

```java
/**
 * @program: GulimallExceptionControllerAdvice
 * @description: 全局异常处理
 * @author: liyifan
 * @create: 2021/08/17/10:35
 */

//@ResponseBody
//@ControllerAdvice(basePackages = "com.li.gulimall.product.controller")
@Slf4j
@RestControllerAdvice(basePackages = "com.li.gulimall.product.controller")
public class GulimallExceptionControllerAdvice {

    /**
     * @Description: 数据校验异常类类
     * @return:
     */
    @ExceptionHandler(value = MethodArgumentNotValidException.class)
    public R validExceptionHandler(MethodArgumentNotValidException e){
    log.error("数据校验出现问题{},异常类型{}",e.getMessage(),e.getClass());
    BindingResult bindingResult =e.getBindingResult();
    Map<String, String> errormap = new HashMap<>();
    bindingResult.getFieldErrors().forEach(fieldError ->
            errormap.put(fieldError.getField(),fieldError.getDefaultMessage()));
    return R.error(ResultInfo.Valid_Exception_Info).put("data",errormap);
    }

    /**
     * @Description: 未知错误异常类
     * @return:
     */
    @ExceptionHandler(value = Throwable.class)
    public R unknowExceptionHandler(Throwable throwable){
        return R.error(ResultInfo.Unknow_Exception_Info);
    }


}
```



# Jackson 时间格式 时区

```yaml
spring:
 jackson:
  date-format: yyyy-MM-dd HH:mm:ss
  time-zone: GMT+8
```

![image-20210820154802480](E:\code-personal\note\note\assert\image-20210820154802480.png)

# Elastic Search

springboot整合：

1.maven

```java
<dependency>
    <groupId>org.elasticsearch.client</groupId>
    <artifactId>elasticsearch-rest-high-level-client</artifactId>
    <version>7.4.2</version>
</dependency>
    
<properties>
    <elasticsearch.version>7.4.2</elasticsearch.version>
</properties>
```

2.配置configuration

```java
@Configuration
public class GulimallElasticSearchConfig {

    public static final RequestOptions COMMON_OPTIONS;

    static {
        RequestOptions.Builder builder = RequestOptions.DEFAULT.toBuilder();

        COMMON_OPTIONS = builder.build();
    }
    
    @Bean
    public RestHighLevelClient esRestClient() {

        RestClientBuilder builder = null;
        // 可以指定多个es
        builder = RestClient.builder(new HttpHost("47.98.43.244", 9200, "http"));

        RestHighLevelClient client = new RestHighLevelClient(builder);
        return client;
    }
}
```

3.测试api

```java
    /**
     * @Description: 查询es数据
     * @return:
     */
    @Test
    void search() throws IOException {
        //创建索引请求
        SearchRequest searchRequest=new SearchRequest();
        //指定索引
        searchRequest.indices("bank");
        //指定DSL检索条件
        SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
//        searchSourceBuilder.query();
//        searchSourceBuilder.from();
//        searchSourceBuilder.size();
        searchSourceBuilder.query(QueryBuilders.matchQuery("address","mill"));
        //聚合
        TermsAggregationBuilder size = AggregationBuilders.terms("ageagg").field("age").size(10);
        searchSourceBuilder.aggregation(size);
        AvgAggregationBuilder balanceAvg = AggregationBuilders.avg("balanceAvg").field("balance");
        searchSourceBuilder.aggregation(balanceAvg);

        //指定source
        searchRequest.source(searchSourceBuilder);
        //检索 获取结果
        SearchResponse searchResponse = restHighLevelClient.search(searchRequest, GulimallElasticSearchConfig.COMMON_OPTIONS);
        System.out.println(searchResponse);

    }

    /**
     * @Description: 添加或更新数据到es
     * @return:
     */
    @Test
    void index() throws IOException {
        IndexRequest indexRequest = new IndexRequest("request");
        indexRequest.id("1");
        User user = new User("lyf", 13, "男");
        String s = JSON.toJSONString(user);
        indexRequest.source(s, XContentType.JSON);

        IndexResponse index = restHighLevelClient.index(indexRequest, GulimallElasticSearchConfig.COMMON_OPTIONS);
        System.out.println(index);


    }
```



# 压力测试

![image-20210824112947390](E:\code-personal\note\note\assert\image-20210824112947390.png)

cmd:jvisiuamvm打开 java visu vm

![image-20210824113558159](E:\code-personal\note\note\assert\image-20210824113558159.png)



# nginx

域名映射

负载均衡

动静分离：将静态资源放在nginx，配置访问静态资源的路径  比如  static/

![image-20210824125548560](E:\code-personal\note\note\assert\image-20210824125548560.png)



# 缓存

![image-20210824190947109](E:\code-personal\note\note\assert\image-20210824190947109.png)

产生堆外内存溢出 OutOfDirectMemoryError

1.sprigboot2.0后用lettuce作为操作redis的客户端 它使用netty进行网络通信

2.lettuce的bug导致netty堆外内存溢出（没有及时减堆外内存）  netty如果没有指定堆外内存 默认使用-xmx 。。。m

​	可以通过 -Dio.netty.maxDirectMemory

## 缓存三个问题

![image-20210825104300920](E:\code-personal\note\note\assert\image-20210825104300920.png)

![image-20210825104316097](E:\code-personal\note\note\assert\image-20210825104316097.png)

![image-20210825104416976](E:\code-personal\note\note\assert\image-20210825104416976.png)

- 解决方案：

  1.空结果缓存：解决缓存穿透

  2.设置过期时间（加随机值）：解决缓存雪崩

  3.加锁：解决缓存击穿

  ​    加锁：单例   synchronized  无缓存-----》锁（无缓存？查数据库：返回缓存数据）   注意锁操作中还要再看一次有无缓存  双重检验

  ![image-20210825110020313](E:\code-personal\note\note\assert\image-20210825110020313.png)

  ![image-20210825110419780](E:\code-personal\note\note\assert\image-20210825110419780.png)

  ​	集群：分布式锁



# 分布式锁

## 自己简单实现

![image-20210825111440469](E:\code-personal\note\note\assert\image-20210825111440469.png)

![image-20210825111806099](E:\code-personal\note\note\assert\image-20210825111806099.png)

== set lock 111  NX

问题：若抢占到锁的线程在释放锁之前 服务器宕机  ，其他所有线程都再也拿不到锁  死锁！！

解决： 设置锁自动过期时间



![image-20210825112038448](E:\code-personal\note\note\assert\image-20210825112038448.png)

![image-20210825112209879](E:\code-personal\note\note\assert\image-20210825112209879.png)

问题：设置锁后 设置过期时间前正好宕机？

解决方式：放锁的时候一起放过期时间 （原子操作）

![image-20210825112320250](E:\code-personal\note\note\assert\image-20210825112320250.png)

![image-20210825112558968](E:\code-personal\note\note\assert\image-20210825112558968.png)

问题：自己业务超长 在业务过程中锁已过期 导致其他线程抢占了锁   而自己业务结束时  执行删除锁操作  把所有人的锁都删了？

解决方法：占锁指定uuid自己删自己的锁

![image-20210825112850719](E:\code-personal\note\note\assert\image-20210825112850719.png)

![image-20210825113119218](E:\code-personal\note\note\assert\image-20210825113119218.png)

问题：读取lock.value——————》与自己uuid对比   过程之间lock失效？

解决：用lua脚本直接对比后删除或不删除

![image-20210825113746331](E:\code-personal\note\note\assert\image-20210825113746331.png)

![image-20210825113804079](E:\code-personal\note\note\assert\image-20210825113804079.png)



最后代码：

![image-20210825113950656](E:\code-personal\note\note\assert\image-20210825113950656.png)

锁自旋地方记得sleep（）一会  不然栈溢出，或者用while自旋



## redission

**pom**

```java
<!-- 分布式锁 -->
<dependency>
    <groupId>org.redisson</groupId>
    <artifactId>redisson</artifactId>
    <version>3.15.1</version>
</dependency>
```

**配置类**

@Configuration
public class MyRedissonConfig {

```java
/**
 * 所有对Redisson的使用都是通过RedissonClient
 * @return
 * @throws IOException
 */
@Bean(destroyMethod="shutdown")
public RedissonClient redisson() throws IOException {
    //1、创建配置
    Config config = new Config();
    config.useSingleServer().setAddress("redis://47.98.43.244:6379");

    //2、根据Config创建出RedissonClient实例
    //Redis url should start with redis:// or rediss://
    RedissonClient redissonClient = Redisson.create(config);
    return redissonClient;
}
}
```

**lock方法  可重入锁**

![image-20210825124022585](E:\code-personal\note\note\assert\image-20210825124022585.png)



**注意**

![image-20210825124907132](E:\code-personal\note\note\assert\image-20210825124907132.png)

手动指定过期时间不会自动续期

不指定的话 每1/3看门狗时间（1/3 * 10）自动续期



**读写锁**

![image-20210825125751176](E:\code-personal\note\note\assert\image-20210825125751176.png)

![image-20210825125817982](E:\code-personal\note\note\assert\image-20210825125817982.png)

读时排他锁 写时共享锁

可以读读（相当于无锁）  不能读写  不能写写 不能写读

![image-20210825130123781](E:\code-personal\note\note\assert\image-20210825130123781.png)



**信号量**

![image-20210825150654774](E:\code-personal\note\note\assert\image-20210825150654774.png)

****

**countdownlatch 闭锁**

![image-20210825151216547](E:\code-personal\note\note\assert\image-20210825151216547.png)

**数据缓存一致性**

![image-20210825152628162](E:\code-personal\note\note\assert\image-20210825152628162.png)

修改数据库后直接修改缓存



![image-20210825152739985](E:\code-personal\note\note\assert\image-20210825152739985.png)



![image-20210825153903153](E:\code-personal\note\note\assert\image-20210825153903153.png)



![image-20210825154346171](E:\code-personal\note\note\assert\image-20210825154346171.png)



# 整合springchache

依赖

```java
<dependency>
    <groupId>org.springframework.b oot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
```

指定缓存类型并在主配置类上加上注解@EnableCaching

```yaml
spring:
  cache:
  	#指定缓存类型为redis
    type: redis
    redis:
      # 指定redis中的过期时间为1h
      time-to-live: 3600000
```

![image-20210825163121501](E:\code-personal\note\note\assert\image-20210825163121501.png)

默认使用jdk进行序列化（可读性差），默认ttl为-1永不过期，自定义序列化方式需要编写配置类        

```java
@Configuration
public class MyCacheConfig {
    @Bean
    public RedisCacheConfiguration redisCacheConfiguration( CacheProperties cacheProperties) {
   CacheProperties.Redis redisProperties = cacheProperties.getRedis();
    org.springframework.data.redis.cache.RedisCacheConfiguration config = org.springframework.data.redis.cache.RedisCacheConfiguration
        .defaultCacheConfig();
    //指定缓存序列化方式为json
    config = config.serializeValuesWith(
        RedisSerializationContext.SerializationPair.fromSerializer(new GenericJackson2JsonRedisSerializer()));
    //设置配置文件中的各项配置，如过期时间
    if (redisProperties.getTimeToLive() != null) {
        config = config.entryTtl(redisProperties.getTimeToLive());
    }

    if (redisProperties.getKeyPrefix() != null) {
        config = config.prefixKeysWith(redisProperties.getKeyPrefix());
    }
    if (!redisProperties.isCacheNullValues()) {
        config = config.disableCachingNullValues();
    }
    if (!redisProperties.isUseKeyPrefix()) {
        config = config.disableKeyPrefix();
    }
    return config;
}
}
```
2) 缓存自动配置

```java
// 缓存自动配置源码
@Configuration(proxyBeanMethods = false)
@ConditionalOnClass(CacheManager.class)
@ConditionalOnBean(CacheAspectSupport.class)
@ConditionalOnMissingBean(value = CacheManager.class, name = "cacheResolver")
@EnableConfigurationProperties(CacheProperties.class)
@AutoConfigureAfter({ CouchbaseAutoConfiguration.class, HazelcastAutoConfiguration.class,
                     HibernateJpaAutoConfiguration.class, RedisAutoConfiguration.class })
@Import({ CacheConfigurationImportSelector.class, // 看导入什么CacheConfiguration
         CacheManagerEntityManagerFactoryDependsOnPostProcessor.class })
public class CacheAutoConfiguration {

    @Bean
    @ConditionalOnMissingBean
    public CacheManagerCustomizers cacheManagerCustomizers(ObjectProvider<CacheManagerCustomizer<?>> customizers) {
        return new CacheManagerCustomizers(customizers.orderedStream().collect(Collectors.toList()));
    }

    @Bean
    public CacheManagerValidator cacheAutoConfigurationValidator(CacheProperties cacheProperties,
                                                                 ObjectProvider<CacheManager> cacheManager) {
        return new CacheManagerValidator(cacheProperties, cacheManager);
    }

    @ConditionalOnClass(LocalContainerEntityManagerFactoryBean.class)
    @ConditionalOnBean(AbstractEntityManagerFactoryBean.class)
    static class CacheManagerEntityManagerFactoryDependsOnPostProcessor
        extends EntityManagerFactoryDependsOnPostProcessor {

        CacheManagerEntityManagerFactoryDependsOnPostProcessor() {
            super("cacheManager");
        }

    }
```

```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnClass(RedisConnectionFactory.class)
@AutoConfigureAfter(RedisAutoConfiguration.class)
@ConditionalOnBean(RedisConnectionFactory.class)
@ConditionalOnMissingBean(CacheManager.class)
@Conditional(CacheCondition.class)
class RedisCacheConfiguration {
@Bean // 放入缓存管理器
RedisCacheManager cacheManager(CacheProperties cacheProperties, 
                               CacheManagerCustomizers cacheManagerCustomizers,
                               ObjectProvider<org.springframework.data.redis.cache.RedisCacheConfiguration> redisCacheConfiguration,
                               ObjectProvider<RedisCacheManagerBuilderCustomizer> redisCacheManagerBuilderCustomizers,
                               RedisConnectionFactory redisConnectionFactory, ResourceLoader resourceLoader) {
    RedisCacheManagerBuilder builder = RedisCacheManager.builder(redisConnectionFactory).cacheDefaults(
        determineConfiguration(cacheProperties, redisCacheConfiguration, resourceLoader.getClassLoader()));
    List<String> cacheNames = cacheProperties.getCacheNames();
    if (!cacheNames.isEmpty()) {
        builder.initialCacheNames(new LinkedHashSet<>(cacheNames));
    }
    redisCacheManagerBuilderCustomizers.orderedStream().forEach((customizer) -> customizer.customize(builder));
    return cacheManagerCustomizers.customize(builder.build());
}
```

3) 缓存使用@Cacheable@CacheEvict

第一个方法存放缓存，第二个方法清空缓存

```java
// 调用该方法时会将结果缓存，缓存名为category，key为方法名
// sync表示该方法的缓存被读取时会加锁 // value等同于cacheNames // key如果是字符串"''"
@Cacheable(value = {"category"},key = "#root.methodName",sync = true)
public Map<String, List<Catalog2Vo>> getCatalogJsonDbWithSpringCache() {
    return getCategoriesDb();
}

//调用该方法会删除缓存category下的所有cache，如果要删除某个具体，用key="''"
@Override
@CacheEvict(value = {"category"},allEntries = true)
public void updateCascade(CategoryEntity category) {
    this.updateById(category);
    if (!StringUtils.isEmpty(category.getName())) {
        categoryBrandRelationService.updateCategory(category);
    }
}

如果要清空多个缓存，用@Caching(evict={@CacheEvict(value="")})
```

4) SpringCache原理与不足

1）、读模式

    缓存穿透：查询一个null数据。解决方案：缓存空数据，可通过spring.cache.redis.cache-null-values=true
    缓存击穿：大量并发进来同时查询一个正好过期的数据。解决方案：加锁 ? 默认是无加锁的;
        使用sync = true来解决击穿问题
    缓存雪崩：大量的key同时过期。解决：加随机时间。

2)、写模式：（缓存与数据库一致）

    读写加锁。
    引入Canal，感知到MySQL的更新去更新Redis
    读多写多，直接去数据库查询就行

3）、总结：

常规数据（读多写少，即时性，一致性要求不高的数据，完全可以使用Spring-Cache）：

写模式(只要缓存的数据有过期时间就足够了)

特殊数据：特殊设计


```yaml
spring：
   cache:
     type: redis
```



# 线程池

四种方式：1.继承thread 重写run方法 然后 start

​					2.实现runable接口 重写run方法 丢进new Thread 然后 start

​					3.实现callable接口 重写call方法  丢进futureTask  futureTask丢进new Thread 然后start  futureTask.get可以拿到call方法的返回值

​					4.线程池

**为什么用线程池？** 更好地管理线程 不然进来一个请求new一个线程  那么会撑爆资源 而线程池比如最多就产生50个线程。自己去一个个创建的话，看起来是同步  实际上cpu要不断上下文切换，java线程是映射到操作系统内核线程的，来回切换很耗资源

![image-20210826125711696](E:\code-personal\note\note\assert\image-20210826125711696.png)



**流程**

![image-20210826124743737](E:\code-personal\note\note\assert\image-20210826124743737.png)![image-20210826125413499](E:\code-personal\note\note\assert\image-20210826125413499.png)



# 遇到的问题

1.redis 堆外内存溢出  OutOfDirectMemoryError

- sprigboot2.0后用lettuce作为操作redis的客户端 它使用netty进行网络通信

- lettuce的bug导致netty堆外内存溢出  netty如果没有指定堆外内存 默认使用-xmx 。。。m

​	可以通过 -Dio.netty.maxDirectMemory进行设置

解决方案：不能使用-Dio.netty.maxDirectMemory只去调大堆外内存

方法一：升级lettuce客户端   方法二：使用jredis



2.redis的缓存穿透 缓存雪崩 缓存击穿

解决方案： 

1.空结果缓存：解决缓存穿透

2.设置过期时间（加随机值）：解决缓存雪崩

3.加锁：解决缓存击穿



3.遇到死锁？

redis分布式锁

自己实现分布式锁各种问题？

![image-20210825111806099](E:\code-personal\note\note\assert\image-20210825111806099.png)

若抢占到锁的线程在释放锁之前 服务器宕机  ，其他所有线程都再也拿不到锁  死锁！！

解决： 设置锁自动过期时间

设置过期时间前过期？

解决：设置锁的时候一起放过期时间



4.验证码恶意多次刷新？

验证码中储存时间戳，发送验证码前拿出时间戳数据进行对比，超过1分钟以上放行



5.rabbitmq自动释放问题？

b接收到a通过mq发送的自动关闭的消息后，b要通过关闭消息（比如订单是a 库存是b）的id去数据库 确认库存消息，假如订单是未解锁 才释放。因为a可能会回滚  也就是说（通过数据库实时数据 status）判断是否解锁 已经解锁过的就不去解锁了  保证幂等（否则订单a主动释放给b的消息释放了一次库存，库存系统锁单时候自动发送的释放订单的mq又解锁一次，加了两次库存 就不幂等）



# 社交登录（gitee）

![image-20210831115058752](E:\code-personal\note\note\assert\image-20210831115058752.png)

1.到第三方提供的页面获取code

    请求方式：Get
    请求路径： https://gitee.com/oauth/authorize?client_id={client_id}&redirect_uri={redirect_uri}&response_type=code
    请求参数

参数名	                                  参数内容	                                                        参数讲解
client_id	            xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx	                         上面你自己的 client_id
redirect_uri	      http://localhost:8811/callback	                                   回调地址
response_type	 code	                                                                               不用改

2.根据code获取token

通过 code，获取 access_token

使用 postman 工具获取 access_token
请求方式：Post
请求路径：https://gitee.com/oauth/token?grant_type=authorization_code&code={code}&client_id={client_id}&redirect_uri={redirect_uri}&client_secret={client_secret}

3.根据token获取用户数据

- 请求方式：Get
- 请求地址：`https://gitee.com/api/v5/user?access_token={access_token}`
- 返回结果就是我们想要的用户数据



# session共享（springsession-redis）

![image-20210831152412153](E:\code-personal\note\note\assert\image-20210831152412153.png)

![image-20210831152444765](E:\code-personal\note\note\assert\image-20210831152444765.png)

![image-20210831152550486](E:\code-personal\note\note\assert\image-20210831152550486.png)

![image-20210831152927695](E:\code-personal\note\note\assert\image-20210831152927695.png)



1.pom

```java
<!--        springsession，redis存储 共享session-->
<dependency>
    <groupId>org.springframework.session</groupId>
    <artifactId>spring-session-data-redis</artifactId>
</dependency>
```

2.配置(还要配redis连接信息)

```yaml
spring: 
 session:
   store-type: redis
```

3.启动类 @EnableRedisHttpSession

4.修改序列化为json （不然要让类实现序列化接口）

5.springsession配置放大作用域和序列化方式（json）

```java
@Configuration
public class GulimallSessionConfig {

    @Bean
    public CookieSerializer cookieSerializer() {

        DefaultCookieSerializer cookieSerializer = new DefaultCookieSerializer();

        //放大cookie作用域
//        cookieSerializer.setDomainName("gulimall.com");
        cookieSerializer.setDomainName("localhost");
        //设置cookiename
        cookieSerializer.setCookieName("GULISESSION");

        return cookieSerializer;
    }


    @Bean
    public RedisSerializer<Object> springSessionDefaultRedisSerializer() {
        return new GenericJackson2JsonRedisSerializer();
    }

}
```

6.直接放session  取

```java
session.setAttribute(LOGIN_USER,data);                  //sessionName，内容
```



7.springsession原理

![image-20210831162512079](E:\code-personal\note\note\assert\image-20210831162512079.png)

![image-20210831162648666](E:\code-personal\note\note\assert\image-20210831162648666.png)

简而言之，在这个doFilterInternal方法中用过滤器，把原生的request和response包装成了自己提供的wrappedRequest和wrapperdResponse后放行。之后在controller中 request.getSession获取session的时候用的已经是他们封装过的sessionRepository了 所有的操作封装了操作redis。



# 单点登录

![image-20210831173608440](E:\code-personal\note\note\assert\image-20210831173608440.png)

1.系统1去登陆中心登录 登陆成功后，登陆中心生成一个token放在redis中，登陆成功后重定向到系统页面中，把token的key返回给客户端并放在cookie中。

2.如果客户端得到了token的话，处理token，获取信息。

3.其它系统登陆时，如果有cookie，直接不用登陆了





# 用到xxxx技术？

1.用到aop？

spring-cache源码中  用到了aop     aop（查看是否命中缓存？返回缓存：去到真实业务代码）代码

![image-20210825175301607](E:\code-personal\note\note\assert\image-20210825175301607.png)

2.ioc问题

哪里见过postprocessor方法？ springsession中用到了@PostConstruct做初始化工作，提供了一个序列化器



3.用到注解 反射？

- 自己实现登陆权限的时候，使用自定义注解放在controller上，配合拦截器，拦截器通过通过反射读取注解信息 实现权限控制

- jsr303自定义校验，使用到了自定义注解，配合逻辑控制器实现校验

- spring中大量用到反射  用注解开发的话用到大量注解

- 为什么用注解？我认为注解是一种标志，并且带有一些信息。比如scope（singleton）  就可以扫描到这个注解的时候我们知道噢 他可能要以某种方式来实例化 ，以什么方式？单例。

  

4.用到redis？

缓存、分布式锁、储存验证码、配合springsesison做session共享



5.用到拦截器？

1.自己实现身份认证的时候  拦截器配合反射、自定义注解  认证身份

2.购物车业务

3.springsession源码中用到  用拦截器替换了request和response



5. 拦截器?----   临时购物车   保存临时会员身份  配合threadlocal
6. 哈希冲突？   拉链法hashmap       找下一个key-- threadlocal

7.内存泄漏？  threadlocal  弱引用 一定要手动清空

8.mq？  订单释放、库存释放

# rabitMq

![image-20210901161246701](E:\code-personal\note\note\assert\image-20210901161246701.png)

订单：

下订单---->锁库存

​     锁完库存给mq队列发消息（订单信息id、锁库存信息、库存工作单）（延迟队列 40min）

   40min后收到消息假如库存工作单显示已锁定 通过订单id去数据库查 若查不到id（说明下单出现异常 回滚）则解锁库存（通过库存信息 比如-4了  那就+4）把status改为已解锁  若查到id 则订单成功 不用做什么 若没解锁成功把消息返回队列  下次继续尝试解锁 

![image-20210903174231047](E:\code-personal\note\note\assert\image-20210903174231047.png)

![image-20210903174433512](E:\code-personal\note\note\assert\image-20210903174433512.png)

订单释放后主动给解锁库存的队列发一条消息，主动释放一次，防止订单释放出现网络延迟比库存慢

![image-20210904091141588](E:\code-personal\note\note\assert\image-20210904091141588.png)

![image-20210904091234778](E:\code-personal\note\note\assert\image-20210904091234778.png)



![image-20210904091253241](E:\code-personal\note\note\assert\image-20210904091253241.png)

监听到数据后，拿数据去数据库真正做了操作（比如释放库存），但是在手动ack之前宕机了。消息重新放进队列，造成重复消费

解决方法：消费代码设计成幂等的、或者用防重表（记录消息是否被处理过）、也可以看消息的redelivered字段 查看消息是否是重新投递的



![image-20210904091653954](E:\code-personal\note\note\assert\image-20210904091653954.png)

# 遇到的设计模式？

1.装饰者模式

​	springsession用到了装饰者模式  包装原来的类  详见笔记session共享（springsession-redis）的7的内容

