---
title: Java并发编程基础
date: 2024-12-17
slug: java-basic-concurrent
categories:
  - Java
description: Java并发编程的基础
draft: false
lastmod: 2025-02-12
---
##  1. 基础概念

### 1.1 进程、线程与虚拟线程

**进程**是程序的运行态，磁盘上静态的程序被 OS 加载进内存并分配所需的系统资源，包括如 CPU 时间、IO、堆栈和寄存器状态等一系列环境，这个过程便是在创建一个进程。

进程之间互相独立，拥有自己的地址空间，所以当一个进程崩溃时，通常对其他进程毫无影响。

进程是 OS 进行资源分配的最小单位，但其创建和销毁等需要较大的系统开销，若是系统中存在大量的进程，调度时它们之间的切换便会消耗大量时间。

---

**线程**是进程中更小的执行单元，同一进程内的线程共享堆和方法区，但拥有独立的程序计数器和栈。线程属于进程，一个进程通常包含多个线程，所有线程共享该进程的资源。

线程是 OS 最小的调度单位，同一进程的多个线程可同时在 CPU 的不同核心上运行。

由于多个线程共享同一进程的资源，所以同属一个进程的线程之间通信十分方便，但也因此容易互相影响，一个线程崩溃很容易导致其他线程崩溃。

相较于进程，线程的创建和销毁所造成的系统开销较小，调度时的上下文切换也更快。

Java 程序从 main() 方法启动后就会开启一个 JVM 进程，进程中可创建多个线程（例如通过 new Thread() 或线程池），而在现代 JVM 中，Java 线程一般映射为操作系统的内核线程（采用一对一模型）。

---

**虚拟线程**的正式版由 JDK21 正式发布，是更轻量级的线程实现，虚拟线程同一由 JVM 进行管理，而非 OS。

虚拟线程与 OS 线程并非一一对应，而是一个 OS 线程可能对应多个虚拟线程，并且这种对应关系并非绑定的，若是一个线程比较空闲，其他的虚拟线程可以将其作为载体进行工作，从而提高 CPU 利用率。

虚拟线程是用户态的，它的创建、销毁和调度等需要的系统开销相比于 OS 线程更少，因此虚拟线程可以大量存在却不占据大量系统资源，因此大大提高了系统的并发性。

---

**并发与并行**

- **并发**：多个任务在同一时间段内交替执行；
- **并行**：多个任务在同一时刻真正同时执行（多核环境下）。

---

**线程安全**：一个类或方法在多线程环境中，如果不需要额外同步就能保证数据的一致性，就称为线程安全；反之，则可能产生竞态条件和数据不一致问题。

---

**线程生命周期与上下文切换**

- 线程状态主要包括：NEW、RUNNABLE、BLOCKED、WAITING、TIMED_WAITING 和 TERMINATED。
- **上下文切换**指操作系统在不同线程间保存和恢复状态的过程，由于保存/恢复上下文需要 CPU 开销，频繁切换会降低效率。

## 创建线程

### 实现方式

`new Thread(){...};` 来创建一个新的线程，通过以下方法来分配任务

- 继承 `Thread` 并重写 `run()` 可分配任务给线程执行
- 实现 `Runnable` 接口，然后作为参数传入 `Thread()` 中，以此来分配任务
- 创建线程池，将任务投入线程池中

```java
// 实现 Runnable
public class MyRunnable implements Runnable {  
  
    @Override  
    public void run() {  
        System.out.println("MyRunnable is running");  
    }  
}

// 继承 Thread 
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

## 2. Java 内存模型（JMM）

1. 主要内容
	JMM 定义了线程之间共享变量的访问规则、内存可见性和有序性，并引入了 **happens-before** 规则来确保同步操作的正确性。
2. **happens-before** 规则
	程序顺序规则、锁定规则、volatile 规则、线程启动规则、线程终结规则等，确保在同步代码块中写入的内容对后续获取同一锁的线程可见。
3. 主内存与工作内存
	 所有共享变量存放在主内存中，线程各自有自己的工作内存，读写共享变量时必须进行主内存与工作内存之间的交互。

------

## 3. 并发控制机制

### 3.1 基于锁的并发控制

#### 3.1.1 synchronized
  - 隐式的对象锁，使用简单、语法简洁、自动释放锁，能保证互斥和可见性；
  - 缺点是功能较弱（如不能中断等待、无法精细控制锁的释放）且在竞争激烈时可能导致性能瓶颈。
  - 锁升级机制：
	1. **初始状态**：无锁/偏向锁状态
	2. **偏向锁**：
		- 当对象处于无竞争状态时，JVM会让第一个获取锁的线程将对象标记为“偏向”于它，此后该线程再次进入同步代码块无需额外CAS操作。
		- 如果其他线程尝试竞争这个锁，JVM首先会撤销偏向锁，将锁升级为轻量级锁。
	3. **轻量级锁**：
		- 偏向锁撤销后，JVM通过CAS机制在对象头中记录当前线程的锁记录，将锁升级为轻量级锁。
		- 适用于竞争不激烈时，通过自旋获取锁而避免线程阻塞。但如果竞争依然激烈（例如CAS反复失败），则会升级为重量级锁。
	4. **重量级锁**：
		- 当自旋竞争失败或等待时间过长时，轻量级锁会膨胀为重量级锁，此时采用操作系统互斥量来实现阻塞式同步，进入的线程会真正被挂起，直到锁被释放。
#### 3.1.2 Lock 接口及其实现
  - 最常用的实现，`ReentrantLock`：支持可重入（同一线程可多次获得同一锁）、中断响应（lockInterruptibly()）、尝试获取锁（tryLock()，可设置超时）以及公平/非公平两种策略；
  - `ReentrantReadWriteLock`：提供一对相关的锁，**读锁**允许多个线程同时读，但当有线程获得**写锁**时，读锁请求会被阻塞，此时写操作独占，可提高读多写少场景的并发性能；
#### 3.1.3 StampedLock
  - Java8 引入，相对于 ReentrantReadWriteLock 提供更细粒度的控制，它支持三种模式：写锁、悲观读锁以及乐观读锁，提高读操作效率（但使用上较为复杂）；
  - 虽然 StampedLock 并不直接实现 Lock 接口，但它提供了一套类似于 Lock 的 API，用于实现高效的读写控制和乐观锁定；
4. **两阶段锁协议（2PL）**
  - 事务在执行时先进入扩展阶段（不断申请锁）再进入收缩阶段（释放锁），确保调度与某个串行执行等价；
  - 严格2PL要求写锁直到事务结束才释放，以防止脏读。
#### 3.1.4 Lock 与 synchronized 的区别
1. **使用方式与灵活性**
    - **synchronized**
        - 是 Java 语言内置的关键字，使用简单（通过修饰方法或代码块实现同步），并且在异常发生时自动释放锁。
        - 受限于语言结构，不能指定等待时间、响应中断或创建多个条件变量。
    - **Lock 接口**
        - 是基于 API 的显式锁，使用时必须手动调用 lock() 获取锁，通常在 finally 块中调用 unlock() 释放锁。
        - 提供更多灵活特性：例如 tryLock()（带超时），lockInterruptibly()（可以被中断），以及可以通过 newCondition() 创建多个 Condition 以实现更细粒度的线程通信。
2. **中断响应**
    - synchronized 无法响应中断，即一旦进入同步代码块，线程除非执行完毕或抛出异常，否则不能被中断。
    - Lock 接口（如 ReentrantLock）的 lockInterruptibly() 方法可以让等待锁的线程响应中断，从而更灵活地处理阻塞情况。
3. **公平性**
    - synchronized 的锁由 JVM 管理，通常是非公平的（具体行为依赖于 JVM 实现）。
    - ReentrantLock 可以通过构造方法指定公平策略，使得等待时间较长的线程能优先获得锁。
4. **性能和扩展性**
    - 早期版本中 synchronized 的性能较低，但随着 JVM 的优化（如偏向锁、轻量级锁）在 JDK 6 以后，其性能已大为提升。
    - Lock 提供了更多扩展和控制能力，特别在高并发场景下，当需要复杂的锁调度和条件控制时，Lock（及其 Condition）可以提供更好的解决方案。

### 3.2 Synchronized 的使用
#### 3.2.1 同步实例方法
在类的实例方法上直接加上 **synchronized** 修饰符，相当于对当前对象（this）加锁，这样，同一时刻只有一个线程可以执行该方法，其他线程必须等待锁释放后才能进入。
```java
public class Counter {
    private int count = 0;

    // 同步实例方法，锁住的是当前对象实例
    public synchronized void increment() {
        count++;
    }

    public int getCount() {
        return count;
    }
}
```

> 在这种方式下，如果多个线程对同一个对象调用 `increment()` 方法，只有获得锁的线程能够执行，其他线程需要等待，从而确保对共享变量 `count` 的操作是线程安全的。

#### 3.2.2 同步静态方法
当使用 **synchronized** 修饰静态方法时，锁定的是该类的 **Class 对象**，这意味着对该类的所有实例而言，同一时刻只有一个线程可以执行该静态同步方法。

```java
public class SharedCounter {
    private static int count = 0;

    // 同步静态方法，锁住的是 SharedCounter.class
    public static synchronized void increment() {
        count++;
    }

    public static int getCount() {
        return count;
    }
}
```

> 这种方式确保了即使创建了多个对象实例，静态方法的调用也能保持线程安全。

#### 3.2.2 同步代码块
使用 **synchronized** 代码块可以更灵活地控制锁的范围和锁定对象，通过指定一个对象作为锁，只有获得该对象锁的线程才能进入同步代码块。

```java
public class Counter {
    private final Object lock = new Object();
    private int count = 0;

    public void increment() {
        // 使用自定义锁对象进行同步
        synchronized (lock) {
            count++;
            // 其他需要线程安全环境的代码
        }
    }
}
```

> 利用同步代码块可以避免将整个方法都标记为同步，从而提高并发性能，也提升了代码灵活性，让我们可以根据实际需要选择合适的锁对象，比如 this、某个类的 Class 对象或自定义的锁对象来控制同步粒度。

### 3.3 基于 CAS 和原子类

- **CAS 操作**
  - CAS（Compare-And-Swap）是一种无锁算法，利用硬件提供的原子操作指令实现对变量的更新；
  - 常见问题包括 ABA 问题，解决办法有 `AtomicStampedReference` 等。
- **java.util.concurrent.atomic 包**
  - 提供 `AtomicInteger`、`AtomicLong`、`AtomicReference` 等类，用于实现简单数据类型和引用的原子操作；
  - 当线程争用激烈时，CAS 的自旋可能导致较高的 CPU 消耗，这时可以考虑 LongAdder 等更适合高并发计数的方案。
------

## 4. 并发容器与工具类

### 4.1 并发集合

- ConcurrentHashMap
  - JDK 7 使用分段锁机制，JDK 8 采用 CAS + synchronized（数组+链表+红黑树）实现，线程安全且具有较高的并发性。
- CopyOnWriteArrayList / CopyOnWriteArraySet
  - 写操作时复制数组，读操作无锁，适合读多写少的场景；
- ConcurrentLinkedQueue
  - 基于无锁CAS设计的队列，适用于高并发场景下的非阻塞队列。

### 4.2 J.U.C 同步工具

- **CountDownLatch**：用于使一个线程等待一组其他线程完成任务。
- **CyclicBarrier**：允许一组线程互相等待直到到达公共屏障点，且可重复使用。
- **Semaphore**：通过维护许可的计数，控制对某一资源的并发访问。
- **Phaser**：更灵活的同步工具，支持动态调整参与者数量。

### 4.3 Future 和线程池

- **Future、Callable、FutureTask**
  - 用于提交可返回值的任务，支持任务取消、阻塞获取任务结果等。
- **Executor 和 ExecutorService**
  - 提供了基于线程池管理任务执行的框架，避免频繁创建销毁线程的开销。
- **ThreadPoolExecutor**
  - 核心线程池实现，包含核心线程数、最大线程数、任务队列、拒绝策略等参数；
- **ScheduledExecutorService**
  - 支持延迟执行和周期性任务调度。
- **Fork/Join 框架**
  - 利用分治算法并行执行任务，采用工作窃取策略，是利用多核 CPU 进行并行计算的有效工具。