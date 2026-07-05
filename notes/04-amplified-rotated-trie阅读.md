# Amplified Rotated Trie 阅读

## 一、这一节在整篇论文中的位置

Section 3 给出了 `rotated radix trie`，已经说明：

- 可以不直接依赖传统 `hash functions`；
- 可以得到线性空间、常数时间、high probability 的 dictionary；
- 但它的 failure probability 还不够强。

所以 Section 4 的任务很明确：

> 把 warmup 结构推进成真正接近论文主结果的版本。

作者在开头直接写道：

> In this section, we modify the rotated radix trie to reduce its probability of failure ... to $\frac{1}{n^{n^{1-\epsilon}}}$.

这说明这一节不是修修补补，而是第一次真正实现摘要里最抓人的那个结论。

## 二、这一节的核心问题

Section 3 里，统一数组中的某个 bin 一旦超过容量 $\ell = \operatorname{polylog} n$，就会出问题。

这一节的第一步做法是：

- 把这些超出容量的球称为 `overflow balls`；
- 不再强行把它们塞进原来的 bin；
- 而是把它们转存到一个辅助结构 $Q$。

作者原文是：

> Whenever a ball $b$ is inserted into a bin $j$ that already contains $\ell = \operatorname{polylog} n$ other balls, the ball $b$ is regarded as an overflow ball.

以及：

> We instead store the overflow balls in a secondary data structure $Q$ that is implemented as a $n^\delta$-ary trie.

这一步的意图很好理解：

- 主结构继续保持简单、快速；
- 极少数麻烦元素交给备用结构处理。

但问题马上出现了。

## 三、为什么“加一个备用结构”还不够

辅助结构 $Q$ 虽然能在常数时间里支持这些 overflow balls 的查询和插入，但空间效率很差。

如果 overflow balls 的数量是 $q$，那么它的空间可能达到：

$$
q n^\delta.
$$

因此如果想保持总空间线性，就必须证明：

$$
\Pr[q \ge n^{1-\delta}] \le O\left(\frac{1}{n^{n^{1-\epsilon}}}\right).
$$

作者把这件事直接写成了公式 (1)。这其实就是这一节真正要完成的分析目标。

也就是说，这一节的关键不再只是“单个 bin 会不会爆”，而是：

> 全局来看，overflow balls 的总数能不能以超高概率保持极小。

## 四、旧结构为什么做不到这一点

作者随后给出一个非常关键的反例思路。

问题不在 balls 总数，而在依赖性。

如果很多 balls 来自同一个 source node，那么它们的去向会一起受到同一个 rotation $r_i$ 控制。这样一来，很多事件并不是近似独立的，而是强耦合的。

作者构造的病态场景是：

- rotated trie 只有 $2\ell$ 个 internal nodes；
- 每个 internal node 都有 $\Theta(n/\ell)$ 个 balls；
- 如果这 $2\ell$ 个节点刚好都满足 $r_i = 0$，
- 那么会有大约一半的 balls 同时涌向前面那批 bins，
- 结果就是大量 overflow。

作者原文总结得很直接：

> What makes the above pathological example possible is that it is possible to have only a small number of internal nodes in our rotated trie.

这句话非常重要，因为它指出了 Section 3 结果的真正瓶颈：

- 不是 Chernoff bound 不够强；
- 不是 dynamic fusion node 不够好；
- 而是影响结构的随机源太少。

如果 internal nodes 太少，那么真正决定 balls 落点的随机变量也太少，最后就很难得到 super-high-probability guarantees。

## 五、作者的修正：降低 fanout，强行增加 internal nodes 数量

作者的核心修正非常漂亮：

- 把 rotated radix trie 的 fanout 从 $n$ 降到 $n^\delta$；
- 这样每个 internal node 最多只含有 $n^\delta$ 个 balls；
- 因而总共有至少 $n^{1-\delta}$ 个 internal nodes。

作者原文是：

> To fix this problem, we reduce the fanout of our rotated radix trie from $n$ to $n^\delta$.

以及：

> This ensures that there are always at least $n^{1-\delta}\log n$ random bits affecting the trie's structure.

这一步的本质是：

> 通过降低每个节点“管辖”的规模，换取更多彼此独立的局部随机选择。

从设计思路上看，这相当于作者在主动制造“随机性分散化”。

## 六、balls-to-bins 映射其实没有变

虽然 fanout 变成了 $n^\delta$，但 balls-to-bins 的映射形式本身并没有本质变化。

每个 ball 仍写成 $(s,c)$，其中：

- $s \in [m]$ 是 source node；
- $c \in [n^\delta]$ 是 child index。

映射仍然是：

$$
\phi(s,c) = ((r_s + c) \bmod n).
$$

也就是说，作者并没有推翻 Section 3 的基本机制，而是在保留其核心形状的前提下，改变了参数区间，让它更适合做超高概率分析。

## 七、这一节真正的分析对象：overflow balls 总数 $q$

接下来作者不再逐个分析 bin，而是直接把

$$
q = \text{number of overflow balls}
$$

视为整体随机变量。

定义：

$$
f(r_1,\dots,r_m) := q.
$$

这里 $r_1,\dots,r_m$ 是所有 internal nodes 的随机 rotation。

这一步非常关键，因为它意味着作者准备使用一个“整体 concentration inequality”，而不是像 Section 3 那样只盯着固定的某个 bin。

## 八、为什么 McDiarmid 可以用上

作者此处调用的是 McDiarmid’s inequality。

原文先给出定义：

> Call a function $f : [0,1)^m \to \mathbb{R}$ $L$-Lipschitz if ...

然后给出 McDiarmid 的标准结论：如果输入变量独立，且函数对单个坐标变化不太敏感，那么整个函数会围绕其期望高度集中。

把这个工具放到本文场景中，关键就是证明：

1. $r_1,\dots,r_m$ 相互独立；
2. $f(r_1,\dots,r_m)=q$ 对单个 $r_i$ 的变化不太敏感。

作者给出的 Lipschitz 界是：

$$
L = n^\delta.
$$

原因很直接：

- 一个 rotation $r_i$ 只会影响 source node $i$ 发出的那些 balls；
- 而一个 source node 最多只有 $n^\delta$ 个 balls；
- 所以改动一个 $r_i$，最多只会让 $q$ 改变 $n^\delta$。

作者原文是：

> Observe that $f$ is $n^\delta$-Lipschitz, since each $r_i$ can determine the outcome of at most $n^\delta$ different balls.

这正是 McDiarmid 可以启动的关键。

## 九、为什么这个界足够强

作者还用到了一个事实：

$$
\mathbb{E}[q] = \frac{1}{\operatorname{poly}(n)}.
$$

于是通过 McDiarmid，可以推出：

$$
\Pr[q \ge n^{1-\delta}] \le e^{-\Omega(n^{1-4\delta})}.
$$

再令

$$
\delta = \epsilon / 5,
$$

就得到：

$$
\Pr[q \ge n^{1-\delta}] \le O\left(n^{-n^{1-\epsilon}}\right).
$$

这就正好完成了前面需要的公式 (1)。

这里最值得体会的是：作者没有直接把每个 ball 都搞成独立，而是换了一个更适合的分析目标。

也就是说，真正高明的地方不是“把依赖完全消灭”，而是：

> 找到一个即使存在局部依赖，也仍能集中分析的全局函数 $q$。

## 十、这一节得到的主结论

于是作者得到 Theorem 2：

> The $n^{\epsilon/5}$-ary amplified rotated radix trie is a randomized linear-space dictionary ...

其核心保证可以概括为：

- 存储 $n$ 个 $\Theta(\log n)$-bit keys/values；
- 线性空间；
- 每次操作常数时间；
- 失败概率为

$$
O\left(\frac{1}{n^{n^{1-\epsilon}}}\right).
$$

这就是摘要里第一条主结果的主体实现。

## 十一、为什么作者说它“接近最优”

这一节后半部分还有一个很重要的理论定位，也就是 Lemma 3。

作者说明：

> if there were to exist a randomized linear-space dictionary with failure probability of $1/n^{\epsilon n}$, that would imply the existence of a deterministic ... constant-time dictionary.

这句话的意义是：

- amplified rotated trie 虽然没有做到 $\frac{1}{n^{\epsilon n}}$ 这种更强的界；
- 但如果真能做到那种级别，就会碰到 deterministic constant-time dictionary 这个更大的 open question。

所以作者想表达的是：

> 当前结果已经逼近了理论上“再往前走就会很危险”的边界。

这也解释了为什么摘要里会说这个结果是 “nearly optimal”。

## 十二、这一节最该记住的思想

我觉得 Section 4 最值得记住的不是具体常数，而是下面这条升级路线：

1. Section 3 先证明“不用 hash functions 也能做出像 hash table 的结构”；
2. Section 4 发现真正卡住超高概率的是随机源过少、依赖过强；
3. 解决办法不是回到传统 hash-function family，而是重新组织 trie 的 fanout；
4. 然后用 McDiarmid 直接控制全局 overflow 数量。

这说明作者的方法论非常一致：

> 先从数据结构形状上改造随机性的分布，再选择与这种分布匹配的概率工具。

## 十三、可以怎样理解这一节与摘要的对应关系

摘要第二段说：

> Building on this, we are able to construct a hash table using $O(n)$ random bits that achieves failure probability $1 / n^{n^{1-\epsilon}}$ ...

这里的 “Building on this” 指的就是：

- 先有 rotated radix trie 这个 warmup；
- 再在其上通过 fanout 调整和 overflow 控制，
- 得到 amplified rotated trie。

所以如果把摘要和正文对照来看，Section 4 就是在把摘要的第二段完整落地。

## 十四、Mermaid 结构图

```mermaid
flowchart TD
    A[Section 3 的 rotated trie]
    B[已有 high probability 但还不够强]
    C[尝试把 overflow balls 放进辅助结构 Q]
    D[目标：控制 overflow 总数 q]
    E[旧结构失败：source node 太少导致强依赖]
    F[修正：把 fanout 从 n 降到 n^delta]
    G[结果：至少有 n^(1-delta) 个 internal nodes]
    H[更多独立随机变量共同决定结构]
    I[定义 q = f(r1 ... rm)]
    J[证明 f 是 n^delta-Lipschitz]
    K[应用 McDiarmid]
    L[得到 q 超过阈值的概率极小]
    M[得到 amplified rotated trie]
    N[失败概率达到 1 over n to the n to the 1 minus epsilon]

    A --> B
    B --> C
    C --> D
    D --> E
    E --> F
    F --> G
    G --> H
    H --> I
    I --> J
    J --> K
    K --> L
    L --> M
    M --> N
```

## 十五、接下来读 Section 5 时要带着的问题

读完这一节后，下一节最值得带着的几个问题是：

1. Section 4 已经能把 failure probability 压得极低，为什么还要再设计 `budget rotated trie`？
2. Section 5 为什么要把随机比特数从 $O(n)$ 进一步压到接近 $\tilde{O}(\log n)$？
3. `gradually-increasing-independence hash functions` 为什么在这里突然变得可用了？
4. Section 5 和 Section 6 的 succinct transformation 之间到底有什么结构上的衔接？

## 十六、简短总结

Section 4 的核心贡献，是把 `rotated trie` 从一个高概率 warmup，推进成一个几乎达到理论极限的结果。作者真正解决的问题不是“单个 bin 会不会太满”，而是“overflow 总量能不能以超高概率保持极小”。为此他降低 fanout、增加 internal nodes 数量、分散随机性来源，并最终用 McDiarmid 控制全局变量 $q$。这一节标志着论文已经从“概念上绕开 hash functions”进入“定量上压出极强保证”的阶段。
