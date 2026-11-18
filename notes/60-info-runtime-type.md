# 消息字段完全相同，为什么仍可能没有进入提示词

两个消息拥有相同字段和值，不表示它们属于同一个运行时类型。若代理只接收某个具体类，而边界又没有显式报错，结构相同的消息也可能被直接跳过。排查“模型没看见任务”时，应该先核对最终提示词，而不是仅检查上游消息看起来正确。

## 原实现检查的是类身份

ADAS固定提交在模块中定义 `Info = namedtuple('Info', ['name','author','content','iteration_idx'])`。`LLMAgentBase.generate_prompt` 遍历input_infos时，仅在 `isinstance(input_info, Info)` 为真时解包；否则continue。任务字段name为task时，才把content放入 `# Your Task` 段落。

这个分支没有对被跳过的输入报错。若所有输入都不满足类型检查，最终prompt仍可包含调用者传入的instruction，却没有这些输入的任务内容；后续生成调用并不因此自动停止。

## 相等的namedtuple也可以被跳过

构造规范Info对象，四个字段为 `('task','User','unique_question_xyz',-1)`。然后再调用一次namedtuple工厂，使用完全相同的类名和字段名，构造另一个对象：

| 输入表示 | 与规范Info的四元组值相同 | isinstance检查 | 提示词含任务标记 |
| --- | --- | --- | --- |
| 原Info类实例 | 是 | 通过 | 是 |
| 重新创建的同名namedtuple类实例 | 是 | 不通过 | 否 |
| 普通tuple | 是 | 不通过 | 否 |
| 同字段dict | 字段和值相同，但不是四元组 | 不通过 | 否 |
| 继承原Info类的子类实例 | 是 | 通过 | 是 |

本地用原generate_prompt逐项核对，三种不通过的情况都只剩instruction。第二行的两个对象甚至可用tuple值比较判定相等，仍不能因此通过类检查；第五行则说明该检查允许子类，并不要求type(x)恰等于Info。

## 边界应选择一种协议并可见地失败

如果接口有意要求原Info类，应复用同一个类定义，在进入模型调用前拒绝不匹配对象，并记录字段名或输入位置。若需要跨进程、模块或序列化边界传递消息，可以先规定结构化记录，再由唯一适配器验证字段、类型、必需内容并转换为规范Info。仅把isinstance删除并无完整验证可替代，会引入新的错误输入。

结构转换成功也不说明模型正确使用了任务，仍应保留最终消息或可审计摘要、输入数量和被拒绝数量。另一方面，当前反例不说明正常使用原Info的作者示例会丢任务，也不据可构造的类型错配估计实际发生率。

练习：用dict保存了四个正确字段，再传给这段原generate_prompt，任务会出现吗？答案：不会；原实现没有进行dict到Info的转换。需要在明确的边界适配后重新核对最终prompt。

来源：[ADAS固定提交2702bee中的Info定义与generate_prompt](https://github.com/ShengranHu/ADAS/blob/2702bee8fefda42255efc5be9f60e3bd3db96ae4/_mmlu/search.py#L21)。Git blob已核对为 `00f46e0c8feda7e1acf022bea0ac706499d198ef`；提取原类，以Python 3.14.8真实namedtuple、dict和子类进行本地提示词构造，未调用SDK、模型或真实任务基准。Codex辅助整理。
