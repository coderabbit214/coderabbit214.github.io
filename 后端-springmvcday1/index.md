# 后端-SpringMVC(day1)


# 后端-SpringMVC(day1)

1. jar
   - commons-logging-1.1.1.jar
   - spring-aop-4.3.9.RELEASE.jar
   - spring-beans-4.3.9.RELEASE.jar
   - spring-context-4.3.9.RELEASE.jar
   - spring-core-4.3.9.RELEASE.jar
   - spring-expression-4.3.9.RELEASE.jar
   - spring-web-4.3.9.RELEASE.jar
   - spring-webmvc-4.3.9.RELEASE.jar

2. 配置

   - springmvc 配置 在src下

     > 给使用注解的包配置扫描器
     >
     > 配置视图解析器 前缀 和 后缀

     ```xml
     <?xml version="1.0" encoding="UTF-8"?>
     <!--suppress ALL -->
     <beans xmlns="http://www.springframework.org/schema/beans"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xmlns:aop="http://www.springframework.org/schema/aop"
            xmlns:context="http://www.springframework.org/schema/context"
            xmlns:mvc="http://www.springframework.org/schema/mvc"
            xsi:schemaLocation="http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-4.3.xsd
     	    http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
     		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.3.xsd
     		http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.3.xsd">
         
         
     <!--    扫描 使用注解需要-->
         <context:component-scan base-package="org.jsh.handler"></context:component-scan>
         
     <!--配置视图解析器-->
         <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
             <property name="prefix" value="/views/"></property>
             <property name="suffix" value=".jsp"></property>
         </bean>
     
     
     </beans>
     
     ```

   - web.xml 配置

     ```xml
         <servlet>
             <servlet-name>springDispatcherServelet</servlet-name>
             <!--springmvc 类-->
             <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
             <!--加载springmvc配置文件-->
             <init-param>
                 <param-name>contextConfigLocation</param-name>
                 <param-value>classpath:springmvc.xml</param-value>
             </init-param>
             <load-on-startup>1</load-on-startup>
         </servlet>
     
         <servlet-mapping>
             <servlet-name>springDispatcherServelet</servlet-name>
             <!--拦截所有-->
             <url-pattern>/</url-pattern>
         </servlet-mapping>
     
     ```

3. 编写

   - 第一个页面

     ```html
     <%@ page contentType="text/html;charset=UTF-8" language="java" %>
     <html>
       <head>
         <title>$Title$</title>
       </head>
       <body>
       $END$
       <a href="welcome">first springmvc</a>
       </body>
     </html>
     
     ```

   - 拦截

     > @Controller 使用注解 将普通类 变成控制器
     >
     > @RequestMapping("welcome") 在方法上加注解 拦截对应的请求
     >
     > return "succes"   配合springmvc 配置 拼接成 具体跳转目录

     ```java
     package org.jsh.handler;
     
     import org.springframework.stereotype.Controller;
     import org.springframework.web.bind.annotation.RequestMapping;
     
     @Controller
     public class SpringMVCHandler {
         @RequestMapping("welcome")
         public String welcome(){
             return "succes";  //  view/succes.jsp
         }
     }
     
     ```

   - 第二个页面 在 web/views/

     ```html
     <%@ page contentType="text/html;charset=UTF-8" language="java" %>
     <html>
     <head>
         <title>Title</title>
     </head>
     <body>
         <h1>welcome come111</h1>
     </body>
     </html>
     ```

     


