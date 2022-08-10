# springcloud2020 cloud-consumer-order80 消费者


# springcloud2020 cloud-consumer-order80 消费者

## 1.创建

1. 建立module

   ```xml
    <parent>
           <artifactId>cloud2020</artifactId>
           <groupId>com.atguigu.springcloud</groupId>
           <version>1.0-SNAPSHOT</version>
       </parent>
       <modelVersion>4.0.0</modelVersion>
   
       <artifactId>cloud-consumer-order80</artifactId>
   ```

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
   
       <artifactId>cloud-consumer-order80</artifactId>
       
       <dependencies>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-web</artifactId>
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
     port: 80
   ```
   
4. 主启动

   ```java
   package com.atguigu.springcloud;
   
   import org.springframework.boot.SpringApplication;
   import org.springframework.boot.autoconfigure.SpringBootApplication;
   
   @SpringBootApplication
   public class OrderMain80 {
       public static void main(String[] args) {
           SpringApplication.run(OrderMain80.class,args);
       }
   }
   
   ```

## 2.业务

1. entities

   > cloud-provider-payment8001 中复制粘贴

   - Payment.java
   - CommonResult.java

2. **RestTemplate**

   > RestTemplate提供了多种便捷访问远程Http服务的方法，是一种简单便捷的访问restful服务模板类，是Spring提供的用于访问Rest服务的客户端模板工具集
   >
   > 
   >
   > 使用：
   >
   > - 使用restTemplate访问restful接口非常的简单粗暴无脑。
   > - `(url, requestMap, ResponseBean.class)`这三个参数分别代表。
   > - REST请求地址、请求参数、HTTP响应转换被转换成的对象类型。

3. **RestTemplate**配置

   ```java
   package com.atguigu.springcloud.config;
   
   import org.springframework.context.annotation.Bean;
   import org.springframework.context.annotation.Configuration;
   import org.springframework.web.client.RestTemplate;
   
   @Configuration
   public class ApplicationContextConfig {
       @Bean
       public RestTemplate getRestTemplate(){
           return new RestTemplate();
       }
   }
   
   ```

4. **RestTemplate**使用 controller

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
       public static final String PAYMENT_URL = "http://localhost:8001";
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

5. 测试出现问题

   > 问题：create方法无法保存信息（数据传输无法接收）
   >
   > 解决：在cloud-provider-payment8001工程的`PaymentController`中create添加`@RequestBody`注解。

6. ```java
   package com.atguigu.springcloud.controller;
   
   import com.atguigu.springcloud.entities.CommonResult;
   import com.atguigu.springcloud.entities.Payment;
   import com.atguigu.springcloud.service.PaymentService;
   import lombok.extern.slf4j.Slf4j;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.web.bind.annotation.*;
   
   @RestController
   @Slf4j
   @RequestMapping("/payment")
   public class PaymentController {
       @Autowired
       private PaymentService paymentService;
   
       @PostMapping(value = "/create")
       public CommonResult create(@RequestBody Payment payment){
           int result = paymentService.create(payment);
           log.info("结果："+ result);
           if (result > 0){
               return new CommonResult(200,"插入成功",result);
           }
           return new CommonResult(444,"插入错误",null);
       }
       @GetMapping(value = "/get/{id}")
       public CommonResult getPamentById(@PathVariable("id")Long id){
           Payment paymentById = paymentService.getPaymentById(id);
           log.info("结果："+ paymentById);
           if (paymentById != null){
               return new CommonResult(200,"查询成功",paymentById);
           }
           return new CommonResult(444,"没有对应记录"+id,null);
       }
   }
   
   ```

# 代码重构（重复了entities）

## 1.新建cloud-api-commons

1. 建立module

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
   
       <artifactId>cloud-api-commons</artifactId>
   
       <dependencies>
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
               <groupId>cn.hutool</groupId>
               <artifactId>hutool-all</artifactId>
               <version>5.1.0</version>
           </dependency>
       </dependencies>
   </project>
   ```

3. entities

   > 将cloud-consumer-order80与cloud-provider-payment8001两工程的公有entities包移至cloud-api-commons工程下

4. 初始化cloud-api-commons

   ```
   maven clean install
   ```

5. 订单80和支付8001分别改造

   - 将cloud-consumer-order80与cloud-provider-payment8001两工程的公有entities包移除

   - 引入cloud-api-commons依赖

     ```xml
     <dependency>
         <groupId>com.lun.springcloud</groupId>
         <artifactId>cloud-api-commons</artifactId>
         <version>${project.version}</version>
     </dependency>
     ```

     


