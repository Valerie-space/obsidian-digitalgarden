---
{"dg-publish":true,"dg-permalink":"git1","permalink":"/git1/","title":"Git 的配置与仓库初始化（WSL2）","created":"2026-07-01T16:53:50.129+08:00","updated":"2026-08-22T16:46:49.682+08:00","dg-note-properties":{"title":"Git 的配置与仓库初始化（WSL2）","aliases":[null],"created":"2026-07-01","updated":"2026-08-22 16:45","type":"skill","description":null,"status":"done"}}
---


# Git 的配置与仓库初始化（WSL2）

> 参考：
> - 书籍：[Git](https://git-scm.com/book/zh/v2)
> - 游戏：[Learn Git Branching](https://learngitbranching.js.org/?locale=zh_CN)

## 安装 Git 与配置

> 如果是 Windows 环境，可以参考这篇教程的第一步到第六步 [【2025年最新版】Git安装及环境配置超详细教程（以win11为例子）_git安装及配置教程-CSDN博客](https://blog.csdn.net/Little_Carter/article/details/155110165) 

打开 WSL 终端，依次执行以下命令：

```bash
# 更新包列表（在安装任何软件之前，都建议先更新包列表）
sudo apt update

# 安装Git
sudo apt install git

# 验证Git是否成功安装
git --version
```

安装完成后，需要配置 Git 的用户名和电子邮件，在提交时能够标识个人身份。

```bash
# 设置提交时显示的署名名称
git config --global user.name "xxxx"

# 设置你的 GitHub 邮箱（确保填 GitHub 绑定的邮箱，否则绿墙不记录绿点）
git config --global user.email "your_email@example.com"

# 验证是否设置成功
git config --global --list
```

## 生成 SSH Key（每台设备仅首次需配置）

> [!NOTE]
> SSH Key（SSH 密钥）本质上是一对基于非对称加密算法生成的公钥（Public Key）和私钥（Private Key）。
> 
> 在设备本地生成一对公私钥，私钥由个人保管，公钥放到 GitHub 或其她服务器上。
> 
> 例如，当我们执行将代码上传至 GitHub 仓库时，GitHub 上的公钥将会和设备本地的私钥通过算法进行匹配，匹配成功即意味着这台设备有权限上传代码。
> 
> SSH Key 负责的是通信和权限，用来验证当前设备是否有权限。
> git config 负责的是署名，记录提交的所属人。

### 1. 检查本地是否已生成过 SSH Key

在终端运行以下命令：

```shell
# Windows运行：
dir ~/.ssh -Force

# Linux运行：
ls -la ~/.ssh
```

如果有以下内容，则说明已经生成过 SSH Key。那么跳过第二步，直接进行第三步（将公钥添加到 GitHub）即可。

- `id_ed25519_标识符`：一个 Ed25519 类型的私钥。必须保密，不能泄露给任何人或上传到公开仓库。
- `id_ed25519_标识符.pub`：对应的公钥。可以安全地添加到 GitHub、GitLab 等服务的 SSH Keys 设置中。

如果显示 ` No such file or directory`，说明未配置 SSH Key。

### 2. 本地生成 SSH Key

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

系统交互：

1. `Enter file in which to save the key (/home/username/.ssh/id_ed25519):`，回车即可，SSH Key 会保存在默认目录下
2. `Enter passphrase (empty for no passphrase)`，即输入密码。回车即可留空，这样使用 SSH 时无需输入密码。

参数说明：

- ed25519：使用 ed25519 算法。
- `-C "your_email@example.com"`：Comment，即注释。
	- 生成公私钥和这个邮箱无关，但如果服务器后台添加了多台设备的公钥，这些注释能够方便人类辨认出这把钥匙的所有人。
	- 注释内容不一定是要邮箱，也可以是任意文本，比如 `ssh-keygen -t ed25519 -C "ROG-Laptop-Ubuntu"`，写邮箱仅仅是因为开源社区的推荐。

### 3. 将公钥添加到 GitHub

在终端查看并复制公钥：

```bash
cat ~/.ssh/id_ed25519.pub
```

- 全选并复制终端输出的整段文本（通常以 `ssh-ed25519 AAAA...` 开头，以你的注释内容结尾）。

在 GitHub 网页进行操作：

1. 登录 GitHub，点击右上角头像 $\rightarrow$ Settings。
2. 左侧菜单找到并点击 SSH and GPG keys。
3. 点击右上角绿色的 New SSH key。
4. 填写信息：
	
	- Title：起一个便于识别的名称（如 `ROG-Laptop` 或 `Ubuntu-WSL`）。
	- Key type：保持默认的 `Authentication Key`。
	- Key：将刚才复制的公钥文本完整粘贴进去。
		
5. 点击 Add SSH key，按提示输入 GitHub 密码完成验证。

在终端验证是否添加成功：

```bash
ssh -T git@github.com
```

- 这条命令本质上是在主动向 GitHub 的 SSH 服务器发起一次连接请求，以此验证“你的本地私钥与 GitHub 上的公钥能否配对成功”。
- 如果提示 `Are you sure you want to continue connecting (yes/no/[fingerprint])?`，输入 **`yes`** 并回车。
- 看到以下输出即代表配置成功：

```bash
Hi <你的用户名>! You've successfully authenticated, but GitHub does not provide shell access.
```

> [!NOTE]
> 没有做额外配置时，SSH 客户端会按照默认的优先级顺序去 `~/.ssh/` 目录下挨个尝试密钥，直到找到能配对成功的那一个。
> 
> 如果同一台电脑存在多个 SSH 密钥（GitHub 要求同一个 SSH 公钥绝对不能被绑定到两个不同的 GitHub 账号下），则需要使用 `~/.ssh/config` 配置文件进行路由。它可以通过为同一个服务器定义不同的 Host 别名，将不同的私钥精确路由到对应的账号。
> 
> 具体操作这里不展开说明。

## 创建 GitHub 仓库

在网页端建一个完全空白的仓库

- 在 GitHub 点 New repository，输入仓库名。
- 注意：不要勾选 Add a README、.gitignore 或 License（保持仓库完全为空）。
- 点击 Create repository，页面会停在一个带有命令提示的界面。

创建完成后，复制仓库的 SSH 地址。例如 `git@github.com:your-username/my-project.git`

> [!NOTE]
> 本地生成 SSH key 和仓库的 SSH 地址负责的是不同的事情：
> - 本地生成的 SSH key 相当于自己的身份凭证，负责的是权限
> - 仓库的 SSH 地址负责的是告诉本地 Git 应该把代码推送到云端的哪个仓库

## 本地初始化仓库

打开终端：

```bash
# 1. 切换到项目所在目录（请根据实际路径调整）
cd ~/projects/my-project

# 2. 初始化本地 Git 仓库
git init

# 3. 关联在 GitHub 上新建的远程仓库（请根据实际路径调整）
git remote add origin git@github.com:your-username/my-project.git
```

在仓库根目录创建一个.gitignore 文件，排除不想同步的文件。如：

- 实际排除什么文件，建议问问 ai

```bash
*.log
tmp/
temp/
```

## 首次提交并推送至 GitHub

```bash
# 1. 将所有文件添加到暂存区
git add .

# 2. 提交到本地版本库
git commit -m "Initial commit"

# 3. 重命名本地主分支为main（因为本地创建的主分支可能叫master，而GitHub上新建仓库的主分支名为main，要保持主分支命名一致）
git branch -M main

# 推送到远程仓库
git push -u origin main

# 如果推送失败，试一下能否连接，若成功，再次推送
ssh -T git@github.com
```

如果正在使用代理服务器，可能导致连接失败（回国试试）。

## 其她操作

删除原有的 Git 历史记录：

```bash
# Windows PowerShell 环境执行: 
Remove-Item -Recurse -Force .git 

# macOS / Linux / Git Bash 环境执行: 
rm -rf .git
```
