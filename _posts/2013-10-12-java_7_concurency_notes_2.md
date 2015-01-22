---
layout: post
title: "Java 7 并发编程学习笔记（2）-- 线程管理（下）"
category: 技术
tags: Concurrency
---

## 守护线程的创建和运行

Java有一种特别的线程叫做守护线程。这种线程的优先级非常低，通常在程序里没有其他线程运行时才会执行它。当守护线程是程序里唯一在运行的线程时，JVM会结束守护线程并终止程序。

根据这些特点，守护线程通常用于在同一程序里给普通线程（也叫使用者线程）提供服务。它们通常无限循环的等待服务请求或执行线程任务。它们不能做重要的任务，因为我们不知道什么时候会被分配到CPU时间片，并且只要没有其他线程在运行，它们可能随时被终止。JAVA中最典型的这种类型代表就是垃圾回收器。

	使用 setDaemon() 方法让此线程成为守护线程。

**注意**

只能在start() 方法之前可以调用 setDaemon() 方法。一旦线程运行了，就不能修改守护状态。

可以使用 isDaemon() 方法来检查线程是否是守护线程（方法返回 true) 或者是使用者线程 (方法返回 false)。

<!--more-->

## 在线程里处理不受控制的异常

Java里有2种异常:

* 检查异常（Checked exceptions）: 这些异常必须强制捕获它们或在一个方法里的throws子句中。 例如， IOException 或者ClassNotFoundException。
* 未检查异常（Unchecked exceptions）: 这些异常不用强制捕获它们。例如， NumberFormatException。

在一个线程对象的 run() 方法里抛出一个检查异常，我们必须捕获并处理他们。因为 run() 方法不接受 throws 子句。当一个非检查异常被抛出，默认的行为是在控制台写下stack trace并退出程序。

幸运的是, Java 提供我们一种机制可以捕获和处理线程对象抛出的未检测异常来避免程序终结。

可以像下边一样实现一个类来处理非检查异常：


	public class ExceptionHandler implements UncaughtExceptionHandler{
		public void uncaughtException(Thread t, Throwable e){
			System.out.printf("An exception has been captured\n");
			System.out.printf("Thread: %s\n",t.getId());
			System.out.printf("Exception: %s: %s\n",e.getClass().getName(),e.getMessage());
			System.out.printf("Stack Trace: \n");
			e.printStackTrace(System.out);
			System.out.printf("Thread status: %s\n",t.getState());
		}
	}


当在一个线程里抛出一个异常，但是这个异常没有被捕获（这肯定是非检查异常）， JVM 检查线程的相关方法是否有设置一个未捕捉异常的处理者 。如果有，JVM 使用Thread 对象和 Exception 作为参数调用此方法 。

如果线程没有捕捉未捕获异常的处理者， 那么 JVM会把异常的 stack trace 写入操控台并结束任务。

**注意**

Thread 类有其他相关方法可以处理未捕获的异常。静态方法 setDefaultUncaughtExceptionHandler() 为应用里的所有线程对象建立异常 handler 。

当一个未捕捉的异常在线程里被抛出，JVM会寻找此异常的3种可能潜在的处理者（handler）。

首先, 它寻找这个未捕捉的线程对象的异常handle，如我们在在这个指南中学习的。如果这个handle 不存在，那么JVM会在线程对象的ThreadGroup里寻找非捕捉异常的handler，如在处理线程组内的不受控制异常里介绍的那样。如果此方法不存在，正如我们在这个指南中学习的，那么 JVM 会寻找默认非捕捉异常handle。

如果没有一个handler存在, 那么 JVM会把异常的 stack trace 写入操控台并结束任务。

## 使用本地线程变量

并发应用的一个关键地方就是共享数据。这个对那些扩展Thread类或者实现Runnable接口的对象特别重要。

如果你创建一个类对象，实现Runnable接口，然后多个Thread对象使用同样的Runnable对象，全部的线程都共享同样的属性。这意味着，如果你在一个线程里改变一个属性，全部的线程都会受到这个改变的影响。

有时，你希望程序里的各个线程的属性不会被共享。 Java 并发 API提供了一个很清楚的机制叫本地线程变量。

在一个实现了Runnable接口的类中可以做如下定义

	private static ThreadLocal<Date> startDate= new ThreadLocal<Date>() {
		protected Date initialValue(){
			return new Date();
		}
	};

本地线程变量为每个使用这些变量的线程储存属性值。可以用 get() 方法读取值和使用 set() 方法改变值。 如果第一次你访问本地线程变量的值，如果没有值给当前的线程对象，那么本地线程变量会调用 initialValue() 方法来设置值给线程并返回初始值。

**注意**

本地线程类还提供 remove() 方法，删除存储在线程本地变量里的值。

Java 并发 API 包括 InheritableThreadLocal 类提供线程创建线程的值的遗传性 。如果线程A有一个本地线程变量，然后它创建了另一个线程B，那么线程B将有与A相同的本地线程变量值。 你可以覆盖 childValue() 方法来初始子线程的本地线程变量的值。 它接收父线程的本地线程变量作为参数。

## 线程组

Java并发 API里有个有趣的方法是把线程分组。这个方法允许我们按线程组作为一个单位来处理。例如，你有一些线程做着同样的任务，你想控制他们，无论多少线程还在运行，他们的状态会被一个call 中断。

Java 提供 ThreadGroup 类来组织线程。 ThreadGroup 对象可以由 Thread 对象组成和由另外的 ThreadGroup 对象组成,生成线程树结构。

	使用 activeCount() 和 enumerate() 方法来获取线程个数和与ThreadGroup对象关联的线程的列表。
	用interrupt() 方法中断组里的其他线程。
	
## 处理线程组内的不受控制异常

创建一个方法，捕获所有被ThreadGroup类的任何线程抛出的非捕捉异常。

	覆盖ThreadGroup 类的 uncaughtException() 方法。

ThreadGroup 类的其中一个线程抛出异常时，就会调用此方法 。在这里，这个方法会把异常和抛出它的线程的信息写入操控台并中断ThreadGroup类的其余线程。

当一个非捕捉异常在线程内抛出，JVM会为这个异常寻找3种可能handlers。

首先, 它寻找这个未捕捉的线程对象的异常handle，如果这个handle 不存在，那么JVM会在线程对象的ThreadGroup里寻找非捕捉异常的handler。如果此方法不存在，那么 JVM 会寻找默认非捕捉异常handle。如果没有 handlers存在, 那么 JVM会把异常的 stack trace 写入控制台并结束任务。

## 用线程工厂创建线程

在面向对象编程的世界中，工厂模式是最有用的设计模式。它是一个创造模式，还有它的目的是创建一个或几个类的对象的对象。然后，当我们想创建这些类的对象时，我们使用工厂来代替new操作。

有了这个工厂，我们有这些优势来集中创建对象们：

* 更简单的改变了类的对象创建或者说创建这些对象的方式。
* 更简单的为了限制的资源限制了对象的创建。 例如， 我们只new一个此类型的对象。
* 更简单的生成创建对象的统计数据。

Java提供一个接口， ThreadFactory 接口实现一个线程对象工厂。 并发 API 使用线程工厂来创建线程的一些基本优势。

ThreadFactory 接口只有一个方法是 newThread。它接收 Runnable 对象作为参数并返回一个 Thread 对象。当你实现一个 ThreadFactory 接口，你必须实现此接口并覆盖这个方法。最基本的 ThreadFactory只有一行。

	return new Thread(r);

你可以加一些变量来提升这个实现：

* 在这个例子中创建自定义线程，名字使用特别形式或者继承Java Thread类来创建自己的Thread类。
* 保存线程创建数据，如之前的例子。
* 限制线程创建个数。
* 验证线程的创建。
* 和你能想到的任何东西。

使用工厂设计模式是一种最佳实践，但是，如果你实现一个 ThreadFactory 接口来集中创建线程，那么你必须再检查代码确保使用的线程都是用这个工厂创建的。
