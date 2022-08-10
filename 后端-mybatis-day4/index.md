# 后端-Mybatis-(day4)


# 后端-Mybatis-(day4)

## parameterType 输入对象为HashMap

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

## 通过调用【存储过程】 实现查询

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


