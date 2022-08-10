# 后端-Mybatis-day6


# 后端-Mybatis-day6

## 动态SQL

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

#### 对象属性

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


