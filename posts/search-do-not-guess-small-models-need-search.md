---
layout: post
title: "小模型明明更不确定，为什么反而更少搜索？"
description: "从 Always-Search Policy 追问：模型的不确定性为什么没有自动变成信息获取行动？"
date: 2026-08-24
categories: [研究笔记]
tags: [Language Model, Search Agent, 蒸馏]
permalink: /posts/search-do-not-guess-small-models-need-search/
---

最近读了篇论文：[Search, Do not Guess: Teaching Small Language Models to Be Effective Search Agents](https://arxiv.org/abs/2604.04651)。它提出的办法叫 Always-Search Policy（ASP）：训练小语言模型做搜索 agent 时，尽量让它不要依赖参数记忆，而是先调用搜索工具，再基于检索到的证据回答。

方法本身并不复杂。真正让我停下来的是论文开头的一个反直觉观察：**小模型明明知道得更少，却比大模型调用搜索更少，也更容易幻觉。**

直觉上，小模型应该更不自信才对。既然更不确定，为什么不去查？

我觉得，这个问题比 ASP 本身有意思得多。

## “不知道”不会自动变成“去搜索”

先把几个概念分开。模型对答案的置信度、搜索的预期价值，以及最终是否真的调用工具，并不是同一个东西：

$$
c \neq V(\text{search}) \neq P(\text{search}).
$$

小模型的参数知识更少，完全可能使答案置信度 $c$ 下降。但要让搜索概率 $P(\text{search})$ 上升，它还需要学会一条额外的映射：

$$
\text{低置信度}
\rightarrow
\text{诊断缺失的事实}
\rightarrow
\text{设计查询}
\rightarrow
\text{合法调用工具}.
$$

这条链并不是知识缺失后的自然反应。

比如有一道题需要沿着“人物—作品—导演—国籍”做四跳推理。小模型可能确实不知道答案，也不确定几个候选实体谁才对。但这种不确定性并没有告诉它“当前缺的是导演信息，请先搜索人物对应的作品”。它可能只是从几个低概率候选里选一个继续生成。

直接猜答案是语言模型最熟悉的动作；识别缺口、拆分问题、写出查询、遵守工具格式，则是一套需要单独训练的控制能力。于是会出现一种看似奇怪、其实并不矛盾的状态：**模型更不确定，但更不会行动。**

还有一个更具体的可能：**搜索本身也需要知识。** 小模型也许缺少构造有效 query、判断应该沿哪条路径搜索、以及理解搜索结果所需的背景知识。在“直接回答”和“乱搜、搜错、甚至搜不成”之间，它可能更倾向于前者。直接回答至少能沿着熟悉的语言生成路径继续下去；搜索则要求模型先知道自己缺什么、该向哪里找，以及什么结果算有用。

论文的 Figure 2 只观察到了外显的工具调用次数。它没有区分四种情况：模型没有意识到自己不确定；意识到了但不知道缺什么；知道缺什么但 query 写不好；query 写对了却没有生成可解析的工具格式。Appendix C–D 恰好同时报告了这些可能的失败：insufficient retrieval、hallucination、tool-call syntax failure，以及拿到证据后仍然怀疑证据。

所以，论文把 under-searching 直接归结为“过度依赖参数知识”，我觉得还不够精确。更准确的问题是：**不确定性为什么没有被转换成信息获取行动？**

## ASP 做了什么？其实就是给小模型立规矩

ASP 的方法很轻。

作者先让 Qwen3-32B teacher 生成搜索轨迹，然后只保留答案 String-F1 高于 0.65、确实调用了搜索工具、没有明显直接凭记忆回答的轨迹。接着用这些轨迹训练 Qwen3-1.7B：

- 用 SFT 模仿“先搜索、再回答”的轨迹；
- 用 on-policy distillation 继续强化这种行为；
- 在一些设置中先 SFT，再做 OPD；
- 对无效的工具调用格式进行重试或丢弃。

它没有增加新的模型结构，也没有真的学会一个复杂的 search-value estimator。核心变化只是：**不要把 teacher 所有成功轨迹都当成可蒸馏的示范，只蒸馏那些显式依赖外部证据的轨迹。**

这条经验确实有效。Bamboogle 上，普通 distilled Qwen3-1.7B 的 String-F1 是 53.2，ASP-SFT 提升到 70.6，teacher Qwen3-32B 是 73.1。HotpotQA 上，ASP-SFT 的平均搜索次数从 vanilla 的 1.72 增加到 2.47；平均延迟约 3.1 秒，仍低于 Qwen3-32B 的 10.3 秒。

但这更像一个训练纪律，而不是完整的搜索理论：**当前小模型不擅长判断什么时候可以不搜，那就先把“总是搜”作为工程上的安全默认值。**

## 一个证据注入实验，能证明什么？

论文有一个很关键、也很容易被过度解读的实验。

作者从 Qwen3-32B 已经答对的问题中抽取成功轨迹里的 retrieved evidence，直接交给 Qwen3-1.7B 生成答案。1.7B 的准确率从 47.9% 提升到 74.7%。作者据此认为，主要瓶颈在检索，而不是推理。

这个结果当然说明外部证据很重要，但它并不能单独定位瓶颈。因为实验直接绕过了：

- 是否意识到需要搜索；
- 下一步该搜索什么；
- query 是否写对；
- 工具是否调用成功；
- 是否需要继续多跳搜索。

它只证明了一个条件命题：

$$
P(\text{正确回答}\mid \text{高质量 teacher evidence})
>
P(\text{正确回答}\mid \text{没有这些 evidence}).
$$

但 evidence 可能同时完成了实体链接、问题分解和关键桥接事实的提供。它与“检索是瓶颈”相容，也与“student 不会规划下一步 search”相容，更与“拿到证据后 student 其实能完成推理”相容。

因此，我不会把 47.9% 到 74.7% 复述成“证明了小模型的推理能力没问题”。更稳妥的说法是：**高质量外部证据能显著释放小模型的答案生成能力，但这个实验没有分离检索、查询规划、工具调用和推理的贡献。**

## Teacher 的轨迹为什么不一定是 student 的程序？

论文说，LLM-generated trajectories 往往隐含依赖 SLM 没有的 parametric knowledge。这句话是有道理的，但需要准确理解。

teacher 的动作可以写成：

$$
a_t=f(q,o_{<t},z_T),
$$

其中 $z_T$ 是 teacher 的参数知识。比如 teacher 凭记忆知道人物对应哪部作品，于是直接搜索导演国籍。student 看到的是同一个 query，却不知道这个 query 为什么成立。

所以 teacher 的轨迹可能只是“带隐藏前提的专家操作记录”，不是脱离 teacher 记忆后仍然闭合的 action program。普通蒸馏复制了动作顺序，却没有自动补上那些 teacher 没有写出来、但实际依赖的事实。

不过，刚才的 evidence injection 实验也不能证明这个解释；它只是绕过了 student 生成 search 的环节。要真正区分原因，应该逐级做实验：只给缺失的桥接事实、给 teacher query 让 student 自己执行搜索、给 student 自己搜到的 evidence、最后才给完整 teacher evidence。这样才能知道模型究竟卡在“找什么”“怎么搜”“搜到什么”，还是“拿到证据后怎么推理”。

## 这篇论文留下的真正问题

ASP 的结论可以接受：在当前这些小模型和任务上，强制搜索比让模型根据自身置信度自适应地跳过搜索更可靠。论文的 Table 2 显示，SFT-1.7B 即使只放过 probe 置信度最高的 5% 问题直接回答，性能仍下降 4.8 分；OPD-1.7B 下降 9.0 分。

但这不等于自适应搜索原则上不可能。它更可能说明，当前模型的 confidence 没有校准成一个好的“搜索价值”信号，或者模型不会把这个信号转成可靠的行动。

于是我最后留下的问题不是“如何让小模型总是搜索”，而是：

> **如何把模型的不确定性，转化为对缺失信息的诊断，再转化为一次有效的搜索行动？**

如果能解决这个问题，Always-Search 就会从一个安全的默认策略，退回到它应该所在的位置：一种简单但昂贵的 baseline。现在它之所以有效，恰恰是因为小模型还没有学会在“不确定”与“下一步该做什么”之间建立稳定的联系。

## 参考

- Yizhou Liu, Qi Sun, Yulin Chen, Siyue Zhang, Chen Zhao. [Search, Do not Guess: Teaching Small Language Models to Be Effective Search Agents](https://arxiv.org/abs/2604.04651), 2026.
