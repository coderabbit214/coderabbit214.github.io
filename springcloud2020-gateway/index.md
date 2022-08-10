# springcloud2020 gateway


# springcloud2020 gateway

> 服务网关
>
> **SpringCloud Gateway是基于WebFlux框架实现的，而WebFlux框架底层则使用了高性能的Reactor模式通信框架Netty**。
>
> **作用**
>
> - 方向代理
> - 鉴权
> - 流量控制
> - 熔断
> - 日志监控

## GateWay非阻塞异步模型

SpringCloud Gateway具有如下特性

1. 基于Spring Framework 5，Project Reactor和Spring Boot 2.0进行构建；
2. 动态路由：能够匹配任何请求属性；
3. 可以对路由指定Predicate (断言)和Filter(过滤器)；
4. 集成Hystrix的断路器功能；
5. 集成Spring Cloud 服务发现功能；
6. 易于编写的Predicate (断言)和Filter (过滤器)；
7. 请求限流功能；
8. 支持路径重写。

SpringCloud Gateway与Zuul的区别

1. 在SpringCloud Finchley正式版之前，Spring Cloud推荐的网关是Netflix提供的Zuul。
2. Zuul 1.x，是一个基于阻塞I/O的API Gateway。
3. Zuul 1.x基于Servlet 2.5使用阻塞架构它不支持任何长连接(如WebSocket)Zuul的设计模式和Nginx较像，每次I/О操作都是从工作线程中选择一个执行，请求线程被阻塞到工作线程完成，但是差别是Nginx用C++实现，Zuul用Java实现，而JVM本身会有第-次加载较慢的情况，使得Zuul的性能相对较差。
4. Zuul 2.x理念更先进，想基于Netty非阻塞和支持长连接，但SpringCloud目前还没有整合。Zuul .x的性能较Zuul 1.x有较大提升。在性能方面，根据官方提供的基准测试,Spring Cloud Gateway的RPS(每秒请求数)是Zuul的1.6倍。
5. Spring Cloud Gateway建立在Spring Framework 5、Project Reactor和Spring Boot2之上，使用非阻塞API。
6. Spring Cloud Gateway还支持WebSocket，并且与Spring紧密集成拥有更好的开发体验

Gateway模型

WebFlux是什么？官方文档

传统的Web框架，比如说: Struts2，SpringMVC等都是基于Servlet APl与Servlet容器基础之上运行的。

但是在Servlet3.1之后有了异步非阻塞的支持。而WebFlux是一个典型非阻塞异步的框架，它的核心是基于Reactor的相关API实现的。相对于传统的web框架来说，它可以运行在诸如Netty，Undertow及支持Servlet3.1的容器上。非阻塞式+函数式编程(Spring 5必须让你使用Java 8)。

Spring WebFlux是Spring 5.0 引入的新的响应式框架，区别于Spring MVC，它不需要依赖Servlet APl，它是完全异步非阻塞的，并且基于Reactor来实现响应式流规范。



## Gateway工作流程

三大核心概念

- Route(路由) - 路由是构建网关的基本模块,它由ID,目标URI,一系列的断言和过滤器组成,如断言为true则匹配该路由；
- Predicate(断言) - 参考的是Java8的java.util.function.Predicate，开发人员可以匹配HTTP请求中的所有内容(例如请求头或请求参数),如果请求与断言相匹配则进行路由；
- Filter(过滤) - 指的是Spring框架中GatewayFilter的实例,使用过滤器,可以在请求被路由前或者之后对请求进行修改。

## Gateway9527搭建

1. 新建 cloud-gateway-gateway9527 模块

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
   
       <artifactId>cloud-gateway-gateway9527</artifactId>
   
       <dependencies>
           <!--gateway-->
           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-starter-gateway</artifactId>
           </dependency>
           <!--eureka-client-->
           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
           </dependency>
           <!-- 引入自己定义的api通用包，可以使用Payment支付Entity -->
           <dependency>
               <groupId>com.atguigu.springcloud</groupId>
               <artifactId>cloud-api-commons</artifactId>
               <version>1.0-SNAPSHOT</version>
           </dependency>
           <!--一般基础配置类-->
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
   
       <repositories>
           <repository>
               <id>ali-maven</id>
               <url>http://maven.aliyun.com/nexus/content/groups/public</url>
           </repository>
       </repositories>
   </project>
   ```

3. yml

   ```yml
   server:
     port: 9527
   
   spring:
     application:
       name: cloud-gateway
   
   eureka:
     instance:
       hostname: cloud-gateway-service
     client: #服务提供者provider注册进eureka服务列表内
       register-with-eureka: true
       fetch-registry: true
       service-url:
         defaultZone: http://eureka7001.com:7001/eureka
   ```

4. 主启动

   ```java
   package com.atguigu.springcloud;
   import org.springframework.boot.SpringApplication;
   import org.springframework.boot.autoconfigure.SpringBootApplication;
   import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
   
   
   @SpringBootApplication
   @EnableEurekaClient
   public class GateWayMain9527
   {
       public static void main(String[] args) {
           SpringApplication.run(GateWayMain9527.class, args);
       }
   }
   
   ```

5. 路由映射yml

   ```yml
   spring:
     application:
       name: cloud-gateway
     #############################新增网关配置###########################
     cloud:
       gateway:
         routes:
           - id: payment_routh #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
             uri: http://localhost:8001          #匹配后提供服务的路由地址
   #          uri: lb://cloud-payment-service #匹配后提供服务的路由地址
             predicates:
   #            - Header=X-Request-Id, \d+
   #            - Cookie=username,zzyy
   #            - After=2021-05-16T19:53:59.024+08:00[Asia/Shanghai]
               - Path=/payment/get/**         # 断言，路径相匹配的进行路由
   
           - id: payment_routh2 #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
             uri: http://localhost:8001          #匹配后提供服务的路由地址
   #          uri: lb://cloud-payment-service #匹配后提供服务的路由地址
             predicates:
               - Path=/payment/lb/**         # 断言，路径相匹配的进行路由
   ####################################################################
   
   ```

6. 测试

   - 启动7001

   - 启动8001-cloud-provider-payment8001

   - 启动9527网关

   - 访问说明

     - 添加网关前 - http://localhost:8001/payment/get/1

     - 添加网关后 - http://localhost:9527/payment/get/1

     - 两者访问成功，返回相同结果

       

### Gateway配置路由的两种方式Route

1. yml配置 看上一部分

2. **代码中注入RouteLocator的Bean**

   ```java
   package com.atguigu.springcloud.config;
   
   import org.springframework.cloud.gateway.route.RouteLocator;
   import org.springframework.cloud.gateway.route.builder.RouteLocatorBuilder;
   import org.springframework.context.annotation.Bean;
   import org.springframework.context.annotation.Configuration;
   
   @Configuration
   public class GateWayConfig {
       @Bean
       public RouteLocator customRouteLocator(RouteLocatorBuilder routeLocatorBuilder){
           RouteLocatorBuilder.Builder routes = routeLocatorBuilder.routes();
           routes.route("path_route_atguigu",
                   r -> r.path("/guonei").uri("http://news.baidu.com/guonei")).build();
   
           routes.route("path_route_atguigu1",
                   r -> r.path("/game").uri("http://news.baidu.com/game")).build();
           return routes.build();
       }
   }
   
   ```

3. 测试

   浏览器输入http://localhost:9527/guonei，返回http://news.baidu.com/guonei相同的页面。

### GateWay配置动态路由

> 默认情况下Gateway会根据注册中心注册的服务列表，以注册中心上微服务名为路径创建**动态路由进行转发，从而实现动态路由的功能**（不写死一个地址）。

1. 9527yml更改

   ```yml
   spring:
     application:
       name: cloud-gateway
     #############################新增网关配置###########################
     cloud:
       gateway:
         routes:
           - id: payment_routh #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
   #          uri: http://localhost:8001          #匹配后提供服务的路由地址
             uri: lb://cloud-payment-service #匹配后提供服务的路由地址
             predicates:
               - Path=/payment/get/**         # 断言，路径相匹配的进行路由
   
           - id: payment_routh2 #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
   #          uri: http://localhost:8001          #匹配后提供服务的路由地址
             uri: lb://cloud-payment-service #匹配后提供服务的路由地址
             predicates:
               - Path=/payment/lb/**         # 断言，路径相匹配的进行路由
   ####################################################################
   
   ```

2. 测试 成功

### GateWay常用的Predicate(断言)

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

  cloud:
    gateway:
      routes:
        - id: payment_routh #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
#          uri: http://localhost:8001          #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service #匹配后提供服务的路由地址
          
          
          
          predicates:
#            - Header=X-Request-Id, \d+
#            - Cookie=username,zzyy
#            - After=2021-05-16T19:53:59.024+08:00[Asia/Shanghai]
            - Path=/payment/get/**         # 断言，路径相匹配的进行路由

```

获取时间戳

```java
import java.time.ZonedDateTime;

public class Test {
    public static void main(String[] args) {
        ZonedDateTime zonedDateTime = ZonedDateTime.now();//默认时区
        System.out.println(zonedDateTime);
    }

}
```

header cookie 举一反三

### GateWay的Filter(过滤)

#### 自定义

GateWay9527项目添加MyLogGateWayFilter类：

```java
package com.atguigu.springcloud.filter;

import lombok.extern.slf4j.Slf4j;
import org.springframework.cloud.gateway.filter.GatewayFilterChain;
import org.springframework.cloud.gateway.filter.GlobalFilter;
import org.springframework.core.Ordered;
import org.springframework.http.HttpStatus;
import org.springframework.stereotype.Component;
import org.springframework.web.server.ServerWebExchange;
import reactor.core.publisher.Mono;

import java.util.Date;


@Component
@Slf4j
public class MyLogGateWayFilter implements GlobalFilter, Ordered {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        log.info("全局过滤器"+new Date());
        String uname = exchange.getRequest().getQueryParams().getFirst("uname");
        if (uname == null) {
            log.info("用户名为空，非法用户");
            exchange.getResponse().setStatusCode(HttpStatus.NOT_ACCEPTABLE);
            return exchange.getResponse().setComplete();
        }
        return chain.filter(exchange);
    }

    @Override
    public int getOrder() {
        return 0;
    }
}

```

测试

[localhost:9527/payment/get/1?uname=1](http://localhost:9527/payment/get/1?uname=1)正常

[localhost:9527/payment/get/1](http://localhost:9527/payment/get/1?uname=1)失败

