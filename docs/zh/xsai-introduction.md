# 概述 {#sec:xsai-introduction}

## 适用范围

香炉峰（Xianglufeng，项目代号 XSAI）是在香山昆明湖 V2R2 基础上扩展的、面向 AI 推理的 RISC-V 处理器。本手册只描述**相对昆明湖 V2R2 的增量**；通用部分请参见《香山开源处理器用户手册》。

## 与昆明湖 V2R2 的关系

* 基线：香山昆明湖 V2R2
* 增量主线：AME 矩阵计算扩展、矩阵控制通路、高带宽 L2（HBL2）、外部 LLC、异步矩阵执行
* 软件可见变化：新增指令与 CSR、设备树能力描述、矩阵访存对齐与转置约束、异步同步语义
* 调试可见变化：矩阵寄存器的差分对比支持、矩阵相关性能计数器、checkpoint 采样机制

## 增量一览

| 维度 | 增量内容 | 详见 |
| --- | --- | --- |
| 指令集 | ZAMES / ZMASYNC、矩阵访存与转置、低精度转换自定义指令 | [指令集扩展](xsai-instruction-set.md#sec:xsai-instruction-set) |
| 寄存器 | 矩阵寄存器、配置寄存器、同步寄存器 | [数据寄存器](xsai-registers.md#sec:xsai-registers) |
| CSR | 矩阵扩展状态与精度能力描述 | [特权模式与控制状态寄存器](xsai-csr.md#sec:xsai-csr) |
| 异常 | 新增指令的非法指令/对齐/未定义行为约定 | [异常与中断](xsai-exception.md#sec:xsai-exception) |
| 内存 | HBL2、矩阵访存通路、RAW 旁路与 miss 处理 | [内存子系统与 HBL2](xsai-memory.md#sec:xsai-memory) |
| 性能监测 | AI 相关 HPM 事件、结构化追踪 | [性能监测](xsai-pmu.md#sec:xsai-pmu) |
| 调试 | 差分验证的矩阵支持、checkpoint 采样 | [调试支持](xsai-debug.md#sec:xsai-debug) |
| 时钟 | CUTE 与核心的异步/同频直连（可配置） | [微架构增量](xsai-microarchitecture.md#sec:xsai-microarchitecture) |

## 阅读路径

* 硬件使用者：指令集扩展 → 数据寄存器 → CSR → 内存子系统与 HBL2
* 验证与调试：[仿真与调试]各章
* 软件开发者：[软件开发]各章
