---
layout: post
title: "JAVA GC原理"
description: ""
category: 技术分享
tags: [blog]
---
{% include JB/setup %}
# JAVA垃圾回收机制原理
---
### JAVA对象引用类型  
#### 强引用（Strong reference）  

	一般编程使用的是强引用。例如 String a = "abc". 此时 a 就持有"abc"的强引用. 当一个对象的强引用计数为0时，GC就可以立即回收该对象内存（该对象没有其他类型引用的情况下，例如软引用)

#### 弱引用 (weak reference)  

    Counter counter = new Counter(); // strong reference - line 1
    WeakReference<Counter> weakCounter = new WeakReference<Counter>(counter); //weak reference
    counter = null; // now Counter object is eligible for garbage collection
当执行到第三行，把counter变量置为null时，weakCounter对counter的引用不会阻止垃圾回收器对count对象的回收。
#### 软引用 (soft reference)
	Counter prime = new Counter(); // prime holds a strong reference – line 2
	SoftReference soft = new SoftReference(prime) ; //soft reference variable has SoftReference to Counter Object created at line 2

	prime = null; // now Counter object is eligible for garbage collection but only be collected when JVM absolutely needs memory
 当执行到第三行将prime置为null时，垃圾回收器不会立即回收prime引用的对象，因为soft持有对该对象的软引用，只有当jvm需要内存空间时（内存不足）才会回收prime对象。软引用适合做cache或着pool
#### 虚引用 (phantom reference)
	
虚引用与软引用和弱引用的一个区别在于：虚引用必须和引用队列（ReferenceQueue）联合使用。当垃圾回收器准备回收一个对象时，如果发现它还有虚引用，就会在回收对象的内存之前，把这个虚引用加入到与之关联的引用队列中，当用户代码再次从引用队列里取出虚引用对象时，gc才会执行引用对象的清理动作。虚引用最常见的用法是以某种可能比使用 Java 终结机制更灵活的方式来指派 pre-mortem 清除动作。

*参考：[《Java源码分析》：ReferenceQueue、Reference及其子类](http://blog.csdn.net/u010412719/article/details/52035792)、[话说ReferenceQueue](http://hongjiang.info/java-referencequeue/)*
### JAVA内存模型
![JAVA内存模型](http://7xvn6m.com1.z0.glb.clouddn.com/Java-Memory-Model-450x186.png)

*待续。。。。*