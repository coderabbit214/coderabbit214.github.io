# SpringBoot-Mybatis


# SpringBoot-Mybatis

## maven依赖

> jdbc mybatis mysql

```xml
 <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.1.2</version>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
    </dependencies>
```



## 注解方式

```java
package com.jsh.springbootmybatis.mapper;

import com.jsh.springbootmybatis.entity.Student;
import org.apache.ibatis.annotations.*;

@Mapper
public interface StudentMapper {
    @Select("select * from student where stunum = #{stunum}")
    public Student getStudentByStunum(Integer stunum);
    @Delete("delete from student where stunum = #{stunum}")
    public int delStudentByStunum(Integer stunum);
    @Insert("insert into student(stunum,name,age,sex,classid) values(#{stunum},#{name},#{age},#{sex},#{classID})")
    public int insertStudent(Student student);
    @Update("update Student set name = #{name} where stunum = #{stunum}")
    public int updateStudent(Student student);
}

```

## 配置方式

> yml 配置

```
mybatis:
  # 指定全局配置文件位置
  config-location: classpath:mybatis/mybatis-config.xml
  # 指定sql映射文件位置
  mapper-locations: classpath:mybatis/mapper/*.xml
```

```java
package com.jsh.springbootmybatis.mapper;

import com.jsh.springbootmybatis.entity.Student;
import org.apache.ibatis.annotations.Mapper;

@Mapper
public interface StudentMapperByConfig {
    public Student getStudentByStunum(Integer stunum);
}
```

> mapper

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.jsh.springbootmybatis.mapper.StudentMapperByConfig">
    <select id="getStudentByStunum" resultType="com.jsh.springbootmybatis.entity.Student">
    select * from student where stunum = #{stunum}
  </select>
</mapper>
```

## 其他

> mybatis-config.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!--    配置驼峰命名-->
    <settings>
        <setting name="mapUnderscoreToCamelCase" value="true"/>
    </settings>
</configuration>
```

> 扫描所有接口

```java
使用MapperScan批量扫描所有的Mapper接口；
@MapperScan(value = "com.atguigu.springboot.mapper")
@SpringBootApplication
public class DomeApplication {

	public static void main(String[] args) {
		SpringApplication.run(DomeApplication.class, args);
	}
}
```


