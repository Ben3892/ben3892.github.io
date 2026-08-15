---
layout: archive
title: "简历"
permalink: /cv/
author_profile: true
redirect_from:
  - /resume
---

<p class="archive-intro">Ben (Jiangan) Yuan · 大模型训练、知识蒸馏与高性能计算</p>

## 当前职位

- **Baidu, Inc.** — 【待填写：职位名称与所属团队】
- 【待填写：起止时间】
- 【待填写：用 2–3 个要点概括职责、技术贡献和可量化结果】

## 教育经历

- **【待填写：学校名称】** — 【待填写：学位与专业】， 【待填写：起止年份】
  - 【待填写：导师、研究方向、论文题目或重要经历】
- **【待填写：学校名称】** — 【待填写：学位与专业】， 【待填写：起止年份】

## 工作经历

- **【待填写：公司或机构】** — 【待填写：职位】， 【待填写：起止时间】
  - 【待填写：最重要的工作内容和贡献】
  - 【待填写：性能、规模、质量或业务结果】

## 研究与项目

- **【待填写：项目或研究名称】**
  - 【待填写：问题背景、个人贡献、技术方案和结果】
  - 【待填写：论文、代码或项目主页链接】

## 论文

{% assign visible_publications = site.publications | where_exp: "post", "post.published != false" %}
{% assign publication_count = visible_publications | size %}
{% if publication_count > 0 %}
  <ul>
  {% for post in visible_publications reversed %}
    {% include archive-single-cv.html %}
  {% endfor %}
  </ul>
{% else %}
  <p class="content-placeholder">【待填写：代表论文；也可以先维护 <code>_publications/</code>，这里会自动生成。】</p>
{% endif %}

## 技术能力

- **大模型训练：**【待填写：预训练、后训练、蒸馏等具体能力】
- **分布式系统：**【待填写：Megatron-LM、并行策略、通信优化等】
- **高性能计算：**【待填写：CUDA、Triton、算子融合和性能分析等】
- **编程语言：**【待填写：Python、C++、CUDA 等】

## 奖项与服务

- 【待填写：奖项、专利、开源社区贡献、审稿或演讲经历】

## 下载

【待填写：将 PDF 简历放入 <code>files/</code>，然后在这里添加下载链接。】
