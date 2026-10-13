# 常见故障排查 {#sec:debug-troubleshooting}

## 构建类

### mix Nix / 非 Nix 的 rootfs 产物

**日志**：`error: patchelf is required to rewrite interpreter /nix/store/...`

**原因**：旧的 Nix 构建产物被复用到非 Nix 构建中。

**处理**：

```bash
source env.sh
make -C firmware/riscv-rootfs clean
make firmware
```

### 内核 panic：No working init found

**日志**：`Failed to execute /init (error -2)` 后 `Kernel panic - not syncing`。

**原因**：initramfs 内有 `/init`，但 busybox 指向的加载器路径在镜像中不存在。

**处理**：重建 rootfs（切换过环境时先清理旧产物）：

```bash
make -C firmware/riscv-rootfs clean
make firmware
```

## 运行类

### RTL 下 Linux 启动极慢

**现象**：OpenSBI 与早期 Linux 日志正常，但后续启动耗时数分钟至数小时，且 guest 时间远慢于主机时间。

**原因**：Linux 的调度时间来自 `mtime` 与设备树的 `timebase-frequency`：

```text
每 guest 秒所需 RTL 周期数 = RTC 分频 * timebase-frequency
```

仿真硬件默认对 RTL 时钟 100 分频，若 DTB 声明 10 MHz，则一个 guest 秒需要 10 亿周期——在 6 kHz 的仿真速率下约合 46 小时。仓库自带的 XSAI Linux 镜像与生成的 DTS 使用 1 MHz，而 QEMU 的 `nemu` 机器使用 10 MHz，**因此 DTB 不能在 QEMU 与 RTL 之间直接搬运**。当前 FPGA 打包流程驱动 XSTop 50 MHz、RTC 分频 100，对应 500 kHz。

**检查**：

```bash
fdtget firmware/nemu_board/dts/build/xiangshan_ai.dtb /cpus timebase-frequency
grep -E 'CONFIG_(HZ|NO_HZ|RISCV_TIMER|CPU_FREQ|RTC_CLASS)' firmware/riscv-linux/.config
```

内核启动日志中的 `Platform Timer Device : aclint-mtimer @ ...Hz` 应与之一致。注意：启用 RTC 驱动或 CPUFreq **不能**修正 mtime 比例问题。

### 挂起与卡死

* 通用卡死看门狗只统计"完成类里程碑"（命令被接受、释放脉冲、内存侧完成），阈值随已接收命令自适应，超时可输出确定性签名，便于对比复现；
* 排查顺序：是否仍有指令提交 → 矩阵完成事件是否产生 → 访存响应是否返回；
* 云平台任务连续 600 s 无串口输出会被判卡死并终止，避免长期占板。

### 结果不一致

1. 先排除**未定义行为**：读取未写入的矩阵寄存器区域、缺少必要同步、越界读写；
2. 用差分验证把问题收敛到具体指令与时刻；
3. 确认参考模型与 DUT 使用**相同配置**（例如裁剪配置对完整配置会产生系统性误报）。

### 矩阵访存 miss 行为

历史上软件默认保证矩阵 load/store 命中 L2（早期缺陷的规避写法），因此常规测试路径**不会触发 miss 通路**，导致 miss 相关统计长期为异常值（如全是 hit）。若需评估 miss 性能，应先在裸机小用例上验证 miss 通路本身是否正确。

## 云平台与 CI

| 现象 | 判断 |
| --- | --- |
| job `pending`，另有 job `running` | 正常排队，不必重推或取消 |
| job `running`，串口长时间静默 | 多为负载计算期；构建/烧录阶段超过 5 分钟无输出才可疑 |
| CI 大面积失败/挂起 | 先确认是否并发占用导致，排队重跑 |
