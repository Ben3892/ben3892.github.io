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
- 数据改写侧
  - beyondweb：发现对高质量数据进行改写能提升模型benchmark分数。高质量数据+改写>高质量数据>低质量数据+改写>教科书类型合成数据

  - front loading to reasoning（nvidia）：mid-train应该更关注数据的多样性，sft阶段更应该关注数据质量、数据难度。

  - How to Synthesize Text Data without Model Collapse?: 发现教科书类型合成数据容易导致模型表征collapse，合成数据的ppl低且集中，而真实数据的ppl存在长尾分布，论文提出了一个方法：对于模型不确定的部分进行重新采样来合成数据，避免降低数据多样性。

- reasoning数据合成


- 代码数据合成
  - kod coder：合成12种类型的代码qa数据。包含代码文档、数据结构/算法、基础知识、现实问题pr
  - github repo级别数据合成

## 我的工作

- 设计覆盖问题与答案合成、LLM 过滤、知识规则约束、多答案投票及沙箱执行检查的流水线。
- 过滤不可执行、题意歧义、沙箱失败样本，累计形成约 **70B 有效 Token**。
- 数学数据合成/过滤
  - 1. 使用rewrite方式对高质量数据进行改写
  - 2. 使用知识体系合成数据，对qa pair提取出知识点，通过共现采样控制多样性，2个问题合成1个问题，在使用几个小模型rollout出answer，通过投票的方式筛选出符合要求的数据，prompt内控制多样性和难度（题目类型、背景和难度）
  - 3. 使用qwen3-30b-a3b-instruct-2507作为质量过滤模型，评价prompt指标：（precision和recall，recall更重要一些）

- 一些失败的尝试：
  - 1. 使用过程奖励模型对模型的步骤打分，组合分数（排序越靠后的步骤分数权重越低，类似于rl的return计算），precision和recall都低于llm based filtering

- 实践细节：
  - llm打分后，分数应该根据已经标注的领域选择阈值，而不是全局阈值

- 使用等 Token 对照实验拆分规模收益与质量收益，并通过细粒度能力评测和错误分析反向调整清洗、采样与合成策略。
- 发现代码上存在某些特定错误模式：生成模板数据、answer生成中断（未达到最大token数）。因此通过标注评估集错误来降低模型固定（且容易发现的）错误。
  - 对于数学benchmark的模型response，标注了“知识缺失”、“知识错误使用”、“计算错误”、“推理中断”错误原因
  - 对于代码benchmark的模型response，标注了“模板回答”、“执行错误”、“引用不存在的库”、“执行超时”错误原因
  - 错误-评估打标闭环：对于上述错误类型，使用大模型生成质量过滤的prompt，重新标注一批数据后再训练模型，得到模型benchmark分数后重新标注模型并使用llm 分析错误类型并生成质量过滤prompt来形成数据-评估闭环
  - 模型解题所需状态转移-评估分数-重新采样闭环
  - 模型子领域错误分数-重新采样闭环
## 结果

- 在代表性实验中，MATH 中等难度能力提升约 **1 个百分点**。
- 代码评测 **Pass@128 从 0.77 提升至 0.83**。
