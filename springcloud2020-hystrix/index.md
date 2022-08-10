# springcloud2020 Hystrix


# springcloud2020 Hystrix

# 介绍

概述

分布式系统面临的问题

复杂分布式体系结构中的应用程序有数十个依赖关系，每个依赖关系在某些时候将不可避免地失败。

服务雪崩

多个微服务之间调用的时候，假设微服务A调用微服务B和微服务C，微服务B和微服务C又调用其它的微服务，这就是所谓的“扇出”。如果扇出的链路上某个微服务的调用响应时间过长或者不可用，对微服务A的调用就会占用越来越多的系统资源，进而引起系统崩溃，所谓的“雪崩效应”.
对于高流量的应用来说，单一的后避依赖可能会导致所有服务器上的所有资源都在几秒钟内饱和。比失败更糟糕的是，这些应用程序还可能导致服务之间的延迟增加，备份队列，线程和其他系统资源紧张，导致整个系统发生更多的级联故障。这些都表示需要对故障和延迟进行隔离和管理，以便单个依赖关系的失败，不能取消整个应用程序或系统。

所以，通常当你发现一个模块下的某个实例失败后，这时候这个模块依然还会接收流量，然后这个有问题的模块还调用了其他的模块，这样就会发生级联故障，或者叫雪崩。

Hystrix是什么

Hystrix是一个用于处理分布式系统的延迟和容错的开源库，在分布式系统里，许多依赖不可避免的会调用失败，比如超时、异常等，Hystrix能够保证在一个依赖出问题的情况下，不会导致整体服务失败，避免级联故障，以提高分布式系统的弹性。

"断路器”本身是一种开关装置，当某个服务单元发生故障之后，通过断路器的故障监控（类似熔断保险丝)，向调用方返回一个符合预期的、可处理的备选响应（FallBack)，而不是长时间的等待或者抛出调用方无法处理的异常，这样就保证了服务调用方的线程不会被长时间、不必要地占用，从而避免了故障在分布式系统中的蔓延，乃至雪崩。

# 能做什么

- 服务降级
- 服务熔断
- 服务限流
- 接近实对的监控

# 服务降级

服务器忙，请稍后再试，不让客户端等待并立刻返回一个友好提示，fallback

**哪些情况会出发降级**

- 程序运行导常
- 超时
- 服务熔断触发服务降级
- 线程池/信号量打满也会导致服务降级

## Hystrix支付微服务构建

> 以此module为根基平台

1. 建cloud-provider-hygtrix-payment8001

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
   
       <artifactId>cloud-provider-hygtrix-payment8001</artifactId>
   
       <dependencies>
           <!--hystrix-->
           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
           </dependency>
           <!--eureka client-->
           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
           </dependency>
           <!--web-->
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-web</artifactId>
           </dependency>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-actuator</artifactId>
           </dependency>
           <dependency><!-- 引入自己定义的api通用包，可以使用Payment支付Entity -->
               <groupId>com.atguigu.springcloud</groupId>
               <artifactId>cloud-api-commons</artifactId>
               <version>${project.version}</version>
           </dependency>
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
     port: 8001
   
   spring:
     application:
       name: cloud-provider-hystrix-payment
   
   eureka:
     client:
       register-with-eureka: true
       fetch-registry: true
       service-url:
         #defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka
         defaultZone: http://eureka7001.com:7001/eureka
   ```

4. 启动类

   ```java
   package com.atguigu.springcloud;
   
   import org.springframework.boot.SpringApplication;
   import org.springframework.boot.autoconfigure.SpringBootApplication;
   import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
   
   @SpringBootApplication
   @EnableEurekaClient
   public class PaymentHystrixMain8001
   {
       public static void main(String[] args) {
           SpringApplication.run(PaymentHystrixMain8001.class, args);
       }
   }
   ```

5. service

   ```java
   package com.atguigu.springcloud.service;
   
   import org.springframework.stereotype.Service;
   import java.util.concurrent.TimeUnit;
   
   @Service
   public class PaymentService {
       public String paymentInfo_OK(Integer id)
       {
           return "线程池:  "+Thread.currentThread().getName()+"  paymentInfo_OK,id:  "+id+"\t"+"O(∩_∩)O哈哈~";
       }
     
       public String paymentInfo_TimeOut(Integer id)
       {
           try {
               TimeUnit.MILLISECONDS.sleep(3000);
           } catch (InterruptedException e) {
               e.printStackTrace();
           }
           return "线程池:  "+Thread.currentThread().getName()+" id:  "+id+"\t"+"O(∩_∩)O哈哈~"+"  耗时(秒): 3";
       }
   
   }
   
   ```

6. controller

   ```java
   package com.atguigu.springcloud.controller;
   
   import com.atguigu.springcloud.service.PaymentService;
   import lombok.extern.slf4j.Slf4j;
   import org.springframework.beans.factory.annotation.Value;
   import org.springframework.web.bind.annotation.GetMapping;
   import org.springframework.web.bind.annotation.PathVariable;
   import org.springframework.web.bind.annotation.RestController;
   
   import javax.annotation.Resource;
   
   /**
    */
   @RestController
   @Slf4j
   public class PaymentController
   {
       @Resource
       private PaymentService paymentService;
   
       @Value("${server.port}")
       private String serverPort;
   
       @GetMapping("/payment/hystrix/ok/{id}")
       public String paymentInfo_OK(@PathVariable("id") Integer id)
       {
           String result = paymentService.paymentInfo_OK(id);
           log.info("*****result: "+result);
           return result;
       }
   
       @GetMapping("/payment/hystrix/timeout/{id}")
       public String paymentInfo_TimeOut(@PathVariable("id") Integer id)
       {
           String result = paymentService.paymentInfo_TimeOut(id);
           log.info("*****result: "+result);
           return result;
       }
   }
   ```

7. 测试

## JMeter高并发压测后卡顿

> **Jmeter压测结论**
>
> 上面还是服务提供者8001自己测试，假如此时外部的消费者80也来访问，那消费者只能干等，最终导致消费端80不满意，服务端8001直接被拖慢。

## 订单微服务

1. 建 cloud-consumer-feign-hystrix-order80

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
   
       <artifactId>cloud-consumer-feign-hystrix-order80</artifactId>
   
       <dependencies>
           <!--openfeign-->
           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-starter-openfeign</artifactId>
           </dependency>
           <!--hystrix-->
           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
           </dependency>
           <!--eureka client-->
           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
           </dependency>
           <!-- 引入自己定义的api通用包，可以使用Payment支付Entity -->
           <dependency>
               <groupId>com.atguigu.springcloud</groupId>
               <artifactId>cloud-api-commons</artifactId>
               <version>${project.version}</version>
           </dependency>
           <!--web-->
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-web</artifactId>
           </dependency>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-actuator</artifactId>
           </dependency>
           <!--一般基础通用配置-->
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
     port: 80
   
   eureka:
     client:
       register-with-eureka: false
       service-url:
         defaultZone: http://eureka7001.com:7001/eureka/
   
   ```

4. 启动类

   ```java
   package com.atguigu.springcloud;
   
   import org.springframework.boot.SpringApplication;
   import org.springframework.boot.autoconfigure.SpringBootApplication;
   import org.springframework.cloud.openfeign.EnableFeignClients;
   
   /**
    */
   @SpringBootApplication
   @EnableFeignClients
   public class OrderHystrixMain80
   {
       public static void main(String[] args)
       {
           SpringApplication.run(OrderHystrixMain80.class,args);
       }
   }
   ```

5. service

   ```java
   package com.atguigu.springcloud.service;
   
   import org.springframework.stereotype.Component;
   import org.springframework.web.bind.annotation.GetMapping;
   import org.springframework.web.bind.annotation.PathVariable;
   
   /**
    */
   @Component
   @FeignClient(value = "cloud-provider-hystrix-payment")
   public interface PaymentHystrixService
   {
       @GetMapping("/payment/hystrix/ok/{id}")
       public String paymentInfo_OK(@PathVariable("id") Integer id);
   
       @GetMapping("/payment/hystrix/timeout/{id}")
       public String paymentInfo_TimeOut(@PathVariable("id") Integer id);
   }
   
   ```

6. controller

   ```java
   package com.atguigu.springcloud.controller;
   
   import com.atguigu.springcloud.service.PaymentHystrixService;
   import com.netflix.hystrix.contrib.javanica.annotation.DefaultProperties;
   import com.netflix.hystrix.contrib.javanica.annotation.HystrixCommand;
   import com.netflix.hystrix.contrib.javanica.annotation.HystrixProperty;
   import lombok.extern.slf4j.Slf4j;
   import org.springframework.web.bind.annotation.GetMapping;
   import org.springframework.web.bind.annotation.PathVariable;
   import org.springframework.web.bind.annotation.RestController;
   
   import javax.annotation.Resource;
   
   @RestController
   @Slf4j
   @DefaultProperties(defaultFallback = "payment_Global_FallbackMethod")
   public class OrderHystirxController {
       @Resource
       private PaymentHystrixService paymentHystrixService;
   
       @GetMapping("/consumer/payment/hystrix/ok/{id}")
       public String paymentInfo_OK(@PathVariable("id") Integer id)
       {
           String result = paymentHystrixService.paymentInfo_OK(id);
           return result;
       }
   
       @GetMapping("/consumer/payment/hystrix/timeout/{id}")
       public String paymentInfo_TimeOut(@PathVariable("id") Integer id) {
           String result = paymentHystrixService.paymentInfo_TimeOut(id);
           return result;
       }
   
   }
   ```

7. 测试正常

8. 高并发测试

   2W个线程压8001

   消费端80微服务再去访问正常的Ok微服务8001地址

   http://localhost/consumer/payment/hystrix/ok/32

   消费者80被拖慢

   原因：8001同一层次的其它接口服务被困死，因为tomcat线程池里面的工作线程已经被挤占完毕。

   正因为有上述故障或不佳表现才有我们的降级/容错/限流等技术诞生

## 降级容错解决的维度要求

> 超时导致服务器变慢(转圈) - 超时不再等待
>
> 出错(宕机或程序运行出错) - 出错要有兜底
>
> 解决：
>
> 对方服务(8001)超时了，调用者(80)不能一直卡死等待，必须有服务降级。
> 对方服务(8001)down机了，调用者(80)不能一直卡死等待，必须有服务降级。
> 对方服务(8001)OK，调用者(80)自己出故障或有自我要求(自己的等待时间小于服务提供者)，自己处理降级。

## Hystrix之服务降级支付侧fallback

> @HystrixCommand
>
> @EnableCircuitBreaker

### 8001提供服务者自我降级

降级配置 - @HystrixCommand

8001先从自身找问题

设置自身调用超时时间的峰值，峰值内可以正常运行，超过了需要有兜底的方法处埋，作服务降级fallback。

8001fallback

业务类启用 - @HystrixCommand报异常后如何处理

—旦调用服务方法失败并抛出了错误信息后，会自动调用@HystrixCommand标注好的fallbackMethod调用类中的指定方法

1. service

   ```java
   package com.atguigu.springcloud.service;
   
   import com.netflix.hystrix.contrib.javanica.annotation.HystrixCommand;
   import com.netflix.hystrix.contrib.javanica.annotation.HystrixProperty;
   import org.springframework.stereotype.Service;
   
   import java.util.concurrent.TimeUnit;
   
   @Service
   public class PaymentService {
       public String paymentInfo_OK(Integer id)
       {
           return "线程池:  "+Thread.currentThread().getName()+"  paymentInfo_OK,id:  "+id+"\t"+"O(∩_∩)O哈哈~";
       }
   
       @HystrixCommand(
               fallbackMethod = "paymentInfo_TimeOutHandler",
               commandProperties = {
               @HystrixProperty(
                       name = "execution.isolation.thread.timeoutInMilliseconds",
                       value = "5000"
               )
       })
       public String paymentInfo_TimeOut(Integer id)
       {
   //        int age = 10/0;
           try {
               TimeUnit.MILLISECONDS.sleep(3000);
           } catch (InterruptedException e) {
               e.printStackTrace();
           }
           return "线程池:  "+Thread.currentThread().getName()+" id:  "+id+"\t"+"O(∩_∩)O哈哈~"+"  耗时(秒): 3";
       }
   
       public String paymentInfo_TimeOutHandler(Integer id)
       {
           return "线程池:  "+Thread.currentThread().getName()+"  8001系统繁忙或者运行报错，请稍后再试,id:  "+id+"\t"+"o(╥﹏╥)o";
       }
   }
   
   ```

2. 主启动类激活

   ```java
   package com.atguigu.springcloud;
   
   import org.springframework.boot.SpringApplication;
   import org.springframework.boot.autoconfigure.SpringBootApplication;
   import org.springframework.cloud.client.circuitbreaker.EnableCircuitBreaker;
   import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
   
   @SpringBootApplication
   @EnableEurekaClient
   @EnableCircuitBreaker//
   public class PaymentHystrixMain8001
   {
       public static void main(String[] args) {
           SpringApplication.run(PaymentHystrixMain8001.class, args);
       }
   }
   ```

### 80客户端降级

1. yml

   ```yml
   server:
     port: 80
   
   eureka:
     client:
       register-with-eureka: false
       service-url:
         defaultZone: http://eureka7001.com:7001/eureka/
   
   #开启
   feign:
     hystrix:
       enabled: true
   
   ```

2. 主启动

   ```java
   import org.springframework.boot.SpringApplication;
   import org.springframework.boot.autoconfigure.SpringBootApplication;
   import org.springframework.cloud.netflix.hystrix.EnableHystrix;
   import org.springframework.cloud.openfeign.EnableFeignClients;
   
   @SpringBootApplication
   @EnableFeignClients
   @EnableHystrix//添加到此处
   public class OrderHystrixMain80{
       
       public static void main(String[] args){
           SpringApplication.run(OrderHystrixMain80.class,args);
       }
   }
   ```

3. 业务类controller

   ```java
   package com.atguigu.springcloud.controller;
   
   import com.atguigu.springcloud.service.PaymentHystrixService;
   import com.netflix.hystrix.contrib.javanica.annotation.DefaultProperties;
   import com.netflix.hystrix.contrib.javanica.annotation.HystrixCommand;
   import com.netflix.hystrix.contrib.javanica.annotation.HystrixProperty;
   import lombok.extern.slf4j.Slf4j;
   import org.springframework.web.bind.annotation.GetMapping;
   import org.springframework.web.bind.annotation.PathVariable;
   import org.springframework.web.bind.annotation.RestController;
   
   import javax.annotation.Resource;
   
   @RestController
   @Slf4j
   @DefaultProperties(defaultFallback = "payment_Global_FallbackMethod")
   public class OrderHystirxController {
       @Resource
       private PaymentHystrixService paymentHystrixService;
   
       @GetMapping("/consumer/payment/hystrix/ok/{id}")
       public String paymentInfo_OK(@PathVariable("id") Integer id)
       {
           String result = paymentHystrixService.paymentInfo_OK(id);
           return result;
       }
   
       @GetMapping("/consumer/payment/hystrix/timeout/{id}")
       @HystrixCommand(
               fallbackMethod = "paymentTimeOutFallbackMethod",
               commandProperties = {
               @HystrixProperty(
                       name="execution.isolation.thread.timeoutInMilliseconds",
                       value="1500"
               )
       })
       public String paymentInfo_TimeOut(@PathVariable("id") Integer id) {
           int age = 10/0;
           String result = paymentHystrixService.paymentInfo_TimeOut(id);
           return result;
       }
   
       //善后方法
       public String paymentTimeOutFallbackMethod(@PathVariable("id") Integer id){
           return "我是消费者80,对方支付系统繁忙请10秒钟后再试或者自己运行出错请检查自己,o(╥﹏╥)o";
       }
   }
   ```

### Hystrix之全局服务降级DefaultProperties

> **目前问题1** 每个业务方法对应一个兜底的方法，代码膨胀

1. controller

   > 解决方法 每个类配置一个公用的兜底方法
   >
   > 在类上添加注解DefaultProperties
   >
   > 在方法上添加注解@HystrixCommand 不写参数 写了就是单独的

   ```java
   package com.atguigu.springcloud.controller;
   
   import com.atguigu.springcloud.service.PaymentHystrixService;
   import com.netflix.hystrix.contrib.javanica.annotation.DefaultProperties;
   import com.netflix.hystrix.contrib.javanica.annotation.HystrixCommand;
   import com.netflix.hystrix.contrib.javanica.annotation.HystrixProperty;
   import lombok.extern.slf4j.Slf4j;
   import org.springframework.web.bind.annotation.GetMapping;
   import org.springframework.web.bind.annotation.PathVariable;
   import org.springframework.web.bind.annotation.RestController;
   
   import javax.annotation.Resource;
   
   @RestController
   @Slf4j
   @DefaultProperties(defaultFallback = "payment_Global_FallbackMethod")
   public class OrderHystirxController {
       @Resource
       private PaymentHystrixService paymentHystrixService;
   
       @GetMapping("/consumer/payment/hystrix/ok/{id}")
       public String paymentInfo_OK(@PathVariable("id") Integer id)
       {
           String result = paymentHystrixService.paymentInfo_OK(id);
           return result;
       }
   
       @GetMapping("/consumer/payment/hystrix/timeout/{id}")
   //    @HystrixCommand(
   //            fallbackMethod = "paymentTimeOutFallbackMethod",
   //            commandProperties = {
   //            @HystrixProperty(
   //                    name="execution.isolation.thread.timeoutInMilliseconds",
   //                    value="1500"
   //            )
   //    })
       @HystrixCommand
       public String paymentInfo_TimeOut(@PathVariable("id") Integer id) {
           int age = 10/0;
           String result = paymentHystrixService.paymentInfo_TimeOut(id);
           return result;
       }
   
       //善后方法
       public String paymentTimeOutFallbackMethod(@PathVariable("id") Integer id){
           return "我是消费者80,对方支付系统繁忙请10秒钟后再试或者自己运行出错请检查自己,o(╥﹏╥)o";
       }
   
       //下面是全局fallback
       public String payment_Global_FallbackMethod(){
           return "Global异常处理信息，请稍后再试，全局";
       }
   }
   ```

### Hystrix之通配服务降级FeignFallback

> 统一和自定义的分开，代码混乱
>
> 服务降级，客户端去调用服务端，碰上服务端宕机或关闭
>
> 本次案例服务降级处理是在客户端80实现完成的，与服务端8001没有关系，只需要为Feign客户端定义的接口添加一个服务降级处理的实现类即可实现解耦
>
> 未来我们要面对的异常
>
> 运行
> 超时
> 宕机
> 修改cloud-consumer-feign-hystrix-order80

1. service 

   新建PaymentFallbackService类 实现 PaymentHystrixService接口

   ```java
   package com.atguigu.springcloud.service;
   
   import org.springframework.stereotype.Component;
   
   @Component
   public class PaymentFallbackService implements PaymentHystrixService{
   
       @Override
       public String paymentInfo_OK(Integer id) {
           return "--- PaymentFallbackService --- fallback --- paymentInfo_OK";
       }
   
       @Override
       public String paymentInfo_TimeOut(Integer id) {
           return "--- PaymentFallbackService --- fallback --- paymentInfo_TimeOut";
       }
   }
   
   ```

2. 修改 PaymentHystrixService接口

   ```java
   package com.atguigu.springcloud.service;
   
   import org.springframework.cloud.openfeign.FeignClient;
   import org.springframework.stereotype.Component;
   import org.springframework.web.bind.annotation.GetMapping;
   import org.springframework.web.bind.annotation.PathVariable;
   
   /**
    */
   @Component
   @FeignClient(value = "cloud-provider-hystrix-payment",fallback = PaymentFallbackService.class)
   public interface PaymentHystrixService
   {
       @GetMapping("/payment/hystrix/ok/{id}")
       public String paymentInfo_OK(@PathVariable("id") Integer id);
   
       @GetMapping("/payment/hystrix/timeout/{id}")
       public String paymentInfo_TimeOut(@PathVariable("id") Integer id);
   }
   
   ```

3. 测试

   单个eureka先启动7001

   PaymentHystrixMain8001启动

   正常访问测试 - http://localhost/consumer/payment/hystrix/ok/1

   故意关闭微服务8001

   客户端自己调用提示 - 此时服务端provider已经down了，但是我们做了服务降级处理，让客户端在服务端不可用时也会获得提示信息而不会挂起耗死服务器。

# 服务熔断

> 断路器，相当于保险丝。
>
> 熔断机制概述
>
> 熔断机制是应对雪崩效应的一种微服务链路保护机制。当扇出链路的某个微服务出错不可用或者响应时间太长时，会进行服务的降级，进而熔断该节点微服务的调用，快速返回错误的响应信息。当检测到该节点微服务调用响应正常后，恢复调用链路。
>
> 在Spring Cloud框架里，熔断机制通过Hystrix实现。Hystrix会监控微服务间调用的状况，当失败的调用到一定阈值，缺省是5秒内20次调用失败，就会启动熔断机制。熔断机制的注解是@HystrixCommand。
>
> 修改 cloud-provider-hystrix-payment8001

1. service

   ```java
   @Service
   public class PaymentService{    
   
       ...
       
       //=====服务熔断
       @HystrixCommand(fallbackMethod = "paymentCircuitBreaker_fallback",commandProperties = {
               @HystrixProperty(name = "circuitBreaker.enabled",value = "true"),// 是否开启断路器
               @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold",value = "10"),// 请求次数
               @HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds",value = "10000"), // 时间窗口期
               @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage",value = "60"),// 失败率达到多少后跳闸
       })
       public String paymentCircuitBreaker(@PathVariable("id") Integer id) {
           if(id < 0) {
               throw new RuntimeException("******id 不能负数");
           }
           String serialNumber = IdUtil.simpleUUID();
   
           return Thread.currentThread().getName()+"\t"+"调用成功，流水号: " + serialNumber;
       }
       public String paymentCircuitBreaker_fallback(@PathVariable("id") Integer id) {
           return "id 不能负数，请稍后再试，/(ㄒoㄒ)/~~   id: " +id;
       }
   
   }
   ```

2. controller

   ```java
   @RestController
   @Slf4j
   public class PaymentController
   {
       @Resource
       private PaymentService paymentService;
   
       ...
       
       //====服务熔断
       @GetMapping("/payment/circuit/{id}")
       public String paymentCircuitBreaker(@PathVariable("id") Integer id)
       {
           String result = paymentService.paymentCircuitBreaker(id);
           log.info("****result: "+result);
           return result;
       }
   }
   ```

3. 测试

   > 正确 - http://localhost:8001/payment/circuit/1
   >
   > 错误 - http://localhost:8001/payment/circuit/-1
   >
   > 多次错误，再来次正确，但错误得显示
   >
   > 重点测试 - 多次错误，然后慢慢正确，发现刚开始不满足条件，就算是正确的访问地址也不能进行

4. 所有配置

   ```java
   @HystrixCommand(fallbackMethod = "fallbackMethod", 
                   groupKey = "strGroupCommand", 
                   commandKey = "strCommand", 
                   threadPoolKey = "strThreadPool",
                   commandProperties = {
                       // 设置隔离策略，THREAD 表示线程池 SEMAPHORE：信号池隔离
                       @HystrixProperty(name = "execution.isolation.strategy", value = "THREAD"),
                       // 当隔离策略选择信号池隔离的时候，用来设置信号池的大小（最大并发数）
                       @HystrixProperty(name = "execution.isolation.semaphore.maxConcurrentRequests", value = "10"),
                       // 配置命令执行的超时时间
                       @HystrixProperty(name = "execution.isolation.thread.timeoutinMilliseconds", value = "10"),
                       // 是否启用超时时间
                       @HystrixProperty(name = "execution.timeout.enabled", value = "true"),
                       // 执行超时的时候是否中断
                       @HystrixProperty(name = "execution.isolation.thread.interruptOnTimeout", value = "true"),
                       
                       // 执行被取消的时候是否中断
                       @HystrixProperty(name = "execution.isolation.thread.interruptOnCancel", value = "true"),
                       // 允许回调方法执行的最大并发数
                       @HystrixProperty(name = "fallback.isolation.semaphore.maxConcurrentRequests", value = "10"),
                       // 服务降级是否启用，是否执行回调函数
                       @HystrixProperty(name = "fallback.enabled", value = "true"),
                       // 是否启用断路器
                       @HystrixProperty(name = "circuitBreaker.enabled", value = "true"),
                       // 该属性用来设置在滚动时间窗中，断路器熔断的最小请求数。例如，默认该值为 20 的时候，如果滚动时间窗（默认10秒）内仅收到了19个请求， 即使这19个请求都失败了，断路器也不会打开。
                       @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold", value = "20"),
                       
                       // 该属性用来设置在滚动时间窗中，表示在滚动时间窗中，在请求数量超过 circuitBreaker.requestVolumeThreshold 的情况下，如果错误请求数的百分比超过50, 就把断路器设置为 "打开" 状态，否则就设置为 "关闭" 状态。
                       @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage", value = "50"),
                       // 该属性用来设置当断路器打开之后的休眠时间窗。 休眠时间窗结束之后，会将断路器置为 "半开" 状态，尝试熔断的请求命令，如果依然失败就将断路器继续设置为 "打开" 状态，如果成功就设置为 "关闭" 状态。
                       @HystrixProperty(name = "circuitBreaker.sleepWindowinMilliseconds", value = "5000"),
                       // 断路器强制打开
                       @HystrixProperty(name = "circuitBreaker.forceOpen", value = "false"),
                       // 断路器强制关闭
                       @HystrixProperty(name = "circuitBreaker.forceClosed", value = "false"),
                       // 滚动时间窗设置，该时间用于断路器判断健康度时需要收集信息的持续时间
                       @HystrixProperty(name = "metrics.rollingStats.timeinMilliseconds", value = "10000"),
                       
                       // 该属性用来设置滚动时间窗统计指标信息时划分"桶"的数量，断路器在收集指标信息的时候会根据设置的时间窗长度拆分成多个 "桶" 来累计各度量值，每个"桶"记录了一段时间内的采集指标。
                       // 比如 10 秒内拆分成 10 个"桶"收集这样，所以 timeinMilliseconds 必须能被 numBuckets 整除。否则会抛异常
                       @HystrixProperty(name = "metrics.rollingStats.numBuckets", value = "10"),
                       // 该属性用来设置对命令执行的延迟是否使用百分位数来跟踪和计算。如果设置为 false, 那么所有的概要统计都将返回 -1。
                       @HystrixProperty(name = "metrics.rollingPercentile.enabled", value = "false"),
                       // 该属性用来设置百分位统计的滚动窗口的持续时间，单位为毫秒。
                       @HystrixProperty(name = "metrics.rollingPercentile.timeInMilliseconds", value = "60000"),
                       // 该属性用来设置百分位统计滚动窗口中使用 “ 桶 ”的数量。
                       @HystrixProperty(name = "metrics.rollingPercentile.numBuckets", value = "60000"),
                       // 该属性用来设置在执行过程中每个 “桶” 中保留的最大执行次数。如果在滚动时间窗内发生超过该设定值的执行次数，
                       // 就从最初的位置开始重写。例如，将该值设置为100, 滚动窗口为10秒，若在10秒内一个 “桶 ”中发生了500次执行，
                       // 那么该 “桶” 中只保留 最后的100次执行的统计。另外，增加该值的大小将会增加内存量的消耗，并增加排序百分位数所需的计算时间。
                       @HystrixProperty(name = "metrics.rollingPercentile.bucketSize", value = "100"),
                       
                       // 该属性用来设置采集影响断路器状态的健康快照（请求的成功、 错误百分比）的间隔等待时间。
                       @HystrixProperty(name = "metrics.healthSnapshot.intervalinMilliseconds", value = "500"),
                       // 是否开启请求缓存
                       @HystrixProperty(name = "requestCache.enabled", value = "true"),
                       // HystrixCommand的执行和事件是否打印日志到 HystrixRequestLog 中
                       @HystrixProperty(name = "requestLog.enabled", value = "true"),
   
                   },
                   threadPoolProperties = {
                       // 该参数用来设置执行命令线程池的核心线程数，该值也就是命令执行的最大并发量
                       @HystrixProperty(name = "coreSize", value = "10"),
                       // 该参数用来设置线程池的最大队列大小。当设置为 -1 时，线程池将使用 SynchronousQueue 实现的队列，否则将使用 LinkedBlockingQueue 实现的队列。
                       @HystrixProperty(name = "maxQueueSize", value = "-1"),
                       // 该参数用来为队列设置拒绝阈值。 通过该参数， 即使队列没有达到最大值也能拒绝请求。
                       // 该参数主要是对 LinkedBlockingQueue 队列的补充,因为 LinkedBlockingQueue 队列不能动态修改它的对象大小，而通过该属性就可以调整拒绝请求的队列大小了。
                       @HystrixProperty(name = "queueSizeRejectionThreshold", value = "5"),
                   }
                  )
   public String doSomething() {
   	...
   }
   ```

   

# 图形化

1. 新建cloud-consumer-hystrix-dashboard9001

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
   
       <artifactId>cloud-consumer-hystrix-dashboard9001</artifactId>
   
       <dependencies>
           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
           </dependency>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-actuator</artifactId>
           </dependency>
   
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
     port: 9001
   ```

4. 启动类

   > @EnableHystrixDashboard

   ```java
   package com.atguigu.springcloud;
   
   import org.springframework.boot.SpringApplication;
   import org.springframework.boot.autoconfigure.SpringBootApplication;
   import org.springframework.cloud.netflix.hystrix.dashboard.EnableHystrixDashboard;
   
   @SpringBootApplication
   @EnableHystrixDashboard
   public class HystrixDashboardMain9001
   {
       public static void main(String[] args) {
           SpringApplication.run(HystrixDashboardMain9001.class, args);
       }
   }
   ```

5. 所有Provider微服务提供类(8001/8002/8003)都需要监控依赖配置

   ```xml
   <!--web-->
   <dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-web</artifactId>
   </dependency>
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-actuator</artifactId>
   </dependency>
   ```

6. 启动cloud-consumer-hystrix-dashboard9001该微服务

   浏览器输入http://localhost:9001/hystrix

   ![WeChatd439ffe7e60e5561761f57ab04c70f8c](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/WeChatd439ffe7e60e5561761f57ab04c70f8c.png)

## 实战

1. 新版本Hystrix需要在主启动类PaymentHystrixMain8001（被监控的启动类）中指定监控路径

   ```java
   package com.atguigu.springcloud;
   
   import com.netflix.hystrix.contrib.metrics.eventstream.HystrixMetricsStreamServlet;
   import org.springframework.boot.SpringApplication;
   import org.springframework.boot.autoconfigure.SpringBootApplication;
   import org.springframework.boot.web.servlet.ServletRegistrationBean;
   import org.springframework.cloud.client.circuitbreaker.EnableCircuitBreaker;
   import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
   import org.springframework.context.annotation.Bean;
   
   @SpringBootApplication
   @EnableEurekaClient
   @EnableCircuitBreaker
   public class PaymentHystrixMain8001
   {
       public static void main(String[] args) {
           SpringApplication.run(PaymentHystrixMain8001.class, args);
       }
   
   
       /**
        *此配置是为了服务监控而配置，与服务容错本身无关，springcloud升级后的坑
        *ServletRegistrationBean因为springboot的默认路径不是"/hystrix.stream"，
        *只要在自己的项目里配置上下面的servlet就可以了
        *否则，Unable to connect to Command Metric Stream 404
        */
       @Bean
       public ServletRegistrationBean getServlet() {
           HystrixMetricsStreamServlet streamServlet = new HystrixMetricsStreamServlet();
           ServletRegistrationBean registrationBean = new ServletRegistrationBean(streamServlet);
           registrationBean.setLoadOnStartup(1);
           registrationBean.addUrlMappings("/hystrix.stream");
           registrationBean.setName("HystrixMetricsStreamServlet");
           return registrationBean;
       }
   }
   ```

2. 9001监控8001 - 填写监控地址 - http://localhost:8001/hystrix.stream 到 http://localhost:9001/hystrix页面的输入框。

   ![WeChat4d78c7ad26f05cbd3978bde5c6448d67](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/WeChat4d78c7ad26f05cbd3978bde5c6448d67.png)


