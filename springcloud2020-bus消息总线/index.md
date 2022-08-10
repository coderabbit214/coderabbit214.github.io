# springcloud2020 Bus消息总线


# springcloud2020 Bus消息总线

> 分布式自动刷新配置功能

## 环境准备

### 必须先具备良好的RabbitMQ环境 可以打开网页端控制台

### 新建3366

1. 建cloud-config-client-3366

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
   
       <artifactId>cloud-config-client-3366</artifactId>
   
       <dependencies>
           <!--添加消息总线RabbitMQ支持-->
           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-starter-bus-amap</artifactId>
           </dependency>
           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-starter-config</artifactId>
           </dependency>
           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
           </dependency>
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

3. yml（bootstrap）

   ```yml
   server:
     port: 3366
   
   spring:
     application:
       name: config-client
     cloud:
       #Config客户端配置
       config:
         label: master #分支名称
         name: config #配置文件名称
         profile: dev #读取后缀名称   上述3个综合：master分支上config-dev.yml的配置文件被读取http://config-3344.com:3344/master/config-dev.yml
         uri: http://localhost:3344 #配置中心地址k
     #rabbitmq相关配置<--------------------------
     rabbitmq:
       host: localhost
       port: 5672
       username: guest
       password: guest
   
   
   #服务注册到eureka地址
   eureka:
     client:
       service-url:
         defaultZone: http://localhost:7001/eureka
   
   # 暴露监控端点
   management:
     endpoints:
       web:
         exposure:
           include: "*"
   ```

4. 主启动

   ```java
   package com.atguigu.springcloud;
   
   import org.springframework.boot.SpringApplication;
   import org.springframework.boot.autoconfigure.SpringBootApplication;
   import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
   
   @EnableEurekaClient
   @SpringBootApplication
   public class ConfigClientMain3366
   {
       public static void main(String[] args)
       {
           SpringApplication.run(ConfigClientMain3366.class,args);
       }
   }
   
   ```

5. controller

   ```java
   package com.atguigu.springcloud.controller;
   
   import org.springframework.beans.factory.annotation.Value;
   import org.springframework.cloud.context.config.annotation.RefreshScope;
   import org.springframework.web.bind.annotation.GetMapping;
   import org.springframework.web.bind.annotation.RestController;
   
   
   @RestController
   @RefreshScope
   public class ConfigClientController
   {
       @Value("${config.info}")
       private String configInfo;
   
       @Value("${server.port}")
       private String serverPort;
   
       @GetMapping("/configInfo")
       public String getConfigInfo()
       {
           return configInfo+serverPort;
       }
   }
   
   ```

## 设计思想

> 利用消息总线触发一个服务端ConfigServer的/bus/refresh端点，而刷新所有客户端的配置

![e2809f728b8eb3e776883e4f905b8712](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/e2809f728b8eb3e776883e4f905b8712.png)

## Bus动态刷新全局广播配置实现

1. 3344修改

   - pom

     ```xml
     <!--添加消息总线RabbitNQ支持-->
     <dependency>
     	<groupId>org.springframework.cloud</groupId>
     	<artifactId>spring-cloud-starter-bus-amap</artifactId>
     </dependency>
     <dependency>
     	<groupId>org-springframework.boot</groupId>
     	<artifactId>spring-boot-starter-actuator</artifactId>
     </dependency>
     ```

   - yml

     ```yml
     server:
       port: 3344
     
     spring:
       application:
         name:  cloud-config-center #注册进Eureka服务器的微服务名
       cloud:
         config:
           server:
             git:
               uri: https://gitee.com/coderabbit214/springcloud-config.git #GitHub上面的git仓库名字
               ####搜索目录
               search-paths:
                 - springcloud-config
     
           ####读取分支
           label: master
     
     #rabbitmq相关配置<--------------------------
     rabbitmq:
       host: localhost
       port: 5672
       username: guest
       password: guest
     
     ##rabbitmq相关配置,暴露bus刷新配置的端点<--------------------------
     management:
       endpoints: #暴露bus刷新配置的端点
         web:
           exposure:
             include: 'bus-refresh'
     
     #服务注册到eureka地址
     eureka:
       client:
         service-url:
           defaultZone: http://localhost:7001/eureka
     ```

2. 3355修改

   - pom

     ```xml
     <!--添加消息总线RabbitNQ支持-->
     <dependency>
     	<groupId>org.springframework.cloud</groupId>
     	<artifactId>spring-cloud-starter-bus-amap</artifactId>
     </dependency>
     <dependency>
     	<groupId>org-springframework.boot</groupId>
     	<artifactId>spring-boot-starter-actuator</artifactId>
     </dependency>
     ```

   - yml

     ```yml
     server:
       port: 3355
     
     spring:
       application:
         name: config-client
       cloud:
         #Config客户端配置
         config:
           label: master #分支名称
           name: config #配置文件名称
           profile: dev #读取后缀名称   上述3个综合：master分支上config-dev.yml的配置文件被读取http://config-3344.com:3344/master/config-dev.yml
           uri: http://localhost:3344 #配置中心地址k
     #rabbitmq相关配置<--------------------------
       rabbitmq:
         host: localhost
         port: 5672
         username: guest
         password: guest
     
     #服务注册到eureka地址
     eureka:
       client:
         service-url:
           defaultZone: http://localhost:7001/eureka
     
     # 暴露监控端点
     management:
       endpoints:
         web:
           exposure:
             include: "*"
     ```

3. 3366

   - pom

     ```xml
     <!--添加消息总线RabbitNQ支持-->
     <dependency>
     	<groupId>org.springframework.cloud</groupId>
     	<artifactId>spring-cloud-starter-bus-amap</artifactId>
     </dependency>
     <dependency>
     	<groupId>org-springframework.boot</groupId>
     	<artifactId>spring-boot-starter-actuator</artifactId>
     </dependency>
     ```

   - yml

     ```yml
     server:
       port: 3366
     
     spring:
       application:
         name: config-client
       cloud:
         #Config客户端配置
         config:
           label: master #分支名称
           name: config #配置文件名称
           profile: dev #读取后缀名称   上述3个综合：master分支上config-dev.yml的配置文件被读取http://config-3344.com:3344/master/config-dev.yml
           uri: http://localhost:3344 #配置中心地址k
       #rabbitmq相关配置<--------------------------
       rabbitmq:
         host: localhost
         port: 5672
         username: guest
         password: guest
     
     
     #服务注册到eureka地址
     eureka:
       client:
         service-url:
           defaultZone: http://localhost:7001/eureka
     
     # 暴露监控端点
     management:
       endpoints:
         web:
           exposure:
             include: "*"
     ```

4. 测试

   - 启动
     - EurekaMain7001
     - ConfigcenterMain3344
     - ConfigclientMain3355
     - ConfigclicntMain3366
   - 运维工程师
     - 修改Github上配置文件内容，增加版本号
     - 发送POST请求
       - `curl -X POST "http://localhost:3344/actuator/bus-refresh"`
       - **—次发送，处处生效**
   - 配置中心
     - http://localhost:3344/config-dev.yml
   - 客户端
     - http://localhost:3355/configlnfo
     - http://localhost:3366/configInfo
     - 获取配置信息，发现都已经刷新了

   **—次修改，广播通知，处处生效**

   

## Bus动态刷新定点通知

> 不想全部通知，只想定点通知

例：

- 只通知3355
- 不通知3366

运行

- 我们这里以刷新运行在3355端口上的config-client（配置文件中设定的应用名称）为例，只通知3355，不通知3366
- `curl -X POST "http://localhost:3344/actuator/bus-refresh/config-client:3355`

   

