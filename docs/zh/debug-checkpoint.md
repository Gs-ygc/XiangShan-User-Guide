# Checkpoint 与切片 {#sec:debug-checkpoint}

## 采样点语义化

- `CHECKPOINT_HERE`：在插入 `nemu_signal(CHECKPOINT_HERE)` 的位置直接采样，使 checkpoint 携带准确的 workload 语义
- 节点名唯一化流程：自动为计算图节点生成唯一名字，避免人工修改

TODO：补充使用方法与注意事项。

## QEMU checkpoint + RTL restore

TODO：补充流程步骤、常见失败模式（死循环、非法指令等）与修复情况。

## 模型打包

- 大模型可打包进 `gcpt.bin` 以便在功能模拟器与 RTL 之间复用

TODO：补充打包脚本与体积限制。

## 切片与加速

- 按算子/层切片运行，缩短单次仿真时间

TODO：补充切片粒度建议。
