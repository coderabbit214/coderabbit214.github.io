# 后端-Mybatis-二级缓存(day9)


# 后端-Mybatis-二级缓存(day9)

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
   	 开启二级缓存
      	<setting name="cacheEnabled" value="true"/>

2. .在具体的mapper.xml中声明开启(studentMapper.xml中)
   		<mapper namespace="org.lanqiao.mapper.StudentMapper">

3. 	 声明次namespace开启二级缓存
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

## Ehcache

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


