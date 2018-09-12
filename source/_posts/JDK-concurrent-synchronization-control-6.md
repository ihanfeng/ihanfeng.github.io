---
title: 【Java并发】JDK并发包之同步控制(6)
tags:
  - JAVA
  - JAVA并发
  - 多线程
  - 并发包
  - null
categories:
  - JAVA
  - JAVA并发
abbrlink: 63647
date: 2017-11-01 10:07:33
---
# 摘要

为了更好地支持并发程序，JDK内部提供了大量实用的API和框架。同步控制是并发编程必不可少的重要手段。

# 重入锁(RetreenLock)

重入锁完全可以替代``synchronized``关键字，在JDK5.0，重入锁的性能要好于synchronized，在JDK6.0后synchronized做了大量的优化，两者性能差不多。

<!-- more -->

重入锁使用JDK自带的并发包``java.util.concurrent.locks.ReentrantLock``类实现。

## 代码实践
```java
public class RetreenLockThread implements Runnable {
	public static ReentrantLock lock = new ReentrantLock();
	public static int i = 0;

	@Override
	public void run() {
		for (int j = 0; j < 10000; j++) {
			lock.lock();
			try {
				i++;
			} finally {
				lock.unlock();
			}
		}
	}

	public static void main(String[] args) throws InterruptedException {
		RetreenLockThread thread = new RetreenLockThread();

		Thread t1 = new Thread(thread);
		Thread t2 = new Thread(thread);
		t1.start();
		t2.start();
		t1.join();
		t2.join();
		System.out.println("结果：" + i);
	}
}
```

上面的代码，我们使用重入锁保护临界区的资源i，确保多线程对i操作的安全。重入锁中，我们可以自己控制何时加锁何时释放锁，比synchronized来说更加灵活。使用重入锁，要保证最后一定要释放锁，否则其他线程就没办法方法临界区资源，从而进入死锁。

重入锁可以加多把锁，同时释放锁是否加几把锁就要释放几次。

## 中断响应

对于synchronized来说，如果一个线程在等待锁，那么结果要么是继续等待，要么是获得锁继续执行。

## 锁申请等待限时

除了等待外部通知外，要避免死锁就需要采用限时等待，即``tryLock(long timeout, TimeUnit unit)``。

## 公平锁

在大多数情况下，锁都是非公平的。重入锁允许我们锁的公平性进行设置，通过构造函数进行``public ReentrantLock(boolean fair)``设置。当fair为true时表示锁是公平的。

正常需求情况是不推荐使用公平锁，因为实现成本高，性能低下。

综上，重入锁的实现包括三要素

1. 原子状态

  原子状态使用CAS操作来存储当前锁的状态，判断锁是否已经被别的线程持有。

2. 等待队列

  所有没有请求道锁的线程，都会进入等待队列进行等待。待有线程释放锁以后，系统就能够从等待队列中唤醒一个线程继续工作。

3. 阻塞原语park()和unpark(),用来挂起和恢复线程。没有得到锁的线程会被挂起。

# Condition条件

Condition对象和Object.wait()及notify()方法作用类似。wait()和notify()方法是和synchronized关键字一起使用，而condtion是与重入锁相关联的。

Condition接口提供如下基本方法：
```java
void await() throws InterruptedException;
void awaitUninterruptibly();
long awaitNanos(long nanosTimeout) throws InterruptedException;
boolean await(long time, TimeUnit unit) throws InterruptedException;
boolean awaitUntil(Date deadline) throws InterruptedException;
void signal();
void signalAll();
```

* await()方法会使当前线程等待，同时释放当前锁，当其他线程中使用signal()或signalAll()时，线程会重新获得锁并继续执行。或者当线程被中断时也能跳出等待。

* awaitUninterruptibly()和await()类似，但是不会再等待过程中响应中断。

* signal()唤醒一个在等待中的线程，signalAll()唤醒所有在等待中的线程。


# 信号量(Semaphore)

...

# 读写锁(ReadWriteLock)

ReadWriteLock是JDK5中提供的读写分离锁。读写分离锁可以有效地帮助减少锁竞争，以提高系统性能。

读写锁允许多个线程同时读，但是考虑数据的完整性，写写操作和读写操作之间依然需要相互等待和持有锁。

# 倒计时器(CountDownLatch)

CountDownLatch是通过一个计数器来实现的，计数器的初始值为线程的数量。每当一个线程完成了自己的任务后，计数器的值就会减1。当计数器值到达0时，它表示所有的线程已经完成了任务，然后在闭锁上等待的线程就可以恢复执行任务。

CountDownLatch的构造函数接收一个整数作为参数。
```java
public void CountDownLatch(int count) {...}
```

构造器中的计数值（count）实际上就是闭锁需要等待的线程数量。这个值只能被设置一次，而且CountDownLatch没有提供任何机制去重新设置这个计数值。

与CountDownLatch的第一次交互是主线程等待其他线程。主线程必须在启动其他线程后立即调用CountDownLatch.await()方法。这样主线程的操作就会在这个方法上阻塞，直到其他线程完成各自的任务。

其他N 个线程必须引用闭锁对象，因为他们需要通知CountDownLatch对象，他们已经完成了各自的任务。这种通知机制是通过 CountDownLatch.countDown()方法来完成的；每调用一次这个方法，在构造函数中初始化的count值就减1。所以当N个线程都调 用了这个方法，count的值等于0，然后主线程就能通过await()方法，恢复执行自己的任务。

## 使用场景

1. <Strong>实现最大的并行性：</Strong>有时我们想同时启动多个线程，实现最大程度的并行性。例如，我们想测试一个单例类。如果我们创建一个初始计数为1的CountDownLatch，并让所有线程都在这个锁上等待，那么我们可以很轻松地完成测试。我们只需调用 一次countDown()方法就可以让所有的等待线程同时恢复执行。

2. <Strong>开始执行前等待n个线程完成各自任务：</Strong>例如应用程序启动类要确保在处理用户请求前，所有N个外部系统已经启动和运行了。

3. <Strong>死锁检测：</Strong>一个非常方便的使用场景是，你可以使用n个线程访问共享资源，在每次测试阶段的线程数目是不同的，并尝试产生死锁。
## 代码实践

...


# 循环栅栏(CyclicBarrier)

CyclicBarrier是另外一种多线程并发控制实用工具，其功能和CountDownLatch类似，也可以实现线程间的计数等待。CyclicBarrier也可以理解为循环栅栏，计数器可以反复使用。


# 线程阻塞工具(LockSupport)

LockSupport是个方便实用的线程阻塞工具，可以在线程内任意位置让线程阻塞。
