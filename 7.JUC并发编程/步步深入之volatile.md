# 步步深入之volatile

## 一、多线程操作共享数据带来的问题

### 1.代码描述问题

两个线程同时操作同一共享变量，一个自增，一个自减，次数都为5000次，结果会是0吗？🤔

```java
static int counter = 0;
public static void main(String[] args) throws InterruptedException {
	Thread t1 = new Thread(() -> {
		for (int i = 0; i < 5000; i++) {
			counter++;
		}
	}, "t1");
	Thread t2 = new Thread(() -> {
		for (int i = 0; i < 5000; i++) {
			counter--;
		}
	}, "t2");
    
	t1.start();
	t2.start();
	t1.join();
	t2.join();
	System.out.println(counter);
}
```

测试结果正、负、零都有，这是为啥呢？**因为自增自减操作并不是原子操作**

### 2.问题分析

想要了解自增自减是不是原子操作必须从字节码开始，













栈与栈帧
Java Virtual Machine Stacks （Java 虚拟机栈）
我们都知道 JVM 中由堆、栈、方法区所组成，其中栈内存是给谁用的呢？其实就是线程，每个线程启动后，虚拟机就会为其分配一块栈内存。

1.每个栈由多个栈帧（Frame）组成，对应着每次方法调用时所占用的内存
2.每个线程只能有一个活动栈帧，对应着当前正在执行的那个方法

线程上下文切换（Thread Context Switch）
因为以下一些原因导致 cpu 不再执行当前的线程，转而执行另一个线程的代码

1.线程的 cpu 时间片用完
2.垃圾回收
3.有更高优先级的线程需要运行
4.线程自己调用了 sleep、yield、wait、join、park、synchronized、lock 等方法

当 Context Switch 发生时，需要由操作系统保存当前线程的状态，并恢复另一个线程的状态，Java 中对应的概念就是程序计数器（Program Counter Register），它的作用是记住下一条 jvm 指令的执行地址，是线程私有的

1.状态包括程序计数器、虚拟机栈中每个栈帧的信息，如局部变量、操作数栈、返回地址等
2.Context Switch 频繁发生会影响性能


3.上下文切换
上下文：线程的运行条件与状态
当主动让出CPU(sleep(),wwait())，阻塞，时间片用完或终止运行的线程时线程不在占用CPU，前三种都涉及线程切换，需保留当前线程的上下文，待再次占用CPU时，恢复现场并加载下一个占用CPU线程的上下文
频繁的切换上下文会消耗CPU





并发编程的本质：充分利用CPU资源