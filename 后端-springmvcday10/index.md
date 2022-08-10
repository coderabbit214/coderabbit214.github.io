# 后端-SpringMVC(day10)


# 后端-SpringMVC(day10)

## 文件上传

> 和Servlet方式的本质一样，都是通过commons-fileupload.jar和commons-io.jar
> SpringMVC可以简化文件上传的代码，但是必须满足条件：实现MultipartResolver接口 ；而该接口的实现类SpringMVC也已经提供了CommonsMultipartResolver

1. jar

   commons-fileupload.jar

   commons-io.jar

2. 配置

   > CommonsMultipartResolver类中有许多属性可以配置

   ```xml
   <!--    配置CommonsMultipartResolver，用于实现文件上传，将其加入到SpringIOC容器中,固定id：multipartResolver-->
       <bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
           <property name="defaultEncoding" value="UTF-8"></property>
   <!--        上传单个文件最大值，单位Byte -1表示无限制-->
           <property name="maxUploadSize" value="1024000000"></property>
       </bean>
   ```

3. 处理方法

   > 文件使用@RequestParam("file") MultipartFile file来接收
   >
   > 然后用流的方法保存

   ```java
    @RequestMapping(value="testUpload")
       public String testUpload(@RequestParam("desc") String desc,@RequestParam("file") MultipartFile file) throws IOException {
           System.out.println(desc);
   //        file
           InputStream inputStream = file.getInputStream();
           String fileName = file.getOriginalFilename();
           OutputStream outputStream = new FileOutputStream("d:\\"+fileName);
           byte[] bs = new byte[1024];
           int len = -1;
           while ((len = inputStream.read(bs))!=-1){
               outputStream.write(bs,0,len);
           }
           outputStream.close();
           inputStream.close();
           return "succes";
       }
   ```

   前端页面

   ```jsp
   <form action="SpringMVCHandler/testUpload" method="post" enctype="multipart/form-data">
       描述：<input name="desc" type="text">
       <input type="file" name="file">
       <input type="submit" value="提交">
     </form>
   ```

   

## 拦截器

> 拦截器的原理和过滤器相同。
> SpringMVC：要想实现拦截器，必须实现一个接口HandlerInterceptor

1. 编写拦截器

   ```java
   package org.jsh.interceptor;
   
   import org.springframework.web.servlet.HandlerInterceptor;
   import org.springframework.web.servlet.ModelAndView;
   
   import javax.servlet.http.HttpServletRequest;
   import javax.servlet.http.HttpServletResponse;
   
   public class MyInterceptor implements HandlerInterceptor {
   //拦截请求
       @Override
       public boolean preHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o) throws Exception {
           System.out.println("拦截请求");
           return true;//true：拦截之后放行 false：拦截之后不放行
       }
   //拦截响应
       @Override
       public void postHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, ModelAndView modelAndView) throws Exception {
           System.out.println("拦截响应");
       }
   //视图(jsp)被渲染完毕
       @Override
       public void afterCompletion(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) throws Exception {
           System.out.println("视图(jsp)被渲染完毕");
       }
   }
   
   ```

2. 配置

   ```xml
   <!--    配置拦截器-->
       <mvc:interceptors>
   <!--&lt;!&ndash;        默认拦截全部&ndash;&gt;-->
   <!--        <bean class="org.jsh.interceptor.MyInterceptor"></bean>-->
   <!--        配置具体的拦截路径-->
           <mvc:interceptor>
           <!--            指定拦截的路径，基于ant风格 -->
           <mvc:mapping path="/**"/>
           <!--            指定不拦截的路径-->
           <mvc:exclude-mapping path="/SpringMVCHandler/testUpload"/>
           <bean class="org.jsh.interceptor.MyInterceptor"></bean>
       </mvc:interceptor>
       
       </mvc:interceptors>
   ```

> 如果有多个拦截器 拦截顺序
>
> ：
>
> 拦截请求
> 拦截请求2
> zs
> 拦截响应2
> 拦截响应
> 视图(jsp)被渲染完毕2
> 视图(jsp)被渲染完毕

