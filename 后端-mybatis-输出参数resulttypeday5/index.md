# 后端-Mybatis-输出参数resultType(day5)


# 后端-Mybatis-输出参数resultType(day5)

## 八个简单类型+String

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

## 对象类型

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

## 实体对象的集合类型

> resultType不变 函数返回值变为列表

## HashMap类型（起别名）

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

## resultMap(实体类的属性和数据表的字段：类型，名字不同时)

//resultMap="queryStudentByID"   id="queryStudentByID"对应

property 实体类属性名（严格大小写）

column 数据表属性名

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


