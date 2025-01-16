---
title: Java 开发笔记
date: 2025-01-13
slug: dev-notes
categories:
  - Java
description: 开发项目时学习到的小知识点
draft: true
lastmod: 2025-01-13
---
## 使用 Stream 处理列表中每一个元素

首次编码：

```java
Optional<List<User>> users = userMapper.selectAll();
return users.stream().flatMap(List::stream)
		.map(UserDTO::new).collect(Collectors.toList());
```

发现：`Optional` 本身就可直接 `.map()`，无需先转为**流**，可去除 `stream()`

再思考：`flatMap(List::stream)` 是先将 `Optional<List<User>>` 展开为一个 `User` 对象流，再对流中每个 `User` 对象应用 `map()`

改进：在查询时先判空，得到 `List<User>`，再直接使用 `.stream().map()` 处理每个元素，得到如下代码：

```java
public List<UserDTO> selectAll() {  
    log.info("开始查询所有用户");  
    List<User> userList = userMapper.selectAll().orElseGet(List::of);  
    return userList.stream().map(UserDTO::new).collect(Collectors.toList());  
}
```

AI 建议：将 `List::of` 换为 `Collections::emptyList`
理由：因为 `emptyList()` 返回的是一个不可变的空列表的共享实例，对于不需要修改的集合来说，这是更高效的选择。

## `flatMap()` 和 `map()` 的区别

都是用于处理集合或流的元素，二者的关键区别是处理结果的**结构层次不同**

### `map()`

`<R> Stream<R> map(Function<? super T, ? extends R> mapper);`

接收一个`Function` 类型的函数式接口，返回一个任意类型的 `Stream`

功能：将流中的每个元素一对一地映射为另一个元素，即使存在流的嵌套，处理前后流的结构也不变

### `flatMap()`

`<R> Stream<R> flatMap(Function<? super T, ? extends Stream<? extends R>> mapper);`

`flatMap()` 同样返回一个 `Stream`，但这个 `Stream` 是经过展平之后的，会将里面嵌套的集合或流展开，返回结果一个 `Stream` 直接包含所有元素

实际执行了两个操作：
1. 映射：将每个元素转换为一个流
2. 扁平化：将所有生成的流合并成一个单独的流

|       | `map()`   | `flatMap()`         |
| ----- | --------- | ------------------- |
| 适用场景  | 简单的元素类型转换 | 涉及对集合或流的嵌套处理，如集合拆分  |
| 输入处理  | T -> R    | T -> Stream<R> -> R |
| 结果流结构 | 无改变       | 打开嵌套结构，展平           |
## `getClass().getName()` 和 `getClass().getSimpleName()`有什么区别

### `this.getClass().getName()`

- **返回值**：返回类的完整路径名，包括包名
- **适用于**：需要完整的类标识符来区分不同包中的同名类，或者需要使用类名进行反射等操作

### `this.getClass().getSimpleName()`

- **返回值**：返回不包含包名的类名称
- **适用于**：只需要类的名字而不需要其所在的包名，如日志记录、显示简洁的错误信息