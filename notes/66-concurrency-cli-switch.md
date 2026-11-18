# 命令行里有并发开关，为什么仍可能无法关闭并发

复现实验需要记录实际解析后的配置。参数名看起来像布尔开关，不代表命令行能够表达True和False；如果默认值与置True动作重合，带不带这个开关就没有区别。

## 固定ADAS中的参数定义

原命令行将multiprocessing定义为 `action='store_true', default=True`。store_true在出现该选项时设为True，省略时采用默认True，所以这两个常用入口都不会产生False。

只提取原argparse配置，未运行search、evaluate或线程池，得到：

| argv | 原解析结果 |
| --- | --- |
| 空列表 | multiprocessing=True，max_workers=48 |
| --multiprocessing | multiprocessing=True，max_workers=48 |
| --multiprocessing False | 解析失败，退出码2 |
| --no-multiprocessing | 未识别选项，退出码2 |

第三行的False不是store_true接受的布尔值；第四行的否定形式也不会凭空出现。这不等于Python调用者不能构造multiprocessing=False的args对象，只说明原CLI没有提供这条配置路径。

## 实际用的是线程池及独立的worker数量

evaluate_forward_fn在multiprocessing为真时，设max_workers为min(任务数,args.max_workers)，否则设为1；随后两条路径都进入ThreadPoolExecutor。该参数名称不能用来证明启动了多进程。

将max_workers设为1可限制这个线程池的工作线程数，但不等于清除候选内部并发、SDK重试或外部服务排队；也不等于命令行布尔值已经变成False。应把解析配置、执行器类型、实际worker上限分别记入运行账本。

若接口需要显式支持两种模式，可以提供成对的正/负选项或BooleanOptionalAction，并验证省略、显式开启、显式关闭三个入口。改变默认值可能影响现有命令，应明确兼容策略；本篇只核对原解析行为，不声称已经修改作者实现。

并发度影响吞吐、限流和调用交错，因此相同题目、模型与随机种子仍未涵盖完整运行条件。这里没有启动线程、请求模型或测量速度，不从解析表推算任何性能收益。

练习：想关闭并发时传--multiprocessing False，原脚本会得到False吗？答案：不会；这个action不接收后续值，原解析器将报错。

来源：[ADAS固定提交2702bee的CLI与evaluate_forward_fn](https://github.com/ShengranHu/ADAS/blob/2702bee8fefda42255efc5be9f60e3bd3db96ae4/_mmlu/search.py)。固定Git blob核对，AST原argparse配置与四组argv实际执行，线程池类型及worker赋值按原代码核对。无SDK、模型、线程或基准运行。Codex辅助整理。
