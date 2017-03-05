---
layout: post
title: "JAVA GC原理"
description: ""
category: 技术分享
tags: [blog]
---
{% include JB/setup %}
# JAVA Notify 会导致线程死锁！
---
### 这个问题以前没有关注过，下面先引用一位知友的回答： 
> 
>作者：宗文龙  
>链接：https://www.zhihu.com/question/37601861/answer/145545371  
>来源：知乎  
>著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。  

>今天正好碰到这个问题，也疑惑了好久。看了一圈知乎上的答案，感觉没说到根上。所以自己又好好Google了一下，终于找到了让自己信服的解释。  
>先说两个概念：锁池和等待池锁池:  
>1. 假设线程A已经拥有了某个对象(注意:不是类)的锁，而其它的线程想要调用这个对象的某个synchronized方法(或者synchronized块)，由于这些线程在进入对象的synchronized方法之前必须先获得该对象的锁的拥有权，但是该对象的锁目前正被线程A拥有，所以这些线程就进入了该对象的锁池中。  
>2. 等待池:假设一个线程A调用了某个对象的wait()方法，线程A就会释放该对象的锁后，进入到了该对象的等待池中  
>Reference：[java中的锁池和等待池](https://link.zhihu.com/?target=http%3A//blog.csdn.net/emailed/article/details/4689220)   
>然后再来说notify和notifyAll的区别  
>1. 如果线程调用了对象的 wait()方法，那么线程便会处于该对象的等待池中，等待池中的线程不会去竞争该对象的锁。  
>2. 当有线程调用了对象的 notifyAll()方法（唤醒所有 wait 线程）或 notify()方法（只随机唤醒一个 wait 线程），被唤醒的的线程便会进入该对象的锁池中，锁池中的线程会去竞争该对象锁。也就是说，调用了notify后只要一个线程会由等待池进入锁池，而notifyAll会将该对象等待池内的所有线程移动到锁池中，等待锁竞争  
>3. 优先级高的线程竞争到对象锁的概率大，假若某线程没有竞争到该对象锁，它还会留在锁池中，唯有线程再次调用 wait()方法，它才会重新回到等待池中。而竞争到对象锁的线程则继续往下执行，直到执行完了 synchronized 代码块，它会释放掉该对象锁，这时锁池中的线程会继续竞争该对象锁。  
Reference：[线程间协作：wait、notify、notifyAll](https://link.zhihu.com/?target=http%3A//wiki.jikexueyuan.com/project/java-concurrency/collaboration-between-threads.html)   
>综上，所谓唤醒线程，另一种解释可以说是将线程由等待池移动到锁池，notifyAll调用后，会将全部线程由等待池移到锁池，然后参与锁的竞争，竞争成功则继续执行，如果不成功则留在锁池等待锁被释放后再次参与竞争。而notify只会唤醒一个线程。  
> 有了这些理论基础，后面的notify可能会导致死锁，而notifyAll则不会的例子也就好解释了  

### 上面这个没有详细讲notify导致死锁的场景，下面引用stackoverflow的一个回答： 
[Java: notify() vs. notifyAll() all over again](http://stackoverflow.com/questions/37026/java-notify-vs-notifyall-all-over-again)  
相应的中文翻译：   
[notify和notifyAll的一段代码分析](http://www.importnew.com/10173.html)


