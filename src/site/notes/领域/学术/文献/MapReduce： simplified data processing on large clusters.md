---
{"dg-publish":true,"dg-path":"Lecture/MIT6.5840/MapReduce论文笔记","dg-permalink":"mapreduce-paper","permalink":"/mapreduce-paper/","title":"MapReduce： simplified data processing on large clusters","tags":["分布式","MapReduce"],"created":"2026-08-01T02:49:30.222+08:00","updated":"2026-08-23T09:41:33.095+08:00","dg-note-properties":{"title":"MapReduce： simplified data processing on large clusters","aliases":null,"author":["Dean Jeffrey","Ghemawat Sanjay"],"published":2008,"source":"Communications of the ACM","url":"https://dl.acm.org/doi/10.1145/1327452.1327492","citation":null,"created":"2026-08-01","updated":"2026-08-23 09:34","finished":null,"type":"literature","subtype":"article","level":null,"description":null,"status":"active","tags":["分布式","MapReduce"]}}
---


# MapReduce： simplified data processing on large clusters

**DOI**：[10.1145/1327452.1327492](https://dl.acm.org/doi/10.1145/1327452.1327492)

**PDF**：[2008-MapReduce simplified data processing on large clusters.pdf](file:///F:%5CResearch%5CZotero%5CData%5Cstorage%5CE7HDPKR3%5C2008-MapReduce%20simplified%20data%20processing%20on%20large%20clusters.pdf)

>[!note]- 摘要
> MapReduce is a programming model and an associated implementation for processing and generating large datasets that is amenable to a broad variety of real-world tasks. Users specify the computation in terms of a _map_ and a _reduce_ function, and the underlying runtime system automatically parallelizes the computation across large-scale clusters of machines, handles machine failures, and schedules inter-machine communication to make efficient use of the network and disks. Programmers find the system easy to use: more than ten thousand distinct MapReduce programs have been implemented internally at Google over the past four years, and an average of one hundred thousand MapReduce jobs are executed on Google's clusters every day, processing a total of more than twenty petabytes of data per day.

HIDDEN_END

> 相关笔记：
> - [[领域/学术/如何阅读文献——综述篇\|如何阅读文献——综述篇]]
> - [[领域/学术/如何阅读文献——期刊篇\|如何阅读文献——期刊篇]]
> - [[领域/学术/如何阅读文献-里程碑论文\|如何阅读文献-里程碑论文]]

HIDDEN_END

## 阅读流程笔记

### 【背景和动机】

#### 这篇论文解决了当时的什么痛点？

- 即使业务逻辑非常简单（比如词频统计），处理海量数据仍需要在多台机器上分布式运行。因此程序员无法专注于业务逻辑，她们需要写大量底层分布式相关的复杂代码。

#### 在此之前，主流/传统方案有什么缺陷？论文提出了哪些创新点？

对于手写专用分布式程序：

- 缺陷：每个新的业务逻辑都需要重新开始写整套底层代码，无法**复用**、代码量臃肿、容易引入分布式 bug。
- MR：提出函数式编程模型。将复杂计算抽象成两个用户自定义函数（Map 和 Reduce）。底层通信、容错与调度完全由框架托管。

对于传统的并行编程模型（MPI、BSP）：

- 缺陷：1）虽然提供了并行原语，但暴露了大量通信、同步与消息传递细节，编程门槛极高；2）容错能力差，节点故障需要程序员人工介入；3）采用静态数据分区，不支持负载均衡
- MR：1）自动化容错机制（失败任务重试）；2）备份任务（即推测执行），为 Straggler 启动冗余实例。3）通过动态分区实现负载均衡

对于专用的排序/数据库系统（NOW-Sort）

- 缺陷不具备通用的编程能力。（一套专用机器只能用于做排序，而其她逻辑却做不了）
- MR：在 MR 框架之上，程序员能够实现任意能由 Map 和 Reduce 表达的业务逻辑。

对于单机计算模型

- 缺陷：受限于物理内存与 CPU，无法横向扩展。
- MR：支持通过增加 Worker 节点线性扩展算力，突破单机瓶颈。

传统分布式架构：

- 缺陷：调度器只看 cpu 是否空闲，导致海量原始数据必须跨网络搬运。而网络带宽比 cpu 更加昂贵。
- MR：提出数据本地化调度（Data Locality），优先将 Map 任务调度至存有输入数据副本的机器上，大幅削减跨机架网络传输；同时引入 Combiner（合并器），在 Map 端本地执行预聚合后再传输中间结果，进一步节省带宽。

### 【核心思想与方法论】

它的工作原理是什么？

![Pasted image 20260801033819.png](/img/user/%E9%99%84%E4%BB%B6/%E5%9B%BE%E7%89%87/Pasted%20image%2020260801033819.png)

论文将一次 MapReduce 执行划分为 M 个 Map 分片和 R 个 Reduce 分区。以下是基于论文 图 1 和 第 3.1 节 的标准化执行流程：

> 这些流程直接来源于论文，更偏向设计的角度。在 [[领域/计算机/MIT6.5840 引言\|MIT6.5840 引言]] 中，我会通过举例，以实现的角度出发，重新阐述各个阶段。

阶段 0：初始化与拆分（启动）  

- 用户程序调用 MapReduce 函数，库自动将输入数据切分为 M 个数据分片（通常每个 16MB~64MB）。随后，系统在集群中启动多个 Worker 进程，其中一个是 Master（主控节点），其余为 Worker（工作节点）。Master 负责空闲 Worker 的调度，并维护 M 个 Map 任务和 R 个 Reduce 任务的状态。

阶段 1：Map 任务执行（数据读取与映射）  

- 被分配 Map 任务的 Worker 读取对应的输入分片，解析出键值对（Key/Value），并调用用户定义的 Map 函数。Map 函数产生的中间键值对（Intermediate K/V）会先缓存在内存中。

阶段 2：分区与溢写（Partitioning & Local Write）  

- 内存中的中间结果会周期性溢写（Spill）到本地磁盘。在写入前，系统会通过用户指定的分区函数（默认 `hash(key) mod R`）将数据划分为 R 个区域。溢写完成后，Worker 将这些本地文件的位置信息（元数据） 发送给 Master，由 Master 负责将这些位置转发给对应的 Reduce Worker。

阶段 3：混洗与排序（Shuffle & Sort）  

- 当 Reduce Worker 被 Master 通知了 Map 输出的位置后，它通过远程过程调用（RPC） 从 Map Worker 的本地磁盘拉取属于自己的那部分中间数据。数据拉取完成后，Reduce Worker 对所有的中间键进行排序（若数据量过大则使用外部排序），将所有相同键（Key）的数值聚集在一起。

阶段 4：Reduce 任务执行（归约与输出）  

- Reduce Worker 遍历排序后的中间数据。对于每一个唯一的 Key，它将该 Key 和对应的 Value 集合（通过迭代器传入）交给用户定义的 `Reduce` 函数进行处理。Reduce 函数的最终结果会原子地写入到全局分布式文件系统（GFS）上的最终输出文件（每个 Reduce 任务生成一个文件，共 R 个文件）。

阶段 5：完成与返回  

- 当所有 Map 和 Reduce 任务都完成后，Master 唤醒用户程序，MapReduce 调用结束。用户无需合并 R 个输出文件，因为它们可以直接作为下一次 MapReduce 任务的输入。

### 【关键权衡与假设 (Trade-offs & Assumptions)】

#### 系统的假设是什么？（系统的前提）

- 硬件环境：廉价且不可靠。机器故障是常态。
- 数据本地性假设：底层依赖 GFS（Google 文件系统），数据被切分为 64MB 块并在多台机器上复制。
- 工作负载假设：输入数据量大 + 业务逻辑简单且独立。任务主要是扫描全量数据并生成派生数据（如倒排索引、日志统计），不要求毫秒级实时响应，只关注总吞吐量。
- 用户函数假设：用户提供的 `Map` 和 `Reduce` 函数是确定性的（Deterministic），即相同输入永远产生相同输出。

#### 权衡取舍：论文如何进行 trade-offs？获得了什么优势？牺牲了什么？

| 权衡维度        | 牺牲                                                          | 优势                                                |
| ----------- | ----------------------------------------------------------- | ------------------------------------------------- |
| 编程模型        | 强制将计算嵌套在极其受限的 Map-Shuffle-Reduce 三步模型中。无法表达复杂的迭代计算或有状态的流式处理 | 系统可以自动并行化、自动分区、自动负载均衡。降低分布式编程门槛。                  |
| 容错机制（状态恢复）  | 失败任务从头重算，无增量恢复，造成计算资源浪费                                     | 极简的系统设计。Master 无需维护复杂的中间状态日志。“以计算换可靠性”的策略，适合廉价的集群 |
| 计算调度策略      | 调度灵活性受限。Map 任务必须优先等待存储副本所在的特定机器。                            | 节省了稀缺的网络资源                                        |
| 抗落后者机制      | 牺牲额外的 CPU 和内存开销。启动“备份任务”意味着在计算末期，同一份工作最多可能被两台机器同时计算         | 用少量冗余资源换取了对抗“木桶效应”（即落后者拖延整体）的能力。                  |
| Master 的高可用性 | 单点故障。如果 Master 宕机，当前作业直接 Abort（终止）。                          | 设计极度简化。因此放弃复杂的选举机制，换来代码的轻量和维护的便捷。                 |

> **Master 高可用性（High Availability, HA）** 指的是：当分布式系统中的管理核心（Master 节点）发生宕机、网络中断或硬件损坏等故障时，系统能够自动感知并快速恢复，确保整个集群持续提供服务，避免整个系统瘫痪。

#### 系统的边界是什么？（哪些场景不适用）

基于上述假设和权衡，MapReduce 的**天然边界**非常清晰，这也解释了为什么后来被 Spark 等取代（在特定场景）：

- **不适合低延迟交互式查询**：因为 Map 和 Reduce 阶段间的“落盘（写本地磁盘）”和“排序（Sort）”代价高昂，响应时间以分钟计，而非秒或毫秒。
- **不适合迭代计算**：每个 Job 结束后都要写回 GFS，下一轮重新读入，磁盘 I/O 开销巨大（Spark 通过内存 Resilient Distributed Datasets 解决了这一点）。
- **不适合非确定性逻辑**：如果 Map/Reduce 函数有随机性或依赖外部时间，故障重算会导致最终数据不一致。

### 【实验与验证 (Evaluation & Results)】

#### 作者是如何设计实验验证其方案的？使用了哪些 Benchmark、Baseline 和指标？

- 选取了两个 Benchmark：分布式 Grep 和分布式 Sort。
- 选取了两个 Baseline：
	- 针对 Straggler：：将 **Backup Task（备份任务）机制关闭**，观察尾部延迟（Straggler）对总耗时的影响
	- 针对容错：在作业运行中期**主动杀死 200 个 Worker 进程**（约占总数的 11.4%），观察恢复开销
- 并未与第三方系统进行横向对比，而是将自身作为实验的对照组。通过开关关键特性和主动注入故障来验证系统的有效性。

> [!NOTE] Benchmark：基准任务
> Benchmark 是指实验对象；Baseline 是指比较对象。
> 由于论文篇幅有限，不可能枚举所有场景，所以作者选择了两个极端且对立的任务，证明系统在两个极端的资源瓶颈下都能跑的好：
> - **Grep（扫描类）**：**I/O 瓶颈**。几乎不占 CPU，不占网络，只测硬盘能读多快。用来证明“数据本地化”起作用了，能把硬盘读满
> - **Sort（排序类）**：**网络 +CPU 瓶颈**。所有数据都要重新打乱分发（Shuffle）。用来证明“网络调度”和“Combiner”起作用了

### 【局限性与漏洞 (Limitations & Critical Assessment)】

#### 该方案存在哪些明显的短板、边界瓶颈或未解决的问题？

**1. 不支持迭代计算，机器学习场景下效率极低**

MapReduce 的每个作业完成后，中间结果必须写回磁盘，下一轮迭代再重新从磁盘读取。对于需要多轮迭代的机器学习算法（如梯度下降、收敛判断），这种“磁盘落盘 - 重新加载”的模式导致大量无效的磁盘 I/O，性能开销巨大。这被广泛认为是 MapReduce 在机器学习领域最关键的短板。

**2. 高延迟，不适合实时查询与交互式分析**

MapReduce 的设计目标是**批处理吞吐量**，而非**低延迟响应**。Map 和 Reduce 阶段之间的数据需要完整的“落盘 - 混洗 - 排序”流程，整个过程以分钟甚至小时为单位

**3. FIFO 调度策略，缺乏优先级与动态适应性**

传统 MapReduce 的任务调度采用 **FIFO（先来先服务）** 策略，不考虑任务优先级和节点的实时状态 。紧急任务（如电商大促期间的实时数据分析）可能因等待常规任务而延误。在负载均衡方面，静态分配机制难以应对动态负载变化。

**4. 强制 Shuffle 与排序，存在不必要的性能开销**

MapReduce 强制要求所有中间结果在进入 Reduce 阶段前进行**全局排序和分区**。然而，很多计算任务（如简单的聚合、过滤）并不需要全局有序的中间数据。这种设计带来了不必要的网络传输和 CPU 排序开销。

**5. 编程复杂性仍然较高**

尽管 MapReduce 提供了比裸写 MPI 更高层次的抽象，但开发人员仍需深入理解 Map/Reduce 的编程模型、分布式计算原理以及框架的调优参数（如 M/R 数量、内存配置等），编写和调试作业仍然是一项复杂任务

### 【后续】

这篇论文后来（至今）衍生出哪些东西？

- Apache Hadoop：开源的分布式基础架构，由 HDFS（分布式存储） 和 MapReduce（计算引擎） 组成。实现了“计算向数据移动”的核心思想。
- Apache Hive/Apache Pig 等高层框架：直接编写 MapReduce 程序依然复杂。为此，业界在 Hadoop 之上封装了更高层的框架，通过类 SQL 或脚本语言简化开发
- Apache Spark：MapReduce 将中间结果写入磁盘，**效率**较低。Spark 将数据尽可能缓存在内存中，省去大量磁盘 IO。
- Apache Flink：使用流处理引擎，在需要低延迟和**实时**计算的场景中具有优势。

> [!NOTE]
> MapReduce 衍生的技术解决了速度、实时的问题，后来的大数据技术（数据湖、云原生等）已经从“更好的计算”转向“更好地管理、集成、服务于业务”。

---

## 精读

### 英文

名词/术语

- Large Clusters：大规模集群。把很多独立的计算机通过网络连接起来，让它们像一个整体一样协同工作。
- commodity machines：可批量采购的普通商用计算机。
- derived data：通过对原始数据进行加工、计算后得到的派生/衍生数据
- library：在计算机领域中指实现了特定功能的代码集合，即程序库、函数库。
- primitive：在编程语言的语境下，它通常被翻译为“原语”、“基本操作”或者“底层基础构件”
- refinements：原意“改良品”，技术文档中常指对原有模型、方案的细节优化调整。
- invocation：祈祷、召唤、程序调用
- pseudo-code：伪代码
- specification：规范、说明书。
- dual-processor 双处理器
- semantics：语义。semantics in the presence of failures 指的是当系统遇到故障时，程序最终会呈现出什么样的行为表现和计算结果。
- locality 局部性
- replica 复制品。数据的冗余副本

动词

- partition v. 分区、划分。把输入数据划分为多个部分，以便分配到集群的不同机器上并行处理。
- crawl v. 网络爬虫
- conspire v. 指多个不利因素共同作用引发后续问题。
- obscure v. 掩盖
- reduce v. 减少。在数据处理中，相当于把大量数据缩减的过程，即“归约”、“合并”
- emits v. 排放。可指程序输出、返回某个数据结果的动作
- invoke v. 祈求、调用。invoke the function.
- parse v. 对输入的文本、数据按照特定规则进行拆解、转化、处理。（原意：从语法上描述/分析词句）

> [!NOTE] parallelize the computation, distribute the data
> - parallelize 并行化。解决“怎么算”的问题。把大任务拆成可以同时进行的小任务。
> - distribute 分发、分布式处理。解决“数据在哪”的问题。即数据的物理分配。

### Programming Model

Map：由用户编写的函数。处理输入键值对，并生成中间阶段（intermediate）键值对。 

Reduce：还是由用户编写。会把所有 key 值相同的 intermediate 键值对进行合并，生成 0 或 1 个输出值（仍然是 key-value 格式）

其她：

- 分布式文本匹配/搜索（distributed grep）：
	- map 函数会检查每一行输入，如果这一行与用户所给的条件匹配，则会输出这一行。
	- reduce 函数直接把 map 的中间输出作为最终输出。
- url 访问频率统计：
	- map 函数处理网页请求日志，并输出 `<URL,1>`（该 url 被访问一次）
	- reduce 合并所有 url 相同的中间键值对，输出 `<URL, total count>`
- 反向网页链接图（Reverse Web-Link Graph）：即列出链接到 target 网页的所有 source 网页。
	- map 函数记录每个链接：`<target, source>` （source → target）
	- reduce 函数合并所有 target 相同的中间键值对，输出 `<target, list(source)>`
- 每个网站的词向量（Term-Vector per Host）：
	- 词向量 `list(<word,frequency>)` ，即一组单词 - 词频键值对。
	- map 函数输出 `<hostname, term vector>`（hostname 相当于某一个网站，是多个网页的聚合。这一步的 term vector 包含网站的全部单词）
	- reduce 函数合并，并丢弃低频的 term，输出 `<hostname, term vector>`
- 倒排索引（Inverted Index）：快速查找哪些文档包含给定 word
	- map 处理每个文档，输出 `<word, document ID>`
	- reduce 合并、排序，输出 `<word, list(document ID)>`
- 分布式排序（Distributed Sort）
	- 需要额外步骤——分区

### Implementation

环境：

- 机器采用双处理器 x86 架构。运行 Linux 系统。每台机器 2-4GB 内存。
- 采用商用网络硬件。单机网速是 100 Mbps（megabits/second） 或者 1Gbps（1gigabit/second）。不过集群的平均对分带宽要低得多

> [!NOTE] 对分带宽（bisection bandwidth）
> 假设把整个集群的机器分为左右两半，对分带宽是指：连接左、右两半边的所有网线加在一起的总带宽

- 一个集群有成百上千台机器，且机器故障是普遍的
- 数据存储：
	- 用廉价的本地硬盘（IDE disk,但它现在已经被 SATA 和 NVMe 取代）
	- 用 Google 内部开发的分布式文件系统（GFS），该文件系统会把数据放在多个机器上进行备份。
- 用户将 job 提交给调度系统。每个 job 由一组 tasks 组成。调度器会把这一组 tasks 分配给一组可用的机器。

![Pasted image 20260801085426.png](/img/user/%E9%99%84%E4%BB%B6/%E5%9B%BE%E7%89%87/Pasted%20image%2020260801085426.png)

1. MapReduce 把 input files 分成 M 小块
2. 有 M 个 map 任务和 R 个 reduce 任务；一个 Master 节点（进程）和多个 worker 节点。Master 负责把任务分配到空闲的 worker 上。
3. MapReduce 库的解析器从 input date 中提取出键值对，作为 map 函数的输入。worker 运行 map 函数，获得 intermediate 键值对，并缓存在内存中。
4. 内存中的 intermediate 键值对会被定期写入 local disk，并被 partitioning function 划分为 R 个区域。这些键值对在本地磁盘上的存储位置会被传回给 master，由 master 节点负责把这些位置信息发给 reduce worker。
5. reduce work 获得信息后，对 map worker 发起 remote read，读取 local disk 上属于自己分区的那一部分数据。接着对获得的全部 intermediate 键值对进行排序（将相同键的数据放在一块）。
	- 若中间数据量过大，无法存入内存中，则需要进行外部排序
6. 进行 reduce 操作。将 key 相同的键值对归为一个集合，对每个集合进行 reduce 操作（具体逻辑用户自定义）。reduce 的输出结果 append（追加写入）进该 reduce 分区对应的输出文件中。每个 reduce 任务独立生成一个输出文件（共 R 个）
7. 所有任务完成后，master 唤醒用户程序，返回结果。

Master Data Structures：

- master 内部会维护：每个 map task 和 reduce task 的状态（idle、in-progress、completed），以及执行该任务的 worker 标识。
- master 相当于 map 到 reduce 的中介。对于每个已完成的 map 任务，它会存储该 map 任务所生成的 R 个 intermediate file 所在区域的位置和大小。

Fault Tolerance

- Wroker Failure
	- Master 会定期 ping 每一个 worker，如果联系不上：
		- 已完成/进行中的的 map 任务会重新执行（因为 map 任务产出的数据存在 worker 的本地）
		- 进行中的 reduce 任务会重新执行（已完成的 reduce 任务产出的数据存于 global 文件系统，所以无需重新执行）
- Master Failure
	- 通过 checkpoints（检查点）来恢复状态
- Semantics in the Presence of Failures：在故障发生的情况下，MapReduce 算出的结果仍然正确。
	- 依赖 Atomic Commit（原子提交）机制：要么完全不生成最终文件，要么生成完整正确的最终文件。
		- 对于 Map 任务：执行时并不直接生成最终文件，而是生成 R 个以自己 ID 命名的私有临时文件。当 Map 任务彻底处理完所有输入后，Worker 才会发消息（R 个文件名）给 Master。
		- 对于 reduce 任务：Reduce 任务会把结果写入私有临时文件。当 reduce 任务处理完成后，会调用 atomically rename 操作，把这个临时文件改成最终的输出文件名。

Locality
