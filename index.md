---
layout: default
title: 首页
description: "Limafang 的读书与研究随记。"
---

<section class="home-intro">
  <div class="intro-copy">
    <div class="eyebrow">LIMAFANG · 随手记</div>
    <h1>这世间本就是各人下雪，<br><em>各人有各人的隐晦和皎洁。</em></h1>
    <p>读书、读论文，顺手记下一些当时想到的东西。</p>
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
  <div class="note-content">
    <p>诸君，且努力向前，须知天日昭昭。</p>
  </div>
</section>
