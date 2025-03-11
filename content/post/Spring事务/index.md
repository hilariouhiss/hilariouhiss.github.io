---
title: Spring 事务总结
date: 2025-02-12
slug: spring-transaction
categories:
  - Spring
description: Spring 事务相关知识的总结
draft: false
lastmod: 2025-03-11
---
## 事务基本概念与 ACID 特性

事务是一组逻辑上不可分割的操作，要么全部执行成功，要么全部失败回滚。事务的核心目的是保证数据的一致性、完整性和可靠性，其本身具有四大特性，通常被称为 ACID：

- **原子性（Atomicity）**  
    事务内所有操作要么全部成功，要么全部失败，不会出现部分成功的情况。
- **一致性（Consistency）**  
    事务执行前后，数据库必须保持一致状态。例如在转账操作中，转出账户与转入账户金额之和在事务前后应保持不变。
- **隔离性（Isolation）**  
    并发执行的事务相互之间不应产生干扰，不同事务在操作相同数据时，应保证彼此独立。
- **持久性（Durability）**  
    一旦事务提交，其对数据库的修改即使遇到系统故障也能持久保存。

## Spring 中的事务管理方式

Spring 提供了两种事务管理方式：

### 编程式事务管理

通过代码显式调用事务管理器（例如使用 `TransactionTemplate` 或直接调用 `PlatformTransactionManager` 的 `getTransaction()`, `commit()` 和 `rollback()` 方法）来管理事务。

```java
@Autowired
private PlatformTransactionManager transactionManager;

public void doTransaction() {
    TransactionStatus status = transactionManager.getTransaction(new DefaultTransactionDefinition());
    try {
        // 实际操作
        transactionManager.commit(status);
    } catch (Exception e) {
        transactionManager.rollback(status);
    }
}

```

这种方式灵活但代码侵入性较大，通常仅用于特殊需求或底层封装。

### 声明式事务管理

使用 `@Transactional` 注解或 XML 配置的方式，由 Spring AOP 自动为方法创建事务代理。声明式事务的优点在于代码侵入性低，能以最小代价为业务方法添加事务支持，且事务的属性（传播行为、隔离级别、只读、超时、回滚规则）均可通过注解参数配置。

```java
@Service
public class AccountService {

    @Transactional(propagation = Propagation.REQUIRED,
                   isolation = Isolation.DEFAULT,
                   timeout = 30,
                   readOnly = false,
                   rollbackFor = Exception.class)
    public void transferMoney(String from, String to, double amount) {
        // 执行转账操作：扣款与加款必须在同一事务中完成
    }
}

```

声明式事务管理的实现基于 AOP 和动态代理，在方法调用前后自动开启或结束事务

## Spring 事务的核心接口

Spring 事务管理中有三个重要接口：

- **PlatformTransactionManager**  
    这是 Spring 的事务管理器接口，定义了获取、提交和回滚事务的方法。Spring 针对不同数据访问技术提供了不同的实现，例如 `DataSourceTransactionManager`（JDBC）、`HibernateTransactionManager`、`JpaTransactionManager` 等。
    
- **TransactionDefinition**  
    用于描述事务的属性，包括传播行为、隔离级别、超时时间、是否只读以及事务名称等。Spring 定义了一系列常量（如 `PROPAGATION_REQUIRED`、`ISOLATION_REPEATABLE_READ` 等）来便于配置。
    
- **TransactionStatus**  
    表示当前事务的状态，允许查询事务是否为新事务、是否已标记回滚等。通过此接口也可以在代码中主动设置回滚状态。
    

这三个接口共同构成了 Spring 事务管理的底层支撑。


## 事务属性详解

### 事务传播行为（Propagation）

事务传播行为决定了在一个事务方法内部调用另一个事务方法时，如何处理事务边界。

- **REQUIRED**：存在事务则加入，否则新建。默认
- **REQUIRES_NEW**：总是新建事务，挂起当前事务
- **NESTED**：嵌套事务。嵌套事务可以单独回滚，但最终仍依赖于外部事务的提交与否。并且依赖于数据库是否支持
- **SUPPORTS**：有事务则加入，无则以非事务运行
- **NOT_SUPPORTED**：总是以非事务运行，有则挂起
- **NEVER**：必须以非事务运行，有则抛异常
- **MANDATORY**：必须以事务运行，无则抛异常

### 隔离级别（Isolation）

隔离级别用于控制并发事务之间的数据可见性，常见的隔离级别包括：

- **DEFAULT**：使用底层数据库默认的隔离级别。
- **ISOLATION_READ_UNCOMMITTED**：最低隔离级别，允许脏读、不可重复读和幻读。
- **ISOLATION_READ_COMMITTED**：禁止脏读，但仍可能发生不可重复读或幻读（Oracle 默认隔离级别）。
- **ISOLATION_REPEATABLE_READ** ：防止脏读和不可重复读，但可能发生幻读（MySQL 默认隔离级别）。
- **ISOLATION_SERIALIZABLE**：最高隔离级别，通过串行化事务执行完全避免并发问题，但性能开销较大。

### 只读属性（readOnly）与超时（timeout）

- **readOnly**  
对于只执行查询操作的方法，可以将事务标记为只读，底层数据库可以利用此提示进行优化。但对于写操作必须设为 `false`。
- **timeout**  
指定事务执行的最长时间，若超时则自动回滚。这可以防止长事务占用数据库资源导致锁等待等问题。

### 回滚规则

默认情况下，Spring 事务只在遇到运行时异常（`RuntimeException` 及其子类）和错误（`Error`）时回滚，而对于检查型异常则不会自动回滚。如果需要对特定异常进行回滚，可通过 `rollbackFor` 属性指定。

例如：`@Transactional(rollbackFor = { Exception.class })`

这保证了在捕获任何异常类型时都能触发回滚。

## 声明式事务的实现原理及常见坑

### 实现原理

Spring 声明式事务基于 AOP 技术实现。框架在启动时扫描带有 `@Transactional` 注解的方法，并为这些方法生成代理对象。在代理方法执行前，事务拦截器会根据注解属性决定是否创建新事务或加入现有事务；方法执行结束后，根据执行结果提交或回滚事务。整个过程对业务代码透明无侵入。

### 常见问题

- **方法必须为 public**  
只有 public 方法才能被 Spring AOP 代理拦截，如果将 `@Transactional` 应用在非 public 方法上将不起作用。    
- **自调用问题**  
同一类中内部方法调用不会经过代理，从而导致事务注解失效。解决办法是将事务方法放到单独的 Bean 中或使用 AspectJ 实现事务增强。    
- **Bean 必须被 Spring 管理**  
只有由 Spring 容器管理的 Bean 才能应用事务代理。    
- **数据库引擎支持**  
数据库（例如 MySQL 的 InnoDB）必须支持事务，否则无论如何配置事务都不会生效。    
- **异常被捕获**  
如果在事务方法内部捕获异常而不重新抛出，则事务管理器无法感知异常，可能导致事务错误提交。    