# springcloud2020 OpenFeign


# springcloud2020 OpenFeign

> Feign可以与Eureka和Ribbon组合使用以支持负载均衡

## OpenFeign服务调用

1. 新建cloud-consumer-feign-order80

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
   
       <artifactId>cloud-consumer-feign-order80</artifactId>
   
       <dependencies>
           <!--openfeign-->
           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-starter-openfeign</artifactId>
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
   
   spring:
     application:
       name: cloud-order-service
   
   eureka:
     client:
       register-with-eureka: false
       service-url:
         defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/
   
   ```

4. 主启动

   > @EnableFeignClients

   ```java
   package com.atguigu.springcloud;
   
   import org.springframework.boot.SpringApplication;
   import org.springframework.boot.autoconfigure.SpringBootApplication;
   import org.springframework.cloud.openfeign.EnableFeignClients;
   
   @SpringBootApplication
   @EnableFeignClients
   public class OrderFeignMain80 {
       public static void main(String[] args) {
           SpringApplication.run(OrderFeignMain80.class, args);
       }
   }
   
   ```

5. 业务类service

   > 直接复制对应微服务的controller 
   >
   > 注意⚠️：  请求路径注意复制完整

   ```java
   package com.atguigu.springcloud.service;
   
   import com.atguigu.springcloud.entities.CommonResult;
   import com.atguigu.springcloud.entities.Payment;
   import org.springframework.cloud.openfeign.FeignClient;
   import org.springframework.stereotype.Component;
   import org.springframework.web.bind.annotation.GetMapping;
   import org.springframework.web.bind.annotation.PathVariable;
   
   @Component
   @FeignClient(value = "cloud-payment-service")
   public interface PaymentFeignService {
   
       @GetMapping(value = "/payment/get/{id}")
       public CommonResult getPamentById(@PathVariable("id")Long id);
   }
   
   ```

6. controller

   ```java
   package com.atguigu.springcloud.controller;
   
   import com.atguigu.springcloud.entities.CommonResult;
   import com.atguigu.springcloud.entities.Payment;
   import com.atguigu.springcloud.service.PaymentFeignService;
   import lombok.extern.slf4j.Slf4j;
   import org.springframework.web.bind.annotation.GetMapping;
   import org.springframework.web.bind.annotation.PathVariable;
   import org.springframework.web.bind.annotation.RestController;
   import javax.annotation.Resource;
   
   @RestController
   @Slf4j
   public class OrderFeignController
   {
       @Resource
       private PaymentFeignService paymentFeignService;
   
       @GetMapping(value = "/consumer/payment/get/{id}")
       public CommonResult<Payment> getPaymentById(@PathVariable("id") Long id)
       {
           return paymentFeignService.getPamentById(id);
       }
   
   }
   ```

## OpenFeign超时控制

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

## OpenFeign日志增强

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

