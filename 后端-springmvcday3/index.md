# 后端-SpringMVC(day3)


# 后端-SpringMVC(day3)

## REST风格

Springmvc:  

GET  :查

POST  ：增

DELETE ：删

PUT ：改

普通浏览器 只支持get post方式 ；其他请求方式 如 delelte|put请求是通过 过滤器新加入的支持。

## 实现

1. 增加过滤器 web.xml

   ```xml
      <filter>
           <filter-name>HiddenHttpMethodFilter</filter-name>
           <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
       </filter>
       <filter-mapping>
           <filter-name>HiddenHttpMethodFilter</filter-name>
           <url-pattern>/*</url-pattern>
       </filter-mapping>
   ```

2. 表单 

   > DELETE ：删
   >
   > PUT ：改
   >
   > 必须是post方式
   > 通过隐藏域 的value值 设置实际的请求方式 DELETE|PUT

```html
  <form action="SpringMVCHandler/testPost/1234" method="post">
    <input type="submit" value="增">
  </form>
  <form action="SpringMVCHandler/testDelete/1234" method="post">
    <input type="hidden" name="_method" value="DELETE">
    <input type="submit" value="删">
  </form>
  <form action="SpringMVCHandler/testPut/1234" method="post">
    <input type="hidden" name="_method" value="PUT">
    <input type="submit" value="改">
  </form>
  <form action="SpringMVCHandler/testGet/1234" method="get">
    <input type="submit" value="差">
  </form>
```

3. 控制器

   ```java
   
       @RequestMapping(value="testPost/{id}",method = RequestMethod.POST)
       public String testPost(@PathVariable("id") int id){
           System.out.println("增"+id);
           return "succes";  //  view/succes.jsp
       }
   
       @RequestMapping(value="testGet/{id}",method = RequestMethod.GET)
       public String testGet(@PathVariable("id") int id){
           System.out.println("查"+id);
           return "succes";  //  view/succes.jsp
       }
       @RequestMapping(value="testDelete/{id}",method = RequestMethod.DELETE)
       public String testDelete(@PathVariable("id") int id){
           System.out.println("删"+id);
           return "succes";  //  view/succes.jsp
       }
       @RequestMapping(value="testPut/{id}",method = RequestMethod.PUT)
       public String testPut(@PathVariable("id") int id){
           System.out.println("改"+id);
           return "succes";  //  view/succes.jsp
       }
   ```

   > 映射名相同时@RequestMapping(value="testRest)，可以通过method处理不同的请求。 重名时会调用 不同的请求方法


