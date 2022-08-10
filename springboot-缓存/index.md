# Springboot-缓存


# Springboot-缓存

## 一.搭建基本环境

### 1.导入数据库文件 创建出 Department Employee 表

```sql
SET FOREIGN_KEY_CHECKS=0;

-- ----------------------------
-- Table structure for department
-- ----------------------------
DROP TABLE IF EXISTS `department`;
CREATE TABLE `department` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `departmentName` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

-- ----------------------------
-- Table structure for employee
-- ----------------------------
DROP TABLE IF EXISTS `employee`;
CREATE TABLE `employee` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `lastName` varchar(255) DEFAULT NULL,
  `email` varchar(255) DEFAULT NULL,
  `gender` int(2) DEFAULT NULL,
  `d_id` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

### 2.封装数据 创建实体类

Department

```java
package com.jsh.cache.bean;

public class Department {
	
	private Integer id;
	private String departmentName;
	
	
	public Department() {
		super();
		// TODO Auto-generated constructor stub
	}
	public Department(Integer id, String departmentName) {
		super();
		this.id = id;
		this.departmentName = departmentName;
	}
	public Integer getId() {
		return id;
	}
	public void setId(Integer id) {
		this.id = id;
	}
	public String getDepartmentName() {
		return departmentName;
	}
	public void setDepartmentName(String departmentName) {
		this.departmentName = departmentName;
	}
	@Override
	public String toString() {
		return "Department [id=" + id + ", departmentName=" + departmentName + "]";
	}
		
}

```

Employee

```java
package com.jsh.cache.bean;

public class Employee {
	
	private Integer id;
	private String lastName;
	private String email;
	private Integer gender; //性别 1男  0女
	private Integer dId;
	
	
	public Employee() {
		super();
	}

	
	public Employee(Integer id, String lastName, String email, Integer gender, Integer dId) {
		super();
		this.id = id;
		this.lastName = lastName;
		this.email = email;
		this.gender = gender;
		this.dId = dId;
	}
	
	public Integer getId() {
		return id;
	}
	public void setId(Integer id) {
		this.id = id;
	}
	public String getLastName() {
		return lastName;
	}
	public void setLastName(String lastName) {
		this.lastName = lastName;
	}
	public String getEmail() {
		return email;
	}
	public void setEmail(String email) {
		this.email = email;
	}
	public Integer getGender() {
		return gender;
	}
	public void setGender(Integer gender) {
		this.gender = gender;
	}
	public Integer getdId() {
		return dId;
	}
	public void setdId(Integer dId) {
		this.dId = dId;
	}
	@Override
	public String toString() {
		return "Employee [id=" + id + ", lastName=" + lastName + ", email=" + email + ", gender=" + gender + ", dId="
				+ dId + "]";
	}
}

```

### 3.整合Mybatis

#### (1)配置数据源

```properties
spring.datasource.url=jdbc:mysql://localhost/spring_cache
spring.datasource.password=123456
spring.datasource.username=root
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
```

#### (2)使用注解版mybatis

```java
public interface EmployeeMapper {
    @Select("SELECT * FROM employee WHERE id = #{id}")
    public Employee getEmpById(Integer id);

    @Update("UPDATE employee SET lastName=#{lastName},email=#{email},gender=#{gender},d_id=#{dId} WHERE id=#{id}")
    public void updateEmp(Employee employee);

    @Delete("DELETE FROM employee WHERE id=#{id}")
    public void deleteEmpById(Integer id);

    @Insert("INSERT INTO employee(lastName,email,gender,d_id) VALUES(#{lastName},#{email},#{gender},#{dId})")
    public void insertEmployee(Employee employee);

    @Select("SELECT * FROM employee WHERE lastName = #{lastName}")
    Employee getEmpByLastName(String lastName);
}

```

#### (3)开启驼峰命名法

```properties
#开启驼峰命名
mybatis.configuration.map-underscore-to-camel-case=true
```

## 二.快速体验缓存

### 1.开启基于缓存的注解

主程序标注@EnableCaching

```java
@MapperScan("com.jsh.cache.mapper")
@SpringBootApplication
@EnableCaching
public class SpringBootCacheApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringBootCacheApplication.class, args);
    }

}
```

### 2.Service层方法标注缓存注解

#### (1)@Cacheable

> @Cacheable 对方法结果进行缓存 方法调用之前运行
> ```
> @Cacheable 将方法的运行结果进行缓存；以后再要相同的数据，直接从缓存中获取，不用调用方法
>  CacheManager管理多个Cache组件，对缓存的真正CRUD操作在Cache组件中，每一个缓存组件有自己唯一的名字
>  几个属性
>       1.cacheNames/value:指定缓存组件的名字
>       2.key:缓存数据使用的key；可以用它来指定。默认使用方法参数的值 --- 方法的返回值
>               编写SqEL: #id:参数id的值 #a0 #p0 #root.args[0]
>       3.keyGenerator:key的生成器；可以自己指定key的生成器的组件id
>           key/keyGenerator二选一使用
>       4.cacheManager:缓存管理器 或者使用cacheResolver指定缓存管理器
>       5.condition：指定符合条件的情况下才缓存
>           condition = "#id>0"
>       6.unless:否定缓存；当unless指定条件为true，方法的返回值就不会被缓存；可以获取结果进行缓存
>           unless = "#result == null"
>       7.sync:是否使用异步模式
> ```

```java
package com.jsh.cache.service;

@Service
public class EmployeeService {
    @Autowired
    EmployeeMapper employeeMapper;
    @Cacheable(cacheNames = {"emp"})
    public Employee getEmpById(Integer id){
        System.out.println("查询"+id+"号员工");
        return employeeMapper.getEmpById(id);
    }
}

```

* keyGenerator的使用

  1. 创建生成器

     ```java
     package com.jsh.cache.config;
     
     @Configuration
     public class MyCacheConfig {
     
         @Bean("myKeyGenerator")
         public KeyGenerator keyGenerator(){
             return new KeyGenerator(){
     
                 @Override
                 public Object generate(Object target, Method method, Object... params) {
                     return method.getNjavaame()+"["+ Arrays.asList(params).toString()+"]";
                 }
             };
         }
     }
     
     ```

  2. 使用

     ```java
         @Cacheable(cacheNames = {"emp"},keyGenerator = "myKeyGenerator")
         public Employee getEmpById(Integer id){
             System.out.println("查询"+id+"号员工");
             return employeeMapper.getEmpById(id);
         }
     ```

#### (2)@CachePut

> ```
> @CachePut:即调用方法，又更新缓存数据
>  使用场景：修改数据库的某个数据，同时更新缓存
>  方法调用之后运行
>       1.先调用目标方法
>       2.将目标方法的结果缓存起来
>  测试步骤：
>   1、查询1号员工；查到的结果会放在缓存中；
>           key：1  value：lastName：张三
>   2、以后查询还是之前的结果
>   3、更新1号员工；【lastName:zhangsan；gender:0】
>           将方法的返回值也放进缓存了；
>           key：传入的employee对象  值：返回的employee对象；
>   4、查询1号员工？
> 
>       为什么是没更新前的？【1号员工没有在缓存中更新】
> 
>       应该是更新后的员工；
>               解决
>            key = "#employee.id":使用传入的参数的员工id；
>                 key = "#result.id"：使用返回后的id
>                    @Cacheable的key是不能用#result
> ```

```java
    @CachePut(cacheNames = "emp",key = "#a0.id")
    public Employee upadteEmp(Employee employee){
        System.out.println("员工更新了");
        employeeMapper.updateEmp(employee);
        return employee;
    }
```

#### (3)@CacheEvict

> ```
> @CacheEvict:清除缓存
>  allEntries = true 删除所有emp中的缓存 默认false
>  beforeInvocation = true 缓存的清除是否在方法之前执行 默认false
>       false时 如果方法出现异常则不会清除异常
> ```

```
    @CacheEvict(cacheNames = "emp",key = "#id"/*,allEntries = true*/,beforeInvocation = true)
    public void deleteEmp(Integer id){
        System.out.println("员工删除了");
    }
```

#### (4)@Caching

> 组合@CacheEvict @CachePut @Cacheable

```java
 @Caching(
            cacheable = {
                    @Cacheable(cacheNames = "emp",key = "#a0")
            },
            put = {
                    @CachePut(cacheNames = "emp",key = "#result.id"),
                    @CachePut(cacheNames = "emp",key = "#result.email")
            }
    )
    public Employee getEmpByLastName(String lastName){
        System.out.println(2);
        return employeeMapper.getEmpByLastName(lastName);
    }
```

#### (5)@CacheConfig

> 在类上使用 用于整体设置

```java
@Service
@CacheConfig(cacheNames = "emp")
public class EmployeeService {......
```

## 三.Springboot-Redis

### 1.配置

pom.xml

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
```

application.properties

```
spring.redis.host=127.0.0.1
```

### 2.测试

> 实体类 需要序列化
>
> 将数据以JSON存储
>  redisTemplate.setKeySerializer(new StringRedisSerializer());
>
> redisTemplate.opsForXXX来操纵五种数据

```java
 @Autowired
    StringRedisTemplate stringRedisTemplate; //操作字符串
    @Autowired
    RedisTemplate redisTemplate; //K-V
    @Test
    public void testRedis(){
//        stringRedisTemplate.opsForValue().set("a","a");
//        测试存储对象
        Employee employee = employeeMapper.getEmpById(1);
//        将数据以JSON存储
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        redisTemplate.opsForValue().set("emp-01",employee);

        Employee e = (Employee) redisTemplate.opsForValue().get("emp-01");
        System.out.println(e);
    }
```

### 3.使用

> 当配置好redis时 默认使用Redis

