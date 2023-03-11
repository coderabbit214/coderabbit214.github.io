# Neo4j实践


# Neo4j实践

## 关于

### 是什么

图形数据库管理系统，是一个高性能、面向对象的数据库系统，用于存储、管理和查询大规模的图形数据。它是一个开源的、ACID兼容的数据库，采用了图形模型来表示数据，这使得它在处理复杂的关系数据方面表现出色。Neo4j是目前最受欢迎的图形数据库之一，它可以被用于许多应用程序领域，包括社交网络、推荐系统、网络安全、生物信息学等。它支持多种编程语言和平台，并且具有良好的可扩展性和高可用性。

### 为什么使用

1. 更适合处理复杂关系型数据：Neo4j采用图形模型来表示数据，可以很好地处理复杂的关系型数据。与传统的关系型数据库相比，Neo4j更加灵活和高效。
2. 更快的查询速度：Neo4j是一个面向对象的数据库系统，它使用了索引和缓存技术来提高查询性能。在查询图形数据时，Neo4j可以使用遍历算法来优化查询性能，从而实现更快的查询速度。
3. 更好的可扩展性：Neo4j可以轻松地扩展到多个节点，可以实现分布式计算，因此可以处理更大的数据集。
4. 更适合处理实时数据：Neo4j是一个内存数据库，支持实时数据处理，适用于实时应用程序和实时数据分析。
5. 更容易理解和使用：Neo4j使用基于图形模型的查询语言Cypher，这种查询语言与现代应用程序的数据结构更加相似，因此更容易理解和使用。

综上所述，如果你需要处理复杂关系型数据，需要更快的查询速度、更好的可扩展性和实时数据处理能力，Neo4j可能是更好的选择。而如果你需要处理传统的关系型数据，或者需要使用SQL查询语言，那么传统的关系型数据库可能更适合你的需求。

## 实践

### 安装

```yaml
version: '3'
services:
  neo4j:
    image: neo4j:4.3.7
    volumes:
      - $PWD/data:/data #把容器内的数据目录挂载到宿主机的对应目录下
      - $PWD/logs:/logs #挂载日志目录
      - $PWD/conf:/var/lib/neo4j/conf #挂载配置目录
      - $PWD/import:/var/lib/neo4j/import #挂载数据导入目录
      - $PWD/plugins:/plugins
    environment:
      - NEO4J_dbms_memory_heap_maxSize=4G
      - NEO4J_AUTH=neo4j/123456  #修改默认用户密码
    ports:
      - "7474:7474" #映射容器的端口号到宿主机的端口号
      - "7687:7687"
    restart: always

```

### Cypher

#### 简介

| 用法             |                              |
| :--------------- | ---------------------------- |
| CREATE 创建      | 创建节点，关系和属性         |
| MATCH 匹配       | 检索有关节点，关系和属性数据 |
| RETURN 返回      | 返回查询结果                 |
| WHERE 哪里       | 提供条件过滤检索数据         |
| DELETE 删除      | 删除节点和关系               |
| REMOVE 移除      | 删除节点和关系的属性         |
| ORDER BY 以…排序 | 排序检索数据                 |
| SET              | 添加或更新标签 设置属性      |

#### LOAD

```CQL
load cv from "file:///xxx.csv" as line
create (:xiyouRelation (from: line[1],relation:line[3],to: line[0]})
```

![截屏2023-03-11 11.01.53](https://images-jsh.oss-cn-beijing.aliyuncs.com/work/2023/03/11/20230311-110158.png)

#### MATCH

```cql
// 查询某个标签全部节点
match (n:xiyouRelation) return n

// 查询所有数量
match (n:xiyouRelation) return count(n)

// 条件查询 in,is null
match (n:xiyouRelation {from:"孙悟空"}) return n
match (n:xiyouRelation) where n.from in["孙悟空","猪八戒"]  return n

//关系查询 [:西游关系*n] n确定层级 0只返回自己
MATCH (n:person {name:'孙悟空'})-[:西游关系*2]->(b:person) return n,b
MATCH (n:person {name:'孙悟空'})<-[:西游关系*2]-(b:person) return n,b

// 查询其他参数
// ORDER BY
MATCH (n:person {name:'孙悟空'})-[:西游关系*2]->(b:person) return n,b ORDER BY n.name
// LIMIT
MATCH (n:person {name:'孙悟空'})-[:西游关系*2]->(b:person) return n,b LIMIT 10
// SKIP
MATCH (n:person {name:'孙悟空'})-[:西游关系*2]->(b:person) return n,b SKIP 10
```

#### CREATE

```cql
// 创建一个标签为person的节点，节点有一个name属性，属性值为'小红'
create (n:person {name:"小红"}) return n

// 属性设置
match (n:person {name:"小红"}) set n.age=18 return n

// 给某个标签对外的所有关系增加属性
MATCH p=(person {name:'孙悟空'})-[r:西游关系]->() SET r={since:"2017-01-02"}  RETURN p; 
```

#### DELETE

```cql
// 删除a节点
match (n:person {name:"小红"}) delete n

//删除关系属性 
match (n:person {name:'孙悟空'})-[r:西游关系]->(m:person) REMOVE  r.since
```

### Springboot 集成

> springboot版本 3.0.4

1. 引入依赖

   ```gradle
   implementation 'org.springframework.boot:spring-boot-starter-data-neo4j'
   ```

2. 配置

   ```yaml
   spring:
     neo4j:
       uri: bolt://localhost:7687
       authentication:
         username: neo4j
         password: 123456
     data:
       neo4j:
         database: neo4j
   
   logging:
     level:
       org:
         springframework:
           data:
             neo4j: debug
   ```

3. 创建实体

   ```java
   @Data
   @Node("person")
   public class Person {
       @Id
       @GeneratedValue
       private Long id;
   
       @Property
       private String name;
   }
   ```

4. Repository

   ```java
   @Repository
   public interface PersonRepository extends Neo4jRepository<Person, Long> {
         /**
        * 根据名称查询
        * @param name
        * @return
        */
       Person findByName(String name);
   
       /**
        * 创建关系
        *
        * @param from
        * @param relation
        * @param to
        */
       @Query("MATCH (n:person {name:$from}),(m:person {name:$to}) CREATE (n)-[:西游关系{relation:$relation}]->(m)")
       void createRelation(String from, String relation, String to);
   }
   
   ```

5. 测试

   ```java
   @Test
       public void testFindAll() {
           List<Person> all = personRepository.findAll();
           for (Person person : all) {
               System.out.println(person);
           }
       }
   
       @Test
       public void testFindById() {
           Optional<Person> person = personRepository.findById(295L);
           person.ifPresent(System.out::println);
       }
   
       @Test
       public void testDelete() {
           personRepository.deleteById(295L);
       }
   
       @Test
       public void testCreateObject() {
           Person person = new Person();
           person.setName("牛魔王");
           Person save = personRepository.save(person);
           System.out.println(save);
       }
   ```

#### 关系

##### 简单关系

> 在自身中存储

```java
@Data
@Node("person")
public class Person {
    @Id
    @GeneratedValue
    private Long id;

    @Property
    private String name;

    @Relationship(type = "师傅")
    public Set<Person> masters;

    @Relationship(type = "师兄")
    public Set<Person> brothers;

    //指定师傅关系
    public void masters(Person person) {
        if (masters == null) {
            masters = new HashSet<>();
        }
        masters.add(person);
    }

    public void brothers(Person person) {
        if (brothers == null) {
            brothers = new HashSet<>();
        }
        brothers.add(person);
    }


    public String toString() {
        return this.name + " 师傅 => "
                + Optional.ofNullable(this.masters).orElse(
                Collections.emptySet()).stream().map(
                Person::getName).collect(Collectors.toList())
                + " 师兄 => "
                + Optional.ofNullable(this.brothers).orElse(
                Collections.emptySet()).stream().map(
                Person::getName).collect(Collectors.toList());
    }


}
```

test

```java
    @Test
    public void testCreateRelation() {
        Person san = new Person();
        san.setName("张三");

        Person si = new Person();
        si.setName("李四");

        Person wu = new Person();
        wu.setName("王武");

        san.masters(si);
        san.brothers(wu);
        personRepository.save(san);

        // 张三 师傅 => [李四] 师兄 => [王武]
        Person name = personRepository.findByName("张三");
        System.out.println(name);
    }
```

##### 复杂关系(关系中有一些其他属性)

> 创建关系类
>
> @RelationshipProperties : 指定是一个关系类
>
> @TargetNode : 指定关联对象

```java
@RelationshipProperties
@Data
public class XiYouRelationPerson {

    @RelationshipId
    private Long id;

    private String relation;

    @TargetNode
    private Person property;
}

```

使用

```java
@Data
@Node("person")
public class Person {
    @Id
    @GeneratedValue
    private Long id;

    @Property
    private String name;

    @Relationship(type = "西游关系")
    public List<XiYouRelationPerson> xiYouRelationPeople;

}
```

