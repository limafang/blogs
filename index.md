---
layout: default
title: 首页
description: "记录论文阅读、研究想法与方法论追问。"
---

<section class="home-intro">
  <div class="intro-copy">
    <div class="eyebrow">RESEARCH NOTES · 2026</div>
    <h1>沿着证据<br><em>思考下去</em></h1>
    <p>这里记录论文阅读中没有被摘要带走的东西：反直觉的机制、尚未闭合的因果链，以及值得继续追问的问题。</p>
  </div>
  <div class="intro-aside">
    <span class="index-number">02</span>
    <span class="index-label">已发布文章</span>
  </div>
</section>

<section class="section-heading">
  <div>
    <span class="eyebrow">LATEST WRITING</span>
    <h2>最近写的</h2>
  </div>
  <span class="section-note">按时间倒序</span>
</section>

<section class="post-list" aria-label="文章列表">
  {% assign posts = site.pages | where: "layout", "post" | sort: "date" | reverse %}
  {% for post in posts %}
  <a class="post-card" href="{{ post.url | relative_url }}">
    <div class="post-card-index">{{ forloop.index | prepend: '0' }}</div>
    <div class="post-card-body">
      <div class="post-card-kicker">{{ post.categories | first | default: '文章' }} <span>/</span> {{ post.date | date: "%Y.%m.%d" }}</div>
      <h3>{{ post.title }}</h3>
      <p>{{ post.description }}</p>
      <span class="read-more">阅读全文 <span aria-hidden="true">→</span></span>
    </div>
    <div class="post-card-arrow" aria-hidden="true">↗</div>
  </a>
  {% endfor %}
</section>

<section class="home-note">
  <div class="note-mark">“</div>
  <p>好的研究笔记，不是把论文再讲一遍，而是把一个值得验证的疑问留下来。</p>
</section>
