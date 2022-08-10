# Springboot自动装配


# Springboot自动装配

## Springboot自动装配

### 怎么给spring容器添加组件

原来的SSM使用xml文件

```xml
    <bean id="druidDataSource" class="com.alibaba.druid.pool.DruidDataSource" init-method="init" destroy-method="close">
        <property name="driverClassName" value="${jdbc.driverClassName}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>
```

Springboot 方式

- @Configuration   等同于配置文件 
  - 参数：proxyBeanMethods: true单实例(别人依赖时使用) false多实例(别人不依赖时使用)
- @Bean                   等同于原来的bean 把返回值放在spring容器中 默认单实例
- @ConditionalOnBean(条件) 可以加在方法上和类上，满足条件后才会加载

![截屏2021-10-04 18.49.23](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/%E6%88%AA%E5%B1%8F2021-10-04%2018.49.23.png)

- @Import({class, class...}) 可以自动创建出这几个类型的组件，默认是全类名(包名➕类名)
- @ImportResource("资源路径") 解析xml文件到spring容器中
- @ConfigurationProperties("j j j") 获取配置的变量  需要和一下两个注解结合使用
  - @Configuration              和@ConfigurationProperties在一起使用
  - @EnableConfigurationProperties(XXXProperties.class)   引用时使用

### 自动配置原理

- SpringBootApplication

  - @SpringBootConfiguration表示是一个配置类

  - @ComponentScan 包扫描

  - @EnableAutoConfiguration 重要 自动装配

    - @AutoConfigurationPackage

      - @Import(AutoConfigurationPackages.Registrar.class)

        利用Registrar给容器中导入一系列组件

        将指定的（MainApplication）包下的所有组件倒入容器中

    - @Import(AutoConfigurationImportSelector.class)

      - AutoConfigurationImportSelector.class

        ![截屏2021-10-04 19.21.12](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/%E6%88%AA%E5%B1%8F2021-10-04%2019.21.12.png)

        主要是这句获取容器中所有组件 

        ```java
        List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);
        ```

        ![截屏2021-10-04 19.23.31](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/%E6%88%AA%E5%B1%8F2021-10-04%2019.23.31.png)

        一路找下去 最后到

        ![截屏2021-10-04 19.24.25](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/%E6%88%AA%E5%B1%8F2021-10-04%2019.24.25.png)

        这个方法扫描spring-boot-autoconfigure自动配置类

        文件里边写死了springboot一启动就要加载的组件默认全部加载，最终会按需配置（@ConditionalXXX）



## 自定义starter

![截屏2021-10-06 12.30.38](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/%E6%88%AA%E5%B1%8F2021-10-06%2012.30.38.png)

- starter:是一种开发场景，可以引入很多autoconfigure
- autoconfigure：自动装配

### 代码实现

![截屏2021-10-06 12.34.48](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/%E6%88%AA%E5%B1%8F2021-10-06%2012.34.48.png)

1. starter

   > 只需要引入autoconfigure即可

   - pom

     ```xml
     <?xml version="1.0" encoding="UTF-8"?>
     <project xmlns="http://maven.apache.org/POM/4.0.0"
              xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
              xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
         <modelVersion>4.0.0</modelVersion>
         <groupId>com.jsh</groupId>
         <artifactId>jsh-spring-boot-starter</artifactId>
         <version>1.0.0</version>
     
         <dependencies>
             <dependency>
                 <groupId>com.jsh</groupId>
                 <artifactId>jsh-spring-boot-starter-autoconfigure</artifactId>
                 <version>0.0.1-SNAPSHOT</version>
             </dependency>
         </dependencies>
     
     </project>
     ```

2. autoconfigure

   - 编写XXXProperties引入配置的变量

     ```java
     @ConfigurationProperties("jsh")
     public class JshProperties {
         private String prefix;
         private String suffix;
     
         public JshProperties() {
     
         }
     
         public JshProperties(String prefix, String suffix) {
             this.prefix = prefix;
             this.suffix = suffix;
         }
     
         public String getPrefix() {
             return prefix;
         }
     
         public void setPrefix(String prefix) {
             this.prefix = prefix;
         }
     
         public String getSuffix() {
             return suffix;
         }
     
         public void setSuffix(String suffix) {
             this.suffix = suffix;
         }
     }
     ```

   - 编写具体的业务或者配置

     - service

       ```java
       /**
        * 默认不要放在容器中
        */
       public class HelloService {
           @Autowired
           JshProperties jshProperties;
       
           public String sayHello(String userName){
               return jshProperties.getPrefix()+":"+userName+":"+jshProperties.getSuffix();
           }
       }
       
       ```

   - XXXAutoConfiguration

     ```java
     @Configuration
     @EnableConfigurationProperties(JshProperties.class) //默认JshProperties放在容器中
     public class JshServiceAutoConfiguration {
         @Bean
         @ConditionalOnMissingBean(HelloService.class)
         public HelloService helloService(){
             return new HelloService();
         }
     }
     
     ```

   - META-INF

     - spring.factories

       ```
       # Auto Configure
       org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
       com.jsh.auto.JshServiceAutoConfiguration
       ```

### 使用

1. 引入依赖
2. 配置
3. 使用










