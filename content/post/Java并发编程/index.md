---
title: Java 并发编程
date: 2024-12-17
categories:
  - 知识
  - Java
description: J.U.C 下的各类并发工具的学习
draft: true
tags:
  - Java
  - 并发
---
## 什么是线程


## 创建线程

### 两种实现方式

两种基础方式如下：

```java
// 通过实现Runnable接口创建线程
public class MyRunnable implements Runnable {  
  
    @Override  
    public void run() {  
        System.out.println("MyRunnable is running");  
    }  
}

// 通过继承Thread类创建线程  
public class MyThread extends Thread {  
  
    @Override  
    public void run() {  
        System.out.println("MyThread is running");  
    }  
}
```

在主线程尝试运行

```java
public class App {  
    public static void main(String[] args) throws Exception {  
        MyThread t1 = new MyThread();  
        t1.start();  
  
        Thread thread = new Thread(new MyRunnable());  
        thread.start();  
  
        System.out.println("主线程执行中...");  
    }  
}

// 输出：
// MyThread is running
// 主线程执行中...
// MyRunnable is running
```

可以看见三个线程的执行顺序并不相同

而在某些场景下，有时候线程用完即弃，并不考虑复用，我们可以使用匿名类和 Lambda 表达式来更方便地创建线程，示例如下：

**匿名**

```java
new Thread() {  
    @Override  
    public void run() {  
        System.out.println("匿名线程1启动...");  
    }  
}.start();  
  
new Thread(new Runnable() {  
    @Override  
    public void run() {  
        System.out.println("匿名线程2启动...");  
    }  
}).start();
```

**Lambda 表达式**

```java
new Thread(() -> System.out.println("匿名线程1启动...")).start();
  
new Thread(() -> System.out.println("匿名线程2启动...")).start();
```

此时两个线程直接传入的就是“做什么”，这样代码更加简洁明了

### 区别与联系

#### 区别

1. 继承的方式：
	- 类本身变成线程的一部分，与线程紧密耦合，扩张性和灵活性都受到极大的限制
	- 每创建一个类的实例，就是创建一个线程，当大量实例被创建时，将极大地占用系统资源
1. 实现接口的方式：
	- 类封装的是线程的任务，可将类实例传给其他线程或线程池，可提高线程复用率的同时，让任务的执行更加灵活
	- 抽象层面上，这种方式更好地遵循了面向对象的设计原则，使得代码更加模块化和易于维护

#### 联系
1. 二者目的相同，都是为了让线程完成某一任务
2. 两种方法创建的线程无本质区别，都遵循相同的启动流程和生命周期

### 为什么不能直接 run()

因为 `run()` 是同步方法，直接调用该方法，会是当前线程去执行任务，需要等待代码执行到调用 `run()` 的时候，当前线程才真正地执行 `run()` 里定义的任务
而 `Thread.start()` 后，线程进入可执行状态，一旦获得 CPU 时间片，将立即执行 `run()` 中的代码，并且`start()` 只能被执行一次