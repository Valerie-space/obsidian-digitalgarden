---
{"dg-publish":true,"dg-path":"Lecture/MIT6.5840/mit6.5840-lab1","dg-permalink":"mit6.5840-lab1","permalink":"/mit6.5840-lab1/","title":"MIT6.5840 Lab 1：MapReduce","created":"2026-08-04T12:14:28.256+08:00","updated":"2026-08-25T22:16:34.670+08:00","dg-note-properties":{"title":"MIT6.5840 Lab 1：MapReduce","aliases":[null],"created":"2026-08-04","updated":"2026-08-22 18:05","area":"计算机","type":"study","description":null,"status":"active"}}
---


# MIT6.5840 Lab 1：MapReduce

> 相关：

实验目标：建立一个 MapReduce 系统。包括 Worker 进程和 Coordinator 进程。

> 实验中以 “coordinator” 来替代 paper 中的 “master”

实验环境：

- WSL2： [[技能/WSL2+OpenClaw安装记录\|WSL2+OpenClaw安装记录]]
- Go： [6.5840 Go](https://pdos.csail.mit.edu/6.824/labs/go.html) 。

> [!NOTE]
> 通过 git 拉取仓库时，不能直接拉取到 Windows 的盘符下，必须拉取到 WSL2 的文件系统中（如 `\\wsl.localhost\Ubuntu-24.04\home\username`）。
> 
> 如果是在 Windows 中拉取（则通过 `/mnt/c` 访问文件），WSL2 访问 Windows 的 NTFS 文件系统，本质上是通过 9p 协议（一种网络文件系统协议）进行转义的，IO （如文件读写）性能极差。

## Go 

- [[技能/Go语言安装-WSL2环境（宝宝巴士版）\|Go语言安装-WSL2环境（宝宝巴士版）]]
- [[领域/计算机/Go语言\|Go语言]]

## 开始

打开 VS Code，下载插件：`WSL`、（选择下载量最高的就行，无需额外配置）

> WSL 扩展将 VS Code 拆分为“客户端 - 服务器”体系结构，客户端（用户界面）在 Windows 计算机上运行，服务器（代码、Git、插件等）在 WSL 分发版中“远程”运行。

创建实验文件夹：

```bash
mkdir MIT6.5840
cd MTI6.5840
```

克隆库：

```bash
git clone git://g.csail.mit.edu/6.5840-golabs-2026 6.5840
code 6.5840    #用VSCode打开项目，出现Installing VS Code Server for Linux x64 是正常的
```

检查 VSCode 是否连接到了 WSL 远程环境：

- 查看左下角 `><` 是否有显示远程服务器的名称，如我的是 `WSL:Ubuntu-24.04`
- 或者打开终端 (快捷键 `ctrl+shift+反引号`)，查看命令提示符是否和 WSL2 中的一致。如我的是 ` username@ROG :~/MIT6.5840/6.5840$ `

在 VSCode 的扩展商店安装扩展：

- Go（必须）
- Code Runner（可选）
- 你喜爱的 ai 插件（可选）

打开终端，输入 `go version` ，确认 Go 环境已正确配置。

---

Trae

1. 在 Windows 中直接打开 Trae
2. 点击左侧活动栏的“远程资源管理器”
3. 在“WSL 目标”下能看到自己的 Ubuntu 发行版，点击“在新窗口中连接”
4. 在新窗口中打开项目文件夹

Trae 会提示“未安装 go 所需的扩展，是否需要安装？”，点击安装扩展即可。

> [!NOTE]- 若 dlv 安装错误
> 
> 如果出现该错误：
> ```bash
> 2026-08-09T00:38:44.363+08:00 [info] 1 tools failed to install.
> 
> 2026-08-09T00:38:44.363+08:00 [info] dlv: failed to install dlv(github.com/go-delve/delve/cmd/dlv@latest): ExecaError: Command failed with exit code 1: /usr/local/go/bin/go install -v 'github.com/go-delve/delve/cmd/dlv@latest'
> 
> go: github.com/go-delve/delve/cmd/dlv@latest: module github.com/go-delve/delve/cmd/dlv: Get "https://proxy.golang.org/github.com/go-delve/delve/cmd/dlv/@v/list": dial tcp 142.251.33.209:443: i/o timeout
> ```
> 说明网络连接超时，无法安装 `dlv` 调试器。只需修改 `GOPROXY` 即可。
> 
> 执行：
> ```bash
> go env -w GOPROXY=https://goproxy.cn,direct
> ```
>
> 关闭当前终端（输入 `exit` 退出）
> 重新打开一个 WSL 终端，在新的终端中输入 `go env GOPROXY`，确认代理已生效
> 
> 在 WSL 终端中尝试访问国内镜像：
>
> ```bash
> curl -I https://goproxy.cn
> ```
>
> 如果返回 `HTTP/1.1 200 OK`，说明网络正常，可以继续安装。
> 
> 执行：
> ```bash
> go install github.com/go-delve/delve/cmd/dlv@latest
> ```

---

trae 中的快捷操作：

- F12：跳转到定义

---

git 操作后

1. 初始化

```bash
cd /home/username/MIT6.5840/6.5840
git init
```

## 了解目录结构

代码包含了该课程的所有实验。

Lab1：MapReduce

|路径|角色|说明|
|---|---|---|
|src/mr/|**核心实现**|Coordinator、Worker、RPC 接口定义（你需要写代码的地方）|
|src/mrapps/|测试插件|WordCount、Indexer、Crash、Timing 等 Map/Reduce 插件（作为测试用例）|
|src/main/mrsequential.go|入口|单机版 MapReduce（参考基线）|
|src/main/mrcoordinator.go|入口|分布式 Coordinator 启动入口|
|src/main/mrworker.go|入口|分布式 Worker 启动入口|
|src/main/pg-*.txt (8 个)|测试数据|8 本英文小说（作为 WordCount 等任务的输入数据）|

Lab2：单机 KV 服务 + 基于 RPC 的分布式锁，在不可靠网络下的行为

|路径|角色|说明|
|---|---|---|
|src/kvsrv1/|**核心实现**|单机 KV 服务（Get/Put，含 Version 版本号机制）|
|src/kvsrv1/rpc/|RPC 定义|KV 服务的 RPC 请求/响应结构体|
|src/kvsrv1/lock/|**核心实现**|基于 KV 存储实现的分布式 Lock 服务|
|src/main/kvsrv1d.go|入口|KV 服务器启动入口|
|src/main/lockd.go|入口|Lock 服务器启动入口|
|src/main/lockc.go|入口|Lock 客户端命令行工具|

Lab3：实现完整的 Raft 一致性协议

|路径|角色|说明|
|---|---|---|
|src/raft1/|**核心实现**|Raft 算法主体（你需要写大量代码的地方）|
|src/raftapi/|接口定义|Raft 需要实现的 API 接口（供 tester 调用）|
|src/main/raft1d.go|入口|单节点 Raft 服务器启动入口（调试用）|

Lab4：基于 Raft 构建 RSM 的容错 KV 数据库

|路径|角色|说明|
|---|---|---|
|src/kvraft1/|**核心实现**|容错 KV 服务器（Get/Put 通过 Raft 同步）|
|src/kvraft1/rsm/|**核心实现**|Replicated State Machine 框架（把 Raft 和 KV 逻辑解耦）|
|src/main/kvraft1d.go|入口|容错 KV 服务器启动入口|
|src/main/rsm1d.go|入口|RSM 服务器启动入口|

Lab5：将数据分片到多个 Raft 组，实现水平扩展。

|路径|角色|说明|
|---|---|---|
|src/shardkv1/|**核心实现**|分片 KV 系统顶层目录和测试|
|src/shardkv1/shardcfg/|子模块|分片配置（sharding configuration）的定义和工具|
|src/shardkv1/shardctrler/|**核心实现**|分片控制器（shard master / controller），管理分片分配|
|src/shardkv1/shardgrp/|**核心实现**|分片组服务器（每个 Raft 组负责部分分片）|
|src/shardkv1/shardgrp/shardrpc/|RPC 定义|分片组之间的迁移/通信 RPC 结构体|
|src/main/shardgrp1d.go|入口|分片组服务器启动入|

跨 Lab 共享的工具库：

| 路径                       | 作用                                      | 被哪些 Lab 使用  |
| ------------------------ | --------------------------------------- | ----------- |
| src/labrpc/              | 自定义 RPC 框架（支持模拟网络延迟、丢包、重排）              | Lab 2/3/4/5 |
| src/labgob/              | Go gob 序列化的封装（用于 Raft 持久化、快照）           | Lab 3/4/5   |
| src/tester1/             | 测试框架（模拟节点崩溃重启、网络分区、客户端管理）               | Lab 2/3/4/5 |
| src/tester1/demux/       | 多路复用器（支持多客户端并发测试）                       | Lab 3/4/5   |
| src/tester1/sockrpc/     | 基于 Socket 的 RPC（真正跨进程通信）                | 全部（可选）      |
| src/models1/             | Porcupine 线性一致性验证模型（KV 规范定义）            | Lab 2/4/5   |
| src/kvtest1/             | KV 测试 + 线性一致性检查（调用 models1 + porcupine） | Lab 2/4/5   |
| src/models1/kv.go        | KV 系统的 " 正确性规范 "                           | Lab 2/4/5   |
| src/kvtest1/porcupine.go | 记录操作历史并调用 Porcupine 检查                  | Lab 2/4/5   |
| src/kvtest1/kvtest.go    | KV 压力测试生成器                              | Lab 2/4/5   |
| src/go.mod, src/go.sum   | Go 模块依赖（porcupine 等）                    | 全部          |
| src/Makefile             | 编译/测试目标定义                               | 全部          |

遗留文件：

以下文件引用了 **当前目录中不存在的包**，是旧版 MIT 6.824 的遗留代码，**在本课程版本（6.5840 2026）中已不再使用**：

|文件|引用的缺失包|推测历史归属|
|---|---|---|
|src/main/viewd.go|6.5840/viewservice|旧版 Lab 2 的 ViewServer（视图服务，成员管理）|
|src/main/pbd.go|6.5840/pbservice|旧版 Lab 的 Primary-Backup 复制服务|
|src/main/pbc.go|6.5840/pbservice|旧版 pbservice 的客户端|
|src/main/diskvd.go|6.5840/diskv|旧版 Lab 的磁盘持久化服务器（可能和旧版 shardkv 有关）|

---

其她文件：

- `go.sum` 是一个哈希校验文件，它记录了项目所依赖的第三方包的加密哈希值（SHA-256）
- `go.mod` 用于定义项目的模块信息和直接/间接**依赖**的库版本。它的角色类似于 Node.js 的 `package.json` 或 Python 的 `requirements.txt`。

```go
module github.com/yourname/myproject // 1. 项目的唯一标识路径，告诉 Go 编译器其她地方如何 import 当前项目的代码。

go 1.22                              // 2. 建议的最低 Go 编译器版本

require (                            // 3. 项目依赖的库及其版本
	github.com/gin-gonic/gin v1.9.1
	golang.org/x/crypto v0.21.0 // indirect  // 间接依赖（即依赖的依赖）
)

replace (                            // 4. 替换依赖（用于本地调试或替换被墙的包）
	golang.org/x/crypto => github.com/golang/crypto v0.21.0
)

exclude (                            // 5. 禁用某些特定版本的依赖
	github.com/foo/bar v1.2.3
)
```

## 实验指导

### 测试与提交

有两个 Makefile：

1. `6.5840/Makefile`：做完实验要提交作业到 Gradescope 时，它会帮你在本地检查编译，并把源码打包成合格的压缩文件。
使用：
打包命令：`make [lab1|lab2|lab3a|lab3b|lab3c|lab3d|lab4a|lab4b|lab4c|lab5a|lab5b|lab5c]`。如命令 `make lab1` 能够生成 lab1-handin.tar.gz。将该压缩包上传到 Gradescope 即可。

```bash
make lab1    # 如果你做的是 Lab 1 (MapReduce)
make lab2    # 如果你做的是 Lab 2 (Raft)
make lab3a   # 如果你做的是 Lab 3a
```

执行成功后，目录下会生成一个类似 **`lab1-handin.tar.gz`** 的压缩包，把它下载下来上传到 Gradescope 即可。

2. `6.5840/src/Makefile`：专门用于本地开发与测试，负责在本地编译代码、运行测试用例
使用：
- 跑某个实验的全部测试：`make [mr|kvsrv1|lock1|raft1|rsm1|kvraft1|shardkv]` （如 `make mr` 用于编译和测试名为 mr 的实验）
- 跑某个特定的测试：使用 `RUN="-run Wc"` 可以单独运行名为 Wc 的测试。例如运行 `$ make RUN="-run Wc" mr` 会只执行 `mr` 实验中名称包含 Wc 的测试用例。

### 实验难度

- Easy：几个小时
- Moderate：大约六个小时
- Hard：超过六个小时

每个实验的代码量大约是几百行。

### 提示

需要学习：

- Go：
	- 需要完成：[A Tour of Go](https://go.dev/tour/welcome/1)
	- 遇到问题时查询：[Effective Go - The Go Programming Language](https://go.dev/doc/effective_go)
	- Go 的格式化打印字符串 `Printf`： [Go format strings](https://golang.org/pkg/fmt/).
- Git 版本控制：[Pro Git book](https://git-scm.com/book/en/v2) 或者 [git user's manual](http://www.kernel.org/pub/software/scm/git/docs/user-manual.html).
- Debugging 的时候记得做笔记，这样可以防止自己忘记之前踩过的坑。
- 多加 print 语句来定位错误
- 在写代码的时候，给代码添加显式条件检查（可以使用 `panic`）

## 试着执行

> 对于本次实验以及后续的所有实验，我们可能会更新提供的代码。为了确保你能顺利拉取这些更新，并用 `git pull` 轻松合并，最好把我们提供的代码保留在原始文件中。你可以按照实验文档的指示在原有代码上添加内容，但千万别挪动它们的位置。当然，把你自己的新函数写在单独的新文件里是完全可以的。

- `src/main/mrsequential.go` 是一个简单的顺序版 MapReduce 实现。它在单个进程中，一个接一个依次执行 Map、Reduce 操作。
	- 输入文件为 `pg-xxx.txt`
	- 输出文件为 `mr-out-0`
- 实验提供了几个 MapReduce 应用示例：
	- `mrapps/wc.go`：词频统计
	- `mrapps/indexer.go`：文本索引器

可以按照以下方法来运行词频统计程序

```bash
$ cd ~/6.5840
$ cd src/main
$ go build -buildmode=plugin ../mrapps/wc.go  #编译到当前目录下，生成wc.so
$ rm mr-out*    #清理旧输出
$ go run mrsequential.go wc.so pg*.txt #编译并运行单机顺序版的MR框架主程序，传入刚刚编译好的词频统计插件，并传入待处理的输入文本文件。
$ sort mr-out-0
```

> [!NOTE] 为什么不全用 go build 或者全用 go run？
>
> | 特性         | `go run`                     | `go build`                 |
> | ---------- | ---------------------------- | -------------------------- |
> | **主要用途**   | **开发调试、快速验证代码**              | **项目构建、部署、正式运行**           |
> | **产物文件**   | 不会在当前目录留下任何可执行文件             | 在当前目录（或指定目录）生成**可执行二进制文件** |
> | **编译机制**   | 在临时目录编译 → 立即执行 → 自动清理临时二进制文件 | 编译并输出结果文件到磁盘               |
> | **二次运行速度** | 每次都需要重新走一遍编译 + 运行过程            | 直接运行生成的二进制文件（极快）           |
> | **依赖环境**   | 目标机器必须安装 Go 环境               | 运行机器**不需要安装 Go 环境**        |
>
> - `wc.go` 不是一个独立运行的程序，它里面没有 `main()` 函数。因此这里的目标是把它编译成一个 `.so` 格式的动态链接库（插件）。 `go run` 的本质是“编译并**立即执行 `main` 函数**”。既然 `wc.go` 没有 `main` 函数，直接 `go run` 会直接报错。必须用 `go build` 并加上指定参数 `-buildmode=plugin` 才能把它打包成插件文件 `wc.so`。
> - `mrsequential.go` 是包含 `main()` 函数的主函数。用 `go run` 可以一步编译 + 执行 + 清理。当然也可以全部用 `go build`：
> ```bash
> # 1. 编译 MapReduce 应用插件
> go build -buildmode=plugin ../mrapps/wc.go
> 
> # 2. 编译主程序引擎，生成可执行文件 mrsequential
> go build mrsequential.go
> 
> # 3. 清理旧输出
> rm mr-out*
> 
> # 4. 直接运行编译好的二进制文件
> ./mrsequential wc.so pg*.txt
> 
> # 5. 排序查看结果
> sort mr-out-0
> ```

> [!NOTE] 什么是动态链接库和插件？
> - 动态链接库是一组编译好的可执行代码和数据的文件。Linus 上以 `.so` 结尾，在 Windows 上以 `.dll` 结尾
> 	- 不独立运行：没有 `main` 函数，不能像软件一样双击直接启动
> 	- 共享与解耦：主程序可以在启动/运行时去加载它，直接调用里面的函数
> 	- 减少体积：不同软件可以共享同一个 `.so` 文件，无需每个软件都拷贝一份代码。
> - 插件：是一种软件设计模式，动态链接库是实现插件的一种底层技术载体。
> 	- 插件机制允许软件在不重新编译主程序的情况下，动态地扩展、替换功能。

> [!NOTE]
> - mrapps 下的文件会报错：`xx redeclared in this block`,因为 src/mrapps/ 目录下的多个 Go 文件都声明了各自的 Map 和 Reduce 函数，而你可能尝试一次性编译整个目录，因此报错。
> - 这些 Go 文件分别表示一个独立的 MapReduce 应用**插件**，每次只能编译一个文件。一次性编译整个目录会导致重复声明错误

## 代码逻辑总结

### mrsequential.go

它的核心作用是：**在单个进程中，按顺序模拟运行标准 MapReduce 的完整生命周期（动态加载插件 $\rightarrow$ 执行 Map $\rightarrow$ 按 Key 排序归并 $\rightarrow$ 执行 Reduce $\rightarrow$ 输出到文件）**。

实验采用的排序方法是 `sort.Sort`：定义一个自定义类型`ByKey`，并实现`sort.Interface`接口，从而实现`ByKey` 类型变量的排序。这是 Go 最原始的排序方式，目前在新版本中已被 slices 包取代。：具体见[[领域/计算机/Go语言：从排序功能看接口、反射、闭包、泛型\|Go语言：从排序功能看接口、反射、闭包、泛型]]

实验代码中采用 `os.Open` + `ioutil.ReadAll` + `file.Close`进行“打开文件、读取全部内容、自动关闭文件”，这个过程已经被`os.ReadFile(filename)` 替代：[[领域/计算机/Go语言中的标准库-基础输入输出\|Go语言中的标准库-基础输入输出]]

`main` 函数逻辑：

1. 初始化与插件加载：
	1. 命令行参数校验。
	2. 用 `loadPlugin` 动态提取用户自定义的 Map 函数和 Reduce 函数。
2. Map 阶段：
	1. 依次遍历命令行参数中给出的文件，读取文件内容。对于每个文件：
		1. 将内容和文件名传入 `mapf` 函数，获得一批中间键值对（==为什么不是 map？）==
		2. 将键值对合并到内存切片 `intermediate[]`
3. Shuffle 阶段（分组与排序）：
	1. 对内存中收集到的所有中间键值对按照 Key 排序。

> 通过排序，保证所有相同 Key 的数据在内存数组中处于连续紧邻的位置，为接下来的合并分组做准备。
> 真正的分布式 MapReduce 会将中间数据按 Hash 分发到 $M \times N$ 个 Bucket 中；而这里的串行版本直接将所有数据存在单一内存空间里统一排序。

4. Reduce 阶段
	1. 每次循环处理一个 key（分组）
		1. 将所有 key 相同的键值对提取到 `values[]` 中
		2. 将 Key 和 `values[]`传入`reducef` 函数，进行化简。
		3. 将这个 key 的最终化简结果打印到指定文件中。进行下一个循环

> [!NOTE]
> 为什么要用 `loadPlugin`函数来动态加载插件？而不是用`import`来导入`map`和`reduce` 函数？
> 
> - `src/mrapps`下有许多 MapReduce 应用，每个应用的 Map 和 Reduce 函数都不一样（由用户自定义）。如果使用 `import` 来导入，例如`import "6.5840/mrapps/wc"`，硬编码导入 wc 包，框架只能使用 wc 这个应用。如果要修改应用，必须修改框架源代码 + 重新编译。
> - 由于所有函数都被命名为 `Map`和`Reduce`，因此也无法在一个文件里通过`import` 把它们同时导进来。Go 不支持同一个包里出现同名符号
> - 用 `loadPlugin` 方式，**框架代码只需编译一次**，生成一个可执行文件。不同的业务逻辑独立编译成不同的插件，运行时只需通过命令行更换参数即可。

`loadPlugin` 函数（命令行参数为 `wc.so`，并以`map` 的加载为例）

1. `plugin.Open()`: 将 `wc.so` 载入当前进程的内存地址空间
2. `p.Lookup("Map")`：在符号表中查找 "Map" 内存地址，返回一个空接口类型 `plugin.Symbol`,表示一个通用的内存地址指针，Go 还不知道这个符号具体是什么类型的函数或变量。
3. 类型断言：将 `xmapf`强转为特定的函数类型（与`Map` 的函数签名保持一致），返回函数指针，赋值给`mapf`
4. 提取出的 `mapf` 作为普通的 Go 函数变量返回。

> [!NOTE] 为什么 map 的命名从 Map->xmapf->mapf
> - Map：是一个函数类型。可以用它来声明一个函数变量（就如可以用 int 类型来声明一个 num 变量一样）
> - xmapf：第二步中，返回的是一个空接口类型的变量，Go 不确定当前变量的具体类型。`x` 是一个常见的命名惯例，用来表示“未经类型转换前的原始变量/未知变量”
> - mapf：它是通过类型断言强转后得到的函数变量，是一个具有特定签名的标准 Go 函数。另外，`map`是 Go 语言的关键字，不能作为变量名；用 `mapf`表示`Map Function`，即可以明确变量是一个函数类型，也可以区分阶段和方法（Map 可以指代“Map 阶段/Map 任务”，而 `mapf` 则特指“用户写的那个负责具体映射计算的函数”。）
> 	- 也可以用 `mapFunc` 来命名。

> [!NOTE] 为什么是被指定为 `func(string, string) []mr.KeyValue`，而不是指定为`Map`？
> 类型断言的语法规则是：接口变量.(目标类型)。括号里必须填入一个“类型”。而类型/函数/变量的区别如下：
> 
> Map 函数的签名是：`func Map(filename string, contents string) []mr.KeyValue`
> - **类型**：`func(string, string) []mr.KeyValue` 描述的是一个具体的函数签名类型，（即：接受两个 string 参数，返回一个 KeyValue 切片的函数类型）。我们给这个类型取别名为 `A`。
> - **函数**：`Map`是 `A`类型的一个函数，其中`Map` 是一个标识符，标识某个具体的函数。这个具体的函数本质上是只读的代码质量集合，它存储在**内存的代码段**中（静态）
> - **函数变量/变量**：`loadPlugin` 函数中的 `mapf` 即一个函数变量，本质上是一个指针，指向某个具体的函数。存储在内存的堆栈中。

## 工作

写两个程序：coordinator 和 worker

- 并行执行一个 coordinator 进程和一个或多个 worker 进程
- worker 进程通过 RPC 和 coordinator 通信

官方给出了两个初始代码，帮忙起步：coordinator（`main/mrcoordinator.go`）和 worker(`main/mrworker.go`)。这两个文件不能修改

我们需要把实现写在 `mr/coordinator.go`、`mr/worker.go`、`mr/rpc.go`

> [!NOTE]
> mrcoordinator.go 和 coordinator.go 有什么区别？分别用来做什么？
> - 前者是进程入口。包括：解析命令行、启动 Coordinator、轮询是否已完成。相当于 Java 中的 Controller 层
> - 后者是业务逻辑库。相当于 Java 中的 Service 层
> 
> 实际上，如果把 Coordinator 的相关代码全写在 mrcoordinator.go 中，也能跑通。但要将它们解耦为两个文件，因为：
> - 自动化测试。课程的评测脚本 `mr/mr_test.go` 需要能自动启动 Coordinator、创建 Worker、模拟节点崩溃等。
> - 职责分离

> [!NOTE] 什么是 RPC？
> 即 Remote Procedure Call，远程过程调用。是一种计算机通信协议。它的核心思想是：让你能够像调用“本地函数”一样，去调用另一台计算机（或者另一个进程）上的函数，而隐藏了底层的网络通信细节。

举例：word-count 应用

1. 编译 word-count 插件

```bash
cd MIT6.5840/6.5840/src/main
go build -buildmode=plugin ../mrapps/wc.go
```

2. 在其中一个命令行窗口，运行 coordinator
	1. `sock123` 参数指定了一个套接字（socket），协调器（coordinator）通过它来接收来自 Worker 节点的 RPC 请求。
	2. `pg-*.txt` 参数则是输入文件；每个文件代表一个“数据分片”（split），并且会作为单个 Map 任务的输入。

```bash
cd MIT6.5840/6.5840/src/main
rm mr-out*
go run mrcoordinator.go sock123 pg-*.txt
```

3. 在另一个命令行窗口，运行一些 worker

```bash
cd MIT6.5840/6.5840/src/main
go run mrworker.go wc.so sock123
```

4. workers 和 coordinator 完成后，输出结果在 `mr-out-*`。输出结果应该和`src/main/mrsequential.go` 的实现结果是一致的。

官方提供了测试代码（位于 `mr/mr_test.go`），可以在`src` 目录下运行测试：

- 正确性：它会检查 `wc`和`indexer` 两个 MapReduce 应用是否能产生正确输出
- 并发：Map 和 Reduce 任务是否是并行运行
- 容错：Worker 节点发生崩溃时，系统能否成功进行故障恢复。

```bash
cd src
make mr
```

预期输出如下：

```bash
=== RUN   TestWc
--- PASS: TestWc (8.64s)
=== RUN   TestIndexer
--- PASS: TestIndexer (5.90s)
=== RUN   TestMapParallel         # 测试 Map 是否并行
--- PASS: TestMapParallel (7.05s)
=== RUN   TestReduceParallel      # 测试 Reduce 是否并行
--- PASS: TestReduceParallel (8.05s)
=== RUN   TestJobCount            # 测试 Job 计数
--- PASS: TestJobCount (10.04s)
=== RUN   TestEarlyExit
--- PASS: TestEarlyExit (6.05s)
=== RUN   TestCrashWorker         # 测试 Worker 崩溃容错
2026/01/22 14:58:14 *re*-starting map ... # 看到重新启动任务的日志
--- PASS: TestCrashWorker (40.18s)
PASS
ok      6.5840/mr       86.932s   # 总用时 1 分多钟

```

如果出现 `2026/02/11 16:21:32 dialing:dial unix /var/tmp/5840-mr-501: connect: connection refused` 这种报错日志，是正常的。当所有任务做完后，Coordinator 进程会先退出并关闭 RPC 服务。此时可能还有一些“不知情”的 Worker 还在尝试通过 RPC 连 Coordinator，就会报“连接被拒绝”。

### 规则

- Map 阶段需要将 intermediate key 拆分成 `nReduce`个桶。每个 Mapper 都需要创建`nReduce` 个 intermediate 文件供 reduce tasks 使用。
- 应该把 Map 阶段的输出放在当前目录下，方便 Reduce 任务进行读取。
- 第 X 个 Reduce 任务的输出必须写到名为 `mr-out-X` 的文件中（如`mr-out-3`）
- `mr-out-X` 文件中，每一行分别表示一个 Reduce 函数的输出。输出格式为 `"%v %v"`，分别表示 key 和 value。（可以参考`main/mrsequential.go`）
- 可以修改 `mr/coordinator.go`、`mr/worker.go`、`mr/rpc.go`。也可以临时修改`main/`目录下的文件来做本地 Debug，但是官方评测时会重置并使用原始版本的 `main/` 文件。
- `mr/coordinator`需要实现 `Done()`。`mrcoordinator.go` 会不断调用 `coordinator.Done()`。只有当 Map 和 Reduce 所有任务都彻底完成时，`Done()` 返回 `true`，`mrcoordinator` 进程才会安全退出。
- Job 结束后，Coordinator 会退出。Worker 需要感知到这一点并且终止进程。

推荐：

- 把 Map 阶段输出的中间文件命名为 `mr-X-Y`
	- X 表示 Map Task 编号（第几个 Map 任务）
	- Y 表示 Reduce Task 编号（分区 ID）
- 设置 Worker 超时时间为 10s

### 提示

## 我的实现

> 我并没有先看 Hint 部分。

首先要考虑从哪入手。我考虑过两个方向：

1. 从顶向下，查看课程给出的所有函数，分析调用的方向、参数，依次填充函数内部逻辑。这样遇到的问题是，实现问题变成了纯粹的工程问题，且各个函数之间的关系的交叉的，纯粹的填充函数会导致思路混乱
2. 按流程出发。根据 [[领域/计算机/MIT6.5840 引言#案例：MapReduce\|MIT6.5840 引言#案例：MapReduce]] 中分析的流程，来依次写函数逻辑。

初始数据：`src/main/pg-*.txt`

设置参数（这些参数都是已经设定的、无需修改的）：

- M：即传给 Coordinator 的输入文件的数量。
- R：即 `src/mrcoordinator.go`中，调用`mr.MakeCoordinator` 时传入的参数`nReduce: 10`
- 哈希分区函数：（由于是在 Map 阶段进行分区的，所以应该在 `worker.go` 中寻找）`ihash`（用的是 FNV-1a 算法）

> [!NOTE] 为什么实验会选择 FNV-1a 算法？
> 首先要在加密哈希算法和非加密哈希算法之间选择。MapReduce 中，我们需要将海量的 Key 快速打散并分发，因此非加密哈希算法是最优解。
> 	1. 加密哈希算法（MD5、SHA-1、SHA-256）：重点在于安全性与防篡改，计算极其复杂，大量消耗 CPU 资源。
> 	2. 非加密哈希算法（FNV、MurmurHash、xxHash）：重点在于计算速度和离散度。
> 
> 接着，我们要在非加密哈希算法之间选择一个合适的算法：
>
> | 算法                      | 性能                        | 分布均匀性                                         | 典型应用场景                                    |
> | ----------------------- | ------------------------- | --------------------------------------------- | ----------------------------------------- |
> | **MurmurHash**          | 极快                        | 非常均匀 [](https://m.yisu.com/zixun/983139.html)。 | 分布式系统（如 Kafka, Hadoop, Redis）的哈希分片、布隆过滤器。 |
> | **xxHash**              | **极快**，常被认为是速度最快的非加密哈希之一。 | 良好。以牺牲少量均匀性换取极致性能                             | 对速度要求极高的场景，如实时数据处理、数据库内部索引。               |
> | **CityHash / FarmHash** | 非常快，尤其是在处理长字符串时。          | 良好。                                           | Google BigTable、LevelDB 等大数据存储系统。         |
> | **FNV-1a**              | 快，但通常比 MurmurHash 慢。      | 良好。                                           | 对实现简洁性要求高，且性能要求不是极致的场景。如教学实验              |
>
> 实验中选择 FNV-1a，实际上是出于教学目的。它简洁、易于实现。

---

### 阶段一：

> 注意：实验中 Coordinator 等价于论文中的 Master

这个阶段中，我们需要做什么？

首先要看我们有什么。实验通过通过命令 `go run mrcoordinator.go sock123 pg-*.txt` 创建一个 Coordinator 进程。因此，Coordinator 所拥有的数据是：

- sockname（相当于 IP:Port，用来标识 Coordinator 进程的位置）
- 一系列输入文件

> [!NOTE]
> 在操作系统的视角里，`mrcoordinator.go` 和 `coordinator.go` 在编译后，被打包进了同一个二进制可执行文件中。进程中始终只有一个 Coordinator 进程。而`mr.MakeCoordinator` 只是创建了一个 Coordinator 对象（结构体对象）

我们想要完成的是输入分片与任务分配。对于输入分片：

Coordinator 需要向文件系统查询，将 Block 映射成逻辑上的 Input Splits（即 Map Tasks）。我们假设：

- 每个 `pg-*.txt` 文本文件恰好是一个数据块（Block）
- Coordinator 将这些 Block 映射为 Input Split，分片元数据用文件名表示。
- 不考虑数据本地化调度。

> [!NOTE]
> 
> 在工业级 MapReduce 中，并不是靠文件名来区分 map task，因为一个大文件会被切成很多个 task，MapReduce 传输的是某个字节区间的元数据。但在该实验中采用文件名传输，这是为了降低实验过程难度而做的简化。

因此输入分片无需我们完成，我们只需考虑任务分配。

从直觉上看，任务分配有两种方向：

1. 推送（Push）：由 coordinator 主动把任务分配给 worker。
	- coordinator 必须掌握每一个 worker 的状态，对于分布式系统来说，状态同步、网络变化，都是难以维护的。
2. 拉取（Pull）当有 worker 空闲的时候通知 coordinator，再由 coordinator 将任务分配给该 worker。
	- 它天然实现了负载均衡。且 Coordinator 无需掌握每一个 Worker 的状态。

因此，我们采用 Pull 的方式进行任务分配。但问题是：Worker 是如何通知 Coordinator 的？Coordinator 分配任务，任务数据是什么？

> [!NOTE]
> 实际上，这属于进程间通信问题。
> 
> 进程间通信问题，包含单机内进程通信，和跨网络节点通信。
> - 单机内进程通信的方法用于同一台物理机上不同进程之间交换数据。主要通信方式有：管道、信号、消息队列、共享内存、信号量、内存映射文件。
> - 跨网络节点通信的方法，用于进程分布在不同机器时。主要通信方式有：套接字（Socket）、远程过程调用（RPC）、消息队列（MQ）、RESTful API 等
> 
> 不同通信方案的选择要看具体的场景。MapReduce 选择 RPC 的原因主要有：同步响应、编写简单。RPC 让工作节点向主节点请求任务，就像调用一个本地函数一样简单。开发者无需关心数据是如何打包、传输和接收的

我们选用 RPC 方案，让 Worker 和 Coordinator 两个进程之间进行通信。

由于是由 Worker 发起的通信，我们可以将其具体化为：Worker 通过调用某个函数，在这个函数里，Coordinator 给 Worker 分配任务。这个函数即 `coordinator.go`中的`AssignTask` 方法。

`AssignTask` 方法里包含两个结构体：请求结构体和返回结构体。它们满足 RPC 的参数传递规则：

> [!NOTE] Go 语言 RPC 的参数传递规则：
> - 客户端和服务器交互时，请求参数（Args）和返回结果（Reply）都必须打包成结构体。
> 	- 定义一个请求结构体，即 Worker 向 Coordinator 索要任务时传的参数
> 	- 定义一个返回结构体，即 Coordinator 返回给 Worker 的任务信息
> - 字段名必须首字母大写

分配任务，实际上是告诉 Worker 某个信息，让它根据这些信息，去自行完成任务。那么，Worker 需要哪些信息才能完成任务？

- 当前任务是 Map 还是 Reduce？任务 ID 是？
- 当前任务的输入数据是什么？

同时，Coordinator 本身还需要去维护一个任务列表，用于标记任务是否已经完成。

但，这些数据怎么来，从哪来？

如果我们观察 `coordinator.go`，会发现它包含：

- Coordinator 结构体
- Coordinator 结构体的方法
- MakeCoordinator 函数：用于创建 Coordinator 对象，并执行相关操作

可以理解为：结构体用于存储数据与维护状态、方法用于执行动作、函数用于编排流程。

那么我们就可以开始定义 Coordinator 结构体了。它存储无非是所有任务的状态列表。因此我额外定义了两个 Task 结构体。最终代码如下：

```go
type MapTask struct {
	TaskID int
	FileName string
	TaskDone bool
	WorkerID int
	
}

type ReduceTask struct {
	TaskID int
	FileName string
	TaskDone bool
	WorkerID int
}

type Coordinator struct {
	MapTasks []MapTask
	ReduceTasks []ReduceTask
}
```

最初我考虑过用 MapTask 和 ReduceTask 两个不同的结构体。但它们有大量的代码冗余，且在 Coordinator 眼里，Map 任务和 Reduce 任务本质上都是一个“需要被分发、跟踪超时、等待完成的有限状态机”。

自然的，我们要接着考虑把数据存入 Coordinator 中。即创建任务。代码逻辑如下：

1. 初始化任务列表（即切片）
	- 这里我用的初始化方法是 `s := make([]T, 0, 100)`，即声明时指定长度和容量。这是工程上的最佳实践。
2. 给每个输入文件创建一个 map 任务
3. 创建 nReduce 个 reduce 任务

接下来，一旦有 Worker 调用 `AssignTask`，则分配一个未完成的 Map 任务给它。代码逻辑如下：

- 如果还有 Map 任务没有完成，先分配 Map 任务
- 循环找到一个未分配的 Map 任务

> [!question]
> 问题：
> - 如果 Map 任务已经分配出去了，task 对应的 WorkerID 不为 -1，但 worker 出错了怎么办？
> - 如果一个 worker 调用 assignTask 失败，怎么办？

---

Worker 向 Coordinator 请求分配任务

`worker.go` 中请求调用的实例代码 `CallExample`，模仿它写一个真实的请求分配任务的代码即可。

```go
func getTask() (Task) {
	var args AssignTaskArgs
	var reply AssignTaskReply
	ok := call("Coordinator.AssignTask", &args, &reply)
	if !ok {
		return Task{}
	}
	return reply.Task

}
```

在主函数逻辑中，一个 Worker 的生命周期是：

1. 空闲
2. 循环请求分配任务
	1. 分配到任务，开始执行
		1. （有可能出错）
		2. 执行完毕，通知 Coordinator
	2. 未分配到任务，等待一段时间后重新请求
3. 被 Coordinator 通知，关闭进程。

因此主函数里写一个循环，每次循环 `getTask` 一次，根据不同任务类型来处理任务。

### Map 阶段

> [!NOTE]
> `mapTask` 应该要先把 Block 数据块解析为键值对，以偏移量的形式传给 Map 函数。但由于实验中的 Map 函数化简了，所需参数为文件名，因此跳过这一步。
> 且 Map 函数应该要通过 Emit 动作来添加键值对，从而让 Map Worker 能够根据它的 emit 内容来分桶、写入缓冲区。但实验简化了这一步，让 Map 以切片数据结构一次性返回所有中间键值对
> 在原始的 MapReduce 实现中，map 函数是不需要返回中间键值对的，因为它在函数内部已经将中间键值对 emit 上去了
> 并且，实际中单个 Task 并不会输出 R 个分区文件，它们以 (partition, key, value) 的格式写入单个中间文件。这是为了防止文件句柄被耗尽。

mapTask：

1. 读取数据
2. mapTask 执行用户定义的 Map 函数，获得中间键值对 `kva`（key-value array）
3. 对中间键值对排序、合并（Combiner），这里参考 [[领域/计算机/Go语言：从排序功能看接口、反射、闭包、泛型\|Go语言：从排序功能看接口、反射、闭包、泛型]]
4. 将中间键值对存入 mr-X-Y 文件

由于实验中的中间文件和 MR 实际中间文件的存储方式有很大不同，我们只从实验的中间文件来考虑，如果要写入文件，哪种方式最好。我考虑了以下两种

- 循环每个中间键值对，每次循环过程中判断其所属哈希桶，然后追加进对应分区文件。
	- 需要同时打开多个文件，
	- 每次写入都会访问磁盘。并且文件在磁盘上的物理位置不连续，磁盘磁头寻道开销较大
- 先在内存中分桶，分完所有键值对后，一次性将分桶结果写入对应文件。
	- 同一时刻只打开一个文件。

我们采用第二种方式。那么如何将分桶结果（即结构体切片数组）写入对应文件？

> [!NOTE]
> 写数据时，可以分为两个维度考虑：
> 
> - 数据表示与序列化维度（数据格式层）：回答 **“数据存成什么格式？”**
> 	- **核心关注点**：内存里的 Go 对象（结构体、Slice、Map），如何转换为二进制字节流（`[]byte`）。
> 	- **代表技术**：`序列化为 JSON`、`Protobuf`、`XML`、`Gob`、`纯文本（key\tvalue\n）`。
> 	- 序列化：原始数据先通过某种技术，**格式化**为特定格式（如 JSON 格式、XML 格式等），再进一步编码映射为字节流
> 		- **逻辑格式化（Format）**：将复杂内存数据拍平为**特定协议标准**的中间形态（文本或二进制 Schema）。
> 		- **物理字节化（Encode）**：将中间形态按照**编码映射表**转译为物理上的 **`[]byte`**。
> 	- **职责**：只负责**数据的物理表达与结构还原**。它根本不知道也不关心数据最终会被写进文件、发往网络 RPC，还是丢进 Kafka。
> - I/O 调度与刷盘策略维度（传输与存储层）：回答 **“字节流怎么写进磁盘？”**
> 	- 关注点：拿到序列化的字节流后，以什么方式向操作系统发起 `sys_write` 系统调用。
> 	- 概念：
> 		- 一次性写入（`os.WriteFile`），适用于数据量不大的场景。先把几百 MB 数据全拼在 RAM 里，在最后时刻一口气调用 `os.WriteFile` 落盘（有 OOM，即内存溢出 风险）。
> 		- 标准写入（通过 `os` 包打开或创建文件，获得文件对象后再进行操作。），灵活可控，适用于大多数场景。写 1 个字节就触发 1 次 `sys_write`，CPU 频繁在用户态和内核态切换（极其缓慢）。
> 		- 缓冲写入（`bufio.Writer`），适用于大文件或高频写入。先在内存 `bufio` 里开辟一块缓冲区（如 64KB），攒满一块再一次性发起 `sys_write`

> [!NOTE]
> 序列化：本质上是把分散的数据组织成内存上连续的数据。
> 
> 高级数据结构在内存中的存放方式通常散落在各处。例如在 Go 语言中定义一个结构体：
> ```go
> type Student struct {
>     ID   int
>     Name string    // string 本质是一个指针 + 长度，指向堆内存的另一块区域
>     Tags []string  // slice 本质是一个指针 + 长度 + 容量，指向堆内存的又一片区域
> }
> ```
> 当我们实例化这个结构体时，它在内存中的实际物理布局是：
> ```text
> [ 内存地址 A ] Student 结构体主体 (包含 ID，以及指向 Name 和 Tags 的指针)
>      │
>      ├───► 指向 ───► [ 内存地址 B ] Name 的真实字符串数据 ("Alice")
>      │
>      └───► 指向 ───► [ 内存地址 C ] Tags 的切片底层数组 ("Math", "CS")
> 
> ```
> - **物理状态**：这些数据通过**内存地址（指针）**相互关联，但在物理内存芯片上，地址 A、B、C 可能相隔极其遥远。
> 	
> - **致命问题**：**指针（Pointer）只在当前的进程内存空间里有效**。如果你把包含指针的内存直接发给另一台机器，或者写进磁盘，另一个进程读取时，那个指针地址（如 `0x7fff5fbff410`）在它的内存里要么是无效垃圾数据，要么直接导致程序非法访问 Segmentation Fault 崩溃
> 序列化的过程，本质上就是一个**“深度遍历（Deep Traversal）与内存拍平”**的过程：
> 
> 1. **寻线追踪**：序列化程序顺着指针去寻找分散在内存地址 A、B、C 各处的数据。
> 	
> 2. **复制与连续拼接**：剥离掉无意义的“内存地址/指针”，把这些真实的数据按照特定的协议顺序，**挨个紧密地复制到一块新开辟的、绝对连续的内存区域（即 `[]byte` 切片）中**。
> ```text
> [ 序列化后连续的内存空间：[]byte ]
> ┌──────────┬─────────────┬─────────────────┐
> │ ID: 1001 │ Name: Alice │ Tags: Math, CS  │
> └──────────┴─────────────┴─────────────────┘
>  (首尾相连，没有指针，不依赖任何特定内存地址)
> ```
>
> 只有变成连续的 `[]byte`，才能拥有以下两个特性：
> 1. IO 刷盘与传输能力。
> 2. “可反序列化”的能力。因为数据在连续空间里是按既定顺序排布的，接收方才能顺着这块连续内存的开头逐字节往后读，重新在自己的内存里重新分配地址，**把数据还原（Unflatten）回原来的树状或图状结构**。

我们有大量数据，所以应该选择缓冲写入的方式。不过，结构体切片应该被序列化为什么呢？

> [!NOTE]
> 1. 序列化为文本字符流（JSON、XML）。这种方式开发效率最高，可读性最强，兼容性极高。无需额外逻辑来处理特殊字符，最安全且通用。
> 2. 序列化为二进制流。它的序列化速度最快、体积最小，但不支持文件追加，人类不可读。
> 3. 序列化为固定/自定义分隔符文本（如 CSV）。字段之间通过特定控制字符隔开，结尾加上 `\n` 单行字符流。如果数据本身不包含空格换行，这种格式非常轻量，但遇到复杂字符串时极易发生字段解析错乱。
> 
> 我们选择第一种。序列化为 Json 格式。

在 Go 语言中，将 JSON 序列化与缓冲读写结合起来的工程实现方式，是使用 json.NewEncoder 和 json.NewDecoder 方法（即 Go 标准包 `encoding/json`）

> [!NOTE]
> 
> 这个库提供了两套 API，分别是全量读取文件和流式读取文件。不过，读取后的结果如何处理？是否要存储到内存中？这也是值得考虑的问题
>
> | 操作维度                  | **内存级（全量）**                 | **流式级（IO） + 不存储（推荐）**                     | **流式级（IO） + 存储切片（你指出的陷阱）**                      |
> | --------------------- | --------------------------- | ----------------------------------------- | ----------------------------------------------- |
> | **API 调用**            | `Unmarshal(data, &v)`       | `Decoder.Decode(&item)` + `process(item)` | `Decoder.Decode(&item)` + `append(slice, item)` |
> | **解析器（Decoder）本身的内存** | 解析时需持有完整 `[]byte`，内存 = 文件大小 | 仅持有 4KB~32KB 缓冲区                          | 仅持有 4KB~32KB 缓冲区                                |
> | **你的业务数据（结果集）内存**     | 全部载入 `v` 中                  | **0**（处理完就丢，不保存）                          | **全部载入切片中**，线性增长                                |
> | **最终总体内存特征**          | **O(N) 且双倍风险**（原始字节 + 结构体）  | **O(1) 恒定**（真正的内存安全）                      | **O(N) 线性增长**（依然会 OOM）                          |

Map 任务完成后，需要发起 RPC 通知 Coordinator 任务已完成。

- Worker 新增一个 `doneTask` 函数，用于发起 RPC
- Coordinator 新增一个 `DoneTask` 函数，找到对应已完成的任务，标记其已完成，并修改对应任务的计数器。

> [!NOTE]
> Combiner
> Combiner 阶段和执行 Reduce 前合并 key 的工作是相同的，都是要把 key 相同的键值对合并成迭代器。但我采用了不同的实现方式。
> - reduceTask 中，我选择了用逐行扫描方案，遇到新的 Key 时，触发上一组的 Reduce。这种方案实现起来复杂，但内存占用低，因为是流式逐行读取的，无需将数据全量加载进内存。
> - 在 Combiner 中，我选择了更简单的 map 方案，通过 map 结构自动将 Key 分组。这种方案健壮性极高，不会出现边界问题，代码复杂度极简，但内存占用高，需要将当前任务的所有键值对全都加载进内存。
> 但！该实验中不能用 Combiner。因为 wc.go 中的 Reduce 的合并逻辑是计算 values 的长度，而不是求和。

### 阶段三

当所有 Map 任务完成后，才会开始这一阶段，

worker 从 Coordinator 获取任务，如果任务类型是 Reduce，`reduceTask` 函数

1. 根据 task.TaskID（即哈希桶的序号），合并所有桶中的数据
2. 排序，结果写进 pre 文件
3. 分组
4. 执行 reduce 函数，获得最终键值对
5. 将键值对存储到最终文件中

Map 阶段生成的每个文件都是有序的，因此对于第一步、第二步，实则完成的是同一个目标：将 N 个有序的小文件，合并成 1 个全局有序的大文件。这是多路归并问题。

> [!NOTE]
> 多路归并
> 
> 1. **打开所有文件**：同时打开 `chunk_1` 到 `chunk_N` 这 N 个文件，为每个文件创建一个 `json.Decoder`（流式读取）。
> 2. **初始化堆（Heap）**：从每个文件中读取**第一条（即最小的一条）**记录，放入一个最小堆（Min-Heap）中。此时堆里只有 N 条数据（N 等于分块数量，可能只有几十或几百，内存占用极小）。
> 3. **循环弹出**：
> 	
> 	- 从堆中弹出最小的那条记录，写入最终的输出文件。
> 	- 记录这条记录来自哪个文件（比如来自 `chunk_5`）。
> 	- 从 `chunk_5` 中**再读取下一条记录**，放入堆中。
> 		
> 4. **重复步骤 3**，直到所有文件都读取完毕。
>
> > **此时结果**：最终输出文件就是全局有序的，而整个过程中内存里最多只存了（分块数 + 当前单条记录）的数据量。

不过，如果单纯采用流式读取 + 不存储的方案，每次最小记录写入文件都需要写一次硬盘，开销极大。因此在写入时，我们采用缓冲写入的方案。

---

对于第三步和第四步，可以合并成一步：从 pre 文件中流式读取键值对，每分组一次，就把分组结果送入 reduce，把 reduce 的返回内容（最终键值对）写进最终文件。

- 流式按行读取：`bufio.Scanner`
将 reduce 生成的结果追加进最终文件。

任务完成后，通知 Coordinator

---

优化

代码当前的流程是：读所有 Map 文件 -> 归并排序 -> 写入本地 `mr-pre-*` 文件 -> 重新打开该文件 -> 按行读取 -> 分组交由 Reduce 处理 -> 写入最终文件。

这里多了一步磁盘读写。

实际上，归并排序时，每次弹出的值都是有序的，可以直接进行分组，并交给 Reduce 处理。

流程优化为：

1. 【初始化】创建一个小根堆（优先队列），进行堆初始化（该桶的所有文件的第一个键值对入堆）
2. 【初始化】创建最终文件 mr-out-\*
3. 【初始化】维护一个 groups，用于存储分组
4. 多路归并排序，不断从堆顶获取最小值（出队）。
	1. 如果最小值所属文件已经被读完，则关闭文件，并直接进入下一次读取
	2. 如果最小值所属文件仍有数据未读，则将第一条数据入队。
5. 如果获取到的最小值 key 和上一次获取的 key 不同，则将分组结果 groups 送入 reduce，并更新 groups
6. 将 reduce 结果追加写入 mr-out-\*

### 阶段五

当所有任务结束后，`MapCount`和 `ReduceCount`都为 0。`mrcoordinator`周期性调用`Done()` 了来判断 Coordinator 任务是否已经完成。

## 如何 debug？

> [!NOTE]
> `log`/`slog` 还是 `fmt.Println`?
> - `fmt.Println`：是面向用户的交互与标准输出。常用于**向终端用户**打印提示信息、帮助文档、操作结果等。
> - `log`/`slog`：是面向系统的运维与诊断。
> 因此，如果是临时排查，写完就删，则用 fmt.Println。如果是长期保留，需要在不同环境切换控制日志输出，则用 log
> 
> `log` 还是 `slog`？
> - 传统 `log` 是纯文本输出。如果要收集日志中的数据用于统计，需要写复杂的正则表达式。它不带日志级别，只能通过特殊函数来控制程序（可以决定打印日志后是否要退出程序）。
> - `slog` 是键值对格式。它自带日志级别，且所有数据都是明确的键值对。
> 
> 日志级别是控制日志输出的重要手段。通过设置不同的日志级别，我们可以灵活地控制日志的详细程度。在 Go 语言中，常见的日志级别有 `DEBUG`、`INFO`、`WARN`、`ERROR` 和`FATAL`。不同级别的日志用于记录不同类型的信息，例如：
> 
> - `DEBUG`：用于记录详细的调试信息，仅在**开发环境中**启用。例如变量、函数入参等
> - `INFO`：用于记录正常的业务流程信息，例如服务启动、配置加载、任务完成、请求的处理、数据的加载等。
> - `WARN`：用于记录可能存在的问题或异常情况，但不影响系统的正常运行。例如用户输入了非法参数、限流触发、某个请求超时但已重试。
> - `ERROR`：用于记录严重的错误信息，这些错误可能导致系统无法正常运行。例如数据库连接端口
> - `FATAL`：用于记录非常严重的错误信息，这些错误会导致程序立即退出。

![Pasted image 20260821174337.png](/img/user/%E9%99%84%E4%BB%B6/%E5%9B%BE%E7%89%87/Pasted%20image%2020260821174337.png)

coordinator 和 worker 运行后，控制台没有任何输出，我不知道这些进程正在做什么，卡在哪一步。因此，我需要添加日志，来确认每个进程都在正常运行。

> 由于课程的要求：
> 1. 输出结果要干净
> 2. 不能修改 main 函数所在的文件
> 因此，我把 logger 的初始化放在了 coordinator.go 和 worker.go 的入口。并且通过设置 slog.HandlerOptions 来一键显隐不同的日志等级
> 
> 由于 worker 和 coordinator 的 logger 初始化是一样的，因此我考虑过在 mr 包下新建了一个 util.go，定义一个统一的初始化函数，在两个入口处调用一次。但实验要求我们只能修改 mr/worker.go、mr/coordinator.go 和 mr/rpc.go 三个文件，因此这个方案 pass。

由于我需要进行多进程的监控，因此我初始化了一个带有 PID 的 logger。

```go
//初始化logger
func initLogger() {
	handler :=slog.NewTextHandler(os.Stdout, &slog.HandlerOptions{
		Level: slog.LevelDebug})
	logger := slog.New(handler).With("Coordinator.PID", os.Getpid())
	//设置为全局默认Logger
	slog.SetDefault(logger)
	slog.Info("Coordinator 已成功启动")	
}
```

---

运行 Coordinator 和 Worker，输出日志如下：

Worker 进程：

![Pasted image 20260820204359.png](/img/user/%E9%99%84%E4%BB%B6/%E5%9B%BE%E7%89%87/Pasted%20image%2020260820204359.png)

Coordinator 进程

![Pasted image 20260820204439.png](/img/user/%E9%99%84%E4%BB%B6/%E5%9B%BE%E7%89%87/Pasted%20image%2020260820204439.png)

---

因此，代码能够实现基本的流程。但我并未做错误处理（如 worker 宕机）等

`makr mr` 后，出现 DATA RACE 报错

报错指出：`main` 协程在调用 `Coordinator.Done()` 读取数据时，RPC 协程（Goroutine 28）正在调用 `Coordinator.DoneTask()` 修改数据，且**两边没有加锁互斥**，访问了同一个内存地址（`0x00c000210198`）。

因此，我们虽然完成了基本的业务逻辑代码，但没有做好临界区的处理。

那么有哪些需要处理的临界区变量？

Coordinator 和不同 Worker 所处的进程空间不同，它们不会访问同一个内存地址。

单个 Worker 内部所有的任务处理都是线性的，也无需处理。

Coordinator 内部，同一时间会启动多个协程

- main 在循环执行 Done() 时，读 MapCount 和 ReduceCount
- http.Serve 由 main 函数显式调用，会常驻内存，
- 每个 RPC 请求都会调用一个 gorotine 来跑对应的函数。如果有 N 个 Worker 同时发起任务分配请求，M 个 Worker 通知 Coordinator 任务已完成，则会有 2+M+N 个协程运行。

> [!NOTE]
> Go 并发原语选型指南
>
> |对比维度|**互斥锁 (`sync.Mutex / RWMutex`)**|**通道 (`Channel`)**|**原子操作 (`sync/atomic`)**|**单例执行 (`sync.Once`)**|
> |---|---|---|---|---|
> |**核心设计理念**|**共享内存**：保护临界区数据结构|**传递数据**：控制协程间的执行流|**硬件级无锁**：极轻量的数据修改|**保证唯一性**：仅且执行一次|
> |**最适合场景**|多字段关联状态机、复杂结构体（如 Map/Slice/对象缓存）|协程间任务分发、数据所有权转移、事件异步通知/超时控制|无关联的**单一数值**（如请求计数器、状态标志位、指针替换）|延迟加载、全局配置加载、数据库连接池初始化|
> |**典型代码模式**|`mu.Lock()` ... `defer mu.Unlock()`|`ch <- task` / `<-ch` / `select`|`atomic.AddInt64()` / `atomic.LoadPointer()`|`once.Do(func() { ... })`|
> |**性能开销**|低（无竞争时极快；高竞争时引发上下文切换）|中（涉及内存拷贝、队列锁与调度器介入）|**极低**（直接映射 CPU 汇编指令，零阻塞）|极低（仅首次存在原子锁校验，后续直接返回）|
> |**潜在风险**|容易死锁（忘记 Unlock、锁嵌套）；粒度过大影响并发|容易阻塞死锁、Channel 泄漏、向已关闭的 Channel 写数据导致崩溃|无法保证“组合逻辑”的原子性（如：先读再修改多字段）|初始化函数内部若崩溃/panic，不会重新执行|
>
> ```bash
> 需要并发保护的代码
>        │
>        ├─► 涉及到 2 个以上的变量？ (例如: a 和 b 必须同时修改)
>        │    └─► 必须用 【互斥锁 Mutex】
>        │
>        ├─► 包含 “先判断，后修改” 的组合逻辑？ (例如: if x == 0 { x = 1 })
>        │    └─► 必须用 【互斥锁 Mutex】
>        │
>        └─► 仅仅是对“单个孤立的数字/指针”做纯粹的自增、赋值或读取？
>             └─► 可以用 【原子操作 Atomic】
> ```

为了防止多个协程同时读写计数器、修改任务状态。我们需要给 Coordinator 加上互斥锁。对数据进行读写时，都需要加锁

```go
type Coordinator struct {
	mu sync.Mutex   //互斥锁，保护所有字段
	Tasks []Task
	MapCount int
	ReduceCount int
}
```

例如在分配任务时：

```go
func (c *Coordinator) AssignTask(args *AssignTaskArgs, reply *AssignTaskReply) error {
	c.mu.Lock()         //加锁
	defer c.mu.Unlock() //解锁
	...	
}
```

---

- 我在实现 Map 阶段的 Combiner 时，错误地将其理解为是简单的 key 合并，但实际上 combiner 需要调用 reducef。
- 匹配所有 "`mr-*-TaskID`" 的文件时，错误匹配到了无关文件（如 `mr-pre-TaskID`）。正则式应该改为`mr-[0-9]*-%d`
- 任务失败后，使用了 `log.Fatalf`来记录日志。`log.Fatalf`会指定`os.Exit(1)`来杀死进程。应该用`slog.Error`来打印错误日志，并用`continue` 进入下一次循环来领取任务。
- 我使用 `bufio.Write`写文件，但是由于关闭文件前没有调用`Flush()` 刷盘，因此缓冲区中残留的数据没有写入硬盘。导致我每个文件 s~z 的数据丢失。

` TestWc` 测试失败

观察

![Pasted image 20260821111505.png](/img/user/%E9%99%84%E4%BB%B6/%E5%9B%BE%E7%89%87/Pasted%20image%2020260821111505.png)

- 【优化】多个 Worker 会同时分配一个任务，这种分配方式是低效的，因此我们可以对 Coordinator 的任务分配方式进行优化。
	- 优化 Task 的状态。每次分配任务时，会先分配“未分配”状态的任务，如果所有任务已分配，则分配“已分配时间最长的任务”。
	- 由于 Map 任务的分配和 Reduce 任务的分配只有 TaskType 字段不同，因此写了一个 `selectTask` 去选择一个具体任务进行分配。
- 先前我对于 Worker 错误处理和并发性的设计是这样的：Worker 发起请求后，Coordinator 分配任意一个未完成（即使已分配）的任务给它。但优化了 Task 的状态后，这个逻辑也需要跟着修改。
	- 【优化】没有空闲任务的时候，某个空闲 Worker 请求任务时，将无法得到任务，降低了程序的并发性。因此要使得 Worker 能够得到已分配的运行中任务，
	- 【错误】Cordinator 需要检测到 Worker 超时的情况。实际上，应该是检测 task 超时的情况，所以添加一个 `AssignedAt` 字段，记录上一次分配该任务的时间。
- 【错误】如果使用 range 循环，要修改数组中的数据，则必须用下标来访问数据，而不是直接修改。比如：

```go
for _, task := range c.Tasks{
	if task.TaskType == t && task.Status == Idle{
		task.AssignedAt = time.Now()  //这里的task仅仅是一个副本，并不会修改真正的task的值
		task.Status = Running
		return task
	}
}
```

- 【错误】多个 Map Worker 过程中会将数据写入相同的文件，
	- 使用原子写入（先写入临时文件，再用 `os.Rename` 原子性重命名）

```go
func atomicWriteFile(filename string, data []byte) error {
    // 1. 在同一目录下创建临时文件
    tmpFile, err := os.CreateTemp(".", "tmp-*")
    if err != nil {
        return err
    }
    tmpName := tmpFile.Name()

    // 2. 写入数据
    if _, err := tmpFile.Write(data); err != nil {
        tmpFile.Close()
        os.Remove(tmpName)
        return err
    }

    // 3. 刷盘，确保数据真正写入磁盘
    if err := tmpFile.Sync(); err != nil {
        tmpFile.Close()
        os.Remove(tmpName)
        return err
    }
    tmpFile.Close()

    // 4. 原子重命名（同一文件系统内，rename 是原子操作）
    return os.Rename(tmpName, filename)
}
```

> [!NOTE]
> 待分配队列用 `chan Task` 还是`[]Task`？
> - 如果同个任务只会被分配到一个机器上，那么用 `chan Task`
> - 如果同个任务可能会被分配到多个机器上，那么用 `[]Task`

` TestJobCount` 测试失败

- 【设计】我的设计中，只要有 worker 请求任务，且存在未完成的任务，那么 worker 就能领取到任务，即使这个任务没有超时。这样能够天然实现负载均衡（1.分配一个未完成的、类型为 t 的任务；2.如果全都分配完，但是仍有任务未完成，则分配一个已分配时间最长的任务给 Worker）。但是这无法通过 ` TestJobCount` 测试。实验的要求是：如果任务已分配，未超时，那么它就不能被分配给其她 Worker
	- 从功能性角度上来说，我的逻辑是正确的。但这是一种激进的重试。标准的推测执行是只在拖后腿的任务上启动备份，用少量冗余计算换取总执行时间的缩短，整体资源开销小；我的执行会**耗费大量的计算资源和网络带宽**，造成 CPU 和内存的浪费。
	- 因此，我需要开一个协程，来定期检测各个任务是否超时未完成

> [!NOTE]
> 在 MapReduce 中，两个 worker 同时执行同一个 map 任务主要发生在以下两种场景。
> - 推测执行（优化速度。并发执行）。由于机器负载不均、硬件老化或网络延迟等，一个作业中的某些任务可能远慢于其他任务，成为整个作业的瓶颈。为了避免这种情况拖慢整个作业，框架会为该任务启动一个备份任务
> - 任务失败重试（确保可靠性。顺序执行）。这种重试是顺序的，即先等失败的任务结束后，再重新开始一个新任务。
>
> |特性|**任务失败重试 (Fault-tolerance Retry)**|**推测执行 (Speculative Execution)**|
> |---|---|---|
> |**主要目的**|**保证可靠性**：确保即使 Worker 故障，任务也能最终完成。|**优化性能**：为“拖后腿”的慢任务启动备份，加速整体进度。|
> |**触发条件**|**任务失败**：Worker 崩溃、无响应或超过 10 秒未完成。|**任务缓慢**：任务仍在运行，但进度远落后于同类任务。|
> |**执行时序**|**先后执行**：旧任务失败后，新任务才启动。|**并行执行**：备份任务与原始任务同时运行，先完成的被采纳。|
> |**在 6.5840 中**|**必须实现**（核心要求）。|**可选实现**（实验不要求）。|
>
> 在实验中要求超过 10 秒没有完成的任务即为超时。这是属于任务失败重试的情况。超过 10 秒没有完成这个任务，就可以视为这个 worker 已经崩溃了。这种情况下应该被归为容错，而不是性能优化。Coordinator 会把该任务的状态设置为空闲。
> 
> 如果是推测执行及并行的情况，coordinator 会知道原来的 worker 是活着的并且正常，但它认为对方太慢了，所以会把这个任务分配给另外一个 worker，并且采纳最先返回的结果，然后主动通过 RPC 通知慢的那个 worker 放弃任务。

----

## 时间优化

官方实验的预期输出：

```go
=== RUN   TestWc
--- PASS: TestWc (8.64s)
=== RUN   TestIndexer
--- PASS: TestIndexer (5.90s)
=== RUN   TestMapParallel         # 测试 Map 是否并行
--- PASS: TestMapParallel (7.05s)
=== RUN   TestReduceParallel      # 测试 Reduce 是否并行
--- PASS: TestReduceParallel (8.05s)
=== RUN   TestJobCount            # 测试 Job 计数
--- PASS: TestJobCount (10.04s)
=== RUN   TestEarlyExit
--- PASS: TestEarlyExit (6.05s)
=== RUN   TestCrashWorker         # 测试 Worker 崩溃容错
2026/01/22 14:58:14 *re*-starting map ... # 看到重新启动任务的日志
--- PASS: TestCrashWorker (40.18s)
PASS
ok      6.5840/mr       86.932s   # 总用时 1 分多钟
```

第一轮测试：

```go
=== RUN   TestWc
--- PASS: TestWc (9.33s)
=== RUN   TestIndexer
--- PASS: TestIndexer (5.60s)
=== RUN   TestMapParallel
--- PASS: TestMapParallel (8.07s)
=== RUN   TestReduceParallel
--- PASS: TestReduceParallel (9.07s)
=== RUN   TestJobCount
--- PASS: TestJobCount (13.08s)
=== RUN   TestEarlyExit
--- PASS: TestEarlyExit (7.08s)
=== RUN   TestCrashWorker
--- PASS: TestCrashWorker (54.31s)
PASS
ok      6.5840/mr       107.559s
```

第二轮测试：

- 优化了 shuffle 阶段的 pipeline，减少一轮磁盘读写

```go
=== RUN   TestWc
--- PASS: TestWc (8.32s)
=== RUN   TestIndexer
--- PASS: TestIndexer (5.60s)
=== RUN   TestMapParallel
--- PASS: TestMapParallel (8.07s)
=== RUN   TestReduceParallel
--- PASS: TestReduceParallel (9.08s)
=== RUN   TestJobCount
--- PASS: TestJobCount (11.08s)
=== RUN   TestEarlyExit
--- PASS: TestEarlyExit (7.08s)
=== RUN   TestCrashWorker
--- PASS: TestCrashWorker (32.19s)
PASS
ok      6.5840/mr       82.433s
```

可以发现一个问题：TestMapParallel、TestReduceParallel、TestJobCount、TestEarlyExit 这四个测试是测量变形度、任务数量精确性、退出时机的。共同点在于：任务的分配时序和并行问题。

我需要寻找和 `time.Sleep` 相关的代码。

于是在 `mrcoordinator.go` 中发现：

```go
func main() {
	if len(os.Args) < 3 {
		fmt.Fprintf(os.Stderr, "Usage: mrcoordinator sockname inputfiles...\n")
		os.Exit(1)
	}

	m := mr.MakeCoordinator(os.Args[1], os.Args[2:], 10)
	for m.Done() == false {
		time.Sleep(time.Second)
	}

	time.Sleep(time.Second)  //这个是官方自带的，不能改
}

```

```go
key, value := strings.SplitN(line, " ", 2)[0], strings.SplitN(line, " ", 2)[1]
```

`strings.SplitN` 内部会分配底层数组并遍历字符串。对同一行数据调用了两次，白白浪费了一倍的 CPU 计算时间和垃圾回收（GC）压力。优化后：

```go
parts := strings.SplitN(line, " ", 2)
if len(parts) == 2 {
    key, value = parts[0], parts[1]
}
```

在持有 `c.mu` 的情况下 sleep，那么所有 Worker 的 RPC 都会被阻塞，包括 AssignTask。

```go
func (c *Coordinator) Done() bool {
	ret := false
	c.mu.Lock()
	defer c.mu.Unlock()

	if c.MapCount == 0 && c.ReduceCount == 0 {
		ret = true
		slog.Info("所有任务已完成,Coordinator等待Worker退出")
		//等待所有Worker退出
		time.Sleep(50 * time.Millisecond)

		slog.Info("Coordinator 已成功退出")
	}
	return ret
}
```

修改后（锁内只判断状态，锁外 sleep）：

---

常规的文件写入不是原子的，如果在写入过程中发生程序崩溃、断电、或者有其她并发进程正在读取该文件，就会导致数据损坏或读到脏数据。因此，在以下几种情况，必须使用原子写入：

- 更新配置文件或元数据
- 读写并发
- 保存数据快照

Coordinator 可能因任务超时将同一个任务重新分配给另一个 worker。此时旧 worker 可能仍在写文件，新 worker 也写同一组文件。如果直接写入，两个进程的数据可能交错，导致文件损坏。
