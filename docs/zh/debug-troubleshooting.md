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

## 软件适配（推理框架）高频故障

接矩阵单元后的问题集中在"结果错"与"崩溃挂死"两类，按现象检索下表。

### 结果错

| 现象 | 根因 | 处理 |
| --- | --- | --- |
| 某层输出全 0，测试报 `error=0` | 核在 `K_pad` 超限时 `memset(dst,0); return`，静默写零 | 提高容量上限，并让形状门禁回落到框架自带核 |
| 像素行错位（第 1 行等于第 3 行） | 累加器行步长硬编码 64，而实际 `tile_n=128` | 读累加器一律用 `tile_n` |
| 尾包（`real_count < DST_XUNIT`）结果错 | A 的 per-sz 步长用了固定的 `DST_XUNIT`，尾包实际是紧凑布局 | 用包内像素数 `min(real_count, EX)` |
| 板级整层错、QEMU 全对 | 该 bit 用了非原生 tile 规格 | 只验探针选中的规格，不要跑另一套 |
| bf16 结果差 1 ulp 级、测试边界挂 | 结果未截断到 bf16 网格 | 参考实现与 epilogue 统一截断 |
| attention 输出 0 或 inf | Q 打包与 KV cache 按 fp32 存，而 bf16 核按 2 字节读 | 同时修 Q 打包、KV 存储、指针与跨距的字节单位 |
| GEMV / depthwise 路径结果与 bf16 语义不符 | 这些旁路没走低精度约定 | 补 bf16 语义，或显式回落 |

### 崩溃与挂死

| 现象 | 根因 | 处理 |
| --- | --- | --- |
| 板上 `rc=134`（`SIGABRT`） | 产物里硬编码了匿名池模式，`/dev/mem` 初始化失败路径 abort | 池模式只由外部环境传入 |
| 板上 `rc=255` | 匿名池（非物理连续）下算错，被进程内差分门禁拦截返回 -1 | 同上；板上必须走物理池 |
| 结果偶发错、非稳定复现 | `mrelease` / `macquire` 未覆盖所有退出分支，或 MSC 基址寄存器被编译器回收 | 钉 `s11`，同步覆盖全部退出路径 |
| 某个框架路径在该模拟器上 illegal instruction | 框架自带的 fp16 RVV GEMM 与模拟器版本不兼容，与矩阵单元无关 | 关掉该特性开关绕开 |

### 验证误判

| 现象 | 根因 | 处理 |
| --- | --- | --- |
| 输出解析为空 | awk 未初始化变量把空值传播给索引（argmax 落在 0 时打印空串） | `print idx + 0`；每次运行前删除旧输出文件 |
| "加了模型但没进镜像" | 只看构建日志，没核对镜像内容 | 构建后解包核对，或校验镜像尺寸变化 |
| 测试全绿但结论错 | 测试跑的是另一套规格或另一条路径 | 确认选中规格与实测路径一致（打印规格与调用计数） |
| 差分 dump 比对假失败 | 读到了 16 B 对齐的 padding（未初始化内存） | 只比对有效元素 |
| 自检"自洽地错" | 自检参考沿用了核的错误布局假设 | 参考实现的索引推导必须独立于核 |
| 预打包的 B tile 是旧数据 | `create_pipeline` 早于量化阶段 | 确认时序为 `load_param → load_model`（量化）`→ create_pipeline` |

### 构建与集成

| 现象 | 根因 | 处理 |
| --- | --- | --- |
| 交叉编译 CMake configure 失败 | 仍启用了宿主 x86 选项，`CMAKE_MAKE_PROGRAM` 未设 | 关闭宿主 ISA 选项，显式设置 `CMAKE_MAKE_PROGRAM` |
| 编译报找不到 `platform.h` | 漏了 build 目录的 include 路径 | 补 `-I<build>/src` |
| `sysroot` 变成 `/sysroot` | 同一行 `export A=x B=$A/y` 中 `$A` 尚不生效 | 分行 `export` |
| 模拟器报 `/lib/ld-linux-*.so.1 not found` | 漏设前缀环境变量 | 显式设置 `QEMU_LD_PREFIX` |
| 环境变量传 `"0"` 却当开关打开 | 判定用了 `getenv() != NULL`，只判空不判值 | 改为取值比较 |
