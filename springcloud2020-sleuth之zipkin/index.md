# springcloud2020 Sleuth之zipkin


# springcloud2020 Sleuth之zipkin

> 微服务调用跟踪

## Sleuth之zipkin搭建安装

1. zipkin

   - SpringCloud从F版起已不需要自己构建Zipkin Server了，只需调用jar包即可
   - https://dl.bintray.com/openzipkin/maven/io/zipkin/java/zipkin-server/
   - zipkin-server-2.12.9-exec.jar

2. 运行

   ```
   java -jar zipkin-server-2.12.9-exec.jar
   ```

3. 控制台

   ```
   http://localhost:9411/zipkin/
   ```

## Sleuth链路监控实现

> 为了简单 在cloud-consumer-order80和cloud-provider-payment8001上修改

1. 8001

   - pom

     ```xml
     <!--包含了sleuth+zipkin-->
     <dependency>
         <groupId>org.springframework.cloud</groupId>
         <artifactId>spring-cloud-starter-zipkin</artifactId>
     </dependency>
     
     ```

   - yml

     ```yml
     spring:
       application:
         name: cloud-payment-service #服务名称
     
       zipkin: #<-------------------------------------关键
         base-url: http://localhost:9411
         sleuth: #<-------------------------------------关键
           sampler:
           #采样率值介于 0 到 1 之间，1 则表示全部采集 一般设置0.5
           probability: 1
     
       datasource:
         type: com.alibaba.druid.pool.DruidDataSource  #当前数据源操作类型
         driver-class-name: com.mysql.cj.jdbc.Driver
         url: 
         username: 
         password: 
     ```

   - 业务类PaymentController

     ```java
     @RestController
     @Slf4j
     public class PaymentController {
         
         ...
         
      	@GetMapping("/payment/zipkin")
         public String paymentZipkin() {
             return "hi ,i'am paymentzipkin server fall back，welcome to here, O(∩_∩)O哈哈~";
         }    
     }
     ```

2. 80

   - pom

     ```xml
     <dependency>
         <groupId>org.springframework.cloud</groupId>
         <artifactId>spring-cloud-starter-zipkin</artifactId>
     </dependency>
     ```

   - yml

     ```yml
     spring:
         application:
             name: cloud-order-service
         zipkin:
           base-url: http://localhost:9411
         sleuth:
           sampler:
             probability: 1
     ```

   - 业务类PaymentController

     ```java
         public static final String PAYMENT_URL = "http://CLOUD-PAYMENT-SERVICE";
     
         // ====================> zipkin+sleuth
         @GetMapping("/consumer/payment/zipkin")
         public String paymentZipkin()
         {
             String result = restTemplate.getForObject(PAYMENT_URL+"/payment/zipkin/", String.class);
             return result;
         }
     ```

3. 测试

   - 依次启动eureka7001/8001/80 - 80调用8001几次测试下
   - 打开浏览器访问: http://localhost:9411
   - 查看调用记录


