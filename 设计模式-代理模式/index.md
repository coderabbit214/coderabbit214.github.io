# 设计模式-代理模式


# 设计模式-代理模式

> 代理模式就是多一个代理类出来，替原对象进行一些操作

![1354197582_1664](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/1609929048(1).png)

## 代码实现

```java
package com.designPattern.proxyPattern;

/**
 * 代理模式
 * 代理模式就是多一个代理类出来，替原对象进行一些操作
 * 代理模式的应用场景：
 * 如果已有的方法在使用的时候需要对原有的方法进行改进，此时有两种办法：
 * 1、修改原有的方法来适应。这样违反了“对扩展开放，对修改关闭”的原则。
 *
 * 2、就是采用一个代理类调用原有的方法，且对产生的结果进行控制。这种方法就是代理模式。
 *
 * 使用代理模式，可以将功能划分的更加清晰，有助于后期维护！
 */
public class ProxyPattern {
    public static void main(String[] args) {
        Sourceable sourceable = new Proxy();
        sourceable.method();
    }
}

interface Sourceable {
    void method();
}


class Source implements Sourceable {

    @Override
    public void method() {
        System.out.println("the original method!");
    }
}
//代理类
class Proxy implements Sourceable {

    private Source source;

    public Proxy() {
        this.source = new Source();
    }

    @Override
    public void method() {
        before();
        source.method();
        atfer();
    }

    private void atfer() {
        System.out.println("after proxy!");
    }

    private void before() {
        System.out.println("before proxy!");
    }
}

```

**输出**



![1609929048(1)](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/1354197582_1664.PNG)

## 代理模式的应用场景

如果已有的方法在使用的时候需要对原有的方法进行改进，此时有两种办法：

1、修改原有的方法来适应。这样违反了“对扩展开放，对修改关闭”的原则。

2、就是采用一个代理类调用原有的方法，且对产生的结果进行控制。这种方法就是代理模式。

使用代理模式，可以将功能划分的更加清晰，有助于后期维护！

