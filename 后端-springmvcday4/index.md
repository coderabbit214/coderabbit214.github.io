# 后端-SpringMVC(day4)


# 后端-SpringMVC(day4)

使用注解接收参数

> SpringMVC处理各种参数的流程/逻辑：
> 请求：  前端发请求a-> @RequestMappting("a") 
> 处理请求中的参数xyz：
> 	@RequestMappting("a") 
> 	public String  aa(@Xxx注解("xyz")  xyz)
> 	{
>
> ​	}

```html
 <form action="SpringMVCHandler/testParam" method="get">
    <input type="text" name="uname">
    <input type="text" name="uage">
    <input type="submit" value="提交">
  </form>
```

```java
@RequestMapping(value="testParam")
    public String testParam(@RequestParam("uname") String name,@RequestParam(value = "uage",required = false,defaultValue = "23") int age){
        System.out.println(name+age);
        return "succes";  //  view/succes.jsp
    }
	//获取请求头
    @RequestMapping(value="testRequestHeader")
    public String testRequestHeader(@RequestHeader("Accept-Encoding") String al){
        System.out.println(al);
        return "succes";  //  view/succes.jsp
    }
	//获取Cookie
    @RequestMapping(value="testCookieValue")
    public String testCookieValue(@CookieValue("JSESSIONID") String al){
        System.out.println(al);
        return "succes";  //  view/succes.jsp
    }
```

## Object

> 直接将前端数据封装到实体类中

> 必须与实体类属性一一对应 支持级联

```html

  <form action="SpringMVCHandler/testObjectProperties" method="get">
    <input type="text" name="id">
    <input type="text" name="name">
    <input type="text" name="address.homeAddress">
    <input type="text" name="address.schoolAddress">
    <input type="submit" value="提交">
  </form>
```

```java
 @RequestMapping(value="testObjectProperties")
    public String testObjectProperties(Student student){//student属性必须和from表单中的属性Name一致 支持级联属性
        System.out.println(student);
        return "succes";  //  view/succes.jsp
    }
```

## 使用ServletAPI

> 直接写在参数中

```java
//使用servlet中的对象 直接传递参数
    @RequestMapping(value="testServletAPI")
    public String testServletAPI(HttpServletRequest request, HttpServletResponse response){
        request.getParameter("uname");
        return "succes";  //  view/succes.jsp
    }
```


