# 后端-Spring(day1)


# 后端-Spring(day1)

## 搭建Spring环境

1. 开发spring至少需要使用的jar(5个+1个):

- spring-aop.jar		开发AOP特性时需要的JAR
- spring-beans.jar	处理Bean的jar	
- spring-context.jar	处理spring上下文的jar		
- spring-core.jar		spring核心jar
- spring-expression.jar	spring表达式 
- 三方提供的日志jar      commons-logging.jar	日志

2. 编写配置文件

为了编写时有一些提示、自动生成一些配置信息：
方式一：增加sts插件
可以给eclipse增加 支持spring的插件：spring tool suite(https://spring.io/tools/sts/all)
下载springsource-tool-suite-3.9.4.RELEASE-e4.7.3a-updatesite.zip,然后在Eclipse中安装：Help-Install new SoftWare.. - Add

方式二：
	直接下载sts工具（相当于一个集合了Spring tool suite的Eclipse）: https://spring.io/tools/sts/

新建：bean configuration .. - applicationContext.xml文件

## 开发Spring程序(IOC)

1. applicationContext.xml

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
   	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
   	<!-- 该文件中产生的所有对象，被spring放入了一个称之为spring ioc容器里 -->
   	<!-- id 唯一标识符 calss 实体类全类名 -->
   	<!-- name 属性值 value 值 -->
   	<bean id="student" class="org.jsh.entiy.Student">
   		<property name="stuNo" value="1"></property>
   		<property name="stuName" value="zs"></property>
   		<property name="stuAge" value="12"></property>
   	</bean>
   
   </beans>
   
   ```

2. 新建实体类

   ```java
   package org.jsh.entiy;
   
   public class Student {
   	private int stuNo;
   	private String stuName;
   	public int getStuNo() {
   		return stuNo;
   	}
   
   	public void setStuNo(int stuNo) {
   		this.stuNo = stuNo;
   	}
   
   	public String getStuName() {
   		return stuName;
   	}
   
   	public void setStuName(String stuName) {
   		this.stuName = stuName;
   	}
   
   	public int getStuAge() {
   		return stuAge;
   	}
   
   	public void setStuAge(int stuAge) {
   		this.stuAge = stuAge;
   	}
   
   	private int stuAge;
   	
   	@Override
   	public String toString() {
   		return super.toString();
   	}
   }
   
   ```

3. 使用

   ```java
   package org.jsh.test;
   
   import org.jsh.entiy.Student;
   import org.springframework.context.ApplicationContext;
   import org.springframework.context.support.ClassPathXmlApplicationContext;
   
   public class Test {
   	public static void main(String[] args) {
   		ApplicationContext conext = new ClassPathXmlApplicationContext("applicationContext.xml");
   		Student student = (Student) conext.getBean("student");
   		System.out.println(student.getStuAge());
   	}
   }
   
   ```

> ioc 控制反转/依赖注入
>
> IOC（控制反转）也可以称之为DI（依赖注入）：
> 控制反转：将 创建对象、属性值 的方式 进行了翻转，从new、setXxx()  翻转为了 从springIOC容器getBean()
>
> 依赖注入：将属性值 注入给了属性，将属性 注入给了bean，将bean注入给了ioc容器；
>
> 总结：ioc/di ，无论要什么对象，都可以直接去springioc容器中获取，而不需要自己操作（new\setXxx()）

## IOC超级工厂理解

1. 创建java类

   - ICourse接口

     ```java
     package org.jsh.newinstance;
     
     public interface ICourse {
     	void learn();
     }
     
     ```

   - 实现类 HtmlCourse.java 和 JavaCourse

     ```java
     package org.jsh.newinstance;
     
     public class HtmlCourse implements ICourse{
     	 @Override
     	 public void learn() {
     	 	System.out.println("html");
     	 }
     
     }
     
     ```

     ```java
     package org.jsh.newinstance;
     
     public class JavaCourse implements ICourse{
      @Override
     public void learn() {
     	System.out.println("java");
     	
     }
     }
     ```

2. 加入applicationContext.xml中

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
   	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
   	<!-- 该文件中产生的所有对象，被spring放入了一个称之为spring ioc容器里 -->
   	<!-- id 唯一标识符 calss 实体类全类名 -->
   	<!-- name 属性值 value 值 -->
   	<bean id="student" class="org.jsh.entiy.Student">
   		<property name="stuNo" value="1"></property>
   		<property name="stuName" value="zs"></property>
   		<property name="stuAge" value="12"></property>
   	</bean>
   	
   	<bean id="javaCourse" class="org.jsh.newinstance.JavaCourse">
   
   	</bean>
   	<bean id="htmlCourse" class="org.jsh.newinstance.HtmlCourse">
   
   	</bean>
   </beans>
   
   ```

3. 在Student类中加入学习方法

   ```java
   	public void learn(String name) {
   		ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
   		ICourse course = (ICourse)context.getBean(name);
   		course.learn();
   	}
   ```

4. 使用

   ```java
   	public static void learnCourseWithFactory() {
   		ApplicationContext conext = new ClassPathXmlApplicationContext("applicationContext.xml");
   		Student student = (Student) conext.getBean("student");
   		student.learn("htmlCourse");
   	}
   ```

   


