# 后端-springweb


# 后端-springweb

## Web项目如何初始化SpringIOC容器 ：

思路：当服务启动时（tomcat），通过监听器将SpringIOC容器初始化一次（该监听器 spring-web.jar已经提供）
因此用spring开发web项目 至少需要7个jar： 

spring-java的6个jar + spring-web.jar，

注意：web项目的jar包 是存入到WEB-INF/lib中

方法一：

web.xml

```xml
<listener>
  <!-- 配置spring-web.jar提供的监听器，此监听器可以在服务器启动时 初始化IOC容器 
  		初始化IOC容器（applicationContext.xml）,需要指定位置context-param
  -->
  	<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
  </listener>
  <!-- 指定IOC容器的位置 -->
  <context-param>
  <!-- 监听器父类中 有一个属性contextConfigLocation 需要给赋值告诉配置文件位置 -->
  	<param-name>contextConfigLocation</param-name>
  	<param-value>classpath:applicationContext.xml</param-value>
  </context-param>
```

方法二：约定 放入web-inf中 不用指定位置 且名字只能为applicationContext.xml

## 配置文件拆分

方法一 value 中写多个值

web.xml 

```xml
<context-param>
  <!-- 监听器父类中 有一个属性contextConfigLocation 需要给赋值告诉配置文件位置 -->
  	<param-name>contextConfigLocation</param-name>
  	<param-value>classpath:applicationContext.xml,
  				 classpath:applicationContext-*.xml
  	</param-value>
  </context-param>
```

方法二 在总的applicationContext.xml中引入

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
	<import resource="classpath:applicationContext-*.xml"/>

</beans>
```



## 三层使用

> 将ioc容器与web容器打通 在init初始化时
>
> ApplicationContext context = WebApplicationContextUtils.getWebApplicationContext(this.getServletContext());

```java
package org.jsh.servlet;

import java.io.IOException;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.jsh.service.IStudentService;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.springframework.web.context.support.WebApplicationContextUtils;

/**
 * Servlet implementation class QueryStudentByIdServlet
 */
public class QueryStudentByIdServlet extends HttpServlet {
	private static final long serialVersionUID = 1L;
	IStudentService studentService;
    public void setStudentService(IStudentService studentService) {
		this.studentService = studentService;
	}
    
    @Override
    public void init() throws ServletException {
   /* 	ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext-Service.xml");*/
    	ApplicationContext context = WebApplicationContextUtils.getWebApplicationContext(this.getServletContext());
    	studentService = (IStudentService)context.getBean("studentService");
    }

	protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		String name = studentService.queryStudentById();
		request.setAttribute("name", name);
		request.getRequestDispatcher("result.jsp").forward(request, response);
	}

	
	protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		// TODO Auto-generated method stub
		doGet(request, response);
	}

}

```


