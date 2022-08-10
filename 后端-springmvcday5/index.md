# 后端-SpringMVC(day5)


# 后端-SpringMVC(day5)

## ModelAndView

> 如果跳转时需要带数据：V、M,则可以使用以下方式：
> ModelAndView、ModelMap  、Map、Model   -数据放在了request作用域 

### ModelAndView

> 方法的返回值变为 ModelAndView
>
>  ModelAndView modelAndView = new ModelAndView("succes");
>
> 使用addObject方法将数据加入到request域中

```java
 @RequestMapping(value="testModelAndView")
    public ModelAndView testModelAndView(){//ModelAndView既有数据又有视图
        ModelAndView modelAndView = new ModelAndView("succes"); //  view/succes.jsp
        Student student = new Student();
        student.setName("aaa");
        modelAndView.addObject("student",student);//相当于request.setAttribute("student",student)
        return modelAndView;
    }
```

### ModelMap  、Map、Model

> 三者基本一样
>
> 返回值为String类型
>
> 传递参数
>
> 使用方法加入到request域中

```java
@RequestMapping(value="testModelMap")
    public String testModelMap(ModelMap modelMap){//ModelAndView既有数据又有视图
        Student student = new Student();
        student.setName("aaa");
        modelMap.put("student",student);
        return "succes";
    }

    @RequestMapping(value="testModel")
    public String testModel(Model model){//ModelAndView既有数据又有视图
        Student student = new Student();
        student.setName("aaa");
        model.addAttribute("student",student);
        return "succes";
    }

    @RequestMapping(value="testMap")
    public String testMap(Map<String,Object> map){//ModelAndView既有数据又有视图

        Student student = new Student();
        student.setName("aaa");
        map.put("student",student);
        return "succes";
    }
```

## 注解@SessionAttributes

> 将数据存放在session域中 request中也会保存（从request中复制到session）
>
> 位置: 类前
>
> 多个值用大括号

```java
//@SessionAttributes("student") //存放student对象 request域中也有 复制了一份到session中
//@SessionAttributes(types = Student.class) //将Student类型的存放在session中
```

## 注解@ModelAttribute 

> i.经常在 更新时使用
> ii.在不改变原有代码的基础之上，插入一个新方法。

必须满足的约定：
map.put(k,v) 其中的k 必须是即将查询的方法参数 的首字母小写
testModelAttribute(Student xxx)  ，即student；
如果不一致，需要通过@ModelAttribute声明

```java
 @ModelAttribute//在执行任何一次请求前都会执行
//    在请求该类的方法前都被执行： 思想：一个控制器只做一个功能
    public void queryStudentById(Map<String,Object> map){
        //模拟查询
        Student student = new Student();
        student.setId(31);
        student.setName("zs");
        Address address = new Address();
        address.setHomeAddress("sss");
        address.setSchoolAddress("bbb");
        student.setAddress(address);
        //map.put("student",student);//约定：map的key 就是方法参数类型的首字母的小写
        map.put("stu",student);
    }

    @RequestMapping(value="testModelAttribute")
    public String testModelAttribute(@ModelAttribute("stu")Student student){//ModelAndView既有数据又有视图
        student.setName("ls");
        System.out.println(student);
        return "succes";
    }
```


