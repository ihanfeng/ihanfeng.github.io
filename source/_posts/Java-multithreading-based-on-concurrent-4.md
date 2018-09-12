---
title: 【Java并发】之线程控制(4)
tags:
  - JAVA
  - JAVA并发
  - 多线程
categories:
  - JAVA
  - JAVA并发
abbrlink: 40961
date: 2017-10-31 21:53:48
---
# 摘要

Java的线程支持提供了一些便捷的工具方法，通过这些便捷的工具方法可以很好地控制线程的执行

1. join线程控制，让一个线程等待另一个线程完成的方法

2. 后台线程，又称为守护线程或精灵线程。它的任务是为其他的线程提供服务，如果所有的前台线程都死亡，后台线程会自动死亡

3. 线程睡眠sleep，让当前正在执行的线程暂停一段时，并进入阻塞状态

4. 线程让步yield，让当前正在执行的线程暂停，但它不会阻塞该线程，它只是将该线程转入就绪状态

<!-- more -->

# join线程

Thread提供了让一个线程等待另一个线程完成的方法join()方法。当在某个程序执行流中调用其他线程的join()方法时，调用线程将被阻塞，直到被join()方法加入的join线程执行完为止。join()方法通常由使用线程的程序调用，以将大问题划分成许多小问题，每个小问题分配一个线程。当所有的小问题都得到处理后，再调用主线程来进一步操作。

## 实践
```java
public class JoinThread {
	public static class CurrentThread implements Runnable {

		@Override
		public void run() {
			System.out.println("开始当前线程：" + ZonedDateTime.now());
		}
	}

	public static class NewThread implements Runnable {

		@Override
		public void run() {
			System.out.println("你要等我执行完毕" + ZonedDateTime.now());
		}
	}

	public static void main(String[] args) {
		new Thread(new CurrentThread()).start();

		for (int i = 0; i < 10; i++) {
			if (i == 5) {
				Thread newThread = new Thread(new NewThread());
				// main线程调用了jt线程的join()方法，main线程
				// 必须等jt执行结束才会向下执行
				try {
					newThread.start();
					newThread.join();
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
			System.out.println(Thread.currentThread().getName() + "" + i);
		}

	}
}
```
结果：
```Java
main0
main1
main2
main3
main4
你要等我执行完毕2017-10-31T22:16:33.759+08:00[Asia/Shanghai]
开始当前线程：2017-10-31T22:16:33.759+08:00[Asia/Shanghai]
main5
main6
main7
main8
main9
```

上面有三个线程,当主线程执行到i=5的时候，启动了一个新的线程NewThread，该线程不会和main线程并发执行，main线程在此刻必须等待新线程执行结束后才会继续执行。在新线程执行的时候，main实际上就处于等待状态。

# 后台线程（守护线程）

有一种线程，它是在后台运行的，它的任务是为其他的线程提供服务，这种线程被称为后台线程（Daemon Thread），又称为守护线程或精灵线程。JVM的垃圾回收线程就是典型的后台线程。后台线程有个特征：如果所有的前台线程都死亡，后台线程会自动死亡。

调用Thread对象的setDaemon(true)方法可将指定线程设置成后台线程。下面程序将执行线程设置成后台线程，可以看到当所有的前台线程死亡时，后台线程随之死亡。当整个虚拟机中只剩下后台线程时，程序就没有继续运行的必要了，所以虚拟机也就退出了。

## 代码实践
```java
public class DaemonThread extends Thread {
	// 定义后台线程的线程执行体与普通线程没有任何区别
	public void run() {
		for (int i = 0; i < 1000; i++) {
			System.out.println(getName() + "" + i);
		}
	}

	public static void main(String[] args) {
		DaemonThread t = new DaemonThread();
		// 将此线程设置成后台线程
		t.setDaemon(true);
		// 启动后台线程
		t.start();
		for (int i = 0; i < 10; i++) {
			System.out.println(Thread.currentThread().getName() + "" + i);
		}
		// -----程序执行到此处，前台线程（main线程）结束------
		// 后台线程也应该随之结束
	}
}
```
上面程序中的t线程设置成后台线程，然后启动该线程，本来该线程应该执行到i等于999时才会结束，但运行程序时不难发现该后台线程无法运行到999，因为当主线程也就是程序中唯一的前台线程运行结束后，JVM会主动退出，因而后台线程也就被结束了。Thread类还提供了一个isDaemon0方法，用于判断指定线程是否为后台线程

从上面程序可以看出，主线程默认是前台线程， t线程默认也是前台线程。并不是所有的线程默认都是前台线程，有些线程默认就是后台线程——前台线程创建的子线程默认是前台线程，后台线程创建的子线程默认是后台线程

前台线程死亡后，JVM会通知后台线程死亡，但从它接收指令到做出响应，需要一定时间。而且要将某个线程设置为后台线程，必须在该线程启动之前设置，也就是说setDaemon(true)必须在start()方法之前调用，否则会引发IllegaIThreadStateException异常。

# 线程睡眠（sleep）
如果需要让当前正在执行的线程暂停一段时，并进入阻塞状态，则可以通过调用Thread类的静态sleep()方法来实现。当当前线程调用sleep()方法进入阻塞状态后，在其睡眠时间段内，该线程不会获得执行的机会，即使系统中没有其他可执行的线程，处于sleep()中的线程也不会执行，因此sleep()方法常用来暂停程序的执行。

下面程序调用sleep()方法来暂停主线程的执行，因为该程序只有一个主线程，当主线程进入睡眠后，系统没有可执行的线程，所以可以看到程序在sleep()方法处暂停

# 线程让步（yield）
yield()方法是一个和sleep()方法有点相似的方法，它也是Thread类提供的一个静态方法，它也可以让当前正在执行的线程暂停，但它不会阻塞该线程，它只是将该线程转入就绪状态。yield()只是让当前线程暂停一下，让系统的线程调度器重新调度一次，完全可能的情况是：当某个线程调用了yield()方法暂停之后，线程调度器又将其调度出来重新执行。

实际上，当某个线程调用了yield()方法暂停之后，只有优先级与当前线程相同，或者优先级比当前线程更高的处于就绪状态的线程才会获得执行的机会。下面程序使用yield()方法来让当前正在执行的线程暂停。

## sleep()方法和yield()方法的区别

1. sleep()方法暂停当前线程后，会给其他线程执行机会，不会理会其他线程的优先级；但yield()方法只会给优先级相同，或优先级更高的线程执行机会

2. sleep()方法会将线程转入阻塞状态，直到经过阻塞时间才会转入就绪状态：而yield()不会将线程转入阻塞状态，它只是强制当前线程进入就绪状态。因此完全有可能某个线程调用yield()方法暂停之后，立即再次获得处理器资源被执行

3. sleep()方法声明抛出了InterruptcdException异常，所以调用sleep()方法时要么捕捉该异常，要么显式声明抛出该异常；而yield()方法则没有声明抛出任何异常

4. sleep()方法比yield()方法有更好的可移植性，通常不建议使用yield()方法来控制并发线程的执行
