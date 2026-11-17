# 公开资料起点

登记日期：2026-11-17。以下为原始论文入口；标题和可访问性已核对，尚未完成逐节技术复核。发表版本与 arXiv 修订可能不同，引用具体结论前须固定版本、章节和实验表，不把摘要当作已复现实验。

| 原始资料 | 改进对象与反馈 | 首批阅读问题 |
| --- | --- | --- |
| Shinn 等，2023，[Reflexion: Language Agents with Verbal Reinforcement Learning](https://arxiv.org/abs/2303.11366)，从 v1 开始对照后续修订 | 代理保存文字反思，后续尝试使用反馈；需区分上下文记忆与权重更新 | 反思内容是否在未见任务有效？评估器给了哪些额外信息？ |
| Madaan 等，2023，[Self-Refine: Iterative Refinement with Self-Feedback](https://arxiv.org/abs/2303.17651)，v1 起点 | 生成、反馈和修订输出的迭代 | 固定模型改善单次输出，何时可以称为持久的系统改进？ |
| Hu 等，2024，[Automated Design of Agentic Systems](https://arxiv.org/abs/2408.08435)，v1 起点 | 搜索代理设计，以任务评价反馈筛选候选 | 搜索预算、验证集复用与迁移评价如何区分？ |

上述工作提供相关机制，不能仅因标题涉及反馈或自动设计就直接认定实现了开放式 RSI。下一步由协作者为每篇补固定版本、相关章节、评价设置和可支持的最窄结论。

评价方法参考：Cawley 与 Talbot，[On Over-fitting in Model Selection and Subsequent Selection Bias in Performance Evaluation](https://jmlr.org/papers/v11/cawley10a.html)，JMLR 11 (2010), 2079–2107，最终发表版。已核对 §4、§5、§5.1 与本文使用的论点；说明选择过程如何影响评价，见[选择偏差笔记](notes/02-selection-bias.md)，不作为 RSI 实现证据。
