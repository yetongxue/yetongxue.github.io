---
title: 线程池主线程等待所有子线程执行完成后，再继续后面工作
date: 2018-07-12 10:09:55
tags: [Java,线程]
categories:
  - Java
  - thread
---

# 代码分析
先来看一下这段代码，运行的结果会是什么？
```java
public class Test {
	public static void main(String[] args) {
		Test test = new Test();
		ExecutorService fixedThreadPool = Executors.newFixedThreadPool(5); 
		for (int i = 1; i < 3; i++) {
			final int index = i;
			Future<Integer> future = fixedThreadPool.submit(new Callable<Integer>() {
				public Integer call() {
				try {
					if(index==1){
						Integer res = test.querySlowMethod1(1);
						System.out.println("loop num："+res);
					}
					else if(index==2){
						Integer res2 = test.querySlowMethod2(2);
						System.out.println("loop num："+res2);
					}
				} catch (Exception e) {
					e.printStackTrace();
				}
				return index;
				}
			});
		}
		fixedThreadPool.shutdown();
		System.out.println("---------------------main thread end---------------------");
	}
	public Integer querySlowMethod1(int i){
			try {
				System.out.println("querySlowMethod1 start ,threadName:"+Thread.currentThread().getName());
				Thread.sleep(5000);
			} catch (Exception e) {
				e.printStackTrace();
			}
			System.out.println("querySlowMethod1----end");
			return i;
	}
	public Integer querySlowMethod2(int i){
		try {
			System.out.println("querySlowMethod2 start,threadName:"+Thread.currentThread().getName());
			Thread.sleep(100);
		} catch (Exception e) {
			e.printStackTrace();
		}
		System.out.println("querySlowMethod2----end");
		return i;
	}
}
```
输出结果是：
```base
---------------------main thread end---------------------
querySlowMethod1 start ,threadName:pool-1-thread-1
querySlowMethod2 start,threadName:pool-1-thread-2
querySlowMethod2----end
loop num：2
querySlowMethod1----end
loop num：1
```
<!-- more -->
有时候我们希望等所有子线程执行完成后，才继续主线程后面的工作，即我们希望最后输出的是：
---------------------main thread end---------------------
这时，我们可以使用future.get()的阻塞性来实现，实现代码如下：
```java
public class Test {
	public static void main(String[] args) {
		Test test = new Test();
		List<Future<?>> futures = new ArrayList<Future<?>>(100);
		ExecutorService fixedThreadPool = Executors.newFixedThreadPool(5); 
		for (int i = 1; i < 3; i++) {
			final int index = i;
			Future<Integer> future = fixedThreadPool.submit(new Callable<Integer>() {
				public Integer call() {
					try {
						if(index==1){
							Integer res = test.querySlowMethod1(1);
							System.out.println("loop num："+res);
						}
						else if(index==2){
							Integer res2 = test.querySlowMethod2(2);
							System.out.println("loop num："+res2);
						}
					} catch (Exception e) {
						e.printStackTrace();
					}
					return index;
				}
			});
			futures.add(future);
		}
		fixedThreadPool.shutdown();
		for(Future<?> item:futures){
			try {
				System. out.println("线程运行结果---------------" + item.get());
			} catch (InterruptedException e) {
				e.printStackTrace();
			} catch (ExecutionException e) {
				e.printStackTrace();
			}
		}

		System.out.println("---------------------main thread end---------------------");
	}
	public Integer querySlowMethod1(int i){
		try {
			System.out.println("querySlowMethod1 start ,threadName:"+Thread.currentThread().getName());
			Thread.sleep(5000);
		} catch (Exception e) {
			e.printStackTrace();
		}
		System.out.println("querySlowMethod1----end");
		return i;
	}
	public Integer querySlowMethod2(int i){
		try {
			System.out.println("querySlowMethod2 start,threadName:"+Thread.currentThread().getName());
			Thread.sleep(100);
		} catch (Exception e) {
			e.printStackTrace();
		}
		System.out.println("querySlowMethod2----end");
		return i;
	}
}
```
输出结果为：
```base
querySlowMethod1 start ,threadName:pool-1-thread-1
querySlowMethod2----end
loop num：2
querySlowMethod1----end
loop num：1
线程运行结果---------------1
线程运行结果---------------2
---------------------main thread end---------------------
```