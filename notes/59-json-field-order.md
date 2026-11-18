# 同一组JSON字段，为什么可能被读成不同答案

验证字段集合，只解决“有没有这些字段”；把对象再转成列表时，还要解决“按什么顺序传递”。如果生产者按响应中的成员顺序输出，消费者却约定第二项一定是答案，同一组字段和值也可能产生不同评分。

## ADAS中两个位置的契约没有自动对齐

固定ADAS提交的 `LLMAgentBase.query` 在得到response_json后，用 `for key, value in response_json.items()` 依次生成Info，最后返回列表。这里没有按self.output_fields重排。

同一文件的 `evaluate_forward_fn` 在处理列表结果的分支中，取 `res[1]`，再把其content解释成选项；该分支没有检查所取Info的name是否为answer。只有候选实际把这个列表作为最终结果返回时，才会走到这条位置依赖路径，不能据此断言所有设计都会受影响。

构造要求字段 `['reasoning','answer']`，让本地桩返回以下两段JSON。reasoning取字母A仅用于使误取字段仍能被解析，答案真值为B：

| 响应文本 | 原query产生的Info顺序 | 第二项内容 | 对B计分 |
| --- | --- | --- | --- |
| `{"reasoning":"A","answer":"B"}` | reasoning、answer | B | 1 |
| `{"answer":"B","reasoning":"A"}` | answer、reasoning | A | 0 |

两者解析后的字段集合和值完全相同，数量检查也都通过。Python 3.14.8的本地json.loads保留这两次插入顺序，原query继续传递该顺序，原列表解析循环遂得到[1,0]。这定位了实际依赖：不是内容发生变化，而是接收方把第二项的意义固定成了答案。

## 显式选择字段，再验证唯一性

可以让生产者按预定义字段顺序输出，同时让消费者按name查找答案，避免把响应文本顺序暗中当作接口。查找也不是“找到一个就行”：如果列表里有多个answer，应明确拒绝重复或定义消歧规则；否则顺序问题会变成另一个取首项问题。

至少应核对必需字段是否齐全、名称是否唯一、内容类型及选项范围，然后再用外部标签核对正确性。若选择固定位置接口，则把字段顺序写进契约，并在边界检查长度及每个位置的名称；不能只靠自然语言请求模型“按示例输出”。

另一个容易混淆的问题是缺失字段却长度正确，见尚待协作合并的[字段集合PR](https://github.com/ZicongHe2002/rsi-notes/pull/51)。本文的两个对象均没有缺字段，单独增加键集合检查不能修复这里的顺序依赖。

练习：把第二行的reasoning改成“我认为答案为B”，位置协议会自动恢复正确吗？答案：不会；它仍然取reasoning。可能因内容不属于选项而计0，也可能走其他解析路径，不能把碰巧含有正确字母当成字段选择正确。

来源：[ADAS固定提交2702bee的query与evaluate_forward_fn](https://github.com/ShengranHu/ADAS/blob/2702bee8fefda42255efc5be9f60e3bd3db96ae4/_mmlu/search.py)。已核对Git blob `00f46e0c8feda7e1acf022bea0ac706499d198ef`，AST提取原类与原逐结果循环，以本地JSON桩复核，未运行模型、SDK或基准；两段响应为教学构造，不是论文观测。Codex辅助整理。
