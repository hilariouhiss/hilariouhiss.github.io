---
title: JUnit5 与 Mockito 基础
date: 2025-03-03
slug: junit5-mockito-basic-integration
categories:
  - Java
description: 本文通过示例演示了JUnit5 与 Mockito常用测试注解的基础方法。
draft: false
lastmod: 2025-03-14
---

## 准备工作

Maven项目引入相关依赖：

```xml
<!--Mockito与JUnit5集成-->
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-junit-jupiter</artifactId>
    <version>5.14.2</version>
    <scope>test</scope>
</dependency>
<!--JUnit5-->
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter</artifactId>
    <version>5.11.4</version>
    <scope>test</scope>
</dependency>
```

受测试的类：

```java
public class Calculator {
    public int add(int a, int b) {
        return a + b;
    };
}
```

## 开始测试

测试类：

```java
public class CalculatorTest {

    @Test
    void testAdd() {
        // 获取一个模拟的 Random 类
        Random random = Mockito.mock(Random.class);
        // Random 进行了一个行为
        System.out.println(random.nextInt());
        // 校验 Random 对象中方法的调用情况，调用的次数，参数是什么
        Mockito.verify(random, Mockito.times(1)).nextInt();
        // 当不对 mock 对象的行为进行定义（打桩）时，mock 对象方法的返回值为该返回类型的默认值
        // 所以random.nextInt() 一直返回 0
    }
}
```

### @Mock 注解

```java
public class CalculatorTest {

    @Mock
    private Random random;

    @Test
    void testAdd() {
        // 使@Mock生效
        MockitoAnnotations.openMocks(this);
        // 打桩
        Mockito.when(random.nextInt()).thenReturn(100);
        Assertions.assertEquals(100, random.nextInt());
        // 通过
    }
}
```

### @Spy 注解

调用真实的实例方法，而不是Mock出来的

```java
public class CalculatorTest {

    @Spy
    private Calculator calculator;

    @BeforeEach
    void setUp(){
        // 使@Mock生效
        MockitoAnnotations.openMocks(this);
    }

    @Test
    void testAdd() {
        // Mockito.when(calculator.add(1, 2)).thenReturn(1);
        // 不打桩，走真实的方法
        Assertions.assertEquals(3, calculator.add(1,2));
        // 通过
    }
}
```

### 补充注解

- **@BeforeEach** ：会在每个测试方法执行前运行，适用于需要为每个测试单独初始化环境的场景（例如重置数据、创建新对象等），确保测试间彼此独立。
- **@BeforeAll**：只在整个测试类中第一次测试前运行一次。适用于那些只需要一次性执行的初始化工作，如建立数据库连接、加载共享资源等。在JUnit 5中，除非使用 PER_CLASS 测试实例生命周期，否则 **@BeforeAll** 方法必须是静态的。
- **@AfterEach**：在每个测试方法执行完后运行，通常用来清理每个测试产生的状态或释放资源。
-  **@AfterAll**：在整个测试类中所有测试方法都执行完毕后运行，用于做全局性的清理工作（例如关闭共享的数据库连接或释放其他全局资源）。在JUnit 5中，除非设置测试实例为 PER_CLASS，否则 **@AfterAll** 方法必须是静态的。

