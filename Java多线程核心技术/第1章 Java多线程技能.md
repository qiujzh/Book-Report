# 第1章 Java多线程技能 #

- 线程的启动
- 如何使线程暂停
- 如何使线程停止
- 线程的优先级
- 线程安全相关的问题

## 1.1进程和线程 ##
### 进程 ###

	1. 进程是操作系统分配资源的最小单位
	
	2. 进程是操作系统结构的基础，是一次程序的执行，是一个程序及其数据在处理机上顺序执行时所发生的活动，是程序在一个数据集合上运行的过程
	
	3. 举例：QQ是一个进程
### 线程 ###
	1. 线程是操作系统调度的最小单位
	
	2. 线程是进程中独立运行的子任务
	
	3. 举例：QQ中的聊天、文件下载是线程
## 1.2 线程的创建 ##
线程的创建有两种方式，一是继承Thread类，二是实现Runnable接口；推荐使用第二种方式，因为Java是单继承的，如果已经继承了其他类则无法再继承Thread类。   
### 继承Thread类 ###
```java
public class MyThread extends Thread{
	public void run(){
		System.out.println(Thread.currentThread.getName());
	}
}
```
### 实现Runnable接口 ###
```java
public class MyThread implements Runnable{
	public void run(){
		System.out.println(Thread.currentThread.getName());
	}
}
```
## 1.3 Thread常用方法 ## 
### currentThread() ###
返回调用当前代码的线程
```java
public class MyThread extends Thread{
	
	public MyThread() {
		System.out.println("MyThread begin");
		System.out.println("----MyThread--Thread.currentThread().getName()：" + Thread.currentThread().getName());
		System.out.println("----MyThread--this.getName()：" + this.getName());
		System.out.println("MyThread end");
	}

	@Override
	public void run(){
		System.out.println("run begin");
		System.out.println("----run--Thread.currentThread().getName()：" + Thread.currentThread().getName());
		System.out.println("----run--this.getName()：" + this.getName());
		System.out.println("run end");
	}
}
}

public class Main {

	public static void main(String[] args){
		currentThreadDemo();
	}
	public static void currentThreadDemo(){
		MyThread mt = new MyThread();
		mt.setName("B");
		Thread t = new Thread(mt);
		t.setName("A");
		t.start();
	}
}
```
main方法运行返回内容如下：
```
MyThread begin
----MyThread--Thread.currentThread().getName()：main
----MyThread--this.getName()：Thread-0
MyThread end
run begin
----run--Thread.currentThread().getName()：A
----run--this.getName()：B
run end
```
其中线程Thread-0与线程B是同一线程，对currentThreadDemo()方法进行简单修改即可验证此结论：
```java
	public static void currentThreadDemo(){
		MyThread mt = new MyThread();
		//mt.setName("B");
		Thread t = new Thread(mt);
		t.setName("A");
		t.start();
	}
```
注释掉mt的命名后，输出结果如下：
```
MyThread begin
----MyThread--Thread.currentThread().getName()：main
----MyThread--this.getName()：Thread-0
MyThread end
run begin
----run--Thread.currentThread().getName()：A
----run--this.getName()：Thread-0
run end

```
this.getName()返回的是当前类对象所代表的线程（this指代当前对象，所以当前对象所代表的线程就是mt，构造方法时名字为Thread-0，重命名后为B），有点绕，别晕！
### isAlive() ###
判断线程是否处于活动状态（活动状态是指线程已经启动且尚未终止，也就是调用start()方法开始到线程终止这段时间所处的状态都是活动状态）。
创建测试类isAliveDemo
```java
public class IsAliveThread extends Thread {
	@Override
	public void run(){
		System.out.println("run: " + this.isAlive());
	}
}
```
Main类中增加测试方法isAliveDemo()
```java
public static void isAliveDemo(){
		IsAliveThread iat = new IsAliveThread();
		System.out.println("main begin: " + iat.isAlive());
		iat.start();
		//Thread.sleep(2000);
		System.out.println("main end: " + iat.isAlive());
	}
```
在main()方法中调用isAliveDemo()方法，输出结果如下：
```
main begin: false
main end: true
run: true
```
### sleep() ###
让当前正在执行的线程休眠一段时间（暂停执行，线程阻塞），这里正在执行的线程指的是this.currentThread()返回的线程。
```java
public class SleepThread extends Thread {

	@Override
	public void run() {
		System.out.println("run threadName = " + this.currentThread().getName() + " begin");
		try {
			Thread.sleep(2000);
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		System.out.println("run threadName = " + this.currentThread().getName() + " end");
	}
}
```
Main中增加测试方法sleepDemo
```java
public static void sleepDemo(){
		SleepThread st = new SleepThread();
		System.out.println("begin " + System.currentTimeMillis());
		st.run();
		//st.start;
		System.out.println("end " + System.currentTimeMillis());
	}
```
运行main方法，运行结果如下：
```
begin 1542348266719
run threadName = main begin
run threadName = main end
end 1542348268721
```
修改sleepDemo方法
```
public static void sleepDemo(){
		SleepThread st = new SleepThread();
		System.out.println("begin " + System.currentTimeMillis());
		//st.run();
		st.start();
		System.out.println("end " + System.currentTimeMillis());
	}
```
运行main方法，运行结果如下：
```
begin 1542348504391
end 1542348504392
run threadName = Thread-0 begin
run threadName = Thread-0 end
```
### getId() ###
返回线程的唯一标识
Main中增加测试方法 getIdDemo
```java
public static void getIdDemo(){
		Long threadId = Thread.currentThread().getId();
		String threadName = Thread.currentThread().getName();
		System.out.println(threadName +": " + threadId);
	}
```
运行main方法，输出结果如下：
```
main: 1
```
## 1.4 停止线程 ## 
停止线程有三种方式： 
 	1）run方法完成后线程终止，正常退出。 
 	2）使用stop方法强制终止，不建议使用，可能会造成不也预料的结果。 
 	3）使用interrupt方法中断线程，建议使用这种方式停止线程。 
### 判断线程是否是停止状态 ### 
 1）this.interrupted()：测试当前线程是否已经中断，这里的当前线程指的是调用this.interrupted()方法的线程。 
 2）this.isInterrupted()：测试线程是否已经中断，这里的线程指的是this所指代的线程。
this.interrupted()示例： 
创建测试类InterruptedThread
```java
public class InterruptedThread extends Thread {

	@Override
	public void run() {
		// TODO Auto-generated method stub
		for(int i = 0; i < 500; i++){
			System.out.println("i = " + i);
		}
```
Main类中增加测试方法interruptedDemo()：
```java
	public static void interruptedDemo(){
		InterruptedThread it = new InterruptedThread();
		it.start();
		it.interrupt();
		System.out.println(it.interrupted());
		System.out.println(it.interrupted());
	}
```
运行main方法，输出结果如下：
```
false
false
i = 0
i = 1
i = 2
i = 3
i = 4
i = 5
.
.
.
i=499
```
it.interrupt()中断的是it线程，**但it.interrupted()方法判断的是调用该行代码的线程，也就是main线程**，main线程未被中断，所以两次判断返回的都是false。

修改测试方法如下，使得main方法被中断：
```java
	public static void interruptedDemo(){
		InterruptedThread it = new InterruptedThread();
		it.start();
		//it.interrupt();
		Thread.currentThread().interrupt();//中断main方法
		System.out.println(it.interrupted());
		System.out.println(it.interrupted());
	}	
```
输出结果如下：
```
true
false
i = 0
i = 1
true
false
i = 0
i = 1
.
.
.
i=499
```
通过测试结果可看出，main方法被中断，但第二次中断状态的判断结果返回的是false，这是由于this.interrupted()方法执行后会将状态标志重置为false的功能造成的，具体原因不清楚，有时间可以研究一下。

this.isInterrupted()示例：
创建测试类IsInterruptedThread
```java
public class IsInterruptedThread extends Thread {

	@Override
	public void run() {
		// TODO Auto-generated method stub
		for(int i = 0; i < 500; i++){
			System.out.println("i = " + i);
		}
	}
}
```
Main类中增加测试方法isInterruptedDemo()
```java
	public static void isInterruptedDemo(){
		IsInterruptedThread iit = new IsInterruptedThread();
		iit.start();
		iit.interrupt();
		System.out.println(iit.isInterrupted());
		System.out.println(iit.isInterrupted());
	}
```
运行main方法，输出结果如下：
```
true
true
i = 0
i = 1
i = 2
.
.
.
i=499
```
iit线程被终止，且第二次状态判断返回的是true，说明isInterrupted方法并没有将线程状态重置的功能。
### 异常终止法 ###
该方法是通过抛异常的方式来实现线程的终止（若判断状态为true，那么抛出异常来终止线程），直接看例子：
创建测试类ExceptionInterruptThread
```java
public class ExceptionInterruptThread extends Thread {

	@Override
	public void run() {
		// TODO Auto-generated method stub
		try {
			for(int i = 0; i < 5000; i++){
				if(this.interrupted()){
					System.out.println("线程被终止，抛出异常来结束线程");
					throw new InterruptedException();
				}
				System.out.println("i = " + i);
			}
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			System.out.println("异常已抛出，线程真的结束了");
		}
	}
	
}
```
Main类中增加测试方法exceptionInterruptDemo()
```java
public static void exceptionInterruptDemo(){
		ExceptionInterruptThread eit = new ExceptionInterruptThread();
		eit.start();
		try {
			eit.sleep(10);
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		eit.interrupt();
		System.out.println("main end");
	}
```
运行main方法，输出结果如下：
```
i = 0
i = 1
.
.
.
i = 168
i = 169
i = 170
main end
线程被终止，抛出异常来结束线程
异常已抛出，线程真的结束了
```
通过抛异常的方式来终止线程的方式需要注意try-catch块要覆盖整个run方法，也就是说所有代码都要放在try-catch中，try-catch后面不要出现任何代码，否则还是会执行。  

**这种方式的思想是让代码执行到catch代码块中的代码就结束，实际上线程并未被强制终止，而是线程执行完catch块后再也没有其他代码可执行了，所以生命周期结束，线程终止。**  

### 在沉睡中（sleep过程中）停止 ###
首先看一下sleep方法的声明
![](./images/sleep.jpg)
方法声明中抛出了InterruptedException异常，也就是说如果有线程中断了当前线程，那么就会抛出异常，而且线程的中断状态会被清除（即调用interrupted方法会返回false）。  

举例1



