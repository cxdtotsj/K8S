# Job与CronJon

## 三种常用的、使用Job对象的方法

1. 也是最简单粗暴的用法：外部管理器 +Job 模板。

2. 拥有固定任务数目的并行 Job。

3. 指定并行度（parallelism），但不设置固定的 completions 的值。