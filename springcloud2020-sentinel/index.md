# springcloud2020 Sentinel


# springcloud2020 **Sentinel**

> Sentinel 以流量为切入点，从流量控制、熔断降级、系统负载保护等多个维度保护服务的稳定性。

## 安装运行

服务使用中的各种问题：

- 服务雪崩
- 服务降级
- 服务熔断
- 服务限流

Sentinel 分为两个部分：

- 核心库（Java 客户端）不依赖任何框架/库，能够运行于所有 Java 运行时环境，同时对 Dubbo / Spring Cloud 等框架也有较好的支持。
- 控制台（Dashboard）基于 Spring Boot 开发，打包后可以直接运行，不需要额外的 Tomcat 等应用容器。

安装步骤：

1. 下载
   - https://github.com/alibaba/Sentinel/releases
   - 下载jar包直接运行
2. 命令
   - 前提
     - java8环境
     - 8080端口不能被占用
   - 命令
     - java -jar sentinel-dashboard-版本.jar
3. 访问Sentinel管理界面
   - localhost:8080
   - 登录账号密码均为sentinel

## Sentinel初始化监控

1. 新建**cloudalibaba-sentinel-service8401**

2. pom

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <project xmlns="http://maven.apache.org/POM/4.0.0"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
       <parent>
           <artifactId>cloud2020</artifactId>
           <groupId>com.atguigu.springcloud</groupId>
           <version>1.0-SNAPSHOT</version>
       </parent>
       <modelVersion>4.0.0</modelVersion>
   
       <artifactId>cloudalibaba-sentinel-service8401</artifactId>
   
       <dependencies>
   
           <dependency><!-- 引入自己定义的api通用包，可以使用Payment支付Entity -->
               <groupId>com.atguigu.springcloud</groupId>
               <artifactId>cloud-api-commons</artifactId>
               <version>${project.version}</version>
           </dependency>
           <!--SpringCloud ailibaba nacos -->
           <dependency>
               <groupId>com.alibaba.cloud</groupId>
               <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
           </dependency>
           <!--SpringCloud ailibaba sentinel-datasource-nacos 后续做持久化用到-->
           <dependency>
               <groupId>com.alibaba.csp</groupId>
               <artifactId>sentinel-datasource-nacos</artifactId>
           </dependency>
           <!--SpringCloud ailibaba sentinel -->
           <dependency>
               <groupId>com.alibaba.cloud</groupId>
               <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
           </dependency>
           <!--openfeign-->
           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-starter-openfeign</artifactId>
           </dependency>
           <!-- SpringBoot整合Web组件+actuator -->
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-web</artifactId>
           </dependency>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-actuator</artifactId>
           </dependency>
           <!--日常通用jar包配置-->
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-devtools</artifactId>
               <scope>runtime</scope>
               <optional>true</optional>
           </dependency>
           <dependency>
               <groupId>cn.hutool</groupId>
               <artifactId>hutool-all</artifactId>
               <version>4.6.3</version>
           </dependency>
           <dependency>
               <groupId>org.projectlombok</groupId>
               <artifactId>lombok</artifactId>
               <optional>true</optional>
           </dependency>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-test</artifactId>
               <scope>test</scope>
           </dependency>
   
       </dependencies>
   
   </project>
   ```

3. yml

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
   
   management:
     endpoints:
       web:
         exposure:
           include: '*'
   
   feign:
     sentinel:
       enabled: true # 激活Sentinel对Feign的支持
   ```

4. 主启动

   ```java
   package com.atguigu.springcloud;
   
   import org.springframework.boot.SpringApplication;
   import org.springframework.boot.autoconfigure.SpringBootApplication;
   import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
   
   @EnableDiscoveryClient
   @SpringBootApplication
   public class MainApp8401 {
       public static void main(String[] args) {
           SpringApplication.run(MainApp8401.class, args);
       }
   }
   
   ```

5. 业务类FlowLimitController

   ```java
   package com.atguigu.springcloud.controller;
   
   import com.alibaba.csp.sentinel.annotation.SentinelResource;
   import com.alibaba.csp.sentinel.slots.block.BlockException;
   import lombok.extern.slf4j.Slf4j;
   import org.springframework.web.bind.annotation.GetMapping;
   import org.springframework.web.bind.annotation.RequestParam;
   import org.springframework.web.bind.annotation.RestController;
   
   import java.util.concurrent.TimeUnit;
   
   
   @RestController
   @Slf4j
   public class FlowLimitController {
   
       @GetMapping("/testA")
       public String testA()
       {
           return "------testA";
       }
   
       @GetMapping("/testB")
       public String testB()
       {
           log.info(Thread.currentThread().getName()+"\t"+"...testB");
           return "------testB";
       }
   }
   ```

6. 启动nacos 启动**Sentinel8080**`java -jar sentinel-dashboard-1.7.0.jar`

7. 启动微服务**8401**

8. **查看sentienl控制台**

   - 刚启动，空空如也，啥都没有
   - Sentinel采用的懒加载说明
     - 执行一次访问即可
       - http://localhost:8401/testA
       - http://localhost:8401/testB
     - 效果 - sentinel8080正在监控微服务8401

## Sentinel流控规则简介

![d8ae2bea252af0bb278332b3aeb8fb77](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/d8ae2bea252af0bb278332b3aeb8fb77.png)

> - 资源名：唯一名称，默认请求路径。
> - 针对来源：Sentinel可以针对调用者进行限流，填写微服务名，默认default（不区分来源）。
> - 阈值类型/单机阈值：
>   - QPS(每秒钟的请求数量)︰当调用该API的QPS达到阈值的时候，进行限流。
>   - 线程数：当调用该API的线程数达到阈值的时候，进行限流。
> - 是否集群：不需要集群。
> - 流控模式：
>   - 直接：API达到限流条件时，直接限流。
>   - 关联：当关联的资源达到阈值时，就限流自己。
>   - 链路：只记录指定链路上的流量（指定资源从入口资源进来的流量，如果达到阈值，就进行限流)【API级别的针对来源】。
> - 流控效果：
>   - 快速失败：直接失败，抛异常。
>   - Warm up：根据Code Factor（冷加载因子，默认3）的值，从阈值/codeFactor，经过预热时长，才达到设置的QPS阈值。
>   - 排队等待：匀速排队，让请求以匀速的速度通过，阈值类型必须设置为QPS，否则无效。

### Sentinel流控-QPS直接失败(太多了就记一个)

**直接 -> 快速失败（系统默认）**

**配置及说明**

表示1秒钟内查询1次就是OK，若超过次数1，就直接->快速失败，报默认错误

![56642cc2b7dd5b0d1252235c84f69173](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/56642cc2b7dd5b0d1252235c84f69173.png)

**测试**

快速多次点击访问http://localhost:8401/testA

**结果**

返回页面 Blocked by Sentinel (flow limiting)

## Sentinel降级简介

> 熔断降级概述
>
> 除了流量控制以外，对调用链路中不稳定的资源进行熔断降级也是保障高可用的重要措施之一。一个服务常常会调用别的模块，可能是另外的一个远程服务、数据库，或者第三方 API 等。例如，支付的时候，可能需要远程调用银联提供的 API；查询某个商品的价格，可能需要进行数据库查询。然而，这个被依赖服务的稳定性是不能保证的。如果依赖的服务出现了不稳定的情况，请求的响应时间变长，那么调用服务的方法的响应时间也会变长，线程会产生堆积，最终可能耗尽业务自身的线程池，服务本身也变得不可用。
>
> 现代微服务架构都是分布式的，由非常多的服务组成。不同服务之间相互调用，组成复杂的调用链路。以上的问题在链路调用中会产生放大的效果。复杂链路上的某一环不稳定，就可能会层层级联，最终导致整个链路都不可用。因此我们需要对不稳定的弱依赖服务调用进行熔断降级，暂时切断不稳定调用，避免局部不稳定因素导致整体的雪崩。熔断降级作为保护自身的手段，通常在客户端（调用端）进行配置。

![6a002ef360a4e5f20ee2748a092f0211](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/6a002ef360a4e5f20ee2748a092f0211.png)

- RT（平均响应时间，秒级）
  - 平均响应时间 超出阈值 且 在时间窗口内通过的请求>=5，两个条件同时满足后触发降级。
  - 窗口期过后关闭断路器。
  - RT最大4900（更大的需要通过-Dcsp.sentinel.statistic.max.rt=XXXX才能生效）。
- 异常比列（秒级）
  - QPS >= 5且异常比例（秒级统计）超过阈值时，触发降级;时间窗口结束后，关闭降级 。
- 异常数(分钟级)
  - 异常数(分钟统计）超过阈值时，触发降级;时间窗口结束后，关闭降级

Sentinel熔断降级会在调用链路中某个资源出现不稳定状态时（例如调用超时或异常比例升高)，对这个资源的调用进行限制，让请求快速失败，避免影响到其它的资源而导致级联错误。

当资源被降级后，在接下来的降级时间窗口之内，对该资源的调用都自动熔断（默认行为是抛出 DegradeException）。

Sentinei的断路器是没有类似Hystrix半开状态的。(Sentinei 1.8.0 已有半开状态)

半开的状态系统自动去检测是否请求有异常，没有异常就关闭断路器恢复使用，有异常则继续打开断路器不可用

### Sentinel降级-RT(太多了就记一个)

是什么？

> 平均响应时间(DEGRADE_GRADE_RT)：当1s内持续进入5个请求，对应时刻的平均响应时间（秒级）均超过阈值（ count，以ms为单位），那么在接下的时间窗口（DegradeRule中的timeWindow，以s为单位）之内，对这个方法的调用都会自动地熔断(抛出DegradeException )。注意Sentinel 默认统计的RT上限是4900 ms，超出此阈值的都会算作4900ms，若需要变更此上限可以通过启动配置项-Dcsp.sentinel.statistic.max.rt=xxx来配置。

注意：Sentinel 1.7.0才有平均响应时间（DEGRADE_GRADE_RT），Sentinel 1.8.0的没有这项，取而代之的是慢调用比例 (SLOW_REQUEST_RATIO)。

> 慢调用比例 (SLOW_REQUEST_RATIO)：选择以慢调用比例作为阈值，需要设置允许的慢调用 RT（即最大的响应时间），请求的响应时间大于该值则统计为慢调用。当单位统计时长（statIntervalMs）内请求数目大于设置的最小请求数目，并且慢调用的比例大于阈值，则接下来的熔断时长内请求会自动被熔断。经过熔断时长后熔断器会进入探测恢复状态（HALF-OPEN 状态），若接下来的一个请求响应时间小于设置的慢调用 RT 则结束熔断，若大于设置的慢调用 RT 则会再次被熔断。

测试

代码

```java
@RestController
@Slf4j
public class FlowLimitController {
	...

    @GetMapping("/testD")
    public String testD() {
        try { 
            TimeUnit.SECONDS.sleep(1); 
        } catch (InterruptedException e) { 
            e.printStackTrace(); 
        }
        log.info("testD 测试RT");
    }
}
```

配置

![3a608908cef3d557322967e6bc0e5696](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/3a608908cef3d557322967e6bc0e5696.png)

jmeter压测 线程数10 无限循环 



结论

按照上述配置，永远一秒钟打进来10个线程（大于5个了）调用testD，我们希望200毫秒处理完本次任务，如果超过200毫秒还没处理完，在未来1秒钟的时间窗口内，断路器打开（保险丝跳闸）微服务不可用，保险丝跳闸断电了后续我停止jmeter，没有这么大的访问量了，断路器关闭（保险丝恢复），微服务恢复OK。

## Sentinel热点key

![9d2aa6d777767b3233aa643330eb9cf4](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/9d2aa6d777767b3233aa643330eb9cf4.png)

> 何为热点？热点即经常访问的数据。很多时候我们希望统计某个热点数据中访问频次最高的 Top K 数据，并对其访问进行限制。比如：
>
> - 商品 ID 为参数，统计一段时间内最常购买的商品 ID 并进行限制
> - 用户 ID 为参数，针对一段时间内频繁访问的用户 ID 进行限制
> 热点参数限流会统计传入参数中的热点参数，并根据配置的限流阈值与模式，对包含热点参数的资源调用进行限流。热点参数限流可以看做是一种特殊的流量控制，仅对包含热点参数的资源调用生效

承上启下复习start

兜底方法，分为系统默认和客户自定义，两种

之前的case，限流出问题后，都是用sentinel系统默认的提示: Blocked by Sentinel (flow limiting)

我们能不能自定？类似hystrix，某个方法出问题了，就找对应的兜底降级方法?

结论 - 从HystrixCommand到@SentinelResource

```java
@RestController
@Slf4j
public class FlowLimitController
{

    ...

    @GetMapping("/testHotKey")
    @SentinelResource(value = "testHotKey",blockHandler/*兜底方法*/ = "deal_testHotKey")
    public String testHotKey(@RequestParam(value = "p1",required = false) String p1,
                             @RequestParam(value = "p2",required = false) String p2) {
        //int age = 10/0;
        return "------testHotKey";
    }
    
    /*兜底方法*/
    public String deal_testHotKey (String p1, String p2, BlockException exception) {
        return "------deal_testHotKey,o(╥﹏╥)o";  //sentinel系统默认的提示：Blocked by Sentinel (flow limiting)
    }

}
```

配置

![9620ee4e7e54d48ba7dda394fa1c8cd0](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/9620ee4e7e54d48ba7dda394fa1c8cd0.png)

一

- @SentinelResource(value = "testHotKey")
- 异常打到了前台用户界面看到，不友好

二

- @SentinelResource(value = "testHotKey", blockHandler = "dealHandler_testHotKey")
- 方法testHotKey里面第一个参数只要QPS超过每秒1次，马上降级处理
- 异常用了我们自己定义的兜底方法

测试

error
http://localhost:8401/testHotKey?p1=abc
http://localhost:8401/testHotKey?p1=abc&p2=33
right
http://localhost:8401/testHotKey?p2=abc



### **参数例外项**

- 普通 - 超过1秒钟一个后，达到阈值1后马上被限流
- **我们期望p1参数当它是某个特殊值时，它的限流值和平时不一样**
- 特例 - 假如当p1的值等于5时，它的阈值可以达到200

![3aa08b15109cd346a6083f080a0468fa](/Users/mr_j/Desktop/3aa08b15109cd346a6083f080a0468fa.png)



测试

- right - http://localhost:8401/testHotKey?p1=5
- error - http://localhost:8401/testHotKey?p1=3
- 当p1等于5的时候，阈值变为200
- 当p1不等于5的时候，阈值就是平常的1

前提条件 - 热点参数的注意点，参数必须是基本类型或者String



**其它**

在方法体抛异常

```java
@RestController
@Slf4j
public class FlowLimitController
{

    ...

    @GetMapping("/testHotKey")
    @SentinelResource(value = "testHotKey",blockHandler/*兜底方法*/ = "deal_testHotKey")
    public String testHotKey(@RequestParam(value = "p1",required = false) String p1,
                             @RequestParam(value = "p2",required = false) String p2) {
        int age = 10/0;//<----------------------------会抛异常的地方
        return "------testHotKey";
    }
    
    /*兜底方法*/
    public String deal_testHotKey (String p1, String p2, BlockException exception) {
        return "------deal_testHotKey,o(╥﹏╥)o";  //sentinel系统默认的提示：Blocked by Sentinel (flow limiting)
    }

}
```

将会抛出Spring Boot 2的默认异常页面，而不是兜底方法。

- @SentinelResource - 处理的是sentinel控制台配置的违规情况，有blockHandler方法配置的兜底处理;

- RuntimeException int age = 10/0，这个是java运行时报出的运行时异常RunTimeException，@SentinelResource不管

总结 - @SentinelResource主管配置出错，运行出错该走异常走异常



## Sentinel系统规则

>  Sentinel 系统自适应限流从整体维度对应用入口流量进行控制，结合应用的 Load、CPU 使用率、总体平均 RT、入口 QPS 和并发线程数等几个维度的监控指标，通过自适应的流控策略，让系统的入口流量和系统的负载达到一个平衡，让系统尽可能跑在最大吞吐量的同时保证系统整体的稳定性。

>  系统规则
>
> 系统保护规则是从应用级别的入口流量进行控制，从单台机器的 load、CPU 使用率、平均 RT、入口 QPS 和并发线程数等几个维度监控应用指标，让系统尽可能跑在最大吞吐量的同时保证系统整体的稳定性。
>
> 系统保护规则是应用整体维度的，而不是资源维度的，并且仅对入口流量生效。入口流量指的是进入应用的流量（EntryType.IN），比如 Web 服务或 Dubbo 服务端接收的请求，都属于入口流量。
>
> 系统规则支持以下的模式：
>
> Load 自适应（仅对 Linux/Unix-like 机器生效）：系统的 load1 作为启发指标，进行自适应系统保护。当系统 load1 超过设定的启发值，且系统当前的并发线程数超过估算的系统容量时才会触发系统保护（BBR 阶段）。系统容量由系统的 maxQps * minRt 估算得出。设定参考值一般是 CPU cores * 2.5。
> CPU usage（1.5.0+ 版本）：当系统 CPU 使用率超过阈值即触发系统保护（取值范围 0.0-1.0），比较灵敏。
> 平均 RT：当单台机器上所有入口流量的平均 RT 达到阈值即触发系统保护，单位是毫秒。
> 并发线程数：当单台机器上所有入口流量的并发线程数达到阈值即触发系统保护。
> 入口 QPS：当单台机器上所有入口流量的 QPS 达到阈值即触发系统保护。



## SentinelResource配置

> @SentinelResource(value = "customerBlockHandler",
>             blockHandlerClass = CustomerBlockHandler.class,//<-------- 自定义限流处理类
>             blockHandler = "handlerException2")//<-----------自定义限流处理方法

> ```java
> @SentinelResource(
>         value = "fallback",
>         fallback = "handlerFallback",//fallback只负责业务异常
>         blockHandler = "blockHandler",//blockHandler只负责sentinel控制台配置违规
>         exceptionsToIgnore = {IllegalArgumentException.class}) //exceptionsToIgnore，忽略指定异常，即这些异常不用兜底方法处理。和fallback配合使用
> ```

```java
package com.atguigu.springcloud.controller;

import com.alibaba.csp.sentinel.annotation.SentinelResource;
import com.alibaba.csp.sentinel.slots.block.BlockException;
import com.atguigu.springcloud.entities.CommonResult;
import com.atguigu.springcloud.entities.Payment;
import com.atguigu.springcloud.handler.CustomerBlockHandler;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class RateLimitController {

    @GetMapping("/byResource")
    @SentinelResource(value = "byResource",blockHandler = "handleException")
    public CommonResult byResource() {
        return new CommonResult(200,"按资源名称限流测试OK",new Payment(2020L,"serial001"));
    }

    public CommonResult handleException(BlockException exception) {
        return new CommonResult(444,exception.getClass().getCanonicalName()+"\t 服务不可用");
    }


    @GetMapping("/rateLimit/byUrl")
//    @SentinelResource(value = "byUrl")
    public CommonResult byUrl()
    {
        return new CommonResult(200,"按url限流测试OK",new Payment(2020L,"serial002"));
    }

    @GetMapping("/rateLimit/customerBlockHandler")
    @SentinelResource(value = "customerBlockHandler",
            blockHandlerClass = CustomerBlockHandler.class,//<-------- 自定义限流处理类
            blockHandler = "handlerException2")//<-----------自定义限流处理方法
    public CommonResult customerBlockHandler()
    {
        return new CommonResult(200,"按客戶自定义",new Payment(2020L,"serial003"));
    }
}
```

总结：

1. value ： sentinel资源名称
2. blockHandlerClass ：指定服务降级的处理类
3. blockHandler ： 指定服务降级的处理类中的处理方法
4. fallback ： 业务异常兜底方法
5. exceptionsToIgnore：忽略指定异常



## Sentinel服务熔断Ribbon环境

- 启动nacos和sentinel
- 提供者9003/9004
- 消费者84

### **提供者9003/9004**

1. 新建cloudalibaba-provider-payment9003/9004，两个一样的做法

2. POM

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <project xmlns="http://maven.apache.org/POM/4.0.0"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
       <parent>
           <artifactId>cloud2020</artifactId>
           <groupId>com.atguigu.springcloud</groupId>
           <version>1.0-SNAPSHOT</version>
       </parent>
       <modelVersion>4.0.0</modelVersion>
   
       <artifactId>cloudalibaba-provider-payment9003</artifactId>
   
       <dependencies>
           <!--SpringCloud ailibaba nacos -->
           <dependency>
               <groupId>com.alibaba.cloud</groupId>
               <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
           </dependency>
           <dependency><!-- 引入自己定义的api通用包，可以使用Payment支付Entity -->
               <groupId>com.atguigu.springcloud</groupId>
               <artifactId>cloud-api-commons</artifactId>
               <version>${project.version}</version>
           </dependency>
           <!-- SpringBoot整合Web组件 -->
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-web</artifactId>
           </dependency>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-actuator</artifactId>
           </dependency>
           <!--日常通用jar包配置-->
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-devtools</artifactId>
               <scope>runtime</scope>
               <optional>true</optional>
           </dependency>
           <dependency>
               <groupId>org.projectlombok</groupId>
               <artifactId>lombok</artifactId>
               <optional>true</optional>
           </dependency>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-test</artifactId>
               <scope>test</scope>
           </dependency>
       </dependencies>
   </project>
   ```

3. yml

   ```yml
   server:
     port: 9003
   
   spring:
     application:
       name: nacos-payment-provider
     cloud:
       nacos:
         discovery:
           server-addr: localhost:8848 #配置Nacos地址
   
   management:
     endpoints:
       web:
         exposure:
           include: '*'
   ```

4. 启动类

   ```java
   import org.springframework.boot.SpringApplication;
   import org.springframework.boot.autoconfigure.SpringBootApplication;
   import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
   
   @SpringBootApplication
   @EnableDiscoveryClient
   public class PaymentMain9003 {
       public static void main(String[] args) {
           SpringApplication.run(PaymentMain9003.class, args);
       }
   }
   ```

5. 业务类

   ```java
   import com.atguigu.springcloud.entities.CommonResult;
   import com.atguigu.springcloud.entities.Payment;
   import org.springframework.beans.factory.annotation.Value;
   import org.springframework.web.bind.annotation.GetMapping;
   import org.springframework.web.bind.annotation.PathVariable;
   import org.springframework.web.bind.annotation.RestController;
   
   import java.util.HashMap;
   
   @RestController
   public class PaymentController {
       @Value("${server.port}")
       private String serverPort;
   
       //模拟数据库
       public static HashMap<Long,Payment> hashMap = new HashMap<>();
       static
       {
           hashMap.put(1L,new Payment(1L,"28a8c1e3bc2742d8848569891fb42181"));
           hashMap.put(2L,new Payment(2L,"bba8c1e3bc2742d8848569891ac32182"));
           hashMap.put(3L,new Payment(3L,"6ua8c1e3bc2742d8848569891xt92183"));
       }
   
       @GetMapping(value = "/paymentSQL/{id}")
       public CommonResult<Payment> paymentSQL(@PathVariable("id") Long id)
       {
           Payment payment = hashMap.get(id);
           CommonResult<Payment> result = new CommonResult(200,"from mysql,serverPort:  "+serverPort,payment);
           return result;
       }
   
   }
   ```

### **消费者84**

1. cloudalibaba-consumer-nacos-order84

2. pom

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <project xmlns="http://maven.apache.org/POM/4.0.0"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
       <parent>
           <artifactId>cloud2020</artifactId>
           <groupId>com.atguigu.springcloud</groupId>
           <version>1.0-SNAPSHOT</version>
       </parent>
       <modelVersion>4.0.0</modelVersion>
   
       <artifactId>cloudalibaba-consumer-nacos-order84</artifactId>
   
       <dependencies>
           <!--SpringCloud openfeign -->
           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-starter-openfeign</artifactId>
           </dependency>
           <!--SpringCloud ailibaba nacos -->
           <dependency>
               <groupId>com.alibaba.cloud</groupId>
               <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
           </dependency>
           <!--SpringCloud ailibaba sentinel -->
           <dependency>
               <groupId>com.alibaba.cloud</groupId>
               <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
           </dependency>
           <!-- 引入自己定义的api通用包，可以使用Payment支付Entity -->
           <dependency>
               <groupId>com.atguigu.springcloud</groupId>
               <artifactId>cloud-api-commons</artifactId>
               <version>${project.version}</version>
           </dependency>
           <!-- SpringBoot整合Web组件 -->
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-web</artifactId>
           </dependency>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-actuator</artifactId>
           </dependency>
           <!--日常通用jar包配置-->
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-devtools</artifactId>
               <scope>runtime</scope>
               <optional>true</optional>
           </dependency>
           <dependency>
               <groupId>org.projectlombok</groupId>
               <artifactId>lombok</artifactId>
               <optional>true</optional>
           </dependency>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-test</artifactId>
               <scope>test</scope>
           </dependency>
       </dependencies>
   </project>
   ```

3. yml

   ```yml
   server:
     port: 84
   
   spring:
     application:
       name: nacos-order-consumer
     cloud:
       nacos:
         discovery:
           server-addr: localhost:8848
       sentinel:
         transport:
           #配置Sentinel dashboard地址
           dashboard: localhost:8080
           #默认8719端口，假如被占用会自动从8719开始依次+1扫描,直至找到未被占用的端口
           port: 8719
   
   #消费者将要去访问的微服务名称(注册成功进nacos的微服务提供者)
   service-url:
     nacos-user-service: http://nacos-payment-provider
   
   # 激活Sentinel对Feign的支持
   feign:
     sentinel:
       enabled: true
   ```

4. 启动类

   ```java
   import org.springframework.boot.SpringApplication;
   import org.springframework.boot.autoconfigure.SpringBootApplication;
   import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
   import org.springframework.cloud.openfeign.EnableFeignClients;
   
   @EnableDiscoveryClient
   @SpringBootApplication
   @EnableFeignClients
   public class OrderNacosMain84 {
       public static void main(String[] args) {
           SpringApplication.run(OrderNacosMain84.class, args);
       }
   }
   ```

5. 业务类ApplicationContextConfig

   ```java
   import org.springframework.cloud.client.loadbalancer.LoadBalanced;
   import org.springframework.context.annotation.Bean;
   import org.springframework.context.annotation.Configuration;
   import org.springframework.web.client.RestTemplate;
   
   
   @Configuration
   public class ApplicationContextConfig {
   
       @Bean
       @LoadBalanced
       public RestTemplate getRestTemplate() {
           return new RestTemplate();
       }
   }
   ```

   CircleBreakerController

   ```java
   import com.alibaba.csp.sentinel.annotation.SentinelResource;
   import com.alibaba.csp.sentinel.slots.block.BlockException;
   import com.atguigu.springcloud.alibaba.service.PaymentService;
   import com.atguigu.springcloud.entities.CommonResult;
   import com.atguigu.springcloud.entities.Payment;
   import lombok.extern.slf4j.Slf4j;
   import org.springframework.web.bind.annotation.GetMapping;
   import org.springframework.web.bind.annotation.PathVariable;
   import org.springframework.web.bind.annotation.RequestMapping;
   import org.springframework.web.bind.annotation.RestController;
   import org.springframework.web.client.RestTemplate;
   
   import javax.annotation.Resource;
   
   @RestController
   @Slf4j
   public class CircleBreakerController {
       public static final String SERVICE_URL = "http://nacos-payment-provider";
   
       @Resource
       private RestTemplate restTemplate;
    
       @RequestMapping("/consumer/fallback/{id}")
       @SentinelResource(value = "fallback")//没有配置
       public CommonResult<Payment> fallback(@PathVariable Long id)
       {
           CommonResult<Payment> result = restTemplate.getForObject(SERVICE_URL + "/paymentSQL/"+id,CommonResult.class,id);
   
           if (id == 4) {
               throw new IllegalArgumentException ("IllegalArgumentException,非法参数异常....");
           }else if (result.getData() == null) {
               throw new NullPointerException ("NullPointerException,该ID没有对应记录,空指针异常");
           }
   
           return result;
       }
       
   }
   ```

6. 目的

   fallback管运行异常
   blockHandler管配置违规
   测试地址 - http://localhost:84/consumer/fallback/1

   没有任何配置

   只配置fallback

   只配置blockHandler

   fallback和blockHandler都配置

   忽略属性

7. 详情看项目

## Sentinel服务熔断OpenFeign

**修改84模块**

- 84消费者调用提供者9003
- Feign组件一般是消费侧

1. pom

   ```xml
   <!--SpringCloud openfeign -->
   
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-openfeign</artifactId>
   </dependency>
   ```

2. yml

   ```yml
   # 激活Sentinel对Feign的支持
   feign:
     sentinel:
       enabled: true
   ```

3. 业务类

   带@Feignclient注解的业务接口，fallback = PaymentFallbackService.class

   ```java
   import com.atguigu.springcloud.entities.CommonResult;
   import com.atguigu.springcloud.entities.Payment;
   import org.springframework.cloud.openfeign.FeignClient;
   import org.springframework.web.bind.annotation.GetMapping;
   import org.springframework.web.bind.annotation.PathVariable;
   
   @FeignClient(value = "nacos-payment-provider",fallback = PaymentFallbackService.class)
   public interface PaymentService
   {
       @GetMapping(value = "/paymentSQL/{id}")
       public CommonResult<Payment> paymentSQL(@PathVariable("id") Long id);
   }
   ```

   ```java
   import com.atguigu.springcloud.entities.CommonResult;
   import com.atguigu.springcloud.entities.Payment;
   import org.springframework.stereotype.Component;
   
   @Component
   public class PaymentFallbackService implements PaymentService {
       @Override
       public CommonResult<Payment> paymentSQL(Long id)
       {
           return new CommonResult<>(44444,"服务降级返回,---PaymentFallbackService",new Payment(id,"errorSerial"));
       }
   }
   ```

   controller

   ```java
   @RestController
   @Slf4j
   public class CircleBreakerController {
   
       ...
       
   	//==================OpenFeign
       @Resource
       private PaymentService paymentService;
   
       @GetMapping(value = "/consumer/paymentSQL/{id}")
       public CommonResult<Payment> paymentSQL(@PathVariable("id") Long id)
       {
           return paymentService.paymentSQL(id);
       }
   }
   ```

4. 主启动

   ```java
   import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
   
   @EnableDiscoveryClient
   @SpringBootApplication
   @EnableFeignClients//<------------------------
   public class OrderNacosMain84 {
       public static void main(String[] args) {
           SpringApplication.run(OrderNacosMain84.class, args);
       }
   }
   ```

5. 测试 - http://localhost:84/consumer/paymentSQL/1

   测试84调用9003，此时故意关闭9003微服务提供者，**84消费侧自动降级**，不会被耗死。



## Sentinel持久化规则（有点鸡肋）

是什么

一旦我们重启应用，sentinel规则将消失，生产环境需要将配置规则进行持久化。

怎么玩

将限流配置规则持久化进Nacos保存，只要刷新8401某个rest地址，sentinel控制台的流控规则就能看到，只要Nacos里面的配置不删除，针对8401上sentinel上的流控规则持续有效。

步骤

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



快速访问测试接口 - http://localhost:8401/rateLimit/byUrl - 页面返回Blocked by Sentinel (flow limiting)

停止8401再看sentinel - 停机后发现流控规则没有了



重新启动8401再看sentinel

- 乍一看还是没有，稍等一会儿
- 多次调用 - http://localhost:8401/rateLimit/byUrl
- 重新配置出现了，持久化验证通过

