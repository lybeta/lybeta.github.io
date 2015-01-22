---
layout: post
title: "Java 7 并发编程学习笔记（4）-- 线程同步工具（上）"
category: 技术
tags: Concurrency
---
## 控制并发访问资源

Semaphore是一个控制访问多个共享资源的计数器。

Semaphore的内容是由Edsger Dijkstra引入并在 THEOS操作系统上第一次使用。

当一个线程想要访问某个共享资源，首先，它必须获得semaphore。如果semaphore的内部计数器的值大于0，那么semaphore减少计数器的值并允许访问共享的资源。计数器的值大于0表示，有可以自由使用的资源，所以线程可以访问并使用它们。

另一种情况，如果semaphore的计数器的值等于0，那么semaphore让线程进入休眠状态一直到计数器大于0。计数器的值等于0表示全部的共享资源都正被线程们使用，所以此线程想要访问就必须等到某个资源成为自由的。

当线程使用完共享资源时，他必须释放semaphore使其他线程可以访问共享资源。这个操作会增加semaphore的内部计数器的值。

<!--more-->

当你使用semaphore来实现critical section，并保护共享资源的访问时：

	首先， 你要调用acquire()方法获得semaphore。
	然后， 对共享资源做出必要的操作。
	最后， 调用release()方法来释放semaphore。
	
Semaphore类有另2个版本的 acquire() 方法：

* acquireUninterruptibly()：acquire()方法是当semaphore的内部计数器的值为0时，阻塞线程直到semaphore被释放。在阻塞期间，线程可能会被中断，然后此方法抛出InterruptedException异常。而此版本的acquire方法会忽略线程的中断而且不会抛出任何异常。
* tryAcquire()：此方法会尝试获取semaphore。如果成功，返回true。如果不成功，返回false值，并不会被阻塞和等待semaphore的释放。接下来是你的任务用返回的值执行正确的行动。

>**Semaphores的公平性**

> * fairness的内容是指全java语言的所有类中，那些可以阻塞多个线程并等待同步资源释放的类（例如，semaphore)。**默认情况下是非公平模式。**在这个模式中，当同步资源释放，就会从等待的线程中任意选择一个获得资源，但是这种选择没有任何标准。而公平模式可以改变这个行为并强制选择等待最久时间的线程。

> * 随着其他类的出现，Semaphore类的构造函数容许第二个参数。这个参数必需是 Boolean 值。如果你给的是 false 值，那么创建的semaphore就会在非公平模式下运行。如果你不使用这个参数，是跟给false值一样的结果。如果你给的是true值，那么你创建的semaphore就会在公平模式下运行。

## 控制并发访问多个资源

使用示例如下，注意Semaphore与Lock的搭配使用：

	public class PrintQueue {
		private Semaphore semaphore;			
		private boolean freePrinters[];			
		private Lock lockPrinters;
				
		public PrintQueue(){
			semaphore=new Semaphore(3);
			freePrinters=new boolean[3];
			for (int i=0; i<3; i++){
				freePrinters[i]=true;
			}
			lockPrinters=new ReentrantLock();
		}
		
		public void printJob (Object document){
			try {
				// Get access to the semaphore. If there is one or more printers free,
				// it will get the access to one of the printers
				semaphore.acquire();
				
				// Get the number of the free printer
				int assignedPrinter=getPrinter();
				
				Long duration=(long)(Math.random()*10);
				System.out.printf("%s: PrintQueue: Printing a Job in Printer %d during %d seconds\n",Thread.currentThread().getName(),assignedPrinter,duration);
				TimeUnit.SECONDS.sleep(duration);
				
				// Free the printer
				freePrinters[assignedPrinter]=true;
			} catch (InterruptedException e) {
				e.printStackTrace();
			} finally {
				// Free the semaphore
				semaphore.release();			
			}
		}

		private int getPrinter() {
			int ret=-1;
			
			try {
				// Get the access to the array
				lockPrinters.lock();
				// Look for the first free printer
				for (int i=0; i<freePrinters.length; i++) {
					if (freePrinters[i]){
						ret=i;
						freePrinters[i]=false;
						break;
					}
				}
			} catch (Exception e) {
				e.printStackTrace();
			} finally {
				// Free the access to the array
				lockPrinters.unlock();
			}
			return ret;
		}

	}

**注意**

acquire(), acquireUninterruptibly(), tryAcquire(),和release()方法有一个外加的包含一个int参数的版本。这个参数表示线程想要获取或者释放semaphore的许可数。也可以这样说，这个线程想要删除或者添加到semaphore的内部计数器的单位数量。

## 等待多个并发事件完成

Java并发API提供CountDownLatch类，它允许1个或者多个线程一直等待，直到一组操作执行完成。

它初始一个整数值，此值是线程将要等待的操作数。当某个线程为了想要执行这些操作而等待时，它要使用 await()方法。此方法让线程进入休眠直到操作完成。当某个操作结束，它使用countDown() 方法来减少CountDownLatch类的内部计数器。当计数器到达0时，这个类会唤醒全部使用await() 方法休眠的线程们。

**CountDownLatch类有3个基本元素：**

1. 初始值决定CountDownLatch类需要等待的事件的数量。
2. await() 方法, 被等待全部事件终结的线程调用。
3. countDown() 方法，事件在结束执行后调用。

当创建 CountDownLatch 对象时，对象使用构造函数的参数来初始化内部计数器。每次调用 countDown() 方法, CountDownLatch 对象内部计数器减一。当内部计数器达到0时， CountDownLatch 对象唤醒全部使用 await() 方法睡眠的线程们。

**不可能重新初始化或者修改CountDownLatch对象的内部计数器的值。**一旦计数器的值初始后，唯一可以修改它的方法就是之前用的 countDown() 方法。当计数器到达0时， 全部调用 await() 方法会立刻返回，接下来任何countDown() 方法的调用都将不会造成任何影响。

>此方法与其他同步方法有这些不同：

>CountDownLatch 机制不是用来保护共享资源或者临界区。它是用来同步一个或者多个执行多个任务的线程。它只能使用一次。像之前解说的，一旦CountDownLatch的计数器到达0，任何对它的方法的调用都是无效的。如果你想再次同步，你必须创建新的对象。

**注意**

CountDownLatch 类有另一种版本的 await() 方法，它是：

* await(long time, TimeUnit unit): 此方法会休眠直到被中断； CountDownLatch 内部计数器到达0或者特定的时间过去了。TimeUnit 类包含了:DAYS, HOURS, MICROSECONDS, MILLISECONDS, MINUTES, NANOSECONDS, 和 SECONDS.

## 在同一个点同步任务

Java 并发 API 提供了可以允许2个或多个线程在在一个确定点的同步应用，它是 CyclicBarrier 类。此类与 CountDownLatch 类相似，但是它有一些特殊性让它成为更强大的类。

CyclicBarrier 类有一个整数初始值，此值表示将在同一点同步的线程数量。当其中一个线程到达确定点，它会调用await() 方法来等待其他线程。当线程调用这个方法，CyclicBarrier阻塞线程进入休眠直到其他线程到达。当最后一个线程调用CyclicBarrier 类的await() 方法，它唤醒所有等待的线程并继续执行它们的任务。

CyclicBarrier 类有个有趣的优势是，你可以传递一个外加的 Runnable 对象作为初始参数，并且当全部线程都到达同一个点时，CyclicBarrier类 会把这个对象当做线程来执行。此特点让这个类在使用 divide 和 conquer 编程技术时，可以充分发挥任务的并行性。

CyclicBarrier 类有一个内部计数器控制到达同步点的线程数量。每次线程到达同步点，它调用 await() 方法告知 CyclicBarrier 对象到达同步点了。CyclicBarrier 把线程放入睡眠状态直到全部的线程都到达他们的同步点。

**当全部的线程都到达他们的同步点，CyclicBarrier 对象叫醒全部正在 await() 方法中等待的线程们，然后，选择性的，为CyclicBarrier的构造函数传递的 Runnable 对象创建新的线程执行外加任务。**

>CyclicBarrier 类有另一个版本的 await() 方法:

> * await(long time, TimeUnit unit): 线程会一直休眠直到被中断；内部计数器到达0，或者特定的时间过去了。TimeUnit类有多种常量： DAYS, HOURS, MICROSECONDS, MILLISECONDS, MINUTES, NANOSECONDS, and SECONDS.

>此类也提供了 getNumberWaiting() 方法，返回被 await() 方法阻塞的线程数；

>还有 getParties() 方法，返回将与CyclicBarrier同步的任务数。

**重置 CyclicBarrier 对象**

CyclicBarrier 类与CountDownLatch有一些共同点，但是也有一些不同。最主要的不同是，**CyclicBarrier对象可以重置到它的初始状态，重新分配新的值给内部计数器，即使它已经被初始过了。**

可以使用 CyclicBarrier的reset() 方法来进行重置操作。当这个方法被调用后，全部的正在await() 方法里等待的线程接收到一个 BrokenBarrierException 异常。此异常在例子中已经用打印stack trace处理了，但是在一个更复制的应用，它可以执行一些其他操作，例如重新开始执行或者在中断点恢复操作。

**破坏 CyclicBarrier 对象**

CyclicBarrier 对象可能处于一个特殊的状态，称为 broken。当多个线程正在 await() 方法中等待时，其中一个被中断了，此线程会收到 InterruptedException 异常，但是其他正在等待的线程将收到 BrokenBarrierException 异常，并且 CyclicBarrier 会被置于broken 状态中。

CyclicBarrier 类提供了isBroken() 方法，如果对象在 broken 状态，返回true，否则返回false。

