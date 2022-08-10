# 后端-SpringMVC(day9)


# 后端-SpringMVC(day9)

## 数据校验

1. jar

   hibernate-validator-5.0.0.CR2.jar 	

   classmate-0.8.0.jar 	

   jboss-logging-3.1.1.GA.jar

   validation-api-1.1.0.CR1.jar 	

   hibernate-validator-annotation-processor-5.0.0.CR2.jar

2. 配置

   <mvc:annotation-driven ></mvc:annotation-driven>
   此时mvc:annotation-driven的作用：要实现Hibernate Validator/JSR303 校验（或者其他各种校验），必须实现SpringMVC提供的一个接口：ValidatorFactory

   LocalValidatorFactoryBean是ValidatorFactory的一个实现类。
   <mvc:annotation-driven ></mvc:annotation-driven>会在springmvc容器中 自动加载一个LocalValidatorFactoryBean类，因此可以直接实现数据校验。

3. 使用

   ```java
   public class Student {
   
   	@Past//当前时间以前
   	private Date birthday ;
   }
   	
   
   在校验的Controller中 ，给校验的对象前增加 @Valid
   		public String testDateTimeFormat(@Valid Student student, BindingResult result ,Map<String,Object> map) {
   			{...}
   
   ```

### 注解

| 注解                         | 简介                                                   |
| ---------------------------- | ------------------------------------------------------ |
| @Null                        | 被注释的元素必须为 null。                              |
| @NotNull                     | 被注释的元素必须不为 null。                            |
| @AssertTrue                  | 被注释的元素必须为 true。                              |
| @AssertFalse                 | 被注释的元素必须为 false。                             |
| @Min(value)                  | 被注释的元素必须是一个数字，其值必须大于或等于value。  |
| @Max(value)                  | 被注释的元素必须是一个数字，其值必须小于或等于value。  |
| @DecimalMin(value)           | 被注释的元素必须是一个数字，其值必须大于或等于value。  |
| @DecimalMax(value)           | 被注释的元素必须是一个数字，其值必须小于或等于value。  |
| @Size(max,  min)             | 被注释的元素的取值范围必须是介于min和max之间。         |
| @Digits  (integer, fraction) | 被注释的元素必须是一个数字，其值必须在可接受的范围内。 |
| @Past                        | 被注释的元素必须是一个过去的日期。                     |
| @Future                      | 被注释的元素必须是一个将来的日期。                     |
| @Pattern(value)              | 被注释的元素必须符合指定的正则表达式。                 |

| 注解      | 简介                                     |
| --------- | ---------------------------------------- |
| @Email    | 被注释的元素值必须是合法的电子邮箱地址。 |
| @Length   | 被注释的字符串的长度必须在指定的范围内。 |
| @NotEmpty | 被注释的字符串的必须非空。               |
| @Range    | 被注释的元素必须在合适的范围内。         |

## Ajax请求SpringMVC，并且JSON格式的数据

1. jar

   jackson-annotations-2.8.9.jar

   jackson-core-2.8.9.jar

   jackson-databind-2.8.9.jar

2. handler

   > @ResponseBody//告诉SpringMVC，此时的返回 不是一个 View页面，而是一个 ajax调用的返回值（Json数组）

   ```java
   @ResponseBody
       @RequestMapping(value="testJson")
       public List<Student> SpringMVCHandler(){
           List<Student> students = new ArrayList<>();
           Student student1 = new Student(4,"zs");
           Student student2 = new Student(2,"zs");
           Student student3 = new Student(1,"zs");
           students.add(student1);
           students.add(student2);
           students.add(student3);
           return students;
       }
   ```

3. 前端页面

   ```jsp
   <%@ page contentType="text/html;charset=UTF-8" language="java" %>
   <html>
     <head>
       <title>$Title$</title>
       <script type="text/javascript" src="js/jquery-1.8.3.js"></script>
       <script type="text/javascript">
         $(document).ready(function () {
           $("#testJson").click(function () {
             $.post(
                     "SpringMVCHandler/testJson",//服务器地址
                     function (result) {//回调函数 List<Student> json数组格式
                      for(var i=0;i<result.length;i++){
                         console.log(result[i].id+"-"+result[i].name);
                      }
                     }
             );
           })
   
         })
       </script>
     </head>
     <body>
     <input type="button" value="json" id="testJson">
       </body>
   </html>
   ```

   


