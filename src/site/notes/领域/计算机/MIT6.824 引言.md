---
{"dg-publish":true,"dg-path":"Lecture/MIT6.5840","dg-permalink":"mit6.5840-intro","permalink":"/mit6.5840-intro/","title":"MIT6.824 引言","created":"2026-07-31T20:23:00.117+08:00","updated":"2026-08-22T16:59:55.265+08:00","dg-note-properties":{"title":"MIT6.824 引言","aliases":[null],"created":"2026-07-31","updated":"2026-08-22 16:08","area":"计算机","type":"study","description":null,"status":"active"}}
---


# MIT6.824 引言

> 所属领域：[[领域/计算机/计算机\|计算机]]
> 
> 参考：
> - [Introduction](https://pdos.csail.mit.edu/6.824/notes/l01.txt)
> - [MapReduce（2004）](https://pdos.csail.mit.edu/6.824/papers/mapreduce.pdf)

[[领域/学术/文献/MapReduce： simplified data processing on large clusters\|MapReduce： simplified data processing on large clusters]]

关于课程：

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

什么是分布式系统（Distributed System）？

- 一组计算机集群，协同为上层提供服务。

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

## 案例：MapReduce

**背景**：2004 年，Google 需要在 TB 级别的数据上进行大量计算（比如对所有网页创建索引，非常费时），Google 希望将对大量数据的大量运算并行跑在几千台计算机中，快速完成计算。

按照传统方式，工程师在编写业务代码时，还需要花大量时间编写分布式的相关代码。因此 Google 需要一种框架，使得普通工程师只需编写核心代码，而无需考虑分布式计算的工作。因此 MapReduce 出现了。

MapReduce 原理（以单词计数器为例）：

- 目前有输入文件 1、2、3
- MapReduce 为每个输入文件运行 Map 函数。输入数据为文件、输出数据为 key-value 键值对列表（即中间输出，intermediate output）。
	- 若文件 1 有单词 a 和单词 b，则 Map 函数的输出将会是<a,1>（表示 key=a、value=1 的键值对）、<b,1>
	- 若文件 2 有单词 b，Map 函数输出<b,1>
	- 若文件 3 有单词 a、c，Map 函数输出<a,1>、<c,1>
- MapReduce 收集所有 key 值相同的键值对，提交给 Reduce。为它所有 Map 函数输出的每一个 key，都调用一次 Reduce 函数。
![Pasted image 20260803220041.png](/img/user/%E9%99%84%E4%BB%B6/%E5%9B%BE%E7%89%87/Pasted%20image%2020260803220041.png)

> 这就是一个典型的 MapReduce Job。
> - Job。整个 MapReduce 计算称为 Job。
> - Task。每一次 MapReduce 调用称为 Task。
> 所以，对于一个完整的 MapReduce Job，它由一些 Map Task 和一些 Reduce Task 组成。

> [!NOTE] emit
> worker 进程负责实现 emit。
> - 当 map 函数调用 emit，map worker 进程会把数据写入到本地磁盘的文件中。这些文件包含了 map 函数生成的所有 key 和 value
> - 当 Reduce 函数调用 emit，reduce worker 会把输出写入到 Google 使用的共享文件服务中。（即 GFS，一个共享文件服务）

> [!NOTE] GFS
> GFS 是一个共享文件服务，并且它也运行在 MapReduce 的 worker 集群的物理服务器上。GFS 会自动拆分你存储的任何大文件，并且以 64MB 的块存储在多个服务器之上。所以，如果你有了 10TB 的网页数据，你只需要将它们写入到 GFS，甚至你写入的时候是作为一个大文件写入的，GFS 会自动将这个大文件拆分成 64MB 的块，并将这些块平均的分布在所有的 GFS 服务器之上

### 流程

**前提**：MapReduce 假设输入数据是静态的、全量的。必须 Map workers 将所有输入数据全都处理结束后，才能进行下一步（reduce）。

换句话说，它采用批处理（batch）方式，而不是实时（real-time）或流（streaming）

> [!NOTE]
> 由于 map 和 reduce 是分段进行的。在 MapReduce 论文设计里，所有 Worker 节点是完全对等的。map worker 和 reduce worker 的区分仅仅在于“当前这个 worker 正在做什么”。也就是说，同一个 worker 既可以做 map 工作，也可以做 reduce 工作。

**初始数据**：两份文本文件：File1: `"hello world"`，File2: `"hello mapreduce"`

**设置参数**：

- M=2（2 个 Map 任务）
- R=2（2 个 Reduce 任务），
- 哈希分区函数：`hash(key) mod R`）（来决定 Key 发往哪个 Reduce）
- 假设哈希值：hash("hello")%2=0, hash("world")%2=1, hash("mapreduce")%2=0,

> [!NOTE]
> 这里 M 和 R 分别表示 Map 任务的总数和 reduce 任务的总数，它们并不取决于 worker （即机器）的数量。
> - worker 的数量决定了同一时间能并行跑多少任务
> - map task 的数量=有多少任务需要跑，它由输入数据的物理大小决定。例如有 1TB 的文件，按 64MB 进行分片，则有 1TB/64MB 个 map 任务。论文建议 M 远大于集群中 Worker 机器的数量。
> - reduce task 的数量由用户指定（即输出文件的数量）。论文中给出了两个约束：
> 	- 上限：Master 需要维护 O(M * R) 的内存状态（每个 Map/Reduce 对约 1 字节），因此 R 不能过大导致 Master 内存溢出。
> 	- 下限：通常设为 Worker 机器数量的一个小倍数（例如预期使用 2000 台机器，R 设为 5000）。这样每个 Reduce 任务处理的数据量适中，输出文件数量也便于后续处理。
> 

---

**阶段一：输入分片（Input Splits）与数据本地化调度 (即任务分配)**

1. 物理存储分块（GFS）：输入大文件在写入存储系统时，已被分布式文件系统自动按固定大小（如 64MB）物理切分为多个数据块（Block），并分散存储在不同节点的本地磁盘上
	- Block 0 在 Worker A 的本地硬盘。内容为 `"hello world"`，
	- Block 1 在 Worker B 的本地硬盘，内容为 `"hello mapreduce"`）。

2. 逻辑分片映射（Master）：Master 向文件系统查询，在内存中将这些 Block 映射成逻辑上的 Input Splits（即 Map Task）。以 Hadoop 为例，InputSplit 对象的真实数据结构如下：

```java
public class FileSplit extends InputSplit {
    private Path file;         // 文件路径，例如: hdfs://cluster/logs/2026-08-14.log
    private long start;        // 本分片的起始字节偏移量 (Offset)，例如: 67108864 (64MB)
    private long length;       // 本分片的字节长度 (Length)，例如: 67108864 (64MB)
    private String[] hosts;    // 存储该 Block 副本的节点 IP (用于数据本地化调度)
}

```

3. Master 遵循**数据本地化**原则，将 Map Task 1 分配给 Worker A，将 Map Task 2 分配给 Worker B。数据无需跨网络传输，直接从本地磁盘读取。

---

阶段二：Map 阶段

**Map Task 1（Worker A）的处理流程**（要注意执行主体。MR框架是底层框架的行为，Map函数是用户的行为）

1. MR框架将 Block 数据块解析为键值对：key="0"（表示偏移量）,value="hello world"
2. MR框架执行用户定义的 Map 函数，由Map遍历每个单词，并 emit 中间键值对：
	- `EmitIntermediate("hello", "1")`
	- `EmitIntermediate("world", "1")`
	- 每次Emit动作发生时，MR框架会调用哈希分桶函数，用于决定该条记录应该写入本地磁盘上的哪个分区文件。框架将元组 `(partition, key, value)` 写入内存缓冲区（In-Memory Buffer）。
		- `hash("hello") % 2 = 0` → 进入分区 0
		- `hash("world") % 2 = 1` → 进入分区 1
3. 内存缓冲区中的中间键值对，会被周期性地由 buffered 机制刷新写到本地磁盘的中间文件中。（写入前会先按照中间键值对的所属分区，对中间键值对进行排序）
**Map Task 2（Worker B）的处理流程**：略

> [!NOTE] 为什么解析 Block 时，需要解析成键值对，且 key 用偏移量表示？
> 当然可以把整个 Block 数据块作为一个超级大的 String 传给 Map，但是这极易引发内存溢出。因此 MR 框架往往按行、流式的把整个 Block 喂给 map 函数，处理完一行数据就丢弃一行。因为是流式的数据，所以用偏移量来表示起始位置

> [!NOTE] Combiner（合并器）的优化效果
> 
> 在这个极小数据集中，每个 Map 任务中每个单词只出现一次，因此 Combiner（局部预聚合）不会产生任何实际合并效果。但如果在更大的文件中（例如 File1 内容为 `"hello hello world"`），Worker-A 会在本地先将两个 `<"hello", 1>` 合并为 `<"hello", 2>`，再通过网络发送，从而显著减少网络传输量。
> 

**Map 任务完成**：

- Worker-A 将本地磁盘上分区 0 和分区 1 的中间文件位置报告给 Master。
- Worker-B 将本地磁盘上分区 0 的中间文件位置（该 Worker 没有分区 1 的数据）报告给 Master。
- Master 存储这些中间文件位置信息。一旦某个 Reduce 任务被调度到某个 Worker 上，Master 会主动将对应的 Map 输出位置通知给该 Reduce Worker
- Reduce Worker 收到通知后，再主动发起 RPC（远程过程调用），去 Map Worker 的本地磁盘拉取（Pull）属于自己的那部分分区数据。

---

阶段三：Shuffle（混洗）与 Sort（排序）

**Reduce Worker-0（负责处理分区 0，即 R=0）**：

1. 通过网络 **RPC**，从 **Worker-A** 和 **Worker-B** 的本地磁盘拉取所有属于**分区 0** 的中间数据：
	- 来自 Worker-A：`<"hello", "1">`
	- 来自 Worker-B：`<"hello", "1">` 和 `<"mapreduce", "1">`
2. 排序：Reduce 0 将拉取到的两股数据流合并成一个按 Key 全局排序的大文件。合并后的内存/磁盘数据呈现为：
  `[("hello", 1), ("hello", 1), ("mapreduce", 1)]` （按字典序排列，相同的 Key 紧挨着）。
3. **分组**（预处理）：Reduce Worker 在排序完成后，遍历排序好的列表，将相同 Key 的 Values 封装成一个迭代器（Iterator），即转化为逻辑上的 `(Key, list(Values))` 格式，例如 `("hello", [1,1])` 和 `("mapreduce", [1])` 

> [!NOTE]
> 数据在物理上是有序的键值对，逻辑上通过迭代器呈现为列表形式。
> 
> 什么是迭代器？
> - 迭代器是一个指针，它提供了同一的方法，让用户一个一个地取出容器中的下一个元素，而忽略了元素是什么（数组/列表/键值对，等）。
> 
> 什么是“封装成迭代器”？
> - 键值对的物理形态为 `[("hello", 1), ("hello", 1), ("mapreduce", 1)]`。如果不封装，则取出 key="hello" 的键值对时，会复制两次 `("hello", 1)`。而框架会创建一个迭代器对象。这个迭代器记录的是：
> 	- 当前 key 的值：`"hello"`
> 	- 指向第一个 `("hello", 1)` 的读指针（起始位置）
> 	- 指向 `("mapreduce", 1)` 之前的边界指针（结束位置）
> 	- 当调用 `value()` 时，迭代器根据当前的读指针，直接从原数据所在位置中读出。内存中始终只存有一条正在处理的数据。
> - 如果不封装，则内存有可能溢出。

---

阶段四：Reduce 阶段

1. Reduce Worker-0 执行用户定义的 Reduce 函数：
	- 对于键 `"hello"`，传入迭代器 `[1,1]`，在 Reduce 函数中累加：`result = 1 + 1 = 2`。发出最终结果 `Emit("hello", 2)`，将键值对写入临时文件
	- 对于键 `"mapreduce"`，传入迭代器 `[1]`，累加：`result = 1`。发出最终结果 `Emit("mapreduce", 1)`，将键值对追加（Append）写入同一个临时文件。
2. 当所有 Key 都遍历完毕，执行一次原子重命名（Atomic Rename）操作，正式生成为最终输出文件 `part-r-00000`（Hadoop 的惯用命名）。

---

阶段五：结果呈现

Master 唤醒用户程序。用户得到 **R=2** 个输出文件，内容分别为：

`part-r-00000`（由 Reduce 0 产出）：

  ```bash
  hello 2
  mapreduce 1
  ```

`part-r-00001`（由 Reduce 1 产出）：

  ```bash
  world 1
  ```

---

### 关注点

MR 如何优化网络通信？

1. 数据本地性优化（Locality Optimization）
	- Master 会优先将 Map 任务调度到包含该输入数据副本的机器上，减少网络传输。
2. Map 端本地聚合（Combiner）
	- Map 任务在将数据写入本地磁盘之前，先对相同 Key 的中间结果进行合并。减少了跨网络的中间数据量。

MR 如何实现负载均衡（load balance）？

1. 细粒度任务划分（Fine-grained Task Granularity）
	- M 和 R 远大于集群中的 worker 数量。这样每个 worker 执行多个任务，性能快的机器执行更多任务、慢的执行更少。
2. 动态调度（Dynamic Scheduling）
	- Master 维护所有任务的状态（idle/in-progress/completed）。它会动态地将空闲任务分配给空闲的 worker。
3. 备份任务（Backup Tasks）
	- 当计算接近完成时，Master 会为剩余未完成的 in-progress 任务启动备份执行（即 speculative execution）。- 无论是主执行还是备份执行，谁先完成，该任务就标记为完成。这有效解决了因磁盘坏道、CPU 争用等导致的个别机器“拖后腿”问题，直接均衡了最终完成时间。

MR 如何实现容错（fault tolerance）？（有哪些可能出现的错误？如何解决？）

1. Worker 故障（Worker Failure）
	- 表现：机器宕机、网络不可达、进程崩溃。
	- 检测方式：Master 会周期性 ping 每个 worker。若在一段时间内未收到响应，则将该 worker 标记为 failed。
	- 处理方式：
		- 已完成（completed）的 Map 任务：需要重新执行。因为 Map 的输出存储在故障机器的本地磁盘上，无法访问。这些任务会被重置为 idle 状态，重新调度给其他 worker。
		- 已完成（completed）的 Reduce 任务：不需要重新执行。因为 Reduce 的输出存储在全局文件系统（GFS）中，其他机器可访问。
		- 正在执行（in-progress）的 Map 或 Reduce 任务：同样重置为 idle，重新调度。

2. Master 故障（Master Failure）
	- 处理方式：论文提到，Master 会定期将数据结构写为检查点（checkpoint）。如果 Master 进程死亡，可以从最后一个检查点恢复。
	- 当前实现：但由于只有一个 Master，其故障概率很低，论文中实际的实现是如果 Master 失败，则中止整个 MapReduce 计算，由客户端决定是否重试。

3. 语义一致性保证（处理重复执行和原子性）
	- 由于上述故障会导致任务重试（尤其是 Map 任务可能被多个 Reduce 读，或同一 Reduce 被执行多次），MapReduce 依赖原子提交（atomic commit）来保证结果正确性。
	- 机制（论文 3.3 节）：
		- 每个 Map 任务产生 R 个临时文件（对应 R 个 Reduce 分区），完成后将文件名通知 Master。
		- 每个 Reduce 任务将其输出写入临时文件，任务完成后原子性地 rename 为最终输出文件。
		- 依赖底层文件系统（GFS）的原子 rename 操作，确保即使同一 Reduce 任务在多台机器上执行多次，最终文件系统状态中也只有一次成功执行的结果。

4. 用户代码导致的“坏记录”（Bad Records）
	- 表现：用户 Map 或 Reduce 函数在处理某条特定记录时，因 bug 导致崩溃（如段错误、总线错误），且反复执行都会崩。
	- 解决方式（论文 4.6 节）：
		- Worker 安装信号处理器（捕获 segmentation violations 等）。
		- 调用用户函数前，记录当前处理记录的序列号。
		- 若崩溃，信号处理器会发送包含该序列号的 UDP “last gasp” 包给 Master。
		- 当 Master 发现同一条记录失败超过一次，会在后续重新执行该任务时指示 worker 跳过该记录，从而让整个作业继续推进（适用于统计分析等可容忍少量丢失的场景）。

5. “落后者”（Stragglers）
	- 表现：磁盘坏道（读速从 30MB/s 降到 1MB/s）、CPU 缓存被禁用、被其他任务抢占资源等。
	- 解决方式：即上述的 “备份任务（Backup Tasks）”机制（3.6 节），本质上是一种针对“性能故障”的容错手段。
