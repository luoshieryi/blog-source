---
title: Java 多线程
date: 2021-10-13
tags: [java, thread]
---

# 多线程

## 简介

- 多线程实际上并不是同时执行, 多个任务在极短时间内交替执行, 由于 CPU 运算速度很快, 所以看起来像是同时执行

线程与进程: 

- 在计算机中的一个任务称为一个进程, 一个进程包含一至多个线程
- 运行程序时, 可以使用: 多进程+单线程, 单进程+多线程, 多进程+多线程
- 进程的特点: 创建时的开销更大, 不同进程间数据通信慢, 稳定性更高(一个进程奔溃不影响其他进程运行)

多线程: 一个 Java 程序其实是一个 JVM 进程, JVM 进程中有一个 main 主线程, 我们在 main 中执行各种方法, 启动多个线程

## 创建一个新线程

创建一个新线程, 需要实例化一个`Thread`实例, 调用它的`start()`方法, 之后`start()`方法会自动调用这个实例的`run()`方法, 启动线程

三种创建线程的方法: `implements Runnable` , `extends Thread` , 使用Callable和Future

- 区别: 实现 Runnable 与 Callable 接口后可以继承其他类, 继承 Thread 类在一些操作上更简单

`extends Thread`示例: 

```java
class horseRun extends Thread {
  
  public static final int LENGTH = 188;
  
  char c;
  
  horseRun(char c) {
        this.c = c;
  }
  
  @Override
    public void run() {
        for (int i = 0; i < LENGTH; i ++) {
            System.out.print(c);
        }
    }
}

public class Main {

    public static void main(String[] args) {

        Thread horse1 = new horseRun('%');
        Thread horse2 = new horseRun('$');
        horse1.start();
        horse2.start();

    }
}
```

输出: `$$$$$$$$$$$$$$$$$$$$%%%%%%%%%%%%%%%%%%%%%%%%%%%%$$$$$$$$$$$$$$$$$$%%%%%%%%%%%%%%%%%%$$$$$$$$$$$$%%%%`

- 每次的结果都不一样, 两个线程同时开始运行, 由**操作系统进行调度, 程序无法决定**

尝试修改进程的优先级: `horse1.setPriority(4); horse2.setPriority(6);`
输出结果: `$$$$$%%%$$$$$$$%%%%%%$$$$$$$$%%%%$$$$$$$%%%$$$$$$$$$$$$$$$$$$$$$$$%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%`

- 操作系统可能对高优先级线程调度更频繁, 但是并不能保证线程一定会先执行