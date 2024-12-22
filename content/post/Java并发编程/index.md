---
title: Java并发编程
date: 2024-12-17
slug: java-concurrent
categories:
  - 知识
  - Java
description: J.U.C 下的各类并发工具的学习
draft: true
lastmod: 2024-12-21
---
## 线程，进程，虚拟线程

### 进程

进程是程序的运行态，磁盘上静态的程序被 OS 加载进内存并分配所需的系统资源，包括如 CPU 时间、IO、堆栈和寄存器状态等一系列环境，这个过程便是在创建一个进程。

进程之间互相独立，拥有自己的地址空间，所以当一个进程崩溃时，通常对其他进程毫无影响。

进程是 OS 进行资源分配的最小单位，但其创建和销毁等需要较大的系统开销，若是系统中存在大量的进程，调度时它们之间的切换便会消耗大量时间。

### 线程

线程属于进程，一个进程通常包含多个线程，所有线程共享该进程的资源。

线程是 OS 最小的调度单位，同一进程的多个线程可同时在 CPU 的不同核心上运行。

由于多个线程共享同一进程的资源，所以同属一个进程的线程之间通信十分方便，但也因此容易互相影响，一个线程崩溃很容易导致其他线程崩溃。

相较于进程，线程的创建和销毁所造成的系统开销较小，调度时的上下文切换也更快。

### 虚拟线程

虚拟线程的正式版由 JDK21 正式发布，是更轻量级的线程实现，虚拟线程同一由 JVM 进行管理，而非 OS。

虚拟线程与 OS 线程并非一一对应，而是一个 OS 线程可能对应多个虚拟线程，并且这种对应关系并非绑定的，若是一个线程比较空闲，其他的虚拟线程可以将其作为载体进行工作，从而提高 CPU 利用率。

虚拟线程是用户态的，它的创建、销毁和调度等需要的系统开销相比于 OS 线程更少，因此虚拟线程可以大量存在却不占据大量系统资源，因此大大提高了系统的并发性。

### 区别于联系

区别：

- 资源占用：进程 > 线程 > 虚拟线程
- 运行环境：进程与线程运行在内核态，由 OS 管理，虚拟线程运行在用户态，由 JVM 管理

联系：

- 所有线程都属于某一进程，而虚拟线程需要以某一线程为载体进行工作
- 虚拟线程类似于用户态线程，在使用方式上与线程类似

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

而在某些场景下， 有时候线程用完即弃， 并不考虑复用，我们可以使用匿名类和 Lambda 表达式来更方便地创建线程，示例如下：

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