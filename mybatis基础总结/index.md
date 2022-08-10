# Mybatis基础总结


# Mybatis基础总结

## 一.配置

1. 导入所需jar

   - Mybatis-jar
   - 数据库所需jar

2. 在src下配置conf.xml

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

3. 创建数据库中表

4. 创建java实体类 与表中属性一一对应

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

5. 映射文件xxxMapper.xml

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

6. 测试

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

## 二.动态代理 接口开发

> 省略掉 statement ，即根据约定 直接定位出sql语句
>
> 约定优于配置
>
> 根据接口的方法名找到 xxxmapper.xml 文件中的sql标签（方法名 = sql标签的id值）
>
> - 方法的 输入参数 和mapper.xml文件中标签的 parameterType类型一致 (如果mapper.xml的标签中没有 parameterType，则说明方法没有输入参数)
> - 方法的返回值  和mapper.xml文件中标签的 resultType类型一致 （无论查询结果是一个 还是多个（student、List<Student>），在mapper.xml标签中的resultType中只写 一个（Student）；如果没有resultType，则说明方法的返回值为void）
> - 习惯：SQL映射文件（mapper.xml） 和 接口放在同一个包中 （注意修改conf.xml中加载mapper.xml文件的路径）

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

使用

```java
StudentMapper studentMapper=session.getMapper(StudentMapper.class);
		studentMapper.方法();
```

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

## 三.分离配置信息

1. 将配置信息放入 db.properties文件中

    db.properties： k=v

   ```properties
   driver=com.mysql.cj.jdbc.Driver
   url=jdbc:mysql://localhost:3306/testdata?useSSL=true&serverTimezone=UTC
   username=root
   password=123456
   ```

2. 动态引入

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

## 四.Mybatis全局参数

```xml
<!--    全局变量设置-->
  <settings>
  	<setting name="cacheEnabled" value="false"/>
  </settings>
```



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

## 五.别名设置

> 消除studentMapper.xml 中需要写全类名的繁琐形式

conf.xml :

```xml
<configuration>
...

<!--    定义单个/多个别名-->
    <typeAliases>
<!--        单个别名 不区分大小写-->
<!--        <typeAlias type="org.jsh.entity.student" alias="student" />-->
<!--        批量定义别名 不区分大小写 别名就是不带包名的类名-->
        <package name="org.jsh.entity"/>
    </typeAliases>

 ...
</configuration>
```

## 六.类型转换器(不太懂)

1. 自带 例如：int→number。。。

2. 自定义

   示例：
   实体类Student :  boolean   stuSex  	
   			true:男
   			false：女

   表student：	number  stuSex
   			1:男
   			0：女
   自定义类型转换器（boolean -number）步骤：

   1. 创建转换器：需要实现TypeHandler接口

      - 实现接口TypeHandler接口
      - 继承BaseTypeHandler

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

   2. 配置conf.xml

      ```xml
      
      <configuration>
      。。。
          
      <!--    配置转换器-->
          <typeHandlers>
              <typeHandler handler="org.jsh.converter.BooleanAndIntConverter" javaType="Boolean" jdbcType="INTEGER" />
          </typeHandlers>
          
        。。。
      </configuration>
      ```

   3. 使用

      查询

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

 		增删改

```xml
<insert id="addStudentWithConverter" parameterType="student" >
        insert into student(stunum,name,age,sex) values(#{stunum},#{name},#{age},#{sex,javaType=Boolean,jdbcType=INTEGER})
    </insert>
```

## 七.输入参数 parameterType

1. 类型为 简单类型（8个基本类型+String)

   - #{任意值} 

     - 自动给String类型加上 ' '
     - 可以防止SQL注入

   - ${value}

     - 其中标识符只能是value 原样输出 
     - 如果是String 类型 则需要在外加上单引号  '${value}'
     - 适合动态排序（动态字段）

     ```xml
      <select id="queryAllStudentsOrderByColumn" parameterType="String" resultType="student">
             select * from student order  by ${value} asc
         </select>
     ```

     

     - 不防止SQL注入

2. 类型为对象：

   - #{}

  - #{对象属性名} 或者使用parameterMap

    ```xml
      <select id="queryStudentsOrderByColumn" parameterType="student" resultType="student">
            select * from student where stunum = #{stunum} or name like #{name}
        </select>
    ```
 - 模糊查询需要在传递参数时加上 %
  ```java
    public static void queryStudentsOrderByColumn() throws IOException {
     
             Reader reader = Resources.getResourceAsReader("conf.xml");
             SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(reader, "development");
             SqlSession session = sessionFactory.openSession();
             Student student = new Student(3, "%j%", 24,true);
             StudentMapper studentMapper = session.getMapper(StudentMapper.class);
             List<Student> students = studentMapper.queryStudentsOrderByColumn(student);
             System.out.println(students);
             session.close();
         }
  ```

   - ${}

     - sql语句直接写好

     ```xml
      <select id="queryStudentsOrderByColumn" parameterType="student" resultType="student">
             select * from student where stunum = #{stunum} or name like '%${name}%'
         </select>
     ```

     - 传参

     ```java
      public static void queryStudentsOrderByColumn() throws IOException {
     
             Reader reader = Resources.getResourceAsReader("conf.xml");
             SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(reader, "development");
             SqlSession session = sessionFactory.openSession();
             Student student = new Student(3, "j", 24,true);
             StudentMapper studentMapper = session.getMapper(StudentMapper.class);
             List<Student> students = studentMapper.queryStudentsOrderByColumn(student);
             System.out.println(students);
             session.close();
         }
     ```


3. 嵌套属性 

   数据表加入schooladdress，homeaddress

   java实体类 Address  属性： schooladdress，homeaddress

   Student类中 加入Address 属性

   1. 方法一

   - xml

   ```xml
   <select id="selectStudentByaddress" parameterType="address" resultType="student">
           select * from student where homeaddress = #{homeAddress} or schooladdress = '${schoolAddress}'
       </select>
   ```

   - java

   ```java
    public static void selectStudentByaddress() throws IOException {
   
           Reader reader = Resources.getResourceAsReader("conf.xml");
           SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(reader, "development");
           SqlSession session = sessionFactory.openSession();
   
           Address address = new Address("xa","dl");
           StudentMapper studentMapper = session.getMapper(StudentMapper.class);
           List<Student> students = studentMapper.selectStudentByaddress(address);
           System.out.println(students);
           session.close();
       }
   ```

   2. 方法二，支持级联

   - xml

   ```xml
   <select id="selectStudentByaddress" parameterType="student" resultType="student">
           select * from student where 
       homeaddress = #{address.homeAddress} or 
       schooladdress = '${address.schoolAddress}'
       </select>
   ```

   - java

   ```java
    public static void selectStudentByaddress() throws IOException {
   
           Reader reader = Resources.getResourceAsReader("conf.xml");
           SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(reader, "development");
           SqlSession session = sessionFactory.openSession();
   
           Address address = new Address("xa","dl");
           Student student = new Student();
           student.setAddress(address);
           StudentMapper studentMapper = session.getMapper(StudentMapper.class);
           List<Student> students = studentMapper.selectStudentByaddress(student);
           System.out.println(students);
           session.close();
   ```

4. 输入对象为HashMap

   - mapper.xml

   ```xml
   <select id="queryStudentsOrderByColumnWithHashMap" parameterType="HashMap" resultType="student">
           select * from student where stunum = #{stunum} or name like '%${name}%'
       </select>
   ```

   

   - interface

   ```java
       List<Student> queryStudentsOrderByColumnWithHashMap(Map<String,Object> map);
   
   ```

   

   - Test

   ```java
   public static void queryStudentsOrderByColumnWithHashMap() throws IOException {
   
           Reader reader = Resources.getResourceAsReader("conf.xml");
           SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(reader, "development");
           SqlSession session = sessionFactory.openSession();
   
           Map<String,Object>  studentMap = new HashMap<>();
           studentMap.put("stunum","2");
           studentMap.put("name","jsh");
       
           StudentMapper studentMapper = session.getMapper(StudentMapper.class);
           List<Student> students = studentMapper.queryStudentsOrderByColumnWithHashMap(studentMap);
           System.out.println(students);
           session.close();
       }
   ```

5. 通过调用【存储过程】 实现查询

   statementType="CALLABLE"

   1. mapper.xml

   - statementType="CALLABLE"
   - parameterType必须是HashMap
   - 没有返回值

   ```XML
   <!--    通过调用【存储过程】 实现查询，statementType="CALLABLE"-->
       <select id="queryCountByGradeWithProcedure" parameterType="HashMap" statementType="CALLABLE">
               {CALL queryCountByGradeWithProcedure(#{gName,jdbcType=VARCHAR,mode=IN},#{scount,jdbcType=INTEGER,mode=OUT})}
       </select>
   </mapper>
   ```

   2. interface

   ```
    void queryCountByGradeWithProcedure(Map<String,Object> map);
   ```

   3. Test.java

   - 通过Map传值
   - 输出过程需要另行接收 (通过Map的形式)

   ```java
   public static void queryCountByGradeWithProcedure() throws IOException {
   
           Reader reader = Resources.getResourceAsReader("conf.xml");
           SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(reader, "development");
           SqlSession session = sessionFactory.openSession();
   
           Map<String,Object>  studentMap = new HashMap<>();
           studentMap.put("gName","s1");
           StudentMapper studentMapper = session.getMapper(StudentMapper.class);
           studentMapper.queryCountByGradeWithProcedure(studentMap);
   //        获取存储过程输出
           Object count = studentMap.get("sCount");
           session.close();
       }
   ```


## 八.输出参数resultType

1. 八个简单类型+String

   1. mapper.xml

   ```xml
   <!--输出参数resultType-->
       <select id="selectStudentCount" resultType="int">
           select count(*) from student
       </select>
   ```

   2. interface

   ```java
   int selectStudentCount();
   ```

   3. test.java

   ```java
   public static void selectStudentCount() throws IOException {
   
           Reader reader = Resources.getResourceAsReader("conf.xml");
           SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(reader);
           SqlSession session = sessionFactory.openSession();
   
           StudentMapper studentMapper = session.getMapper(StudentMapper.class);
           int count = studentMapper.selectStudentCount();
           System.out.println(count);
           session.close();
       }
   
   ```

2. 对象类型

   1. mapper.xml

   ```xml
   <select id="queryStuByNum" parameterType="int" resultType="student">
           select * from student where stunum = #{stunum}
       </select>
   ```

   2. interface

   ```java
   Student queryStuByNum(int stunum);
   ```

   3. test.java

   ```java
   public static void queryStuByNum() throws IOException {
   
           Reader reader = Resources.getResourceAsReader("conf.xml");
           SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(reader);
           SqlSession session = sessionFactory.openSession();
   
           StudentMapper studentMapper = session.getMapper(StudentMapper.class);
           Student student= studentMapper.queryStuByNum(2);
           System.out.println(student);
           session.close();
       }
   ```

3. 实体对象的集合类型

   > resultType不变 函数返回值变为列表

4. HashMap类型（起别名）

   一个map一个学生 多人用list

   mapper.xml

   ```xml
   <select id="queryStudentOutByHashMap" resultType="HashMap">
           select stunum "no",name "name" from student
       </select>
   ```

   interface

   ```java
   //返回一个
   HashMap<String,Object> queryStudentOutByHashMap();
   //返回多个
   List<HashMap<String,Object>> queryStudentOutByHashMap();
   ```

   test

   ```java
   public static void queryStudentOutByHashMap() throws IOException {
   
           Reader reader = Resources.getResourceAsReader("conf.xml");
           SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(reader);
           SqlSession session = sessionFactory.openSession();
   
           StudentMapper studentMapper = session.getMapper(StudentMapper.class);
       //返回多个
           List<HashMap<String,Object>> map = studentMapper.queryStudentOutByHashMap();
       //返回一个
       HashMap<String,Object> map = studentMapper.queryStudentOutByHashMap();
           System.out.println(map);
           session.close();
       }
   ```

5. resultMap(实体类的属性和数据表的字段：类型，名字不同时)

   > resultMap="queryStudentByID"   id="queryStudentByID"对应
   >
   > property 实体类属性名（严格大小写）
   >
   > column 数据表属性名

   ```xml
      <select id="queryStudentByID" parameterType="int" resultMap="queryStudentByID">
           select stunum,name from student where stunum = #{stunum}
       </select>
   
       <resultMap id="queryStudentByID" type="student">
   <!--        主键-->
           <id property="stunum" column="stuNum"></id>
   <!--        其他-->
           <result column="name" property="name"></result>
       </resultMap>
   ```


## 九.动态SQL

### where if

- mapper.xml

```xml
<select id="queryStudentByNameOrAgeSQLTag" parameterType="student" resultType="student">
        select stunum,name from student
        <where>
        <if test="stunum != null and stunum!=''">
          and name = #{name}
        </if>
        <if test="age != null and age!=0">
            and age = #{age}
        </if>
        </where>
    </select>
```

> where 语句用标签显示 
>
> if语句为条件 
>
> ​		test中表示属性存在且不为空
>
> > sql 语句前加and 
> >
> > where标签会自动解析

### foreach

查询学号为1，2，3的学生信息

SQL：select * from student where stunum in(1,2,3)

作用：stunums = {1,2,3}

foreach 迭代的类型：数组，集合，对象数组，对象属性

1. Grade.java

```java
package org.jsh.entity;

import java.util.List;

public class Grade {
    //学号
    private List<Integer> stunums;

    public List<Integer> getStunums() {
        return stunums;
    }

    public void setStunums(List<Integer> stunums) {
        this.stunums = stunums;
    }
    //
}

```



2. mapper.xml

> collection: 对象的属性名
>
> open：循环变量前边部分
>
> close ：循环变量后边部分
>
> separator： 循环变量的分隔符

```xml
<select id="queryStudentsWithGrade" parameterType="Grade" resultType="student">
        select * from student
        <where>
            <if test="stunums != null and stunums.size>0">
                <foreach collection="stunums" open=" and stunum in (" close=")" item="stunum" separator=",">
                    #{stunum}
                </foreach>
            </if>
        </where>
    </select>
```

3. interface

```java
List<Student> queryStudentsWithGrade(Grade grade);
```

4. test

```java
  public static void queryStudentsWithGrade() throws IOException {

        Reader reader = Resources.getResourceAsReader("conf.xml");
        SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(reader);
        SqlSession session = sessionFactory.openSession();

        StudentMapper studentMapper = session.getMapper(StudentMapper.class);
        Grade grade = new Grade();
        List<Integer> list = new ArrayList<>();
        list.add(1);
        list.add(2);
        list.add(3);
        grade.setStunums(list);
        List<Student> students = studentMapper.queryStudentsWithGrade(grade);
        System.out.println(students);
        session.close();
    }

```

## 十.关联查询

### 业务扩展类

> 核心：用resultType指定类的属性 包含 多表查询的所有字段

StudentBusiness.java

```java
package org.jsh.entity;

public class StudentBusiness extends Student{
    private int cardID;
    private String cardInfo;

    public String getCardInfo() {
        return cardInfo;
    }

    public void setCardInfo(String cardInfo) {
        this.cardInfo = cardInfo;
    }

    public int getCardID() {
        return cardID;
    }

    public void setCardID(int cardID) {
        this.cardID = cardID;
    }

    @Override
    public String toString() {
        return "StudentBusiness{" +
                "cardID=" + cardID +
                super.toString()+
                ", cardInfo='" + cardInfo + '\'' +
                '}';
    }
}

```

mapper.xml

```xml
<select id="queryStudentsByNumWithSAndC" parameterType="int" resultType="StudentBusiness">
        SELECT s.*,c.* FROM student s INNER JOIN studentcard c ON s.`stucardid`=c.`cardid` WHERE s.`stunum` = #{stunum}
    </select>
```

### resultMap

>  通过属性成员将两个类建立起链接

StudentCard.java

```java
package org.jsh.entity;

public class StudentCard {
private int cardID;
private String cardInfo;

public String getCardInfo() {
        return cardInfo;
        }

public void setCardInfo(String cardInfo) {
        this.cardInfo = cardInfo;
        }

public int getCardID() {
        return cardID;
        }

public void setCardID(int cardID) {
        this.cardID = cardID;
        }

    @Override
    public String toString() {
        return "StudentCard{" +
                "cardID=" + cardID +
                ", cardInfo='" + cardInfo + '\'' +
                '}';
    }
}

```

Student.java

```java
package org.jsh.entity;

public class Student {
    private int stunum;
    private String name;
    private int age;
    private boolean sex;
    private Address address;
    private StudentCard studentCard;
    public Student() {

    }

    public Student(int stunum, String name, int age, boolean sex) {
        this.stunum = stunum;
        this.name = name;
        this.age = age;
        this.sex = sex;
    }

    public Student(int stunum, String name, int age, boolean sex,Address address) {
        this.stunum = stunum;
        this.name = name;
        this.age = age;
        this.sex = sex;
        this.address = address;
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
                ", sex=" + sex +
                ", address=" + address +
                ", studentBusiness=" + studentCard +
                '}';
    }

    public boolean isSex() {
        return sex;
    }

    public void setSex(boolean sex) {
        this.sex = sex;
    }

    public Address getAddress() {
        return address;
    }

    public void setAddress(Address address) {
        this.address = address;
    }
}

```

mapper.xml

> 使用resultmap
>
> 一对一对象成员用 association 映射  javaType指定该属性的类型
>
> <学生>
>
> ​	<属性/>
>
> ​	<属性/>
>
> ​	<association>
>
> ​		<属性/>
>
> ​		<属性/>
>
> ​	</association>
>
> </学生>

```xml
 <select id="queryStudentsByNumWithSAndC" parameterType="int" resultMap="student_card_map">
        SELECT s.*,c.* FROM student s INNER JOIN studentcard c ON s.`stucardid`=c.`cardid` WHERE s.`stunum` = #{stunum}
    </select>


    <resultMap id="student_card_map" type="Student">
<!--        学生信息-->
        <id property="stunum" column="stunum"></id>
        <result property="name" column="name"></result>
        <result property="age" column="age"></result>
<!--        一对一对象成员用association映射  javaType指定该属性的类型 -->
        <association property="studentCard" javaType="StudentCard">
            <id property="cardID" column="cardID"></id>
            <result property="cardInfo" column="cardInfo"></result>
        </association>
    </resultMap>
```

### 一对多

studentclasss->student 一对多

studentclass.java

```java
package org.jsh.entity;

import java.util.List;

public class StudentClass {
    private  int classID;
    private  String classname;
    private List<Student> students;

    public StudentClass() {
    }

    public int getClassID() {
        return classID;
    }

    public void setClassID(int classID) {
        this.classID = classID;
    }

    public String getClassname() {
        return classname;
    }

    public void setClassname(String classname) {
        this.classname = classname;
    }

    public List<Student> getStudents() {
        return students;
    }

    public void setStudents(List<Student> students) {
        this.students = students;
    }

    @Override
    public String toString() {
        return "StudentClass{" +
                "classID=" + classID +
                ", classname='" + classname + '\'' +
                ", students=" + students +
                '}';
    }
}

```

mapper.xml

> 一对多用collection 映射
>
> ```
> javatype指定的是user对象的属性的类型（例如id，posts），而oftype指定的是映射到list集合属性中pojo的类型
> ```

```xml
<!--    一对多-->
    <select id="queryClassAndStudents" parameterType="int" resultMap="class_student_map">
        SELECT c.*,s.* FROM student s INNER JOIN studentclass c ON c.`classid`=s.`classid` WHERE s.`classid` = #{classID}
    </select>
    <resultMap id="class_student_map" type="StudentClass">
        <id property="classID" column="classID"/>
        <result property="classname" column="classname"/>
        <collection property="students" ofType="student">
            <id property="stunum" column="stunum"></id>
            <result property="name" column="name"></result>
            <result property="age" column="age"></result>
        </collection>
    </resultMap>
</mapper>

```

## 十一.日志(Log4j)

1. 导入 Log4j.jar

2. conf.xml配置

```xml
<settings>
<!--        开启日志，并指定使用的具体日志-->
        <setting name="logImpl" value="LOG4J"/>
    </settings>
```

> 如果不指定，Mybatis就会根据以下顺序寻找日志
>
> SLF4J → Apache Commons Logging  → Log4j 2 → Log4j → JDK logging

3. 编写日志输出文件 log4j.properties

```properties
log4j.rootLogger=debug,stdout
log4j.appender.stdout=org.apache.log4j.ConsoleAppender 
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout 
log4j.appender.stdout.layout.ConversionPattern=%5p [%t] - %m%n
```

> 日志级别
>
> DEBUG<INFO<WARN<ERROR
>
> 如果设置为DEBUG 则只显示DEBUG及以上级别的信息
>
> 建议 在开发时设置为DEBUG 在运行时设置为INFO及以上

## 十二.延迟加载

1. conf.xml 配置

```xml
 <settings>
<!--        开启日志，并指定使用的具体日志-->
        <setting name="logImpl" value="LOG4J"/>
        
<!--        开启延迟加载-->
        <setting name="lazyLoadingEnabled" value="true"/>
<!--        关闭立即加载-->
        <setting name="aggressiveLazyLoading" value="false"/>
    </settings>
```

2. mapper.xml

```xml
 <select id="queryStudentsByNumWithSAndC1" parameterType="int" resultMap="student_card_lazyload_map">
        SELECT * FROM student
    </select>
    <resultMap id="student_card_lazyload_map" type="Student">
        <!--        学生信息-->
        <id property="stunum" column="stunum"></id>
        <result property="name" column="name"></result>
        <result property="age" column="age"></result>

        <association property="studentCard" javaType="StudentCard" select="org.jsh.mapper.studentCardMapper.queryCardByID" column="cardID">
        </association>
    </resultMap>

```

```xml
<mapper namespace="org.jsh.mapper.studentCardMapper">
    <select id="queryCardByID" parameterType="int" resultType="studentCard">
        select * from studnetCard where cardid = ${cardID}
    </select>
</mapper>
```

## 十三.二级缓存

![一级缓存](E:\Typore笔记\img\一级缓存.png)

![二级缓存](E:\Typore笔记\img\二级缓存.png)



查询缓存
	一级缓存 ：同一个SqlSession对象
	MyBatis默认开启一级缓存，如果用同样的SqlSession对象查询相同的数据，
	则只会在第一次 查询时 向数据库发送SQL语句，并将查询的结果 放入到SQLSESSION中（作为缓存在）；
	后续再次查询该同样的对象时，
	则直接从缓存中查询该对象即可（即省略了数据库的访问）	

二级缓存
	MyBatis默认情况没有开启二级缓存，需要手工打开。

1. conf.xml
   <!-- 开启二级缓存 -->
     	<setting name="cacheEnabled" value="true"/>

2. .在具体的mapper.xml中声明开启(studentMapper.xml中)
   <mapper namespace="org.lanqiao.mapper.StudentMapper">

3. <!-- 声明次namespace开启二级缓存 -->
    	<cache/>

4. MyBatis的二级缓存 是将对象 放入硬盘文件中	
   				序列化：内存->硬盘
   				反序列化：硬盘->内存

   准备缓存的对象，必须实现了序列化接口 （如果开启的缓存Namespace="org.lanqiao.mapper.StudentMapper"），可知序列化对象为Student，因此需要将Student序列化 （序列化Student类，以及Student的级联属性、和父类）

   ```java
   implements Serializable
   ```

5. 触发将对象写入二级缓存的时机：SqlSession对象的close()方法。

6. test

```java
    public static void queryStudentByStuno2() throws IOException {
        Reader reader = Resources.getResourceAsReader("conf.xml") ;
        SqlSessionFactory sessionFacotry = new SqlSessionFactoryBuilder().build(reader,"development") ;
//        第一次查询
        SqlSession session = sessionFacotry.openSession() ;
        StudentMapper studentMapper = session.getMapper(StudentMapper.class) ;
        Student student = studentMapper.selectStudentByNum(1) ;
        session.close();
//       第二次查询
        SqlSession session2 = sessionFacotry.openSession() ;
        StudentMapper studentMapper2 = session2.getMapper(StudentMapper.class) ;
        Student student2 = studentMapper2.selectStudentByNum(1) ;//接口中的方法->SQL语句
        session2.close();
        System.out.println(student.getStunum()+","+student2.getStunum());

    }
```

> -->namespace决定了studentMapper对象的产生
> 	结论：只要产生的xxxMapper对象 来自于同一个namespace，则 这些对象 共享二级缓存。
> 	注意：二级缓存 的范围是同一个namespace, 如果有多个xxMapper.xml的namespace值相同，则通过这些xxxMapper.xml产生的xxMapper对象 仍然共享二级缓存。

禁用 ：select标签中useCache="false"

```xml
    <select id="selectStudentByNum" parameterType="int" resultType="student" useCache="false">
-- sql语句
        select * from student where stunum = #{stunum}
    </select>
```

7. commit() 清理缓存 一级二级一样

### Ehcache

整合ehcache二级缓存

1. ehcache-core.jar
   mybatis-Ehcache.jar
   slf4j-api.jar
2. 编写ehcache配置文件 Ehcache.xml

```xml
<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="../config/ehcache.xsd">
  <!--当二级缓存的对象 超过内存限制时（缓存对象的个数>maxElementsInMemory），存放入的硬盘文件  -->
 <diskStore path="F:\Ehcache"/>
 <!-- 
 	maxElementsInMemory:设置 在内存中缓存 对象的个数
    maxElementsOnDisk：设置 在硬盘中缓存 对象的个数
    eternal：设置缓存是否 永远不过期
    overflowToDisk：当内存中缓存的对象个数 超过maxElementsInMemory的时候，是否转移到硬盘中
    timeToIdleSeconds：当2次访问 超过该值的时候，将缓存对象失效 
    timeToLiveSeconds：一个缓存对象 最多存放的时间（生命周期）
    diskExpiryThreadIntervalSeconds：设置每隔多长时间，通过一个线程来清理硬盘中的缓存
    memoryStoreEvictionPolicy：当超过缓存对象的最大值时，处理的策略；LRU，FIFO,LFU
  -->		     
 
 <defaultCache
  maxElementsInMemory="1000"
  maxElementsOnDisk="1000000"
  eternal="false"
  overflowToDisk="false"
  timeToIdleSeconds="100"
  timeToLiveSeconds="100"
  diskExpiryThreadIntervalSeconds="120"
  memoryStoreEvictionPolicy="LRU">
 </defaultCache>
</ehcache>
```

3. 开启EhCache二级缓存

在xxxMapper.xml中开启

```xml
<!--    开启ehcache-->
    <cache type="org.mybatis.caches.ehcache.EhcacheCache">
<!--        覆盖Ehcache.xml里的值-->
        <property name="maxElementsInMemory" value="100"/>
    </cache>
```

## 十四.逆向工程

表、类、接口、mapper.xml四者密切相关，因此 当知道一个的时候  其他三个应该可以自动生成。

表->其他三个

实现步骤：

1. mybatis-generator-core.jar、mybatis.jar、ojdbc.jar

2. 逆向工程的配置文件generator.xml

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <!DOCTYPE generatorConfiguration
     PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
     "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
   <generatorConfiguration>
      <context id="DB2Tables" targetRuntime="MyBatis3">
      <commentGenerator>
      <!--
   			suppressAllComments属性值：
   				true:自动生成实体类、SQL映射文件时没有注释
   				true:自动生成实体类、SQL映射文件，并附有注释
   		  -->
     <property name="suppressAllComments" value="true" />
    </commentGenerator>
    
    
    <!-- 数据库连接信息 -->
     <jdbcConnection driverClass="com.mysql.cj.jdbc.Driver"
      connectionURL="jdbc:mysql://localhost:3306/testdata?useSSL=true&amp;serverTimezone=UTC"
      userId="root"  password="123456">
     </jdbcConnection>
     <!-- 
   			forceBigDecimals属性值： 
   				true:把数据表中的DECIMAL和NUMERIC类型，
   解析为JAVA代码中的java.math.BigDecimal类型 
   				false(默认):把数据表中的DECIMAL和NUMERIC类型，
   解析为解析为JAVA代码中的Integer类型 
   		-->
    <javaTypeResolver>
     	<property name="forceBigDecimals" value="false" />
    </javaTypeResolver>
    <!-- 
   		targetProject属性值:实体类的生成位置  
   		targetPackage属性值：实体类所在包的路径
   	-->
    <javaModelGenerator targetPackage="org.jsh.entity"
                               targetProject=".\src">
     <!-- trimStrings属性值：
   			true：对数据库的查询结果进行trim操作
   			false(默认)：不进行trim操作       
   		  -->
     <property name="trimStrings" value="true" />
    </javaModelGenerator>
    <!-- 
   		targetProject属性值:SQL映射文件的生成位置  
   		targetPackage属性值：SQL映射文件所在包的路径
   	-->
     <sqlMapGenerator targetPackage="org.jsh.mapper"
   			targetProject=".\src">
     </sqlMapGenerator>
     <!-- 生成动态代理的接口  -->
    <javaClientGenerator type="XMLMAPPER" targetPackage="org.jsh.mapper" targetProject=".\src">
    </javaClientGenerator>
    
    <!-- 指定数据库表  -->
     <table tableName="Student"> </table>
     <table tableName="studentCard"> </table>
     <table tableName="studentClass"> </table>
    </context>
   </generatorConfiguration>
   ```

   

3. 执行

   ```java
   package org.jsh.test;
   
   import org.mybatis.generator.api.MyBatisGenerator;
   import org.mybatis.generator.config.Configuration;
   import org.mybatis.generator.config.xml.ConfigurationParser;
   import org.mybatis.generator.exception.InvalidConfigurationException;
   import org.mybatis.generator.exception.XMLParserException;
   import org.mybatis.generator.internal.DefaultShellCallback;
   
   import java.io.File;
   import java.io.IOException;
   import java.sql.SQLException;
   import java.util.ArrayList;
   import java.util.List;
   
   public class Test {
       public static void main(String[] args) throws IOException, XMLParserException, InvalidConfigurationException, SQLException, InterruptedException {
   
           File file = new File("src/generator.xml") ;//配置文件
   
           List<String> warnings = new ArrayList<>();
           ConfigurationParser cp = new ConfigurationParser(warnings);
   
           Configuration config = cp.parseConfiguration(file);
   
   
           DefaultShellCallback callBack = new DefaultShellCallback(true);
   
           //逆向工程的核心类
           MyBatisGenerator generator = new MyBatisGenerator(config, callBack,warnings  );
           generator.generate(null);
       }
   }
   
   ```

   

