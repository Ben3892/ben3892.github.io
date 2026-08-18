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
  - 使用opd的方式（student rollout出trajectory，teacher在student的trajectory上生成logits供student学习，使用rkl作为损失函数）训练模型
- abkd损失函数
  - 开发一个蒸馏loss，融合forward-kl、reverse-kl等模式训练模型，但是有2个超参数需要调
- jsd loss
  - student学习student和teacher的logits平均值，降低teacher-student capacity gap的问题。
- sparse logits sampling
  - 重采样teacher logits，降低存储成本以及提供更真实的logits给student学习

蒸馏超参数的研究（top-k、top-p、temperature、loss）
- distillation pretraining（智谱）
  - 单点探索各种蒸馏超参数的最优设置，然后组合单点最优设置到正式训练
- distilled pretrain（meta）
  - 探索蒸馏对pass@k、不同领域的影响，提出蒸馏可能损害in-context learning，通过case study提出entropy based token routing策略训练模型，结果能保护蒸馏的in-context learning benchmark分数

业界相关实践
- gemma 2、gemma 3
- slimqwen
- mistral 3

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
