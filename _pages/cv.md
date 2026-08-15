---
layout: archive
title: "简历"
permalink: /cv/
author_profile: false
redirect_from:
  - /resume
  - /resume/
  - /cv-json/
  - /resume-json
  - /resume-json/
---

{% assign cv_pdf = "/files/yuan-jiangan-cv-2026.pdf" | relative_url %}

<div class="resume-page">
  <header class="resume-hero">
    <p class="resume-hero__eyebrow">CURRICULUM VITAE · UPDATED 2026</p>
    <div class="resume-hero__top">
      <div class="resume-hero__identity">
        <h2 class="resume-hero__name">
          袁江岸 <span class="resume-hero__english-name">Ben Yuan</span>
        </h2>
        <p class="resume-hero__role">大模型算法工程师 · LLM Training / Distillation / Data</p>
      </div>
      <div class="resume-hero__actions">
        <a class="btn btn--primary" href="{{ cv_pdf }}" target="_blank" rel="noopener noreferrer">下载 PDF 简历</a>
        <a class="btn btn--inverse" href="mailto:{{ site.author.email }}">邮件联系</a>
      </div>
    </div>

    <ul class="resume-hero__meta" aria-label="基本信息">
      <li><span>当前</span><strong>百度搜索策略部</strong></li>
      <li><span>方向</span><strong>预训练 / 中训练 / 知识蒸馏</strong></li>
      <li><span>所在地</span><strong>北京</strong></li>
      <li><span>邮箱</span><a href="mailto:{{ site.author.email }}">{{ site.author.email }}</a></li>
    </ul>
    <p class="resume-hero__note">网页版本默认不展示手机号与年龄；PDF 为 2026-07-20 完整版本。</p>
  </header>

  <section class="resume-section" aria-labelledby="resume-work">
    <div class="resume-section__heading">
      <span class="resume-section__index">01</span>
      <h2 id="resume-work">工作经历</h2>
    </div>

    <article class="resume-entry">
      <div class="resume-entry__content">
        <div class="resume-entry__header">
          <div>
            <h3 class="resume-entry__title">百度 · 搜索策略部</h3>
            <p class="resume-entry__role">大模型预训练 / 中训练算法</p>
          </div>
          <time class="resume-entry__period" datetime="2025-08">2025.08 — 至今</time>
        </div>
        <p class="resume-entry__summary">
          参与 100B 级 MoE 搜索大模型预训练与中训练，负责数学/代码数据策略、训练目标设计、评测归因与搜索 Agent 数据合成。
        </p>
        <ul class="resume-bullets">
          <li><strong>蒸馏目标：</strong>完成裁剪后模型的离线 Forward KL 恢复训练，在代表性评测上保留 Teacher 约 92% 的能力；系统分析 Forward KL 与 NTP 的领域差异，并设计动态目标路由方案。</li>
          <li><strong>数学与代码数据：</strong>搭建合成、过滤、沙箱校验、投票与评测闭环，产出约 70B 有效 Token；等 Token 实验中，MATH 中等难度提升约 1 个百分点，Pass@128 从 0.77 提升至 0.83。</li>
          <li><strong>搜索 Agent：</strong>基于 Wikipedia、BM25、向量召回、重排与实体图构建多跳推理数据，围绕查询改写、证据定位等能力进行课程式训练与评测。</li>
        </ul>
      </div>
    </article>

    <article class="resume-entry">
      <div class="resume-entry__content">
        <div class="resume-entry__header">
          <div>
            <h3 class="resume-entry__title">字节跳动 · 飞书搜索团队</h3>
            <p class="resume-entry__role">企业知识问答模型训练与优化</p>
          </div>
          <time class="resume-entry__period" datetime="2024-07">2024.07 — 2025.02</time>
        </div>
        <p class="resume-entry__summary">围绕企业知识问答场景开展 SFT 训练与效果优化，并关注模型推理效率与工程可用性。</p>
      </div>
    </article>
  </section>

  <section class="resume-section" aria-labelledby="resume-education">
    <div class="resume-section__heading">
      <span class="resume-section__index">02</span>
      <h2 id="resume-education">教育经历</h2>
    </div>

    <article class="resume-entry">
      <div class="resume-entry__content">
        <div class="resume-entry__header">
          <div>
            <h3 class="resume-entry__title">香港中文大学（深圳）</h3>
            <p class="resume-entry__role">数据科学 · 硕士（全日制）</p>
          </div>
          <time class="resume-entry__period">2023.09 — 2025.07</time>
        </div>
      </div>
    </article>

    <article class="resume-entry">
      <div class="resume-entry__content">
        <div class="resume-entry__header">
          <div>
            <h3 class="resume-entry__title">华南理工大学</h3>
            <p class="resume-entry__role">工业工程 · 本科（全日制）</p>
          </div>
          <time class="resume-entry__period">2019.09 — 2023.07</time>
        </div>
      </div>
    </article>
  </section>

  <section class="resume-section" aria-labelledby="resume-impact">
    <div class="resume-section__heading">
      <span class="resume-section__index">03</span>
      <h2 id="resume-impact">代表成果</h2>
    </div>

    <div class="resume-metrics" aria-label="项目指标">
      <article class="resume-metric">
        <strong>≈ 92%</strong>
        <span>裁剪后模型保留的 Teacher 评测能力</span>
      </article>
      <article class="resume-metric">
        <strong>≈ 70B</strong>
        <span>数学与代码高质量有效训练 Token</span>
      </article>
      <article class="resume-metric">
        <strong>0.77 → 0.83</strong>
        <span>等 Token 质量实验中的 Pass@128</span>
      </article>
    </div>

    <article class="resume-publication">
      <p class="resume-publication__label">PREPRINT · 2026</p>
      <h3>Let the Data Decide: Supervision Analysis, Capability Trade-offs, and Adaptive Objective Routing in Continued Pre-Training via Off-Policy Distillation</h3>
      <p>主导研究思路、实验设计、实现与评估，分析持续预训练中不同监督信号的能力权衡与自适应选择。</p>
      <a href="https://arxiv.org/abs/2607.16246" target="_blank" rel="noopener noreferrer">查看 arXiv <span aria-hidden="true">↗</span></a>
    </article>

    <div class="resume-links">
      <a href="{{ "/portfolio/llm-distillation-objectives/" | relative_url }}">蒸馏目标设计</a>
      <a href="{{ "/portfolio/math-code-data-pipeline/" | relative_url }}">数学与代码数据闭环</a>
      <a href="{{ "/portfolio/search-agent-data-synthesis/" | relative_url }}">搜索 Agent 数据合成</a>
    </div>
  </section>

  <section class="resume-section" aria-labelledby="resume-skills">
    <div class="resume-section__heading">
      <span class="resume-section__index">04</span>
      <h2 id="resume-skills">专业能力</h2>
    </div>

    <div class="resume-skills">
      <article class="resume-skill-group">
        <h3>模型训练</h3>
        <div class="resume-chip-list">
          <span class="resume-chip">LLM Pre-training</span>
          <span class="resume-chip">Mid-training</span>
          <span class="resume-chip">Knowledge Distillation</span>
          <span class="resume-chip">Forward KL</span>
          <span class="resume-chip">NTP</span>
        </div>
      </article>
      <article class="resume-skill-group">
        <h3>数据与评测</h3>
        <div class="resume-chip-list">
          <span class="resume-chip">Data Synthesis</span>
          <span class="resume-chip">Quality Filtering</span>
          <span class="resume-chip">Benchmark Attribution</span>
          <span class="resume-chip">Curriculum Learning</span>
        </div>
      </article>
      <article class="resume-skill-group">
        <h3>检索与 Agent</h3>
        <div class="resume-chip-list">
          <span class="resume-chip">BM25</span>
          <span class="resume-chip">Embedding / Reranking</span>
          <span class="resume-chip">Entity Graph</span>
          <span class="resume-chip">Multi-hop Reasoning</span>
        </div>
      </article>
      <article class="resume-skill-group">
        <h3>训练系统</h3>
        <div class="resume-chip-list">
          <span class="resume-chip">Distributed Training</span>
          <span class="resume-chip">CUDA</span>
          <span class="resume-chip">Triton</span>
          <span class="resume-chip">Kernel Fusion</span>
        </div>
      </article>
    </div>
  </section>

  <footer class="resume-hero resume-hero--compact">
    <div class="resume-hero__top">
      <div>
        <p class="resume-hero__eyebrow">CONTACT</p>
        <h2>欢迎交流大模型训练、蒸馏与数据研究。</h2>
      </div>
      <div class="resume-hero__actions">
        <a class="btn btn--primary" href="mailto:{{ site.author.email }}">{{ site.author.email }}</a>
        <a class="btn btn--inverse" href="https://github.com/{{ site.author.github }}" target="_blank" rel="noopener noreferrer">GitHub</a>
        {% if site.author.googlescholar %}
          <a class="btn btn--inverse" href="{{ site.author.googlescholar }}" target="_blank" rel="noopener noreferrer">Google Scholar</a>
        {% endif %}
      </div>
    </div>
  </footer>
</div>
