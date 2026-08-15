---
permalink: /
title: "Ben (Jiangan) Yuan"
author_profile: true
---

<section class="home-intro" aria-labelledby="home-intro-title">
  <p class="home-intro__eyebrow">LLM SYSTEMS · DISTILLATION · HIGH-PERFORMANCE COMPUTING</p>
  <h2 id="home-intro-title" class="home-intro__title">让大模型训练更高效、更可靠。</h2>
  <p class="home-intro__lead">
    我目前在百度搜索策略部从事大语言模型预训练与中训练，关注知识蒸馏、训练数据闭环、搜索 Agent，以及 CUDA / Triton 高性能算子优化。
  </p>
  <div class="home-intro__actions">
    <a class="btn btn--primary" href="{{ site.baseurl }}/publications/">查看论文</a>
    <a class="btn btn--inverse" href="{{ site.baseurl }}/blog/">阅读博客</a>
  </div>
</section>

<section class="home-section" aria-labelledby="about-heading">
  <div class="section-heading">
    <p class="section-heading__index">01</p>
    <h2 id="about-heading">关于我</h2>
  </div>
  <p>
    2025 年 8 月至今，我在百度参与 100B 级 MoE 搜索大模型的预训练与中训练，主要负责蒸馏目标、数学与代码数据策略、评测归因和搜索 Agent 数据合成。
  </p>
  <p>
    我于香港中文大学（深圳）获得数据科学硕士学位，本科毕业于华南理工大学工业工程专业。此前曾在字节跳动飞书搜索团队从事企业知识问答模型的 SFT 训练与优化。
  </p>
</section>

<section class="home-section" aria-labelledby="research-heading">
  <div class="section-heading">
    <p class="section-heading__index">02</p>
    <h2 id="research-heading">研究方向</h2>
  </div>
  <div class="interest-grid">
    <article class="interest-card">
      <p class="interest-card__number">01</p>
      <h3>LLM Training</h3>
      <p>大语言模型预训练、后训练和稳定高效的训练方法。</p>
    </article>
    <article class="interest-card">
      <p class="interest-card__number">02</p>
      <h3>Knowledge Distillation</h3>
      <p>面向大模型的知识迁移、分布对齐和高效蒸馏目标。</p>
    </article>
    <article class="interest-card">
      <p class="interest-card__number">03</p>
      <h3>Distributed Systems</h3>
      <p>分布式训练、并行策略，以及大规模训练系统性能优化。</p>
    </article>
    <article class="interest-card">
      <p class="interest-card__number">04</p>
      <h3>CUDA / Triton Kernels</h3>
      <p>训练关键算子的融合、显存优化与硬件效率提升。</p>
    </article>
  </div>
</section>

<section class="home-section" aria-labelledby="work-heading">
  <div class="section-heading">
    <p class="section-heading__index">03</p>
    <h2 id="work-heading">代表成果</h2>
  </div>
  <div class="work-grid">
    <article class="work-card">
      <p class="work-card__type">LLM TRAINING</p>
      <h3>大模型蒸馏目标设计</h3>
      <p>分析 Forward KL 与 NTP 的领域权衡，并以动态监督策略完成规模化验证；裁剪后模型保留 Teacher 约 92% 的评测能力。</p>
      <p class="work-card__links"><a href="{{ '/portfolio/llm-distillation-objectives/' | relative_url }}">项目详情 →</a></p>
    </article>
    <article class="work-card">
      <p class="work-card__type">PREPRINT · 2026</p>
      <h3>Let the Data Decide</h3>
      <p>主导研究思路、实验设计、实现与评估，系统研究持续预训练中的监督信号选择、能力权衡与自适应目标路由。</p>
      <p class="work-card__links"><a href="https://arxiv.org/abs/2607.16246" target="_blank" rel="noopener noreferrer">arXiv ↗</a></p>
    </article>
    <article class="work-card">
      <p class="work-card__type">DATA & EVALUATION</p>
      <h3>数学与代码数据质量闭环</h3>
      <p>构建合成、过滤、沙箱校验与评测归因链路，形成约 70B 有效 Token；代表性实验中 Pass@128 从 0.77 提升至 0.83。</p>
      <p class="work-card__links"><a href="{{ '/portfolio/math-code-data-pipeline/' | relative_url }}">项目详情 →</a></p>
    </article>
  </div>
</section>

<section class="home-section home-columns" aria-label="动态与博客">
  <div>
    <div class="section-heading">
      <p class="section-heading__index">04</p>
      <h2>最新动态</h2>
    </div>
    <ul class="timeline-list">
      <li><time>2026-07</time><span>预印本 <em>Let the Data Decide</em> 发布于 arXiv。</span></li>
      <li><time>2025-08</time><span>加入百度搜索策略部，从事大模型预训练与中训练。</span></li>
      <li><time>2025-07</time><span>获香港中文大学（深圳）数据科学硕士学位。</span></li>
    </ul>
  </div>

  <div>
    <div class="section-heading">
      <p class="section-heading__index">05</p>
      <h2>近期文章</h2>
    </div>
    {% assign visible_posts = site.posts | where_exp: "post", "post.published != false" %}
    {% if visible_posts.size > 0 %}
      <ul class="recent-posts">
        {% for post in visible_posts limit:3 %}
          <li>
            <time datetime="{{ post.date | date_to_xmlschema }}">{{ post.date | date: "%Y-%m-%d" }}</time>
            <a href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a>
          </li>
        {% endfor %}
      </ul>
    {% else %}
      <div class="empty-state">第一篇技术文章正在整理中。</div>
      <!-- 维护提示：从 _drafts/post-draft.md 复制模板后发布到 _posts/。 -->
    {% endif %}
  </div>
</section>

<section class="home-section contact-panel" aria-labelledby="contact-heading">
  <div>
    <p class="section-heading__index">06</p>
    <h2 id="contact-heading">联系与合作</h2>
    <p>欢迎围绕大模型训练、知识蒸馏、分布式系统和高性能算子进行交流。</p>
    <p>学术讨论、工程实践或潜在合作，都可以通过邮件与我联系。</p>
  </div>
  <div class="contact-panel__actions">
    <a class="btn btn--primary" href="mailto:{{ site.author.email }}">发送邮件</a>
    <a class="btn btn--inverse" href="https://github.com/{{ site.author.github }}">GitHub</a>
  </div>
</section>
