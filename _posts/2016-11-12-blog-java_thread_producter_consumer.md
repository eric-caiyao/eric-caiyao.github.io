---
layout: post
title: "java多线程经典问题仿真程序"
description: "学习"
category: 技术学习
tags: [技术学习]
---
{% include JB/setup %}
# javae线程经典问题仿真程序  
###生产者消费者问题：    
JAVA实现生产者消费者有四种方式： 1. wait()/notifyAll();2.使用管道（PipeWriter/PipeReader）在线程间通信；4. 使用Exchanger构件在生产者和消费者之间交换共享数据；4. 使用阻塞队列；JAVA并发包里提供了很多BlockedQueue的实现，LinkedBlockedQueue可以实现无界队列和有界队列；ArrayBlockedQueue只能实现有界队列。以下代码实现制作土司过程的一个仿真程序，制作土司有三个步骤： 1. 制作土司； 2. 在原始土司上涂黄油； 3. 在涂过黄油的土司上涂果酱。  
<!--break-->

	import java.util.concurrent.ExecutorService;
	import java.util.concurrent.Executors;
	import java.util.concurrent.LinkedBlockingQueue;
	import java.util.concurrent.TimeUnit;

	/**
	 * 制作土司仿真程序
	 * 
	 * 三个过程： 
	 * 1. 制作土司
	 * 2. 给土司抹黄油
	 * 3. 在抹过黄油的土司上涂果酱
	 * 
	 * @author caiyao_home
	 *
	 */
	public class ToasteMaker {
	/**
	 * 
	 * 三个任务分别完成 制作土司、给土司抹黄油、涂果酱
	 * 
	 * 
	 * 三个无界队列分别存放 原始土司、抹过黄油的土司、涂过果酱的土司
	 * 
	 * 
	 */
	public LinkedBlockingQueue<Toast> 
	dryQueue = new LinkedBlockingQueue<>(), 
	butterQueue = new LinkedBlockingQueue<>(), 
	jamQueue = new LinkedBlockingQueue<>();
	
	class DryTask implements Runnable{
		@Override
		public void run() {
			for(int i = 0 ; i < 10 ; i ++){
				Toast toast = new Toast();
				try {
					dryQueue.put(toast);
				} catch (InterruptedException e) {
					System.out.println("dryTask interrupted !");
				}
			}
		}
	}
	class ButterTask implements Runnable{
		@Override
		public void run() {
			while(true){
				try {
					Toast tempToast = dryQueue.take();
					tempToast.currentStatus = Toast.Status.BUTTERED;
					butterQueue.put(tempToast);
				} catch (InterruptedException e) {
					System.out.println("ButterTask interrupted !");
				}
			}
		}
	}
	class JamTask implements Runnable {
		@Override
		public void run() {
			while(true){
				try {
					Toast tempToast = butterQueue.take();
					tempToast.currentStatus = Toast.Status.JAM;
					jamQueue.put(tempToast);
				} catch (InterruptedException e) {
					System.out.println("JamTask interrupted !");
				}
			}
		}
	}
	class Consumer implements Runnable{
		@Override
		public void run() {
			while(true){
				try {
					Toast tempToast = jamQueue.take();
					System.out.println("consuemr eat one toast !" + tempToast.toString());
				} catch (InterruptedException e) {
					System.out.println("consumer interrupted !");
				}
			}
		}
	}
	public static void main(String[] args) {
		
		ToasteMaker test = new ToasteMaker();
		
		ExecutorService pool = Executors.newCachedThreadPool();
		
		DryTask dryTask = test.new DryTask();
		
		ButterTask butterTask = test.new ButterTask();
		
		JamTask jamTask = test.new JamTask();
		
		Consumer consumer = test.new Consumer() ;
		
		
		pool.execute(dryTask);
		
		pool.execute(butterTask);
		
		pool.execute(jamTask);
		
		pool.execute(consumer);
		
		try {
			TimeUnit.SECONDS.sleep(5);
			pool.shutdownNow();
			System.out.println("finish !");
		} catch (InterruptedException e) {
			System.out.println("main thread had been interrupted !");
		}
		
	}
	}


	class Toast {
	
	public enum Status{DRY,BUTTERED,JAM}
	
	public Status currentStatus = Status.DRY;
	
	public void butter(){this.currentStatus = Status.BUTTERED;}
	
	public void jam(){this.currentStatus = Status.JAM;}
	
	}


### 哲学家就餐问题
	场景： 5个哲学家在一个圆桌上就餐，只有5只筷子，每位哲学家会思考和就餐轮流进行。就餐都需要同时拿到左右两边的两只筷子。以下是仿真程序，第一种设定每个哲学家会首先拿到左手边筷子再拿右手边筷子，这种情况会产生死锁，因为会存在循环等待现象，就是大家都同时拿起了左手边的筷子，都在等待右手边筷子。 第二种设定让其中一个哲学家先拿起右手边筷子再拿起左手边筷子，这样解除了循环等待，同时消除死锁。   
	
	import java.util.HashMap;
	import java.util.Map;
	import java.util.Random;
	import java.util.concurrent.ExecutorService;
	import java.util.concurrent.Executors;
	import java.util.concurrent.TimeUnit;

	/**
	 * 
	 * dead lock version 
	 * 
	 * @author caiyao_home
	 *
	 */
	public class PhilosopherEatProblem {

	class Chopstick {
		private boolean isUse = false;
		private int no;
		public Chopstick(int no){
			this.no = no;
		}
		public synchronized void take(){
			try {
				while(isUse){
					wait();
				}
				isUse = true ;
			} catch (Exception e) {
				System.out.println(Thread.currentThread().getName() + "interrupted !");
				return;
			}
		}
		public synchronized void drop(){
			isUse = false;
			notifyAll();
			System.out.println(Thread.currentThread().getName() + " drop chopstick[" + no + "]");
		}
	}
	
	class Philosopher implements Runnable{
		
		private Chopstick left;
		
		private Chopstick right;
		
		private int no;
		
		public Philosopher(Chopstick left , Chopstick right,int no){
			this.left = left;
			this.right = right;
			this.no = no;
		}
		@Override
		public void run() {
			try {
				Random random = new Random(100) ;
				while(true){
					// thinking
					System.out.println("[" + no + "] start thinking ~" );
					TimeUnit.SECONDS.sleep(0);
					
					System.out.println("[" + no + "] wait chopstick [" + left.no + "]");
					left.take();
					System.out.println("[" + no + "] get chopstick [" + left.no + "]");
					
					System.out.println("[" + no + "] wait chopstick [" + right.no + "]");
					right.take();
					System.out.println("[" + no + "] get chopstick [" + right.no + "]");
					
					Long eatingTime = random.nextLong() ;
					System.out.println("[" + no + "] start eating ~ time : " + eatingTime + " millseconds ");
					TimeUnit.SECONDS.sleep(1); // holding
					System.out.println("[" + no + "] eating finish !" );
					left.drop();
					System.out.println("[" + no + "] drop chopstick [" + left.no + "]");
					right.drop();
					System.out.println("[" + no + "] drop chopstick [" + right.no + "]");
				}
			} catch (Exception e) {
				System.out.println("[" + no + "] philosopher thread interrupted !" );
			}
		}
	}
	public static void main(String[] args) {
		
		PhilosopherEatProblem test = new PhilosopherEatProblem(); 
		
		Map<Integer , Chopstick> chopstickMap = new HashMap<>() ;
		
		
		for(int i = 0; i < 5; i ++){
			chopstickMap.put(i,test.new Chopstick(i)) ;
		}
		ExecutorService pool = Executors.newFixedThreadPool(5) ;
		
		// 以下是存在死锁版本
		for(int i = 0; i < 5; i ++){
			pool.execute(
					test.new Philosopher(
							chopstickMap.get(i), 
							chopstickMap.get((i + 1) % chopstickMap.entrySet().size()) , 
							i
					));
		}
		// 以下是不存在死锁版本
		/*for(int i = 0; i < 4; i ++){
			pool.execute(
					test.new Philosopher(
							chopstickMap.get(i), 
							chopstickMap.get(i + 1) , 
							i
					));
		}
		pool.execute(test.new Philosopher(chopstickMap.get(0), chopstickMap.get(4), 4)); */ 
	}
	}


    
**坚持就会胜利，努力就能成功！**  
　　　　　　　　　　　　　　　　　　　　　　　update : 2016年11月13日
　　　　　　　　　　　　　　　　　　　　　　
