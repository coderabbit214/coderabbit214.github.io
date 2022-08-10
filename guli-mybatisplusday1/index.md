# Mybatis-Plus


# Mybatis-Plus

## 一.配置

- pom.xml

  ```xml
  <!--        mybatisPlus-->
          <dependency>
              <groupId>com.baomidou</groupId>
              <artifactId>mybatis-plus-boot-starter</artifactId>
              <version>3.0.5</version>
          </dependency>
  <!--        lombok用来简化实例 需要安装lombok插件-->
          <dependency>
              <groupId>org.projectlombok</groupId>
              <artifactId>lombok</artifactId>
              <version>1.18.12</version>
          </dependency>
  ```

- application.properties

  ```properties
  #数据库配置
  spring.datasource.url=jdbc:mysql://localhost:3306/testdata
  spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
  spring.datasource.username=root
  spring.datasource.password=123456
  ```

## 二.编写代码

### 1.实体类

> 使用Lombok简化代码

```java
@Data
public class User {
private Long id;
private String name;
private Integer age;
private String email;
}
```

### 2.数据库

```sql
CREATE TABLE USER
(
 id BIGINT(20) NOT NULL COMMENT '主键ID',
 NAME VARCHAR(30) NULL DEFAULT NULL COMMENT '姓名',
 age INT(11) NULL DEFAULT NULL COMMENT '年龄',
 email VARCHAR(50) NULL DEFAULT NULL COMMENT '邮箱',
 PRIMARY KEY (id)
);

INSERT INTO USER (id, NAME, age, email) VALUES
(1, 'Jone', 18, 'test1@baomidou.com'),
(2, 'Jack', 20, 'test2@baomidou.com'),
(3, 'Tom', 28, 'test3@baomidou.com'),
(4, 'Sandy', 21, 'test4@baomidou.com'),
(5, 'Billie', 24, 'test5@baomidou.com');
```

### 3.mapper

> 需要继承：BaseMapper<实体>

```java
package com.jsh.mybatisplus.mapper;

import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.jsh.mybatisplus.entity.User;
import org.springframework.stereotype.Repository;

@Repository
public interface UserMapper extends BaseMapper<User> {
}
```

## 三.整体测试代码

```java
package com.jsh.mybatisplus;

import com.baomidou.mybatisplus.core.conditions.query.QueryWrapper;
import com.baomidou.mybatisplus.extension.plugins.pagination.Page;
import com.jsh.mybatisplus.entity.User;
import com.jsh.mybatisplus.mapper.UserMapper;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import java.util.Arrays;
import java.util.HashMap;
import java.util.List;

@SpringBootTest
public class MybatisplusApplicationTests {

//  测试mybatisPlus
    @Autowired
    private UserMapper userMapper;
//  查询user表所有数据
    @Test
    public void testSelectList() {
        List<User> userList = userMapper.selectList(null);
        System.out.println(userList);
    }
//  添加操作
    @Test
    public void testInsert(){
        User user = new User();
        user.setName("GG1");
        user.setAge(20);
        user.setEmail("1157237955@qq.cmm");
        int insert = userMapper.insert(user);
        System.out.println(insert);
    }
//  修改操作
    @Test
    public void testUpdate() {
        User user = new User();
        user.setId(1288368399486828545L);
        user.setName("P");
        userMapper.updateById(user);
    }
//  测试乐观锁
    @Test
    public void OptimisticLocker() {
        //根据id查询数据
        User user = userMapper.selectById(1288368399486828545L);
        user.setName("P");
        userMapper.updateById(user);
    }
//  多个id批量查询
    @Test
    public void testSelectDemo1() {
        List<User> userList = userMapper.selectBatchIds(Arrays.asList(1L,2L,3L));
        System.out.println(userList);
    }
//  根据条件查询(map)
    @Test
    public void testDeleteByMap() {
       HashMap<String, Object> map = new HashMap<>();
       map.put("name", "Helen");
       map.put("age", 18);
       List<User> userList = userMapper.selectByMap(map);
       System.out.println(userList);
    }
//  分页查询
    @Test
    public void testSelectPage() {
        //1.创建page对象
        //两个参数：当前页，每页显示数据
        Page<User> page = new Page<>(1,3);
        //调用mp分页查询方法:底层封装（把分页的所有数据封装到page中）
        userMapper.selectPage(page,null);
        //通过page对象获取分页数据
        System.out.println(page.getCurrent());//当前页
        System.out.println(page.getRecords());//每页数据的list集合
        System.out.println(page.getSize());//每页显示记录数
        System.out.println(page.getTotal());//总记录数
        System.out.println(page.getPages());//总页数

        System.out.println(page.hasNext());//是否有下一页
        System.out.println(page.hasPrevious());//是否有上一页
    }
    //删除操作 物理删除
    @Test
    public void testDelete() {
        int result = userMapper.deleteById(1288412637062803457L);
        System.out.println(result);
    }
    //批量删除
    @Test
    public void testDeleteBatchIds() {
        int result = userMapper.deleteBatchIds(Arrays.asList(2,1,3));
        System.out.println(result);
    }
    //复杂条件查询 构建条件
    @Test
    public void testQueryWapper(){
        //创建QueryWrapper对象
        QueryWrapper<User> queryWrapper = new QueryWrapper<>();
        //通过QueryWrapper设置条件
        //ge(大于等于),gt(大于),le(小于等于),lt(小于)
//        queryWrapper.gt("age",20);

        //eq(等于),ne(不等于)
//        queryWrapper.eq("name","GG");

        //between(查询区间 包括临界值)
//        queryWrapper.between("age",20,30);

        //like(模糊查询)
//        queryWrapper.like("name","G");

        //orderByDesc(排序)
//        queryWrapper.orderByDesc("id");

        //last(在sql语句后拼接)
//        queryWrapper.last("limit 1");

        //select(指定要查询的列)
        queryWrapper.select("id","name");

        userMapper.selectList(queryWrapper);
    }
}

```

## 四.配置日志

- application.properties

  ```properties
  #mybatis日志
  mybatis.configuration.log-impl=org.apache.ibatis.logging.stdout.StdOutImpl
  ```

## 五.主键策略

```java
//  主键生成策略
    /**
     * AUTO：自动生长
     * INPUT：手动输入设置
     * NONE：没有策略
     * UUID：随即唯一的值
     *
     * mybatisPlus自带策略：（不写会默认识别）
     *    ID_WORKER：生成19位值，数字类型使用这种策略，例如long
     *    ID_WORKER_STR：生成19位值，字符串类型使用这种策略
     */
    @TableId(type = IdType.ID_WORKER)
    private Long id;
```

## 六.自动填充

1. 数据库表中添加自动填充字段 在User表中添加datetime类型的新的字段 create_time、update_time

2. 实体上添加注解

   ```java
   //creat_time 添加时 INSERT
       @TableField(fill = FieldFill.INSERT)
       private Date createTime;
       //update_time 添加和修改时 INSERT_UPDATE
       @TableField(fill = FieldFill.INSERT_UPDATE)
       private Date updateTime;
   ```

3. 实现元对象处理器接口(不要忘记添加 @Component 注解)

   ```java
   package com.jsh.mybatisplus.handler;
   
   import com.baomidou.mybatisplus.core.handlers.MetaObjectHandler;
   import org.apache.ibatis.reflection.MetaObject;
   import org.springframework.stereotype.Component;
   
   import java.util.Date;
   
   @Component
   public class MyMetaObjectHandler implements MetaObjectHandler {
   
       @Override
       public void insertFill(MetaObject metaObject) {
           //三个参数：修改的参数名，修改的值，metaObject
           this.setFieldValByName("createTime",new Date(),metaObject);
           this.setFieldValByName("updateTime",new Date(),metaObject);
       }
   
       @Override
       public void updateFill(MetaObject metaObject) {
           this.setFieldValByName("updateTime",new Date(),metaObject);
       }
   }
   
   ```

## 七.乐观锁

> 主要适用场景：当要更新一条记录的时候，希望这条记录没有被别人更新，也就是说实现线程安全的数据更新 
>
> 乐观锁实现方式： 
>
> 1. 取出记录时，获取当前version 
>
> 2. 更新时，带上这个version 执行更新时， set version = newVersion where version = oldVersion 
>
> 3. 如果version不对，就更新失败

1. 数据库中添加version字段

2. 实体类添加version字段 并添加 @Version 注解

   ```java
    //版本号
       @Version
       @TableField(fill = FieldFill.INSERT)
       private Integer version;
   ```

3. 元对象处理器接口添加version的insert默认值

   ```
   @Override
   public void insertFill(MetaObject metaObject) {
    ......
   this.setFieldValByName("version", 1, metaObject);
   }
   ```

4. 在MpConfig(用于配置)中注册 Bean

   ```java
   @Configuration
   @MapperScan("com.jsh.mybatisplus.mapper")
   public class MpConfig {
       //乐观锁插件
       @Bean
       public OptimisticLockerInterceptor optimisticLockerInterceptor(){
           return new OptimisticLockerInterceptor();
       }
   }
   ```

5. 测试

   > 测试后分析打印的sql语句，将version的数值进行了加1操作

   ```java
   //  测试乐观锁
       @Test
       public void OptimisticLocker() {
           //根据id查询数据
           User user = userMapper.selectById(1288368399486828545L);
           user.setName("P");
           userMapper.updateById(user);
       }
   ```

## 八.分页查询

> MyBatis Plus自带分页插件，只要简单的配置即可实现分页功能

1. 配置分页插件

   ```java
   @Configuration
   @MapperScan("com.jsh.mybatisplus.mapper")
   public class MpConfig {
       //分页插件
       @Bean
       public PaginationInterceptor paginationInterceptor() {
           return new PaginationInterceptor();
       }
   
   }
   
   ```

2. 测试

   > 最终通过page对象获取相关数据

   ```java
   //  分页查询
       @Test
       public void testSelectPage() {
           //1.创建page对象
           //两个参数：当前页，每页显示数据
           Page<User> page = new Page<>(1,3);
           //调用mp分页查询方法:底层封装（把分页的所有数据封装到page中）
           userMapper.selectPage(page,null);
           //通过page对象获取分页数据
           System.out.println(page.getCurrent());//当前页
           System.out.println(page.getRecords());//每页数据的list集合
           System.out.println(page.getSize());//每页显示记录数
           System.out.println(page.getTotal());//总记录数
           System.out.println(page.getPages());//总页数
   
           System.out.println(page.hasNext());//是否有下一页
           System.out.println(page.hasPrevious());//是否有上一页
       }
   ```

## 九.逻辑删除

> 物理删除：真实删除，将对应数据从数据库中删除，之后查询不到此条被删除数据 
>
> 逻辑删除：假删除，将对应数据中代表是否被删除字段状态修改为“被删除状态”，之后在数据库中仍 旧能看到此条数据记录

1. 数据库中添加 deleted字段

2. 实体类添加deleted 字段

   ```java
   //逻辑删除标识符
       @TableLogic
   	@TableField(fill = FieldFill.INSERT)
       private Integer deleted;
   ```

3. 元对象处理器接口添加deleted的insert默认值

   ```
   public void insertFill(MetaObject metaObject) {
    ......
   this.setFieldValByName("deleted", 0, metaObject);
   }
   ```

4. application.properties

   ```
   #mybatis逻辑删除标志定义 默认是0和1
   mybatis-plus.global-config.db-config.logic-delete-value=1
   mybatis-plus.global-config.db-config.logic-not-delete-value=0
   ```

5. 在 MyConfig 中注册 Bean

   ```
   @Configuration
   @MapperScan("com.jsh.mybatisplus.mapper")
   public class MpConfig {
       //逻辑删除插件 mybatis-Plus版本3.3.0以后不需要
       @Bean
       public ISqlInjector iSqlInjector() {
           return new LogicSqlInjector();
       }
   }
   
   ```

6. 测试

   > 测试后发现，数据并没有被删除，deleted字段的值由0变成了1 
   >
   > 测试后分析打印的sql语句，是一条update
   >
   >  注意：被删除数据的deleted 字段的值必须是 0，才能被选取出来执行逻辑删除的操作

   ```java
   @Test
       public void testDelete() {
           int result = userMapper.deleteById(1288412637062803457L);
           System.out.println(result);
       }
   ```

7. 测试逻辑删除后的查询

   > MyBatis Plus中查询操作也会自动添加逻辑删除字段的判断
   >
   > 测试后分析打印的sql语句，包含 WHERE deleted=0 
   >
   > SELECT id,name,age,email,create_time,update_time,deleted FROM user WHERE deleted=0

   ```java
   /**
   * 测试 逻辑删除后的查询：
   * 不包括被逻辑删除的记录
   */
   @Test
   public void testLogicDeleteSelect() {
   User user = new User();
   List<User> users = userMapper.selectList(null);
   users.forEach(System.out::println);
   }
   ```

## 十.性能分析

1. 在 MyConfig 中注册 Bean

   ```java
   @Configuration
   @MapperScan("com.jsh.mybatisplus.mapper")
   public class MpConfig {
       /**
        * sql执行性能分析插件
        * 开发环境使用，线上不推荐，maxTime:最大执行时常
        *
        * 三种环境
        * dev:开发环境
        * test:测试环境
        * prod:生产环境
        */
       @Bean
       @Profile({"dev","test"})//设置 dev test 环境开启
       public PerformanceInterceptor performanceInterceptor() {
           PerformanceInterceptor performanceInterceptor = new PerformanceInterceptor();
           performanceInterceptor.setMaxTime(500);//ms,超过500ms sql语句不执行
           performanceInterceptor.setFormat(true);//格式化日志
           return performanceInterceptor;
       }
   }
   ```

2. Spring Boot 中设置dev环境

   ```properties
   #环境设置
   spring.profiles.active=dev
   ```

## 十一.条件查询(QueryWrapper)

```
@SpringBootTest
public class MybatisplusApplicationTests {

//  测试mybatisPlus
    @Autowired
    private UserMapper userMapper;
    //复杂条件查询 构建条件
    @Test
    public void testQueryWapper(){
        //创建QueryWrapper对象
        QueryWrapper<User> queryWrapper = new QueryWrapper<>();
        //通过QueryWrapper设置条件
        //ge(大于等于),gt(大于),le(小于等于),lt(小于)
//        queryWrapper.gt("age",20);

        //eq(等于),ne(不等于)
//        queryWrapper.eq("name","GG");

        //between(查询区间 包括临界值)
//        queryWrapper.between("age",20,30);

        //like(模糊查询)
//        queryWrapper.like("name","G");

        //orderByDesc(排序)
//        queryWrapper.orderByDesc("id");

        //last(在sql语句后拼接)
//        queryWrapper.last("limit 1");

        //select(指定要查询的列)
        queryWrapper.select("id","name");

        userMapper.selectList(queryWrapper);
    }
}

```






