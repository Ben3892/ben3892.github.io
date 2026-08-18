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
主要是关于query合成的研究
- scaling agent via mid-training（qwen团队）
  - 
- red searcher（xhs团队）
  - 通过entity之间提取evidence的方式来构建entity到entity之间的关系，通过抽取子图的方式控制query难度。
- lite searcher
  - 抽取不同结构的子图控制query的多样性。

## 我的工作

- 基于 Wikipedia 构建离线检索系统，组合 BM25、Qwen3 Embedding 与Re-ranker模型。
- 从文档中抽取实体，通过页面链接与随机游走构建实体图，再使用 LLM 合成问题和搜索轨迹。
- 标注trajectory中不同entity以及hidden state的状态，将 Agent trajectory的过程为evidence grounding、query reformulation等原子能力，构建不同类型的原子能力数据，并采用课程式中训练组织不同难度数据。
- 自建原子能力benchmark评估mid-training训练效果，并接入后训练sft使用browsecomp评估集评估在线search agent能力。

## 结果

形成了“检索构建—图上采样—轨迹合成—质量控制—能力评测”的完整多跳数据生产流程。
