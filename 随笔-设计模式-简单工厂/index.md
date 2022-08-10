# 随笔-设计模式-简单工厂


# 随笔-设计模式-简单工厂

1. 学习标准规范类

   ```java
   package org.jsh.newinstance;
   
   public interface ICourse {
   	void learn();
   }
   ```

2. 学习html类

   ```java
   package org.jsh.newinstance;
   
   public class HtmlCourse implements ICourse{
   	 @Override
   	 public void learn() {
   	 	System.out.println("html");
   	 }
   }
   ```

3. 学习Java类

   ```java
   package org.jsh.newinstance;
   
   public class JavaCourse implements ICourse{
    @Override
   public void learn() {
   	System.out.println("java");
   }
   }
   ```

4. 学习代工厂（Factory）

   ```java
   package org.jsh.factory;
   
   import org.jsh.newinstance.HtmlCourse;
   import org.jsh.newinstance.ICourse;
   import org.jsh.newinstance.JavaCourse;
   
   public class CourseFactory {
   //	根据名字获取
   	public static ICourse getCourse(String name) {
   		if(name.equals("java")) {
   			return new JavaCourse();
   		}else {
   			return new HtmlCourse();
   		}
   	}
   }
   
   ```

5. 演示

   ```java
   	public void learn(String name) {
   		ICourse course = CourseFactory.getCourse(name);
   		course.learn();
   	}
   ```

   

