# springcloud2020  cloud-provider-payment8001模块


## springcloud2020  cloud-provider-payment8001模块

## 1.创建

1. 建model

   name：cloud-provider-payment8001

2. 改pom

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
   
       <artifactId>cloud-provider-payment8001</artifactId>
   
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
               <groupId>org.mybatis.spring.boot</groupId>
               <artifactId>mybatis-spring-boot-starter</artifactId>
           </dependency>
           <dependency>
               <groupId>com.alibaba</groupId>
               <artifactId>druid-spring-boot-starter</artifactId>
           </dependency>
           <dependency>
               <groupId>mysql</groupId>
               <artifactId>mysql-connector-java</artifactId>
           </dependency>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-jdbc</artifactId>
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
           </dependency>
       </dependencies>
   
   </project>
   ```

3. 写yml

   ```yml
   server:
     port: 8001
   
   spring:
     application:
       name: cloud-payment-service #服务名称
   
     datasource:
       type: com.alibaba.druid.pool.DruidDataSource  #当前数据源操作类型
       driver-class-name: com.mysql.cj.jdbc.Driver
       url: jdbc:mysql://locahost:3306/cloud?characterEncoding=utf8&useSSL=false&serverTimezone=UTC&rewriteBatchedStatements=true
       username: 
       password: 
   
   mybatis:
     mapper-locations: classpath:mapper/*.xml
     type-aliases-package: com.atguigu.springcloud.entities  #所有entity别名所在包
   
   ```

4. 主启动

   ```java
   package com.atguigu.springcloud;
   
   import org.springframework.boot.SpringApplication;
   import org.springframework.boot.autoconfigure.SpringBootApplication;
   
   @SpringBootApplication
   public class PaymentMain8001 {
       public static void main(String[] args) {
           SpringApplication.run(PaymentMain8001.class,args);
       }
   }
   
   ```

## 2.业务

1. 数据库

   ```sql
   create table payment(
       id bigint(20) not null auto_increment primary key comment 'id',
       serial varchar(200) default ''
   )
   ```

2. entities

   ```java
   package com.atguigu.springcloud.entities;
   
   import lombok.AllArgsConstructor;
   import lombok.Data;
   import lombok.NoArgsConstructor;
   
   import java.io.Serializable;
   
   @Data
   @AllArgsConstructor
   @NoArgsConstructor
   public class Payment implements Serializable {
       private long id;
       private String serial;
   }
   
   ```

   > 前后端分离 封装
   >
   > ```java
   > package com.atguigu.springcloud.entities;
   > 
   > import lombok.AllArgsConstructor;
   > import lombok.Data;
   > import lombok.NoArgsConstructor;
   > 
   > @Data
   > @AllArgsConstructor
   > @NoArgsConstructor
   > public class CommonResult<T> {
   >     //code + 说明
   >     private Integer code;
   >     private String  message;
   >     private T       data;
   >     public CommonResult(Integer code,String message) {
   >         this(code,message,null);
   >     }
   > }
   > 
   > ```

3. dao

   - dao

     ```java
     package com.atguigu.springcloud.dao;
     
     import com.atguigu.springcloud.entities.Payment;
     import org.apache.ibatis.annotations.Mapper;
     import org.apache.ibatis.annotations.Param;
     
     @Mapper
     public interface PaymentDao {
         //add
         public int create(Payment payment);
         //select
         public Payment getPaymentById(@Param("id") Long id);
     }
     
     ```

   - mapper(resources/mapper/PaymentMapper.xml)

     ```xml
     <?xml version="1.0" encoding="UTF-8" ?>
     <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
     
     <mapper namespace="com.atguigu.springcloud.dao.PaymentDao">
     
         <resultMap id="BaseResultMap" type="Payment">
             <id column="id" property="id" jdbcType="BIGINT"/>
             <id column="serial" property="serial" jdbcType="VARCHAR"/>
         </resultMap>
     <!-- useGeneratedKeys="true" keyProperty="id" 获取自动生成的主键值 -->
         <insert id="create" parameterType="Payment" useGeneratedKeys="true" keyProperty="id">
             insert into payment(serial)  values(#{serial});
         </insert>
     
         <select id="getPaymentById" parameterType="Long" resultMap="BaseResultMap">
             select * from payment where id=#{id};
         </select>
     
     </mapper>
     
     ```

   - yml配置

     ```yml
     mybatis:
       mapper-locations: classpath:mapper/*.xml
       type-aliases-package: com.atguigu.springcloud.entities  #所有entity别名所在包
     
     ```

4. Service

   - 接口

     ```java
     package com.atguigu.springcloud.service;
     
     import com.atguigu.springcloud.entities.Payment;
     
     public interface PaymentService {
         //add
         public int create(Payment payment);
         //select
         public Payment getPaymentById(Long id);
     }
     
     ```

   - 实现类

     ```java
     package com.atguigu.springcloud.service.impl;
     
     import com.atguigu.springcloud.dao.PaymentDao;
     import com.atguigu.springcloud.entities.Payment;
     import com.atguigu.springcloud.service.PaymentService;
     import org.springframework.beans.factory.annotation.Autowired;
     import org.springframework.stereotype.Service;
     
     @Service
     public class PaymentServiceImpl implements PaymentService {
         @Autowired
         private PaymentDao paymentDao;
     
         @Override
         public int create(Payment payment) {
             int i = paymentDao.create(payment);
             return i;
         }
     
         @Override
         public Payment getPaymentById(Long id) {
             Payment payment = paymentDao.getPaymentById(id);
             return payment;
         }
     }
     
     ```

5. controller

   ```java
   package com.atguigu.springcloud.controller;
   
   import com.atguigu.springcloud.entities.CommonResult;
   import com.atguigu.springcloud.entities.Payment;
   import com.atguigu.springcloud.service.PaymentService;
   import lombok.extern.slf4j.Slf4j;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.stereotype.Controller;
   import org.springframework.web.bind.annotation.*;
   
   @RestController
   @Slf4j
   @RequestMapping("/payment")
   public class PaymentController {
       @Autowired
       private PaymentService paymentService;
   
       @PostMapping(value = "/create")
       public CommonResult create(Payment payment){
           int result = paymentService.create(payment);
           log.info("结果："+ result);
           if (result > 0){
               return new CommonResult(200,"插入成功",result);
           }
           return new CommonResult(444,"插入错误",null);
       }
       @GetMapping(value = "/get/{id}")
       public CommonResult getPamentById(@PathVariable("id")Long id){
           Payment paymentById = paymentService.getPaymentById(id);
           log.info("结果："+ paymentById);
           if (paymentById != null){
               return new CommonResult(200,"查询成功",paymentById);
           }
           return new CommonResult(444,"没有对应记录"+id,null);
       }
   }
   
   ```

   


