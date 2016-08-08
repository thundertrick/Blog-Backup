---
title: Effective Java - 1.1 类的创建
date: 2016-08-08 20:13:19
tags: [android,java]
---

获得一个实例最常见的方法就是构造器。但构造器并非在所有场景下都是最佳方案。本文收集几个特殊的构造方法以备需要。文中大部分内容源自 Joshua Bloch 的 Effective Java （second edition），实际上也是一个读书笔记。

<!--more-->

## 0x01 静态工厂方法

### 1. 方法示例：

~~~java
public static Boolean valueOf(boolean b) {
	return b ? Boolean.TRUE : Boolean.FALSE;
}
~~~

### 2. 优点：

1. **有名称**。构造器实际上只能用类名命名，无法更多地描述构造信息。并且实际使用中，同一个类常有数个不同的构造方法，如果不加以区分（缺少必要注释）则很容易使用错误。而静态工厂方法，可以使用方法名去描述构造实例的更多信息，易于区分。
2. **不必每次调用都创建一个新的对象**。重复调用可以利用好预先构建好的实例，从而减少了重复创建的开支。这点我目前的理解是静态工厂方法类内部可以调用其他的单例实例。
3. **可以返回原类型的任何子类型**。这样在选择返回对象时就有了更大的灵活性。
4. **创建参数化实例时代码更加简洁**。举例说明，原始代码：

~~~java
Map<String, List<String>> m = new HashMap<String, List<String>>;
~~~

假设`HashMap`提供了这样的静态工厂方法：

~~~java
public static <K, V> HashMap<K, V> newInstance() {
	return new HashMap<K, V>();
}
~~~

那么原始代码就可以简化为：

~~~java
Map<String, List<String>> m = HashMap.newInstance();
~~~

### 3. 缺点：
1. 类如果不含公有的或者受保护的构造器，就不能被子类化。
2. 与其他静态方法没有本质上的区别。这个主要是对JavaDoc不友好，无法像构造方法一样被标识。

### 使用
静态工厂方法常用的方法名称如下：

1. `valueOf`, `of`
2. `getInstance`, `newInstance`
3. `getType`, `newType`


