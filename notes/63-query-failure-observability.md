# 模型调用失败被补成空字段后，评分还能区分什么

无效答案、服务异常和正确返回但答错，是不同事件。若接口把多种失败都折叠成空字符串，后续准确率即使仍可计算，也无法单靠这个分数解释系统究竟改进了什么。

## 原query的异常边界

固定ADAS的LLMAgentBase.query先准备空response_json，再调用辅助函数；异常时，只有错误文本包含小写的 `maximum context length` 且处于SEARCHING_MODE，才抛出新的AssertionError。其余异常会进入字段修补，对空对象补齐要求的键并设为空字符串，随后照常返回Info列表。这个返回列表没有携带原失败类型或原因。

给原query注入本地辅助函数桩，直接抛ValueError，仅检查辅助函数异常已经传播到query之后的行为，不调用SDK或重试装饰器：

| 搜索模式 | 桩异常文本 | 原query的结果 |
| --- | --- | --- |
| True | temporary unavailable | 返回reasoning和answer，两值都为空 |
| True | maximum context length exceeded | 抛AssertionError |
| False | maximum context length exceeded | 返回两个空字段 |
| True | Maximum context length exceeded | 返回两个空字段 |

最后一行首字母大写，原区分大小写的子串检查没有命中。这不说明任何真实SDK必定采用某个文本；它说明控制流依赖错误消息的具体拼写。实际服务调用可能先重试、抛其他类型或返回其他内容，本地桩没有测量那些路径。

## 空字段可以保留分母，却丢失原因

若候选把这个两项Info列表直接返回给原评估器，第二项空答案不在A/B/C/D中，解析异常会计0。一次正常返回但答案为错误选项，也可能计0；仅保存acc_list，无法区分这两种来源。另一方面，上表的AssertionError可能向外传播并触发候选调试，不能把所有异常都当成自动按题计0。

将失败按0计分可以是事先规定的总体可靠性指标，但仍应保留原定任务分母、失败类别和重试成本。如果改用“仅对成功调用算准确率”，分母改变，不能和前一种指标直接比较。一次得分提高可能来自接口更稳定或解题更准确，需要诊断记录才可区分。

## 用显式状态跨越接口

可以让返回记录同时包含status、字段内容、error_type、可公开的诊断摘要与尝试次数，由预先规定的策略决定重试、判失败或终止候选。优先使用已确认的结构化异常信息；仅把文本转小写只能修正一个匹配例子，不能建立完整错误协议。

状态记录不能取代正确性标签，也不能把暂时服务异常自动称为算法失败。需要分别评价运行可靠性和有效返回条件下的解题表现，再说明组合指标及成本。与worker直接抛异常的边界对照见尚待协作的[任务异常PR](https://github.com/ZicongHe2002/rsi-notes/pull/55)。

练习：两个版本在20道题都记10个0分，能否推出它们的服务可靠性相同？答案：不能；这些0可能分别来自正常错误答案、空字段或其他解析失败，必须读取失败类别和实际调用记录。

来源：[ADAS固定提交2702bee，LLMAgentBase.query及结果解析](https://github.com/ShengranHu/ADAS/blob/2702bee8fefda42255efc5be9f60e3bd3db96ae4/_mmlu/search.py#L109)。Git blob `00f46e0c8feda7e1acf022bea0ac706499d198ef`已核对，原类AST与四个本地异常桩实际执行，未模拟真实服务分布、SDK重试或模型效果。Codex辅助整理。
