# 每个候选超时3秒，为什么不等于整轮3秒截止

单次检查预算、排队等待和整轮截止时间是不同约束。一个timeout参数即使被传给每个检查器，也不能由此推断所有候选会在相同总时长内完成。

## 固定HumanEval里的两个局部边界

评价器将timeout传给每个check_correctness调用，并用固定n_workers的线程池调度。check_correctness先创建Manager和子进程，再执行 `p.join(timeout=timeout+1)`；子进程仍存活时调用kill。子进程内部另在执行检查程序的位置进入time_limit(timeout)。

因此默认timeout=3不是整个check_correctness函数从入口到返回的严格3秒上界，更不是所有样本的共同截止。初始化、调度及外层等待处在不同边界。源码的time_limit使用ITIMER_REAL/SIGALRM，不能把仅阅读该路径说成已在Windows成功执行计时器。

本地仅用不启动进程的Manager/Process桩执行原check_correctness：传timeout=3，实际记录join参数4；is_alive桩为真时记录kill，为假时不调用kill。共享结果列表为空时，原返回记录均为timed out。这验证参数传递与分支，不是实际运行时间测量。

## 无真实等待的调度算例

设5项任务从时刻0一起可用、2个worker、每项实际服务恰为3个抽象时间单位，无启动与切换开销。虚拟时钟得到：

| 任务 | 开始 | 完成 |
| --- | ---: | ---: |
| 0、1 | 0 | 3 |
| 2、3 | 3 | 6 |
| 4 | 6 | 9 |

单项服务都是3，整批完成却为9；同质理想模型的一般式是ceil(N/W)乘服务时长。这里的3来自明确假设，不是对原检查器或真实机器耗时的估计；不能用这张表给实际整轮墙钟时间作严格上界。

## 共同截止需要另一层协议

若目标是整轮在某个总预算内结束，应单独记录批次起点、队列等待、实际服务、剩余截止时间、取消与未完成状态。每项超时不自动取消尚未开始的所有任务；超时/取消也不能伪装成已完成的正确性标签。报告质量时保留原定任务分母和明确政策，报告成本时保留已发生的工作。

练习：最后一个任务直到第6单位才启动，给它3单位服务预算，是否保证整批在第3单位结束？答案：不保证；排队时间与服务预算属于不同边界。

来源：[HumanEval固定提交b9453e8的线程池调度](https://github.com/openai/human-eval/blob/b9453e8f1865df0a0057a1383767caa400acfbcf/human_eval/evaluation.py)、[check_correctness与time_limit](https://github.com/openai/human-eval/blob/b9453e8f1865df0a0057a1383767caa400acfbcf/human_eval/execution.py)。两个Git blob核对，原检查器仅配合进程桩执行，另用纯虚拟时钟复算；未启动线程、进程、计时器，未运行生成代码、模型或真实基准。Codex辅助整理。
