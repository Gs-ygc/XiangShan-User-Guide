# 调试支持 {#sec:xsai-debug}

## 差分验证支持

* 矩阵指令的差分对比，含矩阵寄存器哈希对比；
* 支持 CPU 参考模型与 GPU 参考模型两条路径。

详见[差分验证](debug-difftest.md#sec:debug-difftest)。

## Checkpoint 与采样

* `CHECKPOINT_HERE` 语义化采样点；
* QEMU checkpoint 与 RTL restore 流程。

详见[Checkpoint 与切片](debug-checkpoint.md#sec:debug-checkpoint)。

## 调试接口

TODO：补充与基线调试接口的差异（断点、单步、与矩阵异步状态的一致性）。

## 已知问题

* 裁剪配置下向量指令在特定场景卡死（完整配置不复现），待定位归属并上报。

TODO：补充结论。
