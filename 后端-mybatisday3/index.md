# 后端-Mybatis-${}和{}(day3)


# 后端-Mybatis-${}和#{}(day3)

 ## 输入参数：parameterType

1. 类型为 简单类型（8个基本类型+String)

   - #{任意值} 

     - 自动给String类型加上 ' '
     - 可以防止SQL注入

   - ${value}

     - 其中标识符只能是value 原样输出 
     - 如果是String 类型 则需要在外加上单引号  '${value}'
     - 适合动态排序（动态字段）

     ```xml
      <select id="queryAllStudentsOrderByColumn" parameterType="String" resultType="student">
             select * from student order  by ${value} asc
         </select>
     ```

     

     - 不防止SQL注入

2. 类型为对象：

   - #{}
     
  - #{对象属性名} 或者使用parameterMap
    
     ```xml
       <select id="queryStudentsOrderByColumn" parameterType="student" resultType="student">
             select * from student where stunum = #{stunum} or name like #{name}
         </select>
     ```
  ```
     
  
     
  - 模糊查询需要在传递参数时加上 % 
     
     ```java
     public static void queryStudentsOrderByColumn() throws IOException {
     
             Reader reader = Resources.getResourceAsReader("conf.xml");
             SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(reader, "development");
             SqlSession session = sessionFactory.openSession();
             Student student = new Student(3, "%j%", 24,true);
             StudentMapper studentMapper = session.getMapper(StudentMapper.class);
             List<Student> students = studentMapper.queryStudentsOrderByColumn(student);
             System.out.println(students);
             session.close();
         }
  ```

   - ${}
   
     - sql语句直接写好
     
     ```xml
      <select id="queryStudentsOrderByColumn" parameterType="student" resultType="student">
             select * from student where stunum = #{stunum} or name like '%${name}%'
         </select>
     ```
   
     - 传参
     
     ```java
      public static void queryStudentsOrderByColumn() throws IOException {
     
             Reader reader = Resources.getResourceAsReader("conf.xml");
             SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(reader, "development");
             SqlSession session = sessionFactory.openSession();
             Student student = new Student(3, "j", 24,true);
             StudentMapper studentMapper = session.getMapper(StudentMapper.class);
             List<Student> students = studentMapper.queryStudentsOrderByColumn(student);
             System.out.println(students);
             session.close();
         }
     ```


3. 嵌套属性 

   数据表加入schooladdress，homeaddress

   java实体类 Address  属性： schooladdress，homeaddress

   Student类中 加入Address 属性

   1. 方法一

   - xml

   ```xml
   <select id="selectStudentByaddress" parameterType="address" resultType="student">
           select * from student where homeaddress = #{homeAddress} or schooladdress = '${schoolAddress}'
       </select>
   ```

   - java

   ```java
    public static void selectStudentByaddress() throws IOException {
   
           Reader reader = Resources.getResourceAsReader("conf.xml");
           SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(reader, "development");
           SqlSession session = sessionFactory.openSession();
   
           Address address = new Address("xa","dl");
           StudentMapper studentMapper = session.getMapper(StudentMapper.class);
           List<Student> students = studentMapper.selectStudentByaddress(address);
           System.out.println(students);
           session.close();
       }
   ```

   2. 方法二，支持级联

   - xml

   ```xml
   <select id="selectStudentByaddress" parameterType="student" resultType="student">
           select * from student where 
       homeaddress = #{address.homeAddress} or 
       schooladdress = '${address.schoolAddress}'
       </select>
   ```

   - java

   ```java
    public static void selectStudentByaddress() throws IOException {
   
           Reader reader = Resources.getResourceAsReader("conf.xml");
           SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(reader, "development");
           SqlSession session = sessionFactory.openSession();
   
           Address address = new Address("xa","dl");
           Student student = new Student();
           student.setAddress(address);
           StudentMapper studentMapper = session.getMapper(StudentMapper.class);
           List<Student> students = studentMapper.selectStudentByaddress(student);
           System.out.println(students);
           session.close();
   ```

   


