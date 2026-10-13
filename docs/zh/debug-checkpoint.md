# Checkpoint 与切片 {#sec:debug-checkpoint}

## 为什么要切片

完整负载在 RTL 上代价很高（例如单层 checkpoint 需 2000 万周期以上、耗时 5–6 小时）。把关注区域切出来，才能把仿真代价压到可迭代的量级。

## 验证梯度

软件问题的默认梯度是"先快后慢"：

```bash
make run-qemu                                  # 最快，先排软件与启动路径
make run-nemu                                  # 次快，且是 golden model 侧的中间层
make ckpt                                      # 制作 checkpoint
make run-nemu PAYLOAD=<checkpoint.zstd>        # 在 NEMU 上验证 checkpoint
make run-emu  PAYLOAD=<checkpoint.zstd>        # 最后在 RTL 上运行
```

## workload 侧约定

关注区域由 workload 自己通过 `nemu_signal(...)` 声明，外层脚本负责选择 checkpoint 流程：

| 信号 | 含义 |
| --- | --- |
| `DISABLE_TIME_INTR` | 准备采样：关闭定时器中断 |
| `NOTIFY_PROFILER` | 进入关注区域 |
| `NOTIFY_PROFILE_EXIT` | 退出关注区域 |
| `GOOD_TRAP` | 正常结束 |

典型最小骨架：在关注区域前后各发一次信号，`make profile`、`make cluster`、`make ckpt` 即可据此工作。

## CHECKPOINT_HERE

`CHECKPOINT_HERE` 允许在插入 `nemu_signal(CHECKPOINT_HERE)` 的位置**直接采样**，而无需等到基本块结束：

* checkpoint 携带准确的 workload 语义（不必事后猜测采样点对应哪段软件）；
* 计算图中的节点名会被自动唯一化，采样点通过运行参数指定，不需修改代码；
* 便于后续多次插入、切换不同算子组合的采样需求。

## 模型打包与系统级场景

* 大模型可打包进 `gcpt.bin`，在功能模拟器与 RTL 之间复用，避免重复的权重加载；
* 需要 Linux 环境的负载，可使用带操作系统的镜像，并配合 checkpoint restore 流程；
* restore 路径历史上出现过死循环、非法指令等问题，相关修复已合并，遇到异常请先确认所使用的版本。

## 注意

* QEMU 与 RTL 的 `timebase-frequency` 假设不同，DTB 不可直接混用（见[常见故障排查](debug-troubleshooting.md#sec:debug-troubleshooting)）；
* 切片越细，单次迭代越快，但需要确认切片起点处的缓存状态与完整运行一致，否则性能结论会偏移。
