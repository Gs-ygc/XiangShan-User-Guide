# 性能分析与调优 {#sec:sw-perf}

## Benchmark

TODO：补充基准集构成与运行方式。

报告里的性能数字必须写清口径，否则数字无意义：

| 口径 | 含义 |
| --- | --- |
| 单 call 周期 | 一次算子调用（含同步开销）的周期数 |
| per-MAC 周期 | 除以 MAC 数，用于横向比较不同形状 |
| 整网吞吐 | 端到端 token/s 或 fps |

## Profiling 工具

* 算子性能表：cycles/run、IPC、GOP/s、有效带宽、算术强度、峰值占比；
* 热点定位：PC 热点、uop 展开、issue wait、访存 ROI；
* 结构化追踪与报告产出。

TODO：补充工具用法与报告示例。

## 调优案例

TODO：补充典型算子的优化过程与收益（GEMM、softmax、FlashAttention 等）。

## 常见瓶颈

* **小算子启动开销占比过高**：需要算子融合，或用工作量门禁把不值得的任务交回 RVV；
* **访存带宽利用率不足**：部分算子的有效带宽明显低于峰值；
* **tile 划分与缓冲策略影响显著**：M 利用率不足时（卷积挂接点每次只给 8 个像素）应落到小核；
* **时钟配置**：矩阵单元与核不同频时，同步点位置会显著影响整体收益（见[微架构增量](xsai-microarchitecture.md#sec:xsai-microarchitecture)）。
