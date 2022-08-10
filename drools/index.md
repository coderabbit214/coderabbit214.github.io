# Drools


# Drools

> 规则引擎
>
> **规则引擎**，全称为**业务规则管理系统**，英文名为BRMS(即Business Rule Management System)。规则引擎的主要思想是将应用程序中的业务决策部分分离出来，并使用预定义的语义模块编写业务决策（业务规则），由用户或开发者在需要时进行配置、管理。

## 优点

1、业务规则与系统代码分离，实现业务规则的集中管理

2、在不重启服务的情况下可随时对业务规则进行扩展和维护

3、可以动态修改业务规则，从而快速响应需求变更

4、规则引擎是相对独立的，只关心业务规则，使得业务分析人员也可以参与编辑、维护系统的业务规则

5、减少了硬编码业务规则的成本和风险

6、使用规则引擎提供的规则编辑工具，使复杂的业务规则实现变得的简单

## 应用场景

对于一些存在比较复杂的业务规则并且业务规则会频繁变动的系统比较适合使用规则引擎



## 初体验

### spring方式

1. 新建maven项目

2. pom

   ```xml
           <dependency>
               <groupId>org.drools</groupId>
               <artifactId>drools-compiler</artifactId>
               <version>7.10.0.Final</version>
           </dependency>
           <!--  测试   -->
           <dependency>
               <groupId>junit</groupId>
               <artifactId>junit</artifactId>
               <version>4.12</version>
           </dependency>
   ```

3. 根据drools要求创建resources/META-INF/kmodule.xml配置文件

   ```xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <kmodule xmlns="http://www.drools.org/xsd/kmodule">
       <!--
           name:指定kbase的名称，可以任意，但是需要唯一
           packages:指定规则文件的目录，需要根据实际情况填写，否则无法加载到规则文件
           default:指定当前kbase是否为默认
       -->
       <kbase name="myKbase1" packages="rules" default="true">
           <!--
               name:指定ksession名称，可以任意，但是需要唯一
               default:指定当前session是否为默认
           -->
           <ksession name="ksession-rule" default="true"/>
       </kbase>
   </kmodule>
   ```

4. 创建实体类

   ```java
   
   /**
    * 订单
    */
   public class Order {
       private Double originalPrice;//订单原始价格，即优惠前价格
       private Double realPrice;//订单真实价格，即优惠后价格
   
       public String toString() {
           return "Order{" +
                   "originalPrice=" + originalPrice +
                   ", realPrice=" + realPrice +
                   '}';
       }
   
       public Double getOriginalPrice() {
           return originalPrice;
       }
   
       public void setOriginalPrice(Double originalPrice) {
           this.originalPrice = originalPrice;
       }
   
       public Double getRealPrice() {
           return realPrice;
       }
   
       public void setRealPrice(Double realPrice) {
           this.realPrice = realPrice;
       }
   }
   ```

5. 创建规则文件resources/rules/bookDiscount.drl

   ```java
   //图书优惠规则
   package book.discount
   import com.itheima.drools.entity.Order
   
   //规则一：所购图书总价在100元以下的没有优惠
   rule "book_discount_1"
       when
           $order:Order(originalPrice < 100)
       then
           $order.setRealPrice($order.getOriginalPrice());
           System.out.println("成功匹配到规则一：所购图书总价在100元以下的没有优惠");
   end
   
   //规则二：所购图书总价在100到200元的优惠20元
   rule "book_discount_2"
       when
           $order:Order(originalPrice < 200 && originalPrice >= 100)
       then
           $order.setRealPrice($order.getOriginalPrice() - 20);
           System.out.println("成功匹配到规则二：所购图书总价在100到200元的优惠20元");
   end
   
   //规则三：所购图书总价在200到300元的优惠50元
   rule "book_discount_3"
       when
           $order:Order(originalPrice <= 300 && originalPrice >= 200)
       then
           $order.setRealPrice($order.getOriginalPrice() - 50);
           System.out.println("成功匹配到规则三：所购图书总价在200到300元的优惠50元");
   end
   
   //规则四：所购图书总价在300元以上的优惠100元
   rule "book_discount_4"
       when
           $order:Order(originalPrice >= 300)
       then
           $order.setRealPrice($order.getOriginalPrice() - 100);
           System.out.println("成功匹配到规则四：所购图书总价在300元以上的优惠100元");
   end
   ```

6. 测试

   ```java
   @Test
   public void test1(){
       KieServices kieServices = KieServices.Factory.get();
       KieContainer kieClasspathContainer = kieServices.getKieClasspathContainer();
       //会话对象，用于和规则引擎交互
       KieSession kieSession = kieClasspathContainer.newKieSession();
   
       //构造订单对象，设置原始价格，由规则引擎根据优惠规则计算优惠后的价格
       Order order = new Order();
       order.setOriginalPrice(210D);
   
       //将数据提供给规则引擎，规则引擎会根据提供的数据进行规则匹配
       kieSession.insert(order);
   
       //激活规则引擎，如果规则匹配成功则执行规则
       kieSession.fireAllRules();
       //关闭会话
       kieSession.dispose();
   
       System.out.println("优惠前原始价格：" + order.getOriginalPrice() +
                          "，优惠后价格：" + order.getRealPrice());
   }
    //使用字符串
       @Test
       public void test2(){
           String myRule = "//图书优惠规则\n" +
                   "package book.discount\n" +
                   "import com.jsh.droolsdemo.entity.Order\n" +
                   "\n" +
                   "//规则一：所购图书总价在100元以下的没有优惠\n" +
                   "rule \"book_discount_1\"\n" +
                   "    when\n" +
                   "        $order:Order(originalPrice < 100)\n" +
                   "    then\n" +
                   "        $order.setRealPrice($order.getOriginalPrice());\n" +
                   "        System.out.println(\"成功匹配到规则一：所购图书总价在100元以下的没有优惠\");\n" +
                   "end\n" +
                   "\n" +
                   "//规则二：所购图书总价在100到200元的优惠20元\n" +
                   "rule \"book_discount_2\"\n" +
                   "    when\n" +
                   "        $order:Order(originalPrice < 200 && originalPrice >= 100)\n" +
                   "    then\n" +
                   "        $order.setRealPrice($order.getOriginalPrice() - 20);\n" +
                   "        System.out.println(\"成功匹配到规则二：所购图书总价在100到200元的优惠20元\");\n" +
                   "end\n" +
                   "\n" +
                   "//规则三：所购图书总价在200到300元的优惠50元\n" +
                   "rule \"book_discount_3\"\n" +
                   "    when\n" +
                   "        $order:Order(originalPrice <= 300 && originalPrice >= 200)\n" +
                   "    then\n" +
                   "        $order.setRealPrice($order.getOriginalPrice() - 50);\n" +
                   "        System.out.println(\"成功匹配到规则三：所购图书总价在200到300元的优惠50元\");\n" +
                   "end\n" +
                   "\n" +
                   "//规则四：所购图书总价在300元以上的优惠100元\n" +
                   "rule \"book_discount_4\"\n" +
                   "    when\n" +
                   "        $order:Order(originalPrice >= 300)\n" +
                   "    then\n" +
                   "        $order.setRealPrice($order.getOriginalPrice() - 100);\n" +
                   "        System.out.println(\"成功匹配到规则四：所购图书总价在300元以上的优惠100元\");\n" +
                   "end";
           //构造订单对象，设置原始价格，由规则引擎根据优惠规则计算优惠后的价格
           Order order = new Order();
           order.setOriginalPrice(210D);
   
           KieHelper kieHelper = new KieHelper();
           kieHelper.addContent(myRule, ResourceType.DRL);
           KieSession kieSession = kieHelper.build().newKieSession();
   
           kieSession.insert(order);
           //激活规则引擎，如果规则匹配成功则执行规则
           kieSession.fireAllRules();
           //关闭会话
           kieSession.dispose();
   
           System.out.println("优惠前原始价格：" + order.getOriginalPrice() +
                   "，优惠后价格：" + order.getRealPrice());
       }
   ```

### springboot方式

1. 新建springboot项目

2. pom

   ```xml
   <properties>
       <java.version>1.8</java.version>
       <drools.version>7.14.0.Final</drools.version>
   </properties>
   <dependencies>
   <!-- drools依赖 -->
       <dependency>
           <groupId>org.drools</groupId>
           <artifactId>drools-core</artifactId>
           <version>${drools.version}</version>
       </dependency>
       <dependency>
           <groupId>org.drools</groupId>
           <artifactId>drools-compiler</artifactId>
           <version>${drools.version}</version>
       </dependency>
       <!-- 决策表 -->
       <dependency>
           <groupId>org.drools</groupId>
           <artifactId>drools-decisiontables</artifactId>
           <version>${drools.version}</version>
       </dependency>
       <!-- 模板 -->
       <dependency>
           <groupId>org.drools</groupId>
           <artifactId>drools-templates</artifactId>
           <version>${drools.version}</version>
       </dependency>
       
       <dependency>
           <groupId>org.kie</groupId>
           <artifactId>kie-api</artifactId>
           <version>${drools.version}</version>
       </dependency>
   </dependencies>
   ```

3. config

   ```java
   @Configuration
   public class KiaSessionConfig {
   
       private static final String RULES_PATH = "rules/";
   
       @Bean
       public KieFileSystem kieFileSystem() throws IOException {
           KieFileSystem kieFileSystem = getKieServices().newKieFileSystem();
           for (Resource file : getRuleFiles()) {
               kieFileSystem.write(ResourceFactory.newClassPathResource(RULES_PATH + file.getFilename(), "UTF-8"));
           }
           return kieFileSystem;
       }
   
       private Resource[] getRuleFiles() throws IOException {
   
           ResourcePatternResolver resourcePatternResolver = new PathMatchingResourcePatternResolver();
           final Resource[] resources = resourcePatternResolver.getResources("classpath*:" + RULES_PATH + "**/*.*");
           return resources;
   
       }
   
       @Bean
       public KieContainer kieContainer() throws IOException {
   
           final KieRepository kieRepository = getKieServices().getRepository();
           kieRepository.addKieModule(new KieModule() {
               @Override
               public ReleaseId getReleaseId() {
                   return kieRepository.getDefaultReleaseId();
               }
           });
   
           KieBuilder kieBuilder = getKieServices().newKieBuilder(kieFileSystem());
           kieBuilder.buildAll();
           return getKieServices().newKieContainer(kieRepository.getDefaultReleaseId());
   
       }
   
       private KieServices getKieServices() {
           return KieServices.Factory.get();
       }
   
       @Bean
       public KieBase kieBase() throws IOException {
           return kieContainer().getKieBase();
       }
   
       @Bean
       public KieSession kieSession() throws IOException {
           return kieContainer().newKieSession();
       }
   }
   
   ```

4. 创建实体类

   ```java
   /**
    * 订单
    */
   public class Order {
       private Double originalPrice;//订单原始价格，即优惠前价格
       private Double realPrice;//订单真实价格，即优惠后价格
   
       public String toString() {
           return "Order{" +
                   "originalPrice=" + originalPrice +
                   ", realPrice=" + realPrice +
                   '}';
       }
   
       public Double getOriginalPrice() {
           return originalPrice;
       }
   
       public void setOriginalPrice(Double originalPrice) {
           this.originalPrice = originalPrice;
       }
   
       public Double getRealPrice() {
           return realPrice;
       }
   
       public void setRealPrice(Double realPrice) {
           this.realPrice = realPrice;
       }
   }
   ```

5. 创建规则文件resources/rules/bookDiscount.drl

   ```java
   //图书优惠规则
   package book.discount
   import com.itheima.drools.entity.Order
   
   //规则一：所购图书总价在100元以下的没有优惠
   rule "book_discount_1"
       when
           $order:Order(originalPrice < 100)
       then
           $order.setRealPrice($order.getOriginalPrice());
           System.out.println("成功匹配到规则一：所购图书总价在100元以下的没有优惠");
   end
   
   //规则二：所购图书总价在100到200元的优惠20元
   rule "book_discount_2"
       when
           $order:Order(originalPrice < 200 && originalPrice >= 100)
       then
           $order.setRealPrice($order.getOriginalPrice() - 20);
           System.out.println("成功匹配到规则二：所购图书总价在100到200元的优惠20元");
   end
   
   //规则三：所购图书总价在200到300元的优惠50元
   rule "book_discount_3"
       when
           $order:Order(originalPrice <= 300 && originalPrice >= 200)
       then
           $order.setRealPrice($order.getOriginalPrice() - 50);
           System.out.println("成功匹配到规则三：所购图书总价在200到300元的优惠50元");
   end
   
   //规则四：所购图书总价在300元以上的优惠100元
   rule "book_discount_4"
       when
           $order:Order(originalPrice >= 300)
       then
           $order.setRealPrice($order.getOriginalPrice() - 100);
           System.out.println("成功匹配到规则四：所购图书总价在300元以上的优惠100元");
   end
   ```

6. 测试

   ```java
   @SpringBootTest
   class DroolsDemoApplicationTests2 {
       @Autowired
       private KieSession session;
   
       @Autowired
       private KieBase kieBase;
   
       @Test
       public void test1(){
           //构造订单对象，设置原始价格，由规则引擎根据优惠规则计算优惠后的价格
           Order order = new Order();
           order.setOriginalPrice(210D);
           //将数据提供给规则引擎，规则引擎会根据提供的数据进行规则匹配
           session.insert(order);
           //激活规则引擎，如果规则匹配成功则执行规则
           session.fireAllRules();
           //关闭会话
           session.dispose();
           System.out.println("优惠前原始价格：" + order.getOriginalPrice() +
                   "，优惠后价格：" + order.getRealPrice());
       }
   }
   ```

## 匹配规则

![WeChat0b24cae95a8cca9e1be7a269a3a0b750](https://images-jsh.oss-cn-beijing.aliyuncs.com/work/WeChat0b24cae95a8cca9e1be7a269a3a0b750.png)

规则格式（举例）

```java
rule "book_discount_1"
    when
        $order:Order(originalPrice < 100)
    then
        $order.setRealPrice($order.getOriginalPrice());
        System.out.println("成功匹配到规则一：所购图书总价在100元以下的没有优惠");
end
```

- When(匹配规则)

  - 没有约束（只匹配类型）

    ```
    when
    	Order()
    ```

  - 条件约束

    ```
    when
    	Order(originalPrice < 100)
    ```

  - 匹配规则并绑定数据

    ```
    when
      $order:Order(originalPrice < 100)
    ```

  - 可以多个类

    ```java
        @Test
        public void test2(){
            //构造订单对象，设置原始价格，由规则引擎根据优惠规则计算优惠后的价格
            Cat cat = new Cat();
            cat.setSex(0);
            Order order = new Order();
            order.setOriginalPrice(99D);
            //将数据提供给规则引擎，规则引擎会根据提供的数据进行规则匹配
            session.insert(order);
            session.insert(cat);
    
            //激活规则引擎，如果规则匹配成功则执行规则
            session.fireAllRules();
            //关闭会话
            session.dispose();
        }
    ```

    ```
    when
      $order:Order(originalPrice < 100) and $cat:Cat(sex == 0)
    ```

### 约束

| 约束                   | 描述                                                         |
| ---------------------- | ------------------------------------------------------------ |
| !.                     | 使用此运算符可以以空安全的方式取消引用属性。!.运算符左侧的值不能为null（解释为!= null） |
| []                     | 按List索引访问值或Map按键访问值                              |
| <，<=，>，>=           | 在具有自然顺序的属性上使用这些运算符                         |
| ==, !=                 | 在约束中使用这些运算符作为equals()和!equals()方法            |
| &&，\|\|               | 组合关系条件                                                 |
| matches，not matches   | 使用这些运算符可以指示字段与指定的Java正则表达式匹配或不匹配 |
| contains，not contains | 使用这些运算符可以验证Array或字段是否包含或不包含指定值      |
| memberOf，not memberOf | 使用这些运算符可以验证字段是否为定义为变量Array的成员        |
| soundslike             | 使用英语发音来验证单词是否具有与给定值几乎相同的声音（类似于该matches运算符） |
| in，notin              | 使用这些运算符可以指定一个以上的可能值来匹配约束（复合值限制） |

- from 取集合中的元素

  例：

  实体类

  ```java
  @Data
  public class Animal {
      private List<Cat> cats;
  }
  
  
  @Data
  public class Cat {
      private int sex;
      private String name;
  }
  ```

  drl

  ```java
  package com.jsh.animal
  import com.jsh.droolsdemo.entity.Animal
  import com.jsh.droolsdemo.entity.Cat
  
  rule "from"
  when
      $an : Animal()
      $p : Cat(sex != 3) from $an.cats
      then
          System.out.println($p);
      end
  ```

  测试

  ```java
      //测试from
      @Test
      public void test3(){
          //构造订单对象，设置原始价格，由规则引擎根据优惠规则计算优惠后的价格
          List<Cat> cats = new ArrayList<>();
          Cat cat = new Cat();
          cat.setSex(0);
          cat.setName("11");
          Cat cat1 = new Cat();
          cat1.setSex(0);
          cat1.setName("22");
          Cat cat2 = new Cat();
          cat2.setSex(0);
          cat2.setName("33");
          Cat cat3 = new Cat();
          cat3.setSex(0);
          cat3.setName("44");
          cats.add(cat);
          cats.add(cat1);
          cats.add(cat2);
          cats.add(cat3);
          Animal animal = new Animal();
          animal.setCats(cats);
          session.insert(animal);
  
          //激活规则引擎，如果规则匹配成功则执行规则
          session.fireAllRules();
          //关闭会话
          session.dispose();
      }
  ```

  输出

  ![截屏2021-10-21 下午2.45.01](https://images-jsh.oss-cn-beijing.aliyuncs.com/work/%E6%88%AA%E5%B1%8F2021-10-21%20%E4%B8%8B%E5%8D%882.45.01.png)

- collect 获取集合 



## RHS动作

### 主要操作

| 动作          | 描述                                                     |
| ------------- | -------------------------------------------------------- |
| set           | 给属性赋值                                               |
| modify        | 将改变通知drolls引擎                                     |
| update        | 将改变通知drolls引擎                                     |
| insert        | 将新实事插入到drools引擎的工作                           |
| insertLogical | insert增强版，需声明撤回事件，或待不在匹配条件后自动撤回 |
| delete        | 删除实事                                                 |

#### update

> Update用于将数据的更改更新到引擎，并通知引擎重新匹配该事实

```java
rule "update"
when
    $an : Cat(sex == 0)
    then
        System.out.println($an);
        $an.setSex(1);
        update($an)
    end

rule "update2"
when
    $an : Cat(sex == 1)
    then
        System.out.println($an);
    end
```

- 测试

```java
  //  测试update
      @Test
      public void test4(){
          Cat cat = new Cat();
          cat.setSex(0);
          session.insert(cat);
  
          //激活规则引擎，如果规则匹配成功则执行规则
          session.fireAllRules();
          //关闭会话
          session.dispose();
      }
```

- 输出

  ![截屏2021-10-21 下午3.05.11](https://images-jsh.oss-cn-beijing.aliyuncs.com/work/%E6%88%AA%E5%B1%8F2021-10-21%20%E4%B8%8B%E5%8D%883.05.11.png)

#### modify

> 和update功能一样

```java
rule "update"
when
    $an : Cat(sex == 0)
    then
        System.out.println($an);
//        $an.setSex(1);
//        update($an)
        modify($an){
            setSex(1)
        }
    end
```



## 核心算法Rete

> 拉丁语单词"rete"的意思是"网络"或"网络"。Rete 算法可分为两个部分：规则编译和运行时间执行。

[原文链接](https://docs.jboss.org/drools/release/5.3.0.Final/drools-expert-docs/html/ch03.html)

![Rete_Nodes](https://images-jsh.oss-cn-beijing.aliyuncs.com/work/Rete_Nodes.png)

### ReteNode

是所有对象进入网络的入口，然后进入到TypeNode

### ObjectTypeNode

是我们规则所用到的pojo，ObjectTypeNode就是类型检查，引擎只让匹配Object 类型的对象到达节点

![Object_Type_Nodes](https://images-jsh.oss-cn-beijing.aliyuncs.com/work/Object_Type_Nodes.png)

### AlphaNode

Drools 通过散列法优化了从 ObjectTypeNode 到 AlphaNode 的传播。每次一个 AlphaNode 被加到一个 ObjectTypeNode 的时候，就以字面值（ literal value ）作为 key ，以 AlphaNode 作为 value 加入 HashMap 。当一个新的实例进入 ObjectTypeNode 的时候，不用传递到每一个 AlphaNode ，它可以直接从 HashMap 中获得正确的 AlphaNode ，避免了不必要的字面检查。

例如：Cheese （name＝”cheddar” ,strengh==”strong”)

![Alpha_Nodes](https://images-jsh.oss-cn-beijing.aliyuncs.com/work/Alpha_Nodes.png)

### JoinNode

用来对2个对象进行对比、检查。约定JoinNode的2个输入称为左边和右边。左边通常是一个运算后的结果（LeftInputNodeAdapter），右边通常是一个ObjectTypeNode。

![Join_Node](https://images-jsh.oss-cn-beijing.aliyuncs.com/work/Join_Node.png)

举例：

```java
rule
    when
        Cheese( $chedddar : name == "cheddar" )
        $person : Person( favouriteCheese == $cheddar )
    then
        System.out.println( $person.getName() + " likes cheddar" );
    end
    
   
    
    rule
    when
        Cheese( $chedddar : name == "cheddar" )
        $person : Person( favouriteCheese != $cheddar )
    then
        System.out.println( $person.getName() + " does not like cheddar" );
    end
```

这里就共用了一个`Cheese( $chedddar : name == "cheddar" )`的结果

![Node_Sharing](https://images-jsh.oss-cn-beijing.aliyuncs.com/work/Node_Sharing.png)



## 问题

1. 同一规则有多个或的判断条件，如何分组、指定优先级呢?

- - 分组(如果传入的是列表，只会触发列表中的一个数据)

  - - activation-group(同一个组内只有一个会执行)
    - agenda-group(同一个组内，要不都执行，要不都不执行)

  - 优先级

  - - salience：数值越大越先执行

2. 规则的数量会影响Drools的执行效率?

- - 不会

- - Rete算法是一种前向规则快速匹配算法，其匹配速度与规则数目无关
  - 规则会在项目启动时创建rete网络
  - 在运行时只会运行匹配的规则
  - 用空间换时间

3. Drools的function、global属性的作用？能运用到什么场景？

- - global：定义全局变量

  - - 用于多个规则都需要使用的变量
    - 规则内部使用的类无法得到的变量

  - function：定义方法（和java一样）

  - - 使用静态类对外赋值

4. 如何指定特定规则运行？

- - 在fireAllRules中传入过滤器（AgendaFilter的子类）

5. 如何动态管理规则？因为在实际业务中，规则条件极有可能发生变化。【思考题】



5. 当存在大量的规则编写时，如何提高规则编写的效率。【开发问题】

- - 通过规则分组，或者通过指标编号分组

7. Drools为什么快？核心算法及其原理是什么？【进阶】

- - Rete(拉丁语,英语:net)算法

  - 原理：

  - - 通过规则条件生成了一个网络，每个规则条件是网络中的一个节点
    - rete可以被分为两部分：规则编译和运行时执行。规则编译是指根据规则集生成推理网络的过程，运行时执行指将数据送入推理网络进行筛选的过程

## 使用心得

- 如何调用外界的方法（类似放入需要规则判断的数据，在drools文件中可以调用方法）

  1. 在kieSession中放入类（java）

     ```java
     kieSession.insert();
     ```

  2. 声明类，调用方法（drools文件）

     ```java
     //声明
     $fs: financialServiceImpl()
     //使用（对别的类的参数进行判断）
     $fs.getcompAveBvSecuritvCode(参数...)
     ```

- 如何在Drools中对外（Java）写出。（使用静态方法）

  1. 声明及静态方法（java）

     ```
     public class MessageList {
         public static List<String> testList = new Vector<>();
     
         public static void init(){
             testList.clear();
         }
     
         public static void add(Object result){
             if (result != null) {
                 if (result instanceof String) {
                     testList.add((String) result);
                 }
             }
         }
     }
     ```

  2. 调用方法进行操作(java)

     ```java
         @Test
         public void test1(){
             //构造订单对象，设置原始价格，由规则引擎根据优惠规则计算优惠后的价格
             Order order = new Order();
             order.setOriginalPrice(59D);
             //将数据提供给规则引擎，规则引擎会根据提供的数据进行规则匹配
             Order order1 = new Order();
             order1.setOriginalPrice(91D);
     
             MessageList.init();
     
             session.insert(order);
     //        session.fireAllRules();
             session.insert(order1);
             //激活规则引擎，如果规则匹配成功则执行规则
             session.fireAllRules();
             //关闭会话
             session.dispose();
             System.out.println("优惠前原始价格：" + order.getOriginalPrice() +
                     "，优惠后价格：" + order.getRealPrice());
             System.out.println("优惠前原始价格：" + order1.getOriginalPrice() +
                     "，优惠后价格：" + order1.getRealPrice());
             for (String s : MessageList.testList) {
                 System.out.println(s);
             }
         }
     ```

  3. drools

     ```java
     //添加方法
     function void setString(String s) {
         MessageList.add(s);
     }
     
     //规则一：所购图书总价在100元以下的没有优惠
     rule "book_discount_1"
     
         when
             $order:Order(originalPrice < 100)
         then
             $order.setRealPrice($order.getOriginalPrice() - 10);
             setString($order.getOriginalPrice().toString());
     end
     ```

- 单条数据对应单个规则可以批量传入数据，**如果对应多段规则必须分组单条传入**

