# 附录 A：相对昆明湖 V2R2 的差异清单 {#sec:appendix-diff}

| 模块 | 增量 | 影响面 | 详见 |
| --- | --- | --- | --- |
| 矩阵计算单元 | 新增矩阵计算与访存通路、转置引擎 | 指令集、访存、性能 | [微架构增量](xsai-microarchitecture.md#sec:xsai-microarchitecture) |
| 矩阵控制通路 | 新增命令下发、完成通知与提交语义 | 指令提交、异常 | 同上 |
| 寄存器 | 矩阵、配置、同步寄存器 | 编程模型 | [数据寄存器](xsai-registers.md#sec:xsai-registers) |
| CSR/设备树 | 扩展状态与精度能力描述方式变更 | 固件、OS | [特权模式与控制状态寄存器](xsai-csr.md#sec:xsai-csr) |
| 高带宽 L2 | HBL2 与矩阵专用访存通路 | 内存子系统、性能 | [内存子系统与 HBL2](xsai-memory.md#sec:xsai-memory) |
| 时钟域 | 矩阵单元可异步，支持同频直连 | 集成、时序 | [微架构增量](xsai-microarchitecture.md#sec:xsai-microarchitecture) |
| 性能监测 | AI 相关事件与结构化追踪 | 调优 | [性能监测](xsai-pmu.md#sec:xsai-pmu) |
| 验证与调试 | 差分验证的矩阵支持、checkpoint 采样 | 验证流程 | [差分验证](debug-difftest.md#sec:debug-difftest)、[Checkpoint](debug-checkpoint.md#sec:debug-checkpoint) |

TODO：补充通过率/覆盖率等量化状态。
