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
excerpt: "分析mid-train的退火阶段中 Forward KL 与 NTP 的领域能力权衡，并将动态监督策略从代理实验扩展到目标模型。"
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

- **[MiniLLM: Knowledge Distillation of Large Language Models](https://arxiv.org/abs/2306.08543)** 使用 Reverse KL 进行生成式语言模型蒸馏：由 Student 生成轨迹，再由 Teacher 在同一轨迹上提供分布监督。这表明蒸馏目标与训练数据的采样分布需要联合设计。
- **[ABKD: Pursuing a Proper Allocation of the Probability Mass in Knowledge Distillation via α-β-Divergence](https://arxiv.org/abs/2505.04560)** 通过 α-β Divergence 在 Forward KL 与 Reverse KL 的概率质量分配特性之间进行连续调节，扩展了损失函数的设计空间，但也额外引入了需要搜索的超参数。
- **Jensen–Shannon Divergence（JSD）** 以 Teacher 和 Student 输出概率分布的混合分布为中介，以更对称的方式约束两者差异，为缓解 Teacher–Student Capacity Gap 提供了另一种目标设计思路。
- **[Sparse Logit Sampling: Accelerating Knowledge Distillation in LLMs](https://arxiv.org/abs/2503.16870)** 通过重要性采样构造 Teacher 分布的无偏梯度估计，在尽量保留蒸馏信号的同时，降低离线 Logits 的存储与训练成本。

#### 公式分析：KL 方向

令 \(x\) 表示某个 Token 之前的上下文，\(\mathcal{V}\) 表示词表，\(p_T(v\mid x)\) 和 \(p_S(v\mid x)\) 分别表示 Teacher 与 Student 对下一个 Token \(v\) 的概率分布。在固定上下文上，两种 Token-level KL 目标可写为：

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

两者的差异不只是书写顺序：Forward KL 在 Teacher 分布下取期望，更强调覆盖 Teacher 认为重要的概率质量；Reverse KL 在 Student 分布下取期望，更聚焦于 Student 实际访问的高概率区域。上式描述的是固定上下文上的分布差异；在 MiniLLM 的 On-policy 设定中，这些上下文来自 Student 自身生成的轨迹。这也说明了为什么需要同时考察损失方向、采样分布与不同数据域上的能力收益。

这些工作共同扩展了蒸馏目标的设计空间。综合考虑训练稳定性、超参数数量和规模化成本后，本项目优先验证 Forward KL、NTP 及二者的组合方式。

### 蒸馏配置

- **[Pre-training Distillation for Large Language Models: A Design Space Exploration](https://arxiv.org/abs/2410.16215)** 系统考察了 Logits 处理、损失函数、Scaling Law 以及离线/在线 Logits 等设计维度，为本项目利用代理模型搜索 Top-k、Temperature 和 Loss Weight 等配置提供了方法参考。
- **[Distilled Pretraining: A Modern Lens of Data, In-Context Learning and Test-Time Scaling](https://arxiv.org/abs/2509.01649)** 分析了蒸馏对 Pass@k、Test-time Scaling 和 In-context Learning 的影响，并尝试在 Teacher 输出的低熵 Token 上关闭蒸馏损失、回退到 NTP 监督。该 Token Routing 方案直接构成了本项目细粒度路由实验的对照方向。

### 数据选择

- **[MiniPLM: Knowledge Distillation for Pre-Training Language Models](https://arxiv.org/abs/2410.17215)** 使用 Teacher 与小型 Reference Model 对完整序列的概率差异进行 Difference Sampling：上采样 Teacher 更偏好、但 Reference Model 认为较难的数据，下采样常见易学模式，并剔除噪声样本。这表明除了损失函数，数据分布本身也可以作为蒸馏优化的控制轴。

基于上述调研，本项目依次验证了蒸馏超参数、Token 级路由信号与数据域级目标选择，并重点考察不同方案能否从代理实验稳定扩展到目标模型。

### 与本项目的联系：领域级目标路由

令 \(d(x)\) 表示样本所属领域，\(y=(y_1,\ldots,y_n)\) 表示目标 Token 序列。NTP 损失定义为：

$$
\mathcal{L}_{\mathrm{NTP}}(x,y)
= -\frac{1}{n}\sum_{t=1}^{n}
\log p_S\!\left(y_t\mid x,y_{<t}\right)
$$

在 Forward KL 与 NTP 均按有效 Token 数取平均的约定下，领域级训练目标可统一表示为：

$$
\mathcal{L}(x,y)
= \lambda_{d(x)}\mathcal{L}_{\mathrm{FKL}}(x)
+ \left(1-\lambda_{d(x)}\right)\mathcal{L}_{\mathrm{NTP}}(x,y),
\qquad \lambda_d\in\{0,1\}
$$

其中 \(\lambda_d\) 是领域 \(d\) 的固定目标开关，而不是由模型动态学习的路由权重。在当前实验中，General/STEM 使用 \(\lambda_d=1\)，Math/Code 使用 \(\lambda_d=0\)。该形式化直接表达了“按数据域路由监督目标”的核心思路。

## 我的工作

- 搭建离线 Forward KL 蒸馏训练流程，并在 T 级预训练数据上进行模型恢复训练。
- 对不同领域、Top-k、Temperature、loss比例开展系统消融，定位 Forward KL 与 NTP 在不同能力维度上的收益差异。
- 发现Forward KL和NTP loss各自存在不同的优势区间，因此探索在不同数据上使用不同的loss训练模型是否可以同时获得各自的优势区间，最后结果是在general、stem上只使用Forward-KL loss；在math、code上只使用NTP loss能同时获得Forward KL和NTP loss的优势
- 失败的一些尝试：希望能找到一个sign来知道数据应该选择使用NTP loss还是使用Forward KL loss，使用meta提出的根据teacher entropy来routing loss，低于teacher entropy使用NTP loss，高于teacher entropy使用Forward KL loss发现和纯Forward KL loss持平。使用teacher loss作为routing同样不能获得NTP loss的优势部分。因此，可能不能使用细粒度的sign作为routing标志，更合适的方式是使用粗粒度的sign（例如数据所属领域）
- 更广义的问题定义：如果说不同的数据领域最好使用不同的loss，那么上述例子只是一个特例，更一般的问题描述是：对于每一个数据领域，如何找到最优超参来让模型收敛更快？定义收敛更快的指标是validataion loss。
  - 做法：定义一个总的validation loss，蒸馏超参数topk、temperature、lm loss weight。grid search出不同数据领域不同组数的实验组（temperature*3, top-k * 3, lm loss weight *3, data field * 4），一共4*3个输入参数，108个实验组，使用1B 模型进行实验并预测出固定token下的validation loss，进行拟合得到最优解（linear regression、random forest、xgboost）。发现math/code实际上可以设置蒸馏超参为topk=4，temperature=1，优于上述结果。

## 结果

- v1版本模型裁剪后 Base 模型在代表性评测上保留 Teacher 约 **92%** 的能力（benchmark分数），数学为85%、代码为88%。动态目标策略补充了studnet在数学和代码的短板。
- 动态目标策略在代理实验和目标模型上均优于单独使用 KL 或 NTP 的方案。（具体形式：general benchmark上优于NTP方案、math/code benchmark上优于Forward KL方案）

> 公开页面仅展示可对外说明的方法与结果，具体模型和内部实验配置已省略。
