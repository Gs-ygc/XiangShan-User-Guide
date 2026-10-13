# 系统软件与运行时 {#sec:sw-system}

## 固件与设备树

* 固件负责矩阵扩展初始化与相关 CSR 支持；
* 设备树描述矩阵能力（`riscv,matrix-isa-caps`）与相关硬件资源；
* **注意 `timebase-frequency`**：仿真与 FPGA 的 RTC 分频不同，DTB 不可在模拟器与 RTL 之间直接混用（见[常见故障排查](debug-troubleshooting.md#sec:debug-troubleshooting)）。

TODO：补充固件修改点与设备树示例。

## Linux 与 rootfs

* 支持在 riscv64 Linux 上运行推理框架；
* rootfs/initramfs 构建流程；模型可打包进 `gcpt.bin`。

TODO：补充构建步骤与体积约束。

## 运行时约束

* 矩阵访存的对齐与 stride 要求；
* tile 形状与数据布局约束；
* 量化格式与缩放因子约定。

TODO：补充约束清单与常见错误。
