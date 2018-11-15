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
