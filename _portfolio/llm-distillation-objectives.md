---
title: "大模型蒸馏目标设计与规模化验证"
collection: portfolio
permalink: /portfolio/llm-distillation-objectives/
published: true
order: 1
period: "2025.08 — 2026.04"
project_type: "LLM TRAINING"
role: "训练目标设计 · 实验归因"
stack:
  - Knowledge Distillation
  - Forward KL
  - NTP
  - MoE
excerpt: "分析 Mid-training 退火阶段中 Forward KL 与 NTP 的领域能力权衡，并将领域条件监督策略从代理实验扩展到目标模型。"
author_profile: true
share: false
comments: false
read_time: false
---

## 项目背景

参与 100B 级 MoE 搜索大模型预训练与中训练，围绕模型裁剪后的能力恢复、领域自适应训练以及蒸馏监督信号选择开展实验。
- 在数据实验中发现Forward KL和NTP loss各自存在不同的优势区间，因此尝试在模型训练中把NTP和Forward KL的优势融合到一个模型里

## 相关研究

本项目从训练目标、蒸馏配置和数据选择三个维度梳理现有方法，并据此确定实验优先级。

### 训练目标

- **[MiniLLM: On-Policy Distillation of Large Language Models](https://arxiv.org/abs/2306.08543)** 使用 Reverse KL 进行生成式语言模型蒸馏：以 Student 的 On-policy 轨迹为核心，并通过 Teacher-mixed Sampling 改善采样稳定性，再由 Teacher 在同一轨迹上提供分布监督。这表明蒸馏目标与训练数据的采样分布需要联合设计。
- **[ABKD: Pursuing a Proper Allocation of the Probability Mass in Knowledge Distillation via α-β-Divergence](https://arxiv.org/abs/2505.04560)** 通过 α-β Divergence 在 Forward KL 与 Reverse KL 的概率质量分配特性之间进行连续调节，扩展了损失函数的设计空间，但也额外引入了需要搜索的超参数。
- **Jensen–Shannon Divergence（JSD）** 以 Teacher 和 Student 输出概率分布的混合分布为中介，以对称且有界的方式度量两者差异。它提供了 KL 方向之外的候选目标，但其性质本身并不能保证缓解 Teacher–Student Capacity Gap，仍需通过规模化实验验证。
- **[Sparse Logit Sampling: Accelerating Knowledge Distillation in LLMs](https://arxiv.org/abs/2503.16870)** 通过重要性采样构造 Teacher 分布的无偏梯度估计，在尽量保留蒸馏信号的同时，降低离线 Logits 的存储与训练成本。

#### 公式分析：KL 方向

令 \\(x\\) 表示某个 Token 之前的上下文，\\(\mathcal{V}\\) 表示词表，\\(p_T(v\mid x)\\) 和 \\(p_S(v\mid x)\\) 分别表示 Teacher 与 Student 对下一个 Token \\(v\\) 的概率分布。在固定上下文上，两种 Token-level KL 目标可写为：

$$
\mathcal{L}_{\mathrm{FKL}}(x)
= \sum_{v\in\mathcal{V}} p_T(v\mid x)
\log\frac{p_T(v\mid x)}{p_S(v\mid x)}
$$

$$
\mathcal{L}_{\mathrm{RKL}}(x)
= \sum_{v\in\mathcal{V}} p_S(v\mid x)
\log\frac{p_S(v\mid x)}{p_T(v\mid x)}
$$

两者的差异不只是书写顺序：Forward KL 在 Teacher 分布下取期望，更强调覆盖 Teacher 认为重要的概率质量；Reverse KL 在 Student 分布下取期望，更聚焦于 Student 实际访问的高概率区域。上式描述的是固定上下文上的分布差异；在 MiniLLM 的 On-policy 设定中，这些上下文主要来自 Student 轨迹，实际优化还采用 Teacher-mixed Sampling。这也说明了为什么需要同时考察损失方向、采样分布与不同数据域上的能力收益。

#### 公式分析：JSD 的对称中介

令 Teacher 与 Student 的混合分布为：

$$
m(v\mid x)
=
\frac{1}{2}
\left[p_T(v\mid x)+p_S(v\mid x)\right].
$$

JSD 定义为：

$$
\mathcal{L}_{\mathrm{JSD}}(x)
=
\frac{1}{2}D_{\mathrm{KL}}\!\left(p_T\parallel m\right)
+
\frac{1}{2}D_{\mathrm{KL}}\!\left(p_S\parallel m\right).
$$

JSD 对 Teacher 与 Student 是对称的；使用自然对数时，其目标值满足 \\(0\leq\mathcal{L}_{\mathrm{JSD}}\leq\log 2\\)，并且在两侧分布支持集不完全重合时仍保持有限。它因此适合作为 Forward KL 与 Reverse KL 之外对称且有界的候选对照，但“对称、有界”并不等于一定更易优化：当两个分布相距很远时，JSD 可能接近上界并减弱有效梯度。ABKD 则进一步使用 \\(\alpha\\) 与 \\(\beta\\) 连续控制概率质量更新的偏好；相较之下，本项目优先验证超参数更少、规模化成本更清晰的 Forward KL、NTP 及二者的组合方式，并将 JSD 保留为候选扩展。

这些工作共同扩展了蒸馏目标的设计空间。综合考虑训练稳定性、超参数数量和规模化成本后，本项目优先验证 Forward KL、NTP 及二者的组合方式。

### 蒸馏配置

- **[Pre-training Distillation for Large Language Models: A Design Space Exploration](https://arxiv.org/abs/2410.16215)** 系统考察了 Logits 处理、损失函数、Scaling Law 以及离线/在线 Logits 等设计维度，为本项目利用代理模型搜索 Top-k、Temperature 和 Loss Weight 等配置提供了方法参考。
- **[Distilled Pretraining: A Modern Lens of Data, In-Context Learning and Test-Time Scaling](https://arxiv.org/abs/2509.01649)** 分析了蒸馏对 Pass@k、Test-time Scaling 和 In-context Learning 的影响，并尝试在 Teacher 输出的低熵 Token 上关闭蒸馏损失、回退到 NTP 监督。该 Token Routing 方案直接构成了本项目细粒度路由实验的对照方向。

#### 公式分析：Top-k、Temperature 与 Loss Weight

令 \\(z_T(v\mid x)\\) 表示 Teacher Logit，Temperature 为 \\(\tau>0\\)，则处理后的 Teacher 分布为：

$$
p_T^{(\tau)}(v\mid x)
=
\frac{\exp\!\left(z_T(v\mid x)/\tau\right)}
{\sum_{u\in\mathcal{V}}\exp\!\left(z_T(u\mid x)/\tau\right)}.
$$

若 \\(S_k(x)\\) 表示 Teacher Logit 最大的 \\(k\\) 个 Token，其中 \\(1\leq k\leq|\mathcal{V}|\\)，Top-k 截断后的 Teacher 分布可写为：

$$
\widetilde p_T^{(\tau,k)}(v\mid x)
=
\frac{
\mathbf{1}[v\in S_k(x)]p_T^{(\tau)}(v\mid x)
}{
\sum_{u\in S_k(x)}p_T^{(\tau)}(u\mid x)
}.
$$

按照所引设计空间论文只处理 Teacher Logits 的约定，对应的混合目标可统一写成：

$$
\mathcal{L}_{\gamma,\tau,k}
=
(1-\gamma)\mathcal{L}_{\mathrm{NTP}}
+
\gamma D_{\mathrm{KL}}\!\left(
\widetilde p_T^{(\tau,k)}
\parallel p_S
\right).
$$

其中 \\(\gamma\in[0,1]\\)。三个参数作用在不同层面：\\(k\\) 决定保留的 Teacher Token 数量以及离线 Logits 的存储成本，并间接决定保留的概率质量；\\(\tau\\) 决定 Teacher 分布的平滑程度；\\(\gamma\\) 决定硬标签与软分布监督的相对权重。它们共同改变 Student 实际接收到的梯度，而不是彼此独立的数值旋钮。本项目针对 Top-k、Temperature 与 Loss Weight 的代理实验，本质上是在搜索上述目标的不同实例。

#### 公式分析：基于 Teacher Entropy 的 Token Routing

对位置 \\(t\\) 的上下文 \\(c_t=(x,y_{<t})\\)，Teacher Entropy 为：

$$
H_T(c_t)
=
-\sum_{v\in\mathcal{V}}
p_T(v\mid c_t)\log p_T(v\mid c_t).
$$

令 \\(h_q\\) 表示当前序列中最低 \\(q\%\\) Teacher Entropy 对应的分位点，其中 \\(q\in[0,100]\\)，并定义蒸馏开关：

$$
r_t
=
\mathbf{1}\!\left[H_T(c_t)>h_q\right].
$$

《Distilled Pretraining》中的路由可以抽象为：

$$
\mathcal{L}_{\mathrm{token\text{-}route}}
=
\frac{1}{n}\sum_{t=1}^{n}
\left[
(1-\gamma)\ell_{\mathrm{NTP},t}
+
\gamma r_t\ell_{\mathrm{FKL},t}
\right].
$$

其中 \\(\ell_{\mathrm{NTP},t}=-\log p_S(y_t\mid c_t)\\)，\\(\ell_{\mathrm{FKL},t}=D_{\mathrm{KL}}(p_T(\cdot\mid c_t)\parallel p_S(\cdot\mid c_t))\\)，\\(\gamma\in[0,1]\\)。这里 NTP 监督保留在所有位置，最低熵的一部分 Token 只关闭蒸馏项；论文实验使用了每条序列中最低熵的 15% Token。

相比之下，本项目还测试了更强的硬路由——低熵位置只用 NTP、高熵位置只用 Forward KL——但没有恢复 NTP 在 Math/Code 上的优势。在本项目的模型、数据和阈值设置下，该结果表明 Teacher Entropy 单独作为路由信号不足以恢复 NTP 在 Math/Code 上的优势；这促使我们进一步考察领域级目标选择。

### 数据选择

- **[MiniPLM: Knowledge Distillation for Pre-Training Language Models](https://arxiv.org/abs/2410.17215)** 使用 Teacher 与小型 Reference Model 对完整序列的概率差异进行 Difference Sampling：上采样 Teacher 更偏好、但 Reference Model 认为较难的数据，下采样常见易学模式，并剔除噪声样本。这表明除了损失函数，数据分布本身也可以作为蒸馏优化的控制轴。

#### 公式分析：Difference Sampling

对固定长度序列 \\(\mathbf{x}=(x_1,\ldots,x_n)\\)，令 \\(p_R\\) 表示小型 Reference Model 的分布。MiniPLM 使用 Teacher 与 Reference Model 的序列级对数似然差作为数据得分：

$$
\begin{aligned}
s(\mathbf{x})
&=
\log\frac{p_T(\mathbf{x})}{p_R(\mathbf{x})}\\
&=
\sum_{t=1}^{n}
\left[
\log p_T(x_t\mid x_{<t})
-
\log p_R(x_t\mid x_{<t})
\right].
\end{aligned}
$$

排除用于训练 Reference Model 的语料 \\(\mathcal{D}_{\mathrm{ref}}\\) 后，令 \\(\operatorname{rank}_s(\mathbf{x})\\) 表示样本在候选集 \\(\mathcal{D}\setminus\mathcal{D}_{\mathrm{ref}}\\) 中按 \\(s\\) 从高到低的排序位置，\\(K_{\mathrm{data}}\\) 表示需要选取的样本数。Difference Sampling 得到：

$$
\mathcal{D}'
=
\left\{
\mathbf{x}\in\mathcal{D}\setminus\mathcal{D}_{\mathrm{ref}}
\;\middle|\;
\operatorname{rank}_s(\mathbf{x})\leq K_{\mathrm{data}}
\right\}.
$$

按照 MiniPLM 的建模假设，较高的 \\(s(\mathbf{x})\\) 表示 Teacher 对该序列的建模明显优于小型 Reference Model，因此样本更可能包含“小模型尚未掌握、但大模型能够解释”的模式。Difference Sampling 与本项目的领域级路由作用在两个互补的控制轴上：前者决定“训练什么数据”，后者决定“在给定数据上使用什么监督”。如果候选序列长度不同，则还需要显式说明是否使用长度归一化得分，避免序列长度主导排序。

基于上述调研，本项目依次验证了蒸馏超参数、Token 级路由信号与数据域级目标选择，并重点考察不同方案能否从代理实验稳定扩展到目标模型。

### 与本项目的联系：领域级目标路由

令 \\(d(x)\\) 表示样本所属领域，\\(y=(y_1,\ldots,y_n)\\) 表示目标 Token 序列，\\(c_t=(x,y_{<t})\\) 表示位置 \\(t\\) 的上下文。为与序列级 NTP 保持一致，Forward KL 也按有效 Token 数取平均：

$$
\mathcal{L}_{\mathrm{FKL}}(x,y)
=
\frac{1}{n}\sum_{t=1}^{n}
\sum_{v\in\mathcal{V}}
p_T(v\mid c_t)
\log\frac{p_T(v\mid c_t)}{p_S(v\mid c_t)}.
$$

NTP 损失定义为：

$$
\mathcal{L}_{\mathrm{NTP}}(x,y)
= -\frac{1}{n}\sum_{t=1}^{n}
\log p_S\!\left(y_t\mid x,y_{<t}\right)
$$

在 Forward KL 与 NTP 均按有效 Token 数取平均的约定下，领域级训练目标可统一表示为：

$$
\mathcal{L}(x,y)
= \lambda_{d(x)}\mathcal{L}_{\mathrm{FKL}}(x,y)
+ \left(1-\lambda_{d(x)}\right)\mathcal{L}_{\mathrm{NTP}}(x,y),
\qquad \lambda_d\in\{0,1\}
$$

其中 \\(\lambda_d\\) 是领域 \\(d\\) 的固定目标开关，而不是由模型动态学习的路由权重。在当前实验中，General/STEM 使用 \\(\lambda_d=1\\)，Math/Code 使用 \\(\lambda_d=0\\)。该形式化直接表达了“按数据域路由监督目标”的核心思路。

## 我的工作

- 搭建离线 Forward KL 蒸馏训练流程，并在 T 级预训练数据上进行模型恢复训练。
- 对不同领域、Top-k、Temperature、loss比例开展系统消融，定位 Forward KL 与 NTP 在不同能力维度上的收益差异。
- 在领域级硬路由实验中，General/STEM 仅使用 Forward KL，Math/Code 仅使用 NTP，使单个模型同时保留两类目标各自占优的能力区间。
- 尝试使用 Teacher Entropy 和 Teacher Loss 等 Token 级信号选择训练目标。在当前实验设置下，细粒度路由与纯 Forward KL 基本持平，未能恢复 NTP 在 Math/Code 上的优势，因此转向使用数据领域这一更粗粒度且稳定的路由信号。
- 将问题进一步推广为“如何为每个数据领域选择收敛更快的蒸馏配置”，并以固定 Token 预算下的 Validation Loss 作为优化指标。
  - 采用类似 RegMix 的代理建模方法，将 Temperature、Top-k、LM Loss Weight 和 Data Field 各取 4 档组合，共 \\(4^4=256\\) 组实验。使用 1B Proxy 模型生成训练结果，再通过多层感知机、LightGBM 与 XGBoost 拟合超参数到 Validation Loss 的映射，并以 MSE 作为预测损失。代理实验发现，Math/Code 采用 Top-k = 4、Temperature = 1 的蒸馏配置优于领域级硬路由。

## 结果

- V1 版本模型裁剪后，Base 模型在代表性评测上保留 Teacher 约 **92%** 的能力，其中数学为 **85%**、代码为 **88%**。领域级硬路由补充了 Student 在数学和代码上的短板，使其提升到与其他领域持平或略高。
- 领域级硬路由在代理实验和目标模型上均优于单独使用 KL 或 NTP：General Benchmark 优于纯 NTP，Math/Code Benchmark 优于纯 Forward KL。
- 后续 1B Proxy 超参数搜索得到的 Top-k = 4、Temperature = 1 配置在 Math/Code 上进一步优于硬路由；其向目标模型的迁移效果仍需单独验证。

## 思考
- 领域条件目标也可以从不同训练目标学习速度不一致的角度理解，后续可结合 GradNorm 等工作继续分析。

- Teacher Logits 可以视为比 One-hot Label 更密集的训练信号。

> 公开页面仅展示可对外说明的方法与结果，具体模型和内部实验配置已省略。
