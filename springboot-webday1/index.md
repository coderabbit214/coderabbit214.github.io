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

```
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


