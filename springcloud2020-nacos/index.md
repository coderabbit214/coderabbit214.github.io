# springcloud2020 nacos


# springcloud2020 nacos

![总体结构图](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/%E6%80%BB%E4%BD%93%E7%BB%93%E6%9E%84%E5%9B%BE.png)

## 简介

1. 为什么叫Nacos
   - 前四个字母分别为Naming和Configuration的前两个字母，最后的s为Service。

2. 是什么

   - 一个更易于构建云原生应用的动态服务发现、配置管理和服务管理平台。
   - Nacos: Dynamic Naming and Configuration Service
   - Nacos就是注册中心＋配置中心的组合 -> Nacos = Eureka+Config+Bus
3. 能干嘛
  - 替代Eureka做服务注册中心
  - 替代Config做服务配置中心

## Nacos之服务提供者注册9001 9002

1. 新建Module - cloudalibaba-provider-payment9001

2. pom

   - 父pom

     ```xml
                 <!--spring cloud alibaba 2.1.0.RELEASE-->
                 <dependency>
                     <groupId>com.alibaba.cloud</groupId>
                     <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                     <version>2.2.0.RELEASE</version>
                     <type>pom</type>
                     <scope>import</scope>
                 </dependency>
     ```

   - 子pom

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
     
         <artifactId>cloudalibaba-provider-payment9001</artifactId>
     
         <dependencies>
             <!--SpringCloud ailibaba nacos -->
             <dependency>
                 <groupId>com.alibaba.cloud</groupId>
                 <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
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
     port: 9001
   
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

4. 主启动

   > @EnableDiscoveryClient

   ```java
   package com.atguigu.springcloud;
   
   import org.springframework.boot.SpringApplication;
   import org.springframework.boot.autoconfigure.SpringBootApplication;
   import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
   
   @EnableDiscoveryClient
   @SpringBootApplication
   public class PaymentMain9001 {
       public static void main(String[] args) {
           SpringApplication.run(PaymentMain9001.class, args);
       }
   }
   
   ```

5. 业务类controller

   ```java
   package com.atguigu.springcloud.controller;
   
   import org.springframework.beans.factory.annotation.Value;
   import org.springframework.web.bind.annotation.GetMapping;
   import org.springframework.web.bind.annotation.PathVariable;
   import org.springframework.web.bind.annotation.RestController;
   
   @RestController
   public class PaymentController {
       @Value("${server.port}")
       private String serverPort;
   
       @GetMapping(value = "/payment/nacos/{id}")
       public String getPayment(@PathVariable("id") Integer id) {
           return "nacos registry, serverPort: "+ serverPort+"\t id"+id;
       }
   }
   
   ```

6. 测试

   - nacos控制台显示
   - http://localhost:9001/payment/nacos/1

7. 为了演示负载均衡 搭建9002

## Nacos之服务消费者注册和负载 83

1. 新建Module - cloudalibaba-consumer-nacos-order83

2. pom

   > 为什么nacos支持负载均衡？因为spring-cloud-starter-alibaba-nacos-discovery内含netflix-ribbon包。

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
   
       <artifactId>cloudalibaba-consumer-nacos-order83</artifactId>
   
       <dependencies>
           <!--openfeign-->
           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-starter-openfeign</artifactId>
           </dependency>
           <!--SpringCloud ailibaba nacos -->
           <dependency>
               <groupId>com.alibaba.cloud</groupId>
               <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
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
     port: 83
   
   spring:
     application:
       name: nacos-order-consumer
     cloud:
       nacos:
         discovery:
           server-addr: localhost:8848
   
   #消费者将要去访问的微服务名称(注册成功进nacos的微服务提供者)
   service-url:
     nacos-user-service: http://nacos-payment-provider
   
   ```

4. 启动类

   ```java
   package springcloud;
   
   import org.springframework.boot.SpringApplication;
   import org.springframework.boot.autoconfigure.SpringBootApplication;
   import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
   import org.springframework.cloud.openfeign.EnableFeignClients;
   
   @EnableDiscoveryClient
   @SpringBootApplication
   @EnableFeignClients
   public class OrderNacosMain83 {
       public static void main(String[] args) {
           SpringApplication.run(OrderNacosMain83.class, args);
       }
   }
   
   ```

5. 业务类

   - service

     ```java
     package springcloud.service;
     
     import org.springframework.cloud.openfeign.FeignClient;
     import org.springframework.stereotype.Component;
     import org.springframework.web.bind.annotation.GetMapping;
     import org.springframework.web.bind.annotation.PathVariable;
     
     @Component
     @FeignClient(value = "nacos-payment-provider")
     public interface OrderService {
         @GetMapping(value = "/payment/nacos/{id}")
         public String getPayment(@PathVariable("id") Integer id);
     }
     
     ```

   - controller

     ```java
     package springcloud.controller;
     
     import lombok.extern.slf4j.Slf4j;
     import org.springframework.beans.factory.annotation.Autowired;
     import org.springframework.beans.factory.annotation.Value;
     import org.springframework.web.bind.annotation.GetMapping;
     import org.springframework.web.bind.annotation.PathVariable;
     import org.springframework.web.bind.annotation.RestController;
     import org.springframework.web.client.RestTemplate;
     import springcloud.service.OrderService;
     
     import javax.annotation.Resource;
     
     @RestController
     @Slf4j
     public class OrderNacosController {
     
     //    @Resource
     //    private RestTemplate restTemplate;
         @Autowired
         private OrderService orderService;
     
         @Value("${service-url.nacos-user-service}")
         private String serverURL;
     
         @GetMapping(value = "/consumer/payment/nacos/{id}")
         public String paymentInfo(@PathVariable("id") Long id)
         {
             return orderService.getPayment(Math.toIntExact(id));
         }
     
     }
     
     ```

6. 测试
   - 启动nacos控制台
   - http://localhost:83/Eonsumer/payment/nacos/13
   - 83访问9001/9002，轮询负载OK

## **Nacos和CAP**

> nacos可以切换ap cp

—般来说，如果不需要存储服务级别的信息且服务实例是通过nacos-client注册，并能够保持心跳上报，那么就可以选择AP模式。当前主流的服务如Spring cloud和Dubbo服务，都适用于AP模式，AP模式为了服务的可能性而减弱了一致性，因此AP模式下只支持注册临时实例。

如果需要在服务级别编辑或者存储配置信息，那么CP是必须，K8S服务和DNS服务则适用于CP模式。CP模式下则支持注册持久化实例，此时则是以Raft协议为集群运行模式，该模式下注册实例之前必须先注册服务，如果服务不存在，则会返回错误。

切换命令：

```
curl -X PUT '$NACOS_SERVER:8848/nacos/v1/ns/operator/switches?entry=serverMode&value=CP
```



## Nacos之服务配置中心

### 基础配置3377

1. 建module   cloudalibaba-config-nacos-client3377

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
   
       <artifactId>cloudalibaba-config-nacos-client3377</artifactId>
   
       <dependencies>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-security</artifactId>
           </dependency>
           <!--nacos-config-->
           <dependency>
               <groupId>com.alibaba.cloud</groupId>
               <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
           </dependency>
           <!--nacos-discovery-->
           <dependency>
               <groupId>com.alibaba.cloud</groupId>
               <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
           </dependency>
           <!--web + actuator-->
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-web</artifactId>
           </dependency>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-actuator</artifactId>
           </dependency>
           <!--一般基础配置-->
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

   - bootstrap

     ```yml
     # nacos配置
     server:
       port: 3377
     
     spring:
       application:
         name: nacos-config-client
       cloud:
         nacos:
           discovery:
             server-addr: localhost:8848 #Nacos服务注册中心地址
           config:
             server-addr: localhost:8848 #Nacos作为配置中心地址
             file-extension: yaml #指定yaml格式的配置
     #        group: TEST_GROUP
     #        namespace: abf23dd4-f61a-470a-bb92-34b6879f5438
     
     
     
     # ${spring.application.name}-${spring.profile.active}.${spring.cloud.nacos.config.file-extension}
     # nacos-config-client-dev.yaml
     ```

   - application

     ```yml
     spring:
       profiles:
         active: dev # 表示开发环境
     #    active: test # 表示测试环境
         #active: prod # 表示生产环境
     #    active: info
     ```

4. 主启动

   ```java
   package springcloud;
   
   import org.springframework.boot.SpringApplication;
   import org.springframework.boot.autoconfigure.SpringBootApplication;
   import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
   
   @SpringBootApplication
   @EnableDiscoveryClient
   public class NacosConfigClientMain3377 {
       public static void main(String[] args) {
           SpringApplication.run(NacosConfigClientMain3377.class,args);
       }
   }
   
   ```

5. 业务类

   ```java
   package springcloud.controller;
   import org.springframework.beans.factory.annotation.Value;
   import org.springframework.cloud.context.config.annotation.RefreshScope;
   import org.springframework.web.bind.annotation.GetMapping;
   import org.springframework.web.bind.annotation.RestController;
   
   @RestController
   @RefreshScope //支持Nacos的动态刷新功能。
   public class ConfigClientController
   {
       @Value("${config.info}")
       private String configInfo;
   
       @GetMapping("/config/info")
       public String getConfigInfo() {
           return configInfo;
       }
   }
   
   ```

6. **在Nacos中添加配置信息**

   - Nacos中的dataid的组成格式及与SpringBoot配置文件中的匹配规则

     > 说明：之所以需要配置spring.application.name，是因为它是构成Nacos配置管理dataId 字段的一部分。
     >
     > 在 Nacos Spring Cloud中,dataId的完整格式如下：
     >
     > ```
     > ${prefix}-${spring-profile.active}.${file-extension}
     > ```
     >
     > - prefix默认为spring.application.name的值，也可以通过配置项spring.cloud.nacos.config.prefix来配置。
     > - spring.profile.active即为当前环境对应的 profile，详情可以参考 Spring Boot文档。注意：当spring.profile.active为空时，对应的连接符 - 也将不存在，datald 的拼接格式变成${prefix}.${file-extension}
     > - file-exetension为配置内容的数据格式，可以通过配置项spring .cloud.nacos.config.file-extension来配置。目前只支持properties和yaml类型。
     > - 通过Spring Cloud 原生注解@RefreshScope实现配置自动更新。

   - 配置

![05d45948bf637614dbd70e2bc8ce992d](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/05d45948bf637614dbd70e2bc8ce992d.png)



![c61619bbe5ea16f34efca8103b0f90ba](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/c61619bbe5ea16f34efca8103b0f90ba.png)

![b3bffc4a646b30f9bf64fc649bf26f7d](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/b3bffc4a646b30f9bf64fc649bf26f7d.png)

7. 测试

   - 调用接口查看配置信息 - http://localhost:3377/config/info

8. **自带动态刷新**

   > 修改下Nacos中的yaml配置文件，再次调用查看配置的接口，就会发现配置已经刷新。

### Nacos之命名空间分组和DataID三者关系

实际开发中，通常一个系统会准备

1. dev开发环境
2. test测试环境
3. prod生产环境。

如何保证指定环境启动时服务能正确读取到Nacos上相应环境的配置文件呢?



一个大型分布式微服务系统会有很多微服务子项目，每个微服务项目又都会有相应的开发环境、测试环境、预发环境、正式环境…那怎么对这些微服务配置进行管理呢?![3a7d1ad9bea8356742997ed3ebbe9be3](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/3a7d1ad9bea8356742997ed3ebbe9be3.png)

![fe336f99f44c4b0aefddf0ae38d1c470](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/fe336f99f44c4b0aefddf0ae38d1c470.png)

**Namespace+Group+Data lD三者关系？为什么这么设计？**

1. 是什么

   类似Java里面的package名和类名最外层的namespace是可以用于区分部署环境的，Group和DatalD逻辑上区分两个目标对象。

2. 三者情况

   ![60712abd615dd86ac6c119bf132a28d6](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/60712abd615dd86ac6c119bf132a28d6.png)

   默认情况：Namespace=public，Group=DEFAULT_GROUP，默认Cluster是DEFAULT

   - acos默认的Namespace是public，Namespace主要用来实现隔离。

     > 比方说我们现在有三个环境：开发、测试、生产环境，我们就可以创建三个Namespace，不同的Namespace之间是隔离的。

   - Group默认是DEFAULT_GROUP，Group可以把不同的微服务划分到同一个分组里面去

   - Service就是微服务:一个Service可以包含多个Cluster (集群)，Nacos默认Cluster是DEFAULT，Cluster是对指定微服务的一个虚拟划分。

   - Instance，就是微服务的实例。



### Nacos之DataID配置

指定spring.profile.active和配置文件的DatalD来使不同环境下读取不同的配置

默认空间+默认分组+新建dev和test两个DatalD

 ![b41fe36b41fa2d5abc6e5e492ee3625d](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/b41fe36b41fa2d5abc6e5e492ee3625d.png)

通过spring.profile.active属性就能进行多环境下配置文件的读取

![281a70d387cb48ce82e94421adf17747](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/281a70d387cb48ce82e94421adf17747.png)

测试

- http://localhost:3377/config/info
- 配置是什么就加载什么 test/dev

### Nacos之Group分组方案

> 通过Group实现环境区分

新建配置

![bdf592aa566fe50f7f454118a70ca03c](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/bdf592aa566fe50f7f454118a70ca03c.png)

![28aee2b45901bbb9a6776d5c4398a6bb](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/28aee2b45901bbb9a6776d5c4398a6bb.png)

bootstrap+application

在config下增加一条group的配置即可。可配置为DEV_GROUP或TEST GROUP

![342a167a8bd948d8ba5cbfd760cf66a6](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/342a167a8bd948d8ba5cbfd760cf66a6.png)

### Nacos之Namespace空间方案

新建dev/test的Namespace![a10c71978c75c214aca5fa7057bb2834](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/a10c71978c75c214aca5fa7057bb2834.png)

回到服务管理-服务列表查看

![2a9f3fa415f5cead0219d404a47131a0](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/2a9f3fa415f5cead0219d404a47131a0.png)

按照域名配置填写

![2177c126090c0db553a8ce77e838a7c9](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/2177c126090c0db553a8ce77e838a7c9.png)

yml

```yml
# nacos配置
server:
  port: 3377

spring:
  application:
    name: nacos-config-client
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848 #Nacos服务注册中心地址
      config:
        server-addr: localhost:8848 #Nacos作为配置中心地址
        file-extension: yaml #指定yaml格式的配置
        group: TEST_GROUP
        namespace: abf23dd4-f61a-470a-bb92-34b6879f5438



# ${spring.application.name}-${spring.profile.active}.${spring.cloud.nacos.config.file-extension}
# nacos-config-client-dev.yaml
```

### 总而言之 DataID+Group+Namespace确定读取哪个配置文件

## nacos持久化

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

## Nacos集群

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

