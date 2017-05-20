---
layout: post
title: "思考"
description: ""
category: 思考
tags: [blog]
---
{% include JB/setup %}

### Mac VirtualBox Linux虚拟机安装及共享文件夹配置
#### 准备资源：

- [x] 已安装Mac版VirtualBox
- [x] 相应版本Linux镜像文件(本文使用Oracle Linux6.5)
- [x] VirtualBox 虚拟机增强包(VBoxGuestAdditions_5.1.6.iso)

#### 安装步骤：
1. 新建，设置虚机电脑名和系统类型
![新建，设置虚机电脑名和系统类型](http://7xvn6m.com1.z0.glb.clouddn.com/virtualhost_and_mysql_install_1.png)
![新建，设置虚机电脑名和系统类型](http://7xvn6m.com1.z0.glb.clouddn.com/virtualhost_and_mysql_install_2.png)
2. 内存设置，既然是Linux安装图形界面就没有什么必要了，每个Linux发行版里都会有说明图形界面需要的内存最小值。这里设置为512会
自动以Text Mode安装，具体该发行版(Oracle Linux6.5)的准确限制是多少我没有找到。
![](http://7xvn6m.com1.z0.glb.clouddn.com/virtualhost_and_mysql_install_3.png)
3. 硬盘设置
![](http://7xvn6m.com1.z0.glb.clouddn.com/virtualhost_and_mysql_install_4.png)
4. 虚拟硬盘类型。 VDI： VirtualBox所属；VHD： 微软的；VMDK： 通常是Vmware的，这三种类型在VirtualBox的说明里有(具体
可以在VirtualBox里点击帮助->内容->Virtual Storage->Disk image files查看)
![](http://7xvn6m.com1.z0.glb.clouddn.com/virtualhost_and_mysql_install_5.png)
5. 设置硬盘文件位置
![](http://7xvn6m.com1.z0.glb.clouddn.com/virtualhost_and_mysql_install_6.png)
6. 设置系统镜像
![](http://7xvn6m.com1.z0.glb.clouddn.com/virtualhost_and_mysql_install_8.png)
![](http://7xvn6m.com1.z0.glb.clouddn.com/virtualhost_and_mysql_install_9.png)
7. 设置共享文件夹，为与宿主机共享数据所用
![](http://7xvn6m.com1.z0.glb.clouddn.com/virtualhost_and_mysql_install_sharefolder.png)
8. 启动
![](http://7xvn6m.com1.z0.glb.clouddn.com/virtualhost_and_mysql_install_10.png)
9. 因为是通过镜像文件新安装，所以选择第二个Install system with basic video driver
![](http://7xvn6m.com1.z0.glb.clouddn.com/virtualhost_and_mysql_install_11.png)
10. 磁盘测试，比较费时间，我一般不选
![](http://7xvn6m.com1.z0.glb.clouddn.com/virtualhost_and_mysql_install_12.png)
11. 这个问题不影响安装，具体有什么后遗症我待再研究研究。。。 `TODO`
![](http://7xvn6m.com1.z0.glb.clouddn.com/virtualhost_and_mysql_install_13.png)
12. 一系列硬件检查完毕
![](http://7xvn6m.com1.z0.glb.clouddn.com/virtualhost_and_mysql_install_14.png)
13. 设置语言，中文就不需要了吧
![](http://7xvn6m.com1.z0.glb.clouddn.com/virtualhost_and_mysql_install_15.png)
14. 键盘设置 US
![](http://7xvn6m.com1.z0.glb.clouddn.com/virtualhost_and_mysql_install_16.png)
15. 仍然是个未接之谜 ![](http://7xvn6m.com1.z0.glb.clouddn.com/blog-img3573956_132.gif) 相信我会更新的。。。
![](http://7xvn6m.com1.z0.glb.clouddn.com/virtualhost_and_mysql_install_17.png)
16. 选择时区，亚洲/上海
![](http://7xvn6m.com1.z0.glb.clouddn.com/virtualhost_and_mysql_install_18.png)
17. 设置启动密码，也就是root密码
![](http://7xvn6m.com1.z0.glb.clouddn.com/virtualhost_and_mysql_install_19.png)
18. 因为我的密码设的太简单了，管他呢，反正测试用
![](http://7xvn6m.com1.z0.glb.clouddn.com/virtualhost_and_mysql_install_20.png)
19. 硬盘分区，只有一块硬盘，只安装了这一个Linux，就选择Use entire drive吧
![](http://7xvn6m.com1.z0.glb.clouddn.com/virtualhost_and_mysql_install_21.png)
20. 确定写入
![](http://7xvn6m.com1.z0.glb.clouddn.com/virtualhost_and_mysql_install_22.png)
21. 开始安装，这个过程是最耗时的
![](http://7xvn6m.com1.z0.glb.clouddn.com/virtualhost_and_mysql_install_23.png)
22. 安装完成重启
![](http://7xvn6m.com1.z0.glb.clouddn.com/virtualhost_and_mysql_install_24.png)
23. 有个基础版本内核和一个更新版内核，我看系统镜像文件里只有基础版的内核rpm包，因为后面安装
virtualBox增强包的时候需要使用kernel/kernel-devel，所以如果选择2.6.39会出现找不到内核源码
这里就选择2.6.32。**这个地方有点糊涂：  Linux对于多个内核如何管理的？ **
![](http://7xvn6m.com1.z0.glb.clouddn.com/virtualhost_and_mysql_install_25.png)
24. 成功进入系统，使用root登录。
![](http://7xvn6m.com1.z0.glb.clouddn.com/virtualhost_and_mysql_install_26.png)
25. 因为后面需要安装VirtualBox增强包，所以需要安装一些开发工具，系统镜像文件里都有，所以这里
挂载系统镜像文件，从系统镜像里安装，点击下面的小光盘，选择镜像文件。
![](http://7xvn6m.com1.z0.glb.clouddn.com/virtualhost_and_mysql_install_28.png)
26. 挂载光盘，
```shell
# 首先待在/media/下面键iso文件夹哈
mount /dev/cdrom /media/iso/
```
26. 配置本地yum源
```shell
# 备份原有yum配置
mv /etc/yum.repos.d/public-yum-o16.repo /etc/yum.repos.d/public-yum-o16.repo.bak
# 创建新的本地yum源
vi /etc/yum.repos.d/local.repo
# 本地yum配置，baseurl后面设置的是镜像文件挂载路径，/media/iso 这个路径后面
[base]
name=local
baseurl=file:///media/iso
gpgcheck=0
enabled=1
# 查看本地yum是否创建成功
yum repolist all
```
28. 安装gcc/make
```shell
yum -y install gcc
yum -y install make
```
29. 下面就开始安装增强包了，第一步挂载增强包镜像文件，因为只创建了一个光驱，所以需要先把系统镜像
卸载掉，在把增强包镜像挂载上
```shell
# 卸载系统镜像
umount /dev/cdrom
# 点击下面的小光盘图标，重新选择增强包的ISO
# 挂载增强包，先在media下建additions文件夹哈
mount /dev/cdrom /media/additions
```
30. 执行安装增强包
```shell
./media/additions/VBoxLinuxAdditions.run
# 然后出现一个什么skip什么的就成功了，如果有问题会打印出让你看日志
```
31. 挂载共享文件夹
```shell
mount -t vboxsf sharefolder /media/
```
31. 重启系统以后(
注意别选错了内核如果嫌每次启动都选择麻烦可以到/boot/grub/grub.cnf里面把default改成你要启动的)
重启以后会在media下重新生成一个sf_sharefolder，这个是因为当创建共享文件夹的时候选择
了自动加载。至此共享文件夹也是OK的了，如果想把宿主机里的数据同步到虚拟机里就可以直接
把文件复制到这个共享文件夹就行。


写完了，还有一些未解之谜![](http://7xvn6m.com1.z0.glb.clouddn.com/blog-img3573956_192.gif)这个后续会更新，好了先自我赞以下![](http://7xvn6m.com1.z0.glb.clouddn.com/blog-img3573956_201.gif)
