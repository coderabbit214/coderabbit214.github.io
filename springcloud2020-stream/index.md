# springcloud2020 Stream


# springcloud2020 Stream

> 驱动RabbitMQ、 Kafka消息中间件的统一解决方法

## 简介



![1ca02dd31581d92a7a610bcd137f6848](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/1ca02dd31581d92a7a610bcd137f6848.png)

- Binder - 很方便的连接中间件，屏蔽差异。

- Channel - 通道，是队列Queue的一种抽象，在消息通讯系统中就是实现存储和转发的媒介，通过Channel对队列进行配置。

- Source和Sink - 简单的可理解为参照对象是Spring Cloud Stream自身，从Stream发布消息就是输出，接受消息就是输入。



**编码API和常用注解**

|    **组成**     |                           **说明**                           |
| :-------------: | :----------------------------------------------------------: |
|   Middleware    |              中间件，目前只支持RabbitMQ和Kafka               |
|     Binder      | Binder是应用与消息中间件之间的封装，目前实行了Kafka和RabbitMQ的Binder，通过Binder可以很方便的连接中间件，可以动态的改变消息类型(对应于Kafka的topic,RabbitMQ的exchange)，这些都可以通过配置文件来实现 |
|     @Input      |   注解标识输入通道，通过该输乎通道接收到的消息进入应用程序   |
|     @Output     |     注解标识输出通道，发布的消息将通过该通道离开应用程序     |
| @StreamListener |             监听队列，用于消费者的队列的消息接收             |
| @EnableBinding  |              指信道channel和exchange绑定在一起               |



准备：

RabbitMQ环境

工程中新建三个子模块：

- cloud-stream-rabbitmq-provider8801，作为生产者进行发消息模块
- cloud-stream-rabbitmq-consumer8802，作为消息接收模块
- cloud-stream-rabbitmq-consumer8803，作为消息接收模块



## Stream消息驱动之生产者

1. 新建cloud-stream-rabbitmq-provider8801

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
   
       <artifactId>cloud-stream-rabbitmq-provider8801</artifactId>
   
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
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
           </dependency>
           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
           </dependency>
           <!--基础配置-->
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

3. YML

   ```yml
   server:
     port: 8801
   
   spring:
     application:
       name: cloud-stream-provider
     cloud:
       stream:
         binders: # 在此处配置要绑定的rabbitmq的服务信息；
           defaultRabbit: # 表示定义的名称，用于于binding整合
             type: rabbit # 消息组件类型
             environment: # 设置rabbitmq的相关的环境配置
               spring:
                 rabbitmq:
                   host: localhost
                   port: 5672
                   username: guest
                   password: guest
         bindings: # 服务的整合处理
           output: # 这个名字是一个通道的名称
             destination: studyExchange # 表示要使用的Exchange名称定义
             content-type: application/json # 设置消息类型，本次为json，文本则设置“text/plain”
             binder: defaultRabbit # 设置要绑定的消息服务的具体设置
   
   eureka:
     client: # 客户端进行Eureka注册的配置
       service-url:
         defaultZone: http://localhost:7001/eureka
     instance:
       lease-renewal-interval-in-seconds: 2 # 设置心跳的时间间隔（默认是30秒）
       lease-expiration-duration-in-seconds: 5 # 如果现在超过了5秒的间隔（默认是90秒）
       instance-id: send-8801.com  # 在信息列表时显示主机名称
       prefer-ip-address: true     # 访问的路径变为IP地址
   ```

4. 主启动

   ```java
   package com.atguigu.springcloud;
   
   import org.springframework.boot.SpringApplication;
   import org.springframework.boot.autoconfigure.SpringBootApplication;
   
   @SpringBootApplication
   public class StreamMQMain8801 {
       public static void main(String[] args) {
           SpringApplication.run(StreamMQMain8801.class,args);
       }
   }
   
   ```

5. 业务类

   - 发送消息接口

     ```java
     package com.atguigu.springcloud.service;
     
     public interface IMessageProvider {
         public String send();
     }
     
     ```

   - 发送消息接口实现类

     > @EnableBinding(Source.class)//定义消息的推送管道 
     >
     > 定义输入 输出
     >
     >  
     >
     > @Resource    
     >
     > private MessageChannel output; *// 消息发送管道*
     >
     > output.send(MessageBuilder.withPayload(serial).build());
     >
     > 发送

     ```java
     package com.atguigu.springcloud.service.impl;
     
     import com.atguigu.springcloud.service.IMessageProvider;
     import org.springframework.cloud.stream.annotation.EnableBinding;
     import org.springframework.cloud.stream.messaging.Source;
     import org.springframework.integration.support.MessageBuilder;
     import org.springframework.messaging.MessageChannel;
     
     import javax.annotation.Resource;
     import java.util.UUID;
     
     @EnableBinding(Source.class)//定义消息的推送管道
     public class MessageProviderImpl implements IMessageProvider {
         @Resource
         private MessageChannel output;//消息发送的管道
     
         @Override
         public String send() {
             String serial = UUID.randomUUID().toString();
             output.send(MessageBuilder.withPayload(serial).build());
             System.out.println("serial:"+serial);
             return serial;
         }
     }
     
     ```

   - Controller

     ```java
     package com.atguigu.springcloud.controller;
     
     import com.atguigu.springcloud.service.IMessageProvider;
     import org.springframework.web.bind.annotation.GetMapping;
     import org.springframework.web.bind.annotation.RestController;
     
     import javax.annotation.Resource;
     
     @RestController
     public class SendMessageController
     {
         @Resource
         private IMessageProvider messageProvider;
     
         @GetMapping(value = "/sendMessage")
         public String sendMessage() {
             return messageProvider.send();
         }
     
     }
     ```

6. 测试

   - 启动 7001eureka
   - 启动 RabpitMq
     - rabbitmq-plugins enable rabbitmq_management
     - http://localhost:15672/
   - 启动 8801
   - 访问 - http://localhost:8801/sendMessage
     - 后台将打印serial: UUID字符串



## Stream消息驱动之消费者

1. 新建 cloud-stream-rabbitmq-consumer8802

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
   
       <artifactId>cloud-stream-rabbitmq-consumer8802</artifactId>
   
       <dependencies>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-web</artifactId>
           </dependency>
           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
           </dependency>
           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
           </dependency>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-actuator</artifactId>
           </dependency>
           <!--基础配置-->
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
     port: 8802
   
   spring:
     application:
       name: cloud-stream-consumer
     cloud:
       stream:
         binders: # 在此处配置要绑定的rabbitmq的服务信息；
           defaultRabbit: # 表示定义的名称，用于于binding整合
             type: rabbit # 消息组件类型
             environment: # 设置rabbitmq的相关的环境配置
               spring:
                 rabbitmq:
                   host: localhost
                   port: 5672
                   username: guest
                   password: guest
         bindings: # 服务的整合处理
           input: # 这个名字是一个通道的名称
             destination: studyExchange # 表示要使用的Exchange名称定义
             content-type: application/json # 设置消息类型，本次为对象json，如果是文本则设置“text/plain”
             binder: defaultRabbit # 设置要绑定的消息服务的具体设置
             group: A_GROUP
   
   eureka:
     client: # 客户端进行Eureka注册的配置
       service-url:
         defaultZone: http://localhost:7001/eureka
     instance:
       lease-renewal-interval-in-seconds: 2 # 设置心跳的时间间隔（默认是30秒）
       lease-expiration-duration-in-seconds: 5 # 如果现在超过了5秒的间隔（默认是90秒）
       instance-id: receive-8802.com  # 在信息列表时显示主机名称
       prefer-ip-address: true     # 访问的路径变为IP地址
   ```

4. 主启动

   ```java
   package com.atguigu.springcloud;
   
   import org.springframework.boot.SpringApplication;
   import org.springframework.boot.autoconfigure.SpringBootApplication;
   
   @SpringBootApplication
   public class StreamMQMain8802 {
       public static void main(String[] args) {
           SpringApplication.run(StreamMQMain8802.class,args);
       }
   }
   
   ```

5. 业务类

   > @EnableBinding(Sink.class)
   >
   > 定义输入 输出

   ```java
   package com.atguigu.springcloud.controller;
   
   import org.springframework.beans.factory.annotation.Value;
   import org.springframework.cloud.stream.annotation.EnableBinding;
   import org.springframework.cloud.stream.annotation.StreamListener;
   import org.springframework.cloud.stream.messaging.Sink;
   import org.springframework.messaging.Message;
   import org.springframework.stereotype.Controller;
   
   
   @Controller
   @EnableBinding(Sink.class)
   public class ReceiveMessageListenerController
   {
       @Value("${server.port}")
       private String serverPort;
   
   
       @StreamListener(Sink.INPUT)
       public void input(Message<String> message)
       {
           System.out.println("消费者1号,----->接受到的消息: "+message.getPayload()+"\t  port: "+serverPort);
       }
   }
   
   ```

6. 测试

   - 启动EurekaMain7001
   - 启动StreamMQMain8801
   - 启动StreamMQMain8802
   - 8801发送8802接收消息

## Stream之消息重复消费

依照8802，克隆出来一份运行8803 - cloud-stream-rabbitmq-consumer8803。

启动

- RabbitMQ

- 服务注册 - 8801

- 消息生产 - 8801

- 消息消费 - 8802

- 消息消费 - 8802

运行后有两个问题

1. 有重复消费问题
2. 消息持久化问题

**消费**

- http://localhost:8801/sendMessage
- 目前是8802/8803同时都收到了，存在重复消费问题
- 如何解决：分组和持久化属性group（重要）



### Stream之group解决消息重复消费

> 增加 yml配置 使消费者分组 每组只有一个接收

操作

```yml
spring:
  application:
    name: cloud-stream-provider
  cloud:
      stream:
        binders: # 在此处配置要绑定的rabbitmq的服务信息；
          defaultRabbit: # 表示定义的名称，用于于binding整合
            type: rabbit # 消息组件类型
            environment: # 设置rabbitmq的相关的环境配置
              spring:
                rabbitmq:
                  host: localhost
                  port: 5672
                  username: guest
                  password: guest
        bindings: # 服务的整合处理
          output: # 这个名字是一个通道的名称
            destination: studyExchange # 表示要使用的Exchange名称定义
            content-type: application/json # 设置消息类型，本次为json，文本则设置“text/plain”
            binder: defaultRabbit # 设置要绑定的消息服务的具体设置
            group: A_Group #<----------------------------------------关键
```



### 消息持久化解决

删除8002分组

测试：

- 先启动8802，**无分组属性配置**，后台没有打出来消息。

- 再启动8803，**有分组属性配置**，后台打出来了MQ上的消息。(消息持久化体现)

