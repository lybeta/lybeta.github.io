---
layout: post
title: "Java 7 并发编程学习笔记（6）-- 线程执行者（上）"
category: 技术
tags: Concurrency
---
通常，当你在Java中开发一个简单的并发编程应用程序，你会创建一些Runnable对象并创建相应的Thread对象来运行它们。如果你开发一个运行多个并发任务的程序，这种途径的缺点如下：

* 你必须要实现很多相关代码来管理Thread对象（创建，结束，获得的结果）。
* 你必须给每个任务创建一个Thread对象。如果你执行一个大数据量的任务，那么这可能影响应用程序的吞吐量。
* 你必须有效地控制和管理计算机资源。如果你创建太多线程，会使系统饱和。

为了解决以上问题，从Java5开始JDK并发API提供一种机制。这个机制被称为**Executor framework**，接口核心是Executor，Executor的子接口是ExecutorService，而ThreadPoolExecutor类则实现了这两个接口。

这个机制将任务的创建与执行分离。使用执行者，你只要实现Runnable对象并将它们提交给执行者。执行者负责执行，实例化和运行这些线程。除了这些，它还可以使用线程池提高了性能。当你提交一个任务给这个执行者，它试图使用线程池中的线程来执行任务，从而避免继续创建线程。

Callable接口是Executor framework的另一个重要优点。它跟Runnable接口很相似，但它提供了两种改进，如下：

* 这个接口中主要的方法叫call()，可以返回结果。
* 当你提交Callable对象到执行者，你可以获取一个实现Future接口的对象，你可以用这个对象来控制Callable对象的状态和结果。
<!--more-->

## 创建一个线程执行者

使用Executor framework的第一步就是创建一个ThreadPoolExecutor类的对象。你可以使用这个类提供的4个构造器或Executors工厂类来 创建ThreadPoolExecutor。一旦有执行者，你就可以提交Runnable或Callable对象给执行者来执行。

ThreadPoolExecutor有4个不同的构造器，但由于它 们的复杂性，Java并发API提供Executors类来构造执行者和其他相关对象。即使我们可以通过ThreadPoolExecutor类的任意一 个构造器来创建ThreadPoolExecutor，但这里推荐使用Executors类。

	private ThreadPoolExecutor executor = (ThreadPoolExecutor)Executors.newCachedThreadPool();

**注意**

使用通过newCachedThreadPool()方法创建的执行者，只有当你有一个合理的线程数或任务有一个很短的执行时间。

一旦你创建执行者，你可以使用execute()方法提交Runnable或Callable类型的任务。

获得关于执行者信息可以使用以下方法：

* getPoolSize()：此方法返回线程池实际的线程数。
* getActiveCount()：此方法返回在执行者中正在执行任务的线程数。
* getCompletedTaskCount()：此方法返回执行者完成的任务数。
* getLargestPoolSize()方法，返回池中某一时刻最大的线程数。

ThreadPoolExecutor 类和一般执行者的一个关键方面是，你必须明确地结束它。如果你没有这么做，这个执行者会继续它的执行，并且这个程序不会结束。如果执行者没有任务可执行， 它会继续等待新任务并且不会结束它的执行。一个Java应用程序将不会结束，除非所有的非守护线程完成它们的执行。所以，如果你不结束这个执行者，你的应用程序将不会结束。

当执行者完成所有待处理的任务，你可以使用ThreadPoolExecutor类的shutdown()方法来表明你想要结束执行者。在你调用shutdown()方法之后，如果你试图提交其他任务给执行者，它将会拒绝，并且抛出RejectedExecutionException异常。

**ThreadPoolExecutor类也提供其他与结束执行者相关的方法，这些方法是：**

* shutdownNow()：此方法立即关闭执行者。它不会执行待处理的任务，但是它会返回待处理任务的列表。当你调用这个方法时，正在运行的任务继续它们的执行，但这个方法并不会等待它们的结束。
* isTerminated()：如果你已经调用shutdown()或shutdownNow()方法，并且执行者完成关闭它的处理时，此方法返回true。
* isShutdown()：如果你在执行者中调用shutdown()方法，此方法返回true。
* awaitTermination(long timeout, TimeUnit unit)：此方法阻塞调用线程，直到执行者的任务结束或超时。TimeUnit类是个枚举类，有如下常 量：DAYS，HOURS，MICROSECONDS， MILLISECONDS， MINUTES,，NANOSECONDS 和SECONDS。

注意事项：如果你想要等待任务的完成，不管它们的持续时间，请使用大的超时，如：DAYS。

## 创建一个大小固定的线程执行者

Executors类提供一个方法来创建大小固定的线程执行者。这个执行者有最大线程数。 如果你提交超过这个最大线程数的任务，这个执行者将不会创建额外的线程，并且剩下的任务将会阻塞，直到执行者有空闲线程。这种行为，保证执行者不会引发应用程序性能不佳的问题。

	private ThreadPoolExecutor executor=(ThreadPoolExecutor)Executors.newFixedThreadPool(5);

**注意**

Executors类同时提供newSingleThreadExecutor()方法。这是大小固定的线程执行者的一个极端例子。它创建只有一个线程的执行者，所以它在任意时刻只能执行一个任务。

## 执行者执行返回结果的任务

Executor framework的一个优点是你可以并发执行返回结果的任务。Java并发API使用以下两种接口来实现：

* Callable：此接口有一个call()方法。在这个方法中，你必须实现任务的（处理）逻辑。Callable接口是一个参数化的接口。意味着你必须表明call()方法返回的数据类型。
* Future：此接口有一些方法来保证Callable对象结果的获取和管理它的状态。

executor使用submit()方法提交一个Callable对象给执行者执行。这个方法接收Callable对象参数，并且返回一个Future对象，你可以以这两个目的来使用它：

* 你可以控制任务的状态：你可以取消任务，检查任务是否已经完成。使用isDone()方法来检查任务是否已经完成。
* 你可以获取call()方法返回的结果。使用get()方法。这个方法会等待，直到Callable对象完成call()方法的执行，并且返回它的结果。如果线程在get()方法上等待结果时被中断，它将抛出InterruptedException异常。如果call()方法抛出异常，这个方法会抛出ExecutionException异常。

**注意**

当你调用Future对象的get()方法，并且这个对象控制的任务未完成，这个方法会阻塞直到任务完成。Future接口提供其他版本的get()方法：

* get(long timeout, TimeUnit unit)：这个版本的get方法，如果任务的结果不可用，等待它在指定的时间内。如果时间超时，并且结果不可用，这个方法返回null值。 TimeUnit类是个枚举类，有如下常量：DAYS，HOURS，MICROSECONDS， MILLISECONDS， MINUTES,，NANOSECONDS 和SECONDS。

## 运行多个任务并处理第一个结果

在并发编程中的一个常见的问题就是，当有多种并发任务解决一个问题时，你只对这些任务的第一个结果感兴趣。

**ThreadPoolExecutor类中的invokeAny()方法接收任务数列，并启动它们，返回完成时没有抛出异常的第一个任务的结果。该方法返回的数据类型与启动任务的call()方法返回的类型一样。**

如果有两个任务，可以返回true值或抛出异常。有以下4种情况：

* 两个任务都返回ture。invokeAny()方法的结果是第一个完成任务的名称。
* 第一个任务返回true，第二个任务抛出异常。invokeAny()方法的结果是第一个任务的名称。
* 第一个任务抛出异常，第二个任务返回true。invokeAny()方法的结果是第二个任务的名称。
* 两个任务都抛出异常。在本例中，invokeAny()方法抛出一个ExecutionException异常。

## 运行多个任务并处理所有结果

执行者框架允许你在不用担心线程创建和执行的情况下，并发的执行任务。它还提供了Future类，这个类可以用来控制任务的状态,也可以用来获得执行者执行任务的结果。

如果你想要等待一个任务完成，你可以使用以下两种方法：

* 如果任务执行完成，Future接口的isDone()方法将返回true。
* ThreadPoolExecutor类的awaitTermination()方法使线程进入睡眠，直到每一个任务调用shutdown()方法之后完成执行。

这两种方法都有一些缺点。第一个方法，你只能控制一个任务的完成。第二个方法，你必须等待一个线程来关闭执行者，否则这个方法的调用立即返回。

executor使用invokeAll()方法等待提交任务列表的完成。这个方法接收Callable对象列表和返回Future对象列表。这个列表将会有列表中每个任务的一个Future对象。**Future对象列表的第一个对象是Callable对象列表控制的第一个任务，以此类推。**

**两点需要考虑：**

* 第一点要考虑的是，在存储结果对象的列表中声明的Future接口参数化的数据类型必须与使用的Callable对象的参数化相兼容。在本例中，你已经使用相同数据类型：Result类型。

* 另一个重要的一点就是关于invokeAll()方法，你会使用Future对象获取任务的结果。当所有的任务完成时，这个方法结束，如果你调用返回的Future对象的isDone()方法，所有调用都将返回true值。

ExecutorService类提供其他版本的invokeAll()方法：

* invokeAll(Collection<? extends Callable<T>> tasks, long timeout,TimeUnit unit)：此方法执行所有任务，当它们全部完成且未超时，返回它们的执行结果。TimeUnit类是个枚举类，有如下常量：DAYS，HOURS，MICROSECONDS， MILLISECONDS， MINUTES,，NANOSECONDS 和SECONDS。