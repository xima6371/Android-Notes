---
title: Java Collection简介
date: 2019-04-01 22:29:55
tags: Java基础
---



Java集合的简单介绍

<!-- more -->

### Java集合分类

> Java集合是容纳对象（对象的引用）的容器

1. Set（集合）
   * 无序
   * 不可重复
2. List（列表）
   * 有序
   * 可重复
3. Queue（队列 Java5增加）
   * 后进先出
4. Map（映射集合）
   * key不可重复，可为null，否则就覆盖数据



#### Java集合和数组区别

|      | 初始化时容量 |             存储类型             |
| :--: | :----------: | :------------------------------: |
| 集合 |   长度不定   |        基本数据类型，对象        |
| 数组 |   固定长度   | 对象，基本数据类型需转化成包装类 |



#### Java集合继承关系

> Java集合主要继承于Collection、Map两个接口 。（虚线框为interface,实体框为class）

![Collection下的子类](https://camo.githubusercontent.com/b1974bdd58e78c12612907d0ad27b893f8fbdbb9/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f333938353536332d653766656266333634643864383233352e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)

Collection进行迭代需要实现iterator()方法，该方法是为了获取一个迭代器（继承自iterator接口）

```java
public interface Iterator<E> {
    /**
     * Returns {@code true} if the iteration has more elements.
     * (In other words, returns {@code true} if {@link #next} would
     * return an element rather than throwing an exception.)
     *
     * @return {@code true} if the iteration has more elements
     */
    boolean hasNext();

    /**
     * Returns the next element in the iteration.
     *
     * @return the next element in the iteration
     * @throws NoSuchElementException if the iteration has no more elements
     */
    E next();
}
```

*Tip：基本数据类型传递的是值，引用类型传递的仅仅是对象的引用变量*



![Map下继承关系](https://camo.githubusercontent.com/517258d1f1e996c7a728c8fe067c439722fc61ce/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f333938353536332d303630353231303738343961373630332e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)



对于Map中的所有key，可以视为一组Set，因为都满足不可重复，且无序的特性

而Map中的所有value，可视为list，不同的是Map的索引只能为对象，而list的索引是int类型