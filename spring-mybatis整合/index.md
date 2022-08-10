# Spring-Mybatis整合


# Spring-Mybatis整合

> 思路：
> 	SqlSessionFactory -> SqlSession ->StudentMapper ->CRUD
>
> 可以发现 ，MyBatis最终是通过SqlSessionFactory来操作数据库，
> Spring整合MyBatis 其实就是 将MyBatis的SqlSessionFactory 交给Spring

1. jar

   - mybatis-spring.jar	
   - spring-tx.jar	
   - spring-jdbc.jar		
   - spring-expression.jar
   - spring-context-support.jar	
   - spring-core.jar		
   - spring-context.jar
   - spring-beans.jar	
   - spring-aop.jar	
   - spring-web.jar	
   - commons-logging.jar
   - commons-dbcp.jar	
   - mysql-connector.jar	
   - mybatis.jar	
   - log4j.jar	
   - commons-pool.jar

2. 建立 三层架构 表与实体类

3. 通过mapper.xml将 类、表建立映射关系

4. 配置文件 

   > spring applicationContext.xml
   >
   > mybatis conf.xml(包含数据库信息 和加载mapper)合并到spring配置文件中

   applicationContext.xml

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
   	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
   	<!-- 加载db.properties文件 -->
   	<bean  id="config" class="org.springframework.beans.factory.config.PreferencesPlaceholderConfigurer">
   		<property name="locations">
   			<array>
   				<value>classpath:db.properties</value>
   			</array>
   		</property>	
   	</bean>
   	
   	<!--  第一种方式生成mapper对象
   	<bean id="studentMapper" class="org.lanqiao.dao.impl.StudentDaoImpl">
   	将SPring配置的sqlSessionFactory 对象交给mapper(dao) 
   		<property name="sqlSessionFactory" ref="sqlSessionFacotry"></property>
   	</bean>
   	 -->
   	 <!-- 第二种方式生成mapper对象 
   	 <bean id="studentMapper" class="org.mybatis.spring.mapper.MapperFactoryBean">
   	 	<property name="mapperInterface" value="org.lanqiao.mapper.StudentMapper"></property>
   	 	<property name="sqlSessionFactory" ref="sqlSessionFacotry"></property>
   	 </bean>
   	
   	   -->
   	  
   	 <!-- 第三种方式生成mapper对象(批量产生多个mapper)
   	 	批量产生Mapper对在SpringIOC中的 id值 默认就是  首字母小写接口名 (首字母小写的接口名=id值)
   	 	  -->
   	 <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
   	 	<property name="sqlSessionFactoryBeanName" value="sqlSessionFacotry"></property>
   	 	 	 <!--指定批量产生 哪个包中的mapper对象
   	 	 	 	  -->
   	 	 	<property name="basePackage" value="org.lanqiao.mapper"></property>
   	 </bean>
   
   	 
   	
   	<bean id="studentService" class="org.lanqiao.service.impl.StudentServiceImpl">
   		<property name="studentMapper" ref="studentMapper"></property>
   	</bean>
   	
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
   		<!-- 加载mybatis配置文件 
   			<property name="configLocation" value="classpath:conf.xml"></property>
   		-->
   		<!-- 加载mapper.xml路径 -->
   		<property name="mapperLocations" value="org/lanqiao/mapper/*.xml"></property>
   		
   		
   	</bean>
   
   </beans>
   
   ```

5. dao层实体类 在spring中实现的方式

   a.第一种方式
   	DAO层实现类  继承 SqlSessionDaoSupport类		
   	SqlSessionDaoSupport类提供了一个属性 SqlSession
   
b.第二种方式
   	就是省略掉 第一种方式的 实现类
   	直接MyBatis提供的 Mapper实现类：	org.mybatis.spring.mapper.MapperFactoryBean 
   	缺点：每个mapper都需要一个配置一次
   
c.第三种方式 批量配置 实现类
   
   ```xml
<!--  第一种方式生成mapper对象
   	<bean id="studentMapper" class="org.lanqiao.dao.impl.StudentDaoImpl">
   	将SPring配置的sqlSessionFactory 对象交给mapper(dao) 
   		<property name="sqlSessionFactory" ref="sqlSessionFacotry"></property>
   	</bean>
   	 -->
   	 <!-- 第二种方式生成mapper对象 
   	 <bean id="studentMapper" class="org.mybatis.spring.mapper.MapperFactoryBean">
   	 	<property name="mapperInterface" value="org.lanqiao.mapper.StudentMapper"></property>
   	 	<property name="sqlSessionFactory" ref="sqlSessionFacotry"></property>
   	 </bean>
   	
   	   -->
   	  
   	 <!-- 第三种方式生成mapper对象(批量产生多个mapper)
   	 	批量产生Mapper对在SpringIOC中的 id值 默认就是  首字母小写接口名 (首字母小写的接口名=id值)
   	 	  -->
   	 <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
   	 	<property name="sqlSessionFactoryBeanName" value="sqlSessionFacotry"></property>
   	 	 	 <!--指定批量产生 哪个包中的mapper对象
   	 	 	 	  -->
   	 	 	<property name="basePackage" value="org.lanqiao.mapper"></property>
   	 </bean>
   ```
   
   

