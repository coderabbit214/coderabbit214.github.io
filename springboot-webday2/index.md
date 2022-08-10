# SpringBoot-Web


# SpringBoot-Web

## 一. 自动配置

1. SpsringBoot已经帮我们配置好了基础的配置 例如 视图解析器 扫描器等

2. 如果要使用自己定义的配置 只需要创建一个相应的对象 加入到 IOC容器中即可

   例如

   ```java
   //  添加自己的视图解析器
       @Bean
       public ViewResolver myViewReolver(){
           return new MyViewReolver();
       }
       private class MyViewReolver implements ViewResolver{
           @Override
           public View resolveViewName(String s, Locale locale) throws Exception {
               return null;
           }
       }
   ```

## 二.静态资源

### 1.引入jQuery等

[webjars](https://www.webjars.org/)上找对应的maven依赖

```xml
<dependency>
            <groupId>org.webjars</groupId>
            <artifactId>jquery</artifactId>
            <version>3.3.1</version>
        </dependency>
```

### 2. 默认静态文件夹位置

```
"classpath:/META-INF/resources/", 
"classpath:/resources/",
"classpath:/static/", 
"classpath:/public/" 
"/"：当前项目的根路径
```

```properties
#改变静态文件夹，默认的会失效
#spring.resources.static-locations=classpath:/hello/,classpath:/jsh/
```

### 3.欢迎页

localhost:8080/   找index页面

index.html 会自动去静态文件夹下找

### 4.页面图标

favicon.ico 放在静态文件夹下 会自动寻找

## 三.模板引擎Thymeleaf

### 1.引入thymeleaf

```xml
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
```

更改版本

```xml
 <properties>
        <thymeleaf.version>3.0.11.RELEASE</thymeleaf.version>
        <!-- 布局功能的支持程序  thymeleaf3主程序  layout2以上版本 -->
        <!-- thymeleaf2   layout1-->
        <thymeleaf-layout-dialect.version>2.2.2</thymeleaf-layout-dialect.version>
    </properties>
```

### 2.使用

1.导入thymeleaf的名称空间

```xml
<html lang="en" xmlns:th="http://www.thymeleaf.org">
```

2.语法

​	th：任意html属性；来替换原生属性的值

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <h1>成功</h1>
<!--    th:text设置div里边的文本内容 如果没有数据 会显示原本的数据 -->
<!--    th：任意html属性；来替换原生属性的值-->
    <div id="div01" class="myDiv" th:id="${hello}" th:text="${hello}">
        这里应该显示数据
    </div>
<!--    utext 不转义特殊字符 -->
    <div th:text="${hello}"></div>
<!--    行内写法-->
    <div>[[${hello}]]</div>
    <div th:utext="${hello}"></div>
    <div>[(${hello})]</div>
<!--    th:each 遍历-->
    <h4 th:text="${user}" th:each="user:${users}"></h4>
    <h4>
        <span th:each="user:${users}">[[${user}]]</span>
    </h4>
</body>
</html>
```

3.语法规则

```properties
Simple expressions:（表达式语法）
    Variable Expressions: ${...}：获取变量值；OGNL；
    		1）、获取对象的属性、调用方法
    		2）、使用内置的基本对象：
    			#ctx : the context object.
    			#vars: the context variables.
                #locale : the context locale.
                #request : (only in Web Contexts) the HttpServletRequest object.
                #response : (only in Web Contexts) the HttpServletResponse object.
                #session : (only in Web Contexts) the HttpSession object.
                #servletContext : (only in Web Contexts) the ServletContext object.
                
                ${session.foo}
            3）、内置的一些工具对象：
#execInfo : information about the template being processed.
#messages : methods for obtaining externalized messages inside variables expressions, in the same way as they would be obtained using #{…} syntax.
#uris : methods for escaping parts of URLs/URIs
#conversions : methods for executing the configured conversion service (if any).
#dates : methods for java.util.Date objects: formatting, component extraction, etc.
#calendars : analogous to #dates , but for java.util.Calendar objects.
#numbers : methods for formatting numeric objects.
#strings : methods for String objects: contains, startsWith, prepending/appending, etc.
#objects : methods for objects in general.
#bools : methods for boolean evaluation.
#arrays : methods for arrays.
#lists : methods for lists.
#sets : methods for sets.
#maps : methods for maps.
#aggregates : methods for creating aggregates on arrays or collections.
#ids : methods for dealing with id attributes that might be repeated (for example, as a result of an iteration).

    Selection Variable Expressions: *{...}：选择表达式：和${}在功能上是一样；
    	补充：配合 th:object="${session.user}：
   <div th:object="${session.user}">
    <p>Name: <span th:text="*{firstName}">Sebastian</span>.</p>
    <p>Surname: <span th:text="*{lastName}">Pepper</span>.</p>
    <p>Nationality: <span th:text="*{nationality}">Saturn</span>.</p>
    </div>
    
    Message Expressions: #{...}：获取国际化内容
    Link URL Expressions: @{...}：定义URL；
    		@{/order/process(execId=${execId},execType='FAST')}
    Fragment Expressions: ~{...}：片段引用表达式
    		<div th:insert="~{commons :: main}">...</div>
    		
Literals（字面量）
      Text literals: 'one text' , 'Another one!' ,…
      Number literals: 0 , 34 , 3.0 , 12.3 ,…
      Boolean literals: true , false
      Null literal: null
      Literal tokens: one , sometext , main ,…
Text operations:（文本操作）
    String concatenation: +
    Literal substitutions: |The name is ${name}|
Arithmetic operations:（数学运算）
    Binary operators: + , - , * , / , %
    Minus sign (unary operator): -
Boolean operations:（布尔运算）
    Binary operators: and , or
    Boolean negation (unary operator): ! , not
Comparisons and equality:（比较运算）
    Comparators: > , < , >= , <= ( gt , lt , ge , le )
    Equality operators: == , != ( eq , ne )
Conditional operators:条件运算（三元运算符）
    If-then: (if) ? (then)
    If-then-else: (if) ? (then) : (else)
    Default: (value) ?: (defaultvalue)
Special tokens:
    No-Operation: _ 
```

## 四.扩展SpringMVC

原本配置方法

```xml
<mvc:view-controller path="/hello" view-name="success"/>
<mvc:interceptors>
    <mvc:interceptor>
        <mvc:mapping path="/hello"/>
        <bean></bean>
    </mvc:interceptor>
</mvc:interceptors>
```
springBoot配置方法

> 1. 实现WebMvcConfigurer接口
> 2. 添加注释@Configuration 会保留原来的自动配置
> 3. @EnableWebMvc 不会保留 完全由自己编写SpringMvc配置

```java
//所有的WebMvcConfigurer组件都会一起起作用
//使用implements WebMvcConfigurer扩展springmvc功能
//@EnableWebMvc全面接管springmvc 自动配置会失效
@Configuration
//@EnableWebMvc
public class MyMvcConfig implements WebMvcConfigurer{
    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        //super.addViewControllers(registry);
//       浏览器发送 /lujieling 请求来到 success
        registry.addViewController("/lujieling").setViewName("success");
        registry.addViewController("/").setViewName("login");
        registry.addViewController("/index").setViewName("login");
        registry.addViewController("/main.html").setViewName("dashboard");
    }
```

### 1.默认访问首页

```java
@Configuration
public class MyMvcConfig implements WebMvcConfigurer{
    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        //super.addViewControllers(registry);
//       浏览器发送 /lujieling 请求来到 success
        registry.addViewController("/lujieling").setViewName("success");
        registry.addViewController("/").setViewName("login");
        registry.addViewController("/index").setViewName("login");
       
```

### 2.国际化

#### 1.编写国际化配置文件

classpath目录下        i18n

#### 2.直接在页面中使用

> 语法 #{}

```html
<!DOCTYPE html>
<html lang="en"  xmlns:th="http://www.thymeleaf.org">
	<head>
		<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
		<meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
		<meta name="description" content="">
		<meta name="author" content="">
		<title>Signin Template for Bootstrap</title>
		<!-- Bootstrap core CSS -->
		<link href="asserts/css/bootstrap.min.css" th:href="@{/webjars/bootstrap/4.0.0/css/bootstrap.css}" rel="stylesheet">
		<!-- Custom styles for this template -->
		<link href="asserts/css/signin.css" th:href="@{/asserts/css/signin.css}" rel="stylesheet">
	</head>

	<body class="text-center">
		<form class="form-signin" action="dashboard.html">
			<img class="mb-4" th:src="@{/asserts/img/bootstrap-solid.svg}" src="asserts/img/bootstrap-solid.svg" alt="" width="72" height="72">
			<h1 class="h3 mb-3 font-weight-normal" th:text="#{login.tip}">Please sign in</h1>
			<label class="sr-only" th:text="#{login.username}">Username</label>
			<input type="text" class="form-control" placeholder="Username" th:placeholder="#{login.username}" required="" autofocus="">
			<label class="sr-only" th:text="#{login.password}">Password</label>
			<input type="password" class="form-control" placeholder="Password" th:placeholder="#{login.password}" required="">
			<div class="checkbox mb-3">
				<label>
          		<input type="checkbox" value="remember-me"/> [[#{login.remember}]]
        </label>
			</div>
			<button class="btn btn-lg btn-primary btn-block" type="submit" th:text="#{login.btn}">Sign in</button>
			<p class="mt-5 mb-3 text-muted">© 2017-2018</p>
			<a class="btn btn-sm">中文</a>
			<a class="btn btn-sm">English</a>
		</form>

	</body>

</html>
```

#### 3.点击链接切换国际化

1.编写自己的LocaleResolver

```java
package com.jsh.springweb.componet;

import org.springframework.web.servlet.LocaleResolver;
import org.thymeleaf.util.StringUtils;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.util.Locale;

/**
 * 可以在链接上携带区域信息
 */
public class MyLocaleResolver implements LocaleResolver {

    @Override
    public Locale resolveLocale(HttpServletRequest httpServletRequest) {
        String l = httpServletRequest.getParameter("l");
        Locale locale = Locale.getDefault();
        if(!StringUtils.isEmpty(l)){
            String[] strings = l.split("_");
            locale = new Locale(strings[0],strings[1]);
        }
        return locale;
    }

    @Override
    public void setLocale(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Locale locale) {

    }
}

```

2.放在IOC容器中

```java
 @Bean
    public LocaleResolver localeResolver(){
        return new MyLocaleResolver();
    }
}

```

### 3.开发时前端修改不用重启设置

开发期间模板引擎页面修改以后，要实时生效

1）、禁用模板引擎的缓存

```
# 禁用缓存
spring.thymeleaf.cache=false 
```

2）、页面修改完成以后ctrl+f9：重新编译；

### 4.防止表单进行重复提交

```java
@Controller
public class LoginController {
//    @DeleteMapping
//    @PutMapping
//    @GetMapping
    @PostMapping("/user/login")
    public String login(@RequestParam("username") String username,
                        @RequestParam("password") String password,
                        Map<String,Object> maps,
                        HttpSession session){
        if (!StringUtils.isEmpty(username) && "123456".equals(password)){
            session.setAttribute("loginUser",username);
            //            登陆成功,防止表单重复提交，使用重定向
            return "redirect:/main.html";
        }
//        登陆失败
        maps.put("msg","账户名或密码错误");
        return "login";
    }
}

```

```java
@Configuration
public class MyMvcConfig implements WebMvcConfigurer{
    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
     registry.addViewController("/main.html").setViewName("dashboard");
    }
```



### 5.拦截器进行登陆检查

1. 编写拦截器

   ```java
   package com.jsh.springweb.componet;
   
   import org.springframework.web.servlet.HandlerInterceptor;
   import org.springframework.web.servlet.ModelAndView;
   import org.thymeleaf.util.StringUtils;
   
   import javax.servlet.http.HttpServletRequest;
   import javax.servlet.http.HttpServletResponse;
   //登陆检查
   public class LoginHandlerInterceptor implements HandlerInterceptor {
   //   目标方法执行之前
       @Override
       public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
           Object user = request.getSession().getAttribute("loginUser");
           if (user == null){
   //            未登录
               request.setAttribute("mag","没有权限");
               request.getRequestDispatcher("/index.html").forward(request,response);
               return false;
           }else {
   //            已登陆
               return true;
           }
       }
   
       @Override
       public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
   
       }
   
       @Override
       public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
   
       }
   }
   
   ```

2. 注册拦截器

   > 注: SpringBoot已经做好了静态资源映射
   >
   > 方法作用
   >
   > ​	addPathPatterns拦截目标
   >
   > ​	excludePathPatterns排除目标

   ```java
   @Configuration
   public class MyMvcConfig implements WebMvcConfigurer{
   
       @Override
       public void addInterceptors(InterceptorRegistry registry) {
           registry.addInterceptor(new LoginHandlerInterceptor()).addPathPatterns("/**").excludePathPatterns("/index.html","/","/user/login");
       }
   }
   
   ```

   

