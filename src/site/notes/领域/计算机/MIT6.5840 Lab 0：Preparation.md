---
{"dg-publish":true,"dg-path":"Lecture/MIT6.5840/mit6.5840-lab0","dg-permalink":"mit6.5840-lab0","permalink":"/mit6.5840-lab0/","title":"MIT6.5840 Lab 0：Preparation","created":"2026-09-02T10:15:19.034+08:00","updated":"2026-09-02T10:22:22.622+08:00","dg-note-properties":{"title":"MIT6.5840 Lab 0：Preparation","aliases":[null],"created":"2026-09-02","updated":"2026-09-02 10:22","area":"计算机","type":"study","description":null,"status":"active"}}
---


# MIT6.5840 Lab 0：Preparation

> 相关：
> - [6.5840 Lab 1: MapReduce](https://pdos.csail.mit.edu/6.824/labs/lab-mr.html)

## 开发环境配置

> 可以选择任意支持 Go 语言的编辑器或 IDE，比如 VS Code。我选择的是 Trae。

实验环境：

- WSL2： [[技能/WSL2+OpenClaw安装记录\|WSL2安装记录]]
- Go：[[技能/Go语言安装-WSL2环境（宝宝巴士版）\|Go语言安装-WSL2环境（宝宝巴士版）]]。

创建文件夹：

```bash
mkdir MIT6.5840
cd MTI6.5840
```

克隆库：

```bash
git clone git://g.csail.mit.edu/6.5840-golabs-2026 6.5840
code 6.5840    #用VSCode打开项目，出现Installing VS Code Server for Linux x64 是正常的
```

> [!NOTE]
> 通过 git 拉取仓库时，不能直接拉取到 Windows 的盘符下，必须拉取到 WSL2 的文件系统中（如 `\\wsl.localhost\Ubuntu-24.04\home\username`）。
> 
> 如果是在 Windows 中拉取（则通过 `/mnt/c` 访问文件），WSL2 访问 Windows 的 NTFS 文件系统，本质上是通过 9p 协议（一种网络文件系统协议）进行转义的，I/O （如文件读写）性能极差。

在 Trae 中操作：

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

> 建议自己新建一个远程仓库，并设置为私有。用于维护自己的实验代码。具体步骤见 [[技能/Git 的配置与仓库初始化（WSL2）\|Git 的配置与仓库初始化（WSL2）]]（注意需要先[[技能/Git 的配置与仓库初始化（WSL2）#其她操作\|删除原有的 Git 历史记录]]）

## 实验指导（所有实验通用）

### 测试与提交

有两个 Makefile：

1.`6.5840/Makefile`：做完实验要提交作业到 Gradescope 时，它会帮你在本地检查编译，并把源码打包成合格的压缩文件。

使用：

- 打包命令：`make [lab1|lab2|lab3a|lab3b|lab3c|lab3d|lab4a|lab4b|lab4c|lab5a|lab5b|lab5c]`。如命令 `make lab1` 能够生成 lab1-handin.tar.gz。将该压缩包上传到 Gradescope 即可。

```bash
make lab1    # 如果你做的是 Lab 1 (MapReduce)
make lab2    # 如果你做的是 Lab 2 (Raft)
make lab3a   # 如果你做的是 Lab 3a
```

执行成功后，目录下会生成一个类似 **`lab1-handin.tar.gz`** 的压缩包，把它下载下来上传到 Gradescope 即可。

> **Gradescope的提交通道不对外开放**，因此我们使用课程本身提供的本地测试命令。

2.`6.5840/src/Makefile`：专门用于本地开发与测试，负责在本地编译代码、运行测试用例

使用：
- 进入`6.5840/src`文件夹。
- 跑某个实验的全部测试：`make [mr|kvsrv1|lock1|raft1|rsm1|kvraft1|shardkv]` （如 `make mr` 用于编译和测试名为 mr 的实验）
- 跑某个特定的测试：使用 `RUN="-run Wc"` 可以单独运行名为 Wc 的测试。例如运行 `$ make RUN="-run Wc" mr` 会只执行 `mr` 实验中名称包含 Wc 的测试用例。

### 实验难度

- Easy：几个小时
- Moderate：大约六个小时
- Hard：超过六个小时

每个实验的代码量大约是几百行。

### 提示

- Go：
	- 需要完成：[A Tour of Go](https://go.dev/tour/welcome/1)
	- 遇到问题时查询：[Effective Go - The Go Programming Language](https://go.dev/doc/effective_go)
	- Go 的格式化打印字符串 `Printf`： [Go format strings](https://golang.org/pkg/fmt/).
- Git 版本控制：[Pro Git book](https://git-scm.com/book/en/v2) 或者 [git user's manual](http://www.kernel.org/pub/software/scm/git/docs/user-manual.html).
- Debugging 的时候记得做笔记，这样可以防止自己忘记之前踩过的坑。
- 多加 print 语句来定位错误
- 在写代码的时候，给代码添加显式条件检查（可以使用 `panic`）

## 了解目录结构

代码包含了该课程的所有实验。

Lab1：MapReduce

| 路径                        | 角色       | 说明                                                     |
| ------------------------- | -------- | ------------------------------------------------------ |
| src/mr/                   | **核心实现** | Coordinator、Worker、RPC 接口定义（你需要写代码的地方）                 |
| src/mrapps/               | 测试插件     | WordCount、Indexer、Crash、Timing 等 Map/Reduce 插件（作为测试用例） |
| src/main/mrsequential.go  | 入口       | 单机版 MapReduce（参考基线）                                    |
| src/main/mrcoordinator.go | 入口       | 分布式 Coordinator 启动入口                                   |
| src/main/mrworker.go      | 入口       | 分布式 Worker 启动入口                                        |
| src/main/pg-*.txt (8 个)   | 测试数据     | 8 本英文小说（作为 WordCount 等任务的输入数据）                         |

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

| 文件                 | 引用的缺失包             | 推测历史归属                             |
| ------------------ | ------------------ | ---------------------------------- |
| src/main/viewd.go  | 6.5840/viewservice | 旧版 Lab 2 的 ViewServer（视图服务，成员管理）   |
| src/main/pbd.go    | 6.5840/pbservice   | 旧版 Lab 的 Primary-Backup 复制服务       |
| src/main/pbc.go    | 6.5840/pbservice   | 旧版 pbservice 的客户端                  |
| src/main/diskvd.go | 6.5840/diskv       | 旧版 Lab 的磁盘持久化服务器（可能和旧版 shardkv 有关） |

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

