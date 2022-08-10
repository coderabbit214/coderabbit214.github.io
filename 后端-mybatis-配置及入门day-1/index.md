# Mybatis-配置及入门


# Mybatis-配置及入门

## 简介

- ibatis:apache
- 2010 ibatis -> google colde ,Mybatis 

## 作用

简化JDBC操作，实现数据的持久化（包存到数据库）。

### ORM（是一个概念）

- ORM：Object Relational(关系) Mapping
- person对象(java)  <=>  person表(数据库)    映射
- Mybatis是ORM的一个实现
- ORM可以使开发人员像操作对象一样 操作数据库表

## 开发Mybatis程序步骤：

### 下载Mybatis-jar导入集成工具

### conf.xml : 配置数据库信息 和 需要加载的映射文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!--    通过environments的default值和environment的id值指定Mybatis运行时的数据库环境-->
    <environments default="development">

        <!--        开发环境-->
        <environment id="development">
            <!--            配置事务提交方式
                            JDBC：利用JDBC方式处理事务（commit rollback close）手工
                            MANAGED:将事务交由其他组件托管（spring,jobss）自动，默认关闭连接-->
            <!--            默认不关闭-->
            <!--            <transactionManager type="MANAGED"/>-->
            <!--            <property name="closeConnection" value="false"/>-->
            <transactionManager type="JDBC"/>
            <!--                配置数据库信息-->
            <!--            数据源类型：
                                    POOLED：使用数据库连接池（省略数据库的打开和关闭）
                                    UNPOOLED：传统的JDBC模式（不推荐：每次访问数据库都需要打开关闭数据库，消耗性能高）
                                    JNDI：从tomcat中获取一个内置的数据库连接池-->
            <dataSource type="POOLED">
                <!--                数据库驱动-->
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <!--                连接字符串-->
                <property name="url" value="jdbc:mysql://localhost:3306/testdata?useSSL=true&amp;serverTimezone=UTC"/>
                <!--                数据库账号-->
                <property name="username" value="root"/>
                <!--                数据库密码-->
                <property name="password" value="123456"/>
            </dataSource>
        </environment>
        
        
        <!--        运行环境-->
        <environment id="start">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/testdata?useSSL=true&amp;serverTimezone=UTC"/>
                <property name="username" value="root"/>
                <property name="password" value="123456"/>
            </dataSource>
        </environment>

    </environments>

    <mappers>
        <!--        加载映射文件-->
        <mapper resource="org/jsh/entity/studentMapper.xml"/>
    </mappers>

</configuration>
```



### 数据库中 表(和类属性一一对应)

### Java 类

```java
package org.jsh.entity;

public class Student {
    private int stunum;
    private String name;
    private int age;

    public Student() {

    }

    public Student(int stunum, String name, int age) {
        this.stunum = stunum;
        this.name = name;
        this.age = age;
    }

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

    @Override
    public String toString() {
        return "Student{" +
                "stunum=" + stunum +
                ", name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}

```



### 映射文件 xxMapper,xml:

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- namespace:该mapper.xml映射文件的唯一标识-->
<mapper namespace="org.jsh.entity.studentMapper">
    <!--    通过id值定义标签  paparameterType:输入值类型  resultType返回值类型-->
    <select id="selectStudentByNum" parameterType="int" resultType="org.jsh.entity.Student">
-- sql语句
        select * from student where stunum = #{stunum}
    </select>
<!--    Mybatis 规定输入输出在语法上只有一个 paparameterType，resultType只有一个 -->
<!--    如果输入参数是简单类型（8个基本类型+String）可以使用任何占位符-->
<!--    如果是对象类型。则必须是对象的属性 例：#{属性名}-->
    <insert id="addStudent" parameterType="org.jsh.entity.Student" >
        insert into student(stunum,name,age) values(#{stunum},#{name},#{age})
    </insert>
    <update id="updateStudentByNum" parameterType="org.jsh.entity.Student">
        update student set name=#{name},age=#{age} where stunum=#{stunum}
    </update>
    <delete id="deleteStudentByNum" parameterType="int">
        delete from student where  stunum = #{stunum}
    </delete>
<!--    无论返回一个值还是一个列表 resultType不变-->
    <select id="queryAllStudents" resultType="org.jsh.entity.Student">
        select * from student
    </select>
</mapper>

```

### 测试类

```java
package org.jsh.entity;


import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

import java.io.IOException;
import java.io.Reader;
import java.util.List;

public class TestMybatis {
    public static void main(String[] args) throws IOException {
        queryAllStudents();
        // addStudent();
        // deleteStudentByNum();
        updateStudentByNum();
        queryAllStudents();
    }

    //查询单个学生
    public static void queryStudentByNum() throws IOException {
        //        加载Mybatis配置文件
//        conf.xml -> reader
        Reader reader = Resources.getResourceAsReader("conf.xml");
//        reader - > SqlSession
//        build的第二个参数 修改conf.xml中  <environments default="development"> 的默认值
        SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(reader, "development");
//        SqlSession -> connection
        SqlSession session = sessionFactory.openSession();
//        ("namespace.id","sql参数")
//        动态代理 自动将返回值变为Student
        Student student = session.selectOne("org.jsh.entity.studentMapper.selectStudentByNum", 1);
        System.out.println(student);
        session.close();
    }

    //查询多个学生
    public static void queryAllStudents() throws IOException {

        Reader reader = Resources.getResourceAsReader("conf.xml");
        SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(reader, "development");
        SqlSession session = sessionFactory.openSession();

        List<Student> students = session.selectList("org.jsh.entity.studentMapper.queryAllStudents");
        System.out.println(students);
        session.close();
    }

    //增加学生
    public static void addStudent() throws IOException {

        Reader reader = Resources.getResourceAsReader("conf.xml");
        SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(reader, "development");
        SqlSession session = sessionFactory.openSession();

        Student student = new Student(3, "wu", 24);
        int count = session.insert("org.jsh.entity.studentMapper.addStudent", student);
        session.commit();//提交数据
        System.out.println(count);
        session.close();
    }

    //增加学生
    public static void deleteStudentByNum() throws IOException {

        Reader reader = Resources.getResourceAsReader("conf.xml");
        SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(reader, "development");
        SqlSession session = sessionFactory.openSession();

        int count = session.delete("org.jsh.entity.studentMapper.deleteStudentByNum", 3);
        session.commit();//提交数据
        System.out.println(count);
        session.close();
    }

    //修改
    public static void updateStudentByNum() throws IOException {

        Reader reader = Resources.getResourceAsReader("conf.xml");
        SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(reader, "development");
        SqlSession session = sessionFactory.openSession();

//        修改的参数
        Student student = new Student();
//        修改哪个人，wehere stunum = 2
        student.setStunum(2);
//        修改成什么样子
        student.setName("jsh");
        student.setAge(22);
        int count = session.update("org.jsh.entity.studentMapper.updateStudentByNum", student);
        session.commit();//提交数据
        System.out.println(count);
        session.close();
    }
}


```

## 注意事项

- 如果事务提交方式为JDBC 则需要手动commit


