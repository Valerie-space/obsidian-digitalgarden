---
{"dg-publish":true,"dg-path":"Lecture/MIT6.5840/mit6.5840-intro","dg-permalink":"mit6.5840-intro","permalink":"/mit6.5840-intro/","title":"MIT6.5840 引言","created":"2026-07-31T20:23:00.117+08:00","updated":"2026-09-02T10:59:19.982+08:00","dg-note-properties":{"title":"MIT6.5840 引言","aliases":[null],"created":"2026-07-31","updated":"2026-09-02 10:59","area":"计算机","type":"study","description":null,"status":"active"}}
---


【草稿】

# MIT6.5840 引言

> 相关：
> - 官方课程笔记：[Lecture 1：Introduction](https://pdos.csail.mit.edu/6.824/notes/l01.txt)
> - 论文 pdf：[MapReduce（2004）](https://pdos.csail.mit.edu/6.824/papers/mapreduce.pdf)

[[领域/学术/文献/MapReduce： simplified data processing on large clusters\|MapReduce： simplified data processing on large clusters]]

## 课程笔记

课程要完成的五个实验分别是：

- Lab 1: distributed big-data framework (like MapReduce)
- Lab 2: client/server vs unreliable network
- Lab 3: fault tolerance using replication (Raft)
- Lab 4: a fault-tolerant database
- Lab 5: scalable database performance via sharding

为什么需要分布式系统？

- parallelism：更高的性能、更高的容量
- fault tolerance: 提供容错（两台计算机运行完全相同的任务，其中一台故障，可以切换到另一台）
- physical（系统在物理上是分布的，天然无法单机完成）：如不同银行系统之间协作、交互。
- security：通过分割计算机集群，限制代码所运行的环境，从而提高每个业务的安全性。
这门课程关注的是性能和容错。

分布式底层基础设施（distructure infrastructure）主要关注**存储（Storage）**、**通信（Commnication）**、**计算（Computation）**。

- 对于存储和计算，我们的目标是设计一些简单接口，让第三方应用能够使用这些分布式的存储和计算。即通过抽象的接口，隐藏分布式的特性。
- 对于通信，我们一般使用已有的通信工具，而不会深入介绍通信系统。

我们需要关注什么？

- 实现（implementation）：RPC（Remote Procedure Call）、threads、concurrency control（并发控制）
- 性能（performance）:
	- 可扩展性（scalability）：增加 N 台服务器，就能增加 N 倍的吞吐量（throughput）
- 容错（fault tolerance），即实现：
	- 可用性（Availability）：即使发生故障，系统仍然能提供服务
	- 可恢复性（Recoverability）：发生故障后，系统停止工作；但故障修复后，系统仍然能够正常运行。
	- 实现容错的工具：
		- 非易失存储（non-volatile storage）。存放 checkpoint 或者系统状态的 log 在 NV storage（如硬盘、闪存、SSD）中，出现故障后，可以从这些地方读出系统的最新状态，并以该状态继续运行。
		- 复制（replication）：使用多个副本。
- 一致性（consistency）

权衡：容错、一致性、性能不可兼得。

- 容错和一致性需要通信。强一致性需要通信开销、从而降低速度；现实中往往通过弱一致性来保证速度。
