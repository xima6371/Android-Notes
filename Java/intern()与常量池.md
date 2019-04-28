---
title: intern()与常量池
date: 2019-03-01 22:29:55
tags: 常量池
---

本文介绍了调用intern()时的工作机制

<!-- more -->

#### String中intern()与常量池

在《深入了解Java虚拟机》一书中，有这么一段代码来介绍常量池的工作方式



```java
public static void main(String[] args) {
    String a = new StringBuilder("计算机").append("软件").toString();
    String b = new StringBuilder("ja").append("va").toString();

    System.out.println(a.intern() == a);
    System.out.println(b.intern() == b);
}
```



代码运行结果为：

```java
**Jdk1.6**
false   
false	

**Jdk1.7**
true
false
```



按照作者说的在Jdk1.6中，intern()方法会返回一个string object，当常量池中没有该字符串常量时(若有则直接返回常量池中的object)，会在方法区中(即常量池中)创建该常量的实例副本的同时返回object，且在堆上也创建一个相应的string object。

堆上的和方法区的string object地址是不一致的，这就导致了**a.intern() == a**为false的结果，**b.intern() == b**同理。



而在Jdk1.7中，常量池中无对应字符串常量时，则不会在常量池中创建新的string object，而是将堆上对应字符串的地址拷贝到常量池中，故**a.intern() == a**为true，按理说b也应该为true，但是实际却为false

> //new StringBuilder()和调用append()都会在堆上创建对象
> //并在常量池中增加对应的String object
> //toString则仅在堆上创建对象

![Jdk1.7](http://ww1.sinaimg.cn/large/006S5wgFgy1g18gt5lqxej30p20ge0sz.jpg)



那么为什么b调用后仍为false呢，根据[《深入理解java虚拟机》String.intern()探究](http://http://baijiahao.baidu.com/s?id=1568390319555291)一文可知

System类中有这么运行了Version的init()方法，在该类中，有一个常量，其值为java，这就导致了我们常量池中，在调用b.intern()前就已经有了“java”这个string object，由于b是在堆中的，b.inern()实在常量池中的，两者地址不一致，故得到结果false。

```java
public class Version {
    private static final String launcher_name = "java";
 	//其他   
}
```



#### 参考文章

[几张图轻松理解String.intern()](https://blog.csdn.net/soonfly/article/details/70147205)













