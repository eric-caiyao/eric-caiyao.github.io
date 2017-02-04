---
layout: post
title: "java 多线程笔记"
description: "学习"
category: 技术学习
tags: [技术学习]
---
{% include JB/setup %}
# java多线程笔记


##Lock

	java同步的实现方式5种：   
		1. synchronized关键字  
		2. Volatile关键字  
		3. ReentrantLock可重入锁
		4. ThreadLocal<>    

### ReentrantLock

java.util.concurrent.locks.ReentrantLock.ReentrantLock(boolean fair)
该类提供一个构造参数fair（是否公平）。 当设置为true时，当前获取锁的线程会再次获取锁（如果当前线程需要的话）。 当设置为false时，每一次锁被释放都会从等待队列里面找一个等待时间最久的线程来执行（链表头）。 以下测试代码会分别测试公平锁和非公平锁，在Runnable类的run方法里（一个线程里）会尝试多次获取锁。   

	import java.util.concurrent.locks.Lock;
	import java.util.concurrent.locks.ReentrantLock;

	public class TestReteenLock {
		private Lock fairLock = new ReentrantLock(true) ;
		private Lock unfairLock = new ReentrantLock(false) ;
		public void testFairLock(){
			for(int i = 0 ; i < 10 ; i ++){
				new Thread(new Job(fairLock)).start() ;
			}
		}
		public void testUnFairLock(){
			for(int i = 0 ; i < 10 ; i ++){
				new Thread(new Job(unfairLock)).start() ;
			}
		}
		public static void main(String[] args) {
			TestReteenLock test = new TestReteenLock() ;
			//test.testFairLock() ;
			test.testUnFairLock();
		}
		class Job implements Runnable {
			private Lock lock = null ;
			public Job(Lock lock){
				this.lock = lock ;
			}
			@Override
			public void run() {
				for(int i = 0 ; i < 5 ; i ++){
					lock.lock() ; 
					try {
						System.out.println("currentThread : " + Thread.currentThread().getName() + " get lock!");
					} finally{
						lock.unlock() ;
					}
				}
			}
		}
	}

### ReentrantReadWriteLock 
可重入读写锁。   

　　进入读锁的前提是 没有线程已获取写锁，或者获取写锁的线程就是当前线程。 进入写锁的前提是： 没有线程已获取读锁或者写锁。 写锁排斥任何的读锁和写锁；读锁排斥写锁。   
　　规则：   
　　　　读锁不能在次获得写锁
　　　　写锁可以降级为读锁，步骤为  获取写锁 ---> 获取读锁 ---> 释放写锁 
以下是测试代码，代码里会设置一个共享数据，有对该数据操作的get方法和put方法，get会获取读锁，put会获取写锁，会创建10个线程，5个读线程，5个写线程，观察是否会出现异常现象： 

	import java.util.ArrayList;
	import java.util.List;
	import java.util.concurrent.Callable;
	import java.util.concurrent.ExecutorService;
	import java.util.concurrent.Executors;
	import java.util.concurrent.TimeUnit;
	import java.util.concurrent.locks.ReentrantReadWriteLock;

	public class TestReteenReadWriteLock {
	ShareQueue shareQueue = new ShareQueue() ;
	public static void main(String[] args) {
		TestReteenReadWriteLock test = new TestReteenReadWriteLock() ;
		ExecutorService pool = Executors.newCachedThreadPool() ;
		List<Callable<String>> tasks = new ArrayList<Callable<String>>() ;
		// 添加5个写线程
		for(int i = 6 ; i < 10 ; i ++){
			final int tempI = i ;
			final String taskId = String.valueOf(i) ;
			tasks.add(new Callable<String>(){
				@Override
				public String call() {
					test.shareQueue.put(tempI);
					return  taskId ;
				}
			}) ;
		}
		// 添加5个读线程
		for(int i = 0 ; i < 5 ; i ++){
			final String taskId = String.valueOf(i) ;
			tasks.add(new Callable<String>(){
				@Override
				public String call() {
					test.shareQueue.read() ;
					return taskId ;
				}
			}) ;
		}
		
		
		try {
			pool.invokeAll(tasks) ;
			pool.shutdown() ;
			pool.awaitTermination(5, TimeUnit.MINUTES) ; // 最多执行5分钟
		} catch (Exception e) {
			e.printStackTrace() ;
		}
	}
	// 共享数据
	class ShareQueue {
		private Object data ;
		private ReentrantReadWriteLock lock = new ReentrantReadWriteLock() ;
		public void read(){
			lock.readLock().lock() ;
			try {
				System.out.println(Thread.currentThread().getName() + " get read lock ! ") ;
				Thread.sleep(1000) ;
				System.out.println(Thread.currentThread().getName() + " read data : " + data);
			}catch(Exception e){
				System.out.println("sleep failed ! " + e.getMessage());
			}finally{
				lock.readLock().unlock() ;
			}
		}
		public void put(Object o){
			lock.writeLock().lock() ;
			try{
				System.out.println(Thread.currentThread().getName() + " get write lock !") ;
				Thread.sleep(1000) ;
				data = o ;
				System.out.println(Thread.currentThread().getName() + " put value["+o.toString()+"] successful!");
			}catch(InterruptedException e){
				System.out.println("sleep interrupted !") ;
			}finally{
				lock.writeLock().unlock() ; 
			}
		}
	}
	}

参考：   
http://ifeve.com/reentrantlock-and-fairness/    
http://www.cnblogs.com/liuling/archive/2013/08/21/2013-8-21-03.html

*注： 可以被共享的变量有： 实例变量 、静态变量、数组变量*  

### Volatile
　　volatile是轻量级的synchronized。在多处理器的场景中，如果一个处理器写一个共享变量的值，有可能该写操作只是写到内存cache中（L1，L2，L3）。JVM保证在使用volatile修饰共享变量（该变量必须保持有效一致性）后，多处理器读的值是一致的。 volatile只具有synchronized的可见性特性（synchronized同时具有原子性和可见性），在某些情况如果读操作远远大于写操作，使用volatile要优于synchronized。对volatile变量进行读操作的读开销要远远小于非volatile变量，但是对volatile变量写操作的开销要比非volatile变量多很多。    
　　使用volatile的常见几种模式：   
　　1. 状态标识  

	volatile boolean shutdownRequested;  
	
	...

	public void shutdown() { shutdownRequested = true; }

	public void doWork() { 
	    while (!shutdownRequested) { 
	        // do stuff
	    }
	}
   多处理器场景下，如果某一个处理器修改了是否停止(shutdownRequested），需要保证其他处理器立即可见该状态的修改。   
　　2. 开销较低的读写策略  

	@ThreadSafe
	public class CheesyCounter {
    // Employs the cheap read-write lock trick
    // All mutative operations MUST be done with the 'this' lock held
    @GuardedBy("this") private volatile int value;

    public int getValue() { return value; }

    public synchronized int increment() {
        return value++;
    }
	}
该模式应用场景为多读少写，volatile变量value保证了多线程对该变量的可见性，同步锁(synchronized)保证了value更新的原子性和可见性。 

参考： http://www.ibm.com/developerworks/cn/java/j-jtp06197.html

*问题： 对volatile变量修饰的域进行的读取和写入是直接针对内存吗？还是通过其他算法保证读取和写入与内存保持一致？*
答： 对volatile变量的写会刷新到内存，对volatile变量的读会保证本地缓冲和内存中一致

### 原子类
当多个线程试图同时访问同一个同一个变量（实例变量、静态变量、数组变量）而且至少有一个线程试图修改该变量的值时，可以考虑使用concurrent.atomic包下的原子类，以避免由于线程执行顺序不确定导致的共享变量值与预期不相符。这些原子类的底层实现不是通过锁的机制，而是通过CAS（compare and set）的无锁机制，当对变量进行修改的时候会再次判断该变量是否已被别的线程修改，如果已经被修改，则该次访问失败，如果没有被修改则该次访问成功。atomic包下提供了四组原子类：  
##### AtomicBoolean，AtomicInteger，AtomicLong，AtomicReference  

	// 提供对boolean/int/long/对象的原子访问，以下代码为使用AtomicReference实现的无阻塞栈  
	import java.util.concurrent.atomic.AtomicReference;

	public class LinkedStack<T> {
	
	private AtomicReference<Node<T>> stack = new AtomicReference<>() ;
	
	public T push(T newValue){
		
		for(;;){ // 直到CAS成功才返回
			Node<T> oldValue = stack.get() ;
			
			Node<T> newNode = new Node<T>(newValue,oldValue) ;
			
			if(stack.compareAndSet(oldValue, newNode)){ // 判断oldValue是否被修改
				return newValue ;
			}
			
		}
		
	}
	public T pop(){
		
		for(;;){
			Node<T> oldNode = stack.get() ;
			
			Node<T> newNode = oldNode.getNext() ;
			
			if(stack.compareAndSet(oldNode, newNode)){
				return oldNode.getValue() ;
			}
		}
		
	}
	private static final class Node<T>{
		
		private T value ;
		private Node<T> next ;
		
		public Node(T value , Node<T> next){
			this.value = value ;
			this.next = next ;
		}
		
		public T getValue() {
			return value;
		}
		public void setValue(T value) {
			this.value = value;
		}
		public Node<T> getNext() {
			return next;
		}
		public void setNext(Node<T> next) {
			this.next = next;
		}
		
	}
	}	
##### AtomicIntegerArray，AtomicLongArray,AtomicReferenceArray  
	提供int/long/对象数组的原子访问
##### AtomicLongFieldUpdater，AtomicIntegerFieldUpdater，AtomicReferenceFieldUpdater  
	// 提供对域变量的原子访问，该原子类对域变量有一些限制：   
	// 1. 字段必须是volatile类型的
	// 2. 调用者必须能直接操作域变量(例如： 修饰为default，则同一包下的访问是允许的)
	// 3. 只能是实例变量
	// 4. 不能是final修饰的  
	public class FieldUpdaterTest {
	/**
	 * 一百个线程同时修改一个域变量的值，只允许一个执行修改操作
	 */
	public volatile int a = 0 ;
	
	public static FieldUpdaterTest test = new FieldUpdaterTest() ;
	
	public static AtomicIntegerFieldUpdater<FieldUpdaterTest> updater = 
			AtomicIntegerFieldUpdater.newUpdater(FieldUpdaterTest.class, "a") ;
	
	public static void main(String[] args) {
		
		ExecutorService pool = Executors.newFixedThreadPool(100) ;
		
		for(int i = 0 ; i < 100 ; i ++ ){
			pool.execute(new Runnable(){
				@Override
				public void run() {
					System.out.println("execute") ;
					if(updater.compareAndSet(test, 0 , 101)){
						System.out.println("modified !") ;
					}
				}
			});
		}
		pool.shutdown() ; 
	}
	}

##### AtomicMarkableReference，AtomicStampedReference  
	//这两个原子类不仅提供了对对象的原子访问，还附加了一些其他的信息。 AtomicMarkableReference提供了<Reference,Boolean>类型的原子访问； AtomicStampedReference提供了<Reference,Integer>的原子访问。以下为测试代码：  
	   
	import java.util.concurrent.atomic.AtomicMarkableReference;

	public class AtomicMarkableReferenceExample {

	private static Person person;

	private static AtomicMarkableReference<Person> aMRperson;

	public static void main(String[] args) throws InterruptedException {

		Thread t1 = new Thread(new MyRun1());

		Thread t2 = new Thread(new MyRun2());

		person = new Person(15);

		aMRperson = new AtomicMarkableReference<Person>(person, false);

		System.out.println("Person is " + aMRperson.getReference() + "\nMark: "

				+ aMRperson.isMarked());

		t1.start();

		t2.start();

		t1.join();

		t2.join();

	}

	static class MyRun1 implements Runnable {

		public void run() {
			
			for (int i = 0; i <= 5; i++) {

				aMRperson.getReference().setAge(person.getAge() + 1);
				System.out.println(Thread.currentThread().getName() + "after setAge : " + aMRperson.getReference().getAge() );

				boolean isSuccess = aMRperson.compareAndSet(new Person(18), new Person(18), false, true);
				System.out.println(Thread.currentThread().getName() + "compareAndSet: " + isSuccess) ;

				aMRperson.set(aMRperson.getReference(), true);
				
				System.out.println(Thread.currentThread().getName() + 
						String.format(
								"set reference  %s : %s ", 
								aMRperson.getReference(),
								String.valueOf(aMRperson.isMarked()))) ; ;
			}

		}

	}

	static class MyRun2 implements Runnable {

		public void run() {

			for (int i = 0; i <= 5; i++) {

				aMRperson.getReference().setAge(person.getAge() - 1);
				System.out.println(
						Thread.currentThread().getName() 
						+ " after setAge : " 
						+ aMRperson.getReference().getAge());
				
				boolean isSuccess = aMRperson.attemptMark(new Person(40), true);
				System.out.println(
						Thread.currentThread().getName() 
						+ " attemptMark : " + String.valueOf(isSuccess)) ;
			}

		}

	}

	static class Person {

		private int age;

		public Person(int age) {

			this.age = age;

		}

		public int getAge() {

			return age;

		}

		public void setAge(int age) {

			this.age = age;

		}

		@Override

		public String toString() {

			return "Person age is " + this.age;

		}

	}

	}


在CAS里面有个潜在的ABA问题，这个问题可能会在CAS时出现一些意想不到的问题，这里有对ABA问题比较好的解释： [用AtomicStampedReference解决ABA问题](http://blog.hesey.net/2011/09/resolve-aba-by-atomicstampedreference.html)  


资料：   
[1] [Java 理论与实践: 非阻塞算法简介](http://www.ibm.com/developerworks/cn/java/j-jtp04186/)  
  
    
## ThreadLocal
ThreadLocal可以实现保存每个线程自己使用的对象。ThreadLocal变量对于各个线程完全独立互不影响。 Thread类(代表一个独立线程)有一个名叫threadLocals（ThreadLocal.ThreadLocalMap）的域变量，每次通过ThreadLocal.set()方法存储数据，都会将当前ThreadLocal类作为key，set进去的obj作为value，put到currentThread()的threadLocals Map上。 这样就实现了ThreadLocal变量在每一个线程（Thread）上都是独立的。   

## 进入阻塞状态的几种情况  
1. sleep（millisencods）方法调用，任务在指定时间内不会运行。
2. 通过调用wait使线程挂起，直到线程得到notify或notifyAll消息。线程才会进入就绪状态。
3. 等待输入/输出
4. 任务试图在某个对象上调用其同步控制方法。但是对象锁不可用，因为另一个任务已经获得了这个锁


## 线程中断  
java 有三种方式中断线程：   
* Threa.stop() 。 该方法会让线程立即终止，并抛出ThreadDeath（）异常，不推荐使用。因为有可能被终止的线程所获得的资源(锁等)永远得不到释放。  
* 使用退出标志。 当业务逻辑处理完后正常结束，run方法中return  
* 使用interrupt中断线程。使用该方法有两种情况:  
 + 当线程正在运行，调用该线程的interrupt方法后，会把该线程的interrupt属性设置为true,可以通过判断该标志位结束。   
 + 如果线程由于IO/等待锁等情况被阻塞，则线程会抛出一个InterruptException的异常。   
 
以下代码为正常运行线程通过interrupt方法中断：   

	public class InterruptTest {
	static class InterruptedThread extends Thread {
		@Override
		public void run() {
			while(!Thread.currentThread().isInterrupted()){
				System.out.println("going!") ;
			}
			System.out.println("interrupted! ") ;
		}
	}
	public static void main(String[] args) {
		Thread interruptedThread = new InterruptedThread() ;
		interruptedThread.start() ;
		try {
			TimeUnit.SECONDS.sleep(3);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		interruptedThread.interrupt() ; 
	}
	}  
以下代码为正在阻塞中线程被中断：   

	public class InterruptTest {
	static class InterruptedThread extends Thread {
		@Override
		public void run() {
			try {
				Thread.currentThread().sleep(4000);
				System.out.println("thread resume!") ;
			} catch (InterruptedException e) {
				System.out.println("interrupted! ") ;
			}
		}
	}
	public static void main(String[] args) {
		Thread interruptedThread = new InterruptedThread() ;
		interruptedThread.start() ;
		try {
			TimeUnit.SECONDS.sleep(3);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		interruptedThread.interrupt() ; 
	}
	}  
使用interrupt的经典例子：   

	public void run(){    
            try{    
                 ....    
                 while(!Thread.currentThread().isInterrupted()&& more work to do){    
                        // do more work;    
                 }    
            }catch(InterruptedException e){    
                        // thread was interrupted during sleep or wait    
            }    
            finally{    
                       // cleanup, if required    
            }    
    }  
一定要考虑阻塞和非阻塞两种情况，并做出正确的清理操作。 
    
**坚持就会胜利，努力就能成功！**  
　　　　　　　　　　　　　　　　　　　　　　　update : 2016年11月5日
　　　　　　　　　　　　　　　　　　　　　　