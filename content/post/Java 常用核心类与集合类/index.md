---
title: Java 常用核心类与集合类
date: 2024-12-24
slug: java-core-and-collections
categories:
  - Java
description: Java 常用的核心类和集合类的功能与用法。
draft: false
lastmod: 2025-03-14
---

## 常用核心类

1. **`java.lang.Object`**：所有类的根类，提供了一些基本方法如 `equals()`, `hashCode()`, `toString()`, `clone()` 等。
2. **`java.lang.String`**：用于表示字符串，不可变（immutable）。
3. **`java.lang.StringBuilder` 和 `java.lang.StringBuffer`**：用于构建和操作字符串，`StringBuilder` 是非线程安全但性能更高，`StringBuffer` 是线程安全但相对较慢。
4. **`java.lang.Math`**：提供了数学相关的静态方法和常量，如 `abs()`, `sqrt()`, `pow()`, `PI` 等。
5. **`java.lang.System`**：包含了一系列与系统有关的方法和属性，如 `System.out`、`System.err`、`System.in` 以及获取当前时间的方法 `currentTimeMillis()`。
6. **`java.lang.Thread`**：用于创建和管理线程，实现多线程编程。
7. **`java.lang.Runnable`**：一个接口，通常与 `Thread` 一起使用来定义线程执行的任务。
8. **`java.util.Date` 和 `java.time.*` (Java 8 及以上)**：日期和时间处理类，`java.time` 包引入了新的 API 来处理日期和时间，如 `LocalDate`, `LocalTime`, `LocalDateTime` 等。
9. **`java.io.*`**：输入输出流相关类，如 `FileInputStream`, `FileOutputStream`, `BufferedReader`, `BufferedWriter` 等。
10. **`java.nio.*`**：非阻塞 I/O 操作的支持，包括文件通道、字符集等。
11. **`java.net.*`**：网络编程支持，如 `Socket`, `ServerSocket`, `HttpURLConnection` 等。

## 常用集合类

Java 集合框架位于 `java.util` 包中，主要包括以下几类：

1. **List 接口及其实现**：
    
    - `ArrayList`：基于数组实现，允许重复元素，随机访问快，增删效率低。
    - `LinkedList`：基于链表实现，允许重复元素，适合频繁插入和删除操作。
    - `Vector`：类似于 `ArrayList`，但是线程安全，性能较低。
    - `Stack`：继承自 `Vector`，实现了后进先出（LIFO）的数据结构。
2. **Set 接口及其实现**：
    
    - `HashSet`：基于哈希表实现，不允许重复元素，不保证元素顺序。
    - `LinkedHashSet`：保持插入顺序的 `HashSet`。
    - `TreeSet`：基于红黑树实现，自动排序，不允许重复元素。
3. **Map 接口及其实现**：
    
    - `HashMap`：基于哈希表实现，键值对存储，不允许键重复，值可以重复，不保证映射的顺序。
    - `LinkedHashMap`：保持插入顺序的 `HashMap`。
    - `TreeMap`：基于红黑树实现，按键排序，不允许键重复。
    - `Hashtable`：类似于 `HashMap`，但是线程安全，不允许键和值为 `null`。
4. **Queue 接口及其实现**：
    
    - `LinkedList`：除了作为 `List` 的实现外，还可以作为双端队列使用。
    - `PriorityQueue`：根据优先级排序的队列。
    - `Deque`（双端队列）：`ArrayDeque` 和 `LinkedList` 实现，支持在两端添加或移除元素。
5. **其他集合类**：
    
    - `EnumSet`：专门为枚举类型设计的集合。
    - `EnumMap`：专门为枚举类型设计的映射。
    - `ConcurrentHashMap`：线程安全的 `HashMap` 替代品，在高并发环境下性能较好。
    - `CopyOnWriteArrayList`：线程安全的 `List` 实现，适用于读多写少的场景。