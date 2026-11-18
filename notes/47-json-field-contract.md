# JSON字段数量正确，为什么仍可能缺少答案

合法JSON只保证文本能被解析成一种值。即使结果是对象、字段数量也符合要求，字段集合和字段值仍可能不符合代理之间的约定。若只按数量修补，错误字段可以占住缺失答案的位置。

## 固定ADAS路径如何检查

在固定提交的`LLMAgentBase.query`中，辅助函数返回`response_json`后，代码断言`len(response_json)==len(self.output_fields)`。异常处理先尝试补缺失字段，但还要求当前对象长度小于目标长度；随后删除多余字段，删除条件也要求当前长度仍大于目标长度。最后按对象现有条目生成`Info`，没有再验证字段集合相等。

构造`output_fields=['reasoning','answer']`，给原函数注入以下本地字典。表中只写返回字段和值，不调用模型：

| 辅助函数返回 | 最终Info所含字段 | answer状态 |
| --- | --- | --- |
| `{'reasoning':'r','answer':'B'}` | reasoning=r，answer=B | 存在 |
| `{'reasoning':'r','wrong':'B'}` | reasoning=r，wrong=B | 缺失，数量断言仍通过 |
| `{'reasoning':'r'}` | reasoning=r，answer为空串 | 被补空串，未证明有效 |
| `{'reasoning':'r','answer':'B','extra':'x'}` | reasoning=r，answer=B | 多余extra被删除 |

第二行直接说明“两个字段”不等于“要求的两个字段”。第三行虽然恢复了名称，空答案仍不是有效选项。恢复流程是否发生与答案内容是否正确，应该分别记录。

## 修补后的长度也不是后置条件

再构造按顺序插入的`{'reasoning':'r','wrong':'B','extra':'x'}`。最初长度3，数量断言失败；补answer时长度还不小于2，因此不补。删除阶段先删除wrong，长度降到2后便不再删除extra。最终留下reasoning和extra，answer仍缺失。

这里不是“任何多字段输入都会失败”：表中已有一个正确删掉extra的例子。问题是长度约束不能代替名称集合约束，而且先补后删时各阶段的条件会相互影响。所有输入均为教学构造，不声称这些JSON曾由论文模型输出，也不从可构造分支推断实际错误频率。

## 代理消息需要明确的数据契约

若契约要求恰好这两个字段，成功状态应至少满足：结果是对象、键集合等于要求集合、reasoning类型正确、answer属于允许的选项集合。若允许额外字段，则明确哪些字段可忽略，并单独检查必需字段；不要把缺失项默认为空串后继续声称“合法答案”。

结构检查仍不验证推理或选择是否正确。保存原始响应、解析状态、结构错误、修补行为和最终答案，可以区分格式失败、拒答与答错；若统一记0分，也应保留这些诊断类别，以便解释改进究竟减少了哪种失败。

练习：只有`reasoning`和`answer`两个键，但answer为`'E'`，键集合检查是否足够？答案：不足；四选一契约还需要检查值属于A/B/C/D，之后仍需独立标签核对正确性。

来源：ADAS固定提交`2702bee8fefda42255efc5be9f60e3bd3db96ae4`的[`_mmlu/search.py`，`LLMAgentBase.query`](https://github.com/ShengranHu/ADAS/blob/2702bee8fefda42255efc5be9f60e3bd3db96ae4/_mmlu/search.py)。下载后核对Git blob为`00f46e0c8feda7e1acf022bea0ac706499d198ef`；提取原类并以本地桩逐项核对。未运行模型、SDK或基准实验。Codex辅助整理。
