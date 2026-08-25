---
{"dg-publish":true,"dg-path":"Others/go1","dg-permalink":"go1","permalink":"/go1/","title":"Go语言安装-WSL2环境","created":"2026-08-06T04:23:44.643+08:00","updated":"2026-08-25T22:33:48.138+08:00","dg-note-properties":{"title":"Go语言安装-WSL2环境","aliases":[null],"created":"2026-08-06","updated":"2026-08-25 22:33","type":"skill","description":null,"status":"active"}}
---


# Go语言安装-WSL2环境（宝宝巴士版）

> 相关：

**打开 WSL2 终端并更新软件包**：

- 这样能够确保系统环境是最新的，后续安装依赖时不会因为本地包索引过旧而出现问题

```bash
sudo apt update && sudo apt upgrade -y
```

**安装必要的依赖工具**：

- 如果已经有了 git 或者 wget，运行这个命令也是安全的。apt 如果发现系统已经安装了 git，则会直接跳过安装步骤或进行升级。
- wget：命令行下载工具。用于从网络下载 Go 的压缩包
- git：必备的版本管理工具
- build-essential：包含了 Go 在编译某些依赖时可能用到的系统级编译工具（ gcc、g++、make 等 C/C++编译器和构建工具，Go 语言有时会依赖 C 代码）
- `-y` 参数会自动回答“是（Yes）”，避免安装过程中弹出确认提示。

```bash
sudo apt install -y wget git build-essential
```

**下载 Go 二进制包**。查看[官方下载页面](https://go.dev/dl/)，找到最新稳定版的 Linux 包链接（选择适配自己的操作系统和架构，例如我的是 Linux、x86-64 ），在终端中使用 wget 下载。

> 注意不要下载成 `go1.26.5.linux-386.tar.gz`，这是 x86 32 位的

```bash
# 请将链接替换为你需要的版本
wget https://golang.google.cn/dl/go1.26.5.linux-amd64.tar.gz
```

将下载的压缩包**解压**到 `/usr/local` 目录

- `/usr/local` 是 Unix-like 系统上安装用户自行编译的软件的标准路径，符合 Linux 文件系统层级标准（FHS，它规定了哪个目录该放什么类型的文件）
- 需要 `sudo`，因为 `/usr/local` 目录需要管理员权限才能写入。
- 为什么要用 `tar` 解压而不是通过包管理器？这能确保 Go 工具链被完整地、原样地放置在 `/usr/local/go` 目录下，不会像 `apt` 那样将文件分散到系统各处，便于后续的版本管理和卸载

```bash
# 将压缩包替换为你的实际压缩包
sudo tar -C /usr/local -xzf go1.26.5.linux-amd64.tar.gz
```

> [!NOTE] Linux 发行版内部软件：
>
> |路径|用途|示例|
> |---|---|---|
> | `/usr/bin` |**系统命令**：大多数系统自带和通过包管理器安装的可执行程序| `ls`, `gcc`, `python3` |
> | `/usr/local/bin` |**用户程序**：用户自行编译或安装的程序|手动编译安装的软件|
> | `/opt` |**可选软件**：通常用于存放大型、独立的第三方软件包| `google-chrome` |
> | `/home/<用户名>` |**用户主目录**：你的个人文件、下载内容、项目代码和很多软件的配置文件|下载的源码、`pip` 安装的部分包|

（可选）**清理**下载的压缩包

```bash
# 将压缩包替换为你的实际压缩包
rm go1.26.5.linux-amd64.tar.gz
```

**配置环境变量**：

>  `~/.bashrc` 是 Bash 终端的开机自启动脚本。每次在 WSL2 中打开一个新的终端窗口时，Bash 程序在启动之前，都会自动读取并执行 `~/.bashrc` 文件里的所有命令。可以在这个文件里放环境变量、命令别名、终端外观、自动执行的代码等等。

- 编辑 shell 配置文件，WSL2 Ubuntu 默认使用 Bash，配置文件是 `~/.bashrc`
	- 默认你已经在 Windows 上安装了 VS Code。在 WSL2 终端直接输入以下命令，可以在 Windows 的 VS Code 图形界面里直接打开这个文件。

```bash
code ~/.bashrc
```

- 在文件末尾添加：
	- `export GOROOT=/usr/local/go`：显式告诉系统和所有 Go 工具，Go 的安装目录在哪里。虽然 Go 可以自动检测，但显式设置可以避免某些工具找不到路径的奇怪问题。
	- `export GOPATH=$HOME/go`：Go 语言的用户工作区根目录，默认指向 `$HOME/go`。它负责存放全局依赖缓存（`pkg/mod`）和编译好的二进制工具（`bin`）
		- 理论上，可以设置 GOPATH 为任意目录（如 `$HOME/tools/go`）。但 Linux 系统的惯例和标准是：将用户级别的配置和缓存放在主目录下。因此遵循默认配置，不易出错。
	- `export PATH=...`：将 Go 的可执行文件目录（\$GOROOT/bin）和你自己安装的工具目录（\$GOPATH/bin）添加到系统的 PATH 中。这样你就可以在终端任意位置直接执行 go 命令和你自己安装的工具了

```bash
# Go 环境变量
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export PATH=$PATH:$GOROOT/bin:$GOPATH/bin
```

> [!NOTE]
> 如果没有 VS Code，则运行 `nano ~/.bashrc`。同样在文件末尾添加以上内容，然后保存并退出（在 `nano` 中按 `Ctrl+X`，然后按 `Y`，最后按 `Enter`）。

- 让配置立即生效：

```bash
source ~/.bashrc
```

在终端中输入以下命令，**验证** Go 是否安装成功:

```bash
go version
```




## 重装
如果想要删除go软件，重新安装新版本，则执行：
```bash
sudo rm -rf /usr/local/go
```