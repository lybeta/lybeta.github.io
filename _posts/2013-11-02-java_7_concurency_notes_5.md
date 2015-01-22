---
layout: post
title: "Java 7 并发编程学习笔记（5）-- 线程同步工具（下）"
category: 技术
tags: Concurrency
---
## 运行阶段性并发任务

Java 并发 API 提供的一个非常复杂且强大的功能是，能够使用Phaser类运行阶段性的并发任务。当某些并发任务是分成多个步骤来执行时，那么此机制是非常有用的。Phaser类提供的机制是在每个步骤的结尾同步线程，所以没有线程开始第二步直到全部线程完成第一个步骤。

相对于其他同步应用，我们必须初始化Plaser类与这次同步操作有关的任务数，我们可以通过增加或者减少来不断的改变这个数。

<!--more-->

	public class FileSearch implements Runnable {
		private String initPath;				
		private String end;		
		private List<String> results;
		
		/**
		 * Phaser to control the execution of the FileSearch objects. Their execution will be divided
		 * in three phases
		 *  1st: Look in the folder and its subfolders for the files with the extension
		 *  2nd: Filter the results. We only want the files modified today
		 *  3rd: Print the results
		 */	
		private Phaser phaser;
		
		public FileSearch(String initPath, String end, Phaser phaser) {
			this.initPath = initPath;
			this.end = end;
			this.phaser=phaser;
			results=new ArrayList<>();
		}

		@Override
		public void run() {
			
			// Waits for the creation of all the FileSearch objects
			phaser.arriveAndAwaitAdvance();
			
			System.out.printf("%s: Starting.\n",Thread.currentThread().getName());
			
			// 1st Phase: Look for the files
			File file = new File(initPath);
			if (file.isDirectory()) {
				directoryProcess(file);
			}
			
			// If no results, deregister in the phaser and ends
			if (!checkResults()){
				return;
			}
			
			// 2nd Phase: Filter the results
			filterResults();
			
			// If no results after the filter, deregister in the phaser and ends
			if (!checkResults()){
				return;
			}
			
			// 3rd Phase: Show info
			showInfo();
			phaser.arriveAndDeregister();
			System.out.printf("%s: Work completed.\n",Thread.currentThread().getName());

		}

		private void showInfo() {
			for (int i=0; i<results.size(); i++){
				File file=new File(results.get(i));
				System.out.printf("%s: %s\n",Thread.currentThread().getName(),file.getAbsolutePath());
			}
			// Waits for the end of all the FileSearch threads that are registered in the phaser
			phaser.arriveAndAwaitAdvance();
		}

		private boolean checkResults() {
			if (results.isEmpty()) {
				System.out.printf("%s: Phase %d: 0 results.\n",Thread.currentThread().getName(),phaser.getPhase());
				System.out.printf("%s: Phase %d: End.\n",Thread.currentThread().getName(),phaser.getPhase());
				// No results. Phase is completed but no more work to do. Deregister for the phaser
				phaser.arriveAndDeregister();
				return false;
			} else {
				// There are results. Phase is completed. Wait to continue with the next phase
				System.out.printf("%s: Phase %d: %d results.\n",Thread.currentThread().getName(),phaser.getPhase(),results.size());
				phaser.arriveAndAwaitAdvance();
				return true;
			}		
		}

		private void filterResults() {
			List<String> newResults=new ArrayList<>();
			long actualDate=new Date().getTime();
			for (int i=0; i<results.size(); i++){
				File file=new File(results.get(i));
				long fileDate=file.lastModified();
				
				if (actualDate-fileDate<TimeUnit.MILLISECONDS.convert(1,TimeUnit.DAYS)){
					newResults.add(results.get(i));
				}
			}
			results=newResults;
		}

		private void directoryProcess(File file) {

			// Get the content of the directory
			File list[] = file.listFiles();
			if (list != null) {
				for (int i = 0; i < list.length; i++) {
					if (list[i].isDirectory()) {
						// If is a directory, process it
						directoryProcess(list[i]);
					} else {
						// If is a file, process it
						fileProcess(list[i]);
					}
				}
			}
		}

		private void fileProcess(File file) {
			if (file.getName().endsWith(end)) {
				results.add(file.getAbsolutePath());
			}
		}

	}
	
	public class Main {

		public static void main(String[] args) {
			
			// Creates a Phaser with three participants
			Phaser phaser=new Phaser(3);
			
			// Creates 3 FileSearch objects. Each of them search in different directory
			FileSearch system=new FileSearch("C:\\Windows", "log", phaser);
			FileSearch apps=new FileSearch("C:\\Program Files","log",phaser);
			FileSearch documents=new FileSearch("C:\\Documents And Settings","log",phaser);
			
			// Creates a thread to run the system FileSearch and starts it
			Thread systemThread=new Thread(system,"System");
			systemThread.start();
			
			// Creates a thread to run the apps FileSearch and starts it
			Thread appsThread=new Thread(apps,"Apps");
			appsThread.start();
			
			// Creates a thread to run the documents  FileSearch and starts it
			Thread documentsThread=new Thread(documents,"Documents");
			documentsThread.start();
			try {
				systemThread.join();
				appsThread.join();
				documentsThread.join();
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
			
			System.out.printf("Terminated: %s\n",phaser.isTerminated());

		}

	}
	
**Phaser 对象可能是在这2中状态：**

1. Active: 当 Phaser 接受新的参与者注册，它进入这个状态，并且在每个phase的末端同步。 在此状态，Phaser像在这个指南里解释的那样工作。此状态不在Java 并发 API中。
2. Termination: 默认状态，当Phaser里全部的参与者都取消注册，它进入这个状态，所以这时 Phaser 有0个参与者。更具体的说，当onAdvance() 方法返回真值时，Phaser 是在这个状态里。如果你覆盖那个方法,你可以改变它的默认行为。当 Phaser 在这个状态，同步方法 arriveAndAwaitAdvance()会 立刻返回，不会做任何同步。

Phaser 类的一个显著特点是你不需要控制任何与phaser相关的方法的异常。不像其他同步应用，线程们在phaser休眠不会响应任何中断也**不会抛出 InterruptedException 异常。**

**The Phaser类还提供了其他相关方法来改变phase。他们是：**

* arrive(): 此方法示意phaser某个参与者已经结束actual phase了，但是他应该等待其他的参与者才能继续执行。小心使用此法，因为它并不能与其他线程同步。
* awaitAdvance(int phase): 如果我们传递的参数值等于phaser的actual phase，此方法让当前线程进入睡眠直到phaser的全部参与者结束当前的phase。如果参数值与phaser 的 actual phase不等，那么立刻返回。
* awaitAdvanceInterruptibly(int phaser): 此方法等同与之前的方法，只是在线程正在此方法中休眠而被中断时候，它会抛出InterruptedException 异常。

**Phaser的参与者的注册**

当你创建一个 Phaser 对象,你表明了参与者的数量。但是Phaser类还有2种方法来增加参与者的数量。他们是：

* register(): 此方法为Phaser添加一个新的参与者。这个新加入者会被认为是还未到达 actual phase.
* bulkRegister(int Parties): 此方法为Phaser添加一个特定数量的参与者。这些新加入的参与都会被认为是还未到达 actual phase.

Phaser类提供的唯一一个减少参与者数量的方法是arriveAndDeregister() 方法，它通知phaser线程已经结束了actual phase,而且他不想继续phased的操作了。

**强制终止 Phaser**

当phaser有0个参与者，它进入一个称为Termination的状态。Phaser 类提供 forceTermination() 来改变phaser的状态，让它直接进入Termination 状态，不在乎已经在phaser中注册的参与者的数量。此机制可能会很有用在一个参与者出现异常的情况下来强制结束phaser.

当phaser在 Termination 状态， awaitAdvance() 和 arriveAndAwaitAdvance() 方法立刻返回一个负值，而不是一般情况下的正值如果你知道你的phaser可能终止了，那么你可以用这些方法来确认他是否真的终止了。

## 控制并发阶段性任务的改变

Phaser 类提供每次phaser改变阶段都会执行的方法。它是 onAdvance() 方法。它接收2个参数：当前阶段数和注册的参与者数；它返回 Boolean 值，如果phaser继续它的执行，则为 false；否则为真，即phaser结束运行并进入 termination 状态。

如果注册参与者为0，此方法的默认的实现值为真，要不然就是false。如果你扩展Phaser类并覆盖此方法，那么你可以修改它的行为。通常，当你要从一个phase到另一个，来执行一些行动时，你会对这么做感兴趣的。

在这里，你将学习如何控制phaser的 phase的改变，通过实现自定义的 Phaser类并覆盖 onAdvance() 方法来执行一些每个phase 都会改变的行动。如下是一个模拟测验，有些学生要完成他们的练习。全部的学生都必须完成同一个练习才能继续下一个练习。

	public class MyPhaser extends Phaser {

		@Override
		protected boolean onAdvance(int phase, int registeredParties) {
			switch (phase) {
			case 0:
				return studentsArrived();
			case 1:
				return finishFirstExercise();
			case 2:
				return finishSecondExercise();
			case 3:
				return finishExam();
			default:
				return true;
			}
		}
		//省略部分方法实现
	}
	
	public class Student implements Runnable {

		private Phaser phaser;
		
			public Student(Phaser phaser) {
			this.phaser=phaser;
		}
		
		public void run() {
			System.out.printf("%s: Has arrived to do the exam. %s\n",Thread.currentThread().getName(),new Date());
			phaser.arriveAndAwaitAdvance();
			System.out.printf("%s: Is going to do the first exercise. %s\n",Thread.currentThread().getName(),new Date());
			doExercise1();
			System.out.printf("%s: Has done the first exercise. %s\n",Thread.currentThread().getName(),new Date());
			phaser.arriveAndAwaitAdvance();
			System.out.printf("%s: Is going to do the second exercise. %s\n",Thread.currentThread().getName(),new Date());
			doExercise2();
			System.out.printf("%s: Has done the second exercise. %s\n",Thread.currentThread().getName(),new Date());
			phaser.arriveAndAwaitAdvance();
			System.out.printf("%s: Is going to do the third exercise. %s\n",Thread.currentThread().getName(),new Date());
			doExercise3();
			System.out.printf("%s: Has finished the exam. %s\n",Thread.currentThread().getName(),new Date());
			phaser.arriveAndAwaitAdvance();
		}
		//省略部分方法实现
	}
	
	public class Main {

		public static void main(String[] args) {
			
			// Creates the Phaser
			MyPhaser phaser=new MyPhaser();
			
			// Creates 5 students and register them in the phaser
			Student students[]=new Student[5];
			for (int i=0; i<students.length; i++){
				students[i]=new Student(phaser);
				phaser.register();
			}
			
			// Create 5 threads for the students and start them
			Thread threads[]=new Thread[students.length];
			for (int i=0; i<students.length; i++) {
				threads[i]=new Thread(students[i],"Student "+i);
				threads[i].start();
			}
			
			// Wait for the finalization of the threads
			for (int i=0; i<threads.length; i++) {
				try {
					threads[i].join();
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
			
			// Check that the Phaser is in the Terminated state
			System.out.printf("Main: The phaser has finished: %s.\n",phaser.isTerminated());
			
		}

	}
	
## 在并发任务间交换数据

Java 并发 API 提供了一种允许2个并发任务间相互交换数据的同步应用，Exchanger 类。

更具体的说，Exchanger 类允许在2个线程间定义同步点，当2个线程到达这个点，他们相互交换数据类型，使用第一个线程的数据类型变成第二个的，然后第二个线程的数据类型变成第一个的。

**第一个调用 exchange()方法会进入休眠直到其他线程达到。**

**Exchanger 类有另外一个版本的exchange方法：**

* exchange(V data, long time, TimeUnit unit)：V是声明Phaser的参数种类(例子里是 List)。 此线程会休眠直到另一个线程到达并中断它，或者特定的时间过去了。TimeUnit类有多种常量：DAYS, HOURS, MICROSECONDS, MILLISECONDS, MINUTES, NANOSECONDS, 和 SECONDS。