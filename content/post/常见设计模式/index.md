---
title: 常见设计模式
date: 2025-03-11
slug: common-design-patterns
categories:
  - Java
description: Java 常见的 23 种设计模式介绍与示例
draft: true
lastmod: 2025-03-12
---
## 面向对象设计原则

1. ***单一职责原则***：一个类仅承担一个职责，且引起其变化的唯一原因应与其核心功能直接相关。例如，用户管理类应专注于用户增删改查，避免混杂权限校验逻辑。
2. ***开闭原则***：软件实体应对扩展开放，对修改关闭。通过抽象化和多态机制（如接口或继承），允许新增功能时无需修改现有代码。例如，新增支付方式时通过实现支付接口扩展，而非修改原有支付类。
3. ***里氏替换原则***：子类必须能够完全替代父类，且不影响程序正确性。例如，`List<String> list = new ArrayList<>()`中，`ArrayList`作为`List`的子类需保证所有父类方法的行为一致性。
4. ***接口隔离原则***：客户端不应被迫依赖其不需要的接口。通过细化接口功能，避免“臃肿接口”。例如，将`Animal`接口拆分为`Flyable`和`Swimmable`，而非让所有动物类实现无关方法。
5. ***依赖倒置原则***​：高层模块与低层模块应通过抽象交互，而非直接依赖具体实现。例如，数据库操作应依赖抽象的数据访问接口（如`Repository`），而非具体的MySQL或Oracle实现类。

额外：
- ***迪米特法则***：对象间应保持最小知识范围，减少耦合。例如，订单类不直接访问库存数据库，而是通过中间服务类代理。
- ***合成/聚合复用原则***：优先通过组合（has-a）而非继承（is-a）实现代码复用。例如，汽车类包含引擎对象（组合），而非继承引擎类，避免继承带来的强耦合。

## 常见的23种设计模式
### 创建型
#### 单例模式

- **核心作用**
    确保一个类只有一个实例，并提供全局访问点。
- **应用场景**
    配置管理、线程池、日志记录器等需要全局唯一资源的场景。
- **常见实现**
1. 饿汉式
```java
public class SingletonHungry{
	private static final SingletonHungry INSTANCE = new SingletonHungry();
	private SingletonHungry(){}
	public static SingletonHungry getInstance(){
		return INSTANCE; 
	}
}
```
类加载的同时创建对应实例，线程安全，但可能浪费系统资源。

2. 双重检查锁（懒汉式）
```java
public class SingletonLazy {
    private static volatile SingletonLazy instance;
    // volatile 防止指令重排，避免半初始化
    private SingletonLazy() {}
    public static SingletonLazy getInstance() {
        if(instance == null) {
            // 缩小加锁范围，减少锁竞争
            synchronized(SingletonLazy.class) {
                // 确保当前无实例存在
                if(instance == null) {
                    instance = new SingletonLazy();
                }
            }
        }
        return instance;
    }
}
```

3. 静态内部类
```java
public class SingletonInner{
    private SingletonInner(){}
    private static class Holder{
        private static final SingletonInner INSTANCE = new SingletonInner();

    }
    public static SingletonInner getInstance(){
        return Holder.INSTANCE;
    }
}
```
静态内部类在首次调用时加载，由JVM保证线程安全

4. 枚举类
```java
public enum Singleton{
    INSTANCE;
    // doSomething
}
```
JVM保证唯一性，并且防止序列化和反序列化攻击

#### 工厂模式

- **核心作用**  
    定义一个创建对象的接口，由子类决定具体要实例化的类，将对象的实例化延迟到子类。
- **应用场景**  
    当一个类无法预见所需创建对象的类型，或者希望在子类中灵活扩展产品时。
- **常见实现**
1. 简单工厂
```java
// 产品接口
interface Product {
    void use();
}

// 具体产品A
class ProductA implements Product {
    public void use() {
        System.out.println("使用产品A");
    }
}

// 具体产品B
class ProductB implements Product {
    public void use() {
        System.out.println("使用产品B");
    }
}

// 工厂抽象类
abstract class Creator {
    public abstract Product factoryMethod();
    
    public void doSomething() {
        Product product = factoryMethod();
        product.use();
    }
}

// 具体工厂A
class CreatorA extends Creator {
    public Product factoryMethod() {
        return new ProductA();
    }
}

// 具体工厂B
class CreatorB extends Creator {
    public Product factoryMethod() {
        return new ProductB();
    }
}

public class FactoryMethodDemo1 {
    public static void main(String[] args) {
        Creator creator = new CreatorA();
        creator.doSomething();
        creator = new CreatorB();
        creator.doSomething();
    }
}
```

2. 匿名内部类创建
```java
interface Product2 {
    void use();
}

public class FactoryMethodDemo2 {
    // 工厂方法，参数决定创建哪种产品
    public static Product2 createProduct(String type) {
        if ("A".equalsIgnoreCase(type)) {
            return new Product2() {
                public void use() {
                    System.out.println("使用匿名实现的产品A");
                }
            };
        } else if ("B".equalsIgnoreCase(type)) {
            return new Product2() {
                public void use() {
                    System.out.println("使用匿名实现的产品B");
                }
            };
        }
        throw new IllegalArgumentException("未知产品类型");
    }
    
    public static void main(String[] args) {
        Product2 product = createProduct("A");
        product.use();
        product = createProduct("B");
        product.use();
    }
}
```