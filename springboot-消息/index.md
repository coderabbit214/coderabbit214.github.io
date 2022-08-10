# Springboot-消息


# Springboot-消息

## 一.安装rabbitmq

1. > //拉取镜像
   > docker pull rabbitmq:[版本]
   > //运行
   > docker run -d -p 5672:5672 -p 15672:15672 --name myrabbitmq [镜像id]
   
2. 本机访问

3. 如果进不去

   > docker exec -it myrabbitmq /bin/bash
   > rabbitmq-plugins enable rabbitmq_management

## 二.客户端测试

1. 本机进入192.168.0.103:15672 账号密码guest

2. 创建三个类型的交换器

   > 类型：
   >
   > 1. direct  完全匹配
   > 2. fanout 广播模式 会把消息放在每一个消息队列上
   > 3. topic 匹配模式
   >    - ‘#’匹配一个或者多个单词
   >    - ‘*’ 匹配一个单词

   ![tUOqSK.png](https://s1.ax1x.com/2020/06/03/tUOqSK.png)

3. 创建四个消息队列

   ![tUXpFI.png](https://s1.ax1x.com/2020/06/03/tUXpFI.png)

4. 将消息队列绑定在交换器上

![tUXLhq.png](https://s1.ax1x.com/2020/06/03/tUXLhq.png)

> Routing Key :是路由件 根据这个接收消息
>
> topic的路由件名字 需要使用‘*’‘#’进行匹配
>
> ​	例如：#.emps

![tUjE36.png](https://s1.ax1x.com/2020/06/03/tUjE36.png)

5. 发送消息进行测试

   > 通过哪个消息交换器进行发送 就遵守哪个的规则发送的消息队列中

   ![tUjcKU.png](https://s1.ax1x.com/2020/06/03/tUjcKU.png)

## 三.springboot整合

### 1.简单测试

> 自动配置
>  *  1、RabbitAutoConfiguration
>  *  2、有自动配置了连接工厂ConnectionFactory；
>  *  3、RabbitProperties 封装了 RabbitMQ的配置
>  *  4、 RabbitTemplate ：给RabbitMQ发送和接受消息；
>  *  5、 AmqpAdmin ： RabbitMQ系统管理功能组件;
>  *  	AmqpAdmin：创建和删除 Queue，Exchange，Binding
>  *  6、@EnableRabbit +  @RabbitListener 监听消息队列的内容

1. mvn

   ```xml
    <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-amqp</artifactId>
           </dependency>
   ```

2. 使用

   ```java
   package com.jsh.amqp;
   
   import com.jsh.amqp.bean.Book;
   import org.junit.jupiter.api.Test;
   import org.springframework.amqp.rabbit.core.RabbitTemplate;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.boot.test.context.SpringBootTest;
   
   import java.util.Arrays;
   import java.util.HashMap;
   import java.util.Map;
   
   @SpringBootTest
   class SpringbootAmqpApplicationTests {
       @Autowired
       RabbitTemplate rabbitTemplate;
   
       /**
        * 1.单播 ：点对点
        */
       @Test
       void contextLoads() {
           //只需要传入要发送的对象，自动序列化发送给rabbitmq
           //rabbitTemplate.convertAndSend(exchage,routeKey,object);
           Map<String,Object> map = new HashMap<>();
           map.put("msg","第一个消息");
           map.put("data", Arrays.asList("helloworld",123,true));
           //对象被默认序列化后发送出去
           //rabbitTemplate.convertAndSend("exchange.direct","jsh.news",map);
           rabbitTemplate.convertAndSend("exchange.direct","jsh.news",new Book("西游记","吴承恩"));
       }
       //接收数据，如何将数据转为JOSN发送出去
       @Test
       public void receive(){
           Object o = rabbitTemplate.receiveAndConvert("jsh.news");
           System.out.println(o.getClass());
           System.out.println(o);
       }
   
       /**
        * 广播
        */
       @Test
       public void sendMsg(){
           rabbitTemplate.convertAndSend("exchange.fanout","",new Book("西游记","吴承恩"));
       }
   }
   
   ```

3. 配置序列化变为JOSN格式

   ```java
   package com.jsh.amqp.config;
   
   import org.springframework.amqp.support.converter.Jackson2JsonMessageConverter;
   import org.springframework.amqp.support.converter.MessageConverter;
   import org.springframework.context.annotation.Bean;
   import org.springframework.context.annotation.Configuration;
   
   @Configuration
   public class MyAMQPConfig {
       @Bean
       public MessageConverter messageConverter(){
           return new Jackson2JsonMessageConverter();
       }
   }
   
   ```

### 2.监听使用

1. 主程序开启@EnableRabbit注解

   ```java
   package com.jsh.amqp;
   
   import org.springframework.amqp.rabbit.annotation.EnableRabbit;
   import org.springframework.boot.SpringApplication;
   import org.springframework.boot.autoconfigure.SpringBootApplication;
   @EnableRabbit //开启基于注解的RabbitMQ模式
   @SpringBootApplication
   public class SpringbootAmqpApplication {
   
       public static void main(String[] args) {
           SpringApplication.run(SpringbootAmqpApplication.class, args);
       }
   
   }
   
   ```

2. 方法上加入@RabbitListener注解 自动监听消息

   > 两种方法接收参数 
   >
   > 一种只接受消息  Object o
   >
   > 第二种可以接收消息的其他信息 Message message

   ```java
   package com.jsh.amqp.service;
   
   import com.jsh.amqp.bean.Book;
   import org.springframework.amqp.core.Message;
   import org.springframework.amqp.rabbit.annotation.RabbitListener;
   import org.springframework.stereotype.Service;
   
   @Service
   public class BookService {
       @RabbitListener(queues = "jsh.news")
       public void receive(Book book){
           System.out.println("收到:"+book);
       }
       @RabbitListener(queues = "jsh")
       public void receive2(Message message){
           System.out.println(message.getBody());
           System.out.println(message.getMessageProperties());
       }
   }
   
   ```

### 3.AmqpAdmin在程序中管理RabbitMQ

```java
@SpringBootTest
class SpringbootAmqpApplicationTests {
    @Autowired
    AmqpAdmin amqpAdmin;
    @Test
    public void createExchange(){
        //创建交换器
        //amqpAdmin.declareExchange(new DirectExchange("amqpadmin.exchange"));
        //System.out.println("创建完成");
        //创建消息队列
        //amqpAdmin.declareQueue(new Queue("amqpadmin.queue",true));
        //创建绑定规则
        amqpAdmin.declareBinding(new Binding("amqpadmin.queue", Binding.DestinationType.QUEUE,"amqpadmin.exchange","amqp.haha",null));
    }
}

```


