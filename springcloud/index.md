# SpringCloud


# SpringCloud

### 1. Eureka

##### 1. 提供者与消费者

* 服务提供者: 一次业务中,被其他服务调用的服务(提供接口给其他服务)
* 服务消费者: 一次业务中,调用其他服务的服务(调用其他服务提供的接口)
* 提供者与消费者角色是**相对**
* 一个服务可以同时是服务提供者和服务消费者

##### 2. Eureka注册中心

1. 注册服务信息
2. 拉取服务
3. 负载均衡
4. 远程调用
5. 心跳续约 30s/次

##### 1. Eureka的作用

* 消费者该如何获取服务提供者的具体信息?
  1. 服务提供者启动时向eureka注册自己的信息
  2. eureka保存这些信息
  3. 消费者根据服务名称想eureka拉取提供者信息
* 如果有多个服务提供者,消费者该如何选择?
  1. 服务消费者利用负载均衡算法,从服务列表中挑选一个
* 消费者如何感知服务提供者健康状态?
  1. 服务提供者会每隔30s向eurekaServer发送心跳请求,报告健康状态
  2. eureka会更新记录服务列表信息,心跳不正常会被剔除
  3. 消费者就可以拉取最新的信息

##### 2. 服务注册

* 引入eureka-client依赖
* 在application.yml中配置eureka地址



### 2. Ribbon

##### 1.  Ribbon负载均衡规则

* 规则接口是IRule
* 默认实现是ZoneAvoidanceRule,根据zone选择服务列表,然后轮询

##### 2. 负载均衡自定义方式

* 代码方式: 配置灵活,但修改时需要重新打包发布
* 配置方式: 直观,方便,无需重新打包发布,但是无法做全局配置

##### 3. 饥饿加载

* 开启饥饿加载 eager-load
* 指定饥饿加载的服务名称



### 3. Nacos

##### 1. yml配置

```yml
  cloud:
    nacos:
      server-addr: 110.40.236.91:8848 # nacos服务端地址
      discovery:
        cluster-name: HZ # 集群名称
```



##### 2. 根据权重负载均衡

**权重为0不会被访问**

##### 3. 环境隔离-namespace

Nacos中服务存储和数据存储的最外层都是一个名为namespace的东西,用来做最外层隔离!

* namespace用来做环境隔离
* 每个namespace都有唯一id
* 不同namespace下的服务不可见

##### 5. nacos注册中心细节分析

##### 6. 临时实例和非临时实例

服务注册到Nacos时,可以选择注册为临时或非临时实例,通过下面的配置来设置:

```yml
ephemeral: true # 是否是临时实例
```



##### 7. Nacos与eureka异同

1. Nacos与eureka的共同点

* 都支持服务注册和服务拉取
* 都支持服务提供者心跳方式做健康检测

2. Nacos与Eureka的区别

* Nacos支持服务端主动检测提供者状态: 临时实例采用心跳模式,非临时实例采用主动检测模式
* 临时实例心跳不正常会被剔除,非临时实例则不会被剔除
* Nacos支持服务列表变更的消息推送模式,服务列表更新更加及时
* Nacos集群默认采用AP方式,当集群中存在非临时实例时,采用CP模式;Eureka采用AP模式

##### 8. 统一配置管理


将配置交给Naocs管理的步骤

* 在Nacos中添加配置文件
* 在微服务中引入nacos的config依赖
* 在微服务中添加bootstrap.yml,配置nacos地址,当前环境,服务名称,文件后缀名.这些决定了启动时去nacos读取哪个文件

##### 9. 配置自动刷新

Nacos中的配置文件变更后,微服务无需重启就可以感知.不过需要通过下面两种配置实现:

* 方式一: 在@Value注入的变量所在类添加注解@RefreshScope

* 方式二: 使用@ConfigurationProperties注解

  ```java
  @Data
  @Component
  @ConfigurationProperties(prefix = "pattern")
  public class PatternProperties {
      private String dateformat;
  }
  ```

  ```java
  @Autowired
      private PatternProperties patternProperties;
  ```

* 不是所有的配置都适合放到配置更新,维护起来比较麻烦
* 建议将一些关键参数,需要运行时调整的参数放到nacos配置中心,一般都是自定义配置

##### 10. 多环境配置共享

* 微服务启动时会从nacos读取多个配置文件,无论profile如何变化,共享的文件一定会被加载
* 多种配置的优先级: 服务名-profile.yaml > 服务名称.yaml > 本地配置

### 4. Feign

> RestTemplate方式调用存在的问题: 
>
> 1. 代码可读性差,编程体验不统一
> 2. 参数复杂URl难以维护



> Feign: Feign是一个声明式的http客户端,起作用就是帮助我们优雅的实现http请求的发送,解决上面提到的问题.

##### 1. Feign的使用步骤:

1. 引入依赖
2. 添加@EnableFeignClients注解
3. 编写FeignClient接口
4. 使用FeignClient中定义的方法代替RestTemplate

##### 2. 自定义Feign的配置

Feign运行自定义配置来覆盖默认配置,可以修改的配置如下:


一般我们需要配置的就是日志级别



> 配置Feign日志有两种方式: 

* 方式一: 配置文件方式

  1. 全局生效: 

     ```yml
     feign:
       client:
         config:
           default:
             loggerLevel: FULL
     ```

  2. 局部生效:

     ```yml
     feign:
       client:
         config:
           userservice: # 服务名
             loggerLevel: FULL
     ```

* 方式二: java代码方式,需要先声明一个Bean

```java
public class FeignClientConfiguration {
		@Bean
		public Logger.Level FeignLogLevel(){
				return Logger.Level.BASIC;
		}
}
```

1. 如果是全局配置,则霸道放到@EnableFeignClients这个注解中

   ```java
   @EnableFeignClients(defaultConfiguration = FeignClientConfiguration.class)
   ```

2. 如果是局部配置,则把他放到@FeignClient这个注解中

   ```java
   @FeignClient(value = "userservice" , configuration = FeignClientConfiguration.class)
   ```

##### 3. Feign的性能优化

Feign底层的客户端实现:

* URLConnection : 默认实现,不支持连接池
* Apache HttpClient : 支持连接池
* OKHttp: 支持连接池

因此优化Feign的性能主要包括:

* 使用连接池代替默认的URLConnection
* 日志级别,最好用basic 或none

**设置连接池参数**

```yml
  httpclient:
    enabled: true # 开启feign对HttpClient的支持
    max-connections: 200 # 最大的连接数
    max-connections-per-route: 50 # 每个路径的最大连接数
```

##### 4. Feign的最佳实践

> 方式一(继承): 给消费者的feignClient和提供者的controller定义同一的父接口作为标准 

* 服务紧耦合
* 父接口参数列表中的映射不会被继承

> 方式二(抽取): 将FeignClient抽取为独立模块,并且把接口有关的pojo,默认的Feign配置都放到这个模块中,提供给所有消费者使用


##### 5. 抽取FeignClient

> 实现最佳实践方式二的步骤:

1. 创建一个module,命名为feign-api,然后引入feign的starter依赖
2. 将原先的client,pojo都移到feign-api项目中
3. 在@EnableFeignClients注解中添加clients,指定具体FeignClient的字节码

### 5. Gateway

##### 1. 为什么需要网关

* 身份认证和权限校验
* 服务路由,负载均衡
* 请求限流

##### 2. 网关的技术实现

> 在SpringCloud中网关的实现包括两种:

* gateway
* zuul

##### 3. 搭建网关服务

> 1. 引入依赖: 网关依赖,服务发现依赖

> 2. 编写配置文件

```yml
server:
  port: 8084
spring:
  application:
    name: gateway # 服务名称
  cloud:
    nacos:
      server-addr: 110.40.236.91:8848
    gateway:
      routes:
        - id: user-service # 路由标识
          uri: lb://userservice # 路由的目标地址
          predicates: # 路由断言,判断是否符合规则
            - Path=/user/** # 路径断言,.判断路径是否是以/user开头
        - id: order-service # 路由标识
          uri: lb://orderservice # 路由的目标地址
          predicates: # 路由断言,判断是否符合规则
            - Path=/order/** # 路径断言,.判断路径是否是以/user开头
```




