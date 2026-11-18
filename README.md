# Recursive Self-Improvement 研究笔记

第一阶段整理公开研究、机制解释、手算与可核查的问题，不实现自行修改权限或规则的系统。笔记由 Codex 辅助整理，研究结论与假设示例分开记录。

- [公开资料清单](SOURCES.md)
- [首批主题和验收条件](BACKLOG.md)
- [从一次反馈到可检验的改进](notes/01-feedback-and-evaluation.md)
- [同一道题重复四次，为什么不等于四道新题](notes/06-repeated-task-uncertainty.md)
- [累计成功率上升，是否说明代理越来越会做题](notes/05-cumulative-success.md)
- [自动设计代理时，怎样把搜索与运行预算算清楚](notes/04-search-budget.md)
- [反馈之后，究竟什么状态发生了变化](notes/03-feedback-state.md)
- [为什么选出的最好分数会高估改进](notes/02-selection-bias.md)

协作协议与唯一交接队列：[codex-collaboration](https://github.com/lw-liuwang/codex-collaboration)。B=ZicongHe2002 维护本仓库；A=lw-liuwang 参与撰写和核查。问题在 Discussions 的 Q&A 中提出，由另一人回答，提问者实际核验后采纳。未开展训练实验，不将基准提升等同于递归能力提升。

- [通过检查的候选为什么仍可能大多错误](notes/08-verifier-base-rate.md)
- [一次改进与改进能力提升怎样区分](notes/07-recursive-evidence.md)
- [专挑可疑候选审计，怎样还原总体错误率](notes/28-stratified-audit.md)
- [平均延迟相同，截止时间成功率为何倒转](notes/29-deadline-success.md)
- [反复查看测试结果后再决定停止，会怎样改变假阳性](notes/30-repeated-testing-stop.md)
- [核算十步工作流末步改善的成功增益与失败即停成本](notes/31-workflow-joint-success.md)
- [推导重复修订的修复破坏递推与固定点](notes/32-refinement-repair-regression.md)
- [加入完美复核行动后推导直接返回的效用阈值](notes/34-decision-cost-threshold.md)
- [核算严格等效容差下的整数样本门槛与格点边界](notes/33-equivalence-and-improvement.md)
- [补写三策略潜在结果表以分离反馈与额外计算](notes/35-feedback-randomized-comparison.md)
- [用条件路径核算误拒误放与串行后备成本](notes/36-fallible-fallback-router.md)
- [吞吐量为什么不能平均每请求倒数耗时](notes/81-throughput-aggregation.md)
