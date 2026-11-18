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
- [同一组JSON字段为何可能被读成不同答案](notes/59-json-field-order.md)
