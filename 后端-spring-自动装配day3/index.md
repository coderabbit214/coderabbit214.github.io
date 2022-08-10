# 后端-Spring-自动装配(day3)


# 后端-Spring-自动装配(day3)

> 自动装配：
> <bean ... class="org.lanqiao.entity.Course"  autowire="byName|byType|constructor|no" >  byName本质是byId
> byName:  自动寻找：其他bean的id值=该Course类的属性名
> byType:  其他bean的类型(class)  是否与 该Course类的ref属性类型一致  （注意，此种方式 必须满足：当前Ioc容器中 只能有一个Bean满足条件  ）
> constructor： 其他bean的类型(class)  是否与 该Course类的构造方法参数 的类型一致；此种方式的本质就是byType

```xml
	<!-- autowire="byName"
		Course类中有一个ref属性teacher(属性名)，
		并且该ioc容器中恰好有一个bean的id
		bean的id=类的属性名
		就会自动注入
	  -->
	<bean id="course" class="org.jsh.entiy.Course" autowire="byName" >
		<property name="courseName" value="java"></property>
		<property name="courseHour" value="200"></property>
		<!-- <property name="teacher" ref="teacher"></property>	 -->
	</bean> 
```

## 设置全局自动装配

> 头文件

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:p="http://www.springframework.org/schema/p"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd"
	default-autowire="byName"
	>
	
```


