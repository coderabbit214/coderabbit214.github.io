# spring-cloud-stream 3.x 开发


# spring-cloud-stream 3.x 开发

## 基本使用

### 基本配置

1. 依赖

   ```gradel
   implementation 'org.springframework.cloud:spring-cloud-stream:3.2.3'
   ```

2. 配置

   ```yaml
   spring:
     rabbitmq:
       host: 127.0.0.1
       port: 5672
       username: username
       password: password
   ```

### 生产者

#### 配置

注意：analysePdf-out-0，analysePdf发送消息，接收消息都需要使用

```yaml
spring:
  cloud:
    stream:
      rabbit:
        bindings:
          analysePdf-out-0:
            producer:
              destination: analysePdf
              content-type: application/json
      bindings:
        analysePdf-out-0:
          destination: analysePdf
```

#### 代码

```java
    @Autowired
    private StreamBridge streamBridge;

    public void sendMethod() {
        //与配置文件中相同
        streamBridge.send("analysePdf-out-0", "pdf"+i);
    }
```

### 消费者

#### 配置

```yaml
spring:
  cloud:
    stream:
      rabbit:
        bindings:
          analysePdf-in-0:
            consumer:
              destination: analysePdf
              content-type: application/json
      bindings:
        analysePdf-in-0:
          destination: analysePdf
```

#### 代码

注意：方法名需要和配置中心中相同

```java
@Configuration
public class Test {
    @Bean("analysePdf")
  	//方法名与配置中相同
    public Consumer<String> analysePdf() {
        return str -> {
            System.out.println("analysePdf: " + str);
        };
    }
}
```

## 消息分组(多实例下避免重复消费)

消费者配置：group: in，配置后同组消费者下只能消费一次

```yaml
spring:
  cloud:
    stream:
      rabbit:
        bindings:
          analysePdf-in-0:
            consumer:
              destination: analysePdf
              content-type: application/json
      bindings:
        analysePdf-in-0:
        	group: in
          destination: analysePdf
```

## ack配置

消费者配置：acknowledge-mode: auto

```yaml
spring:
  cloud:
    stream:
      rabbit:
        bindings:
          analysePdf-in-0:
            consumer:
              destination: analysePdf
              content-type: application/json
              #ack模式
              acknowledge-mode: auto
              #重试次数
              max-attempts: 5
      bindings:
        analysePdf-in-0:
        	group: in
          destination: analysePdf
```

## 多消息使用

注意配置,其他配置重复一遍

```yaml
spring:
	cloud:
  	function:
    	definition: 
```

完整配置

```yaml
spring:
  cloud:
    function:
      definition: analysePdf;jsh
    stream:
      rabbit:
        bindings:
          analysePdf-in-0:
            consumer:
              destination: analysePdf
              content-type: application/json
              #ack模式
              acknowledge-mode: auto
              #重试次数
              max-attempts: 5
          analysePdf-out-0:
            producer:
              destination: analysePdf
              content-type: application/json
          jsh-in-0:
            consumer:
              destination: jsh
              content-type: application/json
          jsh-out-0:
            producer:
              destination: jsh
              content-type: application/json
      bindings:
        analysePdf-in-0:
          group: in
          destination: analysePdf
        analysePdf-out-0:
          destination: analysePdf
        jsh-in-0:
          group: in
          destination: jsh
        jsh-out-0:
          destination: jsh
```

> [Spring Cloud Stream 官网](https://docs.spring.io/spring-cloud-stream/docs/current/reference/html/spring-cloud-stream-binder-rabbit.html#_rabbitmq_consumer_properties)
>
> [源码地址](https://github.com/coderabbit214/parent-demo/tree/main/sping-stream-demo)


