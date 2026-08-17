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
损失函数的研究
- reverse kl损失（miniLLM）
- abkd损失函数

蒸馏超参数的研究（top-k、top-p、temperature、loss）
- distillation pretraining（智谱）
  - 
- distilled pretrain（meta）

业界相关实践
- gemma2、gemma3
- slimqwen
- mistral 3

## 我的工作

- 搭建离线 Forward KL 蒸馏训练流程，并在 T 级预训练数据上进行模型恢复训练。
- 对不同领域、Top-k、Temperature、loss比例开展系统消融，定位 Forward KL 与 NTP 在不同能力维度上的收益差异。
- 根据 Teacher 优势区域动态选择监督目标，在代理模型完成验证后扩展到目标模型。

## 结果

- v1版本模型裁剪后 Base 模型在代表性评测上保留 Teacher 约 **92%** 的能力（benchmark分数），数学为85%、代码为88%。动态目标策略补充了studnet在数学和代码的短板。
- 动态目标策略在代理实验和目标模型上均优于单独使用 KL 或 NTP 的方案。（具体形式：general benchmark上优于NTP方案、math/code benchmark上优于Forward KL方案）

> 公开页面仅展示可对外说明的方法与结果，具体模型和内部实验配置已省略。
