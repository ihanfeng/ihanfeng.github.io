---
title: 【Java并发】之多线程wait()和notify()(3)
date: 2017-10-31 20:53:23
tags:
  - JAVA
  - JAVA并发
  - 多线程
categories:
  - JAVA
  - JAVA并发
---

# 概要

在多线程的情况下，由于同一进程的多个线程共享同一片存储空间，在带来方便的同时，也带来了访问冲突这个严重的问题。Java语言提供了专门机制以解决这种冲突，有效避免了同一个数据对象被多个线程同时访问。

为了支持多线程之间的协作，JDK提供了2个非常重要的接口线程等待wait()方法和通知notify()方法。结合与synchronized关键字使用，可以建立很多优秀的同步模型。

<!-- more -->
同步分为``类级别``和``对象级别``，分别对应着``类锁``和``对象锁``。类锁是每个类只有一个，如果static的方法被synchronized关键字修饰，则在这个方法被执行前必须获得类锁；

## synchronized中使用

首先，调用一个Object的wait与notify/notifyAll的时候，必须保证调用代码对该Object是同步的，也就是说必须在作用等同于
```java
synchronized(obj){......}
```
的内部才能够去调用obj的wait与notify/notifyAll三个方法，否则就会报错：
```java
  java.lang.IllegalMonitorStateException:current thread not owner
```
在调用wait的时候，线程自动释放其占有的对象锁，同时不会去申请对象锁。当线程被唤醒的时候，它才再次获得了去获得对象锁的权利。

notify与notifyAll没有太多的区别，只是notify仅唤醒一个线程并允许它去获得锁，notifyAll是唤醒所有等待这个对象的线程并允许它们去获得对象锁，只要是在synchronied块中的代码，没有对象锁是寸步难行的。其实唤醒一个线程就是重新允许这个线程去获得对象锁并向下运行。

notifyAll，虽然是对每个wait的对象都调用一次notify，但是这个还是有顺序的，每个对象都保存这一个等待对象链，调用的顺序就是这个链的顺序。其实启动等待对象链中各个线程的也是一个线程，在具体应用的时候，需要注意一下。

<font color=red>
 wait()，notify()，notifyAll()不属于Thread类,而是属于Object基础类,也就是说每个对像都有wait()，notify()，notifyAll()的功能。因为都个对像都有锁,锁是每个对像的基础,当然操作锁的方法也是最基础了。
</font>

### 实践案例
```java
public class SimpleWatiAndNotify {
	private static final Object object = new Object();

	public static class ThreadOne implements Runnable {
		@Override
		public void run() {
			synchronized (object) {
				System.out.println("线程1启动===>" + ZonedDateTime.now());
				try {
					object.wait();
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
				System.out.println("线程1结束===>" + ZonedDateTime.now());
			}
		}
	}

	public static class ThreadTwo implements Runnable {
		@Override
		public void run() {
			synchronized (object) {
				System.out.println("线程2启动===>" + ZonedDateTime.now());
				// 对象消息通知
				object.notify();
				try {
					Thread.sleep(3000);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
				System.out.println("线程2结束===>" + ZonedDateTime.now());
			}
		}
	}

	public static void main(String[] args) {
		Thread one = new Thread(new ThreadOne());
		Thread two = new Thread(new ThreadTwo());
		one.start();
		two.start();
	}
}
```
执行结果：
```java
线程1启动===>2017-10-31T21:24:37.734+08:00[Asia/Shanghai]
线程2启动===>2017-10-31T21:24:37.735+08:00[Asia/Shanghai]
线程2结束===>2017-10-31T21:24:40.735+08:00[Asia/Shanghai]
线程1结束===>2017-10-31T21:24:40.735+08:00[Asia/Shanghai]
```

通过上面代码,我们开启线程1和线程2.其中线程1执行了object.wait()方法，即线程1申请了object的对象锁。那么执行完wait()方法后，线程1就就会进入等待状态，并释放object的锁。线程2在执行notify()方法之前也会先获得object的对象锁。线程1和线程2的结束时间差3S，符合我们的预期。

我们也发现线程2通知线程1后，线程1并没有立即执行，而是在等待线程2释放object的锁,并重写获取到锁之后才会继续执行。

问题1：如果线程2中的	object.notify();注释掉会发生什么。

```java
线程1启动===>2017-10-31T21:33:04.259+08:00[Asia/Shanghai]
线程2启动===>2017-10-31T21:33:04.261+08:00[Asia/Shanghai]
线程2结束===>2017-10-31T21:33:07.261+08:00[Asia/Shanghai]
```

查看执行结果，我们发现线程2都已经执行完毕，已经释放object的锁，但是线程1依旧在等待中。这样就已经进入死锁状态了。

notify方法很容易引起死锁，除非你根据自己的程序设计，确定不会发生死锁，notifyAll方法则是线程的安全唤醒方法。
