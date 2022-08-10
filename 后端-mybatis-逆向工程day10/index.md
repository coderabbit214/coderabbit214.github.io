# 后端-Mybatis-逆向工程(day10)


# 后端-Mybatis-逆向工程(day10)

表、类、接口、mapper.xml四者密切相关，因此 当知道一个的时候  其他三个应该可以自动生成。

表->其他三个

实现步骤：

1. mybatis-generator-core.jar、mybatis.jar、ojdbc.jar

2.  逆向工程的配置文件generator.xml

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <!DOCTYPE generatorConfiguration
     PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
     "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
   <generatorConfiguration>
      <context id="DB2Tables" targetRuntime="MyBatis3">
      <commentGenerator>
      <!--
   			suppressAllComments属性值：
   				true:自动生成实体类、SQL映射文件时没有注释
   				true:自动生成实体类、SQL映射文件，并附有注释
   		  -->
     <property name="suppressAllComments" value="true" />
    </commentGenerator>
    
    
    <!-- 数据库连接信息 -->
     <jdbcConnection driverClass="com.mysql.cj.jdbc.Driver"
      connectionURL="jdbc:mysql://localhost:3306/testdata?useSSL=true&amp;serverTimezone=UTC"
      userId="root"  password="123456">
     </jdbcConnection>
     <!-- 
   			forceBigDecimals属性值： 
   				true:把数据表中的DECIMAL和NUMERIC类型，
   解析为JAVA代码中的java.math.BigDecimal类型 
   				false(默认):把数据表中的DECIMAL和NUMERIC类型，
   解析为解析为JAVA代码中的Integer类型 
   		-->
    <javaTypeResolver>
     	<property name="forceBigDecimals" value="false" />
    </javaTypeResolver>
    <!-- 
   		targetProject属性值:实体类的生成位置  
   		targetPackage属性值：实体类所在包的路径
   	-->
    <javaModelGenerator targetPackage="org.jsh.entity"
                               targetProject=".\src">
     <!-- trimStrings属性值：
   			true：对数据库的查询结果进行trim操作
   			false(默认)：不进行trim操作       
   		  -->
     <property name="trimStrings" value="true" />
    </javaModelGenerator>
    <!-- 
   		targetProject属性值:SQL映射文件的生成位置  
   		targetPackage属性值：SQL映射文件所在包的路径
   	-->
     <sqlMapGenerator targetPackage="org.jsh.mapper"
   			targetProject=".\src">
     </sqlMapGenerator>
     <!-- 生成动态代理的接口  -->
    <javaClientGenerator type="XMLMAPPER" targetPackage="org.jsh.mapper" targetProject=".\src">
    </javaClientGenerator>
    
    <!-- 指定数据库表  -->
     <table tableName="Student"> </table>
     <table tableName="studentCard"> </table>
     <table tableName="studentClass"> </table>
    </context>
   </generatorConfiguration>
   ```

   

3. 执行

   ```java
   package org.jsh.test;
   
   import org.mybatis.generator.api.MyBatisGenerator;
   import org.mybatis.generator.config.Configuration;
   import org.mybatis.generator.config.xml.ConfigurationParser;
   import org.mybatis.generator.exception.InvalidConfigurationException;
   import org.mybatis.generator.exception.XMLParserException;
   import org.mybatis.generator.internal.DefaultShellCallback;
   
   import java.io.File;
   import java.io.IOException;
   import java.sql.SQLException;
   import java.util.ArrayList;
   import java.util.List;
   
   public class Test {
       public static void main(String[] args) throws IOException, XMLParserException, InvalidConfigurationException, SQLException, InterruptedException {
   
           File file = new File("src/generator.xml") ;//配置文件
   
           List<String> warnings = new ArrayList<>();
           ConfigurationParser cp = new ConfigurationParser(warnings);
   
           Configuration config = cp.parseConfiguration(file);
   
   
           DefaultShellCallback callBack = new DefaultShellCallback(true);
   
           //逆向工程的核心类
           MyBatisGenerator generator = new MyBatisGenerator(config, callBack,warnings  );
           generator.generate(null);
       }
   }
   
   ```

   

