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

本项目主要参考了高质量数据改写、训练阶段的数据分配，以及数学与代码数据多样性三个方向的研究。

### 高质量数据改写

- **[BeyondWeb: Lessons from Scaling Synthetic Data for Trillion-scale Pretraining](https://arxiv.org/abs/2508.10975)** 系统比较了源数据质量与改写策略对预训练效果的影响。其结论说明，合成数据的收益不只取决于生成规模，还高度依赖源数据质量和改写方式。这支持本项目优先选择高质量种子，再进行受控改写。
- **[How to Synthesize Text Data without Model Collapse?](https://arxiv.org/abs/2412.14689)** 指出直接扩大合成数据比例可能带来分布偏移和 n-gram 特征过度集中，并通过在人类文本上进行局部 Token Editing 兼顾数据质量与多样性。这启发本项目避免大量生成结构高度相似的模板数据，并在合成阶段显式控制题型、背景和难度。

### 训练阶段的数据分配

- **[Front-Loading Reasoning: The Synergy between Pretraining and Post-Training Data](https://arxiv.org/abs/2510.03264)** 对比了推理数据在不同训练阶段的作用，指出早期训练更受益于推理模式的广泛覆盖，而 SFT 对数据质量更敏感。基于这一观察，本项目在 Mid-training 数据生产阶段优先扩大知识点、题型和推理路径的多样性，再通过过滤和评测控制样本质量。

### 代码数据合成

- **[KodCode: A Diverse, Challenging, and Verifiable Synthetic Dataset for Coding](https://arxiv.org/abs/2503.02951)** 从 12 类数据源合成代码问题，覆盖基础编程、数据结构与算法、代码文档和现实开发任务；同时通过“问题—解答—测试”三元组与执行验证保证可验证性。该工作为本项目设计代码数据的能力覆盖范围提供了参考。
- **Repository 级数据合成** 将代码生成从孤立函数扩展到跨文件、上下文相关的任务，为构造更接近真实软件开发环境的训练数据提供了方向。

上述研究共同形成了本项目的三项数据设计原则：从高质量种子出发，在 Mid-training 阶段保持题型与推理模式的多样性，并针对数学和代码任务分别建立可执行、可评测的数据质量标准。

### 与本项目的联系：一致性投票与分领域过滤

对于问题 \(x\)，设 \(M\) 次独立 Rollout 生成的候选答案为 \(y_1(x),\ldots,y_M(x)\)。在对数值和符号表达进行答案归一化后，多答案投票可表示为：

$$
\hat{y}(x)
= \operatorname*{arg\,max}_{a\in\mathcal{A}(x)}
\sum_{m=1}^{M}\mathbf{1}\!\left[
\operatorname{norm}\!\left(y_m(x)\right)=a
\right]
$$

其中 \(\mathcal{A}(x)\) 是归一化后的候选答案集合。对质量过滤，令 \(s_{\mathrm{quality}}(x)\) 表示分数越高、样本质量越好的 LLM 评分，\(\gamma_{d(x)}\) 表示样本所属领域的过滤阈值，则：

$$
\operatorname{keep}(x)
= \mathbf{1}\!\left[
s_{\mathrm{quality}}(x)\ge \gamma_{d(x)}
\right]
$$

这两个形式分别对应“多模型/多次采样的答案一致性”和“分领域阈值”：前者降低单次生成的偶然性，后者避免统一全局阈值对不同数据域造成不均衡的误杀或漏检。投票公式只用于选择候选答案，样本是否最终保留仍由后续的 LLM 质量过滤、规则约束与沙箱执行检查决定。

## 我的工作

### 数据生产与基础质量控制

- 设计覆盖问题与答案合成、LLM 质量过滤、知识规则约束、多答案投票和沙箱执行检查的数据流水线。
- 过滤不可执行、题意歧义及沙箱执行失败的样本，最终形成约 **70B Token** 的有效训练数据。

### 数学数据合成

- 对高质量数据进行改写，在保留原始知识与解题逻辑的基础上扩展题目表达。
- 构建知识体系驱动的数据合成流程：
  1. 从现有 QA Pair 中抽取知识点；
  2. 基于知识点的共现关系进行采样，控制合成数据的多样性；
  3. 将两个问题组合为一个新问题，提升知识组合与推理复杂度；
  4. 使用多个小模型 Rollout 候选答案，再通过多答案投票筛选符合要求的样本。
- 在合成 Prompt 中显式控制题目类型、背景和难度，提升数据分布的可控性。

### 质量过滤与方案取舍

- 使用 `Qwen3-30B-A3B-Instruct-2507` 作为质量过滤模型，在人工标注集上通过 Precision 和 Recall 评估过滤 Prompt；针对低质量样本的检出，更关注 Recall，以减少漏检。
- 根据数据已标注的领域分别校准过滤阈值，避免统一全局阈值带来误杀或漏检。
- 尝试使用过程奖励模型对推理步骤逐步打分，再按步骤位置加权组合分数，思路类似强化学习中的 Return 计算。该方案的 Precision 和 Recall 均低于 LLM-based Filtering，因此未作为最终方案。

### 评测驱动的数据质量闭环

- 通过等 Token 对照实验区分数据规模收益与数据质量收益，并利用细粒度能力评测和错误分析反向调整清洗、采样与合成策略。
- 对数学评测中的模型输出进行错误归因，包括知识缺失、知识使用错误、计算错误和推理中断。
- 对代码评测中的模型输出进行错误归因，包括模板化回答、执行错误、引用不存在的库和执行超时；针对模板化回答、答案提前中断等反复出现且易于识别的模式，定向调整数据过滤与补充策略。
- 建立持续迭代的“数据—评测”闭环：
  1. 标注并分析模型输出中的错误类型；
  2. 使用 LLM 归纳错误模式，生成或迭代质量过滤 Prompt；
  3. 使用新 Prompt 重新标注训练数据，再进行模型训练与 Benchmark 评测；
  4. 再次分析模型错误，并据此更新过滤、采样和合成策略。
- 将模型解题过程中的状态转移评测结果与子领域错误分数作为重采样信号，针对薄弱能力调整后续训练数据分布。

## 结果

- 在代表性实验中，MATH 中等难度能力提升约 **1 个百分点**。
- 代码评测 **Pass@128 从 0.77 提升至 0.83**。
