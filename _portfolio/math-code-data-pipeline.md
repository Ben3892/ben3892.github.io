---
title: "数学与代码数据合成—过滤—评测闭环"
collection: portfolio
permalink: /portfolio/math-code-data-pipeline/
published: true
order: 2
period: "2025.08 — 2025.12"
project_type: "DATA & EVALUATION"
role: "数据策略 · 质量评测"
stack:
  - Data Synthesis
  - LLM Filtering
  - Sandbox Validation
  - Capability Evaluation
excerpt: "构建数学与代码训练数据的生产和质量闭环，通过等 Token 对照实验拆分数据规模与质量收益。"
author_profile: true
share: false
comments: false
read_time: false
---

## 项目背景

大规模训练数据的收益同时受到数量、难度、可执行性与答案一致性的影响。项目目标是把数据生产、过滤、训练评测和错误归因连成可持续迭代的闭环。


## 相关研究



## 我的工作

- 设计覆盖问题与答案合成、LLM 过滤、知识规则约束、多答案投票及沙箱执行检查的流水线。
- 过滤不可执行、题意歧义和沙箱失败样本，累计形成约 **70B 有效 Token**。
- 使用等 Token 对照实验拆分规模收益与质量收益，并通过细粒度能力评测和错误分析反向调整清洗、采样与合成策略。

## 结果

- 在代表性实验中，MATH 中等难度能力提升约 **1 个百分点**。
- 代码评测 **Pass@128 从 0.77 提升至 0.83**。
