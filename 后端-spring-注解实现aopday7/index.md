# 后端-Spring-注解实现AOP(day7)


# 后端-Spring-注解实现AOP(day7)

1. jar 与实现接口的方式相同

2. 配置

   - 将 业务类，通知 纳入springIOC容器中

   - 开启注解对AOP的支持

     ```xml
     <aop:aspectj-autoproxy></aop:aspectj-autoproxy>
     ```

   - execution 连接通知类和业务类

3. 编写

   通知类

   > @Component("logAnnotation") 加入IOC容器中(不要忘记开启扫描)
   >
   > @Aspect 声明是一个通知
   >
   > @Before() 前置通知

   ```java
   package org.jsh.aop;
   
   import org.aspectj.lang.JoinPoint;
   import org.aspectj.lang.ProceedingJoinPoint;
   import org.aspectj.lang.annotation.After;
   import org.aspectj.lang.annotation.AfterReturning;
   import org.aspectj.lang.annotation.AfterThrowing;
   import org.aspectj.lang.annotation.Around;
   import org.aspectj.lang.annotation.Aspect;
   import org.aspectj.lang.annotation.Before;
   import org.springframework.stereotype.Component;
   
   @Component("logAnnotation")
   @Aspect
   public class LogAnnotation {
   //	前置
   	@Before("execution(public void org.jsh.service.impl.StudentServiceImpl.addStudent(org.jsh.entiy.Student))")//定义切点
   	public void myBefore(JoinPoint jp) {
   		System.out.println("。。。。。注解形式前置:目标对象"+jp.getTarget()+",方法名："+jp.getSignature().getName()+",参数列表"+jp.getArgs().length);
   	}
   //	后置
   	@AfterReturning(pointcut ="execution(public void org.jsh.service.impl.StudentServiceImpl.addStudent(org.jsh.entiy.Student))",returning="returningValue")
   	public void myAfter(JoinPoint jp,Object returningValue) {
   		System.out.println("。。。。。注解形式后置"+returningValue);
   	}
   //	环绕
   	@Around("execution(public void org.jsh.service.impl.StudentServiceImpl.addStudent(org.jsh.entiy.Student))")
   	public void myAround(ProceedingJoinPoint jp) {
   //		前置
   		System.out.println("<前置");
   		try {
   //		方法执行
   		Object result = jp.proceed();
   //		方法执行之后
   		System.out.println("<后置");
   			
   		}catch (Throwable e) {
   			// TODO: handle exception
   			System.out.println("<异常");
   		}finally {
   			System.out.println("<最终");
   		}
   	}
   //	异常通知 如果只捕获特定类型的异常 可以通过第二个参数实现
   	@AfterThrowing(pointcut="execution(public void org.jsh.service.impl.StudentServiceImpl.addStudent(org.jsh.entiy.Student))",throwing="e")
   	public void myException(JoinPoint jp,NullPointerException e) {
   		System.out.println("。。。。。注解形式异常");
   	}
   //	最终
   	@After("execution(public void org.jsh.service.impl.StudentServiceImpl.addStudent(org.jsh.entiy.Student))")
   	public void myAfter() {
   		System.out.println("最终");
   	}
   }
   
   ```

4. 参数问题

   - 参数：（JoinPoint jp）

     目标对象：jp.getTarget()

     方法名：jp.getSignature().getName()

     参数列表：jp.getArgs().length

   - 返回值： 在注解中配置 returning="returningValue"

      				参数 Object returningValue

   - 异常通知 如果只捕获特定类型的异常 可以通过第二个参数实现

     ​				 在注解中配置 throwing="e"

      				参数 NullPointerException(异常类型) e

# Schema配置

1. application.xml

   ```xml
   
   	<!-- 将准备转为 通知的类 纳入ioc容器 -->
   	<bean id="logSchema" class="org.lanqiao.aop.LogSchema"></bean>
   	<aop:config>
   		<!-- 切入点（连接线的一端：业务类的具体方法） -->
   		<aop:pointcut expression="execution(public * org.lanqiao.service.impl.StudentServiceImpl.addStudent(..))"   id="pcShema"/>
   		
   		
   		<!-- （连接线的另一端：通知 类）
   		<aop:advisor advice-ref="logSchea"  pointcut-ref="pcShema" />
   		 -->
   		 <!-- schema方式 -->
   		 <aop:aspect ref="logSchema">
   		  	<!-- 连接线：连接 业务 addStudent  和  通知before -->
   		 	<aop:before method="before" pointcut-ref="pcShema"/>
   		 	<!-- 连接线：连接 业务 addStudent  和  通知afterReturning -->
   		 	<aop:after-returning method="afterReturning" returning="returnValue" pointcut-ref="pcShema"/>
   		 	
   		 	<aop:after-throwing method="whenException" pointcut-ref="pcShema" throwing="e"/>
   		 	<!-- 环绕 -->
   		 	<aop:around method="around" pointcut-ref="pcShema" />
   		
   		 	
   		 </aop:aspect>
   		
   		
   		
   		
   	</aop:config>
   ```

2. 通知类

   ```java
   package org.lanqiao.aop;
   
   import java.lang.reflect.Method;
   
   import org.aspectj.lang.JoinPoint;
   import org.aspectj.lang.ProceedingJoinPoint;
   
   public class LogSchema {
   	
   	//后置通知方法  :JoinPoint适用于注解
   	public void afterReturning(JoinPoint jp,Object returnValue) throws Throwable {
   		System.out.println("》》》》》》》》》》》后置通知：目标对象："+jp.getThis()+",调用的方法名："+jp.getSignature().getName()+",方法的参数个数："+jp.getArgs().length+"，方法的返回值："+returnValue);
   	}
   	
   	public void before() {
   		System.out.println("》》》》》》》》》》》前置通知...");
   	}
   	
   	public void whenException(JoinPoint jp,NullPointerException e) {
   		System.out.println(">>>>>>>>>>>>>>>>异常：" +e.getMessage());
   	}
   	//注意：环绕通知 会返回目标方法的返回值，因此返回值为Object
   	public Object around(ProceedingJoinPoint jp)    {
   		System.out.println("''''''''''''''''''环绕通知：前置通知");
   		Object result = null ; 
   		try {
   			 result = jp.proceed() ;//执行方法
   			 System.out.println("'''''''''"+jp.getSignature().getName()+","+result);
   			System.out.println("''''''''''''''''''环绕通知：后置通知");
   		}catch(Throwable e) {
   			System.out.println("''''''''''''''''''环绕通知：异常通知");
   		}
   		return result ;
   	}
   
   }
   
   ```

   

