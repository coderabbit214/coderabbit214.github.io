# SpringCloud常用


# SpringCloud常用

## Nacos 注册服务中心

### 启动命令

```shell
//单机
sh startup.sh -m standalone
```

### 简介

1. 为什么叫Nacos
   - 前四个字母分别为Naming和Configuration的前两个字母，最后的s为Service。
2. 是什么

   - 一个更易于构建云原生应用的动态服务发现、配置管理和服务管理平台。
   - Nacos: Dynamic Naming and Configuration Service
   - Nacos就是注册中心＋配置中心的组合 -> Nacos = Eureka+Config+Bus
3. **核心功能点**:
   - **服务注册**:  Nacos Client会通过发送REST请求想Nacos Server注册自己的服务，提供自身的元数据，比如IP地址，端口等信息。Nacos Server接收到注册请求后，就会把这些元数据存储到一个双层的内存Map中。

   - **服务心跳**:  在服务注册后，Nacos Client会维护一个定时心跳来维持统治Nacos Server,说明服务一致处于可用状态，防止被剔除，默认5s发送一次心跳

   - **服务同步**:  Nacos Server集群之间会相互同步服务实例，用来保证服务信息的一致性。

   - **服务发现**： 服务消费者(Nacos Client)在调用服务提供的服务时，会发送一个REST请求给Nacos Server,获取上面注册的服务清单，并且缓存在Nacos Client本地,同时会在Nacos Client本地开启一个定时任务拉取服务最新的注册表信息更新到本地缓存。

   - **服务健康检查**:  Nacos Server 会开启一个定时任务来检查注册服务实例的健康情况，对于超过15s没有收到客户端心跳的实例会将他的healthy属性设置为false(客户端服务发现时不会发现)，如果某个实例超过30s没有收到心跳，直接剔除该实例(被剔除的实例如果恢复发送心跳则会重新注册)

### 搭建

1. pom

   ```xml
   <!--nacos客户端-->
   <dependency>
   	<groupId>com.alibaba.cloud</groupId>
   	<artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
   </dependency>
   ```

2. yml（添加注册中心地址）

   ```yml
   spring:
     cloud: 
       nacos: 
         discovery: 
           server-addr: localhost:8848
   ```

3. 在主类上添加**@EnableDiscoveryClient**注解

   ```java
   @SpringBootApplication
   @EnableDiscoveryClient
   public class ProductServer {
       public static void main(String[] args) {
           SpringApplication.run(ProductServer.class,args);
       }
   }
   ```

4. 查看nacos控制台出现服务



### 配置中心

#### Nacos Confifig入门

> Nacos中的dataid的组成格式及与SpringBoot配置文件中的匹配规则
>
> > 说明：之所以需要配置spring.application.name，是因为它是构成Nacos配置管理dataId 字段的一部分。
> >
> > 在 Nacos Spring Cloud中,dataId的完整格式如下：
> >
> > ```
> > ${prefix}-${spring-profile.active}.${file-extension}
> > ```
> >
> > - prefix默认为spring.application.name的值，也可以通过配置项spring.cloud.nacos.config.prefix来配置。
> > - spring.profile.active即为当前环境对应的 profile，详情可以参考 Spring Boot文档。注意：当spring.profile.active为空时，对应的连接符 - 也将不存在，datald 的拼接格式变成${prefix}.${file-extension}
> > - file-exetension为配置内容的数据格式，可以通过配置项spring .cloud.nacos.config.file-extension来配置。目前只支持properties和yaml类型。
> > - 通过Spring Cloud 原生注解@RefreshScope实现配置自动更新。

1. 搭建nacos环境

2. 在商品微服务中引入nacos的依赖

   ```xml
   <dependency>
     <groupId>com.alibaba.cloud</groupId>
     <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId> 
   </dependency>
   ```

3. 在微服务中添加nacos config的配置

   **注意**:不能使用原来的`application.yml`作为配置文件，而是新建一个`bootstrap.yml`作为配置文件

   ```
   配置文件优先级(由高到低):
   bootstrap.properties -> bootstrap.yml -> application.properties -> application.yml
   ```

   ```yaml
   spring:
     application:
       name: product-service
     cloud:
       nacos:
         config:
           server-addr: 127.0.0.1:8848 #nacos中心地址
           file-extension: yaml # 配置文件格式
     profiles: 
       active: dev # 环境标识
   ```

4. 在nacos中添加配置,然后把商品微服务application.yml配置复制到配置内容中.

   **![image-20201102093026806](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/image-20201102093026806.png)**

   ![image-20201102095324857](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/image-20201102095324857.png)

5. 注释本地的application.yam中的内容， 启动程序进行测试

6. 如果依旧可以成功访问程序，说明我们nacos的配置中心功能已经实现

#### 完整配置

> DataID+Group+Namespace确定读取哪个配置文件
>
> dataId的完整格式:${prefix}-${spring-profile.active}.${file-extension}

```yaml
spring:
  application:
    name: product-service
  cloud:
    nacos:
      config:
        server-addr: 127.0.0.1:8848 #nacos中心地址
        file-extension: yaml # 配置文件格式
#        namespace:
#        group:
  profiles:
    active: dev # 环境标识
```

#### 动态刷新

> 在controller上加注解@RefreshScope

#### 配置共享

##### 同一微服务配置共享

> 只需要提取一个以 spring.application.name 命名的配置文件，然后将其所有环境的公共配置放在里面即可
>
> 不加「-开发环境」就可以

##### 不同微服务配置共享

不同为服务之间实现配置共享的原理类似于文件引入，就是定义一个公共配置，然后在当前配置中引入。

1. 在nacos中定义一个DataID为global-config.yaml的配置，用于所有微服务共享

   ```yaml
   globalConfig: global
   ```

   **![image-20201102103412474](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/image-20201102103412474.png)**

2. 修改bootstrap.yaml

   ```yaml
   spring:
     application:
       name: product-service
     cloud:
       nacos:
         config:
           server-addr: 127.0.0.1:8848 #nacos中心地址
           file-cextension: yaml # 配置文件格式
           shared-configs:
             - data-id: global-config.yaml # 配置要引入的配置
               refresh: true
     profiles:
       active: test # 环境标识
   ```

3. 在NacosConfigController.java中新增一个方法

   ```java
   @RestController
   @RefreshScope
   public class NacosConfigController {
       @Value("${appConfig.name}")
       private String appConfigName;
       @Value("${env}")
       private String env;
       @Value("${globalConfig}")
       private String globalConfig;
       @RequestMapping("/nacosConfig1")
       public String nacosConfig(){
           return "远程信息:"+appConfigName;
       }
       @RequestMapping("/nacosConfig2")
       public String nacosConfig2(){
           return "公共配置:"+appConfigName+",环境配置信息:"+env;
       }
       @RequestMapping("/nacosConfig3")
       public String nacosConfig3(){
           return "全局配置:"+globalConfig+",公共配置:"+appConfigName+",环境配置信息:"+env;
       }
   }
   ```

4. 重启服务并测试.



### Nacos持久化

1. nacos-server-1.1.4\nacos\conf录下找到nacos-mysql.sql文件，执行脚本。

2. nacos-server-1.1.4\nacos\conf目录下找到application.properties，添加以下配置（按需修改对应值）。

   ```properties
   spring.datasource.platform=mysql
   
   db.num=1
   db.url.0=jdbc:mysql://localhost:3306/nacos_devtest?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true
   db.user=root
   db.password=1234
   ```

3. 重启



### Nacos集群

> 必须持久化

![681c3dc16a69f197896cbff482f2298e](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/681c3dc16a69f197896cbff482f2298e.png)

1. 配置**cluster.conf**

   ```
   # 三个服务器加端口
   192.168.111.144:3333
   192.168.111.144:4444
   192.168.111.144:5555
   ```

2. 分别启动三个nacos

3. 配置nginx作为负载均衡器

   ```conf
   upstream cluster{
   	server 127.0.0.1:3333;
   	server 127.0.0.1:4444;
   	server 127.0.0.1:5555;
   }
   server {
   	listen			1111;
   	server_name 127.0.0.1;
   	
   	location / {
   			proxy_pass http://cluster;
   	}
   }
   ```

4. 启动2222 3333 4444 nginx

5. 测试

   1. 测试通过nginx，访问nacos - http://127.0.0.1:1111/nacos/#/login
   2. 修改配置看是否会同步？会

6. 让微服务cloudalibaba-provider-payment9002启动注册进nacos集群 - 修改配置文件

   ```yml
   server:
     port: 9002
   
   spring:
     application:
       name: nacos-payment-provider
     c1oud:
       nacos:
         discovery:
           #配置Nacos地址
           #server-addr: Localhost:8848
           #换成nginx的1111端口，做集群
           server-addr: 192.168.111.144:1111
   
   management:
     endpoints:
       web:
         exposure:
           inc1ude: '*'
   ```

   启动微服务cloudalibaba-provider-payment9002

   访问nacos，查看注册结果

7. 总结

   

![42ff7ef670012437b046f099192d7484](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/42ff7ef670012437b046f099192d7484.png)



## Ribbon 负载均衡

### 配置负载均衡策略

com.netflix.loadbalancer.IRule , 具体的负载策略如下图所示:

| 策略名                    | 策略描述                                                     | 实现说明                                                     |
| ------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| AvailabilityFilteringRule | 先过滤掉故障实例，再选择并发较小的实例；                     | 使用一个AvailabilityPredicate来包含过滤server的逻辑，其实就就是检查status里记录的各个server的运行状态 |
| WeightedResponseTimeRule  | 根据相应时间分配一个weight，相应时间越长，weight越小，被选中的可能性越低。 | 一个后台线程定期的从status里面读取评价响应时间，为每个server计算一个weight。Weight的计算也比较简单responsetime 减去每个server自己平均的responsetime是server的权 |
| RetryRule                 | 对选定的负载均衡策略机上重试机制。                           | 在一个配置时间段内当选择server不成功，则一直尝试使用subRule的方式选择一个可用的server |
| RoundRobinRule            | 轮询方式轮询选择server                                       | 轮询index，选择index对应位置的server                         |
| RandomRule                | 随机选择一个server                                           | 在index上随机，选择index对应位置的server                     |
| BestAvailableRule         | 选择一个最小的并发请求的server                               | 逐个考察Server，如果Server被tripped了，则忽略，在选择其中ActiveRequestsCount最小的server |
| ZoneAvoidanceRule(默认)   | 复合判断server所在区域的性能和server的可用性选择server       | 使用ZoneAvoidancePredicate和AvailabilityPredicate来判断是否选择某个server，前一个判断判定一个zone的运行性能是否可用，剔除不可用的zone（的所有server），AvailabilityPredicate用于过滤掉连接数过多的Server。 |

我们可以通过修改配置来调整Ribbon的负载均衡策略，在application.yml中增加如下配置:

```yaml
<调用的服务名>: # 调用的提供者的名称
  ribbon:
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule
```

## OpenFeign 远程调用

### 简单使用

1. pom

   ```xml
   <!--fegin组件-->
   <dependency>
   	<groupId>org.springframework.cloud</groupId>
   	<artifactId>spring-cloud-starter-openfeign</artifactId>
   </dependency>
   ```

2. 启动类 @EnableFeignClients

   ```java
   @SpringBootApplication
   @EnableDiscoveryClient
   @EnableFeignClients
   public class OrderServer {
       public static void main(String[] args) {
           SpringApplication.run(OrderServer.class,args);
       }
   }
   ```

3. 使用

   > 直接复制对应微服务的controller 
   >
   > 注意⚠️：  
   >
   > - 请求路径注意复制完整
   > - 每个参数都需要注解

   ```java
   @Component
   @FeignClient(value = "cloud-payment-service")
   public interface PaymentFeignService {
   		 /**
        * 方法的参数必须有注解
        * @PathVariable 路径变量
        * @RequestParam 参数
        * @RequestBody  参数对象
        *
        * @param pid
        * @return
        */
       @GetMapping(value = "/payment/get/{id}")
       public CommonResult getPamentById(@PathVariable("id")Long id);
   }
   ```

### OpenFeign超时控制

> **OpenFeign默认等待1秒钟，超过后报错**

yml配置

```yml
#设置feign客户端超时时间(OpenFeign默认支持ribbon)(单位：毫秒)
ribbon:
  #指的是建立连接所用的时间，适用于网络状况正常的情况下,两端连接所用的时间
  ReadTimeout: 5000
  #指的是建立连接后从服务器读取到可用资源所用的时间
  ConnectTimeout: 5000
```

### OpenFeign日志增强

1. 配置日志bean

   - NONE：默认的，不显示任何日志;
   - BASIC：仅记录请求方法、URL、响应状态码及执行时间;
   - HEADERS：除了BASIC中定义的信息之外，还有请求和响应的头信息;
   - FULL：除了HEADERS中定义的信息之外，还有请求和响应的正文及元数据。

   ```java
   package com.atguigu.springcloud.config;
   
   import feign.Logger;
   import org.springframework.context.annotation.Bean;
   import org.springframework.context.annotation.Configuration;
   
   @Configuration
   public class FeignConfig
   {
       @Bean
       Logger.Level feignLoggerLevel()
       {
           //NONE：默认的，不显示任何日志;
           //BASIC：仅记录请求方法、URL、响应状态码及执行时间;
           //HEADERS：除了BASIC中定义的信息之外，还有请求和响应的头信息;
           //FULL：除了HEADERS中定义的信息之外，还有请求和响应的正文及元数据。
           return Logger.Level.FULL;
       }
   }
   
   ```

2. **YML文件里需要开启日志的Feign客户端**

   ```yml
   logging:
     level:
       # feign日志以什么级别监控哪个接口
       com.lun.springcloud.service.PaymentFeignService: debug
   ```

### 集成Sentinel,服务降级处理

1. yaml配置

   ```yaml
   product-service: # 调用的提供者的名称
     ribbon:
       NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule
   ```

2. 使用

   ![截屏2021-09-06 17.16.23](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/%E6%88%AA%E5%B1%8F2021-09-06%2017.16.23.png)

   - ProductFeignApi

     ```java
     @FeignClient(name = "product-service",fallback = ProductFeignFallBack.class)
     public interface ProductFeignApi {
         @RequestMapping("/product/{pid}")
         public Product findByPid( @PathVariable("pid") Long pid);
     }
     ```

   - ProductFeignFallBack

     ```java
     @Component
     public class ProductFeignFallBack implements ProductFeignApi {
         @Override
         public Product findByPid(Long pid) {
             Product product = new Product();
             product.setPid(-1L);
             product.setPname("兜底数据");
             product.setPprice(0.0);
             return product;
         }
     }
     ```

     

## Sentinel 服务保护

### 订单微服务集成Sentinel

为微服务集成Sentinel非常简单, 只需要加入Sentinel的依赖即可

在shop-order-server项目的pom文件中添加如下依赖

```xml
<!--sentinel组件-->
<dependency>
	<groupId>com.alibaba.cloud</groupId>
	<artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
</dependency>
```

### 安装Sentinel控制台

Sentinel 提供一个轻量级的控制台, 它提供机器发现、单机资源实时监控以及规则管理等功能。

1. 下载jar包 https://github.com/alibaba/Sentinel/releases

2.  启动控制台

    ```shell
    # 直接使用jar命令启动项目(控制台本身是一个SpringBoot项目) 
    java -Dserver.port=8080 -Dcsp.sentinel.dashboard.server=localhost:8080 -Dproject.name=sentinel-dashboard -jar sentinel-dashboard-1.8.0.jar
    ```

3. 修改shop-order-server项目中的配置文件application.yml,新增如下配置:

   ```yaml
   spring:
     cloud:
       sentinel: 
         transport: 
           port: 9999 #跟控制台交流的端口,随意指定一个未使用的端口即可 
           dashboard: localhost:8080 # 指定控制台服务的地址
   ```

4. 通过浏览器访问localhost:8080 进入控制台 ( 默认用户名密码是 sentinel/sentinel )

   注意: 默认是没显示order-service的，需要访问几次接口，然后再刷新sentinel管控台才可以看到.

   **![image-20201029151420936](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/image-20201029151420936.png)**

### 实现一个接口的限流

第一步: 簇点链路--->流控

**![image-20201029151937258](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/image-20201029151937258.png)**

第二步: 在单机阈值填写一个数值，表示每秒上限的请求数

**![image-20201029152031334](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/image-20201029152031334.png)**

第三步:通过控制台快速频繁访问, 观察效果



**![image-20201029152401508](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/image-20201029152401508.png)**

### Sentinel容错的维度

**![image-20201029153848900](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/image-20201029153848900.png)**

**流量控制**：流量控制在网络传输中是一个常用的概念，它用于调整网络包的数据。任意时间到来的请求往往是

随机不可控的，而系统的处理能力是有限的。我们需要根据系统的处理能力对流量进行控制。

**熔断降级**：当检测到调用链路中某个资源出现不稳定的表现，例如请求响应时间长或异常比例升高的时候，则

对这个资源的调用进行限制，让请求快速失败，避免影响到其它的资源而导致级联故障。

**系统负载保护**：Sentinel 同时提供系统维度的自适应保护能力。当系统负载较高的时候，如果还持续让

请求进入可能会导致系统崩溃，无法响应。在集群环境下，会把本应这台机器承载的流量转发到其

它的机器上去。如果这个时候其它的机器也处在一个边缘状态的时候，Sentinel 提供了对应的保

护机制，让系统的入口流量和系统的负载达到一个平衡，保证系统在能力范围之内处理最多的请

求。

### Sentinel规则种类

Sentinel主要提供了这五种的流量控制，接下来我们每种都给同学们演示一下.

**![image-20201029160041081](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/image-20201029160041081.png)**

### Sentinel规则-流控

#### 流控规则

流量控制，其原理是监控应用流量的QPS(每秒查询率) 或并发线程数等指标，当达到指定的阈值时

对流量进行控制，以避免被瞬时的流量高峰冲垮，从而保障应用的高可用性。

**![image-20201029160919062](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/image-20201029160919062.png)**

**资源名**：唯一名称，默认是请求路径，可自定义

**针对来源**：指定对哪个微服务进行限流，默认指default，意思是不区分来源，全部限制

**阈值类型/单机阈值**：

- QPS（每秒请求数量）: 当调用该接口的QPS达到阈值的时候，进行限流

- 线程数：当调用该接口的线程数达到阈值的时候，进行限流

**是否集群**：暂不需要集群



####  QPS流控

前面演示的QPS流控

#### 线程数流控

1. 删除掉之前的QPS流控，新增线程数流控

   **![image-20201029162926422](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/image-20201029162926422.png)**

2. 在Jmeter中新增线程

   **![image-20201029163205306](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/image-20201029163205306.png)**

   **![image-20201029163233714](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/image-20201029163233714.png)**

3. 访问 localhost:8091/sentinel2 会发现已经被限流

   **![image-20201029163416823](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/image-20201029163416823.png)**

#### 流控模式

点击上面设置流控规则的**编辑**按钮，然后在编辑页面点击**高级选项**，会看到有流控模式一栏。

**![image-20201029164212597](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/image-20201029164212597.png)**

sentinel共有三种流控模式，分别是：

- 直接（默认）：接口达到限流条件时，开启限流

- 关联：当关联的资源达到限流条件时，开启限流 [适合做应用让步]

- 链路：当从某个接口过来的资源达到限流条件时，开启限流

##### 直接流控模式

前面演示的案例就是这种.

##### 关联流控模式

关联流控模式指的是，当指定接口关联的接口达到限流条件时，开启对指定接口开启限流。

**场景**:当两个资源之间具有资源争抢或者依赖关系的时候，这两个资源便具有了关联。比如对数据库同一个字段的读操作和写操作存在争抢，读的速度过高会影响写得速度，写的速度过高会影响读的速度。如果放任读写操作争抢资源，则争抢本身带来的开销会降低整体的吞吐量。可使用关联限流来避免具有关联关系的资源之间过度的争抢.

1. 在SentinelController.java中增加一个方法，重启订单服务

   ```java
   @RequestMapping("/sentinel3")
   public String sentinel3(){
   	return "sentinel3";
   }
   ```

2. 配置限流规则, 将流控模式设置为关联，关联资源设置为的 /sentinel2

   **![image-20201029170800980](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/image-20201029170800980.png)**

3. 通过postman软件向/sentinel2连续发送请求，注意QPS一定要大于3

   **![image-20201029171026368](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/image-20201029171026368.png)**

   **![image-20201029171055757](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/image-20201029171055757.png)**

4. 访问/sentinel3,会发现已经被限流

   **![image-20201029171402061](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/image-20201029171402061.png)**

##### 链路流控模式

链路流控模式指的是，当从某个接口过来的资源达到限流条件时，开启限流。

1. 在shop-order-server项目的application.yml文件中新增如下配置:

   ```yaml
   spring:
     cloud:
       sentinel:
         web-context-unify: false
   ```

2. 在shop-order-server项目中新增TraceServiceImpl.java

   ```java
   package cn.wolfcode.service.impl;
   @Service
   @Slf4j
   public class TraceServiceImpl {
       @SentinelResource(value = "tranceService")
       public void tranceService(){
           log.info("调用tranceService方法");
       }
   }
   ```

3. 在shop-order-server项目中新增TraceController.java

   ```java
   package cn.wolfcode.controller;
   @RestController
   public class TraceController {
       @Autowired
       private TraceServiceImpl traceService;
       @RequestMapping("/trace1")
       public String trace1(){
           traceService.tranceService();
           return "trace1";
       }
       @RequestMapping("/trace2")
       public String trace2(){
           traceService.tranceService();
           return "trace2";
       }
   }
   ```

4. 重新启动订单服务并添加链路流控规则

   **![image-20201029175454431](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/image-20201029175454431.png)**

5.  分别通过 /trace1 和 /trace2 访问, 发现/trace1没问题, /trace2的被限流了

    **![image-20201029175555499](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/image-20201029175559035.png)**

#### 流控效果

- **快速失败（默认）**: 直接失败，抛出异常，不做任何额外的处理，是最简单的效果

- **Warm Up**：它从开始阈值到最大QPS阈值会有一个缓冲阶段，一开始的阈值是最大QPS阈值的

  1/3，然后慢慢增长，直到最大阈值，适用于将突然增大的流量转换为缓步增长的场景。

- **排队等待**：让请求以均匀的速度通过，单机阈值为每秒通过数量，其余的排队等待； 它还会让设

  置一个超时时间，当请求超过超时间时间还未处理，则会被丢弃。



### Sentinel规则-降级

降级规则就是设置当满足什么条件的时候，对服务进行降级。Sentinel提供了三个衡量条件：

- **慢调用比例**: 选择以慢调用比例作为阈值，需要设置允许的慢调用 RT（即最大的响应时间），请求的响应时间大于该值则统计为慢调用。当单位统计时长内请求数目大于设置的最小请求数目，并且慢调用的比例大于阈值，则接下来的熔断时长内请求会自动被熔断。经过熔断时长后熔断器会进入探测恢复状态（HALF-OPEN 状态），若接下来的一个请求响应时间小于设置的慢调用 RT 则结束熔断，若大于设置的慢调用 RT 则会再次被熔断。
- **异常比例**: 当单位统计时长内请求数目大于设置的最小请求数目，并且异常的比例大于阈值，则接下来的熔断时长内请求会自动被熔断。经过熔断时长后熔断器会进入探测恢复状态（HALF-OPEN 状态），若接下来的一个请求成功完成（没有错误）则结束熔断，否则会再次被熔断。异常比率的阈值范围是 `[0.0, 1.0]`，代表 0% - 100%。
- **异常数**：当单位统计时长内的异常数目超过阈值之后会自动进行熔断。经过熔断时长后熔断器会进入探测恢复状态（HALF-OPEN 状态），若接下来的一个请求成功完成（没有错误）则结束熔断，否则会再次被熔断。

#### 慢调用比例案例

1. 在shop-order-server项目中新增FallBackController.java类,代码如下:

   ```java
   package cn.wolfcode.controller;
   @RestController
   @Slf4j
   public class FallBackController {
       @RequestMapping("/fallBack1")
       public String fallBack1(){
           try {
               log.info("fallBack1执行业务逻辑");
               //模拟业务耗时
               TimeUnit.SECONDS.sleep(1);
           } catch (InterruptedException e) {
               e.printStackTrace();
           }
           return "fallBack1";
       }
   }
   ```

2. 新增降级规则:

   **![image-20201030094205307](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/image-20201030094205307.png)**

   上面配置表示，如果在1S之内,有【超过1个的请求】且这些请求中【响应时间>最大RT】的【请求数量比例>10%】，就会触发熔断，在接下来的10s之内都不会调用真实方法，直接走降级方法。

   比如: 最大RT=900,比例阈值=0.1,熔断时长=10,最小请求数=10

   - 情况1: 1秒内的有20个请求，只有10个请求响应时间>900ms, 那慢调用比例=0.5，这种情况就会触发熔断

   - 情况2: 1秒内的有20个请求，只有1个请求响应时间>900ms, 那慢调用比例=0.05，这种情况不会触发熔断
   - 情况3: 1秒内的有8个请求，只有6个请求响应时间>900ms, 那慢调用比例=0.75，这种情况不会触发熔断，因为最小请求数这个条件没有满足.

   **注意**: 我们做实验的时候把最小请求数设置为1，因为在1秒内，手动操作很难在1s内发两个请求过去，所以要做出效果,最好把最小请求数设置为1。

#### 异常比例案例

1. 在shop-order-server项目的FallBackController.java类新增fallBack2方法:

   ```java
   int i=0;
   @RequestMapping("/fallBack2")
   public String fallBack2(){
   	log.info("fallBack2执行业务逻辑");
   	//模拟出现异常，异常比例为33%
       if(++i%3==0){
   		throw new RuntimeException();
   	}
   	return "fallBack2";
   }
   ```

2. 新增降级规则:

   **![image-20201030100912417](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/image-20201030100912417.png)**

   上面配置表示，在1s之内，,有【超过3个的请求】，异常比例30%的情况下，触发熔断，熔断时长为10s.

#### 异常数案例

1. 在shop-order-server项目的FallBackController.java类新增fallBack3方法:

   ```java
   @RequestMapping("/fallBack3")
   public String fallBack3(String name){
   	log.info("fallBack3执行业务逻辑");
   	if("wolfcode".equals(name)){
   		throw new RuntimeException();
   	}
   	return "fallBack3";
   }
   ```

2. 新增降级规则

   **![image-20201030102109574](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/image-20201030102109574.png)**

   上面配置表示，在1s之内，,有【超过3个的请求】，请求中超过2个请求出现异常就会触发熔断，熔断时长为10s

### Sentinel规则-热点

何为热点？热点即经常访问的数据。很多时候我们希望统计某个热点数据中访问频次最高的 Top K 数据，并对其访问进行限制。比如：

- 商品 ID 为参数，统计一段时间内最常购买的商品 ID 并进行限制
- 用户 ID 为参数，针对一段时间内频繁访问的用户 ID 进行限制

热点参数限流会统计传入参数中的热点参数，并根据配置的限流阈值与模式，对包含热点参数的资源调用进行限流。热点参数限流可以看做是一种特殊的流量控制，仅对包含热点参数的资源调用生效。

1. 在shop-order-server项目中新增HotSpotController.java,代码如下:

   ```java
   package cn.wolfcode.controller;
   @RestController
   @Slf4j
   public class HotSpotController {
       @RequestMapping("/hotSpot1")
       @SentinelResource(value = "hotSpot1")
       public String hotSpot1(Long productId){
           log.info("访问编号为:{}的商品",productId);
           return "hotSpot1";
       }
   }
   ```

   注意:一定需要在请求方法上贴@SentinelResource直接，否则热点规则无效

2. 新增热点规则:

   **![image-20201030104511784](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/image-20201030104511784.png)**

3. 在热点规则中编辑规则,在编辑之前一定要先访问一下/hotSpot1,不然参数规则无法新增.

   **![image-20201030104633745](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/image-20201030104633745.png)**

4. 新增参数规则:

   ![1625630523922](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/1625630523922.png)

5. 点击保存，可以看到已经新增了参数规则.

   **![image-20201030104957789](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/image-20201030104957789.png)**

6. 访问localhost:8091/hotSpot?productId=1 访问会降级

   访问localhost:8091/hotSpot?productId=2 访问不会降级

   **![image-20201030105114878](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/image-20201030105114878.png)**

### Sentinel规则-授权

很多时候，我们需要根据调用来源来判断该次请求是否允许放行，这时候可以使用 Sentinel 的来源访问控制（黑白名单控制）的功能。来源访问控制根据资源的请求来源（`origin`）限制资源是否通过，若配置白名单则只有请求来源位于白名单内时才可通过；若配置黑名单则请求来源位于黑名单时不通过，其余的请求通过。

1. 在shop-order-server中新建RequestOriginParserDefinition.java,定义请求来源如何获取

   ```java
   @Component
   public class RequestOriginParserDefinition implements RequestOriginParser {
       @Override
       public String parseOrigin(HttpServletRequest request) {
           /**
            *  定义从请求的什么地方获取来源信息
            *  比如我们可以要求所有的客户端需要在请求头中携带来源信息
            */
           String serviceName = request.getParameter("serviceName");
           return serviceName;
       }
   }
   ```

2. 在shop-order-server中新建AuthController.java,代码如下:

   ```java
   @RestController
   @Slf4j
   public class AuthController {
       @RequestMapping("/auth1")
       public String auth1(String serviceName){
           log.info("应用:{},访问接口",serviceName);
           return "auth1";
       }
   }
   ```

3. 新增授权规则

   **![image-20201030110821985](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/image-20201030110821985.png)**

4. 访问测试

   访问localhost:8091/auth1?serviceName=pc 不能访问

   访问localhost:8091/auth1?serviceName=app 可以访问

   **![image-20201030110934370](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/image-20201030110934370.png)**

###  Sentinel规则-系统规则

系统保护规则是从应用级别的入口流量进行控制，从单台机器的 load、CPU 使用率、平均 RT、入口 QPS 和并发线程数等几个维度监控应用指标，让系统尽可能跑在最大吞吐量的同时保证系统整体的稳定性。

系统保护规则是应用整体维度的，而不是资源维度的，并且**仅对入口流量生效**。入口流量指的是进入应用的流量（`EntryType.IN`），比如 Web 服务或 Dubbo 服务端接收的请求，都属于入口流量。

系统规则支持以下的模式：

- **Load 自适应**（仅对 Linux/Unix-like 机器生效）：系统的 load 作为启发指标，进行自适应系统保护。当系统 load 超过设定的启发值，且系统当前的并发线程数超过估算的系统容量时才会触发系统保护（BBR 阶段）。系统容量由系统的 `maxQps * minRt` 估算得出。设定参考值一般是 `CPU cores * 2.5`。
- **CPU usage**（1.5.0+ 版本）：当系统 CPU 使用率超过阈值即触发系统保护（取值范围 0.0-1.0），比较灵敏。
- **平均 RT**：当单台机器上所有入口流量的平均 RT 达到阈值即触发系统保护，单位是毫秒。
- **并发线程数**：当单台机器上所有入口流量的并发线程数达到阈值即触发系统保护。
- **入口 QPS**：当单台机器上所有入口流量的 QPS 达到阈值即触发系统保护。

**![image-20201030111407594](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/image-20201030111407594.png)**



### Sentinel 自定义异常返回

当前面设定的规则没有满足是，我们可以自定义异常返回.

-  FlowException 限流异常 

-  DegradeException 降级异常 

-  ParamFlowException 参数限流异常 

-  AuthorityException 授权异常 

-  SystemBlockException 系统负载异常

在shop-order-server项目中定义异常返回处理类

```java
package cn.wolfcode.error;
@Component
public class ExceptionHandlerPage implements BlockExceptionHandler {
    @Override
    public void handle(HttpServletRequest request, HttpServletResponse response, BlockException e) throws Exception {
        response.setContentType("application/json;charset=utf-8");
        ResultData data = null;
        if (e instanceof FlowException) {
            data = new ResultData(-1, "接口被限流了");
        } else if (e instanceof DegradeException) {
            data = new ResultData(-2, "接口被降级了");
        }else if (e instanceof ParamFlowException) {
            data = new ResultData(-3, "参数限流异常");
        }else if (e instanceof AuthorityException) {
            data = new ResultData(-4, "授权异常");
        }else if (e instanceof SystemBlockException) {
            data = new ResultData(-5, "接口被降级了...");
        }
        response.getWriter().write(JSON.toJSONString(data));
    }
}
@Data
@AllArgsConstructor//全参构造
@NoArgsConstructor//无参构造
class ResultData {
    private int code;
    private String message;
}
```



###  @SentinelResource的使用

在定义了资源点之后，我们可以通过Dashboard来设置限流和降级策略来对资源点进行保护。同时还能

通过@SentinelResource来指定出现异常时的处理策略。

@SentinelResource 用于定义资源，并提供可选的异常处理和 fallback 配置项。

其主要参数如下:

| 属性                               | 作用                                                         |
| :--------------------------------- | ------------------------------------------------------------ |
| value                              | 资源名称，必需项（不能为空）                                 |
| entryType                          | entry 类型，可选项（默认为 `EntryType.OUT`）                 |
| blockHandler` / `blockHandlerClass | `blockHandler` 对应处理 `BlockException` 的函数名称，可选项。blockHandler 函数访问范围需要是 `public`，返回类型需要与原方法相匹配，参数类型需要和原方法相匹配并且最后加一个额外的参数，类型为 `BlockException`。blockHandler 函数默认需要和原方法在同一个类中。若希望使用其他类的函数，则可以指定 `blockHandlerClass` 为对应的类的 `Class` 对象，注意对应的函数必需为 static 函数，否则无法解析。 |
| fallback` / `fallbackClass         | fallback 函数名称，可选项，用于在抛出异常的时候提供 fallback 处理逻辑。fallback 函数可以针对所有类型的异常（除了 `exceptionsToIgnore` 里面排除掉的异常类型）进行处理。fallback 函数签名和位置要求：<br />1. 返回值类型必须与原函数返回值类型一致； <br />2.方法参数列表需要和原函数一致，或者可以额外多一个 `Throwable` 类型的参数用于接收对应的异常。<br />3.fallback 函数默认需要和原方法在同一个类中。若希望使用其他类的函数，则可以指定 `fallbackClass` 为对应的类的 `Class` 对象，注意对应的函数必需为 static 函数，否则无法解析。 |
| `defaultFallback`                  | 默认的 fallback 函数名称，可选项，通常用于通用的 fallback 逻辑（即可以用于很多服务或方法）。默认 fallback 函数可以针对所有类型的异常（除了 `exceptionsToIgnore` 里面排除掉的异常类型）进行处理。若同时配置了 fallback 和 defaultFallback，则只有 fallback 会生效。defaultFallback 函数签名要求：<br />1. 返回值类型必须与原函数返回值类型一致；<br />2. 方法参数列表需要为空，或者可以额外多一个 `Throwable` 类型的参数用于接收对应的异常。 <br />3. defaultFallback 函数默认需要和原方法在同一个类中。若希望使用其他类的函数，则可以指定 `fallbackClass` 为对应的类的 `Class` 对象，注意对应的函数必需为 static 函数，否则无法解析。 |
| `exceptionsToIgnore`               | 用于指定哪些异常被排除掉，不会计入异常统计中，也不会进入 fallback 逻辑中，而是会原样抛出。 |

**定义限流和降级后的处理方法**

直接将限流和降级方法定义在方法中

```java
package cn.wolfcode.controller;
@RestController
@Slf4j
public class AnnoController {
    @RequestMapping("/anno1")
    @SentinelResource(value = "anno1",
            blockHandler="anno1BlockHandler",
            fallback = "anno1Fallback"
    )
    public String anno1(String name){
        if("wolfcode".equals(name)){
            throw new RuntimeException();
        }
        return "anno1";
    }
    public String anno1BlockHandler(String name,BlockException ex){
        log.error("{}", ex);
        return "接口被限流或者降级了";
    }
    //Throwable时进入的方法
    public String anno1Fallback(String name,Throwable throwable) {
        log.error("{}", throwable);
        return "接口发生异常了";
    }
}
```

### Sentinel规则持久化

#### 推模式(基于文件)

​	通过前面的讲解，我们已经知道，可以通过Dashboard来为每个Sentinel客户端设置各种各样的规

则，但是这里有一个问题，就是这些规则默认是存放在内存中，极不稳定，所以需要将其持久化。

​	本地文件数据源会定时轮询文件的变更，读取规则。这样我们既可以在应用本地直接修改文件来更

新规则，也可以通过 Sentinel 控制台推送规则。以本地文件数据源为例，推送过程如下图所示：

**![image-20201030135911029](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/image-20201030135911029.png)**

首先 Sentinel 控制台通过 API 将规则推送至客户端并更新到内存中，接着注册的写数据源会将新的

规则保存到本地的文件中。

1. 编写处理类

   ```java
   package cn.wolfcode.sentinel;
   public class FilePersistence implements InitFunc {
       @Value("${spring.application.name}")
       private String appcationName;
   
       @Override
       public void init() throws Exception {
           String ruleDir = System.getProperty("user.home") + "/sentinel-rules/" + appcationName;
           String flowRulePath = ruleDir + "/flow-rule.json";
           String degradeRulePath = ruleDir + "/degrade-rule.json";
           String systemRulePath = ruleDir + "/system-rule.json";
           String authorityRulePath = ruleDir + "/authority-rule.json";
           String paramFlowRulePath = ruleDir + "/param-flow-rule.json";
   
           this.mkdirIfNotExits(ruleDir);
           this.createFileIfNotExits(flowRulePath);
           this.createFileIfNotExits(degradeRulePath);
           this.createFileIfNotExits(systemRulePath);
           this.createFileIfNotExits(authorityRulePath);
           this.createFileIfNotExits(paramFlowRulePath);
   
           // 流控规则
           ReadableDataSource<String, List<FlowRule>> flowRuleRDS = new FileRefreshableDataSource<>(
                   flowRulePath,
                   flowRuleListParser
           );
           FlowRuleManager.register2Property(flowRuleRDS.getProperty());
           WritableDataSource<List<FlowRule>> flowRuleWDS = new FileWritableDataSource<>(
                   flowRulePath,
                   this::encodeJson
           );
           WritableDataSourceRegistry.registerFlowDataSource(flowRuleWDS);
   
           // 降级规则
           ReadableDataSource<String, List<DegradeRule>> degradeRuleRDS = new FileRefreshableDataSource<>(
                   degradeRulePath,
                   degradeRuleListParser
           );
           DegradeRuleManager.register2Property(degradeRuleRDS.getProperty());
           WritableDataSource<List<DegradeRule>> degradeRuleWDS = new FileWritableDataSource<>(
                   degradeRulePath,
                   this::encodeJson
           );
           WritableDataSourceRegistry.registerDegradeDataSource(degradeRuleWDS);
   
           // 系统规则
           ReadableDataSource<String, List<SystemRule>> systemRuleRDS = new FileRefreshableDataSource<>(
                   systemRulePath,
                   systemRuleListParser
           );
           SystemRuleManager.register2Property(systemRuleRDS.getProperty());
           WritableDataSource<List<SystemRule>> systemRuleWDS = new FileWritableDataSource<>(
                   systemRulePath,
                   this::encodeJson
           );
           WritableDataSourceRegistry.registerSystemDataSource(systemRuleWDS);
   
           // 授权规则
           ReadableDataSource<String, List<AuthorityRule>> authorityRuleRDS = new FileRefreshableDataSource<>(
                   authorityRulePath,
                   authorityRuleListParser
           );
           AuthorityRuleManager.register2Property(authorityRuleRDS.getProperty());
           WritableDataSource<List<AuthorityRule>> authorityRuleWDS = new FileWritableDataSource<>(
                   authorityRulePath,
                   this::encodeJson
           );
           WritableDataSourceRegistry.registerAuthorityDataSource(authorityRuleWDS);
   
           // 热点参数规则
           ReadableDataSource<String, List<ParamFlowRule>> paramFlowRuleRDS = new FileRefreshableDataSource<>(
                   paramFlowRulePath,
                   paramFlowRuleListParser
           );
           ParamFlowRuleManager.register2Property(paramFlowRuleRDS.getProperty());
           WritableDataSource<List<ParamFlowRule>> paramFlowRuleWDS = new FileWritableDataSource<>(
                   paramFlowRulePath,
                   this::encodeJson
           );
           ModifyParamFlowRulesCommandHandler.setWritableDataSource(paramFlowRuleWDS);
       }
   
       private Converter<String, List<FlowRule>> flowRuleListParser = source -> JSON.parseObject(
               source,
               new TypeReference<List<FlowRule>>() {
               }
       );
       private Converter<String, List<DegradeRule>> degradeRuleListParser = source -> JSON.parseObject(
               source,
               new TypeReference<List<DegradeRule>>() {
               }
       );
       private Converter<String, List<SystemRule>> systemRuleListParser = source -> JSON.parseObject(
               source,
               new TypeReference<List<SystemRule>>() {
               }
       );
   
       private Converter<String, List<AuthorityRule>> authorityRuleListParser = source -> JSON.parseObject(
               source,
               new TypeReference<List<AuthorityRule>>() {
               }
       );
   
       private Converter<String, List<ParamFlowRule>> paramFlowRuleListParser = source -> JSON.parseObject(
               source,
               new TypeReference<List<ParamFlowRule>>() {
               }
       );
   
       private void mkdirIfNotExits(String filePath) throws IOException {
           File file = new File(filePath);
           if (!file.exists()) {
               file.mkdirs();
           }
       }
   
       private void createFileIfNotExits(String filePath) throws IOException {
           File file = new File(filePath);
           if (!file.exists()) {
               file.createNewFile();
           }
       }
       private <T> String encodeJson(T t) {
           return JSON.toJSONString(t);
       }
   }
   ```

2. 添加配置

   在`resources`下创建配置目录 `META-INF/services` ,然后添加文件 `com.alibaba.csp.sentinel.init.InitFunc`

   在文件中添加配置类的全路径

   `cn.wolfcode.sentinel.FilePersistence`



#### 拉模式（nacos）

修改cloudalibaba-sentinel-service8401

POM

```xml
<!--SpringCloud ailibaba sentinel-datasource-nacos 后续做持久化用到-->
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-datasource-nacos</artifactId>
</dependency>
```


YML

```yml
server:
  port: 8401

spring:
  application:
    name: cloudalibaba-sentinel-service
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848 #Nacos服务注册中心地址
    sentinel:
      transport:
        dashboard: localhost:8080 #配置Sentinel dashboard地址
        port: 8719
      datasource: #<---------------------------关注点，添加Nacos数据源配置
        ds1:
          nacos:
            server-addr: localhost:8848
            dataId: cloudalibaba-sentinel-service
            groupId: DEFAULT_GROUP
            data-type: json
            rule-type: flow

management:
  endpoints:
    web:
      exposure:
        include: '*'

feign:
  sentinel:
    enabled: true # 激活Sentinel对Feign的支持
```


添加Nacos业务规则配置

![2401a6b2df715ee64f647da2f31e1eeb](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/2401a6b2df715ee64f647da2f31e1eeb.png)

配置内容解析

```json
[{
    "resource": "/rateLimit/byUrl",
    "IimitApp": "default",
    "grade": 1,
    "count": 1, 
    "strategy": 0,
    "controlBehavior": 0,
    "clusterMode": false
}]
```

- resource：资源名称；
- limitApp：来源应用；
- grade：阈值类型，0表示线程数, 1表示QPS；
- count：单机阈值；
- strategy：流控模式，0表示直接，1表示关联，2表示链路；
- controlBehavior：流控效果，0表示快速失败，1表示Warm Up，2表示排队等待；
- clusterMode：是否集群。

启动8401后刷新sentinel发现业务规则有了



快速访问测试接口 - localhost:8401/rateLimit/byUrl - 页面返回Blocked by Sentinel (flow limiting)

停止8401再看sentinel - 停机后发现流控规则没有了



重新启动8401再看sentinel

- 乍一看还是没有，稍等一会儿
- 多次调用 - localhost:8401/rateLimit/byUrl
- 重新配置出现了，持久化验证通过

## Gateway 网关

三大核心概念

- Route(路由) - 路由是构建网关的基本模块,它由ID,目标URI,一系列的断言和过滤器组成,如断言为true则匹配该路由；
- Predicate(断言) - 参考的是Java8的java.util.function.Predicate，开发人员可以匹配HTTP请求中的所有内容(例如请求头或请求参数),如果请求与断言相匹配则进行路由；
- Filter(过滤) - 指的是Spring框架中GatewayFilter的实例,使用过滤器,可以在请求被路由前或者之后对请求进行修改。

### 模块搭建

1. 新建模块

2. pom

   ```xml
           <!--gateway网关-->
           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-starter-gateway</artifactId>
           </dependency>
           <!--nacos客户端-->
           <dependency>
               <groupId>com.alibaba.cloud</groupId>
               <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
           </dependency>
   ```

3. yaml

   ```yaml
   server:
     port: 9000
   spring:
     application:
       name: api-gateway
     cloud:
       nacos:
         discovery:
           server-addr: localhost:8848
       gateway:
         discovery:
           locator:
             enabled: true # 让gateway可以发现nacos中的微服务
   ```

4. 启动类

   ```java
   @SpringBootApplication
   @EnableDiscoveryClient
   public class ApiGatewayServer {
       public static void main(String[] args) {
           SpringApplication.run(ApiGatewayServer.class,args);
       }
   }
   ```

5. 使用 

   ```
   http://ip地址:端口号/服务名/请求
   例如：
   localhost:9000/order-service/sentinel2
   ```

### 路由

默认情况下Gateway会根据注册中心注册的服务列表，以注册中心上微服务名为路径创建**动态路由进行转发，从而实现动态路由的功能**（不写死一个地址）。

```yaml
spring:
  application:
    name: api-gateway
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848
    gateway:
      discovery:
        locator:
          enabled: true # 让gateway可以发现nacos中的微服务
      routes:
        - id: product_route
          uri: lb://product-service  #匹配后提供服务的路由地址
          predicates:
            - Path=/product-serv/**
          filters:
            - StripPrefix=1
        - id: order_route
          uri: lb://order-service
          predicates:
            - Path=/order-serv/**
          filters:
            - StripPrefix=1
```

### 断言

> 常用的Route Predicate Factory
>
> The After Route Predicate Factory
> The Before Route Predicate Factory
> The Between Route Predicate Factory
> The Cookie Route Predicate Factory
> The Header Route Predicate Factory
> The Host Route Predicate Factory
> The Method Route Predicate Factory
> The Path Route Predicate Factory
> The Query Route Predicate Factory
> The RemoteAddr Route Predicate Factory
> The weight Route Predicate Factory

```yml
spring:
  application:
    name: api-gateway
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848
    gateway:
      discovery:
        locator:
          enabled: true # 让gateway可以发现nacos中的微服务
      routes:
        - id: product_route
          uri: lb://product-service  #匹配后提供服务的路由地址
          
          
          
          predicates:
#            - Header=X-Request-Id, \d+
#            - Cookie=username,zzyy
#            - After=2021-05-16T19:53:59.024+08:00[Asia/Shanghai]
             - Path=/product-serv/**  # 断言，路径相匹配的进行路由
          filters:
            - StripPrefix=1       

```

### 过滤

> StripPrefix=1    去除第一段路由

#### 自定义局部过滤器

> 注意：
>
> - 命名：xxxGatewayFilterFactory          使用：- xxx=
> - 参数顺序

```java
@Component
public class TimeGatewayFilterFactory extends AbstractGatewayFilterFactory<TimeGatewayFilterFactory.Config> {
    private static final String BEGIN_TIME = "beginTime";
    //构造函数
    public TimeGatewayFilterFactory() {
        super(TimeGatewayFilterFactory.Config.class);
    }
    //读取配置文件中的参数 赋值到 配置类中
    @Override
    public List<String> shortcutFieldOrder() {
        return Arrays.asList("show","count");
    }
    @Override
    public GatewayFilter apply(Config config) {
        return new GatewayFilter() {
            @Override
            public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
                if(!config.show){
                    return chain.filter(exchange);
                }
                exchange.getAttributes().put(BEGIN_TIME, System.currentTimeMillis());
                /**
                 *  pre的逻辑
                 * chain.filter().then(Mono.fromRunable(()->{
                 *     post的逻辑
                 * }))
                 */
                return chain.filter(exchange).then(Mono.fromRunnable(()->{
                    Long startTime = exchange.getAttribute(BEGIN_TIME);
                    if (startTime != null) {
                        System.out.println(exchange.getRequest().getURI() + "请求耗时: " + (System.currentTimeMillis() - startTime) + "ms");
                    }
                }));
            }
        };
    }
    @Setter
    @Getter
    static class Config{
        private boolean show;
        private int count;
    }
}

```

#### 全局过滤器

```java
@Component
public class AuthGlobalFilter implements GlobalFilter {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        String token = exchange.getRequest().getQueryParams().getFirst("token");
        if (StringUtils.isBlank(token)) {
            System.out.println("鉴权失败");
            exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);
            return exchange.getResponse().setComplete();
        }
        return chain.filter(exchange);
    }
}
```



### 集成Sentinel实现网关限流

网关是所有请求的公共入口，所以可以在网关进行限流，而且限流的方式也很多，我们本次采用前

面学过的Sentinel组件来实现网关的限流。Sentinel支持对SpringCloud Gateway、Zuul等主流网关进

行限流。

从1.6.0版本开始，Sentinel提供了SpringCloud Gateway的适配模块，可以提供两种资源维度的限流：

- route维度：即在Spring配置文件中配置的路由条目，资源名为对应的routeId

- 自定义API维度：用户可以利用Sentinel提供的API来自定义一些API分组

#### 网关集成Sentinel

https://github.com/alibaba/Sentinel/wiki/网关限流

1. 添加依赖

   ```xml
   <dependency>
   	<groupId>com.alibaba.csp</groupId>
   	<artifactId>sentinel-spring-cloud-gateway-adapter</artifactId>
   </dependency>
   ```

2. 编写配置类

   ```java
   package cn.wolfcode.config;
   @Configuration
   public class GatewayConfiguration {
       private final List<ViewResolver> viewResolvers;
       private final ServerCodecConfigurer serverCodecConfigurer;
   
       public GatewayConfiguration(ObjectProvider<List<ViewResolver>> viewResolversProvider,
                                   ServerCodecConfigurer serverCodecConfigurer) {
           this.viewResolvers = viewResolversProvider.getIfAvailable(Collections::emptyList);
           this.serverCodecConfigurer = serverCodecConfigurer;
       }
       // 配置限流的异常处理器
       @Bean
       @Order(Ordered.HIGHEST_PRECEDENCE)
       public SentinelGatewayBlockExceptionHandler sentinelGatewayBlockExceptionHandler() {
           // Register the block exception handler for Spring Cloud Gateway.
           return new SentinelGatewayBlockExceptionHandler(viewResolvers, serverCodecConfigurer);
       }
       // 初始化一个限流的过滤器
       @Bean
       @Order(Ordered.HIGHEST_PRECEDENCE)
       public GlobalFilter sentinelGatewayFilter() {
           return new SentinelGatewayFilter();
       }
       //增加对商品微服务的 限流
        @PostConstruct
       private void initGatewayRules() {
           Set<GatewayFlowRule> rules = new HashSet<>();
           rules.add(new GatewayFlowRule("product_route")
                   .setCount(3)
                   .setIntervalSec(1)
           );
           GatewayRuleManager.loadRules(rules);
       }
   }
   ```

3. 重启网关服务并测试.

**![image-20201030175846125](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/image-20201030175846125.png)**

#### 修改限流默认返回格式

1. 在配置类GatewayConfiguration.java中添加如下配置

   ```java
   @PostConstruct
   public void initBlockHandlers() {
   	BlockRequestHandler blockRequestHandler = new BlockRequestHandler() {
   	public Mono<ServerResponse> handleRequest(ServerWebExchange serverWebExchange, Throwable throwable) {
   		Map map = new HashMap<>();
   		map.put("code", 0);
   		map.put("message", "接口被限流了");
   			return ServerResponse.status(HttpStatus.OK).
   				contentType(MediaType.APPLICATION_JSON).
   				body(BodyInserters.fromValue(map));
               }
   };
   	GatewayCallbackManager.setBlockHandler(blockRequestHandler);
   }
   ```

2. 重启并测试

   **![image-20201030180400822](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/image-20201030180400822.png)**

#### 自定义API分组

自定义API分组是一种更细粒度的限流规则定义

1. 在shop-order-server项目中添加ApiController

   ```java
   package cn.wolfcode.controller;
   @RestController
   @RequestMapping("/api")
   public class ApiController {
       @RequestMapping("/hello")
       public String api1(){
           return "api";
       }
   }
   ```

2. 重启shop-order-server项目.

3. 在api-gateway项目的配置GatewayConfiguration.java中添加如下配置:

   ```java
   @PostConstruct
   private void initCustomizedApis() {
   	Set<ApiDefinition> definitions = new HashSet<>();
   	ApiDefinition api1 = new ApiDefinition("order_api")
                   .setPredicateItems(new HashSet<ApiPredicateItem>() {{
                       add(new ApiPathPredicateItem().setPattern("/order-serv/api/**").                 setMatchStrategy(SentinelGatewayConstants.URL_MATCH_STRATEGY_PREFIX));
                   }});
   	definitions.add(api1);
   	GatewayApiDefinitionManager.loadApiDefinitions(definitions);
   }
   @PostConstruct
   private void initGatewayRules() {
   	Set<GatewayFlowRule> rules = new HashSet<>();
   	rules.add(new GatewayFlowRule("product_route")
                   .setCount(3)
                   .setIntervalSec(1)
   	);
   	rules.add(new GatewayFlowRule("order_api").
                   setCount(1).
                   setIntervalSec(1));
       GatewayRuleManager.loadRules(rules);
   }
   ```

4. 直接访问localhost:8091/api/hello 是不会发生限流的，访问localhost:9000/order-serv/api/hello 就会出现限流了.



## Sleuth+Zipkin 链路追踪

### 集成链路追踪组件Sleuth

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-sleuth</artifactId>
</dependency>
```

### Zipkin+Sleuth整合

1. 下载Zipkin的jar包，在官网可以下载.

2. 通过命令行，输入下面的命令启动ZipKin Server

   ```shell
   java -jar zipkin-server-2.22.1-exec.jar
   ```

3. 通过浏览器访问 localhost:9411访问

4. 添加依赖

   ```xml
   <dependency>
   	<groupId>org.springframework.cloud</groupId>
   	<artifactId>spring-cloud-starter-zipkin</artifactId>
   </dependency>
   ```

5. yaml

   ```yaml
   spring:
     zipkin:
       base-url: http://127.0.0.1:9411/ #zipkin server的请求地址
       discoveryClientEnabled: false #让nacos把它当成一个URL，而不要当做服务名
     sleuth: 
       sampler: 
         probability: 1.0 #采样的百分比
   ```

6. 调用接口后访问 localhost:9411查看

## 分布式调度 

### Elastic-Job

#### 搭建

> 需要zooker

1. pom

   ```xml
           <dependency>
               <groupId>com.dangdang</groupId>
               <artifactId>elastic-job-lite-spring</artifactId>
               <version>2.1.5</version>
           </dependency>
   ```

2. yaml

   ```yaml
   elasticjob:
     zookeeper-url: localhost:2181
     group-name: elastic-job-group
   ```

3. 注册中心配置类

   ```java
   @Configuration
   public class RegistryCenterConfig {
       
       @Bean(initMethod = "init")
       public CoordinatorRegistryCenter createRegistryCenter(
               @Value("${elasticjob.zookeeper-url}") String zookeeperUrl,
               @Value("${elasticjob.group-name}") String groupName) {
           //zk的配置
           ZookeeperConfiguration zookeeperConfiguration = new ZookeeperConfiguration(zookeeperUrl, groupName);
           //设置zk超时时间
           zookeeperConfiguration.setSessionTimeoutMilliseconds(100);
           //创建注册中心
           return new ZookeeperRegistryCenter(zookeeperConfiguration);
       }
       
   }
   ```

4. 任务调度抽取公共方法

   ```java
   public class ElasticJobLite {
       public static LiteJobConfiguration createJobConfiguration(
               //具体类
               final Class<? extends ElasticJob> jobClass,
               //cron表达式
               final String cron,
               //分片数量
               final int shardingTotalCount,
               //分片表达式
               final String shardingItemParameters,
               //job类型 DataflowJob[true] SimpleJob[false]
               boolean dataflowType
       ) {
           // 定义作业核心配置
           JobCoreConfiguration.Builder jobCoreConfigurationBuilder = JobCoreConfiguration.newBuilder(jobClass.getSimpleName(), cron, shardingTotalCount);
           //分片
           if (!StringUtils.isEmpty(shardingItemParameters)) {
               jobCoreConfigurationBuilder.shardingItemParameters(shardingItemParameters);
           }
           //初始化JobTypeConfiguration
           JobTypeConfiguration jobConfig;
           //DataflowJob[true] SimpleJob[false]
           if (dataflowType) {
               //定义Dataflow类型配置
               jobConfig = new DataflowJobConfiguration(jobCoreConfigurationBuilder.build(), jobClass.getCanonicalName(), true);
           } else {
               // 定义SIMPLE类型配置
               jobConfig = new SimpleJobConfiguration(jobCoreConfigurationBuilder.build(), jobClass.getCanonicalName());
           }
           // 定义Lite作业根配置
           return LiteJobConfiguration.newBuilder(jobConfig).overwrite(true).build();
       }
   }
   ```

5. 任务类

   ```java
   @Component
   public class MyElasticJob implements SimpleJob {
       public void execute(ShardingContext shardingContext) {
           System.out.println("定时任务开始====>"+new Date());
       }
   }
   ```

6. 配置使用

   ```java
   @Configuration
   public class ElasticJobConfig {
       @Autowired
       private CoordinatorRegistryCenter registryCenter;
       @Bean(initMethod = "init")
       public SpringJobScheduler initSimpleElasticJob(MyElasticJob myElasticJob) {
           SpringJobScheduler springJobScheduler = new SpringJobScheduler(myElasticJob, registryCenter, ElasticJobLite.createJobConfiguration(
                   MyElasticJob.class,
                   "0/3 * * * * ?",
                   1,
                   null,
                   false)
           );
           return springJobScheduler;
       }
   
   }
   ```

#### 分片

​	作业分片是指任务的分布式执行，需要将一个任务拆分为多个独立的任务项，然后由分布式的应用实例分别执行某一个或者几个分布项。

​	例如：Elastic-Job快速入门中文件备份的案例，现有两台服务器，每台服务器分别跑一个应用实例。为了快速执行作业，那么可以讲任务分成4片，每个应用实例都执行两片。作业遍历数据逻辑应为：实例1查找text和image类型文件执行备份，实例2查找radio和vedio类型文件执行备份。如果由于服务器拓容应用实例数量增加为4，则作业遍历数据的逻辑应为: 4个实例分别处理text,image,radio,video类型的文件。

​	可以看到，通过对任务的合理分片化，从而达到任务并行处理的效果.

**分片项与业务处理解耦**

​	Elastic-Job并不直接提供数据处理的功能,框架只会将分片项分配至各个运行中的作业服务器，开发者需要自行处理分片项与真实数据的对应关系

**最大限度利用资源**

将分片项设置大于服务器的数据，最好是大于服务器倍数的数量，作业将会合理利用分布式资源，动态的分配分片项.

​	例如: 3台服务器，分成10片，则分片项结果为服务器A=0,1,2;服务器B=3,4,5;服务器C=6,7,8,9.如果   服务器C奔溃，则分片项分配结果为服务器A=0,1,2,3,4;服务器B=5,6,7,8,9.在不丢失分片项的情况下，最大限度利用现有的资源提高吞吐量.

> 注：
>
> - shardingContext.getShardingParameter() 获取分片信息

1. job

   ```java
   @Component
   public class FileCustomElasticJob implements SimpleJob {
       @Autowired
       private FileCustomMapper fileCustomMapper;
       @Override
       public void execute(ShardingContext shardingContext) {
           doWork(shardingContext.getShardingParameter());
       }
       private void doWork(String fileType){
           List<FileCustom> fileList = fileCustomMapper.selecByType(fileType);
           System.out.println("类型为:"+fileType+",文件，需要备份个数:"+fileList.size());
           for(FileCustom fileCustom:fileList){
               backUpFile(fileCustom);
           }
       }
       private void backUpFile(FileCustom fileCustom){
           try {
               //模拟备份动作
               TimeUnit.SECONDS.sleep(1);
           } catch (InterruptedException e) {
               e.printStackTrace();
           }
           System.out.println("执行文件备份====>"+fileCustom);
           fileCustomMapper.changeState(fileCustom.getId(),1);
       }
   }
   ```

2. 配置

   ```java
   @Configuration
   public class ElasticJobConfig {
       @Autowired
       private CoordinatorRegistryCenter registryCenter;
     
       @Bean(initMethod = "init")
       public SpringJobScheduler initFileCustomElasticJob(FileCustomElasticJob fileCustomElasticJob) {
           SpringJobScheduler springJobScheduler = new SpringJobScheduler(
                   fileCustomElasticJob,
                   registryCenter,
                   ElasticJobLite.createJobConfiguration(
                           FileCustomElasticJob.class,
                           "0 0/1 * * * ?",
                           4,
                           "0=text,1=image,2=radio,3=vedio",
                           false
                   )
           );
           return springJobScheduler;
       }
   
   }
   ```


#### Dataflow类型调度任务

> Dataflow类型的定时任务需要实现Dataflowjob接口，该接口提供2个方法供覆盖，分别用于抓取（fetchData）和处理（processData）数据，我们继续对例子进行改造。
>
> Dataflow类型用于处理数据流，他和SimpleJob不同，它以数据流的方式执行，调用fetchData抓取数据，知道抓取不到数据才停止作业。

1. job

   ```java
   @Component
   public class FileDataflowJob implements DataflowJob<FileCustom> {
       @Autowired
       private FileCustomMapper fileCustomMapper;
       @Override
       public List<FileCustom> fetchData(ShardingContext shardingContext) {
           List<FileCustom> fileCustoms = fileCustomMapper.fetchData(2);
           System.out.println("抓取时间:"+new Date()+",个数"+fileCustoms.size());
           return fileCustoms;
       }
       @Override
       public void processData(ShardingContext shardingContext, List<FileCustom> data) {
           try {
               //模拟备份动作
               TimeUnit.SECONDS.sleep(2);
           } catch (InterruptedException e) {
               e.printStackTrace();
           }
           for(FileCustom fileCustom:data){
               backUpFile(fileCustom);
           }
       }
       private void backUpFile(FileCustom fileCustom){
           System.out.println("执行文件备份====>"+fileCustom);
           fileCustomMapper.changeState(fileCustom.getId(),1);
       }
   }
   ```

2. 配置

   ```java
   @Configuration
   public class ElasticJobConfig {
       @Autowired
       private CoordinatorRegistryCenter registryCenter;
   
       @Bean(initMethod = "init")
       public SpringJobScheduler iniDataflowElasticJob(FileDataflowJob fileDataflowJob) {
           SpringJobScheduler springJobScheduler = new SpringJobScheduler(
                   fileDataflowJob,
                   registryCenter,
                   ElasticJobLite.createJobConfiguration(FileDataflowJob.class, "0 0/1 * * * ?", 1, null, true)
           					);
           return springJobScheduler;
       }
   }
   ```

#### 运维控制台

##### 修改Elastic-Job配置类

在ElasticJobConfig配置类中注入DataSource

```java
@Configuration
public class ElasticJobConfig {
    @Autowired
    private DataSource dataSource;
  ......
}
```

在任务配置中增加事件追踪配置

```java
@Bean(initMethod = "init")
    public SpringJobScheduler initFileCustomElasticJob(FileCustomElasticJob fileCustomElasticJob){
        //增加任务事件追踪配置
        JobEventConfiguration jobEventConfiguration = new JobEventRdbConfiguration(dataSource);
        SpringJobScheduler springJobScheduler = new SpringJobScheduler(
                fileCustomElasticJob,
                registryCenter,
                createJobConfiguration(FileCustomElasticJob.class,"0 0/1 * * * ?",4,"0=text,1=image,2=radio,3=vedio",false),
                jobEventConfiguration);
        return springJobScheduler;
    }
```

##### 日志信息表

启动后会发现在elastic-job-demo数据库中新增以下两张表

**job_execution_log**

![1588905075352](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/1588905075352.png)

记录每次作业的执行历史，分为两个步骤：

1.作业开始执行时间向数据库插入数据.

2.作业完成执行时向数据库更新数据，更新is_success,complete_time和failure_cause(如果任务执行失败)



**job_status_trace_log**

![1588905091464](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/1588905091464.png)

记录作业状态变更痕迹表，可通过每次作业运行的task_id查询作业状态变化的生命轨迹和运行轨迹.

##### 控制台搭建

elastic-job中提供了一个elastic-job-lite-console控制台

**设计理念**

1.本 控制台和Elastic-Job并无直接关系，是通过读取Elastic-Job的注册中心数据展示作业状态，或更新注册中心数据修改全局配置。

2.控制台只能控制任务本身是否运行，但不能控制作业进程的启停，因为控制台和作业本身服务器是完全分布式的，控制台并不能控制作业服务器。

**主要功能:**

1.查看作业以及服务器状态

2.快捷的修改以及删除作业配置

3.启用和禁用作业

4.跨注册中心查看作业

5.查看作业运行轨迹和运行状态

**不支持项**

1.添加作业，因为作业都是在首次运行时自动添加，使用控制台添加作业并无必要.直接在作业服务器启动包含Elasitc-Job的作业进程即可。

###### 搭建步骤

- 解压缩`elastic-job-lite-console-2.1.5.tar`

- 进入bin目录，并执行:

  ```
  bin\start.sh
  ```

- 打开浏览器访问`localhost:8899`

  用户名: root 密码: root,进入之后界面如下:

![1588906482667](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/1588906482667.png)

提供两种用户：管理员和访客，管理员拥有全部操作权限，访客仅拥有查看权限。默认管理员账号和面膜是root/root,访客用户名和密码是guest/guest,通过conf\auth.properties可以修改管理员以及访客用户名及密码

###### 配置及使用

- 配置注册中心地址

  先启动zookeeper然后再注册中心配置界面，点添加

![1588906867417](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/1588906867417.png)

点击提交后，然后点连接（zookeeper必须处于启动状态）

![1588906957795](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/1588906957795.png)



连接成功后，在作业纬度下可以显示该命名空间作业名称，分片数量及该作业的cron表达式等信息

在服务器纬度可以查看到服务器ip,当前运行的是实例数，作业总数等信息。

![1588907089548](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/1588907089548.png)

添加数据库连接之后可以查看任务的执行结果

![1588907583020](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/1588907583020.png)

然后在作业历史中就可以看到任务执行历史了。

![1588907629648](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/1588907629648.png)

## Canal 数据同步

### 原理

模仿mysql主从的slave

### 准备

- 对于自建 MySQL , 需要先开启 Binlog 写入功能，配置 binlog-format 为 ROW 模式，my.cnf 中配置如下

  ```
  [mysqld]
  log-bin=mysql-bin # 开启 binlog
  binlog-format=ROW # 选择 ROW 模式
  server_id=1 # 配置 MySQL replaction 需要定义，不要和 canal 的 slaveId 重复
  ```

  - 注意：针对阿里云 RDS for MySQL , 默认打开了 binlog , 并且账号默认具有 binlog dump 权限 , 不需要任何权限或者 binlog 设置,可以直接跳过这一步

- 授权 canal 链接 MySQL 账号具有作为 MySQL slave 的权限, 如果已有账户可直接 grant

  ```mysql
  CREATE USER canal IDENTIFIED BY 'Canal_2021';  
  GRANT SELECT, REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'canal'@'%';
  -- GRANT ALL PRIVILEGES ON *.* TO 'canal'@'%' ;
  FLUSH PRIVILEGES;
  ```

重启mysql    

```
//centos6
service mysqld restart   
//centos7
systemctl restart mysqld
```



### 启动

- 下载canal,我们使用的版本是1.1.4版本

  ```shell
  https://github.com/alibaba/canal/releases/tag/canal-1.1.4
  ```

- 解压缩

  ```
  mkdir /usr/local/canal
   tar -zxvf software/canal.deployer-1.1.4.tar.gz -C /usr/local/canal/
  ```

  - 解压完成后，进入 /tmp/canal 目录，可以看到如下结构

    ```
    drwxr-xr-x 2 jianghang jianghang  136 2013-02-05 21:51 bin
    drwxr-xr-x 4 jianghang jianghang  160 2013-02-05 21:51 conf
    drwxr-xr-x 2 jianghang jianghang 1.3K 2013-02-05 21:51 lib
    drwxr-xr-x 2 jianghang jianghang   48 2013-02-05 21:29 logs
    ```

- 配置修改

  ```
  vi /usr/local/canal/conf/example/instance.properties
  ```

  ```
  ## mysql serverId
  canal.instance.mysql.slaveId = 1234
  #position info，需要改成自己的数据库信息
  canal.instance.master.address = 127.0.0.1:3306 
  canal.instance.master.journal.name = 
  canal.instance.master.position = 
  canal.instance.master.timestamp = 
  #canal.instance.standby.address = 
  #canal.instance.standby.journal.name =
  #canal.instance.standby.position = 
  #canal.instance.standby.timestamp = 
  #username/password，需要改成自己的数据库信息
  canal.instance.dbUsername = canal  
  canal.instance.dbPassword = canal
  canal.instance.defaultDatabaseName =
  canal.instance.connectionCharset = UTF-8
  #table regex
  canal.instance.filter.regex = .\*\\\\..\*
  ```

  - canal.instance.connectionCharset 代表数据库的编码方式对应到 java 中的编码类型，比如 UTF-8，GBK , ISO-8859-1
  - 如果系统是1个 cpu，需要将 canal.instance.parser.parallel 设置为 false

- 启动

  ```
  sh /usr/local/canal/bin/startup.sh
  ```

- 查看 server 日志

  ```shell
  tail -f -n 50 logs/canal/canal.log
  ```

  ```
  2013-02-05 22:45:27.967 [main] INFO  com.alibaba.otter.canal.deployer.CanalLauncher - ## start the canal server.
  2013-02-05 22:45:28.113 [main] INFO  com.alibaba.otter.canal.deployer.CanalController - ## start the canal server[10.1.29.120:11111]
  2013-02-05 22:45:28.210 [main] INFO  com.alibaba.otter.canal.deployer.CanalLauncher - ## the canal server is running now ......
  ```

- 查看 instance 的日志

  ```
  tail -f -n 50 logs/example/example.log
  ```

  ```
  2013-02-05 22:50:45.636 [main] INFO  c.a.o.c.i.spring.support.PropertyPlaceholderConfigurer - Loading properties file from class path resource [canal.properties]
  2013-02-05 22:50:45.641 [main] INFO  c.a.o.c.i.spring.support.PropertyPlaceholderConfigurer - Loading properties file from class path resource [example/instance.properties]
  2013-02-05 22:50:45.803 [main] INFO  c.a.otter.canal.instance.spring.CanalInstanceWithSpring - start CannalInstance for 1-example 
  2013-02-05 22:50:45.810 [main] INFO  c.a.otter.canal.instance.spring.CanalInstanceWithSpring - start successful....
  ```

- 关闭

  ```
  sh /usr/local/canal/bin/stop.sh
  
  不能用 kill -9 进程 
  如果杀了, 需要删除 canal.pid 再次启动,就可以了
  ```

### sh脚本

- starCanal.sh

  ```
  # !/bin/bash
  
  echo '------------------canal-starter-------------------------' 
  sh /usr/local/canal/bin/startup.sh
  echo '------------------canal-started-------------------------'
  ```

- shutdownCanal.sh

  ```
  # !/bin/bash
  
  echo '------------------canal-shutdown-------------------------' 
  sh /usr/local/canal/bin/stop.sh
  echo '------------------canal-shutdowned-------------------------'
  
  ```

### 集成

1.首先启动Canal Server，具体部署参考给的文档

2.添加依赖

```xml
<dependency>
	<groupId>top.javatool</groupId>
	<artifactId>canal-spring-boot-starter</artifactId>
	<version>1.2.1-RELEASE</version>
</dependency>
```

3.添加配置如下

```yaml
canal:
  server: Canal服务部署的地址:11111
  destination: example
logging:
  level:
    root: info
    top:
      javatool:
        canal:
          client:
            client:
              AbstractCanalClient: error
```

4.添加Handler

```java
@Slf4j
@Component
@CanalTable(value = "t_order_info")
public class OrderaInfoHandler implements EntryHandler<OrderInfo> {
    @Autowired
    private StringRedisTemplate redisTemplate;
    @Override
    public void insert( OrderInfo orderInfo) {
        log.info("当有数据插入的时候会触发这个方法");
    }

    @Override
    public void update(OrderInfo before, OrderInfo after) {
        log.info("当有数据更新的时候会触发这个方法");
    }

    @Override
    public void delete(OrderInfo orderInfo) {
        log.info("当有数据删除的时候会触发这个方法");
    }
}
```

实体类

```java
package cn.wolfcode.domain;

import lombok.Getter;
import lombok.Setter;
import lombok.ToString;

import javax.persistence.Column;
import javax.persistence.Table;
import java.io.Serializable;
import java.math.BigDecimal;
import java.util.Date;

@Setter
@Getter
@Table(name = "t_order_info")
@ToString
public class OrderInfo implements Serializable {
    public static final Integer STATUS_ARREARAGE = 0;//未付款
    public static final Integer STATUS_ACCOUNT_PAID = 1;//已付款
    public static final Integer STATUS_CANCEL = 2;//手动取消订单
    public static final Integer STATUS_TIMEOUT = 3;//超时取消订单
    public static final Integer STATUS_REFUND = 4;//已退款
    public static final Integer PAYTYPE_ONLINE = 0;//在线支付
    public static final Integer PAYTYPE_INTERGRAL = 1;//积分支付

    @Column(name = "order_no")
    private String orderNo;//订单编号
    @Column(name = "user_id")
    private Long userId;//用户ID
    @Column(name = "product_id")
    private Long productId;//商品ID

    @Column(name = "product_name")
    private String productName;//商品名称
    @Column(name = "product_img")
    private String productImg;//商品图片
    @Column(name = "product_count")
    private Integer productCount;//商品总数
    @Column(name = "product_price")
    private BigDecimal productPrice;//商品原价
    @Column(name = "seckill_price")
    private BigDecimal seckillPrice;//秒杀价格
    @Column(name = "intergral")
    private Long intergral;//消耗积分
    @Column(name = "status")
    private Integer status = STATUS_ARREARAGE;//订单状态
    @Column(name = "create_date")
    private Date createDate;//订单创建时间

    @Column(name = "seckill_date")
    private Date seckillDate;//秒杀的日期
    @Column(name = "seckill_time")
    private Integer seckillTime;// 秒杀场次
    @Column(name = "seckill_id")
    private Long seckillId;//秒杀商品ID

}

```

## Seata分布式事务

### Seata-At

**Seata主要由三个重要组件组成：**

- TC：Transaction Coordinator 事务协调器，管理全局的分支事务的状态，用于全局性事务的提交

和回滚。

- TM：Transaction Manager 事务管理器，用于开启、提交或者回滚全局事务。

- RM：Resource Manager 资源管理器，用于分支事务上的资源管理，向TC注册分支事务，上报分

支事务的状态，接受TC的命令来提交或者回滚分支事务。

**举例**

![截屏2021-09-26 19.06.51](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/%E6%88%AA%E5%B1%8F2021-09-26%2019.06.51.png)

**程序中**

**![image-20201211151226068](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/image-20201211151226068.png)**

**Seata-AT模式的执行流程如下:** 

1. A服务的TM向TC申请开启一个全局事务，TC就会创建一个全局事务并返回一个唯一的XID

2. A服务的RM向TC注册分支事务，并及其纳入XID对应全局事务的管辖

3. A服务执行分支事务，向数据库做操作4. A服务开始远程调用B服务，此时XID会在微服务的调用链上传播

4. B服务的RM向TC注册分支事务，并将其纳入XID对应的全局事务的管辖

5. B服务执行分支事务，向数据库做操作

6. 全局事务调用链处理完毕，TM根据有无异常向TC发起全局事务的提交或者回滚

7. TC协调其管辖之下的所有分支事务， 决定是否回滚

**Seata-AT模式实现2PC与传统2PC的差别**： 

1. 架构层次方面，传统2PC方案的 RM 实际上是在数据库层，RM本质上就是数据库自身，通过XA协议实现，而 Seata的RM是以jar包的形式作为中间件层部署在应用程序这一侧的。

2. 两阶段提交方面，传统2PC无论第二阶段的决议是commit还是rollback，事务性资源的锁都要保持到Phase2完成才释放。而Seata的做法是在Phase1 就将本地事务提交，这样就可以省去Phase2持锁的时间，整体提高效率。

#### AT模式代码实现

分布式事务发起方只需要贴`@GlobalTransactional`注解即可

分支分布式事务贴上`@Transactional`即可

##### 发起方

> 如果是远程调用需要判断**返回值(关注是否做了统一异常处理以及统一返回类型)**或者**降级**抛出异常

```java
    /**
     * show 订单积分支付
     * @param orderNo 订单号
     * @return 提示信息
     */
    @Override
    @GlobalTransactional
    @Transactional
    public String doIntergral(String orderNo) {
        //1.通过orderInfo 查询订单信息 从mysql中查询
        OrderInfo orderInfo = orderInfoMapper.find(orderNo);
        //2.修改订单状态为已支付,支付类型为积分支付
        orderInfoMapper.changePayStatus(orderNo, OrderInfo.STATUS_ACCOUNT_PAID, OrderInfo.PAYTYPE_INTERGRAL);
        //3.远程调用积分服务，修改用户的积分(减积分)
        OperateIntergralVo vo = new OperateIntergralVo();
        vo.setPk(orderNo);
        vo.setValue(orderInfo.getIntergral());
        vo.setInfo("积分消费");
        vo.setUserId(orderInfo.getUserId());
        Result<String> stringResult = intergralFeign.remoteIntergralPay(vo);
        //远程失败两种情况
        //1 远程发生异常
        //2 远程服务宕机，服务降级
        if (stringResult.getCode() == 500000 || StringUtils.isEmpty(stringResult)) {
            System.out.println("远程异常");
            throw new BusinessException(SeckillCodeMsg.INTERGRAL_SERVER_ERROR);
        }
        System.out.println("远程异常没判断出来");
        //4.支付日志
        PayLog payLog = new PayLog();
        payLog.setTradeNo(orderNo);
        payLog.setOutTradeNo(orderNo);
        SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss:SSS");
        payLog.setNotifyTime(simpleDateFormat.format(new Date()));
        payLog.setTotalAmount(orderInfo.getIntergral().toString());
        payLog.setPayType(OrderInfo.PAYTYPE_INTERGRAL);
        payLogMapper.insert(payLog);
        return "积分支付成功";
    }
```

##### 分支

```java
    @Transactional
    public String intergralPay(OperateIntergralVo vo) {
        //1.添加日志 实现幂等性
        AccountLog accountLog = new AccountLog();
        accountLog.setPkValue(vo.getPk());
        accountLog.setType(AccountLog.TYPE_DECR);
        accountLog.setAmount(vo.getValue());
        accountLog.setGmtTime(new Date());
        accountLog.setInfo(vo.getInfo());
        accountLogMapper.insert(accountLog);
//        int a = 1/0; 模拟异常
        //2.修改积分
        usableIntegralMapper.addIntergral(vo.getUserId(), -vo.getValue());
        return "";
    }
```

### Seata-TCC

#### TCC模型图

![image-20201211153516323](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/image-20201211153516323.png)

#### 异常处理

##### 空回滚

 Try方法未执行,Cancel执行了

**出现原因:**

1. Try超时
2. 分布式事务回滚，触发Cancel

解决方案: Cancel方法需要识别出是否执行Try方法,如果执行了就正常执行Cancel,如果没有就直接结束

增加事务日志表来实现这个功能.

```sql
CREATE TABLE `account_transaction` (
  `tx_id` varchar(100) NOT NULL COMMENT '事务Txid',
  `action_id` varchar(100) NOT NULL COMMENT '分支事务id',
  `gmt_create` datetime NOT NULL COMMENT '创建时间',
  `gmt_modified` datetime NOT NULL COMMENT '修改时间',
  `user_id` varchar(100) NOT NULL COMMENT '账户Uid',
  `amount` varchar(100) NOT NULL COMMENT '变动金额',
  `type` varchar(100) NOT NULL DEFAULT '' COMMENT '变动类型',
  PRIMARY KEY (`tx_id`,`action_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

##### 幂等

多次调用二阶段方法

**出现原因:**

- 网络异常
- 分支事务所在服务器宕机

解决方案: 做幂等性处理

```sql
CREATE TABLE `account_transaction` (
  `tx_id` varchar(100) NOT NULL COMMENT '事务Txid',
  `action_id` varchar(100) NOT NULL COMMENT '分支事务id',
  `gmt_create` datetime NOT NULL COMMENT '创建时间',
  `gmt_modified` datetime NOT NULL COMMENT '修改时间',
  `user_id` varchar(100) NOT NULL COMMENT '账户Uid',
  `amount` varchar(100) NOT NULL COMMENT '变动金额',
  `type` varchar(100) NOT NULL DEFAULT '' COMMENT '变动类型',
  `state` smallint(4) NOT NULL COMMENT '状态: 1.初始化 2.已提交 3.已回滚',
  PRIMARY KEY (`tx_id`,`action_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```



##### 防悬挂

Cancel比Try先执行

出现原因:

1. Try超时(拥堵)
2. 分布式事务回滚触发Cancel

要允许空回滚，但是要拒绝空回滚之后的Try方法.

解决方案: 在Try方法中, 如果根据全局事务ID能查询出数据出来,说明在try方法之前执行了空回滚，此时就不能进行try方法。否则就正常执行try方法.

#### 异常处理流程图

##### Try方法

**![image-20201211173603005](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/image-20201211173603005.png)**

##### Comfirm方法

**![img](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/Confirm%E6%96%B9%E6%B3%95%E9%80%BB%E8%BE%91.jpg)**

##### Cancel方法

![img](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/Cancel%E6%96%B9%E6%B3%95%E9%80%BB%E8%BE%91.jpg)



#### TCC模式代码实现

分布式事务发起方只需要贴`@GlobalTransactional`注解即可

分支事务需要完成下面步骤:

在接口上贴上`@LocalTCC`和`@TwoPhaseBusinessAction`注解

##### 发起方

```java
    @Override
    @GlobalTransactional
    @Transactional
    public String refund(String orderNo) {
        //1.通过orderInfo 查询订单信息 从mysql中查询
        OrderInfo orderInfo = orderInfoMapper.find(orderNo);
        switch (orderInfo.getPayType()) {
            case OrderInfo.PAYTYPE_INTERGRAL:
                refoundIntergral(orderInfo);
                break;
            case OrderInfo.PAYTYPE_ONLINE:
                break;
        }
        return null;
    }

    /**
     * show 订单积分退货
     * @param orderInfo 订单信息
     */
    private void refoundIntergral(OrderInfo orderInfo) {
        //1 增加退款日志
        RefundLog refundLog = new RefundLog();
        refundLog.setOutTradeNo(orderInfo.getOrderNo());
        refundLog.setRefundAmount(orderInfo.getIntergral().toString());
        refundLog.setRefundReason("不想要了");
        refundLog.setRefundType(orderInfo.getPayType());
        refundLog.setRefundTime(new Date());
        refundLogMapper.insert(refundLog);
        //2 订单状态 1>>4
        orderInfoMapper.changeRefundStatus(orderInfo.getOrderNo(), OrderInfo.STATUS_REFUND);
        //3 远程调用积分服务 加积分
        OperateIntergralVo vo = new OperateIntergralVo();
        vo.setPk(orderInfo.getOrderNo());
        vo.setValue(orderInfo.getIntergral());
        vo.setInfo("积分退款");
        vo.setUserId(orderInfo.getUserId());
        Result<String> stringResult = intergralFeign.remoteIntergralUnfundTry(vo);
        if (StringUtils.isEmpty(stringResult) || stringResult.getCode() == 500000) {
            throw new BusinessException(SeckillCodeMsg.INTERGRAL_SERVER_ERROR);
        }
    }

```

##### 分支方

- 接口

  ```java
  @LocalTCC
  public interface IUsableIntegralService {
      /**
       * show TCC事务
       * @param vo
       * @param context
       * @return
       */
      @TwoPhaseBusinessAction(name = "remoteIntergralUnfundTry", commitMethod = "incrIntergralCommit", rollbackMethod = "incrIntergralRollback")
      String remoteIntergralUnfundTry(@BusinessActionContextParameter(paramName = "operateIntergralVo") OperateIntergralVo vo, BusinessActionContext context);
      String incrIntergralCommit(BusinessActionContext context);
      String incrIntergralRollback(BusinessActionContext context);
  }
  
  ```

- 实现类

  ```java
  @Service
  public class UsableIntegralServiceImpl implements IUsableIntegralService {
      @Resource
      private UsableIntegralMapper usableIntegralMapper;
      @Resource
      private AccountTransactionMapper accountTransactionMapper;
      @Resource
      private AccountLogMapper accountLogMapper;
      /**
       * TCC事务 退款
       */
      @Override
      public String remoteIntergralUnfundTry(@BusinessActionContextParameter(paramName = "operateIntergralVo") OperateIntergralVo vo, BusinessActionContext context) {
          //1查询事务信息
          AccountTransaction accountTransactionReturn = accountTransactionMapper.get(context.getXid(), String.valueOf(context.getBranchId()));
          //2判断是否有信息
          if (!StringUtils.isEmpty(accountTransactionReturn)) {
              //有
              switch (accountTransactionReturn.getState()) {
                  //try 幂等
                  case AccountTransaction.STATE_TRY:
                      break;
                  //confirm 抛异常
                  case AccountTransaction.STATE_COMMIT:
                      throw new BusinessException(IntergralCodeMsg.OP_INTERGRAL_ERROR);
                  //cancel 防悬挂
                  case AccountTransaction.STATE_CANCEL:
                      break;
              }
              return "";
          } else {
              //1添加事务日志
              AccountTransaction accountTransaction = new AccountTransaction();
              accountTransaction.setTxId(context.getXid());
              accountTransaction.setActionId(String.valueOf(context.getBranchId()));
              accountTransaction.setUserId(vo.getUserId());
              accountTransaction.setGmtCreated(new Date());
              accountTransaction.setGmtModified(new Date());
              accountTransaction.setAmount(vo.getValue());
              accountTransaction.setType(String.valueOf(AccountLog.TYPE_INCR));
              accountTransaction.setState(AccountTransaction.STATE_TRY);
  
              accountTransactionMapper.insert(accountTransaction);
              //2判断执行结果
              accountTransactionReturn = accountTransactionMapper.get(context.getXid(), String.valueOf(context.getBranchId()));
              //3如果为真
              if (accountTransactionReturn != null) {
                  //正常逻辑 啥也不做
                  return "";
              } else {//4如果为假
                  throw new BusinessException(IntergralCodeMsg.OP_INTERGRAL_ERROR);
              }
          }
      }
  
      @Override
      public String incrIntergralCommit(BusinessActionContext context) {
          //1查询事务信息
          AccountTransaction accountTransactionReturn = accountTransactionMapper.get(context.getXid(), String.valueOf(context.getBranchId()));
          //2判断是否有信息
          if (!StringUtils.isEmpty(accountTransactionReturn)) {
              //有
              switch (accountTransactionReturn.getState()) {
                  //try 正常逻辑 加钱
                  case AccountTransaction.STATE_TRY:
                      String operateIntergralVoToString = context.getActionContext().get("operateIntergralVo").toString();
                      OperateIntergralVo operateIntergralVo = JSON.parseObject(operateIntergralVoToString, OperateIntergralVo.class);
                      this.remoteIntergralUnfund(operateIntergralVo);
                      //修改事务状态
                      accountTransactionMapper.updateAccountTransactionState(
                              context.getXid(),
                              String.valueOf(context.getBranchId()),
                              AccountTransaction.STATE_COMMIT,
                              AccountTransaction.STATE_TRY);
                      break;
                  //confirm 幂等
                  case AccountTransaction.STATE_COMMIT:
                      break;
                  //cancel 异常
                  case AccountTransaction.STATE_CANCEL:
                      throw new BusinessException(IntergralCodeMsg.OP_INTERGRAL_ERROR);
              }
              return "";
          } else {
              //没有
              throw new BusinessException(IntergralCodeMsg.OP_INTERGRAL_ERROR);
          }
      }
  
      @Override
      public String incrIntergralRollback(BusinessActionContext context) {
          String operateIntergralVoToString = context.getActionContext().get("operateIntergralVo").toString();
          OperateIntergralVo operateIntergralVo = JSON.parseObject(operateIntergralVoToString, OperateIntergralVo.class);
          //1查询事务信息
          AccountTransaction accountTransactionReturn = accountTransactionMapper.get(context.getXid(), String.valueOf(context.getBranchId()));
          //2判断是否有信息
          if (!StringUtils.isEmpty(accountTransactionReturn)) {
              //有
              switch (accountTransactionReturn.getState()) {
                  //try 正常逻辑 修改状态
                  case AccountTransaction.STATE_TRY:
                      //修改事务状态
                      accountTransactionMapper.updateAccountTransactionState(
                              context.getXid(),
                              String.valueOf(context.getBranchId()),
                              AccountTransaction.STATE_CANCEL,
                              AccountTransaction.STATE_TRY);
                      break;
                  //confirm 抛异常
                  case AccountTransaction.STATE_COMMIT:
                      throw new BusinessException(IntergralCodeMsg.OP_INTERGRAL_ERROR);
                      //cancel
                  case AccountTransaction.STATE_CANCEL:
                      break;
              }
              return "";
          } else {
              //没有 添加事务日志
              AccountTransaction accountTransaction = new AccountTransaction();
              accountTransaction.setTxId(context.getXid());
              accountTransaction.setActionId(String.valueOf(context.getBranchId()));
              accountTransaction.setUserId(operateIntergralVo.getUserId());
              accountTransaction.setGmtCreated(new Date());
              accountTransaction.setGmtModified(new Date());
              accountTransaction.setAmount(operateIntergralVo.getValue());
              accountTransaction.setType(String.valueOf(AccountLog.TYPE_INCR));
              accountTransaction.setState(AccountTransaction.STATE_CANCEL);
              accountTransactionMapper.insert(accountTransaction);
              return "";
          }
      }
  }
  
  ```

  



## 其他

### 单服务集群IDEA

```
-Dserver.port=8001
```

![截屏2021-09-08 21.13.28](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/%E6%88%AA%E5%B1%8F2021-09-08%2021.13.28.png)


