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
Message Expressions: #{...}：获取国际化内容
Link URL Expressions: @{...}：定义URL；
Variable Expressions: ${...}：获取变量值；OGNL；

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

### 3. 抽取模板

1. 抽取

   > 在需要抽取的部分 使用   th:fragment="模板名称" 
   >
   > th:fragment="topbar" 

   ```html
   <body>
   <nav class="navbar navbar-dark sticky-top bg-dark flex-md-nowrap p-0" th:fragment="topbar">
       <a class="navbar-brand col-sm-3 col-md-2 mr-0" href="http://getbootstrap.com/docs/4.0/examples/dashboard/#">[[${session.loginUser}]]</a>
       <input class="form-control form-control-dark w-100" type="text" placeholder="Search" aria-label="Search">
       <ul class="navbar-nav px-3">
           <li class="nav-item text-nowrap">
               <a class="nav-link" href="http://getbootstrap.com/docs/4.0/examples/dashboard/#">Sign out</a>
           </li>
       </ul>
   </nav>
   
   <nav class="col-md-2 d-none d-md-block bg-light sidebar" id="sidebar">
       <div class="sidebar-sticky">
           <ul class="nav flex-column">
               <li class="nav-item">
                   <a class="nav-link active" th:class="${activeUri=='main.html'?'nav-link active':'nav-link'}" th:href="@{/main.html}" href="#">
                       <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-home">
                           <path d="M3 9l9-7 9 7v11a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2z"></path>
                           <polyline points="9 22 9 12 15 12 15 22"></polyline>
                       </svg>
                       Dashboard <span class="sr-only">(current)</span>
                   </a>
               </li>
               <li class="nav-item">
                   <a class="nav-link" href="http://getbootstrap.com/docs/4.0/examples/dashboard/#">
                       <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-file">
                           <path d="M13 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V9z"></path>
                           <polyline points="13 2 13 9 20 9"></polyline>
                       </svg>
                       Orders
                   </a>
               </li>
               <li class="nav-item">
                   <a class="nav-link" href="http://getbootstrap.com/docs/4.0/examples/dashboard/#">
                       <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-shopping-cart">
                           <circle cx="9" cy="21" r="1"></circle>
                           <circle cx="20" cy="21" r="1"></circle>
                           <path d="M1 1h4l2.68 13.39a2 2 0 0 0 2 1.61h9.72a2 2 0 0 0 2-1.61L23 6H6"></path>
                       </svg>
                       Products
                   </a>
               </li>
               <li class="nav-item">
                   <a class="nav-link" href="#" th:href="@{/emps}" th:class="${activeUri=='emps.html'?'nav-link active':'nav-link'}">
                       <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-users">
                           <path d="M17 21v-2a4 4 0 0 0-4-4H5a4 4 0 0 0-4 4v2"></path>
                           <circle cx="9" cy="7" r="4"></circle>
                           <path d="M23 21v-2a4 4 0 0 0-3-3.87"></path>
                           <path d="M16 3.13a4 4 0 0 1 0 7.75"></path>
                       </svg>
                       员工管理
                   </a>
               </li>
               <li class="nav-item">
                   <a class="nav-link" href="http://getbootstrap.com/docs/4.0/examples/dashboard/#">
                       <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-bar-chart-2">
                           <line x1="18" y1="20" x2="18" y2="10"></line>
                           <line x1="12" y1="20" x2="12" y2="4"></line>
                           <line x1="6" y1="20" x2="6" y2="14"></line>
                       </svg>
                       Reports
                   </a>
               </li>
               <li class="nav-item">
                   <a class="nav-link" href="http://getbootstrap.com/docs/4.0/examples/dashboard/#">
                       <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-layers">
                           <polygon points="12 2 2 7 12 12 22 7 12 2"></polygon>
                           <polyline points="2 17 12 22 22 17"></polyline>
                           <polyline points="2 12 12 17 22 12"></polyline>
                       </svg>
                       Integrations
                   </a>
               </li>
           </ul>
   
           <h6 class="sidebar-heading d-flex justify-content-between align-items-center px-3 mt-4 mb-1 text-muted">
               <span>Saved reports</span>
               <a class="d-flex align-items-center text-muted" href="http://getbootstrap.com/docs/4.0/examples/dashboard/#">
                   <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-plus-circle"><circle cx="12" cy="12" r="10"></circle><line x1="12" y1="8" x2="12" y2="16"></line><line x1="8" y1="12" x2="16" y2="12"></line></svg>
               </a>
           </h6>
           <ul class="nav flex-column mb-2">
               <li class="nav-item">
                   <a class="nav-link" href="http://getbootstrap.com/docs/4.0/examples/dashboard/#">
                       <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-file-text">
                           <path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"></path>
                           <polyline points="14 2 14 8 20 8"></polyline>
                           <line x1="16" y1="13" x2="8" y2="13"></line>
                           <line x1="16" y1="17" x2="8" y2="17"></line>
                           <polyline points="10 9 9 9 8 9"></polyline>
                       </svg>
                       Current month
                   </a>
               </li>
               <li class="nav-item">
                   <a class="nav-link" href="http://getbootstrap.com/docs/4.0/examples/dashboard/#">
                       <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-file-text">
                           <path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"></path>
                           <polyline points="14 2 14 8 20 8"></polyline>
                           <line x1="16" y1="13" x2="8" y2="13"></line>
                           <line x1="16" y1="17" x2="8" y2="17"></line>
                           <polyline points="10 9 9 9 8 9"></polyline>
                       </svg>
                       Last quarter
                   </a>
               </li>
               <li class="nav-item">
                   <a class="nav-link" href="http://getbootstrap.com/docs/4.0/examples/dashboard/#">
                       <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-file-text">
                           <path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"></path>
                           <polyline points="14 2 14 8 20 8"></polyline>
                           <line x1="16" y1="13" x2="8" y2="13"></line>
                           <line x1="16" y1="17" x2="8" y2="17"></line>
                           <polyline points="10 9 9 9 8 9"></polyline>
                       </svg>
                       Social engagement
                   </a>
               </li>
               <li class="nav-item">
                   <a class="nav-link" href="http://getbootstrap.com/docs/4.0/examples/dashboard/#">
                       <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-file-text">
                           <path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"></path>
                           <polyline points="14 2 14 8 20 8"></polyline>
                           <line x1="16" y1="13" x2="8" y2="13"></line>
                           <line x1="16" y1="17" x2="8" y2="17"></line>
                           <polyline points="10 9 9 9 8 9"></polyline>
                       </svg>
                       Year-end sale
                   </a>
               </li>
           </ul>
       </div>
   </nav>
   </body>
   ```

2. 使用

   > **th:insert**：将公共片段整个插入到声明引入的元素中
   >
   > **th:replace**：将声明引入的元素替换为公共片段
   >
   > **th:include**：将被引入的片段的内容包含进这个标签中

   ```html
   <!--		引入抽取的topbar-->
   <!--		模板名：使用thymeleaf的前后缀配置规则进行解析-->
   <div th:replace="~{commons/bar::topbar}"></div>
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


## 五.CRUD

### 1.RestfulCRUD

|      | 普通CRUD（uri来区分操作） | RestfulCRUD       |
| ---- | ------------------------- | ----------------- |
| 查询 | getEmp                    | emp---GET         |
| 添加 | addEmp?xxx                | emp---POST        |
| 修改 | updateEmp?id=xxx&xxx=xx   | emp/{id}---PUT    |
| 删除 | deleteEmp?id=1            | emp/{id}---DELETE |

实验请求架构

| 实验功能                             | 请求URI | 请求方式 |
| ------------------------------------ | ------- | -------- |
| 查询所有员工                         | emps    | GET      |
| 查询某个员工(来到修改页面)           | emp/1   | GET      |
| 来到添加页面                         | emp     | GET      |
| 添加员工                             | emp     | POST     |
| 来到修改页面（查出员工进行信息回显） | emp/1   | GET      |
| 修改员工                             | emp     | PUT      |
| 删除员工                             | emp/1   | DELETE   |

### 2.CRUD-员工添加

1. 前端首页

   ```html
   <h2><a class="btn btn-sm btn-success" href="/emps" th:href="@{/emp}">Section title</a></h2>
   ```

2. Controller

   ```java
   //    来到添加员工页面
       @GetMapping("/emp")
       public String toAddPage(Map<String,Object> maps){
       //        页面要显示的所有部门列表
           Collection<Department> departments = departmentDao.getDepartments();
           maps.put("departments",departments);
           return "emp/add";
       }
   ```

3. 添加员工页面

   > 提交时日期格式设置
   >
   > ```properties
   > #日期格式化器 默认yyyy/MM/dd
   > spring.mvc.date-format=yyyy-MM-dd
   > ```

   关键代码：

   ```html
    <form th:action="@{/emp}" method="post">
   ```

   完整代码：（包含后边的修改功能）

   ```html
   <!DOCTYPE html>
   <!-- saved from url=(0052)http://getbootstrap.com/docs/4.0/examples/dashboard/ -->
   <html lang="en" xmlns:th="http://www.thymeleaf.org">
   
   <head>
       <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
       <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
       <meta name="description" content="">
       <meta name="author" content="">
   
       <title>Dashboard Template for Bootstrap</title>
       <!-- Bootstrap core CSS -->
       <link th:href="@{/webjars/bootstrap/4.0.0/css/bootstrap.css}" href="../../static/asserts/css/bootstrap.min.css" rel="stylesheet">
   
       <!-- Custom styles for this template -->
       <link th:href="@{/asserts/css/dashboard.css}" href="../../static/asserts/css/dashboard.css" rel="stylesheet">
       <style type="text/css">
           /* Chart.js */
   
           @-webkit-keyframes chartjs-render-animation {
               from {
                   opacity: 0.99
               }
               to {
                   opacity: 1
               }
           }
   
           @keyframes chartjs-render-animation {
               from {
                   opacity: 0.99
               }
               to {
                   opacity: 1
               }
           }
   
           .chartjs-render-monitor {
               -webkit-animation: chartjs-render-animation 0.001s;
               animation: chartjs-render-animation 0.001s;
           }
       </style>
   </head>
   
   <body>
   <!--		引入抽取的topbar-->
   <!--		模板名：使用thymeleaf的前后缀配置规则进行解析-->
   <div th:replace="~{commons/bar::topbar}"></div>
   <div class="container-fluid">
       <div class="row">
           <!--				引入侧边栏-->
           <div th:replace="commons/bar::#sidebar(activeUri='emps.html')"></div>
   
           <main role="main" class="col-md-9 ml-sm-auto col-lg-10 pt-3 px-4">
   <!--            区分是员工修改 还是添加 post 添加 -->
               <form th:action="@{/emp}" method="post">
   <!--                put 修改-->
   <!--
                   1.SpringMVC中配置HiddenHttpMethodFilter
                   2.页面创建post表单
                   3.创建input name = "_method" value = 指定的请求方式
                   -->
                   <input th:if="${emp!=null}" type="hidden" name="_method" value="put">
                   <input th:if="${emp!=null}" type="hidden" name="id" th:value="${emp.id}">
                   <div class="form-group">
                       <label>LastName</label>
                       <input th:value="${emp!=null}?${emp.lastName}" name="lastName" type="text" class="form-control" placeholder="zhangsan">
                   </div>
                   <div class="form-group">
                       <label>Email</label>
                       <input th:value="${emp!=null}?${emp.email}" name="email" type="email" class="form-control" placeholder="zhangsan@atguigu.com">
                   </div>
                   <div class="form-group">
                       <label>Gender</label><br/>
                       <div class="form-check form-check-inline">
                           <input th:checked="${emp!=null}?${emp.gender==1}" class="form-check-input" type="radio" name="gender"  value="1">
                           <label class="form-check-label">男</label>
                       </div>
                       <div class="form-check form-check-inline">
                           <input th:checked="${emp!=null}?${emp.gender==0}" class="form-check-input" type="radio" name="gender"  value="0">
                           <label class="form-check-label">女</label>
                       </div>
                   </div>
                   <div class="form-group">
                       <label>department</label>
                       <select name="department.id" class="form-control">
                           <option th:selected="${emp!=null}?${department.id == emp.department.id}" th:value="${department.id}" th:each="department:${departments}" th:text="${department.departmentName}">1</option>
                       </select>
                   </div>
                   <div class="form-group">
                       <label>Birth</label>
                       <input th:value="${emp!=null}?${#dates.format(emp.birth,'yyyy-MM-dd HH:mm')}" name="birth" type="text" class="form-control" placeholder="1999-02-14">
                   </div>
                   <button th:text="${emp!=null}?'修改':'添加'" type="submit" class="btn btn-primary">添加</button>
               </form>
           </main>
       </div>
   </div>
   
   <!-- Bootstrap core JavaScript
   ================================================== -->
   <!-- Placed at the end of the document so the pages load faster -->
   <script type="text/javascript" src="../../static/asserts/js/jquery-3.2.1.slim.min.js"></script>
   <script type="text/javascript" src="../../static/asserts/js/popper.min.js"></script>
   <script type="text/javascript" src="../../static/asserts/js/bootstrap.min.js"></script>
   
   <!-- Icons -->
   <script type="text/javascript" src="../../static/asserts/js/feather.min.js"></script>
   <script>
       feather.replace()
   </script>
   
   <!-- Graphs -->
   <script type="text/javascript" src="../../static/asserts/js/Chart.min.js"></script>
   <script>
       var ctx = document.getElementById("myChart");
       var myChart = new Chart(ctx, {
           type: 'line',
           data: {
               labels: ["Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"],
               datasets: [{
                   data: [15339, 21345, 18483, 24003, 23489, 24092, 12034],
                   lineTension: 0,
                   backgroundColor: 'transparent',
                   borderColor: '#007bff',
                   borderWidth: 4,
                   pointBackgroundColor: '#007bff'
               }]
           },
           options: {
               scales: {
                   yAxes: [{
                       ticks: {
                           beginAtZero: false
                       }
                   }]
               },
               legend: {
                   display: false,
               }
           }
       });
   </script>
   
   </body>
   
   </html>
   ```

4. Controller

   ```java
   //    员工添加功能
   //    自动将参数封装：请求参数名字和对象属性名要相同
       @PostMapping("/emp")
       public String addEmp(Employee employee){
           employeeDao.save(employee);
   //        redirect:表示重定向到一个地址
   //        forword:转发到一个地址
           return "redirect:/emps";
       }
           //    查询所有员工返回列表页面
       @GetMapping("emps")
       public String list(Map<String,Object>  map){
           Collection<Employee> employees = employeeDao.getAll();
           map.put("emps",employees);
           return "emp/list";
       }
   ```

5. 前端list页面

   ```html
   <!DOCTYPE html>
   <!-- saved from url=(0052)http://getbootstrap.com/docs/4.0/examples/dashboard/ -->
   <html lang="en" xmlns:th="http://www.thymeleaf.org">
   
   	<head>
   		<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
   		<meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
   		<meta name="description" content="">
   		<meta name="author" content="">
   
   		<title>Dashboard Template for Bootstrap</title>
   		<!-- Bootstrap core CSS -->
   		<link th:href="@{/webjars/bootstrap/4.0.0/css/bootstrap.css}" href="../../static/asserts/css/bootstrap.min.css" rel="stylesheet">
   
   		<!-- Custom styles for this template -->
   		<link th:href="@{/asserts/css/dashboard.css}" href="../../static/asserts/css/dashboard.css" rel="stylesheet">
   		<style type="text/css">
   			/* Chart.js */
   			
   			@-webkit-keyframes chartjs-render-animation {
   				from {
   					opacity: 0.99
   				}
   				to {
   					opacity: 1
   				}
   			}
   			
   			@keyframes chartjs-render-animation {
   				from {
   					opacity: 0.99
   				}
   				to {
   					opacity: 1
   				}
   			}
   			
   			.chartjs-render-monitor {
   				-webkit-animation: chartjs-render-animation 0.001s;
   				animation: chartjs-render-animation 0.001s;
   			}
   		</style>
   	</head>
   
   	<body>
   <!--		引入抽取的topbar-->
   <!--		模板名：使用thymeleaf的前后缀配置规则进行解析-->
   		<div th:replace="~{commons/bar::topbar}"></div>
   		<div class="container-fluid">
   			<div class="row">
   <!--				引入侧边栏-->
   				<div th:replace="commons/bar::#sidebar(activeUri='emps.html')"></div>
   
   				<main role="main" class="col-md-9 ml-sm-auto col-lg-10 pt-3 px-4">
   					<h2><a class="btn btn-sm btn-success" href="/emps" th:href="@{/emp}">Section title</a></h2>
   					<div class="table-responsive">
   						<table class="table table-striped table-sm">
   							<thead>
   								<tr>
   									<th>#</th>
   									<th>lastName</th>
   									<th>email</th>
   									<th>gender</th>
   									<th>department</th>
   									<th>birth</th>
   									<th>操作</th>
   								</tr>
   							</thead>
   							<tbody>
   								<tr th:each="emp:${emps}">
   									<td th:text="${emp.id}"></td>
   									<td th:text="${emp.lastName}"></td>
   									<td th:text="${emp.email}"></td>
   									<td th:text="${emp.gender}=='1'?'男':'女'"></td>
   									<td th:text="${emp.department.departmentName}"></td>
   									<td th:text="${#dates.format(emp.birth,'yyyy-MM-dd HH:mm')}"></td>
   									<td>
   										<a class="btn btn-sm btn-primary" th:href="@{/emp/}+${emp.id}">编辑</a>
   										<form style="display: inline;" th:action="@{/emp/}+${emp.id}" method="post">
   											<input type="hidden" name="_method" value="delete">
   											<button type="submit" class="btn btn-sm btn-danger">删除</button>
   										</form>
   
   									</td>
   								</tr>
   							</tbody>
   						</table>
   					</div>
   				</main>
   			</div>
   		</div>
   
   		<!-- Bootstrap core JavaScript
       ================================================== -->
   		<!-- Placed at the end of the document so the pages load faster -->
   		<script type="text/javascript" src="../../static/asserts/js/jquery-3.2.1.slim.min.js"></script>
   		<script type="text/javascript" src="../../static/asserts/js/popper.min.js"></script>
   		<script type="text/javascript" src="../../static/asserts/js/bootstrap.min.js"></script>
   
   		<!-- Icons -->
   		<script type="text/javascript" src="../../static/asserts/js/feather.min.js"></script>
   		<script>
   			feather.replace()
   		</script>
   
   		<!-- Graphs -->
   		<script type="text/javascript" src="../../static/asserts/js/Chart.min.js"></script>
   		<script>
   			var ctx = document.getElementById("myChart");
   			var myChart = new Chart(ctx, {
   				type: 'line',
   				data: {
   					labels: ["Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"],
   					datasets: [{
   						data: [15339, 21345, 18483, 24003, 23489, 24092, 12034],
   						lineTension: 0,
   						backgroundColor: 'transparent',
   						borderColor: '#007bff',
   						borderWidth: 4,
   						pointBackgroundColor: '#007bff'
   					}]
   				},
   				options: {
   					scales: {
   						yAxes: [{
   							ticks: {
   								beginAtZero: false
   							}
   						}]
   					},
   					legend: {
   						display: false,
   					}
   				}
   			});
   		</script>
   
   	</body>
   
   </html>
   ```

### 3.CRUD-员工修改

> put 和 delete 需要设置
>
> ```properties
> #配置put delete请求
> spring.mvc.hiddenmethod.filter.enabled=true
> ```

1. 前端首页

   > 因为需要获取修改员工的ID 所以需要传递参数

   ```html
   <a class="btn btn-sm btn-primary" th:href="@{/emp/}+${emp.id}">编辑</a>
   ```

2. Controller

   ```java
   //    查询出当前员工 来到表单页面
       @GetMapping("/emp/{id}")
       public String toEditPage(@PathVariable("id") Integer id, Model model){
   //        查询出当前员工
           Employee employee = employeeDao.get(id);
           model.addAttribute("emp",employee);
   //        页面要显示的所有部门列表
           Collection<Department> departments = departmentDao.getDepartments();
           model.addAttribute("departments",departments);
   //        回到修改页面
           return "emp/add";
       }
   ```

3. 前端修改页面

   关键代码：

   ```html
   <!--            区分是员工修改 还是添加 post 添加 -->
               <form th:action="@{/emp}" method="post">
   <!--                put 修改-->
   <!--
                   1.SpringMVC中配置HiddenHttpMethodFilter
                   2.页面创建post表单
                   3.创建input name = "_method" value = 指定的请求方式
                   -->
                   <input th:if="${emp!=null}" type="hidden" name="_method" value="put">
                   <input th:if="${emp!=null}" type="hidden" name="id" th:value="${emp.id}">
   ```

   全部代码：

   > 增加和修改功能同时实现
   >
   > 通过   ${emp!=null} 判断

   ```html
   <!DOCTYPE html>
   <!-- saved from url=(0052)http://getbootstrap.com/docs/4.0/examples/dashboard/ -->
   <html lang="en" xmlns:th="http://www.thymeleaf.org">
   
   <head>
       <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
       <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
       <meta name="description" content="">
       <meta name="author" content="">
   
       <title>Dashboard Template for Bootstrap</title>
       <!-- Bootstrap core CSS -->
       <link th:href="@{/webjars/bootstrap/4.0.0/css/bootstrap.css}" href="../../static/asserts/css/bootstrap.min.css" rel="stylesheet">
   
       <!-- Custom styles for this template -->
       <link th:href="@{/asserts/css/dashboard.css}" href="../../static/asserts/css/dashboard.css" rel="stylesheet">
       <style type="text/css">
           /* Chart.js */
   
           @-webkit-keyframes chartjs-render-animation {
               from {
                   opacity: 0.99
               }
               to {
                   opacity: 1
               }
           }
   
           @keyframes chartjs-render-animation {
               from {
                   opacity: 0.99
               }
               to {
                   opacity: 1
               }
           }
   
           .chartjs-render-monitor {
               -webkit-animation: chartjs-render-animation 0.001s;
               animation: chartjs-render-animation 0.001s;
           }
       </style>
   </head>
   
   <body>
   <!--		引入抽取的topbar-->
   <!--		模板名：使用thymeleaf的前后缀配置规则进行解析-->
   <div th:replace="~{commons/bar::topbar}"></div>
   <div class="container-fluid">
       <div class="row">
           <!--				引入侧边栏-->
           <div th:replace="commons/bar::#sidebar(activeUri='emps.html')"></div>
   
           <main role="main" class="col-md-9 ml-sm-auto col-lg-10 pt-3 px-4">
   <!--            区分是员工修改 还是添加 post 添加 -->
               <form th:action="@{/emp}" method="post">
   <!--                put 修改-->
   <!--
                   1.SpringMVC中配置HiddenHttpMethodFilter
                   2.页面创建post表单
                   3.创建input name = "_method" value = 指定的请求方式
                   -->
                   <input th:if="${emp!=null}" type="hidden" name="_method" value="put">
                   <input th:if="${emp!=null}" type="hidden" name="id" th:value="${emp.id}">
                   <div class="form-group">
                       <label>LastName</label>
                       <input th:value="${emp!=null}?${emp.lastName}" name="lastName" type="text" class="form-control" placeholder="zhangsan">
                   </div>
                   <div class="form-group">
                       <label>Email</label>
                       <input th:value="${emp!=null}?${emp.email}" name="email" type="email" class="form-control" placeholder="zhangsan@atguigu.com">
                   </div>
                   <div class="form-group">
                       <label>Gender</label><br/>
                       <div class="form-check form-check-inline">
                           <input th:checked="${emp!=null}?${emp.gender==1}" class="form-check-input" type="radio" name="gender"  value="1">
                           <label class="form-check-label">男</label>
                       </div>
                       <div class="form-check form-check-inline">
                           <input th:checked="${emp!=null}?${emp.gender==0}" class="form-check-input" type="radio" name="gender"  value="0">
                           <label class="form-check-label">女</label>
                       </div>
                   </div>
                   <div class="form-group">
                       <label>department</label>
                       <select name="department.id" class="form-control">
                           <option th:selected="${emp!=null}?${department.id == emp.department.id}" th:value="${department.id}" th:each="department:${departments}" th:text="${department.departmentName}">1</option>
                       </select>
                   </div>
                   <div class="form-group">
                       <label>Birth</label>
                       <input th:value="${emp!=null}?${#dates.format(emp.birth,'yyyy-MM-dd HH:mm')}" name="birth" type="text" class="form-control" placeholder="1999-02-14">
                   </div>
                   <button th:text="${emp!=null}?'修改':'添加'" type="submit" class="btn btn-primary">添加</button>
               </form>
           </main>
       </div>
   </div>
   
   <!-- Bootstrap core JavaScript
   ================================================== -->
   <!-- Placed at the end of the document so the pages load faster -->
   <script type="text/javascript" src="../../static/asserts/js/jquery-3.2.1.slim.min.js"></script>
   <script type="text/javascript" src="../../static/asserts/js/popper.min.js"></script>
   <script type="text/javascript" src="../../static/asserts/js/bootstrap.min.js"></script>
   
   <!-- Icons -->
   <script type="text/javascript" src="../../static/asserts/js/feather.min.js"></script>
   <script>
       feather.replace()
   </script>
   
   <!-- Graphs -->
   <script type="text/javascript" src="../../static/asserts/js/Chart.min.js"></script>
   <script>
       var ctx = document.getElementById("myChart");
       var myChart = new Chart(ctx, {
           type: 'line',
           data: {
               labels: ["Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"],
               datasets: [{
                   data: [15339, 21345, 18483, 24003, 23489, 24092, 12034],
                   lineTension: 0,
                   backgroundColor: 'transparent',
                   borderColor: '#007bff',
                   borderWidth: 4,
                   pointBackgroundColor: '#007bff'
               }]
           },
           options: {
               scales: {
                   yAxes: [{
                       ticks: {
                           beginAtZero: false
                       }
                   }]
               },
               legend: {
                   display: false,
               }
           }
       });
   </script>
   
   </body>
   
   </html>
   ```

4. Controller

   ```java
   //    员工修改,需要id
       @PutMapping("/emp")
       public String updateEmployee(Employee employee){
           employeeDao.save(employee);
           //去往查询功能
           return "redirect:/emps";
       }
   ```

### 4.CRUD-员工删除

1. 前端首页

   ```html
   <form style="display: inline;" th:action="@{/emp/}+${emp.id}" method="post">
   											<input type="hidden" name="_method" value="delete">
   											<button type="submit" class="btn btn-sm btn-danger">删除</button>
   										</form>
   ```

2. Controller

   ```java
   //  员工删除
       @DeleteMapping("/emp/{id}")
       public  String deleteEmployeeById(@PathVariable("id") Integer id){
           employeeDao.delete(id);
           return "redirect:/emps";
       }
   ```


## 六.错误机制处理

### 1.SpringBoot默认的错误处理机制

```java
 @ResponseBody
    @RequestMapping("/hello")
    public String hello(@RequestParam("user") String user){
        if(user.equals("aaa")){
            throw new UserNotExistException();
        }
        return "Hello World";
    }
```

1. 浏览器，返回一个默认的错误页面

2. 如果是其他客户端，默认响应一个json数据

   > 通过请求头判断

### 2.原理

可以参照ErrorMvcAutoConfiguration；错误处理的自动配置；

  	给容器中添加了以下组件

​	1、DefaultErrorAttributes：

```java
帮我们在页面共享信息；
@Override
	public Map<String, Object> getErrorAttributes(RequestAttributes requestAttributes,
			boolean includeStackTrace) {
		Map<String, Object> errorAttributes = new LinkedHashMap<String, Object>();
		errorAttributes.put("timestamp", new Date());
		addStatus(errorAttributes, requestAttributes);
		addErrorDetails(errorAttributes, requestAttributes, includeStackTrace);
		addPath(errorAttributes, requestAttributes);
		return errorAttributes;
	}
```



​	2、BasicErrorController：处理默认/error请求

```java
@Controller
@RequestMapping("${server.error.path:${error.path:/error}}")
public class BasicErrorController extends AbstractErrorController {
    
    @RequestMapping(produces = "text/html")//产生html类型的数据；浏览器发送的请求来到这个方法处理
	public ModelAndView errorHtml(HttpServletRequest request,
			HttpServletResponse response) {
		HttpStatus status = getStatus(request);
		Map<String, Object> model = Collections.unmodifiableMap(getErrorAttributes(
				request, isIncludeStackTrace(request, MediaType.TEXT_HTML)));
		response.setStatus(status.value());
        
        //去哪个页面作为错误页面；包含页面地址和页面内容
		ModelAndView modelAndView = resolveErrorView(request, response, status, model);
		return (modelAndView == null ? new ModelAndView("error", model) : modelAndView);
	}

	@RequestMapping
	@ResponseBody    //产生json数据，其他客户端来到这个方法处理；
	public ResponseEntity<Map<String, Object>> error(HttpServletRequest request) {
		Map<String, Object> body = getErrorAttributes(request,
				isIncludeStackTrace(request, MediaType.ALL));
		HttpStatus status = getStatus(request);
		return new ResponseEntity<Map<String, Object>>(body, status);
	}
```



​	3、ErrorPageCustomizer：

```java
	@Value("${error.path:/error}")
	private String path = "/error";  系统出现错误以后来到error请求进行处理；（web.xml注册的错误页面规则）
```



​	4、DefaultErrorViewResolver：

```java
@Override
	public ModelAndView resolveErrorView(HttpServletRequest request, HttpStatus status,
			Map<String, Object> model) {
		ModelAndView modelAndView = resolve(String.valueOf(status), model);
		if (modelAndView == null && SERIES_VIEWS.containsKey(status.series())) {
			modelAndView = resolve(SERIES_VIEWS.get(status.series()), model);
		}
		return modelAndView;
	}

	private ModelAndView resolve(String viewName, Map<String, Object> model) {
        //默认SpringBoot可以去找到一个页面？  error/404
		String errorViewName = "error/" + viewName;
        
        //模板引擎可以解析这个页面地址就用模板引擎解析
		TemplateAvailabilityProvider provider = this.templateAvailabilityProviders
				.getProvider(errorViewName, this.applicationContext);
		if (provider != null) {
            //模板引擎可用的情况下返回到errorViewName指定的视图地址
			return new ModelAndView(errorViewName, model);
		}
        //模板引擎不可用，就在静态资源文件夹下找errorViewName对应的页面   error/404.html
		return resolveResource(errorViewName, model);
	}
```



​	步骤：

​		一但系统出现4xx或者5xx之类的错误；ErrorPageCustomizer就会生效（定制错误的响应规则）；就会来到/error请求；就会被**BasicErrorController**处理；

​		1）响应页面；去哪个页面是由**DefaultErrorViewResolver**解析得到的；

```java
protected ModelAndView resolveErrorView(HttpServletRequest request,
      HttpServletResponse response, HttpStatus status, Map<String, Object> model) {
    //所有的ErrorViewResolver得到ModelAndView
   for (ErrorViewResolver resolver : this.errorViewResolvers) {
      ModelAndView modelAndView = resolver.resolveErrorView(request, status, model);
      if (modelAndView != null) {
         return modelAndView;
      }
   }
   return null;
}
```

### 3.定制错误响应

#### 定制错误页面

1. 有模板引擎的情况下

   error/状态码;** 【将错误页面命名为  错误状态码.html 放在模板引擎文件夹里面的 error文件夹下】，发生此状态码的错误就会来到  对应的页面；

   ​			我们可以使用4xx和5xx作为错误页面的文件名来匹配这种类型的所有错误，精确优先（优先寻找精确的状态码.html）；		

   ​			页面能获取的信息；

   ​				timestamp：时间戳

   ​				status：状态码

   ​				error：错误提示

   ​				exception：异常对象

   ​				message：异常消息

   ​				errors：JSR303数据校验的错误都在这里

2. 没有模板引擎（模板引擎找不到这个错误页面），静态资源文件夹下找；

3. 以上都没有错误页面，就是默认来到SpringBoot默认的错误提示页面；

#### 定制错误的json数据

1. 浏览器和客户端返回都是json

```java
@ControllerAdvice
public class MyExceptionHandler {
//    1.浏览器和客户端返回都是json
    @ResponseBody
    @ExceptionHandler(UserNotExistException.class)
    public Map<String,Object> handlerException(Exception e){
        Map<String,Object> map = new HashMap<>();
       map.put("code","user.notexist");
        map.put("message",e.getMessage());
        return map;
    }
}

```

2. 转发到/error进行自适应响应效果处理

   > 将数据放在request域中

   ```java
       @ExceptionHandler(UserNotExistException.class)
       public String handlerException(Exception e, HttpServletRequest request){
           Map<String,Object> map = new HashMap<>();
           request.setAttribute("javax.servlet.error.status_code",500);
           map.put("code","user.notexist");
           map.put("message",e.getMessage());
           request.setAttribute("ext",map);
           return "forward:/error";
       }
   ```

#### 将我们的定制数据携带出去

> 自定义一个类 继承 DefaultErrorAttributes
>
> 获取  Map<String, Object> ext = (Map<String, Object>) webRequest.getAttribute("ext",0);
>
> 存放在map中

```java
@Component
public class MyErrorAttributes extends DefaultErrorAttributes {
//    返回的map就是页面和json能获取的字段
    @Override
    public Map<String, Object> getErrorAttributes(WebRequest webRequest, boolean includeStackTrace) {
        Map<String, Object> map = super.getErrorAttributes(webRequest, includeStackTrace);
        map.put("jsh","jsh");
//        异常处理器携带的数据
        Map<String, Object> ext = (Map<String, Object>) webRequest.getAttribute("ext",0);
        map.put("ext",ext);
        return map;
    }
}
```

## 七.Servlet容器

### 1）、如何定制和修改Servlet容器的相关配置；

1、修改和server有关的配置（ServerProperties【也是EmbeddedServletContainerCustomizer】）；

```properties
server.port=8081
server.context-path=/crud

server.tomcat.uri-encoding=UTF-8

//通用的Servlet容器设置
server.xxx
//Tomcat的设置
server.tomcat.xxx
```

2、编写一个**EmbeddedServletContainerCustomizer**：嵌入式的Servlet容器的定制器；来修改Servlet容器的配置

```java
//定制嵌入式的servlet容器
    @Bean
    public WebServerFactoryCustomizer<ConfigurableWebServerFactory> webServerFactoryCustomizer(){
        return new WebServerFactoryCustomizer<ConfigurableWebServerFactory>() {
            //            定制嵌入式的servlet容器相关的规则
            @Override
            public void customize(ConfigurableWebServerFactory factory) {
                factory.setPort(8083);
            }
        };
    }
```

### 2）、注册Servlet三大组件【Servlet、Filter、Listener】

由于SpringBoot默认是以jar包的方式启动嵌入式的Servlet容器来启动SpringBoot的web应用，没有web.xml文件。

注册三大组件用以下方式

ServletRegistrationBean

```java
//注册三大组件
@Bean
public ServletRegistrationBean myServlet(){
    ServletRegistrationBean registrationBean = new ServletRegistrationBean(new MyServlet(),"/myServlet");
    return registrationBean;
}

```

FilterRegistrationBean

```java
@Bean
public FilterRegistrationBean myFilter(){
    FilterRegistrationBean registrationBean = new FilterRegistrationBean();
    registrationBean.setFilter(new MyFilter());
    registrationBean.setUrlPatterns(Arrays.asList("/hello","/myServlet"));
    return registrationBean;
}
```

ServletListenerRegistrationBean

```java
@Bean
public ServletListenerRegistrationBean myListener(){
    ServletListenerRegistrationBean<MyListener> registrationBean = new ServletListenerRegistrationBean<>(new MyListener());
    return registrationBean;
}
```



SpringBoot帮我们自动SpringMVC的时候，自动的注册SpringMVC的前端控制器；DIspatcherServlet；

DispatcherServletAutoConfiguration中：

```java
@Bean(name = DEFAULT_DISPATCHER_SERVLET_REGISTRATION_BEAN_NAME)
@ConditionalOnBean(value = DispatcherServlet.class, name = DEFAULT_DISPATCHER_SERVLET_BEAN_NAME)
public ServletRegistrationBean dispatcherServletRegistration(
      DispatcherServlet dispatcherServlet) {
   ServletRegistrationBean registration = new ServletRegistrationBean(
         dispatcherServlet, this.serverProperties.getServletMapping());
    //默认拦截： /  所有请求；包静态资源，但是不拦截jsp请求；   /*会拦截jsp
    //可以通过server.servletPath来修改SpringMVC前端控制器默认拦截的请求路径
    
   registration.setName(DEFAULT_DISPATCHER_SERVLET_BEAN_NAME);
   registration.setLoadOnStartup(
         this.webMvcProperties.getServlet().getLoadOnStartup());
   if (this.multipartConfig != null) {
      registration.setMultipartConfig(this.multipartConfig);
   }
   return registration;
}

```

2）、SpringBoot能不能支持其他的Servlet容器；

### 3）、替换为其他嵌入式Servlet容器

![](D:/系统/桌面/images/搜狗截图20180302114401.png)

默认支持：

Tomcat（默认使用）

```xml
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-web</artifactId>
   引入web模块默认就是使用嵌入式的Tomcat作为Servlet容器；
</dependency>
```

Jetty

```xml
<!-- 引入web模块 -->
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-web</artifactId>
   <exclusions>
      <exclusion>
         <artifactId>spring-boot-starter-tomcat</artifactId>
         <groupId>org.springframework.boot</groupId>
      </exclusion>
   </exclusions>
</dependency>

<!--引入其他的Servlet容器-->
<dependency>
   <artifactId>spring-boot-starter-jetty</artifactId>
   <groupId>org.springframework.boot</groupId>
</dependency>
```

Undertow

```xml
<!-- 引入web模块 -->
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-web</artifactId>
   <exclusions>
      <exclusion>
         <artifactId>spring-boot-starter-tomcat</artifactId>
         <groupId>org.springframework.boot</groupId>
      </exclusion>
   </exclusions>
</dependency>

<!--引入其他的Servlet容器-->
<dependency>
   <artifactId>spring-boot-starter-undertow</artifactId>
   <groupId>org.springframework.boot</groupId>
</dependency>
```

### 4）、嵌入式Servlet容器自动配置原理；



EmbeddedServletContainerAutoConfiguration：嵌入式的Servlet容器自动配置？

```java
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE)
@Configuration
@ConditionalOnWebApplication
@Import(BeanPostProcessorsRegistrar.class)
//导入BeanPostProcessorsRegistrar：Spring注解版；给容器中导入一些组件
//导入了EmbeddedServletContainerCustomizerBeanPostProcessor：
//后置处理器：bean初始化前后（创建完对象，还没赋值赋值）执行初始化工作
public class EmbeddedServletContainerAutoConfiguration {
    
    @Configuration
	@ConditionalOnClass({ Servlet.class, Tomcat.class })//判断当前是否引入了Tomcat依赖；
	@ConditionalOnMissingBean(value = EmbeddedServletContainerFactory.class, search = SearchStrategy.CURRENT)//判断当前容器没有用户自己定义EmbeddedServletContainerFactory：嵌入式的Servlet容器工厂；作用：创建嵌入式的Servlet容器
	public static class EmbeddedTomcat {

		@Bean
		public TomcatEmbeddedServletContainerFactory tomcatEmbeddedServletContainerFactory() {
			return new TomcatEmbeddedServletContainerFactory();
		}

	}
    
    /**
	 * Nested configuration if Jetty is being used.
	 */
	@Configuration
	@ConditionalOnClass({ Servlet.class, Server.class, Loader.class,
			WebAppContext.class })
	@ConditionalOnMissingBean(value = EmbeddedServletContainerFactory.class, search = SearchStrategy.CURRENT)
	public static class EmbeddedJetty {

		@Bean
		public JettyEmbeddedServletContainerFactory jettyEmbeddedServletContainerFactory() {
			return new JettyEmbeddedServletContainerFactory();
		}

	}

	/**
	 * Nested configuration if Undertow is being used.
	 */
	@Configuration
	@ConditionalOnClass({ Servlet.class, Undertow.class, SslClientAuthMode.class })
	@ConditionalOnMissingBean(value = EmbeddedServletContainerFactory.class, search = SearchStrategy.CURRENT)
	public static class EmbeddedUndertow {

		@Bean
		public UndertowEmbeddedServletContainerFactory undertowEmbeddedServletContainerFactory() {
			return new UndertowEmbeddedServletContainerFactory();
		}

	}
```

1）、EmbeddedServletContainerFactory（嵌入式Servlet容器工厂）

```java
public interface EmbeddedServletContainerFactory {

   //获取嵌入式的Servlet容器
   EmbeddedServletContainer getEmbeddedServletContainer(
         ServletContextInitializer... initializers);

}
```

![](D:/系统/桌面/images/搜狗截图20180302144835.png)

2）、EmbeddedServletContainer：（嵌入式的Servlet容器）

![](D:/系统/桌面/images/搜狗截图20180302144910.png)



3）、以**TomcatEmbeddedServletContainerFactory**为例

```java
@Override
public EmbeddedServletContainer getEmbeddedServletContainer(
      ServletContextInitializer... initializers) {
    //创建一个Tomcat
   Tomcat tomcat = new Tomcat();
    
    //配置Tomcat的基本环节
   File baseDir = (this.baseDirectory != null ? this.baseDirectory
         : createTempDir("tomcat"));
   tomcat.setBaseDir(baseDir.getAbsolutePath());
   Connector connector = new Connector(this.protocol);
   tomcat.getService().addConnector(connector);
   customizeConnector(connector);
   tomcat.setConnector(connector);
   tomcat.getHost().setAutoDeploy(false);
   configureEngine(tomcat.getEngine());
   for (Connector additionalConnector : this.additionalTomcatConnectors) {
      tomcat.getService().addConnector(additionalConnector);
   }
   prepareContext(tomcat.getHost(), initializers);
    
    //将配置好的Tomcat传入进去，返回一个EmbeddedServletContainer；并且启动Tomcat服务器
   return getTomcatEmbeddedServletContainer(tomcat);
}
```

4）、我们对嵌入式容器的配置修改是怎么生效？

```
ServerProperties、EmbeddedServletContainerCustomizer
```



**EmbeddedServletContainerCustomizer**：定制器帮我们修改了Servlet容器的配置？

怎么修改的原理？

5）、容器中导入了**EmbeddedServletContainerCustomizerBeanPostProcessor**

```java
//初始化之前
@Override
public Object postProcessBeforeInitialization(Object bean, String beanName)
      throws BeansException {
    //如果当前初始化的是一个ConfigurableEmbeddedServletContainer类型的组件
   if (bean instanceof ConfigurableEmbeddedServletContainer) {
       //
      postProcessBeforeInitialization((ConfigurableEmbeddedServletContainer) bean);
   }
   return bean;
}

private void postProcessBeforeInitialization(
			ConfigurableEmbeddedServletContainer bean) {
    //获取所有的定制器，调用每一个定制器的customize方法来给Servlet容器进行属性赋值；
    for (EmbeddedServletContainerCustomizer customizer : getCustomizers()) {
        customizer.customize(bean);
    }
}

private Collection<EmbeddedServletContainerCustomizer> getCustomizers() {
    if (this.customizers == null) {
        // Look up does not include the parent context
        this.customizers = new ArrayList<EmbeddedServletContainerCustomizer>(
            this.beanFactory
            //从容器中获取所有这葛类型的组件：EmbeddedServletContainerCustomizer
            //定制Servlet容器，给容器中可以添加一个EmbeddedServletContainerCustomizer类型的组件
            .getBeansOfType(EmbeddedServletContainerCustomizer.class,
                            false, false)
            .values());
        Collections.sort(this.customizers, AnnotationAwareOrderComparator.INSTANCE);
        this.customizers = Collections.unmodifiableList(this.customizers);
    }
    return this.customizers;
}

ServerProperties也是定制器
```

步骤：

1）、SpringBoot根据导入的依赖情况，给容器中添加相应的EmbeddedServletContainerFactory【TomcatEmbeddedServletContainerFactory】

2）、容器中某个组件要创建对象就会惊动后置处理器；EmbeddedServletContainerCustomizerBeanPostProcessor；

只要是嵌入式的Servlet容器工厂，后置处理器就工作；

3）、后置处理器，从容器中获取所有的**EmbeddedServletContainerCustomizer**，调用定制器的定制方法



###5）、嵌入式Servlet容器启动原理；

什么时候创建嵌入式的Servlet容器工厂？什么时候获取嵌入式的Servlet容器并启动Tomcat；

获取嵌入式的Servlet容器工厂：

1）、SpringBoot应用启动运行run方法

2）、refreshContext(context);SpringBoot刷新IOC容器【创建IOC容器对象，并初始化容器，创建容器中的每一个组件】；如果是web应用创建**AnnotationConfigEmbeddedWebApplicationContext**，否则：**AnnotationConfigApplicationContext**

3）、refresh(context);**刷新刚才创建好的ioc容器；**

```java
public void refresh() throws BeansException, IllegalStateException {
   synchronized (this.startupShutdownMonitor) {
      // Prepare this context for refreshing.
      prepareRefresh();

      // Tell the subclass to refresh the internal bean factory.
      ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

      // Prepare the bean factory for use in this context.
      prepareBeanFactory(beanFactory);

      try {
         // Allows post-processing of the bean factory in context subclasses.
         postProcessBeanFactory(beanFactory);

         // Invoke factory processors registered as beans in the context.
         invokeBeanFactoryPostProcessors(beanFactory);

         // Register bean processors that intercept bean creation.
         registerBeanPostProcessors(beanFactory);

         // Initialize message source for this context.
         initMessageSource();

         // Initialize event multicaster for this context.
         initApplicationEventMulticaster();

         // Initialize other special beans in specific context subclasses.
         onRefresh();

         // Check for listener beans and register them.
         registerListeners();

         // Instantiate all remaining (non-lazy-init) singletons.
         finishBeanFactoryInitialization(beanFactory);

         // Last step: publish corresponding event.
         finishRefresh();
      }

      catch (BeansException ex) {
         if (logger.isWarnEnabled()) {
            logger.warn("Exception encountered during context initialization - " +
                  "cancelling refresh attempt: " + ex);
         }

         // Destroy already created singletons to avoid dangling resources.
         destroyBeans();

         // Reset 'active' flag.
         cancelRefresh(ex);

         // Propagate exception to caller.
         throw ex;
      }

      finally {
         // Reset common introspection caches in Spring's core, since we
         // might not ever need metadata for singleton beans anymore...
         resetCommonCaches();
      }
   }
}
```

4）、  onRefresh(); web的ioc容器重写了onRefresh方法

5）、webioc容器会创建嵌入式的Servlet容器；**createEmbeddedServletContainer**();

**6）、获取嵌入式的Servlet容器工厂：**

EmbeddedServletContainerFactory containerFactory = getEmbeddedServletContainerFactory();

​	从ioc容器中获取EmbeddedServletContainerFactory 组件；**TomcatEmbeddedServletContainerFactory**创建对象，后置处理器一看是这个对象，就获取所有的定制器来先定制Servlet容器的相关配置；

7）、**使用容器工厂获取嵌入式的Servlet容器**：this.embeddedServletContainer = containerFactory      .getEmbeddedServletContainer(getSelfInitializer());

8）、嵌入式的Servlet容器创建对象并启动Servlet容器；

**先启动嵌入式的Servlet容器，再将ioc容器中剩下没有创建出的对象获取出来；**

**==IOC容器启动创建嵌入式的Servlet容器==**



## 9、使用外置的Servlet容器

嵌入式Servlet容器：应用打成可执行的jar

​		优点：简单、便携；

​		缺点：默认不支持JSP、优化定制比较复杂（使用定制器【ServerProperties、自定义EmbeddedServletContainerCustomizer】，自己编写嵌入式Servlet容器的创建工厂【EmbeddedServletContainerFactory】）；



外置的Servlet容器：外面安装Tomcat---应用war包的方式打包；

### 步骤

1）、必须创建一个war项目；（利用idea创建好目录结构）

2）、将嵌入式的Tomcat指定为provided；

```xml
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-tomcat</artifactId>
   <scope>provided</scope>
</dependency>
```

3）、必须编写一个**SpringBootServletInitializer**的子类，并调用configure方法

```java
public class ServletInitializer extends SpringBootServletInitializer {

   @Override
   protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
       //传入SpringBoot应用的主程序
      return application.sources(SpringBoot04WebJspApplication.class);
   }

}
```

4）、启动服务器就可以使用；

### 原理

jar包：执行SpringBoot主类的main方法，启动ioc容器，创建嵌入式的Servlet容器；

war包：启动服务器，**服务器启动SpringBoot应用**【SpringBootServletInitializer】，启动ioc容器；



servlet3.0（Spring注解版）：

8.2.4 Shared libraries / runtimes pluggability：

规则：

​	1）、服务器启动（web应用启动）会创建当前web应用里面每一个jar包里面ServletContainerInitializer实例：

​	2）、ServletContainerInitializer的实现放在jar包的META-INF/services文件夹下，有一个名为javax.servlet.ServletContainerInitializer的文件，内容就是ServletContainerInitializer的实现类的全类名

​	3）、还可以使用@HandlesTypes，在应用启动的时候加载我们感兴趣的类；



流程：

1）、启动Tomcat

2）、org\springframework\spring-web\4.3.14.RELEASE\spring-web-4.3.14.RELEASE.jar!\META-INF\services\javax.servlet.ServletContainerInitializer：

Spring的web模块里面有这个文件：**org.springframework.web.SpringServletContainerInitializer**

3）、SpringServletContainerInitializer将@HandlesTypes(WebApplicationInitializer.class)标注的所有这个类型的类都传入到onStartup方法的Set<Class<?>>；为这些WebApplicationInitializer类型的类创建实例；

4）、每一个WebApplicationInitializer都调用自己的onStartup；

![](D:/系统/桌面/images/搜狗截图20180302221835.png)

5）、相当于我们的SpringBootServletInitializer的类会被创建对象，并执行onStartup方法

6）、SpringBootServletInitializer实例执行onStartup的时候会createRootApplicationContext；创建容器

```java
protected WebApplicationContext createRootApplicationContext(
      ServletContext servletContext) {
    //1、创建SpringApplicationBuilder
   SpringApplicationBuilder builder = createSpringApplicationBuilder();
   StandardServletEnvironment environment = new StandardServletEnvironment();
   environment.initPropertySources(servletContext, null);
   builder.environment(environment);
   builder.main(getClass());
   ApplicationContext parent = getExistingRootWebApplicationContext(servletContext);
   if (parent != null) {
      this.logger.info("Root context already created (using as parent).");
      servletContext.setAttribute(
            WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE, null);
      builder.initializers(new ParentContextApplicationContextInitializer(parent));
   }
   builder.initializers(
         new ServletContextApplicationContextInitializer(servletContext));
   builder.contextClass(AnnotationConfigEmbeddedWebApplicationContext.class);
    
    //调用configure方法，子类重写了这个方法，将SpringBoot的主程序类传入了进来
   builder = configure(builder);
    
    //使用builder创建一个Spring应用
   SpringApplication application = builder.build();
   if (application.getSources().isEmpty() && AnnotationUtils
         .findAnnotation(getClass(), Configuration.class) != null) {
      application.getSources().add(getClass());
   }
   Assert.state(!application.getSources().isEmpty(),
         "No SpringApplication sources have been defined. Either override the "
               + "configure method or add an @Configuration annotation");
   // Ensure error pages are registered
   if (this.registerErrorPageFilter) {
      application.getSources().add(ErrorPageFilterConfiguration.class);
   }
    //启动Spring应用
   return run(application);
}
```

7）、Spring的应用就启动并且创建IOC容器

```java
public ConfigurableApplicationContext run(String... args) {
   StopWatch stopWatch = new StopWatch();
   stopWatch.start();
   ConfigurableApplicationContext context = null;
   FailureAnalyzers analyzers = null;
   configureHeadlessProperty();
   SpringApplicationRunListeners listeners = getRunListeners(args);
   listeners.starting();
   try {
      ApplicationArguments applicationArguments = new DefaultApplicationArguments(
            args);
      ConfigurableEnvironment environment = prepareEnvironment(listeners,
            applicationArguments);
      Banner printedBanner = printBanner(environment);
      context = createApplicationContext();
      analyzers = new FailureAnalyzers(context);
      prepareContext(context, environment, listeners, applicationArguments,
            printedBanner);
       
       //刷新IOC容器
      refreshContext(context);
      afterRefresh(context, applicationArguments);
      listeners.finished(context, null);
      stopWatch.stop();
      if (this.logStartupInfo) {
         new StartupInfoLogger(this.mainApplicationClass)
               .logStarted(getApplicationLog(), stopWatch);
      }
      return context;
   }
   catch (Throwable ex) {
      handleRunFailure(context, listeners, analyzers, ex);
      throw new IllegalStateException(ex);
   }
}
```

**==启动Servlet容器，再启动SpringBoot应用==**

