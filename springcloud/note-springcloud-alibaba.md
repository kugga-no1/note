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

