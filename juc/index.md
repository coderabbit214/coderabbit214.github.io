# JUC


## JUC

### 什么是JUC

#### 进程与线程

进程：计算机进行资源调度的基本单元（打开一个软件）。

线程：系统分配时间调度的基本单元。

#### 线程的状态

- 新建
- 运行
- 阻塞
- 等待
- 超时等待
- 终止

#### wait和sleep

(1) sleep 是 Thread 的静态方法，wait 是 Object 的方法，任何对象实例都 能调用。

(2) sleep 不会释放锁，它也不需要占用锁。wait 会释放锁，但调用它的前提 是当前线程占有锁(即代码要在 synchronized 中)。

(3) 它们都可以被 interrupted 方法中断。

#### 并发和并行

**并发:**同一时刻多个线程在访问同一个资源，多个线程对一个点 

**并行:**多项工作一起执行，之后再汇总

#### 管程Monitor

就是说的**锁**

![截屏2021-10-02 17.32.19](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/%E6%88%AA%E5%B1%8F2021-10-02%2017.32.19.png)

#### 用户线程和守护线程

**用户线程**:平时用到的普通线程,自定义线程 

**守护线程**:运行在后台,是一种特殊的线程,比如垃圾回收 

当主线程结束后,用户线程还在运行,JVM存活 

如果没有用户线程,都是守护线程,JVM结束

### LOCK接口

#### 多线程编程步骤

![03-多线程编程步骤](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/03-%E5%A4%9A%E7%BA%BF%E7%A8%8B%E7%BC%96%E7%A8%8B%E6%AD%A5%E9%AA%A4.png)

#### Synchronized

synchronized 是 Java 中的关键字，是一种同步锁。它修饰的对象有以下几种:

1. 修饰一个代码块，被修饰的代码块称为同步语句块，其作用的范围是大括号{} 括起来的代码，作用的对象是调用这个代码块的对象;
2. 修饰一个方法，被修饰的方法称为同步方法，其作用的范围是整个方法，作用 的对象是调用这个方法的对象;
   - 虽然可以使用 synchronized 来定义方法，但 synchronized 并不属于方法定 义的一部分，因此，synchronized 关键字不能被继承。如果在父类中的某个方 法使用了 synchronized 关键字，而在子类中覆盖了这个方法，在子类中的这 个方法默认情况下并不是同步的，而必须显式地在子类的这个方法中加上 synchronized 关键字才可以。当然，还可以在子类方法中调用父类中相应的方 法，这样虽然子类中的方法不是同步的，但子类调用了父类的同步方法，因此， 子类的方法也就相当于同步了。
3. 修改一个静态的方法，其作用的范围是整个静态方法，作用的对象是这个类的 所有对象;
4. 修改一个类，其作用的范围是synchronized后面括号括起来的部分，作用主 的对象是这个类的所有对象。

例子：

```java
//创建资源类
class Ticket{
    //票数
    private int number = 30;
    //操作方法：卖票
    public synchronized void sale(){
        //判断：是否有票
        if (number>0) {
            System.out.println(Thread.currentThread().getName()+"::"+number--+"还剩"+number);
        }
    }
}
```

#### ReetranLock可重入锁

可重入锁：一个走了才能再进一个

例子：

```java
class LTicket{
    private int number = 30;
  //创建可重入锁 公平锁
    private final ReentrantLock lock = new ReentrantLock(true);
    public void sale(){
        lock.lock();
        try {
            if (number>0){
                System.out.println(Thread.currentThread().getName()+"::"+number--+"剩余："+number);
            }
        } finally {
            lock.unlock();
        }
    }
}
```

采用 Lock，必须主动去释放锁，并且在发生异常时，不会自动释放锁。因此一 般来说，使用 Lock 必须在 try{}catch{}块中进行，并且将释放锁的操作放在 finally 块中进行，以保证锁一定被被释放，防止死锁的发生。

#### Lock和Synchronized的不同

1. Lock是一个接口，而synchronized是Java中的关键字，synchronized是内 置的语言实现;

2. synchronized在发生异常时，会自动释放线程占有的锁，因此不会导致死锁现 象发生;而 Lock 在发生异常时，如果没有主动通过 unLock()去释放锁，则很 可能造成死锁现象，因此使用 Lock 时需要在 finally 块中释放锁;

3. Lock可以让等待锁的线程响应中断，而synchronized却不行，使用 synchronized 时，等待的线程会一直等待下去，不能够响应中断;

4. 通过Lock可以知道有没有成功获取锁，而synchronized却无法办到。

5. Lock可以提高多个线程进行读操作的效率。

   在性能上来说，如果竞争资源不激烈，两者的性能是差不多的，而当竞争资源 非常激烈时(即有大量线程同时竞争)，此时 Lock 的性能要远远优于 synchronized。

### 线程间通信

#### Synchronized

```java
package com.jsh.juc.sync;

class Share {
    //初始值
    private int number = 0;

    //+1的方法
    public synchronized void add() throws InterruptedException {
      //判断 干活 通知
        while (number != 0) {
            this.wait();
        }
      //干活
        number++;
        System.out.println(Thread.currentThread().getName() + "::" + number);
      //通知
        this.notifyAll();
    }

    //+1的方法
    public synchronized void dno() throws InterruptedException {
        while (number == 0) {
            this.wait();
        }
        number--;
        System.out.println(Thread.currentThread().getName() + "::" + number);
        this.notifyAll();
    }
}

public class ThreadDemo {
    public static void main(String[] args) {
        Share sale = new Share();
        new Thread(() -> {
            for (int i = 0; i < 40; i++) {
                try {
                    sale.add();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "AA").start();

        new Thread(() -> {
            for (int i = 0; i < 40; i++) {
                try {
                    sale.dno();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "BB").start();

        new Thread(() -> {
            for (int i = 0; i < 40; i++) {
                try {
                    sale.add();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "CC").start();
    }
}
```

#### Lock

```java
class Share {
    //初始值
    private int number = 0;

    //创建lock
    private Lock lock = new ReentrantLock();
    private Condition condition = lock.newCondition();

    //+1的方法
    public void add() throws InterruptedException {
        //上锁
        lock.lock();
        try {
            //判断（使用while不使用if）
            while (number != 0) {
                //等待
                condition.await();
            }
            number++;
            System.out.println(Thread.currentThread().getName() + "::" + number);
            //通知
            condition.signalAll();
        } finally {
            //解锁
            lock.unlock();
        }


    }

    //-1的方法
    public void dno() throws InterruptedException {
        //上锁
        lock.lock();
        try {
            while (number == 0) {
                //等待
                condition.await();
            }
            number--;
            System.out.println(Thread.currentThread().getName() + "::" + number);
            //通知
            condition.signalAll();
        } finally {
            //解锁
            lock.unlock();
        }
    }
}

public class ThreadDemo {
    public static void main(String[] args) {
        Share sale = new Share();
        new Thread(() -> {
            for (int i = 0; i < 40; i++) {
                try {
                    sale.add();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "AA").start();

        new Thread(() -> {
            for (int i = 0; i < 40; i++) {
                try {
                    sale.dno();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "BB").start();

        new Thread(() -> {
            for (int i = 0; i < 40; i++) {
                try {
                    sale.add();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "CC").start();
    }
}
```

#### 虚假唤醒问题

等待判断应该放在while中，不应该放在if中（if只能判断一次）

### 线程间定制化通信

>  private Condition c3 = lock.newCondition();
>
>  通知谁 调用谁的signal()方法
>
>  c3.signal();

```java
/*
线程间定制化通知
 */
//第一步 创建资源类
class ShareResource { 
    //定义标志位
    private int flag = 1;//1:aa,2:bb,3:cc
    //创建lock锁
    private Lock lock = new ReentrantLock();

    // 创建三个Condition
    private Condition c1 = lock.newCondition();
    private Condition c2 = lock.newCondition();
    private Condition c3 = lock.newCondition();

    //打印5次，参数第几轮
    public void print5(int loop) throws InterruptedException {
        //上锁
        lock.lock();
        try {
            //判断
            while (flag != 1) {
                c1.await();
            }
            //干活
            for (int i = 0; i < 5; i++) {
                System.out.println(Thread.currentThread().getName()+"::"+i+"第几轮："+loop);
            }
            //通知
            flag = 2;
            //通知c2
            c2.signal();
        }finally {
            lock.unlock();
        }
    }

    //打印10次，参数第几轮
    public void print10(int loop) throws InterruptedException {
        //上锁
        lock.lock();
        try {
            //判断
            while (flag != 2) {
                c2.await();
            }
            //干活
            for (int i = 0; i < 10; i++) {
                System.out.println(Thread.currentThread().getName()+"::"+i+"第几轮："+loop);
            }
            //通知
            flag = 3;
            c3.signal();
        }finally {
            lock.unlock();
        }
    }

    //打印15次，参数第几轮
    public void print15(int loop) throws InterruptedException {
        //上锁
        lock.lock();
        try {
            //判断
            while (flag != 3) {
                c3.await();
            }
            //干活
            for (int i = 0; i < 15; i++) {
                System.out.println(Thread.currentThread().getName()+"::"+i+"第几轮："+loop);
            }
            //通知
            flag = 1;
            c1.signal();
        }finally {
            lock.unlock();
        }
    }

}



public class ThreadDemo3 {

    public static void main(String[] args) {
        ShareResource shareResource = new ShareResource();
        new Thread(() -> {
            try {
                for (int i = 0; i < 5; i++) {
                    shareResource.print5(i);
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        },"aa").start();

        new Thread(() -> {
            try {
                for (int i = 0; i < 5; i++) {
                    shareResource.print10(i);
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        },"bb").start();

        new Thread(() -> {
            try {
                for (int i = 0; i < 5; i++) {
                    shareResource.print15(i);
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        },"cc").start();
    }
}

```

### 集合的线程安全

**ArrayList,HashMap,HashSet线程不安全**

ArrayList解决方法

- Vector

- Collections

- CopyOnWriteArrayList

  写时复制技术：并发读，独立写：每次写入的时候先复制一份，写入新内容后再合并

  ```java
  public class ArrayListSync {
      public static void main(String[] args) {
  //        List<String> list = new ArrayList<>();
          //1.Vector
  //        List<String> list = new Vector<>();
          //2.Collections
  //        List<String> list = Collections.synchronizedList(new ArrayList<>());
          //CopyOnWriteArrayList
          CopyOnWriteArrayList<String> list = new CopyOnWriteArrayList<>();
          for (int i = 0; i < 400; i++) {
              new Thread(()->{
                  list.add(UUID.randomUUID().toString().substring(0,8));
                  System.out.println(list);
              }).start();
          }
      }
  }
  ```

HashSet解决方法

CopyOnWriteArraySet

HashMap解决方法

ConcurrentHashMap

### 多线程锁

#### 锁的八种情况

synchronized实现同步的基础:Java中的每一个对象都可以作为锁具体表现为以下3种形式。

- 对于普通同步方法，锁是当前实例对象。
- 对于静态同步方法，锁是当前类的Class对象。
- 对于同步方法块，锁是Synchonized括号里配置的对象

#### 公平锁和非公平锁

> 多线程之间工作的平均

```java
    //创建lock锁 true：公平 false：非公平
    private Lock lock = new ReentrantLock(true);
```

- 公平锁：效率低，平均
- 非公平锁：效率高，不平均

#### 死锁

![16-死锁](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/16-%E6%AD%BB%E9%94%81.png)



### Callable

|          | Runnable | Callable |
| -------- | -------- | -------- |
| 返回值   | 无       | 有       |
| 异常     | 无       | 有       |
| 实现方法 | run      | call     |

#### 代码实现 

```java
package com.jsh.juc.callable;


import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.FutureTask;

class MyThread implements Callable{

    @Override
    public Object call() throws Exception {
        return 200;
    }
}


public class Demo1 {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        //方法一
        //FutureTask 未来任务
        FutureTask<Integer> futureTask1 = new FutureTask<>(new MyThread());

        //方法二
        //lam表达式
        FutureTask<Integer> futureTask2 = new FutureTask<>(() ->{
            System.out.println(Thread.currentThread().getName()+"2");
            return 1024;
        });

      //创建线程 ，启动
        new Thread(futureTask2,"lucy").start();

        //线程是否结束
        while (!futureTask2.isDone()) {
            System.out.println("wait。。。。。");
        }

        //第一次调用 计算 返回结果
        System.out.println(futureTask2.get());

        //第二次调用直接返回结果
        System.out.println(futureTask2.get());
    }
}

```

### JUC辅助类

#### 减少计数CountDownLatch

> - 创建计数器`CountDownLatch countDownLatch = new CountDownLatch(6);`
> - 计数-1`countDownLatch.countDown();`
> - 等待，当计数为0时继续`countDownLatch.await();`

```java
public class CountDownLatchDemo {
    public static void main(String[] args) throws InterruptedException {

        //计数器
        CountDownLatch countDownLatch = new CountDownLatch(6);
        for (int i = 1; i < 7; i++) {
            new Thread(() -> {
                System.out.println(Thread.currentThread().getName()+"走了");
                //计数-1
                countDownLatch.countDown();
            },String.valueOf(i)).start();
        }

        //等待
        countDownLatch.await();
        System.out.println("班长锁门");
    }
}
```

#### 循环栅栏CyclicBarrier

> - 创建栅栏：
>
>   `CyclicBarrier cyclicBarrier = new CyclicBarrier(7,()->{
>               System.out.println("召唤神龙");
>           });`
>
> - 等待：` cyclicBarrier.await();`

```java
public class CyclicBarrierDemo {
    public static void main(String[] args) {
        //循环栅栏
        CyclicBarrier cyclicBarrier = new CyclicBarrier(7,()->{
            System.out.println("召唤神龙");
        });
        
        for (int i = 1; i <= 7; i++) {
            new Thread(() -> {
                try {
                    System.out.println(Thread.currentThread().getName()+"星龙珠被找到了");
                    //等待
                    cyclicBarrier.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (BrokenBarrierException e) {
                    e.printStackTrace();
                }
            },String.valueOf(i)).start();
        }
    }
}
```

#### 信号灯Semaphore

> 创建：`Semaphore semaphore = new Semaphore(3);`
>
> 占领：`semaphore.acquire();`
>
> 释放：`semaphore.release();`

```java
/**
 * 信号灯
 */
//模拟六辆汽车，停3个停车位
public class SemaphoreDemo {
    public static void main(String[] args) {
        //模拟三个车位
        Semaphore semaphore = new Semaphore(3);

        //模拟6辆汽车
        for (int i = 0; i < 6; i++) {
            new Thread(()->{
                try {
                    //占车位
                    semaphore.acquire();
                    System.out.println(Thread.currentThread().getName()+"抢到了车位");

                    //设置随机停车时间
                    TimeUnit.SECONDS.sleep(new Random().nextInt(5));
                    System.out.println(Thread.currentThread().getName()+"----------离开了车位");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    //释放车位
                    semaphore.release();
                }
            },String.valueOf(i)).start();
        }
    }
}
```

### 读写锁ReentrantReadWriteLock

#### 读写锁概念

现实中有这样一种场景:对共享资源有读和写的操作，且写操作没有读操作那 么频繁。在没有写操作的时候，多个线程同时读一个资源没有任何问题，所以 应该允许多个线程同时读取共享资源;但是如果一个线程想去写这些共享资源， 就不应该允许其他线程对该资源进行读和写的操作了。

针对这种场景，**JAVA 的并发包提供了读写锁 ReentrantReadWriteLock， 它表示两个锁，一个是读操作相关的锁，称为共享锁;一个是写相关的锁，称为排他锁**

#### 演示读写锁

```java
//资源类
class MyCache {
    private volatile Map<String, String> map = new HashMap<>();

    //创建读写锁对象
    private ReadWriteLock readWriteLock = new ReentrantReadWriteLock();

    //放数据
    public void put(String key, String value) {
        //添加写锁
        readWriteLock.writeLock().lock();
        try {
            System.out.println(Thread.currentThread().getName() + "正在写数据");
            //暂停一会
            TimeUnit.MICROSECONDS.sleep(300);
            //放数据
            map.put(key, value);
            System.out.println(Thread.currentThread().getName() + "写完了" + key);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            //释放锁
            readWriteLock.writeLock().unlock();
        }
    }

    //取数据
    public String get(String key) {
        //添加读锁
        readWriteLock.readLock().lock();
        String res = null;
        try {
            System.out.println(Thread.currentThread().getName() + "正在读数据");
            //暂停一会
            TimeUnit.MICROSECONDS.sleep(300);
            //放数据
            res = map.get(key);
            System.out.println(Thread.currentThread().getName() + "读完了" + key);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            //释放锁
            readWriteLock.readLock().unlock();
        }
        return res;
    }
}

public class ReadWriteLockDemo {
    public static void main(String[] args) {
        MyCache myCache = new MyCache();

        //创建线程 放数据
        for (int i = 0; i < 5; i++) {
            int finalI = i;
            new Thread(() -> {
                myCache.put(finalI + "", finalI + "");
            }, i + "存").start();
        }

        //创建线程 取数据
        for (int i = 0; i < 5; i++) {
            int finalI = i;
            new Thread(() -> {
                myCache.get(finalI + "");
            }, i + "取").start();
        }
    }
}

```

#### 读写锁演变

![14-读写锁演变](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/14-%E8%AF%BB%E5%86%99%E9%94%81%E6%BC%94%E5%8F%98.png)

#### 读写锁的降级

> 写锁->读锁

可以先写不释放锁 然后读

不可以先读不释放锁 然后写

![15-读写锁降级](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/15-%E8%AF%BB%E5%86%99%E9%94%81%E9%99%8D%E7%BA%A7.png)



### 阻塞队列BlockingQueue

![截屏2021-10-03 15.55.49](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/%E6%88%AA%E5%B1%8F2021-10-03%2015.55.49.png)

#### 分类

- ArrayBlockingQueue(常用) 有界
- LinkedBlockingQueue(常用) 有界

#### 核心方法

![截屏2021-10-03 16.00.44](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/%E6%88%AA%E5%B1%8F2021-10-03%2016.00.44.png)

#### 实现

```java
public class BlockingQueueDemo {
    public static void main(String[] args) throws InterruptedException {
        //创建阻塞队列
        BlockingQueue<String> blockingQueue = new ArrayBlockingQueue<>(3);

        //第一组
        System.out.println(blockingQueue.add("a"));
        System.out.println(blockingQueue.add("b"));
        System.out.println(blockingQueue.add("c"));
        System.out.println(blockingQueue.element());
        System.out.println(blockingQueue.add("x"));
        System.out.println(blockingQueue.remove());
        System.out.println(blockingQueue.remove());
        System.out.println(blockingQueue.remove());
        System.out.println(blockingQueue.remove());

        //第二组
        System.out.println(blockingQueue.offer("a"));
        System.out.println(blockingQueue.offer("b"));
        System.out.println(blockingQueue.offer("c"));
        System.out.println(blockingQueue.offer("x"));
        System.out.println(blockingQueue.poll());
        System.out.println(blockingQueue.poll());
        System.out.println(blockingQueue.poll());
        System.out.println(blockingQueue.poll());

        // 第三组
        blockingQueue.put("a");
        blockingQueue.put("b");
        blockingQueue.put("c");
        blockingQueue.put("x");
        System.out.println(blockingQueue.take());
        System.out.println(blockingQueue.take());
        System.out.println(blockingQueue.take());
        System.out.println(blockingQueue.take());
        //第四组
        System.out.println(blockingQueue.offer("a"));
        System.out.println(blockingQueue.offer("b"));
        System.out.println(blockingQueue.offer("c"));
        System.out.println(blockingQueue.offer("a", 3L, TimeUnit.SECONDS));
        //第一组
        System.out.println(blockingQueue.add("a"));
        System.out.println(blockingQueue.add("b"));
        System.out.println(blockingQueue.add("c"));
        System.out.println(blockingQueue.element());
    }
}
```

### 线程池ThreadPool

#### 使用

> 三种都是创建ThreadPoolExecutor

```java
//演示线程池的三种常用分类
public class ThreadPoolDemo {
    public static void main(String[] args) {
        //一池五线程
        ExecutorService threadPool1 = Executors.newFixedThreadPool(5);

        //一池一线程
        ExecutorService threadPool2 = Executors.newSingleThreadExecutor();

        //一池可扩容线程
        ExecutorService threadPool3 = Executors.newCachedThreadPool();

        //十个顾客请求
        try {
            for (int i = 0; i < 1000; i++) {
                //执行
                threadPool3.execute(() -> {
                    System.out.println(Thread.currentThread().getName() + " 办理业务");
                });
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            threadPool3.shutdown();
        }
    }
}

```

#### ThreadPoolExecutor

![截屏2021-10-03 18.01.50](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/%E6%88%AA%E5%B1%8F2021-10-03%2018.01.50.png)

![09-线程池七个参数](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/09-%E7%BA%BF%E7%A8%8B%E6%B1%A0%E4%B8%83%E4%B8%AA%E5%8F%82%E6%95%B0.png)

#### 线程池底层工作流程

![10-线程池底层工作流程](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/10-%E7%BA%BF%E7%A8%8B%E6%B1%A0%E5%BA%95%E5%B1%82%E5%B7%A5%E4%BD%9C%E6%B5%81%E7%A8%8B.png)

1. 在创建了线程池后，线程池中的线程数为零

2. 当调用execute()方法添加一个请求任务时，线程池会做出如下判断:

   2.1如果正在运行的线程数量小于 corePoolSize，那么马上创建线程运行这个任务; 

   2.2 如果正在运行的线程数量大于或等于 corePoolSize，那么将这个任务放入 队列; 

   2.3 如果这个时候队列满了且正在运行的线程数量还小于 maximumPoolSize，那么还是要创建非核心线程立刻运行这个任务; 

   2.4 如 果队列满了且正在运行的线程数量大于或等于 maximumPoolSize，那么线程 池会启动饱和拒绝策略来执行。

3. 当一个线程完成任务时，它会从队列中取下一个任务来执行

4. 当一个线程无事可做超过一定的时间(keepAliveTime)时，线程会判断:

   4.1 如果当前运行的线程数大于 corePoolSize，那么这个线程就被停掉。 

   4.2 所以线程池的所有任务完成后，它最终会收缩到 corePoolSize 的大小。



#### 拒绝策略

![截屏2021-10-03 18.12.18](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/%E6%88%AA%E5%B1%8F2021-10-03%2018.12.18.png)

#### 自定义线程池

```java
public class MyThreadPool {
    public static void main(String[] args) {
        ThreadPoolExecutor threadPool = new ThreadPoolExecutor(
                2,
                5,
                2L,
                TimeUnit.SECONDS,
                new ArrayBlockingQueue<>(3),
                Executors.defaultThreadFactory(),
                new ThreadPoolExecutor.AbortPolicy());

        //十个顾客请求
        try {
            for (int i = 0; i < 10; i++) {
                //执行
                threadPool.execute(() -> {
                    System.out.println(Thread.currentThread().getName() + " 办理业务");
                });
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            threadPool.shutdown();
        }
    }
}
```

- ### 线程池核心参数如何填写

  > n是服务器线程数

  - IO密集型：2n
  - cpu密集型：n+1

### Fork/Join分支合并框架

> Fork/Join 它可以将一个大的任务拆分成多个子任务进行并行处理，最后将子 任务结果合并成最后的计算结果，并进行输出。
>
> Fork:把一个复杂任务进行分拆，大事化小 
>
> Join:把分拆任务的结果进行合并

```java
class Mytask extends RecursiveTask<Integer> {
    //拆分差值不超过10,计算10以内的运算
    private static final Integer VALUE = 10;
    private int begin; //拆分开始值
    private int end; //拆分结束值
    private int result; //返回结果

    public Mytask(int begin, int end) {
        this.begin = begin;
        this.end = end;
    }

    @Override
    protected Integer compute() {
        if ((end - begin) < VALUE) {
            //相加
            for (int i = begin; i <= end; i++) {
                result += i;
            }
        } else {//进一步拆分
            int middle = (begin + end) / 2;
            Mytask taskLeft = new Mytask(begin, middle);
            Mytask taskRight = new Mytask(middle+1, end);

            taskLeft.fork();
            taskRight.fork();
            result = taskLeft.join()+taskRight.join();
        }
        return result;
    }
}

public class ForkJoinDemo {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        //创建MyTask对象
        Mytask mytask = new Mytask(0,100);
        //创建分支合并池对象
        ForkJoinPool forkJoinPool = new ForkJoinPool();
        ForkJoinTask<Integer> submit = forkJoinPool.submit(mytask);
        //获取结果
        Integer integer = submit.get();
        System.out.println(integer);
        //关闭池对象
        forkJoinPool.shutdown();
    }
}
```



### 异步回调CompletableFuture

> CompletableFuture 在 Java 里面被用于异步编程，异步通常意味着非阻塞， 可以使得我们的任务单独运行在与主线程分离的其他线程中，并且通过回调可 以在主线程中得到异步任务的执行状态，是否完成，和是否异常等信息。

简单实现

```java
public class CompletableFutureDemo {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        //异步调用 没有返回值
        CompletableFuture<Void> completableFuture1 = CompletableFuture.runAsync(()->{
            System.out.println(Thread.currentThread().getName()+"completableFuture1");

        });
        completableFuture1.get();

        //异步调用 有返回值
        CompletableFuture<Integer> completableFuture2 = CompletableFuture.supplyAsync(()->{
            System.out.println(Thread.currentThread().getName()+"completableFuture2");
            //模拟异常
            int a = 1/0;
            return 2;
        });
        Integer integer = completableFuture2.whenComplete((t,u)->{
            System.out.println(t); // 2：方法返回值
            System.out.println(u); // null：异常信息
        }).get();
        System.out.println(integer);
    }
}

```


