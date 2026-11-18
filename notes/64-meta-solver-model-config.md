# 改了搜索模型参数，子代理模型为什么可能没有改变

自动设计代理有两个不同调用层：产生和反思设计的元模型，以及执行设计中任务的求解代理。一个叫model的搜索参数，不必然覆盖这两层。复现质量和费用时，应追踪实际调用参数，而不是只记录最外层命令行。

## 固定ADAS的两条配置路径

原search把args.model传给get_json_response_from_gpt_reflect，用于生成新设计及两次反思。LLMAgentBase则在构造时保存自己的model和temperature；其历史源码默认值为 `gpt-3.5-turbo-0125` 与0.5，query把这两个实例属性传给另一个辅助函数。原search没有在这个位置重写所有代理实例的model。

因此，候选明确传入了什么参数是必要证据。有些设计可能显式指定不同模型，有些使用默认值；不能由args.model的值推断每次求解都采用它。上述名称只描述固定提交中的默认设置，不声称旧模型当前仍可调用。

## 本地调用参数账本

给原search注入本地元生成与评价桩，运行一代，debug_max=1且评价成功；另用原类创建默认代理和显式配置代理。meta-X与solver-Y都是教学标记，不是实际服务调用：

| 路径 | 构造/搜索配置 | 原路径传给辅助函数的model | temperature |
| --- | --- | --- | --- |
| 初稿及两次反思 | args.model=meta-X | meta-X，共3次 | 此search调用位置未显式传入 |
| 默认LLMAgentBase | 不传model/temperature | gpt-3.5-turbo-0125 | .5 |
| 显式LLMAgentBase | model=solver-Y，temperature=.2 | solver-Y | .2 |

这个账本分别核对两条原调用路径，没有执行生成出的代理代码，也没有声称元模型会自然生成上述两个求解器。它证明各层参数存在独立来源，不能把一层的设置当成另一层已经采用的证据。

## 让配置与实际调用对应

可以将meta_model、solver_default_model及各实例覆盖分开命名，再显式传入候选构造协议。保存每次实际请求所用的模型标识、温度、token上限及调用角色；只有记录了服务返回信息时，才进一步描述具体版本或用量。环境中的默认值也应写入运行配置，避免升级代码后无意改变基线。

等预算比较还应分别计算搜索调用与求解调用，按各自实际模型和返回用量计账。即便两层使用同名模型，也不能仅据调用数假定token费用相同；生成设计、反思设计与回答题目的输入输出长度可能不同。本文没有真实token或价格测量，预算定义见[搜索预算笔记](04-search-budget.md)。

练习：args.model改成meta-X，候选内部仍调用不传model的LLMAgentBase，原类自动改用meta-X吗？答案：不会；必须显式连接配置，或从实际求解请求确认采用了什么模型。若候选自行覆盖，也应记录该覆盖，而不是假定全部实例一致。

来源：[ADAS固定提交2702bee，LLMAgentBase初始化/query与search](https://github.com/ShengranHu/ADAS/blob/2702bee8fefda42255efc5be9f60e3bd3db96ae4/_mmlu/search.py)。Git blob `00f46e0c8feda7e1acf022bea0ac706499d198ef`已核对，AST原类与原search的本地参数轨迹实际执行，未调用SDK、模型、真实候选或基准。Codex辅助整理。
