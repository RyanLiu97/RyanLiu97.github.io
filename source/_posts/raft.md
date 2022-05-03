---
title: raft
date: 2022-05-03 15:10:29
tags:
---
# Raft 共识算法

## 背景

在分布式场景中, 即使机器发生故障的概率很小, 但当主机数很大时(如成千上万时), 总会有个别的主机会出现故障. 为了保证在个别机器故障时, 整个分布式系统依然能够正常对外服务, 这就需要通过一些冗余的机制去实现. 这被称为分布式系统的容错性(Fault Tolerance). 
主要的办法就是通过"复制"(Replication)的手段, 复制一份或多份状态(数据)的副本, 并分布到不同的机器当中, 当机器故障时, 存有副本的机器就能及时上线(Go lives), 接替故障机器的工作.
主要的Repliacation有两种类型:
##### State Transfer : 
直接复制内存的数据, 对整个数据副本进行拷贝, 通常消耗较大.
##### Replicated State Machine
对外部事件进行复制, 如外部的读写时间. 根据状态复制的层级不同, 主要有两种做法:
- 复制应用层面的状态. 如GFS
- 复制机器层面的状态, 完全复制一台机器的寄存器, 中断信号等. 被称为虚拟机(Virtual Machine)

目前主要的技术都集中在Replacated State Machine 上. 主要的难点在于, 如何保证主机Primary与从机replica(s)之间状态的一致性?(Consistency).
### 一致性问题发生在哪?
假设下图的一个场景. 拥有一主一从的分布式系统中, 假设有两个Client(C1, C2)分别对变量x进行写入操作Wx, C1将1写入x中: Wx1, C2将2 写入x中: Wx2. 在一个坏的设计中, 如果不经过一致性的保护, C1, C2仅仅只是分别的将两个请求同时发送到主从服务器中. 但是由于网络的不确定性, 并不能保证Wx1与Wx2两个操作到达Server1与Server2的顺序是否一致. 若顺序不一致, 就会导致主从服务器X变量的值不同, 这就产生了**不一致性**.

而Raft算法就是一种解决主从复制一致性的分布式共识算法.

## Raft 分布式共识算法

>论文链接
>Diego Ongaro and John Ousterhout. 2014. In search of an understandable consensus algorithm. In <i>Proceedings of the 2014 USENIX conference on USENIX Annual Technical Conference</i> (<i>USENIX ATC'14</i>). USENIX Association, USA, 305–320.
>[https://pdos.csail.mit.edu/6.824/papers/raft-extended.pdf](https://pdos.csail.mit.edu/6.824/papers/raft-extended.pdf)

### 提出背景与Paxos

... 待完善


#### Raft算法基础
对于任何一个时刻, 每个服务器只能以下三种状态之一: 
- Leader
- Follower
- Candidate
通常情况下只有1个Leader, 剩下的服务器都是Follower. Follower只被动的接收来自Leader和Candidate的请求,而自身不发出请求. Leader处理所有来自客户端的请求, (若客户端请求了一个Floower, 其会重定向至其Leader).
