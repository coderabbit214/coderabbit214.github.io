# 设计模式-建造者模式


# 设计模式-建造者模式

> 定义：指将一个复杂对象的构造与它的表示分离，使同样的构建过程可以创建不同的表示，这样的设计模式被称为建造者模式。它是将一个复杂的对象分解为多个简单的对象，然后一步一步构建而成。它将变与不变相分离，即产品的组成部分是不变的，但每一部分是可以灵活选择的。
>
> 个人理解:类似为一辆车挑选不同的部件
>
> 应用：
>
> 1、需要生成的对象具有复杂的内部结构。
> 2、需要生成的对象内部属性本身相互依赖。
>
> 优点如下：
>
> 1. 封装性好，构建和表示分离。
> 2. 扩展性好，各个具体的建造者相互独立，有利于系统的解耦。
> 3. 客户端不必知道产品内部组成的细节，建造者可以对创建过程逐步细化，而不对其它模块产生任何影响，便于控制细节风险。
>
>
> 其缺点如下：
>
> 1. 产品的组成部分必须相同，这限制了其使用范围。
> 2. 如果产品的内部变化复杂，如果产品内部发生变化，则建造者也要同步修改，后期维护成本较大。
>
>
> 建造者（Builder）模式和工厂模式的关注点不同：建造者模式注重零部件的组装过程，而工厂方法模式更注重零部件的创建过程，但两者可以结合使用。

## 1. 模式的结构

建造者（Builder）模式的主要角色如下。

1. 产品角色（Product）：它是包含多个组成部件的复杂对象，由具体建造者来创建其各个零部件。
2. 抽象建造者（Builder）：它是一个包含创建产品各个子部件的抽象方法的接口，通常还包含一个返回复杂产品的方法 getResult()。
3. 具体建造者(Concrete Builder）：实现 Builder 接口，完成复杂产品的各个部件的具体创建方法。
4. 指挥者（Director）：它调用建造者对象中的部件构造与装配方法完成复杂对象的创建，在指挥者中不涉及具体产品的信息。

## 2. 图

![41X4](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/41X4.png)

## 3. 模式的实现

1. 产品角色：包含多个组成部件的复杂对象。

```java
class Product {
    private String partA;
    private String partB;
    private String partC;

    public void setPartA(String partA) {
        this.partA = partA;
    }
    public void setPartB(String partB) {
        this.partB = partB;
    }
    public void setPartC(String partC) {
        this.partC = partC;
    }

    @Override
    public String toString() {
        return "Product{" +
                "partA='" + partA + '\'' +
                ", partB='" + partB + '\'' +
                ", partC='" + partC + '\'' +
                '}';
    }
}
```

2. 抽象建造者：包含创建产品各个子部件的抽象方法。

```java
//抽象建造者:包含创建产品各个子部件的抽象方法。
abstract class Builder{
    Product product = new Product();
    public abstract void buildPartA();
    public abstract void buildPartB();
    public abstract void buildPartC();
    //返回产品对象
    public Product getResult() {
        return product;
    }
}
```

3. 具体建造者：实现了抽象建造者接口。

```java
//具体建造者：实现了抽象建造者接口。
class ConcreteBuilder extends Builder {
    public void buildPartA() {
        product.setPartA("建造 PartA");
    }
    public void buildPartB() {
        product.setPartB("建造 PartB");
    }
    public void buildPartC() {
        product.setPartC("建造 PartC");
    }
}
```

4. 指挥者：调用建造者中的方法完成复杂对象的创建。

```java
//指挥者：调用建造者中的方法完成复杂对象的创建。
class Director {
    private Builder builder;
    public Director(Builder builder) {
        this.builder = builder;
    }
    //产品构建与组装方法
    public Product construct() {
        builder.buildPartA();
        builder.buildPartB();
        builder.buildPartC();
        return builder.getResult();
    }
}
```

5. 客户类。

```java
public class BuilderPattern {
    public static void main(String[] args) {
        Builder builder = new ConcreteBuilder();
        Director director = new Director(builder);
        Product product = director.construct();
        System.out.println(product);
    }
}
```

## 4.模拟客厅装修

```java
/**
 *用建造者（Builder）模式描述客厅装修。
 *
 * 分析：客厅装修是一个复杂的过程，它包含墙体的装修、电视机的选择、沙发的购买与布局等。客户把装修要求告诉项目经理，项目经理指挥装修工人一步步装修，
 * 最后完成整个客厅的装修与布局，所以本实例用建造者模式实现比较适合。
 *
 * 这里客厅是产品，包括墙、电视和沙发等组成部分。具体装修工人是具体建造者，他们负责装修与墙、电视和沙发的布局。项目经理是指挥者，他负责指挥装修工人进行装修。
 */
public class ParlourDecorator {
    public static void main(String[] args) {
        Decorator decorator = new ConcreteDecorator2();
        ProjectManager projectManager = new ProjectManager(decorator);
        Parlour decorate = projectManager.decorate();
        System.out.println(decorate);
    }
}

//产品：客厅
class Parlour {
    private String wall;    //墙
    private String TV;    //电视
    private String sofa;    //沙发 

    public void setWall(String wall) {
        this.wall = wall;
    }

    public void setTV(String TV) {
        this.TV = TV;
    }

    public void setSofa(String sofa) {
        this.sofa = sofa;
    }

    @Override
    public String toString() {
        return "Parlour{" +
                "wall='" + wall + '\'' +
                ", TV='" + TV + '\'' +
                ", sofa='" + sofa + '\'' +
                '}';
    }
}

//抽象建造者：装修工人
abstract class Decorator {
    //创建产品对象
    protected Parlour product = new Parlour();

    public void buildWall(){
        product.setWall("w1");
    };

    public abstract void buildTV();

    public abstract void buildSofa();

    //返回产品对象
    public Parlour getResult() {
        return product;
    }
}

//具体建造者：具体装修工人1
class ConcreteDecorator1 extends Decorator {

    public void buildTV() {
        product.setTV("TV1");
    }

    public void buildSofa() {
        product.setSofa("sf1");
    }
}

//具体建造者：具体装修工人2
class ConcreteDecorator2 extends Decorator {

    public void buildTV() {
        product.setTV("TV2");
    }

    public void buildSofa() {
        product.setSofa("sf2");
    }
}

//指挥者：项目经理
class ProjectManager {
    private Decorator builder;

    public ProjectManager(Decorator builder) {
        this.builder = builder;
    }

    //产品构建与组装方法
    public Parlour decorate() {
        builder.buildWall();
        builder.buildTV();
        builder.buildSofa();
        return builder.getResult();
    }
}

```


