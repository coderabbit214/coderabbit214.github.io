# 后端-SpringMVC(day11)-异常


# 后端-SpringMVC(day11)-异常

## @ExceptionHandler

> ExceptionHandlerExceptionResolver： 主要提供了@ExceptionHandler注解，并通过该注解处理异常
>
> @ExceptionHandler 对方法使用 可以捕获 本类中 抛出的异常
>
> 传递异常信息时不可以使用Map
>
> 如果出现两个方法都可以处理异常 最短优先

```java
 @ExceptionHandler({ArithmeticException.class,ArrayIndexOutOfBoundsException.class})
    public ModelAndView handlerArithmeticException(Exception e){
        System.out.println(e);
        ModelAndView modelAndView = new ModelAndView("error");
        modelAndView.addObject("e",e);
        return modelAndView;
    }
```

> 如果要使用该方法处理别的类中的异常
>
> 在类上使用注释@ControllerAdvice

```java
package org.jsh.exception;

import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.servlet.ModelAndView;

//该类中的任何处理异常的方法 可以处理任何类中的异常
@ControllerAdvice
public class MyExceptionHandler {// 不是控制器，处理异常的类
    @ExceptionHandler({ArithmeticException.class,ArrayIndexOutOfBoundsException.class})
    public ModelAndView handlerArithmeticException(Exception e){
        System.out.println(e);
        ModelAndView modelAndView = new ModelAndView("error");
        modelAndView.addObject("e",e);
        return modelAndView;
    }
} 
```

## @ResponseStatus

> ResponseStatusExceptionResolver：自定义异常显示页面 @ResponseStatus

1. 类的方法

```java
package org.jsh.exception;


import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.ResponseStatus;

@ResponseStatus(value = HttpStatus.FORBIDDEN,reason = "数组越界")
public class MyArrayIndexOutofBoundsException extends Exception {
}

```

使用

```java
 @RequestMapping("testMyException")
    public String testMyException(@RequestParam("i") int i) throws MyArrayIndexOutofBoundsException {
        if (i==3){
            throw new MyArrayIndexOutofBoundsException();
        }
        return "succes";
    }
```

2. 方法

   ```java
   @ResponseStatus(value = HttpStatus.FORBIDDEN,reason = "测试")
       @RequestMapping("testResponseStatus")
       public String testResponseStatus(){
           return "succes";
       }
   ```

   使用

   ```java
   @RequestMapping("testMyException2")
       public String testMyException2(@RequestParam("i") int i){
           if (i==3){
               return "redirect:testResponseStatus";
           }
           return "succes";
       }
   ```

## SimpleMappingExceptionResolver

> 通过配置来实现异常的处理

```xml
<!--    SimpleMappingExceptionResolver:以配置方式处理异常-->
    <bean class="org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">
<!--        将错误信息存放在request域中 如果不写 默认存放在exception中-->
<!--        <property name="exceptionAttribute" value="e">-->

<!--        </property>-->
        <property name="exceptionMappings">
            <props>
                <prop key="java.lang.ArithmeticException">
                    error
                </prop>
                <prop key="java.lang.NullPointerException">
                    error
                </prop>
            </props>
        </property>
    </bean>
```


