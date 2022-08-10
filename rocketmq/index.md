# RocketMQ


# RocketMQ

## 应用场景

- 异步解藕
- 削峰填谷
- 消息分发

## 环境搭建

1. 上传rocketmq-all-4.4.0-bin-release.zip 到家目录

2. 使用解压命令进行解压

   ```shell
   unzip /usr/local/rocketmq-all-4.4.0-bin-release.zip
   ```

3. 软件重命名

   ```shell
   mv /usr/local/rocketmq-all-4.4.0-bin-release/ /usr/local/rocketmq-4.4/
   ```

4. 修改启动参数配置

   > JAVA_OPT="${JAVA_OPT} -server **-Xms1g -Xmx1g -Xmn1g**"

   两个文件

   ```shell
   vi /usr/local/rocketmq-4.4/bin/runbroker.sh
   
   vi /usr/local/rocketmq-4.4/bin/runserver.sh
   ```

5. 启动名字服务和代理服务

   ```shell
   nohup sh /usr/local/rocketmq-4.4/bin/mqnamesrv &
   
   # -n localhost:9876 指定名称服务的地址, 类似于zk的地址
   
   nohup sh /usr/local/rocketmq-4.4/bin/mqbroker -n localhost:9876 -c /usr/local/rocketmq-4.4/conf/broker.conf &  
   ```

6. 检验是否启动正常

   使用java的内置命令: **jps 可以看到BrokerStartup和NamesrvStartup进程**

   使用Linux命令**: netstat-ntlp 可以看到9876的端口和10911的端口**

   使用ps-ef |grep java

   查看启动日志:

     tail -100f ~/logs/rocketmqlogs/namesrv.log

     tail -100f ~/logs/rocketmqlogs/broker.log

7. 关闭RocketMQ

   ```shell
   # 1.关闭NameServer
   
   sh /usr/local/rocketmq-4.4/bin/mqshutdown namesrv
   
   # 2.关闭Broker
   
   sh /usr/local/rocketmq-4.4/bin/mqshutdown broker
   ```

### 编写sh脚本文件

- 启动(startRocketMQ.sh)

  ```shell
  # !/bin/bash
  
  echo '------------------rocketmq-nameServer-starter-------------------------' 
  nohup sh /usr/local/rocketmq-4.4/bin/mqnamesrv &
  echo '------------------rocketmq-nameServer-started-------------------------'
  
  echo '------------------rocketmq-brokerServer-starter-----------------------' 
  nohup sh /usr/local/rocketmq-4.4/bin/mqbroker -n localhost:9876 -c /usr/local/rocketmq-4.4/conf/broker.conf &
  echo '------------------rocketmq-brokerServer-started-----------------------' 
  ```

- 关闭(stutdownRocketMQ.sh)

  ```shell
  # !/bin/bash
  
  echo '------------------rocketmq-nameServer-shutdown-------------------------' 
  sh /usr/local/rocketmq-4.4/bin/mqshutdown namesrv
  echo '------------------rocketmq-nameServer-shutdowned-------------------------'
  
  echo '------------------rocketmq-brokerServer-shutdown-----------------------' 
  sh /usr/local/rocketmq-4.4/bin/mqshutdown broker
  echo '------------------rocketmq-brokerServer-shutdowned-----------------------' 
  ```

## 监控平台

使用jar

```shell
nohup  java -jar  rocketmq-console-ng-1.0.1.jar  &
```

 

## SpringBoot集成

### 依赖

```xml
        <dependency>
            <groupId>org.apache.rocketmq</groupId>
            <artifactId>rocketmq-spring-boot-starter</artifactId>
            <version>2.0.3</version>
        </dependency>
```

### 配置

- 生产者

  ```properties
  rocketmq.name-server=127.0.0.1:9876
  rocketmq.producer.group=my-group
  ```

- 消费者

  ```properties
  rocketmq.name-server=127.0.0.1:9876
  ```

### 编码

- 生产者

  ```java
  @RestController
  public class HelloController {
      @Autowired
      private RocketMQTemplate rocketMQTemplate;
      @RequestMapping("01-hello")
      public String sendMsg(String message,String age) throws Exception{
        //发送消息
     		  SendResult sendResult = rocketMQTemplate.syncSend("01-boot:", message);
        
          System.out.println(sendResult.getMsgId());
          System.out.println(sendResult.getSendStatus());
          return "success";
      }
  }
  ```

- 消费者

  ```java
  @Component
  @RocketMQMessageListener(
          topic = "01-boot",
          consumerGroup = "wolfcode-consumer"
  )
  public class HelloConsumer implements RocketMQListener<MessageExt> {
      @Override
      public void onMessage(MessageExt messageExt) {
           System.out.println("消费消息"+messageExt);
      }
  }
  ```

#### 发送消息方式(生产者)

##### 发送类型

- 同步消息

  ```java
  SendResult sendResult = rocketMQTemplate.syncSend("020-boot", msg);
  System.out.println(sendResult.getMsgId());
  System.out.println(sendResult.getSendStatus());
  ```

- 异步消息

  ```java
  rocketMQTemplate.asyncSend("020-boot", msg, new SendCallback() {
              @Override
              public void onSuccess(SendResult sendResult) {
                  System.out.println(sendResult.getMsgId());
                  System.out.println(sendResult.getSendStatus());
              }
  
              @Override
              public void onException(Throwable throwable) {
                  System.out.println(throwable);
              }
          });
  ```

- 一次性消息

  ```java
  rocketMQTemplate.sendOneWay("020-boot", msg);
  ```

##### 发送时间

> 默认立即发送

- 延时发送

  ```java
  // 参数1:主题 2:消息 3:rocket发送最大允许时间 4:延时级别(18级)
  SendResult sendResult = rocketMQTemplate.syncSend("020-boot", MessageBuilder.withPayload(msg).build(),100000,3);
  ```

#### 消费模式(消费者)

> 以组为单位 默认为集群模式

- 集群模式(每组只有一个可以收到)

  ```java
  @Component
  @RocketMQMessageListener(
          topic = "020-boot",
          messageModel = MessageModel.CLUSTERING,
          consumerGroup = "wolfcode-consumer"
  )
  public class MqListenner implements RocketMQListener<String> {
  
      @Override
      public void onMessage(String s) {
          System.out.println("今天上映:"+s);
      }
  }
  ```

- 广播模式(每组的所有消费者都可以收到)

  ```java
  @Component
  @RocketMQMessageListener(
          topic = "020-boot",
          messageModel = MessageModel.BROADCASTING,
          consumerGroup = "wolfcode-consumer"
  )
  public class MqListenner implements RocketMQListener<String> {
  
      @Override
      public void onMessage(String s) {
          System.out.println("今天上映:"+s);
      }
  }
  ```

#### 消息过滤

##### Tag标签模式

> 在发送的消息Topic:Tag 中间使用冒号隔开

- 生产者

  ```java
      @RequestMapping("/sendTagMsg")
      public String sendTagMsg(String msg) {
          rocketMQTemplate.convertAndSend("020-boot:TagB",msg);
          return "success";
      }
  ```

- 消费者

  ```java
  @Component
  @RocketMQMessageListener(
          topic = "020-boot",
          selectorType = SelectorType.TAG,
    //接收TagB或TagA
          secretKey = "TagB || TagA",
          consumerGroup = "wolfcode-consumer"
  )
  public class MqListenner implements RocketMQListener<String> {
      @Override
      public void onMessage(String s) {
          System.out.println("今天上映:"+s);
      }
  }
  ```

##### SQL92过滤

> 注意: 在使用SQL过滤的时候, 需要配置参数enablePropertyFilter=true

- 生产者

  ```java
      //Sql92过滤
      @RequestMapping("/sendSQLMsg")
      public String sendSQLMsg(int age,String msg) {
          Map<String,Object> map=new HashMap<>();
          //用户自定义属性
          map.put("age", age);
          map.put("name", "hesj");
          //也可以设置系统属性
          map.put(MessageConst.PROPERTY_KEYS,age);
          template.convertAndSend("02-RocketMQ-Top7",msg,map);
          return "success";
      }
  ```

- 消费者

  ```java
  @Component
  @RocketMQMessageListener(
          topic = "02-RocketMQ-Top7",
          messageModel = MessageModel.CLUSTERING,
          selectorType = SelectorType.SQL92,
          selectorExpression = "age > 16",
          consumerGroup= "wolfcode-consumer7"
  )
  public class MqListiner7 implements RocketMQListener<String> {
      @Override
      public void onMessage(String msg) {
          System.out.println("消费消息SQl92"+msg);
      }
  }
  ```

  

​	

