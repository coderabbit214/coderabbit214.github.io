# springcloud2020 Eureka


# springcloud2020 Eureka

> 么是服务治理
>
> Spring Cloud封装了Netflix 公司开发的Eureka模块来实现服务治理
>
> 在传统的RPC远程调用框架中，管理每个服务与服务之间依赖关系比较复杂，管理比较复杂，所以需要使用服务治理，管理服务于服务之间依赖关系，可以实现服务调用、负载均衡、容错等，实现服务发现与注册。
>
> 什么是服务注册与发现
>
> Eureka采用了CS的设计架构，Eureka Sever作为服务注册功能的服务器，它是服务注册中心。而系统中的其他微服务，使用Eureka的客户端连接到 Eureka Server并维持心跳连接。这样系统的维护人员就可以通过Eureka Server来监控系统中各个微服务是否正常运行。
>
> 在服务注册与发现中，有一个注册中心。当服务器启动的时候，会把当前自己服务器的信息比如服务地址通讯地址等以别名方式注册到注册中心上。另一方(消费者服务提供者)，以该别名的方式去注册中心上获取到实际的服务通讯地址，然后再实现本地RPC调用RPC远程调用框架核心设计思想:在于注册中心，因为使用注册中心管理每个服务与服务之间的一个依赖关系(服务治理概念)。在任何RPC远程框架中，都会有一个注册中心存放服务地址相关信息(接口地址)
>
> 
>
> Eureka包含两个组件:Eureka Server和Eureka Client
>
> Eureka Server提供服务注册服务
>
> 各个微服务节点通过配置启动后，会在EurekaServer中进行注册，这样EurekaServer中的服务注册表中将会存储所有可用服务节点的信息，服务节点的信息可以在界面中直观看到。
>
> EurekaClient通过注册中心进行访问
>
> 它是一个Java客户端，用于简化Eureka Server的交互，客户端同时也具备一个内置的、使用轮询(round-robin)负载算法的负载均衡器。在应用启动后，将会向Eureka Server发送心跳(默认周期为30秒)。如果Eureka Server在多个心跳周期内没有接收到某个节点的心跳，EurekaServer将会从服务注册表中把这个服务节点移除（默认90秒)

## 1.服务端安装

1. 创建cloud-eureka-server7001的module

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
   
       <artifactId>cloud-eureka-server7001</artifactId>
   
       <dependencies>
           <!--eureka-server-->
           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
           </dependency>
           <!-- 引入自己定义的api通用包，可以使用Payment支付Entity -->
           <dependency>
               <groupId>com.atguigu.springcloud</groupId>
               <artifactId>cloud-api-commons</artifactId>
               <version>1.0-SNAPSHOT</version>
           </dependency>
           <!--boot web actuator-->
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-web</artifactId>
           </dependency>
   <!--        图像监控-->
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-actuator</artifactId>
           </dependency>
           <!--一般通用配置-->
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-devtools</artifactId>
               <scope>runtime</scope>
               <optional>true</optional>
           </dependency>
           <dependency>
               <groupId>org.projectlombok</groupId>
               <artifactId>lombok</artifactId>
           </dependency>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-test</artifactId>
               <scope>test</scope>
           </dependency>
           <dependency>
               <groupId>junit</groupId>
               <artifactId>junit</artifactId>
           </dependency>
       </dependencies>
   
   </project>
   ```

3. yml

   ```yml
   server:
     port: 7001
   
   eureka:
     instance:
       hostname: locathost #eureka服务端的实例名称
     client:
       #false表示不向注册中心注册自己。
       register-with-eureka: false
       #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
       fetch-registry: false
       service-url:
         #设置与Eureka server交互的地址查询服务和注册服务都需要依赖这个地址。
         defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
   ```

4. 启动类

   > @EnableEurekaServer 

   ```java
   package com.atguigu.springcloud;
   
   import org.springframework.boot.SpringApplication;
   import org.springframework.boot.autoconfigure.SpringBootApplication;
   import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;
   
   @SpringBootApplication
   @EnableEurekaServer
   public class EurekaMain7001 {
       public static void main(String[] args) {
           SpringApplication.run(EurekaMain7001.class,args);
       }
   }
   
   ```

5. 启动

   >  浏览器输入`http://localhost:7001/`进入pring Eureka服务主页

## 2.支付微服务8001入驻进EurekaServer

1. 修改cloud-provider-payment8001 pom

   ```xml
   <!--        eureka-client-->
           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
           </dependency>
   
   ```

2. yml

   ```yml
   eureka:
     client:
       #表示是否将自己注册进Eurekaserver默认为true。
       register-with-eureka: true
       #是否从EurekaServer抓取已有的注册信息，默认为true。单节点无所谓，集群必须设置为true才能配合ribbon使用负载均衡
       fetchRegistry: true
       #入住到哪里 那个主机的那个端口
       service-url:
         defaultZone: http://localhost:7001/eureka
   ```

3. 主启动

   > @EnableEurekaClient

   ```java
   package com.atguigu.springcloud;
   
   import org.springframework.boot.SpringApplication;
   import org.springframework.boot.autoconfigure.SpringBootApplication;
   import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
   
   @SpringBootApplication
   @EnableEurekaClient
   public class PaymentMain8001 {
       public static void main(String[] args) {
           SpringApplication.run(PaymentMain8001.class,args);
       }
   }
   
   ```

4. 测试

   ![WeChat04449cfafc48ad28769f57afb32ca11d](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/WeChat04449cfafc48ad28769f57afb32ca11d.png)

## 3.订单微服务80入驻进EurekaServer

1. pom

   ```xml
   <!--        eureka-client-->
           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
           </dependency>
   
   ```

2. yml

   > 注意添加服务名称

   ```yml
   server:
     port: 80
   
   spring:
     application:
       name: cloud-order-service
   
   eureka:
     client:
       #表示是否将自己注册进Eurekaserver默认为true。
       register-with-eureka: true
       #是否从EurekaServer抓取已有的注册信息，默认为true。单节点无所谓，集群必须设置为true才能配合ribbon使用负载均衡
       fetchRegistry: true
       #入住到哪里 那个主机的那个端口
       service-url:
         defaultZone: http://localhost:7001/eureka
   ```

3. 主启动

   > @EnableEurekaClient

   ```java
   package com.atguigu.springcloud;
   
   import org.springframework.boot.SpringApplication;
   import org.springframework.boot.autoconfigure.SpringBootApplication;
   import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
   
   @SpringBootApplication
   @EnableEurekaClient
   public class OrderMain80 {
       public static void main(String[] args) {
           SpringApplication.run(OrderMain80.class,args);
       }
   }
   
   ```

4. 测试

   ------

# Eureka集群环境构建

## 1.环境构建

1. 按照7001创建cloud-eureka-server7002工程

2. 修改本机etc\hosts文件 模拟两台机器

   ```
   127.0.0.1 eureka7001.com
   127.0.0.1 eureka7002.com
   ```

3. 修改cloud-eureka-server7001配置文件

   > hostname  defaultZone

   ```yml
   server:
     port: 7001
   
   eureka:
     instance:
       hostname: eureka7001.com  #eureka服务端的实例名称
     client:
       #false表示不向注册中心注册自己。
       register-with-eureka: false
       #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
       fetch-registry: false
       service-url:
         #设置与Eureka server交互的地址查询服务和注册服务都需要依赖这个地址。
         defaultZone: http://eureka7002.com:7002/eureka/
   ```

4. 修改cloud-eureka-server7002配置文件

   ```yml
   server:
     port: 7002
   
   eureka:
     instance:
       hostname: eureka7002.com #eureka服务端的实例名称
     client:
       #false表示不向注册中心注册自己。
       register-with-eureka: false
       #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
       fetch-registry: false
       service-url:
         #设置与Eureka server交互的地址查询服务和注册服务都需要依赖这个地址。
         defaultZone: http://eureka7001.com:7001/eureka/
   ```

5. 启动7001和7002

   ![WeChat3fe044657ce34190365c6919a7c58325](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/WeChat3fe044657ce34190365c6919a7c58325.png)

   ![WeChatfccbab63e8e0a14dff43a615b6f48eb4](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/WeChatfccbab63e8e0a14dff43a615b6f48eb4.png)

## 2.微服务注册进Eureka集群

支付服务8001微服务，订单服务80微服务

将它们的配置文件的eureka.client.service-url.defaultZone进行修改

```yml
eureka:
  client:
    #表示是否将自己注册进Eurekaserver默认为true。
    register-with-eureka: true
    #是否从EurekaServer抓取已有的注册信息，默认为true。单节点无所谓，集群必须设置为true才能配合ribbon使用负载均衡
    fetchRegistry: true
    #入住到哪里 那个主机的那个端口
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka, http://eureka7002.com:7002/eureka
#      defaultZone: http://localhost:7001/eureka
```

## 3.支付微服务集群配置

1. 新建cloud-provider-payment8002 模拟多台机器

2. 改POM

3. 写YML - 端口8002

4. 主启动

5. 业务类

6. 修改8001/8002的Controller，添加serverPort(目的是为了查看80端口调用哪个微服务)

   ```java
   package com.atguigu.springcloud.controller;
   
   import com.atguigu.springcloud.entities.CommonResult;
   import com.atguigu.springcloud.entities.Payment;
   import com.atguigu.springcloud.service.PaymentService;
   import lombok.extern.slf4j.Slf4j;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.beans.factory.annotation.Value;
   import org.springframework.web.bind.annotation.*;
   
   @RestController
   @Slf4j
   @RequestMapping("/payment")
   public class PaymentController {
       @Autowired
       private PaymentService paymentService;
   
       @Value("${server.port}")
       private String serverPort;
   
       @PostMapping(value = "/create")
       public CommonResult create(@RequestBody Payment payment){
           int result = paymentService.create(payment);
           log.info("结果："+ result);
           if (result > 0){
               return new CommonResult(200,"插入成功"+serverPort,result);
           }
           return new CommonResult(444,"插入错误",null);
       }
       @GetMapping(value = "/get/{id}")
       public CommonResult getPamentById(@PathVariable("id")Long id){
           Payment paymentById = paymentService.getPaymentById(id);
           log.info("结果："+ paymentById);
           if (paymentById != null){
               return new CommonResult(200,"查询成功"+serverPort,paymentById);
           }
           return new CommonResult(444,"没有对应记录"+id,null);
       }
   }
   
   ```

7. 负载均衡 80controller

   > 订单服务访问地址不能写死 
   >
   > 注意：
   >
   > public static final String PAYMENT_URL = "http://CLOUD-PAYMENT-SERVICE";
   
   ```java
   package com.atguigu.springcloud.controller;
   
   import com.atguigu.springcloud.entities.CommonResult;
   import com.atguigu.springcloud.entities.Payment;
   import lombok.extern.slf4j.Slf4j;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.web.bind.annotation.*;
   import org.springframework.web.client.RestTemplate;
   
   @RestController
   @Slf4j
   public class PaymentController {
       public static final String PAYMENT_URL = "http://CLOUD-PAYMENT-SERVICE";
   //    public static final String PAYMENT_URL = "http://localhost:8001";
       @Autowired
       private RestTemplate restTemplate;
   
       @PostMapping("/consumer/payment/create")
       public CommonResult<Payment> create(Payment payment){
           return restTemplate.postForObject(PAYMENT_URL+"/payment/create",payment,CommonResult.class);
       }
       @GetMapping("/consumer/payment/get/{id}")
       public CommonResult<Payment> getPaymentById(@PathVariable("id") Long id){
           return restTemplate.getForObject(PAYMENT_URL+"/payment/get/"+id,CommonResult.class);
       }
   }
   
   ```

   修改配置 使用@LoadBalanced注解赋予RestTemplate负载均衡的能力
   
   **@LoadBalanced**
   
   ```java
   package com.atguigu.springcloud.config;
   
   import org.springframework.cloud.client.loadbalancer.LoadBalanced;
   import org.springframework.context.annotation.Bean;
   import org.springframework.context.annotation.Configuration;
   import org.springframework.web.client.RestTemplate;
   
   @Configuration
   public class ApplicationContextConfig {
       @Bean
       @LoadBalanced
       public RestTemplate getRestTemplate(){
           return new RestTemplate();
       }
   }
   
   ```
   
   > 负载均衡效果达到，8001/8002端口交替出现

## 4.actuator微服务信息完善

> 修改服务名称 显示ip

修改cloud-provider-payment8001，cloud-provider-payment8002

yml 俩个都需要修改

> instance-id: payment8001 
>
> instance-id: payment8002

```yml
eureka:
  ...
  instance:
    instance-id: payment8001 #自定义微服务名称
    prefer-ip-address: true #显示ip 好像不设置也会显示
```

## 5.服务发现Discovery

> 对于注册进eureka里面的微服务，可以通过服务发现来获得该服务的信息

1. 修改cloud-provider-payment8001的Controller

   ```java
   package com.atguigu.springcloud.controller;
   
   import com.atguigu.springcloud.entities.CommonResult;
   import com.atguigu.springcloud.entities.Payment;
   import com.atguigu.springcloud.service.PaymentService;
   import lombok.extern.slf4j.Slf4j;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.beans.factory.annotation.Value;
   import org.springframework.cloud.client.ServiceInstance;
   import org.springframework.cloud.client.discovery.DiscoveryClient;
   import org.springframework.web.bind.annotation.*;
   
   import java.util.List;
   
   @RestController
   @Slf4j
   @RequestMapping("/payment")
   public class PaymentController {
     ...
       @Autowired
       private DiscoveryClient discoveryClient;
   
     ...
       @GetMapping("/discovery")
       public Object discovery(){
           List<String> services = discoveryClient.getServices();//所有微服务名称
           for (String service:services) {
               log.info(service);
           }
           List<ServiceInstance> instances = discoveryClient.getInstances("CLOUD-PAYMENT-SERVICE");//此微服务名称下的 所有微服务名称
           for (ServiceInstance instance:instances) {
               log.info(instance.getServiceId()+"\t"+instance.getHost()+"\t"+instance.getPort()+"\t"+instance.getUri());
           }
           return this.discoveryClient;
       }
   }
   
   ```

2. 8001启动类

   > @EnableDiscoveryClient注解

   ```java
   package com.atguigu.springcloud;
   
   import org.springframework.boot.SpringApplication;
   import org.springframework.boot.autoconfigure.SpringBootApplication;
   import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
   import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
   
   @SpringBootApplication
   @EnableEurekaClient
   @EnableDiscoveryClient
   public class PaymentMain8001 {
       public static void main(String[] args) {
           SpringApplication.run(PaymentMain8001.class,args);
       }
   }
   ```

## 6. Eureka自我保护

- 现象

![WeChatb3d732fb4550fcc1429324286d2b6a18](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/WeChatb3d732fb4550fcc1429324286d2b6a18.png)

- 解释

> 某时刻某一个微服务不可用了，Eureka不会立刻清理，依旧会对该微服务的信息进行保存。

- 如何禁止自我保护

  7001yml

  ```yml
  eureka:
    ...
    server:
      #关闭自我保护机制，保证不可用服务被及时踢除
      enable-self-preservation: false
      eviction-interval-timer-in-ms: 2000
  ```

  客户端（8001）相应设置yml

  ```yml
  eureka:
    ...
    instance:
      instance-id: payment8001
      prefer-ip-address: true
      #心跳检测与续约时间
      #开发时没置小些，保证服务关闭后注册中心能即使剔除服务
      #Eureka客户端向服务端发送心跳的时间间隔，单位为秒(默认是30秒)
      lease-renewal-interval-in-seconds: 1
      #Eureka服务端在收到最后一次心跳后等待时间上限，单位为秒(默认是90秒)，超时将剔除服务
      lease-expiration-duration-in-seconds: 2
  ```

# 停更说明

[停更](https://github.com/Netflix/eureka/wiki)


