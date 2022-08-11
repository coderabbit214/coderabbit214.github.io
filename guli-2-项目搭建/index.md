# guli-2-项目搭建


# guli-2-项目搭建

## 一.工程目录

- guli-parent：在线教学根目录（父工程），管理四个子模块： 
  - canal-client：canal数据库表同步模块（统计同步数据） 
  - common：公共模块父节点 
    - common-util：工具类模块，所有模块都可以依赖于它 
    - service-base：service服务的base包，包含service服务的公共配置类，所有service模块依赖于它 
    - spring-security：认证与授权模块，需要认证授权的service服务依赖于它 
  - infrastructure：基础服务模块父节点 
    - api-gateway：api网关服务 
  
  - service：api接口服务父节点 
    - service-acl：用户权限管理api接口服务（用户管理、角色管理和权限管理等） 
    - service-cms：cms api接口服务 
    - service-edu：教学相关api接口服务 
    - service-msm：短信api接口服务 
    - service-order：订单相关api接口服务 
    - service-oss：阿里云oss api接口服务 
    - service-statistics：统计报表api接口服务 
    - service-ucenter：会员api接口服务 
    - service-vod：视频点播api接口服务

## 二.创建父工程

1. 创建sprigboot工程guli-parent

   > 在idea开发工具中，使用 Spring Initializr 快速初始化一个 Spring Boot 模块，版本使用：2.2.1.RELEASE

2. 删除 src 目录

3. 配置 pom.xml

   - 修改版本为 ：2.2.1.RELEASE

     ```xml
      <parent>
             <groupId>org.springframework.boot</groupId>
             <artifactId>spring-boot-starter-parent</artifactId>
             <version>2.2.1.RELEASE</version>
             <relativePath/> <!-- lookup parent from repository -->
         </parent>
     ```

   - 添加 pom类型

     ```xml
     <artifactId>guli_parent</artifactId>
         <packaging>pom</packaging>
     ```

4. 在pom.xml中添加依赖的版本

   - 删除pom.xml中的内容

     ```xml
     <!-- 以下内容删除 -->
     <dependencies>
     <dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter</artifactId>
     </dependency>
     <dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-test</artifactId>
     <scope>test</scope>
     </dependency>
     </dependencies>
     ```

   - 添加依赖的版本

     ```xml
     <properties>
             <java.version>1.8</java.version>
             <guli.version>0.0.1-SNAPSHOT</guli.version>
             <mybatis-plus.version>3.0.5</mybatis-plus.version>
             <velocity.version>2.0</velocity.version>
             <swagger.version>2.7.0</swagger.version>
             <aliyun.oss.version>2.8.3</aliyun.oss.version>
             <jodatime.version>2.10.1</jodatime.version>
             <poi.version>3.17</poi.version>
             <commons-fileupload.version>1.3.1</commons-fileupload.version>
             <commons-io.version>2.6</commons-io.version>
             <httpclient.version>4.5.1</httpclient.version>
             <jwt.version>0.7.0</jwt.version>
             <aliyun-java-sdk-core.version>4.3.3</aliyun-java-sdk-core.version>
             <aliyun-sdk-oss.version>3.1.0</aliyun-sdk-oss.version>
             <aliyun-java-sdk-vod.version>2.15.2</aliyun-java-sdk-vod.version>
             <aliyun-java-vod-upload.version>1.4.11</aliyun-java-vod-upload.version>
             <aliyun-sdk-vod-upload.version>1.4.11</aliyun-sdk-vod-upload.version>
             <fastjson.version>1.2.28</fastjson.version>
             <gson.version>2.8.2</gson.version>
             <json.version>20170516</json.version>
             <commons-dbutils.version>1.7</commons-dbutils.version>
             <canal.client.version>1.1.0</canal.client.version>
             <docker.image.prefix>zx</docker.image.prefix>
             <cloud-alibaba.version>0.2.2.RELEASE</cloud-alibaba.version>
         </properties>
     ```

   - dependencyManagement配置

     ```xml
     <dependencyManagement>
             <dependencies>
                 <!--Spring Cloud-->
                 <dependency>
                     <groupId>org.springframework.cloud</groupId>
                     <artifactId>spring-cloud-dependencies</artifactId>
                     <version>Hoxton.RELEASE</version>
                     <type>pom</type>
                     <scope>import</scope>
                 </dependency>
                 <dependency>
                     <groupId>org.springframework.cloud</groupId>
                     <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                     <version>${cloud-alibaba.version}</version>
                     <type>pom</type>
                     <scope>import</scope>
                 </dependency>
                 <!--mybatis-plus 持久层-->
                 <dependency>
                     <groupId>com.baomidou</groupId>
                     <artifactId>mybatis-plus-boot-starter</artifactId>
                     <version>${mybatis-plus.version}</version>
                 </dependency>
                 <!-- velocity 模板引擎, Mybatis Plus 代码生成器需要 -->
                 <dependency>
                     <groupId>org.apache.velocity</groupId>
                     <artifactId>velocity-engine-core</artifactId>
                     <version>${velocity.version}</version>
                 </dependency>
                 <!--swagger-->
                 <dependency>
                     <groupId>io.springfox</groupId>
                     <artifactId>springfox-swagger2</artifactId>
                     <version>${swagger.version}</version>
                 </dependency>
                 <!--swagger ui-->
                 <dependency>
                     <groupId>io.springfox</groupId>
                     <artifactId>springfox-swagger-ui</artifactId>
                     <version>${swagger.version}</version>
                 </dependency>
                 <!--aliyunOSS-->
                 <dependency>
                     <groupId>com.aliyun.oss</groupId>
                     <artifactId>aliyun-sdk-oss</artifactId>
                     <version>${aliyun.oss.version}</version>
                 </dependency>
                 <!--日期时间工具-->
                 <dependency>
                     <groupId>joda-time</groupId>
                     <artifactId>joda-time</artifactId>
                     <version>${jodatime.version}</version>
                 </dependency>
                 <!--xls-->
                 <dependency>
                     <groupId>org.apache.poi</groupId>
                     <artifactId>poi</artifactId>
                     <version>${poi.version}</version>
                 </dependency>
                 <!--xlsx-->
                 <dependency>
                     <groupId>org.apache.poi</groupId>
                     <artifactId>poi-ooxml</artifactId>
                     <version>${poi.version}</version>
                 </dependency>
                 <!--文件上传-->
                 <dependency>
                     <groupId>commons-fileupload</groupId>
                     <artifactId>commons-fileupload</artifactId>
                     <version>${commons-fileupload.version}</version>
                 </dependency>
                 <!--commons-io-->
                 <dependency>
                     <groupId>commons-io</groupId>
                     <artifactId>commons-io</artifactId>
                     <version>${commons-io.version}</version>
                 </dependency>
                 <!--httpclient-->
                 <dependency>
                     <groupId>org.apache.httpcomponents</groupId>
                     <artifactId>httpclient</artifactId>
                     <version>${httpclient.version}</version>
                 </dependency>
                 <dependency>
                     <groupId>com.google.code.gson</groupId>
                     <artifactId>gson</artifactId>
                     <version>${gson.version}</version>
                 </dependency>
                 <!-- JWT -->
                 <dependency>
                     <groupId>io.jsonwebtoken</groupId>
                     <artifactId>jjwt</artifactId>
                     <version>${jwt.version}</version>
                 </dependency>
                 <!--aliyun-->
                 <dependency>
                     <groupId>com.aliyun</groupId>
                     <artifactId>aliyun-java-sdk-core</artifactId>
                     <version>${aliyun-java-sdk-core.version}</version>
                 </dependency>
                 <dependency>
                     <groupId>com.aliyun.oss</groupId>
                     <artifactId>aliyun-sdk-oss</artifactId>
                     <version>${aliyun-sdk-oss.version}</version>
                 </dependency>
                 <dependency>
                     <groupId>com.aliyun</groupId>
                     <artifactId>aliyun-java-sdk-vod</artifactId>
                     <version>${aliyun-java-sdk-vod.version}</version>
                 </dependency>
                 <dependency>
                     <groupId>com.aliyun</groupId>
                     <artifactId>aliyun-java-vod-upload</artifactId>
                     <version>${aliyun-java-vod-upload.version}</version>
                 </dependency>
                 <dependency>
                     <groupId>com.aliyun</groupId>
                     <artifactId>aliyun-sdk-vod-upload</artifactId>
                     <version>${aliyun-sdk-vod-upload.version}</version>
                 </dependency>
                 <dependency>
                     <groupId>com.alibaba</groupId>
                     <artifactId>fastjson</artifactId>
                     <version>${fastjson.version}</version>
                 </dependency>
                 <dependency>
                     <groupId>org.json</groupId>
                     <artifactId>json</artifactId>
                     <version>${json.version}</version>
                 </dependency>
                 <dependency>
                     <groupId>commons-dbutils</groupId>
                     <artifactId>commons-dbutils</artifactId>
                     <version>${commons-dbutils.version}</version>
                 </dependency>
                 <dependency>
                     <groupId>com.alibaba.otter</groupId>
                     <artifactId>canal.client</artifactId>
                     <version>${canal.client.version}</version>
                 </dependency>
             </dependencies>
         </dependencyManagement>
     
     ```

## 三.搭建Service模块

1. 在父工程guli-parent下面创建模块service

2. 删除src

3. 添加模块类型是pom

   ```xml
   <artifactId>service</artifactId>
   <packaging>pom</packaging>
   ```

4. 添加项目需要的依赖

   ```xml
       <dependencies>
   <!--        <dependency>-->
   <!--            <groupId>org.springframework.cloud</groupId>-->
   <!--            <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>-->
   <!--        </dependency>-->
   <!--        hystrix依赖，主要是用 @HystrixCommand-->
   <!--        <dependency>-->
   <!--            <groupId>org.springframework.cloud</groupId>-->
   <!--            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>-->
   <!--        </dependency>-->
   <!--        服务注册-->
   <!--        <dependency>-->
   <!--            <groupId>org.springframework.cloud</groupId>-->
   <!--            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>-->
   <!--        </dependency>-->
   <!--        服务调用-->
   <!--        <dependency>-->
   <!--            <groupId>org.springframework.cloud</groupId>-->
   <!--            <artifactId>spring-cloud-starter-openfeign</artifactId>-->
   <!--        </dependency>-->
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-web</artifactId>
           </dependency>
           <!--mybatis-plus-->
           <dependency>
               <groupId>com.baomidou</groupId>
               <artifactId>mybatis-plus-boot-starter</artifactId>
           </dependency>
           <!--mysql-->
           <dependency>
               <groupId>mysql</groupId>
               <artifactId>mysql-connector-java</artifactId>
           </dependency>
           <!-- velocity 模板引擎, Mybatis Plus 代码生成器需要 -->
           <dependency>
               <groupId>org.apache.velocity</groupId>
               <artifactId>velocity-engine-core</artifactId>
           </dependency>
           <!--swagger-->
           <dependency>
               <groupId>io.springfox</groupId>
               <artifactId>springfox-swagger2</artifactId>
           </dependency>
           <dependency>
               <groupId>io.springfox</groupId>
               <artifactId>springfox-swagger-ui</artifactId>
           </dependency>
           <!--lombok用来简化实体类：需要安装lombok插件-->
           <dependency>
               <groupId>org.projectlombok</groupId>
               <artifactId>lombok</artifactId>
           </dependency>
           <!--xls-->
           <dependency>
               <groupId>org.apache.poi</groupId>
               <artifactId>poi</artifactId>
           </dependency>
           <dependency>
               <groupId>org.apache.poi</groupId>
               <artifactId>poi-ooxml</artifactId>
           </dependency>
           <dependency>
               <groupId>commons-fileupload</groupId>
               <artifactId>commons-fileupload</artifactId>
           </dependency>
           <!--httpclient-->
           <dependency>
               <groupId>org.apache.httpcomponents</groupId>
               <artifactId>httpclient</artifactId>
           </dependency>
           <!--commons-io-->
           <dependency>
               <groupId>commons-io</groupId>
               <artifactId>commons-io</artifactId>
           </dependency>
           <!--gson-->
           <dependency>
               <groupId>com.google.code.gson</groupId>
               <artifactId>gson</artifactId>
           </dependency>
           <dependency>
               <groupId>junit</groupId>
               <artifactId>junit</artifactId>
               <version>4.12</version>
           </dependency>
       </dependencies>
   ```

## 四.搭建service-edu模块

1. 在父工程service模块下面创建子模块service-edu

2. 配置 在service下面service-edu模块中创建配置文件

   ```properties
   #数据库配置
   spring.datasource.url=jdbc:mysql://localhost:3306/guli?serverTimezone=GMT%2B8
   spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
   spring.datasource.username=root
   spring.datasource.password=123456
   
   #mybatis日志
   mybatis.configuration.log-impl=org.apache.ibatis.logging.stdout.StdOutImpl
   
   #环境设置:dev,test,prod
   spring.profiles.active=dev
   
   #服务端口
   server.port=8001
   
   #服务名
   spring.application.name=service_edu
   
   #设置时间格式
   spring.jackson.date-format=yyyy-MM-dd HH:mm:ss
   spring.jackson.time-zone=GMT+8
   ```

### 3.MP代码生成器

> 在test/java目录下创建包com.guli.demo，创建代码生成器：CodeGenerator.java

```java
package com.guli.demo;

import com.baomidou.mybatisplus.annotation.DbType;
import com.baomidou.mybatisplus.annotation.IdType;
import com.baomidou.mybatisplus.generator.AutoGenerator;
import com.baomidou.mybatisplus.generator.config.DataSourceConfig;
import com.baomidou.mybatisplus.generator.config.GlobalConfig;
import com.baomidou.mybatisplus.generator.config.PackageConfig;
import com.baomidou.mybatisplus.generator.config.StrategyConfig;
import com.baomidou.mybatisplus.generator.config.rules.DateType;
import com.baomidou.mybatisplus.generator.config.rules.NamingStrategy;
import org.junit.Test;

/**
 * @author
 * @since 2018/12/13
 */
public class CodeGenerator {

    @Test
    public void run() {

        // 1、创建代码生成器
        AutoGenerator mpg = new AutoGenerator();

        // 2、全局配置
        GlobalConfig gc = new GlobalConfig();
        String projectPath = System.getProperty("user.dir");
        gc.setOutputDir("E:\\JAVA存储\\Web\\IDEA\\guli_parent\\service\\service_edu" + "/src/main/java");

        gc.setAuthor("testjava");
        gc.setOpen(false); //生成后是否打开资源管理器
        gc.setFileOverride(false); //重新生成时文件是否覆盖

        //UserServie
        gc.setServiceName("%sService");	//去掉Service接口的首字母I

        gc.setIdType(IdType.ID_WORKER_STR); //主键策略 3.3.0以后需要更改
        gc.setDateType(DateType.ONLY_DATE);//定义生成的实体类中日期类型
        gc.setSwagger2(true);//开启Swagger2模式

        mpg.setGlobalConfig(gc);

        // 3、数据源配置
        DataSourceConfig dsc = new DataSourceConfig();
        dsc.setUrl("jdbc:mysql://localhost:3306/guli?serverTimezone=GMT%2B8");
        dsc.setDriverName("com.mysql.cj.jdbc.Driver");
        dsc.setUsername("root");
        dsc.setPassword("123456");
        dsc.setDbType(DbType.MYSQL);
        mpg.setDataSource(dsc);

        // 4、包配置
        PackageConfig pc = new PackageConfig();
        //包  com.guli.eduservice
        pc.setParent("com.guli");
        pc.setModuleName("eduservice"); //模块名

        //包  com.atguigu.eduservice.controller
        pc.setController("controller");
        pc.setEntity("entity");
        pc.setService("service");
        pc.setMapper("mapper");
        mpg.setPackageInfo(pc);

        // 5、策略配置
        StrategyConfig strategy = new StrategyConfig();

        strategy.setInclude("edu_teacher");//表名，可以加入多张表 用","隔开

        strategy.setNaming(NamingStrategy.underline_to_camel);//数据库表映射到实体的命名策略
        strategy.setTablePrefix(pc.getModuleName() + "_"); //生成实体时去掉"_"变成大写

        strategy.setColumnNaming(NamingStrategy.underline_to_camel);//数据库表字段映射到实体的命名策略
        strategy.setEntityLombokModel(true); // lombok 模型 @Accessors(chain = true) setter链式操作

        strategy.setRestControllerStyle(true); //restful api风格控制器
        strategy.setControllerMappingHyphenStyle(true); //url中驼峰转连字符

        mpg.setStrategy(strategy);


        // 6、执行
        mpg.execute();
    }
}

```

4. 创建SpringBoot配置类

   > 在edu包下创建config包，创建EduConfig.java

   ```java
   @Configuration
   @MapperScan("com.guli.eduservice.mapper")
   public class EduConfig {
      
   }
   ```

5. 创建SpringBoot启动类,注意启动类的创建位置

   ```java
   @SpringBootApplication
   public class EduApplication {
       public static void main(String[] args) {
           SpringApplication.run(EduApplication.class,args);
       }
   }
   ```

### 6.讲师逻辑删除功能

- EduTeacherController添加删除方法

  ```java
  @Autowired
      private EduTeacherService eduTeacherService;
  
  @DeleteMapping("{id}")
      public R deleteTeacherById(
              @ApiParam(name = "id",value = "讲师id",required = true)
              @PathVariable String id){
          return eduTeacherService.removeById(id);
  
      }
  ```

- 配置逻辑删除插件 EduConfig

  ```java
   //逻辑删除插件
      @Bean
      public ISqlInjector sqlInjector() {
          return new LogicSqlInjector();
      }
  ```

## 五.Swagger2

1. 创建common模块 

   在guli-parent下创建模块common 

   配置： groupId：com.guli     artifactId：common

2. 在common中引入相关依赖

   ```xml
   <packaging>pom</packaging>
   
   <dependencies>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-web</artifactId>
               <scope>provided </scope>
           </dependency>
           <!--mybatis-plus-->
           <dependency>
               <groupId>com.baomidou</groupId>
               <artifactId>mybatis-plus-boot-starter</artifactId>
               <scope>provided </scope>
           </dependency>
           <!--lombok用来简化实体类：需要安装lombok插件-->
           <dependency>
               <groupId>org.projectlombok</groupId>
               <artifactId>lombok</artifactId>
               <scope>provided </scope>
           </dependency>
           <!--swagger-->
           <dependency>
               <groupId>io.springfox</groupId>
               <artifactId>springfox-swagger2</artifactId>
               <scope>provided </scope>
           </dependency>
           <dependency>
               <groupId>io.springfox</groupId>
               <artifactId>springfox-swagger-ui</artifactId>
               <scope>provided </scope>
           </dependency>
           <!-- redis -->
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-data-redis</artifactId>
           </dependency>
           <!-- spring2.X集成redis所需common-pool2
           <dependency>
           <groupId>org.apache.commons</groupId>
           <artifactId>commons-pool2</artifactId>
           <version>2.6.0</version>
           </dependency>-->
       </dependencies>
   ```

3. 在common下面创建子模块service-base

4. 在模块service-base中，创建swagger的配置类

   - 创建包com.guli.servicebase，创建类SwaggerConfig

     ```java
     package com.guli.servicebase;
     
     @Configuration
     @EnableSwagger2//Swagger注解
     public class SwaggerConfig {
         @Bean
         public Docket webApiConfig(){
             return new Docket(DocumentationType.SWAGGER_2)
                     .groupName("webApi")
                     .apiInfo(webApiInfo())
                     .select()
                     .paths(Predicates.not(PathSelectors.regex("/admin/.*")))
                     .paths(Predicates.not(PathSelectors.regex("/error.*")))
                     .build();
         }
     
         private ApiInfo webApiInfo(){
             return new ApiInfoBuilder()
                     .title("网站-课程中心API文档")
                     .description("本文档描述了课程中心微服务接口定义")
                     .version("1.0")
                     .contact(new Contact("Mr.J", "233333",
                             "1157237955@qq.com"))
                     .build();
         }
     }
     
     ```

5. 在模块service模块中引入service-base

   ```xml
   <dependency>
    <groupId>com.atguigu</groupId>
    <artifactId>service-base</artifactId>
    <version>0.0.1-SNAPSHOT</version>
   </dependency>
   ```

6. 在service-edu启动类上添加注解，进行测试

   > 目的是扫描到其他项目的com.guli包
   >
   > @ComponentScan(basePackages = "com.guli")

   ```java
   @SpringBootApplication
   @ComponentScan(basePackages = "com.guli")
   public class EduApplication {
       public static void main(String[] args) {
           SpringApplication.run(EduApplication.class,args);
       }
   }
   ```

7. API模型

   - 可以添加一些自定义设置，例如： 定义样例数据

     ```java
         @ApiModelProperty(value = "查询开始时间", example = "2019-01-01 10:10:10")
         private String begin;//注意，这里使用的是String类型，前端传过来的数据无需进行类型转换
         @ApiModelProperty(value = "查询结束时间", example = "2019-12-01 10:10:10")
         private String end;
     ```

8. 定义接口说明和参数说明

   > 定义在类上：@Api   
   >
   > @Api(tags = "讲师管理")
   >
   > 
   >
   > 定义在方法上：@ApiOperation   
   >
   > @ApiOperation(value = "所有讲师列表")
   >
   > 
   >
   > 定义在参数上：@ApiParam
   >
   > @ApiParam(name = "id",value = "讲师id",required = true)

## 六.统一返回数据格式

> 项目中我们会将响应封装成json返回，一般我们会将所有接口的数据格式统一， 使前端(iOS Android, Web)对数据的操作更一致、轻松。
>
>  一般情况下，统一返回数据格式没有固定的格式，只要能描述清楚返回的数据状态以及要返回的具体数 据就可以。但是一般会包含状态码、返回消息、数据这几部分内容 
>
> 例如，我们的系统要求返回的基本数据格式如下：

列表：

```json
{
 "success": true,
 "code": 20000,
 "message": "成功",
 "data": {
 "items": [
 {
 "id": "1",
 "name": "刘德华",
 "intro": "毕业于师范大学数学系，热爱教育事业，执教数学思维6年有余"
 }
 ]
 }
}
```

分页

```json
{
 "success": true,
 "code": 20000,
 "message": "成功",
 "data": {
 "total": 17,
 "rows": [
 {
 "id": "1",
 "name": "刘德华",
 "intro": "毕业于师范大学数学系，热爱教育事业，执教数学思维6年有余"
  }
 ]
 }
}
```

没有返回数据：

```
{
 "success": true,
 "code": 20000,
 "message": "成功",
 "data": {}
}
```

失败：

```
{
 "success": false,
 "code": 20001,
 "message": "失败",
 "data": {}
}
```

因此，我们定义统一结果

```
{
 "success": 布尔, //响应是否成功
 "code": 数字, //响应码
 "message": 字符串, //返回消息
 "data": HashMap //返回数据，放在键值对中
}
```

### 1.创建统一结果返回类

1. 在common模块下创建子模块common-utils

2. 创建接口定义返回码

   > 创建包com.guli.commonutils，创建接口 ResultCode.java

   ```java
   package com.guli.commonutils;
   
   public interface ResultCode {
       public static Integer SUCCESS = 20000;//成功
       public static Integer ERROR = 20001;//失败
   }
   
   ```

3. 创建结果类

   ```java
   package com.guli.commonutils;
   
   import io.swagger.annotations.ApiModelProperty;
   import lombok.Data;
   
   import java.util.HashMap;
   import java.util.Map;
   
   //统一返回结果的类
   @Data
   public class R {
       @ApiModelProperty(value = "是否成功")
       private Boolean success;
   
       @ApiModelProperty(value = "返回码")
       private Integer code;
   
       @ApiModelProperty(value = "返回消息")
       private String message;
   
       @ApiModelProperty(value = "返回数据")
       private Map<String, Object> data = new HashMap<String, Object>();
   
       //构造方法私有化
       private R(){}
   
       //链式编程
       //成功静态方法
       public static R ok(){
           R r = new R();
           r.setSuccess(true);
           r.setCode(ResultCode.SUCCESS);
           r.setMessage("成功");
           return r;
       }
   
       //失败静态方法
       public static R error(){
           R r = new R();
           r.setSuccess(false);
           r.setCode(ResultCode.ERROR);
           r.setMessage("失败");
           return r;
       }
   
       public R success(Boolean success){
           this.setSuccess(success);
           return this;
       }
       public R message(String message){
           this.setMessage(message);
           return this;
       }
       public R code(Integer code){
           this.setCode(code);
           return this;
       }
       public R data(String key, Object value){
           this.data.put(key, value);
           return this;
       }
       public R data(Map<String, Object> map) {
           this.setData(map);
           return this;
       }
   }
   
   ```

### 2.统一返回结果使用

1. 在service模块中添加依赖

   ```xml
   <dependency>
    <groupId>com.atguigu</groupId>
    <artifactId>common_utils</artifactId>
    <version>0.0.1-SNAPSHOT</version>
   </dependency>
   ```

2. 修改Controller中的返回结果

   ```java
    @Autowired
       private EduTeacherService eduTeacherService;
       //1.查询所有数据 
       @ApiOperation(value = "所有讲师列表")
       @GetMapping("findAll")
       public R findAllTeacher() {
           List<EduTeacher> eduTeachers = eduTeacherService.list(null);
   
           return R.ok().data("items",eduTeachers);
       }
       //2.逻辑删除讲师
       @ApiOperation(value = "根据id逻辑删除讲师")
       @DeleteMapping("{id}")
       public R deleteTeacherById(
               @ApiParam(name = "id",value = "讲师id",required = true)
               @PathVariable String id){
           boolean flag = eduTeacherService.removeById(id);
           if (flag) {
               return R.ok();
           }else {
               return R.error();
           }
       }
   ```

## 七.分页

1. EduConfig中配置分页插件

   ```java
   @Configuration
   @MapperScan("com.guli.eduservice.mapper")
   public class EduConfig {
       //逻辑删除插件
       @Bean
       public ISqlInjector sqlInjector() {
           return new LogicSqlInjector();
       }
       //分页插件
       @Bean
       public PaginationInterceptor paginationInterceptor(){
           return new PaginationInterceptor();
       }
   }
   ```

2. 分页Controller方法

   ```java
   //3.分页查询 当前页（current）每页数据个数（limit）
       @ApiOperation(value = "分页查询")
       @GetMapping("pageTeacher/{current}/{limit}")
       public R pageTeacher(
               @PathVariable long current,
               @PathVariable long limit) {
           //两个参数：当前页，每页数据个数
           Page<EduTeacher> pageTeacher = new Page<>(current,limit);
           //返回数据存储在pageTeacher对象中
           eduTeacherService.page(pageTeacher,null);
           long total = pageTeacher.getTotal();//总记录数
           List<EduTeacher> teachers = pageTeacher.getRecords();//每页教师数据集合
   
           return R.ok().data("total",total).data("rows",teachers);
       }
   ```

3. Swagger中测试

## 八.条件查询

> 根据讲师名称name，讲师头衔level、讲师入驻时间gmt_create（时间段）查询

1. 创建查询对象

   > 创建com.guli.eduservice.entity.vo包，创建TeacherQuery.java查询对象

   ```java
   package com.guli.eduservice.entity.vo;
   
   @Data
   public class TeacherQuery {
       private static final long serialVersionUID = 1L;
       @ApiModelProperty(value = "教师名称,模糊查询")
       private String name;
       @ApiModelProperty(value = "头衔 1高级讲师 2首席讲师")
       private Integer level;
       @ApiModelProperty(value = "查询开始时间", example = "2019-01-01 10:10:10")
       private String begin;//注意，这里使用的是String类型，前端传过来的数据无需进行类型转换
       @ApiModelProperty(value = "查询结束时间", example = "2019-12-01 10:10:10")
       private String end;
   }
   
   ```

2. controller

   ```
   //4.分页带条件
       @ApiOperation(value = "条件分页查询")
       @PostMapping("pageTeacherCondition/{current}/{limit}")
       public R pageTeacherCondition(
               @PathVariable long current,
               @PathVariable long limit,
               //RequestBody 只能使用Post,required = false表示这个值可以没有
               @RequestBody(required = false) TeacherQuery teacherQuery) {
           //1.创建page对象
           Page<EduTeacher> pageTeacher = new Page<>(current,limit);
           //2.构建Wrapper
           QueryWrapper<EduTeacher> queryWrapper = new QueryWrapper<>();
           //3.多条件组合查询
           String name = teacherQuery.getName();
           Integer level = teacherQuery.getLevel();
           String begin = teacherQuery.getBegin();
           String end = teacherQuery.getEnd();
           //判断条件值是否为空，如果不为空拼接条件
           if (!StringUtils.isEmpty(name)){
               queryWrapper.like("name",name);
           }
           if (!StringUtils.isEmpty(level)){
               queryWrapper.eq("level",level);
           }
           if (!StringUtils.isEmpty(begin)){
               queryWrapper.ge("gmt_create",begin);
           }
           if (!StringUtils.isEmpty(end)){
               queryWrapper.eq("gmt_modified",end);
           }
   
           //查询
           eduTeacherService.page(pageTeacher,queryWrapper);
           long total = pageTeacher.getTotal();//总记录数
           List<EduTeacher> teachers = pageTeacher.getRecords();//每页教师数据集合
           return R.ok().data("total",total).data("rows",teachers);
       }
   ```

3. Swagger中测试

## 九.自动填充

1. 在service-base模块中添加

   - 创建包handler，创建自动填充类 MyMetaObjectHandler

     ```java
     package com.guli.servicebase.handler;
     
     /**
      * 初始化控制器
      */
     @Component
     public class MyMetaObjectHandler implements MetaObjectHandler {
         @Override
         public void insertFill(MetaObject metaObject) {
             //属性名称不是字段名称
             this.setFieldValByName("gmtCreate", new Date(), metaObject);
             this.setFieldValByName("gmtModified", new Date(), metaObject);
         }
         @Override
         public void updateFill(MetaObject metaObject) {
             this.setFieldValByName("gmtModified", new Date(), metaObject);
         }
     }
     
     ```

2. 在实体类添加自动填充注解

   ```java
       @ApiModelProperty(value = "创建时间")
       @TableField(fill = FieldFill.INSERT)
       private Date gmtCreate;
   
       @ApiModelProperty(value = "更新时间")
       @TableField(fill = FieldFill.INSERT_UPDATE)
       private Date gmtModified;
   ```

## 十.Controller中其他方法

```java
//添加讲师
    @ApiOperation(value = "添加讲师")
    @PostMapping("addTeacher")
    public R addTeacher(@RequestBody EduTeacher eduTeacher) {
        boolean flag = eduTeacherService.save(eduTeacher);
        if (flag) {
            return R.ok();
        } else {
            return R.error();
        }
    }
    //根据id查询讲师
    @ApiOperation(value = "根据id查询讲师")
    @GetMapping("selectTeacherById/{id}")
    public R selectTeacherById(@PathVariable String id) {
        EduTeacher teacher = eduTeacherService.getById(id);
        return R.ok().data("teacher",teacher);
    }
    //修改讲师
    @ApiOperation(value = "修改讲师")
    @PutMapping("updateTeacher")
    public R updateTeacher(@RequestBody EduTeacher eduTeacher) {
        boolean flag = eduTeacherService.updateById(eduTeacher);
        if (flag) {
            return R.ok();
        } else {
            return R.error();
        }
    }
```

## 十一.统一异常处理

> 我们想让异常结果也显示为统一的返回结果对象，并且统一处理系统的异常信息，那么需要统一异常处理

1. 创建统一异常处理器

   > 需要引入 common_utils中的R类
   >
   > ```xml
   > <dependencies>
   >     <dependency>
   >         <groupId>com.guli</groupId>
   >         <artifactId>common_utils</artifactId>
   >         <version>0.0.1-SNAPSHOT</version>
   >     </dependency>
   > </dependencies>
   > ```

   在service-base中创建统一异常处理类GlobalExceptionHandler.java：

   ```java
   package com.guli.servicebase.exceptionHandler;
   /**
    * 统一异常处理类
    */
   @ControllerAdvice
   public class GlobalExceptionHandler {
       //异常处理器：指定处理什么异常
       @ExceptionHandler(Exception.class)
       //为了返回数据
       @ResponseBody
       public R error(Exception e){
           e.printStackTrace();
           return R.error().message("执行了全局异常处理");
       }
   }
   
   ```


## 十二.特定异常处理

> GlobalExceptionHandler.java中添加
>
> 在@ExceptionHandler指定异常类型即可

```java
    //特定异常
    @ExceptionHandler(ArithmeticException.class)
    @ResponseBody
    public R error(ArithmeticException e){
        e.printStackTrace();
        return R.error().message("执行了ArithmeticException异常处理");
    }
```

## 十三.自定义异常

1. 创建自定义异常类

   ```java
   package com.guli.servicebase.exception;
   
   import lombok.AllArgsConstructor;
   import lombok.Data;
   import lombok.NoArgsConstructor;
   
   @Data
   @AllArgsConstructor //生成有参构造
   @NoArgsConstructor  //生成无参构造
   public class GuliException extends RuntimeException{
       private Integer code;//状态码
       private String msg;//异常信息
   }
   
   ```

2. 业务中需要自己抛出指定异常

   ```java
   try {
    int a = 10/0;
   }catch(Exception e) {
    throw new GuliException(20001,"出现自定义异常");
   }
   ```

3. 添加异常处理方法

   > GlobalExceptionHandler.java中添加

   ```java
       //自定义异常
       @ExceptionHandler(GuliException.class)
       @ResponseBody
       public R error(GuliException e){
           log.error(ExceptionUtil.getMessage(e));
           e.printStackTrace();
           return R.error().code(e.getCode()).message(e.getMsg());
       }
   ```

## 十四.日志

1. 配置日志级别

   > 日志记录器（Logger）的行为是分等级的。如下表所示： 
   >
   > 分为：OFF、FATAL、ERROR、WARN、INFO、DEBUG、ALL 
   >
   > 默认情况下，spring boot从控制台打印出来的日志级别只有INFO及以上级别，可以配置日志级别

   ```properties
   # 设置日志级别
   logging.level.root=WARN
   ```

2. Logback日志

   1. 配置logback日志

      - 删除application.properties中的日志配置

        ```properties
        ##mybatis日志
        #mybatis.configuration.log-impl=org.apache.ibatis.logging.stdout.StdOutImpl
        #
        ##设置日志级别
        #logging.level.root=info
        
        ```

      - resources 中创建 logback-spring.xml

        ```xml
        <?xml version="1.0" encoding="UTF-8"?>
        <configuration scan="true" scanPeriod="10 seconds">
            <!-- 日志级别从低到高分为TRACE < DEBUG < INFO < WARN < ERROR < FATAL，如果设
            置为WARN，则低于WARN的信息都不会输出 -->
            <!-- scan:当此属性设置为true时，配置文件如果发生改变，将会被重新加载，默认值
            为true -->
            <!-- scanPeriod:设置监测配置文件是否有修改的时间间隔，如果没有给出时间单位，默认
            单位是毫秒。当scan为true时，此属性生效。默认的时间间隔为1分钟。 -->
            <!-- debug:当此属性设置为true时，将打印出logback内部日志信息，实时查
            看logback运行状态。默认值为false。 -->
            <contextName>logback</contextName>
            <!-- name的值是变量的名称，value的值时变量定义的值。通过定义的值会被插入
            到logger上下文中。定义变量后，可以使“${}”来使用变量。 -->
            <property name="log.path" value="E:/JAVA存储/Web/edu" />
            <!-- 彩色日志 -->
            <!-- 配置格式变量：CONSOLE_LOG_PATTERN 彩色日志格式 -->
            <!-- magenta:洋红 -->
            <!-- boldMagenta:粗红-->
            <!-- cyan:青色 -->
            <!-- white:白色 -->
            <!-- magenta:洋红 -->
            <property name="CONSOLE_LOG_PATTERN"
                      value="%yellow(%date{yyyy-MM-dd HH:mm:ss}) |%highlight(%-5level)
        |%blue(%thread) |%blue(%file:%line) |%green(%logger) |%cyan(%msg%n)"/>
            <!--输出到控制台-->
            <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
                <!--此日志appender是为开发使用，只配置最底级别，控制台输出的日志级别是大于或
                等于此级别的日志信息-->
                <!-- 例如：如果此处配置了INFO级别，则后面其他位置即使配置了DEBUG级别的日
                志，也不会被输出 -->
                <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
                    <level>INFO</level>
                </filter>
                <encoder>
                    <Pattern>${CONSOLE_LOG_PATTERN}</Pattern>
                    <!-- 设置字符集 -->
                    <charset>UTF-8</charset>
                </encoder>
            </appender>
            <!--输出到文件-->
            <!-- 时间滚动输出 level为 INFO 日志 -->
            <appender name="INFO_FILE"
                      class="ch.qos.logback.core.rolling.RollingFileAppender">
                <!-- 正在记录的日志文件的路径及文件名 -->
                <file>${log.path}/log_info.log</file>
                <!--日志文件输出格式-->
                <encoder>
                    <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level
                        %logger{50} - %msg%n</pattern>
                    <charset>UTF-8</charset>
                </encoder>
                <!-- 日志记录器的滚动策略，按日期，按大小记录 -->
                <rollingPolicy
                        class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
                    <!-- 每天日志归档路径以及格式 -->
                    <fileNamePattern>${log.path}/info/log-info-%d{yyyy-MMdd}.%i.log</fileNamePattern>
                    <timeBasedFileNamingAndTriggeringPolicy
                            class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                        <maxFileSize>100MB</maxFileSize>
                    </timeBasedFileNamingAndTriggeringPolicy>
                    <!--日志文件保留天数-->
                    <maxHistory>15</maxHistory>
                </rollingPolicy>
                <!-- 此日志文件只记录info级别的 -->
                <filter class="ch.qos.logback.classic.filter.LevelFilter">
                    <level>INFO</level>
                    <onMatch>ACCEPT</onMatch>
                    <onMismatch>DENY</onMismatch>
                </filter>
            </appender>
            <!-- 时间滚动输出 level为 WARN 日志 -->
            <appender name="WARN_FILE"
                      class="ch.qos.logback.core.rolling.RollingFileAppender">
                <!-- 正在记录的日志文件的路径及文件名 -->
                <file>${log.path}/log_warn.log</file>
                <!--日志文件输出格式-->
                <encoder>
                    <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level
                        %logger{50} - %msg%n</pattern>
                    <charset>UTF-8</charset> <!-- 此处设置字符集 -->
                </encoder>
                <!-- 日志记录器的滚动策略，按日期，按大小记录 -->
                <rollingPolicy
                        class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
                    <fileNamePattern>${log.path}/warn/log-warn-%d{yyyy-MMdd}.%i.log</fileNamePattern>
                    <timeBasedFileNamingAndTriggeringPolicy
                            class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                        <maxFileSize>100MB</maxFileSize>
                    </timeBasedFileNamingAndTriggeringPolicy>
                    <!--日志文件保留天数-->
                    <maxHistory>15</maxHistory>
                </rollingPolicy>
                <!-- 此日志文件只记录warn级别的 -->
                <filter class="ch.qos.logback.classic.filter.LevelFilter">
                    <level>warn</level>
                    <onMatch>ACCEPT</onMatch>
                    <onMismatch>DENY</onMismatch>
                </filter>
            </appender>
            <!-- 时间滚动输出 level为 ERROR 日志 -->
            <appender name="ERROR_FILE"
                      class="ch.qos.logback.core.rolling.RollingFileAppender">
                <!-- 正在记录的日志文件的路径及文件名 -->
                <file>${log.path}/log_error.log</file>
                <!--日志文件输出格式-->
                <encoder>
                    <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level
                        %logger{50} - %msg%n</pattern>
                    <charset>UTF-8</charset> <!-- 此处设置字符集 -->
                </encoder>
                <!-- 日志记录器的滚动策略，按日期，按大小记录 -->
                <rollingPolicy
                        class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
                    <fileNamePattern>${log.path}/error/log-error-%d{yyyy-MMdd}.%i.log</fileNamePattern>
                    <timeBasedFileNamingAndTriggeringPolicy
                            class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                        <maxFileSize>100MB</maxFileSize>
                    </timeBasedFileNamingAndTriggeringPolicy>
                    <!--日志文件保留天数-->
                    <maxHistory>15</maxHistory>
                </rollingPolicy>
                <!-- 此日志文件只记录ERROR级别的 -->
                <filter class="ch.qos.logback.classic.filter.LevelFilter">
                    <level>ERROR</level>
                    <onMatch>ACCEPT</onMatch>
                    <onMismatch>DENY</onMismatch>
                </filter>
            </appender>
            <!--
            <logger>用来设置某一个包或者具体的某一个类的日志打印级别、以及指
            定<appender>。
            <logger>仅有一个name属性，
            一个可选的level和一个可选的addtivity属性。
            name:用来指定受此logger约束的某一个包或者具体的某一个类。
            level:用来设置打印级别，大小写无关：TRACE, DEBUG, INFO, WARN, ERROR, ALL
            和 OFF，
            如果未设置此属性，那么当前logger将会继承上级的级别。
            -->
            <!--
            使用mybatis的时候，sql语句是debug下才会打印，而这里我们只配置了info，所以想
            要查看sql语句的话，有以下两种操作：
            第一种把<root level="INFO">改成<root level="DEBUG">这样就会打印sql，不过
        
            2、将错误日志输出到文件
            GlobalExceptionHandler.java 中
            类上添加注解
            异常输出语句
            这样日志那边会出现很多其他消息
            第二种就是单独给mapper下目录配置DEBUG模式，代码如下，这样配置sql语句会打
            印，其他还是正常DEBUG级别：
            -->
            <!--开发环境:打印控制台-->
            <springProfile name="dev">
                <!--可以输出项目中的debug日志，包括mybatis的sql日志-->
                <logger name="com.guli" level="INFO" />
                <!--
                root节点是必选节点，用来指定最基础的日志输出级别，只有一个level属性
                level:用来设置打印级别，大小写无关：TRACE, DEBUG, INFO, WARN, ERROR,
                ALL 和 OFF，默认是DEBUG
                可以包含零个或多个appender元素。
                -->
                <root level="INFO">
                    <appender-ref ref="CONSOLE" />
                    <appender-ref ref="INFO_FILE" />
                    <appender-ref ref="WARN_FILE" />
                    <appender-ref ref="ERROR_FILE" />
                </root>
            </springProfile>
            <!--生产环境:输出到文件-->
            <springProfile name="pro">
                <root level="INFO">
                    <appender-ref ref="CONSOLE" />
                    <appender-ref ref="DEBUG_FILE" />
                    <appender-ref ref="INFO_FILE" />
                    <appender-ref ref="ERROR_FILE" />
                    <appender-ref ref="WARN_FILE" />
                </root>
            </springProfile>
        </configuration>
        ```

   2. 将错误日志输出到文件（此种方法只能输出一句）

      - GlobalExceptionHandler.java 中 类上添加注解@Slf4j

      - 异常输出语句log.error(e.getMessage());

        ```java
        package com.guli.servicebase.exceptionHandler;
        /**
         * 统一异常处理类
         */
        @ControllerAdvice
        @Slf4j
        public class GlobalExceptionHandler {
            //自定义异常
            @ExceptionHandler(GuliException.class)
            @ResponseBody
            public R error(GuliException e){
                log.error(e.getMessage());
                e.printStackTrace();
                return R.error().code(e.getCode()).message(e.getMsg());
            }
        }
        
        ```

   3. 将日志堆栈信息输出到文件

      - 在common_utils中创建ExceptionUtil.java工具类
   
        ```java
        package com.guli.commonutils;
        
        import java.io.IOException;
        import java.io.PrintWriter;
        import java.io.StringWriter;
        
        public class ExceptionUtil {
                public static String getMessage(Exception e) {
                    StringWriter sw = null;
                    PrintWriter pw = null;
                    try {
                        sw = new StringWriter();
                        pw = new PrintWriter(sw);
                        // 将出错的栈信息输出到printWriter中
                        e.printStackTrace(pw);
                        pw.flush();
                        sw.flush();
                    } finally {
                        if (sw != null) {
                            try {
                                sw.close();
                            } catch (IOException e1) {
                                e1.printStackTrace();
                            }
                        }
                        if (pw != null) {
                            pw.close();
                        }
                    }
                    return sw.toString();
                }
        }
        
        ```
   
      - 调用
   
        ```java
        log.error(ExceptionUtil.getMessage(e));
        ```
   
        ```java
        package com.guli.servicebase.exceptionHandler;
        
        /**
         * 统一异常处理类
         */
        @ControllerAdvice
        @Slf4j
        public class GlobalExceptionHandler {
        
        
            //自定义异常
            @ExceptionHandler(GuliException.class)
            @ResponseBody
            public R error(GuliException e){
                
                log.error(ExceptionUtil.getMessage(e));
                
                e.printStackTrace();
                return R.error().code(e.getCode()).message(e.getMsg());
            }
        }
        
        ```
   
        
   
      

