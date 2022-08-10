# 后端-SpringMVC(day7)


# 后端-SpringMVC(day7)

## 1.SpringMVC实现：index.jsp -> succes.jsp

1. index.jsp

   ```html
   <a href="handler/testMvcViewController">handler/testMvcViewController</a>
   ```

2. springmvc.xml

   > 单独使用mvc:view-controller时会屏蔽原本的@RequestMapping形式跳转
   >
   > 需要使用mvc:annotation-driven协调

   ```xml
   <!--    view-name 会被视图解析器加上 前缀 后缀  -->
       <mvc:view-controller path="handler/testMvcViewController" view-name="succes"></mvc:view-controller>
   <!--    此配置是SpringMVC的基础配置，很多功能都需要通过该注解来协调  -->
       <mvc:annotation-driven></mvc:annotation-driven>
   ```

## 2.指定请求方式

> 默认为request
>
> 指定跳转方式：return "forward:/views/success.jsp";    
>
> forward:   redirect: ，需要注意 此种方式，不会被视图解析器加上前缀(/views)、后缀(.jsp)

```java
 @RequestMapping(value="testModelMap")
    public String testModelMap(ModelMap modelMap){//ModelAndView既有数据又有视图
        Student student = new Student();
        student.setName("aaa");
        modelMap.put("student",student);
         //String s = "forward:/views/succes.jsp";//请求转发 默认
        String s = "redirect:/views/succes.jsp";//重定向
        return s;
    }
```

## 3.处理静态资源

> 原本使用静态资源会被拦截寻找对应的@RequestMapping
>
> ```xml
> <servlet-mapping>
>     <servlet-name>springDispatcherServelet</servlet-name>
>     <!--拦截所有-->
>     <url-pattern>/</url-pattern>
> </servlet-mapping>
> ```
> 

> 解决
>
> 如果是 需要mvc处理的，则交给@RequsetMapping("img.png")处理；如果不需要springmvc处理，则使用 tomcat默认的Servlet去处理。
> tomcat默认的Servlet去处理：如果有 对应的请求拦截,则交给相应的Servlet去处理；如果没有对应的servlet，则直接访问。
>
> 在springmvc.xml中添加两个注解

```xml
<!--    此配置是SpringMVC的基础配置，很多功能都需要通过该注解来协调  -->
    <mvc:annotation-driven></mvc:annotation-driven>
<!--    该注解会让 springmvc：接收一个请求，并且该请求没有对应的@requestmapping时，将该请求交给默认的servlet-->
    <mvc:default-servlet-handler></mvc:default-servlet-handler>
```

## 4.类型转换

1. Spring自带类型转换

2. 自定义类型转换器

   1. 编写 自定义类型转器的类 （实现Converter接口）

      ```java
      package org.jsh.converter;
      
      
      import org.jsh.entity.Student;
      import org.springframework.core.convert.converter.Converter;
      
      public class MyConverter implements Converter<String, Student> {
          @Override
          public Student convert(String source) {
              Student student = new Student();
              String[] studentStrArr = source.split("-");
              student.setId(Integer.parseInt(studentStrArr[0]));
              student.setName(studentStrArr[1]);
              return student;
          }
      }
      
      ```

   2. 配置：将MyConverter加入到springmvc中

      ```xml
      <!--    类型转换器配置-->
      
      <!--    1.将自定义转换器纳入SpringIOC容器中-->
          <bean id="myConverter" class="org.jsh.converter.MyConverter"></bean>
      
      <!--    2.将myConverter纳入SpringMVC提供的转换器bean-->
          <bean id="conversionService" class="org.springframework.context.support.ConversionServiceFactoryBean">
              <property name="converters">
                  <set>
                      <ref bean="myConverter"/>
                  </set>
              </property>
          </bean>
      
      <!--    3.将conversionService注册到annotation-driven中-->
      <!--    此配置是SpringMVC的基础配置，很多功能都需要通过该注解来协调  -->
          <mvc:annotation-driven conversion-service="conversionService"></mvc:annotation-driven>
      ```

   3. 测试

      index.jsp

      ```html
      <form action="SpringMVCHandler/testConverter" method="get">
          学生信息：<input name="studentInfo" type="text">
          <input type="submit" value="提交">
        </form>
      
      ```

      SpringMVCHandler.java

      ```java
       @RequestMapping(value="testConverter")
          public String testConverter(@RequestParam("studentInfo")Student student){//ModelAndView既有数据又有视图
              System.out.println(student);
              return "succes";
          }
      ```

      


