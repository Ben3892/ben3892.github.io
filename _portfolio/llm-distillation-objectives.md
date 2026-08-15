---
title: "大模型蒸馏目标设计与规模化验证"
collection: portfolio
permalink: /portfolio/llm-distillation-objectives/
published: true
order: 1
period: "2025.08 — 至今"
project_type: "LLM TRAINING"
role: "训练目标设计 · 实验归因"
stack:
  - Knowledge Distillation
  - Forward KL
  - NTP
  - MoE
excerpt: "分析持续预训练中 Forward KL 与 NTP 的领域能力权衡，并将动态监督策略从代理实验扩展到目标模型。"
author_profile: true
share: false
comments: false
read_time: false
---

## 项目背景

参与 100B 级 MoE 搜索大模型预训练与中训练，围绕模型裁剪后的能力恢复、领域自适应训练以及蒸馏监督信号选择开展实验。

## 我的工作

- 搭建离线 Forward KL 蒸馏训练流程，并在 T 级预训练数据上进行模型恢复训练。
- 对不同领域、Top-k、Temperature、损失比例与数据配比开展系统消融，定位 Forward KL 与 NTP 在不同能力维度上的收益差异。
- 根据 Teacher 优势区域动态选择监督目标，在代理模型完成验证后扩展到目标模型。

## 结果

- 裁剪后 Base 模型在代表性评测上保留 Teacher 约 **92%** 的能力。
- 动态目标策略在代理实验和目标模型上均优于单独使用 KL 或 NTP 的方案。

> 公开页面仅展示可对外说明的方法与结果，具体模型和内部实验配置已省略。
