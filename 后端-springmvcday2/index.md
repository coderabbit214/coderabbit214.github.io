# 后端-SpringMVC(day2)


# 后端-SpringMVC(day2)

## @RequestMapping 参数

> 映射是 去匹配@RequestMapping注解，可以和方法名、类名不一致
>
> ```
> value="welcome" 映射
> method = RequestMethod.POST 规定提交方式
> params = {"name=zs","age!=23"} 对参数的限制
> headers 网络
> ```

```java
package org.jsh.handler;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

@Controller
@RequestMapping(value="SpringMVCHandler") //映射
public class SpringMVCHandler {
    @RequestMapping(value="welcome",
                    method = RequestMethod.POST,
                    params = {"name=zs","age!=23"})
    public String welcome(){
        return "succes";  //  view/succes.jsp
    }
    @RequestMapping(value="welcome2",
                    headers = {"Accept=text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9","Accept-Encoding= gzip, deflate, br"})
    public String welcome2(){
        return "succes";  //  view/succes.jsp
    }

    @RequestMapping(value="welcome3/*/test")
    public String welcome3(){
        return "succes";  //  view/succes.jsp
    }

}

```

```html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
  <head>
    <title>$Title$</title>
  </head>
  <body>
  $END$
  <a href="SpringMVCHandler/welcome3/ssssssssss/test">first springmvc</a>
  <form action="SpringMVCHandler/welcome" method="post">
    <input type="submit" value="post">
    <input name="name" value="zs">
  </form>
  </body>
</html>

```

##  ant风格的请求路径

?  单字符

*任意个字符（0或多个）

** 任意目录

## 动态参数

> 通过@PathVariable获取

```html
<a href="SpringMVCHandler/welcome4/zs">first springmvc</a>
```

```java
@RequestMapping(value="welcome4/{name}")
    public String welcome4(@PathVariable("name") String name){
        System.out.println(name);
        return "succes";  //  view/succes.jsp
    }
```


