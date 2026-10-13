# 调试支持 {#sec:xsai-debug}

## 差分验证支持

- 矩阵指令的差分对比（含矩阵寄存器 hash 对比）
- 支持 CPU 参考模型与 GPU 参考模型两条路径

详见[@sec:debug-difftest] [差分验证](debug-difftest.md)。

## Checkpoint 与采样

- `CHECKPOINT_HERE` 语义化采样点
- QEMU checkpoint + RTL restore 流程

详见 [@sec:debug-checkpoint] [Checkpoint 与切片](debug-checkpoint.md)。

## 调试接口

TODO：补充与昆明湖 V2R2A 调试接口的差异（断点、单步、与矩阵异步状态的一致性）。

## 已知问题

- 裁剪版配置下 `vle32.v` 向量指令在特定场景卡死（default 配置不复现），待定位归属并上报

TODO：补充结论。
