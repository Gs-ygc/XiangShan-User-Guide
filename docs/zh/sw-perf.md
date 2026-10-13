# 性能分析与调优 {#sec:sw-perf}

## Benchmark

TODO：补充基准集构成、运行方式与指标口径（端到端吞吐/延迟、算子级周期与利用率）。

## Profiling 工具

- 算子性能表：cycles/run、IPC、GOP/s、有效带宽、算术强度、峰值占比
- 结构化追踪与热点定位：PC 热点、uop 展开、issue wait、DRAM ROI

TODO：补充工具用法与报告示例。

## 调优案例

TODO：补充典型算子的优化过程与收益（GEMM、softmax、FlashAttention 等）。

## 常见瓶颈

- 访存带宽利用率不足（部分算子带宽利用率偏低）
- tile 划分与缓冲策略选择
