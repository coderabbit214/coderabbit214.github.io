# 设计模式-原型模式


# 设计模式-原型模式

> 用一个已经创建的实例作为原型，通过复制该原型对象来创建一个和原型相同或相似的新对象。

## 原型模式的结构与实现

由于 Java 提供了对象的 clone() 方法，所以用 Java 实现原型模式很简单。

#### 1. 模式的结构

原型模式包含以下主要角色。

1. 抽象原型类：规定了具体原型对象必须实现的接口。

2. 具体原型类：实现抽象原型类的 clone() 方法，它是可被复制的对象。

3. 访问类：使用具体原型类中的 clone() 方法来复制新的对象。

   ![a22](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/a22.png)

#### 2. 模式的实现

原型模式的克隆分为浅克隆和深克隆。

- 浅克隆：创建一个新对象，新对象的属性和原来对象完全相同，对于非基本类型属性，仍指向原有属性所指向的对象的内存地址。
- 深克隆：创建一个新对象，属性中引用的其他对象也会被克隆，不再指向原有对象地址。

```java
package com.designPattern.prototypePattern;

/**
 * 原型模式
 */
public class PrototypePattern {
    public static void main(String[] args) throws CloneNotSupportedException {
        Realizetype obj1 = new Realizetype();
        Realizetype obj2 = (Realizetype) obj1.clone();
        System.out.println("obj1==obj2?" + (obj1 == obj2));
    }

}
//具体原型类
class Realizetype implements Cloneable {
    Realizetype() {
        System.out.println("具体原型创建成功！");
    }
    public Object clone() throws CloneNotSupportedException {
        System.out.println("具体原型复制成功！");
        return (Realizetype) super.clone();
    }
}

```

   

