---
layout: post
title: "Java 7 并发编程学习笔记（8）-- Fork/Join框架"
category: 技术
tags: Concurrency
---

Java 7包括一个面向特定问题的ExecutorService接口的额外实现，它就是Fork/Join框架。

这个框架被设计用来解决可以使用分而治之技术将任务分解成更小的问题。在一个任务中，检查你想要解决问题的大小，如果它大于一个既定的大小，把它分解成更小的任务，然后用这个框架来执行。如果问题的大小是小于既定的大小，你直接在任务中解决这问题。它返回一个可选地结果。

这个框架基于以下两种操作：

* fork操作：当你把任务分成更小的任务和使用这个框架执行它们。
* join操作：当一个任务等待它创建的任务的结束。

Fork/Join 和Executor框架主要的区别是work-stealing算法。不像Executor框架，当一个任务正在等待它使用join操作创建的子任务的结 束时，执行这个任务的线程（工作线程）查找其他未被执行的任务并开始它的执行。通过这种方式，线程充分利用它们的运行时间，从而提高了应用程序的性能。

<!--more-->

为实现这个目标，Fork/Join框架执行的任务有以下局限性：

* 任务只能使用fork()和join()操作，作为同步机制。如果使用其他同步机制，工作线程不能执行其他任务，当它们在同步操作时。比如，在Fork/Join框架中，你使任务进入睡眠，正在执行这个任务的工作线程将不会执行其他任务，在这睡眠期间内。
* 任务不应该执行I/O操作，如读或写数据文件。
* 任务不能抛出检查异常，它必须包括必要的代码来处理它们。

Fork/Join框架的核心是由以下两个类：

1. ForkJoinPool：它实现ExecutorService接口和work-stealing算法。它管理工作线程和提供关于任务的状态和它们执行的信息。
2. ForkJoinTask： 它是将在ForkJoinPool中执行的任务的基类。它提供在任务中执行fork()和join()操作的机制，并且这两个方法控制任务的状态。通常， 为了实现你的Fork/Join任务，你将实现两个子类的子类的类：RecursiveAction对于没有返回结果的任务和RecursiveTask 对于返回结果的任务。

## 创建一个Fork/Join池


	// 通用 divide-and-conquer 并行算法的伪代码。
	Result solve(Problem problem) { 
    	if (problem.size < SEQUENTIAL_THRESHOLD)
        	return solveSequentially(problem);
    	else {
        	Result left, right;
        	INVOKE-IN-PARALLEL { 
            	left = solve(extractLeftHalf(problem));
            	right = solve(extractRightHalf(problem));
        	}
        	return combine(left, right);
    	}
	}
	
示例：

	public class Task extends RecursiveAction {
	//省略部分代码
		@Override
		protected void compute() {
			if (last-first<10) {
				updatePrices();
			} else {
				int middle=(last+first)/2;
				System.out.printf("Task: Pending tasks: %s\n",getQueuedTaskCount());
				Task t1=new Task(products, first,middle+1, increment);
				Task t2=new Task(products, middle+1,last, increment);
				invokeAll(t1, t2);	
			}
		}
	}

由于Task类没有返回结果，所以它继承RecursiveAction类。	
在使用的地方：

	Task task=new Task(products, 0, products.size(), increment);
	ForkJoinPool pool=new ForkJoinPool();
	pool.execute(task);

为了创建ForkJoinPool对象，使用了无参构造器，它会以默认的配置来执行。它创建一个线程数等于计算机处理器数的池。当ForkJoinPool对象被创建时，这些线程被创建并且在池中等待，直到有任务到达让它们执行。

调用invokeAll()方法，执行每个任务所创建的子任务。这是一个同步调用，这个任务在继续（可能完成）它的执行之前，必须等待子任务的结束。当任务正在等待它的子任务（结束）时，正在执行它的工作线程执行其他正在等待的任务。在这种行为下，Fork/Join框架比Runnable和Callable对象本身提供一种更高效的任务管理。

> ForkJoinTask类的invokeAll()方法是执行者（Executor）和Fork/Join框架的一个主要区别。在执行者框架中，所有任务被提交给执行者，而在这种情况下，这些任务包括执行和控制这些任务的方法都在池内。

**ForkJoinPool类提供其他的方法，用来执行一个任务。这些方法如下：**

* execute (Runnable task)：这是在这个示例中，使用的execute()方法的另一个版本。在这种情况下，你可以提交一个Runnable对象给ForkJoinPool类。注意：ForkJoinPool类不会对Runnable对象使用work-stealing算法。它（work-stealing算法）只用于ForkJoinTask对象。
* invoke(ForkJoinTask<T> task)：当execute()方法使用一个异步调用ForkJoinPool类，正如你在本示例中所学的，invoke()方法使用同步调用ForkJoinPool类。这个调用不会（立即）返回，直到传递的参数任务完成它的执行。
* 你也可以使用在ExecutorService接口的invokeAll()和invokeAny()方法。这些方法接收一个Callable对象作为参数。ForkJoinPool类不会对Callable对象使用work-stealing算法，所以你最好使用执行者去执行它们。

**ForkJoinTask类同样提供在示例中使用的invokeAll()的其他版本。这些版本如下：**

* invokeAll(ForkJoinTask<?>… tasks)：这个版本的方法使用一个可变参数列表。你可以传入许多你想要执行的ForkJoinTask对象作为参数。
* invokeAll(Collection<T> tasks)：这个版本的方法接收一个泛型类型T对象的集合（如：一个ArrayList对象，一个LinkedList对象或者一个TreeSet对象）。这个泛型类型T必须是ForkJoinTask类或它的子类。

虽然ForkJoinPool类被设计成用来执行一个ForkJoinTask，你也可以直接执行Runnable和Callable对象。你也可以使用ForkJoinTask类的adapt()方法来执行任务，它接收一个Callable对象或Runnable对象（作为参数）并返回一个ForkJoinTask对象。

## 加入任务的结果

在任务中，必须使用Java API方法推荐的结构：

	If (problem size < size){
		tasks=Divide(task);
		execute(tasks);
		groupResults()
		return result;
	} else {
		resolve problem;
		return result;
	}
	
示例代码：

	public class DocumentTask extends RecursiveTask<Integer> {
	//省略部分代码
		@Override
		protected Integer compute() {
			Integer result=null;
			if (end-start<10){
				result=processLines(document, start,end,word);
			} else {
				int mid=(start+end)/2;
				DocumentTask task1=new DocumentTask(document,start,mid,word);
				DocumentTask task2=new DocumentTask(document,mid,end,word);
				invokeAll(task1,task2);
				try {
					result=groupResults(task1.get(),task2.get());
				} catch (InterruptedException | ExecutionException e) {
					e.printStackTrace();
				}
			}
			return result;
		}
	}
	
为了获取Task返回的结果，使用了get()方法 。这个方法是在Future接口中声明的，由RecursiveTask类实现的。

**注意**

ForkJoinTask类提供其他的方法来完成一个任务的执行，并返回一个结果，这就是complete()方法。这个方法接收一个RecursiveTask类的参数化类型的对象，并且当join()方法被调用时，将这个对象作为任务的结果返回。 它被推荐使用在：提供异步任务结果。

由于RecursiveTask类实现了Future接口，get()方法其他版本如下：

* get(long timeout, TimeUnit unit)：这个版本的get()方法，如果任务的结果不可用，在指定的时间内等待它。如果超时并且结果不可用，那么这个方法返回null值。TimeUnit类是一个枚举类，它有以下常量：DAYS, HOURS，MICROSECONDS，MILLISECONDS， MINUTES， NANOSECONDS和SECONDS。

## 异步运行任务

当你在ForkJoinPool中执行ForkJoinTask时，你可以使用同步或异步方式来实现。当你使用同步方式时，提交任务给池的方法直到提交的任务完成它的执行，才会返回结果。当你使用异步方式时，提交任务给执行者的方法将立即返回，所以这个任务可以继续执行。

示例代码：

	public class FolderProcessor extends RecursiveTask<List<String>> {
		private static final long serialVersionUID = 1L;
		private String path;
		private String extension;
		public FolderProcessor (String path, String extension) {
			this.path=path;
			this.extension=extension;
		}
		
		@Override
		protected List<String> compute() {
			List<String> list=new ArrayList<>();
			List<FolderProcessor> tasks=new ArrayList<>();
			
			File file=new File(path);
			File content[] = file.listFiles();
			if (content != null) {
				for (int i = 0; i < content.length; i++) {
					if (content[i].isDirectory()) {
						// If is a directory, process it. Execute a new Task
						FolderProcessor task=new FolderProcessor(content[i].getAbsolutePath(), extension);
						task.fork();
						tasks.add(task);
					} else {
						// If is a file, process it. Compare the extension of the file and the extension
						// it's looking for
						if (checkFile(content[i].getName())){
							list.add(content[i].getAbsolutePath());
						}
					}
				}
				
				// If the number of tasks thrown by this tasks is bigger than 50, we write a message
				if (tasks.size()>50) {
					System.out.printf("%s: %d tasks ran.\n",file.getAbsolutePath(),tasks.size());
				}
				
				// Include the results of the tasks
				addResultsFromTasks(list,tasks);
			}
			return list;
		}

		private void addResultsFromTasks(List<String> list,
				List<FolderProcessor> tasks) {
			for (FolderProcessor item: tasks) {
				list.addAll(item.join());
			}
		}

		private boolean checkFile(String name) {
			if (name.endsWith(extension)) {
				return true;
			}
			return false;
		}
	}

上面使用了join()方法来等待任务的结束，并获得它们的结果。对于此，你也可以使用get()方法的两个版本之一来实现：

* get()：这个版本的get()方法，如果ForkJoinTask已经结束它的执行，则返回compute()方法的返回值，否则，等待直到它完成。
* get(long timeout, TimeUnit unit)：这个版本的get()方法，如果任务的结果不可用，则在指定的时间内等待它。如果超时并且任务的结果仍不可用，这个方法返回null值。TimeUnit类是一个枚举类，包含以下常量：DAYS，HOURS，MICROSECONDS， MILLISECONDS，MINUTES， NANOSECONDS 和 SECONDS。

**get()和join()有两个主要的区别：**

* join()方法不能被中断。如果你中断调用join()方法的线程，这个方法将抛出InterruptedException异常。
* 如果任务抛出任何未受检异常，get()方法将返回一个ExecutionException异常，而join()方法将返回一个RuntimeException异常。

## 在任务中抛出异常

在Java中有两种异常：

* 已检查异常（Checked exceptions）：这些异常必须在一个方法的throws从句中指定或在内部捕捉它们。比如：IOException或ClassNotFoundException。
* 未检查异常（Unchecked exceptions）：这些异常不必指定或捕捉。比如：NumberFormatException。

在ForkJoinTask类的`compute()方法中，你不能抛出任何已检查异常`，因为在这个方法的实现中，它没有包含任何抛出（异常）声明。你必须包含必要的代码来处理异常。但是，你可以抛出（或者它可以被任何方法或使用内部方法的对象抛出）一个未检查异常。ForkJoinTask和ForkJoinPool类的行为与你可能的期望不同。程序不会结束执行，并且你将不会在控制台看到任何关于异常的信息。它只是被吞没，好像它没抛出（异常）。

使用awaitTermination()方法等待任务的结束。

	pool.awaitTermination(1, TimeUnit.DAYS);
	
使用isCompletedAbnormally()方法，检查这个任务或它的子任务是否已经抛出异常。

	if (task.isCompletedAbnormally()) {
		System.out.printf("Main: An exception has ocurred\n");
		System.out.printf("Main: %s\n",task.getException());
	}
	
如果不是抛出异常，你也可以使用ForkJoinTask类的completeExceptionally()方法实现上边的效果。代码如下：

	Exception e=new Exception("This task throws an Exception: "+"Task from "+start+" to "+end);
	completeExceptionally(e);
	
## 取消任务

当你在一个ForkJoinPool类中执行ForkJoinTask对象，`在它们开始执行之前，你可以取消执行它们。`ForkJoinTask类提供cancel()方法用于这个目的。当你想要取消一个任务时，有一些点你必须考虑一下，这些点如下：

* ForkJoinPool类并没有提供任何方法来取消正在池中运行或等待的所有任务。
* 当你取消一个任务时，你不能取消一个已经执行的任务。

ForkJoinTask提供cancel()方法，允许你取消一个还未执行的任务。这是一个非常重要的点。如果任务已经开始它的执行，那么调用cancel()方法对它没有影响。这个方法接收一个Boolean值，名为mayInterruptIfRunning的参数。这个名字可能让你觉得，如果你传入一个true值给这个方法，这个任务将被取消，即使它正在运行。

Java API文档指出，在ForkJoinTask类的默认实现中，这个属性不起作用。任务只能在它们还未开始执行时被取消。一个任务的取消不会影响到已经提到到池的（其他）任务。它们继续它们的执行。 Fork/Join框架的一个局限性是，它不允许取消在ForkJoinPool中的所有任务。
