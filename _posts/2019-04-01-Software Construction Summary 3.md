---
layout: post
title: "Software Construction Summary 3"
subtitle: ""
date: 2019-04-01
author: Aether
category: coding
tags: CODE SOFTWARE-CONSTRUCTION JAVA
finished: true
---

## Equality in ADT and OOP

### equals
- equality: 自反、传递、对称
- 引用等价性 referential equality —— `==`
	- 判断是否指向同一块内存地址
- 对称等价性 object equality —— `equals()`
- `instanceof` is dynamic type checking, not the static type checking. It should be disallowed anywhere except for implementing equals. (`getClass()` is also disallowed.)
- `a.equals(b)` ---> `a.hashcode() == b.hashcode()`
- `a.hashcode() == b.hashcode()` --\\-->  `a.equals(b)`
- `x.equals(null)` must return false
- `int` 放入`Map<String, Integer>`后自动变为`Integer`类型，取出后应该用`equals()`方法比较

### clone
- 调用`clone()`返回的对象是一个独立的副本，两个对象地址不同，属性相同。
- 若类型`A`中有mutable成员变量`B`，那么对`A`的clone对象进行修改时将修改原有的`B`
- 如果实现完整的深拷贝，需要被复制对象的继承链、引用链上的每一个对象都实现克隆机制。

### Immutable type
- AF(a) = AF(b)
- 必须同时重写`equals()`和`hashcode()`
- 对于immutable类型的对象来说，因为没有mutable方法，所以观察等价性与行为等价性一致。

### Mutable type
- **观察等价性 observational equality**
	- 在不改变状态的情况下观察两个mutable对象是否看起来一致
- 行为等价性 behavioral equality
	- 调用对象的任何方法都展示出一致的结果
	- mutable内容改变后`hashcode`随之改变
        - 如果某个mutable的对象包含在集合类中，当其发生改变后，集合类的行为不确定😢
        - 只影响`HashSet`，不影响`TreeSet`
- `Collections` 使用**观察等价性** —— `List` 的`equals()`方法被重写
    - List 首先判断Null 然后判断size 最后依次比较每个内容的`equals()`方法比较
- 其他可变类型 (如`StringBuilder`) 使用**行为等价性** —— `StringBuilder` 的 `equals()`方法继承自`Object`类
- 大多情况下无需重写 直接继承

