# 设计模式-桥接模式


# 设计模式-桥接模式

> 处理多层继承结构，处理多维度变化的场景，将各个维度设计成独立的继承结构，使各个维度可以独立的扩展在抽象层建立关联。

## 问题

![20180613211930507](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/20180613213104841.png)

## 解决

![20180613213104841](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/20180613211930507.png)

## 代码实现

1. 创建电脑品牌

   ```java
   //  创建电脑牌子
   class Lenovo implements Brand {
   
       @Override
       public void sale() {
           System.out.println("出售联想");
       }
   
   }
   class Dell implements Brand {
   
       @Override
       public void sale() {
           System.out.println("出售戴尔");
       }
   
   }
   
   // 新增牌子是十分方便的
   class Shenzhou implements Brand {
   
       @Override
       public void sale() {
           System.out.println("出售神州");
       }
   
   }
   ```

2. 创建电脑以及类型

   ```java
   class Computer {
   
       protected Brand brand;
   
       public Computer(Brand b) {
           this.brand = b;
       }
       public void sale(){
           brand.sale();
       }
   }
   //  创建电脑类型
   class Desktop extends Computer {
   
       public Desktop(Brand b) {
           super(b);
       }
   
       @Override
       public void sale() {
           super.sale();
           System.out.println("出售台式电脑");
       }
   }
   
   class Laptop extends Computer {
   
       public Laptop(Brand b) {
           super(b);
       }
   
       @Override
       public void sale() {
           super.sale();
           System.out.println("出售笔记本");
       }
   }
   
   ```

3. 使用

   ```java
   public class BridgePattern {
       public static void main(String[] args) {
           Computer computer = new Desktop(new Dell());
           computer.sale();
       }
   }
   ```

## 完整代码

```java
package com.designPattern.bridgePattern;

/**
 * 桥接模式
 * 处理多层继承结构，处理多维度变化的场景，将各个维度设计成独立的继承结构，使各个维度可以独立的扩展在抽象层建立关联。
 */
public class BridgePattern {
    public static void main(String[] args) {
        Computer computer = new Desktop(new Dell());
        computer.sale();
    }
}
//创建电脑接口
interface Brand {
    void sale();
}

//  创建电脑牌子
class Lenovo implements Brand {

    @Override
    public void sale() {
        System.out.println("出售联想");
    }

}
class Dell implements Brand {

    @Override
    public void sale() {
        System.out.println("出售戴尔");
    }

}

// 新增牌子是十分方便的
class Shenzhou implements Brand {

    @Override
    public void sale() {
        System.out.println("出售神州");
    }

}
class Computer {

    protected Brand brand;

    public Computer(Brand b) {
        this.brand = b;
    }
    public void sale(){
        brand.sale();
    }
}
//  创建电脑类型
class Desktop extends Computer {

    public Desktop(Brand b) {
        super(b);
    }

    @Override
    public void sale() {
        super.sale();
        System.out.println("出售台式电脑");
    }
}

class Laptop extends Computer {

    public Laptop(Brand b) {
        super(b);
    }

    @Override
    public void sale() {
        super.sale();
        System.out.println("出售笔记本");
    }
}

```


