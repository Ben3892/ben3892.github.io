---
title: "搜索 Agent 多跳推理数据合成"
collection: portfolio
permalink: /portfolio/search-agent-data-synthesis/
published: false
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


## 我的工作

- 基于 Wikipedia 构建离线检索系统，组合 BM25、Qwen3 Embedding 与重排模型。
- 从文档中抽取实体，通过页面链接与随机游走构建实体图，再使用 LLM 合成问题和搜索轨迹。
- 将 Agent 能力拆分为查询改写、证据定位等子能力，通过召回率控制样本质量，并采用课程式中训练组织不同难度数据。
- 使用 BrowserComp 等评测分析能力变化，将结果反馈到数据生成策略。

## 结果

形成了“检索构建—图上采样—轨迹合成—质量控制—能力评测”的完整多跳数据生产流程。
