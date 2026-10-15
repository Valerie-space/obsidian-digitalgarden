---
{"dg-publish":true,"dg-permalink":"aa","permalink":"/aa/","title":"FUSE攻击不可行的证明","hide":true,"created":"2026-10-15T21:24:48.660+08:00","updated":"2026-10-16T03:05:01.033+08:00","dg-note-properties":{"project":"[[项目/读研/读研]]","title":"FUSE攻击不可行的证明","aliases":["FUSE攻击不可行的证明"],"created":"2026-10-15","updated":"2026-10-16 03:05","finished":null,"type":"subproject","description":null,"status":"active"}}
---


# AA 机制

> 用电脑看更方便看脚注。脚注里会标注原文的内容。

AA 区块和稳定区块的定义：

1. 某个区块是 $AA_t$ ，当且仅当：**区块在 slot t 中拥有足够投票**（这个区块不一定是 slot t 时提议的）[^1]
2. slot t 提议的 $b$ 是稳定块，当且仅当：**它的 parent $b'$ 在 slot t-1 获得了足够投票**，成为 $AA_{t-1}$ （parent 不一定是 slot t-1 提议的）[^2]

AA 区块和稳定区块的关系：

- AA 区块一定是稳定区块；不稳定区块可能是 AA、也可能不是 AA

> 证明：如果当前区块 b 是不稳定区块，那么 parent(b) 一定是非 AA。parent(b) 是稳定的情况下，当前 slot 的验证者会把票投给 parent(b)，于是 b 变成不稳定的非 AA 区块，而 parent(b) 变成了稳定的 AA 区块。

slot t 的区块 b 的 parent 选择逻辑：[^7]

1. 如果 $AA_{t-1}$ 存在，则 parent(b)=$AA_{t-1}$
2. 否则 parent 为 ForkChoice 的输出。

> AA 的实验代码里，在 `proposer.go`（以及一系列调用文件）中，每个新提议的区块所选择的 parent 都是通过 forkchoice 来获取的。跳过了第一步，但结果是一样的。

诚实 attestor 只可能投票给 ForkChoice 的输出 [^4]

ForkChoice 只可能输出**稳定区块**[^6][^8]

## 原文参考

AA 中所有的，FUSE 所依赖的机制和定义都在这了（除了转发和 u 引用）

由于这篇是二次整理，标章节号什么的有点麻烦，就不标了

### 非重要，仅相关

> 这两句非常非常有误导性，看起来定义的是：区块获得 1/3 投票，就是 stable block。但第六章及以后的伪代码、定义、流程等，都不是这样的。
> 
> （引言）We borrow this concept toEthereum PoS and define **blocks with at least one-third votes as stable blocks**.This guarantees that each stable block is received by at least honest validators. Onethird is the minimum number that can achieve the goal
> 
> （第五章）we define AA as a mechanism that checks whether a block receives matching attestations from at least one-third of validators .In particular, if b is a block for slot t, its child must include attestations for b from at least one-third of validators in slot t.
> **We use the notion of stable block to describe this scenario.**
>
> **Definition 2** (*unstable block*). Block $b$ is an unstable block if $b$ is not a stable block.

> **Remark.** We do not introduce the concept of *weight* in this paper. In Ethereum, the weight of an attestation denotes the fraction of the stake of the attestor. Accordingly, **a block $b$ is stable if the weight of attestations included in $b$ (for the parent block $b'$) is higher than $\vartheta'$,** where $\vartheta'$ is the fraction of stake owned by Byzantine validators.
> 这里也可以确认，区块是否稳定，是取决于它的 parent 的

> Upon receiving a proposed block $b$ in slot $t'$, validator $v_i$ includes $b$ in its block tree $\mathcal{T}$ and checks whether $b$ is a stable block, i.e., $b$ includes at least $\vartheta + 1$ attestations in slot $t' - 1$ (the *head* of the attestations is the parent of $b$). If so, $v_i$ processes the stable block. Otherwise, block $b$ is an unstable block (lines 24-30 in Figure 11).
> 可以证明，判断 b 是否稳定，是看上一个时隙的投票数。而上一个时隙的投票是投给 b 的 parent 的

### 重要

[^1]: > If $f + 1$ validators vote for a block $b$ in slot $t$, at least one honest validator has validated $b$. The $f + 1$ attestations become a *proof* of the availability of $b$. **We use AA$_t$ to denote a block with $f + 1$ attestations in slot $t$.** 

[^2]: > **Definition 1** (*stable block*). A block $b$ proposed in slot $t$ is a *stable block*, if $b$ includes at least $\vartheta + 1$ attestations for $b'$ in slot $t - 1$, where $b'$ is the parent block of $b$.

[^4]: ![Pasted image 20261016012629.png](/img/user/%E9%99%84%E4%BB%B6/%E5%9B%BE%E7%89%87/Pasted%20image%2020261016012629.png)

[^6]: > **Definition 3** (*stable chain*). The chain $c$ is a stable chain, if the leaf block of $c$ is a stable block.

	> We replace the HLMD GHOST with the *longest chain* as the fork choice rule. Our longest chain fork choice rule is very simple: it outputs the head of the chain with the most stable blocks. In case of a tie, the longest chain fork choice rule chooses the chain such that the leaf block has the largest slot number. (lines 33-42 in Figure 11).

	![Pasted image 20261016012908.png](/img/user/%E9%99%84%E4%BB%B6/%E5%9B%BE%E7%89%87/Pasted%20image%2020261016012908.png)

[^7]: ![Pasted image 20261016013600.png](/img/user/%E9%99%84%E4%BB%B6/%E5%9B%BE%E7%89%87/Pasted%20image%2020261016013600.png)

	> If a **stable block** $p$ with $AA_{t-1}$ exists, $v_i$ sets the *parent* field as $p$.  
	> - p 需要同时是 stable 和 AA，slot t 才会选择 p。不过可以证明 AA 必然 stable
	> 
	> If two $AA_{t-1}$ blocks exist, $v_i$ breaks the tie by randomly selecting one. 
	> Otherwise, $v_i$ uses the output of function `FORKCHOICE` as the parent (lines 3-5 in Figure 11).
	> 
	> Note that there exists a case where a Byzantine validator $v_j$ proposes two conflicting blocks in slot $t$ and both blocks form $AA_{t-1}$. Upon receiving such blocks, $v_i$ still randomly selects one block. The proposer of the two blocks will eventually be slashed.

[^8]: 	> **The longest chain fork choice rule.** Our rule selects the chain with the most number of stable blocks, i.e., the chain **led by block $b_2$** 
	>
	> Note that **unstable blocks are not considered in our new fork choice rule**. 
	> 不稳定区块不会被 ForkChoice 输出
	> When the network is synchronous, we can in fact directly ignore unstable blocks. To deal with network asynchrony, we still need to consider unstable blocks. In our protocol, unstable blocks can be included in the blocks if the transactions in unstable blocks do not conflict (i.e., $b_4$ will in the $u$ field of some block in $c_2$ )

	![Pasted image 20261015200056.png](/img/user/%E9%99%84%E4%BB%B6/%E5%9B%BE%E7%89%87/Pasted%20image%2020261015200056.png)

	- root 为最后一个被验证的 checkpoint
	- 本规则会选择包含稳定块数量最多的链，即 b2 引导的链。
		- 注意!!!选择的是 b2 而不是 b4！链 c_2 的头部是 b_2 而不是 b4！
