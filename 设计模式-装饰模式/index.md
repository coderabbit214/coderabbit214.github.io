# 设计模式-装饰模式


# 设计模式-装饰模式

> 装饰模式就是给一个对象增加一些新的功能，而且是动态的，要求装饰对象和被装饰对象实现同一个接口，装饰对象持有被装饰对象的实例

![1354196367_3730](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/1354196367_3730.PNG)

## 代码实现

> Source类是被装饰类，Decorator类是一个装饰类，可以为Source类动态的添加一些功能

```java
package com.designPattern.decoratorPattern;

/**
 * 装饰模式
 * 装饰模式就是给一个对象增加一些新的功能，而且是动态的，要求装饰对象和被装饰对象实现同一个接口，装饰对象持有被装饰对象的实例
 */
public class DecoratorPattern {
    public static void main(String[] args) {
        Sourceable sourceable = new Source();
        Decorator decorator = new Decorator(sourceable);
        decorator.method();
    }
}
//Source类是被装饰类，Decorator类是一个装饰类，可以为Source类动态的添加一些功能
interface Sourceable {
    void method();
}
class Source implements Sourceable {

    @Override
    public void method() {
        System.out.println("Source");
    }
}
class Decorator implements Sourceable {
    Sourceable source;
    Decorator(Sourceable source){
        this.source = source;
    }

    @Override
    public void method() {
        System.out.println("before");
        source.method();
        System.out.println("after");
    }
}


```

**输出**

![1609918610(1)](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/1609918610(1).png)

## 装饰器模式的应用场景

1、需要扩展一个类的功能。

2、动态的为一个对象增加功能，而且还能动态撤销。（继承不能做到这一点，继承的功能是静态的，不能动态增删。）

缺点：产生过多相似的对象，不易排错！

