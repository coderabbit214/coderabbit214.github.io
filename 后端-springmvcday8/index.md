# 后端-SpringMVC(day8)


# 后端-SpringMVC(day8)

## 数据格式化

> 用到的类 org.springframework.format.support.FormattingConversionServiceFactoryBean

1. springmvc中的配置

   ```xml
   <!--配置数据格式化 注解 所依赖的bean-->
       <bean id="conversionService" class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
   
       </bean>
   ```

2. > ```java
   > @DateTimeFormat(pattern = "yyyy-MM-dd")
   > private Date birthday;
   > 
   > @NumberFormat(pattern = "###,#")
   > private int id;
   > ```

3. 应用

   ```html
     <form action="SpringMVCHandler/testDateTimeFarmat" method="get">
       编号：<input name="id" type="text">
       姓名：<input name="name" type="text">
       出生日期：<input name="birthday" type="text">
       <input type="submit" value="提交">
     </form>
   ```

   ```java
    @RequestMapping(value="testDateTimeFarmat")
       public String testDateTimeFarmat(Student student, BindingResult result){
           if(result.getErrorCount()>0){
               for(FieldError error:result.getFieldErrors()){
                   System.out.println(error.getDefaultMessage());
               }
           }
           System.out.println(student);
           return "succes";
       }
   ```

   ### 补充

   > 前边 类型转换器用的类是ConversionServiceFactoryBean
   >
   > 可以改为FormattingConversionServiceFactoryBean 就可以满足两个功能

## 错误消息传前端

错误消息：
public String testDateTimeFormat(Student student, BindingResult result ,Map<String,Object> map) {
需要验证的数据是 Student中的birthday, SPringMVC要求 如果校验失败  则将错误信息 自动放入 该对象之后紧挨着的	BindingResult中。
即Student student, BindingResult result之间 不能有其他参数。

如果要将控制台的错误消息 传到jsp中显示，则可以将 错误消息对象放入request域中，然后 在jsp中 从request中获取。

