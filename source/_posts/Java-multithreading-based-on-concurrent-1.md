---
title: 【Java并发】之多线程基础(1)
tags:
  - JAVA
  - JAVA并发
categories:
  - JAVA
  - JAVA并发
abbrlink: 16139
date: 2017-10-30 22:08:44
---
# 简介

在传统的线程技术中，创建多线程有2中方式

* 继承Thread类，并重写run()方法；
* 实现Runnable接口，覆盖接口的run()方法；

<!-- more -->

# 探究Thread类的run()源码

打开Thread类的定义，我们发现run()的实现如下：

```java
@Override
   public void run() {
       if (target != null) {
           target.run();
       }
   }
```

我们发现run()的实现方法很简单，就是一个简单的if判断语句。那么target又是何方神圣呢？我们通过IDEA跟踪到taget，发现如下：

```java
private Runnable target;
```
查看源码我们发现，原来target其实就是一个类型为Runnable接口的对象，且是通过构造器传入的。

再点击Runnable进去，我们又发现新大陆：
```java
@FunctionalInterface
public interface Runnable {
    public abstract void run();
}
```

Runnable接口中只有1个抽象run()方法。

通过上面的源码查看分析，我们可以知道Thread类中的target如果不为空，就会实现Runnable接口的run()，那么我们执行的run()方法就是Runnable中的方法。

根据上面的源码，我们也就很容易知道为什么创建多线程有2种方式了。

# 通过继承Thread类实现多线程

1. 继承Thread类，并实现run()方法
2. 调用start()方法开启线程

为了简单测试，我们直接在main方法中通过内部类测试
```java
public class TraditionalThread {
	public static void main(String[] args) {
		Thread thread = new Thread() {
			@Override
			public void run() {
				try {
					Thread.sleep(500);//让线程休息500毫秒
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
				System.out.println(Thread.currentThread().getName());//打印出当前线程名
			}
		};
		thread.start();
	}
}
```

# 通过实现Runnable接口实现多线程

1. 实现Runnable接口，并实现run()方法
2. 调用start()方法开启线程

```java
public class TraditionalThread {
	public static void main(String[] args) {
		Thread thread = new Thread(new Runnable() {
			@Override
			public void run() {
				try {
					Thread.sleep(500);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
				System.out.println(Thread.currentThread().getName());
			}
		});
		thread.start();
	}
}
```
