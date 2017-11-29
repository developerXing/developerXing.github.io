---
layout:     post
title:      集群部署之集群概述和搭建Zookeeper集群
subtitle:   集群部署(一)
date:       2017-03-01
author:     Corliss
header-img: img/mao.jpg
catalog: true
tags:
    - 集群
    - zookeeper
    - 初级
    
---

- 集群概述
- 搭建Zookeeper集群



## 1.集群概述

#### 1.1 集群的概念

集群是一种计算机系统， 它通过一组松散集成的计算机软件和/或硬件连接起来高度紧密地协作完成计算工作。在某种意义上，他们可以被看作是一台计算机。集群系统中的单个计算机通常称为节点，通常通过局域网连接，但也有其它的可能连接方式。集群计算机通常用来改进单个计算机的计算速度和/或可靠性。一般情况下集群计算机比单个计算机，比如工作站或超级计算机性能价格比要高得多。

#### 1.2   集群的特点

集群拥有以下两个特点：

1.   可扩展性：集群的性能不限制于单一的服务实体，新的服务实体可以动态的添加到集群，从而增强集群的性能。
2.   高可用性：集群当其中一个节点发生故障时，这台节点上面所运行的应用程序将在另一台节点被自动接管，消除单点故障对于增强数据可用性、可达性和可靠性是非常重要的。
   
#### 1.3  集群的两大能力

集群必须拥有以下两大能力：

1.     负载均衡：负载均衡把任务比较均匀的分布到集群环境下的计算和网络资源，以提高数据吞吐量。
    
2.     错误恢复：如果集群中的某一台服务器由于故障或者维护需要无法使用，资源和应用程序将转移到可用的集群节点上。这种由于某个节点的资源不能工作，另一个可用节点中的资源能够透明的接管并继续完成任务的过程，叫做错误恢复。
     
**负载均衡和错误恢复要求各服务实体中有执行同一任务的资源存在，而且对于同一任务的各个资源来说，执行任务所需的信息视图必须是相同的。**

#### 1.4  集群与分布式的区别

说到集群，可能大家会立刻联想到另一个和它很相近的一个词----“分布式”。那么集群和分布式是一回事吗？有什么联系和区别呢?

**相同点**：
分布式和集群都是需要有很多节点服务器通过网络协同工作完成整体的任务目标。

**不同点**：
分布式是指将业务系统进行拆分，即分布式的每一个节点都是实现**不同**的功能。集群每个节点做的是同一件事情。

现实生活中例子有很多，例如，这样古代乐队的图就属于集群

![gudai](https://i.imgur.com/K5Ypc4z.png)


而现代乐队这样图就是分布式啦

![xiandai](https://i.imgur.com/kP7rlet.png)


## 2. Zookeeper集群

#### 2.1 Zookeeper集群简介

###### 2.1.1为什么搭建Zookeeper集群

大部分分布式应用需要一个主控、协调器或者控制器来管理物理分布的子进程。目前，大多数都要开发私有的协调程序，缺乏一个通用机制，协调程序的反复编写浪费，且难以形成通用、伸缩性好的协调器，zookeeper提供通用的分布式锁服务，用以协调分布式应用。所以说zookeeper是分布式应用的协作服务。
zookeeper作为注册中心，服务器和客户端都要访问，如果有大量的并发，肯定会有等待。所以可以通过zookeeper集群解决。
下面是zookeeper集群部署结构图：
![zookeeper](https://i.imgur.com/wTS2BbF.png)


###### 2.1.2了解Leader选举

Zookeeper的启动过程中leader选举是非常重要而且最复杂的一个环节。那么什么是leader选举呢？zookeeper为什么需要leader选举呢？zookeeper的leader选举的过程又是什么样子的？

首先我们来看看什么是leader选举。其实这个很好理解，leader选举就像总统选举一样，每人一票，获得多数票的人就当选为总统了。在zookeeper集群中也是一样，每个节点都会投票，如果某个节点获得超过半数以上的节点的投票，则该节点就是leader节点了。

以一个简单的例子来说明整个选举的过程. 
	
假设有五台服务器组成的zookeeper集群,它们的id从1-5,同时它们都是最新启动的,也就是没有历史数据,在存放数据量这一点上,都是一样的.假设这些服务器依序启动,来看看会发生什么 。

	1) 服务器1启动,此时只有它一台服务器启动了,它发出去的报没有任何响应,所以它的选举状态一直是LOOKING状态  
	2) 服务器2启动,它与最开始启动的服务器1进行通信,互相交换自己的选举结果,由于两者都没有历史数据,所以id值较大的服务器2胜出,但是由于没有达到超过半数以上的服务器都同意选举它(这个例子中的半数以上是3),所以服务器1,2还是继续保持LOOKING状态.  
	3) 服务器3启动,根据前面的理论分析,服务器3成为服务器1,2,3中的老大,而与上面不同的是,此时有三台服务器选举了它,所以它成为了这次选举的leader.  
	4) 服务器4启动,根据前面的分析,理论上服务器4应该是服务器1,2,3,4中最大的,但是由于前面已经有半数以上的服务器选举了服务器3,所以它只能接收当小弟的命了.  
	5) 服务器5启动,同4一样,当小弟

#### 2.2搭建Zookeeper集群

###### 2.2.1搭建要求

真实的集群是需要部署在不同的服务器上的，但是在我们测试时同时启动十几个虚拟机内存会吃不消，所以我们通常会搭建伪集群，也就是把所有的服务都搭建在一台虚拟机上，用端口进行区分。
我们这里要求搭建一个三个节点的Zookeeper集群（伪集群）。

###### 2.2.2准备工作

重新部署一台虚拟机作为我们搭建集群的测试服务器。

（1）安装JDK  【此步骤省略】。

（2）Zookeeper压缩包上传到服务器

（3）将Zookeeper解压 ，创建data目录 ，将 conf下zoo_sample.cfg 文件改名为 zoo.cfg

（4）建立/usr/local/zookeeper-cluster目录，将解压后的
Zookeeper复制到以下三个目录
/usr/local/zookeeper-cluster/zookeeper-1
/usr/local/zookeeper-cluster/zookeeper-2
/usr/local/zookeeper-cluster/zookeeper-3

（5） 配置每一个Zookeeper 的dataDir（zoo.cfg） clientPort 分别为2181  2182  2183

修改/usr/local/zookeeper-cluster/zookeeper-1/conf/zoo.cfg

    clientPort=2181
    dataDir=/usr/local/zookeeper-cluster/zookeeper-1/data

修改/usr/local/zookeeper-cluster/zookeeper-2/conf/zoo.cfg

    clientPort=2182
    dataDir=/usr/local/zookeeper-cluster/zookeeper-2/data

修改/usr/local/zookeeper-cluster/zookeeper-3/conf/zoo.cfg

	clientPort=2183
	dataDir=/usr/local/zookeeper-cluster/zookeeper-3/data


###### 2.2.3配置集群

（1）在每个zookeeper的 data 目录下创建一个 myid 文件，内容分别是1、2、3 。这个文件就是记录每个服务器的ID

    -------知识点小贴士------
    如果你要创建的文本文件内容比较简单，我们可以通过echo 命令快速创建文件
    格式为： 
    echo 内容 >文件名
    例如我们为第一个zookeeper指定ID为1，则输入命令
![](https://i.imgur.com/JQo9tCA.png)

（2）在每一个zookeeper 的 zoo.cfg配置客户端访问端口（clientPort）和集群服务器IP列表。

集	群服务器IP列表如下

	server.1=192.168.25.140:2881:3881
	server.2=192.168.25.140:2882:3882
	server.3=192.168.25.140:2883:3883
解释：server.服务器ID=服务器IP地址：服务器之间通信端口：服务器之间投票选举端口

-----知识点小贴士-----

我们可以使用EditPlus远程修改服务器的文本文件的内容，更加便捷
（1）在菜单选择FTP Settings 
   ![](https://i.imgur.com/udMRsJD.png)
 
（2）点击ADD按钮
   ![](https://i.imgur.com/HgdxW5Q.png)
 
（3）输入服务器信息
   ![](https://i.imgur.com/AoXozBg.png)

（4）点击高级选项按钮
   ![](https://i.imgur.com/5HdnTpy.png)

（5）选择SFTP  端口22
   ![](https://i.imgur.com/Q0jZh3m.png)
 
（6）OK  。完成配置
   ![](https://i.imgur.com/2fCDiMW.png)
连接：![](https://i.imgur.com/yg1COxz.png)
![](https://i.imgur.com/xj2B9MV.png)
![](https://i.imgur.com/qfaoXd1.png)

哈哈，无敌啦~~~~   你可能要问 : 坏蛋小兴，你为啥不早告诉我有这一招  ！

###### 2.2.4启动集群

启动集群就是分别启动每个实例。
![](https://i.imgur.com/GCrCTZv.png)
启动后我们查询一下每个实例的运行状态

先查询第一个服务
![](https://i.imgur.com/CtS46pI.png)

Mode为follower表示是跟随者（从）

再查询第二个服务Mod 为leader表示是领导者（主）
![](https://i.imgur.com/mzPKGBY.png)

查询第三个为跟随者（从）
![](https://i.imgur.com/X7T1GMT.png)

###### 2.2.5模拟集群异常

（1）首先我们先测试如果是从服务器挂掉，会怎么样
把3号服务器停掉，观察1号和2号，发现状态并没有变化
![](https://i.imgur.com/KgJGTiE.png)
由此得出结论，3个节点的集群，从服务器挂掉，集群正常

（2）我们再把1号服务器（从服务器）也停掉，查看2号（主服务器）的状态，发现已经停止运行了。
![](https://i.imgur.com/yEusx1P.png)

由此得出结论，3个节点的集群，2个从服务器都挂掉，主服务器也无法运行。因为可运行的机器没有超过集群总数量的半数。

（3）我们再次把1号服务器启动起来，发现2号服务器又开始正常工作了。而且依然是领导者。
![](https://i.imgur.com/uBEWCmU.png)

（4）我们把3号服务器也启动起来，把2号服务器停掉（汗~~干嘛？领导挂了？）停掉后观察1号和3号的状态。
![](https://i.imgur.com/KhDEXgF.png)

发现新的leader产生了~  
由此我们得出结论，当集群中的主服务器挂了，集群中的其他服务器会自动进行选举状态，然后产生新得leader 

（5）我们再次测试，当我们把2号服务器重新启动起来（汗~~这是诈尸啊!）启动后，会发生什么？2号服务器会再次成为新的领导吗？我们看结果
![](https://i.imgur.com/OwMOt9G.png)
![](https://i.imgur.com/nrAJ6gP.png)

我们会发现，2号服务器启动后依然是跟随者（从服务器），3号服务器依然是领导者（主服务器），没有撼动3号服务器的领导地位。哎~退休了就是退休了，说了不算了，哈哈。

由此我们得出结论，当领导者产生后，再次有新服务器加入集群，不会影响到现任领导者。

