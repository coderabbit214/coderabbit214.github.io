# 设计模式-抽象工厂模式


# 设计模式-抽象工厂模式

> 抽象工厂模式（Abstract Factory Pattern）是围绕一个超级工厂创建其他工厂。该超级工厂又称为其他工厂的工厂。这种类型的设计模式属于创建型模式，它提供了一种创建对象的最佳方式。
>
> 在抽象工厂模式中，接口是负责创建一个相关对象的工厂，不需要显式指定它们的类。每个生成的工厂都能按照工厂模式提供对象。

## 图

![3E13CDD1-2CD2-4C66-BD33-DECBF172AE03](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/111.png)

## 代码实现

1. 创建图形接口类

   ```java
   interface Shape{
       void draw();
   }
   ```

2. 创建图形实现类

   ```java
   //创建图形实现类
   class Rectangle implements Shape {
   
       @Override
       public void draw() {
           System.out.println("Rectangle");
       }
   }
   //创建图形实现类
   class Square implements Shape {
   
       @Override
       public void draw() {
           System.out.println("Square");
       }
   }
   ```

3. 创建颜色接口类

   ```java
   //创建颜色接口
   interface Color{
       void fill();
   }
   ```

4. 创建颜色实现类

   ```java
   //创建颜色实现类
   class Red implements Color{
   
       @Override
       public void fill() {
           System.out.println("red");
       }
   }
   //创建颜色实现类
   class Blue implements Color{
   
       @Override
       public void fill() {
           System.out.println("blue");
       }
   }
   ```

5. 创建抽象类获取工厂

   ```java
   //创建抽象类获取工厂
   abstract class AbstractFactory {
       public abstract Color getColor(String color);
       public abstract Shape getShape(String shape) ;
   }
   ```

6. 实现抽象类

   ```java
   //实现抽象类
   class ColorFactory extends AbstractFactory{
       public Color getColor(String colorType){
           if(colorType.equalsIgnoreCase("RED")){
               return new Red();
           } else if(colorType.equalsIgnoreCase("BLUE")){
               return new Blue();
           }
           return null;
       }
   
       @Override
       public Shape getShape(String shape) {
           return null;
       }
   }
   //实现抽象类
   class ShapeFactory extends AbstractFactory{
       @Override
       public Color getColor(String color) {
           return null;
       }
   
       public Shape getShape(String shapeType){
           if(shapeType.equalsIgnoreCase("RECTANGLE")){
               return new Rectangle();
           } else if(shapeType.equalsIgnoreCase("SQUARE")){
               return new Square();
           }
           return null;
       }
   }
   ```

7. 创建工厂创造器/生成器类

   ```java
   //创建工厂创造器/生成器类，通过传递形状或颜色信息来获取工厂。
   class FactoryProducer {
       public static AbstractFactory getFactory(String choice){
           if(choice.equalsIgnoreCase("SHAPE")){
               return new ShapeFactory();
           } else if(choice.equalsIgnoreCase("COLOR")){
               return new ColorFactory();
           }
           return null;
       }
   }
   ```

8. 使用

   ```java
   public class AbstractFactoryPattern {
       public static void main(String[] args) {
           AbstractFactory shape = FactoryProducer.getFactory("SHAPE");
           System.out.println(shape.getColor("RED"));
           Shape rectangle = shape.getShape("RECTANGLE");
           rectangle.draw();
       }
   }
   ```

   运行结果

   ![111](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/3E13CDD1-2CD2-4C66-BD33-DECBF172AE03.jpg)

## 整体代码

```java
package com.designPattern.abstractFactoryPattern;


/**
 * 抽象工厂模式
 */
public class AbstractFactoryPattern {
    public static void main(String[] args) {
        AbstractFactory shape = FactoryProducer.getFactory("SHAPE");
        System.out.println(shape.getColor("RED"));
        Shape rectangle = shape.getShape("RECTANGLE");
        rectangle.draw();
    }
}
//创建工厂创造器/生成器类，通过传递形状或颜色信息来获取工厂。
class FactoryProducer {
    public static AbstractFactory getFactory(String choice){
        if(choice.equalsIgnoreCase("SHAPE")){
            return new ShapeFactory();
        } else if(choice.equalsIgnoreCase("COLOR")){
            return new ColorFactory();
        }
        return null;
    }
}
//————————————————————————————————————————————————————————————————————————————————————————————————————————————————//
//创建抽象类获取工厂
abstract class AbstractFactory {
    public abstract Color getColor(String color);
    public abstract Shape getShape(String shape) ;
}
//实现抽象类
class ColorFactory extends AbstractFactory{
    public Color getColor(String colorType){
        if(colorType.equalsIgnoreCase("RED")){
            return new Red();
        } else if(colorType.equalsIgnoreCase("BLUE")){
            return new Blue();
        }
        return null;
    }

    @Override
    public Shape getShape(String shape) {
        return null;
    }
}
//实现抽象类
class ShapeFactory extends AbstractFactory{
    @Override
    public Color getColor(String color) {
        return null;
    }

    public Shape getShape(String shapeType){
        if(shapeType.equalsIgnoreCase("RECTANGLE")){
            return new Rectangle();
        } else if(shapeType.equalsIgnoreCase("SQUARE")){
            return new Square();
        }
        return null;
    }
}
//————————————————————————————————————————————————————————————————————————————————————————————————————————————————//
//创建颜色接口
interface Color{
    void fill();
}
//创建颜色实现类
class Red implements Color{

    @Override
    public void fill() {
        System.out.println("red");
    }
}
//创建颜色实现类
class Blue implements Color{

    @Override
    public void fill() {
        System.out.println("blue");
    }
}
//————————————————————————————————————————————————————————————————————————————————————————————————————————————————//
//创建图形接口类
interface Shape{
    void draw();
}
//创建图形实现类
class Rectangle implements Shape {

    @Override
    public void draw() {
        System.out.println("Rectangle");
    }
}
//创建图形实现类
class Square implements Shape {

    @Override
    public void draw() {
        System.out.println("Square");
    }
}

```

<hr>

> 参考[菜鸟教程](https://www.runoob.com/design-pattern/factory-pattern.html)




