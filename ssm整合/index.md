# SSM整合


# SSM整合

1. jar

   commons-dbcp-1.4.jar

   commons-logging-1.1.1.jar

   commons-pool-1.6.jar

   log4j-1.2.17.jar

   mybatis-3.4.6.jar

   mybatis-spring-1.3.1.jar

   mysql-connector-java-8.0.15.jar

   spring-aop-4.3.9.RELEASE.jar

   spring-beans-4.3.9.RELEASE.jar

   spring-context-4.3.9.RELEASE.jar

   spring-core-4.3.9.RELEASE.jar

   spring-expression-4.3.9.RELEASE.jar

   spring-jdbc-4.3.9.RELEASE.jar

   spring-tx-4.3.9.RELEASE.jar

   spring-web-4.3.9.RELEASE.jar

   spring-webmvc-4.3.9.RELEASE.jar

2. spring 和 mabits整合

   > 数据源，mapper.xml

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
   	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
   <!-- 依赖注入 -->
   	<bean id="studentService" class="org.jsh.service.impl.StudentServiceImpl">
   		<property name="studentMapper" ref="studentMapper"></property>
   	</bean>
   <!-- 	<bean id="studentController" class="org.jsh.controller.StudentController">
   		<property name="studentService" ref="studentService"></property>
   	</bean> -->
   
   <!-- 加载db.properties文件 -->
   	<bean  id="config" class="org.springframework.beans.factory.config.PreferencesPlaceholderConfigurer">
   		<property name="locations">
   			<array>
   				<value>classpath:db.properties</value>
   			</array>
   		</property>
   	</bean>
   	
   <!-- 数据源，mapper.xml  -->
   	<!-- 配置配置数据库信息（替代mybatis的配置文件conf.xml） -->
   	<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource">
   			<property name="driverClassName" value="${driver}"></property>
   			<property name="url" value="${url}"></property>
   			<property name="username" value="${username}"></property>
   			<property name="password" value="${password}"></property>
   	</bean>
   	
   	<!-- 在SpringIoc容器中 创建MyBatis的核心类 SqlSesionFactory -->
   	<bean id="sqlSessionFacotry" class="org.mybatis.spring.SqlSessionFactoryBean">
   		<property name="dataSource" ref="dataSource"></property>
   		<!-- 加载mapper.xml路径 -->
   		<property name="mapperLocations" value="classpath:org/jsh/mapper/*.xml"></property>
   	</bean>
   	
   	
   <!-- Spring整合Mybatis -->
   	 <!-- 第三种方式生成mapper对象(批量产生多个mapper)
   	 	批量产生Mapper对在SpringIOC中的 id值 默认就是  首字母小写接口名 (首字母小写的接口名=id值)
   	 	  -->
   	 <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
   	 	<property name="sqlSessionFactoryBeanName" value="sqlSessionFacotry"></property>
   	 	 	 <!--指定批量产生 哪个包中的mapper对象-->
   	 	 	<property name="basePackage" value="org.jsh.mapper"></property>
   	 </bean>
   </beans>
   
   ```

3. 配置springmvc

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
   	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   	xmlns:mvc="http://www.springframework.org/schema/mvc"
   	xmlns:context="http://www.springframework.org/schema/context"
   	xsi:schemaLocation="http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-4.3.xsd
   		http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
   		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.3.xsd">
   <!--配置视图解析器-->
       <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
           <property name="prefix" value="/views/"></property>
           <property name="suffix" value=".jsp"></property>
       </bean>
   <!--    此配置是SpringMVC的基础配置，很多功能都需要通过该注解来协调  -->
       <mvc:annotation-driven></mvc:annotation-driven>
       <!--    扫描-->
       <context:component-scan base-package="org.jsh.controller"></context:component-scan>
   </beans>
   
   ```

## 代码

### 后端

1. entity

   ```java
   package org.jsh.entity;
   
   public class Student {
   	private int stunum;
   	private String name;
   	private int age ;
   	public int getStunum() {
   		return stunum;
   	}
   	public void setStunum(int stunum) {
   		this.stunum = stunum;
   	}
   	public String getName() {
   		return name;
   	}
   	public void setName(String name) {
   		this.name = name;
   	}
   	public int getAge() {
   		return age;
   	}
   	public void setAge(int age) {
   		this.age = age;
   	}
   	
   }
   
   ```

   

2. dao

   ```xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <!DOCTYPE mapper
   PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
   "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
   
   <!-- namespace:该mapper.xml映射文件的 唯一标识 -->
   <mapper namespace="org.jsh.mapper.StudentMapper">
   	
   	<select id="queryStudentByStuno" 	parameterType="int"  	resultType="org.jsh.entity.Student"  >
   		select * from student where stunum = #{stuNo}
   	</select>
   	
   	
   	<insert id="addStudent" parameterType="org.jsh.entity.Student" >
   		insert into student(stunum,name,age) values(#{stunum},#{name},#{age})
   	</insert>
   	
   </mapper>
   ```

   ```java
   package org.jsh.mapper;
   
   import org.jsh.entity.Student;
   
   public interface StudentMapper {
   	public void addStudent(Student student	);
   	public Student queryStudentByStuno(int stunum);
   	
   }
   
   ```

3. service

   ```java
   package org.jsh.service.impl;
   
   import org.jsh.entity.Student;
   import org.jsh.mapper.StudentMapper;
   import org.jsh.service.StudentService;
   
   public class StudentServiceImpl implements StudentService {
   	private StudentMapper studentMapper;
   	public void setStudentMapper(StudentMapper studentMapper) {
   		this.studentMapper = studentMapper;
   	}
   	public Student queryStudentByNum(int stunum) {
   		Student student =  studentMapper.queryStudentByStuno(stunum);
   		System.out.println(student);
   		return student;
   	}
   
   }
   
   ```

4. controller

   ```java
   package org.jsh.controller;
   
   import java.util.Map;
   
   import org.jsh.entity.Student;
   import org.jsh.service.impl.StudentServiceImpl;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.stereotype.Controller;
   import org.springframework.web.bind.annotation.PathVariable;
   import org.springframework.web.bind.annotation.RequestMapping;
   
   @Controller
   @RequestMapping("studentController")
   public class StudentController {
   	@Autowired
   	StudentServiceImpl studentService;
   	public void setStudentService(StudentServiceImpl studentService) {
   		this.studentService = studentService;
   	}
   	@RequestMapping("queryStudentByNum/{stunum}")
   	public String queryStudentByNum(@PathVariable("stunum") int stunum,Map<String,Object> map) {
   		Student student = studentService.queryStudentByNum(stunum);
   		map.put("student",student);
   		return "result";
   	}
   }
   
   ```

### 前端

web.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://java.sun.com/xml/ns/javaee" xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd" id="WebApp_ID" version="2.5">
  <display-name>SSMProject</display-name>
  <welcome-file-list>
    <welcome-file>index.html</welcome-file>
    <welcome-file>index.htm</welcome-file>
    <welcome-file>index.jsp</welcome-file>
    <welcome-file>default.html</welcome-file>
    <welcome-file>default.htm</welcome-file>
    <welcome-file>default.jsp</welcome-file>
  </welcome-file-list>
  
  <!-- web项目中引入Spring -->
  <!-- needed for ContextLoaderListener -->
	<context-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>classpath:applicationContext.xml</param-value>
	</context-param>

	<!-- Bootstraps the root web application context before servlet initialization -->
	<listener>
		<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
	</listener>
	
	<!-- 配置SpringMVC -->
	<!-- The front controller of this Spring Web application, responsible for handling all application requests -->
	<servlet>
		<servlet-name>springDispatcherServlet</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		<init-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>classpath:applicationContext-controller.xml</param-value>
		</init-param>
		<load-on-startup>1</load-on-startup>
	</servlet>

	<!-- Map all requests to the DispatcherServlet for handling -->
	<servlet-mapping>
		<servlet-name>springDispatcherServlet</servlet-name>
		<url-pattern>/</url-pattern>
	</servlet-mapping>
  
</web-app>
```

index.jsp

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
   %>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Insert title here</title>
</head>
<body>
	<a href="studentController/queryStudentByNum/1">查询一号学生</a>
</body>
</html>
```

result.jsp

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="ISO-8859-1"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Insert title here</title>
</head>
<body>
	${requestScope.student.stunum}+${requestScope.student.name}+${requestScope.student.age}
</body>
</html>
```


