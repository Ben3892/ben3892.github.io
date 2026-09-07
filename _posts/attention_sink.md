# 为什么 Transformer 会产生 Attention Sink？

在理解 Attention Sink 之前，需要先接受一个非常重要的事实：

$$
\boxed{
\text{Attention weight 高，不等于这个 token 对输出贡献大。}
}
$$

原因是，一个 Attention head 真正执行的是：

$$
o_i
=
\sum_j
\alpha_{ij}
W_OW_Vx_j
$$

其中：

$$
\alpha_{ij}
=
softmax_j(s_{ij})
$$

而：

$$
s_{ij}
=
x_i^TA_{j-i}x_j
$$

因此 Attention 至少包含两个完全不同的问题：

$$
\boxed{
QK:
\text{从哪里读？}
}
$$

以及：

$$
\boxed{
OV:
\text{读到以后写什么？}
}
$$

Attention Sink 正是这两部分配合产生的一个很有意思的现象。

---

## 1. 什么是 Attention Sink？

假设某个位置 \(s\) 对大量不同 query 都获得异常高的 Attention：

$$
\alpha_{is}\gg\alpha_{ij},
\qquad
j\neq s
$$

例如：

$$
\alpha_i
=
[0.9,0.02,0.01,\ldots]
$$

而第一个位置可能只是 BOS，甚至没有明显的语义价值。

这种位置就被称为：

$$
\boxed{
Attention Sink
}
$$

直觉上很容易产生一个误解：

> 模型一定认为这个 token 特别重要。

但事实并不一定如此。

---

# 2. 为什么 Attention weight 不能直接解释成语义重要性？

真正写回 residual stream 的是：

$$
\alpha_{ij}
W_OW_Vx_j
$$

而不是：

$$
\alpha_{ij}
$$

本身。

所以即使：

$$
\alpha_{is}\approx1
$$

只要：

$$
W_OW_Vx_s\approx0
$$

那么这个位置对实际输出的贡献仍然可能很小。

因此：

$$
\boxed{
\text{Routing importance}
\neq
\text{Information contribution}
}
$$

Attention Sink 首先是一个 routing 现象，而不一定是一个 semantic importance 现象。

---

# 3. 为什么 Softmax 天然容易产生 Sink？

标准 Attention 使用：

$$
\alpha_i
=
softmax(s_i)
$$

这意味着：

$$
\alpha_{ij}\ge0
$$

并且：

$$
\boxed{
\sum_j\alpha_{ij}=1
}
$$

这看起来非常自然，但它带来了一个结构性限制：

$$
\boxed{
\text{Attention 必须把 100\% 的概率质量分配出去。}
}
$$

这意味着一个 head 永远不能直接表达：

> 我现在什么都不想读。

---

# 4. 但真实计算显然需要 no-op

假设某个 head 的功能是：

> 如果当前位置是代词，就寻找 antecedent。

当它真的看到代词时：

$$
\text{需要工作}
$$

但如果当前 token 是一个数字、标点、代码符号，可能根本没有它需要寻找的关系。

理想输出应该是：

$$
\boxed{
o_i=0
}
$$

也就是说：

> 这个 head 当前关闭。

但 Softmax 不允许：

$$
\alpha_{i1}
=
\alpha_{i2}
=
\cdots
=
0
$$

因为：

$$
\sum_j\alpha_{ij}=1
$$

所以模型必须解决一个问题：

$$
\boxed{
\text{如何在“必须选一个位置”的情况下实现 no-op？}
}
$$

Attention Sink 就是一个可能的解决方案。

---

# 5. 模型可以自己创造一个 NULL address

假设模型选出某个位置 \(s\)，让它满足两个条件。

第一：

$$
\boxed{
\text{非常容易被 QK 找到}
}
$$

也就是：

$$
q_i^Tk_s
$$

对很多 query 都很高。

第二：

$$
\boxed{
\text{它的 OV 输出非常小或足够稳定}
}
$$

即：

$$
W_OW_Vx_s
\approx0
$$

那么：

$$
\alpha_{is}\approx1
$$

时：

$$
o_i
\approx
W_OW_Vx_s
\approx0
$$

于是这个 head 实现了：

$$
\boxed{
\text{“我必须选择一个位置，所以我选择 NULL。”}
}
$$

也就是说：

$$
\boxed{
Attention Sink
\approx
\text{Implicit NULL Memory Slot}
}
$$

---

# 6. Sink 和匹配矩阵 \(A_\Delta\) 有什么关系？

带 RoPE 时：

$$
s_{ij}
=
x_i^TA_{j-i}x_j
$$

对于 sink token \(s\)：

$$
s_{is}
=
x_i^TA_{s-i}x_s
$$

定义：

$$
u_{s,\Delta}
=
A_\Delta x_s
$$

那么：

$$
s_{is}
=
x_i^Tu_{s,\Delta}
$$

如果对于大量 query：

$$
x_i^Tu_{s,\Delta}
$$

都很高，那么：

$$
\boxed{
A_\Delta x_s
}
$$

就相当于一个非常容易与 query 对齐的方向。

因此从我们之前的双线性型视角看：

$$
\boxed{
Sink 的 QK 本质
=
A_\Delta
\text{ 把某个 token 变成了 universal attractive address}
}
$$

也就是说，Sink 与 \(A_\Delta\) 的关系不是：

> \(A_\Delta\) 出问题了。

而是：

> \(A_\Delta\) 学会了一个特殊用途的关系。

---

# 7. 但只看 \(A_\Delta\) 不够

Attention head 至少分成：

$$
\boxed{
QK circuit
}
$$

和：

$$
\boxed{
OV circuit
}
$$

Sink 的完整条件更像：

$$
\boxed{
\text{QK 强吸引}
+
\text{OV 弱输出}
}
$$

如果只有 QK 强吸引，而：

$$
W_OW_Vx_s
$$

本身非常巨大，那么这个位置就不是 harmless sink，而会持续向 residual stream 写入大量内容。

真正功能性的 sink 更接近：

$$
\boxed{
\text{强地址、弱 payload}
}
$$

---

# 8. Attention Sink 可以等价成一个隐式 Gate

假设：

$$
v_s\approx0
$$

那么：

$$
o_i
=
\alpha_{is}v_s
+
\sum_{j\neq s}\alpha_{ij}v_j
$$

近似：

$$
o_i
=
\sum_{j\neq s}\alpha_{ij}v_j
$$

定义：

$$
g_i
=
1-\alpha_{is}
$$

同时：

$$
\tilde\alpha_{ij}
=
\frac{
\alpha_{ij}
}{
1-\alpha_{is}
}
$$

那么：

$$
\boxed{
o_i
=
g_i
\sum_{j\neq s}
\tilde\alpha_{ij}v_j
}
$$

这非常关键。

因为：

$$
\boxed{
g_i
=
1-\alpha_{is}
}
$$

实际上变成了一个 head-level gate。

---

# 9. Sink 越大，Head 越接近关闭

如果：

$$
\alpha_{is}=0.99
$$

那么：

$$
g_i=0.01
$$

这个 Attention head 几乎不输出真实信息。

如果：

$$
\alpha_{is}=0.2
$$

那么：

$$
g_i=0.8
$$

它大量读取其他 token。

因此 Sink 的一个非常自然的解释是：

$$
\boxed{
\text{Softmax Attention 自发形成了一个隐式 output gate}
}
$$

---

# 10. 为什么这种 Gate 是有用的？

因为不是所有 head 都应该在所有 token、所有上下文下工作。

如果某个 head 没有找到自己负责的关系，却被迫：

$$
\sum_j\alpha_{ij}v_j
$$

那么它仍然会从某些 token 搬入信息。

这会导致：

$$
\boxed{
\text{unnecessary information mixing}
}
$$

而 Sink 可以提供一个安全路径：

$$
\boxed{
\text{找不到有意义的对象}
\rightarrow
\text{把 attention 给 NULL}
\rightarrow
\text{尽量不写东西}
}
$$

这可以减少 over-mixing。

---

# 11. 从最大熵视角理解 Sink

标准 Softmax 在概率单纯形上求：

$$
\alpha
=
softmax(s)
$$

候选集合只有：

$$
\{token_1,\ldots,token_n\}
$$

并没有：

$$
NULL
$$

如果我们显式添加：

$$
\{NULL,token_1,\ldots,token_n\}
$$

并定义：

$$
v_{NULL}=0
$$

那么：

$$
\alpha
=
softmax
\begin{bmatrix}
s_{NULL}\\
s_1\\
\vdots\\
s_n
\end{bmatrix}
$$

此时：

$$
\alpha_{NULL}
$$

天然表示：

> 当前不应该读取真实 token 的概率质量。

但标准 Transformer 没有这个状态。

所以：

$$
\boxed{
Attention Sink
}
$$

可以理解成模型训练过程中自己构造出来的：

$$
\boxed{
Implicit NULL State
}
$$

---

# 12. 为什么 Sink 经常出现在第一个 token？

在 causal Attention 中，位置 \(i\) 可以看到：

$$
1,\ldots,i
$$

第一个位置的特殊之处是：

$$
\boxed{
\text{它对所有后续 token 永远可见}
}
$$

如果模型需要一个所有 query 都能访问的 NULL address，第一个 token 是非常自然的候选。

尤其当它是：

$$
BOS
$$

之类的固定 token 时，它还有：

$$
\boxed{
\text{跨样本稳定}
}
$$

这一优势。

因此：

$$
\text{第一个 token}
$$

非常适合被训练成一个全局 anchor。

---

# 13. 为什么 RoPE 不会破坏 Sink？

RoPE 下：

$$
s_{i1}
=
x_i^T
W_Q^T
R_{1-i}
W_K
x_1
$$

随着 \(i\) 改变：

$$
R_{1-i}
$$

也在改变。

所以 Sink 并不是简单依赖一个固定 dot product。

更准确的理解是：

$$
W_Q,\quad W_K,\quad x_s
$$

和 RoPE 共同学习出一种结构，使：

$$
A_{1-i}x_s
$$

在大量相对位置下仍然容易与 query 形成较高 compatibility。

因此 Sink 不是“绕过 RoPE”，而是在 RoPE 定义的关系空间内部学出来的。

---

# 14. RMSNorm 和 Sink 有什么关系？

RMSNorm 主要去掉：

$$
\boxed{
\text{整体尺度}
}
$$

但不会去掉：

$$
\boxed{
\text{异常特殊的方向}
}
$$

假设：

$$
x_s
=
[M,\epsilon_2,\ldots,\epsilon_d]
$$

并且：

$$
M\gg|\epsilon_k|
$$

RMSNorm 后，这个向量不会变成普通方向。

相反，它可能变成一个非常稳定、非常突出的 normalized direction。

于是：

$$
W_KRMSNorm(x_s)
$$

很容易把这种特殊 representation 转换成稳定 sink key。

因此：

$$
\boxed{
RMSNorm 并不会消灭 Sink 所需要的特殊方向结构
}
$$

---

# 15. Massive Activation 和 Attention Sink 是一回事吗？

不是。

两者经常一起观察到，但概念不同。

Massive activation 描述的是：

$$
\boxed{
\text{representation 内部某些维度或方向出现极端幅值}
}
$$

Attention Sink 描述的是：

$$
\boxed{
\text{某些位置长期获得异常大的 attention probability}
}
$$

前者属于 activation geometry；

后者属于 attention routing。

它们可以互相促进，但不能简单等同。

---

# 16. Attention Sink 是一个严重问题吗？

不能简单回答：

$$
\text{是}
$$

或者：

$$
\text{不是}
$$

更准确的说法是：

$$
\boxed{
\text{它可能既是一种有效机制，也带来工程和解释上的问题。}
}
$$

---

# 17. 从模型功能上看，它不一定是 Bug

如果 Sink 承担：

$$
\boxed{
NULL address
}
$$

或者：

$$
\boxed{
head gate
}
$$

那么它已经成为模型计算的一部分。

此时直接删除 sink token，等于修改模型内部 circuit。

所以：

$$
\boxed{
\text{高 Sink 并不自动意味着模型出现故障}
}
$$

---

# 18. 但它会破坏 Attention Weight 的可解释性

如果：

$$
\alpha_{i1}=0.9
$$

不能简单解释成：

> 模型 90% 地依赖第一个 token。

更可能是：

> 这个 head 90% 地选择了 no-op。

因此：

$$
\boxed{
Attention Weight
\neq
Feature Contribution
}
$$

如果要分析因果贡献，需要同时看：

$$
QK
$$

和：

$$
OV
$$

甚至进一步分析 residual write。

---

# 19. 它还会影响 KV Cache

Streaming inference 中，如果简单认为：

> 最老 token 应该最没用。

然后把最前面的 KV 全部删除，就可能删除模型赖以实现 NULL/gating 的 sink state。

这也是为什么某些 streaming 方法会保留少量最初的 sink token，同时滑动其他 KV。

所以 Sink 不只是理论现象，也会影响推理系统设计。

---

# 20. Sink 也可能暴露一个架构缺口

从设计角度看，如果模型只是想表达：

$$
\boxed{
\text{这个 Attention head 当前不工作}
}
$$

那么更直接的架构应该提供：

$$
\boxed{
\text{显式 gate}
}
$$

或者：

$$
\boxed{
\text{显式 NULL key/value}
}
$$

而不是让模型浪费一个真实 token 来承担这个功能。

因此 Attention Sink 也可以理解成：

$$
\boxed{
\text{模型为了补偿 Softmax Attention 的结构限制，自行学出的 workaround}
}
$$

---

# 21. 为什么 Sigmoid Attention 理论上不那么需要 Sink？

Sigmoid Attention 不要求：

$$
\sum_j\alpha_{ij}=1
$$

因此完全可以出现：

$$
\alpha_{i1}
\approx
\alpha_{i2}
\approx
\cdots
\approx0
$$

也就是说：

$$
\boxed{
\text{head 可以直接选择不读取任何位置}
}
$$

所以 Softmax Attention 中产生 sink 的结构性动机，在这种架构里会弱很多。

当然，这不意味着 sigmoid 一定整体优于 Softmax。

Softmax 同时提供了：

* 竞争寻址；
* 稳定 normalization；
* scale control；
* 概率分布结构。

所以这是架构 trade-off，而不是简单的优劣关系。

---

# 22. 一个更直接的设计：显式 NULL token

可以人为定义：

$$
v_\varnothing=0
$$

并计算：

$$
\alpha_i
=
softmax
(
s_{i\varnothing},
s_{i1},
\ldots,
s_{in}
)
$$

那么：

$$
\alpha_{i\varnothing}
$$

直接表示：

> 当前 head 不希望读取真实 context 的程度。

Attention 输出：

$$
o_i
=
\sum_j\alpha_{ij}v_j
$$

自然可以写成：

$$
o_i
=
(1-\alpha_{i\varnothing})
\tilde o_i
$$

于是：

$$
\boxed{
1-\alpha_{i\varnothing}
}
$$

就是显式 gate。

从这个角度看，Attention Sink 做的事情并不神秘：

$$
\boxed{
\text{它只是隐式实现了一个原本可以显式设计的 NULL/gating mechanism。}
}
$$

---

# 23. 最后用匹配矩阵 \(A_\Delta\) 把整个机制统一起来

普通 Attention：

$$
\boxed{
s_{ij}
=
x_i^TA_\Delta x_j
}
$$

其中 \(A_\Delta\) 决定：

> 哪些 feature、哪些位置关系应该产生高 compatibility？

Softmax：

$$
\boxed{
\alpha_i=softmax(s_i)
}
$$

要求：

> 所有候选共享固定概率预算。

但模型有时需要：

$$
\boxed{
\text{不读取任何真实 token}
}
$$

于是训练可能让某个位置 \(s\) 满足：

$$
\boxed{
x_i^TA_{s-i}x_s
\text{ 对大量 query 很高}
}
$$

也就是：

$$
\boxed{
QK:
\text{让它成为 universal address}
}
$$

同时：

$$
\boxed{
W_OW_Vx_s\approx0
}
$$

即：

$$
\boxed{
OV:
\text{让它成为 harmless payload}
}
$$

最终得到：

$$
\boxed{
\text{Attention Sink}
=
\text{Strong Address}
+
\text{Weak Payload}
}
$$

它进一步等价于：

$$
\boxed{
\text{Implicit NULL Memory}
+
\text{Implicit Head Gate}
}
$$

---

# 结语

Attention Sink 最有意思的地方，不是“为什么第一个 token 会获得很多 Attention”。

真正值得理解的是：

$$
\boxed{
\text{为什么模型会主动需要一个可以吸收 Attention 的地方？}
}
$$

答案很可能来自 Softmax Attention 的结构：

$$
\boxed{
\sum_j\alpha_{ij}=1
}
$$

意味着：

> head 永远必须把全部概率质量交给某些 token。

但真实计算却经常需要：

$$
\boxed{
\text{nothing to retrieve}
}
$$

于是模型学习：

$$
\boxed{
\text{一个容易被寻址、但不会真正搬运太多信息的位置}
}
$$

这就是 Attention Sink 最值得关注的机制性解释。

它不是单纯的异常注意力。

更准确地说，它可能是 Transformer 在标准 Softmax Attention 限制下，自发形成的一种：

$$
\boxed{
\text{NULL state / gating circuit}
}
$$

而这也再次说明：

$$
\boxed{
\text{真正理解 Attention，不能只看 Attention Map。}
}
$$

必须同时理解：

$$
\boxed{
QK,\quad Softmax,\quad OV,\quad Residual Stream
}
$$

因为 Attention weight 只告诉我们：

> 信息准备从哪里流。

真正决定模型计算的，是：

> 什么 feature 被搬运、如何写回，以及后面的 circuit 如何继续使用它。
