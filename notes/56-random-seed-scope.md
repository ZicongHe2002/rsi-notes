# 固定洗牌种子，为什么仍不能复现整次评价

复现要控制的是一组随机过程，而不是一个叫seed的数字。代理搜索可能同时使用题目洗牌、候选内部采样、模型服务采样和结果重采样；重置其中一项，不会自动重置其余项。

## ADAS中的两个具体随机源

固定ADAS commit `2702bee8fefda42255efc5be9f60e3bd3db96ae4` 的 `_mmlu/search.py` 中，`evaluate_forward_fn` 在读取数据后执行 `random.seed(args.shuffle_seed)` 和 `random.shuffle(examples)`，再切验证/测试片段。这控制Python标准库模块的洗牌流，数据内容和初始行顺序也须相同。

同一commit的 `_mmlu/utils.py` 中，`bootstrap_confidence_interval` 则调用 `np.random.choice`。它使用NumPy模块的随机状态，没有在该函数内部根据shuffle_seed重置。Python的random与这个NumPy流是不同的状态；上述洗牌设置不能单独保证区间文本重现。

这不是说该种子只影响洗牌。`random.seed`重置的是模块共享流，此后使用同一个random模块的代码也会消费它，例如utils的random_id。若调用顺序改变，后面的模块级抽样可能改变。因此应记录种子控制的对象及消费顺序，不能只凭参数名称划边界。

## 本地状态对照

使用NumPy 2.3.5构造如下顺序，无模型调用：

1. 先将NumPy模块种子设为17，保存其状态；设置Python种子0并洗牌0至7，核对NumPy状态完全没变。
2. 从NumPy抽8个0至99的整数，记录为第一组；只重置Python种子0并重做同一洗牌，再从NumPy抽8个整数。洗牌序列相同，第二组整数与第一组不同，因为NumPy流继续向前。
3. 将NumPy种子重新设为17，第一组整数可以逐项重放。

还AST提取上述原bootstrap函数，对教学数据[0,0,1]只做7次重采样：重置NumPy状态可重放第一次输出，而只重置Python状态不重置其消费位置。这次前后两组格式化区间碰巧相同，说明区间字符串相等也不能用来确认随机状态相同。7次只用于检查状态协议，不能支持有效覆盖率或实用重采样次数建议。

## 怎样记录复现边界

| 对象 | 需要固定或保存什么 | 仅有shuffle_seed是否足够 |
| --- | --- | --- |
| 题目切分 | 数据版本、初始行序、洗牌流与切分参数 | 还缺数据与参数 |
| bootstrap数值 | 输入结果顺序、NumPy状态、次数、分位数约定及版本 | 不足够 |
| 模型输出 | 模型/提示/采样参数、服务支持的种子与服务版本、实际返回 | 不足够 |
| 调用轨迹 | 状态重置、缓存、并发和重试协议、随机流消费顺序 | 不足够 |

保存局部生成器并将它显式传入抽样函数，有助于减少无关代码消费全局流；它仍不能承诺外部服务或不同平台逐位一致。精确重放已记录的一次运行，也不同于跨多组种子估计方法稳健性。公平配对还需要相同任务与对应随机条件，见[配对随机性笔记](24-paired-randomness.md)；区间参数记录见尚待协作合并的[区间标签PR](https://github.com/ZicongHe2002/rsi-notes/pull/62)，以其最终版本为准。

练习：题目顺序完全一致，区间端点不同，是否必然说明模型结果不同？答案：不是；输入结果相同但bootstrap状态不同也可能改变有限次重采样端点。应先核对结果数组，再核对重采样协议，不能仅由区间字符串反推模型行为。

来源：[ADAS洗牌与评估](https://github.com/ShengranHu/ADAS/blob/2702bee8fefda42255efc5be9f60e3bd3db96ae4/_mmlu/search.py#L279)、[ADAS随机ID与bootstrap](https://github.com/ShengranHu/ADAS/blob/2702bee8fefda42255efc5be9f60e3bd3db96ae4/_mmlu/utils.py#L25)。已核对固定Git blob及本地NumPy状态，不声称作者报告了复现失败或调用过模型；本文为Codex辅助整理的代码路径分析与构造验证。
