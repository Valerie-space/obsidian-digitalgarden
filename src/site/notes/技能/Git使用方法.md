---
{"dg-publish":true,"permalink":"//git/","title":"Git使用方法","created":"2026-07-01T16:53:50.129+08:00","updated":"2026-08-22T13:44:45.020+08:00","dg-note-properties":{"title":"Git使用方法","aliases":[null],"created":"2026-07-01","updated":"2026-08-22 13:43","type":"skill","description":null,"status":"active"}}
---


# Git 使用方法（WSL2 环境）

> 参考：
> - 书籍：[Git](https://git-scm.com/book/zh/v2)
> - 游戏：[Learn Git Branching](https://learngitbranching.js.org/?locale=zh_CN)

## 仓库初始化

### 安装 Git 与配置

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



### 生成 SSH Key（每台设备仅首次需配置）

> [!NOTE]
> SSH Key（SSH 密钥）本质上是一对基于非对称加密算法生成的公钥（Public Key）和私钥（Private Key）。
> 
> 在设备本地生成一对公私钥，私钥由个人保管，公钥放到 GitHub 或其她服务器上。
> 
> 例如，当我们执行将代码上传至 GitHub 仓库时，GitHub 上的公钥将会和设备本地的私钥通过算法进行匹配，匹配成功即意味着这台设备有权限上传代码。
> 
> SSH Key 负责的是通信和权限，用来验证当前设备是否有权限。
> git config 负责的是署名，记录提交的所属人。

#### 1. 检查本地是否已生成过 SSH Key

在终端运行以下命令：

```shell
# Windows运行：
dir ~/.ssh -Force

# Linux运行：
ls -la ~/.ssh
```

如果有以下内容，则说明已经生成过 SSH Key。那么跳过第二步，直接进行第三步（将公钥添加到GitHub）即可。

- `id_ed25519_标识符`：一个 Ed25519 类型的私钥。必须保密，不能泄露给任何人或上传到公开仓库。
- `id_ed25519_标识符.pub`：对应的公钥。可以安全地添加到 GitHub、GitLab 等服务的 SSH Keys 设置中。

如果显示 ` No such file or directory`，说明未配置 SSH Key。

#### 2. 本地生成 SSH Key

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



#### 3. 将公钥添加到 GitHub

在终端查看并复制公钥：

```bash
cat ~/.ssh/id_ed25519.pub
```

- 全选并复制终端输出的整段文本（通常以 `ssh-ed25519 AAAA...` 开头，以你的注释内容结尾）。

在GitHub网页进行操作：
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

### 创建 GitHub 仓库

在网页端建一个完全空白的仓库

- 在 GitHub 点 New repository，输入仓库名。
- 注意：不要勾选 Add a README、.gitignore 或 License（保持仓库完全为空）。
- 点击 Create repository，页面会停在一个带有命令提示的界面。

创建完成后，复制仓库的 SSH 地址。例如`git@github.com:your-username/my-project.git`

> [!NOTE]
> 本地生成 SSH key 和仓库的 SSH 地址负责的是不同的事情：
> - 本地生成的 SSH key 相当于自己的身份凭证，负责的是权限
> - 仓库的 SSH 地址负责的是告诉本地Git应该把代码推送到云端的哪个仓库

### 本地初始化仓库

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
- 实际排除什么文件，建议问问ai
```bash
*.log
tmp/
temp/
```

### 首次提交并推送至GitHub

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

### 其她操作

删除原有的 Git 历史记录：

```bash
# Windows PowerShell 环境执行: 
Remove-Item -Recurse -Force .git 

# macOS / Linux / Git Bash 环境执行: 
rm -rf .git
```


## Git 核心操作

.gitignore 文件中，一般把依赖模块忽略了，让用户自己 npm

![Pasted image 20260614040152.png](/img/user/%E9%99%84%E4%BB%B6/%E5%9B%BE%E7%89%87/_%E9%99%84%E4%BB%B6/zyq/Pasted%20image%2020260614040152.png)

基础工作流

- clone
- add
- commit
- push
- pull
历史与差异
- log（查看历史）
- diff（比较两次提交的差异）
分支管理
- branch（创建/删除）
- checkout/switch（切换）
- merge（合并）
撤销与修正
- discard （放弃还没 commit 的文件更改）
- reset（将仓库强制回退到某个历史状态。有搞丢代码的发现，多人协作禁止使用）
- revert（安全撤销，撤销某一次具体的 commit）
- commit --amend（修正提交）

内部协作：

开发者提交分支前，先 pull 主干，检查是否有冲突。然后再合并。由管理员审核代码

![Pasted image 20260614042042.png](/img/user/%E9%99%84%E4%BB%B6/%E5%9B%BE%E7%89%87/_%E9%99%84%E4%BB%B6/zyq/Pasted%20image%2020260614042042.png)

![Pasted image 20260614041733.png](/img/user/%E9%99%84%E4%BB%B6/%E5%9B%BE%E7%89%87/_%E9%99%84%E4%BB%B6/zyq/Pasted%20image%2020260614041733.png)

如果写某分支的代码时，没有提交，那么切换分支后，这个未提交的代码会被带过去。此时可以考虑

- 提交分支代码
- 暂存（Stash）

## 版本控制规范

[语义化版本 2.0.0 | Semantic Versioning](https://semver.org/lang/zh-CN/spec/v2.0.0.html)

版本格式：主版本号. 次版本号. 修订号，版本号递增规则如下

1. 主版本号：进行不向下兼容的修改时，递增主版本号。
2. 次版本号：API 保持向下兼容的新增及修改时，递增次版本号
3. 修订号：修复问题但不影响 API 时，递增修订号
先行版本号及版本编译信息可以加到“主版本号. 次版本号. 修订号”的后面，作为延伸。

对于一个开源软件，可以：

1. 主版本号：做了不兼容的改动，例如配置文件格式重写、插件接口大改、不再支持旧的用户数据。
2. 次版本号：增加了新功能（新窗口、新菜单项、新命令行参数），且完全不影响旧功能的使用习惯。
3. 修订号：只修复 bug、优化性能、调整界面文案（不改变已有的功能）。

预发布版本：

- `1.0.0-alpha` – 内部早期测试版
- `1.0.0-beta` – 公开测试版，功能基本完整
- `1.0.0-rc.1` – 发布候选版（Release Candidate），如无重大问题就作为正式版

### 在 0.y.z 初始开发阶段，我该如何进行版本控制？

最简单的做法是以 0.1.0 作为你的初始化开发版本，并在后续的每次发行时递增次版本号。

### “v1.2.3” 是一个语义化版本号吗？

“v1.2.3” 并不是的一个语义化的版本号。但是，在语义化版本号之前增加前缀 “v” 是用来表示版本号的常用做法。在版本控制系统中，将 “version” 缩写为 “v” 是很常见的。比如：`git tag v1.2.3 -m "Release version 1.2.3"` 中，“v1.2.3” 表示标签名称，而 “1.2.3” 是语义化版本号。

## git 操作 - 基础

> https://learngitbranching.js.org

提交：

`git commit`

创建分支：

```bash
git branch abc    //创建分支
git checkout abc  //进入分支
git commit        //提交
```

---

合并分支：merge 方法

- **公共主分支用 merge**

```bash
//当前在main
git merge abc  //把abc的修改合并进main，创建新的提交记录（但abc指针并没有指向新记录）

//接下来相当于把abc移动到main所指的那个提交记录
git checkout abc
git merge main
```

合并分支：rebase 方法（变基）。

- 个人功能分支可以用 rebase
- 实际上就是取出一系列的提交记录，“复制”它们，然后在另外一个地方逐个的放下去。Rebase 的优势就是可以创造更线性的提交历史

```bash
git checkout abc
git rebase main  //abc分支变更基点，新建一个提交记录，新记录的parent指向main分支指向的记录

git checkout main
git rebase abc     //main分支包含于新节点，所以不新建提交记录，main指针直接指向新结点。
```

交互式变基：就是指使用带参数 `--interactive`（或简写为 `-i`）的 `rebase` 命令。

- 如果你在命令后增加了这个选项, Git 会打开一个用户界面，向你展示哪些提交即将被复制到变基目标的下方。它还会显示这些提交的哈希值和提交信息，这对于理清脉络非常有帮助。
- 可以保留、丢弃提交、修改提交信息、编辑提交本身
- 把代码合并到主分支之前，可以用交互式变基来整理自己的提交记录。（多次 git commit 可能提交信息非常杂乱，但提交时可以交互式变基来整理提交信息、压缩成一个 commit）
- **只对“尚未推送到远程仓库”的本地提交进行交互式变基。**

```bash
//当前分支搬到 <目标> 上，并且在搬的过程中，让你编辑（压缩/修改/删除）从 <目标> 之后到当前 HEAD 之间的所有提交。
//目标本身不会被修改
git rebase -i <目标>  
```

---

指向历史提交（而非分支）：HEAD

- HEAD 未分离时：HEAD -> main -> C1 (这里 C1 表示某次提交记录的哈希值)
- 进行分离 `git checkout C1` （可以分离到任意位置）
- HEAD 分离后：HEAD -> C1

查看提交记录的哈希值 `git log`

指向提交记录 - 相对引用（无需哈希值）。可以从一个易于记忆的地方（比如 `bugFix` 分支或 `HEAD`）开始计算，然后移动

- 使用 `^` 向上移动 1 个提交记录
- 使用 `~<num>` 向上移动多个提交记录，如 `git checkout HEAD~3`

```bash
git checkout main^^  //切换到main的母节点的母节点
```

也可以用 HEAD 作为相对引用的参照

```bash
git checkout C3
git checkout HEAD^
git checkout HEAD^
git checkout HEAD^
```

如果 HEAD 前有两个母节点,则可以用 `git checkout HEAD^2` 移动到第二个母节点

以上操作可以链式

```bash
git checkout HEAD^^2~3
```

---

强制修改分支位置

- _在真实的 Git 环境中，你当前所在的分支上不允许执行 `git branch -f`。_

```bash
git branch -f main HEAD~3
//这条命令会将 main 分支强制指向 HEAD 的第 3 级 parent 提交。
```

---

撤销变更

方法 1：reset

- 分支从 C2 移回 C1，但 C2 所作的变更还在，但是处于未加入暂存区状态。
- reset **只能改写本地分支**，不能改写远程分支
- reset 的参数是“要去哪”

```bash
git reset HEAD~1
```

方法 2：revert

- **可以改写远程分支**
- 实际上是新增了一个提交记录，用来抵消原来的更改
- revert 的参数是“要撤销谁”

```bash
git revert HEAD
```

---

## git 操作 - 进阶

复制其她分支的提交到当前分支

- 可以用来合并分支，但必须要知道提交记录的哈希值
- `git cherry-pick <提交1> <提交2> <...>`

---

写了一个用于解决 bug 的分支，bug 解决后，遗留许多调试性代码。这时候可以利用 `git rebase -i` 和 `git cherry-pick` 来只提交包含解决 bug 的那次提交，这样调试性代码就都不会被提交到主分支了。

---

相信通过前面课程的学习你已经发现了：分支很容易被人为移动，并且当有新的提交时，它也会移动。分支很容易被改变，大部分分支还只是临时的，并且还一直在变。

你可能会问了：有没有什么可以 _永远_ 指向某个提交记录的标识呢，比如软件发布新的大版本，或者是修正一些重要的 Bug 或是增加了某些新特性，有没有比分支更好的可以永远指向这些提交的方法呢？

```bash
git tag v1 C1

这样就可以随时切换到v1了
git checkout v1
```

`git describe` 的​​语法是：`git describe <ref>`

- `<ref>` 可以是任何能被 Git 识别成提交记录的引用，如果你没有指定的话，Git 会使用你目前所在的位置（`HEAD`）。
它输出的结果是这样的：`<tag>-<numCommits>-g<hash>`
- `tag` 表示的是离 `ref` 最近的标签， `numCommits` 是表示这个 `ref` 与 `tag` 相差有多少个提交记录， `hash` 表示的是你所给定的 `ref` 所表示的提交记录哈希值的前几位。
- 当 `ref` 提交记录上有某个标签时，则只输出标签名称

例如

![Git使用方法-1784921420857.webp\|697](/img/user/%E9%99%84%E4%BB%B6/%E5%9B%BE%E7%89%87/Git%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95-1784921420857.webp)

`git describe main` 会输出：`v1-2-gC2`

`git describe side` 会输出：`v2-1-gC4`
