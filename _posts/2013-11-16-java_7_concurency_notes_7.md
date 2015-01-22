---
layout: post
title: "Java 7 并发编程学习笔记（7）-- 线程执行者（下）"
category: 技术
tags: Concurrency
---
## 执行者延迟运行一个任务

执行者框架提供ThreadPoolExecutor类，使用池中的线程来执行Callable和Runnable任务，这样可以避免所有线程的创建操作。当你提交一个任务给执行者，会根据执行者的配置尽快执行它。在有些使用情况下，当你对尽快执行任务不感觉兴趣。你可能想要在一段时间之后执行任务或周期性地执行任务。基于这些目的，执行者框架提供 ScheduledThreadPoolExecutor类。

	ScheduledThreadPoolExecutor executor=(ScheduledThreadPoolExecutor)Executors.newScheduledThreadPool(1);
	executor.schedule(task, 1, TimeUnit.SECONDS);

<!--more-->

使用schedule()方法，让执行者在一段时间后执行任务。这个方法接收3个参数，如下：

* 你想要执行的任务
* 你想要让任务在执行前等待多长时间
* 时间单位，指定为TimeUnit类的常数

ScheduledThreadPoolExecutor类的schedule()方法接收两种类型（Runnable和Callable）的任务。

尽管ScheduledThreadPoolExecutor类是ThreadPoolExecutor类的子类，因此，它继承 ThreadPoolExecutor类的所有功能，但**Java建议使用ScheduledThreadPoolExecutor仅适用于调度任务。**

可以配置ScheduledThreadPoolExecutor的行为，当你调用shutdown()方法时，并且有待处理的任务正在等待它们延迟结束。默认的行为是，不管执行者是否结束这些任务都将被执行。你可以使用ScheduledThreadPoolExecutor类的setExecuteExistingDelayedTasksAfterShutdownPolicy()方法来改变这种行为。使用false，调用 shutdown()时，待处理的任务不会被执行。

## 执行者周期性地运行一个任务

	ScheduledExecutorService executor=Executors.newScheduledThreadPool(1);
	ScheduledFuture<?> result=executor.scheduleAtFixedRate(task,1, 2, TimeUnit.SECONDS);

当你想要使用执行者框架执行一个周期性任务，你需要ScheduledExecutorService对象。Java建议使用 Executors类创建执行者，Executors类是一个执行者对象工厂。

一旦你有执行者需要执行一个周期性任务，你提交任务给该执行者。你已经使用了scheduledAtFixedRate()方法。此方法接收4个参数：

1. 你想要周期性执行的任务
2. 第一次执行任务的延迟时间
3. 两次执行之间的间隔期间
4. 第2、3个参数的时间单位。它是TimeUnit类的常量，TimeUnit类是个枚举类，有如下常量：DAYS，HOURS，MICROSECONDS， MILLISECONDS， MINUTES,，NANOSECONDS 和SECONDS。

很重要的一点需要考虑的是**两次执行之间的（间隔）期间，是这两个执行开始之间的一段时间。**如果你有一个花5秒执行的周期性任务，而你给一段3秒时间，同一时刻，你将会有两个任务在执行。

scheduleAtFixedRate() 方法返回ScheduledFuture对象，它继承Future接口，这个方法和调度任务一起协同工作。ScheduledFuture是一个参数化接口（校对注：ScheduledFuture<V>）。

ScheduledThreadPoolExecutor 提供其他方法来调度周期性任务。这就是scheduleWithFixedRate()方法。它与scheduledAtFixedRate()方法有一样的参数，但它们之间的差异值得注意。

* 在scheduledAtFixedRate()方法中，第3个参数决定`两个执行开始`的一段时间。
* 在 scheduledWithFixedRate()方法中，参数决定任务`执行结束与下次执行开始`之间的一段时间。

当你使用 shutdown()方法时，你也可以通过参数配置一个SeduledThreadPoolExecutor的行为。shutdown()方法默认的行为是，当你调用这个方法时，计划任务就结束。 你可以使用ScheduledThreadPoolExecutor类的 setContinueExistingPeriodicTasksAfterShutdownPolicy()方法设置true值改变这个行为。在调用 shutdown()方法时，周期性任务将不会结束。

## 执行者取消一个任务

**可以使用Future的cancel()方法，它允许你做取消已提交给执行者的任务。**

	Future<String> result=executor.submit(task);
	result.cancel(true);

根据cancel()方法参数和任务的状态不同，这个方法的行为将不同：

* 如果这个任务已经完成或之前的已被取消或由于其他原因不能被取消，那么这个方法将会返回false并且这个任务不会被取消。
* 如果这个任务正在等待执行者获取执行它的线程，那么这个任务将被取消而且不会开始它的执行。如果这个任务已经正在运行，则视方法的参数情况而定。 cancel()方法接收一个Boolean值参数。如果参数为true并且任务正在运行，那么这个任务将被取消。如果参数为false并且任务正在运行，那么这个任务将不会被取消。

如果使用Future对象的get()方法来控制一个已被取消的任务，这个get()方法将抛出CancellationException异常。

## 执行者控制一个任务完成

FutureTask类提供一个done()方法，允许你在执行者执行任务完成后执行一些代码。你可以用来做一些后处理操作，生成一个报告，通过e-mail发送结果，或释放一些资源。当执行的任务由FutureTask来控制完成，FutureTask会内部调用这个方法。这个方法在任务的结果设置和它的状态变成isDone状态之后被调用，不管任务是否已经被取消或正常完成。

默认情况下，这个方法是空的。你可以重写FutureTask类实现这个方法来改变这种行为。

	public class ResultTask extends FutureTask<String> {
		//...省略部分程序
		@Override
		protected void done() {
			if (isCancelled()) {
				System.out.printf("%s: Has been cancelled\n",name);
			} else {
				System.out.printf("%s: Has finished\n",name);
			}
		}
	}
	
在使用的地方

	ExecutableTask executableTask=new ExecutableTask("Task ");
	resultTask=new ResultTask(executableTask);
	executor.submit(resultTask);
	
在建立返回值和改变任务的状态为isDone状态后，done()方法被FutureTask类内部调用。你不能改变任务的结果值和它的状态，但你可以关闭任务使用的资源，写日志消息，或发送通知。

## 执行者分离任务的启动和结果的处理

**CompletionService 类有一个方法来提交任务给执行者和另一个方法来获取已完成执行的下个任务的Future对象。**

在内部实现中，它使用Executor对象执行任务。这种行为的优点是共享一个CompletionService对象，并提交任务给执行者，这样其他（对象）可以处理结果。其局限性是，第二个对象只能获取那些已经完成它们的执行的任务的Future对象，所以，这些Future对象只能获取任务的结果。

使用Executors类的newCachedThreadPool()方法创建ThreadPoolExecutor。然后，使用这个对象初始化一个CompletionService对象，因为CompletionService需要使用一个执行者来执行任务。

	ExecutorService executor=(ExecutorService)Executors.newCachedThreadPool();
	CompletionService<String> service=new ExecutorCompletionService<>(executor);

利用CompletionService执行一个任务，你需要使用submit()方法，如在ReportRequest类中。

当其中一个任务被执行，CompletionService完成这个任务的执行时，这个CompletionService在一个队列中存储Future对象来控制它的执行。

	Future<String> result=service.poll(20, TimeUnit.SECONDS);

poll()方法用来查看这个列队，如果有任何任务执行完成，那么返回列队的第一个元素，它是一个已完成任务的Future对象。当poll()方法返回一个Future对象时，它将这个Future对象从队列中删除。这种情况下，你可以传两个属性给那个方法，表明你想要等任务结果的时间，以防队列中的已完成任务的结果是空的。

**注意**

CompletionService类可以执行Callable和Runnable任务。在这个示例中，你已经使用Callable，但你同样可以提交Runnable对象。由于Runnable对象不会产生结果，CompletionService类的理念不适用于这些情况。

这个类同样提供其他两个方法，用来获取已完成任务的Future对象。这两个方法如下：

* poll()：不带参数版本的poll()方法，检查是否有任何Future对象在队列中。如果列队是空的，它立即返回null。否则，它返回第一个元素，并从列队中删除它。
* take()：这个方法，不带参数。检查是否有任何Future对象在队列中。如果队列是空的，它阻塞线程直到队列有一个元素。当队列有元素，它返回第一元素，并从列队中删除它。


## 执行者控制被拒绝的任务

当你想要结束执行者的执行，你使用shutdown()方法来表明它的结束。执行者等待正在运行或等待它的执行的任务的结束，然后结束它们的执行。

如果你在shutdown()方法和执行者结束之间，提交任务给执行者，这个任务将被拒绝，因为执行者不再接收新的任务。

	public class RejectedTaskController implements RejectedExecutionHandler {

		@Override
		public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) {
			System.out.printf("RejectedTaskController: The task %s has been rejected\n",r.toString());
			System.out.printf("RejectedTaskController: %s\n",executor.toString());
			System.out.printf("RejectedTaskController: Terminating: %s\n",executor.isTerminating());
			System.out.printf("RejectedTaksController: Terminated: %s\n",executor.isTerminated());
		}

	}

为了管理执行者控制拒绝任务，你应该实现RejectedExecutionHandler接口。该接口有带有两个参数的方法rejectedExecution()：

* Runnable对象，存储被拒绝的任务
* Executor对象，存储拒绝任务的执行者

每个被执行者拒绝的任务都会调用这个方法。你必须使用Executor类的setRejectedExecutionHandler()方法设置拒绝任务的处理器。

**注意**

当执行者接收任务时，会检查shutdown()是否已经被调用了。如果被调用了，它拒绝这个任务。首先，它查找 setRejectedExecutionHandler()设置的处理器。如果有一个（处理器），它调用那个类的 rejectedExecution()方法，否则，它将抛RejectedExecutionExeption异常。这是一个运行时异常，所以你不需要 用catch语句来控制它。