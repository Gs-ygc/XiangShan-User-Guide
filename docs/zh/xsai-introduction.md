# 概述 {#sec:xsai-introduction}

## 适用范围

香炉峰（Xianglufeng，项目代号 XSAI）是在香山昆明湖 V2R2A（KMHv2R2A）基础上扩展的、面向 AI 推理的 RISC-V 处理器。本手册只描述 **香炉峰相对昆明湖 V2R2A 的增量**；通用部分请参见《香山开源处理器用户手册》。

## 香炉峰与昆明湖 V2R2A 的关系

- 基线：香山昆明湖 V2R2A
- 增量主线：AME 矩阵计算扩展、矩阵控制通路（AMU/CUTE）、高带宽 L2（HBL2）、外部 LLC、异步矩阵执行
- 软件可见变化：新增指令与 CSR、Device Tree 能力描述、矩阵访存对齐与转置约束、异步同步语义
- 调试可见变化：difftest 对矩阵寄存器的对比支持、矩阵相关性能计数器、checkpoint 采样机制

TODO：补充整体框图（Core + CUTE + AMU + HBL2 + LLC + 时钟域），以及各版本（V2R2A / V3）的差异说明。

## 相对昆明湖 V2R2A 的增量一览

| 维度 | 增量内容 | 详见 |
| --- | --- | --- |
| 指令集 | ZAMES / ZMASYNC 扩展、矩阵访存与转置、低精度转换自定义指令 | [@sec:xsai-instruction-set] [指令集扩展](xsai-instruction-set.md) |
| 寄存器 | 矩阵寄存器（A/B、C/acc tile）、配置寄存器、同步寄存器 | [@sec:xsai-registers] [数据寄存器](xsai-registers.md) |
| CSR | AME 状态与精度能力描述 | [@sec:xsai-csr] [特权模式与控制状态寄存器](xsai-csr.md) |
| 异常 | 新增指令的非法指令/对齐/未定义行为约定 | [@sec:xsai-exception] [异常与中断](xsai-exception.md) |
| 内存 | HBL2、矩阵专用访存通路、RAW 旁路与 miss 处理 | [@sec:xsai-memory] [内存子系统与 HBL2](xsai-memory.md) |
| 性能监测 | AI 相关 HPM 事件、ChiselDB 结构化追踪 | [@sec:xsai-pmu] [性能监测](xsai-pmu.md) |
| 调试 | difftest 矩阵寄存器对比、checkpoint 采样 | [@sec:xsai-debug] [调试支持](xsai-debug.md) |
| 时钟 | acceleratorClock 异步/同频直连（可配置） | [@sec:xsai-microarchitecture] [微架构增量](xsai-microarchitecture.md) |

## 阅读路径建议

- 硬件使用者：第「指令集扩展」「数据寄存器」「CSR」「内存子系统与 HBL2」章
- 验证/调试人员：「仿真与调试」各章
- 软件开发者：「软件开发」各章
