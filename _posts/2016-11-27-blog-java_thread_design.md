---
layout: post
title: "java多线程设计模式"
description: ""
category: 技术分享
tags: [JAVA多线程设计模式]
---
{% include JB/setup %}
# JAVA多线程设计模式学习
---
## java 多线程设计模式
> learn by : 《JAVA多线程编程实战指南（设计模式篇）》

### 保护性暂挂模式
> 场景: 多线程环境下，一个业务方法需要根据共享数据的状态来决定自身执行还是挂起。 使用保护性暂挂模式让各个角色各司其职，避免逻辑代码和线程代码混在一起。在该模式中将对共享数据操作（读写）封装成保护性对象（GuardedObject);共享数据状态封装成Predicate对象；业务方法的执行封装成GuardedAction；线程的执行与挂起由阻断器（Blocker）来完成。 
类图： 
![image](http://7xvn6m.com1.z0.glb.clouddn.com/Guarded_Suspension_Model.png)



