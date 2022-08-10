# 后端-Spring-三种方式的依赖注入(day2)


# 后端-Spring-三种方式的依赖注入(day2)

## 准备

加入两个实体类

Teacher.java

```java
package org.jsh.entiy;

public class Teacher {
	private String name;
	private int age;
	public Teacher() {
	}
	public Teacher(String name, int age) {
		super();
		this.name = name;
		this.age = age;
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
	@Override
	public String toString() {
		// TODO Auto-generated method stub
		return name+age;
	}
}

```

Course.java

```java
package org.jsh.entiy;

public class Course {
	private String courseName;
	private int courseHour;
	private Teacher teacher;
	public Course() {
	}
	public Course(String courseName, int courseHour, Teacher teacher) {
		super();
		this.courseName = courseName;
		this.courseHour = courseHour;
		this.teacher = teacher;
	}
	public String getCourseName() {
		return courseName;
	}
	public void setCourseName(String courseName) {
		this.courseName = courseName;
	}
	public int getCourseHour() {
		return courseHour;
	}
	public void setCourseHour(int courseHour) {
		this.courseHour = courseHour;
	}
	public Teacher getTeacher() {
		return teacher;
	}
	public void setTeacher(Teacher teacher) {
		this.teacher = teacher;
	}
	@Override
	public String toString() {
		// TODO Auto-generated method stub
		return courseName+courseHour+teacher;
	}
}
```

> 在ioc中定义bean的前提：该bean的类 必须提供了 无参构造

## 1.set方式

property

实际是调用实体类的set方法

基本类型 用 value 

引用类型用 ref 指向 该类的id

> 给对象类型赋值null ：
> 		<property name="name" >  
> 				<null/>       -->注意 没有<value>
> 		</property>
> 赋空值 ""  
> 		<property name="name" >  
> 				<value></value>  
> 		</property>

```xml
	<bean id="teacher" class="org.jsh.entiy.Teacher">
		<property name="name" value="zs"></property>
		<property name="age" value="22"></property>
	</bean>
	
	 <bean id="course" class="org.jsh.entiy.Course">
		<!-- 使用set方法 -->
		<property name="courseName" value="java"></property>
		<property name="courseHour" value="200"></property>
		<!-- 将teacher对象注入到course对象中 -->
		<property name="teacher" ref="teacher"></property>
	</bean> 
```

## 2.构造方法

constructor-arg

调用构造方法赋值

如果只用 value 需要与参数顺序一致

否则 需要使用 index name type 指定

```xml
 <bean id="course" class="org.jsh.entiy.Course">
		<constructor-arg value="c" index="0" name="courseName" type="String"></constructor-arg>
		<constructor-arg value="100"></constructor-arg>
		<constructor-arg ref="teacher"></constructor-arg>
	</bean> 
```

## 3.p命名空间注入

简单类型：
	p:属性名="属性值"
引用类型（除了String外）：
	p:属性名-ref="引用的id"
注意多个 p赋值的时候 要有空格

```xml
	<bean id="teacher" class="org.jsh.entiy.Teacher" p:age="22" p:name="zd">
	</bean>
	
	 <bean id="course" class="org.jsh.entiy.Course" p:courseName="hadoop" p:courseHour="200" p:teacher-ref="teacher">
	</bean> 
```


