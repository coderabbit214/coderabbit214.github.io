# 关联查询


# 关联查询

准备 

1. 建立StudentCard表（cardid，cardinfo）
2. 建立外键连接

## 一对一

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

## 一对多

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


