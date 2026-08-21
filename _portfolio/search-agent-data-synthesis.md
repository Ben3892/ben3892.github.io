---
title: "搜索 Agent 多跳推理数据合成"
collection: portfolio
permalink: /portfolio/search-agent-data-synthesis/
published: true
order: 3
period: "2026.04 — 至今"
project_type: "SEARCH AGENT"
role: "数据构造 · 能力拆解"
stack:
  - Wikipedia
  - BM25
  - Embedding
  - Reranking
  - Entity Graph
excerpt: "结合离线检索、实体图与 LLM 合成多跳搜索轨迹，并围绕查询改写、证据定位等能力进行课程式训练。"
author_profile: true
share: false
comments: false
read_time: false
---

## 项目背景

多跳搜索任务要求模型持续形成查询、定位证据并根据新信息调整推理路径。高质量训练数据需要同时保证检索可达、证据可信和轨迹具有学习价值。

## 相关研究

本项目重点关注 Agent 能力应在哪个训练阶段注入，以及如何通过结构化采样控制搜索问题的难度与多样性。

- **[Scaling Agents via Continual Pre-training](https://arxiv.org/abs/2509.13310)** 将 Agent 轨迹引入持续预训练阶段，使模型在后训练之前先建立基础的工具使用与多步搜索能力。这一思路支持本项目在 Mid-training 阶段注入搜索轨迹，而不是仅依赖 SFT 学习完整的 Agent 行为。
- **[REDSearcher: A Scalable and Cost-Efficient Framework for Long-Horizon Search Agents](https://arxiv.org/abs/2602.14234)** 通过图拓扑结构和证据分散度联合控制任务难度，并将知识、规划和函数调用等原子能力前置到 Mid-training。这启发本项目使用实体关系图表示多跳证据链，并通过图上路径采样构造不同难度的问题。
- **[LiteResearcher: A Scalable Agentic RL Training Framework for Deep Research Agent](https://arxiv.org/abs/2604.17931)** 通过协同构建合成任务与本地网页语料，搭建模拟真实搜索动态的本地 Search/Browse 环境；同时将复杂检索任务概括为直接信息获取、聚合、枚举、交叉验证和统计五类原子搜索能力，再按任务难度组织课程式 RL 训练。这启发本项目在图结构之外，进一步从搜索能力类型与训练难度两个维度控制合成数据的多样性。

### Agentic CPT：目标不变，训练分布改变

Agentic CPT 仍然使用标准的自回归预测目标。令 \\(z_{1:T}\\) 表示长度为 \\(T\\) 的序列化 Agent 训练样本，\\(\theta\\) 表示模型参数，则：

$$
\mathcal{L}_{\mathrm{CPT}}(\theta)
=
-\frac{1}{T}\sum_{t=1}^{T}
\log p_{\theta}\!\left(z_t\mid z_{<t}\right)
$$

关键变化不在损失函数，而在训练分布：持续预训练的异构语料中进一步加入或强化问题分析、查询生成、工具调用、环境反馈与证据整合等 Agent 行为序列。因此，Agent 能力可以在 Mid-training 阶段通过数据分布注入，再由 SFT 或 RL 学习完整任务策略。这为本项目在 Mid-training 阶段注入 Agent 行为提供了训练范式上的动机；Query Reformulation、Evidence Grounding 与状态转移则是本项目进一步设计的能力拆分。

### REDSearcher：结构复杂度与证据分散度

只按路径长度控制跳数，并不能完整描述搜索问题的难度。令 \\(G=(V,E)\\) 表示任务的推理图，\\(\operatorname{TD}(G)\\) 表示其所有合法树分解的集合。每个树分解由分解树 \\(\mathcal{T}\\) 和变量袋（Bag）集合 \\(\{X_i\}_{i\in I}\\) 构成，其中 \\(I\\) 是分解树节点的索引集合。REDSearcher 使用树宽刻画推理图中约束的耦合程度：

$$
\operatorname{tw}(G)
=
\min_{(\mathcal{T},\{X_i\}_{i\in I})\in\operatorname{TD}(G)}
\left(\max_{i\in I}|X_i|-1\right).
$$

论文进一步使用下面的粗粒度代理解释结构复杂度：

$$
\mathcal{C}_{\mathrm{reasoning}}
\approx
O\!\left(Nd^{k+1}\right),
\qquad k=\operatorname{tw}(G).
$$

其中 \\(X_i\\) 是树分解中的变量集合，\\(N\\) 是推理步数，\\(d\\) 是每一步保留的候选分支数。第二式是论文用于解释难度变化的粗粒度代理，而不是 LLM 实际运行时间的严格复杂度定理。长链虽然跳数很多，但树宽仍可能很低；菱形、环形或多分支汇合结构则要求 Agent 同时维护多个假设并进行一致性验证。

另一方面，即使推理图很复杂，如果一篇文档包含全部证据，任务仍可能被一次检索绕过。REDSearcher 因此定义最小证据来源数：

$$
\mathcal{D}_{\mathrm{task}}
=
\min_{\mathcal{S}\subseteq\mathcal{W}}
|\mathcal{S}|
\quad
\text{s.t.}
\quad
\operatorname{Cover}(\mathcal{S},G)=\mathrm{True},
$$

其中 \\(\mathcal{W}\\) 是文档集合，\\(\mathcal{S}\\) 是足以覆盖推理图全部必要证据的最小文档子集。\\(\operatorname{tw}(G)\\) 控制“需要同时处理多少相互耦合的约束”，\\(\mathcal{D}_{\mathrm{task}}\\) 控制“这些约束分散在多少来源中”。二者联合使用，比单独增加跳数更能避免检索捷径。对本项目而言，当前的图上路径采样主要控制跳数；分支、汇合结构与证据来源数可作为后续更细粒度的难度控制方向。

### LiteResearcher：在能力边界附近采样

LiteResearcher 对每个问题生成 \\(K=8\\) 条 Rollout，并只保留既没有全部答对、也没有全部答错的样本。令 \\(\mathcal{Q}\\) 表示候选问题集合，\\(R(o_i,q)\in\{0,1\}\\) 表示第 \\(i\\) 条 Rollout 是否正确，论文中的筛选规则可以等价形式化为：

$$
\begin{aligned}
c(q)
&=
\sum_{i=1}^{K}R(o_i,q),\\
\mathcal{Q}_{\mathrm{keep}}
&=
\left\{
q\in\mathcal{Q}
\mid
1\leq c(q)\leq K-1
\right\}.
\end{aligned}
$$

当 \\(c(q)=K\\) 时，问题对当前模型过于简单；当 \\(c(q)=0\\) 时，问题可能超出当前能力边界、不可解或含有噪声。保留 \\(0<c(q)<K\\) 的样本，可让课程数据集中在“有时成功、有时失败”的有效学习区间。虽然 LiteResearcher 将这一规则用于课程式 RL，同样的思想也可以迁移到 Mid-training：分别估计模型在 Query Reformulation、Evidence Grounding 与状态转移任务上的成功率，再据此调整各类数据的难度和比例。

基于上述研究，本项目选择“实体图构建—图上路径采样—问题与轨迹合成”的数据路线，并进一步将完整搜索过程拆解为 Query Reformulation、Evidence Grounding 等原子能力，用于分难度组织训练数据与开展细粒度评测。

### 与本项目的联系：图上数据合成

令 \\(G=(V,E)\\) 表示由实体及其关系构成的图，\\(\pi=(v_0,\ldots,v_h)\\) 表示一条满足 \\((v_{i-1},v_i)\in E\\) 的多跳路径，\\(\mathcal{W}_{\pi}=f(G,\pi)\\) 表示由图与路径确定的文档及证据集合。项目中的两阶段数据合成过程可联合表示为：

$$
p(\pi,q,\tau\mid G)
=
p_{\mathrm{path}}(\pi\mid G)\,
p_{\phi}\!\left(q,\tau\mid\pi,\mathcal{W}_{\pi}\right).
$$

其中 \\(p_{\mathrm{path}}\\) 控制实体路径与跳数；若后续扩展为子图采样或加入拓扑拒绝采样，还可以进一步控制分支、汇合与树宽等结构属性。\\(p_{\phi}\\) 是用于数据合成的 LLM，\\(q\\) 是基于路径与证据生成的问题，\\(\tau\\) 是相应的搜索与推理轨迹。左侧因子主要决定路径结构与跳数，任务难度还取决于证据选择和来源分散度；右侧因子主要决定问题形式、查询过程与 Evidence Grounding 质量。该分解也说明质量控制必须同时覆盖“路径是否形成有效证据链”和“合成轨迹是否忠于证据”。

## 我的工作

- 基于 Wikipedia 构建离线检索系统，组合 BM25、Qwen3 Embedding 与Re-ranker模型。
- 从文档中抽取实体，通过页面链接与llm标注entity与entity的关系构建实体图，再使用 LLM 合成问题和搜索轨迹。
- 标注trajectory中不同entity以及hidden state的状态，将 Agent trajectory的过程为evidence grounding、query reformulation等原子能力，构建不同类型的原子能力数据，并采用课程式中训练组织不同难度数据。
  - hidden state标注：例如对于某对entity-entity的关系，会标注null、hypothesis、commit、reject等状态。
  - 原子任务：
    - 1. belief state classification：给模型context、question以及previous beliefs，让模型回答当前的entity到entity的关系是什么
    - 2. 提取状态转移：null/hypothesis -> commit，对于entity，只提供上下文，要求模型根据surface clues回答这个entity是什么
    - 3. 提取状态转移：hypothesis->commit/reject，提供context、entity的匿名描述，提问模型每个entity的状态是什么（verified、unverified、reject）
    - 4. next focus: 把context压缩成自然语言（已确认的entity、假设、未确认部分），让模型回答下一步的搜索candidates（自然语言描述）
    - 5. query formulation：把context压缩成自然语言（已确认的entity、假设、未确认部分），让模型回答下一步应该搜索什么query，使用recall评价生成的query
- 自建原子能力benchmark评估mid-training训练效果，并接入后训练sft使用browsecomp评估集评估在线search agent能力。

## 结果

形成了“检索构建—图上采样—轨迹合成—质量控制—能力评测”的完整多跳数据生产流程。
