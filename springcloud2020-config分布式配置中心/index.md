# springcloud2020 Config分布式配置中心


# springcloud2020 Config分布式配置中心

> ## 分布式配置中心
>
> 分布式系统面临的配置问题
>
> 微服务意味着要将单体应用中的业务拆分成一个个子服务，每个服务的粒度相对较小，因此系统中会出现大量的服务。由于每个服务都需要必要的配置信息才能运行，所以一套集中式的、动态的配置管理设施是必不可少的。
>
> SpringCloud提供了ConfigServer来解决这个问题，我们每一个微服务自己带着一个application.yml，上百个配置文件的管理.……

## Config配置总控中心搭建

1. 在GitHub上新建一个名为springcloud-config的仓库。

   > 我这里使用gitee
   >
   > [codeRabbit214/springcloud-config - 码云 - 开源中国 (gitee.com)](https://gitee.com/coderabbit214/springcloud-config/tree/master)

2. 拉到本地写入三个文件上传

   - config-dev.yml

     ```yml
     config:
       info: "master branch,springcloud-config/config-dev.yml version=7
     ```

   - config-prod.yml

     ```yml
     config:
       info: "master branch,springcloud-config/config-prod.yml version=1"
     ```

   - config-test.yml

     ```yml
     config:
       info: "master branch,springcloud-config/config-test.yml version=1" 
     ```

3. 新建cloud-config-center-3344模块

4. pom

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
   
       <artifactId>cloud-config-center-3344</artifactId>
   
       <dependencies>
           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-config-server</artifactId>
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

5. YML

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
   
   #服务注册到eureka地址
   eureka:
     client:
       service-url:
         defaultZone: http://localhost:7001/eureka
   ```

6. 启动类

   ```java
   package com.atguigu.springcloud;
   
   import org.springframework.boot.SpringApplication;
   import org.springframework.boot.autoconfigure.SpringBootApplication;
   import org.springframework.cloud.config.server.EnableConfigServer;
   
   @SpringBootApplication
   @EnableConfigServer
   public class ConfigCenterMain3344
   {
       public static void main(String[] args) {
           SpringApplication.run(ConfigCenterMain3344.class, args);
       }
   }
   ```

7. 测试

   - 启动3344

   - 浏览器访问[localhost:3344/master/config-dev.yml](http://localhost:3344/master/config-dev.yml)

   - 页面返回

     ```yml
     config:
       info: "master branch,springcloud-config/config-dev.yml version=7"
     ```

8. 配置读取规则

   > 重要配置细节总结
   >
   > /{name}-{profiles}.yml
   > /{label}-{name}-{profiles}.yml
   > label：分支(branch)
   > name：服务名
   > profiles：环境(dev/test/prod)

   - /{label}/{application}-{profile}.yml（推荐）

   - /{application}-{profile}.yml

   - /{application}/{profile}[/{label}]

## Config客户端配置与测试

1. **新建cloud-config-client-3355**

2. **POM**

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
   
       <artifactId>cloud-config-client-3355</artifactId>
   
       <dependencies>
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

3. **bootstrap.yml**

   > applicaiton.yml是用户级的资源配置项
   >
   > bootstrap.yml是系统级的，优先级更加高

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
   
   
   #服务注册到eureka地址
   eureka:
     client:
       service-url:
         defaultZone: http://localhost:7001/eureka
   ```

4. 主启动

   ```java
   package com.atguigu.springcloud;
   
   import org.springframework.boot.SpringApplication;
   import org.springframework.boot.autoconfigure.SpringBootApplication;
   import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
   
   
   @EnableEurekaClient
   @SpringBootApplication
   public class ConfigClientMain3355
   {
       public static void main(String[] args) {
           SpringApplication.run(ConfigClientMain3355.class, args);
       }
   }
   
   ```

5. 业务类

   ```java
   package com.atguigu.springcloud.controller;
   
   import org.springframework.beans.factory.annotation.Value;
   import org.springframework.web.bind.annotation.GetMapping;
   import org.springframework.web.bind.annotation.RestController;
   
   
   @RestController
   public class ConfigClientController
   {
       @Value("${config.info}")
       private String configInfo;
   
       @GetMapping("/configInfo")
       public String getConfigInfo()
       {
           return configInfo;
       }
   }
   
   ```

6. 测试

   > 启动 7001 3344 3355 

   访问[127.0.0.1:3355/configInfo](http://127.0.0.1:3355/configInfo)

   得到返回值

## Config动态刷新之手动版

> 避免每次更新配置都要重启客户端微服务3355

修改3355

1. pom

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-actuator</artifactId>
   </dependency>
   ```

2. yml

   ```yml
   # 暴露监控端点
   management:
     endpoints:
       web:
         exposure:
           include: "*"
   ```

3. @RefreshScope业务类Controller修改

   ```java
   import org.springframework.cloud.context.config.annotation.RefreshScope;
   ...
   
   @RestController
   @RefreshScope//<-----
   public class ConfigClientController
   {
   ...
   }
   ```

4. 在gitee上修改配置文件

   - [127.0.0.1:3355/configInfo](http://127.0.0.1:3355/configInfo)未发生改变
   - 原因 需要刷新

5. 刷新生效

   ```cmd
   curl -X POST "http://localhost:3355/actuator/refresh"
   ```

   


​     

   

   






