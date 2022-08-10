# 后端-Mybatis-日志(Log4j)和延迟加载(day8)


# 后端-Mybatis-日志(Log4j)和延迟加载(day8)

## Log4j.jar

### 开启日志

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

## 延迟加载

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


