---
layout: post
title: "Java 7 并发编程学习笔记（9）-- 并发集合"
category: 技术
tags: Concurrency
---

Java提供了你可以在你的并发程序中使用的，而且不会有任何问题或不一致的数据集合。基本上，Java提供两种在并发应用程序中使用的集合：

* 阻塞集合：这种集合包括添加和删除数据的操作。如果操作不能立即进行，是因为集合已满或者为空，该程序将被阻塞，直到操作可以进行。
* 非阻塞集合：这种集合也包括添加和删除数据的操作。如果操作不能立即进行，这个操作将返回null值或抛出异常，但该线程将不会阻塞。

<!--more-->

## 使用非阻塞线程安全的列表

非阻塞列表提供这些操作：如果操作不能立即完成（比如，你想要获取列表的元素而列表却是空的），它将根据这个操作抛出异常或返回null值。

Java 7引进实现了非阻塞并发列表的`ConcurrentLinkedDeque`类。

主要方法：

* getFirst()和getLast()：这些方法将分别返回列表的第一个和最后一个元素。它们不会从列表删除返回的元素。如果列表为空，这些方法将抛出NoSuchElementExcpetion异常。
* peek()、peekFirst()和peekLast()：这些方法将分别返回列表的第一个和最后一个元素。它们不会从列表删除返回的元素。如果列表为空，这些方法将返回null值。
* remove()、removeFirst()、 removeLast()：这些方法将分别返回列表的第一个和最后一个元素。它们将从列表删除返回的元素。如果列表为空，这些方法将抛出NoSuchElementExcpetion异常。
* poll()、pollFirst()和pollLast()：这些方法将分别返回列表的第一个和最后一个元素。它们将从列表删除返回的元素。如果列表为空，这些方法将返回null值。

## 使用阻塞线程安全的列表

阻塞列表与非阻塞列表的主要区别是，阻塞列表中的添加和删除元素方法，如果由于列表已满或为空而导致这些操作不能立即进行，它们将阻塞调用的线程，直到这些操作可以进行。

Java包含实现阻塞列表`的LinkedBlockingDeque`类。

主要方法：

* takeFirst() 和takeLast()：这些方法分别返回列表的第一个和最后一个元素。它们从列表删除返回的元素。如果列表为空，这些方法将阻塞线程，直到列表有元素。
* getFirst() 和getLast()：这些方法分别返回列表的第一个和最后一个元素。它们不会从列表删除返回的元素。如果列表为空，这些方法将抛出NoSuchElementExcpetion异常。
* peek()、peekFirst(),和peekLast()：这些方法分别返回列表的第一个和最后一个元素。它们不会从列表删除返回的元素。如果列表为空，这些方法将返回null值。
* poll()、pollFirst()和 pollLast()：这些方法分别返回列表的第一个和最后一个元素。它们从列表删除返回的元素。如果列表为空，这些方法将返回null值。
* add()、 addFirst()、addLast()：这些方法分别在第一个位置和最后一个位置上添加元素。如果列表已满（你已使用固定大小创建它），这些方法将抛出IllegalStateException异常。

## 用优先级对使用阻塞线程安全的列表排序

当你需要使用一个有序列表的数据结构时，Java提供了`PriorityBlockingQueue`类。

你想要添加到PriorityBlockingQueue中的所有元素必须实现Comparable接口。这个接口有一个compareTo()方法，它接收同样类型的对象，你有两个比较的对象：一个是执行这个方法的对象，另一个是作为参数接收的对象。如果本地对象小于参数，则该方法返回小于0的数值。如果本地对象大于参数，则该方法返回大于0的数值。如果本地对象等于参数，则该方法返回等于0的数值。

PriorityBlockingQueue使用compareTo()方法决定插入元素的位置。默认情况下较大的元素将被放在队列的尾部。

阻塞数据结构（blocking data structure）是PriorityBlockingQueue的另一个重要特性。它所包含的方法，如果它们不能立即进行它们的操作，则阻塞这个线程直到它们的操作可以进行。

主要方法：

* poll()：方法从队列中获取元素。这个方法返回并删除队列的第一个元素。
* clear()：这个方法删除队列中的所有元素。
* take()：这个方法返回并删除队列中的第一个元素。如果队列是空的，这个方法将阻塞线程直到队列有元素。
* put(E e)：E是用来参数化PriorityBlockingQueue类的类。这个方法将作为参数传入的元素插入到队列中。
* peek()：这个方法返回列队的第一个元素，但不删除它。

## 使用线程安全的、带有延迟元素的列表

`DelayedQueue`类是Java API提供的一种有趣的数据结构，并且你可以用在并发应用程序中。在这个类中，你可以存储带有激活日期的元素。方法返回或抽取队列的元素将忽略未到期的数据元素。它们对这些方法来说是看不见的。

存储到DelayedQueue类中的元素必须实现Delayed接口。这个接口允许你处理延迟对象，所以你将实现存储在DelayedQueue对象的激活日期，这个激活时期将作为对象的剩余时间，直到激活日期到来。这个接口强制实现以下两种方法：

* compareTo(Delayed o)：Delayed接口继承Comparable接口。如果执行这个方法的对象的延期小于作为参数传入的对象时，该方法返回一个小于0的值。如果执行这个方法的对象的延期大于作为参数传入的对象时，该方法返回一个大于0的值。如果这两个对象有相同的延期，该方法返回0。
* getDelay(TimeUnit unit)：该方法返回与此对象相关的剩余延迟时间，以给定的时间单位表示。TimeUnit类是一个枚举类，有以下常量：DAYS、HOURS、 MICROSECONDS、MILLISECONDS、 MINUTES、 NANOSECONDS 和 SECONDS。

主要方法：

* getDelay()：方法返回在实际日期和激活日期之间的纳秒数。这两个日期都是Date类的对象。
* getTime()：方法返回一个被转换成毫秒的日期，你已转换那个值为作为参数接收的TimeUnit。
* poll()：方法将所有元素写入到控制台。这个方法检索并删除队列的第一个元素。如果队列中没有任务到期的元素，这个方法返回null值。
* clear()：这个方法删除队列中的所有元素。
* offer(E e)：E是代表用来参数化DelayQueue类的类。这个方法插入作为参数传入的元素到队列中。
* peek()：这个方法检索，但不删除队列的第一个元素。
* take()：这具方法检索并删除队列的第一个元素。如果队列中没有任何激活的元素，执行这个方法的线程将被阻塞，直到队列有一些激活的元素。


## 使用线程安全的NavigableMap

Java API 提供了`ConcurrentNavigableMap`接口，也提供了这个接口的实现类，这个类是`ConcurrentSkipListMap`，它实现了非阻塞列表且拥有ConcurrentNavigableMap的行为。

在内部实现中，它使用Skip List来存储数据。Skip List是基于并行列表的数据结构，它允许我们获取类似二叉树的效率。使用它，你可以得到一个排序的数据结构，这比排序数列使用更短的访问时间来插入、搜索和删除元素。

当你往map中插入数据时，它使用key来排序它们，所以，所有元素将是有序的。除了返回具体的元素，这个类也提供了获取map的子map的方法。

主要方法：

* firstEntry()方法返回map第一个元素的Map.Entry对象，且不会删除这个元素。
* lastEntry()方法返回map最后一个元素的Map.Entry对象，且不会删除这个元素。
* headMap(K toKey)：K是参数化ConcurrentSkipListMap对象的Key值的类。返回此映射的部分视图，其键值小于 toKey。
* tailMap(K fromKey)：K是参数化ConcurrentSkipListMap对象的Key值的类。返回此映射的部分视图，其键大于等于 fromKey。
* putIfAbsent(K key, V Value)：如果key不存在map中，则这个方法插入指定的key和value。
* pollFirst()方法来处理subMap()方法返回的这些元素。这个方法将返回并删除submap中的第一个Map.Entry对象。
* pollLastEntry()：这个方法返回并删除map中最后一个元素的Map.Entry对象。
* replace(K key, V Value)：如果这个key存在map中，则这个方法将指定key的value替换成新的value。

## 创建并发随机数

Java并发API提供指定的类在并发应用程序中生成伪随机。它是`ThreadLocalRandom`类，这是Java 7版本中的新类。

它使用线程局部变量。每个线程希望以不同的生成器生成随机数，但它们是来自相同类的管理，这对程序员是透明的。在这种机制下，你将获得比使用共享的Random对象为所有线程生成随机数更好的性能。

使用示例：

	public class TaskLocalRandom implements Runnable {
		public TaskLocalRandom() {
			ThreadLocalRandom.current();
		}
		@Override
		public void run() {
			String name=Thread.currentThread().getName();
			for (int i=0; i<10; i++){
				System.out.printf("%s: %d\n",name,ThreadLocalRandom.current().nextInt(10));
			}
		}
	}

## 使用原子变量

在Java 1.5中就引入了原子变量，它提供对单个变量的原子操作。当一个线程正在操作一个原子变量时，即使其他线程也想要操作这个变量，类的实现中含有一个检查那步骤操作是否完成的机制。 基本上，操作获取变量的值，改变本地变量值，然后尝试以新值代替旧值。如果旧值还是一样，那么就改变它。如果不一样，方法再次开始操作。这个操作称为 Compare and Set（校对注：简称CAS，比较并交换的意思）。

比如说为了实现为余额值加上收入，你要使用 AtomicLong 类的getAndAdd() 方法，用特定的参数值增加它并返回值；为了实现减少余额值，你也要使用 getAndAdd() 方法。

## 使用原子 Array

Java 在原子类中实现了CAS机制。这些类提供了compareAndSet() 方法；这个方法是CAS操作的实现和其他方法的基础。

Java 中还引入了原子Array，用来实现Integer类型和Long类型数组的操作。

主要方法：

* getAndIncrement()：array里的全部元素 +1
* getAndDecrement()：array里的全部元素 -1
* get(int i): 返回array中第i个位置上的值
* set(int I, int newValue): 设置array中第i个位置上的值为newValue

如今，Java仅提供了另一个原子 array类。它是 `AtomicLongArray` 类，与 IntegerAtomicArray 类提供了相同的方法。

