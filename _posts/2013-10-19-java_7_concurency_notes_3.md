---
layout: post
title: "Java 7 并发编程学习笔记（3）-- 基本线程同步"
category: 技术
tags: Concurrency
---

## 同步方法

在Java中我们使用一个最基本的同步方法，即 synchronized关键字来控制并发访问方法。只有一个执行线程将会访问一个对象中被synchronized关键字声明的方法。如果另一个线程试图访问同一个对象中任何被synchronized关键字声明的方法，它将被暂停，直到第一个线程结束方法的执行。

换句话说，每个方法声明为synchronized关键字是一个临界区，Java只允许一个对象执行其中的一个临界区。

静态方法有不同的行为。只有一个执行线程访问被synchronized关键字声明的静态方法，但另一个线程可以访问该类的一个对象中的其他非静态的方法。 你必须非常小心这一点，因为两个线程可以访问两个不同的同步方法，如果其中一个是静态的而另一个不是。如果这两种方法改变相同的数据,你将会有数据不一致的错误。

**使用synchronized关键字，在并发应用程序中，我们保证了正确地访问共享数据。**
<!--more-->

只有一个线程能访问一个对象的声明为synchronized关键字的方法。如果一个线程A正在执行一个 synchronized方法，而线程B想要执行同个实例对象的synchronized方法，它将阻塞，直到线程A执行完。但是如果线程B访问相同类的不同实例对象，它们都不会被阻塞。

**synchronized关键字不利于应用程序的性能**，所以你必须仅在修改共享数据的并发环境下的方法上使用它。如果你有多个线程正在调用一个synchronized方法，在同一时刻只有一个线程执行它，而其他的线程将会等 待。如果这个操作没有使用synchronized关键字，所有线程可以在同一时刻执行这个操作，减少总的执行时间。如果你知道一个方法将不会被多个线程 调用，请不要使用synchronized关键字。

你可以使用递归调用synchronized方法。当线程访问一个对象的synchronized方法，你可以调用该对象的其他synchronized方法，包括正在执行的方法。它将不会再次访问synchronized方法。

我们可以使用synchronized关键字来保护访问的代码块，替换在整个方法上使用synchronized关键字。我们应该使用 synchronized关键字以这样的方式来保护访问的共享数据，其余的操作留出此代码块，这将会获得更好的应用程序性能。这个目标就是让临界区（在同一时刻可以被多个线程访问的代码块）尽可能短。我们已经使用了synchronized关键字来保护访问指令，将不使用共享数据的长操作留出此代码块。当你以这个方式使用synchronized关键字，你必须通过一个对象引用作为参数。只有一个线程可以访问那个对象的synchronized代码（代码块或方法）。通常，我们将使用this关键字引用执行该方法的对象。

	synchronized (this) {
		// Java code
	}
	
## 在同步的类里安排独立属性

当你使用synchronized关键字来保护代码块时，你必须通过一个对象的引用作为参数。通常，你将会使用this关键字来引用执行该方法的对象，但是你也可以使用其他对象引用。通常情况下，这些对象被创建只有这个目的。比如，你在一个类中有被多个线程共享的两个独立属性。你必须同步访问每个变量，如果有一个线程访问一个属性和另一个线程在同一时刻访问另一个属性，这是没有问题的。

当你使用synchronized关键字来保护代码块，你使用一个对象作为参数。JVM可以保证只有一个线程可以访问那个对象保护所有的代码块（请注意，我们总是谈论的对象，而不是类）。

	private long vacanciesCinema1;
	private long vacanciesCinema2;
	public Cinema(){
		controlCinema1=new Object();
		controlCinema2=new Object();
		vacanciesCinema1=20;
		vacanciesCinema2=20;
	}
	
	public boolean sellTickets1 (int number) {
		synchronized (controlCinema1) {
			if (number<vacanciesCinema1) {
				vacanciesCinema1-=number;
				return true;
			} else {
				return false;
			}
		}
	}


注释：在这个示例中，我们用一个对象来控制vacanciesCinema1属性的访问。所以，在任意时刻，只有一个线程能修改该属性。用另一个对象来控制 vacanciesCinema2属性的访问。所以，在任意时刻，只有一个线程能修改这个属性。但是可能有两个线程同时运行，一个修改 vacancesCinema1属性而另一个修改vacanciesCinema2属性。

## 在同步代码中使用条件

在并发编程中的一个经典问题是生产者与消费者问题，我们有一个数据缓冲区，一个或多个数据的生产者在缓冲区存储数据，而一个或多个数据的消费者，把数据从缓冲区取出。

由于缓冲区是一个共享的数据结构，我们必须采用同步机制，比如synchronized关键字来控制对它的访问。但是我们有更多的限制因素，如果缓冲区是满的，生产者不能存储数据，如果缓冲区是空的，消费者不能取出数据。

对于这些类型的情况，Java在Object对象中提供wait()，notify()，和notifyAll() 方法的实现。一个线程可以在synchronized代码块中调用wait()方法。如果在synchronized代码块外部调用wait()方法，JVM会抛出IllegalMonitorStateException异常。当线程调用wait()方法，JVM让这个线程睡眠，并且释放控制 synchronized代码块的对象，这样，虽然它正在执行但允许其他线程执行由该对象保护的其他synchronized代码块。为了唤醒线程，你必 须在由相同对象保护的synchronized代码块中调用notify()或notifyAll()方法。

**在while循环中，你必须保持检查条件和调用wait()方法。你不能继续执行，直到这个条件为true。**

## 使用Lock同步代码块

Java提供另外的机制用来同步代码块。它比synchronized关键字更加强大、灵活。它是基于Lock接口和实现它的类（如ReentrantLock）。这种机制有如下优势：

* 它允许以一种更灵活的方式来构建synchronized块。使用synchronized关键字，你必须以结构化方式得到释放synchronized代码块的控制权。Lock接口允许你获得更复杂的结构来实现你的临界区。
* Lock 接口比synchronized关键字提供更多额外的功能。新功能之一是实现的tryLock()方法。这种方法试图获取锁的控制权并且如果它不能获取该锁，是因为其他线程在使用这个锁，它将返回这个锁。使用synchronized关键字，当线程A试图执行synchronized代码块，如果线程B正在执行它，那么线程A将阻塞直到线程B执行完synchronized代码块。使用锁，你可以执行tryLock()方法，这个方法返回一个 Boolean值表示，是否有其他线程正在运行这个锁所保护的代码。
* 当有多个读者和一个写者时，Lock接口允许读写操作分离。
* Lock接口比synchronized关键字提供更好的性能。

当我们通过锁来实现一个临界区并且保证只有一个执行线程能运行一个代码块，我们必须创建一个ReentrantLock对象。在临界区的起始部分，我们必须通过使用lock()方法来获得锁的控制权。当一个线程A调用这个方法时，如果 没有其他线程持有这个锁的控制权，那么这个方法就会给线程A分配这个锁的控制权并且立即返回允许线程A执行这个临界区。否则，如果其他线程B正在执行由这 个锁控制的临界区，lock()方法将会使线程A睡眠直到线程B完成这个临界区的执行。

**在临界区的尾部，我们必须使用unlock()方法来释放锁的控制权，允许其他线程运行这个临界区。**如果你在临界区的尾部没有调用unlock()方法，那么其他正在等待该代码块的线程将会永远等待，造成 死锁情况。如果你在临界区使用try-catch代码块，别忘了在finally部分的内部包含unlock()方法的代码。

Lock 接口（和ReentrantLock类）包含其他方法来获取锁的控制权，那就是tryLock()方法。这个方法与lock()方法的最大区别是，如果一 个线程调用这个方法不能获取Lock接口的控制权时，将会立即返回并且不会使这个线程进入睡眠。这个方法返回一个boolean值，true表示这个线程 获取了锁的控制权，false则表示没有。

**考虑到这个方法的结果，并采取相应的措施，这是程序员的责任。如果这个方法返回false值，预计你的程序不会执行这个临界区。如果是这样，你可能会在你的应用程序中得到错误的结果。**

ReentrantLock类也允许递归调用（锁的可重入性，译者注），当一个线程有锁的控制权并且使用递归调用，它延续了锁的控制权，所以调用lock()方法将会立即返回并且继续递归调用的执行。此外，我们也可以调用其他方法。

**你必须要非常小心使用锁来避免死锁，这种情况发生在，当两个或两个以上的线程被阻塞等待将永远不会解开的锁。**

## 使用读/写锁同步数据访问

锁所提供的最重要的改进之一就是ReadWriteLock接口和唯一 一个实现它的ReentrantReadWriteLock类。这个类提供两把锁，一把用于读操作和一把用于写操作。同时可以有多个线程执行读操作，但只有一个线程可以执行写操作。当一个线程正在执行一个写操作，不可能有任何线程执行读操作。

正如我们前面提及到的，ReentrantReadWriteLock类有两把锁，一把用于读操作，一把用于写操作。

* 用于读操作的锁，是通过在 ReadWriteLock接口中声明的readLock()方法获取的。这个锁是实现Lock接口的一个对象，所以我们可以使用lock()， unlock() 和tryLock()方法。
* 用于写操作的锁，是通过在ReadWriteLock接口中声明的writeLock()方法获取的。这个锁是实现Lock接口的一个对象，所以我们可以使用lock()， unlock() 和tryLock()方法。

确保正确的使用这些锁，使用它们与被设计的目的是一样的，这是程序猿的职责。当你获得Lock接口的读锁时，不能修改这个变量的值。否则，你可能会有数据不一致的错误。

## 修改Lock的公平性

在ReentrantLock类和 ReentrantReadWriteLock类的构造器中，允许一个名为fair的boolean类型参数，它允许你来控制这些类的行为。

* 默认值为 false，这将启用非公平模式。在这个模式中，当有多个线程正在等待一把锁（ReentrantLock或者 ReentrantReadWriteLock），这个锁必须选择它们中间的一个来获得进入临界区，选择任意一个是没有任何标准的。
* true值将开启公平 模式。在这个模式中，当有多个线程正在等待一把锁（ReentrantLock或者ReentrantReadWriteLock），这个锁必须选择它们中间的一个来获得进入临界区，它将选择等待时间最长的线程。

另外由于tryLock()方法并不会使线程进入睡眠，即使Lock接口正在被使用，这个公平属性并不会影响它的功能。

**注意**

读/写锁在它们的构造器中也有公平参数。这个参数在这种锁中的行为与本指南的解释是一样的。

## 在Lock中使用多个条件

一个锁可能伴随着多个条件。这些条件声明在Condition接口中。 这些条件的目的是允许线程拥有锁的控制并且检查条件是否为true，如果是false，那么线程将被阻塞，直到其他线程唤醒它们。Condition接口提供一种机制，阻塞一个线程和唤醒一个被阻塞的线程。

在并发编程中，生产者与消费者是经典的问题。我们有一个数据缓冲区，一个或多个数据生产者往缓冲区存储数据，一个或多个数据消费者从缓冲区中取出数据。

所有Condition对象都与锁有关，并且使用声明在Lock接口中的newCondition()方法来创建。使用condition做任何操作之前， 你必须获取与这个condition相关的锁的控制。所以，condition的操作一定是在以调用Lock对象的lock()方法为开头，以调用相同 Lock对象的unlock()方法为结尾的代码块中。

当一个线程在一个condition上调用await()方法时，它将自动释放锁的控制，所以其他线程可以获取这个锁的控制并开始执行相同操作，或者由这个个锁保护的其他临界区。

注释：当一个线程在一个condition上调用signal()或signallAll()方法，一个或者全部在这个condition上等待的线程将被唤醒。这并不能保证它们睡眠的条件现在是true，所以你必须在while循环内部调用await()方法。你不能离开这个循环，直到 condition为true。当condition为false，你必须再次调用 await()方法。

你必须十分小心 ，在使用await()和signal()方法时。如果你在condition上调用await()方法而却没有在这个condition上调用signal()方法，这个线程将永远睡眠下去。

在调用await()方法后，一个线程可以被中断的，所以当它正在睡眠时，你必须处理InterruptedException异常。

**注意**

Condition接口提供不同版本的await()方法，如下：

- await(long time, TimeUnit unit):这个线程将会一直睡眠直到：
	
	1. 它被中断
	2. 其他线程在这个condition上调用singal()或signalAll()方法
	3. 指定的时间已经过了
	4. TimeUnit类是一个枚举类型如下的常量：DAYS,HOURS, MICROSECONDS, MILLISECONDS, MINUTES, NANOSECONDS,SECONDS

- awaitUninterruptibly():这个线程将不会被中断，一直睡眠直到其他线程调用signal()或signalAll()方法
- awaitUntil(Date date):这个线程将会一直睡眠直到：
	
	1. 它被中断
	2. 其他线程在这个condition上调用singal()或signalAll()方法
	3. 指定的日期已经到了

同样可以在一个读/写锁中的ReadLock和WriteLock上使用conditions。