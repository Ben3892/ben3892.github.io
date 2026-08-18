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

基于上述研究，本项目选择“实体图构建—图上路径采样—问题与轨迹合成”的数据路线，并进一步将完整搜索过程拆解为 Query Reformulation、Evidence Grounding 等原子能力，用于分难度组织训练数据与开展细粒度评测。

### 与本项目的联系：图上数据合成

令 \\(G=(V,E)\\) 表示由实体及其关系构成的图，\\(\mathcal{D}_{\pi}\\) 表示路径中实体对应的文档与证据，则项目中的数据合成过程可概念化为：

$$
\pi \sim p_{\mathrm{RW}}\!\left(\,\cdot\mid G\,\right)
$$

$$
(q,\tau)
\sim p_{\phi}\!\left(\,\cdot\mid\pi,\mathcal{D}_{\pi}\,\right)
$$

其中 \\(p_{\mathrm{RW}}\\) 是图上的路径采样分布，\\(\pi\\) 是采样得到的多跳实体路径，\\(p_{\phi}\\) 是用于数据合成的 LLM，\\(q\\) 是基于路径与证据合成的问题，\\(\tau\\) 是相应的搜索与推理轨迹。该形式化是对工程流程的概念抽象，将“图上采样”与“基于证据的 LLM 轨迹合成”两个阶段清晰地分离开。

## 我的工作

- 基于 Wikipedia 构建离线检索系统，组合 BM25、Qwen3 Embedding 与Re-ranker模型。
- 从文档中抽取实体，通过页面链接与随机游走构建实体图，再使用 LLM 合成问题和搜索轨迹。
- 标注trajectory中不同entity以及hidden state的状态，将 Agent trajectory的过程为evidence grounding、query reformulation等原子能力，构建不同类型的原子能力数据，并采用课程式中训练组织不同难度数据。
- 自建原子能力benchmark评估mid-training训练效果，并接入后训练sft使用browsecomp评估集评估在线search agent能力。

## 结果

形成了“检索构建—图上采样—轨迹合成—质量控制—能力评测”的完整多跳数据生产流程。
