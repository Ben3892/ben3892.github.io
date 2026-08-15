---
permalink: /
title: "Ben (Jiangan) Yuan"
author_profile: true
---

<section class="home-intro" aria-labelledby="home-intro-title">
  <p class="home-intro__eyebrow">LLM SYSTEMS · DISTILLATION · HIGH-PERFORMANCE COMPUTING</p>
  <h2 id="home-intro-title" class="home-intro__title">让大模型训练更高效、更可靠。</h2>
  <p class="home-intro__lead">
    我目前专注于大语言模型训练、知识蒸馏、分布式训练系统，以及 CUDA / Triton 高性能算子优化。
  </p>
  <div class="home-intro__actions">
    <a class="btn btn--primary" href="{{ site.baseurl }}/publications/">查看论文</a>
    <a class="btn btn--inverse" href="{{ site.baseurl }}/year-archive/">阅读博客</a>
  </div>
</section>

<section class="home-section" aria-labelledby="about-heading">
  <div class="section-heading">
    <p class="section-heading__index">01</p>
    <h2 id="about-heading">关于我</h2>
  </div>
  <p>
    我在百度从事大模型训练与系统优化相关工作，关注如何通过算法与系统协同提升训练效率、模型质量和工程可用性。
  </p>
  <p class="content-placeholder">
    【待填写：用 2–3 句话补充教育背景、此前经历，以及你最希望解决的研究或工程问题。】
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
      <p class="work-card__type">PAPER / PROJECT</p>
      <h3>【待填写：代表论文或项目名称】</h3>
      <p>【待填写：一句话说明问题、你的核心贡献和结果。】</p>
      <p class="work-card__links">【待填写：论文 / 代码 / 项目主页链接】</p>
    </article>
    <article class="work-card">
      <p class="work-card__type">PAPER / PROJECT</p>
      <h3>【待填写：代表论文或项目名称】</h3>
      <p>【待填写：一句话说明问题、你的核心贡献和结果。】</p>
      <p class="work-card__links">【待填写：论文 / 代码 / 项目主页链接】</p>
    </article>
    <article class="work-card">
      <p class="work-card__type">OPEN SOURCE</p>
      <h3>【待填写：代表性开源工作】</h3>
      <p>【待填写：说明项目用途、技术特点和可量化效果。】</p>
      <p class="work-card__links">【待填写：GitHub / 文档链接】</p>
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
      <li><time>【待填写：YYYY-MM】</time><span>【待填写：论文录用、项目发布、演讲或工作动态】</span></li>
      <li><time>【待填写：YYYY-MM】</time><span>【待填写：第二条真实动态】</span></li>
      <li><time>【待填写：YYYY-MM】</time><span>【待填写：第三条真实动态】</span></li>
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
      <p class="content-placeholder">【待填写：在 <code>_posts/</code> 中发布第一篇技术博客。】</p>
    {% endif %}
  </div>
</section>

<section class="home-section contact-panel" aria-labelledby="contact-heading">
  <div>
    <p class="section-heading__index">06</p>
    <h2 id="contact-heading">联系与合作</h2>
    <p>欢迎围绕大模型训练、知识蒸馏、分布式系统和高性能算子进行交流。</p>
    <p class="content-placeholder">【待填写：希望合作的具体主题，或最适合联系你的方式。】</p>
  </div>
  <div class="contact-panel__actions">
    <a class="btn btn--primary" href="mailto:{{ site.author.email }}">发送邮件</a>
    <a class="btn btn--inverse" href="https://github.com/{{ site.author.github }}">GitHub</a>
  </div>
</section>
