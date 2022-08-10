# Springboot自定义starter


# Springboot自定义starter

## 启动器模块

> 普通maven项目

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.jsh.starter</groupId>
    <artifactId>jsh-spring-boot-starter</artifactId>
    <version>1.0-SNAPSHOT</version>
<!--    启动器-->
    <dependencies>
<!--        引入自动配置模块-->
        <dependency>
            <groupId>com.jsh.statrter</groupId>
            <artifactId>jsh-spring-boot-starter-autoconfigurre</artifactId>
            <version>0.0.1-SNAPSHOT</version>
        </dependency>
    </dependencies>

</project>
```

## 自动配置模块

> pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.3.0.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.jsh.statrter</groupId>
	<artifactId>jsh-spring-boot-starter-autoconfigurre</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>jsh-spring-boot-starter-autoconfigurre</name>
	<description>Demo project for Spring Boot</description>

	<properties>
		<java.version>1.8</java.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter</artifactId>
		</dependency>
	</dependencies>


</project>

```

1. properties

   > 用于配置

   ```java
   package com.jsh.statrter;
   
   import org.springframework.boot.context.properties.ConfigurationProperties;
   
   @ConfigurationProperties(prefix = "jsh.hello")
   public class HelloProperties {
       private String prefix;
       private String suffix;
   
       public String getSuffix() {
           return suffix;
       }
   
       public void setSuffix(String suffix) {
           this.suffix = suffix;
       }
   
       public String getPrefix() {
           return prefix;
       }
   
       public void setPrefix(String prefix) {
           this.prefix = prefix;
       }
   }
   
   ```

2. service

   ```java
   package com.jsh.statrter;
   
   public class HelloService {
       HelloProperties helloProperties;
       public HelloProperties getHelloProperties() {
           return helloProperties;
       }
   
       public void setHelloProperties(HelloProperties helloProperties) {
           this.helloProperties = helloProperties;
       }
       public String sayHelloJsh(String name){
           return helloProperties.getPrefix()+"-"+name+"-"+helloProperties.getSuffix();
       }
   }
   
   ```

3. AutoConfiguration

   ```java
   package com.jsh.statrter;
   
   @Configuration//声明是一个配置类
   @ConditionalOnWebApplication//web应用才生效
   @EnableConfigurationProperties(HelloProperties.class)//绑定配置类
   public class HelloServiceAutoConfiguration {
       @Autowired
       HelloProperties helloProperties;
       @Bean
       public HelloService helloService(){
           HelloService service = new HelloService();
           service.setHelloProperties(helloProperties);
           return service;
       }
   }
   
   ```

4. resources/META-INF/spring.factories

   ```
   org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
   com.jsh.statrter.HelloServiceAutoConfiguration
   ```

   


