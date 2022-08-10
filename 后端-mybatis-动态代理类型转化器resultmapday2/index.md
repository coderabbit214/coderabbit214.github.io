# 后端-Mybatis-动态代理、类型转化器、resultMap(day2)


# 后端-Mybatis-动态代理、类型转化器、resultMap(day2)

## mapper动态代理方式的crud(Mybatis接口开发)

1. 基础环境 ： mybatis.jar conf.xml mapper.xml
2. 不同之处 ：省略掉 statement ，即根据约定 直接定位出sql语句

### 约定优于配置

1. 根据接口名找到mapper.xml文件 （namepace=接口全类sssss名）

2. 根据接口的方法名找到 mapper.xml 文件中的sql标签（方法名 = sql标签的id值）

   - 方法的 输入参数 和mapper.xml文件中标签的 parameterType类型一致 (如果mapper.xml的标签中没有 parameterType，则说明方法没有输入参数)
   - 方法的返回值  和mapper.xml文件中标签的 resultType类型一致 （无论查询结果是一个 还是多个（student、List<Student>），在mapper.xml标签中的resultType中只写 一个（Student）；如果没有resultType，则说明方法的返回值为void）
   - 习惯：SQL映射文件（mapper.xml） 和 接口放在同一个包中 （注意修改conf.xml中加载mapper.xml文件的路径）

   ```java
   package org.jsh.mapper;
   
   import org.jsh.entity.Student;
   
   import java.util.List;
   
   //操作Mybatis的接口
   public interface StudentMapper {
   //    约定优于配置
   //    1.方法名和mapper.xml中标签的id值一样
   //    2.输入参数与mapper.xml文件中标签的parameterType类型一致
   //    3.返回值与mapper.xml文件中标签的result类型一致
       Student selectStudentByNum(int stunum);
       List<Student> queryAllStudents();
       void addStudent(Student student);
       void updateStudentByNum(Student student);
       void deleteStudentByNum(int stunum);
   }
   ```

   ```xml
   <!-- studentMapper.xml -->
   <mapper namespace="org.jsh.mapper.StudentMapper">
       <!--    通过id值定义标签  paparameterType:输入值类型  resultType返回值类型-->
       <select id="selectStudentByNum" parameterType="int" resultType="student">
   -- sql语句
           select * from student where stunum = #{stunum}
       </select>
   
       <insert id="addStudent" parameterType="org.jsh.entity.Student" >
           insert into student(stunum,name,age) values(#{stunum},#{name},#{age})
       </insert>
       <insert id="addStudentWithConverter" parameterType="org.jsh.entity.Student" >
           insert into student(stunum,name,age,sex) values(#{stunum},#{name},#{age},#{sex,javaType=Boolean,jdbcType=INTEGER})
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

### 使用

StudentMapper studentMapper =session.getMapper(StudentMapper.class) ;
		studentMapper.方法();

通过session对象获取接口（session.getMapper(接口.class);），再调用该接口中的方法，程序会自动执行该方法对应的SQL。

```java
package org.jsh.test;


import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.jsh.entity.Student;
import org.jsh.mapper.StudentMapper;

import java.io.IOException;
import java.io.Reader;
import java.util.List;

public class TestMybatis {
    public static void main(String[] args) throws IOException {
        queryAllStudents();
        //addStudent();
        //deleteStudentByNum();
        //updateStudentByNum();
        //queryAllStudents();
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

        StudentMapper studentMapper = session.getMapper(StudentMapper.class);
//        接口中的方法
        Student student = studentMapper.selectStudentByNum(1);
        System.out.println(student);
        session.close();
    }

    //查询多个学生
    public static void queryAllStudents() throws IOException {

        Reader reader = Resources.getResourceAsReader("conf.xml");
        SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(reader, "development");
        SqlSession session = sessionFactory.openSession();

        StudentMapper studentMapper = session.getMapper(StudentMapper.class);
        List<Student> students = studentMapper.queryAllStudents();
        System.out.println(students);
        session.close();
    }

    //增加学生
    public static void addStudent() throws IOException {

        Reader reader = Resources.getResourceAsReader("conf.xml");
        SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(reader, "development");
        SqlSession session = sessionFactory.openSession();

        Student student = new Student(3, "wu", 24,true);
        StudentMapper studentMapper = session.getMapper(StudentMapper.class);
        studentMapper.addStudent(student);
        session.commit();//提交数据
        session.close();
    }

    //删除学生
    public static void deleteStudentByNum() throws IOException {

        Reader reader = Resources.getResourceAsReader("conf.xml");
        SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(reader, "development");
        SqlSession session = sessionFactory.openSession();

        StudentMapper studentMapper = session.getMapper(StudentMapper.class);
        studentMapper.deleteStudentByNum(3);
        session.commit();//提交数据
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
        student.setAge(2);
        StudentMapper studentMapper = session.getMapper(StudentMapper.class);
        studentMapper.updateStudentByNum(student);
        session.commit();//提交数据
        session.close();
    }
}

```

## 优化

### 分离配置信息

#### 将配置信息放入 db.properties文件中

 db.properties：
	k=v

```properties
driver=com.mysql.cj.jdbc.Driver
url=jdbc:mysql://localhost:3306/testdata?useSSL=true&serverTimezone=UTC
username=root
password=123456
```

#### 动态引入

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    
    
    
<!--    优化：引入properties文件-->
    <properties resource="db.properties" />
    
    
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${driver}"/>
                <property name="url" value="${url}"/>
                <property name="username" value="${username}"/>
                <property name="password" value="${password}"/>
            </dataSource>
        </environment>
    </environments>

    <mappers>
        <!--        加载映射文件-->
        <mapper resource="org/jsh/mapper/studentMapper.xml"/>
    </mappers>

</configuration>
```

### MyBatis全局参数(轻易不要修改)

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
<!--    优化：引入properties文件-->
    <properties resource="db.properties" />
    
    
    
    
<!--    全局变量设置-->
<!--    <settings>-->
<!--        <setting name="cacheEnabled" value="false"/>-->
<!--    </settings>-->


    
    
    
    <environments default="development">

        <environment id="development">

            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${driver}"/>
                <property name="url" value="${url}"/>
                <property name="username" value="${username}"/>
                <property name="password" value="${password}"/>
            </dataSource>
        </environment>

    </environments>

    <mappers>
        <!--        加载映射文件-->
        <mapper resource="org/jsh/mapper/studentMapper.xml"/>
    </mappers>

</configuration>
```

| 参数                      | 简介                                                         |               有效值               |
| ------------------------- | ------------------------------------------------------------ | :--------------------------------: |
| cacheEnabled              | 在全局范围内，启用或禁用缓存                                 |        true（默认）、false         |
| lazyLoadingEnabled        | 在全局范围内启用或禁用延迟加载。当禁用时，所有相关联的对象都将立即加载（热加载）。 |        true（默认）、false         |
| aggressiveLazyLoading     | 启用时，有延迟加载属性的对象，在被调用时将会完全加载所有属性（立即加载）。否则，每一个属性都将按需加载（即延迟加载）。 |        true（默认）、false         |
| multipleResultSetsEnabled | 允许或禁止执行一条单独的SQL语句后返回多条结果（结果集）；需要驱动程序的支持 |        true（默认）、false         |
| autoMappingBehavior       | 指定数据表字段和对象属性的映射方式。  NONE：禁止自动映射，只允许手工配置的映射  PARTIAL：只会自动映射简单的、没有嵌套的结果  FULL：自动映射任何结果（包含嵌套等） |  NONE、  PARTIAL（默认）、  FULL   |
| defaultExecutorType       | 指定默认的执行器。  SIMPLE：普通的执行器。  REUSE：可以重复使用prepared statements语句。  BATCH：可以重复执行语句和批量更新。 |  SIMPLE（默认）、  REUSE、  BATCH  |
| defaultStatementTimeout   | 设置驱动器等待数据库回应的最长时间                           | 以秒为单位的，任意正整数。无默认值 |
| safeRowBoundsEnabled      | 允许或禁止使用嵌套的语句                                     |        true、false（默认）         |
| mapUnderscoreToCamelCase  | 当在数据表中遇到有下划线的字段时，自动映射到相应驼峰式形式的Java属性名。例如，会自动将数据表中的stu_no字段，映射到POJO类的stuNo属性。 |        true、false（默认）         |
| lazyLoadTriggerMethods    | 指定触发延迟加载的对象的方法                                 | equals、clone、hashCode、toString  |

### 别名设置（实用）

消除studentMapper.xml 中需要写全类名的繁琐形式

conf.xml :

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
<!--    优化：引入properties文件-->
    <properties resource="db.properties" />


<!--    定义单个/多个别名-->
    <typeAliases>
<!--        单个别名 不区分大小写-->
<!--        <typeAlias type="org.jsh.entity.student" alias="student" />-->
<!--        批量定义别名 不区分大小写 别名就是不带包名的类名-->
        <package name="org.jsh.entity"/>
    </typeAliases>


    <environments default="development">

        <environment id="development">

            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${driver}"/>
                <property name="url" value="${url}"/>
                <property name="username" value="${username}"/>
                <property name="password" value="${password}"/>
            </dataSource>
        </environment>

    </environments>

    <mappers>
        <!--        加载映射文件-->
        <mapper resource="org/jsh/mapper/studentMapper.xml"/>
    </mappers>

</configuration>
```

studentMapper.xml :

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- namespace:该mapper.xml映射文件的唯一标识-->
<!--约定 对应接口全类名-->
<mapper namespace="org.jsh.mapper.StudentMapper">
    <!--    通过id值定义标签  paparameterType:输入值类型  resultType返回值类型-->
    <select id="selectStudentByNum" parameterType="int" resultType="student">
-- sql语句
        select * from student where stunum = #{stunum}
    </select>

    <insert id="addStudent" parameterType="student" >
        insert into student(stunum,name,age) values(#{stunum},#{name},#{age})
    </insert>
    <update id="updateStudentByNum" parameterType="student">
        update student set name=#{name},age=#{age} where stunum=#{stunum}
    </update>
    <delete id="deleteStudentByNum" parameterType="int">
        delete from student where  stunum = #{stunum}
    </delete>
<!--    无论返回一个值还是一个列表 resultType不变-->
    <select id="queryAllStudents" resultType="student">
        select * from student
    </select>
</mapper>

```

## 类型转换器

1.MyBatis自带一些常见的类型处理器
	int  - number

2.自定义MyBatis类型处理器

示例：
实体类Student :  boolean   stuSex  	
			true:男
			false：女

表student：	number  stuSex
			1:男
			0：女
自定义类型转换器（boolean -number）步骤：

### 创建转换器：需要实现TypeHandler接口

​	通过阅读源码发现，此接口有一个实现类 BaseTypeHandler ，因此 要实现转换器有2种选择：
​	i.实现接口TypeHandler接口
​	ii.继承BaseTypeHandler

```java
package org.jsh.converter;

import org.apache.ibatis.type.BaseTypeHandler;
import org.apache.ibatis.type.JdbcType;

import java.sql.CallableStatement;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

//BaseTypeHandler<java类型>
public class BooleanAndIntConverter extends BaseTypeHandler<Boolean> {

//  java -> DB
    /*
        PreparedStatement : 操作的PreparedStatement
        i: PreparedStatement对象操作的位置
        Boolean:Java值
        JdbcType：数据库类型
     */
    @Override
    public void setNonNullParameter(PreparedStatement preparedStatement, int i, Boolean aBoolean, JdbcType jdbcType) throws SQLException {
        if(aBoolean){
            preparedStatement.setInt(i,1);
        }else{

        }
    }
//  DB -> java
    @Override
    public Boolean getNullableResult(ResultSet resultSet, String s) throws SQLException {
        int sexNum = resultSet.getInt(s);
        return sexNum == 1?true:false;
    }
    //  DB -> java
    @Override
    public Boolean getNullableResult(ResultSet resultSet, int i) throws SQLException {
        int sexNum = resultSet.getInt(i);
        return sexNum == 1?true:false;
    }
    //  DB -> java
    @Override
    public Boolean getNullableResult(CallableStatement callableStatement, int i) throws SQLException {
        int sexNum = callableStatement.getInt(i);
        return sexNum == 1?true:false;
    }
}

```

b.配置conf.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
<!--    优化：引入properties文件-->
    <properties resource="db.properties" />
    <typeAliases>
        <package name="org.jsh.entity"/>
    </typeAliases>

    
    
    
<!--    配置转换器-->
    <typeHandlers>
        <typeHandler handler="org.jsh.converter.BooleanAndIntConverter" javaType="Boolean" jdbcType="INTEGER" />
    </typeHandlers>
    
    
    
    
    <environments default="development">

        <environment id="development">

            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${driver}"/>
                <property name="url" value="${url}"/>
                <property name="username" value="${username}"/>
                <property name="password" value="${password}"/>
            </dataSource>
        </environment>

    </environments>

    <mappers>
        <!--        加载映射文件-->
        <mapper resource="org/jsh/mapper/studentMapper.xml"/>
    </mappers>

</configuration>
```

### 使用

studentMapper.xml:

1. 查询
   1. 把select标签中resultType属性变为resultMap 值为id
   2. 创建resultMap标签
      - 属性：id:传回select标签
      - 属性： type: 返回值类型
      - 子标签：
        - 表中主键：标签为id 非主键result
        - property:java实体类中属性名
        - column：数据表中属性名
        - javaType ：java实体类中属性类型
        - jdbcType ：数据表中属性类型

```xml

<select id="selectStudentByNumWithConverter" parameterType="int" resultMap="studentResult">
        select * from student where stunum = #{stunum}
    </select>

    <resultMap id="studentResult" type="student">
<!--        主键-->
        <id property="stunum" column="stunum"></id>
<!--        非主键-->
        <result property="name" column="name" />
        <result property="age" column="age" />
        <result property="sex" column="sex" javaType="boolean" jdbcType="INTEGER" />
    </resultMap>

```

2. 查询

```xml
<insert id="addStudentWithConverter" parameterType="student" >
        insert into student(stunum,name,age,sex) values(#{stunum},#{name},#{age},#{sex,javaType=Boolean,jdbcType=INTEGER})
    </insert>
```

## result

resultMap可以实现2个功能：
1.类型转换

2.属性-字段的映射关系(当java实体类中属性名与数据表中属性名不同时) 忽略大小写

```xml
<select id="queryStudentByStuno" 	parameterType="int"  	resultMap="studentMapping" >
		select * from student where stuno = #{stuno}
	</select>
	
	<resultMap type="student" id="studentMapping">
			<!-- 分为主键id 和非主键 result-->
			<id property="id"  column="stuno"  />
			<result property="stuName"  column="stuname" />
			<result property="stuAge"  column="stuage" />
			<result property="graName"  column="graname" />
			<result property="stuSex"  column="stusex"  javaType="boolean" jdbcType="INTEGER"/>
	
	
	</resultMap>
```


