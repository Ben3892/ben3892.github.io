# 理解 Attention：从 \(QK^T\)、RoPE、Softmax 到 FFN 与 RMSNorm

很多 Attention 的介绍会从公式直接开始：

$$
Attention(Q,K,V)
=
softmax\left(\frac{QK^T}{\sqrt{d_k}}\right)V
$$

公式并不复杂，但真正难的是理解：

* 为什么要有 \(Q\) 和 \(K\)；
* \(QK^T\) 到底在计算什么；
* 为什么要加入位置；
* 为什么一定要经过 Softmax；
* \(V\) 到底承担什么角色；
* FFN 和 Attention 是怎样分工的；
* RMSNorm 又为什么放在 Attention 和 FFN 前面。

如果这些模块被分别理解，很容易觉得 Transformer 是很多技巧拼起来的。

但如果从一个统一的线性代数视角来看，它们其实构成了一条很清楚的计算链：

$$
\boxed{
\text{关系检测}
\rightarrow
\text{信息寻址}
\rightarrow
\text{信息读取}
\rightarrow
\text{特征计算}
\rightarrow
\text{状态更新}
}
$$

---

## 1. \(QK^T\) 不只是“相似度”

假设第 \(i\) 个 token 的表示为

$$
x_i
$$

定义：

$$
q_i=W_Qx_i,\qquad
k_i=W_Kx_i
$$

那么两个 token 之间的 attention score 为：

$$
s_{ij}
=
q_i^Tk_j
$$

代入：

$$
s_{ij}
=
x_i^TW_Q^TW_Kx_j
$$

定义：

$$
A=W_Q^TW_K
$$

于是：

$$
\boxed{
s_{ij}=x_i^TAx_j
}
$$

严格来说，这不是二次型，而是双线性型。

真正重要的是 \(A\)。

如果只是普通内积：

$$
x_i^Tx_j
$$

我们是在问：

> \(x_i\) 和 \(x_j\) 在原始 embedding 空间中是否相似？

而：

$$
x_i^TAx_j
$$

是在问：

> 按照模型学出的关系 \(A\)，\(x_i\) 与 \(x_j\) 是否匹配？

所以 Attention 更准确的概念不是 similarity，而是：

$$
\boxed{
compatibility
}
$$

即“关系兼容度”。

---

## 2. 为什么 \(Q\) 和 \(K\) 要分开

如果：

$$
W_Q=W_K
$$

那么 Attention 更接近一种对称的相似度计算。

但真实模型中：

$$
W_Q\neq W_K
$$

所以通常：

$$
A\neq A^T
$$

因此：

$$
s_{ij}\neq s_{ji}
$$

也就是说：

> token \(i\) 需要 token \(j\)

和

> token \(j\) 需要 token \(i\)

是两件不同的事情。

这就是为什么 Attention 本质上更像“关系查询”，而不是简单地寻找相似 token。

可以把：

$$
q_i
$$

理解成：

> 当前 token 在问什么？

而：

$$
k_j
$$

表示：

> token \(j\) 可以被什么问题找到？

于是：

$$
q_i^Tk_j
$$

就是一次 query-key matching。

---

# 3. 加入相对位置之后发生了什么？

以 RoPE 为例。

RoPE 对 \(q,k\) 根据位置进行旋转：

$$
q_i'=R_iq_i
$$

$$
k_j'=R_jk_j
$$

那么：

$$
s_{ij}
=
(q_i')^Tk_j'
$$

展开：

$$
s_{ij}
=
x_i^TW_Q^TR_i^TR_jW_Kx_j
$$

由于：

$$
R_i^TR_j
=
R_{j-i}
$$

于是：

$$
s_{ij}
=
x_i^TW_Q^TR_{j-i}W_Kx_j
$$

定义：

$$
A_\Delta
=
W_Q^TR_\Delta W_K
$$

其中：

$$
\Delta=j-i
$$

最终得到：

$$
\boxed{
s_{ij}
=
x_i^TA_{j-i}x_j
}
$$

这个式子非常重要。

没有相对位置时：

$$
x_i^TAx_j
$$

意味着：

> 无论两个 token 相距多远，都使用同一个关系算子。

加入 RoPE 后：

$$
x_i^TA_\Delta x_j
$$

意味着：

> 不同相对距离使用不同的关系算子。

因此可以理解为：

$$
\boxed{
\text{RoPE 把固定关系 }A
\text{ 变成位置条件化关系 }A_\Delta
}
$$

位置不是额外附加在关系旁边，而是直接进入了关系本身。

---

# 4. Multi-Head Attention 是什么？

每个 head 都有自己的：

$$
W_Q^{(h)},\quad
W_K^{(h)},\quad
W_V^{(h)}
$$

于是每个 head 都有：

$$
A_\Delta^{(h)}
=
(W_Q^{(h)})^TR_\Delta W_K^{(h)}
$$

因此 Multi-Head Attention 可以理解成：

$$
\boxed{
\text{同时使用多套不同的关系规则观察整个序列}
}
$$

不同 head 可以学习不同的关系：

* 局部邻近；
* 长距离依赖；
* 指代关系；
* 语法结构；
* 复制模式；
* 格式结构。

它们不一定能简单对应某一种人类语言学概念，但数学上，每个 head 确实拥有不同的关系算子。

---

# 5. 为什么 \(QK^T\) 后面必须接 Softmax？

到现在为止：

$$
s_{ij}=q_i^Tk_j
$$

只是一个任意实数：

$$
-10,\quad2,\quad7,\quad100
$$

真正的问题是：

> 这些关系分数怎样转化成“我应该从哪些 token 获取多少信息”？

这就是 Softmax 的作用。

$$
\alpha_{ij}
=
\frac{
\exp(s_{ij})
}{
\sum_k\exp(s_{ik})
}
$$

它使：

$$
\alpha_{ij}>0
$$

并且：

$$
\sum_j\alpha_{ij}=1
$$

所以：

$$
\boxed{
\alpha_{ij}
}
$$

可以理解成 token \(i\) 分配给 token \(j\) 的“读取预算”。

---

# 6. Softmax 的本质是竞争式寻址

假设：

$$
s=[2,3,8]
$$

Softmax 会得到一个高度集中的分布。

它表达的不是：

> 每个 token 独立判断自己是否重要。

而是：

> 所有候选 token 共享同一个总预算，因此彼此竞争。

这和 sigmoid 很不一样。

Sigmoid 可以同时给很多 token：

$$
[0.9,0.9,0.9]
$$

而 Softmax 强制：

$$
\sum_j\alpha_j=1
$$

因此它天然是一种：

$$
\boxed{
competitive routing
}
$$

---

# 7. Softmax 还是一个可微的 argmax

定义：

$$
\alpha_j
=
softmax(s_j/T)
$$

当：

$$
T\rightarrow0
$$

有：

$$
\alpha
\rightarrow
onehot(\arg\max s)
$$

因此：

$$
\boxed{
Softmax
\approx
\text{smooth argmax}
}
$$

这使 Attention 变成一种可微分检索。

传统检索：

$$
j^*
=
\arg\max_j s_j
$$

然后：

$$
return\ v_{j^*}
$$

Attention 则是：

$$
\boxed{
o_i
=
\sum_j\alpha_{ij}v_j
}
$$

也就是 soft retrieval。

---

# 8. Softmax 的最大熵解释

Softmax 更深的意义来自下面这个优化问题。

假设我们希望得到一个概率分布：

$$
p\in\Delta
$$

并且希望它尽量选择高 score 的位置：

$$
p^Ts
$$

但又不希望直接变成 hard argmax。

于是加入 entropy：

$$
H(p)
=
-\sum_jp_j\log p_j
$$

求解：

$$
\boxed{
\max_{p\in\Delta}
\left[
p^Ts
+
TH(p)
\right]
}
$$

这个优化问题的精确解是：

$$
\boxed{
p_j
=
\frac{e^{s_j/T}}
{\sum_ke^{s_k/T}}
}
$$

也就是 Softmax。

因此 Softmax 可以理解成：

$$
\boxed{
\text{高匹配收益}
+
\text{保持不确定性}
}
$$

之间的最优折中。

Temperature \(T\) 控制检索的尖锐程度。

---

# 9. Softmax 与信息几何

定义：

$$
\psi(s)
=
\log\sum_j e^{s_j}
$$

即 log-sum-exp。

有：

$$
\boxed{
\nabla\psi(s)
=
softmax(s)
}
$$

而它的 convex conjugate 是：

$$
\psi^*(p)
=
\sum_jp_j\log p_j
=
-H(p)
$$

负熵进一步产生 KL divergence。

因此：

$$
\boxed{
Softmax,\quad
LogSumExp,\quad
Entropy,\quad
KL
}
$$

本质上属于同一套凸对偶与信息几何结构。

这意味着 Softmax 并不是一个随意选择的归一化函数。

它把 score / energy 空间自然映射到了 probability simplex。

---

# 10. 那么 \(V\) 是什么？

现在才真正进入 Attention 的“信息读取”。

$$
v_j
=
W_Vx_j
$$

然后：

$$
o_i
=
\sum_j
\alpha_{ij}v_j
$$

所以：

$$
QK
$$

决定：

> 从哪里读？

Softmax 决定：

> 读多少？

而：

$$
V
$$

决定：

> 实际读取什么？

因此：

$$
\boxed{
QK
=
routing
}
$$

$$
\boxed{
V
=
payload
}
$$

这是理解 Attention 很重要的区分。

---

# 11. Attention 可以看成动态 memory retrieval

把每个 token 看成 memory slot：

$$
(k_j,v_j)
$$

当前 token 产生：

$$
q_i
$$

然后通过：

$$
q_i^Tk_j
$$

寻找最匹配的 memory slot。

因此 Attention 可以被理解成：

$$
\boxed{
\text{soft content-addressable memory}
}
$$

它读取的是当前 context 里动态存在的信息。

---

# 12. 为什么还需要 FFN？

Attention 的核心能力是：

$$
\boxed{
\text{跨 token 搬运信息}
}
$$

但搬过来的信息仍然需要处理。

FFN：

$$
FFN(x)
=
W_2\sigma(W_1x)
$$

如果把：

$$
W_1=
\begin{bmatrix}
k_1^T\\
\vdots\\
k_m^T
\end{bmatrix}
$$

并把：

$$
W_2=
\begin{bmatrix}
v_1&\cdots&v_m
\end{bmatrix}
$$

则：

$$
\boxed{
FFN(x)
=
\sum_r
\sigma(k_r^Tx)v_r
}
$$

这又是一种：

$$
\boxed{
\text{pattern match}
\rightarrow
\text{feature write}
}
$$

因此可以粗略地区分：

$$
\boxed{
Attention
=
Context Retrieval
}
$$

$$
\boxed{
FFN
=
Parametric Computation
}
$$

Attention 从上下文中获取信息；

FFN 根据当前表示激活参数中已经学到的特征变换。

---

# 13. Attention 和 FFN 的最重要分工

可以压缩成一句话：

$$
\boxed{
\text{Attention = Communication}
}
$$

$$
\boxed{
\text{FFN = Computation}
}
$$

Attention 在不同 token 之间搬运 feature。

FFN 在单个 token 内部对已有 feature 做非线性变换。

于是一个 Transformer block 本质上就是：

$$
\boxed{
\text{通信}
\rightarrow
\text{计算}
}
$$

---

# 14. Residual Stream 是整个 Transformer 的共享状态

现代 Transformer 通常使用：

$$
x'
=
x+\Delta_{\text{attn}}
$$

再：

$$
x''
=
x'+\Delta_{\text{ffn}}
$$

因此 residual stream 更适合被理解为：

$$
\boxed{
\text{shared computational workspace}
}
$$

Attention 把新的信息写进去；

FFN 把新的 feature 写进去；

后续层继续读取这些中间状态。

---

# 15. Feature 不一定对应单个 neuron

如果 residual dimension 只有：

$$
d
$$

但模型需要表达的 feature 数量远大于 \(d\)，那么这些 feature 不可能一一对应 neuron。

更合理的是：

$$
\boxed{
\text{feature 是表示空间中的方向}
}
$$

假设：

$$
x=Fz
$$

其中：

* \(z\)：抽象 feature activation；
* \(F\)：feature dictionary。

那么：

$$
x_i^TA_\Delta x_j
=
z_i^T
F^TA_\Delta F
z_j
$$

定义：

$$
M_\Delta
=
F^TA_\Delta F
$$

于是：

$$
\boxed{
s_{ij}
=
z_i^TM_\Delta z_j
}
$$

因此 Attention 可以进一步理解成：

$$
\boxed{
\text{feature-feature relation detection}
}
$$

---

# 16. QK circuit 与 OV circuit

一个 Attention head 可以拆成：

$$
\boxed{
QK:
\text{何时、从哪里读}
}
$$

以及：

$$
\boxed{
OV:
\text{读到之后搬什么、写成什么}
}
$$

所以 Attention 不只是“选一个 token”。

它真正做的是：

$$
\boxed{
\text{条件化的 feature transport}
}
$$

---

# 17. RMSNorm 为什么存在？

对 residual vector：

$$
x\in\mathbb R^d
$$

RMSNorm：

$$
RMSNorm(x)
=
\gamma\odot
\frac{x}{
\sqrt{
\frac1d\sum_kx_k^2+\epsilon
}
}
$$

忽略 \(\gamma,\epsilon\)：

$$
\hat x
=
\sqrt d
\frac{x}{\|x\|_2}
$$

所以它主要去掉：

$$
\boxed{
\text{representation 的整体尺度}
}
$$

而保留：

$$
\boxed{
\text{方向 / feature composition}
}
$$

---

# 18. 为什么 Attention 特别需要 Norm？

因为：

$$
s_{ij}
=
x_i^TA_\Delta x_j
$$

对 \(x\) 的尺度非常敏感。

而 Softmax 又对 score scale 极其敏感。

因此如果 residual norm 漂移，Attention 的 effective temperature 也会不断漂移。

RMSNorm 后：

$$
\boxed{
s_{ij}
=
\hat x_i^TA_\Delta\hat x_j
}
$$

Attention 就更多依赖 representation 的方向关系。

---

# 19. 为什么现代模型通常用 Pre-Norm？

典型结构：

$$
u
=
RMSNorm(x)
$$

$$
y
=
x+Attention(u)
$$

然后：

$$
v
=
RMSNorm(y)
$$

$$
z
=
y+FFN(v)
$$

这里可以这样理解：

$$
\boxed{
Residual Stream
=
Storage
}
$$

而：

$$
\boxed{
RMSNorm
=
Normalized Read Interface
}
$$

也就是说：

> 状态本身不反复 normalize，只在计算模块读取它之前进行尺度标准化。

这样 residual path 始终保留近似 identity highway。

---

# 20. 最后把整个 Attention Block 串起来

现在一层 Transformer 可以写成：

$$
\boxed{
x
\xrightarrow{RMSNorm}
\hat x
}
$$

先稳定读取状态。

然后：

$$
\boxed{
\hat x_i^TA_\Delta^{(h)}\hat x_j
}
$$

检测 feature relation。

然后：

$$
\boxed{
Softmax
}
$$

把 relation score 转换成竞争式概率寻址。

之后：

$$
\boxed{
\sum_j\alpha_{ij}V\hat x_j
}
$$

读取上下文信息。

然后：

$$
\boxed{
Residual Write
}
$$

写回 shared state。

再：

$$
\boxed{
RMSNorm
\rightarrow
FFN
\rightarrow
Residual Write
}
$$

进行 token 内部计算。

因此一个 Transformer block 可以高度概括为：

$$
\boxed{
\text{Normalize Read}
\rightarrow
\text{Detect Relation}
\rightarrow
\text{Route Information}
\rightarrow
\text{Move Features}
\rightarrow
\text{Compute Features}
\rightarrow
\text{Write State}
}
$$

这套视角非常重要，因为理解 Attention Sink 的关键，并不是先记住“某些 token 会吸收大量 attention”。

真正的问题是：

> 当 Softmax 强制一个 head 必须把全部 attention probability 分配出去，而这个 head 有时实际上不想读取任何 token 时，会发生什么？

这正是下一篇文章要讨论的问题。