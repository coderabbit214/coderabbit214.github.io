# 后端-Spring-AOP，expression表达式(day5)


# 后端-Spring-AOP，expression表达式(day5)

## AOP

面向方面编程

## 前置通知

> 实现在执行一个方法前自动执行某个方法

1. jar

   - aopaliance.jar
   - aspectjweaver.jar

2. 配置 MethodBeforeAdvice

   LogBefore.java

   ```java
   package org.jsh.aop;
   
   import java.lang.reflect.Method;
   
   import org.springframework.aop.MethodBeforeAdvice;
   
   //普通类 -> 前置通知 implements MethodBeforeAdvice
   public class LogBefore implements MethodBeforeAdvice{
   	//前置通知的具体内容
   	@Override
   	public void before(Method method, Object[] args, Object target) throws Throwable {
   		// TODO Auto-generated method stub
   		System.out.println("前置通知");
   	}
   }
   
   ```

   applicationContext.xml

   ```xml
   <!-- 配置前置通知 -->
   	<!-- add方法所在类 -->
   	<bean id="studentService" class="org.jsh.service.impl.StudentServiceImpl">
   		<property name="studentDao" ref="studentDao"></property>
   	</bean>
   	<!-- 通知所在类 -->
   	<bean id="logBefore" class="org.jsh.aop.LogBefore">
   	</bean>
   	<!-- 将方法所在类 和 通知 进行关联 -->
   	<aop:config proxy-target-class="true">
   		<!-- 配置切入点 -->
   		<aop:pointcut expression="execution(public void org.jsh.service.impl.StudentServiceImpl.deleteStudent(org.jsh.entiy.Student)) or execution(public void org.jsh.service.impl.StudentServiceImpl.addStudent(org.jsh.entiy.Student))" id="poioncut"/>
   		<!-- 关联:advisor相当于链接切入点和切面的线 -->
   		<aop:advisor advice-ref="logBefore" pointcut-ref="poioncut"/>
   	</aop:config>
   ```

3. 使用

   ```java
   public static void testAOP() {
   		ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
   		StudentServiceImpl studentService = (StudentServiceImpl)context.getBean("studentService");
   		Student student = (Student) context.getBean("student");
   		studentService.addStudent(student);
   		studentService.deleteStudent(student);
   	}
   ```

## 后置通知

通知类 java  AfterReturningAdvice

> 注意参数

```java
package org.jsh.aop;

import java.lang.reflect.Method;

import org.springframework.aop.AfterReturningAdvice;

public class LogAfter implements AfterReturningAdvice {

	@Override
	public void afterReturning(Object arg0, Method arg1, Object[] arg2, Object arg3) throws Throwable {
		// TODO Auto-generated method stub
		System.out.println("后置通知。。。目标对象："+arg3+"目标方法"+arg1+"参数个数"+arg2.length+"方法的返回值"+arg0);
	}
}
```

## 异常通知

> 需要实现 ThrowsAdvice 接口

```java
package org.jsh.aop;

import java.lang.reflect.Method;

import org.springframework.aop.ThrowsAdvice;

public class LogException implements ThrowsAdvice {
	public void afterThrowing(Method method,Object[] args,Object target,Throwable ThrowableSubclass) {
		System.out.println("异常通知");
	}
}

```

## 环绕通知 实现了前三个通知的功能

> MethodInterceptor 接口
>
> 注意参数

```java
package org.jsh.aop;

import org.aopalliance.intercept.MethodInterceptor;
import org.aopalliance.intercept.MethodInvocation;

public class LogAround implements MethodInterceptor {

	@Override
	public Object invoke(MethodInvocation arg0) throws Throwable {
		Object result = null;
		//		方法体一
		try {
//			方法体二
//			前置通知
			System.out.println("环绕前置");
		 	result = arg0.proceed();//控制目标方法的执行 相当于 addStudent()
//			后置通知
			System.out.println("目标对象："+arg0.getThis()+"目标方法"+arg0.getMethod()+"参数个数"+arg0.getArguments().length+"方法的返回值"+result);

		 	System.out.println("环绕后置");
		}catch (Exception e) {
//			异常通知
			System.out.println("环绕异常");
		}
//		目标方法的返回值
		return result;
	}

}

```

applicationContext.xml



```xml

	<!-- 通知所在类 -->
	<bean id="logBefore" class="org.jsh.aop.LogBefore">
	</bean>
	<bean id="logAfter" class="org.jsh.aop.LogAfter">
	</bean>
	<bean id="logException" class="org.jsh.aop.LogException">
	</bean>
	<bean id="logAround" class="org.jsh.aop.LogAround">
	</bean>

	<!-- 将方法所在类 和 通知 进行关联 -->
	<aop:config proxy-target-class="true">
		<!-- 配置切入点 -->
		<aop:pointcut expression="execution(public void org.jsh.service.impl.StudentServiceImpl.deleteStudent(org.jsh.entiy.Student)) or execution(public void org.jsh.service.impl.StudentServiceImpl.addStudent(org.jsh.entiy.Student))" id="poioncut"/>
		<!-- 关联:advisor相当于链接切入点和切面的线 -->
		<aop:advisor advice-ref="logBefore" pointcut-ref="poioncut"/>
	</aop:config>
	<!-- 后置 -->
	<aop:config proxy-target-class="true">
		<!-- 配置切入点 -->
		<aop:pointcut expression="execution(public void org.jsh.service.impl.StudentServiceImpl.addStudent(org.jsh.entiy.Student))" id="poioncut2"/>
		<!-- 关联:advisor相当于链接切入点和切面的线 -->
		<aop:advisor advice-ref="logAfter" pointcut-ref="poioncut2"/>
	</aop:config>
	<!-- 异常 -->
	<aop:config proxy-target-class="true">
		<!-- 配置切入点 -->
		<aop:pointcut expression="execution(public void org.jsh.service.impl.StudentServiceImpl.addStudent(org.jsh.entiy.Student))" id="poioncut3"/>
		<!-- 关联:advisor相当于链接切入点和切面的线 -->
		<aop:advisor advice-ref="logException" pointcut-ref="poioncut3"/>
	</aop:config>
	<!-- 环绕 -->
	<aop:config proxy-target-class="true">
		<!-- 配置切入点 -->
		<aop:pointcut expression="execution(public void org.jsh.service.impl.StudentServiceImpl.addStudent(org.jsh.entiy.Student))" id="poioncut4"/>
		<!-- 关联:advisor相当于链接切入点和切面的线 -->
		<aop:advisor advice-ref="logAround" pointcut-ref="poioncut4"/>
	</aop:config>
```



## expression

expression=*"execution(…)"* 

| 举例                                                         | 含义                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| public  boolean addStudent(org.jsh.entity.Student))          | 所有返回类型为boolean、参数类型为org.jsh.entity.Student的addStudent()方法。 |
| public  boolean org.jsh.service.IStudentService.  addStudent(org.jsh.entity.Student) | org.jsh.service.IStudentService类（或接口）中的addStudent()方法，并且返回类型是boolean、参数类型是org.jsh.entity.Student |
| public  * addStudent(org.jsh.entity.Student)                 | “*”代表任意返回类型                                          |
| public  void *( org.jsh.entity.Student)                      | “*”代表任意方法名                                            |
| public  void addStudent(..)                                  | “..”代表任意参数列表                                         |
| *  org.jsh.service.*.*(..)                                   | org.jsh.service.IStudentService包中，包含的所有方法（不包含子包中的方法） |
| * org.jsh.service..*.*(..)                                   | org.jsh.service.IStudentService包中，包含的所有方法（包含子包中的方法） |


