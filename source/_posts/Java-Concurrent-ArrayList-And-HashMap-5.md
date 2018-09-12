---
title: 【Java并发】之高并发下ArrayList和HashMap运用(5)
tags:
  - JAVA
  - JAVA并发
  - 多线程
categories:
  - JAVA
  - JAVA并发
abbrlink: 41202
date: 2017-10-31 22:59:03
---

# 摘要

在平常的java代码开发过程中，我们会经常性的就会用到ArrayList和HashMap来辅助自己完成需求工作，而且配合的也是相当的默契。但是如果环境发生改变，在并发情况下，他是否还能够顺利的完成我们想要的要求呢？

<!-- more -->

# 并发下的ArrayList

其实对于ArrayList而言，他并非是一个线程安全的容器，如果在多线程环境下使用ArrayList，必然会导致程序出错。

## 代码实践

```java
public class ArrayListMultiThread {
	static List<Integer> list = new ArrayList<Integer>(10);

	public static class AddThread implements Runnable {

		@Override
		public void run() {
			for (int i = 0; i < 1000000; i++) {
				list.add(i);
			}
		}
	}

	public static void main(String[] args) throws InterruptedException {
		//        创建两个线程t1,t2
		Thread t1 = new Thread(new AddThread());
		Thread t2 = new Thread(new AddThread());
//        两个线程运行
		t1.start();
		t2.start();
//        保持顺序输出
		t1.join();
		t2.join();
		System.out.println(list.size());
	}
}
```

 t1,t2两个线程同时向ArrayList中添加容器，我们期望的效果肯定是2000000.但是由于线程不安全，它真的会得到我们想要的结果吗？

情况一：由于ArrayList在扩容的过程中，内部一致性遭到了破坏，但是却没有任何工作去处理，没有得到锁的保护，使其另外一个线程看到了这样的局面，进而出现了越界的问题。最后的结果则出现了异常的情况。

{% asset_img 2017-10-31_230702.png %}

情况二：两个线程同时访问，发生了肢体上的冲突，对容器的同一位置进行了同时刻的赋值，整体下来则出现了不一致的画面：

{% asset_img 2017-10-31_230733.png %}

情况三：当然也有可能输出２００００００，得到我们想要的结果。每次都吵架，也会有和睦相处的时候啊！真心不易。

纵然有时候能够得到我们想要的结果的，但这也并非是我们想要的结果，唯一的解决方案就是更换一个“容器”，能够达到两者的永久性和睦相处，这样问题不就解决了。

我们采取更换的容器为：Vector

```java
	static Vector<Integer> list = new Vector<Integer>(10);
```
更多Vector请查看后续博客（占坑中...）

# 并发下的HashMap

HashMap相比ArrayList而言，也是线程不安全的。

## 代码实践
```java
public class HashMapMultiThread {
	//    定义一个HashMap集合
	static Map<String, String> map = new HashMap<>();

	public static class AddThread implements Runnable {
		int start = 0;

		public AddThread(int start) {
			this.start = start;
		}

		@Override
		public void run() {
			//对i进行+2操作
			for (int i = start; i < 100000; i += 2) {
				//赋值以2为底的无符号整数
				map.put(Integer.toString(i), Integer.toBinaryString(i));
			}
		}
	}

	public static void main(String[] args) throws InterruptedException {
		//创建两个线程t1,t2
		Thread t1 = new Thread(new HashMapMultiThread.AddThread(0));
		Thread t2 = new Thread(new HashMapMultiThread.AddThread(1));
		t1.start();
		t2.start();
		t1.join();
		t2.join();
		System.out.println(map.size());
	}
}
```

上面的代码，我们预期的结果是输出100000，但是实际情况有三种：

1. 程序正常结束，结果符合预期。
2. 程序正常结束，结果不符合预期，size大小不满足100000
3. 程序无法结束。

为了避免进程不安全，在并发的情况下，我们可以使用ConcurrentHashMap来代替HashMap工作。

```java
static Map<String, String> map = new ConcurrentHashMap<>();
```

在并发的环境下，其实有些操作都是可以避免的，比如一些线程不安全我们完全可以用线程安全的去取代其进行工作。
