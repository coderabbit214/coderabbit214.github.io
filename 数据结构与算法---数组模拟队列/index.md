# 数组模拟队列


# 数组模拟队列

## 原理

1. 队列是一个有序列表，可以用数组或链表来实现
2. 遵循先入先出的原则。即：先存入队列的数据要先去除。后存入的后取出。

## 实现简单队列

[![tnWeL8.th.png](https://s1.ax1x.com/2020/05/29/tnWeL8.th.png)](https://imgchr.com/i/tnWeL8)

```java
package com.jsh.DataStructures;

import java.util.Scanner;

/*
    Mr.J
    数组队列
    2020/05/29
    问题：数组使用一次就不能使用，没有达到复用的效果 需要改为环形队列
 */
public class ArrayQueueDemo {
    public static void main(String[] args) {
        ArrayQueue arrayQueue = new ArrayQueue(3);
        char key = ' ';//接收用户输入
        Scanner scanner = new Scanner(System.in);
        boolean loop = true;
        while (loop) {
            System.out.println("s(show):显示队列");
            System.out.println("e(exit):退出队列");
            System.out.println("a(add):添加数据");
            System.out.println("g(get):从队列读取数据");
            System.out.println("h(head):查看队列头的数据");
            key = scanner.next().charAt(0);
            switch (key) {
                case 's':
                    arrayQueue.showQueue();
                    break;
                case 'e':
                    scanner.close();
                    loop=false;
                    break;
                case 'a':
                    System.out.println("请输入一个数");
                    int value = scanner.nextInt();
                    arrayQueue.addQueue(value);
                    break;
                case 'g':
                    try {
                        int res = arrayQueue.getQueue();
                        System.out.println("取出的数据为：" + res);
                    } catch (Exception e) {
                        System.out.println(e.getMessage());
                    }
                    break;
                case 'h':
                    try {
                        int res = arrayQueue.headQueue();
                        System.out.println("取出的数据为：" + res);
                    } catch (Exception e) {
                        System.out.println(e.getMessage());
                    }
                    break;
            }
        }
        System.out.println("程序退出");
    }
}

//使用数组模拟队列-编写一个ArrayQueue类
class ArrayQueue {
    private int maxSize;//表示数组的最大容器
    private int front;//队列头
    private int rear;//队列尾
    private int[] arr;//该数组用于存放数据，模拟队列

    //  创建队列的构造器
    public ArrayQueue(int maxSize) {
        this.maxSize = maxSize;
        arr = new int[maxSize];
        front = -1;//队列头的前一个位置
        rear = -1;//队列尾
    }

    //  判断队列是不是满
    public boolean ifFull() {
        return rear == maxSize - 1;
    }

    //  判断队列是否为空
    public boolean isEmpty() {
        return front == rear;
    }

    //  添加数据到队列
    public void addQueue(int n) {
        if (ifFull()) {
            System.out.println("队列满，不能加入数据");
            return;
        }
        rear++; //让rear后裔
        arr[rear] = n;
    }

    //  数据出队列
    public int getQueue() {
        if (isEmpty()) {
            throw new RuntimeException("队列空，不能取数据");
        }
        front++;
        return arr[front];
    }

    //  显示队列的所有数据
    public void showQueue() {
        if (isEmpty()) {
            System.out.println("队列空");
            return;
        }
        for (int a : arr) {
            System.out.println(a);
        }
    }

    //  显示队列的头数据
    public int headQueue() {
        if (isEmpty()) {
            throw new RuntimeException("队列空");
        }
        return arr[front + 1];
    }
}
```

## 循环队列

```java
package com.jsh.DataStructures;

import java.util.Scanner;

/*
    Mr.J
    数组队列
    2020/05/29
    问题：数组使用一次就不能使用，没有达到复用的效果 需要改为环形队列
 */
public class CircleArrayQueueDemo {
    public static void main(String[] args) {
        CircleArrayQueue arrayQueue = new CircleArrayQueue(3);
        char key = ' ';//接收用户输入
        Scanner scanner = new Scanner(System.in);
        boolean loop = true;
        while (loop) {
            System.out.println("s(show):显示队列");
            System.out.println("e(exit):退出队列");
            System.out.println("a(add):添加数据");
            System.out.println("g(get):从队列读取数据");
            System.out.println("h(head):查看队列头的数据");
            key = scanner.next().charAt(0);
            switch (key) {
                case 's':
                    arrayQueue.showQueue();
                    break;
                case 'e':
                    scanner.close();
                    loop=false;
                    break;
                case 'a':
                    System.out.println("请输入一个数");
                    int value = scanner.nextInt();
                    arrayQueue.addQueue(value);
                    break;
                case 'g':
                    try {
                        int res = arrayQueue.getQueue();
                        System.out.println("取出的数据为：" + res);
                    } catch (Exception e) {
                        System.out.println(e.getMessage());
                    }
                    break;
                case 'h':
                    try {
                        int res = arrayQueue.headQueue();
                        System.out.println("取出的数据为：" + res);
                    } catch (Exception e) {
                        System.out.println(e.getMessage());
                    }
                    break;
            }
        }
        System.out.println("程序退出");
    }
}

//使用数组模拟队列-编写一个ArrayQueue类
class CircleArrayQueue {
    private int maxSize;//表示数组的最大容器
    private int front;//队列头 front 的初始值 = 0 front 变量的含义做一个调整： front 就指向队列的第一个元素, 也就是说 arr[front] 就是队列的第一个元素
    private int rear;//队列尾 rear 变量的含义做一个调整：rear 指向队列的最后一个元素的后一个位置. 因为希望空出一个空间做为约定.rear 的初始值 = 0
    private int[] arr;//该数组用于存放数据，模拟队列

    //  创建队列的构造器
    public CircleArrayQueue(int maxSize) {
        this.maxSize = maxSize;
        arr = new int[maxSize];
//        初始化时为零 可以不写
//        front = 0;//队列头
//        rear = 0;//队列尾
    }

    //  判断队列是不是满
    public boolean ifFull() {
        return (rear + 1) % maxSize == front;
    }

    //  判断队列是否为空
    public boolean isEmpty() {
        return front == rear;
    }

    //  添加数据到队列
    public void addQueue(int n) {
        if (ifFull()) {
            System.out.println("队列满，不能加入数据");
            return;
        }
//      直接将数据加入
        arr[rear] = n;
//      rear后移
        rear = (rear+1) % maxSize;
    }

    //  数据出队列
    public int getQueue() {
        if (isEmpty()) {
            throw new RuntimeException("队列空，不能取数据");
        }
        int value = arr[front];
        front = (front+1) % maxSize;
        return value;
    }

    //  显示队列的所有数据
    public void showQueue() {
        if (isEmpty()) {
            System.out.println("队列空");
            return;
        }
        for (int i = front;i<front+size();i++) {
            System.out.println(arr[i%maxSize]);
        }
    }
//  求出当前队列有效数据的个数
    public int size(){
        return (rear + maxSize -front) % maxSize;
    }

    //  显示队列的头数据
    public int headQueue() {
        if (isEmpty()) {
            throw new RuntimeException("队列空");
        }
        return arr[front];
    }
}
```


