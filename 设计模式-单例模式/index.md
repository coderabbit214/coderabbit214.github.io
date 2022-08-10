# 设计模式-单例模式


# 设计模式-单例模式

> 　java中单例模式是一种常见的设计模式，单例模式的写法有好几种，这里主要介绍三种：懒汉式单例、饿汉式单例、登记式单例。
> 　　单例模式有以下特点：
> 　　1、单例类只能有一个实例。
> 　　2、单例类必须自己创建自己的唯一实例。
> 　　3、单例类必须给所有其他对象提供这一实例。
> 　　单例模式确保某个类只有一个实例，而且自行实例化并向整个系统提供这个实例。在计算机系统中，线程池、缓存、日志对象、对话框、打印机、显卡的驱动程序对象常被设计成单例。这些应用都或多或少具有资源管理器的功能。每台计算机可以有若干个打印机，但只能有一个Printer Spooler，以避免两个打印作业同时输出到打印机中。每台计算机可以有若干通信端口，系统应当集中管理这些通信端口，以避免一个通信端口同时被两个请求同时调用。总之，选择单例模式就是为了避免不一致状态，避免政出多头。
>
> 作用：控制实例数目，节省系统资源

## 图

![62576915-36E0-4B67-B078-704699CA980A](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/62576915-36E0-4B67-B078-704699CA980A.jpg)

## 代码实现

1. 懒汉式，线程不安全

   ```java
   /**
    * 单例模式的几种实现方式:
    * 1、懒汉式，线程不安全
    * 是否 Lazy 初始化：是
    *
    * 是否多线程安全：否
    *
    * 实现难度：易
    *
    * 描述：这种方式是最基本的实现方式，这种实现最大的问题就是不支持多线程。因为没有加锁 synchronized，所以严格意义上它并不算单例模式。
    * 这种方式 lazy loading 很明显，不要求线程安全，在多线程不能正常工作。
    */
   //创建Singleton类
   class Singleton1 {
       //创建类
       private static Singleton1 singleton;
       //私有化构造函数，避免该类被实例化
       private Singleton1(){
   
       }
       //获取唯一可用对象
       public static Singleton1 getSingleton(){
           if (singleton == null){
               singleton = new Singleton1();
           }
           return singleton;
       }
       public void showMessage(){
           System.out.println("Hello World!");
       }
   }
   ```

   > 线程不安全:
   >
   > ![20180327123758498](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/捕获.PNG)

2. 实现线程安全的懒汉模式

   ```java
   /**
    * 2、懒汉式，线程安全
    * 是否 Lazy 初始化：是
    *
    * 是否多线程安全：是
    *
    * 实现难度：易
    *
    * 描述：这种方式具备很好的 lazy loading，能够在多线程中很好的工作，但是，效率很低，99% 情况下不需要同步。
    * 优点：第一次调用才初始化，避免内存浪费。
    * 缺点：必须加锁 synchronized 才能保证单例，但加锁会影响效率。
    * getSingleton() 的性能对应用程序不是很关键（该方法使用不太频繁）。
    */
   //创建Singleton类
   class Singleton2 {
       //创建类
       private static Singleton2 singleton;
       //私有化构造函数，避免该类被实例化
       private Singleton2(){
   
       }
       //获取唯一可用对象
       public synchronized static Singleton2 getSingleton(){
           if (singleton == null){
               singleton = new Singleton2();
           }
           return singleton;
       }
       public void showMessage(){
           System.out.println("Hello World!");
       }
   }
   ```

3. 双重锁模式，实现线程安全的懒汉模式同时效率高

   ```java
   /**
    * 4、双检锁/双重校验锁（DCL，即 double-checked locking）
    * JDK 版本：JDK1.5 起
    *
    * 是否 Lazy 初始化：是
    *
    * 是否多线程安全：是
    *
    * 实现难度：较复杂
    *
    * 描述：这种方式采用双锁机制，安全且在多线程情况下能保持高性能。
    * getSingleton() 的性能对应用程序很关键。
    */
   //创建Singleton类
   class Singleton4 {
       //创建类
       private volatile static Singleton4 singleton;
       //私有化构造函数，避免该类被实例化
       private Singleton4(){
   
       }
       //获取唯一可用对象
       public synchronized static Singleton4 getSingleton(){
           if (singleton == null) {
               synchronized (Singleton4.class) {
                   if (singleton == null) {
                       singleton = new Singleton4();
                   }
               }
           }
           return singleton;
       }
       public void showMessage(){
           System.out.println("Hello World!");
       }
   }
   ```

4. 饿汉式

   > 空间换时间

   ```java
   /**
    * 3、饿汉式
    * 是否 Lazy 初始化：否
    *
    * 是否多线程安全：是
    *
    * 实现难度：易
    *
    * 描述：这种方式比较常用，但容易产生垃圾对象。
    * 优点：没有加锁，执行效率会提高。
    * 缺点：类加载时就初始化，浪费内存。
    * 它基于 classloader 机制避免了多线程的同步问题，不过，singleton 在类装载时就实例化，虽然导致类装载
    * 的原因有很多种，在单例模式中大多数都是调用 getSingleton 方法， 但是也不能确定有其他的方式（或者其他
    * 的静态方法）导致类装载，这时候初始化 singleton 显然没有达到 lazy loading 的效果。
    */
   //创建Singleton类
   class Singleton3 {
       //创建类
       private static Singleton3 singleton = new Singleton3();
       //私有化构造函数，避免该类被实例化
       private Singleton3(){
   
       }
       //获取唯一可用对象
       public static Singleton3 getSingleton(){
           return singleton;
       }
       public void showMessage(){
           System.out.println("Hello World!");
       }
   }
   ```

5. 使用静态内部类实现单例模式

   ```java
   /**
    * 5、登记式/静态内部类
    * 是否 Lazy 初始化：是
    *
    * 是否多线程安全：是
    *
    * 实现难度：一般
    *
    * 描述：这种方式能达到双检锁方式一样的功效，但实现更简单。对静态域使用延迟初始化，应使用这种方式而不是双检锁方式。
    *      这种方式只适用于静态域的情况，双检锁方式可在实例域需要延迟初始化时使用。
    * 这种方式同样利用了 classloader 机制来保证初始化 singleton 时只有一个线程，它跟第 3 种方式不同的是：
    * 第 3 种方式只要 Singleton 类被装载了，那么 singleton 就会被实例化（没有达到 lazy loading 效果），
    * 而这种方式是 Singleton 类被装载了，singleton 不一定被初始化。因为 SingletonHolder 类没有被主动使
    * 用，只有通过显式调用 getSingleton 方法时，才会显式装载 SingletonHolder 类，从而实例化 singleton。
    * 想象一下，如果实例化 singleton 很消耗资源，所以想让它延迟加载，另外一方面，又不希望在 Singleton 类加
    * 载时就实例化，因为不能确保 Singleton 类还可能在其他的地方被主动使用从而被加载，那么这个时候实例化 singleton
    * 显然是不合适的。这个时候，这种方式相比第 3 种方式就显得很合理。
    */
   //创建Singleton类
   class Singleton5 {
       private static class SingletonHolder5 {
           private static final Singleton5 SINGLETON = new Singleton5();
       }
       private Singleton5 (){}
       public static final Singleton5 getSingleton() {
           return SingletonHolder5.SINGLETON;
       }
   }
   ```

6. 枚举

   ```java
   /**
    * 6、枚举
    * JDK 版本：JDK1.5 起
    *
    * 是否 Lazy 初始化：否
    *
    * 是否多线程安全：是
    *
    * 实现难度：易
    *
    * 描述：这种实现方式还没有被广泛采用，但这是实现单例模式的最佳方法。它更简洁，自动支持序列化机制，绝对防止多次实例化。
    * 这种方式是 Effective Java 作者 Josh Bloch 提倡的方式，它不仅能避免多线程同步问题，而且还自动支持序列化机制，
    * 防止反序列化重新创建新的对象，绝对防止多次实例化。不过，由于 JDK1.5 之后才加入 enum 特性，用这种方式写不免让人感觉生疏，在实际工作中，也很少用。
    * 不能通过 reflection attack 来调用私有构造方法。
    */
   enum Singleton6 {
       SINGLETON;
       public void whateverMethod() {
           //功能处理
       }
   }
   ```

## 破坏单例模式的三种方式

> 1. 反射
> 2. 序列化
> 3. 克隆

```java
package com.designPattern.singletonPattern;

import java.io.*;
import java.lang.reflect.Constructor;

/**
 * 序列化对单例的破坏
 * @author Administrator
 *
 */
public class SingletonTest {
	
	public static void main(String[] args) throws Exception{

		Singleton originSingleton = Singleton.getInstance();
		System.out.println("-----------序列化----------------------");
	     ByteArrayOutputStream bos = new ByteArrayOutputStream();
	     ObjectOutputStream oos = new ObjectOutputStream(bos);
	      oos.writeObject(Singleton.getInstance());
	      ByteArrayInputStream bis = new ByteArrayInputStream(bos.toByteArray());
	      ObjectInputStream ois = new ObjectInputStream(bis);
	      Singleton serializeSingleton = (Singleton) ois.readObject();
	      System.out.println(originSingleton == serializeSingleton);//false
	      
	      System.out.println("-----------反射----------------------");
	      //通过反射获取
	        Constructor<Singleton> cons = Singleton.class.getDeclaredConstructor();
	        cons.setAccessible(true);
	        Singleton reflextSingleton = cons.newInstance();
	        System.out.println(reflextSingleton == originSingleton);//false
	        
	        System.out.println("---------------------------克隆----------------------");

	        Singleton cloneSingleton = (Singleton) originSingleton.clone();
	        System.out.println(cloneSingleton == originSingleton);//false
	    
	}
	
	private static class Singleton  implements Serializable,Cloneable{
		/**
		 * 1.构造方法私有化，外部不能new
		 */
		private Singleton() {}
		
		
		//2.本类内部创建对象实例
		private static  volatile  Singleton instance;
		
		
		//3.提供一个公有的静态方法，返回实例对象
		public static  Singleton getInstance() {
			if(instance == null) {
				synchronized (Singleton.class) {
					if(instance == null) {
						instance = new Singleton();
					}
				}
			}
			return instance;
		}
		
		@Override
		 protected Object clone() throws CloneNotSupportedException {
		     return super.clone();
		  }
		 
	}
	
}

```

运行结果

![捕获](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/20180327123758498.png)

**解决方法**

> 1、防止反射
>
>   定义一个全局变量，当第二次创建的时候抛出异常
>
> 2、防止克隆破坏
>
>    重写clone(),直接返回单例对象
>
> 3、防止序列化破坏
>
>   添加readResolve(),返回Object对象

```java
package com.designPattern.singletonPattern;

import java.io.*;
import java.lang.reflect.Constructor;

/**
 * 序列化对单例的破坏
 * @author Administrator
 *
 */
public class SingletonTest {
	
	public static void main(String[] args) throws Exception{

		Singleton originSingleton = Singleton.getInstance();
		System.out.println("-----------序列化----------------------");
	     ByteArrayOutputStream bos = new ByteArrayOutputStream();
	     ObjectOutputStream oos = new ObjectOutputStream(bos);
	      oos.writeObject(Singleton.getInstance());
	      ByteArrayInputStream bis = new ByteArrayInputStream(bos.toByteArray());
	      ObjectInputStream ois = new ObjectInputStream(bis);
	      Singleton serializeSingleton = (Singleton) ois.readObject();
	      System.out.println(originSingleton == serializeSingleton);//false
	      
	      System.out.println("-----------反射----------------------");
	      //通过反射获取
	        Constructor<Singleton> cons = Singleton.class.getDeclaredConstructor();
	        cons.setAccessible(true);
	        Singleton reflextSingleton = cons.newInstance();
	        System.out.println(reflextSingleton == originSingleton);//false
	        
	        System.out.println("---------------------------克隆----------------------");

	        Singleton cloneSingleton = (Singleton) originSingleton.clone();
	        System.out.println(cloneSingleton == originSingleton);//false
	    
	}
	
	private static class Singleton  implements Serializable,Cloneable{
		private static volatile boolean isCreate = false;//默认是第一次创建
		/**
		 * 1.构造方法私有化，外部不能new
		 */
		private Singleton() {
			if(isCreate) {
				throw new RuntimeException("已然被实例化一次，不能在实例化");
			}
			isCreate = true;
		}


		//2.本类内部创建对象实例
		private static  volatile  Singleton instance;


		//3.提供一个公有的静态方法，返回实例对象
		public static  Singleton getInstance() {
			if(instance == null) {
				synchronized (Singleton.class) {
					if(instance == null) {
						instance = new Singleton();
					}
				}
			}
			return instance;
		}

		@Override
		protected Object clone() throws CloneNotSupportedException {
			return instance;
		}

		/**
		 * 防止序列化破环
		 * @return
		 */
		private Object readResolve() {
			return instance;
		}

	}


}


```

## 完整代码

```java
package com.designPattern.singletonPattern;

/**
 * 单例模式
 */
public class SingletonPattern {
    public static void main(String[] args) {
        Singleton singleton = Singleton.getSingleton();
        singleton.showMessage();
    }
}
//创建Singleton类
class Singleton {
    //创建类
    private static Singleton singleton = new Singleton();
    //私有化构造函数，避免该类被实例化
    private Singleton(){

    }
    //获取唯一可用对象
    public static Singleton getSingleton(){
        return singleton;
    }
    public void showMessage(){
        System.out.println("Hello World!");
    }
}
/**
 * 单例模式的几种实现方式:
 * 1、懒汉式，线程不安全
 * 是否 Lazy 初始化：是
 *
 * 是否多线程安全：否
 *
 * 实现难度：易
 *
 * 描述：这种方式是最基本的实现方式，这种实现最大的问题就是不支持多线程。因为没有加锁 synchronized，所以严格意义上它并不算单例模式。
 * 这种方式 lazy loading 很明显，不要求线程安全，在多线程不能正常工作。
 */
//创建Singleton类
class Singleton1 {
    //创建类
    private static Singleton1 singleton;
    //私有化构造函数，避免该类被实例化
    private Singleton1(){

    }
    //获取唯一可用对象
    public static Singleton1 getSingleton(){
        if (singleton == null){
            singleton = new Singleton1();
        }
        return singleton;
    }
    public void showMessage(){
        System.out.println("Hello World!");
    }
}
/**
 * 2、懒汉式，线程安全
 * 是否 Lazy 初始化：是
 *
 * 是否多线程安全：是
 *
 * 实现难度：易
 *
 * 描述：这种方式具备很好的 lazy loading，能够在多线程中很好的工作，但是，效率很低，99% 情况下不需要同步。
 * 优点：第一次调用才初始化，避免内存浪费。
 * 缺点：必须加锁 synchronized 才能保证单例，但加锁会影响效率。
 * getSingleton() 的性能对应用程序不是很关键（该方法使用不太频繁）。
 */
//创建Singleton类
class Singleton2 {
    //创建类
    private static Singleton2 singleton;
    //私有化构造函数，避免该类被实例化
    private Singleton2(){

    }
    //获取唯一可用对象
    public synchronized static Singleton2 getSingleton(){
        if (singleton == null){
            singleton = new Singleton2();
        }
        return singleton;
    }
    public void showMessage(){
        System.out.println("Hello World!");
    }
}
/**
 * 3、饿汉式
 * 是否 Lazy 初始化：否
 *
 * 是否多线程安全：是
 *
 * 实现难度：易
 *
 * 描述：这种方式比较常用，但容易产生垃圾对象。
 * 优点：没有加锁，执行效率会提高。
 * 缺点：类加载时就初始化，浪费内存。
 * 它基于 classloader 机制避免了多线程的同步问题，不过，singleton 在类装载时就实例化，虽然导致类装载
 * 的原因有很多种，在单例模式中大多数都是调用 getSingleton 方法， 但是也不能确定有其他的方式（或者其他
 * 的静态方法）导致类装载，这时候初始化 singleton 显然没有达到 lazy loading 的效果。
 */
//创建Singleton类
class Singleton3 {
    //创建类
    private static Singleton3 singleton = new Singleton3();
    //私有化构造函数，避免该类被实例化
    private Singleton3(){

    }
    //获取唯一可用对象
    public static Singleton3 getSingleton(){
        return singleton;
    }
    public void showMessage(){
        System.out.println("Hello World!");
    }
}
/**
 * 4、双检锁/双重校验锁（DCL，即 double-checked locking）
 * JDK 版本：JDK1.5 起
 *
 * 是否 Lazy 初始化：是
 *
 * 是否多线程安全：是
 *
 * 实现难度：较复杂
 *
 * 描述：这种方式采用双锁机制，安全且在多线程情况下能保持高性能。
 * getSingleton() 的性能对应用程序很关键。
 */
//创建Singleton类
class Singleton4 {
    //创建类
    private volatile static Singleton4 singleton;
    //私有化构造函数，避免该类被实例化
    private Singleton4(){

    }
    //获取唯一可用对象
    public synchronized static Singleton4 getSingleton(){
        if (singleton == null) {
            synchronized (Singleton4.class) {
                if (singleton == null) {
                    singleton = new Singleton4();
                }
            }
        }
        return singleton;
    }
    public void showMessage(){
        System.out.println("Hello World!");
    }
}
/**
 * 5、登记式/静态内部类
 * 是否 Lazy 初始化：是
 *
 * 是否多线程安全：是
 *
 * 实现难度：一般
 *
 * 描述：这种方式能达到双检锁方式一样的功效，但实现更简单。对静态域使用延迟初始化，应使用这种方式而不是双检锁方式。
 *      这种方式只适用于静态域的情况，双检锁方式可在实例域需要延迟初始化时使用。
 * 这种方式同样利用了 classloader 机制来保证初始化 singleton 时只有一个线程，它跟第 3 种方式不同的是：
 * 第 3 种方式只要 Singleton 类被装载了，那么 singleton 就会被实例化（没有达到 lazy loading 效果），
 * 而这种方式是 Singleton 类被装载了，singleton 不一定被初始化。因为 SingletonHolder 类没有被主动使
 * 用，只有通过显式调用 getSingleton 方法时，才会显式装载 SingletonHolder 类，从而实例化 singleton。
 * 想象一下，如果实例化 singleton 很消耗资源，所以想让它延迟加载，另外一方面，又不希望在 Singleton 类加
 * 载时就实例化，因为不能确保 Singleton 类还可能在其他的地方被主动使用从而被加载，那么这个时候实例化 singleton
 * 显然是不合适的。这个时候，这种方式相比第 3 种方式就显得很合理。
 */
//创建Singleton类
class Singleton5 {
    private static class SingletonHolder5 {
        private static final Singleton5 SINGLETON = new Singleton5();
    }
    private Singleton5 (){}
    public static final Singleton5 getSingleton() {
        return SingletonHolder5.SINGLETON;
    }
}
/**
 * 6、枚举
 * JDK 版本：JDK1.5 起
 *
 * 是否 Lazy 初始化：否
 *
 * 是否多线程安全：是
 *
 * 实现难度：易
 *
 * 描述：这种实现方式还没有被广泛采用，但这是实现单例模式的最佳方法。它更简洁，自动支持序列化机制，绝对防止多次实例化。
 * 这种方式是 Effective Java 作者 Josh Bloch 提倡的方式，它不仅能避免多线程同步问题，而且还自动支持序列化机制，
 * 防止反序列化重新创建新的对象，绝对防止多次实例化。不过，由于 JDK1.5 之后才加入 enum 特性，用这种方式写不免让人感觉生疏，在实际工作中，也很少用。
 * 不能通过 reflection attack 来调用私有构造方法。
 */
enum Singleton6 {
    SINGLETON;
    public void whateverMethod() {
        //功能处理
    }
}
```


