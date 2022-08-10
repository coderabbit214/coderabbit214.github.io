# 后端-SpringMVC(day6)


# 后端-SpringMVC(day6)

## 国际化(视图解析器)

1. 创建资源文件

   > 基名 _ 语言 _ 地区.properties
   > 基名_语言.properties
   >
   > > 数据以k v对形式存在

   ```
   //i18n_zh_CN.properties
   resource.welcome=\u4F60\u597D
   resource.exist=\u9000\u51FA
   
   //i18n_en_US.properties
   resource.welcome=WELCOME
   resource.exist=EXIST
   ```

2. 配置springmvc.xml，加载资源文件

   ```xml
   <!--加载国际化配置文件
       1.加入SpringMVC id=""
       2.配置ResourceBundleMessageSource
   -->
       <bean id="messageSource" class="org.springframework.context.support.ResourceBundleMessageSource">
           <property name="basename" value="i18n"></property>
       </bean>
   ```

3. 通过jstl使用国际化 jstl.jar  standar.jar

   ```jsp
   <%--
     Created by IntelliJ IDEA.
     User: Mr.J
     Date: 2020/4/7
     Time: 18:27
     To change this template use File | Settings | File Templates.
   --%>
   <%@ page contentType="text/html;charset=UTF-8" language="java" isErrorPage="true" %>
   <%@ taglib uri="http://java.sun.com/jsp/jstl/fmt" prefix="fmt" %>
   <html>
   <head>
       <title>Title</title>
   </head>
   <body>
       <fmt:message key="resource.welcome"></fmt:message>
   
       <h1>welcome come111</h1>
       ${requestScope.student.name}
       ${requestScope.student.id}
   </body>
   </html>
   
   ```

   

