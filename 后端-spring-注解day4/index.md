# 后端-spring-注解(day4)


# 后端-spring-注解(day4)

1. 使用

   > java
>
   > @Component("studentDao") 相当于
   >
   > <bean id="studentDao" class="org.jsh.dao.studentDao">
   > 	</bean>
   >
   > 如果要给属性赋值  自动装配byType
   >
   > 加入@Autowired
   >
   > 如果要根据名字 byName
   >
   > 加入@Qualifier("属性id") 
   >
   > > 每一个属性都需要

   ```java
   package org.jsh.dao;
   
import org.jsh.entiy.Student;
   import org.springframework.stereotype.Component;
   
   @Component("studentDao")
   public class StudentDaoImpl {
    @Autowired
       @Qualifier("stuDao")
       private IStudentDao studentDao;
       
   	public void addStudent(Student student) {
   		System.out.println("...");
   	}
   }
   
   ```
   
   > applicationContext.xml
   >
   > context头文件
   
   ```xml
   <!-- 配置扫描器 -->
   	<context:component-scan base-package="org.jsh.dao"></context:component-scan>
   ```
   
   > @Component细化：
   >
   > dao层注解：@Repository
   > service层注解：@Service
   > 控制器层注解：@Controller

## 使用注解实现声明式事务

> 目标：通过事务 使以下方法 要么全成功、要么全失败

1. jar

   - spring-tx-4.3.9.RELEASE
   - 数据库jar
   - commons-dbcp.jar  连接池使用到数据源
   - commons-pool.jar  连接池
   - spring-jdbc-4.3.9.RELEASE.jar
   - aopalliance.jar 

2. 配置

   <!-- 增加对事务的支持 -->
   <tx:annotation-driven transaction-manager="txManager"  />

   ```xml
   <!-- 配置数据库相关  事务-->
   	<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource">
   		<property name="driverClassName" value="com.mysql.cj.jdbc.Driver"></property>
   		<property name="url" value="jdbc:mysql://localhost:3306/testdata"></property>
   		<property name="username" value="root"></property>
   		<property name="password" value="123456"></property>
   	</bean>
   	<!-- 配置事务管理器  "txManager"-->
   	<bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
   		<property name="dataSource" ref="dataSource"></property>
   	</bean>
   	<!-- 增加事务相关支持 -->
   	<tx:annotation-driven transaction-manager="txManager"/>
   ```

3. 使用

   > 将需要 成为事务的方法 前增加注解：
   > @Transactional(readOnly=false,propagation=Propagation.REQUIRED)

   ```java
   package org.jsh.service.impl;
   
   import org.jsh.dao.IStudentDao;
   import org.jsh.dao.impl.StudentDaoImpl;
   import org.jsh.entiy.Student;
   import org.jsh.service.IStudentService;
   import org.springframework.transaction.annotation.Propagation;
   import org.springframework.transaction.annotation.Transactional;
   
   public class StudentServiceImpl implements IStudentService {
   	
   	IStudentDao studentDao;
   	public void setStudentDao(IStudentDao studentDao) {
   		this.studentDao = studentDao;
   	}
   	
   	@Transactional(
   			readOnly=false,
   			propagation=Propagation.REQUIRED,
   			rollbackForClassName={"SQLException","ArithmeticException"})
   	public void addStudent(Student student) {
   		studentDao.addStudent(student);
   	}
   
   }
   
   ```

   

Transactional属性 


